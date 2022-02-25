mysql数据库状态解析 ：`show ENGINE INNODB STATUS`

```java

=====================================
2022-02-11 15:18:57 0x7f393133a700 INNODB MONITOR OUTPUT
=====================================
//表示的是过去46秒内数据库状态
Per second averages calculated from the last 46 seconds
-----------------
BACKGROUND THREAD
-----------------
//Master Thread的状态信息
srv_master_thread loops: 26841638 srv_active, 0 srv_shutdown, 5584 srv_idle
srv_master_thread log flush and writes: 26847222
----------
SEMAPHORES
----------
OS WAIT ARRAY INFO: reservation count 7863
OS WAIT ARRAY INFO: signal count 9477
RW-shared spins 0, rounds 24649, OS waits 7626
RW-excl spins 0, rounds 18708, OS waits 39
RW-sx spins 47, rounds 78, OS waits 0
Spin rounds per wait: 24649.00 RW-shared, 18708.00 RW-excl, 1.66 RW-sx
------------
TRANSACTIONS
------------
Trx id counter 8418681
Purge done for trx's n:o < 8418681 undo n:o < 0 state: running but idle
History list length 3
LIST OF TRANSACTIONS FOR EACH SESSION:
---TRANSACTION 421359552301776, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421359552300864, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421359552302688, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
---TRANSACTION 421359552304512, not started
0 lock struct(s), heap size 1136, 0 row lock(s)
--------
//在innodb存储引擎中大量使用了AIO（Async IO）来处理IO请求，并且使用IO Thread来处理这些IO请求的回调处理
FILE I/O
--------
I/O thread 0 state: waiting for i/o request (insert buffer thread)
I/O thread 1 state: waiting for i/o request (log thread)
I/O thread 2 state: waiting for i/o request (read thread)
I/O thread 3 state: waiting for i/o request (read thread)
I/O thread 4 state: waiting for i/o request (read thread)
I/O thread 5 state: waiting for i/o request (read thread)
I/O thread 6 state: waiting for i/o request (write thread)
I/O thread 7 state: waiting for i/o request (write thread)
I/O thread 8 state: waiting for i/o request (write thread)
I/O thread 9 state: waiting for i/o request (write thread)
Pending normal aio reads: [0, 0, 0, 0] , aio writes: [0, 0, 0, 0] ,
 ibuf aio reads:, log i/o's:, sync i/o's:
Pending flushes (fsync) log: 0; buffer pool: 0
512 OS file reads, 4978626 OS file writes, 4492863 OS fsyncs
0.00 reads/s, 0 avg bytes/read, 0.13 writes/s, 0.13 fsyncs/s
-------------------------------------
INSERT BUFFER AND ADAPTIVE HASH INDEX //插入缓冲
-------------------------------------
//size:代表已经合并记录页的数量
//seg size:显示当前Insert Buffer的大小 2 * 16KB
//free list len:代表了空闲列表的长度
Ibuf: size 1, free list len 0, seg size 2, 19 merges
merged operations:
 insert 0, delete mark 0, delete 0
discarded operations: //表示当Change Buffer发生merge时，表已经被删除，此时就无需再将记录合并到辅助索引中去
 insert 0, delete mark 0, delete 0
//以下是自适应hash
Hash table size 199321, node heap has 0 buffer(s)
Hash table size 199321, node heap has 0 buffer(s)
Hash table size 199321, node heap has 0 buffer(s)
Hash table size 199321, node heap has 0 buffer(s)
Hash table size 199321, node heap has 0 buffer(s)
Hash table size 199321, node heap has 0 buffer(s)
Hash table size 199321, node heap has 0 buffer(s)
Hash table size 199321, node heap has 0 buffer(s)
0.00 hash searches/s, 0.13 non-hash searches/s
---
LOG
---
Log sequence number 1249787179 //当前的redo log(in buffer)中的lsn
Log flushed up to   1249787179 //刷到redo log file on disk中的lsn
Pages flushed up to 1249479616 //已经刷到磁盘数据页上的LSN
Last checkpoint at  1249479616 //上一次检查点所在位置的LSN
0 pending log flushes, 0 pending chkp writes
4464325 log i/o's done, 0.13 log i/o's/second
----------------------
BUFFER POOL AND MEMORY
----------------------
Total large memory allocated 826540032
Dictionary memory allocated 2586715
Buffer pool size   49152  //缓冲池中页的数量
Free buffers       44941  //Free列表页的数量
Database pages     4211   //LRU列表页的数量
Old database pages 1540
//脏页的数量
Modified db pages  234
Pending reads      0
Pending writes: LRU 0, flush list 0, single page 0
//Pages made young：LRU列表中从midpoint后端到前端页的数量
//not young：LRU列表中从midpoint前端端到后端页的数量
Pages made young 489, not young 163
//youngs/s、non-youngs/s 每秒操作的次数
0.00 youngs/s, 0.00 non-youngs/s
Pages read 466, created 3797, written 499176
0.00 reads/s, 0.00 creates/s, 0.00 writes/s
//缓冲池的命中率，若为100%说明缓冲池运行状态非常好，若该值小于95%，用户需要观察由于全表扫描引起的LRU列表污染的问题
Buffer pool hit rate 1000 / 1000, young-making rate 0 / 1000 not 0 / 1000
Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
LRU len: 4211, unzip_LRU len: 0
I/O sum[0]:cur[0], unzip sum[0]:cur[0]
--------------
ROW OPERATIONS
--------------
0 queries inside InnoDB, 0 queries in queue
0 read views open inside InnoDB
Process ID=8606, Main thread ID=139883395925760, state: sleeping
Number of rows inserted 208407448, updated 3304984, deleted 1325, read 213737049
12.26 inserts/s, 0.07 updates/s, 0.00 deletes/s, 12.33 reads/s
----------------------------
END OF INNODB MONITOR OUTPUT
============================


```



