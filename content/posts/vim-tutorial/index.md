---
title: "我的 vim 调教随笔"
tags: [vim]
categories: [vim]
date: 2022-05-09T19:28:12+08:00
---

Search a word quickly: put cursor on the word, press `/` and press `<C-R>` `<C-W>`.

&nbsp;
## 缩写的含义(Meaning of abbreviations)
Operation
* d - delete
* y - yank(copy, 因为c被占了)
* c - change
* r - replace
* v - visual select

Scope or location
* i - inside
* a - around
* f - forward
* t - to

Object
* w - word
* s - sentence
* p - paragraph

&nbsp;
## 书签: Bookmark
`ma`: create bookmark `a` inside file.  
`mA`: create global bookmark `A`.  
`` `a ``: jump to bookmark `a`. 


`:marks`: display all bookmarks



## 缩进: indent
- `>`: increase indent  , `<`: decrease indent  ,`=`: auto indent

- `>>`: 增加当前行的缩进

- `gg=G`: 缩进全文, 无论当前光标在哪

> 以上命令都可以配合visual mode使用

### 自动缩进的规则

主要有四种可用缩进的方式, 分别是:

```
'autoindent'    沿用上一行的缩进。
'smartindent'   类似 'autoindent'，但是可以识别一些 C 语法以能在合适的地方
                增加 / 减少缩进。
'cindent'       比上面两个更聪明；可以设置不同的缩进风格。
'indentexpr'    最灵活的一个: 根据表达式来计算缩进。若此选项非空，则优先于其它
                选项覆盖。参见  indent-expression 。
