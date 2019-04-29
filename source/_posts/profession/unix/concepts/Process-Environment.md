---
title: Coding Under Unix -- Process Environment
tags:
  - Unix
  - Processes
  - Coding Under Unix
date: 2019-03-05
---

## Process Environment

> All the summaries are from the book named **[Advanced Programming in the Unix](https://www.amazon.com/Programming-Environment-Addison-Wesley-Professional-Computing/dp/0201563177/ref=sr_1_fkmrnull_1?crid=2YVJXTV3JD1HC&keywords=advance+programming+in+unix&qid=1551765355&s=gateway&sprefix=advance+unix%2Caps%2C467&sr=8-1-fkmrnull)**.

### Questions

- How the `main` function is called **when the program is executed,**?
- How command-line arguments are passed to the new program?
- What the typical memory layout look like?
- How to allocate additional memory?
- How the Process can use environment variables?
- How many ways the process to terminate?

### How the `main` function is called?

Let's see the prototype for the `main` function in C:

```c
int main(int argc, char *argv[])
```

**`argc`**: The number of command-line arguments
**`argv`**: An array of pointers to the arguments

And when C program is executed by the kernel -- by one of the `exec` functions -- a special start-up routine is called before the `main` function is called. And we mentioned in [this blog](https://sherlockblaze.com/2019/02/26/linux/how-linux-works/TheBigPictureOfLinux/#System-calls-and-Support). **The executable program file specifies this routine as the starting address for the program**; this is set up by the link editor when it is invoked by the C compiler. This start-up routine takes from the kernel -- the command-line arguments and the environment.

### How many ways the process to terminate?

There are eight ways for a process to terminate. Normal termination occurs in five ways:

- Return from `main`
- Calling `exit`
- Calling `_exit` or `_Exit`
- Return of the last thread from its start routine
- Calling `pthread_exit`

Abnormal termination occurs in three ways:

- Calling `abort`
- Receipt of a signal
- Response of the last thread to a cancellation request

#### Exit Functions

Three functions terminate a program normally: `_exit` and `_Exit`, which return to the kernel immediately, and `exit`, which performs certain clean processing and then returns to the kernel.

```c
#include <stdlib.h>
void exit(int status);
void _Exit(int status);

#include <unistd.h>
void _exit(int status);
```

The reason for the different headers is that `exit` and `_Exit` are specified by ISO C, whereas `_exit` is specified by POSIX.1

Historically, **the `exit` function has always performed a clean shutdown of the standard I/O library: the `fclose` function is called for all open streams.**

All three exit functions expect a single integer argument, which we call the ***exit status***.

### How command-line arguments are passed to the new program?

When a program is executed, the process that does the `exec` can pass command-line arguments to the new program. This is part of the normal operation of the UNIX system shells.

```c
#include <stdio.h>
#include <stdlib.h>

int
main(int argc, char *argv[])
{
	int		i;

	for (i = 0; i < argc; i++)
		printf("argv[%d]: %s\n", i, argv[i]);
	exit(0);
}
```

And we compile and run this program:

```c
$ ./a.out arg1 arg2 arg3
argv[0]: ./a.out
argv[1]: arg1
argv[2]: arg2
argv[3]: arg3
```

We can use this kind of code to do the argument-processing loop:

```c
for (i = 0; argv[i] != NULL; i++)
```

Because the `argv[argc]` is a null pointer.

### How the Process can use environment variables?

**Each program is also passed an environment list.** Like the argument list, the environment list is an array of character pointers, with each pointer containing the address of a null-terminated C string. The address of the array of pointers is contained in the global variable `environ`:

```c
extern char **environ;
```

For example, if the environment consisted of five string, it could look like following picture. Here are explicitly show the null bytes at the end of each string. We'll call `environ` ***the environment pointer***, the array of pointers the environment list, and the string they point to the environment strings.

![Environment consisting](https://sherlockblaze.com/resources/img/profession/unix/environment-consisting.png)

The environment consists of `name=value` strings. Most predefined names are entirely uppercase, but this is only a convention.

Historically, most UNIX systems have provided a third argument to the `main` function that is the address of the environment list:

```c
int main(int argc, int *argv[], char *envp[]);
```

### What the typical memory layout look like?

We take a look at the memory layout of a C program. Historically, a C program has been composed of the following pieces:

- **Text segment**, consisting of the machine instructions that the CPU executes. Usually, the text segment is sharable so that only a single copy needs to be in memory for frequently executed programs, such as text editors, the C compiler, the shells, and so on. Also, the text segment is often read-only, to prevent a program fro accidentally modifying its instructions.
- **Initialized data segment**, usually called simply the data segment, containing variables that are specifically initialized in the program. For example, the C declaration:

```c
int maxcount = 100;
```

appearing outside any function causes this variable to be stored in the initialized data segment with its initial value.

- **Uninitialized data segment,** often called the "bss" segment, named after an ancient assembler operator that stood for "block started by symbol". Data in this segment is initialized by the kernel to arithmetic 0 or null pointers before the program starts executing. The C declaration:

```c
long sum[1000];
```

appearing outside any function causes this variable to be stored in the uninitialized data segment.

- **Stack**, where automatic variables are stored, along with information that is saved each time a function is called. Each time a function is called, the address of where to return to and certain information about the caller's environment, such as some of the machine registers, are saved on the stack. The newly called function then allocates room on the stack for its automatic and temporary variables. This is how recursive functions in C can work. Each time a recursive function calls itself, a new stack frame is used, so one set of variables doesn't interfere with the variables from another instance of the function.
- **Heap**, where dynamic memory allocation usually takes place. Historically, the heap has been located between the uninitialized data and the stack.

The following picture shows the typical arrangement of these segments. This is a logical picture of how a program looks.

![Typical Memory Arrangement](https://sherlockblaze.com/resources/img/profession/unix/typical-memory-arrangement.png)

> Several more segment types exist in an `a.out`, containing the symbol table, debugging information, linkage tables for dynamic shared libraries, and the like. These additional sections don't get loaded as part of the program's image executed by a process.

Note from above picture that the contents of the uninitialized data segment are not stored in the program file on disk, because the kernel sets the contents to 0 before the program starts running. The only portions of the program that need to be saved in the program file are the text segment and the initialized data.

The `size(1)` command reports the sizes(in bytes) of the text, data, and bss segments. For example:

![size(1) command](https://sherlockblaze.com/resources/img/profession/unix/size-command.png)

The fourth and fifth columns are the total of the three sizes, displayed in decimal and hexadecimal, respectively.

### Shared Libraries

Most UNIX system today support shared libraries. **Shared libraries remove the common library routines from the executable file, instead maintaining a single copy of the library routine somewhere in memory that all processes reference.**

***This reduces the size of each executable file but may add some runtime overhead,*** either when the program is first executed or the first time each shared library function is called. 

**Another advantage of shared libraries is that library functions can be replaced with new versions without having to relink edit every program that uses the library.**

Different systems provide different ways for a program to say that it wants to use or not use the shared libraries. Options for the `cc(1)` and `ld(1)` commands are typical. As an example of the size differences, the following executable file -- the classic  `hello.c` program -- was first created without shared libraries:

```sh
$gcc -static hello.c
$size a.out

 text     data	   bss	 dec	  hex filename
 743281	  20876	   5984	 770141	  bc05d	a.out
```

If we compile this program to use shared libraries, the text and data sizes of the executable file are greatly decreased:

```sh
$gcc hello.c
$size a.out

 text   data    bss    dec	    hex	filename
 1515   600	    8	   2123	    84b	a.out
```

### How to allocate additional memory?

ISO C specifies three functions for memory allocation:

1. `malloc`, which allocates a specified number of bytes of memory. The initial value of the memory is indeterminate.
2. `calloc`, which allocates space for a specified number of objects of a specified size. The space is initialized to all 0 bits.
3. `realloc`, which increases or decreases the size of a previously allocated area. When the size increases, it may involve moving the previously allocated area somewhere else, to provide the additional room at the end. Also, when the size increases, the initial value of the space between the old contents and the end of the new area is indeterminate.

```c
#include <stdlib.h>
void *malloc(size_t size);
void *calloc(size_t nobj, size_t size);
void *realloc(void *ptr, size_t newsize);

// All three return: non-null pointer if OK, NULL on error
void free(void *ptr);
```

The pointer returned by the three allocation functions is guaranteed to be suitably aligned so that it can be used for any data object. For example, if the most restrictive alignment requirement on a particular system requires that `doubles` must start at memory locations that are multiples of 8, then all pointers returned by these three functions would be so aligned.

Because the three `alloc` functions return a generic `void *` pointer, if we `#include<stdlib.h>`, we do not explicitly have to case the pointer returned by these functions when we assign it to a pointer of a different type. The default return value for undeclared functions is `int`, so using a cast without the proper function declaration could hide an error on systems where the size of type `int` differs from the size of a function's return value.

The function `free` causes the space pointed to by `ptr` to be deallocated. **This freed space is usually put into a poll of available memory and can be allocated in a later call to one of the three `alloc` functions.**

The `realloc` function lets us **change the size of a previously allocated area.** For example, if we allocate room for 512 elements in an array that we fill in at runtime but later find that we need more room, we can call `realloc`. **If there is room beyond the end of the existing region for the requested space, the `realloc` simply allocates this additional area at the end and returns the same pointer that we passed it.** But if there isn't room, `realloc` allocates another area that is large enough, **copies the existing 512-element array to the new area, frees the old area**, and returns the pointer to the new area. Because the area may move, we shouldn't have any pointers into this area.

> Note that the final argument to `realloc` is the new size of the region, not the difference between the old an new sizes. ***As a special case, if `ptr` is a null pointer, `realloc` behaves like `malloc` and allocates a region of the specified newsize***.

The allocation routines are usually implemented with the `sbrk(2)` system call. This system call expands(or contracts) the heap of the process.

Although `sbrk` can expand or contract the memory of a process, most versions of `malloc` and `free` never decrease their memory size. **The space that we free is available for a later allocation, but the freed space is not usually returned to the kernel; instead, that space is kept in the `malloc` pool.**

> Most implementations allocate more space than requested and **use the additional space for record keeping** -- the size of the block, a pointer to the next allocated block, and the like. As a consequence, writing past the end or before the start of an allocated area could overwrite this record-keeping information in another block. **These types of errors are often catastrophic, but difficult to find, because the error may not show up until much later.**

> Writing past the end or before the beginning of a dynamically allocated buffer can corrupt more than internal record-keeping information. The memory before and after a dynamically allocated buffer can potentially be used for other dynamically allocated objects. These objects can be unrelated to the code corrupting them, making it even more difficult to find the source of the corruption.

> Other possible errors that can be fatal are freeing a block that was already freed and calling `free` with a pointer that was not obtained from one of the three `alloc` functions. If a process calls `malloc` but forgets to call `free`, its memory usage will continually increase; this is called leakage. If we do not call `free` to return unused space, the size of a process's address space will slowly increase until no free space is left. During this time, performance can degrade from excess paging overhead.

> Because memory allocation errors are difficult to track down, some system provide versions of these functions that do additional error checking every time one of the three `alloc` functions or `free` is called. **These version of the functions are often specified by including a special library for the link editor.**