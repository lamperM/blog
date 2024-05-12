---
title: "AArch64/X86 函数调用约定"
date: 2022-11-21T10:30:35+08:00
tags: [armv8, x86, Operating System]
categories: ["Architecture"]
---

符合调用约定使得调用函数能够正常获取参数, callee结束之后能够回到原来位置继续执行.

## X86 调用约定

### 函数调用

x86架构中, 函数调用以一条`call`指令为分界.

在`call`指令执行之前, 所有的参数必须都躺在栈中, 参数入栈的规则是: **第一个参数最后入栈**. 

另外, 执行`call`指令之前, 必须确保栈指针`esp`是16-byte对齐. 这项工作是**编译器**完成的, 如果它判断参数入栈之后的`esp` 不满足对齐条件, 则会手动调整`esp`使之对齐. 实现方式见下面例子.

`call` 指令的语义是:

```asm
push pc+1    ;push next insttuction
mov pc, func ;set pc = new function
```

`call` 指令之后的下一条指令就是callee的内容了, 至此就算是进入新函数的地盘. 

但是在执行新的任务之前,  callee还需要完成**栈的转换**, 因为此时使用的栈还是caller的. 

```asm
push ebp     ;preserve location of caller's stack
mov ebp, esp ;new ebp is old esp
```

**此时`esp`也就是栈指针等于`ebp`, 这是callee栈的初始条件**. 万事俱备, 可以开始执行callee的实际任务了.

> `ebp`在整个函数执行过程中是固定的, 好处是: 能够**快速的或者函数参数, 返回地址**.

### 函数返回

callee执行完毕后, 需要返回到caller继续执行. 刚才说过, callee的返回地址在栈中, 所以我们要做的是找到返回地址所在的位置, 然后使`pc = 返回地址`. 当然, 还有另一个重要的任务就是**恢复caller的栈**.

上述任务的实现使用两条汇编语句就可完成: `leave` 和 `ret`.

`leave` 负责搞定栈, 其语义为:

```asm
mov esp, ebp   ;回滚栈空间
pop ebp        ;恢复caller的ebp
```

`ret` 负责搞定`pc`, 其语义为:

```asm 
pop ebx        ;取出返回地址
mov pc, ebx    ;jmp to 返回地址
```

`ret` 之后, 就算是返回caller的地盘了. 还有一件小事别忘了做: **用于保存参数的栈空间还没有回收**, 回到caller之后需要先将`esp`的位置进行调整. 

### Example: 函数的调用和返回

一个关于函数调用和返回实现的完整例子.

```c
void caller()
{
    Func(1, 2, 3);
}
void Func(int a, int b, int c)
{
    /* Do something */
}
```

(以下汇编是**AT&T**格式的, 请见谅).

```asm
; Caller
sub    $0x4,%esp  ;make 16-bytes align before call. 0x4 是由编译器计算的
push   $0x3       ;push 参数, 顺序是从右到左
push   $0x2
push   $0x1
call   f01000ad <Func> ;Func()'addr is f01000ad
;===========>> Turn to callee
                            
                            ;Func()
                            push   %ebp      ;preserve old ebp
                            mov    %esp,%ebp ;set new ebp, ebp=esp now
                            /* Do something */
                            leave            ;restore stack
                            ret              ;restore instruction point
                            
;<<=========== Back to caller 
add    $0x10,%esp   ;recycle stack(12 bytes parameters plus 4 bytes alignment)
```

# AArch64 调用约定

大体的思想与x86相似, 只是细节有些许不同

## Call A Function

**ARMv8的函数调用以bl指令为分界**，在bl执行之前，caller需要将参数准备好。**少于8个参数的函数在传参时, 参数是放在x0-x7中**,  最左边的参数先使用x0, 以此类推. 参数超过8个的情况下才使用栈,  这与x86的方式不同.




`bl`指令保存返回地址, 并跳转到callee执行, 其语义是:

```asm
mov lr, pc+1     ;preserve return address
                 ;lr specially used for preservering return addr
mov pc, new_func ;set pc = new function
```

**至此到了callee的职责范围**。进入函数时, 需要完成三个前置动作:

```asm
; 为callee()的执行留出足够的栈空间
; 0x10不是固定的, 编译器计算得到
sub sp, sp, #0x10

; 保存lr和fp, 因为callee可能调用
; 其他的函数, 会破坏当前的
stp x29, x30, [sp]

; 保存sp的值, 所以x29也称为帧指针
; 原因是sp在callee的执行过程中可能
; 会变化. 与退出时配合理解较好
mov x29, sp
```

上述动作完成后, sp是自由的了, callee()的函数体开始执行.

## Function Return
callee()执行完后, 也需要执行一些后置动作, 以便恢复调用前caller()的环境.

```asm
; 经过callee的指令执行过后, sp
; 可能早就不是以前的sp了, 但是帧指针
; fp总是保存着callee初始的sp
mov sp, x29

; callee如果调用了其他函数, lr和fp
; 也会被破坏, 所以从栈中恢复callee
; 正确的lr和fp, 才能正确回到caller
ldp x29, x30, [sp]

; 收回为callee分配的栈空间, 此时sp
; 是调用callee之前的值.
add sp, sp, #0x10
```

后置动作完成后, 环境已经大致恢复到调用callee()之前的状态了, 通用寄存器中的值, 其实不必担心, 因为ARMv8规定了哪些寄存器是caller-saved或者callee-saved, 再说到lr和sp, caller也会保存到栈中, 就像callee调用其他的函数时的情况相同.

所以, 完成后置动作后, 执行`ret`来返回到lr的位置即可.

## 为什么编译出的汇编文件没有`mov sp, x29`?

如果查看实际C函数编译后的汇编结果, 大部分情况下会发现前置和后置行为不是严格的按照A64PCS中规定的样子, 常见的是后置动作缺少恢复sp的执行, 即`mov sp, x29`.

原因其实不难发现, callee中使用的寻址方式都是相对于sp寻址, 即不修改sp. 既然sp从始至终都没被改过, 自然也不需要恢复了. 这样不需要每次都需要sp, 更加高效.

**但是, 前置动作中的保存sp到fp的行为通常是不会被省略的, 我猜是因为调试行为(例如backtrace)的实现需要用到fp.**

不是每次都这样, 也有时会修改sp, 此时就必须恢复.

## 帧指针(fp)存在的意义

上面说了, 如果一直使用sp相对寻址来操作栈, 那么fp的存在似乎是非必要的.

我了解到的事实也是如此, fp存在的必要性似乎就是使得一些调试功能更加方便. **堆栈回溯**是一个典型的利用fp的场景, 从callee的fp可以找到其caller的fp, 也就是caller的堆栈空间, 以此类推可以找到caller-caller的fp...




## 如何关闭帧指针

仅针对AArch64来说, GCC提供了一些相关的编译选项来给程序员选择是否启用fp的权利.

- `-fomit-frame-pointer` 强制省略fp
- `-fno-omit-frame-pointer` 强制不省略fp

# 参考

1. x86 call 指令执行前需要esp对齐到 16-byte: [x86 - What are the following instructions after this call (assembly) - Stack Overflow](https://stackoverflow.com/questions/41971481/what-are-the-following-instructions-after-this-call-assembly)
2. [x86栈帧原理 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/290689333)
3. [破获ARM64位CPU下linux crash要案之神技能：手动恢复函数调用栈](https://www.cnblogs.com/coder51up/p/6940030.html)

## 结语

能够正确达到函数调用和返回的实现方式有很多, **不是仅有这一种方式**, 约定仅仅是一个约定, 大家都这样去做降低了开发的难度. 
