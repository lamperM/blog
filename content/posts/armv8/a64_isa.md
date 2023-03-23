---
title: "A64 指令集"
tags: ["armv8"]
date: 2022-05-07T21:19:01+08:00
---

## Load/Store 指令

### #1 Basic Load/Store

#### Addressing mode
1. Base register - `w0=[x1]`
```c
ldr     w0, [x1]
```

2. Offset addressing mode - `w0=[x1+12]`
```c
ldr     w0, [x1, 12]
```

3. Pre-index addressing mode - `x1+=12; w0=[x1]`
```c
ldr     w0, [x1, 12]!
```

4. Post-index addressing mode - `w0=[x1]; x1+=12`
```c
ldr     w0, [x1], 12
```

#### Load/store instruction example
```c
// load a byte from x1
ldrb    w0, [x1]

// load a signed byte from x1
ldrsb   w0, [x1]

// store a 32-bit word to address in x1
str     w0, [x1]

// load two 32-bit words from stack, then add 8-byte to sp
ldp     w0, w1, [sp], 8

// store two 64-bit words at [sp-96] and subtract 96-byte from sp.
stp     x1, x2, [sp, -96]!

// LDR伪指令. load 32-bit immediate from literal pool(addr: 0x12345678)
ldr     w0, =0x12345678
```



## 数据处理指令

### Bitfield 操作指令

Bitfield指令常用于设置/提取寄存器的某个字段. 

```assembly
;BFI(Bit Field Insert)
BFI w0, w0, #9, #6   ;w0[0, 5] = w0[9, 14]

;BFC(Bit Field Clear)
BFC w0, #4, #2       ;w0[4, 5] = 0

;UBFX(Unsigned Bit Field Extract)
UBFX w1, w0, #18, #7 ;w1=w0[18, 24]
```

>与UBFX相对的是SBFX, 若提取后的字段高位为1, 会进行符号扩展

&nbsp;

## 分支/控制指令

## 其他指令

### #1 ADR/ADRP 指令: 

> 既然有了LDR伪指令, 为什么需要ADR指令呢?
>
> 



&nbsp;

## 有趣特性/常见误区

### '#' before the immediate value
* A64 assembly language does not require the `#` to introduce constant immediate value. But the assembler can also indentify the `#`.
* In armv7, there must be a `#` or `$` before other than using `.syntax unified`. [About syntax unified](https://sourceware.org/binutils/docs/as/ARM_002dInstruction_002dSet.html#ARM_002dInstruction_002dSet).

> [Agreed Recommendation](https://stackoverflow.com/questions/21652884/is-the-hash-required-for-immediate-values-in-arm-assembly)
>
> Use `.syntax unified` in v7 code, and never use `#` on any literal on either v7 or v8.
> Unified syntax is newer and better, and those `#` and `$` signs are just more code noise.

