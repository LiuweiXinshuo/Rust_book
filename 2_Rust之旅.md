# Rust 之旅

​	在本章中，我们将看几个简短的程序，以了解 Rust 的语法，类型和语义如何组合在一起以支持安全，并发和高效的代码。 我们将逐步完成 Rust 的下载和安装过程，展示一些简单的数学代码，试用基于第三方库的 Web 服务器，并使用多个线程来加快绘制 Mandelbrot 集的过程。



### 下载并安装Rust

安装Rust的最佳方法是使用Rust安装程序rustup。 转到 https://rustup.rs 并按照那里的说明进行操作。
您也可以转到 https://www.rust-lang.org，单击“下载”，然后获取适用于Linux，macOS和Windows的预构建软件包。 Rust也包含在某些操作系统发行版中。 我们更喜欢rustup，因为它是用于管理Rust安装的工具，例如Ruby的RVM或Node的NVM。 例如，当发布新版本的Rust时，您可以通过键入 `rustup update` 击来升级.

无论如何，完成安装后，您应该在命令行中使用三个新命令：

```shell
$ cargo --version
cargo 0.18.0 (fe7b0cdcf 2017-04-24)
$ rustc --version
rustc 1.17.0 (56124baa9 2017-04-24)
$ rustdoc --version
rustdoc 1.17.0 (56124baa9 2017-04-24)
$
```

在这里，$是命令提示符；在此笔录中，我们运行安装的三个命令，要求每个命令报告它是哪个版本。 依次执行每个命令：

* cargo 是 Rust 的编译经理，程序包经理和通用工具。 您可以使用Cargo启动新项目，构建和运行程序，以及管理代码所依赖的任何外部库。
* rustc 是 Rust 编译器。 通常，我们让Cargo为我们调用编译器，但有时直接运行它会很有用。
* rustdoc 是 Rust 文档工具。 如果您在程序源代码中以适当形式的注释编写文档，则rustdoc可以从中构建格式精美的HTML。 像rustc一样，我们通常让Cargo为我们运行rustdoc。

为方便起见，Cargo可以为我们创建一个新的Rust包，并适当安排了一些标准元数据：

```shell
$ cargo new --bin hello
	Created binary (application) `hello` project
```

此命令将创建一个名为hello的新程序包目录，并且 `--bin` 标志指示Cargo将其准备为可执行文件，而不是库文件。 查看软件包的顶级目录：

```shell
$ cd hello
$ ls -la
total 24
drwxrwxr-x. 4 jimb jimb 4096 Sep 22 21:09 .
drwx------. 62 jimb jimb 4096 Sep 22 21:09 ..
drwxrwxr-x. 6 jimb jimb 4096 Sep 22 21:09 .git
-rw-rw-r--. 1 jimb jimb 7 Sep 22 21:09 .gitignore
-rw-rw-r--. 1 jimb jimb 88 Sep 22 21:09 Cargo.toml
drwxrwxr-x. 2 jimb jimb 4096 Sep 22 21:09 src
$
```

我们可以看到Cargo创建了一个文件Cargo.toml来保存包的元数据。 目前，该文件包含的内容不多：

```toml
[package]
name = "hello"
version = "0.1.0"
authors = ["You <you@example.com>"]

[dependencies]

```

如果我们的程序获得了对其他库的依赖关系，则可以将其记录在此文件中，并且Cargo将为我们下载，构建和更新这些库。 我们将在第8章中详细介绍 Cargo.toml 文件。

Cargo已设置我们的软件包以与git版本控制系统一起使用，创建了 .git 元数据子目录和 .gitignore 文件。 您可以通过在命令行上指定 `--vcs none` 来告诉 Cargo 跳过此步骤。

src 子目录包含实际的Rust代码：

```shell
$ cd src
$ ls -l
total 4
-rw-rw-r--. 1 jimb jimb 45 Sep 22 21:09 main.rs
```

Cargo 已代表我们编写初始程序。 main.rs 文件包含以下文本：

```rust
fn main() {
    println!("Hello, world!");
}
```

在Rust中，您甚至不需要编写自己的 “Hello，world!" 程序。 这就是新的Rust程序的样板范围：两个文件，总共九行。
我们可以从包中的任何目录调用cargo run命令来构建和运行程序：

