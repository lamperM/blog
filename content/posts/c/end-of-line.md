---
title: "行结束符在windows和linux的区别"
date: 2022-12-24T01:35:24+08:00
tags: [other]
---

使用VIM 打开一个文件时, 有时会看到例如 `^M` 这类字符出现. 下面我会挖一下其出现的原因.

## EOL 字符
EOL 或者说 end-of-line 表示一个新行的开始.

EOL 字符在不同的操作系统中是不同的. 本文中仅以 Linux 和 Windows 为例说明.
 * Windows中是以读到回车\<CR>和换行\<LF> 表示 EOL.
 * Linux 中仅以换行作为EOL

> * 回车\<CR> : Carriage return. 将光标回到行首, 对应C语言中的 `\r`
> * 换行\<LF> : Line feed. 将光标下移一行, 对应C语言中的 `\n`

在 Linux 中打开 Windows 下的文件将多余的回车通常显示成 `^M` 或者 `Control-M`






## Ref
[End Of Line Characters](https://peterbenjamin.com/seminars/crossplatform/texteol.html)
