---
layout: post
title: Azure Data Factory and REST APIs - Ingesting Data into a Data Lake
description: Azure Data Factory and REST APIs - Ingesting Data into a Data Lake
comments: false
keywords: Azure Data Factory REST API COPY ACTIVITY
published: true 
---

In this blog post I would like to show of how to add and configure a Copy Activity which will ingest REST data and store it in a data lake. 
The REST API we are use is secured with OAUTH2 authentication. It requires a Bearer token to be obtained. Such technique was already demonstrated by me in a previous post of the series: <a href='/2019/adfv2-rest-api-part1-oauth2'>Azure Data Factory and REST APIs - Dealing with oauth2 authentication</a>

#### Prerequisites

The walkthrough information of this post does not cover creation of a storage account. In order to proceed, an Azure Data Lake gen2 account has to be created because it will be used as a sink destination later by a Copy activity.

#### How Copy activity works

To use the Copy activity in Azure Data Factory, following steps to be done:
 -	Create linked services for the source data store and the sink data store
 -	Create datasets for the source and sink
 -	Create a pipeline with the Copy activity

Copy activity uses input dataset to fetch data from a source linked service and copies it using an Output Dataset to a sink linked service

This process can be illustrated as a visual flow:
 
<img src="/assets/images/posts/adf-rest-p2/copy_activity_example.png" alt="Step 0" />


In the example picture two linked services created: Amazon S3 and Azure Storage
They act like connection managers in SSIS.

The next abstraction is a Dataset. The Dataset is a named view of data that simply points or references the data that has to bed used in activities as inputs and outputs.

And a final piece – Copy activity. It is a generic task that copies data among various data stores located on-premises and in the cloud.







<br />
<br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br />


#### Copy Activity

##### Step 1. Configure Linked Services

Go to Connections tab and create two linked services:

eolREST

   -	Type: REST
   - Name: eolREST
   - Base URL: https://start.exactonline.es
   - Authentication Type: Anonymous
  - Server Certificate Validation: Disable

Adlsgen2

i.	Type: Azure Data Lake Storage Gen2
ii.	Name: eolADLS
iii.	Authentication Method: Account Key
iv.	Account Selection Method: From Azure Subscription
v.	Choose right subscription and account name
vi.	Click Apply to save a linked Services
2)	Drop Copy Activity and name it “Ingest REST Data”
3)	Open “Source” tab and click on “New” button

##### Step 2. Configure Datasets

1)	Configure a Source Dataset
a.	Click on a plus and choose “New Dataset”
b.	Search “REST”
c.	In a General tab give a name “DS_SRC_REST”
d.	In a Connection tab choose a linked service: eolREST and add a relative url: /api/read/Hosting.svc/ContractStatistics
2)	Configure a Destination Dataset
a.	Search “Data Lake Gen2”
b.	Choose type: Delimited Text
c.	Set name: “DS_DEST_ADLS”
d.	Choose Linked Service “eolADLS”
e.	File Path: Enter a container name into a first field and “eol” into a second field a “contractstatitistics.csv” into a third one
f.	Enable a checkbox: “First row as header”
g.	Choose None for “Import Schema”

##### Step 3. Create a Copy Activity
1)	Drop a Copy Activity on a canvas of a pipeline
2)	Connect exiting Web activity with a new one:
 
3)	On a Source Tab choose a dataset “DS_SRC_REST”. This will show new fields to adjust
a.	Request Method: “GET”
b.	Request Body: Should remain empty
c.	Add new request header:
i.	Name: Authorization
ii.	Value: Bearer @{activity('Login').output.access_token} 
4)	On a Sink tab choose a dataset:
a.	Choose a data source “DS_DEST_ADLS”
b.	Remain columns can remain as is

##### Step 4. Run a test execution
