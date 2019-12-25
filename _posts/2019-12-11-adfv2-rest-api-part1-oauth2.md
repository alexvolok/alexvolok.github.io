---
layout: post
title: Azure Data Factory and REST APIs - Dealing with oauth2 authentication
description: ADFv2 and REST APIs - Dealing with oauth2 authentication
comments: false
keywords: Azure Data Factory REST API OAUTH
published: true 
---

In this first post I am going to discuss how to apply oauth2 authentication to ingest REST APIs data.

OAUTH2 became a standard de facto in cloud and SaaS services, it used widely by Twitter, Microsoft Azure, Amazon. My example is based on EXACT Online API. The end-target of the blog series is to setup an entire pipeline which will ingest data from a REST API and load it to a data lake.

#### The structure of a pipeline

The simplest form of a pipeline contains two activities:

 -	**Web Activity:** Performs a POST call get an authentication token
 -	**Copy Activity:** Fetches data from API and load it to a destination linked service
 
<img src="/assets/images/posts/adf-rest-p1/pipeline.png" alt="Step 0" />      

In this post we will focus on implementing the first activity – Web Activity which has to do small but important part of the work – authenticate and get access token.


#### The first step - prepare request body
The very first step is to generate an authentication string which is going to be send as a request body in order to get access token

The string has a format:

``` grant_type=refresh_token&client_id={client_id}&client_secret={client_secret}&refresh_token={refresh_token} ```

To get a string complete, replace placeholders {client_id}, {client_secret} and {refresh_token} with real values so the final variant will have a look:

```grant_type=refresh_token&client_id=000000-11111-2222-3333-5b2c48c2fae4&client_secret=50DaGMXKDDI1&refresh_token=abcde_very_long_token_abcdefg4343```

This value is going to be used later in a web request.

#### Implementing Web Activity “Login”

##### Step 1: Create pipeline and add a Web Activity task

 1. Create a new pipeline and give a name "loadEOLRestApi"
 2.  Open toolbox “Activities”, expand a section “General” and drop “Web” to a pipeline canvas
 3. Give a name “Login” to a just added Web activity

The authoring canvas should have a such look:

<img src="/assets/images/posts/adf-rest-p1/added_web_activity.png" alt="Step 1" />     


##### Step 2: Configure a Web Activity task
Click on a Login activity and open tab “Settings”. Then fill following fields with values:
 1.	URL: An url to perform authentication and get a token (1). In my case it is: https://start.exactonline.nl/api/oauth2/token
 2. Method: POST (2)
 3. Add a new header (3):
     -  Name: Content-Type
     -  Value: application/x-www-form-urlencoded

 4. Use  a previously generated authentication string in a Body field (4):

<img src="/assets/images/posts/adf-rest-p1/web_activity_settings.png" alt="Step 2" />

##### Step 3. Perform a test execution:
 1. Click on a Debug button (1) 
 2. Open an Output tab of a pipeline (2)
 3. Click on a button “Output” (3)
 4. The output window will show a JSON response (4). Later in a pipeline a value of “access_token” will be used as a key to get access to underlying API methods

<img src="/assets/images/posts/adf-rest-p1/debug_access_token.png" alt="Step 3" />

#### Final words

The first piece of the pipeline – a web call to proceed authentication has been just implemented. It sends a request as a specially prepared string to a remote web API and receives an output in JSON format. The output contains few key-value pairs, like access_token, token_type, expires_in etc which are going to be used later by a Copy Activity.

Because the output in a JSON format, other pipeline activities can get access to a token by using an expression this way:

``` activity('Login').output.access_token ```

The next post is going to be focused on implementing a second piece of a pipeline - Copy Activity.