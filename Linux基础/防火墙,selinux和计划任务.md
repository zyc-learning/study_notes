# 防火墙，selinux和计划任务 

## 一、iptables

iptables过滤数据包的方式：
- 是否允许让 Internet 的封包进入 Linux 主机的某些 port
- 是否允许让某些来源 IP 的封包进入
- 是否允许让带有某些特殊标志( flag )的封包进入
- 分析硬件地址(MAC)来提供服务



### 规则表和规则链
一个表对应多个链，一个链对应多个规则



#### 五链
iptables命令中设置数据过滤或处理数据包的策略叫做规则，将多个规则合成一个链，叫规则链。 规则链则依据处理数据包的位置不同分类：

- PREROUTING：在进行路由判断之前所要进行的规则(DNAT/REDIRECT)
- INPUT：处理入站的数据包
- OUTPUT：处理出站的数据包
- FORWARD：处理转发的数据包
- POSTROUTING：在进行路由判断之后所要进行的规则(SNAT/MASQUERADE)



#### 四表
iptables中的规则表是用于容纳规则链，规则表默认是允许状态的，那么规则链就是设置被禁止的规则，而反之如果规则表是禁止状态的，那么规则链就是设置被允许的规则。

raw表：确定是否对该数据包进行状态跟踪 OUTPUT、PREROUTING
mangle表：是否为数据包设置标记，修改数据包的服务类型（较少使用）PREROUTING、POSTROUTING、INPUT、OUTPUT、FORWARD
nat表：修改数据包中的源、目标IP地址或端口（网络地址转换）PREROUTING、POSTROUTING、OUTPUT
filter表：确定是否放行该数据包（过滤数据包）INPUT、FORWARD、OUTPUT



在四表的顺序下按照五链的顺序进行过滤：
raw PREROUTING ->mangle PREROUTING -> nat PREROUTING -> mangle INPUT  - >... ->nat OUTPUT ->



### 基本选项和用法

```bash
iptables -A/D/R/ (add/delete/replace/[-t table名]) -j 规则
```

选项

| 参数         | 作用                                                         |
| ------------ | ------------------------------------------------------------ |
| -P           | 设置默认策略:iptables -P INPUT (DROP\ACCEPT)类似于黑白名单   |
| -F           | 清空规则链                                                   |
| -L           | 查看规则链                                                   |
| -A           | 在规则链的末尾加入新规则                                     |
| -I num       | 在规则链的头部加入新规则                                     |
| -X           | 删除用户自定义的空链或者指定的，未被其它任何规则引用的，无任何规则的链 |
| -D num       | 删除某一条规则                                               |
| -s           | 匹配来源地址IP/MASK，加叹号"!"表示除这个IP外。               |
| -d           | 匹配目标地址                                                 |
| -i 网卡名称  | 匹配从这块网卡流入的数据                                     |
| -o 网卡名称  | 匹配从这块网卡流出的数据                                     |
| -p           | 匹配协议,如tcp,udp,icmp                                      |
| --dports num | 匹配目标端口号                                               |
| --sports num | 匹配来源端口号                                               |

**为了防止firewalld与iptables的冲突，建议在学习iptables的时候，先关闭firewalld**



#### iptables规则说明

规则：根据指定的匹配条件来尝试匹配每个经流“关卡”的报文，一旦匹配成功，则由规则后面指定的处理动作进行处理

- 匹配条件
  - 基本匹配条件：sip（源）、dip（目的）
  - 扩展匹配条件：sport（源）、dport（目的）
  - 扩展条件也是条件的一部分，只不过使用的时候需要用`-m`参数声明对应的模块
- 处理动作
  - accept：接受
  - drop：丢弃
  - reject：拒绝
  - snat：源地址转换，解决内网用户同一个公网地址上上网的问题
  - masquerade：是snat的一种特殊形式，使用动态的、临时会变的ip上
  - dnat：目标地址转换
  - redirect：在本机作端口映射
  - log：记录日志，/var/log/messages文件记录日志信息，然后将数据包传递给下一条规则



### iptables高级用法

```bash
# iptableschang'y
iptables -t table -A/D/R chain <规则，如：-s sourceaddr -j -p >
-t table   #指定表 raw、mangle、nat、filter（默认）
-A：#添加规则，后面更上你要添加到哪个链上
-D：#删除规则
-R：#对具体某一个num的规则进行修改
-j：#设置处理策略 accept：接受、drop：丢弃、reject：拒绝
-s：#指定数据包的来源地址
-p：# 指定数据包协议类型 tcp、udp、icmp
```



#### 增加规则

```bash
[root@localhost ~]# iptables -A [tables] -s -p -j
```



#### 查看方法

```bash
[root@localhost ~]# iptables [-t tables] [-L] [-nv]
参数：
-t    后面接table,例如nat或filter，如果省略，默认显示filter
-L    列出目前的table的规则
-n    不进行IP与主机名的反查，显示信息的速度会快很多
-v    列出更多的信息，包括封包数，相关网络接口等
--line-numbers 显示规则的序号
```

