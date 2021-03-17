Tomcat调优

JVM调优

jvm参数类型

1. 标配参数：-version -help -showversion跟个版本间很稳定 

2. X参数: -Xint 解释执行 -Xcomp第一次使用就编译成本地代码  -Xmixed 混合使用

3. XX参数(重点)：

   > 又可分为：
   >
   > 1. boolean类型：公式 -XX:+或者-某个属性值 表示开启或者关闭，比如是否打印GC收集(-XX:+PrintGCDetails表示开启，-XX:-PrintGCDetails表示关闭);是否设置为串行垃圾收集器(-XX:-UseSerialGC/-XX:+UseSerialGC )。
   >
   > 2. KV类型(设值类型)：-XX:属性key=value，比如：-XX:MetaspacesSize=128m设置元空间，-XX:MaxTenuringThreshold=15(新生代经历过多次GC晋升到老年代)。
   >
   >    



-Xms1024等价于-XX:InitialHeapSize=1024m 

-Xmx1024等价于-XX:MaxHeapSize=1024m

相当于比较常用，JVM给参数起的别名。







**查看jvm初始化默认值**

> java  -XX+PrintFlagsInitial

**查看jvm修改更新项**

> java  -XX+PrintFlagsFinal

=和:=的区别：=表示JVM默认， :=表示根据系统情况修改过的值。



-Xss:线程栈的大小 相当于 -XX:ThreadStackSize 默认为0代表采用系统默认值（1024K）而非默认大小是0。



-XX:SurvivorRatio默认8表示8:1:1

-XX:NewRatio：配置新生代与老年代的比例,默认值2(分母的大小，分子为1，分子表示新生代，分母为+1),新生代占1老年代占2。占比表示 1/(2+1)。

-XX:MaxTenuringThreshold

查看jvm默认值 

jinfo: -XX+PrintFlagsInitial

+PrintCommendLineFlags

+PrintGCDetails



jps 

jinfo

jinfo -flag PrintGCDetails <pid>



jinfo -flag <options> <pid>：查指定选项的值 

jinfo -flags <pid>：查所有选项

jstack





jdk1.8之后元空间会报内存溢出吗(元空间放在本地内存上，为什么会呢)：



mysql怎么实现的事务，分库分表的实现方式。









1)TCP的三次握手和四次挥手过程，为什么是三次握手不是两次或者四次，还有过程中处于的每个状态的名称

三次握手：

1. client(closed->syn-sent) ----SYN=1 seq=x---->  server(closed->listen)。
2.  server(listen->syn-rcvd)----SYN=1 ACK=1 seq=y, ack=x+1----->client(syn-sent)。
3. (client(syn-sent->estab-lished)) --ACK=1 seq=x+1 ack=y+1 server(listen->->estab-lished)

四次挥手：

2）HTTP协议与TCP协议的区别

HTTP是应用层协议，TCP是传输层协议。Http是基于TCP之上的协议。

3）HTTP协议与HTTPS协议的区别

两者都是无状态的协议，Https是一种加密协议。当初次建立链接时，会采用公钥认证方式识别对方身份。身份认证成功后，基于对称加密算法，进行加密通信。

4）一个用户从客户端向服务端发送信息是怎么保持状态信息（大概这个意思吧有点忘了），具体逻辑是怎么设计的。

基于cookie和session。禁用的情况下是根据URL后边携带sessionid参数。

5）会网络编程吗（忘了。。。。）

6）怎么创建线程

继承Thread或者实现runnable。或者是线程池。本质上都是Thread的子类。

7）线程和进程的区别

进程是

8）启动一个应用程序时候有几条进程，从java的角度讲（我也不知道答案是啥）
9）Java怎么连接数据库的，说一下详细步骤
10）JVM的垃圾回收

11）类的加载过程





经典的排序算法，

MySQL锁机制 explain的常用关键字，

I/O流 netty

分布式锁： redis、zk。

spring顶级接口设计

springboot启动流程

序列化的比较

apache  avro ：与编程语言无关的序列化。



###################################################################################################



## kafka

概念：

**broker**：

**集群控制器**：由broker充当，相当于是broker的leader，负责partiion分配给broker和监控broker。一个partion从属于一个broker，该broker是partion leader，可以跟随几个副本broker。

**群组协调器**：由broker充当，针对消费者而言，不同的群组（消费者groupid）会有不同的协调器。负责监控消费者群组内个消费者的状态（独立线程做心跳），如果宕机，会在几秒内进行再均衡。

**群主**：在群组协调器配合下，第一个和群主协调器进行沟通的将成为群主，群主负责从群组协调器里查看所有群成员信息，并分配对应的分区，之后将分配的结果上报给群主协调器，群主协调器再将分配的结果广播给所有的群成员（每个组员都只能看到自己的分配信息）。









消息保留策略：按时间或者按照大小，先得到满足的生效。

集群配置时：

broker.id：正整数 不重复

port：服务端口

zookeeper.connect：zk链接地址，hostname: port/path，path路径作为chroot路径，以区别于其他zk的客户端。

log.dirs：数据文件存放路径

