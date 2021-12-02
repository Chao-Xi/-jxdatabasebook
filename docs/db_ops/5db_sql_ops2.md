# **第五节 MySQL 操作实例(中)**

### **1、删出表**

```
drop table 表名;  # 线上禁用
```

```
mysql> drop table t9;
Query OK, 0 rows affected (0.18 sec)
```

### **2、查询表**

```
mysql> show tables;
+----------------+
| Tables_in_test |
+----------------+
| t1             |
+----------------+
1 row in set (0.00 sec)
```

## **3、复制表结构**

```
mysql> create table t8 like t1;
Query OK, 0 rows affected (0.33 sec)
```

## **4、操作表数据行**

### **4-1 增加数据**


```
insert into 表名 (列1, 列2) values (值1,'值2');
```
```
# 暴力复制
mysql> insert into t1 (name) select name from t8;
Query OK, 4 rows affected (0.09 sec)
Records: 4 Duplicates: 0  Warnings: 0
```

### **4-2 删除数据**

**delete from 表名 where 条件;**


```
mysql> insert into t1 (id, name) values (1, 'qf1');
mysql> insert into t1 (id, name) values (2, 'qf2');
mysql> insert into t1 (id, name) values (3, 'qf3');
mysql> insert into t1 (id, name) values (4, 'qf4');
mysql> delete from t1 where id=1;
mysql> delete from t1 where id>1;
mysql> delete from t1 where id>=1;
mysql> delete from t1 where id<1;
mysql> delete from t1 where id<=1;
mysql> delete from t1 where id>=1 and id<10;
Query OK, 1 row affected (0.06 sec
```

**truncate 表名; # 没有where条件的**

```
mysql> truncate t1;
Query OK, 0 rows affected (0.25 sec)

mysql> select * from t1;
Empty set (0.00 sec)

mysql> insert into t1 (id, name) values (1, 'qf1');
Query OK, 1 row affected (0.06 sec)

mysql> select * from t1;
+------+------+
| id   | name |
+------+------+
|    1 | qf1 |
+------+------+
1 row in set (0.00 sec)
```

### **4-3 区别**

* delete 之后，插入数据从上一次主键自增加1开始， truncate则是从1开始
* **delete 删除， 是一行一行的删除， truncate：全选删除 truncate删除的速度是高于delete的**


## **5、修改数据**

```
update 表名 set 列名1=新值1,列名2=新值2 where 条件;
```
```
mysql> insert into t1 (id, name) values (1, 'qf1');
Query OK, 1 row affected (0.00 sec)
mysql> insert into t1 (id, name) values (2, 'qf2');
Query OK, 1 row affected (0.00 sec)
mysql> insert into t1 (id, name) values (3, 'qf3');
Query OK, 1 row affected (0.00 sec)
mysql> insert into t1 (id, name) values (4, 'qf4');
Query OK, 1 row affected (0.00 sec)
mysql> select * from t1;
+------+------+
| id   | name |
+------+------+
|    1 | qf1 |
|    2 | qf2 |
|    3 | qf3 |
|    4 | qf4 |
+------+------+
4 rows in set (0.00 sec)
mysql> update t1 set name='qfx' where id=1;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
mysql> select * from t1 ;
+------+------+
| id   | name |
+------+------+
|    1 | qfx |
|    2 | qf2 |
|    3 | qf3 |
|    4 | qf4 |
+------+------+
4 rows in set (0.00 sec)
```

## **6、查询数据**

```
select 列1, 列2 from 表名; （*代表查询所有的列）
select * from 表名; （*代表查询所有的列）
```

