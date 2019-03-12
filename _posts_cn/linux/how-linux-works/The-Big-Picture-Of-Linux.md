---
title: Linux是如何工作的--Linux系统总览
tags:
  - Linux是如何工作的
  - Linux
  - Linux 内核
date: 2019-03-12
---

## Linux 系统总览

> 本文所总结的内容都来自于： **[How Linux Works](https://www.amazon.com/How-Linux-Works-2nd-Superuser/dp/1593275676/ref=sr_1_1?keywords=how+linux+works&qid=1551169061&s=gateway&sr=8-1)**.

### 关于如何理解学习

学习理解一个操作系统如何工作的最好方式，莫过于***抽象***，通过抽象你可以在刚开始的时候去选择忽略掉一些细节。举个例子来讲，当你驾驶一辆车，一般情况下，你不需要去知道车是谁制造的，路是谁搭的。特别的是如果你是一个乘客，那你唯一需要知道的就是这是辆什么车以及怎么去使用它。但是，如果你是驾驶员，你就需要知道更多了，你需要知道如何去操控车，当车出现一些故障的时候，你需要知道如何去处理。

举个例子，我们假设车开起来很不爽，一直在颠簸。现在你需要将 “一辆车开在路上” 这个问题抽象成为：一辆车、一条路以及你驾驶技术。这样可以帮你分解问题：如果这个路有问题，你就不需要特别去考虑你的车或者你驾驶技术的问题。于是你就进一步去探索路的问题。

程序员在创造一个操作系统和在其环境下运行的应用程序时会利用抽象。计算机软件中的抽象细分有很多术语，包括子系统、模块、组件和包，在构建软件组件时，程序员通常不会考虑其他组件的内部结构，但需要了解他们可以使用的其他组件以及如何使用它们。

### Linux 系统中抽象层次和层级

Linux系统中主要有三个层。下面的图片展示了这三个层以及各个层对应的一些组件。**硬件**是最底下的一层，硬件包括了存储以及一个或多个CPU —— 用于从存储中读取数据并计算/写入输入。像一些磁盘，网络接口之类，也是硬件的一部分。

往上一个层级是Linux内核，是操作系统的核心。**内核常驻于内存中，它会告诉CPU去做什么。**它就像是硬件和各个运行程序之间的接口，用来管理硬件。


***进程*** —— 被内核管理的运行中的程序 —— 共同组成了操作系统的上层，被称为***用户空间***。（进程用一个更常见的术语描述是***用户进程***，不管用户是否可以直接和进程进行交互。比如说，所有的Web服务都是以用户进程的方式来运行。）

![General Linux System Organization](https://sherlockblaze.com/resources/img/linux/how-linux-works/general-linux-system-organization.png)

用户进程和内核进程最明显的不同就是：用户进程运行在**用户模式**，而内核进程运行在**内核模式**。在内核模式下运行的程序可以访问到处理器和主存。而只有内核能访问到的区域被称为**内核空间**

> 这是一个很有力但是危险的特权，因为内核进程可以很轻松让整个操作系统瘫痪。

**用户模式**，和内核模式比较着来说，它仅能访问部分内存，和进行被系统认为安全的CPU操作。***用户空间***用来表示在主存里可以被用户进程访问到的空间。如果一个进程出了问题甚至崩溃，它的危害会被限制在系统之外，内核会负责清理掉它。比如说，你的浏览器程序奔溃了，一般来说，是不会影响到你后台运行了几天的科学计算的。

### 硬件：理解主存

在所有的硬件设备里，***主存（内存）***可能是最重要的。在它最原始的表示下，主存只是存放了一堆0/1的区域，每个0/1我们称之为一个***bit（位）***。内存同时也是内核生存的地方，内核常驻于内存，它要负责告诉CPU去做些什么事情，需要去管理硬件。所有的从外设获取到的输入输出都需要经过内存，同样也是一堆 bits。而CPU则是一个内存操作师，它从内存中读取数据（指令等）并向内存中写入处理后的结果。

You'll often hear the term ***state*** in reference to memory, processes, the kernel, and other parts of a computer system. A state is a particular arrangement of bits. For example, if you have four bits in your memory, 0110, 0001, and 1011 represent three different states.

**Note**: Because it's common to refer to the state in abstract terms rather than to the actual bits, the term image refers to a particular physical arrangement of bits.

### 内核

Nearly everything that the kernel does revolves around main memory. One of the kernel's tasks is to **split memory into many subdivisions, and it must maintain certain state information about those subdivisions at all times.** ***Each process gets its own share of memory, and the kernel must ensure that each process keeps to it share.***

The kernel is in charge of managing tasks in **four** general system areas:

- **Processes**. The kernel is responsible for determining which processes are allowed to use the CPU.
- **Memory**. The kernel needs to keep track of all memory -- what is currently allocated to a particular process, what might be shared between processes, and what is free.
- **Device Drivers**. The kernel acts as an interface between hardware and processes. It's usually the kernel's job to operate the hardware.
- **System calls and support**. Processes normally use system calls to communicate with the kernel.

#### 进程管理

***Process Management*** describes the starting, pausing, resuming, and terminating of processes.

On any modern operating system, many processes run "simultaneously". For example, you might have a web browser and a spreadsheet open on a desktop computer at the same time. However, things are not as the appear: The processes behind these applications typically do not run at ***exactly*** the same time.

**Context Switch**: Consider a system with a one-core CPU, many processes may be able to use the CPU, but only one process may actually use the CPU at any given time. In practice, each process uses the CPU for a small fraction of a second, the pauses; the another process uses the CPU for another small fraction of a second; then another process takes a turn, and so on. The act of one process giving up control of the CPU to another process is called a ***context switch***.

**Time Slice**: Each piece of time—called a ***time slice*** — gives a process enough time for significant computation (and indeed, a process often finishes its current task during a single slice). However, because the slices are so small, humans can’t perceive them, and the system appears to be running multiple processes at the same time (a capability known as multitasking).

**How a context switch works??**

> The kernel is responsible for context switching. And here's what happens:

1. The CPU interrupts the current process based on an internal timer, switches into kernel mode, and hands control back to the kernel.
2. The kernel records the current state of the CPU and memory, which will be essential to resuming the process that was just interrupted.
3. The kernel performs any tasks that might have come up during the preceding time slice.(such as collecting data from input and output, or I/O, operations)
4. The kernel is now ready to let another process run. The kernel analyzes the list of processes that are ready to run and choose one.
5. The kernel prepares the memory for this new process, and then prepares the CPU.
6. The kernel tells the CPU how long the time slice for the new process will last.
7. The kernel switches the CPU into user mode and hands control of the CPU to the process.

**The context switch answers the important question of when the kernel runs. The answer is that it runs between process time slices during a context swtich.**

**Attention:**

In the case of a multi-CPU system, things become slightly more complicated because the kernel doesn't need to relinquish control of its current CPU in order to allow a process to run on a different CPU. However, to maximize the usage of all available CPUs, the kernel typically does so anyway(and may use certain tricks to grab a little more CPU time for itself).

#### 内存管理

Because the kernel must manage memory during a context switch, it has a complex job of memory management. The kernel's job is complicated because the following conditions must hold:

- The kernel must have its own private area in memory that user processes can't access.
- Each user process needs its own section of memory
- One user process may not access the private memory of another process.
- User processes can share memory.
- Some memory in user processes can be read-only.
- The system can use more memory than is physically present by using disk space as auxiliary.

It's difficult, but fortunately for the kernel, there is help. Modern CPUs include a ***memory management unit(MMU)*** that enables a memory access scheme called ***virtual memory***. When using virtual memory, **a process does not directly access the memory by its physical location in the hardware.** Instead, the kernel sets up each process to act as if it had an entire machine to itself.
When the process accesses some of its memory, the MMU intercepts the access and uses a memory address map to translate the memory location from the process into an actual physical memory location on the machine.
**The kernel must still initialize and continuously maintain and alter this memory address map. For example, during a context switch, the kernel has to change the map from the outgoing process to the incoming process.**

**Note**: The implementation of a memory address map is called a page table.

#### 设备驱动及管理

The kernel's role with devices is pretty simple. A device is typically accessible only in kernel mode because improper access could crash the machine. Another problem is that different devices rarely have the same programming interface, even if the devices do the same thing, such as two different network cards. Therefore, device drivers have traditionally been part of the kernel, and they strive to present a uniform interface to user processes in order to simplify the software developer's job.

#### 系统调用和支持

There are several other kinds of kernel features available to user processes. For example, ***system calls(or syscalls)*** perform specific tasks that a user process alone cannot do well or at all. For example, the acts of opening, reading, and writing files all involve system calls.

Two system calls, `fork()` and `exec()`, are important to understanding how processes start up:

- **fork()** When a process calls `fork()`, the kernel creates a nearly identical copy of the process.
- **exec()** When a process calls `exec(program)`, the kernel starts program, replacing the current process.

**Other than init, all user processes on a Linux system start as a result of `fork()`**, and most of the time, you also run `exec()` to start a new program instead of running a copy of an existing process. A very simple example is any program that you run at the command line, such as the `ls` command to show the contents of a directory. When you enter `ls` into a terminal window, the shell that's running inside the terminal window calls `fork()` to create a copy of the shell, and then the new copy of the shell calls `exec(ls)` to run `ls`. The process shows as follow:

![start a new process](https://sherlockblaze.com/resources/img/linux/how-linux-works/starting-a-new-process.png)

The kernel also supports user processes with features other than traditional system calls, the most common of which are ***pseudodevices***. Pseudo-devices look like devices to user processes, but they're implemented purely in software. As such, they don't technically need to be in the kernel, but they are usually there for practical reasons. For example, the kernel random number generator device(`/dev/random`) would be difficult to implement securely with a user process.

Technically, a user process that accesses a pseudodevice still has to use a system call to open the device, so processes can't entirely avoid system calls.

### 用户空间

The main memory that the kernel allocates user processes is called ***user space***. Because a process is simply a state(or image) in memory, user space also refers to the memory for the entire collection of running processes.

Most of the real action on a Linux system happens in user space. Although all processes are essentially equal from the kernel's point of view, they perform different tasks for user. There is a rudimentary service level (or layer) structure to the kinds of system components that user processes represent. The following picture shows how an example set of components fit together and interact on a Linux system. Basic services are at the bottom level(closest to the kernel), utility services are in the middle, and applications that users touch are at the top. What in the picture is a greatly simplified diagram because only six components are shown, but you can see that the components at the top are closest to the user; the components in the middle level has a mail server that the web browser uses; and there are several smaller components at the bottom.

![Process types and interactions](https://sherlockblaze.com/resources/img/linux/how-linux-works/process-types-and-interactions.png)

The bottom level tends to consist of small components that perform single, uncomplicated tasks. The middle level has larger components such as mail, print, and database services. Finally, components at the top level perform complicated tasks that the user often controls directly. Components also use other components. Generally, if one component wants to use another, the second component is either at the same service level or below.

However, the above picture is only an approximation of the arrangement of user space. In reality, there are no rules in user space. For example, most applications and services write diagnostic messages known as logs. Most programs use the standard syslog service to write log messages, but some prefer to do all of the logging themselves.

In addition, it’s difficult to categorize some user-space components. Server components such as web and database servers can be considered very high-level applications because their tasks are often complicated, so you might place these at the top level in the above picture. However, user applications may depend on these servers to perform tasks that they’d rather not do themselves, so you could also make a case for placing them at the middle level.

### 用户

The Linux kernel supports the traditional concept of a Unix user. A ***user*** is an entity that can run processes and own files. A user is associated with a ***username***. 

**The kernel does not manage the usernames, instead, it identifies users by simple numeric identifiers called `userids`**

**Users exist primarily to support permissions and boundaries.** Every user-space process has a user ***owner***, and processes are said to run as the owner. A user may terminate or modify the behavior of its own processes(within certain limits), but it cannot interfere with other users' processes. In addition, users may own files and choose whether they share them with other users.

The most important user to know about is ***root***. The root user is an exception to the preceding rules because root may terminate and alter another user's processes and read any file on the local system. FOr this reason, root is known as the **superuser**. A person who can operate as root is said to have root access and is an administrator on a traditional Unix system.

**Groups** are sets of users. The primary purpose of groups is to allow a user to share file access to other users in a group.

> Operating as root can be dangerous. It can be difficult to identify and correct mistakes because the system will let you do anything, even if what you're doing is harmful to the system. For this reason, system designers constantly try to make root access as unnecessary as possible, for example, by not requiring root access to switch between wireless networks on a notebook. In addition, as powerful as the root user is, it still runs in the operating system's user mode, not kernel mode.

### 结论

User processes make up the environment that you directly interact with, the kernel manages processes and hardware. Both the kernel and processes reside in memory.

### 推荐阅读

- 操作系统概念, 9th edition, by Abraham Silberschatz, Peter B. Galvin, and Greg Gagne (Wiley, 2012)
- 现代操作系统, 4th edition, by Andrew S. Tanenbaum and Herbert Bos (Prentice Hall, 2014).