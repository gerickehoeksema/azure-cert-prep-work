# 🚀 AZ-204 Comprehensive Study Guide: Azure App Services

## 📌 Overview

Azure App Service is a fully managed platform for building, deploying, and scaling web apps. It supports multiple languages and frameworks such as .NET, .NET Core, Java, Ruby, Node.js, PHP, and Python.

> 💡 **Exam Tip**: Understand the App Service components and how they interact within Azure's PaaS offerings. App Service is a core service tested heavily in AZ-204.

---

## 📦 App Service Plans

- Defines **compute resources** for an app.
- Determines:
    - Pricing tier
    - VM size
    - Scaling limits
- **Types**:
    - Free (F1) - 60 CPU minutes/day, no SSL, no custom domains
    - Shared (D1) - 240 CPU minutes/day, no SSL, no custom domains
    - Basic (B1, B2, B3) - Dev/test workloads, manual scale
    - Standard (S1, S2, S3) - Production workloads, autoscale (up to 10 instances)
    - Premium (P1V3, P2V3, P3V3) - Enhanced performance, autoscale (up to 30 instances)
    - Isolated (I1, I2, I3) - High-security environments with ASE (App Service Environment)

> ⚠️ **Gotcha**: The exam often tests on knowing which tiers support specific features. Remember that autoscaling is only available in Standard tier and above, and deployment slots are limited in Basic tier.

### App Service Plan Isolation Models

- **Multi-tenant**: Default service hosting where multiple customers share the same infrastructure
- **App Service Environment (ASE)**: Fully isolated and dedicated environment for securely running apps at high scale

> 💡 **Exam Tip**: Know the key differences between ASE v2 and v3, especially around scaling and ILB (Internal Load Balancer) support.

---

## 🚀 Deployment Options

- **Local Git / GitHub / Azure DevOps**
    - Continuous Integration/Continuous Deployment (CI/CD)
    - Branch-based deployment with automatic builds
- **ZIP Deploy**
    - Fast deployment without building on the server
    - Uses Kudu service API
    - Command: `az webapp deployment source config-zip`
- **FTP / FTPS**
    - Legacy deployment method
    - Useful for quick content updates
- **Azure CLI / PowerShell**
    - Automation-friendly deployment
    - Ideal for DevOps pipelines
- **Docker Container Support**
    - Deploy single containers or multi-container apps
    - Supports Compose files on Linux App Service

> ⚠️ **Gotcha**: For CI/CD setups, you need appropriate permissions in both Azure and your source control system. The exam may present scenarios where permissions are the root cause of deployment failures.

### Deployment Slots

- Zero-downtime deployments with validation
- Each slot is a separate instance with its own hostname
- Swap content and configurations (optional) between slots
- **Auto-swap** capability for automated deployment pipelines
- Traffic splitting/routing for A/B testing

> 💡 **Exam Tip**: Understanding which settings swap and which don't is crucial:
> 
> - Settings that swap: App settings with `WEBSITE_` prefix, connection strings marked as "slot"
> - Settings that don't swap: Custom domains, SSL certificates, scaling settings, WebJobs schedules

---

## ⚙️ Configuration & Settings

- **Application Settings**
    
    - Key-value pairs available as environment variables
    - Can be slot-specific or shared across slots
    - Encrypted at rest
- **Connection Strings**
    
    - Secure database connection info
    - Types: SQL Server, MySQL, PostgreSQL, Custom
- **App Service Editor**
    
    - Lightweight in-browser editor
    - Direct file system access
- **Custom Domains & SSL Certificates**
    
    - Bring your own domain and certificate
    - App Service Managed Certificates (free SSL)
    - SNI SSL and IP SSL support

---

## 🔐 Authentication & Authorization (Easy Auth)

Azure App Service provides built-in authentication and authorization support, eliminating the need to handle auth in your application code.

### Authentication Providers

- **Azure Active Directory (Azure AD)**
    
    - Single sign-on with corporate identities
    - Multi-tenant support
    - Conditional access policies
    - B2C integration for customer identities
- **Microsoft Personal Accounts**
    
    - Consumer Microsoft accounts
    - Xbox Live, Outlook.com, Hotmail.com
- **Social Providers**
    
    - Facebook
    - Google
    - Twitter
    - GitHub
- **OpenID Connect Providers**
    
    - Custom OIDC-compatible providers
    - Auth0, Okta, and other identity providers

### Authentication Flow Options

**Server-directed Flow (Web Apps)**

