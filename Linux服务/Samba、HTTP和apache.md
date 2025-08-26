# Samba、HTTP和apache

中间件（介于应用系统和系统软件之间的一类软件，它使用系统软件所提供的基础服务（功能），衔接网络上应用系统的各个部分或不同的应用，能够达到资源共享、功能共享的目的。）：nginx、apache、tomcat



## 一、samba

SAMBA：实现不同操作系统之间实现文件共享和打印服务。跨平台性比较好，多平台支持。支持winserver中很多认证、安全等级制

SAMBA的功能：共享文件和打印，实现在线编辑、实现登录SAMBA用户的身份认证、可以进行NetBIOS名称解析、外围设备共享



### 软件介绍

相关包：`samba` 提供smb服务，`samba-client` 客户端软件，`samba-common` 通用软件，`cifs-utils smb`客户端工具 ，`samba-winbind` 和AD相关



### 相关服务进程

**smbd**：提供文件共享和打印服务，TCP：139、445。

**nmbd**：负责 NetBIOS 名称解析和浏览功能，UDP：137、138。

**winbindd**：用于与 Windows 域集成，支持用户和组的认证。

**samba-ad-dc**：Samba 4 中的域控制器服务。



### Samba主配置文件

主配置文件：/etc/samba/smb.conf 帮助参看：man smb.conf

语法检查： testparm [-v] [/etc/samba/smb.conf]

```bash
[global]
   workgroup = WORKGROUP        # 工作组名称
   server string = Samba Server # 服务器描述
   security = user              # 认证模式
   log file = /var/log/samba/log.%m # 日志文件路径
   max log size = 50            # 最大日志文件大小（KB）
   dns proxy = no               # 禁用 DNS 代理

[shared]
   path = /srv/samba/shared      # 共享路径
   browseable = yes              # 是否可浏览
   writable = yes                # 是否可写
   valid users = @smbgroup       # 允许访问的用户/组
```

#### 全局设置（[global]）

- `workgroup`：指定工作组名称，默认是 `WORKGROUP`。
- security：
  - `user`：用户级认证（常用）。
  - `share`：共享级认证（不推荐）。
  - `domain`：域级认证。
  - `ads`：Active Directory 服务。
- `log file`：日志文件路径。
- `max log size`：限制日志文件大小。

#### 共享设置（[共享名]）

- `path`：共享目录的路径。
- `browseable`：决定共享是否可被浏览。
- `writable`：是否允许写入。
- `valid users`：指定允许访问的用户或组。



### 安装samba和管理SAMBA用户

安装并启动samba服务

```bash
[root@localhost ~]# yum -y install samba
[root@localhost ~]# yum -y install samba-client
[root@localhost ~]# systemctl enable --now smb
[root@localhost ~]# systemctl enable --now nmb
[root@localhost ~]# ss -nlt
#在ss命令的输出中，Recv-Q和Send-Q是与TCP连接相关的两个队列的大小。
Recv-Q表示接收队列的大小。它指示了尚未被应用程序（进程）接收的来自网络的数据的数量。当接收队列的大小超过一定限制时，可能会发生数据丢失。
Send-Q表示发送队列的大小。它指示了应用程序（进程）等待发送到网络的数据的数量。当发送队列的大小超过一定限制时，可能会导致发送缓冲区已满，从而阻塞应用程序发送更多的数据。
```

添加系统用户

```shell
[root@localhost ~]# useradd -r -s /sbin/nologin smb1
```

把系统账号加入samba账号

```shell
[root@localhost ~]# smbpasswd -a smb1
[root@localhost ~]# smbpasswd -e smbuser        # 启用用户
[root@localhost ~]# pdbedit -a -u smb1          # 这个是pdbedit工具的使用
```

添加用户到指定组

```bash
[root@localhost ~]# groupadd smbgroup
[root@localhost ~]# usermod -aG smbgroup smbuser
```

如果已经存在，想修改密码

```shell
[root@localhost ~]# smbpasswd smb1
```

删除用户和密码

```shell
[root@localhost ~]# smbpasswd -x smb1  #只删除密码
[root@localhost ~]# pdbedit -x -u smb1
```

查看samba用户列表

```shell
[root@localhost ~]# pdbedit -L -v
```

查看samba服务器状态

```shell
[root@localhost ~]# smbstatus
```



### samba服务器配置

samba 配置文件/etc/samba/smb.conf格式 ，使用.ini文件的格式

```shell
[root@localhost ~]# vim /etc/samba/smb.conf
[global] #全局参数。
    workgroup = MYGROUP #工作组名称
    server string = Samba Server Version %v #服务器介绍信息，参数%v为显示SMB版本号
    log file = /var/log/samba/log.%m #定义日志文件的存放位置与名称，参数%m为来访的主机名
    max log size = 50 #定义日志文件的最大容量为50KB
    security = user #安全验证的方式，总共有4种
      #share：来访主机无需验证口令；比较方便，但安全性很差
      #user：需验证来访主机提供的口令后才可以访问；提升了安全性
      #server：使用独立的远程主机验证来访主机提供的口令（集中管理账户）
      #domain：使用域控制器进行身份验证
    passdb backend = tdbsam #定义用户后台的类型，共有3种
      #smbpasswd：使用smbpasswd命令为系统用户设置Samba服务程序的密码
      #tdbsam：创建数据库文件并使用pdbedit命令建立Samba服务程序的用户
      #ldapsam：基于LDAP服务进行账户验证
    load printers = yes # 设置在Samba服务启动时是否共享打印机设备
    cups options = raw # 打印机的选项
    hosts allow = 访问的主机的IP地址   # 配置允许访问的主机，可以写成hosts deny来配置拒绝访问的主机
    max log size=50     # 日志文件达到50K，将轮循rotate,单位KB
[homes] #共享参数
    comment = Home Directories # 描述信息
    valid users = %S, %D%w%S
    browseable = no #指定共享信息是否在“网上邻居”中可见
    writable = yes #定义是否可以执行写入操作，与“read only”相反
[printers] #打印机共享参数
    comment = All Printers
    path = /var/spool/samba # 共享文件的实际路径(重要)。
    browseable = no
    guest ok = no     # 是否所有人可见，等同于"public"参数。
    writable = no
    printable = yes
[print$]    # $符号表示隐藏
    comment = Printer Drivers
    path = /var/lib/samba/drivers
    write list = @printadmin root
    force group = @printadmin
    create mask = 0664
    directory mask = 0775
```

宏定义：

- %m：客户端主机的NetBIOS名
- %M：客户端主机的FQDN
- %H：当前用户家目录路径
- %U：当前用户用户名
- %g：当前用户所属组
- %h：samba服务器的主机名
- %L：samba服务器的NetBIOS名
- %I：客户端主机的IP
- %T：当前日期和时间 %S 可登录的用户名
- %S：可登录的用户名
- %D：当前用户所属域或工作组名称
- %w：系统分隔符



### 配置特定目录共享

每个共享目录应该有独立的[ ]部分

- [共享名称]：远程网络看到的共享名称 #真正被共享的名称有Path指定
- comment：注释信息
- path ：所共享的目录路径
- public：能否被guest访问的共享，默认no，和guest ok 类似 #默认不允许匿名访问
- browsable：是否允许所有用户浏览此共享,默认为yes,no为隐藏
- writable=yes
  - 可以被所有用户读写，默认为no #打开之后还需要把文件夹的权限开放
  - 对smb虚拟账户授权：setfacl -m u:smbuser：rwx /path/share 这样就可以上传了
  - 当然也可以 chomd 777 /path/share 最大权限 文件系统级别不控制 在smb级别控制即可
- read only=no：和writable=yes等价，如与以上设置冲突，放在后面的设置生效，默认只读
- write list 用户，@组名，+组名,用，分隔，如writable=no，列表中用户或组可读写，不在列表中用户只读
- valid users 特定用户才能访问该共享，如为空，将允许所有用户，用户名之间用空格分隔



### SAMBA客户端工具

UNC路径: Universal Naming Convention,通用命名规范，格式如下

```plain
\\sambaserver\sharename     #中间为samba服务器名或者ip地址
```

使用smbclient 访问SAMBA服务器

```shell
smbclient -L instructor.example.com
smbclient -L instructor.example.com（服务端IP地址或者主机名） -U smb用户

smbclient //192.168.112.132/share -U smb1
#可以使用-U选项来指定用户%密码，或通过设置和导出USER和PASSWD环境变量来指定
smbclient //instructor.example.com/shared -U user
>cd directory
>get 共享文件名 存放的目标位置 
>get 1.txt /root/1.txt
>put file2
>put 相对路径
```



#### 挂载CIFS文件系统 

```shell
yum -y install cifs-utils   #安装cifs文件系统
mkdir /mnt/smb1
mount -o user=smb1,password=123 //192.168.175.10/share /mnt/smb

# 手动挂载
[root@localhost ~]# mount -o user=smb1,password=123 //192.168.175.19/share /mnt/smb
[root@localhost ~]# df -h

# 开机自动挂载
[root@localhost ~]# echo /etc/fstab
//server/homes /mnt cifs defaults,username='username',password='your password' 0 0

[root@localhost ~]# cat /etc/fstab
//192.168.175.19/share /mnt/smb cifs defaults,username=smb1,password=123 0 0
```



#### 通过用户名共享文件

​       共享销售部`/xsb`这个目录，只有知道用户名和密码的同时可以看这个共享，在/xsb目录中存放销售部重要的数据。需要将security设置为user级别，这样可以启用samba身份验证机制，然后在共享目录`/xsb`下设置valid user 字段，配置只允许销售部员工能访问这个共享目录

- 修改主配置文件安全相关设置

```shell
[root@server1 ~]# vim /etc/samba/smb.conf
[global]
        workgroup = SAMBA
        security = user
#       passdb backend = tdbsam   删除或者注释此行
        passdb backend = smbpasswd
        smb passwd file = /etc/samba/smbpasswd
        printing = cups
        printcap name = cups
        load printers = yes
        cups options = raw
# 重启smb服务之后，会自动生成/etc/samba/smbpasswd该文件
[root@server1 ~]# systemctl restart smb
[root@server1 ~]# ll /etc/samba/smbpasswd
```

- 添加销售部用户和组

```shell
[root@server1 ~]# groupadd xsb
[root@server1 ~]# useradd -g xsb -M -s /sbin/nologin xsb01
[root@server1 ~]# useradd -g xsb -M -s /sbin/nologin xsb02
```

- 添加相应的samba账户

```shell
[root@server1 ~]# smbpasswd -a xsb01
[root@server1 ~]# smbpasswd -a xsb02
```

- 指定共享目录

