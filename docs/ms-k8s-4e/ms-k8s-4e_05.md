

# 第五章：在实践中使用 Kubernetes 资源

在本章中，我们将设计一个虚构的大规模平台，挑战 Kubernetes 的能力和可扩展性。Hue 平台旨在创造一个全知全能的数字助手。Hue 是你的数字延伸。Hue 会帮助你做任何事情，找到任何东西，并且在许多情况下，会代表你做很多事情。显然，它需要存储大量信息，集成许多外部服务，响应通知和事件，并在与你互动时非常智能。

本章将借此机会让我们更好地了解 kubectl 及相关工具，深入探讨我们之前见过的熟悉资源（如 Pods），以及一些新资源（如 Jobs）。我们将探讨高级调度和资源管理。

本章将涉及以下主题：

+   设计 Hue 平台

+   使用 Kubernetes 构建 Hue 平台

+   分离内部和外部服务

+   高级调度

+   使用命名空间限制访问

+   使用 Kustomization 构建层次化集群结构

+   启动任务

+   混合非集群组件

+   使用 Kubernetes 管理 Hue 平台

+   使用 Kubernetes 发展 Hue 平台

为了明确，这只是一个设计练习！我们并不打算真正构建 Hue 平台。这个练习的目的是展示 Kubernetes 在大规模系统中，处理多个活动部分时所能提供的广泛功能。

在本章结束时，你将清楚地看到 Kubernetes 的强大，并了解它如何作为构建复杂系统的基础。

# 设计 Hue 平台

在本节中，我们将为令人惊叹的 Hue 平台奠定基础，并定义其范围。Hue 不是“大哥”；Hue 是“小弟”！Hue 会做你允许它做的任何事情。Hue 能做很多事，这可能会让一些人感到担忧，但你可以选择让 Hue 帮助你多少。准备好迎接一场疯狂的旅程吧！

## 定义 Hue 的范围

Hue 将管理你的数字身份。它会比你更了解你自己。以下是 Hue 可以管理和帮助你的部分服务列表：

+   搜索和内容聚合

+   医疗 – 电子健康记录，DNA 测序

+   智能家居

+   财务 – 银行业务，储蓄，退休，投资

+   办公室

+   社交

+   旅行

+   健康

+   家庭

让我们来看看 Hue 平台的一些功能，例如智能提醒和通知、安全、身份和隐私。

### 智能提醒和通知

让我们想象一下各种可能性。Hue 会了解你，也了解你的朋友，以及所有领域中其他用户的集合。Hue 会实时更新其模型。它不会被过时的数据所困扰。它会代表你行动，呈现相关信息，并不断学习你的偏好。它可以推荐你可能喜欢的新节目或书籍，根据你和你家人或朋友的日程安排预订餐厅，甚至控制你家的自动化系统。

### 安全性、身份和隐私

Hue 是你的在线代理。有人窃取你的 Hue 身份，甚至只是窃听你与 Hue 的互动，将带来灾难性后果。潜在用户可能甚至不愿意将他们的身份交给 Hue 组织。让我们设计一个无信任系统，使用户随时拥有切断 Hue 的权力。以下是一些想法：

+   通过专用设备实现强身份验证，采用多因素授权，包括多个生物识别因素

+   定期更换凭证

+   快速暂停服务并验证所有外部服务的身份（将要求每个服务提供商提供原始身份凭证）

+   Hue 后端将通过短生命周期令牌与所有外部服务进行交互

+   将 Hue 架构设计为松耦合微服务的集合，并确保强大的隔离性

+   GDPR 合规性

+   端到端加密

+   避免拥有关键数据（让外部服务提供商来管理）

Hue 的架构需要支持巨大的变化和灵活性。它还需要非常具有扩展性，因为现有功能和外部服务会不断升级，新的功能和外部服务也会不断集成到平台中。这种规模要求使用微服务架构，每个功能或服务都与其他服务完全独立，除非通过标准和/或可发现的 API 定义了良好的接口。

## Hue 组件

在开始我们的微服务之旅之前，让我们回顾一下我们需要为 Hue 构建的组件类型。

### 用户档案

用户档案是一个主要组件，包含许多子组件。它代表用户的本质、他们的偏好、他们在各个领域的历史记录，以及 Hue 所知道的所有信息。你能从 Hue 获取的收益在很大程度上取决于档案的丰富程度。但档案管理的信息越多，若数据（或部分数据）被泄露，将带来越大的损害。

管理用户档案的一个重要部分是 Hue 为用户提供的报告和洞察。Hue 将采用先进的机器学习技术，更好地了解用户及其与其他用户和外部服务提供商的互动。

### 用户图谱

用户图组件建模了跨多个领域用户之间的交互网络。每个用户都参与多个网络：如 Facebook、Instagram 和 Twitter 等社交网络；专业网络；兴趣爱好网络；以及志愿者社区。这些网络中的一些是临时性的，Hue 将能够结构化它们以造福用户。即使不暴露私密信息，Hue 也能利用其对用户连接的丰富资料来改善交互。

### 身份

如前所述，身份管理至关重要，因此需要一个单独的组件。用户可能希望管理多个相互排斥的个人档案，并拥有不同的身份。例如，用户可能不愿将自己的健康档案与社交档案混合，担心无意间将个人健康信息暴露给朋友。虽然 Hue 可以为你找到有用的连接，但你可能更倾向于在更多隐私和更高能力之间做出权衡。

### 授权器

授权器是一个关键组件，用户明确授权 Hue 执行某些操作或代表其收集各种数据。这涉及到访问物理设备、外部服务账户和操作权限等方面。

### 外部服务

Hue 是外部服务的聚合器。它并不是用来替代你的银行、健康服务提供者或社交网络的。它将保留大量关于你活动的元数据，但内容仍将保留在外部服务中。每个外部服务都需要一个专门的组件来与外部服务 API 和政策进行交互。当没有可用 API 时，Hue 会通过自动化浏览器或本地应用程序来模拟用户行为。

### 通用传感器

Hue 的一个重要价值 proposition 是代表用户执行操作。为了有效执行此操作，Hue 需要了解各种事件。例如，如果 Hue 为你预订了假期，但它检测到有更便宜的航班，它可以自动更改你的航班或要求你确认。有无数的事件需要感知。为了控制感知范围，需要一个通用传感器。通用传感器将是可扩展的，但它暴露出一个通用接口，其他 Hue 组件即使在添加更多传感器时，也可以统一使用该接口。

### 通用执行器

这是通用传感器的对应组件。Hue 需要代表你执行某些操作；例如，预订航班或预约医生。为了做到这一点，Hue 需要一个通用执行器，该执行器可以扩展以支持特定功能，但能以统一的方式与其他组件（如身份管理器和授权器）进行交互。

### 用户学习者

这是 Hue 的大脑。它将持续监控你所有授权的互动，并更新其对你和你网络中其他用户的模型。这将使 Hue 随着时间的推移变得越来越有用，能够预测你的需求和兴趣，提供更好的选择，在合适的时间展示更相关的信息，避免令人烦恼和过于强势。

### Hue 微服务

每个组件的复杂性都非常庞大。一些组件，比如外部服务、通用传感器和通用执行器，将需要跨越数百、数千甚至更多的外部服务，这些服务会不断变化，超出 Hue 的控制范围。即使是用户学习系统，也需要在多个领域和范畴中学习用户的偏好。微服务通过允许 Hue 渐进式发展并在不因其自身复杂性而崩溃的情况下，增长出更多独立的功能来解决这一需求。每个微服务通过标准接口与通用的 Hue 基础设施服务进行交互，并且可以通过明确定义且版本化的接口与少数其他服务进行交互。每个微服务的表面面积是可管理的，微服务之间的编排遵循标准的最佳实践。

### 插件

插件是扩展 Hue 的关键，而无需大量界面。关于插件的一个特点是，你通常需要跨越多个抽象层的插件链。例如，如果你想为 Hue 添加 YouTube 集成功能，那么你可以收集大量特定于 YouTube 的信息——你的频道、收藏的视频、推荐内容以及你观看过的视频。为了向用户展示这些信息并允许他们进行操作，你需要跨多个组件的插件，并最终出现在用户界面中。智能设计将通过汇总推荐、选择和延迟通知等行为类别，来帮助许多服务进行聚合。

插件的一个重要优点是它们可以由任何人开发。最初，Hue 开发团队将需要开发这些插件，但随着 Hue 的流行，外部服务将希望与 Hue 集成并构建 Hue 插件以启用他们的服务。当然，这将导致一个完整的插件注册、审批和管理生态系统。

### 数据存储

Hue 将需要几种类型的数据存储，每种类型还需要多个实例，以管理其数据和元数据：

+   关系型数据库

+   图形数据库

+   时序数据库

+   内存缓存

+   Blob 存储

由于 Hue 的规模，每个数据库都必须进行集群化、可扩展性和分布式处理。此外，Hue 还将使用边缘设备上的本地存储。

### 无状态微服务

微服务应该大多是无状态的。这将允许特定实例快速启动和终止，并根据需要在基础设施中迁移。状态将由存储管理，微服务通过短期访问令牌访问这些状态。Hue 将在适当的时候将频繁访问的数据存储在易于加速的缓存中。

### 无服务器函数

每个用户的 Hue 功能的大部分将涉及与外部服务或其他 Hue 服务的相对较短的交互。对于这些活动，可能无需运行一个完整的持久性微服务，也不需要扩展和管理它。更合适的解决方案可能是使用更轻量的无服务器函数。

### 事件驱动交互

所有这些微服务需要相互通信。用户将请求 Hue 代表他们执行任务。外部服务将通知 Hue 各种事件。队列与无状态微服务的结合提供了完美的解决方案。

每个微服务的多个实例将监听不同的队列，并在相关事件或请求从队列中弹出时作出响应。特定事件也可能触发无服务器函数。这种安排非常健壮且易于扩展。每个组件都可以是冗余的并且具有高可用性。虽然每个组件都有可能发生故障，但系统本身非常容错。

队列也可以用于异步的 RPC 或请求-响应风格的交互，其中调用实例提供一个私有队列名，响应将被发布到该私有队列。

也就是说，有时候通过一个明确界定的接口进行直接的服务对服务交互（或无服务器函数对服务交互）更为合适，并能简化架构。

