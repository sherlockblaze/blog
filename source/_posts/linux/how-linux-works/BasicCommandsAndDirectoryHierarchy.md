---
title: How Linux Works(Chapter Two)--Basic Commands And Directory Hierarchy(Part One)
tags:
  - How Linux Works
  - Linux
  - Linux kernel
date: 2019-02-27
---

## Basic Commands And Directory Hierarchy(Part One)

> All the summaries are from the book named **[How Linux Works](https://www.amazon.com/How-Linux-Works-2nd-Superuser/dp/1593275676/ref=sr_1_1?keywords=how+linux+works&qid=1551169061&s=gateway&sr=8-1)**.

### The Bourne Shell

The shell is one of the most important parts of a Unix system. A `shell` is a program that runs commands, like the ones that users enter. The shell also serves as a small programming environment. **Unix programmers often break common tasks into little componenets and use the shell to manage tasks and piece things together.**

One of the best things about the shell is that if you make a mistake, you can easily see what you typed to find out what weng wrong, and then try again.

> Linux uses an enhanced version of the Bourne shell called `bash` or the "Bourne-again" shell. The `bash` shell is the default shell on most Linux distributions, and `/bin/sh` is normally a link to `bash` on a Linux system. You should use `bash` shell when running the examples I copy from the [book](https://www.amazon.com/How-Linux-Works-2nd-Superuser/dp/1593275676/ref=sr_1_1?keywords=how+linux+works&qid=1551169061&s=gateway&sr=8-1).

### Using the Shell

When you install Linux, you should create at least one regular user in addition to the root user; this will be your personal account.

### Recommended Reading

- The Linux Command Line (No Starch Press, 2012)
- UNIX for the Impatient (Addison-Wesley Professional, 1995)
- Learning the UNIX Operating System, 5th edition (O’Reilly, 2001).