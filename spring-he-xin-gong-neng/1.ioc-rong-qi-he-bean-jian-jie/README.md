# 1.IOC容器和Bean简介

#### 1.1. Spring IoC 容器和 Bean 简介

本章介绍了控制反转 (IoC) 原则的 Spring Framework 实现。IoC 也称为依赖注入 (DI)。这是一个过程，对象仅通过构造函数参数、工厂方法的参数或在对象实例被构造或从工厂方法返回后设置的属性来定义它们的依赖关系（即与它们一起工作的其他对象） . 然后容器在创建 bean 时注入这些依赖项。这个过程基本上是 bean 本身通过使用类的直接构造或诸如服务定位器模式之类的机制来控制其依赖关系的实例化或位置的逆过程（因此称为控制反转）。

`org.springframework.beans`和`org.springframework.context`包是 Spring Framework 的 IoC 容器的基础。 [`BeanFactory`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/beans/factory/BeanFactory.html) 接口提供了一种高级配置机制，能够管理任何类型的对象。 [`ApplicationContext`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/context/ApplicationContext.html) 是`BeanFactory`的子接口。它增加了如下拓展：

* 更容易与 Spring 的 AOP 功能集成
* 消息资源处理（用于国际化）
* 事件发布
* 应用层特定上下文，例如用于 Web 应用程序的上下文`WebApplicationContext`。

简而言之，`BeanFactory`提供了配置框架和基本功能，而`ApplicationContext`增加了更多的企业特定功能。`ApplicationContext`是`BeanFactory`的完整超集，并且在本章中专门用于描述 Spring 的 IoC 容器。有关使用`BeanFactory`代替`ApplicationContext`更多信息，请参阅涵盖 [`BeanFactory`API](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-beanfactory)的部分。

在 Spring 中，构成应用程序主干并由 Spring IoC 容器管理的对象称为 bean。bean 是由 Spring IoC 容器实例化、组装和管理的对象。否则，bean 只是应用程序中的众多对象之一。Bean 以及它们之间的依赖关系反映在容器使用的配置元数据中。

####

####

####

#### 1.5. Bean作用域

当您创建一个 bean 定义时，您创建了一个用于创建由该 bean 定义定义的类的实际实例的方法。bean 定义是一个配方的想法很重要，因为这意味着，与一个类一样，您可以从一个配方创建许多对象实例。

您不仅可以控制要插入到从特定 bean 定义创建的对象中的各种依赖项和配置值，还可以控制从特定 bean 定义创建的对象的作用域。这种方法功能强大且灵活，因为您可以通过配置选择您创建的对象的作用域，而不必在 Java 类级别烘焙对象的作用域。可以将 Bean 定义为部署在多个作用域之一中。Spring 框架支持六个作用域，其中四个仅在您使用 web-aware `ApplicationContext`时可用。您还可以创建 [自定义作用域。](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-custom)

下表描述了支持的作用域：

