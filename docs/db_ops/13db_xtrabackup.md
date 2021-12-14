# **第十三节 MySQL 物理备份 xtrabackup**

## **一、xtrabackup 介绍**

Xtrabackup 是一个开源的免费的热备工具，在 Xtrabackup 包中主要有 Xtrabackup 和 innobackupex 两个工具。

**其中 Xtrabackup 只能备份 InnoDB 和 XtraDB 两种引擎; innobackupex则 是封装了Xtrabackup，同时增加了备份MyISAM引擎的功能**。

Xtrabackup备份时不能备份表结构、触发器等等，也不能智能区分`.idb` 数据文件。

**另外 innobackupex还不能完全支持增量备份，需要和xtrabackup结合起来实现全备的功能。**


## **二、xtrbackup 安装**

### **1、下载安装包**

Xtrabackup 下载地址 https://www.percona.com/downloads/ 选择自己的版本下载。

[https://www.percona.com/downloads/Percona-XtraBackup-2.4/Percona-XtraBackup-2.4.18/binary/redhat/7/x86_64/Percona-XtraBackup-2.4.18-r29b4ca5-el7-x86_64-bundle.tar](https://www.percona.com/downloads/Percona-XtraBackup-2.4/Percona-XtraBackup-2.4.18/binary/redhat/7/x86_64/Percona-XtraBackup-2.4.18-r29b4ca5-el7-x86_64-bundle.tar) 


依赖包下载，下载地址 [http://rpmfind.net/linux/atrpms/el6-x86_64/atrpms/stable/libev-4.04- 2.el6.x86_64.rpm](http://rpmfind.net/linux/atrpms/el6-x86_64/atrpms/stable/libev-4.04- 2.el6.x86_64.rpm)。


### **2、解压安装**

```
# ls
libev-4.04-2.el6.x86_64.rpm Percona-XtraBackup-2.4.14-ref675d4-el7-x86_64-bundle.tar

# rpm -ivh libev-4.04-2.el6.x86_64.rpm
warning: libev-4.04-2.el6.x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID 
66534c2b: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:libev-4.04-2.el6                 ################################# [100%]
   
# tar -xf Percona-XtraBackup-2.4.14-ref675d4-el7-x86_64-bundle.tar

# ls
libev-4.04-2.el6.x86_64.rpm
Percona-XtraBackup-2.4.14-ref675d4-el7-x86_64-bundle.tar
percona-xtrabackup-24-2.4.14-1.el7.x86_64.rpm
percona-xtrabackup-24-debuginfo-2.4.14-1.el7.x86_64.rpm
percona-xtrabackup-test-24-2.4.14-1.el7.x86_64.rpm

# yum -y install percona-xtrabackup-24-2.4.14-
1.el7.x86_64.rpm # 这里用yum安装，因为还有其他的依赖包
```

### **3、配置文件**

修改配置文件`/etc/my.cnf`，**保证`[mysqld]`模块存在参数`datadir=/var/lib/mysql`（指向数据目 录），因为`xtrbackup`是根据`/etc/my.cnf`配置文件来获取需要备份的文件**。


### **4、重启mysqld**

## **三、xtrbackup 使用**

一般使用的是 `innobackupex` 脚本，因为 `innobackupex` 是 `perl` 脚本对 `xtrbackup` 的封装和功能扩展。


### **1、用户权限说明**

备份数据库时会涉及到两个用户：**系统用户与数据库内部的用户**。

**1)系统用户**

**需要在 datadir（配置文件内设置的目录）上具有读写执行权限（rwx）**。


**2)数据库内部用户**


* `RELOAD` 和 `LOCK TABLES` 权限，执行 `FLUSH TABLES WITH READ LOCK；`
* `REPLICATION CLIENT` 权限，获取 `binary log`（二进制日志文件）位置；
* `CREATE TABLESPACE`权限，导入表，用户表级别的恢复；
* SUPER权限，在slave环境下备份用来启动和关闭slave线程。


### **2、常用命令格式和常用参数**

**1)命令格式：**

```
innobackupex [参数] [目的地址|源地址]
```

**2)常用参数**

```
--user # 以什么用户身份进行操作
--password   # 数据库用户的密码
--port       # 数据库的端口号，默认3306
--stream   # 打包（数据流）
--defaults-file   # 指定默认配置文件，默认读取/etc/my.cnf
--no-timestamp   # 不创建时间戳文件，而改用目的地址（可以自动创建）
--copy-back           # 备份还原的主要选项
--incremental         # 使用增量备份，默认使用的完整备份
--incremental-basedir # 与--incremental选项联合使用，该参数指定上一级备份的地址来做增量备份
```

### **3、xtrbackup 完整备份和还原**

**1)完整备份**

