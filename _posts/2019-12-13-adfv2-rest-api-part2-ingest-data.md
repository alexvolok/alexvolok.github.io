---
layout: post
title: Azure Data Factory and REST APIs - Ingesting Data
description: Azure Data Factory and REST APIs - Ingesting Data
comments: false
keywords: Azure Data Factory REST API COPY ACTIVITY
published: true 
---

In this blog post I would like to show of how to add and configure a Copy Activity which will ingest REST data and store it in a data lake. 
The REST API we are use is secured with OAUTH2 authentication. It requires a Bearer token to be obtained. Such technique was already demonstrated by me in a previous post of the series: <a href='/2019/adfv2-rest-api-part1-oauth2'>Azure Data Factory and REST APIs - Dealing with oauth2 authentication</a>

#### Prerequisites

The walkthrough information of this post does not cover creation of a storage account. In order to proceed, an Azure Data Lake gen2 account has to be created because it will be used as a sink destination later by a Copy activity.

### How Copy activity works

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


### Making Copy Activity working
To make a Copy activity working it is not enough just drop such object to a canvas, because Linked Services and Datasets are also requred. Following few steps covers these actions.

##### Step 1. Create and Configure Linked Services

Go to Connections page (1) and click on "+ New" to create linked services:


<img src="/assets/images/posts/adf-rest-p2/step1-1.png" alt="Step 1.1" />

**1. Linked Service to a Data Source:**

 - Data Store: REST
 - Name: LS_REST_EOL (1)
 - Base URL: https://start.exactonline.nl (2)
 - Authentication Type: Anonymous (3)
 - Server Certificate Validation: Disable (4)

<img src="/assets/images/posts/adf-rest-p2/step1-2.png" alt="Step 1.2" />

**2. Linked Service to a Data Sink:**

 - Data Store: Azure Data Lake Storage Gen2
 - Name: LS_ADLS_EOL (1)
 - Authentication Method: Account Key (2)
 - Account Selection Method: From Azure Subscription (3)
 - Choose right subscription and account name (4)
 
<img src="/assets/images/posts/adf-rest-p2/step1-3.png" alt="Step 1.3" />



##### Step 2. Create and Configure Datasets

In this step two datasets to be created. One per corresponding linked service. Therefore, source and sink datasets. To create them, click on a three dots that stands next to a Datasets in left pane and then choose "New dataset":

<img src="/assets/images/posts/adf-rest-p2/step2-1.png" alt="Step 2.1" />


 ###### 1. Create a source dataset:
  -	In a Data Store wizard search for a “REST”
  - In a General tab give a name: “DS_SRC_REST”
  - In a Connection tab choose a linked service: LS_REST_EOL and set a relative url: /api/read/Hosting.svc/ContractStatistics

<img src="/assets/images/posts/adf-rest-p2/step2-2.png" alt="Step 2.2" />

###### 2. Create a sink dataset:
  - In a Data Store wizard search for a keyword “Data" and choose "Data Lake gen2"
  - Choose the format type: Delimited Text
  - Set a name: “DS_SINK_ADLS”
  - Choose Linked Service "LS_ADLS_EOL"
 - File Path: The first field receives a storage container name. In my case it is "dwh". The second field is for a folder name -  “eol” and “contractstatitistics.csv” is a file name. Therefore, the full path is: ```/dwh/eol/contractstatitistics.csv```
  - Enable a checkbox: “First row as header”
  - Choose None for “Import Schema”

<img src="/assets/images/posts/adf-rest-p2/step2-3.png" alt="Step 2.3" />

##### Step 3. Create a Copy Activity

 1.	Drop a Copy activity on a canvas of a pipeline and give it a name "Ingest REST Data"
 2.	Connect exiting "Login" activity with a just added one: 

<img src="/assets/images/posts/adf-rest-p2/step3-1.png" alt="Step 3.1" />

 3.	On a Source Tab choose a dataset “DS_SRC_REST”. This will show new fields to adjust
     -	Request Method: “GET”
     -	Request Body: Should remain empty
     -	Add new request header:
         -	Name: ```Authorization```
         -	Value: ```Bearer @{activity('Login').output.access_token}``` 

     <img src="/assets/images/posts/adf-rest-p2/step3-2.png" alt="Step 3.2" />
 4.	On a Sink tab choose a dataset:
    -	Choose a data source “DS_DEST_ADLS”
    -	Remain columns can remain as is
    
    <img src="/assets/images/posts/adf-rest-p2/step3-3.png" alt="Step 3.3" />

##### Step 4. Run a test execution







