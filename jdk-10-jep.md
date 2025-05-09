# JDK 10 JEP

## JEP 286：局部变量类型推断

### 摘要

将 Java 语言增强，以扩展类型推断到具有初始值的局部变量声明。

### 目标

我们希望通过减少编写 Java 代码的仪式感来改善开发者的体验，同时保持 Java 对静态类型安全的承诺，通过允许开发者省略局部变量类型的显式声明。此功能将允许，例如以下声明：

```
var list = new ArrayList<String>();  // infers ArrayList<String>
var stream = list.stream();          // infers Stream<String>
```

这种处理将仅限于具有初始值的局部变量、增强的 `for`-循环中的索引，以及在传统 `for`-循环中声明的局部变量；它不适用于方法形式参数、构造函数形式参数、方法返回类型、字段、catch 形式参数或任何其他类型的变量声明。

### 成功标准

从定量角度来看，我们希望这项功能能够将真实代码库中相当一部分局部变量声明转换为使用该功能，并推断出合适的类型。

从定性角度来看，我们希望用户能够了解局部变量类型推断的局限性及其动机。（当然，在一般情况下这是不可能实现的；我们不仅无法为所有局部变量推断出合理的类型，而且一些用户认为类型推断是一种心灵感应，而不是一种约束求解算法，在这种情况下，任何解释似乎都不合理。）但我们寻求划定界限，以便能够清楚地说明为什么某个特定的结构超出了界限——并且使得编译器诊断能够有效地将用户代码中的复杂性与其中的任意语言限制联系起来。

### 动机

开发者经常抱怨 Java 中需要编写大量样板代码。局部变量的显式类型声明通常被认为是不必要的，甚至有时会妨碍理解；只要变量命名良好，通常就能清楚地知道发生了什么。

每个变量都需要提供显式类型声明，这无意中鼓励开发者使用过于复杂的表达式；使用更简洁的声明语法，可以减少将复杂的链式或嵌套表达式拆分成简单表达式的阻力。

几乎所有其他流行的静态类型“大括号”语言，无论是在 JVM 上还是在其他平台上，都已经支持某种形式的局部变量类型推断：C++（auto），C#（var），Scala（var/val），Go（使用 `:=` 的声明）。Java 几乎是唯一一个没有采用局部变量类型推断的流行静态类型语言；此时，这个特性应该不再具有争议性。

Java SE 8 中，类型推断的范围得到了显著扩展，包括嵌套和链接泛型方法调用的扩展推断，以及 lambda 形参的推断。这使得构建调用链 API 变得非常容易，并且此类 API（如 Streams）非常受欢迎，表明开发人员已经习惯了中间类型的推断。在一个调用链中：

```
int maxWeight = blocks.stream()
                      .filter(b -> b.getColor() == BLUE)
                      .mapToInt(Block::getWeight)
                      .max();
```

没有人会感到困扰（甚至都不会注意到），中间类型 `Stream<Block>` 和 `IntStream`，以及 lambda 形参 `b` 的类型，都没有在源代码中显式出现。

局部变量类型推断允许在结构不太紧密的 API 中实现类似的效果；许多局部变量的使用本质上就是链式调用，同样能从推断中受益，例如：

```
var path = Paths.get(fileName);
var bytes = Files.readAllBytes(path);
```

### 描述

对于具有初始值的局部变量声明、增强的 `for`-循环索引以及传统 `for` 循环中声明的索引变量，允许使用保留类型名称 `var` 来代替显式类型：

```
var list = new ArrayList<String>(); // infers ArrayList<String>
var stream = list.stream();         // infers Stream<String>
```

标识符 `var` 不是关键字；它是一&#x4E2A;_&#x4FDD;留类型名称_ 。这意味着使用 `var` 作为变量、方法或包名称的代码将不受影响；而使用 `var` 作为类或接口名称的代码将受到影响（但在实践中，这些名称很少见，因为它们违反了通常的命名规范）。

缺少初始值的局部变量声明、声明多个变量、具有额外数组维度括号或引用正在初始化的变量的形式是不允许的。拒绝没有初始值的局部变量可以缩小该功能的作用范围，避免“远程行动”推断错误，并且仅排除了典型程序中的一小部分局部变量。

推断过程基本上就是给变量赋予其初始表达式类型。一些细节：

* 初始化器没有目标类型（因为我们还没有推断它）。需要这种类型的多态表达式，如 lambda 表达式、方法引用和数组初始化器，将触发错误。
* 如果初始化器是 null 类型，会发生错误——就像没有初始化器的变量一样，这个变量可能是打算稍后初始化，而我们不知道需要什么类型。
* 捕获变量及其嵌套捕获变量的类型&#x88AB;_&#x6295;&#x5F71;_&#x5230;不提及捕获变量的超类型。这种映射用它们的上界替换捕获变量，并用有界通配符替换提及捕获变量的类型参数（然后递归）。这保留了捕获变量传统的有限作用域，即仅在单个语句内考虑。
* 除了上述例外之外，非指称类型，包括匿名类类型和交集类型，也可能被推断。编译器和工具需要考虑这种可能性。

#### 适用性和影响

扫描 OpenJDK 代码库中的局部变量声明，我们发现 13% 无法使用 `var` 来编写，因为没有初始化器，初始化器的类型为 null，或者（很少）初始化器需要目标类型。在其余的局部变量声明中：

* 94% 的初始化器与源代码中存在的确切类型相同（63% 的情况为参数化类型）
* 5% 有一个带有更精确可指称类型的初始化器（在参数化类型的29%的案例中）
* 1% 有一个初始化器，其类型提到了捕获变量（在参数化类型的情况下占7%）
* <1% 有一个初始化器，其类型为匿名类或交叉类型（参数化类型的情况相同）

### 备选方案

我们可以继续要求显式声明局部变量类型。

与其支持 `var`，我们也可以将支持范围限制在变量声明中的 diamond 用法；这可以解决 `var` 所涵盖的一部分情况。

上述设计中包含了对作用域、语法和非可指称类型的几个决策；此处记录了那些被考虑过的替代方案。

#### 作用域选择

我们还可以用其他几种方式来限定这个特性的作用域。我们曾考虑将特性限制在有效的最终局部变量（`val`）上。然而，我们最终放弃了这个立场，因为：

* 大多数（在 JDK 和更广泛的语料库中都超过 75%）具有初始值的局部变量本来就已经是有效地不可变的，这意味着这项功能所能提供的任何“推动”远离可变性的效果都会是有限的；
* 通过 lambda/内部类提供的可捕获性已经为有效地最终局部变量提供了显著的推动；
* 在一个包含（比如）7个有效最终局部变量和2个可变变量的代码块中，可变变量的类型会显得非常突兀，从而削弱了该特性的许多好处。

另一方面，我们本可以将此特性扩展到包括局部变量的“空白”最终变量（即，不需要初始化器，而是依赖确定分配分析。）我们选择将限制设置为“仅包含初始化器的变量”，因为它涵盖了相当大一部分候选者，同时保持了特性的简单性，并减少了“远距离作用”错误。

类似地，我们也本可以将所有赋值都考虑在内进行类型推断，而不仅仅是初始化器；虽然这将进一步提高可以利用此特性的局部变量的百分比，但它也会增加“远距离作用”错误的风险。

#### 语法选择

关于语法存在多种不同的意见。这里主要有两个自由度：使用什么关键字（例如 `var`、`auto` 等），以及是否为不可变局部变量提供一个单独的新形式（例如 `val`、`let`）。我们考虑了以下语法选项：

* `var x = expr` 仅（如 C#）
* `var`，加上 `val` 用于不可变局部变量（如 Scala、Kotlin）
* `var`，加上 `let` 用于不可变局部变量（如 Swift）
* `auto x = expr`（如 C++）
* `const x = expr`（已经是一个保留字）
* `final x = expr`（已经是一个保留字）
* `let x = expr`
* `def x = expr`（类似于 Groovy）
* `x := expr`（类似于 Go）

在收集了大量意见后，`var` 明显比 Groovy、C++或 Go 的方法更受欢迎。对于不可变局部变量的第二种语法形式（`val`、`let`）存在较大的意见分歧；这将是额外仪式与额外设计意图捕获之间的权衡。最终，我们选择仅支持 `var`。有关其合理性的详细信息，可以在这里找到 [。](http://mail.openjdk.java.net/pipermail/platform-jep-discuss/2016-December/000066.html)

#### 非指称类型

有时初始值的类型是非指称类型，例如捕获变量类型、交叉类型或匿名类类型。在这种情况下，我们有选择：i) 推断类型，ii) 拒绝表达式，或 iii) 推断一个指称的超类型。

编译器（以及细心的程序员！）必须已经习惯了处理非指称类型。然而，将它们用作局部变量的类型将显著增加它们的曝光率，暴露编译器/规范错误，并迫使程序员更频繁地面对它们。从教学的角度来看，显式类型和隐式类型声明之间有一个简单的语法转换是件好事。

所以说，仅仅因为类型不可指代就拒绝初始化器（这常常让程序员感到意外，例如在声明 `var c = getClass()` 时）是不够严谨且无益的。而且映射到超类型可能会出乎意料并且造成信息丢失。

