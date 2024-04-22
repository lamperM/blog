---
title: "Linux cmdline 配置"
date: 2024-04-16T10:30:35+08:00
tags: [Linux]
---
内核启动时会打印当前生效的 cmdline。Command Line 相当于外部传给 Linux 内核的参数，内核针对他们做相应处理，并打印无法是被的参数。

如何看当前的cmdline：
1. 内核启动日志会输出
    ```
    [    0.000000] Kernel command line: earlycon rw rdinit=/linuxrc root=/dev/vda nokaslr
    [    0.000000] Unknown command line parameters: nokaslr
    ```
2. `cat /proc/cmdline/`

用户有几种方式来注入cmdline：
- 设备树 bootargs。 linuxkernel的设备树是QEMU生成的，实际就是用的启动参数 `--append`。

    ```bash
    qemu-system-aarch64 \
        -nographic -machine virt,secure=on \
        -cpu cortex-a53 -smp 2 -m 4G \
        -d guest_errors,unimp \
        -gdb tcp::1234 \
        -bios ./arm-trusted-firmware/build/qemu/debug/bl1.bin \
        -kernel ./wupeng/linux/arch/arm64/boot/Image \
        -initrd ./rootfs.cpio.gz \
        -append "earlycon rw rdinit=/linuxrc nokaslr" \                                                                  
        -serial mon:stdio \
        -semihosting-config enable=on,target=native
    ```


- uboot boot_args

    ```
    setenv bootargs 'init=xx'
    ```


- Linux menuconfig

{{< figure src="/cmdline_menuconfig.png" width="80%" >}}

优先级：设备树 > uboot > linux kconfig