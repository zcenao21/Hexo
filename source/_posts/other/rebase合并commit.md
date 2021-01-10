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

rebase翻译过来是变基，主要作用有两个

- **合并代码不会有冲突，因为在rebase过程中已经解决了**。程序员很多时候需要协同开发，假设有如下场景，分别有开发的程序猿A和程序员B，以及他们的leader来合并A和B的代码。程序猿A和B同时从master分支拉代码并新建为各自的分支，之后程序猿A和B同时对一个文件做了修改，这在日常coding中非常常见。这时如果A的代码由leader合并到了master分支，B在A合并之后提交自己的修改。由于对同一个文件做了修改，leader必须处理完冲突之后才能将B的代码合并。通过rebase，还是上述的过程，不过这次B这次做了rebase到master分支的操作，在这个过程中就已经处理完冲突。由于B对于自己的修改是很清楚的，由他来处理冲突更合适。leader就不需要处理冲突了。
- **让提交信息更加清晰**。在开发过程中，我们经常会保存修改的代码到程序仓库(如github, gitlab)。在保存之前会有一个commit信息，很多人都是随便写这个commit信息，而合并代码之后这些commit信息会现实在修改历史中，很多时候这些信息都是没必要的。假如有多个程序猿同时开发，这些commit信息就会多如牛毛，之后看修改信息就很困难了。其实只需要把所有修改的内容总结到一个commit信息中即可，这样提交的代码修改逻辑就十分清晰。rebase可以实现合并commit信息的功能。

<!--more-->

从网上盗张图，侵删

