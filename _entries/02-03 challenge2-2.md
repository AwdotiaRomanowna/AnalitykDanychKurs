---

sectionid: db
sectionclass: h2
parent-id: upandrunning
title: Multiversion Concurrency Control, MVCC
hide: false
published: true
---

This section describes the behavior of the PostgreSQL database system when two or more sessions try to access the same data at the same time. The goals in that situation are to allow efficient access for all sessions while maintaining strict data integrity. Every developer of database applications and DBA should be familiar with the topics covered in this chapter.


**Task Hints**
* We will use the portal to change the PostgreSQL parameters.
* Inspect the table size and see the impact of autovacuum.


### Tasks

#### Inspect and change server paramerters

You can list, show, and update configuration parameters for an Azure Database for PostgreSQL server through the Azure portal.

* Go to the Azure PostgreSQL resource

![Go PostgreSQL server parameteres](media/mvcc-postgres-server-params-2.png)

* Change Autovacuum server paramerter to off and **Save**

![Go PostgreSQL server parameteres](media/mvcc-postgres-autovacuum-off-3.png)


On **psql** or your faviourt IDE/GUI

* Check if Autovacuum settings

```sql
SHOW AUTOVACUUM ;

```
It should return this:

      postgres=> SHOW AUTOVACUUM ;
      autovacuum
      ------------
      off
      (1 row)

      postgres=>

Make sure that you have the *random_data* table

```sql
drop TABLE IF EXISTS random_data;

CREATE TABLE random_data AS
SELECT s                    AS first_column,
   md5(random()::TEXT)      AS second_column,
   md5((random()/2)::TEXT)  AS third_column
FROM generate_series(1,500000) s;
```

In case you wanted to see the path of the table:

```sql
SELECT pg_relation_filepath('random_data');
```
*Output*


      pg_relation_filepath
      ----------------------
      base/14417/16507
      (1 row)

      postgres=>

*Output*


      postgres=> SELECT pg_relation_filepath('random_data');
      pg_relation_filepath
      ----------------------
      base/14417/16507
      (1 row)


* Display the table size:

```sql
SELECT pg_relation_size('random_data');
```

*Output*

      postgres=> SELECT pg_relation_size('random_data');
      pg_relation_size
      ------------------
              50569216
      (1 row)


* Display the table size in MB:

```sql
SELECT pg_size_pretty(pg_relation_size('random_data'));
```
*Output*

      postgres=> SELECT pg_size_pretty(pg_relation_size('random_data'));
      pg_size_pretty
      ----------------
      48 MB
      (1 row)

      postgres=>


Inserting more records in the *random_data*
```sql
INSERT INTO random_data
SELECT s,
       md5(random() :: TEXT),
       md5((random() / 2) :: TEXT)
FROM generate_series(1, 1000000) s;

SELECT count(*) FROM random_data;
```

*Output*

      postgres=> INSERT INTO random_data
      postgres-> SELECT s,
      postgres->        md5(random() :: TEXT),
      postgres->        md5((random() / 2) :: TEXT)
      postgres-> FROM generate_series(1, 1000000) s;
      INSERT 0 1000000
      postgres=>
      postgres=> SELECT count(*) FROM random_data;
        count
      ---------
      1500000
      (1 row)  


Check the table size after the insersion

```sql

SELECT pg_size_pretty(pg_relation_size('random_data'));
```

*Output*

      postgres=> SELECT pg_size_pretty(pg_relation_size('random_data'));
      pg_size_pretty
      ----------------
      145 MB
      (1 row)

      postgres=>

Notice, that there is a signigcant increase in the table size. Now, delete all the recrods:

```sql
DELETE from random_data;
```

*Output*

      postgres=> DELETE from random_data;
      DELETE 1500000

Checking the table size after deletion of the records:
```sql
SELECT pg_size_pretty(pg_relation_size('random_data'));

```
*Output*

      postgres=> SELECT pg_size_pretty(pg_relation_size('random_data'));
      pg_size_pretty
      ----------------
      145 MB
      (1 row)  

You can see that the table size is still 145 MB, if we started the VACUUM process things would change:

```sql
VACUUM VERBOSE random_data;
```
*Output*

    postgres=> VACUUM VERBOSE random_data;
    INFO:  vacuuming "public.random_data"
    INFO:  "random_data": removed 1500000 row versions in 18519 pages
    INFO:  "random_data": found 1500000 removable, 0 nonremovable row versions in 18519 out of 18519 pages
    DETAIL:  0 dead row versions cannot be removed yet, oldest xmin: 595
    There were 0 unused item pointers.
    Skipped 0 pages due to buffer pins, 0 frozen pages.
    0 pages are entirely empty.
    CPU: user: 0.25 s, system: 0.00 s, elapsed: 0.23 s.
    INFO:  "random_data": truncated 18519 to 0 pages
    DETAIL:  CPU: user: 0.06 s, system: 0.01 s, elapsed: 0.04 s
    INFO:  vacuuming "pg_toast.pg_toast_16507"
    INFO:  index "pg_toast_16507_index" now contains 0 row versions in 1 pages
    DETAIL:  0 index row versions were removed.
    0 index pages have been deleted, 0 are currently reusable.
    CPU: user: 0.00 s, system: 0.00 s, elapsed: 0.00 s.
    INFO:  "pg_toast_16507": found 0 removable, 0 nonremovable row versions in 0 out of 0 pages
    DETAIL:  0 dead row versions cannot be removed yet, oldest xmin: 596
    There were 0 unused item pointers.
    Skipped 0 pages due to buffer pins, 0 frozen pages.
    0 pages are entirely empty.
    CPU: user: 0.00 s, system: 0.00 s, elapsed: 0.00 s.
    VACUUM
    postgres=> SELECT pg_size_pretty(pg_relation_size('random_data'));
    pg_size_pretty
    ----------------
    0 bytes
    (1 row)

* Check the table size after applying the Vacuum process:

```sql
 SELECT pg_size_pretty(pg_relation_size('random_data'));
```

*Output*

      postgres=> SELECT pg_size_pretty(pg_relation_size('random_data'));
      pg_size_pretty
      ----------------
      0 bytes
      (1 row)