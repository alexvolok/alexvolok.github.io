---
layout: post
title: Azure Data Factory and REST APIs – Ingesting Data into a Data Lake
description: Azure Data Factory and REST APIs – Ingesting Data into a Data Lake
comments: false
keywords: Azure Data Factory REST API COPY ACTIVITY
published: true 
---

In this first post I am going to discuss how to apply oauth2 authentication to ingest REST APIs data.
OAUTH2 became a standard de facto in cloud and SaaS services, it used widely by Twitter, Microsoft Azure, Amazon. My example is based on EXACT Online API. The end-target of the blog series is to setup an entire pipeline which will ingest data from a REST API and load it to a data lake.

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