```shell
[root@server1 ~]# mkdir /xsb
[root@server1 ~]# cp /etc/hosts /xsb
[root@server1 ~]# vim /etc/samba/smb.conf
[xsb]
comment = 销售部重要文件
path = /xsb
valid user = xsb01 xsb02
#valid user = xsb01，@xsb
```

- 重启服务

```shell
[root@server1 ~]# systemctl restart smb.service 
[root@server1 ~]# systemctl restart nmb.service
```

- 检查139和445端口号

```shell
[root@server1 ~]# ss -tanl
State       Recv-Q Send-Q  Local Address:Port                 Peer Address:Port              
LISTEN      0      50                  *:139                             *:*                  
LISTEN      0      50                  *:445                             *:*                  
LISTEN      0      50                 :::139                            :::*                  
LISTEN      0      50                 :::445                            :::*
```



客户端验证

```shell
# linux上验证
[root@server2 ~]# yum install samba-client -y
[root@server2 ~]# smbclient -L //192.168.80.151/xsb -U xsb01
Enter SAMBA\xsb01's password: 
    Sharename       Type      Comment
    ---------       ----      -------
    print$          Disk      Printer Drivers
    xsb             Disk      Xsb Data
    IPC$            IPC       IPC Service (Samba 4.10.16)
    xsb01           Disk      Home Directories
Reconnecting with SMB1 for workgroup listing.
    Server               Comment
    ---------            -------
    Workgroup            Master
    ---------            -------
    SAMBA                SERVER1
# 在windows上进行验证
    windows验证：
    在Window运行输入地址：\\192.168.10.10
    用户名：******
    密码：******
   ## 可以在DOS窗口中使用命令net use * /delete 清空用户缓存信息 ##
```



#### 不同账户访问不同目录

```shell
#创建三个samba用户,并指定密码为centos
useradd -s /sbin/nologin -r smb1   #加选项-r 不创建家目录
useradd -s /sbin/nologin -r smb2
useradd -s /sbin/nologin -r smb3
smbpasswd -a smb1       #创建对应账号的口令  ，不加-a表示修改已经存在的账号的口令
smbpasswd -a smb2
smbpasswd -a smb3
[root@SMB ~]#pdbedit -L    #查看samba账号
smb1:995:
smb3:993:
smb2:994:
#修改samba配置文件
vim /etc/samba/smb.conf
#在[global]的workgroup下加一行 
config file= /etc/samba/conf.d/%U    # 说明：%U表示用户名  这个步骤为关键步骤
[share]               #共享文件夹在最后添加
    Path=/data/dir        #指定分享文件夹的路径
    writeable = yes
    browseable = yes       
    Guest ok=yes        #是否所有人可见，等同于"public"参数。
#配置共享目录和文件
[root@localhost ~]# mkdir -p /data/samba/share
[root@localhost ~]# mkdir -p /data/samba/smb1
[root@localhost ~]# mkdir -p /data/samba/smb2
[root@localhost ~]# touch /data/samba/share/share.txt        # 共享目录及文件
[root@localhost ~]# touch /data/samba/smb1/smb1.txt            # smb1目录及文件
[root@localhost ~]# touch /data/samba/smb2/smb2.txt            # smb2目录及文件
# 将/data/samba目录权限放开
[root@localhost ~]# chmod 777 -R /data/samba

#针对smb1和smb2用户创建单独的配置文件
[root@localhost ~]# mkdir /etc/samba/conf.d/ -pv
[root@localhost ~]# vim /etc/samba/conf.d/smb1
[share]
Path=/data/dir1
Read only= NO #等价于writable = yes
Create mask=0644
#说明：默认为744
[root@localhost ~]# vim /etc/samba/conf.d/smb2
[share]
path=/data/dir2
[root@localhost ~]# systemctl restart smb nmb    #重启对应的服务
# 如果失败，将多余缩进删除
cat -A /etc/samba/smb.conf

#用户smb1，smb2,smb3访问share共享目录，看到目录是不同目录，smb3访问的是默认的share目录
```

- 在客户机上进行测试

```shell
[root@client ~]#smbclient //192.168.32.18/share -U smb1%centos
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Fri Dec 20 13:11:40 2019
  ..                                  D        0  Fri Dec 20 13:10:56 2019
  smb1.txt                            N        0  Fri Dec 20 13:11:40 2019
        52403200 blocks of size 1024. 52004560 blocks available
smb: \> exit   # 输入q也可以退出
[root@client ~]#smbclient //192.168.32.18/share -U smb2%centos
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Fri Dec 20 13:12:53 2019
  ..                                  D        0  Fri Dec 20 13:10:56 2019
  smb2.txt                            N        0  Fri Dec 20 13:12:53 2019
        52403200 blocks of size 1024. 52004560 blocks available
smb: \> exit
[root@client ~]#smbclient //192.168.32.18/share -U smb3%centos
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Fri Dec 20 13:13:12 2019
  ..                                  D        0  Fri Dec 20 13:10:56 2019
  share.txt                           N        0  Fri Dec 20 13:11:26 2019
        52403200 blocks of size 1024. 52004560 blocks available
smb: \> exit
```



## 二、HTTP

### HTTP 工作过程

- **建立连接：** 客户端（如浏览器）与服务器之间通过TCP三次握手建立连接。
- **发送请求：** 客户端向服务器(Apache、Nginx、IIS服务器)发送HTTP请求报文，请求资源或操作。
- **服务器处理请求：** HTTP服务器接收到请求后，处理请求并生成响应。
- **返回响应：** 服务器将响应报文返回给客户端。
- **断开连接：** 通常在响应完成后关闭TCP连接（HTTP/1.0默认短连接，HTTP/1.1支持长连接）。



### URL

HTTP使用统一资源标识符（Uniform Resource Identifiers, URI）来传输数据和建立连接。URI是统一资源标识符，用于标识互联网上的资源。URI分为两种

- **URL（Uniform Resource Locator）：** 统一资源定位符，用于描述资源的地址。
- **URN（Uniform Resource Name）：** 统一资源名称，用于标识资源的名称，不依赖物理位置

URL,全称是UniformResourceLocator, 中文叫统一资源定位符,是互联网上用来标识某一处资源的地址。URL是一种特殊类型的URI，包含了用于查找某个资源的足够的信息。以下面这个URL为例，介绍下普通URL的各部分组成：

```plain
http://iproute.cn:80/news/search?keyword=123&enc=utf8#name=321
```

从上面的URL可以看出，一个完整的URL包括以下几部分：

1. 协议部分：该URL的协议部分为“http：”，这代表网页使用的是HTTP协议。在Internet中可以使用多种协议，如HTTP，FTP等等本例中使用的是HTTP协议。在"HTTP"后面的“//”为分隔符
2. 域名部分：该URL的域名部分为“iproute.cn”。一个URL中，也可以使用IP地址作为域名使用
3. 端口部分：跟在域名后面的是端口，域名和端口之间使用“:”作为分隔符。端口不是一个URL必须的部分，如果省略端口部分，将采用默认端口
4. 虚拟目录部分：从域名后的第一个“/”开始到最后一个“/”为止，是虚拟目录部分。虚拟目录也不是一个URL必须的部分。本例中的虚拟目录是“/news/”
5. 文件名部分：从域名后的最后一个“/”开始到“？”为止，是文件名部分，如果没有“?”,则是从域名后的最后一个“/”开始到“#”为止，是文件部分，如果没有“？”和“#”，那么从域名后的最后一个“/”开始到结束，都是文件名部分。本例中的文件名是“search”。文件名部分也不是一个URL必须的部分，如果省略该部分，则使用默认的文件名
6. 锚部分：从“#”开始到最后，都是锚部分。本例中的锚部分是“name”。锚部分也不是一个URL必须的部分
7. 参数部分：从“？”开始到“#”为止之间的部分为参数部分，又称搜索部分、查询部分。本例中的参数部分为“keyword=123&enc=utf8”。参数可以允许有多个参数，参数与参数之间用“&”作为分隔符。



### HTTP注意事项

1. HTTP是无连接：无连接的含义是限制每次连接只处理一个请求。服务器处理完客户的请求，并收到客户的应答后，即断开连接。采用这种方式可以节省传输时间。
2. HTTP是媒体独立的：这意味着，只要客户端和服务器知道如何处理的数据内容，任何类型的数据都可以通过HTTP发送。客户端以及服务器指定使用适合的MIME-type内容类型。
3. HTTP是无状态：HTTP协议是无状态协议。无状态是指协议对于事务处理没有记忆能力。缺少状态意味着如果后续处理需要前面的信息，则它必须重传，这样可能导致每次连接传送的数据量增大。另一方面，在服务器不需要先前信息时它的应答就较快。



### HTTP 消息结构

HTTP是基于客户端/服务端（C/S）的架构模型，通过一个可靠的链接来交换信息，是一个无状态的请求/响应协议。

一个HTTP"客户端"是一个应用程序（Web浏览器或其他任何客户端），通过连接到服务器达到向服务器发送一个或多个HTTP的请求的目的。

一个HTTP"服务器"同样也是一个应用程序（通常是一个Web服务，如Apache Web服务器或IIS服务器等），通过接收客户端的请求并向客户端发送HTTP响应数据。

HTTP使用统一资源标识符（Uniform Resource Identifiers, URI）来传输数据和建立连接。

一旦建立连接后，数据消息就通过类似Internet邮件所使用的格式[RFC5322]和多用途Internet邮件扩展（MIME）[RFC2045]来传送。

HTTP响应也由四个部分组成，分别是：状态行、消息报头、空行和响应正文



### http请求头部

请求方法+URL+协议版本+请求头部（头部字段名+值+...）+请求数据



#### 常见http请求报文头部属性
Accpet：告诉服务端,客户端接收什么类型的响应
Referer：提供了包含当前请求URI的文档的URL,比如想在网上购物,但是不知道选择哪家电商平台,你就去问百度,说哪家电商的东西便宜啊,然后一堆东西弹出在你面前,第一给就是某宝,当你从这里进入某宝的时候,这个请求报文的Referer就是百度的URL
Cache-Control：对缓存进行控制,如一个请求希望响应的内容在客户端缓存一年,或不被缓可以通过这个报文头设置
Accept-Encoding：这个属性是用来告诉服务器能接受什么编码格式,包括字符编码,压缩形式(一般都是压缩形式)。例如:Accept-Encoding:gzip, deflate(这两种都是压缩格式)
Host：指定要请求的资源所在的主机和端口
User-Agent：告诉服务器，客户端使用的操作系统、浏览器版本和名称



#### POST和GET请求方法区别

提交的过程
- GET提交，请求的数据会附在URL之后（就是把数据放置在HTTP协议头中），以?分割URL和传输数据，多个参数用&连接
- POST提交：把提交的数据放置在是HTTP包的包体中

