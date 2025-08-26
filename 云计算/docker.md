# 一、docker简介

## 一、什么是docker

Docker 是一种“轻量级”容器技术，是一个**容器引擎，它使用 Linux 内核功能**（如命名空间和控制组）在操作系统上创建容器。因此，可以将其称为操作系统级虚拟化。



### docker与虚拟机的区别

| **虚拟化**                                      | **容器**                                               |
| ----------------------------------------------- | ------------------------------------------------------ |
| 隔离性强，有独立的GUEST OS                      | 共享内核和OS，隔离性弱!                                |
| 虚拟化性能差(>15%)                              | 计算/存储无损耗，无Guest OS内存开销(~200M)             |
| 虚拟机镜像庞大(十几G~几十G), 且实例化时不能共享 | Docker容器镜象200~300M，且公共基础镜象实例化时可以共享 |
| 虚拟机镜象缺乏统一标准                          | Docker提供了容器应用镜象事实标准，OCI推动进一 步标准化 |
| 虚拟机创建慢(>2分钟)                            | 秒级创建(<10s)相当于建立索引                           |
| 虚拟机启动慢(>30s) 读文件逐个加载               | 秒级(<1s,不含应用本身启动)                             |
| 资源虚拟化粒度低，单机10~100虚拟机              | 单机支持1000+容器密度很高，适合大规模的部署            |

- 资源利用率更高：一台物理机可以运行数百个容器，但一般只能运行数十个虚拟机
- 开销更小：不需要启动单独的虚拟机占用硬件资源
- 启动速度更快：可以在数秒内完成启动



## 二、Docker的组成

Docker主机 host：一个物理机或者虚拟机，用于运行docker服务进程和容器

Docker服务端 Server：Docker守护进程，运行docker容器

Docker客户端 client：客户端使用docker命令或其他工具调用docker api

Docker仓库 registry：保存镜像的仓库，类似于git或svn这样的版本控制器

Docker镜像 images：镜像可以理解为创建实例使用的模板

Docker容器 container：容器是从镜像生成对外提供服务的一个或一组服务



docker三大核心组件：镜像，仓库，容器

镜像：Docker镜像就是一个Linux的文件系统（Root FileSystem），这个文件系统里面包含可以运行在Linux内核的程序以及相应的数据。

特征：镜像是分层（Layer）的：即一个镜像可以多个中间层组成，多个镜像可以共享同一中间层，我们也可以通过在镜像添加多一层来生成一个新的镜像。

镜像是只读的（read-only）：镜像在构建完成之后，便不可以再修改，而上面我们所说的添加一层构建新的镜像，这中间实际是通过创建一个临时的容器，在容器上增加或删除文件，从而形成新的镜像，因为容器是可以动态改变的。

容器：镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。容器却是可读可写的，因为容器是在镜像上面添一层读写层（writer/read layer）来实现的。

容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的 命名空间。因 此容器可以拥有自己的 root 文件系统、自己的网络配置、自己的进程空间，甚至自己的用户 ID 空间。 容器内的进程是运行在一个隔离的环境里，使用起来，就好像是在一个独立于宿主的系统下操作一样。 这种特性使得容器封装的应用比直接在宿主运行更加安全。
仓库：仓库是集中存储镜像的地方。通常，一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本。我们可以指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 latest 作为默认标签。

Docker Hub，是Docker官方提供的一个仓库服务器

三个核心原理：namesapce cgroup unionFS



## 三、Docker安装及基础命令

- 安装docker-ce以及客户端

```bash
centos7命令：
[root@docker-server ~]# yum install wget.x86_64 -y
[root@docker-server ~]# rm -rf /etc/yum.repos.d/*
[root@docker-server ~]# wget -O /etc/yum.repos.d/Centos-7.repo http://mirrors.aliyun.com/repo/Centos-7.repo
[root@docker-server ~]# wget -O /etc/yum.repos.d/epel-7.repo http://mirrors.aliyun.com/repo/epel-7.repo
[root@docker-server ~]# wget -O /etc/yum.repos.d/docker-ce.repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
[root@docker-server ~]# yum install docker-ce -y
rockylinux9命令：
[root@localhost ~]# yum install -y yum-utils
[root@localhost ~]# yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
[root@localhost ~]# sed -i 's+download.docker.com+mirrors.aliyun.com/docker-ce+' /etc/yum.repos.d/docker-ce.repo
[root@localhost ~]# yum install docker-ce -y
```

- 启动docker

```bash
[root@docker-server ~]# systemctl enable docker.service 
Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
[root@docker-server ~]# systemctl start docker.service
```

- 使用国内镜像加速器

```bash
[root@localhost ~]# mkdir -p /etc/docker
[root@localhost ~]# vim /etc/docker/daemon.json
{
    "registry-mirrors": ["https://docker.m.daocloud.io"]
}
或者
{
    "registry-mirrors": [
        "https://docker.1panel.live",
        "https://hub.uuuadc.top",
        "https://docker.anyhub.us.kg",
        "https://dockerhub.jobcher.com",
        "https://dockerhub.icu",
        "https://docker.ckyl.me",
        "https://docker.awsl9527.cn"
    ]
}
# 重启容器服务
[root@localhost ~]# systemctl daemon-reload
[root@localhost ~]# systemctl restart docker

# 可选加速地址：
1、https://docker.m.daocloud.io 
2、https://dockerpull.com 
3、https://atomhub.openatom.cn 
4、https://docker.1panel.live 
5、https://dockerhub.jobcher.com 
6、https://hub.rat.dev 
7、https://docker.registry.cyou 
8、https://docker.awsl9527.cn 
9、https://do.nark.eu.org/ 
10、https://docker.ckyl.me 
11、https://hub.uuuadc.top
12、https://docker.chenby.cn
13、https://docker.ckyl.me
```

- 快速开始

```bash
[root@docker-server ~]# docker pull nginx
[root@docker-server ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED      SIZE
nginx        latest    d1a364dc548d   5 days ago   133MB
[root@docker-server ~]# docker run -d -p 80:80 nginx
e617ca1db9a5d242e6b4145b9cd3dff9f7955c6ab1bf160f13fb6bec081a29e4
[root@docker-server ~]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                               NAMES
e617ca1db9a5   nginx     "/docker-entrypoint.…"   6 seconds ago   Up 5 seconds   0.0.0.0:80->80/tcp, :::80->80/tcp   intelligent_turing
[root@docker-server ~]# docker exec -it e617ca1db9a5 bash
root@e617ca1db9a5:/# cd /usr/share/nginx/html/
root@e617ca1db9a5:/usr/share/nginx/html# ls
50x.html  index.html
root@e617ca1db9a5:/usr/share/nginx/html# echo 'docker nginx test' > index.html 
[root@docker-server ~]# curl 192.168.88.10
docker nginx test

[root@admin ~]# docker ps
CONTAINER ID   IMAGE     COMMAND                   CREATED                                                        NAMES
e0a818c40b7e   nginx     "/docker-entrypoint.…"   About an hour ago  0.0.0.0:9000->9000/tcp, :::9000->9000/tcp   determined_sanderson
9b066ef4bcd2   nginx     "/docker-entrypoint.…"   About an hour ago  90->80/tcp, :::90->80/tcp                   vigorous_hypatia
[root@admin ~]# docker stop e0a818c40b7e
e0a818c40b7e
[root@admin ~]# docker stop 9b066ef4bcd2
9b066ef4bcd2
```

### 参数说明

```bash
-d, --detach: 以守护进程方式运行容器
-p, --publish: 映射容器端口到宿主机端口
格式: -p [hostPort]:[containerPort]
-P（大写）：随机端口映射
-v, --volume: 挂载数据卷
格式: -v [hostPath]:[containerPath]
-e, --env: 设置环境变量
--name: 为容器指定名称
--network: 指定容器所属网络
--restart: 容器退出时的重启策略
可选值: no, on-failure, unless-stopped, always
-i, --interactive: 保持标准输入打开
-t, --tty: 分配一个伪终端
-u, --user: 指定运行容器的用户
--entrypoint: 覆盖容器的默认入口点
--rm: 容器退出后自动删除
--hostname: 设置容器主机名
--add-host: 添加自定义主机名到 IP 的映射
--link: 添加到另一个容器的链接
--expose: 暴露容器端口
--volume-driver: 指定数据卷驱动程序
--cpu-shares: 设置 CPU 权重
--memory: 设置容器内存限制
```



## 四、Linux namespace技术

Namespace是Linux内核的一项功能，用于 隔离系统资源，使得不同Namespace中的进程拥有独立的：进程树（PID Namespace）、网络接口（Network Namespace）、文件系统挂载点（Mount Namespace）、用户和用户组（User Namespace）。但是容器技术是在一个进程内实现运行指定服务的运行环境，并且还可以保护宿主机内核不受其他进程的干扰和影响，如文件系统、网络空间、进程空间等，目前主要通过以下技术实现容器运行空间的相互隔离：

| 隔离类型                                     | 功能                               | 系统调用参数  | 内核   |
| -------------------------------------------- | ---------------------------------- | ------------- | ------ |
| MNT Namespace（mount）                       | 提供磁盘挂载点和文件系统的隔离能力 | CLONE_NEWNS   | 2.4.19 |
| IPC Namespace（Inter-Process Communication） | 提供进程间通信的隔离能力           | CLONE_NEWIPC  | 2.6.19 |
| UTS Namespace（UNIX Timesharing System）     | 提供主机名隔离能力                 | CLONE_NEWUTS  | 2.6.19 |
| PID Namespace（Process Identification）      | 提供进程隔离能力                   | CLONE_NEWPID  | 2.6.24 |
| Net Namespace（network）                     | 提供网络隔离能力                   | CLONE_NEWNET  | 2.6.29 |
| User Namespace（user）                       | 提供用户隔离能力                   | CLONE_NEWUSER | 3.8    |



### namespace创建流程

- **clone()**系统调用：Docker通过clone()（而非fork()）创建新进程，并传入CLONE_NEW*标志（CLONE_NEWPID）
- **内核分配新Namespace**：Linux内核为进程分配独立的资源视图
- **容器进程运行在隔离环境**：进程只能看到当前Namespace内的资源



### MNT Namespace

每个容器都要有独立的根文件系统有独立的用户空间，以实现容器里面启动服务并且使用容器的运行环境。

- 启动三个容器

```bash
[root@docker-server ~]# docker run -d --name nginx-1 -p 80:80 nginx
0e72f06bba417073d1d4b2cb53e62c45b75edc699b737e46a157a3249f3a803e
[root@docker-server ~]# docker run -d --name nginx-2 -p 81:80 nginx
c8ce6a0630b66e260eef16d8ecf48049eed7b893b87459888b634bf0e9e40f23
[root@docker-server ~]# docker run -d --name nginx-3 -p 82:80 nginx
1cddbd412b5997f8935815c2f588431e100b752595ceaa92b95758ca45179096
```

- 连接进入某一个容器中，并创建一个文件

```bash
[root@docker-server ~]# docker exec -it nginx-1 bash
root@0e72f06bba41:/# echo 'hello world test!' > /opt/test1
root@0e72f06bba41:/# exit
```

- 宿主机是使用了chroot技术把容器锁定到一个指定的运行目录里

```bash
[root@docker-server diff]# find / -name test1
/var/lib/docker/overlay2/f9cc560395b5e3b11d2b1293922c4d31e6a6a32ca59af3d9274eabdfc6832424/diff/opt/test1
/var/lib/docker/overlay2/f9cc560395b5e3b11d2b1293922c4d31e6a6a32ca59af3d9274eabdfc6832424/merged/opt/test1
```

在 Docker 中，文件系统是通过分层存储机制实现的，这与 Docker 的镜像和容器的架构有关。看到的两个文件路径反映了 Docker 的存储驱动（如 Overlay2）的工作原理。

#### Docker 的存储架构

1. 镜像层（Read-Only）：
   - Docker 镜像是由多个只读层组成的。每一层代表了镜像的某个状态或修改。
   - 这些层是不可变的，一旦创建，不会被修改。
2. 容器层（Read-Write）：
   - 当你运行一个容器时，Docker 会在镜像层之上添加一个可写层。
   - 容器的所有写操作（如创建文件、修改文件等）都会在这个可写层中进行，而不会影响下面的镜像层。



#### Overlay2 存储驱动

Docker 默认使用 Overlay2 存储驱动（在支持的系统上）。Overlay2 的工作机制如下：

- `merged` 目录：
  - 这是容器的根文件系统，是镜像层和容器层的联合视图。
  - 当你在容器中访问文件时，看到的是 `merged` 目录中的内容。
  - 例如，你在容器中创建的文件 `/opt/test_nginx-1`，在宿主机上可以通过 `/var/lib/docker/overlay2/<id>/merged/opt/test_nginx-1` 访问。
- **`diff` 目录**：
  - 这是容器的可写层，记录了容器对文件系统的修改。
  - 当你在容器中创建或修改文件时，实际的文件数据会存储在 `diff` 目录中。
  - 例如，你在容器中创建的文件 `/opt/test_nginx-1`，其实际数据存储在 `/var/lib/docker/overlay2/<id>/diff/opt/test_nginx-1`。
- 当你在容器中创建文件 `/opt/test_nginx-1` 时：
  - 文件的实际数据被写入到 `diff` 目录中。
  - 在 `merged` 目录中，通过联合文件系统（OverlayFS）的机制，将 `diff` 目录中的文件映射到 `merged` 目录中，让你在容器中看到完整的文件系统视图。



### IPC Namespace

一个容器内的进程间通信，允许一个容器内的不同进程数据互相访问，但是不能跨容器访问其他容器的数据

UTS Namespace包含了运行内核的名称、版本、底层体系结构类型等信息用于系统表示，其中包含了hostname和域名，它使得一个容器拥有属于自己hostname标识，这个主机名标识独立于宿主机系统和其上的其他容器。



### PID Namespace

Linux系统中，有一个pid为1的进程（init/systemd）是其他所有进程的父进程，那么在每个容器内也要有一个父进程来管理其下属的进程，那么多个容器的进程通PID namespace进程隔离

- 安装软件包

```bash
[root@localhost ~]# docker exec -it 065f06e5caa4 bash
root@0e72f06bba41:/# apt update
# ifconfig
root@0e72f06bba41:/# apt install net-tools
# top
root@0e72f06bba41:/# apt install procps
# ping
root@0e72f06bba41:/# apt install iputils-ping
root@0e72f06bba41:/# ps -ef
```

**那么宿主机的PID与容器内的PID是什么关系？**

```bash
[root@docker-server ~]# yum install psmisc
[root@docker-server ~]# pstree -p

[root@localhost ~]# ps aux | grep 065f06e5caa4
```

1. 独立的 PID 命名空间:
   - 每个 Docker 容器都有自己独立的 PID 命名空间。
   - 容器内的进程 PID 从 1 开始编号,与宿主机上的 PID 是相互独立的。
2. PID 映射:
   - 容器内的进程 PID 与宿主机上的进程 PID 之间是有映射关系的。
   - 通过 `docker inspect <container_id>` 命令,可以查看容器内进程的 PID 与宿主机上进程 PID 的对应关系。
3. PID 可见性:
   - 容器内的进程只能看到容器内部的 PID。
   - 宿主机上的进程可以看到容器内部的 PID,但容器内的进程无法看到宿主机上的 PID。
4. PID 隔离:
   - 容器内的进程无法访问或影响宿主机上的其他进程。
   - 宿主机上的进程可以访问和管理容器内的进程。



### Net Namespace

每一个容器都类似于虚拟机一样有自己的网卡、监听端口、TCP/IP协议栈等，Docker使用network namespace启动一个vethX接口，这样容器将拥有它自己的桥接IP地址，通常是docker0，而docker0实质就是linux的虚拟网桥。

```bash
[root@docker-server ~]# yum install bridge-utils.x86_64 -y
[root@docker-server ~]# brctl show
bridge name    bridge id        STP enabled    interfaces
docker0        8000.0242c83ab23e    no        veth3ad3c5b
[root@docker-server ~]# ifconfig 
```

- 查看docker内部网卡

```bash
root@0d5d7069b9d9:/# ifconfig 
```



### User Namespace

各个容器内可能会出现重名的用户和用户组名称，或重复的用户UID或者GID，那么怎么隔离各个容器内的用户空间呢？

User Namespace允许在各个宿主机的各个容器空间内创建相同的用户名以及相同的uid和gid，只是此用户的有效范围仅仅是当前的容器内，不能访问另外一个容器内的文件系统，即相互隔离、互不影响、永不相见



## 五、Linux control groups

cgroups(Control Groups) 是 linux 内核提供的一种机制， 这种机制可以根据需求把一 系列系统任务及其子任务整合(或分隔)到按资源划分等级的不同组内，从而为系统资源管理提供一个统一的框架。 简单说， cgroups 可以限制、记录任务组所使用的物理资源。本质上来说， cgroups 是内核附加在程序上的一系列钩子(hook)，通过程序运行时对资源的调度触发相应的钩子以达到资源追踪和限制的目的。


cgroups 的用途
Resource limitation: 限制资源使用，例：内存使用上限/cpu 的使用限制
Prioritization: 优先级控制，例： CPU 利用/磁盘 IO 吞吐
Accounting: 一些审计或一些统计
Control: 挂起进程/恢复执行进程

- 验证系统内核层已经默认开启cgroup功能

```bash
[root@docker-server ~]# cat /boot/config-3.10.0-957.el7.x86_64| grep cgroup -i
CONFIG_CGROUPS=y
# CONFIG_CGROUP_DEBUG is not set
CONFIG_CGROUP_FREEZER=y
CONFIG_CGROUP_PIDS=y
CONFIG_CGROUP_DEVICE=y
CONFIG_CGROUP_CPUACCT=y
CONFIG_CGROUP_HUGETLB=y
CONFIG_CGROUP_PERF=y
CONFIG_CGROUP_SCHED=y
CONFIG_BLK_CGROUP=y
# CONFIG_DEBUG_BLK_CGROUP is not set
CONFIG_NETFILTER_XT_MATCH_CGROUP=m
CONFIG_NET_CLS_CGROUP=y
CONFIG_NETPRIO_CGROUP=y
```

- 关于内存的模块

