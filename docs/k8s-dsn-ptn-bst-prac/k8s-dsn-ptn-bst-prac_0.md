# 前言

在深入了解 Kubernetes 工作原理之前，本书会首先向你介绍容器编排的世界，并描述应用开发中的最新变化。它帮助你理解 Kubernetes 解决的问题，并展示如何使用 Kubernetes 资源来部署应用。你还将学习如何应用 Kubernetes 集群的安全模型。本书还描述了运行在 Kubernetes 中的服务如何利用平台的安全特性。你将学习如何排查 Kubernetes 集群问题并调试 Kubernetes 应用。你还将学习分析 Kubernetes 中的网络模型及其替代方案，并将设计模式作为最佳实践应用于 Kubernetes。等你读完本书后，你将全面掌握使用 Kubernetes 管理容器的能力。

完成本书后，你将能够：

+   理解并根据云原生范式对软件设计模式进行分类

+   将设计模式作为最佳实践应用于 Kubernetes

+   使用客户端库通过编程方式访问 Kubernetes API

+   使用自定义资源和控制器扩展 Kubernetes

+   集成访问控制机制并与 Kubernetes 中的资源生命周期进行交互

+   在 Kubernetes 中开发和运行自定义调度器

# 本书适合的人群

如果你对配置和排查 Kubernetes 集群问题、以及在 Kubernetes 集群上开发基于微服务的应用感兴趣，那么这本书将对你非常有用。具备基础 Docker 知识的 DevOps 工程师将发现这本书很有价值。

# 本书内容简介

第一章，*Kubernetes 设计模式*，将帮助你理解 Kubernetes 模式，并通过 Kubernetes 本身及外部应用的示例进行讲解。

第二章，*Kubernetes 客户端库*，将帮助你了解如何通过原始 HTTP 查询到复杂的库，涵盖集群内外的访问 Kubernetes API 示例。

第三章，*Kubernetes 扩展*，将介绍 Kubernetes 的扩展功能，包括自定义资源定义、自定义控制器、动态准入控制器和自定义调度器。

# 如何最大化地利用本书

我们假设你已经熟悉命令行工具和计算机编程概念及语言。最低硬件要求为：Intel Core i7 或同等性能处理器，8GB RAM，35GB 硬盘，稳定的互联网连接。你还需要提前安装以下软件：

+   访问版本为 1.10 或更高的 Kubernetes 集群

    本地 Kubernetes 解决方案，如 minikube 或托管在云提供商中的集群：[`github.com/kubernetes/minikube`](https://github.com/kubernetes/minikube)

+   Kubernetes 命令行工具 kubectl 是通过终端访问 Kubernetes 的必要工具：

    [`kubernetes.io/docs/tasks/tools/install-kubectl/`](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

+   构建和测试客户端库需要 Docker 客户端和服务器的最低版本为 18.03：

    [`www.docker.com/get-started`](https://www.docker.com/get-started)

+   安装 Python 和 Go 并非必需，但建议在本地玩转客户端库时安装：

    +   [`www.python.org/downloads/`](https://www.python.org/downloads/)

    +   [`golang.org/`](https://golang.org/)

# 下载示例代码文件

您可以从 [www.packt.com](http://www.packtpub.com) 的帐户下载本书的示例代码文件。如果您是在其他地方购买本书，您可以访问 [www.packt.com/support](http://www.packtpub.com/support) 并注册，将文件直接通过电子邮件发送给您。

您可以按照以下步骤下载代码文件：

1.  在 [www.packt.com](http://www.packtpub.com/support) 登录或注册。

1.  选择 SUPPORT 标签页。

1.  点击“代码下载与勘误”。

1.  在搜索框中输入书名，并按照屏幕上的指示操作。

文件下载完成后，请确保使用最新版本的以下工具解压或提取文件夹：

+   适用于 Windows 的 WinRAR/7-Zip

+   适用于 Mac 的 Zipeg/iZip/UnRarX

+   适用于 Linux 的 7-Zip/PeaZip

本书的代码包也托管在 GitHub 上，地址是 [`github.com/TrainingByPackt/Kubernetes-Design-Patterns-and-Extensions`](https://github.com/TrainingByPackt/Kubernetes-Design-Patterns-and-Extensions)。如果代码有更新，将会在现有的 GitHub 仓库中更新。

我们的丰富书籍和视频目录中也有其他代码包，您可以在 **[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)** 上查看！快去看看吧！

# 使用的约定

本书中使用了若干文本约定。

`CodeInText`：表示文本中的代码字、数据库表名、文件夹名称、文件名、文件扩展名、路径名、虚拟网址、用户输入以及 Twitter 用户名。以下是一个示例：“使用这种方法，`kubectl` 会使用其自身凭证安全地连接到 API 服务器，并为本地系统上的应用程序创建代理。”

代码块如下所示：

```
{
  "apiVersion":"v1",
  "kind":"Namespace",
  "metadata":{
    "name":"packt-client”
  }
}
```

当我们希望您特别关注代码块的某部分时，相关行或项目会以粗体显示：

```
curl -X POST http://localhost:8080/api/v1/namespaces/  \
--header "Content-Type:application/json" \
```

**活动**：这些基于场景的活动将让您在完整章节学习过程中实际应用所学内容。它们通常是基于真实问题或情境。

警告或重要说明以此方式显示。

# 联系我们

我们总是欢迎读者的反馈。

**一般反馈**：请发送邮件至 `feedback@packtpub.com`，并在邮件主题中提及书名。如果您对本书的任何方面有疑问，请通过 `questions@packtpub.com` 与我们联系。

**勘误**：虽然我们已尽最大努力确保内容的准确性，但错误总会发生。如果您在本书中发现任何错误，我们将感激您向我们报告。请访问[www.packt.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击“勘误提交表格”链接并填写相关信息。

**盗版**：如果您在互联网上发现任何形式的非法复制我们作品的情况，我们将感激您提供相关位置地址或网站名称。请通过`copyright@packt.com`与我们联系，并附上相关材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域具有专长，并且有兴趣撰写或为书籍做贡献，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。一旦您阅读并使用了本书，为什么不在您购买书籍的网站上留下评论呢？潜在读者可以参考您的公正意见做出购买决策，我们 Packt 公司可以了解您对我们产品的看法，而我们的作者也能看到您对他们书籍的反馈。感谢您的支持！

想了解更多关于 Packt 的信息，请访问[packt.com](https://www.packtpub.com/)。

所有活动的解决方案都在*附录*部分中。
