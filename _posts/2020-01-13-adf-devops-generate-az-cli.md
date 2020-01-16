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



### Coding a PowerShell script

##### Step 1: A naming convention and resource names

First things first, lets define a basic input like the location of the new azure resources, name of the environment and pick a right stage: dev, test or production:




```powershell
#az login  
  
# Step 1: Configure input parameters  
$EnvironmentName = "adf-devops"  
$Stage = "dev" # other options: stg | prd  
$Location= "westeurope"
$OutputFormat = "table"  # other options: json | jsonc | yaml | tsv

# internal: assign resource names
$ResourceGroupName = "rg-$EnvironmentName-$Stage"
$ADFName = "adf-$EnvironmentName-$Stage"
$KeyVaultName ="kv-$EnvironmentName-$Stage"
$StorageName = "adls$EnvironmentName$Stage".Replace("-","")
```


And a one-liner to check values of variables:

```powershell
Get-Variable ResourceGroupName, ADFName, KeyVaultName, StorageName | Format-Table 
```

Which results to a desired result:
<img src="/assets/images/posts/adf-cicd-p1/ps-variables.png" alt="the roadmap" />  



##### Step 2: Resource Group

The next logical step is to create a Resource Group that will act as a container for remaining services:

```powershell
# Step 2: Create a Resource Group
if (-Not (az group list --query "[].{Name:name}" -o table).Contains($ResourceGroupName))
{
    "Create a new resource group: $ResourceGroupName" 
    az group create --name $ResourceGroupName --location $Location --output $OutputFormat
}
else
{
   "Resource Group: $ResourceGroupName already exists"
}
```

Please note that a check of existing of the resource priors to the creation of such resource. For the resource group it is not a big deal, since Az CLI will not raise alerts or warnings if the resource group already exists. However, some other services will raise it.

##### Step 3: Key Vault and Storage Account
Creation of Key Vault and Storage accounts is similar to a Resource Group:

```powershell
# Step 3a: Create a Key Vault
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

# Step 3b: Create a Storage Account
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
```

##### Step 4: Data Factory

In contrast to other azure services created in this post, the Data Factory still has no support of Azure CLI, therefore, there still no command like:

```powershell
az datafactory create …
```

The workaround is to use an ARM template which can be placed to a storage account or, in case of a local PowerShell session, can be invoked from a local drive. 

The template:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "name": {
            "defaultValue": "myv2datafactory",
            "type": "String"
        },
        "location": {
            "defaultValue": "East US",
            "type": "String"
        },
        "apiVersion": {
            "defaultValue": "2018-06-01",
            "type": "String"
        }
    },
    "resources": [
        {
            "type": "Microsoft.DataFactory/factories",
            "apiVersion": "[parameters('apiVersion')]",
            "name": "[parameters('name')]",
            "location": "[parameters('location')]",
            "identity": {
                "type": "SystemAssigned"
            },
            "properties": {}
        }
    ]
}
```

I uploaded the template to a cloud storage and will access it directly by a `az group deployment` command:

```powershell
# Step 4: Create a Data Factory
az group deployment create `
  --resource-group $ResourceGroupName `
  --template-uri "https://csb17117e738f3dx40f1xa56.blob.core.windows.net/templates/adf.json" `
  --parameters name=$ADFName location=$Location `
  --output $OutputFormat

```


##### Step 5: The Final Script
It is a time to glue all parts together into a single script that will be used to generate development, test and production environments:

```powershell
# Step 1: Input parameters  
param([String]$EnvironmentName = "adf-devops",` 
      [String]$Stage = "dev",` 
      [String]$Location = "westeurope"`
      )


$OutputFormat = "table"  # other options: json | jsonc | yaml | tsv


# internal: assign resource names
$ResourceGroupName = "rg-$EnvironmentName-$Stage"
$ADFName = "adf-$EnvironmentName-$Stage"
$KeyVaultName ="kv-$EnvironmentName-$Stage"
$StorageName = "adls$EnvironmentName$Stage".Replace("-","")


Get-Variable ResourceGroupName, ADFName, KeyVaultName, StorageName | Format-Table 


# Step 2: Create a Resource Group
if (-Not (az group list --query "[].{Name:name}" -o table).Contains($ResourceGroupName))
{
    "Create a new resource group: $ResourceGroupName" 
    az group create --name $ResourceGroupName --location $Location --output $OutputFormat
}
else
{
   "Resource Group: $ResourceGroupName already exists"
}





# Step 3a: Create a Key Vault
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

# Step 3b: Create a Storage Account
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
  --template-file "./adf.json" `
  --parameters name=$ADFName location=$Location `
  --output $OutputFormat

```


### Generating three isolated environments

```powershell
# Development, Staging and Production:
.\Create-Environment.ps1 -EnvironmentName "adf-devops2020" -Stage "dev" -Location "westeurope"
.\Create-Environment.ps1 -EnvironmentName "adf-devops2020" -Stage "stg" -Location "westeurope"
.\Create-Environment.ps1 -EnvironmentName "adf-devops2020" -Stage "prd" -Location "westeurope"
```





#### Final words

..

Many thanks for reading.