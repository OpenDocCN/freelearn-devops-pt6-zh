# 解决方案

本节包含了每个课时活动的解答。请注意，对于描述性问题，您的答案可能与本节提供的答案不完全一致。只要答案的本质保持相同，就可以认为它们是正确的。

# 第一章：Kubernetes 设计模式

以下是本章活动的解决方案。

# 活动：运行带有同步的 Web 服务器

在`sidecar.yaml`文件中，提供了一个包含两个容器的 Pod 定义，分别为`server`和`sync`。在 server 容器中，httpd 在 80 端口提供源卷服务。在 sync 容器中，git 每 30 秒运行一次以同步源卷。这两个容器独立工作；然而，它们共享源卷来实现文件同步：

```
apiVersion: v1
kind: Pod
metadata:
       name: sidecar
spec:
     containers:
...
volumes:
- emptyDir: {}
    name: source 
```

按照以下步骤获取解决方案：

1.  使用以下命令创建 Pod：

```
 kubectl apply -f sidecar.yaml 
```

1.  检查名为 sidecar 的 Pod 是否准备好：

```
 kubectl get pod sidecar 
```

1.  当 Pod 准备好时，检查同步 sidecar 容器的日志：

```
 kubectl logs sidecar -c sync

```

1.  使用以下命令将 Pod 的服务器端口转发到 localhost：

```
 kubectl port-forward sidecar 8000:80

```

1.  在浏览器中打开`localhost:8000`。预期看到 2048 游戏。

# 活动：在内容准备后运行 Web 服务器

在`init-container.yaml`文件中，提供了一个包含初始化容器和主容器的 Pod 定义，分别为`content`和`server`。在 content 容器中，`"Welcome from Packt"`被写入索引文件。在 server 容器中，nginx 在 80 端口提供源卷服务。这两个容器独立工作；然而，它们共享`workdir`卷来进行文件准备：

```
apiVersion: v1
kind: Pod
metadata:
      name: init-container
spec:
     initContainers:
-
...
volumes:
- name: workdir
    emptyDir: {} 
```

按照以下步骤获取解决方案：

1.  使用以下命令创建 Pod：

```
kubectl apply -f init-container.yaml

```

1.  检查初始化容器的状态：

```
kubectl describe pod init-container

```

1.  当 Pod 正在运行时，使用以下命令将 Pod 的服务器端口转发到 localhost：

```
kubectl port-forward init-container 8000:80 
```

1.  在另一个终端中，使用以下命令检查服务器的内容：

```
curl localhost:8000

```

1.  运行以下命令进行清理：

```
kubectl delete deployment go-client 
```

# 活动：向应用程序注入数据

在`inject.yaml`文件中，有一个 Pod 定义，其中的容器每 10 秒记录一次运行时信息。为内存和 CPU 定义了资源请求和限制，用于利用率和性能信息。

环境变量通过`valueFrom`块定义，表示在运行时填充的值：

按照以下步骤获取解决方案：

1.  使用以下命令创建 Pod：

```
kubectl apply -f inject.yaml

```

1.  检查 inject Pod 是否准备好：

```
kubectl get pod inject

```

1.  当 Pod 准备好时，检查日志：

```
kubectl logs inject

```

# 第二章：Kubernetes 客户端库

以下是本章活动的解决方案。

# 活动：在集群内使用 Kubernetes Go 客户端

1.  使用前一个练习中的示例客户端 Docker 镜像创建一个部署：

```
kubectl run go-client --image=onuryilmaz/k8s-clientexample:go

```

1.  使用以下命令等待 Pod 启动：

```
kubectl get pods -w

```

1.  使用以下命令获取部署 Pod 的日志：

```
kubectl logs $(kubectl get pods --selector run=go-client -o jsonpath="{.items[0].metadata.name}")
```
