---
layout: post
title: Azure Data Factory and REST APIs - Encrypting Secrets with a Key Vault
description: Azure Data Factory and REST APIs - Encrypting Secrets with a Key Vault
comments: true
keywords: Azure Data Factory REST API Encrypting Key Vault
tags: [ADF, KeyVault]
category: Dev
published: true 
---


In this blog post I would like to put a light on a mapping and then pagination of a Copy activity which are often required for the ingestion of REST data. 

Mapping is optional for structured data, like databases, parquet or csv files, because the incoming dataset contains an inherited structure, so the Data Factory is smart enough to set it as a default mapping on a fly. 

However, to ingest hierarchical data like JSON or REST and then load it in as a tabular format an explicit schema mapping is required. In the previous blog post – <a href="/2019/adfv2-rest-api-part2-copy-activity">Azure Data Factory and REST APIs - Setting up a Copy activity</a> – such mapping was not provided yet explicitly. As a result, ADF was not able to write an incoming data streams in a tabular form and created an empty csv file. This post will fill such gap. Also, it will cover pagination which is also a common thing for REST APIs.


#### Prerequisites

 1.   Azure Key Vault
 2.	REST API Data source. I use ExactOnline SaaS service in this post as an example.
 3.	Azure Storage account.


<img src="/assets/images/posts/adf-rest-p4/step1-1.png" alt="Step 1-1" /> 

<img src="/assets/images/posts/adf-rest-p4/step1-2.png" alt="Step 1-2" /> 

<img src="/assets/images/posts/adf-rest-p4/step1-3.png" alt="Step 1-3" /> 

<img src="/assets/images/posts/adf-rest-p4/step2-1.png" alt="Step 2-1" /> 

<img src="/assets/images/posts/adf-rest-p4/step2-2.png" alt="Step 2-2" /> 

<img src="/assets/images/posts/adf-rest-p4/step2-3.png" alt="Step 2-3" /> 

<img src="/assets/images/posts/adf-rest-p4/step2-4.png" alt="Step 2-4" /> 

<img src="/assets/images/posts/adf-rest-p4/step2-5.png" alt="Step 2-5" /> 

<img src="/assets/images/posts/adf-rest-p4/step3-1.png" alt="Step 3-1" /> 

<img src="/assets/images/posts/adf-rest-p4/step3-2.png" alt="Step 3-2" /> 

<img src="/assets/images/posts/adf-rest-p4/step3-3.png" alt="Step 3-3" /> 


### Final words

The second piece of the pipeline – a Copy activity now is finally looking complete. It does not just establishes connections with a REST data source, also it fetches all expected rows and transforms them from a hierarchical into a tabular format.

In the next post of this series, it is time to touch another important topic – security and storing sensitive data like client secrets or passwords outside of Azure Data Factory.

Many thanks for reading.