## 规划工作流

Hue 经常需要支持工作流。一个典型的工作流会处理一个高层次的任务，比如预约看牙医。它会提取用户的牙医信息和日程安排，匹配用户的日程，选择多个选项中的一个，可能需要与用户确认，完成预约并设置提醒。我们可以将工作流分为完全自动化工作流和涉及人类的人工工作流。还有一些工作流涉及花费金钱，可能需要额外的审批层级。

### 自动化工作流

自动化工作流无需人工干预。Hue 具有从头到尾执行所有步骤的完全权限。用户分配给 Hue 的自主权越大，Hue 的效果就越好。用户将能够查看并审计所有的工作流，无论是过去的还是当前的。

### 人工工作流

人类工作流需要与人类互动。最常见的是用户需要从多个选项中做出选择或批准某个操作。但它也可能涉及其他服务中的人。例如，要与牙医预约，Hue 可能需要从秘书那里获取可用时间列表。在未来，Hue 将能够处理与人类的对话，并可能自动化一些这些工作流。

### 预算感知工作流

一些工作流，如支付账单或购买礼物，涉及花费金钱。虽然理论上可以给 Hue 授权访问用户的银行账户，但大多数用户可能会更愿意为不同的工作流设置预算，或者仅将支出作为一个人类批准的活动。用户可能会授予 Hue 访问一个专用账户或一组账户的权限，并根据提醒和报告，按需为 Hue 分配更多或更少的资金。

到目前为止，我们已经覆盖了很多内容，查看了构成 Hue 平台及其设计的不同组件。现在是时候看看 Kubernetes 如何帮助构建像 Hue 这样的平台了。

# 使用 Kubernetes 构建 Hue 平台

