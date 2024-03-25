---
title: "AArch64 MMU介绍"
date: 2022-09-29T08:01:33+08:00
tags: [armv8]
categories: ["Architecture"]
---

## Introduction

MMU: 专用于将虚拟地址转换为物理地址. 通常配合分页机制来工作.

页表: 页表中的表项包含提供虚拟地址和物理地址之间的映射. 

MMU就是直接访问页表, 并且通过将频繁使用的映射缓存到TLB中.

### MMU 的结构

MMU是一种硬件, 可以通过在适当的安全状态下对其进行配置. 每个Core都有自己的MMU, 每个MMU包括:

1. 一个TLB, 缓存最近访问的映射.
2. 一个Table Walk Unit, 从内存中查询页表, 得到最终的虚拟地址-物理地址的映射.

MMU 控制着整个系统的缓存策略, 内存属性和访问权限. MMU开启后, 软件发出的所有内存访问都使用虚拟地址, 要求MMU为每次访问进行地址转换.

### MMU 的配置

在启用MMU前, 必须告知其页表存放的位置. 

### MMU 地址转换的过程

对于每个转换请求, MMU首先检查TLB是否已经对该地址缓存, 如果该地址未缓存, 则需要遍历页表.

页表遍历单元在页表中搜索相关的映射表项. 

- 一旦找到映射, MMU就会检查权限和属性. 决定允许本次访问, 或者发出故障信号.
- 若未找到映射, 则触发缺页异常. 

### 页表的工作原理

页表的工作方式是将虚拟地址空间和物理地址空间划分为大小相等的块, 称为页面. 

页表中的每个表项对应着一块虚拟地址空间中的块, 表项的值就是这块虚拟地址空间对应的物理地址块, 以及访问物理地址时要使用的属性.

在查表过程中, 将虚拟地址分为两部分:

- 高阶位用作页表的索引. 用来找到对应的物理块
- 低地址是块内的偏移量, 不会因为映射而改变. 页表项中的物理地址与该偏移组合形成用于访问内存的物理地址.

#### 多级页表

实际实现中, 多采用多级页表的方案, 各级页表自定向下组成树的形式, 协作实现虚拟到物理地址的转换.

树中的分支成为页目录, 页目录中的表项不是直接存储目标物理地址, 而是下一级页表的地址; 最后一级页表的表项中保存着目标物理地址. 

> 多级页表是减小页表占用存储空间过大的有效方案.

顶级页表将地址空间划分为大块, 每个表项可以指向大小相等的内存块. 也可以指向将块进行再次细分的下一级页表. 支持大块的优点:

- 大的内存块需要查表的次数更少
- 提升TLB的效率, 因为一个TLB表项覆盖更大的内存区域.

凡事都是有利有弊, 使用大块也增加了内存浪费, 实际使用时需要根据需要来权衡.

## 内存类型

### 普通类型内存

普通类型的内存是弱一致性的(weakly ordered)内存模型, 没有额外的约束, 可以提供最高的内存访问性能. 

通常代码段, 数据段以及其他数据都放在普通内存中. 

普通内存允许处理器做很多优化, 如分支预测, 数据预取, Cache line预取, 乱序执行等.

### 设备类型内存

CPU访问设备内存会有很多限制, 如不能进行数据预取等. 设备类型的内存严格按照指令的顺序来执行的.

设备类型内容通常留给设备来访问, 例如中断控制器(GIC), 串口, 定时器等.

## 两套页表

* 当CPU访问的地址属于*用户空间*时, MMU会自动选择**TTBR0**指向的页表. 
* 当CPU访问的地址属于*内核空间*时. MMU会自动选择**TTBR1**指向的页表

EL2和EL3没有TTBR1, 只有TTBR0. 也就意味着:

• If EL2 is using AArch64, it can only use Virtual Addresses in the range 0x0 to 0x0000FFFF_FFFFFFFF.
• If EL3 is using AArch64, it can only use Virtual Addresses in the range 0x0 to 0x0000FFFF_FFFFFFFF.

## 越权, 越界

