<!-- TOC -->

- [开启延时初始化](#开启延时初始化)
- [什么是SpringBoot场景启动器(`spring-boot-starter-*`)](#什么是springboot场景启动器spring-boot-starter-)

<!-- /TOC -->

# 开启延时初始化
可以使用`springApplication.setLazyInitialization(true);`实现应用的初始化延迟。这样程序相关的`Bean`就不会在程序启动时创建，而是在被使用到时才创建。所以这样会提高程序的启动效率，但是一些启动时可能会出现的错误只有在`Bean`被初始化时才会出现。并且这样的话需要确保JVM内存设置合适，否则启动时需要的内存和程序运行时创建Bean之后所需内存不同造成系统的异常。

# 什么是SpringBoot场景启动器(`spring-boot-starter-*`)
springboot场景启动器就是我们在springboot项目中常使用的starters，在starter中包含了一组方便的依赖关系符，可以将此starter配置在工程中，以获得spring及相关技术的一站式服务，而无需寻找各个单独相关的依赖自行组合。比如`spring-boot-stater-data-jpa`，其中至少包含了数据库驱动、数据库连接器、操作数据库的API，当我们需要使用spring进行数据库的访问时，就可以引入该依赖使用。而不需要以前使用spring时需要引入spring-xxx.jar、connector-java-xxx.jar、jdbc-xxx.jar这么多自行组合的相关依赖。所以这也可以避免在处理依赖版本问题耗费大量的精力。


