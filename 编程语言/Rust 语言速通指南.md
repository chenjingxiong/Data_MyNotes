---
title: Rust 语言速通指南
date: 2026-04-04
tags:
  - rust
  - 编程语言
  - 系统编程
  - 速通指南
---

# Rust 语言速通指南

> 一份面向有编程基础的开发者的 Rust 快速入门指南，覆盖核心概念、关键语法和实战要点。

---

## 1. Rust 是什么

- **系统级编程语言**，性能媲美 C/C++，同时保证内存安全
- **零成本抽象**：高级特性不带来运行时开销
- **无 GC**：通过所有权（Ownership）系统管理内存
- 由 Mozilla 发起，现由 Rust Foundation 维护
- 连续多年 Stack Overflow "最受喜爱语言" 第一

**适用场景**：系统工具、WebAssembly、嵌入式、CLI 工具、网络服务、区块链

---

## 2. 环境安装

### 2.1 安装 Rust（推荐 rustup）

**Linux / macOS：**

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
# 按提示选择默认安装即可
# 安装完成后重启终端，或执行：
source "$HOME/.cargo/env"
```

**Windows 安装（详细步骤）：**

1. **安装 C++ 构建工具**（Rust 在 Windows 上依赖 MSVC 链接器）：
   - 下载 [Visual Studio Build Tools](https://visualstudio.microsoft.com/visual-cpp-build-tools/)
   - 安装时勾选 **"使用 C++ 的桌面开发"** 工作负载
   - 或者安装完整版 [Visual Studio Community](https://visualstudio.microsoft.com/vs/)（免费），同样勾选 C++ 工作负载

2. **下载并运行 rustup 安装器**：
   - 访问 [https://rustup.rs](https://rustup.rs) 下载 `rustup-init.exe`
   - 双击运行，会出现命令行安装界面：

   ```
   Welcome to Rust!
   1) Proceed with standard installation (default)
   2) Customize installation
   3) Cancel installation
   >1                    # 直接回车选默认即可
   ```

3. **选择 ABI**（安装器会自动检测已安装的 MSVC，一般无需手动选择）：
   - `x86_64-pc-windows-msvc` —— 推荐，性能最佳，需要 MSVC
   - `x86_64-pc-windows-gnu` —— 使用 MinGW，无需 MSVC，但兼容性稍差

4. **验证安装**：

   ```powershell
   # 重新打开终端（PowerShell 或 CMD），执行：
   rustc --version
   cargo --version
   rustup --version
   ```

5. **环境变量**（通常安装器会自动配置，如果命令找不到）：
   ```powershell
   # 检查 PATH 是否包含（通常自动添加）：
   # %USERPROFILE%\.cargo\bin
   # %USERPROFILE%\.rustup
   ```

> [!tip] Windows 小贴士
> - 推荐使用 **Windows Terminal + PowerShell 7** 作为开发终端
> - 如果用 **WSL2** 开发，在 WSL 内按 Linux 方式安装即可，两套环境互不干扰
> - `cargo build` 产出的 `.exe` 可直接双击运行，无需额外运行时

**包管理器安装（可选）：**

```powershell
# Scoop
scoop install rustup
rustup-init

# Chocolatey
choco install rustup.install

# winget
winget install Rustlang.Rustup
```

### 2.2 更新与卸载

```bash
rustup update      # 更新到最新稳定版
rustup self uninstall  # 卸载
```

### 2.3 工具链管理

```bash
rustup default stable          # 使用稳定版
rustup default nightly         # 使用 nightly（实验特性）
rustup target add x86_64-unknown-linux-musl  # 交叉编译目标
```

---

## 3. Cargo 项目管理

Cargo 是 Rust 的构建工具 + 包管理器，相当于 npm/pip/cargo 一体。

```bash
# 创建新项目
cargo new my_project
cd my_project

