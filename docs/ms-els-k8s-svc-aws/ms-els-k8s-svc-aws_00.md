# 前言

欢迎！这是一本关于使用**弹性 Kubernetes 服务**（**EKS**）在 AWS 上轻松部署和管理 Kubernetes 集群的实用书籍。使用 EKS，运行 Kubernetes 变得轻而易举，因为你不再需要担心管理底层基础设施的复杂性。**Kubernetes**（**K8s**）是全球发展最快的开源项目之一，并迅速成为云原生应用程序的事实标准容器编排平台。

但是，对于那些不熟悉 AWS 的人来说，你可能会想，*“为什么在 AWS 上运行 Kubernetes 这么有挑战？”* 这里有一些因素可能使之变得困难。一个主要的问题是配置和管理 AWS 基础设施，包括虚拟网络和安全组。此外，管理 Kubernetes 集群所需的资源本身也可能带来一些挑战。与其他 AWS 服务（如负载均衡器和存储）集成，也可能引入复杂性。然而，EKS 已经启用了许多功能来简化这些问题，因此请放心，通过时间和努力，你可以精通在 AWS 上管理 Kubernetes 集群——而且回报将是值得的。

本书详细介绍了 AWS 托管的 EKS 服务，从其基本架构和配置到高级用例，如 GitOps 或服务网格。该书旨在带领读者从对 K8s 和 AWS 平台的基础理解，逐步掌握如何创建 EKS 集群，并在其上构建和部署生产工作负载。

在本书中，我们将深入探讨各种技术，帮助你优化 EKS 集群。内容涵盖广泛，包括网络、安全、存储、扩展、可观测性、服务网格和集群升级策略。我们将本书结构化为一个逐步引导你掌握 AWS 上 EKS 的指南。每章涵盖一个特定主题，并包括实用示例、技巧和最佳实践，帮助你理解并在现实场景中应用这些概念。

我们的目标不仅是帮助你掌握成功所需的技术技能，还希望加深你对基础概念的理解，以便你能够将其应用于自己独特的情况。

# 本书的读者群体

本书面向那些对 AWS 平台和 K8s 几乎没有经验的工程师和开发人员，旨在帮助他们了解如何使用 EKS 在其环境中运行容器化工作负载，并将其与其他 AWS 服务集成。这是一本实用的指南，包含大量代码示例，因此建议熟悉 Linux、Python、Terraform 和 YAML。

总的来说，这本书的目标读者主要包括三个角色，他们将从中获得实用的见解：

+   **开发人员和 DevOps 工程师**：他们将理解 AWS 上的 Kubernetes 环境，知道如何通过 EKS 配置集群以运行云原生应用程序，并学习 CI/CD 实践。

+   **云架构师**：他们将全面了解在 AWS 上运行 Kubernetes 时，如何设计架构良好的云基础设施。

+   **Kubernetes 管理员**：集群管理员将学习如何在 AWS 上管理 Kubernetes 工作负载的实际操作方法。此外，他们将全面了解 EKS 的功能，以增强集群的可扩展性、可用性和可观察性。

无论你是刚刚开始学习云计算，还是希望拓展知识和技能，本书都适合每个拥有 AWS 账户并希望开始 EKS 之旅的人。

# 本书涵盖的内容

*第一章*，*Kubernetes 和容器的基础知识*，介绍了 Kubernetes 和容器技术。它还深入讲解了容器的构成元素、容器编排器的概念以及 Kubernetes 架构。

*第二章*，*介绍 Amazon EKS*，提供了全面的指南，解释了什么是 Amazon EKS、它背后的架构、定价模型以及用户可能犯的常见错误。本章还简要概述了在 AWS 上运行工作负载的选择：使用 EKS 或自管理的 Kubernetes 集群。

*第三章*，*构建你的第一个 EKS 集群*，一步步探索创建第一个 EKS 集群的不同选项，并概述了构建工作流时的自动化过程，包括 AWS 控制台、AWS CLI、eksctl、AWS CDK 和 Terraform。

*第四章*，*在 EKS 上运行你的第一个应用程序*，讲解了在 EKS 上部署和操作一个简单应用程序的不同方式，包括如何实现并暴露应用程序以使其对外可访问。它还涉及了可视化工作负载的工具。

*第五章*，*使用 Helm 管理 Kubernetes 应用程序*，重点介绍了如何安装和使用 Helm 简化 Kubernetes 部署体验。本章还涉及 Helm 图表的细节、架构以及常见的使用场景。

*第六章*，*在 EKS 上保护和访问集群*，深入探讨了 Kubernetes 中身份验证和授权的基本方面，以及它们如何应用于 EKS。本章解释了配置客户端工具和安全访问 EKS 集群的重要性。

*第七章*，*EKS 中的网络*，解释了 Kubernetes 网络，并演示了如何将 EKS 与 AWS **虚拟私有云** (**VPC**) 无缝集成。