- 查看具体每个表的规则
  - policy：当前链的默认策略，当所有规则都没有匹配成功时执行的策略
  - packets：当前链默认策略匹配到的包的数量
  - bytes：当前链默认策略匹配到的包的大小
  - pkts：对应规则匹配到的包数量
  - bytes：对应规则匹配到的包大小
  - target：对应规则执行的动作
  - prot：对应的协议，是否针对某些协议应用此规则
  - opt：规则对应的选项
  - in：数据包由哪个接口流入
  - out：数据包由哪个接口流出
  - source：源ip地址
  - distination：目的ip地址

```bash
iptables -A INPUT -p tcp --dport 80 -j DROP
#让80端口发送的数据包被丢弃

iptables -vnL		
# 查看默认的filter表中所有的链上的规则

iptables -vnL INPUT --line-numbers
# 查看默认的filter表中INPUT链上规则并且列出序号

iptables -t net -vnL --line-numbers
# 查看nat表上的所有链的规则，有序号
```



#### 删除规则

先查找到你想要删除的规则

- 通过num序号进行删除

```bash
删除input链上num为1的规则
[root@localhost ~]# iptables -D INPUT 1
```

- 通过规则匹配删除

```bash
[root@localhost ~]# iptables -D INPUT -p tcp --dport 80 -j DROP
```

- 清空所有的规则

```bash
[root@localhost ~]# iptables -F
```



#### 修改规则

方案一：通过`iptables -D`删除原有的规则后添加新的规则

方案二：通过`iptables -R`可以对具体某一个num的规则进行修改

```bash
# 修改端口的为8080
[root@localhost ~]# iptables -R INPUT 1 -p tcp --dport 8080 -j ACCEPT
```



#### 保存规则

```bash
[root@localhost ~]#service iptables save   # 把配置保存在/etc/sysconfig/ip
```



### 自定义链

iptables中除了系统自带的五个链之外，还可以自定义链，来实现将规则进行分组，重复调用的目的

```bash
#添加WEB_CHAIN自定义链，并且查看
[root@localhost ~]# iptables -t filter -N web_chain
```

```bash
# 可以通过-E参数修改自定义链
[root@localhost ~]# iptables -t filter -E web_chain WEB_CHAIN
```

添加规则类似自带的系统链

```bash
# 5. 将自定义链关联到系统链上才能使用
# 因为数据包只会经过上面讲过的五个系统链，不会经过我们的自定义链，所以需要把自定义链关联到某个系统链上
# 我们允许来自IP：192.168.88.1的访问，随后拒绝其他所有的访问，这样，只有192.168.88.1的主机可以访问这个网站
[root@localhost ~]# iptables -t filter -A INPUT -s 192.168.88.1 -j WEB_CHAIN       #来自于192.168.88.1进入INPUT链时，调用的策略是“WEB_CHAIN”.
[root@localhost ~]# iptables -t filter -A INPUT -p tcp -m multiport --dports 80,443 -j DROP
```

```bash
# 删除自定义链
# 先清空自定义链上的规则
[root@localhost ~]# iptables -t filter -F WEB_CHAIN
# 然后通过-X选项删除自定义链
[root@localhost ~]# iptables -t filter -X WEB_CHAIN
```



### 其他用法（模块）

- tcp/udp
  - --dport：指定目的端口
  - --sport：指定源端口
- iprange：匹配报文的源/目的地址所在范围
  - --src-range
  - --dst-range

- string：指定匹配报文中的字符串
  - --algo：指定匹配算法，可以是bm/kmp
    - bm：Boyer-Moore
    - kmp：Knuth-Pratt-Morris
  - --string：指定需要匹配的字符串
  - --from offset：开始偏移
  - --to offset：结束偏移

- time：指定匹配报文的时间
  - --timestart
  - --timestop
  - --weekdays
  - --monthdays
  - --datestart
  - --datestop

- connlimit：限制每个ip连接到server端的数量，不需要指定ip默认就是针对每个ip地址,可防止Dos(Denial of Service，拒绝服务)攻击
  - --connlimit-above：最大连接数

- limit：对报文到达的速率继续限制，限制包数量
  - 10/second
  - 10/minute
  - 10/hour
  - 10/day
  - --limit-burst：空闲时可放行的包数量，默认为5，前多少个包不限制

- 指定TCP匹配扩展

```bash
使用 --tcp-flags 选项可以根据tcp包的标志位进行过滤。
[root@localhost ~]# iptables -A INPUT -p tcp --tcp-flags SYN,FIN,ACK SYN
[root@localhost ~]# iptables -A FROWARD -p tcp --tcp-flags ALL SYN,ACK
上实例中第一个表示SYN、ACK、FIN的标志都检查，但是只有SYN匹配。第二个表示ALL（SYN，ACK，FIN，RST，URG，PSH）的标志都检查，但是只有设置了SYN和ACK的匹配。
[root@localhost ~]# iptables -A FORWARD -p tcp --syn
选项--syn相当于"--tcp-flags SYN,RST,ACK SYN"的简写。
```

- state模块：用于针对tcp连接进行限制，较耗资源