# 项目结构
# my_project/
# ├── Cargo.toml      # 项目配置（类似 package.json）
# ├── src/
# │   └── main.rs     # 入口文件
# └── target/         # 编译输出（自动生成）

# 构建 & 运行
cargo build          # 编译（debug 模式）
cargo build --release # 编译（release 优化）
cargo run             # 编译并运行
cargo check           # 只检查语法，不生成二进制（更快）

# 测试
cargo test

# 格式化 & 检查
cargo fmt             # 自动格式化代码
cargo clippy          # 代码静态分析（linter）
```

### Cargo.toml 示例

```toml
[package]
name = "my_project"
version = "0.1.0"
edition = "2021"

[dependencies]
serde = { version = "1.0", features = ["derive"] }
tokio = { version = "1", features = ["full"] }
reqwest = "0.12"
```

---

## 4. 基础语法速查

### 4.1 变量与可变性

```rust
let x = 5;           // 不可变（默认）
let mut y = 10;      // 可变
y = 20;              // OK

const MAX: u32 = 100; // 常量，必须标注类型

let z: i32 = -42;    // 显式类型标注

// 变量遮蔽（shadowing）—— 不同于 mut！
let x = x + 1;       // 新变量，可以改变类型
let x = "now string"; // 合法！
```

### 4.2 数据类型

```rust
// 标量类型
let a: i8 = -1;           // 有符号整数 i8, i16, i32, i64, i128
let b: u8 = 255;          // 无符号整数 u8, u16, u32, u64, u128
let c: f64 = 3.14;        // 浮点数 f32, f64
let d: bool = true;       // 布尔
let e: char = '中';       // Unicode 字符（4 字节）

// 复合类型
let tup: (i32, f64, &str) = (500, 6.4, "hello");
let (x, _, z) = tup;      // 解构，_ 忽略
let first = tup.0;         // 索引访问

let arr: [i32; 5] = [1, 2, 3, 4, 5];
let arr2 = [0; 10];        // [0, 0, ..., 0] 重复初始化
let slice = &arr[1..3];    // 切片 [2, 3]
```

### 4.3 函数

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b  // 最后一行是返回值（无分号）
}

fn no_return() {
    println!("无返回值，返回单元类型 ()");
}

// 闭包
let square = |x: i32| x * x;
let capture = |x| x + a;  // 自动捕获环境变量
```

### 4.4 控制流

```rust
// if 表达式（有返回值！）
let num = if condition { 5 } else { 6 };

// loop（无限循环 + 返回值）
let result = loop {
    break 42;
};

// while
while count > 0 {
    count -= 1;
}

// for（最常用）
for item in collection.iter() { /* ... */ }
for i in 0..10 { /* ... */ }        // 0 到 9
for i in 0..=10 { /* ... */ }       // 0 到 10
```

### 4.5 模式匹配（Rust 杀手锏）

```rust
match value {
    1 => println!("one"),
    2..=5 => println!("two to five"),
    n if n % 2 == 0 => println!("even: {}", n),
    _ => println!("other"),  // _ 通配
}

// if let 简化
if let Some(x) = option {
    println!("got: {}", x);
}

// while let
while let Some(item) = iterator.next() {
    // ...
}
```

---

## 5. 所有权系统（核心难点）

Rust 最独特的特性，理解它就理解了 Rust。

### 5.1 三条规则

1. Rust 中每个值都有一个 **所有者**（owner）
2. 同一时刻，值只能有 **一个** 所有者
3. 当所有者离开作用域，值被 **丢弃**（drop）

```rust
{
    let s1 = String::from("hello");
    let s2 = s1;         // 所有权转移（move）！s1 不再有效
    // println!("{}", s1); // 编译错误！s1 已被 move
    println!("{}", s2);   // OK
}  // s2 离开作用域，内存自动释放
```

### 5.2 克隆

