# **3 Online WAL[Write Head Log]日志（redo)**

### **关于Online WAL日志** 


这个日志存在的目的是为了保证崩溃后的安全，如果系统崩溃，可以"重放"从最后一次检查点以来的日志项来恢复数据库的一致性。 **但是也存在日志膨胀问题** 


### **设置`Online WAL`日志** 

> pg提供如下参数控制wal日志的大小 

```
max_wal_size=1GB 
min_wal_size=8OMB
```

 
* **`max_wal_size`(integer)**

在自动WAL检查点之间允许WAL增长到的最大尺寸。这是一个软限制，在特殊的情况下WAL尺寸可能会超过`max_wal_size`， 例如在重度负荷下、`archive_command`失败或者高的`wal_keep_segments`设置。 

如果指定值时没有单位，则以兆字节为单位。默认为1GB。增加这个参数可能导致崩溃恢复所需的时间。这个参数只能在`postgresql.conf`中设置或者服务器命令行中设置。 


* **`min_wal_size`(integer)**

只要WAL磁盘用量保持在这个设置之下，在检查点时旧的WAL文件总是被回收以便未来使用，而不是直接被删除。这可以被用来确保有足够的WAL空间被保留来应付WAL使用的高峰，例如运行大型的批处理任务。

如果指定值时没有单位，则以兆字节为单位。默认是`80 MB`这个参数只能在`postgresql.conf`或者服务器命令行中设置。 


* **wal位置**
 
> `wal`在`＄PGDATA/pg_wal`下。10之前为`pg_xlog` 

```
cd /pgdata/12/data/pg_wal
[pg_wal]$ ls -trl
total 16384
drwx------. 2 postgres postgres        6 Apr 24 03:58 archive_status
-rw-------. 1 postgres postgres 16777216 Apr 26 04:42 000000010000000000000001
```

* **wal命名格式** 

文件名称为16进制的24个字符组成，每8个字符一组，每组的意义如下：

``` 
00000001 00000000 00000001 
时间线    逻辑id    物理id
```

* **查看WAL时间** 

```
postgres=# select pg_walfile_name(pg_current_wal_lsn());
     pg_walfile_name
--------------------------
 000000010000000000000001
(1 row)

postgres=# select * from pg_ls_waldir() order by modification asc;
           name           |   size   |      modification
--------------------------+----------+------------------------
 000000010000000000000001 | 16777216 | 2022-04-26 04:42:31+00
(1 row)
```

```
postgres=# select pg_current_wal_lsn();
 pg_current_wal_lsn
--------------------
 0/1654A38
(1 row)
```


* `pg_current_wal_lsn()` is a system function returning the current write-ahead log write location.
* `pg_walfile_name()` : is a system function for obtaining the name of the WAL file corresponding to the provided LSN( Log Sequence Number).
* `pg_ls_waldir()`: **returns a list of all files in the `pg_wal` directory together with their size and modification timestamp**.

* **切换WAL日志** 

```
postgres=# select pg_switch_wal();
 pg_switch_wal
---------------
 0/1654A50
(1 row)

postgres=# select * from pg_ls_waldir() order by modification asc;
           name           |   size   |      modification
--------------------------+----------+------------------------
 000000010000000000000001 | 16777216 | 2022-04-26 09:28:21+00
 000000010000000000000002 | 16777216 | 2022-04-26 09:28:22+00
(2 rows)
```


* `pg_switch_wal`: is a system function which **forces PostgreSQL to switch to a new WAL file**. `pg_switch_wal() → pg_lsn`

* **`pg_waldump`查看 wal** 

`pg_waldump`可以查看wal的具体内容 

```
postgres=# select pg_walfile_name(pg_current_wal_lsn());
     pg_walfile_name
--------------------------
 000000010000000000000002
(1 row)

postgres=# select * from pg_ls_waldir() order by modification asc;
           name           |   size   |      modification
--------------------------+----------+------------------------
 000000010000000000000003 | 16777216 | 2022-04-26 09:28:21+00
 000000010000000000000002 | 16777216 | 2022-04-26 09:32:37+00
(2 rows)
```

```
$ pg_waldump  000000010000000000000002
rmgr: Standby     len (rec/tot):     50/    50, tx:          0, lsn: 0/02000028, prev 0/01654A38, desc: RUNNING_XACTS nextXid 501 latestCompletedXid 500 oldestRunningXid 501
rmgr: Standby     len (rec/tot):     50/    50, tx:          0, lsn: 0/02000060, prev 0/02000028, desc: RUNNING_XACTS nextXid 501 latestCompletedXid 500 oldestRunningXid 501
rmgr: XLOG        len (rec/tot):    114/   114, tx:          0, lsn: 0/02000098, prev 0/02000060, desc: CHECKPOINT_ONLINE redo 0/2000060; tli 1; prev tli 1; fpw true; xid 0:501; oid 16396; multi 1; offset 0; oldest xid 480 in DB 1; oldest multi 1 in DB 1; oldest/newest commit timestamp xid: 0/0; oldest running xid 501; online
rmgr: Standby     len (rec/tot):     50/    50, tx:          0, lsn: 0/02000110, prev 0/02000098, desc: RUNNING_XACTS nextXid 501 latestCompletedXid 500 oldestRunningXid 501
pg_waldump: fatal: error in WAL record at 0/2000110: invalid record length at 0/2000148: wanted 24, got 0
```

