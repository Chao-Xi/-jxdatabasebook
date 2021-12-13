# **第六节 Mysql操作实例(下)**


## **1、多表操作**

概念：当在查询时，所需要的数据不在一张表中，可能在两张表或多张表中。此时需要同时操作这 些表。即关联查询。

* 等值连接：在做多张表查询时，这些表中应该存在着有关联的两个字段。使用某一张表中的一条记 录与另外一张表通过相关联的两个字段进行匹配，组合成一条记录。
* 笛卡尔积：在做多张表查询时，使用某一张表中的每一条记录都与另外一张表的所有记录进行组 合。比如表A有x条，表B有y条，最终组合数为`x*y`，这个值就是笛卡尔积，通常没有意义。
* 内连接：只要使用了`join on`。就是内连接。查询效果与等值连接一样。

用法

```
表A [inner] join 表B on 关联条件
```

* 外连接：在做多张表查询时，所需要的数据，除了满足关联条件的数据外，还有不满足关联条件的 数据。此时需要使用外连接。

	* 驱动表(主表)：除了显示满足条件的数据，还需要显示不满足条件的数据的表 
	* 从表(副表)：只显示满足关联条件的数据的表 
	* 外连接分为三种

> 左外连接：表A left [outer] join 表B on 关联条件，表A是驱动表，表B是从表 右外连接：
> 
> 表A
right [outer] join 表B on 关联条件，表B是驱动表，表A是从表 全外连接：
> 
> 表A full [outer] join 表B on 关联条件，两张表的数据不管满不满足条件，都做显示。

* 自连接：在多张表进行关联查询时，这些表的表名是同一个。即自连接。外键：占用空间少，方便修改数据

### **1-1 一对多**

```
constraint 外键名 foreign key (被约束的字段) references 约束的表(约束的字段)
```

```
mysql> create table dep(
id int auto_increment primary key,
name varchar(32) not null default ''
)charset=utf8;

mysql> insert into dep (name) values ('研发部'),('运维部'),('行政部'),('市场部');

mysql> create table userinfo (
id int auto_increment primary key,
name varchar(32) not null default '',
depart_id int not null default 1,
constraint fk_user_depart foreign key (depart_id) references dep(id)
)charset utf8;

mysql> insert into userinfo (name, depart_id) values ('qfedu a',1);
mysql> insert into userinfo (name, depart_id) values ('qfedu b',2);
mysql> insert into userinfo (name, depart_id) values ('qfedu c',3);
mysql> insert into userinfo (name, depart_id) values ('qfedu d',4);
mysql> insert into userinfo (name, depart_id) values ('qfedu e',1);
mysql> insert into userinfo (name, depart_id) values ('qfedu f',2);
mysql> insert into userinfo (name, depart_id) values ('qfedu g',3);

# 以上7行符合外键要求，所以能插入不报错，但下边一行插入时会报错
mysql> insert into userinfo (name, depart_id) values ('qfedu h',5);
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that
corresponds to your MySQL server version for the right syntax to use near
'mysql> insert into userinfo (name, depart_id) values ('qfedu h',5)' at line 
```

### **1-2 多对多**

创建男生表

```
mysql> create table boy(
id int auto_increment primary key,
bname varchar(32) not null default ''
)charset=utf8;

mysql> insert into boy (bname) values ('xiaoming'),('xiaogang'),('xiaoqiang');
```

创建女生表

```
mysql> create table girl(
id int auto_increment primary key,
gname varchar(32) not null default ''
)charset=utf8;

mysql> insert into girl (gname) values ('xiaohong'),('xiaoli'),('xiaojiao');
```

创建关联表

```
mysql> create table b2g(
id int auto_increment primary key,
bid int not null default 1,
gid int not null default 0,
constraint fk_b2g_boy foreign key (bid) references boy(id),
constraint fk_b2g_girl foreign key (gid) references girl(id)
)charset utf8;

mysql> insert into b2g (bid, gid) values (1,1),(1,2),(2,3),(3,3),(2,2);
```

**用到 left jion**

```
mysql> select * from boy left join b2g on boy.id=b2g.bid left join girl on girl.id=b2g.gid;
+----+-----------+------+------+------+------+----------+
| id | bname     | id   | bid | gid | id   | gname   |
+----+-----------+------+------+------+------+----------+
|  1 | xiaoming |    1 |    1 |    1 |    1 | xiaohong |
|  1 | xiaoming |    2 |    1 |    2 |    2 | xiaoli   |
|  2 | xiaogang |    5 |    2 |    2 |    2 | xiaoli   |
|  2 | xiaogang |    3 |    2 |    3 |    3 | xiaojiao |
|  3 | xiaoqiang |    4 |    3 |    3 |    3 | xiaojiao |
+----+-----------+------+------+------+------+----------+
5 rows in set (0.01 sec)


mysql> select bname, gname from boy left join b2g on boy.id=b2g.bid left join girl on girl.id=b2g.gid;
+-----------+----------+
| bname     | gname   |
+-----------+----------+
| xiaoming | xiaohong |
| xiaoming | xiaoli   |
| xiaogang | xiaoli   |
| xiaogang | xiaojiao |
| xiaoqiang | xiaojiao |
+-----------+----------+
5 rows in set (0.00 sec)
```

### **1-3 一对一**

**创建员工信息表**

