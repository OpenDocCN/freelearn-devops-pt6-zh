# 序言

本书的创意来源于我的一位客户，他希望在其资源受限的边缘设备上实现一个最小化的容器编排引擎。部署完整的 Kubernetes 解决方案并不是最佳选择，但随后我发现了最小化的 Kubernetes 发行版，并经过多次实验后，选择了 MicroK8s 成功地为他们构建了多个边缘计算的用例和场景。

*Canonical 的 MicroK8s* Kubernetes 发行版小巧、轻量且完全符合标准。它是一个简约型的发行版，专注于性能和简易性。由于其小巧的体积，MicroK8s 可以轻松部署在物联网和边缘设备中。本书结束时，你将了解如何使用 MicroK8s 有效地实施以下边缘计算用例和场景：

+   启动并运行你的 Kubernetes 集群

+   启用核心 Kubernetes 插件，如**域名系统**（**DNS**）和仪表盘

+   在多节点 Kubernetes 集群上创建、扩展和执行滚动更新

+   使用各种容器网络选项，如 Calico、Flannel 和 Cilium

+   设置 MetalLB 和 Ingress 选项以实现负载均衡

+   使用 OpenEBS 存储复制实现有状态应用

+   配置 Kubeflow 并运行 AI/ML 用例

+   配置与 Istio 和 Linkerd 的服务网格集成

+   使用 Knative 和 OpenFaaS 运行无服务器应用

+   配置日志记录和监控选项（Prometheus、Grafana、Elastic、Fluentd 和 Kibana）

+   配置一个多节点、高可用性的 Kubernetes 集群

+   配置 Kata 实现安全容器

+   配置严格的隔离模式运行

