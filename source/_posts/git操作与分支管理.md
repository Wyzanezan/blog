---
title: git操作与分支管理
date: 2020-06-21 21:38:08
tags: Git
categories: Python
---

git 是一个版本控制系统，现在大部分项目都会使用它来管理开发的项目，下面整理了 git 的常用命令及使用方式。

<!--more-->

# Git基础

## git最小配置

配置 username 和 email

```txt
git config --global user.name 'your name'
git config --global user.email 'your email'
```

配置 email 的目的，当版本库中的代码变更时，会记录对应开发人员的 email 等信息，当进行 code review 时，若相应变更的代码有问题，会发送邮件通知到对应的开发者。



config的 三个作用域：

```txt
git config --local  缺省配置，表示只对某个仓库有效
git config --global 对当前用户所有仓库有效
git config --system 对系统所有的登录用户有效
```

当我们需要显示 config 的配置信息时，使用 --list 就行了：

```txt
git config --list --local
git config --list --global
git config --list --system
```

例如：

```txt
λ git config --list --global
user.email=wyzane1207@163.com
user.name=wyzane
```

```txt
λ git config --list
core.symlinks=false
core.autocrlf=true
core.fscache=true
color.diff=auto
color.status=auto
color.branch=auto
color.interactive=true
help.format=html
rebase.autosquash=true
http.sslbackend=openssl
http.sslcainfo=D:/software/Git/mingw64/ssl/certs/ca-bundle.crt
credential.helper=manager
user.email=wyzane1207@163.com
user.name=wyzane
```



## 创建仓库

### 把已有项目代码纳入版本管理

```txt
cd 项目代码所在文件夹
git init
```

### 新建一个git仓库

```txt
cd 某个文件夹
git init 目录名
cd 目录名
```

新建一个git仓库的例子如下：

```txt
λ git init demo1
Initialized empty Git repository in F:/GitTest/demo1/.git/
```

此时执行 git config --global --list 的结果如下：

```txt
λ git config --global --list
user.email=wyzane1207@163.com
user.name=wyzane
```

在 demo1 仓库中，执行如下配置：

```txt
λ git config --local user.name 'wz'
λ git config --local user.email 'wyzane1207@gmail.com'
```



## 添加一个文件

在 dem01 目录下，新建文件 readme.md，此时执行 git status，显示如下内容：

```txt
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        readme.md

nothing added to commit but untracked files present (use "git add" to track)
```

此时显示，readme.md 文件是未被跟踪的文件，执行 git add readme.md，此时再执行 git status，显示如下：

```txt
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

        new file:   readme.md
```

然后执行 git commit -m 'readme.md'，表示将文件提交到本地仓库中，执行完，显示结果如下：

```txt
λ git commit -m 'reaeme.md'
[master (root-commit) 28f3903] 'reaeme.md'
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 readme.md
```

此时，可以用 git log 命令查看提交记录：

```txt
λ git log
commit 28f3903644aea0102c9e7b7a6789b5990a45db50 (HEAD -> master)
Author: wz <wyzane1207@163.com>
Date:   Mon Jun 22 22:41:14 2020 +0800

    'reaeme.md'
```

从上面的结果中可以看出 local 配置会覆盖 global 配置。



## 往仓库里添加文件

上面使用的 git add、git commit 等命令的功能如下：

```txt
git status: 查看工作目录和暂存区的状态
git add: 将工作目录的改动添加到暂存区
	git add -u：将工作区中，已经被git管理的文件添加到暂存区（没有被管理的则不会添加到暂存区）
git commit: 将暂存区的高的那个添加到本地仓库
git log: 查看当前分支的本地 commit 的历史记录 （查看所有分支的提交记录使用 git log --all）
```

git log 显示结果如下：

```txt
λ git log
commit 683cbacc9aae84b55bf03a37217f09b23ec71b5a (HEAD -> master)
Author: wz <wyzane1207@163.com>
Date:   Tue Jun 23 22:50:22 2020 +0800

    update

commit 9548167eda1557775206f988e129625c8ebc664e
Author: wz <wyzane1207@163.com>
Date:   Tue Jun 23 22:41:51 2020 +0800

    add index

commit 28f3903644aea0102c9e7b7a6789b5990a45db50
Author: wz <wyzane1207@163.com>
Date:   Mon Jun 22 22:41:14 2020 +0800

    add readme
```



## 文件重命名

