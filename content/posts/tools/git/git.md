---
title: "git 宝典"
description:
date: 2022-12-13T17:39:42+08:00
tags: [git, tools]
categories: ["Git"]
---

## 合并操作: git merge

merge行为的语义是将其他分支的修改合并到当前分支。由此就产生了两种内部的实现原理，
以下均假设当前分支为main，其他分支为dev：
- fast-forward：
- three-way merger：

### Fast-forward Merge

main是dev的某个直接祖先，或者说他们之间是一条线的关系。此时将dev的修改合并进来相当于移动main指针指向dev的最新commit(F1)。此时merge称为 fast-forward.

例如, 当前在 main, 执行`git merge dev`的过程如下:

```
     main                              main
       |                                |
M1 --- M2          ===>    M1 --- M2 -- F1
       \                                |
        \--- F1                        dev
             |
           dev
```

### three-way Merge

合并的两者不构成直接的祖先-孩子关系，也就意味着main和dev分别位于两个分叉上（见下图左）。此时merge的步骤就相对复杂：
1. 找到main和dev的公共祖先M2
2. 列出main和dev分别基于公共祖先来说做了哪些修改
   1. 如果两条分叉的修改不冲突，完美合并
   2. 经常出现的是，两个分叉难免对同一片段做了不同修改，此时**标记为冲突**，等待用户解决
3. 因为main和dev位于两个分叉，合并会新建一个节点（M4），commit的信息是“Merge dev into main”。如果步骤2中产生冲突了，那么解决冲突的行为就被记录到M4的diff。没有冲突时diff是空的。


```
             main                                main
              |                                   |
M1 --- M2 --- M3    ===>     M1 --- M2 --- M3 --- M4
       \                            \            /
        \--- F1                      \--- F1 ---
              |                           |
             dev                         dev
```

因为此时merge需要借助三个 commit(main, dev, 公共祖先)，这种操作就叫做 three-way merge。


## 检出: Checkout

> ps: 就不能想出更易懂的名词吗？非得用一个“检出”？

checkout是一个非常综合的命令，其后面既可以接 分支名、commitid、文件。其实也很难用一个词概括它的行为吧。

### 初版:  `checkout <file>`
我感觉checkout最初的功能是+文件名，使工作区恢复为HEAD的状态，但不影响暂存区（这是和reset命令不同的地方）。==> 通俗来说，该文件先回到HEAD，再应用暂存区的修改 ==> 再通俗点，放弃工作区中没有被暂存的改动。

这种case下的checkout命令等价于 `git restore`。

### 扩展1:  `checkout <commit>`
扩展基础功能，`git checkout <commit>` 将仓库管理的所有文件都恢复到commit的状态。**也是只影响在工作区中的改动，这个和初版相同**。如果commit是HEAD以前的，那么此时如何标记分支状态？ ==> 用一个特殊的detached状态来标定。

### 特殊case1: `checkout <branch>`
分支切换，跟扩展1不同，不会影响工作区的改动，仅仅是切换分支，如果有文件冲突，执行就会失败。

这种case下的checkout命令等价于 `git switch`

## 特殊case2: `checkout <commit> <file>`
特殊的case，`git checkout <commmit> <file>`，如果指定了commit，就不再只影响工作区了，也会影响暂存区。

效果等同于`git reset --hard <file>`。
**这个貌似也太难理解了，只能记住了。**

## 使用技巧
太麻烦了，还是少用git checkout。git也是感觉到了，所以才新建了restore和switch命令来代替。只是为了兼容不得不继续支持以前的功能。总结来说：
1. 首先暂时切换到某个commit，用checkout detached state还是比较方便的。
2. 分支切换和新建，也习惯了，继续用也行。
3. 至于放弃某个文件工作区的状态，还是用新的restore命令吧。checkout万一用错了（初版和特殊case2），会造成暂存区内容的丢失。





## 变基: rebase

rebase 命令需要指定一个**基准分支**，`git rebase <base-branch>`，
rebase 会将当前所处分支整体移动到`base-branch`之后，即改变了当前分支的历史。