这些考虑使我们得出了不同的答案：

* 类型为 `null` 的变量实际上没有用，对于推断类型也没有好的替代方案，所以我们拒绝这些。
* 允许捕获变量流入后续语句为语言增加了新的表达能力，但这不是这个特性的目标。相反，我们需要&#x7684;_&#x6295;&#x5F71;_&#x64CD;作本来就需要用来解决类型系统中的各种错误（例如，[JDK-8016196](https://bugs.openjdk.java.net/browse/JDK-8016196)），因此在这里应用它是合理的。
* 交集类型特别难以映射到一个超类型——它们是无序的，所以交集中的一个元素并不一定“优于”其他元素。超类型的稳定选择是所有元素的所有下确界，但那通常会是 `Object` 或类似的无用之物。所以我们允许它们。
* 匿名类类型不能命名，但它们很容易理解——它们就是类。允许变量具有匿名类类型为声明本地类的单例实例引入了一种有用的简写方式。我们允许它们。

### 风险和假设

风险：因为 Java 已经在右侧（lambda 形式参数、泛型方法类型参数、菱形）进行了大量的类型推断，因此尝试在左侧使用 `var` 可能会失败，并且可能会出现难以阅读的错误消息。

我们已经通过在左侧进行推断时使用简化的错误消息来缓解了这个问题。

示例:

```
Main.java:81: error: cannot infer type for local
variable x
        var x;
            ^
  (cannot use 'val' on variable without initializer)
​
Main.java:82: error: cannot infer type for local
variable f
        var f = () -> { };
            ^
  (lambda expression needs an explicit target-type) 
​
Main.java:83: error: cannot infer type for local
variable g
        var g = null;
            ^
  (variable initializer is 'null')
​
Main.java:84: error: cannot infer type for local
variable c
        var c = l();
            ^
  (inferred type is non denotable)
​
Main.java:195: error: cannot infer type for local variable m
        var m = this::l;
            ^
  (method reference needs an explicit target-type)
​
Main.java:199: error: cannot infer type for local variable k
        var k = { 1 , 2 };
            ^
  (array initializer needs an explicit target-type)
```

风险：源不兼容（有人可能已经将 `var` 用作类型名称。）

通过保留类型名称来缓解；像 `var` 这样的名称不符合类型的命名规范，因此不太可能被用作类型。名称 `var` _&#x662F;_&#x901A;常用作标识符；我们继续允许这样做。

风险：降低可读性，重构时出现意外。