```txt
文件的重命名可以在工作目录中完成，然后添加到暂存区，再提交到本地仓库。这样的方式有如下问题：
F:\GitTest\demo1 (master -> origin)
λ mv index2.html index02.html

F:\GitTest\demo1 (master -> origin)
λ git status
On branch master
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        deleted:    index2.html

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        index02.html

no changes added to commit (use "git add" and/or "git commit -a")

从上面的显示可以看出，在工作目录中重命名是先删除文件，再创建新文件。然后还需要执行以下命令：
git add index02.html
git rm index2.html  删除暂存区中的文件
执行 git status 打印如下：
λ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        renamed:    index2.html -> index02.html
```

把暂存区中所有的修改还原：

```txt
git reset --hard
```

上面的更改文件名的操作，可以使用下面的命令直接修改暂存区：

```txt
git mv index.html index02.html

此时再执行 git status，结果如下：
λ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        renamed:    index2.html -> index02.html
        
这个结果，跟上面的结果一致。 
```



## 查看版本历史

我们可以使用 git log 命令查看版本历史，git log 命令还有以下几种用法：

```txt
1. git log --oneline  查看简洁的 log 信息
febad01 (HEAD -> master) 修改文件名
683cbac update
9548167 add index
28f3903 add readme

2. git log -n2  使用 -n 加数字的方式查看最近的几次提交 或者 git log -n2 --oneline
commit febad01c1b094e73047b204fd31b827cb2a86ff2 (HEAD -> master)
Author: wz <wyzane1207@163.com>
Date:   Tue Jun 23 23:02:37 2020 +0800

    修改文件名

commit 683cbacc9aae84b55bf03a37217f09b23ec71b5a
Author: wz <wyzane1207@163.com>
Date:   Tue Jun 23 22:50:22 2020 +0800

    update
    
3. git log --all --graph  使用图形化的形式展示分支的提交父子关系


上面的所有参数都可以组合使用，还可以使用 git help --web log 查看更多 git log 参数。
```

使用 gitk 命令可以打开图形界面，查看项目的版本历史。



## .git目录介绍

执行 git init 后，会生成一个 ,git 文件夹，文件夹内容如下：

```txt
-rw-r--r-- 1 dell 197121  16 6月  23 23:02 COMMIT_EDITMSG
-rw-r--r-- 1 dell 197121 180 6月  22 22:29 config
-rw-r--r-- 1 dell 197121  73 6月  22 22:24 description
-rw-r--r-- 1 dell 197121 216 6月  25 14:55 gitk.cache
-rw-r--r-- 1 dell 197121  21 6月  24 23:07 HEAD
drwxr-xr-x 1 dell 197121   0 6月  22 22:24 hooks/
-rw-r--r-- 1 dell 197121 217 6月  24 23:08 index
drwxr-xr-x 1 dell 197121   0 6月  22 22:24 info/
drwxr-xr-x 1 dell 197121   0 6月  22 22:41 logs/
drwxr-xr-x 1 dell 197121   0 6月  23 23:02 objects/
-rw-r--r-- 1 dell 197121  41 6月  23 22:59 ORIG_HEAD
drwxr-xr-x 1 dell 197121   0 6月  22 22:24 refs/
```

每个文件的含义如下：

```txt
HEAD: 文件，保存了当前正在工作的分支，切换分支时内容会改变，内容为：ref: refs/heads/master
config: 文件，保存了与本地仓库相关的配置信息，内容如下：
	[core]
        repositoryformatversion = 0
        filemode = false
        bare = false
        logallrefupdates = true
        symlinks = false
        ignorecase = true
	[user]
        name = 'wz'
        email = 'wyzane1207@163.com'
refs: 目录，包含 heads(保存分支) 和 tags(保存标签)
	refs/heads/master 中保存的是一个 commit 的 id，指向一个 commit
objects: 目录，保存了 tree、glob 等对象的信息
```



## git对象之间的关系

commit、tree、blob三者之间的关系如下：

```txt
一个commit对应一个tree，这个tree表示当时commit后，项目的整个目录结构，这个tree下面有可以包含多个子tree(目录)和blob(文件)。
子tree会对应项目中的一个目录，子tree中也包含很多blob和子tree。
一个blob对应一个文件，只要文件内容相同，就是同一个blob对象。
```

执行如下命令：

