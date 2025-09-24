# Building an Azure (Insecure) Static Website hosted in Storage Account

<img width="1619" height="916" alt="Screenshot 2025-09-19 at 9 38 44 AM" src="https://github.com/terrythomas00/azure_static_website_storage_account/blob/main/az_static_web.png" />

# The Architecture
![test](https://github.com/terrythomas00/azure_static_website_storage_account/blob/d0eb082394fbbd331a637964b3b564765d4f751c/architecture.png)
# Prerequisites
Before deploying this project, please ensure you have the following installed: 
## 1. **Install Azure CLI**
* Download and install Azure CLI: [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
* Install it based on the instructions for your OS. 
* Verify installation by running:
```bash
az --version
```
## 2. **Set Up VS Code (Optional, but Highly Recommended)**
* Install VS Code: VS [Code Download](https://code.visualstudio.com/download)

# Project Structure
```bash
    site
    ├── 404.html
    ├── css
    │   └── styles.css
    ├── img
    │   ├── car1.jpg
    │   ├── car2.jpg
    │   ├── car3.jpg
    │   ├── car4.jpg
    │   ├── car5.jpg
    │   ├── car6.jpg
    │   ├── car7.jpg
    │   └── car8.jpg
    └── index.html
```
# Deployment Guide
## Step 1: **Set your Variables**
```bash
RG=rg-static-demo
LOC=eastus
SA=staticsite$RANDOM  # storage account name must be globally unique, lowercase, <=24 chars
INDEX=index.html
ERROR=404.html
```
## Step 2: **Login into Azure via CLI**
```bash
az login
```
## Step 3: **Create your Resource Group**
```bash
az group create -n $RG -l $LOC
```
## Step 4: **Create insecure Storage account**
* Allows public blob access 
* Allows HTTP (Microsoft actually allows both http and https even though you disable https)
* Minimum TLS set to low (1.0)
* Shared keys rmain enabled (we'll use them here - insecure)

```bash
az storage account create \
-g $RG -n $SA -l $LOC \
--sku Standard_LRS --kind StorageV2 \
--allow-blob-public-access true \
--https-only false \
--min-tls-version TLS1_0
```
## Step 5: **Enable Static Website on $web Container**
```bash
az storage blob service-properties update \
--account-name $SA \
--static-website \
--index-document $INDEX \
--404-document $ERROR
```
## Step 6: **Upload using account key (insecure auth)**
```bash
KEY=$(az storage account keys list -n $SA -g $RG --query "[0].value" -o tsv)

az storage blob upload-batch -s ./site -d '$web' --account-name $SA --account-key $KEY
```
## Step 7: **Get the Website URL and Test**
```bash
WEB=$(az storage account show -n $SA -g $RG --query "primaryEndpoints.web" -o tsv)
echo "Website: $WEB"
```
# Next Steps - Hardened and Secure
