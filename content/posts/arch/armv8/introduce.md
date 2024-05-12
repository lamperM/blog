---
title: "ARMv8 基础"
tags: ["arch", "armv8"]
categories: ["Architecture"]
date: 2023-05-09T21:19:01+08:00
---

## ARMv8 与 ARMv7 相比的改动

- 指令集： 新增 A64 指令集， 但也兼容原来的 A32 指令集
- 权限等级： AArch64 下新增 EL0-EL3 异常等级，对应 V7 的特权等级
- 通用寄存器：31 个通用寄存器，V7 15 个
- 虚拟地址长度：64 位的地址长度，理论支持 256TB 的寻址范围

## 关于 Spsel 寄存器的使用

linux 内核里，内核（EL1）和用户态（EL0）都使用各自的栈空间，即 spsel 始终为 1。
这种情况下，当内核里时，sp_el0 是可以复用的寄存器。进入内核前保存原值，然后将其保存当前进程 task_strcut 结构体的地址。因为内核中经常会调用 current 宏，这时可以快很多。

atf 中，在 EL3 用 sp_el0 作为运行时栈空间，而 sp_el3 保存一个重要结构上下文的地址。在进入 EL3 时，系统会自动切换到 spsel=1，即 sp_el3，此时
（1）保存当前的上下文到 sp_el3
（2）切换到 sp_el0 当作 c 调用栈
看起来好像是反过来，我能想到的原因是：

- ATF 在 ELF 如果用 sp_el0 指向结构体，在外面有可能被破坏？而用 sp_el3 在外面不会被动

https://developer.arm.com/documentation/ka005621/latest/

## 关于 PMU

PMU 是一个独立的单元，不和体系结构绑定。而是每个 SOC 都可以不同。比如说 Cortex-A53 实现了 PMUv3 架构，但别的基于 ARMv8 架构的 Soc 可能实现 PMUv4 或者其他版本。

PMU 内部有六个计数器，所以可以记录六个事件的发生次数。计数器的数值不一定绝对的正确，因为管道的存在，所以一般来说还是通过长时间计数来减弱影响。

### PMU 和 ETM 的区别

记录的事件不同

- PMU：Cache Miss、分支预测失败、TLB Miss 等
- ETM：记录分支指令、内存屏障指令等所有指令的执行，包括地址、结果等。
  另外还可以记录数据读写的地址、结果（可选）。

记录的粒度不同

- PMU：仅用计数器来记录事件发生的次数
- ETM：指令的类型、地址、执行结果等。数据访问也类似。

所以说，ETM 的信息量大，需要专门的缓存机制。而 PMU 只需在定时器结束时记录发生的次数就行，
不需要什么缓存，没有实际的数据流。

## 通用寄存器

1. `x0-x7` 参数寄存器: Restore function parameters and return vaule.
2. `x9-x15` caller-saved 临时寄存器: callee 默认可以直接使用来保存临时变量, 不需要保存和恢复. 如果 caller 在里面存储了非临时信息, 那么在函数调用之前应当由 caller 负责保存.
3. `x19-x28` callee-saved 寄存器: callee 应该避免使用. 如果必须要使用，那么在返回前必须恢复.
4. 特殊寄存器:
   - `x8` restore indirect result. Commonly used when returning a struct.
   - `x18` platform reserved register.
   - `x29` frame pointer register(FP).
   - `x30` link register(LR).

> All general-purpose register `xN` is 64-bit width. They all have corresponding `wN` register using the lower 32-bit of `xN`. And write to `wN` will clear the upper 32bit of `xN`.

{{< notice info "Caller-saved&callee-saved" >}}

- Caller-saved 寄存器又称为*临时寄存器*, 常用来存放临时变量. 例如 A() 调用 B(), 那么 B() 可以直接使用 caller-saved 寄存器, 也就是说 A() 在调用 B() 之前不会在这些寄存器里保存重要信息(编译器实现), 不能保证调用 B() 前后其值不变. 如果必须要保证, 那么保存和恢复(利用栈)这件事是 A() 来做.
- Callee-saved 寄存器则相反, 通常持续使用的值会保存到这些寄存器中. 还是拿 A() call B() 来举例. 如果 A() 中的一个变量需要在调用 B() 前后持续有效, 那么它应当保存到 callee-saved 寄存器中. 而且 B() 正常来说不应该动这些寄存器, 如果非得动(例如寄存器不够用), 那么 B() 需要在使用他们的前后进行保存和恢复(利用栈).
  {{< /notice >}}

