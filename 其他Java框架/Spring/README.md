# Spring框架



## 软件架构设计原则

1. 开闭原则

   >即，对修改关闭，对扩展放开。用抽象构建框架，用具体实现扩展细节。在Spring中AbstractApplicationContext#refresh()容器启动流程中，流程顺序已确定，但是各子流程由其不同的子类实现，实现功能的扩展。例如：loadBeanDefinitions()加载BeanDefinition，实现类XmlWebApplicationContext中通过XML配置文件加载，而AnnotationConfigWebApplicationContext则是通过注解实现。

2. 依赖倒置原则
   
   >指高层模块不应该依赖底层模块的具体实现，而是依赖其抽象。在Spring中的体现通常是Spring的各顶级	
   
3. 单一职责原则   

   > 在程序设计时，一个模块/接口/类应该保持单一的功能。因为功能越多，涉及到调用的地方也就越多，逻辑被修改的可能性也就越大，复杂度也随之增大。

4. 接口隔离原则

   > 类似单一职责原则，在程序设计时，接口的依赖，方法入参类型等都应该建立在最小接口之上。该原则主要保证了程序的高内聚低耦合的思想。
   
7. 合成复用原则

   > 是指尽量使用对象组合/聚合而不是继承关系得到软件复用的目的。可以使系统更加灵活，降低类与类之间的耦合度。例如在AbstractApplicationContext是BeanFactory的子类，但在实现方式上是通过内部成员变量beanFactory来实现其功能的。
