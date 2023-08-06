---
title: "常用的 shell 命令"
date: 2022-11-27T14:45:58+08:00
tags: [shell, bash]
---



# 开发

## mkfs.ext4
格式化文件为ext4分区
```sh
mkfs.ext4 <file> # 将file格式化为ext4
```

## dd

https://www.runoob.com/linux/linux-comm-dd.html

## mount 
```sh
sudo mount [file] [dir] # 挂载file到dir
sudo umount [dir] 
sudo mount # 输出当前已经挂载的分区
```


# 通用
### where and which

which 查看可执行文件的位置

```shell
$ which python3
/usr/bin/python3
```

whereis 除了可执行文件还能搜索其他类型的文件, 不常用, 详见 `man whereis`

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



> `-` 如何被解析是**取决于命令的实现**, 非标准. 比如 `cd -` 就有特殊的含义