```
# innobackupex --user=root --password=Qfedu.123com 
/backup/mysql/ # 有大量输出备份信息

# innobackupex --user=root --password=Qfedu.123com 
/backup/mysql/ 2>>/backup/mysql/backup.log # 将备份输出信息保存到文件

# ls /backup/mysql/
2019-06-16_15-49-44  2019-06-16_15-51-23 backup.log

# ls /backup/mysql/2019-06-16_15-49-44/
backup-my.cnf ibdata1 performance_schema xtrabackup_checkpoints 
xtrabackup_logfile ib_buffer_pool mysql sys xtrabackup_info

# innobackupex --user=root --password=Qfedu.123com --notimestamp /backup/mysql/test/ 2>>/backup/mysql/backup.log 
# 不使用时间戳创建目录，可自动创建目的地址

 ls /backup/mysql/
2019-06-16_15-49-44  2019-06-16_15-51-23 backup.log test

# ls /backup/mysql/test/
2019-06-16_15-54-20

# ls /backup/mysql/test/2019-06-16_15-54-20
backup-my.cnf   ibdata1 performance_schema xtrabackup_checkpoints 
xtrabackup_logfile
ib_buffer_pool mysql   sys                 xtrabackup_info
```

**2)数据还原**


**注意：`innobackupex-copy-back`不会覆盖已存在的文件。而且还原时需要先关闭服务，如果服务 是启动的，那么就不能还原到 `datadir`**

```
# systemctl stop mysqld
# rm -rf /var/lib/mysql/*             # 危险操作，请在测试环境测试

# innobackupex --copy-back /backup/mysql/2019-06-16_15-49-44/2>>/backup/mysql/copyback.log

# ll /var/lib/mysql
总用量 12324
-rw-r----- 1 root root      292 6月  16 17:08 ib_buffer_pool
-rw-r----- 1 root root 12582912 6月  16 17:08 ibdata1
drwxr-x--- 2 root root     4096 6月  16 17:08 mysql
drwxr-x--- 2 root root     8192 6月  16 17:08 performance_schema
drwxr-x--- 2 root root     8192 6月  16 17:08 sys
-rw-r----- 1 root root      423 6月  16 17:08 xtrabackup_info

# chown -R mysql:mysql /var/lib/mysql 
# 重新授权，否则mysqld无法启动

# systemctl start mysqld

# mysql -uroot -pQfedu.123com
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor. Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.26 MySQL Community Server (GPL)
Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.
Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql             |
| performance_schema |
| sys               |
+--------------------+
4 rows in set (0.00 sec)
```

### **4、xtrbackup 增量备份的还原**

增量备份的实现，依赖于innodb 页上面的 LSN（log sequence number），每次对数据库的修改 都会导致 LSN 自增。**增量备份会复制指定 LSN<日志序列号>之后的所有数据页**。

**1)查看完整备份的LSN**

```
# innobackupex --user=root --password=Qfedu.123com 
/backup/mysql/

# cat /backup/mysql/2020-03-12_16-45-17/xtrabackup_checkpoints
backup_type = full-backuped                          # 代表完整备份
from_lsn = 0
to_lsn = 2525919
last_lsn = 2525928
compact = 0
recover_binlog_info = 0
flushed_lsn = 2525928
```

**2)以全备创建增量备份**

创建一些数据，以`2020-03-12_16-45-17`时间戳创建第一个增量备份，并查看 LSN

```
# mysql -uroot -pQfedu.123com
mysql> create database test_db;
Query OK, 1 row affected (0.00 sec)
mysql> use test_db;
Database changed
mysql> create table user_tb(id int,name varchar(20));
Query OK, 0 rows affected (0.01 sec)
mysql> insert into user_tb values(1,'zhangsan');
Query OK, 1 row affected (0.00 sec)
mysql> exit
Bye

# innobackupex --user=root --password=Qfedu.123com --incremental /backup/mysql/ --incremental-basedir=/backup/mysql/2020-03-12_16-45-17/

# innobackupex --user=root --password=Qfedu.123com --incremental --incremental-basedir=/backup/mysql/2020-03-12_16-45-17/ /backup/mysql/ 2>>/backup/mysql/backup.log

# cat /backup/mysql/2020-03-12_16-48-08/xtrabackup_checkpoints
backup_type = incremental # 表示增量备份
from_lsn = 2525919
to_lsn = 2530689
last_lsn = 2530698
compact = 0
recover_binlog_info = 0
flushed_lsn = 2530698
```

**3)恢复数据准备全量备份**

```
# innobackupex --apply-log --redo-only /backup/mysql/2020-03-12_16-45-17/ 2>>/backup/mysql/copyback.log
```

**4)应用第一次增量备份到全量备份**

```
# innobackupex --apply-log --redo-only /backup/mysql/2020-03-12_16-45-17/ --incremental-dir=/backup/mysql/2020-03-12_16-48-08 2>>/backup/mysql/copyback.log
```

