---
title: Linux下搭建开发环境
date: 2019-12-29 21:47:00
tags: linux
categories: Linux
---

开发中，我们经常需要用到虚拟机，在虚拟机中创建 Linux 操作系统，配置开发环境。对于不经常配置 Linux 虚拟机的同学来说，配置一遍可能需要花费很长时间。

<!--more-->

下面会以 Ubuntu18.04 和 Python 开发为主，介绍下虚拟机创建完成后，需要安装的额外软件及安装步骤。这些软件有的是必须的，有的能提高开发效率。主要介绍下面一些软件的安装步骤：

```txt
openssh-server
vmware tools
sogou pinyin
zsh
pip
virtualenv
```



#### openssh-server

安装openssh-server是为了允许远程登陆到ubuntu18.04虚拟机上，步骤如下：

```txt
1. 安装openssh-server软件包：sudo apt-get install openssh-server
2. 开启防火墙：sudo ufw enable
3. 开放22端口：ufw allow 22/tcp
```



#### vmware tools

vmware tools是 vmware 虚拟机中自带的一种增强工具，可以提供一些额外的功能，如可以使虚拟机窗口全屏显式等。如果直接登陆虚拟机操作，全屏显示这个功能还是很有必要的，如果使用远程连接工具（xshell，secureCRT等）连接虚拟机，全屏显示功能也没有影响。

vmware tools的安装步骤如下：

```txt
1. 点击 VMWare Workstation 软件中的 虚拟机 --> 安装VMWare Tools选项，会在虚拟机的Linux操作系统
	上生成VMWare Tools的tar包
2. 解压VMWare Tools的tar包： tar -zxvf tar包，进入生成的 vmware-tools-distrib 目录中；
3. 执行 vmware-install.pl 文件，安装过程中，需要选择一些选项；执行完成后，vmware tools 即安装完
	成。
```



#### sogoupinyin

https://blog.csdn.net/qq_40563761/article/details/82664851

sogoupinyin的安装步骤如下：

1. 去官网下载 Linux 版本相应的deb安装包：sogoupinyin_2.3.1.0112_amd64.deb

   下载地址：https://pinyin.sogou.com/linux/?r=pinyin；

2. 执行 sudo apt-get install -f，安装fcitx相关软件包；

3. 安装第1步中下载的 deb 包：sudo dpkg -i sogoupinyin_2.3.1.0112_amd64.deb；

4. 找到系统设置中的 Language Support 选项，如下图：

   ![](Language Support.png)

5. 点击进入后，如下图：

​        ![](Language Support2.png)

​        刚进入时，可能需要安装其他软件包，此时点击安装即可，安装完成后将箭头指定的配置选择为 fcitx；

6. 然后重启系统；

7. 重启成功后，找到系统中的 Fcitx Configuration 配置项，进入后如下图：

   ![](fcitx.png)

​        点击 + 号，选择 Sogou Pinyin即可。这时就可以使用搜狗输入法了。



#### zsh

zsh的介绍就不多说了，安装 zsh 的步骤如下：

```txt
1. sudo apt-get install zsh
2. 修改默认shell为zsh：chsh -s /bin/zsh
3. 配置密码文件, 用来解决改变shell时出现的PAM认证问题
   sudo vim /etc/passwd
   然后将第一行中的/bin/bash改成/bin/zsh即可   
4. 安装oh-my-zsh
   安装git：
   sudo apt install git
   安装oh-my-zsh：
   sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-
   zsh/master/tools/install.sh)"  
5. 修改 zsh 的主题
   sudo vim .zshrc
   修改ZSH_THEME="robbyrussell"为ZSH_THEME="cloud"
```



#### pip

安装python3版本的pip：

```txt
1. 安装pip：sudo apt install python3-pip
2. 安装相关开发工具：sudo apt install build-essential python3-dev  python3-setuptools
3. 安装完成后，输入 pip3 即可安装 python 包；但是我们还可以创建软连接，使得输入 pip 时就可以安装 
	python包了。创建软连接步骤如下：
   1）进入 /usr/bin 目录
   2）执行 ln -s pip3 pip，表示在当前目录创建一个软连接 pip，指向当前目录的 pip3
```



#### virtualenv

安装 virtualenv 和 virtualenvwrapper 的步骤如下：

```txt
virtualenv的安装与使用：
1. pip install virtualenv
   安装完成后，可以执行以下命令来创建和使用虚拟环境：
   1）创建虚拟环境 virtualenv --python=python3  env_test
   2）进入虚拟环境 source env_test/bin/activate
   3）使用pip安装软件包 pip install requests
   4）退出虚拟环境 deactivate env_test

virtualenvwrapper可以更方便的管理虚拟环境，其安装步骤如下：
1. pip install virtualenvwrapper
2. 设置环境变量，修改 .zshrc 文件，添加如下信息：
    export WORKON_HOME=/home/wyzane/virtualenvs
    export VIRTUALENVWRAPPER_VIRTUALENV_ARGS='--no-site-packages'
    export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3
	plugins=(virtualenvwrapper)  # 指定 virtualenvwrapper 插件，由于是在zsh的配置文
		件.zshrc中添加的配置，所以需要这一配置。不同的shell配置文件可能不同。

3. 使用
	1）创建虚拟环境 mkvirtualenv env_python
	2）进入虚拟环境 workon env_python
	3）退出虚拟环境 deactivate env_python
```



对于 python 开发的同学而言，以上就是 Linux 环境需要安装的基本软件。