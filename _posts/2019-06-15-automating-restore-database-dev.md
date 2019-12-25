---
layout: post
title: Automated Refresh of Databases on Development Servers
description: Automated Refresh of Databases on Development Servers
comments: false
keywords: SQL Server Restore Database Development Server
published: true 
---

#### Databases that have to be copied from a production server

It happens that databases in testing and especially acceptance environments needed to be refreshed by restoring backups from a production environment.  

The backup restore is seems like a trivial operation that can be done via SQL Server Management Studio wizard or via prepared T-SQL scripts. 

However, there are some complications that come into play during this:

 * Multiple databases to be restored at once
 * The data refresh requires both: the latest FULL and the latest DIFF to be restored one by one.
 * The outdated databases still have established sleeping sessions, they all have to be terminated before firing a restore
 * And the most important, developers want to perform such refreshes in an automated way, without manual scripting and without walking through all knobs of the wizard


#### A stored procedure to have the routine automated

Because those listed above complications happens on a frequent basis I've created a stored procedure  `[dbo].[uspRefreshDB]`. 

<!-- This procedure..

Add section with disclaimer.. -->


It has the following parameters:

| Parameter               | Description                                                                                                                   | Type    | Default Value | 
| ----------------------- |----------------------------------------------------------------------------------------------------------------------------- | --------:| -------------: | 
| @dbName  | A database name to refresh | sysname |  No default value         |         
| @FullBackupDir          | Path to full backups                           | nvarchar(500) | '\\\NAS\Backup\FULL'    |        
| @DiffBackupDir           | Path to DIFF backups                                            | nvarchar(500)     | '\\\NAS\Backup\DIFF'             |


#### Installation

The source code can be retrieved from the GitHub repository [github/avolok/scripts](https://github.com/avolok/scripts/tree/master/DBA) or by direct download: [[dbo].[uspRefreshDB]](https://raw.githubusercontent.com/avolok/scripts/master/DBA/uspRefreshDB.sql).



#### Examples

##### A.	Refreshing a single database

This example refreshes just a single database by restoring a latest full and then diff backup

```sql
    EXEC [dbo].[uspRefreshDB] 'DWH_ERR' 
```

It will create a snapshot with name *AdventureWorks2016_snapshot* . If such a snapshot already exists the procedure will results to an information message:

```sql

-- Restoring database: [DWH_ERR]. 
-- Full backup to be restored: NLC1DWHSQLCLS1_DWHDEPROD_DWH_ERR_Full_20190915_030700.BAK
-- Diff backup to be restored: NLC1DWHSQLCLS1_DWHDEPROD_DWH_ERR_Diff_20190914_043839.BAK




-- Command 1: ALTER DATABASE   [DWH_ERR] SET SINGLE_USER WITH ROLLBACK IMMEDIATE

--------------------------------------
	
-- Command 2: RESTORE DATABASE [DWH_ERR] FROM DISK = '\\backupshare\sql_backups\Regular\Full\NLC1DWHSQLCLS1\NLC1DWHSQLCLS1_DWHDEPROD_DWH_ERR_Full_20190915_030700.BAK' WITH NORECOVERY, REPLACE, STATS=20

-- 39 percent processed.
-- 59 percent processed.
-- 78 percent processed.
-- 98 percent processed.
-- 100 percent processed.
-- Processed 384 pages for database 'DWH_ERR', file 'DWH_ERR' on file 1.
-- Processed 136 pages for database 'DWH_ERR', file 'DWH_ERR_DATA' on file 1.
-- Processed 128 pages for database 'DWH_ERR', file 'DWH_ERR_INDEX' on file 1.
-- Processed 2 pages for database 'DWH_ERR', file 'DWH_ERR_LOG' on file 1.
-- RESTORE DATABASE successfully processed 650 pages in 0.249 seconds (20.364 MB/sec).

```


##### B.	Restoring Multiple Databases 

Folowing snippet executes restore commands sequentially:


```sql
EXEC [dbo].[uspRefreshDB] 'DWH_ERR'
EXEC [dbo].[uspRefreshDB] 'DWH_LOG'
```

