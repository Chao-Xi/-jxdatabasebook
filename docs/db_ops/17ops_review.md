# **Mysql OPS Review**

## **1 查询语言 SQL**

**数据定义语言 (DDL)**

```
show databases; # 查看所有数据库：
show tables; # 查看所有表：    

drop database db1; # 删除数据库    
create database db1 default character set utf8; # 创建数据库    

use database; # 选择数据库
create table t1(id int(3),name varchar(20)); # 创建表
desc t1 # 查看表

insert into t1 values(1,'user1'); # 插入数据
alter table t1 add(age int(3)); # 添加表字段语句  
alter table t1 drop id; # 删除表字段语句  
alter table t1 modify age varchar(2); # 修改表字段类型格式  
alter table t1 change age p1age int(3); # 修改表字段名称  
alter table t1 rname per; # 修改表名
truncate table t1; # 清空表结构  
drop table t1; # 删除表结构  
```

### **数据操纵语言(DML)**

```
insert into test(id,name,address) values(102,'rose','beijing');

# 删除表test01中的所有数据。
delete from test;

# 删除表中test01中的id为101的记录。
delete from test where id=101;

# 修改表 test01 中的 id 为 102 的 address 为english，人名为 micheal。
update test01 set address='english',name='micheal' where id=102;


# 将 id 为 102 的 address 改为 null.
update test01 set address=null where id=102;
```

### **事物控制语言(TCL)**

事物控制语言 （Transation Control Language） 有时可能需要使用 DML 进行批量数据的删除， 修改，增加。

### **数据查询语言（Data Query Language）**

```
#查询员工表中部门号为 10 和 20 的员工的编号，姓名，职位，工资
select testid,name,job,sal from test where deptid=10 or deptid =20;

# 使用in ，not in修改上面的练习
select testid,name,job,sal from test where deptid in(10,20);
select * from test where deptid not in (10,20);

# 查询员工test,blake,clark三个人的工资
select * from test where sal>all(select sal from test where name in
('test','blake','clark'));


# 查询工资大于等于1500并且小于等于2500的员工的所有信息。
select * from test where sal between 1500 and 2500;

# 查询工资小于2000和工资大于2500的所有员工信息。
select * from test where sal not between 2000 and 2500;

# 查询员工姓名中有a和s的员工信息。
select name,job,sal,comm,deptid from test where name like '%a%' and name like
'%s%';


# 排序规则: ASC：升序 ，DESC: 降序，默认是升序排序
select * from test order by sal DESC;

# Distinct 去重
select distinct name from test;

# 分组查询 Group By 子句
# 查询以部门id为分组员工的平均工资
select deptid,avg(sal) as av from test group by deptid;

# 分组查询添加条件 Having 子句
# 查询以部门id为分组员工的平均工资并且大于2000的部门
select deptid,avg(sal) as av from test group by deptid having avg(sal)>2000;

# 高级关联查询
# 查询表中各部门人员中大于部门平均工资的人
select name,sal,a.deptid ,b.av
from test a,
(select deptid,avg(ifnull(sal,0)) as av from test group by deptid) b
where a.deptid=b.deptid and a.sal>b.av
order by deptid ASC;
```

### **数据控制语言(DCL)**

数据控制语言 Data Control Language,作用是用来创建用户，给用户授权，撤销权限，删除用户。格式:

```
# 创建用户
create user username@ip identified by newPwd;
create user 'test'@'192.168.152.166' identified by 'test.123com';

# 显示用户的权限
show grants for username@ip;

#  授权
grant select,drop,insert on test.* to 'test'@'192.168.152.166';

# 撤销权限
revoke drop on test.* to 'test'@'192.168.152.166';

#  删除用户
drop user username;
drop user test;

# 使权限立即生效
flush privileges
```

## **2 MySQL 操作实例**

```
mysql> create table slt(
num int auto_increment primary key,
name char(10),
job char(10),
age int,
salary int,
descrip char(128)not null default ''
)charset=utf8;
```

* `auto_increment` : 自增 `1 primary key` : 主键索引，加快查询速度, 列的值不能重复 `NOT NULL` 标识该字段不能为空 `DEFAULT` 为该字段设置默认值

