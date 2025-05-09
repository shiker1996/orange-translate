# JDK 9 JEP翻译

## \[功能] JEP 102: 进程 API 更新

### 摘要

优化操作系统进程控制和管理的 API。

### 动机

当前 API 的限制往往迫使开发者求助于本地代码。

### 描述

ava SE 对本地操作系统进程的支持有限。它提供了一个基本的 API 来设置环境并启动进程。自 Java SE 7 以来，进程流可以被重定向到文件、管道，或者可以被继承。一旦启动，API 可以用来销毁进程和/或等待进程终止。

java.lang.Process 类得到了增强，以提供进程的操作系统特定进程 ID、进程信息（包括参数、命令、进程启动时间、进程累积 CPU 时间）以及进程的用户名。

java.lang.ProcessHandle 类返回操作系统提供的每个进程的信息，包括进程 ID、参数、命令、启动时间等。ProcessHandle 可以返回进程的父进程、直接子进程，以及通过 ProcessHandle 流的所有后代。

ProcessHandles 可以用来销毁进程和监控进程的存活状态。通过 ProcessHandle.onExit，可以使用 CompletableFuture 的异步机制在进程退出时安排执行某个操作。

对进程信息和进程控制访问受安全管理器权限限制，并且受限于正常的操作系统访问控制。

### 示例

比如获取本地终端的命令行：

```
    public static void main(String[] args) {
        ProcessBuilder pb = new ProcessBuilder("cmd.exe", "/c", "dir");
        try {
            Process process = pb.start();
            System.out.println("pid: " + process.toHandle().pid());
            System.out.println("info: " + process.toHandle().info());
            System.out.println("args: " + process.toHandle().info().arguments());
            System.out.println("start time: " + process.toHandle().info().startInstant());
            process.waitFor();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
```

输出信息：

```
pid: 23900
info: [user: Optional[ORANGE\18846], cmd: C:\Windows\System32\cmd.exe, startTime: Optional[2025-04-12T10:27:42.529Z], totalTime: Optional[PT0S]]
args: Optional.empty
start time: Optional[2025-04-12T10:27:42.529Z]
```

## \[孵化] JEP 110: HTTP/2 客户端

### 摘要

