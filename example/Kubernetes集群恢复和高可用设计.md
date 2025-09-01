# Kubernetes集群恢复和高可用设计项目实战

## 一、学习目标

K8S集群的故障恢复（ IP地址发生变更后，我们要怎么恢复？需要熟悉k8s中的相关组件和各组件的协调工作的流程 ）
    Calico 网络插件
    LNMP 业务异常（主要变更导致的） 
    MySQL 变更管理 
    数据持久化：当我们的POD重启/ 销毁重建的时候，DB（MySQL Redis） 数据不能丢
    现有集群负载比较高：需要新增 node 节点
    当业务流量突增的时候， 我们POD要能够水平扩缩容 HPA控制器 扩副本？ 垂直扩缩容VPA？扩cpu和mem 
    MySQL高可用部署？ 
    Redis如果进行哨兵/集群模式的高可用部署？



## 二、实战内容

任务1：将每个节点的IP地址更改为需求的网段IP

可能会出现的异常现象：

异常现象1: INTERNAL-IP 字段是不对的？
异常现象2: [root@master01 ~]# kubectl get nodes
Unable to connect to the server: dial tcp 10.3.201.100:6443: connect: no route to host
异常现象3: Unable to connect to the server: tls: failed to verify certificate: x509: certificate is valid for 10.0.0.1, 10.3.201.100, not 10.3.204.100


任务2:：解决完上述所有异常现象，然后 kubectl get nodes -o wide 命令输出的集群信息是正常的


任务3：恢复 kube-system 里 calico 组件

```shell
[root@master01 ~]# kubectl get pods -n kube-system | grep calico
calico-kube-controllers-558d465845-4blqv   0/1     CrashLoopBackOff   8 (2m5s ago)      148d
calico-node-4t2zw                          0/1     Init:1/3           0                 8s
calico-node-sggl6                          0/1     Init:1/3           0                 8s
calico-node-tfvhv                          0/1     Init:1/3           0                 8s
calico-typha-5b56944f9b-gnrpr              1/1     Running            1 (18m ago)       148d
```


任务4: LNMP业务异常恢复

- 将`/root/resources/01.nginx.yaml,02.phpfpm.yaml,03.mysql.yaml`等文件中NFS地址指向本集群master01
- 为`nginx.yaml,phpfpm.yaml`等文件添加健康检查机制
- 检查`nginx php-fpm mysql`服务
- 通过`bbs.iproute.cn`访问正常


任务5: MySQL变更管理

- 修改新密码为:`123456`
- 访问 `bbs.iproute.cn` 正常


任务6: Redis持久化管理

- 配置文件通过Configmap挂载至 `/usr/local/etc/redis/redis.conf`
- 数据目录通过NFS挂载至 `/data`
- 测试验证


任务7：新增工作节点，添加1台node


扩展：k8s master 节点的高可用方案；水平扩缩容；垂直扩缩容



## 三、操作命令

任务1：将每个节点的IP地址更改为需求的网段IP

