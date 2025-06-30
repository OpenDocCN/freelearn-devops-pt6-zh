# 10

# 使用基础设施即代码进行自动化

本章将探讨如何使用**基础设施即代码**（**IaC**）工具来自动化管理 Grafana 可观察性平台的各个组成部分。我们将重点介绍**Ansible**、**Terraform**和**Helm**，这些工具可以帮助团队以可重复和自动化的方式管理其系统的多个方面。本章将平台分为*收集和处理*层、*存储*层和*可视化*层，并概述如何自动化每个组件。本章将提供技术工具来创建一个易于管理且高度可扩展的可观察性平台，并结合*第十一章*中的信息，你将能够引领你的组织轻松地利用可观察性的强大功能。

本章将涵盖以下主要内容：

+   自动化 Grafana 的好处

+   介绍可观察性系统的组成部分

+   使用 Helm 或 Ansible 自动化收集基础设施

+   掌握 Grafana API

+   使用 Terraform 或 Ansible 管理仪表盘和警报

# 技术要求

本章涉及使用 Ansible、Terraform 和 Helm，建议在开始阅读之前先安装它们。本章还将讨论几个你应该至少有所了解的概念：

+   Kubernetes 对象

+   Kubernetes 操作员模式

# 自动化 Grafana 的好处

可观察性工具结合了来自许多应用程序、基础设施服务和其他系统组件的遥测数据的收集、存储和可视化。自动化为我们提供了一种可测试、可重复的方式来满足这些需求。使用 Helm、Ansible 和 Terraform 等行业标准工具有助于我们长期维护这些系统。使用自动化有很多好处，包括以下几点：

+   它减少了与手动流程相关的风险。

+   领域专家可以为开发人员交互的系统提供自动化服务。这使得开发团队有信心正确使用这些不熟悉的系统。这些知识的形式如下：

    +   遥测数据架构

    +   可重复的系统架构

    +   数据可视化管理的最佳实践

+   通过提供自动化，领域专家能够通过让团队使用更简单、更易用的系统自助服务，从而专注于更高价值的工作。

+   它提供了一条黄金路径，使得开发团队能够轻松、快速地采用可观察性，并能将更多时间专注于增值活动。

+   它使最佳实践和运营流程的扩展变得更加容易。这对于正在增长的组织尤为重要，因为一个可能适用于少数团队的流程无法扩展到数十个团队。

+   它确保成本信息始终可以追溯。

现在我们已经介绍了为什么你想使用自动化，接下来让我们看看构成可观察性平台的组件，这样我们就能轻松地自动化系统的不同方面。

# 介绍可观察性系统的组件

可观察性系统包含许多组件，涉及数据的生产、消费、转换、存储和使用。在本章中，我们将把这些组件分成四个不同的系统，以便明确我们在讨论可观察性平台的哪个方面。自动化的不同方面将吸引不同的受众。我们将讨论的系统如下：

1.  **数据生产系统**：这些是生成数据的系统。应用程序、基础设施，甚至数据收集系统的组件都会产生数据。我们来看看其关键特性：

    +   这些系统由开发人员如*Diego*或运营专家如*Ophelia*管理（有关这些角色的详细信息，请参见**第一章**）。

    +   这些系统作为应用程序或组件测试过程的一部分进行测试。如果使用数据模式，则可以通过诸如 JSON 模式之类的工具进行验证。

1.  **数据收集系统**：这些系统收集由数据生产系统生成的日志、度量和跟踪。它们通常提供用于转换数据的工具。它们的关键特性如下：

    +   这些系统通常由专业的运营团队、可观察性工程师、站点可靠性工程师或平台工程师管理。

    +   这些系统是配置的基础设施。

    +   自动化涉及使用基础设施即代码（IaC）工具和静态分析工具（如可用）来验证基础设施。

1.  **数据存储系统**：这些是用于存储数据并使其可搜索的系统。如果你的可观察性平台利用了 SaaS 工具，那么这些系统将由你的供应商提供。Loki、Prometheus、Mimir 和 Tempo 都是存储系统的例子。这些系统的一些重要特性包括：

    +   这些系统通常由专门的第三方管理，但当它们在组织内部管理时，通常会由与数据收集系统相同的团队管理。

    +   这些系统是配置的基础设施。

    +   自动化涉及使用 IaC 来配置本地资源，或者利用 SaaS 工具，如使用 Grafana Labs 与 IaC 配置。

