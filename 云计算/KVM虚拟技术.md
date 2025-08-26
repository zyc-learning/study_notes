# 虚拟化技术

## 一、什么是虚拟化

虚拟化的本质是一种资源管理技术，将计算机的各种物理资源（如服务器、网络、内存存储等）抽象化后呈现出来，这些资源不受现有资源的架构方式、地域或物理设备所限制；然后将这些资源组合为一个或多个计算机的配置环境，打破了物理设备结构间不可切割的障碍

即：没有虚拟化前软硬件绑定，虚拟化后软硬件解耦（将软件和硬件模块分离，降低之间的依懒性）

Linux内核态 用户态

```
	在Linux系统中，用户态和内核态是两种不同的运行模式，它们主要区别在于程序所处的权限和访问硬件资源的方式。

	用户态是指程序运行在较低的权限级别，不能直接访问系统底层的资源，如硬件设备和内核态的代码。程序在用户态下执行时，需要向操作系统发出系统调用请求，由操作系统代表程序去执行相关的操作。用户态下的程序主要执行一些应用程序逻辑，例如文本编辑器、浏览器、音乐播放器等。

	内核态是指操作系统运行在较高的权限级别，能够直接访问硬件设备和系统资源，并且可以执行操作系统内核的代码。操作系统内核是系统中最底层的软件，它管理着整个系统的资源和进程，并且能够对外提供系统调用接口。内核态下的程序主要执行一些底层的操作，例如驱动程序、系统服务等。

	两者之间的联系在于，用户态和内核态是通过系统调用来进行通信的。当用户态的程序需要访问系统底层资源时，需要向操作系统发出系统调用请求。操作系统在接收到请求后，会将程序的权限提升到内核态，执行相应的操作，并将结果返回给用户态程序。用户态程序再将结果处理后，继续在用户态下执行。

	总体来说，用户态和内核态是系统中的两种不同的运行模式，它们各自拥有不同的权限和访问资源的方式。通过系统调用机制，用户态和内核态能够进行通信和交互，实现对系统资源的访问和管理。
```



## 二、虚拟化方案

### 软件虚拟化和硬件虚拟化

​		**软件虚拟化**，就是通过软件模拟来实现VMM层，通过纯软件的环境来模拟执行客户机里的指令。

​		**最纯粹的软件虚拟化**实现当属**QEMU**。在没有启用硬件虚拟化辅助的时候，它通过软件的二进制翻译仿真出目标平台呈现给客户机，客户机里的每一条目标平台指令都会被QEMU截取，并翻译成宿主机平台的指令，然后交给实际的物理平台执行。由于每一条都需要这么操作一下，其虚拟化性能是比较差的，同时期团建复杂的也大大增加。但好处是可以呈现给各种平台给客户机。

​		**硬件虚拟化**技术就是指计算机硬件本身提供能力让客户机指令独立执行，而不需要VMM截获重定向。Intel从2005年开始在其x86cpu中加入硬件虚拟化的支持，简称Intel VT技术



### 半虚拟化和全虚拟化

​		**部分虚拟化（Partial Virtualization）**VMM 只模拟部分底层硬件，因此客户机操作系统不做修改是无法在虚拟机中运行的，其它程序可能也需要进行修改。在历史上，部分虚拟化是通往全虚拟化道路上的重要里程碑,最早出现在第一代的分时系统 CTSS 和 IBM M44/44X 实验性的分页系统中。

​		**全虚拟化（Full Virtualization）**全虚拟化是指虚拟机**模拟了完整的底层硬件，包括处理器、物理内存、时钟、外设等**，使得为原始硬件设计的操作系统或其它系统软件完全不做任何修改就可以在虚拟机中运行。

​		**超虚拟化（Paravirtualization）**这是一种修改 Guest OS 部分访问特权状态的代码以便直接与 VMM 交互的技术。在超虚拟化虚拟机中，部分硬件接口以软件的形式提供给客户机操作系统，这可以通过 Hypercall（VMM 提供给 Guest OS 的直接调用，与系统调用类似）的方式来提供。

​		这种分类并不是绝对的，一个优秀的虚拟化软件往往融合了多项技术。例如 **VMware Workstation 是一个著名的全虚拟化的 VMM**，但是它使用了一种被称为动态二进制翻译的技术把对特权状态的访问转换成对影子状态的操作，从而避免了低效的 Trap-And-Emulate 的处理方式，这与超虚拟化相似，只不过超虚拟化是静态地修改程序代码。对于超虚拟化而言，如果能利用硬件特性，那么虚拟机的管理将会大大简化，同时还能保持较高的性能。



### Type1虚拟化和Type2虚拟化

​		Type1类型也叫裸金属架构，这类虚拟化层直接运行在硬件之上，没有所谓的宿主机操作系统。他们直接控制硬件资源以及客户机。典型地如xen和vmware ESXI

​		Type2类型也叫宿主机型架构，这类VMM通常就是宿主机操作系统上的一个应用程序，像其他应用程序一样受宿主机操作系统的管理，通常抽象为进程。例如，VMware workstation、KVM



## 三、KVM虚拟化技术简介

### KVM架构

KVM架构：客户模式（执行非I/O的客户代码，虚拟机运行在这个模式下）+用户模式（代表用户执行I/O指令，qemu运行在这个模式下）+内核模式（实现客户模式的切换，kvm 模块工作在这个模式下）

KVM虚拟化的核心主要由以下两个模块组成

