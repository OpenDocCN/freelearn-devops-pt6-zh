# 前言

## 关于

本节简要介绍了作者和审阅者、本书的内容、你需要掌握的技术技能，以及完成所有主题所需的硬件和软件。

## Azure 上的 Kubernetes 实战 – 第三版

容器和 Kubernetes 容器通过实现更高效的版本控制、增强的安全性和可移植性，促进了云部署和应用程序开发。

本书第三版更新了关于基于角色的访问控制、Pod 身份、存储机密和 AKS 中网络安全的章节，首先介绍了容器、Kubernetes 和 **Azure Kubernetes 服务**（**AKS**），并引导你通过不同方式部署 AKS 集群。接着，你将深入了解 Kubernetes 的具体细节，通过在 AKS 上部署一个示例的留言本应用，并使用 Helm 安装复杂的 Kubernetes 应用。通过真实世界的示例，你还将学会如何扩展你的应用和集群。

随着学习的深入，你将学会如何克服 AKS 中常见的挑战，并使用 HTTPS 来保护你的应用。你还将学会如何通过专门的安全章节保护你的集群和应用。在最后一部分，你将了解高级集成，学习如何在 AKS 上创建 Azure 数据库、运行无服务器功能，并通过 GitHub Actions 将 AKS 与持续集成和持续交付（CI/CD）管道进行集成。

到本书结束时，你将能够熟练地在 Microsoft Azure 上部署容器化的工作负载，且管理开销最小。

### 关于作者

**Nills Franssens** 是一位技术爱好者，也是多种开源技术的专家。自 2013 年以来，他一直从事公共云技术的工作。

在他目前担任微软首席云解决方案架构师的职位上，他与微软的战略客户合作，推动他们的云采纳。他曾与多个客户合作，帮助他们将应用程序迁移到 Azure 上的 Kubernetes 运行。Nills 的专业领域包括 Kubernetes、网络和 Azure 存储。

当他不工作时，你可以找到 Nills 和他的妻子 Kelly 以及朋友们一起玩桌游，或在加利福尼亚州圣荷西的多个小径上跑步。

**Shivakumar Gopalakrishnan** 是 Varian Medical Systems 的 DevOps 架构师。他向 Varian 的产品开发引入了 Docker、Kubernetes 和其他云原生工具，以实现“万物皆代码”。

他在软件开发方面拥有多年的经验，涉猎广泛，包括网络、存储、医学影像，目前主要从事 DevOps 工作。他曾致力于开发专门为医学影像需求量身定制的可扩展存储设备，并帮助架构基于微服务的云原生解决方案，以交付模块化的 AngularJS 应用程序。他在多个活动中发表过关于将 AI 和机器学习融入 DevOps 以促进大企业学习文化的演讲。

他曾帮助高度监管的大型医疗企业团队采用现代敏捷/DevOps 方法，包括 "You build it, you run it" 模型。他制定并领导了 DevOps 路线图的实施，将传统团队转变为能够无缝采用以安全和质量为先的 CI/CD 工具的团队。他拥有吉因迪工程学院的工学学士学位，以及马里兰大学帕克分校的理学硕士学位。

**Gunther Lenz** 是 Varian 技术办公室的高级总监。他是一位创新的软件研发领导者、架构师、MBA、出版作者、公众演讲者和战略技术远见者，拥有超过 20 年的经验。

他有着成功领导超过 50 人的大型、创新性和转型性软件开发和 DevOps 团队的显著经验，专注于持续改进。他通过利用突破性的过程、工具和技术（如云、DevOps、精益/敏捷、微服务架构、数字化转型、软件平台、人工智能和分布式机器学习）在整个软件产品生命周期中领导和定义分布式团队。

他曾获得微软软件架构领域的微软最有价值专家（MVP）称号（2005-2008）。Gunther 已出版两本书：《.NET – 完整开发周期》和《.NET 实践软件工厂》。

### 关于评审人员