1.  **数据可视化系统**：这些是允许用户搜索存储在存储系统中的数据，并生成可视化、警报和其他理解数据的方法的系统。Grafana 是一个可视化系统的例子。以下是此类系统的一些重要特性：

    +   这一层的管理通常是共享责任。管理特定系统的开发人员和运维人员应该被授权负责他们的仪表盘。管理数据收集和存储层的团队通常是赋能整个组织的团队。

    +   这些系统是预配置的基础设施。

    +   自动化涉及使用基础设施即代码（IaC）来预配置本地资源，或者利用像 Grafana Labs 这样的 SaaS 工具，结合 IaC 配置。

在本章中，我们将讨论前面列表中的*系统 2*、*系统 3* 和 *系统 4*。虽然*系统 1*很重要，但它是一个非常广泛的领域，且自动化策略对于不同类型的数据生产者有所不同。然而，在大多数情况下，团队可以依赖他们所使用的库进行的测试，或者他们运行的第三方系统。

让我们先来看看如何使用 Terraform 或 Ansible 来部署数据收集系统。

# 使用 Helm 或 Ansible 自动化收集基础设施

自动化安装用于收集遥测数据的基础设施是构建一个优秀可观察性平台的关键部分。支持这些功能的工具取决于你部署的基础设施。在这一部分，我们将通过以下工具来研究如何安装**OpenTelemetry Collector**和**Grafana Agent**：

+   **Helm** 是一个用于打包和管理 Kubernetes 应用程序的工具。Helm 图表包含了应用程序所需的各种 Kubernetes 组件的所有配置文件，通常处理应用程序的变量设置。我们将在 Kubernetes 环境中使用 Helm。

+   **Ansible** 是一个将操作标准化为可重复执行的剧本的工具。它使用简单的 YAML 配置文件来定义需要执行的操作，并利用 OpenSSH 连接到目标服务器以执行操作。我们将在虚拟或裸机环境中使用 Ansible，但它也可以用来管理 Kubernetes 环境。

重要提示

OpenTelemetry 和 Grafana 都提供了 Kubernetes 操作符，可以通过 Helm 安装。我们也会概述这些工具。

现在让我们看看如何使用 Helm 和 Ansible 来自动化安装 OpenTelemetry Collector 和 Grafana Agent。

## 自动化安装 OpenTelemetry Collector

在本书中，我们一直使用 OpenTelemetry Collector 从 OpenTelemetry 演示应用程序收集数据并将其发送到我们的 Grafana 实例。首先，我们将使用已经部署的配置，探索使用 OpenTelemetry 提供的 Helm 图表将 Collector 部署到 Kubernetes 集群中。

### OpenTelemetry Collector Helm 图表

我们首先在*第三章*安装了 OpenTelemetry Helm 图表，然后在*第四章*、*第五章*和*第六章*中更新了配置。OpenTelemetry 在其 Git 仓库中提供了关于可用配置选项的详细信息，地址为 https://github.com/open-telemetry/opentelemetry-helm-charts/tree/main/charts/opentelemetry-collector。

让我们看看我们在*第六章*中应用的最终配置，看看我们是如何配置 OTEL Helm 图表的。你可以在 Git 仓库中的`/chapter6/OTEL-Collector.yaml`文件中找到它。

我们将首先查看的配置块是`mode`，它描述了收集器将在 Kubernetes 中如何部署。可用的选项有`deployment`、`daemonset`和`statefulset`。在这里，我们使用`deployment`选项：

```
mode: deployment
```

让我们详细探索一下可用选项：

+   `deployment`会以固定数量的 Pod 进行部署，这就是 Kubernetes 中的`replicaCount`。在我们的参考系统中，我们使用了这种模式，因为我们知道系统将部署到单节点 Kubernetes 集群，并且它允许我们将通常在多节点集群中独立使用的预设组合起来。

+   一个`daemonset`会与一个收集器一起部署到集群中的每个节点。

+   `statefulset`会以独特的网络接口和一致的部署进行部署。

我们在讨论架构时会在*第十一章*中讨论如何选择适当的模式。这些部署模式也可以组合在一起，以在 Kubernetes 集群中提供特定功能。

接下来我们将查看的配置块是`presets`：

