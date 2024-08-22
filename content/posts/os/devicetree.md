---
title: "设备树 Device Tree"
tags: ["Operating System"]
categories: ["Operating System"]
date: 2024-07-26T19:28:12+08:00
---

设备树是一种描述硬件信息的文件格式。

设备树实现了驱动代码与硬件设备信息的分离，驱动代码可以有很多，但是不一定每个设备都要用。在设备树中配置了启用哪些设备，和一些关于设备的配置，比如说中断号是多少、内存地址是哪里。就能与驱动代码分离开，即便是不用板子的外设，只要型号一致就能用同一份驱动。其他的尽量做成可配置的。

## 设备树的加载流程

Uboot将设备树的地址传给 Linux内核

```shell
bootm <uImage_addr> <initrd_addr> <dtb_addr>
```


`start_kernel`是内核进入C环境调用的第一个函数，不久后便会调用dtb的解析。

```c
start_kernel()
    => setup_arch(&command_line);
        => setup_machine_fdt(__fdt_pointer);
```

__fdt_pointer 是在汇编中赋值的，存储由Uboot传来的fdt地址。
```asm
/*
 * Preserve the arguments passed by the bootloader in x0 .. x3
 */
SYM_CODE_START_LOCAL(preserve_boot_args)
	mov	x21, x0				// x21=FDT

	adr_l	x0, boot_args			// record the contents of
	stp	x21, x1, [x0]			// x0 .. x3 at kernel entry
	stp	x2, x3, [x0, #16]

	dmb	sy				// needed before dc ivac with
						// MMU off

	mov	x1, #0x20			// 4 x 8 bytes
	b	__inval_dcache_area		// tail call
SYM_CODE_END(preserve_boot_args)

...
str_l	x21, __fdt_pointer, x5		// Save FDT pointer

```

TBD：继续分析 `setup_machine_fdt()` ？


## 设备树调试

查看原始的dtb文件

```sh 
ls /sys/firmware/fdt 
hexdump -C /sys/firmware/fdt
```

查看设备树信息。以目录结构程现的dtb文件, 根节点对应base目录, 每一个节点对应一个目录, 每一个属性对应一个文件
/proc/device-tree 是链接文件, 指向 /sys/firmware/devicetree/base

```shell
ls /sys/firmware/devicetree
ls /proc/device-tree
```

查看所有硬件信息。系统中所有的platform_device, 有来自设备树的, 也有来有.c文件中注册的。

```shell
ls /sys/devices/platform
```