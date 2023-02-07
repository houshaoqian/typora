## core->none
1. asm
2. cglib
3. AliasFor
4. Order
5. AliasRegistry
6. Environment
7. Profiles
8. Resource
## beans->(spring-core)
1. Autowired
2. Qualifier
3. Lookup
4. Value
5. BeanDefinition
    bean元数据的定义;存在RootBeanDefinition和GenericBeanDefinition等几种实体模型,但需要关注的是概念模型MergedBeanDefinition.
    只有该抽象模型才能被实例化为Bean,即其他BeanDefinition必须转化或合并为MergedBeanDefinition才能够表示完整的信息可以被实例化.
6. BeanDefinitionHolder
    BeanDefinition数据的持有者,保存了额外的信息,比如别名等.
7. FactoryBean和ObjectFactory
    FactoryBean在实例化(doGetBean)的时候有特殊调用(该类型的实例化结果返回的是getObject()类型而非其本身).
    ObjectFactory的作用是Bean工厂方法,强调的是Bean的实例化封装,是否返回新实例等.比如单例类型Bean会通过Scope来返回已经创建的对象,
    原型类型则每次都创建一个新的实例.
8. BeanFactory
    以区别ObjectFactory,此处翻译为'Bean容器'(强调保存而非生成)更适合。在web项目中,常见的应用上下文实现类都实现了该接口,但在功能
    实现上是内部持有一个BeanFactory子类来实现的(组合模式).因各个Bean的生命周期Scope不同,因此各Bean的存储位置也不同.
    单例模式(signle):保存在DefaultSingletonBeanRegistry(singletonObjects)中.单例有三级缓存,参考getSingleton()方法.
    原型模式(prototype): 每次都是创建新对象,因此无需保存.
    其他模式:包括request session等,该模式采用的是通过全局的不同类型的Scope(SessionScope,RequestScope)对象来保存实例.
