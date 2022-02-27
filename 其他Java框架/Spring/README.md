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



## Spring中常用的设计模式

1. 工厂模式

   > 应用场景：
   >
   > ​	避免相同类型的对象在不同的地方使用时，实例化和初始化的代码的冗余。相当于是抽取公共部分。
   >
   > 可细分为：**简单工厂模式**、**工厂方法模式**、**抽象工厂模式**。
   >
   > 简单工厂模式：简单的代码抽取，实现同一个类型对象的创建。
   >
   > 工厂方法模式：定义一个创建对象的接口，让实现这个接口的类来决定实例化哪个具体的类。可以实现不同类型对象的创建，比如sl4j的LoggerFactory#getLogger()方法。
   >
   > 抽象工厂模式：是指工厂类定义了同时提供多个类型的创建方法。这些被实例化的类型往往具有相关性。

2. 单例模式

   > ​	类在整个运行期间只会产生一个对象，相关写法有很多，其可以分为两个类型和多个安全等级。类型是**指延迟实例化**和**非延迟实例化**（也被称为饿汉模式，是指无论对象是否被用到都会实例化）。安全等级是指**线程安全**、**反射安全**、**序列化安全**。线程安全的提升可以通过加锁实现，反射是否安全通过在构造函数中添加相关判断实现，序列化安全可以通过增加readResolve()方法，通过该方法返回已存在的单例对象即可。另外Cloneable类型也会破坏单例模式，但既然是单例了，程序在设计时也不会去实现Cloenable接口。
   >
   > 代码如下：XXX
   >
   > ~~~java
   > 
   > ~~~
   >
   > 

3. 原型模式

   > ​	典型的应用是当一个实体类中有大量属性需要set时使用原型模式。在Spring中的scop='prototype'就是该模式的应用。
   >
   > 该模式主要在实体类中添加clone()方法来实现对象的复制，因此就涉及到浅复制和深复制。

4. 代理模式

   > ​	毋庸置疑，Spring中使用最多的设计模式，AOP的基础。其定义很简单，为其他对象提供代理功能，以实现对这个对象访问。代理模式属于机构型设计模式，主要目的有两个，一是保护目标对象，二是增强目标对象，比如AOP。代理模式可以分为静态代理和动态代理。
   >
   > 静态代理：
   >
   > ​	静态代理是指硬编码的代理方式，通常代理对象和被代理对象是一一对应的关系。
   >
   > 动态代理：
   >
   > ​	动态代理是指在程序运行时，确定代理对象与被代理对象的关系，实际中往往一个代理对象对应了n多个被代理对象。在Spring中实现方式有两种，JDK动态代理和Cglib动态代理，默认使用JDK动态代理。
   >
   > 1. JDK动态代理，要求是，被代理对象必须至少实现了一个接口。代理类是通过实现相同的接口来实现代理服务。
   >
   > 2. Cglib动态代理，要求是，对被代理对象无要求，代理类是通过继承被代理对象来实现代理服务。
   >
   > 3. 两种代理方式都会在运行期间生成字节码，JDK动态代理会直接写Class字节码，Cglib使用ASM框架写字节码，**仅针对写字节码而言**，前者效率更高，后者更为复杂。
   >
   > 4. JDK动态代理是通过**反射**实现对原方法的调用，而Cglib动态代理是通过FastClass机制，直接实现对原方法的调用，就执行效率而言，Cglib更高。
   >
   >    
   >
   > JDK动态代理，代码如下：XXX
   >
   > ~~~java
   > 
   > ~~~
   >
   > Cglib动态代理，代码如下：XXX
   >
   > ~~~java
   > 
   > ~~~
   >
   > 

5. 委派模式

   > 不属于23中设计模式之一，但在Spring中也很常用，个人理解为就是功能的分派，例如SpringMVC中的DipatcherServlet集中接受所有的Http请求，然后依据不同URL指派给不同的业务Controller。