```
mysql> create table user(
id int auto_increment primary key,
name varchar(32) not null default ''
)charset=utf8;

mysql> insert into user (name) values ('xiaoming'),('xiaogang'),('xiaoqiang');

mysql> select * from user;
+----+-----------+
| id | name     |
+----+-----------+
|  1 | xiaoming |
|  2 | xiaogang |
|  3 | xiaoqiang |
+----+-----------+
3 rows in set (0.00 sec)
```

**创建员工工资表**

```
mysql> create table priv(
id int auto_increment primary key,
salary int not null default 0,
uid int not null default 1,
constraint fk_priv_user foreign key (uid) references user(id),
unique(uid)
)charset=utf8;

mysql> insert into priv (salary, uid) values (2000, 1),(2500,2),(3000,3);

mysql> select * from priv;
+----+--------+-----+
| id | salary | uid |
+----+--------+-----+
|  1 |   2000 |   1 |
|  2 |   2500 |   2 |
|  3 |   3000 |   3 |
+----+--------+-----+
3 rows in set (0.00 sec)
```

## **2、多表联查**

```
mysql>  select userinfo.name as uname, dep.name as dname from userinfo left join dep on depart_id=dep.id;
+---------+-----------+
| uname   | dname     |
+---------+-----------+
| qfedu a | 研发部   |
| qfedu e | 研发部   |
| qfedu b | 运维部   |
| qfedu f | 运维部   |
| qfedu c | 行政部   |
| qfedu g | 行政部   |
| qfedu d | 市场部   |
+---------+-----------+
7 rows in set (0.00 sec)

mysql> select userinfo.name as uname, dep.name as dname from userinfo left join
dep on depart_id=dep.id;
+---------+-----------+
| uname   | dname     |
+---------+-----------+
| qfedu a | 研发部   |
| qfedu e | 研发部   |
| qfedu b | 运维部   |
| qfedu f | 运维部   |
| qfedu c | 行政部   |
| qfedu g | 行政部   |
| qfedu d | 市场部   |
+---------+-----------+
7 rows in set (0.00 sec)
```

## **3、单表多表查询练习**

### **3-1 创建班级表**

```
mysql> create table class1(
cid int auto_increment primary key,
caption varchar(32) not null default ''
)charset=utf8;
mysql> insert into class1 (caption) values ('三年二班'),('一年三班'),('三年一班');


mysql> select * from class1;
+-----+--------------+
| cid | caption     |
+-----+--------------+
|   1 | 三年二班     |
|   2 | 一年三班     |
|   3 | 三年一班     |
+-----+--------------+
3 rows in set (0.00 sec)
```

### **3-2 创建学生表**

```
mysql> create table stu(
sid int auto_increment primary key,
sname varchar(32) not null default '',
gender varchar(32) not null default '',
class_id int not null default 1,
constraint fk_stu_class1 foreign key (class_id) references class1(cid)
)charset=utf8;


mysql> insert into stu (sname, gender, class_id) values ('小薇','女',1),('小雪','女',1),('小明','男',2);

mysql> select * from stu;
+-----+--------+--------+----------+
| sid | sname | gender | class_id |
+-----+--------+--------+----------+
|   1 | 小薇   | 女     |        1 |
|   2 | 小雪   | 女     |        1 |
|   3 | 小明   | 男     |        2 |
+-----+--------+--------+----------+
3 rows in set (0.00 sec)
```

### **3-3 创建老师表**

```
mysql> create table tea(
tid int auto_increment primary key,
tname varchar(32) not null default ''
)charset=utf8;

mysql> insert into tea (tname) values ('qfedu'),('Frank'),('Julie');

mysql> select * from tea;
+-----+-------+
| tid | tname |
+-----+-------+
|   1 | qfedu |
|   2 | Frank |
|   3 | Julie |
+-----+-------+
3 rows in set (0.00 sec)
```

### **3-4 创建课程表**

```
mysql> create table cour(
cid int auto_increment primary key,
cname varchar(32) not null default '',
tea_id int not null default 1,
constraint fk_cour_tea foreign key (tea_id) references tea(tid)
)charset=utf8;

mysql> insert into cour(cname, tea_id) values ('生物',1),('体育',1),('物理',2);

mysql> select * from cour;
+-----+--------+--------+
| cid | cname | tea_id |
+-----+--------+--------+
|   1 | 生物   |      1 |
|   2 | 体育   |      1 |
|   3 | 物理   |      2 |
+-----+--------+--------+
3 rows in set (0.00 sec)
```

### **3-5 成绩表**

```
mysql> create table sco(
sid int auto_increment primary key,
stu_id int not null default 1,
cou_id int not null default 1,
number int not null default 0,
constraint fk_sco_stu foreign key (stu_id) references stu(sid),
constraint fk_sco_cour foreign key (cou_id) references cour(cid)
)charset utf8;

mysql> insert into sco (stu_id, cou_id, number) values (1,1,60),(1,2,59), (2,2,100);

mysql> select * from sco;
+-----+--------+--------+--------+
| sid | stu_id | cou_id | number |
+-----+--------+--------+--------+
|   1 |      1 |      1 |     60 |
|   2 |      1 |      2 |     59 |
|   3 |      2 |      2 |    100 |
+-----+--------+--------+--------+
3 rows in set (0.00 sec)
```

**3-6 查询所有大于60分的学生的姓名和学号 (DISTINCT: 去重)**

```
mysql> select distinct stu.sid,stu.sname from sco left join stu on
stu.sid=stu_id where number>=60;
+------+--------+
| sid | sname |
+------+--------+
|    1 | 小薇   |
|    2 | 小雪   |
+------+--------+
2 rows in set (0.00 sec)
```
