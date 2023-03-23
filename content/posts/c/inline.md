---
title: "C语言 'inline' 关键字"
date: 2022-11-24T01:35:24+08:00
tags: [c]
---







TODO: inline 的发展历程: [Myth and reality about inline in C99 – Jens Gustedt's Blog (wordpress.com)](https://gustedt.wordpress.com/2010/11/29/myth-and-reality-about-inline-in-c99/)



## GNU89:

函数的**实现**之前添加不同的关键字:

- `inline`: 表明这个函数可能被优化. 如果没有被优化, 编译器就会视为一个常规函数的定义.

- `extern inline`: 表明这个函数可能被优化. 如果没有被优化, 编译器就将这个函数的定义**转换为该函数的声明**, 即 `extern inline func();` 因此当此函数被调用时, 可以调用一个外部的函数来替代. 如果没有函数调用它, 那么也可以没有外部的替代函数实现.
- `static inline`: 表明这个函数可能被优化. 如果没有被优化, 编译器就会视为一个**常规静态函数**.

## C99:

函数的实现之前添加不同的关键字:

- `inline`: 等效于gnu89中的`extern inline`
- `extern inline`: 等效于gnu89中的`inline`
- `static inline`: 与gnu89相同含义.

## C++:

只有`inline`一个关键字, 如果不能优化就定义为普通函数



> Ref:
>
>  [c++ - What does extern inline do? - Stack Overflow](https://stackoverflow.com/questions/216510/what-does-extern-inline-do/216546#216546)
>
> [Myth and reality about inline in C99 – Jens Gustedt's Blog (wordpress.com)](https://gustedt.wordpress.com/2010/11/29/myth-and-reality-about-inline-in-c99/)

> [C demo](https://github.com/wangloo/inline-c99-gnu89-demo) 关于以上的各种情况
