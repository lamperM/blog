---
title: "Improving the quality of C code"
description: Some useful rules to improve programming quality of C.
tags: [c]
date: 2022-06-14T17:59:22+08:00
---

## 添加更多的编译选项(comiler options)来防止bug

对于我常用的`GCC`, 推荐开启一下的compiler options:

* `-Wall`: enable a lot of common warnings

* `-Wno-format-truncation`: warns about the snprintf output buffer not being 
large enough for a corresponding “%s” in the format string.

* `-Werror`: turn warnings into errors.

&nbsp;
## 动态申请的空间到底要不要释放

When using a barebones embedded OS, you absolutely need to tightly manage your memory.

但是, 如果你是写应用业务的代码, 特别是在内存足够的场景下. 最好不要手动释放内存, 
因为当线程/进程退出时, 操作系统会自动帮我们释放. **某些情况下, 释放内存的操作会很大程度上增加逻辑的复杂度**.

> 如果你是一个内核程序员, 则必须手动的释放. 不用怀疑.

&nbsp;
### 尽可能在创建变量时赋初值

放置某些变量创建后是 `magic value`. 而使用这些变量可能不会立马导致错误, 但是这是一个隐患.

但这会产生一个问题, 有时我们定义变量之后的不久之后就会对其赋予正确的值, 这时候初值就是
多余的. 而且维护者可能认为这个值是meaningful, 这就要求我们如果要赋初值, 就要说明这个值
仅仅是**无意义的**初值.


&nbsp;
### 使用`#define`, `enum`

对于代码在不同地方使用的同一个值, 应使用`#define`来声明使得代码**maintainable**.

如果这些值有多个且能规划为同一类别, 则还可将`#define`的方式换为`enum`. 这会使代码更加**meaningful**

> 使用`enum`使还要注意其所占内存空间在不同架构中可能不同的问题, see [enum的优势和漏洞](https://www.cnblogs.com/bluettt/p/16041867.html)


&nbsp;
### 使用`typedef`优化function pointer


&nbsp;
### 重定义一套自己的类型

在开发大项目时, 需要考虑可移植性的情况下, 最好利用`typedef`对类型进行重定义.
```c
#if SYSTEM1
    typedef  int  INT32;
    ...
#else
    typedef  long INT32;
    ...
#endif
```

如上, 对于某些架构`int`类型可能不是32bit, 此时就要使用`long`. 这种定义的方式会保证我们的系统
在任何架构中都不会出现类型的bug. 而且也增加了代码的**readability**.


&nbsp;
## 善用`~0`

在做嵌入式编程时, 有时在设置掩码(mask)或者其他情况会要用到**全1**的变量值, 你是否经常这样声明?
```c
int mask = 0xffff;
```

暂且不谈`int`类型到底占多少字节的问题. 就像上面一样, 我们程序员经常忘记某个类型的大小, 
而少添加了`f`. 会导致变量`mask`的值不是全1(32位情况下).

这是要变换一下思维, 使用`~0`的定义方法就可轻松化解, 无需管变量的类型是什么.
```c
int mask = ~0;
```

&nbsp;
## 合理的使用`goto`语句

在大学课堂中, 我们老师说过禁止使用`goto`语句, 但却没有给出明确的原因.

实际上, **合理的**使用`goto`能够极大的减少程序的冗余度.

`goto`语句常用于程序出现错误要退出时, 可能有多个情况会使用重复的代码处理, 
例如释放一些allocated memory. 相较于使用`flag`, 使用`goto`显然更加clearly and readability. 

所以, 在面对重复的错误处理代码时, 想想能不能用`goto`进行优化. 当然, **避免过早优化**.

> 注意, `goto`出现的场景其实很受限. Never use a backward `goto` or jump into control statements.

&nbsp;
## 永远为你的函数设置error return value

一旦你的函数可能被其他人调用, 那么养成设置return value的习惯. 即便你现在的实现
并不会产生任何错误, 也请返回`success`. 

这样做的原因是, caller可以根据你的定义做错误判断, 即便以后你的实现加上了出错情况,
上层的代码也不需要修改.


&nbsp;
## 变量类型的选择

* 名字, 特定不变的字符串使用`const char *`, 甚至`const char const*`
* 长度使用`size_t`
* 表示类型的参数尽可能使用`enum`
* 循环变量i使用`signed`, 避免溢出后出错

&nbsp;
## Reference

[How I Improve My (C) Code Quality](https://www.msweet.org/blog/2020-12-31-how-i-improve-my-c-code-quality.html)

[Ten Fallacies of Good C Code](https://www.codeproject.com/Articles/357065/Ten-Fallacies-of-Good-C-Code)
