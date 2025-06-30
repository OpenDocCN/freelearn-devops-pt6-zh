

# 第九章：打包应用程序

本章我们将探讨 Helm，这个流行的 Kubernetes 包管理工具。每一个成功且非平凡的平台都必须拥有一个良好的打包系统。Helm 是由 Deis 开发的（Deis 于 2017 年 4 月被微软收购），后来直接贡献给 Kubernetes 项目。它在 2018 年成为 CNCF 项目。我们将从理解 Helm 的动机、架构和组件开始。然后，我们将动手实践，看看如何在 Kubernetes 中使用 Helm 和其图表。这包括查找、安装、定制、删除和管理图表。最后，我们将讨论如何创建自己的图表以及如何处理版本控制、依赖关系和模板化。

我们将讨论的主题如下：

+   理解 Helm

+   使用 Helm

+   创建你自己的图表

+   Helm 替代品

# 理解 Helm

Kubernetes 提供了许多方法来在运行时组织和协调容器，但它缺乏将一组镜像高层次组织在一起的方式。这就是 Helm 的作用所在。在本节中，我们将讨论 Helm 的动机、架构和组件。我们将讨论 Helm 3。你可能仍然会在某些地方看到 Helm 2，但它的生命周期已经在 2020 年底结束。

如你所知，Kubernetes 在希腊语中意为舵手或导航员。Helm 项目非常重视这个航海主题，正如项目名称所暗示的那样。Helm 的主要概念是图表。就像航海图详细描述一个海域或沿海区域一样，Helm 图表详细描述了一个应用程序的所有部分。

Helm 的设计目的是执行以下操作：

+   从零开始构建图表

+   将图表打包成归档文件（`.tgz`）

+   与包含图表的仓库互动

+   在现有的 Kubernetes 集群中部署和移除图表

+   处理已安装图表的生命周期

## Helm 的动机

Helm 支持几个重要的使用案例：

+   管理复杂性

+   简单升级

+   简单共享

+   安全回滚

图表可以定义最复杂的应用程序，提供一致的应用安装，并充当中央权限源。就地升级和自定义钩子允许轻松更新。共享图表非常简单，这些图表可以版本控制并托管在公共或私有服务器上。当你需要回滚最近的升级时，Helm 提供一个简单的命令来回滚基础设施的一整套更改。

## Helm 3 架构

Helm 3 架构完全依赖于客户端工具，并将其状态作为 Kubernetes 秘密保存。Helm 3 包含几个组件：发布秘密、客户端和库。

客户端是命令行接口，通常 CI/CD 管道用于打包和安装应用程序。客户端利用 Helm 库执行请求的操作，且每个已部署应用程序的状态都存储在发布秘密中。

让我们回顾一下组件。

### Helm 发布秘密

Helm 将其发布存储为目标命名空间中的 Kubernetes 密钥。这意味着你可以有多个同名的发布，只要它们存储在不同的命名空间中。以下是一个发布密钥的示例：

```
$ kubectl describe secret sh.helm.release.v1.prometheus.v1 -n monitoring
Name:         sh.helm.release.v1.prometheus.v1
Namespace:    monitoring
Labels:       modifiedAt=1659855458
name=prometheus
owner=helm
status=deployed
version=1
Annotations:  <none>
Type:  helm.sh/release.v1
Data
====
release:  51716 bytes 
```

数据被双重 Base64 编码并且经过 GZIP 压缩。

### Helm 客户端

你需要在你的机器上安装 Helm 客户端。Helm 执行以下任务：

+   促进本地 chart 开发

+   管理仓库

+   监督发布

+   与 Helm 库进行交互：

    +   部署新发布

    +   升级现有发布

    +   删除现有发布

### Helm 库

Helm 库是 Helm 核心组件，负责执行所有繁重的任务。Helm 库与 Kubernetes API 服务器通信，并提供以下功能：

+   结合 Helm charts、模板和值文件来构建发布

+   将发布安装到 Kubernetes 中

+   创建发布对象

+   升级和卸载 charts

## Helm 2 与 Helm 3 的区别

Helm 2 非常出色，并在 Kubernetes 生态系统中发挥了重要作用。但是，它的服务器端组件 Tiller 受到了很多批评。Helm 2 在 RBAC 成为官方访问控制方法之前设计和实现。在可用性方面，Tiller 默认安装时具有非常开放的权限集。要为生产环境锁定它并不容易，尤其是在多租户集群中。

Helm 团队听取了批评意见，提出了 Helm 3 设计。Helm 3 取代了集群内的 Tiller 组件，使用 Kubernetes API 服务器本身，通过 CRD 来管理发布的状态。最重要的是，Helm 3 是一个仅客户端程序。它仍然可以管理发布并执行与 Helm 2 相同的任务，但不需要安装服务器端组件。

这种方法更符合 Kubernetes 原生设计，且不那么复杂，安全性问题也得到了消除。Helm 用户只能根据其 kube 配置执行 Helm 允许的操作。

# 使用 Helm

Helm 是一个功能强大的包管理系统，它让你执行所有必要的步骤来管理集群中已安装的应用程序。让我们卷起袖子，开始吧。我们将一起安装 Helm 2 和 Helm 3，但在所有实际操作和演示中我们将使用 Helm 3。

## 安装 Helm

安装 Helm 涉及安装客户端和服务器。Helm 是用 Go 实现的。Helm 2 可作为客户端或服务器使用。如前所述，Helm 3 是一个仅客户端程序。

### 安装 Helm 客户端

你必须正确配置 Kubectl 才能与 Kubernetes 集群进行通信，因为 Helm 客户端使用 Kubectl 配置来与 Kubernetes API 服务器通信。