```
select distinct name, age from slt where job='teacher'

# 查看岗位是teacher且年龄大于30岁的员工姓名、年龄
mysql> select distinct name, age from slt where job='teacher' and age>30;

# 查看岗位是 teacher且薪资在1000-9000范围内的员工姓名、年龄、薪资
mysql> select distinct name, age, salary from slt where job='teacher' and
(salary>1000 and salary<9000)

# 查看岗位描述不为NULL的员工信息
mysql> select * from slt where descrip!=NULL

# 查看岗位是teacher且名字是'jin'开始的员工姓名、年薪
mysql> select * from slt where job='teacher' and name like 'jin%' ;

```

### 修改表

```
# ALTER TABLE 旧表名 RENAME 新表名;
alter table t3 rename t4;

ALTER TABLE 表名
ADD 字段名 列类型 [可选的参数],
ADD 字段名 列类型 [可选的参数];

alter table t88 add name varchar(32) not null default '';


<!--添加到开头语法
ALTER TABLE 表名
ADD 字段名 列类型 [可选的参数] FIRST-->
alter table t88 add name3 varchar(32) not null default '' first;

<!--添加到指定字段后面
ALTER TABLE 表名
ADD 字段名 列类型 [可选的参数] AFTER 字段名;-->
alter table t88 add name4 varchar(32) not null default '' after name;
 
# 删除字段
# ALTER TABLE 表名  DROP 字段名;
alter table t88 drop name4;

# 修改字段
# ALTER TABLE 表名 MODIFY 字段名 数据类型 [完整性约束条件…];
alter table t88 modify name char(20);
```

```
删除表
drop table t9;

查询表
show tables;

复制表结构
create table t8 like t1;

delete from t1 where id>=1 and id<10;
truncate t1;
```

* delete 之后，插入数据从上一次主键自增加1开始， truncate则是从1开始
* **delete 删除， 是一行一行的删除， truncate：全选删除 truncate删除的速度是高于delete的**

### 修改数据

```
# update 表名 set 列名1=新值1,列名2=新值2 where 条件;
update t1 set name='qfx' where id=1;

# 分组：Group By
注意：出现 `sql_mode=only_full_group_by` 错误 需修改 配置文件`/etc/my.cnf [mysqld] ` 段添加 sql模式，重启 mysqld

sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_
ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION

select depart_id,name, max(age) from test group by depart_id;
select depart_id,count(depart_id) from test group by depart_id;
select depart_id, avg(age) from test group by depart_id;
select depart_id, avg(age) from test group by depart_id having avg(age)>35;

select * from test order by age asc, depart_id desc;

select * from test order by depart_id asc, age desc;

#  Limit 分页：Limit Offset, Size
# offset 表示 行数据索引；size 表示取多少条数据
# 从第 offset 行开始，取 size 行数据。

# 从第6行开始取10行 imit 6,10
select * from test limit 6,10;
```

### 多表操作

```
# 一对多
constraint 外键名 foreign key (被约束的字段) references 约束的表(约束的字段)
create table dep(
id int auto_increment primary key,
name varchar(32) not null default ''
)charset=utf8;


create table userinfo (
id int auto_increment primary key,
name varchar(32) not null default '',
depart_id int not null default 1,
constraint fk_user_depart foreign key (depart_id) references dep(id)
)charset utf8;

select * from boy left join b2g on boy.id=b2g.bid left join girl on girl.id=b2g.gid

# 多对多
create table b2g(
id int auto_increment primary key,
bid int not null default 1,
gid int not null default 0,
constraint fk_b2g_boy foreign key (bid) references boy(id),
constraint fk_b2g_girl foreign key (gid) references girl(id)
)charset utf8;

# 多表联查
select userinfo.name as uname, dep.name as dname from userinfo left join dep on depart_id=dep.id
```

## **3 MySQL 数据类型约束**

*  非空约束:  就是限制数据库中某个值是否可以为空，null字段值可以为空，not null字段值不能为空

