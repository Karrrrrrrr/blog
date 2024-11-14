---
title: "Docker基础组件实现原理"
date: 2024-11-14T14:00:00+08:00

categories: ["Docker"]

tags: ["Docker"]
---

# Docker基础组件实现原理



## 下载镜像
```shell    
docker pull xxx
```

### 原理
使用```http.Client```下载tar.gz文件, 然后解压到某个目录

优化点: hash相同的层只会解压一次

## 上传镜像
```shell
docker push xxx 
```
### 原理
把文件夹打包为镜像, 然后使用```http.Client```上传

## 提交镜像
```shell
docker commit 
```
把文件夹多个分层归档为一个文件, 然后压缩, 记录每个层的元数据(比如hash值)

## 删除镜像
```shell
docker rmi 
```
### 原理
调用删除API, 对每个层的占用数量-1, 然后删除占用数量为0的层

## 删除容器
```shell
docker rm 
```
### 原理
调用删除API, 删除容器单独创建的层
## 创建网络
```shell
docker network create 
```

执行 ```docker network create``` 命令时，Docker 会创建一个虚拟网络（通常是一个虚拟桥接网络），并为其配置虚拟网卡。不同的网络驱动（如 bridge、host、overlay 等）会根据不同的需求创建不同类型的虚拟网络和网络设备。

具体来说：

> Bridge 网络（默认网络）：Docker 创建一个虚拟的桥接设备（如 docker0），并为每个容器创建一个虚拟网络接口（如 veth）。这些虚拟网络接口通过桥接设备相连。

> Overlay 网络：Docker 使用 VXLAN 技术，在不同 Docker 主机间创建一个跨主机的虚拟网络。

> Host 网络：容器共享宿主机的网络栈，不进行虚拟网卡的创建。

> None 网络：容器没有网络，完全隔离。

底层创建虚拟网络的步骤
虚拟网卡：创建一个虚拟网卡（如 veth，即虚拟以太网设备），它将连接到宿主机的虚拟网桥（如 docker0）。每个容器会有一个虚拟网卡，它通过这个虚拟网卡与宿主机和其他容器进行通信。

虚拟网桥（Bridge）：如果是 bridge 网络驱动，Docker 会在宿主机上创建一个虚拟网桥（通常是 docker0），然后将容器的虚拟网卡连接到这个网桥上，从而实现容器与宿主机之间的通信。

路由和 NAT：Docker 会配置 iptables 规则来管理容器和宿主机之间的流量转发，确保容器可以访问外部网络，同时也确保外部网络能够通过端口映射访问容器中的服务。

如何自己实现类似 Docker 网络功能
如果想自己实现类似于 Docker 网络的功能（创建虚拟网络、虚拟网卡以及连接容器等），可以按以下步骤进行：

1. 创建虚拟网卡

    在 Linux 上，可以使用 ip 命令或者通过编程接口来创建 虚拟以太网设备（veth）。veth 是一对虚拟网卡，类似于 docker 创建的容器网络接口。
    
    使用 ip link add 命令创建虚拟网卡：
    
    ```bash    
    ip link add veth0 type veth peer name veth1
    ```
    这条命令会创建一对虚拟网卡 veth0 和 veth1，veth0 会连接到容器的网络，而 veth1 需要连接到宿主机的虚拟网桥。

2. 创建虚拟网桥（Bridge）

    使用 brctl 或者 ip 命令来创建一个虚拟网桥（如 br0）并将虚拟网卡连接到该网桥。
    
    创建网桥：
    ```bash
    ip link add name br0 type bridge
    ip link set br0 up
    ```
    将 veth1 接入网桥：
    
    ```bash
    ip link set veth1 master br0
    ```
    这样，veth1 就连接到了网桥 br0，容器就可以通过 veth0 和 veth1 进行通信。

3. 配置网络地址和路由
    一旦虚拟网卡和网桥创建完成，下一步是给容器分配 IP 地址，并确保容器和宿主机之间的通信。可以使用 ip addr 和 ip route 命令来配置 IP 地址和路由。
    
    配置 veth0 的 IP 地址：
    
    ```bash
    ip addr add 172.18.0.2/16 dev veth0
    ip link set veth0 up
    ```
    
    配置容器网络和宿主机的网络路由：
    
    如果希望容器能够访问外部网络，需要通过 NAT（Network Address Translation）来转发流量。可以使用 iptables 或 nftables 来进行配置。Docker 通常会使用 iptables 创建 NAT 规则。

