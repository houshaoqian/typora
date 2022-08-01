MySQL主备机：

整体环境

| docker容器名称 | ip          | 备注 |
| -------------- | ----------- | ---- |
| mysql-maste    | 172.18.0.11 | 主机 |
| mysql-slave    | 172.18.0.12 | 备机 |



#### 创建Docker自定义网络

~~~shell
# 创建自定义网络, my_network_u0可自定义
sudo docker network create --driver bridge --subnet 172.18.0.0/16 my_network_u0
~~~

#### MySql主机

##### 1. 创建mysql-master容器

~~~shell
# 基于自定义网络my_network_u0 创建容器 并指定ip
sudo docker run -p 3306:3306 --name mysql-master --network my_network_u0 --network-alias mysql-master-11 --ip 172.18.0.11 -e MYSQL_ROOT_PASSWORD=123456 -d mysql
~~~

##### 2.修改MySQL配置

~~~properties
[mysqld]
## 设置server_id，一般设置为IP，同一局域网内注意要唯一
server_id=100
## 复制过滤：也就是指定哪个数据库不用同步（mysql库一般不同步）
binlog-ignore-db=mysql
## 开启二进制日志功能，可以随便取，最好有含义（关键就是这里了）
log-bin=mysql-bin
## 为每个session 分配的内存，在事务过程中用来存储二进制日志的缓存
binlog_cache_size=1M
## 主从复制的格式（mixed,statement,row，默认格式是statement）
binlog_format=mixed
## 二进制日志自动删除/过期的天数。默认值为0，表示不自动删除。
expire_logs_days=7
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
slave_skip_errors=1062
~~~

#### MySql备机

##### 1.创建mysql-slave容器

~~~shell
# 基于自定义网络my_network_u0 创建容器 并指定ip
sudo docker run -p 3307:3306 --name mysql-slave --network my_network_u0 --network-alias mysql-salve-12 --ip 172.18.0.12 -e MYSQL_ROOT_PASSWORD=123456 -d mysql
~~~

##### 2.修改MySQL配置

~~~properties
[mysqld]
## 设置server_id，一般设置为IP,注意要唯一
server_id=101
## 复制过滤：也就是指定哪个数据库不用同步（mysql库一般不同步）
binlog-ignore-db=mysql
## 开启二进制日志功能，以备Slave作为其它Slave的Master时使用,指定binlog文件名称
log-bin=edu-mysql-slave1-bin
## 为每个session 分配的内存，在事务过程中用来存储二进制日志的缓存
binlog_cache_size=1M
## 主从复制的格式（mixed,statement,row，默认格式是statement）
binlog_format=mixed
## 二进制日志自动删除/过期的天数。默认值为0，表示不自动删除。
expire_logs_days=7
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
slave_skip_errors=1062
## relay_log配置中继日志
relay_log=edu-mysql-relay-bin
## log_slave_updates表示slave将复制事件写进自己的二进制日志
log_slave_updates=1
## 防止改变数据(除了特殊的线程)
read_only=1
~~~

#### 建立主备关系

1. mysql-master上查看当前binlog状态

~~~shell
# 查看当前master位置
mysql> show master status;
+----------------------+----------+--------------+------------------+-------------------+
| File                 | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+----------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001     |     2687 |              | mysql            |                   |
+----------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
~~~

2. 建立主备关系，在从机上执行以下命令

~~~mysql
mysql> change master to master_host='172.18.0.11', master_user='root', master_password='123456', master_port=3306, master_log_file='edu-mysql-bin.000001', master_log_pos=1135, master_connect_retry=30;
~~~

master_host: Master 的IP地址
master_user: 在 Master 中授权的用于数据同步的用户
master_password: 同步数据的用户的密码
master_port: Master 的数据库的端口号
master_log_file: 指定 Slave 从哪个日志文件开始复制数据，即上文中提到的 File 字段的值
master_log_pos: 从哪个 Position 开始读，即上文中提到的 Position 字段的值
master_connect_retry: 当重新建立主从连接时，如果连接失败，重试的时间间隔，单位是秒，默认是60秒。



3. 开始备份

~~~mysql
mysql> start slave;
~~~

4. 查看备机状态

~~~mysql
mysql> show slave status \G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for source to send event
                  Master_Host: 172.18.0.11
                  Master_User: root
                  Master_Port: 3306
                Connect_Retry: 30
              Master_Log_File: edu-mysql-bin.000001
          Read_Master_Log_Pos: 2687
               Relay_Log_File: edu-mysql-relay-bin.000002
                Relay_Log_Pos: 1880
        Relay_Master_Log_File: edu-mysql-bin.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
    ......
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 2687
              Relay_Log_Space: 2093
              Until_Condition: None
	......
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 11
                  Master_UUID: 33971314-0bfe-11ec-9a3b-0242ac12000b
             Master_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Replica has read all relay log; waiting for more updates
           Master_Retry_Count: 86400

1 row in set, 1 warning (0.00 sec)

ERROR: 
No query specified
~~~

Tips：

1. 重点检查Slave_IO_Running状态是否是Yes，Yes代表成功。
2. 如果不成功，Last_IO_Error:代表错误信息。

#### 常用MySQL命令

~~~mysql 
# 展示主机的binlog情况
mysql> show master status;
# 展示当前备机同步binlog状态，\G：表示格式化输出 
mysql> show slave status \G;
# 在备机上执行，表示开始同步数据
mysql> start slave;
# 在备机上执行，表示停止同步数据
mysql> stop slave;
# 在备机上执行，表示重置备机同步数据，重新备份数据时，需要重新执行主机，binlog位置等信息。
# 当备机数据丢失，或者需要重新指定备机时，可以先全量恢复数据后重新开始备份
mysql> reset slave;
~~~

