# MySQL

## 一、安装MySQL

安装包安装：rpm   /   yum 安装     如：yum -y install mariadb /    安装二进制文件  

### 二进制安装

- 下载二进制包并且解压

```shell
[root@localhost ~]# wget https://downloads.mysql.com/archives/get/p/23/file/mysql-5.6.40-linux-glibc2.12-x86_64.tar.gz
[root@localhost ~]# tar xzvf mysql-5.6.40-linux-glibc2.12-x86_64.tar.gz
```

```shell
[root@localhost ~]# mkdir /application
[root@localhost ~]# mv mysql-5.6.40-linux-glibc2.12-x86_64 /application/mysql-5.6.40
[root@localhost ~]# ln -s /application/mysql-5.6.40 /application/mysql 
#为了后续写的脚本，方便监控mysql

[root@localhost ~]# cd /application/mysql/support-files    
#该文件夹有mysql初始化（预设）配置文件，覆盖文件是因为注释更全。

[root@localhost ~]# cp my-default.cnf /etc/my.cnf
[root@localhost ~]# mkdir /etc/init.d
[root@localhost ~]# cp mysql.server /etc/init.d/mysqld    
# mysql.server包含如何启动mysql的脚本命令，让系统知道通过该命令启动mysql时的动作，该目录存放系统中各种服务的启动/停止脚本
[root@localhost ~]# cd /application/mysql/scripts
[root@localhost scripts]# useradd mysql -s /sbin/nologin -M

[root@localhost scripts]# yum -y install autoconf
[root@localhost scripts]# yum install -y perl-Sys-Hostname
[root@localhost scripts]# ./mysql_install_db --user=mysql --basedir=/application/mysql --data=/application/mysql/data
[root@localhost ~]# vim /etc/profile.d/mysql.sh    
#写到环境变量的子配置文件内，未修改前只能使用/bin/mysql的命令才能启用，而不能全局使用
export PATH="/application/mysql/bin:$PATH"

[root@localhost ~]# source /etc/profile    #否则要重启系统才生效
```



**需要注意，官方编译的二进制包默认是在/usr/local目录下的，我们需要修改配置文件**

```shell
[root@localhost ~]#sed -i 's#/usr/local#/application#g' /etc/init.d/mysqld /application/mysql/bin/mysqld_safe
```



此时不可以通过systemctl命令启动，只能通过/etc/init.d/mysql start启动（nginx也是，如果此时通过这样的命令启动Nginx，会导致systemctl start Nginx失败，因为冲突。）

```shell
#创建systemd管理文件，并且测试是否正常使用
[root@localhost ~]# vim /usr/lib/systemd/system/mysqld.service
[Unit]
Description=MySQL Server
Documentation=man:mysqld(8)
Documentation=https://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target
After=syslog.target
[Install]
WantedBy=multi-user.target
[Service]
User=mysql
Group=mysql
ExecStart=/application/mysql/bin/mysqld --defaults-file=/etc/my.cnf    
#强制从my.cnf读配置，不然会从多个路径读配置
LimitNOFILE = 5000
server_id = 1    
#用作主从的时候生效

[root@localhost ~]# vim /etc/my.cnf
basedir = /application/mysql/
datadir = /application/mysql/data

# 关闭防火墙和selinux，负责会因为权限问题启动不了
[root@localhost scripts]# setenforce 0
[root@localhost scripts]# systemctl stop firewalld

# 启动mysqld服务
[root@localhost ~]# systemctl daemon-reload
[root@localhost ~]# systemctl start mysqld
[root@localhost ~]# systemctl enable mysqld
#此时ss -ntl 可以看到3306端口

[root@localhost ~]# mysqladmin -uroot password '123456

# 由于RockyLinux9版本的lib库较新，为了适配mysql5.6版本，我们通过软连接的方式降级
[root@localhost ~]# ln -s /usr/lib64/libncurses.so.6 /usr/lib64/libncurses.so.5
[root@localhost ~]# ln -s /usr/lib64/libtinfo.so.6 /usr/lib64/libtinfo.so.5
```



### 编译安装

下载源码，并且配置编译环境

```shell
下载源码，并且配置编译环境
wget https://downloads.mysql.com/archives/get/p/23/file/mysql-5.6.40.tar.gz
tar xzvf mysql-5.6.40.tar.gz
cd mysql-5.6.40
yum install -y ncurses-devel libaio-devel cmake gcc gcc-c++ glibc
```

创建mysql用户

```shell
useradd mysql -s /sbin/nologin -M
```

编译并安装

```shell
[root@localhost mysql-5.6.40]# mkdir /application
#此处为预编译
[root@localhost mysql-5.6.40]# cmake . -DCMAKE_INSTALL_PREFIX=/application/mysql-5.6.40 \
-DMYSQL_DATADIR=/application/mysql-5.6.40/data \
-DMYSQL_UNIX_ADDR=/application/mysql-5.6.40/tmp/mysql.sock \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci \
-DWITH_EXTRA_CHARSETS=all \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_FEDERATED_STORAGE_ENGINE=1 \
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
-DWITHOUT_EXAMPLE_STORAGE_ENGINE=1 \
-DWITH_ZLIB=bundled \
-DWITH_SSL=bundled \
-DENABLED_LOCAL_INFILE=1 \
-DWITH_EMBEDDED_SERVER=1 \
-DENABLE_DOWNLOADS=1 \
-DWITH_DEBUG=0


[root@localhost mysql-5.6.40]# echo $?    #查看上条命令是否运行正确，0表示正确
[root@localhost mysql-5.6.40]# make -j 4  #用四个核心编译会快一些
[root@localhost mysql-5.6.40]# make install
```

创建配置文件

```shell
ln -s /application/mysql-5.6.40/ /application/mysql   #用硬连接来为文件重命名
cd /application/mysql/support-files/
cp my-default.cnf /etc/my.cnf
cp：是否覆盖"/etc/my.cnf"？ y
```

创建启动脚本

```shell
cp mysql.server /etc/init.d/mysqld
```

初始化数据库

```shell
cd /application/mysql/scripts/
yum -y install autoconf
./mysql_install_db --user=mysql --basedir=/application/mysql --datadir=/application/mysql/data/
```

启动数据库

```shell
mkdir /application/mysql/tmp
chown -R mysql.mysql /application/mysql*
/etc/init.d/mysqld start
```

配置环境变量

```shell
vim /etc/profile.d/mysql.sh
export PATH="/application/mysql/bin:$PATH"
source /etc/profile
```

systemd管理mysql

```shell
vim /usr/lib/systemd/system/mysqld.service
[Unit]
Description=MySQL Server
Documentation=man:mysqld(8)
Documentation=https://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target
After=syslog.target
[Install]
WantedBy=multi-user.target
[Service]
User=mysql
Group=mysql
ExecStart=/application/mysql/bin/mysqld --defaults-file=/etc/my.cnf
LimitNOFILE = 5000

vim /etc/my.cnf
 basedir = /application/mysql/
 datadir = /application/mysql/data

systemctl daemon-reload
```

开始mysql服务

```shell
systemctl start mysqld
```

设置mysql密码，并且登录测试

```shell
mysqladmin -uroot password '123456'
mysql -uroot -p123456
mysql> show databases;
mysql> \q
Bye
```



## 二、客户端工具

### mysql

用来**连接**和**管理**数据库     mysql -u<用户名> -p<密码>



### mysqladmin

- “强制回应 (Ping)”服务器
- 关闭服务器
- 创建和删除数据库
- 显示服务器和版本信息
- 显示或重置服务器状态变量
- 设置口令
- 重新刷新授权表
- 刷新日志文件和高速缓存
- 启动和停止复制

```shell
[root@localhost ~]# mysqladmin -uroot -p123456 create hellodb
[root@localhost ~]# mysqladmin -uroot -p123456 drop hellodb
[root@localhost ~]# mysqladmin -uroot -p123456 ping     # 检查服务端状态的
[root@localhost ~]# mysqladmin -uroot -p123456 status     # 服务器运行状态
[root@localhost ~]# mysqladmin -uroot -p123456 status     # 服务器状态 --sleep 2 --count 10 每两秒钟显示⼀次。服务器实时状态⼀共显示10次
Uptime:是mysql正常运行的时间。
Threads:指开启的会话数。
Questions： 服务器启动以来客户的问题(查询)数目  （应该是只要跟mysql作交互：不管你查询表，还是查询服务器状态都问记一次）。
Slow queries：按字面意思是慢查询的意思，不知道mysql认为多久才足够算为长查询，这个先放着。
Opens： 服务器已经打开的数据库表的数量
Flush tables: 服务器已经执行的flush ...、refresh和reload命令的数量。
open tables：通过命令是用的 数据库的表的数量，以服务器启动开始。
Queries per second avg：select语句平均查询时间
[root@localhost ~]# mysqladmin -uroot -p123456 extended-status 显示状态变量
[root@localhost ~]# mysqladmin -uroot -p123456 variables 显示服务器变量
[root@localhost ~]# mysqladmin -uroot -p123456 flush-privileges 数据库重读授权表，等同于reload
[root@localhost ~]# mysqladmin -uroot -p123456 flush-tables 关闭所有已经打开的表
[root@localhost ~]# mysqladmin -uroot -p123456 flush-threds 重置线程池缓存
[root@localhost ~]# mysqladmin -uroot -p123456 flush-status 重置⼤多数服务器状态变量
[root@localhost ~]# mysqladmin -uroot -p123456 flush-logs ⽇志滚动。主要实现⼆进制和中继⽇志滚动
[root@localhost ~]# mysqladmin -uroot -p123456 flush-hosts 清楚主机内部信息
[root@localhost ~]# mysqladmin -uroot -p123456 kill 杀死线程
[root@localhost ~]# mysqladmin -uroot -p123456 refresh 相当于同时执⾏flush-hosts flush-logs
[root@localhost ~]# mysqladmin -uroot -p123456 shutdown 关闭服务器进程
[root@localhost ~]# mysqladmin -uroot -p123456 version 服务器版本以及当前状态信息
[root@localhost ~]# mysqladmin -uroot -p123456 start-slave 启动复制，启动从服务器复制线程
[root@localhost ~]# mysqladmin -uroot -p123456 stop-slave 关闭复制线程
```



### mysqldump

- 备份数据库和表的内容

```shell
mysqldump -uroot -p --all-databases > /backup/mysqldump/all.db
# 备份所有数据库
mysqldump -uroot -p test > /backup/mysqldump/test.db
# 备份指定数据库
mysqldump -uroot -p  mysql db event > /backup/mysqldump/2table.db
# 备份指定数据库指定表(多个表以空格间隔)
mysqldump -uroot -p test --ignore-table=test.t1 --ignore-table=test.t2 > /backup/mysqldump/test2.db
# 备份指定数据库排除某些表
```

- 还原的方法

```shell
mysqladmin -uroot -p create db_name 
mysql -uroot -p  db_name < /backup/mysqldump/db_name.db
# 注：在导入备份数据库前，db_name如果没有，是需要创建的； 而且与db_name.db中数据库名是一样的才可以导入。
mysql > use db_name
mysql > source /backup/mysqldump/db_name.db
# source也可以还原数据库
```



## 三、MySQL架构

### 客户端与服务器模型

- mysql是一个典型的C/S服务结构
  - mysql自带的客户端程序（/application/mysql/bin）
  - mysql
  - mysqladmin
  - mysqldump

连接方式：tcp连接和socket连接

mysql -uroot -p123456 -h127.0.0.1 #TCP连接方式   

mysql -uroot -p123456 -S /application/mysql/tmp/mysql.sock #套接字连接方式



### MySQL服务器构成

#### mysqld服务结构

架构：主要包括 Connection Pool 链接池、Service & utilities 服务治理和工具、SQL interface SQL接口、Parser 解析器、Optimizer 查询优化器、Caches 缓存等模块。

**connection pool 链接池**

​	Connection Pool 连接池负责处理和存储数据库和客户端之间创建的链接。一个线程负责管理一个链接。连接池包括用户认证模块，该模块对用户的登录身份进行认证、认证和安全管理，即用户执行操作权限验证。

**service and utilities 服务治理和工具集**

Service & utilities 是服务治理&工具集，包括Backup & Restore（数据备份和还原）、Security（安全）、Replication（复制）、Cluster（聚簇）、Partitioning（分区）、Workbench（工作台）等等集群管理服务和工具。
**sql interface sql接口** 

SQL interface负责接收客户端发送的各种 SQL 语句，比如 DML、DDL 和 Stored Procedure （存储过程）、Triggers（触发器）、Views（视图）等。

**parser 解析器**

Parser 解析器会对 SQL 语句进行Syntactic（语法）、Lexical（词汇）、Semantic（语义）等等解析生成解析树。

**optimizer 查询优化器**

Optimizer查询优化器将基于解析树生成执行计划，选择适当的索引，然后根据执行计划执行SQL语言，并与每个存储引擎交互。

**caches 缓存**

Caches 缓存包括各种存储引擎的缓存，例如InnoDB的缓冲池 Buffer Pool 和MyISAM的 key buffer 密钥缓冲区。缓存还缓存一些权限，包括 Session 会话级缓存。

**storage engines 存储引擎**

存储引擎包括MyISAM、InnoDB、Archive和Memory。MySQL是一个插件存储引擎。只要正确定义了与MySQL Server的接口，任何引擎都可以访问MySQL，这也是MySQL受欢迎的原因之一。

存储引擎底部是物理存储层，是文件的物理存储层，包括Binary（二进制日志）、Error（错误日志）、Redo log、Undo log、Data（数据文件）、Index、慢查询日志、全日志等。

<img src="E:\英格\图片\mysql结构.png" alt="mysql结构" style="zoom:67%;" />

- 连接层
  - 验证用户的合法性(ip,端口,用户名)
  - 提供两种连接方式(socket,TCP/IP)
  - 验证操作权限
  - 提供一个与SQL层交互的专用线程
- SQL层
  - 接受连接层传来的SQL语句
  - 检查语法
  - 检查语义(DDL,DML,DQL,DCL)
  - 解析器，解析SQL语句，生成多种执行计划
  - 优化器，根据多种执行计划，选择最优方式
  - 执行器，执行优化器传来的最优方式SQL
    - 提供与存储引擎交互的线程
    - 接收返回数据，优化成表的形式返回SQL
  - 将数据存入缓存
  - 记录日志，binlog
- 存储引擎
  - 接收上层的执行结构
  - 取出磁盘文件和相应数据
  - 返回给SQL层，结构化之后生成表格，由专用线程返回给客户端



### mysql逻辑结构

MySQL的逻辑对象：做为管理人员或者开发人员操作的对象

- 库
- 表：元数据+真实数据行
- 元数据：列+其它属性（行数+占用空间大小+权限）
- 列：列名字+数据类型+其他约束（非空、唯一、主键、非负数、自增长、默认值）



### mysql的物理结构

- MySQL的最底层的物理结构是数据文件，也就是说，存储引擎层，打交道的文件，是数据文件。
- 存储引擎分为很多种类
- 不同存储引擎的区别：存储方式、安全性、性能

myisam:

- mysql自带的表部分就是使用的myisam

innodb:

- 自己创建一个表，在编译的时候已经默认指定使用innodb



### 段、区、页（块）

- 段：理论上一个表就是一个段，由多个区构成，（分区表是一个分区一个段）
- 区：物理内存管理的基本单位，操作系统将物理内存划分为多个区块
- 页：页是虚拟内容管理的基本单位，与`区`构成映射关系



## 四、应用

### Mysql用户权限管理

#### Mysql用户基础操作

- Mysql用户的作用

  - 登录Mysql数据库  mysql -u<用户名>  -p<密码>
  - 管理数据库对象

- Mysql用户管理

  - 创建用户:create user
  - 删除用户:delete user drop user
  - 修改用户:update

- 用户的定义

  - username@'主机域'
  - 主机域:可以理解为是Mysql登录的白名单
  - 主机域格式：
    - 10.1.1.12
    - 10.1.0.1%
    - 10.1.0.%
    - 10.1.%.%
    - %
    - localhost
    - 192.168.1.1/255.255.255.0

- 刚装完mysql数据库该做的事情

  先truncate mysql.user；再重启mysql发现进不去了

  - 设定初始密码

```shell
mysqladmin -uroot password '123456'
* 忘记root密码
/etc/init.d/mysqld stop
mysqld_safe --skip-grant-tables --skip-networking    #skip-networking禁止掉3306端口，不允许网络登录
# 修改root密码
update user set password=PASSWORD('123456') where user='root' and host='localhost';
flush privileges;
```



#### 用户管理

- 创建用户

```sql
mysql> create user user01@'192.168.175.%' identified by '123456';
```

- 查看用户

```sql
mysql> select user,host from mysql.user;
```

- 查看当前用户

  ```sql
  mysql> select user();
  ```

- 删除用户

```sql
mysql> drop user user01@'192.168.175.%';
```

- 修改密码

```sql
mysql> set password=PASSOWRD('123456')
mysql> update user set password=PASSWORD('user01') where user='root' and host='localhost';
```



用户权限介绍

- 权限

```sql
INSERT,SELECT, UPDATE, DELETE, CREATE, DROP, RELOAD, SHUTDOWN,  PROCESS, FILE, REFERENCES, INDEX, ALTER, SHOW DATABASES, SUPER, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, CREATE USER, EVENT, TRIGGER, CREATE TABLESPACE
```