```
create table testnull(id int, username varchar(20) not null);  # 创建testnull 设置 username 字段为非空约束 notnull
```

* 唯一约束： 字段添加唯一约束之后，该字段的值不能重复，也就是说在一列当中不能出现一样的值

```
# 添加唯一约束
ALTER TABLE tbl_name ADD [CONSTRAINT[symbol]]
UNIQUE [INDEX|KEY] [index_name] [index_type] (index_col_name)

# 删除唯一约束
ALTERT TABLE tbl_name DROP {INDEX|KEY} index_name

alter table testnull add unique(id);   # 设置 id 字段添加一个唯一约束 unique
```
 
* 主键约束
	*  主键保证记录的唯一性，主键自动为NOT NULL
	*  每张数据表只能存在一个主键 NOT NULL + UNIQUE KEY 一个UNIQUE KEY
	*  一个NOT NULL的时候，那么它被当做PRIMARY KEY主键
	*  当一张表里没有一个主键的时候，第一个出现的非空且为唯一的列被视为有主键。

```
# 添加主键约束
ALTER TABLE tbl_name ADD [CONSTRAINT[sysbol]]
PRIMARY KEY [index_type] (index_col_name)

# 删除主键约束
ALTER TABLE tbl_name DROP PRIMARY KEY
```

* 自增长

自增长 `AUTO_INCREMENT` 自动编号，且必须与主键组合使用 默认情况下，起始值为1，每次的 增量为1。当插入记录时，如果为AUTO_INCREMENT数据列明确指定了一个数值，则会出现两种 情况：

如果插入的值与已有的编号重复，则会出现出错信息，因为AUTO_INCREMENT数据列的值必须是 唯一的；

```
ALTER TABLE user CHANGE id id INT NOT NULL AUTO_INCREMENT;
 

#  删除自增长
ALTER TABLE user CHANGE id id INT NOT NULL;

默认约束
添加/删除默认约束
ALTER TABLE tbl_name ALTER [COLUMN] col_name {SET DEFAULT literal | DROP DEFAULT}
```

## **MySQL 索引**

* 普通索引（INDEX）：索引列值可重复
	* 普通索引常用于过滤数据。例如，以商品种类作为索引，检索种类为“手机”的商品。
* **唯一索引（UNIQUE）：索引列值必须唯一，可以为NULL**
	* 唯一索引主要用于标识一列数据不允许重复的特性，相比主键索引不常用于检索的场景。
* **主键索引（PRIMARY KEY）：索引列值必须唯一，不能为NULL，一个表只能有一个主键索引**
	* 主键索引是行的唯一标识，因而其主要用途是检索特定数据。 
* **全文索引（FULL TEXT）：给每个字段创建索引**
	 * 全文索引效率低，常用于文本中内容的检索。

```
# 普通索引（INDEX）

# 在创建表时指定
mysql> CREATE TABLE student (
   id INT NOT NULL,
   name VARCHAR(100) NOT NULL,
   birthday DATE,
   sex CHAR(1) NOT NULL,
    INDEX nameIndex (name(50))
);

# 基于表结构创建
mysql> CREATE INDEX nameIndex ON student(name(50));

# 修改表结构创建
mysql> ALTER TABLE student ADD INDEX nameIndex(name(50));
```

* 在创建表时指定： `INDEX nameIndex (name(50))`
* 基于表结构创建： `CREATE INDEX nameIndex ON student(name(50));`
* 修改表结构创建: `ALTER TABLE student ADD INDEX nameIndex(name(50));`

```
基于表结构创建: CREATE UNIQUE INDEX idIndex ON student(id)
```

* **主键索引（PRIMARY KEY）: 主键索引不能使用基于表结构创建的方式创建**。

```
# 修改表结构创建

ALTER TABLE student ADD PRIMARY KEY (id):
```

### 删除索引

```
# 普通索引（INDEX）

# 直接删除
mysql> DROP INDEX nameIndex ON student;

# 修改表结构删除
mysql> ALTER TABLE student DROP INDEX nameIndex;

# 唯一索引（UNIQUE）
# 直接删除
mysql> DROP INDEX idIndex ON student;

# 修改表结构删除
mysql> ALTER TABLE student DROP INDEX idIndex;

# 主键索引（PRIMARY KEY）
mysql> ALTER TABLE student DROP PRIMARY KEY;

# 查看索引
mysql> SHOW INDEX FROM student;
```