```shell
Unable to connect to the server: dial tcp 10.3.201.100:6443: connect: no route to host --> (master) api_server 配置没有更新

master -> 10.3.202.100 

# 更新 k8s master节点相关配置文件，将旧IP地址更新为新IP地址
  813  sed  -i 's/10.3.201/10.3.202/g' /etc/kubernetes/manifests/kube-apiserver.yaml
  815  sed  -i 's/10.3.201/10.3.202/g' /etc/kubernetes/manifests/etcd.yaml
  817  sed  -i 's/10.3.201/10.3.202/g' /etc/kubernetes/super-admin.conf
  818  sed  -i 's/10.3.201/10.3.202/g' /etc/kubernetes/admin.conf
  819  sed  -i 's/10.3.201/10.3.202/g' /etc/kubernetes/scheduler.conf
  820  sed  -i 's/10.3.201/10.3.202/g' /etc/kubernetes/manifests/etcd.yaml
  821  sed  -i 's/10.3.201/10.3.202/g' /etc/kubernetes/kubelet.conf

# 检查 /etc/kubernetes/ 不存在旧的IP地址
[root@master01 ~]# grep -nr  '201.100' /etc/kubernetes/*

# 发现 .kube/config 里还有指向旧 api_server 的ip地址
[root@master01 ~]# ls -l .kube/config
-rw------- 1 root root 5652 Feb 19 10:34 .kube/config
sed  -i 's/10.3.201/10.3.202/g' .kube/config

# 重启 docker 服务和 kubelet 服务
 systemctl restart docker kubelet

# api_server & controller-manager & scheduler 默认安装在 kube-system 命名空间下
[root@master01 ~]# kubectl get pods -n kube-system -o wide | grep 100 

# 检查 node 节点，仅 kubelet 服务的配置需要更新
sed  -i 's/10.3.201/10.3.202/g' /etc/kubernetes/kubelet.conf

# ca 证书验证失败，我们需要重新生成新的证书
[root@master01 ~]# kubectl get nodes -o wide

# E0717 18:54:19.018747 1117322 memcache.go:265] couldn't get current server API group list: Get "https://10.3.202.100:6443/api?timeout=32s": tls: failed to verify certificate: x509: certificate is valid for 10.0.0.1, 10.3.201.100, not 10.3.202.100

# 备份配置文件
[root@master01 ~]# mv /etc/kubernetes/pki/apiserver.crt backup/
[root@master01 ~]# mv /etc/kubernetes/pki/apiserver.key backup/

## 输出 kubeadm init 操作时默认的模版配置文件
[root@master01 ~]# kubeadm  config print init-defaults

## 仅重新生成 api_server 的初始化
[root@master01 ~]# kubeadm init phase certs apiserver
I0717 19:02:38.392738 1119172 version.go:256] remote version is much newer: v1.33.3; falling back to: stable-1.29
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local master01] and IPs [10.96.0.1 10.3.202.100]

## 测试验证
[root@master01 ~]# kubectl get nodes -o wide
NAME       STATUS   ROLES           AGE    VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE                      KERNEL-VERSION                 CONTAINER-RUNTIME
master01   Ready    control-plane   148d   v1.29.2   10.3.202.100   <none>        Rocky Linux 9.4 (Blue Onyx)   5.14.0-427.13.1.el9_4.x86_64   docker://27.2.0
node01     Ready    <none>          148d   v1.29.2   10.3.202.101   <none>        Rocky Linux 9.4 (Blue Onyx)   5.14.0-427.13.1.el9_4.x86_64   docker://27.2.0
node02     Ready    <none>          148d   v1.29.2   10.3.202.102   <none>        Rocky Linux 9.4 (Blue Onyx)   5.14.0-427.13.1.el9_4.x86_64   docker://27.2.0

## 还有一种做法处理node节点？？ 先把node节点从集群中踢出去，踢出去之后再加进来。
1. 驱除工作节点 kubectl drain <node>
2. 删除pod kubectl delete pod <node_ip>  (--all-namespaces)
3. 设置为不可调度节点： kubectl uncordon <node>
4. kubeadm join xxxx 

## 工作场景：比如有一台k8s的node发生物理故障（网络经常性抖动 相对于其他节点上的pod 延时会高或者忽高忽低 ｜ cpu ｜ mem ） 

## 优雅下线机器？ 确保pod在其他node上有重新创建新pod 暴力下线（xxx）
## kube-controller 组件？？  检查当前pod/其他api对象的副本数量 是否 跟预期状态一致？ -> kube-scheduler 重新调度 -> kubelet在对应的node节点上创建新的pod
```



任务2：解决完上述所有异常现象，然后 kubectl get nodes -o wide 命令输出的集群信息是正常的

异常现象：

1. k8s集群上还有很多异常的pod
   副本不符合预期的：
    kube-system   coredns-857d9ff4c9-26lmv                   0/1     Running            234 (5m40s ago)   148d
   STATUS是 ： ImagePullBackOff、ImagePullBackOff 都是一些异常的 要怎么处理？？ 