传输数据的大小
- HTTP协议没有对传输的数据大小、URL长度进行限制
- GET提交，特定浏览器和服务器对URL长度有限制
- POST提交，由于不是通过URL传值，理论上数据不受限

安全性
- POST的安全性要比GET的安全性高
- 登录页面有可能被浏览器缓存，而缓存的是URL
- 其他人查看浏览器的历史纪录，那么别人就可以拿到你的账号和密码了
- 使用GET提交数据还可能会造成Cross-site request forgery攻击



### http响应头部

协议版本+状态码+状态码描述+响应头部（头部字段名+值+...）+响应正文



#### 常见http响应报文头部属性

- Cache-Control:响应输出到客户端后,服务端通过该属性告诉客户端该怎么控制响应内容的缓存
- ETag:表示你请求资源的版本,如果该资源发生啦变化,那么这个属性也会跟着变
- Location:在重定向中或者创建新资源时使用
- Set-Cookie:服务端可以设置客户端的cookie



### HTTP状态码

- 状态代码有三位数字组成，第一个数字定义了响应的类别，共分五种类别:
  - 1xx：指示信息--表示请求已接收，继续处理（一般不会出现）
  - 2xx：成功--表示请求已被成功接收、理解、接受
  - 3xx：重定向--要完成请求必须进行更进一步的操作
  - 4xx：客户端错误--请求有语法错误或请求无法实现
  - 5xx：服务器端错误--服务器未能实现合法的请求
- 常见状态码

```xml
100	Continue	继续。客户端应继续其请求
101	Switching Protocols	切换协议。服务器根据客户端的请求切换协议。只能切换到更高级的协议，例如，切换到HTTP的新版本协议
2开头的状态码
200	OK	请求成功。一般用于GET与POST请求
201	Created	已创建。成功请求并创建了新的资源
202	Accepted	已接受。已经接受请求，但未处理完成
203	Non-Authoritative Information	非授权信息。请求成功。但返回的meta信息不在原始的服务器，而是一个副本
204	No Content	无内容。服务器成功处理，但未返回内容。在未更新网页的情况下，可确保浏览器继续显示当前文档
205	Reset Content	重置内容。服务器处理成功，用户终端（例如：浏览器）应重置文档视图。可通过此返回码清除浏览器的表单域
206	Partial Content	部分内容。服务器成功处理了部分GET请求
3开头的状态码
300	Multiple Choices	多种选择。请求的资源可包括多个位置，相应可返回一个资源特征与地址的列表用于用户终端（例如：浏览器）选择
301	Moved Permanently	永久移动。请求的资源已被永久的移动到新URI，返回信息会包括新的URI，浏览器会自动定向到新URI。今后任何新的请求都应使用新的URI代替
302	Found	临时移动。与301类似。但资源只是临时被移动。客户端应继续使用原有URI
303	See Other	查看其它地址。与301类似。使用GET和POST请求查看
304	Not Modified	未修改。所请求的资源未修改，服务器返回此状态码时，不会返回任何资源。客户端通常会缓存访问过的资源，通过提供一个头信息指出客户端希望只返回在指定日期之后修改的资源
305	Use Proxy	使用代理。所请求的资源必须通过代理访问
306	Unused	已经被废弃的HTTP状态码
307	Temporary Redirect	临时重定向。与302类似。使用GET请求重定向
4开头的状态码
400	Bad Request	客户端请求的语法错误，服务器无法理解
401	Unauthorized	请求要求用户的身份认证
402	Payment Required	保留，将来使用
403	Forbidden	服务器理解请求客户端的请求，但是拒绝执行此请求
404	Not Found	服务器无法根据客户端的请求找到资源（网页）
405	Method Not Allowed	客户端请求中的方法被禁止
406	Not Acceptable	服务器无法根据客户端请求的内容特性完成请求
407	Proxy Authentication Required	请求要求代理的身份认证，与401类似，但请求者应当使用代理进行授权
408	Request Time-out	服务器等待客户端发送的请求时间过长，超时
409	Conflict	服务器完成客户端的PUT请求是可能返回此代码，服务器处理请求时发生了冲突
410	Gone	客户端请求的资源已经不存在。410不同于404，如果资源以前有现在被永久删除了可使用410代码，网站设计人员可通过301代码指定资源的新位置
411	Length Required	服务器无法处理客户端发送的不带Content-Length的请求信息
412	Precondition Failed	客户端请求信息的先决条件错误
413	Request Entity Too Large	由于请求的实体过大，服务器无法处理，因此拒绝请求。为防止客户端的连续请求，服务器可能会关闭连接。如果只是服务器暂时无法处理，则会包含一个Retry-After的响应信息
414	Request-URI Too Large	请求的URI过长（URI通常为网址），服务器无法处理
415	Unsupported Media Type	服务器无法处理请求附带的媒体格式
416	Requested range not satisfiable	客户端请求的范围无效
417	Expectation Failed	服务器无法满足Expect的请求头信息
5开头的状态码
500	Internal Server Error	服务器内部错误，无法完成请求
501	Not Implemented	服务器不支持请求的功能，无法完成请求
502	Bad Gateway	充当网关或代理的服务器，从远端服务器接收到了一个无效的请求
503	Service Unavailable	由于超载或系统维护，服务器暂时的无法处理客户端的请求。延时的长度可包含在服务器的Retry-After头信息中
504	Gateway Time-out	充当网关或代理的服务器，未及时从远端服务器获取请求
505	HTTP Version not supported	服务器不支持请求的HTTP协议的版本，无法完成处理
```



### HTTP请求方法

根据HTTP标准，HTTP请求可以使用多种请求方法。

HTTP1.0定义了三种请求方法： GET, POST 和 HEAD方法。

HTTP1.1新增了五种请求方法：OPTIONS, PUT, DELETE, TRACE 和 CONNECT 方法。

- GET 请求指定的页面信息，并返回实体主体。
- HEAD 类似于get请求，只不过返回的响应中没有具体的内容，用于获取报头
- POST 向指定资源提交数据进行处理请求（例如提交表单或者上传文件）。数据被包含在请求体中。POST请求可能会导致新的资源的建立和/或已有资源的修改。
- PUT 从客户端向服务器传送的数据取代指定的文档的内容。
- DELETE 请求服务器删除指定的页面。
- CONNECT HTTP/1.1协议中预留给能够将连接改为管道方式的代理服务器。
- OPTIONS 允许客户端查看服务器的性能。
- TRACE 回显服务器收到的请求，主要用于测试或诊断。



### HTTP工作原理

在浏览器地址栏键入URL，按下回车之后会经历以下流程：

1. 浏览器向 DNS 服务器请求解析该 URL 中的域名所对应的 IP 地址;
2. 解析出 IP 地址后，根据该 IP 地址和默认端口 80，和服务器建立TCP连接;
3. 浏览器发出读取文件(URL 中域名后面部分对应的文件)的HTTP 请求，该请求报文作为 TCP 三次握手的第三个报文的数据发送给服务器;
4. 服务器对浏览器请求作出响应，并把对应的 html 文本发送给浏览器;
5. 释放 TCP连接;
6. 浏览器将该 html 文本并显示内容;



### 长连接与短连接

HTTP1.1规定了默认保持长连接（HTTP persistent connection ，也有翻译为持久连接），数据传输完成了保持TCP连接不断开（不发RST包、不四次握手），等待在同域名下继续用这个通道传输数据；相反的就是短连接。

在实际使用中，HTTP头部有了Keep-Alive这个值并不代表一定会使用长连接，客户端和服务器端都可以无视这个值，也就是不按标准来。毕竟TCP是一个双向连接的协议，双方都可以决定是不是主动断开。

客户端的长连接不可能无限期的拿着，会有一个超时时间，服务器有时候会告诉客户端超时时间。下图中的Keep-Alive: timeout=20，表示这个TCP通道可以保持20秒



## 三、apache

- Apache是什么：Apache HTTP Server简称为Apache，是一个高性能、功能强大、见状可靠、又灵活的开放源代码的web服务软件。

- 特点：功能强大，高度模块化，采用MPM多路处理模块，配置简单，速度快，应用广泛，性能稳定可靠，可做代理服务器或负载均衡来使用，双向认证，支持第三方模块
- 应用场合
  - 使用Apache运行静态HTML网页、图片
  - 使用Apache结合PHP、Linux、MySQL可以组成LAMP经典架构
  - 使用Apache作代理、负载均衡等



### appache初级命令

#### apache安装

- centos7的软件仓库中存在此软件包，可以直接通过yum进行安装

```bash
[root@localhost ~]# firewall-cmd --add-port=80/tcp --permanent
[root@localhost ~]# firewall-cmd --reload
[root@localhost ~]# yum -y install httpd
[root@localhost ~]# systemctl start httpd
[root@localhost ~]# httpd -v
```

- 使用curl或者浏览器测试网页是否正常

```bash
[root@localhost ~]# curl -I 127.0.0.1
#windows查本机公共ip:curl ipconfig.me   |   curl ifcfg.me
#windows查本机私网ip：ipconfi
```



#### httpd命令

httpd为Apache HTTP服务器程序。直接执行程序可启动服务器的服务

```bash
httpd [-hlLStvVX][-c<httpd指令>][-C<httpd指令>][-d<服务器根目录>][-D<设定文件参数>][-f<设定文件>]
```

选项

-  **-c：**在读取配置文件前，先执行选项中的指令。
- **-C：**在读取配置文件后，再执行选项中的指令。
- **-d<服务器根目录>：**指定服务器的根目录。
- **-D<设定文件参数>：**指定要传入配置文件的参数。
- **-f<设定文件>：**指定配置文件。
- **-h：**显示帮助。
- **-l：**显示服务器编译时所包含的模块。
- **-L：**显示httpd指令的说明。
- **-S：**显示配置文件中的设定。
- **-t：**测试配置文件的语法是否正确。
- **-v：**显示版本信息。
- **-V：**显示版本信息以及建立环境。
- **-X：**以单一程序的方式来启动服务器。
- -**M:**查看启用的模块



#### 指定服务器名字

- 服务器名指定了之后，需要保障域名能够解析到这台服务器上，不然将无法访问

```shell
[root@localhost ~]# vim /etc/httpd/conf/httpd.conf
ServerName www.example.com:80
[root@localhost ~]# httpd -t    # 检查语法
Syntax OK
```

- 指定服务器监听地址，默认是80端口

```shell
[root@localhost ~]# vim /etc/httpd/conf/httpd.conf
Listen 80
[root@localhost ~]# systemctl reload httpd
```



#### 持久连接

Persistent Connection：连接建立，每个资源获取完成后不会断开连接，而是继续等待其它的请求完成，默认关闭持久连接 断开条件：时间限制：以秒为单位， 默认5s，httpd-2.4 支持毫秒级 副作用：对并发访问量大的服务器，持久连接会使有些请求得不到响应折中：使用较短的持久连接时间