auto. create. topics. enable：是否自动创建主题（1.生产者生产时或者消费者消费时，2.当任意一个客户端向主题发送元数据时）。默认为false。

log. retention. ms：消息保留的时间

log. retention. bytes：消息保留的总大小。

log.segment.ms：消息片段保留多久会被关闭。

log.segment.bytes：消息片段的大小。

borker中数据是以topic分类，以segment为单元进行存储的。可以指定segment的大小和关闭时间。当消息比较少时，消息已过期了，但是当前segment并未存满且segment并未关闭，这就会导致该消息不过期。

message.max.bytes：单个消息的最大字节数。



数据存放位置：

zk：broker、topic、分区等元数据；

>在kafka.0.9之前的版本里，zk还会保存消费者相关信息，比如消费者群组，主题信息，消费分区的偏移量等。

在高版本上，消费者将消费信息提交给broker而不再是zookeeper。

borker中数据是以topic分类，以segment为单元进行存储的。可以指定segment的大小和消息过期时间。当消息比较少时，消息已过期了，但是当前segment并未存满，这就会导致该消息不过期。

消费者

![消费者](./img/kafka_sender.png)





生产消息的要素：

1. 必要（topic）
2. 非必要（分区，key，）

流程：

消息提交模式：（会将消息提交本地队列里，会有专门的单/多线程进行处理）

1. 提交并遗忘，消息提交到队列里即认为成功。
2. 同步发送：提交到队列后，通过Future对象来获取结果信息。
3. 异步发送：提交到队列的同时，会指定一个回调函数处理响应。

配置：

1. ack：0：只要提交成功就算，1：首领节点收到即可 n：表示n个节点收到，all表示所有节点。
2. buffer.memory：生产者本地缓存区大小，取决于当前待发送队列的数据量。
3. compression.type：消息发送前被压缩的方式，默认不压缩。
4. retries：重试次数。
5. client.id：标识客户端。
6. max. in. flight. requests. per. connection：生产者在收到broker响应前可以发送多少个消息，值越大 吞吐量越大，但是重试时可能会影响消息的顺序。设为1时可以保证重试时也能保证顺序。

消息key的两个作用：1.作为消息的补充 2.在分区不变的情况下，相同的key值 总是会被发送到同一分区上。



## 消费者

一个线程只能运行一个消费者、一个消费者也只能运行在一个线程内。如果需要在一个群组内运行多个消费者，必须使用多线程。

订阅类型：订阅类型可以分为全匹配和正则匹配。

正则匹配是指，topic符合订阅的正则表达式，即使是在订阅之后，新创建的的topic也会导致再均衡，导致该消费者新增的topic。

再均衡：

​	再均衡期间，整个消费群组无法进行消费，消费者当前读取状态会消失，甚至可能去刷新缓存(？？？)。

​	触发条件：消费者加入/离开 组群 增加partion 

配置：

bootstrap.server：broker的链接地址

key/value 序列化方式：

group.id：消费者分组id。同一个id隶属于同一个消费群组

轮询：是指在消费者订阅了topic之后，消费者就会处理所有的细节问题（群组协调、再均衡、发送心跳、拉取消息进行消费等）。

**提交偏移量**：消费者在消费完消息后，需要将消息的偏移量offset提交给broker。这个过程其实是消费者向一个特殊的主题_consumer_offset里发送消息，提交当前的offset。

提交策略：默认是自动提交，每过5s（由auto.commit.interval.ms控制）会提交一下offset。

再均衡监听器：再分区之前，消费者会暂停消费，提交offset，清理本地空间等。

消费者除了可以正常订阅主题外，还可以独立消费分区，这种情况下，不再进行再分区。不过当新增分区时，需要重新签订分区才能消费到所有的分区。

**深入**

1. broker以临时节点的状态存储在zk上（/broker/ids）。
2. 第一个连上zk的broker成为控制器（创建/controller），负责分区首领的选举，其他broker启动时，也会尝试创建/controller节点，不过不会成功，但是会创建watch进行监视该节点。当控制器宕机后，其他broker由于有watch监视，会第一时间尝试将自己注册为/controller节点。
3. 当某个broker失联后，控制器通过观察zk路径，就知道哪些分区失去了leader,控制器会从该分区的跟随者副本中选择一个充当分区leader。

首选首领：即分区leader，创建topic时，控制器会依据各broker的负载选取该分区的首领。

请求：无论生产者消费者都必须将请求发送对应分区首领上，才能获得正常的响应，否则，就会得到一个非对应首领的错误，此时客户端会先请求broker（broker会缓存zk上的元数据信息，topic，分区，分区首领等）元数据，通过元数据获取正确的broker，才能正常的发送/拉取消息。

**生产者的acks值只能是0,1，all。**

跟随者副本：是指未跟上首领副本的broker，当数据都同步完成，跟上首领副本时，会转变为同步副本。

同步副本：是指跟上首领副本的broker。

消息必须都在同步副本上都存储之后，该消息才能被消费者甚至和跟随者副本拉取到。（数据未同步前，认为该数据是不安全的，因为有可能此时分区首领宕机）。

kafka是采用**零复制**的机制将数据传递给其他副本或者消费者的。