
## Clang是个啥

### Clang与LLVM

llvm有两种含义：llvm框架和llvm Core

- llvm框架定义了一个编译器的组成架构，此时Clang可以作为框架中针对C/C++语言的前端。
当然llvm框架也并不只是包含这一个前端，也可以有其他的前端。
- llvm Core是除了前端之外的其他构成编译器的部分，包括中端优化和后端。
  



### Clang与Gcc
- Clang是一个编译器前端，Clang+llvm Core=编译器，Gcc也是一个编译器
- Clang的代码结构更好，扩展性强

## Clangd
clangd是llvm项目推出的C/C++语言服务器，通过LSP(Language Server Protocal)协议向编辑器如vscode/vim/emacs提供语法补全、错误检测、跳转、格式化等等功能。C++的LSP曾经是cquery, ccls, clangd三足鼎立。但是clangd支持clang-tidy实时检查的功能是另外两者不具备的，而且cquery和ccls都是单个开发者主导的项目，clangd背后则是有llvm的背书。

一般写C代码，在vscode中用`C/C++`这个插件来进行自动补全，这类插件在复杂工程中效果不太好因为**它是基于代码进行分析**，理论上不能准确的区别同名函数到底是调用谁的情况。而且在实践中也经常产生莫名其妙找不到定义的情况。

而Clangd则是**支持基于编译的分析做代码补全**，
通过解析一个调用数据库文件`compile_commands.json`
来提供错误检查和补全等功能。

### 生成 `compile_commands.json`

一般采用make工具构建的工程，通过bear工具可以在编译中自动分析并生成compile_commands.json。

特别对于Linux kernel，有专门的工具`scripts/clang-tools/gen_compile_commands.py`，在编译后执行该脚本即可在根目录生成compile_commands.json。

### VSCode配置Clangd补全

1. 安装clangd插件
2. clangd插件与c/c++插件冲突，**目前的方法时禁用C/C++插件**。
3. 配置clangd服务的路径，可以在User或者Workspace。
4. 如果有其他参数需求，可以按需添加。

{{< notice note >}}
不生效的排查方法

clangd服务会输出一些日志，当你切换文件或者鼠标悬浮在代码上时会输出解析的日志，如果有Error可以在这里排查。比如[vscode使用clangd报error: invalid AST错误-CSDN博客](https://blog.csdn.net/qq_37812160/article/details/135506142)。
{{< /notice >}}


## Vim配置Clangd补全

我目前使用`coc-nvim`插件来调用Clangd进行补全。

## Clangd配置文件`.clangd.

Clangd支持的所有配置项：https://clangd.llvm.org/config.html

有两种方式对Clangd的配置进行设定：
1. 全局配置 `~/.config/clangd/config.yaml`
2. 工程内部 `.clangd` 仅在工程目录内生效

以上两个都是YAML类型的文件，我目前使用到了忽略一些编译参数:
```yaml
CompileFlags:
    Remove: [-march=armv8-a, -march=armv7-a] # invalid target CPU values in clangd-15
```

## Clang-format

代码格式化工具

## Reference
1. [Clangd config](https://ahmadsamir.github.io/posts/12-clangd-config-tweaks.html)
2. 
