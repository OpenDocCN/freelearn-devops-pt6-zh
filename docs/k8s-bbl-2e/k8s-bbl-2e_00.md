# 前言

容器的广泛采用使得虚拟化领域实现了真正的飞跃，因为它们提供了更大的灵活性，特别是在如今，像**云**、**敏捷**和**DevOps**这样的流行词汇几乎是每个人嘴边的常谈。

如今，几乎没有人质疑容器的使用——它们几乎无处不在，尤其是在 Docker 成功和 Kubernetes 作为领先容器编排平台崛起之后。

容器为组织带来了巨大的灵活性，但当组织面临将容器部署到生产环境中的挑战时，容器的使用一直备受质疑。多年来，许多公司将容器用于概念验证项目、本地开发和类似的目的，但将容器用于实际生产工作负载的想法，对于许多组织来说是不可想象的。

容器编排器是改变游戏规则的工具，Kubernetes 处于领先地位。最初由 Google 构建，今天，Kubernetes 是领先的容器编排器，提供了部署生产环境中容器所需的所有功能。Kubernetes 非常流行，但也很复杂。这个工具非常多功能，以至于入门并逐步掌握高级用法并非易事：它并不是一个容易学习和操作的工具。

作为一个编排器，Kubernetes 有其独立于容器引擎的概念。但当容器引擎与编排器结合使用时，你将得到一个非常强大的平台，能够将你的云原生应用程序部署到生产环境中。作为每天与 Kubernetes 打交道的工程师，我们和许多人一样确信，它是一项必须掌握的技术，因此我们决定分享我们的知识，涵盖大部分这个编排器的内容，让 Kubernetes 变得更易于理解。

本书完全致力于 Kubernetes，并且是我们工作成果的结晶。它为 Kubernetes 提供了一个广阔的视角，涵盖了编排器的许多方面，从纯粹的容器 Pod 创建到在公有云上部署编排器。我们不希望本书仅仅是一本*入门指南*。

我们希望本书能教会你所有关于 Kubernetes 的知识！

# 本书适合的人群

本书适合那些打算将 Kubernetes 与容器运行时结合使用的人。尽管 Kubernetes 通过**容器运行时接口**（**CRI**）支持各种容器引擎，并且并不依赖于任何特定的引擎，但 Kubernetes 与`containerd`的组合依然是最常见的使用案例之一。

本书内容高度技术化，主要关注 Kubernetes 和容器运行时，从工程师的角度进行探讨。它面向工程师，无论他们是来自开发还是系统管理员背景，并不针对项目经理。本书是那些每天使用 Kubernetes 的人的“圣经”，或者是那些希望了解这一工具的人。你不应该害怕在终端上输入一些命令。

无论你是 Kubernetes 的完全初学者，还是拥有中级水平，都不会成为学习本书的障碍。虽然我们在章节中涵盖了一些容器基础知识，但对容器有基本的技术了解会更有帮助。如果你正在将现有应用迁移到 Kubernetes，本书也可以作为一份指南。

本书包含了可以帮助读者在公共云平台上部署 Kubernetes 的内容，比如 Amazon EKS 或 Google GKE。希望在云上将 Kubernetes 添加到技术栈中的云用户将会很受益。

# 本书涵盖的内容

*第一章*，*Kubernetes 基础知识*，是对 Kubernetes 的介绍。我们将解释什么是 Kubernetes，为什么它被创造出来，谁创造了它，谁在维护这个项目，以及你在何时、为何应将它作为技术栈的一部分使用。

*第二章*，*Kubernetes 架构——从容器镜像到运行的 Pod*，讲述了 Kubernetes 是如何构建成一个分布式软件的，技术上它不是一个单一的巨型二进制文件，而是由一组微服务相互交互构建的。在这一章中，我们将解释这种架构，以及 Kubernetes 如何将你的指令转换为运行的容器。

