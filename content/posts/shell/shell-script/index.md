---
title: "武器库: shell scripts"
tags: ["shell", "bash"]
categories: ["weapons"]
date: 2022-07-20T11:54:13+08:00
tags: [shell, bash]
---

:information_source: 以下命令/脚本的执行环境均为 *Bash*.

## 统计代码量

> 使用到的命令包含: find, wc, xargs, sort等

列出*所有的文件及其代码行数*, 只统计.c和.h, 过滤`./scripts`目录. 

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



&nbsp;

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

&nbsp;
## 自动拷贝文件到SD Card

> TODO
>  1. 添加进度条
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

&nbsp;
## 获取所有文件信息(可递归进入子目录)
获取`dir`路径下的所有文件的信息, 这里获取的是文件的**完整路径**.

> TODO
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

&nbsp;
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
