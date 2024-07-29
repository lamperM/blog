---
title: "Zephry：编译"
tags: ["Operating System"]
categories: ["Operating System"]
date: 2024-07-29T19:28:12+08:00
---

## 配置
```shell
# 下载源代码
git clone https://gitee.com/coollh/zephyr.git

# 下载SDK
wget https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v0.16.8/zephyr-sdk-0.16.8_linux-x86_64.tar.xz

# 进入 sdk 目录，setup
tar xvf zephyr-sdk-0.16.8_linux-x86_64.tar.xz 
cd zephyr-sdk-0.16.8/
./setup.sh

# 加入环境变量到 zephyr-env.sh
export ZEPHYR_GCC_VARIANT=zephyr
export ZEPHYR_SDK_INSTALL_DIR=../zephyr-sdk-0.16.8   ## Zephyr SDK 安装路径
export ZEPHYR_BASE=. ## Zephyr 源码路径

# source 环境变量文件
source zephyr-env.sh

# 安装依赖的库
pip3 install --user -r ~/zephyr/scripts/requirements.txt

```

## 编译并在qemu-x86上仿真运行 hello-world
```sh
cd zephyr/samples/hello_world/
mkcd build
cmake -GNinja -DBOARD=qemu_x86 ..
ninja # build
ninja run # build and run
```

## Ref
- [《嵌入式系统 – Zephyr开发笔记》 第2章 Zephyr 编译环境搭建（Linux） - BruceOu的博客](https://blog.bruceou.cn/2020/09/2-zephyr-compilation-environment-setup-linux/237/)