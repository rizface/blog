---
title: "What I Learn Today: PostgreSQL Transaction & Locking Part 1"
date: 2024-06-17T19:00:09+07:00
draft: false
toc: true
images:
tags:
  - PostgreSQL
---

## Everything Is A Transaction
Today i learned that everything in PostgreSQL runs inside a transaction. Even if you just send a simple query like `SELECT * FROM table` without explicitly beginning a transaction it is executed within a transaction under the hood. This approach leverages the concept of autocommit, where each individual statement is treated as a single transaction if no explicit transaction commands are given.
## Behavior Of CURRENT_TIMESTAMP & NOW()
 I was very supprised to learn that `CURRENT_TIMESTAMP` & `NOW()` do not return the current timestamp. I discovered this behavior few days a go when my team encountered a bug related to data sorting based on `created_at` column. The issue arose because two data had identically `created_at` value. 
 
 After look up to a process where data is inserted i know that the process is executed in long-running transaction because it insert `N` data inside a loop.
```sql
BEGIN;
  FOREACH values as v
    INSERT INTO x (column) values (v)
  END
COMMIT;
```
The default value for `created_at` is `CURRENT_TIMESTAMP`, which means if no value provided for the column, it uses `CURRENT_TIMESTAMP`. However, `CURRENT_TIMESTAMP` returns the time when transaction started, causing multiple data to have the same `created_at`. Thus making it impossible to sort correctly.
## Read Only Transaction
Read Only Transaction is a transaction mode where only read operations are allowed. This mode is useful to ensuring that the data is not modified during transaction.
```sql
BEGIN TRANSACTION READ ONLY;
 -- operations
END
```
## Savepoint
Savepoint is like a checkpoint in a game you can rollback to spesific savepoint and start from there onward. This is useful when you encountered error in your transaction and need to rollback to spesific point. Savepoint is short-lived state which means when you commit or abort the transaction you are unable to access savepoint after that.
```sql
BEGIN;

SELECT 1+1;
SELECT 2+2;
SAVEPOINT one;

SELECT 3+3;
SELECT 4+4;
SAVEPOINT two;


SELECT 5+5;
SELECT 6+6;
SAVEPOINT three;

SELECT 1/0; -- this will error

ROLLBACK TO SAVEPOINT three;
END
```
When you `ROLLBACK TO SAVEPOINT three` you still able to ROLLBACK again to savepoint one or two but if you `ROLLBACK TO SAVEPOINT two` you unable to `ROLLBACK` to savepoint three, because at that savepoint you are not execute `SAVEPOINT three` yet.
## DDLs Transaction
DDL (Data Definition Language) is a command that use to define a database/table/schema/data type. Supprisingly DDL can also executed within a transaction. This is useful when you write long DDL and need to ensure all command is successfully executed.
## MVCC (Multi Version Concurrency Control)
MVCC (Multi Version Concurrency Control) is a mechanism to crete snapshot for each transaction session. this is useful to ensure many operations read same data wont block each other because they read they own snapshot, the snapshot always be updated when other transaction is committed as long your transaction isolation mode is not `REPEATABLE READ`.
## SELECT FOR UPDATE
`SELECT FOR UPDATE` is read operations that locks the row it reads. This very useful when you want to select a data and modify it at application level before update the data and this is very useful to prevent race condition when select & update data because only one transaction session can hold the locks at one time.
```sql
BEGIN;
  SELECT * FROM tables WHERE id = 1 FOR UPDATE;
END
```
<!-- ## SELECT FOR SHARE
## Check For Locks
## NOWAIT
## SKIP LOCKED
## Isolation Level
## Deadlock
## Advisory Locks --> -->