1. **KVM内核模块**，它属于标准Linux内核的一部分，是一个专门提供虚拟化功能的模块，主要负责**CPU和内存的虚拟化**，包括：客户机的创建、虚拟内存的分配、CPU执行模式的切换、vCPU寄存器的访问、vCPU的执行。KVM模块是KVM虚拟化的核心模块，它在内核中有两部分组成，一个是处理器架构无关的部分，可以用lsmod命令看到，叫做kvm模块；另一个是处理器架构相关的部分，在intel平台上就是kvm_intel这个内核模块。KVM主要功能是初始化CPU硬件，打开虚拟化模式，然后将虚拟客户机运行在虚拟机模式下，并对虚拟客户机的运行提供一定的支持。
2. **QEMU用户态工具**，它是一个普通的Linux进程，为客户机提供**设备模拟**的功能，包括模拟BIOS、数据总线、磁盘、网卡、显卡、声卡、键盘、鼠标等。同时它通过系统调用与内核态的KVM模块进行交互。作为一个存在已久的虚拟机监控器软件，QEMU的代码中有完整的虚拟机实现，包括处理器虚拟化、内存虚拟化，以及KVM也会用到的设备模拟功能。总之，QEMU既是一个功能完整的虚拟机监控器，也在QEMU/KVM软件栈中承担设备模拟的功能。
3. 在KVM虚拟化架构下，每一个客户机就是一个QEMU进程，在一个宿主机上有多少个虚拟机就会有多少个QEMU进程；客户机中的每一个虚拟CPU对应QEMU进程中的一个执行线程；一个宿主机只有一个KVM内核模块，所有客户机都与这个内核模块进行交互。



​		kvm就是一个普通的linux进程，由linux内核调度程序进行调度，vm因此可以使用linux内核已有的功能。vm的执行本质就是vm中cpu的执行，因此vm的每个cpu就是普通的linux进程。

​		KVM有一个内核模块叫 kvm.ko ，kvm.ko只提供 CPU 和内存的虚拟化，而针对于IO及其他硬件设备（网络及存储等）的虚拟化，则是交给qemu实现，qemu运行在用户态通过/dev/kvm接口设置一个客户机虚拟机服务器的地址空间，向kvm提供模拟的I/O，并且将它的视频显示映射回宿主的显示屏。



### KVM上层管理工具

1. libvirt libvirt是使用最广泛的对KVM虚拟化进行管理的工具和**应用程序接口**，已经是事实上的虚拟化接口标准，后部分介绍的许多工具都是基于libvirt的API来实现的。作为通用的虚拟化API，libvirt不但能管理KVM，还能管理VMware、Hyper-v等其他虚拟化方案
2. virsh virsh是一个常用的管理KVM虚拟化的**命令行工具**，对于系统管理员在单个宿主机上进行运维操作，virsh命令行可能是最佳选择。virsh是用c语言编写的一个使用libvirt API的虚拟化管理工具，其源代码也是在libvirt这个开源项目中的。
3. virt-manager virt-manager是专门针对虚拟机的**图形化管理软件**，底层与虚拟化交互的部分仍然是调用libvirt API来操作的。virt-manager除了提供虚拟机生命周期（包括：创建、启动、停止、打快照、动态迁移等）管理的基本功能，还提供了性能和资源使用率的监控。



### KVM原理

ioctl（定义)：专用于设备输入输出操作的系统调用

libvirt：KVM管理工具

用户模式的qemu利用libkvm通过ioctl进入内核模式，kvm模块未虚拟机创建虚拟内存，虚拟CPU后执行VMLAUCH指令进入客户模式。加载Guest OS并执行。如果Guest OS 发生外部中断或者影子页表缺页之类的情况，会暂停Guest OS的执行，退出客户模式出行异常处理，之后重新进入客户模式，执行客户代码。如果发生I/O事件或者信号队列中有信号到达，就会进入用户模式处理。



## 四、KVM部署

### 开启虚拟化支持

- 查看CPU支持的功能中是否存在
  - vmx:INTEL的虚拟化功能
  - svm:AMD的虚拟化功能

```bash
[root@localhost ~]# cat /proc/cpuinfo |grep -E 'vmx|svm'
```



### 环境准备

kvm有良好的图形化支持，所以我们可以先装一下桌面组件

安装epel源并且更新系统中所有的软件包

```bash
[root@localhost ~]# yum install -y epel-release
[root@localhost ~]# yum upgrade -y
```

安装GNOME 桌面软件包组

```bash
[root@localhost ~]# yum grouplist
[root@localhost ~]# yum groupinstall -y "GNOME 桌面"/yum groupinstall -y "Server with GUI"
```

启动图形化

```bash
#开机自动开启图形化
[root@localhost ~]# systemctl set-default graphical.target
#启动图形化
[root@localhost ~]# startx &
```



开启/关闭图形化方式

```bash
# 开启图形化
# 临时启动
systemctl isolate graphical.target

# 永久启动
systemctl set-default graphical.target

# 关闭图形化
# 临时关闭
systemctl isolate multi-user.target

# 永久关闭
systemctl set-default multi-user.target
```



### KVM部署

#### 清理环境

清理干净KVM相关的预装环境

```bash
[root@localhost ~]# yum remove `rpm -qa |egrep 'qemu|virt|kvm'` -y
[root@localhost ~]# rm -rf /var/lib/libvirt /etc/libvirt/
```



#### 安装KVM

```bash
[root@localhost ~]# uname -r
3.10.0-957.el7.x86_64
[root@localhost ~]# cat /etc/centos-release
CentOS Linux release 7.6.1810 (Core)
centos7:
[root@localhost ~]# yum install *qemu* *virt* librbd1-devel -y
(在安装虚拟机出错的情况下，一般是操作系统的版本问题)
rockylinux9:
[root@localhost ~]#yum install -y qemu-kvm libvirt virt-manager virt-install bridge-utils virt-top libguestfs-tools bridge-utils virt-viewer
```

