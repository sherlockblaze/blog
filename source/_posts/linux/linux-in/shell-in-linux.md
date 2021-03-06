---
title: Linux 常用命令及技巧
tags:
  - Linux
date: 2020-04-19
---

## 什么是 shell

Shell 在 Linux 中的作用就是围绕在 Linux 内核之外的一个“壳” 程序用户在操作系统上完成的所有任务都是通过 Shell 与 Linux 的交互来实现的。

**shell 既是一种命令解释程序，又是一种功能强大的解释型程序设计语言。**

作为**命令解释程序**， shell 解释用户输入的命令，然后提交到内核处理，最后把结果返回给用户。

为了加速命令的运行，同时更有效地定制 shell 程序，shell 中定义了一些内置命令，**一般把 shell 自身解释执行的命令称为内置命令**。

> 当用户登录系统后，shell 以及内置命令就被系统载入到内存，并且一直运行，直到用户退出系统为止。"#" 表示登录的用户是超级用户， "$" 表示登录到系统的是普通用户。

> **shell 执行命令解释的具体过程为:** 用户在命令行输入命令并提交后，shell 程序**首先检测它是否为内置命令**，如果是，就通过 shell 内部的解释器将命令解释为系统调用，然后提交给内核执行；如果不是 shell 内置的命令，那么 shell 会按照用户给出的路径或者根据系统环境变量的配置信息在硬盘寻找对应的命令，然后将其调入内存中，最后再将其解释为系统调用，提交给内核执行。

而作为**解释型程序设计语言**，它定义了各种选项和变量，几乎支持高级程序语言的所有程序结构，如变量、函数、表达式和循环等。利用 shell 可以编写 shell 脚本。

## Shell 命令的语法分析

shell 语法分析是指 shell 对**命令的扫描处理过程，**也就是，把命令或者用户输入的内容分解成要处理的各个部分的操作。在 Linux 系统下，shell 语法分析包含很多内容，如重定向、文件扩展名和管道等。

### shell 的命令格式

**shell 遵循一定的语法格式将用户输入的命令进行分析解释并传递给系统内核。**

shell 命令的一般格式为：

```sh
command [options] [arguments]
```

- command: 表示命令的名称
- options: 表示命令的选项
- arguments: 表示命令的参数

### shell 的通配符

### shell 的重定向

### shell 的管道

### shell 的自动补全命令行