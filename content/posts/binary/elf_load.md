---
title: "Elf 加载器的工作流程"
date: 2023-11-24T16:21:27+08:00
tags: ["Binary"]
categories: ["Binary"]
---

# 分析Elf文件

# 映射 Segments

# 对栈进行预处理

>```c
>int main(int argc, char **argv, char **envp) {...}
>```
>见到一个main函数的定义，你是否考虑过:
>1. main函数使用这些参数的作用分别是什么?
>2. Elf运行前，他们是如何被正确放置的?
>3. 我们又如何正确的访问?

内核中的Elf加载器还需要将辅助向量和其他信息(argc,argv,envp)一起放在栈上。
初始化后，进程的堆栈如下所示(64位架构下):
```
position            content                     size (bytes) + comment
  ------------------------------------------------------------------------
                    [ free used for process ]
  stack pointer ->  [ argc = number of args ]     8
                    [ argv[0] (pointer) ]         8   (program name)
                    [ argv[1] (pointer) ]         8
                    [ argv[..] (pointer) ]        8 * x
                    [ argv[n - 1] (pointer) ]     8
                    [ argv[n] (pointer) ]         8   (= NULL)

                    [ envp[0] (pointer) ]         8
                    [ envp[1] (pointer) ]         8
                    [ envp[..] (pointer) ]        8
                    [ envp[term] (pointer) ]      8   (= NULL)

                    [ auxv[0] (Elf64_auxv_t) ]    16
                    [ auxv[1] (Elf64_auxv_t) ]    16
                    [ auxv[..] (Elf64_auxv_t) ]   16
                    [ auxv[term] (Elf64_auxv_t) ] 16  (= AT_NULL vector)

                    [ padding ]                   0 - 16

                    [ argument ASCIIZ strings ]   >= 0
                    [ environment ASCIIZ str. ]   >= 0

  (0xbffffffc)      [ end marker ]                8   (= NULL)

  (0xc0000000)      < bottom of stack >           0   (virtual)
  ------------------------------------------------------------------------
```

在这之上，我们最常用的是argc和argv，所以还是需要介绍一下envp和auxv。

## Environment
环境变量的结构和argv相同，都是一些字符串指针的方式去访问。
只不过没有用于表示数量的"envc"，而是以NULL表示结尾。



## Auxiliary Vectors
Auxiliary Vectors 简称Auxv, OS内核在加载Elf时, 
可以将一些**键值对**类型的数值传递给用户态程序，
即进程启动时内核向用户态传递信息的一种方式。

Auxv的存储结构不是argv和envp那样的间接形式，而是顺序存一些`Elf64_auxv_t`结构体，在Linux上，结构体的定义在`/usr/include/elf.h`中。

```c
typedef struct
{
	uint64_t a_type;              /* Entry type */
	uint64_t a_val;
} Elf64_auxv_t;
```

可以看到就是一个类型和数值，直接就能存下，不支持字符串的形式。

我们可以在Linux上测试Auxv，在启动一个用户程序之前加上`LD_SHOW_AUXV=1`,
可以打印出所有的Auxv。

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

一般来说，普通的用户程序不需要获取内核的信息，但对于一些位于用户态和内核态之间的、
交互非常紧密的**用户态程序**而言则比较有用。比如说解释器、init进程、微内核OS中的root-service等等。

## 测试你的Elf Loader
在Elf中输出这些辅助信息，看是否能顺利读到。这里分别给出（1）只打印Auxv（2）打印所有
```c
#include <stdio.h>
#include <elf.h>

main(int argc, char* argv[], char* envp[])
{
  Elf32_auxv_t *auxv;
  while (*envp++ != NULL); /* from stack diagram above: *envp = NULL marks end of envp */

  /* auxv->a_type = AT_NULL marks the end of auxv */
  for (auxv = (Elf32_auxv_t *)envp; auxv->a_type != AT_NULL; auxv++) {
    if (auxv->a_type == AT_SYSINFO)
      printf("AT_SYSINFO is: 0x%x\n", auxv->a_un.a_val);
  }
}
```

```c
int main(int argc, char *argv[], char *envp[])
{
  int i;
  
  // Print all arguements
  printf("argc: %d\n", argc);
  for (i = 0; i < argc; i++) {
    printf("argv[%d]: %s\n", i, argv[i]);
  }

  // Print all enviroment variables
  for (i = 0; envp[i]; i++) {
    printf("env[%d]: %s\n", i, envp[i]);
  }

  // Print all auxv
  Elf64_auxv_t *auxv = (Elf64_auxv_t *)(envp+i+1);
  for (; auxv->a_type != AT_NULL; auxv++) {
    if (auxv->a_type == AT_PAGESZ) {
      printf("AT_PAGESZ: 0x%lx\n", auxv->a_un.a_val);
    } 
  }
}
```

# 设置User Context

# Reference 
- [About ELF Auxiliary Vectors](https://articles.manugarg.com/aboutelfauxiliaryvectors.html)

