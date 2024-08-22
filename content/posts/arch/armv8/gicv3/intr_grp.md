---
title: "ARMv8 中断管理(4): 中断分组"
tags: ["armv8", "gic"]
categories: ["Architecture"]
date: 2024-08-02T23:51:49+08:00
---

{{< figure src="/interrupt_grp.jpg" caption="" attr="" attrlink="" width="70%">}}

- NS Group 1： 想给 REE 处理的中断
- S Group 1：想给 TEE 处理的中断
- Group 0：想给 ATF 处理的中断

## IRQ 和 FIQ 的含义

FIQ 并不是以前意义上的快速中断，而是 Forward 中断，需要被转发的中断。IRQ 和 FIQ 的优先级是相同的。

## 中断分组产生的原因？

## 怎么就能做到上述的约定？

根据 3 种当前环境（REE、TEE、ATF）和 3 种中断分组，总共有 9 种可能得情况，依次进行分析。ATF 在这里也就等价于 BL31。

{{< notice info  >}}
以下所有的 Case 描述都基于一个前提：

1. BL31 非安全上下文中 SCR_EL3.IRQ=0,SCR_EL3.FIQ=1
2. 安全上下文中 SCR_EL3.IRQ=0,SCR_EL3.FIQ=0。

这是 BL31 初始化时的默认配置，2022 年前大部分厂家也是用的这种配置。但是现在随着功能的不断演进，已经有的厂家开始修改这种路由规则了。但是我感觉，只要你了解底层原理，也能快速地理解他们这样配置的理由。

TODO：EHF？
{{< /notice >}}

### Case 1：REE 时收到 NSG1

{{< figure src="/gic_case1.jpg" caption="" attr="" attrlink="" width="70%">}}

这是最简单的情况：

- 因为 REE 处于 NS 状态，根据 GICv3 上述的真值表，NSG1 会触发 IRQ。
- 由于此时 SCR_EL3.IRQ=0，所以 IRQ 并不会 Trap 到 EL3，就是默认的情况在 Linux Kernel 处理。

### Case 2：TEE 收到 NSG1

{{< figure src="/gic_case2.jpg" caption="" attr="" attrlink="" width="70%">}}

这种情况就相对来说比较复杂，但是很经典：

1. 由于当前在 S 状态，NSG1 会触发为 FIQ
2. 大部分的 TEE 对 FIQ 没有做任何操作，**特别是没有 ack 这个中断**，直接执行 smc 下去。
3. 到了 BL31，切换安全状态，进入 REE。（猜测由于是 SMC 懈怠的参数识别了这种情况？）
4. 由于中断没有 ack，所以还处于 pendding 状态，但切换到 NS 状态后，触发的就是 IRQ 了。
5. 在 REE linux kernel 处理后，回到 BL31，准备返回 TEE。（猜测是由于设置了某种标志才知道 IRQ 处理完是要返回 BL31 的）
6. BL31 返回 TEE 继续执行。

### Case 3：BL31 收到 NSG1

{{< figure src="/gic_case3.jpg" caption="" attr="" attrlink="" width="70%">}}

这种情况其实最终会转化成以上两种：
BL31 有两种 context，安全和非安全，NSG1 在 EL3 会触发为 FIQ，两种状态下默认 FIQ 都是被 MASK 的，所以根据你之前是从哪里陷入的就会再哪里去处理这个 NSG1 中断。

### Case 4：TEE 收到 SG1

也是比较简单的情况，当前是 S 状态，收到 SG1，触发 IRQ，直接在 TEE 里被处理，不涉及异常等级的切换。

### Case 5：REE 收到 SG1

{{< figure src="/gic_case5.jpg" caption="" attr="" attrlink="" width="70%">}}

1. 当前执行环境为 NS，SG1 被触发为 FIQ。因为 SCR_EL3.FIQ=1，所以中断直接到 BL31 FIQ handler。
2. BL31 实际不做处理，只是将 ELR 设置成 TEE 的处理函数，进行异常返回。（为什么没有在 TEE 重新被触发，而是在 EL3 被触发，因为 BL31 切换到 TEE 时就 MASK 所有的中断，所以用的配置 ELR 的方式进入异常处理函数）
3. TEE 处理完成后，返回 BL31
4. BL31 返回 REE

### Case 6：ATF 收到 SG1

和 ATF 收到 NSG1 情况类似，最后都是转成上面的两种 case，因为 BL31 中无论是 NSG1 还是 SG1 触发的都是 FIQ，而 FIQ 被 MASK，所以最后怎么处理还是看 BL31 返回时到 REE 还是 TEE。

### Case 7：TEE 收到 G0

{{< figure src="/gic_case7.jpg" caption="" attr="" attrlink="" width="70%">}}

1. TEE 属于 S 状态，G0 被触发为 FIQ。因为 SCR.FIQ=0，所以 FIQ 在 TEE 触发。
2. TEE 不会对 FIQ 做任何处理，也不会 ack 中断，直接 smc 进入 BL31。
3. 这个 BL31 不是 FIQ 的处理函数，而是一个 SMC 处理函数，因为此时中断是 MASK 的，所以不会在 BL31 再次出发。BL31 的做法是切换安全状态，回到 REE。
4. 在 REE，G0 会再次触发，根据真值表来看仍然是 FIQ，而此时因为 SCR.FIQ=1，所以 BL31 可以直接进入中断处理函数。

### Case 8：REE 收到 G0

比较简单，REE 时 G0 触发为 FIQ，因为 SCR_EL3.FIQ=1，所以中断在 BL31 触发，直接被处理。再返回 REE。

### Case 9：BL31 收到 G0

默认 BL31 屏蔽，所以不会被触发。一直处于 pending 状态，直到返回 REE 或者 TEE，就又回到了 Case7 和 Case8 的情况。

TODO：EHF？
