---
title: "A64 Instruction Set"
tags: ["armv8", "a64"]
categories: ["embedded"]
date: 2022-05-07T21:19:01+08:00
---

# Load/Store Addressing
## Addressing mode
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

## Load/store instruction example
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

// load 32-bit immediate from literal pool(addr: 0x12345678)
ldr     w0, =0x12345678
```


# Interesting Features
## '#' before the immediate value
* A64 assembly language does not require the `#` to introduce constant immediate value. But the assembler can also indentify the `#`.
* In armv7, there must be a `#` or `$` before other than using `.syntax unified`. [About syntax unified](https://sourceware.org/binutils/docs/as/ARM_002dInstruction_002dSet.html#ARM_002dInstruction_002dSet).

> [Agreed Recommendation](https://stackoverflow.com/questions/21652884/is-the-hash-required-for-immediate-values-in-arm-assembly)
>
> Use `.syntax unified` in v7 code, and never use `#` on any literal on either v7 or v8.
> Unified syntax is newer and better, and those `#` and `$` signs are just more code noise.