```
mysql> select * from t1;
+------+------+------+
| id   | name | age |
+------+------+------+
|    1 | qfx | NULL |
|    2 | qf2 | NULL |
|    3 | qf3 | NULL |
|    4 | qf4 | NULL |
|    5 | qf3 | NULL |
+------+------+------+
6 rows in set (0.00 sec)
            
# between..and...: 取值范围是闭区间
            
mysql> select * from t1 where id between 2 and 3;
+------+------+------+
| id   | name | age |
+------+------+------+
|    2 | qf2 | NULL |
|    3 | qf3 | NULL |
+------+------+------+
2 rows in set (0.00 sec)
        
# 避免重复DISTINCT
mysql> insert into t1 (id, name) values (5, 'qf3');
Query OK, 1 row affected (0.00 sec)
mysql> select * from t1;
+------+------+
| id   | name |
+------+------+
|    1 | qfx |
|    2 | qf2 |
|    3 | qf3 |
|    4 | qf4 |
|    5 | qf3 |
+------+------+
5 rows in set (0.00 sec)
mysql> select distinct name from t1;
+------+
| name |
+------+
| qfx |
| qf2 |
| qf3 |
| qf4 |
+------+
4 rows in set (0.00 sec)
            
# 通过四则运算查询 （不要用）
mysql> alter table t1 add age int;
Query OK, 0 rows affected (0.03 sec)
Records: 0 Duplicates: 0  Warnings: 0
mysql> insert into t1 (id, name,age) values (5, 'qf3',10);
Query OK, 1 row affected (0.01 sec)
mysql> select * from t1;
+------+------+------+
| id   | name | age |
+------+------+------+
|    1 | qfx | NULL |
|    2 | qf2 | NULL |
|    3 | qf3 | NULL |
|    4 | qf4 | NULL |
|    5 | qf3 | NULL |
|    5 | qf3 |   10 |
+------+------+------+
6 rows in set (0.00 sec)
mysql> select name, age*10 from t1;
+------+--------+
| name | age*10 |
+------+--------+
| qfx |   NULL |
| qf2 |   NULL |
| qf3 |   NULL |
| qf4 |   NULL |
| qf3 |   NULL |
| qf3 |    100 |
+------+--------+
6 rows in set (0.00 sec)
mysql>  select name, age*10 as age from t1;
+------+------+
| name | age |
+------+------+
| qfx | NULL |
| qf2 | NULL |
| qf3 | NULL |
| qf4 | NULL |
| qf3 | NULL |
| qf3 |  100 |
+------+------+
6 rows in set (0.00 sec)
mysql> select * from t1 where id in (3,5,6);
+------+------+------+
| id   | name | age |
+------+------+------+
|    3 | qf3 | NULL |
|    5 | qf3 | NULL |
|    5 | qf3 |   10 |
+------+------+------+
3 rows in set (0.00 sec)
# like : 模糊查询
# 以x开头：
mysql> select * from t1 where name like 'q%';
+------+------+------+
| id   | name | age |
+------+------+------+
|    1 | qfx | NULL |
|    2 | qf2 | NULL |
|    3 | qf3 | NULL |
|    4 | qf4 | NULL |
|    5 | qf3 | NULL |
|    5 | qf3 |   10 |
+------+------+------+
6 rows in set (0.00 sec)
                
# 以x结尾
mysql> select * from t1 where name like '%3';
+------+------+------+
| id   | name | age |
+------+------+------+
|    3 | qf3 | NULL |
|    5 | qf3 | NULL |
|    5 | qf3 |   10 |
+------+------+------+
3 rows in set (0.00 sec)
            
# 包含x的：
mysql> select * from t1 where name like '%2%';
+------+------+------+
| id   | name | age |
+------+------+------+
|    2 | qf2 | NULL |
+------+------+------+
1 row in set (0.00 sec)
```

查询时用‘Name Is Null’ 作为条件

```
mysql>create table t8(
id int auto_increment primary key,
name varchar(32),
email varchar(32)
)charset=utf8;
mysql>insert into t8(email) values ('test');
mysql> select * from t8;
+----+------+-------+
| id | name | email |
+----+------+-------+
|  1 | NULL | test |
+----+------+-------+
1 row in set (0.01 sec)
mysql> select * from t8 where name is null;
+----+------+-------+
| id | name | email |
+----+------+-------+
|  1 | NULL | test |
+----+------+-------+
1 row in set (0.00 sec)
```

* 查询时用`‘Name=’‘ ’`作为查询条件

```
mysql> create table t9(
id int auto_increment primary key,
name varchar(32) not null default '',
email varchar(32) not null default ''
)charset=utf8;
mysql> insert into t9 (email) values ('test');
mysql> select * from t9;
+----+------+-------+
| id | name | email |
+----+------+-------+
|  1 |     | test |
+----+------+-------+
1 row in set (0.00 sec)
mysql> select * from t9 where name='';
+----+------+-------+
| id | name | email |
+----+------+-------+
|  1 |     | test |
+----+------+-------+
1 row in set (0.01 sec)
```

## **7、单表操作**

### **7-1 单表查询的语法：**

```
select 字段1，字段2 from 表名  
                        where 条件
                        group by field
                        having 筛选
                        order by field
                        limit 限制条数
```


### **7-2 分组：Group By**

分组指的是：将所有记录按照某个相同字段进行归类，比如
针对员工信息表的职位分组，或者按照 性别进行分组等

用法：select 聚合函数，字段名 from 表名

* group by 分组的字段；group by 是分组的关键词， 必须和聚合函数 一起出现
* where 条件语句和 group by 分组语句的先后顺序：
* `where > group by > having`

