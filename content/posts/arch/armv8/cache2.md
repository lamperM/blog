---
title: "ARMv8: cache相关知识2"
date: 2024-07-14T22:02:04+08:00
tags: [armv8]
categories: ["Architecture"]
---

作为一个 OS 开发者，到底什么时候需要维护 Cache ？

## Cache alias（别名）问题

## Cache 的组织方式

如何检索 Cache 表项，因为访问地址/数据时有虚拟地址转物理地址的过程，所以检索可能以虚拟地址为索引，或者是先翻译，再以物理地址为索引去查找 Cache。Cache 检索时的行为取决与 Cache 的组织方式，这是由硬件决定的。

?I?T，I 代表 Index，即以哪种地址去索引。T 代表 Tag，即索引到表项后以什么地址去检查。因为 Cache 支持多路的原因，同一个 Index 可能对应多个表项，需要依次的和 Tag 比对，直到匹配成功或者 Cache Miss。

### VIVT(Virtual Index Virtual Tag)

早期的 ARM 处理器一般采用这种方式，全使用虚拟地址，这种方式会导致 cache 别名(cache alias)问题。

### VIPT(Virtual Index Physical Tag)

使用虚拟地址做索引，物理地址做 Tag。在利用虚拟地址索引 cache 同时，同时会利用 TLB/MMU 将虚拟地址转换为物理地址。然后将转换后的物理地址，与虚拟地址索引到的 cache line 中的 Tag 作比较，如果匹配则命中。

这种方式要比 VIVT 实现复杂，当进程切换时，不在需要对 cache 进行 invalidate 等操作(因为匹配过程中需要借物理地址)。但是这种方法仍然存在 cache 别名的问题，但是可以通过一定手段解决。

### PIPT(Physical Index Physical Tag)

使用物理地址做索引，物理地址做 Tag。现代的 ARM Cortex-A 大多采用 PIPT 方式，由于采用物理地址作为 Index 和 Tag，所以不会产生 cache alias 问题。不过 PIPT 的方式在芯片的设计要比 VIPT 复杂得多，而且需要等待 TLB/MMU 将虚拟地址转换为物理地址后，才能进行 cache line 寻找操作。

> Data cache invalidate on reset

> The Armv8-A architecture does not support an operation to invalidate the entire data cache. If software requires this function, it must be constructed by iterating over the cache geometry and executing a series of individual invalidate by set/way instructions.

outof-order and speculative execution

## Memory Barriers（内存屏障）

指令流水线，指令预取，Store buffer 等很多优化为了提高性能。然而，有时候一些前后的依赖微架构自身并不能保证，所以 ARM 指令集提供了一些显式的或者说强制的 barrier 来保证一些指令执行顺序/数据读写顺序。

### Instruction Synchronization Barrier

Instruction Synchronization Barrier(ISB) 指令确保后续指令在 isb 执行之后重新 fetch，即起到了刷新 pipeline 的效果。
用到 isb 的地方一般与内存、Cache 操作相关。确保ISB之前的context-changing操作会被之后的指令看到。

```assembly
	MRS X1, CPACR_EL1 // Copy contents of CPACR to X1
	ORR X1, X1, #(0x3 << 20) // Write to bit 20 of X1. (Enable FPU and SIMD)
	MSR CPACR_EL1, X1 // Write contents of X1 to CPACR
	ISB  // enable FPU 和 SIMD 后，后续指令执行的方式就会发生变化，
		 // 所以当前流水线上的指令必须重新fetch
```

```assembly
// Linux kernel: arch/arm64/kernel/head.S
// 其他的指令后面会介绍到，这个例子中可以仅关注isb
	msr	sctlr_el1, x19			// re-enable the MMU
	isb               // 开启MMU后，流水线上按照原来页表取得的指令就可能不对
					// 需要重新按照新的mapping来fetch
	ic	iallu				// 改变了mapping，Icache上的映射需要invalid
	dsb	nsh				// 确保前面icache invalid操作执行完毕
	isb               // 确保后面预取的指令不受 ICache old value 的影响
	                  // ，后续之行必须重新 Fetch
```

## Data Synchronization Barrier(DSB)

当处理器执行 DSB 指令时，会 block 后面指令执行，直到 DSB 前面的指令执行完成。

## DSB, DMB 携带参数

