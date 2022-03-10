## **SQL操作精简版**


**数据定义语言 (DDL)**

```
show databases; # 查看所有数据库： /  show  tables; # 查看所有表： 
drop database db1; # 删除数据库   

create database db1 default character set utf8; # 创建数据库    
use database; # 选择数据库

create table t1(id int(3),name varchar(20)); # 创建表

insert into t1 values(1,'user1'); # 插入数据 

alter table t1 \...\
			 			add(age int(3)); # 添加表字段语句
			  			drop id; # 删除表字段语句  
						modify age varchar(2); # 修改表字段类型格式  
						age p1age int(3); # 修改表字段名称  	
				  
truncate table t1; # 清空表结构   /  drop table t1; # 删除表结构 
```

**数据操纵语言(DML)**

```
insert into test(id,name,address) values(102,'rose','beijing');

# 删除表中test01中的id为101的记录。
delete from test where id=101;

update test01 set address='english',name='micheal' where id=102;
```

**事物控制语言 （Transation Control Language） 有时可能需要使用 DML 进行批量数据的删除， 修改，增加。**

* **数据查询语言（Data Query Language）**

```
from test where deptid=10 or deptid=20;

# 使用in ，not in修改上面的练习
where deptid in(10,20);
where not in (10,20);

# 查询员工test,blake,clark三个人的工资
select  ... sal > all(select sal from test where name in ('test','blake','clark'))

# 查询工资大于等于1500并且小于等于2500的员工的所有信息。
where sal between 1500 and 2500;

# 查询工资小于2000和工资大于2500的所有员工信息。
not between 2000 and 2500;

# 查询员工姓名中有a和s的员工信息。
where name like '%a%' and name like '%s%';

# 排序规则: ASC：升序 ，DESC: 降序，默认是升序排序
order by sal DESC;
select * from test order by depart_id asc, age desc;

# Distinct 去重
select distinct name from test;


# 分组查询 Group By 子句 / # 查询以部门id为分组员工的平均工资
select deptid,avg(sal) as av from test group by deptid;

# 分组查询添加条件 Having 子句
group by deptid having avg(sal)>2000;

# 高级关联查询
# 查询表中各部门人员中大于部门平均工资的人
select name,sal,a.deptid ,b.av
from test a,
(select deptid,avg(ifnull(sal,0)) as av from test group by deptid) b
where a.deptid=b.deptid and a.sal>b.av
order by deptid ASC;

#  Limit 分页：Limit Offset, Size
# offset 表示 行数据索引；size 表示取多少条数据
# 从第 offset 行开始，取 size 行数据。
select * from test limit 6,10;
```

* **数据控制语言(DCL)**

数据控制语言 Data Control Language,作用是用来创建用户，给用户授权，撤销权限，删除用户。格式:

```
# 创建用户
create user username@ip identified by newPwd;
# 显示用户的权限
show grants for username@ip;
#  授权
grant select,drop,insert on test.* to 'test'@'192.168.152.166';
# 撤销权限  revoke drop on test.* to 'test'@'192.168.152.166';
#  删除用户 drop user username;   
# 使权限立即生效 flush privileges
```

* 多表操作

```
# 一对多
constraint 外键名 foreign key (被约束的字段) references 约束的表(约束的字段)

constraint fk_user_depart foreign key (depart_id) references dep(id)

select * from boy left join b2g on boy.id=b2g.bid left join girl on girl.id=b2g.gid


# 多对多
constraint fk_b2g_boy foreign key (bid) references boy(id),
constraint fk_b2g_girl foreign key (gid) references girl(id)
# 多表联查
select userinfo.name as uname, dep.name as dname from userinfo left join dep on depart_id=dep.id
```

## **MySQL 索引**

```
# 基于表结构创建  CREATE INDEX nameIndex ON student(name(50));

# 修改表结构创建   ALTER TABLE student ADD INDEX nameIndex(name(50));

# 基于表结构创建: CREATE UNIQUE INDEX idIndex ON student(id)

# 修改表结构创建 ALTER TABLE student ADD PRIMARY KEY (id):
```

**删除索引**

```
# 直接删除 DROP INDEX nameIndex ON student;
# 修改表结构删除   ALTER TABLE student DROP INDEX nameIndex;

# 主键索引（PRIMARY KEY） ALTER TABLE student DROP PRIMARY KEY;
# 查看索引 SHOW INDEX FROM student;
```

## **MySQL 日志管理**

```
$ vim /etc/my.cnf
log_error=/data/3306/data/mysql.log #这里的路径和文件名称可以随便定义

查看错误日志
mysql> select @@log_error;

查看二进制日志的配置信息
mysql> show variables like '%log_bin%';

手工清理
PURGE BINARY LOGS BEFORE now() - INTERVAL 3 day;
PURGE BINARY LOGS TO 'mysql-bin.000009';

查看慢日志
show variables like '%slow_query%';

``