- 相关配置

```shell
[root@localhost ~]# vim /etc/httpd/conf/httpd.conf
Keepalive On|Off
KeepaliveTimeout 15 #连接持续15s,可以以ms为单位,默认值为5s
MaxKeepaliveRequests 500 #持久连接最大接收的请求数,默认值100
```

- 测试方法：默认情况下响应完请求信息后连接就断开了

```shell
[root@localhost ~]# yum -y install telnet
[root@localhost ~]# telnet 127.0.0.1 80  
#telnet ip port，出现Escape character is '^]'.说明成功。ctrl + ]  后q退出
GET / HTTP/1.1
Host:127.0.0.1
```

- 特殊场景下，可以设置超时时间为毫秒级，指定 ms 时间单位即可

```shell
[root@localhost ~]#cat /etc/httpd/conf/httpd.conf|grep Keepalive
Keepalive on
Keepalivetimeout 30000ms
```



### apache功能模块

- httpd 有静态功能模块和动态功能模块组成，分别使用 httpd -l 和 httpd -M 查看
- Dynamic Shared Object，加载动态模块配置，不需重启即生效
- 动态模块所在路径：/usr/lib64/httpd/modules/
- 主配置/etc/httpd/conf/httpd.conf文件中指定加载模块配置文件

```shell
# 加载模块示例
[root@localhost ~]# vim /etc/httpd/conf/httpd.conf
Include conf.modules.d/*.conf

# 配置指定实现模块加载格式
[root@localhost ~]# vim /etc/httpd/conf/httpd.conf
LoadModule <mod_name> <mod_path>
```

模块文件路径可使用相对路径：相对于ServerRoot（默认/etc/httpd） 范例：查看模块加载的配置文件

```shell
root@localhost ~]# ls /etc/httpd/conf.modules.d/
00-base.conf  00-dav.conf  00-lua.conf  00-mpm.conf  00-proxy.conf  00-systemd.conf  01-cgi.conf
# 可以指定加载的模块
[root@localhost ~]# cat /etc/httpd/conf.modules.d/00-base.conf
```



### MPM多路处理模块

httpd 支持三种MPM工作模式：prefork, worker, event

- MPM工作模式
  - prefork：多进程I/O模型，每个主进程，管理多个子进程，一个子进程处理一个请求。
  - worker：复用的多进程I/O模型，多进程多线程，每个主进程，管理多个子进程，一个子进程管理多个线程，每个 线程处理一个请求。
  - event：事件驱动模型，多个主进程，每个主进程管理多个子进程，每个子进程管理多个线程，每个线程不再阻塞等待，而是监听套接字等待。当有请求的时候，线程立即处理请求并返回。
- 查看centos7中默认的mpm

```shell
[root@localhost ~]# cat /etc/httpd/conf.modules.d/00-mpm.conf |grep mpm
LoadModule mpm_prefork_module modules/mod_mpm_prefork.so  #默认使用prefork
#LoadModule mpm_worker_module modules/mod_mpm_worker.so
#LoadModule mpm_event_module modules/mod_mpm_event.so
```

- 查看进程

```shell
[root@localhost ~]# ps aux |grep httpd
root       7116  0.0  0.2 224072  5016 ?        Ss   10:39   0:00 /usr/sbin/httpd -DFOREGROUND
apache     7124  0.0  0.1 224072  2960 ?        S    10:39   0:00 /usr/sbin/httpd -DFOREGROUND
apache     7125  0.0  0.1 224072  2960 ?        S    10:39   0:00 /usr/sbin/httpd -DFOREGROUND
apache     7126  0.0  0.1 224072  2960 ?        S    10:39   0:00 /usr/sbin/httpd -DFOREGROUND
apache     7127  0.0  0.1 224072  2960 ?        S    10:39   0:00 /usr/sbin/httpd -DFOREGROUND
apache     7128  0.0  0.1 224072  2960 ?        S    10:39   0:00 /usr/sbin/httpd -DFOREGROUND
root       7214  0.0  0.0 112724   984 pts/0    S+   10:50   0:00 grep --color=auto httpd
[root@localhost ~]# pstree -p 7116
httpd(7116)─┬─httpd(7124)
            ├─httpd(7125)
            ├─httpd(7126)
            ├─httpd(7127)
            └─httpd(7128)
```

- 修改MPM工作模式为`mod_mpm_worker.so`

```shell
[root@localhost ~]# cat /etc/httpd/conf.modules.d/00-mpm.conf |grep mpm
#LoadModule mpm_prefork_module modules/mod_mpm_prefork.so
LoadModule mpm_worker_module modules/mod_mpm_worker.so
#LoadModule mpm_event_module modules/mod_mpm_event.so
[root@localhost ~]# systemctl restart httpd
[root@localhost ~]# ps aux |grep httpd
root       7231  0.2  0.2 224276  5224 ?        Ss   10:52   0:00 /usr/sbin/httpd -DFOREGROUND
apache     7232  0.0  0.1 224024  2928 ?        S    10:52   0:00 /usr/sbin/httpd -DFOREGROUND
apache     7233  0.0  0.3 576640  7560 ?        Sl   10:52   0:00 /usr/sbin/httpd -DFOREGROUND
apache     7234  0.0  0.3 511104  7564 ?        Sl   10:52   0:00 /usr/sbin/httpd -DFOREGROUND
apache     7235  0.0  0.3 576640  7568 ?        Sl   10:52   0:00 /usr/sbin/httpd -DFOREGROUND
root       7318  0.0  0.0 112724   984 pts/0    S+   10:52   0:00 grep --color=auto httpd
[root@localhost ~]# pstree -p 7231
httpd(7231)─┬─httpd(7232)
            ├─httpd(7233)─┬─{httpd}(7241)
            │             ├─{httpd}(7259)
            │             ├─{httpd}(7261)
            │             ├─{httpd}(7263)
            │             ├─{httpd}(7265)
            │             ├─{httpd}(7267)
```



#### prefork工作模式

默认的工作模式是Prefork MPM，这种模式采用的是预派生子进程方式，用单独的子进程来处理请求，子进程间互相独立，互不影响，大大的提高了稳定性，但每个进程都会占用内存，所以消耗系统资源过高。

Prefork MPM 工作原理：控制进程Master首先会生成“StartServers”个进程，“StartServers”可以在Apache主配置文件里配置，然后为了满足“MinSpareServers”设置的最小空闲进程个数，会建立一个空闲进程，等待一秒钟，继续创建两个空闲进程，再等待一秒钟，继续创建四个空闲进程，以此类推，会不断的递归增长创建进程，最大同时创建32个空闲进程，直到满足“MinSpareServers”设置的空闲进程个数为止。Apache的预派生模式不必在请求到来的时候创建进程，这样会减小系统开销以增加性能，不过Prefork MPM是基于多进程的模式工作的，每个进程都会占用内存，这样资源消耗也较高。

- 将apache切换到prefork的模式下，可以通过`httpd -V`来查看

```shell
[root@localhost ~]# httpd -V
```

- 通过调整配置文件，可以修改prefork的参数

```shell
[root@localhost ~]# vim /etc/httpd/conf.d/mpm.conf
StartServers 5              #开始访问进程
MinSpareServers 5           #最小空闲进程
MaxSpareServers 10           #无人访问时，留下空闲的进程
ServerLimit 256               #最多进程数,最大值 20000
MaxRequestWorkers 256         #最大的并发连接数，默认256
MaxConnectionsPerChild 4000    
#子进程最多能处理的请求数量。在处理MaxRequestsPerChild 个请求之后,子进程将会被父进程终止，这时候子进程占用的内存就会释放(为0时永远不释放）
MaxRequestsPerChild 4000    #从 httpd.2.3.9开始被MaxConnectionsPerChild代替
[root@localhost ~]# systemctl restart httpd
```

- 查看进程数

```shell
[root@localhost ~]# pstree -p 7606
httpd(7606)─┬─httpd(7607)
            ├─httpd(7608)
            ├─httpd(7609)
            ├─httpd(7610)
            └─httpd(7611)
```

- 修改prefork参数

```shell
[root@localhost ~]# vim /etc/httpd/conf.d/mpm.conf
StartServers 10
MaxSpareServers 15
MinSpareServers 10
MaxRequestWorkers 256
MaxRequestsPerChild 4000
[root@localhost ~]# systemctl restart httpd
```

```shell
#使用ab进行压力测试
[root@localhost ~]# ab -c 1000 -n 1000000 http://127.0.0.1/
# -c    即concurrency，用于指定的并发数
# -n    即requests，用于指定压力测试总共的执行次数

# 测试过程中，可以看到最大进程数
[root@localhost ~]# ps aux |grep httpd |wc -l
258
#结束ab的压力测试，等待一段时间，可以看到进程数慢慢减少
[root@localhost ~]# ps aux |grep httpd |wc -l
12
```



#### worker工作模式

Worker MPM也是基于多进程的，但是每个进程会生成多个线程，由线程来处理请求，这样可以保证多线程可以获得进程的稳定性；

Worker MPM工作原理： 控制进程Master在最初会建立“StartServers”个进程，然后每个进程会创建“ThreadPerChild”个线程，多线程共享该进程内的资源，同时每个线程独立的处理HTTP请求，为了不在请求到来的时候创建线程，Worker MPM也可以设置最大最小空闲线程，Worker MPM模式下同时处理的请求=ThreadPerChild*进程数，也就是MaxClients，如果服务负载较高，当前进程数不满足需求，Master控制进程会fork新的进程，最大进程数不能超过ServerLimit数，如果需要，可以调整这些对应的参数，比如，如果要调整StartServers的数量，则也要调整 ServerLimit的值

- 修改mpm文件为官方提供的默认数值，然后切换模式到worker模式

```shell
[root@localhost ~]# cat /etc/httpd/conf.modules.d/00-mpm.conf |grep mpm
#LoadModule mpm_prefork_module modules/mod_mpm_prefork.so
LoadModule mpm_worker_module modules/mod_mpm_worker.so
#LoadModule mpm_event_module modules/mod_mpm_event.so
[root@localhost ~]# systemctl restart httpd
[root@localhost ~]# httpd -V
```

- 查看压力测试前后的进程数，查看线程数

```shell
[root@localhost ~]# ps aux |grep httpd |wc -l
5
[root@localhost ~]# pstree -p 8615 | wc -l
53
[root@localhost ~]# ab -c 1000 -n 1000000 http://127.0.0.1/
[root@localhost ~]# ps aux |grep httpd |wc -l
9
[root@localhost ~]# pstree -p 8393 | wc -l
157
```



#### event工作模式

