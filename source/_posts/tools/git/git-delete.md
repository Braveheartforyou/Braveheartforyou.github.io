---
title: git中删除git本地远程分支、本地分支、远程分支
date: 2019-01-23 22:10:09
tags: [Git]
categories: [Git]
description: 删除本地git的远程分支和删除远程git分支
---

## 简述

在项目中使用 git 管理代码后，有些时候会创建很多不同名称的分支，以此区分各个分支代码功能。 而随着代码的合并，以前的分支就可能不再需要保存了，所以就要对没有用的分支进行删除，包括紧急回滚时从中抽取某一个版本记录所创建的临时分支。 这时候就可以使用下面的命令：

### 删除本地 git 分支

```javascript
git branch -D branchName
// -D 是 -delete缩写
```

### 删除本地 git 远程分支

```javascript
git branch -D origin/branchName
// -D 是 -delete缩写
```

### 删除远程 git 分支

```javascript
git push origin -D origin/branchName
// -D 是 -delete缩写
```
