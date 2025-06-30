

# 第二十章：故障排除常见问题

在本书中，我们创建了集群并添加了控制器、附加组件、工作负载等，但并不是一切都按计划进行。虽然 K8s 在易用性方面做得很好，但你可能会在遵循本书或日常工作中遇到至少一个问题。

理解如何进行故障排除以及使用哪些工具来识别根本原因并希望解决它们，是使用或运行**弹性 Kubernetes 服务**（**EKS**）的重要组成部分。在本章中，我们将介绍与 EKS 一起使用的常见技术和工具，以及在使用 Amazon EKS 时可能遇到的常见错误。

在本章中，我们将讨论开始使用 Amazon EKS 时的一些常见问题，逐步介绍细节，并学习如何进行故障排除，例如以下内容：

+   常见的 K8s 工具/技术用于故障排除 EKS

+   常见的集群访问问题

+   常见节点/计算问题

+   常见的 Pod 网络问题

+   常见工作负载问题

让我们开始吧！

# 技术要求

读者应对 YAML、AWS IAM 和 EKS 架构有一定了解。在开始本章之前，请确保以下内容：

+   你可以访问你的 EKS 集群 API 端点的网络连接

+   AWS CLI、Docker 和 `kubectl` 二进制文件已安装在你的工作站上，并且具有管理员访问权限

# 常见的 K8s 工具/技术用于故障排除 EKS

任何故障排除过程都始于理解问题，并区分症状和根本原因。症状常常会被误认为是根本原因，因此故障排除过程通常是反复的，伴随着不断的测试和观察，专注于实际问题而忽略症状和误报。

一旦理解了问题，接下来就是理解如何缓解、解决或忽略它。并不是所有问题都能立即解决，因此你可能需要一个策略暂时绕过它们。

最终阶段将是问题的解决/修复。这可能需要更新你的集群、应用代码，或者两者都需要更新，具体取决于问题的性质，你管理的集群数量以及对用户/客户的影响，这可能是一个相当复杂的过程。

通常，当我试图理解问题时，我使用以下检查清单：

+   *有什么变化？* – 大多数支持工程师的座右铭；是否进行了新的部署？集群或附加组件升级了？AWS/本地基础设施发生了变化吗？接下来我们将讨论工具，但记录任何变化（至少在生产环境中）是一个良好的实践，可以帮助工程师确定在这三个领域中发生了什么变化。

+   *调查影响* – 在这里，你需要观察症状。问题表现为何？是处理速度变慢还是完全失败？问题的范围是什么？仅限于某个命名空间？某个集群？某个 VPC？结合对变化的理解，这通常可以帮助工程师将问题缩小到根本原因，但可能需要你通过多个假设进行迭代。是集群中新添加的节点类型导致的吗？不，是节点上的新部署！或者，是部署中的应用程序库导致的！ |

+   *规划修复或缓解措施* – 你能轻松修复问题且不会产生影响，还是需要规划变更？制定计划并与团队、应用所有者以及任何客户代表讨论。如果你能够在非生产环境中复现问题和修复过程，那么在进行任何生产变更之前，应该这么做，这有助于整体计划的执行。

现在我们已经基本理解了你通常会经历的故障排除步骤。接下来，让我们看看一些你可能用来进行故障排除的工具。

## 常见的 EKS 故障排除工具

以下表格列出了常见的故障排除工具；它并不是详尽无遗的，因为 K8s 生态系统庞大且有很多贡献者，但这些是我使用过的或者曾经使用过的工具。（如果你的最爱没有包括在内，请见谅！）

