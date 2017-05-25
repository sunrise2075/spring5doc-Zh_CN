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

- 超时（Timeout）：在超时或者是被底层事务基础设施自动回滚之前，当前事务可以运行多长时间？

- 只读状态（Read Only Status）： 如果你的代码只是读取数据，但是不会写入数据，那么你就会用到只读事务。在某些情形下，比如Hibernate，只读事务是一种非常有用的优化。

以上这些配置反映了一些标准的事务概念。如果有需要，您可以参考其他一些讨论事务隔离级别和核心概念的材料。理解这些概念，无论对于使用Spring框架，还是其他任何事务管理方案，都很有必要。

The TransactionStatus interface provides a simple way for transactional code to control transaction execution and query transaction status. The concepts should be familiar, as they are common to all transaction APIs:

代表事务状态的`TransactionStatus`接口为应用程序代码提供了控制事务执行和查询事务状态的简单途径。对于不同的事务管理API，这些概念都非常相似：

    public interface TransactionStatus extends SavepointManager {
    
    	boolean isNewTransaction();
    
    	boolean hasSavepoint();
    
    	void setRollbackOnly();
    
    	boolean isRollbackOnly();
    
    	void flush();
    
    	boolean isCompleted();
    
    }

Regardless of whether you opt for declarative or programmatic transaction management in Spring, defining the correct PlatformTransactionManager implementation is absolutely essential. You typically define this implementation through dependency injection.

无论选择Spring的声明式事务管理或者编程式事务管理，你绝对有必要定义一个正确的`PlatformTransactionManager`实现。你通常可以通过依赖注入来定义这个实现。

PlatformTransactionManager implementations normally require knowledge of the environment in which they work: JDBC, JTA, Hibernate, and so on. The following examples show how you can define a local PlatformTransactionManager implementation. (This example works with plain JDBC.)

定义一个事务管理器`PlatformTransactionManager`的具体实现，你需要知道与事务管理器所在工作环境有关的知识：JDBC，JTA，Hibernate， 或者其他什么。下面的例子向你展示了如何定义一个本地事务管理器的具体实现。（它可以与JDBC一起工作）。

你要先定义一个JDBC数据源：

    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    	<property name="driverClassName" value="${jdbc.driverClassName}" />
    	<property name="url" value="${jdbc.url}" />
    	<property name="username" value="${jdbc.username}" />
    	<property name="password" value="${jdbc.password}" />
    </bean>

相关的事务管理器`PlatformTransactionManager`的Java Bean定义需要引用上面的数据源定义。就像这样：

    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    	<property name="dataSource" ref="dataSource"/>
    </bean>

If you use JTA in a Java EE container then you use a container DataSource, obtained through JNDI, in conjunction with Spring’s JtaTransactionManager. This is what the JTA and JNDI lookup version would look like:

倘若选用了JavaEE容器，与一个Spring框架提供的JTA事务管理器`JtaTransactionManager`一起，你还得用到通过JNDI查找到的容器数据源。想要定义一个JNDI版本的JTA事务管理器，你应该按照下面的样子做：

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xmlns:jee="http://www.springframework.org/schema/jee"
    	xsi:schemaLocation="
    		http://www.springframework.org/schema/beans
    		http://www.springframework.org/schema/beans/spring-beans.xsd
    		http://www.springframework.org/schema/jee
    		http://www.springframework.org/schema/jee/spring-jee.xsd">
    
    	<jee:jndi-lookup id="dataSource" jndi-name="jdbc/jpetstore"/>
    
    	<bean id="txManager" class="org.springframework.transaction.jta.JtaTransactionManager" />
    
    	<!-- other <bean/> definitions here -->
    
    </beans>

上面的JTA事务管理器`JtaTransactionManager`不需要直接引用数据源或者是任何其他资源，原因是：它使用了Java企业容器的全局事务管理基础设施。

