## 核心功能之自动配置

### @SpringBootApplication注解
```mermaid

graph TD 
SpringBootApplication[&#64SpringBootApplication] --> SpringBootConfiguration[&#64SpringBootConfiguration]
SpringBootApplication --> EnableAtuoConfiguration[&#64EnableAtuoConfiguration]
SpringBootApplication --> ComponentScan[&#64ComponentScan]
SpringBootConfiguration --> Configuration[&#64Configuration]
EnableAtuoConfiguration --> AutoConfiuration[&#64AutoConfigurationPackage]
EnableAtuoConfiguration --> Import[&#64Import]

```
　													**@ SpringBootApplication组合 结构图**
@SpringBootApplication =  @EnableAutoConfiguration + @SpringBootConfiguration + @ComponentScan；

1. @EnableAutoConfiguration：
   1. 开启自动配置功能，加载各种XXXAutoCOnfiguration。
   2. 被注解的类所在路径为@Entity扫描的根路径。
2. @SpringBootConfiguration：组合了@Configuration注解，表示入口类也可以当做配置类去使用（入口类也可以使用@Bean等）。
3. @ComponentScan：开启自动扫描功能（@Controller/@Service/@Component/@Repository），默认值为当前被注解类所在的路径（因此在默认情况下，SpringBoot入口类必须在最顶级包下）。



------

### @EnableAutoConfiguration

~~~java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {

	String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

	Class<?>[] exclude() default {};

	String[] excludeName() default {};
}
~~~

1. **该注解自动配置功能的核心实现者是@AutoConfigurationImportSelector注解。**
2. 可根据环境变量中的spring.boot.enableautoconfiguration的值来开启(true)或关闭(false)自动配置功能，默认开启。
3. exclude指定排除的自动配置类，该类是在spring.factories文件中由org.springframework.boot.autoconfigure.EnableAutoConfiguration指定的类。spring.factories：配置文件，位于META-INF文件下（该文件的也可以配置其他待注册的类型???）。
4. excludeName作用同exclude，通过全类名进行排除。
5. @Import注解功能同xml文件中的<import/>标签，可以导入@Configuration配置类、ImportSelector实现类、普通的POJO等注册为Spring Bean。

> spring.factories文件：位于各个jar包的/META-INF文件下（该文件的也可以配置其他待注册的类型，AutoConfigurationImportFilter、AutoConfigurationImportListener、AutoConfigurationImportListener、ApplicationContextInitializer、ApplicationListener等）。



------

### @AutoConfigurationImportSelector

~~~java
public interface ImportSelector {
    String[] selectImports(AnnotationMetadata var1);

    @Nullable
    default Predicate<String> getExclusionFilter() {
        return null;
    }
}
~~~

1. ImportSelector，Import选择器，自动选择导入哪些@Configuration配置类。每个实现类代表了一个类型的选择器，比如AutoConfigurationImportSelector表示自动配置选择器，
2. 选择的结果通过selectImports方法实现，参数AnnotationMetadata类型，返回的数组代表将被@Import导入的配置类的全类名。

#selectImports()自动配置流程：

~~~java
@Override
public String[] selectImports(AnnotationMetadata annotationMetadata) {
    if (!isEnabled(annotationMetadata)) {
        return NO_IMPORTS;
    }
    AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
        .loadMetadata(this.beanClassLoader);
    AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(autoConfigurationMetadata,
                                                                              annotationMetadata);
    return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
}

protected AutoConfigurationEntry getAutoConfigurationEntry(AutoConfigurationMetadata autoConfigurationMetadata,
			AnnotationMetadata annotationMetadata) {
		if (!isEnabled(annotationMetadata)) {
			return EMPTY_ENTRY;
		}
		AnnotationAttributes attributes = getAttributes(annotationMetadata);
		List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
		configurations = removeDuplicates(configurations);
		Set<String> exclusions = getExclusions(annotationMetadata, attributes);
		checkExcludedClasses(configurations, exclusions);
		configurations.removeAll(exclusions);
		configurations = filter(configurations, autoConfigurationMetadata);
		fireAutoConfigurationImportEvents(configurations, exclusions);
		return new AutoConfigurationEntry(configurations, exclusions);
	}
~~~

* 检查自动配置开关（"spring.boot.enableautoconfiguration"配置是否开启，默认开启），关闭状态时直接返回空数组。

* 加载自动配置元数据（META-INF/spring-autoconfigure-metadata. properties）。

  > 1. 加载自动配置元数据的作用是为了过滤不需要或者不符合条件的自动配置类。
  >
  > 2. 该文件格式为：自动配置类的全类名.注解名称=值（多个值用逗号分割）。
  > 3. 自动配置类是指Spring预定义的各种自动配置类（XXXAutoConfiguration），注解是指各种条件化注解（ConditionalOnClass、ConditionalOnBean、ConditionalOnWebApplication、AutoConfigureAfter、AutoConfigureBefore等），值是指需要满足的条件，ConditionalOnBean的值表示容器中需要包含值对应的Bean。

* 加载自动配置组件。

  > 1. 加载各个jar包中META-INF/spring.factories文件中的自动配置类（key等于"org.springframework.boot.autoconfigure.EnableAutoConfiguration"）。
  > 2. 加载的过程是将spring.factories中内容封装成Map<String, List>的过程，String表示配置类型（EnableAutoConfiguration、AutoConfigurationImportFilter、AutoConfigurationImportListener此处为EnableAutoConfiguration）。List表示自动配置类的集合。加载结果将会被缓存。

