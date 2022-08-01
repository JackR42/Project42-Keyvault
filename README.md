# Project42-Keyvault: https://youtu.be/3IrzFrHn434

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
