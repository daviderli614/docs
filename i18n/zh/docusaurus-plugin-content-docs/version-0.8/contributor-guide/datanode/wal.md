# 预写日志

## 介绍

我们的存储引擎受到了日志结构合并树（Log-structured Merge Tree，LSMT）的启发。对数据的变更操作直接应用于 MemTable 而不是持久化到磁盘上的数据页，这显著提高了性能，但也带来了持久化相关的问题，特别是在 Datanode 意外崩溃时。与所有类似 LSMT 的存储引擎一样，GreptimeDB 使用预写日志（Write-Ahead Log，WAL）来确保数据被可靠地持久化，并且保证崩溃时的数据完整性。

预写日志是一个仅提供追加写的文件组。所有的 INSERT、UPDATE 和 DELETE 操作都被转换为操作日志，然后追加到 WAL。一旦操作日志被持久化到底层文件，该操作才可以进一步应用到 MemTable。

当数据节点重新启动时，WAL 中的操作条目将被重放，以重建正确的 MemTable 状态。

![WAL in Datanode](/wal.png)

## 命名空间

WAL 的命名空间用于区分来自不同 region 的条目。追加和读取操作必须提供一个命名空间。目前，region ID 被用作命名空间，因为每个 region 都有一个在数据节点重新启动时需要重构的 MemTable。

## 同步/异步刷盘

默认情况下，WAL 的追加写是异步的，这意味着写入方不会等待操作日志被刷入到磁盘并持久化。这个默认设置提供了更高的性能，但在服务器意外关闭时可能会丢失数据。另一方面，同步刷新提供了更高的可靠性，但其代价是性能更低。

在 v0.4 版本中，新的 region worker 架构可以使用批处理来减轻同步刷盘的开销。