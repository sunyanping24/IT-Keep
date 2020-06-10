<!-- TOC -->

- [Spring Boot介绍和特点](#spring-boot介绍和特点)
- [Spring Boot的核心注解`@SpringBootApplication`](#spring-boot的核心注解springbootapplication)
- [开启延时初始化](#开启延时初始化)
- [什么是SpringBoot场景启动器(`spring-boot-starter-*`)](#什么是springboot场景启动器spring-boot-starter-)
- [bootstrap.yml 和 application.yml 的区别](#bootstrapyml-和-applicationyml-的区别)
- [Http异步](#http异步)
- [SpringBoot自动装配](#springboot自动装配)

<!-- /TOC -->
# Spring Boot介绍和特点
`Spring Boot`是开发者和`Spring` 本身框架的中间层，帮助开发者统筹管理应用的配置，提供基于实际开发中常见配置的默认处理（即习惯优于配置），简化应用的开发，简化应用的运维；总的来说，其目的`Spring Boot`就是为了对`Java web` 的开发进行“**简化**”和“**加快**”速度，简化开发过程中引入或启动相关`Spring` 功能的配置。这样带来的好处就是降低开发人员对于框架的关注点，可以把更多的精力放在自己的业务代码上。   

1. 为Spring开发提供更加简单的使用和快速开发的技巧。--简化编码，不需要做大量的依赖匹配工作。
2. 具有开箱即用的默认配置功能，能根据项目依赖自动进行装配。--这也被称为"固执己见的默认配置"，当然这些默认的配置是可以进行修改的。
3. 不需要编写大量样板代码、XML配置和注释。在Spring Boot种更推荐使用JavaBean 的方式来进行配置。
4. Spring Boot可以以jar包的形式独立运行，运行一个项目只需要`java -jar xxx`来运行。--应用程序提供嵌入式HTTP服务器，如Tomcat和Jetty，可以轻松地开发和测试web应用程序。所以也可以不用使用war包的形式运行。

# Spring Boot的核心注解`@SpringBootApplication`
`@SpringBootApplication`是一个复合注解，其中最主要的就是：`@Configuration`、`@ComponentScan`、`@EnableAutoConfiguration`。所以在Spring Boot的启动类上同时加这3个注解和加1个复合注解是同等的，一般为了方便都会使用符合注解。   
[对于三体注解的解释，详看这一篇文章，讲的非常详细: http://c.biancheng.net/view/4625.html](http://c.biancheng.net/view/4625.html)


# 开启延时初始化
可以使用`springApplication.setLazyInitialization(true);`实现应用的初始化延迟。这样程序相关的`Bean`就不会在程序启动时创建，而是在被使用到时才创建。所以这样会提高程序的启动效率，但是一些启动时可能会出现的错误只有在`Bean`被初始化时才会出现。并且这样的话需要确保JVM内存设置合适，否则启动时需要的内存和程序运行时创建Bean之后所需内存不同造成系统的异常。

# 什么是SpringBoot场景启动器(`spring-boot-starter-*`)
springboot场景启动器就是我们在springboot项目中常使用的starters，在starter中包含了一组方便的依赖关系符，可以将此starter配置在工程中，以获得spring及相关技术的一站式服务，而无需寻找各个单独相关的依赖自行组合。比如`spring-boot-stater-data-jpa`，其中至少包含了数据库驱动、数据库连接器、操作数据库的API，当我们需要使用spring进行数据库的访问时，就可以引入该依赖使用。而不需要以前使用spring时需要引入spring-xxx.jar、connector-java-xxx.jar、jdbc-xxx.jar这么多自行组合的相关依赖。所以这也可以避免在处理依赖版本问题耗费大量的精力。**当然在starter中也可以实现一些公共的业务逻辑。**

Spring Boot 自定义Starter命名规则（官方建议）：
- Spring官方：`spring-boot-starter-{模块名称}`，比如`spring-boot-starter-redis`
- 第三方：`{模块名}-spring-boot-starter`      
虽说是官方建议，但是使用springboot框架就要遵循“约定高于配置”这一原则。所以按照官方的建议来命名吧。

![](http://sunyanping.gitee.io/it-keep/ASSET/starter核心说明.png)

# bootstrap.yml 和 application.yml 的区别
在SpringBoot的官方文档中并没有这块的说明，但是在SpringCloud的官方文档中有提及到，大意如下。
> Spring Cloud 构建于 Spring Boot 之上，在 Spring Boot 中有两种上下文，一种是 bootstrap, 另外一种是 application, bootstrap 是应用程序的父上下文，也就是说 bootstrap 加载优先于 applicaton。bootstrap 主要用于从额外的资源来加载配置信息，还可以在本地外部配置文件中解密属性。这两个上下文共用一个环境，它是任何Spring应用程序的外部属性的来源。bootstrap 里面的属性会优先加载，它们默认也不能被本地相同配置覆盖。

**总结一下：**    
- Spring Cloud 构建于 Spring Boot 之上，**在 Spring Boot 中有两种上下文，一种是 bootstrap,另外一种是 application**。
- **application 配置文件主要用于 Spring Boot 项目的自动化配置。**
- bootstrap 主要用于从额外的资源来加载配置信息，还可以在本地外部配置文件中解密属性。
- **bootstrap 是应用程序的父上下文，也就是说 bootstrap 加载优先于 applicaton。**
- **boostrap 由父 ApplicationContext 加载，会优先加载，它们默认也不能被本地相同配置覆盖。即就是bootstrap中的配置不能被application中的配置覆盖。**
- 这两个上下文共用一个环境，它是任何Spring应用程序的外部属性的来源。

# Http异步
HTTP异步的目的在帮助`DispatcherServlet`分担压力，提升吞吐量。但如果运行在NIO模式的服务容器上，就会产生负面影响，因为NIO本身就做了类似的事情，此时再加HTTP异步，则相当于又加了N多不必要的线程，导致性能主要消耗在线程的开销上，所以建议使用tomcat作为内嵌容器并且没有开启tomcat的NIO模式时，可以配合HTTP异步来提升程序性能。尤其是当业务繁重时，提升效果尤其明显。
在SpringBoot中编写HTTP异步代码，简单展示如下：(主要使用的是`WebAsyncTask`)    
```
@GetMapping("/hi")
    public WebAsyncTask<String> sayHi() {
        log.info("say hi ==> {}", Thread.currentThread().getName());
        WebAsyncTask<String> asyncTask = new WebAsyncTask<>(8000, () -> {
            log.info("say hi ==> {}", Thread.currentThread().getName());
            return "hi";
        });

        asyncTask.onTimeout(() -> {
            log.info("超时了");
            return "timeout";
        });

        asyncTask.onError(() -> {
            return "HI I ERROR";
        });

        return asyncTask;
    }
```

# SpringBoot自动装配
SpringBoot自动装配的关键点在于`@SpringBootApplication`注解。该注解是一个复合注解，里面包含了`@EnableAutoConfiguration`、`@SpringBootConfiguration`两个主要的注解，`@EnableAutoConfiguration`注解就是用来自动加载默认组件自动装配的。该注解如下：
```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```
其中`@Import({AutoConfigurationImportSelector.class})`这个注解引入了`AutoConfigurationImportSelector`这个自动配置选择器，在这个类里面包含了一些加载配置的实现逻辑。其中有这么一块代码：
```
private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
    ...
    Enumeration<URL> urls = classLoader != null ? classLoader.getResources("META-INF/spring.factories") : ClassLoader.getSystemResources("META-INF/spring.factories");
    ...
}
```
里面存在加载`META-INF/spring.factories`代码，这个文件里面配置的就是需要进行自动装配的配置类。这个文件是类似properties文件的来配置方式，以K,V的像是存在，K-接口路径,V-接口的实现(多个之间使用逗号,分割)




**Spring Boot比较好的，我个人推荐的一些文章。这些文章可以帮助我们更好更容易的理解Spring Boot框架。我个人计划在将来会根据Spring Boot的官方文档进行系统的学习，并记录相关的学习笔记。**     
参考文档：    
- [Spring Boot官方文档：https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/index.html](https://docs.spring.io/spring-boot/docs/2.2.6.RELEASE/reference/html/index.html)👍👍👍  
-  [Spring Boot框架入门教程: http://c.biancheng.net/spring_boot/](http://c.biancheng.net/spring_boot/)👍👍👍