*第八章*，*管理 EKS 中的工作节点*，探讨了 EKS 工作节点的配置和有效管理。强调了使用 EKS 优化镜像（AMIs）和托管节点组的好处，并提供了它们相对于自管理替代方案的优势。

*第九章*，*EKS 中的高级网络配置*，深入探讨了 EKS 中的高级网络场景。内容涵盖了管理 Pod IP 地址（使用 IPv6）、实施网络策略以控制流量，以及利用复杂的基于网络的信息系统（如 Multus CNI）等话题。

*第十章*，*升级 EKS 集群*，重点介绍了升级 EKS 集群以利用新特性并确保持续支持的策略。提供了关于关键领域的指导，包括控制平面、关键组件、节点组的就地升级和蓝绿升级，以及将工作负载迁移到新集群的内容。

*第十一章*，*构建应用并推送到 Amazon ECR*，探讨了在 Amazon ECR 上构建和存储容器镜像以供 EKS 部署使用的过程。内容涵盖了存储库认证、推送容器镜像、利用 ECR 的高级功能以及将 ECR 集成到 EKS 集群中的话题。

*第十二章*，*使用 Amazon 存储部署 Pods*，解释了 Kubernetes 卷、**容器存储接口**（**CSI**）以及在 Kubernetes Pods 中对持久化存储的需求，并展示了如何在 EKS 上使用 EBS 和 EFS。还介绍了安装和配置 AWS CSI 驱动程序，以便在应用中使用 EBS 和 EFS 卷的详细信息。

*第十三章*，*使用 IAM 为应用授权访问权限*，讨论了 Pod 安全性，并结合了将 IAM 与容器化应用集成的场景。内容包括定义 Pod 的 IAM 权限、使用**IAM 服务账户角色**（**IRSA**）以及排查与 EKS 部署相关的 IAM 问题。

*第十四章*，*为 EKS 应用设置负载均衡*，探讨了 EKS 应用负载均衡的概念。还扩展了可扩展性和弹性的话题，并提供了关于 AWS 中可用的**弹性负载均衡器**（**ELB**）选项的见解。

*第十五章*，*使用 AWS Fargate*，介绍了 AWS Fargate 作为在 EKS 中托管 Pods 的无服务器替代方案。分析了使用 Fargate 的好处，提供了创建 Fargate 配置文件、将 Pods 无缝部署到 Fargate 环境的指导，以及排查可能出现的常见问题。

*第十六章*，*与服务网格一起工作*，探讨了使用服务网格技术来增强 EKS 上微服务生态系统的控制、可见性和安全性。本章涵盖了 AWS App Mesh Controller 的安装、与 Pods 的集成、利用 AWS Cloud Map 以及故障排查 Envoy 代理。

*第十七章*，*EKS 可观察性*，描述了在 EKS 部署中可观察性的重要性，并提供了关于监控、日志记录和追踪技术的见解。本章涵盖了用于监控 EKS 集群和 Pod 的原生 AWS 工具，使用 Managed Prometheus 和 Grafana 构建仪表板，利用 OpenTelemetry，以及利用机器学习功能通过 DevOps Guru 捕获集群状态。

*第十八章*，*扩展你的 EKS 集群*，讨论了 EKS 中容量规划的挑战，并探讨了用于扩展集群以满足应用需求的各种策略和工具，同时优化成本。本章介绍了如何使用 Cluster Autoscaler 和 Karpenter 扩展节点组，如何使用**水平 Pod 自动扩展器**（**HPA**）扩展应用程序，描述自定义指标的使用案例，以及如何利用 KEDA 优化事件驱动的自动扩展。

*第十九章*，*在 EKS 上开发*，探讨了如何通过不同的自动化工具和 CI/CD 实践来提高开发人员和 DevOps 工程师在构建 EKS 集群时的效率。本章重点介绍了包括 Cloud9、EKS 蓝图、Terraform、CodePipeline、CodeBuild、ArgoCD 和 GitOps 在内的各种工具，以简化工作负载部署。

*第二十章*，*排查常见问题*，提供了 EKS 故障排查检查表，并讨论了常见问题及其解决方案。

# 为了充分利用本书的内容

你将需要一个 AWS 账户和一个操作系统来运行下表中列出的应用程序。为了确保顺利阅读，建议具备基本的 AWS 概念知识，如**虚拟私有云**（**VPC**）、**弹性块存储**（**EBS**）、EC2、**弹性负载均衡器**（**ELB**）、**身份与访问管理**（**IAM**）以及 Kubernetes。

| **本书涵盖的软件** | **先决条件** |
| --- | --- |
| 亚马逊弹性 Kubernetes 服务（EKS） | 一个 AWS 账户 |
| 亚马逊命令行界面（AWS CLI） | Linux/macOS/Linux |
| kubectl | Linux/macOS/Linux |
| eksctl | Linux/macOS/Linux |
| Helm | Linux/macOS/Linux |
| Lens (Kubernetes IDE) | Linux/macOS/Linux |