4. 配置 iptables 转发规则

    配置 iptables 规则来确保宿主机和容器之间可以进行流量转发。Docker 会自动在宿主机上配置相关的 NAT 规则，使得容器能够访问外部网络，并且宿主机的端口可以映射到容器。例如，配置 NAT 转发：
    
    ```bash
    iptables -t nat -A POSTROUTING -s 172.18.0.0/16 ! -d 172.18.0.0/16 -j MASQUERADE
    iptables -A FORWARD -s 172.18.0.0/16 -j ACCEPT
    iptables -A FORWARD -d 172.18.0.0/16 -j ACCEPT
    ```
    这段规则确保了宿主机上的流量能够正确地转发到容器的网络，同时容器也可以访问外部网络。

5. 容器间通信

    如果希望多个容器之间进行通信，可以使用类似的方法将每个容器的虚拟网卡连接到相同的虚拟网桥（如 br0）。这将允许容器之间互相访问。

6. 容器连接网络

    如果希望实现容器动态加入网络的功能，可以编写一个管理工具来控制容器的网络连接。比如通过接口管理容器的 veth 网络接口，动态配置 ip addr 和 iptables 规则。

完整步骤概述
1. 使用 ip link 或 brctl 创建虚拟网卡和网桥。
2. 配置虚拟网卡（如 veth）的 IP 地址。
3. 配置网桥和容器之间的网络连接。
4. 配置路由和 NAT 规则，使容器能够访问外部网络。
5. 如果需要容器间通信，确保它们连接到相同的虚拟网桥。
6. 配置和管理 iptables 或 nftables 规则，确保网络流量转发。



## 端口映射
Docker 的端口映射功能是通过 iptables 来实现的。Docker 使用 iptables 进行 网络地址转换 (NAT)，以便容器的端口能够映射到宿主机的端口上，允许外部流量通过宿主机的 IP 地址和端口访问容器。

端口映射实现过程
当运行一个 Docker 容器并指定端口映射（例如：```docker run -p 8080:80```），Docker 会使用 iptables 配置一组 NAT 规则，使宿主机的 8080 端口的流量转发到容器的 80 端口。这是通过修改 NAT 表 和 filter 表 来实现的。

具体来说，Docker 会创建以下几种 iptables 规则：

NAT 表：用于修改数据包的目标地址或源地址。
filter 表：用于控制数据包是否被允许进出网络接口。

端口映射的步骤

创建 NAT 规则： Docker 在宿主机上使用 iptables 配置 PREROUTING 和 POSTROUTING 链上的 NAT 规则。当外部流量到达宿主机的端口时，这些规则会将流量转发到正确的容器上。

PREROUTING 链： 当流量到达宿主机时，PREROUTING 链会修改流量的目标地址，将宿主机端口（如 8080）的流量重定向到容器的 IP 地址和端口（如容器的 172.17.0.2:80）。

例如，使用 ```docker run -p 8080:80``` 启动容器后，iptables 会添加类似以下规则：

```bash
-A DOCKER -p tcp --dport 8080 -j DNAT --to-destination 172.17.0.2:80
```
这条规则的作用是：将访问宿主机的 TCP 8080 端口的流量重定向到容器 IP 172.17.0.2 的 80 端口。

POSTROUTING 链： 在容器内部响应外部请求时，POSTROUTING 链会通过源地址转换（Source NAT）将容器的响应数据包的源 IP 地址修改为宿主机的 IP 地址。

例如，Docker 会添加类似以下规则：

```bash
-A POSTROUTING -s 172.17.0.2 -o eth0 -j MASQUERADE
```
这条规则会确保从容器 172.17.0.2 发出的响应流量，源 IP 地址会被改成宿主机的 IP 地址，以便外部主机可以正确地响应。

过滤规则： iptables 还会在 filter 表中设置规则，确保只有宿主机允许的流量可以进入容器。Docker 默认会配置 INPUT、FORWARD 和 OUTPUT 链上的规则来控制容器网络流量的进出。

例如，Docker 会添加如下规则，允许宿主机与容器之间的流量转发：

```bash
-A FORWARD -d 172.17.0.2 -p tcp --dport 80 -j ACCEPT
-A FORWARD -s 172.17.0.2 -p tcp --sport 80 -j ACCEPT
```
端口映射的工作原理总结\
宿主机端口映射到容器端口：通过 iptables 在宿主机的 PREROUTING 链中设置 DNAT 规则，确保外部请求到达宿主机指定端口时，流量会转发到正确的容器端口。\
容器端口返回外部：通过 POSTROUTING 链中的 SNAT 规则，确保容器的响应流量会使用宿主机的 IP 地址作为源地址，从而确保外部能够接收到容器的响应。\
流量过滤：通过 FORWARD 链等规则，控制哪些流量能够通过宿主机转发到容器。\
查看 Docker 设置的 iptables 规则\
使用 iptables 命令查看 Docker 设置的规则，特别是 DOCKER 链，它包含了与容器相关的所有规则。

