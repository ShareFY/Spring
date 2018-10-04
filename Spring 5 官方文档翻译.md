#Spring框架文档（版本：5.1.0）-- 核心技术
[原文档链接](https://docs.spring.io/spring/docs/5.1.0.RELEASE/spring-framework-reference/core.html)<br>

如有问题，欢迎指正。

参考文档的这一部分涵盖了Spring框架中不可或缺的所有技术。<br/>

其中最重要的是Spring框架的控制反转(IoC)容器。对Spring框架的IoC容器进行彻底的处理之后，紧接着是对Spring面向方面编程(AOP)技术的全面覆盖。Spring框架有自己的AOP框架，它在概念上很容易理解，并且成功地解决了Java企业编程中AOP需求的80%优点。<br/>

本文还介绍了Spring与AspectJ的集成(就特性而言，目前是最丰富的，当然也是Java企业空间中最成熟的AOP实现)。<br/>

## 1、IoC容器
本章介绍Spring的反转控制(IoC)容器。<br/>

### 1、1 Spring IoC容器和Beans介绍
本章介绍了控制反转(IoC)原理的Spring框架实现。(参见 [IoC介绍](https://docs.spring.io/spring/docs/5.1.0.RELEASE/spring-framework-reference/overview.html#background-ioc) )IoC也称为依赖注入(DI)。在这个过程中，对象只通过构造函数参数、工厂方法的参数或对象实例在构造或从工厂方法返回后设置的属性来定义它们的依赖关系(也就是说，它们使用的其他对象)。这个过程基本上是bean本身通过使用类的直接构造或诸如服务定位器模式之类的机制来控制其依赖关系的实例化或依赖位置的反转。<br/>

org.springframework.beans 和 org.springframework.context 这两个包是Spring Framework的IoC容器的基础。[BeanFactory](https://docs.spring.io/spring-framework/docs/5.1.0.RELEASE/javadoc-api/org/springframework/beans/factory/BeanFactory.html)接口提供了一种能够管理任何类型对象的高级配置机制。[ApplicationContext](https://docs.spring.io/spring-framework/docs/5.1.0.RELEASE/javadoc-api/org/springframework/context/ApplicationContext.html)是BeanFactory的子接口，它添加了如下内容：

```
* 更容易与Spring的AOP功能集成
* 消息资源处理（用于国际化）
* 事件发布
* 特定于应用程序层的上下文，例如用于Web应用程序的WebApplicationContext。

```

简而言之，BeanFactory提供配置框架和基本功能，ApplicationContext添加了更多特定于企业的功能。 ApplicationContext是BeanFactory的完整超集，在本章中仅用于Spring的IoC容器的描述。有关使用BeanFactory而不是ApplicationContext的更多信息，请参阅 [BeanFactory介绍](#beanfactory)。<br/>

在Spring中，构成应用程序主干并由Spring IoC容器管理的对象称为bean。Bean是一个由Spring IoC容器实例化，组装和管理的对象。 否则，bean只是应用程序中众多对象之一。 Bean及其之间的依赖关系反映在容器使用的配置元数据中。

### 1、2 容器概述
org.springframework.context.ApplicationContext接口表示Spring IoC容器，负责实例化，配置和组装bean。容器通过读取配置元数据获取有关要实例化，配置和组装对象的指令。通过XML，Java注解或Java代码进行元数据的配置。它允许你表达组成应用程序的对象以及这些对象之间丰富的相互依赖性。<br/>

Spring提供了ApplicationContext接口的几个实现。在独立应用程序中，通常会创建ClassPathXmlApplicationContext或FileSystemXmlApplicationContext的实例。 虽然XML是定义配置元数据的传统格式，但您可以通过提供少量XML配置来声明容器使用Java注释或代码作为元数据格式，以声明方式启用对这些其他元数据格式的支持。<br/>

在大多数应用程序场景中，不需要使用显式用户代码来实例化Spring IoC容器的一个或多个实例。例如，在Web应用程序场景中，应用程序的web.xml文件中的简单八行（或大约八行）web模板的XML描述符通常就足够了（请参阅[Web应用程序的便捷ApplicationContext实例化](http://www.baidu.com)）。如果您使用Spring Tool Suite（基于Eclipse的开发环境），只需点击几下鼠标或按键即可轻松创建此样板配置。<br/>

下图显示了Spring如何工作的上层视图。你的应用程序类与配置元数据相结合，以便在创建和初始化ApplicationContext之后，你拥有一个完全配置且可执行的系统或应用程序。
![The Spring IoC container](https://raw.githubusercontent.com/ShareFY/Spring/master/images/The%20Spring%20IoC%20container.png)

#### 1、2、1 配置元数据
如上面的图所示，Spring容器使用一组配置元数据，这些配置元数据代表了作为应用开发者的你告诉Spring容器如何去进行初始化、配置和装配那些在你应用程序中用到的对象。<br/>

传统上，配置元数据以简单直观的XML格式提供，本章大部分内容使用该种方式来介绍Spring IoC容器的关键概念和功能。<br/>

```
注意：基于XML的元数据配置不是唯一被允许的配置形式。Spring IoC容器本身与配置元数据的方式完全分离。目前，许多开发人员为其Spring应用程序选择基于Java的配置。
```

想了解更多关于在Spring容器使用其他形式配置元数据的方式，可以参考如下内容：

* **基于注解的配置：**从Spring2.5引入了对基于注解配置元数据的支持
* **基于Java的配置：** 从Spring 3.0开始，Spring JavaConfig项目提供的许多功能都成为Spring核心框架的一部分。因此，你可以使用Java在应用程序类的外部定义bean，而不使用XML文件。要使用这些新功能，请参阅[@Configuration](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Configuration.html)，[@Bean](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Bean.html)，[@Import](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Import.html)和[@DependsOn](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/DependsOn.html)注解。

Spring配置包含至少一个由容器管理的Bean定义，并且通常不止一个。基于XML方式配置元数据，通常使用<bean/>来定义一个Bean，而<bean/>节点位于顶级<beans/>节点内。基于Java方式的配置，通常使用在@Configuration注解的类中的@Bean注解方法中。<br/>

这些bean定义对应于构成应用程序的实际对象。通常，定义服务层对象、数据访问对象（DAOs）、表现层对象（如Struts Action实例），基础结构对象（如Hibernate SessionFactories，JMS队列等）。通常，不会在容器中配置细粒度域对象，因为创建和加载域对象通常由DAOs和业务逻辑负责。但是，你可以使用Spring与AspectJ的集成来配置在IoC容器控制之外创建的对象。 请参阅[Using AspectJ to dependency-inject domain objects with Spring](http://www.baidu.com)。





















<span id="beanfactory"></span>















































