---
title: "ARMv8-A Register"
tags: ["armv8"]
categories: ["embedded"]
date: 2022-05-07T20:19:44+08:00
---

# Classification
## General-purpose Register
1. `x0-x7` 参数寄存器: Restore function parameters and return vaule.
2. `x9-x15` caller-saved 临时寄存器: callee 默认可以直接使用来保存临时变量, 不需要保存和恢复. 如果 caller 在里面存储了非临时信息, 那么在函数调用之前应当由 caller 负责保存.
3. `x19-x28` callee-saved 寄存器: callee 应该避免使用. 如果必须要使用，那么在返回前必须恢复.
4. special registers:
    * `x8` restore indirect result. Commonly used when returning a struct.
    * `x18` platform reserved register.
    * `x29` frame pointer register(FP).
    * `x30` link register(LR).
> All general-purpose register `xN` is 64-bit width. They all have corresponding `wN` register using the lower 32-bit of `xN`. And write to `wN` will clear the upper 32bit of `xN`.

## Special Register per EL
1. `sp_el0/1/2/3` stack pointer register of each EL.
2. `elr_el1/2/3` exception link register of each EL except EL0.
3. `spsr_el1/2/3` save program status register of each EL except EL0.
> `sp` is an alias of `sp_el0`. Do NOT treat `sp` as general-purpose register.