本书将探索各种工具，并学习如何管理 AWS 上的 Kubernetes 集群。你可以通过以下指南找到最新版本并下载所需的软件：

+   AWS CLI: [`docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html`](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

+   kubectl: [`docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html`](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html)

+   eksctl: [`docs.aws.amazon.com/eks/latest/userguide/eksctl.html`](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)

+   Helm: [`docs.aws.amazon.com/eks/latest/userguide/helm.html`](https://docs.aws.amazon.com/eks/latest/userguide/helm.html)

+   Lens: [`k8slens.dev/`](https://k8slens.dev/)

我们尽力将本书制作成一本实用的书籍，包含了大量的代码示例。为了充分利用这本书，你应该对 AWS、Linux、YAML 和 K8s 架构有基本的了解。

# 下载彩色图像

我们还提供了一个 PDF 文件，包含了本书中使用的截图和图表的彩色版本。你可以在这里下载：[`packt.link/g2oZN`](https://packt.link/g2oZN)。

# 使用的约定

本书中使用了一些文本约定。

`文本中的代码`：表示文本中的代码词语、数据库表名、文件夹名称、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名。举个例子：“`MAINTAINER`和`CMD`命令不会生成层。”

一段代码块的格式如下：

```
apiVersion: v1 
kind: Pod 
metadata: 
  name: nginx 
spec: 
  containers:
```

所有的命令行输入或输出格式如下：

```
$ docker run hello-world
```

**粗体**：表示新术语、重要词汇，或屏幕上出现的词语。例如，菜单或对话框中的词语会以**粗体**显示。举个例子：“最常见的例子是**OverlayFS**，它包含在 Linux 内核中，并且 Docker 默认使用它。”

提示或重要说明

显示方式如下。

# 联系我们

我们总是欢迎读者的反馈。

**一般反馈**：如果你对本书的任何部分有问题，可以通过 email 联系我们，邮箱地址是 customercare@packtpub.com，并在邮件主题中提及书名。

**勘误表**：虽然我们已尽一切努力确保内容的准确性，但错误还是难免。如果你发现了本书中的错误，我们将感激你向我们报告。请访问[www.packtpub.com/support/errata](http://www.packtpub.com/support/errata)并填写表格。

**盗版**：如果你在互联网上发现我们作品的任何非法复制品，我们将非常感谢你提供其地址或网站名称。请通过 email 联系我们，邮箱是 copyright@packt.com，并附上相关材料的链接。

**如果你有兴趣成为作者**：如果你在某个领域具有专业知识，并且有兴趣写作或参与编写一本书，请访问[authors.packtpub.com](http://authors.packtpub.com)。

# 分享你的想法

一旦你阅读完《掌握 AWS 上的弹性 Kubernetes 服务（EKS）》后，我们很想听听你的想法！请点击这里，直接进入本书的 Amazon 评论页面并分享你的反馈。

你的评论对我们以及技术社区都很重要，它将帮助我们确保提供优质的内容。

# 下载本书的免费 PDF 副本

感谢购买本书！

喜欢随时随地阅读，但又无法随身携带纸质书籍吗？

您的电子书购买是否与您选择的设备不兼容？

不用担心，现在购买每本 Packt 书籍时，您都可以免费获得该书的无 DRM PDF 版本。

在任何地方、任何设备上阅读。搜索、复制并粘贴您最喜欢的技术书籍中的代码，直接应用到您的应用程序中。

优惠不仅如此，您还可以获得独家折扣、时事通讯以及每日送到您邮箱的精彩免费内容。

按照以下简单步骤即可享受福利：

1.  扫描二维码或访问下面的链接

![下载本书的免费 PDF 副本](img/B18129_QR_Free_PDF.jpg)

[`packt.link/free-ebook/9781803231211`](https://packt.link/free-ebook/9781803231211)

1.  提交您的购买凭证

1.  就这样！我们会直接将您的免费 PDF 及其他福利发送到您的电子邮件。

# 第一部分：开始使用 Amazon EKS

在本部分中，您将全面了解 Kubernetes 和容器，并深入了解 Amazon EKS 及其架构。您还将通过逐步指南，准备好您的 EKS 集群。在本部分结束时，您将学习如何在 EKS 上部署和操作应用程序的基础知识，并了解如何使用 Helm 简化 Kubernetes 应用程序的部署。

本部分包含以下章节：

+   *第一章*，*Kubernetes 和容器基础*

+   *第二章*，*介绍 Amazon EKS*

+   *第三章*，*构建您的第一个 EKS 集群*

+   *第四章*，*在 EKS 上运行您的第一个应用程序*

+   *第五章*，*使用 Helm 管理 Kubernetes 应用程序*
