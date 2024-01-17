
## 好的资料

- [介绍PMU分析和采样的最基本原理](https://easyperf.net/blog/2018/06/01/PMU-counters-and-profiling-basics)
- [关于PMC的论文不知道有没有用](https://dl.acm.org/doi/pdf/10.1145/3629525)
- 

## 调测系统与OS Kernel
KMonitor属于内嵌于Kernel中的一段代码，被执行当且仅当：
1. 系统发生异常时进行得到运行。
因为OS的配置是无论用户态还是内核态发生异常都是在Kernel中处理，
所以KMonitor理论上能够管理用户态和内核态的调测。
2. Kernel初始化时，在必要组件初始化完毕后会先进入KMonitor

无论是哪种情况进入KMonitor之后，会等待用户输入调测命令，
如果系统是因为调试被停下，而不是发生异常情况，这时KMonitor处理完毕后可以选择继续执行Kernel，
返回Kernel之前的代码开始执行，直到下一次发生异常再次进入KMonitor。




## 调测系统的架构设计

KMonitor各个组件根据功能的不同可划分为：
1. 主模块
2. 用户交互终端
3. 数据存储
4. 符号处理

[假装有一张各个组件的图]

### 主模块
主模块是KMonitor的核心，它主要负责与OS Kernel进行交互和控制权的转换，
并在合适的时机调用其他子模块完成相应功能。

### 用户交互终端
作为一个功能强大、灵活度高的调测系统，KMonitor必然支持动态选择调试方法，
即为用户提供一个可以键入调测命令的终端，主模块接受解析后的命令及参数完成相应的调测动作。

### 数据存储
对于Debug命令来说，数据还可以直接通过函数参数返回，
比如打印某个内存地址的值，或者查看某个寄存器的值等。
但是对于Trace功能来说，显然产生的数据量是非常大的，并不能直接通过函数来返回，
需要做一个中间的存储工具。

数据存储模块的功能就是将KMonitor产生的数据（主要是Trace）存储下来，
当用户需要查看的时候再将数据返回，这样更加灵活。

### 符号处理
作为一个现代的调测工具，使用符号去下达命令、格式化输出数据是必不可少的功能项。
如果仅仅是一串连续的地址或者数据，你必须挨个查阅可执行文件的反汇编结果才能了解它的含义，
而有了符号以后，很大程度上简化了调试的流程，一切数据都可以输出为你一眼就能看懂的形式。

## 调测系统的能力范围






## 控制流和数据流

# Stack Unwinding
Reference:
- https://gitee.com/aosp-riscv/working-group/blob/master/articles/20220719-stackuw-fp.md


## Stack Unwinding是什么

Stack Unwinding（栈展开） 是指在程序执行过程中，当发生异常或错误时，
系统会沿着调用栈（call stack）逐层回退，
展示此次发生异常的路径，帮助排查问题。

Gdb提供了栈回溯的命令: backtrace

当操作系统发生错误或者异常时，例如栈溢出错误，
查看函数调用关系图就能很容易定位到问题发生的代码路径。
甚至，高级Stack Unwinding的实现方法支持查看每次调用携带的参数，
更加有利于排查问题。

## 实现Stack Unwinding的几种方式
### 基于Fp寄存器

AArch64体系结构中, 帧指针(Frame Pointer)是一个通用寄存器,别称x29。
引入Fp的目的就是辅助Sp实现栈回溯，当仅存在Sp时，调用栈的结构是：

TODO

这个过程无法实现Stack unwind的原因是每个函数的栈大小都不相等，
是动态的，就无法获取调用者的栈地址。

而引入Fp之后它的作用就是在函数执行过程中永远指向函数栈的栈底，
这样在函数执行的任意时刻都能找到当前栈的开始，也就是调用者栈的结束位置。

与返回地址一样，为了防止在多层调用的过程中Fp的值被破坏，
在函数的prologue和epilogure部分需要添加相应的处理，
将当前栈的Lr和Fp一起保存到下一个函数栈的栈底, 并在函数返回之前恢复，
才能保证循环回溯正确实现。总的来说，引入Fp之后栈帧的结构如下图所示：

TODO

基于Fp的Stack Unwinding有以下的缺点：
- 首先，对性能会产生影响。在函数的prologue和epilogure阶段要多保存和恢复一个寄存器。
- 其次，减少了一个通用寄存器。如果x29不用于Fp，那么可以作为普通通用寄存器使用。
- 然后，Stack Unwinding的准确性一般。在函数的prologue和epilogue阶段，
  如果Fp的串联关系还没有建立完整，就会出现解析失败的情况。
- 只能恢复每个栈帧的Sp和Fp，并不能恢复其他的寄存器信息。




### 基于Dwarf

Dwarf在编译过程中会生成.debug_frame section来支持实现Stack Unwinding。

实际上.debug_frame是一张巨大的表格，用于查询对于每个指令地址该如何得到其返回地址(ra)和Callee-saved寄存器的值。

例如，，，TODO



### 基于LBR

LBR(Last Branch Record)是Intel x86_64处理器提供的基于硬件实现的分支调用跟踪组件，
有助于实现调试和性能分析。

LBR的实现包括以下组件：
1. LBR Stack: 存储最近的分支跳转情况，其大小是可配置的，通常可以存储数十条到几百条分支跳转。
2. LBR Control Register: LBR的控制寄存器，可以对LBR特性进行配置，包括LBR Stack的容量等。
3. LBR Interrupt: 可以对一些LBR事件添加中断，调试器通过中断可以了解分支跳转发生。

LBR特性有以下特点：
1. 准确性高；能够准确记录每一次跳转
2. 灵活；可以通过参数配置来适应不同场景，例如LBR Stack大小、模式等。
3. 程序运行时动态获取数据，相比静态分析来说容易进行性能优化。

但是使用LBR实现Stack Unwinding也有一定的局限性：
1. 限定Intel支持64位的处理器，并且必须支持LBR特性
2. LBR Stack的容量有限，不能处理调用层次较深的情况





# Function Trace

Reference：
- https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=f48e8682c14355df15567be2c95307485edb1587#page=47

## Function Trace是什么


## 如何实现Function Trace
Dynamic Function Trace的作用原理是Gcc编译器提供的"-pg"编译选项，
"-pg"的存在就是为了辅助实现profiling和tracing，
添加此编译选项后，会在所有函数的Prologue过程增加一次特殊的函数调用，
这个特殊的函数被称为"mcount"。


源码:
```c
int calculate(int n)
{
  return subcalculate(n);
}
```


```asm
0000000000000090 <calculate>:
  90:   a9be7bfd        stp     x29, x30, [sp, #-32]!
  94:   910003fd        mov     x29, sp
  98:   b9001fe0        str     w0, [sp, #28]
  9c:   b9401fe0        ldr     w0, [sp, #28]
  a0:   94000000        bl      70 <subcalculate>
  a4:   a8c27bfd        ldp     x29, x30, [sp], #32
  a8:   d65f03c0        ret
```



```asm
00000000000000d0 <calculate>:
  d0:   a9be7bfd        stp     x29, x30, [sp, #-32]!
  d4:   910003fd        mov     x29, sp
  d8:   aa1e03e1        mov     x1, x30
  dc:   b9001fe0        str     w0, [sp, #28]
  e0:   aa0103fe        mov     x30, x1
  e4:   d50320ff        xpaclri
  e8:   aa1e03e0        mov     x0, x30
  ec:   94000000        bl      0 <_mcount>
  f0:   b9401fe0        ldr     w0, [sp, #28]
  f4:   94000000        bl      9c <subcalculate>
  f8:   a8c27bfd        ldp     x29, x30, [sp], #32
  fc:   d65f03c0        ret
```




表1和表2展示了启动"-pg"选项前后编译函数calculate()生成的汇编代码，
通过执行以下指令实现：
```sh
aarch64-none-linux-gnu-gcc mcount.c -c -o mcount.o  -pg
```
编译器版本:
```sh
$ aarch64-none-linux-gnu-gcc --version
aarch64-none-linux-gnu-gcc (GNU Toolchain for the A-profile Architecture 10.3-2021.07 (arm-10.29)) 10.3.1 20210621
Copyright (C) 2020 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```
通过比较不难发现，增加的指令数并没有很多，_mcount()需要我们自己定义。
默认情况下，每个函数的追踪开关都是关闭状态，此时_mcount()什么都不做直接返回。
当用户显式的启用对某个函数的追踪时，_mcount()被替换成相应的追踪函数。
这样做使得，在每个函数调用前增加相关指令不会对性能造成严重影响，
因为大部分函数的追踪开关都是关闭状态。

如果对某个函数启用了追踪，在执行时会选择真正记录追踪信息的_mcount(), 
而不是空的函数。在新的_mcount()内部，会记录一些相关的数据，
以配合Stack Unwind的_mcount()为例，因为ARM64 Call Convention规定函数调用传参通过寄存器，
导致无论是基于Fp还是Dwarf的栈回溯方式无法恢复每个调用所携带参数。
如果想要同时查看每次调用时的参数，可以通过Function Trace辅助实现。
在开启函数追踪时，_mcount()会分析此函数的参数并存于Ringbuf中，
在调用Stack Unwind处打印Ringbuf的内容就能对应得到调用栈对应的参数了。

需要注意的是，"-pg"会与一些编译器优化选项产生冲突，
例如不能同时使用-fomit-frame-pointers，
gcc在执行时会帮我们进行检测，如果有相冲突的选择，则会报错。


# Hardware Event Trace