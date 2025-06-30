# 序言

Kubernetes Secrets 管理是通过一系列实践和工具，帮助用户在 Kubernetes 集群内安全存储和管理敏感信息（如密码、令牌和证书）并确保其安全。保护 Secrets（如密码、API 密钥和其他敏感信息）对防止未经授权的访问、保护应用程序和数据至关重要。了解 Kubernetes Secrets 管理的开发人员可以帮助确保 Secrets 得到安全有效的管理，从而降低安全漏洞的风险。许多行业和监管框架对敏感数据的管理有具体要求。通过学习 Kubernetes Secrets 管理的实践，开发人员可以确保他们的应用程序遵守这些要求，从而避免潜在的法律或财务处罚。

# 本书适合谁阅读

本书适用于希望在 Kubernetes 上部署和管理 Secrets 的软件工程师、DevOps 工程师和系统管理员。具体来说，面向以下人员：

+   已经熟悉 Kubernetes 并希望理解如何有效管理 Secrets 的开发人员。这包括已经使用 Kubernetes 进行应用程序部署的人员，以及那些对 Kubernetes 平台感兴趣并希望进一步了解其能力的新手。

+   对于那些希望学习如何在 Kubernetes 环境中安全管理 Secrets 的安全专业人士来说，这本书也非常适用。这些人可能负责保护应用程序、基础设施或网络的安全，或者负责合规性和监管要求的人员。

+   任何有兴趣使用 Kubernetes 安全部署和管理应用程序，并希望理解如何在该环境中有效管理 Secrets 的人。

# 本书的内容

*第一章*，*理解 Kubernetes Secrets 管理*，介绍了 Kubernetes 及其在 Kubernetes 上部署应用程序时 Secrets 管理的重要性。它概述了管理 Secrets 的挑战和风险、目标以及本书的范围。

*第二章*，*深入讲解 Kubernetes Secrets 管理概念*，涵盖了 Kubernetes Secrets 管理的基础知识，包括不同类型的 Secrets、它们的使用场景、如何在 Kubernetes 中创建、修改和删除 Secrets，以及安全存储和访问控制。还讨论了如何通过 RBAC 和 Pod 安全标准安全访问 Secrets，以及如何审计和监控 Secrets 的使用情况。

*第三章*，*以 Kubernetes 原生方式加密 Secrets*，教你如何在传输和 etcd 中存储的 Secrets 加密，以及在 Kubernetes 中的密钥管理和轮换。

*第四章*，*调试和故障排除 Kubernetes Secrets*，提供了识别和解决管理 Kubernetes Secrets 时常见问题的指导。涵盖了调试和故障排除 Secrets 的最佳实践，包括使用监控和日志工具，确保基于 Kubernetes 的应用程序的安全性和可靠性。

*第五章*，*安全性、审计与合规性*，重点讨论了在 Kubernetes 中管理 Secrets 时合规性和安全性的重要性。涵盖了如何遵循安全标准和法规，减轻安全漏洞，并确保 Kubernetes Secrets 管理的安全性。

*第六章*，*灾难恢复和备份*，帮助你了解 Kubernetes Secrets 的灾难恢复和备份。还涵盖了备份策略和灾难恢复计划。

*第七章*，*管理 Secrets 的挑战和风险*，重点讨论了在混合云和多云环境中管理 Secrets 时的挑战和风险。还涵盖了缓解 Kubernetes Secrets 管理中的安全风险的策略、确保 Kubernetes Secrets 管理安全的指南以及可用于 Kubernetes Secrets 管理的工具和技术。

*第八章*，*探索 AWS 上的云密钥存储*，介绍了 AWS Secrets Manager 和 KMS 以及如何将它们与 Kubernetes 集成。还涵盖了使用 AWS CloudWatch 对 Kubernetes Secrets 进行监控和日志记录操作。

