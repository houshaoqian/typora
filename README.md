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







