---
title: "Qemu 启动 Linux Kernel(Arm64)"
tags: ["Qemu"]
categories: ["Qemu"]
date: 2024-01-04T19:28:12+08:00
---

## 最终效果

用的环境和各个软件版本为：

- Qemu: 8.1.50 (qemu-system-aarch64 -M virt)
- linux-4.9.1
- u-boot-2023.10
- busybox-1.34.0

经过一番折腾，还是没有成功 Qemu+Uboot 来引导 linux 内核，
因为 virt 板级不支持`-sd`参数，主要的折腾过程见下。
但理论上也可以，只是后面发现没啥必要，用`-kernel`也能完成目前需求。
`-kernel`形式下功能没问题，有 Rootfs，可以在 Guest 中读写。

## 准备 Linux 内核镜像

### 下载

- [Linux 内核版本历史 - 维基百科](https://zh.wikipedia.org/wiki/Linux%E5%86%85%E6%A0%B8%E7%89%88%E6%9C%AC%E5%8E%86%E5%8F%B2)
- [上海交通大学镜像站](http://ftp.sjtu.edu.cn/sites/ftp.kernel.org/pub/linux/kernel/)

### 编译

Linux kernel 使用 make 来构建，可以键入`make help`查看支持的命令：

```
Cleaning targets:
  clean           - Remove most generated files but keep the config and
                    enough build support to build external modules
  mrproper        - Remove all generated files + config + various backup files
  distclean       - mrproper + remove editor backup and patch files

Configuration targets:
  config          - Update current config utilising a line-oriented program
  nconfig         - Update current config utilising a ncurses menu based
                    program
  menuconfig      - Update current config utilising a menu based program
  xconfig         - Update current config utilising a Qt based front-end
  gconfig         - Update current config utilising a GTK+ based front-end
```

### 不同 Linux 内核镜像的区别

#### vmlinux

vmlinux 是可引导的、未压缩、可压缩的内核镜像，vm 代表 Virtual Memory。
（表示 Linux 支持虚拟内存，因此得名 vm）它是由用户对内核源码编译得到，
实质是 elf 格式的文件.也就是说 vmlinux 是编译出来的最原始的内核文件，未压缩。

{{<figure src="/vmlinux.jpg" width="80%" >}}

#### vmlinuz

vmlinuz 是可执行 的 Linux 内核，它位于/boot/vmlinuz，
它一般是一个软链接，比如是 vmlinuz-3.13.0-32-generic 的软链接。
vmlinuz 是 vmlinux 的压缩文件。vmlinuz 的建立有两种方式。
一是编译内核时通过“make zImage”创建，二是内核编译时通过命令 make bzImage 创建。

#### Image

Image 是经过 objcopy 处理的只包含二进制数据的内核代码，它已经不是 elf 格式了，但这种格式的内核镜像还没有经过压缩.

#### zImage

zImage 是 ARM linux 常用的一种压缩镜像文件，它是由 vmlinux 经过 objcopy ，
objcopy 实现由 vmlinux 的 elf 文件拷贝成纯二进制数据文件加上解压代码经 gzip 压缩而成，
命令格式是#make zImage.这种格式的 Linux 镜像文件多存放在 NAND 上。
适用于小内核的情况，它的存在是为了向后的兼容性。

{{<figure src="/zImage.jpg" width="80%" >}}

#### bzImage

bzImage 不是用 bzip2 压缩的，bz 表示 big zImage,其格式与 zImage 类似，
但采用了不同的压缩算法，注意，bzImage 的压缩率更高 ，是压缩的内核映像。

zImage/bzImage：它们不仅是一个压缩文件，而且在这两个文件的开头部分内嵌有解压缩代码。
两者的不同之处在于，老的 zImage 解压缩内核到低端内存(第一个 640K)，
bzImage 解压缩内核到高端内存(1M 以上)。如果内核比较小，那么可以采用 zImage 或 bzImage 之一，
两种方式引导的系统运行时是相同的。大的内核采用 bzImage，不能采用 zImage。

#### uImage

uImage 是 uboot 专用的镜像文件，它是在 zImage 之前加上一个长度为 0x40 的头信息(tag)（也就是说 uImage 是一个二进制文件），在头信息内说明了该镜像文件的类型、加载 位置、生成时间、大小等信息.换句话说，若直接从 uImage 的 0x40 位置开始执行，则 zImage 和 uImage 没有任何区别.命令格式是#make uImage.这种格式的 Linux 镜像文件多存放在 NAND 上.

{{<figure src="/uImage.jpg" width="80%" >}}

{{< notice info  >}}
如何生成 uImage？

在 uboot 的/tools 目录下寻找 mkimage 文件，把其 copy 到系统/usr/local/bin 目录下，
这样就完成制作工具。然后在内核目录下运行 make uImage，如果成功，
便可以在 arch/arm/boot/目录下发现 uImage 文件，其大小比 zImage 多 64 个字节。

由于 bootloader 一般要占用 0x0 地址，所以，uImage 相比 zImage 的好处就是可以和 bootloader 共存。
其实就是一个自动跟手动的区别,有了 uImage 头部的描述,u-boot 就知道对应 Image 的信息,
如果没有头部则需要自己手动去搞那些参数。
{{< /notice >}}

### 内核编译脚本

编译完成后，用--kerenel 应该能够执行，因为目前没有 rootfs，所以会 panic 进不了 shell。

```shell
# get linux source code
wget http://ftp.sjtu.edu.cn/sites/ftp.kernel.org/pub/linux/kernel/
# extract
tar xvf linux-4.12.1.tar.gz
# enter dir
cd linux-4.12.1/

# generate .config
make ARCH=arm64 CROSS_COMPILE=aarch64-none-linux-gnu- defconfig
make ARCH=arm64 CROSS_COMPILE=aarch64-none-linux-gnu- menuconfig
# compile
make ARCH=arm64 CROSS_COMPILE=aarch64-none-linux-gnu- Image -j16
```

{{< notice warning >}}
亲测 Wsl2 编译 Linux 内核时不能用`-j`而不手动指定任务数，会使 Wsl 爆内存。
{{< /notice >}}

## 构建根文件系统

根文件系统有两种方式传递给 Qemu:

1. 通过在-append 参数执行 root 的设备名称，hdx 指定的可以是任意格式，ext4、raw
2. 通过使用—initrd 参数指定 inital ramdisk 进行加载，必须是 linux 能识别的 ramfs 格式(cpio+gzip 可以）

{{< notice info  >}}
--append 中的 root=/dev/vdb 和 -hda -hdb 有什么关系？

如果说只有一个磁盘文件，那么传递-hda 还是 hdb 都会在启动时映射到/dev/vda,
只有当同时传入多个-hda 和 hdb 时，这是通过 root=来指定 rootfs 是哪个。
{{< /notice >}}

### 源码下载

我使用 Busybox 构建, 下载源码时可能比较慢, 暂时没有发现国内镜像站。

```shell
# Download busybox source code
wget https://busybox.net/downloads/busybox-1.35.0.tar.bz2
```

### 编译

Compile and install to `_install/`.
**Change to static build.**

```sh
make ARCH=arm64 CROSS_COMPILE=aarch64-none-linux-gnu- menuconfig
make ARCH=arm64 CROSS_COMPILE=aarch64-none-linux-gnu-  -j
make ARCH=arm64 CROSS_COMPILE=aarch64-none-linux-gnu- install
```

### 适配`-initrd=`方式

Create rootfs

```sh
cd ../
# Create null rootfs, `also can be done by `dd` command
# TODO: different from initrd and initramfs
mkdir myinitramfs && cd myinitramfs
# Copy target files from busybox just compiled
cp -r ../busybox-1.34.0/_install/* .
# Create other necessary dirs
mkdir proc sys dev etc lib

# Copy lib/ from cross compiler
# TODO: No need now!
```

Continue configging rootfs: make init script rcS.

fstab 是另一个需要创建的文件，
fstab 在 Linux 开机以后自动配置哪些需要自动挂载的分区，
这样在 rcS 中调用`mount -a`将这些分区进行挂载。

```sh
# Config /etc/
mkdir etc/init.d
touch etc/init.d/rcS etc/fstab
chmod +x etc/init.d/rcS
```

将以下内容赋值到 rcS:

```shell
#!/bin/sh
mount -a
/sbin/mdev -s
mount -a
```

将以下内容赋值到 fstab:

```sh
#device  mount-point    type     options   dump   fsck order
proc /proc proc defaults 0 0
tmpfs /tmp tmpfs defaults 0 0
sysfs /sys sysfs defaults 0 0
tmpfs /dev tmpfs defaults 0 0
debugfs /sys/kernel/debug debugfs defaults 0 0
tracefs /sys/kernel/tracing tracefs defaults        0       0
```

创建 inittab 文件:

打包 initrd，此时的目录为:`/home/loo/linux-qemu/myinitramfs`

```sh
find . | cpio -o -H newc | gzip -c > ../myinitramfs.cpio.gz
```

启动脚本如下，**这种方式的缺点是修改仅在内存中，重启后失效**。

```sh
#!/bin/bash
qemu-system-aarch64 \
  -machine virt,gic-version=3,its=off,secure=on,virtualization=on \
  -cpu cortex-a53 -smp 4 -m 2G \
  -kernel ./linux-4.9.1/arch/arm64/boot/Image \
  -initrd ./myinitramfs.cpio.gz \
  -append "rw rdinit=/linuxrc" \
  -serial mon:stdio \
  -gdb tcp::1234 \
  -nographic \
  $1
```

### 适配`-append="root=/dev/vda`方式

因为通过-inird=指定的方法不能将修改保存到本地，
所以还得是借助非 ramdisk 的方式来实现根文件系统。

```sh
dd if=/dev/zero of=rootfs.ext4.img bs=1M count=512
mkfs.ext4 rootfs.ext4.img

sudo mount rootfs.ext4.img ./mnt-tmp
cd ./mnt-tmp

# 根文件系统的内容都一样，这个可以直接拷贝
cp ../myinitramfs/* .

cd ..
sudo umount ./mnt-tmp
```

启动脚本为，在 Guest 中的修改能够写回根文件系统，但是注意等待写回之后再退出 Qemu，
否则可能造成在内存中缓存的情况。后续可以查资料关闭文件在内存中的缓存。

```sh
#!/bin/bash                                                                                    qemu-system-aarch64 \
  -machine virt,gic-version=3,its=off,secure=on,virtualization=on \
  -cpu cortex-a53 -smp 4 -m 2G \
  -kernel ./linux-4.9.1/arch/arm64/boot/Image \
  -hda ./rootfs.ext4.img\
  -append "root=/dev/vda rw rdinit=/linuxrc" \
  -serial mon:stdio \
  -gdb tcp::1234 \
  -nographic \
  $1
```

## 准备 Uboot（未完成）

Image 不是内核的二进制文件吗？怎么可以使用 Qemu `--kernel`引导呢？
其实在 Qemu 内部有一个 Bios，我们当然想尽可能接近真实的工作环境，
所以还是能用 Uboot 最好。

### 源码下载

- https://ftp.denx.de/pub/u-boot/

### 编译

```sh
# 不能用 make ARCH= 来指定，编译会报错，详见
# https://forums.raspberrypi.com/viewtopic.php?t=345377
export ARCH=arm64
export CROSS_COMPILE=aarch64-none-linux-gnu-
make  qemu_arm64_defconfig
make -j
```

### 失败的折腾

Uboot 编译后好，先尝试直接启动 Uboot，没问题

```sh
#!/bin/bash
qemu-system-aarch64 \
  -machine virt\
  -cpu cortex-a53 -smp 4 -m 2G \
  -bios ./u-boot-2023.10/u-boot.bin \
  -serial mon:stdio \
  -gdb tcp::1234 \
  -nographic \
  $1
```

{{< notice warning  >}}
启动参数不能带`virt,secure=on`，会导致同步异常 uboot 卡死。
{{< /notice >}}

{{< notice info >}}

{{< /notice >}}

接下来就是结合 Uboot 和 linux kernel，最终和真实开发一样的流程：
内核镜像、设备树、根文件系统放在 sd 卡中，当 Uboot 启动后将其加载到内存，并引导。

#### 制作 Sd 卡

在 uboot 的 menuconfig 中加入 mmc 的支持:

```
 Symbol: MMC [=y]                                                                       │
  │ Type  : bool                                                                           │
  │ Prompt: MMC/SD/SDIO card support

 Symbol: CMD_MMC [=n]                                                                   │
  │ Type  : bool                                                                           │
  │ Prompt: mmc

```

启动时会报错，该 model 不支持 if=sd,bus=0....,

> "-sd sd.img" is shorthand for "-drive if=sd,index=0,file=sd.img". Like
> all -drive (except for if=none), it requests the board to create a
> suitable device. Boards act on some requests, and ignore others.
> mpc8544ds ignores if=sd.

后续： virt 开发板不支持-sd 选项，但是 raspi3b 支持，

用`-hda`传入镜像可以，在 uboot 里用 `virtio ls` 命令可以看到，后面觉得算了
没必要去折腾这些，用`--kernel` 也还行。

## 制作HOST共享目录

QEMU8.1默认支持9p virtio文件系统，实现HOST与GUEST共享目录，
在调试时比添加硬盘（-hda）更加方便：
1. host中创建 `share/` 目录
2. qemu启动参数指定
```bash
-fsdev local,security_model=passthrough,id=fsdev0,path=./share \
-device virtio-9p-pci,id=fs0,fsdev=fsdev0,mount_tag=hostshare \
```
3. 启动guest linux后，在命令行中挂载9p文件系统
```bash
mkdir /mnt/share
mount -t 9p -o trans=virtio,version=9p2000.L hostshare /mnt/share
```

{{< notice info >}}
添加到启动脚本中，自动挂载。
```bash


```
{{< /notice >}}

## 增加 perf

**perf**已经集成到了 Linux 主分支中，源码的位置在`tools/perf`

使用 perf 的基础功能不需要修改内核配置文件，但是貌似有些功能比如说 function Trace 是需要的，
目前没有用到，所以之后再来补充吧。

```sh
cd tools/perf
make clean
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- LDFLAGS=-static WERROR=0
```

{{< notice warning >}}
这里换了编译器，可能是之前用的编译器版本太新了，编译会出错。
目前编译成功的版本是：
`gcc-arm-8.3-2019.03-x86_64-aarch64-linux-gnu`
{{< /notice >}}

编译成功后，在当前目录下会有静态编译的 perf 可执行程序，移动到 rootfs 中就能直接使用了。

```shell
~/linux-qemu/linux-4.12.1/tools/perf $ file perf
perf: ELF 64-bit LSB executable, ARM aarch64, version 1 (GNU/Linux), statically linked, for GNU/Linux 3.7.0, with debug_info, not stripped
```

## 试用 KGDB 调试内核

## 增加 strace

下载源码：[Releases · strace/strace (github.com)](https://github.com/strace/strace/releases)

编译&&安装:

```sh
mkdir build && cd build
../configure --host=aarch64-none-linux-gnu --prefix=$HOME/linux-qemu/strace-6.0/_install --enable-mpers=no

# 编译为静态链接方式
make LDFLAGS+='-static -pthread' -j16

# 拷贝到 _install 目录
make install
```
