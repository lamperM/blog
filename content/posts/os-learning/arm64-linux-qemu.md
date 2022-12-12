## 编译Linux 源码

[上海交通大学镜像站](http://ftp.sjtu.edu.cn/sites/ftp.kernel.org/pub/linux/kernel/)

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



## 构建根文件系统

使用 Busybox 构建, 下载源码时可能比较慢, 暂时没有发现国内镜像站

```shell
# Download busybox source code
wget https://busybox.net/downloads/busybox-1.35.0.tar.bz2

# menuconfig - generate .config
make ARCH=arm64 CROSS_COMPILE=aarch64-none-linux-gnu- menuconfig
# compile
make ARCH=arm64 CROSS_COMPILE=aarch64-none-linux-gnu-  -j16

make ARCH=arm64 CROSS_COMPILE=aarch64-none-linux-gnu- install

# create rootfs
cd ../
qemu-img create -f raw qemu_rootfs.img  128M
# set ext4 filesystem
mkfs.ext4 qemu_rootfs.img
# mount on host
mkdir mnt-tmp
sudo mount -o loop qemu_rootfs.img ./mnt-tmp
# copy file
sudo cp busybox-1.35.0/_install/* mnt-tmp/ -r
cd mnt-tmp
# create other path
sudo mkdir proc sys dev etc etc/init.d
```

在`etc/init.d/rcS`中写入启动脚本

```shell
#!/bin/sh
mount -t proc none /proc
mount -t sysfs none /sys
/sbin/mdev -s
```

赋予rcS可执行权限

```shell
sudo chmod +x rcS
```



改为`mount -a` 之后启动脚本会自动挂在/etc/fstab文件中声明的所有分区

fstab 在 Linux 开机以后自动配置哪些需要自动挂载的分区  

```sh
#device  mount-point    type     options   dump   fsck order
proc /proc proc defaults 0 0
tmpfs /tmp tmpfs defaults 0 0
sysfs /sys sysfs defaults 0 0
tmpfs /dev tmpfs defaults 0 0
debugfs /sys/kernel/debug debugfs defaults 0 0
tracefs /sys/kernel/tracing tracefs defaults        0       0
```



## 添加软件

### perf

**perf**已经集成到了Linux 主分支中，源码的位置在`tools/perf`

编译命令:



### strace

下载源码：[Releases · strace/strace (github.com)](https://github.com/strace/strace/releases)

编译&&安装

```sh
# 确定编译的参数
./configure \
--host=aarch64-linux \
--prefix=/home/soben/linux-qemu/strace-6.0/_install \
--enable-mpers=no \
CC=aarch64-none-linux-gnu-gcc \
LD=aarch64-none-linux-gnu-ld \
RANLIB=aarch64-none-linux-gnu-ranlib

# 编译为静态链接方式
make LDFLAGS+='-static -pthread' -j16

# 拷贝到 _install 目录
make install
```







## 启动 QEMU 运行 Linux