| 作用域                                                                                                                           | 描述                                                                                                                             |
| ----------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| [singleton](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-singleton)     | （默认）将单个 bean 定义限定为每个 Spring IoC 容器的单个对象实例。                                                                                     |
| [prototype](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-prototype)     | 将单个 bean 定义限定为任意数量的对象实例。                                                                                                       |
| [request](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-request)         | 将单个 bean 定义限定为单个 HTTP 请求的生命周期。也就是说，每个 HTTP 请求都有自己的 bean 实例，该实例是在单个 bean 定义的后面创建的。仅在 Web 感知 Spring 的上下文中有效`ApplicationContext`。 |
| [session](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-session)         | 将单个 bean 定义限定为 HTTP 的生命周期`Session`。仅在 Web 感知 Spring 的上下文中有效`ApplicationContext`。                                               |
| [application](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-application) | 将单个 bean 定义限定为`ServletContext`. 仅在 Web 感知 Spring 的上下文中有效`ApplicationContext`。                                                  |
| [application](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#websocket-stomp-websocket-scope)   | 将单个 bean 定义限定为`WebSocket`. 仅在 Web 感知 Spring 的上下文中有效`ApplicationContext`。                                                       |

从 Spring 3.0 开始，线程作用域可用，但默认情况下未注册。有关详细信息，请参阅 [`SimpleThreadScope`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/context/support/SimpleThreadScope.html). 有关如何注册此或任何其他自定义作用域的说明，请参阅 [使用自定义作用域](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-custom-using)。

**1.5.1. 单例作用域**

只有一个单例 bean 的共享实例被管理，并且所有对具有与该 bean 定义匹配的一个或多个 ID 的 bean 的请求都会导致 Spring 容器返回一个特定的 bean 实例。

换句话说，当您定义 bean 定义并将其限定为单例时，Spring IoC 容器会创建该 bean 定义所定义的对象的一个实例。此单个实例存储在此类单例 bean 的缓存中，并且该命名 bean 的所有后续请求和引用都返回缓存的对象。下图显示了单例作用域的工作原理：

![单身人士](https://docs.spring.io/spring-framework/reference/_images/singleton.png)

Spring 的单例 bean 概念不同于设计模式 (GoF) 模式书中定义的单例模式。GoF 单例对对象的作用域进行硬编码，以便每个 ClassLoader 创建一个且仅一个特定类的实例。Spring 单例的作用域最好描述为每个容器和每个 bean。这意味着，如果您在单个 Spring 容器中为特定类定义一个 bean，则 Spring 容器会创建该 bean 定义所定义的类的一个且仅一个实例。单例作用域是 Spring 中的默认作用域。要将 bean 定义为 XML 中的单例，您可以定义一个 bean，如下例所示：

```xml
<bean id="accountService" class="com.something.DefaultAccountService"/>

<!-- the following is equivalent, though redundant (singleton scope is the default) -->
<bean id="accountService" class="com.something.DefaultAccountService" scope="singleton"/>
```

**1.5.2. 原型作用域**

bean 部署的非单例原型作用域导致每次对特定 bean 发出请求时都会创建一个新的 bean 实例。也就是说，将 bean 注入到另一个 bean 中，或者您通过`getBean()`容器上的方法调用来请求它。通常，您应该对所有有状态 bean 使用原型作用域，对无状态 bean 使用单例作用域。

下图说明了 Spring 原型作用域：

![原型](https://docs.spring.io/spring-framework/reference/_images/prototype.png)

（数据访问对象 (DAO) 通常不配置为原型，因为典型的 DAO 不保存任何会话状态。我们更容易重用单例图的核心。）

以下示例将 bean 定义为 XML 中的原型：

```xml
<bean id="accountService" class="com.something.DefaultAccountService" scope="prototype"/>
```

与其他作用域相比，Spring 不管理原型 bean 的完整生命周期。容器实例化、配置和以其他方式组装原型对象并将其交给客户端，而没有进一步记录该原型实例。因此，尽管在所有对象上调用初始化生命周期回调方法而不考虑作用域，但在原型的情况下，不会调用配置的销毁生命周期回调。客户端代码必须清理原型作用域的对象并释放原型 bean 拥有的昂贵资源。要让 Spring 容器释放原型作用域 bean 持有的资源，请尝试使用自定义[bean 后处理器](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-extension-bpp)，它包含对需要清理的 bean 的引用。

在某些方面，Spring 容器在原型作用域 bean 方面的角色是 Java`new`运算符的替代品。此后的所有生命周期管理都必须由客户处理。（有关 Spring 容器中 bean 的生命周期的详细信息，请参阅[Lifecycle Callbacks](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle)。）

**1.5.3. 具有原型 bean 依赖关系的单例 bean**

当您使用具有原型 bean 依赖关系的单例作用域 bean 时，请注意依赖关系在实例化时解决。因此，如果您将原型作用域的 bean 依赖注入到单例作用域的 bean 中，则会实例化一个新的原型 bean，然后将依赖注入到单例 bean 中。原型实例是提供给单例作用域 bean 的唯一实例。

但是，假设您希望单例作用域的 bean 在运行时重复获取原型作用域的 bean 的新实例。您不能将原型作用域的 bean 依赖注入到单例 bean 中，因为该注入仅发生一次，当 Spring 容器实例化单例 bean 并解析并注入其依赖项时。如果您在运行时多次需要原型 bean 的新实例，请参阅[方法注入](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-method-injection)。

**1.5.4. 请求、会话、应用程序和 WebSocket 作用域**

`request`、`session`、`application`和`websocket`作用域仅在您使用可Web `ApplicationContext`的 Spring实现（例如`XmlWebApplicationContext` ）时才可用。如果您将这些作用域与常规 Spring IoC 容器（例如`ClassPathXmlApplicationContext` ）一起使用，则会抛出一个抱怨未知 bean 作用域的`IllegalStateException`问题。

**初始 Web 配置**

为了支持`request`、`session`、`application`和 `websocket`级别的 bean 作用域（网络作用域的 bean），在定义 bean 之前需要进行一些小的初始配置。（标准作用域不需要此初始设置：`singleton`和`prototype`。）

如何完成此初始设置取决于您的特定 Servlet 环境。

如果您在 Spring Web MVC 中访问作用域 bean，实际上，在 Spring 处理的请求中`DispatcherServlet`，不需要特殊设置。 `DispatcherServlet`已经暴露了所有相关状态。

如果您使用 Servlet 2.5 Web 容器，请求在 Spring 之外处理 `DispatcherServlet`（例如，当使用 JSF 或 Struts 时），您需要注册 `org.springframework.web.context.request.RequestContextListener` `ServletRequestListener`. 对于 Servlet 3.0+，这可以通过使用`WebApplicationInitializer` 接口以编程方式完成。或者，或者对于较旧的容器，将以下声明添加到您的 Web 应用程序的`web.xml`文件中：

```xml
<web-app>
    ...
    <listener>
        <listener-class>
            org.springframework.web.context.request.RequestContextListener
        </listener-class>
    </listener>
    ...
</web-app>
```

或者，如果您的侦听器设置存在问题，请考虑使用 Spring 的 `RequestContextFilter`. 过滤器映射取决于周围的 Web 应用程序配置，因此您必须根据需要进行更改。以下清单显示了 Web 应用程序的过滤器部分：

```xml
<web-app>
    ...
    <filter>
        <filter-name>requestContextFilter</filter-name>
        <filter-class>org.springframework.web.filter.RequestContextFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>requestContextFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    ...
</web-app>
```

`DispatcherServlet`, `RequestContextListener`, 和`RequestContextFilter`都做完全相同的事情，即将 HTTP 请求对象绑定到为该`Thread`请求提供服务的对象。这使得请求和会话作用域的 bean 在调用链的下游可用。

**请求作用域**

考虑以下 bean 定义的 XML 配置：

```xml
<bean id="loginAction" class="com.something.LoginAction" scope="request"/>
```

Spring 容器通过为每个 HTTP 请求使用`LoginAction` bean 定义来创建`loginAction` bean 的新实例。也就是说， `loginAction`bean 的作用域是 HTTP 请求级别。您可以根据需要更改创建的实例的内部状态，因为从同一`loginAction`bean 定义创建的其他实例看不到这些状态更改。它们是针对个人要求的。当请求完成处理时，该请求作用域内的 bean 将被丢弃。

当使用注解驱动的组件或 Java 配置时，`@RequestScope`注解可用于将组件分配给`request`作用域。以下示例显示了如何执行此操作：

```java
@RequestScope
@Component
public class LoginAction {
    // ...
}
```

**会话作用域**

考虑以下 bean 定义的 XML 配置：

```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session"/>
```

Spring 容器通过在单个 HTTP 会话的生命周期内使用 `userPreferences bean`定义来创建 `UserPreferences bean` 的新实例。换句话说，`userPreferences bean` 的有效范围是 HTTP 会话级别。与请求范围的 bean 一样，您可以根据需要更改所创建的实例的内部状态，因为您知道也使用从同一 `userPreferences bean` 定义创建的实例的其他 HTTP Session 实例看不到这些状态更改，因为它们特定于单个 HTTP 会话。当 HTTP 会话最终被丢弃时，作用域为该特定 HTTP 会话的 bean 也会被丢弃。

在使用注解驱动的组件或 Java 配置时，您可以使用 `@SessionScope`注解将组件分配给`session`作用域。

```java
@SessionScope
@Component
public class UserPreferences {
    // ...
}
```

**适用作用域**

考虑以下 bean 定义的 XML 配置：

```xml
<bean id="appPreferences" class="com.something.AppPreferences" scope="application"/>
```

Spring 容器通过为整个 Web 应用程序使用一次`AppPreferences` bean 定义来创建`appPreferences` bean 的新实例。也就是说， `appPreferences`bean 是在`ServletContext`级别上限定的，并存储为常规 `ServletContext`属性。这有点类似于 Spring 单例 bean，但在两个重要方面有所不同：它是每个`ServletContext`的单例，而不是 Spring `ApplicationContext`的单例（在任何给定的 Web 应用程序中可能有多个单例），并且它实际上是公开的，因此作为`ServletContext`属性可见.

在使用注解驱动的组件或 Java 配置时，您可以使用 `@ApplicationScope`注解将组件分配给`application`作用域。以下示例显示了如何执行此操作：

```java
@ApplicationScope
@Component
public class AppPreferences {
    // ...
}
```

**WebSocket 作用域**

WebSocket 作用域与 WebSocket 会话的生命周期相关联，适用于 STOMP over WebSocket 应用程序，请参阅 [WebSocket 作用域](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#websocket-stomp-websocket-scope)了解更多详细信息。

**作用域 Bean 作为依赖项**

Spring IoC 容器不仅管理对象（bean）的实例化，还管理协作者（或依赖项）的连接。如果你想（例如）将一个 HTTP 请求作用域的 bean 注入到另一个具有更长生命周期的 bean 中，你可以选择注入一个 AOP 代理来代替这个作用域 bean。也就是说，您需要注入一个代理对象，该对象公开与作用域对象相同的公共接口，但也可以从相关作用域（例如 HTTP 请求）检索真实目标对象，并将方法调用委托给真实对象。

您还可以在作用域为`singleton` 的 bean 之间使用`<aop:scoped-proxy/>`，然后引用通过可序列化的中间代理，因此能够在反序列化时重新获取目标单例 bean。

当针对作用域的 bean声明时`prototype`使用`<aop:scoped-proxy/>`，共享代理上的每个方法调用都会导致创建一个新的目标实例，然后将调用转发到该实例。

此外，作用域代理并不是以生命周期安全的方式从较短的作用域访问 bean 的唯一方法。您还可以将您的注入点（即构造函数或 setter 参数或自动装配字段）声明为`ObjectFactory<MyTargetBean>`，从而允许`getObject()`在每次需要时调用以按需检索当前实例 - 无需保留实例或单独存储它。

作为扩展变体，您可以为`ObjectProvider<MyTargetBean>` 声明提供几个额外的访问变体，包括`getIfAvailable`和`getIfUnique`。调用它的 JSR-330 变体，`Provider`并与`Provider<MyTargetBean>` 声明和`get()`每次检索尝试的相应调用一起使用。有关整体 JSR-330 的更多详细信息，请参见[此处](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-standard-annotations)。

以下示例中的配置只有一行，但重要的是要了解其背后的“为什么”以及“如何”：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- an HTTP Session-scoped bean exposed as a proxy -->
    <bean id="userPreferences" class="com.something.UserPreferences" scope="session">
        <!-- instructs the container to proxy the surrounding bean -->
        <aop:scoped-proxy/> 
    </bean>

    <!-- a singleton-scoped bean injected with a proxy to the above bean -->
    <bean id="userService" class="com.something.SimpleUserService">
        <!-- a reference to the proxied userPreferences bean -->
        <property name="userPreferences" ref="userPreferences"/>
    </bean>
</beans>
```

要创建这样的代理，请将子`<aop:scoped-proxy/>`元素插入到作用域 bean 定义中（请参阅[选择要创建的代理类型](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-other-injection-proxies)和 [基于 XML 模式的配置](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#xsd-schemas)）。为什么在`request`,`session`和自定义作用域级别的 bean 定义需要该`<aop:scoped-proxy/>`元素？考虑以下单例 bean 定义并将其与您需要为上述作用域定义的内容进行对比（请注意，以下 `userPreferences`bean 定义是不完整的）：

```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session"/>

<bean id="userManager" class="com.something.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

在前面的示例中，单例 bean ( `userManager`) 被注入了对 HTTP`Session`作用域 bean ( `userPreferences`) 的引用。这里的重点是 `userManager`bean 是一个单例：每个容器只实例化一次，并且它的依赖项（在本例中只有一个，`userPreferences`bean）也只注入一次。这意味着`userManager`bean 仅对完全相同的`userPreferences`对象（即最初注入的对象）进行操作。

当将较短生命周期的作用域 bean 注入较长生命周期的作用域 bean 时，这不是您想要的行为（例如，将 HTTP 会话作用域协作 bean 作为依赖项注入到单例 bean 中）。相反，您需要一个 `userManager` 对象，并且在 HTTP 会话的生命周期内，您需要一个特定于 HTTP 会话的 `userPreferences` 对象。因此，容器创建一个公开与 `UserPreferences` 类完全相同的公共接口的对象（最好是一个 `UserPreferences` 实例的对象），该对象可以从作用域机制（HTTP 请求、Session 等）获取真正的 `UserPreferences` 对象。容器将此代理对象注入到 `userManager` bean 中，而 `userManager` bean 并不知道此 `UserPreferences` 引用是一个代理。在此示例中，当 `UserManager` 实例调用依赖注入的 `UserPreferences` 对象上的方法时，它实际上是调用代理上的方法。然后，代理从（在本例中）HTTP 会话获取真实的\`UserPreferences 对象，并将方法调用委托给检索到的真实 UserPreferences 对象。

因此，在将`request-`和 `session-scoped` bean 注入协作对象时，您需要以下（正确且完整的）配置，如以下示例所示：

```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session">
    <aop:scoped-proxy/>
</bean>

<bean id="userManager" class="com.something.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

**选择要创建的代理类型**

默认情况下，当 Spring 容器为使用`<aop:scoped-proxy/>`元素标记的 bean 创建代理时，会创建基于 CGLIB 的类代理。

CGLIB 代理只拦截公共方法调用！不要在此类代理上调用非公共方法。它们没有委托给实际作用域的目标对象。

或者，您可以配置 Spring 容器，通过指定元素`<aop:scoped-proxy/>`的`proxy-target-class`属性为`false`，为此类作用域 bean 创建基于标准 JDK 接口的代理。使用基于 JDK 接口的代理意味着您不需要应用程序类路径中的其他库来影响此类代理。但是，这也意味着作用域 bean 的类必须实现至少一个接口，并且注入作用域 bean 的所有协作者都必须通过其接口之一引用该 bean。以下示例显示了基于接口的代理：

```xml
<!-- DefaultUserPreferences implements the UserPreferences interface -->
<bean id="userPreferences" class="com.stuff.DefaultUserPreferences" scope="session">
    <aop:scoped-proxy proxy-target-class="false"/>
</bean>

<bean id="userManager" class="com.stuff.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

有关选择基于类或基于接口的代理的更多详细信息，请参阅[代理机制](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-proxying)。

**1.5.5. 自定义作用域**

bean 作用域机制是可扩展的。您可以定义自己的作用域，甚至重新定义现有作用域，尽管后者被认为是不好的做法，并且您不能覆盖内置`singleton`和`prototype`作用域。

**创建自定义作用域**

要将您的自定义作用域集成到 Spring 容器中，您需要实现 `org.springframework.beans.factory.config.Scope`接口，这将在本节中描述。有关如何实现自己的作用域的想法，请参阅Spring `Scope` 框架本身和 [`Scope`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/beans/factory/config/Scope.html)javadoc 提供的实现，其中更详细地解释了您需要实现的方法。

`Scope`接口有四种方法可以从作用域中获取对象，将它们从作用域中移除，并让它们被销毁。

例如，会话作用域实现返回会话作用域的 bean（如果它不存在，则该方法在将其绑定到会话以供将来参考之后返回 bean 的新实例）。以下方法从底层作用域返回对象：

```java
Object get(String name, ObjectFactory<?> objectFactory)
```

例如，会话作用域实现从底层会话中删除会话作用域 bean。该对象应该被返回，但是`null`如果没有找到具有指定名称的对象，您可以返回。以下方法从底层作用域中删除对象：

```java
Object remove(String name)
```

以下方法注册了一个回调，当作用域被销毁或作用域中的指定对象被销毁时应调用该回调：

```java
void registerDestructionCallback(String name, Runnable destructionCallback)
```

有关销毁回调的更多信息，请参阅[javadoc](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/beans/factory/config/Scope.html#registerDestructionCallback) 或 Spring 作用域实现。

以下方法获取基础作用域的对话标识符：

```java
String getConversationId()
```

这个标识符对于每个作用域都是不同的。对于会话作用域的实现，此标识符可以是会话标识符。

**使用自定义作用域**

在编写和测试一个或多个自定义`Scope`实现之后，您需要让 Spring 容器知道您的新作用域。以下方法是向Spring `Scope` 容器注册新的中心方法：

```java
void registerScope(String scopeName, Scope scope);
```

此方法在`ConfigurableBeanFactory`接口上声明，可通过 Spring 附带的大多数具体`ApplicationContext`实现的`BeanFactory`属性获得。

`registerScope(..)`方法的第一个参数是与作用域关联的唯一名称。Spring 容器本身中此类名称的示例是`singleton`和 `prototype`。`registerScope(..)`方法的第二个参数是您希望注册和使用的自定义`Scope`实现的实际实例。

假设您编写了自定义`Scope`实现，然后按照下一个示例所示进行注册。

下一个示例使用`SimpleThreadScope`，它包含在 Spring 中，但默认情况下未注册。对于您自己的自定义`Scope` 实现，说明将是相同的。

```java
Scope threadScope = new SimpleThreadScope();
beanFactory.registerScope("thread", threadScope);
```

然后，您可以创建符合您的自定义作用域`Scope`规则的 bean 定义， 如下所示：

```xml
<bean id="..." class="..." scope="thread">
```

使用自定义`Scope`实现，您不仅限于作用域的编程注册。您还可以`Scope`使用 `CustomScopeConfigurer`类以声明方式进行注册，如以下示例所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean class="org.springframework.beans.factory.config.CustomScopeConfigurer">
        <property name="scopes">
            <map>
                <entry key="thread">
                    <bean class="org.springframework.context.support.SimpleThreadScope"/>
                </entry>
            </map>
        </property>
    </bean>

    <bean id="thing2" class="x.y.Thing2" scope="thread">
        <property name="name" value="Rick"/>
        <aop:scoped-proxy/>
    </bean>

    <bean id="thing1" class="x.y.Thing1">
        <property name="thing2" ref="thing2"/>
    </bean>

</beans>
```

当您在实现的 `FactoryBean`的`<bean>`声明中放置`<aop:scoped-proxy/>`时，作用域是工厂 bean 本身，而不是从`getObject()`.

#### 1.6. 自定义 Bean 的性质

Spring Framework 提供了许多接口，您可以使用它们来自定义 bean 的性质。本节将它们分组如下：

* [生命周期回调](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle)
* [`ApplicationContextAware`和`BeanNameAware`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-aware)
* [其他`Aware`接口](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aware-list)

**1.6.1. 生命周期回调**

要与容器对 bean 生命周期的管理进行交互，可以实现 Spring`InitializingBean`和`DisposableBean`接口。容器调用了 `afterPropertiesSet()`方法和`destroy()`方法让 bean 在初始化和销毁 bean 时执行某些操作。

JSR-250`@PostConstruct`和`@PreDestroy`注解通常被认为是在现代 Spring 应用程序中接收生命周期回调的最佳实践。使用这些注解意味着您的 bean 不会耦合到 Spring 特定的接口。有关详细信息，请参阅[使用`@PostConstruct`和`@PreDestroy`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-postconstruct-and-predestroy-annotations)。如果您不想使用 JSR-250 注解但仍想移除耦合，请考虑使用`init-method`和`destroy-method`bean 定义元数据。

在内部，Spring 框架使用`BeanPostProcessor`实现来处理它可以找到的任何回调接口并调用适当的方法。如果你需要自定义特性或其他生命周期行为 Spring 默认不提供，你可以自己实现一个`BeanPostProcessor`。有关详细信息，请参阅 [容器扩展点](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-extension)。

除了初始化和销毁回调之外，Spring 管理的对象还可以实现`Lifecycle`接口，以便这些对象可以参与容器自身生命周期驱动的启动和关闭过程。

本节介绍生命周期回调接口。

**初始化回调**

在容器为 bean 设置了所有必要的属性后，`org.springframework.beans.factory.InitializingBean`接口允许 bean 执行初始化工作。`InitializingBean`接口指定了一个方法：

```java
void afterPropertiesSet() throws Exception;
```

我们建议您不要使用`InitializingBean`接口，因为它不必要地将代码耦合到 Spring。或者，我们建议使用[`@PostConstruct`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-postconstruct-and-predestroy-annotations)注解或指定 POJO 初始化方法。在基于 XML 的配置元数据的情况下，您可以使用`init-method`属性来指定具有无效无参数签名的方法的名称。通过 Java 配置，您可以使用 `@Bean`的`initMethod`属性. 请参阅[接收生命周期回调](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java-lifecycle-callbacks)。考虑以下示例：

```xml
<bean id="exampleInitBean" class="examples.ExampleBean" init-method="init"/>
```

```java
public class ExampleBean {

    public void init() {
        // do some initialization work
    }
}
```

前面的示例与下面的示例（由两个列表组成）具有几乎完全相同的效果：

```xml
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
```

```java
public class AnotherExampleBean implements InitializingBean {

    @Override
    public void afterPropertiesSet() {
        // do some initialization work
    }
}
```

但是，前面两个示例中的第一个没有将代码耦合到 Spring。

**销毁回调**

实现该`org.springframework.beans.factory.DisposableBean`接口可以让 bean 在包含它的容器被销毁时获得回调。 `DisposableBean`接口指定了一个方法：

```java
void destroy() throws Exception;
```

我们建议您不要使用`DisposableBean`回调接口，因为它不必要地将代码耦合到 Spring。或者，我们建议使用[`@PreDestroy`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-postconstruct-and-predestroy-annotations)注解或指定 bean 定义支持的通用方法。使用基于 XML 的配置元数据，您可以使用`<bean/>`的`destroy-method`属性. 通过 Java 配置，您可以使用`@Bean`的`destroyMethod`属性. 请参阅 [接收生命周期回调](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java-lifecycle-callbacks)。考虑以下定义：

```xml
<bean id="exampleInitBean" class="examples.ExampleBean" destroy-method="cleanup"/>
```

```java
public class ExampleBean {

    public void cleanup() {
        // do some destruction work (like releasing pooled connections)
    }
}
```

前面的定义与下面的定义几乎完全相同：

```xml
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
```

```java
public class AnotherExampleBean implements DisposableBean {

    @Override
    public void destroy() {
        // do some destruction work (like releasing pooled connections)
    }
}
```

但是，前面两个定义中的第一个没有将代码耦合到 Spring。

您可以为 `<bean>`元素的 `destroy-method` 属性分配一个特殊的`(inferred)`值，该值指示 Spring 自动检测特定 bean 类上的公共 `close` 或 `shutdown` 方法。 （因此，任何实现 `java.lang.AutoCloseable` 或 `java.io.Closeable` 的类都会匹配。）您还可以在`<beans>` 元素的 `default-destroy-method` 属性上设置此特殊`(inferred)`值，以将此行为应用于一整套 bean（请参阅默认初始化和销毁方法）。请注意，这是 Java 配置的默认行为。

**默认初始化和销毁方法**

当您编写不使用 Spring 特定`InitializingBean`和`DisposableBean`回调接口的初始化和销毁方法回调时，您通常会编写名称为`init()`、`initialize()`、`dispose()`等的方法。理想情况下，此类生命周期回调方法的名称在整个项目中是标准化的，以便所有开发人员使用相同的方法名称并确保一致性。

您可以将 Spring 容器配置为“查找”命名初始化并销毁每个 bean 上的回调方法名称。这意味着，作为应用程序开发人员，您可以编写应用程序类并使用名为`init()` 的初始化回调 ，而无需为`init-method="init"`每个 bean 定义配置属性。Spring IoC 容器在创建 bean 时调用该方法（并根据[前面描述](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle)的标准生命周期回调协定）。此功能还为初始化和销毁方法回调强制执行一致的命名约定。

假设您的初始化回调方法已命名为`init()`，而您的销毁回调方法已命名为`destroy()`。然后，您的类类似于以下示例中的类：

```java
public class DefaultBlogService implements BlogService {

    private BlogDao blogDao;

    public void setBlogDao(BlogDao blogDao) {
        this.blogDao = blogDao;
    }

    // this is (unsurprisingly) the initialization callback method
    public void init() {
        if (this.blogDao == null) {
            throw new IllegalStateException("The [blogDao] property must be set.");
        }
    }
}
```

然后，您可以在类似于以下内容的 bean 中使用该类：

```xml
<beans default-init-method="init">

    <bean id="blogService" class="com.something.DefaultBlogService">
        <property name="blogDao" ref="blogDao" />
    </bean>

</beans>
```

顶级`<beans/>`元素属性上的属性`default-init-method`的存在导致 Spring IoC 容器将 bean 类上调用的`init`方法识别为初始化方法回调。当创建和组装一个 bean 时，如果 bean 类有这样的方法，它会在适当的时候被调用。

您可以使用顶级`<beans/>`元素上的属性`default-destroy-method`类似地配置销毁方法回调（即在 XML 中） 。

如果现有的 bean 类已经具有命名与约定不一致的回调方法，您可以通过使用`<bean/>` 自身的`init-method`和`destroy-method`属性指定（在 XML 中）方法名称来覆盖默认值。

Spring 容器保证在为 bean 提供所有依赖项后立即调用配置的初始化回调。因此，在原始 bean 引用上调用初始化回调，这意味着 AOP 拦截器等尚未应用于 bean。首先完全创建一个目标 bean，然后应用一个 AOP 代理（例如）及其拦截器链。如果目标 bean 和代理是分开定义的，您的代码甚至可以绕过代理与原始目标 bean 交互。因此，将拦截器应用于该`init`方法将是不一致的，因为这样做会将目标 bean 的生命周期与其代理或拦截器耦合，并在您的代码直接与原始目标 bean 交互时留下奇怪的语义。

**结合生命周期机制**

从 Spring 2.5 开始，您可以通过三个选项来控制 bean 生命周期行为：

* 和[`InitializingBean`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-initializingbean)回调 [`DisposableBean`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-disposablebean)接口
* 自定义`init()`和`destroy()`方法
* [`@PostConstruct`和`@PreDestroy` 注解](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-postconstruct-and-predestroy-annotations)。\_ 您可以结合这些机制来控制给定的 bean。

如果为一个 bean 配置了多个生命周期机制，并且每个机制都配置了不同的方法名称，那么每个配置的方法都按照本注解后列出的顺序运行。`init()`但是，如果为多个生命周期机制 配置了相同的方法名称（例如， 对于初始化方法），则该方法将运行一次，如上[一节所述](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-default-init-destroy-methods)。

为同一个bean配置的多个生命周期机制，不同的初始化方法，调用如下：

1. 用注解的方法`@PostConstruct`
2. `afterPropertiesSet()`由`InitializingBean`回调接口定义
3. 自定义配置`init()`方法

销毁方法的调用顺序相同：

1. 用注解的方法`@PreDestroy`
2. `destroy()`由`DisposableBean`回调接口定义
3. 自定义配置`destroy()`方法

**启动和关闭回调**

`Lifecycle`接口定义了任何具有自己生命周期要求的对象的基本方法（例如启动和停止某些后台进程）：

```java
public interface Lifecycle {

    void start();

    void stop();

    boolean isRunning();
}
```

任何 Spring 管理的对象都可以实现`Lifecycle`接口。然后，当 `ApplicationContext`自身接收到启动和停止信号时（例如，对于运行时的停止/重新启动场景），它会将这些调用级联到该上下文中定义的所有`Lifecycle`实现。它通过委托给`LifecycleProcessor`来做到这一点，如以下清单所示：

```java
public interface LifecycleProcessor extends Lifecycle {

    void onRefresh();

    void onClose();
}
```

请注意，`LifecycleProcessor`本身就是`Lifecycle` 接口的扩展。它还添加了另外两种方法来对正在刷新和关闭的上下文做出反应。

请注意，常规`org.springframework.context.Lifecycle`接口是显式启动和停止通知的简单约定，并不意味着在上下文刷新时自动启动。要对特定 bean 的自动启动（包括启动阶段）进行细粒度控制，请考虑实施`org.springframework.context.SmartLifecycle`。另外，请注意，停止通知不能保证在销毁之前发出。在常规关闭时，所有`Lifecycle`bean 在传播一般销毁回调之前首先收到停止通知。但是，在上下文的生命周期内进行热刷新或停止刷新尝试时，只会调用销毁方法。

启动和关闭调用的顺序可能很重要。如果任何两个对象之间存在“依赖”关系，则依赖方在其依赖之后开始，并在其依赖之前停止。但是，有时，直接依赖关系是未知的。您可能只知道某种类型的对象应该先于另一种类型的对象开始。在这些情况下，`SmartLifecycle`接口定义了另一个选项`getPhase()`，即在其超接口`Phased`上定义的方法 . 以下清单显示了`Phased`接口的定义：

```java
public interface Phased {

    int getPhase();
}
```

以下清单显示了`SmartLifecycle`接口的定义：

```java
public interface SmartLifecycle extends Lifecycle, Phased {

    boolean isAutoStartup();

    void stop(Runnable callback);
}
```

启动时，相位最低的对象首先启动。停止时，按照相反的顺序。因此，实现`SmartLifecycle`并且其`getPhase()`方法返回的对象`Integer.MIN_VALUE`将是第一个开始和最后一个停止的对象。在频谱的另一端，相位值 `Integer.MAX_VALUE`表示对象应该最后启动并首先停止（可能是因为它依赖于正在运行的其他进程）。在考虑阶段值时，了解任何未实现`SmartLifecycle`的“正常”`Lifecycle`对象的默认阶段为`0`也很重要。因此，任何负相位值都表示对象应该在这些标准组件之前开始（并在它们之后停止）。对于任何正相位值，反之亦然。

`SmartLifecycle`定义的 stop 方法接受一个回调。在该实现的关闭过程完成后，任何实现都必须调用该回调的`run()`方法。这会在必要时启用异步关闭，因为接口`LifecycleProcessor` 的默认实现`DefaultLifecycleProcessor`，等待每个阶段内的对象组调用该回调的超时值。默认的每阶段超时为 30 秒。您可以通过在上下文中定义一个命名的`lifecycleProcessor` bean 来覆盖默认的生命周期处理器实例 。如果您只想修改超时，定义以下内容就足够了：

```xml
<bean id="lifecycleProcessor" class="org.springframework.context.support.DefaultLifecycleProcessor">
    <!-- timeout value in milliseconds -->
    <property name="timeoutPerShutdownPhase" value="10000"/>
</bean>
```

如前所述，该`LifecycleProcessor`接口还定义了用于刷新和关闭上下文的回调方法。后者驱动关闭过程，就好像`stop()`已被显式调用一样，但它发生在上下文关闭时。另一方面，“刷新”回调启用了 `SmartLifecycle`bean 的另一个特性。刷新上下文时（在所有对象都已实例化和初始化之后），将调用该回调。此时，默认生命周期处理器会检查每个 `SmartLifecycle`对象的`isAutoStartup()`方法返回的布尔值。如果`true`，则该对象在该点启动，而不是等待上下文或其自身的显式调用`start()`方法（与上下文刷新不同，对于标准上下文实现，上下文启动不会自动发生）。如前所述，`phase`值和任何“依赖”关系决定了启动顺序。

**在非 Web 应用程序中优雅地关闭 Spring IoC 容器**

本节仅适用于非 Web 应用程序。Spring 的基于 Web 的 `ApplicationContext`实现已经有代码可以在相关 Web 应用程序关闭时优雅地关闭 Spring IoC 容器。

如果您在非 Web 应用程序环境中（例如，在富客户端桌面环境中）使用 Spring 的 IoC 容器，请向 JVM 注册一个关闭挂钩。这样做可确保正常关闭并在单例 bean 上调用相关的销毁方法，以便释放所有资源。您仍然必须正确配置和实现这些销毁回调。

要注册关闭挂钩，请调用接口`registerShutdownHook()`上声明的方法`ConfigurableApplicationContext`，如以下示例所示：

```java
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public final class Boot {

    public static void main(final String[] args) throws Exception {
        ConfigurableApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");

        // add a shutdown hook for the above context...
        ctx.registerShutdownHook();

        // app runs here...

        // main method exits, hook is called prior to the app shutting down...
    }
}
```

**1.6.2. `ApplicationContextAware`和`BeanNameAware`**

当 an`ApplicationContext`创建一个实现 `org.springframework.context.ApplicationContextAware`接口的对象实例时，会为该实例提供对该 的引用`ApplicationContext`。以下清单显示了`ApplicationContextAware`接口的定义：

```java
public interface ApplicationContextAware {

    void setApplicationContext(ApplicationContext applicationContext) throws BeansException;
}
```

因此，bean 可以通过 `ApplicationContext` 接口或通过将引用强制转换为该接口的已知子类（例如 `ConfigurableApplicationContext`，它公开了附加功能），以编程方式操作创建它们的 `ApplicationContext`。一种用途是以编程方式检索其他 bean。有时此功能很有用。然而，一般来说，您应该避免它，因为它将代码耦合到 Spring 并且不遵循控制反转风格，在这种风格中，协作者作为属性提供给 bean。 `ApplicationContext` 的其他方法提供对文件资源的访问、发布应用程序事件以及访问 `MessageSource`。这些附加功能在 `ApplicationContext` 的附加功能中进行了描述。

自动装配是获取 `ApplicationContext` 引用的另一种替代方法。 _传统_ `constructor`模式和自动装配`byType`模式（如[Autowiring Collaborators](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-autowire)中所述）可以分别为构造函数参数或 setter 方法参数提供`ApplicationContext`类型依赖 。要获得更大的灵活性，包括自动装配字段和多个参数方法的能力，请使用基于注解的自动装配功能。如果你这样做，如果相关的字段、构造函数或方法带有`@Autowired`注解，则`ApplicationContext`会自动装配到期望`ApplicationContext`类型的字段、构造函数参数或方法参数中。有关详细信息，请参阅 [使用](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-autowired-annotation).[`@Autowired`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-autowired-annotation)

当创建一个实现 `org.springframework.beans.factory.BeanNameAware`接口的`ApplicationContext`类时，该类被提供了对其关联对象定义中定义的名称的引用。以下清单显示了 BeanNameAware 接口的定义：

```java
public interface BeanNameAware {

    void setBeanName(String name) throws BeansException;
}
```

在填充普通 bean 属性之后但在初始化回调（例如`InitializingBean.afterPropertiesSet()`自定义 init 方法）之前调用回调。

**1.6.3. 其他`Aware`接口**

除了`ApplicationContextAware`和`BeanNameAware`（前面讨论[过](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-aware)）之外，Spring 提供了广泛的`Aware`回调接口，让 bean 向容器指示它们需要特定的基础设施依赖项。作为一般规则，名称表示依赖类型。下表总结了最重要的`Aware`接口：

| 名称                               | 注入依赖                                                                | 解释...                                                                                                                                          |
| -------------------------------- | ------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `ApplicationContextAware`        | 声明`ApplicationContext`.                                             | [`ApplicationContextAware`和`BeanNameAware`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-aware) |
| `ApplicationEventPublisherAware` | 封闭的事件发布者`ApplicationContext`。                                       | [的附加功能`ApplicationContext`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#context-introduction)                |
| `BeanClassLoaderAware`           | 类加载器用于加载 bean 类。                                                    | [实例化 Bean](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-class)                                  |
| `BeanFactoryAware`               | 声明`BeanFactory`.                                                    | [`BeanFactory`API \_](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-beanfactory)                         |
| `BeanNameAware`                  | 声明 bean 的名称。                                                        | [`ApplicationContextAware`和`BeanNameAware`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-aware) |
| `LoadTimeWeaverAware`            | 定义的编织器，用于在加载时处理类定义。                                                 | [在 Spring 框架中使用 AspectJ 进行加载时编织](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-aj-ltw)                     |
| `MessageSourceAware`             | 用于解析消息的配置策略（支持参数化和国际化）。                                             | [的附加功能`ApplicationContext`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#context-introduction)                |
| `NotificationPublisherAware`     | Spring JMX 通知发布者。                                                   | [通知](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#jmx-notifications)                                   |
| `ResourceLoaderAware`            | 为对资源进行低级访问而配置的加载程序。                                                 | [资源](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#resources)                                                  |
| `ServletConfigAware`             | 当前`ServletConfig`容器在其中运行。仅在可感知网络的 Spring 中有效 `ApplicationContext`。  | [spring MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc)                                                 |
| `ServletContextAware`            | 当前`ServletContext`容器在其中运行。仅在可感知网络的 Spring 中有效 `ApplicationContext`。 | [spring MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc)                                                 |

再次注意，使用这些接口将您的代码绑定到 Spring API，并且不遵循 Inversion of Control 样式。因此，我们建议将它们用于需要以编程方式访问容器的基础设施 bean。

#### 1.7. Bean定义继承

一个 bean 定义可以包含很多配置信息，包括构造函数参数、属性值和特定于容器的信息，例如初始化方法、静态工厂方法名称等。子 bean 定义从父定义继承配置数据。子定义可以根据需要覆盖某些值或添加其他值。使用父子bean定义可以节省大量输入。实际上，这是一种模板形式。

如果您以编程方式使用接口`ApplicationContext`，则子 bean 定义由`ChildBeanDefinition`类表示。大多数用户不在此级别上与他们合作。相反，他们在诸如`ClassPathXmlApplicationContext`. 当您使用基于 XML 的配置元数据时，您可以通过使用属性来指示子 bean 定义`parent`，将父 bean 指定为该属性的值。以下示例显示了如何执行此操作：

```xml
<bean id="inheritedTestBean" abstract="true"
        class="org.springframework.beans.TestBean">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>

<bean id="inheritsWithDifferentClass"
        class="org.springframework.beans.DerivedTestBean"
        parent="inheritedTestBean" init-method="initialize">  
    <property name="name" value="override"/>
    <!-- the age property value of 1 will be inherited from parent -->
</bean>
```

如果没有指定子 bean 定义，则使用父定义中的 bean 类，但也可以覆盖它。在后一种情况下，子 bean 类必须与父类兼容（即，它必须接受父类的属性值）。

子 bean 定义从父 bean 继承作用域、构造函数参数值、属性值和方法覆盖，并可选择添加新值。您指定的任何作用域、初始化方法、销毁方法或`static`工厂方法设置都会覆盖相应的父设置。

其余的设置总是取自子定义：依赖、自动装配模式、依赖检查、单例和惰性初始化。

前面的示例通过使用`abstract`属性将父 bean 定义显式标记为抽象。如果父定义未指定类，则将父 bean 定义显式标记`abstract`为必需，如以下示例所示：

```xml
<bean id="inheritedTestBeanWithoutClass" abstract="true">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>

<bean id="inheritsWithClass" class="org.springframework.beans.DerivedTestBean"
        parent="inheritedTestBeanWithoutClass" init-method="initialize">
    <property name="name" value="override"/>
    <!-- age will inherit the value of 1 from the parent bean definition-->
</bean>
```

父 bean 不能自己实例化，因为它不完整，而且它也显式标记为`abstract`. 当定义为`abstract`时，它只能用作纯模板 bean 定义，用作子定义的父定义。尝试单独使用这样的`abstract`父 bean，通过将其引用为另一个 bean 的 ref 属性或`getBean()`使用父 bean ID 进行显式调用会返回错误。同样，容器的内部 `preInstantiateSingletons()`方法会忽略定义为抽象的 bean 定义。

默认情况下`ApplicationContext`预实例化所有单例。因此，重要的是（至少对于单例 bean），如果您有一个（父）bean 定义，您打算仅将其用作模板，并且此定义指定了一个类，则必须确保&#x5C06;_&#x62BD;&#x8C61;_&#x5C5E;性设置&#x4E3A;_&#x74;rue_，否则应用程序上下文将实际（尝试）预实例化`abstract`bean。

#### 1.8. 容器扩展点

通常，应用程序开发人员不需要子类化`ApplicationContext` 实现类。相反，可以通过插入特殊集成接口的实现来扩展 Spring IoC 容器。接下来的几节描述了这些集成接口。

**1.8.1. 通过使用自定义 Bean`BeanPostProcessor`**

`BeanPostProcessor`接口定义了您可以实现的回调方法，以提供您自己的（或覆盖容器的默认）实例化逻辑、依赖关系解析逻辑等。如果你想在 Spring 容器完成实例化、配置和初始化 bean 之后实现一些自定义逻辑，你可以插入一个或多个自定义`BeanPostProcessor`实现。

您可以配置多个`BeanPostProcessor`实例，并且可以通过设置`order`属性来控制这些`BeanPostProcessor`实例的运行顺序。仅当`BeanPostProcessor`实现`Ordered` 接口时才能设置此属性。如果你自己写`BeanPostProcessor`，你也应该考虑实现`Ordered`接口。有关详细信息，请参阅 [`BeanPostProcessor`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/beans/factory/config/BeanPostProcessor.html) 和[`Ordered`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/core/Ordered.html)接口的 javadoc。另请参阅有关[实例的](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-programmatically-registering-beanpostprocessors)[编程注册的`BeanPostProcessor`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-programmatically-registering-beanpostprocessors)说明。

`BeanPostProcessor`实例对 bean（或对象）实例进行操作。也就是说，Spring IoC 容器实例化一个 bean 实例，然后`BeanPostProcessor` 实例完成它们的工作。`BeanPostProcessor`实例的作用域是每个容器。这仅在您使用容器层次结构时才相关。如果您在一个容器中定义`BeanPostProcessor`，它只会对该容器中的 bean 进行后处理。换句话说，在一个容器中定义的 bean 不会被另一个容器中定义的 bean 进行后处理`BeanPostProcessor`，即使两个容器是同一层次结构的一部分。要更改实际的 bean 定义（即定义 bean 的蓝图），您需要使用 a`BeanFactoryPostProcessor`，如 使用[自定义配置元数据中所述`BeanFactoryPostProcessor`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-extension-factory-postprocessors)。

该`org.springframework.beans.factory.config.BeanPostProcessor`接口恰好由两个回调方法组成。当这样的类注册为容器的后处理器时，对于容器创建的每个 bean 实例，后处理器都会在容器初始化方法（例如`InitializingBean.afterPropertiesSet()`或任何声明`init`的方法）之前从容器中获取回调调用，并在任何 bean 初始化回调之后。后处理器可以对 bean 实例采取任何行动，包括完全忽略回调。一个 bean 后处理器通常检查回调接口，或者它可以用代理包装一个 bean。一些 Spring AOP 基础结构类被实现为 bean 后处理器，以提供代理包装逻辑。

自动检测在实现接口`ApplicationContext`的配置元数据中定义的任何 bean 。`ApplicationContext`将 这些 `BeanPostProcessor`bean 注册为后处理器，以便稍后在创建 bean 时调用它们。Bean 后处理器可以以与任何其他 bean 相同的方式部署在容器中。

请注意，当在配置类上使用工厂方法`@Bean`声明 `BeanPostProcessor` 时，工厂方法的返回类型应该是实现类本身或至少是`org.springframework.beans.factory.config.BeanPostProcessor` 接口，清楚地表明该 bean 的后处理器性质。否则，在 `ApplicationContext`完全创建之前无法按类型自动检测它。由于需要尽早实例化`BeanPostProcessor` 以应用于上下文中其他 bean 的初始化，因此这种早期类型检测至关重要。

以编程方式注册`BeanPostProcessor`实例虽然推荐的`BeanPostProcessor`注册方法是通过 `ApplicationContext`自动检测（如前所述），但您可以`ConfigurableBeanFactory`使用该`addBeanPostProcessor` 方法以编程方式注册它们。当您需要在注册之前评估条件逻辑，甚至在层次结构中跨上下文复制 bean 后处理器时，这可能很有用。但是请注意，以`BeanPostProcessor`编程方式添加的实例不遵从`Ordered`接口。在这里，注册的顺序决定了执行的顺序。另请注意，以`BeanPostProcessor`编程方式注册的实例始终在通过自动检测注册的实例之前处理，无论任何显式排序如何。

`BeanPostProcessor`实例和 AOP 自动代理实现`BeanPostProcessor`接口的类是特殊的，被容器区别对待。它们直接引用的所有`BeanPostProcessor`实例和 bean 都在启动时实例化，作为`ApplicationContext`. 接下来，所有`BeanPostProcessor`实例都以排序方式注册并应用于容器中的所有其他 bean。因为 AOP 自动代理是作为`BeanPostProcessor`自身实现的，所以`BeanPostProcessor` 实例和它们直接引用的 bean 都没有资格进行自动代理，因此没有将方面编织到其中。对于任何这样的 bean，您应该会看到一条信息性日志消息：`Bean someBean is not eligible for getting processed by all BeanPostProcessor interfaces (for example: not eligible for auto-proxying)`.如果您`BeanPostProcessor`使用自动装配或 `@Resource`（可能回退到自动装配）将 bean 连接到您的 bean，则 Spring 在搜索类型匹配依赖项候选时可能会访问意外的 bean，因此，使它们没有资格进行自动代理或其他类型的 bean 发布-加工。例如，如果您有一个依赖项注解，`@Resource`其中字段或 setter 名称不直接对应于 bean 的声明名称并且没有使用 name 属性，则 Spring 会访问其他 bean 以按类型匹配它们。

以下示例展示了如何`BeanPostProcessor`在`ApplicationContext`.

**示例：Hello World, `BeanPostProcessor`-style**

第一个示例说明了基本用法。该示例显示了一个自定义 `BeanPostProcessor`实现，该实现调用`toString()`容器创建的每个 bean 的方法，并将结果字符串打印到系统控制台。

以下清单显示了自定义`BeanPostProcessor`实现类定义：

```java
package scripting;

import org.springframework.beans.factory.config.BeanPostProcessor;

public class InstantiationTracingBeanPostProcessor implements BeanPostProcessor {

    // simply return the instantiated bean as-is
    public Object postProcessBeforeInitialization(Object bean, String beanName) {
        return bean; // we could potentially return any object reference here...
    }

    public Object postProcessAfterInitialization(Object bean, String beanName) {
        System.out.println("Bean '" + beanName + "' created : " + bean.toString());
        return bean;
    }
}
```

以下`beans`元素使用`InstantiationTracingBeanPostProcessor`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:lang="http://www.springframework.org/schema/lang"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/lang
        https://www.springframework.org/schema/lang/spring-lang.xsd">

    <lang:groovy id="messenger"
            script-source="classpath:org/springframework/scripting/groovy/Messenger.groovy">
        <lang:property name="message" value="Fiona Apple Is Just So Dreamy."/>
    </lang:groovy>

    <!--
    when the above bean (messenger) is instantiated, this custom
    BeanPostProcessor implementation will output the fact to the system console
    -->
    <bean class="scripting.InstantiationTracingBeanPostProcessor"/>

</beans>
```

请注意`InstantiationTracingBeanPostProcessor`是如何定义的。它甚至没有名字，而且，因为它是一个 bean，它可以像任何其他 bean 一样被依赖注入。（前面的配置还定义了一个由 Groovy 脚本支持的 bean。Spring 动态语言支持在“ [动态语言支持](https://docs.spring.io/spring-framework/docs/current/reference/html/languages.html#dynamic-language)”一章中有详细说明。）

以下 Java 应用程序运行上述代码和配置：

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.scripting.Messenger;

public final class Boot {

    public static void main(final String[] args) throws Exception {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("scripting/beans.xml");
        Messenger messenger = ctx.getBean("messenger", Messenger.class);
        System.out.println(messenger);
    }

}
```

上述应用程序的输出类似于以下内容：

```
Bean“信使”创建：org.springframework.scripting.groovy.GroovyMessenger@272961 
org.springframework.scripting.groovy.GroovyMessenger@272961
```

**示例：`AutowiredAnnotationBeanPostProcessor`**

将回调接口或注解与自定义`BeanPostProcessor` 实现结合使用是扩展 Spring IoC 容器的常用方法。一个例子是 Spring 的`AutowiredAnnotationBeanPostProcessor` ——一个`BeanPostProcessor`随 Spring 发行版一起提供的实现，并自动连接带注解的字段、setter 方法和任意配置方法。

**1.8.2. 自定义配置元数据`BeanFactoryPostProcessor`**

我们要看的下一个扩展点是 `org.springframework.beans.factory.config.BeanFactoryPostProcessor`. 此接口的语义与 的语义相似，但`BeanPostProcessor`有一个主要区别：`BeanFactoryPostProcessor`对 bean 配置元数据进行操作。也就是说，Spring IoC 容器允许`BeanFactoryPostProcessor`读取配置元数据并可能在容器实例化除实例之外的任何 bea&#x6E;_&#x4E4B;前_`BeanFactoryPostProcessor`更改它。

您可以配置多个实例，并且可以通过设置属性`BeanFactoryPostProcessor`来控制这些`BeanFactoryPostProcessor`实例的运行顺序。`order`但是，您只能在`BeanFactoryPostProcessor`实现 `Ordered`接口时设置此属性。如果你自己写`BeanFactoryPostProcessor`，你也应该考虑实现`Ordered`接口。[`BeanFactoryPostProcessor`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/beans/factory/config/BeanFactoryPostProcessor.html) 有关更多详细信息，请参阅和[`Ordered`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/core/Ordered.html)接口的 javadoc 。

如果您想更改实际的 bean 实例（即从配置元数据创建的对象），那么您需要使用 a `BeanPostProcessor` （前面在[使用 a 自定义 Bean`BeanPostProcessor`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-extension-bpp)中进行了描述）。虽然在技术上可以在 a 中使用 bean 实例`BeanFactoryPostProcessor`（例如，通过使用 `BeanFactory.getBean()`），但这样做会导致 bean 过早实例化，从而违反标准容器生命周期。这可能会导致负面影响，例如绕过 bean 后处理。此外，`BeanFactoryPostProcessor`实例的作用域是每个容器。这仅在您使用容器层次结构时才相关。如果您在一个容器中定义 `BeanFactoryPostProcessor`，它仅适用于该容器中的 bean 定义。一个容器中的 Bean 定义不会由另一个容器中的`BeanFactoryPostProcessor`实例进行后处理，即使两个容器都属于同一层次结构。

bean 工厂后处理器在 `ApplicationContext` 中声明时会自动运行，以便将更改应用于定义容器的配置元数据。Spring 包括许多预定义的 bean factory 后处理器，例如`PropertyOverrideConfigurer`和 `PropertySourcesPlaceholderConfigurer`. 您还可以使用自定义`BeanFactoryPostProcessor` - 例如，注册自定义属性编辑器。

`ApplicationContext`自动检测部署到其中实现`BeanFactoryPostProcessor`接口的任何 bean。它在适当的时候将这些 bean 用作 bean 工厂后处理器。您可以像部署任何其他 bean 一样部署这些后处理器 bean。

与`BeanPostProcessor`s 一样，您通常不希望将 `BeanFactoryPostProcessor`s 配置为延迟初始化。如果没有其他 bean 引用 `Bean(Factory)PostProcessor`，则该后处理器根本不会被实例化。因此，将其标记为延迟初始化将被忽略，并且 `Bean(Factory)PostProcessor`即使您 在元素`<beans />` 的声明中将`default-lazy-init`属性设置为`true`，也会立即实例化。

**示例：类名替换`PropertySourcesPlaceholderConfigurer`**

您可以使用标准 Java格式`PropertySourcesPlaceholderConfigurer`将 bean 定义中的属性值外部化到单独的`Properties`文件中。这样做使部署应用程序的人员能够自定义特定于环境的属性，例如数据库 URL 和密码，而无需修改容器的主要 XML 定义文件或文件的复杂性或风险。

考虑以下基于 XML 的配置元数据片段，其中 定义了带有占位符值的`DataSource`：

```xml
<bean class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">
    <property name="locations" value="classpath:com/something/jdbc.properties"/>
</bean>

<bean id="dataSource" destroy-method="close"
        class="org.apache.commons.dbcp.BasicDataSource">
    <property name="driverClassName" value="${jdbc.driverClassName}"/>
    <property name="url" value="${jdbc.url}"/>
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>
```

该示例显示了从外部`Properties`文件配置的属性。在运行时，将 a`PropertySourcesPlaceholderConfigurer`应用于替换 DataSource 的某些属性的元数据。要替换的值被指定为表单的占位符`${property-name}`，它遵循 Ant 和 log4j 以及 JSP EL 样式。

实际值来自另一个标准 Java`Properties`格式的文件：

```
jdbc.driverClassName=org.hsqldb.jdbcDriver
jdbc.url=jdbc:hsqldb:hsql://production:9002
jdbc.username=sa
jdbc.password=root
```

因此，该`${jdbc.username}`字符串在运行时被值“sa”替换，同样适用于与属性文件中的键匹配的其他占位符值。检查 bean 定义的大多数属性和属性中的`PropertySourcesPlaceholderConfigurer`占位符。此外，您可以自定义占位符前缀和后缀。

使用Spring 2.5 中引入的命名空间`context`，您可以使用专用配置元素配置属性占位符。您可以在属性中以逗号分隔列表的形式提供一个或多个位置`location`，如以下示例所示：

```xml
<context:property-placeholder location="classpath:com/something/jdbc.properties"/>
```

`PropertySourcesPlaceholderConfigurer`不仅在您指定的文件中查找属性`Properties` 。默认情况下，如果在指定的属性文件中找不到属性，它会检查 Spring`Environment`属性和常规 Java`System`属性。

您可以使用`PropertySourcesPlaceholderConfigurer`替换类名，当您必须在运行时选择特定的实现类时，这有时很有用。以下示例显示了如何执行此操作：

```xml
<bean class="org.springframework.beans.factory.config.PropertySourcesPlaceholderConfigurer">
	<property name="locations">
		<value>classpath:com/something/strategy.properties</value>
	</property>
	<property name="properties">
		<value>custom.strategy.class=com.something.DefaultStrategy</value>
	</property>
</bean>

<bean id="serviceStrategy" class="${custom.strategy.class}"/>
```

如果该类在运行时无法解析为有效类，则该 bean 在即将创建时解析失败，这是在`ApplicationContext` 非惰性初始化 bean的`preInstantiateSingletons()` 阶段。

**示例：`PropertyOverrideConfigurer`**

另一个 bean 工厂后处理器`PropertyOverrideConfigurer`，类似于`PropertySourcesPlaceholderConfigurer` ，但与后者不同的是，原始定义可以具有默认值或根本没有 bean 属性的值。如果覆盖 `Properties`文件没有特定 bean 属性的条目，则使用默认上下文定义。

请注意，bean 定义不知道被覆盖，因此从 XML 定义文件中不能立即看出正在使用覆盖配置器。如果有多个`PropertyOverrideConfigurer`实例为同一个 bean 属性定义不同的值，由于覆盖机制，最后一个会获胜。

属性文件配置行采用以下格式：

```
beanName.property=值
```

以下清单显示了格式的示例：

```
dataSource.driverClassName=com.mysql.jdbc.Driver
dataSource.url=jdbc:mysql:mydb
```

这个示例文件可以与一个容器定义一起使用，该容器定义包含一个名为的 bean ，该 `dataSource`bean具有`driver`和`url`属性。

还支持复合属性名称，只要路径的每个组件（除了要覆盖的最终属性）都已经非空（可能由构造函数初始化）。在以下示例中，将 bean `tom`的属性`fred`的属性`bob`的`sammy`属性设置为标量值`123`：

```
tom.fred.bob.sammy=123
```

指定的覆盖值始终是文字值。它们不会被翻译成 bean 引用。当 XML bean 定义中的原始值指定 bean 引用时，该约定也适用。

使用Spring 2.5 中引入的命名空间`context`，可以使用专用配置元素配置属性覆盖，如以下示例所示：

```xml
<context:property-override location="classpath:override.properties"/>
```

**1.8.3. 自定义实例化逻辑`FactoryBean`**

您可以为本身是工厂的对象实现接口`org.springframework.beans.factory.FactoryBean`。

该`FactoryBean`接口是 Spring IoC 容器的实例化逻辑的可插入点。如果您有复杂的初始化代码，用 Java 更好地表达而不是（可能）冗长的 XML，您可以创建自己的 `FactoryBean`，在该类中编写复杂的初始化，然后将您的自定义`FactoryBean`插入容器中。

该`FactoryBean<T>`接口提供了三种方法：

* `T getObject()`：返回此工厂创建的对象的实例。该实例可能会被共享，具体取决于该工厂是返回单例还是原型。
* `boolean isSingleton()`：`true`如果`FactoryBean`返回单例或 `false`其他，则返回。此方法的默认实现返回`true`.
* `Class<?> getObjectType()`：返回`getObject()`方法返回的对象类型，或者`null`如果事先不知道类型。

Spring `FactoryBean`框架中的许多地方都使用了概念和接口。Spring `FactoryBean`本身提供了超过 50 个接口的实现。

当您需要向容器请求实际的 FactoryBean 实例本身而不是它生成的 bean 时，请在调用 ApplicationContext 的 getBean() 方法时在 bean 的 id 前加上与号 (&) 前缀。因此，对于 id 为 myBean 的给定 FactoryBean，在容器上调用 getBean("myBean") 将返回 FactoryBean 的乘积，而调用 getBean("\&myBean") 将返回 FactoryBean 实例本身。

#### 1.9. 基于注解的容器配置

在配置 Spring 时，注解是否比 XML 更好？

基于注解的配置的引入提出了这种方法是否比 XML“更好”的问题。简短的回答是“视情况而定”。长答案是每种方法都有其优点和缺点，通常由开发人员决定哪种策略更适合他们。由于它们的定义方式，注解在其声明中提供了大量上下文，从而使配置更短、更简洁。然而，XML 擅长在不触及源代码或重新编译它们的情况下连接组件。一些开发人员更喜欢在源附近进行布线，而另一些开发人员则认为带注解的类不再是 POJO，此外，配置变得分散且更难控制。

无论选择如何，Spring 都可以同时适应这两种风格，甚至可以将它们混合在一起。值得指出的是，通过其[JavaConfig](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java)选项，Spring 允许以非侵入性的方式使用注解，而无需触及目标组件的源代码，并且在工具方面， [Spring Tools for Eclipse](https://spring.io/tools)支持所有配置样式。

基于注解的配置提供了 XML 设置的替代方案，它依赖于字节码元数据来连接组件，而不是尖括号声明。开发人员不使用 XML 来描述 bean 连接，而是通过在相关类、方法或字段声明上使用注解将配置移动到组件类本身。如[示例中所述：`AutowiredAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-extension-bpp-examples-aabpp) ,`BeanPostProcessor`与注解一起使用是扩展 Spring IoC 容器的常用方法。例如，Spring 2.0 引入了使用[`@Required`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-required-annotation)注解强制执行所需属性的可能性。Spring 2.5 使得遵循相同的通用方法来驱动 Spring 的依赖注入成为可能。本质上，`@Autowired`[annotation 提供了与Autowiring Collaborators](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-autowire)中描述的相同的功能，但具有更细粒度的控制和更广泛的适用性。Spring 2.5 还增加了对 JSR-250 注解的支持，例如 `@PostConstruct`和`@PreDestroy`. Spring 3.0 增加了对包中包含的 JSR-330（Java 依赖注入）注解的支持，`javax.inject`例如`@Inject` 和`@Named`. 有关这些注解的详细信息，请参见 [相关部分](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-standard-annotations)。

注解注入在 XML 注入之前执行。因此，XML 配置覆盖了通过这两种方法连接的属性的注解。

与往常一样，您可以将后处理器注册为单独的 bean 定义，但也可以通过在基于 XML 的 Spring 配置中包含以下标记来隐式注册它们（注意包含`context`命名空间）：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

</beans>
```

该`<context:annotation-config/>`元素隐式注册以下后处理器：

* [`ConfigurationClassPostProcessor`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/context/annotation/ConfigurationClassPostProcessor.html)
* [`AutowiredAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/beans/factory/annotation/AutowiredAnnotationBeanPostProcessor.html)
* [`CommonAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/context/annotation/CommonAnnotationBeanPostProcessor.html)
* [`PersistenceAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/orm/jpa/support/PersistenceAnnotationBeanPostProcessor.html)
* [`EventListenerMethodProcessor`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/context/event/EventListenerMethodProcessor.html)

`<context:annotation-config/>`仅在定义它的同一应用程序上下文中查找 bean 上的注解。这意味着，如果您在为 `DispatcherServlet`所属的`WebApplicationContext`中配置 `<context:annotation-config/>` ，它只会检查您的controllers中的`@Autowired` bean，而不是您的服务。有关详细信息，请参阅 [DispatcherServlet](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#mvc-servlet)。

**1.9.1. @Required**

`@Required`注解适用于 bean 属性设置方法，如下例所示：

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Required
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

此注解指示必须在配置时通过 bean 定义中的显式属性值或通过自动装配来填充受影响的 bean 属性。如果受影响的 bean 属性尚未填充，则容器将引发异常。这允许急切和明确`NullPointerException` 的失败，避免以后出现实例等。我们仍然建议您将断言放入 bean 类本身（例如放入 init 方法）。即使您在容器外部使用类，这样做也会强制执行这些必需的引用和值。

必须将其[`RequiredAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/beans/factory/annotation/RequiredAnnotationBeanPostProcessor.html) 注册为 bean 以启用对`@Required`注解的支持。

从Spring Framework 5.1 开始正式弃用`@Required`注解 和`RequiredAnnotationBeanPostProcessor`，支持使用构造函数注入进行所需设置（或自定义实现`InitializingBean.afterPropertiesSet()` 或自定义`@PostConstruct`方法以及 bean 属性设置方法）。

**1.9.2. 使用`@Autowired`**

在本节包含的示例中，`@Inject`可以使用JSR 330 的注解代替 Spring 的注解。`@Autowired`有关更多详细信息，请参见[此处](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-standard-annotations)。

您可以将`@Autowired`注解应用于构造函数，如以下示例所示：

```java
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

从 Spring Framework 4.3 开始，如果目标 bean 仅定义一个构造函数开始，则不再需要对此类构造函数进行`@Autowired`注解。但是，如果有多个构造函数可用并且没有主/默认构造函数，则必须至少对其中一个构造函数进行`@Autowired`注解，以指示容器使用哪一个构造函数。[有关详细信息，请参阅构造函数解析](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-autowired-annotation-constructor-resolution)的讨论 。

您还可以将`@Autowired`注解应用&#x4E8E;_&#x4F20;&#x7EDF;_&#x7684;setter 方法，如以下示例所示：

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

您还可以将注解应用于具有任意名称和多个参数的方法，如以下示例所示：

```java
public class MovieRecommender {

    private MovieCatalog movieCatalog;

    private CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public void prepare(MovieCatalog movieCatalog,
            CustomerPreferenceDao customerPreferenceDao) {
        this.movieCatalog = movieCatalog;
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

您也可以应用于`@Autowired`字段，甚至可以将其与构造函数混合使用，如以下示例所示：

```java
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    private MovieCatalog movieCatalog;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

确保您的目标组件（例如，`MovieCatalog`或`CustomerPreferenceDao`）始终由您用于带`@Autowired`注解的注入点的类型声明。否则，注入可能会由于运行时出现“找不到类型匹配”错误而失败。对于通过类路径扫描找到的 XML 定义的 bean 或组件类，容器通常预先知道具体类型。但是，对于`@Bean`工厂方法，您需要确保声明的返回类型具有足够的表现力。对于实现多个接口的组件或可能由其实现类型引用的组件，请考虑在您的工厂方法中声明最具体的返回类型（至少与引用您的 bean 的注入点所要求的一样具体）。

您还可以通过将`@Autowired`注解添加到需要该类型数组的字段或方法来指示 Spring`ApplicationContext` 提供特定类型的所有 bean ，如以下示例所示：

```java
public class MovieRecommender {

    @Autowired
    private MovieCatalog[] movieCatalogs;

    // ...
}
```

这同样适用于类型化集合，如以下示例所示：

```java
public class MovieRecommender {

    private Set<MovieCatalog> movieCatalogs;

    @Autowired
    public void setMovieCatalogs(Set<MovieCatalog> movieCatalogs) {
        this.movieCatalogs = movieCatalogs;
    }

    // ...
}
```

如果您希望数组或列表中的项目按特定顺序排序，您的目标 bean 可以实现`org.springframework.core.Ordered`接口或使用`@Order`或标准`@Priority`注解。否则，它们的顺序遵循容器中相应目标 bean 定义的注册顺序。您可以`@Order`在目标类级别和`@Bean`方法上声明注解，可能针对单个 bean 定义（在使用相同 bean 类的多个定义的情况下）。`@Order`值可能会影响注入点的优先级，但请注意它们不会影响单例启动顺序，这是由依赖关系和`@DependsOn`声明确定的正交问题。请注意，标准`javax.annotation.Priority`注解在该 `@Bean`级别不可用，因为它不能在方法上声明。它的语义可以通过`@Order`结合`@Primary`每个类型的单个 bean 的值来建模。

只要预期的键类型是`String` ，即使是类型化的`Map`实例也可以自动装配。映射值包含预期类型的所有 bean，键包含相应的 bean 名称，如以下示例所示：

```java
public class MovieRecommender {

    private Map<String, MovieCatalog> movieCatalogs;

    @Autowired
    public void setMovieCatalogs(Map<String, MovieCatalog> movieCatalogs) {
        this.movieCatalogs = movieCatalogs;
    }

    // ...
}
```

默认情况下，当给定注入点没有匹配的候选 bean 时，自动装配会失败。在声明的数组、集合或映射的情况下，至少需要一个匹配元素。

默认行为是将带注解的方法和字段视为指示所需的依赖项。您可以按照以下示例所示更改此行为，使框架能够通过将其标记为非必需（即，通过将`required`属性设置`@Autowired`为`false`）来跳过不可满足的注入点：

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired(required = false)
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

如果非必需方法的依赖项（或其依赖项之一，如果有多个参数）不可用，则根本不会调用非必需方法。在这种情况下，根本不会填充非必填字段，而保留其默认值。

注入的构造函数和工厂方法参数是一种特殊情况，因为Spring 的构造函数解析算法可能会处理多个构造函数，因此`@Autowired`中的`required` 属性的含义有些不同。默认情况下，构造函数和工厂方法参数是有效的，但在单构造函数场景中有一些特殊规则，例如如果没有匹配的 bean 可用，多元素注入点（数组、集合、映射）解析为空实例。这允许一种通用的实现模式，其中所有依赖项都可以在唯一的多参数构造函数中声明——例如，声明为没有`@Autowired`注解的单个公共构造函数。

任何给定 bean 类中只有一个构造函数可以声明 @Autowired，并将 required 属性设置为 true，指示该构造函数在用作 Spring bean 时自动装配。因此，如果 required 属性保留其默认值 true，则只能使用 @Autowired 注解单个构造函数。如果多个构造函数声明该注释，则它们都必须声明 required=false 才能被视为自动装配的候选者（类似于 XML 中的 autowire=constructor）。将选择具有最大数量的依赖关系的构造函数，这些依赖关系可以通过匹配 Spring 容器中的 bean 来满足。如果没有一个候选可以满足，则将使用主要/默认构造函数（如果存在）。类似地，如果一个类声明了多个构造函数，但没有一个构造函数用 @Autowired 注释，则将使用主/默认构造函数（如果存在）。如果一个类一开始只声明一个构造函数，那么即使没有注释，它也将始终被使用。请注意，带注释的构造函数不必是公共的。

或者，您可以通过 Java 8 表达特定依赖项的非必需性质`java.util.Optional`，如以下示例所示：

```java
public class SimpleMovieLister {

    @Autowired
    public void setMovieFinder(Optional<MovieFinder> movieFinder) {
        ...
    }
}
```

从 Spring Framework 5.0 开始，您还可以使用`@Nullable`注解（任何包中的任何类型 - 例如，`javax.annotation.Nullable`来自 JSR-305）或仅利用 Kotlin 内置的空安全支持：

```java
public class SimpleMovieLister {

    @Autowired
    public void setMovieFinder(@Nullable MovieFinder movieFinder) {
        ...
    }
}
```

您还可以`@Autowired`用于众所周知的可解析依赖项的接口：`BeanFactory`、`ApplicationContext`、`Environment`、`ResourceLoader`、 `ApplicationEventPublisher`和`MessageSource`. 这些接口及其扩展接口，例如`ConfigurableApplicationContext`或 `ResourcePatternResolver`，会自动解析，无需特殊设置。以下示例自动装配一个`ApplicationContext`对象：

```java
public class MovieRecommender {

    @Autowired
    private ApplicationContext context;

    public MovieRecommender() {
    }

    // ...
}
```

`@Resource`、`@Autowired`、`@Inject`和`@Value`注解由 Spring`BeanPostProcessor` 实现处理。这意味着您不能在您自己的`BeanPostProcessor`或类型`BeanFactoryPostProcessor`（如果有）中应用这些注解。这些类型必须通过使用 XML 或 Spring `@Bean` 方法显式“连接”起来。

**1.9.3. 微调基于注解的自动装配`@Primary`**

由于按类型自动装配可能会导致多个候选者，因此通常需要对选择过程进行更多控制。实现这一点的一种方法是使用 Spring 的 `@Primary`注解。`@Primary`指示当多个 bean 是自动装配到单值依赖项的候选对象时，应该优先考虑特定的 bean。如果候选中恰好存在一个主 bean，则它将成为自动装配的值。

考虑以下定义`firstMovieCatalog`为主要的配置`MovieCatalog`：

```java
@Configuration
public class MovieConfiguration {

    @Bean
    @Primary
    public MovieCatalog firstMovieCatalog() { ... }

    @Bean
    public MovieCatalog secondMovieCatalog() { ... }

    // ...
}
```

使用上述配置，以下`MovieRecommender`内容与 自动装配 `firstMovieCatalog`：

```java
public class MovieRecommender {

    @Autowired
    private MovieCatalog movieCatalog;

    // ...
}
```

对应的bean定义如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog" primary="true">
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```

**1.9.4. 使用限定符微调基于注解的自动装配**

当可以确定一个主要候选者时，是一种通过类型`@Primary`使用多个实例的自动装配的有效方法。当您需要对选择过程进行更多控制时，可以使用 Spring 的`@Qualifier`注解。您可以将限定符值与特定参数相关联，缩小类型匹配的作用域，以便为每个参数选择特定的 bean。在最简单的情况下，这可以是一个简单的描述性值，如以下示例所示：

```java
public class MovieRecommender {

    @Autowired
    @Qualifier("main")
    private MovieCatalog movieCatalog;

    // ...
}
```

您还可以在单个构造函数参数或方法参数上指定`@Qualifier`注解，如下例所示：

```java
public class MovieRecommender {

    private MovieCatalog movieCatalog;

    private CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public void prepare(@Qualifier("main") MovieCatalog movieCatalog,
            CustomerPreferenceDao customerPreferenceDao) {
        this.movieCatalog = movieCatalog;
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

以下示例显示了相应的 bean 定义。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog">
        <qualifier value="main"/> 

        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier value="action"/> 

        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```

对于回调匹配，bean 名称被视为默认限定符值。因此，您可以使用`main`的`id`代替嵌套的限定符元素来定义 bean ，从而获得相同的匹配结果。但是，尽管您可以使用此约定按名称引用特定 bean，但从`@Autowired`根本上讲，它是关于带有可选语义限定符的类型驱动注入。这意味着限定符值，即使使用 bean 名称回退，也总是在类型匹配集中具有缩小的语义。它们不会在语义上表达对唯一 bean `id`的引用。好的限定符值是`main` 或 `EMEA`或 `persistent`，表示独立于 bean `id`的特定组件的特征，在匿名 bean 定义（例如前面示例中的那个）的情况下，它可能会自动生成。

如前所述，限定符也适用于类型化集合——例如，对`Set<MovieCatalog>`. 在这种情况下，根据声明的限定符，所有匹配的 bean 都作为集合注入。这意味着限定符不必是唯一的。相反，它们构成过滤标准。例如，您可以定义多个`MovieCatalog`具有相同限定符值“action”的 bean，所有这些 bean 都被注入到带有`@Qualifier("action")`的`Set<MovieCatalog>`.

在类型匹配的候选对象中，让限定符值针对目标 bean 名称进行选择，不需要在注入点进行`@Qualifier`注解。如果没有其他解析指标（例如限定符或主标记），对于非唯一依赖情况，Spring 将注入点名称（即字段名称或参数名称）与目标 bean 名称匹配并选择同名候选人（如有）。

也就是说，如果您打算按名称表示注解驱动的注入，请不要主要使用`@Autowired`，即使它能够在类型匹配候选者中按 bean 名称进行选择。相反，使用 JSR-250`@Resource`注解，它在语义上定义为通过其唯一名称标识特定目标组件，声明的类型与匹配过程无关。`@Autowired`具有相当不同的语义：在按类型选择候选 bean 之后，指定的`String` 限定符值仅在那些类型选择的候选者中考虑（例如，将`account`限定符与标记有相同限定符标签的 bean 匹配）。

对于本身定义为集合`Map`或数组类型的 bean，这`@Resource` 是一个很好的解决方案，通过唯一名称引用特定的集合或数组 bean。也就是说，从 4.3 开始，您也可以通过 Spring 的`@Autowired`类型匹配算法匹配集合、`Map`和数组类型 ，只要元素类型信息保留在`@Bean`返回类型签名或集合继承层次结构中即可。在这种情况下，您可以使用限定符值在相同类型的集合中进行选择，如上一段所述。

从 4.3 开始，`@Autowired`还考虑了注入的自引用（即，对当前注入的 bean 的引用）。请注意，自注入是一种后备。对其他组件的常规依赖始终具有优先权。从这个意义上说，自我参考不参与常规的候选人选择，因此尤其不是主要的。相反，它们总是以最低优先级结束。在实践中，您应该仅将自引用用作最后的手段（例如，通过 bean 的事务代理在同一实例上调用其他方法）。在这种情况下，考虑将受影响的方法分解为单独的委托 bean。或者，您可以使用`@Resource`，它可以通过其唯一名称获取返回到当前 bean 的代理。

尝试从`@Bean`同一配置类上的方法注入结果实际上也是一种自引用场景。要么在实际需要的方法签名中延迟解析此类引用（与配置类中的自动装配字段相反），要么将受影响的`@Bean`方法声明为`static`，将它们与包含的配置类实例及其生命周期解耦。否则，仅在回退阶段考虑此类 bean，而将其他配置类上的匹配 bean 选为主要候选者（如果可用）。

`@Autowired`适用于字段、构造函数和多参数方法，允许在参数级别通过限定符注解缩小作用域。相反，`@Resource` 仅支持具有单个参数的字段和 bean 属性设置器方法。因此，如果您的注入目标是构造函数或多参数方法，您应该坚持使用限定符。

您可以创建自己的自定义限定符注解。为此，请定义注解并`@Qualifier`在定义中提供注解，如以下示例所示：

```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Genre {

    String value();
}
```

然后，您可以在自动装配的字段和参数上提供自定义限定符，如以下示例所示：

```java
public class MovieRecommender {

    @Autowired
    @Genre("Action")
    private MovieCatalog actionCatalog;

    private MovieCatalog comedyCatalog;

    @Autowired
    public void setComedyCatalog(@Genre("Comedy") MovieCatalog comedyCatalog) {
        this.comedyCatalog = comedyCatalog;
    }

    // ...
}
```

接下来，您可以提供候选 bean 定义的信息。您可以添加 `<qualifier/>`标签作为`<bean/>`标签的子元素，然后指定`type`和`value`以匹配您的自定义限定符注解。该类型与注解的完全限定类名匹配。或者，如果不存在名称冲突的风险，为方便起见，您可以使用短类名。以下示例演示了这两种方法：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="Genre" value="Action"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="example.Genre" value="Comedy"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```

在[Classpath Scanning and Managed Components](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-classpath-scanning)中，您可以看到基于注解的替代方法，以在 XML 中提供限定符元数据。具体来说，请参阅[提供带有注解的限定符元数据](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-scanning-qualifiers)。

在某些情况下，使用没有值的注解可能就足够了。当注解服务于更通用的目的并且可以应用于多种不同类型的依赖项时，这可能很有用。例如，您可以提供一个离线目录，当没有可用的 Internet 连接时可以搜索该目录。首先，定义简单的注解，如下例所示：

```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Offline {

}
```

然后将注解添加到要自动装配的字段或属性中，如下例所示：

```java
public class MovieRecommender {

    @Autowired
    @Offline 
    private MovieCatalog offlineCatalog;

    // ...
}
```

现在 bean 定义只需要一个 qualifier `type`，如下例所示：

```xml
<bean class="example.SimpleMovieCatalog">
    <qualifier type="Offline"/> 
    <!-- inject any dependencies required by this bean -->
</bean>
```

除了`value`或代替简单属性，您还可以定义接受命名属性的自定义限定符注解。如果随后在要自动装配的字段或参数上指定多个属性值，则 bean 定义必须匹配所有此类属性值才能被视为自动装配候选者。例如，考虑以下注解定义：

```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface MovieQualifier {

    String genre();

    Format format();
}
```

在这种情况下`Format`是一个枚举，定义如下：

```java
public enum Format {
    VHS, DVD, BLURAY
}
```

要自动装配的字段使用自定义限定符进行注解，并包括两个属性的值：`genre`和`format`，如以下示例所示：

```java
public class MovieRecommender {

    @Autowired
    @MovieQualifier(format=Format.VHS, genre="Action")
    private MovieCatalog actionVhsCatalog;

    @Autowired
    @MovieQualifier(format=Format.VHS, genre="Comedy")
    private MovieCatalog comedyVhsCatalog;

    @Autowired
    @MovieQualifier(format=Format.DVD, genre="Action")
    private MovieCatalog actionDvdCatalog;

    @Autowired
    @MovieQualifier(format=Format.BLURAY, genre="Comedy")
    private MovieCatalog comedyBluRayCatalog;

    // ...
}
```

最后，bean 定义应该包含匹配的限定符值。此示例还演示了您可以使用 bean 元属性而不是 `<qualifier/>`元素。如果可用，则`<qualifier/>`元素及其属性优先，但 如果不存在此类限定符，则自动装配机制将依赖`<meta/>`标签中提供的值，如以下示例中的最后两个 bean 定义：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="MovieQualifier">
            <attribute key="format" value="VHS"/>
            <attribute key="genre" value="Action"/>
        </qualifier>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="MovieQualifier">
            <attribute key="format" value="VHS"/>
            <attribute key="genre" value="Comedy"/>
        </qualifier>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <meta key="format" value="DVD"/>
        <meta key="genre" value="Action"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <meta key="format" value="BLURAY"/>
        <meta key="genre" value="Comedy"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

</beans>
```

**1.9.5. 使用泛型作为自动装配限定符**

除了`@Qualifier`注解之外，您还可以使用 Java 泛型类型作为限定的隐式形式。例如，假设您有以下配置：

```java
@Configuration
public class MyConfiguration {

    @Bean
    public StringStore stringStore() {
        return new StringStore();
    }

    @Bean
    public IntegerStore integerStore() {
        return new IntegerStore();
    }
}
```

假设前面的 bean 实现了一个泛型接口，（即`Store<String>`和 `Store<Integer>`），您可以`@Autowire`将`Store`接口和泛型用作限定符，如以下示例所示：

```java
@Autowired
private Store<String> s1; // <String> qualifier, injects the stringStore bean

@Autowired
private Store<Integer> s2; // <Integer> qualifier, injects the integerStore bean
```

通用限定符也适用于自动装配列表、`Map`实例和数组。以下示例自动装配一个泛型`List`：

```java
// Inject all Store beans as long as they have an <Integer> generic
// Store<String> beans will not appear in this list
@Autowired
private List<Store<Integer>> s;
```

**1.9.6. 使用`CustomAutowireConfigurer`**

[`CustomAutowireConfigurer`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/beans/factory/annotation/CustomAutowireConfigurer.html) 是一个`BeanFactoryPostProcessor`允许您注册自己的自定义限定符注解类型，即使它们没有使用 Spring 的注解进行`@Qualifier`注解。下面的例子展示了如何使用`CustomAutowireConfigurer`：

```xml
<bean id="customAutowireConfigurer"
        class="org.springframework.beans.factory.annotation.CustomAutowireConfigurer">
    <property name="customQualifierTypes">
        <set>
            <value>example.CustomQualifier</value>
        </set>
    </property>
</bean>
```

通过以下`AutowireCandidateResolver`方式确定自动接线候选者：

* `autowire-candidate`每个bean定义的值
* 元素`default-autowire-candidates`上可用的任何模式`<beans/>`
* 注解的存在`@Qualifier`和任何注册的自定义注解`CustomAutowireConfigurer`

当多个 bean 有资格成为自动装配候选者时，“主要”的确定如下：如果候选者中恰好一个 bean 定义的`primary` 属性设置为`true`，则选择它。

**1.9.7. 注射用`@Resource`**

Spring 还通过在字段或 bean 属性设置器方法上使用 JSR-250`@Resource`注解 ( )来支持注入。`javax.annotation.Resource`这是 Java EE 中的常见模式：例如，在 JSF 管理的 bean 和 JAX-WS 端点中。Spring 也支持 Spring 管理的对象的这种模式。

`@Resource`采用名称属性。默认情况下，Spring 将该值解释为要注入的 bean 名称。换句话说，它遵循按名称语义，如以下示例所示：

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Resource(name="myMovieFinder") 
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

如果没有明确指定名称，则默认名称派生自字段名称或 setter 方法。如果是字段，则采用字段名称。对于 setter 方法，它采用 bean 属性名称。以下示例将把名为 bean 的 bean`movieFinder`注入到它的 setter 方法中：

```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Resource
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

随注解提供的名称被解析为 bean 名称，由 bean `ApplicationContext`知道`CommonAnnotationBeanPostProcessor`。如果显式配置 Spring，则可以通过 JNDI 解析名称 [`SimpleJndiBeanFactory`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/jndi/support/SimpleJndiBeanFactory.html) 。但是，我们建议您依赖默认行为并使用 Spring 的 JNDI 查找功能来保留间接级别。

在没有指定显式名称的排他性`@Resource`使用情况下，类似于`@Autowired`，`@Resource`查找主类型匹配而不是特定的命名 bean 并解析众所周知的可解析依赖项：`BeanFactory`、 `ApplicationContext`、`ResourceLoader`、`ApplicationEventPublisher`和`MessageSource` 接口。

因此，在以下示例中，该`customerPreferenceDao`字段首先查找名为“customerPreferenceDao”的 bean，然后回退到 type 的主要类型匹配 `CustomerPreferenceDao`：

```java
public class MovieRecommender {

    @Resource
    private CustomerPreferenceDao customerPreferenceDao;

    @Resource
    private ApplicationContext context; 

    public MovieRecommender() {
    }

    // ...
}
```

该`context`字段是根据已知的可解析依赖类型注入的： `ApplicationContext`.

**1.9.8. 使用`@Value`**

`@Value`通常用于注入外部属性：

```java
@Component
public class MovieRecommender {

    private final String catalog;

    public MovieRecommender(@Value("${catalog.name}") String catalog) {
        this.catalog = catalog;
    }
}
```

使用以下配置：

```java
@Configuration
@PropertySource("classpath:application.properties")
public class AppConfig { }
```

以及以下`application.properties`文件：

```java
catalog.name=MovieCatalog
```

在这种情况下，`catalog`参数和字段将等于该`MovieCatalog`值。

Spring 提供了一个默认的宽松嵌入式值解析器。它将尝试解析属性值，如果无法解析，属性名称（例如`${catalog.name}`）将作为值注入。如果要严格控制不存在的值，则应声明一个`PropertySourcesPlaceholderConfigurer`bean，如以下示例所示：

```java
@Configuration
public class AppConfig {

    @Bean
    public static PropertySourcesPlaceholderConfigurer propertyPlaceholderConfigurer() {
        return new PropertySourcesPlaceholderConfigurer();
    }
}
```

配置`PropertySourcesPlaceholderConfigurer`使用 JavaConfig 时， `@Bean`方法必须是`static`.

`${}` 如果无法解析任何占位符，使用上述配置可确保 Spring 初始化失败。也可以使用 `setPlaceholderPrefix`, `setPlaceholderSuffix`, 或`setValueSeparator`自定义占位符等方法。

Spring Boot 默认配置一个`PropertySourcesPlaceholderConfigurer`bean，该 bean 将从`application.properties`和`application.yml`文件中获取属性。

Spring 提供的内置转换器支持允许自动处理简单的类型转换（to`Integer` 或example）。`int`多个逗号分隔的值可以自动转换为`String`数组，无需额外的努力。

可以提供如下默认值：

```java
@Component
public class MovieRecommender {

    private final String catalog;

    public MovieRecommender(@Value("${catalog.name:defaultCatalog}") String catalog) {
        this.catalog = catalog;
    }
}
```

Spring在后台`BeanPostProcessor`使用 a来处理将值转换为目标类型的过程。如果您想为您自己的自定义类型提供转换支持，您可以提供您自己的 bean 实例，如以下示例所示：`ConversionService``String``@Value``ConversionService`

```java
@Configuration
public class AppConfig {

    @Bean
    public ConversionService conversionService() {
        DefaultFormattingConversionService conversionService = new DefaultFormattingConversionService();
        conversionService.addConverter(new MyCustomConverter());
        return conversionService;
    }
}
```

当`@Value`包含[`SpEL`表达式](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#expressions)时，该值将在运行时动态计算，如以下示例所示：

```java
@Component
public class MovieRecommender {

    private final String catalog;

    public MovieRecommender(@Value("#{systemProperties['user.catalog'] + 'Catalog' }") String catalog) {
        this.catalog = catalog;
    }
}
```

SpEL 还支持使用更复杂的数据结构：

```java
@Component
public class MovieRecommender {

    private final Map<String, Integer> countOfMoviesPerCatalog;

    public MovieRecommender(
            @Value("#{{'Thriller': 100, 'Comedy': 300}}") Map<String, Integer> countOfMoviesPerCatalog) {
        this.countOfMoviesPerCatalog = countOfMoviesPerCatalog;
    }
}
```

**1.9.9. 使用`@PostConstruct`和`@PreDestroy`**

`CommonAnnotationBeanPostProcessor`不仅可以识别`@Resource`注解，还可以识别 JSR-250 生命周期注解：`javax.annotation.PostConstruct`和 `javax.annotation.PreDestroy`. [在 Spring 2.5 中引入，对这些注解的支持提供了初始化回调](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-initializingbean)和 [销毁回调](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-disposablebean)中描述的生命周期回调机制的替代方案 。如果 在 Spring `ApplicationContext`中注册了`CommonAnnotationBeanPostProcessor`，则在生命周期中与对应的 Spring 生命周期接口方法或显式声明的回调方法相同的点调用带有这些注解之一的方法。在以下示例中，缓存在初始化时预先填充并在销毁时清除：

```java
public class CachingMovieLister {

    @PostConstruct
    public void populateMovieCache() {
        // populates the movie cache upon initialization...
    }

    @PreDestroy
    public void clearMovieCache() {
        // clears the movie cache upon destruction...
    }
}
```

组合各种生命周期机制的效果的详细信息，请参见 [组合生命周期机制](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-combined-effects)。

就像`@Resource`，`@PostConstruct`和`@PreDestroy`注解类型是从 JDK 6 到 8 的标准 Java 库的一部分。但是，整个`javax.annotation` 包在 JDK 9 中与核心 Java 模块分离，并最终在 JDK 11 中被删除。如果需要，`javax.annotation-api`工件需要现在通过 Maven Central 获得，只需像任何其他库一样添加到应用程序的类路径中。

#### 1.10. 类路径扫描和管理组件

本章中的大多数示例都使用 XML 来指定在 Spring 容器内生成每个 `BeanDefinition` 的配置元数据。上一节（[基于注解的容器配置](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-annotation-config)) 演示如何通过source级注解提供大量配置元数据。然而，即使在这些示例中，“基本”bean 定义也在 XML 文件中明确定义，而注解仅驱动依赖注入。本节描述了通过扫描类路径隐式检测候选组件的选项。候选组件是与过滤条件匹配的类，并在容器中注册了相应的 bean 定义。这消除了使用 XML 来执行 bean 注册的需要。相反，您可以使用注解（例如`@Component`）、AspectJ 类型表达式或您自己的自定义过滤条件来选择哪些类具有向容器注册的 bean 定义。

从 Spring 3.0 开始，Spring JavaConfig 项目提供的许多特性都是核心 Spring Framework 的一部分。这允许您使用 Java 而不是使用传统的 XML 文件来定义 bean。查看`@Configuration`、`@Bean`、 `@Import`和`@DependsOn`注解，了解如何使用这些新功能的示例。

**1.10.1. `@Component`和进一步的原型注解**

注解`@Repository`是满足存储库（也称为数据访问对象或 DAO）角色或原型的任何类的标记。此标记的用途之一是异常的自动翻译，如 [Exception Translation](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#orm-exception-translation)中所述。

Spring 提供了更多的原型注解：`@Component`、`@Service`和 `@Controller`. `@Component`是任何 Spring 管理的组件的通用构造型。 `@Repository`, `@Service`和`@Controller`是`@Component`针对更具体用例（分别在持久层、服务层和表示层）的特化。因此，您可以使用 `@Component` 注解组件类，但是通过使用 `@Repository`、`@Service` 或者`@Controller`注解它们，您的类更适合工具处理或与切面关联。例如，这些原型注解是切入点的理想目标。`@Repository`, `@Service`, 和`@Controller`还可以在 Spring 框架的未来版本中携带额外的语义。因此，如果您在对于你的服务层使用`@Component`或者`@Service`，`@Service`显然是更好的选择。同样，如前所述，`@Repository`已经支持作为持久层中自动异常转换的标记。

**1.10.2. 使用元注解和组合注解**

Spring 提供的许多注解都可以在您自己的代码中用作元注解。元注解是可以应用于另一个注解的注解。例如，前面提到的[注解](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-stereotype-annotations)`@Service`是 用`@Component` 元注解的，如以下示例所示：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component 
public @interface Service {

    // ...
}
```

您还可以组合元注解来创建“组合注解”。例如，Spring MVC 的`@RestController`注解由`@Controller`和`@ResponseBody` 组成。

此外，组合注解可以选择从元注解中重新声明属性以允许自定义。当您只想公开元注解属性的子集时，这可能特别有用。例如，Spring 的 `@SessionScope`注解将作用域名称硬编码为`session`，但仍允许自定义`proxyMode`. 以下清单显示了 `SessionScope`注解的定义：

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Scope(WebApplicationContext.SCOPE_SESSION)
public @interface SessionScope {

    /**
     * Alias for {@link Scope#proxyMode}.
     * <p>Defaults to {@link ScopedProxyMode#TARGET_CLASS}.
     */
    @AliasFor(annotation = Scope.class)
    ScopedProxyMode proxyMode() default ScopedProxyMode.TARGET_CLASS;

}
```

然后，您可以在`@SessionScope`不声明`proxyMode`如下的情况下使用：

```java
@Service
@SessionScope
public class SessionScopedService {
    // ...
}
```

您还可以覆盖`proxyMode` 的值，如以下示例所示：

```java
@Service
@SessionScope(proxyMode = ScopedProxyMode.INTERFACES)
public class SessionScopedUserService implements UserService {
    // ...
}
```

有关更多详细信息，请参阅 [Spring Annotation Programming Model](https://github.com/spring-projects/spring-framework/wiki/Spring-Annotation-Programming-Model) wiki 页面。

**1.10.3. 自动检测类和注册 Bean 定义**

Spring可以自动检测构造型类并向`ApplicationContext`注册相应的`BeanDefinition`实例。 例如，以下两个类可以进行此类自动检测：

```java
@Service
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

```java
@Repository
public class JpaMovieFinder implements MovieFinder {
    // implementation elided for clarity
}
```

要自动检测这些类并注册相应的 bean，您需要添加 `@ComponentScan`到您的`@Configuration`类中，其中`basePackages`属性是两个类的公共父包。（或者，您可以指定一个逗号或分号或空格分隔的列表，其中包括每个类的父包。）

```java
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig  {
    // ...
}
```

为简洁起见，前面的示例可能使用了注解的`value`属性（即`@ComponentScan("org.example")`）。

以下替代方法使用 XML：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="org.example"/>

</beans>
```

使用`<context:component-scan>`会隐式启用 `<context:annotation-config>`. 使用`<context:component-scan>`时通常不需要包含该 `<context:annotation-config>`元素。

类路径包的扫描需要类路径中存在相应的目录条目。当您使用 Ant 构建 JAR 时，请确保您没有激活 JAR 任务的仅文件开关。此外，在某些环境中，类路径目录可能不会根据安全策略公开——例如，JDK 1.7.0\_45 及更高版本上的独立应用程序（需要在清单中设置“Trusted-Library”——请参阅 [https://stackoverflow.com/问题/19394570/java-jre-7u45-breaks-classloader-getresources](https://stackoverflow.com/questions/19394570/java-jre-7u45-breaks-classloader-getresources)）。

在 JDK 9 的模块路径（Jigsaw）上，Spring 的类路径扫描通常按预期工作。但是，请确保您的组件类在您的`module-info` 描述符中导出。如果您希望 Spring 调用类的非公共成员，请确保它们是“开放的”（即，它们使用`opens`声明而不是`module-info`描述符中的 `exports`声明）。

此外，当您使用 component-scan 元素时， `AutowiredAnnotationBeanPostProcessor`和`CommonAnnotationBeanPostProcessor`都被隐式包含在内。这意味着这两个组件会被自动检测并连接在一起——所有这些都不需要 XML 中提供任何 bean 配置元数据。

您可以禁用注册`AutowiredAnnotationBeanPostProcessor`和`CommonAnnotationBeanPostProcessor`通过设置属性`annotation-config`值为`false` 。

**1.10.4. 使用过滤器自定义扫描**

默认情况下，使用`@Component`、`@Repository`、`@Service`、`@Controller`、 `@Configuration`注解的类或本身带有`@Component`注解的自定义注解是唯一检测到的候选组件。但是，您可以通过应用自定义过滤器来修改和扩展此行为。将它们添加为 `@ComponentScan` 注释的 `includeFilters` 或 `exceptFilters` 属性（或者作为 XML 配置中`<context:component-scan>`元素的`<context:include-filter />` 或 `<context:exclude-filter />` 子元素）。每个过滤器元素都需要类型和表达式属性。下表描述了过滤选项：

| 过滤器类型   | 示例表达式                        | 描述                                                                 |
| ------- | ---------------------------- | ------------------------------------------------------------------ |
| 注解（默认）  | `org.example.SomeAnnotation` | 在目标组件的类型级&#x522B;_&#x5B58;&#x5728;_&#x6216;_元存&#x5728;_&#x7684;注解。 |
| 可分配的    | `org.example.SomeClass`      | 目标组件可分配（扩展或实现）的类（或接口）。                                             |
| aspectj | `org.example..*Service+`     | 要由目标组件匹配的 AspectJ 类型表达式。                                           |
| 正则表达式   | `org\.example\.Default.*`    | 与目标组件的类名匹配的正则表达式。                                                  |
| 自定义     | `org.example.MyTypeFilter`   | 接口的自定义实现`org.springframework.core.type.TypeFilter`。                |

以下示例显示了忽略所有`@Repository`注解并使用“存根”存储库的配置：

```java
@Configuration
@ComponentScan(basePackages = "org.example",
        includeFilters = @Filter(type = FilterType.REGEX, pattern = ".*Stub.*Repository"),
        excludeFilters = @Filter(Repository.class))
public class AppConfig {
    // ...
}
```

以下清单显示了等效的 XML：

```xml
<beans>
    <context:component-scan base-package="org.example">
        <context:include-filter type="regex"
                expression=".*Stub.*Repository"/>
        <context:exclude-filter type="annotation"
                expression="org.springframework.stereotype.Repository"/>
    </context:component-scan>
</beans>
```

您还可以通过在注解上设置`useDefaultFilters=false`或`use-default-filters="false"`作为 `<component-scan/>`元素的属性提供来禁用默认过滤器。这有效地禁用了用`@Component`, `@Repository`, `@Service`, `@Controller`, `@RestController`, 或`@Configuration`注解或元注解的类的自动检测。

**1.10.5. 在组件中定义 Bean 元数据**

Spring 组件还可以将 bean 定义元数据贡献给容器。您可以使用`@Bean`用于在带 `@Configuration`注解的类中定义 bean 元数据的相同注解来执行此操作。以下示例显示了如何执行此操作：

```java
@Component
public class FactoryMethodComponent {

    @Bean
    @Qualifier("public")
    public TestBean publicInstance() {
        return new TestBean("publicInstance");
    }

    public void doWork() {
        // Component method implementation omitted
    }
}
```

> 前面的类是一个 Spring 组件，它的 `doWork()`方法中包含特定于应用程序的代码。但是，它还提供了一个 bean 定义，该定义具有引用方法的工厂方法`publicInstance()`。`@Bean`注解标识工厂方法和其他 bean 定义属性，例如通过注解`@Qualifier`的限定符值。可以指定的其他方法级注解是 `@Scope`,`@Lazy`和自定义限定符注解。
>
> 除了用于组件初始化之外，您还可以将 `@Lazy` 注解放置在标有 `@Autowired` 或 `@Inject` 的注入点上。在这种情况下，它会导致注入惰性解析代理。然而，这种代理方法相当有限。对于复杂的惰性交互，特别是与可选依赖项结合使用，我们建议改为使用 `ObjectProvider<MyTargetBean>`。

如前所述，支持自动装配的字段和方法，并额外支持`@Bean`方法的自动装配。以下示例显示了如何执行此操作：

```java
@Component
public class FactoryMethodComponent {

    private static int i;

    @Bean
    @Qualifier("public")
    public TestBean publicInstance() {
        return new TestBean("publicInstance");
    }

    // use of a custom qualifier and autowiring of method parameters
    @Bean
    protected TestBean protectedInstance(
            @Qualifier("public") TestBean spouse,
            @Value("#{privateInstance.age}") String country) {
        TestBean tb = new TestBean("protectedInstance", 1);
        tb.setSpouse(spouse);
        tb.setCountry(country);
        return tb;
    }

    @Bean
    private TestBean privateInstance() {
        return new TestBean("privateInstance", i++);
    }

    @Bean
    @RequestScope
    public TestBean requestScopedInstance() {
        return new TestBean("requestScopedInstance", 3);
    }
}
```

该示例将方法`String`类型的参数 `country`自动连接到 另一个名为`privateInstance`的 bean 上的`age`属性值。Spring 表达式语言元素通过`#{ <expression> }`表示法定义属性的值。对于`@Value` 注解，表达式解析器被预先配置为在解析表达式文本时查找 bean 名称。

从 Spring Framework 4.3 开始，您还可以声明类型 `InjectionPoint`（或其更具体的子类：`DependencyDescriptor`）的工厂方法参数来访问触发当前 bean 创建的请求注入点。请注意，这仅适用于 bean 实例的实际创建，不适用于现有实例的注入。因此，此功能对于原型范围的 bean 最有意义。对于其他范围，工厂方法只看到在给定范围内触发创建新 bean 实例的注入点（例如，触发创建惰性单例 bean 的依赖项）。在这种情况下，您可以使用提供的带有语义关怀的注入点元数据。下面的例子展示了如何使用`InjectionPoint`：

```java
@Component
public class FactoryMethodComponent {

    @Bean @Scope("prototype")
    public TestBean prototypeInstance(InjectionPoint injectionPoint) {
        return new TestBean("prototypeInstance for " + injectionPoint.getMember());
    }
}
```

常规 Spring 组件中的方法的处理方式与 Spring `@Configuration`类中`@Bean`的对应方法不同。不同之处在于`@Component` 类没有通过 CGLIB 增强来拦截方法和字段的调用。CGLIB 代理是调用`@Configuration`类`@Bean`方法中的方法或字段创建协作对象的 bean 元数据引用的方法。这样的方法不是用普通的 Java 语义调用的，而是通过容器来提供 Spring bean 的通常的生命周期管理和代理，即使通过对`@Bean`方法的编程调用来引用其他 bean 也是如此。相比之下，在普通 `@Component` 类中调用`@Bean`方法中的方法或字段具有标准 Java 语义，无需特殊的 CGLIB 处理或其他约束。

您可以将`@Bean`方法声明为`static`，允许在不创建包含它们的配置类作为实例的情况下调用它们。 这在定义后处理器 bean（例如，类型`BeanFactoryPostProcessor`或 `BeanPostProcessor`）时特别有意义，因为这些 bean 在容器生命周期的早期就被初始化，并且应该避免在那个时候触发配置的其他部分。

由于技术限制，对静态`@Bean`方法的调用永远不会被容器拦截，甚至在 `@Configuration`类中也不会（如本节前面所述）：CGLIB 子类化只能覆盖非静态方法。因此，直接调用另一个具有标准的 Java 语义的`@Bean`方法，从而导致直接从工厂方法本身返回一个独立的实例。

`@Bean`方法的 Java 语言可见性不会立即影响 Spring 容器中生成的 bean 定义。您可以自由地声明您认为适合非`@Configuration`类的工厂方法，也可以在任何地方声明静态方法。但是，类中的常规`@Bean`方法`@Configuration`需要是可覆盖的——也就是说，它们不能被声明为`private`或 `final`。

`@Bean`方法也在给定组件或配置类的基类上发现，以及在组件或配置类实现的接口中声明的 Java 8 默认方法上发现。这为组合复杂的配置安排提供了很大的灵活性，甚至可以通过 Spring 4.2 的 Java 8 默认方法实现多重继承。

最后，单个类可以为同一个 bean 创建`@Bean`多个方法，作为多个工厂方法的排列，根据运行时可用的依赖关系使用。这与在其他配置场景中选择“最贪婪”的构造函数或工厂方法的算法相同：在构造时选择具有最多可满足依赖项的变体，类似于容器如何在多个`@Autowired`构造函数之间进行选择。

**1.10.6. 命名自动检测到的组件**

当一个组件作为扫描过程的一部分被自动检测到时，它的 Bean 名称由该扫描器已知的 `BeanNameGenerator` 策略生成。默认情况下，任何包含名称值的 Spring 构造型注释（`@Component`、`@Repository`、`@Service` 和 `@Controller`）都会将该名称提供给相应的 bean 定义。

如果这样的注解不包含名称`value`或任何其他检测到的组件（例如由自定义过滤器发现的组件），则默认 bean 名称生成器将返回未大写的非限定类名称。例如，如果检测到以下组件类，则名称为`myMovieLister`和 `movieFinderImpl`：

```java
@Service("myMovieLister")
public class SimpleMovieLister {
    // ...
}
```

```java
@Repository
public class MovieFinderImpl implements MovieFinder {
    // ...
}
```

如果您不想依赖默认的 bean 命名策略，可以提供自定义 bean 命名策略。首先，实现 [`BeanNameGenerator`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/beans/factory/support/BeanNameGenerator.html) 接口，并确保包含一个默认的无参数构造函数。然后，在配置扫描器时提供完全限定的类名，如以下示例注解和 bean 定义所示。

如果由于多个自动检测到的组件具有相同的非限定类名（即，具有相同名称但位于不同包中的类）而遇到命名冲突，您可能需要配置`BeanNameGenerator`默认为生成的完全限定类名的Bean名。从 Spring Framework 5.2.3 开始， 位于`org.springframework.context.annotation`包中的`FullyQualifiedAnnotationBeanNameGenerator`可用于此类目的。

```java
@Configuration
@ComponentScan(basePackages = "org.example", nameGenerator = MyNameGenerator.class)
public class AppConfig {
    // ...
}
<beans>
    <context:component-scan base-package="org.example"
        name-generator="org.example.MyNameGenerator" />
</beans>
```

作为一般规则，只要其他组件可能显式引用它，请考虑使用注解指定名称。另一方面，只要容器负责接线，自动生成的名称就足够了。

**1.10.7. 为自动检测的组件提供作用域**

与一般 Spring 管理的组件一样，自动检测组件的默认和最常见作用域是`singleton`. 但是，有时您需要可以由`@Scope`注解指定的不同作用域。您可以在注解中提供作用域的名称，如以下示例所示：

```java
@Scope("prototype")
@Repository
public class MovieFinderImpl implements MovieFinder {
    // ...
}
```

`@Scope`注解仅在具体 bean 类（用于注解组件）或工厂方法（用于`@Bean`方法）上进行自省。与 XML bean 定义相比，没有 bean 定义继承的概念，并且类级别的继承层次结构与元数据无关。

有关 Web 特定范围的详细信息，例如 Spring 上下文中的“请求”或“会话”，请参阅[请求、会话、应用程序和 WebSocket 范围](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-other)。与这些范围的预构建注解一样，您也可以使用 Spring 的元注解方法来编写自己的范围注解：例如，使用元注解`@Scope("prototype")`的自定义注解，也可能声明自定义范围代理模式。

要为范围解析提供自定义策略而不是依赖基于注解的方法，您可以实现该 [`ScopeMetadataResolver`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/context/annotation/ScopeMetadataResolver.html) 接口。确保包含一个默认的无参数构造函数。然后，您可以在配置扫描器时提供完全限定的类名，如以下注解和 bean 定义示例所示：

```java
@Configuration
@ComponentScan(basePackages = "org.example", scopeResolver = MyScopeResolver.class)
public class AppConfig {
    // ...
}
<beans>
    <context:component-scan base-package="org.example" scope-resolver="org.example.MyScopeResolver"/>
</beans>
```

当使用某些非单例作用域时，可能需要为作用域对象生成代理。原因在[Scoped Beans as Dependencies](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-other-injection)中进行了描述。为此，component-scan 元素上提供了 scoped-proxy 属性。三个可能的值是：`no`、`interfaces`和`targetClass`。例如，以下配置会生成标准 JDK 动态代理：

```java
@Configuration
@ComponentScan(basePackages = "org.example", scopedProxy = ScopedProxyMode.INTERFACES)
public class AppConfig {
    // ...
}
<beans>
    <context:component-scan base-package="org.example" scoped-proxy="interfaces"/>
</beans>
```

**1.10.8. 提供带有注解的限定符元数据**

`@Qualifier`注解在 [Fine-tuning Annotation-based Autowiring with Qualifiers](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-autowired-annotation-qualifiers)中讨论。该部分中的示例演示了使用`@Qualifier`注解和自定义限定符注解在解析自动装配候选时提供细粒度控制。因为这些示例基于 XML bean 定义，所以通过使用 XML 中`bean`元素的`qualifier`或`meta` 子元素在候选 bean 定义上提供限定符元数据。当依赖类路径扫描来自动检测组件时，您可以在候选类上为限定符元数据提供类型级别的注解。以下三个示例演示了这种技术：

```java
@Component
@Qualifier("Action")
public class ActionMovieCatalog implements MovieCatalog {
    // ...
}
```

```java
@Component
@Genre("Action")
public class ActionMovieCatalog implements MovieCatalog {
    // ...
}
```

```java
@Component
@Offline
public class CachingMovieCatalog implements MovieCatalog {
    // ...
}
```

与大多数基于注解的替代方案一样，请记住注解元数据绑定到类定义本身，而 XML 的使用允许相同类型的多个 bean 提供其限定符元数据的变体，因为元数据是根据每个-实例而不是每个类。

**1.10.9. 生成候选组件的索引**

虽然类路径扫描非常快，但可以通过在编译时创建静态候选列表来提高大型应用程序的启动性能。在这种模式下，作为组件扫描目标的所有模块都必须使用这种机制。

您现有的`@ComponentScan`或`<context:component-scan/>`指令必须保持不变，才能请求上下文以扫描某些包中的候选人。当 `ApplicationContext`检测到这样的下标时，它会自动使用它而不是扫描类路径。

要生成索引，请向每个包含作为组件扫描指令目标的组件的模块添加一个附加依赖项。以下示例显示了如何使用 Maven 执行此操作：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context-indexer</artifactId>
        <version>5.3.22</version>
        <optional>true</optional>
    </dependency>
</dependencies>
```

对于 Gradle 4.5 及更早版本，应在`compileOnly` 配置中声明依赖项，如以下示例所示：

```groovy
dependencies {
    compileOnly "org.springframework:spring-context-indexer:5.3.22"
}
```

对于 Gradle 4.6 及更高版本，应在`annotationProcessor` 配置中声明依赖项，如下例所示：

```groovy
dependencies {
    annotationProcessor "org.springframework:spring-context-indexer:5.3.22"
}
```

`spring-context-indexer`artifact 会生成一个包含在 jar 文件中的`META-INF/spring.components`文件。

在 IDE 中使用此模式时，`spring-context-indexer`必须将其注册为注解处理器，以确保更新候选组件时索引是最新的。

当在类路径中找到`META-INF/spring.components`文件时，索引会自动启用。如果索引对某些库（或用例）部分可用，但无法为整个应用程序构建，您可以通过设置`spring.index.ignore`为`true`（作为JVM 系统属性或通过 [`SpringProperties`](https://docs.spring.io/spring-framework/docs/current/reference/html/appendix.html#appendix-spring-properties)机制）来回退到常规类路径安排（好像根本不存在索引）

#### 1.11. 使用 JSR 330 标准注解

从 Spring 3.0 开始，Spring 提供对 JSR-330 标准注解（依赖注入）的支持。这些注解的扫描方式与 Spring 注解相同。要使用它们，您需要在类路径中有相关的 jar。

如果您使用 Maven，则`javax.inject`工件在标准 Maven 存储库 ( https://repo1.maven.org/maven2/javax/inject/javax.inject/1/ ) 中可用。您可以将以下依赖项添加到文件 pom.xml 中：

```xml
<dependency>
    <groupId>javax.inject</groupId>
    <artifactId>javax.inject</artifactId>
    <version>1</version> 
</dependency>
```

**1.11.1. `@Inject`和`@Named`依赖注入**

相对于`@Autowired`，您可以使用`@javax.inject.Inject`如下：

```java
import javax.inject.Inject;

public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    public void listMovies() {
        this.movieFinder.findMovies(...);
        // ...
    }
}
```

与 `@Autowired`一样，您可以在字段级别、方法级别和构造函数参数级别使用`@Inject`。此外，您可以将注入点声明为 `Provider`，从而允许按需访问范围更短的 bean 或通过`Provider.get()`调用延迟访问其他 bean。以下示例提供了前面示例的变体：

```java
import javax.inject.Inject;
import javax.inject.Provider;

public class SimpleMovieLister {

    private Provider<MovieFinder> movieFinder;

    @Inject
    public void setMovieFinder(Provider<MovieFinder> movieFinder) {
        this.movieFinder = movieFinder;
    }

    public void listMovies() {
        this.movieFinder.get().findMovies(...);
        // ...
    }
}
```

如果您想为应该注入的依赖项使用限定名称，则应使用`@Named`注解，如以下示例所示：

```java
import javax.inject.Inject;
import javax.inject.Named;

public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(@Named("main") MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

与`@Autowired`相同,`@Inject`也可以与`java.util.Optional`或 `@Nullable`一起使用。`@Inject`在这里更适用，因为没有`required`属性。以下一对示例展示了如何使用`@Inject`和 `@Nullable`：

```java
public class SimpleMovieLister {

    @Inject
    public void setMovieFinder(Optional<MovieFinder> movieFinder) {
        // ...
    }
}
```

```java
public class SimpleMovieLister {

    @Inject
    public void setMovieFinder(@Nullable MovieFinder movieFinder) {
        // ...
    }
}
```

**1.11.2.`@Named`和`@ManagedBean`注解：`@Component`的标准等效项**

您可以使用`@javax.inject.Named`或`javax.annotation.ManagedBean`代替`@Component`，如以下示例所示：

```java
import javax.inject.Inject;
import javax.inject.Named;

@Named("movieListener")  // @ManagedBean("movieListener") could be used as well
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

在不指定组件名称的情况下 使用`@Component`是很常见的。可以以类似的方式使用`@Named`，如以下示例所示：

```java
import javax.inject.Inject;
import javax.inject.Named;

@Named
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

当您使用`@Named`或`@ManagedBean`时，您可以使用与使用 Spring 注解时完全相同的方式使用组件扫描，如以下示例所示：

```java
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig  {
    // ...
}
```

与 `@Component`相比，JSR-330`@Named`和 JSR-250`@ManagedBean` 注解是不可组合的。您应该使用 Spring 的原型模型来构建自定义组件注解。

**1.11.3. JSR-330 标准注解的限制**

使用标准注解时，您应该知道某些重要功能不可用，如下表所示：

| spring              | javax.inject.\*       | javax.inject 限制/注释                                                                                                                                                                                                                                                             |
| ------------------- | --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| @Autowired          | @Inject               | `@Inject`没有“required”属性。可以与 Java 8 `Optional`一起使用。                                                                                                                                                                                                                             |
| @Component          | @Named / @ManagedBean | JSR-330 不提供可组合模型，仅提供一种识别命名组件的方法。                                                                                                                                                                                                                                               |
| @Scope("singleton") | @Singleton            | JSR-330 默认范围类似于 Spring 的`prototype`. 但是，为了保持它与 Spring 的一般默认值一致，在 Spring 容器中声明的 JSR-330 bean 是默认的`singleton`。为了使用`singleton` 以外的范围，您应该使用 Spring 的`@Scope`注解。`javax.inject`还提供了一个 [@Scope](https://download.oracle.com/javaee/6/api/javax/inject/Scope.html)注解。然而，这个仅用于创建您自己的注解。 |
| @Qualifier          | @Qualifier / @Named   | `javax.inject.Qualifier`只是用于构建自定义限定符的元注解。具体`String`的限定符（如`@Qualifier`带有值的 Spring）可以通过`javax.inject.Named`.                                                                                                                                                                     |
| @Value              | -                     | 没有等价物                                                                                                                                                                                                                                                                          |
| @Lazy               | -                     | 没有等价物                                                                                                                                                                                                                                                                          |
| ObjectFactory       | Provider              | `javax.inject.Provider`是 Spring `ObjectFactory`的直接替代品，只是`get()`方法名称更短。它还可以与 Spring`@Autowired`或未注解的构造函数和 setter 方法结合使用。                                                                                                                                                        |

#### 1.12. 基于 Java 的容器配置

本节介绍如何在 Java 代码中使用注解来配置 Spring 容器。它包括以下主题：

* [基本概念：`@Bean`和`@Configuration`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java-basic-concepts)
* [通过使用实例化 Spring 容器`AnnotationConfigApplicationContext`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java-instantiating-container)
* [使用`@Bean`注解](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java-bean-annotation)
* [使用`@Configuration`注解](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java-configuration-annotation)
* [组合基于 Java 的配置](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java-composing-configuration-classes)
* [Bean 定义配置文件](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-definition-profiles)
* [`PropertySource`抽象](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-property-source-abstraction)
* [使用`@PropertySource`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-using-propertysource)
* [语句中的占位符解析](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-placeholder-resolution-in-statements)

**1.12.1. 基本概念：`@Bean`和`@Configuration`**

Spring 新的 Java 配置支持中的核心工件是带 `@Configuration`注解的类和带`@Bean`注解的方法。

`@Bean`注解用于表示一个方法实例化、配置和初始化一个由 Spring IoC 容器管理的新对象。对于熟悉 Spring 的XML `<beans/>`配置的人来说，注解`@Bean`与元素`<bean/>`的作用相同。您可以将`@Bean`-annotated 方法与任何 Spring `@Component`一起 使用。但是，它们最常与`@Configuration`Bean类一起使用。

用`@Configuration` 注解一个类表明它的主要目的是作为 bean 定义的来源。此外，`@Configuration`类允许通过调用`@Bean`同一类中的其他方法来定义 bean 间的依赖关系。最简单的`@Configuration`类如下所示：

```java
@Configuration
public class AppConfig {

    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```

前面的`AppConfig`类等价于下面的 Spring `<beans/>`XML：

```xml
<beans>
    <bean id="myService" class="com.acme.services.MyServiceImpl"/>
</beans>
```

完整的@Configuration 与“精简”@Bean 模式？

> 当`@Bean`方法在没有用`@Configuration` 注解的类中声明时 ，它们被称为以“精简”模式处理。在一个或什至在一个普通的旧类中声明的 Bean 方法`@Component`被认为是“精简版”，包含类的不同主要目的和一种`@Bean`方法在那里是一种奖励。例如，服务组件可以通过`@Bean`每个适用组件类上的附加方法向容器公开管理视图。在这种情况下，`@Bean`方法是一种通用的工厂方法机制。
>
> 与全配置`@Configuration` 不同，轻量的`@Bean`方法不能声明 bean 间的依赖关系。相反，它们对其包含组件的内部状态进行操作，并且可以选择对它们可能声明的参数进行操作。因此，`@Bean`方法不应调用其他 `@Bean`方法。每个这样的方法实际上只是特定 bean 引用的工厂方法，没有任何特殊的运行时语义。这里的积极副作用是在运行时不必应用 CGLIB 子类化，因此在类设计方面没有限制（即包含类可能是`final`等等）。
>
> 在常见情况下，`@Bean`方法将在`@Configuration`类中声明，确保始终使用“完整”模式，并且跨方法引用因此被重定向到容器的生命周期管理。这可以防止 `@Bean`通过常规 Java 调用意外调用相同的方法，这有助于减少在“精简”模式下操作时难以追踪的细微错误。

以下部分将深入讨论`@Bean`和`@Configuration`注解。然而，首先，我们介绍了使用基于 Java 的配置创建 Spring 容器的各种方法。

**1.12.2. 通过使用`AnnotationConfigApplicationContext`实例化 Spring 容器**

以下部分记录了 Spring 3.0 中引入的 Spring `AnnotationConfigApplicationContext`。这种通用`ApplicationContext`的实现不仅能够接受 `@Configuration`类作为输入，还能够接受普通`@Component`类和使用 JSR-330 元数据注解的类。

当`@Configuration`类作为输入提供时，`@Configuration`类本身被注册为 bean 定义，并且`@Bean`类中所有声明的方法也被注册为 bean 定义。

当`@Component`和 JSR-330 类被提供时，它们被注册为 bean 定义，并且假定 DI 元数据，如`@Autowired`或`@Inject`在必要时在这些类中使用。

**简单的构造**

与实例化 `ClassPathXmlApplicationContext` 时使用 Spring XML 文件作为输入的方式大致相同，您可以在实例化`AnnotationConfigApplicationContext`时使用 @Configuration 类作为输入如以下示例所示：

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

如前所述，`AnnotationConfigApplicationContext`不仅限于使用`@Configuration`类。任何`@Component`或 JSR-330 注解类都可以作为输入提供给构造函数，如以下示例所示：

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(MyServiceImpl.class, Dependency1.class, Dependency2.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

前面的示例假定`MyServiceImpl`、`Dependency1`和`Dependency2`使用 Spring 依赖注入注解，例如`@Autowired`.

**通过使用`register(Class<?>…)`以编程方式构建容器**

您可以使用无参数构造函数实例化一个`AnnotationConfigApplicationContext`，然后使用`register()`方法对其进行配置。这种方法在以编程方式构建`AnnotationConfigApplicationContext`. 以下示例显示了如何执行此操作：

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.register(AppConfig.class, OtherConfig.class);
    ctx.register(AdditionalConfig.class);
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

**通过`scan(String…)`启用组件扫描**

要启用组件扫描，您可以如下注解您的`@Configuration`类：

```java
@Configuration
@ComponentScan(basePackages = "com.acme") 
public class AppConfig  {
    // ...
}
```

有经验的 Spring 用户可能熟悉 Spring 的 `context:namespace` 中的等效 XML 声明，如下例所示：：

```xml
<beans>
    <context:component-scan base-package="com.acme"/>
</beans>
```

在前面的示例中，扫描包`com.acme`以查找任何 带`@Component`注解的类，并且这些类在容器中注册为 Spring bean 定义。`AnnotationConfigApplicationContext`暴露了`scan(String…)`方法以允许相同的组件扫描功能，如以下示例所示：

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.scan("com.acme");
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
}
```

请记住，`@Configuration`类是用[元注解](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-meta-annotations) `@Component`的，因此它们是组件扫描的候选对象。在前面的示例中，假设`AppConfig`在`com.acme`包（或下面的任何包）中声明了 ，在调用`scan()`. 在 `refresh()`之后，它的所有`@Bean` 方法都被处理并注册为容器中的 bean 定义。

**使用`AnnotationConfigWebApplicationContext`支持 Web 应用程序**

`WebApplicationContext`的变体`AnnotationConfigApplicationContext`可用于`AnnotationConfigWebApplicationContext`. 您可以在配置 Spring `ContextLoaderListener`servlet 侦听器、Spring MVC `DispatcherServlet`等时使用此实现。以下`web.xml`代码片段配置了一个典型的 Spring MVC Web 应用程序（注意使用`contextClass`的context-param 和 init-param）：

```xml
<web-app>
    <!-- Configure ContextLoaderListener to use AnnotationConfigWebApplicationContext
        instead of the default XmlWebApplicationContext -->
    <context-param>
        <param-name>contextClass</param-name>
        <param-value>
            org.springframework.web.context.support.AnnotationConfigWebApplicationContext
        </param-value>
    </context-param>

    <!-- Configuration locations must consist of one or more comma- or space-delimited
        fully-qualified @Configuration classes. Fully-qualified packages may also be
        specified for component-scanning -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>com.acme.AppConfig</param-value>
    </context-param>

    <!-- Bootstrap the root application context as usual using ContextLoaderListener -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!-- Declare a Spring MVC DispatcherServlet as usual -->
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- Configure DispatcherServlet to use AnnotationConfigWebApplicationContext
            instead of the default XmlWebApplicationContext -->
        <init-param>
            <param-name>contextClass</param-name>
            <param-value>
                org.springframework.web.context.support.AnnotationConfigWebApplicationContext
            </param-value>
        </init-param>
        <!-- Again, config locations must consist of one or more comma- or space-delimited
            and fully-qualified @Configuration classes -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>com.acme.web.MvcConfig</param-value>
        </init-param>
    </servlet>

    <!-- map all requests for /app/* to the dispatcher servlet -->
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/app/*</url-pattern>
    </servlet-mapping>
</web-app>
```

对于编程用例， `GenericWebApplicationContext`可以用作`AnnotationConfigWebApplicationContext`. 有关详细信息，请参阅 [`GenericWebApplicationContext`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/web/context/support/GenericWebApplicationContext.html) javadoc。

**1.12.3. 使用`@Bean`注解**

`@Bean`是方法级别的注解，是 XML`<bean/>`元素的直接模拟。注解支持 提供的一些属性`<bean/>`，例如：

* [初始化方法](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-initializingbean)
* [销毁方法](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-disposablebean)
* [自动装配](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-autowire)
* `name`.

您可以在`@Configuration`-annotated 或 `@Component`-annotated 类中使用`@Bean`注解。

**声明一个 Bean**

要声明一个 bean，你可以用注解来注解一个方法`@Bean`。您可以使用此方法在指定为方法返回值的类型中向`ApplicationContext`注册 bean 定义。默认情况下，bean 名称与方法名称相同。以下示例显示了一个`@Bean`方法声明：

```java
@Configuration
public class AppConfig {

    @Bean
    public TransferServiceImpl transferService() {
        return new TransferServiceImpl();
    }
}
```

前面的配置完全等价于下面的 Spring XML：

```xml
<beans>
    <bean id="transferService" class="com.acme.TransferServiceImpl"/>
</beans>
```

这两个声明都使`ApplicationContext` bean中的可用`transferService` bean绑定到`TransferServiceImpl` type 的对象实例，如以下文本图像所示：

```
transferService -> com.acme.TransferServiceImpl
```

您还可以使用默认方法来定义 bean。这允许通过在默认方法上实现带有 bean 定义的接口来组合 bean 配置。

```java
public interface BaseConfig {

    @Bean
    default TransferServiceImpl transferService() {
        return new TransferServiceImpl();
    }
}

@Configuration
public class AppConfig implements BaseConfig {

}
```

您还可以使用接口（或基类）返回类型声明您的`@Bean`方法，如以下示例所示：

```java
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }
}
```

但是，这会将高级类型预测的可见性限制为指定的接口类型 ( `TransferService`)。然后，只有在实例化受影响的单例 bean 后，容器才知道完整类型 ( `TransferServiceImpl`)。非惰性单例 bean 会根据它们的声明顺序进行实例化，因此您可能会看到不同的类型匹配结果，具体取决于另一个组件何时尝试通过未声明的类型进行匹配（例如`@Autowired TransferServiceImpl`，仅在`transferService`bean 被实例化后才解析）。

如果您始终通过声明的服务接口引用您的类型，则您的 `@Bean`返回类型可以安全地加入该设计决策。但是，对于实现多个接口的组件或可能由其实现类型引用的组件，声明最具体的返回类型可能更安全（至少与引用您的 bean 的注入点所要求的一样具体）。

**Bean 依赖项**

`@Bean`-annotated 方法可以具有任意数量的参数，这些参数描述了构建该 bean 所需的依赖项。例如，如果我们`TransferService` 需要一个`AccountRepository`，我们可以使用方法参数实现该依赖项，如下例所示：

```java
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }
}
```

解析机制与基于构造函数的依赖注入几乎相同。有关详细信息，请参阅[相关部分。](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-constructor-injection)

**接收生命周期回调**

使用`@Bean`注解定义的任何类都支持常规生命周期回调，并且可以使用 JSR-250 中的`@PostConstruct`和`@PreDestroy`注解。有关详细信息，请参阅 [JSR-250 注解](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-postconstruct-and-predestroy-annotations)。

也完全支持常规的 Spring[生命周期回调。](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-nature)如果 bean 实现`InitializingBean`、`DisposableBean`或`Lifecycle`，则容器调用它们各自的方法。

还完全支持标准的`*Aware`接口集（例如[BeanFactoryAware](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-beanfactory)、 [BeanNameAware](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-aware)、 [MessageSourceAware](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#context-functionality-messagesource)、 [ApplicationContextAware](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-aware)等）。

`@Bean`注解支持指定任意初始化和销毁回调方法，很像 Spring XML`init-method`和`bean`元素上的`destroy-method`属性，如以下示例所示：

```java
public class BeanOne {

    public void init() {
        // initialization logic
    }
}

public class BeanTwo {

    public void cleanup() {
        // destruction logic
    }
}

@Configuration
public class AppConfig {

    @Bean(initMethod = "init")
    public BeanOne beanOne() {
        return new BeanOne();
    }

    @Bean(destroyMethod = "cleanup")
    public BeanTwo beanTwo() {
        return new BeanTwo();
    }
}
```

默认情况下，使用 Java 配置定义的具有公共`close`或`shutdown` 方法的 bean 会自动加入销毁回调。如果您有一个公共 `close`或`shutdown`方法并且您不希望在容器关闭时调用它，您可以添加`@Bean(destroyMethod="")`到您的 bean 定义以禁用默认`(inferred)`模式。默认情况下，您可能希望对使用 JNDI 获取的资源执行此操作，因为它的生命周期在应用程序之外进行管理。特别是，请确保始终为`DataSource`.以下示例显示了如何防止 a 的自动销毁回调 `DataSource`：

```java
@Bean(destroyMethod="")
public DataSource dataSource() throws NamingException {
    return (DataSource) jndiTemplate.lookup("MyDS");
}
```

此外，对于`@Bean`方法，您通常使用程序化 JNDI 查找，通过使用 Spring`JndiTemplate`或`JndiLocatorDelegate`帮助程序或直接使用 JNDI `InitialContext`但不使用`JndiObjectFactoryBean`变体（这将迫使您将返回类型声明为`FactoryBean`类型而不是实际的目标类型，从而更难用于其他`@Bean`方法中的交叉引用调用，这些方法旨在引用此处提供的资源）。

在上述示例`BeanOne`的情况下，在构造过程中直接调用该`init()` 方法同样有效，如下例所示：

```java
@Configuration
public class AppConfig {

    @Bean
    public BeanOne beanOne() {
        BeanOne beanOne = new BeanOne();
        beanOne.init();
        return beanOne;
    }

    // ...
}
```

当您直接在 Java 中工作时，您可以对您的对象做任何您喜欢的事情，而不必总是依赖容器生命周期。

**指定 Bean 范围**

Spring 包含`@Scope`注解，以便您可以指定 bean 的范围。

**使用`@Scope`注解**

您可以指定使用`@Bean`注解定义的 bean 应具有特定范围。您可以使用 [Bean Scopes](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes)部分中指定的任何标准范围。

默认范围是`singleton`，但您可以使用`@Scope`注解覆盖它，如以下示例所示：

```java
@Configuration
public class MyConfiguration {

    @Bean
    @Scope("prototype")
    public Encryptor encryptor() {
        // ...
    }
}
```

**`@Scope`和`scoped-proxy`**

Spring 提供了一种通过 [作用域代理](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-other-injection)处理作用域依赖的便捷方式。使用 XML 配置时创建此类代理的最简单方法是`<aop:scoped-proxy/>`元素。使用注解在 Java 中配置您的 bean提供了对属性`@Scope`的等效支持。`proxyMode`默认值为`ScopedProxyMode.DEFAULT`，这通常表示不应创建作用域代理，除非在组件扫描指令级别配置了不同的默认值。您可以 指定`ScopedProxyMode.TARGET_CLASS`、`ScopedProxyMode.INTERFACES`或`ScopedProxyMode.NO`。

如果您将 XML 参考文档中的作用域代理示例（请参阅 [作用域代理](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes-other-injection)）移植到我们使用Java的`@Bean` ，它类似于以下内容：

```java
// an HTTP Session-scoped bean exposed as a proxy
@Bean
@SessionScope
public UserPreferences userPreferences() {
    return new UserPreferences();
}

@Bean
public Service userService() {
    UserService service = new SimpleUserService();
    // a reference to the proxied userPreferences bean
    service.setUserPreferences(userPreferences());
    return service;
}
```

**自定义 Bean 命名**

默认情况下，配置类使用`@Bean`方法的名称作为生成的 bean 的名称。但是，可以使用`name`属性覆盖此功能，如以下示例所示：

```java
@Configuration
public class AppConfig {

    @Bean("myThing")
    public Thing thing() {
        return new Thing();
    }
}
```

**Bean 别名**

正如[命名 Bean](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-beanname)中所讨论的，有时需要为单个 bean 提供多个名称，也称为 bean 别名。注解`@Bean`的`name`属性 为此目的接受一个字符串数组。以下示例显示了如何为 bean 设置多个别名：

```java
@Configuration
public class AppConfig {

    @Bean({"dataSource", "subsystemA-dataSource", "subsystemB-dataSource"})
    public DataSource dataSource() {
        // instantiate, configure and return DataSource bean...
    }
}
```

**Bean描述**

有时，提供更详细的 bean 文本描述会很有帮助。当 bean 被暴露（可能通过 JMX）用于监视目的时，这可能特别有用。

要向`@Bean` 添加描述，您可以使用 [`@Description`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/context/annotation/Description.html) 注解，如以下示例所示：

```java
@Configuration
public class AppConfig {

    @Bean
    @Description("Provides a basic example of a bean")
    public Thing thing() {
        return new Thing();
    }
}
```

**1.12.4. 使用`@Configuration`注解**

`@Configuration`是一个类级别的注解，表明一个对象是 bean 定义的来源。`@Configuration`类通过`@Configuration`-annotated 方法声明 bean 。对`@Configuration`类上的方法的`@Bean`调用也可用于定义 bean 间的依赖关系。请参阅[基本概念：`@Bean`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java-basic-concepts)[和](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java-basic-concepts)`@Configuration`一般介绍。

**注入 内部bean 依赖**

当 bean 相互依赖时，表达这种依赖关系就像让一个 bean 方法调用另一个方法一样简单，如以下示例所示：

```java
@Configuration
public class AppConfig {

    @Bean
    public BeanOne beanOne() {
        return new BeanOne(beanTwo());
    }

    @Bean
    public BeanTwo beanTwo() {
        return new BeanTwo();
    }
}
```

在前面的示例中，通过构造函数注入`beanOne`接收对`beanTwo`的引用。

这种声明 bean 间依赖关系的方法仅在`@Bean`方法在`@Configuration`类中声明时才有效。您不能使用普通`@Component`类来声明 bean 间的依赖关系。

**查找方法注入**

如前所述，[查找方法注入](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-method-injection)是您应该很少使用的高级功能。在单例范围的 bean 依赖于原型范围的 bean 的情况下，它很有用。使用 Java 进行这种类型的配置为实现这种模式提供了一种自然的方式。下面的例子展示了如何使用查找方法注入：

```java
public abstract class CommandManager {
    public Object process(Object commandState) {
        // grab a new instance of the appropriate Command interface
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    // okay... but where is the implementation of this method?
    protected abstract Command createCommand();
}
```

通过使用 Java 配置，您可以创建一个子类，`CommandManager`中抽象`createCommand()`方法被覆盖，从而查找新的（原型）命令对象。以下示例显示了如何执行此操作：

```java
@Bean
@Scope("prototype")
public AsyncCommand asyncCommand() {
    AsyncCommand command = new AsyncCommand();
    // inject dependencies here as required
    return command;
}

@Bean
public CommandManager commandManager() {
    // return new anonymous implementation of CommandManager with createCommand()
    // overridden to return a new prototype Command object
    return new CommandManager() {
        protected Command createCommand() {
            return asyncCommand();
        }
    }
}
```

**有关基于 Java 的配置如何在内部工作的更多信息**

考虑下面的例子，它显示了一个被`@Bean`注解的方法被调用了两次：

```java
@Configuration
public class AppConfig {

    @Bean
    public ClientService clientService1() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());
        return clientService;
    }

    @Bean
    public ClientService clientService2() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());
        return clientService;
    }

    @Bean
    public ClientDao clientDao() {
        return new ClientDaoImpl();
    }
}
```

`clientDao()`已被`clientService1()`调用一次和`clientService2()`调用一次。由于此方法会创建一个新`ClientDaoImpl`实例并返回它，因此您通常会期望有两个实例（每个服务一个实例）。那肯定会有问题：在 Spring 中，实例化的 bean默认有一个`singleton`作用域。这就是神奇之处：所有`@Configuration`类在启动时都使用`CGLIB`. 在子类中，子方法在调用父方法并创建新实例之前，首先检查容器中是否有任何缓存（作用域）bean。

根据 bean 的范围，行为可能会有所不同。我们在这里谈论单例。

从 Spring 3.2 开始，不再需要将 CGLIB 添加到类路径中，因为 CGLIB 类已被重新打包`org.springframework.cglib`并直接包含在 spring-core JAR 中。

由于 CGLIB 在启动时动态添加功能，因此存在一些限制。特别是，配置类不能是最终的。但是，从 4.3 开始，配置类上允许使用任何构造函数，包括使用 `@Autowired`或使用单个非默认构造函数声明进行默认注入。如果您希望避免任何 CGLIB 强加的限制，请考虑 在非`@Configuration`类上声明您的`@Bean`方法（例如，改为在普通`@Component`类上）。方法之间的跨方法调用`@Bean`不会被拦截，因此您必须完全依赖构造函数或方法级别的依赖注入。

**1.12.5. 组合基于 Java 的配置**

Spring 的基于 Java 的配置功能允许您编写注解，这可以降低配置的复杂性。

**使用`@Import`注解**

就像`<import/>`在 Spring XML 文件中使用该元素来帮助模块化配置一样，`@Import`注解允许`@Bean`从另一个配置类加载定义，如以下示例所示：

```java
@Configuration
public class ConfigA {

    @Bean
    public A a() {
        return new A();
    }
}

@Configuration
@Import(ConfigA.class)
public class ConfigB {

    @Bean
    public B b() {
        return new B();
    }
}
```

现在，不需要同时指定`ConfigA.class`和`ConfigB.class`. 在实例化上下文时，只需要显式提供`ConfigB`，如以下示例所示：

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(ConfigB.class);

    // now both beans A and B will be available...
    A a = ctx.getBean(A.class);
    B b = ctx.getBean(B.class);
}
```

这种方法简化了容器的实例化，因为只需要处理一个类，而不是要求您 `@Configuration`在构造过程中记住大量潜在的类。

从 Spring Framework 4.2 开始，`@Import`还支持对常规组件类的引用，类似于`AnnotationConfigApplicationContext.register`方法。如果您想通过使用一些配置类作为入口点来显式定义所有组件来避免组件扫描，这将特别有用。

**`@Bean`注入对导入定义的依赖**

前面的示例有效，但过于简单。在大多数实际场景中，bean 跨配置类相互依赖。使用 XML 时，这不是问题，因为不涉及编译器，您可以声明 `ref="someBean"`并信任 Spring 在容器初始化期间解决它。使用`@Configuration`类时，Java 编译器对配置模型施加约束，因为对其他 bean 的引用必须是有效的 Java 语法。

幸运的是，解决这个问题很简单。正如[我们已经讨论过](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java-dependencies)的，一个`@Bean`方法可以有任意数量的参数来描述 bean 的依赖关系。考虑以下具有多个`@Configuration` 类的更真实的场景，每个类都依赖于其他类中声明的 bean：

```java
@Configuration
public class ServiceConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }
}

@Configuration
public class RepositoryConfig {

    @Bean
    public AccountRepository accountRepository(DataSource dataSource) {
        return new JdbcAccountRepository(dataSource);
    }
}

@Configuration
@Import({ServiceConfig.class, RepositoryConfig.class})
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return new DataSource
    }
}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    // everything wires up across configuration classes...
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```

还有另一种方法可以达到相同的结果。请记住，`@Configuration`类最终只是容器中的另一个 bean：这意味着它们可以像任何其他 bean 一样利用`@Autowired`和`@Value`注入其他特性。

确保您以这种方式注入的依赖项只是最简单的类型。`@Configuration` 类在上下文初始化期间很早就被处理，并且强制以这种方式注入依赖项可能会导致意外的早期初始化。尽可能使用基于参数的注入，如前面的示例所示。

此外，请特别注意`BeanPostProcessor`和的`BeanFactoryPostProcessor`定义`@Bean`。这些通常应该被声明为`static @Bean`方法，而不是触发它们包含的配置类的实例化。否则，`@Autowired`和`@Value`可能无法在配置类本身上工作，因为可以将其创建为早于 [`AutowiredAnnotationBeanPostProcessor`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/beans/factory/annotation/AutowiredAnnotationBeanPostProcessor.html).

以下示例显示了如何将一个 bean 自动装配到另一个 bean：

```java
@Configuration
public class ServiceConfig {

    @Autowired
    private AccountRepository accountRepository;

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl(accountRepository);
    }
}

@Configuration
public class RepositoryConfig {

    private final DataSource dataSource;

    public RepositoryConfig(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(dataSource);
    }
}

@Configuration
@Import({ServiceConfig.class, RepositoryConfig.class})
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return new DataSource
    }
}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    // everything wires up across configuration classes...
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```

仅从 Spring Framework 4.3 开始支持`@Configuration`类中的 构造函数注入。另请注意，如果目标 bean 仅定义一个构造函数 ，则无需指定`@Autowired`。

完全合格的导入Bean，便于导航

在前面的场景中，使用`@Autowired`运行良好并提供了所需的模块化，但确定自动装配 bean 定义的确切声明位置仍然有些模棱两可。例如，作为开发人员，查找`ServiceConfig`，您如何知道`@Autowired AccountRepository`bean 的确切声明位置？它在代码中并不明确，这可能还好。请记住， [Eclipse 的 Spring Tools](https://spring.io/tools)提供的工具可以渲染显示所有连接方式的图形，这可能就是您所需要的。此外，您的 Java IDE 可以轻松找到`AccountRepository`类型的所有声明和使用，并快速向您显示返回该类型的`@Bean`方法的位置。

如果这种歧义是不可接受的，并且您希望在 IDE 中从一个`@Configuration`类直接导航到另一个类，请考虑自动装配配置类本身。以下示例显示了如何执行此操作：

```java
@Configuration
public class ServiceConfig {

    @Autowired
    private RepositoryConfig repositoryConfig;

    @Bean
    public TransferService transferService() {
        // navigate 'through' the config class to the @Bean method!
        return new TransferServiceImpl(repositoryConfig.accountRepository());
    }
}
```

在上述情况下，where `AccountRepository`is defined 是完全明确的。但是，`ServiceConfig`现在与`RepositoryConfig`. 这就是权衡。通过使用基于接口或基于抽象类的类可以在一定程度上缓解这种紧密耦合`@Configuration`。考虑以下示例：

```java
@Configuration
public class ServiceConfig {

    @Autowired
    private RepositoryConfig repositoryConfig;

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl(repositoryConfig.accountRepository());
    }
}

@Configuration
public interface RepositoryConfig {

    @Bean
    AccountRepository accountRepository();
}

@Configuration
public class DefaultRepositoryConfig implements RepositoryConfig {

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(...);
    }
}

@Configuration
@Import({ServiceConfig.class, DefaultRepositoryConfig.class})  // import the concrete config!
public class SystemTestConfig {

    @Bean
    public DataSource dataSource() {
        // return DataSource
    }

}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```

相对于具体的`DefaultRepositoryConfig`， `ServiceConfig` 是松散耦合的 ，并且内置的 IDE 工具仍然有用：您可以轻松获得`RepositoryConfig`实现的类型层次结构。通过这种方式，导航`@Configuration`类及其依赖项与导航基于接口的代码的通常过程没有什么不同。

如果您想影响某些 bean 的启动创建顺序，请考虑将其中一些声明为`@Lazy`（用于在首次访问时创建而不是在启动时创建）或`@DependsOn`某些其他 bean（确保在当前 bean 之前创建特定的其他 bean，超出后者的直接依赖意味着什么）。

**有条件地包含`@Configuration`类或`@Bean`方法**

基于某些任意系统状态，有条件地启用或禁用完整的`@Configuration`类甚至单个`@Bean`方法通常很有用。一个常见的例子是，只有在 Spring 中启用了特定配置文件时才使用`@Profile`注解来激活 bean `Environment`（有关详细信息，请参阅[Bean 定义配置文件](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-definition-profiles) ）。

`@Profile`注解实际上是通过使用更灵活的注解来实现的，称为[`@Conditional`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/context/annotation/Conditional.html). `@Conditional`注解指示 在注册`@Bean`之前应参考`org.springframework.context.annotation.Condition`的具体实现。

接口的实现`Condition`提供了一个`matches(…)` 返回`true`或的方法`false`。例如，以下清单显示了 `Condition`用于`@Profile` 的实际实现：

```java
@Override
public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
    // Read the @Profile annotation attributes
    MultiValueMap<String, Object> attrs = metadata.getAllAnnotationAttributes(Profile.class.getName());
    if (attrs != null) {
        for (Object value : attrs.get("value")) {
            if (context.getEnvironment().acceptsProfiles(((String[]) value))) {
                return true;
            }
        }
        return false;
    }
    return true;
}
```

有关更多详细信息，请参阅[`@Conditional`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/context/annotation/Conditional.html) javadoc。

**结合 Java 和 XML 配置**

Spring 的`@Configuration`类支持并非旨在 100% 完全替代 Spring XML。一些工具，例如 Spring XML 命名空间，仍然是配置容器的理想方式。在 XML 方便或必要的情况下，您可以选择：或者以“以 XML 为中心”的方式实例化容器，例如，`ClassPathXmlApplicationContext`，或者通过使用 `AnnotationConfigApplicationContext` 和 `@ImportResource` 注释以“以 Java 为中心”的方式实例化它，以根据需要导入 XML。

**以 XML 为中心的`@Configuration`类的使用**

最好从 XML 引导 Spring 容器并 以特别的方式包含`@Configuration`类。例如，在使用 Spring XML 的大型现有代码库中，更容易根据需要创建`@Configuration`类并从现有 XML 文件中包含它们。在本节的后面部分，我们将介绍在这种“以 XML 为中心”的情况下使用`@Configuration`类的选项。

将类声明`@Configuration`为普通 Spring`<bean/>`元素

请记住，`@Configuration`类最终是容器中的 bean 定义。在本系列示例中，我们创建了一个名为`AppConfig`的`@Configuration`类，并将其`system-test-config.xml`作为`<bean/>`定义包含在其中。因为 `<context:annotation-config/>`是开启的，所以容器会识别 `@Configuration`注解并正确处理`AppConfig`其中`@Bean`声明的方法 。

以下示例显示了 Java 中的一个普通配置类：

```java
@Configuration
public class AppConfig {

    @Autowired
    private DataSource dataSource;

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(dataSource);
    }

    @Bean
    public TransferService transferService() {
        return new TransferService(accountRepository());
    }
}
```

以下示例显示了示例`system-test-config.xml`文件的一部分：

```xml
<beans>
    <!-- enable processing of annotations such as @Autowired and @Configuration -->
    <context:annotation-config/>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>

    <bean class="com.acme.AppConfig"/>

    <bean class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
</beans>
```

以下示例显示了一个可能的`jdbc.properties`文件：

```properties
jdbc.url=jdbc:hsqldb:hsql://localhost/xdb
jdbc.username=sa
jdbc.password=
```

```java
public static void main(String[] args) {
    ApplicationContext ctx = new ClassPathXmlApplicationContext("classpath:/com/acme/system-test-config.xml");
    TransferService transferService = ctx.getBean(TransferService.class);
    // ...
}
```

在`system-test-config.xml`文件中，`AppConfig` `<bean/>`不声明`id` 元素。虽然这样做是可以接受的，但这是不必要的，因为没有其他 bean 曾经引用过它，并且不太可能通过名称从容器中显式获取。类似地，`DataSource`bean 仅按类型自动装配，因此并不严格要求显式 bean `id` 。

使用 `<context:component-scan/>` 拾取`@Configuration`类

因为`@Configuration`是用 `@Component`元注解的，带`@Configuration`注解的类自动成为组件扫描的候选对象。使用与前面示例中描述的相同场景，我们可以重新定义`system-test-config.xml`以利用组件扫描。请注意，在这种情况下，我们不需要显式声明 `<context:annotation-config/>`，因为`<context:component-scan/>`启用了相同的功能。

以下示例显示了修改后的`system-test-config.xml`文件：

```xml
<beans>
    <!-- picks up and registers AppConfig as a bean definition -->
    <context:component-scan base-package="com.acme"/>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>

    <bean class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
</beans>
```

**`@Configuration`以类为中心使用 XML`@ImportResource`**

在`@Configuration`类是配置容器的主要机制的应用程序中，仍然可能至少需要使用一些 XML。在这些场景中，您可以根据`@ImportResource`需要使用和定义尽可能多的 XML。这样做实现了一种“以 Java 为中心”的方法来配置容器并将 XML 保持在最低限度。以下示例（包括配置类、定义 bean 的 XML 文件、属性文件和`main`类）显示了如何使用`@ImportResource`注解来实现“以 Java 为中心”的配置，该配置根据需要使用 XML：

```java
@Configuration
@ImportResource("classpath:/com/acme/properties-config.xml")
public class AppConfig {

    @Value("${jdbc.url}")
    private String url;

    @Value("${jdbc.username}")
    private String username;

    @Value("${jdbc.password}")
    private String password;

    @Bean
    public DataSource dataSource() {
        return new DriverManagerDataSource(url, username, password);
    }
}

properties-config.xml
<beans>
    <context:property-placeholder location="classpath:/com/acme/jdbc.properties"/>
</beans>
 
jdbc.properties
jdbc.url=jdbc:hsqldb:hsql://localhost/xdb
jdbc.username=sa
jdbc.密码=
```

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    TransferService transferService = ctx.getBean(TransferService.class);
    // ...
}
```

#### 1.13. 抽象环境

[`Environment`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/core/env/Environment.html)接口是集成在容器中的抽象，它对应用程序环境的两个关键方面进行建模：配置[文件](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-definition-profiles) 和[属性](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-property-source-abstraction)。

配置文件是一个命名的、逻辑的 bean 定义组，仅当给定的配置文件处于活动状态时才向容器注册。可以将 Bean 分配给配置文件，无论是在 XML 中定义还是使用注解定义。与配置文件相关的`Environment`对象的作用是确定哪些配置文件（如果有）当前处于活动状态，以及哪些配置文件（如果有）默认情况下应该是活动的。

属性在几乎所有应用程序中都发挥着重要作用，并且可能源自多种来源：属性文件、JVM 系统属性、系统环境变量、JNDI、servlet 上下文参数、ad-hoc`Properties`对象、`Map`对象等。与属性相关的`Environment`对象的作用是为用户提供一个方便的服务接口，用于配置属性源并从中解析属性。

**1.13.1. Bean 定义配置文件**

Bean 定义配置文件在核心容器中提供了一种机制，允许在不同环境中注册不同的 bean。“环境”这个词对不同的用户可能意味着不同的东西，这个功能可以帮助许多用例，包括：

* 在开发中处理内存中的数据源，而不是在 QA 或生产中从 JNDI 中查找相同的数据源。
* 仅在将应用程序部署到性能环境时才注册监控基础架构。
* 为客户 A 和客户 B 部署注册定制的 bean 实现。

考虑实际应用程序中需要 `DataSource`. 在测试环境中，配置可能类似于以下内容：

```java
@Bean
public DataSource dataSource() {
    return new EmbeddedDatabaseBuilder()
        .setType(EmbeddedDatabaseType.HSQL)
        .addScript("my-schema.sql")
        .addScript("my-test-data.sql")
        .build();
}
```

现在考虑如何将此应用程序部署到 QA 或生产环境中，假设应用程序的数据源已在生产应用程序服务器的 JNDI 目录中注册。我们的`dataSource`bean 现在看起来像下面的清单：

```java
@Bean(destroyMethod="")
public DataSource dataSource() throws Exception {
    Context ctx = new InitialContext();
    return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
}
```

问题是如何根据当前环境在使用这两种变体之间进行切换。随着时间的推移，Spring 用户设计了许多方法来完成此任务，通常依赖于系统环境变量和包含`<import/>`标签的 XML 语句的组合，`${placeholder}`根据环境变量的值解析为正确的配置文件路径。Bean 定义概要文件是一个核心容器特性，它为这个问题提供了解决方案。

如果我们概括前面环境特定 bean 定义示例中所示的用例，我们最终需要在某些上下文中注册某些 bean 定义，但在其他上下文中不需要。您可以说您想在情况 A 中注册特定的 bean 定义配置文件，在情况 B 中注册不同的配置文件。我们首先更新配置以反映这种需求。

**使用`@Profile`**

当一个或多个指定的配置文件处于活动状态时，[`@Profile`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/context/annotation/Profile.html) 注解可让您指示组件有资格注册。使用我们前面的示例，我们可以重写`dataSource`配置如下：

```java
@Configuration
@Profile("development")
public class StandaloneDataConfig {

    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .addScript("classpath:com/bank/config/sql/test-data.sql")
            .build();
    }
}
```

```java
@Configuration
@Profile("production")
public class JndiDataConfig {

    @Bean(destroyMethod="")
    public DataSource dataSource() throws Exception {
        Context ctx = new InitialContext();
        return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
    }
}
```

如前所述，对于`@Bean`方法，您通常选择使用程序化 JNDI 查找，通过使用 Spring 的`JndiTemplate`/`JndiLocatorDelegate`助手或`InitialContext`前面显示的直接 JNDI 用法，而不是`JndiObjectFactoryBean` 变体，这将迫使您将返回类型声明为`FactoryBean`类型。

配置文件字符串可能包含一个简单的配置文件名称（例如，`production`）或配置文件表达式。配置文件表达式允许表达更复杂的配置文件逻辑（例如，`production & us-east`）。配置文件表达式中支持以下运算符：

* `!`：配置文件的逻辑“非”
* `&`：配置文件的逻辑“与”
* `|`：配置文件的逻辑“或”

不能在不使用括号的情况下混合使用`&`和`|`运算符。例如， `production & us-east | eu-central`不是一个有效的表达式。它必须表示为 `production & (us-east | eu-central)`。

您可以将`@Profile`其用作[元注解](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-meta-annotations)以创建自定义组合注解。以下示例定义了一个自定义 `@Production`注解，您可以将其用作 `@Profile("production")`的替代品 ：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Profile("production")
public @interface Production {
}
```

如果一个`@Configuration`类用`@Profile` 标记，则与该类关联的所有`@Bean`方法和 `@Import`注解都将被绕过，除非一个或多个指定的配置文件处于活动状态。如果一个`@Component`或`@Configuration`类标有`@Profile({"p1", "p2"})`，则除非已激活配置文件“p1”或“p2”，否则不会注册或处理该类。如果给定配置文件以 NOT 运算符 (`!` ) 为前缀，则仅当配置文件不活动时才注册带注解的元素。例如，`@Profile({"p1", "!p2"})`如果配置文件“p1”处于活动状态或配置文件“p2”未处于活动状态，则会发生注册。

`@Profile`也可以在方法级别声明为仅包含配置类的一个特定 bean（例如，对于特定 bean 的替代变体），如以下示例所示：

```java
@Configuration
public class AppConfig {

    @Bean("dataSource")
    @Profile("development") 
    public DataSource standaloneDataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .addScript("classpath:com/bank/config/sql/test-data.sql")
            .build();
    }

    @Bean("dataSource")
    @Profile("production") 
    public DataSource jndiDataSource() throws Exception {
        Context ctx = new InitialContext();
        return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
    }
}
```

`@Bean`方法使用`@Profile`，可能会应用一种特殊情况：在相同 Java 方法名称的重载`@Bean`方法的情况下（类似于构造函数重载），需要在所有重载方法上一致地声明一个`@Profile`条件。如果条件不一致，则仅重载方法中第一个声明的条件。因此，`@Profile`不能用于选择具有特定参数签名的重载方法而不选择另一个。同一 bean 的所有工厂方法之间的解析在创建时遵循 Spring 的构造函数解析算法。如果要定义具有不同配置文件条件的替代 bean，请使用不同的 Java 方法名称，这些`@Bean`方法名称通过使用name 属性指向相同的 bean 名称，如前面的示例所示。如果参数签名都相同（例如，所有变体都有无参数工厂方法），这是首先在有效 Java 类中表示这种安排的唯一方法（因为只能有一个特定名称和参数签名的方法）。

**XML Bean 定义配置文件**

XML 对应物是`<beans>`元素的`profile`属性。我们前面的示例配置可以重写为两个 XML 文件，如下所示：

```xml
<beans profile="development"
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jdbc="http://www.springframework.org/schema/jdbc"
    xsi:schemaLocation="...">

    <jdbc:embedded-database id="dataSource">
        <jdbc:script location="classpath:com/bank/config/sql/schema.sql"/>
        <jdbc:script location="classpath:com/bank/config/sql/test-data.sql"/>
    </jdbc:embedded-database>
</beans>
<beans profile="production"
    xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jee="http://www.springframework.org/schema/jee"
    xsi:schemaLocation="...">

    <jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/datasource"/>
</beans>
```

也可以避免在同一文件中拆分和嵌套`<beans/>`元素，如以下示例所示：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:jdbc="http://www.springframework.org/schema/jdbc"
    xmlns:jee="http://www.springframework.org/schema/jee"
    xsi:schemaLocation="...">

    <!-- other bean definitions -->

    <beans profile="development">
        <jdbc:embedded-database id="dataSource">
            <jdbc:script location="classpath:com/bank/config/sql/schema.sql"/>
            <jdbc:script location="classpath:com/bank/config/sql/test-data.sql"/>
        </jdbc:embedded-database>
    </beans>

    <beans profile="production">
        <jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/datasource"/>
    </beans>
</beans>
```

XML 对应项不支持前面描述的配置文件表达式。但是，可以使用`!`运算符来否定配置文件。也可以通过嵌套配置文件来应用逻辑“和”，如以下示例所示：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"    xmlns:jdbc="http://www.springframework.org/schema/jdbc"    xmlns:jee="http://www.springframework.org/schema/jee"    xsi:schemaLocation="...">     <!-- other bean definitions -->
	<beans profile="production"> 
		<beans profile="us-east"> 
			<jee:jndi-lookup id="dataSource" jndi-name="java:comp/env/jdbc/datasource"/>         </beans>  
	</beans> 
</beans>
```

在前面的示例中，如果`production`和 `us-east` 配置文件都处于活动状态，则公开 `dataSource`bean 。

**激活配置文件**

现在我们已经更新了配置，我们仍然需要指示 Spring 哪个配置文件处于活动状态。如果我们现在启动示例应用程序，我们会看到抛出`NoSuchBeanDefinitionException`异常，因为容器找不到名为`dataSource` 的 Spring bean 。

激活配置文件可以通过多种方式完成，但最直接的方法是以编程方式针对通过 `ApplicationContext` 提供的`Environment` API 来执行此操作。. 以下示例显示了如何执行此操作：

```java
AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
ctx.getEnvironment().setActiveProfiles("development");
ctx.register(SomeConfig.class, StandaloneDataConfig.class, JndiDataConfig.class);
ctx.refresh();
```

此外，您还可以通过 `spring.profiles.active`属性以声明方式激活配置文件，可以通过系统环境变量、JVM 系统属性、`web.xml` servlet 上下文参数来指定，甚至可以作为 JNDI 中的条目（参见[`PropertySource`Abstraction](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-property-source-abstraction)）。在集成测试中，可以使用`spring-test` 模块中的`@ActiveProfiles`注解来声明活动配置文件（请参阅环境配置文件的[上下文配置](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testcontext-ctx-management-env-profiles)）。

请注意，配置文件不是“非此即彼”的命题。您可以一次激活多个配置文件。以编程方式，您可以为 `setActiveProfiles()`接受`String…`可变参数的方法提供多个配置文件名称。以下示例激活多个配置文件：

```java
ctx.getEnvironment().setActiveProfiles("profile1", "profile2");
```

以声明方式，`spring.profiles.active`可以接受以逗号分隔的配置文件名称列表，如以下示例所示：

```
    -Dspring.profiles.active="profile1,profile2"
```

**默认配置文件**

默认配置文件表示默认启用的配置文件。考虑以下示例：

```java
@Configuration
@Profile("default")
public class DefaultDataConfig {

    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .build();
    }
}
```

如果没有激活的配置文件，`dataSource`则创建。您可以将此视为一种为一个或多个 bean 提供默认定义的方法。如果启用了任何配置文件，则默认配置文件不适用。

您可以使用`Environment` 的`setDefaultProfiles()`方法或以声明方式使用`spring.profiles.default`属性来更改默认配置文件的名称。

**1.13.2.`PropertySource`抽象**

Spring 的`Environment`抽象提供了对属性源的可配置层次结构的搜索操作。考虑以下清单：

```java
ApplicationContext ctx = new GenericApplicationContext();
Environment env = ctx.getEnvironment();
boolean containsMyProperty = env.containsProperty("my-property");
System.out.println("Does my environment contain the 'my-property' property? " + containsMyProperty);
```

在前面的`my-property`代码片段中，我们看到了一种询问 Spring 是否为当前环境定义属性的高级方法。为了回答这个问题，`Environment`对象对一组对象执行搜索[`PropertySource`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/core/env/PropertySource.html) 。`PropertySource`是对任何键值对源的简单抽象，Spring[`StandardEnvironment`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/core/env/StandardEnvironment.html) 配置了两个 PropertySource 对象——一个代表 JVM 系统属性集（`System.getProperties()`），一个代表系统环境变量集（`System.getenv()`）。

这些默认属性源用于`StandardEnvironment`, 用于独立应用程序。[`StandardServletEnvironment`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/web/context/support/StandardServletEnvironment.html) 填充了其他默认属性源，包括 servlet 配置和 servlet 上下文参数。它可以选择启用[`JndiPropertySource`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/jndi/JndiPropertySource.html). 有关详细信息，请参阅 javadoc。

具体来说，当您使用`my-property` 时，如果系统属性或环境变量在运行时存在`my-property`，则调用`StandardEnvironment`的`env.containsProperty("my-property")` 返回 true 。

执行的搜索是分层的。默认情况下，系统属性优先于环境变量。因此，如果在调用`env.getProperty("my-property")` 期间恰好在两个位置都设置了`my-property`属性，则系统属性值“获胜”并返回。请注意，属性值不会合并，而是完全被前面的条目覆盖。对于 common `StandardServletEnvironment`，完整的层次结构如下，最高优先级的条目位于顶部：

1. ServletConfig 参数（如果适用——例如，在`DispatcherServlet`上下文的情况下）
2. ServletContext 参数（web.xml 上下文参数条目）
3. JNDI 环境变量（`java:comp/env/`条目）
4. JVM 系统属性（`-D`命令行参数）
5. JVM系统环境（操作系统环境变量）

最重要的是，整个机制是可配置的。也许您有一个想要集成到此搜索中的自定义属性源。为此，请实现并实例化您自己的`PropertySource`并将其添加到当前`Environment`的`PropertySources`集合中. 以下示例显示了如何执行此操作：

```java
ConfigurableApplicationContext ctx = new GenericApplicationContext();
MutablePropertySources sources = ctx.getEnvironment().getPropertySources();
sources.addFirst(new MyPropertySource());
```

在前面的代码中，`MyPropertySource`已在搜索中以最高优先级添加。如果它包含一个`my-property`属性，则检测并返回该属性，以支持`my-property`任何其他 `PropertySource`中的任何属性。[`MutablePropertySources`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/core/env/MutablePropertySources.html) API 公开了许多允许精确操作属性源集的方法。

**1.13.3. 使用`@PropertySource`**

[`@PropertySource`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/context/annotation/PropertySource.html) 注解提供了一种方便且声明性的机制，用于将 `PropertySource` 添加到Spring 的`Environment`.

给定一个包含键值对的名为`testbean.name=myTestBean`的`app.properties`文件，以下`@Configuration`类使用`@PropertySource`以调用`testBean.getName()`返回`myTestBean`：

```java
@Configuration
@PropertySource("classpath:/com/myco/app.properties")
public class AppConfig {

    @Autowired
    Environment env;

    @Bean
    public TestBean testBean() {
        TestBean testBean = new TestBean();
        testBean.setName(env.getProperty("testbean.name"));
        return testBean;
    }
}
```

资源位置中存在的任何`${…}`占位符都会`@PropertySource`针对已针对环境注册的属性源集进行解析，如以下示例所示：

```java
@Configuration
@PropertySource("classpath:/com/${my.placeholder:default/path}/app.properties")
public class AppConfig {

    @Autowired
    Environment env;

    @Bean
    public TestBean testBean() {
        TestBean testBean = new TestBean();
        testBean.setName(env.getProperty("testbean.name"));
        return testBean;
    }
}
```

假设`my.placeholder`存在于已注册的属性源之一（例如，系统属性或环境变量）中，则占位符被解析为相应的值。如果不是，则将`default/path`其用作默认值。如果未指定默认值且无法解析属性， 则抛出 `IllegalArgumentException`。

根据Java 8 约定，`@PropertySource`注解是可重复的。但是，所有此类`@PropertySource`注解都需要在同一级别声明，或者直接在配置类上声明，或者作为同一自定义注解中的元注解。不建议混合直接注解和元注解，因为直接注解有效地覆盖了元注解。

**1.13.4. 语句中的占位符解析**

从历史上看，元素中占位符的值只能根据 JVM 系统属性或环境变量来解析。这已不再是这种情况。因为抽象`Environment`是在整个容器中集成的，所以很容易通过它来路由占位符的解析。这意味着您可以以任何您喜欢的方式配置解析过程。您可以更改搜索系统属性和环境变量的优先级或完全删除它们。您还可以根据需要将自己的属性源添加到组合中。

具体来说，无论属性在何处定义`customer`，只要它在 `Environment`中可用，以下语句都有效：

```xml
<beans>
    <import resource="com/bank/service/${customer}-config.xml"/>
</beans>
```

#### 1.14. 注册一个`LoadTimeWeaver`

当`LoadTimeWeaver`类加载到 Java 虚拟机 (JVM) 中时，Spring 使用它来动态转换类。

要启用加载时编织，您可以将`@EnableLoadTimeWeaving` 添加到您的 `@Configuration`类之一，如以下示例所示：

```java
@Configuration
@EnableLoadTimeWeaving
public class AppConfig {
}
```

或者，对于 XML 配置，您可以使用以下`context:load-time-weaver`元素：

```xml
<beans>
    <context:load-time-weaver/>
</beans>
```

一旦为`ApplicationContext` 配置 ，`ApplicationContext`中的任何 bean 都 可以实现`LoadTimeWeaverAware`，从而接收对加载时编织器实例的引用。[这在与Spring 的 JPA 支持](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#orm-jpa)结合使用时特别有用， 其中 JPA 类转换可能需要加载时编织。有关更多详细信息，请参阅 [`LocalContainerEntityManagerFactoryBean`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/orm/jpa/LocalContainerEntityManagerFactoryBean.html) javadoc。有关 AspectJ 加载时编织的更多信息，请参阅[Spring Framework 中使用 AspectJ 进行加载时编织](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-aj-ltw)。

#### 1.15.`ApplicationContext`的附加功能

正如在[介绍章节](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans)中所讨论的，该`org.springframework.beans.factory` 包提供了管理和操作 bean 的基本功能，包括以编程方式。该`org.springframework.context`包添加了 [`ApplicationContext`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/context/ApplicationContext.html) 接口，它扩展了`BeanFactory`接口，此外还扩展了其他接口以提供更多面向应用程序框架的样式的附加功能。许多人以完全声明`ApplicationContext`的方式使用它，甚至没有以编程方式创建它，而是依赖于支持类，例如在Java EE Web 应用程序的正常启动过程中`ContextLoader`自动实例化一个 `ApplicationContext`。

`BeanFactory`为了以更加面向框架的风格增强功能，上下文包还提供了以下功能：

* 通过`MessageSource`访问 i18n 风格的消息。
* 通过`ResourceLoader`接口访问资源，例如 URL 和文件。
* `ApplicationListener`事件发布，即通过使用`ApplicationEventPublisher接口发布给实现接口的bean` 。
* 加载多个（分层）上下文，让每个上下文都通过`HierarchicalBeanFactory`接口专注于一个特定的层，例如应用程序的 Web 层 。

**1.15.1. 国际化使用`MessageSource`**

`ApplicationContext`接口扩展了一个名为`MessageSource`的接口，因此提供了国际化（“i18n”）功能。Spring 还提供了 `HierarchicalMessageSource`接口，可以分层解析消息。这些接口共同提供了 Spring 影响消息解析的基础。这些接口上定义的方法包括：

* `String getMessage(String code, Object[] args, String default, Locale loc)`: 用于从`MessageSource`获取消息. 如果未找到指定语言环境的消息，则使用默认消息。使用标准库`MessageFormat`提供的功能，传入的任何参数都将成为替换值。
* `String getMessage(String code, Object[] args, Locale loc)`：与上一种方法基本相同，但有一个区别：不能指定默认消息。如果找不到消息，`NoSuchMessageException`则抛出 a。
* `String getMessage(MessageSourceResolvable resolvable, Locale locale)`: 上述方法中使用的所有属性也都包装在一个名为 的类 `MessageSourceResolvable`中，您可以在此方法中使用该类。

加载`ApplicationContext`时，它会自动搜索上下文中定义的`MessageSource` bean。bean 必须具有`messageSource`名称。如果找到这样的 bean，则对前面方法的所有调用都委托给消息源。如果未找到消息源，则`ApplicationContext`尝试查找包含同名 bean 的父级。如果是这样，它将使用该 bean 作为`MessageSource`. 如果 `ApplicationContext`找不到任何消息源，则实例化一个空 `DelegatingMessageSource`，以便能够接受对上述方法的调用。

Spring 提供了三个`MessageSource`实现`ResourceBundleMessageSource`，`ReloadableResourceBundleMessageSource` 和`StaticMessageSource`。所有这些`HierarchicalMessageSource`都是为了进行嵌套消息传递而实现的。`StaticMessageSource`很少使用，但`ResourceBundleMessageSource`提供了将消息添加到源的编程方式。以下示例显示：

```xml
<beans>
    <bean id="messageSource"
            class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basenames">
            <list>
                <value>format</value>
                <value>exceptions</value>
                <value>windows</value>
            </list>
        </property>
    </bean>
</beans>
```

该示例假定您有三个名为 `format`，`exceptions`和`windows` 的资源包在您的类路径中定义。任何解析消息的请求都以通过`ResourceBundle`对象解析消息的 JDK 标准方式处理。出于示例的目的，假设上述两个资源包文件的内容如下：

```
# in format.properties
message=Alligators rock!
# in exceptions.properties
argument.required=The {0} argument is required.
```

下一个示例显示了一个运行该`MessageSource`功能的程序。请记住，所有`ApplicationContext`实现也是`MessageSource` 实现，因此可以转换为`MessageSource`接口。

```java
public static void main(String[] args) {
    MessageSource resources = new ClassPathXmlApplicationContext("beans.xml");
    String message = resources.getMessage("message", null, "Default", Locale.ENGLISH);
    System.out.println(message);
}
```

上述程序的结果输出如下：

```
Alligators rock!
```

总而言之，`MessageSource`是在一个名为 `beans.xml`的文件中定义的，该文件位于类路径的根目录中。`messageSource` bean 定义通过其属性引用了许多资源包。`basenames`列表中传递给`basenames`属性的三个文件作为文件存在于类路径的根目录中，分别称为`format.properties`、`exceptions.properties`和 `windows.properties`。

下一个示例显示传递给消息查找的参数。这些参数被转换为`String`对象并插入到查找消息中的占位符中。

```xml
<beans>

    <!-- this MessageSource is being used in a web application -->
    <bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basename" value="exceptions"/>
    </bean>

    <!-- lets inject the above MessageSource into this POJO -->
    <bean id="example" class="com.something.Example">
        <property name="messages" ref="messageSource"/>
    </bean>

</beans>
```

```java
public class Example {

    private MessageSource messages;

    public void setMessages(MessageSource messages) {
        this.messages = messages;
    }

    public void execute() {
        String message = this.messages.getMessage("argument.required",
            new Object [] {"userDao"}, "Required", Locale.ENGLISH);
        System.out.println(message);
    }
}
```

调用该`execute()`方法的结果输出如下：

```
The userDao argument is required.
```

关于国际化（“i18n”），Spring 的各种`MessageSource` 实现遵循与标准 JDK 相同的语言环境解析和回退规则 `ResourceBundle`。简而言之，继续前面定义的`messageSource`示例，如果您想根据英国 ( `en-GB`) 语言环境解析消息，您将分别创建名为`format_en_GB.properties`、`exceptions_en_GB.properties`和 `windows_en_GB.properties`的文件。

通常，区域设置解析由应用程序的周围环境管理。在以下示例中，手动指定解析（英国）消息的语言环境：

```
# in exceptions_en_GB.properties
argument.required=Ebagum lad, the ''{0}'' argument is required, I say, required.
```

```java
public static void main(final String[] args) {
    MessageSource resources = new ClassPathXmlApplicationContext("beans.xml");
    String message = resources.getMessage("argument.required",
        new Object [] {"userDao"}, "Required", Locale.UK);
    System.out.println(message);
}
```

上述程序运行的结果如下：

```
Ebagum lad, the 'userDao' argument is required, I say, required.
```

您还可以使用该`MessageSourceAware`接口来获取对 已定义的任何`MessageSource`内容的引用。在创建和配置 bean 时，在实现 MessageSourceAware 接口的 ApplicationContext 中定义的任何 bean 都会被注入应用程序上下文的 MessageSource。

因为 Spring`MessageSource`是基于 Java 的`ResourceBundle`，所以它不会合并具有相同基本名称的包，而只会使用找到的第一个包。具有相同基本名称的后续消息包将被忽略。

作为 `ResourceBundleMessageSource`的替代方案，Spring 提供了一个 `ReloadableResourceBundleMessageSource`类。此变体支持相同的捆绑文件格式，但比基于标准 JDK 的 `ResourceBundleMessageSource`实现更灵活。特别是，它允许从任何 Spring 资源位置（不仅从类路径）读取文件，并支持捆绑属性文件的热重载（同时在它们之间有效地缓存它们）。有关详细信息，请参阅[`ReloadableResourceBundleMessageSource`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/context/support/ReloadableResourceBundleMessageSource.html) javadoc。

**1.15.2. 标准和自定义事件**

中的事件处理`ApplicationContext`是通过`ApplicationEvent` 类和`ApplicationListener`接口提供的。如果将实现 `ApplicationListener`接口的 bean 部署到上下文中，则每次 `ApplicationEvent`发布到 `ApplicationContext`时，都会通知该 bean。本质上，这是标准的观察者设计模式。

从 Spring 4.2 开始，事件基础结构得到了显着改进，并提供了[基于注解的模型](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#context-functionality-events-annotation)以及发布任意事件的能力（即，不一定从 扩展的对象`ApplicationEvent`）。当这样的对象发布时，我们会为您将其包装在一个事件中。

下表描述了 Spring 提供的标准事件：

| 事件                           | 解释                                                                                                                                                                                                                                                                                                       |
| ---------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ContextRefreshedEvent`      | 在初始化或刷新时发布`ApplicationContext`（例如，通过使用接口`refresh()`上的方法`ConfigurableApplicationContext`）。这里，“初始化”意味着所有 bean 都已加载，后处理器 bean 被检测并激活，单例被预实例化，并且`ApplicationContext`对象已准备好使用。只要上下文没有关闭，就可以多次触发刷新，前提是所选择的`ApplicationContext`实际支持这种“热”刷新。例如，`XmlWebApplicationContext`支持热刷新，但 `GenericApplicationContext`不支持。 |
| `ContextStartedEvent`        | 使用接口上的方法 `ApplicationContext`启动时发布。在这里，“已启动”意味着所有 bean 都接收到一个明确的启动信号。通常，此信号用于在显式停止后重新启动 bean，但它也可用于启动尚未配置为自动启动的组件（例如，尚未在初始化时启动的组件）。`start()``ConfigurableApplicationContext``Lifecycle`                                                                                                                  |
| `ContextStoppedEvent`        | 使用接口上的方法 `ApplicationContext`停止时发布。在这里，“停止”意味着所有 的 bean 都会收到一个明确的停止信号。可以通过 调用重新启动已停止的上下文。`stop()``ConfigurableApplicationContext``Lifecycle``start()`                                                                                                                                                    |
| `ContextClosedEvent`         | 在`ApplicationContext`使用接口`close()`上的方法`ConfigurableApplicationContext`或通过 JVM 关闭挂钩关闭时发布。在这里，“关闭”意味着所有的单例 bean 都将被销毁。一旦上下文关闭，它就到了生命的尽头，无法刷新或重新启动。                                                                                                                                                         |
| `RequestHandledEvent`        | 一个特定于 Web 的事件，告诉所有 bean 一个 HTTP 请求已得到服务。此事件在请求完成后发布。此事件仅适用于使用 Spring 的 Web 应用程序`DispatcherServlet`。                                                                                                                                                                                                      |
| `ServletRequestHandledEvent` | 它的子类`RequestHandledEvent`添加了 Servlet 特定的上下文信息。                                                                                                                                                                                                                                                           |

您还可以创建和发布自己的自定义事件。以下示例显示了一个扩展 Spring`ApplicationEvent`基类的简单类：

```java
public class BlockedListEvent extends ApplicationEvent {

    private final String address;
    private final String content;

    public BlockedListEvent(Object source, String address, String content) {
        super(source);
        this.address = address;
        this.content = content;
    }

    // accessor and other methods...
}
```

要发布自定义`ApplicationEvent`，请调用 `ApplicationEventPublisher`的`publishEvent()`. 通常，这是通过创建一个实现 `ApplicationEventPublisherAware`并将其注册为 Spring bean 的类来完成的。下面的例子展示了这样一个类：

```java
public class EmailService implements ApplicationEventPublisherAware {

    private List<String> blockedList;
    private ApplicationEventPublisher publisher;

    public void setBlockedList(List<String> blockedList) {
        this.blockedList = blockedList;
    }

    public void setApplicationEventPublisher(ApplicationEventPublisher publisher) {
        this.publisher = publisher;
    }

    public void sendEmail(String address, String content) {
        if (blockedList.contains(address)) {
            publisher.publishEvent(new BlockedListEvent(this, address, content));
            return;
        }
        // send email...
    }
}
```

在配置时，Spring 容器检测到`EmailService`实现 `ApplicationEventPublisherAware`并自动调用 `setApplicationEventPublisher()`. 实际上，传入的参数是Spring容器本身。您正在通过其 `ApplicationEventPublisher`接口与应用程序上下文进行交互。

要接收自定义的`ApplicationEvent`，您可以创建一个实现 `ApplicationListener`并将其注册为 Spring bean 的类。下面的例子展示了这样一个类：

```java
public class BlockedListNotifier implements ApplicationListener<BlockedListEvent> {

    private String notificationAddress;

    public void setNotificationAddress(String notificationAddress) {
        this.notificationAddress = notificationAddress;
    }

    public void onApplicationEvent(BlockedListEvent event) {
        // notify appropriate parties via notificationAddress...
    }
}
```

请注意，`ApplicationListener`通常使用自定义事件的类型进行参数化（在前面的`BlockedListEvent`示例中）。这意味着该 `onApplicationEvent()`方法可以保持类型安全，避免任何向下转换的需要。您可以根据需要注册任意数量的事件侦听器，但请注意，默认情况下，事件侦听器会同步接收事件。这意味着该`publishEvent()`方法会阻塞，直到所有侦听器都完成了对事件的处理。这种同步和单线程方法的一个优点是，当侦听器接收到事件时，如果事务上下文可用，它会在发布者的事务上下文中运行。如果需要另一种事件发布策略，请参阅 javadoc 了解 Spring 的 [`ApplicationEventMulticaster`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/context/event/ApplicationEventMulticaster.html)接口和[`SimpleApplicationEventMulticaster`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/context/event/SimpleApplicationEventMulticaster.html) 配置选项的实现。

以下示例显示了用于注册和配置上述每个类的 bean 定义：

```xml
<bean id="emailService" class="example.EmailService">
    <property name="blockedList">
        <list>
            <value>known.spammer@example.org</value>
            <value>known.hacker@example.org</value>
            <value>john.doe@example.org</value>
        </list>
    </property>
</bean>

<bean id="blockedListNotifier" class="example.BlockedListNotifier">
    <property name="notificationAddress" value="blockedlist@example.org"/>
</bean>
```

总而言之，当调用`emailService` bean 的`sendEmail()`方法时，如果有任何电子邮件消息应该被阻止， 则会发布一个自定义`BlockedListEvent`类型的事件。`blockedListNotifier`bean 注册为 an `ApplicationListener`并接收`BlockedListEvent`，此时它可以通知适当的各方。

Spring 的事件机制是为同一应用程序上下文中的 Spring bean 之间的简单通信而设计的。然而，对于更复杂的企业集成需求，单独维护的 [Spring Integration](https://projects.spring.io/spring-integration/)项目为 构建基于众所周知的 Spring 编程模型的 轻量级、[面向模式、事件驱动的架构提供了完整的支持。](https://www.enterpriseintegrationpatterns.com/)

**基于注解的事件监听器**

`@EventListener`您可以使用注解在托管 bean 的任何方法上注册事件侦听器 。`BlockedListNotifier`可以改写如下：

```java
public class BlockedListNotifier {

    private String notificationAddress;

    public void setNotificationAddress(String notificationAddress) {
        this.notificationAddress = notificationAddress;
    }

    @EventListener
    public void processBlockedListEvent(BlockedListEvent event) {
        // notify appropriate parties via notificationAddress...
    }
}
```

方法签名再次声明了它所侦听的事件类型，但是这一次使用了一个灵活的名称并且没有实现特定的侦听器接口。只要实际事件类型在其实现层次结构中解析您的泛型参数，也可以通过泛型来缩小事件类型。

如果您的方法应该监听多个事件，或者如果您想在没有参数的情况下定义它，也可以在注解本身上指定事件类型。以下示例显示了如何执行此操作：

```java
@EventListener({ContextStartedEvent.class, ContextRefreshedEvent.class})
public void handleContextStart() {
    // ...
}
```

还可以通过使用定义[`SpEL`表达式](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#expressions)的注解`condition`属性添加额外的运行时过滤，表达式应该匹配以实际调用特定事件的方法。

以下示例显示了如何重写我们的通知器以仅在事件的属性`content`等于`my-event`时才被调用 ：

```java
@EventListener(condition = "#blEvent.content == 'my-event'")
public void processBlockedListEvent(BlockedListEvent blEvent) {
    // notify appropriate parties via notificationAddress...
}
```

每个`SpEL`表达式都针对专用上下文进行评估。下表列出了对上下文可用的项目，以便您可以将它们用于条件事件处理：

| 名称     | 位置    | 描述                                                                                            | 例子                                                |
| ------ | ----- | --------------------------------------------------------------------------------------------- | ------------------------------------------------- |
| 事件     | 根对象   | 实际的`ApplicationEvent`.                                                                        | `#root.event`或者`event`                            |
| 参数数组   | 根对象   | 用于调用方法的参数（作为对象数组）。                                                                            | `#root.args`或`args`；`args[0]`访问第一个参数等。            |
| _参数名称_ | 评估上下文 | 任何方法参数的名称。如果由于某种原因，名称不可用（例如，因为编译的字节码中没有调试信息），也可以使用代表参数索引的`#a<#arg>`语法`<#arg>`（从 0 开始）使用单独的参数。 | `#blEvent`或`#a0`（您也可以使用`#p0`或`#p<#arg>`参数表示法作为别名） |

请注意，`#root.event`使您可以访问底层事件，即使您的方法签名实际上是指已发布的任意对象。

如果您需要发布一个事件作为处理另一个事件的结果，您可以更改方法签名以返回应该发布的事件，如以下示例所示：

```java
@EventListener
public ListUpdateEvent handleBlockedListEvent(BlockedListEvent event) {
    // notify appropriate parties via notificationAddress and
    // then publish a ListUpdateEvent...
}
```

[异步侦听](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#context-functionality-events-async) 器不支持此功能 。

`handleBlockedListEvent()`方法为它处理的每一个`ListUpdateEvent`发布一个新的`BlockedListEvent`。如果您需要发布多个事件，则可以改为返回一个`Collection`或一组事件。

**异步侦听器**

如果您希望特定侦听器异步处理事件，则可以重用 [常规`@Async`支持](https://docs.spring.io/spring-framework/docs/current/reference/html/integration.html#scheduling-annotation-support-async)。以下示例显示了如何执行此操作：

```java
@EventListener
@Async
public void processBlockedListEvent(BlockedListEvent event) {
    // BlockedListEvent is processed in a separate thread
}
```

使用异步事件时请注意以下限制：

* 如果异步事件侦听器抛出`Exception`，它不会传播给调用者。有关 [`AsyncUncaughtExceptionHandler`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/aop/interceptor/AsyncUncaughtExceptionHandler.html) 更多详细信息，请参阅。
* 异步事件侦听器方法不能通过返回值来发布后续事件。如果您需要发布另一个事件作为处理的结果，请 [`ApplicationEventPublisher`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/context/ApplicationEventPublisher.html) 手动注入一个来发布该事件。

**订购听众**

如果您需要在另一个侦听器之前调用一个侦听器，您可以`@Order` 在方法声明中添加注解，如以下示例所示：

```java
@EventListener
@Order(42)
public void processBlockedListEvent(BlockedListEvent event) {
    // notify appropriate parties via notificationAddress...
}
```

**通用事件**

您还可以使用泛型来进一步定义事件的结构。考虑使用 `EntityCreatedEvent<T>`，`T`是创建的实际实体的类型。例如，您可以创建以下侦听器定义以仅接收`Person`类型的`EntityCreatedEvent`：

```java
@EventListener
public void onPersonCreated(EntityCreatedEvent<Person> event) {
    // ...
}
```

由于类型擦除，这仅在触发的事件解析了事件侦听器过滤的通用参数（即类似的东西 `class PersonCreatedEvent extends EntityCreatedEvent<Person> { … }`）时才有效。

在某些情况下，如果所有事件都遵循相同的结构，这可能会变得非常乏味（就像前面示例中的事件一样）。在这种情况下，您可以实施`ResolvableTypeProvider`以引导框架超出运行时环境提供的范围。以下事件显示了如何执行此操作：

```java
public class EntityCreatedEvent<T> extends ApplicationEvent implements ResolvableTypeProvider {

    public EntityCreatedEvent(T entity) {
        super(entity);
    }

    @Override
    public ResolvableType getResolvableType() {
        return ResolvableType.forClassWithGenerics(getClass(), ResolvableType.forInstance(getSource()));
    }
}
```

这不仅适用于 ApplicationEvent，还适用于作为事件发送的任何任意对象。

**1.15.3. 方便访问底层资源**

为了优化使用和理解应用程序上下文，您应该熟悉 Spring 的`Resource`抽象，如[参考资料](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#resources)中所述。

应用程序上下文是`ResourceLoader`，可用于加载`Resource`对象。`Resource`本质上是 JDK`java.net.URL`类的功能更丰富的版本。事实上，在适当的地方包装一个`Resource`实例的实现。`java.net.URL`A`Resource`可以以透明的方式从几乎任何位置获取低级资源，包括从类路径、文件系统位置、可使用标准 URL 描述的任何位置以及其他一些变体。如果资源位置字符串是没有任何特殊前缀的简单路径，则这些资源的来源是特定的并且适合于实际的应用程序上下文类型。

您可以配置部署到应用程序上下文中的 bean 来实现特殊的回调接口，`ResourceLoaderAware`在初始化时自动回调，应用程序上下文本身作为`ResourceLoader`. 您还可以公开 type 的属性，`Resource`用于访问静态资源。它们像任何其他属性一样被注入其中。您可以将这些`Resource` 属性指定为简单路径，并在部署 bean 时`String`依赖从这些文本字符串到实际对象的自动转换。`Resource`

提供给构造函数的一个或多个位置路径`ApplicationContext`实际上是资源字符串，并且以简单的形式，根据特定的上下文实现进行适当的处理。例如`ClassPathXmlApplicationContext`，将简单的位置路径视为类路径位置。您还可以使用带有特殊前缀的位置路径（资源字符串）来强制从类路径或 URL 加载定义，而不管实际的上下文类型如何。

**1.15.4. 应用程序启动跟踪**

`ApplicationContext`管理 Spring 应用程序的生命周期并围绕组件提供丰富的编程模型。因此，复杂的应用程序可以具有同样复杂的组件图和启动阶段。

使用特定指标跟踪应用程序启动步骤可以帮助了解在启动阶段花费的时间，但它也可以用作更好地了解整个上下文生命周期的一种方式。

（`AbstractApplicationContext`及其子类）使用 `ApplicationStartup`进行检测 ，它收集`StartupStep`有关各种启动阶段的数据：

* 应用程序上下文生命周期（基础包扫描、配置类管理）
* bean 生命周期（实例化、智能初始化、后处理）
* 应用事件处理

以下是`AnnotationConfigApplicationContext`仪器仪表的示例：

```java
// create a startup step and start recording
StartupStep scanPackages = this.getApplicationStartup().start("spring.context.base-packages.scan");
// add tagging information to the current step
scanPackages.tag("packages", () -> Arrays.toString(basePackages));
// perform the actual phase we're instrumenting
this.scanner.scan(basePackages);
// end the current step
scanPackages.end();
```

应用程序上下文已经配备了多个步骤。记录后，可以使用特定工具收集、显示和分析这些启动步骤。有关现有启动步骤的完整列表，您可以查看 [专用的附录部分](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#application-startup-steps)。

默认`ApplicationStartup`实现是无操作变体，以最小化开销。这意味着默认情况下在应用程序启动期间不会收集任何指标。Spring Framework 附带了一个使用 Java Flight Recorder 跟踪启动步骤的实现： `FlightRecorderApplicationStartup`. 要使用此变体，您必须在创建它后立即配置它的实例`ApplicationContext`。

如果开发人员提供自己的 `AbstractApplicationContext` 子类，或者希望收集更精确的数据，那么他们还可以使用 `ApplicationStartup` 基础设施

`ApplicationStartup`仅在应用程序启动期间和核心容器中使用；这绝不是 Java 分析器或[Micrometer](https://micrometer.io/)等指标库的替代品。

要开始收集自定义 `StartupStep`，组件可以 直接从应用程序上下文中获取`ApplicationStartup`实例，使它们的组件实现`ApplicationStartupAware`，或者在任何注入点请求`ApplicationStartup`类型。

开发人员在创建自定义启动步骤时 不应使用`"spring.*"`命名空间。这个命名空间是为内部 Spring 使用而保留的，并且可能会发生变化。

**1.15.5. 方便的 Web 应用程序 ApplicationContext 实例化**

您可以使用例如`ApplicationContext`以声明方式创建实例 `ContextLoader`。当然，您也可以使用其中一种`ApplicationContext`实现以编程方式创建`ApplicationContext`实例。

您可以使用`ContextLoaderListener`注册一个`ApplicationContext`，如以下示例所示：

```xml
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/daoContext.xml /WEB-INF/applicationContext.xml</param-value>
</context-param>

<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

侦听器检查`contextConfigLocation`参数。如果该参数不存在，则侦听器`/WEB-INF/applicationContext.xml`用作默认值。当参数确实存在时，侦听`String`器使用预定义的分隔符（逗号、分号和空格）分隔 ，并将这些值用作搜索应用程序上下文的位置。也支持 Ant 样式的路径模式。示例是`/WEB-INF/*Context.xml`（对于名称以 `Context.xml`结尾 且驻留在`WEB-INF`目录中的所有文件）和`/WEB-INF/**/*Context.xml` （对于`WEB-INF` 的任何子目录中的所有此类文件）。

**1.15.6. 将 Spring 部署`ApplicationContext`为 Java EE RAR 文件**

可以将 Spring 部署`ApplicationContext`为 RAR 文件，将上下文及其所有必需的 bean 类和库 JAR 封装在 Java EE RAR 部署单元中。`ApplicationContext`这相当于引导一个能够访问 Java EE 服务器设施的独立设备（仅托管在 Java EE 环境中）。RAR 部署是部署无头 WAR 文件的一种更自然的替代方案——实际上，一个没有任何 HTTP 入口点的 WAR 文件，仅用于 `ApplicationContext`在 Java EE 环境中引导 Spring。

RAR 部署非常适合不需要 HTTP 入口点而是仅包含消息端点和计划作业的应用程序上下文。这种上下文中的 Bean 可以使用应用程序服务器资源，例如 JTA 事务管理器和 JNDI 绑定的 JDBC `DataSource`实例和 JMS`ConnectionFactory`实例，还可以向平台的 JMX 服务器注册——所有这些都通过 Spring 的标准事务管理和 JNDI 和 JMX 支持工具。应用程序组件还可以`WorkManager`通过 Spring 的`TaskExecutor`抽象与应用程序服务器的 JCA 交互。

[`SpringContextResourceAdapter`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/jca/context/SpringContextResourceAdapter.html) 有关RAR 部署中涉及的配置详细信息，请参阅该类的 javadoc 。

对于将 Spring ApplicationContext 简单部署为 Java EE RAR 文件：

1. 将所有应用程序类打包成一个 RAR 文件（这是一个具有不同文件扩展名的标准 JAR 文件）。
2. 将所有必需的库 JAR 添加到 RAR 存档的根目录中。
3. 添加 `META-INF/ra.xml`部署描述符（如[javadoc 中`SpringContextResourceAdapter`](https://docs.spring.io/spring-framework/docs/5.3.22/javadoc-api/org/springframework/jca/context/SpringContextResourceAdapter.html)所示）和相应的 Spring XML bean 定义文件（通常 `META-INF/applicationContext.xml`）。
4. 将生成的 RAR 文件拖放到应用程序服务器的部署目录中。

这种 RAR 部署单元通常是独立的。它们不会将组件暴露给外部世界，甚至不会暴露给同一应用程序的其他模块。与基于 RAR 的交互`ApplicationContext`通常通过它与其他模块共享的 JMS 目标发生。例如，基于 RAR 的程序`ApplicationContext`还可以安排一些作业或对文件系统中的新文件（或类似文件）做出反应。如果它需要允许来自外部的同步访问，它可以（例如）导出 RMI 端点，这些端点可以被同一台机器上的其他应用程序模块使用。

#### 1.16. `BeanFactory`API

API 为 Spring的`BeanFactory`IoC 功能提供了底层基础。它的具体契约多用于与 Spring 的其他部分和相关的第三方框架的集成，它的`DefaultListableBeanFactory`实现是上层`GenericApplicationContext`容器内的关键委托。

和`BeanFactory`相关的接口（例如`BeanFactoryAware`、`InitializingBean`、 `DisposableBean`）是其他框架组件的重要集成点。通过不需要任何注解甚至反射，它们允许容器与其组件之间非常有效的交互。应用程序级别的 bean 可以使用相同的回调接口，但通常更喜欢声明性依赖注入，或者通过注解或通过编程配置。

请注意，核心`BeanFactory`API 级别及其`DefaultListableBeanFactory` 实现不会对要使用的配置格式或任何组件注解做出假设。所有这些风格都通过扩展（例如`XmlBeanDefinitionReader`和`AutowiredAnnotationBeanPostProcessor`）出现，并将共享`BeanDefinition`对象作为核心元数据表示进行操作。这就是使 Spring 的容器如此灵活和可扩展的本质。

**1.16.1.`BeanFactory`还是`ApplicationContext`？**

`BeanFactory`本节解释了容器级别和 容器级别之间的差异`ApplicationContext`以及对引导的影响。

`ApplicationContext`除非您有充分的理由不这样做，否则 您应该使用 an`GenericApplicationContext`及其子类`AnnotationConfigApplicationContext` 作为自定义引导的常见实现。这些是 Spring 核心容器的主要入口点，用于所有常见目的：加载配置文件、触发类路径扫描、以编程方式注册 bean 定义和带注解的类，以及（从 5.0 开始）注册功能 bean 定义。

因为 an`ApplicationContext`包含 a 的所有功能`BeanFactory`，所以通常建议在 plain 上使用`BeanFactory`，除了需要完全控制 bean 处理的场景。在一个`ApplicationContext`（例如 `GenericApplicationContext`实现）中，按照约定（即按 bean 名称或按 bean 类型——特别是后处理器）检测几种 bean，而 plain`DefaultListableBeanFactory`对任何特殊 bean 是不可知的。

对于许多扩展容器特性，例如注解处理和 AOP 代理，[`BeanPostProcessor`扩展点](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-extension-bpp)是必不可少的。如果您仅使用普通`DefaultListableBeanFactory`的，则默认情况下不会检测和激活此类后处理器。这种情况可能会令人困惑，因为您的 bean 配置实际上没有任何问题。相反，在这种情况下，需要通过额外的设置来完全引导容器。

下表列出了`BeanFactory`和 `ApplicationContext`接口和实现提供的功能。

| 特征                             | `BeanFactory` | `ApplicationContext` |
| ------------------------------ | ------------- | -------------------- |
| Bean实例化/织入                     | 是的            | 是的                   |
| 集成的生命周期管理                      | 不             | 是的                   |
| 自动`BeanPostProcessor`注册        | 不             | 是的                   |
| 自动`BeanFactoryPostProcessor`注册 | 不             | 是的                   |
| 方便`MessageSource`的访问（国际化）      | 不             | 是的                   |
| 内置`ApplicationEvent`发布机制       | 不             | 是的                   |

要使用`DefaultListableBeanFactory` 显式注册 bean 后处理器，您需要以编程方式调用`addBeanPostProcessor`，如以下示例所示：

```java
DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
// populate the factory with bean definitions

// now register any needed BeanPostProcessor instances
factory.addBeanPostProcessor(new AutowiredAnnotationBeanPostProcessor());
factory.addBeanPostProcessor(new MyBeanPostProcessor());

// now start using the factory
```

要将 `BeanFactoryPostProcessor`应用于`DefaultListableBeanFactory`，您需要调用其`postProcessBeanFactory`方法，如以下示例所示：

```java
DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(factory);
reader.loadBeanDefinitions(new FileSystemResource("beans.xml"));

// bring in some property values from a Properties file
PropertySourcesPlaceholderConfigurer cfg = new PropertySourcesPlaceholderConfigurer();
cfg.setLocation(new FileSystemResource("jdbc.properties"));

// now actually do the replacement
cfg.postProcessBeanFactory(factory);
```

在这两种情况下，显式注册步骤都很不方便，这就是为什么在 Spring 支持的应用程序中，各种`ApplicationContext`变体比普通的更受青睐 ，尤其是在典型企业设置中依赖实例来扩展容器功能时`DefaultListableBeanFactory`。`BeanFactoryPostProcessor``BeanPostProcessor`

An`AnnotationConfigApplicationContext`已注册所有常见的注解后处理器，并且可以通过配置注解引入额外的处理器，例如`@EnableTransactionManagement`. 在 Spring 的基于注解的配置模型的抽象级别上，bean 后处理器的概念变成了单纯的内部容器细节。
