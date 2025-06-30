

# 第八章：探索 AWS 上的云秘密存储

在云中存储秘密的一个非常常见的方法是利用云服务提供商提供的基础设施。主要的云服务提供商有两个重要的基础设施组件，可以帮助我们在 Kubernetes 上高效地管理秘密：秘密存储和 KMS。

本章将探讨 AWS Secrets Manager、**弹性 Kubernetes 服务**（**EKS**）的集成，以及使用 KMS 进行秘密加密。具体来说，我们将涵盖以下主题：

+   AWS Secrets Manager 概述

+   秘密存储 CSI 驱动程序

+   AWS EKS 集群和 AWS Secrets Manager

+   KMS 用于 AWS Kubernetes 加密

到本章结束时，我们应该能够将 AWS Secrets Manager 作为外部 Kubernetes 秘密存储来使用，使用 AWS KMS 对 Kubernetes 上的秘密进行加密，并通过 AWS CloudTrail 和 AWS CloudWatch 搜索秘密操作的审计日志。

# 技术要求

为了将概念与实践相结合，我们将使用一系列常用的工具和平台来与 AWS API 和 Kubernetes 交互：

+   需要一个 AWS 免费套餐账户。免费套餐对于新的 AWS 客户在限定时间内可用。一旦该期限到期或使用量超过免费套餐使用限制，将按需付费。

