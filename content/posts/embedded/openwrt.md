---
title: "openwrt 开发日记"
tags: [tools]
date: 2023-08-05T19:28:12+08:00
---


# 构建 openWRT

> 我在此步骤失败了，后面项目没有依赖完整的编译过程，
> 所以可能对你不构成参考


过程可参考[官方教程](https://openwrt.org/docs/guide-developer/toolchain/use-buildsystem), 
编译过程非常长，使用到的工具非常多，这里提供两个优化的思路:

**提前安装本地依赖**，忘了`./scripts/feeds update -a`还是`./scripts/feeds install -a`时需要检查系统的各种依赖, 可以提前统一安装一波.

```sh
sudo apt install g++
sudo apt install libncurses5-dev
sudo apt install zlib1g-dev
sudo apt install bison
sudo apt install flex
sudo apt install unzip
sudo apt install autoconf
sudo apt install gawk
sudo apt install make
sudo apt install gettext
sudo apt install gcc
sudo apt install binutils
sudo apt install patch
sudo apt install bzip2
sudo apt install libz-dev
sudo apt install asciidoc
sudo apt install subversion
sudo apt install python
sudo apt install git
```

**提前下载dl**, dl是默认在编译时下载的一些工具源码, 你可以将他们提前下载好
放到`dl/`下, 即可省去下载的时间, 特别当你不能翻墙时.

就像[这个仓库](https://gitee.com/whilewell/openwrt-dl)这样, 但是它里面
的软件版本可能比较老了而且有的软件是缺失的, 以后如果真的要自己编译, 
需要查makefile去替换真正依赖的软件和其对应的版本.

最终还是因为编译某个模块失败, 且编译时间太长(连交叉编译工具链都需要现场编译)
, 导致排查困难, 没有编译成功。好在后面也没有直接依赖编译的结果。

## 编译的tips
1. `make V=99` build with verbose

# ptgen

正如上面所言, 我最终没有完整的编译成功。但其中的一个小工具`ptgen`是我
需要用的到，它在过程中被编译出来了，相对独立些。

`ptgen`是OpenWRT开发的一个用来生成gpt分区表的工具，创建的sdcard镜像，
只有配合分区表才能正确的被bootrom加载起来。

## 使用方法及参数
`ptgen`使用的参数说明:
```sh

ptgen [-v] -h <heads> -s <sectors> -o <outputfile> [-a 0..4] [-l <align kB>] [[-t <type>] -p <size>...]

-v: 指定是否打印调试信息,可选
-h: 指定起始磁头号
-s: 指定起始扇区号
-o: 指定输出文件名
-a: 指定激活分区为哪个, 可选
-l:  指定多少KiB对齐,可选，这个参数会决定每个分区的偏移扇区号，非常重要
-t:  指定文件系统分区标志类型值,是0x83指linux,0x0b指Win95 FAT32,可选
-p  指定分区大小,可选
```

## ptgen使用案例
这里贴出我使用ptgen创建一个BananaPi M2 Ultra可以识别的sd卡镜像文件，
对bootfs进行挂载可放入一些文件，在uboot下能访问。

```sh
#!/bin/bash

if [ -f "sd.img" ]; then
        echo "warning: sd.img already exist, do nothing"
        exit
fi

if [ -f "bootfs.ext4" ]; then
        echo "warning: bootfs.ext4 already exist, do nothing"
        exit
fi

BOOTFS_SIZE=16M
# make ext4 fs which including kernel.bin
dd bs="$BOOTFS_SIZE" if=/dev/zero of=bootfs.ext4 count=1
sudo mkfs.ext4 bootfs.ext4
[ -d "./mnt" ] || mkdir ./mnt
sudo mount -o loop bootfs.ext4 ./mnt
sudo cp kernel.bin ./mnt
sudo umount ./mnt


# create empty image
dd bs=32M if=/dev/zero of=sd.img  count=1


# generate parition table
# -t 0xc: FAT32
# -t 0x83: ext4
set $(./ptgen -o sd.img -h 4 -s 63 -l 1024 -t 0x83 -p "$BOOTFS_SIZE")


BOOTFS_OFFSET="$(($1 / 512))"
# write in uboot and bootfs
dd bs=1024 if=u-boot-sunxi-with-spl.bin of=sd.img  seek=8 conv=notrunc
dd bs=512 if=bootfs.ext4 of=sd.img seek="$BOOTFS_OFFSET" conv=notrunc


echo -e "\n\nsd.img is ok"
```


我正是依赖这个工具生成了最终的镜像而已，其他的模块其实并不是特别需要。
本来u-boot也是必须的，但是后面发现我用的硬件(BananaPi M2 Ultra)
在uboot中有直接的defconfig，所以也就不依赖openWRT的编译结果了。

