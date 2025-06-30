# 第八章：*第八章*：将外部管理的集群导入 Rancher

在之前的章节中，我们涵盖了由 Rancher 创建的集群和托管的集群。本章将涵盖由 Rancher 导入的集群，以及在执行此操作时的要求和限制。接着，我们将深入探讨几个示例设置，最后在本章结束时探讨 Rancher 如何访问一个导入的集群。

在本章中，我们将涵盖以下主要内容：

+   什么是导入的集群？

+   要求和限制

+   构建解决方案的规则

+   如何让 Rancher 访问集群？

# 什么是导入的集群？

创建第一个 `etcd` 备份后，Rancher 无法访问它。原因是 Rancher 只能通过 kube-api 端点访问集群。Rancher 无法直接访问节点、`etcd` 或集群中更深层次的内容。

## 在我的新 Rancher 实例中，这个本地集群是什么？

Rancher 将自动导入安装 Rancher 的集群。需要注意的是，这是默认行为。在 Rancher v2.5.0 之前的版本中，曾有一个名为 `addLocal=false` 的 Helm 选项，允许你禁用 Rancher 导入本地集群。但在 Rancher v2.5.0 中，这个功能被移除，并用 `restrictedAdmin` 标志取而代之，该标志限制对本地集群的访问。同样需要注意的是，默认情况下本地集群被称为 `local`，但你可以像对 Rancher 中的任何其他集群一样重命名它。通常会将本地集群重命名为更有帮助的名称，比如 `rancher-prod` 或 `rancher-west`。在本节的其余部分，我们将称此集群为本地集群。

## 为什么本地集群是一个导入的集群？

本地集群是在 Rancher 外部构建的，并且 Rancher 不管理它。这是因为你会遇到鸡生蛋问题。如果没有 Kubernetes 集群，你如何安装 Rancher，但你需要 Rancher 来创建一个集群？因此，我们将进一步解释这个问题。在 Rancher v2.5.0 之前，你需要创建一个 RKE 集群（有关创建 RKE 集群的详细安装说明，请参考 *第四章*，*创建 RKE 和 RKE2 集群*）。但是因为这个集群由 RKE 管理，而不是由 Rancher 管理，所以本地集群需要是一个导入的集群。

## 为什么有些导入的集群特殊？