```shell
$ cargo run
 Compiling hello v0.1.0 (file:///home/jimb/rust/hello)
 Finished dev [unoptimized + debuginfo] target(s) in 0.27 secs
 Running `/home/jimb/rust/hello/target/debug/hello`
Hello, world!
$
```

在这里，Cargo调用了Rust编译器 rustc，然后运行它生成的可执行文件。 Cargo将可执行文件放在包顶部的目标子目录中：

```shell
$ ls -l ../target/debug
total 580
drwxrwxr-x. 2 jimb jimb 4096 Sep 22 21:37 build
drwxrwxr-x. 2 jimb jimb 4096 Sep 22 21:37 deps
drwxrwxr-x. 2 jimb jimb 4096 Sep 22 21:37 examples
-rwxrwxr-x. 1 jimb jimb 576632 Sep 22 21:37 hello
-rw-rw-r--. 1 jimb jimb 198 Sep 22 21:37 hello.d
drwxrwxr-x. 2 jimb jimb 68 Sep 22 21:37 incremental
drwxrwxr-x. 2 jimb jimb 4096 Sep 22 21:37 native
$ ../target/debug/hello
Hello, world!
$
```

完成后，Cargo可以为我们清理生成的文件：

```shell
$ cargo clean
$ ../target/debug/hello
bash: ../target/debug/hello: No such file or directory
$
```



### 一个简单函数

Rust的语法故意不是原始的。 如果您熟悉C，C++，Java、JavaScript，则可能会找到Rust程序的一般结构。 这是一个使用Euclid算法计算两个整数的最大公约数的函数：

```rust
fn gcd(mut n: u64, mut m: u64) -> u64 {
    assert!(n != 0 && m != 0);
    while m != 0 {
        if m < n {
            let t = m;
            m = n;
            n = t;
        }
        m = m % n;
    }
    n
}
```

`fn` 关键字引入了一个函数。 在这里，我们定义了一个名为 `gcd` 的函数，该函数带有两个参数 `n` 和 `m`，每个参数的类型均为 `u64`（一个无符号的64位整数）。 `->` 标记位于返回类型之前：我们的函数返回 `u64` 值。 四空格缩进是标准的Rust样式。

Rust 的计算机整数类型名称反映了它们的大小和符号性：`i32` 是带符号的 32位整数； `u8` 是一个无符号的八位整数（用于“字节”值），依此类推。`isize` 和 `usize` 类型保存指针大小的有符号和无符号整数，在32位平台上为32位长，在64位平台上为64位长。 Rust还具有两个浮点类型，即 `f32` 和 `f64`，它们是IEEE单精度和双精度浮点类型，例如 C和C++ 中的 float 和 double。

默认情况下，变量一旦初始化，就无法更改其值，但可以在参数 `n` 和 `m` 之前放置 `mut` 关键字，以允许我们的函数体为其分配值。 实际上，大多数变量不会分配给； 那些关键字上的 `mut` 关键字可以在阅读代码时提供帮助。

该函数的主体从调用 `assert!` 宏开始 ，验证两个参数都不为零！ 字符将其标记为宏调用，而不是函数调用。 像 C 和 C++ 中的 `assert` 宏一样，Rust的 `assert！` 检查其参数是否为真，如果不正确，则通过一条有用的消息终止程序，包括失败检查的源位置； 这种突然终止被称为恐慌。 与可以跳过断言的 C和C++不同，Rust始终检查断言，无论程序如何编译。 还有一个`debug_assert！` 宏，为提高速度而编译程序时，将跳过其断言。

我们函数的核心是一个 `while` 循环，其中包含一个 `if` 语句和一个赋值。 与 C 和 C++不同，Rust不需要在条件表达式两边加上括号，但是需要在它们控制的语句两边加上花括号。

`let` 语句声明一个局部变量，例如我们函数中的`t`。 只要Rust可以从变量的使用方式中推断出 `t` 的类型，我们就不需要写出它的类型。 在我们的函数中，唯一适用于 `t` 的类型是 `u64`，匹配 `m` 和 `n` 。 Rust仅会推断函数体内的类型：您必须像以前一样写出函数参数的类型并返回值。 如果我们想拼写出 `t` 的类型，可以这样写：