```rust
let s1 = String::from("hello");
let s2 = s1.clone();    // 深拷贝，两个都有效
println!("{} {}", s1, s2); // OK
```

### 5.3 引用与借用

```rust
fn calculate_length(s: &String) -> usize {  // 借用（不获取所有权）
    s.len()
}

let s = String::from("hello");
let len = calculate_length(&s);  // 传引用
println!("{}: {}", s, len);      // s 仍然有效

// 可变借用
fn append_world(s: &mut String) {
    s.push_str(", world");
}

let mut s = String::from("hello");
append_world(&mut s);
```

**借用规则**（编译器强制）：
- 同一时刻，可以有 **任意多个** 不可变引用 `&T`
- 或者，**仅一个** 可变引用 `&mut T`
- 二者不能同时存在（防止数据竞争）

### 5.4 生命周期

```rust
// 显式生命周期标注
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}

// 大多数情况编译器可以自动推断（生命周期省略规则）
fn first_word(s: &str) -> &str {  // 编译器自动推断
    // ...
}
```

---

## 6. 枚举与 Option/Result

### 6.1 枚举

```rust
enum Shape {
    Circle(f64),
    Rectangle { width: f64, height: f64 },
    Triangle(f64, f64, f64),
}

fn area(shape: &Shape) -> f64 {
    match shape {
        Shape::Circle(r) => std::f64::consts::PI * r * r,
        Shape::Rectangle { width, height } => width * height,
        Shape::Triangle(a, b, c) => {
            let s = (a + b + c) / 2.0;
            (s * (s - a) * (s - b) * (s - c)).sqrt()
        }
    }
}
```

### 6.2 Option —— 没有 null

```rust
// Option<T> = Some(T) | None
fn find_user(id: u32) -> Option<String> {
    if id == 1 {
        Some("Alice".to_string())
    } else {
        None
    }
}

// 安全使用
match find_user(1) {
    Some(name) => println!("Found: {}", name),
    None => println!("Not found"),
}

// 常用方法
let name = find_user(1).unwrap_or("Unknown".to_string());
let name = find_user(1).unwrap();  // None 时 panic！慎用
if let Some(name) = find_user(1) { /* ... */ }
```

### 6.3 Result —— 没有 try/catch

```rust
// Result<T, E> = Ok(T) | Err(E)
use std::fs;

fn read_config() -> Result<String, std::io::Error> {
    fs::read_to_string("config.toml")
}

// 错误处理方式
// 1. match
match read_config() {
    Ok(content) => println!("{}", content),
    Err(e) => eprintln!("Error: {}", e),
}

// 2. ? 运算符（错误传播，最常用）
fn load_config() -> Result<String, std::io::Error> {
    let content = fs::read_to_string("config.toml")?;  // 错误时自动 return Err
    Ok(content)
}

// 3. unwrap / expect（原型开发快速报错）
let content = fs::read_to_string("config.toml").expect("Failed to read config");
```

---

## 7. 结构体与 Trait

### 7.1 结构体

```rust
#[derive(Debug, Clone)]  // 自动派生 trait
struct User {
    name: String,
    age: u32,
    active: bool,
}

impl User {
    // 关联函数（类似静态方法）
    fn new(name: String, age: u32) -> Self {
        Self { name, age, active: true }
    }

    // 方法（&self 借用）
    fn summary(&self) -> String {
        format!("{} ({}), active: {}", self.name, self.age, self.active)
    }

    // 可变方法
    fn birthday(&mut self) {
        self.age += 1;
    }
}

let mut user = User::new("Alice".to_string(), 30);
println!("{:?}", user);         // Debug 输出
println!("{}", user.summary()); // 方法调用
user.birthday();
```

### 7.2 元组结构体 & 单元结构体

```rust
struct Color(u8, u8, u8);
let red = Color(255, 0, 0);

struct Marker; // 无字段
```

### 7.3 Trait（类似接口）

