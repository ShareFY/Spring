Spring框架文档（版本：5.1.0）-- 核心技术
========================================

[原文档链接](https://docs.spring.io/spring/docs/5.1.0.RELEASE/spring-framework-reference/core.html)<br>

如有问题，欢迎指正。<br/>

说明：因部分链接内容暂时未翻译，为了防止遗漏，所以暂时将链接指向个人网站，待后续对应内容翻译完成，会进行替换。

参考文档的这一部分涵盖了Spring框架中不可或缺的所有技术。<br/>

其中最重要的是Spring框架的控制反转(IoC)容器。对Spring框架的IoC容器进行彻底的处理之后，紧接着是对Spring面向切面编程(AOP)技术的全面覆盖。Spring框架有自己的AOP框架，它在概念上很容易理解，并且成功地解决了Java企业编程中AOP需求的80%。<br/>

本文还介绍了Spring与AspectJ的集成(就特性而言，目前是最丰富的，当然也是Java企业空间中最成熟的AOP实现)。<br/>

目录
---------
-   [1、IoC容器](#ioc容器)
    -   [1、1 Spring IoC容器和Beans介绍](#spring-ioc容器和beans介绍)
    -   [1、2 容器概述](#容器概述)
        -   [1、2、1 配置元数据](#配置元数据)
        -   [1、2、2 实例化容器](#实例化容器)
        -   [1、2、3 使用容器](#使用容器)
    -   [1、3 Bean概述](#beanOverview)
        -   [1、3、1 命名Beans](#命名beans) 
        -   [1、3、2 实例化Beans](#实例化beans)
    -   [1、4 依赖关系](#依赖关系)
    	 -   [1、4、1 依赖注入](#依赖注入) 
    	 -   [1、4、2 依赖关系和配置的详细介绍](#依赖关系和配置的详细介绍)  


<a id="ioc容器"></a>
1、IoC容器
----------

本章介绍Spring的反转控制(IoC)容器。<br/>

<a id="spring-ioc容器和beans介绍"></a>
### 1、1 Spring IoC容器和Beans介绍

本章介绍了控制反转(IoC)原理的Spring框架实现。(参见
[IoC介绍](https://docs.spring.io/spring/docs/5.1.0.RELEASE/spring-framework-reference/overview.html#background-ioc)
)IoC也称为依赖注入(DI)。在这个过程中，对象只通过构造函数参数、工厂方法的参数或对象实例在构造或从工厂方法返回后设置的属性来定义它们的依赖关系(也就是说，它们使用的其他对象)。这个过程基本上是bean本身通过使用类的直接构造或诸如服务定位器模式之类的机制来控制其依赖关系的实例化或依赖位置的反转。<br/>

org.springframework.beans 和 org.springframework.context
这两个包是Spring
Framework的IoC容器的基础。[BeanFactory](https://docs.spring.io/spring-framework/docs/5.1.0.RELEASE/javadoc-api/org/springframework/beans/factory/BeanFactory.html)接口提供了一种能够管理任何类型对象的高级配置机制。[ApplicationContext](https://docs.spring.io/spring-framework/docs/5.1.0.RELEASE/javadoc-api/org/springframework/context/ApplicationContext.html)是BeanFactory的子接口，它添加了如下内容：

    * 更容易与Spring的AOP功能集成
    * 消息资源处理（用于国际化）
    * 事件发布
    * 特定于应用程序层的上下文，例如用于Web应用程序的WebApplicationContext。

简而言之，BeanFactory提供配置框架和基本功能，ApplicationContext添加了更多特定于企业的功能。
ApplicationContext是BeanFactory的完整超集，在本章中仅用于Spring的IoC容器的描述。有关使用BeanFactory而不是ApplicationContext的更多信息，请参阅
[BeanFactory介绍](#beanfactory)。<br/>

在Spring中，构成应用程序主干并由Spring
IoC容器管理的对象称为bean。Bean是一个由Spring
IoC容器实例化，组装和管理的对象。 否则，bean只是应用程序中众多对象之一。
Bean及其之间的依赖关系反映在容器使用的配置元数据中。

<a id="容器概述"></a>
### 1、2 容器概述

org.springframework.context.ApplicationContext接口表示Spring
IoC容器，负责实例化，配置和组装bean。容器通过读取配置元数据获取有关要实例化，配置和组装对象的指令。通过XML，Java注解或Java代码进行元数据的配置。它允许你表达组成应用程序的对象以及这些对象之间丰富的相互依赖性。<br/>

Spring提供了ApplicationContext接口的几个实现。在独立应用程序中，通常会创建ClassPathXmlApplicationContext或FileSystemXmlApplicationContext的实例。
虽然XML是定义配置元数据的传统格式，但您可以通过提供少量XML配置来声明容器使用Java注释或代码作为元数据格式，以声明方式启用对这些其他元数据格式的支持。<br/>

在大多数应用程序场景中，不需要使用显式用户代码来实例化Spring
IoC容器的一个或多个实例。例如，在Web应用程序场景中，应用程序的web.xml文件中的简单八行（或大约八行）web模板的XML描述符通常就足够了（请参阅[Web应用程序的便捷ApplicationContext实例化](http://www.isharefy.com/)）。如果您使用Spring
Tool
Suite（基于Eclipse的开发环境），只需点击几下鼠标或按键即可轻松创建此样板配置。<br/>

下图显示了Spring如何工作的上层视图。你的应用程序类与配置元数据相结合，以便在创建和初始化ApplicationContext之后，你拥有一个完全配置且可执行的系统或应用程序。
![The Spring IoC
container](https://raw.githubusercontent.com/ShareFY/Spring/master/images/The%20Spring%20IoC%20container.png)

<a id="配置元数据"></a>
#### 1、2、1 配置元数据

如上面的图所示，Spring容器使用一组配置元数据，这些配置元数据代表了作为应用开发者的你告诉Spring容器如何去进行初始化、配置和装配那些在你应用程序中用到的对象。<br/>

传统上，配置元数据以简单直观的XML格式提供，本章大部分内容使用该种方式来介绍Spring
IoC容器的关键概念和功能。<br/>

    注意：基于XML的元数据配置不是唯一被允许的配置形式。Spring IoC容器本身与配置元数据的方式完全分离。
    目前，许多开发人员为其Spring应用程序选择基于Java的配置。

想了解更多关于在Spring容器使用其他形式配置元数据的方式，可以参考如下内容：

-   **基于注解的配置：** 从Spring2.5引入了对基于注解配置元数据的支持
-   **基于Java的配置：** 从Spring 3.0开始，Spring
    JavaConfig项目提供的许多功能都成为Spring核心框架的一部分。因此，你可以使用Java在应用程序类的外部定义bean，而不使用XML文件。要使用这些新功能，请参阅[@Configuration](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Configuration.html)，[@Bean](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Bean.html)，[@Import](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Import.html)和[@DependsOn](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/DependsOn.html)注解。

Spring配置包含至少一个由容器管理的Bean定义，并且通常不止一个。基于XML方式配置元数据，通常使用```<bean/>```来定义一个Bean，而```<bean/>```节点位于顶级```<beans/>```节点内。基于Java方式的配置，通常使用在@Configuration注解的类中的@Bean注解方法中。<br/>

这些bean定义对应于构成应用程序的实际对象。通常，定义服务层对象、数据访问对象（DAOs）、表现层对象（如Struts
Action实例），基础结构对象（如Hibernate
SessionFactories，JMS队列等）。通常，不会在容器中配置细粒度域对象，因为创建和加载域对象通常由DAOs和业务逻辑负责。但是，你可以使用Spring与AspectJ的集成来配置在IoC容器控制之外创建的对象。
请参阅[Using AspectJ to dependency-inject domain objects with
Spring](http://www.isharefy.com/)。<br/>

以下示例显示了基于XML方式配置元数据的基本格式：

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

    说明：
        ① id属性是一个用来标识每个Bean定义的字符串。
        ② class属性定义了Bean的类型并且需要使用类的完全限定名（类的完整路径）。

id属性的值引用协作对象。本例中没有显示引用协作对象的XML。有关更多信息，请参见[依赖关系](http://www.isharefy.com/)。

<a id="实例化容器"></a>
#### 1、2、2 实例化容器

提供给ApplicationContext构造函数的定位路径实际上是用来提供给容器加载配置元数据的资源字符串，它允许容器从各种外部资源（如本地文件系统，Java
CLASSPATH等）加载配置元数据。

    ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

注意：在了解了Spring的IoC容器之后，你可能想要了解有关Spring的资源抽象的更多信息（如[参考资料](http://www.isharefy.com/)中所述），它提供了一种从URI语法中定义的位置读取InputStream的便捷机制。特别是，资源路径用于构建应用程序上下文，如[应用程序上下文和资源路径](http://www.isharefy.com/)中所述。<br/>

以下示例显示了服务层对象 **（services.xml）** 的配置文件：

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

以下示例显示了数据访问对象 **（daos.xml）** 文件：

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

在前面的示例中，服务层由PetStoreServiceImpl类和两个类型为JpaAccountDao和JpaItemDao的数据访问对象组成（基于JPA对象关系映射标准）。属性的name元素代表JavaBean属性的名称，ref元素引用另一个bean定义的名称。id和ref元素之间的这种联系表达了协作对象之间的依赖关系。有关配置对象依赖项的详细信息，请参阅[依赖关系](http://www.isharefy.com/)。<br/>

**编写基于XML方式配置元数据**<br/>
通过多个XML文件来进行Bean的定义通常较为有用。通常，每个单独的XML配置文件都代表你架构中的一个逻辑层或模块。<br/>

你可以使用应用程序上下文构造函数从所有这些XML片段加载bean定义。如前面章节所述，此构造函数接受多个资源定位符。或者，使用一个或多个```<import/>```元素来从另一个或多个文件加载bean定义。以下示例显示了如何执行此操作：

    <beans>
        <import resource="services.xml"/>
        <import resource="resources/messageSource.xml"/>
        <import resource="/resources/themeSource.xml"/>

        <bean id="bean1" class="..."/>
        <bean id="bean2" class="..."/>
    </beans>

在前面的示例中，外部bean定义从三个文件加载：services.xml，messageSource.xml和themeSource.xml。
所有位置路径都与执行导入的定义文件相关，因此services.xml必须与执行导入的文件位于相同的目录或类路径位置，而messageSource.xml和themeSource.xml必须位于该位置下的资源位置
导入文件。如你所见，最前边的斜杠被忽略了。但是，鉴于这些路径是相对的，最好不要使用斜杠。根据Spring
Schema，正在导入的文件的内容（包括顶层```<beans/>```元素）必须是有效的XML
bean定义。

    注意：
        可以（但不推荐）使用相对“../”路径引用父目录中的文件。这样做会对当前应用程序之外的文件创建依赖关系。
        特别是，不建议对classpath使用此引用：URL（例如，classpath：../ services.xml），
        其中运行时解析过程选择“最近的”类路径根，然后查看其父目录。
        类路径配置更改可能导致选择不同的，不正确的目录。
        你始终可以使用完全限定的资源位置而不是相对路径：例如，file：C：/config/services.xml或classpath：/config/services.xml。
        但是，请注意您将应用程序的配置与特定的绝对位置耦合。通常最好为这样的绝对位置保持间接联系 - 例如，通过在运行时针对JVM系统属性解析的“${...}”占位符。

命名空间本身提供了导入指令功能。在Spring提供的XML名称空间(例如context和util命名空间)中，除了普通bean定义之外，还有其他配置特性。<br/>

**Groovy Bean定义DSL**<br/>
作为外部化配置元数据的另一个示例，bean定义也可以在Spring的Groovy
Bean定义DSL中表示，如我们所知道的Grails框架。通常，此类配置位于".groovy"文件中，其结构如下例所示：

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

此配置样式在很大程度上等同于XML
bean定义，甚至支持Spring的XML配置命名空间。它还允许通过importBeans指令导入XML
bean定义文件。

<a id="使用容器"></a>
#### 1、2、3 使用容器

ApplicationContext是高级工厂的接口，能够维护不同Beans对象的注册以及他们之间的依赖关系。通过使用方法
T getBean(String name, Class<T>
requiredType)，您可以获取到Beans的实例。<br/>

您可以使用ApplicationContext来获取beans的定义并访问它们，如以下示例所示：

    // 创建并配置beans
    ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

    // 获取配置实例
    PetStoreService service = context.getBean("petStore", PetStoreService.class);

    // 使用配置实例
    List<String> userList = service.getUsernameList();

使用Groovy配置，bootstrapping看起来非常相似。它有一个不同的上下文实现类Groovy-aware（但也能识别XML
bean的定义）。以下示例显示了Groovy配置：

    ApplicationContext context = new GenericGroovyApplicationContext("services.groovy", "daos.groovy");

最灵活的变体是GenericApplicationContext与reader delegates相结合 -
例如，使用XML文件的XmlBeanDefinitionReader，如以下示例所示：

    GenericApplicationContext context = new GenericApplicationContext();
    new XmlBeanDefinitionReader(context).loadBeanDefinitions("services.xml", "daos.xml");
    context.refresh();

您还可以将GroovyBeanDefinitionReader用于Groovy，如以下示例所示：

    GenericApplicationContext context = new GenericApplicationContext();
    new GroovyBeanDefinitionReader(context).loadBeanDefinitions("services.groovy", "daos.groovy");
    context.refresh();

您可以在同一个ApplicationContext中混合和匹配这些reader
delegates，从不同的配置源中读取bean的定义。<br/>

然后，您可以使用getBean来获取Bean的实例。ApplicationContext接口有一些其他方法可以获取Beans，但理想情况下，您的应用程序代码永远不应该使用它们。实际上，您的应用程序代码根本不应该调用getBean()方法，因此根本不用依赖于Spring
APIs。例如，Spring与Web框架的集成为各种Web框架组件（如控制器和JSF托管bean）提供依赖注入，允许您通过元数据（例如自动装配注释）声明对特定bean的依赖性。

<a id="beanOverview"></a>
### 1、3 Bean概述

Spring
IoC容器管理一个或多个Beans,这些Beans是通过您提供给容器的配置元数据创建的（例如，用XML的```<bean/>```格式定义的）。<br/>

在容器内部，这些bean定义表示为BeanDefinition对象，其中包含（以及其他信息）以下元数据：

    * 包限定类名:通常是定义的bean的实际实现类。
    * Bean行为配置元素，它说明bean在容器中的行为方式（范围，生命周期回调函数等）
    * 引用bean执行其工作时所需的其他bean，这些引用也称为协作者或依赖项
    * 要在新创建的对象中设置的其他配置设置 - 例如，连接池的大小限制或在管理连接池的Bean中使用的连接数。

这些元数据转换为组成每个bean定义的一组属性。下表描述了这些属性：

*表1. bean定义*
<table>
<tr>
<th width="20%," bgcolor="yellow">
参数
</th>
<th width="30%," bgcolor="yellow">
详细解释
</th>
</tr>
<tr>
<td>
Class
</td>
<td>
[实例化Beans](#实例化beans)
</td>
</tr>
<tr>
<td>
Name
</td>
<td>
[命名Beans](#命名beans)
</td>
</tr>
<tr>
<td>
Scope
</td>
<td>
Beans范围
</td>
</tr>
<tr>
<td>
Constructor arguments
</td>
<td>
[依赖注入](#依赖注入)
</td>
</tr>
<tr>
<td>
Properties
</td>
<td>
[依赖注入](#依赖注入)
</td>
</tr>
<tr>
<td>
Autowiring mode
</td>
<td>
自动化装配
</td>
</tr>
<tr>
<td>
Lazy initialization mode
</td>
<td>
懒加载Beans
</td>
</tr>
<tr>
<td>
Initialization method
</td>
<td>
回调函数初始化
</td>
</tr>
<tr>
<td>
Destruction method
</td>
<td>
回调函数的销毁
</td>
</tr>
</table>
除了Bean定义包含有关如何创建特定bean的信息之外，ApplicationContext的实现也允许注册在容器外部（由用户）创建的现有对象。这是通过getBeanFactory()方法访问ApplicationContext的BeanFactory来完成的，该方法返回BeanFactory
DefaultListableBeanFactory的实现。DefaultListableBeanFactory通过registerSingleton(..)和registerBeanDefinition(..)方法来支持此注册。但是，典型应用程序仅适用于通过元数据bean的定义来定义bean对象。

    注意：需要尽早注册Bean元数据和手动提供的单例实例，以便容器在自动装配和其他内省步骤期间正确推断它们。 
    虽然在某种程度上支持覆盖现有元数据和现有单例实例，但是在运行时注册新bean（与对工厂的实时访问同时进行）并未得到官方支持，
    并且可能导致并发访问异常、bean容器中的状态不一致，或者两者都有。

<a id="命名beans"></a>
#### 1、3、1 命名Beans

每个bean都有一个或多个标识符，这些标识符在承载bean的容器中必须是唯一的。bean通常只有一个标识符，但是，如果它需要多个，则额外的可以被视为别名。<br/>

在基于XML的配置元数据中，您可以使用id属性、name属性或两个属性一起使用来指定bean标识符。id属性允许您指定一个id，通常，这些名称是字母数字（'myBean'，'someService'等），但它们也可以包含特殊字符。如果要为bean引入其他别名，还可以在name属性中指定它们，使用逗号(,)，分号(;)或空格分隔。在历史记录中，在Spring
3.1之前的版本中，id属性被定义为xsd:ID类型，它约束了可能的字符。从3.1开始，它被定义为xsd:string类型。
请注意，bean的id唯一性仍然由容器强制执行，但不再由XML解析器强制执行了。<br/>

您不需要为bean指定名称或ID标识，如果没有明确指定名称或id标识，则容器会为该Bean生成一个唯一的名称。但是，如果要通过名称引用该bean，通过使用ref元素或Service
Locator样式查找，则必须提供名称。不提供名称的动机与使用[内部beans](#内部Beans)和[自动装配协作](http://www.isharefy.com)有关。<br/>

    Bean命名约定：
        惯例是在命名bean时使用标准Java约定来实例字段名称。也就是说，bean名称以小写字母开头，并且遵从驼峰命名规则。
        此类名称的示例包括accountManager，accountService，userDao，loginController等。
        一致的bean命名规则可以使您的配置更容易阅读和理解。
        另外，如果您使用Spring AOP，将建议方式应用于名称相关的一组bean时，它会有很大帮助。

    注意：通过类路径中的组件扫描，Spring按照前面描述的规则为未命名的组件生成bean名称：基本上，采用简单的类名并将其初始字符转换为小写。
    但是，在（不常见的）特殊情况下，当有多个字符并且第一个和第二个字符都是大写字母时，原始情况将被保留。
    这些规则与java.beans.Introspector.decapitalize（Spring在此处使用）中定义的规则相同。

**在Bean定义之外的Bean别名** <br/>

在bean定义本身中，您可以为bean提供多个名称，方法是使用id属性指定的最多一个名称和name属性中的任意数量的其他名称。这些名称可以是同一个bean的等效别名，并且在某些情况下非常有用，例如让应用程序中的每个组件通过使用特定于该组件本身的bean名称来引用公共依赖项。<br/>

但是，指定实际定义的bean的所有别名并不总是足够的。有时需要为其他地方定义的bean引入别名。在大型系统中通常就是这种情况，其中配置在每个子系统之间分配，每个子系统具有其自己的一组对象定义。在基于XML的配置元数据中，您可以使用```<alias/>```元素来完成此任务。
以下示例显示了如何执行此操作：

    <alias name="fromName" alias="toName"/>

在这种情况下，在使用此别名定义之后，名为fromName的bean（在同一容器中）也可以称为toName。<br/>

例如，子系统A的配置元数据可以通过subsystemA-dataSource的名称来引用DataSource。子系统B的配置元数据可以通过subsystemB-dataSource的名称来引用DataSource。在编写使用这两个子系统的主应用程序时，主应用程序通过myApp-dataSource的名称引用DataSource。
要使所有三个名称引用同一对象，可以将以下别名定义添加到配置元数据中：

    <alias name="subsystemA-dataSource" alias="subsystemB-dataSource"/>
    <alias name="subsystemA-dataSource" alias="myApp-dataSource" />

现在，每个组件和主应用程序都可以通过一个唯一的名称引用dataSource，并保证不与任何其他定义冲突（有效地创建命名空间），但它们引用相同的bean。<br/>

Java配置:<br/>
如果使用Java配置，则\@Bean注解可用于提供别名。有关详细信息，请参阅[使用@Bean注解](http://www.isharefy.com)。

<a id="实例化beans"></a>
#### 1、3、2 实例化Beans

bean定义本质上是用于创建一个或多个对象的。容器在被询问时查看命名bean的配方，并使用由该bean定义封装的配置元数据来创建（或获取）实际对象。<br/>

如果使用基于XML的配置元数据，则指定要在```<bean />```元素的class属性中实例化的对象的类型（或类）。
此类属性（在内部，是BeanDefinition实例上的Class属性）通常是必需的。
（有关例外，请参阅使用实例工厂方法和Bean定义继承进行实例化。）您可以使用以下两种方法之一来使用Class属性：

-   通常，在容器本身通过反向调用其构造函数直接创建bean的情况下指定要构造的bean类，基本等同于使用new运算符的Java代码。
-   要指定包含为创建对象而调用的静态工厂方法的实际类，在不太常见的情况下，容器在类上调用静态工厂方法来创建bean。从静态工厂方法的调用返回的对象类型可以完全是同一个类或另一个类。

<!-- -->
    内部类名
    如果要为静态嵌套类配置bean定义，则必须使用嵌套类的二进制名称。
    例如，如果在com.example包中有一个名为SomeThing的类，并且此SomeThing类具有一个名为OtherThing的静态嵌套类，
    	则bean定义上的class属性值将为com.example.SomeThing$OtherThing。
    请注意，在名称中使用$字符可以将嵌套类名与外部类名分开。


**使用构造函数实例化** <br/>
当您通过构造方法创建bean时，所有普通类都可以使用并且与Spring兼容。也就是说，正在开发的类不需要实现任何特定接口或者使用特定的编码方式。简单地指定bean类就足够了。但是，根据您使用特定bean的IoC类型不同，您可能需要一个默认（空）构造函数。<br/>

Spring IoC容器几乎可以管理您想让它管理的任何类。它不仅限于管理真正的JavaBeans。大多数Spring用户更喜欢实际的JavaBeans，在容器中它仅有一个默认（无参数的）构造函数，并且在属性之后有合适的setters和getters方法。在容器中您也可以有更多外来的非bean类型的类。例如，您需要使用一个完全不遵守JavaBean规范的旧连接池，Spring也可以管理它。<br/>

使用基于XML的配置元数据，您可以按如下方式指定bean类：

	<bean id="exampleBean" class="examples.ExampleBean"/>
	<bean name="anotherExample" class="examples.ExampleBeanTwo"/>

更多关于为构造函数提供参数的机制（如果需要）以及在构造对象后设置对象实例属性的详细信息，请参阅[依赖注入](http://www.isharefy.com)。 <br/>

**使用静态工厂方法实例化** <br/>
定义通过静态工厂方法创建的bean时，您可以使用class属性指定包含静态工厂方法的类，用factory-method属性指定工厂方法本身的名称。您应该能够调用此方法（使用后边描述的可选参数）并返回一个实时对象，随后对这个对象进行处理，就好像这个对象是通过构造函数创建的一样。这种bean定义的一个用途是在遗留代码中调用静态工厂。<br/>

以下bean定义指定了一个通过调用工厂方法创建的bean。该定义未指定返回对象的类型（类），仅指定包含工厂方法的类。在此示例中，createInstance()方法必须是一个静态方法。以下示例显示如何指定工厂方法：

```
<bean id="clientService"
    class="examples.ClientService"
    factory-method="createInstance"/>
```

以下示例显示了一个可以使用前面的bean定义的类：

```
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
```

更多有关向工厂方法提供(可选的)参数以及在从工厂返回对象后设置对象实例属性的机制的详细信息，请参阅[依赖关系和详细配置](http://www.isharefy.com)。

**使用实例工厂方法实例化** <br/>
与通过静态工厂方法实例化类似，使用实例工厂方法进行实例化会从容器调用现有bean的非静态方法来创建新bean。 要使用此机制，请将class属性保留为空，并在factory-bean属性中指定当前（或父级或祖先）容器中bean的名称，该容器包含要调用以创建对象的实例方法。使用factory-method属性设置工厂方法本身的名称。以下示例显示如何配置此类bean：

```
<!-- 工厂bean，包含一个名为 createInstance() 的方法 -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- 注入此定位器bean所需的任何依赖项 -->
</bean>

<!-- 通过工厂bean创建的bean -->
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>
```

以下示例显示了相应的Java类：

```
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }
}
```

一个工厂类也可以包含多个工厂方法，如以下示例所示：

```
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>

<bean id="accountService"
    factory-bean="serviceLocator"
    factory-method="createAccountServiceInstance"/>
```

以下示例显示了相应的Java类：

```
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    private static AccountService accountService = new AccountServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }

    public AccountService createAccountServiceInstance() {
        return accountService;
    }
}
```

这种方法表明可以通过依赖注入（DI）来管理和配置工厂bean本身。请参阅[Dependencies and Configuration in Detail](http://www.isharefy.com)。

```
在Spring文档中，“factory bean”是指在Spring容器中配置并通过实例或静态工厂方法创建对象的bean。
相比之下，FactoryBean（注意大小写）是指Spring特定的FactoryBean。
```

<a id="依赖关系"></a>
### 1、4 依赖关系
典型的企业应用程序不包含单个对象（或Spring用法中的bean）。即使是最简单的应用程序也有一些对象可以协同工作，以呈现给最终用户所看到的连贯应用程序。下一节将解释如何从定义独立的许多bean定义过渡到一个完全实现的应用程序，在这个应用程序中，对象协作以实现目标。

<a id="依赖注入"></a>
#### 1、4、1 依赖注入
依赖注入（DI）是一个过程，在这个过程中，对象只能通过构造函数参数、工厂方法的参数或在构造对象实例后在对象实例上设置的属性来定义它们的依赖关系（即，它们使用的其他对象）、从工厂方法返回。然后容器在创建bean时注入这些依赖项。这个过程本质上是bean本身的反向（因此称为控制反转），它通过使用类的直接构造或服务定位器模式来控制其依赖项的实例化或位置。<br/>

使用DI原则的代码更清晰，当对象具有依赖关系时，解耦更有效。对象不查找其依赖项，也不知道依赖项的位置或类。因此，您的类变得更容易测试，特别是当依赖关系在接口或抽象基类上时，这允许在单元测试中使用存根或模拟实现。<br/>

依赖注入存在两个主要变体：[基于构造函数的依赖注入](#基于构造函数的依赖注入)和[基于Setter的依赖注入](#基于Setter的依赖注入)。<br/>

<a id="基于构造函数的依赖注入"></a>
**基于构造函数的依赖注入** <br/>
基于构造函数的DI由容器调用具有多个参数的构造函数来完成，每个参数表示一个依赖项。调用具有特定参数的静态工厂方法来构造bean几乎是等效的，本讨论同样处理构造函数和静态工厂方法的参数。以下示例显示了一个只能使用构造函数注入进行依赖注入的类：

```
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on a MovieFinder
    private MovieFinder movieFinder;

    // a constructor so that the Spring container can inject a MovieFinder
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```

请注意，这个类没有什么特别之处。它是一个POJO，它不依赖于容器特定的接口、基类或注释。

<a id="构造函数参数解析"></a>
*构造函数参数解析* <br/>
使用参数的类型进行构造函数参数解析匹配。如果bean定义的构造函数参数中不存在潜在的歧义，那么在bean定义中定义构造函数参数的顺序是在实例化bean时将这些参数提供给适当的构造函数的顺序。考虑以下类：

```
package x.y;

public class ThingOne {

    public ThingOne(ThingTwo thingTwo, ThingThree thingThree) {
        // ...
    }
}
```

假设ThingTwo和ThingThree类与继承无关，则不存在潜在的歧义。因此，以下配置工作正常，您无需在```<constructor-arg/>```元素中显式指定构造函数参数索引或类型。

```
<beans>
    <bean id="thingOne" class="x.y.ThingOne">
        <constructor-arg ref="thingTwo"/>
        <constructor-arg ref="thingThree"/>
    </bean>

    <bean id="thingTwo" class="x.y.ThingTwo"/>

    <bean id="thingThree" class="x.y.ThingThree"/>
</beans>
```

当引用另一个bean时，类型是已知的，并且可以进行匹配（与前面的示例一样）。当使用简单类型时，例如<value> true </value>，Spring无法确定值的类型，因此无法在没有帮助的情况下按类型进行匹配。考虑以下类：

```
package examples;

public class ExampleBean {

    // Number of years to calculate the Ultimate Answer
    private int years;

    // The Answer to Life, the Universe, and Everything
    private String ultimateAnswer;

    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

*构造函数参数类型匹配* <br/>
在前面的场景中，如果使用type属性显式指定构造函数参数的类型，则容器可以使用与简单类型匹配的类型。如下例所示：

```
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
</bean>
```

*构造函数参数索引* <br/>
您可以使用index属性显式指定构造函数参数的索引，如以下示例所示：

```
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>
```

除了解决多个简单值的歧义之外，指定索引还可以解决构造函数具有相同类型的两个参数的歧义。<br/>
*注意：index是从0开始的* <br/> 


*构造函数参数名称* <br/>
您还可以使用构造函数参数名称来消除值歧义，如以下示例所示：

```
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```

请记住，为了使这项工作开箱即用，您的代码必须在启用调试标志的情况下进行编译，以便Spring可以从构造函数中查找参数名称。如果您不能或不想使用调试标志编译代码，可以使用@ConstructorProperties JDK注解显式地为构造函数参数命名。简单的类示例如下所示：

```
package examples;

public class ExampleBean {

    // Fields omitted

    @ConstructorProperties({"years", "ultimateAnswer"})
    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

<a id="基于Setter的依赖注入"></a>
**基于Setter的依赖注入** <br/>
在调用无参数构造函数或无参数静态工厂方法实例化bean之后，基于setter的依赖注入由bean上的容器通过调用setter方法来完成依赖注入。<br/>

以下示例显示了一个只通过使用setter进行依赖注入的类。这个类是常规的Java。它是一个POJO，不依赖于容器特定的接口、基类或注释。

```
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on the MovieFinder
    private MovieFinder movieFinder;

    // a setter method so that the Spring container can inject a MovieFinder
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```

ApplicationContext管理的bean支持基于构造函数和基于setter的依赖注入。在通过构造函数方法注入了一些依赖项之后，它还支持基于setter的依赖注入。您可以用BeanDefinition与PropertyEditor实例结合的形式配置依赖项，从而将属性从一种格式转换为另一种格式。但是，大多数Spring用户并不直接使用这些类（即以编程方式），而是使用XML bean定义、带注解的组件（即使用@Component，@Controller等注解的类）或基于Java的@Configuration类中的@Bean方法。然后，这些源在内部转换为BeanDefinition的实例，并用于加载整个Spring IoC容器实例。

```
选择基于构造函数的依赖注入还是选择基于setter的依赖注入？

由于您可以混合使用基于构造函数和基于setter的依赖注入，因此将构造函数用于强制依赖项
和将setter方法或者配置方法用于可选依赖项是一个很好的经验法则。
请注意，在setter方法上使用@Required注解可用于使属性成为必需的依赖项。

Spring团队通常提倡构造函数注入，因为它可以将您应用程序组件的实现作为不可变对象，
并确保必须的依赖项不为null。此外，构造函数注入的组件总是以完全初始化的状态返回到
客户端（调用）代码。作为旁注，大量的构造函数参数是一个糟糕的代码方式，暗示该类
可能有太多的职责，应该重构以更好地解决职责的正确分离。

Setter注入应该只用于可在类中指定合理默认值的可选依赖项。否则，在代码使用依赖项的
任何位置都必须执行非空检查。setter注入的一个好处是：setter方法使该类的对象可以在
以后重新配置或重新注入。因此，通过JMX MBean进行管理是二次注入的一个引人注目的用例。

使用对特定类最有意义的依赖注入格式。有时候，在处理没有源代码的第三方类时，您将会做出选择。
例如，如果第三方类没有公开任何setter方法，那么构造函数注入可能是唯一可用的依赖注入形式。
```

**依赖性解析过程** <br/>
容器执行bean依赖性解析如下所示：

* 	使用配置元数据创建和初始化的ApplicationContext用来描述所有beans。配置元数据可以通过XML、Java代码或注解指定。
*  对于每个bean，它的依赖关系以属性、构造函数参数或静态工厂方法的参数的形式表示（如果使用它而不是普通的构造函数）。实际创建bean的时候，会将这些依赖项提供给bean。
*  每个属性或构造函数参数都是要设置的值的实际定义，或者是对容器中另一个bean的引用。
*  作为值的每个属性或构造函数参数都从其指定的格式转换为该属性或构造函数参数的实际类型。默认情况下，Spring可以将以字符串格式提供的值转换为所有内置类型，例如int，long，String，boolean等。

Spring容器在创建容器时验证每个bean的配置。然而，直到真正创建bean时，才会设置bean属性本身。当容器创建时，将创建单例作用域并设置为预实例化（默认值）的bean。作用域定义在[Bean作用域](http://www.isharefy.com)中。否则，只有在请求时才创建bean。创建bean可能会导致创建bean的图形，因为bean的依赖关系及其依赖关系（诸如此类）被创建和分配。请注意，这些依赖项之间的解析不匹配可能出现的较晚 - 即在第一次创建受影响的bean时。<br/>

```
循环依赖

如果您主要使用构造函数注入，则可能创建无法解析的循环依赖关系场景。

例如：类A通过构造函数注入需要的类B的实例，而类B通过构造函数注入
需要的类A的实例。如果为A类和B类配置bean以便相互注入，
则Spring IoC容器会在运行时检测此循环引用，
并抛出BeanCurrentlyInCreationException异常。

一种可能的解决方案是编辑由setter而不是构造函数配置的某些类的源代码。
或者，避免使用构造函数注入，只使用setter注入。换句话说，尽管不推荐使用，
但您可以使用setter注入配置循环依赖关系。

与典型情况（没有循环依赖）不同，bean A和bean B之间的循环依赖强制其中一个bean在完全初始化之前被注入另一个bean（经典的鸡与鸡蛋场景）。
```

你通常可以相信Spring会做正确的事。它在容器加载时检测配置问题，例如检测不存在的bean和循环依赖的引用。当实际创建bean时，Spring会尽可能晚地设置属性并解析依赖项。这意味着，如果在创建该对象或其中一个依赖项时出现问题，则在请求对象时，正确加载的Spring容器会在以后生成异常 - 例如，bean会抛出缺少或无效属性异常。这可能会延迟一些配置问题的可见性，这就是默认情况下ApplicationContext实现预先实例化单例bean的原因。在实际需要之前以一些前期时间和内存为代价来创建这些bean，您会在创建ApplicationContext时发现配置问题，而不是更晚。您仍然可以覆盖此默认行为，以便单例bean可以懒加载，而不是预先实例化。<br/>

如果不存在循环依赖，当一个或多个协作bean被注入依赖bean时，每个协作bean在被注入依赖bean之前完全配置。 这意味着，如果bean A依赖于bean B，则Spring IoC容器在调用bean A上的setter方法之前完全配置bean B.换句话说，bean被实例化（如果它不是预先实例化的单例 ），设置其依赖项，并调用相关的生命周期方法（如[配置的初始化方法](http://www.isharefy.com)或[Bean初始化回调方法](http://www.isharefy.com)）。<br/>


**依赖注入的示例** <br/>
以下示例将基于XML的配置元数据用于基于setter的依赖注入。Spring XML配置文件的一小部分指定了一些bean定义，如下所示：

```
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- 使用嵌套的ref元素进行setter注入 -->
    <property name="beanOne">
        <ref bean="anotherExampleBean"/>
    </property>

    <!-- 使用整洁的属性进行setter注入 -->
    <property name="beanTwo" ref="yetAnotherBean"/>
    <property name="integerProperty" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

以下示例显示了相应的ExampleBean类：

```
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public void setBeanOne(AnotherBean beanOne) {
        this.beanOne = beanOne;
    }

    public void setBeanTwo(YetAnotherBean beanTwo) {
        this.beanTwo = beanTwo;
    }

    public void setIntegerProperty(int i) {
        this.i = i;
    }
}
```

在前面的示例中，setter被声明为与XML文件中指定的属性匹配。以下示例使用基于构造函数的依赖注入：

```
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- 使用嵌套的ref元素进行构造函数注入 -->
    <constructor-arg>
        <ref bean="anotherExampleBean"/>
    </constructor-arg>

    <!-- 使用整洁的属性进行构造函数注入 -->
    <constructor-arg ref="yetAnotherBean"/>

    <constructor-arg type="int" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

以下示例显示了相应的ExampleBean类：

```
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public ExampleBean(
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {
        this.beanOne = anotherBean;
        this.beanTwo = yetAnotherBean;
        this.i = i;
    }
}
```

bean定义中指定的构造函数参数用作ExampleBean的构造函数的参数。<br/>

现在考虑这个示例的变体，其中，不使用构造函数，而是告诉Spring调用静态工厂方法来返回对象的实例：

```
<bean id="exampleBean" class="examples.ExampleBean" factory-method="createInstance">
    <constructor-arg ref="anotherExampleBean"/>
    <constructor-arg ref="yetAnotherBean"/>
    <constructor-arg value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

以下示例显示了相应的ExampleBean类：

```
public class ExampleBean {

    // 一个私有的构造函数
    private ExampleBean(...) {
        ...
    }

    // 一个静态工厂方法;无论实际使用这些参数的方式如何，都可以将此方法的参数视为返回的bean的依赖关系。 
    public static ExampleBean createInstance (
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {

        ExampleBean eb = new ExampleBean (...);
        // 一些其他操作...
        return eb;
    }
}
```

静态工厂方法的参数由```<constructor-arg/>```元素提供，与实际使用的构造函数完全相同。工厂方法返回的类的类型不必与包含静态工厂方法的类相同（尽管在本例中是这样的）。实例（非静态）工厂方法可以以基本相同的方式使用（除了使用factory-bean属性而不是class属性），因此我们不在这里讨论这些细节。

<a id="依赖关系和配置的详细介绍"></a>
#### 1、4、2 依赖关系和配置的详细介绍
如[上一节](#依赖注入)所述，您可以将bean属性和构造函数参数定义作为其他被管理的bean（协作者）的引用或者作为内联定义的值。Spring基于XML配置元数据支持在```<property/>```和```<constructor-arg/>```元素中使用子元素类型。

**直接数值（原始数值、字符串等）** <br/>
```<property/>```元素的value属性将属性或构造函数参数指定为人类可读的字符串形式。Spring的[转换服务](http://www.isharefy.com)被用于将这些值从字符串转换为属性或参数的实际类型。下面的例子显示了要设置的各种值：

```
<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <!-- results in a setDriverClassName(String) call -->
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
    <property name="username" value="root"/>
    <property name="password" value="masterkaoli"/>
</bean>
```

以下示例使用[p命名空间](http://www.isharefy.com)进行更简洁的XML配置：

```
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close"
        p:driverClassName="com.mysql.jdbc.Driver"
        p:url="jdbc:mysql://localhost:3306/mydb"
        p:username="root"
        p:password="masterkaoli"/>

</beans>
```

前面的XML更简洁。但是，除非您在创建bean定义时使用支持自动补全属性的IDE（例如[IntelliJ IDEA](https://www.jetbrains.com/idea/)或[Spring Tool Suite](https://spring.io/tools3/sts)），否则会在运行时而不是设计时发现拼写错误。强烈建议使用此类IDE的提示功能。<br/>

您还可以配置一个java.util.Properties实例，如下所示：

```
<bean id="mappings"
    class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">

    <!-- typed as a java.util.Properties -->
    <property name="properties">
        <value>
            jdbc.driver.className=com.mysql.jdbc.Driver
            jdbc.url=jdbc:mysql://localhost:3306/mydb
        </value>
    </property>
</bean>
```

Spring容器通过使用JavaBeans的PropertyEditor机制将```<value/>```元素内的文本转换为java.util.Properties实例。这是一个很好的快捷方式，并且是Spring团队支持在value属性样式上使用嵌套```<value/>```元素的少数几个地方之一。<br/>

*idref元素* <br/>
idref元素只是一种防错控制的方法，可以将容器中另一个bean的id（字符串值 - 而不是引用）传递给```<constructor-arg/>```或```<property/>```元素。以下示例显示了如何使用它：

```
<bean id="theTargetBean" class="..."/>

<bean id="theClientBean" class="...">
    <property name="targetName">
        <idref bean="theTargetBean"/>
    </property>
</bean>
```

前面bean定义的代码段与以下代码段完全等效（在运行时）：

```
<bean id="theTargetBean" class="..." />

<bean id="client" class="...">
    <property name="targetName" value="theTargetBean"/>
</bean>
```

第一种形式优于第二种形式，第一种形式使用的idref标记允许容器在部署时验证引用的、指定的bean确实存在。在第二种形式中，不对传递给client bean的targetName属性的值进行验证，当客户端bean实际被实例化时，才会发现错误（最可能是致命的结果）。如果client bean是一个标准的bean，则只能在部署容器后很长时间才能发现此错误和产生的异常。

```
注意：在4.0的beans XSD中不再支持idref元素的local属性，因为它不再提供常规bean引用的值。
升级到4.0架构时，将现有的idref local引用更改为idref bean。
```

```<idref/>```元素带来值的一个常见位置（至少在Spring 2.0之前的版本中）是在ProxyFactoryBean bean定义中的[AOP拦截器](https://docs.spring.io/spring/docs/5.1.0.RELEASE/spring-framework-reference/core.html#aop-pfb-1)的配置中。指定拦截器名称时使用```<idref/>```元素可以防止对拦截器ID的拼写错误。

**引用其他Beans（合作者）** <br/>
ref元素是```<constructor-arg />```或```<property />```定义元素中的最后一个元素。在这里，您将bean的指定属性的值设置为对容器管理的另一个bean（协作者）的引用。引用的bean是要设置其属性的bean的依赖项，并且在设置该属性之前根据需要对其进行初始化。（如果协作者是单例bean，它可能已经被容器初始化。）所有的引用最终都是对另一个对象的引用。范围和验证取决于您是否是通过bean，local或parent属性指定其他对象的ID或者名称。<br/>

通过```<ref />```标记的bean属性指定目标bean是最常用的形式，并允许创建对同一容器或父容器中的任何bean的引用，而不管它是否在同一XML文件中。bean属性的值可以与目标bean的id属性相同，或者与目标bean的name属性中的值相同。以下示例显示如何使用ref元素：

	<ref bean="someBean"/>

通过parent属性指定目标bean会创建对当前容器的父容器中的bean的引用。parent属性的值可以与目标bean的id属性或目标bean的name属性中的一个值相同。目标bean必须位于当前bean的父容器中。当您有一个容器层次结构，并且希望使用与父bean同名的代理将现有bean包装在父容器中时，您应该主要使用此bean引用变量。下面两个清单展示了如何使用父属性:

```
<!-- 在父级对象环境中 -->
<bean id="accountService" class="com.something.SimpleAccountService">
    <!-- insert dependencies as required as here -->
</bean>
```

```
<!-- 在子级环境中 -->
<bean id="accountService" <!-- bean名称与父bean相同 -->
    class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target">
        <ref parent="accountService"/> <!-- 注意我们如何引用父bean -->
    </property>
    <!-- insert other configuration and dependencies as required here -->
</bean>
```

```
注意：4.0版本的 beans XSD不再支持ref元素的local属性，因为它不再提供常规bean引用的值。
升级到4.0架构时，将现有的ref本地引用更改为ref bean。
```

<a id="内部Beans"></a>
**内部Beans** <br/>
```<property />```或```<constructor-arg />```元素中的```<bean />```元素定义了内部bean，如下例所示：

```
<bean id="outer" class="...">
    <!-- 只需定义内联目标bean，而不是使用对目标bean的引用 -->
    <property name="target">
        <bean class="com.example.Person"> <!-- 这是内部bean -->
            <property name="name" value="Fiona Apple"/>
            <property name="age" value="25"/>
        </bean>
    </property>
</bean>
```

内部bean定义不需要定义的ID或者名称。如果指定ID或者名称，则容器不会使用这样的值作为标识符。容器还会在创建时忽略scope标志，因为内部bean始终是匿名的，并且总是与外部bean一起创建的。不可能独立访问内部bean，也不能将它们注入协作bean(而不同于注入到封闭的bean)。<br/>

作为一种很少出现的情况，从特定的域中有可能会收到销毁回调函数，例如，对于请求域内的内部bean包含单例bean：内部bean实例的创建会绑定到它的包含bean，但销毁回调函数允许它进入到请求域的生命周期中。这不是一个常见的场景；内部bean通常简单的共享它们的包含bean的作用域。<br/>

**集合** <br/>

































