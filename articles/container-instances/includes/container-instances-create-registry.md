---
services: container-instances
author: tomvcassidy
ms.service: azure-container-instances
ms.topic: include
ms.date: 08/29/2024
ms.author: tomcassidy
ms.custom: devx-track-azurecli
# Customer intent: "As a cloud developer, I want to create an Azure container registry and sign in to it, so that I can manage and deploy my container images efficiently."
---

## Create Azure container registry

Before you create your container registry, you need a *resource group* to deploy it to. A resource group is a logical collection into which all Azure resources are deployed and managed.

Create a resource group with the [az group create][az-group-create] command. In the following example, a resource group named *myResourceGroup* is created in the *eastus* region:

```azurecli
az group create --name myResourceGroup --location eastus
```

Once you create the resource group, create an Azure container registry with the [az acr create][az-acr-create] command. The container registry name must be unique within Azure, and contain 5-50 alphanumeric characters. Replace `<acrName>` with a unique name for your registry:

```azurecli
az acr create --resource-group myResourceGroup --name <acrName> --sku Basic
```

Here's partial output for a new Azure container registry named *mycontainerregistry082*:

```output
{
  "creationDate": "2020-07-16T21:54:47.297875+00:00",
  "id": "/subscriptions/<Subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.ContainerRegistry/registries/mycontainerregistry082",
  "location": "eastus",
  "loginServer": "mycontainerregistry082.azurecr.io",
  "name": "mycontainerregistry082",
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
  "sku": {
    "name": "Basic",
    "tier": "Basic"
  },
  "status": null,
  "storageAccount": null,
  "tags": {},
  "type": "Microsoft.ContainerRegistry/registries"
}
```

The rest of the tutorial refers to `<acrName>` as a placeholder for the container registry name that you chose in this step.

## Sign in to container registry

You must sign in to your Azure Container Registry instance before pushing images to it. Use the [az acr login][az-acr-login] command to complete the operation. You must provide the unique name you chose for the container registry when you created it.

```azurecli
az acr login --name <acrName>
```

For example:

```azurecli
az acr login --name mycontainerregistry082
```

The command returns `Login Succeeded` once completed:

```output
Login Succeeded
```

<!-- LINKS - Internal -->
[az-acr-create]: /cli/azure/acr#az_acr_create
[az-acr-login]: /cli/azure/acr#az_acr_login
[az-group-create]: /cli/azure/group#az_group_create
