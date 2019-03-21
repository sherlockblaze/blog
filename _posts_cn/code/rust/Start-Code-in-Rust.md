---
title: Rust 编程初体验
tags:
  - Rust
  - 编程语言
  - 编程
date: 2019-03-21
---

## Rust 编程初体验

### Rust 的优势

1. 性能超好（可以用于编写关键组件，能够提供超高性能）
2. 存储效率高
3. 没有Runtime和GC
4. 可以运行在嵌入式设备
5. 可以很方便的和其他语言携作

### 使用范围

技能范围很广。可以用于：

1. 命令行程序
2. Web组件
3. 网络程序
4. 嵌入式程序(运行在性能比较相对较低的硬件)

### Rust的自身定位

从 《The Rust Programming Language》这本书的前言来看，要表述的大概是以下的意思：

> Rust 这门语言就很屌，就不管你以前用什么语言，不管是Java、Python、Go、还是C++，用我都可以写出来比用这些语言写的代码高到不知道到哪里去的代码来。底层操作，高并发，包管理应有尽有。而且呢，我不仅用来写底层应用，还用用来写一些其他的程序，比如上面提到的命令行程序、Web组件等。而且啊，你不用为了写底层应用学一套，写Web组件学一套，你用写底层应用掌握的技能，同样可以用在Web组件上。
总之，Rust 就很屌。

### 安装 Rust

