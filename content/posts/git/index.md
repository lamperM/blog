---
title: "git 宝典"
description: 
date: 2022-12-13T17:39:42+08:00
tags: [git]
categories: [git]
---

## 合并操作: git merge

merge 有两种方式:
- fast-forward
- three-way merger

### Fast-forward Merge
假设合并的双方为`main`为`dev`, 如果其中一个是另一个的祖先, 此时直接移动HEAD到前方即可, 称为fast-forward.

例如, 当前在main, 执行`git merge dev`的过程如下:
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
合并的两者不构成直接的祖先-孩子关系, 产生了分叉. 此时进行合并就需要有个基准(参考), 对于两边相较于基准的每个diff来说:
- 合并的两者都在基准上进行了改动, 且改动不一致, **标记为冲突**
- 如果该diff**仅在其中一方**有改动, 那么就保留此次改动

合并时使用的参考就是**两个合并commit的最近公共祖先**, 这种借助三个commit(main, dev, 公共祖先)才能完成的合并操作就叫做 three-way merge.

例如, 当前在main, 执行`git merge dev`的过程如下:
```
             main                                main
              |                                   |
M1 --- M2 --- M3    ===>     M1 --- M2 --- M3 --- M4
       \                            \            /
        \--- F1                      \--- F1 ---
              |                           |
             dev                         dev
```

>three-way 的合并方式如果发生了冲突, 会产生一次额外的 merge commit, 下面介绍它




### 什么情况下 merge commit 没有任何diff?
按照上面的例子, three-way merge 发生冲突后会产生一次额外的merge commit, 即M4. 如果这是去查看M4 相较前一次commit的diff, 有时是没有的, 有时又会产生diff.

如果在解决冲突的过程中, 我们仅仅是接收了 M2,M3 或者F1的修改, 那么此时merge commit就不会有diff.

然而, 在解决冲突时, 我们也可以不采用来自两条路径的修改, 做一次新的修改(可以说, 同时接收两条diff就是这种情况), 此时查看merge commit的diff就是有内容的.
