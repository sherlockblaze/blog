---
title: LinkedList
---
### LinkedList

- [Operations](#linkedlist_operations)
- [Conclusion](#linkedlist_conclusion)

Here is the linked list. It looks like this.

![LinkedList](https://i.loli.net/2019/01/21/5c450dedcdec0.png)

> In order to avoid the linear cost of insertion and deletion, we need to ensure that the list is not stored contiguously. By using this kind of list, we can make the cost of insertion and deletion be O(1).
The linked list consists of a series of structures, which are not necessarily adjacent in memory.

<!-- more -->

Each node contains the element and a pointer points to the next node, we call it Next pointer, And the last node's Next pointer points to NULL. And ANSI C specifies that NULL is zero.

In my version, I put a head node to save the length of the linked list.

![With Head Node](https://i.loli.net/2019/01/21/5c450dedd519d.png)

Now we can see the operations of LinkedList.

<h4 id="linkedlist_operations">Operations</h4>

- [Insert](#linkedlist_insert)
- [Delete](#linkedlist_delete)

<h5 id="linkedlist_insert">Insert</h5>

![Insert Step1](https://i.loli.net/2019/01/21/5c450dedc904d.png)

It's the first step of the insert operation.

As we can see, we got Node A, B, C, and the C is the newest node we wanna insert into this list. First we make the Next pointer of C equals Next pointer of A, then the C node's Next Pointer points to node B.

second step, we let the A's Next pointer points to our new node C.

![Insert Step2](https://i.loli.net/2019/01/21/5c450dedd05af.png)

Finally, we finished it.

***Insert Successed!!***

![Insert Successed](https://i.loli.net/2019/01/21/5c450dedd2ab6.png)

<h5 id="linkedlist_delete">Delete</h5>

We'll show two steps of delete operation.

First step, we let the node A's Next pointer equals the Next pointer of node C.

![Delete Step1](https://i.loli.net/2019/01/21/5c450dedc4676.png)

Second step, because we just get one Next pointer for each node, so, it just make no pointer points to node C.

![Delete Step2](https://i.loli.net/2019/01/21/5c450dedc6b16.png)

**So delete Successed!!**

![Delete Successed](https://i.loli.net/2019/01/21/5c450dedcb471.png)

<h5 id="linkedlist_conclusion">Conclusion</h5>

We know that if you just calculate the cost of insertion or deletion, you'll find T(n) = O(1).
But you know if we wanna insert or delete a value with speicify index, it'll cost O(n) in whole operation. But the cost of insertion or deletion still is O(1). Here, we just talk aboult the cost of insertion or deleteion.