1. **INSERT**: 用于向数据库表中插入新数据。
2. **SELECT**: 用于从数据库表中查询和检索数据。
3. **UPDATE**: 用于更新数据库表中的现有数据。
4. **DELETE**: 用于从数据库表中删除数据。
5. **CREATE**: 用于创建数据库对象,如表、视图、存储过程等。
6. **DROP**: 用于删除数据库对象,如表、视图、存储过程等。
7. **RELOAD**: 用于重新加载 MySQL 服务器的配置文件。
8. **SHUTDOWN**: 用于关闭 MySQL 服务器。
9. **PROCESS**: 用于查看和管理 MySQL 服务器的进程。
10. **FILE**: 用于读写服务器上的文件。
11. **REFERENCES**: 用于创建外键约束。
12. **INDEX**: 用于创建和管理数据库索引。
13. **ALTER**: 用于修改数据库对象的结构,如表、视图、存储过程等。
14. **SHOW DATABASES**: 用于查看当前 MySQL 服务器上的所有数据库。
15. **SUPER**: 是一种特殊的权限,允许用户执行一些高级操作,如查看和杀死其他用户的进程。
16. **CREATE TEMPORARY TABLES**: 用于创建临时表。
17. **LOCK TABLES**: 用于手动锁定数据库表。
18. **EXECUTE**: 用于执行存储过程和函数。
19. **REPLICATION SLAVE**: 用于配置数据库复制时的从库权限。
20. **REPLICATION CLIENT**: 用于获取数据库复制状态信息。
21. **CREATE VIEW**: 用于创建数据库视图。
22. **SHOW VIEW**: 用于查看数据库视图的定义。
23. **CREATE ROUTINE**: 用于创建存储过程和函数。
24. **ALTER ROUTINE**: 用于修改存储过程和函数。
25. **CREATE USER**: 用于创建新的数据库用户账号。
26. **EVENT**: 用于创建和管理数据库事件。
27. **TRIGGER**: 用于创建和管理数据库触发器。
28. **CREATE TABLESPACE**: 用于创建数据库表空间。

- 每次设定只能有一个属主，没有属组或其他用户的概念

```sql
grant     all privileges    on     *.*    to   user01@''192.168.175.%''  identified by    ''123'';
                权限               作用对象          归属               密码
```

作用对象解释

- “  *   .  *    ” [当前MySQL实例中所有库下的所有表]
- wordpress.* [当前MySQL实例中wordpress库中所有表（单库级别）]
- wordpress.user [当前MySQL实例中wordpress库中的user表（单表级别）]



### Mysql启动关闭流程

- 启动

  /usr/lib/systemd/system/mysqld.service告知系统从哪执行mysqld

  mysld_safe是个脚本，mysqld才是binary

```shell
/etc/init.d/mysqld start ------> mysqld_safe ------> mysqld
```

- 关闭

```shell
/etc/init.d/mysqld stop 
mysqladmin -uroot -p123456 shutdown
kill -9 pid ?    #一般不杀进程
killall mysqld ?
pkill mysqld ?   
pkill mysqld_safe
```



### Mysql实例初始化配置

- 预编译：cmake去指定，硬编译到程序当中去
- 在命令行设定启动初始化配置

```shell
--skip-grant-tables 
--skip-networking
--datadir=/application/mysql/data
--basedir=/application/mysql
--defaults-file=/etc/my.cnf
--pid-file=/application/mysql/data/db01.pid
--socket=/application/mysql/data/mysql.sock
--user=mysql
--port=3306
--log-error=/application/mysql/data/db01.err
```

- 初始化配置文件（/etc/my.cnf）
- --defaults-file：默认配置文件
  - 如果使用./bin/mysqld_safe 守护进程启动mysql数据库时，使用了 --defaults-file=<配置文件的绝对路径>参数，这时只会使用这个参数指定的配置文件。

```shell
#cmake：
socket=/application/mysql/tmp/mysql.sock
#命令行：
--socket=/tmp/mysql.sock
#配置文件：
/etc/my.cnf中[mysqld]标签下：socket=/opt/mysql.sock
#default参数：
--defaults-file=/tmp/a.txt配置文件中[mysqld]标签下：socket=/tmp/test.socket
```



### MySQL多实例配置

- 多套后台进程+线程+内存结构
- 多个配置文件
  - 多端口
  - 多socket文件
  - 多个日志文件
  - 多个server_id
- 多套数据

实战配置

```shell
#创建数据目录
mkdir -p /data/330{7..9}
#创建配置文件
touch /data/330{7..9}/my.cnf
touch /data/330{7..9}/mysql.log
#编辑3307配置文件
vim /data/3307/my.cnf
[mysqld]
basedir=/application/mysql
datadir=/data/3307/data
socket=/data/3307/mysql.sock
log_error=/data/3307/mysql.log
log-bin=/data/3307/mysql-bin
server_id=7
port=3307
[client]
socket=/data/3307/mysql.sock
#编辑3308，3309配置文件，与3307类似，仅需修改330x处

#初始化3307，3308，3309数据,每条命令需要间隔若干秒，否则失败
/application/mysql/scripts/mysql_install_db \
--user=mysql \
--defaults-file=/data/3307/my.cnf \
--basedir=/application/mysql --datadir=/data/3307/data
#修改目录权限
chown -R mysql.mysql /data/330*
#启动多实例
mysqld_safe --defaults-file=/data/3307/my.cnf &
mysqld_safe --defaults-file=/data/3308/my.cnf &
mysqld_safe --defaults-file=/data/3309/my.cnf &
#设置每个实例的密码
#mysqladmin -S /data/3307/mysql.sock -uroot password '123456'
#查看server_id
mysql -S /data/3307/mysql.sock -e "show variables like 'server_id'"
mysql -S /data/3308/mysql.sock -e "show variables like 'server_id'"
mysql -S /data/3309/mysql.sock -e "show variables like 'server_id'"
# 进入单独的mysql实例
mysql -S /data/3307/mysql.sock -uroot
# 关闭实例，如果设置了密码登录就需要密码了 用参数-p
mysqladmin -S /data/3307/mysql.sock -uroot shutdown
mysqladmin -S /data/3308/mysql.sock -uroot shutdown
mysqladmin -S /data/3309/mysql.sock -uroot shutdown
```



## 五、SQL语句

- SQL是结构化的查询语句
- SQL的种类
  - DDL:数据定义语句
  - DCL:数据控制语言
  - DML:数据操作语言
  - DQL:数据查询语言

delete 是删除数据，drop是删除数据库或者表



### DDL数据定义语句

- 对库或者表进行操作的语句
- 创建数据库

```sql
create database db01;
# 创建数据库
create database DB01;
# 数据库名区分大小写(注意windows里面不区分)
show variables like 'lower_case_table_names';
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| lower_case_table_names | 0     |
+------------------------+-------+

具体lower_case_table_names 取值详解如下：
0：区分大小写
1：不区分大小写，存储的时候，将表名都转换成小写
2：不区分大小写，存储的时候，表名跟建表时大小写保持一致

show databases;               # 查看数据库(DQL)
show create database db01;    # 查看创建数据库语句
help create database;        # 查看创建数据库语句帮助
create database db02 charset utf8;# 创建数据库的时候添加属性
```

- 删除数据库

```sql
drop database db02;         # 删除数据库db02
```

- 修改定义库

```sql
alter database db01 charset utf8;
```

- 创建表

```sql
use db01;    # 进入库中才能创建表
create table student(
sid int,
sname varchar(20),
sage tinyint,
sgender enum('m','f'),
comtime datetime
);# 创建表，并且定义每一列
```

- 数据类型(下面有完整的)

  ![MySQL数据类型](E:\英格\图片\MySQL数据类型.jpg)

```sql
drop table student; # 删除表

# 带数据属性创建学生表
create table student(
sid int not null primary key auto_increment comment '学号',
sname varchar(20) not null comment '学生姓名',
sgender enum('m','f') not null default 'm' comment '学生性别',
cometime datetime not null comment '入学时间'
)charset utf8 engine innodb;


show create table student; # 查看建表语句
show tables;  # 查看表
desc student;# 查看表中列的定义信息
```

- 数据属性

  ![MySQL数据属性](E:\英格\图片\MySQL数据属性.jpg)

- 删除表

```sql
drop table student;
```

- 修改表的定义

```sql
alter table student rename stu;                 # 修改表名
alter table stu add age int;                    # 添加列和列数据类型的定义
alter table stu add test varchar(20),add qq int;# 添加多个列
alter table stu add sid varchar(20) primary key auto_increment ;# 添加属性
alter table stu add classid varchar(20) first;  # 指定位置进行添加列(表首)
alter table stu add phone int after age;        # 指定位置进行添加列（指定列）
alter table stu drop qq [后接要删除的属性];                        # 删除指定的列及定义
alter table stu drop sid not null;
alter table stu modify sid varchar(20)  [同add];         # 修改列及定义（列属性）
alter table stu change phone telphone char(20); # 修改列及定义（列名及属性）
```



### DCL数据控制语言

- DCL是针对权限进行控制
- 授权

```sql
grant all on *.* to root@'192.168.175.%' identified by '123456'
# 授予root@'192.168.175.%'用户所有权限(非超级管理员)
grant all on *.* to root@'192.168.175.%' identified by '123456' with grant option;
# 授权一个超级管理员
max_queries_per_hour：一个用户每小时可发出的查询数量
max_updates_per_hour：一个用户每小时可发出的更新数量
max_connections_per_hour：一个用户每小时可连接到服务器的次数
max_user_connections：允许同时连接数量
```

- 收回权限

```sql
revoke select on *.* from root@'192.168.175.%';# 收回select权限
show grants for root@'192.168.175.%';# 查看权限
```



### DML数据操作语言

- 操作表中的数据
- 插入数据

```sql
insert into stu valus('linux01',1,NOW(),'zhangsan',20,'m',NOW(),110,123456);# 基础用法，插入数据
insert into stu(classid,birth.sname,sage,sgender,comtime,telnum,qq) values('linux01',1,NOW(),'zhangsan',20,'m',NOW(),110,123456);# 规范用法，插入数据
insert into stu(classid,birth.sname,sage,sgender,comtime,telnum,qq) values('linux01',1,NOW(),'zhangsan',20,'m',NOW(),110,123456),
('linux02',2,NOW(),'zhangsi',21,'f',NOW(),111,1234567);# 插入多条数据
insert into stu(classid,birth.sname,sage,sgender,comtime,qq)
values('linux01',1,NOW(),'zhangsan',20,'m',NOW(),123456)；#少了电话
```

- 更新数据

```sql
update student set sgender='f';# 不规范，会导致全表修改
update student set sgender='f' where sid=1;# 规范update修改
update student set sgender='f' where 1=1;# 如果非要全表修改
update mysql.user set password=PASSWORD('123456') where user='root' and host='localhost';# 修改密码，需要刷新权限flush privileges
```

- 删除数据

```sql
delete from student;# 不规范
delete from student where sid=3;# 规范删除（危险）
truncate table student;# DDL清空表中的内容，恢复表原来的状态，自增重新从1开始，否则delete依旧正常增长
```

- 使用伪删除（标记状态）
  - 有时部分数据暂时不用时，标记出来，查询时不显示

```sql
alter table student add status enum(1,0) default 1;# 额外添加一个状态列
update student set status='0' where sid=1;# 使用update
select * from student where status=1;# 应用查询存在的数据
```



### DQL数据查询语言

```sql
SHOW TABLES;    #查看创建的表以及架构
desc / describe student #查看student表结构
SELECT * FROM student;   # 从student表中读取所有的数据
select distinct t_depart from teacher; #筛选出老师属于哪些专业，不能重复
select * from score where sc_degree between 60 and 80;  #筛选出score表中60分到80分的学生

select * from score where sc_degree=85 or sc_degree =86 or sc_degree =87; #筛选得到85或86或87分的同学
或者
select * from score where sc_degree in(85,86,87);

select * from student where s_class = 95031 or s_sex = '女'; #找到学生，要求班级为95031号或者女生

#order by,asc 升序 desc 降序
select * from student order by s_class desc; #以班级号码倒叙排列所有学生
select * from score order by c_no asc,sc_degree desc;#以升序排列班级号码，倒叙排列成绩

#count 统计数量
select count(*) from student where s_class='95033'; #统计班级号为95033的同学总共有多少

#子查询 相当于把查询语句嵌套进查询条件
select s_no,c_no from score where sc_degree = (select max(sc_degree) from score); # .求最高分学生的学号和课程编号，max()  

# 切片 类似python里的
select s_no,c_no from score where sc_degree order by sc_degree desc limit 0,1;  #展示成绩表中，倒叙排列成绩，只显示学号和班级号，最后切片，只显示第一个

#计算平均成绩（avg,group by）
select c_no,avg(sc_degree) from score group by c_no;  #查询每门课的平均成绩

# 分组条件及模糊查询（group by having,like）
select c_no,avg(sc_degree),count(c_no) from score group by c_no 
having count(c_no) >2 and c_no like '3%';  # 查询score表中至少有两名学生选修并以3开头的课程的平均分数

# 范围查询（between and）
select s_no,sc_degree from score where sc_degree >70 and sc_degree <90; #查询分数大于70，小于90的s_no列
或者
select s_no,sc_degree from score where sc_degree between 70 and 90;

select * from score where sc_degree between 60 and 80;  #筛选出score表中60分到80分的学生

# 多表查询(寻找相同条件)
select s_name,c_no,sc_degree from student,score where student.s_no=score.s_no; #查询所有学生的s_name\c_no\sc_degree 
select s_no,c_name,sc_degree from score,course where score.c_no=course.c_no; #查询所有学生的s_no, c_name, sc_degree列

# 范围查询 in 
select avg(sc_degree) from score where s_no in (select s_no from student where s_class='95031') group by c_no;

#年份函数(year)
select s_no,s_name,s_birthday from student
where year(s_birthday) in (select year(s_birthday) from student where s_no in (101,108));
#查询所有学号为108.101的同学同年出生的所有学生的s_no,s_name和s_birthday

#求并集操作(union)
#查询'计算机系'与'电子工程系' 不同职称的教师的t_name和rof
select t_name,t_rof from teacher where t_depart='计算机系' 
and t_rof not in (select t_rof from teacher where t_depart='电子工程系')
union
select t_name,t_rof from teacher where t_depart='电子工程系' 
and t_rof not in (select t_rof from teacher where t_depart='计算机系');

#多个值比较多个值：至少（any）
#查询选修编号为"3-105"课程且成绩至少高于选修编号为'3-245'同学的c_no,s_no和sc_degree,并且按照sc_degree从高到地次序排序
select c_no,s_no,sc_degree from score where c_no='3-105' and sc_degree > any(select sc_degree from score where c_no='3-245') order by sc_degree desc;

#多个值比较多个值：所有（all）
#查询选修编号为"3-105"且成绩高于选修编号为"3-245"课程的同学c_no.s_no和sc_degree
select c_no,s_no,sc_degree from score where c_no='3-105' and sc_degree > all(select sc_degree from score where c_no='3-245');

#使用别名(as)
#查询所有教师和同学的 name ,sex, birthday
select s_name as name,s_sex as sex,s_birthday as birthday from student 
union 
select t_name as name,t_sex as sex,t_birthday as birthday from teacher;

# 模糊查询取反(not like,%)
# 查询student 表中不姓"王"的同学的记录
select * from student where s_name not like '王%';

#连接查询
# 内连接（inner join …… on）
select * from person inner join card on person.cardId=card.id;

#外连接
# 左外连接(left join ...... on) 左外连接会把左边的表的数据全部取出来 ，右边表如果没有就用NULL补上
select * from person left join card on person.cardId=card.id;

# 右外连接(right join ...... on) 右外连接会把左边的表的数据全部取出来，左边表如果没有就用NULL补上
select * from person right join card on person.cardId=card.id;

# 全连接（full join）
select * from person full join card on person.cardId=card.id;
```



## 六、数据类型

数值、字符、二进制、时间

### 数值

![数值类型](E:\英格\图片\数值类型.jpg)



### 字符

![字符类型](E:\英格\图片\字符类型.jpg)



### 二进制

![二进制](E:\英格\图片\二进制.jpg)



### 列属性

![列属性](E:\英格\图片\列属性.jpg)



## 七、索引

索引能加快数据检索速度，提高查询性能，通过唯一索引保证数据的唯一性。

### 索引分类

#### 按数据结构分类

mysql主要使用了B树、B+树

**B树**

是一种多路平衡查找树，满足平衡二叉树的规则，但可以有多个子树。子树的数量取决于关键字的数量，比如根节点有两个关键字，那么它能拥有的子树数量是关键字数量加 1。在存储同样数据量的情况下，平衡二叉树的高度要大于 B 树。

**B+树**

B 树一个节点里存的是数据，而 B+树存储的是索引（地址），所以B树里一个节点存不了很多个数据，但是 B+树一个节点能存很多索引，B+树叶子节点存所有的数据。
B+树的叶子节点是数据阶段用一个链表串联起来，便于范围查找。
B+树节点存储的是索引，在单个节点存储容量有限的情况下，单节点也能存储大量索引，使得整个 B+树高度降低，减少磁盘 IO。其次，B+树的叶子节点是真正数据存储的地方，叶子节点用有序链表连接起来，在范围查找时效率更高。因此MySQL的索引用的就是 B+树，B+树在查找效率、范围查找中都有着非常不错的性能。



#### 按功能分类

主键索引：特殊的唯一索引，不允许有空值

