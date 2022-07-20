---
title: "ClickHouse S3 Engine and Cloudflare's R2 Object storage"
date: 2022-06-17T23:53:00+01:00
summary: "Trying ClickHouse S3 Engine with Cloudflare's R2 Object storage"
summaryImage: "clickhouse-db.png"
tags: ["cloudflare r2", "r2", "clickhouse"]
---


Cloudflare has recently launched R2 open beta. R2 is an object storage similary to S3.
In this blog post, we will create a ClickHouse table using S3 engine to store the data in Cloudflare R2.

**ClickHouse S3 table engine**
First, lets create a table with S3 engine. To create this table, we need to provide bucket web URL,
`access_key` and `secret_key`.

```
clickhouse-0.clickhouse.cluster.local :) CREATE TABLE s3_engine_table_v2 (name String, value UInt32) ENGINE=S3('https://account_id.r2.cloudflarestorage.com/my-bucket/test.csv', 'ACCESS_KEY', 'SECRET_KEY', 'CSV') SETTINGS input_format_with_names_use_header = 0;

CREATE TABLE s3_engine_table_v2
(
    `name` String,
    `value` UInt32
)
ENGINE = S3('https://account_id.r2.cloudflarestorage.com/my-bucket/test.csv', 'ACCESS_KEY', 'SECRET_KEY', 'CSV')
SETTINGS input_format_with_names_use_header = 0

Query id: e888ae76-3a44-4038-97bb-17b7c061ea1f

Ok.

0 rows in set. Elapsed: 0.007 sec.
```

**Data insertion**

Next insert data into it.

```
clickhouse-0.clickhouse.cluster.local :) INSERT INTO s3_engine_table_v2 VALUES ('one', 1), ('two', 2), ('three', 3);

INSERT INTO s3_engine_table_v2 FORMAT Values

Query id: fe4b8d96-06d1-45f4-a9d3-85ff99dc76b6

Ok.

3 rows in set. Elapsed: 1.757 sec.
```

**Query R2**

Query R2 to get the inserted data.

```
clickhouse-0.clickhouse.svc.cluster.local :) SELECT * FROM s3_engine_table_v2 LIMIT 2;

SELECT *
FROM s3_engine_table_v2
LIMIT 2

Query id: 59211c5b-fd15-4da6-8f4b-4c4363ee0f37

┌─name─┬─value─┐
│ one  │     1 │
│ two  │     2 │
└──────┴───────┘

2 rows in set. Elapsed: 1.149 sec.
```

If you are able to read the inserted keys succesfully, then you can as well verify by logging into dash.cloudflare.com R2 section.


