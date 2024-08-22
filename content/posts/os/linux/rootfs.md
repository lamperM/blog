---
title: "Linux 易混淆的 ramdisk/ramfs/tmpfs/initrd/initramfs/rootfs"
tags: ["Operating System", "Linux"]
categories: ["Operating System"]
date: 2024-08-18T10:51:49+08:00
---

## Rootfs 定义和存在的问题

Linux 启动后用到的文件都存储在“根文件系统”中，也叫 rootfs，里面存放着一些系统程序和用户文件。因此，rootfs 一般是存储在块设备中，比如 UFS、EMMC 等。

所以说，要想访问这些文件，执行这些系统程序需要先 **初始化块设备的驱动程序**，我理解这个过程是很慢的，相当于所有的任务都会被这个驱动初始化给 delay。

- 消耗的时间包括：块设备驱动初始化+文件系统格式初始化
- 我们可能会有一些任务比如说展示开机动画这些，本身是不依赖块设备，但却都堵塞在这。

**为了解决这个问题，先后有两种策略**：

1. initrd
2. initramfs

## “initrd” based on “Ramdisk”

initrd 的提出就是为了解决上述问题，initrd=init ramdisk，本质上属于一个 ramdisk 设备，所以这里需要先对 ramdisk 进行介绍。

ramdisk 是一个用内存模拟的块设备，就和 EMMC、UFS（这俩都属于 SCSI）类似。既然作为块设备，在它之上需要一个文件系统格式（例如 ext4、f2fs）才能使用。
ramdisk 可能有其他的用处，ramdisk 的使用 **不需要那么复杂的设备驱动程序**。

Linux 在加载时，除了内核和设备树文件同时加载了一个带有文件系统格式的镜像到内存中，然后在内核初始化 ramdisk 后，就去把这个镜像挂载到一个特殊的 ramdisk 设备上，也称为 init ramdisk。尽早的进入了用户态，完成一些需要先执行的任务后（/linuxrc ），可以慢慢加载其他物理的块设备，挂载真正的 rootfs， 执行里面的/sbin/init。

## “initramfs” based on “ramfs、tmpfs”

ramdisk 作为一个内存中模拟的块设备，自身占用一部分内存。同时文件系统格式的缓冲区又占用一块内存，一个文件的读写需要先到文件系统的缓冲，合适的时候会同步到块设备的存储区域中。

- 这种方式对于一般的物理块设备是很正常的，但对于 ramdiks 来说，两个缓冲都是在内存，互相拷贝就没什么必要。

于是 linux 在 2.6 引入了一个特殊的文件系统格式：ramfs，这种文件格式的操作不需要下级的物理设备，修改仅仅保存在内存中。

- tmpfs 是在 ramfs 的基础上做了一些优化，本质相同。

initramfs 就是基于 tmpfs 对 initrd 进行优化，省去了创建 ramdisk 的必要性，更加方便，速度也更快。

- 用户引导时直接传入一个 cpio gz 或者 lz4 的压缩文件，内核启动时将其格式化为 tmpfs 文件系统。

## 总结

- ramdisk：使用内存模拟的特殊的块设备，像是 EMMC、UFS 这种
- ramfs、tmpfs：文件系统格式，像是 EXT4、F2FS 这种
- initrd：init ramdisk，一个启动阶段专用的 ramdisk，存放第一级“临时 rootfs”
- initramfs：基于 tmpfs 的、专门用于启动阶段，同样存放第一级“临时 rootfs”
- rootfs：不是一种文件系统格式，而是一堆文件的统称。系统启动后，指那些真正的用户文件和系统程序，一般来说 rootfs 使用的文件系统可能是 EXT4 或 f2fs，底层的块设备是 EMMC 或 UFS。

通过使用 initramfs，Linux 可以逐渐将早期引导功能的执行从内核空间转移到用户空间，为处理复杂的启动要求提供了更可定制和可扩展的环境。

## 参考

- [深入理解 Linux 2.6 的 initramfs 機制](https://xstarcd.github.io/wiki/Linux/ShengRuLiJie_linux_2.6_initramfs.html)
