---
title: "虚拟化：Virtio"
tags: ["Virtualization", "Virtio"]
categories: ["Virtualization"]
date: 2024-04-19T19:28:12+08:00
---

从最近开始接触虚拟化的基础知识，一直不太理解设备虚拟化的理念。然而通过最近对 Virtio 的了解，可能稍微有一些见解，在这里记录下。

## Virtio 是干啥的

虚拟化中，设备的虚拟化是很复杂、很关键的一点，Linux 代码中大量的设备驱动，**如何对这些驱动的行为进行模拟成为了一个很难解决的问题**。
Virtio 提供了一种高性能的设备虚拟化方案。

## 以前的设备虚拟化为什么性能低

最早，虚拟化技术刚刚提出的时候，实现虚拟化的是的方案是--**设备“全虚拟化”**。关键的点是：**VM 不必知道自己运行在 Hypervisor 之上，也不用修改任何的代码**，直接就能实现设备的访问。

实现这种方案的关键是：对所有的设备访问都要 Trap 到 Hypervisor 处理。OS 和设备交互的方式是 MMIO 和中断，以串口举例，我们要发送一个字符到串口中，就需要不断的读 busy 寄存器，直到空闲然后写 data 寄存器。

- 不能直接把设备 MMIO 地址给 VM 操作，因为 hypervisor 之上运行着多个 VM，他们不知道其他人是否在占用设备，会造成冲突。
- 所以唯一的方法就是：**每一次 MMIO 的访问，都 Trap 到 Hypervisor**，它能看到所有 VM 的状态，在合适的时候将这个请求转发给物理设备。中断也是如此，Hypervisor 拦截所有的中断。

**这种做法显然造成了频繁的 Trap**，性能很差！！

{{< figure src="/virtio_1.jpg" width="60%">}}

## Virtio 如何提高性能

Virtio 的设计原则是：放弃一部分设备全虚拟化的优势，VM 得知道自己运行在 Hypervisor 之上。然后，在 VM 上运行“改良过的”设备驱动，来提高性能。

这个改良做的是什么呢？

- Virtio 分为前后端，前端在 VM 中，替换原来的设备驱动。
- 后端在 Hypervisor/HostVM 中，相当于是原来对物理设备实际操作的代码。

原来的全虚拟化，不是每次寄存器操作都要 Trap 吗？现在的方案是，

- 在前后端之间（不论后段在 Hypervisor 还是 HostVM），有一个 ringbuf，两个端口可以分别将数据放到 ringbuf 中，甚至是两个分离的 ringbuf。
- 加以合适的通知机制，在数据准备好后通知后段进行处理，或者后段通知前端，原理是相同的。

这种方案能提高性能主要在于：

1. GuestVM 和直接驱动硬件设备的代理方（可能是 Hypervisor/HostVM）通信不需要每次都 Trap，直接等到某个条件下通过中断告之，去 ringbuf 取就可以了。你可能会问，全虚拟化中，也可以做缓存啊？但问题是，因为 VM 不知道自己运行在虚拟机上，所以缓存不能做成共享内存的形式（或者难做、不通用），而 VirtIO 中的 ringbuf 是完全的共享内存的形式，零拷贝，速度快。

另外，Virtio 前后端分离的设计，也解决了一部分通用性问题。前后端在设备初始化时会握手，只要满足某种约束就行相互配合，可以独立设计实现。

- 站在 Hypervisor/HostVM 的角度，不用对所有的设备驱动都分别做代理判断了，而是划分为了几种类别（Virtio-blk、Virtio-console 等）。不管你是什么设备，只要 VM 有 virtio 驱动，Hypervisor/HostVM 就不用更改，直接支持。

## 几种 Virtio 的经典实现

### 后端在 Hypervisor

- VM 驱动通过 hvc 调用来通知 hyperv
- Hypervisor 通过中断注入来通知 VM

{{< figure src="/virtio_hyp.jpg" caption="" width="60%" >}}

### 后端在 HostVM

- VM 后端是 HostVM 里的一个**用户线程**，专门处理前端请求
- VM 如何通知 HostVM，我理解只有 Host 能主动发起通知，难道还是 hvc 吗？
- VM 和 HostVM 之间的 virtqueue 通过共享内存来实现，可以承载于 uio 设备
- HostVM 专门用于处理请求的，所以它可以直接操作硬件，Hypervisor 不需要 Trap 吧

TODO： 展示一个完整的数据流程，假设 GusetVM 要操作硬件，比如说写入数据

1. 首先 GuestVM 用户程序调用驱动 write()，
2. 驱动将数据写入到 virtqueue 里，hvc 通知 virtio 后端？
3. hypervisor 负责通知 Hostvm 处理 virtio 请求，中断注入到 Hostvm
4. HostVM 在收到 virtio 请求中断时，调度 EL0 的后端线程，在此之前后端线程已经打开了/dev 下的 virtio 设备，所以要做的就是判断 guestVM 此次是写哪个设备，做一下代理转换。
5. 目的地（设备）决定好后，向 virtio device 写入数据即可。
6. 写完之后，怎么通知 hypervisor，再最终通知到 guestVM？

{{< figure src="/virtio_host.jpg" caption="" width="60%" >}}

## 进一步提升性能：Vhost

在 HostVM 作为后端的模式下，因为 virtio 后端原本是用户线程，在准备处理请求时还得先陷入到 HostVM Kernel 中，请求完毕还得返回 HostVM EL0.

Vhost 就是把后端放在 HostVM 的内核中，避免 HostVM 处理请求时多次的内核陷入。

[vhost：一种 virtio 高性能的后端驱动实现 - bakari - 博客园](https://www.cnblogs.com/bakari/p/8341133.html)

## Virtio QEMU 进一步提升性能：VHE

VHE 适用于 QEMU KVM 吧。

启用 ARM VHE 特性，VM Kernel 运行在 EL2

## 我是一个 VMM 开发者，如何支持 Virtio

## 我是一个 Linux 驱动开发者，如何支持 Virtio
