### 设置静态ip

1. 配置网卡

~~~prope
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
NAME=eth0
UUID=0784a96e-55ef-414f-9d52-03ab55fc6e83
DEVICE=eth0
ONBOOT=yes
IPADDR=192.168.0.200
NETMASK=255.255.255.0
GATEWAY=192.168.0.1
DNS1=8.8.8.8
PREFIX=16
~~~

2. 重启网卡

~~~shell
nmcli c reload
nmcli c up eth0
~~~

