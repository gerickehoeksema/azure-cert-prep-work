## 📌 Overview
Azure Container Registry (ACR) is a managed, private Docker registry service that stores and manages container images and related artifacts for all types of container deployments. It's fully integrated with other Azure services and provides security, high availability, and reliable content delivery for your containerized applications.

## ✨ Key Features

### 🔒 Security
- **Private registry**: Store and manage container images in a secure, private environment
- **Integrated authentication**: Authentication through Azure Active Directory (Microsoft Entra ID)
- **Image encryption**: Encrypts images at rest
- **Network security**: Restrict access using virtual networks and firewalls
- **Content trust**: Enable Docker Content Trust for image signing and verification

### ⚡ Performance and Scalability
- **Geo-replication**: Replicate registry across multiple Azure regions
- **Tiered service**: Basic, Standard, and Premium tiers with increasing storage and throughput
- **Scalable storage**: Automatically scales with your needs
- **Fast deployment**: Quick pull/push operations for global deployments

### 🔄 DevOps Integration
- **CI/CD integration**: Works with Azure DevOps, GitHub Actions, and other CI/CD platforms
- **Build services**: Build container images in Azure (ACR Tasks)
- **Webhooks**: Trigger events when images are pushed or updated
- **Automated builds**: Trigger builds when base images are updated or on code commits

### 🏆 Enterprise Features (Premium Tier)
- **Geo-replication**: Store images across multiple regions
- **Private link**: Access registries privately from your VNet
- **Content trust**: Sign and verify images
- **Image quarantine**: Review images before they're available

## 🔐 Authentication Options

| Method | How to Authenticate | Use Cases | Limitations |
|--------|---------------------|-----------|------------|
| 🧑‍💻 **Individual Microsoft Entra ID** | `az acr login` | Interactive development | Token expires after 3 hours |
| 🤖 **Service Principal** | `docker login` with SP credentials | CI/CD, unattended access | Password expires after 1 year by default |
| 🔑 **Managed Identity** | Azure services use built-in auth | Azure services integration | Only from supported Azure services |
| 🔄 **AKS Integration** | Attach registry to AKS | Kubernetes deployments | Pull access only |
| 👑 **Admin Account** | `docker login` with admin creds | Testing, simple setups | Not recommended for production or multiple users |
| 🎫 **Token-based Repository** | `docker login` with token | Granular repository access | Not integrated with Microsoft Entra ID |

## 🔰 Built-in RBAC Roles

| Role | Permissions | Use Case |
|------|-------------|----------|
| **Owner** | Full control | Administrators |
| **Contributor** | Push/pull images, everything except role assignments | DevOps teams |
| **Reader** | Pull images and read properties | Consumption-only scenarios |
| **AcrPush** | Push and pull images only | CI/CD pipelines, deployment teams |
| **AcrPull** | Pull images only | Application deployments, consumption |
| **AcrDelete** | Delete repository data | Cleanup operations |
| **AcrImageSigner** | Sign images (content trust) | Content verification |

## 📝 Common Operations

### Creating a Registry
```bash
# Azure CLI
az acr create --resource-group myResourceGroup --name myRegistry --sku Basic

# PowerShell
New-AzContainerRegistry -ResourceGroupName myResourceGroup -Name myRegistry -Sku Basic
```

### Importing Images
```bash
az acr import \
  --name myregistry \
  --source docker.io/library/hello-world:latest \
  --image hello-world:latest \
  --username <Docker Hub username> \
  --password <Docker Hub token>
```

### Tagging and Pushing Images
```bash
# Login to registry
az acr login --name myregistry

# Tag the image
docker tag nginx myregistry.azurecr.io/samples/nginx

# Push the image
docker push myregistry.azurecr.io/samples/nginx
```

### Pulling Images
```bash
docker pull myregistry.azurecr.io/samples/nginx
```

### Removing Images
```bash
# Using Azure CLI
az acr repository delete --name myregistry --image samples/nginx:latest

# Using PowerShell
Remove-AzContainerRegistryManifest -RegistryName myregistry -RepositoryName samples/nginx -Tag latest
```

