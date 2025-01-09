---
title: "setvbuf: 更改文件缓冲区的行为"
tags: ["C"]
categories: ["C"]
date: 2025-01-09T20:51:49+08:00
---

```c
/**
 * setbuf()和setvbuf()函数的实际意义在于：用户打开一个文件后，可以建立自己的文件缓冲区，
 * 而不必使用fopen()函数打开文件时设定的默认缓冲区。这样就可以让用户自己来控制缓冲区，
 * 包括改变缓冲区大小、定时刷新缓冲区、改变缓冲区类型、删除流中默认的缓冲区、为不带缓冲区的流开辟缓冲区等。
 */
#include <stdio.h>
#include <unistd.h>

int main(void) {

  while (1) {
#if 0
    // 1. 默认行为
    // 检验stdout默认是基于行的缓冲，即输出道stdout的字符都会被缓冲，
    // 直到碰到一个换行符，或者buf满，才会被输出到屏幕上
    printf("Hello stdout");
    // stderr 模式是不缓冲的
    perror("Hello stderr");
#endif

#if 0
    // 2. stdout添加换行后立马输出
    printf("Hello stdout\n");
#endif

#if 1
    // 3. 更改stdout的默认缓冲行为，将line buffered修改为unbuffered
    setvbuf(stdout, NULL, _IONBF, 0); // 其实在外面设置一次就行
    printf("Hello stdout");
#endif

    sleep(1);
  }

  return 0;
}
```
