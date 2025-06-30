# 19

# 构建开发者门户

近年来 DevOps 和自动化领域最受欢迎的概念之一就是提供**内部开发者门户**（**IDP**）。该门户的目的是为开发者和基础设施团队提供一个单一的服务点，使他们能够访问架构服务，而无需向 IT 部门的“某个人”发送电子邮件。这通常是基于云服务的承诺，尽管实现这一目标需要大量的定制开发。它还为创建开发可管理架构所需的护栏提供了基础。

本章将结合我们在*第十八章*《提供多租户平台》中讨论的理论，以及我们在本书中学到的大部分概念和技术，来构建一个 IDP。完成本章后，你将对如何为你的基础设施构建 IDP 有一个概念，并理解我们通过本书构建并集成到 Kubernetes 中的各种技术如何结合在一起。

本章将涵盖以下主题：

+   技术要求

+   部署我们的 IDP

+   租户入驻

+   部署应用程序

+   扩展我们的平台

最后，在深入本章之前，我们想说一声*谢谢！* 本书对我们来说是一段精彩的旅程。自从我们编写第二版以来，看到行业发生了如此大的变化，我们也取得了很多进步，真是令人惊叹。感谢你加入我们，一起探索如何构建企业级 Kubernetes，以及各种技术如何与企业世界的需求相结合。

# 技术要求

本章比之前的章节有更多的技术要求。你需要三个 Kubernetes 集群，具有以下要求：

+   **计算**：16 GB 内存和 8 核。你将运行 GitLab、Vault、OpenUnison、Argo CD 等。这将需要一些强劲的计算能力。

+   **访问**：确保你可以更新并访问本地节点。你需要这样做才能将我们的 CA 证书添加到节点中，以便在拉取容器镜像时能够信任它。

+   **网络**：你不需要公共 IP 地址，但你需要能够从工作站访问所有三个集群。如果为每个集群使用负载均衡器，会使实现过程更为简便，但这并不是强制要求。

