---
layout: post
title: How To Simplify Database Snapshots Creation in SQL Server
description: How To Simplify Database Snapshots Creation in SQL Server
comments: false
keywords: SQL Server Database Snapshots
published: true 
---

**Database Snapshots** is a powerful feature that can be used for quick reverts of the database to the state as it was in when a given database snapshot was created, as well as data and schema comparison between a source database and a snapshot.

While this feature was shipped in SQL Server 2005 it was available only in the enterprise edition. In release SQL Server 2016 SP1 Microsoft made a generous step and unlocked plenty of such enterprise-grade features. Since that snapshots become ready to use in all editions, including Express.  



> **Warning:** Database snapshots are dependent on the source database. Therefore, database snapshots are not to substitute  your primary **backup and restore strategy**. Performing all your scheduled backups remains essential.

### Problem and solution

While this feature has a clear T-SQL syntax SSMS has no wizard or GUI for it, so creation of multiple snapshots requires some tedious coding like:

```sql
CREATE DATABASE [AdventureWorks2016_snapshot] 
ON (
    NAME = [AdventureWorks2016_Data]
,   FILENAME ='H:\SQL_Data\Data\AdventureWorks2016_Data.mdf.snapshot'
) 
AS SNAPSHOT OF [AdventureWorks2016];
```



Because of absence of UI and other helpers I created a stored procedure `[dbo].[uspCreateSnapshot]`. 

This procedure has the following parameters:

| Parameter               | Description                                                                                                                   | Type    | Default Value | 
| ----------------------- |----------------------------------------------------------------------------------------------------------------------------- | --------:| -------------: | 
| @SourceDBSearchPattern  | Determines a list of databases to process, it uses LIKE pattern seach. When value set to All then all user databases selected | sysname | 'all'         |         
| @SnashotSuffix          | Snapshot name suffix, the full snapshot name will have a value: databasename_suffixname                                       | sysname | 'snapshot'    |        
| @DropIfExists           | Remove old snapshot on the same source database if  snapshot name is also the same                                            | bit     | 0             |
| @Debug                  | Prints output as SQL Script, but does not execute it                                                                          | bit     | 0             |


### Installation

The source code can be retrieved from the GitHub repository [github/avolok/scripts](https://github.com/avolok/scripts/tree/master/DBA) or by direct download: [[dbo].[uspCreateSnapshot]](https://raw.githubusercontent.com/avolok/scripts/master/DBA/uspCreateSnapshot.sql).



### Examples

#### A.	Creating a snapshot on a single database

This example creates a snapshot of a single database using default values. 

    EXEC [dbo].[uspCreateSnapshot] 'AdventureWorks2016'

It will create a snapshot with name *AdventureWorks2016_snapshot* . If such a snapshot already exists the procedure will results to an information message:

```sql
-- Processing database: [AdventureWorks2016]
-- Snapshot [AdventureWorks2016_Snapshot] already created on database [AdventureWorks2016], nothing more to do
```


#### B.	Creating snapshots on multiple databases selected by a search pattern 

This example creates a snapshot of every database that matches `Name LIKE 'DBName0%'`  

```sql
EXEC [dbo].[uspCreateSnapshot] @SourceDBSearchPattern = 'DBName0%', 
                               @SnashotSuffix = 'ss',
                               @DropIfExists = 1                
```
Because @SnapshotSuffix is set to `'ss'` snapshots will be named as *databasename_ss*. If snapshot with such name already exists, it will be dropped firstly and a new one will be created


#### C.	Generating T-SQL for snapshot creation
This example generates T-SQL script for snapshots creation, however, it does not execute it.

```sql
EXEC [dbo].[uspCreateSnapshot] @SourceDBSearchPattern = 'AdventureWorks2016',
                               @Debug = 1                                
```

Results to output:

```sql
-- Processing database: [AdventureWorks2016]

-- Executing SQL:
CREATE DATABASE [AdventureWorks2016_Snapshot] 
ON (
    NAME = [AdventureWorks2016_Data], 
    FILENAME ='H:\SQL_Data\Data\AdventureWorks2016_Data.mdf.Snapshot'
)
AS SNAPSHOT OF [AdventureWorks2016];	
```

Such option can be useful for a generation of the code that has to be  validated or reused
