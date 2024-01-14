# 7.9. 单元测试

Spring Boot 提供了许多实用程序和注解来帮助测试您的应用程序。测试支持由两个模块提供：包含核心项目的`spring-boot-test`，并支持测试自动配置的`spring-boot-test-autoconfigure`。

大多数开发人员使用`spring-boot-starter-test`“Starter”，它导入了 Spring Boot 测试模块以及 JUnit Jupiter、AssertJ、Hamcrest 和许多其他有用的库。

如果您有使用 JUnit 4 的测试，则可以使用 JUnit 5 的老式引擎来运行它们。要使用老式引擎，请添加对`junit-vintage-engine`的依赖项，如以下示例所示：

```xml
<dependency>
    <groupId>org.junit.vintage</groupId>
    <artifactId>junit-vintage-engine</artifactId>
    <scope>test</scope>
    <exclusions>
        <exclusion>
            <groupId>org.hamcrest</groupId>
            <artifactId>hamcrest-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

`org.hamcrest:hamcrest` 的`hamcrest-core`被排除在外，那是`spring-boot-starter-test`的一部分。

####

####

####

####