2. kube-system   calico-kube-controllers-558d465845-4blqv   0/1     CrashLoopBackOff   6 (104s ago)      148d
   有个核心组件（k8s的集群网络组件）



任务3：恢复 kube-system 里 calico 组件

```shell
[root@master01 ~]# kubectl get pods -n kube-system | grep calico
calico-kube-controllers-558d465845-4blqv   0/1     CrashLoopBackOff   8 (2m5s ago)      148d
calico-node-4t2zw                          0/1     Init:1/3           0                 8s
calico-node-sggl6                          0/1     Init:1/3           0                 8s
calico-node-tfvhv                          0/1     Init:1/3           0                 8s
calico-typha-5b56944f9b-gnrpr              1/1     Running            1 (18m ago)       148d

calico-node: 每个节点上都会部署，负责网络的转发，数据层面
calico-kube-controllers : master节点上，负责网络控制层面

[root@master01 ~]# kubectl describe pod calico-node-4t2zw  -n kube-system | grep -A10 Events
Events:
  Type     Reason   Age                     From     Message

----     ------   ----                    ----     -------

  Warning  BackOff  5m9s (x10989 over 47h)  kubelet  Back-off restarting failed container install-cni in pod calico-node-4t2zw_kube-system(ce84e3b8-a32f-4f5b-86df-0bbdc1c01f46)

Pod 是由一个/多个 container 

calico-node 

- initContainers : 初始化container，一般pod在启动过程中会先等待init_container运行成功
  upgrade-ipam
  install-cni
  mount-bpffs
- containers
  calico-node

要学会通过日志找到问题
[root@master01 resources]# kubectl logs -n kube-system calico-node-4t2zw -c install-cni
W0719 11:42:34.766867       1 client_config.go:618] Neither --kubeconfig nor --master was specified.  Using the inClusterConfig.  This might not work.
2025-07-19 11:42:34.777 [ERROR][1] cni-installer/<nil> <nil>: Unable to create token for CNI kubeconfig error=Unauthorized
2025-07-19 11:42:34.777 [FATAL][1] cni-installer/<nil> <nil>: Unable to create token for CNI kubeconfig error=Unauthorized

install-install -> api_server 权限是不是有问题？有没有配置正确？？
[root@master01 resources]# kubectl logs calico-kube-controllers-558d465845-4blqv -n kube-system | grep -i err
2025-07-19 11:45:24.309 [ERROR][1] client.go 295: Error getting cluster information config ClusterInformation="default" error=Get "https://10.0.0.1:443/apis/crd.projectcalico.org/v1/clusterinformations/default": dial tcp 10.0.0.1:443: connect: no route to host
2025-07-19 11:45:24.309 [INFO][1] main.go 138: Failed to initialize datastore error=Get "https://10.0.0.1:443/apis/crd.projectcalico.org/v1/clusterinformations/default": dial tcp 10.0.0.1:443: connect: no route to host

10.0.0.1:443 -> api_server对内的访问地址 
10.3.202.100:6443 -> 对外的访问地址

# 将 kube-proxy 里的旧IP地址更改为新，保存退出后热加载配置
kubectl edit cm -n kube-system kube-proxy

# 删除旧 pod 
kubectl delete pod -n kube-system -l k8s-app=kube-proxy --force=true
kubectl delete pod -n kube-system -l k8s-app=calico-node --force=true
kubectl delete pod -n kube-system -l k8s-app=calico-kube-controllers --force=true

# 再去追踪新的日志
[root@master01 resources]# kubectl logs -n kube-system calico-node-4stnd -c install-cni
2025-07-19 11:56:18.810 [ERROR][1] cni-installer/<nil> <nil>: Unable to create token for CNI kubeconfig error=Post "https://10.0.0.1:443/api/v1/namespaces/kube-system/serviceaccounts/calico-cni-plugin/token": tls: failed to verify certificate: x509: certificate is valid for 10.96.0.1, 10.3.202.100, not 10.0.0.1
2025-07-19 11:56:18.812 [FATAL][1] cni-installer/<nil> <nil>: Unable to create token for CNI kubeconfig error=Post "https://10.0.0.1:443/api/v1/namespaces/kube-system/serviceaccounts/calico-cni-plugin/token": tls: failed to verify certificate: x509: certificate is valid for 10.96.0.1, 10.3.202.100, not 10.0.0.1

# 仅重新生成 api_server 的初始化
[root@master01 ~]# kubeadm init phase certs apiserver
I0717 19:02:38.392738 1119172 version.go:256] remote version is much newer: v1.33.3; falling back to: stable-1.29
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local master01] and IPs [10.96.0.1 10.3.202.100] # SAN 10.96.0.1 10.3.202.100 这两个IP 

# 再重新上生成一个证书
[root@master01 ~]# cat kubeadm_config.yaml
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
apiServer:
  certSANs:
    - "10.3.202.100"
    - "10.0.0.1"

[root@master01 ~]# kubeadm init phase certs apiserver --config kubeadm_config.yaml
I0719 20:02:55.309534 2040872 version.go:256] remote version is much newer: v1.33.3; falling back to: stable-1.29
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local master01] and IPs [10.96.0.1 10.3.202.100 10.0.0.1]

[root@master01 ~]# kubectl logs -n kube-system  calico-kube-controllers-558d465845-nsp27
2025-07-19 12:05:48.772 [ERROR][1] client.go 295: Error getting cluster information config ClusterInformation="default" error=Get "https://10.0.0.1:443/apis/crd.projectcalico.org/v1/clusterinformations/default": dial tcp 10.0.0.1:443: connect: no route to host

## 将 cluster-info cm 旧IP地址转为新IP
[root@master01 ~]# kubectl edit cm cluster-info -n kube-public

## 完全清理calico相关pod
[root@master01 ~]# kubectl delete pod $(kubectl get pods -A  | grep calico | awk '{print $2}') -n kube-system

[root@master01 ~]# kubectl get pods -A  | grep calico
kube-system   calico-kube-controllers-558d465845-xh7sd   1/1     Running            0                 4m37s
kube-system   calico-node-f4ncr                          1/1     Running            0                 4m36s
kube-system   calico-node-h7zl9                          1/1     Running            0                 4m36s
kube-system   calico-node-zg2bb                          1/1     Running            0                 4m36s
kube-system   calico-typha-5b56944f9b-fkbsc              1/1     Running            0                 4m36s
```



