# Spring核心启动流程

Spring核心启动流程是在AbstractApplicationContext#refresh中实现的。先看下其类图。

![AbstractApplicationContext类图](./images/AbstractApplicationContext.png)

从上图中可以看出，AbstractApplicationContext类包含的角色有BeanFactory、EnvironmentCapable环境容器[^1]、ApplicationEventPublisher事件广播器、ResourceLoader资源加载器、MessageSource国际化等。Lifecycle表示该类具有生命周期（start/stop/isRunning）。



## 1. 上下文预设置

Spring启动前的准备工作，主要包括Spring启动状态、启动时间的设置以及环境变量的初始化。

1. 设置启动时间#startupDate。
2. 设置启动状态#closed=false及#active=true。
3. 初始化属性源。用实际实例替换任何根属性源。比如应用类型为传统web应用(包含web.xml文件)时，此处会将servletContextInitParams和servletConfigInitParams设置到属性源中，方便后续参数的读取。
4. 校验属性源中是否包含必须属性。默认必须属性为空，可通过ConfigurablePropertyResolver#setRequiredProperties进行设置。
5. 设置早期监听器。默认为空，容器启动前，没有事件广播器，故早期监听器也不起作用。
6. 初始化早期事件。默认空集合。

## 2. 获取新容器

​	获取一个全新的容器。Spring中默认有两个实现类AbstractRefreshableApplicationContext和GenericApplicationContext。在传统的Spring中使用的是前者，即可刷新容器，而在SpringBoot中使用的是后者。两者的区别主要在是否可刷新上。SpringBoot中在该流程前容器已启动（[传送门](../SpringBoot/README.md)），因此在GenericApplicationContext中设置序列化id后，直接返回当前容器即可。而在AbstractRefreshableApplicationContext中，如果当前容器已启动，则先关闭该容器。然后实例化一个全新的容器，并进行系列的设置。其详细流程如下：

1.  判断容器是否已启动。Spring上下文是通过组合模式实现容器BeanFactory功能的，因此判断当前容器是否已启动的逻辑为，在加锁[^2]的前提下，判断属性#beanFactory是否为空即可。
2.  销毁容器中的bean实例。
3.  关闭容器。
4.  实例化一个全新的容器。
5.  为当前新容器设置全新的序列化id。该id的组成格式为`className + '@' + hex(hashCode) `。
6.  定制当前容器，包含两项内容，是否允许重写BeanDefinition和是否允许嵌套引用。此处由于还未解析属性源，因此都是默认值false。
7.  加载BeanDefinition。

## 3. 容器预设置

## 4. 容器初始化后设置

## 5. 回调容器后置处理器

## 6. 注册bean后置处理器

## 7. 初始化国际化

## 8. 初始化应用广播器

## 9. 刷新时

## 10. 注册监听器

### 11. 完成容器初始化

### 12. 完成Spring的启动













------

[^1]: AbstractApplicationContext类通过私有的属性变量`environment`来实现的，具体类型是ConfigurableEnvironment。

[^2]: 该锁对象为一个final类型的Object对象#beanFactoryMonitor。
------

