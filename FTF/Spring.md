### Springboot

#### Springboot核心功能

1. 独立运行Spring项目。
2. 内嵌Servlet容器，默认为Tomcat。
3. 自动装配（基于Spring5的条件化配置）。
4. 提供starter简化maven配置。
5. 准生产的应用监控actuator，也是基于starter实现的。

#### SpringBoot零配置是如何完成父子容器初始化的呢

> Tomcat加载项目由两种方式，一是常规的加载web.xml，二是通过加载ServletContainerInitializer（META-INF/services/javax.servlet.ServletContainerInitializer）实现类来实现。
>
> SpringBoot零配置就是基于第二种方式实现的。
>
> 1. 在spring子模块spring-web中，对应的文件夹下放了该文件(META-INF/services/javax.servlet.ServletContainerInitializer)，该文件内容是Springboot的实现的子类全类名(org.springframework.web.SpringServletContainerInitializer)。
> 2. 该实现类需要实现onStartup(@Nullable Set<Class<?>> webAppInitializerClasses, ServletContext servletContext)接口。该接口第二个参数是应用上下文类型，第一个是class类型，具体类型参考下一步。
> 3. 在该实现类上需要配上注解@HandlesTypes(WebApplicationInitializer.class)，表示该启动类实现类是指定类，也即第二步中，第一个参数类型，是Springboot启动的关键实现类。



#### SpringBoot启动流程

1. SpringApplication初始化（new一个新对象，配置类加载器和指定应用类型web应用或响应式等）。
2. Spring容器的启动（指定环境变量environment，资源，监听器，应用上下文创建）标准Spring容器启动refresh()。
3. 加载自动化配置（各starter.jar包下得MATE-INF/spring.factory文件里的自动配置类）。