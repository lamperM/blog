---
title: "GNU 二进制工具集"
date: 2023-12-14T13:21:27+08:00
tags: ["Binary"]
categories: ["Binary"]
---

```sh
# 输出 section header table
readelf -S xxx.elf  

# 输出 program header table
readelf -l xxx.elf

# 输出 ELF header
readelf -h xxx.elf  

# 输出 elf header，section header table，program header table(常用）
readelf -e xxx.elf 

# 输出Elf中的所有符号(基于符号表非调试信息)
nm xxx.elf
readelf -s xxx.elf  # detailed

# 打印某个section的内容
readelf -p .strtab xxx.elf

# 判断编译时有没有使用-g
readelf -n bin

# 输出源码和汇编
objdump -DS your_binary

# 仅输出汇编, 所有汇编, 不仅仅是代码相关
objdump -D your_binary

```