它和 worker模式很像，最大的区别在于，它解决了 keep-alive 场景下 ，长期被占用的线程的资源浪费问题（某些线程因为被keep-alive，挂在那里等待，中间几乎没有请求过来，一直等到超时）。（饥饿？）

event MPM中，会有一个专门的线程来管理这些 keep-alive 类型的线程，当有真实请求过来的时候，将请求传递给服务线程，执行完毕后，又允许它释放。这样，一个线程就能处理几个请求了，实现了异步非阻塞。

event MPM在遇到某些不兼容的模块时，会失效，将会回退到worker模式，一个工作线程处理一个请求。官方自带的模块，全部是支持event MPM的。



### 主要配置文件  httpd.conf

```shell
[root@localhost ~]# vim /etc/httpd/conf/httpd.conf
DocumentRoot "/var/www/html"  #DocumentRoot指向的路径为URL路径的起始位置
<Directory "/var/www"> #当访问的URL只是指定的一个目录，并没有给出具体访问的文件的时候，这个模块会在`/`后面自动加上index.html(默认的行为是重定向到index.html上面)
    Require all granted
</Directory>
```

- 路径必须显式授权后才可以访问
- 可以修改httpd的配置文件来修改这个默认的首页文件

```shell
[root@localhost ~]# httpd -M |grep dir
 dir_module (shared)
[root@localhost ~]# vim /etc/httpd/conf/httpd.conf
<IfModule dir_module>
    DirectoryIndex index.html
</IfModule>
# 用于指定当请求一个目录时,服务器应该提供的默认文件
```

- 关于dir_mod可以查看中文手册：https://www.php.cn/manual/view/17830.html
- 当访问的URL只是指定的一个目录，并没有给出具体访问的文件的时候，这个模块会在`/`后面自动加`index.html`(默认的行为是重定向到index.html上面)
- 可以修改httpd的配置文件来修改这个默认的首页文件



#### 案例-修改默认网址目录

- 默认情况下主页存放于`/var/www/html`目录下，下面修改默认资源存放路径，指定为`/data/html`

```shell
[root@localhost ~]# mkdir -p /data/html
[root@localhost ~]# echo "<h1>hello world</h1>" > /data/html/index.html
[root@localhost ~]# vim /etc/httpd/conf/httpd.conf
DocumentRoot "/data/html"    # 修改资源存放路径
<Directory "/data/html">      # 资源匹配
    AllowOverride None
    Require all granted
</Directory>
[root@localhost ~]# systemctl restart httpd
[root@localhost ~]# chcon -R -t httpd_sys_content_t /data/html  # 设置selinux权限
[root@localhost ~]# setenforce 0    # 或者临时关闭selinux
```

- 在dir_mod中添加`index.htm`

```shell
[root@localhost ~]# echo "<h1>hello linux</h1>" > /data/html/index.htm
[root@localhost ~]# vim /etc/httpd/conf/httpd.conf
<IfModule dir_module>
    DirectoryIndex index.html index.htm  #访问这个页面，发现index.html优先于index.htm
</IfModule>
[root@localhost ~]# systemctl restart httpd
```



#### 默认页面

如果在`DocumentRoot`目录下没有任何文件，会发现有一个默认的页面，也就是显示`Testing 123..`的那个页面，这个是因为有一个`welcome.conf`的配置文件导致的。上面的配置显示，当访问服务器时，提示的 http 错误代码为 403 时，使用`/.noindex.html`页面响应用户请求。如果将其注释，或者修改这个文件，将会就会显示apache的403报错页面。如果我们想自定义报错页面，可以修改这个文件



### 访问控制

#### URL匹配规则

Apache httpd的URL匹配规则主要包括以下几种指令:

1. `<Location>`: 基于URL路径进行匹配。
2. `<LocationMatch>`: 使用正则表达式进行URL路径匹配。
3. `<Directory>`: 基于服务器文件系统的目录路径进行匹配。
4. `<DirectoryMatch>`: 使用正则表达式进行目录路径匹配。
5. `<Files>`: 基于文件名进行匹配。
6. `<FilesMatch>`: 使用正则表达式进行文件名匹配。



**优先级**

这些指令的优先级从高到低为:

1. `<Files>` 和 `<FilesMatch>`
2. `<Directory>` 和 `<DirectoryMatch>`
3. `<Location>` 和 `<LocationMatch>`



**指令常用选项**

指令:Location

- `path`: 指定需要匹配的 URL 路径。可以使用通配符。
- `Order`: 控制允许和拒绝操作的顺序。可以是 `Allow,Deny` 或 `Deny,Allow`。
- `Allow`: 指定允许访问的主机或 IP 地址。
- `Deny`: 指定拒绝访问的主机或 IP 地址。
- `Require`: 指定需要通过身份验证才能访问的用户或组。

 

指令:LocationMatch

- `regex`: 指定一个正则表达式来匹配 URL。
- 其他选项同 `<Location>` 指令。

 

指令:Directory

- `path`: 指定需要匹配的目录路径。可以使用通配符。
- `Options`: 设置目录的访问选项,如 `FollowSymLinks`、`Indexes` 等。
- `AllowOverride`: 控制 .htaccess 文件的覆盖范围。
- 其他选项同 `<Location>` 指令

 

指令: Files 

- `filename`: 指定需要匹配的文件名。可以使用通配符。
- 其他选项同 `<Location>` 指令。

 

指令: FilesMatch

- `regex`: 指定一个正则表达式来匹配文件名。
- 其他选项同 `<Files>` 指令



#### 配置方法

```shell
#基于目录
<Directory “/path">
...
</Directory>
#基于文件
<File “/path/file”>
...
</File>
#基于文件通配符
<File “/path/*file*”>
...
</File>
#基于正则表达式
<FileMatch “regex”>
...
</FileMatch>
```



#### 匹配案例

1. **精确匹配**:

   ```
   <Location "/admin">
       AuthType Basic
       AuthName "Admin Area"
       AuthUserFile "/path/to/htpasswd"
       Require valid-user
   </Location>
   ```

   在这个例子中,任何访问 `/admin` URL 的请求都会被要求进行基本认证(Basic Authentication)。只有通过验证的用户才能访问这个 URL。

2. **前缀匹配**:

   ```bash
   <Location "/documents">
       Options Indexes FollowSymLinks
       AllowOverride None
       Require all granted
   </Location>
   ```

   在这个例子中,任何访问 `/documents` 的 URL 的请求都会被允许执行目录索引和跟踪符号链接。所有用户都被允许访问这些 URL。

3. **正则表达式匹配**:

   ```
   <LocationMatch "\.php$">
       SetHandler application/x-httpd-php
   </LocationMatch>
   ```

   在这个例子中,任何访问以 `.php` 结尾的 URL 的请求都会被 Apache 识别为 PHP 脚本,并使用 PHP 处理器来执行它们。

4. **通配符匹配**:

   ```bash
   <Directory "/var/www/*/images">
       Options Indexes FollowSymLinks
       AllowOverride None
       Require all granted
   </Directory>
   ```

   在这个例子中,任何位于 `/var/www/*/images` 目录下的文件都会被允许通过目录索引和符号链接访问,所有用户都被允许访问这些文件

5. **特定文件类型匹配**:

   ```bash
   <Files "*.html">
       SetOutputFilter INCLUDES
       # SetOutputFilter INCLUDES 是 Apache httpd 中一个非常有用的配置指令,它可以启用 Server-Side Includes (SSI) 功能。
   </Files>
   ```

   在这个例子中,任何以 `.html` 结尾的文件都会被 Apache 处理为包含服务器端包含指令(Server-Side Includes)的文件。

6. **主机名匹配**:

   ```bash
   <VirtualHost www.example.com:80>
       DocumentRoot "/var/www/example"
       ServerName www.example.com
   </VirtualHost>
   ```

   在这个例子中,任何发送到 `www.example.com:80` 的请求都会被映射到 `/var/www/example` 目录,并由 Apache 配置为 `www.example.com` 的虚拟主机。



#### Options指令

- 后跟1个或多个以空白字符分隔的选项列表， 在选项前的+，- 表示增加或删除指定选项：

  ```shell
  options +/- none  [+/- indexes] 
  # 如果我们在可选配置项前加上了符号”+”或”-“，那么表示该可选项将会被合并。所有前面加有”+”号的可选项将强制覆盖当前的可选项设置，而所有前面有”-“号的可选项将强制从当前可选项设置中去除。
  ```

- 常见选项（**默认是全部禁用**）：
  - Indexes：指明的URL路径下不存在与定义的主页面资源相符的资源文件时，返回索引列表给用户
  - FollowSymLinks：允许访问符号链接文件所指向的源文件
  - MultiViews：允许使用mod_negotiation模块提供内容协商的”多重视图”。也就是说，如果客户端请求的路径可能对应多种类型的文件，那么服务器将根据客户端请求的具体情况自动选择一个最匹配客户端要求的文件。
  - SymLinksIfOwnerMatch：只有当符号连接和符号连接指向的目标文件或目录的所有者是同一用户时，才会使用符号连接(如果该配置选项位于配置段中，将会被忽略)
  - ExecCGI：允许使用mod_cgi模块执行CGI脚本
  - Includes：允许使用mod_include模块提供的服务器端包含功能
  - IncludesNOEXEC：允许服务器端包含，但禁用”#exec cmd”和”#exec cgi”。但仍可以从ScriptAlias目录使用”#include virtual”虚拟CGI脚本
  - None：全部禁用
  - All： 全部允许
  
- 在html目录下产生如下目录和文件，然后通过浏览器访问这个目录

```bash
[root@localhost ~]vim /etc/httpd/conf/httpd.conf
<Directory "/data/html">
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>
[root@localhost ~]# mkdir -p /data/html/dir
[root@localhost ~]# cd /data/html/dir
[root@localhost dir]# touch f1 f2
```

- 可以看到其他的目录显示出来，这样是不安全的。因为如果没有index.html文件就会把其他的目录显示出来。所以要修改配置

```shell
[root@localhost ~]# vim /etc/httpd/conf/httpd.conf
<Directory "/data/html">
    Options -Indexes
    #Options Indexes FollowSymLinks 注释掉，默认就会关闭
```

- 
- 可以访问软连接指定文件中的内容。这样也会导致很大的安全风险。

```shell
[root@localhost ~]# vim /etc/httpd/conf/httpd.conf
<Directory "/data/html">
    Options -FollowSymLinks # 关闭FollowSymLinks选项后再次查看，发现软链接文件已经不显示了
    # Options Indexes #把FollowSymLinks删掉
```



#### AllowOverride指令

用于控制是否允许在 .htaccess 文件中覆盖主配置文件的设定，通常用于目录级的动态配置，无需重启服务即可生效。作用域限制为：仅能在 `<Directory>` 块中生效，在 `<Location>`、`<Files>` 等配置段中无效。

