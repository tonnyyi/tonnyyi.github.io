---
title: MySQL进程号、连接ID、查询ID、InnoDB线程与系统线程如何对应
tags:
  - MySQL
categories:
  - MySQL
date: 2022-08-11 11:13:40
---

有时候，怀疑某个MySQL内存查询导致CPU或磁盘I/O消耗特别高，但又不确定具体是哪个SQL引起的。或者当InnoDB引擎内部有`semaphore wait`时，想知道具体是哪个线程/查询引起的。

> 当有`semaphore wait`事件超过600秒的话，InnoDB会发出crash信号：
>
> ```log
> InnoDB: ###### Diagnostic info printed to the standard error stream
> 2020-12-13T09:41:33.810011Z 0 [ERROR] [FATAL] InnoDB: Semaphore wait has lasted > 600 seconds. We intentionally crash the server because it appears to be hung.
> 2020-12-13 10:41:33 0x7f3d92a4e700 InnoDB: Assertion failure in thread 139902430013184 in file ut0ut.cc line 917
> InnoDB: We intentionally generate a memory trap.
> InnoDB: Submit a detailed bug report to http://bugs.mysql.com.
> InnoDB: If you get repeated assertion failures or crashes, even
> InnoDB: immediately after the mysqld startup, there may be
> InnoDB: corruption in the InnoDB tablespace. Please refer to
> InnoDB: http://dev.mysql.com/doc/refman/8.0/en/forcing-innodb-recovery.html
> InnoDB: about forcing recovery.
> 09:41:33 UTC - mysqld got signal 6 ;
> ```
>
> 因此也要监控InnoDB的`semaphore wait`状态，一旦超过阈值，就要尽快报警并分析出问题原因，及时杀掉或停止引起等待的查询请求。

## 1、操作系统进程ID

MySQL是一个单进程多线程的服务程序，用 `ps -ef|grep mysqld` 就能看到其系统进程ID了。另外，当 `my.cnf` 配置文件中增加一行 `innodb_status_file = 1` 时，也会生成带有系统进程ID的innodb status 文件

```bash
[root@172 ~]# ps -ef | grep mysqld
mysql    114560      1  0  2021 ?        16:54:04 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid

[root@yejr.run]# ls -la innodb_status.38801
-rw-r----- 1 mysql mysql 4906 Jun 14 14:26 innodb_status.38801
```

文件 `innodb_status.pid` 的作用是每隔15秒左右输出innodb引擎各种状态信息，和执行 `SHOW ENGINE INNODB STATUS` 的作用相同。二者的区别在于，前者（文件输出方式）的输出内容长度不受限制，而后者（命令行输出）则最多只显示1MB内容，更多的会被截断。所以务必设置 `innodb_status_file = 1` 选项。

## 2、系统线程和MySQL连接ID、查询ID等的关系

从MySQL 5.7开始，`performance_schema.threads` 表增加 `THREAD_OS_ID` 列，用于记录MySQL内部线程对应的系统线程ID。

```bash
mysql> show processlist;
+---------+------+----------------------+----------------------+---------+------+----------+------------------+
| Id      | User | Host                 | db                   | Command | Time | State    | Info             |
+---------+------+----------------------+----------------------+---------+------+----------+------------------+
| 2123567 | root | 10.1.117.18:62734    | test                 | Query   |    0 | starting | show processlist |

mysql> begin;select *,sleep(1000) from msg_log for update;
Query OK, 0 rows affected (0.00 sec)		<-- 这个SQL会运行很长时间，方便我们观察
```

新开一个窗口，查看 `performance_schema.threads` 表：

```bash
mysql> select * from threads where processlist_id = 2123567\G
*************************** 1. row ***************************
          THREAD_ID: 2123593		<-- MySQL内部线程ID，也是PFS的内部计数器
               NAME: thread/sql/one_connection
               TYPE: FOREGROUND
     PROCESSLIST_ID: 2123567
   PROCESSLIST_USER: root
   PROCESSLIST_HOST: 10.1.117.18
     PROCESSLIST_DB: test
PROCESSLIST_COMMAND: Query
   PROCESSLIST_TIME: 3
  PROCESSLIST_STATE: User sleep
   PROCESSLIST_INFO: select *,sleep(1000) from msg_log for update	<-- 正在运行的SQL
   PARENT_THREAD_ID: NULL
               ROLE: NULL
       INSTRUMENTED: YES
            HISTORY: YES
    CONNECTION_TYPE: TCP/IP
       THREAD_OS_ID: 20072	<-- 对应操作系统的线程ID
1 row in set (0.00 sec)
```