*第九章*，*探索 Azure 上的云密钥存储*，教你如何将 Kubernetes 与 Azure Key Vault 集成进行密钥存储，以及如何加密存储在 etcd 上的密钥。还涵盖了通过 Azure 的可观察性工具对 Kubernetes Secrets 进行监控和日志记录操作。

*第十章*，*探索 GCP 上的云密钥存储*，介绍了 GCP Secret Manager 和 GCP KMS 以及如何将它们与 Kubernetes 集成。还涵盖了使用 GCP 监控和日志对 Kubernetes Secrets 进行监控和日志记录操作。

*第十一章*，*探索外部密钥存储*，探讨了不同类型的第三方外部密钥存储，如 HashiCorp Vault 和 CyberArk Secrets Manager。教你如何使用外部密钥存储来存储敏感数据及其最佳实践。此外，本章还涵盖了使用外部密钥存储的安全影响，以及它们如何影响 Kubernetes 集群的整体安全性。

*第十二章*，*与 Secret 存储的集成*，教你如何将第三方 Secrets 管理工具与 Kubernetes 集成。它介绍了 Kubernetes 中的外部 Secret 存储及可以使用的不同类型的外部 Secret 存储。你还将了解使用外部 Secret 存储的安全影响，并学习如何通过多种方式（如初始化容器、边车容器、CSI 驱动程序、操作员和密封 Secrets）使用它们来存储敏感数据。本章还涵盖了使用外部 Secret 存储的最佳实践，以及它们如何影响 Kubernetes 集群的整体安全性。

*第十三章*，*案例研究与实际应用示例*，涵盖了 Kubernetes Secrets 在生产环境中的实际应用案例。它介绍了已经在 Kubernetes 中实施 Secrets 管理的组织案例研究，以及从实际部署中获得的经验教训。此外，你还将学习如何在 CI/CD 流水线中管理 Secrets，并将 Secrets 管理集成到 CI/CD 过程中。本章还介绍了用于在流水线中管理 Secrets 的 Kubernetes 工具和安全的 CI/CD Secrets 管理最佳实践。

*第十四章*，*总结与 Kubernetes Secrets 管理的未来*，概述了 Kubernetes Secrets 管理的当前状态以及未来的发展趋势。它还讨论了如何跟进 Kubernetes Secrets 管理的最新趋势和最佳实践。

# 为了从本书中获得最大的收获

你应该理解 Bash 脚本、容器化以及 Docker 的工作原理。你还应该理解 Kubernetes 和基本的安全概念。了解 Terraform 和云服务提供商的知识也将大有裨益。

| **本书涵盖的软件** | **操作系统要求** |
| --- | --- |
| Docker | Windows、macOS 或 Linux |
| Shell 脚本 |  |
| Podman 和 Podman Desktop |  |
| minikube |  |
| Helm |  |
| Terraform |  |
| GCP |  |
| Azure |  |
| AWS |  |
| OKD 和 Red Hat OpenShift |  |
| StackRox 和 Red Hat 高级集群安全 |  |
| Aqua 的 Trivy |  |
| HashiCorp Vault |  |

**如果你使用的是本书的数字版，我们建议你自己输入代码，或从本书的 GitHub 仓库访问代码（下一节会提供链接）。这样可以避免由于复制和粘贴代码而可能出现的错误。**

# 下载示例代码文件

