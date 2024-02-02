---
title: Redis主从复制的搭建
date: 2020-07-04 10:01:34
tags: Redis
categories: Python
---

Redis 的主从复制是一个很有用的功能，在大型系统中，为了分担 Redis 的读写压力，可以使用主从复制功能。下面介绍下 Redis 主从复制的搭建。

<!--more-->

# 主从复制原理

## Redis 持久化方式

先介绍下 Redis 的持久化方式。Redis 的持久化方式有两种 rdb和 aof，默认是 rdb 的方式。

### rdb 备份方式

rdb 是半持久化的存储方式，它会定期将内存中数据保存到磁盘上，从而保证数据的持久化，永久保存Redis数据。它是通过快照（snapshotting）的方式完成的，当满足在 Redis.conf 配置文件中设置的条件时 Redis 会自动将内存中的所有数据进行快照并存储在硬盘上，完成数据备份。

Redis进行 rdb 快照的条件由用户在配置文件中自定义，由两个参数构成：时间和改动键的个数。当在指定时间内被更改键的个数大于指定的数值时就会进行快照。配置文件中默认预值了3个条件：

```txt
save 900 1       #900秒内有至少1个键被更改则进行快照；
save 300 10      #300秒内有至少10个键被更改则进行快照；
save 60  10000   #60秒内有至少10000个键被更改则进行快照
```

默认可以存在多个条件，条件之间是“或”的关系，只要满足其中任意一个，就会进行快照备份数据。若想要禁用自动快照，只需删除所有save参数即可。

Redis 默认会将快照文件存储在 Redis 数据目录中，默认文件名是：dump.rdb，可通过配置dir和dbfilename两个参数来指定快照文件的存储路径和文件名。

Redis快照过程：Redis使用fork函数复制一份当前进程（父进程）的副本（子进程），父进程负责接收和处理客户端发来的命令，而子进程负责将内存中的数据写入硬盘中的临时文件，当子进程写完所有数据后会用该临时文件替换旧的 rdb 文件，至此一次快照操作完成。
rdb 文件是经过压缩的二进制格式，所以占用的空间会小于内存中的数据大小。除了自动快照，还可以手动发送SAVE和BGSAVE 命令让Redis执行快照。两个命令区别是：SAVE是由主进程进行快照操作，会阻塞其他请求，BGSAVE会通过fork 子进程进行快照操作。

手动执行备份：redis-cli bgsave

```txt
save  // 手动备份
bgsave  // 手动备份
config get dir  // 获取redis的dir
```

### aof 备份方式

aof 是全持久化的备份方式，使用这种方式，Redis 会实时将内存中数据刷到磁盘上，从而保证数据的持久化，永久保存Redi s数据。如果数据非常重要无法承受任何损失，可以使用 aof 方式进行持久化，默认 Redis 没有开启 aof（append only file）方式的全持久化模式。

在启动时 Redis 会逐个执行 apf 文件中的命令来将硬盘中的数据载入到内存中，载入的速度相对于 rdb 方式慢些，开启 aof 持久化后每执行一条都会更改Redis中的数据的命令，Redis就会将命令写入硬盘中的 aof 文件。

Redis允许同时开启 aof 和 rdb ，既保证了数据安全又使得进行备份等操作十分容易。

Redis aof持久化参数配置详解：

```txt
appendonly  yes                   #开启AOF持久化功能；
appendfilenameappendonly.aof      #AOF持久化保存文件名；
appendfsyncalways                 #每次执行写入都会执行同步，最安全也最慢；
#appendfsynceverysec              #每秒执行一次同步操作；
#appendfsyncno                    #不主动进行同步操作，而是完全交由操作系统来做，每30秒一次，最快也最不安全；
auto-aof-rewrite-percentage  100  #当AOF文件大小超过上一次重写时的AOF文件大小的百分之多少时会再次进行重写，如果之前没有重写过，则以启动时的AOF文件大小为依据；
auto-aof-rewrite-min-size    64mb #允许重写的最小AOF文件大小配置写入AOF文件后，要求系统刷新硬盘缓存的机制。
```

在配置文件中进行以下配置：

```txt
appendonly yes
appendfsync everysec
```

aof 刷新日志到disk的规则：

```txt
appendfsync always   #always 表示每次有写操作都进行同步，非常慢，非常安全。
appendfsync everysec #everysec表示对写操作进行累积，每秒同步一次
```

官方的建议的everysec，安全，就是速度不够快，如果是机器出现问题可能会丢失1秒的数据。

手动执行备份：

```txt
redis-cli bgrewriteaof
```



## 主从数据同步

当 slave 初始化的时候，会将 master 上的数据复制一份到自己服务器上。此后，每当 master 新增数据后，都会将写命令发送一份到 slave 上，slave 接收到写命令后将数据写入。我们可以通过 monitor 命令来看 slave 的写入过程。

例如：当 master 执行 set name wyzane 后，使用 monitor 查看到 slave 的执行如下：

```txt
1593857563.031330 [0 192.168.0.105:6379] "PING"
1593857553.918113 [0 192.168.0.105:6379] "set" "name" "wyzane"
1593857563.031330 [0 192.168.0.105:6379] "PING"
```



# 主从复制搭建

下面在一个服务器上开启三个 Redis 进程为例，来介绍下主从复制的搭建过程。

## 不需要权限校验的主从复制

配置的时候，需要修改 Redis .conf 配置文件，master 和 slave 的配置分别如下。其中，protected-mode no 表示 slave 连接 master 时不需要密码校验。

master配置：

```txt
bind 192.168.0.105
protected-mode no
port 6379
```

slqve1 的配置如下：

```txt
bind 192.168.0.105
slaveof 192.168.0.105 6379
protected-mode no
port 6380
```

slave2 的配置如下：

```txt
bind 192.168.0.105
slaveof 192.168.0.105 6379
protected-mode no
port 6381
```

配置完成后，执行以下命令启动这三个服务：

```txt
redis-server redis.conf （master）
redis-server redis2.conf （slave1）
redis-server redis3.conf （slave2）
```

服务启动后，需要连接这三个服务：

```txt
redis-cli -h 192.168.0.105 -p 6379
redis-cli -h 192.168.0.105 -p 6380
redis-cli -h 192.168.0.105 -p 6381
```

连接上客户端，在 master 上 set 数据后，可以分别在 slave1、slave2 上看到这些数据。



## 权限校验的主从配置

在生产环境上，为了保证安全性，权限校验是非常重要的。所以，我们配置主从复制时，一定要把权限校验的功能加上去。权限校验首先需要把 master redis.conf 中的 protected-mode 值设置为 yes（默认），然后再设置密码。设置密码的方式如下：

```txt
protected-mode yes
requirepass wyzane  # wyzane 就是密码
```

设置完成后，使用 redis-cli 时，需要执行 auth wyzane， 即输入密码。

slave1 设置：

```txt
masterauth wyzane
```

slave2 设置：

```txt
masterauth wyzane
```

设置完成后，重启 Redis 服务就可以了。