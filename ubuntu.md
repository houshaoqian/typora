

~~~shell
								# 查看已安装的软件
dpkg -l #打印所有
dpkg -l | grep <name> #过滤name

# 卸载已安装的程序
dpkg -P <name>#删除安装文件
dpkg -P --purge <name> #删除安装文件且删除配置文件

# apt-get 方式卸载
apt-get remove  --purge <name>

# 将服务设置为开机启动
systemctl enable <serviceId>

# 修改主机名称
sudo vi /etc/hostname

# 查看端口被占用的进程
lsof -i:<port>
~~~



### 设置静态IP

1. 找到配置文件,name为动态值

~~~shell
sudo vi /etc/netplan/${name}-cloud-init.yaml
~~~

2. 修改配置文件内容为
~~~yaml
network:
    ethernets:
        <eth0>:
            dhcp4: false
            addresses: [192.168.0.201/16]
            optional: true
            gateway4: 192.168.0.1
            nameservers:
                    addresses: [223.5.5.5,223.6.6.6]
 
    version: 2
~~~

3. 设置配置生效

~~~shell
sudo netplan apply
~~~



### 安装工作keepalived（ubuntu server 20）

~~~shell
# 1.准备环境(依赖的包)
	# c语言编译器
sudo apt install gcc
	# ssh依赖的包, openssl & libssl-dev, 不同的linux包名不同
sudo apt-get install openssl
sudo apt-get install libssl-dev
	# 负责解析编译Makefile文件
sudo apt install make
# 2.下载keepalived安装包到指定位置
sudo wget https://www.keepalived.org/software/keepalived-2.0.18.tar.gz
# 解压安装包
sudo tar -zxvf keepalived-2.0.18.tar.gz

#进入到软件包目录内
cd keepalived-2.0.18/
# 编译 & 安装 keepalived, --prefix指定安装的位置, --sysconf指定配置文件的位置
sudo ./configure --prefix=/usr/local/keepalived --sysconf=/etc
sudo make && sudo make install

# 将keepalived注册为系统服务
sudo cp keepalived/etc/init.d/keepalived /etc/init.d/
sudo cp keepalived/etc/sysconfig/keepalived /etc/init.d/

# keepalived常用操作
	# 启动
systemctl start keepalived.service
	# 停止
systemctl stop keepalived.service
	# 重启
systemctl restart keepalived.service
~~~



~~~conf
! Configuration File for keepalived

global_defs {
   # 全局唯一
   router_id LVS_201
   
}

# 计算机节点
vrrp_instance VI_1 {
	# 表示当前节点的状态MASTER/BACKUP,MASTER节点代表主机，可以都为MASTER,表示都接受请求,BACKUP表示备机,只有当master挂了才会变为MASTER，
    state MASTER
    # 当前实列绑定的网卡
	interface eth0
    # 表示路由id,同一集群保持唯一
	virtual_router_id 51
    # 优先级,仅针对BACKUP选取MASTER时使用
	priority 100
    # 主备之间同步检查时间间隔,单位秒
	advert_int 1
	#认证授权密码, 防止非法节点的进入
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.0.244
    }
}

}
~~~

### keepalived的使用场景

1. 主备高可用：

   原理：通过虚拟一个IP，一台主机Master负责处理请求，一台备机待命。

   优点：高可用，MASTER宕机后，备机负责使用

   缺点：浪费资源，备机资源浪费。

   

2. 双主热备：

   原理：虚拟出两个ip，每个ip由不同的主机充当MASTER，两个ip承载着相同的服务，两个ip有DNS轮询实现负载均衡。

   优点：资料利用率高，能够承载更高的并发量。

   缺点：负载均衡机制依赖DNS进行ip轮询，未真正意义上实现负载均衡。

### LVS + DR

~~~shell
# 关闭网络
sudo ifconfig down
# 开启网路
sudo ifconfig up

~~~
