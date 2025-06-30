

# 第十二章：与秘密存储集成

Kubernetes 提供了一个基本的系统来管理 Secrets，但通常不被认为足够安全，尤其是在生产环境中处理密码、令牌或密钥等敏感数据时。为了解决这个问题，将先进的秘密管理工具集成到 Kubernetes 中变得至关重要。这些工具通过加密增强了安全性，并提供了敏感信息的集中管理。它们超越了 Kubernetes Secrets 的原生功能，使得安全性更加强大和合规。本章将讲解如何将秘密管理工具与 Kubernetes 集成。内容包括如何在 Kubernetes 中配置外部秘密存储，以及可用的不同类型的外部秘密存储。你将了解使用外部秘密存储的安全性影响，并学习如何通过不同的方法，如 init 容器、sidecar、CSI 驱动程序、操作员和封印的 Secrets，来存储敏感数据。本章还将涵盖使用外部秘密存储的最佳实践，以及它们如何影响 Kubernetes 集群的整体安全性。本章我们将涉及以下主要主题：

+   在 Kubernetes 中配置外部秘密存储

+   与外部秘密存储的集成

+   安全性影响与最佳实践

+   实际与理论的平衡

# 技术要求

为了将概念与实际操作示例结合，我们将使用一系列常用于与外部秘密管理和 Kubernetes 交互的工具和平台：

