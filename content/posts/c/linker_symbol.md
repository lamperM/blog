---
title: "C/C++ 符号管理的区别"
date: 2023-12-14T12:21:27+08:00
tags: ["C"]
categories: ["C"]
---

## 背景

今天在写 C 代码时，遇到一个问题，我忘记 include 头文件而调用某个函数，
一般情况下在编译时会报警告 ⚠，然后也会链接成功，所以我这次就没管它因为只是暂时测试一下。
然而令我费解的是函数的执行结果异常，检查汇编后发现，我声明的函数返回 u64 类型，
而编译后的代码在返回前裁切成了 32 位，就是这里导致的错误！

这与我之前的理解不同，我以为要么就链接找不到符号，要不就成功链接，
为什么会有这种返回类型识别错误呢？

## 思考

我忽然想起，**会不会是因为编译器将返回值识别为了默认的 int 类型**，
进而，我的猜想是：

- 链接时能用符号名找到符号的地址，所以能成功调用
- 但因为没有参数和返回类型的说明（没有 include），所以导致类型出错

简单验证之后，确实我的猜想是正确的，我调用时多传入一个参数，
还是能够成功编译，汇编只是把参数寄存器赋值，内部用不用得到无法判定。
返回类型则统一认定为默认的`int`类型。

由此，我又产生了一个想法，既然C语言只使用符号名作为匹配的标准，
**那么必然不支持同名函数（参数、返回类型不同）。然而C++明确是支持的，**
那么C与C++的符号管理有什么不同吗？

## 进一步验证

我写了内容相同的C和C++两个文件来尝试解答问题：
```c
// Same code in demo.c/demo.cpp
void func(int a, int b)
{
  func(b, a);
}
```

对他们进行编译，查看大小仅相差8字节，猜测是符号的管理有所不同。
```sh
$ ls -al
total 8
drwxr-xr-x 1 loo loo  512 Dec 14 14:09 .
drwxr-xr-x 1 loo loo  512 Dec 14 12:03 ..
-rw-r--r-- 1 loo loo   42 Dec 14 14:09 demo.c
-rw-r--r-- 1 loo loo 1480 Dec 14 12:04 demo.c.o
-rw-r--r-- 1 loo loo   42 Dec 14 12:04 demo.cpp
-rw-r--r-- 1 loo loo 1488 Dec 14 12:04 demo.cpp.o
```

进一步查看section的差别，忽略地址的差异，其实差异就在.shstrtab的大小，
C++编译处的目标文件多了几个字节，而`.shstrtab`我们给的样例中也就是存符号`func`。
```diff
$ diff <(readelf -S demo.c.o) <(readelf -S demo.cpp.o)
1c1
< There are 13 section headers, starting at offset 0x288:
---
> There are 13 section headers, starting at offset 0x290:
10c10
<   [ 2] .rela.text        RELA             0000000000000000  000001e8
---
>   [ 2] .rela.text        RELA             0000000000000000  000001f0
24c24
<   [ 9] .rela.eh_frame    RELA             0000000000000000  00000200
---
>   [ 9] .rela.eh_frame    RELA             0000000000000000  00000208
29,30c29,30
<        000000000000000d  0000000000000000           0     0     1
<   [12] .shstrtab         STRTAB           0000000000000000  00000218
---
>        0000000000000014  0000000000000000           0     0     1
>   [12] .shstrtab         STRTAB           0000000000000000  00000220
```

继续查看.shstrtab的内容，就能得到最终的结果，**确实C语言只存储符号名，
而C++则既有符号名，也有参数和返回值类型。**
```sh
$ readelf -p .strtab demo.c.o

String dump of section '.strtab':
  [     1]  demo.c
  [     8]  func

$ readelf -p .strtab demo.cpp.o

String dump of section '.strtab':
  [     1]  demo.cpp
  [     a]  _Z4funcii
```

>通过`nm`命令也一样能够查看
>```sh
>$ nm demo.cpp.o
>0000000000000000 T _Z4funcii
>```

## 总结
确实C语言只存储符号名，而C++则既有符号名，也有参数和返回值类型。
这是C++能够支持重载的本质。 有时我们在写C++时，如果找不到函数会报一个乱码的函数名，
其实也就是这种经过压缩存储的符号。
