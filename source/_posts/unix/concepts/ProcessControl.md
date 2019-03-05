---
title: Coding Under Unix -- Process Control
tags:
  - Unix
  - Processes
  - Coding Under Unix
date: 2019-03-05
---

## Process Control

> All the summaries are from the book named **[Advanced Programming in the Unix](https://www.amazon.com/Programming-Environment-Addison-Wesley-Professional-Computing/dp/0201563177/ref=sr_1_fkmrnull_1?crid=2YVJXTV3JD1HC&keywords=advance+programming+in+unix&qid=1551765355&s=gateway&sprefix=advance+unix%2Caps%2C467&sr=8-1-fkmrnull)**.

### Process Identifiers

**Every process has a unique process ID, a non-negative integer.**

The **process ID** is the only well-known identifier of a process that is always unique, it's often used as a piece of other identifiers, to guarantee uniqueness.

Although unique, **process IDs are reused**. As processes terminate, their IDs become candidates for reuse. **Most UNIX systems implement algorithms to dalay reuse**, however, so that newly created processes are assigned IDs different from those used by processes that terminated recently. ***This prevents a new process from being mistaken for the previous process to have used the same ID.***

**Process ID 0** is usually the scheduler process and is often known as the swapper. It is a part of the kernel and is known as a system process.
**Process ID 1** is usually the `init` proces and is invoked by the kernel at the end of bootstrap procedure. **This process is responsible for bringing up a UNIX system after the kernel has been bootstrapped.** `init` usually reads the system-dependent initialization files -- the `/etc/rc*` files or `/etc/inittab` and the files in `/etc/init.d` -- and brings the system to a certain state, such as multiuser. The `init` process never dies. **It is a normal user process**, but it does run with superuser privileges.

In Mac OS X, the `init` process was replaced with the `launchd` process, which performs the same set of tasks as `init`, but has expanded functionality.

In addition to process ID, there are other identifiers for every process. These following functions return these identifiers.

```c
#include <unistd.h>
// Returns: process ID of calling process
pid_t getpid(void);

// Returns: Parent process ID of calling process
pid_t getppid(void);

// Returns: real user ID of calling process
uid_t getuid(void);

// Returns: effective user ID of calling process
uid_t geteuid(void);

// Returns: real group ID of calling process
gid_t getgid(void);

// Returns: effective group ID of calling process
gid_t getegid(void);
```