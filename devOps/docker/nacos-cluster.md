docker pull nacos/nacos-server #拉取nacos镜像
#创建挂载文件路径
mkdir -p /opt/nacos/logs1 /opt/nacos/logs2 /opt/nacos/logs3
mkdir -p /opt/nacos/conf
docker cp 18fa206c4883:/home/nacos/conf /opt/nacos/conf #复制nacos配置目录到宿主机（或从其他地方复制需要application.properties、cluster.conf）
#创建自定义网络
docker network create --driver bridge --subnet 172.18.0.0/16 self_network
#启动mysql容器
docker run -d \
-p 3306:3306  --network self_network --network-alias mysql --ip 172.18.0.02 --name mysql8.0 \
-e MYSQL_ROOT_PASSWORD=Zw123456 \
-v /usr/etc/mysql8.0/mysql/conf:/etc/mysql \
-v /usr/etc/mysql8.0/mysql/logs:/var/log/mysql \
-v /usr/etc/mysql8.0/mysql/data:/var/lib/mysql \
-v /usr/etc/mysql8.0/mysql/mysql-files:/var/lib/mysql-files  mysql
#创建三个nacos容器
docker run -d \
--network self_network --network-alias nacos-server-1 --ip 172.18.0.03 --name nacos-server-1 \
-e PREFER_HOST_MODE=hostname \
-e MODE=cluster \
-e NACOS_SERVER_PORT=8848 \
-e NACOS_SERVERS="172.18.0.03:8848 172.18.0.04:8848 172.18.0.05:8848" \
-e NACOS_SERVER_IP=172.18.0.03 \
-e JVM_XMS=256m -e JVM_XMX=512m  \
-v /opt/nacos/logs1:/home/nacos/logs \
-v /opt/nacos/conf:/home/nacos/conf \
-p 18846:8848 \
nacos/nacos-server

docker run -d \
--network self_network --network-alias nacos-server-2 --ip 172.18.0.04  --name nacos-server-2 \
-e PREFER_HOST_MODE=hostname \
-e MODE=cluster \
-e NACOS_SERVER_PORT=8848 \
-e NACOS_SERVERS="172.18.0.03:8848 172.18.0.04:8848 172.18.0.05:8848" \
-e NACOS_SERVER_IP=172.18.0.04 \
-e JVM_XMS=256m -e JVM_XMX=512m  \
-v /opt/nacos/logs1:/home/nacos/logs \
-v /opt/nacos/conf:/home/nacos/conf \
-p 18847:8848 \
nacos/nacos-server

docker run -d \
--network self_network --network-alias nacos-server-3 --ip 172.18.0.05  --name nacos-server-3 \
-e PREFER_HOST_MODE=hostname \
-e MODE=cluster \
-e NACOS_SERVER_PORT=8848 \
-e NACOS_SERVERS="172.18.0.03:8848 172.18.0.04:8848 172.18.0.05:8848" \
-e NACOS_SERVER_IP=172.18.0.05 \
-e JVM_XMS=256m -e JVM_XMX=512m  \
-v /opt/nacos/logs1:/home/nacos/logs \
-v /opt/nacos/conf:/home/nacos/conf \
-p 18848:8848 \
nacos/nacos-server


docker run -d \
--network self_network --network-alias nginx --ip 172.18.0.06  --name nginx \
-p 7848:8848 \
-p 7777:80 \
nginx 

http{
	upstream nacos-cluster {
		server 172.18.0.03:8848;
		server 172.18.0.04:8848;
		server 172.18.0.05:8848;
	}
	server {
		listen 8848;
		location /{
			proxy_pass http://nacos-cluster;
		}
	}
}


docker run --network self_network --network-alias sentinel --ip 172.18.0.07 --name sentinel  -p 8858:8858 -d bladex/sentinel-dashboard