---
title: How Linux Works(Chapter Two)--Directory Hierarchy Essentials
tags:
  - How Linux Works
  - Linux
  - Linux kernel
date: 2019-03-04
---

## Directory Hierarchy Essentials

> All the summaries are from the book named **[How Linux Works](https://www.amazon.com/How-Linux-Works-2nd-Superuser/dp/1593275676/ref=sr_1_1?keywords=how+linux+works&qid=1551169061&s=gateway&sr=8-1)**.

Let's check the directory hierarchy of linux first.

![Linux directory hierarchy](https://sherlockblaze.com/resources/img/linux/how-linux-works/linux-directory-hierarchy.png)

This picture offers a simplified overview of the hierarchy, showing some of the directories under `/`, `/usr`, and `/var`. Notice that the directory structure under `/usr` contains some of the same directory names as `/`.

Here are the most important subdirectories in `root`:

- `/bin` Contains ready-to-run programs(also known as an executables), including most of the basic Unix commnads such as `ls` and `cp`. Most of the programs in `/bin` are in binary format, having been created been by a C compiler, but some are shell scripts in modern systems.
- `/dev` Contains device files.
- `/etc` This core system configuration directory contains the user password, boot, device, networking, and other setup files. Many items in `/etc` are specific to the machine's hardware. For example, the `/etc/X11` directory contains graphics card and window system configurations.
- `/home` Holds personal directories for regular users. Most Unix installations conform to this standard.
- `/lib` An abbreviation for library, this directory holds library files containing code that that executables can use. There are ***two*** types of libraries: **static** and **shared**. **The `/lib` directory should contain only shared libraries**, but other lib directories, such as `/usr/lib`, contain both varieties as well as other auxiliary files.
- `/proc` Provides  system statistis through a browsable directory-and-file interface. Much of the `/proc` subdirectory structure on Linux is unique, but many other Unix variants have similar features. The `/proc` directory contains information about currently running processes as well as some kernel parameters.
- `/sys` This directory is similar to `/proc` in that it provides a device and system interface.
- `/sbin` The place for system executables. Programs in `/sbin` directories relate to system management, so regular users usually do not have `/sbin` components in their command paths. Many of the utilities found here will not work if you're not running them as root.

### Recommended Reading

[Filesystem Hierarchy Standard](http://www.pathname.com/fhs/)