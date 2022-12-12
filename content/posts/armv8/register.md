---
title: "ARMv8-A Register"
tags: ["armv8"]
categories: ["embedded"]
date: 2022-05-07T20:19:44+08:00 
---



# 寄存器分类

## 通用寄存器
1. `x0-x7` 参数寄存器: Restore function parameters and return vaule.
2. `x9-x15` caller-saved 临时寄存器: callee 默认可以直接使用来保存临时变量, 不需要保存和恢复. 如果 caller 在里面存储了非临时信息, 那么在函数调用之前应当由 caller 负责保存.
3. `x19-x28` callee-saved 寄存器: callee 应该避免使用. 如果必须要使用，那么在返回前必须恢复.
4. special registers:
    * `x8` restore indirect result. Commonly used when returning a struct.
    * `x18` platform reserved register.
    * `x29` frame pointer register(FP).
    * `x30` link register(LR).
> All general-purpose register `xN` is 64-bit width. They all have corresponding `wN` register using the lower 32-bit of `xN`. And write to `wN` will clear the upper 32bit of `xN`.

> 💫 The different between **Caller-saved** and **callee-saved** registers
>
> * Caller-saved 寄存器又称为*临时寄存器*, 常用来存放临时变量. 例如A() 调用 B(), 那么 B() 可以直接使用 caller-saved 寄存器, 也就是说 A() 在调用 B() 之前不会在这些寄存器里保存重要信息(编译器实现), 不能保证调用 B() 前后其值不变. 如果必须要保证, 那么保存和恢复(利用栈)这件事是 A() 来做. 
> * Callee-saved 寄存器则相反, 通常持续使用的值会保存到这些寄存器中. 还是拿 A() call B() 来举例. 如果 A() 中的一个变量需要在调用 B() 前后持续有效, 那么它应当保存到 callee-saved 寄存器中. 而且 B() 正常来说不应该动这些寄存器, 如果非得动(例如寄存器不够用), 那么 B() 需要在使用他们的前后进行保存和恢复(利用栈).

## 每个EL的特殊寄存器

1. `sp_el0/1/2/3` stack pointer register of each EL.
2. `elr_el1/2/3` exception link register of each EL except EL0.
3. `spsr_el1/2/3` save program status register of each EL except EL0.
> `sp` is an alias of `sp_el0`. Do NOT treat `sp` as general-purpose register.
