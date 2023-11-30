# 3. 验证、数据绑定和类型转换

将验证视为业务逻辑有利有弊，Spring 提供了一种验证（和数据绑定）设计，不排除其中任何一个。具体来说，验证不应该绑定到 Web 层并且应该易于本地化，并且应该可以插入任何可用的验证器。考虑到这些问题，Spring 提供了一个`Validator`在应用程序的每一层都基本可用的契约。

数据绑定对于让用户输入动态绑定到应用程序的域模型（或用于处理用户输入的任何对象）非常有用。 Spring 提供了恰当命名的 `DataBinder` 来完成此任务。 `Validator`和`DataBinder`组成了验证包，主要用于但不限于Web层。

`BeanWrapper`是 Spring 框架中的一个基本概念，在很多地方都有使用。但是，您可能不需要直接使用`BeanWrapper`。但是，因为这是参考文档，所以我们认为可能需要进行一些解释。我们将在本章中解释它，因为如果您要使用`BeanWrapper`，您很可能在尝试将数据绑定到对象时这样做。

Spring`DataBinder`和较低级别`BeanWrapper`都使用`PropertyEditorSupport` 实现来解析和格式化属性值。`PropertyEditor`和 `PropertyEditorSupport`类型是 JavaBeans 规范的一部分，本章也进行了说明。Spring 3 引入了一个提供通用类型转换工具的包`core.convert`，以及一个用于格式化 UI 字段值的更高级别的“格式”包。您可以将这些包用作实现 `PropertyEditorSupport`的更简单的替代方案。本章还将讨论它们。

Spring 通过设置基础设施和 Spring 自己的验证器契约的适配器来支持 Java Bean 验证。应用程序可以全局启用一次 Bean 验证（如 Java Bean 验证中所述），并将其专门用于所有验证需求。在 Web 层中，应用程序可以进一步注册每个 `DataBinder` 的控制器本地 Spring `Validator` 实例，如配置 `DataBinder` 中所述，这对于插入自定义验证逻辑非常有用。

####

####

####

####

####

####

####
