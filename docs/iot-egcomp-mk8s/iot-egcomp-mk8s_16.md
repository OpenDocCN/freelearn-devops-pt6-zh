# 16

# 展望未来

根据最近的 CNCF 调查报告（[`www.cncf.io/reports/cncf-annual-survey-2021/`](https://www.cncf.io/reports/cncf-annual-survey-2021/)），96% 的企业正在使用或考虑使用 Kubernetes。一般来说，容器技术，尤其是 Kubernetes，似乎随着技术的成熟而使用频率降低。组织似乎比以往更 intensively 使用无服务器和托管服务，且用户不再需要了解或理解底层的容器技术。

近年来，随着云原生技术的广泛应用，行业中出现了指数级增长。使用 Kubernetes 和容器对应用程序进行现代化已经成为许多企业的共同主题。对于成熟企业和初创公司而言，基于容器的**持续集成/持续部署**（**CI/CD**）已成为事实上的 DevOps 标准。Kubernetes 是执行边缘计算工作负载的理想平台。此外，Kubernetes 已发展为一个混合计算平台，允许公共云服务提供商在本地环境中设置的集群中运营其托管服务。

基于边缘的基础设施在资源和工作负载管理方面面临诸多挑战。在较短的时间内，成千上万的边缘节点和远程边缘节点需要被控制。组织的边缘架构旨在提供更多的云独立性、高安全性要求和最低延迟。

本书中，我们已覆盖了以下实施方面，解决了使用 MicroK8s 的 IoT/边缘计算场景：

+   启动并运行 Kubernetes 集群

+   启用核心的 Kubernetes 插件，如 DNS 和仪表盘

+   创建、扩展并执行多节点 Kubernetes 集群的滚动更新

+   使用各种容器网络选项进行网络配置——Calico/Flannel/Cilium

+   设置 MetalLB 和 Ingress 选项以进行负载均衡

+   使用 OpenEBS 存储复制进行有状态应用程序的管理

+   配置 Kubeflow 并运行 AI/ML 用例

+   配置与 Istio/Linkerd 的服务网格集成

+   使用 Knative 和 OpenFaaS 运行无服务器应用程序

+   配置日志记录/监控选项（Prometheus、Grafana、Elastic、Fluentd 和 Kibana）

+   配置多节点高可用的 Kubernetes 集群

+   配置 Kata 容器以实现安全容器

+   配置严格的隔离运行环境

此外，在每一章节中，我们还讨论了设计和有效实施 Kubernetes 用于边缘工作负载的指南和最佳实践。

随着企业拥抱数字化转型、工业 4.0、工业自动化、智能制造及其他先进用例，Kubernetes、边缘计算与云计算的协同作用，推动理性商业决策的重要性变得越来越明显。

正在转型为数字优先企业的企业越来越依赖 Kubernetes。Kubernetes 显然是边缘计算的首选平台，至少对于那些需要动态编排应用程序和集中管理工作负载的边缘设备来说。通过在解耦的云环境中实现灵活和自动化的应用管理，Kubernetes 将云原生计算软件开发的优势扩展到边缘。在本章的最后，我们将讨论以下主要主题：

+   MicroK8s 如何在加速 IoT 和边缘部署方面具有独特的优势

+   展望未来 – Kubernetes 趋势与行业前景

# MicroK8s 如何在加速 IoT 和边缘部署方面具有独特的优势

边缘网关必须高效利用计算资源，同时处理多种协议，包括蓝牙、Wi-Fi、3G、4G 和 5G。由于边缘网关的计算能力有限，直接在边缘服务器上操作 Kubernetes 是具有挑战性的。以下是一些问题：

+   为了更好的监控和管理，将控制平面和工作节点从边缘分离，并将控制平面转移到云端，在那里控制平面和工作节点承担工作负载。

+   将集群数据存储分离以处理重负载。

+   专门为进出流量设置工作节点将有助于改善流量管理。

这些问题将导致多个集群的开发，从而使得整个基础设施的管理变得更加具有挑战性。

MicroK8s 迎难而上，它作为边缘集群和主流 Kubernetes 之间的桥梁。由于运行资源有限，因此需要较小的占用空间，同时可以协调完整的云资源池。我们在前面的章节中看到，MicroK8s 利用 Kubernetes 中的不可变容器来提高安全性并简化操作。它有助于创建自愈的高可用集群，自动选择最佳节点来处理 Kubernetes 数据存储。当集群数据库节点之一发生故障时，另一个节点会自动提升，无需管理员干预。MicroK8s 易于安装和升级，并且具备强大的安全性，非常适合微型云和边缘计算。

## 操作 IoT 边缘的一些显著挑战

本节将探讨与 IoT 边缘操作相关的一些重大问题：

+   **计算和资源限制**：IoT 边缘设备的 CPU 和内存资源通常受到限制，因此必须明智地利用这些资源，并保持解决方案的关键功能。

+   **远程和资源管理**：当集群或边缘网络快速扩展时，手动部署、管理和维护设备将变得困难且耗时。以下是一些突出问题：

    +   高效利用设备资源，包括 CPU、内存、网络和边缘设备的 I/O 端口，以及它们的远程监控和管理。

    +   将 CPU 核心和协处理器（例如 GPU）分配给特定工作负载，并承载和扩展任意混合的应用程序。

    +   自动化的远程更新，并具有回滚功能，以避免设备“砖化”。

    +   容易迁移到不同的后端，并自动连接到一个或多个后端（如云或本地基础设施）。

    +   一个分布式、安全的防火墙，根据定义的策略安全地在网络中路由数据。

+   **安全性和可信度**：IoT 边缘设备必须防止未经授权的访问。大规模环境对设备匿名性和可追溯性、发现、认证以及在 IoT 边缘构建信任提出了严峻挑战。为了确保多个 IoT 应用在设备中相互隔离运行，额外的安全层是至关重要的。

+   **可靠性和容错性**：由于系统中 IoT 设备的数量庞大，边缘网络需要自我管理和自我配置的解决方案。IoT 应用需要具备解决在其生命周期内出现的任何问题的能力。IoT 边缘的常见要求包括抗故障能力和缓解拒绝服务攻击。

+   **可扩展性**：在 IoT 生态系统中，传感器或执行器正在逐步负责所有操作。数据收集点的数量和规模都在迅速增长。在 IoT 环境中（如智能城市和智能交通系统）短时间内添加数百个新传感器或执行器是常见的，而该环境仍在多个应用中运行。因此，扩展 IoT 生态系统和数据管理的需求至关重要。此外，边缘服务还面临成本以及其他因素的挑战，包括工作负载监控、存储容量、动态资源分配和数据传输速率。

+   **调度和负载均衡**：为了支持多个服务之间共享数据的大型系统，边缘计算完全依赖于负载均衡和调度方法。必须以安全、可靠和灵活的方式，以更低的成本提供数据、软件和基础设施，以确保计算资源的最优使用。此外，可靠的调度和负载均衡系统也是必不可少的。

既然我们已经了解了管理 IoT 边缘基础设施的主要难点，我们将探讨 MicroK8s Kubernetes 如何有效地解决这些挑战。

## MicroK8s Kubernetes 如何使边缘设备受益。

由于 MicroK8s 能够提高 Kubernetes 的生产力并减少复杂性，它在加速 IoT 和边缘部署方面占据了有利位置。在本节中，我们将看看 MicroK8s Kubernetes 如何使边缘设备受益：

+   **可扩展性**：对于许多物联网解决方案来说，可扩展性是主要关注点。必须有一个能够独立水平或垂直扩展的基础设施，才能支持额外的设备并实时处理 TB 级数据。与传统的虚拟机相比，容器因其轻量性而能更快速地生成。MicroK8s Kubernetes 的主要优势之一就是它在网络集群中扩展的简便性、容器独立扩展的能力以及自动重启而不影响服务的能力。

+   **高可用性**：为了让物联网解决方案能够进行关键业务功能，边缘设备必须随时可用且可靠。由于每个容器都有自己的 IP 地址，因此可以轻松地在容器之间分配负载，并在容器停止工作时重新启动应用程序。我们已经看到如何利用负载均衡功能并运行多个副本以实现高可用性的各种示例。此外，我们还研究了如何设置 HA 集群以应对组件故障的步骤。

+   **资源的高效利用**：由于其有效的资源管理，Kubernetes 降低了托管物联网应用的成本。MicroK8s 是 Kubernetes 的紧凑版、优化版，它在托管的虚拟机、裸金属实例或云上提供了一层抽象。管理员可以专注于将应用服务部署到尽可能多的基础设施上，这样可以降低运行物联网应用的整体基础设施成本。

+   **部署到物联网边缘**：向边缘设备部署软件更新而不干扰服务是一个重要的物联网挑战。通过 Kubernetes 可以运行逐步推出更新的微服务。Kubernetes 安装中通常采用滚动更新策略来推出 Pod 版本更新。通过在更新过程中保持某些实例（例如 Pod 中断预算）运行，能够实现零服务停机。旧的 Pod 仅在新部署版本的流量准备好的 Pod 启用并准备替换时才会被驱逐。因此，应用程序可以通过单个命令水平或垂直扩展。

+   **为物联网启用 DevOps**：为了满足消费者需求，物联网解决方案必须能够平滑地更新，且不会造成用户停机。开发团队可以利用 Kubernetes 中可用的 CI/CD 工具高效地验证、推出并部署物联网服务的变更。此外，Kubernetes 得到多个云服务提供商的支持，包括 Azure、Google Cloud 和 AWS。因此，将来切换到任何云服务都将变得简单。

依赖物联网的行业正在集中力量在边缘设备上实现关键任务服务，以提高解决方案的响应能力并降低成本。基于 Kubernetes 平台构建的解决方案为在边缘实施物联网服务提供了标准框架。Kubernetes 社区的持续进步使得构建可扩展、可靠且可以在分布式环境中部署的物联网解决方案成为可能。

在下一节中，我们将探讨一些推动 Kubernetes 及其采用的趋势。

# 展望未来——Kubernetes 趋势和行业前景

根据 Gartner 于*2021 年 10 月发布的《新兴技术：Kubernetes 与云原生基础设施之争》*报告，“*到 2025 年，85%的组织将在生产环境中运行容器，较 2020 年不到 30%的比例大幅增长*。”在本节中，我们将探讨一些将推动企业采用 Kubernetes 的关键趋势。

## 趋势 1——安全仍然是每个人的关注点

容器和 Kubernetes 已经带来了显著的安全隐患。过去 12 个月，93%的 Kubernetes 环境至少发生了一次安全事件。这可能是由于多个问题所致，例如缺乏容器和 Kubernetes 的安全专业知识、安全工具不足或不适用，以及安全团队未能跟上迅速发展的应用开发团队的步伐，后者往往将安全视为事后考虑。

应用程序的安全态势可能会受到 Kubernetes 中几个配置选项的影响。由于容器和 Kubernetes 环境中的配置错误，可能会出现暴露风险。现在企业已经知道，如果没有在开发生命周期的每个阶段都融入安全措施，就无法充分保障容器化环境的安全。DevSecOps 方法论现在正成为管理容器化环境的重要组成部分。

我在最近的*Kubernetes 和云原生操作报告，2022*中强调了 DevSecOps 的必要性。有关分析和要点，敬请阅读：[`juju.is/cloud-native-kubernetes-usage-report-2022#key-takeaways`](https://juju.is/cloud-native-kubernetes-usage-report-2022#key-takeaways)。

另一个可能受到影响的方面是软件行业的供应链。创建现代软件的过程涉及将多个开放源代码项目自由访问的部分进行组合和合并。一个脆弱的软件组件可能会严重损害应用程序的其他部分以及整个部署，甚至影响复杂的软件供应链。在接下来的日子里，可能会推出新的举措、项目等，以保障软件供应链的安全。

下一个突破是**扩展伯克利数据包过滤器**（**eBPF**），它为云原生开发人员提供了灵活性，可以创建用于安全网络、服务网格和可观测性的组件。我们在*第六章*，*配置容器连接性*中已经看到了使用 eBPF 和 Cilium 的示例。在未来的日子里，eBPF 可能会在安全和网络领域变得普及。

## 趋势 2 – GitOps 用于持续部署

GitOps 提供了著名的基于 Git 的流程，是一个重要的工具，因为它支持快速回滚，并可以作为状态协调的唯一真实来源。

本地集成 GitOps 的方法有很多，包括 Flux CD、Argo CD、Google Anthos 配置管理、Codefresh 和 Weaveworks。

成千上万的 Kubernetes 集群在边缘或混合环境中运行，现在可以轻松通过 GitOps 对多租户和多集群部署的支持进行管理。

因此，GitOps 正在成为持续部署的首选方法。在即将到来的日子里，GitOps 将成为运行和部署 Kubernetes 应用及集群的黄金标准。

## 趋势 3 – 操作员的应用商店

Kubernetes 无需额外的技术专长即可扩展和管理无状态应用，包括 web 应用、移动后端和 API 服务。Kubernetes 内置的功能简单地处理这些任务。

然而，像数据库和监控系统这样的有状态应用需要额外的领域专长，而 Kubernetes 本身并不具备这些。为了扩展、更新和重新配置这些应用，需要额外的了解已部署应用的能力。为了管理和自动化应用生命周期，Kubernetes 操作员将这些独特的领域知识包含在其扩展中。

Kubernetes 操作员通过消除繁琐的手动应用管理任务，使这些流程可扩展、可重复和标准化。

操作员使得应用开发人员更容易部署和维护其应用所需的支持服务。此外，操作员提供了在 Kubernetes 集群上分发应用的标准化方法，并通过发现并修复基础设施工程师和供应商的应用问题，减轻了对支持的要求。我们在*第八章*，*监控基础设施和应用健康*中已经看到了操作员模式的示例，在该示例中，我们为 Kubernetes 部署了 Prometheus Operator，简化了 Kubernetes 服务的监控定义，以及 Prometheus 实例的部署和管理。

然而，有关操作员的“*真实*”来源和可访问性存在担忧，以缓解组织采纳新技术，特别是开源解决方案时的根本性顾虑。

像 Charmhub.io（[`charmhub.io/`](https://charmhub.io/)）一样，应该有一个类似应用商店的中心平台，供人们发布和使用运维工具。这里会有特定的工件所有权、验证和不同的版本。而且这个“商店”将包含足够的信息，供人们根据文档、评分、不同的发布者等，选择合适的版本。

我在最近的 *Kubernetes 和云原生操作报告 2022* 中概述了 *面向运维人员的应用商店* 的想法。阅读更多分析和收获请访问：[`juju.is/cloud-native-kubernetes-usage-report-2022#key-takeaways`](https://juju.is/cloud-native-kubernetes-usage-report-2022#key-takeaways)。

## 趋势 4 – 无服务器计算与容器

Gartner 的分析师早在很久以前就预测了无服务器计算或 **功能即服务** (**FaaS**) 的增长（[`blogs.gartner.com/tony-iams/containers-serverless-computing-pave-way-cloud-native-infrastructure/`](https://blogs.gartner.com/tony-iams/containers-serverless-computing-pave-way-cloud-native-infrastructure/)）。

想象一下，你有一个复杂的容器化系统，正在执行由事件触发的共享服务（例如集成、数据库操作和身份验证）。为了减轻容器化设置的复杂性，你可以将这些任务分离成一个无服务器函数，而不是在容器中运行它们。

此外，使用容器可以轻松扩展无服务器应用程序。在大多数场景中，无服务器函数用于保存数据；你可以通过将这些服务挂载为 Kubernetes 持久化存储卷，来在无服务器和容器架构之间集成并通信有状态数据。

**基于 Kubernetes 的事件驱动自动扩展器** (**KEDA**)（[`keda.sh/`](https://keda.sh/)）帮助运行事件驱动的 Kubernetes 工作负载，如容器化函数，因为它提供了细粒度的自动扩展功能。函数的运行时通过 KEDA 接受事件驱动的扩展功能。根据负载，KEDA 可以从 *零* 个实例（*当没有事件发生时*）扩展到 *n* 个实例。通过使 Kubernetes 自动扩展器（*水平 Pod 自动扩展器*）能够使用自定义指标，它实现了自动扩展。任何 Kubernetes 集群都可以通过利用函数、容器和 KEDA 来复制无服务器函数的能力。

**Knative**（[`knative.dev/docs/`](https://knative.dev/docs/)）是另一个框架，它整合了扩展、Kubernetes 部署模型以及事件和网络路由。通过 Knative 服务资源，基于 Kubernetes 构建的 Knative 平台在工作负载管理方面采取了明确的立场。CloudEvents 是 Knative 的基础，Knative 服务本质上是由事件触发并由事件进行扩展的函数，无论它们是 CloudEvents 事件还是简单的 HTTP 请求。Knative 能够快速响应事件速率的变化，因为它使用 Pod 辅助容器来监控事件速率。此外，Knative 提供了零扩展功能，使工作负载扩展更加精确，特别适用于微服务和函数。

传统的 Kubernetes 部署/服务用于实现 Knative 服务，并且对 Knative 服务的更改（如添加新的容器镜像）会生成相应的 Kubernetes 部署/服务资源。由于 HTTP 流量的路由是 Knative 服务资源描述的一部分，Knative 利用这一点实现蓝绿部署和金丝雀部署模式。

因此，在设计应用程序在 Kubernetes 上的部署时，开发人员应使用 Knative 服务资源及其相关资源来指定事件路由。使用 Knative 意味着开发人员主要关注 Knative 服务，而部署由 Knative 平台处理，这类似于我们今天通过部署资源处理 Kubernetes，并让 Kubernetes 管理 Pods。

**OpenFaaS** 是一个使用 Docker 和 Kubernetes 容器技术创建无服务器函数的框架。任何进程都可以包装成一个函数，使其能够消费各种 Web 事件，而无需一遍又一遍地编写模板代码。这是一个开源倡议，在社区中获得了广泛的关注。

我在我的博客中介绍了 OpenFaaS 框架：[`www.upnxtblog.com/index.php/2018/10/19/openfaas-tutorial-build-and-deploy-serverless-java-functions/`](https://www.upnxtblog.com/index.php/2018/10/19/openfaas-tutorial-build-and-deploy-serverless-java-functions/)。

在*第十章*，《使用 Knative 和 OpenFaaS 框架无服务器化》一章中，我们已经看过如何在 Knative 和 OpenFaaS 平台上部署示例，并通过 CLI 使用其端点进行调用。我们还看到了无服务器框架如何在没有请求时将 Pods 缩放到零，并在有更多请求时启动新的 Pods。我们还讨论了一些在开发和部署无服务器应用程序时需要牢记的指导原则。

在接下来的日子里，可能会有新的倡议和开源项目推出，这些项目有可能促进在无服务器和容器运行方面的创新。

## 趋势 5 – 人工智能/机器学习与数据平台

对于包括**机器学习**和**人工智能**（**ML**和**AI**）在内的工作负载，Kubernetes 已被广泛使用。组织已尝试多种技术来交付这些能力，包括裸金属上的手动扩展、公共云基础设施上的虚拟机扩展以及**高性能计算**（**HPC**）系统。然而，AI 算法通常需要大量的计算能力。

Kubernetes 可能是最有效且直接的选择。将 AI/ML 工作负载打包成容器并在 Kubernetes 上作为集群运行的能力，为 AI 项目提供了灵活性，最大化了资源使用，并为数据科学家提供了自服务环境。

容器让数据科学团队能够构建并可靠地重现验证的设置，无需为每个工作负载调整 GPU 支持。NVIDIA 和 AMD 已经为最新版本的 Kubernetes 添加了实验性的 GPU 支持。此外，NVIDIA 还提供了一系列预加载容器和 GPU 优化的容器化 ML 应用程序（[`developer.nvidia.com/ai-hpc-containers`](https://developer.nvidia.com/ai-hpc-containers)）。

在*第九章*，《使用 Kubeflow 运行 AI/MLOps 工作负载》中，我们介绍了如何设置一个 ML 管道，使用 Kubeflow ML 平台开发和部署一个示例模型。我们还注意到，Kubeflow 在 MicroK8s 上的设置和配置非常简单，且轻量级，能够在构建、迁移和部署管道时模拟现实世界的条件。

我们可以预见，越来越多的 AI/ML 和数据平台将转向 Kubernetes。

## 趋势 6 – 有状态应用

如今，有状态应用已成为常态。尽管容器和微服务等技术创新简化了基于云系统的开发，但它们的敏捷性使得管理有状态进程变得更加困难。

有状态应用必须更频繁地在容器中执行。在边缘、公共云和混合云等复杂环境中，容器化应用可以简化部署和运营。为了使**CI/CD**能够顺利从开发过渡到生产，保持状态同样至关重要。

Kubernetes 通过为平台管理员和应用开发者提供必要的抽象，做出了重大改进，以便运行有状态的工作负载。这些抽象确保了无论容器被调度到哪里，不同类型的文件和块存储都能可用。

在*第十一章*，《使用 OpenEBS 管理存储复制》中，我们讨论了如何配置和实现利用 OpenEBS 存储引擎的 PostgreSQL 有状态工作负载。我们还讨论了一些 Kubernetes 存储的最佳实践，以及选择数据引擎的指导方针。

总结来说，我们可以预见到，2022 年容器和 Kubernetes 基础设施的自动化安全性和持续合规性将成为强劲趋势，同时最佳实践的发展也会成为重点。这对那些必须遵守严格合规标准的企业尤为重要。

# 总结

总结来说，Kubernetes、边缘计算和云计算的协同作用，推动明智的商业决策，随着企业拥抱数字化转型、工业 4.0、工业自动化、智能制造以及所有先进的应用案例，变得越来越显著。我们还探索了不同的部署方法，展示了 Kubernetes 如何用于运行边缘工作负载。本书中，我们涵盖了使用 MicroK8s 实现物联网/边缘计算应用的大部分实施方面。Kubernetes 无疑是边缘计算的首选平台。

我们还看到，MicroK8s 在加速物联网和边缘部署方面具有独特优势。此外，我们还探讨了将塑造未来的一些关键趋势。

恭喜！您已经成功完成了本书。在继续您的 Kubernetes 之旅时，我相信您会从我们在本书中讨论的示例、场景、用例、最佳实践和建议中受益匪浅。

总结来说，Kubernetes 以其快速扩展的生态系统以及多样化的工具、支持和服务，正迅速成为一种有用的工具，尤其是在越来越多的组织转向云计算的过程中。

根据 Canonical 2022 年的 *Kubernetes 和云原生操作调查*（https://juju.is/cloud-native-kubernetes-usage-report-2022），48% 的受访者表示，迁移到或使用 Kubernetes 和容器的最大障碍是缺乏内部能力和人员有限。

如报告所示，存在技能短缺和知识空白，我相信本书可以通过涵盖关键领域来帮助您快速掌握相关内容。

若要跟上更新，您可以订阅我的博客：[`www.upnxtblog.com`](https://www.upnxtblog.com)。

我也期待听到您在 [`twitter.com/karthi4india`](https://twitter.com/karthi4india) 上分享您的经历、意见和建议。

以下是一些优秀的 MicroK8s 资源，助您在旅程中获得支持。

# 进一步阅读

+   官方 MicroK8s 文档：[`microk8s.io/docs`](https://microk8s.io/docs)。

+   MicroK8s 教程：[`microk8s.io/docs/tutorials`](https://microk8s.io/docs/tutorials)。

+   MicroK8s 命令参考：[`microk8s.io/docs/command-reference`](https://microk8s.io/docs/command-reference)。

+   使用的服务和端口：[`microk8s.io/docs/services-and-ports`](https://microk8s.io/docs/services-and-ports)。

+   如果你无法解决问题，并且认为这是 MicroK8s 中的一个 bug，请提交问题到项目仓库：[`github.com/ubuntu/microk8s/issues/`](https://github.com/ubuntu/microk8s/issues/)。

+   贡献 MicroK8s 文档: [`microk8s.io/docs/docs`](https://microk8s.io/docs/docs)。

+   贡献 MicroK8s: [`github.com/canonical/microk8s/blob/master/docs/build.md`](https://github.com/canonical/microk8s/blob/master/docs/build.md)。

# 关于 MicroK8s 的常见问题解答

以下常见问题并不全面，但它们对于运行 Kubernetes 集群非常重要：

1.  如何查看部署的状态？

使用 `kubectl get deployment <deployment>` 命令。如果 `DESIRED`、`CURRENT` 和 `UP-TO-DATE` 都相等，则说明部署已成功。

1.  如何排查状态为 `Pending` 的 pod？

状态为 Pending 的 pod 不能被调度到节点上。使用 `kubectl describe pod <pod>` 命令可以查看 pod 卡住的原因。此外，你还可以使用 `kubectl logs <pod>` 命令来了解是否存在竞争问题。

这个问题最常见的原因是某些 pod 请求了更多的资源。

1.  如何排查状态为 `ContainerCreating` 的 pod？

与 `Pending` 状态的 pod 不同，`ContainerCreating` 状态的 pod 已经被调度到节点上，但由于某些其他原因，它无法正确启动。使用 `kubectl describe pod <pod>` 可以查看 pod 卡住在 `ContainerCreating` 状态的原因。

上述问题的最常见原因包括 CNI 启动错误，也可能是由于卷挂载失败导致的错误。

1.  如何排查状态为 `CrashLoopBackoff` 的 pod？

当一个 pod 因为错误失败时，这是标准的错误信息。`kubectl logs <pod>` 命令通常会显示最近执行的错误信息。通过这些信息，你可以找出导致问题的原因并加以解决。

如果容器仍在运行，你可以使用 `kubectl exec -it <pod> -- bash` 命令进入容器的 shell 进行调试。

1.  如何回滚特定的部署？

如果你在执行 `kubectl apply` 命令时使用了 `–record` 参数，Kubernetes 会将上次的部署记录存储在历史记录中。你可以使用 `kubectl rollout history deployment <deployment>` 命令查看历史部署。

可以使用 `kubectl rollout undo deployment <deployment>` 命令恢复上次的部署。

1.  如何强制将 pod 运行在特定的节点上？

一些常用的方法包括使用 `nodeSelector` 字段以及亲和性和反亲和性。

最简单的建议是，在 pod 定义中使用 `nodeSelector` 字段来定义目标节点应该具有的节点标签。Kubernetes 使用这些信息只将 pod 调度到带有你指定标签的节点上。

1.  如何强制将副本分配到不同的节点？

Kubernetes 默认尝试节点反亲和性，但这不是一个硬性要求；它是尽力而为，但如果没有其他选择，它将在同一节点上调度许多 pod。您可以在这里了解更多关于节点选择的信息：[`kubernetes.io/docs/user-guide/node-selection/`](http://kubernetes.io/docs/user-guide/node-selection/)。

1.  如何列出节点上的所有 pod？

使用以下命令：

```
kubectl get pods -A  --field-selector spec.nodeName=<node name> | awk '{print $2"  "$4}'
```

更详细的 kubectl 技巧表可以在 [`kubernetes.io/docs/reference/kubectl/cheatsheet/`](https://kubernetes.io/docs/reference/kubectl/cheatsheet/) 找到。

1.  如何监视一个始终运行的 pod？

为此，您可以利用存活探针功能。

存活探针始终检查 pod 中的应用程序是否在运行，如果检查失败，则容器将被重新启动。这在容器运行但其中的应用程序崩溃的许多情况下非常有用。

以下代码片段演示了存活探针功能：

```
spec:
containers:
- name: liveness
image: k8s.gcr.io/liveness
args:
- /server
livenessProbe:
      httpGet:
        path: /healthcheck
```

1.  复制控制器和副本集之间有什么区别？

选择器是复制控制器和副本集之间的唯一区别。 Kubernetes 的最新版本不再支持复制控制器，它们的规范也不提及选择器。

更多详细信息可在 [`Kubernetes.io/docs/concepts/workloads/controllers/replicaset/`](https://Kubernetes.io/docs/concepts/workloads/controllers/replicaset/) 找到。

1.  `kube-proxy`的角色是什么？

下面是`kube-proxy`的角色和责任：

+   对于每个服务，它会向其运行的节点分配一个随机端口，并为服务分配代理。

+   安装并维护 iptable 规则，拦截传入到虚拟 IP 和端口的连接，并将其路由到端口。

kube-proxy 组件负责主机子网化，并使服务可用于其他组件。由于 kube-proxy 管理网络通信，关闭控制平面不会阻止节点处理流量。它的操作方式类似于服务。连接将通过 iptables 转发到 kube-proxy，然后 kube-proxy 将使用代理连接到服务的一个 pod。无论端点中有什么，kube-proxy 都将使用目标地址通过转发。

1.  如何在不执行部署清单的情况下测试它？

要测试清单，请使用`--dry-run`标志。这对于确定 YAML 语法是否适合特定的 Kubernetes 对象非常有用，并确保规范包含所需的键值对：

```
kubectl create -f <test manifest.yaml> --dry-run
```

1.  如何打包 Kubernetes 应用程序？

Helm 是一个允许用户打包、配置和部署 Kubernetes 应用程序和服务的包管理器。您可以在这里了解更多关于 Helm 的信息：[`helm.sh/`](https://helm.sh/)。

若要查看快速入门指南，请参见[`www.upnxtblog.com/index.php/2019/12/02/helm-3-0-0-is-outhere-is-what-has-changed/`](https://www.upnxtblog.com/index.php/2019/12/02/helm-3-0-0-is-outhere-is-what-has-changed/)。

1.  什么是`init`容器？

在 Kubernetes 中，一个 Pod 可以包含多个容器。`init` 容器会在 Pod 中的其他容器之前运行。

以下是一个示例，定义了一个简单的 Pod，其中包含两个 `init` 容器。第一个等待 `myservice`，第二个等待 `mydb`。当两个 `init` 容器都完成后，Pod 会根据其 `spec:` 部分执行应用容器。

更多详情请参见此处：[`kubernetes.io/docs/concepts/workloads/pods/init-containers/`](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)。

以下代码片段展示了`initContainers`是如何工作的：

```
apiVersion: v1
kind: Pod
metadata:
  name: sample-app-pod
  labels:
    app: sample-app
spec:
  containers:
  - name: sample-app-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]
```

1.  如何将 Pod 从节点中驱逐进行维护？

使用`drain`命令，方法如下：

```
kubectl drain <node>
```

当你执行上述命令时，它会将节点标记为不调度新的 Pod，然后驱逐或删除现有的 Pod。

一旦你完成了节点的维护并希望将其重新加入集群，执行`uncordon`命令，方法如下：

```
kubectl uncordon <node>
```

更多详情请参见[`kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/`](https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/)。

1.  什么是 Pod 安全策略？

Kubernetes 中的 Pod 安全策略是控制 Pod 可以访问哪些安全功能的配置。它们是一种集群级资源，帮助你控制 Pod 的安全性。

更多详情请参见[`kubernetes.io/docs/concepts/security/pod-security-policy/`](https://kubernetes.io/docs/concepts/security/pod-security-policy/)。

1.  什么是`ResourceQuota`，我们为什么需要它？

`ResourceQuota` 对象限制每个命名空间的资源总消耗。它可以限制在命名空间中根据类型生成的对象数量，以及该项目中资源所能消耗的计算资源总量。

更多详情请参见[`kubernetes.io/docs/concepts/policy/resource-quotas/`](https://kubernetes.io/docs/concepts/policy/resource-quotas/)。
