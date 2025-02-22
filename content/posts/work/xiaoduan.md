---
title: "面试总结：希奥端秋招一面"
tags: ["work"]
categories: ["Job"]
date: 2023-08-28T08:51:49+08:00
draft: true
---

- 岗位: 嵌入式软件开发
- 结果: 未知

# 感悟

1. 礼貌; 面试官大哥非常的有礼貌, 我每次回答他的问题都会得到响应, 即便是没有回答上来也会被告知"没关系".
   结束的时候还说了"谢谢你的时间", 这是我参加了这么多场面试以来听到的第一句, 给人留下很好的印象
2. 专业性; 这次的难度小于地平线, 问的问题比较基础, 但是也契合你简历上写的内容, 总体上来说专业性还是不错


# 回顾

## 手撕代码: 用单链表实现栈的出栈和入栈操作

## NorFlash和NandFlash的区别?

## GICv3相比GICv2有什么提升?
* 结构上面增加了新的期间: Redistributor, 专门负责私有中断, 可以无需经过 Distributor, 减轻其负担
* 新的 "affinity routing" 方案来做中断路由, 可以支持更多处理器
* 支持中断分组, 为了配合ARMv8的异常等级模型
* 新增中断类型: SGI, 软件生成中断
* 新增中断类型: SPI, Shared Peripheral Interrupts  
* 对于CPU interface的寄存器, 可直接使用系统寄存器接口(system register interface)来访问, 比memory-mapped的方式快.


## 你做的内存分配器是什么思路?

参考slab分配器, 做了一些简化.

slab分配器的核心思想是

## ARMv8的页表属性和权限
[ARMv8 内存属性]({{< ref "posts/arch/armv8/memory_attr" >}})

## ARMv8Cache分为几级,分别位于哪里
[ARMv8 cache 相关知识]({{< ref "posts/arch/armv8/cache" >}})
## ARMv8有几个通用寄存器?

拿64位来说, 有31个通用寄存器, x0~x30

## ARMv8分为哪几个异常等级, 分别用作实现什么?
共有四个异常等级: EL0-EL3
- EL3: Secure monitor, 负责安全世界和非安全世界的切换
- EL2: Hypervisior, 实现虚拟化, 管理虚拟机
- EL1: 非安全EL1实现普通OS, 安全EL1跑安全OS, 例如OP-TEE
- EL0: 跑用户态程序

## ARMv8的异常分为哪几类?
- 同步异常: 可细分位指令预取异常/Data Abort/MMU翻译错误等
- IRQ
- FIQ
- System Error


# 反问

## 贵公司该岗位的培养方式是什么？都有哪些方向？
