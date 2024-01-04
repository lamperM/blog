---
title: "Linux mmap 函数"
tags: ["Operating System"]
categories: ["Operating System"]
date: 2023-10-06T14:51:49+08:00
---

{{<figure src="/linux_addrspace_1.png" width="60%" >}}

在进程的地址空间中，栈和堆直接夹着的区域为**文件映射区**。
它的空间是动态的，和堆空间一起实现动态内存的分配与释放。

文件映射区中包含了一段段的**虚拟内存区域（也称线性区）**，代码里标识符是`struct vm_area_struct`。其中包含文件映射和匿名映射。

>匿名映射是`malloc()`的底层实现之一，当请求大块内存时，移动brk可能带来大碎片，
>不如用匿名`mmap()`来的灵活。

下图就展示了一个进程地址空间中即存在文件映射，又存在匿名映射的情况：


{{<figure src="/linux_addrspace_2.png" width="100%" >}}

## Linux 地址空间线性区组织形式

不只是文件映射区包含线性区，所有其他的区域（代码段、数据段等）都可以用线性区来描述，
统一进行维护。代码里用`struct vm_area_struct`描述一个线性区，其中重要的成员有:

- `vm_mm(struct mm_struct *)`: 指向所属的地址空间描述符
- `vm_start(unsigned long)`: 此线性区的开始
- `vm_end(unsigned long)`: 下一个线性区的开始(此线性区结束地址+1）
- `vm_next(struct vm_area_struct *)`: 指向进程线性区的 next
- `vm_rb(struct rb_node)`: 此线性区对应红黑树中的节点

此线性区的大小就可以表示为: `vm_end - vm_start`.

{{<figure src="/linux_addrspace_3.jpg" width="100%" >}}

### 双向链表和红黑树

进程虚拟内存空间中的所有 VMA 在内核中有两种组织形式：一种是双向链表，用于高效的遍历进程 VMA，这个 VMA 双向链表是有顺序的，所有 VMA 节点在双向链表中的排列顺序是按照虚拟内存低地址到高地址进行的。 第一个区在`mm_struct->mmap`, 下一次通过`vm_area_struct->vm_next`找到，依次类推。并且，`mmstruct->map_count`成员记录了进程所有线性区的数量。

另一种则是用红黑树进行组织，用于在进程空间中高效的查找 VMA， 正常来说，想要查找某个地址是否存在于进程的地址空间，遍历上述链表的效率是 O(n)。

**通常一个进程地址空间的文件映射区会有非常多的线性区**。因此，Linux2.6 引入红黑树来优化查找速度， 所有线性区同时组织成一个红黑树，
首部通过`mm_struct.mm_rb`指向。 然后每个线性区的`vm_area_struct.vm_rb`
存储节点的颜色和双亲信息。

现在，当需要插入/删除一个线性区描述符时，用红黑树查找前后元素，再操作链表进行插入。


## `mmap()`的使用方式
mmap()用于在文件映射区创建一个真实的文件映射或者匿名映射。

```c
void* mmap(void* addr, size_t length, int prot, int flags, int fd, off_t offset);
```

### 参数prot
通过 mmap 系统调用中的参数 prot 来指定其在进程虚拟内存空间中映射出的这段虚拟内存区域 VMA 的访问权限，它的取值有如下四种, **组合使用**：
```c
#define PROT_READ 0x1  /* page can be read */
#define PROT_WRITE 0x2  /* page can be written */
#define PROT_EXEC 0x4  /* page can be executed */
#define PROT_NONE 0x0  /* page can not be accessed */
```
- PROT_READ 表示该虚拟内存区域背后映射的物理内存是可读的。
- PROT_WRITE 表示该虚拟内存区域背后映射的物理内存是可写的。
- PROT_EXEC 表示该虚拟内存区域背后映射的物理内存所存储的内是可以被执行的，该内存区域内往往存储的是执行程序的机器码，比如进程虚拟内存空间中的代码段，以及动态链接库通过文件映射的方式加载进文件映射与匿名映射区里的代码段，这些 VMA 的权限就是 PROT_EXEC 。
- PROT_NONE 表示这段虚拟内存区域是不能被访问的，既不可读写，也不可执行。用于实现防范攻击的 guard page。如果攻击者访问了某个 guard page，就会触发 SIGSEV 段错误。除此之外，指定 PROT_NONE 还可以为进程预先保留这部分虚拟内存区域，虽然不能被访问，但是当后面进程需要的时候，可以通过 mprotect 系统调用修改这部分虚拟内存区域的权限。

>`mprotect` 系统调用可以动态修改进程虚拟内存空间中任意一段虚拟内存区域的权限。

### 参数flag
flags决定了这段线性区的映射方式。
```c
#define MAP_FIXED   0x10        /* Interpret addr exactly */
#define MAP_ANONYMOUS   0x20        /* don't use a file */

#define MAP_SHARED  0x01        /* Share changes */
#define MAP_PRIVATE 0x02        /* Changes are private */
```

但如果我们指定的 addr 是一个非法地址，比如 [addr , addr + length] 这段虚拟内存地址已经存在映射关系了，那么内核就会自动帮我们选取一个合适的虚拟内存地址开始映射，但是当我们在 mmap 系统调用的参数 flags 中指定了 MAP_FIXED, 这时参数 addr 就变成强制要求了，如果 [addr , addr + length] 这段虚拟内存地址已经存在映射关系了，那么内核就会将这段映射关系 unmmap 解除掉映射，然后重新根据我们的要求进行映射，如果 addr 是一个非法地址，内核就会报错停止映射。

当我们将 mmap 系统调用参数 flags 指定为 MAP_ANONYMOUS 时，表示我们需要进行匿名映射，既然是匿名映射，fd 和 offset 这两个参数也就没有了意义，fd 参数需要被设置为 -1 。当我们进行文件映射的时候，只需要指定 fd 和 offset 参数就可以了。

而根据 mmap 创建出的这片虚拟内存区域背后所映射的物理内存能否在多进程之间共享，又分为了两种内存映射方式：
- MAP_SHARED 表示共享映射，通过 mmap 映射出的这片内存区域在多进程之间是共享的，一个进程修改了共享映射的内存区域，其他进程是可以看到的，用于多进程之间的通信。
- MAP_PRIVATE 表示私有映射，通过 mmap 映射出的这片内存区域是进程私有的，其他进程是看不到的。如果是私有文件映射，那么多进程针对同一映射文件的修改将不会回写到磁盘文件上。

>这里介绍的这些 flags 参数枚举值是可以相互组合的, `MAP_PRIVATE | MAP_ANONYMOUS` 表示私有匿名映射，我们常常利用这种映射方式来申请虚拟内存