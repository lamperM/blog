---
title: "Linux 进程间通信: 管道"
tags: ["Operating System"]
date: 2023-05-11T20:51:49+08:00
---

管道属于实现进程间通信的一种方式，正如其名，一个进程在一头读，另一个进程在一头写。

**管道被看做是打开的文件**，但在已安装的文件系统中没有相应的实体，即并不是一个
真正的文件。

## 管道的创建和使用

可以使用`pipe()`系统调用来创建一个管道(后面会介绍另一个方式)，其返回一对文件
描述符，一个用来写一个用来读。必须返回两个描述符的原因是： POSIX 只定义了半双工
的管道，所以读写需要两个端口。

> POSIX 另外要求使用一个描述符前需要关闭另一个描述符。 但 Linux 中则可以不关闭，
> 可以实现全双工，但为了可移植性， 一般还是将另一个先关闭。

用`ls | more`组合命令来解释如何使用`pipe()`实现通信:

1. shell 调用`pipe()`, 返回 fd3(对应读通道),fd4(对应写通道)
2. 两次调用 `fork()` 创建两个子进程，由于属于不同的地址空间，
   所以操作自己的文件描述符不会影响其他进程，但都指向同一个管道
3. 父进程调用`close()`关闭这两个文件描述符

第一个子进程执行`ls`程序，其操作如下，

1. 调用`dup2(fd4, stdout)`, 执行文件描述符的拷贝，从此`stdout`
   就代表管道的写通道
2. 由于`stdout`代表写通道，所以可将 fd3 和 fd4 均关闭
3. `exec()`执行`ls`程序，默认情况下，其输出结果到 `stdout`，
   当下即管道的写通道，即向管道中写了数据

第二个子进程执行`more`程序，其操作如下：

1. 调用`dup2(fd3, stdin)`, 从此`stdin`代表管道的读通道
2. 同样可以将 fd3 和 fd4 关闭
3. `exec()`执行`more`程序，由于现在`stdin`就是管道的读通道,
   上面的子进程向管道中写了数据，所以`stdin`现在有数据，`more`
   可以正常输出

## `popen()`: 更简单的 API

当管道的使用是单向的，即某个进程仅仅想知道另一个进程的执行输出，或者
某个进程想把数据灌入到另一个进程的输入。

此时 Linux C 库中的`popen()`和`pclose()`简化使用`pipe()`中
调用`dup2()`, `close()`这些繁琐的步骤。

`popen()`接受两个参数: **可执行文件的路径和使用方式 type**, 返回
一个指向 `FILE` 的指针。

`popen()`做的事包含:

1. `pipe()`创建一个管道
2. 创建一个新进程，执行:
   1. 如果使用方式是 r, 绑定管道的写通道到`stdout`, 否则绑定读通道
      到`stdin`.
   2. 关闭`pipe()`返回的两个描述符
   3. `exec()`执行指定的可执行文件
3. 回到父进程，如果使用方式是 r, ·关闭管道的写通道。否则关闭读通道
4. 返回管道剩下的文件描述符地址

这样父子进程就能单向通信了，如何使用方式是 r, 父进程可用`popen()`
返回的 FILE 来读取可执行文件的输出；相反，父进程可向 FILE 中写数据
到可执行文件的输入。

因为返回的是 FILE 指针，所以通常配合`fprintf()`, `fscanf()`, `fgets()`
这类函数使用。

`pclose()`等待`popen()`创建的子进程结束。

## `popen()`样例

父进程读

```c
#include <stdio.h>
#include <unistd.h>

int main(void)
{
  FILE *fp = NULL;
  char s1[32], s2[32];

  fp = popen("ls", "r");

  // read the first two files and print their name
  fscanf(fp, "%s %s ", s1, s2);
  printf("s1 = %s, s2 = %s\n", s1, s2);

  // close the read side of pipe
  pclose(fp);
  return 0;
}
```

父进程写

```c
#include <stdio.h>
#include <unistd.h>

int main(void)
{
  FILE *fp = NULL;
  char s1[32], s2[32];

  fp = popen("wc -l", "w");
  fprintf(fp, "one\ntwo\nthree\n");
  pclose(fp);

  return 0;
}
```
