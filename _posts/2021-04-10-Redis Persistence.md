---
layout: post
title: Redis Persistence
tags: Redis
---
## Redis Persistence
### RDB
Redis  fork子进程，数据集写入临时文件，写入成功替换之前文件，二进制压缩。
#### 优势
1. 一个文件
2. 性能fork子进程
3. 数据集大，相比于启动效率高

#### 劣势
1. 高可用低
2. fork导致服务停止，在fork执行过程中，父进程需要拷贝内存页表给子进程

#### 配置
```
save 900 1    #900s 1个key变化    dump内存快照
save 300 10   #300s 10个key变化   dump内存快照
save 60 1000  #60s  1000个key变化 dump内存快照
```

### AOF
Redis 同步写操作指令。

#### 优势
1. 高可用，每秒同步fork子进程，每修改同步同步，不同步
2. append不会影响存量。写一半失败，下次启动前，redis-check-aof
3. 可读性
4. 自动重写或`bgrewriteaof`，fork一个子进程将内存数据写入一个临时文件，完成后替换aof文件，过程中命令提交到缓存，随后追加到aof文件

#### 劣势
1. 数据集大，恢复慢
2. 效率慢

#### 配置
```
appendfsync always
appendfsync everysec
appendfsync no
```

### 迁移

Redis版本>=2.2，在线切换rdb切换aof，**会阻塞生成全量**。结束后`redis.conf`开启AOF，否则重启按照原设置重启。
```
save                       # 主进程快照
#besave                    # fork子进程快照
config set appendonly yes  # redis阻塞直到最初aof创建完成
config set save ""         # 可选是否关闭rdb
```
AOF开启且文件存在，加载aof文件；否则加载rdb文件

1. RDB：优先级低
    ```
    service redis stop
    # 替换dump.rdb
    service redis start
    ```

2. AOF：优先级高
    ```
    redis-cli -h -p -a --pipe < appendonly.aof
    ```