1. User navigates to protected resource
2. App Service redirects to provider login page
3. User authenticates with provider
4. Provider redirects back to App Service with token
5. App Service validates token and creates authenticated session
6. User is redirected to original resource

**Client-directed Flow (Mobile/SPA)**

1. Client app authenticates directly with provider
2. Client receives access token from provider
3. Client posts token to App Service `/.auth/login/<provider>`
4. App Service validates token and returns auth token
5. Client uses auth token for subsequent requests

### Authentication Modes

- **Allow Anonymous Requests**
    
    - No automatic action on unauthenticated requests
    - Application handles authentication logic
    - Useful for public APIs with optional authentication
- **Require Authentication**
    
    - Redirects unauthenticated users to configured provider
    - All requests must be authenticated
    - Suitable for internal applications

### Token Store & Session Management

- **Token Store**: Securely stores OAuth tokens on behalf of users
- **Session cookies**: Maintains authentication state across requests
- **Token refresh**: Automatically refreshes expired tokens when possible
- **Cross-origin requests**: CORS support for JavaScript clients

### Authorization Capabilities

- **Role-based access**: Configure user roles and permissions
- **Claims transformation**: Modify or add custom claims
- **API permissions**: Scope-based access control for APIs
- **Custom authorization**: Implement custom authorization logic

> 💡 **Exam Tip**: Understand the difference between authentication (who you are) and authorization (what you can do). Easy Auth handles authentication, but authorization logic often needs to be implemented in your application.

### Authentication Endpoints

```
/.auth/login/<provider>     # Initiate login
/.auth/logout              # Sign out user
/.auth/refresh             # Refresh tokens
/.auth/me                  # Get user information
```

### Code Examples

