# k8s安装到搭建LNMP架构

## 一、前置准备

Linux版本：RockyLinux9.4 

内核版本：Linux version 5.14.0-427.13.1.el9_4.x86_64 

单机编排，主机名：docker-server01 IP地址：192.168.173.100



### 操作系统环境初始化

```bash
#需要将防火墙firewalld改为iptables
systemctl stop firewalld/systemctl disabled firewalld
yum -y install iptables-services
systemctl start iptables
iptables -F
systemctl enable --now iptables

#禁用selinux
setenforce 0 / sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config

#设置时区
timedatectl set-timezone Asia/Shanghai

#关闭swap分区
swapoff -a / sed -i 's:/dev/mapper/rl-swap:#/dev/mapper/rl-swap:g' /etc/fstab

#安装ipvs
yum install -y ipvsadm

#开启路由转发
echo 'net.ipv4.ip_forward=1' >> /etc/sysctl.conf
sysctl -p

#加载 bridge
yum install -y epel-release
yum install -y bridge-utils

modprobe br_netfilter
echo 'br_netfilter' >> /etc/modules-load.d/bridge.conf
echo 'net.bridge.bridge-nf-call-iptables=1' >> /etc/sysctl.conf
echo 'net.bridge.bridge-nf-call-ip6tables=1' >> /etc/sysctl.conf

sysctl -p
```



#### 添加 docker-ce yum 源

```bash
# 中科大源(ustc)
sudo dnf config-manager --add-repo https://mirrors.ustc.edu.cn/docker-ce/linux/centos/docker-ce.repo
cd /etc/yum.repos.d
# 切换中科大源
sed -i 's#download.docker.com#mirrors.ustc.edu.cn/docker-ce#g' docker-ce.repo
# 安装 docker-c
yum -y install docker-ce
# 配置 daemon.
cat > /etc/docker/daemon.json <<EOF
{
  "data-root": "/data/docker",
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m",
    "max-file": "100"
  },
  "insecure-registries": ["iproute.cn:6443"],
  "registry-mirrors": ["https://iproute.cn:6443"]
}
EOF
mkdir -p /etc/systemd/system/docker.service.d

# 重启docker服务
systemctl daemon-reload && systemctl restart docker && systemctl enable docker
```



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

# 初始化主节点
kubeadm init\
 --apiserver-advertise-address=192.168.173.100\
 --image-repository registry.aliyuncs.com/google_containers\
 --kubernetes-version 1.29.2\
 --service-cidr=10.10.0.0/12\
 --pod-network-cidr=10.244.0.0/16\
 --ignore-preflight-errors=all\
 --cri-socket unix:///var/run/cri-dockerd.sock

# node 加入
kubeadm join 192.168.173.100:6443 --token jghzcm.mz6js92jom1flry0 \
        --discovery-token-ca-cert-hash sha256:63253f3d82fa07022af61bb2ae799177d2b0b6fe8398d5273098f4288ce67793  --cri-socket unix:///var/run/cri-dockerd.sock        

# work token 过期后，重新申请
kubeadm token create --print-join-command
```



### 部署网络插件

```bash
https://docs.tigera.io/calico/latest/getting-started/kubernetes/self-managed-onprem/onpremises #install-calico-with-kubernetes-api-datastore-more-than-50-nodes


curl https://raw.githubusercontent.com/projectcalico/calico/v3.26.3/manifests/calico-typha.yaml -o calico.yaml
    CALICO_IPV4POOL_CIDR    指定为 pod 地址

# 修改为 BGP 模式
# Enable IPIP
- name: CALICO_IPV4POOL_IPIP
  value: "Always"  #改成Off

kubectl apply -f calico-typha.yaml
kubectl get pod -A
```



### 修改kube-proxy 模式为 ipvs

```bash
# kubectl edit configmap kube-proxy -n kube-system
mode: ipvs

kubectl delete pod -n kube-system -l k8s-app=kube-proxy
```



## 二、docker-compose环境准备

###  yum安装docker-compese

```bash
[root@docker-server1 ~]# yum install epel-release.noarch -y
[root@docker-server1 ~]# yum install docker-compose.noarch -y
[root@docker-server1 ~]# docker-compose version 
docker-compose version 1.18.0, build 8dd22a9
docker-py version: 2.6.1
CPython version: 3.6.8
OpenSSL version: OpenSSL 1.0.2k-fips  26 Jan 2017
```

或者使用二进制安装，进入官网下载对应版本即可https://github.com/docker/compose/releases



#### 相关参数

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



### 启动单个容器

#### 编写docker-compose文件

```bash
[root@docker-server1 ~]# mkdir /opt/docker
[root@docker-server1 ~]# cd /opt/docker/
[root@docker-server1 docker]# vim docker-compose.yml
services:
  service-nginx:
    image: nginx
    container_name: nginx_web1
    ports:
      - "80:80"
      
 #如果要启动多个容器，只需要在docker-compose.yml文件追加要启动容器的配置即可
```



#### 启动容器

```bash
[root@docker-server1 docker]# docker-compose up -d
Creating docker_service-nginx_1 ... done
```



### 定义数据卷挂载

#### 创建数据卷目录和文件

```bash
[root@docker-server1 docker]# mkdir -p /data/nginx
[root@docker-server1 docker]# echo 'docker nginx' > /data/nginx/index.html
```

编辑配置文件，然后curl一下，进行访问测试

```bash
[root@docker-server1 docker]# curl localhost
docker nginx
```



#### 其他操作

```bash
重启单个容器
[root@docker-server1 docker]# docker-compose restart service-nginx 

重启所有容器
[root@docker-server1 docker]# docker-compose restart 

停止单个容器
[root@docker-server1 docker]# docker-compose stop service-nginx 

启动单个容器
[root@docker-server1 docker]# docker-compose start service-nginx 

停止所有容器
[root@docker-server1 docker]# docker-compose stop 

启动所有容器
[root@docker-server1 docker]# docker-compose start 
```



## 三、搭建LNMP架构

**使用docker-compose实现编排nginx+phpfpm+mysql容器，实现博客系统的部署**

### 编写配置文件

```yml
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



### 在`/opt/stack/lnmp/nginx/default.conf` 中写入nginx的配置文件

```bash
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



### 准备好php的探针

```php
<?php
    $dbhost = "mysql";
    $dbuser = "root";
    $dbpass = "123456";
    $db = "login";
    $conn = mysqli_connect($dbhost, $dbuser, $dbpass, $db) or exit("数据库连接失败！");
    echo "数据库连接成功";
?>
```

然后访问这个页面，发现php缺少mysqli的组件



### 编写`Dockerfile` 来对php镜像进行定制

```bash
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



#### 修改yaml配置文件，让docker-compose帮我们构建php的镜像

```yml
php:
    # 把images改成build
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
```



然后就可以使用wordpress搭建自己的博客了