根据*Canonical 2022 年 Kubernetes 和云原生操作报告*（[`juju.is/cloud-native-kubernetes-usage-report-2022`](https://juju.is/cloud-native-kubernetes-usage-report-2022)），48%的受访者表示，迁移到或使用 Kubernetes 和容器的最大障碍是缺乏内部能力和有限的人员。

如报告所示，存在技能缺口和知识空白，我相信本书将通过涵盖解决这些关键领域，帮助你在短时间内迅速掌握所需知识。

# 本书适合谁阅读

本书面向 DevOps 和云工程师、Kubernetes **站点可靠性工程师**（**SREs**）以及希望实现高效部署软件解决方案的应用开发人员。它对技术架构师和技术领导者也非常有用，尤其是那些希望采用云原生技术的人。对容器化应用设计与开发、虚拟机、网络、数据库以及编程有基本了解，将有助于你更好地从本书中受益。

# 本书内容

*第一章*，*Kubernetes 入门*，介绍了 Kubernetes 及其各个组件，以及相关的抽象概念。

*第二章*，*介绍 MicroK8s*，介绍了 MicroK8s 并展示了如何安装它，如何验证其安装状态，以及如何监控和管理 Kubernetes 集群。我们还将学习如何使用一些附加组件并部署一个示例应用程序。

*第三章*，*物联网和边缘计算基础*，深入探讨了 Kubernetes、边缘计算和云计算如何协同工作，推动智能商业决策。本章概述了**物联网**（**IoT**）、边缘计算及其相互关系，以及边缘计算的优势。

*第四章*，*为物联网和边缘计算处理 Kubernetes 平台*，分析了 Kubernetes 如何为边缘计算提供有力的价值主张，以及展示如何将 Kubernetes 用于边缘工作负载的不同架构方法，同时支持满足企业应用需求的架构——低延迟、资源受限、数据隐私和带宽可扩展性。

*第五章*，*创建和实现更新的多节点 Raspberry Pi Kubernetes 集群*，探讨了如何设置一个 MicroK8s Raspberry Pi 多节点集群，部署一个示例应用程序，并执行对已部署应用的滚动更新。我们还将了解如何扩展已部署的应用程序。我们还将讨论一些构建可扩展、安全和高度优化的 Kubernetes 集群模型的推荐实践。

*第六章*，*配置容器的连接性*，探讨了在 Kubernetes 集群中如何处理网络连接。此外，我们将了解如何使用 Calico、Cilium 和 Flannel CNI 插件来为集群配置网络。我们还将讨论选择 CNI 服务时需要考虑的最重要因素。

*第七章*，*设置 MetalLB 和 Ingress 进行负载均衡*，深入研究了如何使用 MetalLB 和 Ingress 技术将服务暴露到集群外部。

*第八章*，*监控基础设施和应用程序的健康状况*，探讨了监控、日志记录和警报的各种选择，并提供了如何配置它们的详细步骤。我们还将了解需要关注的关键指标，以成功管理您的基础设施和应用程序。

*第九章*，*使用 Kubeflow 运行 AI/MLOps 工作负载*，讲解了如何使用 Kubeflow ML 平台开发和部署一个示例 ML 模型。我们还将讨论一些在 Kubernetes 上运行 AI/ML 工作负载的最佳实践。

*第十章*，*使用 Knative 和 OpenFaaS 框架实现无服务器架构*，审视了 MicroK8s 中包含的两个最流行的无服务器框架 Knative 和 OpenFaaS，这两个基于 Kubernetes 的平台用于开发、部署和管理现代无服务器工作负载。我们还将讨论开发和部署无服务器应用程序的最佳实践。

*第十一章*，*使用 OpenEBS 管理存储复制*，介绍了如何使用 OpenEBS 实现存储复制，确保数据在多个节点之间同步。我们将详细介绍配置和实施一个使用 OpenEBS Jiva 存储引擎的 PostgreSQL 状态应用程序的步骤。我们还将探讨 Kubernetes 存储的最佳实践以及数据引擎的建议。

*第十二章*，*为横切关注点实现服务网格*，带您完成部署 Istio 和 Linkerd 服务网格的步骤。您还将学习如何部署和运行示例应用程序，并了解如何配置和访问仪表盘。

*第十三章*，*通过 HA 集群进行组件故障恢复*，向您介绍设置一个高可用集群的步骤，该集群能够承受组件故障并继续为工作负载提供服务。我们还将讨论在生产就绪集群上实施 Kubernetes 应用程序的一些最佳实践。

*第十四章*，*利用硬件虚拟化确保容器安全*，探讨了如何使用 Kata Containers（一种安全的容器运行时）通过利用硬件虚拟化技术来提供更强的工作负载隔离。我们还讨论了在生产级集群上建立容器安全性的最佳实践。

*第十五章*，*为隔离容器实现严格的限制*，向您展示如何安装带有严格限制选项的 MicroK8s snap，监控安装进度，并管理在 Ubuntu Core 上运行的 Kubernetes 集群。我们还将部署一个示例应用程序，并检查该应用程序是否能够在启用严格限制的 Kubernetes 集群上运行。

*第十六章*，*深入未来*，探讨了 Kubernetes 和 MicroK8s 如何独特地为加速物联网（IoT）和边缘计算部署提供支持，以及塑造我们新未来的关键趋势。

*关于 MicroK8s 的常见问题*

# 为了充分利用本书

对基于容器的应用程序设计和开发、虚拟机、网络、数据库和编程的基本了解，将有助于您最大限度地利用本书*。* 以下是构建 MicroK8s Kubernetes 集群的前提条件：

| **本书涵盖的软件/硬件** | **操作系统要求** |
| --- | --- |

|

+   一张 microSD 卡（最低 4GB，推荐 8GB）

+   一台带有 microSD 卡驱动器的计算机

+   一台 Raspberry Pi 2、3 或 4（3 个或更多）

+   一根 micro-USB 电源线（Pi 4 需要 USB-C）

+   一个 Wi-Fi 网络或一根带有互联网连接的以太网线

+   （可选）一台带 HDMI 接口的显示器

+   （可选）适用于 Pi 2 和 3 的 HDMI 线和适用于 Pi 4 的 micro-HDMI 线

+   （可选）一台 USB 键盘

+   一个 SSH 客户端，例如 PuTTY

+   一个虚拟化管理程序，例如 Oracle VM VirtualBox 6.1，用于创建虚拟机

| 运行 Ubuntu 虚拟机的 Windows 或 Linux |
| --- |

**如果您使用的是本书的数字版本，我们建议您自己输入代码，或从书中获取代码。这样做将帮助您避免因复制和粘贴代码而导致的潜在错误。**

# 下载示例代码文件

您可以从 GitHub 下载本书的示例代码文件：[`github.com/PacktPublishing/IoT-Edge-Computing-with-MicroK8s`](https://github.com/PacktPublishing/IoT-Edge-Computing-with-MicroK8s)。如果代码有更新，它将会在 GitHub 仓库中更新。

我们还有其他代码包，来自我们丰富的书籍和视频目录，您可以在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)查看。快来看看吧！

# 下载彩色图片

我们还提供了一份包含本书中使用的截图和图表的彩色图片的 PDF 文件。你可以在这里下载：[`packt.link/HprZX`](https://packt.link/HprZX)。

# 使用的约定

本书中使用了多种文本约定。

`文本中的代码`：表示文本中的代码词语、数据库表名、文件夹名称、文件名、文件扩展名、路径名、虚拟网址、用户输入以及 Twitter 用户名。例如：“要查看可用的已安装插件列表，请使用`status`命令。”

一块代码设置如下：

```
apiVersion: v1
kind: Service
metadata:
  name: metallb-load-balancer
spec:
  selector:
    app: whoami
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
```

所有的命令行输入或输出如下所示：

```
kubectl apply -f loadbalancer.yaml
```

**粗体**：表示新术语、重要词汇或屏幕上显示的词语。例如，菜单或对话框中的词语以**粗体**显示。举个例子：“在 Kubernetes 仪表盘中的**命名空间**下，导航到**监控**，然后点击**服务**。”

提示或重要说明

显示如下：

# 联系我们

我们欢迎读者提供反馈。

**一般反馈**：如果您对本书的任何内容有疑问，请通过电子邮件联系我们，邮箱地址是 customercare@packtpub.com，并在邮件主题中注明书名。

**勘误表**：虽然我们已尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在本书中发现错误，欢迎向我们报告。请访问[www.packtpub.com/support/errata](http://www.packtpub.com/support/errata)并填写表单。

**盗版问题**：如果您在互联网上发现我们任何形式的作品的非法副本，请您提供位置地址或网站名称。请通过发送链接到 copyright@packt.com 与我们联系。

**如果你对成为作者感兴趣**：如果你有专业知识，并且对撰写或贡献书籍感兴趣，请访问[authors.packtpub.com](http://authors.packtpub.com)。

# 分享您的想法

一旦您阅读了*使用 MicroK8s 进行 IoT 边缘计算*，我们希望听到您的想法！请[点击此处直达亚马逊评论页面](https://packt.link/r/1803230630)来为本书分享您的反馈。

您的评论对我们和技术社区至关重要，并将帮助我们确保提供卓越的内容质量。

# 第一部分：Kubernetes 和 MicroK8s 的基础知识

在这部分中，您将介绍 MicroK8s 及其生态系统。您还将学习如何安装 MicroK8s Kubernetes 集群并使其运行起来。

本书的这部分包括以下章节：

+   *第一章*，*开始学习 Kubernetes*

+   *第二章*，*介绍 MicroK8s*
