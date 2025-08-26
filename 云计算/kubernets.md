# 一、kubernetes介绍

## 云计算的三种模式

### 单机到云原生的演变

```
第一阶段：业务模式主要为场地出租，为客户提供服务器托管服务。在这个阶段的软件架构以单体架构软件为主

第二阶段：业务模式为提供场地、存储、以及互联网接入服务，为客户提供服务器托管和虚拟主机出租服务。在第二阶段的软件架构主要是分布式架构

第三阶段：业务模式为提供计算、存储、网络服务，提供可动态调度的算力服务。此阶段的软件架构发展到了微服务架构和容器化架构
```



### IAAS

Infrastructure as a Service

**IaaS**（Infrastructure as a Service，**基础架构即服务**）是基础层。在这一层，通过虚拟化、动态化将IT基础资源（计算、网络、存储）聚合形成资源池。资源池即计算能力的集合，终端用户（企业）可以通过网络获得自己需要的计算资源，运行自己的业务系统。这种方式使用户不必自己建设这些基础设施，而是通过付费即可使用这些资源。



### PAAS

platform as a service

**PaaS**（Platform as a Service，**平台即服务**）层。这一层除了提供基础计算能力，还具备了业务的开发运行环境，提供包括应用代码、SDK、操作系统以及API在内的IT组件，供个人开发者和企业将相应功能模块嵌入软件或硬件，以提高开发效率。对于企业或终端用户而言，这一层的服务可以为业务创新提供快速、低成本的环境。



### SAAS

Software as a Service

**SaaS**（Software as a Service，**软件即服务**）。实际上，SaaS在云计算概念出现之前就已经存在，并随着云计算技术的发展得到了更好的发展。SaaS的软件是“拿来即用”的，不需要用户安装，软件升级与维护也无须终端用户参与。同时，它还是按需使用的软件，与传统软件购买后就无法退货相较具有无可比拟的优势。



## 容器编排工具

### Borg

Borg 是 Google 早期开发的集群管理系统，用于管理大规模的容器化⼯作负载。它是 Google 内部的⼀个关键基础设施，自 2003 年开始使用。



### Omega

Omega 是 Borg 的⼀个后继系统，于 2013 年开始开发，旨在解决 Borg 在大规模、动态、多租户环境下多租户、声 明 式 配 置。



### kubernetes

kubernetes是一个生产级别的开源平台，可编排在计算机集群内和跨计算机集群的应用容器的部署和执行

kubernetes这个名字源于希腊于，意为舵手或飞行员。k8s这个缩写是因为k和s之间有八个字符的关系。google在2014年开源了kubernetes项目。kubernetes建立在google在大规模运行生产工作负载方面拥有十几年的经验的基础上，结合了社区中最好的想法和实践。



## kubernetes架构

kubernetes由一个master和一组用于运行容器化应用的工作机器组成，这些工作机器被称为node，每个集群至少需要一个工作节点来运行node，node托管着组成应用负载的pod，控制平面管理集群中的工作节点和pod。生产环境中，控制平面通常跨多台计算机运行，而一个集群通常运行多个节点，以提供容错和高可用



## kubernetes优势

- 服务发现和负载均衡
- 存储编排（添加任何本地或云服务器）
- 自动部署和回滚
- 自动分配	CPU/内存资源
- 弹性伸缩
- 自我修复（需要时启动新容器）
- Secret（安全相关信息）和配置管理
- 大型规模的支持
  - 每个节点的Pod数量不超过	110
  - 节点数不超过	5000
  - Pod总数不超过	150000
  - 容器总数不超过	300000
- 开源



## kubernetes组件