```txt
λ git cat-file -p f9c5d761a  查看f9c5d761a的内容
tree b4f5e1498a011eb1267852ff76ccbad35c08aac4
parent febad01c1b094e73047b204fd31b827cb2a86ff2
author wyzane <wyzane1207@163.com> 1593072058 +0800
committer wyzane <wyzane1207@163.com> 1593072058 +0800

添加用户页面

F:\GitTest\demo1 (master -> origin)
λ git cat-file -t f9c5d761a  查看f9c5d761a的类型
commit

F:\GitTest\demo1 (master -> origin)
λ git cat-file -p b4f5e1498a011   查看f9c5d761a下tree对象b4f5e1498a011的内容
100644 blob fd98613334e79fb5a12f537462627af2b7efdacd    index.html
100644 blob 9b562d265a03e923b86bfc3b9b8418463f4f1c70    index02.html
100644 blob e69de29bb2d1d6434b8b29ae775ad8c2e48c5391    readme.md
040000 tree fa6ab77bfb3a6f0cbecd74ef7e98e8624ee4c099    test

F:\GitTest\demo1 (master -> origin)
λ git cat-file -t b4f5e1498a011  查看b4f5e1498a011的类型
tree

F:\GitTest\demo1 (master -> origin)
λ git cat-file -p fa6ab77bfb3
100644 blob 41d4171c5880f9cdab07eadc8564d04e7ee1e055    user.html

从上面的打印信息中可以看出，一个commit对应一个tree，这个tree下面可以有blob对象和子tree对象，子tree下面还有blob对象。

执行 git cat-file -p fd98613334e79fb，显示出 blob 对象的内容如下：
<h1>git test<h1/>
<h1>git test<h1/>
<h1>git test<h1/>
```

## 分离头指针

```txt
分离头指针是指，做的变更不是基于一个分支。
HEAD 可以指向一个分支（最终还是指向一个commit），也可以指向一个commit，当使用 git checkout commitID 时，HEAD会切换到一个特定的commit。此时我们做的任何变更不是基于分支的，而是基于这个commit，当切换回其他分支时，在commit上做的变更就会被git丢弃掉。
当我们需要做一些实验性的变更时，可以这样做。
```

正常情况下，HEAD会指向一个分支：

```txt
λ git log
commit f9c5d761a8922dd69345da752ab135881418bc51 (HEAD -> master)
Author: wyzane <wyzane1207@163.com>
Date:   Thu Jun 25 16:00:58 2020 +0800

    添加用户页面

commit febad01c1b094e73047b204fd31b827cb2a86ff2
Author: wz <wyzane1207@163.com>
Date:   Tue Jun 23 23:02:37 2020 +0800

    修改文件名

commit 683cbacc9aae84b55bf03a37217f09b23ec71b5a
Author: wz <wyzane1207@163.com>
Date:   Tue Jun 23 22:50:22 2020 +0800

    update

commit 9548167eda1557775206f988e129625c8ebc664e (temp)
Author: wz <wyzane1207@163.com>
Date:   Tue Jun 23 22:41:51 2020 +0800

    add index
```

我们执行 git checkout 9548 febad01c1b 后，HEAD会指向一个 commit：

```txt
λ git checkout febad01c1b
Note: checking out 'febad01c1b'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b <new-branch-name>

HEAD is now at febad01 修改文件名

λ git log
commit febad01c1b094e73047b204fd31b827cb2a86ff2 (HEAD)
Author: wz <wyzane1207@163.com>
Date:   Tue Jun 23 23:02:37 2020 +0800

    修改文件名

commit 683cbacc9aae84b55bf03a37217f09b23ec71b5a
Author: wz <wyzane1207@163.com>
Date:   Tue Jun 23 22:50:22 2020 +0800

    update

commit 9548167eda1557775206f988e129625c8ebc664e (temp)
Author: wz <wyzane1207@163.com>
Date:   Tue Jun 23 22:41:51 2020 +0800

    add index
```

## 修改commit的msg

```txt
修改最近一次 commit 的 msg 使用如下命令：
git commit --amend

如何修改其他 commit 的 msg
```

## 修改其他 commit 的 msg

对其他 commit 的 msg 修改时，需要用到 git rebase 命令。
首先我们要确定待修改commit的上一个commitID，以这个commit为基础。然后执行 git rebase -i 上一个commitID，会进入一个编辑页面，内容如下：

```txt
pick f5b4cb5 修改文件名
pick a0b86da 添加用户页面2

# Rebase 683cbac..a0b86da onto 683cbac (2 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup <commit> = like "squash", but discard this commit's log message
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
# .       create a merge commit using the original merge commit's
# .       message (or the oneline, if no original merge commit was
# .       specified). Use -c <commit> to reword the commit message.
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

上面的文件中，挑选出了两个 commit，然后下面有选择执行的命令。修改 commit 时，我们可以选择 commit message 命令，即把第一行的 pick 改成 reword，然后保存。保存后又会进入一个编辑页面：

```txt
修改文件名

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Author:    wz <wyzane1207@163.com>
# Date:      Tue Jun 23 23:02:37 2020 +0800
#
# interactive rebase in progress; onto 683cbac
# Last command done (1 command done):
#    reword f5b4cb5 修改文件名2
# Next command to do (1 remaining command):
#    pick a0b86da 添加用户页面2
# You are currently editing a commit while rebasing branch 'master' on '683cbac'.
#
# Changes to be committed:
#       renamed:    index2.html -> index02.html
```

这时我们修改完 msg 后保存即可。



## 合并commit

对其他 commit 的 msg 修改时，需要用到 git rebase 命令。
首先我们要确定待修改commit的上一个commitID，以这个commit为基础。然后执行 git rebase -i 上一个commitID，会进入一个编辑页面，内容如下：

```txt
pick 9548167 add index
pick 683cbac update
pick 4a558f9 修改文件名2
pick 8d1455b 添加用户页面2

