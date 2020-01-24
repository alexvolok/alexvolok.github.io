---
layout: post
title: Azure Data Factory & DevOps - Post Deployment Configuration via Azure CLI
description: Azure Data Factory & DevOps - Post Deployment Configuration via Azure CLI
comments: true
keywords: Azure Data Factory & DevOps - Post Deployment Configuration via Azure CLI
tags: [Azure]
category: Dev
published: true 
---

Previous post <> is an example example of how to get a brand-new deployment by firing a script. However, it comes pretty blank and unconfigured. In this post I would like to talk about an automated post configuration of Azure Data Factory environments. At least a few things can be automated in the most of deployments: creation of storage containers and upload of the sample data. Also, creation of secrets in a Key Vault and granting access of Data Factory to them. 

#### Prerequisites
 -	**Azure CLI.** This is a modern cross-platform command-line tool to manage Azure services. It comes to a replacement to the older library AzureRM. Read more: [Azure PowerShell – Cross-platform “Az” module replacing “AzureRM”](https://azure.microsoft.com/es-es/blog/azure-powershell-cross-platform-az-module-replacing-azurerm/).
 - **MoviesDB.csv.** This flat dataset is often used by a data engineering trainings and available by a link Github (https://raw.githubusercontent.com/djpmsft/adf-ready-demo/master/moviesDB.csv)


<br />

### Coding a PowerShell script

#### Step 1. A naming convention and resource names 

This block is a to define name of resources as variables that later going to be used by a configuration calls.

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

In this step the script firstly programantically retrieves a connection string of a storage account which is necessary to access this account later. Then it creates a sample container with a name “dwh” and disabled public access and upload a sample file: MoviesDB.csv

```powershell
# Configuring a storage account:
Write-Host "#Step 1.1: Obtaining a connection string"
$connectionString= (az storage account show-connection-string `
                                -n $StorageName `
                                -g $ResourceGroupName `
                                --query connectionString `
                                -o tsv `
                    )


Write-Host "#Step 1.2: Creating a container `dwh` "
az storage container create `
            --name "dwh" `
            --public-access off `
            --connection-string $connectionString `
            --output $OutputFormat 

Write-Host "#Step 1.3: uploading a sample dummy file to a container"
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
After that a service principal account of a Data Factory will be assigned with permissions to List and Get secrets from it. This step is mandatory, since Azure Data Factory runs under own managed account and it has to be explicitly granted to have access to a storage of secrets.

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

Finally, lets put all pieces together into a post-configuration script **<script name>**: 


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

"#Step 1.3: uploading a sample dummy file to a container"
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

<img src="/assets/images/posts/adf-cicd-p1/generated-objects.png" alt="the roadmap" />  


#### Final words

The Azure CLI script is complete. It automates the creation of the entire data engineering landscape and brings some extra goodies. Those goodies are enforced naming convention, standardization and some time-saving. Especially if the same task repeats during each project intake step.

Many thanks for reading.