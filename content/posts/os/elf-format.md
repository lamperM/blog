---
title: "ELF 文件的链接与加载"
date: 2022-06-20T16:21:27+08:00
tags: [Operating System]
---

# ELF不只是可执行文件

Files in `ELF` format includes:

| Type               | description  | 实例   |
|--------------------|--------------|--------|
| Relocatable File   | 这些文件包含了代码和data, 可以被用来链接成可执行文件或共享目标文件. | `.o`, `.a` |
| Executable File    | 直接可执行的文件 | `/bin/ls` |
| Shared Object File | Including code and data. 链接器可将其与其他Relocatable File或Shared Object File结合, 生成新的目标文件. 动态链接器可将其与Executable File结合, 作为进程映像的一部分来运行. | `.so` |
| Core Dump File     | Restore critical infomation when process is terminated unexpectedly | `core dump` |

> :pushpin: `file` command in Linux can output the format of a file.


# ELF 文件组成的结构
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
|                                 |  与section相关最重要的结构
|       Section Header table      |  描述了每个section的名称,长度,权限...
|                                 |
+---------------------------------+
|                                 |  与segment相关最重要的结构
|       Program Header table      |  描述了每个segment的位置、属性(RWX)、size...
|                                 |
+---------------------------------+
|         String tables           |
|         Symbol tables           |
+---------------------------------+
```

## ELF Header

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


## Program Header Table
每个表项对应一个segment，表明其位置、属性(LOAD?动态链接用?)、memory size和file size，权限(RWX)等。

>Memory Size >= Segment Size, 因为有BSS段存在。

## Section Header Table
相应地，每个表项指定一个section的信息，包括名字、大小、地址等。

# 为什么目标文件中代码和数据要分开放?
一方面, 程序被加载进内存后, 代码段和数据段分别被映射到**两个virtual memory region**.
通过MMU的支持, 可以将代码段的区域设置为只读, 防止恶意篡改.

另一方面, 当下CPU Cache多划分为*Instruction Cache*和*Data Cache*, 再配合互相独立的
地址区域能够提高**局部性原理**的效果.

最后, 代码段可以被多个进程共享(例如都调用同一外部函数), 节省内存空间.

> 针对嵌入式设备, 如果内存空间不够大, 只读的代码段可存放在ROM中

# 为什么要区分section和segment
segment是加载关心的, section是链接过程关心的。

- segment中包含了多个section, 这些section地址相连、属性类似, 加载器只按照segment去加载。
- 而链接时, 链接器会对每个section进行重定位, 同时也需要 `.rel*` section来完成重定位。
- 调试信息也是按照section进行存储的，调试器依赖他们得到符号信息。

# 加载ELF时要将AUX信息放在栈底， AUX是什么

AUX全称Auxiliary Information，即辅助信息。

输出一个ELF的AUX:
```sh
~ $ LD_SHOW_AUXV=1 /bin/ls
AT_SYSINFO_EHDR:      0x7ffc3b3de000
AT_HWCAP:             1f8bfbff
AT_PAGESZ:            4096
AT_CLKTCK:            100
AT_PHDR:              0x55a318711040
AT_PHENT:             56
AT_PHNUM:             13
AT_BASE:              0x7f0f5ac91000
AT_FLAGS:             0x0
AT_ENTRY:             0x55a3187177d0
AT_UID:               1000
AT_EUID:              1000
AT_GID:               1000
AT_EGID:              1000
AT_SECURE:            0
AT_RANDOM:            0x7ffc3b272ab9
AT_HWCAP2:            0x2
AT_EXECFN:            /bin/ls
AT_PLATFORM:          x86_64

