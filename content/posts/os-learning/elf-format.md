---
title: "ELF 文件的链接与加载"
date: 2022-06-20T16:21:27+08:00
---

## `ELF` is a file format

Files in `ELF` format includes:

| Type               | description  | 实例   |
|--------------------|--------------|--------|
| Relocatable File   | 这些文件包含了代码和data, 可以被用来链接成可执行文件或共享目标文件. | `.o`, `.a` |
| Executable File    | 直接可执行的文件 | `/bin/ls` |
| Shared Object File | Including code and data. 链接器可将其与其他Relocatable File或Shared Object File结合, 生成新的目标文件. 动态链接器可将其与Executable File结合, 作为进程映像的一部分来运行. | `.so` |
| Core Dump File     | Restore critical infomation when process is terminated unexpectedly | `core dump` |

> :pushpin: `file` command in Linux can output the format of a file.

&nbsp;
## ELF 文件组成的结构
```
+---------------------------------+
|           ELF Header            |  包含描述整个ELF的基本信息, 如版本, 入口地址...
+---------------------------------+
|           .text                 |
+---------------------------------+
|           .data                 |
+---------------------------------+  紧接着是各个段
|           .bss                  |
+---------------------------------+
|           ...                   |  
|           other sections        |
+---------------------------------+
|                                 |  段表: 与段相关最重要的结构
|       Section Header table      |  描述了每个段的name, length, authority...
|                                 |
+---------------------------------+
|         String tables           |
|         Symbol tables           |
+---------------------------------+
```

|字段| 含义 |
|--|--|
| e_machine | 目标架构 |
| e_entry | 入口地址 |
| e_machine | 目标架构 |
| e_phnum | number of entries in the **program header table** |
| e_shnum | number of entries in the **section header table** |
| e_shoff | offset, in bytes, of the section header table |
| e_phoff | offset, in bytes, of the program header table |
| e_machine | 目标架构 |
| e_machine | 目标架构 |


### 段表
ELF文件中的各个段的基本属性就是保存在**段表**中，是分析ELF文件最重要的字段。存放了每个段的信息，例如，段名，段的长度，在文件中的偏移，读写权限以及其他属性。

编译器、链接器都是依靠段表来定位和访问各个段的属性的。

如何找到段表？ **`e_shoff`字段**

>使用`readelf -S <elfname>` 就能查看ELF文件的段表


### 程序头表

&nbsp;
## 分析ELF文件的工具
### 1. objdump

### 2. readelf

&nbsp;
## 为什么目标文件中代码和数据要分开放?
一方面, 程序被加载进内存后, 代码段和数据段分别被映射到**两个virtual memory region**.
通过MMU的支持, 可以将代码段的区域设置为只读, 防止恶意篡改.

另一方面, 当下CPU Cache多划分为*Instruction Cache*和*Data Cache*, 再配合互相独立的
地址区域能够提高**局部性原理**的效果.

最后, 代码段可以被多个进程共享(例如都调用同一外部函数), 节省内存空间.

> 针对嵌入式设备, 如果内存空间不够大, 只读的代码段可存放在ROM中

&nbsp;
## 关于静态库
一个静态库可以简单的看作是 a set of object file.  
这些 object file 可能包括: 输入输出相关的`printf.o`, `scanf.o`, 日期时间相关的`time.o`, `date.o`等.

:question: 为什么不直接提供这些*目标文件*呢? 

这些**零散的**文件若直接提供给使用者, 很大程度上造成文件传输, 管理等方面的不便.  
于是人们通常使用`ar`压缩程序将这些目标文件压缩到一起.

:question: 如何查看一个静态库是由哪些object file压缩到一起的?

Shell command`ar -t libc.a`  可以查看`libc.a`中包含的所有object files.

&nbsp;
## ELF文件加载-运行流程
> 废了半天劲编译生成的ELF文件, 想要最终跑起来则包含的instruction and data必须要在内存中. 

