# **第三节 结构化查询语言 SQL**


## **1、数据定义语言 (DDL)**


* Data Dafinitaon Language
* 如创建表 create
* 删除表 drop
* 修改表 alter
* 清空表 truncate，彻底清空，无法找回

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

## **2、数据操纵语言(DML)**

* insert：向表中插入数据
* delete：删除表中的数据，格式：`delete from tablname [where 条件]`
* update：修改表中的数据 格式：**`update tablname set colName1=value1[,colName2=value2] [where 条件]`**
* where : 对表中的数据增加条件进行限制，起到过滤的作用。
	* 格式: `where colName` 关系运算符 `value [or|and 条件2]`
	* 关系运算符: `>，>=，<，<=`, 等于：`=`，不等于：`!=` 或` <>`

* null值操作：比较null时，不能使用=或者!= 或者<>，而是使用` is`或者`is not`，在`select`子句中，使 用关系运算符


### **2-1 实验案例**

**1)创建一个表**

```
create table test(
id int,
name varchar(20),
birth date,
address varchar(50)
);
```

**2)插入数据**

```
插入数据  100，'jack' ,'shanghai' ，下面是两种插入方式，第二种可以批量插入。只需添加多个用
逗号隔开即可

insert into test values (101,'jack',null,'shanghai');
insert into test(id,name,address) values(102,'rose','beijing');
```

**3)删除数据**

```
# 删除表test01中的所有数据。
delete from test;

# 删除表中test01中的id为101的记录。
delete from test where id=101;

# 删除表test 中 id为102和地址为上海的的数据

select * from test;
delete from test where id=102 and address='shanghai';
```

**4)修改数据**

```
#修改表 test01 中的字段 address 为中国。
update test01 set address='china';

# 修改表 test01 中的 id 为 102 的 address 为english，人名为 micheal。
update test01 set address='english',name='micheal' where id=102;

# 将生日为 null 的记录的 name 改为 'general'
update test01 set name='general' where tbirth is null;

# 将 id 为 101 的 birth 改为'2000-8-8';
update test01 set birth = '2000-8-8' where id = 101;

# 将 id 为 102 的 address 改为 null.
update test01 set address=null where id=102;
```


### **2-3 事物控制语言(TCL)**

**事物控制语言 （Transation Control Language） 有时可能需要使用 DML 进行批量数据的删除， 修改，增加。**

比如，在一个员工系统中，想删除一个人的信息。除了删除这个人的基本信息外，还 应该删除与此人有关的其他信息，如邮箱，地址等等。那么从开始执行到结束，就会构成一个事 务。对于事务，要保证事务的完整性。要么成功，要么撤回。

**事务要符合四个条件(ACID)：**

* 原子性(Atomicity)：事务要么成功，要么撤回。不可切割性。
* 一致性(Consistency)：事务开始前和结束后，要保证数据的一致性。转账前账号A和账号B的钱的总数为10000，转账后账号A和账号B的前的总数应该还是10000;
* 隔离性(Isolation)：当涉及到多用户操作同一张表时，数据库会为每一个用户开启一个事务。那么当其中一个事务正在进行时，其他事务应该处于等待状态。保证事务之间不会受影响。
* 持久性(Durability)：当一个事务被提交后，我们要保证数据库里的数据是永久改变的。即使数据库崩溃了，我们也要保证事务的完整性。


* commit 提交 rollback 撤回，回滚。
* savepoint 保存点 只有DML操作会触发一个事务。

存储引擎(ENGINE):就是指表类型.当存储引擎为innodb时，才支 持事务。默认的存储引擎为 Myisam。不支持事务。

**事务的验证：**

* 第一步：start transaction (交易) 
* 第二步：savepoint 保存点名称。
* 第三步：DML 
* 第四步：commit/rollback;


## **3、数据查询语言（Data Query Language）**

数据查询语言（Data Query Language）

**1、Select/for 基本查询语句**

```
格式：select子句  from子句     select colName[,colName.......]  from tablname;
```

**2、给列起别名**

```
格式：select 列名1 as "要起的名" [, 列名2 as "要起的名" ,... ]  from tablname;
```

**3、Where**

作用：在增删改查时，起到条件限制的作用 多条件写法：

