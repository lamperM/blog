---
title: "Linux SLAB 内存分配器(3): SLUB/SLOB"
tags: ["linux", "Operating System"]
categories: ["Operating System"]
date: 2023-05-26T18:51:49+08:00
---

slub 和 slob 是基于 slab 思想针对某些场景下的优化实现。

### SLUB

1. 当 slab 分配器面对过多的申请需求时，cache 中就会有多个 slab (struct slab),
   在以前的 slab 分配器设计中， slab 描述符是放在物理页中的，即物理页的结构为：
   （slab 描述符+freelist+对象 s）,管理数据结构的开销就比较大。后期 SLUB 首先将
   slab 描述符与`struct page`共用（通过 union 实现）。后面该思想被 SLAB 采纳。
2. SLAB 中每个 cache node 有三个 list: free, partial, full， 管理起来很麻烦，
   SLUB 中只有一个 partial 链表。
3. 放弃着色，效果不明显

### SLOB

SLOB 的设计更加简洁，只有 600 行左右代码（SLAB，SLUB 都是 4000+），适合小内存的嵌入式设备。

SLOB 中没有对象的概念，每个 slab 中分配的小块内存**大小可以是不同的**，
通过长度+偏移来记录下一个小块内存的位置。

另外，SLOB 基本上放弃了 cache 的思想，系统中通过创建三个全局的链表:
small, medium, large, 分别应对<256b, <1k, <PAGESIZE 的请求，
slab 直接挂在这三个链表上，因为 slab 中的内存分配大小可以不同，
用三个链表可以加速查找。

> 从思想上，SLOB 仅仅保留 slab 中分配小块内存的思想，舍弃了 cache
> 的设计方案，所以实现上非常简单。