运行 ps -Lef 查看对应的系统线程

```bash
[root@172 run]# ps -Lef | grep 20072	<-- 上面查询看到 THREAD_OS_ID 列的值
mysql    114560      1  20072  0  140 7月04 ?       00:01:41 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid
```

如果设置了 `general_log=1`，那么也能看到 `general_log` 里有这样的记录：

```bash
[root@yejr.run]# cat yejr.run.log

/usr/local/GreatSQL-8.0.22/bin/mysqld, Version: 8.0.22-13 (Source distribution). started with:
Tcp port: 6001  Unix socket: mysql.sock
#几个列分别是：时间、连接ID、请求类型、详细的SQL
Time                 Id Command    Argument
...
2021-06-14T14:46:47.474393+08:00    25 Query select *,sleep(1000) from t1 for update  <-- 可以看到连接ID是25
...
```

再查询 `pfs.events_statements_current` 表：

```bash
mysql> select * from events_statements_current where thread_id = 2123593\G
*************************** 1. row ***************************
              THREAD_ID: 2123593
               EVENT_ID: 22
           END_EVENT_ID: NULL
             EVENT_NAME: statement/sql/select
                 SOURCE: socket_connection.cc:98
            TIMER_START: 5875987852231112768
              TIMER_END: 5876157107557196768
             TIMER_WAIT: 169255326084000
              LOCK_TIME: 79000000
               SQL_TEXT: select *,sleep(1000) from msg_log for update
...
            MYSQL_ERRNO: 0
      RETURNED_SQLSTATE: NULL
           MESSAGE_TEXT: NULL
                 ERRORS: 0
               WARNINGS: 0
          ROWS_AFFECTED: 0
              ROWS_SENT: 0
          ROWS_EXAMINED: 0
CREATED_TMP_DISK_TABLES: 0
     CREATED_TMP_TABLES: 0
       SELECT_FULL_JOIN: 0
 SELECT_FULL_RANGE_JOIN: 0
           SELECT_RANGE: 0
     SELECT_RANGE_CHECK: 0
            SELECT_SCAN: 1
      SORT_MERGE_PASSES: 0
             SORT_RANGE: 0
              SORT_ROWS: 0
              SORT_SCAN: 0
          NO_INDEX_USED: 1
     NO_GOOD_INDEX_USED: 0
       NESTING_EVENT_ID: NULL
     NESTING_EVENT_TYPE: NULL
    NESTING_EVENT_LEVEL: 0
1 row in set (0.00 sec)
```

执行 `SHOW ENGINE INNODB STATUS\G` 查看事务状态：

