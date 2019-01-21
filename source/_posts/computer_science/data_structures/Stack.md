---
title: Stack
tags: Data Structures
---
### Stack

- [Operations](#Operations)
- [Conclusion](#Conclusion)

A stack is a list with the restriction that insertions and deletions can be performed in only one position. We call this behavior last in first out(**LIFO**), and the position, we call it **top**, top of a stack.

Those Pictures shows a stack can be.

<!-- more -->

It can be ...

![ArrayList Stack](https://i.loli.net/2019/01/21/5c450f7098931.png)

or It can be ...

![LinkedList Stack](https://i.loli.net/2019/01/21/5c450f70a57d3.png)

As we can see, The most recently inserted element can be examined prior to performing a Pop by use of Top routine.

#### Operation

- [Push](#Push)
- [Pop](#Pop)
- [Implemention](#Implemention)

##### Push

It easy to understand how to push a value into a stack.

![Push](https://i.loli.net/2019/01/21/5c450f70adb4e.png)

##### Pop

It also easy to understand how to pop a value into a stack.

![Pop](https://i.loli.net/2019/01/21/5c450f70ad3bb.png)

##### Implemention

Actually, stack still is a list, whatever a linkelist or arraylist, all of them can be the data structure of a stack.
In my version, I use the linkedlist as the data structure of it.

![My Stack](https://i.loli.net/2019/01/21/5c450f70b2095.png)

For better push of element, I make the head node points to the **Top**, and the last node of this linkedlist called **Bottom**. If we always insert or delete node at the index 0, we can get a stack.

![More about My Stack](https://i.loli.net/2019/01/21/5c450f70ae1e4.png)
