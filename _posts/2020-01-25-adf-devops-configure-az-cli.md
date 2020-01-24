---
layout: post
title: Azure Data Factory & DevOps - Test of Warning
description: Azure Data Factory & DevOps - Post Deployment Configuration via Azure CLI
comments: true
keywords: Azure Data Factory & DevOps - Post Deployment Configuration via Azure CLI
tags: [Azure]
category: Dev
published: true 
---

Previous post - [Automated Deployment via Azure CLI](/2020/adf-devops-generate-az-cli) is an example of how to get a brand-new environment by firing a script. However, it comes pretty blankly and unconfigured. In this post, I would like to talk about an automated configuration of Azure Data Factory deployments. At least a few things can be scripted in most of the cases: creation of storage containers, upload of the sample data, creation of secrets in a Key Vault and granting access of a Data Factory to them. 

#### 1
>[!WARNING]
>This is a warning

#### 2
_**Warning:** Be carefull

#### 3
> **Warning:** Be carefull

#### Prerequisites
 -	**Azure CLI.** This is a modern cross-platform command-line tool to manage Azure services. It comes to a replacement to the older library AzureRM. Read more: [Azure PowerShell – Cross-platform “Az” module replacing “AzureRM”](https://azure.microsoft.com/es-es/blog/azure-powershell-cross-platform-az-module-replacing-azurerm/){:target="_blank"}.
 - **MoviesDB.csv.** This flat dataset is often referenced in data engineering training. It is downloadable from a [Github repository] (https://raw.githubusercontent.com/djpmsft/adf-ready-demo/master/moviesDB.csv){:target="_blank"}


<br />

### Coding a PowerShell script

#### Step 1. A naming convention and resource names 

This block is to define the name of resources as variables that are later going to be used by configuration commands.

```powershell
param([String]$EnvironmentName = "adf-devops2020",` 
      [String]$Stage = "dev",` 
      [String]$Location = "westeurope"`
      )
$OutputFormat = "table"  # other options: json | jsonc | yaml | tsv
# internal: assign resource names
$ResourceGroupName = "rg-$EnvironmentName-$Stage"
$ADFName = "adf-$EnvironmentName-$Stage"
$KeyVaultName ="kv-$EnvironmentName-$Stage"
$StorageName = "adls$EnvironmentName$Stage".Replace("-","")
```

#### Step 2. Post-configuration of a storage account

The script firstly programmatically  retrieves a connection string of a storage account. Then it creates a sample container with a name “dwh” and disabled public access and uploads a sample file: MoviesDB.csv.

```powershell
"#Step 1.1: Obtaining a connection string"
$connectionString= (az storage account show-connection-string `
                                -n $StorageName `
                                -g $ResourceGroupName `
                                --query connectionString `
                                -o tsv `
                    )


"#Step 1.2: Creating a container `dwh` "
az storage container create `
            --name "dwh" `
            --public-access off `
            --connection-string $connectionString `
            --output $OutputFormat 

"#Step 1.3: uploading a sample file to a container"
az storage blob upload `
    --name "MoviesDB.csv" `
    --container "dwh" `
    --file "C:\adf-devops\MoviesDB.csv" `
    --connection-string $connectionString `
    --no-progress `
    --output $OutputFormat
```

#### Step 3. Post-configuration of a Key Vault

This is a final step in which a previously retrieved connection string is to be added to a Key Vault. 
When that is done, a service principal account of a Data Factory will be assigned with permission to List and Get secrets. This step is mandatory since Azure Data Factory runs under own managed identity and it has to be explicitly granted to have access to a Key Vault.

```powershell
# Adding a secret to a Key Vault

"#Step 2.1: Adding a storage account connection string to a key vault"
az keyvault secret set `
            --vault-name $KeyVaultName `
            --name "AzStorageKey" `
            --value $connectionString `
            --output $OutputFormat


"#Step 2.2: Obtaining an Object ID of Azure Data Factory instance"
$ADF_Object_ID =  (az ad sp list `
                        --display-name $ADFName `
                        --output tsv `
                        --query "[].{id:objectId}" `
                   )

"#Step 2.3: Granting access permissions of ADF to a KeyVault"
az keyvault set-policy `
            --name $KeyVaultName `
            --object-id $ADF_Object_ID `
            --secret-permissions get list `
            --query "{Status:properties.provisioningState}" `
            --output $OutputFormat
```

#### Step 4. Placing it all together

Finally, lets put all pieces together into a post-configuration script:


```powershell
param([String]$EnvironmentName = "adf-devops2020",` 
      [String]$Stage = "dev",` 
      [String]$Location = "westeurope"`
      )


$OutputFormat = "table"  # other options: json | jsonc | yaml | tsv


# internal: assign resource names
$ResourceGroupName = "rg-$EnvironmentName-$Stage"
$ADFName = "adf-$EnvironmentName-$Stage"
$KeyVaultName ="kv-$EnvironmentName-$Stage"
$StorageName = "adls$EnvironmentName$Stage".Replace("-","")


# Configuring a storage account:

"#Step 1.1: Obtaining a connection string"
$connectionString= (az storage account show-connection-string `
                                -n $StorageName `
                                -g $ResourceGroupName `
                                --query connectionString `
                                -o tsv `
                    )


"#Step 1.2: Creating a container `dwh` "
az storage container create `
            --name "dwh" `
            --public-access off `
            --connection-string $connectionString `
            --output $OutputFormat 

"#Step 1.3: uploading a sample file to a container"
az storage blob upload `
    --name "MoviesDB.csv" `
    --container "dwh" `
    --file "C:\adf-devops\MoviesDB.csv" `
    --connection-string $connectionString `
    --no-progress `
    --output $OutputFormat



# Adding a secret to a Key Vault

"#Step 2.1: Adding a storage account connection string to a key vault"
az keyvault secret set `
            --vault-name $KeyVaultName `
            --name "AzStorageKey" `
            --value $connectionString `
            --output $OutputFormat


"#Step 2.2: Obtaining an Object ID of Azure Data Factory instance"
$ADF_Object_ID =  (az ad sp list `
                        --display-name $ADFName `
                        --output tsv `
                        --query "[].{id:objectId}" `
                   )

"#Step 2.3: Granting access permissions of ADF to a KeyVault"
az keyvault set-policy `
            --name $KeyVaultName `
            --object-id $ADF_Object_ID `
            --secret-permissions get list `
            --query "{Status:properties.provisioningState}" `
            --output $OutputFormat
```
This script is named as *Configure-Environment.ps1* and placed to the same folder as *MoviesDB.CSV* – `C:\adf-devops\`

### A post-configuration script in action

Since the environments are already generated and a configuration script is ready, let's run it to make a production stage adjusted:

```powershell
.\Configure-Environment.ps1 -EnvironmentName "adf-devops2020" -Stage "prd" 
```

In contrast to the duration of generation of the objects, their cconfiguration is a relatively fast operation, so the execution duration of the script takes seconds. When it is done, let's check the result using Azure Portal:

Check 1. Storage account has a container "dwh" that contains MoviesDB.csv:

<img src="/assets/images/posts/adf-cicd-p2/step-1-storage.png" alt="step-1-storage" />  

 
Check 2. Key Vault keeps a new secret - "AzStorageKey":

<img src="/assets/images/posts/adf-cicd-p2/step-2-keyvault-secret.png" alt="step-2-keyvault-secret" />  

 
Check 3. Key Vault has an Access Policy that allows Data Factory to run operations *List* and *Get*:

<img src="/assets/images/posts/adf-cicd-p2/step-2-keyvault-access-policy.png" alt="step-3-keyvault-access" />  


#### Final words

The second Azure CLI script is complete. It automates the post-deployment configuration of the environment - creates a storage container and uploads a sample data and then configures a Key Vault to store a connection string to be used later in a Data Factory. In the next post, I will cover a version control of data factory pipelines using Azure DevOps.

Many thanks for reading.