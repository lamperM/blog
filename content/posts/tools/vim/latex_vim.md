---
title: "LaTeX Vim Tutorial"
description: A tutorial about how I confiure LaTeX in vim.
tags: ["Vim"]
categories: ["Vim"]
date: 2022-05-04T17:07:51+08:00 
---


## Use plugin `vimtex`
Vim build-in support of LaTeX files is just OK. When we need more excellent exprience, good plugins is very recommended.

[vimtex](https://github.com/lervag/vimtex#configuration) is a nice and modern vim plugin for LaTeX files.

**Useful Futures of vimtex IMO**
* `<leader>ll` Complier. By default, it will auto-complier when you type `:w`.
* `<leader>lt` Open content tree as a sidebar.
* `<leader>lv` View PDF with configured PDF viewer.
* `<leader>li` File information.
* `cse` Change surrounding `\begin \end` environment.
* `tse` Exchange between `\begin{env}` and `\begin{env*}`.
* `tsc` Exchange between `\command{}` and `\command*{}`.

&nbsp;
&nbsp;
## Add Support of Simplified Chinese
### Install `xetex`
I use `xetex` to add supports for Chinese fonts in LaTex files. Actually the magician is amacro package of `xetex` named `xeCJK`. 

And `xetex` is included in`texlive`. so we install it from source:
```shell
sudo apt install texlive-xetex
```

### Install Chinese Font
If there is no Chinese font in your system, you must install one. I choose WinQingYuan microhei as a instance.
```shell
sudo apt install ttf-wqy
```
Excute `fc-list` to check if install successfully, here is excepted output:
```shell
fc-list  | grep wqy
```
```shell
/usr/share/fonts/truetype/wqy/wqy-microhei.ttc: WenQuanYi Micro Hei,文泉驛微米黑,文泉驿微米黑:style=Regular
/usr/share/fonts/truetype/wqy/wqy-microhei.ttc: WenQuanYi Micro Hei Mono,文泉驛等寬微米黑,文泉驿等宽微米黑:style=Regular
```
### Configure your tex file
```latex
\documentclass {article}
\usepackage{xeCJK}
\setCJKmainfont{WenQuanYi Micro Hei}

\begin{document}
Hello, LaTeX!

你好, LaTex!

\end{document}
```
Complier it and see, the Chinese font is displayed!

&nbsp;
&nbsp;
## Confusing Tools
### Difference between {pdf,lua,xe}Tex and {pdf,lua,xe}LaTeX
If a `.tex`file starts with `\documentclass`, it's a _LaTex_ format file rather than the _Plain Tex_ format file. 

The LaTeX format file has some specific macro like `\documentclass` that cannot be compliered by `[pdf]Tex`, so that's the job of `[pdf]LaTeX`. Same goes for other engines.

### What is `xetex/xelatex`?
`xetex/xelatex` is one of the TeX/LaTeX engines. Others are `pdfTex`, `LuaTex`, etc. [Wiki](https://fr.wikipedia.org/wiki/XeTeX)

`xetex/xelatex` add fonts and character sets support for TeX/LaTeX file.
* Treat input as Unicode
* Allow us to use many system fonts in LaTeX file easily

### What is `latexmk`?
LaTeXmk 是一个集成化的命令行工具, it must work with one LaTeX engine.

The fundamental issue that `latexmk` solves is that the number of runs of `[pdf]latex` is highly dynamically dependent on the document and the class file used. `latex` just need to be run once a time.


### Different between `CTeX`/`MiKTeX`/`TeXlive` ?
They are all 包含与.tex文件关联的各种编辑、查看工具、常用宏包及文档.

[CTex](http://www.ctex.org/HomePage) packages add complete Chinese support based on [MiKTeX](https://miktex.org/).
* CTex is only avilable in windows.
