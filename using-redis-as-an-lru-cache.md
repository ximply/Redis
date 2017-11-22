# [原文链接](https://redis.io/topics/lru-cache#using-redis-as-an-lru-cache)

# Redis 用作 LRU 缓存

当 Redis 用作缓存时, 当你添加新数据时它通常会将旧数据进行自动回收处理. 这个行为在社区开发者中是众所周知的, 因为这是当下流行的内存缓存系统默认的行为.

LRU 实际上只是支持的回收算法之一. 本页涉及更多关于 Redis maxmemory 指令的普通话题, 这个指令限制了最大使用内存, 然后也会涉及 Redis 所使用的 LRU algorithm 的内部细节, 实际上是近似于真正的 LRU.

从 Redis 4.0 版本开始, 引入了 LFU (最近频繁使用) 这个新的回收策略. 文档中有单独的章节会涉及到.

### Maxmemory 配置指令
maxmemory 配置指令用于配置 Redis 数据集的限定内存使用大小. 可以在 redis.conf 配置文件中配置, 或者过后使用 CONFIG SET 命令在运行时进行配置也可以.

例如限制内存为 100 MB, 在 redis.conf 配置文件中使用以下指令.
```Java
maxmemory 100mb
```

设置 maxmemory 为 0 的话表示不限制内存. 64 位操作系统上默认设置为 0, 而 32 位操作系统上可以显示地指定为 3GB.

当达到限定的内存大小时, 可以在不同行为之间进行选择, 也就是所谓的策略. 内存不够用时 Redis 只是在执行命令时返回错误, 或者每次新增数据时回收一些旧数据.

### 回收策略
当内存达到限制值时 Redis 的实际行为会遵循 maxmemory-policy 配置命令的配置项.
有以下这些策略可以使用:
* noeviction: 客户端尝试执行会导致需要使用更多内存的命令时(大部分的写命令, 除了 DEL 命令以及其他异常), 这个时候如果内存达到限制值则会返回错误.
* allkeys-lru: 首先通过尝试移除最近最少使用的 (LRU) keys 进行 keys 回收, 为新增数据腾出空间.
* volatile-lru: 首先通过尝试移除最近最少使用的 (LRU) keys 进行 keys 回收, 但仅在那些有过期的 set 的 keys 之间进行, 为新增数据腾出空间.
* allkeys-random: 对 keys 进行随机回收然后为新增数据腾出空间.
* volatile-random: 对 keys 进行随机回收然后为新增数据腾出空间, 但仅在那些有过期的 set 的 keys 之间进行.
* volatile-ttl: 在那些有过期的 set 的 keys 之间进行回收, 然后首先尝试回收那些存活时间(TTL)较短的 keys, 为新增数据腾出空间.

Volatile-lru, volatile-random 以及 volatile-ttl 这个三个策略在没有 keys 可以回收的前提下所表现的行为是跟 noeviction 这个策略类似的.

根据应用的访问方式来选择正确的回收策略是很重要的, 不过你可以在应用运行的时候对策略进行重新配置, 然后使用 Redis INFO 输出来监控缓存击穿和命中的次数进行配置调优.

一般可以这么做:

* 使用 allkeys-lru 策略, 如果你期望大部分的请求按幂次定律分布的话, 也就是说, 你期望元素的子集比个别的更经常被访问. 如果你不是很有把握的话这会是一个好的选择.
* 使用 allkeys-random 策略, 如果你有周期性循环地持续扫描所有的 keys 的情况, 或者你期望均匀分布 (所有元素被访问的概率可能是一样的).
* 使用 volatile-ttl 策略, 如果你想要在创建缓存对象时, 通过使用不同的 TTL 值, 然后可以提供合适的关于过期的候选方案的建议给 Redis 的话.


