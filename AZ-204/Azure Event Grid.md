# 📬 Azure Event Grid - AZ-204 Study Guide

## 🔍 Overview
Azure Event Grid is a fully managed event routing service that enables event-driven architectures. It sits between event sources (publishers) and event handlers (subscribers), facilitating the delivery of events in near real-time.

## 🏗️ Core Concepts

### 📦 Events
- **Definition**: Smallest unit of information describing something that happened in the system
- **Format**: JSON objects with required fields:
  - `id`: Unique identifier
  - `subject`: Resource path to the event source
  - `data`: Event-specific information
  - `eventType`: Registered event type
  - `eventTime`: Event generation time (UTC)
  - `dataVersion`: Schema version
  - `topic`: Full resource path to the event source (system-generated)

### 🔌 Event Sources (Publishers)
- Azure services that emit events:
  - 💾 Storage Accounts
  - 🧠 Azure App Service
  - 🚪 Event Hubs
  - ⚡ IoT Hub
  - 📱 Resource Groups/Subscriptions
  - 📈 Custom topics (your applications)

### 👂 Event Handlers (Subscribers)
Services that can receive and process events:
- ⚙️ Azure Functions
- 🔄 Logic Apps
- 🌉 Event Hubs
- 🚌 Service Bus
- 🔗 Webhooks
- 🧵 Queue Storage

### 📋 Topics
- **System Topics**: Pre-defined topics by Azure services
- **Custom Topics**: Application-defined topics for custom events
- **Partner Topics**: Topics for third-party services

### 🔀 Subscriptions
- **Definition**: The mechanism that defines which events from a topic are delivered to which handler
- **Filters**: Allow selective processing of events:
  - 📎 Subject filters: Filter by subject prefix/suffix/contains
  - 📊 Event Type filters: Filter by specific event types
  - 🧩 Advanced filters: Filter using JSON path expressions

## 💻 Key Features

### 🔄 Delivery & Retries
- Events are delivered within milliseconds
- Retry policy:
  - Default: 24-hour expiration with exponential backoff
  - Customizable TTL (1 min to 24 hours)
  - Dead-lettering support for undeliverable events

### 🔒 Security
- **Authentication Methods**:
  - Shared Access Signatures (SAS)
  - Service Principal Authentication
  - Managed Identity
  - Private endpoints

### 🔍 Monitoring
- Metrics available:
  - Published/delivered/dropped events
  - Match/delivery latency
  - Publish failures

## 🛠️ Implementation Examples (C#)

### Creating a Custom Topic
```csharp
// Create custom topic
string topicName = "mytopic";
var topicClient = new TopicClient(topicEndpoint, topicKey);

// Create event
EventGridEvent myEvent = new EventGridEvent(
    id: Guid.NewGuid().ToString(),
    subject: "myapp/vehicles/motorcycles",
    data: new { Make = "Ducati", Model = "Monster" },
    eventType: "recordInserted",
    eventTime: DateTime.UtcNow,
    dataVersion: "1.0"
);

// Publish event
await topicClient.PublishEventsAsync(new List<EventGridEvent> { myEvent });
```

### Subscribe to Events
```csharp
// Azure Function with EventGrid Trigger
[FunctionName("EventGridTrigger")]
public static void Run(
    [EventGridTrigger] EventGridEvent eventGridEvent,
    ILogger log)
{
    log.LogInformation($"Event Data: {eventGridEvent.Data}");
    log.LogInformation($"Event Type: {eventGridEvent.EventType}");
}
```

## 📋 Exam Tips & Gotchas

### 🚨 Common Pitfalls
- **Subject Filtering**: Remember that subject filters are prefix/suffix matches, not regex
- **Event Ordering**: No guarantee of event ordering (design for idempotent operations)
- **Timeouts**: Webhooks must respond to validation requests within 30 seconds
- **Delivery Guarantee**: At-least-once delivery (design for duplicate handling)
- **Size Limits**: 
  - Maximum event size: 1MB
  - Maximum batch size: 1MB
  - Maximum custom topic payload: 1MB

### 💡 Tips for the Exam
- Understand the difference between Event Grid, Event Hubs, and Service Bus:
  | Service | Purpose | When to Use |
  |---------|---------|-------------|
  | Event Grid | Reactive programming | React to status changes |
  | Event Hubs | Big data pipeline | Telemetry and distributed data streaming |
  | Service Bus | High-value enterprise messaging | Order processing and financial transactions |

- Know how to handle event validation for webhook endpoints:
  - Synchronous validation: Return validation code immediately 
  - Asynchronous validation: Manually send validation code to validation URL

- Be familiar with security concepts:
  - Event Grid supports Azure AD authentication for both publishing and subscribing
  - Webhook endpoints can be secured with validation keys or validation events

### 🔄 Integration with Other Azure Services
- **Functions**: Serverless event handling with Event Grid trigger
- **Logic Apps**: Visual workflow designer for event processing
- **Service Bus**: Combine with Event Grid for advanced message routing
- **Event Hubs**: Use for high-throughput event ingestion before processing

## 🔍 Advanced Scenarios

### 🌐 Disaster Recovery
- Event Grid is region-specific
- For multi-region resilience:
  - Deploy custom topics in multiple regions
  - Use Traffic Manager for webhook endpoints

### 📊 Schema Registry (Preview)
- Centrally manage event schemas
- Enforce schema validation
- Support schema evolution

### 🔐 Private Endpoints
- Secure communication over private network
- Eliminates public internet exposure
- Provides service endpoint policies

## 📝 Exam Practice Questions

1. Which of the following is NOT an event handler for Event Grid?
   - Azure Functions
   - Logic Apps
   - Cosmos DB ✓
   - Storage Queues

2. What is the maximum event size for Event Grid?
   - 64KB
   - 256KB
   - 1MB ✓
   - 5MB

3. Which authentication method is NOT supported by Event Grid?
   - SAS Token
   - Managed Identity
   - Basic Authentication ✓
   - Azure AD

4. How long does Event Grid retry delivering events?
   - 1 hour
   - 24 hours ✓
   - 48 hours
   - 7 days

## 🌟 Final Checklist
- Understand core concepts: events, publishers, subscribers
- Know authentication and security mechanisms
- Familiarize yourself with event schema and filtering
- Practice implementing custom topics and subscriptions in C#
- Understand integration patterns with other Azure services
- Remember the limitations and delivery guarantees
