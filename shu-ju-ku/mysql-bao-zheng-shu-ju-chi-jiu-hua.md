---
description: MySQL使用WAL管理数据变化，在此机制上，如何保证数据持久性？
---

# MySQL保证数据持久性

## WAL\(write ahead log\)

WAL的关键，是binlog 和 innodb log（redolog）的两阶段提交。

1. 开启事务，执行语句，更新 data page，写binlog、innodb log；
2. 完成事务后，innodb通知server，server将binlog写入binlog file；
3. server调用innodb提交事务接口，innodb log改为commit状态；

只要 redo log 和 binlog 保证持久化到磁盘，就能确保 MySQL 异常重启后，数据可以恢复。即保证数据的持久性和一致性。

## binlog持久化

* 事务执行时，server将 binlog 写入 binlog cache，且每个线程分配一个内存空间作为 binlog cache。
* 事务提交时，server一次性将完整事务的 binlog cache write（写入）binlog，并清空 binlog cache。

保证binlog事务的完整性，所以 binlog 的 cache按线程隔离，一次性写入binlog file。

### binlog是什么，怎么管理、使用？

Mysql server层维护的日志，binlog，二进制日志格式（无法直接查看）。记录每个事务的信息（以事务为单位）。记录格式有三种：

* statement：sql语句（存在主从不一致问题）
* rows：每一行的更改内容（占用空间大）
* mixed：sql语句+rows（存在主从不一致风险时）

通过mysql参数、命令管理binlog

* `log_bin`，`sql_log_bin`，开关binlog。
  * `sql_log_bin`，无需重启数据库，只需新建连接即生效。用于还原数据库时，临时关闭binlog防止重复记录日志。
* `show binary logs`，查看binlog列表。
* `show master status`，查看当前使用的binlog文件和Position
* `/usr/bin/mysqlbinlog`，脚本，或show binlog events in binlog.\*\*查看binlog内容

具体使用细节如下：

```sql
1.查看binlog文件列表
show binary logs; 
+------------------+-----------+
| Log_name         | File_size |
+------------------+-----------+
| binlog.000000001 |      2048 |
| binlog.000000002 |      1024 |
+------------------+-----------+

2.查看当前使用的binlog文件和Position
show master status;
 +------------------+----------+
 | File             | Position |
 +------------------+----------+
 | binlog.000000009 |     9710 |
 +------------------+----------+
```

```sql
3.查看binlog
mysqlbinlog  binlog.000007 --start-position --stop-position、--start-time= --stop-time
# at 21021
#190308 10:10:00 server id 1  end_log_pos 21094 CRC32 0x7a405abc     Query   thread_id=112   exec_time=0 error_code=0
SET TIMESTAMP=1552011000/*!*/;
BEGIN
/*!*/;

show binlog events in 'mysql-bin.000007' from position limit offset
+-------------+---+--------------+---------+-----------+------------------------------------+
|Log_name     |Pos|Event_type    |Server_id|End_log_pos|Info                                |
+-------------+---+--------------+---------+-----------+------------------------------------+
|binlog.000005|4  |Format_desc   |1        |125        |Server ver: 8.0.23, Binlog ver: 4   |
|binlog.000005|125|Previous_gtids|1        |156        |                                    |
|binlog.000005|156|Anonymous_Gtid|1        |235        |SET @@SESSION.GTID_NEXT= 'ANONYMOUS'|
+-------------+---+--------------+---------+-----------+------------------------------------+

```

binlog内容（rows格式为例）主要有：  
- position：事务位点。\(\# at 21021\)  
- timestamp：事务时间戳。\(190308 10:10:00\)  
- server id：mysql实例标识，区分不同实例的binlog。\(server id 1\)  
- thread\_id：执行该事务的线程。\(thread_id=112\)  
-_ Event\_type：事件类型。\(Query\)

### binlog持久化方式

事务提交时，server一次性将完整事务的 binlog cache写入binlog文件。

* 调用 write\(\) 则写入内存的Page Cache，由操作系统决定从内存到硬盘（持久化）的时机。
* 调用 fsync\(\) 则主动从内存的Page Cache写入硬盘（持久化）。

write 和 fsync 的时机，由参数 sync\_binlog 控制：

* sync\_binlog=0：每次提交事务只 write，不 fsync
* sync\_binlog=1 ：每次提交事务都会执行 fsync
* sync\_binlog=N\(N&gt;1\)：每次提交事务都 write，累积 N 个事务后才 fsync。

保证数据持久化时，sync\_binlog=0；从库需加快sql执行速度时，sync\_binlog=N。

## innodb log持久化

启动事务后，生成的redolog不会马上写入redolog文件（ib\_logfile+ 数字），而会先写入redolog buffer。

所以innodb log保存到log文件会分三个状态：redolog buffer，（write）Page Cache，（fsync）log文件。

write 和 fsync 的时机，由参数 innodb\_flush\_log\_at\_trx\_commit 控制：

* trx\_commit=0：每次提交事务，redolog留在redolog buffer中
* trx\_commit=1：每次提交事务都执行 fsync（持久化）
* trx\_commit=2：每次提交事务都执行 write（写入PageCache）

除了参数 innodb\_flush\_log\_at\_trx\_commit，innodb也会通过以下方式将log写入Page Cache或磁盘文件。



