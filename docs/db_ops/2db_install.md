## **第二节 安装 MySQL Repository**

## **1、默认 yum 存储库安装**

```
$ yum -y install wget     # 安装 wget下载工具
$ wget https://repo.mysql.com/mysql57-community-release-el7-11.noarch.rpm # 下载 mysql 官方 yum 源安装包

$ yum -y localinstall mysql57-community-release-el7-11.noarch.rpm # 安装 mysql 官方 yum 源
```
 
## **2、选择指定发行版本安装**

使用 MySQL Yum 存储库时，默认情况下会选择要安装的最新GA版本MySQL。默认启用最新GA 系列（当前为MySQL 8.0）的子存储库，而所有其他系列（例如，MySQL 5.7系列）的子存储库均 被禁用。查看已启用或禁用了存储库。

### **2-1 列出所有版本**

```
$ yum repolist all | grep mysql
```

发现启用最新`8.0`版本是 `enabled` 的，`5.7`版本是 `disabled` 的，需要安装5.7版本时，所以把8.0的进行禁用，然后再启用5.7版本

### **2-2 安装 yum 配置工具**

```
$ yum -y install yum-utils
```

### **2-3 禁用 8.0 版本**

```
$ yum-config-manager --disable mysql80-community
```

### **2-4 启用 5.7 版本**

```
yum-config-manager --enable mysql57-community
```

### **2-5 检查启用版本**

注意：进行安装时请确保只有一个版本启用，否则会显示版本冲突

```
$ yum repolist enabled | grep mysql
```

## **3、安装 MySQL**

需要安装MySQL Server, MySQL client 已经包括在 server 套件内

```
$ yum -y install mysql-community-server mysql     # 安装服务端，客户端

$ systemctl start mysqld # 启动 mysql服务

$ systemctl enable mysqld             # 设置 mysql服务开机启动

$ ls /var/lib/mysql           # 查看 mysql安装

$  grep 'tqfeduorary password' /var/log/mysqld.log # 获取首次登录密码

$ mysql -uroot -p'awm3>!QFl6zR'         # 登录mysql数据库

mysql > alter user 'root'@'localhost' identified by 'Qf.123com';     # 修改 mysql 数据库密码（密码必须符合复杂性要求，包含字母大小写，数字，特赦符号，长度不少于8位）

$ mysql -uroot -p'Qf.123com'                       # 用新密码登录数据库
```

**重启 MySQL**

```
重启 MySQL
```

**创建 qfedu 库并设置权限**

```
mysql -u root -p
Enter password:
Welcome to the MySQL monitor. Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.6.39 MySQL Community Server (GPL)
Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.
Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
mysql>CREATE DATABASE qfedudb CHARACTER SET utf8 COLLATE utf8_bin;
Query OK, 1 row affected (0.00 sec)
mysql>GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER,INDEX on qfedudb.* TO
'qfedu'@'localhost' IDENTIFIED BY 'Yangge.123com';
Query OK, 0 rows affected, 1 warning (0.00 sec)
mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)
mysql> SHOW GRANTS FOR 'qfedu'@'localhost';
+-------------------------------------------------------------------------------
------------------------------+
| Grants for qfedu@localhost                                                    
                              |
+-------------------------------------------------------------------------------
------------------------------+
| GRANT USAGE ON *.* TO 'qfedu'@'localhost' IDENTIFIED BY PASSWORD
'*841E9705B9F4BD3195B7314CA58A7E3B3B349F71' |
| GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER ON
`qfedudb`.* TO 'qfedu'@'localhost'       |
+-------------------------------------------------------------------------------
------------------------------+
2 rows in set (0.00 sec)
mysql> SHOW GRANTS FOR 'qfedu'@'172.16.0.122';
+-------------------------------------------------------------------------------
---------------------------------+
| Grants for qfedu@172.16.0.122
|
+-------------------------------------------------------------------------------
---------------------------------+
| GRANT USAGE ON *.* TO 'qfedu'@'172.16.0.122' IDENTIFIED BY PASSWORD
'*841E9705B9F4BD3195B7314CA58A7E3B3B349F71' |
| GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER ON
`qfedudb`.* TO 'qfedu'@'172.16.0.122'       |
+-------------------------------------------------------------------------------
---------------------------------+
2 rows in set (0.01 sec)
mysql> \q
Bye
```


