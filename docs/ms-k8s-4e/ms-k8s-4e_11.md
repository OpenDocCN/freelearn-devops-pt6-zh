

# 第十一章：在多个集群上运行 Kubernetes

在这一章中，我们将进一步探讨在多个云平台和多个集群上运行 Kubernetes 和部署工作负载的选项。由于单个 Kubernetes 集群有其限制，一旦超过这些限制，你就必须运行多个集群。一个典型的 Kubernetes 集群是一个紧密连接的单元，所有组件都在相对靠近的地方运行，并通过快速网络（通常是物理数据中心或云提供商的可用区）连接。这适用于许多用例，但也有一些重要的用例，系统需要超越单个集群的规模，或者集群需要跨多个可用区扩展。

这是 Kubernetes 当前非常活跃的一个领域。在本书的前一版中，这一章介绍了 Kubernetes 联邦和 Gardener。自那时以来，Kubernetes 联邦项目已被弃用。目前有许多项目提供不同类型的多集群解决方案，如直接管理、虚拟 Kubelet 解决方案，以及非常独特的 gardener.cloud 项目。

我们将涵盖的主题包括：

+   扩展集群与多个集群的对比

+   Kubernetes 集群联邦的历史

+   Cluster API

+   Karmada

+   Clusternet

+   Clusterpedia

+   开放集群管理

+   虚拟 Kubelet 解决方案

+   Gardener 项目简介

# 扩展 Kubernetes 集群与多集群 Kubernetes 的对比

运行多个 Kubernetes 集群的原因有几个：

+   你希望在集群运行所在的地理区域出现问题时有冗余

+   你需要更多的节点或 pod，而单个 Kubernetes 集群无法支持

+   你希望因安全原因在不同集群之间隔离工作负载

由于第一个原因，可以使用扩展集群；而出于其他原因，你必须运行多个集群。

## 理解扩展的 Kubernetes 集群

扩展集群（也叫广域集群）是一个单一的 Kubernetes 集群，其中控制平面节点和工作节点分布在多个地理可用区或区域。云提供商为 HA 管理的 Kubernetes 集群提供这种模式。

### 扩展集群的优点

扩展集群模型有几个优点：

+   你的集群，通过适当的冗余，能够防止数据中心故障作为 **SPOF**（**单点故障**）

+   操作单个 Kubernetes 集群的简便性是一个巨大的优势（日志记录、度量和升级）

+   当你运行自己的非托管扩展集群时，可以透明地将其扩展到其他位置（本地、边缘或其他云提供商）

### 扩展集群的缺点

然而，扩展模型也有其缺点：

+   你不能超过单个 Kubernetes 集群的限制

+   由于跨区网络连接导致的性能下降

+   在云端跨区网络的费用可能相当可观

+   集群升级是一个全有或全无的事务

简而言之，拥有拉伸集群的选项是好事，但如果一些缺点是无法接受的，则要准备切换到多集群模型。

## 理解多集群 Kubernetes

多集群 Kubernetes 意味着提供多个 Kubernetes 集群。基于前述的各种原因，大规模系统通常无法部署在单个集群上。这意味着你需要提供多个 Kubernetes 集群，然后弄清如何在所有这些集群上部署你的工作负载，并处理各种使用案例，比如一些集群不可用或性能下降的情况。这带来了更多的自由度。

### 多集群 Kubernetes 的优点

这里是多集群模型的一些好处：

+   系统可以任意扩展 —— 集群数量没有固有的限制

+   在 RBAC 级别为敏感工作负载提供集群级别的隔离

+   在不产生过多成本的情况下（只要大部分流量仍然保持在同一云服务提供商的区域内），利用多个云服务提供商

+   升级和执行增量操作，即使是针对整个集群的操作

### 多集群 Kubernetes 的缺点

然而，多集群模型也存在一些非常不平凡的缺点：

+   部署和管理一系列集群的非常高复杂度

+   需要弄清如何连接所有的集群

+   需要弄清如何存储并提供对所有集群中数据的访问

+   在设计多集群部署时，有很多方式会让你自食其果

+   需要努力在所有集群中提供集中观测能力

目前有一些问题的解决方案，但目前还没有一个清晰的赢家可以轻松配置以适应你组织的多集群结构的需求。相反，你需要根据组织的具体问题来适应和解决问题。

# Kubernetes 中集群联合的历史

在书的早期版本中，我们讨论了 Kubernetes 集群联合作为管理多个 Kubernetes 集群的单一概念集群的解决方案。不幸的是，自 2019 年以来，该项目一直处于停滞状态，并且 Kubernetes 多集群**特别兴趣小组（SIG）**正在考虑将其存档。在我们描述更现代化的方法之前，让我们先了解一些历史背景。谈论一个像 Kubernetes 这样的项目的历史，甚至在 2014 年之前根本不存在，这是很有趣的，但是其发展速度和大量贡献者的参与加速了 Kubernetes 的进化。这对于 Kubernetes 联合尤为重要。

