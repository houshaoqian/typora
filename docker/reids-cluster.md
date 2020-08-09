# Docker 在Ubuntu 18.0.4上安装redis集群

## 前言

今年受到疫情的影响，再加上年前来挂职的上层领导的变更，导致公司环境一落千丈。安逸的日子到头了，和呆了四年的公司就要说拜拜了。应对面试，感觉啥都会又啥都不会，没办法，只能系统的把知识捋一捋，自己动手实践一下了。早就听说docker比较火，加上本地搭建集群模式的困难，就趁此机会在本地用docker练练手了。

## 环境

系统：Ubuntu 18.04.3 LTS

docker版本：19.03.12, build 48a66213fe

redis镜像版本：redis:5.0.2

注意：第一次使用的的redis镜像版本是最新版本redis:latest，在配置集群模式时，卡在了 （Waiting for the cluster to join ），一直以为是防火墙的问题，鼓捣了半天，换到redis:5.0.2就好了。原因还是未知。

## 准备

创建宿主机和docker的映射文件

创建本地文件：/data/docker/redis-cluster，将下载的[redis配置文件](https://raw.githubusercontent.com/redis/redis/5.0/redis.conf)拖到该路径下。

修改配置文件内容（端口号为占位符，通过shell脚本生成各节点映射路径时替换）

~~~shell
# 服务端口
port ${PORT}
# 非本机也可访问
bind 0.0.0.0
# 访问密码
requirepass 123456
# 主从复制密码
masterauth 123456
# 日志文件
logfile /redis-cluster/log/log
# 数据存放路径
dir /redis-cluster/data
~~~

创建各集群节点的映射目录，对应映射关系如下

| /data/docker/redis-cluster/ | docker工作区的主目录                                        |
| :-------------------------- | ----------------------------------------------------------- |
| -redis.conf                 | redis的配置文件模板                                         |
| -/7001                      | redis集群节点7001对应的目录，映射到容器的/redis-cluster目录 |
| -/data/                     | 节点7001对应的数据存放路径                                  |
| -/conf/redis.conf           | 节点7001对应启动的配置文件，依据模板生成                    |
| -/log/log                   | 节点7001对应的日志文件                                      |
| -/7002/...                  | redis集群节点7002对应的目录，子目录与上述相同               |
| -/7003/...                  | ...                                                         |
| -/7004/...                  | ...                                                         |
| -/7005/...                  | ...                                                         |
| -/7006/...                  | ...                                                         |

注：只用手动生成根目录/data/docker/redis-cluster/和redis配置模板文件即可，其他的可通过脚本动态生成



## 启动容器

通过同一镜像redis:5.0.2启动6个容器（3主3从），端口分别为7001，7002，7003，7004，7005，7006。

基本命令如下：

~~~shell
sudo docker run -d --net host -v /data/docker/redis-cluster/7001:/redis-cluster --restart always --name=redis-7001  redis:5.0.2 redis-server /redis-cluster/conf/redis.conf;
~~~

-d：以后台服务运行

--net host：docker的网络模式，公用主机网络

-v：对宿主机和容器的路径进行映射

--restart always：docker重启时，对该容器进行启动。防止而每次启动时，手动重启。



##  集群环境配置

熟悉redis集群的同学都知道，集群节点启动时，各节点还不是集群关系，需要进行手动（或者通过ruby脚本）配置。

~~~she
# 指定集群为6台机器，每各个主节点保持一个副节点。
redis-cli -a 123456 --cluster create 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 127.0.0.1:7006 --cluster-replicas 1
~~~



## 查看集群信息

集群启动后，查看集群是否正常，是否覆盖了所有的槽点（redis总共为16384个槽点，因此集群最大为16384）。

~~~shell
# 进入一个节点
sudo docker exec -it redis-7001 /bin/bash
# 查看集群状态
127.0.0.1:7001> cluster nodes
dcc0d03c2318167f0cf47a830b5fe122d141e7c3 127.0.0.1:7005@17005 slave 71889bb10cefe1d7897d2b79b8404c940498dd2d 0 1596986900886 5 connected
a3b98c3dc1ca8b141ce18a3b90599cef828b7127 127.0.0.1:7003@17003 master - 0 1596986900000 3 connected 10923-16383
71889bb10cefe1d7897d2b79b8404c940498dd2d 127.0.0.1:7001@17001 myself,master - 0 1596986901000 1 connected 0-5460
2b11ab9a7c7e0bf61c61a21e1ea7cac30beceb0a 127.0.0.1:7006@17006 slave fbabdc9ab7aec3b5cc2abab9765444b07f8d2718 0 1596986899000 6 connected
fbabdc9ab7aec3b5cc2abab9765444b07f8d2718 127.0.0.1:7002@17002 master - 0 1596986899000 2 connected 5461-10922
449c60f884e0dad9b512ca5819513cb7fe354bb8 127.0.0.1:7004@17004 slave a3b98c3dc1ca8b141ce18a3b90599cef828b7127 0 1596986901888 4 connected
~~~

从打印结果可以看出，集群完美的覆盖了所有的槽点。

至此docker上安装redis集群结束。



## 安装过程所有命令

安装时，用到的命令：

~~~shell
# 创建主工作区目录 & 下载配置文件 & 修改配置文件内容
(base) sheart@sheart-desktop:/data/docker$ sudo mkdir redis-cluster
(base) sheart@sheart-desktop:/data/docker$ sudo chmod -R 777 /data/docker/redis-cluster
(base) sheart@sheart-desktop:/data/docker$ cd /data/docker/redis-cluster
(base) sheart@sheart-desktop:/data/docker/redis-cluster$ wget https://raw.githubusercontent.com/antirez/redis/5.0/redis.conf
# 利用脚本创建各节点映射目录，该脚本需要当前路径在主工作区目录/data/docker/redis-cluster下执行
for port in `seq 7001 7006`; do \
mkdir -p ./${port}/conf \
&& PORT=${port} envsubst < ./redis.conf> ./${port}/conf/redis.conf \
&& mkdir -p ./${port}/data \
&& mkdir -p ./${port}/log \
&& touch ./${port}/log/log; \
done
# 防止节点启动时，权限不足，提前赋权限
(base) sheart@sheart-desktop:/data/docker/redis-cluster$ sudo chmod -R 777 /data/docker/redis-cluster

# 通过一下脚本，启动各个节点
for port in `seq 7001 7006`; do
sudo docker run -d --net host -v /data/docker/redis-cluster/${port}:/redis-cluster --restart always --name=redis-${port}  redis:5.0.2 redis-server /redis-cluster/conf/redis.conf;
done

# 进入其中一个节点
(base) sheart@sheart-desktop:/data/docker/redis-cluster$ sudo docker exec -it redis-7001 /bin/bash

# 配置节点
root@sheart-desktop:/data# redis-cli -a 123456 --cluster create 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 127.0.0.1:7006 --cluster-replicas 1
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 127.0.0.1:7004 to 127.0.0.1:7001
Adding replica 127.0.0.1:7005 to 127.0.0.1:7002
Adding replica 127.0.0.1:7006 to 127.0.0.1:7003
>>> Trying to optimize slaves allocation for anti-affinity
[WARNING] Some slaves are in the same host as their master
M: 9fcdcdfb347b510f3bab0e502f0e4fe029ce18a8 127.0.0.1:7001
   slots:[0-5460] (5461 slots) master
M: 53d163e9a262a49d96bcd906c63c68552766ee94 127.0.0.1:7002
   slots:[5461-10922] (5462 slots) master
M: 28af284c5b1f46f72d57da862741583f3cc6bb9b 127.0.0.1:7003
   slots:[10923-16383] (5461 slots) master
S: f7eebb12a822791419d7a23f64035ba3808ca2cb 127.0.0.1:7004
   replicates 9fcdcdfb347b510f3bab0e502f0e4fe029ce18a8
S: 80768d176962e7a76fbdf7da2649677d7df0efa9 127.0.0.1:7005
   replicates 53d163e9a262a49d96bcd906c63c68552766ee94
S: 3f3999a56842d96fa6f59d7e6fbb49472136de78 127.0.0.1:7006
   replicates 28af284c5b1f46f72d57da862741583f3cc6bb9b
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
....
>>> Performing Cluster Check (using node 127.0.0.1:7001)
M: 9fcdcdfb347b510f3bab0e502f0e4fe029ce18a8 127.0.0.1:7001
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: 3f3999a56842d96fa6f59d7e6fbb49472136de78 127.0.0.1:7006
   slots: (0 slots) slave
   replicates 28af284c5b1f46f72d57da862741583f3cc6bb9b
S: 80768d176962e7a76fbdf7da2649677d7df0efa9 127.0.0.1:7005
   slots: (0 slots) slave
   replicates 53d163e9a262a49d96bcd906c63c68552766ee94
M: 53d163e9a262a49d96bcd906c63c68552766ee94 127.0.0.1:7002
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
M: 28af284c5b1f46f72d57da862741583f3cc6bb9b 127.0.0.1:7003
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: f7eebb12a822791419d7a23f64035ba3808ca2cb 127.0.0.1:7004
   slots: (0 slots) slave
   replicates 9fcdcdfb347b510f3bab0e502f0e4fe029ce18a8
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.

# 查看redis集群状态
root@sheart-desktop:/data# redis-cli -a 123456 -h 127.0.0.1 -p 7001
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
127.0.0.1:7001> cluster nodes
3f3999a56842d96fa6f59d7e6fbb49472136de78 127.0.0.1:7006@17006 slave 28af284c5b1f46f72d57da862741583f3cc6bb9b 0 1596988842479 6 connected
80768d176962e7a76fbdf7da2649677d7df0efa9 127.0.0.1:7005@17005 slave 53d163e9a262a49d96bcd906c63c68552766ee94 0 1596988840000 5 connected
9fcdcdfb347b510f3bab0e502f0e4fe029ce18a8 127.0.0.1:7001@17001 myself,master - 0 1596988841000 1 connected 0-5460
53d163e9a262a49d96bcd906c63c68552766ee94 127.0.0.1:7002@17002 master - 0 1596988840473 2 connected 5461-10922
28af284c5b1f46f72d57da862741583f3cc6bb9b 127.0.0.1:7003@17003 master - 0 1596988840000 3 connected 10923-16383
f7eebb12a822791419d7a23f64035ba3808ca2cb 127.0.0.1:7004@17004 slave 9fcdcdfb347b510f3bab0e502f0e4fe029ce18a8 0 1596988842000 4 connected
~~~

其他命令：

~~~sh
# 批量删除容器
for port in `seq 7001 7006`; do
sudo docker rm -f redis-${port}
sudo rm -rf ${port}
done


~~~

