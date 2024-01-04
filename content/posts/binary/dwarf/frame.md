---
title: "Dwarf: Stack Unwinding"
tags: ["Dwarf"]
categories: ["Dwarf"]
date: 2023-12-16T15:51:49+08:00
---

## 前言

栈回溯是调试代码常用的功能之一，Gdb 中对应的命令是`bt`,`info frame`等。
这篇文件将介绍利用 Dwarf 生成的调试信息实现栈回溯的方法。

## 原理

Dwarf v2 开始提供一种叫做 Call Frame Information（简称 CFI）的信息，
它存储在`.debug_frame`中，调试器可以通过解析这个 Section 完成栈回溯。
`.debug_frame`里的内容可以看做是一张二维表格，一列是 pc，
另一列是**对于此 Pc 如何查找上一个 Frame**。

### Demo

例如，对于以下的 C 代码和对应的汇编（通过`object -S`生成)，
汇编代码有一点长，但没关系我们不需要关注每一条汇编指令。
这段代码共有两个函数，`main()`和`fibonacci()`，
由 main 函数调用 fibonacci 来计算第 10 个 bibonacci 数。
目前暂时不需要看汇编。

选择 fibonacci()作为例子的原因是**模拟一个非叶子函数**，
因为 Arm64 下对叶子函数可能不会生成正确的 CFI 信息，
因为这种情况不常见，所以我们先讨论普通的情况。
另外，我知道这个计算 fibonacci 数的算法不是最优的，
但是我们毕竟不是算法优化的主题，所以能够说明问题即可。

```c
int fiboncci(int n)
{
  if (n <= 2)
    return 1;
  else
    return fiboncci(n-1) + fiboncci(n-2);
}

int main(void)
{
  int result;

  result = fiboncci(10);
  return 0;
}
```

```assembly
int fiboncci(int n)
{
  400594: a9bd7bfd  stp x29, x30, [sp, #-48]!
  400598: 910003fd  mov x29, sp
  40059c: f9000bf3  str x19, [sp, #16]
  4005a0: b9002fe0  str w0, [sp, #44]
  if (n <= 2)
  4005a4: b9402fe0  ldr w0, [sp, #44]
  4005a8: 7100081f  cmp w0, #0x2
  4005ac: 5400006c  b.gt  4005b8 <fiboncci+0x24>
    return 1;
  4005b0: 52800020  mov w0, #0x1                    // #1
  4005b4: 14000009  b 4005d8 <fiboncci+0x44>
  else
    return fiboncci(n-1) + fiboncci(n-2);
  4005b8: b9402fe0  ldr w0, [sp, #44]
  4005bc: 51000400  sub w0, w0, #0x1
  4005c0: 97fffff5  bl  400594 <fiboncci>
  4005c4: 2a0003f3  mov w19, w0
  4005c8: b9402fe0  ldr w0, [sp, #44]
  4005cc: 51000800  sub w0, w0, #0x2
  4005d0: 97fffff1  bl  400594 <fiboncci>
  4005d4: 0b000260  add w0, w19, w0
}
  4005d8: f9400bf3  ldr x19, [sp, #16]
  4005dc: a8c37bfd  ldp x29, x30, [sp], #48
  4005e0: d65f03c0  ret

00000000004005e4 <main>:

int main(void)
{
  4005e4: a9be7bfd  stp x29, x30, [sp, #-32]!
  4005e8: 910003fd  mov x29, sp
  int result;

  result = fiboncci(10);
  4005ec: 52800140  mov w0, #0xa                    // #10
  4005f0: 97ffffe9  bl  400594 <fiboncci>
  4005f4: b9001fe0  str w0, [sp, #28]
  return 0;
  4005f8: 52800000  mov w0, #0x0                    // #0
}
  4005fc: a8c27bfd  ldp x29, x30, [sp], #32
  400600: d65f03c0  ret
  400604: d503201f  nop
  400608: d503201f  nop
  40060c: d503201f  nop
```