```
// before rebase
[A]---[B]---[C]---[D]<-dev
  \
   \--[E]<-master

// after rebase
[A]---[E]---[B]---[C]---[D]<-dev
       |
       master
```

### 交互式 rebase

交互式 rebase 是一种更高级的用法。基础的 rebase 上面说了是将当前分支的所有提交移动到 base-branch 之后。而交互式 rebase 提供一个方法，在移动之前"挑选"当前分支的 commit。

实际工程中，通常来说，我们将开发分支移动到 master 之前，可以经过交互 rebase 来整理开发分支中混乱的 commit 记录。

具体的使用方法是，为`git rebase`指令提供`-i`参数:

```sh
git checkout dev
git rebase -i master
```

这个命令会打开一个文本编辑器，列出当前 feature 分支的所有提交:

```
pick 33d5b7a Message for commit #1
pick 9480b3d Message for commit #2
pick 5c67e61 Message for commit #3
```

列出的内容就能完整的表示 dev 分支的所有提交，按照顺序。而我们不仅可以任意的重排这些 commit，而且修改`pick`关键字就能对这些 commit 做改动。举个例子，我们可能发现 commit2 只是对于 commit1 做了一个很小的改动，它们完全可以合并为一个 commit，那么直接 commit2 的`pick`修改为`fixup`，整个内容变为:

```
pick 33d5b7a Message for commit #1
fixup 9480b3d Message for commit #2
pick 5c67e61 Message for commit #3
```

当你保存并退出这个文件时，改动就会生效，不仅将 dev 整体移动到了 master 后，并且合并了前两个 commit。

```
// after rebase interactive
[A]---[E]---[B]---[D]<-dev
       |
       master
```

### rebase 整理多个 commit

如上面交互式 rebase 所述，当你开发完 dev 分支，需要`merge`到 main 分支时，可以先利用交互式 rebase"整理"一下 dev 分支的 commit。

这里其实要用到一个小 trick，上面说过 rebase 命令需要指定一个 base-branch，实际上是一个 base-commit，这种场景下我们不是要合并其他分支，所以**base-branch**可以选择当前 dev 分支的前面某一次 commit。

```sh
git checkout dev
git rebase -i HEAD~3
```

以上指令实现的功能就是给你整理最后 3 次提交的机会，但**不会合并其他分支的东西**。

那如果我想整理整个 dev 分支呢？是向上找到 dev 的第一次 commit 吗？ Git 提供了一个方便找到 dev 分叉出来的那次 commit，将其输出传递给`git rebase -i`即可实现整理整个 dev 的所有 commit。

```sh
git merger-base dev main
```

## `git merege` vs `git rebase`

### 准则

- 如果分支已经被提交到远程仓库，就不能再改变他的历史了，即不能使用 rebase。
  git 也会阻止你这么做，因为分支的历史已经被修改，除非 force-push。

- 你能进行 rebase 的分支是本地的”私人分支”，私人表示为: 只有你自己在使用，别人不会基于你的分支做东西。

### dev 同步主分支的改动: rebase

假设我们正在本地的 dev 分支开发一个特性，此时你的同事在 main(也可以是其他的远程分支)上提交了一个重要的 commit，以至于你需要它来继续你的开发任务。

这种情况我们使用 rebase 和 merge 都能完成目标，但是 rebase 是更好的选择。

- 首先满足 rebase 的使用条件，我们仅仅是破坏了本地 dev 分支的历史，并没有动到其他的远程分支，所以就不存在干涉别人
- 其次，在 dev 上 merge 其他分支会产生一次不必要的*merge commit*，其不代表任何实际意义，没必要存在的

### 合并 dev 到主分支: merge

很简单的逻辑，主分支或者其他合作开发的分支并不是你一个人在用，并且需要最后同步到远程仓库，不符合使用 rebase 的准则

## git log

### 参数:

