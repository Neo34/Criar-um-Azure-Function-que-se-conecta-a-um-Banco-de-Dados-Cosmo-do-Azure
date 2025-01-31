# Criar-um-Azure-Function-que-se-conecta-a-um-Banco-de-Dados-Cosmo-do-Azure

Entrar no Azure
O Cloud Shell é autenticado automaticamente na conta inicial usada para entrar. Use o seguinte script para entrar com outra assinatura, substituindo subscriptionId pela ID de sua assinatura do Azure.

subscription="subscriptionId" # Set Azure subscription ID here

az account set -s $subscription # ...or use 'az login'

Executar o script

# Function app and storage account names must be unique.

# Variable block
let "randomIdentifier=$RANDOM*$RANDOM"
location="eastus"
resourceGroup="msdocs-azure-functions-rg-$randomIdentifier"
tag="create-function-app-connect-to-cosmos-db"
storage="msdocsaccount$randomIdentifier"
functionApp="msdocs-serverless-function-$randomIdentifier"
skuStorage="Standard_LRS"
functionsVersion="4"

# Create a resource group
echo "Creating $resourceGroup in "$location"..."
az group create --name $resourceGroup --location "$location" --tags $tag

# Create a storage account for the function app.
echo "Creating $storage"
az storage account create --name $storage --location "$location" --resource-group $resourceGroup --sku $skuStorage

# Create a serverless function app in the resource group.
echo "Creating $functionApp"
az functionapp create --name $functionApp --resource-group $resourceGroup --storage-account $storage --consumption-plan-location "$location" --functions-version $functionsVersion

# Create an Azure Cosmos DB database account using the same function app name.
echo "Creating $functionApp"
az cosmosdb create --name $functionApp --resource-group $resourceGroup

# Get the Azure Cosmos DB connection string.
endpoint=$(az cosmosdb show --name $functionApp --resource-group $resourceGroup --query documentEndpoint --output tsv)
echo $endpoint

key=$(az cosmosdb keys list --name $functionApp --resource-group $resourceGroup --query primaryMasterKey --output tsv)
echo $key

# Configure function app settings to use the Azure Cosmos DB connection string.
az functionapp config appsettings set --name $functionApp --resource-group $resourceGroup --setting CosmosDB_Endpoint=$endpoint CosmosDB_Key=$key

Limpar os recursos

az group delete --name $resourceGroup