**理查德·胡珀**，网名 PixelRobots，居住在英国纽卡斯尔，他是微软 Azure MVP 和微软认证讲师（MCT），现担任总部位于荷兰的公司 Intercept 的 Azure 架构师。他在 IT 行业拥有超过 15 年的职业经验。尽管他职业生涯一直与微软技术打交道，但也曾涉足 Linux。他对 Azure 和 Azure Kubernetes 服务（AKS）充满热情，并每天使用它们。在业余时间，他喜欢通过博客、播客、视频以及各种技术分享自己的知识，帮助他人，希望能为 Azure 之路上的人们提供帮助。理查德热衷于博客写作和学习，这使他每周都有新的发现。当有机会成为 AKS 书籍的技术审阅员时，他毫不犹豫地接受了这个挑战！你可以在 Twitter 上找到他，用户名是 @pixel_robots。

**Swaminathan Vetri**（Swami）在 Maersk 技术中心班加罗尔担任架构师，使用 Azure 的各种 PaaS 产品和 Kubernetes 构建云原生应用程序。自 2016 年以来，他因在开发者社区的技术贡献而被微软评为 MVP - 开发者技术专家。除了写技术博客，他还经常在本地开发者会议、用户小组会议、聚会等场合发言，涉及的主题包括 .NET、C#、Docker、Kubernetes、Azure DevOps、GitHub Actions 等等。作为一个持续学习者，他热衷于与社区分享他的知识。您可以在 Twitter 和 GitHub 上关注他，用户名是 @svswaminathan。

### 学习目标

+   在生产环境中计划、配置和运行容器化应用程序。

+   使用 Docker 构建容器中的应用程序，并将其部署到 Kubernetes 上。

+   监控 AKS 集群和应用程序。

+   使用 Azure Monitor 监控 Kubernetes 中的基础设施和应用程序。

+   使用 Azure 原生安全工具保护您的集群和应用程序。

+   将应用程序连接到 Azure 数据库。

+   使用 Azure 容器注册表安全地存储容器镜像。

+   使用 Helm 安装复杂的 Kubernetes 应用程序。

+   将 Kubernetes 与多个 Azure PaaS 服务集成，例如数据库、Azure 安全中心和 Functions。

+   使用 GitHub Actions 执行持续集成和持续交付到您的集群。

### 受众

如果您是一个有抱负的 DevOps 专业人员、系统管理员、开发人员或网站可靠性工程师，想要学习如何最大限度地利用容器和 Kubernetes，那么本书适合您。

### 方法

本书专注于实践经验和理论知识的良好平衡结合，并伴有与 Kubernetes 平台上专业人员工作直接相关的真实世界场景。每一章都明确设计，帮助您在实践中最大化应用所学。

### 硬件和软件要求

**硬件要求**

为了获得最佳实验体验，我们推荐以下硬件配置：

+   处理器：Intel Core i5 或同等配置

+   内存：4GB RAM（推荐 8 GB）

+   存储：35 GB 可用空间

**软件要求**

我们还建议您提前配置以下软件：

+   一台运行 Linux、Windows 10 或 macOS 操作系统的计算机

+   需要一个互联网连接和网页浏览器，以便您能够连接到 Azure。

### 约定

文本中的代码词汇、数据库名称、文件夹名称、文件名和文件扩展名如下所示。

`front-end-service-internal.yaml` 文件包含使用 Azure 内部负载均衡器创建 Kubernetes 服务的配置。以下代码是该示例的一部分：

```
1   apiVersion: v1
2   kind: Service
3   metadata:
4     name: frontend
5     annotations:
6       service.beta.kubernetes.io/azure-load-balancer-internal: "true"
7     labels:
8       app: guestbook
9       tier: frontend
10  spec:
11    type: LoadBalancer
12    ports:
13    - port: 80
14    selector:
15      app: guestbook
16      tier: frontend
```

### 下载资源

本书的代码包可在 https://github.com/PacktPublishing/Hands-on-Kubernetes-on-Azure-Third-Edition 下载。

我们还有来自我们丰富书籍和视频目录的其他代码包，可以在 https://github.com/PacktPublishing/ 查找。快去看看吧！
