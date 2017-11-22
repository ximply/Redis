# [原文链接](https://redis.io/topics/lru-cache#using-redis-as-an-lru-cache)

# Redis 用作 LRU 缓存

当 Redis 用作缓存时, 当你添加新数据时它通常会将老的数据进行自动回收处理. 这个行为在社区开发者中是众所周知的, 因为这是当下流行的内存缓存系统默认的行为.

LRU 实际上只是支持的回收算法之一. 本页涉及更多关于 Redis maxmemory 指令的普通话题, 这个指令限制了最大使用内存, 然后也会涉及 Redis 所使用的 LRU algorithm 的内部细节, 实际上是近似于真正的 LRU.

从 Redis 4.0 版本开始, 引入了 LFU (最近频繁使用) 这个新的回收策略. 文档中有单独的章节会涉及到.

### Maxmemory 配置指令
maxmemory 配置指令用于配置 Redis 数据集的限定内存使用大小. 可以在 redis.conf 配置文件中配置, 或者过后使用 CONFIG SET 命令在运行时进行配置也可以.

例如限制内存为 100 MB, 在 redis.conf 配置文件中使用以下指令.
