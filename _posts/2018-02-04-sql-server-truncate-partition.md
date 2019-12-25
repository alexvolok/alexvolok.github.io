---
layout: post
title: TRUNCATE PARTITION in SQL Server
description: TRUNCATE partition in SQL Server
comments: false
keywords: TRUNCATE partition in SQL Server
published: true 
---

`TRUNCATE TABLE` removes all rows from a table, but the table structure and its columns, constraints, indexes, and so on remain. That is what the documentation says and that is what we daily use in our scripts, jobs and SSIS packages. Starting with SQL Server 2016 such operation can be done also on a partition level giving extra flexibility and convenience.

#### Directly to an example:

Partition truncation in SQL Server 2016 as simple as following line of code:

```sql
 --SQL Server Execution Times: CPU time = 0 ms,  elapsed time = 324 ms.
TRUNCATE TABLE dbo.PartitionedTable WITH (PARTITIONS (1, 6 TO 10));
```

The command has removed partitions: 1, 6, 7, 8, 9, 10. Each one held one million rows,  therefore, 6 000 000 rows removed in a 324 ms. This is amazing result which outperform `PARTITION SWITCHING` timings.

Behind of the hood, `TRUNCATE` is a DDL command and this fact is a reason of superior performance of rows/pages release. However, it also leads to requirement of the minimum permission level on an object  – `ALTER TABLE`.

Another necessary part of the command is the list of partition ids to remove corresponding segment. Partition Id can be obtained multiple ways. As example, via `$PARTITION.partition` function call. Following query has been executed before and after truncation:

```sql
SELECT 
	$PARTITION.[PartitioningByRowID] (RowId) AS PartitionID
,	COUNT(*) AS [RowCount]
,	MIN(RowId) AS MinRowID
,	MAX(RowId) AS MaxRowID
FROM dbo.PartitionedTable
GROUP BY $PARTITION.[PartitioningByRowID] (RowId)
```

Alternative way to get partition ids and metadata is by querying querying dynamic management object – [sys.dm_db_partition_stats][1].


#### Test the feature in own sandbox?

Following script has necessary parts to reproduce this operation on your side:

```sql
-- step 1: build partition function and partition scheme
CREATE PARTITION FUNCTION [PartitioningByRowID] (int)
AS RANGE RIGHT FOR VALUES 
(1000000, 2000000, 3000000,4000000, 5000000
, 6000000, 7000000, 8000000, 9000000,10000000);

CREATE PARTITION SCHEME [PartitionByRowID]
AS PARTITION [PartitioningByRowID]
TO ([PRIMARY], [PRIMARY], [PRIMARY], [PRIMARY], [PRIMARY]
, [PRIMARY], [PRIMARY], [PRIMARY], [PRIMARY], [PRIMARY], [PRIMARY]);

-- step 2: create partitioned table
CREATE TABLE [dbo].[PartitionedTable](
	[RowId] [INT] NOT  NULL,
	[DataFiller] [char](500) NULL,
	[DateInserted] [datetime] NOT NULL
) ON [PartitionByRowID](RowID)

GO

-- step 3: build a table
INSERT dbo.PartitionedTable
SELECT TOP 10000000 
	s3.number * 1000000  +  s2.number * 1000 + s1.number  AS RowId
,	CAST(replicate ('a',500) AS CHAR(500)) AS  DataFiller 
,	GETDATE() AS DateInserted
FROM master..spt_values s1
CROSS JOIN master..spt_values s2 
CROSS JOIN master..spt_values s3
WHERE s1.number BETWEEN 0 AND 999 AND s1.type = 'P'
AND s2.number BETWEEN 0 AND 999 AND s2.type = 'P'
AND s3.number BETWEEN 0 AND 9 AND s3.type = 'P'
ORDER BY 1

-- step 4: get number of rows per partition
SELECT 
	$PARTITION.[PartitioningByRowID] (RowId) AS PartitionID
,	COUNT(*) AS [RowCount]
,	MIN(RowId) AS MinRowID
,	MAX(RowId) AS MaxRowID
FROM dbo.PartitionedTable
GROUP BY $PARTITION.[PartitioningByRowID] (RowId)

SET STATISTICS IO, TIME ON 

-- step 5: perform partition truncation
 --SQL Server Execution Times: CPU time = 0 ms,  elapsed time = 324 ms.
TRUNCATE TABLE dbo.PartitionedTable
WITH (PARTITIONS (1, 6 TO 10));
```


[1]:(https://docs.microsoft.com/en-us/sql/relational-databases/system-dynamic-management-views/sys-dm-db-partition-stats-transact-sql?view=sql-server-2017)