#### JVM加载class原理机制

1. 由三个实现类，分别是启动加载类BootstrapClassLoader、扩展加载类ExtClassLoader、应用加载类AppClassLoader，通过双亲委托机制实现类的加载。
2. 三个类加载器存在父子容器关系(语法继承上无关系)，分别是：AppClassLoader -> ExtClassLoader -> BootstrapClassLoader。
3. 三个类加载器的加载范围 Bootstrap： jre所在的指定的包(例如rt.jar)，Ext： lib/ext下的jar包，App加载用户代码，即Classpath路径下的jar包。
4. 类加载的几个步骤：加载 -> 连接(验证、准备、解析) -> 初始化 -> 使用 -> 卸载。

#### Tomcat类加载顺序

![Tomcat类加载设计图](./img/tomcat-classLoader.png)

1. 在JVM的基础上进行了封装和定制，新增了几个类加载器：Bootstrap、System、Common、webapp。

   > **Bootstrap**类加载器包含了JVM中Bootstrap和Ext类加载器，负责加载标准类库和jre/lib/ext下的jar。
   >
   > **System**负责加载Tomcat的核心类库，比如bootstrap.jar，通常在catalina.bat或者catalina.sh中指定。位于CATALINA_HOME/bin下。
   >
   > **Common**加载tomcat使用以及应用通用的一些类，位于CATALINA_HOME/lib下，比如servlet-api.jar
   >
   > **webapp**加载部署的应用类。

2. 是否违反了双亲委派机制？
   
   > 违反了，webapp类加载器按照双亲委派机制在加载本应用的类时应该委托父加载器进行，实际上却在加载不到时，才会委托父加载器进行加载。也正因此做到了一个容器多应用的相互隔离环境。

#### JVM的运行时数据区

1. JVM内存区域大致可以分为两类堆(Heap)和栈(Stack)。jdk1.8之后新增的元空间(Meta Space)不占用内部空间。

2. 栈：存取速度快。栈中数据是由栈帧压入组成。栈帧：局部变量表、操作数栈、动态链接、方法出口等构成。

3. 堆：堆大致可以分为：新生代、老年代、持久代（1.8已被元空间替代）。

   > 年轻代又可分为：一个Eden区和两个Survivor区构成。新生成的对象分配在此空间。
   >
   > 老年代：经过指定次数的GC后，仍然存活的对象将被移动到老年代。

4. 各版本运行时数据区的区别
   > 1. 各版本的区别主要体现在持久代上的变化。
   > 2. 1.7前， 堆内存中存在方法区，存储运行时常量池，类信息等。使用分代收集方式，利用永久代实现了方法区。
   > 3. 1.7时，将运行时常量池从方法区中移除，单独处理。
   > 4. 1.7后，彻底把方法区删除，利用元空间实现了旧方法区的逻辑。
   

#### JVM垃圾回收

1. 可被回收对象的认定方式

   > 1. 计数法：简单易用，存在互相引用的问题。
   >
   > 2. 可达性分析法：JVM一般使用的算法。
   >
   >    > 在可达性分析算法中，可作为root节点的对象由：
   >    >
   >    > 1. 栈(栈帧中本地变量表)中引用的对象。
   >    > 2. 方法区中**类静态属性**引用的对象
   >    > 3. 方法区中**常量**引用的对象
   >    > 4. 本地方法栈中引用的对象。

2. 常用垃圾回收算法

   > 1. 复制算法：效率高，空间使用率低(多出一块被复制的空间)，适合新生代(存活对象较少的情况)。新生代多采用该算法实现(eden+suv1->suv2 )。
   > 2. 标记-清除：效率低，内存利用率高，STW时间较长，且会产生空间碎片。
   > 3. 标记-整理：效率中等，内存利用率高，无空间碎片和空间利用率问题，常用在老年代的GC中。

3. JVM垃圾回收算法

   > 在上述三种回收算法的基础上，提炼出“分代算法”，即把堆内存中划分为不同的区域，利用不同的算法进行回收。
   >
   > 新生代：据统计，此区域的对象存活率不足2%，因此采用“复制算法”。将新生代分为Eden和两个Survivor
   >
   > 老年代：采用“标记-清除” 或 “标记-整理”算法实现，不同的虚拟机实现方式不同。
   
 #### 常用垃圾回收器

垃圾回收器除了G1外，都只针对新生代或老年代进行回收。

1. Serial：包括Serial New和Serial Old，分别对新生代和老年代进行回收。又叫串行收集器。工作模式为单线程。client模式下默认的收集器。

2. ParNew：Serial的多线程版本，仅针对新生代回收。在单CPU情况下，效率并不会比Serial更高。

3. Parallel Scavenge：包括Parallel Scavenge 和 Parallel Old，分别针对新生代和老年代。多线程，类同ParNew，但是更关注吞吐量(即CPU利用率，区别于CMS的更短响应时间)。

4. CMS：仅针对老年代，采用“标记-清除”算法，以最短停顿时间为目标的收集器。

   > CMS进行老年回收的流程分为：
   >
   > 1. 初始化标记：STW(Stop The World)，仅仅记录GC Root能**直接**关联到的对象，速度很快，但需暂停用户线程。
   >
   > 2. 并发标记：(针对非直接关联GC Root的对象)进行可达性分析，可以伴随着用户线程进行。
   >
   > 3.  重新标记：STW，对并发标记中被修改过的对象进行再标记，确定最终是否GC Root可达。需暂停用户线程。
   >
   > 4. 并发清除：对不可达的对象进行清除工作。
   >
   > 因此，CMS会存在以下问题：
   >
   > 1. 无法处理浮动垃圾，即在并发标记的过程中，用户线程产生的新的垃圾。因此得预留足够多空间给并行用户线程使用，如果不满足并行的用户线程，会有“Ｃoncurrent Mode Fail”错误，从而CMS会利用Serial Old进行一次Full GC。
   >
   > 2. 空间碎片问题。当进行足够多次的GC后，碎片问题会很严重，CMS会进行一次空间压缩式的Full GC，这会导致不能并行的进行回收。
   > 3. 对CPU资源敏感，因为在GC的过程中，会有新的JVM线程进行垃圾回收工作。
   
5. G1：可同时工作于新生代和老年代。以最短停顿时间为目标的收集器。大体上采用“标记-整理”算法。

   使用G1之后的JVM，将整个堆内存划分成很多个大小相等的独立区域(Region)。新生代和老年代不再存在物理上的隔离，而是逻辑上的概念。他们都只是一部分Region(不连续)的集合。G1工作流程大致分为：

  > 1. 初始标记：类同CMS的初始化标记，仅仅记录GC Root能**直接**关联到的对象，速度很快，但需暂停用户线程。
  >
  > 2. 并发标记：类同CMS的初始化标记，(针对非直接关联GC Root的对象)进行可达性分析，可以伴随着用户线程进行。
  >
  > 3. 最终标记：类同CMS的初始化标记，STW，对并发标记中被修改过的对象进行再标记，确定最终是否GC Root可达。需暂停用户线程。
  >
  > 4. 筛选回收：针对各个Region区域的回收价值和成本进行统计，并根据回收计划进行回收。
  >
  > 优点：基于可控的STW时间进行垃圾回收。不会存在空间碎片。
  > 缺点：

#### JVM中的分布式垃圾回收(DGC)

仅存在于RMI(远程调用)子系统中。

> 1)服务端的一个远程对象在3个地方被引用：









