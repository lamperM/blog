---
title: "Sed/Awk/Grep 三剑客"
tags: [tools]
categories: ["DevTools"]
date: 2023-01-03T19:28:12+08:00
---

## Sed

Sed stands for **Stream Editor**. 

Basic sed syntax:  

```
sed [options] {sed-commands} {input-file}
```

Sed reads one line at a time from the {input-file} and executes the {sed-commands} on that particular line

> The input 并非必须是文件
>
> `echo "Some string" | sed ''` 当然也是支持的.

### Option: `-n `

通常sed是按照行来处理文本的, 然后打印处理后的结果. 然而并不是符合匹配的行被打印, 所有的行都会被打印. 例如`sed 's/t/T/'`会输出所有的行, 并且替换其中某些行的`t`.

这样的结果不是我们想见的(大多数情况下), 所以, 我们可以添加`-n` option 来 *禁止自动打印所有内容*,  例如`sed -n 's/t/T/'` 不会输出任何结果. 

如果想要将匹配的结果单独打印, 则sed为我们提供了`p`命令. 例如, `sed -n 's/t/T/p'` 只会打印替换后的行.

### Option: `-i`

As we know, sed doesn't modify the input files by default. Sed writes the output to standard output. When you want to store that in a file, you redirect it to a file (or use the `w` command. 

如果要修改到源文件, 我们以前只能这样做:

```shell
sed 's/John/Johnny/' employee.txt > new-employee.txt
mv new-employee.txt employee.txt
```

> Don't do `sed 's/John/Johnny/' employee.txt > employee.txt`
>
> Because  whell shell see `> employee.txt` in the command line, it opens the file `employee.txt` for **writing**, wiping off all its previous contents.

但现在, 有了`-i`, 我们可以选择直接修改input file.

```shell
sed -i 's/John/Johnny/' employee.txt
```

:bangbang: 直接修改源文件是很危险的, 你也可以使用`-ibak`来做备份.

```shell
sed -ibak 's/John/Johnny/' employee.txt
```

### Option: `-z`

原本sed是按行处理的,  也就是`\n`结尾. 使用`-z` 选项后, 转换为以`\0` 即0x00来结尾.  借用`-z` 一次跨行对整个文本进行处理.

### Commands

`g`

`p`

`w`

`i` (ignore case)

### Command: `s` (substitute)

The most powerful command in the stream editor is **s**ubstitute.  

```shell
sed '[address-range|pattern-range] s/old/new/[substitute-flags]' inputfile
```

* 匹配范围字段是可选的. 未指定情况下默认是*all lines*. 指定范围Example: `sed '/Sales/s/Manager/Director/' employee.txt` 仅替换**包含**Sales字段的行.

#### Sed 替换分隔符

When there is a slash`/` in the original-string or the replacement-string, we need to escape it using `\`.
For this example create a path.txt file which contains a directory path as shown below.  

```shell
sed 's/\/usr\/local\/bin/\/usr\/bin/' path.txt
```



#### Substitution Grouping  

Sed中可以使用正则表达式中*匹配分组*. A group is opened with `\(` and closed with `\)`.

Example: 输出每个行第一个`,`前的所有内容.

```
 sed 's/\([^,]*\).*/\1/g' employee.txt
```

> [^,]* means zero or more non-comma. 
>
> In `[]`, a `,` means just a comma. And the leading `^` means "anything but..". 

#### Power of & - Get Matched Pattern  

`&` replaces it with whatever text matched the original-string or the regular-expression.  

```c
$ echo 1234 | sed 's/123/<&>/'
<123>4
```

## Sed 处理Bash变量

> 一旦你的pattern中包含变量, 整个pattern必须由**双引号**包围. 

对于某些变量里含有forward slash(`/`), 需要换一个pattern delimiter. Sed 允许替换为任意的字符, 需要做的工作针对不同的使用情况有些许区别.

**Scenario #1: `s` command**, 直接替换即可, 

```shell
sed -i "s:$pushed_dir:REPLACE:g" input_file
```

**Scenario #2: For patterns used in addresses**, 需要首先对新的delimiter使用`\:`进行escaped(转义)

```shell
sed -i "\:$pushed_dir:d" input_file
```

> See: [Using different delimiters in sed « \1 (backreference.org)](https://backreference.org/2010/02/20/using-different-delimiters-in-sed/index.html)