> ![[Note]](http://i.imgur.com/GDinJMk.png)上面的数据源定义用到了jee命名空间的<jndi-lookup/>标签。需要了解更多有关基于schema方案的配置文件的信息，你可以查看 38章XML：基于schema方案的XML配置。需要了解更多jee命名空间标签的信息，请查看38章里面的38.2.3.“jee方案”一小节。

参考下面的例子，你也可以轻松地使用Hibernate本地事务。此时你需要定义一个Hibernate的本地session工厂`LocalSessionFactoryBean`，你的应用代码会通过它获取Hibernate的session实例。

这里的数据源定义方法与之前展示的本地JDBC数据源相同，因此不再赘述。

> ![[Note]](http://i.imgur.com/GDinJMk.png)那些被非JTA的事务管理器使用到的，通过JNDI查找、由Java企业容器管理的数据源，必须是非事务性的。因为，Spring框架（而不是Java企业容器）将会管理这些事务。

此处，名称为`txManager`的Java Bean的类型是`HibernateTransactionManager`。与`DataSourceTransactionManager`需要指向一个数据源类似，`HibernateTransactionManager`需要指向一个`SessionFactory`（会话工厂）实例。

    <bean id="sessionFactory" class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
    	<property name="dataSource" ref="dataSource"/>
    	<property name="mappingResources">
    		<list>
    			<value>org/springframework/samples/petclinic/hibernate/petclinic.hbm.xml</value>
    		</list>
    	</property>
    	<property name="hibernateProperties">
    		<value>
    			hibernate.dialect=${hibernate.dialect}
    		</value>
    	</property>
    </bean>
    
    <bean id="txManager" class="org.springframework.orm.hibernate5.HibernateTransactionManager">
    	<property name="sessionFactory" ref="sessionFactory"/>
    </bean>

如果你选用了Hibernate和由Java企业容器管理的JTA事务管理方案，你只需如之前基于JDBC数据源的JTA事务管理器的例子代码那样，轻松地使用`JtaTransactionManager`。

    <bean id="txManager" class="org.springframework.transaction.jta.JtaTransactionManager"/>

> ![[Note]](http://i.imgur.com/GDinJMk.png)  如果选择了JTA事务，那么，无论底层你采用什么数据访问技术，事务管理器的定义看起来都没有两样。因为存在这样明显的一个事实：JTA事务是全局事务，全局事务会囊括所有的事务资源。

在以上所有的情况里，你都无需改动应用程序代码。只需要修改配置，你就可以改动事务管理方案。哪怕你在本地事务和全局事务来回切换的时候也可以这样。

## 13.4 在事务里面同步资源

你现在已经很清楚，如何创建不同的事务管理器、把它们跟那些需要被同步到事务的资源（比如说`DataSourceTransactionManager`与一个`JDBC`数据源一起，`HibernateTransactionManager`与一个`Hibernate`的`SessionFactory`会话工厂，等等，诸如此类）关联起来。这一小节将要描述，直接或者间接使用了比如JDBC、Hibernate或者JPA这类持久化API的应用程序代码，在事务管理的过程中，如何确保这些资源被恰当地创建、重用以及清理。这一小节也会讨论如何通过有关的`PlatformTransactionManager`（事务管理器）触发事务同步。

### 13.4.1 位于更高层次的事务资源同步方法

这种广为接受的做法，或者是集成了Spring框架最高层次的基于模板的持久化API，或者是把对象关系映射框架的内部API跟
明确了事务上下文的工厂Bean或是代理对象联合在一起来管理本地资源工厂。这些具备明确的事务上下文的方案，从内部处理好了资源的创建、重用、清理，（可选的）事务的资源同步，还有异常映射。因而，用户的数据访问代码无需关注此类底层任务，只需专心处理好非样板的（non-boilerplate）的数据持久化逻辑。一般的，在借助`JdbcTemplate`来完成JDBC访问的时候，你就是在使用对象关系映射框架（ORM）的本地API和基于模板的办法。本参考文档会在接下来的几个章节里面详细论述这些做法。

### 13.4.2 位于更低层次的事务资源同步方法

诸如`DataSourceUtils`（针对JDBC），`EntityManagerFactoryUtils`（针对JPA），`SessionFactoryUtils`（针对Hibernate）之类的Class类型存在于更低的抽象层面。在需要让应用程序代码直接处理原生（最底层）持久化API这种类型的资源时，你可以用这些Class类型来确定地获取到由Spring框架管理的对象实例，此时此刻，事务的资源被同步好了（如果需要），发生在这个过程的资源也会被恰当地映射到一个一致的API。

For example, in the case of JDBC, instead of the traditional JDBC approach of calling the getConnection() method on the DataSource, you instead use Spring’s org.springframework.jdbc.datasource.DataSourceUtils class as follows:
比如JDBC为例，无需按照传统的做法在数据源对象上调用`getConnection()`方法，你只要按照下面的样子使用`org.springframework.jdbc.datasource.DataSourceUtils`Class类型：

    Connection conn = DataSourceUtils.getConnection(dataSource);

倘若当前存在的事务已经与一个数据源与之关联，那么数据源对象会被返回；否则，调用此方法会触发创建一个新数据源，和当前事务关联数据源（可选）的过程；在之后的事务调用过程中这个数据源将会被重复使用。正如之前提到过的，所有发生在这里的的`SQLException`都会被包装成为Spring框架所提供的`CannotGetJdbcConnectionException`，这是一种位于由Spring框架封装的异常类型`DataAccessExceptions`的层次结构上的非受检异常。比起得到单纯的`SQLException`类型异常，这种做法不但可以为你提供更丰富的信息，而且还能够保证应用代码在不同数据库、不同的持久化技术之间的可移植性。

在没有Spring事务管理（非必须的事务资源同步）的情况下，这种做法仍然有效。所以无论是否采用Spring事务管理，你都可以使用这种做法。

当然，在引入了Spring框架的JDBC支持、JPA支持或者是Hibernate支持以后，一般来说你都不会继续倾向使用`DataSourceUtils`这样的Class类型或是其他帮助类。道理很简单，比起直接操作由Spring框架封装在底层的API，面向Spring框架抽象层会让你的工作过程更开心。

### 13.4.3 TransactionAwareDataSourceProxy

类型为`TransactionAwareDataSourceProxy`的Class类型存在于更低的框架层次上。它不但是目标数据源的代理，而且在封装目标数据源的基础上，为数据源定义了明确具体的Spring事务上文。从这个意义上说，它和Java企业容器的事务性JNDI数据源相似之处。

除非是一定要给现有的应用程序代码传入一个严格的JDBC数据源的实现，使用这个Class的做法就不只是没必要，而且也不可取。在使用了这个Class类型的情况下，也许你的代码能用，但是参与Spring框架的事务管理就成问题了。更好的做法是，你应该像之前提到过的那样，在更高的抽象层次上写自己的代码。

## 13.5 声明式事务管理

> ![[Note]](http://i.imgur.com/GDinJMk.png)大多数Spring框架的用户都会选择声明式事务管理。这样做对应用程序代码的侵入性最小，与非侵入轻量级容器的观念能够最大程度保持一致。

Spring框架的声明式事务是因为Spring的面向切面编程（AOP）功能才成为可能。事实上，由于事务切面代码已经和Spring框架一起发布，并且被以样板的风格使用，在有效的使用事务切面代码的过程中，你基本上无须理解AOP的概念。

与EJB的容器事务很相似，你可以Spring声明式事务管理中规定具体某一个方法的事务行为（当然也可以不这样做）。必要的话，你还可以在事务上下文调用`setRollbackOnly()`。以上两种类型事务的区别是：

- EJB的容器事务与JTA紧密联系。不同的是，Spring框架的声明式事务可以在任何环境下工作。后者不但可以与JTA一起工作，而且只需简单的调整配置文件，它就可以同样适用JDBC、JPA或Hibernate的本地事务。
- 你可以把Spring声明式事务应用在任何一种Class上面，而不仅仅是EJB这样特殊的Class类型。
- Spring框架提供了声明式事务的回滚规则的定义方法，EJB没有提供类似功能。无论是编程式事务还是声明式事务，Spring都支持对回滚规则的配置。
- 借助于APO，Spring框架可以让你自定义事务的行为。比如说，你可以在事务回滚的时候插入自定义行为。除了切面上事务性的Advice，你也可以在切面上增加其他任意类型Advice。对于EJB事务容器中，除了调用`setRollbackOnly()`方法，你无法干预容器的事务管理过程。
- 与高端的应用服务器不一样，在多次远程调用之间，Spring框架不支持事务上下文的传播属性。如果需要这个特性，那么我们推荐您使用EJB。在正常情况下，我们不需要一个事务跨越多次远程调用。

> `TransactionProxyFactoryBean`在哪里出现呢?
> 
> 在2.0版本以上的Spring框架当中配置声明式事务，与更早版本的Spring大有不同。最主要的区别就是：再也不需要配置`TransactionProxyFactoryBean`了。
> 
> 早于2.0版本的Spring配置方法依然有效。你只需简单地把后来的`<tx:tags/>`标签当做为你一次性定义好`TransactionProxyFactoryBean`就行了。 

回滚规则的概念很重要：由此，你可以定义在出现那种异常（exceptions，包括throwables类型对象）的情况下，现有事务将会自动回滚。你需要在配置文件里，而非代码里面声明回滚规则。这样做，你还是能够通过调用`TransactionStatus`对象上的`setRollbackOnly()`来回滚当前事务；更常见的情况是，你可以定义一个规则：当事务过程抛出`MyApplicationException`（一种由您自己扩展的非受检异常）类型异常的时候，当前事务会自动回滚。这么做的好处是，业务对象无需依赖于底层的事务管理基础设施；比如，通常不必引入Spring的事务管理API或是其它类型的SpringAPI。

虽然EJB容器的默认行为是在发生系统异常（通常是运行时异常，runtime exception）的时候自动回滚事务，但EJB容器事务在遇到应用异常（通常是除`java.rmi.RemoteException`类型以外的受检异常）的时候却不会自动回滚事务。Spring声明式事务管理的默认行为是遵从EJB的规范（也就是说，只有发生非受检异常的时候才会自动回滚事务），与此同时，定制这个行为非常在实际编程中很有用处。

### 13.5.1 理解Spring框架声明式事务的实现

想要简单地告诉你说用`@Transactional`标注自定义Class，用`@EnableTransactionManagement`标注你的配置Class类就期望你理解如何进行事务管理，这样做肯定是不行的。下面这个小节将会向你介绍Spring框架声明式事务管理基础设施的内部工作原理，以期明了一些与事务有关的问题。

需要你牢牢掌握的最重要的概念是，Spring框架对声明式事务的支持是借助于通过AOP（Aspect Oriented Programming，面向切面编程）代理实现，其中对切面事务性的建议（Advice）由元数据定义（目前主要是基于XML文件或者基于标注的元数据）。把AOP和定义事务的元数据结合在一起诞生了一个AOP代理；这个代理需要用到一个`TransactionInterceptor`类型的事务拦截器、一个合适的`PlatformTransactionManager`类型事务管理器的具体实现在多次方法调用之间驱动有关事务。

> ![[Note]](http://i.imgur.com/GDinJMk.png)“Spring AOP”在[第7章 Spring面向切面编程]()有深入讲解。

从概念上来看，调用一个事务代理的过程就像下面这样：

![tx](http://i.imgur.com/Ki82CLw.png)

### 13.5.2 声明式事务实现样例
Foo
设想有下面这样的接口以及随之而来的接口实现。这个例子采用`Foo`和`Bar`作为Class占位符，你只要集中注意力在事物的使用上，而不必去关注具体的领域模型。服务于这个样例的目的，`DefaultFooService`接口里面每一个方法体实现都会抛出一个`UnsupportedOperationException`类型的对象实例。这样你就可以看到事务被创建的过程，还有再抛出`UnsupportedOperationException`类型异常的时候事务如何自动回滚。

    // the service interface that we want to make transactional
    
    package x.y.service;
    
    public interface FooService {
    
    	Foo getFoo(String fooName);
    
    	Foo getFoo(String fooName, String barName);
    
    	void insertFoo(Foo foo);
    
    	void updateFoo(Foo foo);
    
    } 

    // an implementation of the above interface
    
    package x.y.service;
    
    public class DefaultFooService implements FooService {
    
    	public Foo getFoo(String fooName) {
    		throw new UnsupportedOperationException();
    	}
    
    	public Foo getFoo(String fooName, String barName) {
    		throw new UnsupportedOperationException();
    	}
    
    	public void insertFoo(Foo foo) {
    		throw new UnsupportedOperationException();
    	}
    
    	public void updateFoo(Foo foo) {
    		throw new UnsupportedOperationException();
    	}
    
    }

假设`FooService`接口的前两个方法，也就是`getFoo(String)`和`getFoo(String, String)`继续运行于具有只读语义的事务上下文中，并且另外两个方法，也就是`insertFoo(Foo)`和`updateFoo(Foo)`，必须运行于具有读写语义的事务上下文中。接下来的几个段落将会详细的解释如下配置文件的含义：

    <!-- from the file 'context.xml' -->
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xmlns:aop="http://www.springframework.org/schema/aop"
    	xmlns:tx="http://www.springframework.org/schema/tx"
    	xsi:schemaLocation="
    		http://www.springframework.org/schema/beans
    		http://www.springframework.org/schema/beans/spring-beans.xsd
    		http://www.springframework.org/schema/tx
    		http://www.springframework.org/schema/tx/spring-tx.xsd
    		http://www.springframework.org/schema/aop
    		http://www.springframework.org/schema/aop/spring-aop.xsd">
    
    	<!-- this is the service object that we want to make transactional -->
    	<bean id="fooService" class="x.y.service.DefaultFooService"/>
    
    	<!-- the transactional advice (what 'happens'; see the <aop:advisor/> bean below) -->
    	<tx:advice id="txAdvice" transaction-manager="txManager">
    		<!-- the transactional semantics... -->
    		<tx:attributes>
    			<!-- all methods starting with 'get' are read-only -->
    			<tx:method name="get*" read-only="true"/>
    			<!-- other methods use the default transaction settings (see below) -->
    			<tx:method name="*"/>
    		</tx:attributes>
    	</tx:advice>
    
    	<!-- ensure that the above transactional advice runs for any execution
    		of an operation defined by the FooService interface -->
    	<aop:config>
    		<aop:pointcut id="fooServiceOperation" expression="execution(* x.y.service.FooService.*(..))"/>
    		<aop:advisor advice-ref="txAdvice" pointcut-ref="fooServiceOperation"/>
    	</aop:config>
    
    	<!-- don't forget the DataSource -->
    	<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    		<property name="driverClassName" value="oracle.jdbc.driver.OracleDriver"/>
    		<property name="url" value="jdbc:oracle:thin:@rj-t42:1521:elvis"/>
    		<property name="username" value="scott"/>
    		<property name="password" value="tiger"/>
    	</bean>
    
    	<!-- similarly, don't forget the PlatformTransactionManager -->
    	<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    		<property name="dataSource" ref="dataSource"/>
    	</bean>
    
    	<!-- other <bean/> definitions here -->
    
    </beans>

检查上面的配置文件：你为自己的服务对象（名称为fooService的Bean）定义了事务支持。应用于此处的事务语义被封装于写法为`<tx:advice/>`的标签里面。这个标签的定义可以这样解读：所有的名称以get开头的方法，都将运行于一个只读事务的上下文，所有其他类型的方法遵照默认的事务语义来执行。`<tx:advice/>`标签的`transaction-manager`属性值指示了了驱动当前事务的PlatformTransactionManager事务管理器的名字，这里也就是名称为txManager的Bean。

> ![Tip](http://i.imgur.com/jvUOmJI.png)如果类型为`PlatformTransactionManager`的事务管理器名字正好是`transactionManager`，你就可以省略掉`<tx:advice/>`标签内部transaction-manager属性的定义。加入`PlatformTransactionManager`这个Bean有其他的名字，你就必须按照上面的样例说明的样子，明确地的定义transaction-manager属性值。

### 13.5.3 声明式事务的回滚

### 13.5.4 为不同的Bean配置不同的事务语义

## 13.6 编程式事务管理

## 13.7 二选其一

## 13.8 与事务绑定的应用程序事件

## 13.9 应用如何与专有服务器集成

## 13.10 常见问题解决方案

## 12.11 更多资源

.