![img](https://i-blog.csdnimg.cn/blog_migrate/c263cfabe9ae5193aa993944f6cb7401.png)

一个kubernetes集群由一组被称作节点的机器组成。这些节点上运行Kubernetes所管理的容器化应用。集群具有至少一个工作节点。

工作节点托管作为应用负载的组件的Pod。控制平面管理集群中的工作节点和pod。为集群提供故障转移和高可用性，这些控制平面一般跨多主机运行，集群跨多个节点运行。

**Master：**集群中的一台服务器用作Master，负责管理整个集群。Master是集群的网关和中枢，负责诸如为用户和客户端暴露API、跟踪其他服务器的健康状态、以最优方式调度工作负载，以及编排其他组件之间的通信任务，它是用户或客户端与集群之间的核心联络点，并负载Kubernetes系统的大多数集中式管控逻辑。单个master节点即可完成其所有的功能，但出于冗余及负载均衡等目的，生产环境中通常需要协同部署多个此类主机。

**Node：** Node是Kubernetes集群的工作节点，负责接收来自master的工作指令相应地创建或销毁Pod对象，以及调整网络规则以合理地路由和转发流量等。理论上讲，Node可以是任何形式的计算设备，不过Master会统一将其抽象为Node对象进行管理。

Kubernetes将所有Node资源集结于一处形成一台更加强大的服务器，在用户将应用部署于其上时，Master会使用调度算法将其自动指派至某个特定的Node运行。在Node加入集群或从集群中移除时，Master也会按需重新编排影响到的Pod，于是用户无须关心其应用究竟运行何处。



### kubernetes核心组成

控制平面组件可以在集群中的任何节点上运行。然后，为了简单起见，设置脚本通常会在同一个计算机上启动所有控制平面组件，并且不会在此计算机上运行用户容器。



#### Master组件

##### kube-apiserver

apiserver是kubernetes master的组件，该组件负责公开kubernetes api，负责处理接受请求的工作，apiserver是kubernetes master的前端。可通过部署多个实例来进行扩缩，可运行kube-apiserver的多个实例，并在这些势力之间平衡流量。

##### etcd

一致且高可用的键值存储，用作kubernetes所有集群数据的后台数据库

##### kube-controller-manager

负责运行控制器进程。

##### kube-scheduler

负责监视新创建的、未指定运行节点的的pod，并选择节点让pod在上面运行

##### cloud-controller-manager

嵌入了特定于云平台的控制逻辑，允许将集群连接到云提供商的api上，并将与该云平台交互的组件同与集群交互的组件分离开来。cloud-controller-manager仅运行特定于云平台的控制器



#### Node组件

##### kubelet

kubelet是运行于工作节点上的守护进程，保证容器都运行在pod中

##### kube-proxy

是集群中每个节点上所运行的网络代理，维护节点上的一些网络规则，这些网络规则允许从集群内部或外部的网络与pod进行网络通信。

##### 容器运行时环境

使kubernetes有效的运行容器，管理kubernetes环境中容器的执行和生命周期



## kubernetes插件与附件

### 组件

apiserver，scheduler，controller-manager，kubelet，kubeproxy，...



### 插件

DNS服务器，dashboard仪表盘，容器资源监控，集群层面日志，网络插件，....



### 附件

premetheus，dashboard，federation



### 补充知识Restful

对应的中文是rest式的；Restful web service是⼀种常见的rest的应用,是遵守了rest风格的web服务；rest式的web服务是⼀种ROA(The Resource-Oriented Architecture)(面向资源的架构)。

在Restful之前的操作：

- http://127.0.0.1/user/query/1 GET  根据用户id查询用户数据
- http://127.0.0.1/user/save POST 新增用户
- http://127.0.0.1/user/update  POST 修改用户信
- http://127.0.0.1/user/delete/1 GET/POST 删除用户信

RESTful用法：

- http://127.0.0.1/user/1 GET 根据用户id查询用户数据
- http://127.0.0.1/user POST 新增用户
- http://127.0.0.1/user PUT 修改用户信
- http://127.0.0.1/user DELETE 删除用户信



## Node

节点是一个虚拟机或者物理机，它在kubernetes集群中充当工作机器的角色。每个节点都有kubelet用于管理节点而且是节点与控制面通信的代理。节点使用控制面所公布的kubernetes api与控制面通信，终端用户也可以使用kubernetes api与集群交互



## k8s常用命令

```shell
查看资源信息类指令
kubectl get pods # 获取pod列表
kubectl get pods -n ingress-nginx # 获取指定名称空间pod列表
kubectl get pods -o wide # 获取pod详细信息
kubectl get pods --show-labels # 获取pod并查看pod标签
kubectl get pod -w # 监视pod资源变动信息
kubectl get pods -A|grep -v Running # 获取集群所有非Running状态pod
kubectl get pods,services -o wide # 格式化输出pod及services信息
kubectl exec podname env # 查看运行的pod的环境变量
kubectl describe pods podname --namespace=xxxnamespace # 查看指定名称空间下指定pod的详细信息
kubectl logs podname 
kubectl logs podname -f -c container_name -n kube-system  # 查看pod日志(-f 持续监控，-c如果pod中只有一个容器不用加)
kubectl get namespaces # 查看集群内所有名称空间
kubectl get cs # 查看集群健康状态
kubectl get events # 查看事件
kubectl get nodes # 查看集群全部节点
kubectl get deployment --all-namespaces/-A # 查看集群内所有deployment
kubectl get deployments deployment_name --watch # 监控deployment的更新过程
kubectl get rc,services # 查看集群默认名称空间的副本控制器和services
kubectl get pods podname -o yaml # 查看pod的yaml信息
kubectl version --short=true # 查看客户端及服务端程序版本信息
kubectl api-versions # 查看api版本信息
kubectl cluster-info # 查看集群信息
kubectl top nodes #查看节点资源使用情况(集群内需安装metric-server组件)
kubectl top pod --all-namespaces # 查看pod资源使用率
kubectl config view # 显示合并的kubeconfig配置

#  查看 deploy 的资源分配情
kubectl get deploy -o=custom-columns=name:.metadata.name,ns:.metadata.namespace,replicas:.spec.replicas,request-cpu:.spec.template.spec.containers[0].resources.requests.cpu,limit-cpu:.spec.template.spec.containers[0].resources.limits.cpu,request-memory:.spec.template.spec.containers[0].resources.requests.memory,limit-memory:.spec.template.spec.containers[0].resources.limits.memory,affinity:.spec.template.spec.affinity

# 查看 pod 的资源分配情况
kubectl get pod -o=custom-columns=name:.metadata.name,ns:.metadata.namespace,request-cpu:.spec.containers[0].resources.requests.cpu,limit-cpu:.spec.containers[0].resources.limits.cpu,request-memory:.spec.containers[0].resources.requests.memory,limit-memory:.spec.containers[0].resources.limits.memory,nodeName:.spec.nodeName,node:.spec.nodeName,affinity:.spec.affinity

操作编辑类指令
kubectl drain <nodename> --force --ignore-daemonsets # 排空节点忽略daemonsets应用并设置节点为不可调度
kubectl uncordon <nodename> # 将节点设置为可调度
kubectl scale deployment --replicas=<期望pod数> # 扩缩容pod数
kubectl rollout history deployments deployment-demo # 查看历史版本
# 创建一个名字为nginx-demo 副本数为3 标签为app=nginx_demo 镜像为nginx:1.17 容器端口为80的容器实例
kubectl run nginx-demo --replicas=3 --labels="app=nginx_demo" --image=nginx:1.17 --port=80
# 为容器组nginx-demo创建一个services端口类型为nodeportr 暴漏的端口为88可以让外部可以访问
kubectl expose deployment nginx-demo --port=88 --type=NodePort --target-port=80 --name=nginx-svc
# 设置容器组nginx-demo中所有容器的request(请求)和limit(限制)值
kubectl set resources deployment nginx-demo - --limits=cpu=200m,memory=512Mi
# 将deployment中的nginx-demo容器镜像修改为nginx-1.10
kubectl set image deployment/nginx-demo busybox=busybox nginx=nginx:1.10
# 编辑deployment中nginx-demo的资源信息
kubectl edit deployment nginx-demo
# 给pod名为nginx-demo的pod打标签
kubectl label pods nginx-demo app=nginx
# 覆盖现有标签的value
kubectl label --overwrite pods nginx-demo app=nginx2

复制pod内文件到宿主机
kubectl cp -n namespace xxxxx:logs/testfile ./testfile.txt (namespace: 为pod所在名称空间 xxxxx：是pod的名称)
#需要进入pod所在node节点服务器中
#查找到pod所运行的docker container id
docker ps -a|grep test-app-name
#然后cp 容器内文件到宿主机
docker cp container id:dir/filename ./tmp

将宿主机文件复制到pod中
kubectl cp -n <本地宿主机的文件路径> namespace xxxxx:logs/testfile  (namespace: 为pod所在名称空间 xxxxx：是pod的名称)

k8s添加本地hosts解析
  hostAliases:
  - ip: 192.168.1.135
    hostnames:
    - "test.app.com"

删除资源类命令
kubectl delete -f demo-deployment.yaml # 根据yaml文件删除对应资源，只删除资源不删除yaml文件
kubectl delete node  <nodename> # 删除node节点
kubectl delete <resources_name> # 删除指定资源
# 批量删除驱逐状态pod
kubectl get pods -n namespace | grep  Evicted| awk '{print $1}' | xargs -n 1 kubectl delete pods -n namespace 

创建资源类命令
#创建ingress tls证书
kubectl  create secret tls \<tls-name\> --cert=\</tls/xxx.crt\> --key=\</tls/xxxxx.key\> --namespace=xxx

# 获取当前的资源，pod
$ kubectl get pod 
	-A,--all-namespaces 查看当前所有名称空间的资源
	-n  指定名称空间，默认值 default，kube-system 空间存放是当前组件资源
	--show-labels  查看当前的标签
	-l  显示标签，筛选资源，key、key=value
	-o wide  详细信息包括 IP、	分配的节点
	-w  监视，打印当前的资源对象的变化部分
	
# 进入 Pod 内部的容器执行命令
$ kubectl exec -it podName -c cName -- command
	-c  可以省略，默认进入唯一的容器内部
	
# 查看资源的描述
$ kubectl explain pod.spec

# 查看 pod 内部容器的 日志
$ kubectl logs podName -c cName

# 查看资源对象的详细描述
$ kubectl describe pod podName

# 删除资源对象
$ kubectl delete kindName objName
	--all 删除当前所有的资源对象
```

# 二、Pod介绍

## pod概念

pod是一个或多个应用容器（例如docker）的组合，并且包含共享的存储卷、IP地址和有关如何运行它们的信息。

pod是kubernetes抽象出来的，表示一组一个或者多个应用容器，以及这些容器的一些共享资源。



### Pause特性

- Pod内部第一个启动的容器，相当于Linux守护进程
- 初始化网络栈
- 挂载需要的存储卷
- 回收僵⼫进程
- 其他容器与Pause容器共享名字空间(Network、PID、IPC)



## 实战：基于Docker模拟pod

- 编写Nginx配置文件

```bash
cat <<EOF>> nginx.conf
error_log stderr;
events {
    worker_connections 1024;
}
http {
    access_log /dev/stdout combined; 
    server {
        listen 80 default_server; 
        server_name example.com www.example.com; 
        
        location / {
            proxy_pass http://127.0.0.1:2368;
        }
    }
}
EOF
```



- 启动容器

```bash
docker run --name pause -p 8080:80 -d k8s.gcr.io/pause:3.1
docker run --name nginx -v `pwd`/nginx.conf:/etc/nginx/nginx.conf --net=container:pause --ipc=container:pause --pid=container:pause -d nginx
docker run -d --name ghost -e NODE_ENV=development --net=container:pause --ipc=container:pause --pid=container:pause ghost
```



## kubernetes网络

### 基本概述

Kubernetes的网络模型假定了所有Pod都在一个可以直接连通的扁平的网络空间中，这在GCE（Google Compute Engine）里面是现成的网络模型，Kubernetes假定这个网络已经存在。而在私有云里搭建Kubernetes集群，就不能假定这个网络已经存在了。我们需要自己实现这个网络假设，将不同节点上的Docker容器之间的互相访问先打通，然后运行Kubernetes。



### 网络模型原则

- 在不使用网络地址转换(NAT)的情况下，集群中的Pod能够与任意其他Pod进行通信
- 在不使用网络地址转换(NAT)的情况下，在集群节点上运行的程序能与同一节点上的任何Pod进行通信
- 每个Pod都有自己的IP地址（IP-per-Pod），并且任意其他Pod都可以通过相同的这个地址访问它



### CNI

借助CNI标准，Kubernetes可以实现容器网络问题的解决。通过插件化的方式来集成各种网络插件，实现集群内部网络相互通信，只要实现CNI标准中定义的核心接口操作（ADD，将容器添加到网络；DEL，从网络中删除一个容器；CHECK，检查容器的网络是否符合预期等）。CNI插件通常聚焦在容器到容器的网络通信。



CNI的接口并不是指HTTP，gRPC这种接口，CNI接口是指对可执行程序的调用（exec）可执行程序，Kubernetes节点默认的CNI插件路径为/opt/cni/bin

```bash
[root@master01 ~]# ls /opt/cni/bin/
bandwidth  calico    dhcp   firewall  host-device  install  loopback  portmap  sbr     tap     vlan
bridge     calico-ipam  dummy  flannel   host-local   ipvlan   macvlan   ptp      static  tuning  vrf
```

CNI通过JSON格式的配置文件来描述网络配置，当需要设置容器网络时，由容器运行时负责执行CNI插件，并通过CNI插件的标准输入（stdin）来传递配置文件信息，通过标准输出（stdout）接收插件的执行结果。从网络插件功能可以分为五类：

- **Main插件**

  - 创建具体网络设备

    - bridge：网桥设备，连接container和host；
    - ipvlan：为容器增加ipvlan网卡；
    - loopback：IO设备；
    - macvlan：为容器创建一个MAC地址；
    - ptp：创建一对Veth Pair；
    - vlan：分配一个vlan设备；
    - host-device：将已存在的设备移入容器内

    

- **IPAM插件**：

  - 负责分配IP地址

    - dhcp：容器向DHCP服务器发起请求，给Pod发放或回收IP地址；
    - host-local：使用预先配置的IP地址段来进行分配；
    - static：为容器分配一个静态IPv4/IPv6地址，主要用于debug

    

- **META插件**：

  - 其他功能的插件

    - tuning：通过sysctl调整网络设备参数；
    - portmap：通过iptables配置端口映射；
    - bandwidth：使用TokenBucketFilter来限流；
    - sbr：为网卡设置source based routing；
    - firewall：通过iptables给容器网络的进出流量进行限制

    

- **Windows插件**：专门用于Windows平台的CNI插件（win-bridge与win-overlay网络插件）

- **第三方网络插件**：第三方开源的网络插件众多，每个组件都有各自的优点及适应的场景，难以形成统一的标准组件，常用有Flannel、Calico、Cilium、OVN网络插件



#### 调度流程

pod被调度到容器集群的某个节点上，节点的kubelet调用CRI插件来创建pod，CRI插件创建pod sandbos id与pod网络命名空间，CRI插件通过网络命名空间和pod sandbox id 来调用cni插件，cni插件配置pod网络（flannel cni插件->bridge cni -> 主机ipam cni插件->返回pod ip地址），然后把结果返回给plugin组件，创建pause容器并将其添加到pod网络命名空间，kubernetes调用CRI插件拉取应用容器镜像，容器进行时containerd拉取应用容器镜像，kubernetes调用CRI插件来启动应用容器，CRI插件调用容器进行时containerd来启动和配置在pod cgroup和namespace中的应用容器。



### 网络插件

| 提供商            | 网络模型                 | 路由分发 | 网络策略 | 网格 | 外部数据存储   | 加密 | Ingress/Egress策略 |
| ----------------- | ------------------------ | -------- | -------- | ---- | -------------- | ---- | ------------------ |
| Canal             | 封装(VXLAN)              | 否       | 是       | 否   | K8s API        | 是   | 是                 |
| Flannel（佛兰多） | 封装(VXLAN)              | 否       | 否       | 否   | K8s API        | 是   | 中                 |
| Calico            | 封装(VXLAN，IPIP),未封装 | 是       | 是       | 是   | Etcd 和K8s API | 是   | 是                 |
| Weave             | 封装                     | 是       | 是       | 是   | 否             | 是   | 是                 |
| Cilium            | 封装(VXLAN)              | 是       | 是       | 是   | Etcd和K8sAPI   | 是   | 是                 |



#### 网络模型

- underlay network（非封装网络）
  - 现实的物理基础层网络设备
  - underlay就是数据中心场景的基础物理设施，保证任何两个点路由可达，其中包含了传统的网络技术
- overlay network（封装网络）
  - 一个基于物理网络之上构建的逻辑网络
  - overlay是在网络技术领域指的是一种网络架构上叠加的虚拟化技术模式
  - Overlay网络技术多种多样，一般采用TRILL、VxLan、GRE、NVGRE等隧道技术



#### calico

或许是目前最主流的网络解决方案-calico

Calico是一个纯三层的虚拟网络，它没有复用docker的docker0网桥，而是自己实现的，calico网络不对数据包进行额外封装，不需要NAT和端口映射

##### calico架构

- Felix
  
  管理网络接口、编写路由、编写ACL、报告状态
  
- bird（BGPClient）
  
  BGPClient将通过BGP协议⼴播告诉剩余calico节点，从而实现网络互通
  
- confd
  
  通过监听etcd以了解BGP配置和全局默认值的更改。Confd根据ETCD中数据的更新，动态生成BIRD配置文件。当配置文件更改时，confd触发BIRD重新加载新文件
  
- etcd

  分布式键值存储，主要负责网络元数据一致性，确保Calico网络状态的准确性，可以与kubernetes共用；



##### VXLAN

- VXLAN，即VirtualExtensibleLAN（虚拟可扩展局域网），是Linux本身支持的一网种网络虚拟化技术。VXLAN可以完全在内核态实现封装和解封装工作，从而通过“隧道”机制，构建出覆盖网络（OverlayNetwork）
- 基于三层的”二层“通信，三层即vxlan包封装在udp数据包中，要求udp在k8s节点间三层可达；二层即vxlan封包的源mac地址和目的mac地址是自己的vxlan设备mac和对端vxlan设备mac实现通讯。
- 数据包封包：封包，在vxlan设备上将pod发来的数据包源、目的mac替换为本机vxlan网卡和对端节点vxlan 网卡的mac。外层udp目的ip地址根据路由和对端vxlan的mac查fdb表获取


优势：只要k8s节点间三层互通，可以跨网段，对主机网关路由没有特殊要求。各个node节点通过vxlan设备实现基于三层的”二层”互通


缺点：需要进行vxlan的数据包封包和解包会存在一定的性能损耗

配置方法

```bash
# 在calico的配置文件中。如下配置
# Auto-detect the BGP IP address.
- name: IP
value: "autodetect"
# Enable IPIP
- name: CALICO_IPV4POOL_IPIP
value: "Never" #键值“off”代表临时关闭，允许通过其他方式临时启用。“never”强制禁止使用，并且不允许通过任何方式启用
# Enable or Disable VXLAN on the default IP pool.
- name: CALICO_IPV4POOL_VXLAN
value: "Always"
# Enable or Disable VXLAN on the default IPv6 IP pool.
- name: CALICO_IPV6POOL_VXLAN
value: "Always"
# 分割线========================
# calico_backend: "bird"
calico_backend: "vxlan"
# - -felix-live
# - -bird-live
```

VXLAN不需要BGP来建立节点间的邻接关系



##### IPIP

- Linux原生内核支持
- IPIP隧道的工作原理是将源主机的IP数据包封装在一个新的IP数据包中，新的IP数据包的目的地址是隧道的另一端。在隧道的另一端，接收方将解封装原始IP数据包，并将其传递到目标主机。IPIP隧道可以在不同的网络之间建立连接，例如在IPv4网络和IPv6网络之间建立连接。

- 数据包封包：封包，在tunl0设备上将pod发来的数据包的mac层去掉，留下ip层封包。外层数据包目的ip地址根据路由得到。

优点：只要k8s节点间三层互通，可以跨网段，对主机网关路由没有特殊要求。

缺点：需要进行IPIP的数据包封包和解包会存在一定的性能损耗

配置方法

```bash
# 在calico的配置文件中。如下配置
# Auto-detect the BGP IP address.
- name: IP
value: "autodetect
# Enable IPIP
- name: CALICO_IPV4POOL_IPIP
value: "Always"
# Enable or Disable VXLAN on the default IP pool.
- name: CALICO_IPV4POOL_VXLAN
value: "Never"
# Enable or Disable VXLAN on the default IPv6 IP pool.
- name: CALICO_IPV6POOL_VXLAN
value: "Never"
```

IPIP模式需要BGP来建立节点间的邻接关系，VXLAN不需要



##### BGP

边界网关协议（Border Gateway Protocol,BGP）是互联网上一个核心的去中心化自治路由协议。它通过维护IP路由表或‘前缀’表来实现自治系统（AS）之间的可达性，属于矢量路由协议。BGP不使用传统的内部网关协议（IGP）的指标，而使用基于路径、网络策略或规则集来决定路由。因此，它更适合被称为矢量性协议，而不是路由协议。BGP，通俗的讲就是讲接入到机房的多条线路（如电信、联通、移动等）融合为一体，实现多线单IP，BGP机房的优点：服务器只需要设置一个IP地址，最佳访问路由是由网络上的骨⼲路由器根据路由跳数与其它技术指标来确定的，不会占用服务器的任何系统。

- 数据包封包：不需要进行数据包封包

优点：不用封包解包，通过BGP协议可实现pod网络在主机间的三层可达

缺点：跨网段时，配置较为复杂网络要求较高，主机网关路由也需要充当BGPSpeaker。

配置方法

```bash
# 在calico的配置文件中。如下配置
# Auto-detect the BGP IP address.
- name: IP
value: "autodetect"
# Enable IPIP
- name: CALICO_IPV4POOL_IPIP
value: "Off"
# Enable or Disable VXLAN on the default IP pool.
- name: CALICO_IPV4POOL_VXLA
value: "Never"
# Enable or Disable VXLAN on the default IPv6 IP pool.
- name: CALICO_IPV6POOL_VXLAN
value: "Never"
```



# 三、安装

## 安装方式

1.kubeadm：通过容器化方式运行

优势：安装简单，组件损坏只需要kill再重启即可

缺点：封装了底层安装细节，不易理解

2.二进制安装：组件变成系统进程的方式运行

优势：能够更灵活的安装出集群；可以分开安装master组件，形成更大的规模

缺点：每一个配置文件都需要理解，安装较难



## 环境初始化

```bash
# 网卡配置
# cat /etc/NetworkManager/system-connections/ens160.nmconnection
[ipv4]
method=manual
address1=192.168.173.100/24,192.168.173.2
dns=114.114.114.114;114.114.115.115
# cat /etc/NetworkManager/system-connections/ens192.nmconnection
[connection]
autoconnect=false

# 调用 nmcli 重启设备和连接配置
nmcli d d ens192
nmcli d r ens160 
nmcli c r ens160
```



```bash
# Rocky 系统软件源更换
sed -e 's|^mirrorlist=|#mirrorlist=|g' \
    -e 's|^#baseurl=http://dl.rockylinux.org/$contentdir|baseurl=https://mirrors.aliyun.com/rockylinux|g' \
    -i.bak \
    /etc/yum.repos.d/[Rr]ocky*.repo
    
dnf makecache
# 防火墙修改 firewalld 为 iptables
systemctl stop firewalld
systemctl disable firewalld

yum -y install iptables-services
systemctl start iptables
iptables -F
systemctl enable iptables
service iptables save
# 禁用 Selinux
setenforce 0
sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
grubby --update-kernel ALL --args selinux=0
# 查看是否禁用，grubby --info DEFAULT
# 回滚内核层禁用操作，grubby --update-kernel ALL --remove-args selinux
# 设置时区
timedatectl set-timezone Asia/Shanghai
```



```bash
# 关闭 swap 分区
swapoff -a
sed -i 's:/dev/mapper/rl-swap:#/dev/mapper/rl-swap:g' /etc/fstab

# 修改主机名
hostnamectl  set-hostname k8s-node01
# 安装 ipvs
yum install -y ipvsadm
# 开启路由转发
echo 'net.ipv4.ip_forward=1' >> /etc/sysctl.conf
sysctl -p
# 加载 bridge
yum install -y epel-release
yum install -y bridge-utils

modprobe br_netfilter
echo 'br_netfilter' >> /etc/modules-load.d/bridge.conf
echo 'net.bridge.bridge-nf-call-iptables=1' >> /etc/sysctl.conf
echo 'net.bridge.bridge-nf-call-ip6tables=1' >> /etc/sysctl.conf
sysctl -p
```



```bash
# 添加 docker-ce yum 源
# 中科大(ustc)
sudo dnf config-manager --add-repo https://mirrors.ustc.edu.cn/docker-ce/linux/centos/docker-ce.repo
cd /etc/yum.repos.d
# 切换中科大源
sed -i 's#download.docker.com#mirrors.ustc.edu.cn/docker-ce#g' docker-ce.repo

# 安装 docker-ce
yum -y install docker-ce

# 配置 daemon.
cat > /etc/docker/daemon.json <<EOF
{
  "default-ipc-mode":"shareable",
  "data-root": "/data/docker",
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m",
    "max-file": "100"
  },
  "registry-mirrors": [
    "https://docker.1ms.run",
    "https://docker.1panel.live",
    "https://docker.xuanyuan.me",
    "https://dockerproxy.net",
    "https://docker.fast360.cn",
    "https://cloudlayer.icu",
    "https://docker-registry.nmqu.com",
    "https://hub.amingg.com",
    "https://docker.amingg.com",
    "https://docker.hlmirror.com"
  ]
}
EOF
mkdir -p /etc/systemd/system/docker.service.d

# 重启docker服务
systemctl daemon-reload && systemctl restart docker && systemctl enable docker
```

cri-docker启动顺序：Linux->docker->cri-docker->kubelet->api  server/control  manager/scheduler

```bash
# 安装 cri-docker
wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.9/cri-dockerd-0.3.9.amd64.tgz
tar -xf cri-dockerd-0.3.9.amd64.tgz
cp cri-dockerd/cri-dockerd /usr/bin/
chmod +x /usr/bin/cri-dockerd

# 配置 cri-docker 服务
cat <<"EOF" > /usr/lib/systemd/system/cri-docker.service
[Unit]
Description=CRI Interface for Docker Application Container Engine
Documentation=https://docs.mirantis.com
After=network-online.target firewalld.service docker.service
Wants=network-online.target
Requires=cri-docker.socket
[Service]
Type=notify
ExecStart=/usr/bin/cri-dockerd --network-plugin=cni --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.8
ExecReload=/bin/kill -s HUP $MAINPID
TimeoutSec=0
RestartSec=2
Restart=always
StartLimitBurst=3
StartLimitInterval=60s
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
TasksMax=infinity
Delegate=yes
KillMode=process
[Install]
WantedBy=multi-user.target
EOF

# 添加 cri-docker 套接字
cat <<"EOF" > /usr/lib/systemd/system/cri-docker.socket
[Unit]
Description=CRI Docker Socket for the API
PartOf=cri-docker.service
[Socket]
ListenStream=%t/cri-dockerd.sock
SocketMode=0660
SocketUser=root
SocketGroup=docker
[Install]
WantedBy=sockets.target
EOF

# 启动 cri-docker 对应服务
systemctl daemon-reload
systemctl enable cri-docker
systemctl start cri-docker
systemctl is-active cri-docker
```



```bash
# 添加 kubeadm yum 源
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF

# 安装 kubeadm 1.29 版本
yum install -y kubelet-1.29.0 kubectl-1.29.0 kubeadm-1.29.0
systemctl enable kubelet.service
```

**只在master初始化主节点**

```bash
# 初始化主节点
kubeadm init\
 --apiserver-advertise-address=192.168.173.100\
 --image-repository registry.aliyuncs.com/google_containers\
 --kubernetes-version 1.29.2\
 --service-cidr=10.10.0.0/12\
 --pod-network-cidr=10.244.0.0/16\
 --ignore-preflight-errors=all\
 --cri-socket unix:///var/run/cri-dockerd.sock
 
 #创建集群
 mkdir -p $HOME/.kube
 sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
 sudo chown $(id -u):$(id -g) $HOME/.kube/config

```



```bash
# node 加入
kubeadm join 192.168.173.100:6443 --token jghzcm.mz6js92jom1flry0 \
        --discovery-token-ca-cert-hash sha256:63253f3d82fa07022af61bb2ae799177d2b0b6fe8398d5273098f4288ce67793  --cri-socket unix:///var/run/cri-dockerd.sock
        
# work token 过期后，重新申请
kubeadm token create --print-join-command
```



## 部署网络插件

```bash
https://docs.tigera.io/calico/latest/getting-started/kubernetes/self-managed-onprem/onpremises#install-calico-with-kubernetes-api-datastore-more-than-50-nodes

curl https://raw.githubusercontent.com/projectcalico/calico/v3.26.3/manifests/calico-typha.yaml -o calico.yaml
	CALICO_IPV4POOL_CIDR	指定为 pod 地址
	
# 修改为 BGP 模式
# Enable IPIP
- name: CALICO_IPV4POOL_IPIP
  value: "Always"  #改成Off

kubectl apply -f calico-typha.yaml
kubectl get pod -A
```



### 固定网卡(可选)

```bash
# 目标 IP 或域名可达
        - name: calico-node
          image: registry.geoway.com/calico/node:v3.19.1
          env:
            # Auto-detect the BGP IP address.
            - name: IP
              value: "autodetect"
            - name: IP_AUTODETECTION_METHOD
              value: "can-reach=www.google.com"
kubectl set env daemonset/calico-node -n kube-system IP_AUTODETECTION_METHOD=can-reach=www.google.com
```



```bash
# 匹配目标网卡
- name: calico-node
  image: registry.geoway.com/calico/node:v3.19.1
  env:
    # Auto-detect the BGP IP address.
    - name: IP
      value: "autodetect"
    - name: IP_AUTODETECTION_METHOD
      value: "interface=eth.*"
```



```bash
# 排除匹配网卡
- name: calico-node
  image: registry.geoway.com/calico/node:v3.19.1
  env:
    # Auto-detect the BGP IP address.
    - name: IP
      value: "autodetect"
    - name: IP_AUTODETECTION_METHOD
      value: "skip-interface=eth.*"
```



```bash
# CIDR
- name: calico-node
  image: registry.geoway.com/calico/node:v3.19.1
  env:
    # Auto-detect the BGP IP address.
    - name: IP
      value: "autodetect"
    - name: IP_AUTODETECTION_METHOD
      value: "cidr=192.168.200.0/24,172.15.0.0/24"
```



## 修改kube-proxy 模式为 ipvs

```bash
# kubectl edit configmap kube-proxy -n kube-system
mode: ipvs

kubectl delete pod -n kube-system -l k8s-app=kube-proxy
```



# 四、资源清单

## 资源

kubernetes系统的 api server基于http/https接收并响应客户端的操作请求，它提供了一种基于资源的RESTful风格的编程结构，将集群的各种组件都抽象成为标准的REST资源，如Node、Namespace和Pod等，并支持通过标准的HTTP方法以JSON为数据序列化方案进行资源管理操作。

kubernetes系统将一切事物都抽象为API资源。资源实例化之后，叫做对象。



## 资源类别

- 名称空间级别
  - 工作负载型资源：Pod、ReplicaSet、Deployment…
  - 服务发现及负载均衡型资源:Service、Ingress…
  - 配置与存储型资源：Volume、CSI…
  - 特殊类型的存储卷：ConfigMap、Secre…
- 集群级资源(与集群相关的资源)
  - Namespace、Node、ClusterRole、ClusterRoleBinding
- 元数据型资源(为集群内部的其他资源配置其行为或特性)
  - HPA、PodTemplate、LimitRange



## 资源清单编写

**可以通过`kubectl explain xxx`查询需要编写的资源结构**

在kubernetes中基本所有资源的一级属性都是一样的，主要包含5部分：

apiVersion 版本，由kubernetes内部定义，版本号必须可以用 kubectl api-versions 查询到
kind 类型，由kubernetes内部定义，版本号必须可以用 kubectl api-resources 查询到
metadata 元数据，主要是资源标识和说明，常用的有name、namespace、labels等
spec 描述，这是配置中最重要的一部分，里面是对各种资源配置的详细描述
status 状态信息，里面的内容不需要定义，由kubernetes自动生成
在上面的属性中，spec是接下来研究的重点，继续看下它的常见子属性:

containers <[]Object> 容器列表，用于定义容器的详细信息
nodeName 根据nodeName的值将pod调度到指定的Node节点上
nodeSelector <map[]> 根据NodeSelector中定义的信息选择将该Pod调度到包含这些label的Node 上
hostNetwork 是否使用主机网络模式，默认为false，如果设置为true，表示使用宿主机网络
volumes <[]Object> 存储卷，用于定义Pod上面挂在的存储信息
restartPolicy 重启策略，表示Pod在遇到故障的时候的处理策略


```yaml
apiVersion: v1  # 接口组/版本
kind: Pod       # 资源类别
metadata:       # 元数据
  name: pod-demo
  namespace: default   # 编写的时候修改，默认为default
  labels:              
    app: myapp
spec:           # 期望：pod最后达到的状态
  containers:  
    - name: myapp-1
      image: nginx
      resources:
        limits:
          memory: "128Mi"
          cpu: "500m"
    - name: myapp-2
      image: centos
      resources:
        limits:
          memory: "128Mi"
          cpu: "500m"
      command:
        - "/bin/sh"
        - "-c"
        - "sleep 300"
status:             # 状态：当前状态
  conditions:
    - lastProbeTime: "2024-09-04T07:50:54Z"
      lastTransitionTime: "2024-09-04T07:50:54Z"
      status: "True"
      type: Initialized
```

```yaml
[root@localhost ~]# mkdir pods
[root@localhost ~]# cd pods/
[root@localhost pods]# vim test.yaml
[root@localhost pods]# kubectl create -f test.yaml
pod/pod-demo created
[root@localhost pods]# kubectl get pod
NAME       READY   STATUS              RESTARTS   AGE
pod-demo   0/2     ContainerCreating   0          15s

# 查看pod的状态
[root@localhost pods]# kubectl describe pods pod-demo
Name:                      pod-demo
Namespace:                 default
Priority:                  0
Service Account:           default
Node:                      k8s-node02/192.168.88.30
Start Time:                Sun, 30 Mar 2025 22:53:01 +0800
Labels:                    app=myapp
Annotations:               cni.projectcalico.org/containerID: ab12e5f15578131fa38286c99c6984153c54e166c15ed98b9a23f34518b9e77e
                           cni.projectcalico.org/podIP: 10.244.58.194/32
                           cni.projectcalico.org/podIPs: 10.244.58.194/32
Status:                    Terminating (lasts 0s)
Termination Grace Period:  30s
IP:                        10.244.58.194
IPs:
  IP:  10.244.58.194
Containers:
  myapp-1:
    Container ID:   docker://8d380b77c3ca7eaa64db3a017e778f28f05851a6032f6b81bfb422ebbdb2ae03
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:124b44bfc9ccd1f3cedf4b592d4d1e8bddb78b51ec2ed5056c52d3692baebc19
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sun, 30 Mar 2025 22:53:04 +0800
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     500m
      memory:  128Mi
    Requests:
      cpu:        500m
      memory:     128Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-whlhn (ro)
  myapp-2:
    Container ID:  docker://0f50eb125694b8921e6cb464154fd77f13c8de64a48a1e09974e5833642fd3c9
    Image:         centos:7
    Image ID:      docker-pullable://centos@sha256:be65f488b7764ad3638f236b7b515b3678369a5124c47b8d32916d6487418ea4
    Port:          <none>
    Host Port:     <none>
    Command:
      /bin/sh
      -c
      sleep 300
    State:          Running
      Started:      Sun, 30 Mar 2025 22:53:04 +0800
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     500m
      memory:  128Mi
    Requests:
      cpu:        500m
      memory:     128Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-whlhn (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  kube-api-access-whlhn:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Guaranteed
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  39s   default-scheduler  Successfully assigned default/pod-demo to k8s-node02
  Normal  Pulling    39s   kubelet            Pulling image "nginx"
  Normal  Pulled     36s   kubelet            Successfully pulled image "nginx" in 2.479s (2.479s including waiting)
  Normal  Created    36s   kubelet            Created container myapp-1
  Normal  Started    36s   kubelet            Started container myapp-1
  Normal  Pulled     36s   kubelet            Container image "centos:7" already present on machine
  Normal  Created    36s   kubelet            Created container myapp-2
  Normal  Started    36s   kubelet            Started container myapp-2
  Normal  Killing    29s   kubelet            Stopping container myapp-1
  Normal  Killing    29s   kubelet            Stopping container myapp-2
```



## 查询对象属性

```yaml
$ kubectl explain pod.spec.containers
KIND:       Pod
VERSION:    v1

FIELD: containers <[]Container>

DESCRIPTION:
    List of containers belonging to the pod. Containers cannot currently be
    added or removed. There must be at least one container in a Pod. Cannot be
    updated.
    A single application container that you want to run within a pod.
...
```





## 快速体验

弹性伸缩，故障自愈，自动负载

- 创建nginx容器

```bash
[root@k8s-master1 ~]# kubectl create deployment nginx --image=nginx
deployment.apps/nginx created
```

- 暴露对外端口

```bash
[root@k8s-master1 ~]#  kubectl expose deployment nginx --port=80 --type=NodePort
service/nginx exposed
```

- 查看nginx是否运行成功

```bash
[root@k8s-master1 ~]#  kubectl get pod,svc
NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-6799fc88d8-qthk8   1/1     Running   0          71s

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/kubernetes   ClusterIP   172.16.0.1      <none>        443/TCP        34m
service/nginx        NodePort    172.16.170.23   <none>        80:31027/TCP   30s
```

- 扩容

```bash
[root@k8s-master1 ~]# kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-6799fc88d8-qthk8   1/1     Running   0          8m5s
[root@k8s-master1 ~]# kubectl scale deployment nginx --replicas=3
deployment.apps/nginx scaled
[root@k8s-master1 ~]# kubectl get pods
[root@k8s-master1 ~]# kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-6799fc88d8-46gmb   1/1     Running   0          64s
nginx-6799fc88d8-kn2dq   1/1     Running   0          64s
nginx-6799fc88d8-qthk8   1/1     Running   0          9m11s

# 缩容
[root@localhost pods]# kubectl scale deployment nginx --replicas=1
deployment.apps/nginx scaled
[root@localhost pods]# kubectl get pod
NAME                     READY   STATUS    RESTARTS   AGE
nginx-7854ff8877-xnxzl   1/1     Running   0          4m17s

# 故障自愈
[root@localhost pods]# kubectl delete pod/nginx-7854ff8877-hr257

# 观察发现，自动拉起新的pod，维护pod副本的数量
```

- 干跑（只测试不运行）

```shell
[root@localhost pods]# kubectl create deployment myapp --image=xxxx/xxxx -dry-run -o yaml # 以资源清单的方式输出
```



# 五、Pod生命周期

<img src="E:\英格\图片\Pod生命周期.png" alt="Pod生命周期" style="zoom:80%;" />

每个 Init 容器成功退出后才会启动下一个 Init 容器。 如果某容器因为容器运行时的原因无法启动，或以错误状态退出，kubelet 会根据 Pod 的 `restartPolicy` 策略进行重试。 在所有的 Init 容器没有成功之前，Pod 将不会变成 `Ready` 状态。

Init 容器可以包含一些安装过程中应用容器中不存在的实用工具或个性化代码。Init 容器可以安全地运行实用程序或自定义代码，而在其他方式下运行这些实用程序或自定义代码可能会降低应用容器镜像的安全性。

如果 Pod重启，所有 Init 容器必须重新执行。

init容器与普通的容器非常像，除了如下两点：

- init容器总是运行到成功完成为止
- 每个init容器都必须在下一个init容器启动之前成功完成

如果Pod的Init容器失败，Kubernetes会不断地重启该Pod，直到Init容器成功为止（原子性）。然而，如果Pod对应的restartPolicy为Never，它不会重新启动

mainC是并发创建的。



## 检测initC的阻塞性

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: initc-1
  labels:
    name: initc
spec:
  containers:
  - name: myapp-container
    image: centos:7
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
    command: ['sh', '-c', 'echo The app is running && sleep 10']
  initContainers:
  - name: init-myservice
    image: aaronxudocker/tools:busybox
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb
    image: aaronxudocker/tools:busybox
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```

```bash
# 查看日志，看到不停的在尝试
$ kubectl logs initc-1 -c init-myservice

# 创建svc资源，会通过CoreDNS自动将myservice解析成功，详解看后面的service部分
$ kubectl create svc clusterip myservice --tcp=80:80
```



如果initc执行失败了，那么就会重新执行所有的initc

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: initc-2
  labels:
    name: initc
spec:
  containers:
  - name: myapp-container
    image: centos:7
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
    command: ['sh', '-c', 'echo The app is running && sleep 10']
  initContainers:
  - name: init-myservice
    image: aaronxudocker/tools:busybox
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: randexit
    image: aaronxudocker/tools:randexitv1
    args: ['--exitcode=1']
```

```bash
$ kubectl get pod
NAME      READY   STATUS       RESTARTS      AGE
initc-1   0/1     Init:1/2     0             16m
initc-2   0/1     Init:Error   5 (97s ago)   3m42s
$ kubectl logs initc-2 -c randexit
休眠 4 秒，返回码为 1！
```

如果我们让initc的返回码直接为0，那么就可以看到pod正常启动

```bash
$ kubectl get pod
NAME      READY   STATUS     RESTARTS     AGE
initc-1   0/1     Init:1/2   0            19m
initc-2   1/1     Running    1 (7s ago)   72s
```

- InitC与应用容器具备不同的镜像，可以把一些危险的工具放置在initC中，进行使用
- initC多个之间是线性启动的，所以可以做一些延迟性的操作
- initC无法定义readinessProbe，其它以外同应用容器定义无异



## Pod探针

探针是由kubelet对容器执行的定期诊断。要执行诊断，kubelet调用由容器实现的Handler。有三种类型的处理程序：

- ExecAction：在容器内执行指定命令。如果命令退出时返回码为0则认为诊断成功
- TCPSocketAction：对指定端口上的容器的IP地址进行TCP检查。如果端口打开，则诊断被认为是成功的
- HTTPGetAction：对指定的端口和路径上的容器的IP地址执行HTTPGet请求。如果响应的状态码⼤于等于200且小于400，则诊断被认为是成功的

每次探测都将获得以下三种结果之一：

- 成功：容器通过了诊断。
- 失败：容器未通过诊断。
- 未知：诊断失败，因此不会采取任何行动



### 探针的分类

- startupProbe：开始探针，开始检测吗？
- livenessProbe：存活探针，还活着吗？
- readinessProbe：就绪探针，准备提供服务了吗？



### readinessProbe就绪探针

介绍：就绪探针决定容器何时准备好接受流量，就绪探针在容器的整个生命期内持续运行。如果就绪探针返回的状态为失败，Kubernetes 会将该 Pod 从所有对应服务的端点中移除。

选项说明

 - initialDelaySeconds：容器启动后要等待多少秒后就探针开始工作，单位“秒”，默认是0秒，最小值是0
- periodSeconds：执行探测的时间间隔（单位是秒），默认为10s，单位“秒”，最小值是1
 - timeoutSeconds：探针执行检测请求后，等待响应的超时时间，默认为1s，单位“秒”，最小值是1
 - successThreshold：探针检测失败后认为成功的最小连接成功次数，默认值为1。必须为1才能激活和启动。最小值为1。
 - failureThreshold：探测失败的重试次数，重试一定次数后将认为失败，默认值为3，最小值为1。

如果 pod 内部的容器不添加就绪探测，默认就绪。如果添加了就绪探测，只有就绪通过以后，标记才修改为就绪状态。当前 pod  内所有的容器都就绪，才标记当前 pod 就绪



#### 就绪探针实验

基于 HTTP GET 方式

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: readiness-httpget-pod
  labels:
    name: myapp
spec:
  containers:
  - name: readiness-httpget-container
    image: nginx:latest
    imagePullPolicy: IfNotPresent
    readinessProbe:
      httpGet:
        port: 80
        path: /index1.html
      initialDelaySeconds: 1
      periodSeconds: 3
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
```

```bash
# 当前处于没有就绪的状态
$ kubectl get pod
NAME                    READY   STATUS             RESTARTS        AGE
readiness-httpget-pod   0/1     Running            0               4m16s

# 创建一个index1.html
$ kubectl exec -it readiness-httpget-pod -c readiness-httpget-container -- /bin/bash
root@readiness-httpget-pod:/# echo "hehe" > /usr/share/nginx/html/index1.html

# 查看就已经处于就绪的状态了
$ kubectl get pod
NAME                    READY   STATUS             RESTARTS       AGE
readiness-httpget-pod   1/1     Running            0              5m40s

# 在运行过程中，就绪探测一直存在，如果不满足条件，会回到未就绪的情况
```

基于 EXEC 方式

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: readiness-exec-pod
  labels:
    name: myapp
spec:
  containers:
  - name: readiness-exec-container
    image: aaronxudocker/tools:busybox
    imagePullPolicy: IfNotPresent
    command: ["/bin/sh", "-c", "touch /tmp/live ; sleep 60; rm -rf /tmp/live; sleep 3600"]
    readinessProbe:
      exec:
        command: ["test", "-e", "/tmp/live"]
      initialDelaySeconds: 1
      periodSeconds: 3
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
```

```bash
# 可以看到在60秒后就变成非就绪状态了
$ kubectl get pod -w
NAME                 READY   STATUS    RESTARTS   AGE
readiness-exec-pod   1/1     Running   0          7s
readiness-exec-pod   0/1     Running   0          69s
```



基于TCP Check方式（使用较少）

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: readiness-tcp-pod
  labels:
    name: myapp
spec:
  containers:
  - name: readiness-tcp-container
    image: nginx:latest
    imagePullPolicy: IfNotPresent
    readinessProbe:
      tcpSocket:
        port: 80
      initialDelaySeconds: 1
      periodSeconds: 3
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
```



#### 就绪探针流量测试

<img src="E:\英格\图片\就绪探针流量测试.png" alt="就绪探针流量测试" style="zoom:80%;" />



在匹配可用pod的时候，标签必须匹配，状态必须是就绪状态。

```yaml
# pod-1.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-1
    image: nginx:latest
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"

# 两个配置文件放一起中间需要使用“-------------”分离
# pod-2.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-2
  labels:
    app: myapp
    version: v1
spec:
  containers:
  - name: myapp-1
    image: nginx:latest
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
        
 # 确认状态已经就绪
 $ kubectl get pod --show-labels
NAME    READY   STATUS    RESTARTS   AGE     LABELS
pod-1   1/1     Running   0          2m31s   app=myapp
pod-2   1/1     Running   0          32s     app=myapp,version=v1
```



创建service资源

```bash
# 注意myapp就是标签为app=myapp的pod
# 此处不需要理解，后面会细讲，只是用来验证就绪探针对流量的影响
# 此处的作用是形成多个pod的负载均衡
$ kubectl create svc clusterip myapp --tcp=80:80
service/myapp created
$ kubectl get svc
NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.0.0.1      <none>        443/TCP   46h
myapp        ClusterIP   10.8.74.201   <none>        80/TCP    6s
```



将两个pod中的主页文件修改一下，用来作为区分

```bash
# 如果pod中只有一个main容器，那么在exec的时候就不需要指定容器
$ kubectl exec -it pod-1 -- /bin/bash
root@pod-1:/# echo pod-1 > /usr/share/nginx/html/index.html
$ kubectl exec -it pod-2 -- /bin/bash
root@pod-2:/# echo pod-2 > /usr/share/nginx/html/index.html
```



验证负载均衡的状态

```bash
$ curl 10.8.74.201
pod-2
$ curl 10.8.74.201
pod-1
$ curl 10.8.74.201
pod-2
```



创建一个label为 `app: test` 的pod，看下是否能被匹配

```yaml
# 3.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-3
  labels:
    app: test
    version: v1
spec:
  containers:
  - name: myapp-1
    image: nginx:latest
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
```



查看pod状态，修改 `pod-3` 的网页内容

```bash
$ kubectl get pod --show-labels
NAME    READY   STATUS    RESTARTS   AGE     LABELS
pod-1   1/1     Running   0          11m     app=myapp
pod-2   1/1     Running   0          9m57s   app=myapp,version=v1
pod-3   1/1     Running   0          51s     app=test,version=v1

$ kubectl exec -it pod-3 -- /bin/bash
root@pod-3:/# echo pod-3 > /usr/share/nginx/html/index.html
```



验证负载均衡的状态,发现 `pod-3` 并不能被匹配上

```bash
$ curl 10.8.74.201
pod-1
$ curl 10.8.74.201
pod-2
$ curl 10.8.74.201
pod-1
$ curl 10.8.74.201
pod-2
$ curl 10.8.74.201
pod-2
$ curl 10.8.74.201
pod-1
```



修改pod-3的label

```bash
$ kubectl label pod pod-3 app=myapp --overwrite

$ curl 10.8.74.201
pod-2
$ curl 10.8.74.201
pod-3
$ curl 10.8.74.201
pod-1
```



创建一个不满足就绪条件的 `pod-4`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-4
  labels:
    app: myapp
    version: v1
spec:
  containers:
  - name: myapp-1
    image: nginx:latest
    readinessProbe:
      httpGet:
        port: 80
        path: /index1.html
      initialDelaySeconds: 1
      periodSeconds: 3
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
```



修改主页内容

```bash
$ kubectl exec -it pod-4 -- /bin/bash
root@pod-4:/# echo pod-4 > /usr/share/nginx/html/index.html
```



查看状态是未就绪的

```bash
$ kubectl get pod --show-labels
NAME    READY   STATUS    RESTARTS   AGE     LABELS
pod-1   1/1     Running   0          17m     app=myapp
pod-2   1/1     Running   0          15m     app=myapp,version=v1
pod-3   1/1     Running   0          6m49s   app=test,version=v1
pod-4   0/1     Running   0          41s     app=myapp,version=v1
```



验证负载均衡

```bash
$ curl 10.8.74.201
pod-1
$ curl 10.8.74.201
pod-2
$ curl 10.8.74.201
pod-1
$ curl 10.8.74.201
pod-2
$ curl 10.8.74.201
pod-2
$ curl 10.8.74.201
pod-1
```



满足 `pod-4` 的就绪条件

```bash
$ kubectl exec -it pod-4 -- /bin/bash
root@pod-4:/# touch /usr/share/nginx/html/index1.html
```



再次验证负载均衡

```bash
$ curl 10.8.74.201
pod-1
$ curl 10.8.74.201
pod-2
$ curl 10.8.74.201
pod-4
```



### livenessProbe存活探针

介绍：存活探针决定何时重启容器，解决虽然活着但是已经死了的问题。保证正在运行的容器能够提供正常的服务

 选项说明

 - initialDelaySeconds：容器启动后要等待多少秒后就探针开始工作，单位“秒”，默认是0秒，最小值是0
 - periodSeconds：执行探测的时间间隔（单位是秒），默认为10s，单位“秒”，最小值是1
 - timeoutSeconds：探针执行检测请求后，等待响应的超时时间，默认为1s，单位“秒”，最小值是1
 - successThreshold：探针检测失败后认为成功的最小连接成功次数，默认值为1，必须为1才能激活和启动。最小值为1。
 - failureThreshold：探测失败的重试次数，重试一定次数后将认为失败，默认值为3，最小值为1。



#### 存活探针实验

基于 Exec 方式

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-exec-pod
spec:
  containers:
  - name: liveness-exec-container
    image: aaronxudocker/tools:busybox
    imagePullPolicy: IfNotPresent
    command: ["/bin/sh", "-c", "touch /tmp/live; sleep 60; rm -rf /tmp/live; sleep 3600"]
    livenessProbe:
      exec:
        command: ["test", "-e", "/tmp/live"]
      initialDelaySeconds: 1
      periodSeconds: 3
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
```



一段时间以后，可以看到发生重启的事件

```bash
$ kubectl get pod -w
NAME                READY   STATUS    RESTARTS   AGE
liveness-exec-pod   1/1     Running   0          11s
liveness-exec-pod   1/1     Running   1 (1s ago)   101s
liveness-exec-pod   1/1     Running   2 (1s ago)   3m20s
```



基于 HTTP Get 方式

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-httpget-pod
spec:
  containers:
  - name: liveness-httpget-container
    image: nginx:latest
    imagePullPolicy: IfNotPresent
    livenessProbe:
      httpGet:
        port: 80
        path: /index.html
      initialDelaySeconds: 1
      periodSeconds: 3
      timeoutSeconds: 3
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
```



删除 `index.html` 文件，使其不满足存活探测条件

```bash
# 在删除index.html之后，可以看到命令行退出了
$ kubectl exec -it liveness-httpget-pod -- /bin/bash
root@liveness-httpget-pod:/# rm -f /usr/share/nginx/html/index.html 
root@liveness-httpget-pod:/# command terminated with exit code 137
```



重新查看pod状态，可以看到重启了

```bash
$ kubectl get pod
NAME                   READY   STATUS    RESTARTS      AGE
liveness-httpget-pod   1/1     Running   1 (48s ago)   2m39s
```



在运行此pod的节点上查看docker的名字，容器名字是 `集群名-pod名-容器名-hash-重启次数(初始是0)`

```bash
$ docker ps -a |grep liveness-exec-container
18c5ba02d684   39286ab8a5e1                                        "/docker-entrypoint.…"   About a minute ago   Up About a minute                         k8s_liveness-exec-container_liveness-httpget-pod_default_aa36504e-23a9-48d1-988c-4de0398c474f_1
54b3a04bd6b0   39286ab8a5e1                                        "/docker-entrypoint.…"   3 minutes ago        Exited (0) About a minute ago             k8s_liveness-exec-container_liveness-httpget-pod_default_aa36504e-23a9-48d1-988c-4de0398c474f_0
```



基于 TCP Check 方式

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-tcp-pod
spec:
  containers:
  - name: liveness-tcp-container
    image: nginx:latest
    imagePullPolicy: IfNotPresent
    livenessProbe:
      tcpSocket:
        port: 80
      initialDelaySeconds: 1
      periodSeconds: 3
      timeoutSeconds: 3
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
```



### startupProbe启动探针

介绍：k8s在1.16版本后增加startupProbe探针，用于对慢启动容器进行存活性检测，避免它们在启动运行之前就被 kubelet 杀掉。仅在启动时执行一次。

选项说明

 - initialDelaySeconds：容器启动后要等待多少秒后就探针开始工作，单位“秒”，默认是0秒，最小值是0
 - periodSeconds：执行探测的时间间隔（单位是秒），默认为10s，单位“秒”，最小值是1
 - timeoutSeconds：探针执行检测请求后，等待响应的超时时间，默认为1s，单位“秒”，最小值是1
 - successThreshold：探针检测失败后认为成功的最小连接成功次数，默认值为1。必须为1才能激活和启动。最小值为1。
 - failureThreshold：探测失败的重试次数，重试一定次数后将认为失败，默认值为3，最小值为1。



#### 启动探针实验

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: startupprobe-1
spec:
  containers:
  - name: startupprobe
    image: nginx:latest
    imagePullPolicy: IfNotPresent
    readinessProbe:
      httpGet:
        port: 80
        path: /index2.html
      initialDelaySeconds: 1
      periodSeconds: 3
    startupProbe:
      httpGet:
        path: /index1.html
        port: 80
      periodSeconds: 10
      failureThreshold: 30
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
        
# 应用程序将会有最多 5 分钟 failureThreshold * periodSeconds（30 * 10 = 300s）的时间来完成其启动过程。如果到超时都没有启动完成，就会重启。
```



创建 `index1.html` 文件

```bash
$ kubectl exec -it pod/startupprobe-1 -- /bin/bash
root@startupprobe-1:/# touch /usr/share/nginx/index1.html

# 查看依旧是未就绪的状态
$ kubectl get pod
NAME             READY   STATUS    RESTARTS   AGE
startupprobe-1   0/1     Running   0          42s

# 创建index2.html文件
$ kubectl exec -it pod/startupprobe-1 -- /bin/bash
root@startupprobe-1:/# touch /usr/share/nginx/index2.html

# 查看状态
$ kubectl get pod
NAME             READY   STATUS    RESTARTS   AGE
startupprobe-1   1/1     Running   0          43s
```

此时删掉启动探测的 `index1.html` 会怎样？



## Pod钩子

Podhook（钩子）是由Kubernetes管理的kubelet发起的，当容器中的进程启动前或者容器中的进程终止之前运行，这是包含在容器的⽣命周期之中。可以同时为Pod中的所有容器都配置hook

kubernetes包含两种钩子函数：

PostStart：在容器创建后立即执行，启动后钩子。用于资源部署、环境准备等。如果钩子花费时间太长以至于不能运行或者被挂起，容器将不能到达running状态。

PreStop：在容器准备终止前被调用，关闭前钩子。用于优雅的关闭应用程序，通知其他系统等。如果钩子在执行期间被挂起，pod会停留在running状态并且永远达不到failed状态。

当PostStart和PreStop失败，它会杀死容器。

Hook的类型包括两种：

- exec：执行一段命令
- HTTP：发送HTTP请求

在k8s中，理想的状态是pod优雅释放，但是并不是每一个Pod都会这么顺利

- Pod卡死，处理不了优雅退出的命令或者操作
- 优雅退出的逻辑有BUG，陷入死循环
- 代码问题，导致执行的命令没有效果

对于以上问题，k8s的Pod终止流程中还有一个”最多可以容忍的时间”，即graceperiod(在pod.spec.terminationGracePeriodSeconds字段定义)，这个值默认是30秒，当我们执行kubectl delete的时候也可以通过—grace-period参数显示指定一个优雅退出时间来覆盖Pod中的配置，如果我们配置的grace period超过时间之后，k8s就只能选择强制kill Pod。

值得注意的是，这与preStopHook和SIGTERM信号并行发⽣。k8s不会等待preStopHook完成。你的应用程序应在terminationGracePeriod之前退出。



### Pod钩子实验

基于 exec 方式

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hook-exec-pod
spec:
  containers:
  - name: hook-exec-container
    image: nginx:latest
    imagePullPolicy: IfNotPresent
    lifecycle:
      postStart:
        exec:
          command: ["/bin/sh", "-c", "echo postStart > /usr/share/message"]
      preStop:
        exec:
          command: ["/bin/sh", "-c", "echo preStart > /usr/share/message"]
    resources:
      limits:
        memory: "128MiB"
        cpu: "500m"
```



在这个pod的内部，写一个循环查看此文件的shell命令

```bash
$ kubectl exec -it pod/hook-exec-pod -- /bin/bash
root@hook-exec-pod:/# while true;
> do 
> cat /usr/share/message
> done

# 删除此pod就能看到结束的钩子信息了
```



基于 HTTP Get 方式

```bash
# 开启一个测试 webserver
$ docker run -it --rm -p 1234:80 nginx:latest
```



启动一个pod，然后再删除，查看nginx容器日志，可以看到记录了这两次的http请求

```yaml
2024/09/06 07:35:23 [error] 29#29: *1 open() "/usr/share/nginx/html/poststarthook.html" failed (2: No such file or directory), client: 192.168.173.101, server: localhost, request: "GET /poststarthook.html HTTP/1.1", host: "192.168.173.100:1234"
192.168.173.101 - - [06/Sep/2024:07:35:23 +0000] "GET /poststarthook.html HTTP/1.1" 404 153 "-" "kube-lifecycle/1.29" "-"
2024/09/06 07:35:45 [error] 29#29: *1 open() "/usr/share/nginx/html/prestophook.html" failed (2: No such file or directory), client: 192.168.173.101, server: localhost, request: "GET /prestophook.html HTTP/1.1", host: "192.168.173.100:1234"
192.168.173.101 - - [06/Sep/2024:07:35:45 +0000] "GET /prestophook.html HTTP/1.1" 404 153 "-" "kube-lifecycle/1.29" "-"
```



## 总结

Pod⽣命周期中的initC、startupProbe、livenessProbe、readinessProbe、hook都是可以并且存在的，可以选择全部、部分或者完全不用。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: lifecycle-pod
  labels:
    app: lifecycle-pod
spec:
  containers:
  - name: busybox-container
    image: aaronxudocker/tools:busybox
    command: ["/bin/sh","-c","touch /tmp/live ; sleep 600; rm -rf /tmp/live; sleep 3600"]
    livenessProbe:
      exec:
        command: ["test","-e","/tmp/live"]
      initialDelaySeconds: 1
      periodSeconds: 3
    lifecycle:
      postStart:
        httpGet:
          host: 192.168.173.100
          path: poststarthook.html
          port: 1234
      preStop:
        httpGet:
          host: 192.168.173.100
          path: prestophook.html
          port: 1234
      resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
  - name: myapp-container
    image: nginx:latest
    livenessProbe:
      httpGet:
        port: 80
        path: /index.html
      initialDelaySeconds: 1
      periodSeconds: 3
      timeoutSeconds: 3
    readinessProbe:
      httpGet:
        port: 80
        path: /index1.html
        initialDelaySeconds: 1
        periodSeconds: 3
    initContainers:
    - name: init-myservice
      image: aaronxudocker/tools:busybox
      command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
    - name: init-mydb
      image: aaronxudocker/tools:busybox
      command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```



## 调度Pod

![调度Pod](E:\英格\图片\调度Pod.png)

用户把自己的镜像通过docker本地客户端推送到云端仓库中，用户发送命令给本地kubectl，kubectl向api server发起rest请求，api server把请求保存在etcd中，调度器scheduler监听api server确认有没有pod还没有被分配节点，如果有，就根据调度算法为pod分配工作节点，node的kubelet监听api server是否有需要创建的Pod，有就调用CRI（容器进行时）接口实现对应的容器创建。

# 六、Pod控制器

## 控制器

在Kubernetes中运⾏了⼀系列控制器来确保集群的当前状态与期望状态保持⼀致，它们就是Kubernetes集群内部的管理控制中⼼或者说是”中⼼⼤脑”。例如，ReplicaSet控制器负责维护集群中运⾏的Pod数量；Node控制器负责监控节点的状态，并在节点出现故障时，执⾏⾃动化修复流程，确保集群始终处于预期的⼯作状态。

控制器种类：

- ReplicationController 和 ReplicaSet
- Deployment
- DaemonSet
- StateFulSet
- Job/CronJob
- Horizontal Pod Autoscalin



## ReplicationController和ReplicaSet

ReplicationController(RC)⽤来确保容器应⽤的副本数始终保持在⽤户定义的副本数，即如果有容器异常退出，会⾃动创建新的Pod来替代；⽽如果异常多出来的容器也会⾃动回收；

在新版本的Kubernetes中建议使⽤ReplicaSet(RS)来取代ReplicationController。ReplicaSet跟ReplicationController没有本质的不同，只是名字不⼀样，并且ReplicaSet⽀持集合式的selector；多了标签选择的运算方式

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: rc-demo
spec:
  replicas: 3
  selector:
    app: rc-demo
  template:
    metadata:
      name: rc-demo
      labels:
        app: rc-demo
    spec:
      containers:
        - name: rc-demo-container
          image: aaronxudocker/myapp:v1.0
          ports:
            - containerPort: 80
            
# kubectl delete pod rc-demo-fl7vb
# kubectl label pod rc-demo-fl7vb app=rc-demo1 --overwrite
```

 ReplicaSet(RS)控制器

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: rc-ml-demo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: rc-ml-demo
  template:
    metadata:
      name: rc-ml-demo
      labels:
        app: rc-ml-demo
    spec:
      containers:
        - name: rc-ml-demo-container
          image: aaronxudocker/myapp:v1.0
          ports:
            - containerPort: 80
```



### selector.matchExpressions

rs在标签选择器上，除了可以定义键值对的选择形式，还支持matchExpressions字段，可以提供多种选择。
目前支持的操作包括：

- In：label的值在某个列表中
- NotIn：label的值不在某个列表中
- Exists：某个label存在
- DoesNotExist：某个label不存在

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: rc-me-demo
spec:
  replicas: 3
  selector:
    matchExpressions:
      - key: app
        operator: Exists
  template:
    metadata:
      name: rc-me-demo
      labels:
        app: rc-me-demo
    spec:
      containers:
        - name: rc-me-demo-container
          image: aaronxudocker/myapp:v1.0
          ports:
            - containerPort: 80
```

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: rc-me-in-demo
spec:
  replicas: 3
  selector:
    matchExpressions:
      - key: app
        operator: In
        values:
        - spring-k8s
        - xixixi
  template:
    metadata:
      name: rc-me-in-demo
      labels:
        app: rc-me-in-demo
    spec:
      containers:
        - name: rc-me-in-demo-container
          image: aaronxudocker/myapp:v1.0
          ports:
            - containerPort: 80
```



## Deployment

Deployment 用于管理RC和RS，使它们运行一个应用负载的一组 Pod，通常适用于不保持状态的负载。

声明性的东⻄是对终结果的陈述，表明意图⽽不是实现它的过程。在Kubernetes中，这就是说“应该有⼀个包含三个Pod的ReplicaSet”。
命令式充当命令。声明式是被动的，⽽命令式是主动且直接的：“创建⼀个包含三个Pod的ReplicaSet”。

```bash
$ kubectl replace -f deployment.yaml
$ kubectl apply -f deployment.yaml
$ kubectl diff -f deployment.yaml
```



### 替换⽅式

- kubectl replace:使⽤新的配置完全替换掉现有资源的配置。这意味着新配置将覆盖现有资源的所有字段和属性，包括未指定的字段，会导致整个资源的替换
- kubectl apply:使⽤新的配置部分地更新现有资源的配置。它会根据提供的配置文件或参数，只更新与新配置中不同的部分，⽽不会覆盖整个资源的配置



### 字段级别的更新

- kubectl replace:由于是完全替换，所以会覆盖所有字段和属性，⽆论是否在新配置中指定
- kubectl apply:只更新与新配置中不同的字段和属性，保留未指定的字段不受影响



#### 部分更新

- kubectl replace:不⽀持部分更新，它会替换整个资源的配置
- kubectl apply:⽀持部分更新，只会更新新配置中发⽣变化的部分，保留未指定的部分不受影响



#### 与其他配置的影响

- kubectl replace:不考虑其他资源配置的状态，直接替换资源的配置
- kubectl apply:可以结合使⽤-f或-k参数，从文件或⽬录中读取多个资源配置，并根据当前集群中的资源状态进⾏更新



### Deployment介绍

deployment指挥kubernetes如何创建和更新应用的实例，创建deployment后，kubernetes控制平面将deployment中包含的应用实例调度到集群中的各个节点上。创建应用实例后，kubernetes deployment控制器会持续监视这些实例，如果托管实例的节点关闭或者被删除，则deployment控制器会将该实例替换为集群中另一个节点上的实例

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels: 
    app: myapp-deploy
  name: myapp-deploy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp-deploy
  template:
    metadata:
      labels:
        app: myapp-deploy
    spec:
      containers:
      - name: myapp
        image: aaronxudocker/myapp:v1.0
        resources:
          limits:
            memory: "128MiB"
            cpu: "500m"
        ports:
        - containerPort: 80
```



### 更新策略

Deployment可以保证在升级时只有⼀定数量的Pod是down的。默认的，它会确保⾄少有比期望的Pod数量少⼀个是up状态（最多1个不可⽤）

Deployment同时也可以确保只创建出超过期望数量的⼀定数量的Pod。默认的，它会确保最多比期望的Pod数量多⼀个的Pod是up的（最多超过一个）

未来的Kuberentes版本中，将从+-1变+-25%。目前的版本已经支持。

通过修改kubectlexplaindeploy.spec.strategy.type以达到滚动更新的数量的变化

Recreate：在创建出新的Pod之前会先杀掉所有已存在的Pod

rollingUpdate：滚动更新，就是杀死一部分，就启动一部分，在更新过程中，存在两个版本Pod
- maxSurge：指定超出副本数有⼏个，两种⽅式：1、指定数量2、百分比
- maxUnavailable：最多有⼏个不可⽤

```bash
$ kubectl create -f deployment.yaml --record # 基于deployment资源清单以命令式实现控制器的创建，如果此文件描述的对象存在，那么即使文件描述的信息发生了变化，再次提交时也不会应用
# --record 参数可以记录命令，可以看到每次revision的变化
$ kubectl apply -f deployment.yaml #如果目标对象与文件本身发生改变，那么会根据文件指定的地方进行部分更新。
$ kubectl replace -f deployment.yaml # 如果目标对象与文件本身发生改变，那么会重建此对象进行替换。
$ kubectl scale deployment myapp-deploy --replicas 10 # 调节当前容器副本数量，也可以用于扩缩容
$ kubectl autoscale deployment myapp-deploy --min=10 --max=15 --cpu-percent=80 #自动扩缩容
$ kubectl set image deployment/myapp-deploy myapp=aaronxudocker/myapp:v2.0 # 修改当前deployment控制器 pod内部的镜像
$ kubectl rollout history deployment/myapp-deploy
$ kubectl rollout undo deployment/myapp-deploy
```

可以通过设置`.spec.revisionHistoryLimit`项来指定deployment最多保留多少revision历史记录。默认的会保留所有的revision；如果将该项设置为0，Deployment就不允许回退了

当存在两种并行状态时，不是容器先都到中间态再到最终态，而是所有容器不管在哪个状态，都变成最终态。

即kubernetes只会选择最终的版本进行部署，忽略中间的版本的执行



### 金丝雀部署

⾦丝雀部署的核⼼思想是在实际运⾏环境中的⼀⼩部分⽤户或流量上测试新版本的软件，⽽⼤部分⽤户或流量仍然使⽤旧版本。通过对新版本进⾏有限范围的实时测试和监控，可以及早发现潜在的问题，并减少对整个系统的冲击。

```bash
$ kubectl patch deployment myapp-deploy -p '{"spec":{"strategy":{"rollingUpdate":{"maxSurge":1,"maxUnavailable":0}}}}'
# 为myspp-deploy的deployment控制器打补丁：允许pod多出预期1个，不允许少一个

$ kubectl patch deployment myapp-deploy --patch '{"spec": {"template": {"spec": {"containers": [{"name": "myapp","image":"aaronxudocker/myapp:v2.0"}]}}}}'   &&  kubectl rollout pause deploy  myapp-deploy
# 修改镜像为v2.0版本，并且立马停止回滚，可以看到pod数量复合上面的设置

$ kubectl rollout resume deploy myapp-deploy # 原理：通过控制不同的RS当前的期望值来实现不同版本之间的回滚。
# 恢复回滚动作
# v1升级到v2，v2升级到v3，回滚一次到v2版本，再回滚一次回到v3
```

回滚相关的命令

```bash
$ kubectl rollout status deployment myapp-deploy
# 查看上次回滚的状态

$ kubectl rollout history deployment/myapp-deploy
# 查看回滚的历史记录，需要配合--record使用

$ kubectl rollout undo deployment/myapp-deploy --to-revision=2
# 状态回退到李四记录里面的2的状态
# 或者通过apply标记修改过的文件进行手动回滚

$ kubectl rollout pause deployment/myapp-deploy
# 暂停回滚
```



## DaemonSet

DaemonSet确保全部（或者⼀些）Node上运⾏⼀个Pod的副本。当有Node加入集群时，也会为他们新增⼀个Pod。当有Node从集群移除时，这些Pod也会被回收。删除DaemonSet将会删除它创建的所有Pod

使⽤DaemonSet的⼀些典型⽤法：

- 运⾏集群存储daemon，例如在每个Node上运⾏`glusterd`、`ceph`
- 在每个Node上运⾏⽇志收集daemon，例如`fluentd`、`logstash`
- 在每个Node上运⾏监控daemon，例如PrometheusNode Exporter、`collectd`、Datadog代理、NewRelic代理，或Ganglia`gmond`

动态地保证当前节点有且仅有一个pod在运行

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: deamonset-demo
  namespace: default
  labels:
    app: deamonset-demo
spec:
  selector:
    matchLabels:
      app: deamonset-demo
  template:
    metadata:
      labels:
        app: deamonset-demo
    spec:
      containers:
      - name: deamonset-demo
        image: aaronxudocker/myapp:v1.0
```



## Job

Job负责批处理任务，即仅执⾏⼀次的任务，它保证批处理任务的⼀个或多个Pod成功结束

特殊说明

- spec.template格式同Pod
- RestartPolicy仅⽀持Never或OnFailure
- 单个Pod时，默认Pod成功运⾏后Job即结束
- `.spec.completions`标志Job结束需要成功运⾏的Pod个数，默认为1
- `.spec.parallelism`标志并⾏运⾏的Pod的个数，默认为1，为1时，两个Pod先后执行；超过1时，并发执行
- `spec.activeDeadlineSeconds`标志失败Pod的重试最⼤时间，超过这个时间不会继续重试



### 案例：使用马青公式计算圆周率后一千位

可以用来看下节点的计算能力

 这个公式由英国天文学教授约翰·⻢青于1706年发现。他利⽤这个公式计算到了100位的圆周率。⻢青公式每计算⼀项可以得到1.4位的⼗进制精度。因为它的计算过程中被乘数和被除数都不⼤于⻓整数，所以可以很容易地在计算机上编程实现

![](E:\英格\图片\马青公式计算圆周率后一千位.png)



```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: job-demo
spec:
  template:
    metadata:
      name: job-demo-pod
    spec:
      containers:
      - name: job-demo
        image: aaronxudocker/tools:maqingpythonv1
      restartPolicy: Never
```

Job负责批处理任务，即仅执⾏⼀次的任务，它保证批处理任务的⼀个或多个Pod成功结束



### 错误的Job任务

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: rand
spec:
  template:
    metadata:
      name: rand
    spec:
      containers:
      - name: rand
        image: aaronxudocker/tools:randexitv1
        args: ["--exitcode=1"]
      restartPolicy: Never
```

Job控制器会记录成功的次数

```bash
$ kubectl get job 
NAME   COMPLETIONS   DURATION   AGE
rand   0/1           30s        30s
```

改为产生随机返回值

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: rand
spec:
  # completions: 10
  # parallelism: 5
  template:
    metadata:
      name: rand
    spec:
      containers:
      - name: rand
        image: aaronxudocker/tools:randexitv1
      restartPolicy: Never
```

可以看到并行启动5个pod，直到成功为止

```bash
$ kubectl get job -w
NAME   COMPLETIONS   DURATION   AGE
rand   2/10          24s        24s
rand   2/10          50s        50s
rand   2/10          52s        52s
rand   2/10          58s        58s
rand   2/10          59s        59s
rand   3/10          59s        59s
rand   3/10          60s        60s
rand   5/10          60s        60s
rand   5/10          61s        61s
rand   5/10          62s        62s
rand   5/10          67s        67s
rand   5/10          68s        68s
rand   8/10          68s        68s
rand   8/10          69s        69s
rand   8/10          69s        69s


$ kubectl get job -o yaml
......
  spec:
    backoffLimit: 6
# 6次尝试失败，就达到尝试上限
```



## CronJob

CronJob管理基于时间的Job，即：

- 在给定时间点只运⾏⼀次
- 周期性地在给定时间点运⾏

使⽤条件：当前使⽤的Kubernetes集群，版本>=1.8（对CronJob）。创建CronJob操作应该是幂等的。

典型的⽤法如下所⽰：

- 在给定的时间点调度Job运⾏

- 创建周期性运⾏的Job，例如：数据库备份、发送邮件

`.spec.schedule`：调度，必需字段，指定任务运⾏周期，格式同Cron

`.spec.jobTemplate`：Job模板，必需字段，指定需要运⾏的任务，格式同Job

`.spec.startingDeadlineSeconds`：启动Job的期限（秒级别），该字段是可选的。如果因为任何原因⽽错过了被调度的时间，那么错过执⾏时间的Job将被认为是失败的。如果没有指定，则没有期限

`.spec.concurrencyPolicy`：并发策略，该字段也是可选的。它指定了如何处理被CronJob创建的Job的并发执⾏。只允许指定下⾯策略中的⼀种：
	`Allow`（默认）：允许并发运⾏Job
	`Forbid`：禁⽌并发运⾏，如果前⼀个还没有完成，则直接跳过下⼀个
	`Replace`：取消当前正在运⾏的Job，⽤⼀个新的来替换
	注意，当前策略只能应⽤于同⼀个CronJob创建的Job。如果存在多个CronJob，它们创建的Job之间总是允许并发运⾏

`.spec.suspend`：挂起，该字段也是可选的。如果设置为`true`，后续所有执⾏都会被挂起。它对已经开始执⾏的Job不起作⽤。默认值为`false`。适用于项目暂停为后面再启用做准备

`.spec.successfulJobsHistoryLimit`和`.spec.failedJobsHistoryLimit`：历史限制，是可选的字段。它们指定了可以保留多少完成和失败的Job。默认情况下，它们分别设置为`3`和`1`，表示保留三个最新的成功的job控制器信息，一个最新的失败的Job控制器信息。设置限制的值为`0`，相关类型的Job完成后将不会被保留

cronjob只能做到分钟级别的控制，若需要秒级控制，需要进入job控制器内部的Pod里实现。

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cronjob-demo
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      completions: 3
      template:
        spec:
          containers:
          - name: cronjob-demo-container
            image: aaronxudocker/tools:busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```

创建job操作应该是幂等的



## Horizontal Pod Autoscaler(HPA)

已经可以实现通过手工执行`kubectl scale`命令实现Pod扩容或缩容，但是这显然不符合Kubernetes的定位目标—自动化、智能化。 Kubernetes期望可以实现通过监测Pod的使用情况，实现pod数量的自动调整，于是就产生了Horizontal Pod Autoscaler（HPA）这种控制器。

HPA可以获取每个Pod利用率，然后和HPA中定义的指标进行对比，同时计算出需要伸缩的具体值，最后实现Pod的数量的调整。其实HPA与之前的Deployment一样，也属于一种Kubernetes资源对象，它通过追踪分析RC控制的所有目标Pod的负载变化情况，来确定是否需要针对性地调整目标Pod的副本数，这是HPA的实现原理。



查看pod的资源

```bash
# 由于没安装metrics服务，所以无法查看pod资源情况
$ kubectl top pod myapp-deploy-57bff895d5-cvlmg
error: Metrics API not available
```



安装metrics-server

```bash
$ wget https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.7.2/components.yaml

# 修改配置文件
- args:
  - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname,InternalDNS,ExternalDNS
  - --kubelet-insecure-tls
  image: registry.k8s.io/metrics-server/metrics-server:v0.7.2
  
$ kubectl apply -f components.yaml

$ kubectl get pod -n kube-system |grep metrics
metrics-server-5c7fbc6ff6-fssfd            1/1     Running   0             2m42s

# 使用kubectl top node 查看资源使用情况
$ kubectl top node
NAME       CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
master01   116m         2%     1943Mi          55%       
node01     48m          1%     961Mi           27%       
node02     61m          1%     1161Mi          32%

$ kubectl top pod myapp-deploy-57bff895d5-cvlmg
NAME                            CPU(cores)   MEMORY(bytes)   
myapp-deploy-57bff895d5-cvlmg   0m           4Mi
```



准备deployment和service资源

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-pod
  template:
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
      - name: nginx
        image: aaronxudocker/myapp:v1.0
        resources:
          limits:
            cpu: "1"
          requests:
            cpu: "100m"
```



创建service并且查看

```bash
$ kubectl expose deployment nginx --type=NodePort --port=80

$ kubectl get deployment,pod,svc
NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx          1/1     1            1           114s

NAME                                READY   STATUS    RESTARTS   AGE
pod/nginx-7b4664bb57-pgllw          1/1     Running   0          114s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
service/nginx        NodePort    10.8.51.98   <none>        80:30482/TCP   37s
```



创建HPA资源

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-demo
spec:
  minReplicas: 1  #最小pod数量
  maxReplicas: 10 #最大pod数量
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 10 # 在连续10秒内稳定后才考虑进一步扩展操作
      policies:
      - type: Percent
        value: 100 # 当超过当前副本数的100%才进行扩展 
        periodSeconds: 3 # 两次扩容之间的最小时间间隔
    scaleDown:
      stabilizationWindowSeconds: 300 # 在连续300秒内稳定后才考虑缩减操作
      policies:
      - type: Percent
        value: 10 # 当低于当前副本数的10%才进行缩减 
        periodSeconds: 3 # 两次缩减之间的最小时间间隔
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 10 # CPU平均利用率目标设为10%
  scaleTargetRef:   # 指定要控制的nginx信息
    apiVersion: apps/v1
    kind: Deployment
    name: nginx
```



查看hpa资源

```bash
$ kubectl get hpa -w
NAME       REFERENCE          TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
hpa-demo   Deployment/nginx   0%/10%    1         10        1          20s
```



使用压力测试工具

```bash
$ ab -c 1000 -n 1000000 http://192.168.173.100:30482/

$ kubectl get hpa -w
NAME       REFERENCE          TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
hpa-demo   Deployment/nginx   0%/10%    1         10        1          20s
hpa-demo   Deployment/nginx   1%/10%    1         10        1          105s
hpa-demo   Deployment/nginx   366%/10%   1         10        1          2m
hpa-demo   Deployment/nginx   522%/10%   1         10        2          2m15s
hpa-demo   Deployment/nginx   352%/10%   1         10        4          2m30s
hpa-demo   Deployment/nginx   189%/10%   1         10        8          2m45s
hpa-demo   Deployment/nginx   103%/10%   1         10        10         3m
hpa-demo   Deployment/nginx   68%/10%    1         10        10         3m15s
```



压力测试停止之后

```bash
$ kubectl get hpa -w
NAME       REFERENCE          TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
hpa-demo   Deployment/nginx   0%/10%     1         10        10         8m45s
hpa-demo   Deployment/nginx   0%/10%     1         10        9          9m
hpa-demo   Deployment/nginx   0%/10%     1         10        8          9m15s
hpa-demo   Deployment/nginx   0%/10%     1         10        7          9m30s
hpa-demo   Deployment/nginx   0%/10%     1         10        6          9m45s
hpa-demo   Deployment/nginx   0%/10%     1         10        5          10m
hpa-demo   Deployment/nginx   0%/10%     1         10        4          10m
hpa-demo   Deployment/nginx   0%/10%     1         10        3          10m
hpa-demo   Deployment/nginx   0%/10%     1         10        2          10m
hpa-demo   Deployment/nginx   0%/10%     1         10        1          11m
```



# 七、service

## 概念

Kubernetes`Service`定义了这样⼀种抽象：⼀个`Pod`的逻辑分组，⼀种可以访问它们的策略，通常称为微服务。这⼀组`Pod`能够被`Service`访问到，通常是通过`LabelSelector`



### Service类型

ClusterIp：默认类型，自动分配一个仅 Cluster 内部可以访问的虚拟 IP

NodePort：在 ClusterIP 基础上为 Service 在每台机器上绑定一个端口，这样就可以通过 <NodeIP>:

NodePort 来访问该服务

LoadBalancer：在 NodePort 的基础上，借助 cloud provider 创建一个外部负载均衡器，并将请求转

发到  <NodeIP> : NodePort

ExternalName：把集群外部的服务引入到集群内部来，在集群内部直接使用。没有任何类型代理被创建，

这只有 kubernetes 1.7 或更高版本的 kube-dns 才支持



## 工作模式

kube-proxy目前支持三种工作模式:

### userspace模式

userspace模式下，kube-proxy会为每一个Service创建一个监听端口，发向Cluster IP的请求被Iptables规则重定向到kube-proxy监听的端口上，kube-proxy根据LB算法选择一个提供服务的Pod并和其建立链接，以将请求转发到Pod上。 该模式下，kube-proxy充当了一个四层负责均衡器的角色。由于kube-proxy运行在userspace中，在进行转发处理时会增加内核和用户空间之间的数据拷贝，虽然比较稳定，但是效率比较低。

kube-proxy 监听当前  api-server 的负载均衡规则的需要改变的信息。每个节点的各种节点pod先要访问本地的防火墙规则，防火墙将当前流量转发至当前节点的  kube-proxy 或者别的结点的 kube-proxy ，kube-proxy 代理 server-pod 返回给当前的 client-pod 来实现负载均衡。Kube-Proxy 功能：根据监听 service 对象来修改防火墙规则，实现负载均衡的分发；代理用户的请求，返回给客户端。



### iptables模式

iptables模式下，kube-proxy为service后端的每个Pod创建对应的iptables规则，直接将发向Cluster IP的请求重定向到一个Pod IP。该模式下kube-proxy不承担四层负责均衡器的角色，只负责创建iptables规则。该模式的优点是较userspace模式效率更高，但不能提供灵活的LB策略，当后端Pod不可用时也无法进行重试。

kube-proxy 将当前监听的结果写入本机的防火墙规则，后端访问变成了由防火墙转发到本地服务器和远程服务器。kube-proxy 仅监听 api-server 将 service 变化修改本地的 iptables  规则



### ipvs模式

类似Linux架构下的LVS。ipvs模式和iptables类似，kube-proxy监控Pod的变化并创建相应的ipvs规则。ipvs相对iptables转发效率更高。除此以外，ipvs支持更多的LB算法。

kube-proxy 监听api-server 将当前的 service 的集群的负载均衡信息转换成 ipvs 规则，记录在本地机器中。每个机器的客户端 pod 访问本机器的ipvs规则被负载到随机一个 server-pod 上。此功能需要在 Linux 内核中开启

```yaml
# 创建三个pod
apiVersion: apps/v1
kind: Deployment
metadata:
  labels: 
    app: myapp-deploy
  name: myapp-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp-deploy
  template:
    metadata:
      labels:
        app: myapp-deploy
    spec:
      containers:
      - name: myapp
        image: aaronxudocker/myapp:v1.0
        resources:
          limits:
            memory: "128MiB"
            cpu: "500m"
        ports:
        - containerPort: 80
```



```bash
# 启动一个负载均衡的service
$ kubectl create svc clusterip myapp-deploy --tcp=80:80

# 修改ipvs
$ kubectl edit configmap kube-proxy -n kube-system
mode: "ipvs"

# an'zh删除kube-proxy的pod
$ kubectl delete pod -n kube-system -l k8s-app=kube-proxy
pod "kube-proxy-ckwsj" deleted
pod "kube-proxy-t729f" deleted
pod "kube-proxy-z6dt8" deleted

# 查看pod创建的状态
$ kubectl get pod -n kube-system -l k8s-app=kube-proxy
NAME               READY   STATUS    RESTARTS   AGE
kube-proxy-948s5   1/1     Running   0          3s
kube-proxy-ggpwj   1/1     Running   0          3s
kube-proxy-v7lgs   1/1     Running   0          3s

# 查看虚拟IP地址
$ kubectl get svc
NAME           TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
myapp-deploy   ClusterIP   10.9.86.78   <none>        80/TCP    6m54s

# 查看ipvsadm的状态
$ ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn      
TCP  10.9.86.78:80 rr
  -> 10.244.140.106:80            Masq    1      0          0         
  -> 10.244.196.141:80            Masq    1      0          0         
  -> 10.244.196.142:80            Masq    1      0          0
  
# 负载均衡的地址正好对应着pod的ip地址
$ kubectl get pod -o wide
NAME     READY   STATUS    RESTARTS   AGE   IP               NODE     NOMINATED NODE   READINESS
myapp-deploy-57bff895d5-b2hhk   1/1     Running   0          73s   10.244.196.142   node01   <none>           <none>
myapp-deploy-57bff895d5-fbln4   1/1     Running   0          73s   10.244.140.106   node02   <none>           <none>
myapp-deploy-57bff895d5-frnfd   1/1     Running   0          73s   10.244.196.141   node01   <none>           <none>
```



## Service资源清单

```yaml
kind: Service  # 资源类型
apiVersion: v1  # 资源版本
metadata: # 元数据
  name: service # 资源名称
  namespace: default # 命名空间
spec: # 描述
  selector: # 标签选择器，用于确定当前service代理哪些pod
    app: nginx
  type: # Service类型，指定service的访问方式
  clusterIP:  # 虚拟服务的ip地址
  sessionAffinity: # session亲和性，支持ClientIP、None两个选项
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 120  # session的过期时间
  ports: # 端口信息
    - protocol: TCP 
      port: 3017  # service端口
      targetPort: 5003 # pod端口
      nodePort: 31122 # 主机端口
```

可以使用如下命令得到基本的yaml格式的文件

```bash
$ kubectl create svc clusterip nginx --tcp=80:80 --dry-run=client -o yaml
$ ipvsadm -lnc
```

`spec.type`可以选择的类型

- ClusterIP：默认值，它是Kubernetes系统自动分配的虚拟IP，只能在集群内部访问
- NodePort：将Service通过指定的Node上的端口暴露给外部，通过此方法，就可以在集群外部访问服务
- LoadBalancer：使用外接负载均衡器完成到服务的负载分发，注意此模式需要外部云环境支持
- ExternalName： 把集群外部的服务引入集群内部，直接使用



## Service使用

```yaml
# 创建三个pod
apiVersion: apps/v1
kind: Deployment
metadata:
  labels: 
    app: myapp-deploy
  name: myapp-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp-deploy
  template:
    metadata:
      labels:
        app: myapp-deploy
    spec:
      containers:
      - name: myapp
        image: aaronxudocker/myapp:v1.0
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 80
```



测试三个pod

```bash
$ kubectl get pod -o wide
NAME      READY   STATUS    RESTARTS   AGE   IP         NODE     NOMINATED NODE   READINESS GATES
myapp-deploy-57bff895d5-b2hhk   1/1     Running   0          30m   10.244.196.142   node01   <none>           <none>
myapp-deploy-57bff895d5-fbln4   1/1     Running   0          30m   10.244.140.106   node02   <none>           <none>
myapp-deploy-57bff895d5-frnfd   1/1     Running   0          30m   10.244.196.141   node01   <none>           <none>

# 查看一下访问情况
$ curl 10.244.196.142/hostname.html
myapp-deploy-57bff895d5-b2hhk
$ curl 10.244.140.106/hostname.html
myapp-deploy-57bff895d5-fbln4
$ curl 10.244.196.141/hostname.html
myapp-deploy-57bff895d5-frnfd
```



### ClusterIP类型的Service（默认）

集群内部ip上公开service，只能从集群内访问

```yaml
apiVersion: v1
kind: Service
metadata:
  name: service-clusterip
spec:
  selector:
    app: myapp-deploy
  # clusterIP: 172.16.66.66 # service的ip地址，如果不写，默认会生成一个
  type: ClusterIP
  ports:
  - port: 80  # Service端口       
    targetPort: 80 # pod端口
```



查看运行结果

```bash
$ kubectl get svc
NAME                TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service-clusterip   ClusterIP   10.13.125.29   <none>        80/TCP    22s

$ kubectl describe svc service-clusterip 
Name:              service-clusterip
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=myapp-deploy
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.13.125.29
IPs:               10.13.125.29
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.244.140.106:80,10.244.196.141:80,10.244.196.142:80
Session Affinity:  None
Events:            <none>

$ ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn     
TCP  10.13.125.29:80 rr
  -> 10.244.140.106:80            Masq    1      0          0         
  -> 10.244.196.141:80            Masq    1      0          0         
  -> 10.244.196.142:80            Masq    1      0          0     
  
$ while true;do curl 10.13.125.29/hostname.html; done
myapp-deploy-57bff895d5-b2hhk
myapp-deploy-57bff895d5-frnfd
myapp-deploy-57bff895d5-fbln4
myapp-deploy-57bff895d5-b2hhk
myapp-deploy-57bff895d5-frnfd
myapp-deploy-57bff895d5-fbln4
```



### Endpoint

Endpoint是kubernetes中的一个资源对象，存储在etcd中，用来记录一个service对应的所有pod的访问地址，它是根据service配置文件中selector描述产生的。必须要满足就绪探测。

一个Service由一组Pod组成，这些Pod通过Endpoints暴露出来，**Endpoints是实现实际服务的端点集合**。换句话说，service和pod之间的联系是通过endpoints实现的。

```bash
$ kubectl get endpoints -o wide
NAME                ENDPOINTS                                               AGE
service-clusterip   10.244.140.106:80,10.244.196.141:80,10.244.196.142:80   6m27s
```

在deployment中添加一个就绪探测

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels: 
    app: myapp-deploy
  name: myapp-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp-deploy
  template:
    metadata:
      labels:
        app: myapp-deploy
    spec:
      containers:
      - name: myapp
        image: aaronxudocker/myapp:v1.0
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        readinessProbe:
          httpGet:
            port: 80
            path: /index1.html
          initialDelaySeconds: 1
          periodSeconds: 3
        ports:
        - containerPort: 80
```

在不满足就绪探测的情况下，是不会被endpoint采用的

```bash
$ kubectl get pod
NAME                            READY   STATUS    RESTARTS   AGE
myapp-deploy-659f9975b8-2sntn   0/1     Running   0          40s
myapp-deploy-659f9975b8-nd66b   0/1     Running   0          40s
myapp-deploy-659f9975b8-p4j5k   0/1     Running   0          40s

$ kubectl get endpoints
NAME                ENDPOINTS              AGE
service-clusterip                          10s
```

满足了就绪探测和标签被匹配上的pod会被加入endpoint中

```bash
$ kubectl exec -it myapp-deploy-659f9975b8-2sntn -- /bin/bash
root@myapp-deploy-659f9975b8-2sntn:/# echo "hello world" > /usr/share/nginx/html/index1.html

$ kubectl get pod
NAME                            READY   STATUS    RESTARTS   AGE
myapp-deploy-659f9975b8-2sntn   1/1     Running   0          3m4s
myapp-deploy-659f9975b8-nd66b   0/1     Running   0          3m4s
myapp-deploy-659f9975b8-p4j5k   0/1     Running   0          3m4s

$ kubectl get endpoints 
NAME                ENDPOINTS              AGE
service-clusterip   10.244.140.107:80      3m1s

$ ipvsadm -L -n
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn     
TCP  10.12.150.224:80 rr
  -> 10.244.140.107:80            Masq    1      0          0
```



**负载分发策略**

对Service的访问被分发到了后端的Pod上去，目前kubernetes提供了两种负载分发策略：

- 如果不定义，默认使用kube-proxy的策略，比如随机、轮询

- 基于客户端地址的会话保持模式，即来自同一个客户端发起的所有请求都会转发到固定的一个Pod上

  此模式可以使在spec中添加`sessionAffinity: ClientIP`选项

```bash
$ kubectl edit svc service-clusterip
sessionAffinity: ClientIP

$ while true;do curl 10.13.125.29/hostname.html; done
myapp-deploy-57bff895d5-fbln4
myapp-deploy-57bff895d5-fbln4
myapp-deploy-57bff895d5-fbln4
myapp-deploy-57bff895d5-fbln4
myapp-deploy-57bff895d5-fbln4
myapp-deploy-57bff895d5-fbln4
myapp-deploy-57bff895d5-fbln4
myapp-deploy-57bff895d5-fbln4

$ ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn       
TCP  10.13.125.29:80 rr persistent 10800
  -> 10.244.140.106:80            Masq    1      0          155       
  -> 10.244.196.141:80            Masq    1      0          0         
  -> 10.244.196.142:80            Masq    1      0          0 
```



### HeadLess类型的Service

在某些场景中，开发人员可能不想使用Service提供的负载均衡功能，而希望自己来控制负载均衡策略，针对这种情况，kubernetes提供了HeadLiness Service，这类Service不会分配Cluster IP，如果想要访问service，只能通过service的域名进行查询。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: service-headliness
spec:
  selector:
    app: myapp-deploy
  clusterIP: None # 将clusterIP设置为None，即可创建headliness Service
  type: ClusterIP
  ports:
  - port: 80    
    targetPort: 80
```



```bash
$ kubectl get svc -o wide
NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE   SELECTOR
service-headliness   ClusterIP   None            <none>        80/TCP    40s   app=myapp-deploy

$ kubectl describe svc service-headliness
Name:              service-headliness
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=myapp-deploy
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                None
IPs:               None
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.244.140.107:80
Session Affinity:  None
Events:            <none>

$ kubectl exec -it myapp-deploy-659f9975b8-2sntn -- /bin/bash
root@myapp-deploy-659f9975b8-2sntn:/# cat /etc/resolv.conf 
nameserver 10.0.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5

$ dig @10.0.0.10 service-headliness.default.svc.cluster.local
;; ANSWER SECTION:
service-headliness.default.svc.cluster.local. 30 IN A 10.244.140.107
service-headliness.default.svc.cluster.local. 30 IN A 10.244.196.145
service-headliness.default.svc.cluster.local. 30 IN A 10.244.196.144
```



### NodePort类型的Service

使用NAT在集群中每个选定Node的相同端口上公开service，使用nodeip:nodeport从集群外部访问service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: service-nodeport
spec:
  selector:
    app: myapp-deploy
  type: NodePort # service类型
  ports:
  - port: 80
    nodePort: 30002 # 指定绑定的node的端口(默认的取值范围是：30000-32767), 如果不指定，会默认分配
    targetPort: 80
```

查看是否能够正常的访问

```bash
$ for i in {1..6};do curl 192.168.173.100:30002/hostname.html;done
myapp-deploy-659f9975b8-nd66b
myapp-deploy-659f9975b8-p4j5k
myapp-deploy-659f9975b8-2sntn
myapp-deploy-659f9975b8-nd66b
myapp-deploy-659f9975b8-p4j5k
myapp-deploy-659f9975b8-2sntn
```



### LoadBalancer类型的Service

创建一个外部负载均衡器，并为service分配一个固定的外部ip



### ExternalName类型的Service

将service映射到externalname字段的内容如foo.bar.example.com，通过返回带有该名称的cname记录实现，不设置任何类型的代理

```yaml
apiVersion: v1
kind: Service
metadata:
  name: service-externalname
  namespace: dev
spec:
  type: ExternalName # service类型
  externalName: www.baidu.com  #改成ip地址也可以
```

```bash
$ kubectl get svc
NAME                   TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)        AGE
service-externalname   ExternalName   <none>         www.baidu.com   <none>         7s

$ dig @10.0.0.10 service-externalname.default.svc.cluster.local
;; ANSWER SECTION:
service-externalname.default.svc.cluster.local. 30 IN CNAME www.baidu.com.
www.baidu.com.          30      IN      CNAME   www.a.shifen.com.
www.a.shifen.com.       30      IN      A       180.101.50.242
www.a.shifen.com.       30      IN      A       180.101.50.188
```



## Ingress介绍

在前面课程中已经提到，Service对集群之外暴露服务的主要方式有两种：NotePort和LoadBalancer，但是这两种方式，都有一定的缺点：

- NodePort方式的缺点是会占用很多集群机器的端口，那么当集群服务变多的时候，这个缺点就愈发明显
- LB方式的缺点是每个service需要一个LB，浪费、麻烦，并且需要kubernetes之外设备的支持

基于这种现状，kubernetes提供了Ingress资源对象，Ingress只需要一个NodePort或者一个LB就可以满足暴露多个Service的需求。工作机制大致如下图表示：

实际上，Ingress相当于一个7层的负载均衡器，是kubernetes对反向代理的一个抽象，它的工作原理类似于Nginx，可以理解成在**Ingress里建立诸多映射规则，Ingress Controller通过监听这些配置规则并转化成Nginx的反向代理配置 , 然后对外部提供服务**。在这里有两个核心概念：

- ingress：kubernetes中的一个对象，作用是定义请求如何转发到service的规则
- ingress controller：具体实现反向代理及负载均衡的程序，对ingress定义的规则进行解析，根据配置的规则来实现请求转发，实现方式有很多，比如Nginx, Contour, Haproxy等等

Ingress（以Nginx为例）的工作原理如下：

1. 用户编写Ingress规则，说明哪个域名对应kubernetes集群中的哪个Service
2. Ingress控制器动态感知Ingress服务规则的变化，然后生成一段对应的Nginx反向代理配置
3. Ingress控制器会将生成的Nginx配置写入到一个运行着的Nginx服务中，并动态更新
4. 到此为止，其实真正在工作的就是一个Nginx了，内部配置了用户定义的请求转发规则



安装helm

```bash
# 安装helm，helm在kubernetes中相当于yum，是可以在线去获取资源清单，快速部署服务
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh

# 初始化,可以从 https://artifacthub.io/ 中选择一个可用的仓库地址
$ helm repo add bitnami https://charts.bitnami.com/bitnami
$ helm repo list
NAME    URL                               
bitnami https://charts.bitnami.com/bitnami

# 常见操作
$ helm repo update				# 更新chart列表
$ helm show chart bitnami/apache	# 查看chart基本信息
$ helm install bitnami/apache --generate-name		# 部署chart
$ helm list				# 查看部署包，加上--all可以看到所有的
$ helm uninstall apache-1726297430		# 删除这个安装包所有的kubernetes资源

$ helm search hub wordpress		# 在 helm hub(https://hub.helm.sh)上搜索helm chart
$ helm search repo wordpress	# 在repo中搜索
```



### 安装Ingress-nginx

```bash
$ helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
$ helm pull ingress-nginx/ingress-nginx

# 修改 values.yaml 文件
修改 hostNetwork 的值为 true
dnsPolicy的值改为: ClusterFirstWithHostNet
kind类型更改为：DaemonSet
ingressClassResource.default:true
      
# 关闭所有镜像的 digest   

# 如果是本地的helm chart，使用这个命令安装
$ kubectl create ns ingress
$ helm install ingress-nginx -n ingress . -f values.yaml
$ kubectl get pod -n ingress
NAME                             READY   STATUS    RESTARTS   AGE
ingress-nginx-controller-7c4x8   1/1     Running   0          12s
ingress-nginx-controller-bjk4s   1/1     Running   0          12s
```



### 实验测试

创建如下两个资源模型

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: aaronxudocker/myapp:v1.0
        ports:
        - containerPort: 80

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tomcat-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: tomcat
  template:
    metadata:
      labels:
        app: tomcat
    spec:
      containers:
      - name: tomcat
        image: tomcat:8.5-jre10-slim
        ports:
        - containerPort: 8080

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  clusterIP: None
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: tomcat-service
spec:
  selector:
    app: tomcat
  clusterIP: None
  type: ClusterIP
  ports:
  - port: 8080
    targetPort: 8080
```



#### Http代理

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-nginx
spec:
  rules:
  - host: nginx.iproute.cn
    http:
      paths:
      - path: /
        pathType: Prefix
        backend: 
          service:
            name: nginx-service
            port: 
              number: 80
  ingressClassName: nginx

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-tomcat
spec:
  rules:
  - host: tomcat.iproute.cn
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: tomcat-service
            port: 
              number: 8080
  ingressClassName: nginx
```



查看运行状态

```bash
$ kubectl get ing
NAME             CLASS   HOSTS               ADDRESS   PORTS   AGE
ingress-nginx    nginx   nginx.iproute.cn              80      7s
ingress-tomcat   nginx   tomcat.iproute.cn             80      7s

$ kubectl describe ing

Rules:
  Host              Path  Backends
  ----              ----  --------
  nginx.iproute.cn  
                    /   nginx-service:80 (10.244.140.109:80,10.244.196.149:80,10.244.196.150:80)

Rules:
  Host               Path  Backends
  ----               ----  --------
  tomcat.iproute.cn  
                     /   tomcat-service:8080 (10.244.140.110:8080,10.244.196.151:8080,10.244.196.153:8080)
```

访问测试，其中nginx多次访问主机名，可以看到负载均衡



### Https代理

创建证书

```bash
# 生成证书
$ openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/C=CN/ST=BJ/L=BJ/O=nginx/CN=iproute.cn"

# 创建密钥
$ kubectl create secret tls tls-secret --key tls.key --cert tls.crt
```



创建资源清单

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: https-nginx
spec:
  tls:
    - hosts:
      - nginx.iproute.cn
      secretName: tls-secret # 指定秘钥
  rules:
  - host: nginx.iproute.cn
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port: 
              number: 80
  ingressClassName: nginx

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tomcat-https
spec:
  tls:
    - hosts:
      - tomcat.iproute.cn
      secretName: tls-secret # 指定秘钥
  rules:
  - host: tomcat.iproute.cn
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: tomcat-service
            port: 
              number: 8080
  ingressClassName: nginx
```

访问测试，可以看到负载均衡



# 八、数据存储

现在已知，容器的生命周期可能很短，会被频繁地创建和销毁。那么容器在销毁时，保存在容器中的数据也会被清除。这种结果对用户来说，在某些情况下是不乐意看到的。为了持久化保存容器的数据，kubernetes引入了Volume的概念。

Volume是Pod中能够被多个容器访问的共享目录，它被定义在Pod上，然后被一个Pod里的多个容器挂载到具体的文件目录下，kubernetes通过Volume实现同一个Pod中不同容器之间的数据共享以及数据的持久化存储。Volume的生命容器不与Pod中单个容器的生命周期相关，当容器终止或者重启时，Volume中的数据也不会丢失。

kubernetes的Volume支持多种类型，比较常见的有下面几个：

- 简单存储：EmptyDir、HostPath、NFS
- 高级存储：PV、PVC
- 配置存储：ConfigMap、Secret



## 基本存储

### EmptyDir

EmptyDir是最基础的Volume类型，**一个EmptyDir就是Host上的一个空目录**。

EmptyDir是在Pod被分配到Node时创建的，它的初始内容为空，并且无须指定宿主机上对应的目录文件，因为kubernetes会自动分配一个目录，当Pod销毁时， EmptyDir中的数据也会被永久删除。 EmptyDir用途如下：

- 临时空间，例如用于某些应用程序运行时所需的临时目录，且无须永久保留
- 一个容器需要从另一个容器中获取数据的目录（多容器共享目录）

接下来，通过一个容器之间文件共享的案例来使用一下EmptyDir。

在一个Pod中准备两个容器nginx和busybox，然后声明一个Volume分别挂在到两个容器的目录中，然后nginx容器负责向Volume中写日志，busybox中通过命令将日志内容读到控制台。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-emptydir
spec:
  containers:
  - name: nginx
    image: aaronxudocker/myapp:v1.0
    ports:
    - containerPort: 80
    volumeMounts:  # 将logs-volume挂在到nginx容器中，对应的目录为 /var/log/nginx
    - name: logs-volume
      mountPath: /var/log/nginx
  - name: busybox
    image: aaronxudocker/tools:busybox
    command: ["/bin/sh","-c","tail -f /logs/access.log"] # 初始命令，动态读取指定文件中内容
    volumeMounts:  # 将logs-volume 挂在到busybox容器中，对应的目录为 /logs
    - name: logs-volume
      mountPath: /logs
  volumes: # 声明volume， name为logs-volume，类型为emptyDir
  - name: logs-volume
    emptyDir: {}
```



查看运行结果

```bash
$ kubectl get pod -o wide
NAME              READY   STATUS    RESTARTS   AGE    IP               NODE     NOMINATED NODE   READINESS GATES
volume-emptydir   2/2     Running   0          102s   10.244.196.158   node01   <none>           <none>
$ curl 10.244.196.158/hostname.html
volume-emptydir
$ kubectl logs -f volume-emptydir -c busybox
192.168.173.100 - - [14/Sep/2024:08:10:03 +0000] "GET /hostname.html HTTP/1.1" 200 16 "-" "curl/7.76.1" "-"
```



### HostPath

EmptyDir中数据不会被持久化，它会随着Pod的结束而销毁，如果**想简单的将数据持久化到主机**中，可以选择HostPath。

HostPath就是将Node主机中一个实际目录挂在到Pod中，以供容器使用，这样的设计就可以保证Pod销毁了，但是数据依据可以存在于Node主机上。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-hostpath
spec:
  containers:
  - name: nginx
    image: aaronxudocker/myapp:v1.0
    ports:
    - containerPort: 80
    volumeMounts:  # 将logs-volume挂在到nginx容器中，对应的目录为 /var/log/nginx
    - name: logs-volume
      mountPath: /var/log/nginx
  - name: busybox
    image: aaronxudocker/tools:busybox
    command: ["/bin/sh","-c","tail -f /logs/access.log"] # 初始命令，动态读取指定文件中内容
    volumeMounts:  # 将logs-volume 挂在到busybox容器中，对应的目录为 /logs
    - name: logs-volume
      mountPath: /logs
  volumes: # 声明volume， name为logs-volume，类型为emptyDir
  - name: logs-volume
    hostPath: 
      path: /root/logs
      type: DirectoryOrCreate  # 目录存在就使用，不存在就先创建后使用
```

关于type的值的一点说明：

- DirectoryOrCreate 目录存在就使用，不存在就先创建后使用
- Directory 目录必须存在
- FileOrCreate 文件存在就使用，不存在就先创建后使用
- File 文件必须存在
- Socket unix套接字必须存在
- CharDevice 字符设备必须存在
- BlockDevice 块设备必须存在

测试一下

```bash
$ kubectl get pod -o wide
NAME              READY   STATUS    RESTARTS   AGE   IP               NODE     NOMINATED NODE   READINESS GATES
volume-hostpath   2/2     Running   0          55s   10.244.196.157   node01   <none>           <none>

$ curl 10.244.196.157/hostname.html
volume-hostpath

$ ssh root@node01
root@node01's password: 
Last login: Sat Sep 14 16:11:40 2024 from 192.168.173.100
$ cat /root/logs/access.log 
192.168.173.100 - - [14/Sep/2024:08:18:50 +0000] "GET /hostname.html HTTP/1.1" 200 16 "-" "curl/7.76.1" "-"
192.168.173.100 - - [14/Sep/2024:08:23:01 +0000] "GET /hostname.html HTTP/1.1" 200 16 "-" "curl/7.76.1" "-"
```



### NFS

HostPath可以解决数据持久化的问题，但是一旦Node节点故障了，Pod如果转移到了别的节点，又会出现问题了，此时需要准备单独的网络存储系统，**比较常用的用NFS、CIFS**。

NFS是一个网络文件存储系统，可以搭建一台NFS服务器，然后将Pod中的存储直接连接到NFS系统上，这样的话，无论Pod在节点上怎么转移，只要Node跟NFS的对接没问题，数据就可以成功访问。

```bash
# 安装nfs服务
$ yum install nfs-utils -y

# 创建一个共享文件夹
$ mkdir -pv /root/data/nfs
mkdir: created directory '/root/data'
mkdir: created directory '/root/data/nfs'

# 将共享文件夹读写权限暴露给192.168.173.0/24网段中的所有主机
$ vim /etc/exports
/root/data/nfs  192.168.173.0/24(rw,no_root_squash)

# 启动nfs服务
$ systemctl restart nfs-server
```

- 接下来，要在的每个node节点上都安装下nfs，这样的目的是为了node节点可以驱动nfs设备

```bash
# 在node上安装nfs服务，注意不需要启动
$ yum install nfs-utils -y
```

- 接下来，就可以编写pod的配置文件了

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-nfs
spec:
  containers:
  - name: nginx
    image: aaronxudocker/myapp:v1.0
    ports:
    - containerPort: 80
    volumeMounts:
    - name: logs-volume
      mountPath: /var/log/nginx
  - name: busybox
    image: aaronxudocker/tools:busybox
    command: ["/bin/sh","-c","tail -f /logs/access.log"] 
    volumeMounts:
    - name: logs-volume
      mountPath: /logs
  volumes:
  - name: logs-volume
    nfs:
      server: 192.168.173.100  #nfs服务器地址
      path: /root/data/nfs #共享文件路径
```

- 查看一下效果，可以看到文件已经写入到nfs中了

```bash
$ ls /root/data/nfs/
access.log  error.log
```



## 高级存储

前面已经学习了使用NFS提供存储，此时就要求用户会搭建NFS系统，并且会在yaml配置nfs。由于kubernetes支持的存储系统有很多，要求客户全都掌握，显然不现实。为了能够屏蔽底层存储实现的细节，方便用户使用， kubernetes引入PV和PVC两种资源对象。

PV（Persistent Volume）是持久化卷的意思，是对底层的共享存储的一种抽象。一般情况下PV由kubernetes管理员进行创建和配置，它与底层具体的共享存储技术有关，并通过插件完成与共享存储的对接。

PVC（Persistent Volume Claim）是持久卷声明的意思，是用户对于存储需求的一种声明。换句话说，PVC其实就是用户向kubernetes系统发出的一种资源需求申请。

使用了PV和PVC之后，工作可以得到进一步的细分：

- 存储：存储工程师维护
- PV： kubernetes管理员维护
- PVC：kubernetes用户维护



### PV

PV是存储资源的抽象，下面是资源清单文件:

```yaml
apiVersion: v1  
kind: PersistentVolume
metadata:
  name: pv2
spec:
  nfs: # 存储类型，与底层真正存储对应
  capacity:  # 存储能力，目前只支持存储空间的设置
    storage: 2Gi
  accessModes:  # 访问模式
  storageClassName: # 存储类别
  persistentVolumeReclaimPolicy: # 回收策略
```

PV 的关键配置参数说明：

- **存储类型**

  - 底层实际存储的类型，kubernetes支持多种存储类型，每种存储类型的配置都有所差异

- **存储能力（capacity）**

  - 目前只支持存储空间的设置( storage=1Gi )，不过未来可能会加入IOPS、吞吐量等指标的配置

- **访问模式（accessModes）**

  - 用于描述用户应用对存储资源的访问权限，访问权限包括下面几种方式：
    - ReadWriteOnce（RWO）：读写权限，但是只能被单个节点挂载
    - ReadOnlyMany（ROX）： 只读权限，可以被多个节点挂载
    - ReadWriteMany（RWX）：读写权限，可以被多个节点挂载
    - ReadWriteOncePod(RWOP)：卷只能由单个 Pod 以读写方式挂载，kubernetes版本1.22+才支持

  需要注意的是，底层不同的存储类型可能支持的访问模式不同

- **回收策略（persistentVolumeReclaimPolicy）**

  当PV不再被使用了之后，对其的处理方式。目前支持三种策略：

  - Retain （保留） **保留数据**，需要管理员手工清理数据
  - Recycle（回收） **清除 PV 中的数据**，效果相当于执行 rm -rf /thevolume/*
  - Delete （删除） 与 PV 相连的后端存储完成 volume 的删除操作，当然这常见于云服务商的存储服务

  需要注意的是，底层不同的存储类型可能支持的回收策略不同

- **存储类别**

  PV可以通过storageClassName参数指定一个存储类别

  - 具有特定类别的PV只能与请求了该类别的PVC进行绑定
  - 未设定类别的PV则只能与不请求任何类别的PVC进行绑定

- **状态（status）**

  一个 PV 的生命周期中，可能会处于4中不同的阶段：

  - Available（可用）： 表示可用状态，还未被任何 PVC 绑定
  - Bound（已绑定）： 表示 PV 已经被 PVC 绑定
  - Released（已释放）： 表示 PVC 被删除，但是资源还未被集群重新声明
  - Failed（失败）： 表示该 PV 的自动回收失败



#### 实验

使用NFS作为存储，来演示PV的使用，创建3个PV，对应NFS中的3个暴露的路径。

1) 准备NFS环境

```bash
# 创建目录
$ mkdir /root/data/{pv1,pv2,pv3} -pv
mkdir: created directory '/root/data/pv1'
mkdir: created directory '/root/data/pv2'
mkdir: created directory '/root/data/pv3'

# 暴露服务
$ vim /etc/exports
/root/data/pv1  192.168.173.0/24(rw,no_root_squash)
/root/data/pv2  192.168.173.0/24(rw,no_root_squash)
/root/data/pv3  192.168.173.0/24(rw,no_root_squash)

# 重启服务
$ systemctl restart nfs-server
```

2) 创建pv

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name:  pv1
spec:
  capacity: 
    storage: 1Gi
  accessModes:
  - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /root/data/pv1
    server: 192.168.173.100

---
apiVersion: v1
kind: PersistentVolume
metadata:
  name:  pv2
spec:
  capacity: 
    storage: 2Gi
  accessModes:
  - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /root/data/pv2
    server: 192.168.173.100
    
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name:  pv3
spec:
  capacity: 
    storage: 3Gi
  accessModes:
  - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /root/data/pv3
    server: 192.168.173.100
```

```bash
# 创建 pv
$ kubectl create -f pv.yaml
persistentvolume/pv1 created
persistentvolume/pv2 created
persistentvolume/pv3 created

# 查看pv
$ kubectl get pv
NAME   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
pv1    1Gi        RWX            Retain           Available          <unset>           32s
pv2    2Gi        RWX            Retain           Available          <unset>           32s
pv3    3Gi        RWX            Retain           Available          <unset>           32s
```



### PVC

PVC是资源的申请，用来声明对存储空间、访问模式、存储类别需求信息。下面是资源清单文件:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc
spec:
  accessModes: # 访问模式
  selector: # 采用标签对PV选择
  storageClassName: # 存储类别
  resources: # 请求空间
    requests:
      storage: 5Gi
```

PVC 的关键配置参数说明：

- **访问模式（accessModes）**
  - 用于描述用户应用对存储资源的访问权限
- **选择条件（selector）**
  - 通过Label Selector的设置，可使PVC对于系统中己存在的PV进行筛选
- **存储类别（storageClassName）**
  - PVC在定义时可以设定需要的后端存储的类别，只有设置了该class的pv才能被系统选出
- **资源请求（Resources ）**
  - 描述对存储资源的请求



#### 实验

1) 创建pvc申请pv

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc1
spec:
  accessModes: 
  - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc2
spec:
  accessModes: 
  - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc3
spec:
  accessModes: 
  - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
```



```bash
# 创建pvc
$ kubectl create -f pvc.yaml
persistentvolumeclaim/pvc1 created
persistentvolumeclaim/pvc2 created
persistentvolumeclaim/pvc3 created

# 查看pvc
$ kubectl get pvc -o wide
NAME   STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE   VOLUMEMODE
pvc1   Bound    pv1      1Gi        RWX                           <unset>                 26s   Filesystem
pvc2   Bound    pv2      2Gi        RWX                           <unset>                 26s   Filesystem
pvc3   Bound    pv3      3Gi        RWX                           <unset>                 26s   Filesystem

# 查看pv
$ kubectl get pv -o wide
NAME   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM          STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE   VOLUMEMODE
pv1    1Gi        RWX            Retain           Bound    default/pvc1                  <unset>                          6m    Filesystem
pv2    2Gi        RWX            Retain           Bound    default/pvc2                  <unset>                          6m    Filesystem
pv3    3Gi        RWX            Retain           Bound    default/pvc3                  <unset>                          6m    Filesystem
```



2) 创建pod, 使用pv

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  containers:
  - name: busybox
    image: aaronxudocker/tools:busybox
    command: ["/bin/sh","-c","while true;do echo pod1 >> /root/out.txt; sleep 10; done;"]
    volumeMounts:
    - name: volume
      mountPath: /root/
  volumes:
    - name: volume
      persistentVolumeClaim:
        claimName: pvc1
        readOnly: false
---
apiVersion: v1
kind: Pod
metadata:
  name: pod2
spec:
  containers:
  - name: busybox
    image: aaronxudocker/tools:busybox
    command: ["/bin/sh","-c","while true;do echo pod2 >> /root/out.txt; sleep 10; done;"]
    volumeMounts:
    - name: volume
      mountPath: /root/
  volumes:
    - name: volume
      persistentVolumeClaim:
        claimName: pvc2
        readOnly: false
```



```bash
# 查看nfs中的文件存储
$ more /root/data/pv1/out.txt 
pod1
pod1
pod1

$ more /root/data/pv2/out.txt 
pod2
pod2
pod2
```



#### 生命周期

PVC和PV是一一对应的，PV和PVC之间的相互作用遵循以下生命周期：

- **资源供应**：管理员手动创建底层存储和PV
- **资源绑定**：用户创建PVC，kubernetes负责根据PVC的声明去寻找PV，并绑定
  - 在用户定义好PVC之后，系统将根据PVC对存储资源的请求在已存在的PV中选择一个满足条件的
  - 一旦找到，就将该PV与用户定义的PVC进行绑定，用户的应用就可以使用这个PVC了
  - 如果找不到，PVC则会无限期处于Pending状态，直到等到系统管理员创建了一个符合其要求的PV
  - PV一旦绑定到某个PVC上，就会被这个PVC独占，不能再与其他PVC进行绑定了
- **资源使用**：用户可在pod中像volume一样使用pvc
  - Pod使用Volume的定义，将PVC挂载到容器内的某个路径进行使用。
- **资源释放**：用户删除pvc来释放pv
  - 当存储资源使用完毕后，用户可以删除PVC，与该PVC绑定的PV将会被标记为“已释放”，但还不能立刻与其他PVC进行绑定。通过之前PVC写入的数据可能还被留在存储设备上，只有在清除之后该PV才能再次使用。
- **资源回收**：kubernetes根据pv设置的回收策略进行资源的回收
  - 对于PV，管理员可以设定回收策略，用于设置与之绑定的PVC释放资源之后如何处理遗留数据的问题。只有PV的存储空间完成回收，才能供新的PVC绑定和使用



## 配置存储

### ConfigMap

ConfigMap功能在Kubernetes1.2版本中引入，许多应⽤程序会从配置文件、命令⾏参数或环境变量中读取配置信息。ConfigMapAPI给我们提供了向容器中注入配置信息的机制，ConfigMap可以被⽤来保存单个属性，也可以⽤来保存整个配置文件或者JSON⼆进制等对象

```bash
# 创建configMag
$ kubectl create configmap myconfig --from-file=myconfig.conf

# myconfig.conf文件内容
username=admin
password=12345

# 通过选项的方式传递内容
$ kubectl create configmap myconfig myconfig --from-literal=username=admin --from-literal=password=12345
```

ConfigMap在实际使用中最常见的用法是：

- 注入环境变量
- 注入配置文件



#### 环境变量

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: literal-config
  namespace: default
data:
  name: admin
  password: "123456"

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: env-config
  namespace: default
data:
  log_level: INFO

---
apiVersion: v1
kind: Pod
metadata:
  name: cm-env-pod
spec:
  containers:
    - name: myapp-container
      image: aaronxudocker/myapp:v1.0
      command: [ "/bin/sh", "-c", "env" ]
      env:
        - name: USERNAME
          valueFrom:
            configMapKeyRef:
              name: literal-config
              key: name
        - name: PASSWORD
          valueFrom:
            configMapKeyRef:
              name: literal-config
              key: password
      envFrom:
        - configMapRef:
            name: env-config
  restartPolicy: Never
```

查看环境变量

```bash
$ kubectl logs cm-env-pod
USERNAME=admin
log_level=INFO
PASSWORD=123456
```



#### 文件挂载

将ConfigMap当做文件挂载到pod中

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: literal-config
  namespace: default
data:
  name: admin
  password: "123456"

---
apiVersion: v1
kind: Pod
metadata:
  name: cm-volume-pod
spec:
  containers:
    - name: myapp-container
      image: aaronxudocker/myapp:v1.0
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: literal-config
  restartPolicy: Never
```

```bash
$ kubectl exec -it cm-volume-pod -c myapp-container -- /bin/bash
root@cm-volume-pod:/# ls /etc/config/
name  password
root@cm-volume-pod:/# ls -l /etc/config
total 0
lrwxrwxrwx 1 root root 11 Sep 16 05:35 name -> ..data/name
lrwxrwxrwx 1 root root 15 Sep 16 05:35 password -> ..data/password
```



#### 热更新

编写一个nginx的配置文件

```bash
$ cat default.conf 
server {
    listen 80 default_server;
    server_name example.com www.example.com;
    location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
     }
}
```

创建configmap

```bash
$ kubectl create cm default-nginx --from-file=default.conf

$ kubectl describe cm default-nginx
Name:         default-nginx
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
default.conf:
----
server {
    listen 80 default_server;
    server_name example.com www.example.com;
    location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
     }
}


BinaryData
====

Events:  <none>
```

创建nginx的deployment控制器

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: hotupdate-deploy
  name: hotupdate-deploy
spec:
  replicas: 5
  selector:
    matchLabels:
      app: hotupdate-deploy
  template:
    metadata:
      labels:
        app: hotupdate-deploy
    spec:
      containers:
      - image: aaronxudocker/myapp:v1.0
        name: nginx
        volumeMounts:
        - name: config-volume
          mountPath: /etc/nginx/conf.d/
      volumes:
        - name: config-volume
          configMap:
            name: default-nginx
```

创建好之后，检查配置文件

```bash
$ kubectl exec -it hotupdate-deploy-b548444d4-54r88 -- /bin/bash
root@hotupdate-deploy-b548444d4-54r88:/# cat /etc/nginx/conf.d/default.conf 
server {
    listen 80 default_server;
    server_name example.com www.example.com;
    location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
     }
}
```

修改configmap中的端口号配置

```bash
# 修改nginx端口号为8080
$ kubectl edit cm default-nginx
```

检查修改后的情况

```bash
$ kubectl exec -it hotupdate-deploy-b548444d4-54r88 -- /bin/bash
root@hotupdate-deploy-b548444d4-54r88:/# cat /etc/nginx/conf.d/default.conf
server {
    listen 8080 default_server;
    server_name example.com www.example.com;
    location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
     }
}

# 访问8080端口，发现并未生效
$ curl 10.244.196.166
Myapp Version 1.0

$ curl 10.244.196.166:8080
curl: (7) Failed to connect to 10.244.196.166 port 8080: Connection refused
```

可以通过修改pod annotations的⽅式强制触发滚动更新

```bash
$ kubectl patch deployment hotupdate-deploy --patch '{"spec": {"template": {"metadata": {"annotations": {"version/config": "v1.0" }}}}}'

$ kubectl get pod
NAME                                READY   STATUS        RESTARTS   AGE
hotupdate-deploy-7995cdc7dc-2bqvp   1/1     Running       0          7s
hotupdate-deploy-7995cdc7dc-7s5vt   1/1     Running       0          6s
hotupdate-deploy-7995cdc7dc-b5mpx   1/1     Running       0          8s
hotupdate-deploy-7995cdc7dc-bwr5q   1/1     Running       0          8s
hotupdate-deploy-7995cdc7dc-xlvtc   1/1     Running       0          8s
hotupdate-deploy-b548444d4-54r88    1/1     Terminating   0          5m19s
hotupdate-deploy-b548444d4-kgnk7    1/1     Terminating   0          5m19s
hotupdate-deploy-b548444d4-pg2jm    1/1     Terminating   0          5m19s
hotupdate-deploy-b548444d4-tlm8d    1/1     Terminating   0          5m19s
hotupdate-deploy-b548444d4-zbrq4    1/1     Terminating   0          5m19s

$ curl 10.244.140.115:8080
Myapp Version 1.0
```

更新ConfigMap后：

- 使⽤该ConfigMap挂载的Env不会同步更新
- 使⽤该ConfigMap挂载的Volume中的数据需要⼀段时间（实测⼤概10秒）才能同步更新



#### 不可变configmap

Kubernetes给不可变的ConfigMap和Secret提供了⼀种可选配置，可以设置各个Secret和ConfigMap为不可变的。对于⼤量使⽤configmap的集群（⾄少有成千上万各不相同的configmap供Pod挂载），禁⽌变更它们的数据有下列好处：

- 防⽌意外（或非预期的）更新导致应⽤程序中断
- 通过将configmap标记为不可变来关闭kube-apiserver对其的监视，从⽽显著降低kube-apiserver的负载，提升集群性能。

```bash
$ kubectl explain cm
KIND:       ConfigMap
VERSION:    v1

DESCRIPTION:
    ConfigMap holds configuration data for pods to consume.
    
FIELDS:
  immutable     <boolean>
    Immutable, if set to true, ensures that data stored in the ConfigMap cannot
    be updated (only object metadata can be modified). If not set to true, the
    field can be modified at any time. Defaulted to nil.
```



### Secret

Secret对象类型⽤来保存敏感信息，例如密码、OAuth令牌和SSH密钥。将这些信息放在secret中比放在Pod的定义或者容器镜像中来说更加安全和灵活。

- Kubernetes通过仅仅将Secret分发到需要访问Secret的Pod所在的机器节点来保障其安全性
- Secret只会存储在节点的内存中，永不写入物理存储，这样从节点删除secret时就不需要擦除磁盘数据
- 从Kubernetes1.7版本开始，etcd会以加密形式存储Secret，⼀定程度的保证了Secret安全性
- Secret的内置类型有如下表

| 内置类型                            | 用法                                  |
| :---------------------------------- | :------------------------------------ |
| Opaque                              | 用户定义的任意数据                    |
| kubernetes.io/service-account-token | 服务账号令牌                          |
| kubernetes.io/dockercfg             | ~/.dockercfg文件的序列化形式          |
| kubernetes.io/dockerconfigjson      | ~/.docker/config.json文件的序列化形式 |
| kubernetes.io/basic-auth            | 用于基本身份认证的凭据                |
| kubernetes.io/ssh-auth              | 用于SSH身份认证的凭据                 |
| kubernetes.io/tls                   | 用于TLS客户端或者服务器端的数据       |
| bootstrap.kubernetes.io/token       | 启动引导令牌数据                      |

当Secret配置文件中未作显式设定时，默认的Secret类型是Opaque。当你使⽤kubectl来创建⼀个Secret时，你会使⽤generic⼦命令来标明要创建的是⼀个Opaque类型Secret。

```bash
# 存入secret中的数据需要base64编码
$ echo -n "admin" | base64
YWRtaW4=
$ echo -n "123456" | base64
MTIzNDU2

# base64可以轻易的被解码
$ echo -n "YWRtaW4=" | base64 -d
admin
```

创建一个secret类型的资源

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  password: MTIzNDU2
  username: YWRtaW4=
```

查看资源创建

```bash
$ kubectl get secret
NAME         TYPE                DATA   AGE
mysecret     Opaque              2      22s

$ kubectl describe secret mysecret
Name:         mysecret
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password:  6 bytes
username:  5 bytes

$ kubectl get secret mysecret -o yaml
apiVersion: v1
data:
  password: MTIzNDU2
  username: YWRtaW4=
kind: Secret
metadata:
  creationTimestamp: "2024-09-16T06:05:59Z"
  name: mysecret
  namespace: default
  resourceVersion: "676071"
  uid: 30896725-6713-491f-b779-ce75f7e6312e
type: Opaque
```



#### 环境变量

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
 labels:
   app: opaque-secret-env
 name: opaque-secret-env-deploy
spec:
 replicas: 5
 selector:
   matchLabels:
     app: op-se-env-pod
 template:
   metadata:
     labels:
       app: op-se-env-pod
   spec:
     containers:
     - image: aaronxudocker/myapp:v1.0
       name: myapp-continaer
       ports:
       - containerPort: 80
       env:
       - name: TEST_USER
         valueFrom:
           secretKeyRef:
             name: mysecret
             key: username
       - name: TEST_PASSWORD
         valueFrom:
           secretKeyRef:
             name: mysecret
             key: password
```

验证环境

```bash
$ kubectl exec -it opaque-secret-env-deploy-6b5b66b9db-292zl -- /bin/bash
root@opaque-secret-env-deploy-6b5b66b9db-292zl:/# env
TEST_USER=admin
TEST_PASSWORD=123456
```



#### 文件挂载

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: secret-volume
  name: secret-volume-pod
spec:
  volumes:
  - name: volumes12
    secret:
      secretName: mysecret
  containers:
  - image: aaronxudocker/myapp:v1.0
    name: myapp-container
    volumeMounts:
    - name: volumes12
      mountPath: "/data"
```

验证结果

```bash
kubectl exec -it secret-volume-pod -- /bin/bash
root@secret-volume-pod:/# ls /data/
password  username
root@secret-volume-pod:/# ls -l /data/
total 0
lrwxrwxrwx 1 root root 15 Sep 16 06:17 password -> ..data/password
lrwxrwxrwx 1 root root 15 Sep 16 06:17 username -> ..data/username
root@secret-volume-pod:/# cat /data/password 
123456
```

可以指定挂载后文件的权限

```yaml
# 修改权限，256的八进制是400
  volumes:
  - name: volumes12
    secret:
      secretName: mysecret
      defaultMode: 256
```

验证权限

```bash
$ kubectl exec -it secret-volume-pod -- /bin/bash
root@secret-volume-pod:/data/..data# ls -l
total 8
-r-------- 1 root root 12 Sep 16 06:22 password
-r-------- 1 root root  5 Sep 16 06:22 username
```

指定只需要的键

```yaml
volumes:
- name: volumes12
  secret:
    secretName: mysecret
    items:
    - key: username
      path: my-group/my-username
```

验证

```bash
$ kubectl exec -it secret-volume-pod -- /bin/bash
root@secret-volume-pod:/# ls -l /data/
total 0
lrwxrwxrwx 1 root root 15 Sep 16 06:26 my-group -> ..data/my-group
```



# 九、调度器

在默认情况下，一个Pod在哪个Node节点上运行，是由Scheduler组件采用相应的算法计算出来的，这个过程是不受人工控制的。但是在实际使用中，这并不满足的需求，因为很多情况下，我们想控制某些Pod到达某些节点上，那么应该怎么做呢？这就要求了解kubernetes对Pod的调度规则，kubernetes提供了四大类调度方式：

- 自动调度：运行在哪个节点上完全由Scheduler经过一系列的算法计算得出
- 定向调度：NodeName、NodeSelector
- 亲和性调度：NodeAffinity、PodAffinity、PodAntiAffinity
- 污点（容忍）调度：Taints、Toleration



## 定向调度

定向调度，指的是利用在pod上声明nodeName或者nodeSelector，以此将Pod调度到期望的node节点上。注意，这里的调度是强制的，这就意味着即使要调度的目标Node不存在，也会向上面进行调度，只不过pod运行失败而已。



### **NodeName**

NodeName用于强制约束将Pod调度到指定的Name的Node节点上。这种方式，其实是直接跳过Scheduler的调度逻辑，直接将Pod调度到指定名称的节点。



实验：创建一个pod-nodename.yaml文件

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nodename
spec:
  containers:
  - name: nginx
    image: aaronxudocker/myapp:v1.0
  nodeName: node01 # 指定调度到nodem01节点上
```

```bash
#查看Pod调度到NODE属性，确实是调度到了node01节点上
$ kubectl get pods pod-nodename -o wide
NAME           READY   STATUS    RESTARTS   AGE   IP               NODE     NOMINATED NODE   READINESS GATES
pod-nodename   1/1     Running   0          62s   10.244.196.188   node01   <none>           <none> 

# 接下来，删除pod，修改nodeName的值为node03（并没有node03节点）
#再次查看，发现已经向Node03节点调度，但是由于不存在node03节点，所以pod无法正常运行
$ kubectl get pods pod-nodename -o wide
NAME           READY   STATUS    RESTARTS   AGE   IP       NODE     NOMINATED NODE   READINESS GATES
pod-nodename   0/1     Pending   0          10s   <none>   node03   <none>           <none>   
```



### **NodeSelector**

NodeSelector用于将pod调度到添加了指定标签的node节点上。它是通过kubernetes的label-selector机制实现的，也就是说，在pod创建之前，会由scheduler使用MatchNodeSelector调度策略进行label匹配，找出目标node，然后将pod调度到目标节点，该匹配规则是强制约束。

实验：

1 首先分别为node节点添加标签

```bash
$ kubectl label nodes node01 nodeenv=pro
node/node02 labeled
$ kubectl label nodes node02 nodeenv=test
node/node02 labeled
```

2 创建一个pod-nodeselector.yaml文件，并使用它创建Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nodeselector
spec:
  containers:
  - name: nginx
    image: aaronxudocker/myapp:v1.0
  nodeSelector: 
    nodeenv: pro # 指定调度到具有nodeenv=pro标签的节点上
```

```bash
#查看Pod调度到NODE属性，确实是调度到了node1节点上
$ kubectl get pod pod-nodeselector -o wide
NAME               READY   STATUS    RESTARTS   AGE   IP               NODE     NOMINATED NODE   READINESS GATES
pod-nodeselector   1/1     Running   0          14s   10.244.196.189   node01   <none>           <none>

# 接下来，删除pod，修改nodeSelector的值为nodeenv: xxxx（不存在打有此标签的节点）
#再次查看，发现pod无法正常运行,Node的值为none
$ kubectl get pod -o wide
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  36s   default-scheduler  0/3 nodes are available: 1 node(s) had untolerated taint {node-role.kubernetes.io/control-plane: }, 2 node(s) didn't match Pod's node affinity/selector. preemption: 0/3 nodes are available: 3 Preemption is not helpful for scheduling.
```



## 亲和性调度

在定向调度中，介绍了两种定向调度的方式，使用起来非常方便，但是也有一定的问题，那就是如果没有满足条件的Node，那么Pod将不会被运行，即使在集群中还有可用Node列表也不行，这就限制了它的使用场景。

基于上面的问题，kubernetes还提供了一种亲和性调度（Affinity）。它在**NodeSelector**的基础之上的进行了扩展，可以通过配置的形式，实现优先选择满足条件的Node进行调度，如果没有，也可以调度到不满足条件的节点上，使调度更加灵活。

Affinity主要分为三类：

- nodeAffinity(node亲和性）: 以node为目标，解决pod可以调度到哪些node的问题
- podAffinity(pod亲和性) : 以pod为目标，解决pod可以和哪些已存在的pod部署在同一个拓扑域中的问题
- podAntiAffinity(pod反亲和性) : 以pod为目标，解决pod不能和哪些已存在pod部署在同一个拓扑域中的问题

> 关于亲和性(反亲和性)使用场景的说明：
>
> **亲和性**：如果两个应用频繁交互，那就有必要利用亲和性让两个应用的尽可能的靠近，这样可以减少因网络通信而带来的性能损耗。
>
> **反亲和性**：当应用的采用多副本部署时，有必要采用反亲和性让各个应用实例打散分布在各个node上，这样可以提高服务的高可用性。



### **NodeAffinity**

首先来看一下`NodeAffinity`的可配置项：

```yaml
pod.spec.affinity.nodeAffinity
  requiredDuringSchedulingIgnoredDuringExecution  Node节点必须满足指定的所有规则才可以，相当于硬限制
    nodeSelectorTerms  节点选择列表
      matchFields   按节点字段列出的节点选择器要求列表
      matchExpressions   按节点标签列出的节点选择器要求列表(推荐)
        key    键
        values 值
        operator 关系符 支持Exists, DoesNotExist, In, NotIn, Gt, Lt
  preferredDuringSchedulingIgnoredDuringExecution 优先调度到满足指定的规则的Node，相当于软限制 (倾向)
    preference   一个节点选择器项，与相应的权重相关联
      matchFields   按节点字段列出的节点选择器要求列表
      matchExpressions   按节点标签列出的节点选择器要求列表(推荐)
        key    键
        values 值
        operator 关系符 支持In, NotIn, Exists, DoesNotExist, Gt, Lt
	weight 倾向权重，在范围1-100。
```

```bash
关系符的使用说明:

- matchExpressions:
  - key: nodeenv              # 匹配存在标签的key为nodeenv的节点
    operator: Exists
  - key: nodeenv              # 匹配标签的key为nodeenv,且value是"xxx"或"yyy"的节点
    operator: In
    values: ["xxx","yyy"]
  - key: nodeenv              # 匹配标签的key为nodeenv,且value大于"xxx"的节点
    operator: Gt
    values: "xxx"
```

首先演示`requiredDuringSchedulingIgnoredDuringExecution` ,

创建pod-nodeaffinity-required.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nodeaffinity-required
spec:
  containers:
  - name: nginx
    image: aaronxudocker/myapp:v1.0
  affinity:  #亲和性设置
    nodeAffinity: #设置node亲和性
      requiredDuringSchedulingIgnoredDuringExecution: # 硬限制
        nodeSelectorTerms:
        - matchExpressions: # 匹配nodeenv的值在["xxx","yyy"]中的标签
          - key: nodeenv
            operator: In
            values: ["xxx","yyy"]
```

```bash
# 查看pod状态 （运行失败）
$ kubectl get pod -o wide
NAME                        READY   STATUS    RESTARTS   AGE   IP       NODE     NOMINATED NODE   READINESS GATES
pod-nodeaffinity-required   0/1     Pending   0          6s    <none>   <none>   <none>           <none>

# 查看Pod的详情
# 发现调度失败，提示node选择失败
$ kubectl describe pod pod-nodeaffinity-required
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  42s   default-scheduler  0/3 nodes are available: 1 node(s) had untolerated taint {node-role.kubernetes.io/control-plane: }, 2 node(s) didn't match Pod's node affinity/selector. preemption: 0/3 nodes are available: 3 Preemption is not helpful for scheduling.

# 修改文件，将values: ["xxx","yyy"]------> ["pro","yyy"]
# 此时查看，发现调度成功，已经将pod调度到了node1上
$ kubectl get pod -o wide
NAME                        READY   STATUS    RESTARTS   AGE   IP               NODE     NOMINATED NODE   READINESS GATES
pod-nodeaffinity-required   1/1     Running   0          7s    10.244.196.187   node01   <none>           <none>
```

接下来再演示一下`preferredDuringSchedulingIgnoredDuringExecution` ,

创建pod-nodeaffinity-preferred.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nodeaffinity-preferred
spec:
  containers:
  - name: nginx
    image: aaronxudocker/myapp:v1.0
  affinity:  #亲和性设置
    nodeAffinity: #设置node亲和性
      preferredDuringSchedulingIgnoredDuringExecution: # 软限制
      - weight: 1
        preference:
          matchExpressions: # 匹配env的值在["xxx","yyy"]中的标签(当前环境没有)
          - key: nodeenv
            operator: In
            values: ["xxx","yyy"]
```

```bash
# 创建pod
[root@k8s-master01 ~]# kubectl create -f pod-nodeaffinity-preferred.yaml
pod/pod-nodeaffinity-preferred created

# 查看pod状态 （运行成功）
$ kubectl get pod -o wide
NAME                         READY   STATUS    RESTARTS   AGE   IP               NODE     NOMINATED NODE   READINESS GATES
pod-nodeaffinity-preferred   1/1     Running   0          6s    10.244.196.186   node01   <none>           <none>
```

```none
NodeAffinity规则设置的注意事项：
    1 如果同时定义了nodeSelector和nodeAffinity，那么必须两个条件都得到满足，Pod才能运行在指定的Node上
    2 如果nodeAffinity指定了多个nodeSelectorTerms，那么只需要其中一个能够匹配成功即可
    3 如果一个nodeSelectorTerms中有多个matchExpressions ，则一个节点必须满足所有的才能匹配成功
    4 如果一个pod所在的Node在Pod运行期间其标签发生了改变，不再符合该Pod的节点亲和性需求，则系统将忽略此变化
```



### **PodAffinity**

PodAffinity主要实现以运行的Pod为参照，实现让新创建的Pod跟参照pod在一个区域的功能。

首先来看一下`PodAffinity`的可配置项：

```bash
pod.spec.affinity.podAffinity
  requiredDuringSchedulingIgnoredDuringExecution  硬限制
    namespaces       指定参照pod的namespace
    topologyKey      指定调度作用域
    labelSelector    标签选择器
      matchExpressions  按节点标签列出的节点选择器要求列表(推荐)
        key    键
        values 值
        operator 关系符 支持In, NotIn, Exists, DoesNotExist.
      matchLabels    指多个matchExpressions映射的内容
  preferredDuringSchedulingIgnoredDuringExecution 软限制
    podAffinityTerm  选项
      namespaces      
      topologyKey
      labelSelector
        matchExpressions  
          key    键
          values 值
          operator
        matchLabels 
    weight 倾向权重，在范围1-100
```

```none
topologyKey用于指定调度时作用域,例如:
    如果指定为kubernetes.io/hostname，那就是以Node节点为区分范围
	如果指定为beta.kubernetes.io/os,则以Node节点的操作系统类型来区分
```



接下来，演示`requiredDuringSchedulingIgnoredDuringExecution`,

1）首先创建一个参照Pod，pod-podaffinity-target.yaml：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-podaffinity-target
  labels:
    podenv: pro #设置标签
spec:
  containers:
  - name: nginx
    image: aaronxudocker/myapp:v1.0
  nodeName: node01 # 将目标pod名确指定到node01上
```

```bash
# 查看pod状况
$ kubectl get pod -o wide --show-labels
NAME                     READY   STATUS    RESTARTS   AGE   IP               NODE     NOMINATED NODE   READINESS GATES   LABELS
pod-podaffinity-target   1/1     Running   0          10s   10.244.196.191   node01   <none>           <none>            podenv=pro
```

2）创建pod-podaffinity-required.yaml，内容如下：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-podaffinity-required
spec:
  containers:
  - name: nginx
    image: aaronxudocker/myapp:v1.0
  affinity:  #亲和性设置
    podAffinity: #设置pod亲和性
      requiredDuringSchedulingIgnoredDuringExecution: # 硬限制
      - labelSelector:
          matchExpressions: # 匹配env的值在["xxx","yyy"]中的标签
          - key: podenv
            operator: In
            values: ["xxx","yyy"]
        topologyKey: kubernetes.io/hostname
```

上面配置表达的意思是：新Pod必须要与拥有标签nodeenv=xxx或者nodeenv=yyy的pod在同一Node上，显然现在没有这样pod，接下来，运行测试一下。

```bash
# 查看pod状态，发现未运行
$ kubectl get pod -o wide
NAME                       READY   STATUS    RESTARTS   AGE     IP               NODE     NOMINATED NODE   READINESS GATES
pod-podaffinity-required   0/1     Pending   0          67s     <none>           <none>   <none>           <none>
pod-podaffinity-target     1/1     Running   0          2m29s   10.244.196.191   node01   <none>           <none>

# 查看详细信息
$ kubectl describe pod pod-podaffinity-required
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  106s  default-scheduler  0/3 nodes are available: 1 node(s) had untolerated taint {node-role.kubernetes.io/control-plane: }, 2 node(s) didn't match pod affinity rules. preemption: 0/3 nodes are available: 3 Preemption is not helpful for scheduling.

# 接下来修改  values: ["xxx","yyy"]----->values:["pro","yyy"]
# 意思是：新Pod必须要与拥有标签nodeenv=xxx或者nodeenv=yyy的pod在同一Node上
# 然后重新创建pod，查看效果
# 发现此时Pod运行正常
$ kubectl get pod -o wide
NAME                       READY   STATUS    RESTARTS   AGE    IP               NODE     NOMINATED NODE   READINESS GATES
pod-podaffinity-required   1/1     Running   0          15s    10.244.196.129   node01   <none>           <none>
pod-podaffinity-target     1/1     Running   0          4m8s   10.244.196.191   node01   <none>           <none>
```

关于`PodAffinity`的 `preferredDuringSchedulingIgnoredDuringExecution`，这里不再演示。



### **PodAntiAffinity**

PodAntiAffinity主要实现以运行的Pod为参照，让新创建的Pod跟参照pod不在一个区域中的功能。

它的配置方式和选项跟PodAffinty是一样的，这里不再做详细解释，直接做一个测试案例。

1）继续使用上个案例中目标pod

```bash
$ kubectl get pod --show-labels
NAME                     READY   STATUS    RESTARTS   AGE     LABELS
pod-podaffinity-target   1/1     Running   0          5m31s   podenv=pro
```

2）创建pod-podantiaffinity-required.yaml，内容如下：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-podantiaffinity-required
spec:
  containers:
  - name: nginx
    image: aaronxudocker/myapp:v1.0
  affinity:  #亲和性设置
    podAntiAffinity: #设置pod亲和性
      requiredDuringSchedulingIgnoredDuringExecution: # 硬限制
      - labelSelector:
          matchExpressions: # 匹配podenv的值在["pro"]中的标签
          - key: podenv
            operator: In
            values: ["pro"]
        topologyKey: kubernetes.io/hostname
```

**上面配置表达的意思是：新Pod必须要与拥有标签nodeenv=pro的pod不在同一Node上，运行测试一下。**



```bash
# 查看pod
# 发现调度到了node2上
$ kubectl get pod -o wide
NAME                           READY   STATUS    RESTARTS   AGE     IP               NODE     NOMINATED NODE   READINESS GATES
pod-podaffinity-target         1/1     Running   0          6m30s   10.244.196.191   node01   <none>           <none>
pod-podantiaffinity-required   1/1     Running   0          8s      10.244.140.122   node02   <none>           <none>
```



## 污点和容忍

**污点（Taints）**

前面的调度方式都是站在Pod的角度上，通过在Pod上添加属性，来确定Pod是否要调度到指定的Node上，其实我们也可以站在Node的角度上，通过在Node上添加**污点**属性，来决定是否允许Pod调度过来。

Node被设置上污点之后就和Pod之间存在了一种相斥的关系，进而拒绝Pod调度进来，甚至可以将已经存在的Pod驱逐出去。

污点的格式为：`key=value:effect`, key和value是污点的标签，effect描述污点的作用，支持如下三个选项：

- PreferNoSchedule：kubernetes将**尽量避免把Pod调度到具有该污点的Node上**，除非没有其他节点可调度
- NoSchedule：kubernetes将**不会把Pod调度到具有该污点的Node上**，**但不会影响当前Node上已存在的Pod**
- NoExecute：kubernetes将**不会把Pod调度到具有该污点的Node上**，**同时也会将Node上已存在的Pod驱离**

```bash
# 设置污点
kubectl taint nodes node1 key=value:effect

# 去除污点
kubectl taint nodes node1 key:effect-

# 去除所有污点
kubectl taint nodes node1 key-
```



接下来，演示下污点的效果：

```bash
# 为node01设置污点(PreferNoSchedule)
$ kubectl taint node node01 tag=eagle:PreferNoSchedule
node/node01 tainted

# 创建100个pod，全部都运行在了node02节点上
$ kubectl create deployment myapp --image=aaronxudocker/myapp:v1.0 --replicas=100
deployment.apps/myapp created
$ kubectl get pod -o wide |grep node02|wc -l
100

# 创建1000个pod的时候，node02节点性能不足，不满足预选条件，部分pod运行到了node01上面
$ kubectl get pod -o wide |grep node01|wc -l
106
$ kubectl get pod -o wide |grep node02|wc -l
103

# 为node1设置污点(取消PreferNoSchedule，设置NoSchedule)
$ kubectl taint nodes node01 tag:PreferNoSchedule-
node/node01 untainted
$ kubectl taint nodes node01 tag=eagle:NoSchedule
node/node01 tainted

# 创建pod
$ kubectl create deployment myapp --image=aaronxudocker/myapp:v1.0 --replicas=100
deployment.apps/myapp created

$ kubectl get pod -o wide | grep node02 | wc -l
100

# 关闭node02虚拟机后，再次创建，无法被调度
$ kubectl get pod -o wide
NAME                   READY   STATUS    RESTARTS   AGE   IP       NODE     NOMINATED NODE   READINESS GATES
myapp-9d57f8b4-2jf2f   0/1     Pending   0          4s    <none>   <none>   <none>           <none>
myapp-9d57f8b4-2rkhr   0/1     Pending   0          8s    <none>   <none>   <none>           <none>
myapp-9d57f8b4-2sfbn   0/1     Pending   0          7s    <none>   <none>   <none>           <none>
myapp-9d57f8b4-44hjs   0/1     Pending   0          4s    <none>   <none>   <none>           <none>
 

# 为node1设置污点(取消NoSchedule，设置NoExecute)
$ kubectl taint nodes node01 tag:NoSchedule-

# 创建10个pod，里面有部分是在node01上运行的
$ kubectl create deployment myapp --image=aaronxudocker/myapp:v1.0 --replicas=10
deployment.apps/myapp created

$ kubectl get pod -o wide |grep node01 |wc -l
5

# 设置NoExecute，所有的pod运行到了node02上
$ kubectl taint nodes node01 tag=eagle:NoExecute
$ kubectl get pod -o wide |grep node01 |wc -l
0
```

```none
小提示：
    使用kubeadm搭建的集群，默认就会给master节点添加一个污点标记,所以pod就不会调度到master节点上.
```



**容忍（Toleration）**

上面介绍了污点的作用，我们可以在node上添加污点用于拒绝pod调度上来，但是如果就是想将一个pod调度到一个有污点的node上去，这时候应该怎么做呢？这就要使用到**容忍**。

污点就是拒绝，容忍就是忽略，Node通过污点拒绝pod调度上去，Pod通过容忍忽略拒绝

下面先通过一个案例看下效果：

1. 上一小节，已经在node1节点上打上了`NoExecute`的污点，此时pod是调度不上去的
2. 本小节，可以通过给pod添加容忍，然后将其调度上去



创建pod-toleration.yaml,内容如下

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: myapp
  name: myapp
spec:
  replicas: 10
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - image: aaronxudocker/myapp:v1.0
        name: myapp
      tolerations:      # 添加容忍
      - key: "tag"        # 要容忍的污点的key
        operator: "Equal" # 操作符
        value: "eagle"    # 容忍的污点的value
        effect: "NoExecute"   # 添加容忍的规则，这里必须和标记的污点规则相同
```

```bash
# 添加容忍之后的pod
$ kubectl get pod -o wide
NAME      READY   STATUS    RESTARTS   AGE   IP      NODE     NOMINATED NODE   READINESS GATES
myapp-6687b4f9d7-562lv   1/1     Running   0          3s    10.244.140.150   node02   <none>           <none>
myapp-6687b4f9d7-6m7wb   1/1     Running   0          3s    10.244.196.128   node01   <none>           <none>
myapp-6687b4f9d7-d9tgk   1/1     Running   0          3s    10.244.196.168   node01   <none>           <none>
myapp-6687b4f9d7-fsqw7   1/1     Running   0          3s    10.244.140.159   node02   <none>           <none>
myapp-6687b4f9d7-j7c4x   1/1     Running   0          3s    10.244.140.158   node02   <none>           <none>
myapp-6687b4f9d7-j7dfx   1/1     Running   0          3s    10.244.196.178   node01   <none>           <none>
myapp-6687b4f9d7-ll6z2   1/1     Running   0          3s    10.244.196.140   node01   <none>           <none>
myapp-6687b4f9d7-lwqj4   1/1     Running   0          3s    10.244.140.167   node02   <none>           <none>
myapp-6687b4f9d7-mvdtg   1/1     Running   0          3s    10.244.196.167   node01   <none>           <none>
myapp-6687b4f9d7-qlw5x   1/1     Running   0          3s    10.244.140.147   node02   <none>           <none>     
```



下面看一下容忍的详细配置:

```bash
$ kubectl explain pod.spec.tolerations
......
FIELDS:
   key       # 对应着要容忍的污点的键，空意味着匹配所有的键
   value     # 对应着要容忍的污点的值
   operator  # key-value的运算符，支持Equal和Exists（默认）
   effect    # 对应污点的effect，空意味着匹配所有影响
   tolerationSeconds   # 容忍时间, 当effect为NoExecute时生效，表示pod在Node上的停留时间
```



# 十、安全认证

## 访问控制概述

Kubernetes作为一个分布式集群的管理工具，保证集群的安全性是其一个重要的任务。所谓的安全性其实就是保证对Kubernetes的各种**客户端**进行**认证和鉴权**操作。

**客户端**

在Kubernetes集群中，客户端通常有两类：

- **User Account**：一般是独立于kubernetes之外的其他服务管理的用户账号。
- **Service Account**：kubernetes管理的账号，用于为Pod中的服务进程在访问Kubernetes时提供身份标识。



**认证、授权与准入控制**

ApiServer是访问及管理资源对象的唯一入口。任何一个请求访问ApiServer，都要经过下面三个流程：

- Authentication（认证）：身份鉴别，只有正确的账号才能够通过认证
- Authorization（授权）： 判断用户是否有权限对访问的资源执行特定的动作
- Admission Control（准入控制）：用于补充授权机制以实现更加精细的访问控制功能。



## 认证管理

Kubernetes集群安全的最关键点在于如何识别并认证客户端身份，它提供了3种客户端身份认证方式：

- HTTP Base认证：通过用户名+密码的方式认证
  - 这种认证方式是把“用户名:密码”用BASE64算法进行编码后的字符串放在HTTP请求中的Header Authorization域里发送给服务端。服务端收到后进行解码，获取用户名及密码，然后进行用户身份认证的过程。
- HTTP Token认证：通过一个Token来识别合法用户
  - 这种认证方式是用一个很长的难以被模仿的字符串--Token来表明客户身份的一种方式。每个Token对应一个用户名，当客户端发起API调用请求时，需要在HTTP Header里放入Token，API Server接到Token后会跟服务器中保存的token进行比对，然后进行用户身份认证的过程。
- HTTPS证书认证：基于CA根证书签名的双向数字证书认证方式
  - 这种认证方式是安全性最高的一种方式，但是同时也是操作起来最麻烦的一种方式。



**HTTPS认证大体分为3个过程：**

- 证书申请和下发
  - HTTPS通信双方的服务器向CA机构申请证书，CA机构下发根证书、服务端证书及私钥给申请者
- 客户端和服务端的双向认证
  - 客户端向服务器端发起请求，服务端下发自己的证书给客户端，
  - 客户端接收到证书后，通过私钥解密证书，在证书中获得服务端的公钥，
  - 客户端利用服务器端的公钥认证证书中的信息，如果一致，则认可这个服务器
  - 客户端发送自己的证书给服务器端，服务端接收到证书后，通过私钥解密证书，
  - 在证书中获得客户端的公钥，并用该公钥认证证书信息，确认客户端是否合法

- 服务器端和客户端进行通信
  - 服务器端和客户端协商好加密方案后，客户端会产生一个随机的秘钥并加密，然后发送到服务器端。
  - 服务器端接收这个秘钥后，双方接下来通信的所有内容都通过该随机秘钥加密

> 注意: Kubernetes允许同时配置多种认证方式，只要其中任意一个方式认证通过即可



## 授权管理

授权发生在认证成功之后，通过认证就可以知道请求用户是谁， 然后Kubernetes会根据事先定义的授权策略来决定用户是否有权限访问，这个过程就称为授权。

每个发送到ApiServer的请求都带上了用户和资源的信息：比如发送请求的用户、请求的路径、请求的动作等，授权就是根据这些信息和授权策略进行比较，如果符合策略，则认为授权通过，否则会返回错误。

API Server目前支持以下几种授权策略：

- AlwaysDeny：表示拒绝所有请求，一般用于测试
- AlwaysAllow：允许接收所有请求，相当于集群不需要授权流程（Kubernetes默认的策略）
- ABAC：基于属性的访问控制，表示使用用户配置的授权规则对用户请求进行匹配和控制
- Webhook：通过调用外部REST服务对用户进行授权
- Node：是一种专用模式，用于对kubelet发出的请求进行访问控制
- RBAC：基于角色的访问控制（kubeadm安装方式下的默认选项）

RBAC(Role-Based Access Control) 基于角色的访问控制，主要是在描述一件事情：**给哪些对象授予了哪些权限**

其中涉及到了下面几个概念：

- 对象：User、Groups、ServiceAccount
- 角色：代表着一组定义在资源上的可操作动作(权限)的集合
- 绑定：将定义好的角色跟用户绑定在一起



RBAC引入了4个顶级资源对象：

- Role、ClusterRole：角色，用于指定一组权限
- RoleBinding、ClusterRoleBinding：角色绑定，用于将角色（权限）赋予给对象

**Role、ClusterRole**

一个角色就是一组权限的集合，这里的权限都是许可形式的（白名单）。

```yaml
# Role只能对命名空间内的资源进行授权，需要指定nameapce
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: authorization-role
rules:
- apiGroups: [""]  # 支持的API组列表,"" 空字符串，表示核心API群
  resources: ["pods"] # 支持的资源对象列表
  verbs: ["get", "watch", "list"] # 允许的对资源对象的操作方法列表
# ClusterRole可以对集群范围内资源、跨namespaces的范围资源、非资源类型进行授权
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
 name: authorization-clusterrole
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

需要详细说明的是，rules中的参数：

- apiGroups: 支持的API组列表

```
"","apps", "autoscaling", "batch"
```

- resources：支持的资源对象列表

```
"services", "endpoints", "pods","secrets","configmaps","crontabs","deployments","jobs",
"nodes","rolebindings","clusterroles","daemonsets","replicasets","statefulsets",
"horizontalpodautoscalers","replicationcontrollers","cronjobs"
```

- verbs：对资源对象的操作方法列表

```
"get", "list", "watch", "create", "update", "patch", "delete", "exec"
```



**RoleBinding、ClusterRoleBinding**

角色绑定用来把一个角色绑定到一个目标对象上，绑定目标可以是User、Group或者ServiceAccount。

```yaml
# RoleBinding可以将同一namespace中的subject绑定到某个Role下，则此subject即具有该Role定义的权限
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: authorization-role-binding
  namespace: dev
subjects:
- kind: User
  name: eagle
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: authorization-role
  apiGroup: rbac.authorization.k8s.io
# ClusterRoleBinding在整个集群级别和所有namespaces将特定的subject与ClusterRole绑定，授予权限
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
 name: authorization-clusterrole-binding
subjects:
- kind: User
  name: eagle
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: authorization-clusterrole
  apiGroup: rbac.authorization.k8s.io
```



**RoleBinding引用ClusterRole进行授权**

RoleBinding可以引用ClusterRole，对属于同一命名空间内ClusterRole定义的资源主体进行授权。

一种很常用的做法就是，集群管理员为集群范围预定义好一组角色（ClusterRole），然后在多个命名空间中重复使用这些ClusterRole。这样可以大幅提高授权管理工作效率，也使得各个命名空间下的基础性授权规则与使用体验保持一致。

```yaml
# 虽然authorization-clusterrole是一个集群角色，但是因为使用了RoleBinding
# 所以eagle只能读取dev命名空间中的资源
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: authorization-role-binding-ns
  namespace: dev
subjects:
- kind: User
  name: eagle
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: authorization-clusterrole
  apiGroup: rbac.authorization.k8s.io
```



**实战：创建一个只能管理dev空间下Pods资源的账号**

1) 创建账号

```shell
# 1) 创建证书
$ cd /etc/kubernetes/pki/
$ (umask 077;openssl genrsa -out devman.key 2048)

# 2) 用apiserver的证书去签署
# 2-1) 签名申请，申请的用户是devman,组是devgroup
$ openssl req -new -key devman.key -out devman.csr -subj "/CN=devman/O=devgroup"     
# 2-2) 签署证书
$ openssl x509 -req -in devman.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out devman.crt -days 3650

# 3) 设置集群、用户、上下文信息
$ kubectl config set-cluster kubernetes --embed-certs=true --certificate-authority=/etc/kubernetes/pki/ca.crt --server=https://192.168.173.100:6443

$ kubectl config set-credentials devman --embed-certs=true --client-certificate=/etc/kubernetes/pki/devman.crt --client-key=/etc/kubernetes/pki/devman.key

$ kubectl config set-context devman@kubernetes --cluster=kubernetes --user=devman

# 切换账户到devman
$ kubectl config use-context devman@kubernetes
Switched to context "devman@kubernetes".

# 查看default下pod，发现没有权限
$ kubectl get pods
Error from server (Forbidden): pods is forbidden: User "devman" cannot list resource "pods" in API group "" in the namespace "default"

# 切换回admin账户
$ kubectl config use-context kubernetes-admin@kubernetes
Switched to context "kubernetes-admin@kubernetes".
```



2） 创建Role和RoleBinding，为devman用户授权，dev-role.yaml

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: dev-role
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]

---

kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: authorization-role-binding
  namespace: default
subjects:
- kind: User
  name: devman
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: dev-role
  apiGroup: rbac.authorization.k8s.io
```

3) 切换账户，再次验证

```shell
# 切换账户到devman
$ kubectl config use-context devman@kubernetes
Switched to context "devman@kubernetes".

# 再次查看
$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
myapp-6687b4f9d7-562lv   1/1     Running   0          31m
myapp-6687b4f9d7-6m7wb   1/1     Running   0          31m
myapp-6687b4f9d7-d9tgk   1/1     Running   0          31m
myapp-6687b4f9d7-fsqw7   1/1     Running   0          31m
myapp-6687b4f9d7-j7c4x   1/1     Running   0          31m
myapp-6687b4f9d7-j7dfx   1/1     Running   0          31m
myapp-6687b4f9d7-ll6z2   1/1     Running   0          31m
myapp-6687b4f9d7-lwqj4   1/1     Running   0          31m
myapp-6687b4f9d7-mvdtg   1/1     Running   0          31m
myapp-6687b4f9d7-qlw5x   1/1     Running   0          31m

# 为了不影响后面的学习,切回admin账户
$ kubectl config use-context kubernetes-admin@kubernetes
Switched to context "kubernetes-admin@kubernetes".
```



## 准入控制

通过了前面的认证和授权之后，还需要经过准入控制处理通过之后，apiserver才会处理这个请求。

准入控制是一个可配置的控制器列表，可以通过在Api-Server上通过命令行设置选择执行哪些准入控制器：

```
--admission-control=NamespaceLifecycle,LimitRanger,ServiceAccount,PersistentVolumeLabel,
                      DefaultStorageClass,ResourceQuota,DefaultTolerationSeconds
```

只有当所有的准入控制器都检查通过之后，apiserver才执行该请求，否则返回拒绝。

当前可配置的Admission Control准入控制如下：

- AlwaysAdmit：允许所有请求
- AlwaysDeny：禁止所有请求，一般用于测试
- AlwaysPullImages：在启动容器之前总去下载镜像
- DenyExecOnPrivileged：它会拦截所有想在Privileged Container上执行命令的请求
- ImagePolicyWebhook：这个插件将允许后端的一个Webhook程序来完成admission controller的功能。
- Service Account：实现ServiceAccount实现了自动化
- SecurityContextDeny：这个插件将使用SecurityContext的Pod中的定义全部失效
- ResourceQuota：用于资源配额管理目的，观察所有请求，确保在namespace上的配额不会超标
- LimitRanger：用于资源限制管理，作用于namespace上，确保对Pod进行资源限制
- InitialResources：为未设置资源请求与限制的Pod，根据其镜像的历史资源的使用情况进行设置
- NamespaceLifecycle：如果尝试在一个不存在的namespace中创建资源对象，则该创建请求将被拒绝。当删除一个namespace时，系统将会删除该namespace中所有对象。
- DefaultStorageClass：为了实现共享存储的动态供应，为未指定StorageClass或PV的PVC尝试匹配默认的StorageClass，尽可能减少用户在申请PVC时所需了解的后端存储细节
- DefaultTolerationSeconds：这个插件为那些没有设置forgiveness tolerations并具有notready:NoExecute和unreachable:NoExecute两种taints的Pod设置默认的“容忍”时间，为5min
- PodSecurityPolicy：这个插件用于在创建或修改Pod时决定是否根据Pod的security context和可用的PodSecurityPolicy对Pod的安全策略进行控制



# 十一、DashBoard

之前在kubernetes中完成的所有操作都是通过命令行工具kubectl完成的。其实，为了提供更丰富的用户体验，kubernetes还开发了一个基于web的用户界面（Dashboard）。用户可以使用Dashboard部署容器化的应用，还可以监控应用的状态，执行故障排查以及管理kubernetes中各种资源。



## 部署Dashboard

1) 下载yaml，并运行Dashboard

```shell
# 下载yaml
[root@k8s-master01 ~]# wget  https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml

# 修改kubernetes-dashboard的Service类型
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  type: NodePort  # 新增
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 30009  # 新增
  selector:
    k8s-app: kubernetes-dashboard

# 部署
[root@k8s-master01 ~]# kubectl create -f recommended.yaml

# 查看namespace下的kubernetes-dashboard下的资源
[root@k8s-master01 ~]# kubectl get pod,svc -n kubernetes-dashboard
NAME                                            READY   STATUS    RESTARTS   AGE
pod/dashboard-metrics-scraper-c79c65bb7-zwfvw   1/1     Running   0          111s
pod/kubernetes-dashboard-56484d4c5-z95z5        1/1     Running   0          111s

NAME                               TYPE       CLUSTER-IP      EXTERNAL-IP  PORT(S)         AGE
service/dashboard-metrics-scraper  ClusterIP  10.96.89.218    <none>       8000/TCP        111s
service/kubernetes-dashboard       NodePort   10.104.178.171  <none>       443:30009/TCP   111s
```



2）创建访问账户，获取token

```shell
# 创建账号
[root@k8s-master01-1 ~]# kubectl create serviceaccount dashboard-admin -n kubernetes-dashboard

# 授权
[root@k8s-master01-1 ~]# kubectl create clusterrolebinding dashboard-admin-rb --clusterrole=cluster-admin --serviceaccount=kubernetes-dashboard:dashboard-admin

# 获取账号token
[root@k8s-master01 ~]#  kubectl get secrets -n kubernetes-dashboard | grep dashboard-admin
dashboard-admin-token-xbqhh        kubernetes.io/service-account-token   3      2m35s

[root@k8s-master01 ~]# kubectl describe secrets dashboard-admin-token-xbqhh -n kubernetes-dashboard
Name:         dashboard-admin-token-xbqhh
Namespace:    kubernetes-dashboard
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: dashboard-admin
              kubernetes.io/service-account.uid: 95d84d80-be7a-4d10-a2e0-68f90222d039

Type:  kubernetes.io/service-account-token

Data
====
namespace:  20 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6ImJrYkF4bW5XcDhWcmNGUGJtek5NODFuSXl1aWptMmU2M3o4LTY5a2FKS2cifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkYXNoYm9hcmQtYWRtaW4tdG9rZW4teGJxaGgiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGFzaGJvYXJkLWFkbWluIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiOTVkODRkODAtYmU3YS00ZDEwLWEyZTAtNjhmOTAyMjJkMDM5Iiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmVybmV0ZXMtZGFzaGJvYXJkOmRhc2hib2FyZC1hZG1pbiJ9.NAl7e8ZfWWdDoPxkqzJzTB46sK9E8iuJYnUI9vnBaY3Jts7T1g1msjsBnbxzQSYgAG--cV0WYxjndzJY_UWCwaGPrQrt_GunxmOK9AUnzURqm55GR2RXIZtjsWVP2EBatsDgHRmuUbQvTFOvdJB4x3nXcYLN2opAaMqg3rnU2rr-A8zCrIuX_eca12wIp_QiuP3SF-tzpdLpsyRfegTJZl6YnSGyaVkC9id-cxZRb307qdCfXPfCHR_2rt5FVfxARgg_C0e3eFHaaYQO7CitxsnIoIXpOFNAR8aUrmopJyODQIPqBWUehb7FhlU1DCduHnIIXVC_UICZ-MKYewBDLw
ca.crt:     1025 bytes
```



3）通过浏览器访问Dashboard的UI

在登录页面上输入上面的token，出现控制台的页面代表成功



## 使用DashBoard

本章节以Deployment为例演示DashBoard的使用

**查看**

选择指定的命名空间`dev`，然后点击`Deployments`，查看dev空间下的所有deployment

**扩缩容**

在`Deployment`上点击`规模`，然后指定`目标副本数量`，点击确定

**编辑**

在`Deployment`上点击`编辑`，然后修改`yaml文件`，点击确定

**查看Pod**

点击`Pods`, 查看pods列表

**操作Pod**

选中某个Pod，可以对其执行日志（logs）、进入执行（exec）、编辑、删除操作



## kuboard

官方网站：https://kuboard.cn/

### 特点介绍

相较于 Kubernetes Dashboard 等其他 Kubernetes 管理界面，Kuboard 的优势更加明显:

**多种认证方式**

Kuboard 可以使用内建用户库、gitlab / github 单点登录或者 LDAP 用户库进行认证，避免管理员将 ServiceAccount 的 Token 分发给普通用户而造成的麻烦。使用内建用户库时，管理员可以配置用户的密码策略、密码过期时间等安全设置。



**多集群管理**

管理员可以将多个 Kubernetes 集群导入到 Kuboard 中，并且通过权限控制，将不同集群/名称空间的权限分配给指定的用户或用户组。



**微服务分层展示**

在 Kuboard 的名称空间概要页中，以经典的微服务分层方式将工作负载划分到不同的分层，更加直观地展示微服务架构的结构，并且可以为每一个名称空间自定义名称空间布局。



**工作负载的直观展示**

Kuboard 中将 Deployment 的历史版本、所属的 Pod 列表、Pod 的关联事件、容器信息合理地组织在同一个页面中，可以帮助用户最快速的诊断问题和执行各种相关操作。



**工作负载编辑**

Kuboard 提供了图形化的工作负载编辑界面，用户无需陷入繁琐的 YAML 文件细节中，即可轻松完成对容器的编排任务。支持的 Kubernetes 对象类型包括：Node、Namespace、Deployment、StatefulSet、DaemonSet、Secret、ConfigMap、Service、Ingress、StorageClass、PersistentVolumeClaim、LimitRange、ResourceQuota、ServiceAccount、Role、RoleBinding、ClusterRole、ClusterRoleBinding、CustomResourceDefinition、CustomResource 等各类常用 Kubernetes 对象，



**存储类型支持**

在 Kuboard 中，可以方便地对接 NFS、CephFS 等常用存储类型，并且支持对 CephFS 类型的存储卷声明执行扩容和快照操作



**丰富的互操作性**

可以提供许多通常只在 `kubectl` 命令行界面中才提供的互操作手段，例如：

- Top Nodes / Top Pods

- 容器的日志、终端

- 容器的文件浏览器（支持从容器中下载文件、上传文件到容器）

- KuboardProxy（在浏览器中就可以提供 `kubectl proxy` 的功能）



**套件扩展**

  Kuboard 提供了必要的套件库，使得用户可以根据自己的需要扩展集群的管理能力。当前提供的套件有：
  - 资源层监控套件，基于 Prometheus / Grafana 提供 K8S 集群的监控能力，可以监控集群、节点、工作负载、容器组等各个级别对象的 CPU、内存、网络、磁盘等资源的使用情况；

  - 日志聚合套件，基于 Grafana / Loki / Promtail 实现日志聚合；

  - 存储卷浏览器，查看和操作存储卷中的内容；



**告警配置**

 可以通过界面直接配置资源层监控套件发送告警消息：

- 支持邮件、微信发送告警消息；
- 支持告警路由配置；
- 支持告警规则配置等；



**操作审计**

Kuboard 支持操作审计的功能：

- 审计用户通过 Kuboard 界面和 Kuboard API 执行的操作；
- 自定义审计规则；



### 安装教程

https://kuboard.cn/v4/install/quickstart.html

安装好后链接上集群，最终界面类似dashboard
