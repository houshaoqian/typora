## 创建自定义网络
~~~shell
docker network create --driver bridge --subnet 172.18.0.0/16 self_network
~~~



| ip(已分配)  | docker容器名称 | 备注              | 主机端口映射 |
| ----------- | -------------- | ----------------- | ------------ |
| 172.18.0.02 | mysql8.0       | 单机版MySQL服务   | 3306-3306    |
| 172.18.0.03 | nacos-server-1 | Nacos集群-节点1   | 18846-8848   |
| 172.18.0.04 | nacos-server-2 | Nacos集群-节点2   | 18847-8848   |
| 172.18.0.05 | nacos-server-3 | Nacos集群-节点3   | 18848-8848   |
| 172.18.0.06 | nginx          | Nginx单机服务     | 78848-8848;  |
| 172.18.0.07 | sentinel       | sentinel单机服务  |              |
| 172.18.0.08 | seata          | seata单机服务     |              |
| 172.18.0.09 | zookeeper      | zookeeper单机服务 | 2181-2181    |
| 172.18.0.10 | kafka-server-1 | kafka集群节点     | 9092-9092    |
| 172.18.0.11 | kafka-server-2 | kafka集群节点     | 9093-9092    |
| 172.18.0.12 | kafka-server-3 | kafka集群节点     | 9094-9092    |
| 172.18.0.13 | redis          | redis单节点       |              |

