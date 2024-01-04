---
title: "Dwarf(1): 基础"
tags: ["Dwarf"]
categories: ["Dwarf"]
date: 2023-05-09T15:51:49+08:00
---

Dwarf 把源文件中每个可描述的模块（例如函数，变量，结构体的声明等）描述为一个 DIE
(Debugging Information Entry)，所以每个源文件可以描述为若干 DIE 的组合。

每个 DIE 由一个 tag 和若干 attribute-val 键值对构成:

- tag: 描述此 DIE 的类型
- attribute-val: 描述此 DIE 的一些细节属性，项目根据 DIE 的类型不同而有差别

各个 DIE 之间会相互联系，一个 DIE 可能含有 parent，若干的 child 和 sibling，
它们之间组成树的结构。

## 查看一个 ELF 的所有 DIE

ELF 文件中的所有 DIE 存储在`.debug_info` section 中，通过 GNU utils 中的`objdump`工具
可以解析为可阅读的结构:

```sh
objdump --dwarf=info <file>
```

若我们有一个 `demo.c` 如下:

```c
void func(void) {
    int var_local;
}
```

编译为可执行文件后， 执行上述的`objdump`命令， 可以得到如下的输出（节选）：

```sh
 <1><68>: Abbrev Number: 5 (DW_TAG_subprogram)
    <69>   DW_AT_external    : 1
    <69>   DW_AT_name        : (indirect string, offset: 0x32): func
    <6d>   DW_AT_decl_file   : 1
    <6e>   DW_AT_decl_line   : 3
    <6f>   DW_AT_decl_column : 6
    <70>   DW_AT_prototyped  : 1
    <70>   DW_AT_low_pc      : 0x1129
    <78>   DW_AT_high_pc     : 0xb
    <80>   DW_AT_frame_base  : 1 byte block: 9c 	(DW_OP_call_frame_cfa)
    <82>   DW_AT_GNU_all_call_sites: 1
 <2><82>: Abbrev Number: 6 (DW_TAG_variable)
    <83>   DW_AT_name        : (indirect string, offset: 0x28): var_local
    <87>   DW_AT_decl_file   : 1
    <88>   DW_AT_decl_line   : 4
    <89>   DW_AT_decl_column : 9
    <8a>   DW_AT_type        : <0x43>
```

上述例子中节选了两个 DIE，分别是函数`func()`和局部变量`var_local`, 可以看到它们的
tag 是不同的，且都具有一系列属性。

并且，值得注意的是，这两个 DIE 的关系可以从首列`<>`中的数字得知，数字代表 DIE 构成树的
深度，此例子中<1>下面紧靠的<2>就表示<2>是<1>的 child。理论上也是如此，因为局部变量
是属于对应的函数体的。

## 小结

之后的章节打算介绍 Dwarf 是如何描述源文件中的不同对象的(函数,变量,数据类型等)