因为在编译时添加了`-g`选项，所以 Gcc 默认会生成`.debug_frame`，
让我们来看看里面的内容是什么。

```sh
aarch64-none-linux-gnu-objdump --dwarf=frames-interp frame
```

```
00000088 0000000000000020 0000008c FDE cie=00000000 pc=0000000000400594..00000000004005e4
   LOC           CFA      x19   x29   ra
0000000000400594 sp+0     u     u     u
0000000000400598 sp+48    u     c-48  c-40
00000000004005a0 sp+48    c-32  c-48  c-40
00000000004005e0 sp+0     u     u     u

000000ac 0000000000000020 000000b0 FDE cie=00000000 pc=00000000004005e4..0000000000400604
   LOC           CFA      x29   ra
00000000004005e4 sp+0     u     u
00000000004005e8 sp+32    c-32  c-24
0000000000400600 sp+0     u     u
```

这是经过解释之后的`.debug_frame`，在实际存储的时候可能通过压缩。
解释之后就明显展示出二维表格的样貌。上述代码里共有两段表，
第一段对应我们在`fibonacci()`中如何查找上一个 Frame，
第二段则是`main()`函数中的计算规则。

暂时到这里其实就 Ok 了，你只需要知道它起码看起来像是一个二维表格的结构，
我们下面再说它到底是怎么帮助实现栈回溯的。

## 解剖 `.debug_frmae`

### CIE & FDE

上面不是说表格中存的是栈回溯的规则吗，这张大表可以按照函数的界限来划分，
FDE 就是对应某个函数的计算规则。一些函数共同的部分提取成一个 CIE。

### 栈回溯的过程

还是以上面的代码为例，假设我们位于 Fibonacci()中的`4005a4`地址上，
此时我们要进行栈回溯，找到 main()中的调用点。

首先，在第一个 FDE 里找到当前地址`4005a4`的计算规则，
因为连续的一段地址可能计算规则不变，可以将他们存成一条，节约空间。

```
00000088 0000000000000020 0000008c FDE cie=00000000 pc=0000000000400594..00000000004005e4
   LOC           CFA      x19   x29   ra
0000000000400594 sp+0     u     u     u
0000000000400598 sp+48    u     c-48  c-40
00000000004005a0 sp+48    c-32  c-48  c-40
00000000004005e0 sp+0     u     u     u
```

第一列 LOC 是每个规则相同的连续地址块的起始地址，由此得到`4005a4`需要参考第三行规则。

再看到第二列，**CFA 的含义是 caller 的 sp**， 第三列**ra 的含义是返回地址(Returen address)**， 表格中的值 u 代表不变，c 代表 CFA。

所以说，由表格中的数据可知，`caller'sp=sp+48`，而返回地址就存储在`cfa-40`的地址上，
这是一个栈的地址，由于函数调用不会破坏之前栈的内容，所以可以放心取出 ra，也就是返回地址。
这就找到了 main 函数中的调用点，如果 main 之上还有 caller 的话，可以继续查 main 的 FDE 来寻找。

### 原理

其实，这种方法就是利用了每次进入函数时时会在栈上保存返回地址 x30，
基于 Fp 的栈回溯方法也是这样做，Fp 是通过记录每个栈的栈底，
而每次保存 x30 的位置相对于栈底都是固定的，所以知道栈底也就能取出 x30。

`.debug_frame`可以看作将每次函数调用的栈底存在本地成为一张表，
这就省了一个通用寄存器。

### 关于 Caller-saved Regs

## 注意

1. 上面说到，用`.debug_frame`实现栈回溯时，起点不能是叶子函数。
   叶子函数不会调用任何函数，所以 Arm64 会优化而不保存 Lr 在栈上（反正你也不调用其他函数，x30 不会被破坏）。
   我们的方法就不奏效，可以验证叶子函数的 FDE 是空的。
