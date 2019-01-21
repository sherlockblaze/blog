---
title: Doubly LinkedList
---
### Doubly LinkedList

- [Operations](#doublylinkedlist_operations)
- [Conclusion](#doublylinkedlist_conclusion)

Sometimes it's convenient to traverse lists backwards. We just add an extra field to the data structure, containing a pointer to the previous node.

Here is what Doubly LinkedList looks like.

<!-- more -->

![Doubly LinkedList](https://i.loli.net/2019/01/21/5c450d4b7b059.png)

In my version, there's still has a head node of the list, and it's value equals the total number of valid nodes.

![Doubly LinkedList With Head Node](https://i.loli.net/2019/01/21/5c450d4b75a87.png)

Now we can see the basic operations of doubly linkedlist.

<h4 id="doublylinkedlist_operations">Operations</h4>

- [Insert](#doublylinkedlist_insert)
- [Delete](#doublylinkedlist_delete)

<h5 id="doublylinkedlist_insert">Insert</h5>

![Insert Step1](https://i.loli.net/2019/01/21/5c450d4b8f1ec.png)

First Step, we get a new node called NewNode, and get it ready for insertion. Then we let the Next Pointer of NewNode equals the Next pointer of previous node.

![Insert Step2](https://i.loli.net/2019/01/21/5c450d4b89e0e.png)

Then we need to let the Previous pointer of the node that the Next pointer of previous node points to points to the NewNode.

![Insert Step3](https://i.loli.net/2019/01/21/5c450d4b8745a.png)

Let the Next pointer of Previous node points to NewNode

![Insert Step4](https://i.loli.net/2019/01/21/5c450d4b8c789.png)

Now we can let the Previous pointer of NewNode points to the Previous node.

**And then we finished the insertion. Insert at the end of the list is similar to this.**

![Insert Successed](https://i.loli.net/2019/01/21/5c450d4b84b7b.png)

<h5 id="doublylinkedlist_delete">Delete</h5>

First, we call the node we want to delete TargetNode.

![Delete Step1](https://i.loli.net/2019/01/21/5c450d4b7fb36.png)

As the picture shows, we should let the previous pointer of the next node of the TargetNode points to the previous node of the TargetNode.

![Delete Step2](https://i.loli.net/2019/01/21/5c450d4b91a2b.png)

Then we let the Next pointer of previous node points to the next node of the TargetNode.

**Finished!! Just don't forget to free the space of TargetNode.**

![Delete Successed](https://i.loli.net/2019/01/21/5c450d4b82486.png)

<h5 id="doublylinkedlist_conclusion">Conclusion</h5>

The cost of insertion or deletion still O(1).
It's just as same as [linkedlist](https//sherlockblaze.com/2019/01/20/LinkedList) -- The standard implementation.