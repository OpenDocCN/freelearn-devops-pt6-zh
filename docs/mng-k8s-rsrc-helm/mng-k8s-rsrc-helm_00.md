# 前言

容器化目前被认为是实现 DevOps 的最佳方式之一。尽管 Docker 引入了容器并改变了 DevOps 时代，但 Google 开发了一个广泛的容器编排系统——Kubernetes，它如今被认为是行业标准。在本书的帮助下，你将探索使用 Helm 管理 Kubernetes 上运行的应用程序的高效性。

本书首先简要介绍 Helm 及其对使用容器和 Kubernetes 的用户的影响，接着深入讲解 Helm 图表的基本概念及其整体架构和应用场景。之后，你将学习如何编写 Helm 图表，以实现 Kubernetes 上应用程序的自动化部署，并逐步深入到更高级的策略。这些面向企业的模式关注的是超越基础的概念，帮助你充分利用 Helm，包括与自动化、应用程序开发、交付、生命周期管理和安全性相关的主题。

本书结束时，你将学会如何利用 Helm 构建、部署和管理 Kubernetes 上的应用程序。

# 本书适合的读者

本书适合对 Kubernetes 开发或管理感兴趣，并希望学习 Helm 以为 Kubernetes 上的应用开发提供自动化的开发人员或管理员。虽然不需要先前的 Helm 知识，但具有 Kubernetes 应用程序开发的基本知识将会有所帮助。

# 本书内容

*第一章*，*理解 Kubernetes 和 Helm*，在这一章你将学习到部署 Kubernetes 应用程序时遇到的挑战，以及 Helm 如何简化部署过程。

*第二章*，*准备 Kubernetes 和 Helm 环境*，在这一章你将学习如何配置本地开发环境。你将下载 Minikube 和 Helm，并学习基本的 Helm 配置。

*第三章*，*使用 Helm 安装第一个应用程序*，通过让你部署第一个 Helm 图表，教授你 Helm 主要命令的使用。

*第四章*，*创建新的 Helm 图表*，讲解了 Helm 图表的结构，并帮助你创建自己的 Helm 图表。

*第五章*，*Helm 依赖管理*，在这一章你将学习如何管理和使用依赖项来构建和管理复杂的应用程序部署。

*第六章*，*理解 Helm 模板*，探讨 Helm 模板以及如何动态生成 Kubernetes 资源。

*第七章*，*Helm 生命周期钩子*，讲解生命周期钩子以及如何在不同的 Helm 生命周期阶段部署任意资源。

*第八章*，*发布 Helm 图表到 Helm 图表仓库*，讲解了 Helm 图表仓库及其如何用于发布 Helm 图表。

*第九章*，*测试 Helm 图表*，讨论了在 Helm 图表开发过程中测试 Helm 图表的不同策略。

*第十章*，*使用 CD 和 GitOps 自动化 Helm*，探讨了如何使用持续交付和 GitOps 方法自动化 Helm 部署。

*第十一章*，*在 Operator 框架中使用 Helm*，介绍了如何使用 `operator-sdk` 工具包创建一个 Helm 操作符。

*第十二章*，*Helm 安全性考虑事项*，讨论了与 Helm 发布、图表和仓库相关的不同安全话题。

# 为了充分利用本书的内容

为了充分利用本书，你应该按照以下表格中列出的技术进行安装，并跟随示例进行操作。虽然这些是写作过程中使用的版本，但最新版本也应该可以正常使用。

| **书中涵盖的软件/硬件** | **操作系统要求** |
| --- | --- |
| Minikube v1.22.0 | Windows、macOS 或 Linux |
| VirtualBox 6.1.26 | Windows、macOS 或 Linux |
| Kubectl v1.21.2 | Windows、macOS 或 Linux |
| Helm v3.6.3 | Windows、macOS 或 Linux |

**如果你使用的是本书的电子版，我们建议你自己输入代码，或者从本书的 GitHub 仓库中访问代码（下一节中会提供链接）。这样可以帮助你避免与复制粘贴代码相关的潜在错误。**

# 下载示例代码文件

你可以从 GitHub 下载本书的示例代码文件：[`github.com/PacktPublishing/Managing-Kubernetes-Resources-using-Helm`](https://github.com/PacktPublishing/Managing-Kubernetes-Resources-using-Helm)。如果代码有更新，它会在 GitHub 仓库中同步更新。

我们还提供了其他代码包，来自我们丰富的书籍和视频目录，你可以在 [`github.com/PacktPublishing/`](https://github.com/PacktPublishing/) 查找。快来看看吧！

# 下载彩色图片

我们还提供了一个 PDF 文件，包含本书中使用的截图和图表的彩色图片。你可以在这里下载：[`packt.link/zeDY0`](https://packt.link/zeDY0)。

# 使用的约定

本书中使用了多种文本约定。

`文本中的代码`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 账号名。举个例子：“注意，冒号和 `LearnHelm` 字符串之间缺少一个空格。”

一段代码块按如下方式书写：

```
configuration: |
  server.port=8443
  logging.file.path=/var/log
```

当我们希望你特别注意代码块的某部分时，相关的行或项会加粗显示：

```
$ cd ~
$ git clone <repository URI>
```

任何命令行输入或输出均按如下方式书写：

```
$ helm dependency update chapter8/guestbook
$ helm package guestbook chapter8/guestbook
```

**粗体**：表示一个新术语、一个重要的词或你在屏幕上看到的单词。例如，菜单或对话框中的词会以**粗体**显示。举个例子：“点击**生成令牌**按钮来创建令牌。”

提示或重要说明

如此呈现。

# 与我们联系

我们始终欢迎读者的反馈。

**一般反馈**：如果您对本书的任何方面有疑问，请通过电子邮件联系 [customercare@packtpub.com](https://customercare@packtpub.com)，并在邮件主题中注明书名。

**勘误**：尽管我们已尽力确保内容的准确性，但难免会有错误发生。如果您在本书中发现错误，我们将不胜感激，若您能向我们报告。请访问 [www.packtpub.com/support/errata](https://www.packtpub.com/support/errata) 并填写表格。

**盗版**：如果您在互联网上发现我们作品的任何非法复制品，我们将不胜感激，若您能提供该地址或网站名称。请通过 [copyright@packt.com](https://copyright@packt.com) 联系我们，并附上相关资料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专长，并且有兴趣写书或为书籍做贡献，请访问 [authors.packtpub.com](https://authors.packtpub.com)。

# 分享您的想法

一旦您阅读完 *使用 Helm 管理 Kubernetes 资源，第二版*，我们很想听听您的想法！请点击此处直接前往本书的亚马逊评论页面，并分享您的反馈。

您的评论对我们和技术社区非常重要，将帮助我们确保提供高质量的内容。

# 第一部分：介绍与设置

Kubernetes 是一个功能强大的系统，具有复杂的配置。在 *第一部分*，您将学习 Helm 如何通过提供包管理器界面来解决这些复杂性。通过本部分的学习，您将通过部署第一个 Helm 图表获得实践经验。

在本部分中，我们将涵盖以下主题：

+   *第一章**，理解 Kubernetes 和 Helm*

+   *第二章**，准备 Kubernetes 和 Helm 环境*

+   *第三章**，使用 Helm 安装您的第一个应用程序*
