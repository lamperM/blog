---
title: "面试总结：特斯拉实习"
tags: ["work"]
categories: ["Job"]
draft: true
date: 2023-06-10T08:51:49+08:00
---

上午结束了特斯拉*嵌入式实习生-Linux platform*的二面，特斯拉实习生
一共有三轮面试，一轮和二轮都是技术面，三面是主管面。
目前我完成了所有的技术面，且不论结果如何，整个面试过程对我而言还是收获颇丰的，
故以此文整理下自己的欠缺的技术知识，希望下次能够表现的更好。

# 二面

首先，此轮面试的面试官显然比上一轮更有礼貌一些，准时与会+介绍自己，不过有一点
是我开了摄像头他没开，也没有进行说明吧。 不过这些都是小事，我们这次主要谈论
技术的内容。

我在此回顾几个没有回答好的问题，供以后做参考。

## ELF 文件的加载流程

> 原回答
>
> 1. 拿到 ELF 存储的地址后，先将头部读出来，长度是固定的。头部有校验字段，
>    maigic number, 然后是确认 ELF 编译的架构，位数是否正确。
> 2. 确认格式正确后，读取程序头表，其中保存了各个需要加载的段的偏移，根据
>    base+段偏移能够得到该段的位置，然后根据段属性的不同，选择映射到不同
>    的区域和属性，例如 text 段映射为 RE, 代码段映射为 RW，清空 BSS 等
> 3. 说明自己没接触过动态库文件，所以对动态加载不是很熟悉

点评如下: 对段表、程序头表，这些概念的区分还不是很熟，不清楚什么时候用
section table， 什么时候用 program header table. 以前都看过，只是
时间长了不用就忘记了，这一部分需要好好的做下笔记。

## 全局变量存在哪？谁负责初始化的

> 原回答
>
> 不知道。

全局保量存在的位置：
- 未初始化的全局变量 ==> bss段
- const修饰的全局变量 ==> rodata段
- 其他已初始化的全局变量 ==> data段

对于已初始化的全局变量的访问，编译时，编译器将值存入data段，访问的指令是通过
相对寻址来做，例如相对于data段开头。对于静态链接来说，编译完成后访问指令的
基地址和偏移都是空，当链接时修改指令，即重定位的过程。

## aligned_alloc()设计

这是面试最后的程序设计题，我也是没有做好，后面好歹在面试官的无数次提示中，
写出了一个解，题目很棒，只是自己实习不够，怪不得其他。

### 原问题

对`malloc()`和`free()`进行封装，设计一个返回满足任意字节对齐要求的
`align_alloc()`和`align_free()`函数。

```c
void *align_alloc(size_t size, size_t align);
void align_free(void *addr);

int main(void)
{
  size_t alignment[6] = {5, 8, 32, 64, 128, 12};
  int i;
  char *p;

  for (i = 0; i < 6; i++) {
    p = align_alloc(10, alignment[i]);
    if ((unsigned long)p % alignment[i]) {
      printf("FAILED!, alignment: %ld, addr: %p\n",
          alignment[i], p);
      exit(1);
    }
    memset(p, 0, 10);
    align_free(p);
  }
  printf("PASS\n");
  return 0;
}
```

### 思路及解决方案

思路也写在代码里了，就是思路比较难想，代码倒是不难写。

PS: 丢人的是，我在一开始居然还提了: `malloc()`返回的地址能够保证是参数`size`对齐的，
真是令人耻笑啊！！

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
void *align_alloc(size_t size, size_t align);
void align_free(void *addr);
// 如何保证对齐到align?
// ==> 申请更多空间, 返回对齐的地址
//
// 不能保证malloc返回的地址是对齐到哪的，怎么确定多申请多大呢?
// ==> 多申请align的空间，就能保证这align的地址范围里，总有
//     满足对齐条件的地址
//
// 如何计算返回的地址? 比如malloc(10)=12, align=5, 此时我们会
// 申请的空间为malloc(10+5), 即(12-27)都是可用的，我们需要
// 经过计算后返回地址 15
// ==> 计算的规则是(12+5)-(12+5)%5
//
// aligned_free()只接受addr一个参数，如何释放malloc()申请的空间?
// ==> 传入的addr是经过align之后的，确实需要一定的方法才能
//     求得malloc()返回的地址，这通过计算是无法得到的，因为
//     不知道这个地址的align是多少。所以可以想到在p_ret之前
//     再多借用一个地址的长度来存放malloc()返回的原地址。

void *align_alloc(size_t size, size_t align)
{
  void *p_malloc;
  void *p_ret;
  size_t aligned_size;
  size_t ptr_size;

  ptr_size = sizeof(char *);
  aligned_size = size + align + ptr_size;

  p_malloc = malloc(aligned_size);
  p_ret    = (p_malloc+ptr_size+align) -
              ((unsigned long)p_malloc+ptr_size+align) % align;

  *((unsigned long *)p_ret-1) = (unsigned long)p_malloc;

  // debug
  printf("[ALLOC] p_malloc: %p, p_ret: %p, align: %ld\n",
      p_malloc, p_ret, align);

  return p_ret;
}
void align_free(void *addr)
{
  void *p_malloc;

  p_malloc = (void *)(*((unsigned long *)addr-1));

  printf("[ FREE] p_malloc: %p\n", p_malloc);
  free(p_malloc);
}


int main(void)
{
  size_t alignment[6] = {5, 8, 32, 64, 128, 12};
  int i;
  char *p;

  for (i = 0; i < 6; i++) {
    p = align_alloc(10, alignment[i]);
    if ((unsigned long)p % alignment[i]) {
      printf("FAILED!, alignment: %ld, addr: %p\n",
          alignment[i], p);
      exit(1);
    }
    memset(p, 0, 10);
    align_free(p);
  }
  printf("PASS\n");
  return 0;
}
```