6. 策略模式

   > ​	定义了算法家族并封装起来，让他们之间可以相互替换，此模式是用来将算法和场景解耦。使得算法的变更不会影响到业务流程的变更。在日常最多使用的场景就是解决if-else分支。
   >
   > 1. 策略模式符合开闭原则。
   > 2. 可以避免多重if-else分支。
   > 3. 可以提高算法的保密性和安全性。
   > 4. 单独的算法修改不会影响到业务流程代码的变更，更安全。

7. 模板模式

   > 又叫模板方法模式，是指定义一个算法的骨架，并允许子类为一个或多个步骤提供实现。模板模式使得子类可以在不改变算法结构的前提下，重新定义某些步骤，属于行为设计模式。此模式常用在 流程中包含多个子流程，且子流程顺序确定，但部分子流程实现有所不同的业务中。在Spring中，BeanFactory的启动流程就属于典型的魔板模式。
   >
   > ~~~java
   > 	@Override
   > 	public void refresh() throws BeansException, IllegalStateException {
   > 		synchronized (this.startupShutdownMonitor) {
   > 			// Prepare this context for refreshing.
   > 			// 设置容器状态,初始化变量(系统变量和JVM环境变量)等.
   > 			// 1.设置容器 启动时间、启动状态;
   > 			// 2.初始化系统环境变量servletContextInitParams和servletConfigInitParams.
   > 			// 3.校验系统变量是否完整(默认无必须的参数).
   > 			// 4.设置应用早期事件(earlyApplicationEvents,默认空)和早期事件监听器(earlyApplicationListeners,默认无).
   > 			MyLogger.log("5.12WebApplicationContxt启动准备工作(设置启动时间,状态等)");
   > 			prepareRefresh();
   > 
   > 			// 通过抽象方法,让子类去创建BeanFactory并加载BeanDefinition
   > 			MyLogger.log("5.13WebApplicationContext创建BeanFactory,并解析配置文件以加载所有的BeanDefinition");
   > 			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
   > 
   > 			MyLogger.log("5.14配置BeanFactory(BeanPostProcessor,environment等核心组件)");
   > 			// 配置以下类型的内容：
   > 			// 1.指定ClassLoader、指定Spel表达式解析类、指定属性编辑器(Properties变量替换)
   > 			// 2.针对当前容器,忽略指定类型的自动注入(EnvironmentAware、ResourceLoaderAware、ApplicationEventPublisherAware)
   > 			//  忽略的目的是，这些特殊类型属性会通过特殊的方式赋值。
   > 			// 3.针对当前容器，给指定的类型注入指定的实列(BeanFactory、ResourceLoader、ApplicationContext等)
   > 			//  这三者都是当前容器实列来充当对应功能，因此只需要注入自己(this)即可。
   > 			// 4. 注册了一个BeanPostProcessor(ApplicationContextAwareProcessor, Bean后置处理器,invokeBeanFactoryPostProcessors进行回调)
   > 			// Prepare the bean factory for use in this context.
   > 			prepareBeanFactory(beanFactory);
   > 
   > 			try {
   > 				// Allows post-processing of the bean factory in context subclasses.
   > 				// 模板方法,符合"开闭原则",提供"beanFactory初始化后,Bean初始化前功能扩展"的入口.
   > 				// 子类AbstractRefreshableWebApplicationContext扩展了以下功能:
   > 				// 1.注册ServletContextAwareProcessor,可以针对指定Bean做特殊化处理.
   > 				// 2.针对web环境下注册了几种Scop(RequestScope,SessionScope,ServletContextScope);
   > 				MyLogger.log("5.15postProcessBeanFactory(当前类为AbstractApplicationContext对子类扩展提供入口).");
   > 				postProcessBeanFactory(beanFactory);
   > 
   > 				// Invoke factory processors registered as beans in the context.
   > 				MyLogger.log("5.16调用BeanFactory后置处理器");
   > 				// 1.调用BeanFactory后置处理器(默认没有Spring的实现,可扩展点之一)
   > 				invokeBeanFactoryPostProcessors(beanFactory);
   > 
   > 				// Register bean processors that intercept bean creation.
   > 				MyLogger.log("5.17注册BeanPostProcessors(BeanDefinition被初始化时通过回调调用)");
   > 				// 注册Bean后置处理器
   > 				registerBeanPostProcessors(beanFactory);
   > 
   > 				// Initialize message source for this context.
   > 				MyLogger.log("5.18初始化MessageSource()");
   > 
   > 				// MessageSource 国际化相关的功能
   > 				initMessageSource();
   > 
   > 				// Initialize event multicaster for this context.
   > 				MyLogger.log("5.19初始化ApplicationEventMulticaster(事件多路广播)");
   > 				// 初始化事件广播器,向BeanFactory容器中注册单列applicationEventMulticaster(SimpleApplicationEventMulticaster).
   > 				initApplicationEventMulticaster();
   > 
   > 				// Initialize other special beans in specific context subclasses.
   > 				// 模板方法
   > 				onRefresh();
   > 
   > 				// Check for listener beans and register them.
   > 				MyLogger.log("5.21注册监听器Listeners");
   > 				// 监听器依赖ApplicationContext事件广播器(ApplicationEventMulticaster,监听是建立在广播器上的).
   > 				// 1.首先注册静态(Spring提供,默认无)和自定义的ApplicationContext监听器
   > 				// 2.发送ApplicationContext早期事件(earlyApplicationEvents)的通知.
   > 				registerListeners();
   > 
   > 				// Instantiate all remaining (non-lazy-init) singletons.
   > 				MyLogger.log("5.22初始化所有非lazy-init的单例模式的Bean");
   > 				finishBeanFactoryInitialization(beanFactory);
   > 
   > 				// Last step: publish corresponding event.
   > 				MyLogger.log("5.23容器启动完成,处理其他后续事件");
   > 				finishRefresh();
   > 			}
   > 
   > 			catch (BeansException ex) {
   > 				if (logger.isWarnEnabled()) {
   > 					logger.warn("Exception encountered during context initialization - " +
   > 							"cancelling refresh attempt: " + ex);
   > 				}
   > 
   > 				// Destroy already created singletons to avoid dangling resources.
   > 				destroyBeans();
   > 
   > 				// Reset 'active' flag.
   > 				cancelRefresh(ex);
   > 
   > 				// Propagate exception to caller.
   > 				throw ex;
   > 			}
   > 
   > 			finally {
   > 				// Reset common introspection caches in Spring's core, since we
   > 				// might not ever need metadata for singleton beans anymore...
   > 				resetCommonCaches();
   > 			}
   > 		}
   > 	}
   > ~~~
   >
   > 