任务4: LNMP业务异常恢复

```shell
## 指向本集群master01 nfs的需求
使用v1版本的yaml文件： kubectl apply -f xxxx
[root@master01 ~]# cat 01.nginx_v1.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
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
        image: nginx:1.26.3
        ports:
        - containerPort: 80
        volumeMounts:
        - name: html
          mountPath: /usr/share/nginx/html
        - name: config
          mountPath: /etc/nginx/conf.d
        startupProbe:
          httpGet:
            path: /
            port: 80
          failureThreshold: 30
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
      volumes:
      - name: html
        nfs:
          server: 10.3.205.100
          path: /root/data/nfs/html
      volumes:
      - name: config
        nfs:
          server: 10.3.205.100
          path: /root/data/nfs/nginx
      - name: html
        nfs:
          server: 10.3.205.100
          path: /root/data/nfs/html

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

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-nginx
spec:
  rules:
  - host: bbs.iproute.cn
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
  
## 为什么需要增加健康检查？ 通过Pod 的status看着是running 但是业务实际上访问是有问题

## 当加上健康检查后, 判断健康检查是否生效？
nginx-6f844478b4-sqcmm   0/1     Running            0

  Warning  Unhealthy  79s (x5 over 118s)  kubelet            Startup probe failed: HTTP probe failed with statuscode: 404

## phpfpm targetPort 900 -> 9000

## check nginx 
[root@master01 resources]# kubectl get pods -o wide
NAME                     READY   STATUS             RESTARTS        AGE     IP               NODE     NOMINATED NODE   READINESS GATES
mysql-5cbcfd9d85-q24qb   1/1     Running            0               22m     10.244.196.175   node01   <none>           <none>
nginx-6f844478b4-5whnc   1/1     Running            0               38s     10.244.140.127   node02   <none>           <none>
nginx-6f844478b4-hxrtd   1/1     Running            0               50s     10.244.140.126   node02   <none>           <none>
nginx-6f844478b4-pf84b   1/1     Running            3 (5m27s ago)   5m37s   10.244.196.177   node01   <none>           <none>
php-5b8ff558c8-7zgr9     1/1     Running            0               14m     10.244.140.125   node02   <none>           <none>
php-5b8ff558c8-g46sk     1/1     Running            0               14m     10.244.140.124   node02   <none>           <none>
php-5b8ff558c8-x4hr9     1/1     Running            0               14m     10.244.196.173   node01   <none>           <none>

# ingress-controller： svc (LoadBalancer)
[root@master01 resources]# kubectl get svc -A | grep ingress
ingress       ingress-nginx-controller             LoadBalancer   10.10.47.140    <pending>     80:30331/TCP,443:31980/TCP   150d
ingress       ingress-nginx-controller-admission   ClusterIP      10.10.236.175   <none>        443/TCP                      150d

# ingress 资源对象
[root@master01 resources]# kubectl get ingress
NAME            CLASS   HOSTS            ADDRESS   PORTS   AGE
ingress-nginx   nginx   bbs.iproute.cn             80      150d

# pod
[root@master01 resources]# kubectl get pods -A -o wide | grep ingress
ingress       ingress-nginx-controller-27rlh             1/1     Running            13 (2d23h ago)   150d    10.3.205.102     node02     <none>           <none>
ingress       ingress-nginx-controller-mqdll             1/1     Running            2 (3d ago)       150d    10.3.205.101     node01     <none>           <none>              /

# 访问方式1:
[root@master01 resources]# curl -I -H "Host: bbs.iproute.cn" 10.3.205.101:30331
HTTP/1.1 200 OK

# 访问方式2:
[root@master01 resources]# grep bbs /etc/hosts
10.3.205.101 bbs.iproute.cn
[root@master01 resources]# curl -I bbs.iproute.cn
HTTP/1.1 200 OK
```



