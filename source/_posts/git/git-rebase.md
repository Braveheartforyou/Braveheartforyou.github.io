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
![git merge](../../images/git/1-1.png)
如上图所示：
最新的快照c2和c3，还有它们共同的祖先c1进行三方合并，合并的结果会产生以下新的c5，同时太还会对你的master1分支上的合并线条产生不好的结果。
![git merge](../../images/git/1-2.jpg)

## git rebase
我们现在使用 git rebase 来合并master1和dev1分支
``` shell
git checkout dev1
git rebase master1
git checkout master1
git merge dev1
```
![git merge](../../images/git/1-3.png)
如上图所示：
它的原理是回到两个分支最近的共同祖先，根据当前分支（也就是要进行衍合的分支 dev1）后续的历次提交对象（这里只有一个 C3），生成一系列文件补丁，然后以基底分支（也就是主干分支master1）最后一个提交对象（C2）为新的出发点，逐个应用之前准备好的补丁文件，最后会生成一个新的合并提交对象（C3'），从而改写 dev1 的提交历史，使它成为 master1 分支的直接下游.
把 C3 里产生的改变到 C2 上重演一遍。
现在回到 master1 分支，进行一次快进合并.
![git merge](../../images/git/1-4.png)

## git rebase [startpoint]   [endpoint]  --onto  [branchName]
当我们想从master1分支上复制b、c、d节点复制到dev1分支上，如下图所示：
![git rebase [startpoint]   [endpoint]  --onto  [branchName]](../images/git/1-5.png)
master1分支：
```
commit 302d0381120b55518924d0fa5d91aeb651e7d4fd (HEAD -> master1)
Author: zhang.jg <zhangjingguo@unicdata.com>
Date:   Fri Mar 1 14:38:25 2019 +0800

    d

commit 9ef980cce33baa48a96f2d989d17330047cdf36b
Author: zhang.jg <zhangjingguo@unicdata.com>
Date:   Fri Mar 1 14:38:04 2019 +0800

    c

commit 0d00c05dc3cb28447e35d5d4be23ca76cc32741c
Author: zhang.jg <zhangjingguo@unicdata.com>
Date:   Fri Mar 1 14:37:27 2019 +0800

    b

commit 57840ed32293ca5218a9734402b51d6bcad9698d
Author: zhang.jg <zhangjingguo@unicdata.com>
Date:   Fri Mar 1 14:34:50 2019 +0800

    a

commit 24b0963b54f65fc58ca2642e81e7cdc7dae1e8ac (origin/develop, master, develop)
Author: zhang.jg <zhangjingguo@unicdata.com>
Date:   Fri Feb 22 13:28:15 2019 +0800

    Initial commit from Create React App

```
dev1分支：
```
commit 1225ea20d48bd16fbcfe1465e84f06292c52fe42 (HEAD -> dev1)
Author: zhang.jg <zhangjingguo@unicdata.com>
Date:   Fri Mar 1 14:36:58 2019 +0800

    e

commit 57840ed32293ca5218a9734402b51d6bcad9698d
Author: zhang.jg <zhangjingguo@unicdata.com>
Date:   Fri Mar 1 14:34:50 2019 +0800

    a

commit 24b0963b54f65fc58ca2642e81e7cdc7dae1e8ac (origin/develop, master, develop)
Author: zhang.jg <zhangjingguo@unicdata.com>
Date:   Fri Feb 22 13:28:15 2019 +0800

    Initial commit from Create React App

```
我们使用命令的形式为:
```
git rebase   [startpoint]   [endpoint]  --onto  [branchName]
```
其中，[startpoint]  [endpoint]仍然和上一个命令一样指定了一个编辑区间(前开后闭)，--onto的意思是要将该指定的提交复制到哪个分支上。
所以，在找到b(7343021a9f5a65c6042bf62589ca02bd7bb95e7f)和c(d422a0cb01609f9007b5acf8771428d04ef5d963)的提交hash后，我们运行以下命令：
```
git rebase 0d00c05dc^ 302d0381 --onto dev1
```
如果有冲突解决冲突就解决冲突,如果没有就跳过这一步：
```
git add .
git rebase --continue
```
![git rebase [startpoint]   [endpoint]  --onto  [branchName]](../images/git/1-6.png)
当前HEAD处于<font color="#ff502c">游离状态</font>，实际上，此时所有分支的状态应该是这样:
![git rebase [startpoint]   [endpoint]  --onto  [branchName]](../images/git/1-7.png
所以，虽然此时<font color="#ff502c">HEAD</font>所指向的内容正是我们所需要的，但是<font color="#ff502c">dev1</font>分支是没有任何变化的，git只是将<font color="#ff502c">b-c-d</font>部分的提交内容复制一份粘贴到了<font color="#ff502c">dev1</font>所指向的提交后面，我们需要做的就是将<font color="#ff502c">dev1</font>所指向的提交id设置为当前HEAD所指向的提交id就可以了，即:
```
git checkout dev1
git reset --hard 316ad6d
```
这时候就完成了。
<font color="#ff502c">git rebase [startpoint]   [endpoint]  --onto  [branchName] </font>还有一种用法。
参考 ><font color="#ff502c">https://blog.csdn.net/endlu/article/details/51605861</font>

## git cherry-pick
如果只是复制某一两个提交到其他分支，建议使用更简单的命令:git cherry-pick
```
git checkout '你的分支'
git cherry-pick '你要copy的id'
git log //查看
```
## git rebase -i  [startpoint]  [endpoint] // 合并多个commit
其中-i的意思是--interactive，即弹出交互式的界面让用户编辑完成合并操作，[startpoint]  [endpoint]则指定了一个编辑区间，如果不指定[endpoint]，则该区间的终点默认是当前分支HEAD所指向的commit(注：该区间指定的是一个前开后闭的区间)。
在查看到了log日志后，我们运行以下命令：
未完待续。