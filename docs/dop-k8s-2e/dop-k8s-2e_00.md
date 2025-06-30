# 前言

本书解释了实现 DevOps 原则的基本概念和实用技能，重点介绍容器和 Kubernetes。我们的旅程从介绍容器和 Kubernetes 的核心概念开始，接着探索 Kubernetes 提供的各种功能，如为容器持久化状态和数据、不同类型的工作负载、集群网络、集群管理和扩展。为了监督集群活动，我们在 Kubernetes 中实现了监控和日志基础设施。为了更好的可用性和效率，我们还学习了如何自动扩展容器并构建持续交付管道。最后，我们学习了如何操作来自三大主要公共云提供商的托管 Kubernetes 平台。

# 本书适用人群

本书面向具有一定软件开发经验的 DevOps 专业人员，旨在帮助他们扩展、自动化并缩短软件交付到市场的时间。

# 本书所涵盖的内容

第一章，*DevOps 入门*，带你回顾从过去到今天我们所称之为 DevOps 的演变过程，并介绍在这一领域你应该了解的工具。在过去几年中，对具备 DevOps 技能的人才需求急剧增长。DevOps 实践加速了软件开发和交付速度，并帮助了业务的敏捷性。

第二章，*容器与 DevOps*，帮助你学习容器工作的基本知识。随着向微服务的趋势日益增强，容器成为每个 DevOps 从业者的便捷且必备的工具，因为它们能以统一的方式管理异构服务，提高敏捷性。

第三章，*Kubernetes 入门*，探讨了 Kubernetes 中的关键组件和 API 对象，以及如何在 Kubernetes 集群中部署和管理容器。

第四章，*管理有状态工作负载*，描述了不同工作负载的 Pod 控制器，并介绍了用于维护应用状态的卷管理功能。

第五章，*集群管理与扩展*，引导你了解 Kubernetes 的访问控制功能，并考察了内置的准入控制器，它提供了对集群更细粒度的控制。此外，我们还将学习如何构建自己的自定义资源，以通过自定义功能扩展集群。

第六章，*Kubernetes 网络*，解释了 Kubernetes 中默认的网络和路由规则如何工作。我们还将学习如何暴露 HTTP 和 HTTPS 路由以供外部访问。在本章的最后，还介绍了网络策略和服务网格功能，以提高弹性。

第七章，*监控与日志记录*，向你展示如何使用 Prometheus 在应用程序、容器和节点级别监控资源的使用情况。本章还将展示如何通过 Elasticsearch、Fluent-bit/Fluentd 和 Kibana 堆栈收集来自应用程序、服务网格和 Kubernetes 的日志。确保服务正常运行并保持健康是 DevOps 的重要职责之一。

第八章，*资源管理与扩展*，描述了如何利用 Kubernetes 的核心——调度器，动态扩展应用程序，从而高效地利用集群的资源。

第九章，*持续交付*，解释了如何使用 GitHub/DockerHub/TravisCI 构建持续交付流水线。它还解释了如何管理更新、消除滚动更新时的潜在影响，并防止可能的失败。持续交付是一种加速市场时间的方法。

第十章，*AWS 上的 Kubernetes*，带你了解 AWS 的各个组件，并解释如何通过 AWS 托管的 Kubernetes 服务——EKS 来创建集群。EKS 与现有的 AWS 服务有着深度的集成。在本章中，我们将学习如何利用这些功能。

第十一章，*GCP 上的 Kubernetes*，帮助你学习 GCP 的概念以及如何在 GCP 的 Kubernetes 服务——**Google Kubernetes Engine**（**GKE**）中运行你的应用程序。GKE 对 Kubernetes 提供了最原生的支持。在本章中，我们将学习如何管理 GKE。

第十二章，*Azure 上的 Kubernetes*，描述了基本的 Azure 组件，如 Azure 虚拟网络、Azure 虚拟机、磁盘存储选项等。我们还将学习如何通过 Azure Kubernetes Service 来创建和运行 Kubernetes 集群。

# 为了从本书中获得最大的收益

