---
title: Linux服务器之间免密登录的配置
date: 2021-09-04 16:34:49
tags: linux
categories: Linux
---



下面的步骤中以 ubuntu1804 为例，介绍服务器之间配置免密登录的步骤。

<!--more-->

三台服务器的 ip 为：192.168.0.101、192.168.0.103、192.168.0.104，配置使101可以在不需要密码的情况下访问103和104。

## 配置步骤

```txt
1. 三台服务器都安装 ssh 服务端和客户端

2. 修改 103 和 104 服务器的 ssh 配置 /etc/ssh/sshd_config，增加如下配置：
	PermitRootLogin yes  # 允许 root 用户通过 ssh 登录
   修改完成保存后，重启 ssh 服务：service ssh restart
   
3. 在 101 上切换到 root 用户，然后执行：ssh-keygen -t rsa，执行后一路回车，生成公钥和私钥对

4. 在 101 上执行：ssh-copy-id 192.168.0.103
   输入 103 的 root 密码后（如果忘记 root 密码，可以使用 sudo passwd root 重新设置 root 密码），会出现如下信息：
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@192.168.0.103's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh '192.168.0.103'"
and check to make sure that only the key(s) you wanted were added.
  
   上面的信息表示，已经成功将 101 的公钥添加到了 103 服务器的 /root/.ssh/authored_keys 文件中。
   
5. 执行: ssh 192.168.0.103 即可登录 103 服务器，也可以执行 scp file 192.168.0.103:/home/wyzane 将需要的文件拷贝到 103 服务器上

6. 配置 104 服务器时， 重复执行步骤 4 即可。
```

上面就是配置服务器之间免密登录的步骤，当我们在 101 服务器的 /etc/hosts 文件中添加如下配置后：

```txt
192.168.0.103  app-01
192.168.0.104  app-02
```

可以使用 ssh app-01 和 ssh app-02 之类的命令通过 DNS 解析来访问 103 和 104 服务器。