在未使用虚拟地址空间之前, 所有的用户程序都可以访问全部的物理内存, 所以恶意程序可以修改其他程序的内存数据, 这使得整个系统处于危险的状态. 每个进程的地址空间都要受到保护, 以免被其他进程有意/无意的破坏.

现代操作系统中, 每个进程都有独立的虚拟地址空间. 在进程的角度上, 它拥有整个虚拟地址空间.  不同的进程可以同时使用一个虚拟地址, MMU通过页表将其映射到合适的物理地址.

### 两个物理地址空间

ARMv8 体系结构定义两个物理地址空间: secure address space 和 non-secure address space.

理论上, 安全和非安全的地址空间是相互独立的, 然而现实中大多数系统都将安全和非安全视为访问控制的属性. 正常(非安全)世界只能访问非安全的物理内存; 而安全世界可以访问这两个地址空间.

### ARMv8 MMU权限控制

程序请求某个地址时, MMU需要进行权限检查. 如果请求的地址是数据, 则检查读写权限; 如果请求的是地址, 则检查其可执行权限.

ARMv8 页表项的AP字段控制该不同异常等级下, 页面的读写权限.

[表格]

ARMv8 页表项的PNX字段和XN/UXN字段来设置CPU是否对这个页面有执行权限.

- 当系统有两套页表时, UXN是用来设置用户空间页面是否有可执行权限; PXN 用来设置特权空间的页面是否有可执行权限.

- 若系统只有一套页表, 则通过XN字段控制





## 页表的结构

### 地址宽度

48bit

### 页面粒度

页面粒度表示一次最小分配内存块的大小. AArch64支持三种页的大小, 4KB, 16KB, 64KB. 支持哪一种是由实现定义的。创建页表的代码能够读取系统寄存器`ID_AA64MMFR0_EL1`，以找出哪些是受支持的大小。Cortex-A53处理器支持所有三种尺寸，但有些处理器的早期版本并非如此，例如Cortex-A57，它不支持16K粒度。

### AArch64 页表项结构

#### 无效页表项

#### table

#### block

### 页表结构(4KB页面为例)

以4KB页面粒度, 虚拟地址宽度为 48位. 使用4级页表.

48位地址每层转换有9个地址位，即每层512个条目，最后12位选择4kB内的一个字节，直接来自原始地址



## 虚拟地址到物理地址的转换过程

当处理器为获取指令或数据访问发出一个64位的虚拟地址时，MMU硬件将虚拟地址转换为相应的物理地址。对于虚拟地址，前16位[63:47]必须全部为0或1，否则地址将触发故障。

### Non-secure and secure access

ARMv8-A架构定义了两种安全状态:安全的和非安全的。它还定义了两个物理地址空间:安全的和非安全的. 正常(非安全)世界只能访问非安全物理地址空间。安全世界可以访问两个物理地址空间。这也是通过转换表来控制的。

在非安全状态下，转换表中的NS位和NSTable位将被忽略。只能访问非安全内存。在安全状态下，NS位和NSTable位控制虚拟地址转换为安全物理地址还是非安全物理地址。

You can use SCR_EL3.SIF 来禁用安全世界访问非安全地址.

## 相关的寄存器

与地址转换相关的寄存器主要有以下几个:

1. 转换控制寄存器(TCR)
2. 系统控制寄存器(SCTLR)
3. 页表基地址寄存器(TTBR)

### TCR

IPS: 配置地址转换后输出物理地址的最大值

TxSz: 配置输入地址的最大值, 即虚拟地址的宽度

TG1: 配置TTBR1页表的页面粒度大小

SHx: 配置TTBRx相关内存的Cache共享属性

ORGNx: 

IRGNx:

### SCTLR

M: Disable/Enable MMU地址转换

C: Disable/Enable Data Cache

I: Disable/Enable Instruction Cache

### TTBR

存储页表的基地址



## AArch32 虚拟内存系统

ARMv8 AArch32 的虚拟内存系统向后兼容ARMv7, 与ARMv7的基本一致.