*第三章*，*安装你的第一个 Kubernetes 集群*，解释了由于 Kubernetes 的分布式特性，它的安装非常困难，因此为了让学习过程更加容易，可以通过使用其中一个发行版来安装 Kubernetes 集群。在这一章中，我们将探索 Kind 和 minikube 这两个选项，以便让 Kubernetes 集群在你的机器上运行。

*第四章*，*在 Kubernetes 中运行你的容器*，是对 Pod 概念的介绍。

*第五章*，*使用多容器 Pod 和设计模式*，介绍了多容器 Pod 和设计模式，比如代理、适配器或 sidecar，你可以在运行多个容器作为同一 Pod 的一部分时构建这些模式。

*第六章*，*Kubernetes 中的命名空间、配额和多租户限制*，解释了使用命名空间是集群管理的关键方面，在你与 Kubernetes 一起工作的过程中，命名空间是不可避免的。尽管它是一个简单的概念，但它是一个关键概念，你必须完全掌握命名空间，才能在 Kubernetes 中取得成功。我们还将学习如何使用命名空间、配额和限制在 Kubernetes 中实现多租户。

*第七章*，*使用 ConfigMaps 和 Secrets 配置你的 Pod*，解释了在 Kubernetes 中如何将 Kubernetes 应用与其配置分开。由于 ConfigMap 和 Secret 资源，应用和配置都有各自的生命周期。本章将专门讲解这两个对象，以及如何将 ConfigMap 或 Secret 中的数据作为环境变量或挂载卷在你的 Pod 上进行挂载。

*第八章*，*通过服务暴露你的 Pods*，教你 Kubernetes 中的服务概念。每个 Pod 在 Kubernetes 中会动态分配一个 IP 地址。如果你希望为 Pods 提供一个一致的地址，暴露集群中的 Pods 给其他 Pods 或外部世界，可以使用服务，并为其分配一个静态的 DNS 名称。你将在这里学到三种主要的服务类型，分别是 ClusterIp、NodePort 和 LoadBalancer，它们都专注于在 Pod 暴露方面的单一用例。

*第九章*，*Kubernetes 中的持久存储*，讲解了默认情况下，Pods 并不是持久化的。由于它们只是管理原始容器，销毁它们将导致数据丢失。解决方案是使用持久存储，得益于 PersistentVolume 和 PersistentVolumeClaim 资源。本章将专门讲解这两个对象以及 StorageClass 对象：你将了解到 Kubernetes 在存储方面的极高灵活性，并且你的 Pods 可以与多种不同的存储技术进行对接。

*第十章*，*运行生产级 Kubernetes 工作负载*，深入探讨了 Kubernetes 中的高可用性和故障容错机制，使用 ReplicationController 和 ReplicaSet 实现。

*第十一章*，*使用 Kubernetes 部署无状态工作负载*，是上一章的延续，解释了如何使用 Deployment 对象管理多个版本的 ReplicaSets。这是 Kubernetes 上运行无状态应用的基本构建块。

*第十二章*，*StatefulSet – 部署有状态应用*，介绍了下一个重要的 Kubernetes 对象：StatefulSet。这个对象是运行有状态应用的支柱。本章将解释在 Kubernetes 中运行无状态应用和有状态应用之间的最重要区别。

*第十三章*，*DaemonSet – 在节点上维持 Pod 单例*，讲解了 DaemonSet，这是一个特殊的 Kubernetes 对象，用于在 Kubernetes 集群上运行操作性或支持性工作负载。当你需要在单个 Kubernetes 节点上运行精确一个容器 Pod 时，DaemonSet 就是你需要的对象。

*第十四章*，*使用 Helm Charts 和 Operators*，介绍了 Helm Charts，这是一种专门用于 Kubernetes 应用打包和重新分发的工具。在本章获得的知识将帮助你快速设置 Kubernetes 开发环境，甚至规划将你的 Kubernetes 应用以 Helm Chart 形式重新分发。在这一章中，我们还将介绍 Kubernetes 操作符，并讲解它们如何帮助你部署应用栈。