- 常见用法：
  - **AllowOverride All:**.htaccess中所有指令都有效
  - **AllowOverride None：**.htaccess 文件无效，此为httpd 2.3.9以后版的默认值
  - **AllowOverride AuthConfig：**.htaccess 文件中，除了AuthConfig 其它指令都无法生效，指定精确指令



案例：在主配置文件中禁止`Indexes`和`FollowSymLinks`，但是在`.htaccess`中打开

```shell
[root@localhost ~]# vim /etc/httpd/conf/httpd.conf
<Directory "/data/html">
    Options -Indexes -FollowSymLinks
    AllowOverride options=FollowSymLinks,Indexes
```

- 创建`.htaccess`文件，然后发现主配置文件中的设置被修改了

```shell
[root@localhost ~]# echo "Options FollowSymLinks Indexes" > /data/html/dir/.htaccess
```

- 因为有主配置文件中设置了`.htaccess`对应的文件拒绝全部访问，所以相对是安全的

```shell
[root@localhost ~]# vim /etc/httpd/conf/httpd.conf
<Files ".ht*">
    Require all denied
</Files>
```



#### 基于IP地址的访问控制

- 针对各种资源，可以基于以下两种方式的访问控制

  - 客户端来源地址
  - 用户账号

- 基于客户端的IP地址的访问控制

  - 无明确授权的目录，默认拒绝

  - 允许所有主机访问：`Require all granted`

  - 拒绝所有主机访问：`Require all denied`

  - 授权指定来源的IP访问：`Require ip <IPADDR>`

  - 拒绝特定的IP访问：`Require not ip <IPADDR>`

  - 授权特定主机访问：`Require host <HOSTNAME>`

  - 拒绝特定主机访问：`Require not host <HOSTNAME>`

    需要注释掉Directory下的 Require all granted。在Directory标签下编辑。

```shell
# 黑名单
<Directory "/data/html">
  <RequireAll>
    Require all granted
    Require not ip 192.168.175.1 #拒绝特定IP
  </RequireAll>
# Require all granted 这一行记得注释掉,要不然里面写的生效不了
</Directory>

#白名单
<RequireAny>
	Require all denied
	require ip 172.16.1.1 #允许特定IP
</RequireAny>

#只允许特定的网段访问
<directory "/data/html">
<requireany>
    require all denied
    Require ip 192.168.175.0/24
</requireany>
</directory>

#只允许特定的网段访问
<Directory "/data/html">
<Requireany>
        Require all denied
        Require ip 192.168.175.10        #只允许特定的主机访问
</Requireany>
</Directory>
```



#### 基于用户的访问控制

- 认证质询：WWW-Authenticate，响应码为401，拒绝客户端请求，并说明要求客户端需要提供账号和密码
- 认证：Authorization，客户端用户填入账号和密码后再次发送请求报文；认证通过时，则服务器发送响应的资源
- 认证方式两种
  - basic：明文
  - digest：消息摘要认证,兼容性差
- 安全域：需要用户认证后方能访问的路径；应该通过名称对其进行标识，以便于告知用户认证的原因用户的账号和密码
- 虚拟账号：仅用于访问某服务时用到的认证标识

##### 配置方法

- 定义安全域
  - 允许账号文件中的所有用户登录访问：
    - Require valid-user #表示只要在这个文件里面的用户都是有效用户，都可以访问

```shell
<Directory "/path">
Options None
AllowOverride None
AuthType Basic
AuthName "String"                              #文字提示描述
AuthUserFile "/PATH/HTTPD_USER_PASSWD_FILE"    #指定存放密码文件
Require user username1 username2 ...           #限制特定的人才能访问
</Directory>
```

- 提供账号和密码存储（文本文件） 使用专用命令完成此类文件的创建及用户管理

```shell
htpasswd [options] /PATH/HTTPD_PASSWD_FILE username
```

- 选项
  - **-c：**自动创建文件，仅应该在文件不存在时使用，不然就会覆盖
  - **-p：**明文密码
  - **-d：**CRYPT格式加密，默认
  - **-m：**md5格式加密
  - **-s：**sha格式加密
  - **-D：**删除指定用户



##### 方法一：修改主配置文件

```shell
# 1. 生成文件并且创建用户
[root@localhost ~]# htpasswd -c /etc/httpd/conf.d/.httpuser user01 #只需要输入第一个用户密码时加“-c”，在其他用户输入密码时加“-c”会把第一个用户的密码覆盖掉
New password: 
Re-type new password: 
Adding password for user user
[root@localhost ~]# htpasswd /etc/httpd/conf.d/.httpuser user02
New password: 
Re-type new password: 
Adding password for user user02
[root@localhost ~]# cat /etc/httpd/conf.d/.httpuser
user01:$apr1$yE3jfs2/$77r76q0l6lTtREczR6uQf1
user02:$apr1$gpNpvZZr$acEh6USVYPR6/WboOMUl91

# 在配置文件中引用这个文件
[root@localhost ~]# mkdir /data/html/admin
[root@localhost ~]# echo "<h1>Hello Linux</h1>" > /data/html/admin/index.html
[root@localhost ~]# vim /etc/httpd/conf.d/test.conf
<directory /data/html/admin>
    AuthType Basic
    AuthName "FBI warning"
    AuthUserFile "/etc/httpd/conf.d/.httpuser"
    Require user user01 
</directory>
# 在访问的时候就需要输入密码
```



##### 方法二：修改.htaccess文件

```shell
# 1.生成文件并且创建用户
[root@localhost ~]# htpasswd -c /etc/httpd/conf.d/.httpuser user01  
New password: 
Re-type new password: 
Adding password for user user01
[root@localhost ~]# htpasswd /etc/httpd/conf.d/.httpuser user02
New password: 
Re-type new password: 
Adding password for user user02
[root@localhost ~]# cat /etc/httpd/conf.d/.httpuser 
user01:$apr1$yE3jfs2/$77r76q0l6lTtREczR6uQf1
user02:$apr1$gpNpvZZr$acEh6USVYPR6/WboOMUl91

[root@localhost ~]# vim /etc/httpd/conf.d/test.conf
<Directory "/data/html/admin">
    AllowOverride Authconfig     #设置AllowOverride选项为All
    Require all granted
</Directory>

[root@localhost ~]# vim /data/html/admin/.htaccess  #写入.htaccess文件
AuthType Basic
AuthName "FBI warning"
AuthUserFile "/etc/httpd/conf.d/.httpuser"
Require user user01
#最后测试是否成功
```



#### 基于组账号进行认证

```shell
<Directory “/path">   
# 可以对htpasswd产生的虚拟用户进行分组管理
AuthType Basic
AuthName "String“
AuthUserFile "/PATH/HTTPD_USER_PASSWD_FILE"
AuthGroupFile "/PATH/HTTPD_GROUP_FILE"
Require group grpname1 grpname2 ...
</Directory>
```

案例

```shell
# 1.创建用户
[root@localhost ~]# htpasswd -c /etc/httpd/conf.d/.httpuser user01
New password: 
Re-type new password: 
Adding password for user user01
[root@localhost ~]# htpasswd /etc/httpd/conf.d/.httpuser user02
New password: 
Re-type new password: 
Adding password for user user02
[root@localhost ~]# cat /etc/httpd/conf.d/.httpuser 
user01:$apr1$yE3jfs2/$77r76q0l6lTtREczR6uQf1
user02:$apr1$gpNpvZZr$acEh6USVYPR6/WboOMUl91

[root@localhost ~]# vim /etc/httpd/conf.d/.httpgroup  # 创建组
webadmin: user01 user02

[root@localhost ~]# vim /etc/httpd/conf.d/test.conf  #修改httpd配置文件
<Directory "/data/html/admin">
	AuthType Basic
	AuthName "FBI warning"
	AuthUserFile "/etc/httpd/conf.d/.httpuser"
	AuthGroupFile "/etc/httpd/conf.d/.httpgroup"
	Require group webadmin
</Directory>
```



### 日志配置

- httpd有两种日志类型：访问日志、错误日志
- 日志等级：debug, info, notice, warn,error, crit, alert,emerg

#### 日志格式

- 日志格式可以自定义

```shell
[root@localhost ~]# vim /etc/httpd/conf/httpd.conf
<IfModule log_config_module>
    # The following directives define some format nicknames for use with
    # a CustomLog directive (see below).
    LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
    LogFormat "%h %l %u %t \"%r\" %>s %b" common

    <IfModule logio_module>
      # You need to enable mod_logio.c to use %I and %O
      LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %I %O" combinedio
    </IfModule>
    CustomLog "logs/access_log" combined
</IfModule>
```

- 变量参考
  - %h 客户端IP地址
  - %l 远程用户,启用mod_ident才有效，通常为减号“-”
  - %u 验证（basic，digest）远程用户,非登录访问时，为一个减号“-”
  - %t 服务器收到请求时的时间
  - %r First line of request，即表示请求报文的首行；记录了此次请求的“方法”，“URL”以及协议版本
  - %>s 对于已在内部重定向的请求，这是原始请求的状态。使用%>s 的最终状态。类型脚本中的exit 数字
  - %b 响应报文的大小，单位是字节；不包括响应报文http首部
  - %{Referer}i 请求报文中首部“referer”的值；即从哪个页面中的超链接跳转至当前页面。 { }里面内容就是报文中的一个键值对
  - %{User-Agent}i 请求报文中首部“User-Agent”的值；即发出请求的应用程序，多数为浏览器型号
- 日志存放位置

```shell
[root@localhost ~]# ls /var/log/httpd/
这个文件夹默认是软连接过来的
[root@localhost ~]# ll -d /etc/httpd/logs
lrwxrwxrwx. 1 root root 19 5月   2 09:30 /etc/httpd/logs -> ../../var/log/httpd
```



### 别名模块alias_module

alias 别名，可以隐藏真实文件系统路径。这里实现的目的是用news文件目录来替代newsdir/index.html访问文件路径，从而起到隐藏真实文件系统路径的目的。让`dir`目录隐藏，让其可以被`news`路径访问

```shell
[root@localhost ~]# vim /etc/httpd/conf/httpd.conf
<IfModule alias_module>
    ScriptAlias /cgi-bin/ "/var/www/cgi-bin/"
    Alias /news/ /data/html/dir/
    <Directory "/data/html/dir">
        Require all denied
    </Directory>
</IfModule>
[root@localhost ~]# systemctl reload httpd
```



#### Alias 案例

通过alias配置，实现访问路径到本地文件系统的映射