## **4、Jam 数据库安装**

### **4-1 Download these rpm:**

```
ssh dc25jamint3db01

$ sudo -i

dc25jamint3db01:~ # mkdir tmp && cd tmp

wget https://downloads.mysql.com/archives/get/p/23/file/mysql-community-server-5.7.30-1.sles12.x86_64.rpm

wget https://downloads.mysql.com/archives/get/p/23/file/mysql-community-client-5.7.30-1.sles12.x86_64.rpm

wget https://downloads.mysql.com/archives/get/p/23/file/mysql-community-devel-5.7.30-1.sles12.x86_64.rpm

wget https://downloads.mysql.com/archives/get/p/23/file/mysql-community-embedded-devel-5.7.30-1.sles12.x86_64.rpm

wget https://downloads.mysql.com/archives/get/p/23/file/mysql-community-common-5.7.30-1.sles12.x86_64.rpm

wget https://downloads.mysql.com/archives/get/p/23/file/mysql-community-libs-5.7.30-1.sles12.x86_64.rpm

wget https://downloads.mysql.com/archives/get/p/23/file/mysql-community-test-5.7.30-1.sles12.x86_64.rpm

wget https://downloads.mysql.com/archives/get/p/23/file/mysql-community-embedded-5.7.30-1.sles12.x86_64.rpm

 
sudo zypper install libatomic1
sudo zypper install perl-JSON

rpm -ivh *.rpm
```


```
$ rpm -qa | grep mysql
mysql-community-libs-5.7.30-1.sles12.x86_64
mysql-community-embedded-devel-5.7.30-1.sles12.x86_64
mysql-community-server-5.7.30-1.sles12.x86_64
mysql-community-test-5.7.30-1.sles12.x86_64
mysql-community-common-5.7.30-1.sles12.x86_64
mysql-community-devel-5.7.30-1.sles12.x86_64
mysql-community-embedded-5.7.30-1.sles12.x86_64
mysql-community-client-5.7.30-1.sles12.x86_64

$ rpm -qa | grep perl-JSON
perl-JSON-2.90-2.15.noarch

$ rpm -qa | grep libatomic1
libatomic1-11.2.1+git610-1.3.2.x86_64
```

### **4-2 Edit `/etc/my.cnf`**

* Attention that master and slave have the different configuration file

