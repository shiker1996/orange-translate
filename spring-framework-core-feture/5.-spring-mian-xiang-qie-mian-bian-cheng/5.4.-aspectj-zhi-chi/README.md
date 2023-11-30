# 5.4. @AspectJ 支持

@AspectJ 指的是一种将切面声明为带有注解的常规 Java 类的风格。@AspectJ 样式是由 [AspectJ 项目](https://www.eclipse.org/aspectj)作为 AspectJ 5 版本的一部分引入的。Spring 解释与 AspectJ 5 相同的注解，使用 AspectJ 提供的库进行切入点解析和匹配。但是，AOP 运行时仍然是纯 Spring AOP，并且不依赖于 AspectJ 编译器或编织器。

使用 AspectJ 编译器和编织器可以使用完整的 AspectJ 语言，并在[将 AspectJ 与 Spring 应用程序一起使用](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-using-aspectj)中进行了讨论。











