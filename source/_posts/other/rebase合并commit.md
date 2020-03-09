---
title: git rebase合并commits
date: 2020-02-27 22:31:21
tags:
    - Git
    - rebase
    - commits合并
categories: 
    - other
---

# rebase作用

rebase主要和merge对比。相对于merge，rebase可以合并编辑commits历史，从而让更改逻辑一目了然。<!--more-->


# 使用示例

### 首先新建项目并从github上拉下来

```
 % git clone git@github.com:xxx/rebaseTest.git
正克隆到 'rebaseTest'...
remote: Enumerating objects: 3, done.
remote: Counting objects: 100% (3/3), done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
接收对象中: 100% (3/3), 完成.
检查连接... 完成。
```



###  新建分支并切换

```
 % git checkout -b test
切换到一个新分支 'test'
```



### 修改并提交多次

修改一次文件后执行下面的命令一次，message替换为自定义信息

```
git add README.md
git commit -m "message"
```



### 找到要合并的commits

```
 % git log --oneline
4254b18 11111
d024037 00000
f7de784 all messages
```

假设要将最上面两条合并

```
git rebase -i f7de784
```

将pick替换为e和s，分别表示编辑新的信息，合并之前的

```
e d024037 00000
s 4254b18 11111

# Rebase f7de784..4254b18 onto f7de784 (2 command(s))
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message

^G 求助       ^O Write Out  ^W 搜索       ^K 剪切文字   ^J 对齐
^X 离开       ^R 读档       ^\ 替换       ^U Uncut Text ^T 拼写检查
```



### 合并commit信息

```
# This is a combination of 2 commits.

2 merged messages

# 请为您的变更输入提交说明。以 '#' 开始的行将被忽略，而一个空的提交
# 说明将会终止提交。
# 
# 日期：  Thu Feb 27 22:19:08 2020 +0800
# 
# 交互式变基操作正在进行中；至 f7de784
# 最后一条命令已完成（2 条命令被执行）：
#    e d024037 00000
#    s 4254b18 11111
# 未剩下任何命令。
# 您在执行将分支 'test' 变基到 'f7de784' 的操作时编辑提交。
# 
# 要提交的变更：
#       修改：     README.md
```



### 切换到master分支并合并

```
% git checkout master
切换到分支 'master'
您的分支与上游分支 'origin/master' 一致。
will@will-Lenovo-ideapad-720S-14IKB /tmp/test/rebaseTest
 % git merge test
更新 f7de784..687e32e
Fast-forward
 README.md | 25 ++-----------------------
 1 file changed, 2 insertions(+), 23 deletions(-)
```



到github上可以看到master分支只有一条合并后的commit信息

![rebase.png](https://i.loli.net/2020/02/27/zFCWBo4NQarsmgL.png)

