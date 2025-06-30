# 第十六章：评估

# 第一章：Kubernetes 简介

1.  Minikube、Google Cloud Platform 和 Azure Kubernetes Service。

1.  虚拟机、FreeBSD Jail、LXC（Linux 容器）、Open VZ 和 Solaris 容器。

1.  内存、文件系统 CPU、线程、进程、命名空间和内存接口文件。

1.  它允许公司将增量更新交付到软件。它还允许将软件从开发者的笔记本电脑打包到生产环境。

1.  设置帐户和计费。你还需要在 GCE 上启用 API。

1.  `kube-apiserver`、`etcd`、`kube-scheduler`、`kube-controller-manager` 和 `cloud-controller-manager`。

1.  `kops`、`kubespray`、`kubeadm` 和 `bootkube`。

1.  `kubeadm`。

# 第二章：通过核心 Kubernetes 构建基础

1.  HTTP、TCP 和应用程序级健康检查

1.  ReplicaSet

1.  生态系统、接口、治理、应用程序和核心

1.  Calico 和 Flannel

1.  `rkt`、`kata`、`frakti`、`containerd` 和 `runv`

1.  集群控制平面、集群状态和集群节点

1.  基于等式的选择器

# 第三章：与网络、负载均衡器和 Ingress 配合工作

1.  通信是由 Pod 之间的通信进行管理，而不是容器之间的通信。Pod 与服务之间的通信由服务对象提供。K8s 不使用 NAT 来进行容器间的通信。

1.  网络地址转换

1.  使用覆盖网络的 CNI 插件，或使用桥接和主机本地特性的 `kubenet` 插件。

1.  Canal、Calico、Flannel 和 Kube-router。

1.  Pods。

1.  用户空间、iptables 和 ipvs。

1.  虚拟 IP、服务代理和多端口。

1.  规范。

1.  GCE、nginx、Kong、Traefik 和 HAProxy。

1.  使用命名空间、RBAC、容器权限、入口规则以及明确的网络管理。

# 第四章：实施可靠的容器原生应用程序

1.  Kubernetes 部署的四种用例如下：

    +   发布 ReplicaSet

    +   更新一组 Pod 的状态

    +   回滚到部署的早期版本

    +   扩展以适应集群负载

1.  选择器。

1.  记录标志，`--record`。

1.  `ReplicationControllers`。

1.  水平 Pod 自动伸缩。

1.  定时任务。

1.  DaemonSet 简单地定义了一个 Pod，在集群中的每个节点或这些节点的指定子集上运行。这对于许多与生产相关的活动非常有用，例如监控和日志代理、安全代理以及文件系统守护进程。

# 第五章：探索 Kubernetes 存储概念

1.  持久化的临时磁盘、云卷、`emptyDir` 和 `nfs`

1.  `emptydir`

1.  AWS 中的 EBS 卷和 Azure 上的磁盘存储

1.  不同的应用性能或耐用性要求。

1.  绑定、使用、回收和扩展。

1.  持久化卷声明。

# 第六章：应用更新、渐进式发布和自动伸缩

1.  `kubectl scale --replicas`

1.  `rolling-update`

1.  ClientIP

1.  水平 Pod 自动伸缩

1.  最小值和最大值的 CPU 和内存使用设置

1.  Helm

1.  一个图表

# 第七章：为持续集成和交付设计

1.  Node.js

1.  Jenkins

1.  Helm 图表

1.  持久化卷

1.  安装 Jenkins 插件

1.  一个 ReplicationController

1.  npm

# 第八章：监控和日志记录

1.  cAdvisor 和 Heapster。

1.  Kube-system。

1.  Grafana。

1.  一个收集器。

1.  Stackdriver。

