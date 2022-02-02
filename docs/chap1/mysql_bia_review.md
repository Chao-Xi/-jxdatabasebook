# **MySQL 基础知识复习**

## **1 SQL Primary Key**

* **Primary keys must contain unique values.**
* **A primary key column cannot have NULL values.**
* **A table can have only one primary key, which may consist of single or multiple fields**.
* **When multiple fields are used as a primary key, they are called a composite key.**

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
