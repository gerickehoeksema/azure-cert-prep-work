# 📚 AZ-204 Study Guide: Azure Content Delivery Network (CDN)

## 🌐 Introduction to Azure CDN

Azure Content Delivery Network (CDN) is a distributed network of servers that can deliver web content to users with high performance and reliability. CDNs store cached content on edge servers close to end users to minimize latency.

## 🏗️ Azure CDN Architecture

```
                    🌐 Origin Server
                          ↑ ↓
                    🔄 Azure CDN
                 ↙️    ↓    ↘️
         🖥️ Edge     🖥️ Edge    🖥️ Edge
         Server      Server     Server
          ↑            ↑           ↑
          ↓            ↓           ↓
         👤 User      👤 User     👤 User
```

## 📋 Azure CDN Providers

Azure CDN offers multiple provider options, each with specific features:

|Provider|Features|Best For|
|---|---|---|
|**Microsoft**|Deep integration with Azure services, WAF capabilities|General purpose, Azure optimization|
|**Akamai**|Global coverage, media optimization|Media streaming, global presence|
|**Verizon**|Advanced analytics, content optimization|Advanced analytics needs|

> ⚠️ **Exam Tip**: Be able to differentiate between CDN providers and their unique features. Exam questions often ask when to use which provider.

## 💻 Core Concepts

### POP (Point of Presence)

Edge locations where content is cached for faster delivery to nearby users.

### Edge Nodes

The physical servers in a POP location where content caching occurs.

### Origin

The source location where your content originates (Azure Storage, Web App, any public web server).

### Cache Behavior

- **Time-to-Live (TTL)**: How long content remains in cache
- **Cache-Control headers**: HTTP headers that control caching behavior

> ⚠️ **Exam Gotcha**: Remember that changing CDN providers requires creating a new CDN profile. You cannot switch providers for an existing profile.

## 🛠️ Azure CDN Features

### 🔒 HTTPS Support

- Custom domain HTTPS with CDN-managed or BYO certificates
- Supports modern TLS versions

### 🚫 Geo-Filtering

Restrict access to your content based on country/region.

```csharp
// C# example for geo-filtering rule setup
var geoFilterRule = new GeoFilterRule
{
    Action = GeoFilterActions.Block,
    CountryCodes = new List<string> { "US", "CA" }
};
```

### 🔍 Rules Engine (Premium Only)

- URL redirection and rewriting
- HTTP header modification
- Conditional content delivery

### 📊 Real-Time Analytics

- Bandwidth usage
- Cache hit ratio
- Client details

## 🔄 Content Caching Optimization

### Caching Rules

- Configure global and custom caching rules
- Control cache duration with TTL settings

```csharp
// C# example for setting caching rules
var deliveryRule = new DeliveryRule
{
    Name = "CachingRule",
    Order = 1,
    Conditions = new List<DeliveryRuleCondition>
    {
        new UrlPathCondition
        {
            MatchValues = new List<string> { "/images/*" }
        }
    },
    Actions = new List<DeliveryRuleAction>
    {
        new CacheExpirationAction
        {
            CacheBehavior = CacheBehavior.Override,
            CacheDuration = "7.00:00:00"  // 7 days
        }
    }
};
```

### Cache Purging

- Force content refresh by purging cached content
- Can purge individual files or wildcard paths

> ⚠️ **Exam Tip**: For the exam, understand that purging clears the cache immediately but it takes time for new content to propagate to all edge locations.

## 🧩 Azure CDN Integration with Other Services

### Azure Storage

- Static content hosting with CDN for global distribution
- Blob storage as CDN origin

### Azure Web Apps

- Accelerate content delivery for web applications
- Reduce load on app servers

### Azure Media Services

- Video streaming optimization
- DRM support

## 💰 Azure CDN Pricing and SKUs

|Tier|Features|Use Cases|
|---|---|---|
|**Standard**|Basic caching, HTTP/HTTPS|Static content, small websites|
|**Premium**|Rules engine, advanced analytics|Enterprise applications, complex delivery requirements|

> ⚠️ **Exam Gotcha**: Premium features (like Rules Engine) are only available with the Verizon Premium provider. Standard features vary between providers.

## 🌟 Performance Optimization Techniques

### 1. Compression

Enable compression to reduce file sizes and improve delivery speed.

### 2. Preloading/Warming

Load content into the CDN cache before peak demand.

### 3. Query String Handling

Configure how CDN handles query strings in URLs:

- Ignore query strings
- Bypass cache when query string changes
- Cache every unique URL

## 📝 Azure CDN Management

### Azure Portal

- Create and configure CDN profiles and endpoints
- Monitor performance
- Purge cached content

### Azure CLI

```bash
# Create a CDN profile
az cdn profile create --name MyProfile --resource-group MyRG --sku Standard_Microsoft

# Create a CDN endpoint
az cdn endpoint create --name MyEndpoint --profile-name MyProfile --resource-group MyRG --origin-host-header www.contoso.com --origin www.contoso.com
```

### Azure PowerShell

