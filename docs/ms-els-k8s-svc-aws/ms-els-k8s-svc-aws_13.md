

# 第十三章：使用 IAM 授予应用程序访问权限。

AWS 提供超过 200 种服务，从 SQL/NoSQL 数据库到机器学习和量子计算。很可能在某个时候，您希望从部署在 EKS 上的应用程序中使用这些服务之一。

本章将探讨如何向 Pods 授予 IAM 权限，如何在应用程序中使用相关凭证连接到 AWS 服务，并如何解决整个过程中的问题。具体来说，我们将涵盖以下内容：

+   理解 **IAM Service Account 角色**（**IRSA**）是什么及其解决了什么问题。

+   在应用程序中使用 IRSA。

+   如何在 EKS 上解决 IAM 问题。

# 技术要求。

读者应熟悉 YAML、AWS IAM 和 EKS 架构。在开始本章之前，请确保以下内容：

+   您可以连接到您的 EKS 集群 API 终端。

+   AWS CLI、Docker 和 `kubectl` 二进制文件已安装在您的工作站上。

+   您对 AWS IAM 有基本的了解。

# 理解 IRSA。

首先，让我们看看标准 EC2 实例的 IAM 角色分配工作原理。在 AWS IAM 中，角色用于分配权限（使用一个或多个策略）。可以使用实例配置文件将角色分配给 EC2 实例，该配置文件只是附加到特定 EC2 实例的 IAM 角色的容器。

![图 13.1 – EC2 角色分配](img/B18129_13_01.jpg)

图 13.1 – EC2 角色分配。

当创建一个 EC2 实例并分配角色时，AWS 平台将自动创建一个实例配置文件。当该实例启动时，它将向 `169.254.169.254` 发送网络调用，并查询该实例分配了哪个（如果有）实例配置文件（或角色）。如果已分配一个角色，则可以检索访问凭证，下面是一个示例。这些凭证包括访问密钥和秘密密钥，用于所有 AWS API 调用，并标识角色及其授予的权限：

```
{ "Code" : "Success",
  "LastUpdated" : "2022-04-26T16:39:16Z",
  "Type" : "AWS-HMAC",
  "AccessKeyId" : "ASIAIOSFODNN7EXAMPLE",
  "SecretAccessKey" : "bPxRfiCYEXAMPLEKEY",
  "Token" : "token",
  "Expiration" : "2022-05-17T15:09:54Z"}
```

如果登录到 EC2 实例，可以使用下面显示的 `aws sts` 命令查看附加到运行实例的实例配置文件：

```
$ aws sts get-caller-identity
{
 "UserId": "hdghd78898:i-014",
 "Account": "11223344,
 "Arn": "arn:aws:sts::11223344:assumed-role/<IPROFILE>/i-014"
}
```

AWS 引入了 IMDSv2，以在容器平台上提供一些安全控制，让我们来审视主要变更。

## 引入 IMDSv2。

正如我们所讨论的，实例配置文件信息是通过 EC2 IMDS 检索的，该服务在每个 VPC 中的地址为 `169.254.169.254`。此服务对于 EC2 实例的操作至关重要，不应阻止工作节点使用它，但应限制 Pods 使用。原始 IMDS 版本，即版本 1，不允许对谁可以使用它施加任何限制。

IMDSv2 是对 IMDSv1 的增强，使用面向会话的请求为服务增加安全控制。IMDSv2 返回一个令牌，用于请求元数据和凭证。由于令牌从不存储在 IMDSv2 中，因此当使用该令牌的进程或应用程序结束时，令牌就会丢失。元数据服务使用 TTL 跳数在 IPv4/IPv6 数据包中允许请求。默认情况下，它设置为 1，意味着请求只能直接来自 EC2。运行在主机上的任何 Pod 都会将跳数增加到 2（因为它们在主机内使用桥接网络），这会阻止它们直接使用 IMDSv2 服务并获取主机实例配置凭证。

为强制使用 IMDSv2 并限制跳数，你可以在所有工作节点上运行以下命令，或者将配置添加到用于工作节点的启动模板的**基础设施即代码**（**IaC**）定义中：

```
$ aws ec2 modify-instance-metadata-options --instance-id i-1122233 --http-tokens required --http-put-response-hop-limit 1
{
    "InstanceId": "i-112233",
    "InstanceMetadataOptions": {
        "State": "pending",
        "HttpEndpoint": "enabled",
        "HttpTokens": "required",
        "HttpPutResponseHopLimit": 1
    }
}
```