**创建表**

```
create table test(
id int not null unique auto_increment,
name varchar(20) not null,
gender enum('male','female') not null default 'male',
age int(3) unsigned not null default 28,
hire_date date not null,
post varchar(50),
post_comment varchar(100),
salary double(15,2),
office int,          
depart_id int
);
```

**2. 插入数据**

教学部 1，销售部门 2 ，运营部门 3.

```
insert into test(name,gender,age,hire_date,post,salary,office,depart_id) values
('egon','male',18,'20170301','www.test.com',7300.33,401,1),    
('yangsheng','male',78,'20150302','teacher',1000000.31,401,1),
('yangqiang','male',81,'20130305','teacher',8300,401,1),
('liuchao','male',73,'20140701','teacher',3500,401,1),
('luoyinsheng','male',28,'20121101','teacher',2100,401,1),
('nana','female',18,'20110211','teacher',9000,401,1),
('jinxin','male',18,'19000301','teacher',30000,401,1),
('xingdian','male',48,'20101111','teacher',10000,401,1),

('meiling','female',48,'20150311','sale',3000.13,402,2),
('lili','female',38,'20101101','sale',2000.35,402,2),
('xiaomei','female',18,'20110312','sale',1000.37,402,2),
('zhenzhen','female',18,'20160513','sale',3000.29,402,2),
('qianqian','female',28,'20170127','sale',4000.33,402,2),

('zhangye','male',28,'20160311','operation',10000.13,403,3),
('xiaolong','male',18,'19970312','operation',20000,403,3),
('zhaowei','female',18,'20130311','operation',19000,403,3),
('wujing','male',18,'20150411','operation',18000,403,3),
('liying','female',18,'20140512','operation',17000,403,3)
```

以性别为例， 进行分组， 统计一下男生和女生的人数是多少个：

**`group by gender`**

```
mysql> select count(id), gender from test group by gender;
+-----------+--------+
| count(id) | gender |
+-----------+--------+
|        10 | male   |
|         8 | female |
+-----------+--------+
2 rows in set (0.00 sec)
```

对部门进行分组， 求出每个部门年龄最大的那个人

注意：出现 `sql_mode=only_full_group_by` 错误 需修改 配置文件`/etc/my.cnf [mysqld]` 段添加 sql模式，重启 `mysqld`

```
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_
ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
```

```
mysql> select depart_id,name, max(age) from test group by depart_id;
+-----------+---------+----------+
| depart_id | name   | max(age) |
+-----------+---------+----------+
|         1 | egon   |       81 |
|         2 | meiling |       48 |
|         3 | zhangye |       28 |
+-----------+---------+----------+
3 rows in set (0.00 sec)
```

* Min : 求最小的
* Sum ：求和

```
mysql> select depart_id, sum(age) from test group by depart_id;
+-----------+----------+
| depart_id | sum(age) |
+-----------+----------+
|         1 |      362 |
|         2 |      150 |
|         3 |      100 |
+-----------+----------+
3 rows in set (0.00 sec)
```

Count ：计数

```
mysql> select depart_id,count(depart_id) from test group by depart_id;
+-----------+------------------+
| depart_id | count(depart_id) |
+-----------+------------------+
|         1 |                8 |
|         2 |                5 |
|         3 |                5 |
+-----------+------------------+
3 rows in set (0.00 sec)
```

Avg ：平均数

```
mysql> select depart_id, avg(age) from test group by depart_id;
+-----------+----------+
| depart_id | avg(age) |
+-----------+----------+
|         1 |  45.2500 |
|         2 |  30.0000 |
|         3 |  20.0000 |
+-----------+----------+
3 rows in set (0.00 sec)
```

**3. Having**

Having 用于对 Group By 之后的数据进行进一步的筛选

```
mysql> select depart_id, avg(age) from test group by depart_id having avg(age)>35;
+-----------+----------+
| depart_id | avg(age) |
+-----------+----------+
|         1 |  45.2500 |
+-----------+----------+
1 row in set (0.01 sec)
```


**4. Order By: Order By 字段名 Asc（升序）/Desc(降序)**

对多个字段进行排序：

`age asc`, `depart_id desc;` 表示先对age进行降序，再把age相等的行按部门号进行升序排列

