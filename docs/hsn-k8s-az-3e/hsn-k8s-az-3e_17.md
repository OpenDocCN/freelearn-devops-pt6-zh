# 第四部分：与 Azure 托管服务集成

到目前为止，你已经在**Azure Kubernetes Service**（**AKS**）上运行了多个应用程序。这些应用程序总是自包含的，意味着整个应用程序可以在 AKS 上完整运行。将整个应用程序部署在 AKS 上有一些优点。你可以获得应用程序的可移植性，因为你可以将该应用程序轻松迁移到其他任何 Kubernetes 集群。此外，你对应用程序的端到端控制权也非常充分。

权力越大，责任也越大。将应用程序的部分功能卸载到 Azure 提供的某些 PaaS 服务中有其优势。例如，将数据库卸载到托管的 PaaS 服务后，你无需再关心数据库服务的更新，备份会自动执行，而且很多日志记录和监控都会开箱即用。

在接下来的章节中，你将学习更多关于多种高级集成以及它们带来的好处。在阅读完本节内容后，你应该能够安全地访问其他 Azure 服务，如 Azure SQL 数据库和 Azure Functions，并使用 GitHub Actions 执行**持续集成与持续交付**（**CI/CD**）。

本节包含以下章节：

+   *第十二章*，*将应用程序连接到 Azure 数据库*

+   *第十三章*，*Azure 安全中心与 Kubernetes 的集成*

+   *第十四章*，*无服务器函数*

+   *第十五章*，*AKS 的持续集成与持续部署*

本节将从*第十二章*，*将应用程序连接到 Azure 数据库*开始，你将在这一章中将应用程序连接到 Azure MySQL 数据库。
