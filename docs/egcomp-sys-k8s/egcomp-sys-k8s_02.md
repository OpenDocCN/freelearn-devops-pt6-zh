# 2

# K3s 的安装和配置

本章快速深入探讨了 K3s。我们将从理解 K3s 及其架构开始，然后学习如何为 K3s 准备 ARM 设备。接下来，您将学习如何从单节点集群到多节点集群进行基本安装，并使用 MySQL 进行后端配置。此外，本章还涵盖了如何安装 Ingress 控制器，使用 Helm Charts 和 Helm 将您的服务暴露在由 NGINX 创建的负载均衡器上。最后，我们将看看如何卸载 K3s 并排除集群故障。在章节末尾，您将找到额外的资源，以实现 K3s 的其他自定义。

在本章中，我们将涵盖以下主要内容：

+   介绍 K3s 及其架构

+   准备您的边缘环境以运行 K3s

+   创建 K3s 单节点和多节点集群

+   使用外部 MySQL 存储 K3s

+   安装 Helm 以在 Kubernetes 中安装软件包

+   更改默认的 Ingress 控制器

+   从主节点或代理节点卸载 K3s

+   排除 K3s 集群故障

# 技术要求

对于本章，您将需要以下选项之一：

+   建议最低配置为 4 GB RAM 的 Raspberry Pi 4 Model B

+   创建一个 AWS 账号以创建 **Graviton2** 实例

+   任何安装了 Linux 的 x86_64 VM 实例

+   用于本地 K3s 集群的互联网连接和 DHCP 支持

有了这些要求，我们将安装 K3s 并开始尝试这个 Kubernetes 发行版。所以，让我们开始吧。

# 介绍 K3s 及其架构

K3s 是由 Rancher Labs 创建的轻量级 Kubernetes 发行版。它包含所有必要的组件在一个小型二进制文件内。Rancher 删除了这个 Kubernetes 发行版运行集群所需的所有不必要的组件，并添加了其他有用的功能，以在边缘运行 K3s，例如将 MySQL 支持作为 `etcd` 的替代品，优化的 Ingress 控制器，适用于单节点集群的存储等。让我们查看 *图 2.1*，以了解 K3s 的设计和打包方式：

![图 2.1 – K3s 集群组件](img/B16945_Figure_2.1.jpg)

图 2.1 – K3s 集群组件

在前面的图示中，你可以看到 K3s 有两个组件：服务器和代理。每个组件必须安装在一个节点上。节点是一个裸金属机器或虚拟机，作为主节点或代理节点工作。主节点管理并配置 Kubernetes 对象，如部署、服务和 Ingress 控制器，在代理节点内部运行。代理节点负责处理使用这些对象的信息。每个节点使用图示*Figure 2.1*中展示的不同组件，这些组件通过一个单一的二进制文件提供，包含运行主节点和代理节点所需的所有必要组件。在进程层面，主节点运行 K3s 服务器，代理节点运行 K3s 代理。对于每个组件，你会找到一个隧道代理，用于将主节点与代理节点（即工作节点）互联。

默认情况下，K3s 的*代理节点和主节点运行*Flannel 作为默认的**容器网络接口**（**CNI**）插件。CNI 是容器网络的规范，CNI 插件是用于管理容器网络连接的接口。它还安装了**containerd**作为容器引擎，用于创建 Pods。服务器和代理节点有一个共同点，即每个组件都由一个大约 100MB 的单一二进制文件组成，包含运行每个节点所需的所有最小组件。然而，在需要时，你可以添加 K3s 中去除的其他组件，这些组件在原生 Kubernetes 集群中是包含的。

就每个节点的角色而言，主节点被称为**控制平面**，即管理所有 Kubernetes 集群配置、网络等的节点。相比之下，代理节点被称为**数据平面**，它上面运行着所有服务、网络流量和处理任务。

# 准备你的边缘环境以运行 K3s

在安装 K3s 之前，你需要按照以下步骤为你的 ARM 设备配置 K3s 主节点或代理节点。让我们开始吧。

## 可以使用的硬件

首先，你必须准备好你的设备。关于如何准备设备，有几种选择。第一种是购买一台树莓派设备，开始实验并创建一个低成本的边缘系统。在购买设备时，你需要考虑以下硬件规格和组件：

+   至少拥有 4GB 内存的树莓派 4 Model B 作为 ARM 设备。