##  **MySQL 的用户管理和权限管理**

**DCL（数据库控制语言）**

* GRANT 用户授权，为用户赋予访问权限
* REVOKE 取消授权，撤回授权权限

**MySQL 权限表**

*  mysql.user 
	* 用户字段：Host、User、Password
	* 权限字段：`_Priv`结尾的字段
	* 安全字段：ssl x509字段
	* 资源控制字段：`max_`开头的字段

* mysql.db
	* 用户字段：Host、User、Password
	* 权限字段：剩下的_Priv结尾的字段


* 授权级别排列
	* **mysql.user #全局授权**
	* **mysql.db #数据库级别授权**

* 用户和 IP格式
	* 用户名@IP地址 用户只能在改IP下才能访问
	* 用户名@192.168.1.% 用户只能在改IP段下才能访问(通配符%表示任意)
	* 用户名@%.qfedu.com
	* 用户名@% 用户可以再任意IP下访问(默认IP地址为%)

### **MySQL 用户管理**

```
创建实例

CREATE USER 'qfedu'@'localhost' IDENTIFIED BY '123456';
CREATE USER 'qfedu'@'192.168.1.101' IDENDIFIED BY '123456';
CREATE USER 'qfedu'@'192.168.1.%' IDENDIFIED BY '123456';
CREATE USER 'qfedu'@'%' IDENTIFIED BY '123456';
GRANT ALL ON *.* TO 'user3'@’localhost’ IDENTIFIED BY ‘123456’;
# *.*所有数据库

# DROP USER 删除
DROP USER 'user1'@’localhost’;

# DELETE 语句删除
DELETE FROM mysql.user WHERE user=’user2’ AND host=’localhost’;

# 修改用户
RENAME USER 'old_user'@‘localhost’ TO 'new_user'@'localhost';

# 修改密码
# 注意：修改完密码必须刷新权限

FLUSH PRIVILEGES;

# root 用户修改自己密码
方法一：
mysqladmin -uroot -p123 password 'new_password'  # 123为旧密码

方法二： 
alter user 'root'@'localhost' identified by 'new_pssword';

方法三：
SET PASSWORD=password(‘new_password’);

# root 修改其他用户密码
方法一：
alter user 'qfedu'@'localhost' identified by 'Qfedu.1234com';

方法三：
GRANT SELECT ON *.* TO 用户名@’ip地址’ IDENTIFIED BY ‘yuan’;

# 普通用户修改自己密码
SET password=password(‘new_password’);
```

### 找回 root 密码

**在[mysqld]下面加上 `skip-grant-tables`**

```
# vim /etc/my.cnf
[mysqld]
···

#设置免密登录
skip-grant-tables
```

设置密码

```
mysql> update user set authentication_string=password('密码') where user='root';
```

### 密码复杂度

MySQL 默认启用了密码复杂度设置，插件名字叫做 `validate_password`

```
mysql> INSTALL PLUGIN validate_password SONAME 'validate_password.so';
```

修改配置

```
# vim /etc/my.cnf
[mysqld]
plugin-load=validate_password.so
validate_password_policy=0
validate-password=FORCE_PLUS_PERMANENT
```

**查看密码策略**

```
mysql> select @@validate_password_policy; # 查看密码复杂性策略

mysql> select @@validate_password_length; # 查看密码复杂性要求密码最低长度大小
```

### MySQL 权限管理

