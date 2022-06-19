# Spring

## Spring模块

### 分类

根据功能不同，Spring模块可以划分为几大类模块：

1. [核心容器](./modules/spring-core.md)：
   
   包括spring-beans、spring-core、spring-context，spring-expression（Spring Expression Language，SpEL 表达式）,
   
   spring-context-support、spring-context-indexer 等6个模块组成。
   
2. [AOP模块](./modules/spring-aop.md)：

   包括spring-aop、spring-aspects和spring-instrument 3个模块构成。因为核心模块依赖AOP实现，因此严格来说，AOP模块也属于核心模块。

3. [数据访问与集成](./modules/spring-data.md)：

   包括spring-jdbc、spring-tx、spring-orm、spring-oxm和spring-jms 5个模块组成。

4. [web组件](./modules/spring-web.md)：

   包括spring-web、spring-webmvc、spring-websocket和spring-webflux 4个模块组成。

5. 通信报文：

   仅包含spring-messaging模块，它是spring4增加的一个模块，主要职责为Spring框架集成一些基础的报文传送应用。

6. [集成测试](./modules/spring-test.md)：

   单指spring-test模块，主要为单元测试提供支持。

7. 集成兼容：

   单指spring-framework-bom模块，主要解决Spring的不同模块依赖版本不同的问题。

### 依赖关系

 ![Spring模块依赖图](./images/Spring-moduls-dependency.png)



## Spring核心启动流程

