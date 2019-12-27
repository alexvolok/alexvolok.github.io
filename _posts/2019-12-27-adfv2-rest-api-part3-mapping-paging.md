---
layout: post
title: Azure Data Factory and REST APIs - Mapping and Paging
description: Azure Data Factory and REST APIs - Mapping and Paging
comments: false
keywords: Azure Data Factory REST API Copy Activity Mapping Paging
published: true 
---


In this blog post I would like to show of how to setup Mapping and then Paging in an existing Copy Activity that ingests REST data. 

Mapping is optional for a structured data, like databases, parquet or csv files, because the incoming dataset contains an inherited  structure, so the Data Factory is smart enough to use it to make a default mapping on a fly. 

To ingest hierarchical data and load it in a tabular format an explicit schema mapping is required. In the previous blog post – name – such mapping was not provided, as result ADF was not able to write incoming data stream in a tabular form. This blog post will fill this gap. Also it will cover a pagination which is common in REST APIs.

#### Prerequisites

 1.	REST API Data source. I use ExactOnline in this post.
 2.	Azure Storage account



### Mapping

<img src="/assets/images/posts/adf-rest-p3/step1-1.png" alt="Step 1-1" />

<img src="/assets/images/posts/adf-rest-p3/step1-2.png" alt="Step 1-2" />

<img src="/assets/images/posts/adf-rest-p3/step1-3.png" alt="Step 1-3" />

<img src="/assets/images/posts/adf-rest-p3/step1-4.png" alt="Step 1-4" />


### Paging
 
<img src="/assets/images/posts/adf-rest-p3/step2-1.png" alt="Step 2-1" />

<img src="/assets/images/posts/adf-rest-p3/step2-2.png" alt="Step 2-2" />



### Final words
The third piece of the pipeline ...

Many thanks for reading.