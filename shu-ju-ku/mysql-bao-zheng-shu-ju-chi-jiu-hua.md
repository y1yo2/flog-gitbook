---
description: MySQL使用WAL管理数据变化，在此机制上，如何保证数据持久性？
---

# MySQL保证数据持久性

### WAL\(write ahead log\)

WAL的关键，是binlog 和 innodb log（redolog）的两阶段提交。

1. 开启事务，执行语句，更新 data page，写binlog、innodb log；
2. 完成事务后，innodb通知server，server将binlog写入binlog file；
3. server调用innodb提交事务接口，innodb log改为commit状态；

只要 redo log 和 binlog 保证持久化到磁盘，就能确保 MySQL 异常重启后，数据可以恢复。即保证数据的持久性和一致性。

### binlog持久化

事务执行时，server将 binlog 写入 binlog cache，且每个线程分配一个内存空间作为 binlog cache。

事务提交时，server一次性将完整事务的 binlog cache write（写入）binlog，并清空 binlog cache。

保证binlog事务的完整性，所以 binlog 的 cache按线程隔离，一次性写入binlog file



### innodb log持久化







