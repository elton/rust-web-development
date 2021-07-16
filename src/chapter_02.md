# 基础

Rust 是一种复杂的语言，但您不必在开始时甚至在更大的项目中了解所有的来龙去脉。 编译器和其他工具（例如我们将在第 5 章介绍的 Clippy）将极大地帮助以干净漂亮的方式完成代码。 因此，我们不会涵盖语言的所有方面，但是当您遇到这些主题时，您会自信地自行查找主题。

要在 Rust 中自信地工作，您必须掌握以下技能：

- 如何通过 [docs.rs](https://docs.rs/)（Rust 官方文档）查找类型和行为
- 快速迭代您面临的错误或问题
- Rust 所有权系统如何运作
- 什么是宏，如何发现和使用它们
- 通过 `struts` 创建自己的类型并通过 `impl` 实现行为
- 如何在现有类型上实现 `traits` (特征)和 `macros` (宏)
- 带有 `Options` 和 `Results` 的函数式 Rust

## 基础知识

创建一个问答应用。

### 使用 struts 为你的资源建模

基于问答(Q&A)应用，需要创建以下模型：

- Users
- Questions
- Answers

下面是创建和实现自己模型的例子：
![img](https://i.loli.net/2021/07/16/RjEpJuUIi7gcGXd.jpg)

```rust
struct Question {
    id: QuestionId,
    title: String,
    content: String,
    tags: Option<Vec<String>>,
}
struct QuestionId(String);

impl Question {
    fn new(id: QuestionId, title: String, content: String,
                                          tags: Option<Vec<String>>) -> Self {
        Question {
            id,
            title,
            content,
            tags,
        }
    }
}
```

Rust 的函数签名规则：

![img](https://i.loli.net/2021/07/16/BQ8LYPv67DZwgpI.jpg)

Rust 的构造函数没有默认名称，所以最好的做法是使用一个名为 `new` 的方法

### 理解 Options

Rust 中的 Options 是特殊的，它们让我们确保我们不会在期望值的地方使用 null。 通过 Option 枚举，我们可以随时检查提供的值是否存在，并且可以处理不存在的情况。 更好的是，当我们手头有一个 Option 枚举时，编译器确保我们总是涵盖所有情况（一些或无）。 这也让我们可以声明不需要的字段，因此在创建新问题时，我们不需要提供标签列表，但如果我们愿意，我们可以提供。

> 可以随时使用 [Rust Playground](https://play.rust-lang.org/) 来检验自己的代码。

通常使用`match`关键字来检查 Option 的值

```rust
fn main() {
    struct Book {
        title: String,
        is_used: Option<bool>,
    }

    let book = Book { title: "Great book".to_string(), is_used: None };

    match book.is_used {
        Some(v) => println!("Book is used: {}", v),
        None => println!("We don’t know if the book is used"),
    }
}
```

Rust 有两种 `struct` 方法：

- 关联函数 (Associated function)
- 方法 (Method)

关联函数不将 `&self` 作为参数，并使用两个双冒号 (`::`) 调用。 方法采用 `&self` 并通过点 (`.`) 简单地调用。
![img](https://i.loli.net/2021/07/16/jTqeaPpgfyJYrNx.jpg)

### 字符串处理

Rust 中 String 和字符串切片 (`&str`) 之间的主要区别在于 `String` 可以调整大小。 `String` 是字节的集合，它被实现为一个向量。 我们可以查看源代码中的定义。

```rust
pub struct String {
    vec: Vec<u8>,
}
```

字符串是通过 `String::from(“popcorn”);` 创建的。 并且可以在创建后进行更改。 我们看到 `String` 下面实际上是一个向量，这意味着我们可以根据需要在这个向量中删除和插入 `u8` 值。

> **Stack vs. Heap** <br/>
> 堆栈通常由程序控制，每个线程都有自己的堆栈。 它存储简单的指令或变量。 基本上，具有固定大小的所有内容都可以存储在堆栈中。 堆更独特（尽管可以有多个堆）并且在堆上操作更昂贵。 数据没有固定大小，可能是数据被拆分到多个块上，因此读取可能需要更多时间。

`&str`（字符串字面值）是 `u8` 值（文本）的表示，我们无法修改它。 正如我们在图中看到的，两个文本都位于堆上，但在堆栈上分配了不同的指针。

在 Rust 中，原始类型存储在堆栈中，而更复杂的类型存储在堆中； `String` 和 `&str` 指向更复杂的数据类型（UTF-8 值的集合），而 `&str` 有一个胖指针，显示堆上的地址
![img](https://i.loli.net/2021/07/16/e6OmInuZFdqgGSB.jpg)

通过 `struct` 结构创建新数据类型时，通常会创建 `String` 字段类型。在函数中处理文本时，通常使用 `&str`。

### 所有权

```rust
fn main() {
    let x = String::from(“hello”);
    let y = x;

    println!("{}", x);
}
```

编译器将报下面错误：

```rust
error[E0382]: borrow of moved value: `x`
 --> src/main.rs:5:20
  |
2 |     let x = String::from("hello");
  |         - move occurs because `x` has type `String`, which does not implement the `Copy` trait
3 |     let y = x;
  |              - value moved here
4 |
5 |     println!("{}", x);
  |                    ^ value borrowed here after move
```

- `&str` 类型是一个固定大小的值，我们可以存储在栈上，如果我们把这个值赋给另一个值，我们可以简单地复制它，知道没有人可以修改它的值，所以指向它是安全的 .
- `String` 类型更复杂，可以在创建后更改。 如果两个或多个变量可以改变堆栈中的值，我们将不得不以某种方式管理不是两个变量都可以改变底层值。此外，如果我们从堆栈中删除该值，其他变量仍将指向它。因此我们需要确保变量只有一个唯一的所有者，而不是两个或更多。

因此，将值的所有权从 `x` 转移到 `y`。 在 Rust 中，当一个值可以更改时，它一次只能有一个唯一的所有者。 因此，通过将所有权从 `x` 转移到 `y`，`x` 超出范围并且 Rust 在其上调用了一个名为 `.drop()` 的内部方法。

当将复杂类型重新分配给新变量时，Rust 会复制指针信息并将所有权授予新变量。旧的不再需要并且超出范围
![img](https://i.loli.net/2021/07/16/I7BaRY3DFzPgtsh.jpg)

原始类型默认实现了 `Copy` trait，每当你重新分配一个变量时，值都会被简单地复制，并且两个变量都存在，直到函数体完成。 在处理复杂类型时，只有一个指针拥有可以修改它的值的所有权。

请注意，即使将变量传递给函数，也会将此变量的所有权转移给该函数。 当这个函数超出范围时，我们在堆上的值也会丢失，数据也会丢失。

### 使用和实现 trait
