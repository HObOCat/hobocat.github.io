---
title: "Git Use"
aliases: 
tags: [Tool]
date: 2024-08-24
time: 10:19
---


## 开始一个新的项目

### 初始化
```bash
git init
```
### 添加remote
```bash
git remote origin add <url> 
```
### 添加文件
```bash
git add 
```
### 提交
```bash
git commit -m "注释"
```
### 同步
- 远端拉取
```bash
git push -u origin <branchName>
```
- 远端推送
```bash
git pull origin <branchName>
```
## 查看日志和状态

- 日志
```bash
git log --oneline
```
- 状态
```bash
git status
```
## 删除文件
- 彻底删除
```bash
git rm fileName
git rm -r filedir
```
- 从git删除而不物理删除
```bash
git rm --cached fielname 
git rm --cached  -r filedir
```
- 执行了git add，尚未commit
```bash
git restore --staged fielname
```
## 恢复文件
```bash
- 执行了git rm 未commit
git reset HEAD
git restore <file|dir>
```

## 分支
### 查看分支
```sh
git branch
```
  后面没有接任何参数，它仅会输出当前在这个项目中有哪些分支。Git默认会设置一个名为master的分支，前面的星号（*）表示现在正在这个分支上。

### 新增分支
```sh
git branch <分支名>
```

### 改分支名
```sh
git branch -m 老分支名  新分支名
```
### 删除分支

```sh
git branch -d 要删除的分支名
```

若被删除的分支还有未被合并的内容，会有提示，无法删除，此时需要使用`-D`强行删除
```sh
git branch -D 要删除的分支名
```

### 切换分支

| 切换分支之前，首先得查看当前分支状态，看看有无未更改的提交，若不想提交，则需贮藏修改

- 已存在分支切换
```sh
git checkout  分支名
```
- 不存在分支，创建并切换
```sh
git checkout -b 分支名
```

### 恢复已被删除的分支
- 已合并的分支随意删除

- 未合并的分支若被删除

  - 使用 `git reflog` 删前的版本号，reflog 保留30天记录 
  - 使用 `git branch branchName version` 老创建新的分支恢复

### 合并
 概念 在A上合并B， A为当前分支，B为被合并分支

1. `merge` 合并 `git merge B`
  - master 主分支其余分支，使用快转模式直接合并
  - 子分支合并子分支，会产生一次新的commit来处理 
2. `rebase` 合并 `git rebase B`
  - 不会产生一次合并commit

### 合并冲突

- 文本冲突
  - 编辑冲突文件，确认到底保留哪方的内容
  - 然后  add → commit 操作

- 非文本冲突
  - 保留当前分支文件 `git checkout --ours 文件名`
  - 保留被合并分支文件 `git checkout --theirs 文件名`
  - 然后  add → commit 操作


## 贮藏
  有时，当你在项目的一部分上已经工作一段时间后，所有东西都进入了混乱的状态， 而这时你想要切换到另一个分支做一点别的事情。 问题是，你不想仅仅因为过会儿回到这一点而为做了一半的工作创建一次提交。 针对这个问题的答案是 `git stash` 命令。

### 创建贮藏

前提，被贮藏文件是被 暂存（add）的
- 贮藏当前目录全部文件
```sh
git stash
```
- 贮藏某个文件
```sh
git stash push filename
```
### 查看贮藏的东西
```sh
git stash list
```

### 恢复贮藏

- 不指定贮藏名，默认最近的
```sh
git stash apply
```
- 指定贮藏名
```sh
git stash apply stash@{2}
```
### 删除贮藏
- 直接删除
```sh
git stash drop stash@{2}
```
- 应用贮藏然后立即从栈上扔掉它
```sh
git stash pop stash@{2}
```
其余贮藏相关操作详见 [链接](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E8%B4%AE%E8%97%8F%E4%B8%8E%E6%B8%85%E7%90%86) 