# **5 备份恢复恢复**

## **5.1、需要备份什么？** 

**数据 / WAL日志 / 归档日志** 
 
## **5.2、备份方式** 

### **5.2.1 逻辑备份工具** 

**`pg_dump` /  `pg_dumpall`**

### **5.2.2 物理备份工具** 

**`pg_basebak up`** 

## **5.3. `pg_basebackup`物理备份工具应用** 

### **5.3.1 备份**

```
pg_basebackup -D /pgdata/pg_back -Ft -Pv -Upostgres -h 192.168.1.44 -p 1921 -R
```
 
```
cd $PGDATA
$ pg_dump -d pg1 > /tmp/pg1.sql
$ cd /tmp
$ ll
total 154084
-rw-rw-r--. 1 postgres postgres 157778626 Apr 27 00:44 pg1.sql
```

```
$ psql < /tmp/pg1.sql
SET
SET
SET
SET
SET
 set_config
------------

(1 row)

SET
SET
SET
SET
SET
SET
CREATE TABLE
ALTER TABLE
COPY 20000002
```

### **5.3.2 恢复**

```
$ cd /pgdata/
$ mkdir pg_backup
$ chown -R postgres. pg_backup/

$ ll -lt
total 0
drwxrwxr-x. 2 postgres postgres  6 Apr 27 01:56 pg_backup
```


**`vim pg_hba.conf`**

add 

```
...
host    replication     all             0.0.0.0/0               trust
```

```
$ pg_ctl restart -mf
```

```
pg_basebackup -D /pgdata/pg_back -Ft -Pv -Upostgres -h 192.168.1.44 -p 1921 -R


$ pg_basebackup -D /pgdata/pg_back -Ft -Pv -Upostgres -h 192.168.1.44 -p 1921 -R
pg_basebackup: initiating base backup, waiting for checkpoint to complete
pg_basebackup: checkpoint completed
pg_basebackup: write-ahead log start point: 0/61000028 on timeline 1
pg_basebackup: starting background WAL receiver
pg_basebackup: created temporary replication slot "pg_basebackup_18222"
1458149/1458149 kB (100%), 1/1 tablespace
pg_basebackup: write-ahead log end point: 0/61000138
pg_basebackup: waiting for background process to finish streaming ...
pg_basebackup: syncing data to disk ...
pg_basebackup: base backup completed
```

```
$ cd /pgdata/pg_back
$ ls
base.tar  pg_wal.tar
```

**Demo**

```
pg_ctl stop -mf
rm -rf $PGDATA/*
rm -rf /archive/*

cp base.tar $PGDATA/
cp pg_wal.tar $PGDATA/archive/

tar xf base.tar -C $PGDATA/12/data
tar xf pg_wal.tar-C /archive/


# vim postgresql.auto.conf
restore_command = 'cp /archive/%f %p'
recovery_target = 'immediate'

pg_ctl start -mf
touch $PGDATA/recovery.signal
```

```
select pg_wal_replay_resume()
```

## **5.4 PITR实战应用**

### **5.4.1 场景介绍**

* 每天23: 00 PBK备份，周二下午14: 00误删除数据，如何恢复？ 
	* 恢复全备数据`tar xf -c `
	* 归档恢复：备份归档`＋23-14`区间的归档+在线redo 

```
create database pit;
\c pit

create table t1(id int);

insert into t1 values(1);
insert into t1 values(111);
insert into t1 values(1111);

create table t2(id int);
insert into t2 values(1);
insert into t2 values(2);

drop database pit;  # drop the table
```

```
# change to new log 
select pg_switch_wal()
```

**stop server and back up**

```
pg_ctl stop -mf

rm -rf $PGDATA/*
rm -rf /archive/*


$ pg_basebackup -D /pgdata/pg_back -Ft -Pv -Upostgres -h 192.168.1.44 -p 1921 -R
tar xf base.tar -C $PGDATA/12/data
tar xf pg_wal.tar-C /archive/
```


**check the XID（traction ID）before the drop action**

```
cd /archive/

pg_waldump 0000000....XXX
# the XID before drop is 498

# vim postgresql.auto.conf
restore_command = 'cp /archive/%f %p'
recovery_target = '498'
```

```
pg_ctl start 

postgres=# create database aaaa; ERROR: 
cannot execute CREATE DATABASE in a read-only transaction 

postgres=# select  pg_wal_replay_resume(); 
pg_wal_replay_resume 
------------------
(1 row) 
```

### **5.4.2 对于风险极大的操作，可以预先执行一个特殊备份**

```
postgres=# select pg_create_restore_point('test-before-delete'); 
```