```
查看权限
SHOW GRANTS FOR '用户'@'IP地址';
SHOW GRANTS FOR 'root'@'localhost';

授权及设置密码
GRANT 权限 [,权限...[,权限]] NO 数据库.数据表 TO '用户'@'IP地址';

GRANT 权限 [,权限...[,权限]] NO 数据库.数据表 TO '用户'@'IP地址' IDENTIFIED BY '密码';

mysql> GRANT ALL ON *.* TO admin1@'%' IDENTIFIED BY '(Qfedu.123com)';

mysql> GRANT ALL ON *.* TO admin2@'%' IDENTIFIED BY '(Qfedu.123com)' WITH GRANT
OPTION;

mysql> GRANT ALL ON qfedu.* TO admin3@'%' IDENTIFIED BY '(Qfedu.123com)';
mysql> GRANT ALL ON qfedu.* TO admin3@'192.168.122.220' IDENTIFIED BY
'(Qfedu.123com)';

mysql> GRANT ALL ON qfedu.user TO admin4@'%' IDENTIFIED BY '(Qfedu.123com)';
mysql> GRANT SELECT(col1),INSERT(col2,col3) ON qfedu.user TO admin5@'%'
IDENTIFIED BY '(Qfedu.123com)';
```

## **10 MySQL 日志管理**

### **MySQL错误日志**

```
$ vim /etc/my.cnf
log_error=/data/3306/data/mysql.log #这里的路径和文件名称可以随便定义

systemctl restart mysqld
```

**查看错误日志**

```
mysql> select @@log_error;
+---------------------------+
| @@log_error |
+---------------------------+
| /data/3306/data/mysql.log |
+---------------------------+
1 row in set (0.00 sec)
```

### **MySQL 二进制日志**

* 数据恢复必备的日志。
* 主从复制依赖的日志。

```
# vim /etc/my.cnf
server_id=6
log_bin=/data/3306/binlog/mysql-bin
```

配置说明

* mysql-bin 是在配置文件配置的前缀
* 000001 MySQL每次重启，重新生成新的

### **二进制日志内容**

**DML 语句有三种模式：SBR、RBR、MBR**

```
mysql> select @@binlog_format;
+-----------------+
| @@binlog_format |
+-----------------+
| ROW |
+-----------------+
1 row in set (0.00 sec)
```

* `statement---->SBR`：做什么记录什么，即SQL语句
* `row---------->RBR`：记录数据行的变化（默认模式，推荐）
* `mixed-------->MBR`：自动判断记录模式

**二进制日志工作模式**

```
# vim /etc/my.cnf
[mysqld]
binlog_format='ROW'
```

**查看二进制日志工作模式**

```
show variables like "binlog%";
```

二进制日志三种模式的区别

**ROW: 基于行的复制**

* 优点：所有的语句都可以复制，不记录执行的sql语句的上下文相关的信息，仅需要记录那一条记录被 修改成什么了
* 缺点：binlog 大了很多，复杂的回滚时 binlog 中会包含大量的数据

**Statement: 基于sql语句的复制**

* 优点：不需要记录每一行的变化，减少了binlog日志量，节约了IO，提高性能

**mixed模式 ：row 与 statement 结合**


实际上就是前两种模式的结合，在mixed模式下，mysql会根据执行的每一条具体的sql语句来区分对待 记录的日志形式，也就是在statement和row之间选一种。

**查看二进制日志的配置信息**

```
mysql> show variables like '%log_bin%';
+---------------------------------+--------------------------------+
| Variable_name | Value |
+---------------------------------+--------------------------------+
| log_bin | ON |
| log_bin_basename | /var/log/mysql/mysql-bin |
| log_bin_index | /var/log/mysql/mysql-bin.index |
| log_bin_trust_function_creators | OFF |
| log_bin_use_v1_row_events | OFF |
| sql_log_bin | ON |
+---------------------------------+--------------------------------+
6 rows in set (0.00 sec)
```

* `log_bin` 开启二进制日志的开关
* `log_bin_basename` 位置
* `sql_log_bin` 临时开启或关闭二进制日志的小开关

打印出当前MySQL的所有二进制日志，并且显示最后使用到的 position

```
mysql> show binary logs;
mysql> show master status;（常用）
```

**二进制日志内容的查看和截取**

```
# mysqlbinlog /data/3306/binlog/mysql-bin.000001

# mysqlbinlog --base64-output=decode-rows -vvv
/data/3306/binlog/mysql-bin.000001

# mysqlbinlog --start-position=xxx --stop-position=xxx
/data/3306/binlog/mysql-bin.000001 >/data/bin.sql
```

