---
title: "虚拟化：中断虚拟化"
tags: ["Virtualization", "ARM"]
categories: ["Virtualization"]
date: 2024-08-08T19:28:12+08:00
---

探究中断虚拟化的方案，按照 GIC 的不同版本架构进行说明。

带有 Hypervisor 的架构来说，如何处理外部物理设备的中断就变得更加复杂：

1. 有些中断是给 Hypervisor 处理的
2. 有些中断是给 VM 处理的
3. 甚至，当分配给这个 VM 处理的中断到来的时候，这个 VM 没有被调度

在引入虚拟化之后，这些条件都是需要被考虑的。所以说，在设计中断虚拟化时，我们要从以下两个部分进行实现。分块也更加便于我们逐步理解中断虚拟化实现中软件/硬件的任务界限划分。

1. 一套能在 EL2 处理 Hypervisor 中断的机制
2. 一套能将部分中断映射转发到 VM 的机制

## 中断虚拟化的历史

知道历史才能更加清楚一个技术的巧妙。GIC 硬件支持虚拟化是从 GICv2 开始引入的，GICv3 又增加了和虚拟化相关的更多新功能。

软件做--性能太差/实现太复杂--硬件做

## 中断虚拟化的相关配置

vFIQ/vIRQ 是新加入的两条中断线，也就是说现在连接到 CPU 上共有四条中断线（IRQ/FIQ/vIRQ/vFIQ）。vFIQ/vIRQ 的特点是只能在 EL0 和 EL1 触发，而且只能在 NSecure 状态触发。

> To recap, support for virtualization in Secure state was introduced in Armv8.4A. For a virtual interrupt to be signaled in Secure EL0/1, Secure EL2 needs to be supported and enabled. Otherwise virtual interrupts are not signaled in Secure state.

### 触发 vIRQ/vFIQ 的两种方式

There are two mechanisms for generating virtual interrupts:

1. Internally by the core, using controls in HCR_EL2. `HCR.VI` = Setting this bit registers a vIRQ.`HCR_VF` = 触发 vFIQ。缺点是只提供了触发 vIRQ 的方式，但是没有其他的模拟啊，比如说怎么配置优先级？怎么 ACK、EOI 中断这些。所有 VM 对 GIC 的操作都需要 Trap 到 EL2 来模拟（emulate），性能太差。

2. 这种软件的方法由于性能迟早被硬件方法代替，Using a GICv2, or later, interrupt controller. From Arm GICv2, the GIC can signal both physical and virtual interrupts。The advantage of this approach is that the hypervisor only needs to set up the virtual interface, and does not need to emulate it. This approach reduces the number of times that the execution needs to be trapped to EL2, and therefore reduces the overhead of virtualizing interrupts.

接收到一个物理中断并处理的流程：

1. 物理设备产生中断信号，到达 GIC
2. CPU 读到 HCR.IMO/FMO 是 1，该中断会强制路由到 EL2
3. Hypervior 首先判断这个中断是谁处理的，如果是自己那就自己处理完默默回去
4. 如果是 VM 的中断，将 vINTID 和路由到哪个 vCPU 决定好
5. 按照 vINTID 更新 LR，也就是 GIC virtual interface control registers，比如说 vINTID 写入 vGICC_IAR。包括中断所属的 Group 信息。
6. 如果有多个 VM，最好调度到处于中断目标 vCPU 的 VM，减少中断延迟
7. 写完之后，物理 Interface 就可以写 EOI 了，根据 pGICC.EOImode，这个 EOI 不会 deactivate 中断，只是降低优先级而已。使得其他中断能够到达，即便 VM 还没有 ack 这个。但是这个 INTID 的中断不会到，所以不影响当前中断处理。
8. 因为是发送给 VM 的中断，需要由 GIC virtual interface 来发出中断信号，也就是 vIRQ/vFIQ。
9. Hypervisor 将控制权切换到 VM，VM 进入处理函数，在 VM 中读 GICC 其实读的就是 GICV。
10. VM 处理完中断后，写 EOI，根据 EOImode 配置，最好是将 lower running priority 和 deactivate 一步完成。此时才算真正处理完一个中断。

## 相关：中断直通
