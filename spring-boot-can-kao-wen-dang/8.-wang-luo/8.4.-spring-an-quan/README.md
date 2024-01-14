# 8.4. spring安全

如果[Spring Security](https://spring.io/projects/spring-security)位于类路径上，则默认情况下 Web 应用程序是安全的。Spring Boot 依赖 Spring Security 的内容协商策略来确定是否使用`httpBasic`或`formLogin`。要向 Web 应用程序添加方法级安全性，您还可以添加`@EnableGlobalMethodSecurity`所需的设置。[其他信息可以在Spring Security 参考指南](https://docs.spring.io/spring-security/reference/6.2/servlet/authorization/method-security.html)中找到。

默认情况`UserDetailsService`下只有一个用户。用户名是`user`，密码是随机的，并且在应用程序启动时以 WARN 级别打印，如下例所示：

```
Using generated security password: 78fa095d-3f4c-48b1-ad50-e24c31d5cf35

This generated password is for development use only. Your security configuration must be updated before running your application in production.
```

> 如果您微调日志记录配置，请确保`org.springframework.boot.autoconfigure.security`类别设置为日志`WARN`级别消息。否则，不会打印默认密码。

您可以通过提供`spring.security.user.name`和`spring.security.user.password`来更改用户名和密码。

Web 应用程序默认提供的基本功能包括：

* 具有内存存储的`UserDetailsService`（或者在 WebFlux 应用程序的情况下的`ReactiveUserDetailsService`）bean 和具有生成密码的单个用户（请参阅[`SecurityProperties.User`](https://docs.spring.io/spring-boot/docs/3.2.0/api/org/springframework/boot/autoconfigure/security/SecurityProperties.User.html) 参考资料 中的用户属性）。
* 整个应用程序（包括执行器端点，如果执行器位于类路径上）的基于表单的登录或 HTTP 基本安全性（取决于请求中的`Accept`标头）。
* 用于发布身份验证事件的`DefaultAuthenticationEventPublisher`。

您可以通过为其添加一个 bean 来提供不同的`AuthenticationEventPublisher`内容。







