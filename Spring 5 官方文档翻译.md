#Spring框架文档（版本：5.1.0）-- 核心技术<br/>
[原文档链接](https://docs.spring.io/spring/docs/5.1.0.RELEASE/spring-framework-reference/core.html)<br>

如有问题，欢迎指正。<br/>

说明：因部分链接内容暂时未翻译，为了防止遗漏，所以暂时将链接指向个人网站，待后续对应内容翻译完成，会进行替换。

参考文档的这一部分涵盖了Spring框架中不可或缺的所有技术。<br/>

其中最重要的是Spring框架的控制反转(IoC)容器。对Spring框架的IoC容器进行彻底的处理之后，紧接着是对Spring面向切面编程(AOP)技术的全面覆盖。Spring框架有自己的AOP框架，它在概念上很容易理解，并且成功地解决了Java企业编程中AOP需求的80%。<br/>

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

在大多数应用程序场景中，不需要使用显式用户代码来实例化Spring IoC容器的一个或多个实例。例如，在Web应用程序场景中，应用程序的web.xml文件中的简单八行（或大约八行）web模板的XML描述符通常就足够了（请参阅[Web应用程序的便捷ApplicationContext实例化](http://www.isharefy.com/)）。如果您使用Spring Tool Suite（基于Eclipse的开发环境），只需点击几下鼠标或按键即可轻松创建此样板配置。<br/>

下图显示了Spring如何工作的上层视图。你的应用程序类与配置元数据相结合，以便在创建和初始化ApplicationContext之后，你拥有一个完全配置且可执行的系统或应用程序。
![The Spring IoC container](https://raw.githubusercontent.com/ShareFY/Spring/master/images/The%20Spring%20IoC%20container.png)

#### 1、2、1 配置元数据
如上面的图所示，Spring容器使用一组配置元数据，这些配置元数据代表了作为应用开发者的你告诉Spring容器如何去进行初始化、配置和装配那些在你应用程序中用到的对象。<br/>

传统上，配置元数据以简单直观的XML格式提供，本章大部分内容使用该种方式来介绍Spring IoC容器的关键概念和功能。<br/>

```
注意：基于XML的元数据配置不是唯一被允许的配置形式。Spring IoC容器本身与配置元数据的方式完全分离。目前，许多开发人员为其Spring应用程序选择基于Java的配置。
```

想了解更多关于在Spring容器使用其他形式配置元数据的方式，可以参考如下内容：

* **基于注解的配置：** 从Spring2.5引入了对基于注解配置元数据的支持
* **基于Java的配置：** 从Spring 3.0开始，Spring JavaConfig项目提供的许多功能都成为Spring核心框架的一部分。因此，你可以使用Java在应用程序类的外部定义bean，而不使用XML文件。要使用这些新功能，请参阅[@Configuration](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Configuration.html)，[@Bean](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Bean.html)，[@Import](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Import.html)和[@DependsOn](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/DependsOn.html)注解。

Spring配置包含至少一个由容器管理的Bean定义，并且通常不止一个。基于XML方式配置元数据，通常使用<bean/>来定义一个Bean，而<bean/>节点位于顶级<beans/>节点内。基于Java方式的配置，通常使用在@Configuration注解的类中的@Bean注解方法中。<br/>

这些bean定义对应于构成应用程序的实际对象。通常，定义服务层对象、数据访问对象（DAOs）、表现层对象（如Struts Action实例），基础结构对象（如Hibernate SessionFactories，JMS队列等）。通常，不会在容器中配置细粒度域对象，因为创建和加载域对象通常由DAOs和业务逻辑负责。但是，你可以使用Spring与AspectJ的集成来配置在IoC容器控制之外创建的对象。 请参阅[Using AspectJ to dependency-inject domain objects with Spring](http://www.isharefy.com/)。<br/>

以下示例显示了基于XML方式配置元数据的基本格式：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">   
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions go here -->

</beans>
```
```
说明：
	① id属性是一个用来标识每个Bean定义的字符串。
	② class属性定义了Bean的类型并且需要使用类的完全限定名（类的完整路径）。
```

id属性的值引用协作对象。本例中没有显示引用协作对象的XML。有关更多信息，请参见[依赖关系](http://www.isharefy.com/)。

#### 1、2、2 实例化容器
提供给ApplicationContext构造函数的定位路径实际上是用来提供给容器加载配置元数据的资源字符串，它允许容器从各种外部资源（如本地文件系统，Java CLASSPATH等）加载配置元数据。

```
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
```
注意：在了解了Spring的IoC容器之后，你可能想要了解有关Spring的资源抽象的更多信息（如[参考资料](http://www.isharefy.com/)中所述），它提供了一种从URI语法中定义的位置读取InputStream的便捷机制。特别是，资源路径用于构建应用程序上下文，如[应用程序上下文和资源路径](http://www.isharefy.com/)中所述。<br/>

以下示例显示了服务层对象 **（services.xml）** 的配置文件：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- services -->

    <bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <property name="itemDao" ref="itemDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for services go here -->

</beans>
```

以下示例显示了数据访问对象 **（daos.xml）** 文件：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="accountDao"
        class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for data access objects go here -->

</beans>
```

在前面的示例中，服务层由PetStoreServiceImpl类和两个类型为JpaAccountDao和JpaItemDao的数据访问对象组成（基于JPA对象关系映射标准）。属性的name元素代表JavaBean属性的名称，ref元素引用另一个bean定义的名称。id和ref元素之间的这种联系表达了协作对象之间的依赖关系。有关配置对象依赖项的详细信息，请参阅[依赖关系](http://www.isharefy.com/)。<br/>

**编写基于XML方式配置元数据**<br/>
通过多个XML文件来进行Bean的定义通常较为有用。通常，每个单独的XML配置文件都代表你架构中的一个逻辑层或模块。<br/>

你可以使用应用程序上下文构造函数从所有这些XML片段加载bean定义。如前面章节所述，此构造函数接受多个资源定位符。或者，使用一个或多个<import/>元素来从另一个或多个文件加载bean定义。以下示例显示了如何执行此操作：

```
<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>
    <import resource="/resources/themeSource.xml"/>

    <bean id="bean1" class="..."/>
    <bean id="bean2" class="..."/>
</beans>
```

在前面的示例中，外部bean定义从三个文件加载：services.xml，messageSource.xml和themeSource.xml。 所有位置路径都与执行导入的定义文件相关，因此services.xml必须与执行导入的文件位于相同的目录或类路径位置，而messageSource.xml和themeSource.xml必须位于该位置下的资源位置 导入文件。如你所见，最前边的斜杠被忽略了。但是，鉴于这些路径是相对的，最好不要使用斜杠。根据Spring Schema，正在导入的文件的内容（包括顶层<beans/>元素）必须是有效的XML bean定义。

```
注意：
	可以（但不推荐）使用相对“../”路径引用父目录中的文件。这样做会对当前应用程序之外的文件创建依赖关系。特别是，不建议对classpath使用此引用：URL（例如，classpath：../ services.xml），其中运行时解析过程选择“最近的”类路径根，然后查看其父目录。类路径配置更改可能导致选择不同的，不正确的目录。
	你始终可以使用完全限定的资源位置而不是相对路径：例如，file：C：/config/services.xml或classpath：/config/services.xml。但是，请注意您将应用程序的配置与特定的绝对位置耦合。通常最好为这样的绝对位置保持间接联系 - 例如，通过在运行时针对JVM系统属性解析的“${...}”占位符。
```

命名空间本身提供了导入指令功能。在Spring提供的XML名称空间(例如context和util命名空间)中，除了普通bean定义之外，还有其他配置特性。<br/>

**Groovy Bean定义DSL**<br/>
作为外部化配置元数据的另一个示例，bean定义也可以在Spring的Groovy Bean定义DSL中表示，如我们所知道的Grails框架。通常，此类配置位于“.groovy”文件中，其结构如下例所示：

```
beans {
    dataSource(BasicDataSource) {
        driverClassName = "org.hsqldb.jdbcDriver"
        url = "jdbc:hsqldb:mem:grailsDB"
        username = "sa"
        password = ""
        settings = [mynew:"setting"]
    }
    sessionFactory(SessionFactory) {
        dataSource = dataSource
    }
    myService(MyService) {
        nestedBean = { AnotherBean bean ->
            dataSource = dataSource
        }
    }
}
```

此配置样式在很大程度上等同于XML bean定义，甚至支持Spring的XML配置命名空间。它还允许通过importBeans指令导入XML bean定义文件。































<span id="beanfactory"></span>















































