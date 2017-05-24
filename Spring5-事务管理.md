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

Spring框架可以成功化解全局事务和本地事务的不足之处。它帮助应用程序开发者在任何环境下都使用一致的编程模型。你只要写一次代码，就可以轻易受益于不同环境中不同的事务管理策略。Spring框架为您提供了声明式和编程式事务管理方案。多数框架用户倾向于采用编程的办法管理事务。我们也推荐大家在多数情况下这么做。

对于编程式事务管理，开发人员要用到**Spring框架的事务抽象层**————它运行于任何一种底层的事务基础框架之上。作为一种备受偏爱的声明模型，（在使用过程中）开发者只需要很少、甚至完全无需编写与事务管理有关的代码。因而他们也无需依赖Spring框架的事务编程接口（API），和其他的任何一种事务编程接口API。

> 你需要一个应用服务器进行事务管理吗？
> 
> Spring框架对事务管理的支持改变了一件由来已久的事情：究竟何时，一个Java应用程序必须要在应用服务器（Application Server）上面运行？
> 
> 特别值得指出，如果只是想要通过EJB实现声明式事务，你完全无须应用服务器。实际上，就算你的应用服务器已经具备强大的JTA能力，你仍旧有机会选择Spring框架的声明式事务管理（因为，比起基于EJB容器的事务，这是一种更加强大和高效的编程模型）
> 
> 在最一般的情况下，只有需要实现跨越多个资源事务管理时，你才真正需要用到应用服务器的JTA支持。然而，大多数应用并没有这个需求。相反，许多高端的应用只需用到单一的、高度可扩展的数据库（例如Oracle RAC）。像Atomikos和JOTM之类独立的事务管理器也是可选方案。当然，我们不排除下面这样的可能：你还需要用到应用服务器的其他能力，比如Java消息服务（Java Message Service，JMS）或者Java企业连接器结构（Java EE Connector Architecture，JCA）。
> 
> Spring框架可以给您提供选择：什么时候把自己的应用扩展到一个满负荷运行的应用服务器上。The Spring Framework gives you the choice of when to scale your application to a fully loaded application server. 在过去，你只能基于EJB的容器事务或者JTA来编写JDBC本地事务；想要要在全局事务或者EJB容器事务中运行应用，你就必须重新修改代码。Gone are the days when the only alternative to using EJB CMT or JTA was to write code with local transactions such as those on JDBC connections, and face a hefty rework if you need that code to run within global, container-managed transactions.有了Spring框架，你只需要在配置文件里面修改Java bean的定义。 With the Spring Framework, only some of the bean definitions in your configuration file, rather than your code, need to change.

## 13.3 理解Spring框架如何抽象地看待一切事务

理解Spring框架事务管理抽象层的关键是：领会**事务策略**的概念。在Spring框架中，事务策略由名称是`org.springframework.transaction.PlatformTransactionManager`的Java接口定义：

    public interface PlatformTransactionManager {
    
    	TransactionStatus getTransaction(
    			TransactionDefinition definition) throws TransactionException;
    
    	void commit(TransactionStatus status) throws TransactionException;
    
    	void rollback(TransactionStatus status) throws TransactionException;
    }

虽说你可以在自己的应用代码编程过程中使用它，但它的作用主要是一个对外提供事务服务的接口（service provider interface，SPI）。正因为是接口，你可以轻松地（当然是必要的时候）为它制作模拟对象和测试桩。它没有跟诸如JNDI这样的查找策略策略邦定。在Spring的依赖注入（Ioc）容器中，`PlatformTransactionManager`的具体实现就像其他任何对象、Java bean一样被定义。就这一点，即使在采用JTA作为持久化方案的情况下，也能够保证Spring框架的事务管理成为一个有价值的抽象层，。比起你直接用到JTA，这样做（使用Spring框架）也会让你的代码更容易测试。

为了与Spring框架的哲学保持一致，由PlatformTransactionManager接口中每一个方法抛出的异常TransactionException是一种非受检异常（unchecked exception）。换句话说，它扩展了Java基础类库的Class：`java.lang.RuntimeException`。法正在在事务管理的基础设施框架中的失败无一例外都是严重的错误。在极少出现的情况下，应用程序代码可以从事务失败中恢复，应用程序的开发者可以选择捕获并且处理此类事务异常`TransactionException`。重要的一点是，开发者不会被强制要求这么做。

依赖于一个`TransactionDefinition`类型参数，`getTransaction(..)`方法会返回一个代表事务状态的`TransactionStatus`类型对象。这里的事务状态对象，通常代表一个新的事物，或是一个已经存在的事务（条件是：在现有的调用栈里面存在一个与传入参数匹配的事务定义）。这里所说的第二种情况的具体含义是，在Java EE的事务上下文里，表示事物状态的`TransactionStatus`对象跟一个正在运行事务的程序线程紧密关联。

`TransactionDefinition`接口定义了下面的几项内容：

- 隔离级别（Isolation）：当前事务与其他事务所从事的工作的隔离程度。比如说，现在这个事务能否看到其他事务尚未提交的写入？

- 传播属性（Propagation）： 通常来说，在一个事务作用域里运行的所有代码只会在这个事务内部产生效果。然而，假如已经存在一个事务上下文，你可以通过隔离级别这个选项定义当前事务方法的具体行为。比如说，常见的情况下，事务方法的代码可以继续运行于当前这个已经存在的事务上下文；或者是，中止这个事务上下文，重新创建一个事务。Spring框架为您提供了与EJB容器事务（EJB CMT， container managed transaction）一样的全部隔离级别选项。需要了解Spring框架事务隔离级别语义的朋友，可以参考 13.5.7  “Transaction propagation”



## 13.4 调度与事务有关的全部资源

## 13.5 声明式事务管理

## 13.6 编程式事务管理

## 13.7 二选其一

## 13.8 与事务绑定的应用程序事件

## 13.9 应用如何与专有服务器集成

## 13.10 常见问题解决方案

## 12.11 更多资源