```bash
[root@localhost ~]# iptables -A INPUT -m 模块名 --state 状态
参数：
[root@localhost ~]# iptables -A INPUT -p tcp --dport 22 -m state --state NEW -j ACCEPT    
-m state --state NEW：使用状态模块，匹配数据包的连接状态为NEW，即新建的连接。
- 该规则的作用是允许进入系统的TCP数据包，目标端口为22（SSH服务），且连接状态为NEW（新建的连接）。这个规则用于允许建立新的SSH连接，而拒绝已经建立的或结束的连接

[root@localhost ~]# iptables -A INPUT -m mac --mac-source 00:0C:29:56:A6:A2 -j ACCEPT
-m mac --mac-source 00:0C:29:56:A6:A2：使用MAC模块，匹配源MAC地址为00:0C:29:56:A6:A2的数据包
- 该规则的作用是允许源MAC地址为00:0C:29:56:A6:A2的数据包进入系统。这个规则可以用于根据MAC地址限制网络访问，只允许特定的MAC地址通过防火墙进入系统
```



### 规则的保存与恢复

- 用规则文件保存各规则，开机后导入该规则文件中的规则

```bash
# 保存到文件中
[root@localhost ~]# iptables-save > /etc/sysconfig/iptables-config
# 从文件中导入规则
[root@localhost ~]# iptables-restore < /etc/sysconfig/iptables-config
```

- 安装`iptables-services`，在centos7和centos8上，通过该服务来帮助我们自动管理iptables规则。

```bash
[root@localhost ~]# systemctl stop iptables-services
[root@localhost ~]# systemctl disable iptables-services
```



### NAT（地址转换协议）

 网络地址转换 NAT（Network Address Translation），被广泛应用于各种类型 Internet 接入方式和各种类型的网络中。原因很简单，NAT 不仅完美地解决了 IP 地址不足的问题，而且还能够有效地避免来自网络外部的攻击，隐藏并保护网络内部的计算机。



## 二、 firewalld

支持划分区域zone,每个zone可以设置独立的防火墙规则

- 流量的规则匹配顺序
  - 先根据数据包中源地址，将其纳为某个zone
  - 纳为网络接口所属zone
  - 纳入默认zone，默认为public zone,管理员可以改为其它zone
- firewalld中常用的区域名称及策略规则

| 区域     | 默认策略规则                                                 |
| -------- | ------------------------------------------------------------ |
| public   | 默认区域,对应普通网络环境。允许选定的连接,拒绝其他连接       |
| drop     | 所有传入的网络包都会被直接丢弃,不会发送任何响应              |
| block    | 所有传入的网络包会被拒绝,并发送拒绝信息                      |
| external | 用于路由/NAT流量的外部网络环境。与public类似,更适用于网关、路由器等处理外部网络流量的设备 |
| dmz      | 隔离区域,只允许选定的连接，适用于部署公开服务的网络区域,如 Web 服务器，可以最大限度地降低内部网络的风险 |
| work     | 适用于工作环境，开放更多服务,如远程桌面、文件共享等。比 public 区域更加信任 |
| home     | 适用于家庭环境，开放更多服务，比如默认情况下会开放一些如:3074端口(Xbox)、媒体、游戏数据等待 |
| trusted  | 信任区域,允许所有连接                                        |



### 管理工具-firewall-cmd

firewall-cmd命令常用的选项

| 参数                          | 作用                                                 |
| ----------------------------- | ---------------------------------------------------- |
| --get-default-zone            | 查询默认的区域名称                                   |
| --set-default-zone=<区域名称> | 设置默认的区域，使其永久生效                         |
| --get-zones                   | 显示可用的区域                                       |
| --get-services                | 显示预先定义的服务                                   |
| --get-active-zones            | 显示当前正在使用的区域与网卡名称                     |
| --add-source=                 | 将源自此IP或子网的流量导向指定的区域                 |
| --remove-source=              | 不再将源自此IP或子网的流量导向某个指定区域           |
| --add-interface=<网卡名称>    | 将源自该网卡的所有流量都导向某个指定区域             |
| --change-interface=<网卡名称> | 将某个网卡与区域进行关联                             |
| --list-all                    | 显示当前区域的网卡配置参数、资源、端口以及服务等信息 |
| --list-all-zones              | 显示所有区域的网卡配置参数、资源、端口以及服务等信息 |
| --add-service=<服务名>        | 设置默认区域允许该服务的流量                         |
| --add-port=<端口号/协议>      | 设置默认区域允许该端口的流量                         |
| --remove-service=<服务名>     | 设置默认区域不再允许该服务的流量                     |
| --remove-port=<端口号/协议>   | 设置默认区域不再允许该端口的流量                     |
| --reload                      | 让“永久生效”的配置规则立即生效，并覆盖当前的配置规则 |
| --panic-on                    | 开启应急状况模式                                     |
| --panic-off                   | 关闭应急状况模式                                     |
| --permanent                   | 设定的当前规则保存到本地，下次重启生效               |



### 简单用法（初体验）

```bash
# 查看firewalld是否启用，可以看到当前是出于active的状态
[root@localhost ~]# systemctl status firewalld
# 查看当前所在的区域
[root@localhost ~]# firewall-cmd --get-default-zone

# 查看当前网卡所在的区域
[root@localhost ~]# firewall-cmd --get-zone-of-interface=ens33

# 查询public区域是否允许请求SSH或者HTTP协议的流量
[root@localhost ~]# firewall-cmd --zone=public --query-service=ssh
# 查询public区域是否允许请求SSH或者HTTP协议的流量
[root@localhost ~]# firewall-cmd --zone=public --query-service=ssh
yes
[root@localhost ~]# firewall-cmd --zone=public --query-service=http
no
```

修改firewalld策略,使得我们可以访问到已部署的网站

