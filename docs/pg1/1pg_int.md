# **PostgreSQL Interview Questions 2022** 

### **1. What is the process of splitting a large table into smaller pieces called in PostgreSQL?**

It is called **table partitioning**.

### **2. What is a partitioned table in PostgreSQL?**


The partitioned table is a **logical structure**. It is used to split a large table into smaller pieces, which are called partitions.

### **3. What purpose does pgAdmin in PostgreSQL server?**

The pgAdmin in PostgreSQL is a data administration tool. It serves the purpose of retrieving, developing, testing, and maintaining databases.

### **4.How can you avoid unnecessary locking of a database?**

**We can use MVCC (Multi-version concurrency control)** to avoid unnecessary locking of a database.

### **5. What is PL/Python?**


PL/Python is a procedural language to which PostgreSQL provides support.

### **6. Which are the methods PostgreSQL provides to create a new database?**

PostgreSQL provides the following methods to create a new database:

* Using **CREATE DATABASE**, an SQL command
* Using created a command-line executable

### **7 How do you delete the database in PostgreSQL?**

We can delete the database by using any one of the below options:

* Using **DROP DATABASE**, an SQL command
* Using dropdb a command-line executable

### **8 What does a schema contain?**

A schema contains tables along with **data types, views, indexes, operators, sequences, and functions**.

### **9 What are the different operators in PostgreSQL?**

The PostgreSQL operators include - Arithmetic operators, Comparison operators, Logical operators, and Bitwise operators.

### **10. What are database callback functions called? What is its purpose?**

The database callback functions are called PostgreSQL Triggers. When a specified database event occurs, **the PostgreSQL Triggers are performed or invoked automatically.**


### **11. What indexes are used?**

**Indexes are used by the search engine to speed up data retrieval**.


### **12. What does a Cluster index do?**


Cluster index sorts table data rows based on their key values.

### **13. What are the benefits of specifying data types in columns while creating a table?**

Some of these benefits include consistency, compactness, validation, and performance.

### **14. What do you need to do to update statistics in PostgreSQL?**

To update statistics in PostgreSQL, **we need to use a special function called a vacuum.**



### **15. What is the disadvantage of the DROP TABLE command in deleting complete data from an existing table?**


**Though the DROP TABLE command has the ability to delete complete data from an existing table,** the disadvantage with it is - it removes complete table structure from the database. 

Due to this, we need to re-create a table to store data.


### **16. How can you delete complete data from an existing table?**

We can delete complete data from an existing table using the PostgreSQL ***TRUNCATE TABLE command.***

### **17 What are the different properties of a transaction in PostgreSQL? Which acronym is used to refer to them?**

The properties of a transaction in PostgreSQL include Atomicity, Consistency, Isolation, and Durability. These are referred to by the acronym, namely ACID. 

### **18 What purpose does the CTIDs field serve?**


The CTIDs field **identifies the specific physical rows in a table according to their block and offsets positions in that table.**

### **19. Which are the commands used to control transactions in PostgreSQL?**


The commands used to control transactions in PostgreSQL are BEGIN TRANSACTION, COMMIT, and ROLLBACK.


### **20.What are the main differences between SQL and PostgreSQL?**

PostgreSQL is an advanced version of SQL.  Some of the differences between these two include the following:

* Unlike SQL, **views in PostgreSQL are not updatable.**
* Another difference is whereas SQL provides computed columns; the same cannot be expected from PostgreSQL.
* Unlike SQL, **in PostgreSQL, you don’t need to create a DLL to see the code what it is doing.**
* PostgreSQL supports dynamic actions whereas SQL doesn’t support them.

### **21.How is security ensured in PostgreSQL?**

PostgreSQL uses SSL connections to encrypt client or server communications so that security will be ensured.


### **22. What is the function of the Atomicity property in PostgreSQL?**

 Atomicity property ensures the successful completion of all the operations in a work unit.


### **23. What are the advantages of PostgreSQL?**


Some of the advantages of PostgreSQL are open-source DBMS, **community support, ACID compliance, diverse indexing techniques, full-text search, a variety of replication methods**, and diversified extension functions, etc.


### **24 What does Write-Ahead Logging do?**

The Write-Ahead Logging enhances database reliability by logging changes before any changes or updates are made to the database

### **25. What are some of the important data administration tools supported by PostgreSQL?**

Some of the important data administration tools supported by PostgreSQL are Psql,  Pgadmin, and Phppgadmin.

### **26. How can you store the binary data in PostgreSQL?**

We can store the binary data in PostgreSQL either by u**sing bytes or by using the large object feature.**

### **27. What is a non-clustered index?**

In a non-clustered index, the index rows order doesn’t match the order in actual data.

### **28. What is the purpose of table space in PostgreSQL?**

It is a location in the disk. In this, PostgreSQL stores the data files, which contain indices and tables, etc.

### **29 Are there any disadvantages with  PostgreSQL?**

Yes. There are a few disadvantages. Some of these include the following:

* It is slower than MySQL on the performance front.
* It doesn’t have the support of a good number of open source applications when compared to MySQL.
* Since it focuses more on compatibility, changes made to improve the speed need more work.

### **30 What does a token representation in a SQL Statement?**

**In a SQL Statement, a token represents an identifier, keyword, quoted identifier, special character symbol, or a constant.**
