# 前言

Kubernetes 迄今为止是最流行的容器编排工具，但管理该工具的复杂性促使了完全托管的 Kubernetes 服务在过去几年中的兴起。**认证 Kubernetes 管理员**（**CKA**）认证旨在确保认证候选人具备帮助他们在就业市场上建立信誉和价值所需的技能和知识，从而支持业务增长。

本书将从介绍 Kubernetes 架构和 Kubernetes 的核心概念开始，然后我们将深入探讨主要的 Kubernetes 原语，结合安装和配置、集群管理和工作负载调度、网络和安全等实际场景进行学习。此外，我们还将讨论如何在日常实践中排查 Kubernetes 的故障。

到本书结束时，你将熟练掌握 Kubernetes 的安装与配置，能够轻松应对集群管理、存储、网络、安全配置和故障排查等技能，特别是在原生 Kubernetes 环境下。

如果你想了解更多关于 Kubernetes 的信息，可以查看这个播放列表 Kubernetes in 30 days - [`www.youtube.com/watch?v=csPu6y6A7oY&list=PLyDI9q8xNovlhCqRhouXmSKQ-PP6_SsIQ`](https://www.youtube.com/watch?v=csPu6y6A7oY&list=PLyDI9q8xNovlhCqRhouXmSKQ-PP6_SsIQ)

# 本书适合的人群

本书面向希望通过 CKA 考试认证 Kubernetes 管理员技能的应用开发人员、DevOps 工程师、数据工程师和云架构师。建议具备一定的 Kubernetes 基础知识。

# 本书的内容

*第一章*，*Kubernetes 概述*，介绍了 Kubernetes 的架构及其核心概念，深入探讨了常见的 Kubernetes 工具，并通过实际操作展示了 Kubernetes 的不同发行版和生态系统的全貌。

*第二章*，*安装和配置 Kubernetes 集群*，介绍了 Kubernetes 的不同配置，并通过设置一个单节点和多节点 Kubernetes 集群，使用合适的工具进行实践。

*第三章*，*维护 Kubernetes 集群*，介绍了在维护 Kubernetes 集群时的不同方法，并实际操作进行 Kubernetes 集群的升级，备份和恢复 ETCD。本章涵盖了 CKA 考试内容的 25%。

*第四章*，*应用调度与生命周期管理*，描述了如何使用 Kubernetes 部署来部署 pods，扩展 pods，执行滚动更新和回滚，进行资源管理，以及使用 ConfigMaps 配置 pods。本章涵盖了 CKA 考试内容的 15%。

*第五章*，*解密 Kubernetes 存储*，讨论了 Kubernetes 存储的核心概念，针对有状态的工作负载，并展示了如何配置带有挂载存储和动态持久存储的应用程序。本章涵盖了 CKA 考试内容的 10%。

*第六章*，*保障 Kubernetes 安全性*，介绍了 Kubernetes 身份验证和授权模式的工作原理，然后深入讲解 Kubernetes 的**基于角色的访问控制**（**RBAC**）。从那里，我们将管理在 Kubernetes 上部署的应用程序的安全性放入视角。此部分内容不到 CKA 考试内容的 5%。

*第七章*，*解密 Kubernetes 网络*，描述了如何使用 Kubernetes 网络模型和核心概念，以及如何在集群节点上配置 Kubernetes 网络和网络策略，配置 Ingress 控制器和 Ingress 资源，配置和利用 CoreDNS，以及如何选择合适的容器网络接口插件。本章涵盖了 CKA 考试内容的 20%。

*第八章*，*监控和记录 Kubernetes 集群与应用程序*，描述了如何监控 Kubernetes 集群组件和应用程序，以及如何获取基础设施级、系统级和应用程序级的日志，以作为日志分析的来源或进一步故障排除的依据。结合接下来的两章关于故障排除集群组件和应用程序以及故障排除 Kubernetes 安全性和网络设置的内容，它覆盖了 CKA 考试内容的 30%。

*第九章*，*故障排除集群组件和应用程序*，描述了通用的故障排除方法，以及如何排除由于集群组件故障或应用程序部署过程中发生的问题所引起的错误。

*第十章*，*故障排除安全性和网络设置*，紧接着 *第九章*，提供了排除由于 RBAC 限制或网络设置引起的错误的通用故障排除方法。在 *第六章* 中，我们提到过如何启用 Kubernetes RBAC 并配置 Kubernetes DNS。在深入本章之前，一定要回顾这些重要的概念。

# 为了充分利用本书

本书是一本全面的实践学习指南，重点提供与场景相关的实际操作技能，同时提供核心知识帮助读者进行预热。本书涉及的软件和硬件如下：

| **本书涉及的软件/硬件** | **操作系统要求** |
| --- | --- |
| Minikube | Windows、macOS 或 Linux |
| kubectl, kubeadm | Windows 或 Linux |
| Docker Desktop | Windows 10 或 11 |
| WSL 2 | Windows 10 或 11 |

# 下载彩色图像

我们还提供了一份包含本书所用截图和图表彩色图片的 PDF 文件。您可以在此下载：[`packt.link/AKr3r`](https://packt.link/AKr3r)。

# 使用的约定

本书中使用了若干文本约定。

`文本中的代码`：表示文本中的代码字、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 账号。例如：“您可以通过使用`alias k=kubectl`命令为 kubectl 设置别名，然后使用`k get`命令。”

代码块如下所示：

```
apiVersion: v1
kind: Pod
metadata:
   name: melon-serviceaccount-pod
spec:
   serviceAccountName: melon-serviceaccount
   containers:
   - name: melonapp-svcaccount-container
     image: busybox
     command: ['sh', '-c','echo stay tuned!&& sleep 3600']
```

当我们希望引起你对代码块中特定部分的注意时，相关的行或项目会使用**粗体**显示：

```
spec:
   serviceAccountName: melon-serviceaccount
   containers:
```

任何命令行输入或输出都按如下方式编写：

```
kubectl delete samelon-serviceaccount
```

**粗体**：表示一个新术语、一个重要单词，或您在屏幕上看到的单词。例如，菜单或对话框中的词语通常以**粗体**显示。以下是一个示例：“此命令将返回当前显示为**未分配**的节点。”

提示或重要说明

以这种方式显示。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果您对本书的任何方面有疑问，请通过电子邮件联系我们：customercare@packtpub.com，并在邮件主题中注明书名。

**勘误**：尽管我们已经尽力确保内容的准确性，但错误还是难免发生。如果您在本书中发现错误，我们将非常感激您向我们报告。请访问 [www.packtpub.com/support/errata](http://www.packtpub.com/support/errata) 填写表单。

**盗版**：如果您在互联网上发现任何非法复制的我们的作品，我们将非常感激您提供位置地址或网站名称。请通过 copyright@packt.com 联系我们，并提供该材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且有意撰写或贡献书籍，请访问 [authors.packtpub.com](https://authors.packtpub.com)。

# 分享您的想法

阅读完*《认证 Kubernetes 管理员（CKA）考试指南》*后，我们很想听听您的想法！请[点击这里直接前往亚马逊评论页面](https://packt.link/r/1803238267)，分享您的反馈。

您的反馈对我们和技术社区都非常重要，帮助我们确保提供优质的内容。

# 第一部分：集群架构、安装与配置

本部分将介绍 Kubernetes 的概述，并探讨其概念和工具。此外，您将学习如何安装和设置 Kubernetes 集群。本部分涵盖 CKA 考试内容的 25%。

本书的这一部分包括以下章节：

+   *第一章**，Kubernetes 概述*

+   *第二章**，安装与配置 Kubernetes 集群*

+   *第三章**，维护 Kubernetes 集群*