现在，随着 Rancher v2.5.0+ 和 RKE2 的发布，情况发生了变化。当 Rancher 安装在 RKE2 集群上时，你可以将 RKE2 集群导入到 Rancher 中，并允许 Rancher 接管该集群的管理。这是因为引入了一种新的工具，叫做**系统升级控制器**，它通过一组名为**计划**的**自定义资源定义**（**CRD**）来帮助 RKE2 和 k3s 集群在集群内部进行管理。该控制器允许你定义操作，例如升级集群中节点的操作系统、升级 Kubernetes 版本，甚至管理新的 k3OS 操作系统。这些设置只是 Kubernetes 对象，你可以根据需要进行修改。有关系统升级控制器的更多详情，请访问 [`github.com/rancher/system-upgrade-controller`](https://github.com/rancher/system-upgrade-controller)。

## 可以导入哪些类型的集群？

使用 Rancher，你可以导入任何通过 **云原生计算基金会**（**CNCF**）认证的 Kubernetes 集群，只要该集群遵循官方 Kubernetes 仓库中定义的标准，仓库地址为 [`github.com/kubernetes/kubernetes`](https://github.com/kubernetes/kubernetes)。这包括像 **kubernetes-the-hard-way** 这样的完全自定义集群，它是一个 100% 手动构建的 Kubernetes 集群，没有 RKE 等工具来为你处理繁重的工作。请注意，这种集群是为学习优化的，不应视为生产就绪的集群。你可以在 [`github.com/kelseyhightower/kubernetes-the-hard-way`](https://github.com/kelseyhightower/kubernetes-the-hard-way) 上找到更多关于创建这种集群类型的详细信息和步骤。除了完全自定义集群，你还可以导入使用 RKE 或 VMware 的 Tanzu Kubernetes 产品（基于 vSphere 产品）构建的集群，甚至是 **Docker Kubernetes Service**（**DKS**），这是 Docker 企业解决方案的一部分。需要注意的是，像 OpenShift 这样的 Kubernetes 发行版并非 100% CNCF 认证，虽然它可能仍然可以使用，但 Rancher 不会官方支持它。关于 OpenShift 和 Rancher 的更多详情，请访问 [`rancher.com/docs/rancher/v2.5/en/faq/`](https://rancher.com/docs/rancher/v2.5/en/faq/)。

## 为什么我要导入一个 RKE 集群，而不是在 Rancher 中创建一个？

这个问题的答案归结为控制。假设你希望对 Kubernetes 集群拥有完全的控制权，并且不希望 Rancher 为你定义集群。这包括导入一些已经不再受 Rancher 支持的遗留集群，或者从一些第三方云提供商那里导入的集群，而这些集群当前 Rancher 不支持。

## Rancher 可以对导入的集群做什么？

即使在将集群导入 Rancher 时存在一些限制，Rancher 仍然可以为集群提供价值，首个好处是为所有 Kubernetes 集群提供单一视图。我们将在下一节中介绍这些限制。即使 Rancher 的 `kubectl` 没有直接访问集群的权限。

# 要求和限制

现在我们了解了在 Rancher 中导入的集群是什么以及它在 Rancher 中的工作原理，我们将继续讨论托管集群在 Rancher 中的要求和限制，以及在选择托管集群时的设计限制和约束。

## 基本要求

在本节中，我们将介绍 Rancher 需要的 Kubernetes 集群的基本要求。这些在这里概述：

+   Rancher 需要对集群拥有完整的管理员权限，推荐权限级别为 `cluster-admin` 的默认集群角色。

+   导入的集群将需要访问 Rancher API 端点。

+   如果您正在导入 **Elastic Kubernetes Service** (**EKS**) 或 **Google Kubernetes Engine** (**GKE**) 集群，Rancher 服务器应具有服务账户和云提供商所需的权限。有关托管集群和所需权限的详细信息，请参阅前一章节。

+   Rancher 发布了每个 Rancher 发布版本支持的当前 Kubernetes 版本列表。您可以在 [`www.suse.com/suse-rancher/support-matrix/all-supported-versions/`](https://www.suse.com/suse-rancher/support-matrix/all-supported-versions/) 找到此列表。

+   `cattle-node-agent` 将使用主机的网络，需要禁用 `systemd-resolved` 和 `dnsmasq`。此外，应该在 `/etc/resolv.conf` 中配置 DNS 服务器，而不是 `127.0.0.1` 回环地址。

+   对于 k3s 和 RKE2 集群，在将集群导入 Rancher 之前，您需要在集群上安装 `system-upgrade-controller`。

+   从 Rancher v2.6.1 开始，Harvester 集群也可以导入。但是，必须使用位于 https://rancher.com/docs/rancher/v2.6/en/virtualization-admin/#feature-flag 的步骤启用此功能标志。

接下来让我们看看设计考虑事项。

## 设计限制和考虑事项

在本节中，我们将讨论将要导入 Rancher 的集群的限制和注意事项。这些在这里概述：

+   一个集群一次只能导入一个 Rancher 安装。

+   可以在 Rancher 安装之间迁移集群，但是在移动后需要重新创建项目和权限。

+   如果您在为 Rancher API 端点提供访问的过程中使用 **HyperText Transfer Protocol/Secure** (**HTTP/S**) 代理，则需要添加额外的代理环境变量详细信息，可以在 [`rancher.com/docs/rancher/v2.5/en/cluster-provisioning/registered-clusters/`](https://rancher.com/docs/rancher/v2.5/en/cluster-provisioning/registered-clusters/) 找到。

+   如果集群有 `cattle-cluster-agent` 和 `cattle-node-agent`，则需要一个不受限制的策略，因为节点代理将挂载主机文件系统，包括根文件系统。集群代理将需要访问集群中的所有对象。

+   如果集群有 `cattle-system` 命名空间，建议将其加入*忽略*列表。这是因为代理不会设置限制和请求，部署后所做的任何更改将被覆盖。更多详细信息，请参见 OPA Gatekeeper 文档：[`github.com/open-policy-agent/gatekeeper`](https://github.com/open-policy-agent/gatekeeper) 和 [`www.openpolicyagent.org/docs/latest/kubernetes-tutorial/`](https://www.openpolicyagent.org/docs/latest/kubernetes-tutorial/)。

+   需要注意的是，从 Rancher v2.6.2 开始，k3s 和 RKE2 集群仍处于技术预览阶段，因此它们可能缺少某些功能并存在严重的 bug。

+   `/etc/rancher/rke2/config.yaml` 文件中定义的 RKE2 配置设置无法被 Rancher 覆盖，因此你应尽量减少对该文件的自定义修改。

+   对于使用外部管理数据库（如 MySQL、Postgres 或非嵌入式 `etcd` 数据库）的导入 k3s 集群，Rancher 和 k3s 将无法访问和使用所需的工具进行数据库备份。此类任务需要在外部进行管理。

+   如果集群已导入到 Rancher 中，然后又被重新导入到另一个 Rancher 实例中，通过 Rancher 目录部署的任何应用程序都将被导入，并需要重新部署或直接使用 Helm 管理。

+   如果导入的集群使用 Rancher Monitoring v1，你需要在重新启用 Rancher UI 中的监控之前，卸载并清理所有监控命名空间和 CRD。

假设你在集群导入 Rancher 之前已经部署了一个 fleet。在导入 Rancher v2.6.0 之前，应该先卸载该 fleet，因为 fleet 已经集成在 Rancher 中，两个不同的 fleet 代理将相互冲突。

到目前为止，我们已经了解了将外部管理的集群导入 Rancher 的所有需求和限制。我们将在下一节中使用这些内容开始创建我们的集群设计。

# 架构解决方案的规则

在本节中，我们将介绍一些标准设计方案及其优缺点。需要注意的是，每个环境都是独特的，需要根据最佳性能和体验进行调优。还需要指出的是，所有**中央处理单元**（**CPU**）、内存和存储的大小都是推荐的起始点，可能需要根据你的工作负载和部署流程进行增减。此外，我们将涵盖外部管理的 RKE 集群和 Kubernetes The Hard Way 的设计，但你应该能够将核心概念应用到其他基础设施提供商。

在设计解决方案之前，你应该能够回答以下问题：

+   是否会有多个环境共享同一个集群？

+   生产和非生产工作负载是否会在同一个集群上运行？

+   该集群需要什么级别的可用性？

+   该集群是否会跨多个数据中心运行，形成一个城市集群环境？

+   集群中节点之间将有多少延迟？

+   集群中将托管多少个 Pod？

+   您将在集群中部署的 Pod 的平均大小和最大大小是多少？

+   您的某些应用程序是否需要**图形处理单元**（**GPU**）支持？

+   您需要为您的应用程序提供存储吗？

+   如果您需要存储，您只需要**只读写一次**（**RWO**），还是需要**读写多次**（**RWX**）？

## 外部管理的 RKE

在这种类型的集群中，您使用 RKE 工具以及`cluster.yaml`文件手动创建和更新 Kubernetes 集群。从本质上讲，Rancher 启动的集群和外部管理的 RKE 集群都使用 RKE 工具，区别在于谁负责集群及其配置文件的管理。请注意，如果这些文件丢失，将很难继续管理集群，您将需要恢复它们。

**优点**在此列出：

+   控制权归您所有，因为您在集群上手动运行 RKE。您可以控制节点的添加与删除。

+   集群不再依赖于 Rancher 服务器，因此，如果您想从环境中移除 Rancher，可以按照[`rancher.com/docs/rancher/v2.5/en/faq/removing-rancher/`](https://rancher.com/docs/rancher/v2.5/en/faq/removing-rancher/)中的步骤，将 Rancher 从环境中移除，而无需重新构建集群。

**缺点**在此列出：

+   您需要负责保持 RKE 二进制文件的最新状态，并确保 RKE 版本与您的集群匹配。如果不遵循此要求，RKE 可能会进行意外的升级或降级，这可能会破坏您的集群。

+   您需要负责维护`cluster.yaml`文件，随着节点的添加和移除进行更新。

+   您必须拥有一台可以通过**安全外壳协议**（**SSH**）访问集群中所有节点的服务器或工作站。

+   在任何集群创建或更新事件后，您需要负责保护`cluster.rkestate`文件，它包含集群的密钥和证书。没有这个文件，RKE 将无法正常工作。请注意，您可以通过[`github.com/rancherlabs/support-tools/pull/63`](https://github.com/rancherlabs/support-tools/pull/63)中的步骤，从运行中的集群恢复该文件。

## Kubernetes The Hard Way

该集群设计用于希望学习 Kubernetes 且不想自动化集群创建和维护的人员。通常在实验环境中使用这种配置，您可能需要运行一些非常非标准的配置。有关此类集群的详细信息和步骤，请参见[`github.com/kelseyhightower/kubernetes-the-hard-way`](https://github.com/kelseyhightower/kubernetes-the-hard-way)。

**优点**在此列出：

+   **知识**—由于 Kubernetes The Hard Way 是为了学习而优化的，你将亲自处理集群创建和管理过程中的每个步骤。这意味着没有*幕后黑手*为你管理集群。

+   **定制化**—因为你需要部署每个组件，所以你拥有完全的控制权来选择版本、设置，甚至可以将标准组件替换为定制解决方案。

+   运行最新版本的能力，因为大多数 Kubernetes 发行版在上游 Kubernetes 发布新版本后，会有一段时间的滞后才能提供给最终用户。这是因为需要进行测试、代码更改、发布计划等原因。

**缺点**在这里列出：

+   Kubernetes The Hard Way 并非为生产环境设计，且社区支持极为有限。

+   集群的维护非常困难，因为像 RKE 这样的发行版提供了许多维护服务，例如自动化`etcd`备份、证书创建和轮换。而在 Kubernetes The Hard Way 中，你需要自己编写脚本来处理这些任务。

+   **版本匹配**—在 Kubernetes The Hard Way 中，你需要选择每个组件的版本，这需要大量的测试和验证。而发行版会为你处理好这些问题。

## k3s 集群

该集群是一个完全认证的 Kubernetes 发行版，专为边缘计算和远程位置设计。其核心卖点是支持 ARM64 和 ARMv7 架构，使得 k3s 能够在树莓派或其他低功耗服务器上运行。有关 k3s 的详细信息可以查看[`rancher.com/docs/k3s/latest/en/`](https://rancher.com/docs/k3s/latest/en/)和[`k3s.io/`](https://k3s.io/)。我们也在前面的章节中更加深入和详细地介绍了 k3s。

**优点**在这里列出：

+   截至目前，k3s 是唯一支持在 ARM64 和 ARMv7 节点上运行的 Rancher 发行版。未来 RKE2 应该会添加对 ARM64 的全面支持。

    重要提示

    官方支持正在通过[`github.com/rancher/rke2/issues/1946`](https://github.com/rancher/rke2/issues/1946)进行跟踪。

+   k3s 被设计成在集群创建时非常快速。因此，你可以创建一个 k3s 集群，将其导入到 Rancher 中，运行一些测试，然后删除集群，这一切都可以作为一个流水线的一部分，用于测试集群软件，例如特殊控制器和其他集群级别的软件。

+   假设你在一个网络连接较差的远程位置部署了 k3s。你仍然可以将其导入 Rancher，提供统一的管理面板和其他相关功能，但如果 k3s 集群与 Rancher 之间的连接中断，集群将继续运行，应用程序也不会察觉到任何变化。

**缺点**在这里列出：

+   导入的 k3s 集群在 Rancher v2.6.2 中仍处于技术预览阶段，仍然缺少节点创建等功能。

+   k3s 集群仍然需要在 Rancher 外部先行构建，然后再导入到 Rancher，这需要额外的工作和脚本支持。

## RKE2 集群

这种集群是 Rancher 中 Kubernetes 集群的未来，因为 RKE2 从零开始设计，旨在将集群的管理从外部迁移到内部。具体来说，RKE 使用外部工具（RKE 二进制文件），你需要负责配置文件和状态文件，这会带来相当大的管理开销。Rancher 最初通过让 Rancher 服务器接管这一过程来解决这个问题，但这个方法在扩展性上存在问题。如果你有成千上万的集群由 Rancher 管理，仅仅保持所有连接的正常和健康就是一场噩梦，更不用说在每个集群发生变化时运行 `rke up`。RKE2 使用为 k3s 创建的引导过程将这一任务转移到集群本身。在前几章中，我们深入探讨了 RKE2。

**优点**如下：

+   从 Rancher v2.6.0 开始，你可以在 Rancher 外部创建一个 RKE2 集群，导入它，并让 Rancher 接管集群的管理。

+   通过导入一个 RKE2 集群，你不再需要 `cattle-node-agent`，因为 `rke2-agent` 已经取代了这个功能，而且该代理不需要 Rancher 才能工作。

+   一个 RKE2 集群可以导入到 Rancher 中并删除，而不会影响集群本身。

**缺点**如下：

+   RKE2 仍处于技术预览阶段，支持和功能有限。

+   在将集群导入 Rancher 之前，你仍然需要引导集群中的第一个节点，这需要额外的工具或脚本支持。

+   RKE2 不支持 k3OS 操作系统，但通过 Harvester，这个功能目前正在开发中。你可以在[`github.com/harvester/harvester/issues/581`](https://github.com/harvester/harvester/issues/581)查看更多详情。

+   截至目前，导入的 RKE2 集群已正式支持 Windows 节点。你可以在[`docs.rke2.io/install/quickstart/#windows-agent-worker-node-installation`](https://docs.rke2.io/install/quickstart/#windows-agent-worker-node-installation)找到将 Windows 工作节点加入 RKE2 集群的文档化流程。如果你要将此集群导入到 Rancher 中，你必须在集群中拥有一个 Linux 节点来支持 Cattle 代理。

到此为止，我们应该已经确定了设计，并准备好部署集群并将其导入 Rancher。

# Rancher 如何访问一个集群？

在我们深入探讨 Rancher 如何访问导入的集群之前，我们首先需要了解将集群导入 Rancher 的步骤。这个过程相对简单，你只需要在集群上运行 `kubectl` 命令。此命令将会在集群上部署所需的代理。

导入的集群访问下游集群的方式与 Rancher 对其他类型集群的访问方式相同。`cattle-cluster-agent` 进程在下游集群的一个工作节点上运行。然后，代理连接 Kubernetes API 端点，默认情况下使用内部服务记录，但也可以通过 `KUBERNETES_SERVICE_HOST` 和 `KUBERNETES_PORT` 环境变量进行覆盖。不过，通常情况下不需要这样做。集群代理将使用 Cattle 服务账户中定义的凭证连接到 kube-api 端点。如果代理连接失败，它将退出，Pod 会重试直到连接成功。然而，值得注意的是，在这个过程中，假设节点仍处于 `Ready` 状态，Pods 不会被重新调度到其他节点。这可能会导致“僵尸节点”问题，这些节点未正确报告其节点状态。例如，如果节点上的 DNS 出现问题，集群代理将无法建立连接，但节点可能仍处于 `Ready` 状态。需要注意的是，在 Rancher v2.6.0 版本中，两个集群代理具有节点调度规则，确保它们运行在不同的工作节点上。

一旦集群代理成功连接到 Kubernetes API 端点，代理将连接到 Rancher API 端点。代理通过首先向`https://RancherServer/ping`发送 HTTPS 请求，收到 `200 OK` 响应以及 `pong` 输出来完成此操作。这样做是为了验证 Rancher 服务器是否正常运行、健康并准备好接受连接。作为建立此连接的一部分，代理要求连接使用 HTTPS 协议并且具有有效的证书，如果您使用的是来自已知根证书机构的公共签名证书，这没有问题。然而，如果使用自签名证书或内部签署的证书，或者代理的基础镜像不信任该证书机构，便会出现问题。在这种情况下，连接将会失败。为了解决这个问题，代理使用名为 `CATTLE_CA_CHECKSUM` 的环境变量，比较该变量的值是否相同。然后，代理会将该根证书添加到其受信任的根证书列表中，从而允许连接过程继续。如果此检查失败，代理将休眠 60 秒并重新尝试。这就是为什么在更改 Rancher 服务器的根证书之前，务必先更新代理的原因。

注意

如果您想更改 Rancher 服务器的根证书，请按照文档中的过程操作，链接在[`github.com/rancherlabs/support-tools/tree/master/cluster-agent-tool`](https://github.com/rancherlabs/support-tools/tree/master/cluster-agent-tool)。该脚本将使用更新后的值重新部署代理。

一旦代理成功连接到 Rancher，代理将把集群令牌发送给 Rancher，使用该令牌将代理与其集群匹配并处理认证。此时，代理将建立一个 WebSocket 连接到 Rancher，并使用此连接绑定到 Rancher leader pod 内的一个随机回环端口。然后，代理将通过发送探测请求来打开该连接，以防止连接超时。这个连接不应该断开，但如果断开，代理将自动尝试重新连接，并持续重试直到成功。Rancher 服务器随后使用回环端口连接到下游集群。

# 总结

在本章中，我们学习了导入集群及其工作原理，包括代理在导入集群上与其他集群的工作方式不同。我们了解了此类型集群的局限性以及为何可能需要这些局限性。接着，我们讨论了每种解决方案的优缺点。最后，我们详细讲解了创建每种类型集群的步骤。我们以介绍 Rancher 如何提供对导入集群的访问作为本章的结束。

下一章将介绍如何在 Rancher 中管理集群配置，随着时间的推移并按规模扩展。
