---
title: Redis哨兵模式的配置与使用
date: 2021-05-19 22:23:08
tags: Redis
---

今天介绍下 Redis 哨兵模式的配置与使用方法。

在 Redis 中，哨兵是一个独立的进程，它用来监控 Redis 集群中中主从服务器的运行状态，当发现 Redis 主服务器宕机后，哨兵会从集群剩下服务器中重新选择一个主服务器，从而保证集群的正常运行，即保证 Redis 集群的高可用性。

<!--more-->

以下例子中使用的 Redis 版本均为 6.0。



# 哨兵模式的配置

以搭建三个哨兵为例，Redis 哨兵模式的配置步骤如下：

```txt
1. 首先，进入 Redis 所在目录，复制三份 sentinel.conf 文件，分别命名为 sentinel01.conf、sentinel02.conf、sentinel03.conf

2. 配置 sentinel01.conf 文件的如下内容：
sentinel monitor mymaster 192.168.172.130 6379 2
# 监控名为 mymaster 的主服务器，主服务器ip和端口为192.168.172.130，6379，当至少有两个sentinel同意时，该主服务器才会被判定为失效

sentinel auth-pass mymaster "wyzane"
# 设置连接主服务器的密码

sentinel down-after-milliseconds mymaster 10000
# 如果服务器在给定的毫秒数之内，没有返回 Sentinel 发送的 PING 命令的回复，或者返回一个错误，那么 Sentinel 将这个服务器标记为主观下线（subjectively down，简称 SDOWN ）。

sentinel parallel-syncs mymaster 1
# 指定了在执行故障转移时，最多可以有多少个从服务器同时对新的主服务器进行同步，这个数字越小，完成故障转移所需的时间就越长。

3. 对 sentinel02.conf 和 sentinel03.conf 进行同样的配置 
```

配置完成后，使用以下方式启动哨兵：

```txt
方式一，分别执行以下命令：
	redis-sentinel sentinel01.conf
	redis-sentinel sentinel02.conf
	redis-sentinel sentinel03.conf
方式二，分别执行以下命令：
	redis-server sentinel01.conf --sentinel
	redis-server sentinel02.conf --sentinel
	redis-server sentinel03.conf --sentinel
```

以上就是 Redis 哨兵的配置和启动，下面介绍下 Redis 哨兵模式的工作原理。



# 哨兵模式的原理

哨兵模式运行的大致流程如下：

