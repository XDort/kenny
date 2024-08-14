#### 1、CoreCLR 运行时环境 c++编写 作用 

 （1）中间代码解析

源代码（如 C# 或 VB.NET）首先被编译成 IL。IL 是一种与平台无关的中间语言

CoreCLR 负责加载和执行 IL 代码，并将其编译成本机代码，这样应用程序可以在特定的硬件平台上运行

（2）中间代码编译 JIT

CoreCLR 使用即时（JIT）编译器将 IL 代码编译成特定于目标计算机体系结构的本机代码。JIT 编译器在运行时根据需要动态地将 IL 方法编译成本机代码。这种方法使得 CoreCLR 能够针对当前运行环境和硬件平台进行优化，从而提高应用程序的性能和效率。

（3）保证类型安全

每个类型都有严格定义的元数据，CoreCLR 在加载程序集时会读取和验证这些元数据，确保类型的使用符合定义。

JIT在把IL编译成机器代码时会检查类型操作

（4）异常处理 try catch finally

（5）线程管理 托管线程配合垃圾回收机制

CoreCLR 的托管线程是运行在操作系统线程之上的。CLR 会使用少量的操作系统线程来管理和调度多个托管线程。

CLR 的线程池是托管线程的一部分，它负责分配和回收线程资源

（6）GC 回收托管对象内存

采用代际垃圾回收算法，结合可达性分析和标记-清除算法



##### CoreFX 与 .NET Standard

CoreFX 是.NET Core 的核心功能库，它包含了.NET Core 运行时所需的基本组件和标准库。具体来说，CoreFX 提供了.NET Core 应用程序所需的基本类型、集合、IO 操作、网络通信、安全性、并发和异步编程等核心功能

CoreFX 是特定于某个具体.NET 运行时的核心功能库，而 .NET Standard 则是一种跨平台的API规范，用于确保在不同的.NET 平台上的互操作性和兼容性。



##### CLI Common Language Infrastructure

是IL中间语言的一套技术标准



##### Roslyn 一个编译平台



##### ASP.NET Core开发者指南

https://github.com/MoienTajik/AspNetCore-Developer-Roadmap/blob/master/ReadMe.zh-Hans.md



