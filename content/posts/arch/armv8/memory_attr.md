---
title: "AArch64 内存属性与内存类型"
date: 2023-04-12T08:01:33+08:00
tags: [armv8]
categories: ["Architecture"]
---

有了虚拟内存系统之后，MMU 可以抽象出一些可配置的内存属性。

例如，配置某个虚拟内存区域为不可执行、不被 cache 等，不可执行的属性
有助于防范攻击，不进入 cache 经常划分给外设 Memory-mapped 区域。

# 内存属性和内存类型

首先，我们没法直接设置内存类型，我们能设置的是一些细粒度的内存属性字段，
比如说权限(WRX)、cacheable、shareable 等。

我们说的内存类型也就是某些有意义的属性字段相互组合，ARM 给出了两种内存类型: 普通内存和设备内存。

- 普通内存会启用架构提供的所有优化技术，例如合并访存、乱序执行等。所以
  普通内存有最高的性能，但同时不是那么的“安全”，需要底层人员手动使用
  **内存屏障**等手段保证某些情况下的顺序性要求。

- 设备内存，顾名思义，常映射到外设的 Memory-mapped 区域。对于设备来说，
  那些提高性能的技术会造成一些问题，例如某些寄存器的配置必须按照顺序，
  这时就不能使用乱序执行。设备内存就牺牲了性能，优先保证正确性。

配置内存类型也是通过页表项中的其中一个属性字段: `AttrIndx[2:0]`,
它与系统寄存器`MAIR_EL1`配合实现。

具体表现为: `mair_el1`寄存器被划分为 8 个字段，我们为每个字段写入
不同的值可代表不同的内存类型和一些配套属性，具体的真值表可以参见
`mair_el1`寄存器的描述。

> `mair_el1`中内存类型配套属性只是属性的一部分，是和设备类型绑定的那部分。

## cacheable&shareable 傻傻分不清

先说 cacheable，一段内存被设置为 non-cacheable 属性说明不会进入 cache，
inner-cacheable 是实现定义的，可能指进 L1 cache/L2 cache， outer-cacheable
说明会进入 L3 cache。

要注意，**只有普通内存才支持配置是否进入 cache，所有的设备内存需要 non-cacheable**。

> 内存支持配置为是否被 cacheable，这在`mair_el1`的字段中配置。

shareable 说的是一块内存的外部可见性，外部不可见并不是真的看不到，只是说不保证值的正确性。

shareable 属性和 cacheable 其实是有关联的，他们俩比如配合使用，不能随便设置:

- 如果一块内存是 cacheable 的，则需要硬件提供 cache 的一致性维护机制。
  如果不能保证 cache 的一致性，想要启用 shareable 就必须是 non-cacheable
- 对于 non-cacheable 的内存，一定是 shareable 的，
  不需要配置。因为此时对数据的修改直接操作内存，读取操作亦是如此，一定
  是外部可见的

# 如何设置内存属性

> 相关内容可以在 ARMv8 arm 手册 D5.3.3 Attuibute fields in stage 1
> VMSAv8-64 Block and Page descriptors 中找到参考

对于每一个表示内存块(block)的页表项，都有两个属性字段: **lower attr 和 upper attr**.

以下任何类型或者属性的设置都是通过这两个字段完成的。
