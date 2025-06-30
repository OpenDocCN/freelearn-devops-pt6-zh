# 前言

**生成式 AI**（**GenAI**）正在革新组织构建智能系统的方式，使机器能够创建内容、代码、文本、图像等。随着大规模 AI 应用需求的不断增长，**Kubernetes**（**K8s**）已成为管理这些工作负载的事实标准平台，具有可扩展性、韧性和高效性。

本书提供了一本全面的实用指南，介绍如何在 Kubernetes 上使用 Amazon EKS 及其他开源和云原生工具来构建、部署、监控和扩展 GenAI 应用程序。从基础概念到高级 GPU 优化，本书通过实践示例、部署模式和行业最佳实践，全面讲解了 GenAI 项目的生命周期。

# 本书适合哪些人群

本书适合以下人群：

+   解决方案架构师

+   工程领导者

+   DevOps 工程师

+   GenAI 开发者

+   产品经理

+   探索 GenAI 和 Kubernetes 的学生和研究人员

你应该对云计算和 AI/ML 概念有基本的了解。不需要具备 Kubernetes 的前期经验——本书通过现实世界的示例提供了渐进式的学习曲线。

# 本书内容涵盖了什么

*第一章*，*生成式 AI 基础知识*，概述了生成式 AI（GenAI）的基础知识，涵盖了与传统 AI 的区别、机器学习从 CNNs/RNNs 到变换器（transformers）的演变，概述了 GenAI 项目的生命周期，并探索了各行各业的应用。

*第二章*，*Kubernetes 与 GenAI 的介绍与集成*，探讨了大规模运行 GenAI 工作负载的挑战，以及为什么容器化和 Kubernetes 是理想的解决方案。你将构建并在本地运行你的第一个 GenAI 容器镜像。

*第三章*，*在云中开始使用 Kubernetes*，介绍了基于云的 Kubernetes 部署选项，并提供了逐步指导，教你如何使用 Terraform 设置 Amazon EKS 并部署一个 LLM 模型。

*第四章*，*针对特定领域应用的 GenAI 模型优化*，深入探讨了如何优化通用 GenAI 模型以适应领域特定的应用，重点介绍了 **检索增强生成**（**RAG**）、微调以及 LangChain 的使用，以提高聊天机器人和个性化推荐等任务的性能和效率。

*第五章*，*在 K8s 上使用 GenAI——以聊天机器人为例*，展示了如何在 Kubernetes 上使用 Amazon EKS 部署和微调 GenAI 模型，具体示例包括设置 Jupyter Notebook 进行实验、微调 Llama 3 模型，以及部署一个基于 RAG 的聊天机器人，用于个性化电商推荐。

*第六章*，*在 Kubernetes 上扩展 GenAI 应用程序*，探讨了 Kubernetes 应用程序的各种扩展策略和最佳实践，重点关注使用 HPA、VPA、KEDA、Cluster Autoscaler 和 Karpenter 实现高效的资源利用和最佳性能。

*第七章*，*在 Kubernetes 上优化 GenAI 应用程序的成本*，探讨了在 Kubernetes 上优化 GenAI 应用程序成本的策略，重点关注计算、存储和网络。强调资源的合理配置、高效的存储管理和网络最佳实践，并介绍了如 Kubecost 和 Goldilocks 等工具，用于监控和优化资源利用。

*第八章*，*在 K8s 上部署 GenAI 的网络最佳实践*，涵盖了 CNI、Kubelet 和 CRI 等关键网络组件，叠加和原生网络方法，并深入探讨了如 Service Mesh 和 NetworkPolicy 等高级功能，确保网络性能的安全、高效和可扩展性。

*第九章*，*在 Kubernetes 上部署 GenAI 的安全最佳实践*，提供了一个全面的框架，通过深度防御方法保护 Kubernetes 上的 GenAI 应用程序，涵盖了包括供应链、主机、网络和运行时安全等关键安全领域，同时还涉及了机密管理最佳实践和 IAM。