*第十五章*，*在 Google Kubernetes Engine 上使用 Kubernetes 集群*，讲述了如何使用本地命令行客户端和 Google Cloud 控制台将我们的 Kubernetes 工作负载迁移到 Google Cloud。

*第十六章*，*在 Amazon Web Services 上使用 Amazon Elastic Kubernetes Service 启动 Kubernetes 集群*，讨论了如何将我们在上一章中启动的工作负载迁移到 Amazon 的 Kubernetes 服务。

*第十七章*，*在 Microsoft Azure 上使用 Azure Kubernetes Service 启动 Kubernetes 集群*，讨论了如何在 Microsoft Azure 上启动一个集群。

*第十八章*，*Kubernetes 安全性*，讲解了如何使用内置的基于角色的访问控制和授权方案，以及用户管理。本章还教你如何使用准入控制器、基于 TLS 证书的通信和安全上下文的实现。

*第十九章*，*Pod 调度的高级技术*，深入探讨了节点亲和性、节点污点和容忍度，以及一般的高级调度策略。

*第二十章*，*Kubernetes Pod 和节点的自动扩展*，介绍了 Kubernetes 中自动扩展的原理，并解释了如何使用 Vertical Pod Autoscaler（垂直 Pod 自动扩展器）、Horizontal Pod Autoscaler（水平 Pod 自动扩展器）和 Cluster Autoscaler（集群自动扩展器）。

*第二十一章*，*高级 Kubernetes：流量管理*，*多集群策略及更多*，介绍了 Kubernetes 中的 Ingress 对象和 IngressController。我们解释了如何使用 nginx 作为 IngressController 的实现，以及如何在 Azure 环境中使用 Azure 应用程序网关作为本地的 IngressController。我们还将介绍 Kubernetes 的高级主题，包括集群二期任务、最佳实践以及故障排除。

# 为了最大程度地从本书中获益

虽然我们在各章中涉及了容器基础知识，但本书的重点是 Kubernetes。虽然 Kubernetes 支持多种容器引擎，本书内容主要讨论如何在 Kubernetes 中使用 `containerd` 作为运行时。你不需要成为专家，但在深入本书之前，了解如何启动和管理容器应用将会有所帮助。

虽然 Kubernetes 支持运行 Windows 容器，但本书讨论的大多数主题将基于 Linux。了解 Linux 知识会很有帮助，但并非必须。再次强调，你不需要成为专家：了解如何使用终端会话和基本的 Bash 脚本就足够了。

最后，拥有一些关于软件架构的基本知识，比如 REST API，将会非常有益。

| **本书涵盖的软件/硬件** | **操作系统要求** |
| --- | --- |
| Kubernetes >= 1.31 | Windows、macOS、Linux |
| kubectl >= 1.31 | Windows、macOS、Linux |

我们强烈建议你现在不要尝试在你的机器上安装 Kubernetes 或 `kubectl`。Kubernetes 不是一个单一的二进制文件，而是一个由多个组件组成的分布式软件，因此，从零开始安装一个完整的 Kubernetes 集群非常复杂。相反，我们建议你按照本书的第三章进行操作，该章节专门讲解了 Kubernetes 的安装设置。

**如果你正在使用本书的数字版本，我们建议你自己输入代码，或者通过 GitHub 仓库访问代码（链接将在下一节提供）。这样可以帮助你避免与复制和粘贴代码相关的潜在错误。**

请注意，`kubectl`、`helm` 等是我们在本书中最常使用的工具，但 Kubernetes 周围有一个庞大的生态系统，我们可能会安装本节未提到的额外软件。本书还涉及在云中使用 Kubernetes，我们将学习如何在像亚马逊 Web 服务（AWS）和谷歌云平台（GCP）等公共云平台上部署 Kubernetes 集群。在此设置过程中，我们可能会安装一些专门针对这些平台的软件，这些软件不仅与 Kubernetes 相关，还与这些平台提供的其他服务相关。

# 下载示例代码文件

