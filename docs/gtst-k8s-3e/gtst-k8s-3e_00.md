# 前言

本书是一个关于开始使用 Kubernetes 和整体容器管理的指南。我们将向您展示 Kubernetes 的特性和功能，以及它如何融入整体运营策略。您将了解在将容器从开发人员的笔记本转移到更大规模上进行管理时遇到的难题。您还将看到 Kubernetes 如何成为帮助您充满信心地面对这些挑战的完美工具。

# 本书适合谁

无论您是专注于开发，还是深耕运维，抑或是以执行官身份展望未来，Kubernetes 和本书都适合您。*《Getting Started with Kubernetes》*将帮助您了解如何将容器应用程序推向生产环境，并结合最佳实践和逐步操作指南，将其与现实世界的运营策略联系起来。您将了解 Kubernetes 如何融入您的日常运营中，这可以帮助您为生产准备好容器应用程序堆栈。

对 Docker 容器，一般的软件开发和高级操作有一些了解将有所帮助。

# 要充分利用本书

本书将涵盖下载和运行 Kubernetes 项目。如果您在 Windows 上，可以使用 Linux 系统（VirtualBox 可行），并且对命令行有一定的熟悉度。

此外，您应该拥有 Google Cloud Platform 账户。您可以在此处免费试用：[`cloud.google.com/`](https://cloud.google.com/)。

此外，本书的某些部分需要 AWS 账户。您可以在此处免费试用：[`aws.amazon.com/`](https://aws.amazon.com/)。

# 下载示例代码文件

您可以从您的账户在[www.packt.com](http://www.packt.com)下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[www.packt.com/support](http://www.packt.com/support)并注册，文件将通过电子邮件直接发送给您。

您可以按照以下步骤下载代码文件：

1.  登录或注册在[www.packt.com](http://www.packt.com)。

1.  选择“支持”选项卡。

1.  单击“代码下载与勘误”。

1.  在搜索框中输入书名，按照屏幕上的说明操作。

下载文件后，请确保使用最新版本的解压缩或提取文件夹：

+   适用于 Windows 的 WinRAR/7-Zip

+   适用于 Mac 的 Zipeg/iZip/UnRarX

+   适用于 Linux 的 7-Zip/PeaZip

本书的代码捆绑包也托管在 GitHub 上：[`github.com/PacktPublishing/Getting-Started-with-Kubernetes-third-edition`](https://github.com/PacktPublishing/Getting-Started-with-Kubernetes-third-edition)。如果代码有更新，将在现有的 GitHub 仓库中更新。

我们还有来自我们丰富书籍和视频目录的其他代码捆绑包可供查看！请访问**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**。

# 下载彩色图片

我们还提供了一个 PDF 文件，里面包含了本书中使用的截图/图表的彩色版本。您可以在此下载：`www.packtpub.com/sites/default/files/downloads/9781788994729_ColorImages.pdf`。

# 使用的约定

本书中使用了多种文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟网址、用户输入以及 Twitter 用户名。例如：“Master 节点的最后两个主要部分是 `kube-controller-manager` 和 `cloud-controller-manager`。”

一段代码会如下设置：

```
"conditions": [
  {
    "type": "Ready",
    "status": "True"
  }
```

当我们希望引起您对代码块中特定部分的注意时，相关的行或项会以粗体显示：

```
"conditions": [
  {
    "type": "Ready",
    "status": "True"
  }
```

任何命令行输入或输出如下所示：

```
$ kubectl describe pods/node-js-pod
```

**粗体**：表示新术语、重要词汇或您在屏幕上看到的词汇。例如，菜单或对话框中的词汇会这样出现在文本中。比如：“点击 Jobs，然后从列表中选择 long-task，这样我们就可以看到详细信息。”

警告或重要提示以这种形式呈现。

小贴士和技巧如下所示。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果您对本书的任何部分有疑问，请在邮件主题中注明书名，并通过 `customercare@packtpub.com` 给我们发送邮件。

**勘误**：虽然我们已尽力确保内容的准确性，但仍可能会出现错误。如果您在本书中发现任何错误，我们将不胜感激，若您能将其报告给我们。请访问 [www.packt.com/submit-errata](http://www.packt.com/submit-errata)，选择您的书籍，点击“勘误提交表单”链接，并输入相关细节。

**盗版**：如果您在互联网上发现我们作品的任何非法复制品，恳请您提供该材料的地址或网站名称。请通过 `copyright@packt.com` 联系我们，并附上链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且有兴趣写书或为书籍做贡献，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评价。当您阅读并使用了本书后，何不在您购买本书的网站上留下评价？潜在的读者可以查看您的客观意见以做出购买决定，而我们在 Packt 也可以了解您对我们产品的看法，我们的作者也能看到您对其书籍的反馈。谢谢！

欲了解更多关于 Packt 的信息，请访问 [packt.com](http://www.packt.com/)。
