---
title: "优秀文章阅读"
date: 2024-02-01T10:00:35+08:00
tags: ["Thinking"]
categories: ["Thinking"]
---

## Cache 伪共享
>原地址：[Cache伪共享](https://mp.weixin.qq.com/s/zeGxBx77TFGtVeMRBVR-Lg)

Cache的操作单位是CacheLine。
当两块内存AB位于同一个CacheLine时，且有两个Cpu核心分别对AB有修改需求，
此时AB都各自被加载到两个Core的Cache中。

{{< figure src="/fakesharing.jpg" width="80%" >}}

伪共享指的是：若其中一个Core对AB进行修改，那另一个Core内的值变不可信，
需要根据**一致性协议**做出调整（文中举了MESI为例），
使得两边内容一致。如果两边修改的比较频繁，就会导致一致动作经常发生，
**这消耗的时间好似没有Cache存在**，具体的时间损耗依据使用的一致性协议决定。