你可以从 GitHub 下载本书的示例代码文件，地址为 [`github.com/PacktPublishing/The-Kubernetes-Bible-Second-Edition`](https://github.com/PacktPublishing/The-Kubernetes-Bible-Second-Edition)。如果代码有更新，将会在现有的 GitHub 仓库中进行更新。

我们还提供了其他来自我们丰富的书籍和视频目录的代码包，地址为 [`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)。快去看看吧！

# 下载彩色图片

我们还提供了一份 PDF 文件，其中包含本书中使用的截图/图表的彩色图像。你可以在此处下载：[`static.packt-cdn.com/downloads/9781835464717_ColorImages.pdf`](https://static.packt-cdn.com/downloads/9781835464717_ColorImages.pdf)。

# 使用的约定

本书中使用了若干文本约定。

`文中的代码`：表示文本中的代码字、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名。例如：“现在，我们需要为本地 `kubectl` CLI 创建一个 `kubeconfig` 文件。”

一段代码按如下方式编写：

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-Pod 
```

当我们希望将注意力集中到代码块的特定部分时，相关的行或项会以粗体显示：

```
apiVersion: v1
kind: **ReplicationController**
metadata:
  name: nginx-replicationcontroller-example 
```

任何命令行输入或输出都按如下方式编写：

```
$ kubectl get nodes 
```

**粗体**：表示新术语、重要词汇或屏幕上显示的词汇。例如，菜单或对话框中的词汇将在文本中以这种方式显示。以下是一个例子：“在此屏幕上，您应该看到一个**启用计费**按钮。”

**重要说明**

显示如下

**提示**

显示如下。

# 联系我们

我们始终欢迎读者的反馈。

**常规反馈**：如果您对本书的任何部分有疑问，请在邮件主题中提及书名，并通过电子邮件与我们联系 customercare@packtpub.com。

**勘误**：尽管我们已尽力确保内容的准确性，但错误难免。如果您在本书中发现任何错误，我们将非常感谢您向我们报告。请访问 [www.packtpub.com/support/errata](http://www.packtpub.com/support/errata)，选择您的书籍，点击“勘误提交表单”链接，并填写相关信息。

**盗版**：如果您在互联网上发现任何非法复制的我们的作品，感谢您向我们提供该位置或网站名称。请通过电子邮件与我们联系 copyright@packt.com，并附上该内容的链接。

**如果您有意成为作者**：如果您对某个领域有专业知识，并且有兴趣编写或为书籍做贡献，请访问 [authors.packtpub.com](http://authors.packtpub.com)。

# 留下您的评论！

感谢您从 Packt 出版社购买本书——我们希望您喜欢它！您的反馈对我们非常重要，有助于我们改进和发展。读完后，请花点时间在亚马逊上留下评论；这将只需一分钟，但对于像您这样的读者来说，意义重大。

扫描以下二维码，获得您选择的免费电子书。

![一个带黑色方块的二维码描述自动生成](img/review.png)

[`packt.link/NzOWQ`](https://packt.link/NzOWQ)

# 下载本书的免费 PDF 副本

感谢您购买本书！

你喜欢在旅途中阅读，但又无法随身携带纸质书籍吗？

您的电子书购买是否与您选择的设备不兼容？

不用担心，现在购买任何 Packt 图书，您都会免费获得该书的无 DRM 版本 PDF。

在任何地方、任何设备上随时阅读。直接将您最喜欢的技术书籍中的代码搜索、复制并粘贴到您的应用程序中。

优惠不止于此，您还可以独享折扣、新闻通讯以及每日送到邮箱的免费内容。

按照以下简单步骤，您即可享受所有优惠：

1.  扫描二维码或访问以下链接：

![](img/B22019_Free_PDF_QR.png)

[`packt.link/free-ebook/9781835464717`](https://packt.link/free-ebook/9781835464717)

1.  提交您的购买凭证。

1.  就是这样！我们将把免费的 PDF 和其他福利直接发送到您的邮箱。