就像是一本书的目录页,可以快速定位到某一页的内容。每一行数据都有一个独一无二的标识(主键),建立主键索引可以迅速找到对应的行。主键索引是最重要的索引类型,能大大提高查询效率。

普通索引：最基本的索引，没有任何限制。

就像是在一本书的页边做了一些标记,方便以后快速找到感兴趣的内容。普通索引不要求值必须唯一,可以建立在单个列或多个列上。普通索引可以提高查询速度,但更新和插入数据时会稍微损耗一些性能。

唯一索引：索引列的值必须唯一，允许有空值。

可以认为是一种特殊的主键,它要求索引列的值必须是唯一的。唯一索引既可以提高查询效率,又能确保数据的完整性,防止出现重复数据。不过,与主键不同的是,唯一索引允许出现空值。



### 索引管理

索引建立在表的列上(字段)的。在where后面的列建立索引才会加快查询速度。pages<---索引（属性）<----查数据。

```sql
alter table test add index index_name(name);#创建索引
create index index_name on test(name);#创建索引
desc table;#查看索引
show index from table;#查看索引
alter table test drop key index_name;#删除索引
alter table student add unique key uni_xxx(xxx);#添加唯一性索引
alter table studen add primary key(column);#添加主键索引
select count(*) from city;#查看表中数据行数
select count(distinct(name)) from city;#查看去重数据行数
create unique index idx_name on table_name(column_name); #添加唯一索引
```



### 前缀索引

前缀索引是对列的前缀部分创建索引，适用于较长的字符串列。

- 根据字段的前N个字符建立索引

```sql
alter table test add index idx_name(name(10));
#比如表里很多城市以D开头，Dalas Des'Moine Denver Detroit，以字母D建立的索引会加快检索速度
```

- 避免对大列建索引
- 如果有，就使用前缀索引



### 联合索引

联合索引是对多个列组合创建的索引。

- 多个字段建立一个索引
- 原则：把最常用来做为条件查询的列放在最前面

```sql
create table people (id int,name varchar(20),age tinyint,money int ,gender enum('m','f'));
#创建people表
alter table people add index  idx_gam(gender,age,money);
#创建联合索引
```



### explain分析

#### explain详解

EXPLAIN 是 SQL 中一个非常有用的关键字,它可以帮助我们分析和优化 SQL 查询的执行计划。使用EXPLAIN语句可以获取 SQL 查询的执行计划,了解查询是如何执行的,从而帮助我们识别并优化查询性能瓶颈。

-  SQL 语句前加上 `EXPLAIN` 关键字即可,例如: `EXPLAIN SELECT * FROM users WHERE id = 1;`

```sql
mysql> explain select name,countrycode from city where id=1;
```

**输出信息**:

- `id`: 查询编号,表示查询执行的顺序。如果有子查询,每个子查询都会分配一个单独的 id。
- `select_type`: 查询类型,包括 SIMPLE、PRIMARY、SUBQUERY 等。
- `table`: 正在访问的数据表名称。
- `partitions`: 匹配的分区情况。
- `type`: 访问类型,包括 system、const、eq_ref、ref、range、index、ALL 等,访问类型由快到慢依次排列。
- `possible_keys`: 可能使用的索引。
- `key`: 实际使用的索引名称。
- `key_len`: 使用索引的长度。
- `ref`: 列上的比较操作。
- `rows`: 估计要读取的行数。
- `filtered`: 条件过滤后剩余的行占原始行的比例。
- `Extra`: 一些额外的信息,如 Using index、Using where 等。

**应用场景**:

- 查看查询计划,了解 MySQL 是如何执行 SQL 查询的。
- 分析查询瓶颈,如果 `type` 是 `ALL` 表示全表扫描,可能需要优化索引。
- 检查是否使用了正确的索引,`possible_keys` 和 `key` 列可以告诉你 MySQL 选择了哪个索引。
- 估算查询的行数,`rows` 列给出了 MySQL 估计的行数,可以帮助判断查询的效率。



#### MySQL查询数据的方式

全表扫描（在explain语句结果中type为ALL，例如select * from XXX）

- 业务确实要获取所有数据
- 不走索引导致的全表扫描：没索引，索引创建有问题，语句有问题
- 生产中,mysql在使用全表扫描时的性能是极其差的，所以MySQL尽量避免出现全表扫描



索引扫描

- 常见的索引扫描类型：index，range，ref，eq_ref，const，system，null
- 从前到后，性能从最差到最好，我们认为至少要达到range级别



##### index

- Full Index Scan，index与ALL区别为index类型只遍历索引树。

##### range

- 索引范围扫描，对索引的扫描开始于某一点，返回匹配值域的行。显而易见的索引范围扫描是带有between或者where子句里带有<,>查询。

```sql
mysql> alter table city add index idx_city(population);
mysql> explain select * from city where population>30000000;
```

##### ref

- 用非唯一索引扫描或者唯一索引的前缀扫描，返回匹配某个单独值的记录行。

```sql
mysql> alter table city drop key idx_code;
mysql> explain select * from city where countrycode='chn';
mysql> explain select * from city where countrycode in ('CHN','USA');
mysql> explain select * from city where countrycode='CHN' union all select * from city where countrycode='USA';
```

##### eq_ref

- 类似ref，区别就在使用的索引是唯一索引，对于每个索引键值，表中只有一条记录匹配，简单来说，就是多表连接中使用primary key或者 unique key作为关联条件A

```sql
join B 
on A.sid=B.sid
```

##### const、system

- 当MySQL对查询某部分进行优化，并转换为一个常量时，使用这些类型访问。
- 如将主键置于where列表中，MySQL就能将该查询转换为一个常量

```sql
mysql> explain select * from city where id=1000;
```

##### NULL

- MySQL在优化过程中分解语句，执行时甚至不用访问表或索引，例如从一个索引列里选取最小值可以通过单独索引查找完成。

```sql
mysql> explain select * from city where id=1000000000000000000000000000;
```



##### Extra（扩展）

在 MySQL 中,当你执行一条 SQL 语句时,结果集通常包含一个名为 "Extra" 的列,它提供了一些额外的信息,可以帮助我们更好地理解 SQL 语句的执行过程。下面是一些常见的 "Extra" 信息及其含义:

1. No index used
   - 表示在执行 SQL 语句时,数据库没有使用任何索引。这可能会降低查询性能。
2. Using index
   - 表示在执行 SQL 语句时,数据库使用了索引来优化查询。这通常意味着查询性能较好。
3. Using index condition
   - 表示在执行 SQL 语句时,数据库使用了索引条件下推(Index Condition Pushdown)技术来优化查询。这可以减少不必要的行访问。
4. Using index for group-by
   - 表示在执行 GROUP BY 语句时,数据库使用了索引来优化分组操作。
5. Using temporary
   - 表示在执行 SQL 语句时,数据库创建了一个临时表来存储中间结果。这可能会降低查询性能。
6. Using filesort
   - 表示在执行 SQL 语句时,数据库需要对结果集进行额外的排序操作。这可能会降低查询性能。
7. Using join buffer (Block Nested Loop)
   - 表示在执行 JOIN 查询时,数据库使用了连接缓冲区来优化连接操作。
8. Impossible WHERE noticed after reading const tables
   - 表示在执行 SQL 语句时,数据库发现 WHERE 条件永远为 false,因此直接返回空结果集。
9. No tables used
   - 表示在执行 SQL 语句时,数据库没有访问任何表,例如执行 `SELECT 1;`。
10. Select tables optimized away
    - 表示在执行 SQL 语句时,数据库优化掉了部分表的访问,例如在某些情况下,数据库可以直接从索引中获取所需的数据,而无需访问表本身。

这些 "Extra" 信息可以帮助我们了解 SQL 语句的执行过程,识别潜在的性能问题,并针对性地优化查询。在优化 SQL 语句时,我们应该关注那些可能降低性能的信息,例如 "Using temporary"、"Using filesort" 等

- 如果出现Using filesort请检查order by ,group by ,distinct,join 条件列上没有索引

```sql
mysql> explain select * from city where countrycode='CHN' order by population;
```

- 当order by语句中出现Using filesort，那就尽量让排序值在where条件中出现

```sql
mysql> explain select * from city where population>30000000 order by population;
mysql> select * from city where population=2870300 order by population;
* key_len: 越小越好
* 前缀索引去控制，rows: 越小越好
```



##### 建立索引的原则（规范）

- 为了使索引的使用效率更高，在创建索引时，必须考虑在哪些字段上创建索引和创建什么类型的索引。
- 选择唯一性索引
  - 唯一性索引的值是唯一的，可以更快速的通过该索引来确定某条记录。

例如: 学生表中学号是具有唯一性的字段。为该字段建立唯一性索引可以很快的确定某个学生的信息。 如果使用姓名的话，可能存在同名现象，从而降低查询速度。 主键索引和唯一键索引，在查询中使用是效率最高的。

```sql
select count(*) from world.city;
select count(distinct countrycode) from world.city;
select count(distinct countrycode,population ) from world.city;
* 注意：如果重复值较多，可以考虑采用联合索引
```

- 为经常需要排序、分组和联合操作的字段建立索引
  - 经常需要ORDER BY、GROUP BY、DISTINCT和UNION等操作的字段，排序操作会浪费很多时间。
- 为常作为查询条件的字段建立索引
  - 如果某个字段经常用来做查询条件，那么该字段的查询速度会影响整个表的查询速度。
  - 如果经常作为条件的列，重复值特别多，可以建立联合索引
- 尽量使用前缀来索引
  - 如果索引字段的值很长，最好使用值的前缀来索引。例如，TEXT和BLOG类型的字段，进行全文检索会很浪费时间。如果只检索字段的前面的若干个字符，这样可以提高检索速度。
- 限制索引的数目
  - 索引的数目不是越多越好。每个索引都需要占用磁盘空间，索引越多，需要的磁盘空间就越大。
  - 修改表时，对索引的重构和更新很麻烦。越多的索引，会使更新表变得很浪费时间。
- 删除不再使用或者很少使用的索引
  - 表中的数据被大量更新，或者数据的使用方式被改变后，原有的一些索引可能不再需要。数据库管理员应当定期找出这些索引，将它们删除，从而减少索引对更新操作的影响。



**重点关注**：

- 没有查询条件，或者查询条件没有建立索引

```sql
select * from table;
select  * from tab where 1=1;# 全表扫描
```



- 在业务数据库中，特别是数据量比较大的表,是没有全表扫描这种需求。
  - 对用户查看是非常痛苦的。
  - 对服务器来讲毁灭性的。
  - SQL改写成以下语句

```sql
# 情况1
select * from table;
#全表扫描
selec  * from tab order by price limit 10;
#需要在price列上建立索引
# 情况2
select * from table where name='zhangsan'; 
#name列没有索引
1、换成有索引的列作为查询条件
2、将name列建立索引
```



- 查询结果集是原表中的大部分数据，应该是25％以上

```sql
mysql> explain select * from city where population>3000 order by population;
* 如果业务允许，可以使用limit控制
* 结合业务判断，有没有更好的方式。如果没有更好的改写方案就尽量不要在mysql存放这个数据了，放到redis里面。
```

- 索引本身失效，统计数据不真实
  - 索引有自我维护的能力。
  - 对于表内容变化比较频繁的情况下，有可能会出现索引失效。
  - 重建索引就可以解决
- 查询条件使用函数在索引列上或者对索引列进行运算，运算包括(+，-，*等)

```sql
错误的例子：select * from test where id-1=9; 
正确的例子：select * from test where id=10;
```

- 隐式转换导致索引失效.这一点应当引起重视,也是开发中经常会犯的错误

```sql
mysql> create table test (id int ,name varchar(20),telnum varchar(10));
# 例如列类型为整形，非要在检索条件中用字符串
mysql> insert into test values(1,'zs','110'),(2,'l4',120),(3,'w5',119),(4,'z4',112);
mysql> explain select * from test where telnum=120;
mysql> alter table test add index idx_tel(telnum);
mysql> explain select * from test where telnum=120;
mysql> explain select * from test where telnum=120;
mysql> explain select * from test where telnum='120';
```

- <> ，not in 不走索引

```sql
mysql> select * from tab where telnum <> '1555555';
mysql> explain select * from tab where telnum <> '1555555';
```

- 单独的>,<,in 有可能走，也有可能不走，和结果集有关，尽量结合业务添加limit
- or或in尽量改成union

```sql
EXPLAIN  SELECT * FROM teltab WHERE telnum IN ('110','119');
#改写成
EXPLAIN SELECT * FROM teltab WHERE telnum='110'
UNION ALL
SELECT * FROM teltab WHERE telnum='119'
```

- like "%_" 百分号在最前面不走索引

```sql
#走range索引扫描
EXPLAIN SELECT * FROM teltab WHERE telnum LIKE '31%';
#不走索引
EXPLAIN SELECT * FROM teltab WHERE telnum LIKE '%110';
```

%linux%类的搜索需求，可以使用Elasticsearch -------> ELK

- 单独引用联合索引里非第一位置的索引列

```sql
CREATE TABLE t1 (id INT,NAME VARCHAR(20),age INT ,sex ENUM('m','f'),money INT);
ALTER TABLE t1 ADD INDEX t1_idx(money,age,sex);
DESC t1
SHOW INDEX FROM t1
#走索引的情况测试
EXPLAIN SELECT NAME,age,sex,money FROM t1 WHERE money=30 AND age=30  AND sex='m';
#部分走索引
EXPLAIN SELECT NAME,age,sex,money FROM t1 WHERE money=30 AND age=30;
EXPLAIN SELECT NAME,age,sex,money FROM t1 WHERE money=30  AND sex='m'; 
#不走索引
EXPLAIN SELECT  NAME,age,sex,money FROM t1 WHERE age=20
EXPLAIN SELECT NAME,age,sex,money FROM t1 WHERE age=30 AND sex='m';
EXPLAIN SELECT NAME,age,sex,money FROM t1 WHERE sex='m';
```



## 八、MySQL的存储引擎

### 存储引擎简介

Mysql的数据用不同的技术存储在文件(或内存)中，不同的技术拥有不同的存储机制、索引技巧、锁定水平，通过选择不同的技术获得不同的功能，从而改善应用的整体功能，而这些不同的技术和功能就是存储引擎（也称作表引擎）

Mysql支持的存储引擎：InnoDB、MyISAM、MEMORY、CSV、BLACKHOLE、FEDERATED、MRG_MYISAM、 ARCHIVE、PERFORMANCE_SCHEMA。其中NDB和InnoDB使用较多。



#### InnoDB

MySql 5.6 版本默认的存储引擎。InnoDB 是一个事务安全的存储引擎，它具备提交、回滚以及崩溃恢复的功能以 保护用户数据。InnoDB 的行级别锁定以及 Oracle 风格的一致性无锁读提升了它的多用户并发数以及性能。

InnoDB 将用户数据存储在聚集索引中以减少基于主键的普通查询所带来的 I/O 开销。为了保证数据的完整性，InnoDB 还支持外键约束。

应用场景：并发高、数据一致、更新和删除操作较多、事务的完整提交和回滚



**innodb体系结构：**

innodb主要包括 内存池、后台线程以及存储文件

内存池是由多个内存块组成的，主要包括缓存磁盘数据，redolog 缓冲等

后台线程则包括了： Master Thread、IO Thread、Purge Thread等

由 InnoDB 存储引擎实现的表的存储结构文件一般包括表结构文件（.frm）、共享表空间文件（ibdata1）、独占表空间文件（ibd）以及日志文件（redo 文件等）等。



#### MyISAM(读、插入操作)

MyISAM既不支持事务、也不支持外键、其优势是访问速度快，但是表级别的锁定限制了它在读写负载方面的性能， 因此它经常应用于只读或者以读为主的数据场景。

应用场景：读操作和插入操作为主



#### Memory（内存存储数据，快速定位记录）

在内存中存储所有数据，应用于对非关键数据由快速查找的场景。Memory类型的表访问数据非常快，因为它的数据 是存放在内存中的，并且默认使用HASH索引，但是一旦服务关闭，表中的数据就会丢失

应用场景：快速定位记录

- MEMORY是MySQL中一类特殊的存储引擎。它使用存储在内存中的内容来创建表，而且数据全部放在内存中。这些特性与前面的两个很不同。
- 每个基于MEMORY存储引擎的表实际对应一个磁盘文件。该文件的文件名与表名相同，类型为frm类型。该文件中只存储表的结构。而其数据文件，都是存储在内存中，这样有利于数据的快速处理，提高整个表的效率。值得注意的是，服务器需要有足够的内存来维持MEMORY存储引擎的表的使用。如果不需要了，可以释放内存，甚至删除不需要的表。
- MEMORY默认使用哈希索引。速度比使用B型树索引快。当然如果你想用B型树索引，可以在创建索引时指定。
- 注意，MEMORY用到的很少，因为它是把数据存到内存中，如果内存出现异常就会影响数据。如果重启或者关机，所有数据都会消失。因此，基于MEMORY的表的生命周期很短，一般是一次性的。



#### BLACKHOLE（只接收不保存数据）

黑洞存储引擎，类似于 Unix 的 /dev/null，Archive 只接收但却并不保存数据。对这种引擎的表的查询常常返回一个空集。这种表可以应用于 DML 语句需要发送到从服务器，但主服务器并不会保留这种数据的备份的主从配置中。



#### CSV

CSV（Comma-Separated Values，逗号分隔值）存储引擎是一种 **基于文本文件** 的存储引擎，**适用于数据导入/导出**，但不支持事务、索引、外键等高级功能。可以直接使用文本编辑器或 Excel 打开 CSV 文件进行查看或编辑。