* `in | not in` (集合元素，使用逗号分开); 注意：同一个字段有多个值的情况下使用。
* in 相当于 or， 
* `not in` 相当于 `and all | any `与集合连用，此时集合中的元素不能是固定的必须是从表中查询到的数据。
* 范围查询：`colName between val1 and val2;` 查询指定列名下的`val1`到`val2`范围中的数据 模糊查询：`like`
	* 通配符：`%` 表示`0`或`0`个以上字符。`_ `表示匹配一个字符
	* 格式: `colName like value; `


## **4、 实验实例**

### **4-1 创建表**

```
create table test (
testid int(5),
name VARCHAR(10),
job VARCHAR(9),
mgr int(4),
hiredate DATE,
sal int(7),
comm int(7),
deptid int(2)
);
```

### **4-2 添加数据**

```
insert into test (testid,name,job,mgr,hiredate,sal,comm,deptid) values
(7369 ,'SMITH','CLERK',7902,'1980-12-17',800,NULL,20),
(7499 ,'test','SALESMAN' , 7698 , '1981-2-10' ,  1600 , 300 , 30),
(7521 ,'WARD', 'SALESMAN' , 7698,  '1981-2-22' ,  1250 , 500 , 30),
(7566 ,'JONES','MANAGER' ,  7839,  '1981-4-2' ,  2975 , NULL , 20),
(7654 ,'MARTIN','SALESMAN' , 7698,  '1981-9-28',   1250 , 1400 , 30),
(7698 ,'BLAKE','MANAGER' ,  7839 , '1981-5-1' ,   2850 , NULL , 30),
(7782 ,'CLARK','MANAGER' ,  7839 , '1981-6-9' ,   2450 , NULL , 10),
(7788 ,'SCOTT','ANALYST' ,  7566,  '1987-4-19',   3000 , NULL , 20),
(7839 ,'KING','PRESIDENT' ,NULL,  '1981-11-17',  5000 , NULL , 10),
(7844 ,'TURNER','SALESMAN' , 7698,  '1981-9-8',    1500 , 0   ,  30),
(7876 ,'ADAMS','CLERK' ,   7788 , '1987-5-23',   1100,  NULL , 20),
(7900 ,'JAMES','CLERK' ,    7698 , '1981-12-3',   950 ,  NULL , 30),
(7902 ,'FORD','ANALYST' ,  7566 , '1981-12-3' ,  3000 , NULL , 20),
(7934 ,'MILLER','CLERK',     7782 , '1982-1-23',   1300 , NULL , 10),
(8002 ,'IRONMAN','MANAGER',   7839 , '1981-6-9',    1600, NULL , 10),
(8003 ,'SUPERMAN','MANAGER',   7839 , '1981-6-9',    1600 , NULL , NULL);
```

### **4-3 查询数据**

```
# 查询员工表 test 中的 员工姓名，员工职位，员工入职日期和员工所在部门。
select name,job,hiredate,deptid from test;
```


### **4-4 给列起别名**


```
# 查询员工姓名和员工职位。分别起别名，姓名和职位。
select name as "姓名" ,job as "职位" from test;
```

### **4-5 Where 子句**

```
#查询员工表中部门号为 10 和 20 的员工的编号，姓名，职位，工资
select testid,name,job,sal from test where deptid=10 or deptid =20;

# 查询员工表中部门号不是 10 和 20 的员工的所有信息。
select * from test where deptid<>10 and deptid<>20;

# 使用in ，not in修改上面的练习
select testid,name,job,sal from test where deptid in(10,20);
select * from test where deptid not in (10,20);
```

### **4-6 `All|Any`与集合连用**

```
# 查询员工test,blake,clark三个人的工资
select * from test where sal>all(select sal from test where name in
('test','blake','clark'));
```

### **4-7 范围查询**

```
# 查询工资大于等于1500并且小于等于2500的员工的所有信息。
select * from test where sal between 1500 and 2500;

# 查询工资小于2000和工资大于2500的所有员工信息。
select * from test where sal not between 2000 and 2500;
```

### **4-8 模糊查询：Like**

```
# 查询员工姓名中有a和s的员工信息。
select name,job,sal,comm,deptid from test where name like '%a%' and name like
'%s%';

select name,job,sal,comm,deptid from test where name like '%a%s%' or name like
'%s%a%';
```

### **4-9、排序查询 Order By子句**

当在查询表中数据时，记录比较多，有可能需要进行排序，此时可以使用 order by子句。