| Parameter                | Description                                                        |
| ------------------------ | ------------------------------------------------------------------ |
| non-param                | 列出所有历史提交的 SHA、作者、提交日期和 commit                    |
| -p                       | 按补丁显示每次更新，比--stat 更全                                  |
| --stat                   | 显示每次更新修改文件的名称及添加（删除）的行数。比--name-only 更全 |
| --name-only              | 显示文件清单                                                       |
| --name-status            | 显示文件清单及改动方式(新增、删除、修改)                           |
| --oneline                | 只显示前 6 位 SHA 值和 commit                                      |
| -n                       | 显示前 n 条 log                                                    |
| \<branch\>               | 查看某个分支的历史提交。**该参数只能 log 命令之后**                |
| \<branch1\>..\<branch2\> |                                                                    |
|                          |                                                                    |
|                          |                                                                    |

参考网站：https://www.cnblogs.com/bellkosmos/p/5923439.html

### Example 1: 彩色显示重要信息

```sh
git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
```

![](gitlog_1.png)

### Example 2: 查看本地分支和对应远程分支的 commit 差异

```sh
git log --oneline main..origin/main
```

## 远程仓库
### 克隆远程仓库

默认克隆master分支
```bash
git clone <shortname> <url>
```
需要克隆指定的分支
```bash
git clone -b <branch> <shortname> <url>
```
克隆并指定文件名
```bash
git clone <shortname> <url> <dirname>
```

{{< notice note >}}``
clone或者fetch会将历史都下载到本地，内容可能很大。
**使用`--depth`参数只克隆最近几次commit的内容**，减少下载的时间。
```bash
git clone <url> --depth=1 # Clone最近一次提交only
```
以后如果需要下载历史，执行`git fetch --unshallow`即可下载完整内容。

{{< /notice >}}

## 分支操作

```sh
# 创建远程分支关联的本地分支，假设已经存在origin/v5.10
git checkout v5.10
```

## 子模块: submodule

### 新增一个子模块

(1) 将子模块上传到远端仓库上，或者使用已有的第三方项目

(2) 执行
```sh
git submodule add [url] [path]
```
(3) 此时看`git status` 会有两个changes，分别是：

  1. `.gitmodules`中会增加一条项目, 记录子模块的名称和地址
      ```
      [submodule "SubmoduleTestRepo"]
          path = SubmoduleTestRepo
          url = https://github.com/jzaccone/SubmoduleTestRepo.git   
      ```
  2. `[子模块同名的文件]`: 记录主项目追踪的是子模块的哪个commit

>.gitmodule和子模块同名文件的作用
>
>- 先说比较简单的.gitmodule，**记录主项目中用到的所有子模块**。
>一般来说仅仅记录子模块的路径和url，但有时还会有指定的分支名。
>这个我们在下面会介绍。
>
>- 然后是和子模块同名的文件，我们想一下，上面的.gitmodules仅仅告诉了从哪来，
>但是一个仓库是有多个分支，对应多个commit的，那么需要有一种方式去指定使用绑定哪个commit，
>这就是同名文件的作用。在同名文件中会记录一下commitid，表明主项目依赖这个commitid下的子模块，
>他们之间具有绑定关系。

(4) `git add`这两处改动，`git commit -m "add submodule xxx"`



### Submodule Command

Execute `git submodule --help` to get more info.

#### `git submodule init`

此命令clone新项目的时候会用到, 完整的命令是
```
git submodule init [path]
```

>**在详细介绍前需要说明两个文件的差别: .gitmodule和.git/config**:
>- .gitmoudles在上面已经介绍过一些，这里主要想说**它是跟随主项目一起提交的**，
>而.git/config是后面生成的本地文件。
>- .gitmodules会给出列出所有子模块，但是不一定每次都全部使用。
>**具体会启用哪些由.git/config文件规定。


`git submodule init`指令会指定生成.git/config文件的内容。
选择某几个子模块启用即写入`.git/config`。


- 在缺省参数下的含义是添加所有在.gitmodules中的子模块。
- 未来`update`指令只会更新`.git/config`中的项目。

#### `git submodule update`

刚才说了，因为.gitmodules中的所有子模块并不一定都使用，
所以新clone下来的项目并不会同时下载所有的子模块。
在上面的init阶段生成了.git/config来指定需要哪些之后，
update命令就是用于下载指定的子模块。

