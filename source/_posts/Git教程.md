---
title: Git教程
date: 2018-08-21 21:03:48
categories: "工具"
tags:
     - git
---
工欲善其器, 必先利其器.
今天给项目组的小伙伴们分享了 git 的使用, 所以就顺便再分享给大家.

<!-- more -->

## Git 的诞生
很多人都知道，Linus在1991年创建了开源的Linux，从此，Linux系统不断发展，已经成为最大的服务器系统软件了。
Linus虽然创建了Linux，但Linux的壮大是靠全世界热心的志愿者参与的，这么多人在世界各地为Linux编写代码，那Linux的代码是如何管理的呢？
事实是，在2002年以前，世界各地的志愿者把源代码文件通过diff的方式发给Linus，然后由Linus本人通过手工方式合并代码！
不过，到了2002年，Linux系统已经发展了十年了，代码库之大让Linus很难继续通过手工方式管理了，社区的弟兄们也对这种方式表达了强烈不满，于是Linus选择了一个商业的版本控制系统BitKeeper，BitKeeper的东家BitMover公司出于人道主义精神，授权Linux社区免费使用这个版本控制系统。
安定团结的大好局面在2005年就被打破了，原因是Linux社区牛人聚集，不免沾染了一些梁山好汉的江湖习气。开发Samba的Andrew试图破解BitKeeper的协议（这么干的其实也不只他一个），被BitMover公司发现了（监控工作做得不错！），于是BitMover公司怒了，要收回Linux社区的免费使用权。
Linus可以向BitMover公司道个歉，保证以后严格管教弟兄们，嗯，这是不可能的。实际情况是这样的：
Linus花了两周时间自己用C写了一个分布式版本控制系统，这就是Git！一个月之内，Linux系统的源码已经由Git管理了！牛是怎么定义的呢？大家可以体会一下。
Git迅速成为最流行的分布式版本控制系统，尤其是2008年，GitHub网站上线了，它为开源项目免费提供Git存储，无数开源项目开始迁移至GitHub，包括jQuery，PHP，Ruby等等。

## Git 是什么

**分布式版本控制系统**

### 版本控制系统

记录一个或若干个内容变化, 以便将来查阅特定版本修订情况的系统

### 集中式与分布式对比

![centralization](/images/centralization.png)
![distributed](/images/distributed.png)

## Git 的安装

* Windows: git bash
* Linux: apt-get install git
* ac: 通过homebrew安装Git

配置:
* $ git config --global user.name "Your Name"
* $ git config --global user.email "email@example.com"
注: 可以针对不同的 repo 配置不同的用户名及邮箱

## Git 版本库的初始化

```
git init
```
Or
```
git clone xx@xxxx
```
![version_repository](/images/version_repository.png)

## Git 的常见操作

```
git status
要查看哪些文件处于什么状态
```

```
git add <file>
git add .
暂存已修改文件
```

```
git reset HEAD <file>
假如你修改了两个文件并且想要将它们作为两次独立的修改提交，但是却意外地输 入了 git add . 暂存了它们两个，此时取消一个文件<file>的暂存用git reset HEAD <file>
```

```
git branch
查看本地分支

git branch -a
查看本地+远程分支

git branch 分支名
创建本地分支

git branch -D 分支名
删除本地分支
```

```
git checkout 分支名
切换分支

git checkout -b 分支名
基于当前分支创建一个分支, 并切换到当前分支
(git branch 分支名 + git checkout 分支名)
```

```
git diff
查看工作区文件更改差异

git diff --staged
查看暂存区文件差异
```

```
git commit -m <commit message>
提交更新

git commit --amend
修改上次提交

建议:
先用 git status 看下，是不是修改都已暂存起来了
commit 格式: 模块: 内容
```

```
git checkout -- <file>
如果你并不想保留对文件<file>的修改，你可以将它还原成上次提交时的样子，即撤销对<file>的更改（不可恢复）。

git reset HEAD^
撤销上次提交 
```

```
git log
查看提交历史

git log -p -2
一个常用的选项是 -p，用来显示每次提交的内容差异。 你也可以加上 -2 来仅显示最近两次提交
```

```
git push
推送到远程仓库

git pull 
 例如:git pull origin develop:branch 
取回远程 develop 分支，与本地branch 分支合并。省略:branch 默认和当前分支合并
 实质上，git pull = git fetch + git merge

git fetch
下载远端的版本库
```

```
git merge <branch>
与branch分支合并，git merge origin/<branch> 与远程分支合并。  

git rebase <branch>
rebase 意思为变基。git rebase origin/<branch>与远程分支合并 
```

### git rebase 和 git merge 的区别

![merge_rebase](/images/merge_rebase.png)

#### git rebase

```
git rebase --continue
在rebase的过程中，也许会出现冲突(conflict). 在这种情况，Git会停止rebase并会让你去解决 冲突；在解决完冲突后，用"git-add"命令去更新这些内容的索引(index), 然后，你无需执行 git-commit
这样git会继续应用余下的补丁

 git rebase --abort
在任何时候，你可以用--abort参数来终止rebase的行动，并且"mywork" 分支会回到rebase开始前的状态。
```

#### git merge

```
使用merge出现冲突时，会出现 CONFLICT ，后面提示冲突的文件，解决所有冲突后，使用git add . 将已解决的冲突添加到暂存并提交。
如有需要，再使用git commit --amend 把暂存区的改动提交到上一次提交中。  
```

### commit 的合并

```
git rebase -i HEAD~2
```
使用此命令来合并多个提交，HEAD~2 中的2表示将最近的两次提交合并成一个提交。
使用后会出现类似如下界面：  
```
pick f7f3f6d changed my name a bit
pick 310154e updated README formatting and added blame   
```
将第二个 pick 改为 squash 或 s 后保存退出，即将两次提交合成一个。
或者使用 git reset <sha> 回退到某一次提交后，重新 git commit -m <commit message> 可以实现相同的效果。

### 二分法快速定位问题代码

```
git bisect start
git bisect bad
git bisect good a67f7
git bisect reset
```

## 总结: GIT 的通用流程

```
git clone
git  checkout -b <分支名>
git add .
git commit -m “”
git push origin <分支名>

git fetch
git rebase origin/dev
git add .
git rebase --continue
git push origin <分支名>
```