然而，这只有在 Pod 使用 IMDSv2 时才有效。在大多数情况下，Pod 使用 AWS SDK 来调用 AWS API，这默认使用 IMDSv2 来获取凭证。这些凭证将应用跳数限制。然而，如果 Pod 使用 IMDSv1，它仍然可以获取主机凭证。在这种情况下，最佳实践是仅给工作节点最小的权限（例如，仅允许拉取 ECR 容器），或者使用 Calico 网络策略或`iptables`规则限制对 IMDSv1 的访问。

现在，我们已经了解了实例和 Pod 如何使用 IMDS 来获取分配给主机的凭证（以及如何限制它们这样做）。接下来，让我们看看如何使用 IRSA 将特定的角色直接分配给 Pods。

## IRSA 的工作原理

IRSA 允许你通过将 Kubernetes 的`AssumeRoleWithWebIdentity` API 调用与 AWS **安全令牌服务**（**STS**）关联，从而为特定的 Pod 分配特定权限，STS 将把 Kubernetes 生成的凭证交换为 AWS IAM 生成的凭证。它通过将特定 EKS 集群的 OIDC 提供程序作为主体，并假设在 SA 注释中定义的角色来实现这一点（步骤**3**）。

![图 13.2 - IRSA 的工作原理](img/B18129_13_02.jpg)

图 13.2 - IRSA 的工作原理

现在我们大致了解了 IRSA 的工作原理，接下来让我们看看需要配置什么，才能让 Pod 使用 IRSA。

# 在应用程序中使用 IRSA

现在你已经了解了 IRSA 的基本概念，接下来让我们看看如何在应用程序中配置和使用它。我们将查看如何手动部署 Pod 并配置它以使用 IRSA，然后我们将看看如何通过`eksctl`简化整个过程。

## 如何部署 Pod 并使用 IRSA 凭证

第一步是确保已为集群配置了 OIDC 提供程序。如果你使用了`eksctl`，则此配置已经完成：

```
$ aws eks describe-cluster --name myipv4cluster --query "cluster.identity.oidc.issuer" --output text
https://oidc.eks.eu-central-1.amazonaws.com/id/763683678
```

如果你还没有启用它，可以使用以下`eksctl`命令：

```
$ eksctl utils associate-iam-oidc-provider --cluster cluster_name –approve
```

既然我们已经为集群创建了身份，并且可以使用它，在 IAM 中，我们可以创建相关的策略和角色。假设我们希望将 Pod 的访问权限授予账户中的所有 S3 存储桶和对象。那么，我们将使用以下策略来提供 S3 访问：

```
{ "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetAccountPublicAccessBlock",
                "s3:GetBucketAcl",
                "s3:GetBucketLocation",
                "s3:GetBucketPolicyStatus",
                "s3:GetBucketPublicAccessBlock",
                "s3:ListAccessPoints",
                "s3:ListAllMyBuckets"
            ],
            "Resource": "*"
        }
    ]
}
```

然后，我们将使用以下命令在 AWS 账户中创建策略：

```
$ aws iam create-policy --policy-name bespoke-pod-policy --policy-document file://s3-policy.json
{
    "Policy": {
        "PolicyName": "bespoke-pod-policy",
        "PermissionsBoundaryUsageCount": 0,
        "CreateDate": "2022-11-09T15:03:58Z",
        "AttachmentCount": 0,
        "IsAttachable": true,
        "PolicyId": "ANPARDV7UN626ZCPMFH4X",
        "DefaultVersionId": "v1",
        "Path": "/",
        "Arn": "arn:aws:iam::112233444:policy/bespoke-pod-policy",
        "UpdateDate": "2022-11-09T15:03:58Z"
    }
}
```

现在，我们已经拥有了创建角色所需的策略，并允许 EKS 集群 OIDC 提供者和 Kubernetes SA 组合来假设该角色。以下命令将帮助您完成这一操作。首先，我们需要为 AWS 账户、EKS OIDC 提供者、Kubernetes 命名空间和 SA 设置一些环境变量：

```
$ export account_id=$(aws sts get-caller-identity --query "Account" --output text)
$ export oidc_provider=$(aws eks describe-cluster --name myipv4cluster --region eu-central-1 --query "cluster.identity.oidc.issuer" --output text | sed -e "s/^https:\/\///")
$ export namespace=default
$ export service_account=s3-access
```

现在，我们可以使用以下命令创建 Pod 将要假设的角色的信任关系：

```
cat >trust-relationship.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::$account_id:oidc-provider/$oidc_provider"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "$oidc_provider:aud": "sts.amazonaws.com",
          "$oidc_provider:sub": "system:serviceaccount:$namespace:$service_account"
        }
      }
    }
  ]
}
EOF
```

这一切只是定义 EKS OIDC 提供者、Kubernetes 命名空间和 SA 的映射关系，因此我们需要使用以下命令创建具有此信任关系的角色，并附加之前创建的策略，以便使用 S3 访问：

