---
title: "Linux Rootfs 二级加载"
tags: ["Operating System", "Linux"]
categories: ["Operating System"]
date: 2024-08-18T10:51:49+08:00
---

一些常用名次的概念区分：

- ramdisk：使用内存模拟的特殊的块设备，像是 EMMC、UFS 这种
- ramfs、tmpfs：文件系统格式，像是 EXT4、F2FS 这种
- initrd：init ramdisk，一个启动阶段专用的 ramdisk，存放第一级“临时 rootfs”
- initramfs：基于 tmpfs 的、专门用于启动阶段，同样存放第一级“临时 rootfs”
- rootfs：不是一种文件系统格式，而是一堆文件的统称。系统启动后，指那些真正的用户文件和系统程序，一般来说 rootfs 使用的文件系统可能是 EXT4 或 f2fs，底层的块设备是 EMMC 或 UFS。

## Rootfs 定义和存在的问题

Linux 启动后用到的文件都存储在“根文件系统”中，也叫 rootfs，里面存放着所有系统程序和用户文件。因此，rootfs 一般是存储在块设备中，比如 UFS、EMMC、SCSI 等。

所以说，要想访问这些文件，执行这些系统程序需要先 **初始化块设备的驱动程序**，我理解这个过程是很慢的，相当于所有的任务都会被这个驱动初始化给 delay。

- 消耗的时间包括：块设备驱动初始化+文件系统格式初始化
- 我们可能会有一些任务比如说展示开机动画这些，本身是不依赖块设备，但却都堵塞在这。

**为了解决这个问题，先后有两种策略**：

1. initrd(based on ramdisk)
2. initramfs(based on tmpfs)

## “initrd” based on “Ramdisk”

initrd 首先被提出以解决上述问题，initrd=init ramdisk，本质上属于一个 ramdisk 设备，所以这里需要先对 ramdisk 进行介绍。

- ramdisk 是一个用内存模拟的块设备，就和 EMMC、UFS（这俩都属于 SCSI）类似。既然作为块设备，在它之上需要一个文件系统格式（例如 ext4、f2fs）才能使用。
  ramdisk 可能有其他的用处，ramdisk 的使用 **不需要那么复杂的设备驱动程序**。

initrd 的使用方法是：在 Linux 加载时，除了内核和设备树之外，同时加载了一个带有文件系统格式的镜像到内存中（例如.f2fs 格式）。内核初始化完 ramdisk 后，就去把这个镜像挂载到一个特殊的 ramdisk 设备上，也称为 init ramdisk。尽早的进入了用户态，完成一些需要先执行的任务后（/linuxrc ），可以慢慢加载其他物理的块设备，挂载真正的 rootfs， 执行里面的/sbin/init。

## “initramfs” based on “ramfs、tmpfs”

介绍 initramfs 是基于 tmpfs 对 initrd 方式的一种优化，首先要介绍 ramfs/tmpfs：

- ramdisk 作为一个内存中模拟的块设备，自身占用一部分内存。同时文件系统格式的缓冲区又占用一块内存，一个文件的读写需要先到文件系统的缓冲，合适的时候会同步到块设备的存储区域中。

- 这种方式对于一般的物理块设备是很正常的，但对于 ramdiks 来说，两个缓冲都是在内存，互相拷贝就没什么必要。

- 于是 linux 在 2.6 引入了一个**特殊的文件系统格式：ramfs**，这种文件格式的操作**不需要下级的物理设备，修改仅仅保存在内存中。**

- tmpfs 是在 ramfs 的基础上做了一些优化，本质相同。

initramfs 就是基于 tmpfs 对 initrd 进行优化，省去了创建 ramdisk 的必要性，更加方便，速度也更快。

- 用户引导时直接传入一个 cpio gz 或者 lz4 的压缩文件，内核启动时将其格式化为 tmpfs 文件系统。

## 概念总结

系统启动过程的最后一个阶段：挂载根文件系统、执行根文件系统中的 init 程序完成到用户空间的切换。然而根文件系统可能是在不同的硬件设备上，如 SCSI 硬盘、SATA 硬盘、Flash 设备等，后续会出现更多的硬件设备；根文件系统可以是 xfs、ext4、NFS 等不同的文件系统；为了成功挂载根文件系统，内核需要具备相应的设备驱动、文件系统驱动，如果为了兼容所有的根文件系统，将所有相关驱动编译进内核，会增大内核大小，并在实际环境中引入一些无用的驱动。

initramfs/initrd 作为一个过渡文件系统解决了挂载根文件系统的兼容性。