+   **Pulumi 和 Python 3**：我们将使用 Pulumi 部署我们的平台，运行在 Python 3 上。你使用的工作站需要能够运行这些工具。我们在 macOS 上构建并编写了本章，使用 Homebrew（[`brew.sh/`](https://brew.sh/)）安装 Python。

在开始构建之前，我们将花一些时间讨论本章的技术要求以及它们为什么是必需的，并将这些要求与常见的企业场景联系起来。首先，让我们来看一下我们的计算要求。

## 满足计算要求

在本书的其余部分，我们的目标是将所有实验运行在一台虚拟机上。我们这样做有几个原因：

+   **成本**：我们知道在学习技术的过程中成本会迅速上升，所以我们希望确保不会让你为了这个目的而额外花费。

+   **简易性**：Kubernetes 已经足够复杂了，更不用说在企业环境中如何设置计算和网络！我们也不想担心存储问题，因为它带来了许多复杂性。

+   **实现和支持的简易性**：我们希望确保能够帮助你完成实验，所以限制部署方式使我们更加轻松地提供支持。

总的来说，本章内容有所不同。你可以使用运行 KinD 的三台虚拟机，但那样可能会引发更多问题，远不值得。我们将讨论两种主要选项：使用云和建立家庭实验室。

### 使用云托管的 Kubernetes

使用托管 Kubernetes 很受欢迎，尤其是在没有要部署的内容时。每个大型云服务商都有自己的 Kubernetes 托管服务，大多数小型云服务商也是如此。如果你愿意花一些钱，专注于 Kubernetes 而不是支撑它的基础设施，这些服务非常适合你。不过，要确保在设置集群时，你能够通过 SSH 或其他方式直接访问你的工作节点。

许多基于云的托管集群默认情况下不允许访问你的节点，从安全角度看，这是个好事！你无法攻击你无法访问的东西！但缺点是，你也无法进行自定义配置。我们将在下一部分讨论这一点，但大多数企业即使使用云托管 Kubernetes，也要求对节点进行某种程度的自定义。

此外，确保你能承担费用。三集群的配置不太可能在你获得的免费积分内长时间保持。你需要确保自己能支付这笔费用。也就是说，如果你不愿意将钱投入云计算，那么也许家庭实验室是你最好的选择。

### 建立家庭实验室

云计算可能变得非常昂贵，而且那笔钱几乎是浪费掉的。你根本没有什么可展示的成果！另一种选择是建立一个家庭实验室来运行你的集群。根据写这本书时的情况，现在在自己的家或公寓里运行企业级基础设施比以往任何时候都要容易。它不需要大额投资，而且通常比仅使用云管理集群一个月或两个月还要便宜得多。

你可以非常简单地从 eBay 等拍卖网站上以不到$500 的价格，购买一台翻新的或自建的服务器。拿到服务器后，安装 Linux 和虚拟化管理程序，你就可以开始了。当然，这需要花费更多时间来处理底层基础设施，远超我们目前的操作，但从个人和经济层面来看，这将是非常有价值的。如果你是带着进入企业级 Kubernetes 领域的目标在读这本书，了解你的基础设施如何运作将使你在其他候选人中脱颖而出。

如果你有条件为你的家庭实验室投入更多资金，有一些项目能让你更容易地搭建实验室。以下是两个这样的项目：

+   **Metal as a Service (MaaS)**：这个来自 Canonical 的项目（[`maas.io/`](https://maas.io/)）通过提供资源管理、DNS、网络启动等功能，使得快速将基础设施接入实验室变得更加容易。Canonical 是创建 Ubuntu Linux 发行版的公司。虽然它最初是为了快速接入硬件而开发的项目，但它也支持通过`virsh`协议管理 KVM，这样就可以通过 SSH 管理虚拟机。现在，我正在使用这个项目运行我的家庭实验室，设备是在几台运行 Ubuntu 的家庭组装的 PC 上。

+   **Container Craft Kargo**：这是一个相对较新的平台（[`github.com/ContainerCraft/Kargo`](https://github.com/ContainerCraft/Kargo)），它将多个“企业级”系统结合起来，构建一个基于 Kubernetes 的家庭实验室。这个项目的亮点是它从 Talos 开始，Talos 是一个操作系统与 Kubernetes 发行版的结合体，并使用 KubeVirt 利用 Kubernetes API 来部署虚拟机。这是一个很棒的项目，我已经开始使用并投入其中，也将把我的家庭实验室迁移到这个平台。

在研究了如何构建家庭实验室以及在哪里部署你的 IDP 之后，我们接下来将探讨为什么你需要直接访问你的节点。

## 自定义节点

在使用如 Amazon 或 Azure 这样的托管 Kubernetes 时，节点是由服务提供商提供的。你也可以选择禁用外部访问。从安全的角度来看，这非常好，因为你不需要保护那些无法访问的内容！

正如我们之前所说，安全性与合规性是有区别的。虽然一个完全托管的节点可能更安全，但你的合规性规则可能要求你作为集群管理员，必须有流程来管理节点访问。简单地移除所有访问权限可能无法解决这一合规问题。

然而，这种方法有一个功能上的缺点；你现在无法自定义节点。这个缺点可以通过几种方式表现出来：

+   **自定义证书**：我们在本书中多次提到，企业通常会维护内部的**证书颁发机构**（**CAs**）。我们通过使用自己的内部 CA 来颁发用于 Ingress 和其他组件的证书，来模仿这一过程。这包括我们的 Harbor 实例，这意味着为了让我们的集群能够拉取镜像，运行它的节点必须信任我们的 CA。为了完成这项工作，节点必须配置为信任我们的 CA。不幸的是，Kubernetes 没有 API 来信任私有 CA。

+   **驱动程序**：虽然在云管理的 Kubernetes 中不那么重要，但企业通常会使用经过认证的特定硬件堆栈，以确保与特定硬件兼容。例如，你的**存储区域网络**（**SAN**）可能有特定的内核驱动程序。如果你没有访问节点的权限，就无法安装这些驱动程序。

+   **支持的操作系统**：许多企业，特别是在高度监管行业的企业，想确保他们运行的是受支持的操作系统和配置。例如，如果你正在运行 Azure Kubernetes，但你的企业已经标准化使用 **Red Hat Enterprise Linux**（**RHEL**），你将需要创建一个自定义的节点镜像。

虽然需要访问你的节点会在多个方面复杂化部署，例如需要管理和保护访问权限，但它通常是部署和管理 Kubernetes 的一个必要“坏事”。

虽然你可能最熟悉在 Ubuntu、RHEL 或 RHEL 克隆系统上构建节点，但 Sidero 的 Talos Linux ([`www.talos.dev/`](https://www.talos.dev/)) 提供了一种创新的方式，通过将操作系统精简到启动 Kubernetes 所需的最小功能。意味着所有与操作系统的交互都是通过 API 完成的，来自 Kubernetes 或 Talos。这种管理方式非常有趣，因为你不再需要担心修补操作系统；所有升级都是通过 API 完成的。不再需要保护 SSH，但你仍然需要锁定 API。无法访问操作系统意味着你也不能直接部署驱动程序。我们之前提到的 Kargo 项目使用 Talos 作为其操作系统。当我想要集成我的 Synology **网络附加存储**（**NAS**）时，我不得不为此创建一个 `DaemonSet`，以便它能与 iSCSI 一起工作 ([`github.com/ContainerCraft/Kargo/blob/main/ISCSI.md`](https://github.com/ContainerCraft/Kargo/tree/main))。

由于我们使用的是自己的内部证书颁发机构（CA），因此你的节点至少需要能够包括自定义证书，以便进行定制。

你可能认为避免这种情况的一个简单方法是使用 Let’s Encrypt ([`letsencrypt.org/`](https://letsencrypt.org/)) 来生成证书，从而避免使用自定义证书颁发机构。这个方法的问题在于，它避免了企业常见的需求——使用自定义证书，而 Let’s Encrypt 并没有提供一个标准的内部证书颁发方式。它的自动颁发 API 基于公共验证技术，例如通过公开的 URL 或 DNS。这两种方式都不被大多数企业接受，因此 Let’s Encrypt 通常不允许用于内部系统。由于 Let’s Encrypt 通常不用于企业的内部系统，我们在这里也不使用它。

现在我们知道了为什么需要访问我们的节点，接下来我们将讨论网络管理。

## 访问节点上的服务

在本书中，我们假设一切都在单个虚拟机上运行。即使我们在 KinD 中运行多个节点，我们也通过端口转发来访问这些节点上运行的容器。由于这些集群更大，你可能需要采取不同的方法。我们在*第四章*《服务、负载均衡和网络策略》中介绍了 MetalLB，作为一个负载均衡器，可能是多个节点集群的一个很好的选择。你也可以将你的 `Ingress` 部署为一个 `DaemonSet`，让 Pods 使用主机端口来监听所有节点，然后使用 DNS 来解析所有节点。

无论你使用哪种方法，我们假设所有服务都会通过你的 `Ingress` 控制器进行访问。这包括文本或二进制协议：

+   控制平面：

    +   80/443: http/https

    +   22: ssh

    +   3306: MySQL

+   开发节点：

    +   80/443: http/https

    +   3306: MySQL

+   生产节点：

    +   80/443: http/https

    +   3306: MySQL

当我们开始部署时，我们会看到 NGINX 可以用来转发 Web 协议和二进制协议，这样你就可以为每个集群使用一个 `LoadBalancer`。需要注意的是，控制平面集群需要能够访问开发节点和生产节点上的 HTTPS 和 MySQL。同时需要注意，端口 `22` 将被控制平面集群使用，因此如果你计划直接为节点支持 SSH，如果没有使用像 MetalLB 这样的外部 `LoadBalancer`，你需要为其配置其他端口。

我们知道如何运行我们的集群，如何自定义工作节点，以及如何访问它们的服务。我们的最后一步是准备好 Pulumi。

## 部署 Pulumi

在上一章中，我们介绍了**基础设施即代码**（**IaC**）的概念，并表示我们将使用 Pulumi 的 IaC 工具来部署我们的 IDP。为了使用 Pulumi，你需要一个工作站来托管和运行客户端。

在深入讨论如何部署 Pulumi 之前，理解一些与 IaC 相关的关键概念非常重要，这些概念经常被忽视。所有 IaC 工具至少由三个组件组成：

+   **控制器**：控制器通常是运行 IaC 工具的工作站或服务。在本书中，我们假设一个工作站将运行我们的 Pulumi 程序。对于大规模或生产环境的实现，通常更好的做法是部署一个控制器服务，代替你运行 IaC 工具。对于 Pulumi，这可以是他们自己的 SaaS 服务或 Kubernetes 操作员。

+   **工具**：这是 IaC 的核心组成部分。它是你为构建基础设施而创建的部分，且特定于每个 IaC 工具。

+   **远程 API**：每个 IaC 工具通过 API 与远程系统进行交互。最初的 IaC 工具通过 SSH 与 Linux 服务器交互，然后通过它们自己的 API 与云服务进行交互。如今，IaC 工具通过自己的提供程序与单个系统进行交互，这些提供程序封装了目标系统的 API。这个过程的一个难点是如何保护这些 API。我们在本书中花了大量篇幅强调短生命周期令牌的重要性，这也可以应用于我们的 IaC 实现。

除了上述三个组件外，许多 IaC 工具还包括某种状态管理文件。在上一章中，我们描述了像 Pulumi 和 Terraform 这样的 IaC 工具如何基于你的 IaC 工具生成一个预期状态，并将其应用到下游系统中。这个状态将包含与基础设施相同的特权信息，应该视为“机密”。例如，如果你通过 IaC 配置一个数据库密码，你的状态文件就包含了该密码的副本。

对于我们的部署，我们将使用本地状态文件。Pulumi 提供了将状态存储在远程服务（如 S3 存储桶）或使用其自己的云服务中的选项。虽然这些选项比本地文件更适合管理，但我们不希望你为了阅读这本书和进行练习而需要注册任何服务，所以我们将使用本地文件来管理所有 Pulumi 状态。

如果你关注 IaC 行业新闻，可能看到过 HashiCorp（Terraform 和 Vault 的创建公司）在 2023 年夏季将开源许可证更改为“商业源代码许可证”。此举是为了应对大量提供“Terraform as a Service”而且没有向 HashiCorp 付费的 SaaS 提供商。这一许可证的变化导致许多这些 SaaS 提供商创建了 OpenTofu，这是 Terraform 的一个分支，遵循原始的 Apache 2 许可证。我们对这个情况不做任何评判或建议，仅仅指出，围绕状态和控制器的托管服务是 IaC 公司收入的主要来源。

由于 Pulumi 是一个商业化的开源软件包，你需要按照他们的说明将命令行 Pulumi 工具安装到你希望作为控制器的工作站上：[`www.pulumi.com/docs/install/`](https://www.pulumi.com/docs/install/)。

最后，你需要获取本章的 GitHub 代码库。你可以在以下 GitHub 仓库访问本章的代码：[`github.com/PacktPublishing/Kubernetes-An-Enterprise-Guide-Third-Edition/tree/main/chapter19`](https://github.com/PacktPublishing/Kubernetes-An-Enterprise-Guide-Third-Edition/tree/main/chapter19)。

既然我们已经了解了 IDP 构建的环境和要求，接下来可以深入到 IDP 的部署过程。

# 部署我们的 IDP

在我们技术要求处理完毕后，让我们来部署我们的门户！首先，我假设你有三个运行中的集群。如果每个集群都有一个`LoadBalancer`解决方案，那么下一步就是部署 NGINX。我们没有在 Pulumi 工具中包括 NGINX，因为根据集群的部署方式，这会影响 NGINX 的部署方式。例如，我没有使用典型的集群和`LoadBalancer`；我只是使用了单节点集群，并通过主机端口修补了 NGINX 以支持`80`和`443`端口。

我们还假设你已经附加了一种本地存储，并设置了默认的`StorageClass`。

我们将运行 NGINX，假设它将作为 HTTP(S)、MySQL 和 SSH 的 Ingress。这可以通过 NGINX 的 Helm chart 轻松实现。在所有三个集群上运行以下命令：

```
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace \
  --set tcp.3306=mysql/mysql:3306 
```

这将部署 NGINX 作为 Ingress 控制器，并启动一个`LoadBalancer`。

如果你使用的是单节点集群，现在可以用类似这样的命令来修补你的`Deployment`：`kubectl patch deployments ingress-nginx-controller -n ingress-nginx -p '{"spec":{"template":{"spec":{"containers":[{"name":"controller","ports":[{"containerPort":80,"hostPort":80,"protocol":"TCP"},{"containerPort":443,"hostPort":443,"protocol":"TCP"},{"containerPort":22,"hostPort":22,"protocol":"TCP"},{"containerPort":3306,"hostPort":3306,"protocol":"TCP"}]}]}}}}`。

这将强制通过你集群上部署的 NGINX 来转发端口。命令位于`chapter19/scripts/patch-nginx.txt`。

一旦 NGINX 部署完成，你将需要为所有三个`LoadBalancer` IP 配置 DNS 通配符。如果你没有 DNS 访问权限，可能会很想直接使用 IP 地址，但千万不要这样做！IP 地址在证书管理中可能会以奇怪的方式处理。如果你没有可用的域名，可以像本书中一样使用`nip.io`。我使用了三个域名，在所有示例中都会用到：

+   **控制平面 –** *`.idp-cp.tremolo.dev`

+   **开发集群 –** *`.idp-dev.tremolo.dev`

+   **生产集群 –** *`.idp-prod.tremolo.dev`

现在我们的环境已经准备好进行部署，接下来让我们通过创建一个 Pulumi 虚拟环境开始。

## 设置 Pulumi

在我们开始运行 Pulumi 程序启动部署之前，我们首先需要创建一个 Python 虚拟环境。Python 会动态链接到库，这种方式可能会与其他基于 Python 构建的系统发生冲突。为了避免这些冲突，在我们的 Windows 编程时代，这种问题被称为“DLL 地狱”，你需要创建一个仅用于你的 Pulumi 程序的虚拟环境。在一个新目录下，运行以下命令：

```
$ python3 -m venv . 
```

这将创建一个虚拟环境，你现在可以使用它来运行 Pulumi，而不会干扰其他系统。接下来，我们需要“激活”这个环境，使得我们的执行使用它：

```
$ . ./bin/activate 
```

最后一步是下载你的依赖项，前提是你已经检出了本书的最新 Git 仓库：

```
$ pip3 install -r /path/to/Kubernetes-An-Enterprise-Guide-Third-Edition/chapter19/pulumi/requirements.txt 
```

`pip3`命令会读取`requirements.txt`中列出的所有包并将它们安装到我们的虚拟环境中。此时，Python 已经准备好，我们需要初始化我们的堆栈。

准备 Pulumi 的第一步是“登录”以存储你的状态文件。有多种选项，从使用 Pulumi 的云平台到 S3 桶再到你的本地计算机。你可以在其官网上查看各种选项：[`www.pulumi.com/docs/concepts/state/`](https://www.pulumi.com/docs/concepts/state/)。我们将使用我们的本地目录：

```
$ pulumi login file://. 
```

这将在当前目录下创建一个名为`./pulumi`的目录，其中将包含你的后端。接下来，我们需要初始化一个 Pulumi“堆栈”以跟踪部署的状态：

```
$ cd chapter19
$ git archive --format=tar HEAD > /path/to/venv/chapter19/chapter19.tar
$ cd /path/to/venv/chapter19/
$ tar -xvf chapter19.tar
$ rm chapter19.tar 
```

接下来，编辑`/path/to/venv/chapter19/pulumi/Pulumi.yaml`，将`runtime.options.virtualenv`更改为指向`/path/to/venv`。最后，我们可以初始化我们的堆栈：

```
$ cd /path/to/venv/chapter19/pulumi
$ pulumi stack init bookv3-platform 
```

系统会要求你提供密码以加密你的机密信息。确保将其写在一个安全的地方！现在你将有一个名为`/path/to/venv/chapter19/pulumi/Pulumi.bookv3-platform.yaml`的文件，用于跟踪你的状态。

我们的环境现在已经准备好！接下来，我们将配置变量并开始部署。

## 初始部署

> 最终一致性是一个谎言——古老的云原生西斯格言

在 Kubernetes 世界中，我们经常假设“最终一致性”的理念，即创建一个控制循环，等待我们期望的条件变为现实。这通常是对系统的过于简化的看法，尤其是在处理企业系统时。所有这些的意思是，尽管我们几乎所有的部署都在一个 Pulumi 程序中管理，但它需要多次运行才能完全部署环境。在我们逐步讲解的过程中，我们将解释为什么需要单独运行每个步骤。

解决了这个问题后，我们需要配置我们的变量。为了最小化配置的数量，你需要设置以下内容：

| **选项** | **描述** | **示例** |
| --- | --- | --- |
| `openunison.cp.dns_suffix` | 控制平面集群的 DNS 域名 | `idp-cp.tremolo.dev` |
| `kube.cp.context` | 控制平面在控制平面 kubectl 配置中的 Kubernetes 上下文 | `kubernetes-admin@kubernetes` |
| `harbor:url` | 部署后 Harbor 的 URL | `https://harbor.idp-cp.tremolo.dev` |
| `kube.cp.path` | 控制平面集群的 kubectl 配置文件路径 | `/path/to/idp-cp` |
| `harbor:username` | Harbor 的管理员用户名 | `Always admin` |
| `openunison.dev.dns_suffix` | 开发集群的 DNS 后缀 | `idp-dev.tremolo.dev` |
| `openunison.prod.dns_suffix` | 生产集群的 DNS 后缀 | `idp-prod.tremolo.dev` |

表 19.1：Kubernetes 集群中的配置选项

为了简化操作，您可以自定义`chapter19/scripts/pulumi-initialize.sh`并运行它。您也可以通过运行以下命令手动设置这些选项：

```
$ pulumi config set option value 
```

其中`option`是您想设置的选项，`value`是它的`value`。最后，我们可以运行部署：

```
$ cd /path/to/venv/chapter19/pulumi
$ pulumi up -y 
```

您将被要求提供密码以解密您的密钥。一旦完成，初始部署将需要一段时间。根据您的网络连接速度和控制平面集群的性能，可能需要 10 到 15 分钟。

一旦所有内容部署完成，您将看到类似这样的消息：

```
.
.
.
Resources:
    + 53 created
Duration: 7m6s 
```

如果您查看命令的输出，您将看到所有已创建的资源！这与我们在上一章设计的内容一致。

我们不会逐行讲解所有的代码。代码超过了 45,000 行！部署完成后，我们将讲解其中的重点内容。

到目前为止，我们还有一些差距：

+   **Vault**：Vault 已部署，但尚未配置。我们不能配置 Vault，直到它被解封。

+   **GitLab**：GitLab 的基础部署已经完成，但我们还没有办法运行工作流。我们还需要生成一个访问令牌，以便 OpenUnison 可以与其交互。

+   **Harbor**：Harbor 正在运行，但没有 Harbor 管理员密码，我们无法完成 SSO 集成。我们还需要这个密码来与 OpenUnison 进行集成。

+   **OpenUnison**：基础的 OpenUnison 命名空间即服务门户已经部署，但我们还没有部署任何额外的配置来支持我们的 IDP。

接下来，让我们解封 Vault 并准备好部署。

## 解封 Vault

还记得在*第八章*，*管理密钥*中，我们需要通过提取随机生成的密钥并在 Pod 中运行脚本来解封 Vault 吗？在 Pulumi 中没有简单的方法来实现这一点，因此我们需要使用 Bash 脚本。我们还希望能够将解封的密钥存储在安全的地方，因为一旦获取，它们就不能再次获取。幸运的是，Pulumi 的密钥管理使得将密钥与其余配置存储在同一位置变得容易。首先，让我们解封我们的 Vault：

```
$ cd /path/to/venv/chapter19/pulumi
$ export KUBECONFIG=/path/to/cp/kubectl.conf
$ export PULUMI_CONFIG_PASSPHRASE=mysecretpassword
$ ../vault/unseal.sh 
```

在运行`unseal.sh`之前，重要的是设置`PULUMI_CONFIG_PASSPHRASE`或`PULUMI_CONFIG_PASSPHRASE_FILE`。设置完成后，如果运行`pulumi config`，你会看到多了两个密钥：

```
$ pulumi pulumi config --show-secrets
.
.
.
vault.key  hvs.I...
vault.tokens  {
                "unseal_keys_b64": 
.
.
. 
```

现在你的配置已存储在 Pulumi 配置中。如果你使用的是集中式配置，比如通过 Pulumi Cloud 或 S3 存储桶，这可能会更加有用！如果你因为某些原因需要重启 pod，你可以通过运行以下命令再次解锁：

```
$ cd /path/to/venv/chapter19/pulumi
$ export KUBECONFIG=/path/to/cp/kubectl.conf
$ export PULUMI_CONFIG_PASSPHRASE=mysecretpassword
$ ../vault/unseal_after_init.sh 
```

随着 Vault 准备好配置，接下来，我们将准备 Harbor 的配置。

## 完成 Harbor 配置

配置 Harbor 实际上非常简单：

```
$ cd /path/to/venv/chapter19/pulumi
$ export KUBECONFIG=/path/to/cp/kubectl.conf
$ export PULUMI_CONFIG_PASSPHRASE=mysecretpassword
$ ../scripts/harbor-get-root-password.sh 
```

这个脚本完成了两件事：

+   从`harbor-admin`密钥获取随机生成的密码，并将其存储在 Pulumi 配置中

+   设置一个标志，让我们的 Pulumi 程序知道要完成 SSO 配置

我们必须执行此步骤，因为 Harbor 提供程序使用了特定于`harbor`命名空间的配置选项。这与我们其他的配置选项不同。我们来看看在你的 Pulumi 程序中类似这样的一段代码：

```
my_config = pulumi.config("someconfig") 
```

你的代码并不是在说“获取名为`someconfig`的配置”；它是在说“获取我栈命名空间中名为`someconfig`的配置”。命名空间之间的分隔意味着我们的代码无法从另一个命名空间获取配置。实际上，这意味着我们在`harbor:url`、`harbor:password`以及`harbor:username`之间多次定义相同的信息。

在我们的场景中，这种做法看起来效率低下且容易出错，但在大规模部署中，这是一种确保分隔孤岛的好方法。在许多部署中，拥有 Harbor 的人和可能拥有自动化的人并不是同一个人。通过不允许我们的代码访问`harbor`命名空间，但能够调用依赖于它的库，我们能够使用这些密钥数据而不实际了解它！当然，由于我们使用的是单一的密钥集，并且我们有权限访问它，这个安全性优势被抵消了。不过，如果你使用的是集中管理的 Pulumi 控制器服务，它允许开发人员编写从不直接知道所依赖的密钥数据的代码。

现在 Harbor 已准备好进行最终配置，我们需要在 GitLab 中执行一些手动步骤。我们将在下一节中进行。

## 完成 GitLab 配置

在 GitLab 中我们缺少两个关键组件。首先，我们需要生成一个令牌，供 OpenUnison 在自动化 GitLab 时使用。另一个是我们需要手动配置一个 runner。稍后，我们将使用 GitLab 集成的工作流构建一个容器，并将其推送到 Harbor。GitLab 通过启动一个 Pod 来完成此操作，这需要一些自动化。启动这些 Pod 的服务需要在 GitLab 中注册。这是有道理的，因为你可能希望在本地 Kubernetes 集群或远程云中运行服务。为了处理这种情况，我们需要告诉 GitLab 生成一个 runner 并给我们一个注册令牌。首先，我们将生成 runner 注册令牌。

### 生成 GitLab Runner

第一步是登录到 GitLab。我们还没有配置单点登录（SSO），所以你需要使用 root 凭证登录。这些凭证作为一个秘密存储在 `GitLab` 命名空间下，文件名以 `gitlab-initial-root-password` 结尾。一旦获得密码，使用用户名 `root` 登录到 GitLab。URL 将是 `https://gitlab.controlplane.dns.suffix`，其中 `controlplane.dns.suffix` 是你的控制平面集群的 DNS 后缀。对我来说，URL 是 `https://gitlab.idp-cp.tremolo.dev/`。

登录后，点击左下角的 **Admin Area**：

![图形用户界面，网站  描述自动生成图 19.1：GitLab 主屏幕接下来，展开 **CI/CD** 并点击 **Runners**：![图形用户界面，应用程序，团队  描述自动生成](img/B21165_19_02.png)

图 19.2：GitLab 管理区域

屏幕加载完成后，点击 **新实例 runner**：

![图形用户界面，应用程序  描述自动生成](img/B21165_19_03.png)

图 19.3：GitLab Runners

当**新实例 runner**屏幕加载时，勾选**运行未标记的作业**，因为我们只在自己的集群上运行作业。你可以使用这些标签来管理跨多个平台运行的作业，类似于如何使用节点标签在 Kubernetes 中管理工作负载的运行位置。接下来，点击**创建 runner**：

![图形用户界面，文本，应用程序，电子邮件  描述自动生成](img/B21165_19_04.png)

图 19.4：GitLab 新实例 runner

最后，你将获得一个令牌，我们可以在我们的 Pulumi 配置中进行配置：

![图形用户界面，应用程序，团队  描述自动生成](img/B21165_19_05.png)

图 19.5：新 runner 令牌

复制此令牌并在 Pulumi 中进行配置：

```
$ cd /path/to/venv/chapter19/pulumi
$ export PULUMI_CONFIG_PASSPHRASE=mysecretpassword
$ pulumi config set gitlab.runner.token \
  'glrt-Y-fSdvy_6_xgXcyFW_PW' --secret 
```

接下来，我们将配置一个令牌来自动化 GitLab。

### 生成 GitLab 个人访问令牌

在上一节中，我们配置了一个 runner。接下来，我们需要一个令牌，以便 OpenUnison 可以自动化将 GitLab 配置到系统中。不幸的是，GitLab 不提供令牌的替代方式。你可以定期使令牌过期，但需要替换它。也就是说，当你以 root 用户身份登录到 GitLab 时，点击左上角的彩色图标，然后点击**偏好设置**：

![图形用户界面，网站 描述自动生成](img/B21165_19_06.png)

图 19.6：GitLab 偏好设置

一旦**偏好设置**页面加载完毕，点击**访问令牌**，然后点击**添加新令牌**：

![图形用户界面，文本，应用程序，电子邮件 描述自动生成](img/B21165_19_07.png)

图 19.7：GitLab 访问令牌

当新的令牌页面加载时，你需要为它命名并设置过期时间，并点击**api**选项以授予完全访问权限。完成后，点击页面底部的**创建个人访问令牌**按钮：

![图形用户界面，应用程序 描述自动生成](img/B21165_19_08.png)

图 19.8：创建新的 GitLab 个人访问令牌

令牌生成后，最后一步是复制令牌，并将其配置为 Pulumi 配置机密：

![图形用户界面，文本，应用程序，电子邮件 描述自动生成](img/B21165_19_09.png)

图 19.9：GitLab 个人访问令牌

复制令牌并将其设置到 Pulumi 配置中：

```
$ cd /path/to/venv/chapter19/pulumi
$ export PULUMI_CONFIG_PASSPHRASE=mysecretpassword
$ pulumi config set gitlab.root.token \
  'glpat-suHtqupeNetAsYfGwVyz' --secret 
```

我们现在已经完成了完成控制平面配置所需的额外步骤。接下来，我们将在 Pulumi 程序中完成控制平面的部署。

## 完成控制平面部署

下一步是重新运行我们的 Pulumi 程序以完成集成：

```
$ cd /path/to/venv/chapter19/pulumi
$ export PULUMI_CONFIG_PASSPHRASE=mysecretpassword
$ pulumi up -y 
```

这应该比初始运行所需的时间更短，但仍然需要几分钟。OpenUnison 需要再次部署，并应用新的配置选项，这将是最耗时的部分。

部署完成后，我们还有一项任务需要在控制平面上完成。我们需要更新 NGINX，将 `22` 端口的 SSH 转发到 GitLab shell `服务`。我们可以通过获取 `GitLab` 命名空间中的 shell `服务` 名称，然后更新我们的控制平面 NGINX 来实现：

```
$ cd chapter19/scripts
$ ./patch_nginx_ssh.sh 
```

一旦 NGINX 恢复运行，你应该可以通过 SSH 登录到 GitLab。此时，你可以访问 `https://k8sou.idp-cp.tremolo.dev`（将 `idp-cp.tremolo.dev` 替换为你的控制平面后缀），并使用用户名 `mmosley` 和密码 `start123` 登录到 OpenUnison：

![图形用户界面，应用程序 描述自动生成](img/B21165_19_10.png)

图 19.10：OpenUnison 主页面

我们还没有完成开发或生产系统的集成，但你应该通过 OpenUnison 使用 SSO 登录 Vault、GitLab、Harbor、ArgoCD 和控制平面 Kubernetes。由于你是第一个登录的人，你将自动成为控制平面集群管理员和顶级审批人。

配置好控制平面后，Pulumi 的最后一步是将开发和生产集群加入到系统中，接下来我们将介绍这个步骤。

## 集成开发和生产

到目前为止，我们的所有工作都集中在控制平面上。然而，这里不会有任何用户工作负载。我们的租户将位于开发和生产集群中。为了确保我们的自动化计划正常运行，我们需要将这些集群集成到 OpenUnison 中，以便为所有自动化 API 调用使用短生命周期的令牌。幸运的是，OpenUnison 已经具备我们需要的所有功能，并且已经集成到 OpenUnison 的 Helm 图表中。

第一步是将 NGINX 部署到每个集群。我们还将为 MySQL 转发`3306`端口，以便控制平面上的 OpenUnison 可以与每个集群上的 MySQL 通信。虽然我们本可以使用控制平面上的 MySQL 来支持 vClusters，但我们不希望在控制平面出现问题时影响到开发或生产集群的运行。通过在每个集群上运行 MySQL，一个集群的故障不会影响到其他集群的操作。运行以下命令：

```
$ export KUBECONFIG=/path/to/idp-dev.conf
$ helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace \
  --set tcp.3306=mysql/mysql:3306
$ export KUBECONFIG=/path/to/idp-prod.conf
$ helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace \
  --set tcp.3306=mysql/mysql:3306 
```

启动后，您将能够更新 Pulumi 的配置。

| **选项** | **描述** | **示例** |
| --- | --- | --- |
| `kube.dev.path` | 开发集群的 kubectl 配置文件路径 | `/path/to/idp-dev` |
| `kube.dev.context` | 开发集群的 Kubernetes 上下文，在其 kubectl 配置文件中 | `kubernetes-admin@kubernetes` |
| `kube.prod.path` | 生产集群的 kubectl 配置文件路径 | `/path/to/idp-prod` |
| `kube.prod.context` | 生产集群的 Kubernetes 上下文，在其 kubectl 配置文件中 | `kubernetes-admin@kubernetes` |

表 19.2：Kubernetes 配置选项以及开发和生产集群中的路径

配置添加完毕后，我们可以完成 Pulumi 的部署：

```
$ cd /path/to/venv/chapter19/pulumi
$ export PULUMI_CONFIG_PASSPHRASE=mysecretpassword
$ pulumi up -y 
```

这将需要几分钟时间，但完成后，我们将在 OpenUnison 中进行最后一步以完成部署。如果一切顺利，您应该会看到类似的内容：

```
Resources:
    + 46 created
    ~ 2 updated
    +-1 replaced
    49 changes. 68 unchanged
Duration: 4m59s 
```

恭喜，我们的基础设施已经部署完成！三个集群、十多个系统，全部集成完毕！接下来，我们将使用 OpenUnison 的工作流完成最后的集成步骤。

## 使用 OpenUnison 引导 GitOps

我们已经部署了几个系统来支持我们的 GitOps 工作流，并通过 SSO 将它们集成，用户可以登录，但我们还没有启动 GitOps 引导过程。在这个上下文中，“引导”是指在 GitLab 中设置一些初始的仓库，将它们集成到 Argo CD，并使它们同步到控制平面、开发集群和生产集群。这样，当我们添加新租户时，我们将通过在 Git 中创建清单而不是直接写入集群的 API 服务器来进行。我们仍然会向 API 服务器写入短暂对象，比如用于部署 vClusters 并将其与控制平面集成的`Jobs`，但除此之外，我们希望所有内容都写入 Git 中。

我们将把最后一步放在 OpenUnison 中，而不是 Pulumi。您可能会想，为什么我们在这部分使用 OpenUnison，而不是使用 Pulumi？最初的计划是使用 Pulumi，但不幸的是，Pulumi 的 GitLab 提供程序中存在一个已知的 bug，导致我们无法在 GitLab 社区版中创建组。由于 OpenUnison 的工作流引擎已经具备此功能，并且这是一个只需要执行一次的步骤，我们决定直接在 OpenUnison 的工作流引擎中完成它。

说完这些后，使用上一节的说明登录到 OpenUnison。接下来，点击左侧的**请求访问**，选择**Kubernetes 管理**，然后通过将**Kubernetes-prod 集群管理员**和**Kubernetes-dev 集群管理员**添加到购物车来添加开发和生产集群：

![图形用户界面，应用程序描述自动生成](img/B21165_19_11.png)

图 19.11：OpenUnison 请求访问开发和生产集群管理

添加到购物车后，点击左侧的新**结账**菜单选项，添加请求的理由，然后点击**提交您的请求**。

![图形用户界面，应用程序，Teams 描述自动生成](img/B21165_19_12.png)

图 19.12：提交集群管理访问请求

刷新屏幕后，您将看到有两个待处理的请求。像在*第九章*中一样处理它们，*使用 vClusters 构建多租户集群*，然后退出。当您重新登录时，您将访问开发和生产集群以及控制平面。

登录后，返回到**请求访问**，点击**OpenUnison 内部管理工作流**，然后将**初始化 OpenUnison**添加到购物车。像之前一样，以某个理由将其从购物车结账。这次不会有审批步骤。虽然需要几分钟，但一旦完成，您可以登录 Argo CD，您将看到三个新项目：

![图形用户界面，应用程序描述自动生成](img/B21165_19_13.png)

图 19.13：OpenUnison 初始化后的 Argo CD

状态为**未知**，因为我们还没有信任 GitLab 的 ssh 服务的密钥。使用您喜欢的方法下载 Argo CD 命令行工具并运行以下命令：

```
$ argocd login --grpc-web \
  --sso argocd.idp-cp.tremolo.dev
$ ssh-keyscan gitlab-ssh.idp-cp.tremolo.dev \
  | argocd cert add-ssh --batch 
```

这将把正确的密钥添加到 Argo CD。几分钟后，你会看到我们的应用程序在 Argo CD 中正在同步！现在是个好时机来浏览 Argo CD 和 GitLab。你将看到我们的 GitOps 基础设施的基本框架是怎样的。你还可以查看`chapter19/pulumi/src/helm/kube-enterprise-guide-openunison-idp/templates/workflows/initialization/init-openunison.yaml`中的入职流程。我认为你最重要的发现是，我们如何通过身份将所有内容联系在一起。这在 DevOps 世界中是常常被忽视的，尤其是在 Argo CD 中，我们创建了一个`AppProject`，限制可以添加哪些仓库和集群。然后我们为每个集群创建一个`Secret`，但该`Secret`不包含任何秘密数据。最后，我们生成`ApplicationSets`来生成`Application`对象。在部署我们的租户时，我们还会遵循这个模式。

现在，你已经拥有了一个可工作的多租户 IDP！这大约花费了 30 页的解释，可能几个小时的集群设计和设置，以及几千行的自动化代码，但你已经拥有它了！接下来，是时候部署一个租户了！

# 租户入职

到目前为止，我们已经花费了所有的时间来搭建我们的基础设施。我们有一个 Git 仓库，控制平面的集群，以及开发和生产环境的集群，一个 GitOps 控制器，一个秘密管理器，和一个 SSO 和自动化系统。现在我们可以构建我们的第一个租户了！好消息是，这部分非常简单，不需要任何命令行工具。就像我们在*第九章*中介绍的那样，*使用 vClusters 构建多租户集群*，我们将使用 OpenUnison 作为请求和批准新租户的门户。如果你已经登录到 OpenUnison，请退出并重新登录，这次使用用户名`jjackson`和密码`start123`。你会注意到，在主页上，你的徽章数量大大减少，因为你还没有任何权限！我们将通过创建一个新租户来解决这个问题。点击**新 Kubernetes 命名空间**徽章：

![图形用户界面，应用程序 描述自动生成](img/B21165_19_14.png)

图 19.14：使用用户名 jjackson 登录 OpenUnison

打开后，使用名称`myapp`，理由为`new application`，然后点击**保存**：

![图形用户界面，应用程序，团队 描述自动生成](img/B21165_19_15.png)

图 19.15：新项目屏幕

保存后，退出并重新登录，但这次使用用户名`mmosley`和密码`start123`。你会看到有一个待处理的审批。点击**打开审批**，**处理请求**，提供一个理由，然后点击**批准请求**。当按钮变为青绿色时，点击**确认审批**。这个过程会花一些时间。并不是说步骤本身计算量很大，而是大部分时间是等待系统同步。这是因为我们并没有直接向集群的 API 服务器写入数据，而是通过 GitLab 部署创建清单，然后通过 Argo CD 将这些清单同步到集群中。我们还在部署多个 vCluster 和 OpenUnison，这也需要时间。如果你想查看进度，可以在控制平面的`openunison-orchestra` pod 中查看日志。这可能需要 10 到 15 分钟才能完全部署。由于我们没有一个工作的邮件服务器，因此你可以通过登录 Argo CD 来了解工作流是否完成，并且你将看到两个新应用程序：一个是我们的开发 vCluster，另一个是我们的生产 vCluster：

![图形用户界面，文本，应用程序  自动生成的描述](img/B21165_19_16.png)

图 19.16：新租户部署后的 Argo CD

如果你点击`myapp/k8s-myapp-prod`应用程序，你会看到我们并没有同步太多内容：

![图形用户界面，文本，应用程序，聊天或文本消息  自动生成的描述](img/B21165_19_17.png)

图 19.17：生产租户 Argo CD 应用程序

到目前为止，我们创建了一个可以与 Vault 通信的`ServiceAccount`，以及一个同步我们为 Harbor 生成的拉取秘钥的`ExternalSecret`，将其同步到默认命名空间。接下来，我们要查看我们的新租户！确保退出所有内容，或者直接在不同的浏览器中打开一个隐身/私密窗口，再次以`jjackson`身份登录。

![图形用户界面，应用程序  自动生成的描述](img/B21165_19_18.png)

图 19.18：租户部署后作为 jjackson 的 OpenUnison

你会看到现在我们的集群管理应用程序有了徽章。当你登录到这些应用程序时，你会发现你的视图是有限制的。Argo CD 仅允许你与租户的`Application`对象进行交互。GitLab 仅允许你查看与租户相关的项目。Harbor 仅允许你查看租户的镜像，最后，Vault 仅允许你与租户的秘密进行交互。如果你查看屏幕顶部树形结构中的`myapp-dev`和`myapp-prod`部分，你将看到现在你有了两个 vCluster 的令牌和仪表板。身份管理真是太神奇了！

到目前为止，我们已经构建了大量的基础设施，并使用共同的身份创建了一个租户，允许每个系统的基于策略的系统来决定如何划定边界。接下来，我们将向我们的租户部署一个应用程序。

# 部署应用程序

我们已经建立了相当多的基础设施来支持我们的多租户平台，并且现在已经有一个租户可以部署我们的应用程序了。让我们继续部署我们的应用程序吧！

如果我们以 `jjackson` 登录 GitLab，我们会看到有三个项目：

+   **myapp-prod/myapp-application**：此代码库将存储我们的应用程序代码和生成容器的构建文件。

+   **myapp-dev/myapp-ops**：存储开发集群清单文件的代码库。

+   **myapp-prod/myapp-ops**：存储生产集群清单文件的地方。

从开发项目到生产项目没有直接的分支。这是我们最初的设计思路，但从开发到生产的严格路径效果不好。开发环境和生产环境通常不同，而且基础设施的所有者也往往不同。例如，我维护一个公共安全身份提供者，而我们的开发环境并未与生产环境所涉及的所有司法管辖区建立信任。为了解决这个问题，我们设置了一个额外的系统来替代这些身份提供者。这些差异使得直接自动将开发中的更改合并到生产环境中变得困难。

话虽如此，让我们构建我们的应用程序。第一步是生成一个 SSH 密钥并将其添加到 `jjackson` 在 GitLab 上的个人资料中，以便我们可以将代码提交进去。更新 SSH 密钥后，克隆 `myapp-dev/myapp-ops` 项目。该项目包含了你的开发集群的清单文件。你会看到已经有清单文件支持将 Harbor 的拉取密钥同步到默认命名空间。创建一个名为 `yaml/namespaces/default/deployments` 的文件夹，并将 `chapter19/examples/ops/python-hello.yaml` 添加进去。提交并推送到 GitLab。几分钟后，Argo CD 将尝试同步，但会失败，因为镜像指向的是不存在的内容。这没关系，接下来我们会处理这个问题。

![图形用户界面，应用程序 描述自动生成](img/B21165_19_19.png)

图 19.19：Argo CD 部署失败

我们的部署中的镜像标签将在应用程序构建成功后更新。这为我们提供了所需的自动化，并且如果需要的话，可以轻松回滚。我们的应用程序是一个简单的 Python Web 服务。克隆 `myapp-prod/myapp-application` 项目并将其复制到 `chapter19/examples/myapp` 文件夹中。需要注意两点：

+   **source**：此文件夹包含我们的 Python 源代码。这部分不太重要，我们不会花太多时间在它上面。

+   **.gitlab-ci.yml**：这是 GitLab 构建脚本，负责生成 Docker 容器，将其推送到 Harbor，然后在 Git 中修补我们的 Deployment。

如果你查看 `.gitlab-ci.yml` 文件，你可能会注意到它看起来与我们在之前版本中构建的 Tekton 任务类似。GitLab 的管道的相似之处在于，每个阶段都作为集群中的 pod 运行。我们有两个阶段。第一个阶段构建我们的容器，第二个阶段则部署它。

构建阶段使用了 Google 提供的一款名为**Kaniko**的工具（[`github.com/GoogleContainerTools/kaniko`](https://github.com/GoogleContainerTools/kaniko)），它无需与 Docker 守护进程交互即可构建和推送 Docker 镜像。这意味着我们的构建容器不需要特权容器，从而使得构建环境更容易确保安全。Kaniko 使用 Docker 配置来管理凭据。如果你使用 AWS 或其他主要云服务提供商，可以直接与其 IAM 解决方案集成。在这个实例中，我们使用 Harbor，因此我们的 OpenUnison 工作流将 Docker `config.json` 作为变量配置到 GitLab 项目中。构建过程使用 Git SHA 哈希的简短版本作为标签。这样，我们可以将每个容器与生成它的构建和提交进行关联。

构建的第二阶段是部署。此阶段首先检出我们的`myapp-dev/myapp-ops`项目，使用正确的镜像 URL 修补我们的`Deployment`，然后提交并推送回 GitLab。不幸的是，GitLab 和 GitHub 都没有提供便捷的方式使用工作流的身份检出代码。为了解决这个问题，我们的 OpenUnison 入职工作流为我们的 Ops 项目创建了一个“部署密钥”，将其标记为可写，然后将私钥添加到应用项目中。这样，应用项目就可以配置 SSH，允许克隆 DevOps 仓库，生成补丁，然后提交并推送。完成后，Argo CD 会在几分钟内检测到变化，并将其同步到你的开发 vCluster 中。

![图形用户界面，应用程序 描述自动生成](img/B21165_19_20.png)

图 19.20：具有工作同步部署的 Argo CD

假设一切顺利，你现在拥有了一个自动更新的开发环境！你现在可以在推送到生产环境之前运行任何自动化测试。由于我们目前没有自动化的流程来完成这项工作，让我们探讨一下如何可以实现它。

## 推送到生产环境

到目前为止，我们已经将应用程序部署到开发 vCluster。那么我们的生产 vCluster 呢？`myapp-dev/myapp-ops` 项目和 `myapp-prod/myapp-ops` 项目之间没有直接关系。如果你读过这本书的前两版，你可能还记得开发项目是生产项目的一个分支。你可以通过提交合并请求（GitLab 的版本是拉取请求）将容器从开发环境推广到生产环境，一旦通过，开发中的更改就会合并到生产环境中，允许 Argo CD 将它们同步到我们的集群中。这个方法的问题在于它假设开发和生产是完全相同的，而这从来都不是事实。即使是数据库的主机名不同，也会导致这个方法失效。我们需要打破这种模式，才能使我们的集群可用。

对于我们的实验室，最简单的方法是创建 `myapp-prod/myapp-ops` 项目中的 `yaml/namespaces/default/deployments` 目录，将 `chapter19/examples/ops/python-hello.yaml` 文件添加到其中，并将镜像更新为指向我们在 Harbor 中的当前镜像。这并不是一个高度自动化的过程，但它是可行的。最终，Argo CD 会完成清单的同步，我们的服务将运行在生产集群中。

如果你想更进一步地自动化，以下是多个选项：

+   **创建 GitLab 工作流**：工作流可以用于自动化更新，方式类似于生产环境。你需要某种触发方式，但这个解决方案效果不错，因为你可以利用 GitOps 来跟踪发布过程。

+   **创建自定义任务或脚本**：我们正在运行 Kubernetes，因此有多种方法可以创建批处理任务来执行升级过程。根据你选择在 Kubernetes 中运行批处理任务的方式，这也可以通过 GitOps 来实现。

+   **Akuity Kargo**：一个新的项目，专注于以一致的方式解决这个问题。

前两个选项是同一主题的变种：定制化的发布脚本。并没有说这是一种反模式，只是看起来可能有更好的方法，或者至少有更一致的方法来实现这一点。于是出现了 Akuity Kargo 项目（[`github.com/akuity/kargo`](https://github.com/akuity/kargo)），不要与 Container Craft Kargo 项目混淆！Akuity 是由许多 Argo CD 的原始开发者创立的。Kargo 项目不属于 Argo 生态系统，它是完全独立的。它采用了一种有趣的方法来自动化跨多个仓库同步的过程，以推动系统在不同环境中的推广。我们最初考虑将这个工具直接集成到我们的集群中，但它仍然处于版本 1 之前，因此我们决定暂时提及它。毫无疑问，这是一个我们会持续关注的项目！

现在我们已经将应用程序发布到生产环境，接下来可以做什么呢？当然是添加更多用户！我们接下来将探讨如何扩展我们的平台。

# 向租户添加用户

现在我们已经创建了租户并部署了应用程序，也许是时候看看如何添加新用户了。好消息是我们可以添加更多用户！OpenUnison 使得团队成员可以轻松请求访问权限。登录 OpenUnison，点击**请求访问**，然后选择你想要的应用程序和角色。应用程序所有者将负责批准该访问权限。这个方法的一个优点是集群所有者无需参与。完全由应用程序所有者决定谁可以访问他们的系统。然后，权限将在平台的所有组件中按需进行配置。

![图形用户界面，应用程序，Teams  描述自动生成](img/B21165_19_21.png)

图 19.21：在 OpenUnison 中为应用程序角色添加用户

我们已经部署了平台，并且知道如何部署租户以及如何向租户中添加新成员。我们可以做些什么来改善我们的新平台？有哪些空白需要填补？接下来我们将逐一讲解。

# 扩展我们的平台

在过去的两章中，我们已经涵盖了构建多租户平台的相当多内容。我们讲解了 GitOps 的工作原理、不同的策略以及像 Pulumi 这样的 IaC 工具如何简化自动化。最后，我们在三个集群上构建了我们的多租户平台。我们的平台包括使用 GitLab 的 Git 和构建、使用 Vault 的机密管理、使用 Argo CD 的 GitOps、Harbor 中的 Docker 注册表，最后，所有这些都通过 OpenUnison 的身份集成在一起。就这样，对吧？不，遗憾的是还没有。这一节将涵盖一些空白或我们的平台可以扩展的地方。首先，我们从身份管理开始。

## 不同的身份来源

本书中我们特别关注的一个领域是用户身份如何跨越构成集群的各个系统边界。在这个平台中，我们使用 Active Directory 进行用户身份验证，并使用 OpenUnison 的内部组进行授权。类似于*第九章*，我们也可以将企业的组集成到授权中。我们还可以扩展至 Active Directory 以外，使用 Okta、Entra ID（前身为 Azure AD）、GitHub 等。一个很好的补充是集成多因素身份验证。我们在本书中多次提到过这一点，但值得再强调一遍：多因素身份验证是帮助锁定环境最简单的方法之一！

在了解了其他用户识别方式之后，让我们看看监控和日志记录。

## 集成监控和日志记录

在*第十五章*，《监控集群和工作负载》中，我们学习了如何使用 Prometheus 监控 Kubernetes，并使用 OpenSearch 聚合日志。我们没有将这些系统集成到我们的平台中。我们这么做有几个原因：

+   **简化**：这两章已经足够复杂了，不需要更多集成的内容！

+   **缺乏多租户**：Prometheus 没有身份的概念，Grafana Community 版只允许两个角色。看来 OpenSearch 支持多租户，但这需要相当大的工程工作量。

+   **vCluster 的复杂性**：这和多租户问题类似；我们会为每个 vCluster 配置一个 Prometheus 吗？虽然有方法可以做到，但它们可能需要一本独立的书来详细讨论。

鉴于该解决方案是为生产环境设计的，你需要将某种监控和日志记录工具集成到你的主集群和 vCluster 中。这可能会很复杂，但绝对值得。想象一下，应用程序的拥有者能够从一个地方查看所有日志，而无需登录到集群。工具都已经具备，关键在于如何集成！

我们知道监控集群很重要，但接下来我们来谈谈政策管理。

## 集成政策管理

本书涵盖了两章关于政策管理的内容，并在我们关于 Istio 的章节中包含了授权管理。这是 Kubernetes 的一个重要方面，但我们没有在我们的平台中包括它。我们还花了一章讨论了运行时安全性。我们决定不包括政策管理，因为虽然它在生产环境中是必需的，但对实验室本身并没有提供额外的好处。不过，对于任何生产部署，它绝对是需要包括的！

现在我们已经讨论了在平台中未包括的技术，让我们来谈谈你可以替换的部分。

## 替换组件

我们之所以选择这些组件，是因为我们希望展示如何基于书中学到的内容构建平台。我们有意识地选择了仅使用开源项目，避免使用任何需要注册或试用的服务。话虽如此，你也可以用像 **AKeyLess** 这样的服务替换 Vault，或者用 GitHub 替换 GitLab 等。这是你的环境，选择最适合你的工具吧！

平台的故事远未结束，但这也标志着我们书籍的结束！

# 概述

本章涉及了相当多的内容。我们从构建三个 Kubernetes 集群来支持我们的平台开始。一旦集群搭建好，我们通过 Pulumi 部署了 `cert-manager`、Argo CD、MySQL、GitLab、Harbor、Vault 和 OpenUnison，将它们集成在一起。有了平台，我们部署了一个租户，看看如何使用自动化和 GitOps 简化管理，最后讨论了更新和调整多租户平台的不同方式。要在一个章节中覆盖这么多内容，实属不易！

我要感谢 Kat Morgan 为本章提供了 Pulumi 代码，作为我开始该章的基础，并在我遇到问题时给予了帮助。

最后，*感谢您！* 通过 19 章和数十种不同的技术、实验和系统，您和我们一起踏上了企业云原生领域的精彩旅程。我们希望您在阅读和使用本书的过程中，能像我们编写它时一样感到愉快。如果您遇到问题或只是想打个招呼，请在我们的 GitHub 仓库中提一个问题。如果您有任何想法或建议，我们也非常欢迎！我们不能再强调这一点了，再次感谢您与我们一起同行！

# 问题

1.  Kubernetes 有一个用于信任证书的 API：

    1.  正确

    1.  错误

1.  Pulumi 可以在哪里存储状态？

    1.  本地文件

    1.  S3 存储桶

    1.  Pulumi 的 SaaS 服务

    1.  上述所有

1.  OpenUnison 可以使用哪些身份源？

    1.  Active Directory

    1.  Entra ID（前身为 Azure AD）

    1.  Okta

    1.  GitHub

    1.  上述所有

1.  GitLab 让你使用工作流的令牌从另一个代码库中检出代码：

    1.  正确

    1.  错误

1.  Argo CD 可以检查代码库的变动：

    1.  正确

    1.  错误

# 答案

1.  b: 错误：节点的操作系统必须信任远程证书。

1.  d: 所有这些选项都是有效的。

1.  e: OpenUnison 支持所有这些选项。

1.  b: 错误：GitLab 和 GitHub 都不支持使用工作流令牌来检出其他代码库。

1.  a: 正确：让 Argo CD 检查代码库的更新，而不是使用 webhook，可以简化部署过程。

# 加入我们书籍的 Discord 空间

加入本书的 Discord 工作区，与作者进行每月一次的 *问我任何问题* 会话：

[`packt.link/K8EntGuide`](https://packt.link/K8EntGuide)

![](img/QR_Code965214276169525265.png)