```
$ aws iam create-role --role-name s3-access-default --assume-role-policy-document file://trust-relationship.json --description "s3 access role for pod SA s3-access/default"
{
    "Role": {
        ………
        "Arn": "arn:aws:iam::11223344:role/s3-access-default"
    }
}
$ aws iam attach-role-policy --role-name s3-access-default --policy-arn=arn:aws:iam::$account_id:policy/bespoke-pod-policy
```

现在我们已经在 AWS IAM 中设置好了所需的一切，接下来只需要配置一个 Pod，使其使用此处定义的命名空间中的 SA。我们从配置`default`命名空间中的 SA 开始：

```
$ cat >my-service-account.yaml <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: s3-access
  namespace: default
EOF
$ kubectl apply -f my-service-account.yaml
$ kubectl annotate serviceaccount -n $namespace $service_account eks.amazonaws.com/role-arn=arn:aws:iam::$account_id:role/s3-access-default
serviceaccount/s3-access annotated
$ kubectl describe sa $service_account
Name:                s3-access
Namespace:           default
Labels:              <none>
Annotations:         eks.amazonaws.com/role-arn: arn:aws:iam::11223344:role/s3-access-default
Image pull secrets:  <none>
Mountable secrets:   s3-access-token-9z6
Tokens:              s3-access-token-9z6
Events:              <none>
```

我们现在可以使用以下命令运行带有 SA 的 Pod。此命令将使用`aws-cli`镜像并运行`aws s3 ls`命令，应该能够列出存储桶，因为分配的 SA 具有通过角色暴露的必要权限：

```
$ kubectl run –rm -ti cli  --image=amazon/aws-cli –overrides="{ 'spec': { 'serviceAccount': 's3-access' }  }" s3 ls
2022-09-29 09:26:05 ingress-123-bb
2022-03-29 17:49:50 servicecatalog456643
……..
```

如果将`serviceAccount`值更改为`default`，您会看到命令失败，因为默认的 SA 没有注解，因此没有映射到有效的 IAM 角色。现在，让我们来看一下如何使用 IaC 简化这个过程。

## 如何通过编程方式创建 IRSA 角色

我们已经使用了上一章中的`eksctl create iamserviceaccount`命令，允许托管存储控制器的 Pod 与 AWS 存储 API 进行通信。

如果我们查看通用命令，可以看到我们已将特定命名空间中的 Kubernetes SA 与特定 IAM 策略关联起来。该策略定义了可以执行的 API 操作，而将 SA 与 Pod 关联起来，使得该 Pod 可以执行该操作：

```
$ eksctl create iamserviceaccount --cluster=<clusterName> --name=<serviceAccountName> --namespace=<serviceAccountNamespace> --attach-policy-arn=<policyARN>
```

因此，与上一部分相比，我们唯一需要预先准备的是具有相关权限的策略，这样我们就可以提供`policyARN`。`eksctl`工具将执行以下操作：

1.  确定 EKS 集群的 OIDC 提供者。

1.  使用派生的 OIDC 详细信息以及提供的 Kubernetes SA 名称和命名空间创建带有信任策略的角色。

1.  将预先创建的策略附加到 IAM 角色。

1.  在正确的命名空间中创建具有正确注解的 Kubernetes SA。

现在，您可以在命名空间中运行之前使用的`kubectl run`命令，然后使用在`eksctl`命令中指定的 SA，所有操作应该都能正常运行。接下来让我们看看如何在遇到问题时排查 IRSA。

# 如何排查 EKS 中的 IAM 问题

首先需要做的是确定这是否是 IAM 权限问题。如果我们查看以下示例中的错误信息，可以看到 AWS API 操作的`AccessDenied`错误信息——在这个例子中，是`ListBuckets`操作。这是一个明显的指示，表明这是一个 IAM 错误：

```
$ kubectl run -ti cli  --image=amazon/aws-cli --overrides='{ "spec": { "serviceAccount": "default" }  }' s3 ls
If you don't see a command prompt, try pressing enter.
An error occurred (AccessDenied) when calling the ListBuckets operation: Access Denied
```

第一步是确定使用的是哪个 SA，然后从那里向后追溯。在这个例子中很清楚，因为我们有`run`命令。但是，假设我们没有它，我们可以使用下一个命令来找出它：

```
$ kubectl get po cli -o yaml | grep serviceAccountName
  serviceAccountName: default
```

然后，我们可以运行以下命令来确保注释已经设置，并识别将要假定的角色：

```
$ kubectl describe sa default
Name:                default
Namespace:           default
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   default-token-wjpnc
Tokens:              default-token-wjpnc
Events:              <none>
```