**数据恢复实例**

```
mysql> create database binlog charset utf8mb4;

mysql> use binlog;

mysql> show binlog events in 'mysql-bin.000001';
起点：
| mysql-bin.000001 | 381 | Query | 6 | 497 | create
database binlog charset utf8mb4 |
终点：
| mysql-bin.000001 | 1575 | Query | 6 | 1673 | drop
database binlog

# 截取日志
# mysqlbinlog --start-position=381 --stop-position=1575
/data/3306/binlog/mysql-bin.000003>/data/bin.sql

# 恢复日志
mysql> set sql_log_bin=0; # 临时关闭当前会话的binlog记录
mysql> source /data/bin.sql;
mysql> set sql_log_bin=1; # 打开当前会话的binlog
```

### **二进制日志其他操作**

```
# 查看自动清理周期
mysql> show variables like '%expire%';
+--------------------------------+-------+
| Variable_name | Value |
+--------------------------------+-------+
| disconnect_on_expired_password | ON |
| expire_logs_days | 0 |
+--------------------------------+-------+
2 rows in set (0.00 sec)

# 零时设置自动清理周期
mysql> set global expire_logs_days=8;

mysql> show variables like '%expire%';
+--------------------------------+-------+
| Variable_name | Value |
+--------------------------------+-------+
| disconnect_on_expired_password | ON |
| expire_logs_days | 8 |
+--------------------------------+-------+
2 rows in set (0.00 sec)
```

永久生效

```
# vim /etc/my.cnf
[mysqld]
expire_logs_days=15;
```

**手工清理**

```
PURGE BINARY LOGS BEFORE now() - INTERVAL 3 day;
PURGE BINARY LOGS TO 'mysql-bin.000009';
```

* 注意: 不要手工 rm binlog文件
* 主从关系中，主库执行此操作 reset master; ，主从环境必崩

**二进制日志的滚动**

```
mysql> flush logs;
mysql> select @@max_binlog_size;
```

### **慢日志简介**

* 记录运行较慢的语句记录slowlog中。
* 功能是辅助优化的工具日志。、
* 应激性的慢可以通过show processlist进行监控
* 一段时间的慢可以进行slow记录、统计

**查看慢日志**

```
mysql> show variables like '%slow_query%';
+---------------------+-----------------------------------+
| Variable_name | Value |
+---------------------+-----------------------------------+
| slow_query_log | OFF |
| slow_query_log_file | /var/lib/mysql/localhost-slow.log |
+---------------------+-----------------------------------+
2 rows in set (0.00 sec)
```

**查看阈值**

```
mysql> select @@long_query_time;

mysql> SHOW GLOBAL VARIABLES LIKE 'long_query_time%';

mysql> SHOW VARIABLES LIKE 'long_query_time%';

mysql> show variables like '%log_queries_not_using_indexes%';
```

**配置慢日志**

```
# 零时设置开启
SET GLOBAL slow_query_log = 1; #默认未开启，开启会影响性能，mysql重启会失效

#设置阈值：
SET GLOBAL long_query_time=3;
```

**永久生效**

```
# vim /etc/my.cnf
[mysqld]
slow_query_log=1
slow_query_log_file=/data/3306/data/qfedu-slow.log
long_query_time=0.1 默认配置10秒钟
log_queries_not_using_indexes=1
```

**慢日志分析工具**

```
# mysqldumpslow -s r -t 10 /data/3306/data/qfedu-slow.log
# 得到返回记录集最多的10个SQL

# mysqldumpslow -s c -t 10 /data/3306/data/qfedu-slow.log
# 得到访问次数最多的10个SQL

# mysqldumpslow -s t -t 10 -g "LEFT JOIN"
/data/3306/data/qfedu-slow.log # 得到按照时间排序的前10条里面含有左连接的查询语句

# mysqldumpslow -s r -t 10 /data/3306/data/qfedu-slow.log |
more # 结合| more使用，防止爆屏情况
```

## **11 MySQL备份概述**

**备份必须重视的内容**