它的表是以逗号分隔的文本文件。CSV表允许你以CSV格式导入导出数据，以相同的读和写的格式和脚本和应用交互数据。由于CSV表没有索引，你最好是在普通操作中将数据放在InnoDB表里，只有在导入或导出阶段 使用一下CSV表。



#### NDB

(又名 NDBCLUSTER)——这种集群数据引擎尤其适合于需要最高程度的正常运行时间和可用性的应用。注意：NDB 存 储引擎在标准 MySql 5.6 版本里并不被支持。它主要用于构建高可用性、高可扩展性和高性能的分布式数据库集群。NDB Cluster是MySQL的一个特殊版本，专门设计用于处理大规模的分布式数据存储和处理需求。

目前能够支持 MySql 集群的版本有：基于 MySql 5.1 的 MySQL Cluster NDB 7.1；基于 MySql 5.5 的 MySQL Cluster NDB 7.2；基于 MySql 5.6 的 MySQL Cluster NDB 7.3。同样基于 MySql 5.6 的 MySQL Cluster NDB 7.4



#### Merge（超大规模数据场景）

允许MySql DBA或开发者将一系列相同的MyISAM 表进行分组，并把它们作为一个对象进行引用。适用于超大规模数据场景，如数据仓库。



#### Federated（分布式）

提供了从多个物理机上联接不同的 MySql 服务器来创建一个逻辑数据库的能力。适用于分布式或者数据市场的场景。



#### Example（保存例子）

这种存储引擎用以保存阐明如何开始写新的存储引擎的 MySql 源码的例子。它主要针对于有兴趣的开发人员。这 种存储引擎就是一个啥事也不做的 "存根"。你可以使用这种引擎创建表，但是你无法向其保存任何数据，也无法从 它们检索任何索引。



#### **InnoDB和MyISAM区别**

两种存储引擎的大致区别表现在：

- InnoDB支持事务，MyISAM不支持，这一点是非常之重要。事务是一种高级的处理方式，如在一些列增删改中只要哪个出错还可以回滚还原，而MyISAM就不可以了。
- MyISAM适合查询以及插入为主的应用。
- InnoDB适合频繁修改以及涉及到安全性较高的应用。
- InnoDB支持外键，MyISAM不支持。
- 从MySQL5.5.5以后，InnoDB是默认引擎。
- InnoDB不支持FULLTEXT类型的索引。
- InnoDB中不保存表的行数，如select count(*) from table时，InnoDB需要扫描一遍整个表来计算有多少行，但是MyISAM只要简单的读出保存好的行数即可。注意的是，当count()语句包含where条件时MyISAM也需要扫描整个表。
- 对于自增长的字段，InnoDB中必须包含只有该字段的索引，但是在MyISAM表中可以和其他字段一起建立联合索引。
- DELETE FROM table时，InnoDB不会重新建立表，而是一行一行的删除，效率非常慢。MyISAM则会重建表。
- InnoDB支持行锁（某些情况下还是锁整表，如 update table set a=1 where user like '%lee%'。



### 查看当前MySQL的存储引擎

```sql
mysql> show engines
#查看当前MySQL支持的存储引擎类型
mysql> select table_schema,table_name,engine from information_schema.tables where engine='innodb';
#查看innodb的表有哪些
mysql> select table_schema,table_name,engine from information_schema.tables where engine='myisam';
#查看myisam的表有哪些
```

- innodb和myisam的区别

```shell
#进入mysql目录
[root@localhost~l]# cd /application/mysql/data/mysql
#查看所有user的文件
[root@localhost mysql]# ll user.*
-rw-rw---- 1 mysql mysql 10684 Mar  6  2017 user.frm
-rw-rw---- 1 mysql mysql   960 Aug 14 01:15 user.MYD
-rw-rw---- 1 mysql mysql  2048 Aug 14 01:15 user.MYI
#进入word目录
[root@localhost world]# cd /application/mysql/data/world/
#查看所有city的文件
[root@localhost world]# ll city.*
-rw-rw---- 1 mysql mysql   8710 Aug 14 16:23 city.frm
-rw-rw---- 1 mysql mysql 688128 Aug 14 16:23 city.ibd
```



#### innodb存储引擎的简介

- 在MySQL5.5版本之后，默认的存储引擎，提供高可靠性和高性能。

- 优点:事务安全（遵从 ACID）、MVCC（Multi-Versioning Concurrency Control，多版本并发控制）、InnoDB行级别锁定，另一种DB是表级别的、Oracle样式一致非锁定读取、表数据进行整理来优化基于主键的查询、支持外键引用完整性约束、大型数据卷上的最大性能、将对表的查询与不同存储引擎混合、出现故障后快速自动恢复、用于在内存中缓存数据和索引的缓冲区池

  ![innidb结构](E:\英格\图片\innidb结构.jpg)



- innodb核心特性：MVCC、事务、行级锁、热备份、Crash Safe Recovery（自动故障恢复）

- 查看存储引擎：

  使用 SELECT 确认会话存储引擎

```sql
SELECT @@default_storage_engine;# 查询默认存储引擎
```

使用 SHOW 确认每个表的存储引擎

```sql
SHOW CREATE TABLE City\G
SHOW TABLE STATUS LIKE 'CountryLanguage'\G# 查看表的存储引擎
```

- 存储引擎的设置：在启动配置文件中设置服务器存储引擎

```sql
[mysqld]
default-storage-engine=<Storage Engine># 在配置文件的[mysqld]标签下添加
```

- 使用 SET 命令为当前客户机会话设置

```sql
SET @@storage_engine=<Storage Engine># 在MySQL命令行中临时设置
```

- 在 CREATE TABLE 语句指定

```sql
CREATE TABLE t (i INT) ENGINE = <Storage Engine>;# 建表的时候指定存储引擎
```



#### 案例：存储引擎切换

- 项目背景：

  - 公司原有的架构：一个展示型的网站，LAMT，MySQL5.1.77版本（MYISAM），50M数据量。

- 小问题不断：

  - 表级锁：对表中任意一行数据修改类操作时，整个表都会锁定，对其他行的操作都不能同时进行。
  - 不支持故障自动恢复（CSR）：当断电时有可能会出现数据损坏或丢失的问题。

- 解决方案：

  - 提建议将现有的MYISAM引擎替换为Innodb，将版本替换为5.6.38
  - 如果使用MYISAM会产生”小问题”，性能安全不能得到保证，使用innodb可以解决这个问题。
  - 5.1.77版本对于innodb引擎支持不够完善，5.6.38版本对innodb支持非常完善了。

- 实施过程和注意要素

  备份生产库数据（mysqldump）

```sql
#[root@db01 ~]# mysqldump -uroot -p123 -A --triggers -R --master-data=2 >/tmp/full.sql
#由于没有开启bin logging所以去掉 --master-data=2
[root@db01 ~]# mysqldump -uroot -p123 -A --triggers -R >/tmp/full.sql
```

准备一个5.6.38版本的新数据库

对备份数据进行处理（将engine字段替换）

```shell
[root@db01 ~]# sed -i 's#ENGINE=MYISAM#ENGINE=INNODB#g' /tmp/full.sql
```

- 将修改后的备份恢复到新库
- 应用测试环境连接新库，测试所有功能
- 停应用，将备份之后的生产库发生的新变化，补偿到新库
- 应用割接到新数据库



#### 案例：数据库服务损坏

- 在没有备份数据的情况下，突然断电导致表损坏，打不开数据库。
- 拷贝库目录到新库中

```shell
[root@db01 ~]# cp -r /application/mysql/data/world/ /data/3307/data/
[root@db01 ~]# chown -R mysql.mysql /application/mysql/data/world
```

启动新数据库

```shell
#pkill mysqld 先干掉mysql
[root@db01 ~]# mysqld_safe --defaults-file=/data/3307/my.cnf &
```

登陆数据库查看

```sql
mysql> show databases;
```

查询表中数据

```sql
mysql> select * from city;
ERROR 1146 (42S02): Table 'world.city' doesn't exist
#先 use world；
```

找到**以前的表**结构在新库中创建表，此处演示用的show命令实际需要从原来的开发文档查看

```sql
mysql> show create table world.city;
#删掉外键创建语句
CREATE TABLE `city` (
  `ID` int(11) NOT NULL AUTO_INCREMENT,
  `Name` char(35) NOT NULL DEFAULT '',
  `CountryCode` char(3) NOT NULL DEFAULT '',
  `District` char(20) NOT NULL DEFAULT '',
  `Population` int(11) NOT NULL DEFAULT '0',
  PRIMARY KEY (`ID`),
  KEY `CountryCode` (`CountryCode`),
  KEY `idx_city` (`Population`,`CountryCode`),
    CONSTRAINT `city_ibfk_1` FOREIGN KEY (`CountryCode`) REFERENCES `country` (`Code`)
) ENGINE=InnoDB AUTO_INCREMENT=4080 DEFAULT CHARSET=latin1;


mysql> CREATE TABLE `city_new` (
  `ID` int(11) NOT NULL AUTO_INCREMENT,
  `Name` char(35) NOT NULL DEFAULT '',
  `CountryCode` char(3) NOT NULL DEFAULT '',
  `District` char(20) NOT NULL DEFAULT '',
  `Population` int(11) NOT NULL DEFAULT '0',
  PRIMARY KEY (`ID`),
  KEY `CountryCode` (`CountryCode`)
 # KEY `idx_city` (`Population`,`CountryCode`),
 #干掉外键的约束，否则失败
    #CONSTRAINT `city_ibfk_1` FOREIGN KEY (`CountryCode`) REFERENCES `country` (`Code`)
) ENGINE=InnoDB AUTO_INCREMENT=4080 DEFAULT CHARSET=latin1;
```

删除表空间文件

```sql
mysql> alter table city_new discard tablespace;
```

拷贝旧表空间文件

```sql
[root@db01 world]# cp /data/3307/data/world/city.ibd /data/3307/data/world/city_new.ibd
```

授权

```sql
[root@db01 world]# cd /data/3307/data/world
[root@db01 world]# chown -R mysql.mysql *
```

导入表空间

```sql
mysql> alter table city_new import tablespace;
mysql> alter table city_new rename city;
```



## 九、MySQL事务

### 事务

- 事务的定义：**事务**是一组操作的集合**，**它是一个不可分割的工作单位。**当我们进行事务操作时，事务会把所有的操作**作为一个整体一起向系统提交或撤销操作请求，即这些操作要么同时成功，要么同时失败。
- 事务ACID特性
  - Atomic（原子性）：所有语句作为一个单元全部成功执行或全部取消。
  - Consistent（一致性）：如果数据库在事务开始时处于一致状态，则在执行该事务期间将保留一致状态。
  - Isolated（隔离性）：事务之间不相互影响。
  - Durable（持久性）：事务成功完成后，所做的所有更改都会准确地记录在数据库中。所做的更改不会丢失。



事务的控制语句

```sql
START TRANSACTION（或 BEGIN）：显式开始一个新事务
SAVEPOINT：分配事务过程中的一个位置，以供将来引用
COMMIT：永久记录当前事务所做的更改
ROLLBACK：取消当前事务所做的更改
ROLLBACK TO SAVEPOINT：取消在 savepoint 之后执行的更改
RELEASE SAVEPOINT：删除 savepoint 标识符
SET AUTOCOMMIT：为当前连接禁用或启用默认 autocommit 模式
```

```sql
#一个成功事务的生命周期
begin;
sql1
sql2
sql3
...
commit;

#一个失败事务的生命周期
begin;
sql1
sql2
sql3
...
rollback;
```



- 自动提交

```sql
#默认自动提交，关闭后需要命令commit才生效，此时另一个客户端除非重新登录才能看到变化（隔离性）
#开启两个窗口，不同update，不同时间commit演示
mysql> show variables like 'autocommit';
#查看自动提交
mysql> set autocommit=0;
#临时关闭
[root@db01 world]# vim /etc/my.cnf
[mysqld]
autocommit=0
#永久关闭
```

- 事务演示

  - 成功事务

    需要打开另一个mysql终端查看，分别在执行完语句以及commit后查看

```sql
mysql> create table stu(id int,name varchar(10),sex enum('f','m'),money int);
mysql> begin;
mysql> insert into stu(id,name,sex,money) values(1,'zhang3','m',100), (2,'zhang4','m',110);
mysql> commit;
* 事务回滚
mysql> begin;
mysql> update stu set name='zhang3';
mysql> delete from stu;
mysql> rollback;
```

- 事务隐式提交情况
  - 现在版本在开启事务时，不需要手工begin，只要你输入的是DML语句，就会自动开启事务。
  - 有些情况下事务会被隐式提交
    - 在事务运行期间，手工执行begin的时候会自动提交上个事务
    - 在事务运行期间，加入DDL、DCL操作会自动提交上个事务
    - 在事务运行期间，执行锁定语句（lock tables、unlock tables）
    - load data infile
    - select for update
    - 在autocommit=1的时候



### 事务日志redo

- redo,顾名思义“重做日志”，是事务日志的一种。在事务ACID过程中，实现的是“D”持久化的作用。
- 特性:WAL(Write Ahead Log)日志优先写。REDO：记录的是，内存数据页的变化过程
- REDO工作过程

```sql
update t1 set num=2 where num=1;
```

- 执行步骤
  - 首先将t1表中num=1的行所在数据页加载到内存中buffer page
  - MySQL实例在内存中将num=1的数据页改成num=2
  - num=1变成num=2的变化过程会记录到，redo内存区域，也就是redo buffer page中 commit;
- 提交事务执行步骤
  - 当敲下commit命令的瞬间，MySQL会将redo buffer page写入磁盘区域redo log
  - 当写入成功之后，commit返回ok



### 事务日志undo

- undo,顾名思义“回滚日志”，是事务日志的一种。在事务ACID过程中，实现的是“A”原子性的作用。当然CI的特性也和undo有关
- redo和undo的存储位置

```shell
[root@db01 data]# ll /application/mysql/data/
-rw-rw---- 1 mysql mysql 50331648 Aug 15 06:34 ib_logfile0
-rw-rw---- 1 mysql mysql 50331648 Mar  6  2021 ib_logfile1
# redo位置
[root@db01 data]# ll /application/mysql/data/
-rw-rw---- 1 mysql mysql 79691776 Aug 15 06:34 ibdata1
-rw-rw---- 1 mysql mysql 79691776 Aug 15 06:34 ibdata2
# undo位置
```

- 在MySQL5.6版本中undo是在ibdata文件中，在MySQL5.7版本会独立出来。



### redo和undo的工作过程

![无标题-2024-06-24-1041](E:\英格\图片\无标题-2024-06-24-1041.png)

#### Redo Log（重做日志）工作流程

1. **事务开始**：
   - 当用户执行事务操作（如INSERT、UPDATE、DELETE等）时，事务开始。
   - 数据库会记录这些操作的变更信息到Redo Log中。这些变更信息包括操作的类型、涉及的数据页号、数据的旧值和新值等。
2. **写入 Redo Log**：
   - 数据库会将这些变更信息先写入到 Redo Log Buffer（重做日志缓冲区）中。Redo Log Buffer是内存中的一个区域，用于暂存重做日志信息。
   - 当Redo Log Buffer满了，或者达到一定的条件（如事务提交、日志刷新策略等），Redo Log Buffer中的内容会被刷新到磁盘上的Redo Log File中。
3. **事务提交**：
   - 当事务提交时，数据库会确保Redo Log已经写入磁盘（即Redo Log的持久化）。
   - 这时，即使系统崩溃，数据库也可以通过Redo Log来恢复事务操作，保证事务的持久性。
4. **数据页刷盘**：
   - 数据库后台线程（如Flush线程）会定期将内存中的数据页（Buffer Pool中的脏页）刷盘到磁盘上的数据文件中。
   - 这个过程是异步的，不一定在事务提交时立即完成。但是，只要有Redo Log，数据库就可以在恢复时重新应用这些变更。
5. **崩溃恢复**：
   - 如果数据库崩溃，重启时会读取Redo Log，根据日志中的记录重新应用变更，将数据恢复到崩溃前的状态。
   - 这个过程称为Redo操作，它确保了所有已经提交的事务都被正确地持久化到数据文件中。



#### Undo Log（回滚日志）工作流程

1. **事务开始**：
   - 当用户执行事务操作时，事务开始。
   - 数据库会记录这些操作的变更信息到Undo Log中。Undo Log用于记录数据的旧值，以便在事务回滚时可以恢复到事务开始前的状态。
2. **写入 Undo Log**：
   - 数据库会将变更操作的旧值记录到Undo Log中。这些记录包括操作的类型、涉及的数据页号、数据的旧值等。
   - Undo Log是存储在表空间中的，它是一个持久化的存储结构。
3. **事务回滚**：
   - 如果事务执行过程中发生错误，或者用户主动执行ROLLBACK命令，事务需要回滚。
   - 数据库会读取Undo Log，根据记录的旧值，将数据恢复到事务开始前的状态。
   - 这个过程称为Undo操作，它确保了事务的原子性。
4. **事务提交**：
   - 当事务正常提交时，Undo Log中与该事务相关的记录不会立即删除，因为可能还有其他事务需要依赖这些记录来完成一致性读操作。
   - 数据库会标记这些Undo Log记录为“已提交”，但不会立即清理它们。
5. **清理 Undo Log**：
   - 数据库后台线程（如Purge线程）会定期检查Undo Log，清理那些不再需要的记录。
   - 一个Undo Log记录只有在所有依赖它的事务都完成一致性读操作后，才会被清理。