![rebase](https://i.loli.net/2020/04/12/alxyPthifAuvTb3.png)

从图中很明显的可以看出，变基之后相当于你从master分支最后的修改处继续做了改动。






# 使用示例

### 首先新建项目并从github上拉下来

```
git clone git@github.com:zcenao21/rebaseTest.git
正克隆到 'rebaseTest'...
remote: Enumerating objects: 3, done.
remote: Counting objects: 100% (3/3), done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
接收对象中: 100% (3/3), 完成.
检查连接... 完成。
```



###  新建分支

分别为程序猿A，B和leader的分支，A和B分别做了修改然后提交到leader分支作为阶段性的分支。

```
% git checkout -b apeA origin/master
分支 apeA 设置为跟踪来自 origin 的远程分支 master。
切换到一个新分支 'apeA'
will@will-Lenovo-ideapad-720S-14IKB ~/study/projects/rebaseTest
 % git checkout -b apeB origin/master
分支 apeB 设置为跟踪来自 origin 的远程分支 master。
切换到一个新分支 'apeB'
will@will-Lenovo-ideapad-720S-14IKB ~/study/projects/rebaseTest
 % git checkout -b leader origin/master
分支 leader 设置为跟踪来自 origin 的远程分支 master。
切换到一个新分支 'leader'
```



### 修改并提交多次

程序猿A和B分别做了两次修改，而且都是修改的READ.md文件

![image.png](https://i.loli.net/2020/04/12/6ZE9Lkne4vfxisD.png)

![image.png](https://i.loli.net/2020/04/12/CPpraVdR1lIAE7g.png)



### 合并程序猿A的修改

leader合并完A的修改后（假装有三个人的样子），leader分支commit信息如下

![image.png](https://i.loli.net/2020/04/12/DwsLNIZdr6EBOYJ.png)



### 合并程序猿B的修改

程序猿B提出merge到leader分支的请求，哦哟，有冲突！

![image.png](https://i.loli.net/2020/04/12/wQdYqLaGxEPsFe4.png)

leader看了一下B修改的内容和B的是不同内容，需要解决冲突后合并

解决冲突前：

![image.png](https://i.loli.net/2020/04/12/k7eIldPu3Fwnh9Q.png)

解决冲突后：

![image.png](https://i.loli.net/2020/04/12/QAomr4j5e9FWMBL.png)



合并到leader分支之后

![image.png](https://i.loli.net/2020/04/12/FGXkKUIOL9wRzn3.png)

哦哟，A和B只是修改了一点点代码（其实就是划水改了一下READ.md），就这么多commit信息了。leader过了几天发现代码有Bug，然后看看修改历史，头有点疼。。。而且这只是A和B各只commit两次的结果。



### rebase闪亮登场

rebase旁白：让我来干掉这些乱七八糟的修改历史吧！

一顿操作如下：

```
% git rebase -i leader-rebase
```

进入如下修改页面（默认nano编辑器，可以修改为vim，不赘述）。将提交信息8ffeb8e那行前面的pick改为s，表示第二行的信息合并到第一行中去，commit信息之后可以编辑。

ps: s是squash的缩写，表示使用commit信息但是合并到前一条commit信息中。其他的选项也都有解释，不赘述。

![image.png](https://i.loli.net/2020/04/12/DoeBx9jqRvy25hp.png)

^x离开，选择保存修改信息，Y

![image.png](https://i.loli.net/2020/04/12/CxJTn6gi9VU4PIo.png)

然后就是合并commit信息，就像编辑文本那样编辑就行

![image.png](https://i.loli.net/2020/04/12/AajJzqU8iXEmOTh.png)

编辑完如下图，保存。A的rebase工作顺利完成。然后让leader合并。

![image.png](https://i.loli.net/2020/04/12/u2MPf8QYrhtDbGk.png)

```
 % git rebase -i leader-rebase
[分离头指针 c7d70d0] apeA: 1st commit
 Date: Sun Apr 12 00:23:10 2020 +0800
 1 file changed, 5 insertions(+)
Successfully rebased and updated refs/heads/apeA.
```

切换到leader-rebase分支，然后merge

```
 % git merge apeA
更新 5966a99..c7d70d0
Fast-forward
 README.md | 5 +++++
 1 file changed, 5 insertions(+)
```

下面是B的rebase操作了，和A类似的一顿操作。然鹅，出现了如下错误：

```
% git rebase -i leader-rebase 
error: 不能应用 3a09941... apeB: first commit

当您解决了此问题后，执行 "git rebase --continue"。
如果您想跳过此补丁，则执行 "git rebase --skip"。
要恢复原分支并停止变基，执行 "git rebase --abort"。
Could not apply 3a09941bc634a59c6f3c238c8352fc7175834a63... apeB: first commit
```

这是因为和A的代码冲突了。稳住，解决冲突。根据提示信息

```
% git rebase --continue
README.md: needs merge
您必须编辑所有的合并冲突，然后通过 git add
命令将它们标记为已解决
```

然后使用编辑器编辑README.md文件，修改完成后如下：

![image.png](https://i.loli.net/2020/04/12/chXH6Fdy5AziRu4.png)

这只是处理了apeB第一次的commit信息，接下来处理第二次的commit信息

```
git add README.md
```

![image.png](https://i.loli.net/2020/04/12/OnpoFCRrtLANckD.png)

修改完commit信息后如下：

![image.png](https://i.loli.net/2020/04/12/WtB3lLXKiko7mRY.png)

保存之后，出现如下信息！rebase成功

```
% git rebase --continue
[分离头指针 abb911a] apeB: first commit
 1 file changed, 4 insertions(+)
[分离头指针 5deef87] apeB: 1st commit
 Date: Sun Apr 12 00:28:26 2020 +0800
 1 file changed, 7 insertions(+)
Successfully rebased and updated refs/heads/apeB.
```

切换到leader-rebase分支，合并

```
% git merge apeB
更新 c7d70d0..5deef87
Fast-forward
 README.md | 7 +++++++
 1 file changed, 7 insertions(+)
```

![image.png](https://i.loli.net/2020/04/12/Q7f5v4XF2bAtPCZ.png)

perfect！清爽！和谐！
