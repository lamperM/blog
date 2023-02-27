---
title: "C 内敛汇编"
date: 2023-01-03T20:54:31+08:00
tags: [c, assembly]
draft: true
---

**如果要将多条语句放在一个`asm`关键字中**, 每条语句之间必须使用分隔符, 常见的为`\n\t`. 例如:

```c
asm volatile(
    "isb   \n\t"
    "dsb   \n\t"
    "eret  \n\t"
    );
```

> 貌似对于 arm 汇编, 只用 `\n` 也OK? 
> TODO: 找到依据

## volatile

#### cc

This stands for "condition codes". Since the add instruction will affect the carry flag amongst other things, we need to tell gcc about it. Otherwise it might want to split a test-and-branch around our code. If it did so, the branch might go the wrong way due to the condition codes being corrupted. Basically, any inline asm that does arithmetic should explicitly clobber the flags like this.

#### memory
The "memory" clobber tells the compiler that the assembly code performs memory reads or writes to items other than those listed in the input and output operands (for example, accessing the memory pointed to by one of the input parameters). To ensure memory contains correct values, GCC may need to flush specific register values to memory before executing the asm. Further, the compiler does not assume that any values read from memory before an asm remain unchanged after that asm; it reloads them as needed. Using the "memory" clobber effectively forms a read/write memory barrier for the compiler.

Note that this clobber does not prevent the processor from doing speculative reads past the asm statement. To prevent that, you need processor-specific fence instructions.




