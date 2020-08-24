~~~shell
docker run \
-v \d\data\docker\mysql-signle\etc:/etc/mysql \
-v \d\data\docker\mysql-signle\logs:/var/log/mysql \
-v \d\data\docker\mysql-signle\data:/var/lib/mysql \
-p 3306:3306 --name mysql-signle \
-e MYSQL_ROOT_PASSWORD=123456 \
--restart always \
-d mysql:5.7
~~~

default machine with IP 192.168.99.100