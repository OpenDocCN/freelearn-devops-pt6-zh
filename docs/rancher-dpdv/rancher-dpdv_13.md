# *第九章*：集群配置备份与恢复

前几章讲解了如何将外部管理的集群导入到 Rancher 中。本章将重点介绍如何在 Rancher 中管理 RKE1 和 RKE2 集群，尤其是在集群的备份和恢复方面。我们将包括设置备份的一些最佳实践。然后，我们将演示一个 etcd 恢复，最后讨论 etcd 备份的限制。

本章将涵盖以下主要内容：

+   什么是 etcd 备份？

+   为什么我需要备份我的 etcd？

+   etcd 备份是如何工作的？

+   etcd 恢复是如何工作的？

+   何时需要进行 etcd 恢复？

+   etcd 备份无法保护哪些内容？

+   如何配置 etcd 备份？

+   如何进行 etcd 备份？

+   如何从 etcd 备份中恢复？

+   设置实验环境以测试常见故障场景

# 什么是 etcd 备份？

正如我们在*第二章*《Rancher 和 Kubernetes 高级架构》中所讲的，etcd 是 Kubernetes 的数据库，用于存储集群的配置。RKE1 和 RKE2 都使用 etcd 来完成这一角色，但其他发行版，如 k3s，可能使用不同的数据库，如 MySQL、PostgreSQL 或 SQLite。对于本章内容，我们将仅关注 etcd。在 Kubernetes 中，所有组件都设计为无状态的，并且不在本地存储任何数据。唯一的例外是 etcd，它的唯一任务就是为集群存储持久化数据。这包括集群的所有设置，以及所有部署、Secrets 和 ConfigMap 的定义。这意味着如果 etcd 集群丢失，你将丢失整个集群，这也是为什么从可用性角度来看，保护 etcd 集群至关重要的原因，我们在*第二章*《Rancher 和 Kubernetes 高级架构》和*第四章*《创建 RKE 和 RKE2 集群》中都有提到。

# 为什么我需要备份我的 etcd？

当人们开始 Kubernetes 之旅时，常见的一个问题是：“为什么我需要备份 etcd？”接下来常问的问题是：“如果发生任何问题，我不能重新部署集群吗？”我回答这个问题的方法是：“是的，在理想的情况下，你应该能够通过重新部署一切来从零开始重建集群。但我们并不生活在完美的世界里。如果你丢失了集群，在现实世界中重新部署 100% 的应用程序是非常困难的。”

我经常举的例子是，假设现在是星期五的深夜，你刚刚进行了一次 Kubernetes 升级，但现在一切都失败了。应用程序崩溃，你无法找到解决方法来继续升级。如果你有一次升级前的 etcd 备份，使用 Rancher 只需要几次点击，就能将集群恢复到升级前的状态，而不是花费数小时启动一个新集群，然后再花几个小时在集群上部署所有核心服务，比如监控、日志和存储。更何况，谁知道你能多快地重新部署所有应用程序，前提是启动过程已经完备记录，或者仍然有效。

强烈建议无论在什么环境下，包括开发和测试环境，都进行 etcd 备份，因为它为你提供了更多选择。我一直遵循的原则是 *没人因为备份太多而被解雇*。值得注意的是，在 Rancher 部署的集群中，备份默认是开启的。这是因为 etcd 备份通常只占用几百兆存储，在灾难发生时非常有价值。

