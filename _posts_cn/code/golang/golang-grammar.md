---
title: Golang 语法
tags:
  - Golang
  - 编程语言
  - 编程
date: 2019-05-07
---

先讨论一些规范问题：

1. 如何将代码组织成包
2. 如何操作这些包

## 包

有这么一些目录，目录下存放一些列以 `.go` 为扩展名的相关文件，这个目录，称之为**包**。

同一个目录下的所有 `.go` 文件必须声明为同一个包名，比如包为 `beauty`，则需要在代码中标明 `package beauty`。

### main 包

所有用 Go 语言编译的可执行程序都必须有一个名叫 `main` 的包，当编译器找到 `main` 包，会去找寻 `main()` 函数，如果没有，则不会生成可运行二进制文件。

### 导入

导入一个包极其简单 ---- `import "beauty"`。注意，如果你导入了一个包，却不用，编译器会报错。

### init 函数

如你在[Golang 初探](https://sherlockblaze.com/2019/05/07/code/golang/golang-start/)所看到的，在 `matcher` 中，你会看到 `init` 函数，如果你以这种方式 `import _ "beauty"` 导入包，那么会在程序开始运行时，执行这个包下面的 `init` 函数。

## 数组

### 使用

```golang
// 声明一个长度为5的数组，其中的值初始化为0
var array1 [5]int
// 声明一个长度为5的数组，值为1、2、3、4、5
array2 := [5]int{1, 2, 3, 4, 5}
// 声明一个值为1、2、3、4、5，长度自动生成，为5
array3 := [...]int{1, 2, 3, 4, 5}
// 声明一个长度为5的数组，下标为1的值为1，下标为2的值为2，其他位置为0
array4 := [5]int{1: 1, 2: 2}
// 声明包含5个元素的指向整数的数组
array5 := [5]*int{0: new(int), 1: new(int)}
*array5[0] = 1
*array5[1] = 2
// 将数组array1赋值给array6，注意，是全盘复制，到时候你会有两个一样的数组
var array6 [5]int
array6 = array1
```

在函数间直接传递数组是直接**拷贝数组**，如果数组过大，直接传递指针即可。通过 `&` 符号，可以获取指定数组的地址，如：`&array1`。

## 切片

### 什么是切片

**切片**是一种数据结构，这种数据结构便于使用和管理数据集合。切片是围绕动态数组的概念来构建的，可以按需自动增长和缩小。切片的动态增长是通过函数 `append` 来实现的。同时，**切片的底层内存也是在连续块中分配的，所以切片还能获得索引、迭代以及为垃圾回收优化的好处。**

### 实现原理

![](https://sherlockblaze.com/resources/img/code/golang/slice.png)

### 使用

```golang
// 声明一个长度、容量都为5的切片
slice1 := make([]int, 5)
// 声明一个长度为3、容量为5的切片
slice2 := make([]int, 3, 5)
// 声明一个长度为3、容量为3的切片
slice3 := []int{1, 2, 3}
// 声明一个容量为长度为101、容量为101的切片，第101个元素值为100
slice4 := []int{100: 100}
// 声明 nil 切片
slice5 := []int
// 声明一个空切片
slice6 := make([]int, 0)
// 声明一个空切片
slice7 := []int{}
```

```golang
slice := []int{1, 2, 3, 4, 5}
newSlice := slice[1:3]
```

通过一张图片观察上面两行代码所做的事。

![](https://sherlockblaze.com/resources/img/code/golang/assign-a-slice-to-another.png)

接着我们再通过一段代码来看如何对切片进行遍历:

```golang
slice := []int{1, 2, 3, 4}
for index, value := range slice {
    fmt.Println("Index: %d Value: %d\n", index, value)
}
```

需要注意的只有两点：

1. 使用 `range` 函数获得的两个值，一个是索引，一个是索引对应的值。
2. `range` 其实是对每个元素提供了一个副本。

你可以通过下划线 `_` 来忽略函数返回的值。

```golang
slice := []int{1, 2, 3, 4}
for _, value := range slice {
    fmt.Println("Value: %d\n", value)
}
```

那么如何在函数中传递切片呢？

```golang
slice := make([]int, 1e6)

slice = foo(slice)

func foo(slice []int) []int {
    ...
    return slice
}
```

1. 在一个64位的机器上，一个切片需要24字节的内存：指针字段8字节、长度和容量分别需要8字节
2. 函数中传递时仅复制切片本身，不复制底层数组

## 映射

### 什么是映射

映射是一种数据结构，用于存储一系列无需的键值对。

### 实现

![](https://sherlockblaze.com/resources/img/code/golang/hashmap-impl.png)

映射的散列表包含一组桶。在存储、删除或者查找键值对的时候，所有操作都要先选择一个桶。**把操作映射时制定的键传给映射的散列函数**，就能选中对应的桶。

**散列函数的目的是生成一个索引，这个索引将最终将键值对分布到所有可用的桶里。**

### 使用

先看怎么创建、初始化

```golang
dict := make(map[string]int)
dict := map[string]int{"Red":1, "Black":0}
// 声明一个值为 nil 的映射，这种映射无法存放键值对，注意
var colors map[string]int
```

只需要注意一点，作为键的类型，可以是内置的类型，也可以是结构类型，只要这个类型可以使用 `==` 来比较。

```golang
value, exists := colors["Blue"]
if exists {
    return true
}
```

通过这种方式，可以判断是否存在需要的键值对。

```golang
colors := map[string]string {
    "AliceBlue": "#f0f8ff"
}

for key, value := range colors {
    fmt.Println("Key: %s Value: %s\n", key, value)
}
```

通过上述代码，我们可以轻松遍历 map 中的键值对，同样，你可以使用 `_` 符号来忽略函数的某个/所有返回值。

Golang 中删除映射中的键值对也是很方便的。

```golang
colors := map[string]string {
    "AliceBlue": "#f0f8ff"
}

delete(colors, "AliceBlue")
```

那么如何在函数中传递使用映射呢？

```golang
func main() {
    colors := map[string]string {
        "AliceBlue":   "#f0f8ff",
        "Coral":       "#ff7F50",
        "DarkGray":    "#a9a9a9",
        "ForestGreen": "#228b22",
    }

    for key, value := range colors {
        fmt.Println("Key: %s Value: %s\n", key, value)
    }

    removeColor(colors, "Coral")

    for key, value := range colors {
        fmt.Println("Key: %s Value: %s\n", key, value)
    }
}

func removeColor(colors map[string]string, key string) {
    delete(colors, key)
}
```

通过运行上述代码，我们可以发现，在调用了 `removeColor` 函数之后再遍历映射，会发现被删除的元素不存在了。这其中的道理很简单，传递映射时并没有对其进行复制，使用的底层数组仍然是同一个。

## 类型系统

Golang 是一种静态编译的语言 -- 编译器需要在编译时知晓程序里每个值的类型，用于提供编译器对代码进行一些性能优化，提高执行效率。

值的类型给编译器提供两部分信息:

1. 需要分配多少内存
2. 这段内存表示什么

### 结构体

#### 什么是结构体

Golang 允许用户定义类型，当用户声明一个新类型时，这个声明就给编译器提供了一个框架，告知必要的内存大小和表示信息。

#### 声明

有两个方式声明结构体。

1. 利用 `struct` 关键字

```golang
type user struct {
    name       string
    email      string
    ext        int
    privileged bool
}
```

2. 基于一个已有的类型，将其作为新类型的类型说明

```golang
type Length int64
```

#### 初始化

```golang
yorr := user {
    name:       "yorr",
    email:      "beauty@love.com",
    ext:        1,
    privileged: true, // 别漏了最后一个逗号
}

// 用下面这种方式，需要按照结构体声明里的顺序来赋值
yorr := user {"yorr", "beauty@love.com", 1, true}

// 可以使用另一个类型来作为另一个类型中的字段
type struct admin {
    person user
    level string
}

yorr := admin {
    person: user {
        name:      "yorr",
        email:     "beauty@love.com",
        ext:       1,
        privilege: true,
    },
    level: "top",
}

// 将已有类型作为新类型
type Length int64
var length Length
length = int64(100)
```

> 嵌入类型: Golang 允许用户扩展或者修改已有类型的行为，这个功能对代码复用很重要，在修改已有类型以符合新类型的时候也很重要。通过 `嵌入类型` 完成。

```golang
package main

import (
    "fmt"
)

type user struct {
    name  string
    email string
}

func (u *user) notify() {
    fmt.Printf("Sending user email to %s<%s>\n",
        u.name,
        u.email)
}

type admin struct {
    user
    lever string
}

func main() {
    ad := admin {
        user: user{
            name:  "sherlock blaze",
            email: "sherlockblaze@love.com"
        },
        level: "top",
    }

    // 可以直接访问内部类型的方法
    ad.user.notify()
    // 内部类型的方法也可以被提升到外部类型
    ad.notify()
}
```

以上我们注意到一个函数 -- `notify`。可以给我们引入一个新的话题 —— **值接收者**、**指针接收者**。

### 值接收者&指针接收者

```golang
func (u user) notify() {
    mt.Printf("Sending user email to %s<%s>\n",
        u.name,
        u.email)
}

func (u *user) changeEmail(email string) {
    u.email = email
}
```

`notify` 和 `changeEmail` 称为结构体 `user` 的方法，如果一个函数有接收者，这个函数就被称为**方法**。在 `func` 关键字和方法名之间的参数，称之为 **接收者**。而接收者分为两类，**值接收者**和**指针接收者**。

```golang
package main

import (
    "fmt"
)

type user struct {
    name  string
    email string
}

func (u user) notify() {
    fmt.Printf("Sending User Email To %s<%s>\n",
        u.name,
        u.email)
}

func changeEmail(email string) {
    u.email = email
}

func main() {
    bill := user{"Bill", "bill@email.com"}
    bill.notify()

    lisa := &user{"Lisa", "lisa@email.com"}
    lisa.notify()

    bill.changeEmail("bill@newdomain.com")
    bill.notify()

    lisa.changeEmail("lisa@newdomain.com")
    lisa.notify()
}
```

我们可以看到，`bill` 作为一个值，可以调用值接收者声明的方法，也可以调用指针接受者声明的方法，其实里面有一个语法糖。在用值对象调用指针接收者声明的方法时，golang 底层做了这样一个操作 `(&bill).changeEmail()`，同理，指针对象调用值接收者声明的方法时，做了这样的操作 `(*lisa).notify()`。

使用**值接收者**方法时，实际上会创建一个对象的副本，**指针接收者**则是利用指针，如果修改，会直接修改结构体里对应的项。

### 内置类型

内置类型是由语言提供的一组类型。如: 数值类型、字符串类型和布尔类型。这些类型本质上是原始的类型，因此，当对这些值进行增加或者删除的时候，会创建一个新值。**基于这个理论，当把这些类型的值传递给方法或者函数时，应该传递一个对应值的副本。**

```golang
func Trim(s string, cutset string) string {
    if s == "" || cutset == "" {
        return s
    }
    return TrimFunc(s, makeCutsetFunc(cutset))
}
```

`Trim` 函数传入一个 `string` 类型的值作操作，在传入一个 `string` 类型的值用于查找，之后