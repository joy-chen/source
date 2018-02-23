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