```
[client]
port            = 3306
socket          = /mysqldata/data/mysqld.sock
#character-sets-dir=/opt/mysql/current/charsets

[mysqld_safe]
socket          = /mysqldata/data/mysqld.sock
nice            = 0

[mysqld]
#large-pages
auto-increment-increment = 2
auto-increment-offset = 1
read_only=OFF
# read_only=ON   SLAVE

character-set-server = utf8
collation-server = utf8_unicode_ci
performance_schema = ON
innodb_stats_persistent_sample_pages=512
#large_pages=0
#skip-name-resolve

user            = mysql
pid-file        = /mysqldata/data/mysqld.pid
socket          = /mysqldata/data/mysqld.sock
port            = 3306
basedir         = /opt/mysql/current
datadir         = /mysqldata/data
tmpdir          = /mysqltemp/tmp
#language        = /opt/mysql/current/share/english
skip-external-locking

#key_buffer = 384M
max_allowed_packet      = 100M
thread_stack            = 1M
#thread_cache_size       = 8
max_connections        = 1024
table_open_cache            = 3000
#thread_concurrency     = 8
innodb_thread_concurrency = 8
sort_buffer_size = 64M
read_buffer_size = 32M
read_rnd_buffer_size = 256M
myisam_sort_buffer_size = 256M
thread_cache_size = 250
query_cache_limit       = 0
query_cache_type = 0
query_cache_size = 0
wait_timeout=300
join_buffer_size = 18M
tmp_table_size= 2G
max_heap_table_size=1G
log-slave-updates = ON
sql_mode                = NO_ENGINE_SUBSTITUTION,STRICT_ALL_TABLES
innodb_data_home_dir = /mysqldata/data/
innodb_data_file_path = ibdata1:2000M;ibdata2:10M:autoextend
innodb_log_group_home_dir = /mysqldata/data/

innodb_buffer_pool_size = 15G
#innodb_additional_mem_pool_size = 64M
innodb_adaptive_hash_index_parts = 18
innodb_log_file_size = 512M
innodb_log_buffer_size = 32M
innodb_flush_log_at_trx_commit = 1
innodb_flush_method=O_DIRECT
innodb_read_io_threads = 16
innodb_write_io_threads = 16
innodb_lock_wait_timeout = 50
innodb_file_per_table
innodb_buffer_pool_instances = 8
innodb_buffer_pool_dump_pct = 75
#innodb_numa_interleave = 1
safe_user_create  = 1
local-infile  = 0
# Log monitoring output to a file (inndb_status.<pid> in the data directory) when monitoring is enabled
innodb-status-file=1

server-id               = 888
log_bin                 = /mysqllog/data/mysql-bin
log_bin_index           = /mysqllog/data/mysql-bin.index
expire_logs_days        = 7
max_binlog_size         =1073741824
binlog_format           =ROW
binlog_error_action     =IGNORE_ERROR
sync_binlog             =1
log_error               =/mysqllog/data/mp13jamdb46.err
#log-slow-queries       =/mysqllog/data/mp13jamdb46-slow-queries.log
#log-queries-not-using-indexes
long_query_time         =2
slow_query_log = 1
replicate-wild-ignore-table = roltatusc.%
replicate-wild-ignore-table = cmdb.%
relay-log-info-repository=TABLE
master-info-repository=TABLE

[mysqldump]
quick
quote-names
max_allowed_packet      = 100M

[mysql]
#no-auto-rehash # faster start of mysql but no tab completition

[isamchk]
key_buffer = 256M
sort_buffer_size = 256M
read_buffer = 2M
write_buffer = 2M
```

### **4-3 change user to root**

```
$ sudo userdel -r mysql

no crontab for mysql
userdel: mysql mail spool (/var/mail/mysql) not found


$ sudo groupdel mysql

$ sudo groupadd -r -g 116 mysql

* -g, --gid GID
* -r, --system

Create a system group.


$ sudo useradd -g mysql -u 116 -M mysql
```
```
sudo su root

chown mysql:mysql /opt/mysql/

chown mysql:mysql /app

chown mysql:mysql /var/run/mysql/

## mkdir /mysqlbackup /mysqldata /mysqllog /mysqltemp #if not exist

cd /mysql
mysqlbackup/ mysqldata/   mysqllog/    mysqltemp/

chown mysql:mysql /mysql*
```

```
/ # ls -la | grep mysql
drwxr-xr-x   3 mysql    mysql       18 Oct 22 16:53 app
drwxr-xr-x   2 mysql    mysql     4096 Oct 26 15:08 mysqlbackup
drwxr-xr-x   2 mysql    mysql        6 Oct 22 16:51 mysqldata
drwxr-xr-x   2 mysql    mysql        6 Oct 22 16:51 mysqllog
drwxr-xr-x   2 mysql    mysql        6 Oct 22 16:51 mysqltemp
```

```
# change user to mysql

sudo -i
su mysql

mysql@dc25jamint3db01:/root>

cd /opt/mysql

ln -s /usr current

$ ls -la
total 0
drwxr-xr-x 2 mysql mysql  21 Oct 29 09:02 .
drwxr-xr-x 8 root  root  100 Oct 25 20:49 ..
lrwxrwxrwx 1 mysql mysql   4 Oct 29 09:02 current -> /usr

mkdir /mysqllog/data

mkdir /mysqltemp/tmp


## change user back to root

exit

/usr/sbin/mysqld --defaults-file=/etc/my.cnf --initialize --user=mysql


## Get the temporary password from /mysqllog/data/*.* (the log file configured in /etc/my.conf)

2021-10-29T09:22:54.034162Z 1 [Note] A temporary password is generated for root@localhost: +uzoCaBR%5sr

systemctl start mysql

mysql -uroot -p (as root user)

mysql> set password=PASSWORD('***');
Query OK, 0 rows affected, 1 warning (0.00 sec)
```


### **4-4 Configure master server**

**Create a user which slave uses to login the master**

