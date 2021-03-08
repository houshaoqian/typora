#### JVM加载class原理机制

1. 由三个实现类，分别是启动加载类BootstrapClassLoader、扩展加载类ExtClassLoader、应用加载类AppClassLoader，通过双亲委托机制实现类的加载。
2. 三个类加载器存在父子容器关系(语法继承上无关系)，分别是：AppClassLoader -> ExtClassLoader -> BootstrapClassLoader。
3. 三个类加载器的加载范围 Bootstrap： jre所在的指定的包(例如rt.jar)，Ext： lib/ext下的jar包，App加载用户代码，即Classpath路径下的jar包。
4. 类加载的几个步骤：加载 -> 连接(验证、准备、解析) -> 初始化 -> 使用 -> 卸载。