### 事务中的锁

- “锁”顾名思义就是锁定的意思。在事务ACID特性过程中，“锁”和“隔离级别”一起来实现“I”隔离性的作用。
- 行级锁分为共享锁和排他锁
- 排他锁：保证在多事务操作时，数据的一致性。
- 共享锁：保证在多事务工作期间，数据查询时不会被阻塞。
- 多版本并发控制（MVCC）
  - 只阻塞修改类操作，不阻塞查询类操作
  - 乐观锁的机制（谁先提交谁为准）
- 锁的粒度:MyIsam：低并发锁（表级锁）; Innodb：高并发锁（行级锁）
- 表级锁：开销小，加锁快；不会出现死锁；锁定粒度大，发出锁冲突的概率最高，并发度最低。
- 行级锁：开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度最高。
- 事务的隔离级别有四种隔离级别
  - READ UNCOMMITTED（独立提交）：允许事务查看其他事务所进行的未提交更改，一个用户update命令后未commit，另一个用户看到的是修改的内存里的，即使没有写入硬盘
  - READ COMMITTED：允许事务查看其他事务所进行的已提交更改
  - REPEATABLE READ：确保每个事务的 SELECT 输出一致，一个用户update命令即使commit。另一个用户看到的未修改的，除非重新登录或者commit+（InnoDB 的默认级别）
  - SERIALIZABLE：将一个事务的结果与其他事务完全隔离，一个用户update命令后未commit，另一个用户即使select都看不到。



## 十、MySQL日志管理

### 日志简介

| 日志文件 | 选项                               | 文件名，表名              | 程序          |
| :------- | :--------------------------------- | :------------------------ | :------------ |
| 错误     | --log-error                        | host_name.err             |               |
| 常规     | --general_log                      | host_name.log general_log |               |
| 慢速查询 | --slow_query_log --long_query_time | host_name-slow.log        | mysqldumpslow |
| 二进制   | --log-bin --expire-logs-days       | host_name-bin.000001      | mysqlbinlog   |
| 审计     | --audit_log --audit_log_file ...   | audit.log                 |               |



### 错误日志

- 记录mysql数据库的一般状态信息及报错信息，是我们对于数据库常规报错处理的常用日志。
- 默认位置：$MYSQL_HOME/data/
- 开启方式：MySQL安装完后默认开启

```shell
[root@db01 ~]# vim /etc/my.cnf
# 编辑配置文件
[mysqld]
log_error=/application/mysql/data/$hostname.err
mysql> show variables like 'log_error';
# 查看方式
```



### 一般查询日志

- 记录mysql所有执行成功的SQL语句信息，可以做审计用，但是我们很少开启。
- 默认位置：$MYSQL_HOME/data/
- 开启方式：MySQL安装完之后默认不开启

```shell
[root@db01 ~]# vim /etc/my.cnf
[mysqld]
general_log=on
general_log_file=/application/mysql/data/$hostnamel.log
# 编辑配置文件
mysql> show variables like '%gen%';
# 查看方式
```



### 二进制日志

- 记录已提交的DML事务语句，并拆分为多个事件（event）来进行记录；所有DDL、DCL等语句。二进制日志会记录所有对数据库发生修改的操作
- 二进制共有三种日志模式
  - statement：语句模式
  - row：行模式，即数据行的变化过程（推荐企业使用）
  - mixed：以上两者的混合模式。
- 二进制日志模式优缺点
  - statement模式
    - 优点：简单明了，容易被看懂，就是sql语句，记录时不需要太多的磁盘空间
    - 缺点：记录不够严谨
  - row模式：记录了底层操作的所有事情
    - 优点：记录更加严谨
    - 缺点：有可能会需要更多的磁盘空间，不太容易被读懂
- binlog的作用
  - 可以把数据恢复到任意时刻。对数据的备份恢复，对数据的复制



#### 二进制日志的管理操作实例

- 开启方式

```shell
# MySQL版本为5.6.40
[root@db01 data]# vim /etc/my.cnf
[mysqld]
log-bin=mysql-bin
binlog_format=row

# 注意:在mysql5.7中开启binlog必须要加上server-id。
[mysqld]
log-bin=mysql-bin
binlog_format=row
server_id=1
```



- 二进制日志的操作

```shell
[root@db01 data]# ll /application/mysql/data/       #物理查看
-rw-rw---- 1 mysql mysql      285 Mar  6  2021 mysql-bin.000001

mysql> show binary logs;        #在数据库命令行查看
mysql> show master status;
mysql> show binlog events in 'mysql-bin.000007';
#查看binlog事件
```



事件：在binlog中最小的记录单元为event，一个事务会被拆分成多个事件（event）

事件特性：每个event都有一个开始位置（start position）和结束位置（stop position）。所谓的位置就是event对整个二进制的文件的相对位置。对于一个二进制日志中，前120个position是文件格式信息预留空间。一般MySQL第一个记录的事件，都是从120开始的。

row模式下二进制日志分析及数据恢复

```sql
mysql> show master status;# 查看binlog信息
mysql> create database binlog;# 创建一个binlog库
mysql> use binlog# 使用binlog库
mysql> create table binlog_table(id int);# 创建binglog_table表
mysql> show master status;# 查看binlog信息
mysql> insert into binlog_table values(1);# 插入数据1
mysql> show master status;# 查看binlog信息
mysql> commit;# 提交
mysql> show master status;# 查看binlog信息，可以看到提交以后结束位置才发生变化
mysql> insert into binlog_table values(2);# 插入数据2
mysql> insert into binlog_table values(3);#插入数据3
mysql> show master status;# 查看binlog信息
mysql> commit;# 提交
mysql> delete from binlog_table where id=1;# 删除数据1
mysql> show master status;# 查看binlog信息
mysql> commit;# 提交
mysql> update binlog_table set id=22 where id=2;# 更改数据2为22
mysql> show master status;# 查看binlog
mysql> commit;# 提交
mysql> show master status;# 查看binlog信息
mysql> select * from binlog_table;# 查看数据
mysql> drop table binlog_table;# 删表
mysql> drop database binlog;# 删库
```



恢复数据之前要准备：找错误发生前的那个位置的数值，以此准备一个即将发生错误前的正确的库

```shell
mysql> show binlog events in 'mysql-bin.000013';     # 查看binlog事件
# 也可以使用mysqlbinlog来查看
[root@db01 data]# mysqlbinlog /application/mysql/data/mysql-bin.xxxxxx
[root@db01 data]# mysqlbinlog /application/mysql/data/mysql-bin.xxxxxx|grep -v SET
[root@db01 data]# mysqlbinlog --base64-output=decode-rows -vvv mysql-bin.xxxxxx
# 假设查看二进制日志后，发现删除开始位置是858
# binlog某些内容是以二进制放进去的，加入base64-output用于解码
[root@db01 data]# mysqlbinlog --start-position=120 --stop-position=858 /application/mysql/data/mysql-bin.000013 > /tmp/binlog.sql
mysql> set sql_log_bin=0;#临时关闭binlog，必须关闭，否则source倒入的内容也会被记录进binlog，binlog就脏了。
mysql> source /tmp/binlog.sql#执行sql文件
mysql> show databases;#查看删除的库是否恢复
mysql> use binlog#进binlog库
mysql> show tables;#查看删除的表是否恢复
mysql> select * from binlog_table;#查看表中内容
```



只查看某个数据库的binlog文件

```sql
mysql> flush logs;#刷新一个新的binlog，原来的bin000X.log作废
mysql> create database db1;
mysql> create database db2;#创建db1、db2两个库

mysql> use db1#库db1操作
mysql> create table t1(id int);#创建t1表
mysql> insert into t1 values(1),(2),(3),(4),(5);#插入5条数据
mysql> commit;#提交
mysql> use db2#库db2操作
mysql> create table t2(id int);#创建t2表
mysql> insert into t2 values(1),(2),(3);#插入3条数据
mysql> commit;#提交
mysql> show binlog events in 'mysql-bin.000014';#查看binlog事件
[root@db01 data]# mysqlbinlog -d db1 --base64-output=decode-rows -vvv /application/mysql/data/mysql-bin.000014
#查看db1的操作
```

- 刷新binlog日志flush logs;

  重启数据库时会刷新

  二进制日志具有上限（max_binlog_size）
  
- 删除二进制日志

  原则：在存储能力范围内，能多保留则多保留。基于上一次全备前的可以选择删除

- 删除方式：根据存在时间删除日志

```shell
#临时生效
SET GLOBAL expire_logs_days = 7;
#永久生效
[root@db01 data]# vim /etc/my.cnf
[mysqld]
expire_logs_days = 7
# 使用purge命令删除
PURGE BINARY LOGS BEFORE now() - INTERVAL 3 day;
# 根据文件名删除
PURGE BINARY LOGS TO 'mysql-bin.000010';
# 用reset master
mysql> reset master;
```



### 慢查询日志

在MySQL中，慢查询定义为执行时间超过特定阈值的查询。这个阈值可以通过MySQL的配置选项`long_query_time`来设置。

慢查询日志是将mysql服务器中影响数据库性能的相关SQL语句记录到日志文件，通过对这些特殊的SQL语句分析，改进以达到提高数据库性能的目的

默认位置：MYSQL_HOME/data/data/hostname-slow.log

开启方式（默认没有开启）

```shell
[root@db01 ~]# vim /etc/my.cnf
[mysqld]
#指定是否开启慢查询日志
slow_query_log = 1
#指定慢日志文件存放位置（默认在data）
slow_query_log_file=/application/mysql/data/slow.log
#设定慢查询的阀值(默认10s)
long_query_time=0.05
#不使用索引的慢查询日志是否记录到索引
log_queries_not_using_indexes
#查询检查返回少于该参数指定行的SQL不被记录到慢查询日志（用的少）
min_examined_row_limit=100
```



#### 模拟慢查询语句

```sql
mysql> use world#进入world库
mysql> show tables;#查看表
mysql> create table t1 select * from city;#将city表中所有内容加到t1表中
mysql> desc t1;#查看t1的表结构
mysql> insert into t1 select * from t1;
mysql> insert into t1 select * from t1;
mysql> insert into t1 select * from t1;
mysql> insert into t1 select * from t1;#将t1表所有内容插入到t1表中（多插入几次）
mysql> commit;#提交
mysql> delete from t1 where id>2000;#删除t1表中id>2000的数据
[root@db01 ~]# cat /application/mysql/data/mysql-db01#查看慢日志
```



使用mysqldumpslow命令来分析慢查询日志

```shell
$PATH/mysqldumpslow -s c -t 10 /database/mysql/slow-log
#输出记录次数最多的10条SQL语句
```

参数说明:

-s:是表示按照何种方式排序，c、t、l、r分别是按照记录次数、时间、查询时间、返回的记录数来排序，ac、at、al、ar，表示相应的倒叙；

-t:是top n的意思，即为返回前面多少条的数据；

-g:后边可以写一个正则匹配模式，大小写不敏感的；

```shell
$PATH/mysqldumpslow -s r -t 10 /database/mysql/slow-log#得到返回记录集最多的10个查询
$PATH/mysqldumpslow -s t -t 10 -g "left join" /database/mysql/slow-log#得到按照时间排序的前10条里面含有左连接的查询语句
```



第三方慢查询推荐（扩展）：

```shell
yum install -y percona-toolkit-3.0.11-1.el6.x86_64.rpm
* 使用percona公司提供的pt-query-digest工具分析慢查询日志
[root@mysql-db01 ~]# pt-query-digest /application/mysql/data/mysql-db01-slow.log
pt-query-digest 的分析结果:

总体统计数据:
Queries examined: 总共分析的查询语句数量
Query time: 总查询时间
Min/Max/Avg query time: 最短、最长和平均查询时间
Variance: 查询时间的方差
Aggregated profile: 按查询类型统计的信息,如 SELECT、INSERT、UPDATE 等
Top Queries:
这部分列出了执行时间最长的前 10 个查询语句。
Rank: 查询在整体中的排名
Query ID: 查询的唯一标识
Response Time: 查询的总响应时间
Calls: 查询被执行的次数
R/Call: 每次调用的平均响应时间
V/M: 查询时间的方差与平均值的比率,反映了查询时间的离散程度
Item: 查询语句的文本
查询语句的详细信息:
Query ID: 查询的唯一标识
Database: 查询所针对的数据库
Users: 执行查询的用户
Hosts: 执行查询的主机
Query: 查询语句的文本
Exec time: 查询的总执行时间
Lock time: 查询获取的锁定时间
Rows sent: 查询返回的行数
Rows examined: 查询扫描的行数
Rows affected: 查询影响的行数
Bytes sent: 查询返回的字节数
Tmp tables: 查询使用的临时表数量
Tmp disk tables: 查询使用的临时磁盘表数量
Scan/Join tables: 查询扫描的表数量
```



## 十一、备份与恢复

备份的作用：保护公司的数据，让网站能7*24小时提供服务(用户体验)。备份就是为了在出现错误时恢复，尽量减少数据的丢失（公司的损失）

### 备份的类型

- 冷备份:这些备份在用户不能访问数据时进行，因此无法读取或修改数据。这些脱机备份会阻止执行任何使用数据的活动。这些类型的备份不会干扰正常运行的系统的性能。但是，对于某些应用程序，会无法接受必须在一段较长的时间里锁定或完全阻止用户访问数据。
- 温备份:这些备份在读取数据时进行，但在多数情况下，在进行备份时不能修改数据本身。这种中途备份类型的优点是不必完全锁定最终用户。但是，其不足之处在于无法在进行备份时修改数据集，这可能使这种类型的备份不适用于某些应用程序。在备份过程中无法修改数据可能产生性能问题。
- 热备份:这些动态备份在读取或修改数据的过程中进行，很少中断或者不中断传输或处理数据的功能。使用热备份时，系统仍可供读取和修改数据的操作访问。



### 备份的方式

- 逻辑备份

  SQL软件自带的功能

  - 基于SQL语句的备份：binlog、into outfile、mysqldump

```sql
[root@localhost ~]# vim /etc/my.cnf
[mysqld]
secure_file_priv=/tmp
mysql> select * from world.city into outfile '/tmp/world_city.data';
    * mysqldump
    * replication
```



- 物理备份：通过二进制方式直接拖走所有数据、配置文件
  - 基于数据文件的备份，如使用Xtrabackup（percona公司）



### 备份方法

- 备份策略
  - 全量备份/海量备份 full
  - 增量备份 increamental
- 备份工具
  - mysqldump（逻辑）：mysql原生自带很好用的逻辑备份工具
  - mysqlbinlog（逻辑）：实现binlog备份的原生态命令
  - xtrabackup（物理）：precona公司开发的性能很高的物理备份工具



#### mysqldump

**数据库连接参数**

- `-u` 或 `--user`：指定连接数据库的用户名。
- `-p` 或 `--password`：指定连接数据库的密码。如果省略密码，系统会提示输入。
- `-h` 或 `--host`：指定数据库服务器的主机名或 IP 地址。默认为本地主机（`localhost`）。
- `-P` 或 `--port`：指定数据库服务器的端口号。默认为 `3306`。
- `-S` 或 `--socket`：指定连接到本地 MySQL 服务器的 Unix 套接字文件路径。



**数据库和表选择参数**

- 数据库名：指定要备份的数据库名称。
- `-B` 或 `--databases`：备份多个数据库时使用。例如：`mysqldump -B db1 db2`。
- `--all-databases` 或 `-A`：备份所有数据库。
- `--tables`：指定要备份的表名。例如：`mysqldump db_name table1 table2`。



**备份内容和格式参数**

- `--no-data` 或 `-d`：只备份数据库结构，不备份数据。
- `--no-create-info` 或 `-t`：只备份数据，不备份表结构。
- `--complete-insert`：使用完整的 `INSERT` 语句（包含列名）。
- `--extended-insert`：使用多行 `INSERT` 语句，提高导入效率。
- `--skip-extended-insert`：禁用多行 `INSERT`，每行数据生成单独的 `INSERT` 语句。
- `--hex-blob`：将二进制数据以十六进制形式导出。
- `--set-gtid-purged`：控制是否记录 GTID 信息。选项为 `ON`、`OFF` 或 `AUTO`。
- -x：锁表备份（myisam温备份）
- --single-transaction：快照备份



**性能和优化参数**

- `--quick`：对大表进行快速导出，直接将数据写入输出文件，减少内存占用。
- `--single-transaction`：在导出时启动一个事务，确保数据一致性，适用于 InnoDB 存储引擎。
- `--flush-logs`：在导出开始时刷新 MySQL 日志。
- `--master-data`：包含二进制日志信息，用于主从复制的备份。选项为 `1` 或 `2`，`2` 会将日志信息写入注释中。
- `--lock-tables`：在导出时锁定所有表，确保数据一致性。
- `--skip-lock-tables`：跳过锁定表的操作，适用于大表或高并发环境。



**其他参数**

- `--routines` 或 `-R`：导出存储过程和函数。
- `--triggers`：导出触发器。
- `--events`：导出事件调度器中的事件。
- `--ignore-table`：忽略指定的表。例如：`--ignore-table=db_name.table_name`。
- `--result-file` 或 `-r`：将输出结果写入指定文件，而不是标准输出。
- `--help`：显示帮助信息。