```bash
- 方法一：针对服务协议类型进行放行
# 临时放行
[root@localhost ~]# firewall-cmd --zone=public --add-service=http
# 可以加上--permanent实现永久放行
[root@localhost ~]# firewall-cmd --permanent --zone=public --add-service=http

- 方法二：针对服务具体访问请求的端口号进行放行
[root@localhost ~]# firewall-cmd --zone=public --add-port=80/tcp
success
# 同样也可以加上--permanent实现永久放行
```

取消放行

```bash
[root@localhost ~]# firewall-cmd --zone=public --remove-service=http --permanent
```



### 其他用法

- 把firewalld服务的当前默认区域设置为public

```bash
[root@localhost ~]# firewall-cmd --set-default-zone=public
```

- 查看某个区域的详细配置

```bash
[root@localhost ~]# firewall-cmd --zone=public --list-all
```

- 将firewalld 切换到紧急模式(panic mode)。这种模式下,firewalld 会采取一种非常严格的防护策略:
  1. 所有传入的网络连接都会被拒绝,centos系统除了 ssh 和 ICMP 回送(ping)请求。rockyLinux系统全部拒绝
  2. 所有区域的防火墙规则都会被重置为默认的严格策略。
  3. 所有区域的默认策略都会被设置为 DROP

```bash
[root@localhost ~]# firewall-cmd --panic-on
# 取消紧急模式
[root@localhost ~]# firewall-cmd --panic-off
```

- 临时生效/永久生效

```bash
# 临时生效
[root@localhost ~]# firewall-cmd --zone=public --add-service=http

# 永久生效，加了--permanent所进行的更改，会被写入到配置文件中，下次开机任然生效
[root@localhost ~]# firewall-cmd --permanent --zone=public --add-service=http

# 使得最近对防火墙规则变更或者所作的修改生效
[root@localhost ~]# firewall-cmd --reload
[root@localhost ~]# firewall-cmd --zone=public --query-service=http

-配置文件路径
/etc/firewalld/zones/public.xml
/etc/firewalld/direct.xml
/etc/firewalld/firewalld.conf
```

例子： 把firewalld服务中请求HTTPS协议的流量设置为永久拒绝，并立即生效

```bash
[root@localhost ~]# firewall-cmd --zone=public --remove-service=https --permanent
[root@localhost ~]# firewall-cmd --reload
```

- 把在firewalld服务中访问8080和8081端口的流量策略设置为允许，当前生效

```bash
# 可以指定要开放端口的范围
[root@localhost ~]# firewall-cmd --zone=public --add-port=8080-8081/tcp
[root@localhost ~]# firewall-cmd --zone=public --list-ports
```



### 端口转发

通过firewalld，将用户访问服务器上6666端口的流量转发到22端口上，实现本地端口映射。我们就可以使用6666端口来远程连接到服务器

```bash
[root@localhost ~]# firewall-cmd --permanent --add-forward-port=port=6666:proto=tcp:toport=22
[root@localhost ~]# firewall-cmd --reload
```



### firewalld富规则

富规则是 firewalld 提供的一种灵活且强大的防火墙规则配置方式。与简单的端口和服务规则不同,富规则支持更复杂的匹配条件和操作。

使用富规则,可以实现复杂的防火墙策略,例如:

- 允许特定 IP 地址访问某个端口
- 拒绝特定 IP 地址访问某个服务
- 限制某个网段的连接频率
- 转发某个端口到另一个端口

**富规则的配置方法：**

```bash
firewall-cmd --permanent --add-rich-rule='<rich_rule_definition>'
```

需要注意的是,富规则的语法比较复杂,使用时务必仔细检查,以免引入安全隐患。同时,**在修改完成后记得执行 `firewall-cmd --reload` 命令,让更改生效**

其中rule部分的语法如下：

```bash
rule [family="<family>"] [source address="<address>"][source port="<port>"] [destination address="<address>"][destination port="<port>"] [protocol value="<protocol>"] [icmp-block-inversion][forward-port port="<port>" protocol="<protocol>" to-port="<port>"][masquerade][log [prefix="<prefix>"] [level="<level>"] [limit value="<value>"] [accept][reject][drop]
```

其中各个选项的含义如下:

- `family`: 指定地址族,可以是 `ipv4` 或 `ipv6`
- `source`/`destination`: 指定源/目标地址
- `port`: 指定端口号
- `protocol`: 指定协议,如 `tcp`、`udp` 等
- `icmp-block-inversion`: 反转 ICMP 阻止规则
- `forward-port`: 端口转发规则
- `masquerade`: 启用地址伪装
- `log`: 日志记录规则,可指定前缀、日志级别、限速
- `accept`/`reject`/`drop`: 动作,分别表示允许、拒绝、丢弃

例子：

允许某个IP地址访问系统中的Web网站服务

```bash
[root@localhost ~]# firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.88.1" port port="80" protocol="tcp" accept'
[root@localhost ~]# firewall-cmd --reload
```

```bash
#限制某个区域内的 SSH 连接次数
[root@localhost ~]# firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.88.0/24" port port="22" protocol="tcp" limit value="3/m" accept'
[root@localhost ~]# firewall-cmd --reload[root@localhost ~]# firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.88.0/24" port port="22" protocol="tcp" limit value="3/m" accept'
[root@localhost ~]# firewall-cmd --reload
```



