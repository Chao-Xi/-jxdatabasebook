# **2 常用基础命令介绍**


```
\?
\l
\d
\d t1
\c dbname
\help
\help create user
\du
\x
```


```
postgres=# \?
General
  \copyright             show PostgreSQL usage and distribution terms
  \crosstabview [COLUMNS] execute query and display results in crosstab
  \errverbose            show most recent error message at maximum verbosity
  \g [FILE] or ;         execute query (and send results to file or |pipe)
  \gdesc                 describe result of query, without executing it
  \gexec                 execute query, then execute each value in its result
  \gset [PREFIX]         execute query and store results in psql variables
  \gx [FILE]             as \g, but forces expanded output mode
  \q                     quit psql
  \watch [SEC]           execute query every SEC seconds
...
```

```
postgres=# \h
Available help:
  ABORT                            ALTER ROLE                       CHECKPOINT                       CREATE OPERATOR                  CREATE USER                      DROP MATERIALIZED VIEW           DROP TRIGGER                     RESET
  ALTER AGGREGATE                  ALTER ROUTINE                    CLOSE                            CREATE OPERATOR CLASS            CREATE USER MAPPING              DROP OPERATOR                    DROP TYPE                        REVOKE
```

**show databases;**

```
\l
```

```
postgres=# \l
                                 List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges
-----------+----------+----------+------------+------------+-----------------------
 jam       | admin    | UTF8     | en_US.utf8 | en_US.utf8 |
 pg1       | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
(5 rows)
```


**Use database**


```
\c pg1
You are now connected to database "pg1" as user "postgres".
pg1=# \d
        List of relations
 Schema | Name | Type  |  Owner
--------+------+-------+----------
 public | t1   | table | postgres
(1 row)

pg1=# \dt
        List of relations
 Schema | Name | Type  |  Owner
--------+------+-------+----------
 public | t1   | table | postgres
(1 row)

pg1=# \d t1
                 Table "public.t1"
 Column |  Type   | Collation | Nullable | Default
--------+---------+-----------+----------+---------
 id     | integer |           |          |

pg1=# \dt t1
        List of relations
 Schema | Name | Type  |  Owner
--------+------+-------+----------
 public | t1   | table | postgres
(1 row)

```

```
pg1=# select * from pg_tables;
     schemaname     |        tablename        | tableowner | tablespace | hasindexes | hasrules | hastriggers | rowsecurity
--------------------+-------------------------+------------+------------+------------+----------+-------------+-------------
 public             | t1                      | postgres   |            | f          | f        | f           | f
 pg_catalog         | pg_statistic            | postgres   |            | t          | f        | f           | f
 pg_catalog         | pg_type                 | postgres   |            | t          | f        | f           | f
 pg_catalog         | pg_foreign_server       | postgres   |            | t          | f        | f           | f
```

```
pg1=# select  * from pg_tables where schemaname != 'pg_catalog';
     schemaname     |        tablename        | tableowner | tablespace | hasindexes | hasrules | hastriggers | rowsecurity
--------------------+-------------------------+------------+------------+------------+----------+-------------+-------------
 public             | t1                      | postgres   |            | f          | f        | f           | f
 information_schema | sql_packages            | postgres   |            | f          | f        | f           | f
 information_schema | sql_features            | postgres   |            | f          | f        | f           | f
 information_schema | sql_implementation_info | postgres   |            | f          | f        | f           | f
 information_schema | sql_parts               | postgres   |            | f          | f        | f           | f
 information_schema | sql_languages           | postgres   |            | f          | f        | f           | f
 information_schema | sql_sizing              | postgres   |            | f          | f        | f           | f
 information_schema | sql_sizing_profiles     | postgres   |            | f          | f        | f           | f
```

```
pg1=# select  * from pg_tables where schemaname not in ('pg_catalog','information_schema');
 schemaname | tablename | tableowner | tablespace | hasindexes | hasrules | hastriggers | rowsecurity
------------+-----------+------------+------------+------------+----------+-------------+-------------
 public     | t1        | postgres   |            | f          | f        | f           | f
(1 row)
```

```
pg1=# \x
Expanded display is on.
pg1=# select  * from pg_tables where schemaname not in ('pg_catalog','information_schema');
-[ RECORD 1 ]---------
schemaname  | public
tablename   | t1
tableowner  | postgres
tablespace  |
hasindexes  | f
hasrules    | f
hastriggers | f
rowsecurity | f
```