| **阶段** | **工具** | **描述** |
| --- | --- | --- |
| 发生了什么变化？ | AWS CloudTrail | CloudTrail 可以提供你在 AWS 账户中所做的每一个 API 调用的日志，包括 AWS EKS API 调用（不是 K8s API）、负载均衡器、IAM 等的 API 调用。 |
| AWS CloudWatch | 提供控制面板和日志，用于控制平面和（可选的）数据平面。 |
| Grafana Loki [`grafana.com/oss/loki/`](https://grafana.com/oss/loki/) | 一个开源的日志聚合工具。 |
| `kubectl diff` | `kubectl diff` 允许你比较本地清单文件与运行中的配置，查看将会发生（或已经发生）哪些变化。如果你能访问以前的清单文件或提交记录，这是一个非常有用的工具。 |
| 调查影响 | Linux 命令行工具 | 任何标准的 Linux 工具，如 `dig`、`ping`、`tcpdump` 等，都应该用于确定问题的范围和影响。 |
| kubectl | `kubectl describe`、`kubectl get events` 和 `kubectl logs` 是任何故障排除过程中的基本命令。`kubectl top` 可用于查看 Pod 和 Node 层级的统计信息（前提是已安装度量服务器） |
| Ktop: [`github.com/vladimirvivien/ktop`](https://github.com/vladimirvivien/ktop) | 一个用于集群管理的有用 CLI 工具 |
| k9s: [`github.com/derailed/k9s`](https://github.com/derailed/k9s) | 一个用于集群管理的有用 CLI 工具 |
| AWS Log Collector[`github.com/awslabs/amazon-eks-ami/tree/master/log-collector-script/linux`](https://github.com/awslabs/amazon-eks-ami/tree/master/log-collector-script/linux) | 一个 AWS 支持脚本，用于收集操作系统和 K8s 日志。 |

表 20.1 – 常见的 EKS 故障排除工具

注意

你现在还可以使用`kubectl debug`将故障排除容器注入到运行中的 Pod 中，以便在 K8s 1.25 及以上版本中进行实时调试。

除了技术工具/服务，AWS 还提供了多种支持服务，这些服务在此列出。

### AWS Premium 支持

AWS 为所有客户提供技术支持服务，并提供 365 天全天候 24/7 的服务。该服务旨在为你提供合适的工具和 AWS 专业知识的访问权限，让你可以专注于业务增长，同时优化系统性能，学习如何减少不必要的基础设施成本，并在使用每个 AWS 服务时减少故障排除的复杂性。

支持计划包括不同级别，例如*开发者*、*商业*和*企业*。根据你选择的支持计划，你可以通过电子邮件、电话甚至在线聊天与 AWS 专家联系，寻求有关技术或操作问题的咨询，使用 AWS 支持案例控制台。

注意

如果你对 AWS Premium 支持计划感兴趣，并想了解更多详细信息，可以查看他们的服务功能页面：[`aws.amazon.com/premiumsupport/plans/`](https://aws.amazon.com/premiumsupport/plans/)。

支持模型的一部分是访问知识中心，其中包含了关于所有 AWS 平台产品（不仅仅是 EKS）的大量附加信息。

### AWS 支持知识中心

AWS 支持知识中心由 AWS 维护，是一个集中资源中心，帮助你更好地学习如何修复使用 AWS 服务时的操作问题或配置问题。它列出了 AWS 支持从客户那里收到的最常见问题和请求。你可以通过访问以下链接查看更多详情：[`aws.amazon.com/premiumsupport/knowledge-center/`](https://aws.amazon.com/premiumsupport/knowledge-center/)。

你可以查看 Amazon EKS 部分，了解 Amazon EKS 中常见的问题。有些文章还包括完整的逐步教程的视频。现在我们已经看了一些常见的故障排除工具，接下来我们来看看一些典型的集群访问问题。

# 常见的集群访问问题

本节列出了访问 EKS 集群时常见的问题。接下来的几节将描述常见的症状以及如何修复它们。

## 你无法使用 kubectl 访问你的集群

通过**kubectl**与集群 API 交互，或将其作为 CI/CD 管道或工具的一部分，是使用 EKS 的关键步骤！接下来列出了一些常见问题，并附上了可能的解决方案。

注意

虽然接下来显示的错误可能指向一个特定问题，但它们也可能与这里未记录的其他问题有关。

在显示的错误信息中，shell 没有配置环境变量或`~/.aws`目录中的配置文件凭据：

```
$ kubectl get node
Unable to locate credentials. You can configure credentials by running "aws configure".
$ aws eks update-kubeconfig --name mycluster  --region eu-central-1
Unable to locate credentials. You can configure credentials by running "aws configure".
```

您还可能会看到以下错误消息，虽然不同，但意思相同——该 shell 没有将凭证配置为环境变量或 `~/.aws` 目录中的配置文件：

```
$ kubectl get node
Unable to locate credentials. You can configure credentials by running "aws configure".
```

在下文所示的错误消息中，shell 中配置的凭证没有正确的 IAM 权限或 K8s 权限：

```
$ kubectl get node
error: You must be logged in to the server (Unauthorized)
```

在此错误消息中，最可能的问题是网络连接问题，客户端对私有 EKS 集群的网络访问受限，或者存在 IP 白名单。这个错误有时会在使用没有相关访问权限的 IAM 角色时出现：

```
$ kubectl get node
Unable to connect to the server: dial tcp 3.69.90.98:443: i/o timeout
```

现在我们已经查看了一些常见的集群访问问题，让我们看看一些典型的节点/计算问题。

# 常见的节点/计算问题

本节列出了你可能会在 EKS 集群中看到的工作节点的常见问题，并提供了可能的解决方案。

## 节点/节点无法加入集群

对于这个问题，发生的变化（例如添加了新节点或更改了 IP 白名单）将决定需要修复的内容。通常，错误仅仅是节点处于 `NotReady`/`Unknown` 状态，如下所示，这有许多不同的根本原因：

```
$ kubectl get node
NAME  STATUS   ROLES    AGE    VERSION
ip-172-31-10-50.x.internal    NotReady    <none>   7d4h   v1.24.13-eks-0a21954
ip-172-31-11-89.x.internal    Unknown    <none>   7d4h   v1.24.13-eks-0a21954
```

如果你正在运行自管理节点，EC2 是否执行了 `bootstrap.sh` 脚本？对于托管节点，这将自动发生，但如果没有，您的 EC2 实例将不会自动注册。

如果您在公共集群上有 IP 白名单，是否将您的节点的公共地址或 NAT 网关添加到了白名单中？如果没有，您的节点将无法与 K8s 集群通信。

对于私有或私有/公共集群，您的工作节点安全组是否已经与集群的安全组注册？如果没有，您的节点将无法与 K8s 集群通信。

```
Dec 23 17:42:36 ... kubelet[92039]: E1223 17:42:36.551307 92039 kubelet.go:2240] node "XXXXXXXXX" not found
```

在前面从 `kubelet` 日志中显示的错误消息中，最可能的问题是 VPC DNS 问题，其中 VPC 尚未配置 DNS 主机名或解析。

```
Error: Kubernetes cluster unreachable: Get "https://XXXXXXXXXX.gr7.eu-central-1.eks.amazonaws.com/version?timeout=32s": dial tcp <API_SERVER_IP>:443: i/o timeout
```

在前面从 `kubelet` 日志中显示的错误消息中，最可能的问题是网络问题，其中安全组、NACL、NAT 网关或端点配置错误或不存在。

```
Jan 01 12:23:01 XXXXXXXXX kubelet[4445]: E1012 12:23:01.369732    4445 kubelet_node_status.go:377] Error updating node status, will retry: error getting node "XXXXXXXXX": Unauthorized
```

在前面从 `kubelet` 日志中显示的错误消息中，最可能的问题是 IAM 身份验证问题，其中节点的 IAM 角色没有被添加到 `aws-auth` 配置映射中。

```
onditions:Type   Status  LastHeartbeatTime  LastTransitionTime     Reason          Message
  MemoryPressure   False   Tue, 12 Jul 2022 03:10:33 +0000   Wed, 29 Jun 2022 13:21:17 +0000   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure     True    Tue, 12 Jul 2022 03:10:33 +0000   Wed, 06 Jul 2022 19:46:54 +0000   KubeletHasDiskPressure       kubelet has disk pressure
```

在前面从 `describe node` 命令中显示的错误消息中，最可能的问题是由于磁盘空间不足引起的磁盘问题（通常是由于日志过度控制）。现在我们已经看过一些常见的集群计算问题，让我们看看一些典型的网络问题。

# 常见的 Pod 网络问题

在前面的部分中，我们已经讨论了在 API/控制平面和工作节点环境中的网络问题。本节列出了你可能会在 EKS 集群中看到的 Pod 网络问题，并提供了可能的解决方案。

```
* connect to 172.16.24.25 port 8080 failed: Connection timed out
* Closing connection 0
curl: (7) Failed to connect to 172.16.24.25 port 8080 failed: Connection timed out
```

从 pod 日志中显示的错误消息来看，最可能的问题是工作节点（或 pod）的安全组配置问题。请检查工作节点的安全组配置，确保允许所有必要的端口、IP CIDR 范围和其他安全组（包括其自身）。如果你使用的是 NACL，确保允许外部临时端口，并且任何入口端口也被允许。

```
Error: RequestError: send request failed caused by: Post dial tcp: i/o timeout
```

从 pod 日志中显示的错误消息来看，最可能的问题是 DNS 问题。请确保`clusterDNS`和`CoreDNS`正常工作，并且 VPC 已启用 DNS 解析和主机名功能。

```
Error: RequestError: send request failed caused by: Post dial tcp 172.16.24.25:443: i/o timeout
```

从 pod 日志中显示的前述错误消息来看，最可能的问题是连接问题。

```
Failed create pod mypod: rpc error: code = Unknown desc = NetworkPlugin cni failed to set up pod network: add cmd: failed to assign an IP address to container
```

`pod describe`命令显示的前述错误消息显示 pod 停留在`ContainerCreating`状态。最可能的问题是 VPC 中没有更多可用的 IP 地址。如果没有使用前缀地址分配，EC2 实例类型可能已经用尽了它能够支持的 IP 地址数量。

```
NAME    READY     STATUS             RESTARTS    AGE
aws-node-bbwpq   0/1     CrashLoopBackOff     12         51m
aws-node-nw7v8  0/1     CrashLoopBackOff     12         51m
coredns-12     0/1     Pending         0         54m
coredns-13     0/1     Pending         0         54m
```

从`get pod`命令显示的错误消息来看，最可能的问题是 VPC 或安全组问题，这会对 pod 中的 CNI 或 DNS 操作产生连锁反应。

```
"msg"="Reconciler error" "error"="failed to build LoadBalancer configuration due to retrieval of subnets failed to resolve 2 qualified subnets.
```

从`describe deployment`命令中的错误消息来看，最可能的问题是某个子网没有被标记为内部或外部负载均衡器，因此无法被发现。现在我们已经看了一些常见的 pod 网络问题，接下来我们来看一些典型的工作负载问题。

# 常见的工作负载问题

本节列出了你可能在 EKS 集群中遇到的与 pod 和/或部署相关的常见问题，以及可能的解决方案。

```
State:  Running
Started: Sun, 16 Feb 2020 10:20:09 +0000
Last State: Terminated
Reason: OOMKilled
```

在`kubectl describe pod`命令显示的前述错误消息中，最可能的问题是连接问题或内存问题，因为`OOMKilled`意味着 pod 达到了其内存限制，因此会重启。你需要在部署或 pod 配置中增加内存设置。

```
NAME            READY  STATUS            RESTARTS   AGE
myDeployment1-123... 1/1    Running           1     17m
myDeployment1-234... 0/1    CrashLoopBackOff  2     1m
```

从`kubectl get po`命令中的错误消息来看，可能存在多个问题，无论是由于 DockerFile 错误、拉取镜像文件失败等原因。运行`kubectl logs`命令以获取更多有关错误原因的信息。

```
  Warning  FailedScheduling  22s (x14 over 13m)  default-scheduler  0/3 nodes are available: 3 Insufficient cpu.
```

从`kubectl describe po`命令显示的错误消息中，pod 处于`Pending`状态。最可能的问题是你的工作节点的 CPU 资源不足。

```
  Warning  FailedScheduling  80s (x14 over 15m)  default-scheduler  0/3 nodes are available: 3 Insufficient memory.
```

`kubectl describe po`命令的错误消息显示 pod 处于`Pending`状态，最可能的问题是工作节点的内存资源不足。

```
    State:          Waiting
      Reason:       ErrImagePull
```

从`kubectl describe po`命令显示的前述错误消息中，pod 处于`Pending`状态。最可能的问题是容器镜像不正确、不可用，或者托管在未在工作节点中配置的私有仓库中。

```
Warning  FailedScheduling  77s (x3 over 79s)  default-scheduler  0/1 nodes are available: 1 node(s) had taint {node-typee: high-memory}, that the pod didn't tolerate.
```

来自`kubectl describe po`命令的前面错误信息显示 Pod 处于 `Pending` 状态。最可能的问题是 Pod 不匹配相应的节点容忍度。本章不会涉及你在 K8s/EKS 中可能遇到的所有症状和根本原因，但希望你已经获得足够的信息，能够应对常见的问题。

在本节中，我们查看了用于故障排除 EKS 的工具和技术以及你可能遇到的常见问题。现在，我们将回顾这一章的关键学习要点。

# 总结

在这一章中，我们首先看了一种通用的故障排除方法，并介绍了一些常用工具，这些工具可以帮助我们确定发生了什么变化以及问题的范围/影响。

然后我们转向了在连接到 EKS 集群、计算节点、Pod 网络和工作负载时可能遇到的常见问题/症状，并使用**kubectl**命令来识别这些问题，并提供一些可能的解决方案。

这是最后一章，到现在为止，你应该已经掌握了构建和管理 EKS 集群以及运行在其上的工作负载所需的所有知识。

恭喜你完成了本书的学习！希望你发现它既有信息量又有实用性。

# 进一步阅读

+   调试 K8s 工具：

[`www.cncf.io/blog/2022/09/15/10-critical-kubernetes-tools-and-how-to-debug-them/`](https://www.cncf.io/blog/2022/09/15/10-critical-kubernetes-tools-and-how-to-debug-them/)

+   EKS 官方故障排除指南：

[`docs.aws.amazon.com/eks/latest/userguide/troubleshooting.html`](https://docs.aws.amazon.com/eks/latest/userguide/troubleshooting.html)