```
postgres=# \c pg1
You are now connected to database "pg1" as user "postgres".

pg1=# \dt
        List of relations
 Schema | Name | Type  |  Owner
--------+------+-------+----------
 public | t1   | table | postgres
(1 row)

pg1=# insert into t1 values(2);
INSERT 0 1
```

```
$ cd /pgdata/12/data/pg_wal
[pg_wal]$ pg_waldump  000000010000000000000002
...
rmgr: Standby     len (rec/tot):     50/    50, tx:          0, lsn: 0/02000110, prev 0/02000098, desc: RUNNING_XACTS nextXid 501 latestCompletedXid 500 oldestRunningXid 501
rmgr: Heap        len (rec/tot):     54/   150, tx:        501, lsn: 0/02000148, prev 0/02000110, desc: INSERT off 2 flags 0x00, blkref #0: rel 1663/16384/16385 blk 0 FPW
rmgr: Transaction len (rec/tot):     34/    34, tx:        501, lsn: 0/020001E0, prev 0/02000148, desc: COMMIT 2022-04-26 09:41:34.914619 UTC
rmgr: Standby     len (rec/tot):     54/    54, tx:          0, lsn: 0/02000208, prev 0/020001E0, desc: RUNNING_XACTS nextXid 502 latestCompletedXid 500 oldestRunningXid 501; 1 xacts: 501
```

### **10 ARCH WAL log** 

在生产环境，为了保证数据高可用性，通常需要设置归档，所谓的归档，其实就是把`pg_wal`里面的日志备份出来，当系统故障后可以通过归 档的日志文件对数据进牙琳复。配置归档要开启如下参数： 

* `wal_level=replica` (pg13默认已经开启replica) 

该参数的可选的值有minimal,replica和 logical, wal的级别依次增高，在wal的信息也越多。由于minimal这一级别的wal不包含从基础的备份和wal日志重建数据的足够信息，在该模式下，无法开启wal日志归档 

* `archive_mode=on` 

上述参数为 on，表示打开归档备份，可选的参数为on, off, always默认值为off，所以要手动打开 

* `archive_command= 'test ! -f /mnt/server/archivedir/%f && cp %p /mnt/server/archivedir/%f'` 

该参数的默认值是一个空字符串，他的值可以是一条shell命令或者一个复杂的shell脚本。在shell月却本或命令中可以用"%p"表示将要归档的wal文件包含完整路径的信息的文件名，用"%f"代表不包含路径信息的wal文件的文件名

**注意：`wal_level`和`archive_mode`参数修改都需要重新启动数据库才可以生效。而修改archive command则不需要。所以一般配置新系 统时，无论当时是否需要归档，这要建议将这两个参数开启**

```
#------------------------------------------------------------------------------
# WRITE-AHEAD LOG
#------------------------------------------------------------------------------

# - Settings -

wal_level = replica                     # minimal, replica, or logical
                                        # (change requires restart)
fsync = on                              # flush data to disk for crash safety
                                        # (turning this off can cause
                                        # unrecoverable data corruption)
synchronous_commit = on         # synchronization level;
                                        # off, local, remote_write, remote_apply, or on
wal_sync_method = fsync         # the default is the first option
...

# - Checkpoints -

#checkpoint_timeout = 5min              # range 30s-1d
max_wal_size = 1GB
min_wal_size = 80MB
#checkpoint_completion_target = 0.5     # checkpoint target duration, 0.0 - 1.0
...
# - Archiving -

archive_mode = on               # enables archiving; off, on, or always
                                # (change requires restart)
archive_command =  'test ! -f /mnt/server/archivedir/%f && cp %p /mnt/server/archivedir/%f'             # command to use to archive a logfile segment
                                # placeholders: %p = path of file to archive
                                #               %f = file name only
```


**配置参数**

```
# - Archiving -

archive_mode = on               # enables archiving; off, on, or always
                                # (change requires restart)
archive_command =  'test ! -f /mnt/server/archivedir/%f && cp %p /mnt/server/archivedir/%f'             # command to use to archive a logfile 
```

**重启数据库**

```
$ pg_ctl restart -mf
waiting for server to shut down.... done
server stopped
waiting for server to start....2022-04-26 11:47:06.581 UTC [9661] LOG:  starting PostgreSQL 12.6 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 4.8.5 20150623 (Red Hat 4.8.5-44), 64-bit
2022-04-26 11:47:06.582 UTC [9661] LOG:  listening on IPv4 address "0.0.0.0", port 1921
2022-04-26 11:47:06.583 UTC [9661] LOG:  listening on Unix socket "/tmp/.s.PGSQL.1921"
2022-04-26 11:47:06.596 UTC [9661] LOG:  redirecting log output to logging collector process
2022-04-26 11:47:06.596 UTC [9661] HINT:  Future log output will appear in directory "log".
 done
server started
```

```
$ sudo -i
[root@jabox ~]# mkdir /archive
[root@jabox ~]# chown -R postgres. /archive
```


**插入数据，参看归档**

```
$ psql
psql (12.6)
Type "help" for help.

postgres=#\c pg1
You are now connected to database "pg1" as user "postgres".
pg1=# insert into t1 values(generate_series(1,10000000));
INSERT 0 10000000
```

```
pg1=# select pg_switch_wal();
 pg_switch_wal
---------------
 0/4F428450
(1 row)

pg1=# select pg_switch_wal();
 pg_switch_wal
---------------
 0/50000078
(1 row)
```


