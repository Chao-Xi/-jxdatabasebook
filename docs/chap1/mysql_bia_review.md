# **MySQL 基础知识复习**

## **1 SQL Primary Key**

* **Primary keys must contain unique values.**
* **A primary key column cannot have NULL values.**
* **A table can have only one primary key, which may consist of single or multiple fields**.
* **When multiple fields are used as a primary key, they are called a composite key.**
*  **作为Primary Key的域/域组不能为null，而Unique Key可以**。
*  在一个表中只能有一个Primary Key，而多个Unique Key可以同时存在

## **2 日志系统：执行一条SQL更新语句**

一条查询语句的执行过程一般是经过**连接器、分析器、优化器、执行器等功能模块，最后到达存储引擎**

### **2-1 日志模块：redo log**

MySQL 里经常说到的 WAL 技术，**WAL 的全称是 Write-Ahead Logging，它的关键点就是先写日志，再写磁盘**。

**具体来说，当有一条记录需要更新的时候，InnoDB 引擎就会先把记录写到 redo log里面，并更新内存，这个时候更新就算完成了。**

**同时，InnoDB 引擎会在适当的时候，将这个操作记录更新到磁盘里面，而这个更新往往是在系统比较空闲的时候做。**

### **2-3 日志模块：binlog**

Binlog有两种模式 ： 

* statement 格式的话是记sql语句，
* row格式会记录行的内容，记两条，更新前和更新后都有。

**搜索 log 名称：`show variables like '%log_bin%';`**

**这两种日志有以下三点不同。:**

* **redo log 是 InnoDB 引擎特有的；binlog 是 MySQL 的 Server 层实现的，所有引擎都可以使用**。
* **`redo log` 是物理日志，记录的是“在某个数据页上做了什么修改”**；**binlog 是逻辑日志，记录的是这个语句的原始逻辑，比如“给 ID=2 这一行的 c 字段加 1 ”**。
* redo log 是循环写的，空间固定会用完；binlog 是可以追加写入的。“追加写”是指 binlog 文件写到一定大小后会切换到下一个，并不会覆盖以前的日志。


### **2-4 两阶段提交**

**为什么必须有“两阶段提交”呢？这是为了让两份日志之间的逻辑一致**

如果不使用“两阶段提交”，那么数据库的状态就有可能和用它的日志恢复出来的库的状态不一致。

**<mark>本质上是因为 redo log 负责事务； binlog负责归档恢复； 各司其职，相互配合，才提供(保证)了现有功能的完整性；</mark>**

redolog和binlog具有关联行，**在恢复数据时，redolog用于恢复主机故障时的未更新的物理数据，binlog用于备份操作**。

每个阶段的log操作都是记录在磁盘的，在恢复数据时，redolog 状态为commit则说明binlog也成功，直接恢复数据；

### **2-5 redo log 用于保证 crash-safe 能力**。

* Redo log 记录这个页 “做了什么改动”。 redo log记录的是当时的数据是怎么变化的，比如备份的时候记录table表中字段a的值是1→3（由1变成3）
* Binlog有两种模式，**statement 格式的话是记sql语句** 
	* row格式会记录行的内容，记两条，更新前和更新后都有。
	* binlog里面记录可能就是多条SQL执行之后a的值才等于3，比如`update table set a =1 where ID=1；` ,`update table set a=3 where ID=1；`或者执行过更多SQL才变成3。

## **3 事务隔离**

### **3-1 隔离性**

当数据库上有多个事务同时执行的时候，**就可能出现脏读（dirty read）、不可重复读（non-repeatable read）、幻读（phantom read）的问题，为了解决这些问题，就有了“隔离级别”的概念。**

* **脏读: 读到其他事务未提交的数据**；
* **不可重复读：前后读取的记录内容不一致；**
* **幻读：前后读取的记录数量不一致。**

### **3-2 隔离级别**

**隔离得越严实，效率就会越低。因此很多时候，我们都要在二者之间寻找一个平衡点**。

SQL 标准的事务隔离级别包括：读未提交（read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（serializable ）。下面我逐一为你解释：

* **读未提**交是指，一个事务还没提交时，它做的变更就能被别的事务看到。
* **读提交是指**，**一个事务提交之后，它做的变更才会被其他事务看到。**
* **可重复读是指**，一个事务执行过程中看到的数据，总是跟这个事务在启动时看到的数据是一致的。当然在可重复读隔离级别下，未提交变更对其他事务也是不可见的。
* **串行化**，顾名思义是对于同一行记录，**“写”会加“写锁”，“读”会加“读锁”。当出现读写锁冲突的时候，后访问的事务必须等前一个事务执行完成，才能继续执行。**

**启动参数**： `show variables like 'transaction_isolation';`

* **回滚日志的删除 undo log**

