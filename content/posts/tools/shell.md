---
title: "Shell 命令技巧"
tags: ["Shellscript"]
categories: ["Shellscript"]
date: 2022-07-20T11:54:13+08:00
---
## sh 是个啥
busybox 的 shell 是 ash，ash 不同于主机上的 sh bash tcsh，它是个精简版，很多标准shell 支持的特性它都不支持。

sh 的含义是默认的 shell，可能是bash、可能是dash，一般来说只是一个软链接。

{{< notice info "sh与$0之间的关系？" >}}
$0 具体代表什么含义是每个shell程序实现的，如果sh是指向dash，那么此时 `$0` 就会输出dash。
{{< /notice >}}

>[bash和dash的语法区别 - 博客园](https://www.cnblogs.com/exmyth/p/15876568.html)

## 常用Shell命令

>:information_source: 如未特殊说明，以下的命令在bash和zsh中都能正确生效。

### Grep - 内容匹配

Grep stands for *Global Regular Expression Print*.

```shell
# 语法结构，PATTERN是正则表达式，所以不需要像find命令用*号

# 参数
# -n ==> print line number
# -r ==> recursive search
# -E ==> 扩展正则表达式
# -I ==> 跳过二进制文件
# --exclude/--exlude-dir ==> 排除路径和文件
grep -Inr page_fault 
```


### mkfs.ext4 - 格式化文件

```sh
# 格式化文件为ext4分区
mkfs.ext4 <file> 
```

### dd - 生成和转换文件

https://www.runoob.com/linux/linux-comm-dd.html


### mount - 挂载硬盘分区
```sh
sudo mount [file] [dir] # 挂载file到dir
sudo umount [dir] 
sudo mount # 输出当前已经挂载的分区
```

### find - 查找文件
```bash
# 查找24小时内修改过的文件
find . -mtime 0
# 查找2小时内修改过的文件
find . -mmin -$((2*60))
# 查找24小时内访问过的文件
find . -atime 0s
```

### 十六进制查看
xxd: 按字节解释。因为它来自于文本编辑器 Vim 嘛。

hexdump: 默认类似于 -x 格式，两字节数值解释。小端序下和xxd是双字节内是反的。


### tee - 标准输入到文件
常用的情景是，执行一些命令的输出**不仅想要打印要屏幕，同时想输出到文件中保存**。

```bash
# 与tee结合，同时输出到屏幕和文件
make 2>&1 | tee log.txt 
# 使用tee命令可以将一条命令的输出同时传递给多个命令进行处理。
ls | tee > (grep keyword) > (wc -l) 
```

### echo - 输出字符串
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

### which/whereis - 定位文件路径
which 查看可执行文件的位置，whereis 除了可执行文件还能搜索其他类型的文件, 不常用, 详见 `man whereis`

```shell
$ which python3
/usr/bin/python3
```





## 小技巧


### 如何理解2&>1?

```bash
make               # stdout和stderr，会输出到屏幕上
make 1> stdout.log # 仅将标准输出重定向到stdout.log文件
make > stdout.log  # 同上，简略写法
make 2> stderr.log # 将stderr输出重定向到文件，有错误语句
make 2>1           # 将stderr输出重定向到文件名1的文件
make 2>&1          # 将stderr输出重定向stdout
```

将stdout和stderr结合，有啥用呢？
```bash
make 2>&1 >all.log      # stdout和stderr都会被存到all.log中
make 2>&1 | tee log.txt # 与tee结合，同时输出到屏幕和文件
```

### `-` 的妙用

一些命令支持使用 `-` 代替文件名, 输入输出都可以:

- 代替标准输出; 一些命令会将`-o/-O` 后面的`-`判定为输出到*STDOUT*,  详见下面示例.
- 代替标准输入; 

下面给出两个同时代替输入输出的例子:

```sh
# 将标准输入(STDIN)的内容作为gcc的输入, 编译后的结果输出到标准输出(STDOUT)
echo 'void foo() {}' | gcc -x c -o - -
# 将下载的文件输出到标准输出, 同时作为tar命令的输入文件, 进行解压
wget -O - "https://www.dropbox.com/download?plat=lnx.x86_64" | tar xzf -
```

{{< notice tip  >}}
`-` 如何被解析是**取决于命令的实现**, 非标准. 比如 `cd -` 就有特殊的含义
{{< /notice >}}

### sort 应付多种场景
```sh
# 对版本号进行排序(x.y.z)
echo -ne "1.2.3\n4.5.6\n3.4.5" | sort -V
```

### cp正确链接文件
执行cp命令时，如果目录下有链接文件，拷贝源文件而不是链接文件，这在链接文件指向相对地址时非常有用。
```bash
cp -rL /path/to/ /path/from
```

### 查看当前shell
```shell
echo $0
```

### find 删除文件

rm和find结合，find过滤要删除的文件，传递给rm。**建议删除前先输出到屏幕检查**。
```shell
find . -maxdepth 1 -executable -type f | xargs rm
```


### 统计代码量

```shell
# 列出*所有的文件及其代码行数*, 只统计.c 和.h, 过滤`./scripts`目录.
find -name '*.[c|h]' ! -path './scripts/*' | xargs  wc -l
# +将内容按照代码行数降序排列
find -name '*.[c|h]' ! -path './scripts/*' | xargs  wc -l | sort -rn
# 若仅列出*总的代码行数*, 去除**空行**
(find ./ -name '*.[c|h]' -print0 | xargs -0 cat) | sed '/^\s*$/d' | wc -l
```

### 其他

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


## Shell脚本分析

### shift 用于处理参数
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
### 实战学习：自动更新Git仓库

来源：南京大学ICS PA [ics-pa-gitbook/update.sh](https://github.com/NJU-ProjectN/ics-pa-gitbook/blob/master/update.sh)

```bash
#!/bin/sh

git fetch origin master

# Trick 1: 1:-'@{u}' ==>  1代表参数传入，如果为空就 当前分支所跟踪的远程分支('@{u}')
UPSTREAM=${1:-'@{u}'} . 
# Trick 2: @ ==> 当前分支
# Trick 3: git rev-pase ==> 列出某个分支最新的commit ID
LOCAL=$(git rev-parse @)
REMOTE=$(git rev-parse "$UPSTREAM")

if [ $LOCAL = $REMOTE ]; then
  echo "Already up to date."
  exit
fi

git reset --hard origin/master
git gc
```


### 变量的定义和引用

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


### 判断执行脚本时带的参数

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

### 自动拷贝文件到 SD Card

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

### 获取所有文件信息(可递归进入子目录)

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

### 带颜色的输出

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