9. BeanPostProcessor
    扩展点.Bean的后置处理器,提供在Bean实例化前和实例化后进行功能上的扩展;
    注册时机:BeanFactory初始化之后,非惰性加载单例Bean实例化之前注册自定义的BeanPostProcessor(AbstractApplicationContext#registerBeanPostProcessors).
    一些特殊的Spring内置的处理器(ApplicationContextAwareProcessor)会提前(AbstractApplicationContext#prepareBeanFactory)注册;
    回调时机:#getBean->#doGetBean->#createBean时进行回调.
10. InstantiationAwareBeanPostProcessor 特殊的BeanPostProcessor 
    一种特殊的Bean后置处理器,Spring内部使用,处理优先级高于BeanPostProcessor,在实例化Bean时进行回调(#applyBeanPostProcessorsBeforeInstantiation).
    实现类有:ImportAwareBeanPostProcessor CommonAnnotationBeanPostProcessor AutowiredAnnotationBeanPostProcessor等.
    作用:在实例化BeanName前,此类用来判断该bean是否需要返回代理对象。
11. MergedBeanDefinitionPostProcessor 特殊的BeanPostProcessor
    该类型的后置处理器用来实例化前对BeanDifinition的修改.
11. BeanFactoryPostProcessor
    扩展点.BeanFactory的后置处理器.相较与Bean的后置处理器而言,该扩展点主要用于Spring内部不同BeanFactory实现的特殊处理;
    注册时机:BeanFactory初始化之后,非惰性单例初始化之前(AbstractApplicationContext#invokeBeanFactoryPostProcessors);
    回调时机:在注册后立即进行回调.
12. Scope
    参考BeanFactory
13. InitializingBean(与BeanPostProcessor的区别)
    子类在完成实例化后,BeanFactory会回调#afterPropertiesSet方法进行通知;
    与BeanPostProcessor的区别是:
    1.InitializingBean处理的对象是自身,而BeanPostProcessor处理的是所有Bean;
    2.执行顺序为BeanPostProcessor#postProcessBeforeInitialization > InitializingBean#afterPropertiesSet > BeanPostProcessor#postProcessAfterInitialization;
    3.执行入口都是#doGetBen->#initializeBean->#invokeInitMethods/#applyBeanPostProcessorsAfterInitialization;\

14. BeanNameGenerator
15. BeanWrapper
    对实例化之后的bean进行封装.该机制为动态AOP提供实现方式.

## context->(spring-core,spring-beans,spring-aop,spring-expression)
1. ApplicationContext
2. ApplicationEventPublisher & ApplicationEventMulticaster
    扩展点.事件广播器.针对应用上下文事件进行广播. 真正进行广播动作的是ApplicationEventMulticaster#multicastEvent.
    注册时机:在#refresh过程中,进行初始化的(initApplicationEventMulticaster).
    若要实现自定义的广播通知,需要先获取到应用上下文,根据应用上下文拿到广播器ApplicationEventMulticaster进行广播.
3. ApplicationListener
    与广播器配合使用,监听应用上下文事件.
    
## Spring启动流程
启动流程主要在AbstractApplicationContext#refresh方法中

1. prepareRefresh准备工作.
    1.1 设置容器 启动时间、启动状态;
    1.2 初始化系统环境变量servletContextInitParams和servletConfigInitParams.
    1.3 校验系统变量是否完整(默认无必须的参数).
    1.4 添加应用早期事件(earlyApplicationEvents,默认空)和早期事件监听器(earlyApplicationListeners,默认无).

2. obtainFreshBeanFactory初始化BeanFactory
    2.1 先创建BenFactory,默认类型为DefaultListableBeanFactory(若已启动则先关闭再创建).
    2.2 设置容器Id(beanFactory.setSerializationId(getId()))
    2.3 依据spring的配置文件加载BeanDefinition(不同配置使用不同解析器Parser)

3. prepareBeanFactory准备BeanFactory.
    3.1 指定ClassLoader、指定Spel表达式解析类、指定属性编辑器(Properties变量替换)
    3.2 针对当前容器,忽略指定类型的自动注入(EnvironmentAware、ResourceLoaderAware、ApplicationEventPublisherAware)(忽略的目的是，这些特殊类型属性会通过特殊的方式赋值)
    3.3 针对当前容器，给指定的类型注入指定的实列(BeanFactory、ResourceLoader、ApplicationContext等)这三者都是当前容器实列来充当对应功能，因此只需要注入自己(this)即可.
    3.4 注册了一个BeanPostProcessor(ApplicationContextAwareProcessor, Bean后置处理器,invokeBeanFactoryPostProcessors进行回调)
   
4. postProcessBeanFactory执行BeanFactory的后置处理器(容器自身逻辑).
    4.1 注册ServletContextAwareProcessor,可以针对指定Bean做特殊化处理.
    4.2 针对web环境下注册了几种Scop(RequestScope,SessionScope,ServletContextScope);
   
5. invokeBeanFactoryPostProcessors执行BeanFactory的后置处理器
    5.1 模板方法,给子类提供扩展功能,默认没有实现类. 
   
6. registerBeanPostProcessors注册Bean后置处理器
    通过beanFactory#getBeanNamesForType(BeanPostProcessor.class, true, false)方法从BeanDefinition中获取后置处理器类型. 
    Bean的后置处理器默认按照以下顺序进行注册和执行:PriorityOrdered子类优先于Ordered子类, Ordered子类优先于普通的后置处理器类.

7. initMessageSource国际化
   
8. initApplicationEventMulticaster初始化事件广播器
    8.1 从beanFactory中获取beanName='applicationEventMulticaster'的事件广播器.
    8.2 若不存在事件广播器(默认不存在),则实例化SimpleApplicationEventMulticaster为默认的广播器.

9. onRefresh模板方法
    给子类提供扩展功能,默认为空实现.
   
10. registerListeners注册事件监听器   
    10.1 先注册Spring内部定义的监听器(#prepareRefresh中注册的earlyApplicationListeners).
    10.2 再发送ApplicationContext早期事件(earlyApplicationEvents)的通知.早期的事件和监听在#prepareRefresh中已经添加了,
    但添加时,上下文/容器和事件广播器还没建立,因此事件并未发布出去,在步骤8中,事件广播器已建立,因此在这一步骤中第一时间将earlyApplicationEvents
    事件广播出去.
11. finishBeanFactoryInitialization完成容器初始化
    11.1 装配类型转换Service(beanName='conversionService');
    11.2 装配默认的属性解析器(若BeanFactory中未指定属性解析器则注册一个默认的解析器);
    11.3 LoadTimeWeaverAware先初始化该类型的Bean.该类型Bean的作用为了能够早些注册转换类,
        因此需要优先加载LoadTimeWeaverAware实现类(加载Bean时织入第三方模块，如AspectJ)转换类是指获取Bean时,获取的是其代理对象,以方便后续的切面编程等.
    11.4 初始化所有非惰性的单例Bean(preInstantiateSingletons).

## Bean初始化过程.   
    非惰性的bean在步骤11.4中进行初始化.惰性Bean初始化的时机是用到(从容器中获取#getBean)时进行初始化.真正获取的方法是AbstractBeanFactory#doGetBean.
1. beanName转换为标准名称.
    1.1 去掉beanName的'&'前缀.'&'前缀有特殊含义,表示获取FactoryBean实列本身.
    1.2 别名转换成非别名.针对AliasFor等.
2. 从单例三级缓存中获取该实例,若beanName未初始化则返回null.
    三级缓存指的是,在实例化Bean中,分别保存不同状态Bean的地方:
    1.singletonObjects: 保存的是已经实例化完成的Bean.
    2.earlySingletonObjects: 保存的是正在实例化中的Bean.属性未注入完成的Bean.比如实例化单例A时,A的一个属性B未被初始化,则会将A暂存在earlySingletonObjects中,
    然后去实例化B,实例化完B后再实例化A.
    3.singletonFactories: 保存的是beanName对应的工厂类ObjectFactory,通过该工厂类可以获取对应的Bean.
3. 从父类容器parentBeanFactory中查找.
    通过步骤2,仍未获取到对应实例的话,则从父容器中继续查找.若在父容器中也未找到,说明该Bean确实未初始化过,则进行以下初始化步骤.
4. 初始化前准备,设置状态(#markBeanAsCreated).
    1. 将该beanName标记为已实例化(将该beanName放到alreadyCreated集合中).
    2. 将该BeanDefinition的stale设置为false.意味着该BD需要进行合并才能再次进行初始化.这一步骤的目的是下次初始化的时候Bean元素据定义可能被扩展功能进行修改了,
       因此需要再次合并整理的BeanDefinition才能被再次实例化,以得到最新的Bean元素据.
5. 检查依赖是否已被实例化,未实例化的需要提前实例化.
    此处的依赖是构造函数依赖.先检查是否循环依赖,若不存在构造函数循环依赖,则迭代调用先初始化被依赖的实例.
6. 判断beanName的实例Scope类型,不同类型的实例化逻辑不同:
7. 单例模式:
    7.1 统一的Bean实例化阶段(#createBean):
        7.1.1 实例化Bean对应的Class对象,并将该对象与BeanDefinition绑定.
        7.1.2 初始化前逻辑处理,此处的逻辑处理指的是根据系统配置的InstantiationAwareBeanPostProcessor类来确定该Bean是否需要配置代理等.
            如果是则直接返回代理对象,结束本次实例化过程.
        7.1.3 实例化Bean(#doCreateBean)
        7.1.4 从缓存factoryBeanInstanceCache中获取是否有Bean的封装对象BeanWrapper.
        7.1.5 若无则选择合适的构造函数,进行实例化(#createBeanInstance),此处仅实例化一个Bean对象,未初始化任何属性.
        7.1.6 初始化属性AbstractAutowireCapableBeanFactory#initializeBean,初始化前调用BeanPostProcessor链,初始化完成调用#afterPropertiesSet方法进行通知.
    7.2 步骤7.1是在对应的ObjectFactory中执行的,相当于每一个Bean都对应了一个匿名的ObjectFactory.
    7.3 添加到单例容器中(#getSingleton)
        将单例加入到singletonObjects中并将singletonsCurrentlyInCreation(实例化中Bean集合)中去掉.
