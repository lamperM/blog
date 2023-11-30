
## Stack Trace

### GDB `frame` Commond
看某个frame点的信息, sp的值，caller是谁，prologue阶段存的寄存器是那些，值是什么。
**其实我关心的是参数，它应该是不能实现的**。
```
Stack level 1, frame at 0xffff00004004aff0:
 pc = 0xffff000040023f50 in load_root_service (src/init.c:67); saved pc = 0xffff00004002405c
 called by frame at 0xffff00004004b000, caller of frame at 0xffff00004004afc0
 source language c.
 Arglist at 0xffff00004004afc0, args:
 Locals at 0xffff00004004afc0, Previous frame's sp is 0xffff00004004aff0
 Saved registers:
  x29 at 0xffff00004004afc0, x30 at 0xffff00004004afc8
```

### Method: DWARF CFI
Dwarf(Debugging With Attributed Record Formats)是一种调试信息的存储格式，调试器的开发者经常用到它。

基于Fp的unwinding方法有一个缺点是不够准确，当在Prologue和Epilogue阶段。

Dwarf提出了一种解决方案，在Dwarf的标准中，定义了一种名为CFI(Call Frame Information)的标准，用来帮助实现Stack Trace，**无需Fp的支持**。

简单来说，编译过程中会记录一张表，里面将PC划分了若干个块，每个块内获取到Caller stack frame需要进行的sp偏移、返回地址都是相同的。
在需要进行Stack Unwinding时只需要查这张预先建立的表就能得到结果，与运行时动态回溯的基于Fp的方法不同。

### 为什么.debug_frame还记录Callee-saved寄存器的变化

### 能解决参数的问题吗？
不行。ARM强制大部分参数用寄存器传递，并不是在栈上，所以没办法恢复。




[DWARF可以实现Stack Trace](https://zhuanlan.zhihu.com/p/74503126)


### Good to Read

- [SFrame: The Simple Frame Stack Trace Format](https://www.youtube.com/watch?v=4XrFYpjyodo)
- [介绍很多Stack Unwinding方法](https://blog.csdn.net/pwl999/article/details/107569603)
- [AArch64平台Stack Unwinding的实现](https://bbs.kanxue.com/thread-270936.htm)
- [介绍CFI的原理](https://lesenechal.fr/en/linux/unwinding-the-stack-the-hard-way#h2-eh_frame-call-frame-information-cfi)
- https://nikhilism.com/post/2019/retrieving-function-arguments-while-unwinding-the-stack/
- [.eh_frame](https://www.airs.com/blog/archives/460)
- [.eh_frame vs .debug_frame](https://stackoverflow.com/questions/76200555/eh-frame-vs-debug-frame-section)
- [以问答的形式解释stack unwind](https://blog.reverberate.org/2013/05/deep-wizardry-stack-unwinding.html)
- https://eli.thegreenplace.net/2015/programmatic-access-to-the-call-stack-in-c/


## Hardware Event Trace