```
mysql> select * from test order by age asc, depart_id desc;
+----+-------------+--------+-----+------------+---------------+--------------+-
-----------+--------+-----------+
| id | name       | gender | age | hire_date | post         | post_comment |
salary     | office | depart_id |
+----+-------------+--------+-----+------------+---------------+--------------+-
-----------+--------+-----------+
| 18 | liying     | female |  18 | 2014-05-12 | operation     | NULL         |
  17000.00 |    403 |         3 |
| 17 | wujing     | male   |  18 | 2015-04-11 | operation     | NULL         |
  18000.00 |    403 |         3 |
| 16 | zhaowei     | female |  18 | 2013-03-11 | operation     | NULL         |
  19000.00 |    403 |         3 |
| 15 | xiaolong   | male   |  18 | 1997-03-12 | operation     | NULL         |
  20000.00 |    403 |         3 |
| 11 | xiaomei     | female |  18 | 2011-03-12 | sale         | NULL         |
   1000.37 |    402 |         2 |
| 12 | zhenzhen   | female |  18 | 2016-05-13 | sale         | NULL         |
   3000.29 |    402 |         2 |
|  6 | nana       | female |  18 | 2011-02-11 | teacher       | NULL         |
   9000.00 |    401 |         1 |
|  7 | jinxin     | male   |  18 | 1900-03-01 | teacher       | NULL         |
  30000.00 |    401 |         1 |
|  1 | egon       | male   |  18 | 2017-03-01 | www.test.com | NULL         |
   7300.33 |    401 |         1 |
| 14 | zhangye     | male   |  28 | 2016-03-11 | operation     | NULL         |
  10000.13 |    403 |         3 |
| 13 | qianqian   | female |  28 | 2017-01-27 | sale         | NULL         |
   4000.33 |    402 |         2 |
|  5 | luoyinsheng | male   |  28 | 2012-11-01 | teacher       | NULL         |
   2100.00 |    401 |         1 |
| 10 | lili       | female |  38 | 2010-11-01 | sale         | NULL         |
   2000.35 |    402 |         2 |
|  9 | meiling     | female |  48 | 2015-03-11 | sale         | NULL         |
   3000.13 |    402 |         2 |
|  8 | xingdian   | male   |  48 | 2010-11-11 | teacher       | NULL         |
  10000.00 |    401 |         1 |
|  4 | liuchao     | male   |  73 | 2014-07-01 | teacher       | NULL         |
   3500.00 |    401 |         1 |
|  2 | yangsheng   | male   |  78 | 2015-03-02 | teacher       | NULL         |
1000000.31 |    401 |         1 |
|  3 | yangqiang   | male   |  81 | 2013-03-05 | teacher       | NULL         |
   8300.00 |    401 |         1 |
+----+-------------+--------+-----+------------+---------------+--------------+-
-----------+--------+-----------+
18 rows in set (0.00 sec)
```

**`select * from test order by depart_id asc, age desc;`**

```
mysql> select * from test order by depart_id asc, age desc;
+----+-------------+--------+-----+------------+---------------+--------------+-
-----------+--------+-----------+
| id | name       | gender | age | hire_date | post         | post_comment |
salary     | office | depart_id |
+----+-------------+--------+-----+------------+---------------+--------------+-
-----------+--------+-----------+
|  3 | yangqiang   | male   |  81 | 2013-03-05 | teacher       | NULL         |
   8300.00 |    401 |         1 |
|  2 | yangsheng   | male   |  78 | 2015-03-02 | teacher       | NULL         |
1000000.31 |    401 |         1 |
|  4 | liuchao     | male   |  73 | 2014-07-01 | teacher       | NULL         |
   3500.00 |    401 |         1 |
|  8 | xingdian   | male   |  48 | 2010-11-11 | teacher       | NULL         |
  10000.00 |    401 |         1 |
|  5 | luoyinsheng | male   |  28 | 2012-11-01 | teacher       | NULL         |
   2100.00 |    401 |         1 |
|  6 | nana       | female |  18 | 2011-02-11 | teacher       | NULL         |
   9000.00 |    401 |         1 |
|  7 | jinxin     | male   |  18 | 1900-03-01 | teacher       | NULL         |
  30000.00 |    401 |         1 |
|  1 | egon       | male   |  18 | 2017-03-01 | www.test.com | NULL         |
   7300.33 |    401 |         1 |
|  9 | meiling     | female |  48 | 2015-03-11 | sale         | NULL         |
   3000.13 |    402 |         2 |
| 10 | lili       | female |  38 | 2010-11-01 | sale         | NULL         |
   2000.35 |    402 |         2 |
| 13 | qianqian   | female |  28 | 2017-01-27 | sale         | NULL         |
   4000.33 |    402 |         2 |
| 11 | xiaomei     | female |  18 | 2011-03-12 | sale         | NULL         |
   1000.37 |    402 |         2 |
| 12 | zhenzhen   | female |  18 | 2016-05-13 | sale         | NULL         |
   3000.29 |    402 |         2 |
| 14 | zhangye     | male   |  28 | 2016-03-11 | operation     | NULL         |
  10000.13 |    403 |         3 |
| 15 | xiaolong   | male   |  18 | 1997-03-12 | operation     | NULL         |
  20000.00 |    403 |         3 |
| 16 | zhaowei     | female |  18 | 2013-03-11 | operation     | NULL         |
  19000.00 |    403 |         3 |
| 17 | wujing     | male   |  18 | 2015-04-11 | operation     | NULL         |
  18000.00 |    403 |         3 |
| 18 | liying     | female |  18 | 2014-05-12 | operation     | NULL         |
  17000.00 |    403 |         3 |
+----+-------------+--------+-----+------------+---------------+--------------+-
-----------+--------+-----------+
18 rows in set (0.00 sec)
```

