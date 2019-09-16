---
title: Coding Under Unix -- Process Control
tags:
  - Unix
  - Processes
  - Coding Under Unix
date: 2019-03-05
---

> All the summaries are from the book named **[Advanced Programming in the Unix](https://www.amazon.com/Programming-Environment-Addison-Wesley-Professional-Computing/dp/0201563177/ref=sr_1_fkmrnull_1?crid=2YVJXTV3JD1HC&keywords=advance+programming+in+unix&qid=1551765355&s=gateway&sprefix=advance+unix%2Caps%2C467&sr=8-1-fkmrnull)**.

## Process Identifiers

**Every process has a unique process ID, a non-negative integer.**

The **process ID** is the only well-known identifier of a process that is always unique, it's often used as a piece of other identifiers, to guarantee uniqueness.

Although unique, **process IDs are reused**. As processes terminate, their IDs become candidates for reuse. **Most UNIX systems implement algorithms to delay reuse**, however, so that newly created processes are assigned IDs different from those used by processes that terminated recently. ***This prevents a new process from being mistaken for the previous process to have used the same ID.***

**Process ID 0** is usually the scheduler process and is often known as the swapper. It is a part of the kernel and is known as a system process.
**Process ID 1** is usually the `init` process and is invoked by the kernel at the end of bootstrap procedure. **This process is responsible for bringing up a UNIX system after the kernel has been bootstrapped.** `init` usually reads the system-dependent initialization files -- the `/etc/rc*` files or `/etc/inittab` and the files in `/etc/init.d` -- and brings the system to a certain state, such as multiuser. The `init` process never dies. **It is a normal user process**, but it does run with superuser privileges.

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

## Fork Function

An existing process can create a new one by calling the `fork` function. The new process created by `fork` is called the ***child*** process.

```c
#include <unistd.h>

// Returns: 0 in child, process ID of child in parent -1 on error
pid_t fork(void);
```

**This function is called once but returns twice:**

- **The return value in the child is 0.** The reason `fork` returns 0 to the child is that a process can have only a single parent, and the child can always call `getppid` to obtain the process ID of its parent.
- **The return value in the parent is the process ID of the new child.** The reason the child's process ID is returned to the parent is that a process can have more than one child, and there is no function that allows a process to obtain the process IDs of its children.

**Both the child and the parent continue executing with the instruction that follows the call to `fork`.**

The child is a copy of the parent. For example, the child gets a copy of the parent's data space, heap, and stack. ***Note that this is a copy for the child, the parent and the child do not share these portions of memory. The parent and the child do share the text segment, however.***

> Modern implementations don't perform a complete copy of the parent's data, stack, and heap, since `fork` is often followed by an `exec`. Instead, a technique called ***copy-on-write(COW)*** is used. These regions are shared by the parent and the child and have their protection changed by the kernel to read-only. **If either process tries modify these regions, the kernel then makes a copy of that piece of memory only, typically a "page" in a virtual memory system.**

Let's write some code here:

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int globvar = 6;
char buf[] = "a write to stdout\n";

int
main(void)
{
    int var;
    pid_t pid;

    var = 88;
    if (write(STDOUT_FILENO, buf, sizeof(buf)-1) != sizeof(buf) - 1)
    {
        printf("write failed");
        exit(1);
    }
    printf("before fork\n");

    if ((pid = fork()) < 0)
    {
        printf("fork error");
        exit(1);
    } else if (pid == 0)
    {
        globvar++;
        var++;
    } else
    {
        sleep(2);
    }
    printf("pid = %ld, glob = %d, var = %d\n", (long)getpid(), globvar, var);
    exit(0);
}
```

```sh
$ gcc fork.c
$ ./a.out

a write to stdout
before fork
pid = 20408, glob = 7, var = 89
pid = 20407, glob = 6, var = 88
```

> When we write to standard output, we subtract 1 from the size of `buf` to avoid writing the terminating null byte. Although `strlen` will calculate the length of a string not including the terminating null byte, `sizeof` calculates the size of the buffer, which does include the terminating null byte. **Another difference is that using `strlen` requires a function call, whereas `sizeof` calculates the buffer length at compile time**, as the buffer is initialized with a known string and its size is fixed.