```bash
[root@docker-server ~]#  cat /boot/config-3.10.0-957.el7.x86_64 | grep mem -i | grep cg -i
CONFIG_MEMCG=y
CONFIG_MEMCG_SWAP=y
CONFIG_MEMCG_SWAP_ENABLED=y
CONFIG_MEMCG_KMEM=y
CPU: 这个子系统通过 CPU 调度程序为 cgroup 任务提供 CPU 访问。可以控制每个 cgroup 的 CPU 份额、优先级等。
cpuacct: 这个子系统会产生每个 cgroup 任务的 CPU 资源使用报告,方便监控和管理。
cpuset: 在多核 CPU 环境下,这个子系统可以为 cgroup 任务分配特定的 CPU 核心和内存区域。
devices: 这个子系统可以允许或拒绝 cgroup 任务对特定设备的访问。
freezer: 这个子系统可以暂停和恢复 cgroup 任务的运行。
memory: 这个子系统可以为每个 cgroup 设置内存使用限制,并产生内存资源使用报告。
net_cls: 这个子系统可以为 cgroup 任务生成的网络数据包打标签,方便网络策略的管理。
ns: 这个子系统提供了对 cgroup 的命名空间隔离。
perf event: 这个子系统增强了对 cgroup 任务的性能监控和跟踪能力。
```

扩展阅读：

https://blog.csdn.net/qyf158236/article/details/110475457

### Docker 中的 cgroups 资源限制
1. **CPU 资源限制**
Docker 提供了多种方式来限制容器的 CPU 使用：
**`--cpus`**：限制容器可以使用的 CPU 核心数量。例如，`--cpus="1.5"` 表示容器最多可以使用 1.5 个 CPU。
**`--cpu-shares`**：设置容器的 CPU 使用权重。默认值为 1024，值越高，分配的 CPU 时间片越多。
**`--cpu-period` 和 `--cpu-quota`**：更细粒度地控制 CPU 时间片。`--cpu-period` 设置 CPU 时间片的周期（单位为微秒），`--cpu-quota` 设置每个周期内容器可以使用的 CPU 时间。


2. **内存资源限制**
Docker 可以通过以下参数限制容器的内存使用：
- **`-m` 或 `--memory`**：限制容器的物理内存使用量。例如，`-m 512m` 表示限制容器使用 512MB 的物理内存。
- **`--memory-swap`**：限制容器的总内存使用量（物理内存 + 交换空间）。例如，`--memory-swap=1g` 表示容器可以使用 1GB 的总内存。


3. **磁盘 I/O 资源限制**
Docker 可以限制容器的磁盘 I/O 使用：
**`--blkio-weight`**：设置容器的块设备 I/O 权重，范围为 10 到 1000。
**`--device-read-bps` 和 `--device-write-bps`**：限制特定设备的读写速率。例如，`--device-read-bps /dev/sda:1mb` 表示限制容器对 `/dev/sda` 的读取速率为 1MB/s。


### 查看和管理 cgroups 资源限制
**查看 cgroups 配置**：可以通过访问 `/sys/fs/cgroup` 目录来查看容器的 cgroups 配置。例如，`/sys/fs/cgroup/cpu/docker/<container_id>` 目录下包含了容器的 CPU 资源限制文件。
**动态调整资源限制**：在容器运行时，可以通过修改 cgroups 文件的内容来动态调整资源限制。



### 使用压缩工具测试

```bash
[root@bogon ~]# docker pull lorel/docker-stress-ng
Using default tag: latest
latest: Pulling from lorel/docker-stress-ng
c52e3ed763ff: Pull complete 
a3ed95caeb02: Pull complete 
7f831269c70e: Pull complete 
Digest: sha256:c8776b750869e274b340f8e8eb9a7d8fb2472edd5b25ff5b7d55728bca681322
Status: Downloaded newer image for lorel/docker-stress-ng:latest
```



### 测试CPU

不限制cpu使用

```bash
[root@bogon ~]# docker container run --name stress -it --rm lorel/docker-stress-ng:latest  --cpu 8
stress-ng: info: [1] defaulting to a 86400 second run per stressor
stress-ng: info: [1] dispatching hogs: 8 cpu
[root@bogon ~]# docker stats
CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PIDS
92b0b8d916c1        stress              101.54%             15.81MiB / 983.3MiB   1.61%               648B / 0B           0B / 0B             9
[root@bogon ~]# top
```

可以看到，cpu使用已经满了



重新启动容器加入内存限制参数

```plain
[root@bogon ~]# docker container run --name stress --cpus=0.5 -it --rm lorel/docker-stress-ng:latest  --cpu 8
stress-ng: info: [1] defaulting to a 86400 second run per stressor
stress-ng: info: [1] dispatching hogs: 8 cpu
[root@bogon ~]# docker stats
CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PIDS
845220ef9982        stress              51.57%              20.05MiB / 983.3MiB   2.04%               648B / 0B           0B / 0B             9
```

设置的参数生效



## 六、容器规范

容器技术除了docker之外，还有coreOS的rkt，还有阿里的Pouch，还有红帽 的podman，为了保证容器生态的标准性和健康可持续发展，包括Linux基金会、Docker、微软、红帽、谷歌和IBM等公司在2015年6月共同成立了一个叫open container（OCI）的组织，其目的就是制定开放的标准的容器规范，目前OCI一共发布了两个规范分别是runtime spec和image format spec，不同的容器公司只需要兼容这两个规范，就可以保证容器的可移植性和相互可操作性。



### OCI目标

OCI 的主要目标是制定开放的容器规范，以确保不同容器技术之间的可移植性和互操作性。目前，OCI 已经发布了两个核心规范：

1. **Runtime Spec**：定义了容器运行时的规范，包括容器的生命周期管理、资源隔离和安全等。
2. **Image Format Spec**：定义了容器镜像的格式和元数据，确保镜像可以在不同的容器运行时之间共享和运行。

通过遵循这些规范，不同的容器运行时和工具可以实现互操作性，从而推动容器技术的标准化和健康发展。



### 主流容器运行时

容器运行时是真正运行容器的地方，它需要与操作系统的内核紧密合作，为容器提供隔离的运行环境。以下是目前主流的三种容器运行时：

#### 1. **LXC (Linux Containers)**

- **简介**：LXC 是 Linux 上早期的容器运行时，它利用 Linux 内核的 Namespace 和 Cgroups 技术来实现进程隔离和资源管理。
- 特点：
  - 提供了完整的 Linux 系统环境，支持多种 Linux 发行版。
  - 早期 Docker 也曾使用 LXC 作为其默认的运行时。
- **适用场景**：适用于需要完整 Linux 系统环境的容器化应用。

#### 2. **Runc**

- **简介**：Runc 是目前 Docker 默认的容器运行时，它是一个轻量级的命令行工具，用于运行和管理容器。
- 特点：
  - 完全遵循 OCI 的 Runtime Spec 规范，确保与 OCI 标准的兼容性。
  - 由于其轻量级和高性能的特点，Runc 已经成为许多容器运行时的底层实现。
- **适用场景**：适用于需要高性能和轻量级容器运行环境的场景。

#### 3. **Rkt (Rocket)**

- **简介**：Rkt 是由 CoreOS 开发的容器运行时，旨在提供一个安全、可靠且符合 OCI 规范的容器运行环境。
- 特点：
  - 与 Docker 不同，Rkt 本身是一个独立的容器运行时，不依赖 Docker 的守护进程。
  - 提供了更好的安全性和隔离性，例如通过 AppArmor 和 SELinux 等安全机制。
- **适用场景**：适用于对安全性要求较高的容器化应用。

容器技术的发展离不开标准化的推动。OCI 通过制定 Runtime Spec 和 Image Format Spec，为容器运行时和工具提供了统一的标准，确保了不同容器技术之间的互操作性和可移植性。目前主流的容器运行时（如 LXC、Runc 和 Rkt）都遵循这些规范，从而推动了容器技术的广泛应用和发展。



## 七、docker info信息

```bash
[root@docker-server ~]# docker info # 用于显示 docker 的系统级信息，比如内核，镜像数，容器数等。
```



## 八、docker 存储引擎

### **核心概念及工作原理**

Docker 存储引擎的核心思想是“层”的概念。镜像是由多个只读层组成的，而容器则在镜像的基础上添加了一个可读写的层。这种分层架构使得镜像的复用和部署变得非常方便，同时减少了容器的体积。

Docker 使用联合文件系统（Union File System）来管理容器的文件系统。联合文件系统允许将多个目录（或文件系统）合并为一个统一的文件系统视图。Docker 的存储引擎通过这种机制，将镜像层和容器层合并在一起，使得容器能够看到一个完整的文件系统。

Docker 支持多种存储引擎，每种存储引擎都有其特点和适用场景。以下是一些常见的存储引擎：

**AUFS（Another Union File System）**

- **特点**：AUFS 是一种文件级的存储驱动，允许多个目录共享相同的文件系统层次结构。它通过联合挂载技术将多个目录挂载到一个单一的文件系统上。
- **适用场景**：AUFS 曾是 Docker 早期版本的默认存储驱动，但在较新的 Docker 版本中已被 Overlay2 替代。

**OverlayFS**

- **特点**：OverlayFS 是一种更现代的联合文件系统，从 Linux 内核 3.18 开始支持。它将文件系统简化为两层：一个只读的下层（lowerdir）和一个可读写的上层（upperdir），统一后的视图称为合并层（merged）。
- **优势**：OverlayFS 支持页缓存共享，多个容器如果读取相同层的同一个文件，可以共享页缓存，从而提高内存利用率。此外，OverlayFS 在性能和稳定性方面表现更好，是目前 Docker 的默认存储驱动。



### Docker 的 Overlay2 存储驱动介绍

1. **什么是 Overlay2？**

Overlay2 是 Docker 中的一种存储驱动，用于管理容器和镜像的文件系统。它是 OverlayFS 的改进版本，解决了早期 Overlay 驱动可能遇到的 inode 耗尽问题。Overlay2 使用联合文件系统（Union File System）技术，将多个文件系统层合并为一个统一的文件系统视图，从而实现高效的容器文件系统管理。

1. **Overlay2 的工作原理**

Overlay2 通过以下三个主要目录来管理文件系统：

- **`LowerDir`**：只读层，包含基础镜像的文件系统。可以有多个只读层，每层都是独立的。
- **`UpperDir`**：读写层，用于存储容器运行时的文件系统变更（即 diff 层）。
- **`MergedDir`**：联合挂载后的视图，容器看到的完整文件系统。它将 `LowerDir` 和 `UpperDir` 合并为一个统一的文件系统视图。
- **`WorkDir`**：工作目录，用于联合挂载的内部操作，挂载后内容被清空。

当启动一个容器时，Overlay2 会将镜像层（`LowerDir`）和容器层（`UpperDir`）联合挂载到 `MergedDir`，容器通过这个目录看到完整的文件系统。



### 镜像加速配置

打开网址

http://cr.console.aliyun.com/

登陆之后点击镜像加速器，按照指导说明即可



# 二、docker镜像管理

## 一、搜索镜像

```bash
[root@docker-server ~]# docker search centos  #搜索与centos有关的镜像
NAME               DESCRIPTION                             STARS     OFFICIAL   AUTOMATED
```

在官网搜索时，可以看到返回了很多包含关键字的镜像，其中包括镜像名字、描述、点赞数（表示该镜像的受欢迎程度）、是否官方创建、是否自动创建。默认输出结果按照星级评价进行排序。



## 二、下载镜像

可以使用docker pull命令直接下载镜像，语法为：

```bash
docker pull NAME:TAG
```

其中，NAME是镜像名称，TAG是镜像的标签（往往用来是表示版本信息），通常情况下，描述一个镜像需要包括名称+标签，如果不指定标签，标签的值默认为latest。

- 下载nginx、centos、hello-world镜像

```bash
[root@docker-server ~]# docker pull nginx
[root@docker-server ~]# docker pull centos
[root@docker-server ~]# docker pull hello-world
```



## 三、查看镜像信息

### docker images

- 列出本地所有镜像

```bash
[root@docker-server ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
nginx         latest    d1a364dc548d   2 weeks ago    133MB
hello-world   latest    d1165f221234   3 months ago   13.3kB
centos        latest    300e315adb2f   6 months ago   209MB
```

在列出的信息中可以看到几个字段：

- REPOSITORY：镜像仓库名称
- TAG：镜像的标签信息
- 镜像ID：唯一用来标识镜像，如果两个镜像ID相同，说明他们实际上指向了同一个镜像，只是具有不同标签名称而已
- CREATED：创建时间，说明镜像的最后更新时间
- SIZE：镜像大小，优秀的镜像往往体积都较小



### docker tag

为了方便在后续工作中使用特定镜像，可以使用docker tag命令来为本地镜像任意添加新的标签

```bash
[root@docker-server ~]# docker tag centos:latest mycentos:latest
[root@docker-server ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
nginx         latest    d1a364dc548d   2 weeks ago    133MB
hello-world   latest    d1165f221234   3 months ago   13.3kB
centos        latest    300e315adb2f   6 months ago   209MB
mycentos      latest    300e315adb2f   6 months ago   209MB
```



### docker inspect

可以使用docker inspect命令获取该镜像的详细信息

```bash
[root@docker-server ~]# docker inspect centos:latest 
[
    {
        "Id": "sha256:300e315adb2f96afe5f0b2780b87f28ae95231fe3bdd1e16b9ba606307728f55",
        "RepoTags": [
            "centos:latest",
            "mycentos:latest"
        ],
        "RepoDigests": [
            "centos@sha256:5528e8b1b1719d34604c87e11dcd1c0a20bedf46e83b5632cdeac91b8c04efc1"
        ],
        "Parent": "",
        "Comment": "",
        "Created": "2020-12-08T00:22:53.076477777Z",
        "Container": "395e0bfa7301f73bc994efe15099ea56b8836c608dd32614ac5ae279976d33e4",
        "ContainerConfig": {
            "Hostname": "395e0bfa7301",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "#(nop) ",
                "CMD [\"/bin/bash\"]"
            ],
            "Image": "sha256:6de05bdfbf9a9d403458d10de9e088b6d93d971dd5d48d18b4b6758f4554f451",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "org.label-schema.build-date": "20201204",
                "org.label-schema.license": "GPLv2",
                "org.label-schema.name": "CentOS Base Image",
                "org.label-schema.schema-version": "1.0",
                "org.label-schema.vendor": "CentOS"
            }
        },
        "DockerVersion": "19.03.12",
        "Author": "",
        "Config": {
            "Hostname": "",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/bash"
            ],
            "Image": "sha256:6de05bdfbf9a9d403458d10de9e088b6d93d971dd5d48d18b4b6758f4554f451",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "org.label-schema.build-date": "20201204",
                "org.label-schema.license": "GPLv2",
                "org.label-schema.name": "CentOS Base Image",
                "org.label-schema.schema-version": "1.0",
                "org.label-schema.vendor": "CentOS"
            }
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 209348104,
        "VirtualSize": 209348104,
        "GraphDriver": {
            "Data": {
                "MergedDir": "/var/lib/docker/overlay2/fc171623b3190077aec6cf1fdd969e635e7a5144b30ad8ca94204d02739b1e9e/merged",
                "UpperDir": "/var/lib/docker/overlay2/fc171623b3190077aec6cf1fdd969e635e7a5144b30ad8ca94204d02739b1e9e/diff",
                "WorkDir": "/var/lib/docker/overlay2/fc171623b3190077aec6cf1fdd969e635e7a5144b30ad8ca94204d02739b1e9e/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:2653d992f4ef2bfd27f94db643815aa567240c37732cae1405ad1c1309ee9859"
            ]
        },
        "Metadata": {
            "LastTagTime": "2021-06-09T10:18:13.630764623+08:00"
        }
    }
]
```



### docker history

镜像由多层组成，可以使用history子命令，该命令将列出各层创建信息

```bash
[root@docker-server ~]# docker history centos:latest 
IMAGE          CREATED        CREATED BY                                      SIZE      COMMENT
300e315adb2f   6 months ago   /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B        
<missing>      6 months ago   /bin/sh -c #(nop)  LABEL org.label-schema.sc…   0B        
<missing>      6 months ago   /bin/sh -c #(nop) ADD file:bd7a2aed6ede423b7…   209MB
```



## 四、镜像导入导出

### 导出

可以将镜像从本地导出为一个压缩文件，然后复制到其他服务器进行导入使用

- 导出方法一

```bash
[root@docker-server ~]# docker save centos:latest -o /opt/centos.tar.gz
[root@docker-server ~]# ll /opt/centos.tar.gz 
-rw------- 1 root root 216535040 6月   9 10:33 /opt/centos.tar.gz
```

- 导出方法二

```bash
[root@docker-server ~]# docker save centos:latest > /opt/centos-1.tar.gz
[root@docker-server ~]# ll /opt/centos-1.tar.gz 
-rw-r--r-- 1 root root 216535040 6月   9 10:35 /opt/centos-1.tar.gz
```



### 导入

先将导出的镜像发到需要导入的docker服务器中

- 导入方法一

```bash
[root@docker-server ~]# docker load -i /opt/centos.tar.gz 
Loaded image: centos:latest
```

- 导出方法二

```bash
[root@docker-server ~]# docker load < /opt/centos.tar.gz 
Loaded image: centos:latest
```



## 五、删除镜像

- 使用镜像名称+标签

```bash
[root@docker-server ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
nginx         latest    d1a364dc548d   2 weeks ago    133MB
hello-world   latest    d1165f221234   3 months ago   13.3kB
centos        latest    300e315adb2f   6 months ago   209MB
mycentos      latest    300e315adb2f   6 months ago   209MB
[root@docker-server ~]# docker rmi nginx:latest 
```

- 使用镜像id 

```bash
[root@docker-server ~]# docker images REPOSITORY TAG IMAGE ID CREATED SIZE 
hello-world latest d1165f221234 3 months ago 13.3kB 
centos latest 300e315adb2f 6 months ago 209MB 
mycentos latest 300e315adb2f 6 months ago 209MB

[root@docker-server ~]# docker rmi 300e315adb2f 
Error response from daemon: conflict: unable to delete 300e315adb2f (must be forced) - image is referenced in multiple repositories 

[root@docker-server ~]# docker rmi 300e315adb2f -f 
Untagged: centos:latest Untagged: centos@sha256:5528e8b1b1719d34604c87e11dcd1c0a20bedf46e83b5632cdeac91b8c04efc1 
Untagged: mycentos:latest Deleted: sha256:300e315adb2f96afe5f0b2780b87f28ae95231fe3bdd1e16b9ba606307728f55 Deleted: sha256:2653d992f4ef2bfd27f94db643815aa567240c37732cae1405ad1c1309ee9859

 [root@docker-server ~]# docker images REPOSITORY TAG IMAGE ID CREATED SIZE 
 hello-world latest d1165f221234 3 months ago 13.3kB
```



