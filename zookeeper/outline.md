## 常用命令

~~~shell
# 执行zkCli.sh 连接到zookeeper服务器,-server参数可选，指定连接服务器的连接串,默认为本机2181端口
# /path代表指定该客户端的连接路径,起到区分应用的作用,默认为/根路径.
sh zkCli.sh -server 127.0.0.1:2181/path

# 查看节点,只展示节点名称
ls </path>

# 查看节点,展示节点及其详细信息
ls2 </path>

# 创建节点,-s:有序节点 -e:临时节点,无参表示永久节点,data表示节点内容，可以为空串但必须包含该参数
# nodeName节点名称,必须以‘/’打头,有序节点会在节点名称后追加序列号,例如nodeName0000000001.
create [-s|-e] </nodeName> <"data">

# 查看节点数据
get </path>

# 修改数据, data为修改后的内容
set </path> <"data">

# 删除节点
delete </path>

# 设置watcher通知, stat只能监听create和delete事件
stat </path> watch
# 设置watcher通知, get只能监听update和delete事件
get </path> watch

[zk: localhost:2181(CONNECTED) 37] stat /xlock/lock5 watch
Node does not exist: /xlock/lock5
[zk: localhost:2181(CONNECTED) 38] 
WATCHER::

WatchedEvent state:SyncConnected type:NodeCreated path:/xlock/lock5
~~~