在这种情况下，没有注释，所以很明显这个 SA 没有权限。如果有角色被分配，我们可以使用以下命令来确定是策略还是权限问题：

```
$ kubectl describe sa default
…..
Annotations:         eks.amazonaws.com/role-arn: arn:aws:iam::11223344:role/s3-access-default
…..
$ aws iam list-attached-role-policies --role-name s3-access-default
{
    "AttachedPolicies": [
        {
            "PolicyName": "bespoke-pod-policy",
            "PolicyArn": "arn:aws:iam::11223344:policy/bespoke-pod-policy"
        }
    ]
}
```

一旦我们可以看到附加的策略，在这个例子中只有一个，我们可以遍历它，但首先，你必须使用`get-policy`命令获取策略的版本：

```
$ aws iam get-policy --policy-arn arn:aws:iam::076637564853:policy/bespoke-pod-policy{
    "Policy": {
        "PolicyName": "bespoke-pod-policy",
        "Tags": [],
        "PermissionsBoundaryUsageCount": 0,
        "CreateDate": "2022-11-09T15:03:58Z",
        "AttachmentCount": 1,
        "IsAttachable": true,
        "PolicyId": "ANPARDV7UN626ZCPMFH4X",
        "DefaultVersionId": "v1",
        "Path": "/",
        "Arn": "arn:aws:iam::076637564853:policy/bespoke-pod-policy",
        "UpdateDate": "2022-11-09T15:03:58Z"
    }
}
```

现在我们可以使用`get-policy-version`命令提取权限。在以下示例中，缺少`s3:ListAllMyBuckets`操作，这就是导致问题的原因：

```
$ aws iam get-policy-version    --policy-arn arn:aws:iam::076637564853:policy/bespoke-pod-policy   --version-id v1
{
    "PolicyVersion": {
        "CreateDate": "2022-11-09T15:03:58Z",
        "VersionId": "v1",
        "Document": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Action": [
                        "s3:GetAccountPublicAccessBlock",
                        "s3:GetBucketAcl",
                        "s3:GetBucketLocation",
                        "s3:GetBucketPolicyStatus",
                        "s3:GetBucketPublicAccessBlock",
                        "s3:ListAccessPoints"
                    ],
                    "Resource": "*",
                    "Effect": "Allow"
                }
            ]
        },
        "IsDefaultVersion": true
    }
}
```

你还可以考虑其他几个方面：

+   你可以使用以下命令来检查分配的角色是否信任正确的 EKS OIDC 提供者：

    ```
    $ aws iam get-role --role-name
    ```

+   IRSA 通过一个变更 Webhook 来实现 Pod 身份验证。你可以使用以下命令验证这一点是否已经部署：

    ```
    $ kubectl get mutatingwebhookconfiguration pod-identity-webhook  -o yaml
    ```

其他一些错误在*进一步阅读*部分有介绍。在这一部分，我们探讨了 IRSA 是什么，它是如何工作的以及如何配置的，还讲解了如何进行一些基本的 IRSA 故障排除。接下来，我们将回顾本章的关键学习点。

# 总结

在本章中，我们探讨了如何使用实例配置文件和 IMDS 将 AWS API 权限分配给 EC2 实例（工作节点）和 Pods。我们还注意到，默认情况下，EKS Pods 继承它们运行的工作节点上分配的权限，这可能并不是一个好事，因为我们没有遵循*最小权限*模型，因为许多 Pods 可能根本不需要任何 AWS API 访问权限。

我们讨论了如何使用 IMDSv2 来减少工作节点权限的使用，并应与 IRSA 一起使用，以限制工作节点权限的继承。接着，我们演示了如何从命令行配置和使用 IRSA，以及像`eksctl`这样的 IaC 工具如何显著简化这一过程。最后，我们讨论了如何进行 AWS IAM 权限问题的基本故障排除，从 Kubernetes SA 向后追溯。

在下一章，我们将讨论如何使用 AWS 负载均衡器来使我们的 Kubernetes 服务更加稳健和可扩展。

# 进一步阅读

+   理解 EC2 元数据服务：[`docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html`](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html)

)

+   EKS 更新以支持 IMDSv2： [`aws.amazon.com/about-aws/whats-new/2020/08/amazon-eks-supports-ec2-instance-metadata-service-v2/`](https://aws.amazon.com/about-aws/whats-new/2020/08/amazon-eks-supports-ec2-instance-metadata-service-v2/)

)

+   解决 IRSA 错误： [`aws.amazon.com/premiumsupport/knowledge-center/eks-troubleshoot-IRSA-errors/`](https://aws.amazon.com/premiumsupport/knowledge-center/eks-troubleshoot-IRSA-errors/)