# 三、docker容器管理

## 一、创建容器

### docker create

docker create命令新建的容器处于停滞状态，可以使用docker start命令来启动它

| 选项          | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| -d            | 是否在后台运行容器，默认为否                                 |
| -i            | 保持标准输入打开                                             |
| -P            | 通过NAT机制将容器标记暴露的端口自动映射到本地主机的临时端口  |
| -p            | 指定如何映射到本地主机端口                                   |
| -t            | 分配一个终端                                                 |
| -v            | 挂载主机上的文件卷到容器内                                   |
| --rm          | 容器推出后是否自动删除，不能跟-d同时使用                     |
| -e            | 指定容器内的环境变量                                         |
| -h            | 指定容器内的主机名                                           |
| --name        | 指定容器的别名                                               |
| --cpu-shares  | 允许容器使用cpu资源的相对权重，默认一个容器能用满一个核的cpu |
| --cpuset-cpus | 限制容器能使用哪些cpu核心                                    |
| -m            | 限制容器内使用的内存，单位可以是b、k、m、g                   |

```shell
docker create -it --name mycontainer mycentos bash
docker start mycontainer
```



### docker run

 docker run 选项:

```bash
 --add-host list                  添加自定义主机到IP映射（主机：IP）
 -a, --attach list                    连接到stdin、stdout或stderr
     --blkio-weight uint16            块IO（相对权重），介于10到1000之间，或0到禁用（默认为0）
     --blkio-weight-device list       块IO权重（相对设备权重）（默认值[]）
     --cap-add list                   添加Linux功能
     --cap-drop list                  放弃Linux功能
     --cgroup-parent string           容器的可选父cgroup
     --cidfile string                 将容器id写入文件
     --cpu-period int                 限制CPU CFS（完全公平调度程序）周期
     --cpu-quota int                  限制CPU CFS（完全公平调度程序）配额
     --cpu-rt-period int              限制CPU实时周期（微秒）
     --cpu-rt-runtime int             限制CPU实时运行时间（微秒）
 -c, --cpu-shares int                 CPU共享（相对权重）
     --cpus decimal                   CPU数量
     --cpuset-cpus string             允许执行的CPU（0-3、0、1）
     --cpuset-mems string             允许执行的微机电系统（0-3、0、1）
 -d, --detach                         在后台运行容器并打印容器ID
     --detach-keys string             重写用于分离容器的键序列
     --device list                    将主机设备添加到容器
     --device-cgroup-rule list        将规则添加到cgroup allowed devices列表
     --device-read-bps list           限制设备的读取速率（字节/秒）（默认为[]）
     --device-read-iops list          限制设备的读取速率（IO/秒）（默认值[]）
     --device-write-bps list          限制对设备的写入速率（字节/秒）（默认值[]）
     --device-write-iops list         限制设备的写入速率（IO/秒）（默认值[]）
     --disable-content-trust          跳过图像验证（默认为true）
     --dns list                       设置自定义DNS服务器
     --dns-option list                设置DNS选项
     --dns-search list                设置自定义DNS搜索域
     --domainname string              容器NIS域名
     --entrypoint string              覆盖图像的默认入口点
 -e, --env list                       设置环境变量
     --env-file list                  读取环境变量文件
     --expose list                    公开一个或一系列端口
     --gpus gpu-request               要添加到容器的GPU设备（“all”用于传递所有GPU）
     --group-add list                 添加要加入的其他组
     --health-cmd string              运行以检查运行状况的命令
     --health-interval duration       运行检查之间的时间（m s s m h）（默认为0s）
     --health-retries int             需要连续失败才能报告不正常
     --health-start-period duration   开始运行状况重试倒计时之前容器初始化的开始时间
                                      （m s s m h）（默认为0）
     --health-timeout duration        允许一个检查运行的最大时间（ms s m m h）（默认值0）
     --help                           打印使用
 -h, --hostname string                容器主机名
     --init                           在转发信号和获取进程的容器中运行init
 -i, --interactive                    即使没有连接，也保持stdin打开
     --ip string                      IPv4地址（例如172.30.100.104）
     --ip6 string                     IPv6地址（例如，2001:db8：：33）
     --ipc string                     要使用的IPC模式
     --isolation string               集装箱隔离技术
     --kernel-memory bytes            内核内存限制
 -l, --label list                     在容器上设置元数据
     --label-file list                读取以行分隔的标签文件
     --link list                      将链接添加到另一个容器
     --link-local-ip list             容器IPv4/IPv6链路本地地址
     --log-driver string              容器的日志驱动程序
     --log-opt list                   日志驱动程序选项
     --mac-address string             容器MAC地址（例如92:d0:c6:0a:29:33）
 -m, --memory bytes                   记忆极限
     --memory-reservation bytes       存储器软限制
     --memory-swap bytes              交换限制等于内存加交换：'-1'以启用无限制交换
     --memory-swappiness int          调整容器内存交换（0到100）（默认-1）
     --mount mount                    将文件系统装载附加到容器
     --name string                    为容器指定名称
     --network network                将容器连接到网络
     --network-alias list             为容器添加网络范围的别名
     --no-healthcheck                 禁用任何容器指定的运行状况检查
     --oom-kill-disable               禁用OOM杀手
     --oom-score-adj int              调整主机的OOM首选项（-1000到1000）
     --pid string                     要使用的PID命名空间
     --pids-limit int                 调整容器PIDS限制（设置-1表示无限制）
     --privileged                     授予此容器扩展权限
 -p, --publish list                   将容器的端口发布到主机
 -P, --publish-all                    将所有公开的端口发布到随机端口
     --read-only                      将容器的根文件系统装载为只读
     --restart string                 重新启动策略以在容器退出时应用（默认为“否”）
     --rm                             容器退出时自动拆卸
     --runtime string                 用于此容器的运行时
     --security-opt list              安全选项
     --shm-size bytes                 /dev/shm的大小
     --sig-proxy                      代理接收到进程的信号（默认为true）
     --stop-signal string             停止容器的信号（默认为“sigterm”）
     --stop-timeout int               停止容器超时（秒）
     --storage-opt list               容器的存储驱动程序选项
     --sysctl map                     sysctl选项（默认映射[]）
     --tmpfs list                     装入tmpfs目录
 -t, --tty                            分配一个伪tty
     --ulimit ulimit                  ulimit选项（默认值[]）
 -u, --user string                    用户名或uid（格式：<name uid>[：<group gid>]）
     --userns string                  要使用的用户命名空间
     --uts string                     要使用的uts命名空间
 -v, --volume list                    绑定装入卷
     --volume-driver string           容器的可选卷驱动程序
     --volumes-from list              从指定容器装入卷
 -w, --workdir string                 容器内的工作目录
```

除了创建容器后通过start命令来启动也可以通过docker run直接新建并启动容器。



- 启动一个容器

```bash
[root@docker-server ~]# docker run -it centos:latest bash
[root@4abaf8a399fe /]#
```

- 显示正在运行的容器

```bash
[root@docker-server ~]# docker ps 
CONTAINER ID   IMAGE           COMMAND   CREATED          STATUS          PORTS     NAMES
4abaf8a399fe   centos:latest   "bash"    47 seconds ago   Up 46 seconds             hardcore_perlman
```

- 显示所有容器，包括停止的所有容器

```bash
[root@docker-server ~]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[root@docker-server ~]# docker ps -a
CONTAINER ID   IMAGE           COMMAND   CREATED         STATUS                     PORTS     NAMES
4abaf8a399fe   centos:latest   "bash"    2 minutes ago   Exited (0) 9 seconds ago             hardcore_perlman
```



### 端口映射

- 前台启动随机映射端口

```bash
[root@docker-server ~]# docker pull nginx
[root@docker-server ~]# docker run -P nginx
# 随机映射端口，其实是从32768开始映射
[root@docker-server ~]# ss -tanl
State       Recv-Q Send-Q Local Address:Port               Peer Address:Port              
LISTEN      0      128        *:22                     *:*                  
LISTEN      0      100    127.0.0.1:25                     *:*                  
LISTEN      0      128        *:49153                  *:*                  
LISTEN      0      128       :::22                    :::*                  
LISTEN      0      100      ::1:25                    :::*                  
LISTEN      0      128       :::49153                 :::*
```



- 指定端口映射

```bash
# 方式1，本地端口80映射到容器80端口
[root@docker-server ~]# docker run -p 80:80 --name nginx-1 nginx:latest 
# 方式2，本地ip：本地端口：容器端口
[root@docker-server ~]# docker run -p 192.168.204.135:80:80 --name nginx-1 nginx:latest 
# 方式3，本地ip：本地随机端口：容器端口
[root@docker-server ~]# docker run -p 192.168.175.10::80 --name nginx-1 nginx:latest 
# 方式4，本地ip：本地端口：容器端口/协议默认为tcp协议
[root@docker-server ~]# docker run -p 192.168.175.10:80:80/tcp --name nginx-1 nginx:latest
```

- 查看容器已经映射的端口

```bash
[root@docker-server ~]# docker port nginx-1 
80/tcp -> 0.0.0.0:80
80/tcp -> :::80
```



### 后台启动容器

- 当容器前台启动时，前台进程退出容器也就退出，更多时候需要容器在后台启动

```bash
[root@docker-server ~]# docker run -d -P --name nginx-2 nginx
c75333168c0dad9094d94828c33998294f2809ae8c5b60881707d9cc33ea4893
```

- 传递运行命令

容器需要由一个前台运行的进程才能保持容器的运行，通过传递运行参数是一种方式，另外也可以在构建镜像的时候指定容器启动时运行的前台命令

```bash
[root@docker-server ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[root@docker-server ~]# docker run -d centos
9ef312d30f7a396ecb5c93b7b70e70a742f333bbe01e9112d6f22fc52aeb71b8
[root@docker-server ~]# docker ps 
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[root@docker-server ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS                      PORTS     NAMES
9ef312d30f7a   centos    "/bin/bash"   12 seconds ago   Exited (0) 11 seconds ago             upbeat_noether
[root@docker-server ~]# docker run -d centos tail -f /etc/hosts
7b0700c01f9516f49e70ad92e7256d965e0fe4eb8ccc7b30676a03c1d8046c64
[root@docker-server ~]# docker ps
CONTAINER ID   IMAGE     COMMAND                CREATED         STATUS         PORTS     NAMES
7b0700c01f95   centos    "tail -f /etc/hosts"   3 seconds ago   Up 2 seconds             charming_brahmagupta
```

- 单次运行，容器退出后自动删除

```bash
[root@docker-server ~]# docker run --name hello_world_test --rm hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

[root@docker-server ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```



## 二、停止容器

### 暂停容器

- 挂起容器

```bash
[root@docker-server ~]# docker ps 
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS     NAMES
4c6344c46c80   nginx     "/docker-entrypoint.…"   8 seconds ago   Up 8 seconds   80/tcp    wizardly_hofstadter
[root@docker-server ~]# docker pause wizardly_hofstadter 
wizardly_hofstadter
[root@docker-server ~]# docker ps 
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS                   PORTS     NAMES
4c6344c46c80   nginx     "/docker-entrypoint.…"   17 seconds ago   Up 16 seconds (Paused)   80/tcp    wizardly_hofstadter
```

- 取消挂起容器

```bash
[root@docker-server ~]# docker unpause wizardly_hofstadter 
wizardly_hofstadter
[root@docker-server ~]# docker ps 
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS     NAMES
4c6344c46c80   nginx     "/docker-entrypoint.…"   33 seconds ago   Up 33 seconds   80/tcp    wizardly_hofstadter
```

### 终止容器

```bash
[root@docker-server ~]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED              STATUS              PORTS     NAMES
4c6344c46c80   nginx     "/docker-entrypoint.…"   About a minute ago   Up About a minute   80/tcp    wizardly_hofstadter

[root@docker-server ~]# docker stop wizardly_hofstadter 
wizardly_hofstadter
[root@docker-server ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS                     PORTS     NAMES
4c6344c46c80   nginx     "/docker-entrypoint.…"   2 minutes ago   Exited (0) 5 seconds ago             wizardly_hofstadter

[root@docker-server ~]# docker start wizardly_hofstadter 
wizardly_hofstadter
[root@docker-server ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS     NAMES
4c6344c46c80   nginx     "/docker-entrypoint.…"   2 minutes ago   Up 2 seconds   80/tcp    wizardly_hofstadter
```



## 三、删除容器

### docker rm

- 删除正在运行的容器

```bash
[root@docker-server ~]# docker ps
CONTAINER ID   IMAGE           COMMAND   CREATED         STATUS         PORTS     NAMES
dfd5ba20c3c6   centos:latest   "bash"    8 seconds ago   Up 6 seconds             frosty_elbakyan
[root@docker-server ~]# docker rm -f dfd5ba20c3c6
dfd5ba20c3c6
```

- 批量删除容器

```bash
[root@docker-server ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS     NAMES
8e3cb314c9ad   nginx     "/docker-entrypoint.…"   4 seconds ago   Up 3 seconds   80/tcp    pedantic_lovelace
4ab46864c8a3   nginx     "/docker-entrypoint.…"   5 seconds ago   Up 4 seconds   80/tcp    beautiful_spence
26a154528469   nginx     "/docker-entrypoint.…"   5 seconds ago   Up 5 seconds   80/tcp    serene_booth
2ecbf60d817a   nginx     "/docker-entrypoint.…"   6 seconds ago   Up 6 seconds   80/tcp    dreamy_bassi
d73faf8c2f7d   nginx     "/docker-entrypoint.…"   8 seconds ago   Up 8 seconds   80/tcp    beautiful_solomon
[root@docker-server ~]# docker ps -a -q
8e3cb314c9ad
4ab46864c8a3
26a154528469
2ecbf60d817a
d73faf8c2f7d
[root@docker-server ~]# docker rm -f `docker ps -a -q`
8e3cb314c9ad
4ab46864c8a3
26a154528469
2ecbf60d817a
d73faf8c2f7d
[root@docker-server ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```



## 四、进入容器

#### attach

所有使用此方式进入容器的操作都是同步显示的且exit容器将被关闭，且使用exit退出后容器关闭，不推荐使用

当有一个容器执行exit退出后会导致容器退出



#### exec

执行单次命令与进入容器，退出容器后容器还在运行

```bash
[root@docker-server ~]# docker run -d -it centos
129d518869d550e579bcff38608bae38209923dcbfab49c823d5e1473d38214a
[root@docker-server ~]# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS        PORTS     NAMES
129d518869d5   centos    "/bin/bash"   2 seconds ago   Up 1 second             jovial_haibt
[root@docker-server ~]# docker exec -it jovial_haibt /bin/bash
[root@129d518869d5 /]# echo hello
hello
[root@129d518869d5 /]# exit
exit
[root@docker-server ~]# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS          PORTS     NAMES
129d518869d5   centos    "/bin/bash"   46 seconds ago   Up 45 seconds             jovial_haibt

在Docker中，docker exec命令用于在正在运行的容器中执行命令。docker exec命令的一般语法如下：
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
其中，参数的含义如下：

OPTIONS：可选参数，用于指定一些附加选项。常用的选项包括：

-i, --interactive：保持标准输入（stdin）打开，允许用户与命令交互。
-t, --tty：分配一个伪终端（pseudo-TTY），以便于命令的交互和输出格式化。
-u, --user：指定执行命令的用户或用户组。
-d, --detach：在后台运行命令。
-e, --env：设置环境变量。
-w, --workdir：指定命令执行的工作目录。
等等。

CONTAINER：必需参数，指定要执行命令的容器名称或容器ID。
COMMAND：必需参数，指定要在容器中执行的命令。
ARG：可选参数，传递给命令的参数
```



#### nsenter

nsenter命令需要通过pid进入到容器内部，不过可以使用docker inspect获取到容器的pid

- 可以通过docker inspect获取到某个容器的进程id

```bash
[root@docker-server ~]# docker inspect -f "[.State.Pid]" 129d518869d5
7949
```

- 通过nsenter进入到容器内部

```bash
[root@docker-server ~]# nsenter -t 7949 -m -u -i -n -p
[root@129d518869d5 /]#
```

- 使用脚本方式进入

```bash
[root@docker-server ~]# cat docker_in.sh 
#!/bin/bash
docker_in(){
DOCKER_ID=$1
PID=`docker inspect -f "[.State.Pid]" ${DOCKER_ID}`
nsenter -t $[PID] -m -u -i -n -p
}

docker_in $1
[root@docker-server ~]# chmod +x docker_in.sh 
[root@docker-server ~]# ./docker_in.sh 129d518869d5
[root@129d518869d5 /]# exit
logout
[root@docker-server ~]# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS          PORTS     NAMES
129d518869d5   centos    "/bin/bash"   14 minutes ago   Up 14 minutes             jovial_haibt
```



## 五、指定容器DNS

dns服务，默认采用dns地址

一是通过将dns地址配置在宿主机上

二是将参数配置在docker启动脚本里面

```bash
[root@docker-server ~]# docker run -it --rm --dns 8.8.8.8 centos bash
[root@a6ce80126e75 /]# cat /etc/resolv.conf 
nameserver 8.8.8.8
[root@a6ce80126e75 /]# ping www.baidu.com -c 1
PING www.a.shifen.com (180.101.49.11) 56(84) bytes of data.
64 bytes from 180.101.49.11 (180.101.49.11): icmp_seq=1 ttl=127 time=9.35 ms

--- www.a.shifen.com ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 9.346/9.346/9.346/0.000 ms
[root@a6ce80126e75 /]# exit
exit
```



## 六、导入导出容器

### docker export

导出容器是指，导出一个已经创建的容器到一个文件，不管此时这个容器是否处于运行状态，**导出一个容器快照**

```bash
[root@docker-server ~]# docker run -d -it centos
43f2397b9456d27a3b84dba0d79ae9a1dd8dddf40440d7d73fca71cddea0e10d
[root@docker-server ~]# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS         PORTS     NAMES
43f2397b9456   centos    "/bin/bash"   2 seconds ago   Up 2 seconds             awesome_rubin
[root@docker-server ~]# docker export -o /opt/centos.tar 43f
[root@docker-server ~]# ll /opt/centos.tar
-rw------- 1 root root 216525312 6月   9 13:28 /opt/centos.tar
```