```rust
	let t: u64 = m;
```

Rust有一个 `return` 语句，但是 `gcd `函数不需要一个。 如果函数主体以不带分号的表达式结尾，则该函数的返回值。 实际上，任何用花括号括起来的块都可以用作表达式。 例如，这是一个打印消息并随后产生 `x.cos()` 作为其值的表达式：

```rust
    {
        println!("evaluating cos x");
        x.cos()
    }
```

在Rust中，通常会在控件“掉到尾”时使用这种形式来确定函数的值，并且仅将 `return` 语句用于从函数中间的显式早期返回。



### 编写和运行单元测试

Rust对语言内置的测试提供了简单的支持。 要测试我们的 `gcd` 函数，我们可以编写：

```rust
#[test]
fn test_gcd() {
    assert_eq!(gcd(14, 15), 1);
    assert_eq!(gcd(2 * 3 * 5 * 11 * 17,
        3 * 7 * 11 * 13 * 19),
        3 * 11);
}
```

在这里，我们定义了一个名为 `test_gcd` 的函数，该函数调用 `gcd` 并检查其是否返回正确的值。 定义顶部的 `#[test]` 将 `test_gcd` 标记为测试函数，在常规编译中会跳过，但如果我们使用 cargo test 命令运行程序，则会自动包含并调用该函数。 假设我们已经将 `gcd` 和 `test_gcd` 定义编辑到了本章开头创建的 hello 包中。 如果我们的当前目录位于包的子树中，则可以按以下方式运行测试：

```shell
$ cargo test
	Compiling hello v0.1.0 (file:///home/jimb/rust/hello)
		Finished dev [unoptimized + debuginfo] target(s) in 0.35 secs
		Running /home/jimb/rust/hello/target/debug/deps/hello-2375a82d9e9673d7
		
running 1 test
test test_gcd ... ok
test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured
$
```

我们可以将测试功能分散在我们的源代码树中，放置在它们执行的代码旁边，然后货物测试将自动收集它们并运行它们。

`#[test]` 标记是属性的示例。 属性是一个开放式系统，用于使用附加信息标记函数和其他声明，例如 C++和C# 中的属性或 Java 中的注释。 它们用于控制编译器警告和代码样式检查，有条件地包含代码（例如C和C++中的`#ifdef`），告诉Rust如何与用其他语言编写的代码进行交互，等等。 我们将会看到更多关于属性的示例。



### 处理命令行参数

如果我们希望程序将一系列数字用作命令行参数并打印其最大公约数，则可以将main函数替换为以下内容：

```rust
use std::io::Write;
use std::str::FromStr;

fn main() {
    let mut numbers = Vec::new();
    
    for arg in std::env::args().skip(1) {
        numbers.push(u64::from_str(&arg)
            .expect("error parsing argument"));
    }
    
    if numbers.len() == 0 {
        writeln!(std::io::stderr(), "Usage: gcd NUMBER
            ...").unwrap();
        std::process::exit(1);
    }
    
    let mut d = numbers[0];
    for m in &numbers[1..] {
        d = gcd(d, *m);
    }
    
    println!("The greatest common divisor of {:?} is {}",
        numbers, d);
}
```

这是一大段代码，所以让我们逐段介绍一下：

`use std::io::Write;
use std::str::FromStr;`

使用声明将两个特征 `Write` 和 `FromStr` 纳入了范围。 我们将在第11章中详细介绍特征，但是现在我们只说特征是类型可以实现的方法的集合。 尽管我们从不在程序中的其他位置使用名称 `Write` 或`FromStr` ，但必须使用`trait` 才能使用其方法。 在当前情况下：

* 任何实现 `Write` 特征的类型都具有 `write_fmt` 方法，该方法将格式化的文本写入流中。 `std::io::Stderr` 类型实现 `Write` ，我们将使用`writeln!` 宏以打印错误消息； 该宏扩展为使用`write_fmt` 方法的代码。
* 任何实现 `FromStr` 特征的类型都有一个 `from_str` 方法，该方法尝试从字符串中解析该类型的值。 `u64` 类型实现了 `FromStr`，我们将调用 `u64::from_str` 来解析我们的命令行参数。

