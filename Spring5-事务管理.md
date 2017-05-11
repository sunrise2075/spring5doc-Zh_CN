# 13.事务管理

## 13.1 Spring框架事务管理简介

在选用Spring框架的众多引人瞩目的理由里面，强大的事务支持便是其中之一。Spring框架提供了对事务管理一致的抽象，这种做法为我们带来以下几点好处：

- 为JTA、JDBC、Hibernate和JPA等多种事务管理方案提供一致的编程模型
- 支持声明式事务管理
- 比起JTA之类繁琐无比的事务操作API，Spring框架的事务编程接口更加简洁明了
- 与Spring数据访问抽象层完美集成

在接下来的几个小节里，本文将向大家介绍Spring事务的价值贡献和技术。（这一章还将会包含关于最佳实践、应用服务器集成和对于一些其他常见问题解决方案的讨论）

- Spring框架事务模型的优势一节将会详尽地解释，相比基于EJB容器的事务管理解决方案和借助于专有的编程API（比如说Hibernate）来实现本地事务，为什么你应该选用Spring框架的事务管理的抽象层来应对事务管理问题
- 理解Spring框架如何抽象地看待一切事务一节勾勒了几个关键的Class类型，描述了如何从不同的数据库配置并且获取数据源（data source）
- 在事务内部统一调度资源一节，为大家描述了在一个事务处理的过程中，应用程序代码如何确保各种资源被恰当的创建、重用和清理
- 声明式事务管理一节，为大家讲解Spring框架如何支持您完成声明式事务编程
- 编程式事务管理一节，包含了Spring框架如何支持您在自己的编写的程序代码里面实现编程式事务的内容
- 事务绑定事件一节，将会告诉您，应该怎样在一个事物内部使用应用事件

## 13.2 Spring框架事务模型的优势

Traditionally, Java EE developers have had two choices for transaction management: global or local transactions, both of which have profound limitations. Global and local transaction management is reviewed in the next two sections, followed by a discussion of how the Spring Framework’s transaction management support addresses the limitations of the global and local transaction models.


### 13.2.1 全局事务

Global transactions enable you to work with multiple transactional resources, typically relational databases and message queues. The application server manages global transactions through the JTA, which is a cumbersome API to use (partly due to its exception model). Furthermore, a JTA UserTransaction normally needs to be sourced from JNDI, meaning that you also need to use JNDI in order to use JTA. Obviously the use of global transactions would limit any potential reuse of application code, as JTA is normally only available in an application server environment.

Previously, the preferred way to use global transactions was via EJB CMT (Container Managed Transaction): CMT is a form of declarative transaction management (as distinguished from programmatic transaction management). EJB CMT removes the need for transaction-related JNDI lookups, although of course the use of EJB itself necessitates the use of JNDI. It removes most but not all of the need to write Java code to control transactions. The significant downside is that CMT is tied to JTA and an application server environment. Also, it is only available if one chooses to implement business logic in EJBs, or at least behind a transactional EJB facade. The negatives of EJB in general are so great that this is not an attractive proposition, especially in the face of compelling alternatives for declarative transaction management.

### 13.2.2 本地事务

Local transactions are resource-specific, such as a transaction associated with a JDBC connection. Local transactions may be easier to use, but have significant disadvantages: they cannot work across multiple transactional resources. For example, code that manages transactions using a JDBC connection cannot run within a global JTA transaction. Because the application server is not involved in transaction management, it cannot help ensure correctness across multiple resources. (It is worth noting that most applications use a single transaction resource.) Another downside is that local transactions are invasive to the programming model.

### 13.2.3 一致的事务编程模型

Spring resolves the disadvantages of global and local transactions. It enables application developers to use a consistent programming model in any environment. You write your code once, and it can benefit from different transaction management strategies in different environments. The Spring Framework provides both declarative and programmatic transaction management. Most users prefer declarative transaction management, which is recommended in most cases.

With programmatic transaction management, developers work with the Spring Framework transaction abstraction, which can run over any underlying transaction infrastructure. With the preferred declarative model, developers typically write little or no code related to transaction management, and hence do not depend on the Spring Framework transaction API, or any other transaction API.

> Do you need an application server for transaction management?
> 
> The Spring Framework’s transaction management support changes traditional rules as to when an enterprise Java application requires an application server.
> 
> In particular, you do not need an application server simply for declarative transactions through EJBs. In fact, even if your application server has powerful JTA capabilities, you may decide that the Spring Framework’s declarative transactions offer more power and a more productive programming model than EJB CMT.
> 
> Typically you need an application server’s JTA capability only if your application needs to handle transactions across multiple resources, which is not a requirement for many applications. Many high-end applications use a single, highly scalable database (such as Oracle RAC) instead. Standalone transaction managers such as Atomikos Transactions and JOTM are other options. Of course, you may need other application server capabilities such as Java Message Service (JMS) and Java EE Connector Architecture (JCA).
> 
> The Spring Framework gives you the choice of when to scale your application to a fully loaded application server. Gone are the days when the only alternative to using EJB CMT or JTA was to write code with local transactions such as those on JDBC connections, and face a hefty rework if you need that code to run within global, container-managed transactions. With the Spring Framework, only some of the bean definitions in your configuration file, rather than your code, need to change.

## 13.3 理解Spring框架如何抽象地看待一切事务

## 13.4 调度与事务有关的全部资源

## 13.5 声明式事务管理

## 13.6 编程式事务管理

## 13.7 二选其一

## 13.8 与事务绑定的应用程序事件

## 13.9 应用如何与专有服务器集成

## 13.10 常见问题解决方案

## 12.11 更多资源