```rust
trait Describe {
    fn describe(&self) -> String;

    // 默认实现
    fn brief(&self) -> String {
        self.describe().chars().take(50).collect()
    }
}

impl Describe for User {
    fn describe(&self) -> String {
        format!("User {} is {} years old", self.name, self.age)
    }
}

// Trait 作为参数
fn print_description(item: &impl Describe) {
    println!("{}", item.describe());
}

// 或用 Trait Bound（泛型约束）
fn print_description<T: Describe>(item: &T) {
    println!("{}", item.describe());
}
```

---

## 8. 泛型

```rust
// 泛型函数
fn largest<T: PartialOrd>(list: &[T]) -> &T {
    let mut largest = &list[0];
    for item in &list[1..] {
        if item > largest {
            largest = item;
        }
    }
    largest
}

// 泛型结构体
struct Point<T> {
    x: T,
    y: T,
}

// 多类型参数
struct Pair<T, U> {
    first: T,
    second: U,
}

// Trait Bound 多约束
fn some_func<T: Clone + Debug>(item: &T) { /* ... */ }

// where 子句（更清晰）
fn complex<T, U>(t: &T, u: &U) -> String
where
    T: Display + Clone,
    U: Display + Debug,
{
    format!("{} {:?}", t, u)
}
```

---

## 9. 集合类型

```rust
use std::collections::{HashMap, HashSet, VecDeque, BTreeMap};

// Vec（动态数组，最常用）
let mut v: Vec<i32> = Vec::new();
v.push(1);
v.push(2);
let v2 = vec![1, 2, 3, 4, 5]; // 宏初始化
let third = &v2[2];             // 索引访问
let third = v2.get(2);          // 安全访问 -> Option<&i32>

// HashMap
let mut scores: HashMap<String, i32> = HashMap::new();
scores.insert(String::from("Alice"), 100);
scores.entry(String::from("Bob")).or_insert(50);  // 不存在才插入
let score = scores.get("Alice");  // Option<&i32>

// HashSet
let mut set: HashSet<i32> = HashSet::new();
set.insert(42);
if set.contains(&42) { /* ... */ }

// 迭代器
let nums = vec![1, 2, 3, 4, 5];
let sum: i32 = nums.iter().sum();
let doubled: Vec<i32> = nums.iter().map(|x| x * 2).collect();
let evens: Vec<&i32> = nums.iter().filter(|x| *x % 2 == 0).collect();
```

---

## 10. 错误处理最佳实践

```rust
use std::error::Error;

// 自定义错误类型（使用 thiserror 库）
// Cargo.toml: thiserror = "2"
use thiserror::Error;

#[derive(Error, Debug)]
enum AppError {
    #[error("IO error: {0}")]
    Io(#[from] std::io::Error),   // 自动 From 转换

    #[error("Config error: {message}")]
    Config { message: String },

    #[error("Not found: {0}")]
    NotFound(String),
}

type Result<T> = std::result::Result<T, AppError>;

fn run() -> Result<()> {
    let content = std::fs::read_to_string("config.toml")?;  // ? 自动转换
    if content.is_empty() {
        return Err(AppError::Config { message: "empty config".into() });
    }
    Ok(())
}
```

---

## 11. 并发编程

### 11.1 线程

```rust
use std::thread;
use std::sync::{Arc, Mutex};

let counter = Arc::new(Mutex::new(0));
let mut handles = vec![];

for _ in 0..10 {
    let counter = Arc::clone(&counter);
    let handle = thread::spawn(move || {
        let mut num = counter.lock().unwrap();
        *num += 1;
    });
    handles.push(handle);
}

for handle in handles {
    handle.join().unwrap();
}
println!("Result: {}", *counter.lock().unwrap()); // 10
```

### 11.2 异步编程（Tokio）

```toml
# Cargo.toml
[dependencies]
tokio = { version = "1", features = ["full"] }
```

