# 10.2. 自定义XML Schema

从 2.0 版开始，Spring 提供了一种机制，可以将基于模式的扩展添加到基本的 Spring XML 格式中，用于定义和配置 bean。本节介绍如何编写自己的自定义 XML bean 定义解析器并将此类解析器集成到 Spring IoC 容器中。

为了便于编写使用模式感知 XML 编辑器的配置文件，Spring 的可扩展 XML 配置机制基于 XML Schema。如果您不熟悉标准 Spring 发行版附带的 Spring 当前 XML 配置扩展，您应该首先阅读[XML Schemas](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#xsd-schemas)的上一节。

要创建新的 XML 配置扩展：

1. [创作](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#xsd-custom-schema)一个 XML 模式来描述您的自定义元素。
2. 编写自定义`NamespaceHandler`的实现[代码。](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#xsd-custom-namespacehandler)
3. 编写一个或多个`BeanDefinitionParser`的实现[代码](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#xsd-custom-parser)（这是完成实际工作的地方）。
4. 使用 Spring[注册您的新工件。](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#xsd-custom-registration)

对于一个统一的示例，我们创建一个 XML 扩展（自定义 XML 元素），它允许我们配置 `SimpleDateFormat`类型的对象（来自`java.text`包）。完成后，我们将能够定义`SimpleDateFormat`类型的 bean 定义如下：

```xml
<myns:dateformat id="dateFormat"
    pattern="yyyy-MM-dd HH:mm"
    lenient="true"/>
```

（我们将在本附录后面包含更详细的示例。第一个简单示例的目的是引导您完成制作自定义扩展的基本步骤。）

