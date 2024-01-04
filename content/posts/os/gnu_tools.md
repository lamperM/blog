---
title: "GNU 二进制工具集"
date: 2022-12-03T19:08:22+08:00
tags: [Operating System]
categories: ["Operating System"]
---

```sh
# 输出源码和汇编
objdump -DS your_binary
# 仅输出汇编, 所有汇编, 不仅仅是代码相关
objdump -D your_binary

# 判断编译时有没有使用-g
readelf -n bin

# 输出Elf中的所有符号(基于符号表非调试信息)
nm demo.elf
```