当 Redis 集群和 Redis 哨兵启动后，哨兵通过向主服务器和从服务器发送命令，来获取主、从服务器的运行状态，参数 sentinel down-after-milliseconds 用来控制，主、从服务器收到命令后返回响应的时间。如果在给定时间内，哨兵没有收到服务器发回的响应信息，那么该哨兵就将对应的服务器标记为主观下线状态(subjectively down，简称 SDOWN ），当有多个（一般是半数以上）哨兵都把该服务器标记为主观下线状态时，那么该服务器就被标记为客观下线（objectively down， 简称 ODOWN），此时才会发生故障迁移操作。



Redis 的 Sentinel 中关于下线（down）有两个不同的概念：

```txt
1. 主观下线（Subjectively Down， 简称 SDOWN）指的是单个 Sentinel 实例对服务器做出的下线判断。
2. 客观下线（Objectively Down， 简称 ODOWN）指的是多个 Sentinel 实例在对同一个服务器做出 SDOWN 判断， 并且通过 SENTINEL is-master-down-by-addr 命令互相交流之后， 得出的服务器下线判断。 （一个 Sentinel 可以通过向另一个 Sentinel 发送 SENTINEL is-master-down-by-addr 命令来询问对方是否认为给定的服务器已下线。）
```



下面，我们来搭建一个含有一主两从的 Redis 集群，并配置三个哨兵，看一下哨兵是怎么运行的。

Redis 主从的配置就不多做介绍了，下面直接看启动后的日志信息。

Redis 主从信息如下：

```txt
主：192.168.172.131 6380
从：192.168.172.131 6381
从：192.168.172.131 6382
```

启动 192.168.172.131 6380 和 192.168.172.131 6381 后，主服务器的日志为：

```txt
16243:M 22 May 2021 00:56:23.056 * Replica 192.168.172.131:6381 asks for synchronization
16243:M 22 May 2021 00:56:23.056 * Full resync requested by replica 192.168.172.131:6381
16243:M 22 May 2021 00:56:23.056 * Replication backlog created, my new replication IDs are 'b643fa0d90a895b29c82b71446167e6b60d52283' and '0000000000000000000000000000000000000000'
16243:M 22 May 2021 00:56:23.056 * Starting BGSAVE for SYNC with target: disk
16243:M 22 May 2021 00:56:23.056 * Background saving started by pid 16254
16254:C 22 May 2021 00:56:23.058 * DB saved on disk
16254:C 22 May 2021 00:56:23.059 * RDB: 0 MB of memory used by copy-on-write
16243:M 22 May 2021 00:56:23.095 * Background saving terminated with success
16243:M 22 May 2021 00:56:23.097 * Synchronization with replica 192.168.172.131:6381 succeeded
```

192.168.172.131 6381 服务器的日志为：

```txt
16249:S 22 May 2021 00:56:23.054 * Master replied to PING, replication can continue...
16249:S 22 May 2021 00:56:23.055 * Partial resynchronization not possible (no cached master)
16249:S 22 May 2021 00:56:23.057 * Full resync from master: b643fa0d90a895b29c82b71446167e6b60d52283:0
16249:S 22 May 2021 00:56:23.096 * MASTER <-> REPLICA sync: receiving 680 bytes from master to disk
16249:S 22 May 2021 00:56:23.096 * MASTER <-> REPLICA sync: Flushing old data
16249:S 22 May 2021 00:56:23.097 * MASTER <-> REPLICA sync: Loading DB in memory
16249:S 22 May 2021 00:56:23.097 * Loading RDB produced by version 6.0.6
16249:S 22 May 2021 00:56:23.097 * RDB age 0 seconds
16249:S 22 May 2021 00:56:23.097 * RDB memory usage when created 1.83 Mb
16249:S 22 May 2021 00:56:23.097 * MASTER <-> REPLICA sync: Finished with success
```

上面的过程表示，从服务器连接上主服务器后，进行了数据的全量同步。同样的，启动 192.168.172.131 6382 后，数据也会进行全量同步。



下面再启动 Redis 哨兵。

执行 redis-sentinel sentinel.conf 后，哨兵服务器的日志为：

```txt
16392:X 22 May 2021 01:13:36.020 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
16392:X 22 May 2021 01:13:36.020 # Sentinel ID is 8fd93d8d9547434177903ba11cb5ae0bf603f358
16392:X 22 May 2021 01:13:36.020 # +monitor master mymaster 192.168.172.131 6380 quorum 2
16392:X 22 May 2021 01:13:36.022 * +slave slave 192.168.172.131:6381 192.168.172.131 6381 @ mymaster 192.168.172.131 6380
16392:X 22 May 2021 01:13:36.024 * +slave slave 192.168.172.131:6382 192.168.172.131 6382 @ mymaster 192.168.172.131 6380
```

若将主服务器的进程关掉，则哨兵的日志为：

```txt
16481:X 22 May 2021 01:24:02.032 # +sdown master mymaster 192.168.172.131 6380（当前哨兵认为主服务器宕机，主观下线）
16481:X 22 May 2021 01:24:02.093 # +odown master mymaster 192.168.172.131 6380 （有三个哨兵认为主服务器宕机，超过了两个，客观下线）#quorum 3/2
16481:X 22 May 2021 01:24:02.093 # +new-epoch 9（递增的集群版本号）
16481:X 22 May 2021 01:24:02.093 # +try-failover master mymaster 192.168.172.131 6380（开始尝试对集群进行故障迁移）
16481:X 22 May 2021 01:24:02.097 # +vote-for-leader 12e1af953d931b8ac097876d7bdf803ba39c52f6 9（选出一个哨兵，作为故障迁移的leader，当前哨兵，投票给 Sentinel ID 为 12e1af953d931b8ac097876d7bdf803ba39c52f6 的哨兵）
16481:X 22 May 2021 01:24:02.103 # 2c2c810af7936fba2facbeb159b0f7fa1e87d325 voted for 2c2c810af7936fba2facbeb159b0f7fa1e87d325 9（其它哨兵投票）
16481:X 22 May 2021 01:24:02.106 # 8fd93d8d9547434177903ba11cb5ae0bf603f358 voted for 12e1af953d931b8ac097876d7bdf803ba39c52f6 9（其它哨兵投票）
16481:X 22 May 2021 01:24:02.187 # +elected-leader master mymaster 192.168.172.131 6380（当前的主节点）
16481:X 22 May 2021 01:24:02.188 # +failover-state-select-slave master mymaster 192.168.172.131 6380（分析从节点的状态）
16481:X 22 May 2021 01:24:02.265 # +selected-slave slave 192.168.172.131:6382 192.168.172.131 6382 @ mymaster 192.168.172.131 6380（选出一个从节点）
16481:X 22 May 2021 01:24:02.265 * +failover-state-send-slaveof-noone slave 192.168.172.131:6382 192.168.172.131 6382 @ mymaster 192.168.172.131 6380（将上面选出的从节点与现在的主节点进行主从切换）
16481:X 22 May 2021 01:24:02.355 * +failover-state-wait-promotion slave 192.168.172.131:6382 192.168.172.131 6382 @ mymaster 192.168.172.131 6380（等待从节点升级成主节点）
16481:X 22 May 2021 01:24:02.531 # +promoted-slave slave 192.168.172.131:6382 192.168.172.131 6382 @ mymaster 192.168.172.131 6380（从节点已经升级为主节点）
16481:X 22 May 2021 01:24:02.532 # +failover-state-reconf-slaves master mymaster 192.168.172.131 6380（修改原来主节点的配置文件）
16481:X 22 May 2021 01:24:02.586 * +slave-reconf-sent slave 192.168.172.131:6381 192.168.172.131 6381 @ mymaster 192.168.172.131 6380（重写从节点的配置文件，即为从节点配置新的主节点）
16481:X 22 May 2021 01:24:03.229 # -odown master mymaster 192.168.172.131 6380
16481:X 22 May 2021 01:24:03.565 * +slave-reconf-inprog slave 192.168.172.131:6381 192.168.172.131 6381 @ mymaster 192.168.172.131 6380（从节点正在重新指向为新的主节点）
16481:X 22 May 2021 01:24:03.565 * +slave-reconf-done slave 192.168.172.131:6381 192.168.172.131 6381 @ mymaster 192.168.172.131 6380（从节点配置完毕）
16481:X 22 May 2021 01:24:03.648 # +failover-end master mymaster 192.168.172.131 6380（故障迁移结束）
16481:X 22 May 2021 01:24:03.648 # +switch-master mymaster 192.168.172.131 6380 192.168.172.131 6382（故障迁移成功，哨兵开始监控新的主服务器）
16481:X 22 May 2021 01:24:03.648 * +slave slave 192.168.172.131:6381 192.168.172.131 6381 @ mymaster 192.168.172.131 6382（确认新主节点的从节点）
16481:X 22 May 2021 01:24:03.648 * +slave slave 192.168.172.131:6380 192.168.172.131 6380 @ mymaster 192.168.172.131 6382（确认新主节点的从节点）
16481:X 22 May 2021 01:24:08.662 # +sdown slave 192.168.172.131:6380 192.168.172.131 6380 @ mymaster 192.168.172.131 6382（原主节点变为slave节点并且主观下线）
```

上面的日志是哨兵发现主服务器宕机后，故障恢复的的一个过程。