定义一个新的 HTTP 客户端 API，该 API 实现 HTTP/2 和 WebSocket，并可以替代传统的 `HttpURLConnection` API。该 API 将以**孵化模块**的形式提供， 如 JEP 11 中定义的那样，在 JDK 9 中交付。这意味着： [JEP 11](http://openjdk.java.net/jeps/11)，

* 该 API 和实现将不会成为 Java SE 的一部分。
* API 将位于 `jdk.incubtor` 命名空间下。
* 该模块默认情况下在编译或运行时不会解析。

### 描述

已经为 JDK 9 进行了原型设计工作，为 HTTP 客户端、请求和响应定义了单独的类。使用了构建者模式来区分可变实体和不可变产品。定义了同步阻塞模式用于发送和接收，并基于 `java.util.concurrent.CompletableFuture` 定义了异步模式。

原型是在 NIO SocketChannels 上构建的，异步行为通过选择器和外部提供的 `ExecutorServices` 实现。

原型实现是独立的，即没有更改现有堆栈，以确保兼容性，并允许采用分阶段的方法，即不必一开始就支持所有功能。

原型 API 还包括：

* 分别请求和响应，类似于 Servlet 和 HTTP 服务器 API；
* 异步通知以下事件：
  * 接收到响应头，
  * 响应错误，
  * 响应体已接收，并且
  * 服务器推送（仅限 HTTP/2）
* 通过 `SSLEngine` 的 HTTPS
* 代理
* Cookies
* 认证。

API 中最可能需要进一步工作的部分是 HTTP/2 多响应（服务器推送）和 HTTP/2 配置的支持。原型实现几乎支持所有 HTTP/1.1，但尚未支持 HTTP/2。

HTTP/2 代理将在后续的更改中实现。

## \[JVM] JEP 143：改进竞争锁

## \[JVM] JEP 158：统一 JVM 日志

## \[JVM] JEP 165: 编译器控制

## \[功能] JEP 193: 变量句柄

### 摘要

定义一种标准方法来调用对象字段和数组元素的各种 `java.util.concurrent.atomic` 和 `sun.misc.Unsafe` 操作，一组围栏操作以实现内存排序的精细控制，以及一个标准可达性围栏操作，以确保引用的对象保持强可达性。

### 动机

随着 Java 中的并发和并行编程不断扩展，程序员越来越感到沮丧，因为他们无法使用 Java 构造来对单个类字段进行原子或有序操作；例如，原子地增加 `count` 字段。到目前为止，实现这些效果的唯一方法是通过独立的 `AtomicInteger` （增加了空间开销和额外的并发问题来管理间接引用）或者在某些情况下，使用原子 `FieldUpdater` （通常遇到比操作本身更多的开销），或者使用不安全（且不可移植且不受支持的） `sun.misc.Unsafe` JVM 内联 API。内联 API 更快，因此它们已被广泛使用，这损害了安全性和可移植性。

没有这个 JEP，随着原子 API 扩展到覆盖更多的访问一致性策略（与最近的 C++11 内存模型一致）作为 Java 内存模型修订的一部分，这些问题预计会变得更加严重。

### 描述

变量句柄是对变量的类型引用，支持在多种访问模式下对变量进行读写访问。支持的变量类型包括实例字段、静态字段和数组元素。其他变量类型也在考虑之中，可能得到支持，例如数组视图、将字节数组或 char 数组视为 long 数组，以及由 `ByteBuffer` s 描述的堆外区域的位置。

变量句柄需要库增强、JVM 增强和编译器支持。此外，还需要对 Java 语言规范和 Java 虚拟机规范进行少量更新。还考虑了微小的语言增强，这些增强可以增强编译时类型检查并补充现有语法。

预计生成的规范可以以自然的方式扩展到其他原始类型值或数组类型，如果它们将来被添加到 Java 中。但这不是一个通用的交易机制，用于控制对多个变量的访问和更新。在 JEP 的过程中，可能会探索表达和实现此类结构的替代形式，这些可能成为后续 JEP 的主题。

变量句柄通过一个单一的抽象类进行建模，即 `java.lang.invoke.VarHandle` ，其中每个变量访问模式都由一个签名多态方法表示。

访问模式集代表了一个最小可行集，旨在与 C/C++11 原子操作兼容，而不依赖于 Java 内存模型的修订更新。如有需要，将添加额外的访问模式。某些访问模式可能不适用于某些变量类型，如果这样，在调用相关的 `VarHandle` 实例时将抛出 `UnsupportedOperationException` 。

访问模式被分为以下几类：

1. **读取访问模式**，例如以 volatile 内存排序效果读取变量；
2. **编写访问模式**，例如更新变量时具有释放内存顺序效果的；
3. **原子更新访问模式**，例如在具有读写 volatile 内存顺序效果的变量上执行比较并设置；
4. **数字原子更新访问模式**，例如 get-and-add，对于写入具有普通内存顺序效果，对于读取具有 acquire 内存顺序效果；
5. **位原子更新访问模式**，例如 get-and-bitwise-and，对于写入具有释放内存顺序效果，对于读取具有普通内存顺序效果；

后三种类别通常被称为读写修改模式。

访问模式方法的签名多态特性使得变量句柄能够仅使用一个抽象类就支持许多变量种类和变量类型，从而避免了变量种类和类型特定类的爆炸性增长。此外，尽管访问模式方法签名被声明为 `Object` 的可变参数数组，但这种签名多态特性确保了原始值参数不会装箱，也不会将参数打包到数组中。这使 HotSpot 解释器和 C1/C2 编译器在运行时能够实现可预测的行为和性能。

创建 `VarHandle` 实例的方法位于与产生 `MethodHandle` 实例并访问等效或类似变量种类的方法相同的区域。

创建实例和静态字段变量种类的 `VarHandle` 实例的方法位于 `java.lang.invoke.MethodHandles.Lookup` 中，并且通过在关联的接收类中查找字段的过程来创建。例如，对于在接收类 `Foo` 上名为 `i` 的类型为 `int` 的字段进行 `VarHandle` 查找的过程可能如下所示：

```
class Foo {
    int i;

    ...
}

...

class Bar {
    static final VarHandle VH_FOO_FIELD_I;

    static {
        try {
            VH_FOO_FIELD_I = MethodHandles.lookup().
                in(Foo.class).
                findVarHandle(Foo.class, "i", int.class);
        } catch (Exception e) {
            throw new Error(e);
        }
    }
}
```

对于访问字段的 `VarHandle` 查找，在生成并返回 `VarHandle` 之前，将执行与查找 `MethodHandle` 为该字段提供读写访问权限相同的精确访问控制检查（代表查找类执行）（参见 `MethodHandles.Lookup` 类中的 `find{,Static}{Getter,Setter}` 方法）。

在以下条件下，访问模式方法将抛出 `UnsupportedOperationException` ：

* 对最终字段进行 `VarHandle` 的写访问模式方法。
* 基于数字的访问模式方法（ `getAndAdd` 和 `addAndGet` ）用于引用变量类型或非数字类型（例如 `boolean` ）。
* 基于位运算的引用变量类型或 `float` 和 `double` 类型的访问模式方法（后者可能在未来的修订中取消限制）

当需要执行易失性访问的 `VarHandle` 时，字段不必标记为 `volatile` 。实际上，如果存在， `volatile` 修饰符将被忽略。这与 `java.util.concurrent.atomic.Atomic{Int, Long, Reference}FieldUpdater` 的行为不同，其中相应的字段必须标记为易失性。在某些情况下，这可能过于严格，因为已知某些易失性访问并不总是需要的。

创建基于数组的变量类型的 `VarHandle` 实例的方法位于 `java.lang.invoke.MethodHandles` 中（请参阅 `MethodHandles` 类中的 `arrayElement{Getter, Setter}` 方法）。例如，以下是如何创建指向 `int` 数组的 `VarHandle` 的：

```
VarHandle intArrayHandle = MethodHandles.arrayElementVarHandle(int[].class);
```

在以下条件下调用访问模式方法将抛出 `UnsupportedOperationException` ：

* 基于数字的访问模式方法（ `getAndAdd` 和 `addAndGet` ）用于数组组件引用变量类型或非数字类型（如 `boolean` ）
* 基于位运算的访问模式方法用于引用变量类型或 `float` 和 `double` 类型（后者限制可能在未来的修订中取消）

所有原始类型和引用类型都支持变量类型的变量种类，包括实例字段、静态字段和数组元素。其他变量种类可能支持所有或部分这些类型。

用于创建基于数组视图的变量类型的 `VarHandle` 实例的方法也位于 `java.lang.invoke.MethodHandles` 中。例如，创建一个 `VarHandle` 来将 `byte` 的数组视为未对齐的 `long` 数组的示例如下：

```
VarHandle longArrayViewHandle = MethodHandles.byteArrayViewVarHandle(
        long[].class, java.nio.ByteOrder.BIG_ENDIAN);
```

虽然可以使用 `java.nio.ByteBuffer` 实现类似机制，但它要求创建一个包装 `byte` 数组的 `ByteBuffer` 实例。这并不总是能保证可靠性能，因为逃逸分析可能很脆弱，并且访问必须通过 `ByteBuffer` 实例进行。在非对齐访问的情况下，除了普通访问模式的方法外，其他方法都会抛出 `IllegalStateException` 。在对齐访问的情况下，根据变量类型，某些易失性操作是可能的。这样的 `VarHandle` 实例可以用来向量化数组访问。

访问模式方法的参数数量、参数类型和返回类型由变量种类、变量类型和访问模式的特点决定。 `VarHandle` 创建方法（如之前所述）将记录这些要求。例如，对之前查找的 `VH_FOO_FIELD_I` 处理器进行 `compareAndSet` 操作需要 3 个参数，一个接收器实例 `Foo` 和两个 `int` ，分别表示预期值和实际值：

```
Foo f = ...
boolean r = VH_FOO_FIELD_I.compareAndSet(f, 0, 1);
```

相比之下， `getAndSet` 需要 2 个参数，一个是接收器实例 `Foo` ，另一个是要设置的值 `int` ：

```
int o = (int) VH_FOO_FIELD_I.getAndSet(f, 2);
```

访问数组元素将需要在接收器和值参数（如果有）之间添加一个额外的参数，该参数的类型为 `int` ，它对应于要操作的元素的数组索引。

为了在运行时保持可预测的行为和性能， `VarHandle` 实例应存储在静态最终字段中（如 `Atomic{Int, Long, Reference}FieldUpdater)` 的实例所必需的）。这确保了在访问模式方法调用中会发生常量折叠，例如折叠方法签名检查和/或参数类型转换检查。

> 注意：未来的 HotSpot 增强可能支持对非静态 final 字段、方法参数或局部变量中持有的 `VarHandle` 或 `MethodHandle` 实例进行常量折叠。

可以通过使用 `MethodHandles.Lookup.findVirtual` 来为 `VarHandle` 访问模式的方法生成 `MethodHandle` 。例如，为了生成针对特定变量种类和类型的"compareAndSet"访问模式的 `MethodHandle` ：

```
Foo f = ...
MethodHandle mhToVhCompareAndSet = MethodHandles.publicLookup().findVirtual(
        VarHandle.class,
        "compareAndSet",
        MethodType.methodType(boolean.class, Foo.class, int.class, int.class));
```

然后，可以使用与变量种类和类型兼容的 `VarHandle` 实例作为第一个参数来调用 `MethodHandle` ：

```
boolean r = (boolean) mhToVhCompareAndSet.invokeExact(VH_FOO_FIELD_I, f, 0, 1);
```

或可以将 `mhToVhCompareAndSet` 绑定到 `VarHandle` 实例上，然后调用：

```
MethodHandle mhToBoundVhCompareAndSet = mhToVhCompareAndSet
        .bindTo(VH_FOO_FIELD_I);
boolean r = (boolean) mhToBoundVhCompareAndSet.invokeExact(f, 0, 1);
```

使用 `MethodHandle` 的查找将执行一个 `asType` 转换来调整参数和返回值。其行为与使用 `MethodHandles.varHandleInvoker` 产生的 `MethodHandle` 等效，即 MethodHandles.invoker 的类似物：

```
MethodHandle mhToVhCompareAndSet = MethodHandles.varHandleExactInvoker(
        VarHandle.AccessMode.COMPARE_AND_SET,
        MethodType.methodType(boolean.class, Foo.class, int.class, int.class));

boolean r = (boolean) mhToVhCompareAndSet.invokeExact(VH_FOO_FIELD_I, f, 0, 1);
```

因此， `VarHandle` 可以被包装类在擦除或反射场景中使用，例如替换 `java.util.concurrent.Atomic*FieldUpdater/Atomic*Array` 类中的 `Unsafe` 用法。（尽管还需要进一步的工作，以便更新器能够访问声明类中的查找字段。）

源访问模式方法调用的源代码编译将遵循与签名多态方法调用到 `MethodHandle.invokeExact` 和 `MethodHandle.invoke` 相同的规则。需要向 Java 语言规范中添加以下内容：

1. 引用 `VarHandle` 类中的签名多态访问模式方法。
2. 允许签名多态方法返回类型除了 Object 之外，表示返回类型不是多态的（否则将通过调用点的强制转换声明）。这使得调用返回 void 的基于写入的访问方法以及调用返回 `boolean` 值的 `compareAndSet` 更加容易。

虽然这很理想，但不是必需的，源编译签名多态方法调用应增强以执行多态返回类型的目标类型化，这样就不需要显式转换。

> 注意：使用方法引用的语法查找 `MethodHandle` 或 `VarHandle` 的语法和运行时支持是理想的，但不在本 JEP 的范围内。

访问模式方法调用的运行时调用将遵循与调用 `MethodHandle.invokeExact` 和 `MethodHandle.invoke` 的签名多态方法调用类似的规则。Java 虚拟机规范需要增加以下内容：

1. 在 `VarHandle` 类中引用签名多态访问模式方法。
2. 指定 `invokevirtual` 字节码调用访问模式签名多态方法的特性。预计可以通过定义从访问模式方法调用到 `MethodHandle` 的转换来指定这种行为，然后使用 `invokeExact` 以相同的参数调用该转换（参见 `MethodHandles.Lookup.findVirtual` 的先前使用）。

重要的是，对于支持的变量种类、类型和访问模式， `VarHandle` 实现应该是可靠高效的，并满足性能目标。利用签名多态方法有助于避免装箱和数组打包。实现将包括：

* 存在于 `java.lang.invoke` 包中，其中 HotSpot 将此包中类的 final 字段视为真正的 final，从而在 `VarHandle` 本身在静态 final 字段中引用时启用常量折叠；
* 利用 JDK 内部注解 `@Stable` 进行一次性的值常量折叠，并使用 `@ForceInline` 确保方法即使达到正常内联阈值也能内联；以及；
* 使用 `sun.misc.Unsafe` 进行底层增强 volatile 访问。

需要一些 HotSpot 内联函数，其中一些如下列举：

* 一个用于 `Class.cast` 的内联函数，该函数已被添加（参见 JDK-8054492）。在此内联函数添加之前，常数折叠的 `Class.cast` 可能会留下冗余检查，这可能导致不必要的降级优化。
* 用于 `acquire-get` 访问模式的内联函数，当并发访问变量时，可以与 `set-release` 访问模式的内联函数同步（参见 `sun.misc.Unsafe.putOrdered{Int, Long, Object}` ）。
* 内置数组边界检查机制 JDK-8042997。可以添加静态方法 `java.util.Arrays` 来执行此类检查，并接受一个函数，用于返回抛出的异常或包含在抛出异常中的字符串消息，如果检查失败。这些内置机制使得使用无符号值（因为数组长度始终为正）进行更好的比较成为可能，并且可以更好地将范围检查提升到遍历数组元素的展开循环之外。

此外，HotSpot 对范围检查的进一步改进已经实现（JDK-8073480）或需要（JDK-8003585 以降低 fork/join 框架或类似 `HashMap` 或 `ConcurrentHashMap` 中的范围检查强度）。

`VarHandle` 实现应尽量减少对 `java.lang.invoke` 包内其他类的依赖，以避免增加启动时间和在静态初始化期间出现循环依赖。例如， `ConcurrentHashMap` 被此类使用，如果 `ConcurrentHashMap` 被修改为使用 `VarHandles` ，则需要确保不会引入循环依赖。使用 `ThreadLocalRandom` 及其使用 `AtomicInteger` 可能产生其他更微妙的循环。同时，也希望包含 `VarHandle` 方法调用的方法不会导致 C2 HotSpot 编译时间不当地增加。

### 示例

我们通过一个简单的 Java 示例来展示 `VarHandle` 的使用，包括：

* 普通字段访问
* 原子 `compareAndSet` 操作
* 数组元素访问

```
public class JEP193 {
    static class MyData {
        int value = 42;
    }

    public static void main(String[] args) throws Exception {
        // 创建一个实例
        MyData data = new MyData();

        // 获取 VarHandle 对象
        VarHandle valueHandle = MethodHandles.lookup()
                .in(MyData.class)
                .findVarHandle(MyData.class, "value", int.class);

        // 普通读取
        System.out.println("原始值: " + valueHandle.get(data));  // 输出 42

        // 普通写入
        valueHandle.set(data, 100);
        System.out.println("更新后: " + valueHandle.get(data)); // 输出 100

        // 原子 CAS 操作
        boolean success = valueHandle.compareAndSet(data, 100, 200);
        System.out.println("CAS 是否成功: " + success);         // 输出 true
        System.out.println("CAS 后值: " + valueHandle.get(data)); // 输出 200
    }
}
```

## \[JVM] JEP 197: 分段代码缓存

### 摘要

将代码缓存划分为不同的段，每个段包含特定类型的编译代码，以提高性能并支持未来的扩展。

### 目标

* 区分非方法、已分析和未分析代码
* 由于跳过非方法代码的专用迭代器，扫描时间更短
* 提高某些编译密集型基准测试的执行时间
* 更好地控制 JVM 内存占用
* 减少高度优化代码的碎片化
* 提高代码局部性，因为同一类型的代码很可能会 在时间上接近访问
  * 改善 iTLB 和 iCache 的行为
* 为未来扩展建立基础
  * 改进异构代码管理；例如，Sumatra（GPU 代码）和 AOT 编译代码
  * 可实现按代码堆进行细粒度锁定的可能性
  * 未来代码和元数据的分离（参见 [JDK-7072317](https://bugs.openjdk.java.net/browse/JDK-7072317)）

### 描述

而不是拥有单个代码堆，代码缓存被分割成不同的代码堆，每个代码堆包含特定类型的编译代码。这种设计使我们能够分离具有不同属性的代码。存在三种不同的顶级编译代码类型：

* JVM 内部（非方法）代码
* 分析代码
* 非配置代码

相应的代码堆如下：

* 包含非方法代码的非方法代码堆，例如编译器缓冲区和字节码解释器。此类代码将永远保留在代码缓存中。
* 包含轻量级优化、已配置且生命周期较短的配置代码堆。
* 包含完全优化、未经配置的方法的非配置代码堆，其生命周期可能较长。

非方法代码堆的大小固定为 3MB，以容纳虚拟机内部结构以及编译器缓冲区的额外空间。此额外空间根据 C1/C2 编译器线程的数量进行调整。剩余的代码缓存空间平均分配给配置和未配置的代码堆。

引入以下命令行开关以控制代码堆的大小：

* `-XX:NonProfiledCodeHeapSize`：设置包含非配置方法的代码堆的字节数。
* `-XX:ProfiledCodeHeapSize`：设置包含已分析方法的代码堆的字节数。
* `-XX:NonMethodCodeHeapSize`：设置包含非方法代码的代码堆的字节数。

代码缓存的接口和实现已适配以支持多个代码堆。由于代码缓存是 JVM 的核心组件，因此许多其他组件都会受到这些更改的影响，包括以下内容：

* 代码缓存清理器：现在仅遍历方法代码堆
* 分层编译策略：根据代码堆的空闲空间设置编译阈值
* Java 飞行记录器（JFR）：与代码缓存相关的事件
* 来自以下内容的间接引用：
  * 可用性代理：Java 对代码缓存内部的接口
  * DTrace ustack helper 脚本 (`jhelper.d`)：解析编译的 Java 方法名称
  * Pstack 支持库 (`libjvm_db.c`)：编译的 Java 方法堆栈跟踪

## \[JVM] JEP 199：智能 Java 编译，第二阶段

## \[功能] JEP 200：模块化 JDK

### 摘要

使用由 [JSR 376](http://openjdk.java.net/projects/jigsaw/spec/) 指定并由 [JEP 261](http://openjdk.java.net/jeps/261) 实现的 Java 平台模块系统来模块化 JDK。

### 动机

[Project Jigsaw](http://openjdk.java.net/projects/jigsaw/) 旨在设计和实现 Java SE 平台的标准模块系统，并将其应用于平台本身以及 JDK。其主要目标是使平台的实现更容易扩展到小型设备，提高安全性和可维护性，提升应用程序性能，并为开发者提供更好的大型编程工具。

### 描述

**设计原则**

JDK 的模块化结构遵循以下原则：

1. 由 JCP 管理的标准模块，其名称以字符串 `"java."` 开头。
2. 所有其他模块仅仅是 JDK 的一部分，并且它们的名称以字符串 `"jdk."` 开头。
3. 如果一个模块导出一个包含类型的包，该类型包含一个公共或受保护的成员，该成员反过来又引用了来自其他模块的类型，那么第一个模块必须通过 `requires transitive` 向第二个模块授予隐式可读性。 （这确保了方法调用链以明显的方式工作） 。）
4. 一个标准模块可以包含标准和非标准的 API 包。如果一个标准模块导出： 标准 API 包，则导出可能是合格的；如果是一个标准 模块导出非标准 API 包时，导出必须是 合格。在任一情况下，如果标准模块导出一个包 具备资格后，出口必须是对 JDK 中某些子集的出口 如果标准模块是 Java SE 模块，即包含在 Java SE 平台规范中，那么它不得导出任何非 SE API 包，至少在未加资格的情况下不得导出。 _即_ ，如果是一个 Java SE 模块，那么它不得导出任何非 SE API 包，至少在未加资格的情况下不得导出。
5. 标准模块可以依赖于一个或多个非标准模块。 它不得向任何非标准模块授予隐含的可读性。如果它是一个 Java SE 模块，那么它不得向任何非 SE 模块授予隐含的可读性。
6. 非标准模块不得导出任何标准 API 包。 非标准模块可以授予标准模块隐含的可读性。

原则 4 和 5 的重要后果是，仅依赖于 Java SE 模块的代码将仅依赖于标准 Java SE 类型，因此可以移植到所有 Java SE 平台实现。

**模块图**

JDK 的模块结构可以表示为一个图：每个模块是一个节点，如果第一个模块依赖于第二个模块，则从第一个模块到第二个模块存在一个有向边。完整的模块图边数太多，难以显示；以下是[_传递闭包_](https://en.wikipedia.org/wiki/Transitive_reduction) 图中省略了冗余边（点击放大）：

> <img src="https://bugs.openjdk.java.net/secure/attachment/72525/jdk.png" alt="img" data-size="original">

下面是模块图的导游：

* 标准 Java SE 模块用橙色表示；非 SE 模块用蓝色表示。
* 如果一个模块依赖于另一个模块，并且它授予该模块隐含的可读性，那么从第一个模块到第二个模块的边是实线；否则，边是虚线。
* 在最底层是 `java.base` 模块，其中包含诸如 `java.lang.Object` 和 `java.lang.String` 等基本类。基本模块不依赖于任何模块，而其他所有模块都依赖于基本模块。指向基本模块的边比其他边更浅。
* 在顶部附近是 `java.se.ee` 模块，它汇集了构成 Java SE 平台的所有模块，包括与 Java EE 平台规范重叠的模块。这是一个 _聚合_ 模块，它收集并重新导出其他模块的内容，但本身不添加任何内容。配置为包含 `java.se.ee` 模块的运行时系统将包含 Java SE 平台的 API 包。如果一个模块包含在 Java SE 平台规范中，当且仅当它是从 `java.se.ee` 可达的标准模块。 模块。
* `java.se` 聚合模块汇集了 Java SE 平台中不与 Java EE 重叠的部分。
* 非标准模块包括调试和服务性工具以及 API（例如， _例如_ ，`jdk.jdi`，`jdk.jcmd` 和 `jdk.jconsole`），开发工具（例如， _例如_ ，`jdk.compiler`，`jdk.javadoc`，以及 `jdk.xml.bind`），以及各种服务提供商（例如， `jdk.charsets`，`jdk.scripting.nashorn`，和 `jdk.crypto.ec`），这些 通过现有的方式提供给其他模块 `java.util.ServiceLoader` 机制。
* The `java.smartcardio` 模块是标准的，但不属于 Java SE 平台规范的一部分，因此其名称以字符串 `"java."` 开头，但它被标记为蓝色，并且无法从 `java.se` 模块中访问。

模块图实际上是一种新的 API 类型，并且被如此指定和 发展。模块图中的子图，其根节点为 `java.se.ee` 模块，移除了所有非 SE 模块及其对应边，在 Java SE 平台规范中进行了指定；其后续发展将由 JCP 管理。图中的其余部分的发展将由未来的 JEP 进行覆盖。在任一情况下，如果指定某个模块可供通用使用，则它将受到与其他 API 相同的进化约束。特别是，移除此类模块或以不兼容的方式更改它，至少需要提前一个主要版本发布公共通知。

所有模块的表格总结，包括 Linux/AMD64 构建的占用指标，可在 [此处 ](https://bugs.openjdk.java.net/secure/attachment/72527/module-summary.html)获取。

## \[优化] JEP 211: 在导入语句中省略弃用警告

### 摘要

自 Java SE 8 起，根据 Java 语言规范的合理解释，Java 编译器在按名称导入已弃用类型或静态导入已弃用成员（方法、字段、嵌套类型）时必须发出弃用警告。这些警告没有提供有用信息，不应强制要求。弃用成员的实际使用应保留弃用警告。

### 描述

从规范角度来看，所需更改很小。在 JLS 8 中，关于 `@Deprecated` 的部分说明：

> 一个 Java 编译器必须在类型、 方法、字段或构造函数的声明被注解为 `@弃用`在显式或隐式声明的结构中使用（重写、调用或按名称引用），除非：
>
> * 该使用位于一个自身带有注解 `@Deprecated` 的实体中；或者
> * 该使用位于一个带有注解以抑制警告的实体中 `@SuppressWarnings("deprecation")` ；或者
> * 使用和声明都位于同一个最外层类中。

规范变更可能类似于添加另一个项目，声明额外的排除项：

> * 使用情况在 `import`语句中。

在 `javac` 引用实现中，将会有一个简单的检查来跳过导入语句以查找弃用警告。

## \[优化] JEP 212: 解决 Lint 和 Doclint 警告

### 摘要

JDK 代码库包含许多由 `javac` 报告的 lint 和 doclint 错误。这些警告应该被解决，至少对于平台的基本部分。

### 描述

本 JEP 提议完成对 JDK 8 和 JDK 9 中正在进行中的警告修复工作，并将之前向 jdk9-dev 提出的源代码改进子集正式化。大多数警告通过修改方法体内部结构得到解决。解决一些原始类型警告需要更改方法签名，例如将参数类型从原始的 `java.lang.Class` 更改为 `java.lang.Class<?>` 或更具体的类型。任何 API 更改都将保持在 JDK 的一般演进策略之内。

## \[优化] JEP 213: 磨削项目币

### 摘要

包含在 JDK 7 / Java SE 7 中的 Project Coin / JSR 334 的微小语言更改易于使用，并在实践中表现良好。然而，一些修正可以解决这些更改的粗糙边缘。此外，使用下划线（ `"_"` ）作为标识符，从 Java SE 8 开始会生成警告，应将其转换为错误在 Java SE 9 中。还提议允许接口有私有方法。

### 描述

提出对 Java 编程语言进行五项小的修正：

1. 允许在私有实例方法上使用`@SafeVarargs`。 `@SafeVarargs` 注解只能应用于无法被重写的方法，包括静态方法和 final 实例方法。私有实例方法也是`@SafeVarargs`可以适应的另一个用例。
2. 允许将最终变量有效地用作 try-with-resources 语句中的资源。Java SE 7 中 try-with-resources 语句的最终版本要求为每个由语句管理的资源声明一个新的变量。这与该功能的早期迭代有所不同。JSR 334 的公开审查草案讨论了从允许由语句管理的表达式的早期草案审查版本的 try-with-resources 中改变的理由。JSR 334 专家小组支持对 try-with-resources 的进一步细化：如果资源由 final 或有效 final 变量引用，则 try-with-resources 语句可以管理资源而无需声明新变量。这种受限制的表达式被 try-with-resources 语句管理，避免了导致删除一般表达式支持的语义问题。在专家小组确定这种细化时，发布计划中时间不足，无法容纳这一变化。
3. 允许使用匿名类实现菱形泛型，如果推断类型是可表示的。因为使用匿名类构造函数的推断类型可能超出签名属性支持的类型集合，所以在 Java SE 7 中禁止使用菱形与匿名类。正如 JSR 334 建议的最终草案中提到的，如果推断类型是可表示的，则可以放宽这一限制。
4. 完成从 Java SE 8 开始进行的下划线从合法标识符名称集合中移除的工作。
5. 在 Java SE 8 中，为了添加对 Lambda 表达式的支持，曾短暂考虑将私有方法纳入接口的功能，但后来撤回，以便更好地专注于 Java SE 8 的更高优先级任务。现在提议支持私有接口方法，从而使得接口的非抽象方法能够共享代码。

在 Java 语言变化的范围内，这些改进都是非常小的改动。 `@SafeVarags` 的变化可能只涉及对规范中的一两句话进行修改， `javac` 的改动规模也类似。然而，就像任何 Java 语言变化一样，必须小心处理需要更新的平台所有部分。

### 示例

1. **允许菱形操作符 `< >` 用于匿名类中**

之前 Java 不允许这样写：

```
Map<String, List<String>> map = new HashMap<>() {
    // 匿名类体
};
```

JEP 213 后，这是合法的。

2. **增强 try-with-resources 的使用**

Java 7 的语法：

```
try (BufferedReader br = new BufferedReader(new FileReader(path))) {
    ...
}
```

JEP 213 允许你复用已经声明过的 `AutoCloseable` 变量（只要它是 final 或 effectively final）：

```
BufferedReader br = new BufferedReader(new FileReader(path));
try (br) {
    // br 自动关闭
}
```

3. **简化 lambda 中的 `@SafeVarargs` 使用限制**

Java 之前不允许你在私有方法或 lambda 表达式中用 `@SafeVarargs`，现在允许。

4. **允许接口中的私有方法（Java 9 本身功能）**

虽然 JEP 213 并未引入该功能，但和它一脉相承。Java 9 开始允许接口中定义 `private` 方法用于重构 default/static 方法中的公共逻辑。

## \[JVM] JEP 214: 移除 JDK 8 中已弃用的 GC 组合

### 摘要

通过 JEP 173 移除 JDK 8 中已弃用的 GC 组合。

### 描述

JEP 173 中列出的已弃用的 GC 组合的控制标志，以及启用 CMS 前台收集器（作为 JDK-8027876 的一部分已弃用）的标志将从代码库中移除。这意味着将不再打印有关它们的警告消息；如果使用这些标志，JVM 将无法启动。

一旦移除这些标志，现在已死亡的任何代码都将从 GC 代码库中移除。由于这项工作，代码库中可能存在一些可以进行的简化，但这些简化的范围可能很大。这些简化可能作为单独的更改分离出来。

这里是关于即将停止工作的标志和标志组合的详细总结：

```
DefNew + CMS       : -XX:-UseParNewGC -XX:+UseConcMarkSweepGC
ParNew + SerialOld : -XX:+UseParNewGC
ParNew + iCMS      : -Xincgc
ParNew + iCMS      : -XX:+CMSIncrementalMode -XX:+UseConcMarkSweepGC
DefNew + iCMS      : -XX:+CMSIncrementalMode -XX:+UseConcMarkSweepGC -XX:-UseParNewGC
CMS foreground     : -XX:+UseCMSCompactAtFullCollection
CMS foreground     : -XX:+CMSFullGCsBeforeCompaction
CMS foreground     : -XX:+UseCMSCollectionPassing
```

对于 ParNew + SerialOld 组合，此 JEP 的工作也将包括性能测试，比较 ParNew + SerialOld 与 ParallelScavenge + SerialOld。这应该会导致从 ParNew + SerialOld 迁移到 ParallelScavenge + SerialOld 的调整建议。

## \[JVM] JEP 215: javac 的分层归因

## \[JVM] JEP 216: 正确处理导入语句

### 摘要

修复 `javac` 以正确接受和拒绝程序，无论 `import` 语句和 `extends` 以及 `implements` 子句的顺序如何。

### 描述

`javac` 在编译类时使用几个阶段。考虑到 `导入` 处理，两个重要的阶段是：

* 类型解析，它遍历提供的 AST，寻找类和接口声明，
* 成员解析，包括：
  * (**1a**) 如果 `T` 是顶级，则源文件中定义的 `import` 语句被处理和导入的成员添加到 `T` 的作用域中 `T` 的导入成员在 `T` 的作用域内被处理和添加
  * (**1b**) 如果 `T` 是嵌套的，则直接封装类的解析 `T`（如果有）
  * (**2**) `extends`/`implements` 子句的 `T` 进行类型检查
  * (**3**) `T` 的类型变量进行类型检查

上述阶段是 `javac` 对类进行 _解析_ 过程的一部分，这包括确定类的超类型、类型变量和成员。

要看到这个过程在实际中的效果，请考虑以下代码：

```
package P;

import static P.Outer.Nested.*;
import P.Q.*;

public class Outer {
    public static class Nested implements I {
    }
}

package P.Q;
public interface I {
}
```

在类型解析阶段，可以识别出存在以下类型 `P.Outer`、`P.Outer.Nested` 和 `P.Q.I`。然后，如果要对 `P.Outer` 类进行分析，成员解析阶段的工作方式如下：

| 1. | `P.Outer` 的解析开始                                                                  |
| -- | -------------------------------------------------------------------------------- |
| 2. | 根据 1a，处理 `import static P.Outer.Nested.*;` 开始，这意味着查找 `P.Outer.Nested` 及其转置超类的成员。 |
| 3. | 开始解析 `P.Outer.Nested` 类（静态导入也可以导入继承的类型）。                                         |
| 4. | 触发解析 `P.Outer`，由于它已经在进行中，因此被跳过。                                                  |
| 5. | 运行类型检查 `I`（`implements` 子句），但由于它尚未在作用域内，因此无法解析 `I`。                              |
| 6. | `import P.Q.*` 的解析开始，它将 `P.Q` 的所有成员类型（包括接口 `I`）导入到当前文件的范围内                       |
| 7. | `P.Outer` 和其他类的解析继续                                                              |

如果导入顺序被交换，则步骤 6 会在步骤 5 之前发生，因此 `I` 在步骤5中找到。

上述问题并非与 `import` 处理相关问题的唯一。另一个已知问题是，一个类的类型参数的界限可以合法地引用其声明类可能的内部类。在某些情况下，这目前会导致无法解决的循环，例如：

```
package P;

import static P.Outer.Nested.*;

public class Outer {
    public static class Nested<T extends I> {
        static class I { }
    }
}
```

针对这个问题，设想出的解决方案是将现有的 `javac` 成员解析的第一阶段分为三个阶段：第一个阶段将分析 包含文件导入，第二个阶段将仅构建 类/接口层次结构，不包含任何类型参数、注解、 _等等。_，第三个将正确分析类头，包括类型参数。

预期这个更改将允许 `javac` 接受目前被拒绝的程序，但不会拒绝目前被接受的程序。

## \[JVM] JEP 217: 注解管道 2.0

### 摘要

重新设计 `javac` 注解管道，以更好地满足注解和注解处理工具的需求。

### 描述

重构 `javac` 注解管道。这不应该对外部 产生明显影响，除非我们在修复错误和改进正确性方面 进行。第一步是提高测试覆盖率，以便我们可以衡量和评估 我们的退出标准。之后将进行一系列增量 重构。这项工作将在 OpenJDK 中完成 [注解管道 2.0 项目 ](http://openjdk.java.net/projects/anno-pipeline/)。

关于类型注解，`JavaDoc` 工具存在一些相关问题。然而，`JavaDoc` 作为 [JavaDoc.Next 项目 ](http://openjdk.java.net/projects/javadoc-next/)的一部分，正在进行重大改进。这项工作的一部分包括将 `JavaDoc` 转换为使用 `javax.lang.model` API 而不是较旧的 `com.sun.javadoc` API。因此，本项目的工作目标不是对 `javadoc` 进行操作 确保注释，包括类型注释，得到呈现 正确。预计作为 JavaDoc.Next 项目的一部分， `javadoc`将得到增强，以利用对 Java 文档注释的更新 本项目目标的是 `javax.lang.model` API。

## \[功能] JEP 219: 数据报传输层安全（DTLS）

### 摘要

定义数据报传输层安全（DTLS）版本 1.0（RFC 4347）和 1.2（RFC 6347）的 API。

### 动机

支持 DTLS 对于满足越来越多的数据报兼容应用程序的安全传输需求至关重要。RFC 4347 列出了一些为什么 TLS 对于这些类型的应用程序来说不够充分的原因：

* "TLS 是目前最广泛部署的用于保护网络流量的协议。...然而，TLS 必须在可靠的传输通道上运行——通常是 TCP。因此，它不能用于保护不可靠的数据报文流量。"
* "...越来越多的应用层协议被设计出来，使用 UDP 作为传输。特别是，会话初始化协议（SIP）和电子游戏协议越来越受欢迎。"
* "在许多情况下，保护客户端/服务器应用程序的最佳方式是使用 TLS；然而，对数据报语义的要求自动禁止了 TLS 的使用。因此，一个与数据报兼容的 TLS 变体非常受欢迎。"

支持 DTLS 的协议包括但不限于：

* RFC 5238，基于数据报拥塞控制协议（DCCP）的数据报传输层安全（DTLS）
* RFC 6083，基于流控制传输协议（SCTP）的数据报传输层安全（DTLS）
* RFC 5764，用于建立安全实时传输协议（SRTP）密钥的数据报传输层安全（DTLS）扩展
* RFC 7252，受限应用协议（CoAP）

Google Chrome 和 Firefox 现在支持 DTLS-SRTP 用于 Web 实时通信（WebRTC）。主要的 TLS 提供商和实现，包括 OpenSSL、GnuTLS 和 Microsoft SChannel，都支持 DTLS 版本 1.0 和 1.2。

### 描述

我们预计 DTLS API 和实现将相对较小。新的 API 应该是传输无关的，类似于 `javax.net.ssl.SSLEngine` 。随着工作的进展，将在此处添加有关 API 的更多详细信息。一些初步的设计考虑因素如下：

1. DTLS API 和实现将不会管理读取超时。确定适当的超时值以及何时以及如何触发超时事件将是应用程序的责任。
2. 可能会添加一个新的 API 来设置最大应用程序数据报大小（PMTU 减去 DTLS 每个记录的额外开销）。如果没有明确指定大小，则 DTLS 实现应自动调整大小。如果某个片段丢失两次或三次，实现可能会减小最大应用程序数据报大小，直到它足够小。
3. DTLS 实现应最多消耗或产生一个 TLS 记录，以便每个解包或打包操作，这样记录就可以在数据报层单独交付，或者在交付顺序出错时更容易重新组装。
4. 如果需要，应用程序负责相应地组装顺序错乱的应用数据。DTLS API 应提供对每个 DTLS 消息中应用程序数据的访问。

### 示例

JEP 219（[https://openjdk.org/jeps/219](https://openjdk.org/jeps/219)）引入了对 **DTLS（Datagram Transport Layer Security）1.0** 的支持，这是在 Java 9 中添加到 `JSSE`（Java Secure Socket Extension）中的。

> 📌 **DTLS = 基于 UDP 的 TLS** 适合低延迟、丢包容忍的通信场景，比如语音、视频、在线游戏等。它不像 TLS 一样基于 TCP，而是基于 UDP。

**✅ 实现机制简述**

JDK 在 `javax.net.ssl.SSLEngine` 上做了扩展，新增了一个类：

```
com.sun.net.ssl.internal.ssl.DTLSSocket
```

不过真正核心的用法是：**通过 `SSLEngine` 来实现 DTLS 协议解析和处理逻辑**，然后配合 UDP Socket 手动读写数据。

JSSE 并没有提供直接的 `DTLSSocket` 类供你像 `SSLSocket` 那样开箱即用，而是让你通过 `SSLEngine` 自己控制。

**✅ 示例：DTLS 客户端和服务端通信**

注意：JDK 自带的示例很底层，需要你手动处理 `SSLEngine.wrap` / `unwrap` + UDP 收发，这里我们做一个简化演示（仅作教学用途，不完整处理握手细节和超时等）。

🔧 **1. 服务端代码（DatagramSocket + SSLEngine）**

```
import javax.net.ssl.*;
import java.net.*;
import java.nio.*;
import java.security.KeyStore;

public class DTLSServer {
    public static void main(String[] args) throws Exception {
        int port = 4444;
        DatagramSocket socket = new DatagramSocket(port);

        // 加载密钥库
        KeyStore ks = KeyStore.getInstance("JKS");
        ks.load(DTLSServer.class.getResourceAsStream("/server_keystore.jks"), "password".toCharArray());

        KeyManagerFactory kmf = KeyManagerFactory.getInstance("SunX509");
        kmf.init(ks, "password".toCharArray());

        SSLContext context = SSLContext.getInstance("DTLS");
        context.init(kmf.getKeyManagers(), null, null);

        SSLEngine engine = context.createSSLEngine();
        engine.setUseClientMode(false);
        engine.beginHandshake();

        System.out.println("DTLS server listening on port " + port);
        
        // 简化处理：只接收并打印一条信息
        byte[] buf = new byte[2048];
        DatagramPacket packet = new DatagramPacket(buf, buf.length);
        socket.receive(packet);

        System.out.println("Received raw UDP packet (not yet decrypted)");
        // 真实使用中需要调用 engine.unwrap() 来处理 DTLS 解密和握手
    }
}
```

🔧 **2. 客户端代码（UDP 发送）**

```
import java.net.*;

public class DTLSClient {
    public static void main(String[] args) throws Exception {
        DatagramSocket socket = new DatagramSocket();
        InetAddress address = InetAddress.getByName("localhost");

        String message = "Hello DTLS Server!";
        byte[] buf = message.getBytes();

        DatagramPacket packet = new DatagramPacket(buf, buf.length, address, 4444);
        socket.send(packet);

        System.out.println("Sent UDP packet (not encrypted)");
    }
}
```

**🧠 小结**

| 项目   | 说明                                        |
| ---- | ----------------------------------------- |
| 实现方式 | 基于 `SSLEngine` + `DatagramSocket`         |
| 安全协议 | DTLS 1.0（Java 9），DTLS 1.2（从 Java 13 开始支持） |
| 应用场景 | 音视频通话、在线游戏、IoT 等                          |
| 使用难点 | 手动处理握手、数据包重发、超时逻辑                         |

## \[功能] JEP 220: 模块化运行时镜像

### 摘要

对 JDK 和 JRE 运行时镜像进行重构，以适应模块化并提高性能、安全性和可维护性。定义一个新的 URI 方案来命名存储在运行时镜像中的模块、类和资源，而不泄露镜像的内部结构或格式。根据需要修订现有规范以适应这些更改。

### 描述

**当前运行时图像结构**

JDK 构建系统目前生成两种类型的运行时图像：Java 运行时环境（JRE），它是一个完整的 Java SE 平台实现，以及 Java 开发工具包（JDK），它包含 JRE 和开发工具和库。（三个紧凑配置文件构建是 JRE 的子集。）

JRE 图像的根目录包含两个目录， `bin` 和 `lib` ，内容如下：

* `bin` 目录包含必要的可执行二进制文件，特别是用于启动运行时系统的 `java` 命令。（在 Windows 操作系统上，它还包含运行时系统的动态链接本地库。）
* “ `lib` ”目录包含各种文件和子目录：
  * 各种“ `.properties` ”和“ `.policy` ”文件，其中大部分可能（虽然很少）会被开发者、部署人员和最终用户编辑；
  * 默认情况下不存在的“ `endorsed` ”目录，可以在此目录中放置包含受支持标准和独立技术实现的 JAR 文件；
  * 可以在此目录中放置包含扩展或可选包的 JAR 文件的“ `ext` ”目录；
  * 各种实现内部数据文件，例如字体、颜色配置文件和时区数据等二进制格式文件；
  * 包括 `rt.jar` 在内的各种 JAR 文件，其中包含运行时系统的 Java 类和资源文件。
  * Linux、macOS 和 Solaris 操作系统上运行时系统的动态链接本地库。

JDK 镜像包含 JRE 的副本，位于其 `jre` 子目录中，并包含其他子目录：

* “ `bin` 目录包含命令行开发调试工具，例如 `javac` ， `javadoc` ，和 `jconsole` ，以及 `jre/bin` 目录中二进制文件的副本，以便于使用；”
* “ `demo` 和 `sample` 目录分别包含演示程序和示例代码；”
* “ `man` 目录包含 UNIX 风格的手册页；”
* “ `include` 目录包含用于编译与运行时系统直接交互的本地代码的 C/C++ 头文件；”
* `lib` 目录包含各种 JAR 文件和其他类型的文件，这些文件构成了 JDK 工具的实现，其中包括 `tools.jar` ，它包含了 `javac` 编译器的类。

JDK 图像的根目录或未嵌入 JDK 图像的 JRE 图像的根目录也包含各种 `COPYRIGHT` ， `LICENSE` 和 `README` 文件，以及一个描述图像的 `release` 文件，该文件以简单的键/值属性对的形式描述图像，例如，

```
JAVA_VERSION="1.9.0"
OS_NAME="Linux"
OS_VERSION="2.6"
OS_ARCH="amd64"
```

**新的运行时图像结构**

目前 JRE 和 JDK 图像之间的区别纯粹是历史性的，这是 JDK 1.2 版本开发后期做出的一个实现决策的结果，并且从未重新审视过。新的图像结构消除了这种区别：JDK 图像只是一个包含历史上在 JDK 中找到的完整开发工具和其他项目的运行时图像。

模块化运行时图像包含以下目录：

* `bin` 目录包含由链接到图像的模块定义的任何命令行启动器。（在 Windows 上，它继续包含运行时系统的动态链接本地库。）
* `conf` 目录包含由开发者、部署人员和最终用户编辑的 `.properties` 、 `.policy` 以及其他类型的文件，这些文件以前位于 `lib` 目录或其子目录中。
* 在 Linux、macOS 和 Solaris 上， `lib` 目录包含运行时系统的动态链接本地库，就像今天一样。这些文件名为 `libjvm.so` 或 `libjvm.dylib` ，可能被嵌入运行时系统的程序链接。此目录中的一些其他文件也供外部使用，包括 `src.zip` 和 `jexec` 。
* 所有其他位于 `lib` 目录下的文件和目录都必须被视为运行时系统的私有实现细节。它们不打算供外部使用，其名称、格式和内容可能会在通知的情况下进行更改。
* `legal` 目录包含链接到图像的模块的法律声明，每个模块一个子目录。
* 一个完整的 JDK 图像除了今天所包含的 `demo` 、 `man` 和 `include` 目录外，还包含这些目录。（ `samples` 目录已被 JEP 298 移除。）

运行时图像的根目录还包含由构建系统生成的 `release` 文件。为了便于识别运行时图像中包含的模块， `release` 文件包含一个新属性 `MODULES` ，它是一个由空格分隔的模块名称列表。列表根据模块的依赖关系进行拓扑排序，因此 `java.base` 模块始终排在第一位。

### 说明

文件结构对比：

Java 8：

```
JAVA_HOME/
  |- lib/
      |- rt.jar
      |- tools.jar
```

Java 9+：

```
JAVA_HOME/
  |- lib/
      |- modules       ← 新结构（模块镜像）
  |- jmods/
      |- java.base.jmod
      |- java.sql.jmod
```

## \[功能] JEP 221：新的 Doclet API

### 摘要

提供一个替代的 [Doclet API](http://docs.oracle.com/javase/8/docs/jdk/api/javadoc/doclet/index.html)，以利用适当的 Java SE 和 JDK API，并更新标准 doclet 以使用新 API

### 描述

新的 Doclet API 在 jdk.javadoc.doclet 包中声明。它使用语言模型 API 和编译树 API。

javadoc 工具更新为识别针对新 Doclet API 编写的 doclet。为了过渡目的，将支持旧 Doclet API，但将冻结，即不会更新以支持在过渡期间引入的任何新语言功能。

现有的标准 doclet 支持一个名为 [Taglet API](http://docs.oracle.com/javase/8/docs/technotes/guides/javadoc/taglet/overview.html). Taglets 允许用户定义自定义标签，这些标签可用于文档注释，并指定这些标签在生成的文档中的显示方式。更新的标准 doclet 支持更新的 taglet API。

### 说明

JEP 221（[https://openjdk.org/jeps/221](https://openjdk.org/jeps/221)）引入了 **全新的标准 Doclet API（JavaDoc 工具插件系统）**，这是 Java 9 的一个重大变化，目标是：

> 📌 **替换过时、难用、内部依赖严重的旧 Doclet API**，并让 `javadoc` 工具生成更结构化、模块化的 HTML 文档。

***

**✅ 新旧 Doclet 的主要区别**

| 特性        | 旧 Doclet (`com.sun.javadoc.*`) | 新 Doclet (`jdk.javadoc.doclet.*`)                   |
| --------- | ------------------------------ | --------------------------------------------------- |
| API 包名    | `com.sun.javadoc`（非标准 API）     | `jdk.javadoc.doclet`（标准模块）                          |
| 是否标准化     | ❌ 非标准、非模块化                     | ✅ 正式模块，Java 9+ 标准部分                                 |
| 文档结构      | 操作 class/interface 的模型，结构较松散   | 使用 `Element` / `DocTree`，结构清晰，基于 `javax.lang.model` |
| HTML 生成能力 | 手动拼接 HTML                      | 提供标准 Writer，可扩展自定义                                  |
| 模块支持      | ❌ 不支持模块系统                      | ✅ 可识别 module-info.java，支持 Jigsaw                    |
| 可维护性      | 底层实现复杂，依赖内部工具                  | 更清晰，易于自定义与维护                                        |

***

**✨ 新 Doclet 带来的亮点变化**

**1. 标准模块支持**

新 API 是 `jdk.javadoc` 模块的一部分，不再依赖 `tools.jar`，支持模块化构建。

**2. 模型清晰**

* 使用 `javax.lang.model.element.Element` 表示类、方法、字段等语法元素
* 使用 `com.sun.source.doctree.*` 表示注释结构（包括 `@param`, `@return` 等）

**3. 更强扩展能力**

支持你自己写一个 `MyDoclet`，通过 `javadoc -doclet` 使用，生成 Markdown、PDF、JSON 甚至数据库内容等。

**✅ 示例：一个最简单的新 Doclet**

```
import jdk.javadoc.doclet.Doclet;
import jdk.javadoc.doclet.StandardDoclet;
import javax.lang.model.element.Element;
import javax.lang.model.element.TypeElement;
import javax.tools.Diagnostic.Kind;
import java.util.Set;
import java.util.Locale;
import java.util.List;

public class MyDoclet extends StandardDoclet {
    @Override
    public boolean run(DocletEnvironment env) {
        for (Element e : env.getIncludedElements()) {
            if (e instanceof TypeElement) {
                System.out.println("文档类: " + ((TypeElement) e).getQualifiedName());
            }
        }
        return true;
    }
}
```

编译并运行：

```
javadoc -doclet MyDoclet -docletpath ./out -sourcepath ./src MyClass
```

**🚧 旧 Doclet 使用的限制（已废弃）**

旧接口如 `ClassDoc`, `MethodDoc` 都属于 `tools.jar`，从 Java 9 起：

* `tools.jar` 不再存在
* `com.sun.javadoc.*` 被标记为**过时**
* Java 11+ 中使用会直接报错 ❌

**✅ 总结：新 Doclet 的优势一图胜负**

| 维度   | 新 Doclet                            |
| ---- | ----------------------------------- |
| 模块化  | ✅ 完整模块 `jdk.javadoc`                |
| 可扩展性 | ✅ 适配 `module-info.java`             |
| 标准化  | ✅ 属于 JDK 标准模块                       |
| 清晰度  | ✅ 基于 `javax.lang.model` 和 `DocTree` |
| 未来维护 | ✅ 推荐使用                              |
| 向后兼容 | ❌ 不兼容旧 Doclet，需要重写                  |

如果你想用 JavaDoc 做 Markdown 输出、分析 API 文档结构、生成接口文档（比如 swagger 风格），新 Doclet 就是更好的选择。如果你还在维护旧的 Doclet 插件，建议尽早迁移。

## \[功能] JEP 222: jshell: Java Shell

### 摘要

提供一个交互式工具，用于评估 Java 编程语言的声明、语句和表达式，并提供一个 API，以便其他应用程序可以利用此功能。

### 描述

**功能性**

JShell API 将提供 JShell 的所有评估功能。输入到 API 的代码片段被称为 "片段"。`jshell` 工具也将使用 JShell 完成 API 来确定何时输入不完整（此时需要提示用户输入更多），何时如果添加分号就会完整（在这种情况下，工具将自动添加分号），以及当使用制表符请求完成时如何完成输入。该工具将有一组用于查询、保存和恢复工作以及配置的命令。命令将通过前面的斜杠与片段区分开来。

**文档**

JShell 模块 API 规范可在此处找到：

* [http://download.java.net/java/jdk9/docs/api/jdk.jshell-summary.html](http://download.java.net/java/jdk9/docs/api/jdk.jshell-summary.html)

包括主要的 JShell API（`jdk.jshell` 包）规范：

* [http://download.java.net/java/jdk9/docs/api/jdk/jshell/package-summary.html](http://download.java.net/java/jdk9/docs/api/jdk/jshell/package-summary.html)

`jshell` 工具参考：

* [https://docs.oracle.com/javase/9/tools/jshell.htm](https://docs.oracle.com/javase/9/tools/jshell.htm)

Java 平台，标准版工具参考的一部分：

* [https://docs.oracle.com/javase/9/tools/tools-and-command-reference.htm](https://docs.oracle.com/javase/9/tools/tools-and-command-reference.htm)

**术语**

在本文档中，“类”一词的含义是指 Java 虚拟机规范（JVMS）中使用的含义，包括 Java 语言规范（JLS）中的类、接口、枚举和注解类型。如果表示不同的含义，文本将予以明确。

**程序片段**

代码片段必须对应以下 JLS 语法生成式之一：

* _表达式_
* _语句_
* _类声明_
* _接口声明_
* _方法声明_
* _字段声明_
* _导入声明_

在 JShell 中，"变量"是一个存储位置，并具有关联的类型。变量可以通过一&#x4E2A;_&#x5B57;段声&#x660E;_&#x7247;段显式创建：

```
int a = 42;
```

或者通过表达式隐式创建（见下文）。变量具有一些字段语义/语法（例如，允许使用 `volatile` 修饰符）。然而，变量没有用户可见的类封装它们，通常会被视为和使用本地变量一样。

所有表达式都被接受为片段。这包括没有副作用的表达式，如常量、变量访问和 lambda 表达式：

```
1
a
2+2
Math.PI
x -> x+1
(String s) -> s.length()
```

以及具有副作用的表达式，如赋值和方法调用：

```
a = 1
System.out.println("Hello world");
new BufferedReader(new InputStreamReader(System.in))
```

一些表达式片段隐式创建一个变量来存储表达式的值，以便稍后由其他片段引用。默认情况下，隐式创建的变量名为 `$`_X_，其中 _X_ 是片段标识符。如果表达式是 void（例如 `println`），或者表达式的值已经可以通过简单名称引用（如上面的 'a' 和 'a=1'），则不会隐式创建变量。

所有语句都被接受为片段，除了 'break'、'continue' 和 'return'。但是，片段可以包含 'break'、'continue' 或 'return' 语句，只要它们符合 Java 编程语言的常规封装上下文规则。例如，此片段中的返回语句是有效的，因为它被 lambda 表达式包围。

```
() -> { return 42; }
```

声明片段（ _类声明_ 、 _接口声明_ 、 _方法声&#x660E;_&#x6216;_字段声明_ )是一个显式引入名称的片段，该名称可以被其他片段引用。声明片段受以下规则约束：

* 访问修饰符（`public`、`protected` 和 `private`）被忽略（所有声明片段对所有其他片段都是可访问的）
* 修饰符 `final` 被忽略（允许未来的更改/继承）
* 修饰符 `static` 被忽略（没有用户可见的包含类）
* 默认 `default` 和 `synchronized` 修饰符不允许使用
* 抽象 `abstract` 修饰符仅允许在类中使用

除了形式为 _ImportDeclaration_ 的片段外，所有片段都可以包含嵌套声明。例如，一个表示类实例创建表达式的片段可以指定带有嵌套方法声明的匿名类体。对于嵌套声明的修饰符，应遵循 Java 编程语言的常规规则，而不是上述规则。例如，下面的类片段是可接受的，并且对嵌套方法声明上的私有修饰符表示尊重，因此片段"`new C().secret()`"是不可接受的：

```
class C {
  int answer() { return 2 * secret(); }
  private int secret() { return 21; }
}
```

片段不得声明包或模块。所有 JShell 代码都放置在未命名的模块中的单个包中。包的名称由 JShell 控制。

在 `jshell` 工具中，如果分号是输入（除去空白和注释）的最后一个字符，则可以省略该分号。

**状态**

JShell 的状态保存在一个 `JShell` 实例中。一个代码片段在 `JShell` 中通过 `eval(...)` 方法进行评估，可能会产生错误、声明代码或执行语句或表达式。对于具有初始化器的变量，声明和执行都会发生。一个 `JShell` 实例包含之前定义和修改的变量、方法、类，之前定义的导入声明，之前输入的语句和表达式的副作用（包括变量初始化器），以及外部代码库。

**修改**

由于期望用途是探索，因此声明（变量、方法和类）必须能够在时间上演变，同时保留评估数据。一个选择是在某些或所有情况下将更改的声明作为一个新的附加实体，但这肯定会导致混淆，并且不利于探索声明之间的交互。在 JShell 中，每个唯一的声明键在任何给定时间都只有一个声明。对于变量和类，唯一的声明键是名称，而对于方法，唯一的声明键是名称和参数类型（以允许重载）。由于这是 Java，变量、方法和类都有自己的命名空间。

**前向引用**

在 Java 编程语言中，在类的主体内部，可以出现对后来出现的成员的引用；这是前向引用。由于 JShell 中代码是按顺序输入和评估的，这些引用将暂时未解析。在某些情况下，例如相互递归，需要前向引用。在探索性编程中输入代码时也可能发生这种情况，例如，意识到应该调用另一个（尚未编写的）方法。JShell 支持在方法体、返回类型和参数类型、变量类型以及类内部的前向引用。由于语义要求它们必须立即执行，因此不支持变量初始化器中的前向引用。

**代码片段依赖**

代码状态保持最新和一致；也就是说，当评估代码片段时，对依赖片段的任何更改都会立即传播。

当片段成功声明时，声明将分为三种类型：添加、修改或替换。如果片段是首次使用该键进行声明，则该片段为添加。如果片段的键与之前的片段匹配，但它们的签名不同，则该片段为替换。如果片段的键与之前的片段匹配且它们的签名也匹配，则该片段为修改；在这种情况下，不会影响任何依赖片段。在修改和替换的情况下，之前的片段不再是代码状态的一部分。

当片段&#x88AB;_&#x6DFB;&#x52A0;_&#x65F6;，它可能提供了一个未解决的引用。当片段&#x88AB;_&#x66FF;&#x6362;_&#x65F6;，它可能更新了一个现有的片段。例如，如果方法的返回类型被声明为类 `C`，然后类 `C` &#x88AB;_&#x66FF;换_ ，那么方法的签名已更改，该方法必须&#x88AB;_&#x66FF;换_ 。注意：这可能导致之前有效的类或方法变得无效。

用户希望尽可能使用户数据持久化。这除了在变&#x91CF;_&#x66FF;&#x6362;_&#x7684;情况下才能实现。当一个变量被替换时，无论是用户直接替换还是通过依赖更新间接替换，该变量将被设置为默认值（`null`，因为这种情况只能发生在引用变量上）。

当声明无效时，无论是由于向前引用还是通过更新而变得无效，该声明将被“圈养”。圈养的声明可以在其他声明和代码中使用，但是，如果尝试执行它，将发生运行时异常，该异常将解释未解决的引用或其他问题。

**包装**

在 Java 编程语言中，变量、方法、语句和表达式必须嵌套在其他构造中，最终是类。当 JShell 的实现将变量、方法、语句和表达式片段编译为 Java 代码时，需要一个人工上下文，如下所示：

* 变量、方法、类
  * 作为合成类的静态成员
* 表达式和语句
  * 作为合成类中合成静态方法内的表达式和语句

此包装还支持代码片段更新，因此请注意，代码片段类也被包装在合成类中。

**模块化环境配置**

`jshell` 工具有以下选项用于控制模块化环境：

* `--module-path`
* `--add-modules`
* `--add-exports`

模块化环境也可以通过直接添加到编译器和运行时选项进行配置。编译器标志可以通过 `-C` 选项添加。运行时标志可以通过 `-R` 选项添加。

所有 `jshell` 工具选项均在工具参考中进行了文档说明（见上文）。

模块化环境可以在 API 级别通过 `compilerOptions` 和 `remoteVMOptions` 方法进行配置。 `JShell.Builder`。

JShell 的无名模块读取的模块集与无名模块的默认根模块集相同，这是由 JEP 261 "根模块"所确定的。

* [http://openjdk.java.net/jeps/261](http://openjdk.java.net/jeps/261)

### 命名

* 模块
  * `jdk.jshell`
* 工具启动器
  * `jshell`
* API 包
  * `jdk.jshell`
* SPI 包
  * `jdk.jshell.spi`
* 执行引擎 "库" 包
  * `jdk.jshell.execution`
* 工具启动 API 包
  * `jdk.jshell.tool`
* 工具实现包
  * `jdk.internal.jshell.tool`
* OpenJDK 项目
  * Kulla

### 说明

JEP 222 引入的 [`jshell`](https://openjdk.org/jeps/222) 是 Java 9 中新增的官方 **REPL 工具**（Read-Eval-Print Loop），它让 Java 有了类似 Python、Node.js 那样的交互式命令行工具。

**🎯 JShell 是什么？**

JShell 是一个交互式 Java 命令行工具，允许你像写脚本那样，**一行一行地编写、运行、测试 Java 代码**，而不需要创建完整的类、方法、main 函数。

**✅ 基本使用方法**

打开命令行，直接输入：

```
jshell
```

你会看到类似：

```
|  Welcome to JShell -- Version 21
|  For an introduction type: /help intro
jshell>
```

现在你就进入了 Java 的 REPL 模式，开始实验代码吧！

**🧪 示例演示（一步步来）**

**1. 直接计算表达式**

```
jshell> 1 + 2
$1 ==> 3
```

系统会自动保存结果为 `$1`，可以复用：

```
jshell> $1 * 10
$2 ==> 30
```

**2. 定义变量**

```
jshell> int x = 42;
x ==> 42

jshell> x + 8
$3 ==> 50
```

**3. 定义方法**

```
jshell> int square(int n) {
   ...> return n * n;
   ...> }
|  created method square(int)

jshell> square(5)
$4 ==> 25
```

**4. 定义类**

```
jshell> class Person {
   ...> String name;
   ...> Person(String name) { this.name = name; }
   ...> String greet() { return "Hi " + name; }
   ...> }

jshell> new Person("Alice").greet()
$5 ==> "Hi Alice"
```

**5. 查看当前定义的内容**

```
jshell> /vars     // 查看变量
jshell> /methods  // 查看方法
jshell> /list     // 查看历史代码
```

**6. 保存和加载脚本**

保存当前 session：

```
/jshell> /save myscript.jsh
```

重新加载脚本：

```
jshell> /open myscript.jsh
```

**7. 退出**

```
jshell> /exit
```

**✅ jshell 适合哪些场景？**

| 场景                                           | 是否适合          |
| -------------------------------------------- | ------------- |
| 快速测试一段代码                                     | ✅             |
| 学习 Java 基础语法                                 | ✅             |
| 尝试 Java 新特性（如 Java 21 的 Record、Switch 模式匹配等） | ✅             |
| 写大型项目                                        | ❌（更适合在 IDE 中） |

**🚀 提示：JShell 还能这样用！**

```
jshell myscript.jsh      # 运行脚本文件
jshell --startup DEFAULT # 自定义启动命令
```

你也可以通过 JShell 集成在 IDE 里（IntelliJ、VSCode）作为实验台。

## \[功能] JEP 223: 新的版本字符串方案

### 摘要

定义一个版本字符串方案，该方案可以轻松区分主要版本、次要版本和安全更新版本，并将其应用于 JDK。

### 描述

**版本号**

版本号， `$VNUM` ，是由非空元素组成的序列，元素之间由点字符（U+002E）分隔。元素可以是零，或者没有前导零的无符号整数。版本号的最后一个元素不能为零。格式如下：

```
[1-9][0-9]*((\.0)*\.[1-9][0-9]*)*
```

序列长度可以是任意的，但前三个元素具有特定的含义，具体如下：

```
$MAJOR.$MINOR.$SECURITY
```

* `$MAJOR` --- 主版本号，当包含重大新功能的重大版本发布时增加，例如，Java SE 平台规范的新版本中指定的 JSR 337 用于 Java SE 8。在提前至少一个主要版本通知的情况下，可以在重大版本中删除功能，并在有正当理由时进行不兼容的更改。JDK 8 的 `$MAJOR` 版本号是 `8` ，JDK 9 的 `$MAJOR` 版本号是 `9` 。当 `$MAJOR` 增加时，所有后续元素都将被删除。
* `$MINOR` --- 次要版本号，用于表示可能包含兼容性错误修复、根据相关平台规范维护发布所要求的标准 API 修订以及该规范范围之外的实现功能（如新的 JDK 特定 API、额外的服务提供商、新的垃圾收集器和移植到新的硬件架构）的次要更新发布。
* `$SECURITY` --- 安全级别，用于表示包含关键修复的安全更新发布，包括提高安全性的必要修复。 `$SECURITY` 在 `$MINOR` 增加时不重置为零。因此，对于给定的 `$MAJOR` 值， `$SECURITY` 的值越高，表示的发布越安全，无论 `$MINOR` 的值如何。

第四个及以后的版本号元素由 JDK 代码库的下游消费者自由使用。例如，消费者可以使用第四个元素来标识补丁发布，这些发布包含少量关键非安全修复以及对应安全发布中的安全修复。

版本号不包含尾随的零元素；即当值为零时，省略 `$SECURITY` ，当 `$MINOR` 和 `$SECURITY` 的值都为零时，省略 `$MINOR` 。

版本号中的数字序列按照数值、逐点的方式进行比较；例如， `9.9.1` 小于 `9.10.3` 。如果一个序列比另一个序列短，则较短的序列中缺失的元素被认为是小于较长的序列中相应元素的；例如， `9.1.2` 小于 `9.1.2.1` 。

**版本字符串**

版本字符串 `$VSTR` 由上述描述的版本号 `$VNUM` 组成，可选地后跟预发布和构建信息，格式如下：

```
$VNUM(-$PRE)?\+$BUILD(-$OPT)?
$VNUM-$PRE(-$OPT)?
$VNUM(+-$OPT)?
```

where:

*   `$PRE` , 匹配 `([a-zA-Z0-9]+)` --- 预发布标识符。通常 `ea` ，表示处于早期访问状态且正在积极开发且可能不稳定的版本，或 `internal` ，表示内部开发者构建。

    当比较两个版本字符串时，带有预发布标识符的字符串始终小于不带该标识符的相同版本。当预发布标识符仅由数字组成时，它们按数值比较，否则按字典顺序比较。数值标识符被视为小于非数值标识符。
*   `$BUILD` , 匹配 `(0|[1-9][0-9]*)` --- 构建号，每次提升构建时递增。 `$BUILD` 在 `$VNUM` 的任何部分递增时重置为 1。

    当比较两个版本字符串，且 `$VNUM` 和 `$PRE` 组件相等时，没有 `$BUILD` 组件的字符串总是小于有 `$BUILD` 组件的字符串；否则，比较 `$BUILD` 数字的数值。
*   $OPT，匹配 `([-a-zA-Z0-9\.]+)` --- 如有需要，可以包含额外的构建信息。在 `internal` 构建的情况下，这通常包含构建的日期和时间。

    比较两个版本字符串时，如果存在， `$OPT` 的值可能根据选择的比较方法而具有或不具有意义。

版本号 `10-ea` 与 `$VNUM = "10"` 和 `$PRE = "ea"` 匹配。版本号 `10+-ea` 与 `$VNUM = "10"` 和 `$OPT = "ea"` 匹配。

以下表格比较了 JDK 9 的潜在版本字符串，使用现有和提议的格式：

```
Existing                Proposed
Release Type    long           short    long           short
------------    --------------------    --------------------
Early Access    1.9.0-ea-b19    9-ea    9-ea+19        9-ea
Major           1.9.0-b100      9       9+100          9
Security #1     1.9.0_5-b20     9u5     9.0.1+20       9.0.1
Security #2     1.9.0_11-b12    9u11    9.0.2+12       9.0.2
Minor #1        1.9.0_20-b62    9u20    9.1.2+62       9.1.2
Security #3     1.9.0_25-b15    9u25    9.1.3+15       9.1.3
Security #4     1.9.0_31-b08    9u31    9.1.4+8        9.1.4
Minor #2        1.9.0_40-b45    9u40    9.2.4+45       9.2.4
```

以下表格显示了在新格式下，假设用于某些 JDK 7 更新和安全版本的版本字符串：

```
Actual               Hypothetical
Release Type        long           short    long          short
------------        --------------------    -------------------
Security 2013/04    1.7.0_21-b11    7u21    7.4.10+11    7.4.10
Security 2013/06    1.7.0_25-b15    7u25    7.4.11+15    7.4.11
Minor    2013/09    1.7.0_40-b43    7u40    7.5.11+43    7.5.11
Security 2013/10    1.7.0_45-b18    7u45    7.5.12+18    7.5.12
Security 2014/01    1.7.0_51-b13    7u51    7.5.13+13    7.5.13
Security 2014/04    1.7.0_55-b13    7u55    7.5.14+13    7.5.14
Minor    2014/05    1.7.0_60-b19    7u60    7.6.14+19    7.6.14
Security 2014/07    1.7.0_65-b20    7u65    7.6.15+20    7.6.15
```

**删除版本号中的初始 `1` 元素**

本提议建议从 JDK 版本号中删除初始 `1` 元素。也就是说，JDK 9 的第一个版本将具有版本号 `9.0.0` ，而不是 `1.9.0.0` 。

经过近二十年的发展，目前版本号方案的第二个元素实际上是 JDK 的默认版本号。我们在添加重要新功能以及进行不兼容更改时都会增加该元素。

我们可以将当前方案的第一个元素视为版本号，但这样 JDK 9 的版本号就会变成 `2.0.0` ，尽管大家已经习惯称之为“JDK 9”。这对任何人都没有帮助。

如果我们保留初始的 `1` ，那么 JDK 的版本号将继续违反语义化版本控制的原则，新接触 Java 的开发者将继续对例如 `1.9` 和 `9` 之间的区别感到困惑。

取消初始的 `1` 存在一定风险。比较版本号的方法有很多，其中一些可以正确工作，而另一些则不行。

* 现有的通过解析元素并按数值比较版本号的代码将继续工作，因为九大于一；即， `9.0.0` 将被视为晚于 `1.8.0` 。
* 现有的在初始元素值为 `1` 时跳过该元素的代码也将继续工作，因为在新的方案中，初始元素永远不会具有该值。
* 然而，现有的假设初始元素值为 `1` 的代码将无法正确工作；例如，此类代码将认为 `9.0.1` 在 `1.8.0` 之前。

轶事证据表明，第三类现有的代码并不常见，但我们欢迎相反的数据。

**API**

一个简单的 Java API 将用于解析、验证和比较版本字符串（8072379，8144062）：

```
package java.lang;

import java.util.Optional;

public class Runtime {

    public static Version version();

    public static class Version
        implements Comparable<Version>
    {

        public static Version parse(String);

        public int major();
        public int minor();
        public int security();

        public List<Integer> version();
        public Optional<String> pre();
        public Optional<Integer> build();
        public Optional<String> optional();

        public int compareTo(Version o);
        public int compareToIgnoreOpt(Version o);

        public boolean equals(Object o);
        public boolean equalsIgnoreOpt(Object o);

        public String toString();
        public int hashCode();
    }
}
```

一个等效的 C API 将被定义，很可能是基于修订的 jvm\_version\_info 结构。

JDK 中所有检查和比较 JDK 版本字符串的代码都将更新为使用这些 API。鼓励那些库或应用程序检查和比较 JDK 版本字符串的开发者使用这些 API。

**系统属性**

以下系统属性返回的值将由本 JEP 修改。其一般语法如下：

```
Name                            Syntax
------------------------------  --------------
java.version                    $VNUM(\-$PRE)?  
java.runtime.version            $VSTR
java.vm.version                 $VSTR
java.specification.version      $VNUM
java.vm.specification.version   $VNUM
```

系统属性 `java.class.version` 不受影响。

下表显示了不同发布类型的现有和提议的值：

```
System Property                   Existing      Proposed
-------------------------------   ------------  --------
Early Access 
  java.version                    1.9.0-ea      9-ea
  java.runtime.version            1.9.0-ea-b73  9-ea+73
  java.vm.version                 1.9.0-ea-b73  9-ea+73
  java.specification.version      1.9           9
  java.vm.specification.version   1.9           9

Major (GA)
  java.version                    1.9.0         9
  java.runtime.version            1.9.0-b100    9+100
  java.vm.version                 1.9.0-b100    9+100
  java.specification.version      1.9           9
  java.vm.specification.version   1.9           9

Minor #1 (GA)
  java.version                    1.9.0_20      9.1.2
  java.runtime.version            1.9.0_20-b62  9.1.2+62
  java.vm.version                 1.9.0_20-b62  9.1.2+62
  java.specification.version      1.9           9
  java.vm.specification.version   1.9           9

Security #1 (GA)
  java.version                    1.9.0_5       9.0.1
  java.runtime.version            1.9.0_5-b20   9.0.1+20
  java.vm.version                 1.9.0_5-b20   9.0.1+20
  java.specification.version      1.9           9
  java.vm.specification.version   1.9           9
```

请注意，历史上所有检测这些系统属性中 `.` 作为版本标识一部分的代码都需要进行检查和可能修改。例如， `System.getProperty("java.version").indexof('.')` 将返回 `-1` 以表示主要版本。

**启动器**

在 OpenJDK `java` 启动器实现中，当报告版本信息时使用系统属性，例如 `java -version` ， `java -fullversion` ，和 `java -showversion` 。

启动器的输出继续依赖于以下系统属性：

```
$ java -version
openjdk version \"${java.version}\"
${java.runtime.name} (build ${java.runtime.version})
${java.vm.name} (build ${java.vm.version}, ${java.vm.info})

$ java -showversion < ... >
openjdk version \"${java.version}\"
${java.runtime.name} (build ${java.runtime.version})
${java.vm.name} (build ${java.vm.version}, ${java.vm.info})
[ ... ]

$ java -fullversion
openjdk full version \"${java.runtime.version}\"
```

实现细节可以在源代码中找到。

**`@since` JavaDoc 标签**

`@since` JavaDoc 标签的值将继续与系统属性 `java.specification.version` 对齐；因此，新的 JDK 9 API 将由 `@since 9` 表示。

**Mercurial 变更集标签**

Mercurial 标签用于标识推广的变更集。例如，用于验证推送到 JDK 发布森林的所有变更集的 Code Tool 的 jcheck 工具，将增强以支持使用新版本方案进行标签。

Mercurial 标签的一般语法是 `jdk\-$VNUM\+$BUILD` 。下表展示了不同发布类型的建议值：

```
Release Type      Proposed
----------------  -----------
Major (GA)        jdk-9+100
Minor #1 (GA)     jdk-9.1.2+27
Security #1 (GA)  jdk-9.0.1+3
```

一些工具可能需要同时支持现有和提议的标签格式。

## \[功能] JEP 224: HTML5 Javadoc

### 摘要

增强 javadoc 工具以生成 [HTML5](https://www.w3.org/TR/html5/) 标记。

### 描述

* 在标准 doclet 中添加了一个命令行选项，用于请求特定类型的输出标记。HTML4 是当前类型，将是默认值。HTML5 将在 JDK 10 中成为默认值。
* 通过使用结构化 HTML5 元素（如`页眉` 、`页脚`、`导航`、 _等等。_）来提高生成的 HTML 的语义值。
* HTML5 标记实现了 [WAI-ARIA 标准 ](http://www.w3.org/WAI/intro/aria)，用于可访问性。使用 role 属性将特定角色分配给 HTML 文档中的元素。
* The `-Xdoclint` 功能已更新，用于检查文档注释中的常见错误，基于请求的输出标记类型。

## JEP 226: UTF-8 properties文件

### 摘要

定义一种方法，使应用程序能够指定以 UTF-8 编码的属性文件，并扩展 ResourceBundle API 以加载它们

### 描述

更改 `ResourceBundle` 类的默认文件编码，从 ISO-8859-1 转换为 UTF-8。这样做后，应用程序不再需要使用转义机制转换属性文件。现有的属性文件很少受到影响，因为 ISO-8859-1 的 U+0000-U+007F 与 UTF-8 兼容，而代码点超过 U+00FF 的字符应该已经转义。如果在 UTF-8 中读取属性文件时发生异常，无论是 `MalformedInputException` 还是 `UnmappableCharacterException`，都会重新从头开始读取属性文件，回退到使用 ISO-8859-1 编码。为了在极少数情况下将 ISO-8859-1 属性文件识别为有效的 UTF-8 文件，此 JEP 提供了一种方法，通过设置系统属性"`java.util.PropertyResourceBundle.encoding`"来显式指定编码，无论是 ISO-8859-1 还是 UTF-8。

### 说明

变化前后总结：

| 项目                 | Java 8 及之前                    | Java 9 及之后（JEP 226） |
| ------------------ | ----------------------------- | ------------------- |
| `.properties` 默认编码 | ISO-8859-1（Latin-1）           | ✅ UTF-8             |
| 中文等非 Latin 字符处理    | 必须转义为 `\uXXXX`                | 直接写中文，无需转义          |
| 加载类                | `java.util.Properties.load()` | 同上，但编码行为改变          |
| 工具支持               | `native2ascii` 工具常用           | ✅ 不再需要              |

## \[优化] JEP 227: Unicode 7.0

将现有平台 API 升级以支持[版本 7.0](http://www.unicode.org/versions/Unicode7.0.0/) 的 [Unicode 标准](http://www.unicode.org/standard/standard.html)

## \[JVM] JEP 228：添加更多诊断命令

### 摘要

定义额外的诊断命令，以提高 Hotspot 和 JDK 的可诊断性。

### 描述

这是新命令的列表（确切名称待定）：

**`print_class_summary`**

* 打印所有已加载类及其继承结构的列表。
* 负责小组：运行时

**`print_codegenlist`**

* 打印编译队列中的 C1 或 C2 方法（分别排队）
* 负责小组：编译器

**`print_utf8pool`**

* 打印所有 UTF-8 字符串常量。
* 负责小组：运行时

**`datadump_request`**

* 向 JVM 发送信号以进行 JVMTI 数据转储请求。
* 负责小组：服务性

**`输出代码列表`**

* 打印 n 方法（编译后）的完整签名、地址范围和状态（存活、不可进入和僵尸）。
* 允许选择打印到 stdout 或文件。
* 允许以 XML 或文本格式打印。
* 负责小组：编译器

**`打印代码块`**

* 打印代码缓存的大小以及代码缓存中的块列表，包括地址。
* 负责小组：编译器

**`set_vmflag`**

* 在虚拟机或库中设置命令行标志/选项。
* 负责小组：可用性

## \[优化] JEP 229: 默认创建 PKCS12 密钥库

### 摘要

将默认密钥库类型从 JKS 转换为 [PKCS12](https://en.wikipedia.org/wiki/PKCS_12)。

### 描述

此功能将默认密钥库类型从 JKS 更改为 PKCS12。默认情况下，新密钥库将以 PKCS12 密钥库格式创建。现有密钥库不会更改，密钥库应用程序可以继续明确指定它们所需的密钥库类型。

现有应用程序不应受到影响。密钥库通常具有较长的生命周期，因此我们需要支持跨多个 JDK 版本访问。访问由早期 JDK 版本创建的密钥库的应用程序必须在 JDK 9 上运行而不做任何更改。同样，访问由 JDK 9 创建的密钥库的应用程序应在早期 JDK 版本上运行而不做任何更改。

通过引入一种理解 JKS 和 PKCS12 格式的密钥库检测机制来实现此要求。在加载密钥库之前检查其格式，以确定其类型，然后使用适当的密钥库实现来访问它。该机制默认启用，但在需要时可以禁用。

可将此密钥库检测机制的支持回滚到早期 JDK 版本。

## \[优化] JEP 231: 删除启动时 JRE 版本选择

### 概括

删除在 JRE 启动时请求非正在启动的 JRE 版本的能力。

### 动机

“多个 JRE”（mJRE）功能允许开发者指定可以用于启动应用程序的 JRE 版本或版本范围。版本选择标准可以指定在应用程序的 `jar` 文件（ `JRE-Version` ）的清单条目中，或作为 `java` 启动器的命令行选项（ `-version:` ）。如果启动的 JRE 版本不满足这些标准，则启动器会搜索满足条件的版本，如果找到，则启动该版本。

实际上，部署应用程序需要做的不仅仅是选择特定的 JRE。现代应用程序通常通过[Java Web Start (JNLP)](https://en.wikipedia.org/wiki/Java_Web_Start)、原生操作系统打包系统或主动安装程序进行部署，所有这些技术都有各自的方式为应用程序查找合适的 JRE，有时甚至会安装并随后更新。

mJRE 功能仅解决了整体部署问题的一部分。此外，当它在 JDK 5 中引入时，从未得到充分文档记录： `-version:` 选项在 `java` 命令的文档中提到，但 `JRE-Version` 清单条目在任何常规 JDK 文档中都没有提到，也没有在 Java SE 平台规范中提到。据我们所知，这个功能很少被使用。它无谓地复杂化了 Java 启动器的实现，使得维护和增强变得困难。

### 描述

删除 mJRE 功能。修改启动器如下：

* 如果在命令行上提供了 `-version:` 选项，则发出错误消息并退出
* 如果在 `jar` 文件中找到 `JRE-Version` 清单条目，则发出警告消息并继续。

在第二种情况下，选择警告而不是致命错误的原因是清单条目可能存在于无法轻易修改的旧 `jar` 文件中，因此继续而不是中止会更好。我们预计将在 JDK 10 中将此情况更改为致命错误。

## \[优化] JEP 232: 提升安全应用程序性能

### 摘要

改进使用安全管理者安装的应用程序的性能。

### 描述

我们探索并实现了许多优化和增强，以提高使用安全管理者运行的应用程序的性能。其中一些优化提高了性能，而另一些则没有。还有一些被证明有潜力，但由于各种原因，将不会作为本 JEP 的一部分进行整合。为每个考虑的优化打开了新的 JBS 问题（如果之前不存在），并使用 JMH 创建了微基准测试。

### 优化

基于测试和社区反馈，我们提高性能的主要关注领域是安全策略的执行和权限的评估。权限类和默认 JDK 策略实现被设计为线程安全的。然而，使用多个线程的性能测试表明，这些类是热点。我们实现了几个改进，以提高吞吐量和减少线程竞争：

1. [使用 ConcurrentHashMap 将 ProtectionDomain 映射到 PermissionCollection](https://bugs.openjdk.java.net/browse/JDK-8055753)
2. [SecureClassLoader 应使用 ConcurrentHashMap](https://bugs.openjdk.java.net/browse/JDK-8064890)
3. [移除同步在 identityPolicyEntries 列表上的策略提供者代码](https://bugs.openjdk.java.net/browse/JDK-8065233)
4. [在 Permissions 类中将 PermissionCollection 条目存储在 ConcurrentHashMap 中，而不是 HashMap](https://bugs.openjdk.java.net/browse/JDK-8065942)
5. [在 PermissionCollection 子类中将权限存储在并发集合中](https://bugs.openjdk.java.net/browse/JDK-8056179)

我们还在其他两个关键领域提高了性能：

* 我们将 `java.security.CodeSource` 的 `hashCode` 方法进行了更改，以避免昂贵的 DNS 查找，通过使用 codesource URL 的字符串形式来计算哈希码。有关更多信息，请参阅 JDK-6826789。
* 我们增强了 `java.lang.SecurityManager` 的 `checkPackageAccess` 方法的包检查算法。有关更多信息，请参阅 JDK-8072692。

## \[测试] JEP 233: 自动生成运行时编译器测试

### 摘要

开发一个工具，通过自动生成测试用例来测试运行时编译器。

### 描述

工具将随机生成语法和语义正确的 Java 源代码或字节码，如有必要，将其编译，以解释模式（`-Xint`）和编译模式（`-Xcomp`）运行，并验证结果。

工具将自动运行，无需人工交互。生成的测试将在合理的时间内尽可能覆盖尽可能多的组合。

Java 源代码编译器 `javac` 不使用 Java 的所有字节码，因此仅生成 Java 源代码将导致一些字节码未被覆盖。仅对所有类型的测试生成字节码将是一个更复杂的任务，因此我们将采用混合方法，生成 Java 源代码和字节码。

在测试执行期间编译源代码对于嵌入式平台来说是个问题，因为这些平台上可能没有完整的 JDK，所以该工具将提供一种预编译源代码测试的方法。

生成的测试用例将包括复杂的表达式和控制流图，并使用内联函数、浮点运算、`try-catch-finally` 结构等。将有一种方法来调整工具的配置。

工具将随机生成测试用例，但为了可重现性，它将报告其随机化种子，并接受这样的种子以重新播放生成的测试用例的精确序列。

工具的源代码将被放置在 `hotspot/test/testlibrary/jit-tester` 目录中。可以通过工具的 makefile 中提供的目标生成测试。测试生成的结果是完整的 `jtreg` 测试套件，可以从相同的 makefile 或通过 `jtreg` 直接运行。工具的 makefile 将不会集成到 HotSpot/JDK 构建基础设施中。

由于测试生成过程需要花费相当多的时间，因此生成和运行这些测试不应该是预集成测试的一部分。然而，定期运行预生成的测试，用于可靠性测试，以及运行新生成的测试，以获得更好的代码覆盖率是有意义的。发现错误的生成测试应作为常规回归测试集成到适当的测试套件中，并以与其他回归测试相同的方式进行运行。

## \[测试] JEP 235：[测试 javac 生成的类文件属性](https://openjdk.org/jeps/235)

### 摘要

编写测试以验证由 `javac` 生成的类文件属性的准确性。

### 描述

测试由 `javac` 生成的文件的常用方法是运行编译后的类并验证生成的程序是否按预期行为。这种方法不适用于可选的类文件属性，也不适用于 VM 未验证的属性，因此这两种类型的属性必须通过其他方式测试。将开发出能够接受 Java 源代码作为输入、编译源代码、读取编译后的类文件的类文件属性并验证其正确性的测试。

类文件属性根据以下内容分为三组， [Java 虚拟机规范（JVM）](http://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.7)。

**可选属性**

这些属性对于 `javac`、JVM 或类库的正确运行并非至关重要，但它们被工具所使用。测试这些属性是高优先级的，因为它们不被 JDK 的任何组件所消费。

* `源文件`
* `源调试扩展`
* `行号表`
* `局部变量表`
* `局部变量类型表`
* `已过时`

**JVM 未使用的属性**

这些属性不被 JVM 使用，但它们被 `javac` 或类库使用。测试这些属性是中等优先级。

* `内部类`
* `封装方法`
* `合成`
* `签名`
* `运行时可见注解`
* `运行时不可见注解`
* `RuntimeVisibleParameterAnnotations`
* `RuntimeInvisibleParameterAnnotations`
* `运行时可见类型注解`
* `RuntimeInvisibleTypeAnnotations`
* `默认注解`
* `方法参数`

**属性由 JVM 使用**

这些属性将由 JVM 的字节码验证器进行检查。无需进一步测试。

* `常量值`
* `代码`
* `栈映射表`
* `异常`
* `BootstrapMethods`

## \[JVM] JEP 236：[Nashorn 的解析器 API](https://openjdk.org/jeps/236)

### 摘要

定义 Nashorn 的 ECMAScript 抽象语法树的支持 API。

### 动机

NetBeans 等 IDE 使用 Nashorn 进行 ECMAScript 编辑/调试以及 ECMAScript 代码分析。这些工具和框架目前使用 Nashorn 的内部 AST 表示进行代码分析。这种在 `jdk.nashorn.internal.ir` 包及其子包中使用内部类的方法，阻止了 Nashorn 内部实现类的自由演进。本 JEP 将定义一个在公开包 `jdk.nashorn.api.tree` 中的 Nashorn 解析器 API。类似的抽象语法树 API 已在 `javac` 的 `com.sun.source`[ 包及其子包](http://docs.oracle.com/javase/8/docs/jdk/api/javac/tree/index.html)中得到支持。

解析器 API 将使 IDE 和服务器端框架等程序能够进行 ECMAScript 代码分析，而无需这些程序依赖于 Nashorn 的内部实现类。

### 描述

附件中的 javadoc 文件包含了新 `jdk.nashorn.api.tree` 包中提议的接口和类的文档。API 的起点是 `ParserFactory` 和 `ParserFactoryImpl`。 类。一个 `ParserFactory` 对象接受一个字符串数组，这些字符串是配置解析器的选项。支持的选项与 Nashorn 壳工具 `jjs` 支持的选项相同，以及 nashorn.args Nashorn 脚本引擎的系统属性。

一旦创建了解析器实例，然后从字符串中获取 ECMAScript 源代码 URL 或文件可以提交给解析器，解析器将返回 `编译单元树`对象。任何解析错误将通过调用者提供的 `诊断监听器`对象进行报告。

## \[JVM] JEP 237：[Linux/AArch64 端口](https://openjdk.org/jeps/237)

### 摘要

将 JDK 9 移植到 Linux/AArch64

### 描述

我们（AArch64 移植项目）已将 JDK 移植到新的平台：Linux/AArch64。我们已实现了模板解释器、C1（客户端）和 C2（服务器）JIT 编译器。

本 JEP 的重点不是移植工作本身，这项工作已基本完成，而是将移植版本集成到 OpenJDK 主仓库中。

目前 HotSpot 仓库共享部分存在大量琐碎的变更集。这些大多是#ifdef，用于包含相关平台特定的文件。还有一些其他类型的变更，但同样都由#ifdef AARCH64 保护。因此，对其他平台的风险很低。

此外，还对 HotSpot 和 JDK 的构建机制进行了更改，以添加适当的定义，例如字节序、字大小等。这些更改也不应该影响其他平台。

要集成的多数变更不会以任何方式影响当前 OpenJDK 平台，因为它们仅在新的平台上有效。

此外，构建系统也有一些更改，但它们不应该引起太多麻烦。

[这里](http://cr.openjdk.java.net/~aph/aarch64.1)是一个对 HotSpot 共享代码所需更改的补丁。

所有更改集都收集在一个临时存储库中： [http://hg.openjdk.java.net/aarch64-port/stage/](http://hg.openjdk.java.net/aarch64-port/stage/)

## \[JVM] JEP 238：[多版本 JAR 文件](https://openjdk.org/jeps/238)

### 摘要

扩展 JAR 文件格式，以允许在单个归档中存在多个、针对 Java 版本特定的类文件版本。

### 描述

JAR 文件有一个内容根，其中包含类和资源，以及一个包含 JAR 元数据的 `META-INF` 目录。通过向特定文件组添加一些版本控制元数据，JAR 格式可以以兼容的方式编码针对不同目标 Java 平台版本的多个库版本。

多版本 JAR 文件（"MRJAR"）将包含以下主属性：

```
Multi-Release: true
```

在 JAR 的 `MANIFEST.MF` 主部分中声明。属性名称也声明为常量 `java.util.jar.Attributes.MULTI_RELEASE` 。与其他主属性一样，`MANIFEST.MF` 中声明的名称不区分大小写。值也不区分大小写，但值前不能有前导空格，值后不能有尾随空格（这种限制有助于确保达到性能目标）。

多版本 JAR 文件（"MRJAR"）将包含针对特定 Java 平台版本特定的类和资源目录。一个典型库的 JAR 文件可能看起来像这样：

```
jar root
  - A.class
  - B.class
  - C.class
  - D.class
```

假设有 A 和 B 的替代版本可以利用 Java 9 的特性。我们可以将它们打包成一个 JAR 文件，如下所示：

```
jar root
  - A.class
  - B.class
  - C.class
  - D.class
  - META-INF
     - versions
        - 9
           - A.class
           - B.class
```

在不支持 MRJARs 的 JDK 中，只有根目录中的类和资源可见，两种打包方式无法区分。在支持 MRJARs 的 JDK 中，对应任何后续 Java 平台版本的目录将被忽略；它首先在对应当前运行的主要 Java 平台版本版本的 Java 平台特定目录中搜索类和资源，然后搜索较低版本的目录，最后搜索 JAR 根目录。在 Java 9 JDK 中，这就像有一个包含版本 9 文件的特定 JAR 类路径，然后是 JAR 根目录；在 Java 8 JDK 中，这个类路径只包含 JAR 根目录。

假设将来 Java 10 发布后，A 更新以利用 Java 10 的特性。此时 MRJAR 可能看起来像这样：

```
jar root
  - A.class
  - B.class
  - C.class
  - D.class
  - META-INF
     - versions
        - 9
           - A.class
           - B.class
        - 10
           - A.class
```

通过此方案，为后续 Java 平台版本设计的类版本可以覆盖为早期 Java 平台版本设计的同一类版本。在上面的示例中，当在了解 MRJAR 的 Java 9 JDK 上运行时，它会看到 A 和 B 的 9 特定版本以及 C 和 D 的通用版本；在未来的了解 MRJAR 的 Java 10 JDK 上，它会看到 A 的 10 特定版本和 B 的 9 特定版本；在较旧或非 MRJAR 了解的 JDK 上，它只会看到所有类的根版本。

JAR 元数据，例如在 `MANIFEST.MF` 文件中找到的 《META-INF/services》目录不需要版本化。MRJAR 本质上是一个发布单元，因此它只有一个发布版本（这与通过 Maven Central 分发的普通 JAR 没有区别），尽管它内部包含多个用于不同 Java 平台发布版本的库实现版本。库的每个版本都应该提供相同的 API；需要调查这是否应该严格向后兼容，即 API 完全相同（字节码签名相等），或者是否可以适度放宽，而无需必然引入新的增强功能，这些功能可能会模糊一个发布单元的概念。这至少意味着，在特定发布目录中存在的公共类也应该在根目录中存在，尽管它不需要在早期发布目录中存在。运行时系统将不会验证此属性，但工具可以并应该检测此类 API 兼容性问题，还可以提供一个库方法来执行此类验证（例如在 `java.util.jar.JarFile`）。

最终，此机制使库和框架开发者能够将特定 Java 平台发布版本中 API 的使用与要求所有用户迁移到该版本的需求解耦。库和框架维护者可以逐步迁移到并支持新功能，同时仍然支持旧功能，打破鸡生蛋的循环，这样库就可以“Java 9 就绪”，而实际上并不需要 Java 9。

## \[优化] JEP 240：[删除 JVM TI hprof 代理](https://openjdk.org/jeps/240)

## \[优化] JEP 241：[删除 jhat 工具](https://openjdk.org/jeps/241)

## \[JVM] JEP 243：[Java 级 JVM 编译器接口](https://openjdk.org/jeps/243)

## \[功能] JEP 244：[TLS应用层协议协商扩展](https://openjdk.org/jeps/244)

### 摘要

扩展 `javax.net.ssl` 包以支持 TLS [应用层协议协商（ALPN）扩展 ](http://www.rfc-editor.org/rfc/rfc7301.txt)，它提供了协商 TLS 连接的应用协议的手段

### 描述

该功能定义了一个公共 API，用于协商在给定的 TLS 连接上可以传输的应用层协议。协议名称在初始 TLS 握手期间由客户端和服务器之间传递。

TLS 应用可以使用扩展的 `SSLParameters` 类来获取和设置它可以在给定连接上支持的应用层协议列表。TLS 实现也使用此类来检索应用程序声明的协议名称。

默认行为是选择服务器对启用应用程序协议值的最优选交集值。

服务器应用程序还可以外部扫描初始明文 ClientHellos 以选择此连接的适当 ALPN 协议值。此决策可能基于提供的 TLS 协议、加密套件、服务器名称指示值等。然后服务器应用程序可以：

* 选择其中一种提供的协议，如果它将支持它，
* 决定远程提供的和本地支持的 ALPN 值是互斥的，或
* 完全忽略该扩展。

服务器可以根据连接期间可用的应用协议更改连接参数，例如它所宣传的服务器证书。

在 SSL/TLS 握手开始后，`SSLSocket/SSLEngine` 上有新的方法。 这些方法允许应用程序查询是否已选择 ALPN 值（ `getHandshakeApplicationProtocol()` ）。 一旦 TLS 握手完成，应用程序就可以 检查使用哪个协议进行协商 《getApplicationProtocol()》方法。

建议的设计遵循了用于的类似 API 方法 [服务器名称指示扩展（JEP 114）](http://openjdk.java.net/jeps/114) 在 JDK 8 中引入，但与 ALPN 值与连接相关联，而不是与 `SSL 会话`相关联有所不同。

### 说明

JEP 244: **"TLS Application-Layer Protocol Negotiation Extension"**（TLS 应用层协议协商扩展）在 OpenJDK 中引入了对 [**ALPN**](https://tools.ietf.org/html/rfc7301)（Application-Layer Protocol Negotiation）的支持。

**✅ 为什么需要它？**

某些应用层协议（例如 HTTP/2）必须通过 TLS 的 ALPN 扩展来协商。没有 ALPN，客户端和服务器可能无法达成一致，导致协议降级或连接失败。

**🧪 示例：使用 ALPN 协商 HTTP/2**

Java 代码（客户端）使用 ALPN 与支持 HTTP/2 的服务器通信：

```
import javax.net.ssl.*;
import java.net.Socket;
import java.util.List;

public class ALPNClient {
    public static void main(String[] args) throws Exception {
        SSLSocketFactory factory = (SSLSocketFactory) SSLSocketFactory.getDefault();
        SSLSocket socket = (SSLSocket) factory.createSocket("www.google.com", 443);

        // 设置客户端支持的协议列表（HTTP/2 优先）
        SSLParameters sslParams = socket.getSSLParameters();
        sslParams.setApplicationProtocols(new String[] { "h2", "http/1.1" });
        socket.setSSLParameters(sslParams);

        socket.startHandshake();

        String protocol = socket.getApplicationProtocol(); // 握手后协商出来的协议
        System.out.println("Negotiated protocol: " + protocol);

        socket.close();
    }
}
```

**输出示例：**

```
Negotiated protocol: h2
```

表示 TLS 握手时协商成功，双方同意使用 HTTP/2（"h2" 是 HTTP/2 的 ALPN 标识符）。

***

**🔐 Java 版本要求**

* **JDK 9+**：内置支持 ALPN（来自 JEP 244）
* **JDK 8**：默认不支持，需要使用 Jetty ALPN Boot 或升级到更新版本的 JDK

***

**📦 应用场景**

| 应用层协议    | ALPN 协议 ID     |
| -------- | -------------- |
| HTTP/2   | `"h2"`         |
| HTTP/1.1 | `"http/1.1"`   |
| gRPC     | `"h2"`         |
| QUIC     | 使用类似方式（基于 UDP） |

如果你正在实现基于 HTTP/2 的客户端、gRPC 服务端，或者自己开发 TLS 通讯协议，ALPN 都是必不可少的一环。需要我给出服务端的示例也可以继续补充 👍

## \[JVM] JEP 245：[验证 JVM 命令行标志参数](https://openjdk.org/jeps/245)

### 摘要

验证所有 JVM 命令行标志的参数，以避免崩溃，并在参数无效时显示适当的错误信息。

### 描述

任何接口，无论是程序性的还是用户可见的，都必须提供足够的验证输入值的能力。在命令行的情况下，对需要用户指定值的参数实现范围检查是至关重要的。`globals.hpp` 源文件包含了标志值的来源和基本的范围检查。扩展和完善这一点可以提供正确的覆盖范围。

此外，我们还应该定义一个框架，使得添加新的 JVM 命令行标志的人能够轻松地利用这种有效性检查。该框架应具有灵活性，允许检查单个值、在最小值和最大值之间，或者在一个值集中等。

我们将通过扩展现有的宏表（例如， `RUNTIME_FLAGS`）来添加可选的 `range(min, max)` 和 `constraint(function_pointer)` 条目。当前的检查范围和其他临时验证代码将被移植过来，然后删除。

范围和约束检查在每次标志更改时都会进行，同时在 JVM 初始化例程晚期（即在 `init_globals()`） 在 `stubRoutines_init2()`) 之后，在所有标志都设置最终值时进行。只要 JVM 运行，我们就会继续检查可管理的标志。

对于那些依赖于其他可能不在设置相关标志时设置的标志的标志，我们将提供一种机制，即 API（ `CommandLineFlags::finishedInitializing()` ），以便在所有标志都设置最终值时通知约束函数，并根据需要将行为从 NOP 更改为错误。

在 `CommandLineFlags::xxxxAtPut` 设置器中在低级别拦截标志值更改，以确保可管理的标志被检查其范围和约束（例如，通过 `jcmd` 设置的标志）。例如，使用 `jcmd PID VM.set_flag MinHeapFreeRatio 101` ，它超出了允许的范围，在 `jcmd` 输出中将打印出

```
PID:
MinHeapFreeRatio error: must have value in range [0...100]
```

范围检查不会对 JVM 初始化过程产生任何行为变化。特别是，它们不会终止 JVM，而是将它们的状态传播到使用它们的代码。约束函数可以根据需要终止 JVM 以匹配现有的自定义行为。

默认情况下，范围/约束检查是非详尽的，以抑制它们的 消息不会被打印出来。范围检查会打印错误消息。 JVM 初始化期间的错误流，以便匹配当前 行为。对于可管理的标志，打印到错误流被抑制；任何错误状态将由 而不是由目标进程来处理，而是通过 `可写标志`代码提供详细的输出状态到提供的 `格式化缓冲区` ，以便它可以通过 `jcmd` 进程本身而不是目标进程来打印。

在 JVM 初始化过程中发生范围检查失败时，默认情况下会以以下形式打印错误信息：

```
uintx UnguardOnExecutionViolation = 3 is outside the allowed range [ 0 ... 2 ]
```

然而，我们目前并没有承诺任何特定的格式。现有的测试预期某种消息格式，将需要修改以允许新的格式。

尽管我们有这种能力，但现有的行为并没有改变，即我们不会对范围限制标志进行限制（即我们不限制），我们在初始化 JVM 时检测错误时遵循现有行为（即我们终止进程）。

## \[JVM] JEP 246：[利用 CPU 指令进行 GHASH 和 RSA](https://openjdk.org/jeps/246)

通过利用最近引入的 SPARC 和 Intel x64 CPU 指令，提高 GHASH 和 RSA 加密操作的性能。

## \[JVM] JEP 247：[针对旧平台版本进行编译](https://openjdk.org/jeps/247)

增强 `javac` 以使其能够将 Java 程序编译为在选定的旧平台版本上运行。

## \[JVM] JEP 248：[将 G1 设为默认垃圾收集器](https://openjdk.org/jeps/248)

在 32 位和 64 位服务器配置上，将 G1 作为默认垃圾回收器。

## \[功能] JEP 249：[TLS 的 OCSP 装订](https://openjdk.org/jeps/249)

### 摘要

通过 TLS 证书状态请求扩展（RFC 6066 的第 8 节）和多个证书状态请求扩展（RFC 6961）实现 OCSP Stapling。

### 描述

此功能将在 `SunJSSE` 提供者实现中实现。计划进行一些小的 API 更改，目标是尽可能保持这些更改最小。实现将选择合理的 OCSP 特定参数默认值，并通过以下系统属性提供这些默认值的配置：

* `jdk.tls.client.enableStatusRequestExtension` ：此属性默认为 true。它启用了 `status_request` 和 `status_request_v2` 扩展，并启用了对服务器发送的 `CertificateStatus` 消息的处理。
* `jdk.tls.server.enableStatusRequestExtension` ：此属性默认为 false。它启用了服务器端对 OCSP Stapling 的支持。
* 此属性控制服务器获取 OCSP 响应的最大时间，无论是从缓存中获取还是通过联系 OCSP 响应者。如果适用，将接收到的响应发送到证书状态消息中。此属性以毫秒为单位的整数值，默认值为 5000。
* 此属性控制缓存条目数量的最大值。默认值为 256 个对象。如果缓存已满且需要缓存新的响应，则最不常用的缓存条目将被新的条目替换。此属性的值为零或更小意味着缓存将没有响应数量的上限。
* 此属性控制缓存的响应的最大寿命。该值以秒为单位指定，默认值为 3600（1 小时）。如果响应具有 nextUpdate 字段，并且其过期时间早于缓存寿命，则响应的寿命可能短于此属性设置的值。此属性的值为零或更小将禁用缓存寿命。如果对象没有 nextUpdate 且已禁用缓存寿命，则不会缓存响应。
* 此属性允许管理员在用于 TLS 的证书没有 Authority Info Access 扩展的情况下设置默认 URI。除非设置了 jdk.tls.stapling.responderOverride 属性（见下文），否则它不会覆盖 AIA 扩展的值。此属性默认不设置。
* 此属性允许通过 jdk.tls.stapling.responderURI 属性提供的 URI 覆盖任何 AIA 扩展值。默认为 false。
* 此属性禁用转发在 `status_request` 或 `status_request_v2` TLS 扩展中指定的 OCSP 扩展。默认为 false。

客户端和服务器端的 Java 实现将能够支持 `status_request` 和 `status_request_v2` TLS 握手扩展。 `status_request` 扩展在 RFC 6066 中描述。支持的服务器将包括用于在新的 TLS 握手消息中标识服务器所使用的单个 OCSP 响应（ `CertificateStatus` ）。 `status_request_v2` 扩展在 RFC 6961 中描述。该扩展允许客户端请求服务器在 `CertificateStatus` 消息中提供单个 OCSP 响应（类似于 `status_request` ）或请求服务器为在证书消息中提供的证书列表中的每个证书获取 OCSP 响应（以下称为 `ocsp_multi` 类型）。

**客户端**

* OCSP Stapling 将默认启用，可以通过设置系统属性来禁用。这可以通过 `jdk.tls.client.enableStatusRequestExtension` 属性完成。
* 默认情况下，客户端将断言 `status_request` 和 `status_request_v2` 扩展在 `ClientHello` 握手消息中。对于 `status_request_v2` 扩展，将断言 `ocsp` 和 `ocsp_multi` 类型。
* 创建 hello 扩展将需要在 `sun.security.ssl` 中创建新的类，类似于 `ServerNameIndicator` 、 `RenegotiationInfoExtension` 和其他扩展的实现方式。
* 为了使用新的扩展， `ClientHello` 类将需要定义额外的添加这些扩展的方法。这些方法将从 `ClientHandshaker.clientHello()` 中调用。
* 需要在 `HandshakeMessage` 类中创建一个新的握手消息类来处理 `CertificateStatus` 消息的编码和解码。
*   在 `ExtendedSSLSession` 中需要进行公共 API 更改，以便调用者能够获取握手过程中收到的 OCSP 响应。新方法如下：

    ```
    public List<byte[]> getStatusResponses();
    ```

**服务器端**

* 服务器端实现默认禁用 OCSP Stapling，但可以通过 `jdk.tls.server.enableStatusRequestExtension` 系统属性启用。不支持 OCSP Stapling 的服务器将忽略 `status_request` 和 `status_request_v2` 扩展。
* 服务器端在 `ServerHello` 消息中填充 `status_request` 或 `status_request_v2` 信息将取决于客户端如何声明这些扩展。通常， `ClientHello` 中的相同请求扩展将在 `ServerHello` 中返回，但有以下例外：
  * 接收 `ClientHello` 中的 `status_request` 和 `status_request_v2` 扩展的服务器将在 `ServerHello` 中声明 `status_request_v2` 。
  * 接收 `status_request_v2` 扩展的 `ClientHello` 服务器，在同时具有 `ocsp` 和 `ocsp_multi` 类型的情况下，将在 `ServerHello` 消息中声明 `status_request_v2` ，并在 `CertificateStatus` 消息中声明 `ocsp_multi` 。
  * 如果选择 `status_request_v2` / `ocsp_multi` ，将使用不同的线程来获取每个响应。这将由一个 `StatusResponseManager` 来管理，该 `StatusResponseManager` 将处理 OCSP 响应的获取和缓存。
* 应尽可能缓存 OCSP 响应。未在它们的 `status_request[_v2]` 扩展中指定非 ces 的客户端可能会收到缓存的响应。
  * 缓存的响应不应在当前时间晚于 `nextUpdate` 字段时使用。
  * 缓存中无 `nextUpdate` 字段的已缓存响应可以保留在缓存中，其生命周期由预设值决定（见以下可调整参数）。
  * 接收带有 nonce 扩展的 `status_requests` 的服务器不得在 `CertificateStatus` 消息中返回缓存响应。
* 服务器端 stapling 支持将通过上述系统属性进行调节。
* `StatusResponseManager` 是在 `SSLContext` 实例化过程中创建的。属性值在 `SSLContext` 构建期间进行采样。这些属性值可以被更改，并且当创建一个新的 `SSLContext` 对象时，StatusResponseManager 将具有这些新值。

### Stapling 和 X509ExtendedTrustManagers

开发者在如何处理通过 OCSP Stapling 提供的响应方面有一定的灵活性。本 JEP 对证书路径检查和吊销检查的现有方法没有任何更改。这意味着可以同时让客户端和服务器断言 `status_request` 扩展，通过 `CertificateStatus` 消息获取 OCSP 响应，并允许用户在如何响应吊销信息或缺乏吊销信息方面具有灵活性。

与之前的 JDK 版本一样，如果调用者没有提供 `PKIXBuilderParameters` ，则禁用吊销检查。如果调用者创建 `PKIXBuilderParameters` 并使用 `setRevocationEnabled` 方法启用吊销检查，则将评估 OCSP Stapling 响应。如果将 `com.sun.net.ssl.checkRevocation` 属性设置为 `true` ，也是如此。下表展示了几个不同的示例（假设客户端和服务器都启用了 OCSP Stapling）：

| PKIXBuilderParameters | checkRevocation 属性 | PKIX 撤销检查器                     | 结果                     |
| --------------------- | ------------------ | ------------------------------ | ---------------------- |
| 默认                    | 默认                 | 默认                             | 撤销检查已禁用                |
| 默认                    | 是的                 | 默认                             | 启用吊销检查\*，设置 SOFT\_FAIL |
| 已实例化                  | 默认                 | 默认                             | 启用吊销检查\*，设置 SOFT\_FAIL |
| 已实例化                  | 默认                 | 已实例化，添加到 PKIXBuilderParameters | 启用吊销检查\*，硬失败行为。        |

\* 仅当 `ocsp.enable` 安全属性设置为 true 时，客户端 OCSP 回退才会发生。

关于 `PKIXBuilderParameters` 和 `PKIXRevocationChecker` 对象的配置及其与 JSSE 关系的更多详细信息，可以在 Java PKI API 程序员指南和 JSSE 参考指南中找到。

## \[JVM] JEP 250：[将内部字符串存储在 CDS 档案中](https://openjdk.org/jeps/250)

### 摘要

在类数据共享（CDS）归档中存储内部字符串。

### 描述

在转储时间，在堆初始化期间在 Java 堆中分配一个指定的字符串空间。在写入内部字符串表和 `String` 对象时，修改指向内部 `String` 对象及其底层 `char`-数组对象的指针，就像这些对象来自指定的空间一样。

字符串表被压缩，然后在转储时间存储在存档中。字符串表的压缩技术与共享符号表相同（参见 [JDK-8059510](https://bugs.openjdk.java.net/browse/JDK-8059510)）。使用常规窄 oop 编码和解码来访问从压缩字符串表中的共享 `String` 对象。

在 64 位平台且具有压缩 oop 指针的情况下，窄 oop 使用偏移量（带或不带缩放）从窄 oop 基址进行编码。目前有四种不同的编码模式：32 位未缩放、基于零、基于非重叠堆和基于堆。根据堆大小和堆最小基址，选择合适的编码模式。窄 oop 编码模式（包括编码位移）必须在转储时间和运行时相同，以确保共享字符串空间中的 oop 指针在运行时保持有效。共享字符串空间在运行时可以被视为可重定位的，但有限制。它不需要在转储时间和运行时映射到相同的地址，但应该在与转储时间和运行时相同的偏移量处从窄 oop 基址开始。堆大小在转储时间和运行时不需要相同，只要使用相同的编码模式即可。字符串空间和 oop 编码模式（以及位移）的偏移量应存储在存档中以进行运行时验证。如果编码模式发生变化，将使每个共享的 `char` 数组的 oop 指针编码无效。 在此类情况下，共享字符串数据会被忽略，而其余的共享数据仍然可以被虚拟机使用。虚拟机将报告一条警告信息，指出由于不兼容的 GC 配置，共享字符串未被使用。

在运行时，字符串空间被映射为 Java 堆的一部分，其偏移量与转储时的 oop 编码基相同。映射从存档中保存的字符串空间的最低页对齐地址开始。映射的字符串空间包含共享的 `String` 以及 `字符`数组对象。所有与该映射空间重叠的 G1 区域都将被标记为固定；这些 G1 区域在运行时不可用。在部分重叠的区域中可能会有未使用的空间浪费，但最多只有一个这样的区域，位于映射的末尾。由于使用相同的窄 Oop 编码，因此不需要对字符串空间内的 oop 指针进行修补。共享字符串空间是可写的，但 GC 不应向空间中的 oops 写入，以保持跨不同进程的可共享性。尝试锁定这些共享字符串之一的应用程序，并将写入共享空间，将获得页面的私有副本，因此将失去共享该特定页面的好处。这种情况很少发生。

在运行时，共享字符串表与常规字符串表是分开的。在查找内部字符串时，都会搜索这两个表。共享字符串表在运行时是只读的；不能向其中添加或删除条目。

G1 字符串去重表是一个独立的哈希表，包含用于运行时去重的 `char` 数组。当一个字符串被内部化并添加到 `StringTable` 时，该字符串将被去重，如果它尚未存在于其中，则将底层的 `char` 数组添加到去重表中。去重表不会存储到存档中。去重表在虚拟机启动时使用共享字符串数据填充。作为一个优化，这项工作在 `G1StringDedupThread` （在 `G1StringDedupThread::run()` 之后，在 `initialize_in_thread()` 之前）完成，以减少启动时间。共享字符串的哈希值在转储时预先计算并存储在字符串中，以避免在运行时去重代码写入哈希值。

## \[UI] JEP 251：[多分辨率图像](https://openjdk.org/jeps/251)

定义一个多分辨率图像 API，以便可以轻松地操作和显示具有分辨率变体的图像。

## \[功能] JEP 252：[默认使用 CLDR 区域设置数据](https://openjdk.org/jeps/252)

### 摘要

使用通用区域数据存储库（CLDR）中的区域数据来格式化日期、时间、货币、语言、国家和时区，在标准的 Java API 中。由 Unicode 联盟维护的 CLDR 提供的区域数据质量高于 JDK 8 中的传统数据。区域敏感的应用程序可能会受到切换到 CLDR 区域数据的影响，以及未来 CLDR 区域数据的修订。

### 描述

在 JDK 8 及以后的版本中，存在两个内置的 locale 数据提供者： `JRE` ，它提供来自 20 世纪 90 年代的遗留 locale 数据，以及 `CLDR` ，它提供来自 Unicode 联盟的 CLDR locale 数据。

JDK 8 默认情况下，在运行时仅选择 `JRE` 提供者，因此与 locale 相关的 Java API 仅使用遗留 locale 数据。

JDK 9 默认情况下，将优先选择 `CLDR` 提供者，因此与 locale 相关的 Java API 将优先使用 CLDR locale 数据，而不是遗留 locale 数据。

使用 CLDR locale 数据是 JDK 9 的实现特性；它不是由 Java 平台规范所强制要求的。其他平台的实现不需要默认使用 CLDR locale 数据，甚至不需要将其作为选项提供。这种方法与其他国际化领域（如时区处理）的 Java 平台工作方式相一致（见下文）。

无论提供商如何， `US` 国家区域数据、 `ENGLISH` 语言区域数据和技术根区域数据都包含在 `java.base` 模块中；所有其他区域数据都包含在 `jdk.localedata` 模块中。使用 `jlink` 工具构建自定义运行时镜像的开发者可以通过选择包含在运行时镜像中的区域来节省空间。

**区域数据的使用位置**

应用程序使用以下类别的对象表示日期、时间、货币、语言、国家和时区：

* `java.time`: `Instant`, `LocalDate`, `LocalTime`, `LocalDateTime`, `ZonedDateTime`, `ZoneId`
* `java.util`: `Calendar`, `Currency`, `Date`, `TimeZone`

区域敏感的 API 将这些对象转换为字符串，反之亦然，以便日期、时间、货币、语言、国家或时区可以用纯文本表示。这些 API 在两个方向上使用区域数据：将对象转换为字符串（格式化），以及将字符串转换为对象（解析）。在切换到 CLDR 区域数据后，这些 API 的默认行为将发生变化。

`Calendar` 、 `Currency` 和 `TimeZone` 类在 `java.util` 包中是固有的区域敏感性的，因为它们是根据特定区域进行实例化的。它们提供了使用该特定区域数据的格式化和解析方法。相比之下， `java.util.Date` 和 `java.time` 包中的六个类不是区域敏感性的，因为它们不是根据特定区域进行实例化的。伴随类提供了它们自己的区域敏感 API，例如， `java.text.DateFormat` 类负责格式化和解析 `Date` 对象。一些通用 I/O 类也提供了格式化的区域敏感 API。以下是提供区域敏感 API 的伴随和 I/O 类：

* `java.io`: `PrintStream`, `PrintWriter`
* `java.text`: `BreakIterator`, `Collator`, `DateFormat`, `DateFormatSymbols`, `DecimalFormatSymbols`, `NumberFormat`
* `java.time.format`: `DateTimeFormatter`
* `java.util`: `Formatter`, `Scanner`

一些对本地化至关重要的 API 不是区域敏感性的，因此不受切换到 CLDR 区域数据的影响：

* `java.util.Locale` 声明了各种语言和国家的常量，例如 `ENGLISH` 语言和 `UK` 国家。这些常量及其字符串表示形式均不受切换到 CLDR 区域数据的影响。
* `java.util.ResourceBundle` 为应用程序提供区域特定的数据，但本身没有自己的格式化或解析方法。
* `java.util.Date` 具有故意不区分地区的方法 `toString()` ，以及 `java.time.LocalDate` 、 `java.time.LocalDateTime` 等相同方法。

**CLDR 地区数据如何影响应用程序**

预期使用地区敏感 API 的应用程序将看到在格式化时使用 CLDR 地区数据后的不同结果，并且在解析时可能会抛出异常（当 JDK 9 中使用 CLDR 地区数据时）。

列出所有与旧版和 CLDR 地区数据之间的差异是不切实际的，但以下是七个应用程序将看到的重要差异（列表顺序无任何意义）：

* 国家地区语言：日期组件之间的分隔符在 `JRE` 中是连字符，但在 `CLDR` 中是空格。
* 语言地区（使用英语的国家，如 `UK` ， `US` ，和 `CANADA` ）：
  * 日期和时间之间的分隔符在 `JRE` 中是空格，但在 `CLDR` 中是逗号。
  * 时区全称不同：它们在 `JRE` 中是缩写的，但在 `CLDR` 中是全称的。例如， `PDT` 在 `JRE` 中，但在 `Pacific Daylight Time` 中是 `CLDR` 。
  * 在 `JRE` 中，值 `NaN` 用 `�` （Unicode 替换字符 U+FFFD）表示，但在 `CLDR` 中用 `NaN` 表示。
* `GERMANY` 国家地区：月份的简称（除五月外）不同。它们在 `JRE` 中分别是 `Jan` 、 `Feb` 、 `Mär` 、 `Apr` 、 `Jun` 、 `Jul` 、 `Aug` 、 `Sep` 、 `Okt` 、 `Nov` 、 `Dez` ，但在 `CLDR` 中分别是 `Jan.` 、 `Feb.` 、 `März` 、 `Apr.` 、 `Juni` 、 `Juli` 、 `Aug.` 、 `Sep.` 、 `Okt.` 、 `Nov.` 、 `Dez.` 。
* `ITALY` 语言地区：货币符号（欧元）在 `JRE` 中是货币金额的前缀，但在 `CLDR` 中是后缀。
* `FRENCH` 语言地区：立陶宛语在 `JRE` 中的名称是 `lithuanien` ，但在 `CLDR` 中是 `lituanien` 。

以下是这些差异的示例：

```
System.out.println(DateFormat.getDateInstance(DateFormat.MEDIUM, Locale.UK)
                             .format(new Date()));
// JDK 8:  15-Mar-2024
// JDK 9:  15 Mar 2024

System.out.println(DateFormat.getDateTimeInstance(DateFormat.SHORT,
                                                  DateFormat.SHORT,
                                                  Locale.ENGLISH)
                             .format(new Date()));
// JDK 8:  3/19/24 2:35 PM
// JDK 9:  3/19/24, 2:35 PM

System.out.println(DateFormat.getTimeInstance(DateFormat.FULL, Locale.ENGLISH)
                             .format(new Date()));
// JDK 8:  2:27:03 PM PDT
// JDK 9:  2:27:03 PM Pacific Daylight Time

System.out.println(NumberFormat.getInstance(Locale.ENGLISH).format(Double.NaN));
// JDK 8:  �
// JDK 9:  NaN

System.out.println(new SimpleDateFormat("dd MMM", Locale.GERMANY)
                       .format(new GregorianCalendar(2024, Calendar.MARCH, 19)
                       .getTime()));
// JDK 8:  19 Mär
// JDK 9:  19 März

System.out.println(NumberFormat.getCurrencyInstance(Locale.ITALY).format(100));
// JDK 8:  € 100,00
// JDK 9:  100,00 €

System.out.println(new Locale("lt").getDisplayName(Locale.FRENCH));
// JDK 8:  lithuanien
// JDK 9:  lituanien
```

**在部署到使用默认 CLDR 区域数据集的 JDK 9 或更高版本之前，我们强烈建议您通过在 JDK 8 上运行应用程序并选择 `CLDR` 提供程序来检查兼容性问题。通过启动 Java 8 运行时来完成此操作。**

```
$ java -Djava.locale.providers=CLDR,JRE ...
```

**以确保 CLDR 区域数据优先于旧版区域数据。**

如果您的代码使用了与区域设置相关的 API，我们强烈建议您尽快对其进行修订，以与 CLDR 区域数据保持一致。与区域设置相关的 API 交互的代码必须在使用 CLDR 区域数据格式化和解析日期、时间、货币、语言、国家/地区和时间区域时正常工作。

代码的影响可能取决于日期、时间等的字符串表示是否与应用程序之外的系统进行交换或存储。例如，假设有一个应用程序需要持久化的 `Date` 对象，因此它为 `UK` 区域格式化 `Date` ，并将生成的字符串存储在数据库中。如果应用程序在同一个会话中稍后从数据库检索该字符串，并以 `UK` 区域将其解析为 `Date` ，则切换到 CLDR 区域数据不会产生影响。应用程序将获得与开始时相同的 `Date` ，因为格式化和解析都是在同一个 JDK 上，使用相同的区域数据进行的。

然而，假设应用程序在 JDK 8 上运行时将字符串存储到数据库中，但在 JDK 17 上运行时检索字符串。 `Date` 对象使用旧版区域数据格式化为字符串，但字符串将使用 CLDR 区域数据解析为 `Date` 。代码将触发 `java.text.ParseException` ，例如，连字符字符串 `"15-Mar-2024"` 与 CLDR 中用于 `UK` 日期的 `dd MMM yyyy` 模式不匹配。由于异常，应用程序可能会失败或以意外的方式运行。

除去应用程序本身的代码之外，用于测试应用程序的代码也可能受到切换到 CLDR 区域数据的影响。单元测试通常包含硬编码的日期/时间字符串，应用程序需要以区域敏感的方式解析这些字符串。如果测试是用 JDK 8 编写的，而应用程序迁移到 JDK 9 或更高版本，则测试可能会失败。

**继续使用遗留的区域数据**

如果修改代码以使用 CLDR 区域数据格式化和解析字符串不切实际，您可以采取以下三种措施继续使用遗留区域数据格式化和解析字符串：

1.  强制在启动时使用传统的区域数据来处理区域敏感的 API。通过以这种方式启动 Java 运行时来实现。

    ```
    $ java -Djava.locale.providers=JRE,CLDR ...
    ```

    系统属性值 `COMPAT` 可以用作 `JRE` 的同义词，例如 `-Djava.locale.providers=COMPAT,CLDR ...`

    **强制使用旧版区域数据必须被视为临时措施。在 JDK 9 之后的版本中，将只提供 CLDR 区域数据。**
2.  修改您的代码，使其始终使用与旧版区域数据中相同的模式格式化和解析字符串。

    例如，假设您的代码使用与区域设置相关的 `SimpleDateFormat` API 来格式化 `Date` 对象。在 JDK 8 中，代码可能如下获取 `SimpleDateFormat` ：

    ```
    SimpleDateFormat fmt
        = (SimpleDateFormat)DateFormat.getDateInstance(DateFormat.MEDIUM, Locale.UK);
    // prints "19-Mar-2024" on JDK 8 but "19 Mar 2024" on JDK 9
    System.out.println(fmt.format(new Date()));
    ```

    您可以将代码修改为直接创建一个 `SimpleDateFormat` ，并将所需的模式（日期组件由连字符分隔）传递给 `SimpleDateFormat` 的构造函数：

    ```
    SimpleDateFormat fmt = new SimpleDateFormat("dd-MMM-yyyy", Locale.UK);
    // prints "19-Mar-2024", even on JDK 9
    System.out.println(fmt.format(new Date()));
    ```

    此方案适用于小型应用程序，或者适用于存储在单例变量中的格式，并且在整个代码库中严格强制使用的较大应用程序。
3.  创建一个自定义的 locale 数据提供程序并将其包含在应用程序中。此提供程序可以覆盖 `CLDR` 提供程序，以便在格式化和解析字符串时，locale 敏感的 API 优先考虑自定义提供程序定义的模式。

    例如，以下是一个可以在 JDK 9 上使用的自定义 locale 数据提供程序，用于恢复 JDK 8 中 `UK` 日期的连字符分隔模式：

    ```
    package com.example.localization;
    import java.text.*;
    import java.text.spi.*;
    import java.util.*;

    public class HyphenatedUKDates extends DateFormatProvider {

         @Override
         public Locale[] getAvailableLocales() {
             return new Locale[]{Locale.UK};
         }

         @Override
         public DateFormat getDateInstance(int style, Locale locale) {
             assert locale.equals(Locale.UK);
             switch (style) {
                 case DateFormat.FULL:
                     return new SimpleDateFormat("EEEE, d MMMM yyyy");
                 case DateFormat.LONG:
                     return new SimpleDateFormat("dd MMMM yyyy");
                 case DateFormat.MEDIUM:
                     return new SimpleDateFormat("dd-MMM-yyyy");
                 case DateFormat.SHORT:
                     return new SimpleDateFormat("dd/MM/yy");
                 default:
                     throw new IllegalArgumentException("style not supported");
             }
         }

         @Override
         public DateFormat getDateTimeInstance(int dateStyle, int timeStyle,
                                               Locale locale)
         {
             ...
         }

         @Override
         public DateFormat getTimeInstance(int style, Locale locale) {
             ...
         }

    }
    ```

**未来对遗留区域数据的计划**

在 JDK 9 之后的某个版本中，我们将完全停止提供遗留区域数据。我们将逐步降低对遗留区域数据的支持：

* JDK 21：如果在启动时系统属性 `java.locale.providers` 的值中指定了 `JRE` 或 `COMPAT` ，则 Java 运行时将发出有关即将删除遗留区域数据的警告信息。
* JDK 23：我们将不再将遗留区域数据包含在 JDK 中。通过 `-Djava.locale.providers=...` 指定 `JRE` 或 `COMPAT` 将没有任何效果。

## \[UI] JEP 253：[准备 JavaFX UI 控件和 CSS API 以实现模块化](https://openjdk.org/jeps/253)

定义 JavaFX UI 控件和 CSS 功能性的公共 API，这些功能目前仅通过内部 API 提供，因此将因模块化而变得不可访问。

## \[功能] JEP 254：[紧凑字符串](https://openjdk.org/jeps/254)

### 摘要

采用更节省空间的字符串内部表示形式。

### 描述

我们提议将 `String` 类的内部表示从 UTF-16 `char` 数组更改为 `byte` 数组加上一个编码标志字段。新的 `String` 类将根据字符串内容存储为 ISO-8859-1/Latin-1（每个字符一个字节）或 UTF-16（每个字符两个字节）编码的字符。编码标志将指示使用哪种编码。

与 `AbstractStringBuilder`、`StringBuilder` 和 `StringBuffer` 相关的类将更新为使用相同的表示，HotSpot VM 的内置字符串操作也将如此。

这是一个纯实现变更，不会对现有公共接口进行任何更改。没有计划添加任何新的公共 API 或其他接口。

到目前为止的样机工作证实了预期的内存占用减少、垃圾收集活动的大幅减少以及在某些边缘情况中的轻微性能下降。

欲了解更多详情，请参阅：

* [字符串密度性能状态](http://cr.openjdk.java.net/~shade/density/state-of-string-density-v1.txt)
* [字符串密度对 SPARC 上 SPECjbb2005 的影响](http://cr.openjdk.java.net/~huntch/string-density/reports/String-Density-SPARC-jbb2005-Report.pdf)

### 说明

JEP 254: **Compact Strings（紧凑字符串）** 是 Java 9 引入的一项优化，旨在 **减少内存占用**，同时 **保持 `String` 类的向后兼容性** 和性能表现。

**🔍 紧凑字符串做了什么？**

在 Java 9 之前，每个 `String` 都用一个 **`char[]` 字符数组（UTF-16 编码）** 来存储文本，每个字符占用 2 个字节（即使是 ASCII 字符）。

JEP 254 的核心改动是：

> **引入了字节数组 `byte[]` 来代替 `char[]` 存储字符串内容，并增加了一个 `coder` 字段来标记编码格式（LATIN-1 或 UTF-16）**。

**🧠 新的内部结构**

```
// Java 8 之前
class String {
    private final char[] value;
}

// Java 9 之后（简化示意）
class String {
    private final byte[] value;
    private final byte coder; // 0 = LATIN1, 1 = UTF16
}
```

* 当字符串只包含 Latin-1（ISO-8859-1）字符（如英文、数字、符号等）时，使用 **LATIN1**，每个字符只需 1 个字节。
* 对于含有非 Latin-1 字符（如汉字、日文、表情符号），仍然使用 **UTF-16**（2 字节/字符）。

**🚀 带来的提升**

**✅ 1. 内存节省高达 50%（对于 ASCII/LATIN1 字符串）**

举例：

* `"Hello World"`（11 个字符）
  * Java 8：`char[11]` → 22 字节
  * Java 9：`byte[11]` + 1 字节 coder → 12 字节 ✅

对于大量英文内容（如 JSON、配置、日志字符串），可以显著减少堆内存占用。

**✅ 2. GC 压力减轻**

由于 `byte[]` 更小，堆上对象密度提高，**垃圾回收器的扫描速度更快**，对象复制代价更低。

**✅ 3. 性能基本无损甚至提升**

虽然内部逻辑多了一步判断编码方式（coder），但：

* 编码判断逻辑是 JIT 优化过的；
* 小对象更容易进入 CPU 缓存；
* 字符串拼接、比较等常用操作也进行了优化。

**✅ 4. 向后兼容**

* 所有现有的 `String` API 保持不变（`charAt()`, `length()`, `substring()` 等）
* `charAt()` 等操作会在内部根据 `coder` 解码字符

**🔬 什么时候不节省？**

对于含有多字节字符（如中文 "你好"），仍然使用 UTF-16，不会节省内存。但不会比 Java 8 更差。

**📊 总结对比**

| 特性     | Java 8 `String` | Java 9+ 紧凑字符串              |
| ------ | --------------- | -------------------------- |
| 存储结构   | `char[]`        | `byte[] + coder`           |
| 每个字符内存 | 2 字节            | 1 字节（LATIN1）或 2 字节（UTF-16） |
| 内存利用率  | 固定开销            | 动态节省                       |
| 性能     | 正常              | 持平或略优                      |
| 向后兼容   | ✅               | ✅                          |

## \[XML] JEP 255：[将选定的 Xerces 2.11.0 更新合并到 JAXP](https://openjdk.org/jeps/255)

升级 JDK 中包含的 Xerces XML 解析器的版本，以包含来自 [Xerces 2.11.0](https://xerces.apache.org/xerces2-j/) 的重要更改。

## \[UI] JEP 256：[BeanInfo 注释](https://openjdk.org/jeps/256)

### 摘要

将 `@beaninfo` Javadoc 标签替换为适当的注解，并在运行时处理这些注解以动态生成 `BeanInfo` 类。

### 动机

简化自定义 `BeanInfo` 类的创建，并使客户端库模块化。

### 描述

大多数 `BeanInfo` 类在运行时自动生成，但许多 Swing 类仍然在编译时从 `BeanInfo` 类的 `@beaninfo` Javadoc 标签生成。我们建议用以下注解替换 `@beaninfo` 标签，并扩展现有的反射算法以解释它们：

```
package java.beans;
public @interface JavaBean {
    String description() default "";
    String defaultProperty() default "";
    String defaultEventSet() default "";
}

package java.beans;
public @interface BeanProperty {
    boolean bound() default true;
    boolean expert() default false;
    boolean hidden() default false;
    boolean preferred() default false;
    boolean visualUpdate() default false;
    String description() default "";
    String[] enumerationValues() default {};
}

package javax.swing;
public @interface SwingContainer {
    boolean value() default true;
    String delegate() default "";
}
```

更多详情，请参阅 Javadoc [JavaBean](http://download.java.net/jdk9/docs/api/java/beans/JavaBean.html)， [BeanProperty](http://download.java.net/jdk9/docs/api/java/beans/BeanProperty.html)，和 [SwingContainer](http://download.java.net/jdk9/docs/api/javax/swing/SwingContainer.html).

这些注解将在运行时设置相应的功能属性。 在生成 `BeanInfo` 时，这些注解将设置相应的功能属性。这将使开发者能够直接在 Bean 类中指定这些属性，而不是为每个 Bean 类创建一个单独的 `BeanInfo` 类。这还将允许删除自动生成的类，从而更容易地对客户端库进行模块化。

## \[UI] JEP 257：[将 JavaFX/Media 更新到较新版本的 GStreamer](https://openjdk.org/jeps/257)

## \[UI] JEP 258：[HarfBuzz字体布局引擎](https://openjdk.org/jeps/258)

## \[功能] JEP 259：[堆栈遍历 API](https://openjdk.org/jeps/259)

### 摘要

定义一个高效的标准化堆栈跟踪 API，允许轻松过滤和延迟访问堆栈跟踪中的信息。

### 描述

本 JEP 将定义一个支持惰性、帧过滤、支持短路径遍历（在匹配给定标准的帧处停止）以及支持长路径遍历（遍历整个调用栈）的调用栈遍历 API。

JVM 将增强以提供一种灵活的机制来遍历和实例化所需的调用栈帧信息，并允许在需要时高效地访问额外的调用栈帧。将最小化本地 JVM 转换。实现需要有一个线程调用栈的稳定视图：返回一个包含调用指针的流以供进一步无限制地操作是不行的，因为一旦流工厂返回，JVM 就可以自由地重新组织控制栈（例如通过去优化）。这将影响 API 的定义。

该 API 将指定其在安全模式下运行时的行为，以确保对堆栈帧中`类`对象的访问不会危害安全。

提案是定义一个基于能力的 `StackWalker` API 以遍历堆栈。安全权限检查将在 `StackWalker` 对象构造时执行，而不是每次使用时执行。它将定义以下方法：

```
public <T> T walk(Function<Stream<StackFrame>, T> function);
public Class<?> getCallerClass();
```

`walk` 方法为当前线程打开一个 `StackFrame` 的顺序流，然后应用带有 `StackFrame` 流的函数。流的拆分器以有序方式执行堆栈帧遍历。`Stream<StackFrame>` 对象只能遍历一次，并在 `walk` 方法返回时关闭。一旦关闭，流就不再有效。例如，为了找到第一个调用者并过滤已知实现类列表：

```
Optional<Class<?>> frame = new StackWalker().walk((s) ->
{
    s.filter(f -> interestingClasses.contains(f.getDeclaringClass()))
     .map(StackFrame::getDeclaringClass)
     .findFirst();
});
```

要快照当前线程的堆栈跟踪，

```
List<StackFrame> stack =
     new StackWalker().walk((s) -> s.collect(Collectors.toList()));
```

`getCallerClass()` 方法是为了方便查找调用者的帧，是 `sun.reflect.Reflection.getCallerClass` 的替代品。使用 `walk` 方法获取调用者类的一种等效方法是：

```
walk((s) -> s.map(StackFrame::declaringClass).skip(2).findFirst());
```

### 说明

JEP 259: **Stack-Walking API** 引入了一个新的、**高效且可配置的堆栈遍历 API**，用于替代传统的 `Throwable::getStackTrace()` 和 `Thread::getStackTrace()`，提供了更灵活、延迟加载、按需过滤的调用栈访问方式。

***

**✅ 适用场景**

新的 Stack-Walking API 适用于：

| 场景              | 说明                             |
| --------------- | ------------------------------ |
| 🐛 **诊断与日志记录**  | 只打印部分调用栈（如最近几层）或仅打印特定包名的调用者    |
| 📉 **性能敏感的栈分析** | 想要按需、懒加载访问栈帧，避免构造完整栈数组的开销      |
| 🔒 **安全/授权检查**  | 查找调用者是否来自特定包或类                 |
| 🔄 **框架工具封装**   | 框架（如日志框架、AOP工具）中查找“真正的业务代码”调用点 |

***

**🧪 示例：打印调用链中的前几层**

```
import java.lang.StackWalker;
import java.lang.StackWalker.Option;

public class StackWalkerExample {
    public static void main(String[] args) {
        methodA();
    }

    static void methodA() {
        methodB();
    }

    static void methodB() {
        StackWalker walker = StackWalker.getInstance(Option.RETAIN_CLASS_REFERENCE);
        walker.forEach(frame -> {
            System.out.println("Class: " + frame.getClassName() + ", Method: " + frame.getMethodName());
        });
    }
}
```

输出：

```
Class: StackWalkerExample, Method: methodB
Class: StackWalkerExample, Method: methodA
Class: StackWalkerExample, Method: main
...
```

**✂️ 示例：只获取最顶层调用者**

```
StackWalker walker = StackWalker.getInstance();
String caller = walker.walk(frames ->
    frames.skip(1).findFirst().map(StackWalker.StackFrame::getClassName).orElse("unknown")
);
System.out.println("Caller: " + caller);
```

**🎛️ 可选配置项（`Option`）**

* `RETAIN_CLASS_REFERENCE`：保留 `Class<?>` 对象，可用于做反射
* `SHOW_HIDDEN_FRAMES`：包括隐藏帧（如反射调用、Lambda）
* `SHOW_REFLECT_FRAMES`（JDK 12+）：显示反射层级

**📊 与旧 API 对比**

| 特性         | `Throwable::getStackTrace()` | StackWalker（JEP 259） |
| ---------- | ---------------------------- | -------------------- |
| 是否懒加载      | ❌ 每次构造完整栈数组                  | ✅ 延迟遍历               |
| 是否可过滤      | 手动过滤                         | ✅ 通过 Stream API      |
| 是否可控制深度    | ❌ 不能                         | ✅ 可限制帧数              |
| 是否可保留类引用   | ❌ 只能拿类名                      | ✅ 可拿 `Class<?>` 对象   |
| 是否支持并行安全遍历 | ❌                            | ✅                    |

## \[功能] JEP 260：[封装大多数内部 API](https://openjdk.org/jeps/260)

### 摘要

默认情况下封装 JDK 的大部分内部 API，使其在编译时不可访问，并为未来版本做准备，届时它们将在运行时不可访问。确保关键且广泛使用的内部 API 不会被封装，以便在所有或大多数功能有支持的替代方案之前保持可访问

### 描述

基于对包括 Maven Central 在内的各种大型代码库的分析，以及自 JDK 8 及其依赖分析工具（ `jdeps` ）发布以来收到的反馈，我们将 JDK 的内部 API 分为两大类：

* **非关键内部 API**，这些 API 似乎没有被 JDK 之外的代码使用，或者只是被外部代码出于方便而使用，即用于在支持的 API 中可用的功能或可以由库轻松提供的功能（例如， `sun.misc.BASE64Decoder` ）。
* **临界内部 API 提供关键功能**，这些功能在 JDK 之外实现起来困难，甚至不可能（例如， `sun.misc.Unsafe` ）。

根据 JDK 8 中是否存在支持的替代方案，JDK 9 中是否封装临界内部 API。一个支持的替代方案是指 Java SE 8 标准的一部分，即位于 `java.*` 或 `javax.*` 包中，或者 JDK 特定的，并带有 `@jdk.Exported` 注解，通常位于 `com.sun.*` 或 `jdk.*` 包中。具体如下：

* 在 JDK 8 中存在支持的替代方案的临界内部 API 被封装在 JDK 9 中。
* 在 JDK 8 中不存在支持的替代方案的临界内部 API 没有被封装在 JDK 9 中。下面提供了一个详细列表。
* 对于 JDK 9 中存在支持的替代方案的 critical 内部 API，将弃用并在未来的版本中要么封装要么移除。

JDK 9 中所有非 critical 内部 API 都被封装。

在 JDK 9 中封装的内部 API 在编译时不可访问。可以通过 `--add-exports` 命令行选项在编译时使其可访问。在运行时，如果它们在 JDK 8 中是可访问的，它们仍然可访问；但在未来的版本中，它们将变得不可访问，此时可以使用 `--add-exports` 或 `--add-opens` 选项在运行时使它们可访问。 `--illegal-access` 选项控制这些 API 的运行时访问性，并可用于模拟内部 API 未来的运行时不可访问性。

**未在 JDK 9 中封装的 critical 内部 API**

列出这里的是那些在 JDK 9 中未封装的临界内部 API，因为这些 API 在 JDK 8 中没有支持的替代品。

* `sun.misc.{Signal,SignalHandler}`
* （本类中许多方法的函数可以通过变量句柄（JEP 193）获得。）
* （本方法的函数可以在由 JEP 259 定义的堆栈跟踪 API 中获得。）
* `sun.reflect.ReflectionFactory`
* `com.sun.nio.file.{ExtendedCopyOption,ExtendedOpenOption, ExtendedWatchEventModifier,SensitivityWatchEventModifier}`

这些 API 定义并导出在 JDK 特定的 `jdk.unsupported` 模块中。此模块存在于完整的 JRE 和 JDK 映像中。因此，这些 API 默认对类路径上的代码可访问，并且如果这些模块声明了对 `jdk.unsupported` 模块的依赖，则对模块中的代码可访问。

> **对于在 JDK 9 中引入替代方案的临界内部 API，JDK 9 中已将其弃用，并在未来的版本中将对其进行封装或删除。**

将 `sun.misc` 和 `sun.reflect` 包导出和公开的后果是，这些包中的所有非临界内部 API 要么被移动到其他包中，要么根据需要删除。不应依赖不可升级的标准和 JDK 模块，而应使用适当的内部 API。

对于使用 JDK 9 中存在替代方案的临界内部 API 的库的维护者，可能希望使用多版本 JAR 文件（JEP 238）来发布单件工件，以便在 JDK 9 之前的版本中使用旧 API，在后续版本中使用替代 API。

### 说明

JEP 260: **"Encapsulate Most Internal APIs"（封装大多数内部 API）**，从 Java 9 开始对内部 JDK API（如 `sun.*`, `com.sun.*`）进行封装，**默认不再对外开放访问**，这对**日常 Java 开发可能会带来一些影响**，尤其是依赖于内部 API 的项目或框架。

**🧱 封装了什么？**

JEP 260 把大多数 **JDK 内部实现类**（尤其是 `sun.misc.Unsafe`, `sun.reflect.*`, `com.sun.image.*` 等）做了模块级封装，非官方 API 默认不可被模块系统访问，除非显式开放。

**✅ 对日常开发的影响：视你用不用“内部 API”而定**

| 情况                               | 是否受影响      | 说明                                     |
| -------------------------------- | ---------- | -------------------------------------- |
| 👨‍💻 普通 Java 项目（只用标准库）          | ❌ 基本无影响    | 你只用 `java.*`, `javax.*` 标准 API         |
| 🧰 依赖工具库（如 Guava、Apache Commons） | 🔶 可能间接受影响 | 如果工具库内部偷偷用了 `sun.*`                    |
| ⚙️ 框架/中间件/容器（如 Netty、Spring）     | ✅ 一定要注意    | 某些功能依赖 `Unsafe`, `ReflectionFactory` 等 |
| 🧪 单元测试/Mock 框架（如 Mockito）       | ✅ 有时会受限    | 需要使用深层次反射、字段操作等                        |

**🔧 举例：常见被封装的类**

| 类名                              | 影响说明                          |
| ------------------------------- | ----------------------------- |
| `sun.misc.Unsafe`               | 低层内存操作（如 off-heap 内存），被框架广泛使用 |
| `sun.reflect.ReflectionFactory` | 深度反射构造器调用，常见于序列化框架            |
| `com.sun.image.codec.jpeg.*`    | 图像编解码，很多图像处理老代码依赖它            |
| `sun.net.www.*`                 | 有些网络代码或老工具会用它                 |

**🚨 Java 9+ 使用这些类会报错**

```
java.lang.IllegalAccessError: class MyApp tried to access class sun.misc.Unsafe
```

**🛠️ 如何临时绕过（不推荐生产使用）**

```
--add-exports java.base/sun.misc=ALL-UNNAMED
```

或：

```
--add-opens java.base/sun.reflect=ALL-UNNAMED
```

这告诉模块系统“开放内部包给未命名模块”。

**✅ 正确的应对方式**

1. **避免直接使用内部 API**
2. **找官方替代**（例如 `VarHandle` 替代 `Unsafe`；`java.util.Base64` 替代 `sun.misc.BASE64Encoder`）
3. **升级库版本**（新版本的框架通常会适配模块系统）
4. **使用 Java 模块系统的开放策略做过渡**

**📦 示例：`Base64` 替代方案**

老代码：

```
sun.misc.BASE64Encoder encoder = new sun.misc.BASE64Encoder();
String encoded = encoder.encode(bytes);
```

现代替代：

```
String encoded = java.util.Base64.getEncoder().encodeToString(bytes);
```

**🔚 总结**

* 对 **日常 Java 程序员**，JEP 260 的影响 **不大**
* 对 **底层框架开发者**、**工具链维护者**，JEP 260 是一记警钟：**不要依赖内部 API**
* **长远来看**，这是 Java 模块化、安全性、稳定性的关键一步

## \[功能] JEP 261：[模块系统](https://openjdk.org/jeps/261)

### 摘要

实施 Java 平台模块系统，该系统由 [JSR 376](http://openjdk.java.net/projects/jigsaw/spec/) 规定，并包括相关的 JDK 特定更改和增强。

### 描述

Java 平台模块系统（JSR 376）指定了对 Java 编程语言、Java 虚拟机和标准 Java API 的更改和扩展。本 JEP 实现了该规范。因此， `javac` 编译器、HotSpot 虚拟机和运行时库将模块作为 Java 程序组件的一种基本新类型实现，并在开发的所有阶段提供模块的可靠配置和强封装。

本 JEP 还更改、扩展和添加了与编译、链接和执行相关的 JDK 特定工具和 API，这些工具和 API 超出了 JSR 的范围。对其他工具和 API 的关联更改，例如 `javadoc` 工具和 Doclet API，是其他 JEP 的主题。

本 JEP 假定读者熟悉最新的模块系统状态文档以及其他 Project Jigsaw JEP。

* [200: JDK 9 的模块化](http://openjdk.java.net/jeps/200)
* [201: 模块化源代码](http://openjdk.java.net/jeps/201)
* [220: 模块化运行时镜像](http://openjdk.java.net/jeps/220)
* [260: 封装大部分内部 API](http://openjdk.java.net/jeps/260)
* [282: jlink：Java 链接器](http://openjdk.java.net/jeps/282)

**阶段**

在编译时间（ `javac` 命令）和运行时间（ `java` 运行时启动器）的熟悉阶段之间，我们增加了链接时间的概念，这是一个可选的阶段，在两个阶段之间，可以将一组模块组装并优化成自定义运行时镜像。链接工具 `jlink` 是 JEP 282 的主题； `javac` 和 `java` 实现的大多数新命令行选项也由 `jlink` 实现。

**模块路径**

`javac` 、 `jlink` 和 `java` 命令，以及其他一些命令，现在可以接受选项来指定各种模块路径。模块路径是一个序列，其中每个元素要么是一个模块定义，要么是一个包含模块定义的目录。每个模块定义要么是

* 模块工件，即包含编译模块定义的模块化 JAR 文件或 JMOD 文件，或者
* 展开的模块目录，其名称按照惯例是模块的名称，其内容是与包层次结构相对应的“展开”的目录树。

在后一种情况下，目录树可以是编译后的模块定义，其中包含单个类和资源文件以及根目录下的 `module-info.class` 文件，或者在编译时，是源模块定义，其中包含单个源文件以及根目录下的 `module-info.java` 文件。

模块路径，与其他类型的路径一样，由一系列路径名称组成，这些名称由主机平台的路径分隔符字符分隔（在大多数平台上为 `':'` ，在 Windows 上为 `';'` ）。

模块路径与类路径非常不同：类路径是一种定位单个类型和资源定义的手段，而模块路径是一种定位整个模块定义的手段。类路径的每个元素都是一个类型和资源定义的容器，即一个 JAR 文件或一个展开的、包层次结构的目录树。相比之下，模块路径的每个元素都是一个模块定义或一个目录，其中目录中的每个元素都是一个模块定义，即一个类型和资源定义的容器，即一个模块化的 JAR 文件、一个 JMOD 文件或一个展开的模块目录。

在解析过程中，模块系统根据阶段搜索多个不同的路径来定位模块，并且还会搜索环境内编译的内置模块，按照以下顺序：

* 编译模块路径（由命令行选项 `--module-source-path` 指定）包含源形式的模块定义（仅编译时）。
* 升级模块路径（ `--upgrade-module-path` ）包含旨在优先于系统模块或应用模块路径上现有可升级模块的编译定义的模块（编译时和运行时）。
* 系统模块是环境内构建的编译模块（编译时和运行时）。这些通常包括 Java SE 和 JDK 模块，但在自定义链接图像的情况下，也可以包括库和应用模块。在编译时，可以通过 `--system` 选项覆盖系统模块，该选项指定用于加载系统模块的 JDK 图像。
* 应用程序模块路径（ `--module-path` ，或简称 `-p` ）包含库模块和应用模块（所有阶段）的编译定义。在链接时，此路径还可以包含 Java SE 和 JDK 模块。

这些路径上现有的模块定义，以及系统模块，定义了可观察模块的宇宙。

在搜索特定名称的模块路径时，模块系统采用该名称的第一个模块定义。如果存在版本字符串，则忽略；如果模块路径的元素包含多个具有相同名称的模块定义，则解析失败，编译器、链接器或虚拟机将报告错误并退出。配置模块路径以避免版本冲突是构建工具和容器应用程序的责任；模块系统的目标不是解决版本选择问题。

**根模块**

模块系统通过解析一组根模块相对于可观察模块集合的传递闭包，构建模块图。

当编译器编译无名称模块中的代码，或者调用 `java` 启动器并将应用程序的主类从类路径加载到应用程序类加载器的无名称模块中时，JDK 9 中计算无名称模块的默认根模块集如下：

* `java.se` 模块是根模块，如果存在的话。如果不存在，则升级模块路径上的每个 `java.*` 模块或系统模块中至少导出一个未加限定符的包的模块也是根模块。
* 升级模块路径上的每个非 `java.*` 模块或系统模块中至少导出一个未加限定符的包的模块也是根模块。

_更新，2018 年 6 月：在 JDK 11 中，未命名模块的默认根模块集合已更改。默认集合现在计算如下：_

* _在升级模块路径上的每个模块或导出至少一个包的系统模块，无需指定，都是根模块。_

_`java.se` 模块在 JDK 11 及以后的版本中仍然存在，但它不再是根模块。_

否则，默认的根模块集合取决于阶段：

* 在编译时通常是正在编译的模块集（下面将详细介绍）；
* 在链接时它是空的；并且；
* 在运行时它是应用程序的主模块，通过 `--module` （或简称为 `-m` ）启动器选项指定。

有时有必要将模块添加到默认根集，以确保特定的平台、库或服务提供者模块将存在于生成的模块图中。在任何阶段，此选项

```
--add-modules <module>(,<module>)*
```

当 `<module>` 是一个模块名称时，将指示的模块添加到默认的根模块集合中。此选项可以多次使用。

在运行时，作为一个特殊情况，如果 `<module>` 是 `ALL-DEFAULT` ，则默认的根模块集合（如上所述）将添加到根集合中。这对于应用程序是一个容器，可以托管其他应用程序，而这些应用程序反过来又依赖于容器本身不需要的模块是有用的。

作为运行时的另一个特殊情况，如果 `<module>` 是 `ALL-SYSTEM` ，则无论它们是否在默认集中，所有系统模块都将添加到根集。这有时是测试工具所必需的。此选项会导致许多模块被解析；通常应首选 `ALL-DEFAULT` 。

作为最后的特殊情况，在运行时和链接时，如果 `<module>` 是 `ALL-MODULE-PATH` ，则将所有在相关模块路径上找到的可观察模块添加到根集。 `ALL-MODULE-PATH` 在编译时和运行时都有效。这是为构建工具（如 Maven）提供的，这些工具已经确保了模块路径上的所有模块都是必需的。这也是将自动模块添加到根集的便捷方式。

### 限制可观察模块

有时限制可观察的模块对于调试或减少主模块（由应用程序类加载器为类路径定义的无名模块）解析的模块数量是有用的。可以在任何阶段使用 `--limit-modules` 选项来完成此操作。其语法如下：

```
--limit-modules <module>(,<module>)*
```

其中 `<module>` 是模块名称。此选项的效果是将可观察的模块限制在命名模块的传递闭包中，如果有的话，还包括主模块以及通过 `--add-modules` 选项指定的任何其他模块。

(为解释 `--limit-modules` 选项而计算的传递闭包是一个临时结果，仅用于计算有限的可观察模块集。将再次调用解析器以计算实际的模块图。)

**提高可读性**

在测试和调试过程中，有时需要安排一个模块读取另一个模块，即使第一个模块在其模块声明中没有通过 `requires` 子句依赖于第二个模块。这可能需要，例如，使被测试的模块能够访问测试框架本身，或者访问与框架相关的库。可以在编译时和运行时使用 `--add-reads` 选项来完成此操作。其语法如下：

```
--add-reads <source-module>=<target-module>
```

其中 `<source-module>` 和 `<target-module>` 是模块名称。

`--add-reads` 选项可以多次使用。每个实例的效果是在源模块和目标模块之间添加一个可读性边缘。这本质上是一种模块声明中 `requires` 子句的命令行形式，或者是对 `Module::addReads` 方法的无限制形式调用。因此，源模块中的代码如果目标模块的包通过源模块声明中的 `exports` 子句、 `Module::addExports` 方法的调用或 `--add-exports` 选项（如下定义）导出，则可以在编译时和运行时访问该包中的类型。此外，如果该模块被声明为开放或通过源模块声明中的 `opens` 子句、 `Module::addOpens` 方法的调用或 `--add-opens` 选项（如下定义）打开该包，则该代码还可以在运行时访问目标模块包中的类型。

例如，如果测试框架将白盒测试类注入到 `java.management` 模块中，并且该类扩展了（假设的） `testng` 模块中的导出实用工具类，那么可以通过该选项授予所需的访问权限

```
--add-reads java.management=testng
```

作为特殊情况，如果 `<target-module>` 是 `ALL-UNNAMED` ，则将从源模块添加可读性边缘到所有现有和未来的未命名模块，包括对应于类路径的模块。这允许模块中的代码被尚未转换为模块形式的测试框架测试。

**破坏封装**

有时有必要违反由模块系统定义并由编译器和虚拟机强制执行的访问控制边界，以便允许一个模块访问另一个模块的一些未导出类型。这可能出于以下目的而成为必需，例如，启用内部类型的白盒测试，或将不受支持的内部 API 暴露给已经依赖它们的代码。可以在编译时和运行时使用 `--add-exports` 选项来实现这一点。其语法如下：

```
--add-exports <source-module>/<package>=<target-module>(,<target-module>)*
```

其中 `<source-module>` 和 `<target-module>` 是模块名称， `<package>` 是包的名称。

`--add-exports` 选项可以多次使用，但对于任何特定的源模块和包名组合，最多只能使用一次。每个实例的效果是将命名包从源模块导出到目标模块的合格导出。这本质上是一种模块声明中的 `exports` 子句的命令行形式，或者是对 `Module::addExports` 方法的非限制性调用。因此，如果目标模块通过其模块声明中的 `requires` 子句、 `Module::addReads` 方法的调用或 `--add-reads` 选项的实例读取源模块，则目标模块将能够访问源模块命名包中的公共类型。

如果，例如，模块 `jmx.wbtest` 包含对模块 `java.management` 中未导出的 `com.sun.jmx.remote.internal` 包的白色盒测试，那么可以通过选项授予它所需访问权限

```
--add-exports java.management/com.sun.jmx.remote.internal=jmx.wbtest
```

作为特殊情况，如果 `<target-module>` 是 `ALL-UNNAMED` ，则源包将被导出到所有未命名的模块，无论它们最初是否存在或后来创建。因此，可以通过选项将 `sun.management` 包的访问权限授予类路径上的所有代码。

```
--add-exports java.management/sun.management=ALL-UNNAMED
```

“ `--add-exports` ”选项允许访问指定包的公共类型。有时需要进一步操作，通过核心反射 API 的“ `setAccessible` ”方法启用对所有非公共元素的访问。可以在运行时使用“ `--add-opens` ”选项来完成此操作。它与“ `--add-exports` ”选项具有相同的语法：

```
--add-opens <source-module>/<package>=<target-module>(,<target-module>)*
```

其中 `<source-module>` 和 `<target-module>` 是模块名称， `<package>` 是包的名称。

“ `--add-opens` ”选项可以多次使用，但对于任何特定的源模块和包名组合，最多只能使用一次。每个实例的效果是将源模块中命名的包的合格打开添加到目标模块中。这本质上是一种模块声明中“ `opens` ”子句的命令行形式，或者是对“ `Module::addOpens` ”方法的非限制性调用。因此，只要目标模块读取源模块，目标模块中的代码就可以使用核心反射 API 访问源模块中命名的包中的所有类型，包括公共和非公共类型。

开放包在编译时与非导出包不可区分，因此在该阶段不能使用 `--add-opens` 选项。

> **`--add-exports` 和 `--add-opens` 选项必须谨慎使用。您可以使用它们来访问库模块或甚至 JDK 本身的内部 API，但这样做存在风险：如果该内部 API 被更改或删除，则您的库或应用程序将失败。**

**修补模块内容**

在测试和调试时，有时需要用替代或实验性版本替换特定模块的选定的类文件或资源，或者提供全新的类文件、资源，甚至包。这可以通过 `--patch-module` 选项来实现，无论是在编译时还是在运行时。其语法如下：

```
--patch-module <module>=<file>(<pathsep><file>)*
```

`<module>` 是模块名称， `<file>` 是模块定义的文件系统路径名称， `<pathsep>` 是主机平台的路径分隔符字符。

`--patch-module` 选项可以多次使用，但对于任何特定的模块名称，最多只能使用一次。每个实例的效果是改变模块系统在指定模块中搜索类型的方式。在检查实际模块之前（无论该模块是否为系统的一部分或定义在模块路径上），它首先按顺序检查选项中指定的每个模块定义。补丁路径命名了一组模块定义，但它不是一个模块路径，因为它具有类似类路径的泄漏语义。这允许测试工具，例如，在不将所有测试复制到单个目录的情况下，将多个测试注入到同一个包中。

`--patch-module` 选项不能用来替换 `module-info.class` 文件。如果在补丁路径上的模块定义中找到一个 `module-info.class` 文件，则会发出警告，并忽略该文件。

如果在补丁路径上的模块定义中找到一个包，而这个模块尚未导出或打开该包，那么它仍然不会被导出或打开。可以通过反射 API 或 `--add-exports` 或 `--add-opens` 选项显式导出或打开。

`--patch-module` 选项取代了 `-Xbootclasspath:/p` 选项，后者已被移除（见下文）。

> **`--patch-module` 选项仅适用于测试和调试，在生产环境中使用该选项被强烈建议不要使用。**

**编译时**

`javac` 编译器实现了上述描述的可用于编译时选项： `--module-source-path` ， `--upgrade-module-path` ， `--system` ， `--module-path` ， `--add-modules` ， `--limit-modules` ， `--add-reads` ， `--add-exports` ，和 `--patch-module` 。

编译器运行在三种模式之一，每种模式都实现了额外的选项。

* 当编译环境（由 `-source` 、 `-target` 和 `--release` 选项定义）小于或等于 8 时，将启用传统模式。上述描述的任何模块化选项均不能使用。

在遗留模式下，编译器的行为基本上与 JDK 8 中的行为相同。

* 当编译环境为 9 或更高版本且未使用 `--module-source-path` 选项时，将启用单模块模式。可以使用上述描述的其他模块选项；现有的选项 `-bootclasspath` 、 `-Xbootclasspath` 、 `-extdirs` 、 `-endorseddirs` 和 `-XXuserPathsFirst` 不能使用。

单模块模式用于编译组织在传统的包分层目录树中的代码。它是遗留模式简单使用的自然替代品。

```
$ javac -d classes -classpath classes -sourcepath src Foo.java
```

如果在命令行中指定了形式为 `module-info.java` 或 `module-info.class` 的模块描述符文件，或者该文件在源路径或类路径中找到，则源文件将被编译为该描述符命名的模块的成员，并且该模块将是唯一的根模块。否则，如果存在 `--module <module>` 选项，则源文件将被编译为 `<module>` 的成员，该模块将是根模块。否则，源文件将被编译为无名称模块的成员，根模块的计算方式如上所述。

在此模式下，可以将任意类和 JAR 文件放在类路径上，但这样做并不推荐，因为这相当于将这些类和 JAR 文件视为编译模块的一部分。

* 多模块模式在编译环境为 9 或更高版本且使用 `--module-source-path` 选项时启用。必须同时使用现有的 `-d` 选项来命名输出目录；可以同时使用上述描述的其他模块选项；现有的 `-bootclasspath` 、 `-Xbootclasspath` 、 `-extdirs` 、 `-endorseddirs` 和 `-XXuserPathsFirst` 选项不能使用。

多模块模式用于编译一个或多个模块，这些模块的源代码位于模块源路径上的展开模块目录中。在此模式下，类型的模块成员资格由其源文件在模块源路径中的位置确定，因此命令行上指定的每个源文件必须存在于该路径的某个元素中。根模块集是至少指定了一个源文件的模块集合。

与其他模式不同，在此模式下必须通过 `-d` 选项指定输出目录。输出目录将作为模块路径的一个元素进行结构化，即它将包含展开的模块目录，这些目录本身包含类和资源文件。如果编译器在模块源路径上找到一个模块，但无法找到该模块中某些类型的源文件，则它将在输出目录中搜索相应的类文件。

在大型系统中，特定模块的源代码可能分布在几个不同的目录中。例如，在 JDK 本身中，模块的源文件可能位于 `src/<module>/share/classes` 、 `src/<module>/<os>/classes` 或 `build/gensrc/<module>` 中的任何一个目录中，其中 `<os>` 是目标操作系统的名称。为了在模块源路径中表达这一点，同时保留模块标识，我们允许路径的每个元素使用花括号（ `{` 和 `}` ）来包围逗号分隔的选项列表，并使用单个星号（ `*` ）来表示模块名称。然后，JDK 的模块源路径可以写成如下形式：

```
{src/*/{share,<os>}/classes,build/gensrc/*}
```

在两种模块模式下，编译器默认会生成与模块系统相关的各种警告；这些警告可以通过选项 `-Xlint:-module` 禁用。更精确地控制这些警告，可以通过 `exports` 、 `opens` 、 `requires-automatic` 和 `requires-transitive-automatic` 键来控制 `-Xlint` 选项。

新的选项 `--module-version <version>` 可以用来指定正在编译的模块的版本字符串。

**类文件属性**

一个特定于 JDK 的类文件属性 `ModuleTarget` ，可选地记录包含它的模块描述符的目标操作系统和架构。其格式为：

```
ModuleTarget_attribute {
    u2 attribute_name_index;
    u4 attribute_length;
    u2 os_arch_index; // index to a CONSTANT_utf8_info structure
}
```

常量池中的 UTF-8 字符串在 `os_arch_index` 处的格式为 `<os>-<arch>` ，其中 `<os>` 通常是以下之一： `linux` 、 `macos` 、 `solaris` 或 `windows` ，而 `<arch>` 通常是以下之一： `x86` 、 `amd64` 、 `sparcv9` 、 `arm` 或 `aarch64` 。

**打包：模块化 JAR 文件**

`jar` 工具无需修改即可用于创建模块化 JAR 文件，因为模块化 JAR 文件只是一个在其根目录中包含 `module-info.class` 文件的 JAR 文件。

`jar` 工具实现了以下新选项，以便在打包模块时将附加信息插入模块描述符中：

* `--main-class=<class-name>` 或 `-e <class-name>` 简称，将 `<class-name>` 记录在 `module-info.class` 文件中作为模块的 `public static void main` 入口类。（这不是一个新选项；它已经记录了 JAR 文件清单中的主类。）
* `--module-version=<version>` 将 `<version>` 记录在 `module-info.class` 文件中作为模块的版本字符串。
* `--hash-modules=<pattern>` 将依赖于该模块的特定模块的内容哈希记录在 `module-info.class` 文件中，供后续依赖项验证使用。只有名称与正则表达式 `<pattern>` 匹配的模块的哈希会被记录。如果使用此选项，则必须同时使用 `---module-path` 选项，或简称 `-p` ，以指定用于计算依赖于该模块的模块的观察模块集合。
* `--describe-module` 或 `-d` 简称，显示指定 JAR 文件的模块描述符（如果有）。

该 `jar` 工具的 `--help` 选项可以用来显示其命令行选项的完整摘要。

定义了两个新的针对 JDK 的 JAR 文件清单属性，以对应 `--add-exports` 和 `--add-opens` 命令行选项：

* `Add-Exports: <module>/<package>( <module>/<package>)*`
* `Add-Opens: <module>/<package>( <module>/<package>)*`

每个属性的值是一个由空格分隔的斜杠分隔的模块名/包名对列表。 `<module>/<package>` 属性值中的对与命令行选项 `Add-Exports` 的含义相同。 `--add-exports <module>/<package>=ALL-UNNAMED` 属性值中的对与命令行选项 `<module>/<package>` 的含义相同。

每个属性最多只能出现一次，位于 `MANIFEST.MF` 文件的主节中。特定的对可以出现多次。如果指定的模块未解析，或者指定的包不存在，则相应的对将被忽略。这些属性仅在应用程序的主可执行 JAR 文件中解释，即 Java 运行时启动器的 `-jar` 选项指定的 JAR 文件中；在其他所有 JAR 文件中将被忽略。

**打包：JMOD 文件**

新的 JMOD 格式不仅超越了 JAR 文件，还包括原生代码、配置文件和其他不适合或根本不适合 JAR 文件的数据。JMOD 文件用于打包 JDK 自身的模块；如果需要，开发人员也可以使用它们来打包自己的模块。

JMOD 文件可以在编译时和链接时使用，但不能在运行时使用。要在运行时支持它们，通常需要我们准备好即时提取和链接原生代码库。这在大多数平台上是可行的，尽管这可能非常复杂，而且我们没有看到很多需要这种功能的用例，因此为了简单起见，我们选择限制本版本中 JMOD 文件的功能。

可以使用一个新的命令行工具 `jmod` 来创建、操作和检查 JMOD 文件。其通用语法如下：

```
$ jmod (create|extract|list|describe|hash) <options> <jmod-file>
```

对于 `create` 子命令， `<options>` 可以包括上述 `jar` 工具的 `--main-class` 、 `--module-version` 、 `--hash-modules` 和 `---module-path` 选项，并且还可以：

* `--class-path <path>` 指定一个类路径，其内容将被复制到生成的 JMOD 文件中。
* `--cmds <path>` 指定一个或多个包含要复制的本地命令的目录。
* `--config <path>` 指定一个或多个包含要复制的配置文件的目录。
* 指定要排除的文件，其中 `<pattern-list>` 是一个逗号分隔的模式列表，形式为 `<glob-pattern>` 、 `glob:<glob-pattern>` 或 `regex:<regex-pattern>` 。
* 指定包含要复制的 C 和 C++头文件的目录。
* 指定包含要复制的法律声明的目录。
* 指定包含要复制的本地库的目录。
* 指定一个或多个包含要复制的手册页的目录。
* 指定目标操作系统和架构，并将其记录在 `module-info.class` 文件的 `ModuleTarget` 属性中。

`extract` 子命令接受一个选项 `--dir` ，以指示应将指定 JMOD 文件的内容写入的目录。如果该目录不存在，则会创建它。如果此选项不存在，则内容将提取到当前目录。

`list` 子命令列出指定 JMOD 文件的内容； `describe` 子命令显示指定 JMOD 文件的模块描述符，格式与 `jar` 和 `java` 命令的 `--describe-module` 选项相同。这些子命令不接受任何选项。

可以使用 `hash` 子命令来对现有的 JMOD 文件集进行哈希处理。它需要同时使用 `--module-path` 和 `--hash-modules` 选项。

可以使用 `jmod` 工具的 `--help` 选项来显示其命令行选项的完整摘要。

**链接时间**

命令行链接工具 `jlink` 的详细信息在 JEP 282 中描述。从高层次来看，其一般语法如下：

```
$ jlink <options> ---module-path <modulepath> --output <path>
```

`---module-path` 选项指定了链接器要考虑的可观察模块集合，而 `--output` 选项指定了将包含结果运行时图像的目录路径。其他 `<options>` 可以包括上面描述的 `---limit-modules` 和 `---add-modules` 选项，以及额外的链接器特定选项。

可以使用 `jlink` 工具的 `--help` 选项来显示其命令行选项的完整摘要。

**运行时**

HotSpot 虚拟机实现了适用于运行时的上述选项： `--upgrade-module-path` 、 `--module-path` 、 `--add-modules` 、 `--limit-modules` 、 `--add-reads` 、 `--add-exports` 、 `--add-opens` 和 `--patch-module` 。这些选项可以传递给命令行启动器 `java` ，也可以传递给 JNI 调用 API。

本阶段特有的附加选项，由启动器支持的是：

* `--module <module>` 或简写为 `-m <module>` ，指定了模块化应用程序的主模块。这将是构建应用程序初始模块图的默认根模块。如果主模块的描述符未指示主类，则可以使用语法 `<module>/<class>` ，其中 `<class>` 指定包含应用程序 `public static void main` 入口点的类。

启动器支持的附加诊断选项包括：

* `--list-modules` 显示可观察模块的名称和版本字符串，然后退出，与 `java --version` 的方式相同。
* `--describe-module <module>` 显示指定模块的模块描述符，格式与 `jar -d` 选项和 `jmod describe` 子命令相同，然后退出。
* `--validate-modules` 验证所有可观察模块，检查冲突和其他潜在错误，然后退出。
* `--dry-run` 初始化虚拟机并加载主类，但不调用主方法；这对于验证模块系统的配置很有用。
* `--show-module-resolution` 在构建初始模块图时，使模块系统描述其活动。
* 当 API 中的访问检查失败并抛出异常或错误时，将显示线程转储。这在调试时很有用，因为失败的根本原因可能被隐藏，因为异常被捕获而没有被重新抛出。
* 当模块在运行时模块图中定义和更改时，将导致虚拟机记录调试或跟踪消息。这些选项在启动期间会生成大量输出。
* 这是 `-Xlog:module+load -Xlog:module+unload` 的缩写。
* 如果模块系统的初始化失败，将显示堆栈跟踪。
* `--version` 、 `--show-version` 、 `--help` 和 `--help-extra` 显示相同的信息，并且分别以与现有的 `-version` 、 `-show-version` 、 `-help` 和 `-Xhelp` 选项相同的方式工作，只是它们将帮助文本写入标准输出流而不是标准错误流。

运行时生成的异常堆栈跟踪已扩展，包括相关模块的名称和版本字符串（如有）。异常的详细字符串（如 `ClassCastException` 、 `IllegalAccessException` 和 `IllegalAccessError` ）也已更新，包括模块信息。

现有的 `-jar` 选项已得到增强，如果正在启动的 JAR 文件的清单文件包含 `Launcher-Agent-Class` 属性，则 JAR 文件将以应用程序和该应用程序的代理两种方式启动。这允许使用 `java -jar foo.jar` 代替更冗长的 `java -javaagent:foo.jar -jar foo.jar` 。

**弱封装**

在本版本中，根据 Java SE 9 平台规范，默认情况下放松了 JDK 部分包的强封装。这种放松由一个新的启动选项 `--illegal-access` 控制，其工作方式如下：

*   在运行时图像中打开每个模块中的每个包，以便在所有未命名的模块中编写代码，即在对类路径上的代码进行编码，如果该包存在于 JDK 8 中。这既支持静态访问，即通过编译的字节码，也支持通过平台的各个反射 API 进行深度反射访问。

    对此类包的第一次反射访问操作将发出警告，但之后不再发出警告。此单个警告描述了如何启用进一步的警告。此警告无法被抑制。

    此模式是 JDK 9 的默认模式。它将在未来的版本中逐步淘汰，并最终被移除。
* `--illegal-access=warn` 与 `permit` 相同，但会对每个非法反射访问操作发出警告。
* `--illegal-access=debug` 与 `warn` 相同，但会对每个非法反射访问操作发出警告和堆栈跟踪。
*   `--illegal-access=deny` 禁用所有非法访问操作，除非由其他命令行选项启用，例如 `--add-opens` 。

    此模式将在未来的版本中成为默认模式。

当 `deny` 成为默认非法访问模式时， `permit` 可能会至少在一段时间内继续得到支持，以便开发者可以继续迁移他们的代码。随着时间的推移， `permit` 、 `warn` 、 `debug` 模式以及该选项本身都将被移除。（为了与启动脚本兼容，不支持的模式很可能会被忽略，同时会发出相应的警告。）

默认模式， `--illegal-access=permit` ，旨在让您在类路径上有代码至少一次反射访问某些 JDK 内部 API 时意识到。为了为未来做准备，您可以使用 `warn` 或 `debug` 模式来了解所有此类访问。对于类路径上需要非法访问的每个库或框架，您有两个选择：

* 如果组件的维护者已经发布了不再使用 JDK 内部 API 的新版本，那么您可以考虑升级到该版本。
* 如果组件仍然需要修复，我们鼓励您联系其维护者，并要求他们用适当的导出 API 替换对 JDK 内部 API 的使用，如果可用的话。

如果您必须继续使用需要非法访问的组件，那么您可以通过使用一个或多个 `--add-opens` 选项来仅打开那些需要访问的内部包，从而消除警告信息。

为了验证您的应用程序是否为未来做好准备，请使用 `--illegal-access=deny` 运行它，并附带任何必要的 `--add-opens` 选项。任何剩余的非法访问错误很可能是由于编译代码对 JDK 内部 API 的静态引用造成的。您可以通过运行带有 `--jdk-internals` 选项的 `jdeps` 工具来识别这些错误。（运行时系统不会对非法静态访问操作发出警告，因为这需要深入 VM 变更并降低性能。）

当检测到非法反射访问操作时发出的警告消息具有以下形式：

```
WARNING: Illegal reflective access by $PERPETRATOR to $VICTIM
```

where:

* $PERPETRATOR 是包含调用所涉及反射操作的代码的类型的全限定名称，如果可用，还包括代码源（即 JAR 文件路径），
* $VICTIM 是描述被访问成员的字符串，包括封装类型的全限定名称

在默认模式下， `--illegal-access=permit` ，最多只会发出这些警告消息之一，并伴随额外的指导性文本。以下是一个从运行 Jython 得到的示例：

```
$ java -jar jython-standalone-2.7.0.jar
WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by jnr.posix.JavaLibCHelper (file:/tmp/jython-standalone-2.7.0.jar) to method sun.nio.ch.SelChImpl.getFD()
WARNING: Please consider reporting this to the maintainers of jnr.posix.JavaLibCHelper
WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
WARNING: All illegal access operations will be denied in a future release
Jython 2.7.0 (default:9987c746f838, Apr 29 2015, 02:25:11) 
[OpenJDK 64-Bit Server VM (Oracle Corporation)] on java9
Type "help", "copyright", "credits" or "license" for more information.
>>> ^D
```

运行时系统会尽力抑制对同一$PERPETRATOR 和$VICTIM 的重复警告。

**扩展示例**

假设我们有一个应用程序模块， `com.foo.bar` ，它依赖于一个库模块， `com.foo.baz` 。如果我们有两个模块的源代码都在模块路径目录 `src` 中：

```
src/com.foo.bar/module-info.java
src/com.foo.bar/com/foo/bar/Main.java
src/com.foo.baz/module-info.java
src/com.foo.baz/com/foo/baz/BazGenerator.java
```

那么我们可以一起编译它们：

```
$ javac --module-source-path src -d mods $(find src -name '*.java')
```

输出目录， `mods` ，是一个模块路径目录，包含两个模块的展开、编译定义：

```
mods/com.foo.bar/module-info.class
mods/com.foo.bar/com/foo/bar/Main.class
mods/com.foo.baz/module-info.class
mods/com.foo.baz/com/foo/baz/BazGenerator.class
```

假设 `com.foo.bar.Main` 类包含应用程序的入口点，我们可以直接运行这些模块：

```
$ java -p mods -m com.foo.bar/com.foo.bar.Main
```

或者，我们可以将它们打包成模块化的 JAR 文件：

```
$ jar --create -f mlib/com.foo.bar-1.0.jar \
      --main-class com.foo.bar.Main --module-version 1.0 \
      -C mods/com.foo.bar .
$ jar --create -f mlib/com.foo.baz-1.0.jar \
      --module-version 1.0 -C mods/com.foo.baz .
```

`mlib` 目录是一个模块路径目录，其中包含两个模块打包、编译后的定义：

```
$ ls -l mlib
-rw-r--r-- 1501 Sep  6 12:23 com.foo.bar-1.0.jar
-rw-r--r-- 1376 Sep  6 12:23 com.foo.baz-1.0.jar
```

现在我们可以直接运行打包的模块：

```
$ java -p mlib -m com.foo.bar
```

**`jtreg` 增强**

jtreg 测试工具支持一个新的声明性标签， `@modules` ，用于表达测试对正在测试的系统模块的依赖。它接受一系列由空格分隔的参数，每个参数可以是以下形式：

* `<module>` ，其中 `<module>` 是模块名称，表示必须存在指定的模块；
* `<module>/<package>` ，表示必须存在指定的模块，并且指定的包必须导出到测试模块；或者
* `<module>/<package>:<flag>` ，表示必须存在指定的模块，如果标志为 `open` 则指定的包必须对测试模块开放，否则如果标志为 `+open` 则指定的包必须既导出又对测试模块开放。

可以指定一个默认的 `@modules` 参数集，该参数集将用于所有不包含此类标记的目录层次结构中的所有测试，可以指定为 `modules` 属性的值在 `TEST.ROOT` 文件中或在任何 `TEST.properties` 文件中。

现有的 `@compile` 标签接受一个新选项， `/module=<module>` 。这会以 `--module <module>` 选项（如上所述）调用 `javac` ，将指定的类编译为指示模块的成员。

**类加载器**

Java SE 平台 API 历史上指定了两种类加载器：引导类加载器，它从引导类路径加载类，以及系统类加载器，它是新类加载器的默认委托父类加载器，通常是用于加载和启动应用程序的类加载器。规范没有强制规定这两个类加载器的具体类型，也没有规定它们的确切委托关系。

自 1.2 版本以来，JDK 实现了一个由三个级别的类加载器组成的层次结构，其中每个加载器将委托给下一个加载器：

* 应用程序类加载器，一个实例为 `java.net.URLClassLoader` ，从类路径加载类，除非通过系统属性 `java.system.class.loader` 指定其他系统加载器，否则作为系统类加载器安装。
* 扩展类加载器，也是一个 `URLClassLoader` 的实例，加载通过扩展机制可用的类，以及 JDK 内置的一些资源和服务提供者。（此加载器在 Java SE 平台 API 规范中未明确提及。）
* 引导类加载器，仅由虚拟机内部实现，并在 `ClassLoader` API 中以 `null` 表示，从引导类路径加载类。

JDK 9 保留了这一三级层次结构，以保持兼容性，同时为了实现模块系统，进行了以下更改：

* 应用程序类加载器不再是 `URLClassLoader` 的实例，而是内部类的实例。它是默认加载器，用于加载既不是 Java SE 模块也不是 JDK 模块的命名模块。
* 扩展类加载器不再是 `URLClassLoader` 的实例，而是一个内部类的实例。它不再通过扩展机制加载类，该机制已被 JEP 220 移除。然而，它确实定义了选定的 Java SE 和 JDK 模块，下面将详细介绍。在这个新角色中，这个加载器被称为平台类加载器，它可以通过新的 `ClassLoader::getPlatformClassLoader` 方法访问，并且将被 Java SE 平台 API 规范所要求。
* 引导类加载器在库代码和虚拟机内部都得到了实现，但由于兼容性考虑，在 `ClassLoader` API 中仍然用 `null` 表示。它定义了核心 Java SE 和 JDK 模块。

平台类加载器不仅为了兼容性而保留，还为了提高安全性。由引导类加载器加载的类型隐式地被授予所有安全权限（ `AllPermission` ），但其中许多类型实际上并不需要所有权限。我们通过将它们定义为平台加载器而不是引导类加载器，并在默认安全策略文件中授予它们实际需要的权限，来定义了不需要所有权限的降权模块。定义为平台加载器的 Java SE 和 JDK 模块包括：

```
java.activation*            jdk.accessibility
java.compiler*              jdk.charsets
java.corba*                 jdk.crypto.cryptoki
java.scripting              jdk.crypto.ec
java.se                     jdk.dynalink
java.se.ee                  jdk.incubator.httpclient
java.security.jgss          jdk.internal.vm.compiler*
java.smartcardio            jdk.jsobject
java.sql                    jdk.localedata
java.sql.rowset             jdk.naming.dns
java.transaction*           jdk.scripting.nashorn
java.xml.bind*              jdk.security.auth
java.xml.crypto             jdk.security.jgss
java.xml.ws*                jdk.xml.dom
java.xml.ws.annotation*     jdk.zipfs
```

（在这些列表中，星号\*、 `'*'` 表示可升级模块。）

提供工具或导出工具 API 的 JDK 模块被定义为应用程序类加载器：

```
jdk.aot                     jdk.jdeps
jdk.attach                  jdk.jdi
jdk.compiler                jdk.jdwp.agent
jdk.editpad                 jdk.jlink
jdk.hotspot.agent           jdk.jshell
jdk.internal.ed             jdk.jstatd
jdk.internal.jvmstat        jdk.pack
jdk.internal.le             jdk.policytool
jdk.internal.opt            jdk.rmic
jdk.jartool                 jdk.scripting.nashorn.shell
jdk.javadoc                 jdk.xml.bind*
jdk.jcmd                    jdk.xml.ws*
jdk.jconsole
```

所有其他 Java SE 和 JDK 模块都定义为启动类加载器：

```
java.base                   java.security.sasl
java.datatransfer           java.xml
java.desktop                jdk.httpserver
java.instrument             jdk.internal.vm.ci
java.logging                jdk.management
java.management             jdk.management.agent
java.management.rmi         jdk.naming.rmi
java.naming                 jdk.net
java.prefs                  jdk.sctp
java.rmi                    jdk.unsupported
```

三种内置类加载器协同工作以加载类，具体如下：

* 应用程序类加载器首先搜索所有内置加载器中定义的命名模块。如果其中一个加载器定义了合适的模块，则该加载器将加载该类。如果在这些加载器定义的命名模块中找不到类，则应用程序类加载器委托给其父类。如果父类找不到类，则应用程序类加载器搜索类路径。在类路径上找到的类将作为此加载器未命名模块的成员加载。
* 平台类加载器搜索所有内置加载器中定义的命名模块。如果其中一个加载器定义了合适的模块，则该加载器将加载该类。（因此，平台类加载器现在可以委托给应用程序类加载器，这在升级模块路径上的模块依赖于应用程序模块路径上的模块时非常有用。）如果在这些加载器定义的命名模块中找不到类，则平台类加载器委托给其父类。
* 引导类加载器搜索定义给自己命名的模块。如果引导类加载器定义的命名模块中找不到一个类，则引导类加载器通过 `-Xbootclasspath/a` 选项搜索添加到引导类路径的文件和目录。在引导类路径上找到的类作为此加载器未命名模块的成员被加载。

应用程序和平台类加载器将委托给各自的父加载器，以确保在找不到定义在内置加载器之一的模块中的类时，仍然会搜索引导类路径。

**已移除：引导类路径选项**

在早期版本中， `-Xbootclasspath` 选项允许覆盖默认的引导类路径，而 `-Xbootclasspath/p` 选项允许将一系列文件和目录添加到默认路径之前。此路径的计算值通过 JDK 特定的系统属性 `sun.boot.class.path` 报告。

由于模块系统已就位，引导类路径默认为空，因为引导类是从各自的模块加载的。 `javac` 编译器仅在传统模式下支持 `-Xbootclasspath` 选项， `java` 启动器不再支持这两个选项，并且已删除系统属性 `sun.boot.class.path` 。

编译器的 `--system` 选项可以用来指定系统模块的替代源，如上所述，其 `-release` 选项可以用来指定替代的平台版本，如 JEP 247（为旧平台版本编译）中所述。在运行时，如上所述的 `--patch-module` 选项可以用来将内容注入到初始模块图中模块中。

一个相关的选项 `-Xbootclasspath/a` 允许将文件和目录追加到默认的引导类路径。此选项以及相关的 `java.lang.instrument` 包中的 API 有时被仪器代理使用，因此为了兼容性，在运行时仍然支持。如果指定了其值，将通过 JDK 特定的系统属性 `jdk.boot.class.path.append` 报告。此选项可以传递给命令行启动器 `java` ，也可以传递给 JNI 调用 API。

**测试**

许多现有测试受到了模块系统的引入影响。在 JDK 9 中，根据上述描述，添加了 `@modules` 标签到单元测试和回归测试中，需要时使用，并更新了使用 `-Xbootclasspath/p` 选项或假设系统类加载器是 `URLClassLoader` 的测试。

当然，模块系统本身有一套广泛的单元测试。在 JDK 9 的源代码库中，运行时测试位于 `jdk` 仓库的 test/jdk/modules 目录和 `hotspot` 仓库的 runtime/modules 目录中；编译时测试位于 `langtools` 仓库的 tools/javac/modules 目录中。

包含此处描述的更改的早期访问版本在整个模块系统开发过程中都可用。强烈鼓励 Java 社区的成员测试他们的工具、库和应用程序，以帮助识别兼容性问题。

## \[UI] JEP 262：[TIFF 图像 I/O](https://openjdk.org/jeps/262)

## \[UI] JEP 263：[Windows 和 Linux 上的 HiDPI 图形](https://openjdk.org/jeps/263)

## \[功能] JEP 264：[平台日志记录 API 和服务](https://openjdk.org/jeps/264)

### 摘要

定义一个最小的日志 API，平台类可以使用它来记录消息，同时提供一个服务接口，供消息的消费者使用。库或应用程序可以提供此服务的实现，以便将平台日志消息路由到其选择的日志框架。如果没有提供实现，则使用基于 `java.util.logging` API 的默认实现。

### 描述

使用 `java.util.ServiceLoader` API，一个系统级的 `LoggerFinder` 实现位于并使用系统类加载器加载。如果没有找到具体实现，则使用 JDK 内部默认的 实现的 `LoggerFinder` 服务。默认服务的实现当存在 `java.util.logging` 模块时，使用作为后端，因此默认情况下，日志消息会被路由到 `java.util.logging.Logger`，就像之前一样。然而， `LoggerFinder` 服务使得应用程序/框架能够插入自己的外部日志后端，而无需配置 `java.util.logging` 和该后端。 `LoggerFinder` 服务使得应用程序/框架能够插入自己的外部日志后端，而无需配置 `java.util.logging` 和该后端。

实现的 `LoggerFinder` 服务应能够区分系统日志记录器（由引导类加载器（BCL）中的系统类使用）和应用日志记录器（由应用程序为其自身使用而创建）。这种区分对于平台安全非常重要。日志记录器的创建者可以将创建日志记录器所用的类或模块传递给 `LoggerFinder`，以便 `LoggerFinder` 能够确定返回哪种类型的日志记录器。

JDK 中的类通过调用 `System` 类的工厂方法来获取由 `LoggerFinder` 创建的日志记录器：

```
package java.lang;

...

public class System {

    System.Logger getLogger(String name) { ... }

    System.Logger getLogger(String name, ResourceBundle bundle) { ... }

}
```

JDK 内部 API 将被修订，以便通过这些方法返回的系统日志记录器发出日志消息。

### 说明

JEP 264: **Platform Logging API and Service** 是 Java 9 引入的，用于规范和统一 Java 平台自身的日志输出行为，同时提供一个标准化的扩展点，**允许应用控制 JDK 内部模块的日志行为**。

**✅ 1. 它做了什么？**

主要解决两个核心问题：

**🔹 a. JDK 内部模块的日志标准化**

过去 JDK 内部使用多种日志方案（`System.err`, `java.util.logging`, 自建 Logger），不一致、难配置。

JEP 264 定义了一个 **Platform Logging API**，让所有 JDK 模块通过统一接口记录日志。

**🔹 b. 提供扩展点给开发者**

开发者可以通过注册 PlatformLogger 的日志实现（logging service provider），来**统一配置、拦截、定向** JDK 模块的日志输出。

**🧩 2. 为何区分平台日志和服务日志？**

| 类别       | 定义                                      | 使用者    | 配置方式            |
| -------- | --------------------------------------- | ------ | --------------- |
| **平台日志** | JDK 自身模块产生的日志（如 HTTPClient, JNDI, JAXB） | JDK 本身 | JEP 264 提供的 API |
| **服务日志** | 应用或框架产生的日志（如业务逻辑、日志框架）                  | 应用开发者  | 由用户代码控制         |

✅ 区分的目的：

* 避免平台日志污染应用日志
* 保持应用对日志的控制权（格式、等级、输出目标）
* 实现模块级、源头级的日志管理

**🛠️ 3. 平台日志 API：如何使用？**

核心类是：

```
jdk.internal.logger.DefaultLoggerFinder
java.lang.System.Logger
java.lang.System.LoggerFinder
```

**📌 获取平台 Logger：**

```
System.Logger logger = System.getLogger("com.example.MyComponent");
logger.log(System.Logger.Level.INFO, "Hello from platform logger!");
```

⚠️ 注意：这是 `java.lang.System.Logger`，**不是** `java.util.logging.Logger`！

**📌 自定义日志服务（高级用法）：**

你可以实现自己的 `System.LoggerFinder` 来将平台日志桥接到你的日志系统（如 Log4j、SLF4J）：

```
public class MyLoggerFinder extends System.LoggerFinder {
    @Override
    public System.Logger getLogger(String name, Module module) {
        return new MyCustomLogger(name); // 你自己的 Logger 实现
    }
}
```

并通过 JVM 启动参数注册：

```
--logger-finder=com.example.MyLoggerFinder
```

**✅ JEP 264 带来的好处**

| 优势                  | 描述                       |
| ------------------- | ------------------------ |
| 📦 模块化日志输出          | 不同模块使用标准 API，利于统一处理      |
| 🧩 自定义日志桥接          | 可桥接至 SLF4J / Log4j / JUL |
| 🔍 可控、可追踪           | 可以按模块、级别过滤和格式化平台日志       |
| 📊 更清晰的应用 vs 平台日志分离 | 避免平台日志“污染”业务日志输出         |

**🧪 示例：平台日志桥接到 `java.util.logging`**

如果你不实现自定义 LoggerFinder，默认会桥接到 `java.util.logging`，你可以在 `logging.properties` 中配置它：

```
# 配置平台日志
.level = INFO
com.example.MyComponent.level = FINE
```

**🧵 总结一句话：**

**JEP 264 把 JDK 自己的日志输出“规范化”和“模块化”了，让你可以用自己的方式统一管理和捕捉这些日志**。这对开发中调试、性能分析、日志隔离都非常有用。

## \[UI] JEP 265：[Marlin 图形渲染器](https://openjdk.org/jeps/265)

## \[功能] JEP 266：[更多并发更新](https://openjdk.org/jeps/266)

### 摘要

一个可互操作的发布-订阅框架，对 `CompletableFuture` API 的增强以及各种其他改进。

### 描述

1. 支持反应式流发布-订阅框架的接口，嵌套在新的类 `Flow` 中。 `Publisher` 生成由一个或多个 `Subscriber` 消费的项目，每个 `Subscriber` 由一个 `Subscription` 管理。通信依赖于一种简单的流控制形式（方法 `Subscription.request` ，用于传递背压），这可以用来避免在“推送”系统中可能出现的资源管理问题。提供了一个实用类 `SubmissionPublisher` ，开发人员可以使用它来创建自定义组件。 这些（非常小）的接口对应于由 Reactive Streams 倡议定义的接口，并支持在 JVM 上运行的多个异步系统之间的互操作性。将这些接口嵌套在类中是一种保守的策略，允许它们在各种短期和长期可能性中使用。目前没有计划提供基于网络或 I/O 的 `java.util.concurrent` 组件用于分布式消息传递，但未来 JDK 版本可能在其他包中包含此类 API。
2. 对 `CompletableFuture` API 的增强
   * 添加了基于时间的增强，使未来可以在一定时间后或异常完成，请参阅 `orTimeout` 和 `completeTimeout` 方法。此外，由名为 `delayedExecutor` 的静态方法返回的互补的 `Executor` 允许任务在一段时间后执行。这可以与 `CompletableFuture` 上的 `Executor` 接收方法结合使用，以支持具有时间延迟的操作。
   * 添加了子类增强，使得从 `CompletableFuture` 扩展更容易，例如提供支持替代默认执行器的子类。
3. 自 JDK 8 以来积累的众多实现改进；其中许多都是小的，但也有一些包括 Javadoc 规范重写。

## \[优化] JEP 267：[Unicode 8.0](https://openjdk.org/jeps/267)

将现有平台 API 升级以支持[版本 8.0](http://www.unicode.org/versions/Unicode8.0.0/) [Unicode 标准 ](http://www.unicode.org/standard/standard.html)。

## \[xml] JEP 268：[XML 目录](https://openjdk.org/jeps/268)

制定一个支持的标准 XML 目录 API [OASIS XML 目录标准，v1.1](https://www.oasis-open.org/committees/download.php/14809/xml-catalogs.html)。该 API 将定义目录和目录解析器抽象，这些抽象可以与接受解析器的 JAXP 处理器一起使用。

## \[功能] JEP 269：[集合的便捷工厂方法](https://openjdk.org/jeps/269)

### 摘要

Java 经常因其冗长而受到批评。创建一个小型不可修改的集合（比如一个集合），需要构造它，将其存储在局部变量中，并多次调用 `add()` ，然后将其包装起来。例如，

```
Set<String> set = new HashSet<>();
set.add("a");
set.add("b");
set.add("c");
set = Collections.unmodifiableSet(set);
```

这相当冗长，因为它无法用单个表达式表达，静态集合必须在静态初始化块中填充，而不是通过更方便的字段初始化。或者，也可以使用另一个集合的复制构造函数来填充集合：

```
Set<String> set = Collections.unmodifiableSet(new HashSet<>(Arrays.asList("a", "b", "c")));
```

这仍然有些冗长，并且也不太明显，因为必须先创建 `List` ，然后再创建 `Set` 。另一个选择是使用所谓的“双括号”技术：

```
Set<String> set = Collections.unmodifiableSet(new HashSet<String>() {{
    add("a"); add("b"); add("c");
}});
```

这使用匿名内部类中的实例初始化器构造，这看起来更漂亮。然而，它相当晦涩，并且每次使用都会额外创建一个类。它还持有对封装实例和任何捕获对象的隐藏引用。这可能会导致内存泄漏或序列化问题。因此，最好避免这种技术。

可以使用 Java 8 Stream API 通过组合流工厂方法和收集器来构建小型集合。例如，

```
Set<String> set = Collections.unmodifiableSet(Stream.of("a", "b", "c").collect(toSet()));
```

流式收集器对其返回的集合的可变性不提供任何保证。在 Java 8 中，返回的集合是普通的、可变的集合，例如 `ArrayList` 、 `HashSet` 和 `HashMap` ，但这可能在未来的 JDK 版本中发生变化。

这有些绕弯子，虽然不算晦涩，但也不太明显。这也涉及到一定程度的无谓的对象创建和计算。像往常一样， `Map` 是例外。除非值可以由键计算得出，或者流元素同时包含键和值，否则不能以这种方式使用流来构建 `Map` 。

以前，曾有人提出过一些修改 Java 编程语言的提案，以支持集合字面量。然而，正如语言特性通常所发生的那样，没有哪个特性像人们最初想象的那样简单或干净，因此集合字面量不会出现在下一个版本的 Java 中。

通过提供用于创建小型集合实例的库 API，可以获取集合字面量的大部分好处，与修改语言相比，成本和风险显著降低。例如，创建小型 Set 实例的代码可能如下所示：

```
Set<String> set = Set.of("a", "b", "c");
```

类 `Collections` 中存在现有的工厂方法，用于支持创建空的 `List` 、 `Set` 和 `Map` 。还有用于生成单例 `List` 、 `Set` 和 `Map` 的工厂，这些单例包含一个元素或键值对。 `EnumSet` 包含几个重载的 `of(...)` 方法，可以接受固定或可变数量的参数，方便地创建包含指定元素的 `EnumSet` 。然而，目前还没有一种好的通用方法来创建包含任意类型对象的 `List` 、 `Set` 和 `Map` 。

### **描述**

在 `Collections` 类中存在组合方法，用于创建不可修改的 `List` 、 `Set` 和 `Map` 。这些方法并不创建本质上不可修改的集合。相反，它们接受另一个集合，并将其包装在一个拒绝修改请求的类中，从而创建原始集合的不修改视图。对底层集合的引用仍然允许修改。每个包装器都是额外的对象，需要额外的间接层，并且比原始集合消耗更多的内存。最后，即使不打算修改，包装的集合仍然承担着支持修改的开销。

在 `List` 、 `Set` 和 `Map` 接口上提供静态工厂方法，用于创建不可修改的集合实例。（注意，与类上的静态方法不同，接口上的静态方法不是继承的，因此无法通过实现类或接口类型的实例调用它们。）

对于 `List` 和 `Set` ，这些工厂方法将按以下方式工作：

```
List.of(a, b, c);
Set.of(d, e, f, g);
```

这些方法将包括可变参数重载，因此集合的大小没有固定限制。然而，由此创建的集合实例可能针对较小的尺寸进行调整。将提供针对最多十个元素的固定参数 API（固定参数重载）。虽然这会在 API 中引入一些杂乱，但可以避免由可变参数调用产生的数组分配、初始化和垃圾回收开销。值得注意的是，无论调用的是固定参数还是可变参数重载，调用点的源代码都是相同的。

对于 `Map` ，将提供一组固定参数方法：

```
Map.of()
Map.of(k1, v1)
Map.of(k1, v1, k2, v2)
Map.of(k1, v1, k2, v2, k3, v3)
...
```

我们预计支持最多十个键值对的较小映射将足以覆盖大多数用例。对于更多条目，将提供一个 API，该 API 将根据任意数量的键值对创建一个 `Map` 实例：

```
Map.ofEntries(Map.Entry<K,V>...)
```

虽然这种方法与 `List` 和 `Set` 的等效 varargs API 类似，但它不幸地要求每个键值对都被装箱。一个适合静态导入的装箱键和值的方法将使这一过程更加方便：

```
Map.Entry<K,V> entry(K k, V v)
```

使用这些方法，将能够创建具有任意数量条目的映射：

```
Map.ofEntries(
    entry(k1, v1),
    entry(k2, v2),
    entry(k3, v3),
    // ...
    entry(kn, vn));
```

（未来版本的 JDK 可能会通过使用值类型来减轻装箱的开销。 `entry()` 便利方法实际上将返回一个新引入的具体类型，该类型实现了 `Map.Entry` ，以便促进未来迁移到值类型。）

提供用于创建小型不可变集合的 API 满足大量用例，这有助于保持规范和实现简单。不可变集合避免了需要创建防御性副本的需求，并且更适合并行处理。

小型集合的运行时空间也是一个重要的考虑因素。使用包装 API 直接创建包含两个元素的不可变 `HashSet` ，将包含六个对象：包装器、包含 `HashSet` 的 `HashMap` ，其桶表（数组）以及每个元素一个的 `Node` 实例。与存储的数据量相比，这会带来巨大的开销，并且不可避免地需要多次方法调用和指针解引用来访问数据。针对小型固定大小集合设计的实现可以避免大部分这种开销，使用紧凑的字段或数组布局。不需要支持修改（并且知道创建时的集合大小）也有助于节省空间。

这些工厂返回的具体类将不会作为公共 API 公开。不会对返回集合的运行时类型或标识做出任何保证。这将允许实现随着时间的推移而改变，而不会破坏兼容性。调用者唯一应该依赖的是返回的引用是其接口类型的实现。

生成的对象将是可序列化的。将使用序列化代理对象作为实现类的通用序列化形式。这将防止有关具体实现的信息泄露到序列化形式中，从而保留未来的维护灵活性，并允许具体实现从版本到版本地改变，而不会影响序列化兼容性。

不允许 null 元素、键和值。（最近引入的集合都不支持 null。）此外，禁止 null 提供了更紧凑的内部表示、更快的访问速度和更少的特殊情况。

预期 `List` 实现将提供快速的按索引访问元素，因此它们将实现 `RandomAccess` 标记接口。

这些集合中存储的元素必须支持典型的集合契约，包括对 `hashCode()` 和 `equals()` 的正确支持。如果 `Set` 或 `Map` 的元素或键以影响其 `hashCode()` 或 `equals()` 方法的方式被修改，集合的行为可能会变得不可指定。

一旦构建并安全发布，这些集合实例将能够被多个线程安全访问。

将在 JDK 中搜索可以应用这些新 API 的潜在位置。随着时间和计划的允许，这些位置将被更新以使用新 API。

## \[JVM] JEP 270：[为关键部分保留的堆栈区域](https://openjdk.org/jeps/270)

### 摘要

为关键部分保留额外的线程栈空间，以便即使在栈溢出发生时也能完成。

### 描述

该解决方案的主要思想是为关键部分在执行栈上预留一些空间，以便它们能够在常规代码因栈溢出而中断的地方完成执行。假设关键部分相对较小，不需要在执行栈上占用巨大的空间才能成功完成。目标不是拯救一个达到其栈限制的故障线程，而是保护在关键部分抛出 `StackOverflowError` 时可能被破坏的共享数据结构。

主机制将在 JVM 中实现。唯一的修改 Java 源代码中需要使用的注解 识别临界区。此注释目前命名为 `jdk.internal.vm.annotation.ReservedStackAccess` ，是一个运行时方法注解，任何特权代码类都可以使用（参见下文关于此注解的可访问性段落）。

为了防止共享数据结构的损坏，JVM 将尝试延迟抛出 `StackOverflowError`，直到线程 所讨论的线程已退出其所有关键区域。每个 Java 线程都有 一个在其执行栈中定义的新区域，称为保留区。 区域只能在 Java 线程当前调用栈中存在方法被注解为 `jdk.internal.vm.annotation.ReservedStackAccess` 时使用。 当 JVM 检测到栈溢出条件，并且线程调用栈中存在被注解的方法时，JVM 会临时授予访问权限。 当 JVM 检测到栈溢出条件，并且线程调用栈中存在被注解的方法时，JVM 会临时授予访问权限。 当 JVM 检测到栈溢出条件，并且线程调用栈中存在被注解的方法时，JVM 会临时授予访问权限。 保留区域直到没有更多注解方法存在于调用中 栈。当撤销对保留区的访问时，将延迟 抛出 `StackOverflowError`。如果线程没有注解的方法在 检测到栈溢出条件时，其调用栈 `StackOverflow` 立即抛出异常（这是当前 JVM 的行为）。

注意，保留的栈空间不仅可以由注解方法使用，还可以由直接或间接调用它们的任何方法使用。注解方法的嵌套自然得到支持，但每个线程只有一个共享的保留区域；也就是说，调用注解方法不会添加新的保留区域。保留区域的大小必须根据所有注解关键节点的最坏情况来设置。

默认情况下， `jdk.internal.vm.annotation.ReservedStackAccess` 注解仅适用于特权代码（由引导程序或扩展类加载器加载的代码）。特权代码和非特权代码都可以使用此注解，但默认情况下，JVM 会忽略非特权代码。这种默认策略的依据是，临界区保留的栈空间是所有临界区共享的资源。如果任何任意代码能够使用这个空间，那么它就不再是保留空间了，这将破坏整个解决方案。即使在产品构建中，也有一个 JVM 标志可以放松这项策略，允许任何代码都能从中受益。

## \[JVM] JEP 271：[统一 GC 日志记录](https://openjdk.org/jeps/271)

使用在 [JEP 158](http://openjdk.java.net/jeps/158) 中引入的统一 JVM 日志框架重新实现 GC 日志记录。

### 描述

以尽可能一致的方式重新实现 GC 日志记录。新格式和旧格式之间必然会有一些差异。

**"gc" 标签**

理念是，`-Xlog:gc`（仅在“gc”标签上以 info 级别进行日志记录）应该与 `-XX:+PrintGC` 的效果相似，即每进行一次垃圾回收打印一行。这意味着应该非常谨慎地使用 `log_info(gc)("message")`。除非是每次垃圾回收应该打印的那条消息，否则不要仅在“gc”标签上以 info 级别进行日志记录。

如果将“gc”标签与其他标签结合使用，以 info 级别进行日志记录是可以的。例如：

```
log_info(gc, heap, ergo)("Heap expanded");
```

这里的理念是，`-Xlog:gc` 应该与您以前使用 `-XX:+PrintGCDetails` 得到的效果相似。但这种映射并不像从 `-Xlog:gc` 到 `-XX:+PrintGC` 的映射那样严格。对于 `-XX:+PrintGC` 的规则非常明确：每次垃圾回收一行。对于 `-XX:+PrintGCDetails` 的规则从未非常明确。因此，一些 `-XX:+PrintGCDetials` 的日志可能映射到多个标签，而一些可能仅映射到“gc”标签的 debug 级别。

所有与垃圾回收相关的日志记录都应该使用“gc”标签。大多数日志记录不应仅使用“gc”标签，而应结合其他适当的标签。

也有一些边界情况，不清楚是否应该使用 "gc" 标签，例如在分配代码中。这些情况中，可能大部分都不应该使用 "gc" 标签。

**其他标签**

除了 "gc" 之外，还有很多其他标签。其中一些与旧标志映射得相当清晰。例如，`PrintAdaptiveSizePolicy` 大概映射到 "ergo" 标签（结合 "gc" 标签和可能的其他标签）。

**详细**

大多数受 `Verbose` 标志（一个开发标志）保护的日志应映射到跟踪级别。例外情况是，如果它是从性能角度来看非常昂贵的日志，在这种情况下，它映射到开发级别。

**前缀**

统一日志框架中的前缀支持用于将 GC ID 添加到 GC 日志消息中。GC ID 仅在 GC 期间发生的日志中才有意义。由于前缀是为特定的标签集定义的，即标签的组合，因此必须确保在 GC 之间发生的日志不使用与 GC 期间进行的日志相同的标签集。

**动态配置**

某些日志记录需要收集早期状态的数据。统一的日志框架允许使用 `jcmd` 动态地开启和关闭所有日志记录。这意味着对于依赖于先前收集数据的日志记录，仅检查日志是否开启是不够的；还必须实施检查以确保数据可用。

## \[UI] JEP 272：[特定于平台的桌面功能](https://openjdk.org/jeps/272)

## \[功能] JEP 273：[基于DRBG的安全随机实现](https://openjdk.org/jeps/273)

`SecureRandom` 是为加密、认证、token、安全协议等场景提供**不可预测**随机数的工具，JEP 273 让它更快、更灵活、更安全。

## \[功能] JEP 274：[增强方法句柄](https://openjdk.org/jeps/274)

### 摘要

增强 `MethodHandle`、`MethodHandles` 和 `MethodHandles.Lookup` 通过新的 `MethodHandle` 组合子和查找细化，简化 `java.lang.invoke` 包的类，以方便常见用例并使编译器优化更好。 组合子和查找细化。

### 描述

**循环组合器**

**最通用的循环抽象**

循环的核心抽象包括循环的初始化、检查的谓词和要评估的主体。最通用的 `方法句柄` 组合器用于创建循环，将被添加到 `MethodHandles` 中，如下所示：

```
MethodHandle loop(MethodHandle[]... clauses)
```

构建一个表示具有多个循环变量（在每次迭代中更新和检查）的循环的方法句柄。当循环因其中一个谓词而终止时，将运行相应的终结器，并返回循环的结果，即结果句柄的返回值。

直观上，每个循环由一个或多个“子句”组成，每个子句指定一个局部迭代值和/或循环退出条件。循环的每次迭代按顺序执行每个子句。子句可以可选地更新其迭代变量；它还可以可选地执行测试和条件循环退出。为了用方法句柄表达这种逻辑，每个子句将确定四个操作：

* 循环执行前，迭代变量或循环不变量的初始化。
* 当执行一个子句时，迭代变量的更新步骤。
* 当执行一个子句时，执行谓词以测试循环是否退出。
* 如果一个子句导致循环退出，执行一个最终化步骤以计算循环的返回值。

根据某些规则，这些子句部分可能被省略，并在这种情况下提供有用的默认行为。下面将详细说明。

除了子句初始化器之外，每个子句函数都能够观察整个循环状态，因为它将传递所有当前迭代变量的值以及所有传入的循环参数。大多数子句函数不需要所有这些信息，但它们将通过 `dropArguments` 的形式正式连接。

给定一组子句，将执行一系列检查和调整以连接循环的所有部分。它们在下面的步骤中详细说明。在这些步骤中，每个“必须”一词的出现都对应于如果循环组合器的输入未满足所需约束，则可能会抛出 `IllegalArgumentException` 的地方。对于参数类型列表，“实质上相同”一词意味着它们必须相同，或者一个列表必须是另一个列表的有效前缀。

_步骤 0：确定子句结构。_

* 子句数组（类型为 `MethodHandle[][]`）必须非 `null` 且至少包含一个元素。
* 子句数组不得包含 `null` 或长度超过四个元素的子数组。
* 短于四个元素的子句将被视为用 `null` 元素填充至长度为四。填充操作通过向数组中追加元素来完成。 `null` 元素填充至长度为四。填充操作通过向数组中追加元素来完成。
* 所有 `null` 的子句将被忽略。
* 每个子句被视为一个包含“init”、“step”、“pred”和“fini”四个函数的四元组。

_步骤 1A：确定迭代变量。_

* 逐一检查 init 和 step 函数的返回类型，以确定每个子句的迭代变量类型。
* 如果两个函数都省略，则使用 `void`；否则如果省略一个，则使用另一个的返回类型；否则使用公共返回类型（它们必须相同）。
* 形成返回类型列表（按子句顺序），省略所有出现。 `void`。
* 这个类型列表称为“公共前缀”。

_步骤 1B：确定循环参数。_

* 检查初始化函数的参数列表。
* 被省略的初始化函数被视为具有 `null` 参数列表。
* 所有初始化函数的参数列表必须实质上完全相同。
* 最长的参数列表（必然是唯一的）被称为“公共后缀”。

_步骤 1C：确定循环返回类型。_

* 检查 fini 函数的返回类型，忽略省略的 fini 函数。
* 如果没有 fini 函数，则使用 `void` 作为循环返回类型。
* 否则，使用 fini 函数的通用返回类型；它们必须完全相同。

_步骤 1D：检查其他类型。_

* 至少必须有一个非省略的 pred 函数。
* 每个非省略的 pred 函数都必须有 `布尔` 返回类型。

（实施说明：步骤 1A、1B、1C、1D 彼此独立，可以按任何顺序执行。）

_步骤2：确定参数列表。_

* 结果循环句柄的参数列表将是“公共后缀”。
* 初始化函数的参数列表将调整为“公共后缀”。（注意，它们的参数列表已经实际上与公共后缀相同。）
* 非初始化（step、pred 和 fini）函数的参数列表将被调整为以公共前缀开头，后跟公共后缀，称为“公共参数序列”。
* 每个非初始化、非省略的函数参数列表必须与公共参数序列有效相同。

_第3步：填写省略的函数。_

* 如果省略了初始化函数，则使用适当的常量函数。 `空` /零/`假`/`void` 类型。（为此，一个常量 `void` 简单来说就是一个什么也不做并返回 `void` 的函数；可以通过 `MethodHandle.asType 类型` 从另一个常量函数通过类型转换获得。）
* 如果省略了步骤函数，则使用子句迭代变量类型的恒等函数；在恒等函数参数之前插入丢失的参数，用于前一个子句的非 `void` 迭代变量。（这将使循环变量成为局部循环不变量。）
* 如果省略了 pred 函数，则相应的 fini 函数也必须省略。
* 如果省略了 pred 函数，则使用一个恒等函数 `true`。 (根据这一条款，这将使循环继续。)
* 如果省略了 fini 函数，则使用常量 `null`/零/`false`/`void` 循环返回类型的功能。

_步骤 4：填写缺失的参数类型。_

* 到此为止，每个 init 函数的参数列表实际上都与公共后缀相同，但某些列表可能更短。对于每个参数列表较短的 init 函数，通过删除参数来填充列表的末尾。
* 此时，每个非初始化函数的参数列表实际上都与通用参数序列相同，但某些列表可能更短。对于每个具有短参数列表的非初始化函数，通过省略参数来填充列表的末尾。

_最终观察。_

* 在这些步骤之后，所有子句都已通过提供省略的函数和参数进行调整。
* 所有初始化函数都有一个共同的参数类型列表，最终的循环句柄也将具有该列表。
* 所有 fini 函数有一个共同的返回类型，这个类型最终循环句柄也将具有。
* 所有非初始化函数都有一个共同的参数类型列表，即公共参数序列，由（非 `void`）迭代变量后跟循环参数组成。
* 每对 init 和 step 函数在返回类型上保持一致。
* 每个非初始化函数都能通过公共前缀观察到所有迭代变量的当前值。

_循环执行。_

* 当调用循环时，循环输入值将保存在局部变量中，作为公共后缀传递给每个子句函数。这些局部变量是循环不变的。
* 每个初始化函数按子句顺序执行（传递公共后缀），非 `void` 值（作为公共前缀）保存到局部变量中。这些局部变量是循环变化的（除非它们的步长是恒等函数，如上所述）。
* 所有函数执行（除了初始化函数）都将传递公共参数序列，包括非 `void` 迭代值（按子句顺序）以及循环输入（按参数顺序）。
* 然后按子句顺序执行步进和预测函数（步进在预测之前），直到预测函数返回 `false`。
* 步进函数调用的非 `void` 结果用于更新相应的循环变量。更新后的值立即对所有后续函数调用可见。
* 如果预测函数返回 `false`，则调用相应的结束函数，并将结果作为整个循环的返回值。

方法句柄 `MethodHandle``l` 从 `loop` 返回的语义如下：

```
l(arg*) =>
{
    let v* = init*(arg*);
    for (;;) {
        for ((v, s, p, f) in (v*, step*, pred*, fini*)) {
            v = s(v*, arg*);
            if (!p(v*, arg*)) {
                return f(v*, arg*);
            }
        }
    }
}
```

基于这个最通用的循环抽象，应添加几个方便的组合器到《MethodHandles》。它们将在下文中讨论。

**简单的 while 和 do-while 循环**

这些组合器将被添加到 `MethodHandles` 中：

```
MethodHandle whileLoop(MethodHandle init, MethodHandle pred, MethodHandle body)

MethodHandle doWhileLoop(MethodHandle init, MethodHandle body, MethodHandle pred)
```

从 `MethodHandle` 对象 `wl` 返回的调用语义是： `whileLoop` 的语义如下：

```
wl(arg*) =>
{
    let r = init(arg*);
    while (pred(r, arg*)) { r = body(r, arg*); }
    return r;
}
```

对于从 `MethodHandle``dwl` 返回的 `doWhileLoop`，其语义如下：

```
dwl(arg*) =>
{
    let r = init(arg*);
    do { r = body(r, arg*); } while (pred(r, arg*));
    return r;
}
```

此方案对三个构成 `MethodHandle` 的签名施加了一些限制：

1. 初始化器 `init` 的返回类型也是主体 `body` 和整个循环的返回类型，以及谓词 `pred` 和主体 `body` 的第一个参数的类型。
2. 谓词 `pred` 的返回类型必须是 `boolean`。

**计数循环**

为了方便，以下循环组合器也将提供：

*   `MethodHandle countedLoop(MethodHandle iterations, MethodHandle init, MethodHandle body)`

    一个从 `MethodHandle``cl` 返回的 `countedLoop` 具有以下语义：

    ```
    cl(arg*) =>
    {
        let end = iterations(arg*);
        let r = init(arg*);
        for (int i = 0; i < end; i++) {
            r = body(i, r, arg*);
        }
        return r;
    }
    ```
*   `MethodHandle countedLoop(MethodHandle start, MethodHandle end, MethodHandle init, MethodHandle body)`

    一个从此变体的 `MethodHandle``cl` 返回的 `countedLoop` 具有以下语义：

    ```
    cl(arg*) =>
    {
        let s = start(arg*);
        let e = end(arg*);
        let r = init(arg*);
        for (int i = s; i < e; i++) {
            r = body(i, r, arg*);
        }
        return r;
    }
    ```

在这两种情况下，`body` 的第一个参数类型必须是 `int`，同时 `init` 和 `body` 的返回类型以及 `body` 的第二个参数也必须相同。 必须相同。

**数据结构迭代**

此外，一个循环组合器对于迭代是有帮助的：

*   `MethodHandle iteratedLoop(MethodHandle iterator, MethodHandle init, MethodHandle body)`

    一个从 `MethodHandle``it` 返回的 `iteratedLoop` 具有以下语义：

    ```
    it(arg*) =>
    {
        let it = iterator(arg*);
        let v = init(arg*);
        for (T t : it) {
            v = body(t, v, a);
        }
        return v;
    }
    ```

**备注**

更多的便利循环组合是可想象的。

虽然 `continue` 的语义可以很容易地通过从主体返回来模拟，但如何模拟 `break` 的语义仍然是一个悬而未决的问题。这 可以通过使用专用异常（例如， `LoopMethodHandle.BreakException` ）来实现。

**组合器用于 `try`/`finally` 块**

为了便于从 try/finally 语义构建功能 方法句柄（MethodHandle），以下将引入新的组合器： 方法句柄

```
MethodHandle tryFinally(MethodHandle target, MethodHandle cleanup)
```

调用从 `MethodHandle``tf` 返回的语义如下：

```
tf(arg*) =>
{
    Throwable t;
    Object r;
    try {
        r = target(arg*);
    } catch (Throwable x) {
        t = x;
        throw x;
    } finally {
        r = cleanup(t, r, arg*);
    }
    return r;
}
```

这意味着生成的 `MethodHandle` 的返回类型将是那种类型 目标代码处理。目标代码处理和清理代码都必须具有匹配的参数列表，对于清理代码，它接受一个 `Throwable`。 参数和可能的中继结果。如果发生异常， 在执行`目标`时抛出，此参数将保留该异常。

**参数处理组合器**

作为现有 API 中 `MethodHandles` 的补充，将引入以下方法：

*   对类 `MethodHandle` 的扩展 - 新实例方法：

    ```
    MethodHandle asSpreader(int pos, Class<?> arrayType, int arrayLength)
    ```

    在结果的签名中，在位置 `pos` 处，期望 `arrayLength` 类型为 `arrayType` 的参数。在结果中插入一个消耗 `arrayLength` 个 `this``MethodHandle` 参数。如果 `this` 的签名在该位置没有足够的参数，或者如果该位置不 有足够的参数，或者如果该位置不存在 必须存在于签名中，抛出适当的异常。

    例如，如果 `此` 的签名是 `(Ljava/lang/String;IIILjava/lang/Object;)V` ，调用 `asSpreader(int[].class, 1, 3)` 将导致生成的签名 `(Ljava/lang/String;[ILjava/lang/Object;)V` .
*   类 `MethodHandle` 的扩展 - 新实例方法：

    ```
    MethodHandle asCollector(int pos, Class<?> arrayType, int arrayLength)
    ```

    在 `this` 的签名中，位置 `pos` 处期望一个数组参数。在结果签名的位置 `pos` 将会有 `arrayLength` 该数组的参数类型。所有在 `pos` 之前的参数不受影响。所有在 `pos` 之后的参数将向右移动 `arrayLength` 的参数预期在运行时可用 如果它们不可用，则会抛出 `ArrayIndexOutOfBoundsException` 异常。 例如，如果 `this` 的签名是

    对于 `this` 的签名，例如 `(Ljava/lang/String;[ILjava/lang/Object;)V` 调用 `asCollector(int[].class, 1, 3)` 将导致生成的签名 `(Ljava/lang/String;IIILjava/lang/Object;)V` .
*   向类 `MethodHandles` 添加内容 - 新的静态方法：

    ```
    MethodHandle foldArguments(MethodHandle target, int pos, MethodHandle combiner)
    ```

    结果的 `方法句柄` 在调用时将像现有的方法 `foldArguments(MethodHandle target, MethodHandle combiner)` 一样执行，区别在于现有的方法暗示了一个折叠位置为 `0`，而提议的新方法允许指定一个不同于 `0` 的折叠位置。 `0`，而提议的新方法允许指定一个不同于 `0` 的折叠位置。

    例如，如果 `目标` 签名是 `(ZLjava/lang/String;ZI)I`，并且 `组合器` 签名是 `(ZI)Ljava/lang/String;`，调用 `foldArguments(target, 1, combiner)` 将导致生成的签名 `(ZZI)I`，并且第二个和第三个参数（`布尔值`和 `整数` ）将在每次调用时合并为一个 `字符串` 。

这些新的组合器将通过现有的抽象和 API 实现。如果需要，将修改非公共 API。

**查找**

方法 `MethodHandles.Lookup.findSpecial(Class<?> refc, String name, MethodType type, Class<?> specialCaller)` 的实现将被修改，以允许在接口上找到 `super`-可调用方法。虽然这并不是 API 的变更，但其文档行为发生了显著变化。

此外，`MethodHandles.Lookup` 类将扩展以下两个方法：

*   `Class<?> findClass(String targetName)`

    此操作检索代表由 `targetName` 标识的所需目标类的 `Class<?>` 实例。查找应用由隐式访问上下文定义的限制。如果访问不可行，该方法将引发适当的异常。
*   `Class<?> accessClass(Class<?> targetClass)`

    此操作尝试访问给定的类，应用由隐式访问上下文定义的限制。如果访问不可行，该方法将引发适当的异常。

### 说明

JEP 274: **Enhanced Method Handles** 是对 Java 中 `java.lang.invoke.MethodHandle` API 的增强，主要目标是：

> ✅ **使方法句柄更强大、更易用、更接近 Java 语言级别的表达能力**。

**🔧 一句话总结：**

JEP 274 为方法句柄引入了更多“组合器（combinator）”和便捷工具，使得你可以**像操作函数一样操作方法句柄**，实现类似函数式编程的风格。

**🔍 方法句柄是什么？**

方法句柄（`MethodHandle`）是 Java 7 引入的一种比反射更快的 **低层级方法调用机制**，广泛用于：

* 动态语言运行时（如 Nashorn, Kotlin, Scala）
* Lambda 表达式底层实现（`LambdaMetafactory`）
* 高性能框架（如 Graal, JMH）

**✅ JEP 274 做了什么？**

JEP 274 增强了 `MethodHandles` 工具类，新增了一批组合器函数，例如：

| 新增组合器方法             | 功能            |
| ------------------- | ------------- |
| `filterArguments`   | 修改部分参数前进行转换   |
| `filterReturnValue` | 修改返回值         |
| `guardWithTest`     | 实现 if-else 逻辑 |
| `dropArguments`     | 增加无用参数（用于占位）  |
| `collectArguments`  | 参数聚合，类似柯里化    |
| `foldArguments`     | 参数“折叠”处理      |

**🧪 示例讲解**

**1. `filterArguments`：对某些参数进行转换后再调用原方法**

```
MethodHandle target = lookup.findStatic(Math.class, "abs", methodType(int.class, int.class));
MethodHandle converter = lookup.findVirtual(String.class, "length", methodType(int.class));

// 把 String 参数先变成 int（取 length），再传给 abs
MethodHandle filtered = MethodHandles.filterArguments(target, 0, converter);

// 等价于：abs("hello".length()) => 5
int result = (int) filtered.invoke("hello");
System.out.println(result); // 5
```

**2. `filterReturnValue`：对返回值做额外处理**

```
MethodHandle plus = lookup.findStatic(Math.class, "addExact", methodType(int.class, int.class, int.class));
MethodHandle toString = lookup.findStatic(Integer.class, "toString", methodType(String.class, int.class));

// 把加法结果变成字符串
MethodHandle stringResult = MethodHandles.filterReturnValue(plus, toString);

// stringResult(3, 5) => "8"
String s = (String) stringResult.invoke(3, 5);
System.out.println(s);
```

**3. `guardWithTest`：实现 if-else 调度逻辑**

```
MethodHandle test = lookup.findStatic(SomeUtil.class, "isEven", methodType(boolean.class, int.class));
MethodHandle evenHandler = lookup.findStatic(SomeUtil.class, "handleEven", methodType(String.class, int.class));
MethodHandle oddHandler = lookup.findStatic(SomeUtil.class, "handleOdd", methodType(String.class, int.class));

// 如果 isEven(x) 为 true，则调用 evenHandler(x)，否则调用 oddHandler(x)
MethodHandle guarded = MethodHandles.guardWithTest(test, evenHandler, oddHandler);
String result = (String) guarded.invoke(4); // 偶数路径
```

**补充类：**

```
public class SomeUtil {
    public static boolean isEven(int x) { return x % 2 == 0; }
    public static String handleEven(int x) { return x + " is even"; }
    public static String handleOdd(int x) { return x + " is odd"; }
}
```

**✨ 为什么这很有用？**

JEP 274 的增强让我们：

| 好处              | 描述                                |
| --------------- | --------------------------------- |
| ✅ 更灵活           | 像组合函数一样组合方法调用                     |
| 🚀 性能佳          | 无需反射，底层基于 `invokedynamic`，JIT 可优化 |
| 🤝 支持 Lambda 实现 | `LambdaMetafactory` 内部构造用到了这些组合器  |
| 🧱 支撑动态语言       | JRuby、Nashorn 使用它作为核心执行模型         |

**🔚 总结**

JEP 274 把 Java 的 `MethodHandle` 变成了“函数式第一类公民”，你可以像拼乐高一样组合方法逻辑，灵活、高性能、可读性提升。

## JEP 275：[模块化 Java 应用程序打包](https://openjdk.org/jeps/275)

### 摘要

将 [Project Jigsaw](http://openjdk.java.net/projects/jigsaw) 的功能集成到 Java 打包器中，包括模块意识和自定义运行时创建。

### 描述

在大多数情况下，Java 打包器的工作流程将保持不变。将添加来自 Jigsaw 的新工具，并在某些情况下替换一些步骤。

**仅生成 Java 9 应用程序**

Java 打包器将仅创建使用 JDK 9 运行时的应用程序。这将简化许多代码路径和关于用于组装应用程序和 Java 运行时的工具的假设。如果用户想创建 Java 8 应用程序，则与 JDK 8 一起提供的 Java 打包器的 Java 8 版本将继续工作。我们假设需要同时在工作在 Java 8 和 Java 9 上的自包含应用程序的数量将几乎为零，因为应用程序会自带 JVM。

**使用 `jlink` 生成嵌入式 Java 运行时和应用镜像**

目前 JRE 被复制，并且从复制的运行时中删除了不需要的部分。

Java 链接工具 [Java 链接工具 ](http://openjdk.java.net/jeps/282)，`jlink`，提供了一种生成只包含所需模块的 JRE 镜像的方法。此外，`jlink` 可能暴露一些用于其镜像生成过程的钩子，我们可以利用这些钩子进一步定制镜像，例如，在 `jlink` 处理中添加删除可执行文件，或压缩。

Java 打包器将调用 `jlink` 来创建一个将嵌入到应用程序镜像中的应用程序运行时镜像。如果 `jlink` 失败，Java 打包器将失败并显示适当的错误。预计打包的模块将与 JDK 9 一起发货。

`jlink` 工具包括插件和扩展机制。当使用 `jlink` 生成应用程序镜像时，我们将与这些机制集成，以便 `jlink` 过程的输出是具有适当平台特定布局的应用程序镜像。这将产生一个有益的副作用，即使应用程序镜像生成不依赖于 Java 打包器过程。

**`javapackager` CLI 参数、Ant 任务和 Java 打包器 API**

Java 打包器新增了与 JEP 261 中指定的 Java 工具链选项语法和值相匹配的 CLI 参数：

```
--add-modules <module>(,<module>)*
--limit-modules <module>(,<module>)*
--module-path <path>(:<path>)*
-p <path>(:<path>)*
--module <module>/<classname>
-m <module>/<classname>
```

要指定长选项的参数，可以使用 --\<name>=\<value> 或 --\<name> \<value>。

注意：`--module-path` 与 jlink 的 `--module-path` 映射，但具有可选默认值。更多信息见下文。

将会有新的基于 、 以及新的任务的任务。

例如：

```
<fx:deploy outdir="${bundles.dir}"
           outfile="MinesweeperFX"
           nativeBundles="all"
           verbose="true">

    <fx:runtime strip-native-commands="false"> <-- new
        <fx:add-modules value="java.base"/>
        <fx:add-modules value="jdk.packager.services,javafx.controls"/>
        <fx:limit-modules value="java.sql"/>
        <fx:limit-modules value="jdk.packager.services,javafx.controls"/>
        <fx:module-path value="${java.home}/../images/jmods"/>
        <fx:module-path value="${build.dir}/modules"/>
    </fx:runtime>

    <fx:application id="MinesweeperFX"
                    name="MinesweeperFX"
                    module="fx.minesweeper" <-- new
                    mainClass="minesweeper.Minesweeper"
                    version="1.0">
    </fx:application>

    <fx:secondaryLauncher name="Test2"
                          module="hello.world" <-- new
                          mainClass="com.greetings.HelloWorld">
    </fx:secondaryLauncher>
</fx:deploy>
```

`<fx:runtime>`、`<fx:limit-modules>`、`<fx:add-modules>`、`<fx:modular-path>` 是可选参数。在捆绑模块化应用程序时，`<fx:application>` 上的 `module="module name"` 参数将被使用；否则，如果应用程序是非模块化应用程序，则该参数无效。参数 `<fx:limit-modules>`、`<fx:add-modules>`、`<fx:modular-path>` 可以与本文中使用的 `--add-mods`、`--limit-mods` 和 `--module-path` 互换。请参阅模块配置部分以获取有关模块参数的更多信息。

Java Packager API 将为模块化选项获取新方法。

**移除原生命令**

默认情况下，Java 打包器会移除如 java.exe 之类的命令，但一些开发者需要 java.exe 之类的命令行工具。因此，将提供一项选项，通过关闭移除命令的剥离来包含本地命令：

```
--strip-native-commands false
```

**添加对模块和模块路径的支持**

Jigsaw 在类路径之外引入了“模块路径”的概念。模块路径由库、JDK 模块和应用程序模块的路径组成。包含这些模块的路径通过命令行参数指定：

```
--module-path <path>(:<path>)*
```

它只能提供一次，并且是一个平台路径。根模块及其传递依赖项被链接以创建一个模块化运行时镜像（[JEP 220](http://openjdk.java.net/jeps/220)）。

开发者可以提供一个路径，将打包的模块与默认版本之外的 Java 运行时版本一起打包。如果开发者没有提供打包的 JDK 模块，则 Java 打包器将默认使用 Java 打包器所携带的 JDK 版本提供的打包模块（$JAVA\_HOME/jmods）。

目前 Java 打包器还没有提供一种机制将打包的模块复制到应用程序运行时镜像中，而不是链接到 `jimage`。这种场景最可能的需求是如果应用程序支持插件，并且这些模块位于打包的镜像之外。如果是这种情况，开发者需要通过用户 JVM 参数覆盖来覆盖--module-path 和--add-modules。

**模块配置**

将使用 Java 打包器打包两种类型的 Java 应用程序：非模块化 JAR 文件和模块化应用程序。

非模块化 JAR 包由一个没有包含 module-info.class 文件的 JAR 包组成。对于应用程序，使用 `-appClass` 和 `-BmainJar=`。 开发者将使用与 JDK 9 之前版本相同的参数使用 Java Packager，使用 `-srcfiles`、`-Bclasspath=`、`-appClass` 和 `-BmainJar=` 参数。为了向后兼容，不需要新的模块化参数，并且默认情况下嵌入的 Java 运行时会包含所有可重新分发的模块，因此捆绑的运行时大小不会减小。开发者可以使用 `--module-path`、`--add-modules` 和 `--limit-modules` 来包含第三方模块。

例如：

```
javapackager -deploy -v -outdir output -name HelloWorld -Bclasspath=hello.world.jar -native -BsignBundle=false -BappVersion=1.0 -Bmac.dmg.simple=true -srcfiles hello.world.jar -appClass HelloWorld -BmainJar=hello.world.jar
```

模块化应用程序由一个 JAR、展开的模块或包含 module-info.class 的打包模块组成。要与应用程序捆绑，必须指定 `--module` 和 `--module-path` 参数。 `--module` 与 `-appClass` 和 `-BmainJar=` 互斥。 `--module-path` 必须提供一个包含主模块（通过 `--module` 引用的模块）的路径。其他模块可以通过使用 `run-time image` 中的 `--add-modules` 和 `--limit-modules` 添加。通过核心反射或服务动态加载的模块必须使用 `--add-modules` 手动指定。主模块和通过 --add-modules 提供的模块将定义根模块。 `jlink` 将创建一个包含指定根模块及其传递依赖项的运行时镜像。

例如：

```
javapackager -deploy -v -outdir output -name Test -native -BsignBundle=false -BappVersion=1.0 -Bmac.dmg.simple=true --module-path /path/to/jmod --module hello.world/com.greetings.HelloWorld
```

此命令将生成一个包含主模块及其所有传递依赖项的运行时镜像。其他模块可以通过 --add-modules 选项添加。

**模块**

打包器将被拆分为两个模块：

```
jdk.packager
jdk.packager.services
```

jdk.packager 包含构建应用程序捆绑包和安装程序的 Java 打包器。jdk.packager.services 是一个与应用程序捆绑包捆绑在一起的模块，它可以在运行时提供对打包器服务的访问，例如 JVM 用户参数。

**JNLP**

生成的捆绑包将取决于输入和提供的选项。历史上，-deploy 会生成所有原生捆绑包和 .jnlp 文件。现在，与 -module 结合使用 -deploy 不会生成 .jnlp 文件，因为 JNLP 不支持新的模块化选项。-native 选项将生成所有可用的原生捆绑包。

## \[功能] JEP 276：[语言定义对象模型的动态链接](https://openjdk.org/jeps/276)

提供一种链接高级对象操作（如“读取属性”、“写入属性”、“调用可调用对象”等）的机制，这些操作以在 INVOKEDYNAMIC 调用站点中表达的名字表示。提供一种默认链接器，用于在普通 Java 对象上执行这些操作的常用语义，以及安装特定语言链接器的机制。

JEP 276 是为了更好地支持多语言互操作，特别是为动态语言（比如 JavaScript）在 JVM 上运行提供更强的动态链接能力。

## \[功能] JEP 277：[增强的弃用](https://openjdk.org/jeps/277)

### 摘要

重新设计 `@Deprecated` 注解，并提供加强 API 生命周期的工具。

**规范**

增强废弃的 `@Deprecated` 注解的主要目的是为工具提供更细粒度的信息，以了解 API 的废弃状态。这些工具随后使用该注解向 API 的用户报告信息。由于 `@Deprecated` 注解具有运行时保留功能，因此会消耗堆内存。因此，这里的信息应该是最小化和明确的。

以下元素将被添加到 `java.lang.Deprecated` 注解类型：

* 一个返回 `boolean` 类型的方法 `forRemoval()`。如果 `true`，则表示此 API 元素在未来版本中标记为删除。如果 `false`，则 API 元素已废弃，但目前没有计划在未来版本中删除它。此元素的默认值为 `false`。
* 一个名为 `since()` 的方法，返回 `String`。此字符串应包含此 API 被弃用的版本或版本号。它具有自由格式语法，但版本编号应遵循与包含弃用 API 的项目中的 `@since` Javadoc 标签相同的方案。请注意，此值与 Javadoc `@since` 标签不重复，因为后者记录了 API 被引入的版本，而 `since()` 方法记录的是 API 被弃用的版本。 `@Deprecated` 注解记录了 API 被弃用的版本。此元素的默认值为空字符串。

由于这些元素被添加到现有的 `@Deprecated` 注解中，因此如果处理的是使用低于 JDK 9 版本的编译器编译的类文件，注解处理程序将看到 `forRemoval()` 和 `since()` 的默认值。

API 上存在 `@Deprecated` 注解是 API 作者或维护者向 API 用户传达的信息。通常，弃用是建议用户迁移其使用方式，避免在编写新代码或维护旧代码时添加对这一 API 的依赖，或者使用依赖于这一 API 的代码存在一定风险。推荐这种迁移有许多原因。原因可能包括以下内容：

* **API 存在缺陷，且难以修复，**
* **使用该 API 可能会导致错误，**
* **该 API 已被另一个 API 取代，**
* **该 API 已过时，**
* **该 API 是实验性的，可能存在不兼容的更改，**
* **或者上述任何组合。**

废弃 API 的确切原因通常过于微妙，无法用注解中的标志或元素值来表述。强烈建议在 API 的文档注释中描述废弃 API 的原因。此外，还建议在文档中讨论并链接可能的替代 API。

提供了一个特定的标志值。如果 `forRemoval()` 布尔元素为 `true` ，则表示意图在项目的某个未来版本中删除 API 元素。因此，**API 的使用者会提前得到警告，如果他们不迁移到其他 API，他们的代码在升级到新版本时可能会出现错误**。如果 `forRemoval()` 是 `false` ，则表示建议迁移到已弃用的 API，但没有具体意图删除该 API。

`@Deprecated` 注解和 `@deprecated` javadoc 标签都应该在 API 元素上同时存在或同时不存在。一个存在而另一个不存在的情况被视为错误。如果 API 缺少 `@Deprecated` 注解而存在 `@deprecated` 标签， `javac` lint 标志 `-Xlint:dep-ann` 将发出警告。如果情况相反，则目前没有警告；请参阅 JDK-8141234。

@Deprecated 注解不应直接影响已弃用 API 的行为，并且应该对性能影响微乎其微。

**Java SE 中的使用**

@Deprecated 注解类型出现在 Java SE 中，因此它可能 应应用于使用 Java SE 平台的任何类库的 API。 那些类库如何使用这些规则的详细规则和政策由这些库的维护者决定。建议类库维护者制定并记录此类政策。 `@Deprecated` 注解类型的使用规则由这些库的维护者自行决定。建议类库维护者制定并记录此类政策。

本节描述了在 Java SE API 上使用 `@Deprecated` 注解类型的使用情况以及相关的政策。

将在多个 Java SE API 中添加、更新或删除 `@Deprecated` 注解。以下列出了 Java SE 9 中实现的变化。除非另有说明，否则此处列出的弃用项并非用于删除。请注意，这并非 Java SE 9 中所有弃用项的完整列表。

* 为 boxed primitives（如 `Boolean`、`Integer` 等）的构造函数添加 `@Deprecated`（[JDK-8145468](https://bugs.openjdk.java.net/browse/JDK-8145468)）
* 为 `Runtime.traceInstructions` 和 `Runtime.traceMethodCalls` 方法添加 `@Deprecated(forRemoval=true)`（[JDK-8153330](https://bugs.openjdk.java.net/browse/JDK-8153330)）
* 将 `@Deprecated` 添加到各种 `java.applet` 和相关类中（[JEP 289](http://openjdk.java.net/jeps/289)）
* 将 `@Deprecated` 添加到 `java.util.Observable` 和 `Observer` （[JDK-8154801](https://bugs.openjdk.java.net/browse/JDK-8154801)）
* 将 `@Deprecated(forRemoval=true)` 添加到各种被取代的安全 API 中，包括 `java.security.acl` （JDK-8157847）、 `javax.security.cert` 和 `com.sun.net.ssl` （JDK-8157712）、 `java.security.Certificate` （JDK-8157707）和 `javax.security.auth.Policy` （JDK-8157848）
* 向 `@Deprecated(forRemoval=true)` 添加到 `java.lang.Compiler` ([JDK-4285505](https://bugs.openjdk.java.net/browse/JDK-4285505))
* 向多个 Java EE 模块和 `java.corba` 模块添加 `@Deprecated` ([JDK-8169069](https://bugs.openjdk.java.net/browse/JDK-8169069)，[JDK-8181195](https://bugs.openjdk.java.net/browse/JDK-8181195)，[JDK-8181702](https://bugs.openjdk.java.net/browse/JDK-8181702)，[JDK-8174728](https://bugs.openjdk.java.net/browse/JDK-8174728))
* 修改已弃用的方法 `Thread.destroy()` 、 `Thread.stop(Throwable)` 、 `Thread.countStackFrames()` 、 `System.runFinalizersOnExit()` 以及各种不常用的 `Runtime` 和 `SecurityManager` 方法，使其具有 `@Deprecated(forRemoval=true)` (JDK-8145468)

考虑到 Java SE 中对弃用的历史以及强调跨版本的长期 API 兼容性，移除 API 是一个严重的问题。因此，只有当有明确的计划在下一个 Java SE 平台版本中移除该 API 时，才应应用带有 `forRemoval=true` 元素的弃用。

API 元素不应从 Java SE 规范中移除，除非它已带有 `@Deprecated(forRemoval=true)` 注解 在 Java SE 的早期版本中。对于弃用来说是可以接受的 引入了 `forRemoval=true`。无需先进行弃用处理。 `forRemoval=false`，然后升级到 `forRemoval=true`，在删除 API 之前。

对于 Java SE 9 及以上版本中弃用的 API 元素，`自` 元素应包含表示版本的 Java SE 版本字符串 的 API 元素已被弃用。版本字符串应 符合 [JEP 223](http://openjdk.java.net/jeps/223) 中指定的格式。由于 Java SE 通常仅在主要版本中进行规范变更，版本字符串通常仅包含 "MAJOR" 版本号。因此，对于在 Java SE 9 中弃用的 API 元素，`since` 元素值应简单地为 "9"。

在 Java SE 9 之前被弃用的 API 元素，其 `since` 值将仅在有时间时填写。（为所有 API 执行此操作的价值不大，主要是一项历史研究练习。）在这种情况下使用的 `since` 字符串应遵守用于那些版本的 `@since` javadoc 标签的 JDK 版本约定，通常是 `1.0` 到 `1.8`，有时带有 "micro" 版本号，例如 `1.0.2`。在寻找 Java SE API 上的此值并发现空字符串的注解处理工具应假定弃用发生在 Java SE 8 或更早版本。

废弃 API 将增加项目在针对 Java SE 的新版本构建时遇到的强制警告数量。一些项目，包括 JDK 本身，使用编译器选项来启用详细警告并将警告转换为错误。对于这类项目，将废弃的 API 添加到 Java SE 中可能会引入大量警告，这显著增加了迁移到 Java SE 新版本的难度。现有的管理警告的机制，如 `@SuppressWarnings` 注解和编译器命令行选项，不足以处理这个问题。这实际上限制了在特定 Java SE 版本中可以废弃的 API 数量，并使得废弃过时但流行的 API 几乎不可能实现。这要求未来努力增强管理废弃警告的机制。

**`forRemoval` 对警告策略的影响**

Java 语言规范第 9.6.4.6 节规定了依赖于 API（“声明位置”）的弃用状态以及使用该 API 的代码（“使用位置”）的弃用状态的具体警告行为。 `forRemoval` 元素的增加又引入了必须定义的另一组情况。为了简洁起见，我们将带有 `forRemoval=false` 的弃用称为“普通弃用”，带有 `forRemoval=true` 的弃用称为“终止弃用”。

在 Java SE 8 及之前版本中，`forRemoval` 不存在，因此唯一的弃用类型就是普通弃用。是否发出弃用警告取决于使用位置和声明位置的弃用状态。以下是 Java SE 8 中存在的情况表：

```
use site     | API declaration site
    context      | not dep.   deprecated
                 +-----------------------
    not dep.     |    N          W
                 |
    deprecated   |    N          N (1)

        N = no warning
        W = warning
```

（注1）这是一个特殊情况。如果使用位置和声明位置都已被弃用，则不会发出警告 。如果这两个位置都在一个维护的单个类库中，这就有意义了 作为一个单元发布。由于它们一起维护，发布它们几乎没有意义。 警告在此情况下。然而，如果使用位置在维护的类库中 与声明站点分开，它们可能以不同的速度发展，因此不发布 警告在这种情况下可能是一个误功能。然而，这个机制是有用的。 减少 JDK 编译时警告的数量，在引入 Java SE 5 中的 `@SuppressWarnings` 注解之前。

（JLS 9.6.4.6 还要求，如果使用位置在声明位置 _同一最外层&#x7C7B;_&#x5185;部，则不发出警告。在这种情况下，使用位置和声明位置在定义上是保持在一起的，因此不发出警告的理由同样适用。）

在 Java SE 9 中，引入了 `forRemoval`，这增加了与终止弃用相关的几个新情况。这需要引入一种新的警告类型。

在通常弃用的 API 使用点发出的警告是“普通弃用警告”，这与 Java SE 8 及之前版本相同。这些通常简单地被称为“弃用警告”，作为以前使用的遗留。

在终止弃用的 API 使用点发出的警告可能正式被称为“终止弃用警告”，但这相当冗长。相反，我们将此类警告称为“移除警告”。

建议的案例表如下所示：

```
use site     |      API declaration site
    context      | not dep.   ord. dep.   term. dep.
                 +----------------------------------
    not dep.     |    N         oW (2)       rW (5)
                 |
    ord. dep.    |    N          N (3)       rW (6)
                 |
    term. dep.   |    N          N (4)       rW (7)
```

(注 2) "oW" 指的是一种 "普通弃用警告"，与在 Java SE 8 及更早版本中发生的情况相同的警告。

（注 3）左上角的四个元素与 Java SE 8 表中的相同，出于向后兼容性的原因。

（注4）此处不会通过外推兼容行为发出警告。如果使用和声明位置都通常是废弃的，那么将使用位置更改为最终废弃而引入警告将是荒谬的。因此，在这种情况下不会发出警告。

（注 5）“rW”指的是“移除警告”。所有在最终废弃的 API 的使用站点发布的警告都是移除警告。

（注 6）这个案例非常关键。我们总是希望使用最终废弃的 API 能够生成移除警告，即使使用站点位于废弃的代码中。

（注7）这与（6）类似。有人可能会认为，由于使用和声明站点都是最终废弃的，两者都将“消失”，因此在这里发布警告是没有意义的。但是，可能的情况是声明站点位于一个比使用站点演变更快的库中，因此使用站点可能会比声明站点存在的时间更长。因此，关于声明站点即将被移除的警告是必要的。

涵盖右下角四个元素的一般规则如下。如果使用站点被废弃，无论是通常还是最终废弃，都不会发布常规的废弃警告，但仍然会发布移除警告。

一个普通的弃用警告示例可能如下所示：

```
UseSite.java:3: warning: [deprecation] ordinary() in DeclSite has been deprecated
```

一个移除警告的示例可能如下所示：

```
UseSite.java:4: warning: [removal] removal() in DeclSite has been deprecated and marked for removal
```

警告的具体措辞以及警告的自定义机制可能因编译器而异。

**抑制弃用警告**

在 Java SE 8 及之前版本中，可以通过在使用位置标注 `@SuppressWarnings("deprecation")` 来抑制弃用警告。在终端弃用存在的情况下，此行为需要修改。

考虑一种情况，即使用站点依赖于通常已弃用的 API，并且该警告已被 `@SuppressWarnings("deprecation")` 注释抑制。如果声明站点被修改为最终弃用，我们希望在即使使用站点已抑制警告的情况下，仍然会在使用站点发生删除警告。如果在这种情况下不发出新警告，则可能发生 API 最终弃用然后删除，而其使用站点没有任何警告。

以下场景说明了问题。假设 `@SuppressWarnings("deprecation")` 注解旨在抑制普通弃用警告以及删除警告。然后，可能会发生以下情况：

1. 网站 X 依赖于 API Y，目前尚未弃用
2. Y 的声明更改为普通弃用，在 X 处生成普通弃用警告
3. X 被注解为 `@SuppressWarnings("deprecation")` ，抑制了警告
4. Y 的声明更改为最终弃用；X 处的移除警告仍然被抑制
5. Y 被完全移除，导致 X 突然崩溃

既然弃用（deprecation）的目的是传达关于 API 进化的信息，特别是关于 API 移除的信息，那么在这种情况下没有任何警告是一个严重的问题。**因此，当弃用从普通弃用升级为最终弃用时，即使在该使用位置的警告之前已被抑制，也应该给出警告。**

我们需要一个与当前用于抑制普通弃用警告的机制不同的机制来抑制移除警告。解决方案是使用 `@SuppressWarnings` 注解中的不同字符串。

可以使用注解来抑制移除警告——这些警告是由使用最终弃用 API 引起的。

```
@SuppressWarnings("removal")
```

此注解仅抑制删除警告，而不是普通弃用警告。我们曾考虑将其做成一种强抑制形式，涵盖普通弃用警告和删除警告。然而，这可能导致错误。程序员可能会使用 `@SuppressWarnings("removal")` 来抑制普通弃用的警告。如果将普通弃用更改为最终弃用，这将防止警告出现，导致最终弃用的 API 最终被删除时出现意外的破坏。

如前所述，可以使用注解来抑制使用通常已弃用的 API 产生的警告

```
@SuppressWarnings("deprecation")
```

如上所述，此注解仅抑制普通弃用警告；它不会抑制移除警告。

如果需要在特定位置同时抑制普通弃用警告和移除警告，可以使用以下结构：

```
@SuppressWarnings({"deprecation", "removal"})
```

下面是上一节中警告表的副本，已修改以显示如何抑制不同情况下的警告。

```
use site     |      API declaration site
    context      | not dep.   ord. dep.   term. dep.
                 +----------------------------------
    not dep.     |    -        @SW(d)       @SW(r)
                 |
    ord. dep.    |    -           -         @SW(r)
                 |
    term. dep.   |    -           -         @SW(r)

        @SW(d) = @SuppressWarnings("deprecation")
        @SW(r) = @SuppressWarnings("removal")
```

如果在最终弃用 API 的使用位置使用 `@SuppressWarnings("removal")` 抑制移除警告，并且该 API 被更改为普通弃用，那么出现普通弃用警告似乎有些奇怪。然而，我们预计 API 从最终弃用回到普通弃用的演变路径相当罕见。

## \[测试] JEP 278：[G1 中针对巨型对象的附加测试](https://openjdk.org/jeps/278)

为 G1 垃圾收集器的“巨无霸对象”功能开发额外的白盒测试。

## \[测试] JEP 279：[改进测试失败故障排除](https://openjdk.org/jeps/279)

自动收集可用于进一步故障排除的诊断信息，以应对测试失败和超时情况。

## \[JVM] JEP 280：[Indify 字符串连接](https://openjdk.org/jeps/280)

### 摘要

将由 `javac` 生成的静态 `String` -concatenation 字节码序列更改为使用 `invokedynamic` JDK 库函数调用。这将使未来对 `String` 连接的优化无需对 `javac` 生成的字节码进行进一步更改。

### 描述

我们将利用 `invokedynamic` 的力量：它提供了懒加载链接的设施，通过在初始调用期间一次启动调用目标。这种方法并不新颖，我们从当前将 lambda 表达式转换为代码的代码中广泛借鉴。

想法是将整个 `StringBuilder` 拼接舞蹈替换为一个简单的 `invokedynamic` 调用 `java.lang.invoke.StringConcatFactory` ，该调用将接受需要拼接的值。例如，

```
String m(String a, int b) {
  return a + "(" + b + ")";
}
```

目前编译为：

```
java.lang.String m(java.lang.String, int);
       0: new           #2                  // class java/lang/StringBuilder
       3: dup
       4: invokespecial #3                  // Method java/lang/StringBuilder."<init>":()V
       7: aload_1
       8: invokevirtual #4                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      11: ldc           #5                  // String (
      13: invokevirtual #4                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      16: iload_2
      17: invokevirtual #6                  // Method java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
      20: ldc           #7                  // String )
      22: invokevirtual #4                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      25: invokevirtual #8                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      28: areturn
```

即使是建议实现中可用的简单 indy 翻译，也可以显著简化：

```
java.lang.String m(java.lang.String, int);
       0: aload_1
       1: ldc           #2                  // String (
       3: iload_2
       4: ldc           #3                  // String )
       6: invokedynamic #4,  0              // InvokeDynamic #0:makeConcat:(Ljava/lang/String;Ljava/lang/String;ILjava/lang/String;)Ljava/lang/String;
      11: areturn

BootstrapMethods:
  0: #19 invokestatic java/lang/invoke/StringConcatFactory.makeConcat:(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/String;[Ljava/lang/Object;)Ljava/lang/invoke/CallSite;
```

注意我们传递了一个 `int` 参数而没有装箱。在运行时，引导方法（BSM）运行并链接实际执行拼接的代码。它将 `invokedynamic` 调用重写为适当的 `invokestatic` 调用。这从常量池中加载了常量字符串，但我们可以利用 BSM 静态参数直接将这些和其他常量传递给 BSM 调用。这就是建议的 `-XDstringConcat=indyWithConstants` 风味所做的事情：

```
java.lang.String m(java.lang.String, int);
       0: aload_1
       1: iload_2
       2: invokedynamic #2,  0              // InvokeDynamic #0:makeConcat:(Ljava/lang/String;I)Ljava/lang/String;
       7: areturn

BootstrapMethods:
  0: #15 invokestatic java/lang/invoke/StringConcatFactory.makeConcatWithConstants:(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/String;[Ljava/lang/Object;)Ljava/lang/invoke/CallSite;
    Method arguments:
      #16 \u0001(\u0001)
```

注意我们只将动态参数（ `"a"` 和 `"b"` ）传递给 BSM。静态常量将在链接过程中处理。BSM 方法提供了一个配方，说明了如何按顺序连接动态和静态参数，以及静态参数是什么。这种策略还分别处理 null、原始类型和空字符串。

哪种字节码风格应该是默认的，这是一个悬而未决的问题。关于字节码形状、连接风格和性能数据，更多详细信息可以在实验笔记中找到。这里可以找到提出的引导方法 API。完整的实现可以在 sandbox 分支中找到：

```
$ hg clone http://hg.openjdk.java.net/jdk9/sandbox sandbox 
$ cd sandbox/ 
$ sh ./common/bin/hgforest.sh up -r JDK-8085796-indyConcat
$ sh ./configure 
$ make images
```

可以通过以下方式查看基线和修补后的运行时的差异：

```
$ hg diff -r default:JDK-8085796-indyConcat
$ cd langtools/
$ hg diff -r default:JDK-8085796-indyConcat
$ cd jdk/
$ hg diff -r default:JDK-8085796-indyConcat
```

基准测试可以在以下链接找到：[http://cr.openjdk.java.net/\~shade/8085796/](http://cr.openjdk.java.net/~shade/8085796/)

建议的实现成功构建了 JDK，运行了回归测试（包括测试 `String` 连接的新测试），并在所有平台上通过了烟雾测试。实际连接可以使用多种策略。我们的建议实现表明，当我们把由 `javac` 生成的字节码序列移动到注入相同字节码的 BSM 中时，没有出现吞吐量下降，这验证了该方法。优化策略的表现与基线相当或更好，尤其是在默认 `StringBuilder` 长度不足或 VM 优化失败时。

### 说明

JEP 280 通过把字符串拼接改为使用 `invokedynamic`，让 JVM 能动态选择最优实现，从而提升性能。在 **Java 9 及以后**，**使用 `+` 拼接通常和手写的 `StringBuilder.append()` 差不多快，甚至在某些情况下更快**。所以现在推荐的做法是：

> **写起来简单的 `+`，让 JVM 帮你做优化！**

**📦 背景：Java 字符串拼接原来怎么做的？**

在 Java 8 之前，编译器（javac）对字符串拼接：

```
String s = "a" + b + c + "d";
```

会编译成：

```
new StringBuilder()
    .append("a")
    .append(b)
    .append(c)
    .append("d")
    .toString();
```

这种方式虽然 OK，但有几个问题：

* 每次拼接都硬编码为 `StringBuilder`，不能利用 JVM 的优化能力。
* 对某些场景（比如拼接常量、使用 `String.concat` 更快）没法自动适配。
* 不容易进行逃逸分析或更激进的 JIT 优化。

**⚡ JEP 280 做了什么优化？**

从 Java 9 开始，JEP 280 引入了 **`invokedynamic` 字符串拼接机制**：

```
String s = "a" + b + c + "d";
```

不再直接编译成 `StringBuilder`，而是：

```
invokedynamic "makeConcatWithConstants"
```

这个调用点由 JVM 在运行时绑定，会交给一个叫做 **`java.lang.invoke.StringConcatFactory`** 的类去决定用什么方式来拼接字符串。

**➕ 这有什么好处？**

* JVM 可以根据具体场景选择更高效的拼接方式：
  * 使用 `StringBuilder`
  * 使用 `String.concat`
  * 使用预分配 char\[]
* JIT 编译器可以优化拼接路径，比如消除临时对象。
* 对常量拼接更容易做成编译期优化。

**🔬 举个例子**

```
String s = "Hello, " + name + "! Your score is " + score;
```

Java 8 编译后：

```
new StringBuilder()
    .append("Hello, ")
    .append(name)
    .append("! Your score is ")
    .append(score)
    .toString();
```

Java 9+ 编译后：

```
invokedynamic "makeConcatWithConstants" 
  -> StringConcatFactory::makeConcatWithConstants
```

由 JVM 运行时绑定出最优实现。

**📈 实际性能表现**

* 对于大量字符串拼接的场景（如日志系统、模板渲染），JEP 280 的优化能显著减少临时对象和 GC 压力；
* 对于简单拼接，性能差异可能不大，但为 JIT 和逃逸分析打开了优化空间；
* 你 **不需要改任何代码**，只要升级到 Java 9+，拼接性能自然会提升。

**👨‍💻 对开发者的影响？**

| 问题                             | 回答                            |
| ------------------------------ | ----------------------------- |
| 我要改代码吗？                        | ❌ 不用，Javac 自动处理               |
| 有兼容性风险吗？                       | ❌ 没有，运行时自动降级为 `StringBuilder` |
| 我能手动用 `StringConcatFactory` 吗？ | ✅ 可以，但很少有必要，除非写类库或做性能测试       |
| Java 8 会用这个优化吗？                | ❌ 不会，Java 9 才引入的              |

**✅ 总结**

| 项目   | 内容                                   |
| ---- | ------------------------------------ |
| 名称   | JEP 280: Indify String Concatenation |
| 做了什么 | 将字符串拼接改为 `invokedynamic` 调用          |
| 目的   | 提升运行时拼接性能、减轻 GC 压力                   |
| 好处   | 更高效的拼接、更灵活的优化、更少对象创建                 |
| 影响   | 编码方式不变，性能自然受益（Java 9+）               |

## \[JVM] JEP 281：[HotSpot C++单元测试框架](https://openjdk.org/jeps/281)

启用并鼓励为 HotSpot 开发 C++单元测试。

## \[JVM] JEP 282：[jlink：Java 链接器](https://openjdk.org/jeps/282)

### 摘要

创建一个可以组装和优化一组模块及其的工具体 将依赖项打包到自定义运行时镜像中，该镜像定义于 [JEP 220](http://openjdk.java.net/jeps/220)。

### 描述

链接工具 `jlink` 的基本调用方式是：

```
$ jlink --module-path <modulepath> --add-modules <modules> --limit-modules <modules> --output <path>
```

where:

* `--module-path` 是链接器发现可观察模块的路径；这些可以是模块化的 JAR 文件、JMOD 文件或展开的模块
* `--add-modules` 指定要添加到运行时镜像的模块；这些模块可以通过传递依赖关系导致添加额外的模块
* `--limit-modules` 限制可观察模块的宇宙
* `--output` 是包含结果运行时图像的目录

`--module-path`、`--add-modules` 和 `--limit-modules` 选项的详细信息请参阅 [JEP 261](http://openjdk.java.net/jeps/261)。

`jlink` 将支持的其它选项包括：

* `--help` 打印使用/帮助信息
* `--version` 打印版本信息

## \[UI] JEP 283：[在 Linux 上启用 GTK 3](https://openjdk.org/jeps/283)

启用基于 JavaFX、Swing 或 AWT 的 Java 图形应用程序在 Linux 上使用 GTK 2 或 GTK 3。

## \[JVM] JEP 284：[新的 HotSpot 构建系统](https://openjdk.org/jeps/284)

使用 build-infra 框架重写 HotSpot 构建系统。

## JEP 285：[旋转等待提示](https://openjdk.org/jeps/285)

### 摘要

定义一个 API，允许 Java 代码提示正在执行自旋循环。

### 说明

你提到的是 [JEP 285: Spin-Wait Hints](https://openjdk.org/jeps/285)，这是一个很有意思的底层优化功能。

**🧠 什么是“自旋循环”（Spin Loop / Spin Wait）？**

**自旋循环**是一种高性能并发编程技巧，指的是**一个线程在短时间内反复检查某个条件是否成立，而不进入阻塞状态**。

**✍️ 举个例子：**

```
while (!flag) {
    // do nothing, just wait
}
```

这个线程会不断检查 `flag` 是否为 `true`，**但它不会阻塞（比如 wait/sleep）**，而是在 CPU 上“原地踏步”。

> 这就叫做“自旋” —— 线程在原地空转，等待条件成立。

**❓ 为什么要这么做？**

因为在某些场景下：

* 条件**很快**就会成立；
* 线程阻塞再唤醒的代价（系统调用、上下文切换）太高；
* 所以“原地自旋”等待反而更快。

**🚀 JEP 285 做了什么？**

JEP 285 引入了一个新方法：

```
java.lang.Thread.onSpinWait()
```

这是一个 **自旋提示方法（spin-wait hint）**，它告诉 CPU：

> “我这个线程正在自旋，不需要真正干活，你可以对我做点优化，比如降低功耗或者让其他核心早点抢占我。”

**✅ 这个方法怎么用？**

你可以在自旋循环中加入 `Thread.onSpinWait()`，比如：

```
while (!flag) {
    Thread.onSpinWait();
}
```

相比于什么都不写，这样：

* **更节能**（CPU 会省电、省资源）；
* **更高效**（现代 CPU 可以优化流水线执行）；
* **更具可移植性**（JVM 会在不同平台使用对应的 CPU 指令，比如 x86 上可能是 `PAUSE`）。

**👀 `onSpinWait()` 到底做了什么？**

这个方法在不同平台会被 JIT 编译器翻译为对应的 CPU 指令：

| 平台  | 转换指令          |
| --- | ------------- |
| x86 | `PAUSE`       |
| ARM | `YIELD`       |
| 其他  | 可能是空操作，或者平台相关 |

这可以帮助 CPU **减少功耗**、**缓解缓存一致性压力**、**提升 SMT（超线程）性能**。

**📦 典型使用场景**

1. **无锁队列的 poll 操作**
2. **环形缓冲区等待数据**
3. **内存屏障之后的同步等待**
4. **用户态实现的轻量级锁**

**❌ 使用上的注意点**

* 只用于 **非常短暂的等待**（微秒级别）；
* 如果等待时间不可预测，应该考虑 `LockSupport.park()` 或 `Thread.sleep()`；
* 加了 `onSpinWait()` 并不会**改变逻辑**，只是个 **性能提示**。

**✅ 总结**

| 问题                         | 答案                         |
| -------------------------- | -------------------------- |
| 什么是自旋？                     | 原地反复检查条件，不进入阻塞状态           |
| `Thread.onSpinWait()` 是什么？ | 自旋提示，告诉 JVM 和 CPU 当前线程正在忙等 |
| 有啥用？                       | 优化性能、降低功耗、提升资源利用           |
| 何时用？                       | 自旋等待时，条件马上可能成立的场景          |
| 会改变代码行为吗？                  | ❌ 不会，只是性能 hint             |

## \[功能] JEP 287：[SHA-3哈希算法](https://openjdk.org/jeps/287)

实现 NIST FIPS 202 中指定的 SHA-3 加密散列函数（仅支持 BYTE 类型）。

## \[优化] JEP 288：[禁用 SHA-1 证书](https://openjdk.org/jeps/288)

## \[优化] JEP 289：[弃用 Applet API](https://openjdk.org/jeps/289)

## \[功能] JEP 290：[过滤传入的序列化数据](https://openjdk.org/jeps/290)

### 摘要

允许过滤传入的对象序列化数据流，以提高安全性和健壮性。

### 描述

核心机制是一个由序列化客户端实现的过滤器接口，该接口被设置在 `ObjectInputStream` 上。在反序列化过程中，会调用过滤器接口的方法来验证正在反序列化的类、创建的数组大小，以及描述流长度、流深度和引用数量的指标。过滤器返回一个状态以接受、拒绝或保留状态未决。

对于流中的每个新对象，在对象实例化和反序列化之前，都会调用过滤器，并传入对象的类。对于原始数据类型或流中具体编码的 `java.lang.String` 实例，不会调用过滤器。对于每个数组，无论它是原始数据类型数组、字符串数组还是对象数组，都会传入数组类和数组长度。对于已从流中读取的对象的每个引用，都会调用过滤器，以便它可以检查深度、引用数量和流长度。如果启用了日志记录，则过滤器操作将记录到 `java.io.serialization` 日志记录器中。

对于 RMI，对象通过设置在 `UnicastServerRef` 上的过滤器来导出，该过滤器对 `MarshalInputStream` 进行过滤，以验证反序列化时的调用参数。通过 `UnicastRemoteObject` 导出对象应支持设置用于反序列化的过滤器。

### 说明

**JEP 290** 的目的是为了解决传统 Java 序列化机制的一些安全隐患，特别是反序列化过程中潜在的 **远程代码执行** 和 **反序列化漏洞**。

**问题：**

传统的 Java 序列化机制允许反序列化来自不可信来源的数据。攻击者可以构造恶意数据流，通过 **反序列化** 恶意对象来执行任意代码（比如反序列化恶意对象触发类加载）。

**解决方案：**

JEP 290 通过引入 **过滤机制**，对输入的序列化数据进行检查，避免潜在的恶意操作。具体来说，它会在反序列化数据之前，先通过一个过滤器检查该数据是否安全，防止不受信任的数据执行危险操作。

**🔧 如何使用 JEP 290？**

在 JEP 290 中，引入了一个新的系统属性和接口，使得开发者可以自定义过滤机制。

**关键要素：**

1. **过滤机制**：
   * 使用 `ObjectInputStream` 时，可以启用数据流过滤。
   * 通过 `java.io.ObjectInputFilter` 接口可以对传入的序列化数据进行过滤，决定哪些类是可以反序列化的。
2. **启用过滤**：
   * 你可以通过设置 `-Dcom.sun.serialFilter=<filter>` 来启用序列化数据过滤功能。该属性设置过滤器类（你可以实现一个自定义过滤器）。
3. **内置过滤器**：
   * JEP 290 引入了一些内置的过滤器，可以帮助你选择合适的安全级别。
   * 比如 `ObjectInputFilter.Config` 提供了一些配置项来启用过滤。

**基本使用步骤：**

1.  **实现自定义过滤器**（`ObjectInputFilter`）：

    你可以创建一个自定义过滤器，来指定哪些类或对象是允许反序列化的。示例如下：

    ```
    import java.io.ObjectInputStream;
    import java.io.ObjectInputFilter;

    public class MyObjectInputFilter implements ObjectInputFilter {
        @Override
        public Status checkInput(FilterInfo filterInfo) {
            // 只允许反序列化某些类型的数据
            if (filterInfo.serialClass() != null && filterInfo.serialClass().getName().startsWith("com.mycompany")) {
                return Status.ALLOWED;
            }
            return Status.REJECTED;
        }
    }
    ```
2.  **配置过滤器**： 你可以在启动应用时设置过滤器：

    ```
    java -Dcom.sun.serialFilter=com.mycompany.MyObjectInputFilter MyApp
    ```

    或者在代码中通过 `ObjectInputStream.setObjectInputFilter()` 方法来动态设置过滤器：

    ```
    ObjectInputStream ois = new ObjectInputStream(inputStream);
    ois.setObjectInputFilter(new MyObjectInputFilter());
    ```
3. **启用全局过滤器**： 你也可以通过 JVM 启动时的 `-D` 参数，设置全局的序列化过滤器。

***

**🛡️ 安全性提升：**

JEP 290 的最大好处是：

* **减少反序列化漏洞**：攻击者无法将恶意的、未受信任的数据流传递给反序列化机制。你可以限制反序列化对象的类范围，从而避免潜在的安全漏洞。
* **可配置的灵活性**：你可以通过配置文件、JVM 参数或代码中实现过滤机制，按需启用或禁用某些类的反序列化。

***

**🧑‍💻 实际开发中怎么使用 JEP 290：**

1. **开发时，考虑启用序列化过滤：** 如果你的应用中有反序列化需求，强烈建议启用 JEP 290 来增加安全性。尤其是在接收来自网络或外部源的数据时，使用过滤器可以有效避免恶意反序列化攻击。
2. **创建和注册自定义过滤器：** 如果你的应用只允许某些类进行反序列化，可以根据 JEP 290 提供的 API 来创建过滤器。你可以指定允许反序列化的类，或者排除不需要的类，从而增强安全性。
3. **适配不信任的数据源：** 如果你的应用需要处理外部用户上传的数据（如反序列化外部传来的文件），你可以使用过滤器确保这些数据来源是可信的。确保外部数据不会导致恶意操作。

***

**✅ 总结：**

| 项目          | 内容                                     |
| ----------- | -------------------------------------- |
| **JEP 290** | 通过过滤序列化输入，增强反序列化的安全性                   |
| **作用**      | 防止反序列化漏洞，避免恶意数据执行危险操作                  |
| **如何使用**    | 使用 `ObjectInputFilter` 创建自定义过滤器，配置过滤规则 |
| **开发者推荐**   | 任何涉及反序列化的代码，应该考虑使用过滤器来增强安全性            |

## \[优化] JEP 291：[弃用并发标记清除（CMS）垃圾收集器](https://openjdk.org/jeps/291)

## \[JVM] JEP 292：[在 Nashorn 中实现选定的 ECMAScript 6 功能](https://openjdk.org/jeps/292)

## \[JVM] JEP 294：[Linux/s390x 端口](https://openjdk.org/jeps/294)

## \[JVM] JEP 295：[提前编译](https://openjdk.org/jeps/295)

## \[JVM] JEP 297：[统一 arm32/arm64 端口](https://openjdk.org/jeps/297)

## \[优化] JEP 298：[删除演示和样本](https://openjdk.org/jeps/298)

## \[DOC] JEP 299：[重新组织文档](https://openjdk.org/jeps/299)