##### --init
由于此命令配合init命令使用，两者通常连续操作，所以可以合并为:
```
git submodule update --init
```

##### --remote
一般情况下，下载子模块都是checkout到.git/config指定的commit。
`--remote`则会checkout到**追踪分支的最新commit**，下面会说到如何设置追踪分支，
默认是master。

如果最新的commit不等于.git/config指定的，那么执行完该指令后会产生一次同名文件的改动。

#### `git submoudle set-x`

修改的是哪个文件？

"x"可以是url，修改子模块的地址。

这里我主要想着重介绍`set-branch`命令，**将原来追踪自模块的方式由commit更改为(branch+commit)。** 想要修改追踪子模块的commit到某个分支的最新，有两种方法:
1. 传统方案
   ```sh
   cd lib1/
   git checkout <branch>
   cd ..
   git add lib1
   git commit -m "change lib1's tracked commit"
   ```
2. 使用set-branch参数
   ```sh
   git submodule set-branch -b <branch> lib1/ 
   # 改变设置之后，子模块不会立即变化
   # 必须指定带remote参数的update命令，才会用更新到显式指定的分支最新，
   # 而不是同名文件中规定的commitid
   git submodule update --remote
   git add lib1 .gitmodules
   git commit -m "change lib1's tracked commit"
   ```

>`set-branch`修改了.gitmodules, 也同步到了.git/config中。如果你想手动修改.gitmodules来设置分支，不要忘了`submodule init`来同步到.git/config


### 修改URL
1. 修改.gitmodules中对应子模块的URL
2. `git submodule sync` 同步到`.git/config`文件中
3. `git submodule update`  更新子模块内容


### 使用Submodule注意事项


#### 1.Submodule到底是追踪分支还是commit

事实证明子模块默认追踪的不是分支，容易验证。
我们将子模块基于当前分支（假设master）新建一个子分支，并做一些修改，
可以看到主项目中的子模块同名文件也发生了改动。
所以说，它默认并不是跟踪分支。

我感觉可以证明**它默认跟踪的是HEAD**。还是用上面的例子，
此时主项目中同名文件发生了改动。但是如果将子模块的分支切换回原来的master，
子模块同名文件的改动就消失了。以我目前的见解我暂且这么认为。

#### 2. 同名文件发生了修改，应不应该stage？

分两种场景: (1)如果子项目的这些更新有意义同步到主项目中，那么就add并commit这个同名文件的改动，
message为"更新submodule"。 (2)如若只是更新子项目而已，或许是为了其他依赖的项目所改的，并不
想涉及到本主项目，那么就restore此次更新，或者重新执行`submodule update`即可(前提是对子模块
的修改已经push)。

所以说，同意主项目中的这次change的人必须是更新这次子模块的人，由他决定是否同步到主项目。
其他人**甚至在使用期间都不需要`cd`进入子模块做`git pull`的**，这样也就不会有决策产生，即便
子项目在远端更新了，你要做的就是关注那个同名文件就行，当同名文件更新了，在主项目中 
`submodule update`即可，


### 子模块的优缺点
TODO

## 补丁

关于Git生成补丁/打补丁的方法有两套命令：`diff&apply` 和 `format-patch&am`，这两套方案分别适用于不同的场景。

- git apply命令主要用于应用补丁文件。补丁文件通常是由git diff命令生成的，
- git apply不会自动创建提交记录，并且需要手动执行提交操作。
- git am命令也是用于应用补丁文件，但主要用于处理Git发送的邮件补丁(Mbox格式)。
- git am会自动创建提交记录，并且可以处理多个补丁文件。
- git am会自动忽略已经应用过的补丁文件，这在处理多个补丁文件时非常方便。
- git am还可以处理附件，通过提取附件内容并保存到工作目录中。这在处理有附件的邮件补丁时非常有用。


```bash
# (1) diff & apply
# 生成补丁文件
git diff > patchfile.patch
# 补丁文件应用到当前工作目录
git apply patchfile.patch
# (2) format-patch & am
# TODO
```


### am 和 apply命令的区别

