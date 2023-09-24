---
title: "DWARF(2): basetype类型"
tags: ["Binary", "ELF"]
categories: ["Binary"]
date: 2023-05-09T16:51:49+08:00
---

想要描述一个变量，必须知道它类型信息，才能知道变量的大小、输出的格式等。

Dwarf 为 C 语言定义了一些描述数据类型的 DIE，包括 basetype, array,pointer, structure...

## basetype

今天我们先介绍最简单的 basetype。

basetype 是指那些 C 语言自身定义的基础类型，像`int`, `double`这些。

basetype 类型的 DIE 通常有属性:

- `DW_AT_name`: basetype 的名称
- `DW_AT_byte_size`: 该 basetype 占空间大小

下面给出描述`int`和`double`的 DIE 展示(还是通过`objdump`工具输出）：

```sh
 <1><43>: Abbrev Number: 3 (DW_TAG_base_type)
    <44>   DW_AT_byte_size   : 4
    <45>   DW_AT_encoding    : 5	(signed)
    <46>   DW_AT_name        : int

 <1><60>: Abbrev Number: 4 (DW_TAG_base_type)
    <61>   DW_AT_byte_size   : 8
    <62>   DW_AT_encoding    : 4	(float)
    <63>   DW_AT_name        : (indirect string, offset: 0x9): double


```

## Array

数组表示为 `DW_TAG_array` 的 DIE，通常含有属性:

- `DW_AT_type`： 指向数组元素的 DIE

数组的长度信息如何表示呢？每个 array DIE 的每个 child 都代表一个维度信息，最左边的 child
描述第一个维度信息。

描述维度信息的 DIE 为 `DW_TAG_subrange_type`, 其包含两个属性:

- `DW_AT_type`: 指向所属 array 的 DIE
- `DW_AT_upper_bound`: 表示数据在该维度下的长度, **数值是-1 的**

所以要得到一个数组的总长度，需要遍历其 DIE 的所有 child。

下面给出一个二维数组`int var_array[3][5]`的相关 DIE 信息:

```sh
 <1><43>: Abbrev Number: 3 (DW_TAG_base_type)
    <44>   DW_AT_byte_size   : 4
    <45>   DW_AT_encoding    : 5	(signed)
    <46>   DW_AT_name        : int
 <1><4a>: Abbrev Number: 4 (DW_TAG_array_type)
    <4b>   DW_AT_type        : <0x43>
    <4f>   DW_AT_sibling     : <0x60>
 <2><53>: Abbrev Number: 5 (DW_TAG_subrange_type)
    <54>   DW_AT_type        : <0x60>
    <58>   DW_AT_upper_bound : 2
 <2><59>: Abbrev Number: 5 (DW_TAG_subrange_type)
    <5a>   DW_AT_type        : <0x60>
    <5e>   DW_AT_upper_bound : 4
 <2><5f>: Abbrev Number: 0
```
