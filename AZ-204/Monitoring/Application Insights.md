# 📊 AZ-204 Study Guide: Application Insights

## 🔍 Overview

Application Insights is an extension of Azure Monitor that provides powerful application performance monitoring (APM) features. It helps developers detect anomalies, diagnose issues, and understand how users interact with their applications.

---

## 📋 Key Concepts

### 🧩 Core Components

- **Instrumentation Key/Connection String**: Unique identifier that connects your app to your Application Insights resource
- **Telemetry Data**: Various types of data collected from your application
- **Live Metrics**: Real-time performance monitoring with near-zero impact
- **Smart Detection**: Automatic detection of potential performance issues
- **Application Map**: Visual mapping of dependencies between application components

### 📈 Types of Telemetry

|Type|Description|Exam Importance|
|---|---|---|
|**Request**|HTTP requests received by your server|⭐⭐⭐⭐⭐|
|**Dependency**|Calls your app makes to external services|⭐⭐⭐⭐⭐|
|**Exception**|Captured server-side and client-side exceptions|⭐⭐⭐⭐|
|**Page View**|Browser-side telemetry about web pages|⭐⭐⭐|
|**Custom Events**|Events you define for business tracking|⭐⭐⭐⭐|
|**Metrics**|Performance counters and custom metrics|⭐⭐⭐⭐|
|**Trace**|Diagnostic log messages|⭐⭐⭐|

---

## 💻 Implementation

### 🔧 Server-Side Monitoring

#### ASP.NET Core

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// Add Application Insights telemetry
builder.Services.AddApplicationInsightsTelemetry();
```

#### Manual Tracking

```csharp
// Inject TelemetryClient
private readonly TelemetryClient _telemetryClient;

public SomeController(TelemetryClient telemetryClient)
{
    _telemetryClient = telemetryClient;
}

// Track custom events
public IActionResult SomeAction()
{
    _telemetryClient.TrackEvent("CustomEvent", new Dictionary<string, string> 
    { 
        { "Property1", "Value1" } 
    });
    
    // Track dependencies manually
    using (var operation = _telemetryClient.StartOperation<DependencyTelemetry>("CustomDependency"))
    {
        try
        {
            // Your code here
        }
        catch (Exception ex)
        {
            operation.Telemetry.Success = false;
            _telemetryClient.TrackException(ex);
            throw;
        }
    }
    
    return Ok();
}
```

### 📱 Client-Side Monitoring

#### JavaScript

```javascript
// Add this to your HTML
<script type="text/javascript">
!function(T,l,y){/* Application Insights JavaScript SDK */}(window,document,{
    src: "https://az416426.vo.msecnd.net/scripts/b/ai.2.min.js", 
    cfg: {
        instrumentationKey: "YOUR_INSTRUMENTATION_KEY"
    }});