```bash
mysql> show engine innodb status\G
*************************** 1. row ***************************
  Type: InnoDB
  Name: 
Status: 
=====================================
2022-08-11 11:30:05 0x7f74ffc63700 INNODB MONITOR OUTPUT
=====================================
Per second averages calculated from the last 13 seconds
-----------------
BACKGROUND THREAD
-----------------
srv_master_thread loops: 138748 srv_active, 0 srv_shutdown, 42617474 srv_idle
srv_master_thread log flush and writes: 42755230
----------
SEMAPHORES
----------
OS WAIT ARRAY INFO: reservation count 313118
OS WAIT ARRAY INFO: signal count 346631
RW-shared spins 0, rounds 1346296, OS waits 228949
RW-excl spins 0, rounds 2231816, OS waits 19003
RW-sx spins 22615, rounds 179786, OS waits 2054
Spin rounds per wait: 1346296.00 RW-shared, 2231816.00 RW-excl, 7.95 RW-sx
------------
TRANSACTIONS
------------
Trx id counter 3241619
Purge done for trx's n:o < 3241619 undo n:o < 0 state: running but idle
History list length 2155
LIST OF TRANSACTIONS FOR EACH SESSION:
---TRANSACTION 421617022724832, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
...
# # 事务ID=3241573，运行时长=274秒
---TRANSACTION 3241573, ACTIVE 274 sec
mysql tables in use 1, locked 1
2 lock struct(s), heap size 1136, 1 row lock(s)
# MySQLprocesslist_id=2123567，OS线程句柄 = 140140486006528（后面再介绍），查询ID=129726058（对应上面的 STATEMENT_ID）
MySQL thread id 2123567, OS thread handle 140140486006528, query id 129726058 10.1.117.18 root User sleep
select *,sleep(1000) from msg_log for update
--------
FILE I/O
--------
...
Pending normal aio reads: [0, 0, 0, 0] , aio writes: [0, 0, 0, 0] ,
 ibuf aio reads:, log i/o's:, sync i/o's:
Pending flushes (fsync) log: 0; buffer pool: 0
6115009 OS file reads, 4390358 OS file writes, 1258908 OS fsyncs
0.00 reads/s, 0 avg bytes/read, 0.69 writes/s, 0.62 fsyncs/s
-------------------------------------
INSERT BUFFER AND ADAPTIVE HASH INDEX
-------------------------------------
Ibuf: size 1, free list len 1175, seg size 1177, 75546 merges
merged operations:
 insert 1051343, delete mark 2047, delete 236
discarded operations:
 insert 0, delete mark 0, delete 0
...
1.77 hash searches/s, 20.77 non-hash searches/s
---
LOG
---
Log sequence number 15576583193
Log flushed up to   15576583193
Pages flushed up to 15576583193
Last checkpoint at  15576583184
0 pending log flushes, 0 pending chkp writes
721969 log i/o's done, 0.38 log i/o's/second
----------------------
BUFFER POOL AND MEMORY
----------------------
Total large memory allocated 549715968
Dictionary memory allocated 11590700
Buffer pool size   32768
Free buffers       25473
Database pages     7275
Old database pages 2704
Modified db pages  0
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
Pages made young 3545194, not young 64706515
0.00 youngs/s, 0.00 non-youngs/s
Pages read 6112248, created 432271, written 3434310
0.00 reads/s, 0.00 creates/s, 0.23 writes/s
Buffer pool hit rate 1000 / 1000, young-making rate 0 / 1000 not 0 / 1000
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 7275, unzip_LRU len: 0
I/O sum[12]:cur[0], unzip sum[0]:cur[0]
--------------
ROW OPERATIONS
--------------
0 queries inside InnoDB, 0 queries in queue
0 read views open inside InnoDB
Process ID=114560, Main thread ID=140141670680320, state: sleeping
Number of rows inserted 24852529, updated 226240, deleted 170151, read 117384145467
0.00 inserts/s, 0.08 updates/s, 0.00 deletes/s, 163966.62 reads/s
----------------------------
END OF INNODB MONITOR OUTPUT
============================

1 row in set (0.00 sec)
```

## 3、OS thread handle和操作系统线程ID的对应关系

首先，`OS thread handle 140140486006528` （OS thread handle是进程内部用于识别各个线程的内部ID），这里是个十进制的数值，需要先转成十六进制（有时候可能会直接用十六进制表示，这个问题也有人提出了 MDEV-17237）：

```bash
mysql> select lower(conv(140140486006528, 10, 16));
+--------------------------------------+
| lower(conv(140140486006528, 10, 16)) |
+--------------------------------------+
| 7f74ffe31700                         |
+--------------------------------------+
1 row in set (0.00 sec)
```

再利用 pstack 查询该句柄和操作系统线程ID的关联：

```bash
[root@172 run]# pstack `pidof mysqld` | grep 7f74ffe31700
Thread 62 (Thread 0x7f74ffe31700 (LWP 20072)):
```

可以看到 `LWP = 20072`，对应上面的 `THREAD_OS_ID` 值，LWP是Light-Weight Processes的缩写（轻量级进程）。用 pidstat 也能看到这个LWP：

```bash
[root@172 run]# pidstat -t -p 114560 | grep 20072
12时15分16秒    27         -     20072    0.00    0.00    0.00    0.00     0  |__mysqld
```

**运行pstack会短暂阻塞mysqld进程，所以请切勿在业务高峰期执行，除非万不得已**。

有时候可能会看到类似下面的 innodb status：

```
SEMAPHORES
----------
OS WAIT ARRAY INFO: reservation count 15143
--Thread 140585128785664 has waited at buf0flu.cc line 1209 for 237 seconds the semaphore:
SX-lock on RW-latch at 0x7fdb1fbe3f80 created in file buf0buf.cc line 1460
a writer (thread id 140584786024192) has reserved it in mode SX
number of readers 0, waiters flag 1, lock_word: 10000000
Last time read locked in file row0sel.cc line 3758
Last time write locked in file /export/home/pb2/build/sb_0-34537258-1560179931.8/mysql-5.7.27/storage/innobase/fsp/fsp0fsp.cc line 167
```

