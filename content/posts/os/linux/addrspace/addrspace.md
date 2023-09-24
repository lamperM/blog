---
title: "Linux 进程地址空间 概述"
tags: ["Operating System"]
categories: ["TODO"]
date: 2023-05-07T14:51:49+08:00
---

## 何为进程地址空间?

进程地址空间由允许进程使用的若干**线性地址区域**(也称"虚拟内存区域")构成。

每个线性区域由起始地址、长度和属性来描述。

在进程刚创建时，其地址空间仅包含 3 个线性区，分别是：代码段、数据段和堆区，其中
堆区的初始大小为 0。

> 栈区虽然也属于进程使用的内存区域，但这个区域对用户是透明的，所以我们一般将其
> 归于内核管理，并非进程本身。

**线性区增加的典型情况:**

1. 使用`mmap()`为一个文件映射内存空间
2. 创建一个 IPC 共享线性区与其他进程协作
3. 调用`malloc()`扩张自己的堆区

## Linux 描述地址空间的数据结构

在进程的 tcb 中，描述地址空间相关的结构都保存在成员`mm`中，其类型为`struct mm_struct`,
其中重要的成员有：

- `mmap(struct vm_area_struct*)`: 指向所有线性区的链表头
- `mm_rb(struct rb_root)`: 指向所有线性区对象红黑树的根
- `pgd(pgd_t *)`: 指向进程的页表
- `mmlist(struct list_head)`: 指向下一个地址空间描述符(所有进程的地址空间描述符
  被链接起来)

## Linux 描述线性区的数据结构

用`struct vm_area_struct`描述一个线性去，其中重要的成员有:

- `vm_mm(struct mm_struct *)`: 指向所属的地址空间描述符
- `vm_start(unsigned long)`: 此线性区的开始
- `vm_end(unsigned long)`: 下一个线性区的开始(此线性区结束地址+1）
- `vm_next(struct vm_area_struct *)`: 指向进程线性区的 next
- `vm_rb(struct rb_node)`: 此线性区对应红黑树中的节点

此线性区的大小就可以表示为: `vm_end - vm_start`.

## 进程地址空间所有线性区的组织

进程拥有的所有线性区通过单链表串联（按地址排序），第一个区在`mm_struct->mmap`, 下一次
通过`vm_area_struct->vm_next`找到，依次类推。并且，`mmstruct->map_count`成员
记录了进程所有线性区的数量。

### 红黑树优化查找

正常来说，想要查找某个地址是否存在于进程的地址空间，遍历上述链表的效率是 O(n).

因此，Linux2.6 引入红黑树来优化查找速度， 所有线性区同时组织成一个红黑树，
首部通过`mm_struct.mm_rb`指向。 然后每个线性区的`vm_area_struct.vm_rb`
存储节点的颜色和双亲信息。

现在，当需要插入/删除一个线性区描述符时，用红黑树查找前后元素，再操作链表进行插入。

### 分配一个线性区

接口是`do_mmap()`, 参数为:

- file, offset; 如果有文件映射
- addr, len
- prot; 该线性区的权限

步骤大致包含:

1. 用红黑树确定新线性区的前后， 对应`find_vma_prepare()`
2. slab 分配一个`struct vma_area_struct`，并初始化
3. 操作链表插入，对应`vma_link()`

### 释放一个线性区

接口是`do_munmap()`

步骤大致包含:

1. 红黑树确定要删除线性区的位置，以及做分割（必要时）
2. 调用`detach_vmas_to_unmapped()`将其从链表中删除
3. `unmap_region()`删除页表项
4. 释放 `vma_area_struct` 的空间
