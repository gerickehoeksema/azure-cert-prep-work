# Quickstart: Create a Windows virtual machine with the Azure CLI

## Create a resource group

```bash
resourcegroup="myResourceGroupCLI"
location="westus3"
az group create 
	--name $resourcegroup 
	--location $location
```

## Create virtual machine

```bash
vmname="myVM"
username="azureuser"
az vm create 
	--resource-group $resourcegroup 
	--name $vmname 
	--image Win2022AzureEditionCore 
	--public-ip-sku Standard 
	--admin-username $username
```

## Install web server

```bash
az vm run-command invoke -g $resourcegroup -n $vmname 
	--command-id RunPowerShellScript 
	--scripts "Install-WindowsFeature -name Web-Server -IncludeManagementTools"
```

## Open port 80 for web traffic
By default, only RDP connections are opened when you create a Windows VM in Azure. Use [az vm open-port](https://learn.microsoft.com/en-us/cli/azure/vm) to open TCP port 80 for use with the IIS web server:

```bash
az vm open-port 
	--port 80 
	--resource-group $resourcegroup 
	--name $vmname
```

## Clean up resources

```bash
az group delete --name $resourcegroup
```