主要程序

- qemu-KVM:主包
- libvirt:API接口
- virt-manager:图形管理程序

在所谓的kvm技术中，应用到的其实有2个东西：qemu+kvm

​		kvm负责CPU虚拟化+内存虚拟化，实现了CPU和内存的虚拟化，但kvm不能模拟其他设备

​		qemu是模拟IO设备(网卡，磁盘)，kvm加上qemu之后就能实现真正意义上服务器的虚拟化

因为用到了上面两个东西所以一般都称之为qemu-kvm

libvirt则是调用kvm虚拟化技术的接口用于管理的，用libvirt管理方便



### 启动服务

```bash
[root@localhost ~]# systemctl start libvirtd
[root@localhost ~]# systemctl enable libvirtd
[root@localhost ~]# lsmod |grep kvm        # 查看kvm模块加载
kvm_intel             183621  0 
kvm                   586948  1 kvm_intel
irqbypass              13503  1 kvm
```



### 图形化中检查

在图形化命令行中输入`virt-manager`就可以看到kvm界面



### 脚本部署KVM

```shell
#!/bin/bash
# 检查是否以root用户运行
if [[ $EUID -ne 0 ]]; then
   echo "该脚本需要root权限，请以root用户运行。" 1>&2
   exit 1
fi

# 检查KVM是否已安装
if ! command -v virsh &> /dev/null; then
    echo "KVM未安装，请先安装KVM。"
    exit 1
fi

# 提示用户输入虚拟机信息
echo "请输入虚拟机的名称："
read vm_name

echo "请输入虚拟机的内存大小（单位：MB）："
read vm_memory

echo "请输入虚拟机的CPU核心数："
read vm_cpus

echo "请输入虚拟机的磁盘大小（单位：GB）："
read vm_disk_size

echo "请输入虚拟机的安装镜像路径："
read vm_iso_path

# 检查输入是否为空
if [[ -z "$vm_name" || -z "$vm_memory" || -z "$vm_cpus" || -z "$vm_disk_size" || -z "$vm_iso_path" ]]; then
    echo "输入的虚拟机信息不完整，请重新运行脚本并填写所有信息。"
     exit1
fi

# 检查镜像文件是否存在
if [[ ! -f "$vm_iso_path" ]]; then
    echo "指定的安装镜像文件不存在，请检查路径是否正确。"
    exit 1
fi

# 创建虚拟机
virt-install \
    --name "$vm_name" \
    --ram "$vm_memory" \
    --vcpus "$vm_cpus" \
    --disk path=/var/lib/libvirt/images/${vm_name}.qcow2,size="$vm_disk_size",format=qcow2 \
    --cdrom "$vm_iso_path" \
    --os-type linux \
    --os-variant ubuntu22.04 \
    --network bridge:virbr0 \
    --graphics vnc,listen=0.0.0.0 \
    --noautoconsole \
    --console pty,target_type=serial

echo "虚拟机创建完成！"
echo "您可以通过以下命令连接到虚拟机的VNC控制台："
echo "vncviewer 127.0.0.1:1"
```



## 五、GuestOS创建

在KVM中安装虚拟机，常见的有一下三种方式：

1. 通过KVM图形化安装(简单好用)
2. 通过Linux命令行安装(过程较为复杂)
3. 通过Cockpit安装(使用较少，不推荐)



### 图形化创建

上传ISO镜像到虚拟机中

```bash
[root@localhost ISO]# ls -lh
总用量 1021M
-rw-r--r--. 1 root root 918M 6月  19 15:31 CentOS-7-x86_64-Minimal-1810.iso
```

打开KVM图形化

```bash
[root@localhost ~]# virt-manager
```

开始创建虚拟机，然后点击完成，就可以进入系统的安装界面。然后按照之前安装虚拟机的步骤安装即可，注意勾选网络连接



### Linux命令行创建

有些情况下，没有图形化，这个时候就需要用到命令行创建虚拟机

前置准备

在安装的时候需要用到FTP服务，以及需要关闭防火墙和SElinux

```bash
[root@localhost ~]# systemctl stop firewalld
[root@localhost ~]# setenforce 0

# 安装FTP
[root@localhost ~]# yum -y install vsftpd
[root@localhost ~]# systemctl restart vsftpd
[root@localhost ~]# cd /var/ftp/
[root@localhost ftp]# mkdir centos7
[root@localhost ftp]# mount /root/CentOS-7-x86_64-Minimal-1810.iso /var/ftp/centos7/
#将我们的镜像文件挂载在上面
mount: /dev/loop0 is write-protected, mounting read-only
```

安装virt-install工具

```bash
[root@localhost ~]# yum -y install virt-install
[root@localhost ~]# df -h
文件系统                 容量  已用  可用 已用% 挂载点
devtmpfs                 1.9G     0  1.9G    0% /dev
tmpfs                    1.9G     0  1.9G    0% /dev/shm
tmpfs                    1.9G   14M  1.9G    1% /run
tmpfs                    1.9G     0  1.9G    0% /sys/fs/cgroup
/dev/mapper/centos-root   17G  9.0G  8.1G   53% /
/dev/sda1               1014M  209M  806M   21% /boot
tmpfs                    378M   44K  378M    1% /run/user/0
/dev/sr0                 918M  918M     0  100% /run/media/roo/CentOS 7 x86_64
/dev/loop0               918M  918M     0  100% /var/ftp/centos7u3
# 查看根目录挂载点剩余空间
# /dev/mapper/centos-root   17G  9.0G  8.1G   53% /
```

创建虚拟机

