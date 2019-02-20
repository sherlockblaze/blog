---
title: LinkedList
tags: Data Structures
date: 2019-01-21
---

### LinkedList

- [操作](#操作)
- [总结](#总结)

LinkedList(链表) 看起来大概是这样：

![LinkedList](https://sherlockblaze.com/resources/img/cs/linkedlist/linkedlist.png)

> 为了避免插入/删除操作造成的线性消耗，我们需要确保一个线性表中的元素不是连续存储的。通过使用这种类型的线性表 -- 链表，我们可以让插入和删除操作的时间复杂度变为 O(1)。
链表由一系列在内存中不一定相邻的结构组成。

这篇文章里，我们主要讨论单链表。单链表中，每个节点存放了两个值：元素值 & 一个指针(以下称为 NEXT)，元素值主要存放目标存入到链表中的值，指针则主要用于指向下一个相邻的节点。并且最后一个节点的 NEXT 值为 NULL。在 ANSI C 中，我们认为 NULL 就是 0。

在我的代码中，我放置了一个头节点，里面存放了当前链表的长度，也就是已存放的元素的个数。

![With Head Node](https://sherlockblaze.com/resources/img/cs/linkedlist/linkedlist_with_head_node.png)

现在我们看一下链表基本的操作。

#### 操作

- [插入](#插入)
- [删除](#删除)

##### 插入

![Insert Step1](https://sherlockblaze.com/resources/img/cs/linkedlist/insert_step1.png)

以下是我们插入操作的第一步。

正如我们看到的，我们有A, B, C 三个节点，其中 C 节点上我们计划插入的新节点。首先我们让 C 的 Next 值等于 A 的 Next值，这样我们把 C 和 B 连接起来了。

第二步，我们让 A 的 NEXT 指针指向我们的新节点 C。

![Insert Step2](https://sherlockblaze.com/resources/img/cs/linkedlist/insert_step2.png)

最后，完成！

***插入成功!!***

![Insert Successed](https://sherlockblaze.com/resources/img/cs/linkedlist/insert_successed.png)

简单看一下代码。

+ 在最后插入元素

```cpp
// Insert a value after all elements
void
Insert(List L, ElementType value)
{
	Position NewNode, LastNode;
	NewNode = (struct Node *) malloc(sizeof (struct Node));
	if (NewNode == NULL)
		FatalError("Insert failed. No enough room!!");
	LastNode = L->Head;
	NewNode->Value = value;
	NewNode->Next = NULL;
	if (LastNode == NULL)
		L->Head = NewNode;
	else
	{
		while(LastNode->Next != NULL)
			LastNode = LastNode->Next;
		LastNode->Next = NewNode;
	}
	L->Size += 1;
}
```

+ 在指定位置插入

```cpp
// Insert a Value at the index you give
void
InsertAt(List L, int index, ElementType value)
{
	if (index > L->Size || index < 0)
		FatalError("Illegal index"); 
	if (index == L->Size)
		Insert(L, value);
	else {
		PtrToNode NewNode = (struct Node *)malloc(sizeof(struct Node));
		if (NewNode == NULL)
			FatalError("No Enough room!");
		NewNode->Value = value;
		Position TmpPointer;
		TmpPointer = L->Head;
		if (index == 0)
		{
			NewNode->Next = L->Head;
			L->Head = NewNode;
		}
		else
		{
			int i = 0;
			while (++i < index)
			{
				TmpPointer = TmpPointer->Next;
			}
			NewNode->Next = TmpPointer->Next;
			TmpPointer->Next = NewNode;
		}
		L->Size += 1;
	}
}
```

##### 删除

接下来，我们展示删除操作的步骤。

第一步，我们让 A 的 NEXT 值等于节点 C 的 NEXT 值。

![Delete Step1](https://sherlockblaze.com/resources/img/cs/linkedlist/delete_step1.png)

因为我们只有一个 NEXT 值，所以就么有任何指针指向 C 节点，C 节点就成为了无法访问的节点了。

![Delete Step2](https://sherlockblaze.com/resources/img/cs/linkedlist/delete_step2.png)

**至此，删除成功！**

![Delete Successed](https://sherlockblaze.com/resources/img/cs/linkedlist/delete_successed.png)

同样简单看一下代码：

+ 在末尾删除

```cpp
// Delete the last node of list L
void
Delete(List L)
{
	if (L == NULL || L->Size == 0)
		FatalError("Delete failed. Please try to create a list and insert some nodes into it.");
	Position P, Previous;
	P = L->Head;
	if (L->Size == 1)
	{
		L->Head = NULL;
	}
	else
	{
		while (P->Next != NULL)
		{
			Previous = P;
			P = P->Next;
		}
		Previous->Next = NULL;
		L->Size -= 1;
	}
	free(P);
}
```

+ 在指定位置删除

```cpp
// Delete the node at the index you give
void
DeleteAt(List L, int index)
{
	if (index > L->Size - 1)
		FatalError("Illegal index");
	Position P, NewNext;
	P = L->Head;
	if (index == 0)
	{
		NewNext = P->Next;
		L->Head = NewNext;
		free(P);
	}
	else
	{
		int i = 0;
		while (++i < index)
			P = P->Next;
		NewNext = P->Next->Next;
		free(P->Next);
		P->Next = NewNext;
	}
	L->Size -= 1;
}
```

#### 总结

我们知道如果单纯计算插入和删除操作的时间复杂度，那么 T(n) = O(1)。
但是如果你想计算在某个位置插入或删除某个元素的整个操作过程，那么它的时间复杂度是 O(n)。
但是单纯的插入和删除操作的时间复杂度仍然是O(1)。这里我们仅仅讨论插入和删除操作的时间复杂度。