*第十章*，*在 Kubernetes 中优化 GenAI 应用程序的 GPU 资源*，探讨了在 Kubernetes 中最大化 GPU 效率的策略，讨论了 GPU 资源管理、MIG 和 MPS 等分区技术、GPU 监控以及自动扩展解决方案，以优化性能和成本。

*第十一章*，*GenAIOps——数据管理和 GenAI 自动化流水线*，介绍了 GenAIOps，详细说明了部署、监控和优化 GenAI 模型的工具和工作流程，重点关注数据管理、隐私保护、偏见缓解和持续的模型监控。

*第十二章*，*可观察性——在 K8s 上查看 GenAI*，解释了在 Kubernetes 上监控 GenAI 应用程序时可观察性的重要性，详细说明了如何使用 Prometheus、Grafana 和 NVIDIA DCGM 构建一个强大的监控框架，以提供实时洞察和调试功能。

*第十三章*，*GenAI 应用程序的高可用性和灾难恢复*，探讨了 Kubernetes 上 GenAI 应用程序的高可用性和灾难恢复策略，详细说明了能够在故障期间实现自动扩展和持续服务的架构模式。它涵盖了冗余方法、关键指标（RPO、RTO 和 MTD）以及从备份恢复到多区域部署的实施策略。

*第十四章*，*总结：GenAI 编程助手与进一步阅读*，重点介绍了像 Amazon Q Developer、GitHub Copilot 和 Google Gemini Code Assist 等 GenAI 编程助手，在自动化 IaC、优化工作负载和管理 Kubernetes 集群方面的变革性影响。它还讨论了 AI 驱动的安全性、成本效率和可扩展性改进，并提供了进一步阅读资源，帮助掌握 Kubernetes 和 GenAI 工具。

# 为了最大限度地利用本书内容

如果你熟悉基础的编程（优先选择 Python）、云计算基础以及机器学习的概念，你将从本书中获益最多。不需要深入的 Kubernetes 经验，因为本书会详细介绍设置和配置步骤。

| **本书中涉及的** **软件/硬件** | **操作系统要求** |
| --- | --- |
| 操作系统 | Linux、macOS、Windows（通过 WSL） |
| Kubernetes | Amazon EKS、kind（用于本地测试） |
| AI/ML 框架 | Hugging Face Transformers、PyTorch、TensorFlow |
| 加速器 | NVIDIA GPUs、AWS Trainium/Inferentia |
| 可观测性 | Prometheus、Grafana、OpenTelemetry、Loki |
| 自动化 | Kubeflow、MLflow、Ray、Argo Workflows |
| 安全工具 | OPA、Kyverno |

你将需要以下内容：

+   一个具有足够配额的 AWS 账户用于 EC2 实例（尤其是 GPU 节点）

+   基本的 CLI 工具（kubectl、eksctl、Terraform 和 Helm）

+   一个 Hugging Face 账户用于访问模型

**如果你正在使用本书的数字版，我们建议你自己输入代码，或从本书的 GitHub 仓库访问代码（下一个部分将提供链接）。这样做将帮助你避免与复制和粘贴代码相关的潜在错误。**

# 下载示例代码文件

