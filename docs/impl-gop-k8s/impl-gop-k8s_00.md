# 序言

你好！

在快速发展的软件开发领域，保持部署的一致性、可扩展性和可靠性是一个重大挑战。GitOps 作为一种革命性的方式，弥合了开发与运维之间的鸿沟，尤其是在 Kubernetes 环境中。通过将 Git 作为系统和应用配置的唯一可信来源，GitOps 自动化并标准化了部署，确保部署过程一致、可审计且高效。

GitOps 利用版本控制、基础设施即代码和持续部署的原则，创建无缝透明的工作流。这种方法不仅简化了运维工作，还增强了团队协作并加快了交付速度，使团队可以将更多精力集中在创新上，而不是处理突发问题。

本书《*使用 GitOps 实现 Kubernetes：在 AWS 和 Azure 上自动化、管理、扩展和保护基础设施与云原生应用*》旨在提供一份全面的 GitOps 掌握指南。通过实际案例、逐步教程和行业专家的见解，我们将探讨如何在 Kubernetes 部署中有效实施 GitOps 实践。

我们的旅程从介绍 GitOps 的核心概念和原则开始。我们将深入探讨 Argo CD、Flux CD、Helm 和 Kustomize 等工具的技术细节，以及它们如何与 Kubernetes 集成。接下来，我们将讨论一些高级主题，如多集群管理、安全性和可扩展性，确保你对 GitOps 有全面的理解。

基于我们在云架构和 DevOps 实践方面的丰富经验，我们将分享已在多个行业中验证过的真实场景和最佳实践。通过本书的学习，你将掌握实施 GitOps 策略所需的知识和技能，无论是当前还是未来的 Kubernetes 部署，都能确保降低复杂性并提高可扩展性。

无论你是 DevOps 工程师、站点可靠性工程师、平台工程师，还是云架构师，本书将为你提供成功应对当今云原生环境所需的工具和见解。让我们一起踏上这段旅程，解锁 GitOps 在 Kubernetes 部署中的全部潜力。

# 本书适合的读者

*使用 GitOps 实现 Kubernetes：在 AWS 和 Azure 上自动化、管理、扩展和保护基础设施与云原生应用* 旨在为那些希望提升使用 GitOps 原则部署和管理 Kubernetes 环境的专业人士提供指导。

本书的主要读者包括以下群体：

+   **DevOps 工程师**：负责管理和自动化应用程序及基础设施部署的专业人士。本书将为他们提供先进的 GitOps 技术，以简化工作流，减少部署错误，并确保环境间的一致性。

+   **站点可靠性工程师（SRE）**：专注于维护应用程序可靠性和性能的工程师。本书将提供如何将 GitOps 集成以提高可观察性、自动恢复和高效扩展 Kubernetes 集群的洞见。

+   **平台工程师**：构建和维护支持应用开发与部署的底层平台的人员。他们将学习如何实施 GitOps 以管理基础设施作为代码，确保平台的健壮性、可扩展性和安全性。

+   **云工程师**：与云平台打交道、需要管理复杂 Kubernetes 环境的工程师。本书将教他们如何利用 GitOps 自动化部署、管理多云环境并优化云资源利用。

+   **软件工程师**：希望更好理解部署过程并参与基础设施管理的开发人员。本书将为他们提供全面的 GitOps 实践知识，使他们能与运维团队更高效地协作。

+   **解决方案架构师**：负责设计和实施技术解决方案的专业人员。他们将更深入地理解如何将 GitOps 融入其架构设计，确保解决方案的可扩展性和可维护性。

+   **IT 领导者和经理**：负责在组织中实施 DevOps 和云原生策略的领导者。本书将帮助他们理解 GitOps 的好处，指导他们在团队中做出关于采纳和扩展 GitOps 实践的明智决策。

本书的目标读者理应具备以下背景：

+   **云计算知识**：读者应具备基础的云计算概念和环境理解，因为 GitOps 实践通常涉及将应用程序部署到云平台上。

+   **持续集成和持续部署（CI/CD）原则的熟悉度**：对 CI/CD 原则的基本理解至关重要，因为 GitOps 建立在这些方法论基础上，自动化并简化了部署过程。

+   **基本的 Kubernetes 理解**：具备 Kubernetes 的相关经验会非常有帮助，因为本书深入探讨了如何将 GitOps 与 Kubernetes 环境集成。读者应熟悉 Kubernetes 的基本概念，如 Pod、服务和部署。

+   **版本控制系统的经验**：由于 GitOps 依赖于 Git 进行版本控制，读者应具备 Git 或类似版本控制系统的经验。这包括理解分支、合并和管理代码库。

+   **DevOps 工具和实践**：熟悉如 Docker、Helm 等 DevOps 工具将帮助读者更高效地掌握本书中讨论的高级主题。

到本书结束时，读者将掌握实施 GitOps 策略所需的知识和实际技能，确保在 Kubernetes 部署中减少复杂性、提高可扩展性和改善操作效率。

# 本书涵盖的内容

*第一章*，*GitOps 简介*，提供了 GitOps 的基础理解，探讨了其原则以及它如何转变现代软件开发中的文化、工作流和思维方式。