```

自定义的快速命令:

```vimrc
command IndentOff setl noai nocin nosi indentexpr=""
command IndentOn  setl ai cin si  "indentexpr can't be re-enabled.
command IndentStatus set ai? si? cin? indentexpr?
```



> `cindent` 不一定对所有的语言都有效果. 只是 C-like 风格, 其中一个要求是顶层函数必须在第一列中含有 `{`.

> 只有当`indentexpr`计算不出当前需要缩进几格时(return -1), 才使用上面的三个规则. 它是优先级最高的.




## `F`ind and `T`ail
`f(`: 从当前cursor处向右查找下一个`(`, 并将光标移动到`(`处.  
`F(`: Like `f(`, but 向左查找.  
`t(`: Like `f(`, but 将cursor移动到`(`的前一个.  
`T(`: You can guess.

#### Trick
`vt(c`: With visual, 删除当前光标到下一个`(`前的所有内容.

`;`/`,`: 查找下一个/上一个 `f/F/t/T` 的内容. 

&nbsp;

## `S`ubstitute and `G`lobal

> See: `:help :s` and `:help :g`

这两个都属于vim的命令. vim 的替换和sed 的`s`命令使用方式基本一致. 就不多介绍了. 

而 vim 的 global 命令和sed有些许差别. 使用Sed删除包含个字符串的行的指令为: `sed '/STRING/d' input_file`, 而在vim中则多了一个**g**前缀, `:g/STRING/d`.

global 可以和 substitute 结合使用, 例如想要在包含某个字符串的行中替换`good`为`excellent`

```shell
:g/STRING/s/good/excellent/
```

> TODO: 
>
> 1. More [cmd] in global. [Power of g | Vim Tips Wiki | Fandom](https://vim.fandom.com/wiki/Power_of_g)
> 2. vim subtitute使用的正则表达式集包含 `\zs`和`\ze`, 然而 sed 没有(Sed 为 POSIX Basic Regular Expression).

&nbsp;

## 大小写转换

| cmd  | description |
|------|-------------|
| `g~` | 翻转大小写  |
| `gu` | 转换为小写  |
| `gU` | 转换为大写  |

以上命令(严格来说叫操作符)需要配合**动作命令**来使用.
* `gUaw`: 将光标所在位置的*单词*转为大写
* `gUap`: 将光标所在位置的*段落*转为大写


&nbsp;
## Search and replace

### case 1: search and convert to uppercase/lowercase
我直觉想到的方式是`%s/html/HTML/gc`

这种方式在简单情况下也行, 比较灵活且直观, 但是对于复杂文件不够通用且容易出错

还有一种方式是先搜索, 然后一步步替换
* 搜索: `/\vhtml\C`
* 替换: 执行命令`gUgn`, 然后使用`n`和`.`来重复操作下一个选中项.

> `gn`命令进对于sreach的匹配项使用, 类似于`n`, 但会将下一个匹配项(若光标停在match上, 那则选中当前匹配项)
> 转为visual模式选中的状态.

> 其实对于简单的文本, `n`和`.`也可以简化为`.`. 唯一的坏处就是如果两个匹配的距离太大, 
> 你不能确认是否search了你想要的内容.


### case 2: search the text seleted in *visual mode*
> vim 本身并未提供这个功能, 需要借助一个脚本来完成

[search the text selected in visual mode](#search-text-selected-in-visual-mode)

&nbsp;

## Visual Block 模式

- 选中后, 编辑所有行: `I`(captial i), 编辑完成后按两次`ESC`

- 重复visual 选中上次的 block:  Normal模式下`gv`即可.

&nbsp;
## 单词间跳转

`w`: Move cursor to begin of next word.  
`b`: Move cursor to begin of last word.  
`e`: Move cursor to end of next word.


### Trick
`w`/`b`配合`ce`使用可达到在某一行中快速移动到某个单词, 然后删除该单词开始edit.

`daw`: 即 Delete A Word, 可以删除一个完整的单词, 无论当前光标的位置在哪.

&nbsp;

## 编辑二进制/十六进制文件

可以使用`xxd`命令将一个文件中的文本转换为hex格式显示. 在vim中键入`:%!xxd` 即可. 得到的效果如下:

```xxd
0000000: 5468 6973 2069 7320 6120 7465 7374 0a41  This is a test.A
0000010: 6e6f 7468 6572 206c 696e 650a 416e 6420  nother line.And 
0000020: 7965 7420 616e 6f74 6865 720a            yet another.
```

后面的对应文本是自动生成的, 仅需要修改十六进制的部分即可. 修改完成后, 要返回原本的模式, 键入`:%!xxd -r`.

> 可通过设置文本格式对十六进制内存高亮显示  `set ft=xxd`. 

&nbsp;

## 如何同步 VIM Dotfiles

vim 的 dotfiles 主要包含`.vimrc`和`.vim/`中的插件.

* 对于`.vimrc`, 我选择使用mackup 软件和其他system dotfiles 一起备份. [Git repo](https://github.com/wangloo/dotfiles)

* 对于 plugins, 传统的管理插件的方式(使用`vim-plug`), 也就是放在`~/.vim/plugged/`目录中的, 可以通过`:PlugInstall`命令在新机器上重新从网上克隆. 能够保证使用的是新版本.
* VIM 8.0 之后, 引入 *pack system* 新的插件管理方式. 对于这类的插件, 我们直接利用`submodule`加入另一个备份的 [Git repo](https://github.com/wangloo/vimpack). 使用方法见`README`.

&nbsp;

## Good plugins
> Reference: [The Ultimate vimrc](https://github.com/amix/vimrc)

### TODO

### Installed 
[NERD Commneter - 快速注释](https://github.com/preservim/nerdcommenter#settings)

[NERD Tree - 目录树](https://github.com/preservim/nerdtree)

[Open File Under Cursor - 打开光标处的文件目录](https://github.com/amix/open_file_under_cursor.vim)
* 不支持`vim-plug`安装. 直接clone源码到`plugged`目录即可.
* Usage: `gf`: 在当前window打开文件. `<C-w><C-f>`: **new vertical windows**中打开文件.

[Ack.vim - 快速定位内容](https://github.com/mileszs/ack.vim)

[LeaderF - Like Ctrlp but better?](https://github.com/Yggdroot/LeaderF)

[barbaric - normal模式切换英文输入法](https://github.com/rlue/vim-barbaric)

&nbsp;
## Helpful script
### search text selected in visual mode
```vimrc
xnoremap * :<C-u>call <SID>VSetSearch('/')<CR>/<C-R>=@/<CR><CR>
xnoremap # :<C-u>call <SID>VSetSearch('?')<CR>?<C-R>=@/<CR><CR>
function! s:VSetSearch(cmdtype)
let temp = @s
norm! gv"sy
let @/ = '\V' . substitute(escape(@s, a:cmdtype.'\'), '\n', '\\n', 'g')
let @s = temp
endfunction
```