## 三、服务访问控制

**RockyLinux9已不再支持**

TCP Wrappers是一种用于网络服务访问控制的工具，它使用配置文件中的规则来决定是否允许或拒绝对特定网络服务的访问。控制列表由两个主要文件组成：/etc/hosts.allow和/etc/hosts.deny。这些文件包含服务和客户端的规则，用于控制服务的访问权限

TCP Wrappers服务的控制列表文件中常用的参数

| 客户端类型     | 示例                       | 满足示例的客户端列表              |
| -------------- | -------------------------- | --------------------------------- |
| 单一主机       | 192.168.10.10              | IP地址为192.168.10.10的主机       |
| 指定网段       | 192.168.10.                | IP段为192.168.10.0/24的主机       |
| 指定网段       | 192.168.10.0/255.255.255.0 | IP段为192.168.10.0/24的主机       |
| 指定主机名称   | www.eagleslab.com          | 主机名称为www.eagleslab.com的主机 |
| 指定所有客户端 | ALL，*                     | 所有主机全部包括在内              |

在配置TCP Wrappers服务时需要遵循两个原则：

- 编写拒绝策略规则时，填写的是服务名称，而非协议名称；
- 建议先编写拒绝策略规则，再编写允许策略规则，以便直观地看到相应的效果。

**/etc/hosts.deny：**该文件包含拒绝访问网络服务的规则。如果没有在hosts.allow文件中找到允许访问的规则，TCP Wrappers将检查hosts.deny文件以确定是否拒绝访问。

**/etc/hosts.allow：**该文件包含允许访问网络服务的规则。每个规则占据一行，有两个主要部分：服务和允许访问的客户端。



## 四、SELinux安全子系统

seli主要限制进程能够访问的最小满足正常运行的资源

SELinux 是一个强大的访问控制机制,它建立在 Linux 内核之上,为系统提供了更细粒度的安全策略控制。与传统的基于用户/组的访问控制不同,SELinux 采用基于角色(role)和类型(type)的强制访问控制(Mandatory Access Control, MAC)。

SELinux 的主要特点包括:

1. 基于策略的安全访问控制:SELinux 通过预先定义的安全策略来控制系统进程和资源的访问权限,而不是依赖于用户/组的身份
2. 最小特权原则:SELinux 遵循"最小特权"的安全原则,即只授予程序运行所需的最小权限,大大降低了系统被攻击者利用的风险
3. 灵活的策略配置:SELinux 提供了丰富的策略配置选项,可以根据系统的具体需求进行定制和调整
4. 审计能力:SELinux 内置了强大的审计日志记录功能,可以帮助管理员快速发现和分析系统安全事件

SELinux 的主要工作模式包括:

- Enforcing 模式:完全执行 SELinux 策略,阻止任何未经授权的访问行为
- Permissive 模式:只记录违反 SELinux 策略的行为,但不会阻止它们
- Disabled 模式:完全关闭 SELinux 功能



### 调整SELinux的模式

#### **临时调整：**

```bash
# 临时调整为Permissive
[root@localhost ~]# setenforce 0
[root@localhost ~]# getenforce
Permissive
[0]为Permissive模式，只记录行为，不阻止
[1]为Enforcing模式
```



#### **永久调整：**

通过编辑配置文件`/etc/selinux/config`中的SELINUX字段来更改SELinux的模式

```bash
[root@localhost ~]# vim /etc/selinux/config 
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=enforcing    #修改此处
# SELINUXTYPE= can take one of three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected. 
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```



#### 自主访问控制（DAC）

在没有使用 SELinux 的操作系统中，决定一个资源是否能被访问的因素是：某个资源是否拥有对应用户的权限（读、写、执行）

只要访问这个资源的进程符合以上的条件就可以被访问

而最致命问题是，root 用户不受任何管制，系统上任何资源都可以无限制地访问

这种权限管理机制的主体是用户，也称为自主访问控制（DAC）



#### 强制访问控制（MAC）

在使用了 SELinux 的操作系统中，决定一个资源是否能被访问的因素除了上述因素之外，还需要判断每一类进程是否拥有对某一类资源的访问权限

这样一来，即使进程是以 root 身份运行的，也需要判断这个进程的类型以及允许访问的资源类型才能决定是否允许访问某个资源。进程的活动空间也可以被压缩到最小

即使是以 root 身份运行的服务进程，一般也只能访问到它所需要的资源。即使程序出了漏洞，影响范围也只有在其允许访问的资源范围内。安全性大大增加

这种权限管理机制的主体是进程，也称为强制访问控制（MAC）



SELinux 会为 Apache 进程设置专门的安全上下文(context),限制它只能访问特定的资源。比如:

- Apache 进程的安全上下文是 `httpd_t`
- Apache 网页文件的安全上下文是 `httpd_sys_content_t`
- Apache 日志文件的安全上下文是 `httpd_log_t`

```bash
# 查看Apache服务创建的工作目录和自己手动创建的目录的区别
[root@localhost ~]# ls -Zd /var/www/html/
drwxr-xr-x. root root system_u:object_r:httpd_sys_content_t:s0 /var/www/html/
[root@localhost ~]# ls -Zd /html/
drwxr-xr-x. root root unconfined_u:object_r:default_t:s0 /html/

- 会发现由Apache创建的工作目录 /var/www/html 具有一个httpd_sys_content_t的标签，这样，SELinux就可以通过在内核中对于这个带有这个标签的文件进行限制了
```



