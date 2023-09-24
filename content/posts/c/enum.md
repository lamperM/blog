---
title: "C 语言enum的使用"
date: 2023-03-09T17:18:57+08:00
tags: [c]
categories: ["C Language"]
---

## 枚举类型的优势

枚举类型完全可被宏定义替代，类如

```c
enum Furniture {
	DOOR = 1,
	DESK,
	LOCK,
}
```

与下面的代码等效

```c
#define DOOR  1
#define DESK  2
#define LOCK  3
```

那么我们如何在两种设计方法中选择呢？在我看来某些情况下使用 enum 会有以下优势：

1. 提高代码键入效率；仅适用于所需变量的值是连续的整数，就像上面的情况，可以只给第一个 DOOR 赋值，其余的值累加。如果首个变量的值要求是 0，甚至每一个都无需显式赋值
2. 提高代码的可维护性；可以划定范围，编译器也会检查类型是否正确，偶尔会有用
3. 提高代码的可读性；例如 DOOR, DESK, LOCK... 都属于家具，均定义在 Furniture 中

## 枚举类型所占的大小

枚举类型所占内存的大小，即枚举变量的大小。

由于枚举变量的赋值，一次只能存放枚举结构中的某个常数。所以 枚举变量的大小，实质是常数所占内存空间的大小（常数为 int 类型，当前主流的编译器中一般是 32 位机器和 64 位机器中 int 型都是 4 个字节），枚举类型所占内存大小也是这样。

所以默认情况下，无论枚举变量的值是多少，都是占用 4 个字节。即执行：

```c
printf("sizeof(enum Furniture) = %d\n", sizeof(enum Furniture));
```

输入的结果是 4。

### 编译选项：-fshort-enums

GCC 下关于这个编译选项的介绍：

> **-fshort-enums**
> Allocate to an enum type only as many bytes as it needs for the declared range of possible values. Specifically, the enum type is equivalent to the smallest integer type that has enough room.
> Warning: the -fshort-enums switch causes GCC to generate code that is not binary compatible with code generated without that switch. Use it to conform to a non-default application binary interface.

意思是说使用`-fshort-enum`s 后，对改枚举类型所占空间的分配就会按照实际变量的占用空间，而非总是 4 字节。

启用该选项之后，再打印它的 size 就会是 1，因为用 1 个字节就能表示所有枚举变量的值（DOOR=1，DESK=2，LOCK=3）.

这个“1”不再是固定的，根据其中枚举变量值的不同，动态调整`enum Furniture`的大小。

```c
enum Furniture {
	DOOR = 256,
	DESK,
	LOCK,
}
```

再打印它的 size，结果为 2。因为值 256 无法用 1 个字节存下。


## enum 潜在的可移植性问题

看似好像启用该选项会节约一定的内存空间，是的。但它也有一定的缺点，其一就是可移植性问题。

例如你编写的应用在编译时没有启用了该“优化”选项，默认采用 4 字节存储枚举变量。而链接的库文件在编译时却使用了“优化”选项，则库内部此枚举类型的大小可能为 1 字节。若此时恰好你有调用某个库 API，将 enum 变量作为参数进行传递，那么就会发生错误。

为避免不同库和应用程序使用“优化”选项的差异造成潜在的危险，常用的解决方案是强制使 enum 变量占用 4 个字节，无论其是否开启“优化”。实现方式是在 enum 变量末尾添加一个成员 `XXXX_END = 0xFFFFF`，例如：

```c
enum Furniture {
    DOOR = 1,
    DESK,
    LOCK,
    END = 0xFFFFF,
}
```
