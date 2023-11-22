---
title: "ARMv8: cache相关知识"
date: 2023-08-14T22:02:04+08:00
tags: [armv8]
categories: ["Architecture"]
---

之前在特斯拉面试的时候被问到了 cache 的 maintain 操作有哪些,
一时间竟想不起一个准确的词来, 这里就再学习一下, 把这个坑填上吧。

可能不会说的很细，目的只是把一些概念复习，做到心中大致有数。

## cache 是个硬件

cache 的本质是一种 SRAM, 容量很小, 速度很快(ns 级)。

拿 Cortex-A53 来说，共有三级 Cache：

- L1 cache 是 Core 单独的，分为数据 cache 和指令 cache，容量是 KB 级
- L2 cache 一般是 cluster 内共享，容量是 MB 级
- L3 cache 是所有 core 共享，容量是 MB 级

## cache 控制器

单独的 cache 就是一个存储设备，得有一个控制器告诉它存什么以及什么时候存。

cache 控制器的任务举个例子说：比如 cache miss 的时候，需要从主存向 cache
回填数据，然而此时 CPU 那边记着要数据，我们都知道 cache 操作的单位
是 cache line 嘛，但这是 cache 控制器会有限填充一个 cache line 中
CPU 要的那一条（或几条），最后在后台默默填充完剩下的。

cache 控制器的行为是不可配置的，软件不可见，不可编程。

## cache policy（策略）

cache policy 就说了两件事：

- cache 分配：执行 load 操作时需要分配入 cache 吗？
- cache 更新：执行 str 修改数据时需不需要与主存同步？

cache 的更新方式有两种：

1. Write back，写回；即仅修改 cache 中数据，然后设置一个 dirty 标志位
   ，当且仅当此 cache line 被 clean 时才对 dirty 数据回写主存
2. Write through，写直通；即每次修改 cache 都同步修改主存，也就不需要
   dirty 标志了

## cache 相关的内存属性

一段内存可被设置为 cacheable 或者 uncacheable，uncacheable 的内存直接访问主存。

cacheable 又可划分为 inner 和 outer 两种：

- inner: 可入L1 cache，或者L2 cache，实现定义
- outer: 可入外部板级的 cache(L3 cache)

## cache maintain（维护）

为什么需要维护cache？
- 主存中的内容更新了
- 此部分内存的权限改了或者映射改了
  
cache的三种维护操作（软件可见）：
- \[invalidation\]: 设置cache line对应的invalid位。常见于reset时，cache里的内容不可信，
  所以需要将所有的cache invalid
- \[clean\]: clean的含义是清理，将原有的标记为dirty的line都与主存进行同步。当然，
  **仅更新策略为 write back的时候才需要**
- \[zero\]: 什么也不管，直接将cache line置0，数据丢了我也不关心

>以上的三种操作都是有粒度的，包括整个 cache、指定va范围、某一路等。

>软件只能通过体系结构提供的维护指令来控制 cache。

## ARMv8获取Cache信息

想要管理好整个cache系统, 首先要了解的信息是:

1. 系统实现了几级cache?
2. Cache line 是多大?
3. 对于每一级的Cache, 它的 set/way 分别是多少?

这些信息都可以通过ARMv8的系统控制寄存器来读取. 

### CLIDR_EL1

标识每一级Cache的类型以及系统最多支持几级Cache.

![image-20220925170644091](C:\Users\wangloo\AppData\Roaming\Typora\typora-user-images\image-20220925170644091.png)

`Ctype<n>`字段用来描述缓存的类型. 系统最多支持7级缓存, 软件需要遍历`Ctype<n>`字段,  当读到的值为000时, 说明该级及以上都没有实现.

> 具体每种类型对应的值, 和其他字段的含义, RTFM

### CTR_EL0

记录了Cache line的大小, 以及Cache的策略.

![image-20220925171028981](C:\Users\wangloo\AppData\Roaming\Typora\typora-user-images\image-20220925171028981.png)

`IminLine`: 表示所有指令cache中最小的cache line, 单位是 word.

`DminLine`: 表示所有数据cache中最小的cache line, 单位是 word.

`L1IP`: 表示 L1 指令cache的策略. RTFM

### CSSELR_EL1

与CSSIDR配合工作. CSSELR用于选定查看某级的Cache, 再去读CSSIDR就是该级Cache的信息.

### CSSIDR_EL1

![image-20220925171843984](C:\Users\wangloo\AppData\Roaming\Typora\typora-user-images\image-20220925171843984.png)

`LineSize`: 该级cache的cache line.

`Associativity`: 该级Cache的way

`NumSets`: 该级Cache的set

> 该寄存器的字段分布与是否实现`FEAT_CCIDX`有关, RTFM## 获取Cache信息