```
presets:
  logsCollection:
    enabled: true
    includeCollectorLogs: false
  kubernetesAttributes:
    enabled: true
  kubernetesEvents:
    enabled: true
  clusterMetrics:
    enabled: true
  kubeletMetrics:
    enabled: true
  hostMetrics:
    enabled: true
```

如你所见，这个配置只涉及启用或禁用不同的功能。让我们详细看看这些参数：

+   `logsCollection`参数告诉收集器从 Kubernetes 容器的标准输出收集日志。我们没有包括收集器日志，因为这可能会导致日志级联现象，即收集器读取它自己的日志输出并将收集的日志写入该输出，然后再读取。这种设置下，建议仅在`daemonset`模式下使用`logsCollection`参数。

+   `kubernetesAttributes`参数会在收集器接收日志、指标和追踪数据时收集 Kubernetes 元数据。这包括如`k8s.pod.name`、`k8s.namespace.name`和`k8s.node.name`等信息。属性收集器在所有模式中都是安全使用的。

+   `kubernetesEvents`参数会收集集群中发生的事件，并将它们发布到日志管道中。实际上，集群中发生的每个事件都会在 Loki 中收到一条日志条目，使用此配置。集群事件包括 Pod 创建和删除等。最佳实践是在`deployment`或`statefulset`模式下使用`kubernetesEvents`，以避免事件的重复。

+   三个指标选项会收集关于系统的指标：

    +   `clusterMetrics` 查看整个集群。应该在 `deployment` 或 `statefulset` 模式下使用。

    +   `kubeletMetrics` 收集来自 kubelet 的有关节点、Pod 和容器的指标。应在 `daemonset` 模式下使用。

    +   `hostMetrics` 直接从主机收集数据，例如 CPU、内存和磁盘使用情况。应在 `daemonset` 模式下使用。

我们将跳过一些标准的 Kubernetes 配置块，接下来考虑 `config` 块。`config` 块有几个子块：

+   遥测管道包括以下内容：

    +   `receivers`：接收器位于管道的开始部分。它们接收数据并进行转换，将数据添加到管道中，供其他组件使用。

    +   `processors`：处理器用于管道中执行各种功能。有支持的处理器和贡献的处理器可供使用。

    +   `exporters`：出口器位于管道的末端。它们接收以内部管道格式传递的数据，并进行转换以将数据发送到下一阶段。

    +   `connectors`：这些将 `receivers` 和 `exporters` 结合起来，连接管道。`connectors` 充当 `exporters`，将数据从一个管道发送到下一个管道，同时充当 `receivers`，接收数据并将其添加到另一个管道中。

+   与管道分开的是 `extensions`，它们为收集器添加额外的功能，但不需要访问管道中的遥测数据。

+   最后，还有一个 `service` 块，用于定义正在使用的管道和扩展。

我们在 `config` 块中使用的唯一扩展是 `health_check` 扩展：

```
config:
  extensions:
    health_check:
      check_collector_pipeline:
        enabled: false
```

这启用了一个端点，可用于 Kubernetes 集群中的存活性和/或就绪性探针。这对你来说非常有用，可以轻松查看收集器是否按预期工作。

在我们的 `receivers` 块中，我们配置了两个接收器：`otlp` 和 `prometheus`：

```
config:
  receivers:
    otlp:
      protocols:
        http:
          endpoint: 127.0.0.1:4318
          cors:
            allowed_origins:
              - "http://*"
              - "https://*"
    prometheus:
      config:
        scrape_configs:
          - job_name: 'opentelemetry-collector'
            tls_config:
              insecure_skip_verify: true
            scrape_interval: 10s
            scrape_timeout: 2s
            kubernetes_sd_configs:
              - role: pod
            relabel_configs:
              - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
                action: keep
                regex: "true"
              - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
                action: replace
                target_label: __address__
                regex: ([^:]+)(?::\d+)?;(\d+)
                replacement: $$1:$$2
```

让我们更仔细地看看这些接收器：

+   `OTLP` 接收器配置我们的收集器实例，在 Kubernetes 节点的 `127.0.0.1` 上暴露 `4318` 端口，这使得演示应用程序能够轻松地提交遥测数据。

+   `Prometheus` 接收器用于收集来自收集器本身的指标。此接收器配置显示了一个重新标记的示例，我们将 `meta_kuberentes_pod_annotation_prometheus_io_port` 重新命名为 `__address__`，这是 OTLP 中使用的标准。