* 排除指定的组件。

  > 1. 被排除组件是指在@SpringBootApplication(exclude="")指定的自动配置类。
  > 2. 被排除的类必须是在spring.factories中指定的自动配置类（会校验该类是否在自动配置列表中）。

* 过滤自动配置组件。

  > 1. 过滤的目的是排除SpringBoot预定义的自动配置类，比如DataSourceAutoConfiguration。过滤过程参考"加载自动配置元数据"。
  > 2. 过滤器是spring.factories中指定的AutoConfigurationImportFilter类型（默认有三个OnClassCondition、OnBeanCondition、OnWebApplicationCondition）。
  > 3. 过滤的过程是，双层循环对所有过滤器和所有自动配置类进行match()匹配。
  > 4. 自定义过滤器的话，可继承FilteringSpringBootCondition类，实现#getOutcomes()即可。
  > 5. 过滤和排除指定组件的区别在于，前者是排除不满足条件的预定义自动配置类。排除则侧重于开发者手动排除。逻辑上都是为了让指定的自动配置不进行配置。

* 事件注册。

  > 1. 在经过上述步骤后，得到了两个集合，"自动配置"集合 和 "被排除的自动配置"集合。
  > 2. 将两个集合封装成AutoConfigurationImportEvent事件。
  > 3. 从spring.factories缓存中取出所有的AutoConfigurationImportListener监听器类型。
  > 4. 将事件通过监听器广播出去。



------

## SpringBoot启动流程

SpringBoot启动流程分为两个步骤。

1. SpringApplication实例化。
2. Spring Ioc容器启动。

------

### SpringApplication启动类实例化

~~~java
public SpringApplication(Class<?>... primarySources) {
    this(null, primarySources);
}
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
    this.resourceLoader = resourceLoader;
    Assert.notNull(primarySources, "PrimarySources must not be null");
    this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
    this.webApplicationType = WebApplicationType.deduceFromClasspath();
    setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
    setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
    this.mainApplicationClass = deduceMainApplicationClass();
}
~~~

实例化流程：

1. 参数赋值，resourceLoader资源加载器（默认为空，不指定），primarySources启动类。

   > 构造函数的参数为Class类型，因此启动类可以是非当前mian方法所在的类（任何被@SpringBootApplication注解的类都可以）。

2. 推断应用类型（REACTIVE/NONE/SERVLET）。

   > 1. 应用类型通过枚举类型WebApplicationType（REACTIVE响应式、NONE普通、SERVLET web应用）表示。
   >
   > 2. 推断逻辑主要依靠ClassUtils.isPresent()方法，即判断classpath是否存在对应的类有，判断逻辑如下：
   >
   >    > ①存在org.springframework.web.reactive.DispatcherHandler类且不存在org.springframework.web.servlet.DispatcherServlet 和 org.glassfish.jersey.servlet.ServletContainer的类是响应式REACTIVE应用。
   >    >
   >    > ②不存在javax.servlet.Servlet和org.springframework.web.context.ConfigurableWebApplicationContext的是普通应用。
   >    >
   >    > ③其他的都是web应用SERVLET。

3. 获取ApplicationContextInitializer类集合并设置对应属性值。

   > 1. 从spring.factories缓存中获取ApplicationContextInitializer类型的集合。
   > 2. 将上述集合赋值到SpringApplication#initializers属性上。
   > 3. ApplicationContextInitializer类是Spring应用的一个回调接口，应用在ConfigurableApplicationContext类型（或其子类型）的ApplicationContext做refresh方法调用刷新之前，对ConfigurableApplicationContext实例做进一步的设置或处理。通常用于应用程序上下文进行编程初始化的Web应用程序中。

4. 获取ApplicationListener类集合并设置对应属性值。

   > 1. 从spring.factories缓存中获取ApplicationListener类型的集合。
   > 2. 将上述集合赋值到SpringApplication#listeners属性上。
   > 3. ApplicationListener类是Spring应用监听器。当Spring容器事件ApplicationEvent发布之后，需要处理特定事件时，如数据的加载、初始化缓存、特定任务的注册等操作。而在此阶段，更多的是用于ApplicationContext管理Bean过程的场景。

5. 推断入口类并设置对应属性的值。SpringApplication#mainApplicationClass值。

   ~~~java
   private Class<?> deduceMainApplicationClass() {
       try {
           StackTraceElement[] stackTrace = new RuntimeException().getStackTrace();
           for (StackTraceElement stackTraceElement : stackTrace) {
               if ("main".equals(stackTraceElement.getMethodName())) {
                   return Class.forName(stackTraceElement.getClassName());
               }
           }
       }
       catch (ClassNotFoundException ex) {
       }
       return null;
   }
   ~~~



------

### SpringBoot启动流程

SpringBoot启动流程是指SpringApplication#run()方法运行的过程。

------

spring.factories文件：位于各个jar包的/META-INF文件下，可配置的类型有

1. 自动配置类：AutoConfigurationImportFilter、AutoConfigurationImportListener、AutoConfigurationImportListener。
2. Ioc容器类：ApplicationContextInitializer、ApplicationListener。



@EnableConfigurationProperties和@ConfigurationProperties搭配实现属性注入。