1.  使用 Prometheus 的好理由如下：

    +   **操作简单**：它被设计为使用本地存储作为可靠性的单独服务器运行。

    +   **它很精确**：您可以使用类似于 JQL、DDL、DCL 或 SQL 查询的查询语言来定义警报，并提供状态的多维视图。

    +   **许多库**：您可以使用超过十种语言和众多客户端库来审视您的服务和软件。

    +   **高效**：使用在内存和磁盘上存储的高效、自定义格式，您可以通过分片和联合轻松扩展，从而创建一个强大的平台，用于发出可以构建强大数据模型和特设表、图表和警报的强大查询。

# 第十章：设计高可用性和可扩展性

1.  可用性、响应性和耐久性。

1.  正常运行时间是衡量给定系统、应用程序、网络或其他逻辑和物理对象已*上线*并可供适当终端用户使用的时间。

1.  可用性的五个 9

1.  意味着它以*优雅的方式*失败。

1.  **Google Kubernetes Engine** (**GKE**)。

1.  一组主节点，具有 Kubernetes 控制平面和 collocated 的 `etcd` 服务器。

1.  工作负载 API。

# 第十一章：Kubernetes SIGs、孵化项目和 CNCF

1.  Kubernetes 和 Prometheus。

1.  Linkerd、rkt、CNI、TUF、Vitess、CoreDNS、Jaeger、Envoy。

1.  Spiffe、Spire、Open Policy Agent、Telepresence、Harbor、TiKV、Cortex 和 Buildpacks。查看更多信息：[`www.cncf.io/sandbox-projects/`](https://www.cncf.io/sandbox-projects/)。

1.  委员会旨在定义元标准并解决社区范围的问题。

1.  这是理解 Kubernetes 核心概念和内部工作原理的绝佳方式。这也是与其他有动力、聪明的人们相遇的有趣方式。最后，Kubernetes 本质上是一个社区项目，依赖于其成员和用户的贡献。

1.  SSH 密钥和 SSL 连通性。

# 第十二章：集群联邦和多租户

1.  使用联合，我们可以在本地和一个或多个公共云提供商上运行多个 Kubernetes 集群，并管理利用我们组织资源的整套应用程序。

1.  联合允许您增加 Kubernetes 集群的可用性和租户能力。

1.  跨集群资源同步和多集群服务发现。

1.  Kubefed。

1.  Federation-controller-manager 和 federation-apiserver。

1.  HPA 将以类似于常规 HPA 的方式进行操作，具有相同的功能和相同的基于 API 的兼容性——只是，通过联合，Pod 的管理将穿越您的集群。

1.  部署、副本集、事件、配置映射、守护集、入口、命名空间、秘密和服务。

# 第十三章：集群认证、授权和容器安全

1.  容器仓库或注册表

1.  以下任意三个：Node、ABAC、RBAC、Webhook

1.  特权

1.  `kubectl get secrets`

# 第十四章：Kubernetes 强化

1.  数据加密、机密、服务发现、合规性、RBAC、系统事件跟踪和趋势偏差警报

1.  Stackdriver、Sysdig、Datadog 和 Sensu。

1.  Terraform 或 CloudFormation，以及 Ansible、Chef 或 Puppet

1.  最小权限原则

1.  CPU、内存和限制

1.  最大值和最小值

1.  传输层安全性（TLS）

# 第十五章：Kubernetes 基础设施管理

1.  `kubeadm-dind`、Minikube 和基于 LXD 的 Ubuntu。

1.  Google Kubernetes Engine、Amazon Elastic Container Services、Azure Kubernetes Service 和 Stackpoint.io。

1.  日志记录、身份验证、授权和 Linux 系统参数。

1.  升级每个主要 CSP 托管的 Kubernetes 集群的命令如下：

```
az aks upgrade --name <CLUSTER_NAME> --resource-group <RESOURCE_GROUP> --kubernetes-version <VERSION> 

gcloud container clusters upgrade <CLUSTER_NAME>
```

1.  Google Compute Platform，使用 EKS

1.  `kubectl drain <node>`

1.  `kubectl uncordon <node>`
