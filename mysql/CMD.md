## MySQL５.７

1. 初始化root密码

   ~~~shell
   # 方法1
   mysql>update mysql.user set authentication_string=password("新密码");
   mysql>flush privileges;
   
   # 方法2 该方法支持root远程登录
   mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY '新密码';  
   mysql> flush privileges;
   ~~~

   

**show variables like 'max_connections'** 查看数据库默认链接数 default=151

**show variables liek 'datadir'** 查看数据文件存放位置

mysql参数级别

session级别

global级别

配置文件

mysql的插件式存储引擎（memory/archive/innoDB）





## mysql 执行流程

mysql的语句类型只有两类：
1.doQuery：
2.doUpdate：

存储引擎 会进行预读（读页）默认是16K，与操作系统的页不同（默认是4K）,概念相同，逻辑概念。

随机I/O
顺序I/O

innoDB储存引擎
read log -> buffer poop 持久化数据
undo log -> 撤销事务  回滚

Server
binlog Server层的DML操作日志
作用：主从复制，数据恢复



离散度

回表

覆盖索引



二叉查找树

平衡二叉树B-Tree

加强平衡二叉树B+Tree

索引的字段：where join order by