在我们的配置中，我们设置了 `k8sattributes`、`resource` 和 `attributes` 处理器。`k8sattributes` 处理器从 kubelet 中提取属性，并将其添加到管道中的遥测数据中。`resource` 和 `attributes` 处理器分别插入或修改资源或属性。我们不会详细讨论这些概念，但资源用于标识生产遥测数据的源。

在我们的配置中，我们使用了 `spanmetrics` 和 `servicegraph` 两个连接器：

```
config:
  connectors:
    spanmetrics:
      dimensions:
      - name: http.method
        default: GET
      - name: http.status_code
      namespace: traces.spanmetrics
    servicegraph:
      latency_histogram_buckets: [1,2,3,4,5]
```

两个`connectors`用于从`traces`管道导出数据并将其接收在`metrics`管道中。`spanmetrics`收集`servicegraph`生成的度量，这些度量描述了服务之间的关系，这些度量使得服务图能够在 Tempo 中显示。

我们`config`块中的最后一个子块是`services`。这个子块定义了要加载的扩展和管道的配置。每个管道（`logs`、`metrics`和`traces`）定义了使用的`receivers`、`processors`和`exporters`。让我们来看一下`metrics`管道，因为它是最复杂的：

```
metrics:
        receivers:
          - otlp
          - spanmetrics
          - servicegraph
        processors:
          - memory_limiter
          - filter/ottl
          - transform
          - batch
        exporters:
          - prometheusremotewrite
          - logging
```

如前所述，receivers 是`OTLP`、`spanmetrics`和`servicegraph`。我们接着指示管道按照列出的顺序使用`memory_limiter`、`filter`、`transform`和`batch`处理器。你可能会注意到我们的过滤器命名为`ottl`，采用`processor/name`的语法，这在需要使用相同的处理器并配置不同参数时非常有用。最后，管道使用`prometheusremotewrite`的 exporters 并通过日志输出数据。

你可能已经注意到，`/chapter6/OTEL-Collector.yaml`文件中没有定义 exporters——这是因为它们在`/OTEL-Creds.yaml`中定义，这也突显了 Helm 的一个非常有用的功能，即能够根据功能分离配置文件。当我们安装 Helm chart 时，我们使用类似下面的命令：

```
helm install owg open-telemetry/opentelemetry-collector --values chapter3/OTEL-Collector.yaml --values OTEL-Creds.yaml
```

`-f` 或 `--values` 选项可以对多个 YAML 文件使用多次——如果存在冲突，则始终优先使用最后一个文件。通过这种方式结构化 YAML 文件，我们可以将完整配置拆分成不同部分，以保护敏感信息（如 API 密钥），同时仍然使主配置易于访问。我们还可以将此功能用于其他目的，比如在测试环境中覆盖默认配置。这里需要小心优先级问题，因为重复的数组不会合并。以这种方式部署 collector 在很多情况下非常棒。然而，它有一个限制——每当配置发生变化，或需要安装新版本的 collector 时，都需要进行 Helm 的`install`或`upgrade`操作。这就需要一个了解并可以访问每个集群的系统，才能部署 collector，这可能会带来瓶颈和安全风险。让我们来看看 OpenTelemetry 操作符，它提供了解决这些问题的方法。

### OpenTelemetry Kubernetes 操作符

OpenTelemetry 还提供了一个 Kubernetes 操作符来管理 OpenTelemetry Collector，并允许对工作负载进行自动化仪器化。这个操作符仍在积极开发中，预计功能集会增加。

Kubernetes Operator 使用 **自定义资源定义**（**CRDs**）为 Kubernetes API 提供扩展，这些扩展被 Operator 用来管理系统。使用 Operator 的优势包括以下几点：

+   管理 OpenTelemetry Operator 或自动化仪器化系统所涉及的复杂逻辑可以由 OpenTelemetry 项目中的专家设计。对于负责管理 OpenTelemetry 安装的人来说，Operator 提供了一个定义好的 CRD 规范，可以用来在 CI/CD 管道中验证提议的配置。

+   OpenTelemetry Operator 还允许有限的自动化升级。由于可能会有破坏性变更，次要和主要版本的更新仍需要通过 Helm 升级来应用。

+   它可以与 GitOps 工具结合使用，将解决方案从一个必须知道每个集群并具有必要凭证来部署到这些集群的中央系统，转变为一个每个集群从中央版本控制的仓库中读取所需配置的解决方案。

