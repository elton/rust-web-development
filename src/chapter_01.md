# Why Rust?

Rust 提供了性能（它没有运行时也没有垃圾收集）、安全性（编译器确保一切都是内存安全的，即使在异步环境中）和生产力（它围绕测试、文档和包管理器的内置工具使它构建和维护轻而易举）。

## Rust 工具链

`rustup` 工具将使您能够安装 `Rust` 和其他辅助组件。

```bash
$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Rust 堆栈本身（标准库和 Rust 编译器）是用 Rust 编写的。 编译器本身使用 LLVM 来创建机器代码（LLVM 也被 Swift 和 Scala 等语言使用，并将语言编译器生成的字节码转换为操作系统的机器可读代码，并在其上运行）。

![`rustup`安装后的结构](https://i.loli.net/2021/07/16/2OB3i5WqoT6vYfr.jpg)

### 安装组件

```bash
$ rustup component add rustfmt
```

所有开发`rust`需要的开发工具
![所有 Rust 需要的工具](https://i.loli.net/2021/07/16/PtI2A73JDs1lur8.jpg)

### Rust 编译器

![img](https://i.loli.net/2021/07/16/6t3mfE2nTbLOxcV.jpg)

### 开发 Web Services 所需要的 Rust 组件

在 HTTP 方面，Rust 并没有像 Go 或 NodeJS 那样涵盖那么多的领域。 由于它是一种系统编程语言，因此 Rust 社区决定将实现 HTTP 和其他功能的工作留给社区。

![img](https://i.loli.net/2021/07/16/pkwKsFf3Q5ENX6m.jpg)

为了应对异步请求，Rust 社区提供了下面解决方案：

![img](https://i.loli.net/2021/07/16/1cxZEBFLidvgRTD.jpg)