+   推荐使用 5V、3A 的电源。

+   一根用于互联网连接的以太网线。

+   一条 Micro HDMI 转 HDMI 的线缆。

+   推荐使用一张 MicroSD 卡：SanDisk Extreme MicroSDHC UHS-1 A1 V30 32GB，或者类似的卡片。

+   一张 MicroSD 卡读卡器。

这个配置将为你带来最大的性价比。你可能会想 *为什么选择这个配置？* 好吧，让我简要解释一下。与之前的版本相比，Raspberry Pi 4 Model B 在处理速度上有了很多改进。谈到兼容性，Raspberry Pi 配备的 ARMv7 处理器支持多种编程语言和程序。它还支持 ARM64 或 AArch64 处理器的操作系统，这些处理器用于具有 ARMv8 处理器的设备。Raspberry Pi B 型号支持这些处理器架构。然而，对于更多面向生产的设备，你可能会想选择一款 ARM 64 位设备，比如 UDOO X86 II ULTRA，它配备了 64 位处理器。

关于电源，你需要一个 5V、3A 的设备，以防止 Raspberry Pi 运行变慢。你可以使用 5V/2.4A 的电源，但 5V/3A 的电源对 Raspberry Pi 4 Model B 更合适。如果你预算允许，建议选择带有 8GB 内存的 4 Model B。

最后，针对 MicroSD 卡，选择一张高速卡。当你运行软件时，这样的卡会有更好的性能。SanDisk 有一款不错的 MicroSD 卡；只需查看读写速度，并选择一张至少有 32 GB 存储空间的 MicroSD 卡。如果可能的话，不要使用 Wi-Fi；这也是选择使用以太网电缆的原因，以便保证稳定的连接。

## 适用于 ARM 设备的 Linux 发行版

根据你的使用案例，你可以选择多个 GNU/Linux 发行版或操作系统：

+   **Raspbian**：这是第一个可以在 Raspberry Pi 设备上使用的发行版，专为 Raspberry Pi 设备优化。它可靠且易于使用。

+   **Ubuntu**：这个发行版可以在 Raspberry Pi 设备或其他 ARM 64 位设备上使用，包括 x86_64 设备。Ubuntu 的一个优势是它可以在所有主要的云服务提供商中找到，比如 AWS、Azure 和 GCP。

+   **Alpine**：这是一个小型发行版，具有最小的软件集，旨在成为一个轻量级的发行版。它可以作为你的下一个项目，用来根据项目需求定制你自己的发行版。

+   **k3OS**：这是一个小型发行版，专门设计用于仅在边缘设备上运行 K3s，但它非常灵活。

还有其他发行版，但你可以使用这些作为边缘项目的快速启动。

### 在你的 MicroSD 卡中安装 Ubuntu

现在是时候安装你的操作系统了。要在你的 MicroSD 卡中安装 Linux 发行版，首先，你必须下载适合你系统的 Raspberry Pi Imager。在这里，我们使用的是 Mac 版本。你可以在 [`www.raspberrypi.org/software`](https://www.raspberrypi.org/software) 下载。

要开始在 Raspberry Pi 设备中安装操作系统，请执行以下步骤：

1.  安装前面链接中的二进制文件并打开它；你应该看到类似这样的界面：

![图 2.2 – Raspberry Pi Imager 菜单](img/B16945_Figure_2.2.jpg)

图 2.2 – Raspberry Pi Imager 菜单

1.  点击 **选择操作系统** 按钮，选择适用于 ARM64 的 Ubuntu Server 20.04 64-bit 操作系统，可以通过导航到 **其他通用操作系统** | **Ubuntu** 菜单找到：

![图 2.3 – 树莓派发行版选择](img/B16945_Figure_2.3.jpg)

](img/B16945_Figure_2.3.jpg)

图 2.3 – 树莓派发行版选择

1.  接下来，插入你的 MicroSD 卡（你必须购买适配器来读取 MicroSD 卡）。在点击 **选择存储** 按钮后，你的设备会显示出来：

![图 2.4 – 存储选择](img/B16945_Figure_2.4.jpg)

](img/B16945_Figure_2.4.jpg)

图 2.4 – 存储选择

1.  然后，点击 **写入** 按钮：

![图 2.5 – 将发行版安装到存储设备的最后一步](img/B16945_Figure_2.5.jpg)

](img/B16945_Figure_2.5.jpg)

