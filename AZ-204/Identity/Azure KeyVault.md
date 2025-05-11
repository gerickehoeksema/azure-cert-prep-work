## Create a key vault

```bash
az keyvault create 
	--name "<your-unique-keyvault-name>" 
	--resource-group "myResourceGroup"
```

```bash
New-AzKeyVault 
	-Name "<your-unique-keyvault-name>" 
	-ResourceGroupName "myResourceGroup" 
	-Location "EastUS"
```

## Give your user account permissions to manage secrets in Key Vault
To gain permissions to your key vault through [Role-Based Access Control (RBAC)](https://learn.microsoft.com/en-us/azure/key-vault/general/rbac-guide), assign a role to your "User Principal Name" (UPN) 

```bash
az role assignment create 
	--role "Key Vault Secrets Officer" 
	--assignee "<upn>" 
	--scope "/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.KeyVault/vaults/<your-unique-keyvault-name>"
```

```bash
New-AzRoleAssignment 
	-SignInName "<upn>" 
	-RoleDefinitionName "Key Vault Certificates Officer" 
	-Scope "/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.KeyVault/vaults/<your-unique-keyvault-name>"
```

## Add a secret to Key Vault

```bash
az keyvault secret set 
	--vault-name "<your-unique-keyvault-name>" 
	--name "ExamplePassword" 
	--value "hVFkk965BuUv"
```

## Retrieve a secret from Key Vault

```bash
az keyvault secret show 
	--name "ExamplePassword" 
	--vault-name "<your-unique-keyvault-name>" 
	--query "value"
```

## Add a certificate to Key Vault

```bash
$Policy = New-AzKeyVaultCertificatePolicy 
	-SecretContentType "application/x-pkcs12" 
	-SubjectName "CN=contoso.com" 
	-IssuerName "Self" 
	-ValidityInMonths 6 
	-ReuseKeyOnRenewal

Add-AzKeyVaultCertificate 
	-VaultName "<your-unique-keyvault-name>" 
	-Name "ExampleCertificate" 
	-CertificatePolicy $Policy
```

```bash
Get-AzKeyVaultCertificate 
	-VaultName "<your-unique-keyvault-name>" 
	-Name "ExampleCertificate"
```

# Get Secret Versions

```http
GET {vaultBaseUrl}/secrets/{secret-name}/versions?api-version=7.4
```

With optional parameters:

```http
GET {vaultBaseUrl}/secrets/{secret-name}/versions?maxresults={maxresults}&api-version=7.4
```

## URI Parameters

|Name|In|Required|Type|Description|
|---|---|---|---|---|
|secret-name|path|True|string|The name of the secret.|
|vaultBaseUrl|path|True|string|The vault name, for example [https://myvault.vault.azure.net](https://myvault.vault.azure.net/).|
|api-version|query|True|string|Client API version.|
|maxresults|query||integer (int32)<br><br>minimum: 1  <br>maximum: 25|Maximum number of results to return in a page. If not specified, the service will return up to 25 results.|