</script>
```

---

## 📊 Analysis Features

### 👥 User Behavior Analytics

- **Users, Sessions & Events**: Analyze telemetry from three perspectives (critical for the exam)
    
    - **Users**: Unique visitors counted via anonymous IDs stored in browser cookies
    - **Sessions**: Duration of user activity (reset after 30 minutes of inactivity or 24 hours of continuous use)
    - **Events**: Frequency of feature usage
- **Funnels**: Track user progression through multi-step processes
    
    - Identify abandonment points in workflows
    - Analyze conversion rates
- **User Flows**: Visualize navigation patterns through your application
    
    - Discover most common paths
    - Identify engagement hotspots and problem areas
- **Cohorts**: Group users by common characteristics
    
    - Analyze feature adoption
    - Compare behavior patterns
- **Impact Analysis**: Understand how properties affect conversion
    
    - Measure how performance metrics influence user behavior
    - Identify high-impact improvement areas
- **HEART Framework**: Five dimensions of customer experience
    
    - **H**appiness: Satisfaction metrics
    - **E**ngagement: User involvement depth
    - **A**doption: New user acquisition
    - **R**etention: Returning user rates
    - **T**ask success: Completion efficiency

---

## 📝 Resource Planning

### 📊 Single vs. Multiple Application Insights Resources

#### When to use a Single Resource:

- Applications developed and managed by the same team
- Need for centralized KPIs and dashboards
- Consistent RBAC requirements across components
- Same alert criteria and export settings
- Uniform smart detection and work item integration

#### When to use Multiple Resources:

- Different development stages (dev/test/production)
- Applications with separate teams
- Distinct billing or quota requirements
- Different access control needs
- Applications with radically different usage patterns

---

## 🚨 Alerts & Monitoring

### 📊 Metrics Alert

```csharp
// Monitor server response time
threshold: 500ms
condition: > 
window: 5 minutes
```

### 💥 Failure Alert

```csharp
// Track failed requests
threshold: 5%
condition: >
window: 15 minutes
```

### 📈 Availability Tests

- Configure web tests to monitor endpoint availability
- Set up multi-step tests for critical user flows

---

## 💰 Cost Management

### 🔍 Sampling Methods

- **Adaptive Sampling**: Automatically adjusts data collection rate
    
    - Default for ASP.NET SDK
    - Maintains statistically correct representation
- **Fixed-Rate Sampling**: Consistent percentage reduction
    
    - More predictable volume
    - Configured in code or portal
- **Ingestion Sampling**: Applied at service ingestion point
    
    - Useful when you can't modify the application
    - Applied after data collection

```csharp
// Configure fixed-rate sampling in code
services.ConfigureTelemetryModule<DependencyTrackingTelemetryModule>((module, o) => 
{
    module.SetSamplingPercentage(10); // 10% sampling
});
```

---

## 🎯 Exam Tips & Gotchas

### ⚠️ Common Exam Traps

1. **Instrumentation Key vs. Connection String**:
    
    - Newer SDKs use connection strings
    - Connection strings support additional configuration options
    - Exam might ask about migration scenarios
2. **Sampling Configurations**:
    
    - Know the difference between adaptive, fixed-rate, and ingestion sampling
    - Understand how sampling affects billing and data accuracy
3. **User Identity Tracking**:
    
    - Application Insights tracks anonymous users by default
    - Custom user tracking requires explicit implementation:
    
    ```csharp
    telemetry.Context.User.Id = "user@example.com";
    telemetry.Context.User.AuthenticatedUserId = "authenticated-id";
    ```
    
4. **Log Retention**:
    
    - Default retention period is 90 days
    - Can be extended to 730 days with additional cost
    - Expect questions about long-term storage solutions
5. **SDK Versioning**:
    
    - Application Insights SDK version matters
    - Newer versions have different default behaviors
    - Pay attention to version numbers in exam scenarios

### 🚀 Performance Optimization

- Use `Flush()` before application shutdown to ensure data transmission
- Implement proper exception handling around telemetry code
- Consider disabling telemetry in certain environments:
    
    ```csharp
    TelemetryConfiguration.Active.DisableTelemetry = true;
    ```
    

### 💼 Enterprise Scenarios

- **Multi-region deployment**: Configure separate resources or use role names
- **Microservices architecture**: Use application map to visualize dependencies
- **High-traffic applications**: Implement appropriate sampling strategy

---

## 🔄 Integration Points

### 🔌 Common Integrations

- **Azure DevOps**: Work item creation from issues
- **Power BI**: Custom analytics dashboards
- **Logic Apps**: Automated workflows based on telemetry
- **Azure Functions**: Serverless processing of telemetry data
- **Azure Data Explorer**: Advanced querying capabilities

### 📊 Continuous Export

- Stream telemetry to Azure Storage for long-term retention
- Use for custom processing and analysis
- Configure with caution (affects billing)

---

## 📝 Exam Practice Questions

1. Which sampling method is enabled by default in ASP.NET Core applications using Application Insights?
    
    - A) Fixed-rate sampling
    - B) Adaptive sampling ✓
    - C) Ingestion sampling
    - D) No sampling
2. A company wants to track how users progress through a five-step checkout process. Which Application Insights feature should they use?
    
    - A) Cohorts
    - B) Funnels ✓
    - C) Impact Analysis
    - D) User Flows
3. When is it appropriate to use multiple Application Insights resources for a single application?
    
    - A) When different components have different SLAs ✓
    - B) When using microservices (not always true)
    - C) When using multiple Azure regions (depends on strategy)
    - D) When using different programming languages (incorrect)
4. What is the default data retention period for Application Insights?
    
    - A) 30 days
    - B) 60 days
    - C) 90 days ✓
    - D) 180 days

---

## 🗺️ Exam Strategy

1. **Know the portal interface**: Many questions reference specific locations in the Azure portal
2. **Understand query language**: Be familiar with Kusto Query Language basics
3. **Focus on implementation code**: Pay special attention to SDK integration patterns
4. **Cost optimization scenarios**: Expect questions about managing data volume and costs
5. **Enterprise-level concerns**: Security, scaling, and multi-component scenarios are common

---

## 📚 Additional Resources

- [Official Microsoft Documentation](https://learn.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview)
- [Application Insights SDK Reference](https://learn.microsoft.com/en-us/azure/azure-monitor/app/asp-net-core)
- [Kusto Query Language (KQL) Reference](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/)
- [Azure Monitor Pricing](https://azure.microsoft.com/en-us/pricing/details/monitor/)

---

📌 Remember: Application Insights questions often focus on implementation details, cost management, and enterprise-level usage scenarios. Pay special attention to the SDKs, configuration options, and integration patterns.