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
- 绑定事务事件一节，将会告诉您怎样在针对一个已经存在的事务，在应用程序中绑定、监听与之有关的事件

## 13.2 Spring框架事务模型的优势

一直以来，Java企业级开发者在数据库事务管理的问题上有两个选择：全局事务或者是本地事务。这两种方案都有非常严重的局限和不足。

接下来两个小节，本文将重新审视全局事务和本地事务，继之以对一个问题的简单讨论：Spring框架的事务模型如何支持您克服全局事务和本地事务的局限与不足？

### 13.2.1 全局事务

借助于全局事务，你可以同时操作多个 事务资源，最经典的资源就是关系型数据库和消息队列。应用服务器通过JTA（Java Transaction Interface）管理全局事务；归因于它的异常模型，JTA是一种非常那冗长繁杂的应用编程接口（API）。更加值得注意的是，JTA的用户事务（UserTransaction）正常情况下都来源于JNDI，也就是说，如果你打算使用JTA，那么你就必须使用JNDI。很显然，采用全局事务会大大限制你的应用程序代码被复用，因为只有在使用应用服务器环境的时候，你才能使用JTA。

在之前，使用全局事务的最优方法是借助于EJB容器支持的事务（Container Managed Transaction）：不同于编程式事务管理，CMT是一种声明式事务管理。基于EJB容器的事务管理免除了与事务相关的JNDI查找，虽说使用EJB容器本身就必须要用到JNDI查找。采用这种做法，你就可以尽量少的通过Java代码控制事务。CMT方案本身与JTA和应用服务器环境紧密关联，这是它的一个很明显的弊端。另外，只有当你在EJB（Enterprise Java Bean）里面，或者是在一个事务性的EJB门面背后实现业务逻辑, 你才有机会使用这种方案（EJB CMT）。相比于其他那些引人入胜的声明式事务管理方案，EJB CMT的巨大的负面效应使得它不再是为一个有吸引力的选项。

### 13.2.2 本地事务

本地事务和事务资源类型紧密相关，比如说与JDBC关联的事务。本地事务易于使用，但是也有明显的不足：它不能跨越多个事务资源。举个例子，关联JDBC连接的事务就不能在一个全局JTA事物里面运行。由于应用服务器没有参与事务管理，它就不能确保多个事务资源情形下应用程序的正确性（此处值得一提的是，大都多数应用只需要用到一个事务资源）。本地事务的另一个缺点是，它对于原有编程模型的的入侵性很强。


### 13.2.3 一致的事务编程模型

Spring框架可以成功化解全局事务和本地事务的不足之处。它帮助用用程序开发者在任何环境下都可以使用一致的编程模型。你只要写一次代码，就可以轻易受益于不同环境中不同的事务管理策略。Spring框架为您提供了声明式和编程式事务管理方案。多数框架用户倾向于采用编程的办法管理事务。我们也推荐大家在多数情况下这么做。

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