**字段解释：**

普通目录：unconfined_u:object_r:user_home_dir_t:s0

- 用户身份(user): `unconfined_u`
- 角色(role): `object_r`
- 类型(type): `user_home_dir_t`
- 敏感度级别(sensitivity level): `s0`

Apache工作目录：system_u:object_r:httpd_sys_content_t:s0

- 用户身份(user): `system_u`
- 角色(role): `object_r`
- 类型(type): `httpd_sys_content_t`
- 敏感度级别(sensitivity level): `s0`



### semanage

用于管理SELinux策略的一个工具

```bash
# 安装semanage
[root@localhost ~]# yum install -y policycoreutils-python
```

**用法：**

```
semanage [选项] [文件]
```

**基本选项：**

- -l参数用于查询；
- -a参数用于添加；
- -m参数用于修改；
- -d参数用于删除。

例子：更改httpd网站的默认工作目录，检查能否访问，如果不能访问，则为其添加`httpd_sys_content_t`策略再次访问测试：

```bash
1. 部署httpd
2. 更改httpd的配置文件,将工作目录变成/data/html
[root@localhost ~]# vim /etc/httpd/conf/httpd.conf
DocumentRoot "/data/html"
# Relax access to content within /var/www.
#
<Directory "/var/www">
    AllowOverride None
    # Allow open access:
    Require all granted
</Directory>

# Further relax access to the default document root:
<Directory "/data/html">
3. 在工作目录中添加一个index.html文件做为网站首页
[root@localhost ~]# echo "<h1> hello world </h1>" > /data/html/index.html

4. 浏览完访问192.168.88.136测试，发现访问被拒绝，提示403Forbidden
- 这是因为新工作目录/data/html 没有携带httpd_sys_content_t标签，所以httpd进程受到SELinux限制，无法访问该资源

4. 可以向网站新的工作目录中新添加一条SELinux安全上下文，让这个目录以及里面的所有文件能够被httpd服务程序所访问到
[root@localhost ~]# semanage fcontext -a -t httpd_sys_content_t /data/html
[root@localhost ~]# semanage fcontext -a -t httpd_sys_content_t /data/html/*

#使用restorecon命令来检查SELinux上下文并且递归更新
[root@localhost ~]# restorecon -Rv /data/html


5. 再次访问测试，结果访问成功
```



#### 查看跟Apache（httpd）服务相关的所有标签

使用getsebool命令查询并过滤出所有与HTTP协议相关的安全策略。其中，off为禁止状态，on为允许状态。

```bash
[root@localhost ~]# getsebool -a | grep http
```



## 五、 计划任务

### 单次调度执行at

安装at软件包：

```
yum install -y at
```

启动at服务：

```bash
[root@localhost ~]# systemctl start atd
[root@localhost ~]# systemctl enable atd        # 开机自启动
```

语法结构

```
at [options] time
```

选项：

- -f：指定包含具体指令的任务文件
- -q：指定新任务的队列名称
- -l：显示待执行任务的列表
- -d：删除指定的待执行任务

或者也可以使用一下命令查看和删除

`atq`：查看待执行的任务

`atrm`：通过序号删除任务



#### timespec

```bash
  now +5min            # 从现在开始5分钟后
  teatime tomorrow  # 明天的下午16：00
  noon +4 days        # 4天后的中午
  11:20 AM            # 早上11：20
  .....
  .....

# 对于at任务的时间写法有很多，不需要记忆，用到的时候查询一下就行
```

例一：5分钟后创建一个文件

```bash
[root@localhost ~]# at now +5min
at> touch /root/file2
at> <EOT>            # 按ctrl+D 结束任务编辑，最后显示<EOT>
job 1 at Fri May 31 21:03:00 2024
[root@localhost ~]# at -l        # 查看任务
1       Fri May 31 21:03:00 2024 a root
```

例二：通过文件指定任务

```bash
[root@localhost ~]# vim at.jobs
useradd testuser
[root@localhost ~]# at -l
[root@localhost ~]# at now +5min -f at.jobs
job 2 at Fri May 31 21:03:00 2024
[root@localhost ~]# at -l
2       Fri May 31 21:03:00 2024 a root
```



### 循环调度执行cron

计划任务（Cron）是一种强大的工具，可以自动执行预定的任务。它非常适合定期运行脚本、备份数据、清理临时文件等一系列重复性任务。例如：管理系统的日常任务的调度。

crontab：是cron服务提供的管理工具

**检查cron服务有没有启动：**

```bash
[root@localhost ~]# systemctl status crond.service
# 启动
systemctl start crond.service
```



#### cron的基本语法

cron命令的基本语法如下：

```text
crontab [-l | -r | -e | -n | -m]
```

选项：

`-l`：列出当前用户的定时任务。

`-r`：删除当前用户的定时任务。

`-e`：编辑当前用户的定时任务。

`-n`：检查定时任务是否可用。

`-m`：发送类似于电子邮件的消息，用于通知定时任务执行的结果。

