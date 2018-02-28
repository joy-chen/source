---
title: git常用命令
date: 2017-12-22 17:03:01
tags:
---

<!-- excerpt -->

git checkout -b nativeBranchName remoteBranchPath 切换远程分支到本地
git diff --staged 查看未提交的更改

### 修改上次提交
git add <filename> # 或者 git rm
git commit --amend # 将缓存区的内容做为最近一次提交

### 撤销修改
git checkout filename #回退为提交的文件修改

### 撤销提交分支
git reset --hard xxx
git push --force

### 分支提交
git push origin branch

### 查看git库信息
git remote -v

### 分支合并
git merge origin/xxx branch