2015 年 3 月，Kubernetes 集群联合提案的第一个版本被发布。当时它被亲切地称为“Ubernetes”。基本想法是重用现有的 Kubernetes API 来管理多个集群。这个提案现在被称为 Federation V1，经过了几轮修订和实施，但始终未达到广泛使用，且主仓库已被退休：[`github.com/kubernetes-retired/federation`](https://github.com/kubernetes-retired/federation)。

SIG 多集群工作组意识到，多集群问题比最初认为的要复杂得多。解决这一特定问题有多种方法，而且没有一种适用于所有的解决方案。集群联合的新方向是使用专门的 API 来进行联合。为此，创建并实施了一个新项目和一组工具——Kubernetes Federation V2：[`github.com/kubernetes-sigs/kubefed`](https://github.com/kubernetes-sigs/kubefed)。

不幸的是，这个项目也没有取得成功，关于多集群 SIG 的共识是，由于该项目未再维护，因此需要将其归档。

查看 2022 年 8 月 9 日的会议记录：[`tinyurl.com/sig-multicluster-notes`](https://tinyurl.com/sig-multicluster-notes)。

目前有许多项目正在快速推进，试图解决多集群问题，它们都在不同层面上进行操作。让我们来看看其中一些突出项目。这里的目标仅仅是介绍这些项目以及它们的独特之处。深入探讨每个项目超出了本章的范围。不过，我们将在*第十七章*《在生产环境中运行 Kubernetes》中深入探讨其中一个项目——Cluster API。

# Cluster API

Cluster API（也叫 CAPI）是 Cluster Lifecycle SIG 的一个项目。其目标是简化多个 Kubernetes 集群的配置、升级和操作。它支持基于 kubeadm 的集群以及通过专用提供者管理的集群。它有一个很酷的标志，灵感来自著名的“从头到尾全是乌龟”的故事。这个想法是，Cluster API 使用 Kubernetes 来管理 Kubernetes 集群。

![](img/B18998_11_01.png)

图 11.1：Cluster API 标志

## Cluster API 架构

Cluster API 具有非常清晰和可扩展的架构。其主要组件包括：

+   管理集群

+   工作集群

+   启动提供者

+   基础设施提供者

+   控制平面

+   自定义资源

![](img/B18998_11_02.png)

图 11.2：Cluster API 架构

让我们理解这些组件中的每一个的作用以及它们如何相互作用。

## 管理集群

管理集群是一个负责管理其他 Kubernetes 集群（工作集群）的 Kubernetes 集群。它运行 Cluster API 控制平面和提供者，并托管表示其他集群的 Cluster API 自定义资源。

clusterctl CLI 可以用来操作管理集群。clusterctl CLI 是一个命令行工具，提供了大量的命令和选项。如果你想通过其 CLI 来试验 Cluster API，请访问[`cluster-api.sigs.k8s.io/clusterctl/overview.html`](https://cluster-api.sigs.k8s.io/clusterctl/overview.html)。

## 工作集群

工作集群只是一个普通的 Kubernetes 集群。这些集群是开发人员用来部署其工作负载的。工作集群无需意识到它们是由 Cluster API 管理的。

## 启动提供者

当 CAPI 创建一个新的 Kubernetes 集群时，在创建工作集群的控制平面和最终的工作节点之前，它需要证书。这是启动提供者的工作。它确保满足所有要求，并最终将工作节点加入控制平面。

## 基础设施提供者

基础设施提供者是一个可插拔组件，使 CAPI 能够在不同的基础设施环境中工作，例如云提供商或裸金属基础设施提供商。基础设施提供者实现了 CAPI 定义的一组接口，以提供对计算和网络资源的访问。

请查看当前提供者列表：[`cluster-api.sigs.k8s.io/reference/providers.html`](https://cluster-api.sigs.k8s.io/reference/providers.html)。

## 控制平面

Kubernetes 集群的控制平面由 API 服务器、etcd 状态存储、调度器以及运行控制循环以调和集群中资源的控制器组成。工作集群的控制平面可以通过不同的方式来提供。CAPI 支持以下模式：

+   基于机器的——控制平面组件作为静态 Pod 部署在专用机器上

+   基于 Pod 的——控制平面组件通过`Deployments`和`StatefulSet`部署，API 服务器作为`Service`暴露

+   外部——控制平面由外部提供者（通常是云提供商）提供和管理

## 自定义资源

自定义资源代表由 CAPI 管理的 Kubernetes 集群和机器以及其他附加资源。自定义资源数量众多，其中一些仍被视为实验性。主要的 CRD 有：

+   `Cluster`

+   `ControlPlane`（代表控制平面机器）

+   `MachineSet`（代表工作机器）

+   `MachineDeployment`

+   `Machine`

+   `MachineHealthCheck`

这些通用资源中的一些引用了基础设施提供者提供的相应资源。

以下图示展示了代表集群和机器集的控制平面资源之间的关系：

![](img/B18998_11_03.png)

图 11.3：Cluster API 控制平面资源

CAPI 还具有一组额外的实验性资源，代表一个受管的云提供者环境：

+   `MachinePool`

+   `ClusterResourceSet`

+   `ClusterClass`

查看 [`github.com/kubernetes-sigs/cluster-api`](https://github.com/kubernetes-sigs/cluster-api) 获取更多细节。

# Karmada

Karmada 是一个 CNCF 沙盒项目，专注于跨多个 Kubernetes 集群部署和运行工作负载。它的特点是你无需修改应用程序配置。虽然 CAPI 专注于集群生命周期管理，但 Karmada 在你已经拥有一组 Kubernetes 集群，并希望在所有集群中部署工作负载时发挥作用。从概念上讲，Karmada 是对已废弃的 Kubernetes 联邦项目的现代化改进。

它可以与 Kubernetes 在云端、本地和边缘环境中一起工作。

查看 [`github.com/karmada-io/karmada`](https://github.com/karmada-io/karmada)。

让我们来看看 Karmada 的架构。

## Karmada 架构

Karmada 深受 Kubernetes 的启发。它提供了一个多集群控制平面，包含与 Kubernetes 控制平面类似的组件：

+   Karmada API 服务器

+   Karmada 控制器管理器

+   Karmada 调度器

如果你理解 Kubernetes 的工作原理，那么理解 Karmada 如何将其 1:1 扩展到多个集群就非常简单。

下图展示了 Karmada 的架构：

![](img/B18998_11_04.png)

图 11.4：Karmada 架构

## Karmada 概念

Karmada 以多个作为 Kubernetes CRD 实现的概念为中心。你使用这些概念定义和更新你的应用程序和服务，Karmada 会确保你的工作负载在多集群系统中正确部署和运行。

让我们来看看这些概念。

### ResourceTemplate

资源模板看起来就像常规的 Kubernetes 资源，例如 `Deployment` 或 `StatefulSet`，但它并不会实际部署到 Karmada 控制平面。它仅作为一个蓝图，最终将被部署到成员集群。

### PropagationPolicy

传播策略决定了资源模板应该部署到哪里。这里有一个简单的传播策略，它将 `nginx` 部署到两个集群，分别叫做 `member1` 和 `member2`：

```
apiVersion: policy.karmada.io/v1alpha1
kind: PropagationPolicy
metadata:
  name: cool-policy
spec:
  resourceSelectors:
    - apiVersion: apps/v1
      kind: Deployment
      name: nginx
  placement:
    clusterAffinity:
      clusterNames:
        - member1
        - member2 
```

### OverridePolicy

PropagationPolicy 跨多个集群操作，但有时会有例外。OverridePolicy 允许你应用细粒度规则，以覆盖现有的传播策略。规则有几种类型：

+   `ImageOverrider`：专门用于覆盖工作负载的镜像

+   `CommandOverrider`：专门用于覆盖工作负载的命令

+   `ArgsOverrider`：专门用于覆盖工作负载的参数

+   `PlaintextOverrider`：一个通用工具，用于覆盖任何类型的资源

## 额外功能

Karmada 还有更多功能，例如：

+   多集群重新调度

+   重新调度

+   多集群故障切换

+   多集群服务发现

查看 Karmada 文档了解更多细节：[`karmada.io/docs/`](https://karmada.io/docs/)。

# Clusternet

Clusternet 是一个有趣的项目。它围绕管理多个 Kubernetes 集群的理念展开，目标是像“访问互联网”一样管理集群（因此取名为“Clusternet”）。它支持云端、本地、边缘和混合集群。Clusternet 的核心功能包括：

+   Kubernetes 多集群管理与治理

+   应用协调

+   通过 kubectl 插件提供的 CLI

+   通过对 Kubernetes Client-Go 库的包装进行编程访问

## Clusternet 架构

Clusternet 架构类似于 Karmada，但更为简化。存在一个父集群，它运行 Clusternet hub 和 Clusternet scheduler。在每个子集群上，运行一个 Clusternet agent。下图展示了各个组件之间的结构和交互：

![](img/B18998_11_05.png)

图 11.5：Clusternet 架构

### Clusternet hub

Hub 有多个角色。它负责批准集群注册请求，并为所有子集群创建命名空间、服务账户和 RBAC 资源。它还充当一个聚合的 API 服务器，维护与子集群上代理的 WebSocket 连接。Hub 还提供类似 Kubernetes 的 API，将请求代理到每个子集群。最后但同样重要的是，Hub 协调从单一资源集群部署应用程序及其依赖项到多个集群。

### Clusternet scheduler

Clusternet scheduler 是负责确保资源（在 Clusternet 术语中称为 feeds）根据名为 `SchedulingStrategy` 的策略在所有子集群间部署和均衡的组件。

### Clusternet agent

Clusternet agent 在每个子集群上运行，并与 Hub 通信。子集群上的 agent 相当于节点上的 kubelet。它有多个角色。该 agent 将其子集群注册到父集群。该 agent 向 Hub 提供心跳信息，包含许多内容，如 Kubernetes 版本、运行平台、健康状况、就绪状态和工作负载的存活状态。该 agent 还设置与父集群 Hub 的 WebSocket 连接，以便通过单一 TCP 连接实现全双工通信通道。

## 多集群部署

Clusternet 将多集群部署建模为订阅和 feeds。它提供一个 `Subscription` 自定义资源，可以根据不同的标准将一组资源（称为 feeds）部署到多个集群（称为订阅者）。以下是一个 `Subscription` 示例，它将一个 `Namespace`、一个 `Service` 和一个 `Deployment` 部署到多个集群，并带有 `clusters.clusternet.io/cluster-id` 标签：

```
# examples/dynamic-dividing-scheduling/subscription.yaml
apiVersion: apps.clusternet.io/v1alpha1
kind: Subscription
metadata:
  name: dynamic-dividing-scheduling-demo
  namespace: default
spec:
  subscribers: # filter out a set of desired clusters
    - clusterAffinity:
        matchExpressions:
          - key: clusters.clusternet.io/cluster-id
            operator: Exists
  schedulingStrategy: Dividing
  dividingScheduling:
    type: Dynamic
    dynamicDividing:
      strategy: Spread # currently we only support Spread dividing strategy
  feeds: # defines all the resources to be deployed with
    - apiVersion: v1
      kind: Namespace
      name: qux
    - apiVersion: v1
      kind: Service
      name: my-nginx-svc
      namespace: qux
    - apiVersion: apps/v1 # with a total of 6 replicas
      kind: Deployment
      name: my-nginx
      namespace: qux 
```

更多详情请参见 [`clusternet.io`](https://clusternet.io)。

# Clusterpedia

Clusterpedia 是一个 CNCF 沙箱项目。它的中心隐喻是 Kubernetes 集群的维基百科。它具有多集群搜索、过滤、字段选择和排序等多种能力。这是不同寻常的，因为它是一个只读项目。它不提供帮助管理集群或部署工作负载。它专注于观察您的集群。

## Clusterpedia 架构

架构类似于其他多集群项目。有一个控制平面元素，运行 Clusterpedia API 服务器和 ClusterSynchro 管理器组件。对于每个观察到的集群，都有一个专用组件称为集群同步器，将集群的状态同步到 Clusterpedia 的存储层中。架构的一个最有趣的方面是 Clusterpedia 聚合 API 服务器，它使所有集群看起来像一个巨大的逻辑集群。请注意，Clusterpedia API 服务器和 ClusterSynchro 管理器松散耦合，不直接互动。它们只是从共享的存储层读取和写入。

![](img/B18998_11_06.png)

图 11.6：Clusterpedia 架构

让我们看看每个组件，并理解它们的目的是什么。

### Clusterpedia API 服务器

Clusterpedia API 服务器是一个聚合 API 服务器。这意味着它会向 Kubernetes API 服务器注册自己，并通过自定义端点扩展标准的 Kubernetes API 服务器。当请求发送到 Kubernetes API 服务器时，它会将其转发到 Clusterpedia API 服务器，后者访问存储层来满足这些请求。Kubernetes API 服务器充当 Clusterpedia 处理的请求的转发层。

这是 Kubernetes 的一个高级方面。我们将在 *第十五章* *扩展 Kubernetes* 中讨论 API 服务器聚合。

### ClusterSynchro 管理器

Clusterpedia 观察多个集群以提供其搜索、过滤和聚合功能。一种实现方式是，每当有请求进来时，Clusterpedia 将查询所有观察到的集群，收集结果并返回。这种方法非常问题，因为某些集群可能响应缓慢，并且类似的请求需要返回相同的信息，这是浪费和昂贵的。相反，ClusterSynchro 管理器会集体将每个观察到的集群的状态同步到 Clusterpedia 存储中，Clusterpedia API 服务器可以快速响应。

### 存储层

存储层是一个抽象层，用于存储所有观察到的集群的状态。它提供了一个可以由不同存储组件实现的统一接口。Clusterpedia API 服务器和 ClusterSynchro 管理器与存储层接口进行交互，从不直接互相通信。

### 存储组件

存储组件是实际的数据存储，执行存储层接口并存储观察到的集群状态。Clusterpedia 旨在支持不同的存储组件，以便为用户提供灵活性。目前，支持的存储组件包括 MySQL、Postgres 和 Redis。

## 导入集群

要将集群导入到 Clusterpedia，你需要定义一个 PediaCluster 自定义资源。这非常简单：

```
apiVersion: cluster.clusterpedia.io/v1alpha2
kind: PediaCluster
metadata:
  name: cluster-example
spec:
  apiserver: "https://10.30.43.43:6443"
  kubeconfig:
  caData:
  tokenData:
  certData:
  keyData:
  syncResources: [] 
```

你需要提供访问集群的凭证，然后 Clusterpedia 会接管并同步其状态。

## 高级多集群搜索

这是 Clusterpedia 的亮点。你可以通过 API 或 kubectl 访问 Clusterpedia 集群。通过 URL 访问时，它看起来像是你访问了聚合的 API 服务器端点：

```
kubectl get --raw="/apis/clusterpedia.io/v1beta1/resources/apis/apps/v1/deployments?clusters=cluster-1,cluster-2" 
```

你可以将目标集群指定为查询参数（在此案例中，`cluster-1` 和 `cluster-2`）。

通过 kubectl 访问时，你指定目标集群为标签（在此案例中，`"search.clusterpedia.io/clusters in (cluster-1,cluster-2)"`）：

```
kubectl --cluster clusterpedia get deployments -l "search.clusterpedia.io/clusters in (cluster-1,cluster-2)" 
```

还有其他用于命名空间和资源名称的搜索标签和查询：

+   `search.clusterpedia.io/namespaces`（查询参数是 `namespaces`）

+   `search.clusterpedia.io/names`（查询参数是 `names`）

还有一个实验性的模糊搜索标签 `internalstorage.clusterpedia.io/fuzzy-name` 用于资源名称，但没有查询参数。这很有用，因为资源通常会生成带有随机后缀的名称。

你还可以按创建时间进行搜索：

+   `search.clusterpedia.io/before`（查询参数是 `before`）

+   `search.clusterpedia.io/since`（查询参数是 `since`）

其他功能包括按资源标签或字段选择器进行过滤，以及使用 `OrderBy` 和 `Paging` 来组织结果。

## 资源集合

另一个重要概念是资源集合。标准的 Kubernetes API 提供了一个简单的 REST API，你可以一次列出或获取一种资源。然而，用户通常希望同时获取多种类型的资源。例如，带有特定标签的 `Deployment`、`Service` 和 `HorizontalPodAutoscaler`。这需要通过标准的 Kubernetes API 进行多次调用，即使这些资源都在一个集群中。

Clusterpedia 定义了一个 `CollectionResource`，将以下类别的资源组合在一起：

+   `any`（所有资源）

+   `workloads`（`Deployments`、`StatefulSets` 和 `DaemonSets`）

+   `kuberesources`（除工作负载外的所有资源）

你可以通过传递 API 组和资源类型，在一次 API 调用中搜索任意组合的资源：

```
kubectl get --raw "/apis/clusterpedia.io/v1beta1/collectionresources/any?onlyMetadata=true&groups=apps&resources=batch/jobs,batch/cronjobs" 
```

详情请见 [`github.com/clusterpedia-io/clusterpedia`](https://github.com/clusterpedia-io/clusterpedia)。

# 开放集群管理

**开放集群管理** (**OCM**) 是一个 CNCF 沙箱项目，旨在进行多集群管理，以及多集群调度和工作负载放置。它的特点是紧密遵循许多 Kubernetes 概念，通过插件实现可扩展性，并与其他开源项目（如）进行强集成：

+   Submariner

+   Clusternet（我们之前介绍过的）

+   KubeVela

OCM 的范围涵盖集群生命周期、应用生命周期和治理。

让我们来看一下 OCM 的架构。

## OCM 架构

OCM 的架构遵循中心和辐射模型。它有一个中心集群，即 OCM 控制平面，用于管理多个其他集群（辐射集群）。

控制平面的中心集群运行两个控制器：注册控制器和放置控制器。此外，控制平面还运行多个管理插件，它们是 OCM 可扩展性的基础。在每个托管集群上，都有一个所谓的 Klusterlet，它包含一个注册代理和一个工作代理，与中心集群上的注册控制器和放置控制器进行交互。然后，还有插件代理与中心集群上的插件进行交互。

以下图示展示了 OCM 各个组件如何进行通信：

![](img/B18998_11_07.png)

图 11.7：OCM 架构

让我们来看一下 OCM 的不同方面。

## OCM 集群生命周期

集群注册是 OCM 安全多集群故事中的重要部分。OCM 以其安全的双重确认握手注册为傲。由于中心-辐射式集群可能有不同的管理员，这种模型为每一方提供了对不希望的请求的保护。每一方都可以随时终止关系。

以下图示展示了注册过程（CSR 表示证书签名请求）：

![](img/B18998_11_08.png)

图 11.8：OCM 注册过程

## OCM 应用生命周期

OCM 应用生命周期支持在多个集群之间创建、更新和删除资源。

主要的构建块是 `ManifestWork` 自定义资源，它可以定义多个资源。下面是一个仅包含单个 `Deployment` 的示例：

```
apiVersion: work.open-cluster-management.io/v1
kind: ManifestWork
metadata:
  namespace: <target managed cluster>
  name: awesome-workload
spec:
  workload:
    manifests:
      - apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: hello
          namespace: default
        spec:
          selector:
            matchLabels:
              app: hello
          template:
            metadata:
              labels:
                app: hello
            spec:
              containers:
                - name: hello
                  image: quay.io/asmacdo/busybox
                  command:
                    ["sh", "-c", 'echo "Hello, Kubernetes!" && sleep 3600'] 
```

`ManifestWork` 在中心集群上创建，并根据命名空间映射部署到目标集群。每个目标集群在中心集群中都有一个表示它的命名空间。运行在目标集群上的工作代理将监控它们命名空间中所有来自中心集群的 `ManifestWork` 资源，并同步更改。

## OCM 治理、风险与合规性

OCM 提供了一种基于策略、策略模板和策略控制器的治理模型。这些策略可以绑定到特定的集群集合，从而实现精细的控制。

这是一个示例策略，要求存在一个名为 `Prod` 的命名空间：

```
apiVersion: policy.open-cluster-management.io/v1
kind: Policy
metadata:
  name: policy-namespace
  namespace: policies
  annotations:
    policy.open-cluster-management.io/standards: NIST SP 800-53
    policy.open-cluster-management.io/categories: CM Configuration Management
    policy.open-cluster-management.io/controls: CM-2 Baseline Configuration
spec:
  remediationAction: enforce
  disabled: false
  policy-templates:
    - objectDefinition:
        apiVersion: policy.open-cluster-management.io/v1
        kind: ConfigurationPolicy
        metadata:
          name: policy-namespace-example
        spec:
          remediationAction: inform
          severity: low
          object-templates:
            - complianceType: MustHave
              objectDefinition:
                kind: Namespace # must have namespace 'prod'
                apiVersion: v1
                metadata:
                  name: prod 
```

查看 [`open-cluster-management.io/`](https://open-cluster-management.io/) 获取更多详情。

# 虚拟 Kubelet

虚拟 Kubelet 是一个令人着迷的项目。它伪装成一个 kubelet，将 Kubernetes 连接到其他 API，例如 AWS Fargate 或 Azure ACI。虚拟 Kubelet 对 Kubernetes 集群看起来就像一个节点，但它背后的计算资源是抽象的。虚拟 Kubelet 对 Kubernetes 集群来说看起来就像另一个节点：

![](img/B18998_11_09.png)

图 11.9：虚拟 Kubelet，对 Kubernetes 集群看起来像一个常规节点

虚拟 Kubelet 的特性包括：

+   创建、更新和删除 pod

+   访问容器日志和指标

+   获取 pod、pods 和 pod 状态

+   管理容量

+   访问节点地址、节点容量和节点守护进程端点

+   选择操作系统

+   支持你自己的虚拟网络

详情请见 [`github.com/virtual-kubelet/virtual-kubelet`](https://github.com/virtual-kubelet/virtual-kubelet)。

这个概念也可以用来连接多个 Kubernetes 集群，许多项目采用了这种方式。让我们简要了解一些使用虚拟 Kubelet 进行多集群管理的项目，例如 tensile-kube、Admiralty 和 Liqo。

## Tensile-kube

Tensile-kube 是 Virtual Kubelet 组织在 GitHub 上的一个子项目。

Tensile-kube 提供以下功能：

+   集群资源的自动发现

+   异步通知 pod 修改

+   完全访问 pod 日志和 kubectl exec

+   pod 的全局调度

+   使用 descheduler 重新调度 pod

+   PV/PVC

+   服务

Tensile-kube 使用上层集群（包含虚拟 Kubelet 的集群）和下层集群（作为虚拟节点在上层集群中暴露的集群）这一术语。

这是 tensile-kube 架构：

![](img/B18998_11_10.png)

图 11.10：Tensile-kube 架构

详情请见 [`github.com/virtual-kubelet/tensile-kube`](https://github.com/virtual-kubelet/tensile-kube)。

## Admiralty

Admiralty 是一个由商业公司支持的开源项目。Admiralty 采用了虚拟 Kubelet 概念，并构建了一个复杂的多集群调度和编排解决方案。目标集群作为源集群中的虚拟节点来表示。它有一个相当复杂的架构，涉及三层调度。每当在代理上创建 pod 时，会在源集群上创建 pods，在每个目标集群上创建候选 pod，最终选定一个候选 pod 作为委托 pod，后者是一个真实的 pod，实际运行其容器。所有这一切都由基于 Kubernetes 调度框架构建的自定义多集群调度器支持。要在 Admiralty 上调度工作负载，你需要为任何 pod 模板添加注解 `multicluster.admiralty.io/elect=""`，然后 Admiralty 将接管后续操作。

这是一个展示不同组件之间相互作用的图示：

![](img/B18998_11_11.png)

图 11.11：Admiralty 架构

Admiralty 提供以下功能：

+   高可用

+   实时灾难恢复

+   动态**CDN**（**内容分发网络**）

+   多集群工作流

+   支持边缘计算、物联网（IoT）和 5G

+   治理

+   集群升级

+   集群作为牲畜的抽象

+   全球资源联合

+   云爆发与套利

参见[`admiralty.io`](https://admiralty.io)了解更多详情。

## Liqo

Liqo 是一个基于液态计算概念的开源项目。让你的任务和数据漂浮并找到最佳运行位置。它的范围非常广泛，不仅针对在多个集群中运行 Pod 的计算方面，还提供了网络架构和存储架构。这些连接集群并跨集群管理数据的方面，通常比单纯运行工作负载更难解决。

在 Liqo 的术语中，管理集群称为本地集群，目标集群称为外部集群。本地集群中的虚拟节点称为“大”节点，它们代表外部集群。

Liqo 利用 IP 地址映射实现了跨所有可能存在内部 IP 冲突的外部集群的扁平 IP 地址空间。

Liqo 过滤并批处理来自外部集群的事件，以减轻本地集群的压力。

下面是 Liqo 架构的图示：

![](img/B18998_11_12.png)

图 11.12：Liqo 架构

参见[`liqo.io`](https://liqo.io)了解更多详情。

接下来，让我们深入了解 Gardener 项目，它采用了不同的方法。

# 介绍 Gardener 项目

Gardener 项目是由 SAP 开发的开源项目。它使你能够高效、经济地管理成千上万个（没错，是成千上万个！）Kubernetes 集群。Gardener 解决了一个非常复杂的问题，而且解决方案优雅但并不简单。Gardener 是唯一一个同时处理集群生命周期和应用生命周期的项目。

在本节中，我们将涵盖 Gardener 的术语及其概念模型，深入探讨其架构，并了解其可扩展性功能。Gardener 的主要主题是使用 Kubernetes 来管理 Kubernetes 集群。理解 Gardener 的一个好方法是把它看作是 Kubernetes 控制平面即服务。

参见[`gardener.cloud`](https://gardener.cloud)了解更多详情。

## 理解 Gardener 的术语

如你所料，Gardener 项目采用了植物学术语来描述其世界。园区是一个 Kubernetes 集群，负责管理种子集群。种子是一个 Kubernetes 集群，负责管理一组 shoot 集群。shoot 集群是运行实际工作负载的 Kubernetes 集群。

Gardener 背后的酷点子是，shoot 集群仅包含工作节点。所有 shoot 集群的控制平面作为 Kubernetes Pods 和服务运行在 seed 集群中。

以下图描述了 Gardener 的结构及其各个组件之间的关系：

![](img/B18998_11_13.png)

图 11.13：Gardener 项目结构

不要慌张！在所有这些复杂性背后，是一个清晰明了的概念模型。

## 理解 Gardener 的概念模型

Gardener 的架构图可能会让人感到不知所措。让我们慢慢解开它，揭示其背后的基本原则。Gardener 真正拥抱了 Kubernetes 的精神，将管理大量 Kubernetes 集群的复杂性交给 Kubernetes 自身。Gardener 的核心是一个聚合的 API 服务器，通过各种控制器管理一组自定义资源。它充分利用了 Kubernetes 的可扩展性，这种做法在 Kubernetes 社区中很常见。定义一组自定义资源，让 Kubernetes 来为你管理它们。Gardener 的创新之处在于，它将这一方法推向极致，甚至抽象化了 Kubernetes 基础设施的某些部分。

在“正常”的 Kubernetes 集群中，控制平面与工作节点运行在同一个集群中。通常，在大型集群中，Kubernetes API 服务器和 etcd 等控制平面组件会运行在专用节点上，不与工作节点混合。Gardener 从多个集群的角度进行思考，它将所有 shoot 集群的控制平面集中管理，并且有一个种子集群来管理它们。因此，shoot 集群的 Kubernetes 控制平面作为常规的 Kubernetes `Deployments` 在种子集群中进行管理，这样 Kubernetes 会自动提供复制、监控、自愈和滚动更新等功能。

所以，Kubernetes shoot 集群的控制平面类似于一个 `Deployment`。而种子集群则映射到一个 Kubernetes 节点，它管理多个 shoot 集群。建议每个云提供商使用一个种子集群。Gardener 的开发人员实际上在为种子集群开发一个 gardenlet 控制器，它类似于节点上的 kubelet。

如果种子集群像 Kubernetes 节点，那么管理这些种子集群的 Garden 集群就像是管理其工作节点的 Kubernetes 集群。

通过大力推进 Kubernetes 模型，Gardener 项目利用了 Kubernetes 的优势，达到了构建一个从零开始非常难实现的鲁棒性和性能。

让我们深入探讨架构。

## 深入了解 Gardener 架构

Gardener 会在种子集群中为每个 shoot 集群创建一个 Kubernetes 命名空间。它将 shoot 集群的证书作为 Kubernetes 密钥存储在种子集群中。

### 管理集群状态

每个集群的 etcd 数据存储作为一个带有单个副本的 StatefulSet 部署。此外，事件会存储在一个独立的 etcd 实例中。etcd 数据会定期快照，并存储在远程存储中，以备份和恢复使用。这使得当集群丧失控制平面时（例如，整个种子集群变得无法访问时），能够非常快速地恢复。请注意，当种子集群出现故障时，shoot 集群仍会照常运行。

### 管理控制平面

如前所述，射出集群 X 的控制平面运行在一个独立的种子集群中，而工作节点运行在射出集群中。这意味着射出集群中的 Pods 可以使用内部 DNS 进行相互定位，但与运行在种子集群中的 Kubernetes API 服务器的通信必须通过外部 DNS 来完成。这意味着 Kubernetes API 服务器作为 `LoadBalancer` 类型的 `Service` 运行。

### 准备基础设施

在创建新的射出集群时，提供必要的基础设施非常重要。Gardener 使用 Terraform 来完成这项任务。根据射出集群的规格，动态生成一个 Terraform 脚本，并将其作为 ConfigMap 存储在种子集群中。为了简化这一过程，一个专用组件（Terraformer）作为一个作业运行，执行所有的资源配置，然后将状态写入一个单独的 ConfigMap 中。

### 使用机器控制器管理器

为了以与提供商无关的方式配置节点，Gardener 提供了多个自定义资源，例如 `MachineDeployment`、`MachineClass`、`MachineSet` 和 `Machine`。它们与 Kubernetes 集群生命周期组协作，以统一其抽象，因为它们之间有很多重叠。此外，Gardener 还利用了集群自动扩缩器，以减轻节点池扩展和收缩的复杂性。

### 跨集群网络

种子集群和射出集群可以运行在不同的云提供商上。射出集群中的工作节点通常部署在私有网络中。由于控制平面需要与工作节点（主要是 kubelet）紧密交互，Gardener 创建了一个 VPN 以便直接通信。

### 监控集群

可观测性是运营复杂分布式系统的重要组成部分。Gardener 提供了丰富的监控功能，内置了顶级的开源项目，如部署在花园集群中的中央 Prometheus 服务器，收集所有种子集群的信息。此外，每个射出集群都会在种子集群中获得一个独立的 Prometheus 实例。为了收集指标，Gardener 为每个集群部署了两个 `kube-state-metrics` 实例（一个用于种子集群中的控制平面，一个用于射出集群中的工作节点）。节点导出器也被部署，用于提供有关节点的额外信息。Prometheus `AlertManager` 用于在出现问题时通知操作员。Grafana 用于显示包含系统状态相关数据的仪表板。

### gardenctl CLI

你可以仅使用 kubectl 来管理 Gardener，但在探索不同集群时，你需要频繁切换配置文件和上下文。Gardener 提供了 `gardenctl` 命令行工具，它提供了更高层次的抽象，并且可以同时操作多个集群。以下是一个示例：

```
$ gardenctl ls shoots
projects:
- project: team-a
  shoots:
  - dev-eu1
  - prod-eu1
$ gardenctl target shoot prod-eu1
[prod-eu1]
$ gardenctl show prometheus
NAME           READY     STATUS    RESTARTS   AGE       IP              NODE
prometheus-0   3/3       Running   0          106d      10.241.241.42   ip-10-240-7-72.eu-central-1.compute.internal
URL: https://user:password@p.prod-eu1.team-a.seed.aws-eu1.example.com 
```

Gardener 的一个突出特点是其可扩展性。它有着广泛的适用范围，支持许多环境。让我们来看一下它是如何在设计中实现可扩展性的。

## 扩展 Gardener

Gardener 支持以下环境：

+   阿里云

+   AWS

+   Azure

+   Equinix Metal

+   GCP

+   OpenStack

+   vSphere

它像 Kubernetes 一样，从主 Gardener 仓库中的大量特定于提供商的支持开始。随着时间的推移，它借鉴了 Kubernetes 的做法，将云提供商外部化，并将这些提供商迁移到独立的 Gardener 扩展中。可以使用 CloudProfile CRD 来指定提供商，例如：

```
apiVersion: core.gardener.cloud/v1beta1
kind: CloudProfile
metadata:
  name: aws
spec:
  type: aws
  kubernetes:
    versions:
    - version: 1.24.3
    - version: 1.23.8
      expirationDate: "2022-10-31T23:59:59Z"
  machineImages:
  - name: coreos
    versions:
    - version: 2135.6.0
  machineTypes:
  - name: m5.large
    cpu: "2"
    gpu: "0"
    memory: 8Gi
    usable: true
  volumeTypes:
  - name: gp2
    class: standard
    usable: true
  - name: io1
    class: premium
    usable: true
  regions:
  - name: eu-central-1
    zones:
    - name: eu-central-1a
    - name: eu-central-1b
    - name: eu-central-1c
  providerConfig:
    apiVersion: aws.provider.extensions.gardener.cloud/v1alpha1
    kind: CloudProfileConfig
    machineImages:
    - name: coreos
      versions:
      - version: 2135.6.0
        regions:
        - name: eu-central-1
          ami: ami-034fd8c3f4026eb39
          # architecture: amd64 # optional 
```

然后，一个 shoot 集群将选择一个提供商，并使用必要的信息进行配置：

```
apiVersion: gardener.cloud/v1alpha1
kind: Shoot
metadata:
  name: johndoe-aws
  namespace: garden-dev
spec:
  cloudProfileName: aws
  secretBindingName: core-aws
  cloud:
    type: aws
    region: eu-west-1
    providerConfig:
      apiVersion: aws.cloud.gardener.cloud/v1alpha1
      kind: InfrastructureConfig
      networks:
        vpc: # specify either 'id' or 'cidr'
        # id: vpc-123456
          cidr: 10.250.0.0/16
        internal:
        - 10.250.112.0/22
        public:
        - 10.250.96.0/22
        workers:
        - 10.250.0.0/19
      zones:
      - eu-west-1a
    workerPools:
    - name: pool-01
    # Taints, labels, and annotations are not yet implemented. This requires interaction with the machine-controller-manager, see
    # https://github.com/gardener/machine-controller-manager/issues/174\. It is only mentioned here as future proposal.
    # taints:
    # - key: foo
    #   value: bar
    #   effect: PreferNoSchedule
    # labels:
    # - key: bar
    #   value: baz
    # annotations:
    # - key: foo
    #   value: hugo
      machineType: m4.large
      volume: # optional, not needed in every environment, may only be specified if the referenced CloudProfile contains the volumeTypes field
        type: gp2
        size: 20Gi
      providerConfig:
        apiVersion: aws.cloud.gardener.cloud/v1alpha1
        kind: WorkerPoolConfig
        machineImage:
          name: coreos
          ami: ami-d0dcef3
        zones:
        - eu-west-1a
      minimum: 2
      maximum: 2
      maxSurge: 1
      maxUnavailable: 0
  kubernetes:
    version: 1.11.0
    ...
  dns:
    provider: aws-route53
    domain: johndoe-aws.garden-dev.example.com
  maintenance:
    timeWindow:
      begin: 220000+0100
      end: 230000+0100
    autoUpdate:
      kubernetesVersion: true
  backup:
    schedule: "*/5 * * * *"
    maximum: 7
  addons:
    kube2iam:
      enabled: false
    kubernetes-dashboard:
      enabled: true
    cluster-autoscaler:
      enabled: true
    nginx-ingress:
      enabled: true
      loadBalancerSourceRanges: []
    kube-lego:
      enabled: true
      email: john.doe@example.com 
```

但是，Gardener 的可扩展性目标远不止于仅仅做到与提供商无关。搭建 Kubernetes 集群的整个过程涉及许多步骤。Gardener 项目的目标是让运维人员通过定义自定义资源和 Webhook，来定制每一个步骤。以下是与每个步骤相关的 CRD、变更/验证准入控制器和 Webhook 的总体流程图：

![](img/B18998_11_14.png)

图 11.14：CRD 变更和验证准入控制器的流程图

以下是构成 Gardener 可扩展性空间的 CRD 类别：

+   用于 DNS 管理的提供商，如 Route53 和 CloudDNS

+   用于 Blob 存储的提供商，包括 S3、GCS 和 ABS

+   基础设施提供商，如 AWS、GCP 和 Azure

+   支持 CoreOS Container Linux、Ubuntu 和 FlatCar Linux 等各种操作系统

+   网络插件，如 Calico、Flannel 和 Cilium

+   可选扩展，例如 Let’s Encrypt 的证书服务

我们已经深入探讨了 Gardener，这也标志着本章的结束。

# 总结

在本章中，我们探讨了多集群管理这一激动人心的领域。有许多项目从不同的角度来解决这个问题。Cluster API 项目在解决多个集群生命周期管理的子问题上有着很大的动力。许多其他项目则着眼于资源管理和应用生命周期。这些项目可以分为两类：一类是通过管理集群和被管理集群来显式管理多个集群，另一类是利用虚拟 Kubelet，将整个集群作为虚拟节点呈现在主集群中。

Gardener 项目有一种非常有趣的方式和架构。它从不同的角度解决了多个集群的问题，并专注于集群的大规模管理。它是唯一同时处理集群生命周期和应用生命周期的项目。

在这一点上，你应该已经清楚地了解了当前多集群管理的状况以及不同项目所提供的功能。你可以决定是觉得现在还为时过早，还是准备勇敢尝试。

在下一章，我们将探索 Kubernetes 上激动人心的无服务器计算世界。无服务器可以有两种不同的含义：你不需要为长时间运行的工作负载管理服务器，同时，还可以将函数作为服务运行。Kubernetes 提供了这两种形式的无服务器计算，它们都非常有用。