```bash
[root@localhost ~]# virt-install --connect qemu:///system -n vm1 -r 2048 --disk path=/var/lib/libvirt/images/vm1.img,size=8  --os-type=linux --os-variant=centos7.0 --vcpus=2  --location=ftp://192.168.88.10/centos7 -x console=ttyS0 --nographics
# size大于3，centos系统自身需要一定空间
```

参数说明：

- `--connect qemu:///system`: 连接到系统级别的 QEMU/KVM 虚拟化管理
- `-n vm1`: 为虚拟机指定名称为 `vm1`
- `-r 2048`: 为虚拟机分配 2048MB 内存
- `--disk path=/var/lib/libvirt/images/vm1.img,size=8`: 创建一个 8GB 大小的虚拟磁盘文件 `/var/lib/libvirt/images/vm1.img`
- `--os-type=linux`: 指定操作系统类型为 Linux
- `--os-variant=centos7.0`: 指定操作系统变体为 CentOS 7.0
- `--vcpus=2`: 为虚拟机分配 2 个 vCPU
- `--location=ftp://192.168.88.10/centos7`: 指定从 FTP 服务器 `192.168.88.10` 上的 `centos7` 目录安装 CentOS
- `-x console=ttyS0`: 将控制台重定向到串口设备 `ttyS0`
- `--nographics`: 不使用图形界面安装



如果本地有镜像可以直接使用本地镜像进行安装：

```bash
[root@localhost ~]# virt-install -name=vm2 --ram 512 --vcpus=1 --disk path=/var/lib/libvirt/images/vm2.qcow2,size=5,bus=virtio --accelerate --cdrom /root/CentOS-7-x86_64-Minimal-1810.iso --vnc --vncport=-1 --vnclisten=0.0.0.0 --network bridge=virbr0,model=virtio --noautoconsole
```

参数说明：

- `-name=vm2`: 为虚拟机指定名称为 `vm2`
- `--ram 512`: 为虚拟机分配 512MB 内存
- `--vcpus=1`: 为虚拟机分配 1 个 vCPU
- `--disk path=/var/lib/libvirt/images/vm2.qcow2,size=5,bus=virtio`: 创建一个 5GB 大小、使用 virtio 总线的 QCOW2 格式虚拟磁盘文件 `/var/lib/libvirt/images/vm2.qcow2`
- `--accelerate`: 启用硬件加速
- `--cdrom /root/CentOS-7-x86_64-Minimal-1810.iso`: 指定本地 CentOS 7 Minimal ISO 作为安装介质
- `--vnc --vncport=-1 --vnclisten=0.0.0.0`: 启用 VNC 远程控制台,监听所有网络接口
- `--network bridge=virbr0,model=virtio`: 使用 `virbr0` 网桥并采用 virtio 网卡模型
- `--noautoconsole`: 不自动连接到虚拟机控制台
- 安装设置

```bash
Text mode provides a limited set of installation options. It does not offer
custom partitioning for full control over the disk layout. Would you like tVNC mode instead?

 1) Start VNC
 2) Use text mode

  Please make your choice from above ['q' to quit | 'c' to continue |
  'r' to refresh]:2
# 此处如果客户机电脑有图形化，可以选择启动vnc连接，然后正常安装，如果没有图形化，可以选择2进入命令行安装

======================================================================================================================================================Installation
[anaconda] 1:main* 2:shell  3:log  4:storage-lo> Switch tab: Alt+Tab | Help 1) [x] Language settings                 2) [!] Time settings
        (English (United States))                (Timezone is not set.)
 3) [!] Installation source               4) [!] Software selection
        (Processing...)                          (Processing...)
 5) [!] Installation Destination          6) [x] Kdump
        (No disks selected)                      (Kdump is enabled)
 7) [x] Network configuration             8) [!] Root password
        (Wired (eth0) connected)                 (Password is not set.)
 9) [!] User creation
        (No user will be created)
  Please make your choice from above ['q' to quit | 'b' to begin installation |
  'r' to refresh]:

# 这里的安装选项跟图形化中的选项一致，一次设置存储，网络，密码，语言，时区即可开始安装
```

- 在宿主机下访问

```bash
[root@localhost ~]# virsh console vm1
```

提示：进去之后回车多次即可通过账号密码登录，退出执行`Ctrl+]`



### cockpit创建

安装cockpit

```bash
[root@localhost ~]# yum install cockpit -y
[root@localhost ~]# systemctl start cockpit
```

访问9090端口，创建虚拟机



## 六、KVM基础管理

### 常用管理命令

```bash
[root@localhost ~]# virsh list --all
[root@localhost ~]# virsh dumpxml vm1    # 查看虚拟机的配置
[root@localhost ~]# virsh dumpxml vm1 > vm1.xml.old
[root@localhost ~]# virsh edit vm1
[root@localhost ~]# virsh start vm1
[root@localhost ~]# virsh suspend vm1    # 暂停虚拟机
[root@localhost ~]# virsh resume vm1    # 恢复虚拟机
[root@localhost ~]# virsh shutdown vm1    # 关闭虚拟机
[root@localhost ~]# virsh reboot vm1    # 重启虚拟机
[root@localhost ~]# virsh reset vm1        # 重置虚拟机
[root@localhost ~]# virsh undefine vm1    # 删除虚拟机
[root@localhost ~]# virsh autostart vm1    # 开机自启动虚拟机
[root@localhost ~]# virsh autostart --disable vm1    # 取消开机自动启动虚拟机
[root@localhost ~]# virsh list --all --autostart    # 查看所有开机自启动虚拟机
[root@localhost ~]# ls /etc/libvirt/qemu/autostart    # 有开机自动的虚拟机时自动创建
[root@localhost ~]# virsh destroy vm1    # 强行删除虚拟机，即使虚拟机是开启状态
```