```shell
[root@db01 ~]# mysqldump -uroot -p123456 -A > /backup/full.sql     
[root@db01 ~]# mysqldump -uroot -p123456 db1 > /backup/db1.sql    # 不加参数：单库、单表多表备份
[root@mysql-db01 backup]# mysqldump -uroot -p123456 world city > /backup/city.sql  
[root@db01 ~]# mysqldump -uroot -p123 -B db1 > /backup/db1.sql  # -B：指定库备份
[root@db01 ~]# mysqldump -uroot -p123 -B db1 db2 > /backup/db1_db2.sql 
[root@db01 backup]# mysqldump -uroot -p123 -A -R –triggers -F > /backup/full_2.sql  #-F：flush logs在备份时自动刷新binlog（不怎么常用） -d：仅表结构   -t：仅数据
[root@db01 backup]# mysqldump -uroot -p123 -A -R --triggers > /backup/full_2.sql
[root@db01 backup]# mysqldump -uroot -p123 -A -R --triggers --master-data=2 –-single-transaction>/backup/full.sql
# 加了文件末尾会多一行：CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000003', MASTER_LOG_POS=120;
# 不加master-data就不会记录binlog的位置，导致后续不利于增量备份
```



gzip:压缩备份

```shell
[root@db01 ~]# mysqldump -uroot -p123 -A -R --triggers --master-data=2 –single-transaction|gzip>/backup/full.sql.gz
[root@db01 ~]# gzip -d /backup/full.sql.gz
[root@db01 ~]# zcat /backup/full.sql.gz > linshi.sql
```

- 常用的热备份备份语句

```shell
mysqldump -uroot -p3307 -A -R --triggers --master-data=2 --single-transaction |gzip > /tmp/full_$(date +%F).sql.gz
```

- mysqldump的恢复

```sql
mysql> set sql_log_bin=0;
#先不记录二进制日志
mysql> source /backup/full.sql
#库内恢复操作
[root@db01 ~]# mysql -uroot -p123 < /backup/full.sql
#库外恢复操作
```

- 注意
  - mysqldump在备份和恢复时都需要MySQL实例启动为前提
  - 一般数据量级100G以内，大约15-30分钟可以恢复（PB、EB就需要考虑别的方式）
  - mysqldump是以覆盖的形式恢复数据的



### 实例：企业故障恢复

- 背景：正在运行的网站系统，MySQL数据库，数据量25G，日业务增量10-15M。

- 备份策略：每天23：00，计划任务调用mysqldump执行全备脚本

- 故障发生的事件：上午10点开发人员误删除一个核心业务表，需要恢复

- 思路

  - **停业务避免数据的二次伤害**！
  - 找一个临时的库，恢复前一天的全备
  - 截取前一天23：00到第二天10点误删除之间的binlog，恢复到临时库
  - 测试可用性和完整性
  - 开启业务前的两种方式
    - 直接使用临时库顶替原生产库，前端应用割接到新库
    - 将误删除的表单独导出，然后导入到原生产环境

- 开启业务



故障模拟

```sql
mysql> flush logs;  #刷新binlog使内容更清晰
mysql> show master status;  #查看当前使用的binlog
mysql> create database backup;  #创建backup库
mysql> use backup  #进入backup库
mysql> create table full select * from world.city;  #创建full表
mysql> create table full_1 select * from world.city;  #创建full_1表
mysql> show tables;  #查看表
```

全备

```sql
[root@db01 ~]# mysqldump -uroot -p123 -A -R --triggers --master-data=2 --single-transaction|gzip > /backup/full_$(date +%F).sql.gz
```

模拟数据变化

```sql
mysql> use backup  #进入backup库
mysql> create table new select * from mysql.user;  #创建new表
mysql> create table new_1 select * from world.country; # 创建new_1表
mysql> show tables;  #查看表
mysql> select * from full;  #查看full表中所有数据
mysql> update full set countrycode='CHN' where 1=1;  #把full表中所有的countrycode都改成CHN
mysql> commit;  #提交
mysql> delete from full where id>200;  #删除id大于200的数据
mysql> commit;  #提交
```

模拟故障

```sql
mysql> drop table new;#删除new表
mysql> show tables;#查看表
```



恢复过程

准备临时数据库

```shell
#开启临时的数据库
[root@db02 ~]# mysqld_safe --defaults-file=/data/3307/my.cnf &
#拷贝数据到新库上
[root@db02 ~]# scp /backup/full_2018-08-16.sql.gz root@10.0.0.52:/tmp

[root@db02 ~]# cd /tmp/ #进入tmp目录
[root@db02 tmp]# gzip -d full_2018-08-16.sql.gz #解压全备数据文件

截取二进制日志
[root@db02 tmp]# head -50 full_2018-08-16.sql |grep -i 'change master to'
#查看全备的位置点（起始位置点），忽略大小写，假设找到起始的位置为268002
mysql> show binlog events in 'mysql-bin.000017'\G
#生产环境db01找到drop语句执行的位置点（结束位置点）
[root@db01 tmp]#mysqlbinlog -uroot -p123 --start-position=268002 --stop-position=671148 /application/mysql/data/mysql-bin.000017 > /tmp/inc.sql
#截取二进制日志，把备份点和drop直接的信息倒入到inc.sql
[root@db01 tmp]# scp /tmp/inc.sql root@10.0.0.52:/tmp   #发送增量数据到新库
#在新库内恢复数据
mysql> set sql_log_bin=0;  #不记录二进制日志
mysql> source /tmp/full_2018-08-16.sql  #恢复全备数据
mysql> use backup  #进入backup库
mysql> show tables;  # 查看表，此时没有new表
mysql> source /tmp/inc.sql  #恢复增量数据
mysql> show tables;  #查看表，此时有new表
```



将故障表导出并恢复到生产

```sql
#将现在已经恢复好的数据库backup里的new表备份到/tmp/new.sql文件中
#此时/tmp/目录下有昨天23:00前完整备份文件full_2018-08-16.sql、昨天23:00到drop的增量文件inc.sql。
#通过这两个文件恢复了数据库出事前的所有内容
[root@db02 ~]# mysqldump -uroot -p123 -S /data/3307/mysql.sock backup new > /tmp/new.sql
#导出new表
[root@db02 ~]# scp /tmp/new.sql root@10.0.0.51:/tmp/
#发送到db01的库，此时db01的生产环境backup库里没有new表（被drop掉了）；可直接从new.sql倒入
mysql> use backup#进入backup库，此时生产环境的backup
mysql> source /tmp/new.sql#在生产库恢复数据
mysql> show tables;#查看表
```



### 物理备份（Xtrabackup）

- Xtrabackup安装

```shell
yum -y install epel-release  #安装epel源
yum -y install perl perl-devel libaio libaio-devel perl-Time-HiRes perl-DBD-MySQL  #安装依赖
wget httpss://www.percona.com/downloads/XtraBackup/Percona-XtraBackup-2.4.4/binary/redhat/6/x86_64/percona-xtrabackup-24-2.4.4-1.el6.x86_64.rpm  #下载Xtrabackup
yum localinstall -y percona-xtrabackup-24-2.4.4-1.el6.x86_64.rpm  # 安装
```

- 备份方式（物理备份）
  - 对于非innodb表（比如myisam）是直接锁表cp数据文件，属于一种温备。
  - 对于innodb的表（支持事务），不锁表，cp数据页最终以数据文件方式保存下来，并且把redo和undo一并备走，属于热备方式。
  - 备份时读取配置文件/etc/my.cnf，需要注明socket=/application/mysql/tmp/mysql.sock文件的位置
- 全量备份

```shell
[root@db01 data]# innobackupex --user=root --password=123 /backup  #全备
[root@db01 ~]# innobackupex --user=root --password=123 --no-timestamp /backup/full
#避免时间戳，自定义路径名
[root@db01 backup]# ll /backup/full  #查看备份路径中的内容
-rw-r-----  1 root root       21 Aug 16 06:23 xtrabackup_binlog_info  #记录binlog文件名和binlog的位置点
-rw-r-----  1 root root      117 Aug 16 06:23 xtrabackup_checkpoints
#备份时刻，立即将已经commit过的内存中的数据页刷新到磁盘
#备份时刻有可能会有其他数据写入，已备走的数据文件就不会再发生变化了
#在备份过程中，备份软件会一直监控着redo和undo，一旦有变化会将日志一并备走
-rw-r-----  1 root root      485 Aug 16 06:23 xtrabackup_info  #备份汇总信息
-rw-r-----  1 root root     2560 Aug 16 06:23 xtrabackup_logfile  #备份的redo文件
* 准备备份
* 将redo进行重做，已提交的写到数据文件，未提交的使用undo回滚，模拟CSR的过程
[root@db01 full]# innobackupex --user=root --password=123 --apply-log /backup/full
```

- 恢复备份
  - 前提1：被恢复的目录是空的
  - 前提2：被恢复的数据库的实例是关闭的

```shell
[root@db01 full]# /etc/init.d/mysqld stop  #停库
[root@db01 full]# cd /application/mysql   #进入mysql目录
[root@db01 mysql]# rm -fr data/   #删除data目录（在生产中可以备份一下）
[root@db01 mysql]# innobackupex --copy-back /backup/full   #拷贝数据
[root@db01 mysql]# chown -R mysql.mysql /application/mysql/data/  #授权
[root@db01 mysql]# /etc/init.d/mysqld start  #启动MySQL
```

- 增量备份及恢复
  - 基于上一次备份进行增量
  - 增量备份无法单独恢复，必须基于全备进行恢复
  - 所有增量必须要按顺序合并到全备当中

```shell
[root@mysql-db01 ~]#  innobackupex --user=root --password=123 --no-timestamp /backup/full
#不使用之前的全备，执行一次全备
```

- 模拟数据变化

```sql
mysql> create database inc1;
mysql> use inc1
mysql> create table inc1_tab(id int);
mysql> insert into inc1_tab values(1),(2),(3);
mysql> commit;
mysql> select * from inc1_tab;
```

- 第一次增量备份

```shell
[root@db01 ~]# innobackupex --user=root --password=123 --no-timestamp --incremental --incremental-basedir=/backup/full/ /backup/inc1
参数说明:
--incremental：开启增量备份功能
--incremental-basedir：上一次备份的路径
```

- 再次模拟数据变化

```sql
mysql> create database inc2;
mysql> use inc2
mysql> create table inc2_tab(id int);
mysql> insert into inc2_tab values(1),(2),(3);
mysql> commit;
```

- 第二次增量备份

```shell
[root@db01 ~]# innobackupex --user=root --password=123 --no-timestamp --incremental --incremental-basedir=/backup/inc1/ /backup/inc2
```

- 增量恢复

```shell
[root@db01 ~]# rm -fr /application/mysql/data/
#破坏数据
```

- 准备备份
  - full+inc1+inc2
  - 需要将inc1和inc2按顺序合并到full中
  - 分步骤进行--apply-log
- 第一步：在全备中apply-log时，只应用redo，不应用undo

```shell
[root@db01 ~]# innobackupex --apply-log --redo-only /backup/full/
```

- 第二步：合并inc1合并到full中，并且apply-log，只应用redo，不应用undo

```shell
[root@db01 ~]# innobackupex --apply-log --redo-only --incremental-dir=/backup/inc1/ /backup/full/
```

- 第三步：合并inc2合并到full中，redo和undo都应用

```shell
[root@db01 ~]# innobackupex --apply-log --incremental-dir=/backup/inc2/ /backup/full/
```

- 第四步：整体full执行apply-log，redo和undo都应用

```shell
[root@db01 mysql]# innobackupex --apply-log /backup/full/
copy-back
[root@db01 ~]# innobackupex --copy-back /backup/full/
[root@db01 ~]# chown -R mysql.mysql /application/mysql/data/
[root@db01 ~]# /etc/init.d/mysqld start
```



## 十二、MySQL的主从复制

### 简介及原理

主服务器将所有数据和结构更改记录到二进制日志中。从属服务器从主服务器请求该二进制日志并在本地应用其内容。IO作用：请求主库，获取上一次执行过的新的事件，并存放到relaylog。SQL：从relaylog中将sql语句翻译给从库执行

原理:

- 通过change master to语句告诉从库主库的ip，port，user，password，file，pos
- 从库通过start slave命令开启复制必要的IO线程和SQL线程
- 从库通过IO线程拿着change master to用户密码相关信息，连接主库，验证合法性
- 从库连接成功后，会根据binlog的pos问主库，有没有比这个更新的
- 主库接收到从库请求后，比较一下binlog信息，如果有就将最新数据通过dump线程给从库IO线程
- 从库通过IO线程接收到主库发来的binlog事件，存储到TCP/IP缓存中，并返回ACK更新master.info
- 将TCP/IP缓存中的内容存到relay-log中
- SQL线程读取relay-log.info，读取到上次已经执行过的relay-log位置点，继续执行后续的relay-log日志，执行完成后，更新relay-log.info

1. 当master节点接收到一个写请求时，此时会把写请求的更新操作都记录到binlog日志中。
2. master节点会把数据复制给slave节点，这个过程，需要每个slave节点连接到master节点上，master节点就会为每一 slave节点分别创建一个binlog dump线程，用于向各个slave节点发送binlog日志。
3. binlog dump线程会读取master节点上的binlog日志，然后将binlog日志发送给slave节点上的I/O线程。当主库读取事件的时候，会在binlog上加锁，读取完成之后，再将锁释放掉。
4. slave节点上的I/O线程接收到binlog日志后，会将binlog日志先写入到本地的relaylog中，relaylog中就保存了binlog日志。
5. slave节点上的SQL线程，会来读取relaylog中的binlog日志，将其解析成具体的增删改操作，把这些在master节点上进行过的操作，重新在slave节点上也重做一遍，达到数据还原的效果，这样就可以保证master节点和slave节点的数据一致性了。



缺点：1.同步过快导致错误也一起同步了，解决方法：延时从库

2.dump和I/O线程间有时延，当主机宕机，从库数据落后时，会造成数据不完整，解决方法：半同步复制

3.主库宕机，从库无法主动继续提供服务，解决方法：使用MHA高可用架构实现故障转移



作用：读写分离、数据备份、高可用



### 【实战】MySQL主从复制

本实例中db01是master，db02是slave

#### 主库操作

- 修改配置文件

```shell
[root@db01 ~]# vim /etc/my.cnf  #编辑mysql配置文件
[mysqld]  #在mysqld标签下配置
server_id =1  #主库server-id为1，从库不等于1
log_bin=mysql-bin  #开启binlog日志
```

- 导出一份全备数据给从库

```shell
[root@db01 ~]# mysqldump -uroot -p123456 -A -R --triggers --master-data=2 --single-transaction > /tmp/full.sql
[root@db01 ~]# scp /tmp/full.sql root@192.168.88.140:/tmp/
```

- 创建主从复制用户

```shell
[root@db01 ~]# mysql -uroot -p123456  #登录数据库
mysql> grant replication slave on *.* to slave@'192.168.88.%' identified by '123456';  #创建slave用户
```

#### 从库操作

1. 修改配置文件

```bash
[root@db02 ~]# vim /etc/my.cnf  #修改db02配置文件
[mysqld]
#在mysqld标签下配置
server_id =2/3/4  #只要不为1即可  #主库server-id为1，从库不等于1
log_bin=mysql-bin  #开启binlog日志
[root@db02 ~]# systemctl restart mysqld
```

1. 记录主库当前的binlog位置

```bash
[root@db01 ~]# mysql -uroot -p123456
mysql> show master status;
```

1. 再从库上配置主库等信息

```bash
[root@db02 ~]# mysql -uroot -p123456  #登陆数据库
mysql> change master to
-> master_host='192.168.88.136',
-> master_user='slave',
-> master_password='123456',
-> master_log_file='mysql-bin.000001',
-> master_log_pos=120;
#head -50 /tmp/full.sql内的master_log_file与master_log_pos
-># master_auto_position=1;    # 与前两句冲突，表示自动适配master的file和position，MariaDB5.5不支持GTID特性。此时未开启GTID
# master_auto_position=1 表示开启 MySQL 的基于 GTID 的主从复制功能。
# GTID(Global Transaction Identifier)是一种基于事务的复制方式,相比传统的基于文件和位置的复制方式,GTID 复制更加灵活和可靠。
# 当 master_auto_position=1 时,slave 会自动找到主库上最新的 GTID 位置,而不需要手动指定日志文件和位置。这样可以避免主从同步时由于位置不匹配而导致的问题。
```

#### 开启主从复制

```sql
#执行change master to 语句
mysql> start slave;
mysql> show slave status\G
#看到Slave_IO_Running: Yes    Slave_SQL_Running: Yes表示运行成功
#从数据库：/application/mysql/data目录下出现master.info文件，记录了同步的索引号
#测试方法：在主里面库建表插入内容，从里面可以看到主新增的内容表示同步成功。
```



### 主从复制基本故障处理

- IO线程报错
  - user password ip port
  - 网络：不同，延迟高，防火墙
  - 没有跳过反向解析[mysqld]里面加入skip-name-resolve
- 请求binlog
  - binlog不存在或者损坏
- 更新relay-log和master.info
  - SQL线程
  - relay-log出现问题
  - 从库做写入了
- 操作对象已存在（create）
- 操作对象不存在（insert update delete drop truncate alter）
- 约束问题、数据类型、列属性

**处理方法一**

```sql
mysql> stop slave;  #临时停止同步
mysql> set global sql_slave_skip_counter=1;  #将同步指针向下移动一个（可重复操作），告诉这个执行不了的binlog条目不执行，去执行下一条。
mysql> start slave;  #开启同步
```

**处理方法二**

