# [原文链接](https://redis.io/topics/memory-optimization)

# 内存优化
该页面会持续更新. 目前这只是你需要确认的一个清单， 如果你有内存方面的问题.

### 特殊编码的小型聚合数据类型
从 Redis 2.2 开始许多数据类型已优化过， 它们使用更少的空间来满足固定大小的情况. Hashes, Lists, Sets 仅由整型值组成时, 以及 Sorted Sets, 当元素个数小于给定的值时, and up to a maximum element size, are encoded in a very memory efficient way that uses up to 10 times less memory (with 5 time less memory used being the average saving).