```bash
[root@localhost ~]# crontab -l        # 列出当前用户所有计划任务
[root@localhost ~]# crontab -r        # 删除当前用户计划任务
[root@localhost ~]# crontab -e        # 编辑当前用户计划任务
管理员可以使用 -u username,去管理其他用户的计划任务
[root@localhost ~]# vim /etc/cron.deny    # 这个文件中加入的用户名无法使用cron
```



### cron任务的配置

每个用户可以通过`crontab`命令来编辑自己的任务计划。要编辑cron任务，可以使用`crontab -e`命令。

#### 编辑cron任务

要编辑cron任务计划，可以使用以下命令打开cron编辑器：

```bash
crontab -e
```

#### 配置cron任务的格式

cron任务的格式如下：

```bash
分 时 日 月 星期 命令
* 表示任何数字都符合
*/5 * * * * date >> date.txt    # 每隔5分钟执行一次
0 2 1,4,6 * * date >> date.txt    # 每月1号，4号，6号的2点
0 2 5-9 * * date >> date.txt        # 每月5-9号的2点
其中，第一个字段表示分钟（0-59），第二个字段表示小时（0-23），第三个字段表示天（1-31），第四个字段表示月份（1-12），第五个字段表示星期（0-7）。第六个字段是要执行的命令或脚本。
```



#### 查看任务存放位置

```bash
# 位置：/var/spool/cron/root
[root@localhost ~]# cat /var/spool/cron/root
```



### cron系统任务

- 临时文件的清理`/tmp` `/var/tmp`
- 系统信息的采集 `sar`
- 日志的轮转(切割) `lgrotate`
- 通常不是由用户定义
- 文件的位置

```bash
[root@localhost ~]# vim /etc/crontab    # 默认没有定义任何计划任务
[root@localhost ~]# ls /etc/cron.d        # 定义的计划任务每个小时会执行
0hourly  sysstat
[root@localhost ~]# cat /etc/cron.d/0hourly 
```

- crond仅仅会执行每小时定义的脚本 `/etc/cron.hourly`

```bash
[root@localhost ~]# ls /etc/cron.hourly/
[root@localhost ~]# cat /etc/cron.hourly/0anacron
/usr/sbin/anacron -s        # anacron是用来检查是否有错过的计划任务需要被执行
```



## 六、日志管理

Linux系统和许多程序会产生各种错误信息、告警信息和其他的提示信息，这些各种信息都应该记录到日志文件中。Linux系统日志对管理员来说，是了解系统运行的主要途径，因此需要对Linux日志系统有详细的了解。

常见的日志文件

|      日志文件       |                        解释                         |
| :-----------------: | :-------------------------------------------------: |
|  /var/log/messages  |                   系统主日志文件                    |
|   /var/log/secure   |                记录认证、安全的日志                 |
|  /var/log/maillog   |                  跟邮件postfix相关                  |
|    /var/log/cron    |               crond、at进程产生的日志               |
|   /var/log/dmesg    |        记录系统启动时加载的硬件相关信息日志         |
|  /var/log/yum.log   |                      yum的日志                      |
| /var/log/mysqld.log |                      MySQL日志                      |
|   var/log/xferlog   |                 和访问FTP服务器相关                 |
|  /var/log/boot.log  |              系统启动过程日志记录存放               |
|    /var/log/wtmp    |      当前登录的用户(可以直接在命令行输入w查看)      |
|  /var/log/lastlog   | 所有用户的登录情况(可以直接在命令行输入lastlog查看) |

### 查看日志

有多种方法可以查看日志，可以通过cat、tail等命令来查看

但是往往日志文件中内容都是非常多的，所以通过cat查看不是很直观。这个时候我们可以配合一些其他过滤工具来过滤日志里面的内容。如：grep，awk等等。

#### 例一：查看message日志中关于ens33网卡的信息

```bash
[root@localhost ~]# cat /var/log/messages | grep ens33
```



#### 例二：统计远程登录信息

```bash
[root@localhost ~]# cat /var/log/secure-20240530 | grep Accepted
```



### 日志系统-rsyslogd

#### 处理日志的进程

rsyslogd：记录绝大部分日志记录，和系统操作有关，安全，认证sshd,su，计划任务at,cron

`httpd/nginx/mysql`等等应用可以以自己的方式记录日志

```bash
[root@localhost ~]# ps aux |grep rsyslogd
```



#### 配置文件

```bash
[root@localhost ~]# rpm -qc rsyslog
/etc/logrotate.d/syslog        # 日志轮转(切割)相关
/etc/rsyslog.conf            # rsyslogd的主配置文件
/etc/sysconfig/rsyslog        # rsyslogd相关文件
[root@localhost ~]# vim /etc/rsyslog.conf
# 告诉rsyslogd进程 哪个设备(facility)，关于哪个级别的信息，以及如何处理
```



#### 日志类型(facility)

| 序号  |   Facility    |                             解释                             |
| :---: | :-----------: | :----------------------------------------------------------: |
|   0   | kern (kernel) |    Linux内核产生的信息，大部分是硬件检测和内核功能的启用     |
|   1   |     user      | 用户层级产生的信息，例如后边介绍的通过`logger`产生的日志信息 |
|   2   |     mail      |                    所有邮件收发的相关信息                    |
|   3   |    daemon     |                      系统服务产生的信息                      |
|   4   |     auth      |  与认证、授权相关的信息，如`login`、`ssh`、`su`等产生的信息  |
|   5   |    syslog     |                   `syslogd`服务产生的信息                    |
|   6   |      lpr      |                        打印相关的信息                        |
|   7   |     news      |                      新闻群组相关的信息                      |
|   8   |     uucp      |   Unix to Unix Copy Protocol 早期Unix系统间的数据交换协议    |
|   9   |     cron      |        周期性计划任务程序，如`cron`、`at`等产生的信息        |
|  10   |   authpriv    |  与auth类似，但记录的多为帐号相关的信息，如pam模块的调用等   |
| 16~23 | local0~local7 |      保留给本地用户使用的日志类型，通常与终端交互相关。      |



