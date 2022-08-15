---
title: "ClickHouse system.parts tables"
hideLastModified: true
summary: "ClickHouse system.parts Tables"
summaryImage: "clickhouse-db.png"
tags: ["cloudflare", "pages", "pages.dev", "clickhouse"]
keywords: ["pages", "blog"]
---
System tables

ClickHouse system tables store system information in virtual tables.\
Virtual tables do not store any data on disk, it is in memory and\
these tables are “read only”.

The Systems table provides great visibility into the system.\
The system table queries can be exported as metrics and alerted on or\
it can be used to visualize in the dashboards to correlate with other events.

system.parts tables can be used to understand how much storage space would be required\
before migrating data from some other datastore to ClickHouse or to understand \
the schema storage optimization gains.

Create a table with schema, insert data into the table. Query the system.parts table:

Example query:
```
SELECT
    table,
    formatReadableSize(sum(bytes)) AS replicaSize,
    sum(rows) AS replicaRows,
    formatReadableSize(sum(bytes) / sum(rows)) AS perRow
FROM system.parts
GROUP BY table
ORDER BY sum(bytes) DESC
FORMAT Vertical

Table: 		`tablename`
replicaSize:	1.29 GiB
replicaRows:	7465000
perRow:	185.96 B
```

Storage estimation example:

With current schema, it takes around 190 B to store one row\
If the messages received are 250K requests per second then\
The total space required to store 250K messages = 190 * 250,000 B = 47.5 MB\
For 1 replica, total space required to store 1 day of logs = 47.5 * 60 * 60 * 24 MB = 4.1 TB\
For 3 replicas, total space required to store 1 day of logs = 4.1 * 3 = 12.3 TB

You may perform optimizations or changes to the schema and repeat the query.

Materialized views can be created on top of system tables to make it more efficient.\
Creating a distributed table on top of these materialized view tables would be very\
useful in tracking queries across shards.