+   **minikube**：它在你的计算机上通过**虚拟机**（**VM**）运行一个单节点的 Kubernetes 集群。可以通过 [`minikube.sigs.k8s.io/docs/start/`](https://minikube.sigs.k8s.io/docs/start/) 的指南进行设置。

+   **Helm**：这是一个 Kubernetes 的包管理工具，可以简化部署过程。有关安装的指南，可以查看 [`helm.sh/docs/intro/install/`](https://helm.sh/docs/intro/install/)。

+   **kubectl**：这是 Kubernetes 的命令行工具。安装指南可以在 [`kubernetes.io/docs/tasks/tools/install-kubectl/`](https://kubernetes.io/docs/tasks/tools/install-kubectl/) 找到。

+   **外部秘密管理工具**：虽然可以使用多种工具进行演示，但建议使用 Hashicorp Vault。它的官方安装指南可以在 [`www.vaultproject.io/docs/install`](https://www.vaultproject.io/docs/install) 找到。

将秘密存储与 Kubernetes 集成

正如我们在前几章中探讨的那样，Kubernetes 本身具备秘密管理功能。然而，在大规模操作或特定的安全需求下，原生的 Kubernetes Secrets 可能会有所不足。之前讨论过的各种秘密管理工具的多样性，正是暗示了这一需求。但为什么要将它们与 Kubernetes 集成呢？

将第三方工具与 Kubernetes 集成带来以下好处：

+   **操作一致性**：对于已经在 Kubernetes 外使用工具的组织，集成提供了一个统一的秘密管理体验。

+   **增强的安全特性**：许多外部工具提供了高级功能，如秘密轮换、细粒度的访问控制和多层加密方法，这些功能在 Kubernetes 原生环境中并不容易获得，或者需要额外配置。

+   **可扩展性和性能**：在大规模环境中，仅使用 Kubernetes 原生的 Secrets 管理大量秘密可能变得复杂。为高容量操作设计的外部工具可以有效应对这一挑战。

+   **高级审计追踪**：在日益严格的法规和增加的网络威胁环境中，拥有一个全面的审计能力至关重要，而非奢侈。许多工具配备了完整的日志记录和警报功能。

+   **详细的审计能力**：这些能力确保合规性，提升安全性，增加责任追踪，检测异常活动，支持基于信息的决策，提供法律证据，增强操作效率，建立客户信任，减少内部威胁，并为未来的改进提供历史分析。

+   **跨平台兼容性**：随着混合云和多云策略的兴起，秘密管理器可以在不同的云平台之间提供一致的 Secrets 管理，使得在这些异构环境中管理 Secrets 更加简便。

虽然我们在前面的章节中已经认识到各种云秘密存储和第三方秘密存储的能力，本章旨在弥补这一差距，专注于集成。主要的焦点是展示这些秘密存储如何与 Kubernetes 无缝集成，充分利用两者的优势。

通过后续章节，我们将深入探讨这些集成的机制，提供理论与实践的双重理解。从 Kubernetes 扩展到 Pod 生命周期机制，每种方法都将展示不同的集成策略和方法。到本章结束时，我们的目标是为您提供一套强大的策略和见解，帮助您做出与独特操作需求无缝对接的选择。

# 在 Kubernetes 中配置外部秘密存储

Kubernetes 的去中心化特性和动态工作负载要求有一个强大的秘密管理解决方案。本节提供了关于一般配置过程的见解，并阐明了在 Kubernetes 中使用秘密的两种主要范式。

以下是一般的配置步骤：

1.  **选择秘钥存储**：首先选择一个适合组织需求的 Secrets 管理工具，考虑安全需求、可扩展性、合规标准、团队熟悉度等因素。可选的方案非常多，包括云原生解决方案，如 AWS/GCP Secrets Manager 和 Azure Key Vault，也有诸如 HashiCorp Vault 和 CyberArk 等工具。

1.  **初始化并连接到 Kubernetes**：一旦选择了秘钥存储，就开始其初始化。根据架构偏好，将其部署在 Kubernetes 集群内部或旁边，确保秘钥存储与 Kubernetes 之间的顺畅连接。

1.  **处理认证与授权**：在 Kubernetes 和秘钥存储之间建立强大且安全的通信渠道。机制可以包括 IAM 角色、令牌、服务帐户或客户端证书。同时，建立精细的授权控制，确保只有授权的服务或应用访问指定的 Secrets。

1.  **确定秘钥获取与使用方式**：深入探讨 Secrets 将如何被使用。决定是否将外部存储中的 Secrets 转换为原生 Kubernetes Secrets，或者在需要时直接从外部存储中提取。

1.  **测试配置**：在将集成应用到生产环境之前，进行全面测试。验证密钥的获取、使用以及其他已配置的功能，确保它们按预期工作。

1.  **监控与审计**：作为最后一步，实施监控机制来监督对 Secrets 的访问。并通过日志记录和审计工具增强这一过程，快速发现未经授权的访问尝试或潜在的安全漏洞。

完成这些通用配置步骤为在 Kubernetes 环境中实现安全高效的 Secrets 管理奠定了坚实的基础。现在，秘钥存储已经集成、认证并授权，您可以继续进入下一阶段，确保应用程序能够无缝且安全地使用 Secrets。

## Kubernetes 中的秘钥消费

在集成外部秘钥存储时，有两种主要模式主导了 Kubernetes 中的密钥使用：

+   **转换为原生 Kubernetes Secrets**：将外部存储中的 Secrets 转换为原生 Kubernetes Secrets 使您能够利用 Kubernetes 原生的方法进行 Secrets 管理和访问。这带来了缓存的好处，减少了频繁外部请求的需求。此外，它还消除了一个关键的故障点。然而，也存在诸如冗余和确保两个秘钥存储位置之间同步等挑战。

+   **直接从外部存储中获取**：直接检索机密确保应用程序能够获取到最新的版本，减少了同步的需求，同时也能保持更清晰的审计记录。然而，这种方法可能会由于外部获取操作而引入延迟，并且会对外部存储产生直接依赖。

总结来说，在 Kubernetes 中配置外部机密存储的过程是构建可扩展、安全的云原生基础设施的基础。清楚理解配置步骤和各种机密消费方式为制定有效的机密管理策略奠定了基础。未来的章节将更深入地探讨这些主题及其相关机制。

# 与外部机密存储集成

将外部机密存储与 Kubernetes 集成是保护应用程序和敏感数据的关键组成部分。本节探讨了可以无缝将外部机密存储与 Kubernetes 集群集成的各种机制和模式，从而增强安全性和管理效率。

## Kubernetes 扩展和 API 机制

Kubernetes 提供了各种扩展和 API 机制，可以利用它们与外部机密存储进行连接和交互。在本部分，我们将深入探讨可用选项，并指导你如何有效地利用它们进行机密管理。

### Kubernetes 中的准入控制器和变异 webhook 用于机密管理

Kubernetes 提供了一整套工具，用于控制和修改其环境中的行为。其中，*准入控制器*和*变异 webhook*在增强 Kubernetes 集群的操作和安全性方面发挥了至关重要的作用。特别是在机密管理方面，这些工具可能会带来革命性的变化。

准入控制器是 Kubernetes 控制平面的一部分，负责管理和执行集群的使用方式。它们会在对象持久化之前、但在请求被认证和授权之后，拦截对 Kubernetes API 服务器的请求。通过这样做，准入控制器能够采取特定行动，例如拒绝请求或在存储之前修改对象。有几个内置的准入控制器，但对于像机密管理这样的特定需求，你可能需要自定义控制器。

当我们需要由准入控制器提供的灵活性，但又需要自定义逻辑时，变异 webhook 就会发挥作用。它们允许你在创建或修改特定资源时运行自定义代码（或自定义函数）。这对机密管理非常有价值，因为你可以以编程方式修改 Kubernetes 资源；例如，你可以在 Pod 创建时注入机密引用。

假设有一个场景，你不希望开发者在清单中显式地定义 Secrets。通过使用变更 webhook，你可以建立一个系统，开发者只需要指定标签或注释。Webhook 会拦截 Pod 创建请求，识别标签或注释，并注入所需的 secret 引用，从而抽象化了与 Secrets 的直接交互。

这是设置用于 Secret 注入的变更 webhook 的示例：

```
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingWebhookConfiguration
metadata:
  name: secret-injector-webhook
webhooks:
  - name: secret-injector.example.com
    clientConfig:
      service:
        name: secret-injector-service
        namespace: default
        path: "/mutate"
      caBundle: [CA_BUNDLE]
    rules:
      - operations: ["CREATE"]
        apiGroups: [""]
        apiVersions: ["v1"]
        resources: ["pods"]
```

让我们分解一下这个配置：

+   这个 webhook 被命名为 `secret-injector.example.com`。

+   `clientConfig` 指定了处理 webhook 的服务。在这种情况下，服务名为 `secret-injector-service`。

+   `rules` 部分定义了何时调用此 webhook。在这里，它设置为在创建 Pod 时运行。

+   已创建。

当一个新的 Pod 被创建时，请求会被我们的 webhook 拦截。服务 `secret-injector-service` 然后处理这个请求，检查特定的标签或注释，并决定是否注入 secret 引用。

Admission 控制器和变更 webhook 的结合提供了一种强大的机制，用于简化和强制执行 Secrets 管理的最佳实践。通过将 Secrets 管理的责任交给这些工具，开发者可以专注于他们的应用逻辑，同时确保 Secrets 以安全和合规的方式处理。

总结来说，在希望增强 Kubernetes 集群中的 Secrets 管理功能时，考虑利用 admission 控制器和变更 webhook。它们不仅有助于维护集群的完整性，还能自动化并强制执行处理敏感数据的最佳实践。

### Secrets 管理中的自定义资源定义

Kubernetes 通过使用 **自定义资源定义**（**CRDs**）允许其 API 的扩展性。CRDs 使集群操作员能够在 Kubernetes 中引入新的资源类型，而无需修改 Kubernetes 核心代码库。在处理 Secrets，尤其是那些存储在 Kubernetes 集群外部的系统（如 AWS Secrets Manager 或 HashiCorp Vault）中的 Secrets 时，CRDs 可以提供一种更符合 Kubernetes 原生方式的管理和访问方法。

#### 定义 ExternalSecret CRD

外部 secret 的 CRD 定义可能如下所示：

```
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: externalsecrets.k8s.example.com
spec:
  group: k8s.example.com
  versions:
    - name: v1
      served: true
      storage: true
  scope: Namespaced
  names:
    plural: externalsecrets
    singular: externalsecret
    kind: ExternalSecret
    shortNames:
      - esec
```

一旦定义了 CRD，下一步就是创建这个新资源类型（`ExternalSecret`）的实例。然而，仅仅定义和创建 CRD 并不能赋予其功能。为了使 `ExternalSecret` 资源有意义，你需要一个自定义控制器来理解如何解读和操作这些资源。

#### 使用 ExternalSecret CRD

假设你有一个名为 `database-password` 的 secret 存储在 AWS Secrets Manager 中，你可能会定义一个引用它的 `ExternalSecret` 资源，内容如下：

```
apiVersion: k8s.example.com/v1
kind: ExternalSecret
metadata:
     name: my-database-password
spec:
backendType: awsSecretsManager
data:
  - key: database-password
    name: dbPassword
region: us-west-1
```

这是这个资源的分解：

+   `metadata.name`：这是 Kubernetes 中分配给该`ExternalSecret`的名称。它不必与 AWS Secrets Manager 中的名称完全匹配。

+   `spec.backendType`：表示要使用的外部 Secrets 管理器；在此情况下，是 AWS Secrets Manager。

+   `spec.data`：这是一个列表，指示要获取的密钥。

+   `key`：这是在 AWS Secrets Manager 中密钥的名称或标识符。

+   `name`：这是密钥在呈现给 Kubernetes Pods 时的名称。

+   `spec.region`：指定存储密钥的 AWS 区域。

在应用以下资源后：

```
kubectl apply -f my-external-secret.yaml
```

一个观察`ExternalSecret`资源的自定义控制器将执行以下操作：

1.  检测到新的`ExternalSecret`的创建。

1.  从规格中理解它应与`us-west-1`区域的 AWS Secrets Manager 进行通信。

1.  与 AWS Secrets Manager 进行身份验证（假设它具有必要的权限）。

1.  获取`database-password`密钥。

1.  将此密钥以`dbPassword`的名称展示给 Kubernetes Pods。具体方法可以是创建一个本地 Kubernetes 密钥、将其设置为环境变量，或根据控制器设计将其放入`tmpfs`卷中。

实际上，CRD 结合自定义控制器提供了一种强大的机制，来扩展 Kubernetes 的能力。在密钥管理方面，CRD 使得 Kubernetes 能够自然地与外部密钥存储解决方案集成，使得提取和使用密钥的过程对最终用户无缝化。

### Kubernetes API 扩展：自定义 API 服务器

构建一个自定义 API 服务器使我们能够定义 API 行为，包括与外部密钥存储的交互。Pod 可以通过自定义 API 请求密钥，API 服务器可以从外部存储中获取密钥、处理它并返回。然而，运行和维护一个自定义 API 服务器并不是简单的。你需要设置它，确保它的安全性，并可能处理扩展和故障转移。

请注意，这是一个简化的示例，重点讲解概念配置：

```
apiVersion: v1
kind: Pod
metadata:
  name: custom-api-server
spec:
  containers:
  - name: custom-api-server
    image: my-custom-api-server:latest
    ports:
    - containerPort: 443
    volumeMounts:
    - mountPath: /etc/custom-api-server
      name: config
  volumes:
  - name: config
    configMap:
      name: custom-api-server-config
```

为了让自定义 API 服务器能够与主 Kubernetes API 服务器和外部密钥存储一起工作，它需要在其配置`custom-api-server-config`中进行特定设置。这些设置包括如何验证谁有权限访问，这叫做身份验证，以及如何进行通信的规则，称为 API 规范。通常，这个设置会使用基于服务的身份验证或基于角色的身份验证。基于服务的身份验证检查请求访问的服务的身份，而基于角色的身份验证则查看用户或服务的角色来决定访问权限。一个常见的例子是使用 AWS 中的 IRSA 角色，Kubernetes 服务可以安全地访问 AWS 资源。

这种方法提供了与外部密钥存储的无缝交互，特别是对于那些更熟悉 `kubectl` 而不是 Hashicorp Vault CLI 的团队。通过扩展 API，用户可以保持在熟悉的环境中。然而，尽管 API 扩展非常强大，单靠扩展 API 并不能完成整个过程。你需要额外的组件或流程来确保 Pods 安全高效地使用这些 Secrets。这可以通过代理、控制器或其他编排机制来实现，它们会监控这些自定义或转换后的 Secrets，并使其可供 Pods 使用。

Kubernetes 扩展和 API 机制提供了一种灵活且强大的方式来集成外部密钥存储，提供多种选项以适应不同的用例和需求。了解如何利用这些工具是有效管理 Kubernetes 中 Secrets 的关键。

## Pod 生命周期和管理机制

在 Pod 生命周期中管理 Secrets 对于维护安全性和操作效率至关重要。本节重点介绍 Kubernetes 提供的机制，用于在 Pod 生命周期中注入和管理 Secrets。

### Init 容器

Init 容器在应用容器之前运行，可用于设置任务，例如从外部存储获取 Secrets。如果你的应用需要在启动之前通过配置文件填充 Secrets，Init 容器可以获取这些 Secrets，填充配置文件并将其存储在共享卷中。

下面是一个示例配置：

```
apiVersion: v1
kind: Pod
metadata:
  name: app-with-init-container
spec:
  initContainers:
  - name: fetch-secrets
    image: secret-fetcher:latest
    volumeMounts:
    - name: config-volume
      mountPath: /config
  containers:
  - name: main-app
    image: my-app:latest
    volumeMounts:
    - name: config-volume
      mountPath: /config
  volumes:
  - name: config-volume
    emptyDir: {}
```

通过集成这个示例 Init 容器配置，你可以确保应用在启动之前就能访问所需的 Secrets。

### Sidecar 容器

Sidecar 容器与 Pod 中的主容器一起运行，可以用于在 Pod 生命周期期间动态管理 Secrets。如果你的应用需要定期刷新其 Secrets 而不重启，Sidecar 容器可以获取最新的 Secrets，并更新共享配置或通知主应用。

下面是一个示例配置：

```
apiVersion: v1
kind: Pod
metadata:
  name: app-with-sidecar
spec:
  containers:
  - name: main-app
    image: my-app:latest
    volumeMounts:
    - name: config-volume
      mountPath: /config
  - name: secret-refresher
    image: secret-refresher:latest
    volumeMounts:
    - name: config-volume
      mountPath: /config
  volumes:
  - name: config-volume
    emptyDir: {}
```

Kubernetes 中的 Sidecar 容器通过与主容器并行运行，增强了 Secrets 管理，使 Secrets 可以动态更新，而无需重启应用，正如提供的示例配置所示。

### DaemonSets

DaemonSets 确保所有（或部分）节点运行一个 Pod 的副本，使其适合用于节点级任务，例如设置节点范围的 Secrets 或 Secrets 管理工具。如果你有一个节点级应用（例如，日志代理）需要特定的 Secrets，可以使用 DaemonSet 确保每个节点获取自己的 Secrets。

下面是一个示例配置：

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-level-agent
spec:
  selector:
    matchLabels:
      name: node-level-agent
  template:
    metadata:
      labels:
        name: node-level-agent
    spec:
      containers:
      - name: agent
        image: my-agent:latest
        env:
        - name: NODE_SECRET
          valueFrom:
            secretKeyRef:
              name: node-secret
              key: secret-key
```

通过利用 DaemonSets 来进行此类节点级操作，你可以确保在整个 Kubernetes 集群中一致且安全地分发 Secrets。

### 环境控制器

不同于 CRD，环境控制器并不寻求扩展 Kubernetes API。相反，它们直接在 Pod 的上下文中动态管理环境变量。其优势在于可以在 Pod 层级进行直接集成，避免了需要额外的 CRD 管理或特定于新 CRD 的控制器基础设施。对于从环境变量读取 Secrets 的应用程序，如果你希望避免直接在 Kubernetes 中存储这些 Secrets，环境控制器可以在 Pod 启动前获取并注入这些 Secrets，避免了应用程序或其他容器去获取它们的需求。

假设我们使用了一个自定义控制器，它监控 `EnvSecret` CRD 资源。这里是一个示例配置：

```
apiVersion: mycontroller/v1
kind: EnvSecret
metadata:
  name: database-creds
spec:
  externalRef: db-credentials-in-vault
  target:
    envVarName: DB_CREDS
    podSelector:
      matchLabels:
        app: my-app
```

这个假设的 `EnvSecret` CRD 资源指示控制器从外部存储中获取 `db-credentials-in-vault`，并将其填充到标签为 `app: my-app` 的 Pods 的 `DB_CREDS` 环境变量中。

有效地管理 Secrets 贯穿整个 Pod 生命周期，确保应用程序在需要时能够访问到必要的敏感信息，同时保持高水平的安全性。

## 专用的 Kubernetes 模式 – SealedSecrets

**SealedSecrets** 是一个 Kubernetes 控制器和工具，用于一键加密的 Secrets。它旨在帮助开发人员加密一个 secret 并提交到控制平面（通常是在 Git 仓库中并通过 **持续集成和持续交付** (**CI/CD**）进行管理）。Kubernetes 管理员拥有解密密钥，并且当看到加密的 Secret（SealedSecret）时，控制器会将其解密为常规的 Kubernetes Secret。它通过确保实际的 secret 值不会直接存储在 Git 仓库中，而是以加密格式保存，来增强安全性。

SealedSecrets 的亮点在于它的简洁性：Secrets 以只有集群本身能够解密的方式进行加密，从而允许 Secrets 安全地与应用程序的配置一起存储，通常是在版本控制中。

让我们通过 SealedSecrets 过程的不同阶段来了解一下：

1.  `kubeseal` CLI 工具

1.  使用 `kubeseal` 加密一个 secret，这将创建一个 SealedSecret

1.  `kubectl`，就像其他任何 Kubernetes 资源一样

1.  **解密**：SealedSecret 控制器在集群中运行，它解密 SealedSecret 并创建一个标准的 Kubernetes secret

在 DevOps 中使用 SealedSecrets 的主要好处是其易用性。它允许开发人员将应用程序的配置和 Secrets（加密形式）一起安全地进行版本控制。然而，值得注意的是，SealedSecrets 与常规的 Kubernetes Secrets 并不完全相同。当解密时，SealedSecrets 会在集群内转化为标准的 Kubernetes Secrets。这些 Secrets 只有具有相应权限的工作负载才能访问。

让我们简要探讨 SealedSecrets 的创建和应用，以及它们如何在 Pods 中使用：

1.  **创建** **一个 SealedSecret**：

    这里是一个创建简单 Kubernetes 密钥的快速示例。

    ```
       apiVersion: v1
       kind: Secret
       metadata:
         name: my-secret
       type: Opaque
       data:
         password: [base64_encoded_value]
    sealed-secret.yaml is as follows:
    ```

    ```
       apiVersion: bitnami.com/v1alpha1
       kind: SealedSecret
       metadata:
         name: my-secret
         namespace: default
       spec:
         encryptedData:
           password: [encrypted_value]
    ```

1.  通过`kubectl`执行`sealed-secret.yaml`文件，SealedSecret 控制器将解密该文件并在指定的命名空间中创建一个名为`my-secret`的常规 Kubernetes 密钥。

1.  **在 Pods 中的使用**：

    一旦 SealedSecret 被解密并且常规密钥可用，Pods 可以像引用其他密钥一样引用此密钥，例如，将其挂载为卷或用来设置环境变量：

    ```
       apiVersion: v1
       kind: Pod
       metadata:
         name: my-app
       spec:
         containers:
         - name: app
           image: my-app:latest
           env:
           - name: PASSWORD
             valueFrom:
               secretKeyRef:
                 name: my-secret
                 key: password
    ```

本质上，SealedSecrets 使密钥在集群外部的加密存储和管理成为可能，而集群内的控制器确保它们在需要时被安全解密并转化为可访问的 Kubernetes 密钥。它和谐地弥合了密钥加密的操作需求与这些密钥在 Kubernetes 生态系统中实际使用之间的鸿沟。

## Kubernetes 密钥的 Secret Store CSI 驱动程序

Secret Store CSI 驱动程序为将外部密钥管理平台与 Kubernetes 集成提供了先进的解决方案。这个强大的机制旨在提高 Kubernetes 工作负载中处理密钥的安全性和效率。

### 了解用于密钥管理的 CSI 驱动程序

**容器存储接口**（**CSI**）是将各种存储系统连接到 Kubernetes 等调度器的关键标准。在密钥管理领域，Secret Store CSI 驱动程序充当这个连接桥梁：

+   **CSI 驱动程序**：从根本上讲，这是 Kubernetes 与众多外部存储系统之间的接口。它负责动态地提供密钥。在一个密钥访问及时性至关重要的世界里，驱动程序提供的这一能力是无价的。

+   `secrets-store.csi.k8s.io`使 Kubernetes 能够从高级外部密钥存储中提取并挂载多个密钥、证书和机密。然后，这些数据会作为卷提供给 Pods。连接时，封装的数据会被挂载到 Pod 的文件系统中。这种直接访问确保了应用程序可以轻松使用这些密钥。

CSI 驱动程序作为 Kubernetes 与外部存储解决方案之间的重要桥梁。通过促进密钥、证书和机密的无缝和安全提供，确保了及时访问和高效集成。

### 密钥 CSI 驱动程序的独特方面

Secret Store CSI 驱动程序具有几个独特的特点：

+   **双重架构**：驱动程序将 CRD 和 DaemonSets 结合在一起。CRD 负责引导自定义行为和与外部密钥存储的交互，而 DaemonSets 确保驱动程序在集群中的每个节点上都有一个副本在运行。这种架构确保了密钥在集群中均匀地可用。

+   `tmpfs`内存文件系统。此方法确保密钥不会被写入节点磁盘，从而提高安全性。

+   **节点级接口**：由于它在节点级别与 Kubernetes CSI 接口一起工作，因此驱动程序要求每个主机上具有根用户权限。

Secret Store CSI 驱动程序凭借其创新的双重架构、直接的内存中密钥挂载和节点级操作而脱颖而出，这需要 root 权限，并确保在 Kubernetes 集群中对 Secrets 管理的统一、安全方法。以下是一个示例配置，演示了 Secret Store CSI 驱动程序的端到端使用：

1.  **部署 Secret Store** **CSI 驱动程序**：

    要在 Kubernetes 中快速开始使用 Secret Store CSI 驱动程序，可以使用 Helm 3 进行安装：

    1.  首先，通过以下命令添加驱动程序的 Helm 仓库：

        ```
        kube-system namespace:

        ```

        helm install csi-secrets-store secrets-store-csi-driver/secrets-store-csi-driver --namespace kube-system

        ```

        ```

    精确的部署步骤可能因 Kubernetes 环境而异，但通常你可以选择使用 helm charts 或原始 YAML 文件。这些都可以在官方仓库中找到，网址是 [`github.com/kubernetes-sigs/secrets-store-csi-driver`](https://github.com/kubernetes-sigs/secrets-store-csi-driver)。

1.  **声明** **SecretProviderClass**：

    这是告诉驱动程序在哪里以及如何获取密钥的核心对象：

    ```
    apiVersion: secrets-store.csi.x-k8s.io/v1alpha1
    kind: SecretProviderClass
    metadata:
      name: my-secret-provider
    spec:
      provider: [PROVIDER_NAME]  # e.g., azure, vault
      parameters:
        # Your specific provider parameters here
    ```

1.  `SecretProviderClass` 已设置，可以在你的 Pod 配置中引用它：

    ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: my-pod
    spec:
      containers:
      - name: my-container
        image: my-image
      volumes:
        - name: secrets-volume
          csi:
            driver: secrets-store.csi.k8s.io
            readOnly: true
            volumeAttributes:
              secretProviderClass: "my-secret-provider"
      volumeMounts:
      - name: secrets-volume
        mountPath: "/mnt/secrets"
    ```

提供的示例配置概述了部署驱动程序、声明 `SecretProviderClass` 并将其整合到 Pod 中的步骤。接下来，我们将深入探讨这种方法的优点和限制。

### Secrets CSI 驱动程序的优点和限制

Secret Store CSI 驱动程序虽然功能强大，但也有其优点和挑战：

+   `tmpfs` 确保 Secrets 永远不会保存在节点磁盘上

+   **动态更新**：根据外部存储的能力，Secrets 可以动态更新，确保工作负载访问最新数据

+   **限制**：

    +   **复杂的设置**：双重性质（CRD 和 DaemonSet）可能使初始设置更为复杂

    +   **节点级别访问**：在每个节点上要求 root 权限在某些环境中可能被视为安全隐患

    +   **提供者依赖**：某些功能可能依赖于外部密钥存储的能力

有关详细信息、最佳实践和社区支持，请始终参考官方文档，网址是 [`secrets-store-csi-driver.sigs.k8s.io/`](https://secrets-store-csi-driver.sigs.k8s.io/)。对于那些有兴趣贡献、了解架构或探索其详细功能的人，项目的 GitHub 仓库是一个有价值的资源（GitHub 上的 Secret Store CSI 驱动程序，网址是 [`github.com/kubernetes-sigs/secrets-store-csi-driver`](https://github.com/kubernetes-sigs/secrets-store-csi-driver)）。

总之，Secret Store CSI 驱动程序标志着 Kubernetes 在管理 Secrets 方面的重大进展。采用它可以带来更安全和高效的 Secrets 管理，但与所有工具一样，正确的实施至关重要。

## 服务网格集成用于密钥分发

在 Kubernetes 不断发展的世界中，*服务网格*已经成为处理服务间通信的关键叠加层。它的主要价值主张在于抽象化服务间交互的复杂性，减轻开发者将此逻辑嵌入应用代码的负担。对于密钥分发，尤其是在证书和令牌的上下文中，服务网格发挥着至关重要的作用。总而言之，服务网格是一个为微服务应用提供配置的基础设施层，使得通信变得灵活、可靠且快速。它通过轻量级网络代理实现，这些代理与应用代码一起部署，而无需应用程序感知。

### 服务网格中的密钥 – 证书和令牌

当我们在服务网格的上下文中谈论“密钥”时，我们主要指的是以下内容：

+   **证书**：这些证书用于在网格中的服务之间建立信任。**互信 TLS**（**mTLS**）通常用于确保客户端和服务器服务彼此信任。服务网格自动化这些证书的提供和轮换。

+   **令牌**：对于某些身份验证和授权场景，可能会使用令牌（如 JWT）。这些令牌可以由服务网格生成、验证和轮换，确保应用程序不必处理这些复杂性。

服务网格通过自动化简化并安全管理证书和令牌。在服务网格中，密钥的处理和分发既安全又动态，遵循着整个生命周期中广泛应用且成熟的流程。

1.  **动态密钥创建**：服务网格可以与外部**证书颁发机构**（**CAs**）集成，甚至可以拥有内置的 CA。按需生成证书，当服务加入网格时，自动为其创建证书。

1.  **密钥分发**：生成后，这些证书（或令牌）会被安全地分发给相关服务。这一分发过程通过与每个服务实例配套的边车代理进行。

1.  **轮换和续期**：使用服务网格的一个关键优势是它能够自动化密钥的轮换。此功能通过定期更新这些敏感凭证来增强安全性。经过预定时间后，密钥会被续期，旧的密钥将失效，整个过程不需要停机或手动干预。

1.  **吊销**：在某个服务可能被攻破的情况下，服务网格可以迅速吊销相关的密钥，减轻潜在的损害。

服务网格自动化整个过程，从按需创建和分发证书及令牌到管理它们的轮换和吊销，确保安全高效的操作，并最小化手动干预。

### 服务网格实例 – Istio

虽然有多种服务网格实现可用，但**Istio**作为一个突出的例子脱颖而出。

对于其证书管理，Istio 使用一个名为**Citadel**的组件。它充当证书颁发机构（CA），为网格中的服务生成、分发、轮换证书，并具有撤销证书的能力。借助其内置功能，Istio 的 Citadel 确保网格中的 mTLS 通信始终保持安全。

以下是启用 Istio 中 mTLS 的示例配置：

```
  apiVersion: "security.istio.io/v1beta1"
  kind: "PeerAuthentication"
  metadata:
    name: "default"
    namespace: "foo"
  spec:
    mtls:
      mode: STRICT
```

Istio 作为领先的服务网格实现，利用其 Citadel 组件，不仅用于强大的证书管理，以确保双向 TLS 通信的安全；还扩展了其功能，涵盖身份验证、身份提供和策略执行，成为一个全面的解决方案，用于在服务网格架构中管理安全性。

### 好处与考虑事项

服务网格在 Kubernetes 中集成时，通过自动化证书管理增强安全性，并为所有通信启用 mTLS。这确保了安全策略在整个集群中的一致应用。然而，附加的边车代理层引入了延迟，并给监控和维护带来了挑战。

对于已经在使用或计划使用服务网格的组织，将其用于密钥管理，特别是证书和令牌，变得非常具有吸引力。这种集成简化了密钥管理，确保了安全传输、适当的范围控制和定期轮换。然而，必须认识到，虽然服务网格在管理证书和令牌方面表现出色，但它们并不是适用于所有密钥类型的“一刀切”解决方案。

总之，当服务网格存在于 Kubernetes 设置中时，利用它来管理证书和令牌可以简化操作并增强安全性。然而，它不应被视为全面的密钥管理的完全替代方案。

总结来说，服务网格增强了 Kubernetes 生态系统的安全性，特别是在证书和令牌形式的密钥分发方面。虽然其好处多多，但和任何技术一样，彻底理解并认真实施是实现其全部潜力的关键。

## 代理系统在密钥管理中的作用

在 IT 安全的广阔领域，特别是在讨论密钥管理时，**代理系统**一词作为一个重要角色出现。作为中介，这些代理就像交通警察，确保应用程序在验证其*身份*后获得所需内容。

以下内容可以帮助您理解代理机制如何工作：

+   **请求**：需要密钥的应用程序向代理发送请求。

+   **验证**：代理验证请求，通常会验证发送方的身份和授权。

+   **获取和传输**：一旦验证通过，代理从存储中检索密钥，并安全地将其发送到应用程序。

+   **审计和日志**：所有事务，无论是请求还是获取，都会被适当记录以供审计。这种设计确保了应用程序避免直接与不同的密钥存储交互；它们只需要与代理进行接口对接。

参考以下示例。假设服务 `foo` 需要连接到特定的数据库。与其直接获取数据库凭证，发生的情况如下：

1.  `foo` 发送请求：`我需要凭证` `用于 DB1。`

1.  代理检查 `foo` 是否拥有 `DB1` 的正确权限

1.  确认后，代理获取并交付凭证

在整个过程中，`foo` 不关心确切的密钥存储位置和检索方法。它通过代理安全地获取必要的数据库凭证，代理验证权限并检索信息，确保了安全的访问过程。

### 代理机制与无密钥代理

由于术语相似，很容易将两者混淆，但它们的功能是不同的。无密钥代理更进一步；它们代表应用程序建立连接，而不会将密钥暴露给应用程序。

本质上，它们的区别如下：

+   **代理机制**：它们在验证后将密钥传递给应用程序

+   **无密钥代理**：它们使用密钥促进直接连接，同时保持密钥的隐藏

### 思考泡泡 – 代理和服务网格

此时，值得与服务网格进行类比。服务网格通过代理来控制和管理微服务架构中服务之间的流量。如果你仔细想想，代理不就是像一个代理吗？的确，服务网格的代理确保服务之间的安全通信，可能还会管理证书、令牌以及有时其他密钥。然而，其主要关注点并不是密钥管理，而是促进服务间的安全通信。

### 为什么代理仍然重要？

尽管无密钥代理和服务网格正在取得进展，传统的代理系统仍然不可或缺。它们灵活，适用于各种应用程序，提供密钥分发的集中控制，允许细粒度的访问，并且常常弥合遗留系统的差距。它们的角色不仅仅是检索，还包括密钥治理和生命周期管理。

探索将外部密钥存储与 Kubernetes 集成的领域，本节阐明了对于安全高效的密钥管理至关重要的机制和模式。我们从 Kubernetes 扩展和 API 机制开始，展示了如何将这些工具无缝地融入到您的密钥管理策略中，接着深入研究了 Pod 生命周期和操作机制，确保在 Pod 的生命周期内，密钥始终得到安全管理。接下来的内容讨论了 Kubernetes 的专门模式，重点介绍了 SealedSecrets 这一增强安全性的范式，并深入探讨了服务网格集成，展示了其在安全密钥分发和服务间通信中的强大能力。最后，讨论了密钥管理中的代理系统，强调了它们在应用程序与密钥存储之间创建安全中介层的作用，并确保了一个解耦且灵活的管理系统。总的来说，这些子部分共同构成了一份全面的指南，帮助团队在 Kubernetes 中安全高效地管理密钥，同时应对外部密钥集成的复杂性。

# 安全隐患与最佳实践

随着 Kubernetes 的普及，将其与外部密钥存储集成具有一些特定的优势，如专用加密和审计能力。然而，这种方式也带来了一系列挑战和安全隐患。

以下是这些最佳实践的列表：

+   **对外部系统的依赖**：依赖外部密钥存储意味着引入了一个额外的复杂性和依赖层。外部存储的任何停机或安全漏洞都可能直接影响在 Kubernetes 集群中运行的应用程序。

+   **数据传输暴露**：如果外部存储到 Kubernetes 的密钥传输没有得到妥善加密保护，例如缺乏端到端加密，传输过程中可能会暴露密钥。

+   **通过代理或中介提升权限**：获取密钥的代理或边车可能成为潜在的攻击入口。恶意攻击者若能访问其中一个代理，可能会从外部存储中窃取密钥。

+   **配置和访问策略**：外部密钥存储中的配置错误或过于宽松的访问策略可能会无意间暴露敏感密钥。

+   **版本控制和密钥轮换的挑战**：如果管理不当，Kubernetes 与外部存储之间的密钥版本同步可能会遇到挑战，导致潜在的版本不匹配或使用过时的密钥。

将 Kubernetes 与外部密钥存储集成可能具有挑战性。确保 Kubernetes 中密钥的安全性和完整性需要一系列强有力的实践。在这里，我们概述了需要考虑的关键最佳实践：

+   **保护数据传输**：在从外部存储将密钥传输到 Kubernetes 时，始终使用加密通道（如 TLS）。确保通信的双方相互认证。

+   **限制并监控访问**：在外部密钥存储中实现细粒度的访问控制。只允许特定的实体（如某些代理或 sidecar）获取密钥，并监控它们的活动。

+   **密钥轮换和同步**：定期轮换外部存储中的密钥，并确保有机制高效地将这些变化同步到 Kubernetes 中。这可以避免过时的密钥和潜在的安全漏洞。

+   **加强代理或中介系统的安全性**：如果使用代理、sidecar 或任何其他中介系统来获取密钥，确保它们是安全的、可监控的，并且以最小权限运行。

+   **备份外部存储**：定期备份外部密钥存储。在遭遇安全事件或故障时，这可以确保密钥恢复并快速使服务恢复上线。

+   **审计和异常检测**：利用外部密钥存储的审计功能。监控任何异常的访问模式或可能表明存在安全漏洞或配置错误的异常情况。

通过认识到这些影响并遵循最佳实践，Kubernetes 管理员可以有效且安全地利用外部密钥存储的优势。

# 实践与理论的平衡

在将 Kubernetes 与外部密钥存储集成时，找到正确的平衡至关重要。这种平衡不仅仅涉及技术层面，还包括可扩展性、审计能力、互操作性，甚至成本影响。目标是创建一个强大、可扩展且安全的环境，而不会妥协可用性或成本效益。

安全始终是最重要的。你必须确保密钥在传输过程中或静态存储时不被暴露。外部依赖如果没有得到妥善管理，可能会引入漏洞，单一的安全事件可能引发多米诺效应，危及多个系统。始终确保加密通信，并选择具有强大安全防护措施的密钥存储。

可用性和用户体验通常被视为安全的另一面。如果系统过于繁琐，可能会导致绕过或简化操作，从而抵消安全效益。此外，在评估如何实际应用这些考虑因素时，理解各种机制的最佳使用方式至关重要。基于 Pod 生命周期的方法，如 init 容器和 sidecar，自然与直接获取方法对接，而无需转化为 Kubernetes 原生密钥。相比之下，Kubernetes 扩展和 API 机制，虽然功能多样，但本质上更适合转化为 Kubernetes 资源。

细粒度访问对现代应用至关重要。并非每个应用或服务都需要访问所有秘密。正确实施的细粒度访问可以在某个服务被攻破时最小化风险。遗留系统并不能总是被忽视或立即替换。因此，任何解决方案都必须考虑如何与可能未按照现代安全实践设计的旧系统进行集成或共存。处理外部依赖是一项微妙的任务。过度依赖外部系统可能会给基础设施带来脆弱性。评估这些系统的可靠性并预设应急方案至关重要。

了解故障和恢复模型非常重要，因为故障发生的不是“如果”，而是“何时”；因此，必须制定全面的备份和恢复策略，以便在数据损坏或丢失时恢复秘密。处理潜在的秘密泄露影响范围至关重要。了解泄露的影响：如果一个节点或整个集群被攻破会发生什么？通过尽可能地分隔和隔离秘密，最大限度地减少潜在的损害。可审计性和监控确保秘密访问的可追溯性。全面的日志和实时警报有助于快速识别和纠正可疑活动。

秘密存储的可扩展性必须与组织的增长相匹配。随着集群和部署的增长，秘密存储应该能够无缝处理增加的流量。生命周期管理涉及在整个生命周期中管理秘密——包括创建、更新、轮换和删除——并将这些过程无缝集成到 CI/CD 流水线中。

在我们的多云时代，互操作性是不可谈判的。解决方案必须支持多种环境，确保跨不同云提供商的兼容性。成本不仅仅是直接的财务影响。需要考虑运营成本、潜在的泄露相关成本以及延迟相关的成本，确保解决方案的整体成本效益。地理冗余对于全球运营至关重要，确保从全球任何位置都能实现低延迟和高可用性。过渡的便捷性确保了未来的灵活性。通过优先考虑采用开放标准设计的解决方案，避免被锁定在某个特定的解决方案中。

最后，遵守特定行业的监管和合规要求，确保秘密存储符合 ISO 27001、PCI-DSS 和 HIPAA 等标准。

# 摘要

探索 Kubernetes 与外部秘密存储的集成，揭示了安全高效的秘密管理的基本方法和模式。我们深入研究了关键机制，包括 Kubernetes 扩展、Pod 生命周期操作，以及像 Secret Store CSI 驱动程序这样的创新工具，展示了 Kubernetes 的适应性和对安全性的承诺。

服务网格和代理机制在平衡强大安全性与应用灵活性方面发挥着关键作用，它们作为中介，负责密钥的分发，并将应用与直接访问密钥解耦。实现这一平衡需要关注细粒度的访问控制、遗留系统以及密钥泄露的潜在影响，同时还要满足可扩展性、监控和合规性等需求。

总之，将 Kubernetes 与外部密钥存储系统集成的这一复杂旅程，旨在创建一个强韧且安全的操作环境，确保为在 Kubernetes 生态系统中航行的组织提供可扩展且可持续的未来。

在我们探索将 Kubernetes 与外部密钥存储系统集成的基础上，下一章将呈现一个关于在生产环境中管理密钥生命周期的端到端故事。此章节将涵盖实际应用、挑战与解决方案，展示一种全面的方法，以确保在实际场景中安全高效地管理密钥。