你可以从 GitHub 下载本书的示例代码文件，链接为[`github.com/PacktPublishing/Kubernetes-Secrets-Handbook`](https://github.com/PacktPublishing/Kubernetes-Secrets-Handbook)。如果代码有更新，它将在 GitHub 仓库中同步更新。

我们还有其他来自丰富图书和视频目录的代码包，您可以在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。快来看看吧！

# 使用的约定

本书中使用了多种文本约定。

`文本中的代码`：表示文本中的代码字、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名。举个例子：“`kms`提供程序插件将`kube-apiserver`与外部 KMS 连接，以利用封套加密原则。”

一段代码的设置如下：

```
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - aesgcm:
          keys:
            - name: key-20230616
              secret: DlZbD9Vc9ADLjAxKBaWxoevlKdsMMIY68DxQZVabJM8=
      - identity: {}
```

当我们希望将您的注意力集中在代码块的某个部分时，相关行或项会以粗体显示：

```
apiVersion: v1
kind: ServiceAccount
metadata:
  annotations:
    eks.amazonaws.com/role-arn: "arn:aws:iam::11111:role/eks-secret-reader"
  name: service-token-reader
  namespace: default
```

任何命令行输入或输出如下所示：

```
$ kubectl get events
...
11m         Normal    Pulled              pod/webpage                              Container image "nginx:stable" already present on machin
```

**粗体**：表示新术语、重要词汇或屏幕上显示的词语。例如，菜单或对话框中的词语通常以**粗体**显示。举个例子：“GCP 提供的另一个显著工具是**GKE 安全** **姿态**仪表板，用以提升 GKE 集群的安全性。”

提示或重要说明

显示如下。

# 与我们联系

我们始终欢迎读者的反馈。

**一般反馈**：如果您对本书的任何部分有疑问，请通过电子邮件联系我们：customercare@packtpub.com，并在邮件主题中注明书名。

**勘误**：虽然我们已经尽一切努力确保内容的准确性，但错误有时仍会发生。如果您在本书中发现了错误，请向我们报告。请访问[www.packtpub.com/support/errata](http://www.packtpub.com/support/errata)并填写表格。

**盗版**：如果您在互联网上发现我们作品的任何非法复制品，欢迎您提供其地址或网站名称。请通过 copyright@packtpub.com 与我们联系，并附上该材料的链接。

**如果您有兴趣成为作者**：如果您在某个主题上拥有专长，并且有兴趣写书或为书籍做贡献，请访问[authors.packtpub.com](http://authors.packtpub.com)。

# 分享您的想法

阅读完《*Kubernetes Secrets Handbook*》后，我们希望听听您的想法！请[点击这里直接前往亚马逊评论页面](https://packt.link/r/1-805-12322-X)并分享您的反馈。

您的评论对我们和技术社区至关重要，将帮助我们确保提供高质量的内容。

# 下载本书的免费 PDF 副本

感谢您购买本书！

喜欢随时随地阅读，但又不能随身携带纸质书吗？

您的电子书购买是否与您的设备不兼容？

别担心，现在每本 Packt 图书都提供免费的无 DRM PDF 版本，您可以免费获取。

在任何地方、任何设备上随时阅读。直接从您最喜爱的技术书籍中搜索、复制和粘贴代码到您的应用程序中。

福利不仅仅是这些，您还可以每天在邮箱中获得独家折扣、时事通讯和精彩的免费内容。

按照以下简单步骤获得福利：

1.  扫描二维码或访问以下链接。

![扫码下载本书免费 PDF 版本](https://packt.link/free-ebook/9781805123224)

[`packt.link/free-ebook/9781805123224`](https://packt.link/free-ebook/9781805123224)

1.  提交您的购买凭证。

1.  就这样！我们将直接把您的免费 PDF 和其他福利发送到您的电子邮件中。

# 第一部分：Kubernetes Secrets 管理简介。

本部分将为您提供 Kubernetes Secrets 的基础知识，并介绍其在管理 Kubernetes 上部署应用程序的敏感数据中的重要性。在本部分结束时，您将掌握 Kubernetes Secrets 的目的、功能和使用方法，并通过实际案例进行学习。

本部分包含以下章节：

+   *第一章**，理解 Kubernetes Secrets 管理*。

+   *第二章**，深入了解 Kubernetes Secrets 管理概念*。

+   *第三章**，以 Kubernetes 原生方式加密 Secrets*。

+   *第四章**，调试和排除 Kubernetes Secrets 故障*。
