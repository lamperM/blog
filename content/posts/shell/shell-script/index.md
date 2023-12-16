---
title: "Shellscript"
tags: ["Shellscript"]
categories: ["Shellscript"]
date: 2022-07-20T11:54:13+08:00
---

>:information_source: 如未特殊说明，以下的命令在bash和zsh中都能正确生效。


## 变量的定义和引用

```sh
# 何时变量赋值要加""?
# => 包含分隔符，空格或tab时
qemu_opt="-machine raspi3b"

# 引用变量时为什么加{}? ${var}

# 引用变量时为什么加""? "$var"

# 变量赋值为什么$()?
# => ()里面是shell命令时用
qemu_ver=$($qemu --version | head -n 1) 
```

## 常用Shell命令的注意事项

### echo的参数

```sh
# echo不自动换行
echo -n "some-string"

# echo转义特殊字符
echo    "some-string\n" # 不转义\
echo -e "some-string\n" # 正确转义换行
```
>Zsh中，echo默认带`-e`，bash中则不是。
>所以说，**如果输出的字符串有转义字符，不管要不要转义，都显式指定一下**。
>- `-e`  强制转义
>- `-E`  强制不转义

### sort 应付多种场景

```sh
# 对版本号进行排序(x.y.z)
echo -ne "1.2.3\n4.5.6\n3.4.5" | sort -V
```

### shift 操作参数

`shift`命令将参数左移，应用场景:
```sh
# 场景1：提取第二个参数及后面的所有
# ./shell qemu-system-aarch64 -machine raspi3b -smp
qemu=$1
shift  # 默认左移1
qemu_opt=$@


## 场景2: 处理多参数且位置不定的情况
while [ "$#" -gt 0 ]; do
  case "$1" in
    -f|--file)
      file="$2"
      shift 2
      ;;
    -d|--directory)
      directory="$2"
      shift 2
      ;;
    -m|--mode)
      mode="$2"
      shift 2
      ;;
    *)
      echo "Unknown option: $1"
      exit 1
      ;;
  esac
done
```


## Shellscript的编写技巧

1. Bash脚本正式语句开始前加 `set -e`, 使得任何命令返回非0时立即退出整个Shell脚本。
2. 规定分隔符。**默认Shell会采用空格、tab、换行都作为分隔符**，
   有时需要屏蔽某些，仅使用空格
   ```sh
   export IFS=' ' # 启用空格分隔only
   flag="false"
   for str in $qemu_ver;
   do
     # do something
   done
   unset IFS  # 恢复默认
   ```

## 统计代码量

列出*所有的文件及其代码行数*, 只统计.c 和.h, 过滤`./scripts`目录.

```shell
find -name '*.[c|h]' ! -path './scripts/*' | xargs  wc -l
```

+将内容按照代码行数降序排列

```shell
find -name '*.[c|h]' ! -path './scripts/*' | xargs  wc -l | sort -rn
```

若仅列出*总的代码行数*, 去除**空行**

```shell
(find ./ -name '*.[c|h]' -print0 | xargs -0 cat) | sed '/^\s*$/d' | wc -l
```

## 删除目录下所有的可执行文件

```shell
find . -maxdepth 1 -executable -type f | xargs rm
```

## 判断执行脚本时带的参数

```shell
if [ $# -ne 1 ]; then
    echo "ONE parameter is needed"
    exit -1
fi

if [ $1 == 'build' ]; then
    # do something
elif [ $1 == 'run' ]; then
    # do something
elif [ $1 == 'gdb' ]; then
    # do something
else
    echo "Not supported command"
fi
```

## 自动拷贝文件到 SD Card

> TODO
>
> 1.  添加进度条

```shell
#!/bin/bash
sd_path=$(find /media/$USER -maxdepth 1 -type d -name "*-*")

while [ ! -d "${sd_path}" ]
do
    sleep 1
    echo "waiting for inserting SD-Card"
done

echo "SD-Card is inserted"
cp ./output/kernel/kernel.bin  ${sd_path}
echo "Copy completely"
```

## 获取所有文件信息(可递归进入子目录)

获取`dir`路径下的所有文件的信息, 这里获取的是文件的**完整路径**.

> TODO
>
> 1. 操作数组下标的方式可能有待改进? `filenum`感觉没必要, 暂时还不会改
> 2. 通过拼接获得文件信息(路径)的方式也有点怪异

```shell
dir=./
files=()
filenum=0
function getfiles()
{
    for file in `ls $dir`;
    do
        if [ -d $file ]; then
            cd $file
            getfiles
            cd ..
        else
            files[$filenum]=$(pwd $file)/$(basename $file)
            # echo file=$(pwd $file)/$(basename $file)
            let filenum++
        fi
    done
}
```

## 带颜色的输出

使用[ANSI escape code](https://en.wikipedia.org/wiki/ANSI_escape_code)

```
Black        0;30     Dark Gray     1;30
Red          0;31     Light Red     1;31
Green        0;32     Light Green   1;32
Brown/Orange 0;33     Yellow        1;33
Blue         0;34     Light Blue    1;34
Purple       0;35     Light Purple  1;35
Cyan         0;36     Light Cyan    1;36
Light Gray   0;37     White         1;37
```

Code example:

```shell
#!/bin/bash

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
CYAN='\033[0;36m'
NC='\033[0m' # No Color

echo -e "${YELLOW}HELLO, YELLOW${NC}"
echo -e "${GREEN}HELLO, GREEN${NC}"
echo -e "${RED}HELLO, RED${NC}"
echo -e "${BLUE}HELLO, BLUE${NC}"
echo -e "${CYAN}HELLO, CYAN${NC}"

#########################################################
# generic functions #####################################

function ERROR(){
    echo -e "${RED}[error] $*${NC}";
    exit 1
}

function INFO {
    echo -e "${BLUE}[info] $*${NC}";
}

function WARN {
    echo -e "${YELLOW}[warn] $*${NC}";
}

function LOG {
    echo -e "${GREEN}[log] $*${NC}" >> $LOG
}

INFO "This is an infomation"
WARN  "This is a log"
```