**Accessing User Information (C#)**

```csharp
// In your controller or middleware
public IActionResult GetUserInfo()
{
    var principal = HttpContext.User;
    var userId = principal.FindFirst(ClaimTypes.NameIdentifier)?.Value;
    var userName = principal.FindFirst(ClaimTypes.Name)?.Value;
    var userEmail = principal.FindFirst(ClaimTypes.Email)?.Value;
    
    return Ok(new { UserId = userId, Name = userName, Email = userEmail });
}
```

**Custom Claims (JavaScript)**

```javascript
// Access user info via /.auth/me endpoint
fetch('/.auth/me')
    .then(response => response.json())
    .then(data => {
        const user = data[0]; // First (and usually only) identity
        console.log('User ID:', user.user_id);
        console.log('User Claims:', user.user_claims);
    });
```

> ⚠️ **Gotcha**: Easy Auth tokens are different from the tokens your application might generate. The `/.auth/me` endpoint provides access to user information and claims, not the original OAuth tokens.

### Advanced Configuration

- **Custom domain authentication**: Configure auth for custom domains
- **Multiple authentication providers**: Allow users to choose login method
- **Token store encryption**: Enhanced security for sensitive applications
- **Login parameters**: Pass custom parameters to authentication providers

> 💡 **Exam Tip**: Know how to configure authentication via Azure portal, ARM templates, and Azure CLI. The exam may ask about programmatic configuration of auth settings.

---

## 🌐 Networking & Connectivity

Azure App Service provides multiple networking features to secure and control traffic flow to and from your applications.

### Inbound Network Features

**Public Endpoint (Default)**

- Apps are accessible from the internet
- Default `*.azurewebsites.net` domain
- Support for custom domains and SSL certificates

**Access Restrictions (IP Filtering)**

- Allow/deny rules based on IP addresses or CIDR blocks
- Support for IPv4 and IPv6
- Service tag support (AzureCloud, Internet, etc.)
- Action types: Allow or Deny
- Priority-based rule evaluation

```bash
# Add IP restriction via CLI
az webapp config access-restriction add \
  --resource-group myResourceGroup \
  --name myWebApp \
  --rule-name "AllowOfficeIP" \
  --action Allow \
  --ip-address 203.0.113.0/24 \
  --priority 100
```

**Private Endpoints**

- Private IP address within your VNet
- Eliminates public internet exposure
- DNS integration with Azure Private DNS
- Network Security Group (NSG) support
- Multiple private endpoints per app

> 💡 **Exam Tip**: Private Endpoints are for inbound traffic to your App Service. They provide a private IP address that clients within your VNet can use to access your app.

### Outbound Network Features

**Default Outbound Access**

- NAT gateway with shared public IP addresses
- Dynamic IP assignment
- Cannot be customized or controlled

**VNet Integration**

- Secure outbound connectivity to VNet resources
- Two types available:
    - Regional VNet Integration (recommended)
    - Gateway-required VNet Integration (legacy)

**Regional VNet Integration**

- Direct integration with delegated subnet
- Supports Azure DNS private zones
- Route table support for traffic steering
- Network Security Groups apply to outbound traffic
- No gateway required

```bash
# Configure VNet integration
az webapp vnet-integration add \
  --resource-group myResourceGroup \
  --name myWebApp \
  --vnet myVNet \
  --subnet mySubnet
```

**Gateway-required VNet Integration**

- Requires VPN Gateway (Point-to-Site)
- Legacy option, use Regional VNet Integration instead
- Limited to 5 concurrent connections per gateway

### Hybrid Connectivity

**Hybrid Connections**

- TCP-based connectivity to on-premises resources
- Relay-based service (Azure Relay)
- No VPN or gateway required
- Supports specific hostname:port combinations
- Bi-directional communication

**Service Endpoints**

- Secure access to Azure PaaS services
- Traffic remains on Microsoft backbone
- Available for: Storage, SQL Database, Key Vault, etc.
- Configured at subnet level

### Network Architecture Patterns

**Hub-and-Spoke with App Service**

```
Internet → App Gateway/Front Door → App Service (VNet Integrated)
                                        ↓
                                   Hub VNet (Firewall, DNS)
                                        ↓
                                 Spoke VNets (Backend Services)
```

**App Service Environment (ASE)**

- Fully isolated, single-tenant deployment
- Deployed into customer VNet subnet
- Internal Load Balancer (ILB) or External facing
- Custom DNS and routing table support
- Network Security Groups for traffic control

### DNS Configuration

**Default DNS Resolution**

- Azure-provided DNS for public resources
- Custom DNS servers via VNet integration

**Private DNS Zones**

- Automatic registration for private endpoints
- Custom domain resolution within VNet
- Integration with on-premises DNS

### Load Balancing & Traffic Management

**Azure Front Door**

- Global HTTP load balancer
- SSL termination and WAF capabilities
- Session affinity and URL-based routing
- Backend health probes

**Application Gateway**

- Layer 7 load balancer
- WAF and SSL offloading
- Path-based and host-based routing
- Autoscaling capabilities

**Traffic Manager**

- DNS-based load balancing
- Geographic, weighted, and performance routing
- Health monitoring and automatic failover

### Network Security

**Web Application Firewall (WAF)**

- Protection against common web vulnerabilities
- OWASP Top 10 threat protection
- Custom rules and rate limiting
- Available in Application Gateway and Front Door

**DDoS Protection**

- Built-in DDoS protection for all Azure services
- DDoS Protection Standard for enhanced mitigation
- Real-time attack metrics and alerting

**Network Security Groups (NSGs)**

- Control traffic to/from subnets
- Apply to VNet-integrated App Services
- Stateful firewall rules

### Monitoring Network Traffic

**Connection Monitor**

- Monitor connectivity between App Service and dependencies
- Latency and packet loss metrics
- Integration with Network Watcher

**Network Security Group Flow Logs**

- Capture allowed/denied traffic information
- Integration with Traffic Analytics
- Storage in Azure Storage accounts

> ⚠️ **Gotcha**: VNet Integration only affects outbound traffic from your App Service. For inbound security, you need Private Endpoints, Access Restrictions, or front-end services like Application Gateway.

### Network Troubleshooting Tools

**Kudu Network Tools**

- Available at `https://sitename.scm.azurewebsites.net/DebugConsole`
- Network trace capabilities
- DNS lookup tools
- TCP connectivity testing

**Network Watcher**

- IP flow verify
- Next hop analysis
- Connection troubleshoot
- VPN diagnostics

### CLI Commands for Networking

```bash
# Configure access restrictions
az webapp config access-restriction add \
  --resource-group myRG --name myApp \
  --rule-name "DenyAll" --action Deny \
  --ip-address 0.0.0.0/0 --priority 2000

# Add VNet integration
az webapp vnet-integration add \
  --resource-group myRG --name myApp \
  --vnet myVNet --subnet mySubnet

# Create private endpoint
az network private-endpoint create \
  --resource-group myRG --name myAppPE \
  --vnet-name myVNet --subnet myPESubnet \
  --private-connection-resource-id "/subscriptions/.../Microsoft.Web/sites/myApp" \
  --connection-name myAppConnection \
  --group-ids sites

# Configure hybrid connection
az webapp hybrid-connection add \
  --resource-group myRG --name myApp \
  --namespace myRelay --hybrid-connection myHC
```

> 💡 **Exam Tip**: Understand the different networking options and when to use each one. The exam often presents scenarios where you need to choose between Private Endpoints, VNet Integration, and Hybrid Connections based on requirements.

> ⚠️ **Gotcha**: Remember that some networking features have specific requirements or limitations. For example, Regional VNet Integration requires a dedicated subnet with `/28` or larger address space.

---

### Configuration Best Practices

- Use Key Vault references for sensitive values
- Implement slot settings for environment-specific configs
- Leverage App Configuration service for feature flags and centralized settings

> ⚠️ **Gotcha**: App settings override web.config/appsettings.json values with the same name. The exam may ask about troubleshooting scenarios where local settings appear to be ignored after deployment.

---

## 📊 Monitoring & Diagnostics

- **Application Insights**
    - APM (Application Performance Monitoring)
    - Distributed tracing
    - Live metrics stream
    - User behavior analytics
    - Integration with Azure Monitor
- **App Service Logs**
    - Application logs (streaming logs)
    - Web server logs (HTTP logs)
    - Detailed error messages
    - Failed request tracing
- **Diagnostic Settings**
    - Send logs to:
        - Azure Storage (archival)
        - Event Hub (for external processing)
        - Log Analytics (for queries and alerting)
    - Configure retention periods

| Type                    | Platform       | Location                                           | Description                                                                                                                                                                                                                                                                                                                                   |     |
| ----------------------- | -------------- | -------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --- |
| Application logging     | Windows, Linux | App Service file system and/or Azure Storage blobs | Logs messages generated by your application code. The messages are generated by the web framework you choose, or from your application code directly using the standard logging pattern of your language. Each message is assigned one of the following categories: **Critical**, **Error**, **Warning**, **Info**, **Debug**, and **Trace**. |     |
| Web server logging      | Windows        | App Service file system or Azure Storage blobs     | Raw HTTP request data in the W3C extended log file format. Each log message includes data like the HTTP method, resource URI, client IP, client port, user agent, response code, and so on.                                                                                                                                                   |     |
| Detailed error messages | Windows        | App Service file system                            | Copies of the _.html_ error pages that would otherwise be sent to the client browser. For security reasons, detailed error pages shouldn't be sent to clients in production, but App Service can save the error page each time an application error occurs that has HTTP code 400 or greater.                                                 |     |
| Failed request tracing  | Windows        | App Service file system                            | Detailed tracing information on failed requests, including a trace of the IIS components used to process the request and the time taken in each component. One folder is generated for each failed request, which contains the XML log file, and the XSL stylesheet to view the log file with.                                                |     |
| Deployment logging      | Windows, Linux | App Service file system                            | Helps determine why a deployment failed. Deployment logging happens automatically and there are no configurable settings for deployment logging.                                                                                                                                                                                              |     |


> 💡 **Exam Tip**: Know how to query App Service logs using Kudu console, Azure CLI, and PowerShell. The exam often includes scenarios about troubleshooting app errors by accessing the right log files.

### Health Checks

- Custom endpoint monitoring for container apps
- Integration with App Service's built-in load balancer
- Path-based health probe configuration

> ⚠️ **Gotcha**: Health check endpoints must respond within 10 seconds and return HTTP 200 status code to be considered healthy.

---

## 📈 Scaling Options

- **Manual Scaling**
    - Fixed number of instances
    - Available in all paid tiers
- **Autoscale Rules**
    - Based on:
        - CPU percentage
        - Memory usage
        - HTTP queue length
        - Custom metrics from Application Insights
        - Schedule-based scaling
    - Cooldown periods to prevent flapping
- **Scale Up**: Change pricing tier for more powerful VMs
    - Vertical scaling
    - Requires app restart
- **Scale Out**: Increase number of instances
    - Horizontal scaling
    - No downtime during scale operations

### Instance Limits by Tier

|Tier|Max Instances|
|---|---|
|Free/Shared|1|
|Basic|3|
|Standard|10|
|Premium|30|
|Isolated|100|

> 💡 **Exam Tip**: Understand the different scale metrics and when to use each one. Know how to configure scale conditions with AND/OR operators for complex scaling rules.

> ⚠️ **Gotcha**: Autoscale has a minimum instance count that will always run (and be billed) even during low-traffic periods. The exam may ask about cost optimization scenarios related to scaling.

---

## 🔄 Deployment Slots (Deep Dive)

- Feature for zero-downtime deployments
- Available in Standard tier and above
- Each slot has its own hostname
- Capabilities:
    - **Testing in production**: Validate changes before swap
    - **Staged deployments**: Deploy to staging slot, then swap
    - **Rollback**: Swap back if issues detected
    - **Warm-up**: Application initializes before serving traffic

### Slot Swapping Process

1. Apply settings from target slot to source slot instances
2. Restart source slot instances to apply settings
3. Pre-warm source slot instances (wait for healthy response)
4. Swap routing rules for the two slots
5. Repeat process for the other slot (it now has the old production app)

| Settings that are swapped                                           | Settings that aren't swapped                           |
| ------------------------------------------------------------------- | ------------------------------------------------------ |
| General settings, such as framework version, 32/64-bit, web sockets | Publishing endpoints                                   |
| App settings (can be configured to stick to a slot)                 | Custom domain names                                    |
| Connection strings (can be configured to stick to a slot)           | Nonpublic certificates and TLS/SSL settings            |
| Handler mappings                                                    | Scale settings                                         |
| Public certificates                                                 | WebJobs schedulers                                     |
| WebJobs content                                                     | IP restrictions                                        |
| Hybrid connections                                                  | Always On                                              |
| Azure Content Delivery Network                                      | Diagnostic log settings                                |
| Service endpoints                                                   | Cross-origin resource sharing (CORS)                   |
| Path mappings                                                       | Virtual network integration                            |
|                                                                     | Managed identities                                     |
|                                                                     | Settings that end with the suffix `_EXTENSION_VERSION` |

> 💡 **Exam Tip**: The exam may ask about "swap with preview" and how it differs from regular swap operations. Know that preview allows you to verify the app with target slot settings before completing the swap.

> ⚠️ **Gotcha**: Not all settings swap between slots. Connection strings marked as "sticky" remain with the slot, not the app.

---

## 🐳 App Service for Containers

- Deploy Docker containers directly
- Supports:
    - Custom container images from:
        - Azure Container Registry (ACR)
        - Docker Hub
        - Private registries
    - Multi-container apps using Docker Compose (Linux only)
    - CI/CD with container-based workflows

### Container Configuration

- Environment variables
- Startup commands
- Persistent storage mapping
- HTTPS-only and TLS settings
- Custom startup files

> 💡 **Exam Tip**: Understand the container restart policies and how App Service handles container health monitoring.

> ⚠️ **Gotcha**: Windows containers in App Service have different limitations than Linux containers, such as no multi-container support with Docker Compose.

---

## 🔐 Security Considerations

- **Managed Identity**
    - System-assigned: Tied to app lifetime
    - User-assigned: Independent lifecycle
    - Use for Azure resource access without credentials
- **Custom Authentication Providers**
    - OAuth/OIDC integration
    - Claims transformation
    - Role-based access control

> 💡 **Exam Tip**: The exam often tests on implementing Managed Identities and how to use them in your code to access various Azure services.

> ⚠️ **Gotcha**: System-assigned Managed Identity is automatically deleted when the App Service is deleted, but user-assigned Managed Identity persists independently.

---

## 🧩 WebJobs and Functions Integration

- **WebJobs**:
    
    - Background processing within App Service
    - Continuous or triggered execution
    - Supports various file types (.cmd, .bat, .exe, .ps1, .sh, .php, .py, .js)
- **Functions Integration**:
    
    - Functions can run in the same App Service Plan
    - Shared configuration and networking
    - Unified management

> ⚠️ **Gotcha**: WebJobs don't automatically scale with the App Service - they run on each instance. This can lead to duplicate executions in multi-instance environments.

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

# Create a deployment slot
az webapp deployment slot create --resource-group myResourceGroup --name myWebApp --slot staging

# Configure auto-swap
az webapp deployment slot auto-swap --resource-group myResourceGroup --name myWebApp --slot staging --target-slot production --auto-swap-slot-name staging

# Configure continuous deployment
az webapp deployment source config --resource-group myResourceGroup --name myWebApp --repo-url https://github.com/username/repo --branch main --manual-integration

# Set up autoscaling
az monitor autoscale create --resource-group myResourceGroup --resource myWebApp --resource-type "Microsoft.Web/sites" --name autoscaleConfig --min-count 2 --max-count 5 --count 2

# Add an autoscale rule
az monitor autoscale rule create --resource-group myResourceGroup --autoscale-name autoscaleConfig --condition "Percentage CPU > 70 avg 10m" --scale out 2
```

> 💡 **Exam Tip**: Focus on understanding the parameters of these commands rather than memorizing them. The exam will test your knowledge of what each parameter does and when to use specific flags.

---

## 🔧 App Service Troubleshooting

- **Kudu Console** (accessible at `https://sitename.scm.azurewebsites.net`)
    - Process explorer
    - Environment variables
    - File system access
    - Command line interface
- **Diagnostic Tools**
    - Memory dumps
    - CPU profiling
    - Event viewer
    - Failed request tracing
- **Remote Debugging**
    - Visual Studio integration
    - Snapshot debugging
    - Memory monitoring

> ⚠️ **Gotcha**: The exam may include questions about accessing Kudu service when the main app is unreachable or experiencing issues. Remember that Kudu has its own endpoint independent of your app.

---

## 📖 Pro Tips

- Always enable **Application Insights** for production apps
    
    - Set appropriate sampling to manage costs
    - Configure alerts for key performance metrics
- Use **deployment slots** for zero-downtime deployments
    
    - Implement slot-specific app settings
    - Utilize traffic splitting for controlled rollouts
    - Automate the deployment workflow with webhooks
- Implement **custom domains & SSL** for public-facing apps
    
    - Use App Service Managed Certificates for free SSL
    - Configure minimum TLS version (1.2 recommended)
    - Set up HTTPS-only mode
- Use **Autoscale rules** to optimize performance and cost
    
    - Set appropriate minimum instances for your SLA
    - Consider predictive scaling for known traffic patterns
    - Combine multiple metrics for smarter scaling
- Integrate with **Azure Key Vault** for secure secret management
    
    - Use Managed Identity for Key Vault access
    - Reference secrets directly in app settings
    - Rotate credentials regularly
- **Networking best practices**
    
    - Utilize VNet integration for secure outbound traffic
    - Implement Private Endpoints for inbound security
    - Use Azure Front Door or Application Gateway for WAF capabilities

> 💡 **Exam Tip**: The exam often asks about implementing best practices in real-world scenarios, especially around security and performance optimization.

---

## 🧠 Common Exam Scenarios

1. **Troubleshooting deployments**
    
    - Identifying issues with deployment pipelines
    - Resolving continuous integration failures
    - Debugging container deployment issues
2. **Scaling implementations**
    
    - Designing autoscale rules for specific workload patterns
    - Optimizing for cost vs. performance
    - Implementing proper metrics for scaling triggers
3. **Security configurations**
    
    - Implementing authentication and authorization
    - Securing connections to backend services
    - Restricting network access appropriately
4. **Migration scenarios**
    
    - Moving from on-premises to App Service
    - Upgrading between tiers
    - Transitioning from VMs to PaaS
5. **Multi-region deployment and networking**
    
    - Traffic Manager integration
    - Front Door configuration
    - Geo-replication strategies
    - Cross-region VNet peering scenarios
6. **Authentication and authorization flows**
    
    - Implementing Easy Auth for different scenarios
    - Custom authentication provider integration
    - Token management and refresh strategies

> ⚠️ **Gotcha**: The exam frequently presents scenarios where multiple solutions might work, but you need to select the most cost-effective or performant option given specific constraints.

---
## 📅 Last-Minute Review

- App Service Plans determine compute resources, pricing tier, and available features
- Deployment slots enable zero-downtime deployments and A/B testing
- Authentication/Authorization provides built-in identity support
- Scaling options include manual, automatic, vertical, and horizontal scaling
- VNet Integration allows secure outbound connections to resources in your virtual network
- Private Endpoints provide secure inbound access with private IP addresses
- Easy Auth provides built-in authentication without coding authentication logic
- Multiple authentication providers can be configured for flexible user access
- Hybrid Connections enable secure access to on-premises resources without VPN
- Access Restrictions control inbound traffic based on IP addresses or service tags
- Managed Identity eliminates the need for storing credentials
- Monitoring solutions include Application Insights and App Service Logs
- WebJobs provide background processing capabilities
- Container support allows deploying Docker images

> 💡 **Final Exam Tip**: For the AZ-204 exam, focus on the implementation details rather than just conceptual understanding. Be prepared to select the right CLI commands, identify the correct service tier for specific requirements, and troubleshoot common deployment and configuration issues.