```powershell
# Create a CDN profile
New-AzCdnProfile -ProfileName "MyProfile" -ResourceGroupName "MyRG" -Sku "Standard_Microsoft" -Location "West US"

# Create a CDN endpoint
New-AzCdnEndpoint -EndpointName "MyEndpoint" -ProfileName "MyProfile" -ResourceGroupName "MyRG" -OriginHostHeader "www.contoso.com" -Origin "www.contoso.com"
```

### C# SDK

```csharp
// Create a CDN profile
var profile = new Profile
{
    Location = "WestUS",
    Sku = new Sku { Name = SkuName.StandardMicrosoft }
};
await cdnClient.Profiles.CreateAsync(resourceGroupName, profileName, profile);

// Create a CDN endpoint
var endpoint = new Endpoint
{
    Location = "WestUS",
    IsHttpAllowed = true,
    IsHttpsAllowed = true,
    Origins = new List<DeepCreatedOrigin>
    {
        new DeepCreatedOrigin
        {
            Name = "origin1",
            HostName = "www.contoso.com"
        }
    }
};
await cdnClient.Endpoints.CreateAsync(resourceGroupName, profileName, endpointName, endpoint);
```

## 🚀 Advanced Features

### Dynamic Site Acceleration (DSA)

- Optimizes delivery of dynamic content
- Route optimization, TCP optimization, object prefetching

### Rules Engine (Premium)

- Conditional content delivery
- URL redirection/rewriting
- Custom header modification

## 🧪 Testing and Troubleshooting CDN

### Common Issues

1. **Origin not reachable**: Ensure origin server is accessible
2. **HTTPS configuration errors**: Check certificate validity and configuration
3. **Cache miss ratio high**: Review caching rules and TTL settings

### Debugging Tools

- **Network tracing**: Use browser developer tools to inspect request/response headers
- **Azure Diagnostics**: Enable logging for CDN endpoints
- **CDN analytics**: Review performance metrics in Azure Portal

## 📋 Exam Focus Areas

1. **Provider Selection**: Know when to use which CDN provider
2. **Configuration Options**: Understand caching rules, geo-filtering, and custom domains
3. **Integration with Azure Services**: Know how CDN works with Storage, Web Apps, and Media Services
4. **Performance Optimization**: Understand how to optimize CDN for different scenarios
5. **Management APIs**: Be familiar with how to manage CDN using different interfaces (Portal, CLI, PowerShell, SDK)

## 💡 Exam Tips & Tricks

1. ✅ **Remember the hierarchy**: CDN Profile → CDN Endpoint → Origin
2. ✅ **Caching behavior**: Default TTL is 7 days if not specified by origin
3. ✅ **Provider capabilities**: Only Verizon Premium offers the Rules Engine
4. ✅ **Cache purge limits**: Standard tier limits number of purge operations per day
5. ✅ **Custom domain HTTPS**: Know the difference between CDN-managed and BYO certificates
6. ✅ **Propagation times**: Changes to CDN settings take time to propagate globally
7. ✅ **Premium features**: Dynamic Site Acceleration is only available in Premium tier
8. ✅ **Storage integration**: Know how to integrate Azure Storage static websites with CDN

## 🚨 Common Exam Gotchas

1. ❌ **Provider switching**: Once a CDN profile is created, you cannot change its provider
2. ❌ **Endpoint naming**: Endpoint names must be globally unique and can only use lowercase letters
3. ❌ **Purge timing**: Purging clears the cache immediately, but content takes time to propagate
4. ❌ **Rules engine availability**: Only available with Verizon Premium
5. ❌ **Custom domain HTTPS**: Cannot use BYO certificate with Microsoft provider
6. ❌ **Origin path**: Specifies a subdirectory at the origin but is often overlooked
7. ❌ **Query string caching**: Default behavior varies by provider

## 🔄 Recent Updates to Know

- **Azure Front Door integration**: CDN and Front Door services are being unified
- **Enhanced security features**: WAF capabilities for CDN
- **Modern TLS support**: TLS 1.3 support for secure connections

## 📝 Practice Questions

1. Which CDN provider offers Rules Engine capabilities?
    
    - A) Microsoft
    - B) Akamai
    - C) Verizon Standard
    - D) Verizon Premium
    
    _Answer: D) Verizon Premium_
    
2. What happens when you purge content from Azure CDN?
    
    - A) Content is immediately removed from all edge locations
    - B) Content is marked for deletion but may still be served for some time
    - C) Content is removed from edge locations but takes time to propagate
    - D) Nothing happens until the TTL expires
    
    _Answer: C) Content is removed from edge locations but takes time to propagate_
    
3. When integrating Azure CDN with Azure Storage, what is the recommended way to ensure content is always up-to-date?
    
    - A) Set a low TTL for all content
    - B) Manually purge the CDN after each update
    - C) Use versioned URLs for content that changes
    - D) Disable caching completely
    
    _Answer: C) Use versioned URLs for content that changes_
    

## 📅 Final Preparation Checklist

- [ ] Review all Azure CDN providers and their specific features
- [ ] Understand caching mechanisms and how TTL works
- [ ] Practice creating and configuring CDN profiles and endpoints
- [ ] Study the integration points with other Azure services
- [ ] Review common troubleshooting scenarios
- [ ] Practice writing code samples for CDN management using SDK
- [ ] Understand pricing tiers and their limitations