什么时候删除呢？答案是，在不需要的时候才删除。也就是说，系统会判断，当没有事务再需要用到这些回滚日志时，回滚日志会被删除

*  **尽量不要使用长事务**
*   `set autocommit=1（推荐）`  表示MySQL自动开启和提交事务。

## **4  深入浅出索引**

索引的出现其实就是为了提高数据查询的效率

三种常见、也比较简单的数据结构，**它们分别是哈希表、有序数组和搜索树。**

**每一个索引在 InnoDB 里面对应一棵 B+ 树。**

### **4-1 聚集索引和辅助索引**

数据库中的 `B+` 树索引可以分为聚集索引（`clustered index`）和辅助索引（`secondary index`），它们之间的最大区别就是: 

* **聚集索引中存放着一条行记录的全部信息**
* **辅助索引中只包含索引列和一个用于查找对应行记录的『书签』**

## **5 锁**

我们都知道锁的种类一般分为乐观锁和悲观锁两种，**InnoDB 存储引擎中使用的就是悲观锁，而按照锁的粒度划分，也可以分成行锁和表锁。**

### **5-1 乐观锁**

**乐观锁是一种思想，它其实并不是一种真正的『锁』**，它会先尝试对资源进行修改，在写回时判断资源是否进行了改变，如果没有发生改变就会写回，否则就会进行重试，**在整个的执行过程中其实都没有对数据库进行加锁**

### **5-2 悲观锁**

悲观锁就是一种真正的锁了，它会在获取资源前对资源进行加锁，确保同一时刻只有有限的线程能够访问该资源，其他想要尝试获取资源的操作都会进入等待状态，直到该线程完成了对资源的操作并且释放了锁后，其他线程才能重新操作资源；


## **6 Primary keys**

Primary keys must contain unique values.

A primary key column cannot have NULL values.

A table can have only one primary key, which may consist of single or multiple fields.

When multiple fields are used as a primary key, they are called a composite key.


## **7 同步**

### **如何判断mysql主从是否同步？该如何使其同步？**

```
Slave_IO_Running
Slave_SQL_Running；
```

### **2.mysql的innodb如何定位锁问题，mysql如何减少主从复制延迟？**

在使用 `show engine innodb status` 检查引擎状态时，发现了死锁问题

### **mysql如何减少主从复制延迟:**

* **从库硬件比主库差**，导致复制延迟
* 主从复制单线程，**如果主库写并发太大，来不及传送到从库，就会导致延迟**。更高版本的mysql可以支持多线程复制
* **慢SQL语句过多**
* 网络延迟
* **master负载: 主库读写压力大，导致复制延迟，架构的前端要加buffer及缓存层**

**另外， 2个可以减少延迟的参数:**

`-slave-net-timeout=seconds` 单位为秒 默认设置为 3600秒

参数含义：当slave从主数据库读取log数据失败后，等待多久重新建立连接并获取数据


`-master-connect-retry=seconds` 单位为秒 默认设置为 60秒

参数含义：当重新建立主从连接时，如果连接建立失败，间隔多久后重试。

**MySQL数据库主从同步延迟解决方案**


最简单的减少slave同步延时的方案就是在架构上做优化，尽量让主库的DDL快速执行。还有就是主库是写，对数据安全性较高，比如 `sync_binlog=1`，


### **mysql数据备份工具**

* mysqldump工具
* 基于LVM快照备份
* tar包备份
* percona提供的xtrabackup工具


## **8 NoSQL和关系数据库的区别？**

### **8-1 表结构**

* **SQL数据存在特定结构的表中**；
	* 在SQL中，必须定义好表和字段结构后才能添加数据，例如定义表的主键(primary key)，索引(index),触发器(trigger),存储过程(stored procedure)等。
	* 表结构可以在被定义之后更新，但是如果有比较大的结构变更的话就会变得比较复杂
* 而NoSQL则更加灵活和可扩展，**存储方式可以省是JSON文档、哈希表或者其他方式**。
	* 在NoSQL中，数据可以在任何时候任何地方添加，不需要先定义表。

### **8-2 外部关联数据**


* SQL中如果需要增加外部关联数据的话，规范化做法是在原表中增加一个外键，关联外部数据表。

**NoSQL:**

* 而在NoSQL中除了这种规范化的外部数据表做法以外，
* **我们还能用如下的非规范化方式把外部数据直接放到原数据集中，以提高查询效率**。缺点也比较明显，更新审核人数据的时候将会比较麻烦。


### **8-3 JOIN**

* **SQL中可以使用JOIN表链接方式**将多个关系数据表中的数据用一条简单的查询语句查询出来
* **NoSQL暂未提供类似JOIN的查询方式对多个数据集中的数据做查询**。所以大部分NoSQL使用非规范化的数据存储方式存储数据。

### **8-4 删除外部数据**