### 虚拟机克隆

```bash
[root@localhost ~]# virt-clone -o vm1 --auto-clone
正在分配 'vm1-clone.qcow2'                           |  10 GB  00:19     
正在分配 'vm1-1-clone.qcow2'                         |  10 GB  00:00     

成功克隆 'vm1-clone'。
```

- 可以指定克隆之后的名字

```bash
[root@localhost ~]# virt-clone -o vm1 -n vm2 --auto-clone
正在分配 'vm3.qcow2'                                 |  10 GB  00:03     
正在分配 'vm1-1-clone-1.qcow2'                       |  10 GB  00:00     

成功克隆 'vm3'。
```

- 指定克隆之后虚拟机的磁盘镜像文件

```bash
[root@localhost ~]# virt-clone -o vm1 -n vm4 --auto-clone -f /var/lib/libvirt/images/vm4.qcow2
正在分配 'vm4.qcow2'                                 |  10 GB  00:02     
正在分配 'vm1-1-clone-2.qcow2'                       |  10 GB  00:00     

成功克隆 'vm4'
```



### 虚拟机快照

#### 创建外部快照并指定存放位置

```bash
[root@localhost ~]# virsh snapshot-create-as \
--domain kvm-vm1 \
--name kvm-vm1.snap \
--description "Snapshot for kvm-vm1" \
--disk-only \
--diskspec vda,file=/kvm/data/kvm-vm1-snapshot.qcow2 \
--atomic
```

**参数说明：**

- `--domain kvm-vm1`：指定虚拟机名称。
- `--name my_snapshot`：快照名称。
- `--description "Snapshot for kvm-vm1"`：快照描述。
- `--disk-only`：只快照磁盘，不包含内存状态。
- `--diskspec vda,file=/path/to/snapshots/kvm-vm1-snapshot.qcow2`：指定磁盘快照文件路径。
- `--atomic`：确保快照操作的原子性。

#### 创建内部快照

内部快照将数据存储在同一个 qcow2 虚拟磁盘镜像中，无法指定单独的存放位置。可以使用以下命令创建内部快照：

```bash
[root@localhost ~]# virsh snapshot-create-as \
--domain kvm-vm1 \
--name kvm-vm1.snap \
--description "Internal snapshot for kvm-vm1" \
--atomic
```

#### 查看快照

查看虚拟机的快照列表：

```bash
[root@localhost ~]# virsh snapshot-list kvm-vm1
 名称           生成时间                    状态
-----------------------------------------------------
 kvm-vm1.snap   2025-03-20 20:28:34 +0800   shutoff
```

- 快照就相当于一个还原点，可以看到qcow2的磁盘中有快照的存在

```bash
[root@localhost ~]# virsh snapshot-create-as vm1 vm1.snap
已生成域快照 vm1.snap
[root@localhost ~]# qemu-img info /var/lib/libvirt/images/vm1.qcow2 
image: /var/lib/libvirt/images/vm1.qcow2
file format: qcow2
virtual size: 10G (10737418240 bytes)
disk size: 10G
cluster_size: 65536
Snapshot list:
ID        TAG                 VM SIZE                DATE       VM CLOCK
1         vm1.snap                  0 2020-10-02 15:10:06   00:00:00.000
Format specific information:
    compat: 1.1
    lazy refcounts: true
```

- 还原快照

```bash
[root@localhost ~]# virsh snapshot-revert vm1 vm1.snap --snapshotname kvm-vm1.snap
```

- 删除快照

```bash
[root@localhost ~]# virsh snapshot-delete --snapshotname vm1.snap vm1 --snapshotname kvm-vm1.snap
已删除域快照 vm1.snap

[root@localhost ~]# virsh snapshot-list vm1
 名称               生成时间              状态
------------------------------------------------------------
```



## 七、GusetOS修改配置

这些基本上都可以通过**图形化**的方式进行修改

### 添加磁盘

**创建新的空磁盘卷**

```bash
[root@localhost ~]# cd /var/lib/libvirt/images/
[root@localhost images]# qemu-img create -f qcow2 vm1-1.qcow2 10G
Formatting 'vm1-1.qcow2', fmt=qcow2 size=10737418240 encryption=off cluster_size=65536 lazy_refcounts=off
```

**修改配置文件**

- 比如添加磁盘，那就添加如下配置，slot需要修改

```bash
[root@localhost ~]# vim /etc/libvirt/qemu/vm1.xml
<disk type='file' device='disk'>
  <driver name='qemu' type='qcow2'/>
  <source file='/var/lib/libvirt/images/vm1-1.qcow2'/>
  <target dev='vdb' bus='virtio'/>
  <address type='pci' domain='0x0000' bus='0x00' slot='0x17' function='0x0'/>
</disk>
```

> 特别注意:centos8里面硬盘和网卡的添加已经不能修改slot了，要求修改的是bus地址

**重新定义虚拟机**

```bash
[root@localhost qemu]# virsh define vm1.xml 
定义域 vm1（从 vm1.xml）
```

**检查配置是否升级**

```bash
[root@localhost ~]# virsh start vm1
域 vm1 已开始

[root@localhost ~]# virsh console vm1
连接到域 vm1
换码符为 ^]

CentOS Linux 7 (Core)
Kernel 3.10.0-957.el7.x86_64 on an x86_64

vm1 login: root
密码：
Last login: Fri Oct  2 13:27:39 on tty1
[root@vm1 ~]# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0              11:0    1 1024M  0 rom  
vda             252:0    0   10G  0 disk 
├─vda1          252:1    0    1G  0 part /boot
└─vda2          252:2    0    9G  0 part 
  ├─centos-root 253:0    0    8G  0 lvm  /
  └─centos-swap 253:1    0    1G  0 lvm  [SWAP]
vdb             252:16   0   10G  0 disk
```



