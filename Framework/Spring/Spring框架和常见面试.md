<!-- TOC -->

- [Spring框架简介](#spring框架简介)
- [Spring的`DI`和`IOC`机制](#spring的di和ioc机制)
- [Spring的`AOP`机制](#spring的aop机制)
- [Spring支持的几种bean的作用域](#spring支持的几种bean的作用域)
- [Spring框架中的单例Beans是线程安全的么？](#spring框架中的单例beans是线程安全的么)
- [Spring 框架中都用到了哪些设计模式？](#spring-框架中都用到了哪些设计模式)
- [Spring事务的实现方式和实现原理](#spring事务的实现方式和实现原理)
  - [Spring事务的种类](#spring事务的种类)
  - [spring的事务传播行为](#spring的事务传播行为)
  - [Spring中的隔离级别](#spring中的隔离级别)
- [`@Transactional` 注解在什么情况下不会生效](#transactional-注解在什么情况下不会生效)

<!-- /TOC -->

# Spring框架简介
Spring框架是为了解决企业应用程序开发复杂性而创建的，框架的主要优势之一就是其分层架构，分层架构允许选择使用哪一个组件。

Spring 框架是一个分层架构，由 7 个定义良好的模块组成。Spring 模块构建在核心容器之上，核心容器定义了创建、配置和管理 bean 的方式
![Spring框架的组成](/ASSET/Spring框架的组成模块.jpg)
组成 Spring 框架的每个模块（或组件）都可以单独存在，或者与其他一个或多个模块联合实现。每个模块的功能如下：      
- **核心容器：**  核心容器提供 Spring 框架的基本功能。核心容器的主要组件是 BeanFactory，它是工厂模式的实现。BeanFactory 使用*控制反转* （IOC） 模式将应用程序的配置和依赖性规范与实际的应用程序代码分开。
- **Spring 上下文：**  Spring 上下文是一个配置文件，向 Spring 框架提供上下文信息。Spring 上下文包括企业服务，例如 JNDI、EJB、电子邮件、国际化、校验和调度功能。
- **Spring AOP：**  通过配置管理特性，Spring AOP 模块直接将面向切面的编程功能集成到了 Spring 框架中。所以，可以很容易地使 Spring 框架管理的任何对象支持 AOP。Spring AOP 模块为基于 Spring 的应用程序中的对象提供了事务管理服务。通过使用 Spring AOP，不用依赖 EJB 组件，就可以将声明性事务管理集成到应用程序中。
- **Spring DAO：**  JDBC DAO 抽象层提供了有意义的异常层次结构，可用该结构来管理异常处理和不同数据库供应商抛出的错误消息。异常层次结构简化了错误处理，并且极大地降低了需要编写的异常代码数量（例如打开和关闭连接）。Spring DAO 的面向 JDBC 的异常遵从通用的 DAO 异常层次结构。
- **Spring ORM：**  Spring 框架插入了若干个 ORM 框架，从而提供了 ORM 的对象关系工具，其中包括 JDO、Hibernate 和 iBatis SQL Map。所有这些都遵从 Spring 的通用事务和 DAO 异常层次结构。
- **Spring Web 模块：**  Web 上下文模块建立在应用程序上下文模块之上，为基于 Web 的应用程序提供了上下文。所以，Spring 框架支持与 Jakarta Struts 的集成。Web 模块还简化了处理多部分请求以及将请求参数绑定到域对象的工作。
- **Spring MVC 框架：**  MVC 框架是一个全功能的构建 Web 应用程序的 MVC 实现。通过策略接口，MVC 框架变成为高度可配置的，MVC 容纳了大量视图技术，其中包括 JSP、Velocity、Tiles、iText 和 POI。

**Spring框架的优点**      
- Spring属于低侵入式设计，代码的污染极低
- Spring的`DI`、`IOC`思想机制将对象的创建和对象间的依赖关系交给框架来进行处理，减少组件之间的耦合度
- Spring提供了`AOP`技术，可以面向切面变成，处理一些通用的任务，如安全、事务、日志、权限等可以进行集中式管理，从而可以得到更好的复用
- Spring对主流的应用框架提供了集成支持

# Spring的`DI`和`IOC`机制
实际上`DI`和`IOC`可以算的上是同一个概念，这两者是相辅相成的，缺一不可。原来对象之间的调用关系是由各个对象自己控制的，比如调用者在调用者内部创建被调用的对象，来创建依赖关系。在Spring中创建对象的任务交给了框架来进行处理，不需要再需要调用者来自己创建，所以称之为控制反转(IOC)；而当调用者需要依赖其他对象时，可以使用相关注解来进行注入，所以称为依赖注入(DI)。

# Spring的`AOP`机制
Spring框架的AOP机制可以让开发者把业务流程中的**通用功能抽取**出来，单独编写功能代码。在业务流程执行过程中，Spring框架会根据业务流程要求，自动把独立编写的功能代码**切入到流程**的合适位置。

AOP思想的实现一般都是基于 **代理模式** ，在JAVA中一般采用JDK动态代理模式，但是我们都知道，**JDK动态代理模式只能代理接口而不能代理类**。因此，Spring AOP 会这样子来进行切换，因为Spring AOP 同时支持 CGLIB、ASPECTJ、JDK动态代理。   
- 如果目标对象的**实现类实现了接口**，Spring AOP 将会采用 **JDK 动态代理**来生成 AOP 代理类；
- 如果目标对象的**实现类没有实现接口**，Spring AOP 将会采用 **CGLIB 来生成 AOP 代理类**——不过这个选择过程对开发者完全透明、开发者也无需关心。

# Spring支持的几种bean的作用域
Spring容器中的bean可以分为5个范围：
1. **singleton**：默认，每个容器中只有一个bean的实例，单例的模式由BeanFactory自身来维护。
2. **prototype**：为每一个bean请求提供一个实例。
3. **request**：为每一个网络请求创建一个实例，在请求完成以后，bean会失效并被垃圾回收器回收。
4. **session**：与request范围类似，确保每个session中有一个bean的实例，在session过期后，bean会随之失效。
5. **global-session**：全局作用域，global-session和Portlet应用相关。当你的应用部署在Portlet容器中工作时，它包含很多portlet。如果你想要声明让所有的portlet共用全局的存储变量的话，那么这全局变量需要存储在global-session中。全局作用域与Servlet中的session作用域效果相同。

# Spring框架中的单例Beans是线程安全的么？
 Spring框架并没有对单例bean进行任何多线程的封装处理。关于单例bean的线程安全和并发问题需要开发者自行去搞定。但实际上，大部分的Spring bean并没有可变的状态(比如Service类和DAO类)，所以在某种程度上说Spring的单例bean是线程安全的。如果你的bean有多种状态的话（比如 View Model 对象），就需要自行保证线程安全。最浅显的解决办法就是将多态bean的作用域由“singleton”变更为“prototype”。

# Spring 框架中都用到了哪些设计模式？
1. **工厂模式**：BeanFactory就是简单工厂模式的体现，用来创建对象的实例；
2. **单例模式**：Bean默认为单例模式。
3. **代理模式**：Spring的AOP功能用到了JDK的动态代理和CGLIB字节码生成技术；
4. **模板方法**：用来解决代码重复的问题。比如： RestTemplate, JmsTemplate, JpaTemplate。
5. **观察者模式**：定义对象键一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都会得到通知被制动更新，如Spring中listener的实现--ApplicationListener。

# Spring事务的实现方式和实现原理
Spring事务的本质其实就是数据库对事务的支持，没有数据库的事务支持，spring是无法提供事务功能的。真正的数据库层的事务提交和回滚是通过binlog或者redo log实现的。

## Spring事务的种类
spring支持编程式事务管理和声明式事务管理两种方式：
1. 编程式事务管理使用TransactionTemplate。
2. 声明式事务管理建立在AOP之上的。其本质是通过AOP功能，对方法前后进行拦截，将事务处理的功能编织到拦截的方法中，也就是在目标方法开始之前加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。

> 声明式事务最大的优点就是不需要在业务逻辑代码中掺杂事务管理的代码，只需在配置文件中做相关的事务规则声明或通过@Transactional注解的方式，便可以将事务规则应用到业务逻辑中。    
声明式事务管理要优于编程式事务管理，这正是spring倡导的非侵入式的开发方式，使业务代码不受污染，只要加上注解就可以获得完全的事务支持。唯一不足地方是，最细粒度只能作用到方法级别，无法做到像编程式事务那样可以作用到代码块级别。

## spring的事务传播行为
spring事务的传播行为说的是，当多个事务同时存在的时候，spring如何处理这些事务的行为。
- **PROPAGATION_REQUIRED**：如果当前没有事务，就创建一个新事务，如果当前存在事务，就加入该事务，该设置是最常用的设置。
- **PROPAGATION_SUPPORTS**：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务执行。
- **PROPAGATION_MANDATORY**：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就抛出异常。
- **PROPAGATION_REQUIRES_NEW**：创建新事务，无论当前存不存在事务，都创建新事务。
- **PROPAGATION_NOT_SUPPORTED**：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
- **PROPAGATION_NEVER**：以非事务方式执行，如果当前存在事务，则抛出异常。
- **PROPAGATION_NESTED**：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则按REQUIRED属性执行。

## Spring中的隔离级别
- **ISOLATION_DEFAULT**：这是个 PlatfromTransactionManager 默认的隔离级别，使用数据库默认的事务隔离级别。
- **ISOLATION_READ_UNCOMMITTED**：读未提交，允许另外一个事务可以看到这个事务未提交的数据。
- **ISOLATION_READ_COMMITTED**：读已提交，保证一个事务修改的数据提交后才能被另一事务读取，而且能看到该事务对已有记录的更新。
- **ISOLATION_REPEATABLE_READ**：可重复读，保证一个事务修改的数据提交后才能被另一事务读取，但是不能看到该事务对已有记录的更新。
- **ISOLATION_SERIALIZABLE**：一个事务在执行的过程中完全看不到其他事务对数据库所做的更新。

# `@Transactional` 注解在什么情况下不会生效
 `@Transactional` 注解可以作用在接口、接口方法、类、类方法上：
 - 添加注解的方法不是`public`修饰的，则不生效。
 - 在方法中调用同一个类中的其他方法。只有当前代理类的外部方法调用注解方法时代理才会被拦截。所以即使被调用的方法加了`@Transactional`注解事务也不会生效。
 - 默认抛出`uncheck exception`时会生效，`check exception`不会生效。若要`check exception`生效，可以进行配置`rollbackFor=Exception.class`。

当然上面这几种只是网上网友总结的比较多的几个答案。其他不生效的情况也存在，**尤其是要注意事务的转播性。**


**Spring框架的其他参考文章：**  
  
[Spring Bean的生命周期](https://www.cnblogs.com/zrtqsk/p/3735273.html)：这篇文章中非常详细的讲解了bean从创建到销毁的一系列过程，包含每一个过程涉及到的实现类。并且还有一些示例代码。  
[Spring AOP入门理论讲解，通俗易懂](https://segmentfault.com/a/1190000007469968): 这篇文章使用通俗易懂的方式讲解了Spring AOP的基础理论知识，但是没有涉及到讲解代理这块的知识。这块的理论基本上是基于Java Bean的方式来讲的知识，如果想了解xml配置的方式，请参考其他的文章。       
[Spring AOP底层实现-各种代理模式](https://juejin.im/post/5b90e648f265da0aea695672)： 从AspectJ静态代理、JDK静态代理、 JDK动态代理、CGLIB动态代理一直延伸到Spring AOP。