+   当涉及到通过添加注解使 OpenTelemetry 自动仪器化更容易被应用程序访问时，Operator 真正表现出色。对于大多数使用场景，自动仪器化将提供充足的度量和追踪遥测，以便了解应用程序。

OpenTelemetry Collector 也可以安装在虚拟机或裸金属服务器上，这个过程可以通过如 Ansible 之类的工具来自动化。让我们来看看如何处理这个问题。

### OpenTelemetry 和 Ansible

OpenTelemetry 不提供 Ansible 的官方集合。它分别为 Alpine、Debian 和 Red Hat 系统提供打包版本的 Collector，格式为 `.apk`、`.deb` 和 `.rpm` 文件。使用 `community.general.apk`、`ansible.builtin.apt` 或 `ansible.builtin.yum`，可以用类似以下配置的方式安装此包：

```
- name: Install OpenTelemetry Collector
  ansible.builtin.apt:
    deb: https://github.com/open-telemetry/opentelemetry-collector-releases/releases/download/v0.85.0/otelcol_0.85.0_linux_amd64.deb
```

安装好软件包后，配置收集器所需做的唯一其他事情是应用配置文件。默认配置文件位于 `/etc/otelcol/config.yaml`，并在 systemd 启动收集器时使用。Ansible 可以覆盖此文件或直接修改它。可以使用以下配置来完成这项工作：

```
- name: Copy OpenTelemetry Collector Configuration
  ansible.builtin.copy:
    src: /srv/myfiles/OTEL-Collector.yaml
    dest: /etc/otelcol/config.yaml
    owner: root
    group: root
    mode: '0644'
```

在本书中，我们已经多次讨论了 OpenTelemetry 代理。这样做的一个主要原因是，OpenTelemetry 还提供了我们用来生成真实样本数据的 Demo 应用程序。接下来，让我们来看看 Grafana Agent。

## 自动化安装 Grafana Agent

Grafana 生产了自己的代理，Grafana 推荐在其云平台中使用这个代理。Grafana Agent 还提供自动化选项，我们将在本节中介绍这些选项。

### Grafana Agent Helm charts