8. 适配器模式

   > ​	是指将一个类的接口转化成用户期望的另一个接口，使原本接口不兼容的类可以一起工作，属于结构型设计模式。典型的应用场景，已经存在的类和当前需求不匹配。适配器模式不是程序初始设计时考虑的设计模式，而是伴随着程序版本的迭代产生的。
   >
   > ​	在Spring中，适配器模式也有应用。在Spring AOP中AdvisorAdapter类的三个实现类中，MethodBeforAdvisorAdapter、AfterReturningAdvisorAdapter和 ThrowAdvisorAdapter。在Spring MVC中的HandlerAdapter类，他有多个子类，不同子类通过supports()方法来实现对当前模式是否支持。与策略模式的区别是，测试模式本身逻辑就可以判断出当前支持的实现类，而适配器模式需要遍历调用所有子类的supports()来实现。
   >
   > ```java
   > public interface HandlerAdapter {
   > 
   >    // 判断是否支持该配置器
   >    boolean supports(Object handler);
   > 
   >    
   >    @Nullable
   > 	ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception;
   > 
   > 	
   > 	long getLastModified(HttpServletRequest request, Object handler);
   > 
   > }
   > ```
   >
   > 

9. 装饰者模式

   > 是指在不改变原有对象的基础上，将功能附加到对象上，提供了比继承更有弹性的方案，相当于扩展了原有对象的功能。属于结构型模式。java中的I/O流相关的类就是装饰者模式，比如BufferReader、InputStream、OutputStream等。Spring中的BeanDefinitionWrapper也属于装饰模式，扩展了BeanDefiniton的功能。

10. 观察者模式