图 2.5 – 将发行版安装到存储设备的最后一步

1.  接受写入设备的选项。Raspberry Pi Imager 会要求你输入用户名和密码，以继续将数据写入 MicroSD 卡：

![图 2.6 – 确认写入 MicroSD 卡](img/B16945_Figure_2.6.jpg)

](img/B16945_Figure_2.6.jpg)

图 2.6 – 确认写入 MicroSD 卡

1.  等待写入过程完成：

![图 2.7 – 将操作系统写入 MicroSD 卡](img/B16945_Figure_2.7.jpg)

](img/B16945_Figure_2.7.jpg)

图 2.7 – 将操作系统写入 MicroSD 卡

1.  等待验证过程完成：

![图 2.8 – 验证操作系统是否正确写入](img/B16945_Figure_2.8.jpg)

](img/B16945_Figure_2.8.jpg)

图 2.8 – 验证操作系统是否正确写入

1.  拔出你的 MicroSD 卡：

![图 2.9 – 写入过程完成时显示的对话框](img/B16945_Figure_2.9.jpg)

](img/B16945_Figure_2.9.jpg)

图 2.9 – 写入过程完成时显示的对话框

现在你的 MicroSD 卡已经包含了全新的 Ubuntu 安装。接下来的部分，我们将使用这个全新安装的系统安装 K3s。

### 在安装 K3s 主节点或工作节点之前配置 Ubuntu

现在，你的设备已经准备好首次运行。执行以下步骤，将其配置并安装为单节点集群：

1.  打开你的设备。

1.  当 Ubuntu 提示你输入用户名和密码时，输入用户名和密码为 `ubuntu`；这是首次登录的默认密码。

1.  现在，Ubuntu 会要求你更改默认密码。我们使用 `k3s123-` 作为密码。请记住，在实际生产环境中，你必须使用更强的密码。

1.  现在，我们来配置网络。默认情况下，Ubuntu 使用 `init` 云服务配置网络。我们通过以下命令和内容创建 `99-disable-network-config.cfg` 文件来停用此功能：

    ```
    $ sudo nano /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
    ```

以下是文件的内容：

```
network: {config: disabled}
```

1.  如果你执行 `ifconfig` 命令，你会看到设备是 `eth0`。然而，它也可能被命名为 `es3` 或类似名称。我们需要使用以下命令修改 `50-cloud-init` 文件：

    ```
    $ sudo nano /etc/netplan/50-cloud-init.yaml
    ```

1.  接下来，修改文件内容，应该类似如下所示：

    ```
    network:
      version: 2
      renderer: networkd
      ethernets:
        eth0:
          dhcp4: no
          addresses:
            - 192.168.0.11/24
          gateway4: 192.168.0.1
          nameservers:
              addresses: [8.8.8.8, 1.1.1.1]
    ```

注意

请记住，您应该根据需要修改此文件，按照当前网络或互联网连接更改地址、网关和 DNS 服务器。对于这个本地设置，我们使用支持 DHCP 的互联网连接。

1.  现在应用配置，您可以重启设备来检查操作系统启动时是否设置了您的 IP 地址。为此，执行以下命令：

    ```
    $ sudo netplan apply 
    ```

1.  现在，通过以下命令和内容编辑`/boot/firmware/cmdline.txt`文件来配置启动的内核参数：

    ```
    $ sudo nano /boot/firmware/cmdline.txt
    ```

1.  将以下内容添加到行末，以启用在 K3s 集群中使用`containerd`创建容器：

    ```
    cgroup_memory=1 cgroup_enable=memory
    ```

注意

如果您使用的是 Raspbian，此文件位于`/boot/cmdline.txt`。

1.  使用`nano`编辑`/etc/hostname`文件，并为其指定唯一名称，例如，主节点可以是`master`，工作节点可以是`worker-1`、`worker-2`，依此类推：

    ```
    $ sudo nano /etc/hostname
    ```

这是文件的内容：

```
master
```

1.  编辑`/etc/hosts`文件，添加主机名。至少应该有如下所示的一行：

    ```
    $ sudo nano /etc/hosts
    ```

例如，文件内容可能如下所示：

```
127.0.0.1 localhost master
```

1.  现在重启您的设备：

    ```
    $ sudo reboot
    ```

此配置是为您的设备准备配置 K3s 主节点或代理节点所必需的。在下一节中，您将学习如何在您的设备上安装 K3s。

