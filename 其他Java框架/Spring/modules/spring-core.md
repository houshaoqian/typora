# Spring核心容器

​	包括spring-Beans、spring-core、spring-context，spring-expression（Spring Expression Language，SpEL 表达式）4个模块组成。spring-Beans和spring-core是Spring框架的核心模块，提供了Bean包含了控制反转（Inversion of Control, Ioc）和依赖注入（Dependency Injection，DI）。BeanFactory是实现Ioc和Di的核心接口，提供基础功能。

​	**spring-context**模块是基于spring-Beans和spring-core之上，提供了高级容器的功能，核心接口是ApplicationContext，继承自BeanFactory。该接口在BeanFacotry基础之上，提供了应用上下文，环境等高级特性。

​	**spring-expression**，是统一表达式语言（EL，提供了查询、管理运行中的对象、调用对象方法，操作数组和集合等）的扩展模块。在EL基础之上，提供了额外的功能，比如函数调用，简单字符串的模板函数等。在实际的Spring项目中是不可或缺的模块。

​	spring-context-support，是对Spring Ioc容器的扩展。

​	spring-context-indexer模块是Spring类管理组件和Classpath扫描组件。

## 顶级接口

​	Spring在设计上遵循单一设计原则，在架构实现中，所有单一职责的角色或者功能都被定义成接口，而复杂的功能或者逻辑通过接口的继承和组合的方式实现。因此在理解Spring整体架构时，了解其最顶层接口是非常有必要的。

### spring-core

​	Spring最基础模块，不依赖其他模块，包含以下顶级接口及asm、cglib等字节码工具包。

* @AliasFor，别名。

* @Order，指定Bean等的加载顺序。

* AliasRegistry，别名Ioc容器，提供别名相关服务的ioc容器（注册别名，移除别名，获取别名等）。

* Environment，Spring环境。对当前Spring配置生效namespace（实际开发中的sit、prod等）的抽象，可多个环境同时生效。其实Spring的配置项包括Bean、数据源等都是挂在Environment下的，方便区分环境。

* Profiles，Environment的工具类，用于判断当前生效的是哪个Environment。

* Resource，资源。Spring中所有可加载资源的抽象，比如BeanDefinition的定义可以是本地xml文件或者网络资源等。

