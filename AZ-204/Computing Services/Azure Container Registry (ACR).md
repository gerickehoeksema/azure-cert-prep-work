## Import images
To manage copies of public images, create an Azure container registry if you don't already have one

As a recommended one-time step, [import](https://learn.microsoft.com/en-ca/azure/container-registry/container-registry-import-images) base images and other public content to your Azure container registry. The [az acr import](https://learn.microsoft.com/en-us/cli/azure/acr#az-acr-import) command in the Azure CLI supports importing images from public registries, such as Docker Hub and Microsoft Container Registry, and from private container registries.

`az acr import` doesn't require a local Docker installation. You can run it with a local installation of the Azure CLI or directly in Azure Cloud Shell. It supports images of any OS type, multi-architecture images, or OCI artifacts such as Helm charts.

Depending on your organization's needs, you can import to a dedicated registry or a repository in a shared registry.

```bash
az acr import 
	--name myregistry 
	--source docker.io/library/hello-world:latest 
	--image hello-world:latest 
	--username <Docker Hub username> 
	--password <Docker Hub token>
```

# Azure Container Registry custom roles

## Custom role permissions
To determine which permissions (actions and data actions) should be defined in a custom role, you can:

- Review the JSON definition of [Azure built-in roles directory for Containers](https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles/containers) which includes commonly used permissions (actions and data actions) that are used in ACR built-in roles,
- Review the complete list of `Microsoft.ContainerRegistry` resource provider permissions ([Azure Container Registry reference of actions and data actions](https://learn.microsoft.com/en-us/azure/role-based-access-control/permissions/containers#microsoftcontainerregistry))

To programmatically list all available permissions (actions and data actions) for the `Microsoft.ContainerRegistry` resource provider, you can use the following Azure CLI or Azure PowerShell commands.

```shell
az provider operation show --namespace Microsoft.ContainerRegistry
```

```shell
Get-AzProviderOperation -OperationSearchString Microsoft.ContainerRegistry
```

## Example:
For example, the following JSON defines the minimum permissions (actions and data actions) for a custom role that permits [managing ACR webhooks](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-webhook).

```json
{
   "assignableScopes": [
     "/subscriptions/<optional, but you can limit the visibility to one or more subscriptions>"
   ],
   "description": "Manage Azure Container Registry webhooks.",
   "Name": "Container Registry Webhook Contributor",
   "permissions": [
     {
       "actions": [
         "Microsoft.ContainerRegistry/registries/webhooks/read",
         "Microsoft.ContainerRegistry/registries/webhooks/write",
         "Microsoft.ContainerRegistry/registries/webhooks/delete"
       ],
       "dataActions": [],
       "notActions": [],
       "notDataActions": []
     }
   ],
   "roleType": "CustomRole"
 }
```


# Authenticate

## Authentication options

|Method|How to authenticate|Scenarios|Microsoft Entra role-based access control (RBAC)|Limitations|
|---|---|---|---|---|
|[Individual Microsoft Entra identity](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-authentication?tabs=azure-cli#individual-login-with-azure-ad)|`az acr login` in Azure CLI  <br>  <br>`Connect-AzContainerRegistry` in Azure PowerShell|Interactive push/pull by developers, testers|Yes|Microsoft Entra token must be renewed every 3 hours|
|[Microsoft Entra service principal](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-authentication?tabs=azure-cli#service-principal)|`docker login`  <br>  <br>`az acr login` in Azure CLI  <br>  <br>`Connect-AzContainerRegistry` in Azure PowerShell  <br>  <br>Registry login settings in APIs or tooling  <br>  <br>[Kubernetes pull secret](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-auth-kubernetes)|Unattended push from CI/CD pipeline  <br>  <br>Unattended pull to Azure or external services|Yes|SP password default expiry is 1 year|
|[Microsoft Entra managed identity for Azure resources](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-authentication-managed-identity)|`docker login`  <br>  <br> `az acr login` in Azure CLI  <br>  <br>`Connect-AzContainerRegistry` in Azure PowerShell|Unattended push from Azure CI/CD pipeline  <br>  <br>Unattended pull to Azure services  <br>  <br>For a list of managed identity role assignment scenarios, consult the [ACR role assignment scenarios](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-rbac-built-in-roles-overview).|Yes  <br>  <br>[Microsoft Entra RBAC role assignments with ACR built-in roles](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-rbac-built-in-roles-overview)  <br>  <br>[Microsoft Entra attribute-based access control (ABAC) for **Microsoft Entra-based repository permissions**](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-rbac-abac-repository-permissions)|Use only from select Azure services that [support managed identities for Azure resources](https://learn.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/services-support-managed-identities#azure-services-that-support-managed-identities-for-azure-resources)|
|[AKS cluster node kubelet managed identity](https://learn.microsoft.com/en-us/azure/aks/cluster-container-registry-integration?toc=/azure/container-registry/toc.json&bc=/azure/container-registry/breadcrumb/toc.json)|Attach registry when AKS cluster created or updated|Unattended pull to AKS cluster node in the same or a different subscription|No, pull access only|Only available with AKS cluster  <br>  <br>Can't be used for cross-tenant authentication|
|[AKS cluster service principal](https://learn.microsoft.com/en-us/azure/container-registry/authenticate-aks-cross-tenant)|Enable when AKS cluster created or updated|Unattended pull to AKS cluster from registry in another Entra tenant|No, pull access only|Only available with AKS cluster|
|[Admin user](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-authentication?tabs=azure-cli#admin-account)|`docker login`|Interactive push/pull by individual developer or tester  <br>  <br>Portal deployment of image from registry to Azure App Service or Azure Container Instances|No, always pull and push access|Single account per registry, not recommended for multiple users|
|[Non-Microsoft Entra token-based repository permissions](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-token-based-repository-permissions)|`docker login`  <br>  <br>`az acr login` in Azure CLI  <br>  <br>`Connect-AzContainerRegistry` in Azure PowerShell  <br>  <br>[Kubernetes pull secret](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-auth-kubernetes)|Interactive push/pull to repository by individual developer or tester  <br>  <br>Unattended pull from repository by individual system or external device|Token-based repository permissions **does not** support Microsoft Entra RBAC role assignments.  <br>  <br>For Microsoft Entra-based repository permissions, see [Microsoft Entra attribute-based access control (ABAC) for **Microsoft Entra-based repository permissions**](https://learn.microsoft.com/en-us/azure/container-registry/container-registry-rbac-abac-repository-permissions) instead.|Not currently integrated with Microsoft Entra identity|

## Service principal
If you assign a [service principal](https://learn.microsoft.com/en-us/azure/active-directory/develop/app-objects-and-service-principals) to your registry, your application or service can use it for headless authentication.
Multiple service principals allow you to define different access for different applications.
ACR authentication token gets created upon login to the ACR, and is refreshed upon subsequent operations. The time to live for that token is 3 hours.

## Admin account
Each container registry includes an admin user account, which is disabled by default. You can enable the admin user and manage its credentials in the Azure portal, or by using the Azure CLI, Azure PowerShell, or other Azure tools. The admin account has full permissions to the registry.

The admin account is designed for a single user to access the registry, mainly for testing purposes. We don't recommend sharing the admin account credentials among multiple users.

If the admin account is enabled, you can pass the username and either password to the `docker login` command when prompted for basic authentication to the registry

```bash
docker login myregistry.azurecr.io
```

To enable the admin user for an existing registry, you can use the `--admin-enabled` parameter of the [az acr update](https://learn.microsoft.com/en-us/cli/azure/acr#az-acr-update) command in the Azure CLI:

```shell
az acr update -n <acrName> --admin-enabled true
```

To enable the admin user for an existing registry, you can use the `EnableAdminUser` parameter of the [Update-AzContainerRegistry](https://learn.microsoft.com/en-us/powershell/module/az.containerregistry/update-azcontainerregistry) command in Azure PowerShell:

```bash
Update-AzContainerRegistry -Name <acrName> -ResourceGroupName myResourceGroup -EnableAdminUser
```


## Create an alias of the image
Use [docker tag](https://docs.docker.com/engine/reference/commandline/tag/) to create an alias of the image with the fully qualified path to your registry. This example specifies the `samples` namespace to avoid clutter in the root of the registry.

```bash
docker tag nginx myregistry.azurecr.io/samples/nginx
```

## Push the image to your registry

```bash
docker push myregistry.azurecr.io/samples/nginx
```

## Pull the image from your registry

```bash
docker pull myregistry.azurecr.io/samples/nginx
```

## Start the Nginx container

```bash
docker run -it --rm -p 8080:80 myregistry.azurecr.io/samples/nginx
```

## Remove the image (optional)

```bash
docker rmi myregistry.azurecr.io/samples/nginx
```

To remove images from your Azure container registry, you can use the Azure CLI command [az acr repository delete](https://learn.microsoft.com/en-us/cli/azure/acr/repository#az-acr-repository-delete). For example, the following command deletes the manifest referenced by the `samples/nginx:latest` tag, any unique layer data, and all other tags referencing the manifest.

```bash
az acr repository delete --name myregistry --image samples/nginx:latest
```

The [Az.ContainerRegistry](https://learn.microsoft.com/en-us/powershell/module/az.containerregistry) Azure PowerShell module contains multiple commands for removing images from your container instance. [Remove-AzContainerRegistryRepository](https://learn.microsoft.com/en-us/powershell/module/az.containerregistry/remove-azcontainerregistryrepository) removes all images in a particular namespace such as `samples:nginx`, while [Remove-AzContainerRegistryManifest](https://learn.microsoft.com/en-us/powershell/module/az.containerregistry/remove-azcontainerregistrymanifest) removes a specific tag or manifest.

```bash
Remove-AzContainerRegistryRepository -RegistryName myregistry -Name samples/nginx
```

In the following example, you use the `Remove-AzContainerRegistryManifest` cmdlet to delete the manifest referenced by the `samples/nginx:latest` tag, any unique layer data, and all other tags referencing the manifest.

```bash
Remove-AzContainerRegistryManifest -RegistryName myregistry -RepositoryName samples/nginx -Tag latest
```