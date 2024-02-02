---
title: Linux服务器之间免密登录的配置
date: 2021-09-04 16:34:49
tags: linux
categories: Linux
---



下面的步骤中以 ubuntu1804 为例，介绍服务器之间配置免密登录的步骤。

<!--more-->

三台服务器的 ip 为：192.168.0.101、192.168.0.103、192.168.0.104，配置使101可以在不需要密码（通过配置密钥）的情况下访问103和104。

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



## ansible的使用

如果有多台服务器需要配置密钥登录，可以使用Ansible进行批量配置。

Ansible 是一个开源的自动化平台，它用于配置管理、应用部署、任务自动执行等，下面介绍如何使用它进行批量密钥登录的配置。

假设我们有若干个员工，需要密钥登录多台服务器，那么就需要把多个员工的公钥上传到多台服务器上。

1. 首先创建一个hosts文件，文件中保存了服务器的信息：

```txt
[test]
test_ip ansible_user=test_user ansible_ssh_pass=test_pass ansible_port=22

[preprod]
preprod_ip ansible_user=preprod_user ansible_ssh_pass=preprod_pass ansible_port=2000

[prod]
prod_ip ansible_user=prod_user ansible_ssh_pass=prod_pass ansible_port=22

[nginx]
nginx_ip ansible_user=nginx_user ansible_ssh_pass=nginx_pass ansible_port=22
```

文件中，中括号的内容可以理解为服务器名称，这个名称在下面的文件中会用到，名称下面分别指定了对应的服务器ip、用户名、密码、端口信息。

2. 创建一个 Ansible playbook 配置文件 test.yml，用于保存需要执行的任务

内容可以像下面这样：

```yml
- name: Configure SSH Keys   # 操作的名称
  hosts: test:preprod        # 指定hosts文件中的哪些服务器执行这个操作
  become: yes                # become表示是否使用特权提升，yes表示任务将以超级用户权限运行
  vars:                      # playbook运行时使用的变量
    ansible_become_pass: "Dy18Zz@!@"   # 在提升权限时使用的密码
  tasks:                               # 这个操作中有哪些任务需要执行
    - name: Set authorized key         # 任务名称，设置ssh授权密钥
      authorized_key:
        user: lxtech
        state: present
        key: "{{ lookup('file', item) }}"    # 使用了lookup从文件中读取密钥
      loop:                                  # 通过循环添加多个密钥文件，配置这些员工密钥登录test和preprod服务器
        - ../pub/001.pub
        - ../pub/002.pub
        - ../pub/003.pub
        ...
    - name: Ensure PubkeyAuthentication is enabled   # 任务名称，更新sshd_config文件，设置PubkeyAuthentication yes，允许密钥登录
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#?PubkeyAuthentication'
        line: 'PubkeyAuthentication yes'
        state: present
    - name: Disable Password Authentication   # 禁止账号密码登录
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#?PasswordAuthentication'
        line: 'PasswordAuthentication no'
        state: present
    - name: Disable Root SSH Login   # 禁止使用root用户登录
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#?PermitRootLogin'
        line: 'PermitRootLogin no'
        state: present
    - name: Restart SSH service    # 重启ssh服务
      service:
        name: sshd
        state: restarted

- name: Configure Nginx SSH Keys
  hosts: nginx:prod
  become: yes
  vars:
    ansible_become_pass: "!zE4s678g@"
  tasks:
    - name: Set authorized key
      authorized_key:
        user: root
        state: present
        key: "{{ lookup('file', item) }}"
      loop:
        - /home/dytech/pkgs/ansible/pub/005.pub
        - /home/dytech/pkgs/ansible/pub/006.pub
        - /home/dytech/pkgs/ansible/pub/007.pub
    - name: Ensure PubkeyAuthentication is enabled
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#?PubkeyAuthentication'
        line: 'PubkeyAuthentication yes'
        state: present
    - name: Disable Password Authentication
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^#?PasswordAuthentication'
        line: 'PasswordAuthentication no'
        state: present
    - name: Restart SSH service
      service:
        name: sshd
        state: restarted
```

配置完成后，执行命令 ansible-playbook -i hosts test.yml 批量执行任务就行了。



