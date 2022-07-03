---
title: "The amazing vim"
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
## Bookmark
`ma`: create bookmark `a` inside file.  
`mA`: create global bookmark `A`.  
`` `a ``: jump to bookmark `a`. 


`:marks`: display all bookmarks



## 缩进: indent
`>`: increase indent  
`<`: decrease indent  
`=`: auto indent  

### Trick
> 以下命令都可以配合visual mode使用

`>>`: 增加当前行的缩进

`gg=G`: 缩进全文, 无论当前光标在哪


## `F`ind and `T`ail
`f(`: 从当前cursor处向右查找下一个`(`, 并将光标移动到`(`处.  
`F(`: Like `f(`, but 向左查找.  
`t(`: Like `f(`, but 将cursor移动到`(`的前一个.  
`T(`: You can guess.

#### Trick
`vt(c`: With visual, 删除当前光标到下一个`(`前的所有内容.

`;`/`,`: 查找下一个/上一个 `f/F/t/T` 的内容. 

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
## 单词间跳转

`w`: Move cursor to begin of next word.  
`b`: Move cursor to begin of last word.  
`e`: Move cursor to end of next word.


### Trick
`w`/`b`配合`ce`使用可达到在某一行中快速移动到某个单词, 然后删除该单词开始edit.

`daw`: 即 Delete A Word, 可以删除一个完整的单词, 无论当前光标的位置在哪.

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