# 创建 K3s 单节点和多节点集群

在本节中，您将学习如何在您的 Ubuntu 操作系统上为 ARM 设备配置 K3s 的主节点和代理节点。为了更好地理解我们正在做的事情，让我们更详细地看看*图 2.10*：

![图 2.10 – K3s 集群配置](img/B16945_Figure_2.10.jpg)

图 2.10 – K3s 集群配置

上图展示了您可以以以下配置安装 K3s 集群：

+   **单节点集群**：在此配置中，您只有一个节点，它同时承担主节点和代理/工作节点的角色。您可以将这种类型的集群用于小型应用程序。但不适合重负载工作，因为它可能会减慢所有组件的速度。请记住，这个节点同时作为主节点和代理节点工作。

+   **多节点集群**：在此配置中，您有一个主节点，它控制代理/工作节点；此配置对于高可用性和重处理任务非常有用。

通过这些简短的描述，您可以想象出创建 K3s 集群所需的配置。在下一节中，您将学习如何创建一个单节点集群。

## 使用 Ubuntu 操作系统创建单节点 K3s 集群

要开始安装 K3s，您应该使用 Ubuntu 作为 K3s 的主要发行版。您可能会问，为什么是 Ubuntu？因为 Ubuntu 有很多预构建的功能，可以节省准备设备时的时间。此外，它支持 32 位和 64 位的 ARM 设备。我推荐使用这个发行版，原因是它的兼容性和支持的软件。那么，让我们开始创建这个单节点的 K3s 集群吧。

要安装 K3s（用于主节点或单节点集群），您必须执行以下步骤：

1.  打开您的设备并登录。

1.  一旦登录，执行以下命令在您的终端中执行 K3s 的基本安装：

    ```
    $ curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--write-kubeconfig-mode 644 --no-deploy traefik --disable traefik" sh -s -
    ```

注意

此命令安装 K3s，默认情况下不包含`traefik`作为 Ingress 控制器，并允许您在不使用`sudo`的情况下执行`kubectl`命令。您可以添加特定的标志以使用 K3s 的特定版本；请参阅官方文档以了解有关此参数的更多信息。您可以在本章末尾找到链接。

1.  （*可选*）如果您希望在 AWS Graviton 2 实例或其他云提供商上安装 K3s，其中公共 IP 未关联到 OS 中的网络接口，则必须使用以下命令设置外部 IP 参数为实例的公共 IP：

    ```
    $ PUBLIC_IP=YOUR_PUBLIC_IP|YOUR_PRIVATE_IP
    $ curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--write-kubeconfig-mode 644 --no-deploy traefik --disable traefik --tls-san "$PUBLIC_IP" --node-external-ip "$PUBLIC_IP"" sh -s -
    ```

1.  （*可选*）如果您想执行简单的测试，请执行以下命令以使用 K3s 的`LoadBalancer`功能公开部署：

    ```
    $ kubectl run nginx --image=nginx --restart=Never
    $ kubectl expose pod/nginx --port=8001 --target-port=80 --type=LoadBalancer
    ```

接下来，通过 K3s 节点的公共或私有 IP 地址访问部署的`nginx`服务的端口`8001`；您可以通过执行以下命令测试访问：

```
$ curl http://YOUR_PUBLIC_OR_PRIVATE_IP:8001
```

或者，如果您有私有 IP，请运行以下命令：

```
$ curl http://YOU_PRIVATE_IP:8001
```

注意

此节点将同时兼作主节点和代理节点。

现在我们已经安装了单节点集群。让我们在下一节中继续添加更多节点到您的新集群。

## 向您的 K3s 集群添加更多节点以进行多节点配置

那么，如果您想向单节点集群添加更多节点呢？要向集群添加更多节点，首先，您必须按照*在您的 MicroSD 卡中安装 Ubuntu*部分的步骤为每个新节点安装 Ubuntu。然后，您可以继续以下步骤：

1.  登录到您的主节点：

    ```
    $ ssh ubuntu@MASTER_PUBLIC_OR_PRIVATE_IP
    ```

1.  使用以下命令从主节点提取加入集群的令牌：

    ```
    $ sudo cat /var/lib/rancher/k3s/server/node-token
    ```

1.  从主节点注销。现在您有了加入额外节点的令牌。

