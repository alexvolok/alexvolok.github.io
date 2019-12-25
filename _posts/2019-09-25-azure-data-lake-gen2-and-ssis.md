---
layout: post
title: Azure Data Lake Gen2 and SQL Server Integration Services
description: Azure Data Lake Gen2 and SQL Server Integration Services
comments: false
keywords: Azure Data Lake SSIS
published: true 
---

#### How it used to be

"Azure Data Lake Generation 2" (or ADLS gen2) is the newest cloud data lake offering from a Microsoft. It supposes to bring the best of two worlds together: excelent performance and redundancy of a blob storage and secure filesystem capabilities of a data lake. 

However, for some time the ADLS gen2 had a lack of support by on-premise tools, like SQL Server and SSIS. It has a support of a Databricks and Azure Data Factory. But shops that are heavily relied on SSIS or Azure Data Lake Analytics (U-SQL) still have to stay with a previous generation of the service.

James Serra has an excelent article about limitations: [Ways to access data in ADLS Gen2][1]

#### New release of Azure Feature Pack

The situation changed with an August 2019 release of Azure Feature Pack for Integration Services (SSIS). There is also a blog post with anouncement: [Azure Feature Pack 1.14.0 released with Azure Data Lake Storage Gen2 Support][2], however this release didn't receive a wide spread and we didn't hear loud fanfares. 

Release note describes major updates:


> - **Azure Storage Connection Manager:** The improved Azure Storage Connection Manager now supports both Blob Storage and Data Lake Storage Gen2 services of Azure Storage. We have also added a service principal authentication method on top of the access key one. When you run your packages on Azure-SSIS IR, you have cloud-only security features at your disposal. For example, when you use Execute SSIS Package activities in Azure Data Factory (ADF) pipelines, you can store the sensitive values for account/application keys and application/tenant IDs as secrets in Azure Key Vault (AKV). On top of that, you can do away with sensitive values altogether and authenticate using the managed identity of hosting ADF.
> - **Flexible File Task:** This newly added task is designed to support different kinds of file operations on various supported storage services.
Currently, only Copy operation is provided, and the supported storage services for source/destination are local file system, Blob Storage, and ADLS Gen2.
We expect to support more operations and storage services in the future.
> - **Flexible File Source/Destination:** Like Flexible File Task, the source/destination is designed to support various storage services.
Currently, Blob Storage and ADLS Gen2 are supported.
> - **Foreach Data Lake Storage Gen2 File Enumerator:** This newly added foreach enumerator enables you to list files in a specified folder on ADLS Gen2.



#### Test run: Load parquet file directly to a Data Lake using SSIS dataflow


The goal is to load data directly from a dataflow, using SSIS as the ETL tool. To achieve such result the test run split into four logical steps:

 - Step 1: Configure a connection manager

 - Step 2: Configure a dataflow destination to use a flexible task

 - Step 3: Perform a test run

 - Step 4: Check a data lake for a file presence




##### Step 1: Configure a connection manager

The very first step is to create a connection to get access to Azure storage account. Fields  "Account name" and "Environment" are shared across all authenication options.

Authentication can be done using: 
 - Service Key
 - Service Principals.

Service Key is the simplest option however it grants full access to a storage account.

In scenarios of shared usage and fine-grained ACL permissions on a folder level better to choose an Azure AD Service Principal as a credential to secure the access.

<img src="/assets/images/posts/ssis-adls/step1_1.jpg" alt="Step 1" />

The button "Test connection" will show a success message in case if credential has full access to a storage account. In case if credential can access only some certain subfolder the button will keep prompting about connectivity error, however the dataflow will still able to write data correctly to such directory in a data lake.


##### Step 2: Configure a dataflow destination to use a flexible task

Another logical step: configure a destination task: 
<img src="/assets/images/posts/ssis-adls/step1_2.jpg" alt="Step 2" />

There are four different formats available:
 - CSV
 - AVRO
 - Parquet
 - ORC

 The last two require java libraries installed on a SSIS server.


##### Step 3: Perform a test run

SSIS starts writing data to a data lake the same way as it written to a local disk storage
<img src="/assets/images/posts/ssis-adls/step1_3.jpg" alt="Step 3" />


##### Step 4: Check a data lake for a file presence

Azure Storage Explorer shows that a parquet file was generated:

<img src="/assets/images/posts/ssis-adls/step1_4.jpg" alt="Step 4" />




[1]:https://www.jamesserra.com/archive/2019/09/ways-to-access-data-in-adls-gen2/

[2]:https://techcommunity.microsoft.com/t5/SQL-Server-Integration-Services/Azure-Feature-Pack-1-14-0-released-with-Azure-Data-Lake-Storage/ba-p/830672