```shell
[root@db01 ~]# vim /etc/my.cnf  #编辑配置文件
slave-skip-errors=1032,1062,1007  #在[mysqld]标签下添加以下参数
```

但是以上操作都是有风险存在的

**处理方法三**

```sql
set global read_only=1;  #在命令行临时设置，在从库设置
read_only=1  #在配置文件中永久生效[mysqld]
```



### 延时从库

主从能解决物理层面的损坏，但是如果在逻辑层面下出现问题了，无法有效的防止逻辑损坏。延时从库是为了来解决主从复制逻辑损坏的这一问题的。

步骤：停止主从；导出从库数据；主库导入

```sql
mysql>stop slave;  #停止主从
mysql>CHANGE MASTER TO MASTER_DELAY = 180;  #设置延时为180秒 企业中一般会延时3-6小时
mysql>start slave;  #开启主从
mysql> show slave status \G
SQL_Delay: 60  #查看状态

mysql> stop slave;  #停止主从
mysql> CHANGE MASTER TO MASTER_DELAY = 0;  #设置延时为0
mysql> start slave;  #开启主从
```



### 半同步复制

原理是在客户端提交事务之后不直接将结果返回给客户端，而是等待至少有一个从库收到了Binlog，并且写入到中继日志中，再返回给客户端。

半同步复制开启方法

安装（主库）

```shell
[root@db01 ~]# mysql -uroot -p123456 #登录数据库
mysql> show global variables like 'have_dynamic_loading'; #查看是否有动态支持 have_dynamic_loading=YES
mysql> INSTALL PLUGIN rpl_semi_sync_master SONAME'semisync_master.so';  #安装自带插件
mysql> SET GLOBAL rpl_semi_sync_master_enabled = 1;  #启动插件
mysql> SET GLOBAL rpl_semi_sync_master_timeout = 1000;  #设置超时
[root@db01 ~]# vim /etc/my.cnf#修改配置文件
[mysqld]
rpl_semi_sync_master_enabled=1
rpl_semi_sync_master_timeout=1000    #多长时间认为超时，单位ms
#在[mysqld]标签下添加如下内容（不用重启库）
mysql> show variables like'rpl%';
mysql> show global status like 'rpl_semi%'; #检查安装
```



**从库安装：**

```bash
[root@mysql-db02 ~]# mysql -uroot -p123456  #登录数据库
mysql>  INSTALL PLUGIN rpl_semi_sync_slave SONAME'semisync_slave.so';  #安装slave半同步插件
mysql> SET GLOBAL rpl_semi_sync_slave_enabled = 1;  #启动插件
mysql> stop slave io_thread;
mysql> start slave io_thread;  #重启io线程使其生效
[root@mysql-db02 ~]# vim /etc/my.cnf  #编辑配置文件（不需要重启数据库）
[mysqld]
rpl_semi_sync_slave_enabled =1  #在[mysqld]标签下添加如下内容
```

**注：相关参数说明：**

- rpl_semi_sync_master_timeout=milliseconds
  - 设置此参数值（ms）,为了防止半同步复制在没有收到确认的情况下发生堵塞，如果Master在超时之前没有收到任何确认，将恢复到正常的异步复制，并继续执行没有半同步的复制操作。
- rpl_semi_sync_master_wait_no_slave={ON|OFF}
  - 如果一个事务被提交,但Master没有任何Slave的连接，这时不可能将事务发送到其它地方保护起来。默认情况下，Master会在时间限制范围内继续等待Slave的连接，并确认该事务已经被正确的写到磁盘上。
  - 可以使用此参数选项关闭这种行为，在这种情况下，如果没有Slave连接，Master就会恢复到异步复制。



**测试半同步:**

```sql
mysql> create database test1;
Query OK, 1 row affected (0.04 sec)
mysql> create database test2;
Query OK, 1 row affected (0.00 sec)
#创建两个数据库，test1和test2
mysql> show global status like 'rpl_semi%';  #查看复制状态，Rpl_semi_sync_master_status状态是ON
+--------------------------------------------+-------+
| Variable_name                              | Value |
+--------------------------------------------+-------+
| Rpl_semi_sync_master_clients               | 1     |
| Rpl_semi_sync_master_net_avg_wait_time     | 768   |
| Rpl_semi_sync_master_net_wait_time         | 1497  |
| Rpl_semi_sync_master_net_waits             | 2     |
| Rpl_semi_sync_master_no_times              | 0     |
| Rpl_semi_sync_master_no_tx                 | 0     |
| Rpl_semi_sync_master_status                | ON    |
| Rpl_semi_sync_master_timefunc_failures     | 0     |
| Rpl_semi_sync_master_tx_avg_wait_time      | 884   |
| Rpl_semi_sync_master_tx_wait_time          | 1769  |
| Rpl_semi_sync_master_tx_waits              | 2     |
| Rpl_semi_sync_master_wait_pos_backtraverse | 0     |
| Rpl_semi_sync_master_wait_sessions         | 0     |
#此行显示2，表示刚才创建的两个库执行了半同步
| Rpl_semi_sync_master_yes_tx                | 2     | 
+--------------------------------------------+-------+
14 rows in set (0.06 sec)
mysql> show databases;
#从库查看，test1和test2在其中

# 主库
mysql> SET GLOBAL rpl_semi_sync_master_enabled = 0;  #关闭半同步（1:开启 0:关闭）
mysql> show global status like 'rpl_semi%';  #查看半同步状态
+--------------------------------------------+-------+
| Variable_name                              | Value |
+--------------------------------------------+-------+
| Rpl_semi_sync_master_clients               | 1     |
| Rpl_semi_sync_master_net_avg_wait_time     | 768   |
| Rpl_semi_sync_master_net_wait_time         | 1497  |
| Rpl_semi_sync_master_net_waits             | 2     |
| Rpl_semi_sync_master_no_times              | 0     |
| Rpl_semi_sync_master_no_tx                 | 0     |
| Rpl_semi_sync_master_status                | OFF   | #状态为关闭
| Rpl_semi_sync_master_timefunc_failures     | 0     |
| Rpl_semi_sync_master_tx_avg_wait_time      | 884   |
| Rpl_semi_sync_master_tx_wait_time          | 1769  |
| Rpl_semi_sync_master_tx_waits              | 2     |
| Rpl_semi_sync_master_wait_pos_backtraverse | 0     |
| Rpl_semi_sync_master_wait_sessions         | 0     |
| Rpl_semi_sync_master_yes_tx                | 2     | 
+--------------------------------------------+-------+
14 rows in set (0.00 sec)

mysql> create database test3;
Query OK, 1 row affected (0.00 sec)
mysql> create database test4;
Query OK, 1 row affected (0.00 sec)
#再一次创建两个库
mysql> show global status like 'rpl_semi%';#再一次查看半同步状态，此时Rpl_semi_sync_master_status变成OFF
+--------------------------------------------+-------+
| Variable_name                              | Value |
+--------------------------------------------+-------+
| Rpl_semi_sync_master_clients               | 1     |
| Rpl_semi_sync_master_net_avg_wait_time     | 768   |
| Rpl_semi_sync_master_net_wait_time         | 1497  |
| Rpl_semi_sync_master_net_waits             | 2     |
| Rpl_semi_sync_master_no_times              | 0     |
| Rpl_semi_sync_master_no_tx                 | 0     |
| Rpl_semi_sync_master_status                | OFF   |
| Rpl_semi_sync_master_timefunc_failures     | 0     |
| Rpl_semi_sync_master_tx_avg_wait_time      | 884   |
| Rpl_semi_sync_master_tx_wait_time          | 1769  |
| Rpl_semi_sync_master_tx_waits              | 2     |
| Rpl_semi_sync_master_wait_pos_backtraverse | 0     |
| Rpl_semi_sync_master_wait_sessions         | 0     |
#此行还是显示2，则证明，刚才的那两条并没有执行半同步否则应该是4
| Rpl_semi_sync_master_yes_tx                | 2     | 
+--------------------------------------------+-------+
14 rows in set (0.00 sec)
注:不难发现，在查询半同步状态是，开启半同步，查询会有延迟时间，关闭之后则没
```



##### 过滤复制

- 主库：
  - 白名单:只记录白名单中列出的库的二进制日志
    - binlog-do-db
  - 黑名单：不记录黑名单列出的库的二进制日志
    - binlog-ignore-db
- 从库：
  - 白名单：只执行白名单中列出的库或者表的中继日志
    - --replicate-do-db=test
    - --replicate-do-table=test.t1
    - --replicate-wild-do-table=test.t2
  - 黑名单：不执行黑名单中列出的库或者表的中继日志
    - --replicate-ignore-db
    - --replicate-ignore-table
    - --replicate-wild-ignore-table
- 复制过滤配置

```shell
[root@db01 data]# vim /data/3307/my.cnf 
replicate-do-db=world
#在[mysqld]标签下添加
mysqladmin -S /data/3307/mysql.sock  shutdown
#关闭MySQL
mysqld_safe --defaults-file=/data/3307/my.cnf &
#启动MySQL
#设置主从的server_id，如法炮制设置主从关系,记得change master to时候加上参数master_port = ‘3306’,
```

- 测试复制过滤：
- 第一次测试：
  - 主库：

```sql
[root@db02 ~]# mysql -uroot -p123 -S /data/3308/mysql.sock 
mysql> use world
mysql> create table t1(id int);
* 从库查看结果
[root@db02 ~]# mysql -uroot -p123 -S /data/3307/mysql.sock 
mysql> use world
mysql> show tables;
```

- 第二次测试
  - 主库

```shell
[root@db02 ~]# mysql -uroot -p123 -S /data/3308/mysql.sock 
mysql> use test
mysql> create table tb1(id int); 
* 从库查看结果
[root@db02 ~]# mysql -uroot -p123 -S /data/3307/mysql.sock 
mysql> use test
mysql> show tables;
```



### MHA高可用架构

MHA（Master High Availability）是一套优秀的MySQL高可用环境下故障切换和主从复制的软件。MHA 的出现就是解决MySQL单点的问题。MySQL故障切换过程中，MHA能做到0-30秒内自动完成故障切换操作。MHA能在故障切换的过程中最大程度上保证数据的一致性，以达到真正意义上的高可用。

MHA优点
1.自动故障转移快
2.主库崩溃不存在数据一致性问题
3.不需要对当前mysql环境做重大修改
4.不需要添加额外的服务器(仅一台manager就可管理上百个replication)
5.性能优秀，可工作在半同步复制和异步复制，当监控mysql状态时，仅需要每隔N秒向master发送ping包(默认3秒)，所以6.对性能无影响。你可以理解为MHA的性能和简单的主从复制框架性能一样。
7.只要replication支持的存储引擎，MHA都支持，不会局限于innodb



#### 工作流程

1. 把宕机的master二进制日志保存下来。
2. 找到binlog位置点最新的slave。
3. 在binlog位置点最新的slave上用relay log（差异日志）修复其它slave。（因为relay log修复比bin log快，所以不用master的bin log，slave没有bin log）
4. 将宕机的master上保存下来的二进制日志恢复到含有最新位置点的slave上。
5. 将含有最新位置点binlog所在的slave提升为master。
6. 将其它slave重新指向新提升的master，并开启主从复制。



#### MHA案例

##### 环境准备

至少三个mysql数据库



##### 基于GTID的主从复制

前置条件：主、从库都要开启binlog；主从库server_id不同；需要主从复制用户

主库操作：

修改配置文件

```shell
[root@mysql-db01 ~]# vim /etc/my.cnf  #编辑mysql配置文件
[mysqld]
#在mysqld标签下配置
server_id =1 #主库server-id为1，从库不等于1
log_bin=mysql-bin #开启binlog日志
#如果3台设备是在装了mysql后克隆的，mysql的UUID则相同，需要修改为不同值
[root@mysql-db01 ~]# vim /application/mysql/data/auto.cnf
server-uuid=8108d02e-be0a-11ec-8a15-000c2956d2e2
```

- 创建主从复制用户，**每台设备都要配置，因为slave有可能变成master**

```shell
[root@mysql-db01 ~]# mysql -uroot -p123456#登录数据库
mysql> grant replication slave on *.* to slave@'192.168.88.%' identified by '123456'; #创建slave用户
```



从库操作

修改配置文件

```shell
[root@mysql-db02 ~]# vim /etc/my.cnf  #修改mysql-db02配置文件
[mysqld]
#在mysqld标签下配置
server_id =5 #主库server-id为1，从库必须大于1
log_bin=mysql-bin #开启binlog日志
[root@mysql-db02 ~]# systemctl restart mysqld #重启mysql
[root@mysql-db03 ~]# vim /etc/my.cnf #修改mysql-db03配置文件
[mysqld]
#在mysqld标签下配置
server_id =10 #主库server-id为1，从库必须大于1
log_bin=mysql-bin #开启binlog日志
[root@mysql-db03 ~]# systemctl restart mysqld #重启mysql
```

注：在以往如果是基于binlog日志的主从复制，则必须要记住主库的master状态信息。



**主、从库都要开启GTID**

```shell
mysql> show global variables like '%gtid%';
#没开启之前先看一下GTID的状态
+--------------------------+-------+
| Variable_name            | Value |
+--------------------------+-------+
| enforce_gtid_consistency | OFF   |
| gtid_executed            |       |
| gtid_mode                | OFF   |
| gtid_owned               |       |
| gtid_purged              |       |
+--------------------------+-------+ 
[root@mysql-db01 ~]# vim /etc/my.cnf  #编辑mysql配置文件（主库从库都需要修改）
[mysqld]
#在[mysqld]标签下添加
gtid_mode=ON
log_slave_updates  #重要：开启slave的binlog同步
enforce_gtid_consistency  #开启GTID特性
[root@mysql-db01 ~]# systemctl restart mysqld  #重启数据库
mysql> show global variables like '%gtid%';  #检查GTID状态
+--------------------------+-------+
| Variable_name            | Value |
+--------------------------+-------+
| enforce_gtid_consistency | ON    | #执行GTID一致
| gtid_executed            |       |
| gtid_mode                | ON    | #开启GTID模块
| gtid_owned               |       |
| gtid_purged              |       |
+--------------------------+-------+
```

**注：主库从库都需要开启GTID否则在做主从复制的时候就会报错，因为slave可能变成master**

```sql
[root@mysql-db02 ~]# mysql -uroot -p123456
mysql> change master to
master_host='192.168.88.10',
master_user='slave',
master_password='123456',
master_auto_position=1;
#如果GTID没有开的话会出现：
ERROR 1777 (HY000): CHANGE MASTER TO MASTER_AUTO_POSITION = 1 can only be executed when @@GLOBAL.GTID_MODE = ON.
```

配置主从复制

```shell
[root@mysql-db02 ~]# mysql -uroot -p123456  #登录数据库
mysql> change master to  #配置复制主机信息
-> master_host='10.0.0.51',  #主库IP
-> master_user='rep',  #主库复制用户
-> master_password='123456',  #主库复制用户的密码
-> master_auto_position=1;  #GTID位置点
mysql> start slave;  #开启slave
mysql> show slave status\G  #查看slave状态
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 10.0.0.51
                  Master_User: rep
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000003
          Read_Master_Log_Pos: 403
               Relay_Log_File: mysql-db02-relay-bin.000002
                Relay_Log_Pos: 613
        Relay_Master_Log_File: mysql-bin.000003
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 403
              Relay_Log_Space: 822
              Until_Condition: None
```



从库设置

```shell
[root@mysql-db02 ~]# mysql -uroot -p123456  #登录从库
mysql> set global relay_log_purge = 0;  #禁用自动删除relay log 功能
mysql> set global read_only=1;  #设置只读
[root@mysql-db02 ~]# vim /etc/my.cnf  #编辑配置文件
[mysqld]  #在mysqld标签下添加
relay_log_purge = 0  #禁用自动删除relay log 永久生效
```



##### 部署MHA

环境准备**（所有节点）**

下载MHA工具包：

```bash
[root@localhost ~]# wget "https://download.s21i.faiusr.com/23126342/0/0/ABUIABBPGAAg3OHUiAYolpPt7AQ.zip?f=mysql-master-ha.zip&v=1628778716" -O mysql-master-ha.zip
[root@localhost ~]# mkdir /application/tools
[root@mysql-db01 ~]# unzip mysql-master-ha.zip -d /application/tools/
[root@mysql-db01 ~]# cd /application/tools/mysql-master-ha/
#进入安装包存放目录
[root@mysql-db01 tools]# ll
mha4mysql-manager-0.56-0.el6.noarch.rpm
mha4mysql-manager-0.56.tar.gz
mha4mysql-node-0.56-0.el6.noarch.rpm
mha4mysql-node-0.56.tar.gz

[root@mysql-db01 ~]# yum install perl-DBD-MySQL -y #安装依赖包
[root@mysql-db01 tools]# yum install -y  mha4mysql-node-0.56-0.el6.noarch.rpm
Preparing...                ########################################### [100%]
   1:mha4mysql-node         ########################################### [100%]
#安装node包，所有节点都要安装node包
[root@mysql-db01 tools]# mysql -uroot -p123456  #登录数据库，主库，从库会自动同步
mysql> grant all privileges on *.* to mha@'192.168.88.%' identified by 'mha';  #添加mha管理账号
mysql> select user,host from mysql.user;  #查看是否添加成功
mysql> select user,host from mysql.user;  #主库上创建，从库会自动复制（在从库上查看）
```

- 命令软连接（所有节点）

