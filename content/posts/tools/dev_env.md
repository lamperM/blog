---
title: "我的开发环境构建指南"
tags: [tools]
date: 2023-07-17T19:28:12+08:00
---

# 前言

写这篇博客的背景是**我实在忍受不了每次换新的开发机器都得费好大的劲来完全恢复以前的环境**， 而且，我平常喜欢搜集各种有用的工具、好看的主题，字体这些，如果零零散散的记录，大概率会忘记或者记不得某些细节。

所以，最后期望达到的是**能够使我每次在新机器上搭建环境只需要看这一篇文章就可以了**。因此这里会记录：

- 帮助提升开发效率的小工具
- 好看的字体、主题
- 配置某些环境的要点及注意事项

> 🥀 到目前为止，我还未发现一种方式能够完全达到“一键式布置”，这也不是本文的目的。
> 付出至少半天的时间的一定的，希望未来能发现一种好的方法。

# 字体

## Fira Code

这款字体适合做编程字体，蛮好看的。我在 vscode 和 terminal 下都使用了这款字体。

详情及安装参考[github](https://github.com/tonsky/FiraCode)

## 霞鹜文楷

开源的中文字体，做博客、PPT 不错。

详情及安装参考[github](https://github.com/lxgw/LxgwWenKai)

# vscode

## 主题

# vim

## vimrc

## ycm 特别注意

YCM 插件对 python, vim 的版本均有要求。

### 下载

可以使用 vim-plug 等工具下载, 也可以下载源码然后拷贝到`.vim`目录下

### 编译

编译用到 python3, 这里是问题最多的一步

```shell
# 编译并添加对C的提示支持
python3 install.py --clangd-completer --verbose

Searching Python 3.8 libraries...
...
Downloading Clangd from https://github.com/ycm-core/llvm/releases/download/13.0.0/clangd-13.0.0-x86_64-unknown-linux-gnu.tar.bz2...

```

使用`--clangd-completer`参数时, 脚本会去下载 clangd-14.0.0-x86_64-unknown-linux-gnu.tar.bz2 文件, 比较慢. 也可以**提前根据提示的网站自己手动下载**压缩包.

下载完成后, 放到本地目录下:

```shell
:~/.vim/plugged/YouCompleteMe/third_party/ycmd/third_party/clangd/cache$ ls
clangd-14.0.0-x86_64-unknown-linux-gnu.tar.bz2
```

还需对脚本`YouCompleteMe/third_party/ycmd/build.py`进行修改, 防止重新下载.

```python
def DownloadClangd( printer ):
  ...
  MakeCleanDirectory( CLANGD_OUTPUT_DIR )

  if not p.exists( CLANGD_CACHE_DIR ):
    os.makedirs( CLANGD_CACHE_DIR )
    # 注释下面的语句
    #  elif p.exists( file_name ) and not CheckFileIntegrity( file_name, check_sum ):
    #  printer( 'Cached Clangd archive does not match checksum. Removing...' )
    #  os.remove( file_name )

  if p.exists( file_name ):
    printer( f'Using cached Clangd: { file_name }' )
```

### 配置

YCM 配合一个配置文件`.ycm_c_c++_conf.py`, YCM 搜索的位置在 vimrc 中指定:

```vimrc
Plug 'rdnetto/YCM-Generator', { 'branch': 'stable' }
let g:ycm_global_ycm_extra_conf = "~/.ycm_c_c++_conf.py"
```

其内容的 example:

```py
import os
import ycm_core

flags = [
  '-Wall',
  '-Wextra',
#  '-Werror',
  '-Wno-long-long',
#  '-Wno-variadic-macros',
  '-fexceptions',
  '-ferror-limit=10000',
  '-DNDEBUG',
  '-std=c99',
  '-xc',
  '-isystem/usr/include/',
  ]

SOURCE_EXTENSIONS = [ '.cpp', '.cxx', '.cc', '.c', ]

def FlagsForFile( filename, **kwargs ):
  return {
  'flags': flags,
  'do_cache': True
  }
```

### 优化
:small_red_triangle_down: 对于C/C++来说, YCM的使用最好配合*compilation database* 来使用, 例如[compiledb](https://github.com/nickdiego/compiledb). 否则, 可能头文件的path识别出问题([stackoverflow](https://stackoverflow.com/questions/64277317/youcompleteme-not-work-properly-for-c-headers.)).

2022年2月13日我使用的compilation database生成工具从`compiledb`换成了`bear`, 因为`bear`更好的支持递归, 即有`make -C`的情况. 

需要的compilation database生成工具介绍: [Compilation database — Sarcasm notebook](https://sarcasm.github.io/notes/dev/compilation-database.html)


# 终端软件安装
## 源替换
## apt
```sh
sudo apt install python3-pip
sudo apt install tmux
sudo apt install fzf
sudo apt install zsh
```
## pip3
```shell
# CLI 代码高亮
sudo pip3 install pygments
```

# shell

## zsh
### oh-my-zsh
oh-my-zsh可以看作对zsh的配置文件做一层抽象，使配置更方便。
带来的缺点就是速度变慢。

> 进入git目录下太卡
>
> TODO： 是主题的原因，可以配置

## 配置文件

- .bashrc
- .zshrc
- .bash_aliase
- .bash_path



# terminal
ubuntu自带的终端我觉得还不错，有些人说Terminitor不错，分屏功能还是挺常用的！




# tmux


# ssh 密钥
```sh
ssh-keygen -t rsa -C "cnwanglu@icloud.com"
```

