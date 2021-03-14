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





