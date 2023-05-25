title: Git常用命令
author: Funny Boy
date: 2023-05-25 11:11:14
tags:
---
# 前言
假设：
项目分支是dev、master
项目地址是https://github.com/752841728/752841728.github.io.git

---
# 一、克隆项目
## 1.无本地仓库，有远程仓库

```powershell
git clone https://github.com/752841728/752841728.github.io.git
```
## 2.有本地仓库，有远程仓库

```powershell
git remote rm origin
git remote add origin https://github.com/752841728/752841728.github.io.git
```
# 二、拉取提交
## 1.拉取
本地dev拉取远程dev：

```powershell
git pull origin dev
```
## 2.提交
本地dev提交远程dev：

```powershell
git add .
git commit -m "fix: 修复BUG"
git push origin dev
```
## 3.关联分支
本地dev关联远程dev：

```powershell
git branch --set-upstream-to=origin/dev dev
```
```powershell
git push -u origin dev // 通过提交本地dev关联远程dev
```

关联分支后的命令可以简写：

```powershell
git pull
git push
```
# 三、分支

查看本地已有分支：

```powershell
git branch
```
创建本地master分支：
```powershell
git branch master
```
```powershell
git checkout -b master // 创建新分支并切换到新分支
```
删除本地master分支：

```powershell
git branch -D master
```
本地dev合并本地master：

```powershell
git checkout master // 切换到本地master分支
git merge dev // 将本地dev分支合并到本地master分支
git cherry-pick commit1ID commit2ID // 只合并本地dev分支commit1和commit2的提交到本地master分支
```
# 三、回滚

## 1.撤销某个提交并生成新的提交

```powershell
git revert -n commitID // commit的提交后面的提交记录会保留
git commit -m "feat: 撤销commitID的提交"
git pull
git push
```
## 2.回滚到某个提交

```powershell
git reset --hard commitID // commit的提交后面的提交记录不会保留
git commit -m "feat: 回滚到commitID的提交"
git pull
git push -f // 本地仓库与远程仓库不一致，需要强制提交
```
## 3.撤回本地commit

```powershell
git reset --soft HEAD^
```

---

# 总结
遵守Git的流程和规范，养成代码好习惯