* PropertySource，属性值。Spring中各种配置，一个properties或者yml文件最终被抽象为PropertySource实例。该实例时绑定在Environment环境中的。name通常表示为配置来源，比如'applicationConfig: [classpath:/application.yml]'代表着SpringBoot中application.yml文件，T为文件内容，类型通常为Map类型。

  ~~~java
  public abstract class PropertySource<T> {
  
  	protected final String name;
  
  	protected final T source;
  	/**
  	 * Create a new {@code PropertySource} with the given name and source object.
  	 */
  	public PropertySource(String name, T source) {
  		Assert.hasText(name, "Property source name must contain at least one character");
  		Assert.notNull(source, "Property source must not be null");
  		this.name = name;
  		this.source = source;
  	}
  ~~~

* MutablePropertySources，属性值集合，PropertySource集合。可以同时包含多个PropertySource对象。在Spring中会同时出现多个属性配置源信息。比如在调试SpringBoot项目时，包含的属性配置源中的name有如下几种：

  > 1. configurationProperties：
  > 2. commandLineArgs：对启动命令中参数的封装，以空格为多个参数间的分割
  > 3. servletConfigInitParams：boot中默认为空。
  > 4. servletContextInitParams：boot中默认为空。
  > 5. systemProperties：JVM级别的环境变量。
  > 6. systemEnvironment：操作系统界别的环境变量。
  > 7. random：只包含了一个name=seed的变量，应该是给随机数使用的，待验证。
  > 8. applicationConfig: [classpath:/application-sit.yml]：应用级别的配置文件。
  > 9. applicationConfig: [classpath:/application.yml]：应用级别的配置文件。
  >
  > > Tips：
  > >
  > > 前6种都是Environment实例化时进行初始化的，后3个是容器启动后初始化的。
  > >
  > > random是在ConfigFileApplicationListener#postProcessEnvironment()中添加。



### spring-Beans

​	依赖spring-core模块。主要定义了Bean相关功能的接口。

DI依赖注入相关功能定义：

* @Autowired，
* @Qualifier，
* @Lookup，
* @Value,

factory工厂相关功能Bean定义：

* BeanDefinition，Bean元数据的定义。
* BeanDefinitionHolder，BeanDefinition的扩展类，在保存原数据的前提下，提供额外信息，比如别名，定义Bean的数据源等。
* ObjectFactory，是Bean的工厂方法，强调的是Bean的实例化封装，是否返回新实例等。比如单例类型Bean会通过Scope来返回已经创建的对象，原型类型则每次都创建一个新的实例。
* BeanWrapper，BeanDefinition的包装类。
* FactoryBean，在实例化(doGetBean)的时候有特殊调用(该类型的实例化结果返回的是getObject()类型而非其本身)。
* BeanFactory，核心Ioc容器。在功能实现上是内部持有一个BeanFactory子类来实现的(组合模式)。因各个Bean的生命周期Scope不同，因此各Bean的存储位置也不同。
     单例模式(signle)：保存在DefaultSingletonBeanRegistry(singletonObjects)中.单例有三级缓存，参考getSingleton()方法。
     原型模式(prototype)： 每次都是创建新对象,因此无需保存.
     其他模式：包括request session等,该模式采用的是通过全局的不同类型的Scope(SessionScope,RequestScope)对象来保存实例。
* Scope，生命周期抽象类。

流程相关功能Bean定义：

* BeanPostProcessor，Bean的后置处理器，提供在Bean实例化前和实例化后进行其他逻辑上的扩展，主要用在业务逻辑上的扩展。
* BeanFactoryPostProcessor，BeanFactory的后置处理器。主要用于Spring内部不同BeanFactory实现的特殊处理。
* InitializingBean，类似BeanPostProcessor，Bean在实例化后，Ioc容器会回调#afterPropertiesSet方法进行通知。与BeanPostProcessor的区别是，处理时机和处理范围不同。处理时机，BeanFactory是在Ioc容器初始化之后，对所有的Bean进行处理，InitializingBean是在Bean初始化之后仅对当前Bean进行回调通知。因此其执行顺序如下：
  1. BeanPostProcessor#postProcessBeforeInitialization 
  2. InitializingBean#afterPropertiesSet 
  3. BeanPostProcessor#postProcessAfterInitialization

### spring-context

​	依赖spring-core，spring-beans，spring-aop，spring-expression模块。主要定义了高级Ioc容器（即ApplicationContext，应用上下文）相关接口。

factory工厂相关定义：

* ApplicationContext，BeanFactory的高级形态，继承自BeanFactory。

* ApplicationContextInitializer，它是一个回调接口，主要目的是允许用户在ConfigurableApplicationContext类型（或其子类型）的ApplicationContext做refresh方法调用刷新之前，对ConfigurableApplicationContext实例做进一步的设置或处理。通常用于应用程序上下文进行编程初始化的Web应用程序中。

* ApplicationEvent，对容器中所有事件的抽象，比如容器启动，刷新，停止等。

  > ApplicationEvent是Spring中所有事件的顶级父类。ApplicationContextEvent则是Spring web项目中事件类的顶级类。其实现类包含
  >
  > ContextRefreshedEvent、ContextStartedEvent、ContextStoppedEvent、ContextClosedEvent。

* ApplicationEventPublisher，用于ApplicationContext事件发布。

* ApplicationListener，用于监听ApplicationContext事件。和ApplicationEvent、ApplicationEventPublisher共同构成了事件通知机制，用于Spring内部和应用程序级别的流程通知和控制。

* @Bean，通过注解方式定义Bean。

* @ComponentScans，扫描Bean定义规则。

* Lifecycle，对应用上下文的生命周期的抽象（start，stop，isRunning）。

* LifecycleProcessor，继承自Lifecycle，用于应用上下文的启动（#onRefresh，自动注册各组件component等）和停止（#onClose，卸载注册组件component等）时的通知。

* MessageSource，国际化配置的工具类。

stereotype相关的定义


* Component
* Controller
* Indexed
* Repository
* Service

scheduling相关的定义

* @Async：标识方法为异步执行（加入线程池执行）。
* @EnableAsync：标识在bean上，使得当前Ioc容器能够识别@Async注解。
* @Scheduled：标识方法定时执行策略。
* @EnableScheduling：标识在bean上，使得当前Ioc容器能够识别@Scheduled注解。
* @Schedules：@Scheduled的复数形式，同时定义多个执行策略。
* @SchedulingConfigurer：
* Trigger：
* TaskScheduler：
* TriggerContext：

Spring5条件化配置相关定义

* @Condition：Conditional注解的参数类型，标识一个布尔值类型的表达式。
* @Conditional： 条件化话注解。可以包含多个Condition（多个时，需用大括号括起来）。
* @Import：同xml配置中的<import/>标签，①引入由@Configuration注解的类②ImportSelector和ImportBeanDefinitionRegistrar的实现类③将普通的POJO注册成Spring Bean。
* ImportSelector：

Tips：Spring已经提供的条件化注解

> @ConditionalOnBean：仅仅在当前上下文中存在某个对象时，才会实例化一个Bean。
> @ConditionalOnClass：某个class位于类路径上，才会实例化一个Bean。
> @ConditionalOnExpression：当表达式（EL表达式或者常量）为true的时候，才会实例化一个Bean。
> @ConditionalOnMissingBean：仅仅在当前上下文中不存在某个对象时，才会实例化一个Bean。
> @ConditionalOnMissingClass：某个class类路径上不存在的时候，才会实例化一个Bean。
> @ConditionalOnNotWebApplication：不是web应用，才会实例化一个Bean。
> @ConditionalOnBean：当容器中有指定Bean的条件下进行实例化。
> @ConditionalOnMissingBean：当容器里没有指定Bean的条件下进行实例化。
> @ConditionalOnProperty：当指定的属性有指定的值时进行实例化。
> @ConditionalOnJava：当JVM版本为指定的版本范围时触发实例化。
> @ConditionalOnResource：当类路径下有指定的资源时触发实例化。
> @ConditionalOnJndi：在JNDI存在的条件下触发实例化。
> @ConditionalOnSingleCandidate：当指定的Bean在容器中只有一个，或者有多个但是指定了首选的Bean时触发实例化。

### spring-expression

### spring-context-support

### spring-context-indexer