### docker import

导出的文件可以使用docker import命令导入变成镜像，**导入一个容器快照到本地镜像库**

```bash
[root@docker-server ~]# docker import /opt/centos.tar mycentos:v1
sha256:acf250a6cabb56e0464102dabedb0a562f933facd3cd7b387e665459da46bf29
[root@docker-server ~]# docker images | grep nginx
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
mycentos      v1        acf250a6cabb   9 seconds ago   209MB
nginx         latest    d1a364dc548d   2 weeks ago     133MB
```

适用场景：主要用来制作基础镜像，比如从一个ubuntu镜像启动一个容器，然后安装一些软件和进行一些设置后，使用docker export保存为一个基础镜像。然后把这个镜像分发给其他人使用，作为基础的开发环境。(因为export导出的镜像只会保留从镜像运行到export之间对文件系统的修改，所以只适合做基础镜像)



# 四、docker镜像制作

## 一、手动制作yum版nginx镜像：

​		Docker镜像制作类似于虚拟机的模板制作，即按照公司的实际业务将需要安装的软件、相关配置等基础环境配置完成，然后将虚拟机再提交为模板，最后再批量从模板批量创建新的虚拟机，这样可以极大地简化业务中相同环境的虚拟机运行环境的部署工作，Docker的镜像制作分为手动制作可自动制作（基于DockerFile），企业通常都是基于DockerFile制作镜像。

- 启动一个centos容器，安装好常用软件以及nginx

```bash
[root@docker-server ~]# docker run -it centos bash
# 安装wget、epel、nginx等相关常用软件
[root@0195bc1d0f7b ~]# yum install epel-release -y
[root@0195bc1d0f7b ~]# yum install nginx -y
[root@0195bc1d0f7b ~]# yum install vim wget pcre pcre-devel zlib zlib-devel openssl openssl-devel iproute net-tools iotop -y
```

- 关闭nginx后台运行

```bash
[root@0195bc1d0f7b ~]# vim /etc/nginx/nginx.conf
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;
## daemon off;
```

- 自定义web界面

```bash
[root@0195bc1d0f7b ~]# echo 'Welcome to my nginx' > /usr/share/nginx/html/index.html
```

- 提交为镜像

```bash
[root@docker-server ~]# docker commit --help
Usage:  docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
Create a new image from a container's changes
Options:
  -a, --author string    Author (e.g., "John Hannibal Smith
                         <hannibal@a-team.com>")
  -c, --change list      Apply Dockerfile instruction to the created image
  -m, --message string   Commit message
  -p, --pause            Pause container during commit (default true)
[root@docker-server ~]# docker commit -a "2183677403@qq.com" -m "my nginx image v1" 0195bc1d0f7b centos_nginx:v1
sha256:c29204bfca8d89a06a3bb841cc6ada3c8202798e8f92b5d24c592d48105379d7
[root@docker-server ~]# docker images
REPOSITORY     TAG       IMAGE ID       CREATED             SIZE
centos_nginx   v1        c29204bfca8d   9 seconds ago       386MB
mycentos       v1        acf250a6cabb   About an hour ago   209MB
nginx          latest    d1a364dc548d   2 weeks ago         133MB
hello-world    latest    d1165f221234   3 months ago        13.3kB
centos         latest    300e315adb2f   6 months ago        209MB
```

- docker commit**适用场景：**主要作用是将配置好的一些容器复用，再生成新的镜像。 commit是合并了save、load、export、import这几个特性的一个综合性的命令，它主要做了： 1、将container当前的读写层保存下来，保存成一个新层 2、和镜像的历史层一起合并成一个新的镜像 3、如果原本的镜像有3层，commit之后就会有4层，最新的一层为从镜像运行到commit之间对文件系统的修改。
- 从自己的镜像启动容器

```bash
[root@docker-server ~]# docker run -d -p 80:80 --name my_centos_nginx centos_nginx:v1 /usr/sbin/nginx
```





## 二、DockerFile制作镜像

​		DockerFile可以说是一种可以被Docker程序解释的脚本，DockerFile是由一条条的命令组成的，每条命令对应linux下面的一条命令，Docker程序将这些DockerFile指令再翻译成真正的linux命令，Docker程序读取DockerFile并根据指令生成Docker镜像，相比手动制作镜像的方式，DockerFile更能直观地展示镜像是怎么产生的，有了写好的各种各样的DockerFIle文件，当后期某个镜像有额外的需求时，只要在之前的DockerFile添加或者修改相应的操作即可重新生成新的Docker镜像，避免了重复手动制作镜像的麻烦。

### 指令说明

配置指令

| 指令          | 说明                               |
| ------------- | ---------------------------------- |
| `ARG`         | 定义创建镜像过程中使用的变量       |
| `FROM`        | 指定所创建镜像的基础镜像           |
| `LABEL`       | 为生成的镜像添加元数据标签信息     |
| `EXPOSE`      | 声明镜像内服务监听的端口           |
| `ENV`         | 指定环境变量                       |
| `ENTRYPOINT`  | 指定镜像的默认入口命令             |
| `VOLUME`      | 创建一个数据卷挂载点               |
| `USER`        | 指定运行容器时的用户名或UID        |
| `WORKDIR`     | 配置工作目录                       |
| `ONBUILD`     | 创建子镜像时指定自动执行的操作指令 |
| `STOPSIGNAL`  | 指定退出的信号值                   |
| `HEALTHCHECK` | 配置所启动容器如何进行健康检查     |
| `SHELL`       | 指定默认shell类型                  |

操作指令

| 指令   | 说明                         |
| ------ | ---------------------------- |
| `RUN`  | 运行指定命令                 |
| `CMD`  | 启动容器时指定默认执行的命令 |
| `ADD`  | 添加内容到镜像               |
| `COPY` | 复制内容到镜像               |



### 配置指令

#### ARG

定义创建过程中使用到的变量

比如:`HTTP_PROXY 、HTTPS_PROXY 、FTP_PROXY 、NO_PROXY`不区分大小写



#### FROM

指定所创建镜像的基础镜像

为了保证镜像精简，可以选用体积较小的Alpin或Debian作为基础镜像



**MAINTAINER**

镜像维护者的信息，例如邮箱，电话号码等



#### EXPOSE

声明镜像内服务监听的端口

```bash
EXPOSE 22 80 8443
```

该指令只是起到声明作用，并不会自动完成端口映射



#### ENTRYPOINT

指定镜像的默认入口命令，该入口命令会在启动容器时作为根命令执行，所有传入值作为该命令的参数

支持两种格式

- `ENTRYPOINT ["executable","param1","param2"]`exec调用执行
- `ENTRYPOINT command param1 param2`shell中执行

此时CMD指令指定值将作为根命令的参数

每个DockerFile中只能有一个ENTRYPOINT，当指定多个时只有最后一个起效



#### VOLUME

创建一个数据卷挂载点。

```bash
VOLUME ["/data"]
```



#### WORKDIR

为后续的`RUN 、CMD 、ENTRYPOINT`指令配置工作目录。

```bash
WORKDIR /path/to/workdir
```

可以使用多个`WORKDIR` 指令，后续命令如果参数是相对路径， 则会基于之前命令指定的路径

```bash
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
```

最终路径为`/a/b/c`。因此，为了避免出错，推荐`WORKDIR` 指令中只使用绝对路径。



### 操作指令

#### RUN

运行指定命令

每条RUN指令将在当前镜像基础上执行指定命令，并提交为新的镜像层

当命令较长时可以使用\来换行



#### CMD

CMD指令用来指定启动容器时默认执行的命令

支持三种格式

- `CMD ["executable","param1","param2"]`相当于执行`executable param1 param2`
- `CMD command param1 param2`在默认的shell中执行，提供给需要交互的应用
- `CMD ["param1","param2"]`提供给`ENTRYPOINT`的默认参数

每个Dockerfile 只能有一条`CMD`命令。如果指定了多条命令，只有最后一条会被执行。



#### ADD

添加内容到镜像

```bash
ADD <src> <dest>
```

该命令将复制指定的src路径下内容到容器中的dest路径下

src可以是DockerFIle所在目录的一个相对路径，也可以是一个url，还可以是一个tar

dest可以是镜像内绝对路径，或者相对于工作目录的相对路径



#### COPY

复制内容到镜像。

```bash
COPY <src> <dest>
```

`COPY` 与`ADD`指令功能类似，当使用本地目录为源目录时，推荐使用`COPY`。



## 四、制作nginx镜像

### 1.使用dokcer搭建nginx网站

- 下载镜像

```bash
[root@docker-server ~]# docker pull centos:7
```

- 创建目录环境

```bash
[root@docker-server opt]# mkdir -pv dockerfile/{web/{nginx,a}}
mkdir: 已创建目录 "dockerfile"
mkdir: 已创建目录 "dockerfile/web"
mkdir: 已创建目录 "dockerfile/web/nginx"
```

- 进入到指定目录

```bash
[root@docker-server opt]# cd dockerfile/web/nginx/
[root@docker-server nginx]# pwd
/opt/dockerfile/web/nginx
```

- 下载源码包

```bash
[root@docker-server nginx]# wget http://nginx.org/download/nginx-1.20.1.tar.gz
[root@docker-server nginx]# ls
Dockerfile  nginx-1.20.1.tar.gz
```

- 编写DockerFile

```bash
[root@docker-server nginx]# vim Dockerfile 
# 第一行先定义基础镜像，后面的本地有效的镜像名，如果本地没有，会从远程仓库下载
FROM centos:7

# 镜像维护者的信息
MAINTAINER hugo 2763743788@qq.com

# 将编译安装nginx的步骤执行一遍
RUN yum install -y vim wget tree lrzsz gcc gcc-c++ automake pcre pcre-devel zlib zlib-devel openssl openssl-devel iproute net-tools iotop

# 上传nginx压缩包
ADD nginx-1.20.1.tar.gz /usr/local/src/

RUN cd /usr/local/src/nginx-1.20.1 \
&& ./configure --prefix=/usr/local/nginx --with-http_sub_module \
&& make \
&& make install \
&& cd /usr/local/nginx 

# 可以添加自己事先准备的配置文件
# ADD nginx.conf /usr/local/nginx/conf/nginx.conf

RUN useradd -s /sbin/nologin nginx \
&& ln -sv /usr/local/nginx/sbin/nginx /usr/sbin/nginx \
&& echo 'test nginx !' > /usr/local/nginx/html/index.html

# 声明端口号
EXPOSE 80 443

CMD ["nginx", "-g", "daemon off;"]
```

- 构建镜像

```bash
[root@docker-server nginx]# docker build -t nginx:v1 .
[root@docker-server nginx]# docker images | grep v1
nginx          v1        fbd06c1753c0   8 seconds ago   581MB
```

- 测试

```bash
[root@docker-server nginx]# docker run -d -it -p 80:80 nginx:v1
```



### 2.基于rocklinux9使用docker搭建一个小游戏合集网站

#### 环境准备

一、下载镜像

```bash
[root@localhost ~]# docker pull rockylinux:9
```

二、创建所需文件存放的目录环境

```bash
[root@localhost ~]# mkdir -pv dockerfile/nginx
mkdir: created directory 'dockerfile'
mkdir: created directory 'dockerfile/nginx'
```

三、进入到指定目录中

```bash
[root@localhost ~]# cd dockerfile/nginx/
[root@localhost nginx]# pwd
/root/dockerfile/nginx
```

四、下载nginx和小游戏网页的源码包

```bash
[root@localhost nginx]# wget http://nginx.org/download/nginx-1.22.0.tar.gz
[root@localhost nginx]# wget http://file.eagleslab.com:8889/%E8%AF%BE%E7%A8%8B%E7%9B%B8%E5%85%B3%E8%BD%AF%E4%BB%B6/%E4%BA%91%E8%AE%A1%E7%AE%97%E8%AF%BE%E7%A8%8B/%E8%AF%BE%E7%A8%8B%E7%9B%B8%E5%85%B3%E6%96%87%E4%BB%B6/games.tar.gz
[root@localhost nginx]# ll
total 125100
-rw-r--r--. 1 root root 127022187 Jan 18  2022 games.tar.gz
-rw-r--r--. 1 root root   1073322 May 24  2022 nginx-1.22.0.tar.gz
```



#### 编写Dockerfile

一、编写DockerFile

```bash
[root@docker-server nginx]# vim Dockerfile 
# 第一行先定义基础镜像，后面的本地有效的镜像名，如果本地没有，会从远程仓库下载
FROM rockylinux:9

# 镜像作者的信息
MAINTAINER Eagle_nls 2898485992@qq.com

# 接下来在基础镜像之上构建nginx等所需环境

# 1. 编译安装nginx
# 1.1 安装所需环境及工具
RUN yum install -y vim wget unzip tree lrzsz gcc gcc-c++ automake pcre pcre-devel zlib zlib-devel openssl openssl-devel iproute net-tools iotop

# 1.2 上传nginx的官方源码包到指定目录
ADD nginx-1.22.0.tar.gz /usr/local/src/

# 1.3 由于ADD会直接解压好，所以直接编译nginx
RUN cd /usr/local/src/nginx-1.22.0 \
&& ./configure --prefix=/usr/local/nginx --with-http_sub_module \
&& make \
&& make install \
&& cd /usr/local/nginx

# 如果有配置文件，可以上传自己准备好的配置文件
# ADD nginx.conf /usr/local/nginx/conf/nginx.conf

# 2. 完善网站
RUN useradd -s /sbin/nologin nginx \
&& ln -sv /usr/local/nginx/sbin/nginx /usr/sbin/nginx

# 3. 上传小游戏网页源码
ADD games.tar.gz /usr/local/nginx/html/

# 4. 声明端口号
EXPOSE 80 443

# 5. 设定启动时执行命令
CMD ["nginx", "-g", "daemon off;"]
```



#### 构建镜像

一、通过docker build来构建镜像

```bash
[root@localhost nginx]# docker build -t nginx_games:v1 .
[root@localhost nginx]# docker images |grep nginx_games
nginx_games                                         v1         9c86d96dfba0   4 minutes ago       686MB
```

二、镜像运行测试

```bash
[root@localhost nginx]# docker run -d -it -p 80:80 --name nginx_games nginx_games:v1
e3c415425f95b3deac80ecb772d1de2a89e18a611c2ecc603f3c0b175bbea335
[root@localhost nginx]# docker ps
CONTAINER ID   IMAGE            COMMAND                  CREATED         STATUS         PORTS                                          NAMES
e3c415425f95   nginx_games:v1   "nginx -g 'daemon of…"   3 seconds ago   Up 2 seconds   0.0.0.0:80->80/tcp, [::]:80->80/tcp, 443/tcp   nginx_games
```

三、访问测试



dokcer案例：https://blog.51cto.com/u_14080162/2464376



## 五、镜像上传

### 官方docker仓库

- 准备账户

登陆到docker hub官网创建账号，登陆后点击settings完善信息

- 填写账户基本信息

- 登陆仓库

```bash
[root@docker-server ~]# docker login docker.io
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: smqy
Password:
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
[root@docker-server ~]# ls -a 
.                .bash_history  .bashrc  dockerfile    .tcshrc
..               .bash_logout   .cshrc   docker_in.sh  .viminfo
anaconda-ks.cfg  .bash_profile  .docker  .pki
[root@docker-server ~]# cat .docker/config.json 
{
    "auths": {
        "https://index.docker.io/v1/": {
            "auth": "YmJqMTAzMDp6aGFuZ2ppZTEyMw=="
        }
    }
}[root@docker-server ~]#
```

- 给镜像tag标签并上传

```bash
[root@docker-server ~]# docker tag nginx:v1 docker.io/smqy/nginx:v1
[root@docker-server ~]# docker images
REPOSITORY     TAG       IMAGE ID       CREATED          SIZE
nginx          v1        fbd06c1753c0   12 minutes ago   581MB
smqy/nginx     v1        fbd06c1753c0   12 minutes ago   581MB
centos_nginx   v1        74acdcca8c97   21 hours ago     525MB
nginx          latest    2b7d6430f78d   3 weeks ago      142MB
centos         7         eeb6ee3f44bd   12 months ago    204MB
nginx          1.8       0d493297b409   6 years ago      133MB
[root@docker-server ~]# docker push docker.io/smqy/nginx:v1
The push refers to repository [docker.io/smqy/nginx]
4f86a3bf507f: Pushed
0d11c01ff3ef: Pushed
45fc5772a4e4: Pushed
174f56854903: Mounted from library/centos
v1: digest: sha256:385bfe364839b645f1b2aa70a1d779b0dca50256ea01ccbe4ebde53aabd1d96d size: 1164
```

- 到docker官网进行验证

- 更换到其他docker服务器下载镜像

```bash
[root@docker-server ~]# docker login docker.io
```



### 阿里云仓库

将本地镜像上传至阿里云，实现镜像备份与统一分发的功能

网址：https://cr.console.aliyun.com/

注册账户、创建namespace、创建仓库、修改镜像tag以及上传镜像

登录阿里云Docker Registry