+   **AWS CLI** ([`aws.amazon.com/cli/`](https://aws.amazon.com/cli/)) 是一个统一的工具，用于管理 AWS 服务。通过 AWS CLI 执行的命令会被转换为对 AWS API 的 API 调用。

+   **Terraform** ([`www.terraform.io/`](https://www.terraform.io/)) 是一款基础设施即代码软件，可用于在云中配置和管理基础设施。

+   **kubectl** ([`kubernetes.io/docs/reference/kubectl/`](https://kubernetes.io/docs/reference/kubectl/)) 是一个命令行工具，用于通过 Kubernetes API 与 Kubernetes 集群进行通信。

# AWS Secrets Manager 概述

秘密是一个存在于 Kubernetes 范畴之外的概念。任何类型的应用程序都会在某个时刻需要将敏感信息与每个部署集成。在云中部署的应用程序需要安全地处理秘密。因此，云服务提供商提供了秘密存储组件。

在 Kubernetes 中，我们在 *第一章*，《理解 Kubernetes 秘密管理》中看到，秘密信息存储在 etcd 中。本质上，etcd 是 Kubernetes 的默认秘密存储。关键问题是，除了 etcd，是否有可能在 Kubernetes 上使用外部存储来存储秘密。

这是可行的，前提是你主动使用云服务提供商的秘密存储，或者考虑利用它并在 Kubernetes 上使用它。由于容器存储接口和工作负载身份的存在，我们可以利用现有的秘密存储。

AWS Secrets Manager ([`aws.amazon.com/secrets-manager/`](https://aws.amazon.com/secrets-manager/)) 是 AWS 提供的秘密存储服务。使用 AWS Secrets Manager，我们可以存储多种类型的凭证，如数据库凭证、应用程序凭证和安全令牌。接下来，我们将重点介绍使 AWS Secrets Manager 成为处理密钥的优选方案的功能，从加密开始。

## 加密

AWS Secrets Manager 提供静态加密和传输加密功能。它使用 AWS KMS 执行封套加密。存储在 AWS Secrets Manager 中的值通过数据密钥加密，而数据密钥则使用 AWS KMS 进行加密。数据加密密钥是 AES-256 密钥。当 AWS Secrets Manager 中的值发生变化时，系统会生成一个新的数据加密密钥，并用它加密新值。我们还可以配置 AWS Secrets Manager 使用不同的 KMS 密钥。AWS Secrets Manager 还提供传输加密功能。与 Secrets Manager 的 API 调用通过安全的私有端点进行，每个调用都需要使用 X.509 证书或 Secrets Manager 秘密访问密钥进行签名。

## 版本控制

版本控制是使用 AWS Secrets Manager 的另一个好处。如果我们在 AWS Secrets Manager 中创建一个秘密，系统会为其分配一个版本号。这将是该秘密的第一个版本。一旦我们更新该秘密，就会获得更新后的版本，而旧版本仍然可用。当我们访问该秘密时，将检索到其最新版本。可以配置将先前的版本在某个日期自动删除，这样就无法再检索到它。如果我们想要检索先前的版本，必须先将其从待删除状态中移除。

## 密钥轮换

AWS Secrets Manager 还支持密钥轮换。可以配置一个 AWS Lambda 函数来执行定期的密钥轮换；AWS 文档提供了详细的指南（[`docs.aws.amazon.com/secretsmanager/latest/userguide/rotating-secrets.html`](https://docs.aws.amazon.com/secretsmanager/latest/userguide/rotating-secrets.html)）。在调用时，Lambda 函数会根据我们提供的自定义代码片段进行密钥轮换；函数模板可以在 GitHub 上找到（[`github.com/aws-samples/aws-secrets-manager-rotation-lambdas`](https://github.com/aws-samples/aws-secrets-manager-rotation-lambdas)）。在轮换过程中，将会测试密钥的最新版本。如果失败，轮换将会重试。

## 基于云的功能

除了满足敏感信息存储需求外，AWS Secrets Manager 还是一个托管的 AWS 组件。每个 AWS 组件都有其特定功能：

+   AWS IAM 集成

+   使用日志记录和审计

+   高可用性和灾难恢复

+   与其他 AWS 组件的集成

让我们逐一看看这些功能。

### AWS IAM 集成

通过 AWS **身份与访问管理**（**IAM**），我们可以指定可以访问 AWS 服务和资源的实体。这些实体可以是 AWS 用户或 AWS 角色。

AWS 用户旨在与实际的人类用户关联，这些用户希望与 AWS 服务互动并配置资源。AWS 角色是一种更灵活的身份。AWS 角色用于委派对服务、EC2 机器和 Kubernetes 工作负载的访问权限。

AWS 上的身份可以被授予细粒度的 AWS 服务权限。

### 日志记录与审计

AWS 提供了 CloudWatch，这是一个监控和可观测性解决方案。CloudWatch 从其他 AWS 组件收集实时日志和指标。在我们的案例中，通过 CloudWatch，我们可以通过日志识别 Kubernetes 操作；同时，我们可以基于开箱即用的指标创建自定义警报和仪表板。

CloudTrail 是 AWS 的一项面向审计的服务。通过 CloudTrail，我们可以跟踪 AWS 用户或 AWS 角色执行的操作。这些操作将被记录并通过 AWS CloudTrail 进行访问。例如，假设 EC2 机器上附加的角色检索一个密钥。运行在 EC2 机器上的代码被授权接收该密钥，密钥被检索，并且此操作会被记录到 AWS CloudTrail 中。EC2 机器具有与 AWS Secrets Manager 交互权限的角色；该角色是一个 AWS 身份，类似于 AWS 控制台用户。

在下图中，我们可以看到显示审计信息的 CloudTrail 屏幕：

![图 8.1 – AWS CloudTrail 屏幕](img/B20970_08_1.jpg)

图 8.1 – AWS CloudTrail 屏幕

借助 CloudTrail，我们可以识别在 AWS Secrets Manager 中发生的操作、执行该操作的身份以及执行时间。

### 高可用性和灾难恢复

在 Secrets Manager 上配置的密钥是区域性高度可用的。这意味着密钥操作将跨越一个 AWS 区域的三个可用区。如果某个可用区变得不可用，密钥的请求将由另一个可用区提供服务。

此外，使用 AWS Secrets Manager，我们可以实现灾难恢复。默认情况下，当我们创建一个密钥时，我们会选择一个可用区，这个密钥将驻留在该区域。为了保护我们的工作负载免受某个区域丢失的情况，我们会以灾难恢复可行的方式配置我们的基础设施。AWS Secrets Manager 通过允许我们将密钥复制到另一个区域，使这一过程变得更加简单。

### 与其他 AWS 组件的集成

使用云提供商提供的组件的一个好处是它们的高度集成性。AWS Secrets Manager 可以轻松与其他 AWS 组件集成。在我们的案例中，我们感兴趣的是 EKS，它是 AWS 上 Kubernetes 的托管版本。

我们已经概述了 AWS Secrets Manager，并了解了它如何与其他 AWS 组件集成。接下来，我们将继续介绍如何将 AWS Secrets Manager 与 EKS 集成。为此，我们将概述一款促进这一集成的工具——Secrets Store CSI 驱动。

# Secrets Store CSI 驱动

Kubernetes CSI 是 Kubernetes 的标准化接口，允许我们使用不同的存储提供商与 Kubernetes 配合使用。我们不仅限于使用 Kubernetes 上的默认存储，还有一个接口提供了一个规范，我们可以基于此规范构建存储驱动程序。通过这种方式，我们可以通过实现符合 CSI 接口的新驱动程序，使用多种存储类型。

这里是一些常用的 CSI 驱动：

+   AWS 弹性文件系统

+   Azure 文件

+   Google Cloud Filestore

这种方式适用于多种存储形式，也适用于云服务提供商提供的 Secrets 存储组件。

CSI 驱动由云服务提供商提供，用于秘密存储。Secrets Store CSI 驱动是一个专注于 Secrets 管理的 CSI 接口。我们可以通过它将 Secrets 挂载到 Kubernetes 中，利用另一种形式的存储。通过 Secrets Store CSI 驱动，我们可以从不同的外部来源获取 Secrets，而不是从 etcd 中消费 Secrets，在我们的案例中，这些来源正是由云服务提供商提供的 Secrets 存储解决方案。

在本节中，我们将重点介绍 AWS Secrets Manager，并探讨如何通过使用相应的 Secrets Store CSI 驱动将 Kubernetes 集群与 AWS 集成，从而使用云服务提供商的原生 Secrets 管理解决方案中的 Secrets。

## Secrets Store CSI 驱动的工作原理

我们有一个应用程序需要托管在 Kubernetes 上。该应用程序必须使用存在于云服务提供商 Secrets 存储解决方案中的秘密。该应用程序将会驻留在 Pod 上，因为 Pod 是 Kubernetes 的主要计算构建块。

一旦 Pod 被创建、启动或重启，Secret Store CSI 驱动通过 Secret Store CSI 提供者将与云服务提供商的 Secrets 存储进行通信，并检索凭证。这些凭证将作为卷挂载到 Pod 中。挂载的卷将会附加到指定的目录。

下一个问题是 Secret Store CSI 驱动如何工作。

Secret Store CSI 驱动是一个**DaemonSet**。DaemonSet 会在 Kubernetes 的每个节点上存在。一个 DaemonSet 可以包含多个 Pod。

对于 Secret Store CSI 驱动，我们有以下 Pods：

+   `node-driver-registrar`

+   `secrets-store`

+   `liveness-probe`

### node-driver-registrar

`node-driver-registrar` 将会把 CSI 驱动注册到 kubelet。会创建一个注册套接字，并通过 kubelet 插件注册表中的主机路径暴露出来。

### secrets-store

`secrets-store` 组件负责在 Pod 创建过程中挂载机密卷，以及在 Pod 删除时卸载卷。它基于 gRPC 实现了来自 CSI 规范的 CSI 节点服务。

### liveness-probe

这个 Pod 监控 CSI 驱动程序的健康状况。健康存活探针将检测驱动程序的任何问题，并会重启 Pod 来修复问题。在下图中，我们可以看到所有组件如何协同工作：

![图 8.2 – Secrets Store CSI 与 AWS Secrets Manager 的集成](img/B20970_08_2.jpg)

图 8.2 – Secrets Store CSI 与 AWS Secrets Manager 的集成

现在我们了解了 CSI 驱动程序，接下来将继续将其与 EKS 集成。

# 将 AWS Secrets Manager 与 EKS 集成

为了将 AWS Secrets Manager 与 Kubernetes 集成，我们将使用 Secrets Store CSI 驱动程序。正如预期的那样，AWS 为我们提供了 Secrets Store CSI 驱动程序 ([`github.com/aws/secrets-store-csi-driver-provider-aws`](https://github.com/aws/secrets-store-csi-driver-provider-aws))。为了将驱动程序与 Kubernetes 集成，我们将创建一个 Kubernetes 集群。

## AWS 上的 EKS 集群

如前所述，AWS 中的 **EKS** 代表 **弹性 Kubernetes 服务**。我们有选项在 AWS 云上设置 Kubernetes 集群，就像在其他云平台上一样。为了本章的目的，我们将重点关注托管服务，因为它需要更少的维护和设置开销。

如果已有现成的 EKS 集群，则可以忽略 Terraform 配置说明，因为 AWS 的相应命令行参数会提供。

在 AWS 上实施 Kubernetes 集群可能需要不同程度的努力，具体取决于您想要实现的安装类型。一种选择是创建完全私有的集群，这些集群无法连接到互联网，所有服务都通过 AWS 内部提供。另一种选择是创建一个公有网络上的集群。如今，比较常见的选项是将集群的节点部署在私有子网中，并确保通过 NAT 网关实现与互联网的连接。这就是我们将要遵循的选项。

### 配置 Terraform 项目

在 AWS 上的 Terraform 项目要求我们存储状态。我们可以将状态存储在本地文件系统中，但这对于面向生产环境的基础设施即代码（IaC）来说并不可行。我们的选择是使用 **安全存储服务** (**S3**) 桶来存储状态：

```
terraform {
...
  backend "s3" {
    bucket = "state-bucket"
    key    = "eks-state"
    region = "eu-west-1"
  }
```

默认情况下，Terraform 代码库将使用为 AWS CLI 配置的默认凭证。

我们首先需要使用 `init` 初始化 Terraform 项目：

```
$ terraform init
```

这个命令将初始化我们的项目。

使用 Terraform，我们可以使用 `plan` 选项，它类似于 kubectl 的 `dry-run` 命令。我们不会直接创建资源，而是使用 `plan` 来识别如果应用了 Terraform 脚本，基础设施的状态会是什么样：

```
$ terraform plan
```

为了应用基础设施，我们将使用 `apply` 命令：

```
$ terraform apply
```

#### Terraform 基础

在使用 Terraform 时，基础设施是通过 `.tf` 文件定义的。一个 Terraform 项目包含以下操作：

+   `init`

+   `plan`

+   `apply`

+   `destroy`

到目前为止，我们已经看到 `init`，它用于初始化项目并下载现有状态；`plan`，它用于评估我们所做的更改；以及 `apply`，它是用来执行更改的命令。若要销毁已配置的资源，我们可以使用 `destroy` 命令。

这些操作可以在项目中执行，也可以针对 Terraform 文件中定义的特定资源执行：

```
$ terraform apply -target=aws_kms_key.a
```

对于现有资源，可以选择将这些资源导入 Terraform 状态并在 `.tf` 文件中提供资源定义：

```
$ terraform import aws_kms_key.a 136c1dcb-42b0-4b9a-a569-152b9aba63e1
```

Terraform 项目有推荐的结构。`main.tf` 是定义资源的主要入口点。随着基础设施的复杂性增加，可以使用更多的 `.tf` 文件。`variables.tf` 包含我们希望在配置基础设施时动态设置的变量，`outputs.tf` 应包含我们在使用 Terraform 配置基础设施时希望提取的信息。

Terraform 项目已设置完成，我们将通过 Terraform 配置 VPC。

### 创建 VPC

我们的集群节点将驻留在 AWS VPC 中。我们将使用 `terraform-aws-modules` 项目中的 VPC 模块（https://registry.terraform.io/modules/terraform-aws-modules/vpc/aws/latest）：

```
module "eks_ksm_vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"
  name = "eks-ksm-vpc"
  cidr = "10.0.0.0/16"
  azs  = slice(var.availability_zones, 0, 3)
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.4.0/24", "10.0.5.0/24", "10.0.6.0/24"]
  enable_nat_gateway   = true
  single_nat_gateway   = true
  enable_dns_hostnames = true
...
}
```

通过此配置，我们将在三个可用区中扩展 VPC，从而实现高可用集群。

我们将在每个可用区创建一个私有子网以及公共子网。

Kubernetes 工作负载将托管在私有子网上，外部流量将通过公共网络访问 Kubernetes 工作负载。为了让 Kubernetes 集群能够访问互联网，我们将配置一个 NAT 网关。

由于我们已经配置了 VPC，现在可以继续配置 EKS 集群。

### 配置 EKS 集群

我们已经配置了 VPC，现在我们将继续配置 EKS 集群。我们将从 `terraform-aws-modules` 项目中选择 EKS 模块（[`registry.terraform.io/modules/terraform-aws-modules/eks/aws/latest`](https://registry.terraform.io/modules/terraform-aws-modules/eks/aws/latest)）：

```
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  ...
  cluster_name    = var.cluster_name
  vpc_id          = module.eks_ksm_vpc.vpc_id
  subnet_ids      = module.eks_ksm_vpc.private_subnets
  cluster_endpoint_public_access = true
  eks_managed_node_group_defaults = {
    ami_type = "AL2_x86_64"
  }
  create_cloudwatch_log_group = true
  eks_managed_node_groups = {
    one = {
      ...
      min_size     = 1
    }
  }
}
```

这将创建一个 EKS 集群。主节点由 AWS 管理。在创建集群时，我们指定节点跨越之前在 VPC 部分指定的私有子网。我们还将为 EKS 集群创建一个单独的日志组。

为了便于与 EKS 主节点交互，我们将配置集群端点为公开可访问。此选项可以进一步优化，并指定一组受限的 IP 地址，以便通过 kubectl 工具与集群进行交互。

一旦集群启动并运行，我们可以进行测试，甚至运行一些 kubectl 命令。

要对集群进行身份验证，我们将使用以下命令：

```
$ aws eks --region eu-west-1 update-kubeconfig --name eks-ksm-cluster
```

该命令会根据集群名称和选择的区域有所不同。完成后，我们可以使用 kubectl 指向最近配置的集群：

```
$ kubectl get nodes
NAME                                  STATUS   ROLES    AGE    VERSION
ip-10-0-2-231.eu-west-1.compute.internal   Ready    <none>   6m5s   v1.24.15-eks-a5565ad
```

既然我们已经配置了 EKS 集群，现在可以继续与 AWS Secrets Manager 配合工作。

### 在 Secrets Manager 中创建机密

让我们使用 AWS Secrets Manager 创建一个机密：

```
resource "aws_secretsmanager_secret" "ksm_service_token" {
  name = "service-token"
  replica {
    region = "eu-central-1"
  }
  recovery_window_in_days {
    ...
  }
}
```

在 `replica` 块中，我们指定了复制机密的区域。这将为我们提供灾难恢复能力。同时，`recovery_window_in_days` 块定义了在设置删除操作后，我们能在多少天内恢复该机密。

现在，让我们为指定的机密添加一个版本：

```
resource "aws_secretsmanager_secret_version" "ksm_service_token_first_version" {
  secret_id     = aws_secretsmanager_secret.ksm_service_token.id
  secret_string = "a-service-token"
}
```

这是我们与 AWS Secrets Manager 的第一次交互。我们已创建一个机密，并包含一个字符串的机密版本。

我们还将创建一个具有所需 IAM 绑定的角色：

```
resource "aws_iam_role" "eks_secret_reader_role" {
  name = "eks-secret-reader"
  assume_role_policy = jsonencode({
    Version = "2012-10-17",
    Statement = [
      {
        Effect = "Allow",
        "Principal": {
          "Federated": "arn:aws:iam::${data.aws_caller_identity.current.account_id}:oidc-provider/${module.ksm_eks.oidc_provider}"
        }
        "Action": "sts:AssumeRoleWithWebIdentity",
        "Condition": {
          "StringEquals": {
            "${module.ksm_eks.oidc_provider}:aud": "sts.amazonaws.com",
            "${module.ksm_eks.oidc_provider}:sub": "system:serviceaccount:default:service-token-reader"
        }
      }
      }
    ]
  })
}
resource "aws_iam_role_policy_attachment" "esrrs" {
  policy_arn = aws_iam_policy.ksm_service_token_reader.arn
  role       = aws_iam_role.eks_secret_reader_role.name
}
```

注意这个角色，它与我们惯用的角色非常不同。这个角色是专为 EKS 创建的，实际上它是映射到 Kubernetes 集群中角色的工作负载身份。

该角色在可执行的操作上有所限制。我们无法使用该角色对其他 AWS 资源执行操作。

或者，我们也可以使用 AWS CLI 来配置 Secrets：

```
$ aws secretsmanager create-secret --name service-token --secret-string a-service-token  --add-replica-regions Region=eu-central-1
```

这将创建一个机密及其版本。

我们还可以使用 AWS CLI 创建角色和策略：

```
...
$ aws iam create-role --role-name eks-secret-reader --assume-role-policy-document file://eks-reader-trust-policy.json
...
$ aws iam create-policy --policy-name get-service-token --policy-document  file://policy.json
...
$ aws iam attach-role-policy --role-name eks-secret-reader --policy-arn arn:aws:iam::$account_id:policy/get-service-token
```

我们有一个可以附加到 Kubernetes 并从 Secrets Manager 中获取机密的角色。下一步是安装 CSI 插件。

### EKS 上的 AWS Secrets Manager CSI 提供程序

现在是时候在 EKS 上安装 AWS Secrets Manager CSI 提供程序了。首先要做的是将 Secrets Store CSI 驱动插件添加到 EKS 安装中。

让我们先在 EKS 上安装驱动程序，因为 `secrets-store-csi` 驱动程序不能作为附加组件使用：

```
$ helm repo add secrets-store-csi-driver https://kubernetes-sigs.github.io/secrets-store-csi-driver/charts
$ helm install -n kube-system csi-secrets-store secrets-store-csi-driver/secrets-store-csi-driver
$ helm repo add aws-secrets-manager https://aws.github.io/secrets-store-csi-driver-provider-aws $ helm install -n kube-system secrets-provider-aws aws-secrets-manager/secrets-store-csi-driver-provider-aws
```

以下命令将安装 `secret-store-csi-driver` DaemonSet。我们可以使用以下命令验证 `secret-store-csi-driver` 是否已正确安装：

```
$ kubectl get daemonset -n kube-system
NAME         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR              AGE   <none>                     47m
...
secrets-provider-aws-secrets-store-csi-driver-provider-aws   1         1         1       1            1           kubernetes.io/os=linux     44s
```

我们可以通过 Kubernetes 从机密存储中获取凭证。

### 从 Kubernetes 获取机密

首先，我们需要通过机密提供者类来映射机密。在对象中，我们指定要获取的机密名称：

```
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: service-token
spec:
  provider: aws
  parameters:
    objects: |
        - objectName: "arn:aws:secretsmanager:eu-west-1:1111111:secret:service_token-IJ2VLg"
```

我们将 Kubernetes 密钥映射到 AWS Secrets Manager 提供的机密。

现在我们需要一个具有获取此机密权限的 Kubernetes 服务帐户：

```
apiVersion: v1
kind: ServiceAccount
metadata:
  annotations:
    eks.amazonaws.com/role-arn: "arn:aws:iam::11111:role/eks-secret-reader"
  name: service-token-reader
  namespace: default
```

在 `eks.amazonaws.com/role-arn` 部分，我们指定具有权限的 AWS 角色。

正如我们所见，服务帐户已用 AWS 角色进行了注解，这个角色是我们之前创建的 AWS 角色。

只要该服务账户附加到 Pod，它就会通过 Kubernetes OIDC 提供程序假设这个角色身份。有关 OIDC 和 Kubernetes 服务账户的更多信息，请参阅官方文档（[`docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html`](https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html)）。

当 Pod 创建时，它会使用该服务账户并挂载该秘密：

```
kind: Pod
apiVersion: v1
metadata:
  name: nginx
spec:
  serviceAccountName: service-token-reader
  containers:
  - image: nginx
    name: nginx
    volumeMounts:
    - name: secret-from-asm
      mountPath: "/mnt/secrets-store"
      readOnly: true
  volumes:
    - name: secret-from-asm
      csi:
        driver: secrets-store.csi.k8s.io
        readOnly: true
        volumeAttributes:
          secretProviderClass: "service-token"
```

我们可以通过打印秘密来测试这个：

```
$ kubectl exec -it nginx cat /mnt/secrets-store/arn:aws:secretsmanager:eu-west-1:274402012893:secret:service_token-IJ2VLg
```

总结一下，我们创建了一个带有服务账户的 Pod，该账户映射到 AWS 角色。这个服务账户有权限与 AWS Secrets Manager 交互。因此，我们可以看到之前挂载的秘密。在接下来的章节中，我们将重点介绍如何通过审计跟踪 AWS Secrets Manager 中发生的操作。

# 审计

我们已经成功创建了 Secrets，并通过利用 AWS IAM 确保我们安全地接收它们。由于我们已经解决了安全访问问题，并且拥有细粒度权限，因此我们还希望记录对我们的 Secrets 管理实例的使用和访问。

有两种方式可以评估 Kubernetes 中发生的秘密访问：

+   Kubernetes Secrets 访问日志

+   AWS Secrets Manager 日志

当发生审计时，日志消息会打印在程序输出中。这个功能与 AWS 提供的日志解决方案 CloudWatch 和审计解决方案 CloudTrail 集成。

## Kubernetes 日志在 CloudWatch 上

通过使用 CloudWatch，我们可以浏览 Kubernetes 日志。假设我们创建了一个名为 `audit-test` 的秘密。我们可以在 Logs Insights 中搜索任何操作（[`console.aws.amazon.com/cloudwatch/home?logsV2%3Alogs-insights=#logsV2:logs-insights`](https://console.aws.amazon.com/cloudwatch/home?logsV2%3Alogs-insights=#logsV2:logs-insights)）。我们可以使用以下日志：

```
fields @timestamp, @message, @logStream, @log
| filter @message like 'audit-test'
| sort @timestamp desc
| limit 20
```

结果条目将显示实际的结果：

| **字段** | **值** |
| --- | --- |
| `@``消息` | `{"kind":"Event",...,"verb":"get","user":{"username":"kubernetes-admin", "uid":"aws-iam-authenticator:274402012893:AIDAILH3OPGRUQEOHAR3O", "groups":["system:masters","system:authenticated"], "extra":{"accessKeyId": ["AKIAI5INYQBL233Y7J6Q"],"arn": ["arn:aws:iam::274402012893: user/ gkatzioura"],"canonicalArn": ["arn:aws:iam:: 274402012893: user/gkatzioura"], "principalId": ["AIDAILH3OPGRUQEOHAR3O"],"sessionName": [""]}},"sourceIPs":["90.221.185.67"],"userAgent": "kubectl/v1.25.4 (darwin/arm64) kubernetes/872a965","objectRef":{"resource": "secrets","namespace":"default", "``name":"audit-test", "apiVersion":"v1"},"responseStatus":{"metadata":...}` |
| `@``时间戳` | `1692687754591` |

这个结果看起来很熟悉。它是我们之前在 *第二章* 中看到的日志，*走进 Kubernetes Secrets 管理概念*，当时我们正在审计 Kubernetes Secrets。

此外，CloudWatch 允许我们创建警报。如果存在意外的日志条目，表明访问或尝试访问某个密钥，我们可以配置 CloudWatch 创建警报，并通过我们选择的通信渠道通知我们，例如通过电子邮件。

## AWS Secrets Manager 在 AWS CloudTrail 上的日志记录

AWS CloudTrail 是一个专门用于审计日志的服务。通过 CloudTrail，我们可以识别 Kubernetes 密钥的检索操作。

例如，我们可以使用此 URL 查看我们 AWS 账户中最近检索的密钥：[`console.aws.amazon.com/cloudtrail/home?#/events?EventName=GetSecretValue`](https://console.aws.amazon.com/cloudtrail/home?#/events?EventName=GetSecretValue)。

如果我们访问该 URL，AWS Secrets Manager 中的检索信息将会显示：

```
{
    "userIdentity": {
        "type": "AssumedRole",
        "principalId": "id:secrets-store-csi-driver-provider-aws",
        "arn": "arn:aws:sts::1111111:assumed-role/eks-secret-reader/secrets-store-csi-driver-provider-aws",
    ...
        }
    },
    "eventSource": "secretsmanager.amazonaws.com",
    "eventName": "GetSecretValue",
    ...
    "requestParameters": {
        "secretId": "arn:aws:secretsmanager:eu-west-1:11111111:secret:service_token-93z7he"
    },
    "responseElements": null,
    ...
}
```

用户身份是我们之前创建的 AWS 角色。事件名称是 `GetSecretValue` 事件。`r``equestParameters` 是执行操作的资源。

# AWS Secrets 加密的 KMS

Kubernetes 允许我们在将密钥存储到 etcd 时指定特定的加密提供者。AWS KMS 可以作为加密提供者来加密托管在 etcd 上的密钥。

## 配置 KMS

我们将使用 Terraform 配置一个 KMS 密钥：

```
resource "aws_kms_key" "ksm_kms_key" {
  description             = "ksm_kms_key"
  deletion_window_in_days = 30
  enable_key_rotation     = true
}
```

该密钥现在可以单独使用，也可以与其他 AWS 组件一起使用。

## 在 EKS 中使用 KMS

创建了 EKS 集群后，我们可以指定将解密密钥的凭证：

```
module "ksm_eks" {
    ...
  create_kms_key = false
  cluster_encryption_config = {
    resources = ["secrets"]
    provider_key_arn = aws_kms_key.ksm_kms_key.arn
  }
  ...
}
```

如果我们尝试从 Kubernetes 获取凭证，密钥将使用我们指定的 KMS 进行解密。我们可以使用 AWS CloudTrail 来识别使用情况。我们可以通过以下链接访问 CloudTrail 事件：[`console.aws.amazon.com/cloudtrail/home?#/events?EventName=Decrypt`](https://console.aws.amazon.com/cloudtrail/home?#/events?EventName=Decrypt)。

如果我们访问该页面，解密操作将显示在屏幕上。同时，它将显示进行解密操作的用户，在我们的案例中是与 Kubernetes 集群关联的角色。

信息将以 JSON 格式呈现：

```
{
    "eventSource": "kms.amazonaws.com",
    "eventName": "Decrypt",
    "awsRegion": "eu-west-1",
    "sourceIPAddress": "secretsmanager.amazonaws.com",
    "userAgent": "secretsmanager.amazonaws.com",
    "requestParameters": {
        "encryptionContext": {
            "SecretARN": "arn:aws:secretsmanager:eu-west-1:1212222223:secret:service_token-93z7he",
            "SecretVersionId": "278A157C-EA85-4211-9854-D329D3C9089F"
        },
        "encryptionAlgorithm": "SYMMETRIC_DEFAULT"
    },
    "resources": [
        {
            "ARN": "arn:aws:kms:eu-west-1:111111111:key/aaaaaaa-aaaa-458f-b8d1-aefa58b68d8a"
        }
    ],
    "eventType": "AwsApiCall",
}
```

我们通过使用 AWS KMS，将我们的密钥以加密形式存储在 `etcd` 上。存储在 etcd 上的密钥将被加密，我们应该能够监控任何加密/解密操作。

# 总结

在本章中，我们创建了一个跨越三个可用区的 VPC 网络。在该网络之上，我们配置了一个 EKS 集群。我们成功地将 EKS 集群与 AWS Secrets Manager 集成。这是通过在 AWS 上实现精细粒度的角色映射到我们的 Kubernetes 工作负载来安全完成的。接着，我们通过检查 CloudWatch 和 CloudTrail 日志，确定了与 AWS Secrets Manager 上存储的密钥相关的操作。最后，我们通过使用托管的 KMS 对存储在 EKS 集群上的密钥进行了加密。在下一章中，我们将重点介绍另一个流行的云服务提供商及其密钥处理能力：微软 Azure。
