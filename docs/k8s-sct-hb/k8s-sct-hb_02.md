# 2

# 逐步了解 Kubernetes Secrets 管理概念

在上一章中，我们对 Kubernetes 及其组成组件进行了全面概述，并了解了配置是如何应用和存储的。此外，我们构建了一个 Golang 应用，并成功地在 Kubernetes 上运行了该应用。正如预期的那样，Secrets 需要被添加到我们的应用配置中。Secrets 管理涉及多种问题。从创建、修改到删除，我们需要解决安全性、可扩展性和弹性等方面的挑战。

在本章中，我们将讨论以下主题：

+   什么是 Kubernetes Secrets，它与其他 Kubernetes 对象有何不同？

+   不同类型的 Secrets 及其使用场景

+   在 Kubernetes 中创建、修改和删除 Secrets

+   在不同部署场景中的 Kubernetes Secrets 配置

+   管理机密的要求，包括安全存储和访问控制

+   使用 RBAC 保护对 Secrets 的访问

+   审计和监控 Secret 使用情况

# 技术要求

为了将概念与实践示例结合，我们将利用一系列常用的工具和平台，这些工具和平台通常用于与容器、Kubernetes 和 Secrets 管理进行交互。本章中，我们将使用一个友好的桌面图形解决方案：

+   **Podman Desktop** ([`podman-desktop.io`](https://podman-desktop.io)) 是一个**开源软件**（**OSS**），它与容器进行交互，运行本地 Kubernetes 实例，甚至连接远程平台，如 Red Hat OpenShift、**Azure Kubernetes Service**（**AKS**）等。在本章中，我们将使用 Go 编程语言。要在系统中安装 Go，可以参考官方文档中的安装说明（[`go.dev/doc/install`](https://go.dev/doc/install)）。

+   在本章中，还将使用 **minikube**。要在系统中安装 minikube，可以参考官方文档中的安装说明（[`minikube.sigs.k8s.io/docs/start/`](https://minikube.sigs.k8s.io/docs/start/)）。

+   本书中的所有代码示例都可以在我们的专用 GitHub 仓库中找到，仓库结构清晰，并且为每一章提供了相应的文件夹和说明（[`github.com/PacktPublishing/Kubernetes-Secrets-Handbook`](https://github.com/PacktPublishing/Kubernetes-Secrets-Handbook)）。

# 什么是 Kubernetes Secrets，它与其他 Kubernetes 对象有何不同？

Kubernetes 的一个基本构建模块是 Kubernetes 对象。通过 Kubernetes 对象，我们可以表示系统的状态。运行在 Kubernetes 上的应用程序包括实际的程序、应用程序使用的资源和应用程序的配置（如健康检查）。关于其他横向关注点，如安全性，存在 **基于角色的访问控制**（**RBAC**）配置；这些配置包括集群范围的角色、命名空间角色以及与用户或实体绑定的角色。此外，Kubernetes 对象还包括命名空间，它们充当逻辑容器，以及网络策略，它们是集群范围的流量规则。通过创建 Kubernetes 对象，我们声明了集群的期望状态。Kubernetes 负责并将致力于确保系统的实际状态与我们创建的对象所定义的状态匹配。

一个典型的 Kubernetes 对象有一些必填字段：`apiVersion`、`kind`、`metadata` 和 `spec`。

我们可以看到它的 YAML 表示如下：

```
apiVersion: apps/v1 #version of Kubernetes api
kind: Deployment    #type of Object
metadata:           #metadata information
  name: example-deployment
spec:               #the state the object should be
```

Kubernetes 用户最常接触到的 Kubernetes 对象如下：

+   `Pod`

+   `Deployment`

+   `StatefulSet`

+   `Cronjob`

+   `Service`

+   `Ingress`

+   `NetworkPolicy`

+   `ConfigMap`

+   `Secret`

之前提到的对象可以逻辑上归为表示工作负载的对象。像 `Pod`、`Deployment`、`StatefulSet` 和 `Cronjob` 这样的对象用于定义执行特定任务的计算资源；这些任务可能是运行服务器、执行定时任务，甚至设置分布式内存网格。像 `Service`、`Ingress` 和 `NetworkPolicy` 这样的对象指定了应用程序的网络方面；这可以是负载均衡内部流量、将 Kubernetes 服务暴露到互联网，或在应用程序之间阻止内部流量。到目前为止，提到的 Kubernetes 对象都针对应用程序的计算资源部署和应用程序之间的流量路由。

`ConfigMap` 和 `Secret` 在使用上有所不同，因为它们分别用于配置存储。`ConfigMap` 和 `Secret` 是由在 Kubernetes 上运行的应用程序使用的对象。`ConfigMap` 可以用来存储配置，常见的配置示例包括存储在 `nginx.conf` 中的 `nginx` 配置、基于 JSON 的配置，或基于 YAML 的应用程序配置。`Secret` 对象用于存储敏感数据。例如，在 `nginx` 配置中，我们需要存储在 `.key` 文件中的 TLS 密钥和存储在 `.pem` 文件中的证书。这两者都是敏感文件，需要安全处理。此安全处理还应适用于用户名和密码、访问令牌等凭证。实际上，Kubernetes Secrets 是用于存储敏感配置数据的 Kubernetes 对象，因此应该限制访问，并确保以安全的方式处理 Secrets 中存储的信息。

# 不同类型的 Secrets 及其使用场景

Kubernetes 为我们提供了多种类型的 Secrets。在幕后，它使用我们在 *第一章* 中看到的相同存储机制，*理解 Kubernetes Secrets 管理*；一旦创建，Secrets 将被序列化并存储在 `etcd` 中。不同之处在于这些 Secrets 在使用时的处理方式。Secrets 有多种类型，让我们逐一查看。

## Opaque

Opaque secret 是默认的 secret 类型。每当我们需要添加敏感配置时，无论是文件还是变量，它都会作为 Opaque secret 创建。

Opaque Secrets 可以通过提供键值对来使用：

```
$ kubectl create secret generic opaque-example-from-literals --from-literal=literal1=text-for-literal-1
$ kubectl get secret opaque-example-from-literals -o yaml
apiVersion: v1
data:
  literal1: dGV4dC1mb3ItbGl0ZXJhbC0x
kind: Secret
...
type: Opaque
```

Opaque Secrets 也可以通过应用 YAML 文件来执行：

```
$ kubectl create secret generic opaque-example-from-literals --from-literal=literal1=text-for-literal-1
$ kubectl create secret generic secretfile --from-file=secret-file.txt=./secret.file.txt
kubectl get secret secretfile -o yaml
apiVersion: v1
data:
  secret-file.txt: QSBmaWxlIHdpdGggc2Vuc2l0aXZlIGRhdGE=
kind: Secret
metadata:
...
type: Opaque
```

Opaque Secrets 的示例可以在 `ch02/secret-types/opaque` 文件夹中找到。`opaque.sh` 脚本将运行所需的 Bash 命令以实现最终结果。

## Kubernetes 服务账户令牌

Pod 是 Kubernetes 中的一个工作单元；需要与 Kubernetes API 进行交互的 Pod 需要一个身份。服务账户是可以直接或通过部署间接映射到 Pod 的身份。只要 Pod 配置了服务账户，它就可以与 Kubernetes API 进行交互。附加的服务账户被授权访问感兴趣的资源。在启动时，配置了服务账户的 Pod 会将服务账户令牌附加到其文件系统上。

### 长期有效的访问令牌

在 Kubernetes 中，在 v1.27 版本之前，服务账户令牌会作为 Kubernetes 管理的 Kubernetes secret 进行访问。这被称为长期有效的访问令牌。

在最新版本中，仍然可以创建长期有效的访问令牌。可以通过创建一个空的 secret 并在其中添加一个带有服务账户名称的注解来实现：

```
apiVersion: v1
kind: Secret
metadata:
  name: service-account-secret
  annotations:
    kubernetes.io/service-account.name: example-service-account
type: kubernetes.io/service-account-token
```

如我们所见，我们注意到在 `annotations` 部分中有服务账户。通过运行 `apply` 命令，我们应该能够看到生成了一个令牌：

```
$ kubectl create sa example-service-account
kubectl apply -f service-account-secret.yaml
kubectl get secret service-account-secret -o yaml
apiVersion: v1
data:
  ca.crt: ...==
  namespace: default
  token: eyJhbGxffQ.eyJhdWQ3RlbTpdW50In0.0LyJWAc2M9SdA3g
kind: Secret
metadata:
  annotations:
...
type: kubernetes.io/service-account-token
```

创建长期有效访问令牌的步骤可以参考以下脚本：`ch02/secret-types/service-account/long-live-access-token.sh`

### 服务账户令牌已挂载到 Pod

我们已经看过服务账户作为 secret 的情况；现在让我们看看如何将服务账户令牌挂载到 Pod 上。

一个带有服务账户的 Pod 应该是这样的：

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: example-service-account
---
apiVersion: v1
kind: Pod
...
spec:
  ...
  serviceAccountName: example-service-account
```

一旦我们应用了前面的 YAML 清单，我们可以在刚刚调度的 Pod 中运行一个命令。我们将按如下方式打印挂载的服务账户令牌：

```
$ kubectl exec -it busybox -- cat /var/run/secrets/kubernetes.io/serviceaccount/token
eyJhbGdkxfTlUifQ.eyJhdWQidwid3RlbTpdW50In0.0LyJWAc2M9SdA3g
```

如我们所见，这是一个 **JSON Web Token** (**JWT**) 令牌。

创建一个带有服务账户的 Pod 的步骤可以参考以下脚本：`ch02/secret-types/service-account/service-account-with-pod.sh`

## Docker 配置

在 Pod 上使用镜像时，我们可能希望从替代的容器注册中心拉取镜像。为此，我们需要挂载 Docker 配置，以便能够与我们选择的注册中心进行通信。我们可以通过仅使用本地 Docker 配置来测试这一点。

我们将使用以下模板生成一个 YAML 清单：

```
apiVersion: v1
kind: Secret
metadata:
  name: registry-docker-config
type: kubernetes.io/dockercfg
data:
  .dockercfg: |
     REPLACE_WITH_BASE64
```

你可以看到`REPLACE_WITH_BASE64`字符串；它将被 Docker Hub 的 Docker 配置替换。

在`docker-credentials`文件夹中，已经存在一个用于此目的的 Docker 配置文件，位于`ch02/secret-types/docker-credentials/config.json`，其中没有实际凭证：

```
{
  "auths": {
    "https://index.docker.io/v1/": {}
  }
}
```

我们将发起登录并使用我们的 Docker Hub 凭证：

```
$ docker --config ./ login --username=dockerhub-username --password=dockerhub-password
```

文件将包含连接到 Docker Hub 所需的基本认证：

```
{
...
    "auth": "token"
...
}
```

我们将使用这个配置将其挂载为 Kubernetes 机密：

```
$ DOCKER_CONFIG=$(cat ./config.json|base64)
$ cat docker-credentials-template.yaml|sed "s/REPLACE_WITH_BASE64/$DOCKER_CONFIG/" > docker-credentials.yaml
```

在接下来的步骤中，我们将通过应用我们创建的 YAML 清单将凭证上传到 Kubernetes：

```
$ kubectl apply -f docker-credentials.yaml
```

然后，我们将创建一个 Pod，从注册表拉取：

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
  imagePullSecrets:
  - name: docker-credentials
```

镜像将使用指定的凭证进行拉取。

前面的指令已经通过以下脚本进行编排：`ch02/secret-types/docker-credentials/docker-credentials.sh`

## 基本认证

基本认证由用户名和密码的密钥组合组成。它使我们能够在指定基本认证机密时提供更具声明性的方式。

YAML 清单应包含`username`和`password`键的值：

```
apiVersion: v1
kind: Secret
metadata:
  name: basic-auth-secret
type: kubernetes.io/basic-auth
stringData:
  username: a-user
  password: a-password
```

一旦我们应用前面的 YAML 清单，结果将与不透明机密非常相似。

前面的指令已经通过以下脚本进行编排：`ch02/secret-types/basic-authentication/basic-auth-secret.sh`

## TLS 客户端或服务器

TLS 机密用于存储 SSL/TLS 证书。TLS 机密可以在挂载 TLS 证书时提供更具声明性的方式。然而，TLS 机密在指定 Ingress 时可以直接使用。

Ingress 充当系统的外部负载均衡器，处理 HTTP/HTTPS 流量。流量需要使用 SSL 进行加密。

SSL 机密具有以下格式：

```
apiVersion: v1
kind: Secret
metadata:
  name: ingress-tls
type: kubernetes.io/tls
data:
  tls.crt: CRT
  tls.key: KEY
```

通过使用`ch02/secret-types/ssh/tls.sh`脚本，我们将创建一个证书和密钥，可以在 HTTP 服务器上使用。创建的机密将命名为`ingress-tls`。作为证书使用的脚本将具有`webpage.your.hostname`主机。

让我们使用之前创建的 TLS 证书来创建 Ingress：

```
apiVersion: networking.k8s.io/v1
kind: Ingress
...
spec:
  tls:
  - secretName: ingress-tls
    hosts:
      - webpage.your.hostname
  rules:
  - host: webpage.your.hostname
...
```

通过使用 Ingress，我们可以为主机定义 SSL 配置。

### 关于 minikube 用户的说明

如果你在整个`minikube.sh`脚本中使用了 minikube，你应该按照以下方式在工作站上启用 Ingress：

```
$ minikube addons enable ingress
```

现在我们可以测试 Ingress。请注意，Ingress 需要获得一个 IP 地址：

```
$ kubectl get ing
NAME           CLASS   HOSTS             ADDRESS        PORTS     AGE
webpage-ingress   nginx   webpage.your.hostname   192.168.49.2   80, 443   79s
```

由于此 IP 可能属于内部虚拟机，我们需要执行`minikube tunnel`命令，将流量转发到我们的 Ingress：

```
$ minikube tunnel
```

通过访问[`localhost/`](https://localhost/)，我们可以查看证书：

![图 2.1 – SSL 证书](img/B20970_02_01.jpg)

图 2.1 – SSL 证书

此外，如果我们想验证 Ingress 路由，我们可以更改`/etc/hosts`并将`webpage.your.hostname`的 DNS 映射到`localhost`。

## Token 数据

这种类型的 Secret 是一个引导令牌。它看起来像我们在 REST API 中使用的常规持有令牌；在 Kubernetes 中，它专门用于 Kubernetes 集群的引导过程。在初始化 Kubernetes 集群时，会创建一个引导令牌，之后可以用它将新节点加入到集群中。

## 结论

我们深入探讨了 Kubernetes Secrets，识别了不同类型的 Secrets，并运行了每种类型的示例，展示了它们的使用和特点。有关 Secrets 的最新发展，您可以随时参考官方文档（[`kubernetes.io/docs/concepts/configuration/secret/#secret-types`](https://kubernetes.io/docs/concepts/configuration/secret/#secret-types)）。在本节中，Secrets 的配置是通过 `kubectl` 命令行完成的。在接下来的部分，我们将探索管理 Secrets 的选项，创建、删除和修改它们。

# 在 Kubernetes 中创建、修改和删除 Secrets

之前，我们专注于创建 Secrets 并展示它们的用法。接下来，我们将继续管理 Secrets，并识别可用的命令和选项来配置 Kubernetes Secrets。

## data 和 stringData

我们通过使用 YAML 文件或命令行应用明文 Secrets。在幕后，我们应用的明文 Secrets 会被转换为 `base64` 格式。我们可以选择应用明文 Secrets 或使用 `base64` 格式应用它们；最终，它们会以 `base64` 格式存储在 Kubernetes 中。当我们使用明文值应用一个 Secret 时，我们使用 `stringData` 字段。Kubernetes 会处理我们提供的值的编码和解码。

以以下 Secret 为例：

```
apiVersion: v1
kind: Secret
metadata:
  name: plain-text
type: Opaque
stringData:
  value: non-base64
```

一旦我们创建了 Secret，我们将检索它。它应该是 `base64` 格式的：

```
$ kubectl apply -f plain-text.yaml
$ kubectl get secret plain-text -o yaml|grep value
  value: bm9uLWJhc2U2NA==
```

该值以 `base64` 格式存储。这是 Kubernetes 存储 Secrets 时遵循的一种约定。如果我们考虑到 Secret 可能具有的不同变体，这尤其有用。一个 Secret 可能有一个复杂的值；例如，一个大型 YAML 文件或甚至是二进制文件。

对于之前描述的复杂情况，我们可以选择使用 `data` 字段。当我们以 `base64` 格式应用 Kubernetes Secrets 时，我们使用 `data` 字段：

```
apiVersion: v1
kind: Secret
metadata:
  name: base64-encoded
type: Opaque
data:
  value: bm9uLWJhc2U2NA==
```

现在我们已经掌握了创建 Secrets 的知识，我们将继续进行其他操作，比如 `update` 和 `delete`。

## 更新 Secrets

在 Kubernetes 对象中，有一些基本命令可以帮助我们管理它们。这些命令同样适用于 Secrets，因为它们也是 Kubernetes 对象。

### 编辑 Secrets

编辑一个 Secret 是通过使用 `kubectl` 的 `edit` 命令完成的。`kubectl` 自带一个预配置的编辑器。默认情况下，编辑器是 Vim：

```
$ kubectl edit secret plain-text
```

如我们所见，在编辑 Secret 时，它将以 `base64` 格式呈现。如果我们尝试使用明文更改 Secret，将会失败。当我们编辑 Secret 时，必须提供一个 `base64` 值。

使用 `kubectl` 编辑 secret 时的一个选项是通过 `—``record=true` 参数记录导致更改的命令：

```
$ kubectl edit secret plain-text --record=true
$ kubectl get secret plain-text -o yaml
...
    kubernetes.io/change-cause: kubectl edit secret plain-text --record=true
...
```

正如我们所见，我们发出的 `edit` 命令已被记录下来。

出于备份目的以及跟踪先前状态的需求，在编辑时我们可以使用 `—``save-config=true` 参数：

```
$ kubectl edit secret plain-text --save-config=true
$ kubectl get secret plain-text -o yaml
...
kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion...."type":"Opaque"}
...
```

在 `last-applied-configuration` 字段中，我们将备份先前的配置。

到目前为止，我们已经编辑了 Secrets，并且成功跟踪了导致 Secrets 变化的命令，同时还记录了最后应用的配置。这并不总是如此；有时，我们可能希望 Secrets 是不可变的，这是我们将在下一节实现的目标。

### 不可变的 Secrets

在某些情况下，我们可能希望我们的 Secrets 保持不变；例如，我们希望防止意外的编辑。以下是我们如何实现这一点：

```
apiVersion: v1
kind: Secret
metadata:
  name: immutable-secret
type: Opaque
stringData:
  value: non-base64
immutable: true
```

如果我们尝试编辑以下 secret，一旦我们尝试保存，就会遇到以下错误消息：

```
data: Forbidden: field is immutable when `immutable`is set
```

此外，如果我们将现有的 secret 设置为不可变，那么它将无法被编辑；该 secret 将变得永久不可变。要修改不可变的 secret，唯一的方式是删除该 secret 并重新应用它。接下来，我们将学习如何删除 Kubernetes Secrets。

展示不可变 Secrets 的示例可在 `ch02/secret-types/secret-management/immutable/immutable-secret.sh` 中找到。

## 删除 Secrets

删除 Kubernetes 对象的命令同样适用于 Secrets。

以下示例将删除一个 Kubernetes secret（如果它存在的话）：

```
kubectl delete secret immutable-secret
```

通过删除一个 secret，它将被永久从我们的系统中移除。唯一能够恢复它的方式是恢复一个包含该 secret 的 `etcd` 备份，或者应用一个通过以下命令手动备份的文件：

```
kubectl get secret immutable-secret –o yaml
```

## 结论

在本节中，我们更进一步地管理了 Kubernetes Secrets。我们更新了 Secrets，跟踪了我们的更改，并且还备份了先前的配置。此外，我们创建了不可变的 Secrets，以防止意外编辑，最后但同样重要的是，我们删除了不再需要的 Secrets。在接下来的章节中，我们将重点讨论在不同环境中使用 secret 的相关问题。

# 在不同部署场景中的 Kubernetes Secrets 配置

在**软件开发生命周期**（**SDLC**）中，一个团队可能会使用不同的环境来测试他们的增量更新，然后再将其发布到生产环境中。就像生产部署一样，其他任何环境中的部署也会有一定的配置要求，包括 Secrets。

## 环境间的 Secret 使用

当涉及到 Secrets 时，我们需要确保它们在任何环境下的持久性和完整性。在不同环境中对 Secrets 的处理不同可能会导致长期问题，并且团队将无法完全验证对 secret 处理选择的安全影响。

鉴于环境之间可能存在成本节约要求或完整安装带来更多开销的差异，Secrets 需要被安全存储。可能会出现 Secrets 需要共享的情况。例如，某个外部 SaaS 服务的专有密钥需要在不同环境之间共享。另一个例子是，当多租户云账户托管多个环境时。

## 从开发到部署

要部署一个 secret，敏感信息需要存放在某个地方。这些信息最终会由某个个人插入系统并应用到 Kubernetes。如今，公司将敏感信息存储在专门为此类信息设计的各种系统中。简而言之，安全存储是托管 Secrets 的必要条件。

部署 Kubernetes secret 这类敏感内容的生命周期，从从安全存储中获取证书开始，创建所需的 YAML 文件并应用到 Kubernetes。

对于 CI/CD 任务，大多数 CI/CD 提供商为我们提供了在任务中使用秘密值的选项。这可以帮助我们为 CI/CD 任务提供凭证，以便与秘密存储进行交互。

另一种范式是 GitOps。Argo CD 是一个极受欢迎的工具，可以自定义秘密部署，以便在解密后应用 secret。

## 结论

在不同环境下，我们对 Secrets 的处理应该保持一致，无论环境如何。这有助于自动化以及一致性。

# 管理 Secrets 的要求，包括安全存储和访问控制

在责任方面，Kubernetes 集群有责任安全地容纳 Secrets 并防止未经授权的访问。每个托管在 Kubernetes 上的秘密都是由个人或自动化过程存储的。某个时间点，当前存储在 Kubernetes 上的这个秘密曾经存在于另一个系统中。因此，在 Secrets 到达 Kubernetes 之前，确保它们被安全存储非常重要。

## 安全存储

有许多专门用于安全存储的工具。例如，HashiCorp Vault、**Google Cloud Platform**（**GCP**）Secret Manager 和**Amazon Web Services**（**AWS**）Secrets Manager。这些是外部的 Secrets 管理解决方案。

这些解决方案的好处是，它们既可以作为独立的 Secrets 管理系统使用，也可以直接从 Kubernetes 中使用。在开发过程中，甚至在 CI/CD 任务中使用安全存储也是可行的。

这些解决方案的共同点是它们处理了跨切关注点，如管理、版本控制、加密和访问控制功能。

## 访问控制

访问控制对确保我们的秘密存储安全至关重要。持久性、静态加密和传输加密使得我们与安全存储系统的交互更加安全，但这还不够。当涉及到访问 Secrets 时，我们需要具有精细的访问控制。

我们需要区分用户和他们在组织中的角色。此外，权限可能会因环境而异。另一个方面是审计和识别是否发生了未授权访问事件。

## Git 和加密

除了使用安全存储系统外，另一种流行的选择是将 Secrets 存储为加密形式。通过将加密后的 Secrets 存储在 Git 仓库中，Git 的功能使得多个方面变得可行。例如，版本控制通过提交历史默认启用，访问控制通过 Git 的访问控制规则得到满足，存储的韧性和持久性依赖于提供商的保障。至于加密，解决方案的优劣取决于所选择的加密方式。数据可以以各种形式加密，从 **Pretty Good Privacy** (**PGP**) 密钥（[`www.openpgp.org/`](https://www.openpgp.org/)）到硬件安全模块，再到现代 **云密钥管理服务** (**cloud KMS**) 解决方案。基于此的一个非常流行的工具是 **Mozilla Secrets OPerationS** (**SOPS**): [`github.com/mozilla/sops`](https://github.com/mozilla/sops)。Mozilla SOPS 使用云提供商提供的 KMS 以及 PGP。

## 结论

就像每个机密一样，它们的访问应该受到限制，并且在任何情况下都不应允许未授权人员访问。因此，除了存储 Secrets 的位置外，我们还需要提供适当的访问控制。

# 使用 RBAC 确保对 Secrets 的访问

在 Kubernetes 对象中，一个跨切面的问题是授权访问。总体而言，系统的状态是敏感的。您应当对某些操作有授权访问，例如更改部署的副本数或更改部署的自动扩展规则。Kubernetes 为我们提供的安全机制是 RBAC。

## RBAC 介绍

RBAC 由以下 Kubernetes 对象组成：

+   角色

+   角色绑定

+   集群角色

+   集群角色绑定

我们将分别检查每个组件，看看它们如何与 Kubernetes Secrets 结合使用。

### 角色

角色是一组仅在角色所在命名空间内生效的权限。通过指定一个角色，我们定义了可以对 Kubernetes 资源执行的操作。角色的格式如下：

```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: secret-viewer
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get","list","watch"]
```

`verbs` 是我们应该能够执行的操作，`resources` 是这些操作的目标。`apiGroups` 指向我们将与之交互的资源的 API 组；通过设置空值，它表示核心 API 组。

前面的 `Role` 对象使得拥有该角色的操作员能够获取、列出并观察默认命名空间中的 Secrets。让我们继续并将该角色绑定给一个操作员。

### 角色绑定

通过检查如何使用 YAML 表示 `Role` 对象，我们可以识别出动作和目标。角色绑定帮助我们定义行为者。行为者可以是一个用户（个人或群组）或一个服务账户。角色绑定有以下 YAML 表现：

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: secret-viewer-binding
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: secret-viewer
subjects:
- kind: ServiceAccount
  name: secret-viewer
  namespace: default
```

在定义角色绑定时，命名空间是必须存在的。这是因为角色绑定只在它们所在的命名空间内生效。在`roleRef`中，我们定义一个应该位于同一命名空间中的角色。在`subjects`中，我们定义一个访问该角色的行为者列表。请注意，行为者可以来自不同的命名空间。

### 集群角色

集群角色与角色非常相似：它们定义了一组权限；然而，它们是集群范围内生效的，而不是仅限于某一个命名空间。它们有以下 YAML 表现：

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: secret-admin-cluster
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["*"]
```

这与 `Role` 对象几乎完全相同，唯一的不同是没有命名空间，因为这些规则适用于整个集群。前面的 `ClusterRole` 角色使得具有该角色的行为者能够管理 Kubernetes 集群中所有命名空间的 Secrets。我们现在可以继续将该 `ClusterRole` 对象绑定到一个行为者。

### 集群角色绑定

通过使用集群角色绑定，我们将一个集群角色绑定到一组用户和服务账户。集群角色绑定有以下 YAML 表现：

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: secret-admin-cluster-binding
subjects:
- kind: ServiceAccount
  name: secret-admin
  namespace: default
roleRef:
  kind: ClusterRole
  name: secret-admin-cluster
  apiGroup: rbac.authorization.k8s.io
```

现在我们已经了解了 RBAC 以及如何使用它来保护 Secrets，我们可以开始一个端到端的示例。

## RBAC 和 Secrets

在之前的示例中，我们为集群的秘密资源创建了角色和集群角色。我们集中讨论了一个用于查看权限的角色和一个用于管理 Secrets 的管理员角色，但值得看看是否还有其他选项。

我们可以通过使用 `api-resources` 调用来识别与 Secrets 相关的操作：

```
$ kubectl api-resources -o wide|grep secrets
secrets v1 true Secret [create delete deletecollection get list patch update watch]
```

现在我们已经知道了有哪些选项，我们将为我们的 Secrets 创建自己的 RBAC 配置。

### ClusterRole

我们将使用我们之前创建的 YAML 文件中的 `ClusterRole` 规范创建一个集群角色，以管理 Secrets：

```
$ kubectl create sa secret-admin
$ kubectl apply -f ./secret-admin-cluster.yaml
```

这应该创建一个能够在整个集群范围内管理 Secrets 的集群角色。我们将使用一个附加了 `ClusterRole` 对象的 Pod，并检查秘密的创建：

```
apiVersion: v1
kind: Pod
metadata:
  name: kubectl-create-secret
spec:
  containers:
  - name: kubectl
    image: bitnami/kubectl:latest
    args:
    - create
    - secret
    - generic
    - test
    - --from-literal=literal1=text-for-literal-1
  serviceAccount: secret-admin
  serviceAccountName: secret-admin
```

通过检查日志，我们应该看到以下消息：

```
secret/test created
```

该 Pod 被配置为使用服务账户，而服务账户有一个集群绑定到 `ClusterRole` 对象，该对象具有对 Secrets 的管理员权限。

### 角色

我们将使用之前创建的角色，该角色提供查看 Secrets 的权限：

```
$ kubectl create sa secret-viewer
$ kubectl apply -f ./secret-viewer.yaml
```

我们现在应该在默认命名空间中有一个用于查看 Secrets 的角色。我们将运行一个 Pod 来检索 Secrets：

```
apiVersion: v1
kind: Pod
metadata:
  name: kubectl-get-secrets
spec:
  containers:
  - name: kubectl
    image: bitnami/kubectl:latest
    args:
    - get
    - secret
    - secret-toview
  serviceAccount: secret-viewer
  serviceAccountName: secret-viewer
```

## 结论

在本节中，我们介绍了 RBAC 以管理 Secrets。我们确定了集群内对 Secrets 可执行的操作，并区分了 `ClusterRole` 和 `Role` 对象。接着，我们着手保障集群内 Secret 的安全使用，并提供了细粒度的授权访问，无论是通过角色限制访问某个命名空间，还是在整个集群中授予访问权限。由于我们已完成授权访问的需求，接下来需要关注的是监控我们的 Secret 使用情况。

# 审计和监控 Secret 使用情况

为了记录和监控 Kubernetes 集群中的持续活动，我们可以选择进行审计。Kubernetes 集群中的事件会发送到输出流或保存为日志；这使得我们可以识别系统中发生了什么。

在我们的案例中，我们希望监控 Secret 的使用情况。为了避免其他活动的开销，我们将仅专注于为 Secrets 生成的审计日志。

启用审计功能以监控 Secrets 的配置应如下所示：

```
apiVersion: audit.k8s.io/v1
kind: Policy
omitStages:
  - "RequestReceived"
rules:
  - level: Metadata
    resources:
    - group: ""
      resources: ["secrets"]
```

在 Kubernetes 安装中，可以通过使用 `--audit-policy-file` 标志，并在运行 `kube-apiserver` 时传递它来实现：

```
kube-apiserver --audit-policy-file=/path/to/audit-policy.yaml
```

## minikube note

在 minikube 的情况下，我们需要在启动 minikube 时传递审计配置。

我们已经在 `minikube-script.sh` 脚本中总结了这些操作：

```
minikube start \
  --extra-config=apiserver.audit-policy-file=/etc/ssl/certs/audit-policy.yaml \
  --extra-config=apiserver.audit-log-path=-
```

既然我们已经启用了审计功能，接下来检查日志：

```
$ kubectl logs -f kube-apiserver-minikube -n kube-system | grep audit.k8s.io/v1
```

要触发审计事件，我们可以执行一个 Secret 操作：

```
$ kubectl get secret
```

最终，我们将收到以下日志：

```
{"kind":"Event",...,"verb":"list","user":{"username":"minikube-user","groups":["system:masters","system:authenticated"]},"sourceIPs":["192.168.49.1"],"...,"responseStatus":{"metadata":{},"code":200},...}
```

通过启用 Kubernetes 的审计功能，我们成功地跟踪和监控了 Secret 的变化。

# 摘要

在本章中，我们深入探讨了 Kubernetes Secrets。我们了解了不同类型的 Kubernetes Secrets 及其使用场景，并通过执行代码片段来展示这些用例。此外，由于 Secrets 包含敏感信息，我们进一步探讨了通过应用 RBAC 规则来确保对 Secrets 的访问安全。这帮助我们限制了对 Secrets 的访问，但也为我们的 Pods 提供了授权访问。我们还涵盖了审计这一方面。审计是一个非常重要的环节，因为我们希望能够全面控制对 Secrets 的访问以及其他操作。在下一章中，我们将重点讨论传输中的和静态存储中的 Secrets 加密。
