---
title: "Vim-YouCompleteMe插件国内安装"
tags: [tools]
categories: ["DevTools"]
date: 2023-11-17T19:28:12+08:00
---


# Vim插件YouCompleteMe国内安装

>Update 2023/11/23: vim下现在我不用YCM了，换成Coc.nvim来进行代码补全。
>不过在我看来其实没有特别明显的优势，所以YCM的配置注意事项还是留在这吧。

YCM 插件对 python, vim 的版本均有要求。

## 下载

可以使用 vim-plug 等工具下载, 也可以下载源码然后拷贝到`.vim`目录下

## 编译

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

## 配置

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

## 使用方式
:small_red_triangle_down: 对于C/C++来说, YCM的使用最好配合*compilation database* 来使用, 例如[compiledb](https://github.com/nickdiego/compiledb). 否则, 可能头文件的path识别出问题([stackoverflow](https://stackoverflow.com/questions/64277317/youcompleteme-not-work-properly-for-c-headers.)).

2022年2月13日我使用的compilation database生成工具从`compiledb`换成了`bear`, 因为`bear`更好的支持递归, 即有`make -C`的情况. 

需要的compilation database生成工具介绍: [Compilation database — Sarcasm notebook](https://sarcasm.github.io/notes/dev/compilation-database.html)
