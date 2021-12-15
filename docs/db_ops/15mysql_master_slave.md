# **第十五节 MySQL 传统主从同步配置**

**1、MySQL 主从部署环境**

* Master：192.168.172.110 
* Slave：192.168.172.111
* 端口：3306
* Master, Slave 按照以下步骤安装mysql数据库

**2、MySQL yum包下载**

```
# wget http://repo.mysql.com/mysql57-community-releaseel7-10.noarch.rpm
```

**3、MySQL 软件源安转**

```
# yum -y install mysql57-community-release-el7-10.noarch.rpm
```

**4、MySQL 服务安装**

```
# yum install -y mysql-community-server mysql
```

**5、MySQL 服务启动**

```
# systemctl status mysqld.service
```

**6、MySQL 服务运行状态检查**

```
# systemctl status mysqld.service
```

**7、MySQL 修改临时密码**

* **1)获取MySQL的临时密码**

为了加强安全性，MySQL5.7为root用户随机生成了一个密码，在error log中，关于error log的位 置，如果安装的是RPM包，则默认是`/var/log/mysqld.log`。 

只有启动过一次mysql才可以查看临时密码

```
grep 'temporary password' /var/log/mysqld.log
```

* **2)登陆并修改密码**

使用默认的密码登陆

```
 mysql -uroot -p
```

修改密码

```
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'Qfedu.123com';
```

解决不符合密码复杂性要求

```
ERROR 1819 (HY000): Your password does not satisfy the current policy 
requirements 
```

修改 密码策略

```
mysql> set global validate_password_policy=0;
```

修改密码的长度

```
mysql> set global validate_password_length=1;
```

执行修改密码成功

```
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'root123';
```

**8、MySQL 授权远程主机登录**

```
mysql> GRANT ALL PRIVILEGES ON *.* TO 'slave'@'192.168.%.%' IDENTIFIED BY 
'mypassword' WITH GRANT OPTION;
mysql> FLUSH  PRIVILEGES
```

**9、MySQL 编辑配置文件**

* 1)Master 配置文件

```
# cat /etc/my.cnf
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
default-storage-engine=INNODB
symbolic-links=0
server_id=6
log_bin=/var/log/mysql/mysql-bin
```

* 2)Slave 配置文件

```
# cat /etc/my.cnf
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
default-storage-engine=INNODB
symbolic-links=0
server_id=8
log_bin=/var/log/mysql/mysql-bin
```

**10、MySQL 创建主从同步账号**

在主库创建一个专门用来复制的数据库用户，所有从库都用这个用户来连接主库，确保这个用户只 有复制的权限

```
# mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor. Commands end with ; or \g.
Your MySQL connection id is 7
Server version: 5.7.25-log MySQL Community Server (GPL)
Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.
Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
mysql> 
mysql> CREATE USER 'slave'@'192.168.%.%' IDENTIFIED BY 'Qfedu.123com';
Query OK, 0 rows affected (0.01 sec)
mysql> 
mysql> GRANT REPLICATION SLAVE ON *.* TO 'slave'@'192.168.%.%';
Query OK, 0 rows affected (0.00 sec)
mysql> 
mysql> quit
Bye
```

**11、MySQL 测试账号远程登录**

```
# mysql -uslave -p'Qfedu.123com' -P 3306 -h 
master.qfedu.com
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor. Commands end with ; or \g.
Your MySQL connection id is 20
Server version: 5.7.25-log MySQL Community Server (GPL)
Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.
Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql             |
| performance_schema |
| sys               |
+--------------------+
4 rows in set (0.00 sec)
mysql> 
mysql> QUIT
Bye
```

**12、MySQL 获取主库的日志信息并生成主库数据镜像**

```
mysql> FLUSH TABLES WITH READ LOCK; # 对主库上所有表加锁，停止修改，即
在从库复制的过程中主库不能执行UPDATA，DELETE，INSERT语句！
Query OK, 0 rows affected (0.00 sec)
mysql> 
mysql> SHOW MASTER STATUS; # 获取主库的日志信息，file 表示当
前日志文件名称，position 表示当前日志的位置
+---------------------------------+----------+--------------+------------------
+-------------------+
| File | Position | Binlog_Do_DB | Binlog_Ignore_DB | 
Executed_Gtid_Set |
+---------------------------------+----------+--------------+------------------
+-------------------+
| /var/log/mysql/mysql-bin.000002 |     4095 |             | |
|
+---------------------------------+----------+--------------+------------------
+-------------------+
1 row in set (0.00 sec)
mysql> 
```

**13、MySQL 备份主库数据**

```
# mysqldump -u root -p'Qfedu.123com' --master-data --all-databases > qfedu.com-master.sql 
mysqldump: [Warning] Using a password on the command line interface can be 
insecure.
[root@master.qfedu.com ~]# ll
total 776
-rw-r--r-- 1 root root 791962 Jun 10 19:24 qfedu.com-master.sql
```

**主库数据备份完毕后，释放主库锁，需要注意，在上锁这一段期间，无法对数据库进行写操作，比 如UPDATE，DELETE，INSERT**

```
mysql> UNLOCK TABLES; 
Query OK, 0 rows affected (0.00 sec)
```

**14、MySQL 从库还原数据**

将主库的镜像拷贝当从库中，将数据还原到从库

