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

## AArch64 调用约定

大体的思想与x86相似, 只是细节有些许不同

### 函数调用

ARMv8架构中, 函数调用以一条`bl`指令为分界.

执行`bl`指令之前, 需要将参数准备好. 注意, **ARMv8中, 少于8个参数的函数在传参时, 参数是放在x0-x7中**,  最左边的参数先使用x0, 以此类推. 参数超过8个的情况下才使用栈,  这与x86的方式不同.

`bl`指令保存返回地址, 并跳转到callee执行, 其语义是:

```asm
mov lr, pc+1     ;preserve return address
                 ;lr specially used for preservering return addr
mov pc, new_func ;set pc = new function
```

与x86相同, 跳转到callee之后必须先进行栈的设置, **Arm与x86不同的是它不需要管理栈底寄存器**. 因为参数大部分是通过寄存器来传递, 返回地址也是存储在`lr(x30)`寄存器中, 没必要为了极少的情况来做优化. 

```asm
sub sp, sp, #enough-space
```

### 函数返回

要完成两件事: (1) 恢复栈 (2)返回原来位置执行

先说(2), 由于`lr`寄存器始终保存返回地址, 直接 `mov sp, lr` 就能返回caller继续执行. **这也就是`ret`指令的语义**.

(1)恢复栈的这件事同x86一样由callee完成, 

```asm
add sp, sp, #enough-space
```



## 参考

1. x86 call 指令执行前需要esp对齐到 16-byte: [x86 - What are the following instructions after this call (assembly) - Stack Overflow](https://stackoverflow.com/questions/41971481/what-are-the-following-instructions-after-this-call-assembly)
2. [x86栈帧原理 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/290689333)

## 结语

能够正确达到函数调用和返回的实现方式有很多, **不是仅有这一种方式**, 约定仅仅是一个约定, 大家都这样去做降低了开发的难度. 
