# 前言

随着近年来微服务架构的流行，单体应用被重构为多个微服务。容器简化了由微服务构建的应用的部署。容器管理、自动化和编排已成为关键问题。Kubernetes 正是为了解决这些问题而诞生的。

本书是一本实用指南，提供了逐步的技巧和示例，帮助你在私有云和公有云中构建和运行自己的 Kubernetes 集群。跟随本书的内容，你将理解如何在 Kubernetes 中部署和管理你的应用和服务。你还将深入理解如何扩展和更新实时容器，以及如何在 Kubernetes 中进行端口转发和网络路由。你将通过本书的动手示例学习如何构建一个强大的高可用集群。最后，你将通过集成 Jenkins、Docker 注册表和 Kubernetes，构建一个持续交付管道。

# 本书适用对象

如果你已经使用 Docker 容器一段时间，并且想要以现代的方式管理你的容器，那么本书是你的最佳选择。本书适合那些已经理解 Docker 和容器技术，并希望进一步探索以寻找更好的方式来编排、管理和部署容器的人。本书非常适合那些希望超越单一容器并与容器集群一起工作的读者，学习如何构建自己的 Kubernetes，并使其与持续交付管道无缝对接。

# 本书涵盖内容

第一章，*构建你自己的 Kubernetes 集群*，讲解了如何使用各种部署工具构建自己的 Kubernetes 集群，并在其上运行你的第一个容器。

第二章，*深入理解 Kubernetes 概念*，涵盖了我们需要了解的 Kubernetes 的基础和高级概念。接下来，你将学习如何通过编写和应用配置文件，将这些概念结合起来创建 Kubernetes 对象。

第三章，*玩转容器*，解释了如何扩展容器的规模，并执行滚动更新而不影响应用的可用性。此外，你还将学习如何部署容器来应对不同的应用负载。它还将带你了解配置文件的最佳实践。

第四章，*构建高可用集群*，提供了如何构建高可用的 Kubernetes master 和 etcd 的相关信息。这将防止 Kubernetes 组件成为单点故障。

第五章，*构建持续交付管道*，讨论了如何将 Kubernetes 集成到现有的持续交付管道中，使用 Jenkins 和私有 Docker 注册表。

第六章，*在 AWS 上构建 Kubernetes*，带你了解 AWS 基础知识。你将学习如何在几分钟内在 AWS 上构建一个 Kubernetes 集群。

第七章，*在 GCP 上构建 Kubernetes*，带你进入 Google Cloud Platform 的世界。你将学习 GCP 的基础知识，并了解如何只需几次点击就能启动一个托管的、生产就绪的 Kubernetes 集群。

第八章，*高级集群管理*，讨论了 Kubernetes 中的重要资源管理。本章还讲解了其他重要的集群管理内容，如 Kubernetes 仪表板、身份验证和授权。

第九章，*日志记录与监控*，解释了如何使用 Elasticsearch、Logstash 和 Kibana（ELK）在 Kubernetes 中收集系统和应用程序日志。你还将学习如何利用 Heapster、InfluxDB 和 Grafana 来监控你的 Kubernetes 集群。

# 为了充分利用本书

在整本书中，我们使用至少三台服务器，运行基于 Linux 的操作系统，来构建 Kubernetes 中的所有组件。在本书开始时，你可以使用一台机器，无论是 Linux 还是 Windows，来了解概念和基础部署。从可扩展性的角度来看，我们建议你从三台服务器开始，以便独立扩展组件并将集群推向生产级别。

# 下载示例代码文件

你可以从[www.packtpub.com](http://www.packtpub.com)的账户中下载本书的示例代码文件。如果你从其他地方购买了本书，可以访问[www.packtpub.com/support](http://www.packtpub.com/support)，注册后直接将文件通过邮件发送给你。

你可以通过以下步骤下载代码文件：

1.  登录或注册于[www.packtpub.com](http://www.packtpub.com/support)。

1.  选择*支持*标签。

1.  点击代码下载与勘误。

1.  在*搜索*框中输入书名，并按照屏幕上的指示操作。

下载文件后，请确保使用以下最新版本的工具解压或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

本书的代码包也托管在 GitHub 上，地址是[`github.com/PacktPublishing/Kubernetes-Cookbook-Second-Edition`](https://github.com/PacktPublishing/Kubernetes-Cookbook-Second-Edition)。如果代码有更新，将会更新至现有的 GitHub 仓库。

我们还从丰富的书籍和视频目录中提供其他代码包，访问**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**。快来看看吧！

# 下载彩色图片

我们还提供了一份包含本书中使用的截图/图示彩色图片的 PDF 文件。你可以在这里下载：[`www.packtpub.com/sites/default/files/downloads/KubernetesCookbookSecondEdition_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/KubernetesCookbookSecondEdition_ColorImages.pdf)。

# 使用的约定

本书中使用了若干文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名称、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 账户。这里有一个例子：“准备以下 YAML 文件，这是一个简单的 Deployment，启动两个`nginx`容器。”

代码块如下设置：

```
# cat 3-1-1_deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
```

当我们希望引起你对代码块中特定部分的注意时，相关行或项将以粗体显示：

```
Annotations:            deployment.kubernetes.io/revision=1
Selector:               env=test,project=My-Happy-Web,role=frontend
Replicas:               5 desired | 5 updated | 5 total | 5 available | 0 unavailable
StrategyType:           RollingUpdate
```

任何命令行输入或输出将按如下方式编写：

```
//install kubectl command by "kubernetes-cli" package
$ brew install kubernetes-cli
```

**粗体**：表示新术语、重要词汇，或者你在屏幕上看到的词汇。例如，菜单或对话框中的词汇会以这种方式出现在文本中。这里有一个例子：“安装很简单，所以我们可以选择默认选项并点击‘下一步’。”

警告或重要说明将以这种方式显示。

提示和技巧将以这种方式显示。

# 各节

在本书中，你会发现几个经常出现的标题（*准备工作*、*如何做...*、*如何运作...*、*还有更多...*，以及*另见*）。

为了提供清晰的完成食谱的指导，请按照以下方式使用这些部分：

# 准备工作

本节告诉你在食谱中可以期待什么，并描述了如何设置食谱所需的任何软件或初步设置。

# 如何做...

本节包含执行食谱所需的步骤。

# 如何运作...

本节通常包含对上一节内容的详细解释。

# 还有更多...

本节包含关于食谱的附加信息，帮助你对食谱有更深入的了解。

# 另见

本节提供了指向其他有用信息的链接，这些信息对食谱有帮助。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：发送电子邮件至`feedback@packtpub.com`，并在邮件主题中提及书名。如果你对本书的任何内容有疑问，请通过`questions@packtpub.com`与我们联系。

**勘误**：尽管我们已尽最大努力确保内容的准确性，但错误难免发生。如果你在本书中发现错误，请报告给我们。请访问[www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择你的书籍，点击“勘误提交表单”链接并输入详细信息。

**盗版**：如果你在互联网上发现任何我们作品的非法复制品，我们将感激你提供该位置地址或网站名称。请通过`copyright@packtpub.com`与我们联系，并附上材料的链接。

**如果你有兴趣成为作者**：如果你在某个领域有专业知识，并且有兴趣写书或为书籍做贡献，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。阅读并使用本书后，为什么不在你购买书籍的网站上留下评论呢？潜在读者可以看到并参考你的公正意见来做出购买决策，我们在 Packt 也可以了解你对我们产品的看法，而我们的作者则能看到你对他们书籍的反馈。谢谢！

欲了解更多关于 Packt 的信息，请访问 [packtpub.com](https://www.packtpub.com/)。
