---
title: "GNU C内联汇编学习笔记"
date: 2022-09-24T16:48:58+08:00
tags: [c]
---





### 语句结构

```c
asm asm-qualifiers ( AssemblerTemplate 
                      : OutputOperands
                      : InputOperands
                      : Clobbers
                      : GotoLabels)
```

The `asm` keyword is a GNU extension.  当使用编译选项 `-ansi` 或 `-std` 时, 使用 `__asm__`代替 `asm`.

#### Qualifiers

- volatile: 避免编译器的过分优化
- goto
- inline

#### Parameters

*AssemblerTemplate*: 字符串, 汇编代码的模板

*OutputOperands*: 输出操作数; 指令将会修改的变量集合

*InputOperands*: 输入操作数; 指令将读取的变量集合

*Clobbers*: ???TODO

*GotoLabels*: 仅当 qualifiers 使用`goto`时, 声明label集合.

> The total number of input + output + goto operands is limited to 30.

&nbsp;

### Param #1: AssemblerTemplate

多条语句可以放在一个asm字符串中, 但是更常见的是每条汇编语句使用一个字符串, 并在结束时使用换行符和制表符(`\n`, `\t`)来表示换行.

> 貌似对于 arm 汇编, 只用 `\n` 也OK? 

&nbsp;

### Param #2: OutputOperands

多个 OutputOperands 之间使用`,`隔开,  每个 OutputOperands 的格式如下:

```
[ [asmSymbolicName] ] constraint (cvariablename)
```

*asmSymbolicName*: 指定该操作数的名称

*constraint*: 对该操作数的一些限制

```
// 描述操作数的权限, 输出操作数的约束必须以此开头
=   忽略现有值
+   读写, 当原先值有意义时用它
&   禁止编译器将该操作数与不相关的输入操作数分配同一个寄存器

// 描述输出操作数所在位置, 如果你不知道, 可以同时设置, 编译器会帮你决定
r   寄存器
m   内存
```

*cvariablename*: 输出到的 C 语言变量名

&nbsp;

### Param #3: Input Operands

输入操作数的格式与输出操作数基本一致:

```
[ [asmSymbolicName] ] constraint (cexpression)
```

> 对于输入操作数, 一般没有别的限制, 仅使用`"r"(val)`

&nbsp;

### Param #4: Clobbers

每个 clobber 都是用双引号括起来, 并用逗号分隔的字符串常量.

常用的 clobber 参数:


- "memory"  
  告诉编译器, 这段内联汇编代码对输入和输出操作数中列出的项以外的内存读取或写入操作(例如，访问输入参数之一指向的内存). 为确保内存包含正确的值，GCC可能需要在执行ASM之前将特定寄存器值刷新到内存。此外, 阻止编译器越过该 ASM 语句进行 reorder, 形成针对编译器的 memory barrier. 注意, 此 clobber 不会阻止处理器在ASM语句之后执行推测性读取。为了防止出现这种情况，您需要特定于处理器的防护指令。

- "cc"  
  This stands for "condition codes". Since the add instruction will affect the carry flag amongst other things, we need to tell gcc about it. Otherwise it might want to split a test-and-branch around our code. If it did so, the branch might go the wrong way due to the condition codes being corrupted. Basically, any inline asm that does arithmetic should explicitly clobber the flags like this.


&nbsp;
### Param #5: GotoLabels

尽量不使用, 可以在ASM的内部直接定义 label

TODO



&nbsp;

### 样例

#### 最简单的模板

```c
int src = 1;
int dst;   

asm ("mov %1, %0\n\t"
    "add $1, %0"
    : "=r" (dst) 
    : "r" (src));

printf("%d\n", dst);
```

#### 操作数使用 asmSymbolicName

```c
uint32_t c = 1;
uint32_t d;
uint32_t *e = &c;

asm ("mov %[e], %[d]"
   : [d] "=rm" (d)
   : [e] "rm" (*e));
```

#### 内部定义 label

```c
    long temp;
    long ret;    
	asm volatile(
            "1:         \n"
            "ldxr %0, [%2]\n"
            "sub  %0, %0, %3\n"
            "stxr %w1, %0, [%2]\n"
            "cbnz %w1, 1b\n\t"
            : "=&r"(ret), "=&r"(temp) 
            : "r"(p), "r"(val)
            : "memory"
            );
```



## Reference

[Extended Asm (Using the GNU Compiler Collection (GCC))](https://gcc.gnu.org/onlinedocs/gcc/Extended-Asm.html#Extended-Asm)
