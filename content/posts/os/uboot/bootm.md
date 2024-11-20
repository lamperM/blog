---
title: "UBoot: boom 命令的执行流"
tags: ["Operating System", "UBoot"]
categories: ["Operatsing System"]
date: 2024-09-06T19:28:12+08:00
---

```c
// bootm 命令的入口
int do_bootm(cmd_tbl_t *cmdtp, int flag, int argc, char * const argv[])
    ==> // CONFIG_NEEDS_MANUAL_RELOC 这个选项没开，暂时不研究

    ==> if (有其他的子命令，除了':','#') // 这里我们也一般不加
        ==> do_bootm_subcommand()

    ==> state |= BOOTM_STATE_FINDOTHER // 这个 flag 是一定set的
    ==> state |= BOOTM_STATE_RAMDISK   // ifdef CONFIG_SYS_BOOT_RAMDISK_HIGH，
                                       // 这个是比较关键的，设置 ramdisk 搬移到高地址的 FLAG
    ==> return do_bootm_states(cmdtp, flag, argc, argv, state);

do_bootm_states()
    ==> bootm_find_other
        ==> bootm_find_images
            // 设置 images.rd_start, images.rd_end
            ==> boot_get_ramdisk(argc, argv, &images, IH_INITRD_ARCH,
			                        &images.rd_start, &images.rd_end);

    // 如果启用了 重定位 的config，就会做RAMDISK的搬运。
    // 搬运完成后，设置相应的环境变量，最后会传递给内核，内核启动时就从这里找
    ==> boot_ramdisk_high() // ifdef CONFIG_SYS_BOOT_RAMDISK_HIGH，
    ==> env_set_hex("initrd_start", images->initrd_start);
	    env_set_hex("initrd_end", images->initrd_end);

    ==> boot_selected_os() // never return


// 这个函数的功能是 赋值 images.rd_start，即ramdisk的位置
int boot_get_ramdisk(int argc, char * const argv[], bootm_headers_t *images,
		uint8_t arch, ulong *rd_start, ulong *rd_end)
{
    // 他的逻辑好像是，kernel 后面就是 ramdisk 镜像
    ==> android_image_get_ramdisk((void *)images->os.start, &rd_data, &rd_len);
        ==> *rd_data = (unsigned long)hdr;
        ==> *rd_data += hdr->page_size;
        ==> *rd_data += ALIGN(hdr->kernel_size, hdr->page_size);
        ==> *rd_len = hdr->ramdisk_size;

}

// 这个函数的功能是赋值 images.os 的各个成员
int bootm_find_os(cmd_tbl_t *cmdtp, int flag, int argc,
			 char * const argv[])
    // 关键成员的来源，images.os.start 就是 Kernel Image Header的起始地址，
    // 会被用于加载
    ==> os_hdr = boot_get_kernel(&images.os.image_start)
    ==> images.os.start = map_to_sysmem(os_hdr);


/**
 * boot_get_kernel - find kernel image
 * @os_data: pointer to a ulong variable, will hold os data start address
 * @os_len: pointer to a ulong variable, will hold os data length
 *
 * boot_get_kernel() tries to find a kernel image, verifies its integrity
 * and locates kernel data. 得到 Kernel 镜像的地址
 *
 * returns:
 *     pointer to image header if valid image was found, plus kernel start
 *     address and length, otherwise NULL
 */
void *boot_get_kernel(cmd_tbl_t *cmdtp, int flag, int argc,
				   char * const argv[], bootm_headers_t *images,
				   ulong *os_data, ulong *os_len)
// image header 从 bootm 的第一个参数拿
// 校验之后，赋值给 os_data，以真正的镜像起始地址



// 这个函数比较重要，单独拿出来说一下
// 函数的功能就是加载ramdisk到高地址
// 参数 rd_data: ramdisk 现在所在的内存地址
// 参数 initrd_start: ramdisk 可能被relocation后的新 高地址
boot_ramdisk_high(ulong rd_data, ulong rd_len,
		  ulong *initrd_start, ulong *initrd_end)
    // initrd_high 的含义是：重定位的最高地址。
    // 环境变量里可以设置成 ~0，代表禁止 重定位，这是从运行参数中禁止重定位的一种方法！
    ==> if env_get("initrd_high")
            if (initrd_high 从 env里取 == ~0) // 设置 禁止重定位 标志
    ==> else initrd_high = bootm 能访问的地址上限

    // 如果允许重定位，则动态分配一个地址，进行内存拷贝
    ==> if !禁止重定位
        // 满足 initrd_high 的限制条件下，动态申请一个地址，
        ==> *initrd_start = (ulong)lmb_alloc_base(lmb, rd_len, 0x1000, initrd_high);
        // 重定位 的内存拷贝
        ==> memmove_wd((void *)*initrd_start, (void *)rd_data, rd_len, CHUNKSZ);


```

TODO：打包的时候，在哪里确定 RAMDISK 紧跟在 kernel 的后面？
