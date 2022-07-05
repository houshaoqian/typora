# 基础

~~~shell
# 修改主机名称
hostnamectl set-hostname centos01
~~~





# 静态IP

1. `vim /etc/sysconfig/network-scripts/ifcfg-eth0`其中eth0代表网卡id，不同机器id不同，配置如下：

~~~properties
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=eth0
UUID=6daef629-4a4f-48ac-8b52-d9622a410031
DEVICE=eth0
ONBOOT=yes
IPADDR=172.21.64.3
NETMASK=255.255.240.0
GATEWAY=172.21.64.1
DNS1=172.21.64.1
~~~

2. 重启网卡 `systemctl restart network`



## 防火墙

~~~shell
# 查看防火墙运行状态
[root@localhost ~]# firewall-cmd --state
running

# 关闭防火墙
[root@localhost ~]# systemctl stop firewalld.service
[root@localhost ~]# firewall-cmd --state
not running

# 禁用防火墙开机启动
[root@localhost ~]# systemctl disable firewalld.service
Removed symlink /etc/systemd/system/multi-user.target.wants/firewalld.service.
Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
~~~



# SELinux

安全增强型 Linux（Security-Enhanced Linux）简称 SELinux，它是一个 Linux 内核模块，也是 Linux 的一个安全子系统。
~~~shell
# 查看selinex状态 Enforcing表示运行中
[root@localhost ~]# getenforce
Enforcing

# 临时关闭selinux
[root@localhost ~]# setenforce 0
Permissive

# 永久关闭selinux
[root@localhost ~]# sed -i 's/^ *SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
~~~



# swap交换

类似windows的虚拟内存。
~~~shell
# 查看帮助
[root@localhost ~]# swapoff -help

# 禁用 /proc/swaps 中的所有交换区
[root@localhost ~]# swapoff -a

# 永久禁用
[root@localhost ~]# sed -i.bak '/swap/s/^/#/' /etc/fstab

# 查看内存使用状况
[root@localhost ~]# free -h
              total        used        free      shared  buff/cache   available
Mem:           7.6G        6.7G        350M        9.0M        573M        683M

~~~



# yum源

一个repo文件通常定义了一个或多个软件仓库地址，centos7的repo文件位于`/etc/yum.repos.d`文件夹下。

>repo文件是Fedora中yum源（软件仓库）的配置文件，通常一个repo文件定义了一个或者多个软件仓库的细节内容，例如我们将从哪里下载需要安装或者升级的软件包，repo文件中的设置内容将被yum读取和应用！
>YUM的工作原理并不复杂，每一个 RPM软件的头（header）里面都会纪录该软件的依赖关系，那么如果可以将该头的内容纪录下来并且进行分析，可以知道每个软件在安装之前需要额外安装 哪些基础软件。也就是说，在服务器上面先以分析工具将所有的RPM档案进行分析，然后将该分析纪录下来，只要在进行安装或升级时先查询该纪录的文件，就可 以知道所有相关联的软件。所以YUM的基本工作流程如下：
>服务器端：在服务器上面存放了所有的RPM软件包，然后以相关的功能去分析每个RPM文件的依赖性关系，将这些数据记录成文件存放在服务器的某特定目录内。
>客户端：如果需要安装某个软件时，先下载服务器上面记录的依赖性关系文件(可通过WWW或FTP方式)，通过对服务器端下载的纪录数据进行分析，然后取得所有相关的软件，一次全部下载下来进行安装。
>
>[摘自]: https://www.cnblogs.com/zhouho/p/13410306.html



## yum命令

yum（yellow dog updater, Modified）, 是一个redhat系Linux系统的前端软件包管理器，作用同Ubuntu下的 apt。

~~~shell
# 语法，其中options可选参数有 -y:安装过程中所有的提示选择yes, -q:不显示安装过程
yum [options] [command] [package...]

# 查看可更新软件清单
yum check-update

# 更新所有软件 当指定了package时，仅更新当前软件
yum update [package]

# 删除软件包命令
yum remove [package]

# 列出所有可安装的软件清单
yum list

# 查找软件包命令
yum search [keyword]

# 清楚缓存 all指 packages和headers
yum clean [all]
~~~



# << EOF用法



# systemctl





[centos7安装k8s](https://blog.csdn.net/qq_37481017/article/details/118897138)



## 问题1

1. 出现宿主机能连接上虚拟机，但是虚拟机却连接不到外网，关闭宿主机的的网络共享（宿主机网络共享给虚拟机的交换机）时，导致宿主机连接不上虚拟机，但虚拟机却可以连接上外网。

   QA:[Hyper-V 网络配置学习 (360doc.com)](http://www.360doc.com/content/22/0417/09/73220646_1026899501.shtml)

2. 
