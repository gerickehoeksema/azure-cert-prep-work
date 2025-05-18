## 📋 Table of Contents

- [Overview](https://claude.ai/chat/6413088c-516c-4e15-ab41-39940aed0446#overview)
- [Key Concepts](https://claude.ai/chat/6413088c-516c-4e15-ab41-39940aed0446#key-concepts)
- [Authentication Methods](https://claude.ai/chat/6413088c-516c-4e15-ab41-39940aed0446#authentication-methods)
- [Managing Access](https://claude.ai/chat/6413088c-516c-4e15-ab41-39940aed0446#managing-access)
- [Creating and Managing Secrets](https://claude.ai/chat/6413088c-516c-4e15-ab41-39940aed0446#creating-and-managing-secrets)
- [Working with Keys](https://claude.ai/chat/6413088c-516c-4e15-ab41-39940aed0446#working-with-keys)
- [Managing Certificates](https://claude.ai/chat/6413088c-516c-4e15-ab41-39940aed0446#managing-certificates)
- [Azure Key Vault REST API](https://claude.ai/chat/6413088c-516c-4e15-ab41-39940aed0446#azure-key-vault-rest-api)
- [Application Integration](https://claude.ai/chat/6413088c-516c-4e15-ab41-39940aed0446#application-integration)
- [C# Implementation Examples](https://claude.ai/chat/6413088c-516c-4e15-ab41-39940aed0446#c-implementation-examples)
- [Backup and Recovery](https://claude.ai/chat/6413088c-516c-4e15-ab41-39940aed0446#backup-and-recovery)
- [Monitoring and Logging](https://claude.ai/chat/6413088c-516c-4e15-ab41-39940aed0446#monitoring-and-logging)
- [Performance Considerations](https://claude.ai/chat/6413088c-516c-4e15-ab41-39940aed0446#performance-considerations)
- [💡 Exam Tips and Gotchas](https://claude.ai/chat/6413088c-516c-4e15-ab41-39940aed0446#-exam-tips-and-gotchas)

## Overview

> 🎯 **Exam Focus**: Understanding Key Vault's role in the Azure ecosystem and how it secures application secrets

Azure Key Vault is a cloud service for securely storing and accessing secrets, keys, and certificates. It provides the following primary capabilities:

- **Secrets Management**: Securely store and control access to tokens, passwords, certificates, API keys, and other secrets
- **Key Management**: Create and control encryption keys
- **Certificate Management**: Provision, manage, and deploy public and private SSL/TLS certificates

## Key Concepts

- **Vault**: Container for secrets, keys, and certificates
- **Secret**: Any sensitive data (passwords, connection strings)
- **Key**: Cryptographic keys used for encryption/decryption
- **Certificate**: X.509 certificates used for authentication and encryption
- **Soft Delete**: Recovery option for deleted vaults and secrets
- **Purge Protection**: Additional protection against immediate permanent deletion

## Authentication Methods

Azure Key Vault supports multiple authentication mechanisms:

### 1. Managed Identity

> 🎯 **Exam Focus**: Understanding Managed Identity is crucial for the exam

```csharp
// Using DefaultAzureCredential (recommended approach)
var client = new SecretClient(new Uri("https://myvault.vault.azure.net"), new DefaultAzureCredential());
```

### 2. Service Principal with Client Secret

```csharp
// Using ClientSecretCredential
var client = new SecretClient(
    new Uri("https://myvault.vault.azure.net"),
    new ClientSecretCredential(tenantId, clientId, clientSecret));
```

### 3. User-assigned Managed Identity

```csharp
// Using ManagedIdentityCredential with user-assigned identity
var client = new SecretClient(
    new Uri("https://myvault.vault.azure.net"),
    new ManagedIdentityCredential(clientId));
```

## Managing Access

### RBAC (Role-Based Access Control)

> 🎯 **Exam Focus**: Know the built-in RBAC roles for Key Vault

Key built-in roles:

- **Key Vault Administrator**: Full access to manage all Key Vault resources
- **Key Vault Certificates Officer**: Perform any action on certificates
- **Key Vault Crypto Officer**: Perform any action on keys
- **Key Vault Crypto Service Encryption User**: Read metadata of keys and perform wrap/unwrap operations
- **Key Vault Crypto User**: Perform cryptographic operations using keys
- **Key Vault Reader**: Read Key Vault metadata
- **Key Vault Secrets Officer**: Perform any action on secrets
- **Key Vault Secrets User**: Read secret contents

**CLI Example**:

```bash
az role assignment create --role "Key Vault Secrets Officer" --assignee "user@example.com" --scope "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.KeyVault/vaults/{keyVaultName}"
```

**PowerShell Example**:

```powershell
New-AzRoleAssignment -SignInName "user@example.com" -RoleDefinitionName "Key Vault Certificates Officer" -Scope "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.KeyVault/vaults/{keyVaultName}"
```

### Access Policies (Legacy)

> ⚠️ **Exam Gotcha**: RBAC is recommended over access policies, but both might appear on the exam

```bash
az keyvault set-policy --name "{keyVaultName}" --object-id "{objectId}" --secret-permissions get list set delete
```

```powershell
Set-AzKeyVaultAccessPolicy -VaultName "{keyVaultName}" -ObjectId (Get-AzADUser -UserPrincipalName "user@example.com").Id -PermissionsToSecrets Get, List, Set, Delete
```

## Creating and Managing Secrets

### Creating a Key Vault

**CLI**:

```bash
az keyvault create --name "myKeyVault" --resource-group "myResourceGroup" --location "eastus" --sku "standard"
```

**PowerShell**:

```powershell
New-AzKeyVault -Name "myKeyVault" -ResourceGroupName "myResourceGroup" -Location "EastUS" -Sku "Standard"
```

### Adding and Retrieving Secrets

**CLI**:

```bash
# Set a secret
az keyvault secret set --vault-name "myKeyVault" --name "ExamplePassword" --value "hVFkk965BuUv"

# Get a secret
az keyvault secret show --name "ExamplePassword" --vault-name "myKeyVault" --query "value"
```

**C#**:

```csharp
// Setting a secret
var secretClient = new SecretClient(new Uri("https://myKeyVault.vault.azure.net"), new DefaultAzureCredential());
await secretClient.SetSecretAsync("ExamplePassword", "hVFkk965BuUv");

// Getting a secret
KeyVaultSecret secret = await secretClient.GetSecretAsync("ExamplePassword");
string secretValue = secret.Value;
```

### Managing Secret Versions

```csharp
// List all versions of a secret
Pageable<SecretProperties> secretVersions = secretClient.GetPropertiesOfSecretVersions("ExamplePassword");
foreach (SecretProperties secretVersion in secretVersions)
{
    Console.WriteLine($"Version: {secretVersion.Version}, Enabled: {secretVersion.Enabled}");
}

// Get a specific version
KeyVaultSecret specificVersion = await secretClient.GetSecretAsync("ExamplePassword", "4387e9f3d6e14c459867679a90fd0f79");
```

## Working with Keys

> 🎯 **Exam Focus**: Understand key operations, types, and protection levels

### Key Types

- **RSA**: Used for encryption/decryption and signing/verification
- **EC (Elliptic Curve)**: Used for signing/verification
- **Symmetric**: Used for encryption/decryption

### Protection Levels

- **Software**: Keys protected by software
- **HSM**: Keys protected by Hardware Security Modules (Premium tier)

### Creating and Managing Keys

**CLI**:

```bash
# Create an RSA key
az keyvault key create --vault-name "myKeyVault" --name "MyRSAKey" --kty "RSA" --size 2048

# Create an EC key
az keyvault key create --vault-name "myKeyVault" --name "MyECKey" --kty "EC" --curve "P-256"
```

**C#**:

```csharp
// Creating a key
var keyClient = new KeyClient(new Uri("https://myKeyVault.vault.azure.net"), new DefaultAzureCredential());
var keyOptions = new CreateRsaKeyOptions("MyRSAKey", hardwareProtected: false) { KeySize = 2048 };
KeyVaultKey key = await keyClient.CreateRsaKeyAsync(keyOptions);
```

### Key Operations

```csharp
// Encrypt
EncryptResult encryptResult = await cryptoClient.EncryptAsync(EncryptionAlgorithm.RsaOaep, plainText);

// Decrypt
DecryptResult decryptResult = await cryptoClient.DecryptAsync(EncryptionAlgorithm.RsaOaep, encryptResult.Ciphertext);

// Sign
SignResult signResult = await cryptoClient.SignAsync(SignatureAlgorithm.RS256, dataToSign);

// Verify
VerifyResult verifyResult = await cryptoClient.VerifyAsync(SignatureAlgorithm.RS256, dataToSign, signResult.Signature);
```

## Managing Certificates

> 🎯 **Exam Focus**: Certificate creation methods and management

### Certificate Creation Methods

- **Self-signed certificates**: Generated by Key Vault
- **Certificates from a Certificate Authority (CA)**: Integrated with supported CAs
- **Imported certificates**: Existing certificates uploaded to Key Vault

### Creating and Managing Certificates

**PowerShell**:

```powershell
# Create a certificate policy
$Policy = New-AzKeyVaultCertificatePolicy -SecretContentType "application/x-pkcs12" -SubjectName "CN=contoso.com" -IssuerName "Self" -ValidityInMonths 6 -ReuseKeyOnRenewal

# Create a certificate
Add-AzKeyVaultCertificate -VaultName "myKeyVault" -Name "ExampleCertificate" -CertificatePolicy $Policy

# Get a certificate
Get-AzKeyVaultCertificate -VaultName "myKeyVault" -Name "ExampleCertificate"
```

**C#**:

```csharp
// Creating a certificate
var certificateClient = new CertificateClient(new Uri("https://myKeyVault.vault.azure.net"), new DefaultAzureCredential());

var certificatePolicy = new CertificatePolicy(
    "Self",
    new SubjectAlternativeNames
    {
        DnsNames = { "contoso.com" }
    })
{
    ValidityInMonths = 12
};

await certificateClient.StartCreateCertificateAsync("ExampleCertificate", certificatePolicy);
```

## Azure Key Vault REST API

> 🎯 **Exam Focus**: Understanding API versions and common REST endpoints

### Common Endpoints

1. **Get Secret**:

```http
GET https://{vaultName}.vault.azure.net/secrets/{secretName}?api-version=7.4
```

2. **Get Secret Versions**:

```http
GET https://{vaultName}.vault.azure.net/secrets/{secretName}/versions?api-version=7.4
```

3. **Create/Update Secret**:

```http
PUT https://{vaultName}.vault.azure.net/secrets/{secretName}?api-version=7.4
```

4. **Get Key**:

```http
GET https://{vaultName}.vault.azure.net/keys/{keyName}?api-version=7.4
```

### URI Parameters

|Name|In|Required|Type|Description|
|---|---|---|---|---|
|secretName|path|True|string|The name of the secret.|
|vaultBaseUrl|path|True|string|The vault name, for example https://myvault.vault.azure.net.|
|api-version|query|True|string|Client API version.|
|maxresults|query||integer (1-25)|Maximum number of results to return in a page.|

## Application Integration

> 🎯 **Exam Focus**: How to integrate Key Vault with ASP.NET Core applications

### ASP.NET Core Configuration Integration

#### Package Installation

```bash
dotnet add package Azure.Security.KeyVault.Secrets
dotnet add package Azure.Identity
dotnet add package Microsoft.Extensions.Configuration.AzureKeyVault
```

#### Program.cs Configuration

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureAppConfiguration((context, config) =>
        {
            var builtConfig = config.Build();
            var keyVaultName = builtConfig["KeyVault:Name"];
            var keyVaultUri = $"https://{keyVaultName}.vault.azure.net/";
            
            config.AddAzureKeyVault(
                new Uri(keyVaultUri),
                new DefaultAzureCredential());
        })
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });
```

#### Accessing Secrets

```csharp
public class HomeController : Controller
{
    private readonly IConfiguration _configuration;

    public HomeController(IConfiguration configuration)
    {
        _configuration = configuration;
    }

    public IActionResult Index()
    {
        ViewData["Secret"] = _configuration["ExamplePassword"];
        return View();
    }
}
```

## C# Implementation Examples

### Azure SDK Packages

```csharp
// Add the following packages:
// Azure.Security.KeyVault.Secrets
// Azure.Security.KeyVault.Keys
// Azure.Security.KeyVault.Certificates
// Azure.Identity
```

### Secret Management Example

```csharp
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;
using System;
using System.Threading.Tasks;

public class KeyVaultSecretManager
{
    private readonly SecretClient _secretClient;

    public KeyVaultSecretManager(string keyVaultUrl)
    {
        _secretClient = new SecretClient(new Uri(keyVaultUrl), new DefaultAzureCredential());
    }

    public async Task<string> GetSecretAsync(string secretName)
    {
        try
        {
            KeyVaultSecret secret = await _secretClient.GetSecretAsync(secretName);
            return secret.Value;
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error retrieving secret: {ex.Message}");
            return null;
        }
    }

    public async Task SetSecretAsync(string secretName, string secretValue)
    {
        try
        {
            await _secretClient.SetSecretAsync(secretName, secretValue);
            Console.WriteLine($"Secret '{secretName}' set successfully.");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error setting secret: {ex.Message}");
        }
    }

    public async Task DeleteSecretAsync(string secretName)
    {
        try
        {
            DeleteSecretOperation operation = await _secretClient.StartDeleteSecretAsync(secretName);
            await operation.WaitForCompletionAsync();
            Console.WriteLine($"Secret '{secretName}' deleted successfully.");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error deleting secret: {ex.Message}");
        }
    }
}
```

## Backup and Recovery

> 🎯 **Exam Focus**: Understand soft delete and purge protection features

### Soft Delete

Soft delete is enabled by default for all Key Vaults and cannot be disabled.

**Configuring Soft Delete Retention**:

```bash
az keyvault create --name "myKeyVault" --resource-group "myResourceGroup" --location "eastus" --retention-days 90
```

### Recovering Deleted Secrets

**CLI**:

```bash
# List deleted secrets
az keyvault secret list-deleted --vault-name "myKeyVault"

# Recover a deleted secret
az keyvault secret recover --name "ExamplePassword" --vault-name "myKeyVault"
```

**C#**:

```csharp
// List deleted secrets
await foreach (DeletedSecret deletedSecret in secretClient.GetDeletedSecretsAsync())
{
    Console.WriteLine($"Deleted Secret: {deletedSecret.Name}, Recovery Id: {deletedSecret.RecoveryId}");
}

// Recover a deleted secret
await secretClient.StartRecoverDeletedSecretAsync("ExamplePassword");
```

### Purge Protection

When enabled, a vault or an object in a deleted state cannot be purged until the retention period has passed.

**Enabling Purge Protection**:

```bash
az keyvault update --name "myKeyVault" --resource-group "myResourceGroup" --enable-purge-protection true
```

## Monitoring and Logging

> 🎯 **Exam Focus**: Know how to configure diagnostics and monitor access

### Diagnostic Settings

**CLI**:

```bash
az monitor diagnostic-settings create --name "KeyVaultDiagnostics" \
    --resource "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.KeyVault/vaults/{keyVaultName}" \
    --logs '[{"category":"AuditEvent","enabled":true}]' \
    --metrics '[{"category":"AllMetrics","enabled":true}]' \
    --workspace "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.OperationalInsights/workspaces/{workspaceName}"
```

### Key events to monitor:

- Secret/key/certificate creation, modification, and deletion
- Vault access and authentication failures
- Cryptographic operations

## Performance Considerations

> 🎯 **Exam Focus**: Understanding performance limits and best practices

### Key Vault Limits

- **Transactions**: Standard - 200 requests/sec, Premium - 200 requests/sec
- **Keys**: 5120 max versions per key for RSA/EC keys
- **Secrets**: 25,000 max versions per secret name (across active and deleted secrets)
- **Certificates**: 25,000 max versions per certificate name

### Best Practices

- Use caching for frequently accessed secrets
- Implement retry policies with exponential backoff
- Use separate key vaults for different environments (dev, test, prod)
- Consider regional placement for latency-sensitive applications

## 💡 Exam Tips and Gotchas

### Top Exam Tips

1. **Authentication Methods**:
    
    - 🔍 The exam heavily focuses on using Managed Identity for authentication
    - Know the differences between DefaultAzureCredential and explicit credential types
2. **Access Control**:
    
    - 🔍 Understand the differences between RBAC and Access Policies
    - Know which RBAC roles grant which permissions
3. **Key Vault SKUs**:
    
    - Standard: Software-protected keys
    - Premium: HSM-protected keys
    - 🔍 Know which features are available in which SKU
4. **REST API**:
    
    - 🔍 Understand the API version parameter and common endpoints
    - Be familiar with URI parameter requirements
5. **C# SDK**:
    
    - 🔍 Know the different client classes (SecretClient, KeyClient, CertificateClient)
    - Understand async patterns for Key Vault operations

### Common Gotchas

1. ⚠️ **Soft Delete**: Enabled by default and cannot be disabled; recovery is possible within the retention period.
    
2. ⚠️ **Purge Protection**: Once enabled, it cannot be disabled.
    
3. ⚠️ **Access Policies vs. RBAC**:
    
    - Both can be used simultaneously but can cause confusion
    - RBAC is the recommended approach
4. ⚠️ **Key Vault Firewall**:
    
    - If enabled without proper network rules, your application might lose access
    - Consider private endpoint connections for secure access
5. ⚠️ **Certificate Renewal**:
    
    - Automatic renewal requires configuration of notification email
    - Self-signed certificates can be renewed automatically; CA-issued certificates require additional steps
6. ⚠️ **API Versions**:
    
    - Always specify the API version in REST requests
    - Different API versions may have different features or behaviors
7. ⚠️ **Managed Identity Configuration**:
    
    - System-assigned vs. User-assigned have different implementation approaches
    - Not all Azure services support Managed Identity
8. ⚠️ **DefaultAzureCredential**:
    
    - Tries multiple authentication methods in sequence
    - Different behavior in development vs. production environments

---

## Practice Questions

1. Which of the following authentication methods is recommended for an Azure App Service accessing Key Vault?
    
    - A) Service Principal with Client Secret
    - B) Managed Identity
    - C) Certificate-based authentication
    - D) Username/Password
2. Which RBAC role allows a user to create, but not delete, secrets in a Key Vault?
    
    - A) Key Vault Administrator
    - B) Key Vault Secrets User
    - C) Key Vault Secrets Officer
    - D) Key Vault Reader
3. When using the REST API to retrieve a secret version, what is the correct endpoint format?
    
    - A) GET https://{vaultName}.vault.azure.net/secrets/{secretName}?api-version=7.4
    - B) GET https://{vaultName}.vault.azure.net/secrets/{secretName}/versions/{version}?api-version=7.4
    - C) GET https://{vaultName}.vault.azure.net/secrets/{secretName}?version={version}&api-version=7.4
    - D) GET https://{vaultName}.vault.azure.net/secrets/versions/{secretName}?api-version=7.4
4. Which C# class is used to retrieve secrets from Azure Key Vault?
    
    - A) KeyClient
    - B) SecretClient
    - C) CertificateClient
    - D) AzureKeyVaultClient
5. What happens when you delete a secret from a Key Vault with soft-delete enabled?
    
    - A) The secret is permanently deleted immediately
    - B) The secret is marked as deleted but can be recovered within the retention period
    - C) The secret is archived but still accessible
    - D) The secret requires approval before being deleted

**Answers**: 1-B, 2-C, 3-B, 4-B, 5-B