### Enabling Admin Account
```bash
# Using Azure CLI
az acr update -n myregistry --admin-enabled true

# Using PowerShell
Update-AzContainerRegistry -Name myregistry -ResourceGroupName myResourceGroup -EnableAdminUser
```

## 🚨 ACR Tasks

ACR Tasks provides cloud-based container image building capabilities and automates OS and framework patching.

### Task Types
- **Quick tasks**: One-off container image builds and pushes (`az acr build`)
- **Automatically triggered tasks**: Triggered by source code updates, base image updates, or scheduled runs
- **Multi-step tasks**: Extend build and push with multi-step, multi-container workflows

### Example: Quick Build
```bash
az acr build --registry myregistry --image myapp:latest .
```

### Example: Automatically Triggered Task
```bash
az acr task create \
  --registry myregistry \
  --name buildwebapp \
  --image webimage \
  --context https://github.com/username/webproject.git \
  --file Dockerfile \
  --git-access-token <token>
```

## 📊 Pricing Tiers

| Feature | Basic | Standard | Premium |
|---------|-------|----------|---------|
| Storage | 10 GB | 100 GB | 500 GB |
| Web hooks | Yes | Yes | Yes |
| Authentication | Azure AD/Service Principal | Azure AD/Service Principal | Azure AD/Service Principal |
| Geo-replication | No | No | Yes |
| Private Link | No | No | Yes |
| Content Trust | No | No | Yes |
| Repository Permissions | No | No | Yes |
| Image Quarantine | No | No | Yes |

## 💡 Exam Tips & Gotchas

### Important Exam Tips
1. **Authentication Scenarios**: Understand which authentication method to use for different scenarios (e.g., service principal for CI/CD, Managed Identity for Azure services)
   
2. **Role-Based Access**: Know which built-in role to assign for specific use cases (AcrPush for CI/CD, AcrPull for deployments)

3. **Registry SKUs**: Remember the features available in each tier (e.g., geo-replication only in Premium)

4. **Service Integration**: Understand how to integrate ACR with other Azure services, especially AKS

5. **Repository Namespaces**: Know how to organize images using namespaces (e.g., myregistry.azurecr.io/team/project/image)

6. **Authentication Token TTL**: Remember that tokens expire after 3 hours by default

7. **Multi-Arch Images**: Understand how to work with multi-architecture images and manifests

8. **ACR Tasks**: Know the difference between quick tasks, triggered tasks, and multi-step tasks

### Common Gotchas

1. **Admin Account**: Admin account provides full access and should not be used in production scenarios
   
2. **Service Principal**: Service principal passwords expire after 1 year by default, causing authentication issues

3. **Authentication Scope**: Token-based repository permissions are not integrated with Azure AD roles

4. **Cross-Region Authentication**: When using geo-replication, authentication still happens at the registry URL, not the replica

5. **AKS Integration**: When using ACR with AKS, ensure proper role assignment for the AKS service principal/managed identity

6. **Image References**: Always use fully qualified image names (myregistry.azurecr.io/image:tag)

7. **Repository Deletion**: Deleting an image with `az acr repository delete` removes all tags pointing to that manifest

8. **CI/CD Security**: Never store service principal credentials in source control; use Azure KeyVault or pipeline variable groups

9. **Import vs. Pull/Push**: Using `az acr import` doesn't require a local Docker installation, unlike pull/push

10. **Webhooks Configuration**: Webhooks trigger on push operations but not on imports by default

## 🧪 Practice Scenarios

1. A company needs to deploy a CI/CD pipeline that automatically builds and pushes images to ACR. Which authentication method and role provides the principle of least privilege?
   - Answer: Service Principal with AcrPush role

2. Your organization needs to distribute container images across multiple Azure regions for low-latency access. Which ACR tier and feature should you use?
   - Answer: Premium tier with Geo-replication

3. You need to implement a solution that allows external devices to pull specific images from your registry without using Azure AD. Which feature should you use?
   - Answer: Token-based repository permissions

4. A company wants to ensure that all container images are scanned for vulnerabilities before they're available for deployment. Which feature should they enable?
   - Answer: Image quarantine (Premium tier)

5. You need to configure an AKS cluster to pull images from an ACR in another Azure tenant. Which authentication method should you use?
   - Answer: AKS cluster service principal
