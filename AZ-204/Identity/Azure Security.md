# AZ-204 Study Guide: Azure Security & Virtual Machines

## 🔐 Azure Security - Key Concepts

### Azure Key Vault 🗝️

#### Core Functionality

- **Secure Storage**: Stores application secrets, keys, certificates, and passwords
- **Access Control**: Uses Azure RBAC for granular access management
- **Integration**: Seamlessly integrates with other Azure services
- **Monitoring**: Provides logging and auditing capabilities

#### Key Vault APIs in C#

```csharp
// Install the client library via NuGet
// dotnet add package Azure.Security.KeyVault.Secrets

// Create a client
var client = new SecretClient(new Uri("https://your-key-vault-name.vault.azure.net/"), new DefaultAzureCredential());

// Set a secret
await client.SetSecretAsync("MySecretName", "MySecretValue");

// Get a secret
KeyVaultSecret secret = await client.GetSecretAsync("MySecretName");
string secretValue = secret.Value;
```

#### Managed Service Identity with Key Vault

```csharp
// Access Key Vault with managed identity (no credentials in code)
var kvClient = new SecretClient(
    new Uri($"https://your-keyvault.vault.azure.net/"),
    new DefaultAzureCredential());
```

#### 📝 Exam Tips

- **Authentication Methods**: Know the different authentication options (service principal, managed identity, access policies vs. RBAC)
- **Soft Delete**: Understand how soft-delete and purge protection work
- **Recovery**: Know the process for recovering deleted secrets
- **Network Security**: Understand private endpoints and network service endpoints

#### ⚠️ Common Gotchas

- Access policies are not the same as Azure RBAC - know the difference
- Key Vault references in App Configuration require specific setup
- Secret versions are immutable - you can't modify an existing version
- Understand the differences between keys, secrets, and certificates

---

### Managed Identities 👤

#### Types

- **System-assigned**: Tied to service lifecycle, auto-created/deleted
- **User-assigned**: Independent lifecycle, can be shared across resources

#### C# Implementation

```csharp
// Using a managed identity to access Azure resources
// No credentials in code!
var blobClient = new BlobServiceClient(
    new Uri("https://yourstorageaccount.blob.core.windows.net"),
    new DefaultAzureCredential());
```

#### 📝 Exam Tips

- **TokenCredential**: Understand how DefaultAzureCredential works and its credential chain
- **Service Support**: Know which Azure services support managed identities
- **RBAC**: Understand how to grant permissions to managed identities

#### ⚠️ Common Gotchas

- System-assigned identities are deleted when the resource is deleted
- User-assigned identities must be explicitly assigned to services
- Not all Azure services support managed identities
- Permissions take time to propagate after assignment

---

### Azure AD Authentication 🔑

#### Authentication Methods

- **Interactive**: User signs in with credentials
- **Service Principal**: Application identity with credentials
- **Managed Identity**: Azure-managed service identity
- **Device Code**: For devices without browsers

#### C# Implementation with MSAL

```csharp
// Install the MSAL package
// dotnet add package Microsoft.Identity.Client

var app = PublicClientApplicationBuilder
    .Create("your-client-id")
    .WithAuthority(AzureCloudInstance.AzurePublic, "your-tenant-id")
    .WithRedirectUri("http://localhost")
    .Build();
    
var scopes = new[] { "https://management.azure.com/.default" };

try
{
    var result = await app.AcquireTokenInteractive(scopes).ExecuteAsync();
    string accessToken = result.AccessToken;
    // Use the token to call Azure services
}
catch (Exception ex)
{
    Console.WriteLine($"Error acquiring token: {ex.Message}");
}
```

#### 📝 Exam Tips

- **Token Cache**: Understand token caching mechanisms
- **Scopes**: Know common resource scopes (.default, User.Read, etc.)
- **Confidential vs. Public Clients**: Understand the differences
- **B2C vs. B2B**: Know the differences between B2C and B2B scenarios

#### ⚠️ Common Gotchas

