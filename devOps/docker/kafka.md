## 1.创建zk容器

~~~shell
docker run -d --network self_network --network-alias zookeeper  --ip 172.18.0.09 --name zookeeper -p 2181:2181 wurstmeister/zookeeper
~~~

## 创建kafka服务

1. 创建容器-kafa节点（kafka-server-1）

~~~shell
# PLAINTEXT://192.168.96.1:9092 该ip为宿主机的ip,端口为宿主机映射端口.
docker run  -d \
--network self_network --network-alias zookeeper \
--ip 172.18.0.10 --name kafka-server-1 -p 9092:9092 \
-e KAFKA_BROKER_ID=0 -e KAFKA_ZOOKEEPER_CONNECT=172.18.0.09:2181 \
-e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.96.1:9092 \
-e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 -t \
wurstmeister/kafka
~~~

2. 创建容器-kafa节点（kafka-server-2）
~~~shell
docker run  -d \
--network self_network --network-alias zookeeper \
--ip 172.18.0.11 --name kafka-server-2 -p 9093:9092 \
-e KAFKA_BROKER_ID=1 -e KAFKA_ZOOKEEPER_CONNECT=172.18.0.09:2181 \
-e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.96.1:9093 \
-e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 -t \
wurstmeister/kafka
~~~

3. 创建容器--kafa节点（kafka-server-3）

~~~shell
docker run  -d \
--network self_network --network-alias zookeeper \
--ip 172.18.0.12 --name kafka-server-3 -p 9094:9092 \
-e KAFKA_BROKER_ID=2 -e KAFKA_ZOOKEEPER_CONNECT=172.18.0.09:2181 \
-e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.96.1:9094 \
-e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 -t \
wurstmeister/kafka
~~~



#### 创建TOPIC

~~~shell
# 创建主题topic1
./kafka-topics.sh --create --zookeeper 172.18.0.09:2181 --replication-factor 2 --partitions 2 --topic topic1
# 创建主题topic2
./kafka-topics.sh --create --zookeeper 172.18.0.09:2181 --replication-factor 2 --partitions 2 --topic topic2
~~~