```rust
#[tokio::main]
async fn main() {
    let handle = tokio::spawn(async {
        42
    });
    let result = handle.await.unwrap();
    println!("Result: {}", result);

    // 并发执行
    let (r1, r2) = tokio::join!(fetch_data(), fetch_other());
}

async fn fetch_data() -> String {
    // 异步操作
    String::from("data")
}

async fn fetch_other() -> String {
    String::from("other")
}
```

### 11.3 通道（Channel）

```rust
use std::sync::mpsc;
use std::thread;

let (tx, rx) = mpsc::channel();

thread::spawn(move || {
    let msgs = vec![
        String::from("hello"),
        String::from("from"),
        String::from("thread"),
    ];
    for msg in msgs {
        tx.send(msg).unwrap();
    }
});

for received in rx {
    println!("Got: {}", received);
}
```

---

## 12. 常用工具库速查

| 需求 | 库 | 说明 |
|------|-----|------|
| 序列化 | `serde` + `serde_json` | JSON/YAML/TOML 等序列化 |
| HTTP 客户端 | `reqwest` | 异步 HTTP 请求 |
| Web 框架 | `axum` / `actix-web` | 异步 Web 服务 |
| 异步运行时 | `tokio` | 异步 IO 运行时 |
| 数据库 | `sqlx` / `sea-orm` | 异步数据库操作 |
| CLI 工具 | `clap` | 命令行参数解析 |
| 日志 | `tracing` | 结构化日志 |
| 错误处理 | `thiserror` + `anyhow` | 错误类型定义与传播 |
| 正则 | `regex` | 正则表达式 |
| 时间 | `chrono` / `time` | 日期时间处理 |
| 随机数 | `rand` | 随机数生成 |

---

## 13. 实战：完整 CLI 工具示例

```rust
// src/main.rs —— 一个简单的文件行数统计工具
use std::env;
use std::fs;
use std::process;

fn main() {
    let args: Vec<String> = env::args().collect();

    if args.len() < 2 {
        eprintln!("Usage: {} <file_path>", args[0]);
        process::exit(1);
    }

    let path = &args[1];

    match count_lines(path) {
        Ok(count) => println!("{} has {} lines", path, count),
        Err(e) => {
            eprintln!("Error reading {}: {}", path, e);
            process::exit(1);
        }
    }
}

fn count_lines(path: &str) -> Result<usize, std::io::Error> {
    let content = fs::read_to_string(path)?;
    Ok(content.lines().count())
}
```

---

## 14. 学习路线建议

```
第 1 周：基础语法 + 所有权 + 借用
    ↓
第 2 周：枚举/模式匹配 + 错误处理 + Trait
    ↓
第 3 周：泛型 + 生命周期 + 集合类型
    ↓
第 4 周：并发编程（线程 + async/await）
    ↓
第 5-8 周：实战项目
    ├── CLI 工具（clap + serde）
    ├── Web API（axum + sqlx）
    ├── 系统工具（文件处理/网络编程）
    └── WASM 项目（wasm-pack）
```

---

## 15. 推荐资源

| 资源 | 说明 |
|------|------|
| [The Rust Book](https://doc.rust-lang.org/book/) | 官方教程，必读 |
| [Rust by Example](https://doc.rust-lang.org/rust-by-example/) | 代码示例学习 |
| [Rustlings](https://github.com/rust-lang/rustlings) | 交互式练习题 |
| [Rust 标准库文档](https://doc.rust-lang.org/std/) | API 文档 |
| [crates.io](https://crates.io) | 包注册中心 |
| [Rust 设计模式](https://rust-unofficial.github.io/patterns/) | 惯用写法 |

---

> **速通心得**：Rust 的学习曲线陡峭在**所有权和生命周期**，一旦突破这两个概念，后续学习会顺畅很多。建议边学边写小项目，让编译器当你的老师——Rust 编译器的错误提示是最好的学习材料之一。