[安装指南](https://www.rust-lang.org/learn/get-started)

如果你使用的是类Linux系统，或者Mac，使用以下的命令进行安装。

```sh
$ curl https://sh.rustup.rs -sSf | sh
```

### 基础语法

#### Hello World

接下来是经典的编程语言开头，`Hello World`。

关于IDE，我个人不是很喜欢用IDE，使用的Sublime。如果你喜欢，个人比较推荐宇宙IDE -- `VSCode`。

程序有点出奇的简单，不需要任何引入，不用包名之类，编写以下代码即可：

##### 代码编写

> main.rs

```rust
fn main() {
    println!("Hello World!")
}
```

> 吐槽一个点，println后面要加个!号，暂时不知道为什么要这样。

##### 编译程序

使用以下命令行进行编译：

```sh
$ rustc main.rs
```

执行完以上命令后会在命令执行的目录下生成一个名称为 `main` 的文件。

##### 运行程序

执行以下命令运行程序：

```sh
$ ./main
```

##### 总结

经过 `Hello World` 程序的编写运行后，我们可以得出以下几个结论：

1. Rust 是门编译类型的语言
2. 程序的编译和运行是分阶段的
3. 程序的语法直观上感受比较简单（需要更多探索）
4. 代码文件后缀名为 `.rs`
5. 编译用的命令为 `rustc`，类似 `javac`

#### Cargo

Cargo 是用来管理和编译 Rust 项目的。

它会帮你构建代码，下载你代码使用到的依赖库并且构建这些库。

如果你使用上面介绍的方法来[安装](安装-Rust) Rust 的话，那么 cargo 会被一起安装，如果你使用的其他方法，就自己研究下怎么安装吧。安装完成后，使用以下命令来查看所安装 cargo 版本：

```sh
$ cargo --version
```

##### 使用 Cargo 创建 Rust 程序

使用以下命令格式创建项目：

```sh
$ cargo new project-name
```

其中，将 project-name 替换成你期望的名称。比如，hello-cargo。

使用以下命令可以看到 Cargo 为我们创建的文件夹内容：

```sh
$ ls -lR hello-cargo

hello_cargo/:
total 8
-rw-r--r-- 1 xxx xxx  137 Mar 21 13:35 Cargo.toml
drwxr-xr-x 2 xxx xxx 4096 Mar 21 13:35 src

hello_cargo/src:
total 4
-rw-r--r-- 1 xxx xxx 45 Mar 21 13:35 main.rs
```

***关于 `Cargo.toml` 文件***

使用命令 `less Cargo.toml` 简单展示一下文件内容：

```
[package]
name = "hello_cargo"
version = "0.1.0"
authors = ["DaShuaiBi <dashuaibi@xxx.com>"]
edition = "2018"

[dependencies]
```

我们可以观察到两部分内容，一部分是 `package` 一部分是 `dependencies`。顾名思义，其中，`package`标签下的内容，展示的是你这个项目的一些相关信息。而`dependencies` 标签下的条目则展示了该项目所以依赖的程序库。

##### 使用 Cargo 构建程序

在 `hello-cargo` 目录下，执行 `cargo build` 命令，即可完成编译。

> 如果你不是在hello-cargo目录下或者hello-cargo的子目录下执行上述操作，会收到以下报错：

```
error: could not find `Cargo.toml` in `/home/xxx/Documents/rust-stu` or any parent directory
```

> 由此，我们也可以推断出 Cargo 工作的大致条件。

构建完成后，再次执行 `ls -lR hello-cargo` 命令，得到以下输出：

```
hello_cargo/:
total 16
-rw-r--r-- 1 xxx xxx   52 Mar 21 13:43 Cargo.lock
-rw-r--r-- 1 xxx xxx  137 Mar 21 13:35 Cargo.toml
drwxr-xr-x 2 xxx xxx 4096 Mar 21 13:35 src
drwxr-xr-x 3 xxx xxx 4096 Mar 21 13:43 target

hello_cargo/src:
total 4
-rw-r--r-- 1 xxx xxx 45 Mar 21 13:35 main.rs

hello_cargo/target:
total 4
drwxr-xr-x 8 xxx xxx 4096 Mar 21 13:43 debug

hello_cargo/target/debug:
total 2392
drwxr-xr-x 2 xxx xxx    4096 Mar 21 13:43 build
drwxr-xr-x 2 xxx xxx    4096 Mar 21 13:43 deps
drwxr-xr-x 2 xxx xxx    4096 Mar 21 13:43 examples
-rwxr-xr-x 2 xxx xxx 2423456 Mar 21 13:43 hello_cargo
-rw-r--r-- 1 xxx xxx     124 Mar 21 13:43 hello_cargo.d
drwxr-xr-x 3 xxx xxx    4096 Mar 21 13:43 incremental
drwxr-xr-x 2 xxx xxx    4096 Mar 21 13:43 native

hello_cargo/target/debug/build:
total 0

hello_cargo/target/debug/deps:
total 2372
-rwxr-xr-x 2 xxx xxx 2423456 Mar 21 13:43 hello_cargo-bf3cd57173b95a7e
-rw-r--r-- 1 xxx xxx     223 Mar 21 13:43 hello_cargo-bf3cd57173b95a7e.d

hello_cargo/target/debug/examples:
total 0

hello_cargo/target/debug/incremental:
total 4
drwxr-xr-x 3 xxx xxx 4096 Mar 21 13:43 hello_cargo-3rt3pzkq7vr58

hello_cargo/target/debug/incremental/hello_cargo-3rt3pzkq7vr58:
total 4
drwxr-xr-x 2 xxx xxx 4096 Mar 21 13:43 s-fajlw6y2to-svrb3g-3jvcw2pxdz57w
-rwx------ 1 xxx xxx    0 Mar 21 13:43 s-fajlw6y2to-svrb3g.lock

hello_cargo/target/debug/incremental/hello_cargo-3rt3pzkq7vr58/s-fajlw6y2to-svrb3g-3jvcw2pxdz57w:
total 308
-rw-r--r-- 1 xxx xxx   4768 Mar 21 13:43 3nz1nw4s9w4hsg6z.o
-rw-r--r-- 1 xxx xxx   9096 Mar 21 13:43 4y40mf08ap6oto5i.o
-rw-r--r-- 1 xxx xxx   6192 Mar 21 13:43 4zl24jszwlof50s5.o
-rw-r--r-- 1 xxx xxx   3968 Mar 21 13:43 504r8xv4hlp35in7.o
-rw-r--r-- 1 xxx xxx 219157 Mar 21 13:43 dep-graph.bin
-rw-r--r-- 1 xxx xxx  42201 Mar 21 13:43 query-cache.bin
-rw-r--r-- 1 xxx xxx   3504 Mar 21 13:43 vurxm0o6n9ssdfg.o
-rw-r--r-- 1 xxx xxx    357 Mar 21 13:43 work-products.bin
-rw-r--r-- 1 xxx xxx   5712 Mar 21 13:43 zmt7nwlmqdvxye9.o

hello_cargo/target/debug/native:
total 0
```

观察一下，可以自行估计 Cargo 为我们做了哪些工作。

##### 使用 Cargo 运行程序

在完成构建操作后，我们便可以在 `hello-cargo` 目录或其某个子目录下，执行以下命令运行程序：

```sh
$ cargo run

    Finished dev [unoptimized + debuginfo] target(s) in 0.01s
     Running `target/debug/hello_cargo`
Hello, world!
```

同样，`cargo run` 命令的工作，也需要 `Cargo.toml` 文件的支持。

### 学习资源

[Youtube Channel](https://www.youtube.com/channel/UCaYhcUwRBNscFNUKTjgPFiA)
[Official Document](https://www.rust-lang.org/learn)
[《The Rust Programming Language》](https://doc.rust-lang.org/book/) (免费、英文原版)

推荐网站 [safaribooksonline.com](https://learning.oreilly.com) (需要花钱订阅)

> 网站上提供了丰富的图书资源，大多为 Oreilly 出版的图书内容，同时，也有一些比较经典的书，比如 《Unix环境下高级编程》 等，费用比较昂贵，一个月几百（几十美金），看书效率不是很高的同学，建议跟紧我的博客，或者单独购买图书。
