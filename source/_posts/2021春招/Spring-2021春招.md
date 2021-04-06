---
title: Spring-2021春招
date: 2021-04-06 17:14:54
tags: [Spring,面试]
---

主要介绍Spring的基本概念、Spring框架、Spring机制与实现、Spring应用相关、Context的初始化流程、Bean的生命周期、Spring Boot相关。

<!--more-->

#### 基本概念

![](http://img.wanghaojun.cn//other/20210406151843.png)

##### 1.IOC

IOC，控制反转，在spring中，对象的属性是由对象自己创建的，就是正向流程；如果属性不是对象创建，而是由spring来自动进行装配，就是控制反转。这里的DI也就是依赖注入，就是实现控制反转的方式。正向流程导致了对象于对象之间的高耦合，IOC可以解决对象耦合的问题，有利于功能的复用，能够使程序的结构变得非常灵活。

依赖注入的主要方式：

接口注入

Construct注入

Setter注入

##### 2.context & bean

所有被spring管理的、由spring创建的、用于依赖注入的对象，就叫做一个bean。Spring创建并完成依赖注入后，所有bean统一放在一个叫做context的上下文中进行管理。

##### 3.AOP

AOP就是面向切面编程。如右面的图，一般程序执行流程是从controller层调用service层、然后service层调用DAO层访问数据，最后再逐层返回结果。

这个是图中向下箭头所示的按程序执行顺序的纵向处理。但是，一个系统中会有多个不同的服务，例如用户服务、商品信息服务等等，每个服务的controller层都需要验证参数，都需要处理异常，如果按照图中红色的部分，对不同服务的纵向处理流程进行横切，在每个切面上完成通用的功能，例如身份认证、验证参数、处理异常等等、这样就不用在每个服务中都写相同的逻辑了，这就是AOP思想解决的问题。

AOP以功能进行划分，对服务顺序执行流程中的不同位置进行横切，完成各服务共同需要实现的功能。

#### spring框架

![](http://img.wanghaojun.cn//other/20210406152216.png)

core组件是spring所有组件的核心；bean组件和context组件我刚才提到了，是实现IOC和依赖注入的基础；AOP组件用来实现面向切面编程；web组件包括springmvc是web服务的控制层实现。

#### spring机制与实现

![](http://img.wanghaojun.cn//other/20210406152612.png)

##### AOP

AOP的实现是通过代理模式，在调用对象的某个方法时，执行插入的切面逻辑。实现的方式有动态代理也叫运行时增强，比如jdk代理、CGLIB；静态代理是在编译时进行织入或类加载时进行织入，比如AspectJ。

关于AOP还需要了解一下对应的Aspect、pointcut、advice等注解和具体使用方式。

##### 核心接口类

- ApplicationContext保存了ioc的整个应用上下文，可以通过其中的beanfactory获取到任意到bean；
- BeanFactory主要的作用是根据bean definition来创建具体的bean；
- BeanWrapper是对Bean的包装，一般情况下是在spring ioc内部使用，提供了访问bean的属性值、属性编辑器注册、类型转换等功能，方便ioc容器用统一的方式来访问bean的属性；
- FactoryBean通过getObject方法返回实际的bean对象，例如motan框架中referer对service的动态代理就是通过FactoryBean来实现的。

#### Spring应用相关

![](http://img.wanghaojun.cn//other/20210406153229.png)

##### 类型类注释：

类型类注释包括controller、service、Repository、Component、Configuration、Bean等，需要重点了解

其中component和bean注解的区别如下：

- @Component注解在类上使用表明这个类是个组件类，需要Spring为这个类创建bean。
- @Bean注解使用在方法上，告诉Spring这个方法将会返回一个Bean对象，需要把返回的对象注册到Spring的应用上下文中。

##### 设置类注释：

主要有Required Autowired&Qualifier、Scope等

@Autowired 注解是按照类型（byType）装配依赖对象，默认情况下它要求依赖对象必须存在，如果允许null 值，可以设置它的required属性为 false。如果我们想使用按照名称（byName） 来装配，可以结合@Qualifier 注解一起使用。

@Resource注解如有指定的name属性，先按该属性进行byName方式查找装配；其次再进行默认的byName方式进行装配；如果以上都不成功，则按byType的方式自动装配。

@Autowired 与 @Resource 的异同：

@Autowired与@Resource都可以用来装配bean。都可以写在字段上，或写在setter方法上。

-  @Autowired默认按类型装配（属于spring规范），默认情况下必须要求依赖对象必须存在，如果要允许null 值，可以设置它的required属性为false，如：@Autowired(required=false) ，如果我们想使用名称装配可以结合@Qualifier注解进行使用。
-  @Resource（属于J2EE规范），默认按照名称进行装配，名称可以通过name属性进行指定。如果没有指定name属性，当注解写在字段上时，默认取字段名进行按照名称查找，如果注解写在setter方法上默认取属性名进行装配。 当找不到与名称匹配的bean时才按照类型进行装配。但是需要注意的是，如果name属性一旦指定，就只会按照名称进行装配。

它们的作用相同都是用注解方式注入对象，但执行顺序不同。@Autowired先byType，@Resource先byName。

##### web类注解

主要以了解为主，关注@RequestMapping、@GetMapping、@PostMapping等路径匹配注解，以及@PathVariable、@RequestParam 等参数获取注解。

#### spring 的Context的初始化流程

![](http://img.wanghaojun.cn//other/20210406161646.png)

图中左上角是三种类型的context，xml配置方式的context、springboot的context和web服务的context。不论哪种context，创建后都会调用到AbstractApplicationContext类的refresh方法，这个方法是我们要重点分析的。

**refresh方法中，操作共分13步：**

第1步：对刷新进行准备，包括设置开始时间、设置激活状态、初始化context环境中的占位符，这个动作根据子类的需求由子类来执行，然后验证是否缺失必要的properties；

第2步：刷新并获得内部的bean factory；

第3步：对bean factory进行准备工作，比如设置类加载器和后置处理器、配置不进行自动装配的类型、注册默认的环境bean；

第4步：为context的子类提供后置处理bean factory的扩展能力。如果子类想在bean定义加载完成后，开始初始化上下文之前做一些特殊逻辑，可以复写这个方法；

第5步，执行context中注册的bean factory后缀处理器；

注：这里有两种后置处理器，一种是可以注册bean的后缀处理器，另一种是针对bean factory进行处理的后置处理器。执行的顺序是，先按优先级执行可注册bean的处理器，在按优先级执行针对beanfactory的处理器。

对springboot来说，这一步会进行注解bean definition的解析。流程如右面小框中所示，由ConfigurationClassPostProcessor触发、由ClassPathBeanDefinitionScanner解析并注册到bean factory。

第6步：按优先级顺序在beanfactory中注册bean的后缀处理器，bean后置处理器可以在bean初始化前、后执行处理；

第7步：初始化消息源，消息源用来支持消息的国际化；

第8步：初始化应用事件广播器。事件广播器用来向applicationListener通知各种应用产生的事件，是一个标准的观察者模式；

第9步，是留给子类的扩展步骤，用来让特定的context子类初始化其他的bean；

第10步，把实现了ApplicationListener的bean注册到事件广播器，并对广播器中的早期未广播事件进行通知；

第11步，冻结所有bean描述信息的修改，实例化非延迟加载的单例bean；

第12步，完成上下文的刷新工作，调用LifecycleProcessor的onFresh方法以及发布ContextRefreshedEvent事件；

第13步：在finally中，执行第十三步，重置公共的缓存，比如ReflectionUtils中的缓存、AnnotationUtils中的缓存等等；

至此，spring的context初始化完成。这里仅介绍了最主要的主流程，建议课后阅读源码来复习这个知识点，补全细节。

#### Spring中bean的生命周期

![](http://img.wanghaojun.cn//other/20210406162539.png)

**面试中经常问到的bean的生命周期，先看绿色的部分，bean的创建过程：**

第1步：调用bean的构造方法创建bean；

第2步：通过反射调用setter方法进行属性的依赖注入；

第3步：如果实现BeanNameAware接口的话，会设置bean的name；

第4步：如果实现了BeanFactoryAware，会把bean factory设置给bean；

第5步：如果实现了ApplicationContextAware，会给bean设置ApplictionContext；

第6步：如果实现了BeanPostProcessor接口，则执行前置处理方法；

第7步：实现了InitializingBean接口的话，执行afterPropertiesSet方法；

第8步：执行自定义的init方法；

第9步：执行BeanPostProcessor接口的后置处理方法。

这时，就完成了bean的创建过程。

在使用完bean需要销毁时，会先执行DisposableBean接口的destroy方法，然后在执行自定义的destroy方法。

#### Spring Boot相关

![](http://img.wanghaojun.cn//other/20210406164218.png)

##### 启动流程

主要步骤首先要配置environment，然后准备context上下文，包括执行applicationContext的后置处理、初始化initializer、通知listener处理contextPrepared和contextLoaded事件。最后执行refreshContext，也就是前面介绍过的AbstractApplicationContext类的refresh方法。

##### 配置文件

然后要知道在Spring Boot中有两种上下文，一种是bootstrap, 另外一种是application。

bootstrap是应用程序的父上下文，也就是说bootstrap会先于applicaton加载。bootstrap主要用于从额外的资源来加载配置信息，还可以在本地外部配置文件中解密属性。bootstrap里面的属性会优先加载，默认也不能被本地相同配置覆盖。

##### 注解

@SpringBootApplication包含了@ComponentScan、@EnableAutoConfiguration、@SpringBootConfiguration三个注解

而@SpringBootConfiguration注解包含了@Configuration注解。也就是springboot的自动配置功能。

@Conditional注解就是控制自动配置的生效条件的注解，例如bean或class存在、不存在时进行配置，当满足条件时进行配置等等。

##### 特色模块

- starter是springboot提供的无缝集成功能的一种方式，使用某个功能时开发者不需要关注各种依赖库的处理，不需要具体的配置信息，由Spring Boot自动配置进行bean的创建。例如需要使用web功能时，只需要在依赖中引入spring-boot-starter-web即可。
- actuator是用来对应用程序进行监视和管理，通过restful api请求来监管、审计、收集应用的运行情况。
- devtools提供了一系列开发工具的支持，来提高开发效率。例如热部署能力等。
- CLI就是命令行接口，是一个命令行工具，支持使用Groovy脚本，可以快速搭建spring原型项目。

#### 参考资料

https://zhuanlan.zhihu.com/p/59327709