对于每个要加入集群的工作节点，请执行以下步骤（这是更简单的方法）。

1.  登录要添加到集群的工作节点：

    ```
    $ ssh ubuntu@WORKER_PUBLIC_OR_PRIVATE_IP
    ```

1.  使用主节点生成的令牌设置环境变量：

    ```
    $ export TOKEN=YOUR_MASTER_TOKEN
    ```

1.  使用以下命令注册您的节点：

    ```
    $ curl -sfL https://get.k3s.io | sh -s - agent --server https://MASTER_PUBLIC_OR_PRIVATE_IP:6443 --token ${TOKEN} --with-node-id
    ```

注意

如果所有节点的主机名相同，请添加`--with-node-id`选项，K3s 将在主机名末尾添加随机 ID，以确保集群内的节点名称唯一。

1.  退出工作节点：

    ```
    $ exit
    ```

1.  登录主节点：

    ```
    $ ssh ubuntu@MASTER_PUBLIC_OR_PRIVATE_IP
    ```

1.  使用以下命令检查新节点是否正在运行：

    ```
    $ kubectl get nodes
    ```

注意

您需要等待几分钟，直到节点转换为`Ready`状态。

1.  （*可选*）如果您使用的是与 Ubuntu 不同的 GNU/Linux 发行版，以下步骤将更适合像 Alpine Linux 这样的小型发行版。登录您要添加到集群的工作节点：

    ```
    $ ssh ubuntu@WORKER_PUBLIC_OR_PRIVATE_IP
    ```

1.  使用以下命令在工作节点内下载 K3s 的二进制文件：

    ```
    $ curl -sfL https://github.com/k3s-io/k3s/releases/download/v1.21.1%2Bk3s1/k3s-arm64 > k3s > k3s | chmod +x k3s;sudo mv k3s /sbin
    ```

注意

