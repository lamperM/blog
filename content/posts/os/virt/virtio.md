---
title: "虚拟化：Virtio基础"
tags: ["Operating System"]
categories: ["Operating System"]
date: 2024-04-19T19:28:12+08:00
---

从最近开始接触虚拟化的基础知识，一直不太理解设备虚拟化的理念。然而通过最近对Virtio的了解，可能稍微有一些见解，在这里记录下。

## 设备虚拟化

我喜欢从简单、熟悉的事物中开始理解。串口是相对简单的一个设备，我在日常开发中也经常会用到，所以我想拿串口当作一个例子进行说明Virtio的初衷及原理。

设备虚拟化，实际就是GuestOS(kernel)与VMM之间通信，VMM如何将GuestOS发出的设备请求转换为物理设备的状态转换。


## 为什么选择virtio

virtio 设计的初衷是简化设计、提高性能。

virtio 是一种接口规范，位于VM的前端和VMM的后端都可以随意设计，只要满足约定就能实现设备虚拟化。

站在VM、VMM的角度，互相更换不需要重新思考如何设计设备虚拟的方法，只要双方都支持Virtio，就能相互通信。

站在VMM的角度，如果是VMM自己实现硬件驱动的形式，Virtio能帮你节约大量开发时间、节约工程量。
因为VM可能需要很多设备的驱动（看Linux支持多少驱动），难道作为VMM要为每个驱动都设计映射方案？有那么多类型的串口设备，每个设备的数据寄存器都不同，VMM要知道每个设备寄存器的含义，并转换成真实设备的行为。这个工作量太大了！

## Virtio在QEMU上如何提升性能
为了方便，启用ARM VHE特定，Linux kernel和Hyper都在EL2，避免二者之间的切换。

VM APP想要在串口输出一串字符，未启用virtio时（全虚拟化）：
1. VM kernel: 向串口的数据寄存器写一个字符 ==> trap to Hyper
2. Hyper: 设备请求处理不了，交给QEMU  ==〉 back to Qemu
3. Qemu: 调用putchar打印 ==> syscall to Host kernel
4. Host Kernel: 写到真实的物理串口寄存器上


## 我是一个VMM开发者，如何支持Virtio

## 我是一个Linux驱动开发者，如何支持Virtio

## 我是一个设备厂商，如何支持Virtio

