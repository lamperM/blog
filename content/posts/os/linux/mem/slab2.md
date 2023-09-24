---
title: "Linux SLAB 内存分配器(2): 算法"
tags: ["linux", "Operating System"]
categories: ["Operating System"]
date: 2023-05-20T18:51:49+08:00
---

上一篇介绍了数据结构，这一篇主要介绍 slab 分配器的分配和释放算法。

## 最外层接口: `kmalloc()`/`kfree()`

最上层的接口是`kmalloc(size, flag)`。

slab 分配器维护了多个不同大小的 kmem_cache，放在数组`kmem_caches[]`中,
其对应的 object 大小和该 kmem_cache 的 name 在另一个数组`kmalloc_info[]`
中，它们的下标是对应的。使得我们能根据请求分配的大小来找到对应的`struct kmem_cache`结构。
【代码】

## 专用的"cache"

上面的结构，会遍历系统初始化创建的一些内存池，来寻找一个大小满足要求的 object，
但是通常不能找到大小相等的，如果系统中存在的固定 cache 中 object 的大小太稀疏，
就容易发生空间浪费的问题。

因此，我们可以为某个特定大小的内存请求再创建一个单独的 cache，仅仅用于满足这一类
结构体的申请，也是符合 slab 分配器关于面向对象的设计思想。

slab 分配器提供的相关接口是:

- `kmem_cache_create()`: 创建一个专用 cache
- `kmem_cache_alloc()`： 从指定的 cache 里分配 object
- `kmem_cache_free()`: 释放对象到指定的 cache
- `kmem_cache_destory()`: 销毁某个 cache

## Reference

https://blog.csdn.net/u010923083/article/details/116518646?spm=1001.2014.3001.5502