- Tokens expire - implement proper refresh token handling
- Scope format must be correct (https://resourcename/scope)
- Be careful with storing and handling credentials
- Client ID != Application ID in some contexts

---

### Azure AD B2C 🧑‍🤝‍🧑

#### Key Features

- **Custom Policies**: Advanced user journeys with Identity Experience Framework
- **User Flows**: Predefined, configurable authentication experiences
- **External Identity Providers**: Google, Facebook, Twitter, etc.

#### C# Implementation

```csharp
// Configure authentication in ASP.NET Core
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApi(options =>
    {
        Configuration.Bind("AzureAdB2C", options);
        options.TokenValidationParameters.NameClaimType = "name";
    },
    options => { Configuration.Bind("AzureAdB2C", options); });
```

#### 📝 Exam Tips

- **User Flow Types**: Know the different types (sign up/in, profile editing, password reset)
- **Tokens**: Understand ID tokens vs. access tokens
- **Customization**: Know how to customize the UI with custom HTML/CSS
- **Multi-tenancy**: Understand B2C vs. regular Azure AD multi-tenant apps

#### ⚠️ Common Gotchas

- B2C tenants are separate from regular Azure AD tenants
- Custom policies can be complex to implement
- UI customization has limitations based on the method used
- Password complexity rules may differ from regular Azure AD

---

## 🖥️ Azure Virtual Machines

### VM Creation and Management 🚀

#### Deployment Models

- **ARM Templates**: Infrastructure as code
- **PowerShell/CLI**: Scripted deployment
- **Portal**: Manual deployment
- **Azure SDK**: Programmatic deployment

#### C# Implementation with Azure SDK

```csharp
// Install the Azure Management libraries
// dotnet add package Microsoft.Azure.Management.Compute
// dotnet add package Microsoft.Azure.Management.Network
// dotnet add package Microsoft.Azure.Management.ResourceManager.Fluent

var azure = Microsoft.Azure.Management.Fluent.Azure
    .Configure()
    .WithDefaultCredentials()
    .WithDefaultSubscription();

// Create a VM
var vm = await azure.VirtualMachines.Define("myVM")
    .WithRegion(Region.USWest)
    .WithExistingResourceGroup("myResourceGroup")
    .WithLatestWindowsImage("MicrosoftWindowsServer", "WindowsServer", "2019-Datacenter")
    .WithAdminUsername("adminUser")
    .WithAdminPassword("P@ssw0rd123!")
    .WithSize(VirtualMachineSizeTypes.StandardD2V3)
    .CreateAsync();
```

#### 📝 Exam Tips

- **VM Sizes**: Understand the different VM series and sizes
- **Availability**: Know about availability sets, zones, and scale sets
- **OS Disks**: Understand managed disks, encryption options
- **Network Configuration**: Virtual networks, NSGs, public IPs

#### ⚠️ Common Gotchas

- VM names have specific length and character restrictions
- Not all VM sizes are available in all regions
- Some VM series require special quota approvals
- Deallocated VMs still incur storage costs

---

### VM Scale Sets (VMSS) 📈

#### Key Features

- **Auto-scaling**: Scale based on metrics or schedule
- **Load Balancing**: Automatic distribution of traffic
- **Rolling Updates**: Update VMs without downtime
- **Instance Protection**: Prevent specific instances from scaling in

#### C# Implementation

```csharp
// Create a VM Scale Set
var vmss = await azure.VirtualMachineScaleSets.Define("myScaleSet")
    .WithRegion(Region.USWest)
    .WithExistingResourceGroup("myResourceGroup")
    .WithSku(VirtualMachineScaleSetSkuTypes.StandardD2V3)
    .WithLatestWindowsImage("MicrosoftWindowsServer", "WindowsServer", "2019-Datacenter")
    .WithAdminUsername("adminUser")
    .WithAdminPassword("P@ssw0rd123!")
    .WithCapacity(2)
    .WithAutomaticOSUpgrade(true)
    .CreateAsync();
```

#### Autoscale Configuration

```csharp
// Add autoscale settings
var autoscaleProfile = new AutoscaleProfile
{
    Name = "autoscale-profile",
    Capacity = new ScaleCapacity
    {
        Default = "2",
        Minimum = "1",
        Maximum = "10"
    },
    Rules = new List<AutoscaleRule>
    {
        new AutoscaleRule
        {
            MetricTrigger = new MetricTrigger
            {
                MetricName = "Percentage CPU",
                MetricResourceUri = vmss.Id,
                TimeGrain = TimeSpan.FromMinutes(1),
                Statistic = "Average",
                TimeWindow = TimeSpan.FromMinutes(5),
                TimeAggregation = "Average",
                Operator = "GreaterThan",
                Threshold = 70
            },
            ScaleAction = new ScaleAction
            {
                Direction = "Increase",
                Type = "ChangeCount",
                Value = "1",
                Cooldown = TimeSpan.FromMinutes(5)
            }
        }
    }
};
```

#### 📝 Exam Tips

- **Scaling Policies**: Understand the different scaling options (schedule-based, metric-based)
- **Instance Protection**: Know how to protect specific instances
- **Update Policies**: Understand automatic OS upgrades and application upgrades
- **Custom Images**: Know how to use custom images with scale sets

#### ⚠️ Common Gotchas

- Scale in and scale out have separate rules and settings
- Instance protection can prevent scaling below a certain number of instances
- Rolling upgrades require specific health probe configuration
- Autoscale can affect your budget if not configured properly

---

### VM Extensions 🧩

#### Common Extensions

- **Custom Script**: Run scripts on VMs
- **DSC**: Apply PowerShell DSC configurations
- **Diagnostics**: Collect diagnostic data
- **OMS Agent**: Monitor VMs with Log Analytics

#### C# Implementation

```csharp
// Add a custom script extension to a VM
await azure.VirtualMachines.GetById(vmId)
    .Update()
    .DefineNewExtension("CustomScriptExtension")
    .WithPublisher("Microsoft.Compute")
    .WithType("CustomScriptExtension")
    .WithVersion("1.10")
    .WithMinorVersionAutoUpgrade()
    .WithPublicSetting("fileUris", new List<string> { "https://mystorageaccount.blob.core.windows.net/scripts/setup.ps1" })
    .WithPublicSetting("commandToExecute", "powershell -ExecutionPolicy Unrestricted -File setup.ps1")
    .Attach()
    .ApplyAsync();
```

#### 📝 Exam Tips

- **Extension Sequencing**: Understand how extensions are applied
- **Dependency Agents**: Know how to install dependency agents
- **Error Handling**: Understand extension failure scenarios
- **Azure Policy**: Understand how policies can automatically apply extensions

#### ⚠️ Common Gotchas

- Extensions run with system privileges
- Some extensions are platform-specific (Windows vs. Linux)
- Custom Script extensions have timeouts
- Extensions can interfere with each other

---

### VM Backup and Recovery 🔄

#### Azure Backup

- **Recovery Services Vault**: Central storage for backups
- **Backup Policies**: Define backup frequency and retention
- **Application-Consistent Backups**: VSS integration for Windows

#### C# Implementation

```csharp
// Create a Recovery Services Vault
var vault = await azure.RecoveryServicesVaults.Define("myVault")
    .WithRegion(Region.USWest)
    .WithExistingResourceGroup("myResourceGroup")
    .WithSku(SkuName.Standard)
    .CreateAsync();

// Configure backup policy
var policy = await azure.BackupPolicies.Define("DailyPolicy")
    .WithBackupManagement(BackupManagementType.AzureIaasVM)
    .WithSchedule(SchedulePolicy.Daily)
    .WithRetention(RetentionPolicy.Daily, 30)
    .WithRetention(RetentionPolicy.Weekly, 4)
    .WithRetention(RetentionPolicy.Monthly, 12)
    .WithInstantRpRetention(InstantRpRetentionRangeInDays.Five)
    .CreateAsync();

// Configure backup
await azure.VirtualMachines.GetById(vmId)
    .EnableBackup(vault.Id, policy.Id);
```

#### 📝 Exam Tips

- **Recovery Points**: Understand the different types
- **Cross-Region Restore**: Know how to set up and use
- **Instant Restore**: Understand its limitations
- **Soft Delete**: Know how it works for backups

#### ⚠️ Common Gotchas

- Backup policies cannot be modified after creation
- Some VM configurations might prevent successful backups
- Backup storage costs can add up quickly with long retention periods
- Restores can be time-consuming for large VMs

---

### VM Security 🛡️

#### Disk Encryption

- **Azure Disk Encryption (ADE)**: BitLocker for Windows, DM-Crypt for Linux
- **Server-Side Encryption (SSE)**: Automatic encryption at rest
- **Customer-Managed Keys**: BYO encryption keys via Key Vault

#### Network Security

- **Network Security Groups (NSGs)**: Traffic filtering
- **Application Security Groups (ASGs)**: Logical grouping
- **Private IP Addressing**: Secure communication
- **Just-In-Time Access**: Temporary management port access

#### C# Implementation for Disk Encryption

```csharp
// Enable disk encryption on a VM
await azure.VirtualMachines.GetById(vmId)
    .Update()
    .WithOSDiskEncryption(diskEncryptionSetId)
    .ApplyAsync();
```

#### 📝 Exam Tips

- **Key Encryption Key (KEK)**: Understand how KEKs work with disk encryption
- **Encryption Prerequisites**: Know what's required before enabling encryption
- **NSG Flow Logs**: Understand how to enable and analyze
- **Azure Security Center**: Know how it integrates with VMs

#### ⚠️ Common Gotchas

- Not all VM sizes support encryption
- Encryption performance impact can be significant
- NSGs are stateful (return traffic is allowed)
- JIT access requires Azure Security Center standard tier

---

## 📚 Additional Resources

### Microsoft Learn Paths

- [Implement Azure security](https://docs.microsoft.com/en-us/learn/paths/implement-azure-security/)
- [Deploy and manage Azure compute resources](https://docs.microsoft.com/en-us/learn/paths/deploy-manage-azure-compute-resources/)

### Documentation

- [Azure Key Vault documentation](https://docs.microsoft.com/en-us/azure/key-vault/)
- [Managed identities for Azure resources](https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/)
- [Virtual machines in Azure](https://docs.microsoft.com/en-us/azure/virtual-machines/)

### Practice Exams

- [Microsoft Official Practice Test](https://www.mindhub.com/az-204-developing-solutions-for-microsoft-azure-microsoft-official-practice-test/p/MU-AZ-204)
- [Whizlabs AZ-204 Practice Tests](https://www.whizlabs.com/microsoft-azure-certification-az-204/)

---

## 🎯 Final Exam Preparation Checklist

- [ ] Review all security concepts (Key Vault, Managed Identities, etc.)
- [ ] Practice VM creation and configuration through various methods
- [ ] Understand VM Scale Sets and autoscaling rules
- [ ] Practice implementing VM extensions
- [ ] Review backup and recovery options
- [ ] Understand VM security best practices
- [ ] Take practice exams and review incorrect answers
- [ ] Review Microsoft documentation for any updates

---

## 💡 Top 10 Exam-Day Tips

1. **Read carefully**: Questions may have subtle wording that changes the correct answer
2. **Eliminate obvious wrong answers** first to improve your chances
3. **Watch for "not" questions** that ask which option is incorrect
4. **Pay attention to costs** - sometimes the question asks for the most cost-effective solution
5. **Look for Azure-specific terminology** that may give hints to the correct answer
6. **Time management** is crucial - don't spend too long on any single question
7. **Mark questions for review** if you're unsure and come back to them
8. **Check for "all that apply"** questions where multiple answers may be correct
9. **Consider the context** of the scenario - enterprise vs. small business may change the answer
10. **Remember the exam objectives** - if a question seems off-topic, you might be misinterpreting it