再来看主函数功能，我们的 `main` 函数没有返回值，因此我们可以简单地省略 `->` 并键入通常会紧随参数列表的类型。

```rust
let mut numbers = Vec::new();
```

我们声明一个可变的局部变量号，并将其初始化为一个空向量。 `Vec` 是Rust的可增长向量类型，类似于C++ 的 `std::vector` ，Python `list` 或JavaScript `array` 。 即使将向量设计为可以动态增长和收缩，我们仍然必须标记Rust的可变 `mut` ，以便我们将数字推到其末尾。

数字类型是 `Vec<u64>`，它是 `u64` 值的向量，但是和以前一样，我们不需要将其写出来。 Rust会为我们推断出这一点，部分原因是我们将向量推入的是 `u64` 值，还因为我们将向量的元素传递给了 `gcd`，后者仅接受 `u64` 值。

```rust
for arg in std::env::args().skip(1) {
```

在这里，我们使用for循环来处理命令行参数，依次将变量 `arg` 设置到每个参数，并评估循环体。

`std::env::args` 函数返回一个迭代器，该迭代器是一个根据需要产生每个参数的值，并指示完成的时间。 迭代器在Rust中无处不在。 标准库还包括其他迭代器，这些迭代器生成矢量的元素，文件的行，在通信通道上接收的消息以及几乎所有有意义的循环。 Rust的迭代器非常有效：编译器通常能够将它们转换为与手写循环相同的代码。 我们将在第15章中演示其工作原理并给出示例。

除了与 `for` 循环配合使用外，迭代器还包含多种可以直接使用的方法。 例如，由 `std::env::args` 返回的迭代器产生的第一个值始终是正在运行的程序的名称。 我们想跳过它，因此我们调用迭代器的 `skip` 方法来生成一个新的迭代器，该迭代器将忽略第一个值。

```rust
numbers.push(u64::from_str(&arg)
    .expect("error parsing argument"));
```

在这里，我们调用 `u64::from_str` 尝试将命令行参数 `arg` 解析为无符号的64位整数。 `u64::from_str` 是一种与 `u64` 类型相关联的函数，而不是我们要获取的某些 `u64` 值所使用的方法，类似于 C++或 Java中的静态方法。 `from_str` 函数不会直接返回 `u64`，而是返回一个 `Result` 值，该值指示解析成功还是失败。 结果值是两个变量之一：

* 写入Ok(v) 的值，表示解析成功，并且v是产生的值
* 写入Err(e) 的值，表示解析失败，e是说明原因的错误值

执行输入或输出或与操作系统进行交互的功能都返回Result类型，其Ok变量携带成功的结果（传输的字节数，打开的文件等），而 Err 变量携带系统的错误代码。 与大多数现代语言不同，Rust也不例外：所有错误都使用结果或恐慌来处理，如第7章所述。

我们使用 `Result` 的 `Expect` 方法检查解析是否成功。 如果结果是 `Err(e)` ，则 `Expect` 将显示一条包含 `e` 的描述的消息，并立即退出程序。 但是，如果结果为 `Ok(v)` ，则期望仅返回 `v` 本身，我们最终可以将其推入数字向量的末尾。

```rust
if numbers.len() == 0 {
    writeln!(std::io::stderr(), "Usage: gcd NUMBER ...").unwrap();
    std::process::exit(1);
}
```

空数集没有没有元素，因此我们检查向量是否至少包含一个元素，如果没有，则退出程序并出现错误。 我们使用 `writeln!`宏，将错误消息写入由 `std::io::stderr()` 提供的标准错误输出流。 `.unwrap()` 调用是一种检查打印错误消息的尝试本身是否失败的简便方法。 期望调用也可以，但这可能不值得。

```rust
let mut d = numbers[0];
for m in &numbers[1..] {
    d = gcd(d, *m);
}
```

此循环使用 `d` 作为其运行值，并对其进行更新以使其保持到目前为止我们处理过的所有数字的最大公约数。 和以前一样，我们必须将 `d` 标记为可变的，以便可以在循环中将其赋值。