```
select...from tablname [where 子句][order by 子句]
```
注意: 可以通过一个或多个字段排序。

格式

```
order by colName [ ASC | DESC ][ , colName1....[ ASC | DESC ] ];
```

**排序规则: ASC：升序 ，DESC: 降序，默认是升序排序**

```
select * from test order by sal DESC;
```

### **4-10、Distinct 去重**

有的时候我们需要对重复的记录进行去重操作，比如，查询表中有哪些职位，此时，一种职位只需 要显示一条记录就够。

位置：必须写在select关键字后

```
select distinct name from test;
```


### **4-11、分组查询 Group By 子句**

分组查询与分组函数(聚合函数)：有的时候，我们可能需要查询表中的记录总数，或者查询表中每 个部门的总工资,平均工资，总人数。这种情况需要对表中的数据进行分组统计。需要group by子句。

位置：

```
select...from name [where 条件] [group by子句] [order by子句]
```

用法:

```
group by Field1[,Field2]
```

注意：在分组查询时，select子句中的字段，除了聚合函数外，只能写分组字段。

```
# 查询以部门id为分组员工的平均工资
select deptid,avg(sal) as av from test group by deptid;
```

**聚合函数:**

1. count(Filed) 统计指定字段的记录数。
2. sum(Filed) 统计指定字段的和。
3. avg(Filed) 统计指定字段的平均值
4. max(Filed) 返回指定字段中的最大值。
5. min(Filed) 返回指定字段中的最小值。

聚合函数会忽略null值。因此有时候需要使用函数：`ifnull(field,value)`（如果field字段对应的值不 是null,就使用field的值，如果是`null`,就使用value.）

注意：多字段分组时（多表联合查询时不加任何条件），最多分组的数目为 Filed1Field2[Filed3....]


### **4-12 分组查询添加条件 Having 子句**

在分组查询时，有的时候可能需要再次使用条件进行过滤，这个时候不能 where子句，应该使用 having子句。having子句后可以使用聚合函数。

位置：位于`group by`子句后

```
# 查询以部门id为分组员工的平均工资并且大于2000的部门
select deptid,avg(sal) as av from test group by deptid having avg(sal)>2000;
```

### **4-13 基本查询总结**

基本的查询语句包含的子句有：select子句，from子句，where子句，group by子句，having子 句，order by子句

完整的查询语句：

```
select..from..[where..][group by..][having..][order by..]
```

执行顺序：**先执行from子句，再执行where子句，然后group by子句，再次having子句，之后 select子句，最后order by子句**


### **4-14 高级关联查询**

有的时候，查询的数据一个简单的查询语句满足不了，并且使用的数据在表中不能直观体现出来。而是需要预先经过一次查询才会有所体现。先执行的子查询。子查询嵌入到的查询语句称之为父查 询。

子查询返回的数据特点：

1. 可能是单行单列的数据。
2. 可能是多行单列的数据
3. 可能是单行多列的数据
4. 可能是多行多列的数据
 
子查询可以在where、from、having、select字句中，在select子句中时相当于外连接的另外一种 写法

```
# 查询表中各部门人员中大于部门平均工资的人
select name,sal,a.deptid ,b.av
from test a,
(select deptid,avg(ifnull(sal,0)) as av from test group by deptid) b
where a.deptid=b.deptid and a.sal>b.av
order by deptid ASC;
```

## **5、数据控制语言(DCL)**

数据控制语言 Data Control Language,作用是用来创建用户，给用户授权，撤销权限，删除用户。格式:

### **5-1 创建用户**

```
create user username@ip identified by newPwd;
create user 'test'@'192.168.152.166' identified by 'test.123com';
```

### **5-2 显示用户的权限**

```
show grants for username@ip;
```

### **5-3 授权**

```
grant 权限1，权限2... on 数据库名.* to username@ip;
grant select,drop,insert on test.* to 'test'@'192.168.152.166';
```

* DML 权限：insert,delete,update
* DQL 权限：select
* DDL 权限：create,alter,drop

### **5-4 撤销权限**

```
revoke 权限1，权限2..on 数据库名.* from username@ip;
revoke drop on test.* to 'test'@'192.168.152.166';
```

### **5-5 删除用户**

```
drop user username;
drop user test;
```

### **5-6 使权限立即生效**

```
flush privileges
```


