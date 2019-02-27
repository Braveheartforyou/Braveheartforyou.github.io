---
title: git中git rebase的使用
date: 2019-01-24 16:59:06
tags: [Git]
categories: [Git]
description: 在git中有两种合并方式 git merge、git rebase 还有 git cherry-pcik(单个copy commit)、git rebase --onto (多个commit copy 合并)，合并多个 commit 也要用到 git rebase
---
## 概述
把一个分支中的修改整合到另一个分支的办法有两种：merge 和 rebase（译注：rebase 的翻译暂定为“衍合”，大家知道就可以了。）。在本章我们会学习什么是衍合，如何使用衍合，为什么衍合操作如此富有魅力，以及我们应该在什么情况下使用衍合。
同时，比如要要合并多个commit,就可以用git rebase -i head~2。
```shell
git merge // 合并分支
git cherry-pick // copy单个commit
git rebase // 衍合
git rebase [startpoint]   [endpoint]  --onto  [branchName] // copy多个commit
git rebase -i  [startpoint]  [endpoint] // 合并多个commit
//  pick：保留该commit（缩写:p）
//  reword：保留该commit，但我需要修改该commit的注释（缩写:r）
//  edit：保留该commit, 但我要停下来修改该提交(不仅仅修改注释)（缩写:e）
//  squash：将该commit和前一个commit合并（缩写:s）
//  fixup：将该commit和前一个commit合并，但我不要保留该提交的注释信息（缩写:f）
//  exec：执行shell命令（缩写:x）
//  drop：我要丢弃该commit（缩写:d）
```

## git merge
使用 git merge 合并master1和dev1分支
``` shell
git checkout master1
git merge dev1
```
![git merge](../images/git/1-1.png)
如上图所示：
最新的快照c2和c3，还有它们共同的祖先c1进行三方合并，合并的结果会产生以下新的c5，同时太还会对你的master1分支上的合并线条产生不好的结果。
![git merge](../images/git/1-2.jpg)

## git rebase
我们现在使用 git rebase 来合并master1和dev1分支
``` shell
git checkout dev1
git rebase master1
git checkout master1
git merge dev1
```
![git merge](../images/git/1-3.png)
如上图所示：
它的原理是回到两个分支最近的共同祖先，根据当前分支（也就是要进行衍合的分支 dev1）后续的历次提交对象（这里只有一个 C3），生成一系列文件补丁，然后以基底分支（也就是主干分支master1）最后一个提交对象（C2）为新的出发点，逐个应用之前准备好的补丁文件，最后会生成一个新的合并提交对象（C3'），从而改写 dev1 的提交历史，使它成为 master1 分支的直接下游.
把 C3 里产生的改变到 C2 上重演一遍。
现在回到 master1 分支，进行一次快进合并.
![git merge](../images/git/1-4.png)

## git rebase [startpoint]   [endpoint]  --onto  [branchName]
当我们想从master1分支上复制b、c、d节点复制到dev1分支上，如下图所示：
![git rebase [startpoint]   [endpoint]  --onto  [branchName]](../images/git/1-5.png)
master分支：
```
Author: velen <896662364@qq.com>
Date:   Wed Feb 27 22:27:30 2019 +0800

    d

commit d422a0cb01609f9007b5acf8771428d04ef5d963
Author: velen <896662364@qq.com>
Date:   Wed Feb 27 22:27:02 2019 +0800

    c

commit f379de54fe09093b5f09d4f28e8c1d87318c06df
Author: velen <896662364@qq.com>
Date:   Wed Feb 27 22:24:58 2019 +0800

    b

commit 7343021a9f5a65c6042bf62589ca02bd7bb95e7f (origin/hexo-source, hexo-source)
Author: velen <896662364@qq.com>
Date:   Wed Feb 27 22:20:53 2019 +0800

    a
```
dev1分支：
```
commit 12d7fd6a7c7ff82f433fc47244dfa29df77130a9 (HEAD -> dev1)
Author: velen <896662364@qq.com>
Date:   Wed Feb 27 22:29:47 2019 +0800

    e

commit 7343021a9f5a65c6042bf62589ca02bd7bb95e7f (origin/hexo-source, hexo-source)
Author: velen <896662364@qq.com>
Date:   Wed Feb 27 22:20:53 2019 +0800

    a

```
我们使用命令的形式为:
```
git rebase   [startpoint]   [endpoint]  --onto  [branchName]
```
其中，[startpoint]  [endpoint]仍然和上一个命令一样指定了一个编辑区间(前开后闭)，--onto的意思是要将该指定的提交复制到哪个分支上。
所以，在找到b(7343021a9f5a65c6042bf62589ca02bd7bb95e7f)和c(d422a0cb01609f9007b5acf8771428d04ef5d963)的提交hash后，我们运行以下命令：
```
git rebase 7343021a9f5a65c6042bf62589ca02bd7bb95e7f^ d422a0cb01609f9007b5acf8771428d04ef5d963 --onto dev1
```
