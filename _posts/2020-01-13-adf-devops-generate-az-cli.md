---
layout: post
title: Azure Data Factory & DevOps - Automation via Azure CLI
description: Azure Data Factory & DevOps - Automation via Azure CLI
comments: true
keywords: Azure Data Factory & DevOps - Automation via Azure CLI
tags: [Azure]
category: Dev
published: true 
---


In this post I would like to shift a focus to subjects like automation and DevOps. The intention is to display of how entire Data Factory environment and surrounding services can be deployed a command line or to be more precise, by an Azure CLI. 

Such approach maybe looks tedious and overkilling at a first glance, however it pays back since it brings nice things like reproducibility, enforcement of standards, avoidance of human mistakes.


### Our sample environment

Beside of a plain Data Factory, it is not uncommon that Data Engineers deal with a few additional Azure Services. Often the landscape also includes a Storage account, Key Vault etc. All these pieces create isolated environments for every stage: Development, Acceptance and Production. 

<!-- 
 - Resource Group
   - Key Vault
   - Storage Account
   - Data Factory -->

<img src="/assets/images/posts/adf-cicd-p1/adf-devops-environments.png" alt="the roadmap" /> 

  
The illustration above also shows the importance of standardizations in naming conventions and configurations because all environments expected to be the same

On a first glance it nice to have stages that also named similarly, however the most important strict naming is a base for further automation and DevOps is all about this.



#### Prerequisites
 -	Azure CLI. This is a modern cross-platform command line tool to manage azure services. It comes a replacement to older library AzureRM (https://azure.microsoft.com/es-es/blog/azure-powershell-cross-platform-az-module-replacing-azurerm/).

There are few ways of using Azure CLI:
 -	Using a Cloud Shell. It can be initiated directly from a top azure panel:
 <img src="/assets/images/posts/adf-cicd-p1/cloud-shell.png" alt="the roadmap" /> 
 -	Locally, for instance via PowerShell command line. For this a local installation of AZ CLI is required. For the installation tips check: [Install the Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)



### Building a PowerShell script


##### Step 1: Resources names and a naming convention

```powershell
#az login  
  
# Step 0: Configure parameters  
$EnvironmentName = "adf-devops"  
$Stage = "dev"  
$Location= "westeurope"
$OutputFormat = "table"  # other options: 


# internal: assign resource names
$ResourceGroupName = "rg-$EnvironmentName-$Stage"
$ADFName = "adf-$EnvironmentName-$Stage"
$KeyVaultName ="kv-$EnvironmentName-$Stage"
$StorageName = "adls$EnvironmentName$Stage".Replace("-","")
```


##### Step 2: Resource Group

Create Resource Group

##### Step 3: Key Vault

##### Step 4: Storage Account

##### Step 5: Data Factory

##### Step 6: Final Script
By gathering all parts together the final script:

```powershell
#az login  
  
# Step 0: Configure parameters  
$EnvironmentName = "adf-devops3"  
$Stage = "prd"  
$Location= "westeurope"
$OutputFormat = "table" 


# internal: assign resource names
$ResourceGroupName = "rg-$EnvironmentName-$Stage"
$ADFName = "adf-$EnvironmentName-$Stage"
$KeyVaultName ="kv-$EnvironmentName-$Stage"
$StorageName = "adls$EnvironmentName$Stage".Replace("-","")




# Step 1: Create a Resource Group
if (-Not (az group list --query "[].{Name:name}" -o table).Contains($ResourceGroupName))
{
    "Create a new resource group: $ResourceGroupName" 
    az group create --name $ResourceGroupName --location $Location --output $OutputFormat
}
else
{
   "Resource Group: $ResourceGroupName already exists"
}




# Step 2: Create a Key Vault
if (-Not (az keyvault list --resource-group $ResourceGroupName ` --query "[].{Name:name}" -o table).Contains($KeyVaultName))
{
    Write-Host "Creating a new key vault account: $KeyVaultName"
    az keyvault create `
       --location $Location `
       --name $KeyVaultName `
       --resource-group $ResourceGroupName `
       --output $OutputFormat
    
}
else
{
    "Key Vault: Resource $KeyVaultName already exists"
}


# Step 3: Create a Storage Account
if (-Not (az storage account list --resource-group $ResourceGroupName --query "[].{Name:name}" -o table).Contains($StorageName))
{
       Write-Host "Creating a new storage account: $StorageName"
       
       az storage account create `
       --name $StorageName `
        --resource-group $ResourceGroupName `
        --location $Location `
        --sku "Standard_LRS" `
        --kind "StorageV2" `
        --enable-hierarchical-namespace $true  `
        --output $OutputFormat      
}
else
{
    "Storage: Account $StorageName already exists"
}


# Step 4: Create a Data Factory
az group deployment create `
  --resource-group $ResourceGroupName `
  --template-uri "https://csb17117e738f3dx40f1xa56.blob.core.windows.net/templates/adf.json" `
  --parameters name=$ADFName location=$Location `
  --output $OutputFormat

```

#### Final words

..

Many thanks for reading.