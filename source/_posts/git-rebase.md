---
title: git_rebase
date: 2019-01-24 16:59:06
tags: [Git]
categories: [Git]
description: 在git中有两种合并方式 git merge、git rebase 还有 git cherry-pcik(单个copy commit)、git rebase --onto (多个commit copy 合并)，合并多个 commit 也要用到 git rebase
---
## 概述
把一个分支中的修改整合到另一个分支的办法有两种：merge 和 rebase（译注：rebase 的翻译暂定为“衍合”，大家知道就可以了。）。在本章我们会学习什么是衍合，如何使用衍合，为什么衍合操作如此富有魅力，以及我们应该在什么情况下使用衍合。
同时，比如要要合并多个commit,就可以用git rebase -i head~2。
```shell
1、git merge // 合并分支
2、git cherry-pick // copy单个commit
3、git rebase // 衍合
4、git rebase [startpoint]   [endpoint]  --onto  [branchName] // copy多个commit
5、git rebase -i  [startpoint]  [endpoint] // 合并多个commit
//  pick：保留该commit（缩写:p）
//  reword：保留该commit，但我需要修改该commit的注释（缩写:r）
//  edit：保留该commit, 但我要停下来修改该提交(不仅仅修改注释)（缩写:e）
//  squash：将该commit和前一个commit合并（缩写:s）
//  fixup：将该commit和前一个commit合并，但我不要保留该提交的注释信息（缩写:f）
//  exec：执行shell命令（缩写:x）
//  drop：我要丢弃该commit（缩写:d）
```