**5. Limit 分页：Limit Offset, Size**

* offset 表示 行数据索引；size 表示取多少条数据
* 从第 offset 行开始，取 size 行数据。

**` limit 0,10`**

```
mysql> select * from test limit 0,10;
+----+-------------+--------+-----+------------+---------------+--------------+-
-----------+--------+-----------+
| id | name       | gender | age | hire_date | post         | post_comment |
salary     | office | depart_id |
+----+-------------+--------+-----+------------+---------------+--------------+-
-----------+--------+-----------+
|  1 | egon       | male   |  18 | 2017-03-01 | www.test.com | NULL         |
   7300.33 |    401 |         1 |
|  2 | yangsheng   | male   |  78 | 2015-03-02 | teacher       | NULL         |
1000000.31 |    401 |         1 |
|  3 | yangqiang   | male   |  81 | 2013-03-05 | teacher       | NULL         |
   8300.00 |    401 |         1 |
|  4 | liuchao     | male   |  73 | 2014-07-01 | teacher       | NULL         |
   3500.00 |    401 |         1 |
|  5 | luoyinsheng | male   |  28 | 2012-11-01 | teacher       | NULL         |
   2100.00 |    401 |         1 |
|  6 | nana       | female |  18 | 2011-02-11 | teacher       | NULL         |
   9000.00 |    401 |         1 |
|  7 | jinxin     | male   |  18 | 1900-03-01 | teacher       | NULL         |
  30000.00 |    401 |         1 |
|  8 | xingdian   | male   |  48 | 2010-11-11 | teacher       | NULL         |
  10000.00 |    401 |         1 |
|  9 | meiling     | female |  48 | 2015-03-11 | sale         | NULL         |
   3000.13 |    402 |         2 |
| 10 | lili       | female |  38 | 2010-11-01 | sale         | NULL         |
   2000.35 |    402 |         2 |
+----+-------------+--------+-----+------------+---------------+--------------+-
-----------+--------+-----------+
10 rows in set (0.00 sec)
```

从第6行开始取10行  **`imit 6,10`**

```
mysql> select * from test limit 6,10;
+----+-----------+--------+-----+------------+-----------+--------------+-------
---+--------+-----------+
| id | name     | gender | age | hire_date | post     | post_comment | salary
  | office | depart_id |
+----+-----------+--------+-----+------------+-----------+--------------+-------
---+--------+-----------+
|  7 | jinxin   | male   |  18 | 1900-03-01 | teacher   | NULL         |
30000.00 |    401 |         1 |
|  8 | 成龙     | male   |  48 | 2010-11-11 | teacher   | NULL         |
10000.00 |    401 |         1 |
|  9 | 歪歪     | female |  48 | 2015-03-11 | sale     | NULL         |
3000.13 |    402 |         2 |
| 10 | 丫丫     | female |  38 | 2010-11-01 | sale     | NULL         |
2000.35 |    402 |         2 |
| 11 | 丁丁     | female |  18 | 2011-03-12 | sale     | NULL         |
1000.37 |    402 |         2 |
| 12 | 星星     | female |  18 | 2016-05-13 | sale     | NULL         |
3000.29 |    402 |         2 |
| 13 | 格格     | female |  28 | 2017-01-27 | sale     | NULL         |
4000.33 |    402 |         2 |
| 14 | 张野     | male   |  28 | 2016-03-11 | operation | NULL         |
10000.13 |    403 |         3 |
| 15 | 程咬金   | male   |  18 | 1997-03-12 | operation | NULL         |
20000.00 |    403 |         3 |
| 16 | 程咬银   | female |  18 | 2013-03-11 | operation | NULL         |
19000.00 |    403 |         3 |
+----+-----------+--------+-----+------------+-----------+--------------+-------
---+--------+-----------+
10 rows in set (0.00 sec)
```