---
title: How Linux Works(Chapter Three)--Devices
tags:
  - How Linux Works
  - Linux
  - Linux kernel
date: 2019-03-11
---

## Devices

> All the summaries are from the book named **[How Linux Works](https://www.amazon.com/How-Linux-Works-2nd-Superuser/dp/1593275676/ref=sr_1_1?keywords=how+linux+works&qid=1551169061&s=gateway&sr=8-1)**.

Throughout the history of Linux, there have been many changes to how the kernel presents  devices to the user. We'll begin by looking at the traditional system of device files to see how the kernel provides device configuration information through `sysfs`.

**Goal:** 

1. Be able to extract information about the devices on a system in order to understand a few rudimentary operations.
2. Understand how the kernel interacts with user space when presented with new devices. The `udev` system enables user-space programs to automatically configure and use new devices.

### Device Files

**It's easy to manipulate most devices on a Unix system because the kernel presents many of the device I/O interfaces to user processes as files.**

These device files are sometimes called ***device nodes.***

There's a limit to what you can do with a file interface, so not all devices or device capabilities are accessible with standard file I/O.

Using command `ls -l` and get result like this:

![ls -l](https://sherlockblaze.com/resources/img/linux/how-linux-works/ls-l-command.png)

Note the first character of each line(the first character of the file's mode), if this character is `b`, `c`, `p` or `s`, the file is a device. These letters stand for `block`, `pipe`, and `socket`, respectively.

#### Block device

- Program access data from a ***block device*** in fixed chunks, The `sda` in the preceding example is a ***disk device***, a type of block device. Disk can be easily split up into blocks of data. Because a block device's total size is fixed and easy to index, processes have random access to any block in the device with the help of the kernel.

#### Character device

- Character devices work with data streams. You can only read characters from or write characters to character devices, as previously demonstrated with `/dev/null`. Character devices don't have a size; **when you read from or write to one, the kernel usually performs a read or write operation on the device.** Printers directly attached to your computer are represented by character devices. **It's important to note that during character device interaction, the kernel cannot back up and reexamine the data stream after it has passed data to a device or process.**

#### Pipe device

- Named pipes are like character devices, with another process at the other end of the I/O stream instead of a kernel driver.

#### Socket device

- **Sockets** are special-purpose interfaces that are **frequently used for interprocess communication.** They're often found outside of the `/dev` directory. Socket files represent Unix domain sockets.

> Note all devices have device files because the block and character device I/O interfaces are not appropriate in all cases. For example, network interfaces don't have device files. It is theoretically possible to interact with a network interface using a single character device, but because it would be exceptionally difficult, the kernel uses other I/O interfaces.

### The `sysfs` Device Path

The traditional Unix `/dev` directory is a convenient way for user processes to reference and interface with devices supported by the kernel, but it's also a very simplistic scheme. The name of device in `/dev` tells you a little about device, but not a lot. **Another problem is that the kernel assigns devices in the order in which they are found, so a device may have a different name between reboots.**

To provide a uniform view for attached devices based on their actual hardware attributes, the Linux kernel offers the `sysfs` interface through a system of files and directories. The base path for devices is `/sys/devices`.

For example, the SATA hard disk at `/dev/sda` might have the following path in `sysfs`:

![](https://sherlockblaze.com/resources/img/linux/how-linux-works/device-in-sysfs.png)

The `/dev` file is there so that user processes can use the device, whereas the `/sys/devices` path is used to view information and manage the device.

There are a few shortcuts in the `/sys` directory. For example, `/sys/block` should contain all of the block devices available on a system. However, those are just symbolic links; run `ls -l /sys/block` to reveal the true sysfs paths.

It can be difficult to find the sysfs location of a device in `/dev`. Use the `udevadm` command to show the path and other attributes:

```sh
$ udevadm info --query=all --name=/dev/sda
```