```bash
#新增一个子配置文件，内容如下：

[root@localhost ~]# vim /etc/httpd/conf.d/pub.conf
<VirtualHost 192.168.88.136:80 >
        ServerName 192.168.88.136
        DocumentRoot /data
        alias /pub /data/html/pub
</Virtualhost>

<Directory /data>
        AllowOverride none
        Require all granted
</Directory>
# 通过alias模块，使得我们访问/pub的时候，自动帮我们映射到/data/html/pub

#创建相关目录及访问文件
[root@localhost ~]# mkdir -p /data/html/pub
[root@localhost ~]# echo "in /data/html/pub" > /data/html/pub/index.html

[root@localhost ~]# chmod 755 -R /data #赋予该目录及子目录权限
[root@localhost ~]# httpd -t  #检查配置并且重启httpd服务
[root@localhost ~]# systemctl restart httpd
```



### httpd服务状态信息显示

当我们需要获取 httpd 服务器在运行过程中的实时状态信息时可以使用该功能

```bash
[root@localhost ~]# vim /etc/httpd/conf.d/test.conf
<Location "/status">
    <requireany>
        require all denied
        require ip 192.168.88.0/24
        #定义特定的网段能够访问
    </requireany>
        SetHandler server-status
        #指定状态信息
</Location>
ExtendedStatus On
```

```bash
[root@localhost ~]# httpd -t #检查配置并且重启httpd服务
[root@localhost ~]# systemctl restart httpd
#访问测试
```



### 虚拟主机

- httpd 支持在一台物理主机上实现多个网站，即多虚拟主机
- 网站的唯一标识
  - IP相同，但端口不同
  - IP不同，但端口均为默认端口
  - FQDN不同
- 多虚拟主机有三种实现方案
  - 基于ip：为每个虚拟主机准备至少一个ip地址
  - 基于port：为每个虚拟主机使用至少一个独立的port
  - 基于FQDN：为每个虚拟主机使用至少一个FQDN
- 虚拟主机的基本配置方法

```shell
<VirtualHost IP:PORT>
	ServerName FQDN
	DocumentRoot "/path"
</VirtualHost>
# 建议：上述配置存放在独立的配置文件中
```

- 其它常用可用指令

```shell
ServerAlias：虚拟主机的别名；可多次使用
ErrorLog： 错误日志
CustomLog：访问日志
<Directory "/path"> </Directory>
```



#### 基于端口的虚拟主机

```shell
[root@localhost ~]#mkdir /data/website{1,2,3}
[root@localhost ~]#echo "This is NO.1 website!" > /data/website1/index.html
[root@localhost ~]#echo "This is NO.2 website!" > /data/website2/index.html
[root@localhost ~]#echo "This is NO.3 website!" > /data/website3/index.html
[root@localhost ~]#vim /etc/httpd/conf.d/test.conf 
[root@localhost ~]#cat /etc/httpd/conf.d/test.conf
listen 8001                     
listen 8002
listen 8003
<virtualhost *:8001>      #指定端口                    
    documentroot /data/website1/              #指定定义的路径
    CustomLog logs/website1_access.log combined    #增加对应的日志
<directory /data/website1>              #对于文件给与访问权限
    require all granted
</directory>
</virtualhost>
<virtualhost *:8002>
    documentroot /data/website2/
    CustomLog logs/website2_access.log combined
<directory /data/website2>
    require all granted
</directory>
</virtualhost>
<virtualhost *:8003>
    documentroot /data/website3/
    CustomLog logs/website3_access.log combined
<directory /data/website3>
    require all granted
</directory>
</virtualhost>
[root@localhost ~]#systemctl restart httpd
[root@localhost ~]#[root@localhost ~]#ll /var/log/httpd/   #各有各的访问日志
total 24
-rw-r--r-- 1 root root 2084 Dec 12 21:41 access_log
-rw-r--r-- 1 root root 4441 Dec 12 21:40 error_log
-rw-r--r-- 1 root root  506 Dec 12 21:42 website1_access.log
-rw-r--r-- 1 root root  506 Dec 12 21:43 website2_access.log
-rw-r--r-- 1 root root  570 Dec 12 21:44 website3_access.log
[root@localhost ~]#curl 192.168.32.8:8001
This is NO.1 website!
[root@localhost ~]#curl 192.168.32.8:8002
This is NO.2 website!
[root@localhost ~]#curl 192.168.32.8:8003
This is NO.3 website!
[root@localhost ~]#
```



#### 基于IP的虚拟主机

```bash
#先通过nmtui命令给linux系统配置多个IP地址
#重启网络配置，并且检查IP地址是否添加成功
[root@localhost CSDN]# systemctl restart network
[root@localhost CSDN]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:cb:d5:1a brd ff:ff:ff:ff:ff:ff
    inet 192.168.88.136/24 brd 192.168.88.255 scope global noprefixroute dynamic ens33
       valid_lft 5185795sec preferred_lft 5185795sec
    inet 192.168.88.137/24 brd 192.168.88.255 scope global secondary noprefixroute ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::a49c:12c9:1ebd:8fb2/64 scope link tentative noprefixroute dadfailed
       valid_lft forever preferred_lft forever
    inet6 fe80::70a:e8e6:d043:dcf2/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

```shell
# 配置基于IP地址的虚拟主机
[root@localhost data]# vim /etc/httpd/conf.d/site.conf
<Directory "/site/">
        Require all granted
</Directory>
<VirtualHost 192.168.88.136:80>
        Servername 192.168.88.136
        DocumentRoot "/site/136/"
</VirtualHost>
<VirtualHost 192.168.88.137:80>
        Servername 192.168.88.137
        DocumentRoot "/site/137/"
</VirtualHost>
[root@localhost ~]# mkdir -p /site/137
[root@localhost ~]# mkdir -p /site/136
[root@localhost ~]# echo "in 136" > /site/136/index.html
[root@localhost ~]# echo "in 137" > /site/137/index.html
[root@localhost ~]# curl 192.168.88.136
in 136
[root@localhost ~]# curl 192.168.88.137
in 137
```



#### 基于FQDN虚拟主机

```shell
[root@localhost ~]# cat /etc/httpd/conf.d/site.conf
#Listen 80    因为/etc/httpd/conf/httpd.conf文件里面有IncludeOptional conf.d/*.conf文件包括site.conf，与本文件里的Listen 80
<Directory "/data/">
    Require all granted
</Directory>
<VirtualHost 192.168.80.100:80>
    Servername www.site5.com
    DocumentRoot "/data/site5/"
</VirtualHost>
<VirtualHost 192.168.80.100:80>
    Servername www.site6.com
    DocumentRoot "/data/site6/"
</VirtualHost>            
[root@localhost ~]# cat /etc/hosts
192.168.0.142 www.site5.com
192.168.0.142 www.site6.com
[root@localhost ~]# curl www.site5.com:10101
<h1>This is site5</h1>
[root@localhost ~]# curl www.site6.com:10101
<h1>This is site6</h1>
```



### 网页压缩技术

- 使用mod_deflate模块压缩页面优化传输速度
- 适用场景
  - 节约带宽，额外消耗CPU；同时，可能有些较老浏览器不支持
  - 压缩适于压缩的资源，例如文本文件

```shell
# 确认是否加载浏览器压缩模块
[root@localhost ~]# httpd -M |grep deflate
 deflate_module (shared)
```

压缩指令

```shell
#可选项
SetOutputFilter DEFLATE
# 指定对哪种MIME类型进行压缩，必须指定项，例如：
AddOutputFilterByType DEFLATE text/plain
AddOutputFilterByType DEFLATE text/html
AddOutputFilterByType DEFLATE application/xhtml+xml
AddOutputFilterByType DEFLATE text/xml
AddOutputFilterByType DEFLATE application/xml
AddOutputFilterByType DEFLATE application/x-javascript
AddOutputFilterByType DEFLATE text/javascript
AddOutputFilterByType DEFLATE text/css
# 也可以同时写多个
AddOutputFilterByType DEFLATE text/html text/plain text/xml text/css application/javascript
#压缩级别 (Highest 9 - Lowest 1) ，默认gzip  默认级别是有库决定
DeflateCompressionLevel 9
#排除特定旧版本的浏览器，不支持压缩
#Netscape 4.x 只压缩text/html
BrowserMatch ^Mozilla/4 gzip-only-text/html
#Netscape 4.06-08 三个版本 不压缩
BrowserMatch ^Mozilla/4\.0[678] no-gzip
#Internet Explorer标识本身为“Mozilla / 4”，但实际上是能够处理请求的压缩。如果用户代理首部
匹配字符串“MSIE”（“B”为单词边界”），就关闭之前定义的限制
BrowserMatch \bMSI[E] !no-gzip !gzip-only-text/html
SetOutputFilter DEFLATE        # 启用Gzip压缩
```



#### 压缩对比实验

```shell
[root@localhost ~]#vim /etc/httpd/conf.d/test.conf
[root@localhost ~]#cat /etc/httpd/conf.d/test.conf 
<virtualhost *:80>
        documentroot /data/website1/
        servername www.aaa.com
        <directory /data/website1/>
                require all granted
        </directory>
                CustomLog "logs/a_access_log" combined
                AddOutputFilterByType DEFLATE text/plain     #增加压缩机制
                AddOutputFilterByType DEFLATE text/html      #增加压缩机制
                DeflateCompressionLevel 9                    #选择默认压缩比
</virtualhost>
<virtualhost *:80>
        documentroot /data/website2/
        servername www.bbb.com
        <directory /data/website2/>
                require all granted
        </directory>
                CustomLog "logs/a_access_log" combined
                #AddOutputFilterByType DEFLATE text/plain    #注释掉 形成对比
                #AddOutputFilterByType DEFLATE text/html     #注释掉 形成对比
                #DeflateCompressionLevel 9                   #注释掉 形成对比
