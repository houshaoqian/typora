for port in `seq 7001 7006`; do \
mkdir -p ./${port}/conf \
&& PORT=${port} envsubst < ./redis.conf> ./${port}/conf/redis.conf \
&& mkdir -p ./${port}/data \
&& mkdir -p ./${port}/log \
&& touch ./${port}/log/log; \
done


sudo chmod -R 777 ./

for port in `seq 7001 7006`; do
sudo docker run -d --net host -v /data/docker/redis-cluster/${port}:/redis-cluster --restart always --name=redis-${port}  redis:5.0.2 redis-server /redis-cluster/conf/redis.conf;
done

sudo docker exec -it redis-7001 /bin/bash

redis-cli -a 123456 --cluster create 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 127.0.0.1:7006 --cluster-replicas 1




sudo docker run -d --net=host -v /data/docker/redis-cluster/${port}:/redis-cluster --restart always --name=redis-c${port}  redis redis-server /redis-cluster/conf/redis.conf;



for port in `seq 7001 7006`; do
sudo docker rm -f redis-${port}
done

for port in `seq 7001 7006`; do
sudo docker rm -f redis-${port}
sudo rm -rf ${port}
done


sudo docker run -d --net=host -v /data/docker/redis-cluster/7001:/redis-cluster --restart always --name=redis-c7001  redis redis-server /redis-cluster/conf/redis.conf;


for port in `seq 7002 7006`; do
sudo docker run -d --net=host -v /data/docker/redis-cluster/${port}:/redis-cluster --restart always --name=redis-c${port}  redis redis-server /redis-cluster/conf/redis.conf;
done