## 异常返回指令（ARM32/64）

当异常处理程序结束后，需要执行*异常返回指令*恢复进入异常之前的状态。具体来说:

1. 恢复发生异常前的 PC
2. 从 SPSR 中恢复 PSTATE 寄存器(现场)

异常返回的指令根据当前**执行状态**为 AArch32 还是 AArch64 有所不同.

### AArch32

AArch32 的异常返回指令在不同的**模式**下也有所不同:

- **若异常是在 Hyp 模式下处理:** 仅可执行`ERET`指令从异常返回.

- **若异常是在其他模式下处理**, AArch32 提供了以下的异常返回指令:
  - `ERET` 指令
  - 使用带 S 后缀的数据处理指令直接操作 PC(例如, `MOVS, PC, LR`), 恢复 PSTATE
  - RFE 指令: `RFE <Rn>`. 从基址寄存器<Rn>指向的地址依次加载 PC 和 PSTATE
  - LDM 指令: `LDM <Rn> {pc..}`. 若目标寄存器中包含 PC, 则会同时恢复 PSTATE

### AArch64

AArch64 下**统一使用** `ERET` 指令进行异常返回.

### 指令格式及用法

#### ERET

ERET 指令自动完成:

1. 从 ELR_ELx 中恢复 PC 指针
2. 从 SPSR_ELx 中恢复 PSTATE 寄存器的状态.

#### LDM(Load Multiple)

- 格式: `LDM <Rn> {registers}`
- 含义: 从基址寄存器`<Rn>`指向的地址开始依次加载多个寄存器值. 若目标寄存器中包含 PC, 则同时恢复 PSTATE.

例如: `LDM <r0> {pc, r1}` 等价于:

```assembly
pc = [r0]
r1 = [r0+4]
PSTATE = SPSR  ;仅当目标寄存器包含PC时自动完成
```

#### RFE(Return From Exception)

- 格式: `LDM <Rn> `
- 含义: 从基址寄存器`<Rn>`指向的地址依次加载 PC 和 PSTATE.

例如: `RFE <r0>` 等价于:

```ass
pc = [r0]
PSTATE = [r0+4]
```

## Armv8.2 SPE

SPE 的全称为 Statistical Profiling Extension, 统计分析扩展，
是 Armv8.2 引入的一个特性。

PMU 能统计事件发生的次数，但是无法直到是哪一条指令导致的。
采样因为要通过中断的方式，所以准确性不高，太高的准确率会造成系统很大的负担。
因此开发者一般只能确定是哪一个函数，但是确定哪一行就比较困难。

SPE 就是通过硬件的方式解决这个问题，直接在流水线上对指令进行采样。
用硬件对性能的损耗就会很低。

有个计数器，没取指一次，计数器-1，减到 0 之后的指令就是要采样的指令。

采集的指令非常全面：时间戳、PC、指令的分类、运行的时间等。

采集之后，可以做一次过滤，只保存关心的事件（执行时间大于几个 Cycle？指令类型 lr/str？）

保存的 buffer 满了之后，触发中断，软件读取。

### 借助 perf 使用 SPE

- FEAT_SPE Armv8.2 Support from 4.14 and 5.3
- FEAT_SPEv1p1 Armv8.5 Only support from 5.11

```sh
perf list | grep arm_spe_0

perf record -e arm_spe_0/branch_filter=1,ts_enable=1,pct_enable=1,pa_enable=1,\
load_filter=1,jitter=1,store_filter=1,min_latency=0/ -- ./user.app

perf report -D -i perf.data
```

- [Arm 架构下性能分析与优化介绍-云视频-阿里云开发者社区](https://developer.aliyun.com/live/252287?spm=a2c6h.28322828.J_3909373260.17.88865488AzlJRR)