本书将指导你通过 Docker 容器和 Kubernetes 在 macOS 和公共云服务（AWS、GCP 和 Azure）上进行软件开发和交付的方法。你需要安装 minikube、AWS CLI、Cloud SDK 和 Azure CLI 来运行本书中的代码示例。

# 下载示例代码文件

你可以从你在[www.packt.com](http://www.packt.com)的账户中下载本书的示例代码文件。如果你是在其他地方购买的本书，可以访问[www.packt.com/support](http://www.packt.com/support)并注册，以便将文件直接通过邮件发送给你。

你可以按照以下步骤下载代码文件：

1.  在[www.packt.com](http://www.packt.com)登录或注册。

1.  选择 SUPPORT 标签。

1.  点击代码下载和勘误表。

1.  在搜索框中输入书名，并按照屏幕上的指示操作。

下载文件后，请确保使用最新版本的工具解压或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

书籍的代码包也托管在 GitHub 上，地址为 [`github.com/PacktPublishing/DevOps-with-Kubernetes-Second-Edition`](https://github.com/PacktPublishing/DevOps-with-Kubernetes-Second-Edition)。如果代码有更新，它将会更新在现有的 GitHub 仓库中。

我们还提供来自我们丰富书籍和视频目录的其他代码包，您可以访问**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**查看。快去看看吧！

# 下载彩色图片

我们还提供了一个 PDF 文件，其中包含本书中使用的截图/图表的彩色图片。您可以在此下载：[`www.packtpub.com/sites/default/files/downloads/9781789533996_ColorImages.pdf`](http://www.packtpub.com/sites/default/files/downloads/9781789533996_ColorImages.pdf)。

# 使用的约定

本书中使用了多种文本约定。

`CodeInText`：表示文本中的代码词汇、数据库表名、文件夹名称、文件名、文件扩展名、路径名、虚拟 URL、用户输入以及 Twitter 用户名。例如：“鉴于此，`cgroups` 在此处用于限制资源使用。”

代码块的呈现方式如下：

```
ENV key value
 ENV key1=value1 key2=value2 ...
```

当我们希望引起你对某个代码块中特定部分的注意时，相关的行或项目会使用**粗体**表示：

```
metadata:
  name: nginx-external
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
```

任何命令行输入或输出如下所示：

```
$ docker run -p 80:5000 busybox /bin/sh -c \
  "while :; do echo -e 'HTTP/1.1 200 OK\n\ngood day'|nc -lp 5000; done"
$ curl localhost
good day
```

**粗体**：表示一个新术语、重要单词或屏幕上显示的单词。例如，菜单或对话框中的单词会像这样出现在文本中。这里有一个例子：“点击创建后，控制台将带我们进入以下视图以供探索。”

警告或重要提示如下所示。

小贴士和技巧以这种方式呈现。

# 联系我们

我们非常欢迎读者的反馈。

**一般反馈**：如果您对本书的任何部分有疑问，请在邮件主题中注明书名，并通过`customercare@packtpub.com`联系我们。

**勘误**：尽管我们已尽力确保内容的准确性，但错误仍然会发生。如果您在本书中发现错误，我们将不胜感激，若能向我们报告。请访问 [www.packt.com/submit-errata](http://www.packt.com/submit-errata)，选择您的书籍，点击“勘误提交表格”链接，并输入相关细节。

**盗版**：如果你在互联网上发现任何形式的非法复制品，请将其地址或网站名称提供给我们。请通过`copyright@packt.com`联系我们，并提供该材料的链接。

**如果你有兴趣成为作者**：如果你在某个主题领域拥有专业知识，并且有兴趣编写或参与书籍创作，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 评价

请留下评论。在您阅读并使用本书后，为什么不在您购买书籍的网站上留下评论呢？潜在的读者可以通过您的客观意见来做出购买决策，我们在 Packt 可以了解您对我们产品的看法，而我们的作者也可以看到您对他们书籍的反馈。谢谢！

有关 Packt 的更多信息，请访问 [packt.com](http://www.packt.com/)。