*第二章*，*使用 GitOps 导航云原生操作*，深入探讨了如何使用 GitOps 实践构建和管理容器化应用，涵盖 Kubernetes 基础、容器镜像优化和云原生管道等主题。

*第三章*，*与 Git 和 GitHub 的版本控制和集成*，阐述了 Git 和 GitHub 在 GitOps 中的关键作用，提供了有关有效版本控制和协作开发实践的见解。

*第四章*，*使用 GitOps 工具的 Kubernetes*，探讨了各种 GitOps 工具，如 Helm、Kustomize、Argo CD 和 Flux CD，详细介绍了它们与 Kubernetes 的集成，并提供了比较分析，帮助选择适合特定需求的工具。

*第五章*，*GitOps 的规模化和多租户*，讨论了针对规模化部署和管理多集群环境的高级 GitOps 实践，包括有效的 Git 仓库管理策略和为 Kubernetes 构建服务目录。

*第六章*，*GitOps 架构设计与操作控制*，重点介绍了 GitOps 的架构框架和操作方法，强调在云原生部署中的可扩展性、弹性和效率。

*第七章*，*IT 文化转型以拥抱 GitOps*，重点讨论了采用 GitOps 所需的文化转变，涉及基础设施即代码、不可变基础设施、DORA 指标以及克服组织抗拒的原则。

*第八章*，*在 OpenShift 中使用 GitOps*，深入探讨了在 Red Hat OpenShift 环境中应用 GitOps 原则，包括设置 GitOps 工作流、利用 OpenShift 的 CI/CD 工具以及保护 GitOps 管道。

*第九章*，*Azure 和 AWS 部署中的 GitOps*，介绍了在 Azure 和 AWS 生态系统中实施 GitOps 实践，详细阐述了集成云原生工具和服务，以简化应用和基础设施管理。

*第十章*，*基础设施自动化的 GitOps —— Terraform 和 Flux CD*，深入探讨了如何通过 Terraform 和 Flux CD 集成自动化基础设施管理，涵盖了版本控制、多环境管理和高级自动化技术。

*第十一章*，*在 Kubernetes 上使用 GitOps 部署实际项目*，提供了一个动手实践指南，介绍了如何使用 GitOps 和 Kubernetes 执行实际项目，从设置开发环境到设计、开发和部署可扩展的应用程序。

*第十二章*，*GitOps 可观察性*，探讨了将可观察性实践集成到 GitOps 工作流中的方法，涵盖了 SRE 原则、内部与外部可观察性、SLO 驱动的性能和高级监控技术。

*第十三章*，*GitOps 安全性*，讨论了 GitOps 的安全方面，包括加固声明式 CD、实现政策即代码、管理密钥、维护平台目录和自动化安全扫描。

*第十四章*，*GitOps 的 FinOps、可持续性、AI 和未来趋势*，突出了将 FinOps 与 GitOps 融合，实现可持续和成本效益的运营，涉及成本预测、优化、碳足迹评估、AI 驱动的自动化以及 GitOps 未来的趋势。为了充分发挥本书的价值

为了充分发挥本书的价值，读者应具备云计算和 DevOps 原则的基础知识。熟悉 Kubernetes 和容器化技术（如 Docker）是必需的。具备版本控制系统的经验，特别是 Git，将对 GitOps 有很大帮助，因为 GitOps 严重依赖这些工具。基本了解 CI/CD 流水线和基础设施即代码的概念也将帮助读者理解本书中涉及的高级主题。

| **本书提到的软件/硬件** | **操作系统要求** |
| --- | --- |
| Kubernetes | Windows、macOS 或 Linux |
| Git | Windows、macOS 或 Linux |
| Docker | Windows、macOS 或 Linux |
| Argo CD | Windows、macOS 或 Linux |
| Flux CD | Windows、macOS 或 Linux |
| Helm | Windows、macOS 或 Linux |
| Kustomize | Windows、macOS 或 Linux |
| Terraform | Windows、macOS 或 Linux |
| Azure Kubernetes Service (AKS) | Windows、macOS 或 Linux |
| AWS Elastic Kubernetes Service (EKS) | Windows、macOS 或 Linux |
| OpenShift | Windows、macOS 或 Linux |

本书提供了完成每一章所需的所有必要指令。提供了逐步的指南，确保顺利的设置和实现过程。本书中讨论的示例和项目的源代码已上传到公开的代码库。请参阅下一页获取代码库链接和更多详细信息。

**如果你正在使用本书的数字版，我们建议你自己输入代码，或从本书的 GitHub 仓库获取代码（下节中会提供链接）。这样做可以帮助你避免因复制粘贴代码而可能导致的错误。**

# 下载示例代码文件

