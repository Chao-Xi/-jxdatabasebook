# **第四节 MySQL 操作实例(上)**

## **1、创建表**


### **1-1 语法**

```
create table 表名(
 字段名 列类型 [可选的参数], # 记住加逗号
 字段名 列类型 [可选的参数], # 记住加逗号
 字段名 列类型 [可选的参数]  # 最后一行不加逗号
)charset=utf8; # 后面加分号
```

### **1-2 列约束**

* `auto_increment` : 自增 1 primary key : 主键索引，加快查询速度, 列的值不能重复 `NOT NULL` 标识该字段不能为空 `DEFAULT` 为该字段设置默认值

### **1-3 实例**

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

## **2、查看表结构**

**`desc slt;`**

```
mysql> desc slt;
+---------+-----------+------+-----+---------+----------------+
| Field   | Type     | Null | Key | Default | Extra         |
+---------+-----------+------+-----+---------+----------------+
| num     | int(11)   | NO   | PRI | NULL   | auto_increment |
| name   | char(10) | YES |     | NULL   |               |
| job     | char(10) | YES |     | NULL   |               |
| age     | int(11)   | YES |     | NULL   |               |
| salary | int(11)   | YES |     | NULL   |               |
| descrip | char(128) | NO   |     |   |               |
+---------+-----------+------+-----+---------+----------------+
6 rows in set (0.02 sec)
```

此时表数据为空

```
mysql> select * from slt;
qfeduty set (0.00 sec)
```

## **3、插入数据**

### **3-1 语法**

```
insert into 表名 (列1, 列2) values (值1,'值2');
```

### **3-2 实例**

```
mysql> insert into slt(name, job, age, salary,descrip) values
('Tom','teacher',30,20000,'level2'),
('frank','teacher',31,21000,'level2'),
('jack','teacher',32,22000,'level2'),
('jhon','asistant',23,8000,'level3'),
('hugo','manager',45,30000,'level4'),
('jinhisan','teacher',26,9000,'level1')
;
Query OK, 6 row affected (0.01 sec)
```


## **4 数据查询**

### **4-1 语法**

```
select 列1, 列2 from 表名; （*代表查询所有的列）
```

查看岗位是 teacher 的员工姓名、年龄


```
mysql> select distinct name, age from slt where job='teacher
```

查看岗位是teacher且年龄大于30岁的员工姓名、年龄

```
mysql> select distinct name, age from slt where job='teacher' and age>30;
```

查看岗位是 teacher且薪资在1000-9000范围内的员工姓名、年龄、薪资


```
mysql> select distinct name, age, salary from slt where job='teacher' and
(salary>1000 and salary<9000)
```

查看岗位描述不为NULL的员工信息

```
mysql> select * from slt where descrip!=NULL
```

查看岗位是teacher且薪资是10000或9000或30000的员工姓名、年龄、薪资


```
mysql> select distinct name, age, salary from slt where job='teacher' and
(salary=10000 or salary=9000 or salary=30000);
```

查看岗位是teacher且薪资不是10000或9000或30000的员工姓名、年龄、薪资


```
mysql> select distinct name, age, salary from slt where job='teacher' and
(salary not in (10000,9000,30000));
```

查看岗位是teacher且名字是'jin'开始的员工姓名、年薪


```
mysql> select * from slt where job='teacher' and name like 'jin%' ;
```

### **4-2 实例**

```
例子1：
创建表
mysql> create table t1(
   id int,
   name char(5)
)charset=utf8;
Query OK, 0 rows affected (0.72 sec)   # 如果回显是queryok，代表创建成功


插入数据：
mysql> insert into t1 (id, name) values (1, 'qfedu');
Query OK, 1 row affected (0.00 sec)

mysql> select * from t1;
+------+-------+
| id   | name |
+------+-------+
|    1 | qfedu |
+------+-------+
1 row in set (0.00 sec)
    

例子2:
mysql> create table t2(
   id int auto_increment primary key,
   name char(10)
)charset=utf8;

mysql> insert into t2 (name) values ('qfedu1');



例子3: (推荐)
mysql> create table t3(
   id  int unsigned auto_increment primary key,
   name char(10) not null default 'xxx',
   age int not null default 0
)charset=utf8;
                    

mysql> insert into t3 (age) values (10);
Query OK, 1 row affected (0.00 sec)


mysql> select * from t3;
+----+------+-----+
| id | name | age |
+----+------+-----+
|  1 | xxx |  10 |
+----+------+-----+
1 row in set (0.00 sec) 
```


## **5、修改表**

### **5-1 修改表名**

```
ALTER TABLE 旧表名 RENAME 新表名;
```
实例

```
mysql> alter table t3 rename t4;
Query OK, 0 rows affected (0.19 sec)
```

### **5-2 增加字段**

添加到最后语法

```
ALTER TABLE 表名
ADD 字段名 列类型 [可选的参数],
ADD 字段名 列类型 [可选的参数];
```

实例

```
mysql> create table t88(
   id int,
   age int
)charset=utf8;

mysql> alter table t88 add name varchar(32) not null default '';
Query OK, 0 rows affected (0.82 sec)
Records: 0 Duplicates: 0  Warnings: 0
```

上面添加的列永远是添加在最后一列之后,还可以添加到最前面


**添加到开头语法**


```
ALTER TABLE 表名
ADD 字段名 列类型 [可选的参数] FIRST;
```

**实例**

```
mysql> alter table t88 add name3 varchar(32) not null default '' first;
Query OK, 0 rows affected (0.83 sec)
Records: 0 Duplicates: 0  Warnings:0
```

**添加到指定字段后面**

```
ALTER TABLE 表名
ADD 字段名 列类型 [可选的参数] AFTER 字段名;
```

**实例**

```
mysql> alter table t88 add name4 varchar(32) not null default '' after name;

Query OK, 0 rows affected (0.68 sec)
Records: 0 Duplicates: 0  Warnings: 0
```

### **5-3 删除字段**

语法

```
ALTER TABLE 表名  DROP 字段名;
```

实例

```
mysql> alter table t88 drop name4;
Query OK, 0 rows affected (0.66 sec)
Records: 0 Duplicates: 0  Warnings: 0
```

### **5-4 修改字段**

**语法1**

```
ALTER TABLE 表名 MODIFY 字段名 数据类型 [完整性约束条件…];
```

实例

```
mysql> alter table t88 modify name char(20);

Query OK, 1 row affected (0.88 sec)
Records: 1 Duplicates: 0  Warnings: 0
```

**语法2**

```
ALTER TABLE 表名 CHANGE 旧字段名 新字段名 新数据类型 [完整性约束条件
```

实例2

```
mysql> alter table t88 change name name2 varchar(32) not null default '';
Query OK, 1 row affected (0.82 sec)
Records: 1 Duplicates: 0  Warnings:0
```
