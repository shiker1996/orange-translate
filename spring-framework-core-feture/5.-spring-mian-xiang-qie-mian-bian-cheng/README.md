# 5. Spring 面向切面编程

面向切面编程 (AOP) 通过提供另一种思考程序结构的方式来补充面向对象编程 (OOP)。OOP 中模块化的关键单元是类，而 AOP 中模块化的单元是切面。切面支持跨多种类型和对象的关注点（例如事务管理）的模块化。（这种关注点在 AOP 文献中通常被称为“横切”关注点。）

Spring 的关键组件之一是 AOP 框架。虽然 Spring IoC 容器不依赖 AOP（这意味着如果您不想使用 AOP，则无需使用 AOP），AOP 补充了 Spring IoC 以提供非常强大的中间件解决方案。

带有 AspectJ 切入点的 Spring AOP

Spring 通过使用 [基于模式的方法](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-schema)或[@AspectJ 注解样式](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-ataspectj)提供了编写自定义切面的简单而强大的方法。这两种风格都提供了完全类型化的通知和使用 AspectJ 切入点语言，同时仍然使用 Spring AOP 进行编织。

本章讨论基于模式和@AspectJ 的 AOP 支持。较低级别的 AOP 支持将在[下一章](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-api)中讨论。

AOP 在 Spring Framework 中用于：

* 提供声明式企业服务。最重要的此类服务是 [声明式事务管理](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#transaction-declarative)。
* 让用户实现自定义切面，用 AOP 补充他们对 OOP 的使用。

如果您只对通用声明式服务或其他预打包的声明式中间件服务（例如池）感兴趣，则无需直接使用 Spring AOP，并且可以跳过本章的大部分内容。

####

####

####

####

####

####

####

####

####

####

####
