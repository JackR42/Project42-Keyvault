# Project42-Keyvault

Demoing the use of Azure Key Vault Secrets within Azure DevOps Pipelines.

Video is available here: https://youtu.be/3IrzFrHn434

1) Create Azure Key Vault with Secrets and persmissions
The following script will create a Key Vault and Secret:

# create the variables
KEYVAULT_RG="S2-RG-KeyVault"
KEYVAULT_NAME="S2-KV-KeyVault"
KEYVAULT_REGION="westeurope"
SUBSCRIPTION_ID=$(az account show --query id -o tsv)

# create new resource group
az group create --name $KEYVAULT_RG -l $KEYVAULT_REGION

# create key vault with RBAC option (not Access Policy)
az keyvault create --name $KEYVAULT_NAME \
   --resource-group $KEYVAULT_RG \
   --enable-rbac-authorization

# assign RBAC role to the current user to manage secrets
USER_ID=$(az ad signed-in-user show --query id -o tsv)

KEYVAULT_ID=$(az keyvault show --name $KEYVAULT_NAME \
   --resource-group $KEYVAULT_RG \
   --query id \
   --output tsv)
   
   az role assignment create --role "Key Vault Secrets Officer" \
   --scope $KEYVAULT_ID \
   --assignee-object-id $USER_ID
   
# create a secret
az keyvault secret set --name "DatabaseUserName" \
  --value "admindba" \
  --vault-name $KEYVAULT_NAME
  
 