</virtualhost>
[root@localhost ~]#vim /etc/hosts
[root@localhost ~]#cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.32.8 www.aaa.com  www.bbb.com      #新增DNS解析
#最后测试时，可以看到压缩过的文件占用大小jioa
```



### HTTPS

#### https定义

HTTPS协议是一个应用层协议，是在HTTP协议的基础上引入了一个加密层。使用安全套接字层（SSL，Secure Sockets Layer）或传输层安全（TLS，Transport Layer Security）协议对HTTP进行加密，从而在数据传输过程中提供加密和认证保护。HTTPS协议使用的端口号是443。



#### HTTPS的工作流程

1.客户端发起连接请求：
客户端（通常是浏览器）向服务器发送一个安全连接请求，使用HTTPS的URL或点击HTTPS链接触发。
2.服务器证书发送：
服务器收到请求后，将自己的数字证书发送给客户端。证书中包含了服务器的公钥、数字签名和其他相关信息。
3.客户端验证证书：
浏览器接收到服务器证书后，会进行一系列的验证步骤，包括验证证书是否由受信任的证书颁发机构签发，以及证书中的域名是否与目标服务器的域名相匹配。如果验证通过，客户端可以确定所连接的服务器是可信的。
4.密钥协商：
一旦服务器证书被验证通过，会生成基于双方交换的随机数和密钥交换参数（如 ECDHE 公钥）通过数学计算共同推导出的共享密钥。用于后续的数据加密和解密。
5.通信加密：
服务器使用其私钥解密客户端发送的对称密钥，并与客户端之间建立起一个加密的安全通道。从此之后，客户端和服务器之间的数据传输都会在此加密通道中进行，保证数据的机密性。
6.安全数据传输：
在建立了安全通道后，客户端和服务器可以安全地传输数据了。数据在发送前，会使用对称密钥对数据进行加密，然后在接收端使用相同的对称密钥解密。
7.完整性检查：
为了确保数据在传输过程中没有被篡改，HTTPS使用消息摘要函数进行完整性检查。接收方会对接收到的数据进行校验，并比对校验结果与发送方计算的结果是否相同。



#### 颁发自建证书

一、通过openssl工具来自己生成一个证书，然后颁发给自己

```shell
# 1. 安装mod_ssl和openssl
[root@localhost ~]# yum install mod_ssl openssl -y

# 2.生成2048位的加密私钥
[root@localhost ~]# openssl genrsa -out server.key 2048

# 3.生成证书签名请求（CSR）
[root@localhost ~]# openssl req -new -key server.key -out server.csr
# 一路回车到底，过程暂时不需要管

# 4.生成类型为X509的自签名证书。有效期设置3650天，即有效期为10年
[root@localhost ~]# openssl x509 -req -days 3650 -in server.csr -signkey server.key -out server.crt

# 5.复制文件到相应的位置
[root@localhost ~]# cp server.crt /etc/pki/tls/certs/
[root@localhost ~]# cp server.key /etc/pki/tls/private/     
[root@localhost ~]# cp server.csr /etc/pki/tls/private/

# 6.修改配置文件，指定我们自己的证书文件路径
[root@localhost ~]# vim /etc/httpd/conf.d/ssl.conf
SSLCertificateFile /etc/pki/tls/certs/server.crt
SSLCertificateKeyFile /etc/pki/tls/private/server.key

# 7.重启httpd
[root@node1 ~]# systemctl restart httpd
```

二、检查443端口是否开放

```bash
[root@localhost ~]# ss -nlt
State   Recv-Q  Send-Q   Local Address:Port   Peer Address:Port Process
LISTEN  0       128            0.0.0.0:22          0.0.0.0:*
LISTEN  0       511                  *:443               *:*
LISTEN  0       511                  *:80                *:*
LISTEN  0       128               [::]:22             [::]:*
```

三、https访问测试

可以使用curl命令查看证书，可以看到我们的证书是一个自签名证书

```bash
[root@localhost ~]# curl -kv https://127.0.0.1
#k表示支持https
#v表示显示详细的信息
.....
SSL certificate verify result: self-signed certificate (18), continuing anyway.
.....
```



### URL重定向

- URL重定向，即将httpd 请求的URL转发至另一个的URL
- 重定向指令

```shell
# httpd.conf 文件中的配置
Redirect [status] /old-url /new-url
```

- status状态
  - permanent： 返回永久重定向状态码 301
  - temp：返回临时重定向状态码302. 此为默认值

**文档演示：**

1. 假设有一个旧网站 `www.example.com`，需要将其重定向到新网站 `www.newexample.com`。可以在 Apache 的配置文件中添加以下内容来实现这个重定向:

   ```yaml
   <VirtualHost *:80>
       ServerName www.example.com
       Redirect permanent / https://www.newexample.com/
   </VirtualHost>
   
   <VirtualHost *:443>
       ServerName www.example.com
       Redirect permanent / https://www.newexample.com/
   </VirtualHost>
   ```

   这个配置会对两种情况进行重定向:

   1. 当用户访问 `http://www.example.com` 时,会被永久重定向(301 Moved Permanently)到 `https://www.newexample.com/`。
   2. 当用户访问 `https://www.example.com` 时,也会被永久重定向到 `https://www.newexample.com/`。

   可以根据需要调整重定向的 HTTP 状态码和目标 URL 路径。常见的重定向状态码有:

   - `301 Moved Permanently`: 永久重定向
   - `302 Found`: 临时重定向
   - `307 Temporary Redirect`: 临时重定向(保留请求方法)

   除了使用 `Redirect` 指令,也可以使用 `RedirectMatch` 指令来基于正则表达式进行更复杂的重定向规则。例如:

   ```bash
   RedirectMatch 301 ^/old-page\.html$ https://www.newexample.com/new-page
   ```

   这将把 `/old-page.html` 重定向到 `https://www.newexample.com/new-page`。




## 四、LAMP架构

LAMP就是由Linux+Apache+MySQL+PHP组合起来的架构

并且Apache默认情况下就内置了PHP解析模块，所以无需CGI即可解析PHP代码



### LAMP架构部署

#### 安装Apache

```bash
[root@localhost ~]# yum install -y httpd

# 启动httpd
[root@localhost ~]# systemctl enable --now httpd

# 关闭防火墙和SElinux
[root@localhost ~]# systemctl stop firewalld
[root@localhost ~]# setenforce 0
```

**访问测试：**`http://IP`



#### 安装php环境

```shell
#centos7
# 添加一个php的yum源
vim /etc/yum.repos.d/eagle.repo
[eagle]
name=Eagle's lab
baseurl=http://file.eagleslab.com:8889/%E8%AF%BE%E7%A8%8B%E7%9B%B8%E5%85%B3%E8%BD%AF%E4%BB%B6/%E4%BA%91%E8%AE%A1%E7%AE%97%E8%AF%BE%E7%A8%8B/Centos7%E6%BA%90/
gpgcheck=0
enabled=1

#安装php71w全家桶
yum -y install php71w php71w-cli php71w-common php71w-devel php71w-embedded php71w-gd php71w-mcrypt php71w-mbstring php71w-pdo php71w-xml php71w-fpm php71w-mysqlnd php71w-opcache php71w-pecl-memcached php71w-pecl-redis php71w-pecl-mongodb

#重启httpd.service
systemctl restart httpd.service

#rockyLinux9可用
[root@localhost ~]# yum install -y php*

# 启动php-fpm
[root@localhost ~]# systemctl enable --now php-fpm
[root@localhost ~]# systemctl status php-fpm    检查php是否启动
```



#### 安装Mysql数据库

```bash
# 安装mariadb数据库软件
[root@localhost ~]# yum install -y mariadb-server mariadb

# 启动数据库并且设置开机自启动
[root@localhost ~]# systemctl enable --now mariadb

# 设置mariadb的密码
[root@localhost ~]# mysqladmin password '123456'

# 验证数据库是否工作正常
[root@localhost ~]# mysql -uroot -p123456 -e "show databases;"
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
```



#### 安装phpmyadmin

部署phpmyadmin工具，该工具可以让我们可视化管理我们的数据库

```bash
# 移动到网站根目录
[root@localhost ~]# cd /var/www/html

# 下载phpmyadmin源码
[root@localhost ~]# wget https://files.phpmyadmin.net/phpMyAdmin/5.1.1/phpMyAdmin-5.1.1-all-languages.zip

# 解压软件包，并且重命名
[root@localhost ~]# unzip phpMyAdmin-5.1.1-all-languages.zip
[root@localhost ~]# mv phpMyAdmin-5.1.1-all-languages phpmyadmin
```

访问`http://IP/phpmyadmin`进行测试，用户名和密码为刚才初始化数据库时设置的root和123456，登陆后，会进入图形化管理界面



#### PHP探针测试

在默认的网站根目录下创建`info.php`

```bash
vim /var/www/html/info.php
<?php
    phpinfo();
?>
```

写一个简单的php代码，可以使用phpinfo函数查看php的信息，从而检测是否成功解析php代码

编写好以后，我们访问：`http://IP/info.php`测试



#### 数据库连接测试

编写php代码，用php来连接数据库测试

```bash
vim /var/www/html/mysql.php
<?php
    $servername = "localhost";
    $username = "root";
    $password = "123456";

    // 创建连接
    $conn = mysqli_connect($servername, $username, $password);

    // 检测连接
    if (!$conn) {
         die("Connection failed: " . mysqli_connect_error());
    }
    echo "连接MySQL...成功！";
?>
```

编写好以后，我们访问：`http://IP/mysql.php`测试:



### 部署typecho个人博客

#### 源码获取

下载typecho博客系统源码到`/var/www/html/typecho`

```bash
[root@localhost ~]# cd /var/www/html

# 创建typecho目录
[root@localhost ~]# mkdir typecho
[root@localhost ~]# cd typecho

[root@localhost ~]# wget https://github.com/typecho/typecho/releases/latest/download/typecho.zip

# 解压源码
[root@localhost ~]# unzip typecho.zip
```



#### 创建数据库

在phpadmin界面里点解数据库，进行数据库的创建



#### 安装博客系统

进入网站安装的部分了，访问博客系统页面：http://IP/typecho

提示安装目录下面的/usr/uploads没有权限，那么我们手动赋予该目录w权限

```bash
[root@localhost typecho]# chmod a+w usr/uploads/
```

如果遇到提示无法自动创建配置文件config.inc.php。我们手动在typecho目录中创建这个文件，并且把内容复制进去

```bash
vim config.inc.php
```



#### 切换主题

第三方主题商店：https://www.typechx.com/

```bash
# 第三方主题商店会跳转到github，下载下来解压压缩包到提前准备好的专门放主题的目录，并且将主题文件夹重命名
[root@localhost themes]# mkdir /var/www/html/typecho/usr/themes
[root@localhost themes]# unzip typecho-theme-sagiri-master.zip
[root@localhost themes]# mv typecho-theme-sagiri-master sagiri

# 可以删除旧的压缩包文件
[root@localhost themes]# rm -rf typecho-theme-sagiri-master.zip
# 然后回到博客首页刷新一下，就可以看到新的主题已经应用了。会有一些图片资源的丢失，需要了解一点前端知识，就可以将其完善好了。
```



## 总结

本节重点

samba熟练部署，了解常用功能



Http工作原理

URL

请求头部和响应头部对里面的内容大致有印象

GET,POST，其他请求作为课余了解

常见状态码

长连接与短连接



apache

-输入URL后，经历的过程

-MPM多路处理模块，结构图，

-长连接的配置方法

-apache常用的功能模块

-熟悉httpd软件配置文件的路径

-URL匹配规则

-访问控制 用户名,用户组，IP等

-日志（访问日志，错误日志）

-alias别名模块

-虚拟主机（不同端口，不同IP，不同域名）
