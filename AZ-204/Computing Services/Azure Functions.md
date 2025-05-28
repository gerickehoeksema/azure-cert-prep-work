## 📋 Table of Contents

- [Introduction to Azure Functions](https://claude.ai/chat/28724acd-800c-45c9-8dec-4fc5505553c0#introduction-to-azure-functions)
- [Function Fundamentals](https://claude.ai/chat/28724acd-800c-45c9-8dec-4fc5505553c0#function-fundamentals)
- [Triggers and Bindings](https://claude.ai/chat/28724acd-800c-45c9-8dec-4fc5505553c0#triggers-and-bindings)
- [Function Development Options](https://claude.ai/chat/28724acd-800c-45c9-8dec-4fc5505553c0#function-development-options)
- [Deployment and CI/CD](https://claude.ai/chat/28724acd-800c-45c9-8dec-4fc5505553c0#deployment-and-cicd)
- [Scaling and Performance](https://claude.ai/chat/28724acd-800c-45c9-8dec-4fc5505553c0#scaling-and-performance)
- [Security](https://claude.ai/chat/28724acd-800c-45c9-8dec-4fc5505553c0#security)
- [Monitoring and Troubleshooting](https://claude.ai/chat/28724acd-800c-45c9-8dec-4fc5505553c0#monitoring-and-troubleshooting)
- [Integration with Other Azure Services](https://claude.ai/chat/28724acd-800c-45c9-8dec-4fc5505553c0#integration-with-other-azure-services)
- [Exam Tips & Gotchas](https://claude.ai/chat/28724acd-800c-45c9-8dec-4fc5505553c0#exam-tips--gotchas)
- [Practice Questions](https://claude.ai/chat/28724acd-800c-45c9-8dec-4fc5505553c0#practice-questions)
- [Additional Resources](https://claude.ai/chat/28724acd-800c-45c9-8dec-4fc5505553c0#additional-resources)

## 🚀 Introduction to Azure Functions

Azure Functions is a serverless compute service that enables you to run code on-demand without having to explicitly provision or manage infrastructure.

### Key Concepts:

- **Serverless Compute**: Pay only for the time your code runs
- **Event-driven architecture**: Functions execute in response to events
- **Microservice approach**: Build small, single-purpose functions

### 💡 Exam Tip:

> Know the difference between Functions, Logic Apps, and WebJobs. Focus on when to use each one based on scenarios presented in the exam.

## 🔧 Function Fundamentals

### Function Apps

- A function app provides the execution context for your functions
- Multiple functions can share the same function app, scaling together
- Functions within the same function app share resources, configuration, and connections

### Hosting Plans

1. **Consumption Plan**:
       - Automatic scaling
    - Pay-per-execution billing
    - 5-minute timeout by default
    - 1.5GB memory limit
    
2. **Premium Plan**:
       - Pre-warmed instances for no cold starts
    - Unlimited execution duration
    - Higher memory instances (up to 14GB)
    - VNet connectivity
    - More predictable pricing
    
3. **App Service Plan**:    
    - Run on dedicated VMs
    - No timeout restrictions
    - Maximum memory based on plan size
    - Always running (even when idle)
    
4. **Kubernetes/Container**:    
    - KEDA-based scaling (Kubernetes Event-driven Autoscaling)
    - Run on AKS or any Kubernetes cluster

## Function app time-out duration
The `functionTimeout` property in the _host.json_ project file specifies the time-out duration for functions in a function app. This property applies specifically to function executions. After the trigger starts function execution, the function needs to return/respond within the time-out duration.

| Plan                      | Default | Maximum1   |
| ------------------------- | ------- | ---------- |
| **Flex Consumption plan** | 30      | Unbounded2 |
| **Premium plan**          | 304     | Unbounded2 |
| **Dedicated plan**        | 304     | Unbounded3 |
| **Container Apps**        | 30      | Unbounded5 |
| **Consumption plan**      | 5       | 10         |

### 💡 Exam Tip:

> Memorize the limits and features of each plan. The exam loves to present scenarios where you need to recommend the appropriate hosting plan based on specific requirements.

### ⚠️ Gotcha:

> The Consumption plan has a default timeout of 5 minutes, but can be configured up to 10 minutes. Premium and App Service plans can run for unlimited time.

# Scale Azure Functions

| Plan                  | Scale out                                                                                                                                                                                                   | Max # instances                                                            |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| Consumption plan      | Event driven. Scales out automatically, even during periods of high load. Functions infrastructure scales CPU and memory resources by adding more instances based on the number of incoming trigger events. | **Windows:** 200  <br>**Linux:** 1001                                      |
| Flex Consumption plan | Per-function scaling. Event-driven scaling decisions are calculated on a per-function basis, which provides a more deterministic way of scaling the functions in your app.                                  | Limited only by total memory usage of all instances across a given region. |
| Premium plan          | Event driven. Scale out automatically based on the number of events that its functions are triggered on.                                                                                                    | **Windows:** 100  <br>**Linux:** 20-1002                                   |
| Dedicated plan3       | Manual/autoscale                                                                                                                                                                                            | 10-30  <br>100 (ASE)                                                       |
| Container Apps        | Event driven. Scale out automatically by adding more instances of the Functions host, based on the number of events that its functions are triggered on.                                                    | 10-                                                                        |

## 🔄 Triggers and Bindings

### Triggers

Triggers define how a function is invoked. Each function must have exactly one trigger.

#### Common Triggers:

- **HTTP**: REST API calls
- **Timer**: Time-based triggers using CRON expressions
- **Blob Storage**: When a blob is added/updated
- **Queue Storage**: Process Azure Storage queue messages
- **Event Hub**: Process event streams
- **Service Bus**: Process Service Bus messages
- **Cosmos DB**: React to Cosmos DB changes
- **Event Grid**: Event Grid events or webhooks

```json
{
    "dataType": "binary",
    "type": "httpTrigger",
    "name": "req",
    "direction": "in"
}
```

```json
{
  "disabled": false,
    "bindings": [
        {
            "type": "queueTrigger",
            "direction": "in",
            "name": "myQueueItem",
            "queueName": "myqueue-items",
            "connection":"MyStorageConnectionAppSetting"
        },
        {
          "tableName": "Person",
          "connection": "MyStorageConnectionAppSetting",
          "name": "tableBinding",
          "type": "table",
          "direction": "out"
        }
  ]
}
```
#### Detailed Trigger Capabilities:

##### 1. HTTP Trigger

- **Capabilities**:
    - Supports all HTTP methods (GET, POST, PUT, DELETE, etc.)
    - Route templates for RESTful API patterns
    - Query parameter and request body access
    - Function-level authentication (anonymous, function, admin)
    - OpenAPI definition generation
    - Webhook support with validation
- **Configuration Options**:
    - Route templates: `/products/{id}`
    - Authorization level: `anonymous`, `function`, `admin`
    - Selected HTTP methods: `get`, `post`, etc.
- **C# Implementation**:

```csharp
[FunctionName("HttpTriggerExample")]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = "products/{id}")] HttpRequest req,
    string id,
    ILogger log)
{
    log.LogInformation($"HTTP trigger processing product ID: {id}");
    // Function implementation
    return new OkObjectResult($"Product ID: {id}");
}
```

- **Performance Considerations**:
    - HTTP triggers scale based on request volume
    - Concurrent requests: approx. 100 per instance
    - Cold start affects first-time responses

##### 2. Timer Trigger

- **Capabilities**:
    - Schedule execution using CRON expressions
    - Support for UTC and local time zones
    - Run on startup option
    - Timer status monitoring (to prevent missed executions)
- **Configuration Options**:
    - CRON expression (6-part format)
    - RunOnStartup: boolean
    - UseMonitor: boolean (prevents missed timers)
    - TimeZone: string (default: UTC)
- **C# Implementation**:

```csharp
[FunctionName("TimerTriggerExample")]
public static void Run(
    [TimerTrigger("0 */5 * * * *", RunOnStartup = true)] TimerInfo myTimer, 
    ILogger log)
{
    log.LogInformation($"Timer executed at: {DateTime.Now}");
    // Function implementation
}
```

- **Limitations**:
    - Minimum resolution: 1 minute (Consumption plan)
    - Sub-minute resolution: Premium/App Service plans only
    - Single instance execution in scale-out situations

##### 3. Blob Trigger

- **Capabilities**:
    - Trigger when blobs are added or updated
    - Pattern matching for blob names
    - Metadata access
    - Deserialize blob content to strongly-typed objects
- **Configuration Options**:
    - Path: container/pattern (e.g., `samples/{name}`)
    - Connection: connection string name
- **C# Implementation**:

```csharp
[FunctionName("BlobTriggerExample")]
public static void Run(
    [BlobTrigger("samples/{name}", Connection = "AzureStorageConnection")] Stream myBlob,
    string name,
    ILogger log)
{
    log.LogInformation($"Blob trigger processing: {name}");
    // Function implementation
}
```

- **Important Considerations**:
    - Uses storage polling (not events), which can cause delays
    - Cannot guarantee exactly-once processing
    - Not recommended for high-volume scenarios (consider Event Grid)
    - Lease blob mechanism for scale-out scenarios

##### 4. Queue Trigger

- **Capabilities**:
    - Process messages from Azure Storage queues
    - Automatic retry for poison messages
    - Batch processing support
    - Typed message binding
- **Configuration Options**:
    - QueueName: name of the queue
    - Connection: connection string name
    - BatchSize: how many messages to process at once (default: 16)
    - PoisonQueueName: where to send failed messages
- **C# Implementation**:

```csharp
[FunctionName("QueueTriggerExample")]
public static void Run(
    [QueueTrigger("myqueue", Connection = "AzureStorageConnection")] string myQueueItem,
    ILogger log)
{
    log.LogInformation($"Queue trigger processing: {myQueueItem}");
    // Function implementation
}
```

- **Scaling Behavior**:
    - Scales based on queue length
    - Processes approx. 100 queue messages per function instance
    - Message visibility timeout: 10 minutes (by default)
    - Automatic batching for performance

##### 5. Event Hub Trigger

- **Capabilities**:
    - Process events from Azure Event Hubs
    - High-throughput event processing
    - Consumer groups support
    - Batch processing
    - Checkpoint management
- **Configuration Options**:
    - EventHubName: name of the Event Hub
    - Connection: connection string name
    - ConsumerGroup: consumer group name (default: $Default)
    - CardinaldeBehavior: ProcessingOrder
- **C# Implementation**:

```csharp
[FunctionName("EventHubTriggerExample")]
public static async Task Run(
    [EventHubTrigger("myeventhub", Connection = "EventHubConnection", ConsumerGroup = "$Default")] EventData[] events,
    ILogger log)
{
    foreach (EventData eventData in events)
    {
        string messageBody = Encoding.UTF8.GetString(eventData.Body.Array, eventData.Body.Offset, eventData.Body.Count);
        log.LogInformation($"Event Hub message: {messageBody}");
        // Function implementation
    }
}
```

- **Performance Characteristics**:
    - Designed for high throughput scenarios
    - Partition-based scaling
    - Event batching for efficiency
    - Checkpoint storage in blob storage

##### 6. Service Bus Trigger

- **Capabilities**:
    - Process messages from Service Bus queues/topics
    - Automatic completion
    - Session support
    - Advanced message properties access
    - Dead letter handling
- **Configuration Options**:
    - QueueName/TopicName: queue or topic name
    - SubscriptionName: for topics only
    - Connection: connection string name
    - AutoComplete: automatic completion (default: true)
- **C# Implementation**:

```csharp
[FunctionName("ServiceBusTriggerExample")]
public static void Run(
    [ServiceBusTrigger("myqueue", Connection = "ServiceBusConnection")] Message message,
    ILogger log)
{
    log.LogInformation($"ServiceBus message: {Encoding.UTF8.GetString(message.Body)}");
    log.LogInformation($"MessageId: {message.MessageId}");
    // Function implementation
}
```

- **Advanced Features**:
    - FIFO guarantee (with sessions)
    - Message lock renewal for long-running processing
    - Complete/abandon/dead-letter control
    - Message cloning and forwarding

##### 7. Cosmos DB Trigger

- **Capabilities**:
    - React to inserts/updates in Cosmos DB containers
    - Change feed processing
    - Leases for distributed processing
    - Batch processing
- **Configuration Options**:
    - DatabaseName: Cosmos DB database name
    - CollectionName: Container name
    - Connection: connection string name
    - LeaseCollectionName: lease container name
    - CreateLeaseCollectionIfNotExists: boolean
- **C# Implementation**:

```csharp
[FunctionName("CosmosDBTriggerExample")]
public static void Run(
    [CosmosDBTrigger("database", "container", Connection = "CosmosDBConnection", 
     LeaseCollectionName = "leases", CreateLeaseCollectionIfNotExists = true)] IReadOnlyList<Document> documents,
    ILogger log)
{
    if (documents != null && documents.Count > 0)
    {
        log.LogInformation($"Documents modified: {documents.Count}");
        // Function implementation
    }
}
```

- **Limitations and Considerations**:
    - Uses Change Feed processor internally
    - Only detects inserts and updates (not deletions)
    - Processing order within a partition is guaranteed
    - Eventual consistency when scaling out
    - Performance depends on RU allocation

##### 8. Event Grid Trigger

- **Capabilities**:
    - React to Azure Event Grid events
    - Webhook validation handling
    - Strongly-typed event data
    - CloudEvents support
- **Configuration Options**:
    - No special configuration required
- **C# Implementation**:

```csharp
[FunctionName("EventGridTriggerExample")]
public static void Run(
    [EventGridTrigger] EventGridEvent eventGridEvent,
    ILogger log)
{
    log.LogInformation($"Event Grid event: {eventGridEvent.EventType}");
    log.LogInformation($"Event data: {eventGridEvent.Data}");
    // Function implementation
}
```

- **Use Cases**:
    - Resource events (Blob created, VM started, etc.)
    - Custom events from applications
    - Near real-time reacting to Azure resource changes
    - Serverless event-driven architectures
    - IoT scenarios

### Cross-Trigger Features:

#### 1. Retry Policies

- Configure retry behavior for failed executions
- Fixed delay or exponential backoff strategies
- Maximum retry count configuration

```json
{
  "retry": {
    "strategy": "fixedDelay",
    "maxRetryCount": 5,
    "delayInterval": "00:00:10"
  }
}
```

#### 2. Concurrency Control

- Control the maximum number of concurrent function instances
- Applicable to non-HTTP triggers
- Define in host.json

```json
{
  "extensions": {
    "queues": {
      "maxPollingInterval": "00:00:02",
      "visibilityTimeout": "00:00:30",
      "batchSize": 16,
      "maxDequeueCount": 5,
      "newBatchThreshold": 8
    }
  }
}
```

#### 3. Function-to-Function Communication

- Direct invocation between functions
- Durable functions for complex orchestrations
- Queue/Event-based communication for decoupling

### 💡 Exam Tip:

> Know how to configure different types of triggers and bindings in both host.json and function.json files. Also understand the differences between input and output bindings.

### ⚠️ Gotcha:

> Remember that Event Grid and Event Hub are different services. Event Grid is for discrete events, while Event Hub is for high-throughput event streams. The exam often tries to confuse these.

## 🖥️ Function Development Options

### Languages Supported:

- C# (compiled and script)
- F#
- JavaScript/TypeScript
- PowerShell
- Python
- Java

### Development Environments:

1. **Azure Portal**:
    
    - In-browser development experience
    - Limited testing capabilities
    - No source control integration
2. **Local Development**:
    
    - Visual Studio or VS Code with Azure Functions Core Tools
    - Local debugging
    - Extensions for productivity
3. **Command Line**:
    
    - Azure Functions Core Tools
    - Cross-platform CLI tools
    - Automation friendly

### Function Project Structure:

- **host.json**: Configures the function app
- **local.settings.json**: Local settings and connection strings (local dev only)
- **function.json**: Configuration for individual functions (triggers, bindings)
- **{function-name}/**: Folder containing function code and configuration

### C# Development Approaches:

1. **C# Class Library** (compiled):
    
    - Uses class and method attributes for triggers/bindings
    - Fully typed binding parameters
    - Compiled as DLL
2. **C# Script**:
    
    - Uses function.json for configuration
    - File extension: .csx
    - Interpreted at runtime

### 💡 Exam Tip:

> For C# developers, know both class library and script approaches. The exam may ask about differences between these implementation styles and when to use each.

### ⚠️ Gotcha:

> Understand that the .NET isolated worker process model differs from the in-process model, especially regarding binding implementations. Isolated worker uses different binding attributes and patterns.

## 📦 Deployment and CI/CD

### Deployment Methods:

1. **Zip Deployment**:
    
    - Package and deploy function code as a zip
    - `az functionapp deployment source config-zip`
    - Support for run-from-package (read-only)
2. **GitHub Actions**:
    
    - CI/CD integration with GitHub
    - Automated workflow on push/PR
3. **Azure DevOps Pipelines**:
    
    - Full CI/CD pipeline support
    - Test, build, and deploy
4. **Docker Containers**:
    
    - Package functions as containers
    - Deploy to Premium, App Service, or Kubernetes plans

### Deployment Slots:

- Test in a staging environment
- Swap slots for zero-downtime deployments
- Each slot has its own configuration

### Application Settings:

- Environment variables accessible to functions
- Connection strings and secrets
- Available in Azure Portal or via Azure CLI

### 💡 Exam Tip:

> Know the difference between WEBSITE_RUN_FROM_PACKAGE=1 and WEBSITE_RUN_FROM_PACKAGE=[URL]. One runs from a local package, the other from a URL.

### ⚠️ Gotcha:

> Deployment slots are only available in Premium and App Service plans, not in Consumption plans.

## 📈 Scaling and Performance

### Consumption Plan Scaling:

- Scale based on event rate (queue messages, HTTP requests)
- Scale to zero when idle
- Scale controlled by the scale controller
- Scale limit: 200 instances per function app

### Scale Units:

- Scale controller monitors triggers
- Creates new instances based on predefined thresholds
- HTTP triggers: 100 concurrent requests per instance

### Cold Start:

- Delay when a new instance is initialized
- More noticeable in Consumption plans
- Reduced in Premium plans with pre-warmed instances

### Performance Optimization:

- Use async patterns in C#
- Reuse HTTP clients and other connections
- Implement singleton patterns for shared resources
- Avoid long startup code in function constructors

### 💡 Exam Tip:

> Understand scale controller behavior and how different triggers affect scaling decisions. Know how to mitigate cold starts.

### ⚠️ Gotcha:

> Functions scale differently based on trigger type. HTTP triggers scale based on request rate, while queue triggers scale based on queue length.

## 🔒 Security

### Authentication and Authorization:

- **Function-level auth**: API keys, OAuth tokens
- **Built-in Auth/Auth**: Azure AD, Microsoft accounts, etc.
- **Custom auth**: Implement custom authentication logic

### Managed Identity:

- System-assigned (`default`) or user-assigned (`credential` and `clientID`)
- Access Azure resources without storing credentials
- Simplified secret management

### Secret Management:

- **Application settings**: For simple scenarios
- **Key Vault references**: `@Microsoft.KeyVault(SecretUri=...)`
- **Key Vault with Managed Identity**: Most secure approach

### Network Security:

- **VNet Integration**: Connect to resources in a VNet
- **Private Endpoints**: Expose functions privately
- **IP Restrictions**: Limit inbound traffic

### 💡 Exam Tip:

> Understand how to use managed identities with Azure Functions to access other Azure services securely without storing connection strings.

### CRON Expressions for Timer Triggers

Timer triggers in Azure Functions use CRON expressions to define schedules. Understanding CRON syntax is crucial for the AZ-204 exam.

#### CRON Format in Azure Functions:

Azure Functions uses a 6-part CRON expression:

```
{second} {minute} {hour} {day} {month} {day-of-week}
```

- **Second**: 0-59
- **Minute**: 0-59
- **Hour**: 0-23
- **Day**: 1-31
- **Month**: 1-12 or names (JAN, FEB, etc.)
- **Day-of-week**: 0-6 or names (SUN, MON, etc.) where 0 = Sunday

#### Special Characters:

- *****: Any value (wildcard)
- **,**: Value list separator ("2,4,6")
- **-**: Range of values ("1-5")
- **/**: Step values ("0/15" = every 15 units starting from 0)
- **?**: No specific value (only for day-of-month and day-of-week)

#### Common CRON Examples:

- `0 */5 * * * *`: Every 5 minutes
- `0 0 * * * *`: Every hour (on the hour)
- `0 0 0 * * *`: Once a day at midnight
- `0 0 9-17 * * 1-5`: Every hour from 9 AM to 5 PM, Monday to Friday
- `0 30 9 * * 1`: Every Monday at 9:30 AM
- `0 0 0 1 * *`: First day of every month at midnight
- `0 0 0 * * 0`: Every Sunday at midnight

#### Timer Trigger Configuration:

```json
{
  "schedule": "0 */5 * * * *",
  "name": "myTimer",
  "type": "timerTrigger",
  "direction": "in"
}
```

#### C# Timer Example:

```csharp
[FunctionName("TimerFunction")]
public static void Run(
    [TimerTrigger("0 */5 * * * *")] TimerInfo myTimer,
    ILogger log)
{
    log.LogInformation($"Timer function executed at: {DateTime.Now}");
}
```

#### Additional Timer Settings:

- **RunOnStartup**: Run immediately when function starts (boolean)
- **UseMonitor**: Enable timer monitoring for reliability (boolean)

#### Time Zones in Timer Triggers:

- CRON expressions are evaluated in UTC by default
- To use a different time zone:
    1. Install the `Microsoft.Azure.WebJobs.Extensions.TimerTrigger` NuGet package
    2. Specify the time zone name in the TimerTrigger attribute:

```csharp
[TimerTrigger("0 30 9 * * *", RunOnStartup = false, UseTimeZone = true)] 
TimerInfo timer,
[TimerTriggerTimeZoneInfo("Eastern Standard Time")] TimeZoneInfo timeZone
```

#### ⚠️ Gotchas:

- The 6-part CRON format in Azure Functions is different from standard 5-part Linux CRON
- Azure Functions timer has minimum resolution of 1 minute in Consumption plan
- For sub-minute timers, use Premium or App Service plans
- Timer triggers have reduced accuracy on Consumption plan during scale-out operations

## 📊 Monitoring and Troubleshooting

### Application Insights:

- Automatic integration with Azure Functions
- Log telemetry, exceptions, and dependencies
- Query logs with Kusto Query Language (KQL)

### Function Logs:

- View logs in real-time in the Azure Portal
- Configure log levels in host.json
- Stream logs locally with Azure Functions Core Tools

### Metrics:

- Execution count
- Execution time
- Error rate
- Memory usage

### Distributed Tracing:

- Trace requests across multiple services
- Visualize end-to-end transactions
- Identify bottlenecks

### 💡 Exam Tip:

> Know how to configure different logging levels in host.json and how to query function logs using Application Insights.

### ⚠️ Gotcha:

> Application Insights has its own separate cost. High volume functions can generate significant Application Insights charges if you log too verbosely.

## 🔄 Integration with Other Azure Services

 >⚠️ **Note**
 >An app running in a Consumption or Elastic Premium plan, uses the `WEBSITE_AZUREFILESCONNECTIONSTRING` and `WEBSITE_CONTENTSHARE` settings when connecting to Azure Files on the storage account used by your function app. Azure Files doesn't support using managed identity when accessing the file share.

### Common Integration Patterns:

#### Storage Services:

- Blob Storage: Store and retrieve files
- Table Storage: Simple NoSQL data storage
- Queue Storage: Asynchronous processing

#### Database Services:

- Cosmos DB: Global-scale NoSQL database
- SQL Database: Relational data storage
- Redis Cache: In-memory caching

#### Messaging Services:

- Event Grid: Reactive event handling
- Event Hubs: Big data streaming
- Service Bus: Enterprise messaging

#### Other Services:

- API Management: Expose functions as managed APIs
- Logic Apps: Orchestrate complex workflows
- Durable Functions: Stateful function orchestration

### 💡 Exam Tip:

> Understand the common patterns for integrating Functions with other Azure services. Know which binding to use for each service.

### ⚠️ Gotcha:

> When using App Service Authentication with Azure Functions, it's enabled at the function app level, not the individual function level.

## 🎯 Exam Tips & Gotchas

### Top Exam Tips:

1. **Know your hosting plans**:
    
    - Memorize the limits and features of each plan
    - Understand pricing implications
2. **Understand trigger/binding combinations**:
    
    - HTTP triggers can't use queue output bindings directly
    - Some bindings require extension bundles
3. **Be familiar with function.json and host.json**:
    
    - Configuration properties
    - Version differences
4. **Master C# binding syntax**:
    
    - Attributes for class library approach
    - function.json for script approach
5. **Understand scaling behavior**:
    
    - Different triggers scale differently
    - Know the limits (200 instances in Consumption plan)

### Common Gotchas:

1. **Execution time limits**:
    
    - Consumption: 5-10 minutes max
    - Premium/App Service: Unlimited
2. **Cold starts**:
    
    - More pronounced in Consumption plan
    - Mitigated by Premium plan pre-warmed instances
3. **Binding extensions**:
    
    - Some bindings require extension bundle configuration
    - Different versions support different extensions
4. **Local development**:
    
    - local.settings.json not deployed automatically
    - Set ASPNETCORE_ENVIRONMENT=Development for local debugging
5. **Security considerations**:
    
    - Don't store secrets in code
    - Use managed identities where possible
6. **Networking limitations**:
    
    - VNet integration only in Premium and App Service plans
    - Outbound IPs can change
7. **Durable Functions constraints**:
    
    - Orchestrator code must be deterministic
    - Entity functions have concurrency limitations

## 🧪 Practice Questions

1. **Which hosting plan should you choose for a function that needs to connect to resources in a virtual network and has unpredictable traffic patterns?**
    
    - A) Consumption Plan
    - B) Premium Plan
    - C) App Service Plan
    - D) Kubernetes Plan
    - Answer: B) Premium Plan
2. **You have a function that processes images uploaded to Blob Storage. The processing typically takes 8 minutes per image. Which hosting plan should you use?**
    
    - A) Consumption Plan
    - B) Premium Plan
    - C) App Service Plan
    - D) Either B or C
    - Answer: D) Either B or C (both Premium and App Service plans support executions longer than 5 minutes)
3. **You need to implement a function that triggers when a new document is added to a Cosmos DB container. Which trigger should you use?**
    
    - A) HTTP trigger
    - B) Blob trigger
    - C) Cosmos DB trigger
    - D) Event Grid trigger
    - Answer: C) Cosmos DB trigger
4. **Which of the following statements about Durable Functions is correct?**
    
    - A) Orchestrator functions can use random number generation
    - B) Activity functions cannot call other activity functions
    - C) Entity functions manage state for orchestrations
    - D) Durable Functions require the Premium hosting plan
    - Answer: C) Entity functions manage state for orchestrations
5. **Your function app needs to access a SQL database securely without storing connection strings. What should you use?**
    
    - A) Environment variables
    - B) local.settings.json
    - C) Managed Identity
    - D) Connection strings in app settings
    - Answer: C) Managed Identity

## 🔑 Key Takeaways

- Azure Functions is Microsoft's serverless compute offering
- Choose the right hosting plan based on requirements (Consumption, Premium, App Service)
- Understand triggers and bindings for different Azure services
- Master C# development approaches (class library vs. script)
- Know Durable Functions patterns for stateful workflows
- Be familiar with security, monitoring, and integration patterns