Grafana 提供了两个 Helm 图表，`grafana-agent` 图表和配置选项，请查看 Helm 图表文档 [`github.com/grafana/agent/tree/main/operations/helm/charts/grafana-agent`](https://github.com/grafana/agent/tree/main/operations/helm/charts/grafana-agent)。类似地，`agent-operator` 图表的文档可以在 [`github.com/grafana/helm-charts/tree/main/charts/agent-operator#upgrading-an-existing-release-to-a-new-major-version`](https://github.com/grafana/helm-charts/tree/main/charts/agent-operator#upgrading-an-existing-release-to-a-new-major-version) 找到。可用的 CRD 详细信息可以在操作员架构文档中找到 [`github.com/grafana/agent/blob/v0.36.2/docs/sources/operator/architecture.md`](https://github.com/grafana/agent/blob/v0.36.2/docs/sources/operator/architecture.md)。

### Grafana Agent 和 Ansible

与 OpenTelemetry 不同，Grafana 维护着一个 Ansible 集合（[`docs.ansible.com/ansible/latest/collections/grafana/grafana`](https://docs.ansible.com/ansible/latest/collections/grafana/grafana)），其中包含管理收集、存储和可视化系统的工具，我们将在后续章节中重新访问它。

包含的 `grafana_agent` 角色用于管理数据收集，并将代理安装在 Red Hat、Ubuntu、Debian、CentOS 和 Fedora 发行版上。此角色可以按如下方式使用：

```
- name: Install Grafana Agent
  ansible.builtin.include_role:
    name: grafana_agent
  vars:
    grafana_agent_logs_config:
      <CONFIG>
    grafana_agent_metrics_config:
      <CONFIG>
    grafana_agent_traces_config:
      <CONFIG>
```

日志、指标和追踪的配置特定于该遥测类型，Grafana 提供的文档涵盖了使用该配置来管理代理的内容。

# 熟悉 Grafana API

Grafana 提供了一个功能齐全的 API，适用于 Grafana Cloud 和 Grafana 本身。这个 API 与前端使用的相同，这意味着我们也可以通过直接的 API 调用或使用类似 **Terraform** 的 IaC 工具来操作 Grafana 的功能。我们将首先对 Grafana Cloud 和 Grafana 中可用的 API 进行高层次的了解，然后我们将研究 Grafana 的 Terraform 模块和 Ansible 集合，看看如何使用它们来管理 Grafana Cloud 实例。

## 探索 Grafana Cloud API

Grafana Cloud API 用于管理 Grafana Cloud SaaS 安装的各个方面。让我们高层次地了解一下 Grafana Cloud API 提供的功能：

+   `accessPolicyId`，即访问策略对象的唯一 ID。

+   **堆栈**：这些端点管理 Grafana Cloud 堆栈。以下是它们的主要功能：

    +   堆栈的创建、读取、更新和删除功能

    +   重新启动特定堆栈上的 Grafana

    +   列出特定堆栈上的数据源

+   **Grafana 插件**：这些端点管理与堆栈相关的 Grafana 实例上安装的插件。它们的功能包括创建、读取、更新和删除安装在特定堆栈上的 Grafana 插件的功能。

+   **区域**：这些 API 端点列出了可用的 Grafana Cloud 区域。它们用于读取可以用来托管堆栈的可用 Grafana Cloud 区域的功能。

+   **API 密钥**：这些端点用于管理云 API 密钥，其主要功能是创建、读取和删除 API 密钥。这些端点现已弃用，因为 Grafana 已转向使用基于访问策略和令牌的认证技术。API 密钥端点将在未来的更新中被移除。

我们将在 *第十一章* 中更详细地讨论访问策略和令牌，其中我们会讨论 **访问级别**，作为构建一个优秀可观察性平台的一部分。所有 Grafana Cloud 端点都有一个关联的访问策略，且所使用的令牌必须获得该策略的授权才能成功响应。

更详细的信息可以在 Grafana 的文档中找到，网址为 https://grafana.com/docs/grafana-cloud/developer-resources/api-reference/cloud-api/，其中包括请求中所需的参数和详细信息，以及示例请求和响应。

我们现在已经回顾了 API 端点。接下来，我们将讨论 Grafana 提供的 Terraform 提供者和 Ansible 集合，这些工具可以用于通过基础设施即代码（IaC）自动化与上述 API 进行交互。

## 使用 Terraform 和 Ansible 管理 Grafana Cloud

Grafana 提供了 Terraform 提供者和 Ansible 集合，用于管理组织的云实例。让我们来探讨如何利用这些工具通过 Grafana Cloud API 来管理 Grafana Cloud 实例。

### Grafana Terraform 提供者

**Grafana Terraform 提供者**具有与我们讨论过的云 API 端点相对应的资源。该提供者还提供用于其他 Grafana API 端点的资源，我们将在本章后面讨论管理仪表板和警报时介绍这些资源。该提供者的官方文档可以在 Terraform Registry 上找到，网址为 [`registry.terraform.io/providers/grafana/grafana/latest/docs`](https://registry.terraform.io/providers/grafana/grafana/latest/docs)。

重要提示

本章是基于 Grafana Terraform 提供者的 2.6.1 版本编写的。

以下是一些常用的 Terraform 资源，并附有使用示例：

+   首先，让我们看一下如何使用提供者。`grafana_cloud_stack` 数据提供者用于查找一个名为 `acme-preprod` 的堆栈，我们稍后将使用它来指定我们的访问策略创建位置：

    ```
    data "grafana_cloud_stack" "preprod" {
      slug = "acme-preprod"
    }
    ```

+   `grafana_cloud_access_policy` 资源允许我们创建一个访问策略。在这里，我们将 `region` 值设置为 `us`，并指定名称和显示名称。最后，我们指定策略的作用范围——在这种情况下，我们希望能够写入日志、指标和追踪。然后，我们之前找到的 `stack` ID 将用于指定在哪里创建此访问策略：

    ```
    resource "grafana_cloud_access_policy" "collector-write" {
      region       = "us"
      name         = "collector-write"
      display_name = "Collector write policy"
      scopes = ["logs:write", "metrics:write", "traces:write"]
      realm {
        type       = "stack"
        identifier = data.grafana_cloud_stack.preprod.id
      }
    }
    ```

+   最后，`grafana_cloud_access_policy_token` 资源可用于创建一个新的令牌。我们指定区域、要使用的访问策略和名称。然后，令牌可以通过 `grafana_cloud_access_policy_token.collector-token.token` 读取：

    ```
    resource "grafana_cloud_access_policy_token" "collector-token" {
      region           = "us"
      access_policy_id = grafana_cloud_access_policy.collector-write.policy_id
      name             = "preprod-collector-write"
      display_name     = "Preprod Collector Token"
      expires_at       = "2023-01-01T00:00:00Z"
    }
    ```

这不是我们使用 Grafana Terraform 提供者能做的唯一事情：当我们讨论管理仪表盘和警报时，本章稍后会考虑另一个示例。通常，这会与另一个提供者结合使用，将新创建的令牌记录在秘密管理工具中，例如 **AWS Secrets Manager** 或 **HashiCorp Vault**，以便在部署收集器时可以访问。

让我们看看如何使用 Grafana 提供的另一个 IaC 工具——**Grafana** **Ansible 集合**来管理 Grafana Cloud 系统。

### Grafana Ansible 集合

Grafana Ansible 集合的功能不如 Terraform 提供者那样丰富，尤其是在管理云实例时。然而，可以通过 Ansible URI 模块访问许多来自 Grafana Cloud API 的功能。官方集合文档可以在 Ansible 网站上找到，链接为：[`docs.ansible.com/ansible/latest/collections/grafana/grafana/`](https://docs.ansible.com/ansible/latest/collections/grafana/grafana/)。社区提供的集合也可用，但在此不做讨论。相关文档请参见：[`docs.ansible.com/ansible/latest/collections/community/grafana/index.html`](https://docs.ansible.com/ansible/latest/collections/community/grafana/index.html)。

重要提示

本章是在 Grafana Ansible 集合的 2.2.3 版本下编写的。

我们将通过 Ansible 管理 Grafana Cloud 堆栈。`name` 和 `stack_slug`（这是我们正在交互的堆栈）值按照惯例被设置为相同的字符串。然后我们需要设置堆栈的 `region` 值，以及使用 `org_slug` 设置堆栈将属于的组织：

```
- name: Create preprod stack
  grafana.grafana.cloud_stack:
    name: acme-preprod
    stack_slug: acme-preprod
    cloud_api_key: "{{ grafana_cloud_api_key }}"
    region: us
    org_slug: acme
    state: present
```

到目前为止，我们已经看过了 Grafana Cloud 的 API。这个 API 非常适合管理 Grafana Cloud 中的可观察性平台，但许多团队可能更关注在 Grafana UI 中管理仪表盘、警报和其他项目。Grafana 还提供了一个 API 来管理 Grafana UI 中的对象——让我们现在来看看。

## 探索 Grafana API

虽然 Grafana Cloud API 仅用于管理 Grafana Cloud SaaS 实例，但 **Grafana API** 的覆盖面非常广泛，因为 Grafana 有很多功能。这些 API 可以用于 Grafana Cloud 和本地安装的 Grafana 实例。

与 Grafana Cloud API 类似，所有端点都使用基于角色的访问控制。然而，Grafana API 提供了额外的认证选项：服务帐户。服务帐户应该用于任何需要与 Grafana 交互的应用程序。

由于大多数团队会频繁使用一小部分 API，因此我们这里只讨论一些常见的 API。然而，还有很多其他 API 可以用来自动化管理 Grafana 实例。让我们深入了解一些常用的 API：

+   **仪表盘和文件夹**：这些端点用于管理 Grafana 中的仪表盘和文件夹。它们的功能包括：

    +   创建、读取、更新和删除仪表盘或文件夹的功能

    +   创建、读取、更新和删除仪表盘上的标签

    仪表盘和文件夹都有 ID 和 UID。ID 仅在特定的 Grafana 安装中是唯一的，而 UID 在不同的安装中是唯一的。

+   `1` = `查看`，`2` = `编辑`，`4` = `管理员`。可以为用户角色或 `teamId` 值设置权限。

+   **文件夹/仪表盘搜索**：此 API 允许用户搜索仪表盘和文件夹。该端点允许通过查询参数进行复杂的搜索。响应是一个包含对象 UID 的匹配对象列表。

+   **团队**：这些端点管理 Grafana 中的团队。可以用来执行以下操作：

    +   创建、读取、更新和删除团队

    +   获取、添加和移除团队成员

    +   获取并更新团队偏好设置

+   **警报**：这些复杂的 API 管理警报的所有方面。此 API 管理与 Grafana Alertmanager 相关的所有内容。可以用来创建、读取、更新和删除警报、警报规则、警报组、静默、接收器、模板以及更多的警报对象。

这些 API 端点非常适合管理 Grafana。Grafana 提供了详细的 API 参考，链接为 [`grafana.com/docs/grafana/latest/developers/http_api/`](https://grafana.com/docs/grafana/latest/developers/http_api/)。

现在让我们看看这些 API 端点如何允许我们使用 Terraform 和 Ansible 的基础设施即代码（IaC）工具来管理仪表盘和警报。

# 使用 Terraform 或 Ansible 管理仪表盘和警报

由于仪表盘通常由负责某个服务或应用程序的团队管理，因此最佳实践是将部署仪表盘的工具与管理可观察性基础设施的工具分开。我们将在 *第十四章* 中讨论这个问题的实际操作。

在管理仪表盘时，Terraform 和 Ansible 都利用 Grafana 仪表盘是 JSON 对象的事实，提供了一种将包含仪表盘配置的 JSON 文件上传到 Grafana 实例的机制。让我们看看它是如何工作的。

**Terraform** 代码如下：

```
resource "grafana_dashboard" "top_level " {
  config_json = file("top-level.json")
  overwrite = true
}
```

一组仪表盘 JSON 文件可以通过 Terraform 的 `fileset` 函数和 `for_each` 命令进行迭代。这使得团队能够通过将正确的仪表盘保存到相关文件夹中，自动化地管理所有仪表盘。

**Ansible** 集合的工作方式非常相似：

```
- name: Create Top Level Dashboard
  grafana.grafana.dashboard:
    dashboard: "{{ lookup('ansible.builtin.file', ' top-level.json') }}"
    grafana_url: "{{ grafana_url }}"
    grafana_api_key: "{{ grafana_api_key }}"
    state: present
```

与 Terraform 代码类似，这也可以使用内置的 `with_fileglob` 函数进行迭代，使得团队能够以自动化的方式管理所有的仪表盘。

不幸的是，由于 Grafana 在警报管理方面的最新变化，Ansible 集合尚未更新以支持警报管理。使用 Terraform，您可以以与仪表盘非常相似的方式管理 Grafana 警报。请考虑以下示例代码块：

```
resource "grafana_rule_group" "ateam_alert_rule" {
  name             = "A Team Alert Rules"
  folder_uid       = grafana_folder.rule_folder.uid
  interval_seconds = 240
  org_id           = 1
  rule {
    name           = "Alert Rule 1"
  }
  rule {
    name           = "Alert Rule 2"
  }
```

我们没有包含这里展示的两个规则的完整细节，因为所需的配置块过大。Terraform 文档中有一个非常清晰的完整警报示例，链接为 [`registry.terraform.io/providers/grafana/grafana/latest/docs/resources/rule_group`](https://registry.terraform.io/providers/grafana/grafana/latest/docs/resources/rule_group)。

类似于 `grafana_dashboard` 资源，`grafana_rule_group` 可以通过使用 `dynamic` 块来遍历， 从其他源（例如 JSON 文件）填充每个规则。这样，管理这些规则变得更加用户友好。

# 总结

在本章中，你了解了自动化管理观察平台的好处，并看到投资于优秀的自动化可以让主题专家将重复性和低价值的工作交给组织中的其他人。我们讨论了观察平台的不同方面，包括数据生产、收集、存储和可视化。你还学习了通常由谁负责平台的每个方面。

理论部分大致讲解完后，我们继续讨论如何管理数据收集层，深入分析了用于收集本书中所有数据的 OpenTelemetry Collector Helm 配置。我们对比了 Helm 与 Ansible 在虚拟或物理环境部署中的工作方式，并帮助你掌握了理解每个工具管理文件结构的宝贵技能。我们通过介绍 Helm 图表和用于 Grafana Agent 的 Ansible 集合，完善了数据收集系统的自动化。虽然我们没有像 OpenTelemetry 配置那样深入讲解，但管理 Grafana Agent 所需的技能是相同的。

我们接下来的主题是 Grafana API，在这里你了解到有两个 API，一个用于管理 SaaS Grafana Cloud 解决方案，另一个用于管理 Grafana 实例（包括云端和本地）。然后，你学习了如何通过特定示例使用 Terraform 提供程序来管理云堆栈和 Grafana 实例。接着，我们也看了 Grafana Ansible 集合，了解了它如何用来管理云堆栈和 Grafana 实例，以及数据收集层。

在本书的下一章中，我们将讨论如何架构一个完整的观察平台，能够根据组织的需求进行扩展。
