
# Clang是个啥

**Clang与llvm**

llvm有两种含义：llvm框架和llvm Core

- llvm框架定义了一个编译器的组成架构，此时Clang可以作为框架中针对C/C++语言的前端。
当然llvm框架也并不只是包含这一个前端，也可以有其他的前端。
- llvm Core是除了前端之外的其他构成编译器的部分，包括中端优化和后端。
  



**Clang与Gcc**
- Clang是一个编译器前端，Clang+llvm Core=编译器，Gcc也是一个编译器
- Clang的代码结构更好，扩展性强

# Clangd
## VSCode配置Clangd补全

一般写C代码，用`C/C++`这个插件来进行自动补全，
效果不太好因为**它是基于代码进行分析**，
理论上不能准确的区别同名函数到底是调用谁的情况。
而且在实践中也经常产生莫名其妙找不到定义的情况。

“听说”Clangd可以**支持基于编译的分析做代码补全**，
原理上它解析一个调用数据库文件`compile_commands.json`
来做解析补全。

## Vim配置Clangd补全

我目前使用`coc-nvim`插件来调用Clangd进行补全。

## Clangd配置文件

Clangd支持的所有配置项：https://clangd.llvm.org/config.html

有两种方式对Clangd的配置进行设定：
1. 全局配置 `~/.config/clangd/config.yaml`
2. 工程内部 `.clangd` 仅在工程目录内生效

以上两个都是YAML类型的文件，我目前使用到了忽略一些编译参数:
```yaml
CompileFlags:
    Remove: [-march=armv8-a] # invalid target CPU values in clangd-15
```

# Clang-format

代码格式化工具

# Reference
1. [Clangd config](https://ahmadsamir.github.io/posts/12-clangd-config-tweaks.html)
2. 