### CPU添加

- 热添加的意思就是可以在虚拟机不重启的情况下，完成CPU的升级
- 不支持热减少
- GuestOS只支持Linux发行套件，不支持其他系统

```bash
[root@localhost ~]# virsh edit vm1
<domain type='kvm'>
  <name>vm1</name>
  <uuid>b0c26bcd-ce6b-46ef-96e8-eb12ea52e5e0</uuid>
  <memory unit='KiB'>1048576</memory>
  <currentMemory unit='KiB'>1048576</currentMemory>
  <vcpu placement='auto' current='1'>4</vcpu>    # 最大cpu使用4个，默认是1个
[root@localhost ~]# virsh vcpucount vm1
最大值    配置         4
当前       配置         1
[root@localhost ~]# virsh start vm1
域 vm1 已开始
[root@localhost ~]# virsh dominfo vm1
Id:             2                            
名称：       vm1
UUID:           b0c26bcd-ce6b-46ef-96e8-eb12ea52e5e0
OS 类型：    hvm
状态：       running
CPU：          1                                # 可以看到目前只有一颗CPU
CPU 时间：   16.6s
最大内存： 1048576 KiB
使用的内存： 1048576 KiB
持久：       是
自动启动： 禁用
管理的保存： 否
安全性模式： none
安全性 DOI： 0
[root@localhost ~]# virsh setvcpus vm1 2 --live
[root@localhost ~]# virsh dominfo vm1
Id:             2
名称：       vm1
UUID:           b0c26bcd-ce6b-46ef-96e8-eb12ea52e5e0
OS 类型：    hvm
状态：       running
CPU：          2                            # 现在有两颗CPU
CPU 时间：   19.0s
最大内存： 1048576 KiB
使用的内存： 1048576 KiB
持久：       是
自动启动： 禁用
管理的保存： 否
安全性模式： none
安全性 DOI： 0
```

热添加CPU配置

```bash
[root@localhost kvm]# virsh setvcpus kvm-vm1 3 --live --config
[root@localhost kvm]# virsh dominfo kvm-vm1
Id:             2
Name:           kvm-vm1
UUID:           3a74b2c9-0d7e-4d95-bbd9-2dbd6ef51d01
OS Type:        hvm
State:          running
CPU(s):         3                # 可以看到当前kvm-vm1中cpu是三个核心
CPU time:       116.8s
Max memory:     1048576 KiB
Used memory:    1048576 KiB
Persistent:     yes
Autostart:      disable
Managed save:   no
Security model: selinux
Security DOI:   0
Security label: system_u:system_r:svirt_t:s0:c57,c931 (permissive)
```



### 内存热添加

- 将最大可用内存调整为2048M

```bash
[root@localhost ~]# virsh edit vm1
<domain type='kvm'>
  <name>vm1</name>
  <uuid>b0c26bcd-ce6b-46ef-96e8-eb12ea52e5e0</uuid>
  <memory unit='KiB'>2097152</memory>        # 最大可用内存
  <currentMemory unit='KiB'>1048576</currentMemory>        # 内存当前大小
  <vcpu placement='auto' current='1'>4</vcpu>
```

- 重启虚拟机让其生效

```bash
[root@localhost ~]# virsh reboot vm1
```

- 查看现在内存大小

```bash
[root@localhost ~]# virsh qemu-monitor-command vm1 --hmp --cmd info balloon
balloon: actual=1024
```

- 调整内存到2048M

```bash
[root@localhost ~]# virsh qemu-monitor-command vm1 --hmp --cmd balloon 2048
[root@localhost ~]# virsh qemu-monitor-command vm1 --hmp --cmd info balloon
balloon: actual=2048
```

> 注意！不推荐调整生产环境中的虚拟机内存，有崩溃的可能



## 八、KVM存储

- kvm配置一个目录当做存储磁盘镜像(存储卷)的目录，这个目录为存储池
- 默认存储池：`/var/lib/libvirt/images/`

### 使用KVM存储池

  为简化KVM存储管理的目的，可以创建存储池。在宿主机上创建存储池，可以简化KVM存储设备的管理。采用存储池的方式还可以实现对提前预留的存储空间的分配。这种策略对于大型应用环境很有效，存储管理员和创建虚拟机的管理经常不是同一个人。这样，在创建首台虚拟机之前先完成KVM存储池的创建是很好的方法。



### 存储池创建使用相关命令

存储池是一个由libvirt管理的文件、目录或存储设备，提供给虚拟机使用。存储池被分为存储卷，这些存储卷保存虚拟镜像或连接到虚拟机作为附加存储。libvirt通过存储池的形式对存储进行统一管理、简化操作。对于虚拟机操作来说，存储池和卷并不是必需的。

- 创建基于文件夹的存储池(目录)，并且定义存储池与其目录

```bash
[root@localhost ~]# mkdir -p /data/vmfs
[root@localhost ~]# virsh pool-define-as vmdisk --type dir --target /data/vmfs
定义池 vmdisk·
```

- 创建已定义的存储池，然后查看已定义的存储池，存储池不激活无法使用