常见的问题之一是：“如果我有虚拟机快照，我还需要 etcd 备份吗？”虽然有额外的备份总是好的，但问题在于从快照恢复后如何恢复 etcd。问题是，所有节点必须在快照时处于同步状态，恢复才会成功。如果这是你唯一的选择，你仍然可以从虚拟机快照中恢复 etcd，但你需要恢复一个 etcd 节点，清理其他 etcd 节点，并从恢复的节点重新同步 etcd 数据。你可以在 [`github.com/rancherlabs/support-tools/tree/master/etcd-tools`](https://github.com/rancherlabs/support-tools/tree/master/etcd-tools) 找到此过程和脚本。需要知道的是，这个过程可能非常困难且耗时，并且它不是官方支持的解决方案。

# etcd 备份是如何工作的？

在本节中，我们将了解 etcd 备份如何在 RKE 和 RKE2 集群中工作。

## RKE 集群

对于 RKE 集群，单次快照的过程由 RKE 二进制文件控制，定期快照则由一个由 RKE 部署的独立容器 `etcd-rolling-snapshots` 管理。两个过程遵循相同的基本步骤，第一步是依次进入集群中的每个 etcd 节点，并启动一个名为 `etcd-snapshot-once` 或 `etcd-rolling-snapshots` 的容器，具体取决于备份类型。这个容器将承担大部分的工作。需要注意的是，这是一个 Kubernetes 外部的 Docker 容器，且对这个容器的定制非常有限。一旦容器启动，它会运行一个名为 `rke-etcd-backup` 的工具，这是 Rancher 的 rke-tools 之一，可以在 [`github.com/rancher/rke-tools/`](https://github.com/rancher/rke-tools/) 找到。

该工具主要是一个实用脚本，用于查找证书文件，并在此时执行 `etcdctl snapshot save` 命令。此命令将整个 etcd 数据库导出为一个单一文件。需要特别注意的是，这是一个完整备份，而不是增量或差异备份。此外，etcd 不像其他数据库那样有日志翻译功能，因此快照文件包含了整个数据库作为一个单一文件。

一旦数据库完成备份，RKE 会备份一些额外的文件，以便于集群恢复。这包括从 `kube-system` 命名空间中的 `configmap full-cluster-state` 提取 `cluster.rkestate` 文件。在 v1.0.0 之前的 RKE 版本中，RKE 会备份 `/etc/kubernetes/ssl/` 证书文件夹，但现在不再需要，因为 `rkestate` 文件已经将所有证书及其私钥包含在 JSON 文件中。所有文件创建完毕后，rke-tools 会将所有文件压缩成一个备份文件，并存储在主机的 `/opt/rke/etcd-snapshots/` 目录下。然后，如果已配置 S3 备份，rke-tools 会将备份文件上传到 S3 存储桶。

重要注意事项

默认情况下，rke-tools 会保留本地备份副本，以防万一。

最后，rke-tools 会清理备份。此操作是通过计算所有计划备份文件的总数来完成的。如果该数量超过了保留设置（默认设置为 6），它将开始删除最旧的备份，直到符合保留设置。需要特别注意的是，任何一次性快照将不会被计入和删除。因此，这些备份通常会保留在节点上，直到手动清理。此过程完成后，RKE 将开始处理集群中的下一个 etcd 节点。

注意

存在一个已知的设计特点，对于 S3 备份，集群中的所有 etcd 节点都会进行备份，并且每个节点将把其备份文件上传到 S3 存储桶，且文件名称相同。这意味着在备份过程中，文件会被多次覆盖。

## RKE2/k3s 集群

对于 RKE2 和 k3s 集群，它们共享一个 etcd 备份和恢复的代码。与 RKE 的主要区别在于，etcd 备份过程直接集成在 `rke2-server` 二进制文件中，而不是作为单独的容器存在。对于 RKE2/k3s，etcd 备份默认启用，并通过服务器选项进行配置，我们将在本章后面讲解。另一个主要区别是，在 RKE2 中，备份的唯一文件就是 etcd 快照文件，因为 RKE2 不需要像 RKE 那样的 `rkestate` 文件。对于 RKE2，集群状态存储在引导密钥中，并直接存储在 etcd 数据库中。需要特别注意的是，引导密钥使用 AES SHA1 加密算法，以服务器令牌作为加密密钥进行加密，而不是存储在 etcd 中。您需要在备份过程之外存储并保护令牌。如果丢失了令牌，将无法在不破解加密的情况下恢复集群。

另一个区别是备份的配置方式，因为每个主节点都是独立配置的，这意味着你可以为每个节点设置不同的备份计划。这也包括定期快照的运行方式，实际上它使用了 cronjob 格式，允许你在设定的时间强制执行备份，例如，每晚午夜或每小时的整点。为了解决 RKE 存在的 S3 覆盖问题，RKE2 在备份文件名中使用了节点的主机名。这意味着集群中的每个节点仍会进行 etcd 备份并将其上传到 S3 存储桶中，但不会被覆盖。因此，你的 S3 存储桶中会有重复的备份文件，这意味着如果你在 RKE2 集群中有三个主节点，你将会在 S3 中有三个 etcd 备份文件的副本。再次强调，在 etcd 中，备份通常很小，因此增加的存储通常只是背景噪音。

# etcd 恢复是如何工作的？

接下来，让我们看看在不同集群中，etcd 恢复是如何工作的。

## RKE 集群

对于 RKE 集群，恢复 etcd 的过程是使用`rke etcd snapshot-restore`命令完成的，该命令使用与 RKE 二进制文件用于备份的相同独立容器（rke-tools）。主要区别在于，所有 etcd 节点都需要相同的备份文件来进行恢复。这意味着，当你将快照名称提供给 RKE 二进制文件时，集群中的所有节点必须拥有该文件的副本。

恢复过程的第一步是在每个节点上创建文件的 MD5 哈希值，并比较哈希值以验证所有节点是否一致。如果此检查失败，恢复将停止，并要求用户手动在节点之间复制备份文件。可以向 RKE `restore` 命令添加一个名为`--skip-hash-check=true`的标志，但这是一个安全功能，除非你知道自己在做什么，否则不应禁用。如果你使用的是 S3 选项，RKE 会在每个节点上从 S3 存储桶下载备份文件，然后才会运行此过程，此时哈希验证过程是相同的。

一旦备份文件经过验证，RKE 将拆除 etcd 集群，这意味着 RKE 会停止所有节点上的 etcd 和控制平面容器，此时 RKE 将启动一个名为`etcd-restore`的独立容器，该容器将在每个节点上恢复 etcd 数据目录。这就是为什么所有节点必须拥有相同的快照文件的原因。 一旦所有节点上的恢复容器成功完成，RKE 将运行标准的 RKE up 过程来重新构建 etcd 和控制平面，包括创建新的 etcd 集群，然后启动控制平面服务。

最后，通过更新工作节点来结束这个过程。在此任务期间，集群将会停机大约 5 到 10 分钟，直到恢复过程完成。大多数应用程序 Pods 应该会继续运行而不受影响，前提是它们不依赖于 kube-api 服务。例如，ingress-nginx-controller 在恢复过程中将保持运行，但配置会出现暂停。

## RKE2/k3s 集群

恢复过程在 RKE2/k3s 中与 RKE 非常不同，因为在 RKE 中，集群中的一个主服务器将作为新的引导节点来重置集群。这个过程会停止集群中所有主节点上的 `rke2-server` 服务。新的引导节点 `rke2` 将运行以下命令：

```
rke2 server --cluster-reset --cluster-reset-restore-path=<PATH-TO-SNAPSHOT> 
```

这将创建一个新的 etcd 集群 ID，并将 etcd 快照恢复到新的单节点 etcd 集群中。此时，`rke2-server` 将能够启动。其余的 rke2 主节点需要清理并重新加入集群，作为*新的*节点。一旦所有主节点都恢复并健康运行，工作节点应该会自动重新加入，但这可能较慢且不可靠，因此在恢复后重启 rke2-agents 是标准做法。

重要提示

需要注意的是，恢复过程将使整个集群回滚。这包括任何 Deployments、ConfigMaps、Secrets 等内容。因此，如果你正在恢复以解决应用程序问题，你需要重新应用集群中其他应用程序的任何更改。

# 你什么时候需要进行 etcd 恢复？

当然，以下问题总是会出现：“我什么时候应该进行 etcd 恢复？”和“etcd 恢复仅仅是应急情况吗？”一般来说，etcd 恢复主要用于灾难恢复和回滚 Kubernetes 升级。例如，你不小心删除了集群中大部分或所有的 etcd 节点，或遇到基础设施问题，如停电或存储故障。如果集群能够自行恢复，从事件发生前的最后备份恢复等会是恢复集群服务的最快方式。

进行恢复的另一个主要原因是 Kubernetes 升级失败。和 RKE 一样，没有办法在不从升级前的 etcd 备份恢复集群的情况下回退集群。这就是为什么我们总是建议在升级前拍摄快照。需要注意的是，RKE 二进制文件允许你设置旧的 Kubernetes 版本，并尝试将该版本推送到集群。这一过程通常会破坏集群，并且没有官方支持。在这两种情况下，集群会停机或处于失败状态，我们的目标是尽快恢复服务。

当然，下一个问题是，“什么时候我不应该执行 etcd 恢复？”答案是，你不应该通过恢复来回滚失败的应用程序更改。例如，如果一个应用程序团队推送了一个更改，导致应用程序失败，也就是说，他们的代码中有一个 bug，或者他们在应用程序中配置错误。执行 etcd 恢复到更改之前的状态可以回滚这些更改，但你也会影响集群中所有其他部署的应用程序，并且重新启动集群来回滚本应通过重新部署带有旧代码/设置的应用程序来修复的更改。

注意

为应用程序团队建立回滚部署的流程应该是你环境中的必需项。大多数 CI/CD 系统通常都能选择旧的提交并将其推送出去。

另一个不应该执行 etcd 恢复的主要原因是恢复旧的快照。例如，如果你从几周前的备份恢复，很有可能令牌已经过期，因此恢复后集群无法启动。解决这个问题将需要手动操作来刷新损坏服务的令牌。而且，最大的问题是：“这个集群自从备份以来发生了什么变化？”谁知道自从快照拍摄以来，这个集群发生了哪些升级、部署、代码更改等。你可能在修复一个团队的问题时，破坏了其他团队的应用程序。我遵循的规则是 72 小时。如果快照超过 72 小时，我需要权衡是否恢复它，也就是说，是否大部分时间是在没有进行更改的周末？太好了，我对从周五恢复到周一的快照没问题。但如果我知道应用程序团队喜欢在周四部署，而我正在从周三的快照恢复，那么我应该停下来与应用程序团队沟通，然后再继续操作。

最后，在 Kubernetes 升级后恢复时，我的规则是：升级是恢复的“分界线”，应该只在升级后不久进行恢复。例如，假设我将集群从 v1.19 升级到 v1.20，并且在几分钟内，我的应用程序开始出现问题。那么，太好了，我们可以恢复到升级前的快照。但如果我在周五晚上做了这个升级，而在周二，一个应用程序团队的成员找到了我，说：“嘿，我们看到一些奇怪的错误。你能回滚这个升级吗？”我的回答将是“不行”。自上次升级以来已经过去太长时间，回滚将对集群造成过多影响。当然，我接下来会问他们，“为什么你们的冒烟测试在升级后没发现这个问题？”因为在环境发生重大变化后，冒烟测试应用程序是一个标准流程。

# etcd 备份不能保护什么？

当然，etcd 备份并不覆盖集群中的所有数据，正如我们在本章前面讨论过的，etcd 只存储集群的配置。但集群中还有其他数据没有存储在 etcd 中。最主要的就是卷和卷数据中存储的数据。假设你有一个**PersistentVolumeClaim**（**PVC**）或**PersistentVolume**（**PV**），其中包含卷内的数据。该数据不会存储在 etcd 中，而是存储在存储设备上，即**网络文件系统**（**NFS**）、本地存储、Longhorn 等。etcd 中唯一存储的内容是卷的定义，即卷的名称、大小、配置等。这意味着如果在删除卷后从 etcd 备份恢复集群，根据存储提供商及其保留策略，卷内的数据将丢失。因此，即使你执行 etcd 恢复，集群会创建一个新卷来替代删除的卷，但该卷将是空的，里面没有数据。如果你需要备份卷或其他更高级的备份功能，应该考虑使用像 Veeam 的 Kasten 或 VMware 的 Velero 这样的工具。

另一个在 etcd 备份中没有备份的大项是容器镜像，这意味着使用自定义镜像的部署。etcd 只存储镜像配置，即镜像示例：`docker.io/rancherlabs/swiss-army-knife:v1.0`。但这不包括镜像本身的数据。这通常出现在某人使用自定义镜像部署应用程序后，之后失去对该镜像的访问权限。一个很好的例子是将容器镜像托管在集群中需要的仓库服务器中，例如 Harbor 或 JFrog。

# 如何配置 etcd 备份？

让我们来看看如何为 RKE 和 RKE2 集群配置 etcd 备份。

## RKE 集群

对于 RKE 集群，etcd 备份配置存储在`cluster.yml`文件中。RKE 有两种主要类型的 etcd 备份。第一种是一次性备份，由用户事件手动触发，例如手动运行`rke etcd snapshot-save`命令、升级 Kubernetes 版本或更改集群中的 etcd 节点。第二种是定期快照，这种方式在 RKE v0.1.12 版本发布时默认开启。默认过程是每 12 小时备份一次所有内容。需要注意的是，这个时间表不像 cronjob 那样固定，总是在相同的时间运行，而是基于自上次备份以来经过的时间。

以下是一组用于本地和 S3 等备份的`cluster.yaml`示例文件：

+   仅本地备份 – [`github.com/PacktPublishing/Rancher-Deep-Dive/blob/main/ch09/rke/local-backups.yaml`](https://github.com/PacktPublishing/Rancher-Deep-Dive/blob/main/ch09/rke/local-backups.yaml)

+   本地和 S3 备份 – [`github.com/PacktPublishing/Rancher-Deep-Dive/blob/main/ch09/rke/s3-backups.yaml`](https://github.com/PacktPublishing/Rancher-Deep-Dive/blob/main/ch09/rke/s3-backups.yaml)

+   要查看完整的选项和设置列表，请参阅官方 Rancher 文档，网址为[`rancher.com/docs/rke/latest/en/etcd-snapshots/recurring-snapshots/#options-for-the-etcd-snapshot-service`](https://rancher.com/docs/rke/latest/en/etcd-snapshots/recurring-snapshots/#options-for-the-etcd-snapshot-service)。

## RKE2/k3s 集群

正如我们在本章前面提到的，RKE2 和 k3s 在节点级别处理 etcd 备份，而不是在集群级别处理，这意味着您需要在集群中的每个主节点上定义 etcd 备份计划和其他设置，而不是在集群级别定义。这使您可以做一些有趣的事情，例如为每个节点调整备份计划。例如，第一个节点在 12 A.M.、3 A.M.、6 A.M. 等时间进行备份。第二个节点在 1 A.M.、4 A.M.、7 A.M. 等时间进行备份，第三个节点则在 2 A.M.、5 A.M.、8 A.M. 等时间进行备份。请注意，这通常仅在大型集群中进行，以防止所有 etcd 节点同时备份，因为在备份过程中 etcd 的性能会略微下降。因此，我们希望每次只影响一个 etcd 节点。在低环境中，您也可以仅在第一个节点上配置备份，其中备份很优秀但不是必需的。

要查看完整的选项和设置列表，请参阅 RKE2 的官方 Rancher 文档，网址为 https://docs.rke2.io/backup_restore/ 或 k3s 的文档。有关 k3s 的官方文档，请参见[`rancher.com/docs/k3s/latest/en/backup-restore/`](https://rancher.com/docs/k3s/latest/en/backup-restore/)。需要注意的是，k3s 中的内嵌 etcd 仍处于实验阶段。

注意

如果您需要通过 HTTP 代理访问您的 S3 存储桶，请按照[`docs.rke2.io/advanced/#configuring-an-http-proxy`](https://docs.rke2.io/advanced/#configuring-an-http-proxy)中的文档配置代理设置。

以下是本地和 S3 etcd 备份的示例 rke2 配置文件：

+   仅限本地备份 – [`github.com/PacktPublishing/Rancher-Deep-Dive/blob/main/ch09/rke2/local-backups.yaml`](https://github.com/PacktPublishing/Rancher-Deep-Dive/blob/main/ch09/rke2/local-backups.yaml)

+   本地和 S3 备份 – [`github.com/PacktPublishing/Rancher-Deep-Dive/blob/main/ch09/rke2/s3-backups.yaml`](https://github.com/PacktPublishing/Rancher-Deep-Dive/blob/main/ch09/rke2/s3-backups.yaml)

# 如何进行 etcd 备份？

本节将介绍如何为 RKE 和 RKE2 集群进行 etcd 备份。

## RKE 集群

对于自定义集群，您可以使用以下命令进行一次性备份：

```
rke etcd snapshot-save --config cluster.yml --name snapshot-name 
```

第一个选项设置 `cluster.yml` 文件名。只有在你没有使用默认的 `cluster.yml` 文件名时才需要此选项。第二个选项指定备份的名称。技术上这是可选的，但强烈建议将其设置为具有实际意义的名称，例如 `pre-k8s-upgrade` 和 `post-k8s-upgrade`。也强烈建议避免在文件名中使用特殊字符。如果你使用 S3 备份，设置将默认为 `cluster.yml` 文件中定义的内容。你可以使用命令行标志覆盖这些设置，具体文档见 [`rancher.com/docs/rke/latest/en/etcd-snapshots/one-time-snapshots/#options-for-rke-etcd-snapshot-save`](https://rancher.com/docs/rke/latest/en/etcd-snapshots/one-time-snapshots/#options-for-rke-etcd-snapshot-save)。如果你正在使用通过 Rancher 部署的 RKE 集群，请参阅文档 [`rancher.com/docs/rancher/v2.6/en/cluster-admin/backing-up-etcd/`](https://rancher.com/docs/rancher/v2.6/en/cluster-admin/backing-up-etcd/)。

## RKE2/k3s 集群

RKE2 和 k3s 使用相同的命令来进行备份，只需将 `rke2` 替换为 `k3s`，即可用于 k3s 集群。对于一次性备份，你将运行以下命令：

```
rke2 etcd-snapshot save –name snapshot-name
```

`name` 标志与 RKE 相同，唯一的主要区别是，对于 `config.yaml` 中与 S3 设置无关的所有选项，你可能会遇到 `FATA[0000] flag provided but not defined: ...` 错误。为了解决这个问题，建议仅将 S3 设置复制到一个新的文件 `s3.yaml` 中，路径为 `/etc/rancher/rke2/`，并将 `–config /etc/rancher/rke2/s3.yaml` 标志添加到命令中。

# 如何从 etcd 备份中恢复？

现在让我们来看一下如何从 etcd 备份中恢复数据。

## RKE 集群

对于 RKE 的恢复，你需要运行 `rke etcd snapshot-save --config cluster.yml --name snapshot-name` 命令。至关重要的是，你必须将快照名称设置为你要恢复的快照的文件名，去掉 `.zip` 文件扩展名。假设你是从一个定期快照中恢复。在这种情况下，文件名将包含时间戳的控制字符，因此建议你再次用单引号括起文件名，并确保去除文件扩展名。

注意

如果你正在将 etcd 备份恢复到一个新的集群中，即所有节点都是新的，你将遇到一些令牌问题，并需要解决此问题。你可以使用以下脚本 [`github.com/PacktPublishing/Rancher-Deep-Dive/blob/main/ch09/rke/restore-into-new-cluster.sh`](https://github.com/PacktPublishing/Rancher-Deep-Dive/blob/main/ch09/rke/restore-into-new-cluster.sh) 删除机密并回收服务。此脚本是为一个三节点集群设计的，并假设使用默认设置。

## RKE2/k3s 集群

对于 RKE2 的恢复，工作量比 RKE 稍大一些。第一步是通过`systemctl stop rke2-server`命令停止所有主节点上的`rke2-server`。然后，从一个主节点上重置集群，并使用`rke2 server --cluster-reset --cluster-reset-restore-path=<PATH-TO-SNAPSHOT>`命令恢复 etcd 数据库。恢复完成后，运行`systemctl start rke2-server`命令启动*新的*etcd 集群。接下来，你需要去集群中的其他主节点，并运行`rm -rf /var/lib/rancher/rke2/server/db`命令来删除节点上存储的 etcd 数据，然后重新启动`rke2-server`，使用`systemctl start rke2-server`命令让节点重新加入集群。这将导致一个新的 etcd 成员加入 etcd 集群，并从引导节点同步数据。建议你一次只重新加入一个节点，在该节点进入*Ready*状态后，再重新加入下一个节点。最后，一旦所有主节点重新加入，工作节点应该也能恢复。但 5 分钟后，你可能需要使用`systemctl restart rke2-agent`命令重启 rke2-agent，以加快恢复过程。

# 设置实验环境以测试常见的故障场景

最后，我们通过练习一些常见的故障场景来结束本章。我创建了一个关于此主题的 Kubernetes 课程，名为*使用 Rancher 和 Kubernetes 从灾难中恢复*，可以在 https://github.com/mattmattox/Kubernetes-Master-Class/tree/main/disaster-recovery 上找到，YouTube 视频位于 https://www.youtube.com/watch?v=qD2kFA8THrY。我在这个课程中介绍了一些我创建的训练场景。每个场景都有一个部署实验集群并故障的脚本。然后，我深入讲解了每个场景的故障排除和恢复步骤。最后，课程以一些预防性任务结束。我通常建议新客户至少在将 Rancher/RKE 投入生产前完成这些场景一次。这应该是你感到舒适并且有文档化流程的内容。这包括验证你是否拥有正确的权限，或者是否有文档化的流程来获取这些权限。通常，你需要在所有 etcd、控制平面和主节点上拥有 root/sudo 权限。

# 总结

本章我们学习了 RKE、RKE2、k3s 以及 etcd 的备份与恢复，包括备份和恢复过程的工作原理。我们了解了 etcd 备份的局限性。接着，我们介绍了如何配置定期备份。最后，我们详细讲解了一次性备份和从快照恢复的步骤。本章的结尾，我们讨论了*使用 Rancher 和 Kubernetes 从灾难中恢复*的课程。到此为止，你应该已经能够轻松地备份和恢复你的集群，包括使用 etcd 备份从灾难性故障中恢复。

下一章将讲解 Rancher 中的监控和日志记录。