请访问 [`github.com/k3s-io/k3s/releases`](https://github.com/k3s-io/k3s/releases) 下载二进制文件。选择任何方法将此二进制文件放入您的工作节点中。目标是将 K3s 二进制文件下载到工作节点中。请注意，在之前的命令中，选择了版本 `v1.21.2+k3s1`。因此，请根据您的需要修改 URL 以匹配所需的版本。

1.  设置一个包含主节点生成的令牌的环境变量：

    ```
    $ export TOKEN=YOUR_MASTER_TOKEN
    $ sudo k3s agent --server https://myserver:6443 
      --token ${TOKEN} --with-node-id &
    ```

1.  从工作节点退出：

    ```
    $ exit
    ```

1.  登录到您的主节点：

    ```
    $ ssh ubuntu@MASTER_IP
    ```

如果您想设置节点的角色，请执行以下步骤。

1.  （*可选*）使用以下命令设置新工作节点的角色：

    ```
    $ kubectl label nodes node_name kubernetes.io/role=worker
    ```

1.  从主节点退出：

    ```
    $ exit
    ```

现在您拥有一个多节点的 K3s 集群，准备好使用了。在下一部分中，您将学习如何使用 `kubectl` 命令管理您的集群。

## 提取 K3s kubeconfig 以访问您的集群

现在，使用 `kubectl` 命令配置从计算机访问您的 K3s 集群。要配置从外部连接到新 K3s 集群，请执行以下步骤：

1.  通过运行以下命令安装 `kubectl` 命令（适用于 Mac 安装）：

    ```
    $ curl -LO https://dl.k8s.io/release/v1.22.0/bin/darwin/amd64/kubectl
    $ chmod +x ./kubectl
    $ sudo mv ./kubectl /usr/local/bin/kubectl
    $ sudo chown root: /usr/local/bin/kubectl
    ```

或者，如果您使用的是 Linux，请运行以下命令：

```
$ curl -LO "https://dl.k8s.io/release/v1.22.0/bin/linux/amd64/kubectl"
$ sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

1.  从主节点，将 `/etc/rancher/k3s/k3s.yaml` 文件中的内容复制到本地的 `~/.kube/` 配置文件中

1.  获取服务器值的以下部分：

    ```
    server: https://127.0.0.1:6443
    ```

并将其更改为以下内容：

```
server: https://MASTER_IP:6443
```

1.  使用以下命令更改此文件的权限：

    ```
    $ chmod 0400 ~/.kube/config
    ```

1.  接下来，使用以下命令测试您是否可以访问集群：

    ```
    $ kubectl get nodes
    ```

此命令返回集群节点及其状态的列表。

注意

在将 Rancher `kubeconfig` 文件复制到您的计算机之前，请记得安装 `kubectl` 命令行工具。请记住，`k3s.yaml` 文件的内容必须存储在 `~/.kube/config` 中，并且需要设置为 `0400` 权限。要了解如何安装 `kubectl` 命令，请访问 [`kubernetes.io/docs/tasks/tools/install-kubectl-macos`](https://kubernetes.io/docs/tasks/tools/install-kubectl-macos)。

现在您已经准备好执行更多高级配置来创建新的 K3s 集群。让我们继续下一部分，了解更多内容。

# 高级配置

现在是时候探索可以用来配置边缘 K3s 集群的更多高级配置了。

## 使用外部 MySQL 存储 K3s

K3s 支持 MySQL 和 SQLite，而不是 `etcd`，作为 K3s 集群信息的数据存储。您可以在其他节点、云实例，或像 AWS Aurora 或 Google Cloud SQL 这样的云托管服务中安装 MySQL。例如，让我们尝试在使用 DigitalOcean 的云实例上进行操作。但是，您可以在任何您选择的云中执行此操作。现在，按照以下步骤开始：

1.  登录到您的云实例：

    ```
    $ ssh root@IP_DATASTORE
    ```

1.  使用以下命令安装 Docker：

    ```
    $ apt-get update
    $ apt-get install docker.io -y
    $ docker run -d --name mysql -e MYSQL_ROOT_PASSWORD=k3s123- \
    -e MYSQL_DATABASE="k8s" -e MYSQL_USER="k3sadm" \
    -e MYSQL_PASSWORD="k3s456-" \
    -p 3306:3306 \
    -v /opt/mysql:/var/lib/mysql \
    mysql:5.7
    ```

1.  使用以下命令登出：

    ```
    $ exit
    ```

1.  在您的主节点上执行以下操作：

    ```
    $ curl -sfL https://get.k3s.io | K3S_DATASTORE_ENDPOINT="mysql://k3sadm:k3s456-@tcp(YOUR_CLOUD_INSTANCE_IP:3306)/k8s" INSTALL_K3S_EXEC="--write-kubeconfig-mode 644 --no-deploy traefik --disable traefik" sh -s -
    ```

注意

这将使用你云实例中的 MySQL 安装。你必须将 `YOUR_CLOUD_INSTANCE_IP` 替换为你云实例的 IP 地址。

1.  使用以下命令从主节点提取令牌以加入集群：

    ```
    $ sudo cat /var/lib/rancher/k3s/server/node-token
    ```

1.  从主节点注销：

    ```
    $ exit
    ```

对每个工作节点，执行下一步操作。

1.  安装代理以注册并准备工作节点：

    ```
    curl -sfL https://get.k3s.io | K3S_TOKEN=MASTER_TOKEN sh -s - agent --server https://MASTER_IP:6443
    ```

注意

你可以执行 `kubectl get nodes` 来检查你的工作节点是否已经添加并处于 `Ready` 状态。

现在，你可以使用外部数据存储来使用集群，而不是 `etcd` 或 SQLite。在这种情况下，我们使用一个混合解决方案，利用本地实例和公共实例来通过 MySQL 存储 K3s 配置。记住，你也可以使用 MariaDB 或你最喜欢的云服务提供商的其他 MySQL 托管服务。你可以将多个配置为主节点的节点添加到集群中，以提高集群中 Kubernetes API 等核心组件的可用性。

## 安装 Helm 以在 Kubernetes 中安装软件包

Helm 是 Kubernetes 的包管理器。使用 Helm，你可以通过一个称为 Helm Charts 的包定义将软件安装到 Kubernetes 集群中。你可以使用公共 Helm Chart 仓库或自己的仓库来安装包。要在 Linux 或 Mac 上安装 Helm，执行以下步骤：

1.  在 Linux 上安装 Helm，运行以下命令：

    ```
    $ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
    $ chmod 700 get_helm.sh
    $ ./get_helm.sh
    ```

1.  在 Mac 上安装 Helm，运行以下命令：

    ```
    $ brew install helm
    ```

1.  要开始安装 Helm Charts，你需要通过在 Linux 或 Mac 上运行以下命令来添加一个 chart 仓库：

    ```
    $ helm repo add bitnami https://charts.bitnami.com/bitnami
    ```

现在，让我们来看一下如何更改默认的 Ingress 控制器。

## 更改默认的 Ingress 控制器

在开始本节之前，我们先定义一下什么是 Ingress，然后再定义 Ingress 控制器。根据官方 Kubernetes 网站，Ingress 是一个 Kubernetes 组件，用于暴露匹配集群内服务的 HTTP 或 HTTPS 路由。Service 是 Kubernetes 用来暴露应用程序为网络服务的抽象方式。而 Ingress 控制器是一个负责执行 Ingress 的组件，包括可能配置边缘路由器或代理的负载均衡器。有许多基于不同边缘路由器或代理的 Ingress 控制器实现，如 Traefik、Envoy、Nginx 等。默认情况下，K3s 包含 Traefik 1.0 版本，提供了最基本的功能，可以在不消耗大量资源的情况下路由你的服务。

如果你想使用不同的 Ingress 控制器而不是默认选项（**Traefik**），请使用以下命令安装主节点：

1.  使用以下参数安装主节点：

    ```
    $ curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--write-kubeconfig-mode 644 --no-deploy traefik --disable traefik" sh -s -
    ```

1.  然后，使用以下命令创建命名空间并安装 `nginx` Ingress 控制器：

    ```
    $ kubectl create ns nginx-ingress
    ```

1.  添加 Helm Charts 仓库：

    ```
    $ helm repo add ingress-nginx \
    https://kubernetes.github.io/ingress-nginx 
    ```

1.  更新你的仓库以获取最新版本：

    ```
    $ helm repo update 
    ```

1.  安装 Ingress 控制器：

    ```
    $ helm install nginx-ingress \
      ingress-nginx/ingress-nginx \
      -n nginx-ingress
    ```

(*可选*) 如果你想测试`nginx-ingress`控制器是否工作，请按照接下来的步骤进行操作。

1.  使用`nginx`镜像创建一个部署：

    ```
    $ kubectl create deployment nginx --image=nginx
    ```

1.  使用`ClusterIP`暴露部署：

    ```
    $ kubectl expose deployment/nginx --port=8001 \
      --target-port=80 --type=ClusterIP --name=nginx-srv
    ```

1.  使用以下命令创建`my-ingress.yaml`文件：

    ```
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: my-ingress
      annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /
    spec:
      rules:
      - http:
          paths:
          - path: /mypath
            pathType: Prefix
            backend:
              service:
                name: nginx-srv
                port:
                  number: 8001
    ```

1.  使用以下命令创建 Ingress：

    ```
    $ kubectl create -f my-ingress.yaml
    ```

1.  现在使用以下命令测试它是否有效：

    ```
    $ curl http://LB_IP/my-path
    ```

注意

你必须将`LB_IP`的值替换为由 NGINX Ingress 控制器安装创建的`LoadBalancer`服务的 IP 地址。在这种情况下，它与主节点的 IP 地址相同。

1.  要检查`nginx-ingress`暴露的 IP，请执行以下命令：

    ```
    $ kubectl get services -n nginx-ingress
    ```

注意

请考虑到 K3s 在使用 Kubernetes 服务时有其独特的行为。要了解更多信息，请参阅 https://rancher.com/docs/k3s/latest/en/networking。

现在你已经理解了如何安装 Ingress 控制器以及如何使用它，是时候学习如何在必要时从节点卸载 K3s 了。

## 从主节点或代理节点卸载 K3s

如果你想卸载主节点或代理节点上的 K3s，必须执行 K3s 安装提供的卸载脚本。那么，我们开始吧。

### 从代理节点卸载 K3s

要从代理节点（即工作节点）卸载 K3s，请执行以下步骤：

1.  登录到你的代理节点：

    ```
    $ ssh ubuntu@AGENT_NODE_IP
    ```

1.  卸载代理守护进程并删除此节点上创建的所有容器：

    ```
    $ k3s-agent-uninstall.sh
    $ sudo rm -R /etc/rancher
    $ sudo rm -R /var/lib/rancher
    ```

1.  从代理节点注销：

    ```
    $ exit
    ```

### 从主节点卸载 K3s

要从主节点卸载 K3s，请执行以下步骤：

1.  登录到你的代理节点：

    ```
    $ ssh ubuntu@MASTER_NODE_IP
    ```

1.  卸载代理守护进程并删除此节点上创建的所有容器：

    ```
    $ k3s-uninstall.sh
    $ sudo rm -R /etc/rancher
    $ sudo rm -R /var/lib/rancher
    ```

1.  从代理节点注销：

    ```
    $ exit
    ```

所以，你已经学会了如何卸载 K3s，这在你想在设备上尝试新配置时会很有用。接下来，让我们继续学习如何在下一节中排查集群问题。

# 排查 K3s 集群问题

本节包括一些基本的故障排除命令，你可以使用它们来测试你的集群。有不同的故障排除选项：

1.  如果你想查看节点的状态并检查 Kubernetes 是否运行，请执行以下命令：

    ```
    $ kubectl get nodes
    ```

1.  创建一个 Pod 来检查你的集群是否能调度 Pod：

    ```
    $ kubectl run nginx --image=nginx --restart=Never
    ```

1.  创建一个 Service 来暴露之前创建的 Pod，并测试`LoadBalancer`服务是否有效：

    ```
    $ kubectl expose pod/nginx --port=8001 \
      --target-port=80 \
      --type=LoadBalancer
    ```

1.  如果你想检查服务和端口是否有效并暴露你的服务，可以执行以下命令，这些服务可以是`LoadBalancer`或`NodePort`：

    ```
    $ kubectl get services
    ```

1.  如果你想实时查看系统日志，请执行以下命令：

    ```
    $ journalctl -f
    ```

1.  执行以下命令检查`k3s`服务是否在你的主节点上运行。此命令必须在代理节点内执行：

    ```
    $ systemctl status k3s
    ```

1.  执行以下命令检查`k3s-agent`服务是否在你的代理/工作节点上运行。此命令必须在代理节点内执行：

    ```
    $ systemctl status k3s-agent
    ```

注意

关于 K3s 的不同选项和配置的更多详细信息，请访问 [`rancher.com/docs/k3s/latest/en`](https://rancher.com/docs/k3s/latest/en)。

# 概述

本章涵盖了使用 K3s 的边缘分发版创建和自定义 Kubernetes 集群的第一步。还介绍了高级配置，例如如何为 K3s 配置外部数据存储，以帮助你为边缘 K3s 集群配置更强大且高可用的解决方案。在本章末尾，我们还介绍了一些高级配置，如如何安装不同的 Ingress 控制器、使用 Helm Chart 操作符以及集群的基本故障排除命令。有了这些知识，我们现在可以跳到下一章，了解 k3OS 在快速且轻松地安装 K3s 方面的优势。

# 问题

这里有一些问题可以验证你在本章中学到的内容：

+   我可以使用什么软件来准备我的 ARM 设备以安装 K3s？

+   如何在 ARM 设备上使用 K3s 安装一个基本的多节点集群？

+   如何安装不同的 Ingress 控制器？

+   如何使用 Helm 在我的集群中安装软件包？

+   如何对我的集群进行故障排除？

# 进一步阅读

你可以参考以下资料，获取更多本章所涉及主题的信息：

+   Raspberry Imager 软件：[`www.raspberrypi.org/software`](https://www.raspberrypi.org/software)

+   Ubuntu 网络配置：[`linuxize.com/post/how-to-configure-static-ip-address-on-ubuntu-20-04/#configuring-static-ip-address-on-ubuntu-server`](https://linuxize.com/post/how-to-configure-static-ip-address-on-ubuntu-20-04/#configuring-static-ip-address-on-ubuntu-server)

+   K3s 官方文档：[`rancher.com/docs/k3s/latest/en`](https://rancher.com/docs/k3s/latest/en)

+   K3s 安装选项：[`rancher.com/docs/k3s/latest/en/installation/install-options`](https://rancher.com/docs/k3s/latest/en/installation/install-options)

+   K3s 网络配置：[`rancher.com/docs/k3s/latest/en/networking`](https://rancher.com/docs/k3s/latest/en/networking)

+   Helm 官网：[`helm.sh`](https://helm.sh)

+   K3s Helm Chart 操作符：[`rancher.com/docs/k3s/latest/en/helm`](https://rancher.com/docs/k3s/latest/en/helm)

+   查找你想安装的软件的 Helm Charts Hub：[`artifacthub.io`](https://artifacthub.io)

+   官方 Kubernetes 文档：[`kubernetes.io/docs`](https://kubernetes.io/docs)
