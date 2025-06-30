

# 第四章：调试和故障排除 Kubernetes Secrets

到目前为止，我们已经确定了 Kubernetes Secrets 的攻击面。其中两个是静态加密和传输加密。之前，在 *第三章*，*Kubernetes 本地方式加密 Secrets* 中，静态加密帮助我们增强了静态和传输中的安全性。在本章中，我们将重点讨论可能与 Secrets 相关的调试问题。Secrets 在存储和提供 Kubernetes 环境中运行的应用程序和服务所需的敏感信息方面起着至关重要的作用。它们对我们的应用程序至关重要，了解如何有效地调试与 Secrets 相关的问题可以节省大量的时间和精力。

在本章中，我们将扩展以下主题：

+   讨论 Kubernetes Secrets 的常见问题

+   调试和故障排除 Secrets

+   调试和故障排除 Secrets 的最佳实践

到本章结束时，我们将掌握处理与密钥相关挑战的知识。同时，我们应该能够遵循一些解决方法，避免在调试与 Secret 相关问题时常见的陷阱。

# 技术要求

为了将概念与实践相结合，我们将使用一系列常用的工具和平台来与容器、Kubernetes 和密钥管理进行交互。对于本章，我们需要以下工具：

+   **Docker** ([`docker.com`](https://docker.com)) 或 Podman ([`podman.io`](https://podman.io)) 作为容器引擎。

+   **Golang** ([`go.dev`](https://go.dev))，或 Go，是我们示例中使用的编程语言。请注意，Kubernetes 及其大多数第三方组件都是用 Go 编写的。

+   **minikube** ([`minikube.sigs.k8s.io/docs/`](https://minikube.sigs.k8s.io/docs/)) 允许我们在个人电脑上运行单节点的 Kubernetes 集群，非常适合用于学习和开发目的。

+   **Git** ([`git-scm.com`](https://git-scm.com)) 是一个版本控制系统，我们将使用它来恢复书中的示例，同时利用我们对 Secrets 管理解决方案的发现。

+   **Helm** ([`helm.sh`](https://helm.sh)) 是一个用于 Kubernetes 的包管理工具，我们将使用它来简化 Kubernetes 资源的部署和管理。

+   **GnuPG** ([`gnupg.org/download/`](https://gnupg.org/download/)) 是 OpenPGP 的免费开源实现。OpenPGP 为数据通信提供加密隐私和身份验证。

以下链接提供了本书中使用的数字材料：

+   GitHub 仓库：[`github.com/PacktPublishing/Kubernetes-Secrets-Handbook`](https://github.com/PacktPublishing/Kubernetes-Secrets-Handbook)

请注意，将提到 Kubernetes 发行版，如 Azure Kubernetes Engine、Rancher Kubernetes Engine 和 Red Hat OpenShift，但你不需要这些工作实例来执行动手练习。

# 讨论 Kubernetes Secrets 中常见的问题。

在前面的章节中，我们通过直接命令或使用 YAML 文件与 Kubernetes 进行交互。在应用这些 YAML 规范并执行命令时，沿途很容易出现一些错误。错误的 Secret 名称或 YAML 定义可能会导致大量的故障排除工作，以确定最初导致问题的原因。

基于这些原因，需要遵循某些原则：

+   YAML 文件是结构化的，可以作为可信数据源。

+   Secret 的可重用性最小化了错误。

+   自动化消除了容易出错的人为干预。

每次我们通过命令行想要使用 Secret 时都迫切地应用它，容易在规范中引入错误。通过 YAML 文件定义 Secret，使得通过编辑器检查结构变得容易，并确保我们得到了期望的结果。此外，YAML 文件提供了多次应用相同 Secret 的灵活性。如果发生错误，可以修复相同的文件并重新应用。

另一个原则是 Secret 的可重用性。可重用性可以通过多种方式实现。每次应用相同的 Secret 创建时，你遇到错误的可能性就越大。可以驻留在命名空间中的 Secret 可以被不同的组件使用。

## Helm 和 Helm Secrets

当涉及到为 Helm 图表上的 Secrets 提供加密和解密功能时，我们可以选择 Helm Secrets 插件。使用 Helm Secrets，我们可以加密敏感数据以及驻留在 Kubernetes Secrets 中的机密信息，如密码、证书、密钥等。在 Helm Secrets 中有多种加密选项。包括 AWS KMS 和 GCP KMS 等云 KMS 选项。还有一种非供应商依赖的选项，即通过 `gpg` 命令行工具进行加密，它是 OpenPGP 标准的一个实现。

### 创建 PGP 密钥

使用 Helm Secrets 加密文件的一种选择是使用 PGP 密钥。PGP 使用公钥加密；使用公钥和私钥对。公钥用于加密数据，私钥用于解密数据。公钥可以分发，而私钥应保持机密。任何拥有公钥的人都可以加密信息；然而，解密只能通过持有加密密钥的参与者进行。

如果没有 PGP 密钥，我们可以按照以下步骤生成一个：

```
$ gpg --generate-key
```

该命令将生成带有一些默认选项的密钥，例如一年后过期和默认密钥大小。通过使用 `--full-generate-key` 功能，密钥创建时有更多选项。

会出现一个提示要求输入密码短语；忽略此提示并留空。如果未留空密码短语，则在每次解密时都需要输入，这使得操作难以自动化。

我们可以列出密钥并检索我们已经创建的密钥：

```
$ gpg --list-keys
      00FFFE11421E1F1EED1EEE811E11E111D1111111
uid          [ultimate] test-key-2 <kubernetes@secrets.com>
sub   rsa3072 2023-08-27 [E] [expires: 2025-08-26]
```

现在我们已经生成了密钥，可以继续加密 Secrets。

### 加密 Secrets

假设我们有这个包含 Helm 值的文件：

```
jwt-key:
  value: secret-key
```

假设我们已经拥有 GPG 密钥，Helm Secrets 将使用它来加密敏感值。我们需要创建一个 `.sops.yaml` 文件，指定要使用的 GPG 密钥：

```
creation_rules:
    - pgp: >-
        gpg-key
```

然后我们可以加密这些值：

```
helm secrets enc values.yaml
```

最终，我们的文件应该看起来像这样：

```
jwt-key:
    value:  ENC[AES256_GCM,data:9/0gmkCNm2DbEw==,iv:dbthtzx1t8KUHazh7v48T7ASep0rTbYJBrl/jEw6zWE=,tag:MLVXlruHpkMxYihPvNieVQ==,type:str]
sops:
...
    pgp:
          enc: |
            -----BEGIN PGP MESSAGE-----
             -----END PGP MESSAGE-----
    version: 3.7.3
```

Helm 和 Helm Secrets 是许多工具中的两个，它们可以帮助我们以结构化格式组织我们的 Secrets，并将其保存在磁盘上加密和安全。Helm 和 Helm Secrets 是将 Secrets 保存在 YAML 格式中并通过模板化重用的例子。

总结来说，自动化和适当的 Secret 组织不仅能提高我们的生产力，还能大大减少对人工干预的需求。大多数时候，当出现错误时，很可能是人为干预引起的。

## Secret 应用陷阱

在创建 Kubernetes Secrets 时，我们可能会遇到错误，这些错误可能由多种原因引起。错误可能来自无效的 YAML 语法、无效的 Secret 类型、缺失数据或编码错误。幸运的是，未能正确应用 Secret 会立即给我们反馈。干运行可以帮助我们在执行操作之前验证我们的操作。

重要说明

请注意，所有执行的 Kubernetes 命令都会对默认命名空间生效，除非另行指定或通过 Kubernetes 上下文进行配置。以下命令将对默认命名空间或已配置的命名空间生效。

### 干运行

在应用 Kubernetes Secret 之前，我们可以使用 `--dry-run` 选项来模拟创建或更新 Secret。通过将 `--dry-run` 与 `kubectl` 命令一起使用，我们实际上并不会执行任何操作。这是一个有用的功能，可以帮助我们在应用之前测试和验证我们的 Secret 配置，从而节省排查问题的时间。

例如，我们想从字面值创建一个 Secret：

```
$ kubectl create secret generic opaque-example-from-literals --from-literal=literal1=text-for-literal-1 --dry-run=client
secret/opaque-example-from-literals created (dry run)
```

这个输出可以确认我们的操作将会成功。我们可以更进一步，生成一个 YAML 格式的响应：

```
$ kubectl create secret generic opaque-example-from-literals --from-literal=literal1=text-for-literal-1 --dry-run=client -o yaml
secret/opaque-example-from-literals created (dry run)
apiVersion: v1
data:
  literal1: dGV4dC1mb3ItbGl0ZXJhbC0x
kind: Secret
metadata:
  creationTimestamp: null
  name: opaque-example-from-literals
```

干运行可以帮助我们验证操作而不实际应用它，并且我们可以验证操作的结果，而操作不会产生任何效果。尽管 `--dry-run` 选项非常有用，但它无法帮助我们解决已成功应用但包含无效数据内容的 Secrets。一个常见的问题是 Secret 的 Base64 格式问题。

### Base64

Base64 格式用于表示 Secret 值。默认情况下，在应用 Secret 时，如果值是纯文本，它将被编码并以 Base64 格式存储。另外，除了提供纯文本的值外，我们也可以提交已经编码为 Base64 格式的值。如果我们要提交的值本身是 Base64 编码的，这可能会导致问题。

比如说，一个 AES-256 密钥：

```
$ secretKey=$(openssl rand -hex 32)
$ echo "$secretKey"
80a3284da641ac728b5585fd913b0e60e9c4f61ffe2cfb6f456c16a312552e11
$ echo "$secretKey" |md5sum
6a59e95805ea05ff21a708038be9b130
echo "$secretKey"
```

打印出来的 AES 密钥已经使用 Base64 编码。通过 Base64 编码，二进制数据被表示为可打印的 ASCII 字符格式。这使得我们可以更轻松地在代码库中传递，例如通过环境变量。另外，我们打印了 Secret 的 MD5 哈希，稍后我们将使用该哈希进行故障排除。

现在让我们尝试使用之前通过`openssl`命令创建的 AES-256 密钥来创建一个 Secret：

```
$ kubectl create secret generic aes-key --from-literal=key=$secretKey -o yaml
apiVersion: v1
data:
  key: ODBhMzI4NGRhNjQxYWM3MjhiNTU4NWZkOTEzYjBlNjBlOWM0ZjYxZmZlMmNmY jZmNDU2YzE2YTMxMjU1MmUxMQ==
kind: Secret
metadata:
..
type: Opaque
```

如我们所见，Secret 已经被编码。如果我们尝试将 Secret 挂载到 Pod 中，它将包含正确的值：

```
apiVersion: v1
kind: Pod
...
      command: ["/bin/sh","-c"]
      args: ["echo $(key) | md5sum"]
      envFrom:
        - secretRef:
            name:  aes-key
```

我们已经应用了它，现在来查看日志：

```
$ kubectl logs print-env-pod
6a59e95805ea05ff21a708038be9b130  -
```

校验和匹配。如你所见，我们更喜欢使用 MD5 哈希，而不是直接打印变量。在生产环境中打印变量可能导致数据泄漏。

我们将通过 YAML 文件尝试相同的操作，但我们不会对密钥进行编码：

```
apiVersion: v1
kind: Secret
metadata:
  name: aes-key
type: Opaque
data:
  key: 80a3284da641ac728b5585fd913b0e60e9c4f61ffe2cfb6f456c16a312552e11
```

一旦我们通过 Pod 中的环境变量尝试使用该密钥，我们就会遇到一个问题：

```
$ kubectl apply -f base_64_example.yaml
$ kubectl logs -f print-env-pod
fc7d115eb58e428c53b346659e7604d6
```

MD5 哈希不同。这是因为密钥是以二进制格式传递给 Pod 的。Kubernetes 识别到该 Secret 是 Base64 格式的；因此，在创建 Pod 时，它解码了 AES 密钥并将二进制数据作为环境变量放置。这可能会导致混淆并耗费数小时进行调试。

### 特定类型的 Secret

Kubernetes 为我们提供了特定类型的 Secret。这可能会让我们产生一种印象，即在应用 Secret 之前，会进行某些检查，检查 Secret 的格式。此行为可能会根据 Secret 的类型而有所不同。

#### TLS Secret

当我们应用 TLS Secrets 时，Kubernetes 不会对 Secrets 的内容执行任何检查。例如，我们将尝试使用无效的证书创建 TLS Secret：

```
apiVersion: v1
kind: Secret
metadata:
  name: ingress-tls
type: kubernetes.io/tls
data:
  tls.crt: aW52YWxpZC1zZWNyZXQ=
  tls.key: aW52YWxpZC1zZWNyZXQ=
```

我们可能会认为应用 Secret 时会进行检查；然而，这种情况不会发生。Secret 会被创建，最终，当资源尝试挂载并使用它时，可能会导致问题。

#### 基本认证 Secrets

基本认证 Secrets 与 TLS Secrets 属于同一类别。当应用基本认证 Secrets 时，不会进行验证检查：

```
apiVersion: v1
kind: Secret
metadata:
  name: basic-auth-secret
type: kubernetes.io/basic-auth
stringData:
  no-username: a-user
  password: a-password
  another-key: some-value
```

Secret 已被应用，尽管它显然是错误的。`basic-auth` Secret 应该包含用户名和密码；然而，我们添加了不同名称的变量。

#### docker 配置 Secret

对于 docker 配置 Secrets，Kubernetes 会进行验证，并在 Kubernetes 配置无效时发出错误。如果我们尝试应用一个包含无效 Docker 配置的 Secret：

```
apiVersion: v1
kind: Secret
metadata:
  name: docker-credentials
type: kubernetes.io/dockercfg
data:
  .dockercfg: |
     UkVQTEFDRV9XSVRIX0JBU0U2NA==
```

当我们运行以下命令时，文件将不会被应用。我们提供的 docker 配置是错误的：

```
kubectl apply -f docker-credentials.yaml
The Secret "docker-credentials" is invalid: data[.dockercfg]: Invalid value: "<secret contents redacted>": invalid character 'R' looking for beginning of value
```

应用 Secret 时我们可能会遇到不同的错误。有时，我们甚至会创建一个错误的 Secret，而没有收到任何反馈。

到目前为止，我们已经识别了应用 Secrets 时可能遇到的问题，以及如何通过 dry run（模拟运行）来帮助我们。我们还识别了那些 Secret 应用正确但实际 Secret 内容错误的情况。最后但同样重要的是，我们对 Kubernetes 可能不对其进行任何验证的 Secrets 类型进行了概览。这引出了下一个关于故障排除 Secrets 的章节。

# 调试和故障排除 Secrets

调试 Kubernetes Secrets 是困难的。这主要是因为 Secrets 的问题通常是在其他依赖它们的组件失败时才会显现。例如，假设有一个 Ingress 部署使用了错误的证书。识别实际问题将是一个检查多个组件的过程，直到找到根本原因。在本节中，我们将学习调试常见 Secret 问题的工具和方法，例如不存在的 Secrets、配置错误的 Secrets 等。

## `describe`命令

到目前为止，`kubectl get`一直是我们用来检索 Kubernetes 资源信息的主要命令。作为一个命令，它提供了一种快速获取感兴趣资源及其状态信息的方式。然而，`kubectl get`功能只能满足我们的某些需求。一旦需要更多信息，我们应该使用`kubectl describe`命令。通过使用`kubectl describe`命令，我们可以获取 Kubernetes 资源的详细信息。

假设我们在 Kubernetes 集群中有一个 Nginx 部署 Pod，我们将按如下方式描述该部署：

```
$ kubectl describe deployment.apps/nginx-deployment
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Wed, 28 Jun 2023 23:53:22 +0100
...
Pod Template:
  Labels:  app=nginx
  Annotations:      test-annotation: nginx
  Containers:
   nginx:
...
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deployment-544dc8b7c4 (1/1 replicas created)
Events:          <none>
```

`kubectl describe`命令为我们提供了所有关于资源的必要信息。如我们所见，它可以列出事件、标签、注解，甚至是仅适用于被检查资源的属性。

## 不存在的 Secrets

我们将使用`describe`命令来检查一个尝试使用不存在的 Secret 的部署。我们的示例将尝试挂载一个不存在的 Secret 卷：

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
...
    spec:
      containers:
        - name: nginx
          image: nginx
          volumeMounts:
            - name: users-volume
              mountPath: /users.json
      volumes:
        - name: users-volume
          secret:
            secretName: user-file
```

由于我们指定了一个部署，错误不会立即出现。我们的部署将进入持续的`ContainerCreating`状态，直到 Secret 可用。

在这种情况下，通过使用`describe`命令，我们将获取有助于识别问题的信息：

```
kubectl describe pod nginx-5d66b7fbc-7cb7g
...
  Warning  FailedMount  36s (x9 over 2m44s)  kubelet            MountVolume.SetUp failed for volume "users-volume" : secret "user-file" not found
```

由于`describe`命令，我们可以看到 Secret 不可用。

错误挂载的卷反馈不会立即显现。将 Secrets 映射到环境变量时，情况则不同。

举个例子，假设一个 Nginx Pod 通过一个不存在的 Secret 获取其环境变量：

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
    - name: nginx
      image: nginx
      envFrom:
        - secretRef:
            name: does-not-exist
  restartPolicy: Always
```

让我们应用配置。几秒钟后，我们将看到 Pod 无法启动：

```
$ kubectl get po
NAME      READY STATUS                       RESTARTS   AGE
nginx-pod 0/1   CreateContainerConfigError   0          3s
```

我们将使用`describe`命令深入探讨：

```
$ kubectl describe pod nginx-pod
Warning  Failed 3s (x8 over 87s) kubelet Error: secret "does-not-exist" not found
Normal   Pulled 3s               kubelet Successfully pulled image "nginx" in 968.341209ms
```

正如预期的那样，由于 Secret 缺失，Pod 无法启动。这是我们通过`describe`命令检测到的问题。

## 配置错误的 Secrets

配置错误的 Secrets 可能是导致你花费数小时调试的原因。它们可能会影响应用程序的多个组件，导致广泛的故障排除。

一个复杂的场景可能是带有无效 SSL 证书的 ingress。我们之前创建了一个包含无效证书的 SSL 证书 Secret。我们将创建一个 ingress，使用这些 SSL 证书并尝试识别问题。

我们将应用的 Secrets 如下：

```
apiVersion: v1
kind: Secret
metadata:
  name: ingress-tls
type: kubernetes.io/tls
data:
  tls.crt: aW52YWxpZC1zZWNyZXQ=
  tls.key: aW52YWxpZC1zZWNyZXQ=
```

ingress 如下所示：

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  tls:
    - hosts:
        - bad-ssl
      secretName: ingress-tls
  rules:
    - host: bad-ssl
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx-service
                port:
                  number: 80
```

在某个时刻，ingress 将开始工作。我们可以使用 `kubectl get` `ing` 命令来检查：

```
$ kubectl get ing
NAME            CLASS   HOSTS     ADDRESS        PORTS     AGE
nginx-ingress   nginx   bad-ssl   192.168.49.2   80, 443   2m36s
```

最终，通过浏览器访问 ingress，我们将看到一个由 Kubernetes 生成的证书。如果我们没有提供自己的证书或提供了错误的证书配置，Kubernetes 将自动生成一个证书。

第一阶段是检查 Kubernetes 事件：

```
$ kubectl get events
...
11m         Normal    Pulled              pod/webpage                              Container image "nginx:stable" already present on machine
```

此外，根据我们之前的工作，我们可以仅查看 ingress 的事件：

```
$ kubectl describe ing
...
Events:
  Type    Reason  Age                From                      Message
  ----    ------  ----               ----                      -------
  Normal  Sync    35s (x2 over 37s)  nginx-ingress-controller  Scheduled for sync
```

最终，我们将通过查看控制器日志来识别问题：

```
kubectl logs -f ingress-nginx-controller-755dfbfc65-vf7v6 -n ingress-nginx
...
W0701 10:18:30.928989       7 backend_ssl.go:45] Error obtaining X.509 certificate: unexpected error creating SSL Cert: no valid PEM formatted block found
...
W0701 10:18:35.203316       7 controller.go:1348] Unexpected error validating SSL certificate "default/ingress-tls" for server "bad-ssl": x509: certificate is not valid for any names, but wanted to match bad-ssl
```

日志将引导我们找到配置错误的证书。我们的重点将在此。

## 故障排除和可观察性解决方案

到目前为止，我们的故障排除过程涉及了 `kubectl` 的使用，但这并非总是可行的。根据组织的不同，Kubernetes 安装中可能集成了各种可观察性解决方案。例如，Datadog 是一个流行的可观察性解决方案。在基于云的 Kubernetes 服务中，云服务提供商提供的可观察性工具与 Kubernetes 服务集成。这意味着，像 CloudWatch 和 Google Cloud Monitoring 这样的工具可以帮助我们在不使用 `kubectl` 的情况下识别问题。我们将在 *第八章*、《探索 AWS 上的云秘密存储》、*第九章*、《探索 Azure 上的云秘密存储》以及 *第十章*、《探索 GCP 上的云秘密存储》中看到更多相关内容。

我们看到了一些配置错误的 Secrets 场景。为了识别原因，我们遵循了一个流程，并使用了如`describe`等工具，同时查看了 Kubernetes 的日志。

# 调试和故障排除 Secrets 的最佳实践

当一个 Secret 错误时，它可能会以不易察觉的方式影响我们。我们可以采取自上而下的方法，从检查实际受影响的应用程序开始。最终，我们将找到日志指向配置错误的 Secret。一旦找到该 Secret，我们应该确认它是否已正确应用，或者它是否是错误的 Secret。

在评估 Secret 时，我们可以制作一个检查清单：

1.  确保 Secret 存在。

1.  检查 Secret 的值。

1.  解码 Secret，查看它是否为期望的那个。

1.  使用 MD5 哈希。

1.  避免将 Secrets 下载到本地。

接下来要检查的是 Secret 是否应用错误。假设某个 Secret 错误地挂载到某个部署的 Pod 上。虽然可以不断尝试修改部署，最终找出问题所在，但这可能不会带来最佳结果。通过不断尝试直到发现不工作之处虽然简单，但会浪费时间，并且很难辨别实际发生了什么。

应用中可能出现的错误有很多，这会带来噪音，难以辨别问题所在。排除 Secrets 不是问题的可能性，让我们更接近问题的解决。Secrets 本身就是一个复杂的概念。

一种排除 Secret 使用错误的方法是使用一个简单的 Docker 容器并将 Secrets 挂载到该容器上。一个更简单的容器复杂度较低，可以最小化出错的可能性：

1.  将 Secrets 挂载为环境变量到 Pod。

1.  将 Secrets 挂载为文件到 Pod。

1.  使用你选择的哈希算法确保它们是预期的 Secret。

通过遵循这种方法，可以最大限度地减少由 Secrets 引起的任何 Kubernetes 问题的可能性。

## 避免泄露 Secrets

在故障排除时避免泄露任何 Secrets 是至关重要的。当你遇到 Secret 问题时，通常很容易打开终端会话，连接到 Kubernetes Pod 并运行排查命令。容器中生成的日志会被写入标准输出（`stdout`）和标准错误（`stderr`）流中。Kubernetes 与许多流行的日志解决方案（如 AWS CloudWatch、Datadog 和 Google Cloud Monitoring）集成。通过在 Pod 上打印 Secrets，这些 Secrets 会被写入这些流中，并最终出现在集成的日志解决方案中。日志解决方案可能在一个组织内非常容易访问——甚至比直接访问 Kubernetes 集群中的 Secrets 还要容易。这种行为的结果是数据泄露，一旦发生，Secrets 必须被撤销，带来额外的工作量。

另一个最佳实践是在故障排除时避免将 Secrets 下载到本地。将 Secrets 下载到本地可能会导致违反组织已制定的信息安全政策。

# 小结

在本章中，我们深入探讨了调试 Kubernetes Secrets 的细节，并重点介绍了在 Kubernetes 集群中解决和调试与 Secrets 相关的常见问题。我们了解到保持 Secrets 有序和遵循最佳实践的重要性，以及人为错误可能带来的数小时排查工作。我们还介绍了如何识别 Secrets 问题的过程以及可以使用的工具来找出问题的根源。在下一章中，我们将重点讨论 Kubernetes Secrets 的安全性和合规性。

# 第二部分：高级主题 – 在生产环境中的 Kubernetes Secrets

在这一部分，你将探索与 Kubernetes Secrets 相关的更高级话题，包括安全性和合规性考虑、风险缓解策略以及灾难恢复和备份计划。最后，你将学习如何缓解安全风险，以及如何为 Kubernetes Secrets 建立灾难恢复计划和备份策略。

本部分包括以下章节：

+   *第五章*，*安全性、审计与合规性*

+   *第六章*，*灾难恢复与备份*

+   *第七章*，*管理 Secrets 中的挑战与风险*