在本节中，我们将查看各种 Kubernetes 资源以及它们如何帮助我们构建 Hue。首先，我们将更好地了解多功能的 kubectl，然后我们将了解如何在 Kubernetes 中运行长时间运行的进程，如何内部和外部暴露服务，如何使用命名空间限制访问，如何启动临时作业，以及如何混合非集群组件。显然，Hue 是一个庞大的项目，因此我们将在本地集群上演示这些想法，而不实际构建一个真正的 Hue Kubernetes 集群。将其视为一种思维实验。如果你希望探索如何在 Kubernetes 上构建一个真正的微服务分布式系统，可以查看 *Hands-On Microservices with Kubernetes*：[`www.packtpub.com/product/hands-on-microservices-with-kubernetes/9781789805468`](https://www.packtpub.com/product/hands-on-microservices-with-kubernetes/9781789805468)。

## 高效使用 kubectl

kubectl 是你的瑞士军刀。它几乎可以完成集群中的任何任务。在后台，kubectl 通过 API 连接到你的集群。它读取你的 `~/.kube/config` 文件（默认情况下，这可以通过 `KUBECONFIG` 环境变量或 `--kubeconfig` 命令行参数来覆盖），该文件包含连接到你的集群或多个集群所需的信息。命令被分为多个类别：

+   **通用命令**：以通用方式处理资源：`create`、`get`、`delete`、`run`、`apply`、`patch`、`replace` 等

+   **集群管理命令**：处理节点和整个集群的任务：`cluster-info`、`certificate`、`drain` 等

+   **故障排除命令**：`describe`、`logs`、`attach`、`exec` 等

+   **部署命令**：处理部署和扩展的任务：`rollout`、`scale`、`auto-scale` 等

+   **设置命令**: 处理标签和注释：`label`、`annotate` 等

+   **杂项命令**: `help`、`config` 和 `version`

+   **自定义命令**: 将 kustomize.io 的功能集成到 kubectl 中

+   **配置命令**: 处理上下文，切换集群和命名空间，设置当前上下文和命名空间，等等

你可以使用 Kubernetes 的 `config view` 命令查看配置。

这是我本地 KinD 集群的配置：

```
$ k config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://127.0.0.1:50615
  name: kind-kind
contexts:
- context:
    cluster: kind-kind
    user: kind-kind
  name: kind-kind
current-context: kind-kind
kind: Config
preferences: {}
users:
- name: kind-kind
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED 
```

你的 `kubeconfig` 文件可能与上面的代码示例相似，也可能不同，只要它指向一个正在运行的 Kubernetes 集群，你就可以跟随学习。让我们深入了解 kubectl 清单文件。

## 理解 kubectl 清单文件

许多 kubectl 操作，例如 `create`，需要一个复杂的层级结构（因为 API 要求此结构）。kubectl 使用 YAML 或 JSON 清单文件。YAML 更简洁且易于人类阅读，所以我们大多使用 YAML。下面是一个用于创建 pod 的 YAML 清单文件：

```
apiVersion: v1
kind: Pod
metadata:
  name: ""
  labels:
    name: ""
  namespace: ""
  annotations: []
  generateName: ""
spec:
     ... 
```

让我们来看看清单中的各个字段。

### apiVersion

非常重要的 Kubernetes API 持续发展，可以通过不同版本的 API 支持同一资源的不同版本。

### kind

`kind` 告诉 Kubernetes 它正在处理的资源类型；在这种情况下是 `Pod`。这是必需的。

### metadata

`metadata` 包含了描述 pod 及其操作位置的大量信息：

+   `name`: 在命名空间中唯一标识 pod

+   `labels`: 可以应用多个标签

+   `namespace`: pod 所属的命名空间

+   `annotations`: 可查询的注释列表

### spec

`spec` 是一个 pod 模板，包含启动 pod 所需的所有信息。它可能相当复杂，因此我们将分多个部分来探讨：

```
spec:
  containers: [
      ...
  ],
  "restartPolicy": "",
  "volumes": [] 
```

#### 容器规格

pod spec 的 `containers` 部分是一个容器规格列表。每个容器规格具有以下结构：

```
name: "",
image: "",
command: [""],
args: [""],
env:
    - name: "",
      value: ""
imagePullPolicy: "",
ports: 
    - containerPort": 0,
      name: "",
      protocol: ""
resources:
  requests:
    cpu: ""
    memory: ""
  limits:
    cpu: ""
    memory: "" 
```

每个容器都有一个 `image`，一个命令，如果指定的话，将替代 Docker 镜像命令。它还可以有参数和环境变量。当然，还有镜像拉取策略、端口和资源限制。我们在前面的章节中已经介绍过这些内容。

如果你想深入探索 pod 资源或其他 Kubernetes 资源，那么以下命令将非常有用：`kubectl explain`。

它可以探索资源以及特定的子资源和字段。

尝试以下命令：

```
kubectl explain pod
kubectl explain pod.spec 
```

## 在 pods 中部署长时间运行的微服务

长时间运行的微服务应该运行在 pods 中，并且是无状态的。让我们看看如何为 Hue 的一个微服务——Hue 学习器——创建 pods，Hue 学习器负责学习用户在不同领域的偏好。稍后，我们将提高抽象级别，使用部署（deployment）。

### 创建 pods

让我们从一个普通的 Pod 配置文件开始，来创建一个 Hue 学习器内部服务。该服务不需要公开暴露，它将监听一个队列以接收通知，并将其见解存储在持久存储中。

我们需要一个简单的容器来在 Pod 中运行。以下是可能是最简单的 Docker 文件，它将模拟 Hue 学习器：

```
FROM busybox
CMD ash -c "echo 'Started...'; while true ; do sleep 10 ; done" 
```

它使用 `busybox` 基础镜像，打印 `Started...` 到标准输出，然后进入无限循环，这在所有情况下都是长期运行的。

我已经构建了两个 Docker 镜像，标签为 `g1g1/hue-learn:0.3` 和 `g1g1/hue-learn:0.4`，并将它们推送到 Docker Hub 注册表（`g1g1` 是我的用户名）：

```
$ docker build . -t g1g1/hue-learn:0.3
$ docker build . -t g1g1/hue-learn:0.4
$ docker push g1g1/hue-learn:0.3
$ docker push g1g1/hue-learn:0.4 
```

现在这些镜像可以拉取到 Hue 的 Pod 中的容器里。

我们将在这里使用 YAML，因为它更加简洁和易读。以下是模板和元数据标签：

```
apiVersion: v1
kind: Pod
metadata:
  name: hue-learner
  labels:
    app: hue
    service: learner
    runtime-environment: production
    tier: internal-service 
```

接下来是重要的 `containers` 规格，它为每个容器定义了必需的名称和镜像：

```
spec:
  containers:
  - name: hue-learner
    image: g1g1/hue-learn:0.3 
```

`resources` 部分告诉 Kubernetes 容器的资源需求，从而实现更高效、更紧凑的调度和分配。在这里，容器请求 200 毫 CPU 单位（0.2 核）和 256 MiB（2 的 28 次方字节）：

```
 resources:
      requests:
        cpu: 200m
        memory: 256Mi 
```

`environment` 部分允许集群管理员提供将可用于容器的环境变量。在这里，它告诉容器通过 DNS 查找队列和存储。在测试环境中，可能使用不同的发现方法：

```
 env:
    - name: DISCOVER_QUEUE
      value: dns
    - name: DISCOVER_STORE
      value: dns 
```

### 给 Pod 添加标签

明智地标记 Pod 对灵活操作至关重要。它让你可以实时演进集群，将微服务组织成可以统一操作的组，并且可以快速深入观察不同的子集。

例如，我们的 Hue 学习器 Pod 具有以下标签（以及其他一些标签）：

+   `runtime-environment : production`

+   `tier : internal-service`

`runtime-environment` 标签允许对属于特定环境的所有 Pod 执行全局操作。`tier` 标签可以用于查询属于特定层级的所有 Pod。这些仅仅是示例，你的想象力才是限制。

下面是如何使用 `get pods` 命令列出标签的方法：

```
$ k get po -n kube-system --show-labels
NAME                                         READY   STATUS    RESTARTS   AGE    LABELS
coredns-64897985d-gzrm4                      1/1     Running   0          2d2h   k8s-app=kube-dns,pod-template-hash=64897985d
coredns-64897985d-m8nm9                      1/1     Running   0          2d2h   k8s-app=kube-dns,pod-template-hash=64897985d
etcd-kind-control-plane                      1/1     Running   0          2d2h   component=etcd,tier=control-plane
kindnet-wx7kl                                1/1     Running   0          2d2h   app=kindnet,controller-revision-hash=9d779cb4d,k8s-app=kindnet,pod-template-generation=1,tier=node
kube-apiserver-kind-control-plane            1/1     Running   0          2d2h   component=kube-apiserver,tier=control-plane
kube-controller-manager-kind-control-plane   1/1     Running   0          2d2h   component=kube-controller-manager,tier=control-plane
kube-proxy-bgcrq                             1/1     Running   0          2d2h   controller-revision-hash=664d4bb79f,k8s-app=kube-proxy,pod-template-generation=1
kube-scheduler-kind-control-plane            1/1     Running   0          2d2h   component=kube-scheduler,tier=control-plane 
```

现在，如果你想过滤并仅列出 **kube-dns** Pod，可以输入以下命令：

```
$ k get po -n kube-system -l k8s-app=kube-dns
NAME                      READY   STATUS    RESTARTS   AGE
coredns-64897985d-gzrm4   1/1     Running   0          2d2h
coredns-64897985d-m8nm9   1/1     Running   0          2d2h 
```

### 使用部署部署长期运行的进程

在大规模系统中，Pod 不应只是被创建后任其自由。如果 Pod 因为某种原因意外死亡，你希望有另一个 Pod 来替代它，以维持整体容量。你可以自己创建副本控制器或副本集，但这样会留下错误的隐患，并且有部分失败的可能性。以声明的方式指定你希望启动多少个副本，这样显得更加合理。这正是 Kubernetes 部署的用途。

让我们通过 Kubernetes 部署资源来部署三个实例的 Hue 学习器微服务：

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hue-learn
  labels:
    app: hue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hue
  template:
    metadata:
      labels:
        app: hue
    spec:
      containers:
      - name: hue-learner
        image: g1g1/hue-learn:0.3
        resources:
          requests:
            cpu: 200m
            memory: 256Mi
        env:
        - name: DISCOVER_QUEUE
          value: dns
        - name: DISCOVER_STORE
          value: dns 
```

Pod 规格与之前的 pod 配置文件中的 `spec` 部分完全相同。

让我们创建这个部署并检查其状态：

```
$ k create -f hue-learn-deployment.yaml
deployment.apps/hue-learn created
$ k get deployment hue-learn
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
hue-learn   3/3     3            3           25s
$ k get pods -l app=hue
NAME                         READY   STATUS    RESTARTS   AGE
hue-learn-67d4649b58-qhc88   1/1     Running   0          45s
hue-learn-67d4649b58-qpm2q   1/1     Running   0          45s
hue-learn-67d4649b58-tzzq7   1/1     Running   0          45s 
```

你可以使用 `kubectl describe` 命令获取有关部署的更多信息：

```
$ k describe deployment hue-learn
Name:                   hue-learn
Namespace:              default
CreationTimestamp:      Tue, 21 Jun 2022 21:11:50 -0700
Labels:                 app=hue
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=hue
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=hue
  Containers:
   hue-learner:
    Image:      g1g1/hue-learn:0.3
    Port:       <none>
    Host Port:  <none>
    Requests:
      cpu:     200m
      memory:  256Mi
    Environment:
      DISCOVER_QUEUE:  dns
      DISCOVER_STORE:  dns
    Mounts:            <none>
  Volumes:             <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   hue-learn-67d4649b58 (3/3 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  106s  deployment-controller  Scaled up replica set hue-learn-67d4649b58 to 3 
```

### 更新一个部署

Hue 平台是一个庞大且不断发展的系统。你需要不断地进行升级。部署可以进行更新，以便以一种无痛的方式推出更新。你只需要更改 pod 模板，从而触发一个完全由 Kubernetes 管理的滚动更新。目前，所有 pods 都在运行 0.3 版本：

```
$ k get pods -o jsonpath='{.items[*].spec.containers[0].image}' -l app=hue | xargs printf "%s\n"
g1g1/hue-learn:0.3
g1g1/hue-learn:0.3
g1g1/hue-learn:0.3 
```

让我们更新部署，升级到 0.4 版本。在部署文件中修改镜像版本，不要修改标签，否则会导致错误。将其保存为 `hue-learn-deployment-0.4.yaml`。然后我们可以使用 `kubectl apply` 命令来升级版本，并验证 pods 是否现在运行 0.4 版本：

```
$ k apply -f hue-learn-deployment-0.4.yaml
Warning: resource deployments/hue-learn is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
deployment.apps/hue-learn configured
$ k get pods -o jsonpath='{.items[*].spec.containers[0].image}' -l app=hue | xargs printf "%s\n"
g1g1/hue-learn:0.4
g1g1/hue-learn:0.4
g1g1/hue-learn:0.4 
```

请注意，新的 pods 会被创建，而原来的 0.3 版本的 pods 会以滚动更新的方式终止。

```
$ kubectl get pods
NAME                         READY   STATUS        RESTARTS   AGE
hue-learn-67d4649b58-fgt7m   1/1     Terminating   0          99s
hue-learn-67d4649b58-klhz5   1/1     Terminating   0          100s
hue-learn-67d4649b58-lgpl9   1/1     Terminating   0          101s
hue-learn-68d74fd4b7-bxxnm   1/1     Running       0          4s
hue-learn-68d74fd4b7-fh55c   1/1     Running       0          3s
hue-learn-68d74fd4b7-rnsj4   1/1     Running       0          2s 
```

我们已经讲解了 `kubectl` 清单文件的结构，以及如何应用这些文件来部署和更新集群中的工作负载。接下来，我们将看看这些工作负载如何通过内部服务发现并相互调用，以及如何通过外部暴露的服务从集群外部进行访问。

# 分离内部服务和外部服务

内部服务是只允许集群中其他服务或作业（或登录并运行临时工具的管理员）直接访问的服务。也有一些工作负载完全不被访问。这些工作负载可能会监听某些事件并执行其功能，而不暴露任何 API。

但有些服务需要对用户或外部程序进行暴露。让我们看一个虚拟的 Hue 服务，它管理用户的提醒列表。它实际上做的事情并不多——只返回一个固定的提醒列表——但我们将用它来说明如何暴露服务。我已经将 `hue-reminders` 镜像推送到 Docker Hub：

```
docker push g1g1/hue-reminders:3.0 
```

## 部署一个内部服务

这是该部署文件，它与 `hue-learner` 部署非常相似，除了我去掉了注释、`env` 和 `resources` 部分，保留了一个或两个标签以节省空间，并且添加了一个 `ports` 部分到容器中。这是至关重要的，因为一个服务必须通过某个端口进行暴露，其他服务才能访问它：

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hue-reminders
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hue
      service: reminders
  template:
    metadata:
      name: hue-reminders
      labels:
        app: hue
        service: reminders
    spec:
      containers:
      - name: hue-reminders
        image: g1g1/hue-reminders:3.0
        ports:
        - containerPort: 8080 
```

当我们运行这个部署时，会有两个 `hue-reminders` pods 被添加到集群中：

```
$ k create -f hue-reminders-deployment.yaml
deployment.apps/hue-reminders created
$ k get pods
NAME                            READY   STATUS    RESTARTS   AGE
hue-learn-68d74fd4b7-bxxnm      1/1     Running   0          12h
hue-learn-68d74fd4b7-fh55c      1/1     Running   0          12h
hue-learn-68d74fd4b7-rnsj4      1/1     Running   0          12h
hue-reminders-9bdcd7489-4jqhc   1/1     Running   0          11s
hue-reminders-9bdcd7489-bxh59   1/1     Running   0          11s 
```

好的，Pods 已经在运行。从理论上讲，其他服务可以查找或通过它们的内部 IP 地址进行配置，并直接访问它们，因为它们都在同一个网络地址空间中。但这样并不具有可扩展性。每次提醒的 pod 终止并被新的 pod 替换，或者当我们只是增加 pod 的数量时，所有访问这些 pods 的服务都必须了解这些变更。Kubernetes 服务通过提供一个稳定的单一访问点来解决这个问题，所有共享一组选择器标签的 pods 都可以通过它来访问。这里是服务定义：

```
apiVersion: v1
kind: Service
metadata:
  name: hue-reminders
  labels:
    app: hue
    service: reminders
spec:
  ports:
  - port: 8080
    targetPort: 80
    protocol: TCP
  selector:
    app: hue
    service: reminders 
```

服务有一个 `selector`，通过匹配标签来确定支持的 Pods。它还暴露一个端口，其他服务将通过该端口访问它。这个端口不需要和容器的端口相同。你可以定义一个 `targetPort`。

`protocol` 字段可以是以下之一：TCP、UDP，或者（从 Kubernetes 1.12 开始）SCTP。

## 创建 hue-reminders 服务

让我们创建服务并进行探索：

```
$ k create -f hue-reminders-service.yaml
service/hue-reminders created
$ k describe svc hue-reminders
Name:              hue-reminders
Namespace:         default
Labels:            app=hue
                   service=reminders
Annotations:       <none>
Selector:          app=hue,service=reminders
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.96.152.254
IPs:               10.96.152.254
Port:              <unset>  8080/TCP
TargetPort:        8080/TCP
Endpoints:         10.244.0.32:8080,10.244.0.33:8080
Session Affinity:  None
Events:            <none> 
```

服务已经启动并运行。其他 Pods 可以通过环境变量或 DNS 找到它。所有服务的环境变量在 Pod 创建时设置。这意味着如果你在创建服务时一个 Pod 已经在运行，你必须杀掉它并让 Kubernetes 使用新的服务环境变量重新创建它。

例如，Pod `hue-learn-68d74fd4b7-bxxnm` 在创建 `hue-reminders` 服务之前就已经创建，因此它没有 `HUE_REMINDERS_SERVICE` 的环境变量。打印该 Pod 的环境变量显示该环境变量不存在：

```
$ k exec hue-learn-68d74fd4b7-bxxnm -- printenv | grep HUE_REMINDERS_SERVICE 
```

让我们杀掉该 Pod，当一个新的 Pod 替代它时，我们再试一次：

```
$ k delete po hue-learn-68d74fd4b7-bxxnm
pod "hue-learn-68d74fd4b7-bxxnm" deleted 
```

让我们再次检查 `hue-learn` Pods：

```
$ k get pods | grep hue-learn
hue-learn-68d74fd4b7-fh55c      1/1     Running   0          13h
hue-learn-68d74fd4b7-rnsj4      1/1     Running   0          13h
hue-learn-68d74fd4b7-rw4qr      1/1     Running   0          2m 
```

很好，我们有一个新的 Pod —— `hue-learn-68d74fd4b7-rw4qr`。让我们看看它是否有 `HUE_REMINDERS_SERVICE` 服务的环境变量：

```
$ k exec hue-learn-68d74fd4b7-rw4qr -- printenv | grep HUE_REMINDERS_SERVICE
HUE_REMINDERS_SERVICE_PORT=8080
HUE_REMINDERS_SERVICE_HOST=10.96.152.254 
```

是的，它有！不过使用 DNS 要简单得多。Kubernetes 会为每个服务分配一个内部 DNS 名称。服务的 DNS 名称是：

```
<service name>.<namespace>.svc.cluster.local
$ kubectl exec hue-learn-68d74fd4b7-rw4qr -- nslookup hue-reminders.default.svc.cluster.local
Server:     10.96.0.10
Address:    10.96.0.10:53
Name:   hue-reminders.default.svc.cluster.local
Address: 10.96.152.254 
```

现在，集群中的每个 Pod 都可以通过其服务端点和端口 `8080` 访问 `hue-reminders` 服务：

```
$ kubectl exec hue-learn-68d74fd4b7-fh55c -- wget -q -O - hue-reminders.default.svc.cluster.local:8080
Dentist appointment at 3pm
Dinner at 7pm 
```

是的，目前 `hue-reminders` 总是返回相同的两条提醒：

```
Dentist appointment at 3pm
Dinner at 7pm 
```

这里只是为了演示目的。如果 `hue-reminders` 是一个真实的系统，它会返回实时和动态的提醒。

现在我们已经了解了内部服务及其访问方式，接下来我们来看一下外部服务。

## 暴露服务到外部

服务可以在集群内部访问。如果你想将其暴露给外界，Kubernetes 提供了几种方法：

+   配置 `NodePort` 以便直接访问

+   如果你在云环境中运行，配置一个云负载均衡器

+   如果你在裸金属上运行，配置你自己的负载均衡器

在配置外部访问服务之前，你应该确保它是安全的。我们在*第四章*《Kubernetes 安全性》中已经讲解了这方面的原则。Kubernetes 文档中有一个很好的示例，涵盖了所有细节：[`github.com/kubernetes/examples/blob/master/staging/https-nginx/README.md`](https://github.com/kubernetes/examples/blob/master/staging/https-nginx/README.md)。

这是暴露到外部通过 `NodePort` 的 `hue-reminders` 服务的 `spec` 部分：

```
spec:
  type: NodePort
  ports:
  - port: 8080
    targetPort: 8080
    protocol: TCP
    name: http
  - port: 443
    protocol: TCP
    name: https
  selector:
    app: hue-reminders 
```

通过 `NodePort` 暴露服务的主要缺点是端口号在所有服务之间共享。你必须在整个集群中进行全局协调，以避免冲突。对于大规模集群和有大量开发者部署服务的情况，这并不简单。

但你可能有其他原因不希望直接暴露 Kubernetes 服务，例如安全性问题和缺乏抽象，可能更倾向于在服务前使用 Ingress 资源。

### Ingress

Ingress 是一个 Kubernetes 配置对象，允许你将服务暴露到外部世界，并处理许多细节。它可以做以下事情：

+   为你的服务提供一个对外可见的 URL

+   负载均衡流量

+   终止 SSL

+   提供基于名称的虚拟主机

要使用 Ingress，你必须在集群中运行一个 Ingress 控制器。Ingress 是在 Kubernetes 1.1 中引入的，并在 Kubernetes 1.19 中稳定。Ingress 控制器的一个当前限制是它并不适合大规模使用。因此，它目前不适合 Hue 平台。我们将在 *第十章*，*探索 Kubernetes 网络* 中更详细地讨论 Ingress 控制器。

下面是一个 Ingress 资源的样子：

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx-example
  rules:
  - http:
      paths:
      - path: /testpath
        pathType: Prefix
        backend:
          service:
            name: test
            port:
              number: 80 
```

注意这个注解，它提示这是一个与 Nginx Ingress 控制器配合使用的 Ingress 对象。还有许多其他 Ingress 控制器，它们通常使用注解来编码它们需要的信息，这些信息是 Ingress 对象及其规则本身无法捕获的。

其他 Ingress 控制器包括：

+   Traefik

+   Gloo

+   Contour

+   AWS ALB Ingress 控制器

+   HAProxy Ingress

+   Voyager

可以创建 `IngressClass` 资源并在 `Ingress` 资源中指定它。如果未指定，则使用默认的 `IngressClass`。

在本节中，我们查看了 Hue 平台的不同组件如何通过服务发现并相互通信，以及如何将面向公众的服务暴露给外部世界。在下一节中，我们将探讨如何在 Kubernetes 上高效且具成本效益地调度 Hue 工作负载。

# 高级调度

Kubernetes 最强大的特点之一就是其强大而灵活的调度器。简而言之，调度器的工作就是选择节点来运行新创建的 Pod。理论上，调度器甚至可以将现有的 Pod 在节点之间移动，但实际上它目前并不这样做，而是将这一功能交给其他组件。

默认情况下，调度器遵循几个指导原则，包括：

+   将来自同一副本集或有状态集的 Pod 分布到不同节点

+   将 Pod 调度到具有足够资源以满足 Pod 请求的节点上

+   平衡节点的整体资源利用率

这是一个相当不错的默认行为，但有时你可能希望对特定的 Pod 放置进行更好的控制。Kubernetes 1.6 引入了几个高级调度选项，使你可以精细地控制哪些 Pod 在哪些节点上调度，哪些 Pod 不调度，以及哪些 Pod 要一起调度或分开调度。

让我们在 Hue 的背景下回顾这些机制。

首先，让我们创建一个包含两个工作节点的 k3d 集群：

```
$ k3d cluster create --agents 2
...
INFO[0026] Cluster 'k3s-default' created successfully!
$ k get no
NAME                       STATUS   ROLES                  AGE   VERSION
k3d-k3s-default-agent-0    Ready    <none>                 22s   v1.23.6+k3s1
k3d-k3s-default-agent-1    Ready    <none>                 22s   v1.23.6+k3s1
k3d-k3s-default-server-0   Ready    control-plane,master   31s   v1.23.6+k3s1 
```

让我们看看 Pod 可以如何调度到节点的各种方式，以及每种方法适用的场景。

## 节点选择器

节点选择器非常简单。Pod 可以在其规格中指定它希望调度到的节点。例如，`trouble-shooter` Pod 有一个`nodeSelector`，它指定了`worker-2`节点的`kubernetes.io/hostname`标签：

```
apiVersion: v1
kind: Pod
metadata:
  name: trouble-shooter
  labels:
    role: trouble-shooter
spec:
  nodeSelector:
    kubernetes.io/hostname: k3d-k3s-default-agent-1
  containers:
  - name: trouble-shooter
    image: g1g1/py-kube:0.3
    command: ["bash"]
    args: ["-c", "echo started...; while true ; do sleep 1 ; done"] 
```

创建这个 Pod 时，它确实被调度到了`k3d-k3s-default-agent-1`节点：

```
$ k apply -f trouble-shooter.yaml
pod/trouble-shooter created
$ k get po trouble-shooter -o jsonpath='{.spec.nodeName}'
k3d-k3s-default-agent-1 
```

## 污点与容忍度

你可以对一个节点添加污点，防止 Pods 被调度到该节点。例如，如果你不希望 Pods 被调度到你的控制平面节点，可以使用这种方法。容忍度允许 Pods 声明它们可以“忍受”特定的节点污点，之后这些 Pods 可以调度到被污染的节点。一个节点可以有多个污点，一个 Pod 可以有多个容忍度。污点是一个三元组：键（key）、值（value）、效果（effect）。键和值用于识别污点，效果包括以下几种：

+   `NoSchedule`（除非 Pods 忍受污点，否则不会调度到该节点）

+   `PreferNoSchedule`（`NoSchedule`的软版本；调度器将尽量避免调度那些不忍受污点的 Pods）

+   `NoExecute`（不会调度新的 Pod，但也会驱逐那些不忍受污点的现有 Pod）

让我们在我们的 k3d 集群上部署`hue-learn`和`hue-reminders`：

```
$ k apply -f hue-learn-deployment.yaml
deployment.apps/hue-learn created
$ k apply -f hue-reminders-deployment.yaml
deployment.apps/hue-reminders created 
```

当前，在控制平面节点（`k3d-k3s-default-server-0`）上运行着一个`hue-learn` Pod：

```
$ k get po -o wide
NAME                            READY   STATUS    RESTARTS   AGE     IP          NODE                       NOMINATED NODE   READINESS GATES
trouble-shooter                 1/1     Running   0          2m20s   10.42.2.4   k3d-k3s-default-agent-1    <none>           <none>
hue-learn-67d4649b58-tklxf      1/1     Running   0          18s     10.42.1.8   k3d-k3s-default-server-0   <none>           <none>
hue-learn-67d4649b58-wk55w      1/1     Running   0          18s     10.42.0.3   k3d-k3s-default-agent-0    <none>           <none>
hue-learn-67d4649b58-jkwwg      1/1     Running   0          18s     10.42.2.5   k3d-k3s-default-agent-1    <none>           <none>
hue-reminders-9bdcd7489-2j65p   1/1     Running   0          6s      10.42.2.6   k3d-k3s-default-agent-1    <none>           <none>
hue-reminders-9bdcd7489-wntpx   1/1     Running   0          6s      10.42.0.4   k3d-k3s-default-agent-0    <none>           <none> 
```

让我们给控制平面节点加上污点：

```
$ k taint nodes k3d-k3s-default-server-0 control-plane=true:NoExecute
node/k3d-k3s-default-server-0 tainted 
```

现在我们可以查看污点（taint）：

```
$ k get nodes k3d-k3s-default-server-0 -o jsonpath='{.spec.taints[0]}'
map[effect:NoExecute key:control-plane value:true] 
```

是的，成功了！现在没有 Pods 被调度到主节点上。`k3d-k3s-default-server-0`上的`hue-learn` Pod 被驱逐，并且一个新的 Pod（`hue-learn-67d4649b58-bl8cn`）现在在`k3d-k3s-default-agent-0`上运行：

```
$ k get po -o wide
NAME                            READY   STATUS    RESTARTS   AGE     IP          NODE                      NOMINATED NODE   READINESS GATES
trouble-shooter                 1/1     Running   0          33m     10.42.2.4   k3d-k3s-default-agent-1   <none>           <none>
hue-learn-67d4649b58-wk55w      1/1     Running   0          31m     10.42.0.3   k3d-k3s-default-agent-0   <none>           <none>
hue-learn-67d4649b58-jkwwg      1/1     Running   0          31m     10.42.2.5   k3d-k3s-default-agent-1   <none>           <none>
hue-reminders-9bdcd7489-2j65p   1/1     Running   0          30m     10.42.2.6   k3d-k3s-default-agent-1   <none>           <none>
hue-reminders-9bdcd7489-wntpx   1/1     Running   0          30m     10.42.0.4   k3d-k3s-default-agent-0   <none>           <none>
hue-learn-67d4649b58-bl8cn      1/1     Running   0          2m53s   10.42.0.5   k3d-k3s-default-agent-0   <none>           <none> 
```

若要允许 Pod 忍受污点，可以在其规格中添加一个容忍度（toleration），例如：

```
tolerations:
- key: "control-plane"
  operator: "Equal"
  value: "true"
  effect: "NoSchedule" 
```

## 节点亲和性与反亲和性

节点亲和性（node affinity）是比`nodeSelector`更复杂的一种形式。它有三个主要优点：

+   丰富的选择标准（`nodeSelector`只是标签精确匹配的`AND`运算）

+   规则可以是软性（soft）

+   你可以使用`NotIn`和`DoesNotExist`等运算符实现反亲和性（anti-affinity）

请注意，如果你同时指定了`nodeSelector`和`nodeAffinity`，那么该 Pod 只会调度到满足两个要求的节点上。

例如，如果我们将以下部分添加到我们的`trouble-shooter` Pod，它将无法在任何节点上运行，因为它与`nodeSelector`发生冲突：

```
affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/hostname
            operator: NotIn
            values:
            - k3d-k3s-default-agent-1 
```

## Pod 亲和性与反亲和性

Pod 亲和性和反亲和性提供了另一种管理工作负载运行位置的方式。到目前为止，我们讨论的所有方法——节点选择器、污点/容忍、节点亲和性/反亲和性——都是关于将 pod 分配给节点的。但 pod 亲和性是关于不同 pod 之间的关系。pod 亲和性还涉及几个其他概念：命名空间（因为 pod 是有命名空间的）、拓扑区域（节点、机架、云提供商区域、云提供商区域）、权重（用于首选调度）。一个简单的例子是，如果你希望`hue-reminders`总是与`trouble-shooter` pod 一起调度。让我们看看如何在`hue-reminders`部署的 pod 模板规格中定义它：

```
 affinity:
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: role
                operator: In
                values:
                - trouble-shooter
            topologyKey: topology.kubernetes.io/zone # for clusters on cloud providers 
```

拓扑键是一个节点标签，Kubernetes 会将其视为在调度时相同。在云提供商中，当工作负载应该彼此靠近时，建议使用`topology.kubernetes.io/zone`。在云环境中，区域相当于数据中心。

然后，在重新部署`hue-reminders`之后，所有的`hue-reminders` pod 都被调度到`k3d-k3s-default-agent-1`上，紧邻`trouble-shooter` pod：

```
$ k apply -f hue-reminders-deployment-with-pod-affinity.yaml
deployment.apps/hue-reminders configured
$ k get po -o wide
NAME                             READY   STATUS    RESTARTS   AGE    IP          NODE                      NOMINATED NODE   READINESS GATES
trouble-shooter                  1/1     Running   0          117m   10.42.2.4   k3d-k3s-default-agent-1   <none>           <none>
hue-learn-67d4649b58-wk55w       1/1     Running   0          115m   10.42.0.3   k3d-k3s-default-agent-0   <none>           <none>
hue-learn-67d4649b58-jkwwg       1/1     Running   0          115m   10.42.2.5   k3d-k3s-default-agent-1   <none>           <none>
hue-learn-67d4649b58-bl8cn       1/1     Running   0          87m    10.42.0.5   k3d-k3s-default-agent-0   <none>           <none>
hue-reminders-544d96785b-pd62t   0/1     Pending   0          50s    10.42.2.4   k3d-k3s-default-agent-1   <none>           <none>
hue-reminders-544d96785b-wpmjj   0/1     Pending   0          50s    10.42.2.4   k3d-k3s-default-agent-1   <none>           <none> 
```

## Pod 拓扑分布约束

节点亲和性/反亲和性和 pod 亲和性/反亲和性有时过于严格。你可能希望分散你的 pod——如果同一部署的某些 pod 最终部署在同一节点上也是可以的。pod 拓扑分布约束为你提供了这种灵活性。你可以指定最大偏差，即你能接受的最优分布偏差，同时也可以指定当约束无法满足时的行为（`DoNotSchedule`或`ScheduleAnyway`）。

这是我们的`hue-reminders`部署，带有 pod 拓扑分布约束：

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hue-reminders
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hue
      service: reminders
  template:
    metadata:
      name: hue-reminders
      labels:
        app: hue
        service: reminders
    spec:
      topologySpreadConstraints:
        - maxSkew: 1
          topologyKey: node.kubernetes.io/instance-type
          whenUnsatisfiable: DoNotSchedule
          labelSelector:
            matchLabels:
              app: hue
              service: hue-reminders
      containers:
      - name: hue-reminders
        image: g1g1/hue-reminders:3.0
        ports:
        - containerPort: 80 
```

我们可以看到，在应用清单之后，三个 pod 被分布在两个代理节点上（服务器节点如你所记得，有一个污点）：

```
$  k apply -f hue-reminders-deployment-with-spread-contraitns.yaml
deployment.apps/hue-reminders created
$ k get po -o wide -l app=hue,service=reminders
NAME                             READY   STATUS    RESTARTS   AGE     IP           NODE                      NOMINATED NODE   READINESS GATES
hue-reminders-6664fccb8f-8bvf6   1/1     Running   0          4m40s   10.42.0.11   k3d-k3s-default-agent-0   <none>           <none>
hue-reminders-6664fccb8f-8qrbl   1/1     Running   0          3m59s   10.42.0.12   k3d-k3s-default-agent-0   <none>           <none>
hue-reminders-6664fccb8f-b5pbp   1/1     Running   0          56s     10.42.2.14   k3d-k3s-default-agent-1   <none>           <none> 
```

## 反调度器

Kubernetes 在根据复杂的放置规则将 pod 调度到节点方面表现出色。但是，一旦 pod 被调度，Kubernetes 就不会在原始条件发生变化时将其移动到另一个节点。以下是一些可以从工作负载迁移中受益的用例：

+   某些节点出现了资源利用不足或过度利用的情况。

+   当节点上的污点或标签被修改时，初始调度决策不再有效，导致 pod/node 亲和性要求不再满足。

+   某些节点发生了故障，导致它们的 pod 迁移到其他节点。

+   向集群中引入了额外的节点。

这时，反调度器就发挥作用了。反调度器并不是 Kubernetes 的原生功能。你需要安装它并定义策略，决定哪些正在运行的 pod 可能被驱逐。它可以作为`Job`、`CronJob`或`Deployment`运行。反调度器会定期检查当前 pod 的分布情况，并会驱逐违反某些策略的 pod。这些 pod 会被重新调度，标准的 Kubernetes 调度器会根据当前的条件重新安排它们。

在这里查看：[`github.com/kubernetes-sigs/descheduler`](https://github.com/kubernetes-sigs/descheduler)。

在这一部分中，我们看到了 Kubernetes 提供的高级调度机制，以及像 descheduler 这样的项目，如何帮助 Hue 在可用的基础设施上以最优方式调度其工作负载。在下一部分中，我们将研究如何将 Hue 的工作负载划分到命名空间中，以便更好地管理对不同资源的访问。

# 使用命名空间来限制访问

Hue 项目进展顺利，我们有几百个微服务和大约 100 名开发人员以及 DevOps 工程师在参与其中。相关微服务的组逐渐形成，你会发现这些组相当自治，它们完全忽略了其他组。此外，像健康和财务等敏感区域，你希望更有效地控制访问。于是，命名空间应运而生。

让我们创建一个新的服务`hue-finance`，并将其放入一个名为`restricted`的新命名空间中。

这是新`restricted`命名空间的 YAML 文件：

```
kind: Namespace
apiVersion: v1
metadata:
  name: restricted
  labels:
    name: restricted 
```

我们可以像往常一样创建它：

```
$ kubectl create -f restricted-namespace.yaml
namespace "restricted" created 
```

一旦命名空间创建完成，我们可以为该命名空间配置一个上下文：

```
$ k config set-context k3d-k3s-restricted --cluster k3d-k3s-default --namespace=restricted --user restricted@k3d-k3s-default
Context "restricted" created.
$ k config use-context restricted
Switched to context "restricted". 
```

让我们检查一下我们的集群配置：

```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://0.0.0.0:53829
  name: k3d-k3s-default
contexts:
- context:
    cluster: k3d-k3s-default
    user: admin@k3d-k3s-default
  name: k3d-k3s-default
- context:
    cluster: ""
    namespace: restricted
    user: restricted@k3d-k3s-default
  name: restricted
current-context: restricted
kind: Config
preferences: {}
users:
- name: admin@k3d-k3s-default
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED 
```

正如你所看到的，现在有两个上下文，当前的上下文是`restricted`。如果需要，我们甚至可以创建专用用户，并为他们分配允许在受限命名空间中操作的凭证。根据环境的不同，这可能很简单也可能很复杂，并且可能涉及通过 Kubernetes 证书颁发机构创建证书。云服务提供商提供与其身份和访问管理（IAM）系统的集成。

为了继续进行，我将使用`admin@k3d-k3s-default`用户的凭证，并直接在集群的`kubeconfig`文件中创建一个名为`restricted@k3d-k3s-default`的用户：

```
users:
- name: restricted@k3d-k3s-default
  user:
    client-certificate-data: REDACTED
    client-key-data: REDACTED 
```

现在，在这个空的命名空间中，我们可以创建我们的 `hue-finance` 服务，它将与默认命名空间中的其他服务分开：

```
$ k create -f hue-finance-deployment.yaml
deployment.apps/hue-finance created
$ k get pods
NAME                           READY   STATUS    RESTARTS   AGE
hue-finance-84c445f684-vh8qv   1/1     Running   0          7s
hue-finance-84c445f684-fjkxs   1/1     Running   0          7s
hue-finance-84c445f684-sppkq   1/1     Running   0          7s 
```

你不需要切换上下文。你还可以使用`--namespace=<namespace>`和`--all-namespaces`命令行选项，但当长时间在同一个非默认命名空间中操作时，将上下文设置为该命名空间会更加方便。

# 使用 Kustomization 进行分层集群结构

这不是打字错误。Kubectl 最近集成了 Kustomize 的功能（[`kustomize.io/`](https://kustomize.io/)）。它是一种无需模板即可配置 Kubernetes 的方式。关于 Kustomize 功能如何集成到 kubectl 本身中曾有过很多争议，因为还有其他选择，是否应该让 kubectl 具有这么强的偏好也曾是一个开放的问题。不过，这些都已经过去了。最终结论是，`kubectl apply -k` 解锁了大量的配置选项。让我们理解它帮助我们解决了什么问题，并利用它来帮助我们管理 Hue。

## 理解 Kustomize 的基础

Kustomize 是响应像 Helm 这样的模板驱动方法而创建的，用于配置和定制 Kubernetes 集群。它围绕声明式应用管理的原则设计。它接受一个有效的 Kubernetes YAML 清单（基础配置），并通过覆盖额外的 YAML 补丁（覆盖层）来专门化或扩展它。覆盖层依赖于它们的基础配置。所有文件都是有效的 YAML 文件。没有占位符。

`kustomization.yaml` 文件控制该过程。任何包含 `kustomization.yaml` 文件的目录都称为根目录。例如：

```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: staging
commonLabels:
    environment: staging
bases:
  - ../base
patchesStrategicMerge:
  - hue-learn-patch.yaml
resources:
  - namespace.yaml 
```

Kustomize 在 GitOps 环境中表现良好，其中不同的 Kustomizations 存储在 Git 仓库中，并且对基础配置、覆盖层或 `kustomization.yaml` 文件的更改会触发部署。

Kustomize 的最佳使用场景之一是将你的系统组织成多个命名空间，如 staging 和 production。让我们重构 Hue 平台部署清单。

## 配置目录结构

首先，我们需要一个基础目录，其中包含所有清单的共性。然后我们将拥有一个 `overlays` 目录，该目录包含 `staging` 和 `production` 子目录：

```
$ tree
.
├── base
│   ├── hue-learn.yaml
│   └── kustomization.yaml
├── overlays
│   ├── production
│   │   ├── kustomization.yaml
│   │   └── namespace.yaml
│   └── staging
│       ├── hue-learn-patch.yaml
│       ├── kustomization.yaml
│       └── namespace.yaml 
```

基础目录中的 `hue-learn.yaml` 文件只是一个示例。那里可能有许多文件。让我们快速查看它：

```
apiVersion: v1
kind: Pod
metadata:
  name: hue-learner
  labels:
    tier: internal-service
spec:
  containers:
  - name: hue-learner
    image: g1g1/hue-learn:0.3
    resources:
      requests:
        cpu: 200m
        memory: 256Mi
    env:
    - name: DISCOVER_QUEUE
      value: dns
    - name: DISCOVER_STORE
      value: dns 
```

它与我们之前创建的清单非常相似，但没有 `app: hue` 标签。这个标签并不是必需的，因为标签是由 `kustomization.yaml` 文件提供的，作为所有列出资源的通用标签：

```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
commonLabels:
  app: hue
resources:
  - hue-learn.yaml 
```

## 应用 Kustomizations

我们可以通过在基础目录上运行 `kubectl kustomize` 命令来查看结果。你可以看到，通用标签 `app: hue` 被添加了：

```
$ k kustomize base
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: hue
    tier: internal-service
  name: hue-learner
spec:
  containers:
  - env:
    - name: DISCOVER_QUEUE
      value: dns
    - name: DISCOVER_STORE
      value: dns
    image: g1g1/hue-learn:0.3
    name: hue-learner
    resources:
      requests:
        cpu: 200m
        memory: 256Mi 
```

为了实际部署 Kustomization，我们可以运行 `kubectl -k apply`。但是，基础配置不应该单独部署。让我们深入到 `overlays/staging` 目录并查看它。

`namespace.yaml` 文件仅创建 `staging` 命名空间。正如我们即将看到的，它将受益于所有的 Kustomizations：

```
apiVersion: v1
kind: Namespace
metadata:
  name: staging 
```

`kustomization.yaml` 文件添加了通用标签 `environment: staging`。它依赖于基础目录，并将 `namespace.yaml` 文件添加到资源列表中（该列表已包括基础目录中的 `hue-learn.yaml`）：

```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: staging
commonLabels:
    environment: staging
bases:
  - ../../base
patchesStrategicMerge:
  - hue-learn-patch.yaml
resources:
  - namespace.yaml 
```

但这还不是全部。Kustomizations 最有趣的部分是补丁。

### 补丁

补丁添加或替换清单的部分内容。它们从不删除现有的资源或资源的部分内容。`hue-learn-patch.yaml` 将镜像从 `g1g1/hue-learn:0.3` 更新为 `g1g1/hue-learn:0.4`：

```
apiVersion: v1
kind: Pod
metadata:
  name: hue-learner
spec:
  containers:
  - name: hue-learner
    image: g1g1/hue-learn:0.4 
```

这是一个战略合并。Kustomize 支持另一种补丁类型，称为 `JsonPatches6902`。它基于 RFC 6902 ([`tools.ietf.org/html/rfc6902`](https://tools.ietf.org/html/rfc6902))。通常，它比战略合并更简洁。我们可以使用 YAML 语法为 JSON 6902 补丁编写。这里是使用 `JsonPatches6902` 语法将镜像版本更改为 0.4 的相同补丁：

```
- op: replace
  path: /spec/containers/0/image
  value: g1g1/hue-learn:0.4 
```

### 对整个 staging 命名空间进行 Kustomize 操作

这是在`overlays/staging`目录上运行 Kustomize 时生成的内容：

```
$ k kustomize overlays/staging
apiVersion: v1
kind: Namespace
metadata:
  labels:
    environment: staging
  name: staging
---
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: hue
    environment: staging
    tier: internal-service
  name: hue-learner
  namespace: staging
spec:
  containers:
  - env:
    - name: DISCOVER_QUEUE
      value: dns
    - name: DISCOVER_STORE
      value: dns
    image: g1g1/hue-learn:0.4
    name: hue-learner
    resources:
      requests:
        cpu: 200m
        memory: 256Mi 
```

请注意，命名空间没有继承基础中的`app: hue`标签，而只继承了其本地 Kustomization 文件中的`environment: staging`标签。另一方面，`hue-learn` pod 则得到了所有标签以及命名空间指定。

现在是时候将其部署到集群中了：

```
$ k apply -k overlays/staging
namespace/staging created
pod/hue-learner created 
```

现在，我们可以查看在新创建的`staging`命名空间中的 pod：

```
$ k get po -n staging
NAME          READY   STATUS    RESTARTS   AGE
hue-learner   1/1     Running   0          21s 
```

让我们检查 overlay 是否生效，并且镜像版本确实是 0.4：

```
$ k get po hue-learner -n staging -o jsonpath='{.spec.containers[0].image}'
g1g1/hue-learn:0.4 
```

在本节中，我们介绍了 Kustomize 选项所提供的强大结构化和可重用性。这对像 Hue 平台这样的大规模系统非常重要，因为许多工作负载可以从统一的结构和一致的基础中受益。在下一节中，我们将介绍如何启动短期作业。

# 启动作业

Hue 已经发展起来，并且部署了许多长时间运行的微服务进程，但它也有许多任务会运行、完成某些目标然后退出。Kubernetes 通过`Job`资源支持这种功能。Kubernetes 作业管理一个或多个 pod，并确保它们在成功或失败之前一直运行。如果作业管理的 pod 之一失败或被删除，那么作业会启动一个新的 pod，直到成功为止。

也有许多适用于 Kubernetes 的无服务器或函数即服务解决方案，但它们是建立在原生 Kubernetes 之上的。我们将在*第十二章*中深入讨论无服务器计算，*Kubernetes 上的无服务器计算*。

这是一个运行 Python 进程计算 5 的阶乘（提示：结果是 120）的作业：

```
apiVersion: batch/v1
kind: Job
metadata:
  name: factorial5
spec:
  template:
    metadata:
      name: factorial5
    spec:
      containers:
      - name: factorial5
        image: g1g1/py-kube:0.3
        command: ["python",
                  "-c",
                  "import math; print(math.factorial(5))"]
      restartPolicy: Never 
```

请注意，`restartPolicy`必须是`Never`或`OnFailure`。默认值`Always`无效，因为作业在成功完成后不会重启。

让我们启动作业并检查其状态：

```
$ k create -f factorial-job.yaml
job.batch/factorial5 created
$ k get jobs
NAME         COMPLETIONS   DURATION   AGE
factorial5   1/1           4s         27s 
```

完成任务的 pod 显示为`Completed`状态。请注意，作业 pod 有一个名为`job-name`的标签，标签值为作业的名称，因此很容易过滤出作业 pod：

```
$ k get po -l job-name=factorial5
NAME               READY   STATUS      RESTARTS   AGE
factorial5-dddzz   0/1     Completed   0          114s 
```

让我们查看日志中的输出：

```
$ k logs factorial5-dddzz
120 
```

按顺序启动作业对于某些用例是可以的，但通常运行并行作业会更有用。此外，完成后清理作业并定期运行作业也很重要。让我们看看这是怎么做到的。

## 并行运行作业

你也可以运行带有并行度的作业。在 spec 中有两个字段，分别是`completions`和`parallelism`。默认情况下，completions 设置为 1。如果你想要多个成功的完成次数，可以增加这个值。并行度决定了要启动多少个 pod。作业不会启动超过成功完成所需的 pod，即使并行度数大于该值。

让我们运行另一个作业，它只会休眠 20 秒，直到完成三次成功。我们将使用并行因子为六，但只会启动三个 pod：

```
apiVersion: batch/v1
kind: Job
metadata:
  name: sleep20
spec:
  completions: 3
  parallelism: 6
  template:
    metadata:
      name: sleep20
    spec:
      containers:
      - name: sleep20
        image: g1g1/py-kube:0.3
        command: ["python",
                  "-c",
                  "import time; print('started...');
                   time.sleep(20); print('done.')"]
      restartPolicy: Never 
```

让我们运行作业并等待所有 pod 完成：

```
$ k create -f parallel-job.yaml
job.batch/sleep20 created 
```

现在我们可以看到，所有三个 pods 都已完成，并且这些 pods 现在没有准备好，因为它们已经完成了工作：

```
$ k get pods -l job-name=sleep20
NAME            READY   STATUS      RESTARTS   AGE
sleep20-fqgst   0/1     Completed   0          4m5s
sleep20-2dv8h   0/1     Completed   0          4m5s
sleep20-kvn28   0/1     Completed   0          4m5s 
```

已完成的 pods 不会占用节点上的资源，因此其他 pods 可以在该节点上被调度。

## 清理已完成的作业

当一个作业完成时，它会保留 – 其 pods 也会保留。这是设计使然，目的是让你查看日志或连接到 pods 进行探查。但通常情况下，当一个作业成功完成后，它就不再需要了。清理已完成的作业及其 pods 是你的责任。

最简单的方法是直接删除 `job` 对象，这样也会删除所有的 pods：

```
$ kubectl get jobs
NAME         COMPLETIONS   DURATION   AGE
factorial5   1/1           2s         6h59m
sleep20      3/3           3m7s       5h54m
$ kubectl delete job factorial5
job.batch "factorial5" deleted
$ kubectl delete job sleep20
job.batch "sleep20" deleted 
```

## 调度 cron 作业

Kubernetes cron 作业是按指定时间运行的作业，可以是一次性运行或重复运行。它们的行为与在 `/etc/crontab` 文件中指定的普通 Unix cron 作业一样。

`CronJob` 资源在 Kubernetes 1.21 版本中变得稳定。这里是每分钟启动一个 cron 作业的配置，用来提醒你做伸展运动。在调度中，你可以将 ‘`*`' 替换为 ‘`?`'：

```
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cron-demo
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        metadata:
          labels:
            cronjob-name: cron-demo
        spec:
          containers:
            - name: cron-demo
              image: g1g1/py-kube:0.3
              args:
                - python
                - -c
                - from datetime import datetime; print(f'[{datetime.now()}] CronJob demo here...remember to stretch')
          restartPolicy: OnFailure 
```

在 pod 的 `spec` 中，在作业模板下，我添加了标签 `cronjob-name: cron-demo`。原因是，cron 作业及其 pods 会由 Kubernetes 分配一个随机前缀的名称。这个标签允许你轻松发现某个特定 cron 作业的所有 pods。Pods 还会有 `job-name` 标签，因为一个 cron 作业会为每次执行创建一个 `job` 对象。然而，作业名称本身带有随机前缀，因此它不能帮助我们发现 pods。

让我们运行 cron 作业并在一分钟后观察结果：

```
$ k get cj
NAME        SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
cron-demo   */1 * * * *   False     0        <none>          16s
$ k get job
NAME                 COMPLETIONS   DURATION   AGE
cron-demo-27600079   1/1           3s         2m45s
cron-demo-27600080   1/1           3s         105s
cron-demo-27600081   1/1           3s         45s
$ k get pods
NAME                       READY   STATUS      RESTARTS   AGE
cron-demo-27600080-dmcmq   0/1     Completed   0          2m6s
cron-demo-27600081-gjsvd   0/1     Completed   0          66s
cron-demo-27600082-sgjlh   0/1     Completed   0          6s 
```

如你所见，每分钟 cron 作业都会创建一个新的作业，并且每个作业的 pod 都会被标记上不同的名称。每个作业的 pod 都被标记上其作业名称，并且还会标记上 cron 作业名称 —— `cronjob-demo` —— 以便于汇总所有来自该 cron 作业的 pods。

像往常一样，你可以使用 `logs` 命令查看已完成作业的 pod 输出：

```
$ k logs cron-demo-27600082-sgjlh
[2022-06-23 17:22:00.971343] CronJob demo here...remember to stretch 
```

当你删除一个 cron 作业时，它会停止调度新的作业，并删除所有现有的 `job` 对象以及它创建的所有 pods。

你可以使用指定的标签（在这个例子中是 `name=cron-demo`）来定位 cron 作业启动的所有 `job` 对象：

```
$ k delete job -l name=cron-demo
job.batch "cron-demo-27600083" deleted
job.batch "cron-demo-27600084" deleted
job.batch "cron-demo-27600085" deleted 
```

你也可以暂停一个 cron 作业，这样它就不会创建更多作业，但不会删除已完成的作业和 pods。你还可以通过在 spec 中设置历史限制来管理作业的保留数量：`.spec.successfulJobsHistoryLimit` 和 `.spec.failedJobsHistoryLimit`。

在本节中，我们介绍了启动作业和控制作业的关键主题。这是 Hue 平台的一个关键方面，它需要响应实时事件并通过启动作业以及定期执行短期任务来处理它们。

# 混合非集群组件

Kubernetes 集群中的大多数实时系统组件将与集群外的组件通信。那些组件可以是通过某些 API 访问的完全外部的第三方服务，也可以是运行在同一局域网中的内部服务，由于各种原因，它们不属于 Kubernetes 集群的一部分。

这里有两类：集群内网络和集群外网络。

## 集群外网络组件

这些组件无法直接访问集群。它们只能通过 API、外部可见的 URL 和暴露的服务访问集群。这些组件被视为任何外部用户。通常，集群组件会使用外部服务，这不会造成安全问题。例如，在以前的公司里，我们有一个 Kubernetes 集群，它将异常报告给名为 Sentry 的第三方服务（[`sentry.io/welcome/`](https://sentry.io/welcome/)）。这是 Kubernetes 集群到第三方服务的单向通信。Kubernetes 集群拥有访问 Sentry 的凭据，这就是这单向通信的全部。

## 集群内网络组件

这些是运行在网络内部但不由 Kubernetes 管理的组件。运行这些组件有多种原因。它们可能是尚未“容器化”的遗留应用程序，或者是一些不易在 Kubernetes 内部运行的分布式数据存储。将这些组件运行在网络内部的原因是为了性能，并且与外界隔离，以便这些组件和 Pod 之间的流量更加安全。作为同一网络的一部分可以确保低延迟，并且减少需要为通信打开网络的需求，这不仅方便，而且可以提高安全性。

## 使用 Kubernetes 管理 Hue 平台

在本节中，我们将探讨 Kubernetes 如何帮助运营像 Hue 这样的巨大平台。Kubernetes 本身提供了很多能力来编排 Pod 并管理配额和限制，检测和恢复某些类型的通用故障（硬件故障、进程崩溃和无法访问的服务）。但是，在像 Hue 这样的复杂系统中，Pod 和服务可能已经启动并运行，但处于无效状态，或者正在等待其他依赖项才能执行其职能。这很棘手，因为如果一个服务或 Pod 尚未准备好，但已经在接收请求，那么你需要以某种方式进行管理：失败（责任转移给调用者）、重试（重试多少次？多长时间？多频繁？），以及稍后排队（谁来管理这个队列？）。

如果整个系统能够了解不同组件的就绪状态，或者仅当组件真正就绪时才对外可见，这通常会更好。Kubernetes 不了解 Hue，但它提供了几种机制，如活性探针、就绪探针、启动探针和初始化容器，以支持针对应用程序的集群管理。

## 使用活性探针确保容器存活

kubelet 负责监控你的容器。如果容器进程崩溃，kubelet 会根据重启策略处理它。但在许多情况下，这还不够。你的进程可能不会崩溃，而是陷入死循环或死锁中。重启策略可能不够精细。通过存活探针，你可以决定容器何时被视为存活的。如果存活探针失败，Kubernetes 会重启你的容器。这里是一个 Hue 音乐服务的 pod 模板。它有一个 `livenessProbe` 部分，使用 `httpGet` 探针。HTTP 探针需要一个方案（`http` 或 `https`，默认是 `http`）、一个主机（默认是 `PodIp`）、一个路径和一个端口。如果 HTTP 状态码在 `200` 到 `399` 之间，则认为探针成功。你的容器可能需要一些时间来初始化，因此你可以指定 `initialDelayInSeconds`。在此期间，kubelet 不会进行存活检查：

```
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: music
    service: music
  name: hue-music
spec:
  containers:
    - image: g1g1/hue-music
      livenessProbe:
        httpGet:
          path: /pulse
          port: 8888
          httpHeaders:
            - name: X-Custom-Header
              value: ItsAlive
        initialDelaySeconds: 30
        timeoutSeconds: 1
      name: hue-music 
```

如果任何容器的存活探针失败，那么 pod 的重启策略将生效。确保你的重启策略不是 `Never`，因为那样会使探针失效。

还有三种其他类型的存活探针：

+   `TcpSocket` – 只是检查端口是否打开

+   `Exec` – 运行一个返回 0 表示成功的命令

+   `gRPC` – 遵循 gRPC 健康检查协议 ([`github.com/grpc/grpc/blob/master/doc/health-checking.md`](https://github.com/grpc/grpc/blob/master/doc/health-checking.md))

## 使用就绪探针来管理依赖关系

就绪探针用于不同的目的。你的容器可能已经启动并运行，并通过了存活探针，但它可能依赖于当前不可用的其他服务。例如，`hue-music` 可能依赖于访问包含你听歌历史的数据服务。如果没有访问权限，它将无法履行职责。在这种情况下，其他服务或外部客户端不应向 `hue-music` 服务发送请求，但无需重启它。就绪探针解决了这一用例。当容器的就绪探针失败时，该容器的 pod 会从它注册的任何服务端点中移除。这确保了不会向无法处理请求的服务发送请求。请注意，你还可以使用就绪探针暂时移除过度预订的 pods，直到它们清空一些内部队列。

这里是一个示例就绪探针。我在这里使用 `exec` 探针来执行一个自定义命令。如果命令退出时返回非零的退出码，容器将被销毁：

```
readinessProbe:
  exec:
    command:
        - /usr/local/bin/checker
        - --full-check
        - --data-service=hue-multimedia-service
  initialDelaySeconds: 60
  timeoutSeconds: 5 
```

在同一个容器上同时使用就绪探针和存活探针是可以的，因为它们有不同的目的。

## 使用启动探针

一些应用程序（主要是遗留应用程序）可能具有较长的初始化周期。在这种情况下，活跃性探针可能会失败，并导致容器在完成初始化之前重新启动。这就是启动探针发挥作用的地方。如果配置了启动探针，则会跳过活跃性和就绪性检查，直到启动完成。此时，启动探针不再被调用，正常的活跃性和就绪性探针接管。

例如，在以下配置片段中，启动探针每 10 秒检查 5 分钟，以确定容器是否已启动（使用与活跃探针相同的检查）。如果启动探针失败 30 次（300 秒 = 5 分钟），则容器将被重新启动，并获得额外的 5 分钟来尝试初始化自身。但是，如果在 5 分钟内通过了启动探针的检查，则活跃探针生效，任何活跃检查失败将导致重新启动：

```
ports:
- name: liveness-port
  containerPort: 8080
  hostPort: 8080
livenessProbe:
  httpGet:
    path: /healthz
    port: liveness-port
  failureThreshold: 1
  periodSeconds: 10
startupProbe:
  httpGet:
    path: /healthz
    port: liveness-port
  failureThreshold: 30
  periodSeconds: 10 
```

## 使用初始化容器有序地启动 Pod

活跃性、就绪性和启动探针非常重要。它们认识到，在启动过程中，容器可能尚未就绪，但不应视为失败。为了适应这一点，有一个`initialDelayInSeconds`设置，容器在此期间不会被视为失败。但是，如果这个初始延迟可能非常长怎么办？也许在大多数情况下，容器几秒钟后即可准备好并且可以处理请求，但由于初始延迟设置为 5 分钟以防万一，我们在容器空闲时浪费了大量时间。如果容器是高流量服务的一部分，那么每次升级后许多实例都可能在空闲 5 分钟后坐等，几乎使服务不可用。

初始化容器解决了这个问题。一个 Pod 可以有一组初始化容器在其他容器启动之前运行完成。初始化容器可以处理所有非确定性初始化，并让带有其就绪探针的应用容器具有最小的延迟。

初始化容器特别适用于 Pod 级别的初始化任务，例如等待卷准备就绪。初始化容器和启动探针之间有些重叠，选择取决于具体的用例。

初始化容器在 Kubernetes 1.6 中退出了 beta 版。您可以在 Pod 规范中指定它们作为`initContainers`字段，这与`containers`字段非常相似。以下是一个示例：

```
kind: Pod
metadata:
  name: hue-fitness
spec:
  containers:
  - name: hue-fitness
    image: busybox
  initContainers:
  - name: install
    image: busybox 
```

## Pod 的就绪性和就绪性门

Pod 就绪性是在 Kubernetes 1.11 中引入的，并在 Kubernetes 1.14 中变得稳定。尽管就绪探针允许你在容器级别判断其是否准备好处理请求，但支持将流量传递到 pod 的整体基础设施可能尚未准备好。例如，服务、网络策略和负载均衡器可能需要一些额外的时间。这可能会成为一个问题，尤其是在滚动部署期间，Kubernetes 可能在新 pod 真的准备好之前就终止旧 pod，这将导致服务容量下降，甚至在极端情况下引发服务中断（所有旧 pod 被终止且没有新的 pod 完全准备好）。

这就是 pod 就绪性门控所解决的问题。其思路是将 pod 就绪性的概念扩展到检查额外的条件，而不仅仅是确保所有容器都已就绪。这是通过在 `PodSpec` 中添加一个新的字段 `readinessGates` 来实现的。你可以指定一组必须满足的条件，才能将 pod 视为已准备好。在以下示例中，由于 `www.example.com/feature-1` 条件的 `status: False`，该 pod 还未就绪：

```
Kind: Pod
...
spec:
  readinessGates:
    - conditionType: "www.example.com/feature-1"
status:
  conditions:
    - type: Ready  # this is a builtin PodCondition
      status: "False"
      lastProbeTime: null
      lastTransitionTime: 2023-01-01T00:00:00Z
    - type: "www.example.com/feature-1"   # an extra PodCondition
      status: "False"
      lastProbeTime: null
      lastTransitionTime: 2023-01-01T00:00:00Z
  containerStatuses:
    - containerID: docker://abcd...
      ready: true
... 
```

## 与 DaemonSet pod 共享

DaemonSet pod 是自动部署的 pod，每个节点（或指定的节点子集）上部署一个。它们通常用于监视节点并确保它们处于正常工作状态。这是一个非常重要的功能，我们将在*第十三章*，*Kubernetes 集群监控*中详细讨论。但它们也可以用于更多其他功能。默认的 Kubernetes 调度器的特点是，它根据资源可用性和请求来调度 pod。如果你有很多不需要大量资源的 pod，那么这些 pod 很可能会被调度到同一个节点上。

假设有一个 pod 执行一个小任务，然后每秒将所有活动的总结发送到一个远程服务。现在，假设平均来说，50 个这样的 pod 会被调度到同一个节点上。这意味着每秒钟，50 个 pod 会发起 50 次网络请求，每次请求的数据量非常小。如何将其减少 50 倍，仅发送一个网络请求呢？通过使用 `DaemonSet` pod，其他 50 个 pod 可以与它通信，而不是直接与远程服务进行通信。`DaemonSet` pod 将从这 50 个 pod 收集所有数据，并每秒将其汇总报告到远程服务。当然，这要求远程服务的 API 支持汇总报告。好的一点是，这些 pod 本身无需修改；它们只需配置为与 `DaemonSet` pod 在 `localhost` 上通信，而不是直接与远程服务通信。`DaemonSet` pod 充当了一个聚合代理。它还可以实现重试和其他类似功能。

这个配置文件的有趣之处在于，`hostNetwork`、`hostPID` 和 `hostIPC` 选项被设置为 `true`。这使得 pods 可以高效地与代理进行通信，利用它们运行在同一个物理主机上的事实：

```
apiVersion: apps/v1
kind-fission | fission
kind: DaemonSet
metadata:
  name: hue-collect-proxy kind-fission | fission
  labels:
    tier: stats
    app: hue-collect-proxy
spec:
  selector:
    matchLabels:
      tier: stats
      app: hue-collect-proxy
  template:
    metadata:
      labels:
        tier: stats
        app: hue-collect-proxy
    spec:
      hostPID: true
      hostIPC: true
      hostNetwork: true
      containers:
      - name: hue-collect-proxy
        image: busybox 
```

在本节中，我们探讨了如何在 Kubernetes 上管理 Hue 平台，并确保 Hue 组件在准备好时能够可靠地部署和访问，利用诸如初始化容器、就绪门和 DaemonSets 等功能。在接下来的章节中，我们将讨论 Hue 平台未来可能的发展方向。

# 用 Kubernetes 发展 Hue 平台

在本节中，我们将讨论扩展 Hue 平台并服务于其他市场和社区的其他方式。问题总是，哪些 Kubernetes 功能和能力可以帮助我们应对新的挑战或需求？

这是一个假设性章节，旨在从宏观角度思考，使用 Hue 作为一个复杂系统的例子。

## 在企业中使用 Hue

企业通常无法在云端运行，可能是出于安全性和合规性原因，或是由于性能原因，因为系统必须与不适合迁移到云端的数据和遗留系统进行协作。无论如何，企业版的 Hue 必须支持本地集群和/或裸金属集群。

虽然 Kubernetes 最常在云端部署，并且拥有专门的云服务提供商接口，但它并不依赖于云环境，可以在任何地方部署。它确实需要更多的专业知识，但那些已经在自己数据中心运行系统的企业组织，可能拥有这样的专业知识或能培养出来。

## 用 Hue 推动科学进步

Hue 在集成来自多个来源的信息方面非常出色，对科学界来说将是一个巨大的福音。试想一下，Hue 如何帮助不同学科的科学家之间进行多学科的协作。

一个科学社区的网络可能需要在多个地理位置分布的集群之间进行部署。这时，便引入了多集群 Kubernetes。Kubernetes 考虑到了这一使用场景并不断发展其支持。我们将在*第十一章*，《在多个集群上运行 Kubernetes》中详细讨论这一话题。

## 用 Hue 教育未来的孩子

Hue 可以用于教育，并为在线教育系统提供多种服务。然而，隐私问题可能会阻止将 Hue 部署为一个单一的集中式系统来服务于孩子们。一个可能的方案是设置一个集群，并为不同的学校配置命名空间。另一个部署选项是每个学校或县拥有自己的 Hue Kubernetes 集群。在第二种情况下，Hue 在教育领域的应用必须非常易于操作，以适应那些没有太多技术专长的学校。Kubernetes 可以通过提供自愈和自动扩展的功能和能力，帮助 Hue 尽可能接近零管理。

# 总结

在本章中，我们设计并规划了 Hue 平台的开发、部署和管理——这是一个基于微服务架构构建的虚拟全知全能系统。我们使用 Kubernetes 作为底层编排平台，当然，并深入探讨了它的许多概念和资源。特别地，我们专注于为长期运行的服务部署 pods，而不是为短期任务或定时任务部署 jobs，探索了内部服务与外部服务的区别，还使用了命名空间来划分 Kubernetes 集群。我们研究了 Kubernetes 的各种工作负载调度机制。接着，我们探讨了如何通过存活探针、就绪探针、启动探针、初始化容器和守护进程集来管理像 Hue 这样的庞大系统。

现在你应该能够熟练地设计由微服务组成的 Web 规模系统，并理解如何在 Kubernetes 集群中部署和管理它们。

在下一章，我们将探讨一个至关重要的领域——存储。数据是王者，但它通常是系统中最不灵活的部分。Kubernetes 提供了一个存储模型，并提供了多种与各种存储解决方案集成的选项。
