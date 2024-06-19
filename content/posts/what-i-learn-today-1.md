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
## SELECT FOR SHARE
`SELECT FOR SHARE` is read operations that locks what it reads different from `SELECT FOR UPDATE` where only one transaction session can hold the lock. `SELECT FOR SHARE` shares the lock among transaction session that run `SELECT FOR SHARE` command too but will prevent `UPDATE`, `DELETE`, and `SELECT FOR SHARE` operations from acquire the lock. This very useful to ensuring that the data is not modified while you read it.

```sql
BEGIN;
  SELECT * FROM tables WHERE id = x FOR SHARE
END;

BEGIN;
  SELECT * FROM tables WHERE id = x FOR SHARE -- this query will success and get the row
END;

BEGIN;
  SELECT * FROM tables WHERE id = x FOR UPDATE -- this will waiting untill all select for share query release the lock
END;
```
<!-- ## Check For Locks -->
## NOWAIT
NOWAIT is an option when execute `SELECT FOR UPDATE` when use this option the query will return error when lock is not available. This is very useful when you wont wait for lock to be available.

```sql
-- Imagine Trx 1 and 2 running simultaneously

-- Trx 1
BEGIN;
  SELECT * FROM tables WHERE id = 1 FOR UPDATE NOWAIT;
END;

-- Trx 2
BEGIN;
  SELECT * FROM tables WHERE id = 2 FOR UPDATE NOWAIT;
END;
```
Trx 2 wont wait for lock to be available and will return an error.

## SKIP LOCKED
SKIP LOCKED is option used with `SELECT FOR UPDATE` to avoid waiting rows to become available. It skips locked rows and tries to lock next available rows.
```sql
-- imagine you have 2 seat in a air plane, and 3 users want to book a seat. the request will run simultaneously

-- Trx 1
BEGIN;
  SELECT * FROM seats LIMIT 1 FOR UPDATE SKIP LOCKED; -- this will acquire lock for 1 one
END;

-- Trx 2
BEGIN;
  SELECT * FROM seats LIMIT 1 FOR UPDATE SKIP LOCKED; -- this will skip locked row, and find another one
END;

-- Trx 3
BEGIN;
  SELECT * FROM seats LIMIT 1 FOR UPDATE SKIP LOCKED; -- this will skip locked row, and find another one
END;
```
When the session tries to find available lock and not found, the query will return 0 rows and not considered as error.
## Isolation Level
Isolation Level is the way PostgreSQL isolates one transaction from others, as we know the transaction will always get latest update of tables even if other transaction session commit the changes. This is very useful when you need to process a constrant snapshot, the isolation level consists of 3 types:
- READ COMMIITED, this is the default isoaltion level postgresql uses, transaction started with this isolation level will always get a latest update of tables even if the changes committed by another transaction session.
```sql
-- Trx 1 and 2 will run simultaneously
-- imagine you have 2 rows in tables

-- Trx 1
BEGIN;
  SELECT COUNT(id) FROM tables; -- will return 2
  -- few moment before run second query (deletion of id 1 in proggress)
  SELECT COUNT(id) FROM tables; -- will return 1
END;

BEGIN;
  DELETE FROM tables WHERE id = 1;
  SELECT COUNT(id) FROM tables; -- will return 1
END;
```
The second query in Trx 1 will return 1 because it get updated snapshot everytime changes happens in database

- REPEATABLE READ, when start a transaction using this level of isolation the transactiol will get constant snapshot and wont get latest update of snapshot when changes happens.
```sql
-- Trx 1 and 2 will run simultaneously
-- imagine you have 2 rows in tables

-- Trx 1
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
  SELECT COUNT(id) FROM tables; -- will return 2
  -- few moment before run second query (deletion of id 1 in proggress)
  SELECT COUNT(id) FROM tables; -- will return 2
END;

BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
  DELETE FROM tables WHERE id = 1;
  SELECT COUNT(id) FROM tables; -- will return 1
END;
```
second query in Trx 1 also return 2 because the transaction dont get an updated snapshot.

- READ UNCOMMITED, when a transaction is started using this isolation level postgresql silenty use `READ COMMITTED` level.

## Deadlock
Deadlock is a condition where more than 1 transactions waiting for each other to release a locks.
```sql
-- Trx 1
BEGIN;
  SELECT * FROM tables WHERE id = 1 FOR UPDATE;
  SELECT * FROM tables WHERE id = 2 FOR UPDATE;
END;

-- Trx 2
BEGIN;
  SELECT * FROM tables WHERE id = 2 FOR UPDATE;
  SELECT * FROM tables WHERE id = 1 FOR UPDATE;
END;
```
Imagine those transactions run simultaneously they will wait for each other to release locks which never happens, this condition will produced a deadlock error.

## Advisory Locks
Advisory Lock difference from Transaction Lock. Transaction Lock is locks a table rows while Advisory Lock is locks a number, locked number wont be released with END/ROLLBACK/COMMIT command because it is not part of transaction session. This is very useful when you want to implement election to determine which node going to be a leader.

```sql
SELECT pg_advisory_lock(1); -- will lock `1`
SELECT pg_advisory_lock(1); -- will waiting for others to release the lock for `1`
select pg_try_advisory_lock(1); -- will skip if `1` already locked by others
```