任务5: MySQL变更管理

```shell
# 这里并不能直接去更新 mysql 的密码，仅仅是在pod第一次创建初始化的时候设置的 root 密码
env:
- name: MARIADB_ROOT_PASSWORD
  value: "123"

## 进入 mysql 容器内
[root@master01 ~]# kubectl get pod  -A | grep mysql
default       mysql-6c6797b497-r98zb
[root@master01 ~]# kubectl exec -it mysql-6c6797b497-r98zb -- /bin/bash
root@mysql-6c6797b497-r98zb:/# mysql -uroot -p123
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 47037
Server version: 10.3.39-MariaDB-1:10.3.39+maria~ubu2004 mariadb.org binary distribution
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

# root 用户密码更新
MariaDB [(none)]> ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]> ALTER USER 'root'@'%' IDENTIFIED BY '123456';
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.000 sec)

# 发现 nginx pod 健康检查失败
[root@master01 ~]# kubectl get pod -A | grep nginx
default       nginx-7885fdbbbb-2mcm5                     0/1     Running            1 (57s ago)     21h
default       nginx-7885fdbbbb-d6cjw                     0/1     Running            1 (55s ago)     21h
default       nginx-7885fdbbbb-g6g8g                     0/1     Running            1 (54s ago)     21h
10.3.202.101 - - [21/Jul/2025:10:40:50 +0000] "GET / HTTP/1.1" 503 4209 "-" "kube-probe/1.29" "-"

# 容器内？ NFS？  NFS 的路径目录有挂载给 POD
NFS 配置： /root/data/nfs/php   -> POD: /usr/local/etc
NFS 应用项目： /root/data/nfs/html   ->  POD: /usr/share/nginx/html

# YAML 这种声明式的方式 >> 命令行

# app 应用日志在哪里？ 配置在哪里？ -> 去研发 -> 看整个代码项目结构（存储的位置）
[root@master01 ~]# ls -l /root/data/nfs/html/data/log/
total 44
-rw-r--r-- 1 www-data tape     17187 Feb 21 09:09 202502_cplog.php
-rw-r--r-- 1 www-data tape      1548 Jul 21 18:39 202507_errorlog.php
-rw-r--r-- 1 www-data www-data     0 Feb 20 09:11 index.htm
-rw-r--r-- 1 www-data tape     17066 Feb 20 10:08 install.log

# 处理报错的流程：告警 -> 监控 -> 看日志 -> 明确定义到某一行代码报错 -> 自己尝试复现

# 错误日志 
<?PHP exit;?>	1753094940	<b>(0) notconnect</b><br><b>PHP:</b>index.php#require(%s):0136 -> forum.php#discuz_application->discuz_application->init():0057 -> source/class/discuz/discuz_application.php#discuz_application->discuz_application->_init_db():0066 -> source/class/discuz/discuz_application.php#discuz_database::discuz_database::init():0444 -> source/class/discuz/discuz_database.php#db_driver_mysqli->db_driver_mysqli->connect():0023 -> source/class/db/db_driver_mysqli.php#db_driver_mysqli->db_driver_mysqli->_dbconnect():0074 -> source/class/db/db_driver_mysqli.php#db_driver_mysqli->db_driver_mysqli->halt():0085 -> source/class/db/db_driver_mysqli.php#break():0222	087936707e83fa1c7c591bcfba36bb39	<b>User:</b> uid=0; IP=10.3.202.101; RIP:10.3.202.101 Request: /

# 应用项目关于 mysql 配置项
[root@master01 ~]# grep -nrw '123' /root/data/nfs/html/* | head -n 5
grep: /root/data/nfs/html/data/ipdata/ipv6wry.dat: binary file matches
/root/data/nfs/html/config/config_global.php:9:$_config['db'][1]['dbpw'] = '123';
/root/data/nfs/html/config/config_ucenter.php:9:define('UC_DBPW', '123');

sed -i 's/123/123456/g' /root/data/nfs/html/config/config_global.php
sed -i 's/123/123456/g' /root/data/nfs/html/config/config_ucenter.php

# 让 php 相关应用重启一下？ 直接删pod（其他还有更好的更新策略 -> php 里能够通过健康检查捕获到 mysql密码变更后没法正常访问）

# 测试验证
[root@master01 ~]# curl -I bbs.iproute.cn
HTTP/1.1 200 OK
```