```bash
$ docker login --username=用户名 registry.cn-hangzhou.aliyuncs.com
Password:
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

用于登录的用户名为阿里云账号全名，密码为开通服务时设置的密码。

在访问凭证页面修改凭证密码。



#### 从Registry中拉取镜像

```bash
$ docker pull registry.cn-hangzhou.aliyuncs.com/smqy/nginx:[镜像版本号]
```

#### 将镜像推送到Registry

```bash
$ docker login --username=用户名 registry.cn-hangzhou.aliyuncs.com
$ docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/smqy/nginx:[镜像版本号]
# docker tag  centos_nginx:v1  registry.cn-hangzhou.aliyuncs.com/smqy/nginx:v1
$ docker push registry.cn-hangzhou.aliyuncs.com/smqy/nginx:[镜像版本号]
docker push registry.cn-hangzhou.aliyuncs.com/smqy/nginx:v1
The push refers to repository [registry.cn-hangzhou.aliyuncs.com/smqy/nginx]
5d90084f25f2: Pushed
174f56854903: Pushed
v1: digest: sha256:0d4a376eeb6270baf8739b5bdca1f877e08fc31442ee38cea0fad9fdb01ed45a size: 742
```



**注意事项：**

选择合适的镜像仓库地址

从ECS推送镜像时，可以选择使用镜像仓库内网地址。推送速度将得到提升并且将不会损耗您的公网流量。

如果使用的机器位于VPC网络，需要使用 registry-vpc.cn-hangzhou.aliyuncs.com 作为Registry的域名登录。

示例：使用"docker tag"命令重命名镜像，并将它通过专有网络地址推送至Registry。

```bash
$ docker images
REPOSITORY       TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
registry.cn-hangzhou.aliyuncs.com/smqy/nginx   v1        74acdcca8c97   22 hours ago     525MB
```

使用 "docker push" 命令将该镜像推送至远程。

```bash
$ docker push registry-vpc.cn-hangzhou.aliyuncs.com/acs/agent:0.7-dfb6816
```



# 五、docker数据管理

如果正在运行中的容器修改了已经存在的文件内容或者生成了新的内容，那么新产生的数据将会被复制到读写层。

Docker的镜像是分层设计的，镜像层是只读的，通过镜像启动的容器添加了一层**可读写的文件系统**，用户写入的数据都保存在这一层当中。

如果要将写入到容器的数据永久保存，则需要将容器中的数据保存到宿主机的指定目录，目前Docker的数据类型分为两种分别是数据卷和数据卷容器。

参考联合文件系统文章：https://mp.weixin.qq.com/s?__biz=MzI1OTY2MzMxOQ==&mid=2247495888&idx=1&sn=39ed455b12cc2e3ad18f5f72c8e6cc21&chksm=ea77c468dd004d7e42aa4a9c095822e41fd3fa01a08156181615fb66f724bc0d03ff228a39bd&mpshare=1&scene=23&srcid=0725FohVQp49EcJQS0khWpUC&sharer_sharetime=1635037154550&sharer_shareid=ac5ffb01d260d9cefd367fe5efc4eb13#rd

```bash
[root@docker-server1 ~]# docker inspect nginx:latest 
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/01ad296a438f93e5cb283b6b462ca8da325754c6958c741221979e783f48a82c/diff:/var/lib/docker/overlay2/3639172ee8e2375e9c5d5843d4f3d9da343574bc6ebface8ee3735691fca04ea/diff:/var/lib/docker/overlay2/fdd0aea0a652268ce776642f9dfb67e221015259400579bd4d01da06931ac015/diff:/var/lib/docker/overlay2/1c85c85b62d3d884b51d4c7cebcb7c135c3f8cc00afd1704526b3a5f22f3723f/diff:/var/lib/docker/overlay2/5b2cab96eb3cfea207ddcc5750e9d950437ecdb27f912091d4e467550c2bc775/diff",
                "MergedDir": "/var/lib/docker/overlay2/0e4278c3cbb691146f01f5f69e5cc48199a1b92bda3d78c54cde984bf95921ee/merged",
                "UpperDir": "/var/lib/docker/overlay2/0e4278c3cbb691146f01f5f69e5cc48199a1b92bda3d78c54cde984bf95921ee/diff",
                "WorkDir": "/var/lib/docker/overlay2/0e4278c3cbb691146f01f5f69e5cc48199a1b92bda3d78c54cde984bf95921ee/work"
            },
            "Name": "overlay2"