你可以从 GitHub 上下载本书的示例代码文件，网址是 [`github.com/PacktPublishing/Implementing-GitOps-with-Kubernetes`](https://github.com/PacktPublishing/Implementing-GitOps-with-Kubernetes)。如果代码有更新，它将会在 GitHub 仓库中更新。

我们还有来自丰富书籍和视频目录中的其他代码包可供下载，访问 [`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)。快去看看吧！

# 使用的约定

本书中使用了许多文本约定。

`文本中的代码`：表示文本中的代码词汇、数据库表名、文件夹名称、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名。举个例子：“在 `optimization`/`opencost` 下有全球定义的值，然后为每个特定国家有一个自定义定价模型。”

代码块的格式如下：

```
helm repo add sealed-secrets https://bitnami-	labs.github.io/sealed-secrets
helm install sealed-secrets sealed-secrets/sealed-secrets
#Install e.g. CLI on MacOS
brew install kubeseal
```

当我们希望引起你对某个代码块中特定部分的注意时，相关的行或项目会以粗体显示：

```
clusterName: "aks-excelsior-development-2"globalConfig:  chat_gpt_token: sk-dw******  signing_key: ea657a******  account_id: 7935371f******sinksConfig:- slack_sink:    name: main_slack_sink     slack_channel: pocs     api_key: xoxb******- robusta_sink:    name: robusta_ui_sink     token: eyJhY2NvdW******enablePrometheusStack: true # This part is added to the default generated_values.yaml enablePlatformPlaybooks: true runner:  sendAdditionalTelemetry: true rsa:  private: ******  public: ******# This part is added to the default generated_values.yaml playbookRepos:  chatgpt_robusta_actions:    url: "https://github.com/robusta-dev/kubernetes-chatgpt-bot.git"# This part is added to the default generated_values.yaml customPlaybooks:# Add the 'Ask ChatGPT' button to all Prometheus alerts - triggers:  - on_prometheus_alert: {}  actions:  - chat_gpt_enricher: {}
```

任何命令行输入或输出格式如下：

```
$ aws eks --region eu-central-1 update-kubeconfig --name eksgitopscluster
```

**粗体**：表示新术语、重要词汇或你在屏幕上看到的词汇。例如，菜单或对话框中的词汇会以**粗体**显示。举个例子：“当**常见漏洞和暴露**（**CVE**）被揭示时，如果你选择驾驶舱和舰队方法，采用 GitOps 大规模管理也有助于实施大规模的漏洞管理策略。”

提示或重要注意事项

看起来像这样。

# 联系我们

我们非常欢迎读者的反馈。

**一般反馈**：如果你对本书的任何方面有疑问，请通过 customercare@packtpub.com 给我们发送电子邮件，并在邮件主题中提到书名。

**勘误**：尽管我们已尽力确保内容的准确性，但错误总是难以避免。如果你在本书中发现任何错误，我们将非常感谢你向我们报告。请访问 [www.packtpub.com/support/errata](http://www.packtpub.com/support/errata) 并填写表单。

**盗版**：如果你在互联网上发现我们作品的任何非法副本，无论何种形式，我们将非常感激你提供其位置地址或网站名称。请通过 copyright@packt.com 与我们联系，并附上相关链接。

**如果你有兴趣成为作者**：如果你在某个领域有专长，并且有兴趣写书或为书籍做贡献，请访问 [authors.packtpub.com](http://authors.packtpub.com)。

# 分享你的想法

一旦你阅读了 *实施 Kubernetes 中的 GitOps*，我们很想听听你的想法！请 [点击此处直接访问 Amazon 书评页面](https://packt.link/r/1835884237)并分享你的反馈。

你的评论对我们和技术社区非常重要，它将帮助我们确保提供卓越的高质量内容。

# 下载本书的免费 PDF 版

感谢购买本书！

你喜欢在旅途中阅读，但又无法随身携带纸质书籍吗？

你的电子书购买是否与你选择的设备不兼容？

不用担心，现在购买每本 Packt 图书，你都可以免费获得该书的无 DRM PDF 版本。

在任何地方、任何设备上阅读。直接在应用程序中搜索、复制并粘贴你喜欢的技术书籍中的代码。

福利不止于此，你还可以获得独家折扣、新闻通讯和每日送达的优质免费内容。

按照以下简单步骤获取福利：

1.  扫描二维码或访问以下链接

![](img/B22100_QR_Free_PDF.jpg)

[`packt.link/free-ebook/9781835884225`](https://packt.link/free-ebook/9781835884225)

1.  提交你的购买凭证

1.  就这些！我们会将你的免费 PDF 和其他福利直接发送到你的电子邮箱

# 第一部分：通过简化的编排/Kubernetes 理解 GitOps

在本部分，你将探讨 GitOps 的基础知识。从广泛的概述开始，你将了解 GitOps 如何成为现代软件开发和平台工程中的关键实践。我们将深入探讨如何进行云原生操作，强调 Git 和 GitHub 的集成，以实现有效的版本控制。此外，你将使用 Kubernetes 和各种 GitOps 工具来简化部署过程，全面理解这些技术如何互联互通，以简化和增强软件的部署与管理。

本部分包括以下章节：

+   *第一章*，GitOps 简介

+   *第二章*，通过 GitOps 导航云原生操作

+   *第二章*，版本控制与 Git 和 GitHub 的集成

+   *第四章*，使用 GitOps 工具的 Kubernetes
