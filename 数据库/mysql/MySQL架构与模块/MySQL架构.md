# MySQL架构

## Server层

Server层包括连接器、缓存管理器、解析器、优化器、执行器等多个模块，无论MySQL采用何种存储引擎，这部分内容是一样的，与存储引擎层相互隔离。Server层和存储引擎层通过接口定义解耦。正是因为MySQL的这种解耦的架构模式，使得每个人都可以开发自己的存储引擎，也因此，MySQL 能满足不同的业务需求，从而成就了MySQL今天的地位。

### 链接器

客户端连接服务端有很多种方式，可以是长连接、短链接、同步链接、异步链接等。协议类型有TCP、Unix Socket（同一台主机中，多个进程之间）、Windows下的命名管道和内存共享等。连接器主要负责相应客户段数据请求，权限认证等。

1. 每产生一个链接在服务端都会新建一个线程来处理。

   既然是新建线程，那必然会消耗服务器资源，因此最大链接数并不是越大越好，视情况而定。在MySQL5.7版本中，默认最大链接数为151，可以设的最大值为100000。

2. 参数分为两个级别，全局（global）和会话(session)

   全局级别的参数，设置后，新建会话和当前会话都会受到影响（已存在的会话不会受到影响）。会话级别的参数只在当前窗口生效。

3. 

~~~mysql
# 查看当前有多少个链接
mysql> show global status like '%Thread%';
+------------------------------------------+-------+
| Variable_name                            | Value |
+------------------------------------------+-------+
| Delayed_insert_threads                   | 0     |
| Performance_schema_thread_classes_lost   | 0     |
| Performance_schema_thread_instances_lost | 0     |
| Slow_launch_threads                      | 0     |
| Threads_cached                           | 0     |
| Threads_connected                        | 1     |
| Threads_created                          | 1     |
| Threads_running                          | 1     |
+------------------------------------------+-------+
8 rows in set (0.00 sec)

# 查看当前最大链接数
mysql> show variables like 'max_connections';
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| max_connections | 151   |
+-----------------+-------+
1 row in set (0.00 sec)

# MySQL查看非活动链接超时时间，默认8小时-非交互式（JDBC等）
mysql> show global variables like 'wait_timeout';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| wait_timeout  | 28800 |
+---------------+-------+
1 row in set (0.00 sec)

# MySQL查看非活动链接超时时间，默认8小时-交互式（数据库工具等）
mysql> show global variables like 'interactive_timeout';
+---------------------+-------+
| Variable_name       | Value |
+---------------------+-------+
| interactive_timeout | 28800 |
+---------------------+-------+
1 row in set (0.00 sec)
~~~



### 缓存管理器

缓存管理器是缓存查询结果的模块，由于其结果受很多因素影响，比如查询语句完全匹配和两次查询之间不能有数据修改和新增等，导致缓存命中率很低。实际应用中，缓存一般放在了ORM框架（Hibernate/Mybatis等）中或者直接使用Redis等内存缓存服务更加便捷和高效。因此MySQL5.7中默认关闭了缓存功能，MySQL8.0中直接移除了该功能。

~~~mysql
# MySQL5.7中查看缓存是否一开启
mysql> show variables like 'query_cache%';
+------------------------------+---------+
| Variable_name                | Value   |
+------------------------------+---------+
| query_cache_limit            | 1048576 |
| query_cache_min_res_unit     | 4096    |
| query_cache_size             | 1048576 |
| query_cache_type             | OFF     |
| query_cache_wlock_invalidate | OFF     |
+------------------------------+---------+
5 rows in set (0.00 sec)
~~~

### 语法解析器

词法分析、语法分析，生成词法树。

###   优化器

生成执行计划，索引选择

### 执行器

调用存储引擎，返回操作结果

## 存储引擎层

