---
layout: post
title: Dynamic Data Masking
description: Dynamic Data Masking
comments: false
keywords: Dynamic Data Masking
published: false 
---

> disclaimer: image urls to be fixed

Dynamic data masking is a relativily new feature added in  SQL Server 2016. It helps prevent unauthorized access to sensitive data by giving customers possibility to mask sensitive information without need to change existing applications. Official documentation is pretty clear about the usage and can be a good beginning point. However, I would like to bring also some light into the aspects that are not covered in the manual.

#### Dynamic Data Masking in action

Despite the fact that I mentioned that this post is not a replacement of the official documentation, I still would like to reuse a script from that source for demonstration of the base functionality:

```sql
CREATE TABLE Membership
(
   MemberID int IDENTITY PRIMARY KEY,
   FirstName varchar(100) MASKED WITH (FUNCTION = 'partial(1,"XXXXXXX",0)') NULL,
   LastName varchar(100) NOT NULL,
   Phone# varchar(12) MASKED WITH (FUNCTION = 'default()') NULL,
   EMail varchar(100) MASKED WITH (FUNCTION = 'email()') NULL
);

INSERT Membership (FirstName, LastName, Phone#, EMail) 
VALUES 
('Roberto', 'Tamburello', '555.123.4567', 'RTamburello@contoso.com'),
('Janice', 'Galvin', '555.123.4568', 'JGalvin@contoso.com.co'),
('Zheng', 'Mu', '555.123.4569', 'ZMu@contoso.net');

SELECT * FROM Membership;
```

<01. Execution result. No data masking applied>

The results show that dynamic data masking is turned off. According to Microsoft this is default behavior for sysadmin logins and database owners:

```sql
CONTROL permission on the database includes both the ALTER ANY MASK and UNMASK permission
```

Therefore, to show the data masking, new user has to be created with a plain SELECT rights on a table:

```sql
CREATE USER TestUser WITHOUT LOGIN;
GRANT SELECT ON Membership TO TestUser;

EXECUTE AS USER = 'TestUser';
SELECT * FROM Membership;

REVERT;
```

This time data masking worked well and sensitive data concealed:

<02. Execution result. Data masking works as expected>

However, documentation warns that operations like CAST (COLUMN as VARCHAR(50))  will bypass the masking. Is this the only limitation? Seems, not and following section provides some examples.

 
Evaluation time surprises

In this section I compiled four simple test cases and results, which are not always were expected.
Check 1: Search in a masked column:

This check shows that data masking is not an obstacle for a searching in a masked data:

```sql
EXECUTE AS USER = 'TestUser';
SELECT * FROM Membership
WHERE email LIKE 'ZMU%'
```

<03. Check 01. Data masking doesn't prevent sensitive data exposure>

Check failed.
Check 2: Calculated columns on top of masked one:

In the following script, the masked column is a part of two computations: computed DDL column and in-line calculated column as the part SELECT statement. As theresult  – DDL based column remains unmasked, while inline SELECT based – masked. However, masking of the column [Email3] has different format that the [Email].

```sql
ALTER TABLE Membership ADD Email2 as Email + ' '
EXECUTE AS USER = 'TestUser';
GO
SELECT Email
, Email2
, Email + ' ' as Email3 
FROM Membership;

REVERT;
ALTER TABLE Membership DROP COLUMN Email2 
```

<04. Check 02. Data masking partially prevented sensitive data exposure
>

Check partially failed.
Check 3: SELECT INTO #Table

The check shows, that SELECT INTO does not take dynamic data masking into account:

```sql
EXECUTE AS USER = 'TestUser';
SELECT * INTO #Membership_Copy from Membership
SELECT * FROM #Membership_Copy 
REVERT;
```

<05. Check 03. Data masking doesn’t prevent sensitive data exposure again>

Check failed.

##### Check 4: INSERT #Table EXEC ..

According to the documentation,  the usage of dynamic data masking expects that end-user credentials have no direct SELECT permissions on the “masked” tables. Therefore, usage of the stored procedures is the obvious way to secure masking. Following check will test how output of the stored procedure can be stored in a temporary table:

```sql
CREATE PROC dbo.usp_GetMembership
AS
SELECT * FROM Membership
go
GRANT EXEC on dbo.usp_GetMembership to TestUSer
GO

EXECUTE AS USER = 'TestUser';

CREATE TABLE #Test
 (
	MemberID int,
	FirstName varchar(100),
	LastName varchar(100),
	Phone# varchar(12),
	EMail varchar(100)
)

INSERT #Test EXEC dbo.usp_GetMembership

SELECT * FROM #Test
```

<
06. Check 04. Data masking worked correctly>

The dynamic data masking prevents to get a unscrambled values via nested INSERT .. EXEC. 

Check passed!

#### Conclusions

Dynamic Data Masking is a helpful new feature in a toolset of database specialist, which require some extra efforts with a securing data access to a sensitive data. However, since data is masked just before being returned to the user, the feature cannot be considered as replacement of the transparent data or column level  types of encryption.