```bash
sudo iptables -t nat -L -n -v  ## 查看 NAT 表中的规则
sudo iptables -L -n -v         ## 查看 filter 表中的规则
```
Docker 会在 DOCKER 链下自动添加规则来管理端口映射。例如：
```text
Chain DOCKER (1 references)
 pkts bytes target     prot opt in     out     source               destination
  243 12872 ACCEPT     all  --  *      *       172.17.0.2           0.0.0.0/0
  30  1680 DNAT       tcp  --  *      *       0.0.0.0/0            172.17.0.2         tcp dpt:8080 to:172.17.0.2:80
```
Docker 的端口映射功能依赖于 Linux 系统的 iptables 来实现，通过配置 NAT 规则和路由规则，Docker 将宿主机的端口与容器的端口之间的流量进行转发。容器的流量会通过 PREROUTING 和 POSTROUTING 链进行 NAT 转换，确保容器可以通过宿主机的 IP 地址与外部通信。

## 资源隔离
以sh命令为例, 使用如下代码可以实现资源隔离
```shell
unshare --pid --fork --net --mount --uts --ipc --user sh 
```

每个参数的作用如下：

1. --pid
    
    这个选项会让新进程进入一个新的 PID（进程标识符）命名空间。 
这意味着新进程和宿主机上的其他进程在 PID 上是隔离的。
也就是说，新的进程会有自己的进程树，并且不能看到宿主机或其他命名空间中的进程。

2. --fork

    这个选项会在创建新的命名空间时使 unshare 命令自己创建一个新进程。
没有这个选项，unshare 只是修改当前进程的命名空间，而不是创建一个新进程。通常与其他命名空间隔离选项一起使用。

3. --net

    创建一个新的 网络命名空间。这会让新进程拥有独立的网络接口、路由表和防火墙等。也就是说，新的进程不能访问宿主系统的网络，且它的网络设置是与其他进程隔离的。

4. --mount

    创建一个新的 挂载命名空间。新的进程会拥有自己的文件系统挂载点，无法看到宿主机的挂载点。这个命名空间隔离了文件系统的视图，新的进程可以对其文件系统进行操作，而不影响宿主系统的文件系统。

5. --uts

    创建一个新的 UTS（Unix Timesharing System）命名空间。
这会让新进程拥有独立的主机名和域名。新的进程可以设置自己的主机名，而不会影响宿主系统或其他命名空间中的进程。

6. --ipc

    创建一个新的 IPC（进程间通信）命名空间。
这会让新进程与宿主系统或其他进程间的 IPC 机制（如信号量、消息队列、共享内存）隔离开。新的进程会有自己的 IPC 资源。

7. --user

    创建一个新的 用户命名空间。
在这个命名空间中，进程的 UID 和 GID（用户和组标识符）是与宿主系统隔离的。
也就是说，进程可以拥有与宿主系统不同的权限映射，允许非特权用户以根用户身份运行进程，但在宿主系统中并没有实际的 root 权限。

8. sh
    
    这是最终要执行的命令。在这种情况下，是启动一个新的 sh（shell）进程。因为使用了 unshare 创建了多个命名空间，启动的 shell 进程将在这些新的隔离环境中运行。


### overlay
Docker 使用overlay文件系统(老版本是aufs), 使用以下代码可以模拟一个分层文件系统, 这样容器在写入数据之后不影响镜像
```shell
mkdir overlay
mkdir overlay/mnt
mkdir overlay/container-layer

cd overlay
echo "hello world" > container-layer/container-layer.txt
mkdir ./{image-layer1,image-layer2,image-layer3}
echo "I am image layer 1" > ./image-layer1/image-layer1.txt
echo "I am image layer 2" > ./image-layer2/image-layer2.txt
echo "I am image layer 3" > ./image-layer3/image-layer3.txt
sudo mount -t overlay overlay -o lowerdir=./image-layer1:./image-layer2:./image-layer3,upperdir=./container-layer,workdir=./mnt ./mnt
```



我们把上面的 image-layer称为基础层, container-layer称为容器层
主要的原理如下, 如果基础层里有的数据 容器层没有, 那么以基础层为主, 否则以挂载层为主

实验之后不要忘记卸载文件系统
```shell
sudo umount ./mnt 
```