**备份内容 `databases Binlog my.conf`**

* 数据的一致性
* 服务的可用性

### **MySQL 备份类型**

* 物理备份: 对数据库操作系统的物理文件（如数据文件、日志文件等）的备份。
* 热备(hot backup):  在线备份，数据库处于运行状态，**这种备份方法依赖于数据库的日志文件**
	* 对应用基本无影响(应用程序读写不会阻塞,但是性能还是会有下降,所以尽量不要在主上做备份,在 从库上做)
* 冷备(cold backup)
	* 备份数据文件,需要停机，是在关闭数据库的时候进行的 **备份 datadir 目录下的所有文件**
* 温备(warm backup)

* 逻辑备份

### **MySQL 备份工具**

* ibbackup
*  xtrabackup
*  mysqldump
*  mysqlbackup
	*  **innodb 引擎的表mysqlbackup可以进行热备**
	*  **非innodb表mysqlbackup就只能温备**

### **MySQL 备份策略**

* **完全备份**: 每次对数据进行完整的备份，即对整个数据库的备份、数据库结构和文件结构的备份，保存的是备份完 成时刻的数据库，是差异备份与增量备份的基础。
* **差异备份**: **备份那些自从上次完全备份之后被修改过的所有文件，备份的时间起点是从上次完整备份起，备份数据量会越来越大**。
	* 恢复数据时，只需恢复上次的完全备份与最近的一次差异备份。
*  **增量备份**: 只有那些在上次完全备份或者增量备份后被修改的文件才会被备份。
	* 以上次完整备份或上次的增量备份 的时间为时间点，仅备份这之间的数据变化，因而备份的数据量小，占用空间小，备份速度快。

## **MySQL 主从复制原理介绍**

### **MySQL 主从复制过程**

* 开启binlog日志，通过把主库的 binlog 传送到从库，从新解析应用到从库。
* **复制需要3个线程（dump、io、sql）完成**。
* 复制是异步的过程。主从复制是异步的逻辑的SQL语句级的复制。

### **MySQL 主从复制前提**

* 主服务器一定要打开二进制日志
* 必须两台服务器（或者是多个实例）
* 从服务器需要一次数据初始化
* 如果主从服务器都是新搭建的话，可以不做初始化
* 如果主服务器已经运行了很长时间了，可以通过备份将主库数据恢复到从库。
* 主库必须要有对从库复制请求的用户。
* 从库需要有relay-log设置，存放从主库传送过来的二进制日志 show variables like '%relay%';
* 在第一次的时候，从库需要change master to 去连接主库。
* change master信息需要存放到 master.info 中 show variables like '%master_info%';
* 从库怎么知道，主库发生了新的变化通过relay-log.info记录的已经应用过的relay-log信息。
* 在复制过程中涉及到的线程
	* 从库会开启一个IO thread(线程)，负责连接主库，请求binlog，接收binlog并写入relaylog。
	* 从库会开启一个SQL thread(线程)，负责执行`relay-log`中的事件。
	* 主库会开启一个dump thrad(线程)，负责响应从`IO thread`的请求。

### **MySQL 主从复制实现**

* 通过二进制日志
* 至少两台（主、从）
* 主服务器的二进制日志“拿”到从服务器上再运行一遍。
* 通过网络连接两台机器，一般都会出现延迟的状态。也可以说是异步的。
* 从库通过手工执行change master to语句连接主库，提供了连接的用户一切条件 user、 password、port、ip
* 并且让从库知道，二进制日志的起点位置（file名 position号）
* 启动从库同步服务 start slave
* 从库的IO和主库的dump线程建立连接
* 从库根据`change master to` 语句提供的`file`名和`position`号，IO线程向主库发起binlog的请求
* 主库`dump`线程根据从库的请求，将本地binlog以events的方式发给从库IO线程
* 从库IO线程接收binlog evnets，并存放到本地`relay-log`中，传送过来的信息，会记录到 `master.info`中。
* **从库SQL线程应用`relay-log`，并且把应用过的记录到`relay-log.info`,默认情况下，已经应用过的 relay会自动被清理purge**。
