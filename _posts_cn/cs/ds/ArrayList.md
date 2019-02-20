---
title: ArrayList
tags: 数据结构
date: 2019-01-21
---

### ArrayList

- [操作](#操作)
- [总结](#总结)

现在我们来讨论一下ArrayList，我们知道在访问链表某个位置的元素时时间复杂度为 O(n), 如果你想让这个操作变的更快一些，你应该使用 Arraylist, 它在访问某一个你指定位置（索引）的元素的时候，时间复杂度为 O(1)。因为 ArrayList 的底层是数组，你可以一步完成 ArrayList 的访问，仅仅是 arr[index]。
接下来我们来看看 ArrayList 大概长什么样。

![ArrayList](https://sherlockblaze.com/resources/img/cs/arraylist/arraylist.png)

在我的版本里，仍然有一个头节点，保存两个值，其中一个值用来描述 ArrayList里存放了多少个数据，另一个值用来保存 目前 Arraylist 底层数组的容量，即 ArrayList 当前可保存的数据，此外，头节点里还保存了一个指针，用来指向我们真正用来存放数据的数组的第一个元素。我们先来看看它长什么养。

![Arraylist With HeadNode](https://sherlockblaze.com/resources/img/cs/arraylist/arraylist_with_head_node.png)

***我们首先清楚以下几点:***

> **Size**: 已存放在 ArrayList 里的元素的个数
> **Capacity**: 目前最多可存放到 ArrayList 里元素的个数
> **ArrayList**: 指向存放数据的底层数组的第一个元素的指针

接下来，我们一起复习一下 ArrayList 的基本操作。

#### 操作

- [插入](#插入)
- [删除](#删除)

##### 插入

如果你想像 ArrayList 的最后插入一个元素，很简单，仅仅是 `arr[L->Size] = Value`，但是如果我们想在元素中间插入一个元素，就会稍微复杂一些，我们这样做。

在这个例子里，我们想要插入元素 9 在索引 2 的位置。
第一步，我们需要把在索引2到数组结尾的元素都向后移动一格。

![Insert Step1](https://sherlockblaze.com/resources/img/cs/arraylist/insert_step1.png)

第二步，我们插入目标元素。 `array[2] = 9`
![Insert Step2](https://sherlockblaze.com/resources/img/cs/arraylist/insert_step2.png)

完成！很简单是吧，这个操作的时间复杂度为 O(n)。当然，不要忘记改变 **Size** 的值，它在完成插入操作后值应该加一。

![Insert Successed](https://sherlockblaze.com/resources/img/cs/arraylist/insert_successed.png)

**但是，如果我们想要插入新元素的时候容量不够怎么办？**

下面是问题的答案。

第一步，我们先得到一个值，叫 NewCapacity，在我的版本里，容量的变化遵循下面的等式：`NewCapacity = OldCapacity / 2 * 3 + OldCapacity`， 你可以修改成你喜欢的等式。 然后我们以这个容量为基准申请一个新的数组。

![Insert Without Enough Room Step1](https://sherlockblaze.com/resources/img/cs/arraylist/insert_without_enough_room_step1.png)

第二步，我们把原来数组中的所有元素按顺序拷贝到新数组中。

![Insert Without Enough Room Step2](https://sherlockblaze.com/resources/img/cs/arraylist/insert_without_enough_room_step2.png)

然后我们就可以向往有足够容量的 ArrayList 里插入元素一样进行插入操作了

![Insert Without Enough Room Step3](https://sherlockblaze.com/resources/img/cs/arraylist/insert_without_enough_room_step3.png)

![Insert Without Enough Room Success](https://sherlockblaze.com/resources/img/cs/arraylist/insert_without_enough_room_successed.png)

我们简单看一下代码：

+ 在末尾插入：

```c
void
Insert(List L, ElementType value)
{
    Array array;
    if (L->Size >= L->Capacity)
        IncreaseCapacity(L);
    array = L->ArrayList;
    *(array + L->Size) = value;
    L->Size += 1;
}
```

+ 在指定位置上插入：

```c
void
InsertAt(List L, int index, ElementType value)
{
    Array array;
    if (index > L->Size)
        FatalError("Insert Failed. Illegal index.");
    if (L->Size + 1 > L->Capacity)
        IncreaseCapacity(L);

    array = L->ArrayList;
    if (index == L->Size)
        Insert(L, value);
    else
    {
        MoveTheValuesBackwards(L, index);
        *(array +  index) = value;
    }
    L->Size += 1;
}
```

+ 扩充：

```c
void 
IncreaseCapacity(List L)
{
    Array array;
    int Capacity = L->Capacity;
    int NewCapacity = Capacity / 2 * 3 + Capacity;
    array = (ElementType *)malloc(sizeof(ElementType) * NewCapacity);
    if (array == NULL)
        FatalError("No Enough Room!!");
    for (int i = 0; i < L->Size; ++i)
    {
        *(array + i) = *(L->ArrayList + i);
    }
    L->ArrayList = array;
    L->Capacity = NewCapacity;
}
```

##### 删除

然后我们来讨论一下删除操作。和在尾部插入一个元素一样，如果你想在尾部删除一个操作，很简单，`L->Size -= 1`。但如果你想在中间删除一个元素？？那我们应该这么做。

第一步，把在目标索引到最后一个元素存放的索引位置的所有元素，向前移动一格。

![Delete Step1](https://sherlockblaze.com/resources/img/cs/arraylist/delete_step1.png)

然我我们再这样做就可以了: `L->Size -= 1`。完成，也很简单，不过时间复杂度仍然是 O(n)。

![Delete Step2](https://sherlockblaze.com/resources/img/cs/arraylist/delete_step2.png)

**Delete Successed!!!**
![Delete Successed](https://sherlockblaze.com/resources/img/cs/arraylist/delete_successed.png)

同样简单看一下代码：

```c
void
Delete(List L)
{
    L->Size -= 1;
}

void
DeleteAt(List L, int index)
{
    if (index > L->Size)
        FatalError("Delete Failed. Illegal index.");
    if (index + 1 == L->Size)
        Delete(L);
    else
    {
        MoveTheValuesForWard(L, index);
        L->Size -= 1;
    }
}
```

#### 总结

我们应该记得在链表中删除和插入元素时，时间复杂度是 O(1)，现在我们能看到 ArrayList 和链表的不同。

**这里, 我们只讨论一些基本操作。**

| 实现 | 在尾部删除 | 在尾部插入 | 删除 | 插入 | 访问某一位置的元素 | 访问指定值的元素 |
| --- | --- | --- | --- | --- | --- | --- |
| LinkedList | O(1) | O(1) | O(1) | O(1) | O(n) | O(n) |
| ArrayList | O(1) | O(1) | O(n) | O(n) | O(1) | O(n) |

我们应该保持对这些数据的敏感性，好选择正确的方式去存放你的数据。如果你想查看所有代码，可以去我的[Github](https://github.com/sherlockblaze/all_knowledge_review/blob/master/Data_Structures/lists/arraylist.h)上。