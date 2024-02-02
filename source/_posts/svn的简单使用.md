---
title: svn的简单使用
date: 2020-06-27 15:29:17
tags: SVN
categories: Python
---

与 git 一样，svn （Subversion）也是一个开源的版本控制系统，相比于 git 的分布式管理，svn是一个集中式的版本管理工具。今天介绍下它的简单使用。

<!--more-->

svn 是集中式的版本管理工具，它依赖于 svn 服务器，所有的版本信息都保存在服务器上。

# svn使用总结

## 创建版本库

svn的使用步骤如下：

```txt
ubuntu安装svn
1. apt-get update
2. apt-get install subversion
安装完成后，可以执行 svnserve --version 查看版本信息

创建版本库：
1. 在 /home/wyzane 下创建 svn 目录（作为中央仓库）
2. 创建仓库：svnadmin create /home/wyzane/svn
3. 创建完成后，在 svn 下会生成 conf、db 等目录
目录结构如下：
        .
        ├── conf
        │   ├── authz
        │   ├── hooks-env.tmpl
        │   ├── passwd
        │   └── svnserve.conf
        ├── db
        │   ├── current
        │   ├── format
        │   ├── fsfs.conf
        │   ├── fs-type
        │   ├── min-unpacked-rev
        │   ├── rep-cache.db
        │   ├── rep-cache.db-journal
        │   ├── revprops
        │   ├── revs
        │   ├── transactions
        │   ├── txn-current
        │   ├── txn-current-lock
        │   ├── txn-protorevs
        │   ├── uuid
        │   └── write-lock
        ├── format
        ├── hooks
        │   ├── post-commit.tmpl
        │   ├── post-lock.tmpl
        │   ├── post-revprop-change.tmpl
        │   ├── post-unlock.tmpl
        │   ├── pre-commit.tmpl
        │   ├── pre-lock.tmpl
        │   ├── pre-revprop-change.tmpl
        │   ├── pre-unlock.tmpl
        │   └── start-commit.tmpl
        ├── locks
        │   ├── db.lock
        │   └── db-logs.lock
        └── README.txt
其中，conf目录下存放了配置文件，passwd 是用户配置文件，可以配置用户名和密码。authz 是权限配置文件，可以配置用户对应的读写权限，也可以对用户进行分组配置。svnserve.conf 是基本的配置文件。

启动服务：
svnserve -d -r /home/wyzane/svn  (默认端口 3690)
其中：
	-d：表示在后台运行
	-r：指定服务器的根目录
查看进程：ps aux | grep svnserve

客户端 checkout 代码需要执行以下命令：
svn checkout svn://192.168.0.105/ --username wyzane
```

在 windows 下使用时，我们可以下载 svn 的 windows 版本进行安装，下载地址为：

```txt
http://subversion.apache.org/packages.html
```

## 分支操作

```txt
1. 首先创建 branches 目录并提交
2. 创建分支：svn copy trunk/ branches/sub，并使用 svn commit 提交（基于 trunk 的分支）
3. 修改 sub 分支上的内容并提交
4. 合并分支：切换到 trunk 目录，执行svn merge ../branches/sub/
5. 提交合并后的内容：svn commit -m "update"
```

## tag操作

通过 tag 标签，可以给某一具体的版本加上一个有意义的名称，便于更好的管理，tag 的使用方式如下：

```txt
1. 创建 tags 目录并提交：mkdir tags & svn add tags & svn commit -m "commit tags dir"
2. 在当前的 trunk 版本上打一个 tag：svn copy trunk/ tags/v1.0
3. 提交到版本库：svn commit -m "v1.0"
```

# 常用命令

```txt
svn add test.txt   # 添加文件到版本库中
A         test.txt
svn status         # 查看状态
A       test.txt  
svn commit -m "update"   # 将改动的文件提交到版本库，此时再使用 svn status 时，就不会有输出信息了

svn update  # 更新版本哭的改动到本地

svn log   # 查看版本库的提交历史
```



# 使用时可能的错误

```txt
1. commit failed， 没有找到事务5-9
解决：这是因为 svnserve 的服务没有开启，启动 svnserve 即可。

2. the error was 条目不可读
解决：配置文件中修改 anon-access 的值为 none，然后重启即可
```

