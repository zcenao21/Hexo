---
title: Git的使用
date: 2020-07-26 18:07:21
tags:
    - Git
    - 工具
    - 代码提交
    - 团队开发
categories: 
    - other
---



### Git是做什么的

Git（读音为/gɪt/）是一个开源的分布式版本控制系统，可以有效、高速地处理从很小到非常大的项目版本管理。Git 是 Linus Torvalds为了帮助管理 Linux 内核开发而开发的一个开放源码的版本控制软件[百度百科]。不得不感慨一下，Linus这位大佬是真的厉害，从0-1创造了没有的东西，还不止一个！据传言，这个软件是在两周完成，一个月投入使用的。。。

言归正传，通俗的说，Git的主要用途是用于版本控制，对于纯文本形式的文件尤其适合。非纯文本的比如视频就可以选择其他更好的工具来管理。而且Git的开发初衷是为了管理linux内核源码，也就是说是为代码版本控制量身定制的。

现在的互联网公司基本都用Git做代码版本控制，因为它可以高效完美地解决团队合作开发的问题。另外有各种代码仓库帮我们管理代码，比如Github，Gitlab。

Git的功能十分强大，本文对普通的开发者最常用的功能做了一些小总结。<!--more-->列表如下：

- 同步远程仓库到本地

- 修改并提交
- 回退
- 显示修改log
- 创建分支
- 合并分支
- 暂存修改及恢复
- 重放修改
- 打标签
- 命令简化
- 设置提交者信息
- 合并commit

下面依次进行介绍。为了演示操作，我们选择github创建新项目进行操作。如果没有帐号，请登陆github网站或使用搜索引擎搜索相关注册教程。



### 同步远程仓库到本地

首先我们在github上新建一个仓库，然后同步到本地

```
% git clone git@github.com:zcenao21/git-test.git
正克隆到 'git-test'...
remote: Enumerating objects: 3, done.
remote: Counting objects: 100% (3/3), done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
接收对象中: 100% (3/3), 完成.
检查连接... 完成。
```

命令是git clone xxx.git，ssh串可以通过点击图中code绿色块得到