```shell
[root@mysql-db01 ~]# ln -s /application/mysql/bin/mysqlbinlog /usr/bin/mysqlbinlog
[root@mysql-db01 ~]# ln -s /application/mysql/bin/mysql /usr/bin/mysql
#如果不创建命令软连接，检测mha复制情况的时候会报错，写入环境变量
```

- 部署管理节点（mha-manager:mysql-db03）最好用第四个节点安装mha-manager

```shell
[root@mysql-db03 ~]# yum -y install epel-release  #使用epel源
[root@mysql-db03 ~]# yum install -y perl-Config-Tiny epel-release perl-Log-Dispatch perl-Parallel-ForkManager perl-Time-HiRes  #安装manager依赖包
[root@mysql-db03 tools]# yum install -y  mha4mysql-manager-0.56-0.el6.noarch.rpm 
Preparing...              ########################################### [100%]
1:mha4mysql-manager       ########################################### [100%]
#安装manager包
```

- 编辑配置文件

```shell
[root@mysql-db03 ~]# mkdir -p /etc/mha  #创建配置文件目录
[root@mysql-db03 ~]# mkdir -p /var/log/mha/app1  #创建日志目录
[root@mysql-db03 ~]# vim /etc/mha/app1.cnf  #编辑mha配置文件
[server default]
manager_log=/var/log/mha/app1/manager
manager_workdir=/var/log/mha/app1
master_binlog_dir=/application/mysql/data
user=mha
password=mha
ping_interval=2
repl_password=123456
repl_user=rep
ssh_user=root
[server1]
hostname=10.0.0.51
port=3306
[server2]
candidate_master=1
check_repl_delay=0
hostname=10.0.0.52
port=3306
[server3]
hostname=10.0.0.53
port=3306
```

配置文件详解

```shell
[server default]
manager_workdir=/var/log/masterha/app1  #设置manager的工作目录
manager_log=/var/log/masterha/app1/manager.log   #设置manager的日志
master_binlog_dir=/data/mysql  #设置master 保存binlog的位置，以便MHA可以找到master的日志，我这里的也就是mysql的数据目录
master_ip_failover_script= /usr/local/bin/master_ip_failover  #设置自动failover时候的切换脚本
master_ip_online_change_script= /usr/local/bin/master_ip_online_change  #设置手动切换时候的切换脚本
password=123456  #设置mysql中root用户的密码，这个密码是前文中创建监控用户的那个密码
user=root  #设置监控用户root
ping_interval=1  #设置监控主库，发送ping包的时间间隔，尝试三次没有回应的时候自动进行failover
remote_workdir=/tmp  #设置远端mysql在发生切换时binlog的保存位置
repl_password=123456  #设置复制用户的密码
repl_user=rep  #设置复制环境中的复制用户名 
report_script=/usr/local/send_report
#设置发生切换后发送的报警的脚本
#一旦MHA到server02的监控之间出现问题，MHA Manager将会尝试从server03登录到server02
secondary_check_script= /usr/local/bin/masterha_secondary_check -s server03 -s server02 --user=root --master_host=server02 --master_ip=192.168.0.50 --master_port=3306
shutdown_script=""
#设置故障发生后关闭故障主机脚本（该脚本的主要作用是关闭主机防止发生脑裂,这里没有使用）
ssh_user=root   #设置ssh的登录用户名
[server1]
hostname=10.0.0.51
port=3306
[server2]
hostname=10.0.0.52
port=3306
candidate_master=1
# 设置为候选master，如果设置该参数以后，发生主从切换以后将会将此从库提升为主库，即使这个主库不是集群中事件最新的slave。
check_repl_delay=0
# 默认情况下如果一个slave落后master 100M的relay logs的话，MHA将不会选择该slave作为一个新的master，因为对于这个slave的恢复需要花费很长时间，通过设置check_repl_delay=0,MHA触发切换在选择一个新的master的时候将会忽略复制延时，这个参数对于设置了candidate_master=1的主机非常有用，因为这个候选主在切换的过程中一定是新的master
```

- 配置ssh信任（所有节点）

```shell
[root@mysql-db01 ~]# ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa >/dev/null 2>&1  #创建秘钥对
[root@mysql-db01 ~]# ssh-copy-id -i /root/.ssh/id_dsa.pub root@192.168.88.10
[root@mysql-db01 ~]# ssh-copy-id -i /root/.ssh/id_dsa.pub root@192.168.88.20
[root@mysql-db01 ~]# ssh-copy-id -i /root/.ssh/id_dsa.pub root@192.168.88.30  #发送公钥，包括自己
```

- 启动测试

```shell
[root@mysql-db03 ~]# masterha_check_ssh --conf=/etc/mha/app1.cnf  #测试ssh
#看到如下字样，则测试成功
Tue Mar  7 01:03:33 2017 - [info] All SSH connection tests passed successfully.
[root@mysql-db03 ~]# masterha_check_repl --conf=/etc/mha/app1.cnf  #测试复制
#看到如下字样，则测试成功
#若不在slave库上创建用户会失败，按理说应该slave会同步master的库但创建slave用户是创建主从关系前
#mysql> grant replication slave on *.* to rep@'10.0.0.%' identified by '123456';
MySQL Replication Health is OK.
```

- 启动MHA

```shell
[root@mysql-db03 ~]# nohup masterha_manager --conf=/etc/mha/app1.cnf --remove_dead_master_conf --ignore_last_failover < /dev/null > /var/log/mha/app1/manager.log 2>&1 &  #启动
[root@mysql-db03 ~]# masterha_manager --conf=/etc/mha/app1.cnf --remove_dead_master_conf --ignore_last_failover --shutdown  #关闭
[root@mysql-db03 ~]# masterha_check_status --conf=/etc/mha/app1.cnf  #查看mha是否运行正常，正常会显示master的IP
```

- 切换master测试

```shell
[root@mysql-db02 ~]# mysql -uroot -p123456  #登录数据库（db02）
mysql> show slave status\G  #检查复制情况
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 10.0.0.51
                  Master_User: rep
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000006
          Read_Master_Log_Pos: 191
               Relay_Log_File: mysql-db02-relay-bin.000002
                Relay_Log_Pos: 361
        Relay_Master_Log_File: mysql-bin.000006
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
[root@mysql-db03 ~]# mysql -uroot -p123456  #登录数据库（db03）
mysql> show slave status\G  #检查复制情况
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 10.0.0.51
                  Master_User: rep
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000006
          Read_Master_Log_Pos: 191
               Relay_Log_File: mysql-db03-relay-bin.000002
                Relay_Log_Pos: 361
        Relay_Master_Log_File: mysql-bin.000006
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes

[root@mysql-db01 ~]# /etc/init.d/mysqld stop  #停掉主库
Shutting down MySQL..... SUCCESS!
[root@mysql-db02 ~]# mysql -uroot -p123456  #登录数据库（db02）
mysql> show slave status\G  #查看slave状态
Empty set (0.00 sec)  #db02的slave已经为空
[root@mysql-db03 ~]# mysql -uroot -p123456  #登录数据库（db03）
mysql> show slave status\G  #查看slave状态
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 10.0.0.52
                  Master_User: rep
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000006
          Read_Master_Log_Pos: 191
               Relay_Log_File: mysql-db03-relay-bin.000002
                Relay_Log_Pos: 361
        Relay_Master_Log_File: mysql-bin.000006
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
```

此时停掉主库后再开启db01，db01正常了只能通过**手动方式以从库身份**加入

```sql
[root@mysql-db01 ~]# mysql -uroot -123456
change master to
master_host='192.168.88.136',
master_user='slave',
master_password='123456',
master_auto_position=1;
-> start slave;
```



##### 配置vIP漂移

VIP漂移的两种方式

- 通过keepalived的方式，管理虚拟IP的漂移，与Nginx负载均衡有关
- 通过MHA自带脚本方式，管理虚拟IP的漂移。修改配置文件，failover文件

keepalive方式：

```shell
[root@mysql-db03 ~]# vim /etc/mha/app1.cnf  #编辑配置文件
[server default]  #在[server default]标签下添加
master_ip_failover_script=/etc/mha/master_ip_failover

#使用MHA自带脚本，在下载的mha文件mysql-master-ha.zip解压后的文件里
#tar xzvf mha4mysql-manager-0.56.tar.gz
#cd mha4mysql-manager-0.56/sample/script
#master-ip-failover文件就是自带的

#编辑脚本，该文件从wget而来
[root@mysql-db03 ~]# vim /etc/mha/master_ip_failover
#根据配置文件中脚本路径编辑
my $vip = '10.0.0.55/24';
my $key = '0';
#根据系统修改网卡名，网卡名要改对
my $ssh_start_vip = "/sbin/ifconfig ens33:$key $vip";
my $ssh_stop_vip = "/sbin/ifconfig ens33:$key down"; 
#修改以下几行内容
[root@mysql-db03 ~]# chmod +x /etc/mha/master_ip_failover  #添加执行权限，否则mha无法启动
[root@mysql-db03 ~]# yum install net-tools  #安装IFconfig，每台设备都要安装否则脚本执行失败

* 手动绑定vIP，假设db01是master
[root@mysql-db01 ~]# ifconfig ens33:0 192.168.88.88/24
#绑定vip，第一次要在master上手工配置，后面不需要了
[root@mysql-db01 ~]# ip a |grep eth0  #查看vip
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
   inet 10.0.0.51/24 brd 10.0.0.255 scope global eth0
   inet 10.0.0.55/24 brd 10.0.0.255 scope global secondary eth0:0

*安装dos2unix，因为从wget获取的master_ip_failover在windows下编辑，换行符与linux不一样，需要转换
[root@mysql-db03 mha]# yum install dos2unix
[root@mysql-db03 mha]# dos2unix master_ip_failover

*重启mha
[root@mysql-db03 ~]#masterha_manager --conf=/etc/mha/app1.cnf --remove_dead_master_conf --ignore_last_failover --shutdown   #关闭
[root@mysql-db03 ~]# nohup masterha_manager --conf=/etc/mha/app1.cnf --remove_dead_master_conf --ignore_last_failover < /dev/null > /var/log/mha/app1/manager.log 2>&1 &   #启动
[root@mysql-db03 ~]# masterha_check_status --conf=/etc/mha/app1.cnf   #查看mha是否运行正常，正常会显示master的IP

* 测试ip漂移
#登录db02
[root@mysql-db02 ~]# mysql -uroot -p123456
#查看slave信息
mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 10.0.0.51
                  Master_User: rep
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000007
          Read_Master_Log_Pos: 191
               Relay_Log_File: mysql-db02-relay-bin.000002
                Relay_Log_Pos: 361
        Relay_Master_Log_File: mysql-bin.000007
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
#停掉主库
[root@mysql-db01 ~]# /etc/init.d/mysqld stop
Shutting down MySQL..... SUCCESS!
#在db03上查看从库slave信息
mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 10.0.0.52
                  Master_User: rep
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000006
          Read_Master_Log_Pos: 191
               Relay_Log_File: mysql-db03-relay-bin.000002
                Relay_Log_Pos: 361
        Relay_Master_Log_File: mysql-bin.000006
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes

#在db01上查看vip信息
[root@mysql-db01 ~]# ip a |grep eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
inet 10.0.0.51/24 brd 10.0.0.255 scope global eth0
#在db02上查看vip信息
[root@mysql-db02 ~]# ip a |grep eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    inet 10.0.0.52/24 brd 10.0.0.255 scope global eth0
    inet 10.0.0.55/24 brd 10.0.0.255 scope global secondary eth0:0
```



### MGR高可用架构

​		MySQL Group Replication（简称MGR）是MySQL 5.7.17版本引入的一个服务器插件，可用于创建高可用、可扩展、容错的复制拓扑结构。它基于原生的主从复制，将各节点归入到一个组中，通过组内节点的通信协商(组通信协议基于Paxos算法)，实现数据的强一致性、故障探测、冲突检测、节点加组、节点离组等等功能。


​		以3个节点的组为例，这3个节点互相通信，每当有事件发生，都会向其他节点传播该事件，然后协商，如果大多数节点都同意这次的事件，那么该事件将通过，否则该事件将失败或回滚。这些节点可以是单主模式(single-primary)，也可以是多主模式(multi-primary)。单主模式只有一个主节点可以接受写操作，主节点故障时可以自动选举主节点。多主模型下，所有节点都可以接受写操作，所以没有master-slave的概念。


#### MGR体系结构

API接口：用于控制插件与MySQL服务器的交互方式。这些接口将MySQL服务器核心与MGR插件隔离。服务器向插件通知启动、恢复、准备接收连接、即将提交事务等消息。插件指示服务器执行诸如提交事务、中止正在进行的事务、事务在中继日志中排队等动作。

组件：

- Capture/Apply/Recovery组件：
  - 捕获组件Capture负责跟踪与正在执行的事务相关的上下文；
  - 应用组件Apply负责在数据库上执行远程事务；
  - 恢复组件Recovery管理分布式恢复，负责选择捐赠者，对故障做出反应，执行追赶程序，使加入该组的服务器获得更新。

复制协议逻辑：它处理冲突检测，接收事务并将其传播到组。

组复制插件体系结构底层：最后两层是组通信系统（GCS）API，以及基于Paxos的组通信引擎（XCom）的实现。GCS API将消息传递层的实现与插件上层分离，组通信引擎处理与复制组成员的通信。



#### MGR原理

​		复制组是由能够相互通信的多个服务器节点组成，在通信层则提供了原子消息和完全信息交互等保障机制，实现基于复制协议的多主更新。复制组由多个服务器组成，每个server成员可以独立的执行事务，但所有的读写RW事务只有在冲突检测成功后才会提交，只读RO事务则不需要冲突检测。因此，当一个读写事务准备提交的时候，会自动在组内进行原子性的广播，告知其它节点变更了什么内容、执行了什么事务。这种原子广播的方式，使得这个事务在每一个节点上都保持着同样顺序。这意味着每一个节点都以同样的顺序，接收到了同样的事务日志，所以每一个节点以同样的顺序重演了这些事务日志，最终整个组内保持了完全一致的状态。



#### 复制模式

MGR有两种复制模式：单主模式和多主模式。在单主模式下，组复制具有自动选主功能，每次只有一个节点负责写入，读可以从任意一个节点读取，组内数据保持最终一致。多主模式下，所有的节点都可以同时接受读写，也能够保证组内数据最终一致性。

##### 单主模式

复制组内只有一台节点可写可读，其他节点只可以读。单写模式MGR的部署流程如下：

首先运行主节点（即那个可写可读的节点，read_only = 0）
运行其他的节点，并把这些节点一一加进group。其他的节点就会自动同步主节点上面的变化，然后将自己设置为只读模式（read_only = 1）。
当主节点意外宕机或者下线，在满足大多数节点存活的情况下，group内部发起选举，选出下一个可用的读节点，提升为主节点。
主选举根据group内剩下存活节点的UUID按字典序升序来选择，即剩余存活的节点按UUID字典序排列，然后选择排在最前的节点作为新的主节点。

##### 多主模式

组内的所有机器都是主节点，同时可以进行读写操作，并且数据是最终一致的。

首先关闭单主模式开关loose-group_replication_single_primary_mode=OFF
运行第一个节点，设置loose-group_replication_bootstrap_group=ON
运行其它节点，并START GROUP_REPLICATION加入到group组中
当组内的某个节点发生故障时，会自动从将该节点从组内踢出，与其他节点隔离。剩余的节点之间保持主从复制的正常同步关系。当该节点的故障恢复后，只需手动激活组复制即可(即执行"START GROUP_REPLICATION;");

#### MGR特点

**优点**

高一致性：基于原生复制及paxos协议的组复制技术，对于只读事务，组间实例无需进行通讯，就可以处理事务；对于读写事务，组内所有节点必须经过通讯，共同决定事务提交与否

高容错性：基于分布式一致性算法实现，一个组允许部分节点挂掉，只要保证大多数节点仍然存活，那么这个组对外仍然能够提供服务

高扩展性：支持良好的扩展能力，可动态增删节点，组成员自动管理，结点的新增和移除都是自动的，新节点加入后，会自动从其它节点上同步状态，直到新节点和其它节点保持一致；如果某节点被移除了，其他节点自动更新组信息，自动维护新的组信息

事务冲突处理：在高并发的多写模式下，结点间的事务提交可能会产生冲突，如，两个不同事物在两个节点上进行了同一操作，这时候就会产生冲突，首先，复制组能够识别到这个冲突，并依赖事务提交的时间先后顺序，先发起提交的节点能够正确提交，后面提交的会失败

故障检测：可以识别组内成员是否挂掉，当一个节点失效，将由其他节点决定是否将这个时效的节点从group里面剔除

模式灵活性：单主模式下，mgr会自动选主，所有更新操作都在主上进行，多主模式下，所有节点都可以同时处理更新操作



**缺点**

存储引擎必须为Innodb，即仅支持InnoDB表
每个表必须提供主键，用于做write set的冲突检测
只支持ipv4，网络需求较高;
必须打开GTID特性，二进制日志格式必须设置为ROW，用于选主与write set;
COMMIT可能会导致失败，类似于快照事务隔离级别的失败场景;
目前一个MGR集群组最多支持9个节点;
不支持外键于save point特性，无法做全局间的约束检测与部分部分回滚;
二进制日志binlog不支持Replication event checksums;
多主模式(也就是多写模式) 不支持SERIALIZABLE事务隔离级别;
多主模式不能完全支持级联外键约束;
多主模式不支持在不同节点上对同一个数据库对象并发执行DDL(在不同节点上对同一行并发进行RW事务，后发起的事务会失败);

