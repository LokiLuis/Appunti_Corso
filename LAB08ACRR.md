RESOURCE_GROUP="myResourceGroup"
LOCATION="eastus"
ACR_NAME="mycontainerregistry12345"
IMAGE_NAME="sample/hello-world"
IMAGE_TAG="v1"

az login
az group create --location $LOCATION --name $RESOURCE_GROUP
az acr create --resource-group $RESOURCE_GROUP --name $ACR_NAME --sku Basic

mkdir -p ~/acr-local-build-lab
cd ~/acr-local-build-lab
echo 'FROM mcr.microsoft.com/hello-world' > Dockerfile

az acr login --name $ACR_NAME

docker build -t $IMAGE_NAME:$IMAGE_TAG .
docker tag $IMAGE_NAME:$IMAGE_TAG $ACR_NAME.azurecr.io/$IMAGE_NAME:$IMAGE_TAG
docker push $ACR_NAME.azurecr.io/$IMAGE_NAME:$IMAGE_TAG

az acr repository list --name $ACR_NAME --output table
az acr repository show-tags --name $ACR_NAME --repository $IMAGE_NAME --output table

docker rmi $ACR_NAME.azurecr.io/$IMAGE_NAME:$IMAGE_TAG
docker rmi $IMAGE_NAME:$IMAGE_TAG
docker pull $ACR_NAME.azurecr.io/$IMAGE_NAME:$IMAGE_TAG
docker run --rm $ACR_NAME.azurecr.io/$IMAGE_NAME:$IMAGE_TAG