LoweDir：image镜像层本身（只读）
UpperDir：容器的上层读写层
MergeDir：容器的文件系统，使用Union FS(联合文件系统)将镜像层和容器层合并给容器使用
WorkDir：容器在宿主机的工作目录
```

- 在容器中创建一个文件，观察文件

```bash
root@f59956d6babc:/# touch eagleslab.txt 
root@f59956d6babc:/# md5sum eagleslab.txt 
d41d8cd98f00b204e9800998ecf8427e  eagleslab.txt
[root@docker-server1 ~]# find / -name eagles*
/var/lib/docker/overlay2/5c16401d71448d83bb51efc896201e326b74f50671380f49a995e3d37e1d2363/diff/eagleslab.txt
/var/lib/docker/overlay2/5c16401d71448d83bb51efc896201e326b74f50671380f49a995e3d37e1d2363/merged/eagleslab.txt
[root@docker-server1 ~]# find / -name eagles*
/var/lib/docker/overlay2/5c16401d71448d83bb51efc896201e326b74f50671380f49a995e3d37e1d2363/diff/eagleslab.txt
/var/lib/docker/overlay2/5c16401d71448d83bb51efc896201e326b74f50671380f49a995e3d37e1d2363/merged/eagleslab.txt
[root@docker-server1 ~]# md5sum /var/lib/docker/overlay2/5c16401d71448d83bb51efc896201e326b74f50671380f49a995e3d37e1d2363/diff/eagleslab.txt
d41d8cd98f00b204e9800998ecf8427e  /var/lib/docker/overlay2/5c16401d71448d83bb51efc896201e326b74f50671380f49a995e3d37e1d2363/diff/eagleslab.txt
[root@docker-server1 ~]# md5sum /var/lib/docker/overlay2/5c16401d71448d83bb51efc896201e326b74f50671380f49a995e3d37e1d2363/merged/eagleslab.txt
d41d8cd98f00b204e9800998ecf8427e  /var/lib/docker/overlay2/5c16401d71448d83bb51efc896201e326b74f50671380f49a995e3d37e1d2363/merged/eagleslab.txt
```

- 删除容器后发现文件消失

```bash
[root@docker-server1 ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS     NAMES
f59956d6babc   nginx     "/docker-entrypoint.…"   16 minutes ago   Up 16 minutes   80/tcp    upbeat_greider
[root@docker-server1 ~]# docker rm -fv f59956d6babc
f59956d6babc
[root@docker-server1 ~]# find / -name eagles*
```



## UnionFS（联合文件系统）

UnionFS，全称为联合文件系统（Union File System），是一种用于合并多个不同文件系统的服务。它将多个独立的文件系统层（Layer）联合为一个单一的文件系统视图，并为用户提供读写访问功能。UnionFS的核心思想在于通过分层管理文件系统，使得不同层的内容可以独立存在，又可以联合形成一个完整的系统。


### 核心特点

UnionFS的核心在于两大技术概念：**分层文件系统**和**写时复制**。这两者的结合使得Docker能够高效地管理和使用镜像，优化存储资源并提升容器运行的性能。

1. 分层文件系统

  UnionFS的分层文件系统通过将多个文件系统层次化处理，形成一个联合文件系统视图。在Docker的上下文中，每个镜像由多个层（Layer）组成，而每个层都是一个文件系统。这些层按照构建顺序叠加，最底层是基础镜像（如ubuntu:latest），上面是从基础镜像扩展出来的层，直到形成完整的应用程序镜像。

​	分层文件系统的优势：
​		高效的存储管理：通过分层，Docker镜像可以避免数据冗余，不同的容器可以共享相同的镜像层，减少磁盘空间的占用。
​		快速的镜像构建与部署：因为每个镜像层都是只读且可以缓存，Docker在构建新镜像时，无需重复构建已经存在的层，从而显著提升镜像构建速度。
​		模块化管理：每个层都是一个独立的模块，镜像的更新可以通过新增或修改上层镜像层的方式来完成，而不影响底层。

2. 写时复制（Copy-on-Write）
写时复制是UnionFS中的另一个关键概念。Docker的镜像层是只读的，当容器运行时，如果需要修改某个文件，Docker并不会直接修改镜像层中的文件，而是通过写时复制的机制，先将文件复制到容器的可写层中，容器对该文件的操作只会在这个可写层上进行，而不会影响到其他容器共享的只读层。

这种写时复制的机制不仅保证了镜像层的完整性，还提供了文件共享和隔离的能力：

文件共享与隔离：多个容器可以基于同一个镜像运行，并共享相同的只读层。即使一个容器对某个文件进行了修改，其他容器仍然可以使用该文件的原始版本。
高效的文件操作：Docker只需对需要修改的文件进行操作，而不必重新构建整个文件系统，极大地提高了文件操作的效率。



### 在 Docker 中的应用

UnionFS为Docker的镜像和容器管理提供了强大的底层支持，它在Docker的文件管理和存储机制中扮演着关键角色。

**Docker镜像的分层构建**
每个Docker镜像由多个只读层组成，这些层代表了镜像构建过程中的每个操作。例如，以下Dockerfile：

```bash
FROM ubuntu:latest
RUN apt-get update
RUN apt-get install -y nginx
```

上述文件表示一个分层构建的过程：

第一层：ubuntu:latest镜像，这是一个基础镜像层，通常从官方镜像仓库中拉取；
第二层：apt-get update操作在基础镜像上运行，并生成一个新的只读层；
第三层：apt-get install -y nginx安装了Nginx服务器，又生成了一个新的层。
Docker将这些层通过UnionFS叠加成一个统一的文件系统，形成最终的镜像。当镜像构建完成后，用户只会看到镜像的整体内容，而每个层的内部结构被UnionFS隐藏在背后。每一层都是基于前一层构建的，新的层只包含与前一层的差异部分。

**容器的写时层（Writable Layer）**
当用户基于某个镜像启动一个容器时，Docker会在该镜像之上添加一个可写层，所有对文件系统的修改都会在这个可写层上完成，而不会影响镜像本身。

例如，如果用户在容器中修改了某个文件，Docker会先将该文件从只读层复制到可写层中，接着所有对该文件的操作都在可写层中完成，而原始的只读层文件保持不变。这种方式不仅保证了镜像的完整性，还可以避免在多个容器之间产生文件冲突。

**存储驱动的支持**
Docker中的UnionFS是通过不同的存储驱动实现的，常见的驱动包括：

- OverlayFS：一种现代Linux内核支持的联合文件系统，Docker中使用最广泛的存储驱动，具有较好的性能和简单的设计。

- Aufs（Another Union File System）：Docker最早期使用的存储驱动，具有良好的分层文件系统支持。

- Btrfs：支持高级功能如快照和子卷管理，但在某些情况下性能表现不如OverlayFS。

  不同的存储驱动在性能、稳定性和功能特性上各有差异，Docker默认使用OverlayFS作为主要存储驱动，但用户可以根据自己的需求选择其他合适的存储驱动。
  

### 常见的 UnionFS 实现

虽然 UnionFS 是一个理念，但 Linux 内核中有多种具体的实现方式，Docker 默认使用的是 Overlay2。Overlay2 是一种先进的联合文件系统实现，具有更好的性能和一些先进的功能，如页面缓存共享。

##### 镜像层与可写层

在 Docker 中，镜像层和容器可写层是基于联合文件系统（UnionFS）实现的关键概念。它们共同构成了容器的文件系统结构，使得 Docker 能够高效地管理和运行容器。以下是对镜像层和容器可写层的详细解释：

##### 一、镜像层

镜像层是 Docker 镜像的组成部分，每个镜像由多个只读层组成，这些层是不可修改的。

**镜像层的特点**
**只读性**：镜像层是只读的，一旦创建就不能修改。如果需要修改镜像层中的内容，必须通过构建新的镜像层来实现。
**共享性**：多个容器可以共享同一个镜像层，因为镜像层是不可变的，这大大节省了磁盘空间。
**不可变性**：镜像层的不可变性保证了镜像的一致性和可重复性，无论在什么环境下运行，基于同一镜像启动的容器都具有相同的文件系统结构。

**镜像层的作用**
**构建高效**：通过分层构建，Docker 可以缓存中间层，避免重复构建相同的操作。当基础镜像层和安装 Nginx 的层已经构建过，再次构建时可以直接使用缓存的层，而不是重新执行命令。
**节省空间**：多个容器可以共享镜像层，减少了磁盘空间的占用。例如，10 个基于同一个基础镜像的容器，只需要存储一份基础镜像层，而不是每份都单独存储。
**便于分发**：镜像层的结构使得镜像可以方便地分发和共享。用户可以从 Docker Hub 等镜像仓库拉取镜像，而无需关心镜像的具体构建过程。



##### 二、可写层

可写层是容器运行时特有的一个层，它是可写的，用于存储容器运行时产生的所有变更。

可写层的生成

当启动一个容器时，Docker 会在镜像的最顶层添加一个新的可写层。这个可写层是容器独有的，用于存储容器运行时的文件系统变更。当在容器中创建了一个新文件、修改了一个现有文件或删除了一个文件，这些操作都会反映在容器可写层中，而不会影响到镜像层。

可写层的特点
**可写性**：容器可写层是可写的，容器运行时的所有文件系统操作（如创建、修改、删除文件）都在这个层中进行。
**独立性**：每个容器都有自己的可写层，容器之间的可写层是隔离的。一个容器的变更不会影响到其他容器。
**临时性**：容器可写层的内容在容器被删除时也会被删除。如果需要持久化数据，需要将数据存储在外部存储（如卷）中。

可写层的作用
**隔离性**：容器可写层保证了容器之间的隔离性。每个容器在自己的可写层中进行操作，不会影响到其他容器或镜像层。
**灵活性**：容器可写层允许容器在运行时动态地修改文件系统，而不会影响到镜像的原始状态。这使得容器可以灵活地运行各种应用程序。
**数据持久化**：虽然容器可写层的内容在容器删除时会被删除，但你可以通过挂载卷（Volumes）将数据持久化到宿主机或其他存储设备中，从而实现数据的持久化存储。



##### 三、镜像层与可写层的关系

镜像层和容器可写层通过联合文件系统（UnionFS）联合起来，形成了容器的完整文件系统视图。

1. 联合挂载

   Docker 使用联合文件系统将镜像层和容器可写层联合挂载到同一个目录下。从容器的角度来看，它看到的是一个完整的文件系统，包含了镜像层和容器可写层的内容。

   当你在容器中访问一个文件时，Docker 会先在容器可写层中查找该文件。如果找不到，就会在镜像层中查找。如果在镜像层中找到了文件，就会将该文件的内容返回给容器。

2. 写时复制（Copy-on-Write）

   当容器需要修改一个文件时，Docker 会使用写时复制机制。具体来说，Docker 会将文件从镜像层复制到容器可写层，然后在可写层中进行修改。这样，镜像层保持不变，而容器可写层存储了修改后的文件。

   假设镜像层中有一个文件 `/etc/nginx/nginx.conf`，容器需要修改这个文件。Docker 会将该文件从镜像层复制到容器可写层，然后在可写层中进行修改。从容器的角度来看，它看到的是修改后的文件，而镜像层中的文件保持不变。

3. 删除操作

   当容器删除一个文件时，Docker 会在容器可写层中记录一个删除操作，而不是真正删除镜像层中的文件。这样，镜像层保持不变，而容器可写层记录了文件的删除状态。

   假设容器删除了一个文件 `/etc/nginx/nginx.conf`，Docker 会在容器可写层中记录一个删除标记。从容器的角度来看，该文件已经被删除，而镜像层中的文件仍然存在。



- **镜像层**：是 Docker 镜像的组成部分，是只读的、不可变的，多个容器可以共享镜像层。镜像层通过分层构建，提高了构建效率、节省了磁盘空间，并保证了镜像的一致性和可重复性。
- **可写层**：是容器运行时的可写层，用于存储容器运行时的文件系统变更。容器可写层是独立的、可写的、临时的，保证了容器之间的隔离性和运行时的灵活性。
- **联合文件系统（UnionFS）**：将镜像层和可写层联合挂载，形成了容器的完整文件系统视图。通过写时复制机制，Docker 在不影响镜像层的情况下，允许容器在可写层中进行文件系统操作。



### 查看镜像的详细信息

#### 一、查看镜像数据信息

```bash
[root@localhost nginx]# docker inspect nginx_games:v1
"Architecture": "amd64",
        "Os": "linux",
        "Size": 685997316,
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/ilm8ifbi2gcrqg156w7kbu22f/diff:/var/lib/docker/overlay2/u51fr5eul9csiwghxmnm1acsi/diff:/var/lib/docker/overlay2/lo6sgoxbliga9l2qqqxcu2tvp/diff:/var/lib/docker/overlay2/bi4orr0yujicfufabnw7u8tym/diff:/var/lib/docker/overlay2/e51d5c1c74a99e1b1e41980bb9270fcba562976e40ae7dbf65ff585689a783d2/diff",
                "MergedDir": "/var/lib/docker/overlay2/talgtlohecyw3kt8d8idpzw4g/merged",
                "UpperDir": "/var/lib/docker/overlay2/talgtlohecyw3kt8d8idpzw4g/diff",
                "WorkDir": "/var/lib/docker/overlay2/talgtlohecyw3kt8d8idpzw4g/work"

LoweDir：image镜像层本身（只读）
UpperDir：容器的上层读写层
MergeDir：容器的文件系统，使用Union FS(联合文件系统)将镜像层和容器层合并给容器使用
WorkDir：容器在宿主机的工作目录
```

#### 二、查看镜像层构建过程

```bash
[root@localhost nginx]# docker history nginx_games:v1
IMAGE          CREATED         CREATED BY                                      SIZE      COMMENT
9c86d96dfba0   25 hours ago    CMD ["nginx" "-g" "daemon off;"]                0B        buildkit.dockerfile.v0
<missing>      25 hours ago    EXPOSE map[443/tcp:{} 80/tcp:{}]                0B        buildkit.dockerfile.v0
<missing>      25 hours ago    ADD games.tar.gz /usr/local/nginx/html/ # bu…   164MB     buildkit.dockerfile.v0
<missing>      25 hours ago    RUN /bin/sh -c useradd -s /sbin/nologin ngin…   297kB     buildkit.dockerfile.v0
<missing>      25 hours ago    RUN /bin/sh -c cd /usr/local/src/nginx-1.22.…   16.6MB    buildkit.dockerfile.v0
<missing>      25 hours ago    ADD nginx-1.22.0.tar.gz /usr/local/src/ # bu…   6.46MB    buildkit.dockerfile.v0
<missing>      25 hours ago    RUN /bin/sh -c yum install -y vim wget unzip…   323MB     buildkit.dockerfile.v0
<missing>      25 hours ago    MAINTAINER Eagle_nls 2898485992@qq.com          0B        buildkit.dockerfile.v0
<missing>      16 months ago   CMD ["/bin/bash"]                               0B        buildkit.dockerfile.v0
<missing>      16 months ago   ADD layer.tar.xz / # buildkit                   176MB     buildkit.dockerfile.v0
```

如果想要缩小所构建镜像的大小，我们可以尝试少创建镜像层，在一层中多做一些事情，尽可能减少**RUN命令**



## 数据卷

### 什么是数据卷？

数据卷实际上就是宿主机上的目录或者是文件，可以被直接mount到容器当中使用。

实际生产环境中，需要针对不同类型的服务、不同类型的数据存储要求做相应的规划，最终保证服务的可扩展性、稳定性以及数据的安全性。

![docker数据卷](E:\英格\图片\docker数据卷.png)



### 数据卷案例

- 创建目录并准备页面

```bash
[root@docker-server1 ~]# mkdir -p /data/web
[root@docker-server1 ~]# echo 'eaglslab nginx test!' > /data/web/index.html
[root@docker-server1 ~]# cat /data/web/index.html 
eaglslab nginx test
```

- 启动两个容器并验证数据

```bash
[root@docker-server1 ~]# docker run -d -it --name web1 -v /data/web/:/usr/share/nginx/html/ -p 8080:80 nginx
588c494dc9098e0a43e15bce3162c34676dd981609edc32d46bf4beb59b9cf19
[root@docker-server1 ~]# docker run -d -it --name web2 -v /data/web/:/usr/share/nginx/html/ -p 8081:80 nginx
ff6b3731a9ba3e0f91d2c8d89bb6573eb5e5a9b840163bc1122a9e5678d108b7
[root@docker-server1 ~]# curl 192.168.175.10:8080
eaglslab nginx test!
[root@docker-server1 ~]# curl 192.168.175.10:8081
eaglslab nginx test!
[root@docker-server1 ~]# echo 'hello world!' > /data/web/index.html 
[root@docker-server1 ~]# curl 192.168.175.10:8080
hello world!
[root@docker-server1 ~]# curl 192.168.175.10:8081
hello world!
```

- 进入到容器内测试写入数据

```bash
[root@docker-server1 ~]# docker exec -it web1 bash
root@588c494dc909:/# echo 'docker test!' > /usr/share/nginx/html/index.html 
[root@docker-server1 ~]# curl 192.168.175.10:8080
docker test!
[root@docker-server1 ~]# curl 192.168.175.10:8081
docker test!
```

- 尝试只读挂载

```bash
# 删除容器的时候不会删除宿主机的目录
[root@docker-server1 ~]# docker rm -fv web1
web1
[root@docker-server1 ~]# docker rm -fv web2
web2
[root@docker-server1 ~]# cat /data/web/index.html 
docker test!
# 通过只读方式挂载以后，在容器内部是不允许修改数据的
[root@docker-server1 ~]# docker run -d -it --name web1 -v /data/web/:/usr/share/nginx/html/:ro -p 8080:80 nginx
a395b27958ca0cdcf52a86bd17813dcbcda4ed774895adcc99e85fc114ab84ff
[root@docker-server1 ~]# docker exec -it web1 bash
root@a395b27958ca:/# echo 123 > /usr/share/nginx/html/index.html 
bash: /usr/share/nginx/html/index.html: Read-only file system
```

- 文件挂载

```bash
[root@docker-server1 ~]# docker run -d -it --name web2 -v /data/web/index.html:/usr/share/nginx/html/index.html:ro -p 8081:80 nginx
4b34c957372d314cdb0f85d7e2c65b095615adfe3051ae6b4266b7bacd50f374
[root@docker-server1 ~]# curl 192.168.175.10:8081
docker test!
```



### 数据卷特点

1. 数据卷是宿主机的目录或者文件，并且可以在**多个容器之间共同使用**
2. 在宿主机对数据卷更改数据后会在所有容器里面会**立即更新**
3. 数据卷的数据可以**持久保存**，即使删除使用该数据卷卷的容器也不影响
4. 在容器里面写入数据不会影响到镜像本身(**隔离性**)。
5. 需要挂载多个目录或者文件的时候可以使用多个-v参数指定
6. 数据卷使用场景包括日志输出、静态web页面、应用配置文件、多容器间目录或文件共享



## 数据卷容器

数据卷容器功能是可以让数据在多个docker容器之间共享，即可以让B容器访问A容器的内容，而容器c也可以访问A容器的内容，即先要创建一个后台运行的容器作为Server，用于卷提供，这个卷可以为其他容器提供数据存储服务，其他使用此卷的容器作为客户端。

### 数据卷容器的用途

1. **数据持久化**：数据卷容器可以确保数据即使在容器被删除后也不会丢失。例如，对于数据库应用，数据卷容器可以持久化数据库文件，避免因容器删除而导致数据丢失。
2. **数据共享**：多个容器可以通过挂载同一个数据卷容器来共享数据。这在多容器应用中非常有用，例如在微服务架构中，多个服务容器可以共享数据库数据。
3. **备份与恢复**：数据卷容器可以用于备份和恢复数据。通过将数据卷容器中的数据卷备份到宿主机或其他存储设备，可以在需要时恢复数据。
4. **迁移数据**：数据卷容器可以方便地迁移数据。通过将数据卷容器中的数据卷迁移到其他主机，可以快速实现数据的迁移。

### 数据卷容器与本地数据挂载的区别

| 特性                   | 数据卷容器                                                   | 本地数据挂载                                                 |
| :--------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| **目录来源**           | Docker 自动创建的目录                                        | 宿主机上已存在的目录                                         |
| **适用场景**           | 数据持久化、容器间共享数据                                   | 开发调试、快速文件同步                                       |
| **容器间共享**         | 支持                                                         | 不支持                                                       |
| **宿主机文件系统依赖** | 无依赖                                                       | 依赖宿主机文件系统                                           |
| **数据持久化**         | 数据卷的生命周期独立于容器，即使容器被删除，数据卷仍然存在   | 如果容器被删除，挂载的目录和文件仍然存在，但需要手动管理     |
| **数据初始化**         | 如果宿主机没有对应的文件，数据卷容器会自动将容器运行所需的文件复制到数据卷中 | 如果宿主机没有对应的文件，容器可能会因缺少运行所需的文件而出错 |
| **数据覆盖**           | 数据卷中的数据会覆盖容器内的数据                             | 容器内的数据会覆盖宿主机上的数据                             |



### 实例

- 先启动一个卷容器Server

```bash
[root@docker-server1 ~]# docker run -d -it --name nginx-web -v /data/web/:/usr/share/nginx/html/:ro -p 8081:80 nginx
```

- 启动两个客户端容器

```bash
[root@docker-server1 ~]# docker run -d -it --name web1 -p 8082:80 --volumes-from nginx-web nginx:latest 
ac22faa405ec07c065042465cd7f9d456be891effdd5d13d9571b96ef9c550f7
[root@docker-server1 ~]# docker run -d -it --name web2 -p 8083:80 --volumes-from nginx-web nginx:latest 
e084845475b01dedfdae7362f6fbece7b5ab57ff6289c8c9bf08251f5ba448ed
```

- 访问测试

```bash
[root@docker-server1 ~]# curl 192.168.175.10:8081
docker test!
[root@docker-server1 ~]# curl 192.168.175.10:8082
docker test!
[root@docker-server1 ~]# curl 192.168.175.10:8083
docker test!
```

- 停止卷容器可以创建新容器

```bash
[root@docker-server1 ~]# docker stop nginx-web 
nginx-web
[root@docker-server1 ~]# docker run -d -it --name web3 -p 8084:80 --volumes-from nginx-web nginx:latest 
6ebd95c132ee1a9e4b43d1849efc628ca7185187a59d70b3816ff16dd47b6e8e
[root@docker-server1 ~]# curl 192.168.175.10:8084
docker test!
```

- 删除卷容器之后不可以再创建新容器

```bash
[root@docker-server1 ~]# docker rm -fv nginx-web 
nginx-web
[root@docker-server1 ~]# docker run -d -it --name web4 -p 8085:80 --volumes-from nginx-web nginx:latest 
docker: Error response from daemon: No such container: nginx-web.
See 'docker run --help'.
# 但是之前已经创建好的容器不会有任何影响
[root@docker-server1 ~]# curl 192.168.175.10:8082
docker test!
```

总结：

在当前环境下，即使把提供卷的容器Server删除，已经运行的容器Client依然可以使用挂载的卷，因为容器是通过挂载的方式访问数据的，但是无法创建新的卷容器客户端，但是再把卷容器Server创建后即可正常创建卷容器client，此方式可以用于线上共享数据目录等环境，因为即使数据卷容器被删除了，其他已经运行的容器依然可以挂载使用

数据卷容器可以作为共享的方式为其他容器提供文件共享，可以在容器生成中启动一个实例挂载本地的目录，然后其他的容器分别挂载此容器的目录，即可保证各个容器之间的数据一致性。



# 六、docker网络管理

## 一、容器之间的互联

在同一个宿主机上的容器间可以通过端口映射的方式，经过宿主机中转进行互相访问，也可以通过docker0网桥互相访问

### 直接互联

- 启动两个容器

```bash
[root@docker-server1 ~]# docker run -d -it nginx
[root@docker-server1 ~]# docker run -d -it nginx
```

- 安装相关工具包

```bash
root@855ab8d0bd74:/# apt update
root@855ab8d0bd74:/# apt install net-tools -y
root@855ab8d0bd74:/# apt install iputils-ping -y
root@855ab8d0bd74:/# apt install procps -y
```

- 检测网络连通性

```bash
root@855ab8d0bd74:/# ping 172.17.0.3 -c 2
PING 172.17.0.3 (172.17.0.3) 56(84) bytes of data.
64 bytes from 172.17.0.3: icmp_seq=1 ttl=64 time=0.052 ms
64 bytes from 172.17.0.3: icmp_seq=2 ttl=64 time=0.081 ms

--- 172.17.0.3 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1ms
rtt min/avg/max/mdev = 0.052/0.066/0.081/0.016 ms
```



### 使用名称互联

- 启动两个容器

```bash
[root@docker-server1 ~]# docker run -d -it --name web1 nginx:1.8
[root@docker-server1 ~]# docker run -d -it --name web2 --link web1 nginx:1.8
```

- 查看web2容器的hosts文件，发现已经实现名称解析

```bash
[root@docker-server1 ~]# docker exec -it web2 bash
root@8a3e9cee9e37:/# cat /etc/hosts
127.0.0.1    localhost
::1    localhost ip6-localhost ip6-loopback
fe00::0    ip6-localnet
ff00::0    ip6-mcastprefix
ff02::1    ip6-allnodes
ff02::2    ip6-allrouters
172.17.0.2    web1 622eff54876f
172.17.0.3    8a3e9cee9e37
```

- 连通性测试

```bash
root@8a3e9cee9e37:/# ping web1 -c 2
PING web1 (172.17.0.2) 56(84) bytes of data.
64 bytes from web1 (172.17.0.2): icmp_seq=1 ttl=64 time=0.068 ms
64 bytes from web1 (172.17.0.2): icmp_seq=2 ttl=64 time=0.045 ms

--- web1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1ms
rtt min/avg/max/mdev = 0.045/0.056/0.068/0.013 ms
```



### 使用别名互联

​		自定义的容器名称可能后期会发生变化，那么一旦发生变化也会带来一些影响，这个时候如果每次都更改名称又比较麻烦，这个时候可以使用定义别名的方式解决，即容器名称可以随意更改，只要不更改别名即可。

- 启动一个容器

```bash
[root@docker-server1 ~]# docker run -d -it --name web3 --link web1:nginx-web1 nginx:1.8
```

- 查看容器web3的hosts文件

```bash
[root@docker-server1 ~]# docker exec -it web3 bash
root@c85c73ebf00b:/# cat /etc/hosts
127.0.0.1    localhost
::1    localhost ip6-localhost ip6-loopback
fe00::0    ip6-localnet
ff00::0    ip6-mcastprefix
ff02::1    ip6-allnodes
ff02::2    ip6-allrouters
172.17.0.2    nginx-web1 622eff54876f web1
172.17.0.4    c85c73ebf00b
```

- 连通性测试

```bash
root@c85c73ebf00b:/# ping nginx-web1 -c2
PING nginx-web1 (172.17.0.2) 56(84) bytes of data.
64 bytes from nginx-web1 (172.17.0.2): icmp_seq=1 ttl=64 time=0.112 ms
64 bytes from nginx-web1 (172.17.0.2): icmp_seq=2 ttl=64 time=0.055 ms

--- nginx-web1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 2ms
rtt min/avg/max/mdev = 0.055/0.083/0.112/0.029 ms
```



## 二、Docker网络

### 四类网络模式

| Docker网络模式 | 配置                      | 说明                                                         |
| -------------- | :------------------------ | ------------------------------------------------------------ |
| host模式       | –net=host                 | 容器和宿主机共享Network namespace。                          |
| container模式  | –net=container:NAME_or_ID | 容器和另外一个容器共享Network namespace。 kubernetes中的pod就是多个容器共享一个Network namespace。 |
| none模式       | –net=none                 | 容器有独立的Network namespace，但并没有对其进行任何网络设置，如分配veth pair 和网桥连接，配置IP等。 |
| bridge模式     | –net=bridge               | （默认为该模式）                                             |

​		Docker服务安装完成之后，默认在每个宿主机会生成一个名称为docker0的网卡，其ip地址都是172.17.0.1/16，并且会生成三种不同类型的网络

```bash
[root@docker-server1 ~]# ifconfig 
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:14:75:bf:4c  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.80.10  netmask 255.255.255.0  broadcast 192.168.80.255
        inet6 fe80::eaf3:dc40:2bf:6da2  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:f4:79:06  txqueuelen 1000  (Ethernet)
        RX packets 13079  bytes 18637594 (17.7 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1747  bytes 124995 (122.0 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1  (Local Loopback)
        RX packets 72  bytes 5776 (5.6 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 72  bytes 5776 (5.6 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@docker-server1 ~]# docker network list
NETWORK ID     NAME      DRIVER    SCOPE
787342a0d883   bridge    bridge    local
9a6d7244e807   host      host      local
beace8354cca   none      null      local
```

在启动容器的时候可以使用--network参数去指定网络类型，默认使用的是bridge网络类型



### none网络类型

​		在使用none模式后，docker容器**不会进行任何网络配置**，其没有网卡、没有ip也没有路由，因此默认无法与外界进行通信，需要手动添加网卡配置ip等，所以**极少使用**

```bash
[root@docker-server1 ~]# docker run -d -it --name web4 --network none nginx:1.8
```



### container网络类型

​		这个模式指定新创建的容器和**已经存在的一个容器共享**一个 Network Namespace，而不是和宿主机共享。新创建的容器不会创建自己的网卡，配置自己的 IP，而是和一个指定的容器共享 IP、端口范围等。同样，两个容器除了网络方面，其他的如文件系统、进程列表等还是隔离的。两个容器的进程可以通过 lo 网卡（环回接口）设备通信。

```bash
[root@docker-server1 ~]# docker run -d -it --name web5 --network container:web1 nginx
83db4f9af6f3d9d42bbd57691fcf82ef06cbf1a5874750effa314a4ec242aaaa
```



### host网络类型

​		如果启动容器的时候使用host模式，那么这个容器将不会获得一个独立的Network Namespace，而是和宿主机共用一个Network Namespace。容器将不会虚拟出自己的网卡，配置自己的IP等，而是**使用宿主机的IP和端口。**

​	使用host模式的容器可以直接使用宿主机的IP地址与外界通信**，容器内部的服务端口也可以使用宿主机的端口**，不需要进行NAT，host最大的优势就是**网络性能比较好**，但是docker host上已经使用的端口就不能再用了，**网络的隔离性不好。**

```bash
[root@docker-server1 ~]# docker run  -d -it --name web7 --network host nginx
e90cb3bfc1a3fbd187319ac3b995b116feb37422534b03662d624680e35eb2bb
[root@docker-server1 ~]# docker exec -it web7 bash
root@docker-server1:/# ifconfig 
docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        inet6 fe80::42:14ff:fe75:bf4c  prefixlen 64  scopeid 0x20<link>
        ether 02:42:14:75:bf:4c  txqueuelen 0  (Ethernet)
        RX packets 9647  bytes 394222 (384.9 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 10932  bytes 43303360 (41.2 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.175.10  netmask 255.255.255.0  broadcast 192.168.175.255
        inet6 fe80::eaf3:dc40:2bf:6da2  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:f4:79:06  txqueuelen 1000  (Ethernet)
        RX packets 60855  bytes 81177548 (77.4 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 17770  bytes 1472014 (1.4 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1  (Local Loopback)
        RX packets 72  bytes 5776 (5.6 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 72  bytes 5776 (5.6 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```



### bridge网络类型(默认)

​		当Docker进程启动时，会在主机上创建一个名为**docker0的虚拟网桥**，此主机上启动的Docker容器会连接到这个虚拟网桥上。虚拟网桥的工作方式和物理交换机类似，这样主机上的所有容器就通过交换机连在了一个二层网络中。

​		从docker0子网中分配一个IP给容器使用，并设置docker0的IP地址为容器的默认网关。在主机上创建一对虚拟网卡veth pair设备，Docker将veth pair设备的一端放在新创建的容器中，并命名为eth0（容器的网卡），另一端放在主机中，以vethxxx这样类似的名字命名，并将这个网络设备加入到docker0网桥中。可以通过brctl show命令查看。

bridge模式是docker的**默认网络模式**，不写--network参数，就是bridge模式。**使用docker run -p时，docker实际是在iptables做了DNAT规则，实现端口转发功能。可以使用iptables -t nat -vnL查看。**



## 三、创建自定义网络

可以基于docker命令创建自定义网络，自定义网络可以自定义ip地址范围和网关等信息。

- 创建一个网络

```bash
[root@docker-server1 ~]# docker network create -d bridge --subnet 10.10.0.0/16 --gateway 10.10.0.1 eagleslab-net
74ee6ecdfc0382ac0abb1b46a3c90e3c6a39f0b7388aa9ba99fddc6bac72e8ce
[root@docker-server1 ~]# docker network list
NETWORK ID     NAME            DRIVER    SCOPE
787342a0d883   bridge          bridge    local
74ee6ecdfc03   eagleslab-net   bridge    local
9a6d7244e807   host            host      local
beace8354cca   none            null      local
```

- 使用自定义网络创建容器

```bash
[root@docker-server1 ~]# docker run -d -it --name web8 --network eagleslab-net nginx
```

- 检查网络

```bash
[root@docker-server1 ~]# docker exec -it web8 bash
root@a7edddb4114e:/# ifconfig 
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.10.0.2  netmask 255.255.0.0  broadcast 10.10.255.255
        ether 02:42:0a:0a:00:02  txqueuelen 0  (Ethernet)
        RX packets 1064  bytes 8764484 (8.3 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 738  bytes 41361 (40.3 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
root@a7edddb4114e:/# ping www.baidu.com -c2
PING www.a.shifen.com (112.34.112.83) 56(84) bytes of data.
64 bytes from 112.34.112.83 (112.34.112.83): icmp_seq=1 ttl=127 time=37.8 ms
64 bytes from 112.34.112.83 (112.34.112.83): icmp_seq=2 ttl=127 time=36.9 ms

--- www.a.shifen.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 3ms
rtt min/avg/max/mdev = 36.865/37.320/37.776/0.494 ms
```

- iptables会生成nat的相应规则

```bash
[root@docker-server1 ~]# iptables -t nat -vnL
Chain PREROUTING (policy ACCEPT 12 packets, 759 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    2   136 DOCKER     all  --  *      *       0.0.0.0/0            0.0.0.0/0            ADDRTYPE match dst-type LOCAL

Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 1 packets, 76 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DOCKER     all  --  *      *       0.0.0.0/0           !127.0.0.0/8          ADDRTYPE match dst-type LOCAL

Chain POSTROUTING (policy ACCEPT 1 packets, 76 bytes)
 pkts bytes target     prot opt in     out     source               destination         
   12   759 MASQUERADE  all  --  *      !br-74ee6ecdfc03  10.10.0.0/16         0.0.0.0/0           
   32  1940 MASQUERADE  all  --  *      !docker0  172.17.0.0/16        0.0.0.0/0           

Chain DOCKER (2 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 RETURN     all  --  br-74ee6ecdfc03 *       0.0.0.0/0            0.0.0.0/0           
    1    84 RETURN     all  --  docker0 *       0.0.0.0/0            0.0.0.0/0
```



## 四、Docker 安装 code-server

### **1 改权限（可选）**

在容器里面也能使用 docker 命令，所以需要修改 docker.sock 的权限并将其映射到容器内。

```text
chmod a+rw /run/docker.sock # 或者 chmod a+rw /var/run/docker.sock
```

这里就看 docker.sock 文件在哪里了，都运行一遍也没关系。



### **2 运行**

```text
docker run -d \
--name=code-server \
-e DEFAULT_WORKSPACE=/config/workspace \
-e PASSWORD=123 \
-e SUDO_PASSWORD=123 \
--net host \
-v /home/docker/code-server/config:/config \
-v /root:/config/workspace \
-v /run/docker.sock:/var/run/docker.sock \
-v $(which docker):/usr/bin/docker \
-v $(which docker-compose):/usr/bin/docker-compose \
--restart always \
--privileged=true \
linuxserver/code-server:latest
```

DEFAULT_WORKSPACE 是指定 web 访问时默认的工作目录。 PASSWORD 进入 web 页面的密码。 SUDO_PASSWORD 是容器内使用 sudo 命令时需要的密码。（为了安全，默认账号并不是 root 运行）

`-v /root:/config/workspace` 则是把宿主机的 root 目录作为工作目录。



### **3 修改映射目录的权限（可选）**

```text
chmod -R 777 /root/
```

想要在容器内不出现权限问题，就改 777 权限，这个依据情况而定。

SUDO_PASSWORD 是使用 sudo 命令时候使用的命令。即 root 命令。



### **4 配置 code-server**

### **4.1 显示顶部菜单**

左上角 -> View -> Appearance -> Show Menu Bar



### **4.2 修改主题颜色**

设置 -> Color Theme -> Dark



### **4.3 设置不显示隐藏文件夹**

设置 -> 搜索 file.exclude -> Add Pattern `**/.*`



### **4.4 安装扩展**

- Chinese - 汉化
- Live Server - 静态页面服务器（方便访问）



# 七、资源限制介绍

### 容器资源限制

默认情况下，容器没有资源限制，可以使用主机内核调度程序允许的尽可能多的给定资源，docker提供了控制容器可以限制容器使用多少内存或者cpu的方法，设置docker run命令的运行时配置标志。

其中一些功能要求宿主机的内核支持Linux功能，要检查支持，可以使用docker info命令，如果内核中禁用了某项功能，可能会在输出结尾处看到警告。



### OOM异常

对于Linux主机，如果没有足够的内存来执行其他重要的系统任务，将会抛出OOM异常（内存溢出、内存泄漏、内存异常），随后系统会开始杀死进程以释放内存，凡是运行在宿主机的进程都有可能被kill，包括dockerd和其他的应用程序，如果重要的系统进程被kill，会导致和该进程相关的服务全部宕机。

产生OOM异常时，Dockerd尝试通过调整docker守护程序上的OOM优先级来减轻这些风险，以便它比系统上的其他进程更不可能被杀死，但是容器的OOM优先级未调整时单个容器被杀死的可能性更大（不推荐调整容器的优先级这种方式）。



### Linux进程OOM评分机制

Linux会每个进程算一个分数，最终他会将分数最高的进程kill掉

```bash
/proc/PID/oom_score_adj #范围为-1000到1000，值越高越容易被宿主机kill掉，如果将该值设置为-1000，则进程永远不会被宿主机kernel kill。
/proc/PID/oom_adj #范围为-17到+15，取值越高越容易被干掉，如果是-17,则表示不能被kill，该设置参数的存在是为了和旧版本的Linux内核兼容。
/proc/PID/oom_score #这个值是系统综合进程的内存消耗量、CPU时间(utime+ stime)、存活时间(uptime - start time)和oom_adj计算出的进程得分，消耗内存越多得分越高，越容易被宿主机 kernel强制杀死。
```



## 一、容器的内存限制

Docker可以强制执行**硬性内存限制**，即只允许容器使用给定的内存大小

Docker也可以执行**非硬性内存限制**，即容器可以使用尽可能多的内存，除非内核检测到主机上的内存不够用了

### 内存限制参数

```bash
-m or --memory：容器可以使用的最大内存量，如果设置此选项，则允许的最小值为4m
--memory-swap：容器可以使用的交换分区和物理内存大小总和，必须要在设置了物理内存限制的前提才能设置交换分区的限制，经常将内存交换到磁盘的应用程序会降低性能。如果该参数设置未-1，则容器可以使用主机上swap的最大空间
--memory-swappiness：设置容器使用交换分区的倾向性，值越高表示越倾向于使用swap分区，范围为0-100，0为能不用就不用，100为能用就用。
--kernel-memory：容器可以使用的最大内核内存量，最小为4m，由于内核内存于用户空间内存隔离，因此无法于用户空间内存直接交换，因此内核内存不足的容器可能会阻塞宿主机主机资源，这会对主机和其他容器或者其他服务进程产生影响，因此不要设置内核内存大小
--memory-reservation：允许指定小于--memory的软限制，当Docker检测到主机上的争用或内存不足时会激活该限制，若使用--memory-reservation，则必须将其设置为低于--memory才能使其优先。因为它是软限制，所以不能保证容器不超过限制。
--oom-kill-disable：默认情况下，发生OOM时kernel会杀死容器内进程，但是可以使用该参数可以禁止oom发生在指定的容器上，仅在已设置-m选项的容器上禁用oom，如果-m参数未配置，产生oom时主机为了释放内存还会杀死进程
```



### 案例

如果一个容器未作内存使用限制，则该容器可以利用到系统内存最大空间，默认创建的容器没有做内存资源限制

- 拉取容器压测工具镜像

```bash
[root@docker-server1 ~]# docker pull lorel/docker-stress-ng
[root@docker-server1 ~]# docker run -it --rm lorel/docker-stress-ng -help
```

- 使用压测工具开启两个工作进程，每个工作进程最大允许使用内存256M，且宿主机不限制当前容器的最大内存

```bash
[root@docker-server1 ~]# docker run -it --rm --name test1 lorel/docker-stress-ng --vm 2 --vm-bytes 256m
stress-ng: info: [1] defaulting to a 86400 second run per stressor
stress-ng: info: [1] dispatching hogs: 2 vm

[root@docker-server1 ~]# docker stats
CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT     MEM %     NET I/O     BLOCK I/O   PIDS
3ca32774fc20   test1     185.16%   514.3MiB / 1.781GiB   28.21%    648B / 0B   0B / 0B     5
```

- 宿主机限制最大内存使用

```bash
[root@docker-server1 ~]# docker run -it --rm -m 256m --name test2 lorel/docker-stress-ng --vm 2 --vm-bytes 256m
[root@docker-server1 ~]# docker stats
CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT   MEM %     NET I/O     BLOCK I/O         PIDS
bfff488e6185   test1     169.76%   255.8MiB / 256MiB   99.91%    648B / 0B   3.53GB / 10.6GB   5
```

- 可以通过修改cgroup文件值来扩大内存限制，缩小会报错

```bash
[root@docker-server1 ~]# cat /sys/fs/cgroup/memory/docker/bfff488e618580b227b5411c91b35517850e95af2ac2225b45180937c14e70c2/memory.limit_in_bytes
```



**内存软限制**

软限制不会真正限制到内存的使用

```bash
[root@docker-server1 ~]# docker run -it --rm -m 256m --memory-reservation 128m --name test1 lorel/docker-stress-ng --vm 2 --vm-bytes 256m
[root@docker-server1 ~]# docker stats
CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT   MEM %     NET I/O     BLOCK I/O         PIDS
0ffb4b8fdbde   test1     174.52%   255.9MiB / 256MiB   99.95%    648B / 0B   5.33GB / 18.1GB   5
```

**交换分区限制**

```bash
[root@docker-server1 ~]# docker run -it --rm -m 256m --memory-swap 512m --name test1 lorel/docker-stress-ng --vm 2 --vm-bytes 256m
```



## 二、容器的CPU限制

一个宿主机，有几十个核心的cpu，但是宿主机上可以同时运行成百上千个不同的进程用以处理不同的任务，多进程共用一个cpu的核心依赖计数就是为可压缩资源，即一个核心cpu可以通过调度而运行多个进程，但是在同一个单位时间内只能由一个进程在cpu上运行，那么这么多的进程怎么在cpu上执行和调度的呢？（进程优先级）

默认情况下，每个容器对主机cpu周期的访问权限是不受限制的，但是我们可以人为干扰

### 参数

```bash
--cpus： 指定容器可以使用多少可用cpu资源，例如，如果主机有两个cpu，并且设置了--cpu=1.5，那么该容器将保证最多可以访问1.5个的cpu（如果是4核cpu，那么还可以是4核心上的每核用一点，但是总计是1.5核心的cpu）
--cpu-period：设置cpu的调度周期，必须于--cpu-quota一起使用
--cpu-quota：在容器上添加cpu配额，计算方式为cpu-quota/cpu-period的结果值（现在通常使用--cpus）
--cpuset-cpus：用于指定容器运行的cpu编号，也就是所谓的绑核
--cpuset-mem：设置使用哪个cpu的内存，仅对非统一内存访问（NUMA）架构有效。
--cpu-shares：值越高的容器将会得到更多的时间片（宿主机多核cpu总数为100%，加入容器A为1024，容器B为2048，那么容器B将最大时容器A的可以CPU的两倍），默认的时间片时2014，最大为262144
```



### 案例

**未限制容器cpu**

- 启动1个进程，占用8核cpu，未限制容器会把cpu全部占完

```bash
[root@docker-server1 ~]# docker run -it --rm --name test1 lorel/docker-stress-ng --vm 1 --cpu 8 

[root@docker-server1 ~]# docker stats
CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT     MEM %     NET I/O     BLOCK I/O        PIDS
7eb5882b9379   test1     812.96%   1.387GiB / 1.781GiB   77.88%    648B / 0B   4.02GB / 412MB   25
```



**限制容器cpu**

```bash
[root@docker-server1 ~]# docker run -it --rm  --cpus 4 --name test1 lorel/docker-stress-ng --vm 1 --cpu 8 

[root@docker-server1 ~]# docker stats
CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT     MEM %     NET I/O     BLOCK I/O        PIDS
0a1c3805e5c9   test1     398.99%   1.414GiB / 1.781GiB   79.40%    648B / 0B   1.14GB / 127MB   25

[root@docker-server1 ~]# top
top - 21:19:53 up  2:33,  2 users,  load average: 12.50, 9.48, 4.62
Tasks: 175 total,  17 running, 158 sleeping,   0 stopped,   0 zombie
%Cpu0  : 52.1 us,  0.3 sy,  0.0 ni, 45.5 id,  1.7 wa,  0.0 hi,  0.3 si,  0.0 st
%Cpu1  : 49.8 us,  1.4 sy,  0.0 ni, 15.2 id, 32.5 wa,  0.0 hi,  1.0 si,  0.0 st
%Cpu2  : 48.6 us,  1.7 sy,  0.0 ni,  6.2 id, 41.4 wa,  0.0 hi,  2.1 si,  0.0 st
%Cpu3  : 49.1 us,  2.1 sy,  0.0 ni,  8.0 id, 39.4 wa,  0.0 hi,  1.4 si,  0.0 st
%Cpu4  : 51.6 us,  0.7 sy,  0.0 ni, 46.0 id,  1.4 wa,  0.0 hi,  0.3 si,  0.0 st
%Cpu5  : 48.4 us,  2.4 sy,  0.0 ni,  1.4 id, 46.3 wa,  0.0 hi,  1.4 si,  0.0 st
%Cpu6  : 50.2 us,  1.0 sy,  0.0 ni, 23.9 id, 24.2 wa,  0.0 hi,  0.7 si,  0.0 st
%Cpu7  : 45.3 us,  5.9 sy,  0.0 ni, 21.1 id, 26.6 wa,  0.0 hi,  1.0 si,  0.0 st
```



**将容器运行到指定的cpu上**

```bash
[root@docker-server1 ~]# docker run -it --rm  --cpus 2 --cpuset-cpus 1,3 --name test1 lorel/docker-stress-ng --vm 1 --cpu 4 

[root@docker-server1 ~]# docker stats
CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT     MEM %     NET I/O     BLOCK I/O         PIDS
ee11d834dde5   test1     186.68%   1.488GiB / 1.781GiB   83.60%    648B / 0B   44.8GB / 95.7MB   25

[root@docker-server1 ~]# top
top - 21:27:31 up  2:41,  2 users,  load average: 14.97, 9.00, 5.77
Tasks: 176 total,  19 running, 157 sleeping,   0 stopped,   0 zombie
%Cpu0  :  0.0 us,  0.3 sy,  0.0 ni, 99.0 id,  0.0 wa,  0.0 hi,  0.7 si,  0.0 st
%Cpu1  : 87.5 us, 10.9 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  1.7 si,  0.0 st
%Cpu2  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu3  : 32.9 us, 46.1 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi, 21.1 si,  0.0 st
%Cpu4  :  0.0 us, 17.3 sy,  0.0 ni, 82.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu5  :  0.0 us, 12.7 sy,  0.0 ni, 79.2 id,  0.0 wa,  0.0 hi,  8.2 si,  0.0 st
%Cpu6  :  0.0 us,  0.0 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.3 si,  0.0 st
%Cpu7  :  0.0 us,  0.3 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
```

- 基于cpu-shares对cpu进行切分

```bash
[root@docker-server1 ~]# docker run -it --rm  -d --cpu-shares 1000 --name test1 lorel/docker-stress-ng --vm 1 --cpu 4 
[root@docker-server1 ~]# docker run -it --rm  -d --cpu-shares 500 --name test2 lorel/docker-stress-ng --vm 1 --cpu 4 

[root@docker-server1 ~]# docker stats
```



# 八、docker单机编排

## 一、简介

当在宿主机启动较多的容器时候，如果都是手动操作会觉得比较麻烦而且容器出错，这个推荐使用docker单机编排工具docker-compose，docker-compose是docker容器的一种单机编排服务，docker-compose是一个管理多个容器的工具。比如可以解决容器之间的依赖关系，就像启动一个nginx前端服务的时候会调用后端的tomcat，那就得先启动tomcat，但是启动tomcat容器还需要依赖数据库，那就还得先启动数据库，docker-compose就可以解决这样的嵌套依赖关系，其完全可以替代docker run对容器进行创建、启动、和停止。

docker-compose项目是docker官方的开源项目，负责实现对docker容器集群的快速编排，docker-compose将所管理的容器分为三层，分别是工程（project），服务（service），以及容器（container）。



### Docker Compose 项目结构

Docker Compose 项目将所管理的容器分为三层，分别是：

- **工程（Project）**：工程是 Docker Compose 管理的最高层级，通常对应一个包含多个服务的应用场景。一个工程可以包含多个服务，这些服务通过 `docker-compose.yml` 文件进行定义。
- **服务（Service）**：服务是工程中的一个逻辑单元，通常对应一个容器模板。服务定义了容器的镜像、环境变量、端口映射等配置。一个服务可以启动多个容器实例。
- **容器（Container）**：容器是服务的具体运行实例。Docker Compose 会根据服务的定义创建并管理容器。

通过这种分层结构，Docker Compose 能够高效地管理和编排多个容器，简化复杂的容器依赖关系，提高开发和部署效率。



## 二、基础环境准备

- yum安装docker-compese

```shell
[root@docker-server1 ~]# yum install epel-release.noarch -y
[root@docker-server1 ~]# yum install docker-compose.noarch -y
[root@docker-server1 ~]# docker-compose version 
```

- 或者使用二进制安装，进入官网下载对应版本即可

官方下载网址：https://github.com/docker/compose/releases



- 相关参数

```bash
docker-compose --help
Define and run multi-container applications with Docker.
Usage:
  docker-compose [-f <arg>...] [options] [COMMAND] [ARGS...]
  docker-compose -h|--help

#选项说明：
-f，–file FILE #指定Compose 模板文件，默认为docker-compose.yml。
-p，–project-name NAME #指定项目名称，默认将使用当前所在目录名称作为项目名。
--verbose   #显示更多输出信息
--log-level LEVEL    #定义日志级别 (DEBUG, INFO, WARNING, ERROR, CRITICAL) 
--no-ansi #不显示ANSI 控制字符
-v, --version #显示版本

#以下为命令选项，需要在docker-compose.yml|yaml 文件所在在目录里执行 
build  #构建镜像 
bundle #从当前docker compose 文件生成一个以<当前目录>为名称的json格式的Docker Bundle 备份文件
config  -q #查看当前配置，没有错误不输出任何信息 
create #创建服务，较少使用 
down #停止和删除所有容器、网络、镜像和卷 
#events #从容器接收实时事件，可以指定json 日志格式，较少使用 
exec #进入指定容器进行操作 
help #显示帮助细信息 
images #显示镜像信息，较少使用
kill #强制终止运行中的容器 
logs #查看容器的日志 
pause #暂停服务 
port #查看端口 
ps #列出容器，较少使用
pull #重新拉取镜像，镜像发生变化后，需要重新拉取镜像，较少使用 
push #上传镜像 
restart #重启服务，较少使用 
rm #删除已经停止的服务 
run #一次性运行容器 
scale  #设置指定服务运行的容器个数 
start #启动服务 ，较少使用
stop #停止服务，较少使用 
top #显示容器运行状态 
unpause #取消暂定 
up #创建并启动容器 ，较少使用
```



## 三、启动单个容器

- 编写docker-compose文件

```shell
[root@localhost ~]# mkdir -pv docker-compose/nginx
[root@localhost ~]# cd docker-compose/nginx
[root@localhost nginx]# pwd
/root/docker-compose/nginx
[root@localhost nginx]# vim docker-compose.yml
# docker-compose.yml

services:
  nginx:
    image: nginx
    container_name: nginx_web1
    restart: always
    ports:
      - "80:80"
    volumes:
      - /data/web:/usr/share/nginx/html
```



- 启动容器

```shell
[root@localhost nginx]# docker compose up -d
[+] Running 2/2
 ✔ Network nginx_default  Created                                             0.0s
 ✔ Container nginx_web1   Started                                             0.2s
[root@localhost nginx]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                                 NAMES
09d091a0c1a1   nginx     "/docker-entrypoint.…"   29 seconds ago   Up 28 seconds   0.0.0.0:80->80/tcp, [::]:80->80/tcp   nginx_web1

# 不指定网络的话，会默认创建一个类型为bridge的网络
[root@localhost nginx]# docker network ls
NETWORK ID     NAME            DRIVER    SCOPE
36f9c2f8e090   bridge          bridge    local
93ffe4510dd0   host            host      local
d0456365c64d   nginx_default   bridge    local
```



## 四、启动多个容器

### 编辑docker-compose文件

```shell
[root@docker-server1 docker]# cat docker-compose.yml 
services:
  service-nginx:
    image: nginx
    container_name: nginx_web1
    ports:
      - "80:80"

  service-tomcat:
    image: tomcat
    container_name: tomcat_web1
    ports:
      - "8080:8080"
[root@docker-server1 docker]# docker-compose up -d
nginx_web1    /docker-entrypoint.sh ngin ...   Up      0.0.0.0:80->80/tcp,:::80->80/tcp               
tomcat_web1   catalina.sh run                  Up      0.0.0.0:8080->8080/tcp,:::8080->8080/tcp
```



### 定义数据卷挂载

- 创建数据卷目录和文件

```shell
[root@docker-server1 docker]# mkdir -p /data/nginx
[root@docker-server1 docker]# echo 'docker nginx' > /data/nginx/index.html
```



- 编辑配置文件

```yaml
[root@docker-server1 docker]# cat docker-compose.yml 
service-nginx:
  image: nginx
  container_name: nginx_web1
  volumes:
    - /data/nginx/:/usr/share/nginx/html
  ports:
    - "80:80"

service-tomcat:
  image: tomcat
  container_name: tomcat_web1
  ports:
    - "8080:8080"
```



- 访问测试

```shell
[root@docker-server1 docker]# curl localhost
docker nginx
```



### 相关操作演示

- 重启单个容器

```bash
[root@docker-server1 docker]# docker-compose restart service-nginx 
```

- 重启所有容器

```bash
[root@docker-server1 docker]# docker-compose restart 
```

- 停止单个容器

```bash
[root@docker-server1 docker]# docker-compose stop service-nginx 
```

- 启动单个容器

```bash
[root@docker-server1 docker]# docker-compose start service-nginx 
```

- 停止所有容器

```bash
[root@docker-server1 docker]# docker-compose stop 
```

- 启动所有容器

```bash
[root@docker-server1 docker]# docker-compose start 
```



## 五、部署Dockge

Dockge是一个docker-compose管理工具，可以很方便的管理docker-compose的yml文件和任务。

官方网站为：https://dockge.kuma.pet/

在官网下载compose.yml文件后直接docker-compose启动即可

部署 Dockge 后通过 http://IP:5001 端口访问 Dockge。首次访问需要设置账户密码

compose.yaml 文件都会部署在 /opt/stacks 目录



## 六、docker-compose实战部署lnmp

- 任务：使用docker-compose实现编排nginx+phpfpm+mysql容器，部署一个博客系统
- 首先编写一个如下的配置文件

```yaml
services:
  web:
    image: nginx:latest
    restart: always
    ports:
      - 80:80
    volumes:
      - ./nginx:/etc/nginx/conf.d
      - ./html:/usr/share/nginx/html
    depends_on:
      - php
      - mysql
    networks:
      - lnmp-network
  php:
    image: php:7.4fpm
    restart: always
    volumes:
      - ./html:/usr/share/nginx/html
    networks:
      - lnmp-network
    depends_on:
      - mysql
  mysql:
    image: mysql:5.7
    restart: always
    volumes:
      - dbdata:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: "123456"
      MYSQL_DATABASE: login
    networks:
      - lnmp-network
networks:
  lnmp-network:
    driver: bridge
volumes:
  dbdata: null
```

运行docker-compose.yml

```bash
[root@localhost lnmp]# docker compose up -d
[root@localhost lnmp]# docker ps
CONTAINER ID   IMAGE      COMMAND       CREATED         STATUS         PORTS           NAMES
825b8630de47   nginx:latest   "/docker-entrypoint.…"   4 minutes ago   Up 4 minutes   0.0.0.0:80->80/tcp, [::]:80->80/tcp   lnmp-web-1
d1d6339ba0d7   lnmp-php       "docker-php-entrypoi…"   4 minutes ago   Up 4 minutes   9000/tcp                              lnmp-php-1
109672c0b1f0   mysql:5.7      "docker-entrypoint.s…"   4 minutes ago   Up 
```

在`/opt/stack/lnmp/nginx/default.conf` 中写入nginx的配置文件

```shell
server {
        listen 80;
        root /usr/share/nginx/html;
        
        location / {
                index index.php index.html;
         }
        location ~ \.php$ {
                fastcgi_pass php:9000;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                include fastcgi_params;
         }
}
```

准备好php的探针，然后访问这个页面

```bash
<?php
    $dbhost = "mysql";
    $dbuser = "root";
    $dbpass = "123456";
    $db = "login";
    $conn = mysqli_connect($dbhost, $dbuser, $dbpass, $db) or exit("数据库连接失败！");
    echo "数据库连接成功";
?>
```

访问测试后发现，连接mysql报错，提示找不到mysqli_connect()，发现是官方构建的php没有mysqli模块

所以需要定制带有mysqli模块的php，编写如下dockerfile，来对php镜像进行定制

```shell
FROM php:7.4-fpm
ENV TZ=Asia/Shanghai
RUN mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini" \
  && sed -i 's/deb.debian.org/mirrors.ustc.edu.cn/g' /etc/apt/sources.list \
  && sed -i -e 's|security.debian.org/\? |security.debian.org/debian-security |g' \
  -e 's|security.debian.org|mirrors.ustc.edu.cn|g' \
  -e 's|deb.debian.org/debian-security|mirrors.ustc.edu.cn/debian-security|g' \
  /etc/apt/sources.list \ 
  && apt-get update && apt-get install -y \
  libfreetype6-dev \
  libjpeg62-turbo-dev \
  libpng-dev \
  && docker-php-ext-configure gd -with-freetype -with-jpeg \
  && docker-php-ext-install -j$(nproc) gd mysqli && docker-php-ext-enable mysqli
```



修改yaml配置文件，让docker-compose帮我们构建php的镜像

```yaml
services:
  web:
    image: nginx:l# docker-compose build lnmp

services:
  web:
    image: nginx:latest
    restart: always
    ports:
      - 80:80
    volumes:
      - /data/lnmp/nginx/conf.d:/etc/nginx/conf.d
      - /data/lnmp/nginx/html:/usr/share/nginx/html
      - /data/lnmp/nginx/log:/var/log/nginx
    depends_on:
      - php
      - mysql
    networks:
      - lnmp-network
  php:
    # image: dockerfile:php:7.4-fpm
    # 这里来让compose自动构建，指定dockerfile的文件位置即可
    build:
      context: .
      dockerfile: dockerfile
    restart: always
    volumes:
      - /data/lnmp/nginx/html:/usr/share/nginx/html
    networks:
      - lnmp-network
    depends_on:
      - mysql
  mysql:
    image: mysql:5.7
    restart: always
    volumes:
      - /data/lnmp/dbdata:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: "123456"
      MYSQL_DATABASE: login
    networks:
      - lnmp-network
networks:
  lnmp-network:
    driver: bridge
volumes:
  dbdata: nullatest
    restart: always
    ports:
      - 80:80
    volumes:
      - ./nginx:/etc/nginx/conf.d
      - ./html:/usr/share/nginx/html
    depends_on:
      - php
      - mysql
    networks:
      - lnmp-network
  php:
    build:
      context: .
      dockerfile: dockerfile/php
    restart: always
    volumes:
      - ./html:/usr/share/nginx/html
    networks:
      - lnmp-network
    depends_on:
      - mysql
  mysql:
    image: mysql:5.7
    restart: always
    volumes:
      - dbdata:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: "123456"
      MYSQL_DATABASE: login
    networks:
      - lnmp-network
networks:
  lnmp-network:
    driver: bridge
volumes:
  dbdata: null
```

替换镜像之后，查看探针成功加上mysql的模块



测试数据库是否链接成功

```php
[root@localhost lnmp]# docker compose up -d
[root@localhost lnmp]# docker ps
CONTAINER ID   IMAGE       COMMAND        CREATED         STATUS         PORTS             NAMES
825b8630de47   nginx:latest   "/docker-entrypoint.…"   4 minutes ago   Up 4 minutes   0.0.0.0:80->80/tcp, [::]:80->80/tcp   lnmp-web-1
d1d6339ba0d7   lnmp-php       "docker-php-entrypoi…"   4 minutes ago   Up 4 minutes   9000/tcp                              lnmp-php-1
109672c0b1f0   mysql:5.7      "docker-entrypoint.s…"   4 minutes ago   Up 4 minutes   3306/tcp, 33060/tcp                   lnmp-mysql-1
```



### z-blog博客部署

官网：https://www.zblogcn.com/

一、下载zblog到指定目录，并解压

```bash
[root@localhost lnmp]# cd /data/lnmp/nginx/html/
[root@localhost html]# wget https://update.zblogcn.com/zip/Z-BlogPHP_1_7_4_3430_Shelter.zip
[root@localhost html]# ls
Z-BlogPHP_1_7_4_3430_Shelter.zip  index.html  info.php  mysql.php
[root@localhost html]# unzip Z-BlogPHP_1_7_4_3430_Shelter.zip
[root@localhost html]# chmod -R 777 *
```

浏览器访问

二、环境检查

三、需要进入到mysql容器中，手动创建一个blog的库然后如下填写

四、安装完成

五、优化网站

# 九、docker仓库管理

## 一、Docker Registry

Docker Register作为Docker的核心组件之一负责镜像内容的存储与分发，客户端的docker pull以及push命令都将直接与register进行交互，最初版本的registry由python是心啊，由于涉及初期在安全性、性能以及API的设计上有着诸多的缺陷，该版本在0.9之后停止了开发，由新的项目distribution（新的docker register被称为Distribution）来重新设计并开发了下一代的registry，新的项目由go语言开发，所有的api底层存储方式，系统架构都进行了全面的重新设计已解决上一代registry钟村镇的问题。

官方文档地址：https://docs.docker.com/registry

官方github地址: https://github.com/docker/distribution



### 搭建镜像

- 下载docker registry镜像

```bash
[root@docker-server ~]# docker pull registry
```

- 创建授权使用目录

```bash
[root@docker-server ~]# mkdir /docker/auth -p
```

- 创建用户

```shell
[root@docker-server docker]# yum install httpd-tools.x86_64 -y
[root@docker-server docker]# htpasswd -Bbn jack 123456 > auth/htpasswd
[root@docker-server docker]# cat auth/htpasswd 
jack:$2y$05$a2wtUYyoC8p/eXzoseT9Q.dhMDgQgwkUiKVfs1z6zijk6M4UIiUsq
[root@docker-server docker]# docker run -d -p 5000:5000 -v /docker/auth/:/auth -e \
 "REGISTRY_AUTH=htpasswd" \
 -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
 -e "REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd" registry
7be5539c6cbef114b33a083ef976836a44b97ea54a7c41beb1b568878794b22a
[root@docker-server docker]# docker ps -l
CONTAINER ID   IMAGE      COMMAND                  CREATED         STATUS         PORTS                                       NAMES
7be5539c6cbe   registry   "/entrypoint.sh /etc…"   7 seconds ago   Up 6 seconds   0.0.0.0:5000->5000/tcp, :::5000->5000/tcp   nostalgic_stonebraker
[root@docker-server docker]# 
[root@docker-server docker]# curl http://jack:123456@127.0.0.1:5000/v2/_catalog
{"repositories":[]}
[root@docker-server docker]# docker login  127.0.0.1:5000
Username: jack
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store
Login Succeeded
```

将本机的镜像push到仓库中

```shell
[root@localhost ~]# docker pull centos:7
[root@localhost ~]# docker tag centos:latest 127.0.0.1:5000/test/centos:9
[root@localhost ~]# docker push 127.0.0.1:5000/test/centos:9
The push refers to repository [127.0.0.1:5000/test/centos]
74ddd0ec08fa: Pushed 
9: digest: sha256:a1801b843b1bfaf77c501e7a6d3f709401a1e0c83863037fa3aab063a7fdb9dc size: 529
[root@localhost ~]# curl http://jack:123456@127.0.0.1:5000/v2/_catalog
{"repositories":["test/centos"]}
```

添加本地信任，信任之后就可以从自己的仓库下载东西了，由于我们测试的是127.0.0.1的地址，不用添加

```shell
cat /etc/docker/daemon.json
{
    "registry-mirrors": ["远程的仓库地址"],
    "insecure-registries": ["远程的仓库地址"],
    "exec-opts": ["native.cgroupdriver=systemd"]
}
```

先删除本地的nginx镜像，然后尝试从远程下载一次

```shell
[root@localhost ~]# docker pull 127.0.0.1:5000/test/centos:9
9: Pulling from test/centos
a1d0c7532777: Pull complete 
Digest: sha256:a1801b843b1bfaf77c501e7a6d3f709401a1e0c83863037fa3aab063a7fdb9dc
Status: Downloaded newer image for 127.0.0.1:5000/test/centos:9
127.0.0.1:5000/test/centos:9
```



## 二、Docker 仓库之分布式harbor

参考网址：https://goharbor.io/docs/2.3.0/install-config/

Harbor是一个用于存储和分发Docker镜像的企业级Registry服务器，由vmware开源，其通过添加一些企业必须的功能特性，例如安全、标识和管理等，扩展了开源的Docker Distribution。Harbor也提供了高级的安全特性，诸如用户管理，访问控制和活动审计等。

### 安装

- 下载镜像

```bash
[root@docker-server1 ~]# wget https://github.com/goharbor/harbor/releases/download/v2.3.1/harbor-offline-installer-v2.3.1.tgz
```

- 解压缩

```bash
[root@docker-server1 ~]# tar xzvf harbor-offline-installer-v2.3.1.tgz 
[root@docker-server1 ~]# ln -sv /root/harbor /usr/local/
"/usr/local/harbor" -> "/root/harbor"
```

- 安装harbor

```bash
[root@docker-server1 harbor]# cp harbor.yml.tmpl harbor.yml
[root@docker-server1 harbor]# grep -Ev '#|^$' harbor.yml.tmpl > harbor.yml
[root@docker-server1 harbor]# cat harbor.yml
hostname: 192.168.80.10
http:
  port: 80
harbor_admin_password: Harbor12345
database:
  password: root123
  max_idle_conns: 100
  max_open_conns: 900
data_volume: /data
trivy:
  ignore_unfixed: false
  skip_update: false
  insecure: false
jobservice:
  max_job_workers: 10
notification:
  webhook_job_max_retry: 10
chart:
  absolute_url: disabled
log:
  level: info
  local:
    rotate_count: 50
    rotate_size: 200M
    location: /var/log/harbor
_version: 2.3.0
proxy:
  http_proxy:
  https_proxy:
  no_proxy:
  components:
    - core
    - jobservice
    - trivy
[root@docker-server1 harbor]# curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
[root@localhost harbor]# chmod +x /usr/local/bin/docker-compose
[root@localhost harbor]# docker-compose --version
docker-compose version 1.29.2, build 5becea4c
[root@docker-server1 harbor]# ./prepare 
[root@docker-server1 harbor]# ./install.sh 

[root@docker-server1 harbor]# ss -tnl
State      Recv-Q Send-Q Local Address:Port               Peer Address:Port              
LISTEN     0      128             *:80                          *:*                  
LISTEN     0      128             *:22                          *:*                  
LISTEN     0      100     127.0.0.1:25                          *:*                  
LISTEN     0      128     127.0.0.1:1514                        *:*                  
LISTEN     0      128            :::80                         :::*                  
LISTEN     0      128            :::22                         :::*                  
LISTEN     0      100           ::1:25                         :::*  

---
# 之后的启动关闭可以通过docker-compose管理，自动生成docker-compose.yml文件
[root@docker-server1 harbor]# ls
common     docker-compose.yml    harbor.yml       install.sh  prepare
common.sh  harbor.v2.3.1.tar.gz  harbor.yml.tmpl  LICENSE
```



### web访问

默认用户名和密码在harbor.yml中设置为：harbor_admin_password: Harbor12345

 

### 推送镜像

创建项目

登录后推送镜像

```shell
[root@localhost ~]# cat /etc/docker/daemon.json
{
    "registry-mirrors": ["http://192.168.173.136/"],
    "insecure-registries": ["192.168.173.136"]
}
[root@localhost ~]# systemctl restart docker
# 私有仓库需要在此登录
[root@localhost ~]# docker login 192.168.173.136
Username: admin
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store
Login Succeeded
```



- 打标签推送镜像

```shell
[root@localhost harbor]# docker tag centos:latest 192.168.173.136/eagleslab/centos:9
[root@localhost harbor]# docker push 192.168.173.136/eagleslab/centos:9
The push refers to repository [192.168.173.136/eagleslab/centos]
74ddd0ec08fa: Pushed 
9: digest: sha256:a1801b843b1bfaf77c501e7a6d3f709401a1e0c83863037fa3aab063a7fdb9dc size: 529
```

