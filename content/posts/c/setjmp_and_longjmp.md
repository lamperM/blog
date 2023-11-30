---
title: "ARM64 上实现 setjmp/longjmp"
date: 2022-11-01T23:38:54+08:00
tags: [armv8, c]
categories: ["C Language"]
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

需要保存的上下文包括

* **callee-saved 通用寄存器**, 因为可能第一次调用 `setjmp()` 之后的执行流修改了这些寄存器, 从第二次回到 `setjmp()` 的角度来看, 就是执行setjmp() 中破坏的. caller-saved 寄存器则不必, 因为本来即便看作是 `setjmp()` 破坏的, 也是正常的.

解释一下：在如下的调用场景中, 可以看到在setjmp()之前分别赋值了x2和x19，
这其中x2是caller-saved，x19属于callee-saved。
callee-saved寄存器在子函数调用之前和之后应该是不变的，所以说如果子函数需要修改他们应该在函数进入前进行备份。
所以说显然在(1)的步骤中应该有x19的备份过程，(6)中也应该有对应的恢复。

那么对于setjmp()呢？它自身也是一个函数，也需要满足规定。所以必须保证(5)访问到数据一定是(3)中保存的。
问题是对于setjmp()的使用方法来说，不是都遵循(1)(2)(3)(4)(5)的调用，下次会从别的位置直接跳到(4)。
此时仍然需要确保x19寄存器的正确性，**只能在首次调用setjmp()时将x19保存下来，以后每次不管从哪里调过来先去恢复保存的x19**。这样就能够实现callee-saved寄存器的正确性。

再说说caller-saved寄存器，也就是demo中的x2为例，在中间跨越一个setjmp()的条件下，
arch不能保证(4)中得到的值一定是(2)保存的，因为setjmp()可以随意的修改caller-saved寄存器。所以说，如果func()不得不要求(4)访问值的正确性，
一个解决方法就是**在调用setjmp()之前保存x2到栈中，在setjmp()之后立马恢复**。
这是caller的责任，所以这个过程会在func()完成，而不是setjmp()内部。

以上就是为什么setjmp()只需要保存callee-saved寄存器的解释。
```c
func()
{
  // (1)
  Set    x2  // (2)
  Set    x19 // (3)
  setjmp()
  Access x2  // (4)
  Access x19 // (5)
  // (6)
}
```


## setjmp() 实现 try-catch