# Rebase 28f3903..8d1455b onto 28f3903 (4 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup <commit> = like "squash", but discard this commit's log message
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
# .       create a merge commit using the original merge commit's
# .       message (or the oneline, if no original merge commit was
# .       specified). Use -c <commit> to reword the commit message.
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

我们可以把 683cbac、4a558f9 这两个 commit 合并到第一个 commit 中。根据文件中的说明，我们可以使用 squash（或者简写成s） 命令。修改文件内容如下：

```txt
pick 9548167 add index
squash 683cbac update
squash 4a558f9 修改文件名2
pick 8d1455b 添加用户页面2

...
```

然后保存，会出现如下文件：

```txt
# This is a combination of 3 commits.
# This is the 1st commit message:

add index

# This is the commit message #2:

update

# This is the commit message #3:

修改文件名2

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Author:    wz <wyzane1207@163.com>
# Date:      Tue Jun 23 22:41:51 2020 +0800
#
# interactive rebase in progress; onto 28f3903
# Last commands done (3 commands done):
#    squash 683cbac update
#    squash 4a558f9 修改文件名2
# Next command to do (1 remaining command):
#    pick 8d1455b 添加用户页面2
# You are currently rebasing branch 'master' on '28f3903'.
#
# Changes to be committed:
#       new file:   index.html
#       new file:   index02.html
```

我们可以在第一行下面写合并 commit 的注释，写完保存即可。执行 git log --graph，可以看到下面的 commit 记录：

```txt
λ git log --graph
* commit 04c862efc7b757b2038144ab24b2dc913151213c (HEAD -> master)
| Author: wyzane <wyzane1207@163.com>
| Date:   Thu Jun 25 16:00:58 2020 +0800
|
|     添加用户页面2
|
* commit 2343bc35bef0f096b3768403fbceeb587e8ddfcf
| Author: wz <wyzane1207@163.com>
| Date:   Tue Jun 23 22:41:51 2020 +0800
|
|     合并commit测试
|
|     add index
|
|     update
|
|     修改文件名2
|
* commit 28f3903644aea0102c9e7b7a6789b5990a45db50
  Author: wz <wyzane1207@163.com>
  Date:   Mon Jun 22 22:41:14 2020 +0800

      reaeme.md
```

可以看到， 2343bc35bef0f096b3768403fbceeb587e8ddfcf 这个就是新生成的commit。

# 常用命令

## git check

```txt
git checkout 分支名: 切换到分支
git checkout -b 分支名: 创建新分支并切换到新分支
git checkout -b 新分支名 基于分支名或者commitID: 基于某个分支或者commit创建一个分支并切换到该分支
```

## git branch

```txt
git branch: 查看分支列表
git branch -d 分支名: 删除某个分支 
git branch -D 分支名: 强制删除某个分支 
```

## git diff

```txt
git diff 用户比较两个 commit 的区别
git diff commitID1 commitID2: 比较两个commit
git diff HEAD HEAD~1: 比较当前commit与上一次commit的区别
	git diff HEAD HEAD^^ 与 git diff HEAD HEAD~2 一样
```

## git cat-file

```txt
git cat-file -t 9b562d265a03e923b86bfc3b9b8418463f4f1c70: 查看hash值的类型（这个是blob类型，即文件对象）
git cat-file -p 9b562d265a03e923b86bfc3b9b8418463f4f1c70: 查看hash值的内容
	这个 blog 类型的内容如下：
		<h1>git test<h1/>
        <h1>git test<h1/>
git cat-file -t 239ec593c6a2192e76c005435f748b2ad28be832：查看hash值的类型（这个是tree类型，即目录）
git cat-file -p 239ec593c6a2192e76c005435f748b2ad28be832: 查看输出内容，输出如下（内容是一个blob对象）：
		100644 blob e69de29bb2d1d6434b8b29ae775ad8c2e48c5391    readme.md
```



整理中 .....