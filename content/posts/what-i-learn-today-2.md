---
title: "What I Learn Today : PostgreSQL VACUUM"
date: 2024-06-22T15:17:50+07:00
draft: false
toc: true
images:
tags:
  - PostgreSQL
---

Vacuum is a PostgreSQL command that can be used for garbage collection and analyze a database. Vacuum will do garbage-collection by visiting the pages and track the dead rows. Dead Rows can came from deleted rows of copied of updated rows. When you update a row, PostgreSQL does not overwrite the existing row. Instead it creates a copy of the row. This is very useful if transaction failed the old row  remains intact.

After Vacuum track the dead rows it marks them as free space that will be available for new rows because the free space not returned back to system and shrink the table size. for example:
- Number of rows: 35 and Table size: 35MB
- Update all rows then table size become: 70MB (Update operation will copy rows)
- Vacuum the table
- Copied rows will be mark as free space but table size remains 70MB because the free space not returned back to system.
![PG Vacuum](../images/pg-vacuum.png)

Vacuum can be run using manual and automatic way. For manual way you can run using `VACUUM my_table` command (check `\h vacuum` for detail), and automatic way is running by PostgreSQL tool called autovacuum, autovacuum will executed based on configuration in `postgresql.conf`, below are several common autovacuum configration
- autovacuum_naptime. determine the delay between one autovacuum and the next in minute.
- autovacuum_vacuum_threshold. this determine how many updated/deleted rows must be exists to trigger autovacuum
- autovacuum_vacuum_scale_factor. this determine percentage of updated/deleted rows to trigger autovacuum. so if `autovacuum_vacuum_scale_factor` is `0.2` it mean `50%` if your total rows is `100` then updated/deleted rows must be `50` to trigger auto vacuum.
- autovacuum_vacuum_insert_threshold. this determine how much success `INSERT` operation must be executed in a database before able to trigger autovacuum.