**5)查看全量备份的 `xtrabackup_checkpoints`**

```
#cat /backup/mysql/2019-06-16_15-49-44/xtrabackup_checkpoints
```

对比之前查看的第一次增量备份的 `last_lsn` 位置，在应用第一次增量备份到全量后，可以看到 `last_lsn ` 已经被应用和第一次全量备份的位置相同了

**6)把备份整体进行一次apply操作**

```
# innobackupex --apply-log /backup/mysql/2020-03-12_16-45-17/ 2>>/backup/mysql/copyback.log
```

**7)停止 mysql 服务并移除数据目录（生产环境建议用 mv ）**

```
# systemctl stop mysqld
# rm -rf /var/lib/mysql/* # 危险操作建议用 mv
```

**8)使用`--copy-back` 参数恢复拷贝到data目录**

```
# innobackupex --copy-back /backup/mysql/2020-03-12_16-45-17/ 2>/backup/mysql/copyback.log
```

**9)验证数据还原**

```
# ls -l /var/lib/mysql
```

**10)授权并启动MySQL**

```
# ls -l /var/lib/mysql
```

**11)验证数据完整性**

```
]# mysql -uroot -pQfedu.123com
mysql> use test_db;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A
Database changed
mysql> select * from user_tb;
+------+----------+
| id   | name     |
+------+----------+
| 1   | zhangsan |
| 2   | lisi     |
+------+----------+
2 rows in set (0.00 sec)
```

* 1）增量备份需要使用参数`--incremental`指定需要备份到哪个目录，使用`incremental-dir`指定全备目 录；
* 2）进行数据备份时，需要使用参数`--apply-log redo-only`先合并全备数据目录数据，确保全备数据目 录数据的一致性；
* 3）再将增量备份数据使用参数`--incremental-dir`合并到全备数据当中；
* 4）最后通过最后的全备数据进行恢复数据，注意，如果有多个增量备份，需要逐一合并到全备数据当中，再进行恢复。

### **5、xtrbackup 数据流压缩**

`-stream` 选项 


注意：使用`--stream`选项时，会输出打包的数据流，并不会直接生成打包文件，此时需要使用重定 向或其他命令对数据流进行处理。

**1)使用重定向生成压缩文件**

将标准输出重定向为tar文件，将标准错误重定向到日志文件

```
# innobackupex --databases=test_db --user=root --
password=Qfedu.123com --stream=tar /backup/mysql/> /backup/mysql/`date +%F`.tar 2> /backup/mysql/backup.log

# ls //backup/mysql/
2019-09-29.tar backup.log

# cd //backup/mysql/

# mkdir test_db.bak

# tar xf 2019-09-29.tar -C ./test_db.bak     
# 注意: 解压时指定解压目录，压缩中没有归档目录

# ls test_db.bak
ib_buffer_pool xtrabackup_binlog_info xtrabackup_logfile
backup.log     ibdata1         xtrabackup_checkpoints
backup-my.cnf   test_db         xtrabackup_info
```

**2)使用 ssh 和 cat 组合命令，直接备份到其他服务器上**

```
# ssh-keygen

# ssh-copy-id 192.168.5.70

# ssh root@192.168.5.70 "mkdir /backup/mysql"

# innobackupex --databases=test_db --user=root --
password=Qfedu.123com --stream=tar 2>/backup/mysql/backup.log | ssh 
root@192.168.5.70 "cat - > /backup/mysql/`date +%F`.tar "

# ssh 192.168.5.70

# ls /backup/mysql   # 查看备份文件2019-09-29.tar  

# mkdir /backup/mysql/test_db.bak

# tar xf /2019-09-29.tar -C /backup/mysql/test_db.bak/ # 解压查看解压结果

# ls /backup/mysql/test_db.bak
backup-my.cnf   ibdata1 xtrabackup_binlog_info xtrabackup_info
ib_buffer_pool test_db xtrabackup_checkpoints xtrabackup_logfile

# exit
```

**3)使用 gzip 再压缩一下**

```
# rm -rf /backup/mysql/*

# innobackupex --databases=test_db --user=root --password=Qfedu.123com --stream=tar /backup/mysql/ 2>/backup/mysql/backup.log |  gzip > /backup/mysql/`date +%F`.tar.gz

# ls /backup/mysql/
2019-09-29.tar.gz backup.log

# mkdir /backup/mysql/test_db.bak

# tar zxf /backup/mysql/2019-09-29.tar.gz -C 
/backup/mysql/test_db.bak

# ls /backup/mysql/test_db.bak     # 解压查看
ib_buffer_pool     xtrabackup_binlog_info xtrabackup_logfile
backup.log         ibdata1         xtrabackup_checkpoints
backup-my.cnf     test_db         xtrabackup_info
```



