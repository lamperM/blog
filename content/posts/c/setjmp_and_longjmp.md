---
title: "基于ARM64实现setjmp/longjmp"
date: 2022-11-01T23:38:54+08:00
tags: [armv8]
---





##  介绍

setjmp() and longjmp() 是一对组合使用的函数, 可以实现**全局的goto**. 

setjmp() 构造一个运行环境, 调用longjmp() 则将执行流切换到该环境.

```c
/* setjmp() 保存当前的运行环境(上下文)到 env 参数中 */
int setjmp(jmp_buf env);

/* longjmp() 将控制流切换到 env 指定的运行环境 */
void longjmp(jmp_buf env, int val);
```

## 使用方法

```c
#include <setjmp.h>
#include <stdio.h>

jmp_buf e;

void foo() {
    longjmp(e, 1);
}

int main(void) {
    int ret;

    /* After calling longjmp(), the execution flow back to setjmp(), 
       and setjmp() will return not 0. */
    ret = setjmp(e);
    if (ret == 0) {
        printf("Return from setjmp\n");
        foo();
    } else {
        printf("Return from longjmp\n");
    }

    return 0;
}
```





## 基于 AArch64 的实现

需要保存的上下文包括

* **callee-saved 通用寄存器**, 因为可能第一次调用 `setjmp()` 之后的执行流修改了这些寄存器, 从第二次回到 `setjmp()` 的角度来看, 就是执行setjmp() 中破坏的. caller-saved 寄存器则不必, 因为本来即便看作是 `setjmp()` 破坏的, 也是正常的.

```asm
.macro func _name
.global \_name
.type \_name, %function
\_name:
.endm

.macro endfunc _name
.size \_name, .-\_name
.endm


/**
 * setjmp (jmp_buf env)
 *
 * See also:
 *          longjmp
 *
 * @return 0 - if returns from direct call,
 *         nonzero - if returns after longjmp.
 */

func setjmp
    stp     x19, x20, [x0], #16
    stp     x21, x22, [x0], #16
    stp     x23, x24, [x0], #16
    stp     x25, x26, [x0], #16
    stp     x27, x28, [x0], #16
    stp     x29, x18, [x0], #16
    mov     x9, sp
    stp     lr, x9, [x0], #16
    mov     x0, #0
    
    ret
endfunc setjmp

/**
 * longjmp (jmp_buf env, int val)
 *
 * Note:
 *      if val is not 0, then it would be returned from setjmp,
 *      otherwise - 1 would be returned.
 *
 * See also:
 *          setjmp
 */

func longjmp
    ldp     x19, x20, [x0], #16
    ldp     x21, x22, [x0], #16
    ldp     x23, x24, [x0], #16
    ldp     x25, x26, [x0], #16
    ldp     x27, x28, [x0], #16
    ldp     x29, x18, [x0], #16
    ldp     lr, x9, [x0], #16
    mov     sp, x9

    mov     x0, x1
    cbnz    x0, 1f
    add x0, x0, #1
1:

    ret
endfunc longjmp
```



## setjmp() 实现 try-catch