任务6: Redis持久化管理

```shell
# 参考 04.redis_v1.yaml 文件, apply
## Output: mount.nfs: mounting 10.3.202.100:/root/data/nfs/redis/data failed, reason given by server: No such file or directory

mkdir -pv /root/data/nfs/redis/data

# 进入容器写入测试数据
kubectl exec -it $(kubectl get pods -l app=redis -o jsonpath='{.items[0].metadata.name}') -- redis-cli
> SET testkey "persistence_verified"
> SAVE

# 文件持久化
[root@master01 ~]# tree data/nfs/redis/data/
data/nfs/redis/data/
├── appendonlydir
│   ├── appendonly.aof.1.base.rdb
│   ├── appendonly.aof.1.incr.aof
│   └── appendonly.aof.manifest
└── dump.rdb

# 删除 Pod 触发重建后验证数据
kubectl delete pod $(kubectl get pods -l app=redis -o jsonpath='{.items[0].metadata.name}')
kubectl exec $(kubectl get pods -l app=redis -o jsonpath='{.items[0].metadata.name}') -- redis-cli GET testkey
persistence_verified
```



出现了异常问题

```shell
#   Warning  Failed     21s               kubelet            Failed to pull image "redis:latest": Error response from daemon: Get "https://registry-1.docker.io/v2/": net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)

# master01 & node01 & node02  /etc/docker/daemon.json 里关于 insecure-registries & registry-mirrors 删掉吧（p.iproute.cn:6443服务已经下线）

# 重启 docker & daemon-reload
[root@master01 resources]# systemctl daemon-reload
[root@master01 resources]# systemctl restart docker

[root@master01 ~]# kubectl get nodes
Error from server (Forbidden): nodes is forbidden: User "kubernetes-admin" cannot list resource "nodes" in API group "" at the cluster scope

# Jul 21 19:36:28 master01 cri-dockerd[1130076]: time="2025-07-21T19:36:28+08:00" level=error msg="Error deleting pod kube-system/calico-kube-controllers-558d465845-xh7sd from network {docker 14f017f83f007856d784e497c3ce2a4>

# kubelet -> cri-docker -> docker 
```



