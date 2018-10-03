#Spring框架文档（版本：5.1.0）
## 核心技术 
[原文档链接](https://docs.spring.io/spring/docs/5.1.0.RELEASE/spring-framework-reference/)<br>

参考文档的这一部分涵盖了Spring框架中不可或缺的所有技术。<br/>

其中最重要的是Spring框架的控制反转(IoC)容器。对Spring框架的IoC容器进行彻底的处理之后，紧接着是对Spring面向方面编程(AOP)技术的全面覆盖。Spring框架有自己的AOP框架，它在概念上很容易理解，并且成功地解决了Java企业编程中AOP需求的80%优点。<br/>

本文还介绍了Spring与AspectJ的集成(就特性而言，目前是最丰富的，当然也是Java企业空间中最成熟的AOP实现)。<br/>

### 1、IoC容器
本章介绍Spring的反转控制(IoC)容器。<br/>

#### 1、1 Spring IoC容器和Beans介绍
本章介绍了控制反转(IoC)原理的Spring框架实现。(参见 [IoC介绍](https://docs.spring.io/spring/docs/5.1.0.RELEASE/spring-framework-reference/overview.html#background-ioc) )IoC也称为依赖注入(DI)。在这个过程中，对象只通过构造函数参数、工厂方法的参数或对象实例在构造或从工厂方法返回后设置的属性来定义它们的依赖关系(也就是说，它们使用的其他对象)


