OSHLD Operation that waits only for loads to complete, and only to the outer shareable domain

ISH | Inner SHareable | Operation only to the Inner Shareable domain.

## DC 指令

## IC 指令

```
IC <ic_op>, {<Xt>}
```

- IC IALLU: Invalidate ALL to PoU，无效化所有到 PoU 的指令缓存行。
- IC IALLUIS: 表示 Invalidate all to PoU, Inner Shareable，无效化所有到 PoU 的，内部可共享（Inner Shareable）的指令缓存行。
- IC IVAU: 表示 Invalidate Virtual Address to PoU，无效化虚拟地址到 PoU 的指令缓存行。

## 案例 1

[assembly - Does AArch64 need a DSB after creating a page table entry? - Stack Overflow](https://stackoverflow.com/questions/58636551/does-aarch64-need-a-dsb-after-creating-a-page-table-entry)

填充一个空的页表，然后立马用虚拟地址去访问，这之间需要什么 fence 类指令吗？

```asm
str x1, [x0] ;x1 is phy addr for pte, x0 is pte_entry
; << need any fence?
ldr x2, [x3] ;x3 has VA that is mapped by above instruction
```

ARMv8 requires what they call a "break-before-make" procedure when 更新页表项。 This procedure is described in G5.9.1 in the ARMv8 ARM:

> A break-before-make sequence on 修改页表项 requires the following steps:
>
> 1.  Replace the old translation table entry with **an invalid entry**, and execute a DSB instruction.
> 2.  Invalidate the translation table entry with a broadcast TLB invalidation instruction, and execute a DSB instruction to ensure the completion of that invalidation.
> 3.  Write the new translation table entry, and execute a DSB instruction to ensure that the new entry is visible. This sequence ensures that at no time are both the old and new entries simultaneously visible to different threads of execution, and therefore the problems described at the start of this subsection cannot arise.

对于该问题来说，因为原来已经是无效的表项，所以可以直接跳过 1、2 步，直接执行第三步即可。
同时，针对这种情况，DSB 可以规定为 SH 参数来优化性能。**DSB 之后的 ISB 也是必须的，使后面的 ldr 指令重新 fetch**，因为页表已经更新，可能更新了下一条指令地址的页表项，使得新的 fetch 对应不同的指令，所以加一个 isb 比较安全。

## 案例 2

Linux Kernel `__enable_mmu()`

```asm
SYM_FUNC_START(__enable_mmu)
	...
	msr	ttbr0_el1, x2			// load TTBR0
	offset_ttbr1 x1, x3
	msr	ttbr1_el1, x1			// load TTBR1
	isb
	msr	sctlr_el1, x0
	isb                   // 开了MMU以后，后面根据PC预取的指令可能对应别的指令地址，需要重新fetch
	/*
	 * Invalidate the local I-cache so that any instructions fetched
	 * speculatively from the PoC are discarded, since they may have
	 * been dynamically patched at the PoU.
	 */
	ic	iallu
	dsb	nsh     // 充当一个barrier，确保上面icache clean完毕后，isb才commit
	isb         // 后面的指令可能之前从icache中fetch的，在icache clean后需要重新fetch
	ret
SYM_FUNC_END(__enable_mmu)
```

## 案例 3

自己做的项目中，修改当前 TTBR 寄存器后，没有做相应的处理。
正常来说，在 MMU 和 Cache 都 Enable 时，修改 TTBR 后，需要做几件事：

```asm
dsb st     ; Ensure writes to tables have completed

msr ttbr0_el2, x0 ; Set the root table

tlbi vmalle2 ; Invalidate TLB entry
dsb sy   ; Ensure TLB invalidation has completed
ic iallu ; Clean Icache if VIVT or aliasing VIPT
dsb sy   ; Ensure Icache clean has completed
isb      ; Refetch following instructions using 正确的 pagetable 和 cache、TLB
```



## 案例 4

```asm
.macro	__idmap_cpu_set_reserved_ttbr1, tmp1, tmp2
	adrp	\tmp1, empty_zero_page
	phys_to_ttbr \tmp2, \tmp1
	offset_ttbr1 \tmp2, \tmp1
	msr	ttbr1_el1, \tmp2
	isb   // 必须吗？
	tlbi	vmalle1 
	dsb	nsh
	isb
.endm
```