像任何其他语言特性一样，局部变量类型推断可以用来编写清晰和不清晰的代码；最终编写清晰代码的责任在于用户。请参阅使用 `var` 的[风格指南 ](https://openjdk.org/projects/amber/guides/lvti-style-guide)，以及[常见问题解答 ](https://openjdk.org/projects/amber/guides/lvti-faq)。

## JEP 296：将 JDK 森林整合到单个存储库中

### 摘要

将 JDK 森林的众多仓库合并为一个仓库，以简化并精简开发。

### 非目标

将 FX 源代码添加到 JDK 森林并不是提案的一部分。

### 动机

多年来，JDK 的完整代码库已被分割到多个 Mercurial 仓库中。在 JDK 9 中有八个仓库：root、corba、hotspot、jaxp、jaxws、jdk、langtools 和 nashorn。

虽然这种多仓库模式提供了一些优势，但它也有许多缺点，并且无法很好地支持各种期望的源代码管理操作。特别是，无法跨多个相互依赖的变更集执行原子提交。例如，如果单个错误修复或 RFE 的代码跨越了 jdk 和 hotspot 仓库，那么在托管这两个不同仓库的森林中，对这两个仓库的更改不能原子地完成。跨越多个仓库的更改很常见；超过 1,100 个错误 ID 在 JDK 森林的各个仓库中重复使用。1,100 多个跨仓库的错误只是一个逻辑上跨仓库错误的下限，因为一些工程师使用不同的错误 ID 将更改推送到不同的仓库。

这与现代源代码管理的主要优势之间的不匹配，即跟踪文件集的变化而不是单个文件的变化，削弱了优势。作为推论，这种源代码管理事务与逻辑事务之间的不匹配，使得使用诸如 Mercurial 直观分析等工具变得复杂。

单个仓库没有与 JDK 整体分开的开发周期；所有仓库都与 JDK 推广周期同步前进。仓库的多样性给新开发者带来了比必要的更大的进入障碍，并导致了诸如“获取源代码”脚本等解决方案。

### 描述

为了解决这些问题，一个整合森林的原型已经被开发出来。原型可以在以下位置获得：

```
http://hg.openjdk.java.net/jdk10/consol-proto/
```

一些用于创建原型的辅助转换脚本已作为 `unify.zip` 附件提供。

在原型中，八个仓库已通过自动化转换脚本合并为一个仓库，该脚本在文件级别保留了历史记录，整合后的森林在标记 JDK 发布的标签处同步。变更集注释和创建日期也得以保留。

原型还有另一级别的代码重组。在整合后的森林中，Java 模块的代码通常合并到单个顶层 src 目录下。例如，目前在 JDK 森林中有基于模块的目录，如

```
$ROOT/jdk/src/java.base
...
$ROOT/langtools/src/java.compiler
...
```

在整合后的森林中，这些代码被组织为

```
$ROOT/src/java.base
$ROOT/src/java.compiler
...
```

因此，在仓库的根目录下，模块中源文件的相对路径在整合后和 src 目录组合时得以保留。

对测试目录也进行了类似但不太激进的重组，从

```
$ROOT/jdk/test/Foo.java
$ROOT/langtools/test/Bar.java
```

到

```
$ROOT/test/jdk/Foo.java
$ROOT/test/langtools/Bar.java
```

由于目前的工作是一个原型，因此它的一些部分并不完全完整，某些方面的适配和完成度可以改进。HotSpot C/C++ 源代码被移至共享的 src 目录，与模块化的 Java 代码一起。

虽然回归测试可以在当前的原型状态下运行，但 jtreg 配置文件的进一步整合是可能的，并且未来可能会进行。

### 备选方案

另一种选择是简单地保持当前的一组仓库。在迁移到单个仓库时，一些或所有仓库的历史记录可能会丢失，但这被拒绝了。考虑过整合核心子集的仓库，但最终选择了单个仓库的简单性。

### 测试

为了验证文件内容，对于每个发布标签，使用了一个脚本来验证在该标签下分割森林的内容与整合仓库在该标签下的内容是否匹配。对于最近的 JDK 9 标签，在同一标签下分割森林和整合森林的构建被比较；只有微小且可解释的差异。

### 风险与假设

上述测试应能缓解文件损坏和错误构建的最严重风险。虽然原型中所需工作的主要部分已经完成，但在整合投入生产之前，各种较小的支持功能可能尚未完成。整合前后的代码库在 Mercurial 意义上没有关联。在进行正向和反向移植时，需要使用（经过适当调整的路径的）差异，而不是导出和导入变更集。

## JEP 304：垃圾回收器接口

### 摘要

通过引入一个干净的垃圾收集器（GC）接口，提高不同垃圾收集器之间的源代码隔离。

### 目标

* 为 HotSpot 内部 GC 代码提供更好的模块化
* 使向 HotSpot 添加新的 GC 更简单，而不会干扰当前代码库
* 使从 JDK 构建中排除 GC 更加容易

### 非目标

* 实际上添加或删除 GC 并非目标。
* 这项工作将推动 HotSpot 中 GC 算法的构建时隔离，但&#x5B83;_&#x5E76;&#x975E;_&#x5B9E;现完全构建时隔离的目标（那是另一个 JEP 的目标）。

### 成功指标

* 如果 GC 实现主要包含在其各自的 `src/hotspot/share/gc/$NAME` 目录中，并且可能包含在 `src/hotspot/cpu/share/gc/$NAME` 目录中，则该实现将被视为成功。那些目录之外的最小代码应包括来自这些目录的文件，并且 GC 特定的 `if`-`else` 分支应该非常少。
* 由于这次重构，性能不应该下降。

### 动机

每个垃圾回收器的实现目前都由其 `src/hotspot/share/gc/$NAME` 目录中的源文件组成，例如 G1 位于 `src/hotspot/share/gc/g1`，CMS 位于 `src/hotspot/share/gc/cms` 等。然而，HotSpot 源代码中到处都是零散的片段。例如，大多数垃圾回收器需要某些屏障，这些屏障需要在运行时、解释器、C1 和 C2 中实现。这些屏障不包含在垃圾回收器的特定目录中，而是实现在共享的解释器、C1 和 C2 源代码中（通常由长 `if`-`else`-链保护）。同样的问题也适用于诊断代码，例如 `MemoryMXBeans`。这种源代码布局有几个缺点：

1. 对于垃圾回收器开发者来说，实现一个新的垃圾回收器需要了解所有这些不同的地方，以及如何根据他们的特定需求扩展它们。
2. 对于不是垃圾回收器开发者的 HotSpot 开发者来说，查找特定垃圾回收器的特定代码的位置令人困惑。
3. 在构建时排除特定的垃圾收集器很困难。《`#define``INCLUDE_ALL_GCS`》长期以来一直是构建仅内置串行收集器的 JVM 的方法，但这个机制正变得过于僵化。

更清晰的 GC 接口将大大简化新收集器的实现，使代码更加清晰，并在构建时更容易排除一个或多个收集器。添加新的垃圾收集器应该是一套良好文档化的接口实现的问题，而不是弄清楚 HotSpot 中所有需要更改的地方。

### 描述

GC 接口将由每个垃圾收集器都需要实现的现有类《`CollectedHeap`》定义。《`CollectedHeap`》类将驱动垃圾收集器与 HotSpot 其余部分之间的大多数交互（在实例化《`CollectedHeap`》之前需要一些工具类）。更具体地说，垃圾收集器实现将必须提供：

* 堆，是 `CollectedHeap` 的一个子类
* 障碍集，是 `BarrierSet` 的一个子类，它实现了运行时所需的各类障碍
* `CollectorPolicy` 的一个实现
* `GCInterpreterSupport` 的一个实现，它为解释器实现 GC 所需的各类障碍（使用汇编指令）
* 一个 `GCC1Support` 的实现，它实现了 C1 编译器 GC 的各种屏障
* 一个 `GCC2Support` 的实现，它实现了 C2 编译器 GC 的各种屏障
* 初始化最终的 GC 特定参数
* 设置一个 `MemoryService`，相关的内存池、内存管理器等

实现细节代码应该存在于一个辅助类中，这样它就可以被不同的垃圾收集器轻松使用。例如，可以有一个辅助类来实现各种用于卡片表支持的屏障，任何需要卡片表后屏障的垃圾收集器都会调用该辅助类中相应的 方法。这样，接口提供了实现全新屏障的灵活性，同时允许以混合搭配的方式重用现有代码。

### 备选方案

另一种选择是继续使用当前架构。这在一段时间内很可能会有效，但会阻碍未来新垃圾收集算法的开发和旧算法的移除。

### 测试

这纯粹是重构。之前能正常工作的东西在之后也需要正常工作，性能不应该下降。运行标准的回归测试套件就足够了；不需要开发新的测试。

### 风险和假设

风险较低，这主要是对 HotSpot 内部代码的重构。存在性能受损的风险，例如如果引入了额外的虚拟调用。通过持续的性能测试可以减轻这种风险。

### 依赖

本 JEP 将有助于 [JEP 291: 废弃并发标记清除（CMS）垃圾收集器 ](http://openjdk.java.net/jeps/291)，因为它提供了一种隔离它的方法，并在需要时允许其他人进行维护。

本 JEP 还将有助于 [JEP 189: Shenandoah：一种超低停顿时间的垃圾收集器 ](http://openjdk.java.net/jeps/189)，并使其更改不那么侵入性。

## JEP 307: Parallel Full GC for G1

### 摘要

通过使Full GC 并行来提高 G1 的最坏情况延迟。

### 非目标

使 G1 的Full GC 在所有用例中都能匹配并行收集器的性能。

### 动机

G1 垃圾收集器在 JDK 9 中被设为默认选项。之前的默认并行收集器具有并行Full GC。为了尽量减少用户经历全量 GC 的影响，G1 的Full GC 也应该设为并行。

### 描述

G1 垃圾收集器设计用于避免全量收集，但当并发收集无法足够快地回收内存时，会回退到Full GC。当前 G1 的Full GC 实现使用单线程的标记-清除-压缩算法。我们打算并行化标记-清除-压缩算法，并使用与年轻代和混合收集相同的线程数。线程数可以通过 `-XX:ParallelGCThreads` 选项控制，但这也会影响年轻代和混合收集使用的线程数。

### 测试

* 对全 GC 时间进行分析，以确保Full GC 时间有所改善。由于 G1 的设计旨在避免Full GC，因此仅查看基准分数可能并不足够。
* 使用 VTune 或 Solaris Studio 性能分析器进行运行时分析，以找到不必要的瓶颈。

### 风险和假设

* 这项工作基于一个假设，即 G1 的基本设计不会阻止并行Full GC。
* G1 使用区域这一事实，很可能导致并行全 GC 后比单线程的Full GC 浪费更多空间。

## JEP 310：应用程序类数据共享

### 摘要

为了提高启动速度和占用空间，扩展现有的类数据共享（CDS）功能，允许应用程序类被放置在共享存档中。

### 目标

* 通过在不同 Java 进程间共享公共类元数据来减少内存占用。
* 提升启动时间。
* 扩展 CDS，允许从 JDK 的运行时镜像文件（`$JAVA_HOME/lib/modules`）和应用程序类路径中的归档类加载到内置平台和系统类加载器中。
* 扩展 CDS 以允许存档类被加载到自定义类加载器中。

### 非目标

* 本实现中使用的共享类存档存储格式不会标准化。
* 在本次发布中，CDS 无法存档来自用户定义模块的类（例如在 `--module-path` 中指定的那些类）。我们计划在未来的版本中添加该支持。

### 成功指标

如果我们能够在多个 JVM 进程中实现 Java 类元数据使用的内存显著节省，并且能够显著提高启动时间，那么这个项目将被视为成功。

为了说明目的：

* 对于一个包含 6 个 JVM 进程的 Java EE 应用服务器，我们可以节省大约 340MB 的 RAM，这些 JVM 进程总共消耗了 13GB 的 RAM（其中约 2GB 用于类元数据）。
* 我们可以将 JEdit 基准测试的启动时间提高 20-30%。
* 我们可以通过 4 个 JVM 进程将嵌入式 Felix 基准的 RAM 使用量减少 18%。

这些数字反映的是特定的基准测试，可能不适用于所有情况。这项工作的好处取决于支持的类加载器加载的类数量以及整个应用程序的堆使用情况。

### 描述

类数据共享（Class-Data Sharing）在 JDK 5 中引入，它允许将一组类预处理成一个共享的归档文件，然后在运行时内存映射以减少启动时间。它还可以在多个 JVM 共享同一个归档文件时减少动态内存占用。

目前 CDS 仅允许引导类加载器加载归档类。应用程序 CDS（AppCDS）扩展了 CDS，允许内置系统类加载器（也称为“应用类加载器”）、内置平台类加载器和自定义类加载器加载归档类。

对大规模企业应用程序的内存使用进行分析表明，此类应用程序通常会将数万个类加载到应用程序类加载器中。将 AppCDS 应用于这些应用程序将导致每个 JVM 进程节省数十到数百兆字节的内存。

对无服务器云服务的分析表明，其中许多服务在启动时会加载数千个应用程序类。AppCDS 可以使这些服务快速启动并提高整体系统响应时间。

#### 启用 AppCDS

默认情况下，类数据共享仅在 JVM 的引导类加载器中启用。指定 `-XX:+UseAppCDS` 命令行选项以启用类数据共享，用于系统类加载器（也称为“应用程序类加载器”）、平台类加载器和其他用户定义的类加载器。

#### 确定要存档的类

一个应用程序可能包含大量类，但使用 在正常操作期间，只有其中一小部分被归档。通过仅归档 使用到的类，我们可以减少文件存储大小和运行时 内存使用。为此，首先正常运行应用程序。 `-Xshare:off`，并使用 `-XX:DumpLoadedClassList` 命令行选项来记录所有已加载的类。

请注意，`-XX:DumpLoadedClassList` 默认情况下仅包括 由引导类加载器加载的类。您应该指定 `-XX:+UseAppCDS` 选项，以便系统类加载器和平台类加载器加载的类也被包含。例如：

```
java -Xshare:off -XX:+UseAppCDS -XX:DumpLoadedClassList=hello.lst -cp hello.jar HelloWorld
```

#### 创建 AppCDS 存档

要创建 AppCDS 存档，请指定 `-Xshare:dump -XX:+UseAppCDS` 命令行选项，通过 `-XX:SharedClassListFile` 选项传递类列表，并将类路径设置为 与您的应用程序所使用的相同。您还应该使用 使用 `-XX:SharedArchiveFile` 选项来指定存储类的归档文件的名称。请注意，如果未指定 `-XX:SharedArchiveFile`，则归档的类将被存储在 JDK 的安装目录中，这通常不是你想要的结果。例如：

```
$ java -Xshare:dump -XX:+UseAppCDS -XX:SharedClassListFile=hello.lst \
    -XX:SharedArchiveFile=hello.jsa -cp hello.jar
```

#### 使用 AppCDS 归档

创建 AppCDS 归档后，你可以在启动应用程序时使用它。通过指定 `-Xshare:on -XX:+UseAppCDS` 命令行选项，并使用 `-XX:SharedArchiveFile` 选项来指示归档文件的名称。例如：

```
$ java -Xshare:on -XX:+UseAppCDS -XX:SharedArchiveFile=hello.jsa \
    -cp hello.jar HelloWorld
```

#### 类路径不匹配

使用 `-Xshare:dump` 时所使用的类路径必须与使用 `-Xshare:on` 时所使用的类路径相同，或者是一个其前缀。否则，JVM 将打印关于类路径不匹配的错误消息并拒绝启动。要分析不匹配的情况，可以添加 `-Xlog:class+path=info`。 传递到应用程序的命令行，JVM 将会打印出详细信息 诊断信息，说明预期的类路径是什么，以及什么 classpath 实际上被使用。

#### 使用 `-Xshare:auto`

AppCDS 通过将存档内容映射到固定地址来工作。在某些操作系统上，特别是当地址空间布局随机化（ASLR）启用时，如果所需的地址空间不可用，内存映射操作可能会偶尔失败。如果指定了 `-Xshare:on` 选项，JVM 将将其视为错误条件并失败启动。为了使您的应用程序在这种情况下更加健壮，我们建议使用 `-Xshare:auto` 选项。这样，当 JVM 无法映射存档时，它将禁用 AppCDS 并继续正常运行应用程序。 将此视为错误条件并失败启动。为了使您的 应用程序在这种情况下更加健壮，我们建议使用 `-Xshare:auto` 选项。这样，当 JVM 无法映射存档时，它将禁用 AppCDS 并继续正常运行应用程序。

请注意，`-Xshare:auto` 也会在存在类路径不匹配的情况下禁用 AppCDS。因此，我们建议您首先使用 `-Xshare:on` 进行测试以确保没有类路径不匹配，然后在生产环境中使用 `-Xshare:auto`。 类路径不匹配。因此，我们建议您首先使用 `-Xshare:on` 进行测试以确保没有类路径不匹配，然后使用 `-Xshare:auto` 在生产环境中。 `-Xshare:auto` 在生产环境中。

#### 列出从 AppCDS 存档加载的类

要找出从 AppCDS 存档加载了哪些类，您可以使用 `-Xlog:class+load=info` 命令行选项，该选项会打印 出每个已加载类的名称，以及类从何处加载。从 CDS 存档加载的类将被打印为 `来源：共享对象文件` 。例如： `来源：共享对象文件` 。例如：

```
$ java -Xshare:on   -XX:+UseAppCDS -XX:SharedArchiveFile=hello.jsa \
    -cp hello.jar -Xlog:class+load=info HelloWorld | grep HelloWorld
[0.272s][info][class,load] HelloWorld source: shared objects file
```

#### 实现

* _平台和系统类加载器：_&#x48;otSpot 虚拟机会识别由内置平台和系统类加载器发起的类加载请求。当这些加载器请求一个存在于 CDS 归档中的类时，虚拟机会跳过通常的类文件解析和验证步骤，直接加载归档中的类副本。
* _自定义类加载器：_ 当自定义类加载器调用 `ClassLoader::defineClass`，虚拟机会通过比较类文件数据的指纹来尝试将类文件的内容与归档中的类进行匹配。如果找到匹配项，虚拟机会跳过类文件解析和验证步骤，直接加载归档中的类副本。

### 备选方案

我们曾考虑使用共享内存区域来共享由多个活跃 JVM 进程动态加载的类，但我们发现共享潜力较低且实现难度较大。

我们反而选择使应用程序类数据共享更加静态：

* 需要一个额外的'转储'步骤。
* 当应用程序的 JAR 文件更新时，需要重复转储步骤。

这是在现有的 CDS 基础设施之上构建的，因此实现更简单，我们可以实现与目标用例更高的共享比例。

### 测试

需要进行广泛的测试，以确保兼容性并确认性能优势。

应在所有支持平台上进行测试。在某些平台（尤其是 Windows/x86）上，如果 JVM 无法映射存档，由于地址空间布局随机化（ASLR），测试可能会失败。

### 风险和假设

AppCDS 先前在 Oracle JDK 中为 JDK 8 和 JDK 9 中实现。此 JEP 将源代码移至开源仓库，以使该功能普遍可用。由于 AppCDS 在 JDK 8 和 JDK 9 中经过了广泛测试，因此兼容性和稳定性的风险较低。

## JEP 312：线程本地握手

### 摘要

引入一种在执行回调时无需进行全局虚拟机安全点的方式。使其既可能又经济，可以停止单个线程，而不仅仅是所有线程或没有线程。

### 非目标

在所有支持的架构上高效实现这一点可能并不可行。最初的目标并不是支持所有处理器架构和所有处理器架构版本。

### 成功指标

* 新机制在标准基准测试中的性能开销不超过1%。
* 新机制不会增加达到传统全局安全点的所需时间。

### 动机

能够停止单个线程具有多种应用：

* 改进有偏见的锁撤销，仅停止单个线程以撤销偏见，而不是全部。
* 减少不同类型的服务查询对整体 VM 延迟的影响，例如获取所有线程的堆栈跟踪，这在具有大量 Java 线程的 VM 上可能是一个慢操作。
* 通过减少对信号的依赖来执行更安全的堆栈跟踪采样。
* 使用所谓的非对称 Dekker 同步技术来省略一些内存屏障，通过与 Java 线程进行握手。例如，G1 和 CMS 使用的条件卡标记代码本质上不需要内存屏障。因此，G1 的写后屏障可以进行优化，并且可以删除尝试避免内存屏障的分支。

所有这些都有助于虚拟机通过减少全局安全点的数量来降低延迟。

### 描述

握手操作是一个回调，当每个 JavaThread 处于安全状态时执行。回调由线程本身或虚拟机线程执行，同时保持线程阻塞状态。安全点与握手之间的主要区别在于，每个线程的操作将尽快在所有线程上执行，并且在其自己的操作完成后立即继续执行。如果知道 JavaThread 正在运行，那么也可以与该单个 JavaThread 进行握手。

初始实现中，同一时间将最多有一个握手操作在执行。该操作可以涉及所有 JavaThreads 的任意子集。虚拟机线程将通过一个虚拟机操作来协调握手操作，这将实际上在握手操作期间防止全局安全点发生。

当前的安全点方案被修改为通过每个线程的指针进行间接操作，这将允许单个线程的执行被强制在守卫页面上触发。本质上，任何时候都将有两个轮询页面：一个始终被守卫，一个始终未被守卫。为了强制线程让出，虚拟机将对应线程的每个线程指针更新为指向守卫页面。

线程本地握手将首先在 x64 和 SPARC 平台上实现。其他平台将回退到正常的安全点。一个新的产品选项 `-XX:ThreadLocalHandshakes`（默认值 `true`）允许用户在支持的平台选择正常安全点。

### 备选方案

考虑了多种替代方案：

* 发出条件分支。这会消耗分支预测器状态，并且不如直接加载紧密。在这个领域的实验表明，条件分支的性能可能高度依赖于目标 CPU 的具体微架构。条件分支方法的另一个缺点是，每个条件分支安全点都需要一个相应的桩来输出，以处理返回到轮询位置。
* 有一个想法是牺牲另一个寄存器，然后执行将寄存器持有的地址加载到寄存器本身的操作，假设寄存器的内容是其自己的线程局部字段的地址。线程局部握手开始时，会将字段更改为 `NULL`。下一次轮询时，寄存器会被设置为 `NULL`，而对于第二次轮询，加载操作会触发异常。这需要全局牺牲一个寄存器，触发异常更昂贵，一旦请求线程停止，平均需要两次轮询才能达到安全点。好处是理论上对应用程序执行的影响较低。
* 之前构建了一个原型，其中全局轮询页保持原样，但在虚拟机代码中仅捕获了实际的目标线程。未成为握手目标的手续线程会直接从信号处理程序返回并继续执行。这种方法的缺点是，如果目标线程响应缓慢，则这可能导致其他 Java 线程出现信号风暴，因为轮询页无法在目标线程响应之前解除武装。

## JEP 313：移除本地头文件生成工具（javah）

### 摘要

从 JDK 中移除 `javah` 工具。

### 动机

该工具已被 JDK 8 中添加的更优越的功能所取代（[JDK-7150368](https://bugs.openjdk.java.net/browse/JDK-7150368)）。这项功能能够在 Java 源代码编译时编写原生头文件，从而消除了对单独工具的需求。

专注于 `javac` 提供的支持可以消除升级 `javah` 以支持最近的新范例的需求，例如通过 `javax.tools.*` 中的编译器 API 访问 API，或 JDK 9 中添加的新 `java.util.spi.ToolProvider` SPI。 实现移除将包括从 Mercurial 仓库中删除受影响的文件，包括文档文件，以及支持 makefile 的更改。

### 描述

任何测试都将限于验证 `javah` 命令不存在。

### 测试

专注于 `javah` 提供的支持可以消除升级 `javah` 以支持最近的新范例的需求，例如通过 `javax.tools.*` 中的编译器 API 访问 API，或 JDK 9 中添加的新 `java.util.spi.ToolProvider` SPI。

### 风险和假设

从 JDK 中移除 `javah` 工具没有工程问题，因为该工具不再被 JDK 使用，或在构建 JDK 时不再使用。

用户自 JDK 9 起已被警告即将移除，每次调用 `javah` 工具时都会生成警告。

### 依赖

JDK 没有直接依赖于 `javah` 工具。外部有一些 `javah` 的衍生工具，例如 Ant [javah 任务 ](https://ant.apache.org/manual/Tasks/javah.html)，但正如建议 `javah` 命令的用户使用 `javac -h` 一样，这些依赖的用户也建议使用 `javac` 提供的相应支持。

## JEP 314：附加 Unicode 语言标记扩展

### 摘要

增强 `java.util.Locale` 和相关 API 以实现 BCP 47 语言标签的额外 Unicode 扩展。

### 目标

对 [BCP 47](http://www.rfc-editor.org/rfc/bcp/bcp47.txt) 语言标签的支持最初是在 Java SE 7 中添加的，但对 Unicode 位置扩展的支持仅限于日历和数字。此 JEP 将在相关的 JDK 类中实现最新 [LDML 规范 ](http://www.unicode.org/reports/tr35/tr35.html#Locale_Extension_Key_and_Type_Data)中指定的更多扩展。

### 非目标

除了下面描述的那些之外，其他的 Unicode 语言标记扩展将被忽略。

### 描述

自 Java SE 9 起，[ 支持的 BCP 47 U 语言标记扩展](http://www.oracle.com/technetwork/java/javase/documentation/java9locales-3559485.html)是 `ca` 和 `nu`。本 JEP 将添加对以下额外扩展的支持：

* `cu`（货币类型）
* `fw`（星期第一天）
* `rg`（地区覆盖）
* `tz`（时区）

为了支持这些附加扩展，将对以下 API 进行更改：

* `java.text.DateFormat::get*Instance` 将根据扩展 `ca`、`rg` 和/或 `tz` 返回实例
* `java.text.DateFormatSymbols::getInstance` 将根据扩展 `rg` 返回实例
* `java.text.DecimalFormatSymbols::getInstance` 将根据扩展 `rg` 返回实例
* `java.text.NumberFormat::get*Instance` 将根据扩展 `nu` 和/或 `rg` 返回实例
* `java.time.format.DateTimeFormatter::localizedBy` 将根据扩展 `ca`、`rg` 和/或 `tz` 返回 `DateTimeFormatter` 实例
* `java.time.format.DateTimeFormatterBuilder::getLocalizedDateTimePattern` 将返回基于 `rg` 扩展的模式字符串。
* `java.time.format.DecimalStyle::of` 将返回基于扩展 `nu`，和/或 `rg` 的 `DecimalStyle` 实例。
* `java.time.temporal.WeekFields::of` 将返回基于扩展 `fw` 和/或 `rg` 的 `WeekFields` 实例。
* `java.util.Calendar::{getFirstDayOfWeek,getMinimalDaysInWeek}` 将返回基于扩展 `fw` 和/或 `rg` 的值。
* `java.util.Currency::getInstance` 将根据扩展 `cu` 和/或 `rg` 返回 `Currency` 实例
* `java.util.Locale::getDisplayName` 将返回一个包含这些 U 扩展的显示名称的字符串
* `java.util.spi.LocaleNameProvider` 将为这些 U 扩展的键和类型提供新的 SPI

### 风险和假设

`Locale::getDisplayName` 返回的显示名称取决于每个区域设置提供者提供的本地化数据

## JEP 316：在替代内存设备上进行堆分配

### 摘要

使 HotSpot 虚拟机能够在用户指定的替代内存设备（如 NV-DIMM）上分配 Java 对象堆。

### 动机

随着廉价 NV-DIMM 内存的普及，未来的系统可能会配备异构内存架构。其中一种此类技术是英特尔的三维 XPoint。这种架构除了 DRAM 之外，还将包含一种或多种具有不同特性的非 DRAM 内存。

该 JEP 针对具有与 DRAM 相同语义的替代内存设备，包括原子操作的语义，因此可以在不修改现有应用程序代码的情况下，将这些设备用作对象堆，而所有其他内存结构（如代码堆、元空间、线程栈等）将继续驻留在 DRAM 中。

该提案的一些用例包括：

1. 在多 JVM 部署中，某些 JVM（如守护进程、服务等）的优先级低于其他 JVM。与 DRAM 相比，NV-DIMM 的访问延迟可能更高。低优先级进程可以使用 NV-DIMM 内存作为堆，允许高优先级进程使用更多 DRAM。
2. 大数据和内存数据库等应用程序对内存的需求不断增长。此类应用程序可以使用 NV-DIMM 作为堆，因为与 DRAM 相比，NV-DIMM 的容量可能更大，成本更低。

### 描述

一些操作系统已经通过文件系统暴露了非 DRAM 内存。例如 [NTFS DAX 模式](https://channel9.msdn.com/events/build/2016/p470)和 [ext4 DAX](https://lwn.net/Articles/618064)。这些文件系统中的内存映射文件可以绕过页面缓存，并提供虚拟内存到设备物理内存的直接映射。

为了在这样的内存中分配堆，我们可以添加一个新的选项，`-XX:AllocateHeapAt=< 路径>`。这个选项将接受一个文件系统的路径，并使用内存映射来实现将对象堆分配到内存设备上的预期结果。该 JEP 不打算在多个运行的 JVM 之间共享非易失性区域，或在 JVM 的进一步调用中重用同一区域。

现有的与堆相关的标志，如 `-Xmx`、`-Xms` 等，以及与垃圾回收相关的标志将继续按原样工作。

为了确保应用程序安全，实现必须确保在文件系统中创建的文件满足以下要求：

1. 受到正确的权限保护，以防止其他用户访问。
2. 在任何可能的情况下，当应用程序终止时将被移除。

### 测试

测试不一定需要任何特殊内存；它可以在 ramfs 或 tmpfs 等内存文件系统上进行。

## JEP 317：实验性 Java JIT 编译器

### 摘要

使基于 Java 的 JIT 编译器 Graal 能够在 Linux/x64 平台上作为实验性 JIT 编译器使用。

### 非目标

目标并不是要达到或超过现有 JIT 编译器的性能。

### 动机

Graal，一个基于 Java 的 JIT 编译器，是 JDK 9 中引入的实验性 ahead-of-time (AOT) 编译器的基础。启用它作为实验性 JIT 编译器是 [Project Metropolis](http://mail.openjdk.java.net/pipermail/announce/2017-September/000233.html) 项目的一项举措，也是研究基于 Java 的 JIT 编译器在 JDK 中可行性的下一步。

### 描述

从 Linux/x64 平台开始启用 Graal 作为实验性 JIT 编译器。Graal 将使用 JDK 9 中引入的 JVM 编译器接口（JVMCI）。Graal 已经存在于 JDK 中，因此将其作为实验性 JIT 启用主要将是一项测试和调试工作。

要启用 Graal 作为 JIT 编译器，请在 `java` 命令行上使用以下选项：

```
-XX:+UnlockExperimentalVMOptions -XX:+UseJVMCICompiler
```

### 测试

计划对此项工作进行标准的编译器测试。这包括在 Linux/x64 上使用各种标志选项运行所有 HotSpot 和 JDK 测试。除了这些标准测试外，还将运行专门为 Graal 开发的单元测试。将实现支持在 `jtreg` 框架中运行这些单元测试。这项工作的一部分将进行初始的性能测试和基准测试。

### 风险和假设

这个问题会导致启动性能变慢和 Java 堆内存使用增加，从而影响和/或限制初始开发者评估。某些应用程序和基准测试在 Graal 与现有的 HotSpot JIT 编译器相比时，会看到性能差距。

## JEP 319: 根证书

### 摘要

在 JDK 中提供一组默认的根证书颁发机构（CA）证书。

### 目标

将 Oracle Java SE 根 CA 计划中的根证书开源，以使 OpenJDK 构建对开发者更具吸引力，并减少这些构建与 Oracle JDK 构建之间的差异。

### 动机

JDK 中的 `cacerts` 密钥库，它是 [JDK 的一部分 ](https://docs.oracle.com/javase/9/tools/keytool.htm#GUID-5990A2E4-78E3-47B7-AE75-6D1826259549__CACERTS)，旨在包含一组根证书，这些证书可用于在多种安全协议中使用的证书链中建立信任。然而，JDK 源代码中的 `cacerts` 密钥库目前是空的。因此，TLS 等关键安全组件在 OpenJDK 构建中默认情况下无法工作。为解决这个问题，用户必须按照文档（例如，在 [JDK 9 发布说明](http://www.oracle.com/technetwork/java/javase/9all-relnotes-3704433.html#JDK-8189131)中记录的）配置并填充 `cacerts` 密钥库中的根证书集。

### 描述

Oracle Java SE 根证书计划中 CAs 发行的根证书将填充到 `cacerts` 密钥库中。作为前提条件，每个 CA 必须签署 [Oracle 贡献者协议 (OCA)](http://www.oracle.com/technetwork/community/oca-486395.html) 或等效协议，以授予 Oracle 开源其证书的权利。以下是已签署所需协议的 CA 列表，以及每个 CA 将包含的根证书（通过 Distinguished Name 标识）的列表。此列表包括 Oracle Java SE 根证书计划中当前大多数成员 CA。那些未签署协议的 CA 将不会在此版本中包含。那些处理时间较长的 CA 将在下一个版本中包含。

#### Actalis S.p.A.

1. CN=Actalis 身份验证根 CA, O=Actalis S.p.A./03358520967, L=Milan, C=IT

#### Buypass AS

1. CN=布普斯第二类根证书颁发机构, O=布普斯 AS-983163327, C=挪威
2. CN=布普斯第三类根证书颁发机构, O=布普斯 AS-983163327, C=挪威

#### 卡梅尔法瑞玛

1. CN=商会的根证书, OU=[http://www.chambersign.org](http://www.chambersign.org/), O=AC 卡梅尔法瑞玛 SA CIF A82743287, C=欧盟
2. CN=商会根证书 - 2008, O=AC Camerfirma S.A., SERIALNUMBER=A82743287, L=马德里（请参阅当前地址 [www.camerfirma.com/address](https://www.camerfirma.com/address)），C=欧盟
3. CN=全球商会根证书 - 2008, O=AC Camerfirma S.A., SERIALNUMBER=A82743287, L=马德里（请参阅当前地址 [www.camerfirma.com/address](https://www.camerfirma.com/address)），C=欧盟

#### Certum

1. CN=Certum CA, O=Unizeto Sp. z o.o., C=波兰
2. CN=Certum 可信网络 CA, OU=Certum 认证中心, O=Unizeto Technologies S.A., C=PL

#### 中兴通讯有限公司

1. OU=ePKI 根认证中心, O="中兴通讯有限公司", C=TW

#### Comodo CA Ltd.

1. CN=AddTrust 类 1 CA 根, OU=AddTrust TTP 网络, O=AddTrust AB, C=SE
2. CN=AddTrust 外部 CA 根, OU=AddTrust 外部 TTP 网络, O=AddTrust AB, C=SE
3. CN=AddTrust 合格 CA 根, OU=AddTrust TTP 网络, O=AddTrust AB, C=SE
4. CN=AAA 证书服务, O=Comodo CA 有限公司, L=萨里福德, ST=大曼彻斯特, C=GB
5. CN=COMODO ECC 认证机构, O=COMODO CA 有限公司, L=萨福克, ST=大曼彻斯特, C=GB
6. CN=COMODO RSA 认证机构, O=COMODO CA 有限公司, L=萨福克, ST=大曼彻斯特, C=GB
7. CN=USERTrust ECC 认证机构, O=USERTRUST 网络, L=泽西城, ST=新泽西, C=US
8. CN=USERTrust RSA 认证机构, O=USERTRUST 网络, L=泽西城, ST=新泽西, C=US
9. CN=UTN 用户第一客户端认证和电子邮件, OU=[http://www.usertrust.com](http://www.usertrust.com/), O=USERTRUST 网络, L=盐湖城, ST=犹他州, C=美国
10. CN=UTN 用户第一硬件, OU=[http://www.usertrust.com](http://www.usertrust.com/), O=USERTRUST 网络, L=盐湖城, ST=犹他州, C=美国
11. CN=UTN 用户第一对象, OU=[http://www.usertrust.com](http://www.usertrust.com/), O=USERTRUST 网络, L=盐湖城, ST=犹他州, C=美国

#### DigiCert 公司

1. CN=巴尔的摩赛博信任根证书, OU=赛博信任, O=巴尔的摩, C=爱尔兰
2. CN=巴尔的摩赛博信任代码签名根证书, OU=赛博信任, O=巴尔的摩, C=爱尔兰
3. CN=迪吉多全球根 CA, OU=[www.digicert.com](https://www.digicert.com), O=迪吉多公司, C=美国
4. CN=迪吉多全球根 G2, OU=[www.digicert.com](https://www.digicert.com), O=迪吉多公司, C=美国
5. CN=迪吉多全球根证书 G3, OU=[www.digicert.com](https://www.digicert.com), O=迪吉多公司, C=美国
6. CN=迪吉多可信根证书 G4, OU=[www.digicert.com](https://www.digicert.com), O=迪吉多公司, C=美国
7. CN=迪吉多保证 ID 根 CA, OU=[www.digicert.com](https://www.digicert.com), O=迪吉多公司, C=美国
8. CN=迪吉多保证 ID 根 G2, OU=[www.digicert.com](https://www.digicert.com), O=迪吉多公司, C=美国
9. CN=DigiCert Assured ID Root G3, OU=[www.digicert.com](https://www.digicert.com), O=DigiCert Inc, C=US
10. CN=数字证书认证公司高级 EV 根证书颁发机构, OU=[www.digicert.com](https://www.digicert.com), O=数字证书认证公司, C=美国
11. OU=Equifax 安全证书颁发机构, O=Equifax, C=美国
12. CN=Equifax 安全电子商务 CA-1, O=Equifax 安全公司, C=美国
13. CN=Equifax 安全全球电子商务 CA-1, O=Equifax 安全公司, C=美国
14. CN=GeoTrust 全球 CA, O=GeoTrust 公司, C=美国
15. CN=GeoTrust 主要认证机构, O=GeoTrust 公司, C=美国
16. CN=GeoTrust 主要认证机构 - G2, OU=(c) 2007 GeoTrust 公司 - 仅限授权使用, O=GeoTrust 公司, C=美国
17. CN=GeoTrust 主要认证机构 - G3, OU=(c) 2008 GeoTrust 公司 - 仅限授权使用, O=GeoTrust 公司, C=美国
18. CN=GeoTrust 通用 CA, O=GeoTrust 公司, C=美国
19. CN=GTE CyberTrust 全球根, OU="GTE CyberTrust 解决方案公司", O=GTE 公司, C=美国
20. CN=thawte Primary Root CA, OU="(c) 2006 thawte, Inc. - For authorized use only", OU=Certification Services Division, O="thawte, Inc.", C=US
21. CN=thawte 根证书颁发机构 - G2, OU="(c) 2007 thawte, Inc. - 仅限授权使用", O="thawte, Inc.", C=US
22. CN=thawte 根证书颁发机构 - G3, OU="(c) 2008 thawte, Inc. - 仅限授权使用", OU=认证服务部门, O="thawte, Inc.", C=US
23. EMAILADDRESS=[premium-server@thawte.com](mailto:premium-server@thawte.com), CN=Thawte Premium Server CA, OU=Certification Services Division, O=Thawte Consulting cc, L=Cape Town, ST=Western Cape, C=ZA
24. CN=Thawte 时间戳证书颁发机构, OU=Thawte 认证, O=Thawte, L=Durbanville, ST=西开普省, C=ZA
25. OU=第一类公共主要证书颁发机构, O="VeriSign, Inc.", C=US
26. OU=VeriSign 信任网络, OU="(c) 1998 VeriSign, Inc. - 仅限授权使用", OU=第一类公共主要证书颁发机构 - G2, O="VeriSign, Inc.", C=US
27. CN=VeriSign 类 1 公共主要认证机构 - G3, OU="(c) 1999 VeriSign, Inc. - 仅限授权使用", OU=VeriSign 信任网络, O="VeriSign, Inc.", C=US
28. OU=VeriSign Trust Network, OU="(c) 1998 VeriSign, Inc. - 仅限授权使用", OU=Class 2 公共主要认证机构 - G2, O="VeriSign, Inc.", C=US
29. CN=VeriSign 类 2 公共主要认证机构 - G3, OU="(c) 1999 VeriSign, Inc. - 仅限授权使用", OU=VeriSign 信任网络, O="VeriSign, Inc.", C=US
30. 组织单元=类 3 公共主要认证机构, 组织="VeriSign, Inc.", 国家=美国
31. 组织单元=VeriSign 信任网络, 组织单元="(c) 1998 VeriSign, Inc. - 仅限授权使用", 组织单元=类 3 公共主要认证机构 - G2, 组织="VeriSign, Inc.", 国家=美国
32. 共享名=VeriSign 类 3 公共主要认证机构 - G3, 组织单元="(c) 1999 VeriSign, Inc. - 仅限授权使用", 组织单元=VeriSign 信任网络, 组织="VeriSign, Inc.", 国家=美国
33. CN=VeriSign 类 3 公共主要认证机构 - G4, OU="(c) 2007 VeriSign, Inc. - 仅限授权使用", OU=VeriSign 信任网络, O="VeriSign, Inc.", C=US
34. CN=维信集团第三类公共主要认证中心 - G5, OU="(c) 2006 维信集团，仅限授权使用", OU=维信信任网络, O="维信集团", C=US
35. CN=维信通用根认证中心, OU="(c) 2008 维信集团，仅限授权使用", OU=维信信任网络, O="维信集团", C=US

#### DocuSign

1. CN=类 2 主 CA, O=Certplus, C=FR
2. CN=Class 3P Primary CA, O=Certplus, C=FR
3. CN=KEYNECTIS 根证书, OU=根, O=KEYNECTIS, C=FR

#### D-TRUST GmbH

1. CN=D-TRUST 根证书类别 3 CA 2 2009, O=D-Trust GmbH, C=DE
2. CN=D-TRUST 根证书类别 3 CA 2 EV 2009, O=D-Trust GmbH, C=DE

#### IdenTrust

1. CN=DST 根证书 X3, O=数字签名信托公司
2. CN=IdenTrust 公共部门根证书 1, O=IdenTrust, C=美国
3. CN=IdenTrust 商业部门根证书 1, O=IdenTrust, C=美国

#### Let's Encrypt

1. CN=ISRG 根证书 X1, O=互联网安全研究组, C=美国

#### LuxTrust

1. CN=LuxTrust 全球根证书, O=LuxTrust s.a., C=卢森堡

#### QuoVadis Ltd.

1. CN=QuoVadis 根证书颁发机构, OU=根证书颁发机构, O=QuoVadis 有限公司, C=BM
2. CN=QuoVadis 根证书颁发机构 1 G3, O=QuoVadis 有限公司, C=BM
3. CN=QuoVadis 根证书颁发机构 2, O=QuoVadis 有限公司, C=BM
4. CN=QuoVadis 根证书颁发机构 2 G3, O=QuoVadis 有限公司, C=BM
5. CN=QuoVadis 根证书 3, O=QuoVadis 有限公司, C=BM
6. CN=QuoVadis 根证书 3 G3, O=QuoVadis 有限公司, C=BM

#### Secom Trust Systems

1. 安全通信根证书 1，组织=SECOM Trust.net，国家=JP
2. 安全通信根证书 2，组织=SECOM Trust Systems CO.,LTD.，国家=JP
3. 安全通信 EV 根证书 1，组织=SECOM Trust Systems CO.,LTD.，国家=JP

#### SwissSign AG

1. 瑞士签名黄金 CA - G2, O=瑞士签名 AG, C=CH
2. 瑞士签名白金 CA - G2, O=瑞士签名 AG, C=CH
3. 瑞士签名银色 CA - G2, O=瑞士签名 AG, C=CH

#### Telia

1. CN=Sonera Class2 CA, O=Sonera, C=FI

#### 信任波涛

1. CN=SecureTrust CA, O=SecureTrust Corporation, C=US
2. CN=XRamp 全球认证机构, O=XRamp 安全服务公司, OU=[www.xrampsecurity.com](https://www.xrampsecurity.com), C=美国

### 测试

将创建测试以验证 `cacerts` 密钥库的完整性，方法是验证每个根证书的 SHA-256 指纹。如果可行，还将编写测试来验证由 CA 签发的测试证书，这些证书可以追溯到包含的根证书。此外，将添加额外的测试，以确保依赖于根证书的安全组件在 OpenJDK 构建中开箱即用，无需任何额外配置。

## JEP 322：基于时间的发布版本控制

### 摘要

修订 Java SE 平台和 JDK 的版本字符串方案以及相关的版本信息，以适应当前和未来的基于时间的发布模型。

### 目标

* 重新阐述 [JEP 223](http://openjdk.java.net/jeps/223) 引入的版本号方案，使其更适合基于时间的发布模型，这些模型定义了 _功能发布_ （可以包含新功能）和 _更新发布_ （仅修复错误）。
* 允许除基于时间的发布模型之外的其他发布模型 [当前模型 ](https://mreinhold.org/blog/forward-faster#Proposal)，具有不同的节奏或带&#x6709;_&#x4E2D;间_ 小于功能发布但大于更新版本 版本。
* 保持与整体 JEP 223 版本字符串方案的兼容性。
* 使开发者或最终用户能够轻松判断一个发布版本有多旧，以便他们可以判断是否要升级到具有最新安全补丁和可能的新功能的较新版本。
* 为实现者提供一种方式，以表明一个发布版本是发布系列的一部分，而实现者对该系列提供长期支持。
* 为实现者提供一种方式，以包含和显示一个额外的、实现者特定的版本字符串，以便将发布版本与相关产品保持一致。

### 非目标

修订现有的版本字符串方案以适应与基于时间的发布模型无关的要求并不是一个目标。

修订除 `java` 启动器以外的命令行工具的版本报告输出并不是目标。这样做是可取的，但不是关键的，可以稍后进行。

### 动机

由 [JEP 223](http://openjdk.java.net/jeps/223) 引入的版本字符串方案是一个重大 相较于过去有所改进。然而，那种方案并非如此。 非常适合未来，我们打算发布新版本 Java SE 平台和 JDK 在 [严格的六个月发布周期 ](https://mreinhold.org/blog/forward-faster#Proposal)。

JEP 223 方案的主要困难在于，一个发布的版本号编码了其相对于前者的重要性和兼容性。然而，在基于时间的发布模型中，这些特性是无法预先知道的。它们会在发布开发周期中不断变化，直到最终功能被集成。因此，发布的版本号也无法预先知道。

根据 JEP 223 版本号的语义，所有参与 JDK 发布工作的人，或者构建或使用其上组件的人，最初必须谈论发布的发布日期，然后在版本号确定后切换到谈论版本号。维护库、框架和工具的开发人员必须准备好在 JDK 发布周期的后期更改检查版本号的代码。这对所有相关人员来说都既尴尬又令人困惑。

因此，这里提出的主要变化是重新构建版本号，使其不再编码兼容性和重要性，而是以发布周期为依据来表示时间的流逝。这对于基于时间的发布模型来说更合适，因为每个发布周期以及每个发布的版本号都早已确定。

### 描述

#### 版本号

一个 _版本号_ ，`$VNUM`，是由点字符（U+002E）分隔的非空元素序列。一个元素要么是零，要么是一个不带前导零的无符号整数。版本号的最后一个元素不能为零。当一个元素被递增时，所有后续的元素都会被移除。其格式如下：

```
[1-9][0-9]*((\.0)*\.[1-9][0-9]*)*
```

序列可以是任意长度，但前四个元素被赋予特定的含义，如下所示：

```
$FEATURE.$INTERIM.$UPDATE.$PATCH
```

* `$FEATURE` — 特性发布计数器， 无论发布内容如何，每个特性发布都会递增。 特性发布中可以添加特性；也可以删除特性， 如果提前至少一个特性发布期通知。 时间。在合理的情况下可能会进行不兼容的变更。（以前称为 `$MAJOR`。）
* `$INTERIM` — 临时版本计数器，用于非功能版本发布，包含兼容的 Bug 修复和增强，但没有不兼容的变更、功能移除和标准 API 的变更。（以前称为 `$MINOR`。）
* `$UPDATE` — 更新版本计数器，用于兼容性更新版本，修复安全问题、回归和较新功能中的 Bug。（以前称为 `$SECURITY`，但具有非平凡的增量规则。）
* `$PATCH` — 用于紧急补丁发布的计数器，仅在需要发布紧急版本以修复关键问题时递增。（使用额外的元素来此目的，以尽量减少对正在进行的更新发布的开发者和用户的影响。）

版本号的第五位及以后的元素保留给 JDK 代码库的下游消费者使用。第五位可用于，_e.g._，识别特定实现者的补丁版本。

版本号永远不会以尾随零元素结束。如果一个元素及其后的所有元素逻辑上都是零，则所有这些元素都将被省略。

版本号中的数字序列按数值、逐点的方式与其他序列进行比较；_e.g._，`10.0.4` 小于 `10.1.2`. 如果一个序列比另一个序列短，则较短的序列中缺失的元素被认为是小于较长序列中相应元素的；_e.g._，`10.0.2` 小于 `10.0.2.1`。

#### 六个月发布模型中的版本号

在六个月发布模型下，版本号的不同元素变化如下：

* `$FEATURE` 每六个月增加一次：2018 年 3 月的发布是 JDK 10，2018 年 9 月的发布是 JDK 11，以此类推。
* `$INTERIM` 始终为零，因为六个月模型不包括临时版本。我们在此保留它以保持灵活性，以便未来的版本修订可以包括此类版本，并说明 JDK `$N.1` 和 JDK `$N.2` 是兼容的升级版本 `$N`。例如，JDK 1.4.1 和 1.4.2 版本在本质上都是临时版本，如果按照这个方案编号，将会是 4.1 和 4.2
* `$UPDATE` 在 `$FEATURE` 增加一个月后增加，之后每三个月增加一次：2018 年 4 月的发布是 JDK 10.0.1，7 月的发布是 JDK 10.0.2，以此类推。

我们预期大多数功能发布将至少包含一个或两个重要特性，并且更新发布永远不会包含不兼容的变更。结合 `$INTERIM` 始终为零的事实，在实际应用中，这种方案通常定义的版本号与 JEP 223 方案定义的版本号差别不大。

#### 版本字符串

版本字符串的整体格式与在 [JEP 223](http://openjdk.java.net/jeps/223) 中定义的格式相同。版本字符串是一个版本号 `$VNUM`，可能后跟预发布、构建和其他可选信息，形式为： [JEP 223](http://openjdk.java.net/jeps/223)。版本字符串是一个版本号 `$VNUM`，可能后跟预发布、构建和其他可选信息，形式为：

```
$VNUM(-$PRE)?\+$BUILD(-$OPT)?
$VNUM-$PRE(-$OPT)?
$VNUM(+-$OPT)?
```

其中 `$PRE` 是一个预发布标识符（例如 `ea`），`$BUILD` 是构建编号，而 `$OPT` 是可选的构建信息。

如果一个发布是某个实施者提供长期支持的发布系列的一部分，那么 `$OPT` 的值应以"LTS"开头 `"LTS"` 开头，例如 11.0.2+13-LTS。这将导致"LTS"在 `"LTS"` 的输出中突出显示 `java --version`、 _等_ 。更多内容将在下文中讨论。

#### API

我们修订了 `Runtime.Version` API [由 JEP 223 定义](https://docs.oracle.com/javase/9/docs/api/java/lang/Runtime.Version.html)如下：

* 添加四个新的返回 `int` 类型的访问方法，用于获取版本号的主成分，如上所述：`feature()`， `interim()`、`update()` 和 `patch()`。
* 重新定义现有的访问方法 `major()`、`minor()` 和 `security()`，使其返回与 `feature()`、`interim()` 和 `update()` 相同的值，分别对应。
* 停用现有的访问器方法，但不删除，并建议使用相应的新方法。这将有助于让开发者清楚地了解版本号的新语义。

#### 系统属性

在 [JEP 223](http://openjdk.java.net/jeps/223) 中提到的系统属性中，我们添加了两个新的属性：

* `java.version.date` — 该发布的通用可用（GA）日期，以 ISO-8601 YYYY-MM-DD 格式表示。对于早期访问发布，这将是有意的 GA 日期，_i.e._，某个未来的日期。

这个新属性使得很容易就能判断一个发布版本有多老了，这样作为用户你就能理解你落后了多少。它也反映了发布版本的安全级别：如果一个给定的 GA 发布版本，如果它的版本日期不早于任何其他 GA 发布版本，那么它就包含了最新的安全修复。

* `java.vendor.version` — 一个特定于实现的产品的版本字符串，可由生产特定实现的个人或组织选择性地分配。如果在构建时未分配，则它没有值；否则，其值是一个非空字符串，该字符串与正则表达式 `\p{Graph}+` 匹配。

这个新属性使得实现者能够提供必要的附加版本信息，以便与相关产品保持一致。如果一个产品系列使用基于日期的版本，例如形式为 `$YEAR.$MONTH` 的版本，实现者可以相应地设置此属性，以便他们的 JDK 发布版本能够清晰地与其其他发布版本相关联。（此属性命名为 `java.vendor.version`，而不是更明显的 `java.implementor.version`，以与包含 `vendor` 的现有系统属性保持一致。） [现有系统属性](https://docs.oracle.com/javase/9/docs/api/java/lang/System.html#getProperties--)其名称中包含 `vendor`。

#### 启动器

`java` 启动器将如下显示版本字符串和系统属性，以假设的 JDK 10.0.1 构建 13 为例：

```
$ java --version
openjdk 10.0.1 2018-04-19
OpenJDK Runtime Environment (build 10.0.1+13)
OpenJDK 64-Bit Server VM (build 10.0.1+13, mixed mode)
$
```

类似地，对于一个假设的 JDK 11 构建版本 42，这是一个长期支持版本：

```
$ java --version
openjdk 11 2018-09-20 LTS
OpenJDK Runtime Environment (build 11+42-LTS)
OpenJDK 64-Bit Server VM (build 11+42-LTS, mixed mode)
$
```

如果实现者将供应商版本字符串分配为， _例如_ ，`18.9` 给一个 JDK 11 的长期支持版本构建，那么它将显示：

```
$ java --version
openjdk 11 2018-09-20 LTS
OpenJDK Runtime Environment 18.9 (build 11+42-LTS)
OpenJDK 64-Bit Server VM 18.9 (build 11+42-LTS, mixed mode)
$
```

详细来说 ，`java` 启动器的版本报告选项的输出将格式化为如下，其中 `${LTS}` 扩展为 `"\u0020LTS"`，如果 `$OPT` 的前三个字符是 `"LTS"`，并且 `${JVV}` 扩展为 `"\u0020${java.vendor.version}"` 如果该系统属性被定义： 是 `"LTS"`，并且 `${JVV}` 扩展为 `"\u0020${java.vendor.version}"` 如果该系统属性被定义：

```
$ java --version
openjdk ${java.version} ${java.version.date}${LTS}
${java.runtime.name}${JVV} (build ${java.runtime.version})
${java.vm.name}${JVV} (build ${java.vm.version}, ${java.vm.info})
$ 

$ java --show-version < ... >
openjdk ${java.version} ${java.version.date}${LTS}
${java.runtime.name}${JVV} (build ${java.runtime.version})
${java.vm.name}${JVV} (build ${java.vm.version}, ${java.vm.info})
[ ... ]
$ 

$ java --full-version
openjdk ${java.runtime.version}
$ 

$ java -version
openjdk version \"${java.version}\" ${java.version.date}${LTS}
${java.runtime.name}${JVV} (build ${java.runtime.version})
${java.vm.name}${JVV} (build ${java.vm.version}, ${java.vm.info})
$ 

$ java -showversion < ... >
openjdk version \"${java.version}\" ${java.version.date}${LTS}
${java.runtime.name}${JVV} (build ${java.runtime.version})
${java.vm.name}${JVV} (build ${java.vm.version}, ${java.vm.info})
[ ... ]
$ 

$ java -fullversion
openjdk full version \"${java.runtime.version}\"
$
```

#### `@since` JavaDoc 标签

与 `@since` JavaDoc 标签一起使用的值继续与系统属性 `java.specification.version` 保持一致，因此 JDK 10 中引入的 API 将被标记为 `@since 10`。

#### Mercurial 变更集标签

识别推广版本的 Mercurial 标签的通用语法保持不变：`jdk-$VNUM+$BUILD`

#### 构建配置和输出

三个现有的与版本相关的配置选项将被弃用并忽略，相关的 Make 变量将不再定义：

```
--with-version-major          VERSION_MAJOR
--with-version-minor          VERSION_MINOR
--with-version-security       VERSION_SECURITY
```

将定义五个新的选项和相应的变量：

```
--with-version-feature        VERSION_FEATURE
--with-version-interim        VERSION_INTERIM
--with-version-update         VERSION_UPDATE
--with-version-date           VERSION_DATE
--with-vendor-version-string  VENDOR_VERSION_STRING
```

（无需定义 `--with-version-patch` 和 `VERSION_PATCH`，因为它们已经存在。）

写入 JDK 镜像根目录的 `release` 文件，除了定义现有的 `JAVA_VERSION` 变量外，还将定义 `JAVA_VERSION_DATE`，其值为 `java.version.date` 系统属性，以及 `IMPLEMENTOR_VERSION`，其值为 `java.vendor.version` 系统属性，如果已定义。

### 备选方案

[六个月基于时间的发布模型的提案](https://mreinhold.org/blog/forward-faster#Proposal) 建议功能发布的版本字符串应为以下形式 `$YEAR.$MONTH`。因此，明年的三月发布将是 18.3，九月发布将是 18.9，依此类推每年。

在对此方案提出合理反对意见后，我们 [审查了版本号中编码的各种类型的信息并建议了一些替代方案 ](http://mail.openjdk.java.net/pipermail/jdk-dev/2017-October/000007.html)，然后[总结了后续的讨论并作出了回应 ](http://mail.openjdk.java.net/pipermail/jdk-dev/2017-November/000088.html)，最后[发布了一个广受好评的提案 ](http://mail.openjdk.java.net/pipermail/jdk-dev/2017-November/000089.html)，该提案因此成为此 JEP 的基础。

### 测试

这个较新的版本字符串方案与 [JEP 223](http://openjdk.java.net/jeps/223) 中定义的方案基本兼容，因此测试应该相对直接。主要区别在于第四个元素可能用于紧急补丁发布，这可能需要一些新的单元测试用例。对相关构建配置选项的更改将需要简单的手动测试。

### 风险和假设

这里描述的变更引入了三种次要的不兼容性：

* [JEP 223](http://openjdk.java.net/jeps/223) 规定了 `security()` 方法 `Runtime.Version` API 用于返回版本号中的 `$SECURITY` 元素的值。当其前面的元素 `$MINOR` 被递增时，该元素不会递增。此提案将 `$SECURITY` 元素重命名为 `$UPDATE`，并在其前面的元素 `$INTERIM`（原 `$MINOR`），将被递增。因此，将 `security()` 用 `update()` 重新定义为理论上是不兼容的变更。然而，该 API 是在 JDK 9 中引入的，并且没有计划发布非零 `$MINOR` 值的 JDK 9 版本，所以实际上这个变更应该影响很小。
* `java` 启动器的版本报告选项的输出现在在第一行的末尾包含版本日期，可能随后跟有 `"\u0020LTS"`。现有的代码在假设行的最后一个标记是版本号的情况下解析此输出可能需要调整。
* `java` 启动器的 `--version`、`--show-version`、`-version` 和 `-showversion` 选项的输出将包括第二条和第三条 `java.vendor.version` 系统属性的值，如果该值与 `java.version` 的值不同。现有的解析此输出的代码可能需要调整。 `-version`、`-showversion` 选项的输出将包括第二条和第三条 `java.vendor.version` 系统属性的值，如果该值与 `java.version` 的值不同。现有的解析此输出的代码可能需要调整。 `java.vendor.version` 系统属性的值，如果该值与 `java.version` 的值不同。现有的解析此输出的代码可能需要调整。