```

AUX是内核加载用户程序时可以将ELF的一些信息传到用户态，该应用可以读取。
通常用于加载某个外部解释器，一般程序员用不到。


# ELF 链接

链接即多个.o生成最终可执行程序的过程，编译时对于外部的函数，编译器无法确定实际跳转的地址，
只能先写0，链接过程会对这个值进行修改。

主要包含两个过程：（1）地址空间分配（2）重定位
- 地址空间分配：这么多.o，这么多section，他们结合为可执行文件后地址怎么规划呢？是不是有的
  section 可以合并，比如多个代码段。
- 重定位：对于外部调用的函数，这些实际的值也需要更正，或者采用别的方法来间接寻址（动态链接）

## 静态链接

重定位

合并后的每个section都有一个重定位表，`.rel.xx`， 里面的内容大概是:
```
RELOCATION RECORDS FOR [.text]:
OFFSET TYPE VALUE
0000001c R_386_32 shared
00000027 R_386_PC32 swap
```
对于每个需要重定位的指令，都会在这里表里对应到，所以链接时需要遍历它，填充上真正的地址。


## 动态链接

静态链接的做法就决定了，程序A和B不能共享同一份库，浪费内存。库编进了可执行文件中，所以生成的可执行文件就很大，
另外这样如果要修改库的话就需要对所有依赖的A和B都重新编译。

动态链接是目的是解决上面的问题，也就是说，库不编到ELF里，ELF在运行的时候能找到它就行。
这样一个程序编译链接后其实不能确定库的地址

# ELF 加载
> 废了半天劲编译生成的ELF文件, 想要最终跑起来则包含的instruction and data必须要在内存中. 

我们能想到的最简单的办法是: 把整个ELF的**所有指令和数据**在运行之前就全部load到内存中. 这就是*静态加载*.

更加高效的做法是: 充分利用*局部性原理*, 将指令和数据划分为**模块**, 只有当该模块被使用时, 才load进内存,
否则就在外存中老老实实呆着. 这就是*动态加载*.
## 静态加载
1. 读取ELF header, 校验magic number和架构是否正确
2. 根据ELF header中指定的 program header table地址去读 segments
3. 加载segments 中属性为LOAD的segment, 先要分配对应的虚拟空间, 根据ELF的LMA
4. 加载程序为ELF分配栈空间，并填充argc, argv，env等。
    >参数、环境变量怎么填充需要参考体系结构的ABI手册
5. 将PC设置为ELF header中entry point, 返回到用户态开始执行

## 动态加载

没什么好说的，就是第一次加载时只分配页表，建立映射，并不实际分配物理空间。





# 关于静态库
一个静态库可以简单的看作是 a set of object file.  
这些 object file 可能包括: 输入输出相关的`printf.o`, `scanf.o`, 日期时间相关的`time.o`, `date.o`等.

:question: 为什么不直接提供这些*目标文件*呢? 

这些**零散的**文件若直接提供给使用者, 很大程度上造成文件传输, 管理等方面的不便.  
于是人们通常使用`ar`压缩程序将这些目标文件压缩到一起.

:question: 如何查看一个静态库是由哪些object file压缩到一起的?

Shell command`ar -t libc.a`  可以查看`libc.a`中包含的所有object files.


# 段地址对齐技术
> 由前面动态加载的步骤可知, ELF文件中的代码和数据被按page划分. 并只有在用到时才被加载到内存, 
> 并建立`虚拟内存-物理内存`的映射.

假设一个ELF有三个段需要被`LOAD`, ELF段表如下:

| Segment | Length | offset |
|---------|--------|--------|
| SEG 0   | 127  B | 34  B  |
| SEG 1   | 9899 B | 164 B  |
| SEG 2   | 1988 B | 0   B  |

## :question: 这三个段在ELF文件中的布局如何?
根据前面ELF文件格式的介绍, 这三个段必然是挨着的(简单考虑, ELF中仅有这三个段).

## :question: 这三个段在物理内存中的布局?
发生`page fault`之后, OS会为页面分配合适的物理页面, 如利用`buddy system`等. 

可以保证*段内*的连续, 不能保证*段与段*是连续的.

> 未使用段对齐技术之前, `SEG0`的长度不足一页, 但是也给它分配一页的空间. 同理为`SEG1`分配两页, `SEG2`分配一页.
> 总共占用 `1+2+1=5`个物理页.

## :question: 这三个段在用户virtual addrspace下的布局如何?
todo

## :question: 何为段地址对齐技术?
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





# 分析ELF文件的命令

```sh
# 输出 section header table
readelf -S xxx.elf  
# 输出 program header table
readelf -l xxx.elf
# 输出 ELF header
readelf -h xxx.elf  
# 输出 elf header，section header table，program header table(常用）
readelf -e xxx.elf 
```