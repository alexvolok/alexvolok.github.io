---
layout: post
title: Azure Data Factory and REST APIs - Managing Secrets by a Key Vault
description: Azure Data Factory and REST APIs - Managing Secrets by a Key Vault
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

 1. An instance of an Azure Key Vault
 2.	REST API Data source. I use Exact Online SaaS service in this post as an example
 3.	Azure Storage account, for instance Azure Data Lake gen2


### Migrating a sensitive data to a Key Vault

I will not touch within this post a creation of a new instance of a Key Vault. This strainghtforward process is well ilustrated by a Microsoft’s: <a href="https://docs.microsoft.com/en-us/azure/key-vault/quick-create-portal" target="_blank">Quickstart: Set and retrieve a secret from Azure Key Vault using the Azure portal</a>.

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



### Adding a Key Vault Secret to a pipeline

#### Step 1. Add a new web activity to a pipeline
 1.	Drop a new Web activity and connect it to a “Login”
 2.	Name it “Get Access Token”

 <img src="/assets/images/posts/adf-rest-p4/step2-1.png" alt="Step 2-1" /> 
 
#### Step 2. Configure a  “Get Access Token
 1.	Open a Settings tab of newly added activity
 2.	Paste a Key Vault secret identity URL that was prepared previously (1)
 3.	Choose method: GET
 4.	Set Authentication: MSI. It means that Key Vault will be accessed using a Managed Service Identity of Azure Data Factory. Previously we set to this account permissions to GET and LIST secrets
 5.	Set a resource: https://vault.azure.net
 
 <img src="/assets/images/posts/adf-rest-p4/step2-2.png" alt="Step 2-2" /> 

#### Step 3. Replace a hardcoded credential in a Login activity by a Key Vault Secret
 1.	On a pipeline click on “Login” activity and open tab “Settings”
 2.	In a field Body - Replace a hardcoded credential with an expression: ```@activity('Get Access Token').output.value```
 

<img src="/assets/images/posts/adf-rest-p4/step2-3.png" alt="Step 2-3" /> 

<br /> 

That’s it. The pipeline does not store sensitive data. During each execution the secret will be retreived from a Key Vault.

#### Step 4. Run a test execution

 1.	Hist on “Debug” button and wait when the execution is over. The good sign that all activities executed successfully (1), which means that last our actions didn’t break anything in a pipeline.
 2.	Click on an output of the Get Access Token (2):

<img src="/assets/images/posts/adf-rest-p4/step2-4.png" alt="Step 2-4" /> 

The output window shows credentials still passed as a plain text. Such information is captured by logging

<img src="/assets/images/posts/adf-rest-p4/step2-5.png" alt="Step 2-5" /> 

Luckily, this small issue is easy to fix and it will be done in a next section. 



### Securing input and output streams of pipeline activities

Our pipeline has two web tasks that sends and receives a sensitive data in unencrypted way: 

  <img src="/assets/images/posts/adf-rest-p4/step3-0.png" alt="Step 3-0" /> 

These activities requires small be adjusted one by one 

#### Step 1. Securing an output stream of “Get Access Token” activity
 1.	Click on Get Access Token activity on a canvas pipeline and open tab General
 2.	Select a checkbox “Secure output”
 
    <img src="/assets/images/posts/adf-rest-p4/step3-1.png" alt="Step 3-1" /> 

#### Step 2. Securing an output stream of “Login” activity
 1.	Click on Get Access Token activity on a canvas pipeline and open tab General
 2.	Select a checkbox “Secure output”
 3.	Select a checkbox “Secure input”

     <img src="/assets/images/posts/adf-rest-p4/step3-2.png" alt="Step 3-2" /> 
#### Step 3. Validation

Hit a debug button, wait when the web activities execution is over and examine the output:

<img src="/assets/images/posts/adf-rest-p4/step3-3.png" alt="Step 3-3" /> 
 
This time the output window shows scrambled text only. This is a desired effect, because such non-sensitive text flow comes to the logs instead of secrets which are send and intercepted by logging subsystem as a plain text.


### Final words

The third piece of our pipeline – a management of secrets by a Key Vault also has a place in an implementation. Also, the pipeline activities  do not send secrets as a plain text anymore.

Many thanks for reading.





