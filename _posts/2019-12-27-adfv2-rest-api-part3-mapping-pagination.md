---
layout: post
title: Azure Data Factory and REST APIs - Mapping and Pagination
description: Azure Data Factory and REST APIs - Mapping and Pagination
comments: true
keywords: Azure Data Factory REST API Copy Activity Mapping Paging
published: true 
---


In this blog post I would like to put a light on a mapping and then pagination of a Copy activity which are often required for the ingestion of REST data. 

Mapping is optional for structured data, like databases, parquet or csv files, because the incoming dataset contains an inherited structure, so the Data Factory is smart enough to set it as a default mapping on a fly. 

However, to ingest hierarchical data like JSON or REST and then load it in as a tabular format an explicit schema mapping is required. In the previous blog post – <a href="/2019/adfv2-rest-api-part2-copy-activity">Azure Data Factory and REST APIs - Setting up a Copy activity</a> – such mapping was not provided yet explicitly. As a result, ADF was not able to write an incoming data streams in a tabular form and created an empty csv file. This post will fill such gap. Also, it will cover pagination which is also a common thing for REST APIs.


#### Prerequisites

 1.	REST API Data source. I use ExactOnline SaaS service in this post as an example.
 2.	Azure Storage account.



### Mapping of hierarchical data 
Mapping can be configured in a few various ways:
 -	Fully manual, by adding mapping columns one by one
 -	Automatic, by importing schema from a data source

In this post I will describe a second approach – *import of schema*. This is a straightforward action and can be achieved by opening a tab “Mapping” and clicking on “Import Schemas”, just like on a picture:
 
<img src="/assets/images/posts/adf-rest-p3/step1-01.png" alt="Step 1-1" /> 
<br /><br />

However, because the current example uses oauth2, there is one prerequisite that must be fulfilled - bearer token to be passed on a design time. This token is necessary to get authentication during schema import, just because Azure Data Factory makes a call to API and load sample data for further parsing and extraction of the schema. Because of this, following window will popup and it expects token to be entered:
 
<img src="/assets/images/posts/adf-rest-p3/step1-02.png" alt="Step 1-1" /> 
<br /><br />


#### Step 1: Retrieve a bearer token
The access token can be retrieved by running a debug execution (1) and opening an output window (2) of a Login activity.


<img src="/assets/images/posts/adf-rest-p3/step1-1.png" alt="Step 1-1" />

#### Step 2: Import schema

When the token is copied to a clipboard click on “Import schemas” again, paste the token in a popup window.
As the result, the schema of a hierarchical format is imported. In the current example it contains:

 1.	A Root element – ```d``` (2)
 2.	A collection of items – ```results``` (3)
 3.	A string value ```__next``` (4) holds a URL to another page. Later this value will be used for a pagination


<img src="/assets/images/posts/adf-rest-p3/step1-2.png" alt="Step 1-2" />

#### Step 3: Adjust mapping settings
As soon as schema imported a few more things to be finalized:
 1.	Set a collection reference (1): JSON path to be specified. In our case it is: ```$['d']['results']```
 2.	Expand the node ```results``` and remove columns that do not have to be imported. 
 3.	Exclude value ```__next``` from the mapping, it should not be included in the final export 

#### Step 4. Run a test execution

This time Copy activity loads 22 kb of data or 500 rows which is a right step forward: 


<img src="/assets/images/posts/adf-rest-p3/step1-3.png" alt="Step 1-3" />

However, the data source contains more than 500 rows and the details page shows that only one object was read during fetching from REST. This means that only a single page was processed: 


<img src="/assets/images/posts/adf-rest-p3/step1-4.png" alt="Step 1-4" />



### Setting up a pagination

Normally, REST API limits its response size of a single request under a reasonable number; while to return large amounts of data, it splits the result into multiple pages and requires callers to send consecutive requests to get the next page of the result. Therefore, to get all rows pagination to be configured for a source data store.

#### Step 1. Configure pagination rule

 1.	Open a Source Tab of a Copy activity, navigate to “Pagination Rules” and click on “+ New”
 2.	Add a pagination rule:
      -	Name: ```AbsoluteUrl```
      -	Value: ```$.d.__next```

 
<img src="/assets/images/posts/adf-rest-p3/step2-1.png" alt="Step 2-1" />


#### Step 2. Run another test execution

A second execution shows more realistic stats - 26 REST API calls (or pages) loaded into 12751 rows:


<img src="/assets/images/posts/adf-rest-p3/step2-2.png" alt="Step 2-2" />



### Final words

The second piece of the pipeline – a Copy activity now is finally looking complete. It does not just establishes connections with a REST data source, also it fetches all expected rows and transforms them from a hierarchical into a tabular format.

In the next post of the series - <a href="/2019/adfv2-rest-api-part3-securing-keyvault">Azure Data Factory and REST APIs - Managing Pipeline Secrets by a Key Vault</a> - is a time to touch another important aspect – security and a management of sensitive data like client secrets or passwords.

Many thanks for reading.