```
mysql -uroot -p

# CREATE USER 'jeffrey'@'localhost' IDENTIFIED BY 'password';

create user 'repl'@'10.181.40.83' identified by 'repl@***';

Query OK, 0 rows affected (0.01 sec)

# GRANT REPLICATION SLAVE ON *.* TO 'user'@'host'


# How to grant replication privilege to a database
grant replication slave on *.*  to  'repl'@'10.181.40.83';

mysql> show master status \G;
*************************** 1. row ***************************
             File: mysql-bin.000002
         Position: 876
     Binlog_Do_DB:
 Binlog_Ignore_DB:
Executed_Gtid_Set:
1 row in set (0.00 sec)

ERROR:
No query specified


show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000002 |      876 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

### **4-5 Configure slave server**

```
sudo mysql -uroot -p



change master to master_host='10.181.40.68',  master_user='repl', master_password='repl@***',master_log_file='mysql-bin.000002', master_log_pos=0;

Query OK, 0 rows affected, 2 warnings (0.01 sec)


mysql> start slave;
Query OK, 0 rows affected (0.01 sec)

mysql> show slave status \G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 10.181.40.68
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000002
          Read_Master_Log_Pos: 876
               Relay_Log_File: dc25jamint3db02-relay-bin.000002
                Relay_Log_Pos: 1089
        Relay_Master_Log_File: mysql-bin.000002
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes

...
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates            

```

**On slave node, there are 3 processes started.**

```
mysql> show processlist\G;
*************************** 1. row ***************************
     Id: 4
   User: root
   Host: localhost
     db: NULL
Command: Query
   Time: 0
  State: starting
   Info: show processlist
*************************** 2. row ***************************
     Id: 5
   User: system user
   Host:
     db: NULL
Command: Connect
   Time: 303
  State: Waiting for master to send event
   Info: NULL
*************************** 3. row ***************************
     Id: 6
   User: system user
   Host:
     db: NULL
Command: Connect
   Time: 1169
  State: Slave has read all relay log; waiting for more updates
   Info: GRANT REPLICATION SLAVE ON *.* TO 'repl'@'10.181.40.83'
3 rows in set (0.00 sec)

ERROR:
No query specified
```

**On master nodes, there are 2 processes started:**

```
mysql>  show processlist\G;
*************************** 1. row ***************************
     Id: 23
   User: repl
   Host: dc25jamint3db02.dc025.sf.priv:47150
     db: NULL
Command: Binlog Dump
   Time: 635
  State: Master has sent all binlog to slave; waiting for more updates
   Info: NULL
*************************** 2. row ***************************
     Id: 24
   User: root
   Host: localhost
     db: NULL
Command: Query
   Time: 0
  State: starting
   Info: show processlist
2 rows in set (0.00 sec)

ERROR:
No query specified
```

### **4-6 Configure other important parameters**

```
set global local_infile=on;
Query OK, 0 rows affected (0.00 sec)

sudo vim /etc/my.cnf

[mysqld]

local_infile=on

set global innodb_stats_auto_recalc=off;
```

```
show global variables like "innodb_stats_auto_recalc";
+--------------------------+-------+
| Variable_name            | Value |
+--------------------------+-------+
| innodb_stats_auto_recalc | OFF   |
+--------------------------+-------+
1 row in set (0.01 sec)

mysql> show global variables like "local_infile";
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| local_infile  | ON    |
+---------------+-------+
1 row in set (0.00 sec)
```

### **4-7 CREATE SCHEMA和CREATE DATABASE是等效的.**

```
create schema ct_stage default character set utf8 collate utf8_general_ci;

create schema ps_stage default character set utf8 collate utf8_general_ci;

create schema tenant_migration_transient_ct default character set utf8 collate utf8_general_ci;

create schema tenant_migration_transient_ps default character set utf8 collate utf8_general_ci;
```

```
mysql> show databases;
+-------------------------------+
| Database                      |
+-------------------------------+
| information_schema            |
| ct_stage                      |
| mysql                         |
| performance_schema            |
| ps_stage                      |
| sys                           |
| tenant_migration_transient_ct |
| tenant_migration_transient_ps |
+-------------------------------+
8 rows in set (0.00 sec)
```