### 静态加载与动态加载
我们能想到的最简单的办法是: 把整个ELF的**所有指令和数据**在运行之前就全部load到内存中. 这就是*静态加载*.

更加高效的做法是: 充分利用*局部性原理*, 将指令和数据划分为**模块**, 只有当该模块被使用时, 才load进内存,
否则就在外存中老老实实呆着. 这就是*动态加载*.

### 动态加载的步骤
借助*虚拟内存*技术, 上面提到的**模块**的概念可以自然的被**页**(page)代替.
我们将所有的指令和数据按照page为单位划分.

1. 运行该ELF的线程被创建时, 其virtual space范围被划定, 但其页表是空的, 没有任何映射.

2. OS读取ELF Header, 建立virtual space与**ELF文件**的映射关系. 这个映射关系的表达方式是一个特殊的**数据结构**.
建立该映射关系的原因是: 当程序运行到某个地址发现该页表项是空的(例如 `call 0x1234`), 那么必然触发`page fault`. 
由OS负责到*特定的外存地址*将页面加载到physical memory中. 
```
OS要想知道缺失的内容在ELF文件的哪个位置, 就是利用该映射, 即某个数据结构
```

3. physical memory中有了所需的指令或数据后, 还需建立visual memory到physical memory的映射, 即在*页表项*中写入.
随着程序的运行, 会继续触发`page fault`, 从ELF中不断load page到physical memory, 建立缺页visual addr处的页表映射,
最终填补成一个完整的pagetable.

&nbsp;
## 段地址对齐技术
> 由前面动态加载的步骤可知, ELF文件中的代码和数据被按page划分. 并只有在用到时才被加载到内存, 
> 并建立`虚拟内存-物理内存`的映射.

假设一个ELF有三个段需要被`LOAD`, ELF段表如下:

| Segment | Length | offset |
|---------|--------|--------|
| SEG 0   | 127  B | 34  B  |
| SEG 1   | 9899 B | 164 B  |
| SEG 2   | 1988 B | 0   B  |

### :question: 这三个段在ELF文件中的布局如何?
根据前面ELF文件格式的介绍, 这三个段必然是挨着的(简单考虑, ELF中仅有这三个段).

### :question: 这三个段在物理内存中的布局?
发生`page fault`之后, OS会为页面分配合适的物理页面, 如利用`buddy system`等. 

可以保证*段内*的连续, 不能保证*段与段*是连续的.

> 未使用段对齐技术之前, `SEG0`的长度不足一页, 但是也给它分配一页的空间. 同理为`SEG1`分配两页, `SEG2`分配一页.
> 总共占用 `1+2+1=5`个物理页.

### :question: 这三个段在用户virtual addrspace下的布局如何?
todo

### :question: 何为段地址对齐技术?
上面说了, 在为这三个段分配物理内存时, 虽然他们的真实大小远小于5个页面, 但由于简单采用: `每个段的开头必须是page align`,
导致实际上产生了巨大的**内部碎片**.

段地址对齐实际上就是在为ELF文件中的段分配物理内存时, 不考虑其段的独立性, 强制按照`page`来划分. 划分的行为如下图所示.
结果就是仅需占用`3`个物理页面.
```
+---+---------------+
| P |     SEG0      |
| A +---------------+
| G |               |
| E |               |
+---+               |
| P |     SEG1      |
| A |               |
| G |               |
| E |               |
+---+               |
| P +---------------+
| A |               |
| G |     SEG2      |
| E |               |
+---+---------------+
```
> 目前, gcc(更准确是说是GUN ld)默认启用段对齐技术. 各个段的虚拟地址并不是`page align`.

> :four_leaf_clover: 物理页面到虚拟页面的映射阶段, 那些*同时包含*两个段的页面会被映射两次, 即一个物理页面对应两个
> 虚拟页.
>
> 原因是: 在一个页面的不同段可能**权限不同**, 所以不能使用同一映射.