在上面这段信息中，线程 `Thread 140585128785664` 在 `buf0flu.cc` 代码1209行这里等待了237秒，想要请求 SX-lock 的 RW-latch，被另一个线程 `thread id 140584786024192` 给阻塞了，它持有的是 SX latch。

`pfs.events_statements_current` 中只能查到当前执行的SQL，可以通过 `pfs.events_statements_history` 查看最近执行过的SQL。

```bash
mysql> select * from events_statements_history where thread_id =  2123593\G
*************************** 1. row ***************************
              THREAD_ID: 2123593
               EVENT_ID: 21
           END_EVENT_ID: 21
             EVENT_NAME: statement/sql/begin
                 SOURCE: socket_connection.cc:98
            TIMER_START: 5875987850804953768
              TIMER_END: 5875987850871963768
             TIMER_WAIT: 67010000
              LOCK_TIME: 0
               SQL_TEXT: begin
                 DIGEST: f57daa74a09445d1e1c496f28fe6d906
            DIGEST_TEXT: BEGIN 
         CURRENT_SCHEMA: test
            OBJECT_TYPE: NULL
          OBJECT_SCHEMA: NULL
            OBJECT_NAME: NULL
  OBJECT_INSTANCE_BEGIN: NULL
            MYSQL_ERRNO: 0
      RETURNED_SQLSTATE: 00000
           MESSAGE_TEXT: NULL
                 ERRORS: 0
               WARNINGS: 0
          ROWS_AFFECTED: 0
              ROWS_SENT: 0
          ROWS_EXAMINED: 0
CREATED_TMP_DISK_TABLES: 0
     CREATED_TMP_TABLES: 0
       SELECT_FULL_JOIN: 0
 SELECT_FULL_RANGE_JOIN: 0
           SELECT_RANGE: 0
     SELECT_RANGE_CHECK: 0
            SELECT_SCAN: 0
      SORT_MERGE_PASSES: 0
             SORT_RANGE: 0
              SORT_ROWS: 0
              SORT_SCAN: 0
          NO_INDEX_USED: 0
     NO_GOOD_INDEX_USED: 0
       NESTING_EVENT_ID: NULL
     NESTING_EVENT_TYPE: NULL
    NESTING_EVENT_LEVEL: 0
*************************** 2. row ***************************
              THREAD_ID: 2123593
               EVENT_ID: 22
           END_EVENT_ID: 22
             EVENT_NAME: statement/sql/select
                 SOURCE: socket_connection.cc:98
            TIMER_START: 5875987852231112768
              TIMER_END: 5876987852493681768
             TIMER_WAIT: 1000000262569000
              LOCK_TIME: 79000000
               SQL_TEXT: select *,sleep(1000) from msg_log for update
                 DIGEST: dc7234f2ef044c6ee131ae4275cca79a
            DIGEST_TEXT: SELECT * , `sleep` (?) FROM `msg_log` FOR UPDATE 
         CURRENT_SCHEMA: test
            OBJECT_TYPE: NULL
          OBJECT_SCHEMA: NULL
            OBJECT_NAME: NULL
  OBJECT_INSTANCE_BEGIN: NULL
            MYSQL_ERRNO: 0
      RETURNED_SQLSTATE: NULL
           MESSAGE_TEXT: NULL
                 ERRORS: 0
               WARNINGS: 0
          ROWS_AFFECTED: 0
              ROWS_SENT: 1
          ROWS_EXAMINED: 1
CREATED_TMP_DISK_TABLES: 0
     CREATED_TMP_TABLES: 0
       SELECT_FULL_JOIN: 0
 SELECT_FULL_RANGE_JOIN: 0
           SELECT_RANGE: 0
     SELECT_RANGE_CHECK: 0
            SELECT_SCAN: 1
      SORT_MERGE_PASSES: 0
             SORT_RANGE: 0
              SORT_ROWS: 0
              SORT_SCAN: 0
          NO_INDEX_USED: 1
     NO_GOOD_INDEX_USED: 0
       NESTING_EVENT_ID: NULL
     NESTING_EVENT_TYPE: NULL
    NESTING_EVENT_LEVEL: 0
...    
```



> 来源: [『叶问』#40，MySQL进程号、连接ID、查询ID、InnoDB线程与系统线程如何对应](https://imysql.com/2021/12/15/yewen-40-relationship-processid-connectionid-queryid-innodb-threadid-os-threadid.shtml)