#### 日志级别

| 级别(日志重要级别) |                     解释                     |
| :----------------: | :------------------------------------------: |
|     LOG_EMERG      | 紧急，致命，服务无法继续运行，如配置文件丢失 |
|     LOG_ALERT      |    报警，需要立即处理，如磁盘空间使用95%     |
|      LOG_CRIT      |                   致命行为                   |
|   LOG_ERR(error)   |                   错误行为                   |
|    LOG_WARNING     |                   警告信息                   |
|     LOG_NOTICE     |                     普通                     |
|      LOG_INFO      |                   标准信息                   |
|     LOG_DEBUG      |      调试信息，排错才开，一般不建议使用      |



#### 例子：远程管理日志

两台服务器（server1(生产服务器)和server2(日志服务器)可以通信）

通过配置rsyslog，使得server1上关于ssh连接的日志发送到server2日志服务器上保存，在server2上可以查看到server1上的ssh日志

首先要关闭server1和2的firewalld



**server1上配置：**

1.编辑`/etc/rsyslog.conf`文件

```bash
# 在最下面添加如下字段
[root@localhost ~]# vim /etc/rsyslog.conf
:msg,contains,"sshd" @192.168.88.138:514

#=====字典解释=====
- msg 要发送的消息
- contains,"sshd" 过滤器，过滤所有跟sshd有关的日志
- @192.168.88.138:514  要发送到日志服务器的地址，其中@表示UDP，@@表示TCP,514端口是默认的远程日志端口
```

2.重启rsyslog服务

```bash
[root@localhost ~]# systemctl restart rsyslog
```



**server2上配置：**

修改配置文件

```bash
[root@localhost ~]# vim /etc/rsyslog.conf
# Provides UDP syslog reception
$ModLoad imudp                # 加载imudp模块，启用对UDP网络接口的支持
$UDPServerRun 514             # 用于通信的端口

$ModLoad imtcp                # tcp
$InputTCPServerRun 514        

:msg,contains,"sshd" /var/log/remote_ssh.log        # 任意位置添加规则，

# 重启rsyslog服务
[root@localhost ~]# systemctl restart rsyslog
```

通过`tail -f /var/log/remote_ssh.log`来实时检测是否又日志记录发过来



### 日志文件归档

​       如果我们不管理系统产生的上述各种日志文件，日志文件及内容越堆越多，不仅难以查阅，还会因为单一文件过大而影响新的内容写入的效率。logrotate就是一个不错的日志处理程序，准确的说是对日志进行“归档”之类的工作



#### logrotate(日志轮转)

用于针对具体的某一个日志文件进行配置

- 如果没有日志轮转，日志文件会越来越大。将丢弃系统中最旧的日志文件，以节省空间。事实上`logrotate`是挂在`cron`配置目录`cron.daily`下面的，所以会被`cron`每天执行一次：

```bash
[root@localhost ~]# ll /etc/cron.daily/
-rwx------. 1 root root 219 10月 31 2018 logrotate
-rwxr-xr-x. 1 root root 618 10月 30 2018 man-db.cron

# 查看logrotate内容
[root@localhost ~]# cat /etc/cron.daily/logrotate 
#!/bin/sh
/usr/sbin/logrotate -s /var/lib/logrotate/logrotate.status /etc/logrotate.conf

# 日志轮转状态/var/lib/logrotate/logrotate.status
# 日志轮转规则按照/etc/logrotate.conf中来
```

- logrotate主配置文件

```bash
[root@localhost ~]# vim /etc/logrotate.conf 
weekly                         # 多久会执行一次“轮转”，这里设置的是每周一次
rotate 4                     # 轮转后会保留几个历史日志文件，这里是4，也就是说轮转后会删除编号为5的历史日志
create                         # 轮转后创建新的空白日志
dateext                     # 使用日期而非数字编号作为历史日志的标识进行轮转
include /etc/logrotate.d     # 加载/etc/logrotate.d目录下的配置文件
/var/log/wtmp {             # 对/var/log/wtmp日志的特殊设置
    monthly                 # 每月进行一次轮转
    create 0664 root utmp     # 创建的新的空白日志权限为0664，用户为root，用户组为utmp
        minsize 1M             # 原始日志文件超过1M大小才进行轮转
    rotate 1                 # 仅保留1个历史日志文件
}
/var/log/btmp {             # 对/var/log/btmp日志的特殊设置
    missingok                 # 日志轮转期间任何错误都会被忽略
    monthly                 # 每月进行一次轮转
    create 0600 root utmp     # 创建的新日志文件权限为0600，用户为root，用户组为utmp
    rotate 1                 # 仅保留1个历史日志文件
}
```

- 子配置文件

```bash
[root@localhost ~]# cat /etc/logrotate.d/syslog
```
