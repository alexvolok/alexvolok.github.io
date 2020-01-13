---
layout: post
title: Azure Data Factory CI/CD - DEV-STG-PRD by Azure CLI
description: Azure Data Factory CI/CD - DEV-STG-PRD by Azure CLI
comments: true
keywords: Azure Data Factory CI/CD - DEV-STG-PRD by Azure CLI
tags: [Azure]
category: Dev
published: true 
---


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