```bash
# 构建池 vmdisk
[root@localhost ~]# virsh pool-build vmdisk

# 查看池
[root@localhost ~]# virsh pool-list --all
 名称               状态     自动开始
-------------------------------------------
 default              活动     是       
 vmdisk               不活跃  否       

# 启用存储池
[root@localhost ~]# virsh pool-start vmdisk
池 vmdisk 已启动

# 设置开机自动启动
[root@localhost ~]# virsh pool-autostart vmdisk
池 vmdisk 标记为自动启动

[root@localhost ~]# virsh pool-list --all
 名称               状态     自动开始
-------------------------------------------
 default              活动     是       
 vmdisk               活动     是
```

- 在存储池中创建虚拟机存储卷

```bash
[root@localhost ~]# virsh vol-create-as vmdisk vm1-2.qcow2 10G --format qcow2
创建卷 vm1-2.qcow2
```

> 注1：KVM存储池主要是体现一种管理方式，可以通过挂载存储目录，LVM逻辑卷的方式创建存储池，虚拟机存储卷创建完成后，剩下的操作与无存储卷的方式无任何区别
>
> 注2：KVM存储池也用于虚拟机迁移任务



### 存储池删除相关管理命令

- 在存储池中删除虚拟机存储卷

```bash
[root@localhost ~]# virsh vol-delete --pool vmdisk vm1-2.qcow2
```

- 取消激活存储池

```bash
[root@localhost ~]# virsh pool-destroy vmdisk
```

- 删除存储池定义的目录

```bash
[root@localhost ~]# virsh pool-delete vmdisk
```

- 取消定义存储池

```bash
[root@localhost ~]# virsh pool-undefine vmdisk
```



### 磁盘格式

- raw：原始格式，性能最好 直接占用你一开始给多少 系统就占多少 不支持快照
- qcow：性能远不能和raw相比，所以很快夭折了，所以出现了qcow2。（性能低下 早就被抛弃）
- qcow2：性能上还是不如raw，但是raw不支持快照，qcow2支持快照。



#### 写时拷贝(copy on write)

raw立刻分配空间，不管你有没有用到那么多空间 qcow2只是承诺给你分配空间，但是只有当你需要用空间的时候，才会给你空间。最多只给你承诺空间的大小，避免空间浪费



#### 不同格式磁盘大小比较

- 可以看到qcow2的文件大小初始很小

```bash
[root@localhost ~]# qemu-img create -f qcow2 test1.qcow2 1G
Formatting 'test1.qcow2', fmt=qcow2 size=1073741824 encryption=off cluster_size=65536 lazy_refcounts=off 
[root@localhost ~]# qemu-img create -f raw test2.raw 1G
Formatting 'test2.raw', fmt=raw size=1073741824 
[root@localhost ~]# ls -lh test1.qcow2 test2.raw 
-rw-r--r-- 1 root root 193K 10月  2 14:39 test1.qcow2
-rw-r--r-- 1 root root 1.0G 10月  2 14:39 test2.raw
```



### 虚拟机磁盘挂载至宿主机

- 当虚拟机无法启动的时候，可以将虚拟磁盘文件直接挂载到主机上，以获得其中的文件
- 查看磁盘镜像分区信息

```bash
[root@localhost ~]# virt-df -h -d vm1
文件系统                            大小 已用空间 可用空间 使用百分比%
vm1:/dev/sda1                            1014M       100M       914M   10%
vm1:/dev/centos/root                      8.0G       971M       7.0G   12%
[root@localhost ~]# virt-filesystems -d vm1
/dev/sda1
/dev/centos/root
```

- 挂载磁盘镜像分区，挂载完成之后可以看到vm1虚拟机根目录下的文件

```bash
[root@localhost ~]# guestmount -d vm1 -m /dev/centos/root --rw /root/vm1
[root@localhost ~]# ls vm1/
bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
boot  etc  lib   media  opt  root  sbin  sys  usr
```

- 取消挂载

```bash
[root@localhost ~]# guestunmount /root/vm1
```



## 九、网络管理

- KVM有如下三种网络连接方式
  - nat
  - isolated
  - bridge
- 虚拟交换机
  - linux-bridge(linux自带)
  - ovs(open-vswitch)



**NAT网络拓扑**

- 查看Linux网卡桥接
- NAT网络下
  - 宿主机和虚拟机可以互相访问
  - 虚拟机可以访问宿主机所在的物理网络
  - 宿主机所在的物理网络无法主动访问虚拟机

```bash
[root@localhost ~]# brctl show
bridge name     bridge id               STP enabled     interfaces
virbr0          8000.525400a816c6       yes             virbr0-nic
                                                        vnet0
virbr1          8000.525400a15e56       yes             virbr1-nic
```



**隔离网络拓扑**

- 在隔离网络下
  - 虚拟机和宿主机可以互相访问
  - 宿主机不提供NAT功能，所以虚拟机与宿主机所在的物理网络无法互相访问



**桥接网络拓扑**

- 在桥接网络下
  - 虚拟机和宿主机可以互相访问
  - 虚拟机和宿主机同处于一个物理网络下
  - 物理网络可以直接访问虚拟机，虚拟机也可以直接访问物理网络

可以为网卡添加网络接口

```bash
[root@localhost ~]# brctl delif virbr0 vnet0
[root@localhost ~]# brctl addif virbr0 vnet0
```



### 配置桥接网络

在桥接网络下

- 虚拟机和宿主机可以互相访问
- 虚拟机和宿主机同处于一个物理网络下
- 物理网络可以直接访问虚拟机，虚拟机也可以直接访问物理网络

在宿主机上

- 新建桥接网卡配置文件

```bash
[root@localhost ~]# vim /etc/sysconfig/network-scripts/ifcfg-br0
TYPE=Bridge
NAME=br0
DEVICE=br0
ONBOOT="yes"
BOOTPROTO=dhcp
```

- 修改宿主机网卡配置文件