- 因为是过渡的文件系统，一般来说 initramfs/initrd 都比较小，其中仅包含了必要的硬件设备、文件系统驱动以及驱动的加载工具及其运行环境。大量的用户程序文件还是存储在块设备中。
- **initramfs 可以编译进内核也可以作为单独文件由 bootloader 加载入内存**，initrd 只能通过 bootloader 载入。相见 `populate_rootfs()` 函数。
- 在内核初始化的最后阶段，会解压 initramfs 或挂在 initrd，运行其中的 init 程序完成主根文件系统挂载，并执行根文件系统中的 init 程序，完成内核空间到用户空间的切换。

通过使用 initramfs，Linux 可以逐渐将早期引导功能的执行从内核空间转移到用户空间，为处理复杂的启动要求提供了更可定制和可扩展的环境。

## initrd/initramfs 处理代码分析

对 initrd 的处理函数主要有两个：populate_rootfs()和 prepare_namespace()，针对不同的格式，处理情况各不相同。

### populate_rootfs()

到这里有三种情况：编译进内核的 initramfs、uboot 传来的 initramfs 或者 initrd。为了方便区别，我们将 uboot 传来的两种不同格式称为：cpio-initrd(initramfs)和 image-initrd(initrd)。

1. 优先处理 initramfs 被编译到内核的情况，`__initramfs_start` 就是起始地址。如果`__initramfs_end - __initramfs_start`等于 0，那么 unpack_to_rootfs 函数不会做任何事情，直接退出。系统认为该 initrd 不是 initramfs 文件，所以需要到后面去处理。

2. initrd_start 是从 uboot 得到的，如果 uboot 没有传 cpio-initrd 或 image-initrd，或者 `CONFIG_INITRAMFS_FORCE` 使能强制忽略 uboot 传来的，则直接跳转到 (5)。
3. 按照 cpio-initrd 的格式去解包 uboot 传来的，如果 uboot 传来的是 image-initrd，则此步骤会失败。
4. 到这里的情况只有 uboot 传来的 image-initrd 情况，这也是为什么要检查 `CONFIG_BLK_DEV_RAM` （是否支持 ram 作为块设备，即 ramdisk）。这里面的函数就不进去了，大概的行为就是：首先在前面挂载的根目录 rootfs 上创建一个 `/initrd.image` 文件，再把 initrd_start 到 initrd_end 的内容写入到`/initrd.image` 中。剩下的处理过程在 prepare_namespace()。
5. 最后处理的关于 uboot 传来的 cpio-initrd/image-initrd 占用内存释放的过程。`kexec_free_initrd()` 中去做了判断，只释放不在 crashkernel 范围内的内存。我理解如果是内置的 initramfs(gz)或者 cpio-initrd(gz)，都会有解压的过程。会解压出这个=范围。而一般的情况，都会优先被放到 crashkernel 范围中，不会超出，减少释放的时间损耗。

```c
static int __init populate_rootfs(void)
{
	/* Load the built in initramfs */               // (1)
	char *err = unpack_to_rootfs(__initramfs_start, __initramfs_size);
	if (err)
		panic("%s", err); /* Failed to decompress INTERNAL initramfs */

	if (!initrd_start || IS_ENABLED(CONFIG_INITRAMFS_FORCE))  // (2)
		goto done;

	if (IS_ENABLED(CONFIG_BLK_DEV_RAM))
		printk(KERN_INFO "Trying to unpack rootfs image as initramfs...\n");
	else
		printk(KERN_INFO "Unpacking initramfs...\n");

	err = unpack_to_rootfs((char *)initrd_start, initrd_end - initrd_start); // (3)
	if (err) {
#ifdef CONFIG_BLK_DEV_RAM
		populate_initrd_image(err); // (4)
#else
		printk(KERN_EMERG "Initramfs unpacking failed: %s\n", err);
#endif
	}

done:
	/*
	 * If the initrd region is overlapped with crashkernel reserved region,
	 * free only memory that is not part of crashkernel region.
	 */
	if (!do_retain_initrd && initrd_start && !kexec_free_initrd()) // (5)
		free_initrd_mem(initrd_start, initrd_end);
	initrd_start = 0;
	initrd_end = 0;

	flush_delayed_fput();
	return 0;
}
```

### image-initrd 的解包

prepare_namespace()： 对于 image-initrd 来说，上一步 populate_rootfs() 还没有最终完成解包（挂载到/），只是写入到一个文件`/initrd.image` ，剩下的步骤就是在这里完成。

## 参考

- [深入理解 Linux 2.6 的 initramfs 機制](https://xstarcd.github.io/wiki/Linux/ShengRuLiJie_linux_2.6_initramfs.html)

- [【转载】linux2.6 内核 initrd 机制解析 - 玩意儿 - 博客园](https://www.cnblogs.com/cxd2014/p/4470394.html)

- [启动过程分析--initramfs 阶段 - Just Hack Fun](https://www.justhack.fun/posts/e06690ff/)
