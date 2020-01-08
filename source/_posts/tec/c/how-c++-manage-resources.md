---
title: C++ 如何管理资源
tags:
  - c/c++
  - tec
date: 2019-12-02
---

```sh
$ g++ -std=c++17 -W -Wall -Wfatal-errors file-name
$ clang -std=c++17 -W -Wall -Wfatal-errors file-name
```

## C++ 意义

> C++ 是一门多范式的通用编程语言

C++ 支持面向过程编程，也支持面向对象编程，也支持泛型编程，新版本还可以说支持了函数式编程。

典型情况是，需要性能的组件用 C++ 来写，整个应用程序融合多种不同的语言。

## C++ 竞争

- 专注性能和最小内存占用，C
- 专注抽象表达和可读性，Python 之类的脚本语言
- 图形界面，C# 和 JavaScript 占领了很大一部分市场
- 游戏领域，有了 C++ 写的游戏引擎，游戏用 C# 写也没有问题，Unity 游戏引擎首选开发语言是 C#
- Go 和 Rust

## C++ 如何管理资源

### 堆

英文 heap，在内存管理语境下，指**动态分配内存的区域**。这里的内存被分配之后需要**手动释放**，否则就会造成内存泄露。

C++ 标准有一个相关概念是自由存储区，英文是 **free store**，***特指使用 new 和 delete 来分配和释放内存的区域。***一般而言是堆的一个子集：

- **new 和 delete 操作的区域是 free store**
- **malloc 和 free 操作的区域是 heap**

### 栈

英文 stack，在内存管理的语境下，指的是函数调用过程中产生的本地变量和调用数据的区域。满足“后进先出”(last-in-first-out  LIFO)

### RAII

完整的英文是 Resource Acquisition Is Initialization，是 C++ 所特有的资源管理方式。

RAII 依托栈和析构函数，来对所有的资源——包括堆内存在内——进行管理。对 RAII 的使用，使得 C++ 不需要类似于 Java 那样的垃圾收集方法，也能有效地对内存进行管理。