![ssh.png](https://i.loli.net/2020/07/26/3xsm2vdWutOEe1n.png)



### 修改并提交

克隆到本地后，可以通过git add添加修改到暂存区，git commit命令给修改增加注释信息并添加到本地git仓库中

```
% cd git-test
% vim README.md
```

vim对README.md文件进行修改，修改完成后内容为

```
% cat README.md 
	git test
```

git status命令可以查看文件状态

```
% git status
    位于分支 master
    您的分支与上游分支 'origin/master' 一致。
    尚未暂存以备提交的变更：
      （使用 "git add <文件>..." 更新要提交的内容）
      （使用 "git checkout -- <文件>..." 丢弃工作区的改动）

        修改：     README.md

    修改尚未加入提交（使用 "git add" 和/或 "git commit -a"）
```

然后添加文件到本地仓库并注释

```
% git add README.md 
% git commit -m "first commit"
    [master 98a32a9] first commit
    1 file changed, 1 insertion(+), 2 deletions(-)

```

然后推送到远程仓库，这样就完成了一次完整的提交过程

```
% git push origin master
    对象计数中: 3, 完成.
    写入对象中: 100% (3/3), 260 bytes | 0 bytes/s, 完成.
    Total 3 (delta 0), reused 0 (delta 0)
    To git@github.com:zcenao21/git-test.git
       04ca6fe..98a32a9  master -> master

```



### 回退

如果改错了怎么办？比如我们修改了README.md，然而想回到上一次的提交，可以用git reset命令

```
% vim README.md 
% git commit -am "2nd line"
    [master 55ea9f4] 2nd line
     1 file changed, 1 insertion(+)
% cat README.md 
    git test
    git test 2nd line
```

首先我们修改了README.md，增加了 git test 2nd line这一行，然后提交到本地仓库，现在要回退到修改前的内容，即删掉新增第二行

```
% git reset --hard HEAD^
	HEAD 现在位于 d608dba 1st commit
% cat README.md 
 	git test
```

这样就回到了第一次提交的时候。如果要回到上上次呢？

```
git reset --hard HEAD^^
```

三次到更多次依次类推，更简单的提交方式是

```
git reset --hard HEAD~100
```

表示回到一百次前，估计应该不会用到



### 显示修改log

显示log需要用到git log命令

````
% git log
    commit d608dbac936d917c5e9d5d6b7176d68cea98a0b1
    Author: 张超 <zhangchao29@xiaomi.com>
    Date:   Sun Jul 26 21:43:48 2020 +0800

        1st commit

    commit 04ca6fec277e4da0a142add5826cf64b49fd2756
    Author: zcenao21 <buct_zc@163.com>
    Date:   Sun Jul 26 21:18:00 2020 +0800

        Initial commit
````

但是这样很不好看，有一个输出好看的log的命令

```
% git log --graph --pretty=oneline --abbrev-commit
    * d608dba 1st commit
    * 04ca6fe Initial commit
```

可以看到，d608dba是d608dbac936d917c5e9d5d6b7176d68cea98a0b1这一长串的缩写，是这次commit的唯一标识，非常有用。比如我们想回到初始的版本

```
% git reset --hard 04ca6fe
	HEAD 现在位于 04ca6fe Initial commit
% cat README.md 
    # git-test
    git测试
```

那么如果我们想撤销回退的内容怎么办？提交的内容已经在log中消失了！

```
% git log --graph --pretty=oneline --abbrev-commit
	* 04ca6fe Initial commit
```

git reflog命令可以看到所有的历史修改

```
% git reflog
    04ca6fe HEAD@{0}: reset: moving to 04ca6fe
    d608dba HEAD@{1}: reset: moving to HEAD^
    b2e751e HEAD@{2}: commit: 2nd line
    。。。
    98a32a9 HEAD@{11}: commit: first commit
    04ca6fe HEAD@{12}: clone: from git@github.com:zcenao21/git-test.git
```

然后我们执行如下命令

```
% git reset --hard b2e751e
    HEAD 现在位于 b2e751e 2nd line
    will@will-Lenovo-ideapad-720S-14IKB ~/study/projects/git-test
% cat README.md 
    git test
    git test 2nd line
```

可以看到，git reset可以回退到任意版本



### 创建分支

分支非常重要，是团队合作的利器。每个团队成员从主分支拉取一个分支，然后并行开发各自的模块，最终合并到主分支中去。这样既保持了开发的独立性，又实现了团队合作。创建分支示例如下：

```
% git checkout -b dev
```

dev为分支名。



### 合并分支

假如在分支上做了修改，如何合并到主分支呢？示例如下：

```
% vim README.md 
% git commit -am "3rd revise"
    [dev b4ed33e] 3rd revise
     1 file changed, 1 insertion(+)
% cat README.md 
    git test
    git test 2nd line
    git test 3rd line
% git checkout master
    切换到分支 'master'
    您的分支和 'origin/master' 出现了偏离，
    并且分别有 2 和 1 处不同的提交。
      （使用 "git pull" 来合并远程分支）
% git merge dev 
    更新 b2e751e..b4ed33e
    Fast-forward
     README.md | 1 +
     1 file changed, 1 insertion(+)
```



### 暂存修改及恢复

什么时候需要暂存呢？比如我们正在一个分支dev开发，来了一个新需求，这是我们在dev的开发还没做完，直接合并不行，会导致项目出错，其他人就没办法继续编译开发了。暂存可以将从上次提交完开始到现在做的修改暂时保存起来，之后完成了其他需求再恢复这个暂存项。

```
% vim README.md 
% cat README.md 
    git test
    git test 2nd line
    git test 3rd line.
% git status
    位于分支 dev
    尚未暂存以备提交的变更：
      （使用 "git add <文件>..." 更新要提交的内容）
      （使用 "git checkout -- <文件>..." 丢弃工作区的改动）

        修改：     README.md

    修改尚未加入提交（使用 "git add" 和/或 "git commit -a"）
% git stash
    Saved working directory and index state WIP on dev: b4ed33e 3rd revise
    HEAD 现在位于 b4ed33e 3rd revise
% git status
    位于分支 dev
    无文件要提交，干净的工作区
```

上面的操作是先修改dev分支的readme文件（最后一行新增.号），然后暂存操作，可以发现现在的dev分支为干净状态，恢复到了上次提交的状态。恢复暂回内容的命令为git stash pop

```
% cat README.md 
    git test
    git test 2nd line
    git test 3rd line
% git stash pop
    位于分支 dev
    尚未暂存以备提交的变更：
      （使用 "git add <文件>..." 更新要提交的内容）
      （使用 "git checkout -- <文件>..." 丢弃工作区的改动）

        修改：     README.md

    修改尚未加入提交（使用 "git add" 和/或 "git commit -a"）
    丢弃了 refs/stash@{0} (e47bf4e261b238728cefd414a1fa7948adec5265)
% cat README.md 
    git test
    git test 2nd line
    git test 3rd line.
```

我们也可以多次进行暂存，恢复的命令如下

```
git stash apply xxx //恢复某次stash，xxx可以用git stash list查看
```



### 重放修改

如果在某个分支上做了修改，但是想把这些修改同时应用到其他分支上，即“重放”修改。下面的示例是dev领先master分支一个commit，要在master分支上重放dev分支上的最后一次commit，使用git cherry-pick操作即可

```
% git diff dev master 
    diff --git a/README.md b/README.md
    index 88c9632..9cead72 100644
    --- a/README.md
    +++ b/README.md
    @@ -1,3 +1,3 @@
    -git test
    + git test
     git test 2nd line
     git test 3rd line
% git log --graph --pretty=oneline --abbrev-commit
    * a2b2c9a remove first blank
    * b4ed33e 3rd revise
    * b2e751e 2nd line
    * d608dba 1st commit
    * 04ca6fe Initial commit
% git checkout master
    切换到分支 'master'
    您的分支和 'origin/master' 出现了偏离，
    并且分别有 3 和 1 处不同的提交。
      （使用 "git pull" 来合并远程分支）
 % git cherry-pick a2b2c9a
    [master a8f25bd] remove first blank
     Date: Mon Jul 27 00:46:02 2020 +0800
     1 file changed, 1 insertion(+), 1 deletion(-)
% git log --graph --pretty=oneline --abbrev-commit
    * a8f25bd remove first blank
    * b4ed33e 3rd revise
    * b2e751e 2nd line
    * d608dba 1st commit
    * 04ca6fe Initial commit
% git diff a8f25bd b4ed33e
    diff --git a/README.md b/README.md
    index 88c9632..9cead72 100644
    --- a/README.md
    +++ b/README.md
    @@ -1,3 +1,3 @@
    -git test
    + git test
     git test 2nd line
     git test 3rd line
```



### 打标签

标签和commit很类似，标签常用来管理版本号。commit到一定数量，一个功能开发完成，这时候就可以打一个tag。github还会贴心的为tag打包可以直接下载。

```
% git tag v1.0
% git tag
	v1.0
```

删除及其它操作如下：

```
git tag -a v2.0 -m "move ahead!" 4225b7d  //给标签加上备注
git tag -d v1.0 //删除标签
git push origin :refs/tags/v0.9 //删除远程标签。先删除本地，git tag -d 命令，然后执行此命令删除远程
```



### 命令简化

刚刚的git log后跟的一长串有没有让你留下深刻的印象？Git提供了一种快捷方式。比如缩写git log ...

```
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```

之后我们就可以使用git lg这个命令了



### 设置提交者信息

```
git config --global user.name "名字"
git config --global user.email "邮箱"
```

这是针对全局的设置，如果对于某个仓库想单独设置，可以修改.git/config文件

```
[user]   
	name = XXX(自己的名称英文)   
	email = XXXX(邮箱)
```



### 合并commit

参考{% post_link other/rebase合并commit git rebase%}











