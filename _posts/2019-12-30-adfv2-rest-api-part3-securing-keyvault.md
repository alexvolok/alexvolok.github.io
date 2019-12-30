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


In this post I will touch a slightly different topic from the other few published in a series. The topic is a a security or, to be more precise, the management of secrets like passwords and keys.

The pipeline created <a href="/2019/adfv2-rest-api-part3-mapping-pagination">previously</a> finally works. It ingests data from a third-party REST source and stores it in a data lake. However, to get authorized by REST the client id and client secret stored in a web activity configuration as a plain text.

The solution will be improved and become more secure by adding the Azure Key Vault to the scene, because this service can be used to securely store and tightly control access to tokens, passwords, certificates, API keys, and other secrets.

Also, this post will cover a security of input and output of activities.

#### Prerequisites

 1. An instance of a Azure Key Vault
 2.	REST API Data source. I use Exact Online SaaS service in this post as an example
 3.	Azure Storage account, for instance Azure Data Lake gen2


### Migrating a sensitive data to a Key Vault

I will not touch within this post a creation of a new instance of a Key Vault. This strainghtforward process is well ilustrated by a Microsoft’s: Quickstart: Set and retrieve a secret from Azure Key Vault using the Azure portal
Therefore, stepping directly into a creation of the secret and migration of a sensitive data from a pipeline into it

#### Step 1. Create an Azure Key Vault Secret

 1.	Open a Key Vault service page in Azure Portal
 2.	Click on link “Secrets” which can be found on a  left pane in a section “Settings”.
 3.	In a form “Create secret” fill following fields:
     -	Upload options: Manual
     -	Name: KV-EOL-REFRESH-TOKEN
     -	Value: Copy the value from a “Body” field of an existing Login activity
 4.	Click on “Create”  

 <img src="/assets/images/posts/adf-rest-p4/step1-1.png" alt="Step 1-1" /> 
 
#### Step 2. Prepare an URL of a Key Vault Secret

 1.	As soon as a new secret created open it and copy the URL “Secret Identifier” (1) to the clipboard

<img src="/assets/images/posts/adf-rest-p4/step1-2.png" alt="Step 1-2" /> 

#### Step 3. Grant access of ADF to a Key Vault

 1.	Open “Access Policies”
 2.	Click on “+ Add Access Policy”
 3.	On a form “Add access policy” fill:

     -	Configure from template: Secret Management

     -	Secret permissions: choose Get and List

     -	Select principal: enter a name of Azure Data Factory instance

 4.	Click on Add to submit a form.

 5.	On a parent page “Access Policies” click on Save.

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
