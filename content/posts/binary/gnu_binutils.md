---
title: "Weapon: GNU Binutils"
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

# 查看符号
nm xxx.elf
readelf -s xxx.elf  # detailed

# 打印某个section的内容
readelf -p .strtab xxx.elf
```