`for` 循环有两个令人惊讶的地方。 首先，我们写了 `for m in &numbers[1..]`;  `&` 运算符的作用是什么？ 其次，我们写了 `gcd(d, *m);  `  `*m` 中的 `*` 是做什么用的？ 这两个细节是互补的。

到目前为止，我们的代码仅对简单值（如适合固定大小的内存块的整数）进行运算。 但是现在我们要遍历向量，该向量可以是任何大小，可能非常大。 Rust处理此类值时要谨慎：它希望让程序员可以控制内存消耗，明确每个值的寿命，同时仍确保在不再需要时立即释放内存。

因此，当我们进行迭代时，我们要告诉Rust，向量的所有权应保留为数字； 我们只是在循环中借用它的元素。 `&numbers[1..]` 中的 `&` 运算符从第二个开始借用对该向量元素的引用。 `for` 循环遍历引用的元素，让 `m` 依次借用每个元素。 `*m` 中的 `*` 运算符取消对 `m` 的引用，产生它所引用的值； 这是我们要传递给 `gcd` 的下一个 `u64` 。 最后，由于数字拥有矢量，因此当数字超出 main 范围时，Rust会自动释放它。

Rust 的所有权和引用规则是 Rust 的内存管理和安全并发的关键； 我们将在第4章及其对应的第5章中详细讨论它们。您需要熟悉这些规则才能熟悉Rust，但是对于本入门教程，您需要知道的是 `&x` 借用了 `x` 的引用。 ，`*r` 是参考 `r` 所引用的值。

```rust
println!("The greatest common divisor of {:?} is {}", numbers, d);
```

遍历数字元素之后，程序将结果打印到标准输出流中。`println!`  宏采用模板字符串，用 `{...}` 格式的其余参数的格式版本替换为模板字符串中出现的格式，然后将结果写入标准输出流。

与 C和 C++不同，C和C++ 要求 main 如果程序成功完成则返回零，而如果出问题则返回非零退出状态，Rust假定如果 main 全部返回，则程序成功完成。 只有显式地调用诸如 `Expect` 或 `std::process::exit` 之类的函数，我们才能使程序以错误状态代码终止。

`cargo run` 命令允许我们将参数传递给程序，因此我们可以尝试命令行处理：

```shell
$ cargo run 42 56
	Compiling hello v0.1.0 (file:///home/jimb/rust/hello)
	Finished dev [unoptimized + debuginfo] target(s) in 0.38 secs
	Running `/home/jimb/rust/hello/target/debug/hello 42 56`
The greatest common divisor of [42, 56] is 14
$ cargo run 799459 28823 27347
	Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
	Running `/home/jimb/rust/hello/target/debug/hello 799459 28823 27347`
The greatest common divisor of [799459, 28823, 27347] is 41
$ cargo run 83
	Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
	Running `/home/jimb/rust/hello/target/debug/hello 83`
The greatest common divisor of [83] is 83
$ cargo run
	Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
	Running `/home/jimb/rust/hello/target/debug/hello`
Usage: gcd NUMBER ...
$
```

在本节中，我们使用了 Rust 标准库中的一些功能。 如果您对其他功能感到好奇，我们强烈建议您试用Rust的在线文档。 它具有实时搜索功能，使探索变得容易，甚至包括指向源代码的链接。 当您安装Rust本身时，`rustup` 命令会在您的计算机上自动安装一个副本。 您可以使用以下命令在浏览器中查看标准库文档：

```shell
$ rustup doc --std
```



### 一个简单的Web服务器

Rust 的优势之一是可在 crates.io 网站上免费获得的图书馆软件包集合。 `cargo` 命令使我们自己的代码可以轻松使用 crates.io 软件包：它将下载正确版本的软件包，构建并按要求更新。 一个Rust包，无论是库还是可执行文件，都称为 `crate` ； Cargo和crates.io都以此术语命名。

为了展示其工作原理，我们将使用 `iron` Web 框架， `hyper` HTTP Server以及它们所依赖的其他各种 crates 来组合一个简单的Web服务器。 如图2-1所示，我们的网站将提示用户输入两个数字，并计算其最大公约数。

