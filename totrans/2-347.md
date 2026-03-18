# Rust 入门：安装和您的第一个 CLI 工具——入门指南

> 原文：[`towardsdatascience.com/get-started-with-rust-installation-your-first-cli-tool-a-beginners-guide/`](https://towardsdatascience.com/get-started-with-rust-installation-your-first-cli-tool-a-beginners-guide/)

<mdspan datatext="el1747165949018" class="mdspan-comment">Rust 近年来已成为一种流行的编程语言，因为它结合了安全性和高性能，并且可用于许多应用。它结合了 C 和 C++的积极特性，以及 Python 等现代编程语言的语法简洁性。在本文中，我们将逐步介绍在各个操作系统上安装 Rust 的过程，并设置一个简单的命令行界面，以了解 Rust 的结构和功能。

## 安装 Rust——逐步进行

由于官方安装程序`rustup`在 Rust 网站（[`www.rust-lang.org/`](https://www.rust-lang.org/)）上免费提供，因此无论操作系统如何，安装 Rust 都非常简单。这意味着安装只需几个步骤，并且对于不同的操作系统只有细微的差异。

### 在 Windows 下安装 Rust

在 Windows 中，安装程序完全控制安装过程，您可以按照以下步骤进行：

1.  访问 Rust 官方网站上的“安装”子部分（[`www.rust-lang.org/tools/install`](https://www.rust-lang.org/tools/install)）并下载`rustup-init.exe`文件。该网站会识别底层操作系统，从而直接提供适用于所用系统的适当建议。

1.  下载完成后，即可执行`rustup-init.exe`文件。随后将打开一个包含各种安装说明的命令行。

1.  按下 Enter 键以运行标准安装来安装 Rust。这还包括以下工具：

    +   `rustc`是编译器，它在执行前编译代码并检查错误。

    +   `cargo`是 Rust 的构建和包管理工具。

    +   `rustup`是版本管理器。

安装成功后，Rust 应自动在您的`PATH`中可用。这可以通过以下命令在 PowerShell 或 CMD 中轻松检查：

```py
rustc --version cargo --version
```

如果输出中显示了“rustc”和“cargo”及其相应的版本号，则表示安装成功。然而，如果找不到命令，可能是因为环境变量。要检查这些变量，您可以按照以下路径进行：“此电脑 –> 属性 –> 高级系统设置 –> 环境变量”。一旦到达那里，您应该确保 Rust 的路径，例如“C:\Users\UserName\.cargo\bin”，存在于`PATH`变量中。

### 在 Ubuntu/Linux 下安装 Rust

在 Linux 中，Rust 可以通过终端完全安装，无需从 Rust 网站下载任何内容。要安装 Rust，必须执行以下步骤：

1.  打开终端，例如，使用 Ctrl + Alt + T 键组合。

1.  要安装 Rust，执行以下命令：

```py
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

3. 你将被问及是否应该开始安装。这可以通过输入“1”（默认）来确认。然后，所有必需的包都会被下载，环境会被设置。

4. 你可能需要手动设置路径。在这种情况下，你可以使用以下命令，例如：

```py
source $HOME/.cargo/env
```

安装完成后，你可以检查一切是否正常工作。为此，你可以显式地显示 rustc 和 cargo 的版本：

```py
rustc --version cargo --version
```

### 在 macOS 下安装 Rust

在 macOS 上安装 Rust 有几种方法。如果你已经安装了 Homebrew，你可以简单地使用它来安装 Rust，执行以下命令：

```py
brew install rustup rustup-init
```

或者，你也可以直接使用此脚本安装 Rust：

```py
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

在安装过程中，你会被问及是否想要运行标准安装。你可以简单地按 Enter 键确认。无论选择哪种变体，你都可以通过显示 Rust 的版本来检查安装，以确保一切正常：

```py
rustc --version 
cargo --version
```

## 使用 cargo 创建 Rust 项目

在安装 Rust 的过程中，你可能已经遇到了 `cargo` 程序。这是 Rust 的官方包管理器和构建系统，与 Python 中的 `pip` 相当。`cargo` 执行以下任务，以及其他任务：

+   项目初始化

+   依赖管理

+   编译代码

+   测试执行

+   构建优化

这允许你在 Rust 中管理完整的项目，无需处理复杂的构建脚本。它还帮助你快速轻松地设置新项目，然后可以填充内容。

对于我们的示例，我们将创建一个新的项目。为此，我们进入终端并导航到我们想要保存项目的文件夹。然后执行以下命令来创建一个新的 Rust 项目：

```py
cargo new json_viewer --bin
```

我们称这个项目为 `json_viewer`，因为我们正在构建一个 CLI 工具，可以用来打开和处理 [JSON](https://databasecamp.de/en/data/json-en) 文件。`--bin` 选项表示我们想要创建一个可执行程序而不是库。你现在应该能够在执行命令后看到以下文件夹结构在你的目录中：

```py
json_viewer/ 
├── Cargo.toml # Project configuration 
└── src 
     └── main.rs # File for Rust Code
```

每个新的项目都有这样的结构。`Cargo.toml` 包含了项目所有的依赖和元数据，例如名称、使用的库或版本。另一方面，`src/main.rs`，随后包含实际的 Rust 代码，它定义了程序启动时执行的步骤。

首先，我们可以在终端中定义一个简单的函数来生成输出：

```py
fn main() { 
     println!("Hello, Rust CLI-Tool!"); 
}
```

程序可以通过终端使用 `cargo` 轻易地调用：

```py
cargo run
```

为了使这个调用生效，必须确保你位于项目的根目录中，即 `Cargo.toml` 文件存储的位置。如果一切设置和执行正确，你将收到以下输出：

```py
Hello, Rust CLI-Tool!
```

通过这些简单的步骤，你刚刚创建了一个成功的第一个 Rust 项目，我们将在下一节中继续构建。

## 构建 CLI 工具：简单的 JSON 解析器

现在，我们开始让项目充满活力，创建一个可以读取 JSON 文件并以结构化方式在终端输出其内容的程序。

第一步是定义依赖项，即我们在项目过程中将使用的库。这些存储在`Cargo.toml`文件中。在 Rust 中，所谓的 crate 可以与提供某些预定义功能的库或模块相比较。例如，它们可以由其他开发者编写的可重用代码组成。

我们的项目需要以下 crate：

+   `serde`使数据格式（如[JSON](https://databasecamp.de/en/data/json-en)或[YAML](https://databasecamp.de/en/data/yaml-file)）的序列化和反序列化成为可能。

+   另一方面，`serde_json`是一个专门为处理 JSON 文件而开发的扩展。

为了让项目能够访问这些 crate，它们必须存储在`Cargo.toml`文件中。创建项目后，它看起来是这样的：

```py
[package] 
name = "json_viewer" 
version = "0.1.0" 
edition = "2021" 

[dependencies]
```

我们现在可以在`[dependencies]`部分添加所需的 crate。在这里，我们也定义了要使用的版本：

```py
[dependencies] 
serde = "1.0" 
serde_json = "1.0"
```

为了确保添加的依赖项在项目中可用，它们必须首先下载并构建。为此，可以在主目录中执行以下终端命令：

```py
cargo build
```

在执行过程中，`cargo`会在 Rust 的中心仓库 crates.io 中搜索依赖项和指定的版本以下载它们。然后，这些 crate 将与代码一起编译并缓存，以便在下次构建时无需再次下载。

如果这些步骤都成功了，我们现在就可以编写实际的 Rust 代码来打开和处理 JSON 文件了。为此，你可以打开`src/main.rs`文件，并用以下代码替换现有内容：

```py
use std::fs;
use serde_json::Value;
use std::env;

fn main() {
    // Check arguments
    let args: Vec<String> = env::args().collect();
    if args.len() < 2 {
        println!(“Please specify the path to your file.”);
        return;
    }

    // Read in File
    let file_path = &args[1];
    let file_content = fs::read_to_string(file_path)
        .expect(“File could not be read.”);

    // Parse JSON
    let json_data: Value = serde_json::from_str(&file_content)
        .expect(“Invalid JSON format.”);

    // Print JSON
    println!(" JSON-content:\n{}”, json_data);
}
```

代码遵循以下步骤：

1.  检查参数：

    +   我们通过`env::args()`从命令行读取参数。

    +   用户必须在启动时指定 JSON 文件的路径。

1.  读取文件：

    +   通过`fs::read_to_string()`的帮助，文件内容被读取到一个字符串中。

1.  解析 JSON：

    +   `serde_json` crate 将字符串转换为 Rust 对象，其类型为 Value。

1.  格式化输出：

    +   内容在控制台中清晰输出。

为了测试这个工具，例如，你可以在项目目录下创建一个名为`examples.json`的测试文件：

```py
{
  "name": "Alice",
  "age": 30,
  "skills": ["Rust", "Python", "Machine Learning"]
}
```

程序随后使用`cargo run`执行，并定义了 JSON 文件的路径：

```py
cargo run ./example.json
```

这就标志着我们 Rust 的第一个项目的结束，我们成功构建了一个简单的命令行工具，它可以读取 JSON 文件并将它们输出到命令行。

## 这是你应该带走的东西

+   在许多操作系统中，安装 Rust 既快又简单。其他所需的组件已经安装。

+   在`cargo`的帮助下，可以直接创建一个空项目，该项目包含必要的文件，在其中你可以开始编写 Rust 代码

+   在开始编程之前，你应该输入依赖项并使用构建命令下载它们。

+   现在一切都已经设置好了，你可以开始实际的编程了。
