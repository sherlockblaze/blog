---
title: The Big Picture of Linux
tags:
  - How Linux Works
  - Linux
  - Linux kernel
date: 2019-02-26
---

## The Big Picture of Linux

> All the summaries are from the book named **[How LInux Works](https://www.amazon.com/How-Linux-Works-2nd-Superuser/dp/1593275676/ref=sr_1_1?keywords=how+linux+works&qid=1551169061&s=gateway&sr=8-1)**.

### About Understanding Something

The most effective way to understand how an operating system works is through ***abstraction*** ---- a fancy way of saying that you can ignore most of the details. For example, when you ride in a car , you normally don't need to think about details such as the mounting bolts that hold the motor inside the car or the people who build and maintain the road upon which the car drives. If you're a passenger in a car, all you really need to know is what the car does and a few basics about how to use it.

**But** if you're driving a car, you need to know more. You need to learn how to operate the controls and what to do when something goes wrong.

For example, let's say that the car ride is rough. Now **you can break up the abstraction of "a car that rolls on a road" into tree parts: a car, a road, and the way that you're driving**. This helps isolate the problem: If the road is bumpy, you don't blame the car or the way that you're driving it. Instead, you may want to find out why the road has deteriorated or, if the road is new, why the construction workers did a lousy job.

Software developers use abstraction as a tool when building an operating system and its applications. There are many terms for an abstracted subdivision in computer software, including subsystem, module, and package—but we’ll use the term component in this chapter because it’s simple. When building a software component, developers typically don’t think much about the internal structure of other components, but they do care about what other components they can use and how to use them.

### Levels and Layers of Abstraction in a Linux System

A Linux system has three main levels. The following pictures shows these levels and some of the components inside each level. The ***hardware*** is at the base. Hardware includes the memory as well as one or more CPUs to perform computation and to read from and write to memory. Devices such as disks and network interfaces are also part of the hardware.

The next level up is the kernel, which is the core of the operatin system. **The kernel is software residing in memory that tells the CPU what to do.** The kernel manages the hardware and **acts primarily as an interface** between the hardware and any running program.

***Processes*** -- the running programs that the kernel manages ---- collectively make up the system's upper level, called ***user space***. (A more specific term for process is ***user process***, regardless of whether of a user directly interacts with the process. For example, all web servers run as user processes.)

![General Linux System Organization](https://sherlockblaze.com/resources/img/linux/how-linux-works/general-linux-system-organization.png)

The critical difference between the ways that the kernel and user processes run: The kernel runs in ***kernel mode***, and the user processes run in ***user mode***. Code running in kernel mode has unrestricted access to the processor and main memory. **The area that only the kernel can access is called ___kernel space___.**

> This is a powerful but dangerous privilege that allows a kernel process to easily crash the entire system.

**User mode**, in comparison, restricts access to a subset of memory and safe CPU operations. ***User space*** refers to the parts of main memory that the user processes can access. If a process makes a mistake and crashes, the consequences are limited and can be cleaned up by the kernel. This means that if your web browser crashes, it probably won't take down the scientific computation that you've been running in the background for days.

### Hardware: Understanding Main Memory

Of all of the hardware on a computer system, ***main memory*** is perhaps the most important. In its most raw form, main memory is just a big storage area for a bunch of 0s and 1s. Each 0 or 1 is called a ***bit***. This is where the running kernel and processes reside ---- they're just big collections of bits. All input and output from peripheral devices flows through main memory, also as a bunch of bits. A CPU is just an operator on memory; it reads its instructions and data fromn the memory and writes data back out to the memory.

