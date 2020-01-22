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

In a coming series of posts, I would like to shift a focus to subjects like automation and DevOps. This post will show how the entire Data Factory environment and surrounding services can be deployed by a command line or be more precise, by the Azure CLI. 

Such an approach may look tedious and overkilling at a first glance, however, it pays back since it brings nice things like reproducibility, enforcement of standards, avoidance of human mistakes.


#### Prerequisites
 -	Azure CLI. This is a modern cross-platform command-line tool to manage Azure services. It comes to a replacement to the older library AzureRM. Read more: [Azure PowerShell – Cross-platform “Az” module replacing “AzureRM”](https://azure.microsoft.com/es-es/blog/azure-powershell-cross-platform-az-module-replacing-azurerm/).



### Coding a PowerShell script
lalala
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

Write-Host "#Step 1: Obtaining a connection string"
$connectionString= (az storage account show-connection-string -n $StorageName -g $ResourceGroupName --query connectionString -o tsv)

Write-Host "#Step 2: Creating a container `dwh` "
az storage container create --name "dwh" --public-access off --connection-string $connectionString --output $OutputFormat

Write-Host "#Step 3: uploading a sample dummy file to a container"
az storage blob upload --name "adf.json" --container "dwh" --file "C:\adf-devops\adf.json" --connection-string $connectionString --no-progress --output $OutputFormat


# Adding a secret to a Key Vault

Write-Host "#Step 4: Adding a storage account connection string to a key vault"
az keyvault secret set --vault-name $KeyVaultName --name "AzStorageKey" --value $connectionString --output $OutputFormat


Write-Host "#Step 5: Obtaining an Object ID of Azure Data Factory instance"
$ADF_Object_ID =  (az ad sp list --display-name $ADFName  --output tsv  --query "[].{id:objectId}")

#Step 6: Granting access permissions of ADF to a KeyVault
az keyvault set-policy --name $KeyVaultName --object-id $ADF_Object_ID --secret-permissions get list --query "{Status:properties.provisioningState}" --output $OutputFormat
```

<img src="/assets/images/posts/adf-cicd-p1/generated-objects.png" alt="the roadmap" />  


#### Final words

The Azure CLI script is complete. It automates the creation of the entire data engineering landscape and brings some extra goodies. Those goodies are enforced naming convention, standardization and some time-saving. Especially if the same task repeats during each project intake step.

Many thanks for reading.