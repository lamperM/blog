---
title: "C语言程序设计的一些经验"
date: 2023-02-27T19:20:20+08:00
tags: [c]
---

## 头文件的引用形式

C 中引用一个头文件有两种形式 `#include <>`和`#include ""`，在应用开发中，需要引用一些系统库文件，我们通常使用`<>`，对于自己定义的头文件，我们会使用`""`。

然而对于底层软件的开发，比如说操作系统，用到的库都是自己工程中的文件，那么此时用`""`和`<>`有时都能 work，那么它们的区别是什么呢？

搜索相关关键词得到的结论是: 两种方式的区别是**搜索文件的优先级， `""`优先搜索的当前目录，而`<>`优先搜索系统库文件目录**。对于这个系统库，即那些使用`gcc -I<dir>`参数指定的路径。
当然，如果第一优先级位置没有被找到，也会到另一个目录中搜索。这么两种方式均可，实际工程中也有部分人混合使用，毫不在意规则。但是有时会导致一些细节问题，比如说我们经常会用到`-MMD`或者类似选项生成目标文件的依赖，方便实现增量编译。此时就可能会产生一些问题。

假设你有一个头文件`inc/father.h`, 它里面会引用`inc/child.h`, 对于**根目录下**的源文件`main.c`，其引用语句该如何写呢？以下列出的几种形式都可以，任意的排列组合

```c
// 编译参数: -I. -MMD
// main.c
#include "inc/father.h"
#include <inc/father.h>

// father.h
#include "inc/child.h"
#include <inc/child.h>
#include "child.h"
#include <child.h>
```

- 如果 main.c 是使用系统库路径(`-I.`)来找到的 father.h, 即上面 main.c 的第 2 种情况，那么其生成依赖文件的形式内容都是**绝对路径**，包括 father.h 中的引用（因为即便 child.h 是相对路径找到的，相对的也是 father.h，其基准就是绝对路径）。例如:`main.o: main.c /home/xx/father.h /home/xx/child.h`
- 否则即以相对路径找到 father.h,即上说 main.c 的第 1 种，那么生成 father.h 依赖的方式**一定是相对路径，但 child.h 的形式却取决于其本身**.
  - 也就是说，如果 child.h 的寻找方式是绝对的（上面的第 1,2,4 种），那么依赖文件的形式就是`main.o: main.c inc/father.h /home/xx/child.h`.
  - 如果 child.h 的寻找方式是相对的(上面的第 3 种)，那么依赖文件的形式是`main.o: main.c inc/father.h inc/child.h`

依赖文件的形式很重要，最简单的方式是均使用绝对路径，此时不需要考虑依赖文件在 makefile 中 include 的位置，也就是不需要考虑 make 的“当前路径”。如果非得使用相对路径，那么已经要确定能够 makefile 中 include 时的 make 当前路径就是生成依赖文件的路径，否则不能建立正确的依赖关系。

实际上，在“基础架构”优秀的项目中，不可能出现或者尽量避免出来两种形式都能找到头文件的情况。比如说，我们会将源文件统一放在子目录`src/`下与头文件隔离，这样就从根本上避免了相对依赖的生成，只能通过系统库的形式来找头文件。拿上面的例子来说，正确的方式是：main.c 放入 src/中，然后不管是源文件还是头文件，都统一使用`#include "inc/xxx"`。这样做**即统一，也能保证所有的依赖都是绝对路径形式**

> 另外说一点，其实依赖文件(.d)中源文件的依赖项形式也是需要考虑的，这不能通过系统架构来解决，只能用 Makefile 的技巧来实现。比如说，我们的 make 当前目录总是根目录，而在建立 OBJS 变量时为其加上绝对路径的前缀， 从而 make 不需要进入各级子目录，生成的依赖文件也都是相对于根目录的，include 依赖文件的行为也是在根目录进行的，保证统一。

## 外部库的使用方式

最近我在开发项目是, 需要使用到 libelf 库, 我在 Github 上找到了其源代码.

我之前使用一个 lib 都是以链接的形式使用动态库/静态库, 但是既然它提供了源码, 那么我可以直接将源码拷贝到我的项目中吗? 答案肯定是可以, 那么这两种方案该如何抉择呢?

在查阅了一些资料后, 我总结了以下几个判断依据:

1. 库的大小/对编译时间的敏感度; 如果使用源代码, 每次编译项目时需要额外对库文件进行编译(起码是第一次), 而库文件的定义是不常修改的, 如果库文件比较大, 则会延长整个项目的编译时间.
2. 是否需要版本控制; 要使用的库如果需要区分版本, 或者分配给其他的团队成员使用, 那么用库的形式似乎更为方便
3. 发挥 git submodule 的优势;

> Ref: [c++ - Should I add the source of libraries instead of linking to them? - Software Engineering Stack Exchange](https://softwareengineering.stackexchange.com/questions/313907/should-i-add-the-source-of-libraries-instead-of-linking-to-them)

## `const` 修饰符的妙用

有些时候, 我们设计的结构体中会有*name*字段, 类型是`char *`. 在使用时为它分配空间, 不使用时需要回收.

其实还有另一种情况, 就是*name*要指向预先定义好的"static name list", 适用于 name 的取值是确定的范围. 例如, libdwarf 库中的描述 section name 的`dss_name` 成员.

这时, 为了防止使用者调用`free()`来释放它, 我们可以将其声明为`const char *`, 此时如果调用`free(.dss_name)`, 编译器会给出警告:

```sh
const.c:16:10: warning: passing argument 1 of ‘free’ discards ‘const’ qualifier from pointer target type [-Wdiscarded-qualifiers]
   16 |     free(dss_name);
      |          ^~~~~~~~
```