你可以从 GitHub 下载本书的示例代码文件，网址是[`github.com/PacktPublishing/Kubernetes-for-Generative-AI-Solutions`](https://github.com/PacktPublishing/Kubernetes-for-Generative-AI-Solutions)。如果代码有更新，GitHub 仓库中会更新相应内容。

我们还有其他代码包，来自我们丰富的图书和视频目录，您可以在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。快来看看吧！

使用的约定

本书中使用了许多文本约定。

`代码文本中的代码`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟网址、用户输入以及 X/Twitter 账号。例如：“可以通过向 K8s 服务添加`service.beta.kubernetes.io/aws-load-balancer-nlb-target-type: ip`注解来实现，如下所示：”

代码块的设置如下：

```

...
  metadata {
    name = "gp2"
...
```

当我们希望将注意力引向代码块的特定部分时，相关的行或项会以**粗体**显示：

```

metadata {
    name = "gp3"
    annotations = {
      "storageclass.kubernetes.io/is-default-class": "true"
  ...
```

所有命令行输入或输出如下所示：

```

$ terraform init
$ terraform plan
$ terraform apply -auto-approve
```

**粗体**：表示新术语、重要单词或屏幕上出现的词汇。例如，菜单或对话框中的词汇通常以**粗体**显示。以下是一个示例：“在 Kubecost UI 控制台中，选择左侧菜单的**Savings**，查看成本节省建议。”

提示或重要说明

以此方式显示。

# 与我们联系

我们始终欢迎读者的反馈。

**一般反馈**：如果您对本书的任何内容有疑问，请通过 customercare@packtpub.com 给我们发邮件，并在邮件主题中注明书名。

**勘误**：尽管我们已尽力确保内容的准确性，但错误仍可能发生。如果您在本书中发现任何错误，敬请向我们报告。请访问 www.packtpub.com/support/errata 并填写表格。

**盗版**：如果您在互联网上发现任何我们作品的非法复制品，恳请您提供该地址或网站名称。请通过 copyright@packt.com 联系我们，并附上该资料的链接。

**如果您有兴趣成为作者**：如果您在某个领域拥有专业知识，且有兴趣撰写或参与书籍的编写，请访问 [authors.packtpub.com](http://authors.packtpub.com)。

# 分享您的想法

在阅读完 *Kubernetes for Generative AI Solutions* 后，我们希望听到您的反馈！请[点击这里直接访问亚马逊评价页面](https://packt.link/r/1-836-20993-2)，分享您的想法。

您的评价对我们以及技术社区非常重要，能够帮助我们确保提供高质量的内容。

# 在云计算和 DevOps 领域保持敏锐——加入超过 44,000 名 CloudPro 订阅者

**CloudPro** 是面向云计算专业人员的每周新闻通讯，帮助他们跟上快速发展的云计算、DevOps 和基础设施工程领域的最新动态。

每一期都提供针对性强的高价值内容，涵盖以下主题：

+   AWS、GCP 和多云架构

+   容器、Kubernetes 和编排

+   使用 Terraform、Pulumi 等实现基础设施即代码（IaC）

+   平台工程与自动化工作流

+   可观测性、性能调优和可靠性最佳实践

无论您是云工程师、SRE、DevOps 从业者，还是平台负责人，CloudPro 帮助您在不被杂音干扰的情况下，专注于重要内容。

扫描二维码免费加入并每周获取直达您收件箱的洞察内容：

![](img/NL_Part1.jpg)

[`packt.link/cloudpro`](https://packt.link/cloudpro)

# 下载本书的免费 PDF 版本

感谢您购买本书！

您喜欢随时阅读，但又无法随身携带纸质书籍吗？

您购买的电子书与您选择的设备不兼容吗？

不用担心，现在购买每本 Packt 图书时，您都可以免费获得该书的无 DRM PDF 版本。

在任何地方、任何设备上阅读。直接从你最喜欢的技术书籍中搜索、复制和粘贴代码到你的应用程序中。

优惠不仅如此，你还可以获得独家折扣、新闻简报和每日送到邮箱的精彩免费内容。

按照以下简单步骤获取福利：

1.  扫描二维码或访问以下链接

![](img/B31108_QR_Free_PDF.jpg)

[`packt.link/free-ebook/9781836209935`](https://packt.link/free-ebook/9781836209935)

1.  提交你的购买凭证

1.  就是这样！我们将把你的免费 PDF 和其他福利直接发送到你的邮箱。

# 第一部分：GenAI 和 Kubernetes 基础

本节介绍了**生成性 AI**（**GenAI**）的基础知识，追溯其从传统神经网络到变换器（transformers）的发展历程，并概述了完整的 GenAI 项目生命周期。它探讨了容器和 Kubernetes 如何解决 GenAI 工作负载的挑战，并提供了在云端开始使用 Kubernetes 的指南。

本部分包含以下章节：

+   *第一章*, *生成性 AI 基础*

+   *第二章*, *Kubernetes – 与 GenAI 的介绍与集成*

+   *第三章*, *在云端开始使用 Kubernetes*
