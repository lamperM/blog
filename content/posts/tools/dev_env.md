---
title: "开发环境构建指南"
tags: [tools]
categories: ["DevTools"]
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

## 配置备份
# vim

## 升级Vim

```sh
sudo add-apt-repository ppa:jonathonf/vim
```

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
sudo apt install cmake
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









# ssh 密钥
```sh
ssh-keygen -t rsa -C "cnwanglu@icloud.com"
```

# tmux
tmux 在远程开发时比较有用。我们在用ssh连到服务器时经常需要有多窗口的需求，
比起现有terminal软件自带的多窗口功能(Xshell,mobaxterm等)，使用tmux
会更加方便。
1. 窗口创建、切换等方式可以做到统一，不用追随终端软件
2. 可以保存现场，即便因为网络问题ssh断开，也能随便恢复到之前的状态。因为
   tmux是C/S架构，只要服务器上的server不死，永远可以恢复之前状态！
3. 甚至，tmux提供了将现场保存到本地文件中的功能。


## reference

1. [Tmux使用手册](http://louiszhai.github.io/2017/09/30/tmux/)


# 科学上网 Clash for Linux

目前clash的作者已经删除跑路，但是已经发布的版本还是可以正常使用的。这一章节介绍如何在Linux上用命令行配置Clash。

首先需要准备好一些文件：
1. clash在linux中的命令行可执行文件，目前这里还可以下载 https://github.com/Kuingsmile/clash-core/releases， 去下载clash-linux-amd64-v1.18.0.gz
2. Country.mmdb为全球IP库，可以实现各个国家的IP信息解析和地理定位，没有这个文件clash是无法运行的。 但目前版本的clash有点问题，不会自动生成MMDB文件，所以需要使用命令行下载。去[这个地方](https://github.com/Dreamacro/maxmind-geoip/releases/latest/download/Country.mmdb)或者[这个地方](https://gitee.com/dnqbob/sp_engine/blob/SPcn-01-02-20/GeoLite2-Country.mmdb.gz)看看
3. config.yaml为clash的代理规则和clash的一些其他设置。代理规则不需要我们自己编写，通过订阅地址直接下载即可。
此处将订阅链接粘贴进双引号中间。注意不要删除双引号，不要删除空格。
`wget -O ~/.config/clash/config.yaml "订阅链接"`。或者，如果你有clash的其他客户端，可以由已有的配置导出。

下载上面的文件，就可以准备好安装了：
1.  创建`~/.config/clash/`，并将config.yaml和Country.mmdb放进去，如果已经存在请忽略
    ```sh
    mkdir ~/.config/clash/
    mv Country.mmbd ~/.config/clash/
    mv config.yaml ~/.config/clash/
    ```
2. 解压`clash-linux-amd64-v1.18.0.gz`，并将可执行文件放到一个合适的位置
   ```sh
   gunzip clash-linux-amd64-v1.18.0.gz
   chmod +x clash-linux-amd64-v1.18.0.gz
   mv clash-linux-amd64-v1.18.0 ~/tools/clash-v1.18.0
   cd ~/tools/
   ```
3. 运行clash可执行文件，如果成功运行应该可以看到以下LOG
   ```sh
    $ ./clash-linux-amd64-v1.18.0
    INFO[0000] Start initial compatible provider 故障转移       
    INFO[0000] Start initial compatible provider 自动选择       
    INFO[0000] Start initial compatible provider 大机场 Big Airport 
    INFO[0000] inbound mixed://:7890 create success.        
    INFO[0000] RESTful API listening at: 127.0.0.1:9090
   ```

到此，clash就可以正常运行了，由于我的订阅地址有3条规则，所以会有3条Start initial compatible provider xxxx。

inbound mixed://:7890 则代表已经开启了http(含https)和socks代理，只要服务器内有软件流量通过7890这个端口，流量都将进入clash从而被代理。（但有些不支持设置，后面会说如何使用全局代理）

RESTful API listening at: [::]:9090代表clash已经开启了ui控制面板，是的，Linux的clash有可视化控制面板。

## 验证科学上网
根据我们之前前台运行可得知，默认是监控了自己的7890端口，验证方式可以通过`curl`向google发送一个请求看是否能正常返回。

```sh
curl --proxy 127.0.0.1:7890 www.google.com
```
返回正常 Google成功返回数据，代表7890端口代理正常，clash运行正常。**注意目前必须手动指定代理的地址和端口，后面会介绍启用全局代理。**


## Clash全局代理

可以添加这个配置到你的`.bashrc`/`.zshrc`中，使得终端的所有请求都走代理。
```sh
export http_proxy=127.0.0.1:7890 
export https_proxy=127.0.0.1:7890
```

这里只设置了http和https，初次之外，还可代理其他协议。

可以做成一个sh命令，当想临时关闭科学上网的时候执行unproxy。
```sh
function proxy() {
    export http_proxy=http://127.0.0.1:7890
    export https_proxy=$http_proxy
    echo -e "proxy on!"
}
function unproxy(){
    unset http_proxy https_proxy
    echo -e "proxy off"
}
```

验证全局代理配置成功只需要再执行一次`curl`即可，这次无需手动指定代理。
```sh
curl www.google.com
```


## 将Clash作为一个服务
使clash一直运行在前台会占用一个终端，而且总感觉不是很优雅，更好的方法是将clash作为一个服务来操作。

1. 在`/etc/systemd/system/`目录新建一个clash.service文件
   ```sh
   [Unit]
   Description=Clash service 
   After=network.target 
   
   [Service] 
   Type=simple 
   User=root 
   ExecStart=这里写你的clash运行的绝对路径（本文中的路径是`~/tools/clash-v1.18.0`） 
   Restart=on-failure RestartPreventExitStatus=23 
   
   [Install] 
   WantedBy=multi-user.target
   ```
2. 之后就可以用systemctl系列命令来确认是否启用成功：
    ```sh
    systemctl status clash 查看clash服务
    systemctl start clash 启动clash服务 
    systemctl stop clash 停止clash服务 
    systemctl restart clash 重启clash服务 
    systemctl enable clash 设置开机自启clash服务 
    systemctl daemon-reload 如果修改了clash.service文件，需要此命令来重载被修改的服务文件
    ```

## Reference
1. [Linux中安装Clash并且实现全局代理（纯命令行）](https://www.zztongyun.com/article/clash%20for%20linux%20ubuntu)
2. https://github.com/Kuingsmile/clash-core/releases
3. [clash-for-linux](https://iyuantiao.me/clash-for-linux.html)
4. [Ubuntu配置 命令行Clash 教程](https://zhuanlan.zhihu.com/p/662552814#:~:text=Ubuntu%E9%85%8D%E7%BD%AE%20%E5%91%BD%E4%BB%A4%E8%A1%8CClash%20%E6%95%99%E7%A8%8B%201%201.%E4%B8%8B%E8%BD%BD%E5%AE%A2%E6%88%B7%E7%AB%AF%E4%B8%8E%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%20%E9%A6%96%E5%85%88%EF%BC%8C%E4%BD%A0%E9%9C%80%E8%A6%81%E5%88%B0%20Releases%20%C2%B7,%E5%BD%93%E5%87%BA%E7%8E%B0%E9%97%AE%E9%A2%98%E7%9A%84%E6%97%B6%E5%80%99%EF%BC%8C%20%E5%BB%BA%E8%AE%AE%20tmux%20a%20-t%20clash%20%E6%9F%A5%E7%9C%8B%E6%97%A5%E5%BF%97%E6%9C%89%E4%BD%95%E6%8A%A5%E9%94%99%20)
5. [Clash can't initial MMDB of Country.mmdb](https://zhuanlan.zhihu.com/p/472152669)


# Windows下迁移WSL

WSL2目前已经相当好用了，在对性能要求不极致的场景下用WSL开发非常舒服。

迁移WSL到别的位置/别的机器还是比较方便的，也有人写了[脚本](https://github.com/pxlrbt/move-wsl)来做这些，它是针对迁移到其他硬盘位置的，所以我这次还是自己手动做一遍，原理都是相同的。

## 步骤
### 1. 关闭WSL
```sh
# 检查WSL是否在运行
wsl -l -v
  NAME          STATE           VERSION
* Ubuntu2004    Running         1
# 关闭
wsl --shutdown 
```

### 2. 导出WSL镜像
```sh
wsl --export Ubuntu2004 D:\Ubuntu2004_202311.tar
```

### 3. 注销原系统(可选)
```sh
wsl --unregister Ubuntu2004
```

### 4. 将镜像压缩文件复制到新机器/新位置
如果是新机器，还需要重新配置好WSL，开启一些选项:
```
控制面板->程序->启用或关闭 windows 功能，开启 Windows 虚拟化和 Linux 子系统（WSL2)以及Hyper-V。

勾选完成后，Windows11 会自己下载些东西，并提示你重启。等电脑彻底重启完以后，进行后续操作。
```

![](https://gitee.com/wangloocn/image-bed/raw/master/202311221925110.png)
![](https://gitee.com/wangloocn/image-bed/raw/master/202311221925827.png)


### 5. 在新机器上导入镜像文件
```sh
wsl --import Ubuntu2004 D:\wsl\Ubuntu2004\ D:\Ubuntu2004_202311.tar
```
执行的时间比较长, 执行完后至此WSL就迁移完毕了，剩下的是一下配置的修正。

### 6. 配置

#### 设置默认用户

这样移过来现在登陆就是root，我们需要进行一些配置:

Set your default user inside your distro by adding the following configuration to your /etc/wsl.conf.
```
[user]
default=loo
```
If the file doesn't exist create it manually. Then exit your distro, terminate it (`wsl -t Ubuntu2004`) and start it again.

#### 设置默认distro

```sh
wsl -s Ubuntu2004
```

这样完成后，所有的一切就OK了。

## Reference
1. https://zhuanlan.zhihu.com/p/622706723

# Linux组织Dotfiles

Linux开发环境中的许多软件都由配置文件，重新捣鼓一台新环境时去重新设置这些配置文件是非常复杂的一件事情，所以我想着用一种统一的方式进行管理。

- vim插件使用单独的子仓库管理
- 其他的配置文件暂时都使用mackup管理

>不将vim插件也归于`mackup`管理的原因是: 我的`.vim/pack/xx/`下的所有插件都是通过submodule的方式管理,这样有利于维护和更新。但是在mackup的管理方式中是将整个`pack/`的内容拷贝过来，这就与submodule的理念冲突了。此时去改mackup的实现不如将vim的插件系统单独进行维护更容易。


## 新环境恢复Dotfile

1. 恢复除了vim插件之外的其他配置文件，教程参考：[wangloo/dotfiles](https://github.com/wangloo/dotfiles/tree/master)
2. 单独恢复vim插件，教程参考: [wangloo/myvimpack](https://github.com/wangloo/myvimpack)


## Reference
- [Installing Vim(8) plugins with the native pack system](https://medium.com/@paulodiovani/installing-vim-8-plugins-with-the-native-pack-system-39b71c351fea)