```
# ll
total 776
-rw-r--r-- 1 root root 791962 Jun 10 19:24 qfedu.com-master.sql

# scp qfedu.com-master.sql slave.qfedu.com:/root/
The authenticity of host 'slave.qfedu.com (192.168.172.111)' can't be 
established.
ECDSA key fingerprint is SHA256:BkN6bKO2q5zMgvremE/3rOIsCaq9eTPudgfU0lhbqGo.
ECDSA key fingerprint is MD5:75:bd:cb:be:35:f8:45:a2:ea:74:bc:aa:29:74:4d:0d.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'slave.qfedu.com,192.168.172.107' (ECDSA) to the list 
of known hosts.
root@slave.qfedu.com's password: 
qfedu.com-master.sql                                      100%  773KB  60.7MB/s 
  00:00 
```

从库数据还原

```
# pwd
/root

# ll
total 776
-rw-r--r-- 1 root root 791962 Jun 10 19:37 qfedu.com-master.sql

# mysql -uroot -p'Qfedu.123com'
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor. Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 5.7.25-log MySQL Community Server (GPL)
Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.
Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
mysql> source qfedu.com-master.sql;
Query OK, 0 rows affected (0.00 sec)
Query OK, 0 rows affected (0.00 sec)
.......
Query OK, 0 rows affected (0.00 sec)
mysql>
```

**15、MySQL 从库配置同步**

在从库上建立复制关系，即从库指定主库的日志信息和链接信息

```
mysql> CHANGE MASTER TO
   -> MASTER_HOST='master.qfedu.com',
   -> MASTER_PORT=3306,
   -> MASTER_USER='slave',
   -> MASTER_PASSWORD='Qfedu.123com',
   -> MASTER_LOG_FILE='/var/log/mysql/mysql-bin.000002',
   -> MASTER_LOG_POS=4095;
Query OK, 0 rows affected, 2 warnings (0.01 sec)
mysql>
```

**16、MySQL 从库启动复制进程**

```
mysql> START SLAVE;
Query OK, 0 rows affected (0.00 sec)
mysql> 
mysql> SHOW SLAVE STATUS\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                 Master_Host: master.qfedu.com
                 Master_User: copy
                 Master_Port: 3306
               Connect_Retry: 60
             Master_Log_File: /var/log/mysql/mysql-bin.000002
         Read_Master_Log_Pos: 4095
               Relay_Log_File: node107-relay-bin.000002
               Relay_Log_Pos: 332
       Relay_Master_Log_File: /var/log/mysql/mysql-bin.000002
             Slave_IO_Running: Yes # 观察IO进程是否为yes，如果为yes说明正常，如果长
时间处于"Connecting"状态就得检查你的从库指定的主库的链接信息是否正确
           Slave_SQL_Running: Yes # 观察SQL进程是否为yes
             Replicate_Do_DB: 
         Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
     Replicate_Wild_Do_Table: 
 Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
         Exec_Master_Log_Pos: 4095
             Relay_Log_Space: 541
             Until_Condition: None
               Until_Log_File: 
               Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
             Master_SSL_Cert: 
           Master_SSL_Cipher: 
               Master_SSL_Key: 
       Seconds_Behind_Master: 0 #该参数表示从库和主库有多少秒的延迟，再过多少秒数据
和主库保持一致，如果为0表示当前从库和主库的数据是一致的，如果该数较大的话你得考虑它的合理性。需
要注意下该参数的值。
Master_SSL_Verify_Server_Cert: No
               Last_IO_Errno: 0
               Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
 Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 106
                 Master_UUID: 74227047-8b60-11e9-8cba-000c29985293
             Master_Info_File: /var/lib/mysql/master.info
                   SQL_Delay: 0
         SQL_Remaining_Delay: NULL
     Slave_SQL_Running_State: Slave has read all relay log; waiting for more 
updates
           Master_Retry_Count: 86400
                 Master_Bind: 
     Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
           Executed_Gtid_Set: 
               Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
1 row in set (0.00 sec)
mysql> 
```

**17、MySQL 验证主从数据同步**

* 1)主库中创建 qfedu 数据库

```
# mysql -uroot -p'Qfedu.123com'
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor. Commands end with ; or \g.
Your MySQL connection id is 22
Server version: 5.7.25-log MySQL Community Server (GPL)
Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.
Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
mysql> 
mysql> CREATE DATABASE qfedu DEFAULT CHARACTER SET = utf8;
Query OK, 1 row affected (0.00 sec)
mysql> 
mysql> GRANT ALL PRIVILEGES ON qfedu.* TO 'qfedu'@'192.168.172.%' IDENTIFIED BY 
'Qfedu.123com' WITH GRANT OPTION;
Query OK, 0 rows affected, 1 warning (0.01 sec)
mysql>
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| qfedu               |
| mysql             |
| performance_schema |
| sys               |
+--------------------+
5 rows in set (0.00 sec)
mysql> QUIT
Bye
```

* **2)从库的服务器观察数据是否同步**

```
# mysql -uroot -p'Qfedu.123com'
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor. Commands end with ; or \g.
Your MySQL connection id is 12
Server version: 5.7.25-log MySQL Community Server (GPL)
Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.
Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
mysql> 
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| qfedu             |
| mysql             |
| performance_schema |
| sys               |
+--------------------+
5 rows in set (0.00 sec)
mysql> QUIT
Bye
```

* **3)测试完成后删除 qfedu 库，**

```
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| qfedu             |
| mysql             |
| performance_schema |
| sys               |
+--------------------+
5 rows in set (0.00 sec)
mysql> drop database qfedu;
Query OK, 0 rows affected (0.00 sec)
mysql> 
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
mysql>
```