* SQL中不允许删除已经被使用的外部数据，
* 而NoSQL中则没有这种强耦合的概念，可以随时删除任何数据。


SQL中如果多张表数据需要同批次被更新，即如果其中一张表更新失败的话其他表也不能更新成功。这种场景可以通过事务来控制，可以在所有命令完成后再统一提交事务。而NoSQL中没有事务这个概念，每一个数据集的操作都是原子级的。


**在相同水平的系统设计的前提下，因为NoSQL中省略了JOIN查询的消耗，故理论上性能上是优于SQL的。**



## **9 如何对查询命令进行优化？**

* **<mark>应尽量避免全表扫描</mark>**，首先应考虑在 where 及 order by 涉及的列上建立索。
*  **<mark>应尽量避免在 where 子句中对字段进行 null 值判断，避免使用`!=`或`<>`操作符</mark>**，避免使用 or 连接条件，或在where子句中使用参数、对字段进行表达式或函数操作，否则会导致权标扫描
*  **<mark>应不要在 where 子句中的“=”左边进行函数、算术运算或其他表达式运算</mark>**，否则系统将可能无法正确使用索引
*  使用索引字段作为条件时，如果该索引是复合索引，那么必须使用到该索引中的第一个字段作为条件时才能保证系统使用该索引，否则该索引将不会被使用。
*  很多时候可考虑用 exists 代替 in
*  **<mark>尽量使用数字型字段</mark>**
*   **<mark>尽可能的使用 `varchar/nvarchar` 代替 `char/nchar`</mark>**
* **<mark>任何地方都不要使用 `select * from t` ，用具体的字段列表代替“*”</mark>**，不要返回用不到的任何字段。
*   **<mark>尽量使用表变量来代替临时表。</mark>**
*    **<mark>避免频繁创建和删除临时表，以减少系统表资源的消耗</mark>**
*   **<mark>尽量避免使用游标，因为游标的效率较差。</mark>**
*   在所有的存储过程和触发器的开始处设置 ·SET NOCOUNT ON·，在结束时设置 SET NOCOUNT OFF
*   尽量避免大事务操作，提高系统并发能力。
*   .尽量避免向客户端返回大数据量，若数据量过大，应该考虑相应需求是否合理。

## **10 MSSQL的死锁是如何产生的？**

如下是死锁产生的四个必要条件：

互斥条件：指进程对所分配到的资源进行排它性使用，即在一段时间内某资源只由一个进程占用。如果此时还有其它进程请求资源，则请求者只能等待，直至占有资源的进程用毕释放。

请求和保持条件：指进程已经保持至少一个资源，但又提出了新的资源请求，而该资源已被其它进程占有，此时请求进程阻塞，但又对自己已获得的其它资源保持不放。

不剥夺条件：指进程已获得的资源，在未使用完之前，不能被剥夺，只能在使用完时由自己释放。

环路等待条件：指在发生死锁时，必然存在一个进程——资源的环形链，即进程集合{P0，P1，P2，···，Pn}中的P0正在等待一个P1占用的资源；P1正在等待P2占用的资源，……，Pn正在等待已被P0占用的资源。

## **11 What are the advantages of NoSQL over traditional RDBMS?**

**NoSQL is better than RDBMS** because of the following reasons/properities of NoSQL:

* It supports semi-structured data and volatile data
* It **does not have schema**
* **Read/Write throughput is very high**
* **Horizontal scalability** can be achieved easily
* **Will support Bigdata in volumes** of Terra Bytes & Peta Bytes
* Provides good support for Analytic tools on top of Bigdata
* **Can be hosted in cheaper hardware machines**
* In-memory caching option is available to increase the performance of queries
* Faster development life cycles for developers

**Still, RDBMS is better than NoSQL** for the following reasons/properties of RDBMS:

* Transactions with **ACID properties - Atomicity, Consistency, Isolation & Durability**
* **Adherence to Strong Schema** of data being written/read
* Real time query management ( in case of data size < 10 Tera bytes )
* Execution of complex queries involving **join & group by clauses**


### When should I use a NoSQL database instead of a relational database?


**Relational databases enforces ACID**. So, you will have schema based transaction oriented data stores. It's proven and suitable for 99% of the real world applications. You can practically do anything with relational databases.

But, there are limitations on speed and scaling when it comes to massive high availability data stores. For example, Google and Amazon have terabytes of data stored in big data centers. Querying and inserting is not performant in these scenarios because of the blocking/schema/transaction nature of the RDBMs. That's the reason they have implemented their own databases (actually, key-value stores) for massive performance gain and scalability.

If you need a NoSQL db you usually know about it, possible reasons are:

* client wants 99.999% availability on a high traffic site.
* your data makes no sense in SQL, you find yourself doing multiple JOIN queries for accessing some piece of information.
* you are breaking the relational model, you have CLOBs that store denormalized data and you generate external indexes to search that data.