任务7：新增工作节点，添加1台node

通过有自动化重复性的工作，可以大幅提升效率，标准化的操作可以溯源不会因为个人误操作而产生预期外的问题

1. 单节点：LAMP 搭建
2. 单节点：docker 环境的部署
3. 多节点： nginx作为LB + keepalived + web服务
4. K8s集群的搭建：多节点部署

对于自动化处理重复工作的思路：shell（单个shell文件尽量不要超过100行）+ python（更复杂的逻辑，API接口的调用）+ ansible（规模化1000+ 基本能够cover住）+ python/go （平台化）

```bash
#!/bin/bash
CURRENT_DIR=$(dirname "$0")
ROOT_DIR=$(cd "$CURRENT_DIR/.." && pwd)

# 导入日志模块和检查工具模块
source $ROOT_DIR/logger.sh
source $ROOT_DIR/utils/utils.sh

# 导入基础安装脚本
source $ROOT_DIR/install/install_base.sh
run_base_install

# Check and execute join command
JOIN_SCRIPT="${ROOT_DIR}/join_command.sh"
if [ -f "$JOIN_SCRIPT" ]; then
    log_info "🔄 执行节点加入命令..."
    # Add containerd socket parameter to join command
    JOIN_CMD=$(cat "$JOIN_SCRIPT")
    JOIN_CMD="$JOIN_CMD --cri-socket unix:///var/run/cri-dockerd.sock"
    
    if eval "$JOIN_CMD"; then
        log_info "✅ 节点成功加入集群"
    else
        log_error "❌ 节点加入失败"
        exit 1
    fi

else
    log_warn "⚠️ 控制节点上执行: kubeadm token create --print-join-command"
fi

log_info "🏁 所有组件安装完成，Kubernetes节点就绪"
```



扩展：

k8s master 节点的高可用方案；

Keepalived + Load Balancer ： LB 可以是 LVS、Haproxy 或 Nginx，结合 keepalived 实现负载均衡高可用。
Load Balancer 服务 接受前台用户发送过来的 kubectl 等请求，再通过反向代理转发到后台的 Master 节点上面，
单节点的话，多台 Node 直接指向 一台Master 节点；而多Master集群结构中，Master 会指向 Load Balancer 服务，请求都来自负载均衡服务，所以LB要做高可用。
Master 的 Apiserver 都指向 Keepalived 的虚拟 IP上
Master 上通过 Apiserver 直接 操作 Node 节点上的 kubelet，不需要再通过 VIP 的负载均衡转发。Node 节点会由 Master 管理实现高可用。
首先 ETCD 集群实现 去中心化高可用（奇数台机器），通过 Raft 算法保持数据库数据一致性。



水平扩缩容：

HPA 通过监控预设的性能指标，周期性地检查当前指标值是否超过设定的阈值。若超过，则根据算法自动调整目标 Deployment、ReplicaSet 或 StatefulSet 的副本数量

hpa模板

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



垂直扩缩容
