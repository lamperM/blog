---
title: "Lock"
date: 2022-06-30T11:20:29+08:00
draft: true
---

## When you need a lock

1. Do two or more threads touch a memory location?
2. Does at least one thread write to the memory location?

**If so, you need a lock!**

## Memory ordering

* The compiler and CPU can reorder your code. Legal behaviors are referred to as "memory model"
* If you use locks, you don't have to understand memory ordering
* For exotic lock-free code, you'll need to know every detail

## How to build a lock

背景为: 读写线程共同访问`ringbuffer`. 

设立一个flag? 
* flag++ -> 会编译成`non-atomic` instuction, 即翻译成read, modify, write三个内存访问过程. 
所以可能出现修改过程中另一个线程访问了old value.
* 编译器可能会reorder指令的顺序, 因为flag在置1和置0之间没有用到. `C++`, `go`, `java`等
语言可以保证多线程模型下, 访存指令不会被reorder.


指令集提供了内存顺序保障和原子操作的指令. 这就是`spinlock`的原理.


`spinlock`浪费CPU资源, 如果能让没有获得锁的线程让出CPU则好一些.

Linux中的`futex`即实现了该功能.

Kernel负责的工作:
1. 合理安排线程, 方便未来被唤醒. 通过`hashtable`来存储.
2. deschedule该线程.


[Let's Talk Locks!](https://www.youtube.com/watch?v=7OpCf6f_BAM&t=83s)
