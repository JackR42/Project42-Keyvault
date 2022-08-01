# Project42-Keyvault: https://youtu.be/3IrzFrHn434

# 1) Create Azure Key Vault with Secrets and persmissions

# Create the variables
KEYVAULT_RG="S2-RG-KeyVault42"
KEYVAULT_NAME="S2-KV-KeyVault42"
KEYVAULT_REGION="westeurope"
SUBSCRIPTION_ID=$(az account show --query id -o tsv)

# Create new resource group
az group create --name $KEYVAULT_RG -l $KEYVAULT_REGION

# Create key vault with RBAC option (not Access Policy)
az keyvault create --name $KEYVAULT_NAME \
   --resource-group $KEYVAULT_RG \
   --enable-rbac-authorization

# Assign RBAC role to the current user to manage secrets
USER_ID=$(az ad signed-in-user show --query id -o tsv)

KEYVAULT_ID=$(az keyvault show --name $KEYVAULT_NAME \
   --resource-group $KEYVAULT_RG \
   --query id \
   --output tsv)

az role assignment create --role "Key Vault Secrets Officer" \
   --scope $KEYVAULT_ID \
   --assignee-object-id $USER_ID
   
# Create a secret
az keyvault secret set --name "DatabaseUserName" \
  --value "admindba" \
  --vault-name $KEYVAULT_NAME

#2) Create Service Principal to access Key Vault from Azure DevOps Pipelines
# Create a service principal
SPN=$(az ad sp create-for-rbac -n "S2-SPN-KeyVault42")

echo $SPN | jq .

SPN_APPID=$(echo $SPN | jq .appId)

az ad sp list --display-name "S2-SPN_KeyVault42" --query [0].objectId --out tsv
<!-- SPN_ID=$(az ad sp show --id $SPN_APPID --query objectId --out tsv) -->

# Assign RBAC role to the service principal
az role assignment create --role "Key Vault Secrets User" \
   --scope $KEYVAULT_ID \
   --assignee-object-id $SPN_ID

#3) Create a pipeline to access Key Vault Secrets
#3.1) Create Service Connection using the SPN
Create a service connection in Azure DevOps using the SPN created earlier.

#3.2) Create YAML pipeline