```bash
[root@localhost ~]# cp /etc/sysconfig/network-scripts/ifcfg-ens33{,.old}
[root@localhost ~]# vim /etc/sysconfig/network-scripts/ifcfg-ens33
DEVICE="ens33"
ONBOOT="yes"
BRIDGE=br0
```

- 重启`libvirtd`和`network`服务

```bash
[root@localhost ~]# systemctl restart network libvirtd
```

- 修改虚拟机配置，然后启动虚拟机检查网络是否生效

```bash
[root@localhost ~]# virsh edit vm1
<interface type='bridge'>
    <mac address='52:54:00:63:99:c1'/>
    <source bridge='br0'/>
    <model type='virtio'/>
    <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x0'/>
</interface>
```

删除桥接网卡步骤

- 删除br0的配置文件
- 修改正常网卡的配置文件

```
cd /etc/sysconfig/network-scripts/
rm -rf ifcfg-br0
rm -rf ifcfg-ens33
mv ifcfg-ens33.old ifcfg-ens33
```

- 重启系统



### 配置NAT网络

NAT网络下

- 宿主机和虚拟机可以互相访问
- 虚拟机可以访问宿主机所在的物理网络
- 宿主机所在的物理网络无法主动访问虚拟机

- 复制默认的NAT网络配置

```bash
[root@localhost ~]# cd /etc/libvirt/qemu/networks/
[root@localhost networks]# cp default.xml nat1.xml
```

- 修改配置文件

```bash
[root@localhost networks]# vim nat1.xml 
<network>
  <name>nat1</name>
  <uuid>64fa0e8c-b799-4e27-8b23-7c4d512b50e7</uuid>
  <forward mode='nat'/>
  <bridge name='virbr1' stp='on' delay='0'/>
  <mac address='52:54:00:8a:1e:8c'/>
  <ip address='192.168.66.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.66.2' end='192.168.66.254'/>
    </dhcp>
  </ip>
</network>
```

- 重启`libvirtd`服务，然后激活新的nat网络

```bash
[root@localhost networks]# systemctl restart libvirtd
[root@localhost networks]# virsh net-define nat1.xml
[root@localhost ~]# virsh net-autostart nat1
网络nat1标记为自动启动
[root@localhost ~]# virsh net-start nat1
网络 nat1 已开始
[root@localhost ~]# virsh net-list
 名称               状态     自动开始  持久
----------------------------------------------------------
 default              活动     是           是
 nat1                 活动     是           是
```

- 修改虚拟机配置，然后启动虚拟机检查网络是否生效

```bash
[root@localhost ~]# virsh edit vm1
<interface type='network'>
  <mac address='52:54:00:63:99:c1'/>
  <source network='nat1'/>
  <model type='virtio'/>
  <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x0'/>
</interface>
```



### 配置隔离网络

在隔离网络下

- 虚拟机和宿主机可以互相访问
- 宿主机不提供NAT功能，所以虚拟机与宿主机所在的物理网络无法互相访问

复制默认的NAT网络配置

```bash
[root@localhost ~]# cd /etc/libvirt/qemu/networks/
[root@localhost networks]# cp default.xml isolate1.xml
```

修改配置文件

和创建NAT网络一样，不过需要额外的在配置文件中删除如下一行

```bash
[root@localhost networks]# vim isolate1.xml 
<forward mode='nat'/>
```

重启`libvirtd`服务，然后激活新的nat网络

```bash
[root@localhost networks]# systemctl restart libvirtd
[root@localhost networks]# virsh net-define isolate1.xml
[root@localhost ~]# virsh net-autostart isolate1
网络isolate1标记为自动启动
[root@localhost ~]# virsh net-start isolate1
网络 isolate1 已开始
[root@localhost ~]# virsh net-list
 名称               状态     自动开始  持久
----------------------------------------------------------
 default              活动     是           是
 isolate1             活动     是           是
 nat1                 活动     是           是
```

修改虚拟机配置，然后启动虚拟机检查网络是否生效

```bash
[root@localhost ~]# virsh edit vm1
<interface type='network'>
  <mac address='52:54:00:63:99:c1'/>
  <source network='isolate1'/>
  <model type='virtio'/>
  <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x0'/>
</interface>
```



### virbr0网卡

virbr0是kvm默认创建的一个Bridge，其作用是为连接其上的虚拟机网卡提供DHCP功能

virbr0默认分配一个IP `192.168.122.1`，并为连接其上的其他虚拟机网卡提供网关和DHCP功能

virbr0使用dnsmasq提供DHCP服务

```bash
[root@localhost ~]# ps -elf |grep dnsmasq
5 S nobody     1325      1  0  80   0 - 13475 poll_s 16:02 ?        00:00:00 /usr/sbin/dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/default.conf --leasefile-ro --dhcp-script=/usr/libexec/libvirt_leaseshelper
```

在`/var/lib/libvirt/dnsmasq/`目录下有一个virbr0.status文件，里面记录了所有获得IP的记录

```bash
[root@localhost ~]# cat /var/lib/libvirt/dnsmasq/virbr0.status 
[
  {
    "ip-address": "192.168.122.232",
    "mac-address": "52:54:00:37:68:dc",
    "hostname": "vm1",
    "expiry-time": 1601891907
  }
]
```



### Rocky-Cockpit管理平台

#### 安装cockpit

```bash
[root@localhost ~]# yum install -y cockpit cockpit-machines 
[root@localhost ~]# systemctl start cockpit

# 允许用户登录
[root@localhost ~]# vim /etc/cockpit/disallowed-users
删除root，解除限制root用户登录
[root@localhost ~]# systemctl restart cockpit
```