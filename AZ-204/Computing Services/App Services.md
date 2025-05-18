## 📌 Overview
Azure App Service is a fully managed platform for building, deploying, and scaling web apps. It supports multiple languages and frameworks such as .NET, .NET Core, Java, Ruby, Node.js, PHP, and Python.

---

## 📦 App Service Plans
- Defines **compute resources** for an app.
- Determines:
  - Pricing tier
  - VM size
  - Scaling limits
- **Types**:
  - Free (F1)
  - Shared (D1)
  - Basic (B1, B2, B3)
  - Standard (S1, S2, S3)
  - Premium (P1V3, P2V3, P3V3)
  - Isolated (I1, I2, I3) for high-security environments

---

## 🚀 Deployment Options
- **Local Git / GitHub / Azure DevOps**
- **ZIP Deploy**
- **FTP / FTPS**
- **Azure CLI / PowerShell**
- **Docker Container Support**

---

## ⚙️ Configuration & Settings
- **Application Settings**: Key-value pairs available as environment variables
- **Connection Strings**: Secure database connection info
- **App Service Editor**: Lightweight in-browser editor
- **Custom Domains & SSL Certificates**
- **Authentication / Authorization (Easy Auth)**:
  - Integrates with Azure AD, Facebook, Google, Twitter, Microsoft

---

## 📊 Monitoring & Diagnostics
- **Application Insights** for performance, logs, telemetry
- **App Service Logs**:
  - Application logs
  - Web server logs
  - Detailed error messages
- **Diagnostic Logs to Storage / Event Hub**

---

## 📈 Scaling Options
- **Manual Scaling**
- **Autoscale Rules**:
  - Based on CPU, memory, HTTP queue length, custom metrics
- **Scale Up**: Change pricing tier for more powerful VMs
- **Scale Out**: Increase number of instances

---

## 🐳 App Service for Containers
- Deploy Docker containers directly
- Supports custom container images from:
  - Azure Container Registry (ACR)
  - Docker Hub
  - Private registries
- Multi-container apps using Docker Compose (Linux only)

---

## 🔐 Security Considerations
- **Managed Identity** for Azure resource access
- **VNet Integration** (regional & gateway required)
- **Private Endpoints**
- **Custom Authentication Providers**

---

## 📚 Key CLI Commands
```bash
# Create a resource group
az group create --name myResourceGroup --location "East US"

# Create an App Service plan
az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku B1

# Create a web app
az webapp create --resource-group myResourceGroup --plan myAppServicePlan --name myWebApp --runtime "DOTNET|8.0"

# Deploy using ZIP
az webapp deployment source config-zip --resource-group myResourceGroup --name myWebApp --src myApp.zip
```

---

## 📖 Pro Tips
- Always enable **Application Insights** for production apps.
- Use **deployment slots** for zero-downtime deployments.
- Implement **custom domains & SSL** for public-facing apps.
- Use **Autoscale rules** to optimize performance and cost.
- Integrate with **Azure Key Vault** for secure secret management.