Helm 在这里提供所有平台的二进制发布：[`github.com/helm/helm/releases`](https://github.com/helm/helm/releases)。

对于 Windows，Chocolatey（[`chocolatey.org`](https://chocolatey.org)）包管理器是最好的选择（通常是最新的）：

```
choco install kubernetes-helm 
```

对于 macOS 和 Linux，你可以通过脚本安装客户端：

```
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh 
```

在 macOS 上，你也可以使用 Homebrew（[`brew.sh`](https://brew.sh)）：

```
brew install helm
$ helm version
version.BuildInfo{Version:"v3.9.2", GitCommit:"1addefbfe665c350f4daf868a9adc5600cc064fd", GitTreeState:"clean", GoVersion:"go1.18.4"} 
```

## 查找 charts

要使用 Helm 安装有用的应用程序和软件，你需要先找到它们的 charts。Helm 是为了与多个 charts 仓库一起使用而设计的。Helm 2 默认配置为搜索 stable 仓库，但你可以添加其他仓库。Helm 3 没有默认仓库，但你可以搜索 Helm Hub（[`artifacthub.io`](https://artifacthub.io)）或特定的仓库。Helm Hub 于 2018 年 12 月发布，旨在简化发现 charts 和仓库的过程，这些仓库是托管在 stable 或 incubator 仓库之外的。

这就是`helm search`命令的作用。Helm 可以在 Helm Hub 中搜索特定的仓库。

目前，hub 中包含 9,053 个 charts：

```
$ helm search hub | wc -l
     9053 
```

我们可以在 hub 中搜索特定的关键词，如`mariadb`。这里是前 10 个 charts（总共有 38 个）：

```
$ helm search hub mariadb --max-col-width 60 | head -n 10
URL                                                             CHART VERSION   APP VERSION DESCRIPTION
https://artifacthub.io/packages/helm/cloudnativeapp/mariadb     6.1.0           10.3.15     Fast, reliable, scalable, and easy to use open-source rel...
https://artifacthub.io/packages/helm/riftbit/mariadb            9.6.0           10.5.12     Fast, reliable, scalable, and easy to use open-source rel...
https://artifacthub.io/packages/helm/bitnami/mariadb            11.1.6          10.6.8      MariaDB is an open source, community-developed SQL databa...
https://artifacthub.io/packages/helm/bitnami-aks/mariadb        11.1.5          10.6.8      MariaDB is an open source, community-developed SQL databa...
https://artifacthub.io/packages/helm/camptocamp3/mariadb        1.0.0                       Fast, reliable, scalable, and easy to use open-source rel...
https://artifacthub.io/packages/helm/openinfradev/mariadb       0.1.1                       OpenStack-Helm MariaDB
https://artifacthub.io/packages/helm/sitepilot/mariadb          1.0.3           10.6        MariaDB chart for the Sitepilot platform.
https://artifacthub.io/packages/helm/groundhog2k/mariadb        0.5.0           10.8.3      A Helm chart for MariaDB on Kubernetes
https://artifacthub.io/packages/helm/nicholaswilde/mariadb      1.0.6           110.4.21    The open source relational database 
```

如你所见，有几个 charts 与关键词`mariadb`匹配。你可以进一步调查这些 charts，找到最适合你使用场景的。

### 添加仓库

默认情况下，Helm 3 没有设置任何仓库，因此你只能搜索 hub。过去，由 CNCF 托管的`stable`仓库是寻找 charts 的一个不错选择。但由于 CNCF 不想继续为其托管付费，现在该仓库只包含大量已弃用的 charts。

相反，你可以选择从 hub 安装 charts，或者进行一些研究并添加单独的仓库。例如，针对 Prometheus，有`prometheus-community` Helm 仓库。让我们添加它：

```
$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
"prometheus-community" has been added to your repositories 
```

现在，我们可以搜索`prometheus`仓库：

```
$ helm search repo prometheus
NAME                                                CHART VERSION   APP VERSION DESCRIPTION                                                                                                                       test | default
prometheus-community/kube-prometheus-stack          39.4.1          0.58.0      kube-prometheus-stack collects Kubernetes manif...
prometheus-community/prometheus                     15.12.0         2.36.2      Prometheus is a monitoring system and time seri...
prometheus-community/prometheus-adapter             3.3.1           v0.9.1      A Helm chart for k8s prometheus adapter
prometheus-community/prometheus-blackbox-exporter   6.0.0           0.20.0      Prometheus Blackbox Exporter
prometheus-community/prometheus-cloudwatch-expo...  0.19.2          0.14.3      A Helm chart for prometheus cloudwatch-exporter
prometheus-community/prometheus-conntrack-stats...  0.2.1           v0.3.0      A Helm chart for conntrack-stats-exporter
prometheus-community/prometheus-consul-exporter     0.5.0           0.4.0       A Helm chart for the Prometheus Consul Exporter
prometheus-community/prometheus-couchdb-exporter    0.2.0           1.0         A Helm chart to export the metrics from couchdb...
prometheus-community/prometheus-druid-exporter      0.11.0          v0.8.0      Druid exporter to monitor druid metrics with Pr...
prometheus-community/prometheus-elasticsearch-e...  4.14.0          1.5.0       Elasticsearch stats exporter for Prometheus
prometheus-community/prometheus-json-exporter       0.2.3           v0.3.0      Install prometheus-json-exporter
prometheus-community/prometheus-kafka-exporter      1.6.0           v1.4.2      A Helm chart to export the metrics from Kafka i...
prometheus-community/prometheus-mongodb-exporter    3.1.0           0.31.0      A Prometheus exporter for MongoDB metrics
prometheus-community/prometheus-mysql-exporter      1.9.0           v0.14.0     A Helm chart for prometheus mysql exporter with...
prometheus-community/prometheus-nats-exporter       2.9.3           0.9.3       A Helm chart for prometheus-nats-exporter
prometheus-community/prometheus-node-exporter       3.3.1           1.3.1       A Helm chart for prometheus node-exporter
prometheus-community/prometheus-operator            9.3.2           0.38.1      DEPRECATED - This chart will be renamed. See ht...
prometheus-community/prometheus-pingdom-exporter    2.4.1           20190610-1  A Helm chart for Prometheus Pingdom Exporter
prometheus-community/prometheus-postgres-exporter   3.1.0           0.10.1      A Helm chart for prometheus postgres-exporter
prometheus-community/prometheus-pushgateway         1.18.2          1.4.2       A Helm chart for prometheus pushgateway
prometheus-community/prometheus-rabbitmq-exporter   1.3.0           v0.29.0     Rabbitmq metrics exporter for prometheus
prometheus-community/prometheus-redis-exporter      5.0.0           1.43.0      Prometheus exporter for Redis metrics
prometheus-community/prometheus-snmp-exporter       1.1.0           0.19.0      Prometheus SNMP Exporter
prometheus-community/prometheus-stackdriver-exp...  4.0.0           0.12.0      Stackdriver exporter for Prometheus
prometheus-community/prometheus-statsd-exporter     0.5.0           0.22.7      A Helm chart for prometheus stats-exporter
prometheus-community/prometheus-to-sd               0.4.0           0.5.2       Scrape metrics stored in prometheus format and ...
prometheus-community/alertmanager                   0.19.0          v0.23.0     The Alertmanager handles alerts sent by client ...
prometheus-community/kube-state-metrics             4.15.0          2.5.0       Install kube-state-metrics to generate and expo...
prometheus-community/prom-label-proxy               0.1.0           v0.5.0      A proxy that enforces a given label in a given ... 
```

这里有相当多的 charts。如果你想获取关于某个特定 chart 的更多信息，我们可以使用`show`命令（你也可以使用`inspectalias`命令）。让我们来看一下`prometheus-community/prometheus`：

```
$ helm show chart prometheus-community/prometheus 
apiVersion: v2
appVersion: 2.36.2
dependencies:
- condition: kubeStateMetrics.enabled
  name: kube-state-metrics
  repository: https://prometheus-community.github.io/helm-charts
  version: 4.13.*
description: Prometheus is a monitoring system and time series database.
home: https://prometheus.io/
icon: https://raw.githubusercontent.com/prometheus/prometheus.github.io/master/assets/prometheus_logo-cb55bb5c346.png
maintainers:
- email: gianrubio@gmail.com
  name: gianrubio
- email: zanhsieh@gmail.com
  name: zanhsieh
- email: miroslav.hadzhiev@gmail.com
  name: Xtigyro
- email: naseem@transit.app
  name: naseemkullah
name: prometheus
sources:
- https://github.com/prometheus/alertmanager
- https://github.com/prometheus/prometheus
- https://github.com/prometheus/pushgateway
- https://github.com/prometheus/node_exporter
- https://github.com/kubernetes/kube-state-metrics
type: application
version: 15.12.0 
```

你还可以要求 Helm 显示`README`文件、values，或者与 chart 相关的所有信息。有时这些信息可能会让人感到不知所措。

## 安装包

好的，你找到了理想的包。现在，你可能想把它安装到你的 Kubernetes 集群中。当你安装一个包时，Helm 会创建一个发布，你可以用它来跟踪安装进度。我们使用`helm install`命令在监控命名空间中安装`prometheus`，并指示 Helm 为我们创建命名空间：

```
$ helm install prometheus prometheus-community/prometheus -n monitoring --create-namespace 
```

我们来看一下输出内容。输出的第一部分列出了我们提供的发布名称`prometheus`，它被部署的时间、命名空间和修订版本：

```
NAME: prometheus
LAST DEPLOYED: Sat Aug  6 23:54:50 2022
NAMESPACE: monitoring
STATUS: deployed
REVISION: 1
TEST SUITE: None 
```

接下来的部分是自定义说明，这些内容可能会比较冗长。这里有关于如何连接到 Prometheus 服务器、告警管理器和`Pushgateway`的很多有用信息：

```
NOTES:
The Prometheus server can be accessed via port 80 on the following DNS name from within your cluster:
prometheus-server.default.svc.cluster.local
Get the Prometheus server URL by running these commands in the same shell:                                                                                        test | default
  export POD_NAME=$(kubectl get pods --namespace default -l "app=prometheus,component=server" -o jsonpath="{.items[0].metadata.name}")
  kubectl --namespace default port-forward $POD_NAME 9090
The Prometheus alertmanager can be accessed via port 80 on the following DNS name from within your cluster:
prometheus-alertmanager.default.svc.cluster.local
Get the Alertmanager URL by running these commands in the same shell:
  export POD_NAME=$(kubectl get pods --namespace default -l "app=prometheus,component=alertmanager" -o jsonpath="{.items[0].metadata.name}")
  kubectl --namespace default port-forward $POD_NAME 9093
#################################################################################
######   WARNING: Pod Security Policy has been moved to a global property.  #####
######            use .Values.podSecurityPolicy.enabled with pod-based      #####
######            annotations                                               #####
######            (e.g. .Values.nodeExporter.podSecurityPolicy.annotations) #####
#################################################################################
The Prometheus PushGateway can be accessed via port 9091 on the following DNS name from within your cluster:
prometheus-pushgateway.default.svc.cluster.local
Get the PushGateway URL by running these commands in the same shell:
  export POD_NAME=$(kubectl get pods --namespace default -l "app=prometheus,component=pushgateway" -o jsonpath="{.items[0].metadata.name}")
  kubectl --namespace default port-forward $POD_NAME 9091
For more information on running Prometheus, visit:
https://prometheus.io/ 
```

### 检查安装状态

Helm 不会等待安装完成，因为这可能需要一段时间。`helm status`命令显示发布的最新信息，格式与初始`helm install`命令的输出相同。

如果你只关心状态，而不需要所有额外的信息，可以直接使用`grep`查找`STATUS`行：

```
$ helm status -n monitoring prometheus | grep STATUS
STATUS: deployed 
```

让我们列出`monitoring`命名空间下的所有 Helm 发布，并验证`prometheus`是否在列出之中：

```
$ helm list -n monitoring
NAME        NAMESPACE   REVISION    UPDATED                                 STATUS      CHART               APP VERSION
prometheus  monitoring  1           2022-08-06 23:57:34.124225 -0700 PDT    deployed    prometheus-15.12.0  2.36.2 
```

如你所记得，Helm 将发布信息存储在一个密钥中：

```
$ kubectl describe secret sh.helm.release.v1.prometheus.v1 -n monitoring
Name:         sh.helm.release.v1.prometheus.v1
Namespace:    monitoring
Labels:       modifiedAt=1659855458
              name=prometheus
              owner=helm
              status=deployed
              version=1
Annotations:  <none>
Type:  helm.sh/release.v1
Data
====
release:  51716 bytes 
```

如果你想找到所有命名空间中的所有 Helm 发布，可以使用：

```
$ helm list -A 
```

如果你想深入了解，可以列出所有带有`owner=helm`标签的密钥：

```
$ kubectl get secret -A -l owner=helm 
```

要从一个密钥中实际提取发布数据，你需要绕过一些障碍，因为它被 Base64 编码了两次（为什么？）并且经过了 GZIP 压缩。最终结果是 JSON 格式：

```
kubectl get secret sh.helm.release.v1.prometheus.v1 -n monitoring -o jsonpath='{.data.release}' | base64 --decode | base64 --decode | gunzip > prometheus.v1.json 
```

你可能还对仅提取清单感兴趣，可以使用以下命令：

```
kubectl get secret sh.helm.release.v1.prometheus.v1 -n monitoring -o jsonpath='{.data.release}' | base64 --decode | base64 --decode | gunzip | jq .manifest -r 
```

### 自定义图表

作为用户，你很可能希望自定义或配置你安装的图表。Helm 完全支持通过配置文件进行自定义。要了解可能的自定义选项，你可以再次使用`helm show`命令，但这次重点关注`values`。对于像 Prometheus 这样复杂的项目，`values`文件可能非常庞大：

```
$ helm show values prometheus-community/prometheus | wc -l
    1901 
```

以下是部分输出：

```
$ helm show values prometheus-community/prometheus | head -n 20
rbac:
  create: true
podSecurityPolicy:
  enabled: false
imagePullSecrets:
# - name: "image-pull-secret"
## Define serviceAccount names for components. Defaults to component's fully qualified name.
##
serviceAccounts:
  alertmanager:
    create: true
    name:
    annotations: {}
  nodeExporter:
    create: true
    name:
    annotations: {} 
```

被注释掉的行通常包含默认值，比如`imagePullSecrets`的名称：

```
imagePullSecrets:
# - name: "image-pull-secret" 
```

如果你想自定义 Prometheus 安装的任何部分，可以将值保存到一个文件中，做出任何修改，然后使用自定义值文件安装 Prometheus：

```
$ helm install prometheus prometheus-community/prometheus --create-namespace -n monitoring -f custom-values.yaml 
```

你也可以在命令行上使用`--set`设置单个值。如果`-f`和`--set`都尝试设置相同的值，那么`--set`优先。你可以使用逗号分隔的列表来指定多个值：`--set a=1,b=2`。嵌套的值可以通过`--set outer.inner=value`来设置。

### 额外的安装选项

`helm install`命令可以与多种来源一起工作：

+   图表仓库（如展示所示）

+   本地图表归档（`helm install foo-0.1.1.tgz`）

+   一个提取的图表目录（`helm install path/to/foo`）

+   完整的 URL（`helm install https://example.com/charts/foo-1.2.3.tgz`）

### 升级和回滚发布

你可能想要将已安装的包升级到最新的版本。Helm 提供了`upgrade`命令，它智能地操作，只更新发生变化的部分。例如，让我们检查一下当前`prometheus`安装的值：

```
$ helm get values prometheus -n monitoring
USER-SUPPLIED VALUES:
null 
```

到目前为止，我们还没有提供任何用户值。作为默认安装的一部分，`prometheus`安装了一个告警管理器组件：

```
$ k get deploy prometheus-alertmanager -n monitoring
NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
prometheus-alertmanager   1/1     1            1           19h 
```

让我们通过升级并传入一个新值来禁用告警管理器：

```
$ helm upgrade --set alertmanager.enabled=false \
     prometheus prometheus-community/prometheus \
     -n monitoring
Release "prometheus" has been upgraded. Happy Helming!
NAME: prometheus
LAST DEPLOYED: Sun Aug  7 19:55:52 2022
NAMESPACE: monitoring
STATUS: deployed
REVISION: 2
TEST SUITE: None
NOTES:
... 
```

升级成功完成。我们可以看到输出中不再提到如何获取告警管理器的 URL。让我们验证一下告警管理器的部署是否被移除：

```
$ k get deployment -n monitoring
NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
prometheus-kube-state-metrics   1/1     1            1           20h
prometheus-pushgateway          1/1     1            1           20h
prometheus-server               1/1     1            1           20h 
```

现在，如果我们检查自定义的值，我们可以看到我们的修改：

```
$ helm get values prometheus -n monitoring
USER-SUPPLIED VALUES:
alertmanager:
  enabled: false 
```

假设我们决定警报其实挺重要的，实际上我们希望拥有 Prometheus 警报管理器。没问题，我们可以回滚到最初的安装版本。`helm history`命令会展示所有我们可以回滚的可用修订版：

```
$ helm history prometheus -n monitoring
REVISION    UPDATED                     STATUS      CHART               APP VERSION DESCRIPTION
1           Sat Aug  6 23:57:34 2022    superseded  prometheus-15.12.0  2.36.2      Install complete
2           Sun Aug  7 19:55:52 2022    deployed    prometheus-15.12.0  2.36.2      Upgrade complete 
```

让我们回滚到修订版 1：

```
$ helm rollback prometheus 1 -n monitoring
Rollback was a success! Happy Helming!
$ helm history prometheus -n monitoring
REVISION    UPDATED                     STATUS      CHART           APP VERSION DESCRIPTION
REVISION    UPDATED                     STATUS      CHART               APP VERSION DESCRIPTION
1           Sat Aug  6 23:57:34 2022    superseded  prometheus-15.12.0  2.36.2      Install complete
2           Sun Aug  7 19:55:52 2022    superseded  prometheus-15.12.0  2.36.2      Upgrade complete
3           Sun Aug  7 20:02:30 2022    deployed    prometheus-15.12.0  2.36.2      Rollback to 1 
```

如你所见，回滚实际上创建了一个新的修订号 3。修订版 2 仍然存在，以防我们想要回到它。

让我们验证一下我们的更改是否已经回滚：

```
$ k get deployment -n monitoring
NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
prometheus-alertmanager         1/1     1            1           152m
prometheus-kube-state-metrics   1/1     1            1           22h
prometheus-pushgateway          1/1     1            1           22h
prometheus-server               1/1     1            1           22h 
```

是的，警报管理器已经恢复。

### 删除发布

当然，你也可以使用`helm uninstall`命令卸载发布。

首先，让我们检查发布列表。我们只有一个`prometheus`发布在监控命名空间中：

```
$ helm list -n monitoring
NAME        NAMESPACE   REVISION    UPDATED                                 STATUS      CHART               APP VERSION
prometheus  monitoring  3           2022-08-07 20:02:30.270229 -0700 PDT    deployed    prometheus-15.12.0  2.36.2 
```

现在，让我们卸载它。你可以使用以下任一等效命令：

+   `uninstall`

+   `un`

+   `delete`

+   `del`

在这里我们使用了`uninstall`命令：

```
$ helm uninstall prometheus -n monitoring
release "prometheus" uninstalled 
```

到此为止，没有更多的发布了：

```
$ helm list -n monitoring
NAME    NAMESPACE   REVISION    UPDATED STATUS  CHART   APP VERSION 
```

Helm 也可以跟踪未安装的发布。如果你在卸载时提供了`--keep-history`，那么你可以通过添加`--all`或`--uninstalled`标志到`helm list`命令来查看未安装的发布。

请注意，监控命名空间仍然存在，尽管它是 Helm 在安装 Prometheus 时创建的，但现在它是空的：

```
$ k get all -n monitoring
No resources found in monitoring namespace. 
```

## 使用仓库

Helm 将图表存储在简单的 HTTP 服务器上的仓库中。任何标准的 HTTP 服务器都可以托管 Helm 仓库。在云中，Helm 团队验证了 AWS S3 和 Google Cloud Storage 都可以作为启用 Web 的 Helm 仓库。你甚至可以将 Helm 仓库存储在 GitHub Pages 上。

请注意，Helm 不提供上传图表到远程仓库的工具，因为这需要远程服务器理解 Helm、知道如何放置图表以及如何更新`index.yaml`文件。

请注意，Helm 最近添加了实验性支持，将 Helm 图表存储在 OCI 注册表中。有关更多详情，请查看[`helm.sh/docs/topics/registries/`](https://helm.sh/docs/topics/registries/)。

在客户端，`helm repo`命令可以让你列出、添加、删除、索引和更新：

```
$ helm repo
This command consists of multiple subcommands to interact with chart repositories.
It can be used to add, remove, list, and index chart repositories.
Usage:
  helm repo [command]
Available Commands:
  add         add a chart repository
  index       generate an index file given a directory containing packaged charts
  list        list chart repositories
  remove      remove one or more chart repositories
  update      update information of available charts locally from chart repositories 
```

我们之前已经使用过`helm repo add`和`helm repo list`命令。让我们看看如何创建自己的图表并管理它们。

## 使用 Helm 管理图表

Helm 提供了多个命令来管理图表。

它可以为你创建一个新的图表：

```
$ helm create cool-chart
Creating cool-chart 
```

Helm 会在`cool-chart`下创建以下文件和目录：

```
$ tree cool-chart
cool-chart
├── Chart.yaml
├── charts
├── templates
│  ├── _helpers.tpl
│  ├── deployment.yaml
│  ├── hpa.yaml
│  ├── ingress.yaml
│  ├── NOTES.txt
│  ├── service.yaml
│  ├── serviceaccount.yaml
│  └── tests
│     └── test-connection.yaml 
```

一旦你编辑了你的图表，你可以将其打包成一个`tar.gz`归档文件：

```
 $ helm package cool-chart
Successfully packaged chart and saved it to: cool-chart-0.1.0.tgz 
```

Helm 会创建一个名为`cool-chart-0.1.0.tgz`的归档文件，并将其存储在本地目录中。

你还可以使用`helm lint`来帮助你找到图表格式或信息中的问题：

```
$ helm lint cool-chart
==> Linting cool-chart
[INFO] Chart.yaml: icon is recommended
1 chart(s) linted, 0 chart(s) failed 
```

### 利用启动包

`helm create` 命令提供了一个可选的 `--starter` 标志，允许你指定一个启动图表。启动图表是位于 `$XDG_DATA_HOME/helm/starters` 中的常规图表。作为图表开发者，你可以创建专门用作新图表模板的图表。在开发这类图表时，请牢记以下注意事项：

+   启动图表中的 YAML 内容将被生成器覆盖。

+   用户通常会修改启动图表的内容，因此提供清晰的文档以解释用户如何进行修改至关重要。

当前，没有内置机制用于安装启动图表。将图表添加到 `$XDG_DATA_HOME/helm/starters` 的唯一方法是通过手动复制。如果你创建了启动包图表，请确保图表的文档中明确提到这一要求。

# 创建你自己的图表

图表表示一组定义 Kubernetes 资源的文件。它可以是一个简单的 Memcached pod 部署，也可以是一个复杂的 Web 应用堆栈配置，包括 HTTP 服务器、数据库、缓存、队列等。

为了组织图表，其文件在特定的目录结构中进行存储。然后，这些文件可以打包成带版本的归档文件，便于部署和管理。关键文件是 `Chart.yaml`。

## Chart.yaml 文件

`Chart.yaml` 文件是 Helm 图表的主要文件。它需要包含名称和版本字段：

+   `apiVersion`：图表的 API 版本。

+   `name`：图表的名称，应该与目录名称匹配。

+   `version`：图表的版本，使用 `SemVer` 2 格式。

此外，`Chart.yaml` 文件中可以包含几个可选字段：

+   `kubeVersion`：以 `SemVer` 格式指定的兼容 Kubernetes 版本范围。

+   `description`：项目的简短描述，一句话概括。

+   `keywords`：与项目相关的关键字列表。

+   `home`：项目主页的 URL。

+   `sources`：项目源代码的 URL 列表。

+   `dependencies`：图表的依赖项列表，包括名称、版本、仓库、条件、标签和别名。

+   `maintainers`：图表维护者的列表，包括姓名、电子邮件和网址。

+   `icon`：可用作图标的 SVG 或 PNG 图片的 URL。

+   `appVersion`：图表中包含的应用程序的版本。它不需要遵循 `SemVer`。

+   `deprecated`：布尔值，指示图表是否已弃用。

+   `annotations`：提供额外信息的附加键值对。

### 图表版本控制

`Chart.yaml` 文件中的 `version` 字段对各种 Helm 工具有着至关重要的作用。在创建包时，它会被 `helm package` 命令使用，因为该命令会根据 `Chart.yaml` 中指定的版本构建包的名称。确保包名称中的版本号与 `Chart.yaml` 文件中的版本号一致非常重要。偏离这一预期可能会导致错误，因为系统会假设这些版本号的一致性。因此，必须保持 `Chart.yaml` 文件中的版本字段与生成的包名称之间的一致性，以避免出现问题。

### appVersion 字段

可选的 `appVersion` 字段与 `version` 字段无关。它不被 Helm 使用，而是作为元数据或文档，供希望了解自己正在部署什么的用户参考。Helm 会忽略它。

### 弃用图表

有时，您可能希望弃用一个图表。您可以通过将 `Chart.yaml` 中的可选 `deprecated` 字段设置为 `true` 来标记该图表为弃用。仅弃用图表的最新版本就足够了。您以后可以重新使用图表名称，并发布一个不再弃用的较新版本。弃用图表的工作流程通常包括以下步骤：

1.  **更新 Chart.yaml 文件**：修改图表的 `Chart.yaml` 文件，标明该图表已弃用。可以通过添加 `deprecated` 字段并将其设置为 `true` 来实现。此外，通常做法是更新图表的版本号，以指示已发布包含弃用信息的新版本。

1.  **发布新版本**：将更新后的图表与弃用信息一起打包并发布到图表仓库。这可以确保用户在尝试安装或升级图表时了解弃用信息。

1.  **沟通弃用信息**：与用户沟通弃用信息并提供替代选项或推荐的迁移路径非常重要。可以通过文档、发布说明或其他渠道来实现，确保用户了解弃用信息，并能相应地进行计划。

1.  **从源代码仓库中移除图表**：一旦弃用的图表已发布并通知用户，建议从源代码仓库中移除该图表，例如 Git 仓库，以避免混淆，并确保用户被引导到图表仓库中的最新版本。

通过遵循这些步骤，您可以有效地弃用一个图表，并为用户提供一个清晰的流程，以便他们过渡到更新的版本或替代方案。

## 图表元数据文件

图表可以包含多个元数据文件，例如 `README.md`、`LICENSE` 和 `NOTES.txt`，这些文件提供有关图表的重要信息。格式化为 Markdown 的 `README.md` 文件尤为重要，应该包含以下详细信息：

+   **应用程序或服务描述**：提供图表所代表的应用程序或服务的清晰简明描述。这个描述应帮助用户理解图表的目的和功能。

+   **先决条件和要求**：指定在使用图表之前需要满足的任何先决条件或要求。这可能包括特定版本的 Kubernetes、所需的依赖项或必须满足的其他条件。

+   **YAML 选项和默认值**：记录用户可以在图表的 YAML 文件中配置的可用选项。描述每个选项、其目的、接受的值或格式，以及默认值。这些信息使用户能够根据自己的需求定制图表。

+   **安装和配置说明**：提供有关如何安装和配置图表的清晰说明。这可能涉及指定命令行选项或 Helm 命令来部署图表，以及配置过程中的任何其他步骤或注意事项。

+   **附加信息**：包括任何其他相关信息，以帮助用户在安装或配置图表时。可能包括最佳实践、故障排除提示或已知的限制。

通过在`README.md`文件中包含这些细节，图表用户可以轻松理解图表的目的、要求，以及如何有效地安装和配置图表以适应他们的特定用例。

如果图表包含模板或 `NOTES.txt` 文件，则该文件将在安装后以及查看发布状态或升级时显示并打印。备注应简洁，以避免冗余，并指向 `README.md` 文件以获取详细说明。通常将使用说明和后续步骤放在 `NOTES.txt` 中。请记住，该文件作为模板进行评估。备注将在您运行 `helm install` 和 `helm status` 时显示在屏幕上。

## 管理图表依赖项

在 Helm 中，一个图表可能依赖于其他图表。这些依赖项通过在 `Chart.yaml` 文件的 `dependencies` 字段中显式列出，或直接复制到 `charts/` 子目录中来表达。这提供了一种充分利用并重用他人知识和工作的好方法。Helm 中的依赖项可以是图表归档（例如 `foo-1.2.3.tgz`）或解压后的图表目录。然而，需要注意的是，依赖项的名称不应以下划线（`_`）或点（`.`）开头，因为这些文件会被图表加载器忽略。因此，建议避免以这些字符开头命名依赖项，以确保它们被 Helm 正确识别并加载。

让我们将来自 `prometheus-community` 仓库的 `kube-state-metrics` 添加为我们 `cool-chart` 的依赖项到 `Chart.yaml` 文件中：

```
dependencies:
  - name: kube-state-metrics
    version: "4.13.*"
    repository: https://prometheus-community.github.io/helm-charts
    condition: kubeStateMetrics.enabled 
```

`name` 字段表示您希望安装的图表的名称。它应与仓库中定义的图表名称匹配。

`version`字段指定你想要安装的图表的具体版本。它有助于确保你获得所需版本的图表。

`repository`字段包含图表仓库的完整 URL，图表将从该仓库获取。它指向存储图表及其版本的位置，并且可以访问。

`condition`字段将在后续章节中讨论。

如果仓库尚未添加，请使用`helm repo`将其添加到本地。

一旦定义了依赖项，你可以运行`helm dependency update`命令。Helm 会将所有指定的图表下载到`charts`子目录中：

```
$ helm dep up cool-chart
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "prometheus-community" chart repository
Update Complete. ![](img/B18998_09_001.png)Happy Helming!![](img/B18998_09_001.png)
Saving 1 charts
Downloading kube-state-metrics from repo https://prometheus-community.github.io/helm-charts
Deleting outdated charts 
```

Helm 将依赖图表存储为`charts/`目录中的档案：

```
$ ls cool-chart/charts
kube-state-metrics-4.13.0.tgz 
```

在`Chart.yaml`依赖字段中管理图表及其依赖项（而不是仅将图表复制到`charts/`子目录中）是一种最佳实践。它明确记录了依赖项，促进了团队间的共享，并支持自动化流水线。

### 使用依赖字段的附加子字段

`requirements.yaml`文件中的每个`requirements`条目可能包括可选字段，如`tags`和`condition`。

这些字段可用于动态控制图表的加载（如果未指定，则会加载所有图表）。如果存在`tags`或`condition`字段，Helm 将对其进行评估，并确定目标图表是否应加载。

+   图表依赖中的`condition`字段包含一个或多个以逗号分隔的 YAML 路径。这些路径引用了顶级父图表值文件中的值。如果路径存在并且评估为布尔值，它将决定是否启用或禁用该图表。如果提供了多个路径，则仅评估第一个有效的路径。如果没有路径存在，则条件不起作用，图表将始终被加载。

+   `tags`字段允许你将标签与图表关联。它是一个 YAML 列表，你可以指定一个或多个标签。在顶级父图表的值文件中，你可以通过指定标签和相应的布尔值来启用或禁用所有具有特定标签的图表。这提供了一种方便的方式来基于关联标签管理和控制图表。

这是一个示例`dependencies`字段和一个`values.yaml`，它充分利用了条件和标签来启用或禁用依赖项的安装。`dependencies`字段根据全局启用字段的值和特定子图的启用字段定义了两个安装其依赖项的条件：

```
dependencies:
  - name: subchart1
    repository: http://localhost:10191
    version: 0.1.0
    condition: subchart1.enabled, global.subchart1.enabled
    tags:
      - front-end
      - subchart1
  - name: subchart2
    repository: http://localhost:10191
    version: 0.1.0
    condition: subchart2.enabled,global.subchart2.enabled
    tags:
      - back-end
      - subchart2 
```

`values.yaml`文件为一些`condition`变量分配了值。`subchart2`标签没有值，因此它会自动启用：

```
# parentchart/values.yaml
subchart1:
  enabled: true
tags:
  front-end: false
  back-end: true 
```

你也可以在安装图表时从命令行设置`tags`和`condition`值，这些值会优先于`values.yaml`文件：

```
$ helm install --set subchart2.enabled=false 
```

标签和条件的解析如下：

+   在值中设置的条件会覆盖标签。每个图表中第一个存在的`condition`路径会生效，其他条件将被忽略。

+   如果与图表关联的任何标签在顶级父项的值中设置为`true`，该图表就被认为是启用的。

+   `tags`和`condition`值必须在值文件的顶层设置。

+   当前不支持在全局配置中嵌套标签表或标签。这意味着标签应该直接位于顶层父项的值下，而不是嵌套在其他结构中。

## 使用模板和值

任何非平凡的应用都需要配置并根据特定的用例进行调整。Helm 图表是使用 Go 模板语言填充占位符的模板。Helm 支持来自 Sprig 库的附加函数，该库包含许多有用的助手函数以及其他几种专用函数。模板文件存储在图表的`templates/`子目录中。Helm 会使用模板引擎渲染此目录中的所有文件并应用提供的值文件。

### 编写模板文件

模板文件仅仅是遵循 Go 模板语言规则的文本文件。它们可以生成 Kubernetes 配置文件以及任何其他文件。以下是 Prometheus 服务器的`service.yaml`模板文件，来自`prometheus-community`库：

```
{{- if and .Values.server.enabled .Values.server.service.enabled -}}
apiVersion: v1
kind: Service
metadata:
{{- if .Values.server.service.annotations }}
  annotations:
{{ toYaml .Values.server.service.annotations | indent 4 }}
{{- end }}
  labels:
    {{- include "prometheus.server.labels" . | nindent 4 }}
{{- if .Values.server.service.labels }}
{{ toYaml .Values.server.service.labels | indent 4 }}
{{- end }}
  name: {{ template "prometheus.server.fullname" . }}
{{ include "prometheus.namespace" . | indent 2 }}
spec:
{{- if .Values.server.service.clusterIP }}
  clusterIP: {{ .Values.server.service.clusterIP }}
{{- end }}
{{- if .Values.server.service.externalIPs }}
  externalIPs:
{{ toYaml .Values.server.service.externalIPs | indent 4 }}
{{- end }}
{{- if .Values.server.service.loadBalancerIP }}
  loadBalancerIP: {{ .Values.server.service.loadBalancerIP }}
{{- end }}
{{- if .Values.server.service.loadBalancerSourceRanges }}
  loadBalancerSourceRanges:
  {{- range $cidr := .Values.server.service.loadBalancerSourceRanges }}
    - {{ $cidr }}
  {{- end }}
{{- end }}
  ports:
    - name: http
      port: {{ .Values.server.service.servicePort }}
      protocol: TCP
      targetPort: 9090
    {{- if .Values.server.service.nodePort }}
      nodePort: {{ .Values.server.service.nodePort }}
    {{- end }}
    {{- if .Values.server.service.gRPC.enabled }}
    - name: grpc
      port: {{ .Values.server.service.gRPC.servicePort }}
      protocol: TCP
      targetPort: 10901
    {{- if .Values.server.service.gRPC.nodePort }}
      nodePort: {{ .Values.server.service.gRPC.nodePort }}
    {{- end }}
    {{- end }}
  selector:
  {{- if and .Values.server.statefulSet.enabled .Values.server.service.statefulsetReplica.enabled }}
    statefulset.kubernetes.io/pod-name: {{ template "prometheus.server.fullname" . }}-{{ .Values.server.service.statefulsetReplica.replica }}
  {{- else -}}
    {{- include "prometheus.server.matchLabels" . | nindent 4 }}
{{- if .Values.server.service.sessionAffinity }}
  sessionAffinity: {{ .Values.server.service.sessionAffinity }}
{{- end }}
  {{- end }}
  type: "{{ .Values.server.service.type }}"
{{- end -}} 
```

它可以在这里找到：[`github.com/prometheus-community/helm-charts/blob/main/charts/prometheus/templates/service.yaml`](https://github.com/prometheus-community/helm-charts/blob/main/charts/prometheus/templates/service.yaml)。

如果它看起来有些混乱，不必担心。基本的思路是，你有一个简单的文本文件，其中包含一些占位符值，这些值可以通过不同的方式在后续进行填充，还有一些条件、函数和管道可以应用于这些值。

#### 使用管道和函数

Helm 允许在模板文件中使用丰富且复杂的语法，借助内建的 Go 模板函数、Sprig 函数和管道。这里是一个模板示例，它利用了这些功能。它使用了`repeat`、`quote`和`upper`函数处理`food`和`drink`键，并通过管道将多个函数链在一起：

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  greeting: "Hello World"
  drink: {{ .Values.favorite.drink | repeat 3 | quote }}
  food: {{ .Values.favorite.food | upper }} 
```

让我们添加一个`values.yaml`文件：

```
favorite:
  drink: coffee
  food: pizza 
```

### 测试和故障排除你的图表

现在，我们可以使用`helm template`命令查看结果：

```
$ helm template food food-chart
---
# Source: food-chart/templates/config-map.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: food-configmap
data:
  greeting: "Hello World"
  drink: "coffeecoffeecoffee"
  food: PIZZA 
```

如你所见，我们的模板化工作成功了。饮料`coffee`被重复了三次并加上了引号，食物`pizza`被转为大写`PIZZA`（未加引号）。

另一种调试的好方法是使用`install`命令并加上`--dry-run`标志。这将提供更多的额外信息：

```
$ helm install food food-chart --dry-run -n monitoring
NAME: food
LAST DEPLOYED: Mon Aug  8 00:24:03 2022
NAMESPACE: monitoring
STATUS: pending-install
REVISION: 1
TEST SUITE: None
HOOKS:
MANIFEST:
---
# Source: food-chart/templates/config-map.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: food-configmap
data:
  greeting: "Hello World"
  drink: "coffeecoffeecoffee"
  food: PIZZA 
```

你还可以在命令行中覆盖这些值：

```
$ helm template food food-chart --set favorite.drink=water
---
# Source: food-chart/templates/config-map.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: food-configmap
data:
  greeting: "Hello World"
  drink: "waterwaterwater"
  food: PIZZA 
```

最终的测试，当然，是将你的图表安装到你的集群中。你无需将图表上传到图表仓库进行测试；只需在本地运行`helm install`命令即可：

```
$ helm install food food-chart -n monitoring 
NAME: food
LAST DEPLOYED: Mon Aug  8 00:25:53 2022
NAMESPACE: monitoring
STATUS: deployed
REVISION: 1
TEST SUITE: None 
```

现在，已经有一个名为`food`的 Helm 发布：

```
$ helm list -n monitoring
NAME    NAMESPACE   REVISION    UPDATED                                 STATUS      CHART               APP VERSION
food    monitoring  1           2022-08-08 00:25:53.587342 -0700 PDT    deployed    food-chart-0.1.0    1.16.0 
```

最重要的是，`food-configmap` 配置映射已使用正确的数据创建：

```
$ k get cm food-configmap -o yaml -n monitoring
apiVersion: v1
data:
  drink: coffeecoffeecoffee
  food: PIZZA
  greeting: Hello World
kind: ConfigMap
metadata:
  annotations:
    meta.helm.sh/release-name: food
    meta.helm.sh/release-namespace: monitoring
  creationTimestamp: "2022-08-08T07:25:54Z"
  labels:
    app.kubernetes.io/managed-by: Helm
  name: food-configmap
  namespace: monitoring
  resourceVersion: "4247163"
  uid: ada4957d-bd6d-4c2e-8b2c-1499ca74a3c3 
```

### 嵌入内置对象

Helm 提供了一些内置对象，你可以在模板中使用。在上面的 Prometheus 图表模板中，`Release.Name`、`Release.Service`、`Chart.Name` 和 `Chart.Version` 是 Helm 预定义值的例子。其他对象包括：

+   `Values`

+   `Chart`

+   `Template`

+   `Files`

+   `Capabilities`

`Values` 对象包含在 `values` 文件或命令行中定义的所有值。`Chart` 对象是 `Chart.yaml` 的内容。`Template` 对象包含有关当前模板的信息。`Files` 和 `Capabilities` 是类似映射的对象，通过各种函数访问非专门文件和有关 Kubernetes 集群的一般信息。

请注意，`Chart.yaml` 中的未知字段会被模板引擎忽略，不能用来向模板传递任意结构化数据。

### 从文件中传递值

以下是 Prometheus 服务器默认 `values` 文件的一部分。该文件中的值用于填充多个模板。它们表示默认值，你可以通过复制该文件并修改以适应你的需求来覆盖这些默认值。注意文件中的有用注释，它们解释了每个值的目的和各种选项：

```
server:
  ## Prometheus server container name
  ##
  enabled: true
  ## Use a ClusterRole (and ClusterRoleBinding)
  ## - If set to false - we define a RoleBinding in the defined namespaces ONLY
  ##
  ## NB: because we need a Role with nonResourceURL's ("/metrics") - you must get someone with Cluster-admin privileges to define this role for you, before running with this setting enabled.
  ##     This makes prometheus work - for users who do not have ClusterAdmin privs, but wants prometheus to operate on their own namespaces, instead of clusterwide.
  ##
  ## You MUST also set namespaces to the ones you have access to and want monitored by Prometheus.
  ##
  # useExistingClusterRoleName: nameofclusterrole
  ## namespaces to monitor (instead of monitoring all - clusterwide). Needed if you want to run without Cluster-admin privileges.
  # namespaces:
  #   - yournamespace
  name: server
  # sidecarContainers - add more containers to prometheus server
  # Key/Value where Key is the sidecar `- name: <Key>`
  # Example:
  #   sidecarContainers:
  #      webserver:
  #        image: nginx
  sidecarContainers: {} 
```

这就是深入了解如何使用 Helm 创建自定义图表的内容。实际上，Helm 在 Kubernetes 应用程序的打包和部署中被广泛使用。然而，Helm 并不是唯一的选择。还有一些不错的替代方案，可能会更符合你的需求。在接下来的部分中，我们将回顾一些最具前景的 Helm 替代方案。

# Helm 替代方案

Helm 已经经过战斗测试，并且在 Kubernetes 世界中非常常见，但它也有一些缺点和批评，尤其是在你自己开发图表时。很多批评集中在 Helm 2 及其服务器端组件 Tiller 上。然而，Helm 3 并不是万能的。在大规模的环境中，当你开发自己的图表和包含大量条件逻辑的复杂模板时，再加上庞大的 `values` 文件，管理起来会变得非常具有挑战性。

如果你感到困难，可能想要探索一些替代方案。需要注意的是，这些项目大多数都专注于部署方面。Helm 的依赖管理仍然是其优势之一。让我们看看一些有趣的项目，你或许可以考虑使用它们。

## Kustomize

Kustomize 是一种替代 YAML 模板化的工具，它通过在原始 YAML 文件之上使用覆盖层的概念来实现。它在 Kubernetes 1.14 中被添加到 kubectl。

查看 [`github.com/kubernetes-sigs/kustomize`](https://github.com/kubernetes-sigs/kustomize)。

## Cue

Cue 是一个非常有趣的项目。它的数据验证语言和推理受到了逻辑编程的强烈启发。它不是一种通用编程语言，而是专注于数据验证、数据模板化、配置、查询和代码生成，但也包含一些脚本功能。Cue 的主要概念是类型和数据的统一。这使得 Cue 拥有强大的表达能力，并且不再需要像枚举（enums）和泛型（generics）这样的构造。

请参阅 [`cuelang.org`](https://cuelang.org)。

这里有一个关于用 Cue 替代 Helm 的具体讨论：[`github.com/cue-lang/cue/discussions/1159`](https://github.com/cue-lang/cue/discussions/1159)。

## kapp-controller

kapp-controller 为 Kubernetes 提供了持续交付和包管理功能。

它的声明式 API 和分层方法使你能够有效地构建、部署和管理应用程序。使用 Kapp-controller，你可以将软件打包成可分发的包，并帮助用户无缝地在 Kubernetes 集群上发现、配置和安装这些包。

请参阅 [`carvel.dev/kapp-controller/`](https://carvel.dev/kapp-controller/)。

这就是我们对 Helm 替代方案的简要回顾。

# 总结

在本章中，我们介绍了 Helm，这是一款流行的 Kubernetes 包管理器。Helm 赋予 Kubernetes 管理由多个相互依赖的 Kubernetes 资源组成的复杂软件的能力。它的作用类似于操作系统的包管理器。Helm 组织包，允许你搜索图表（charts），安装和升级图表，并与协作者共享图表。你还可以开发自己的图表并将它们存储在仓库中。Helm 3 是一个仅限客户端的解决方案，使用 Kubernetes secrets 来管理集群中发布的状态。我们还介绍了一些 Helm 的替代方案。

到此为止，你应该已经理解了 Helm 在 Kubernetes 生态系统和社区中的重要角色。你应该能够高效使用 Helm，甚至开发并共享自己的图表。

在下一章，我们将讨论 Kubernetes 如何在较低层次上进行网络配置。
