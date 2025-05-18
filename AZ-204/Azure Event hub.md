
## 📑 Table of Contents

- [📋 Introduction to Azure Event Hubs](https://claude.ai/chat/fa593166-6083-4d53-93d1-4fd28c98d9d7#introduction-to-azure-event-hubs)
- [🔑 Core Concepts](https://claude.ai/chat/fa593166-6083-4d53-93d1-4fd28c98d9d7#core-concepts)
- [🏗️ Architecture](https://claude.ai/chat/fa593166-6083-4d53-93d1-4fd28c98d9d7#architecture)
- [⚖️ Event Hub Tiers and Capabilities](https://claude.ai/chat/fa593166-6083-4d53-93d1-4fd28c98d9d7#event-hub-tiers-and-capabilities)
- [⚙️ Creating and Configuring Event Hubs](https://claude.ai/chat/fa593166-6083-4d53-93d1-4fd28c98d9d7#creating-and-configuring-event-hubs)
- [🔄 Publishers and Consumers](https://claude.ai/chat/fa593166-6083-4d53-93d1-4fd28c98d9d7#publishers-and-consumers)
- [⚡ Event Processing](https://claude.ai/chat/fa593166-6083-4d53-93d1-4fd28c98d9d7#event-processing)
- [🔒 Authentication and Authorization](https://claude.ai/chat/fa593166-6083-4d53-93d1-4fd28c98d9d7#authentication-and-authorization)
- [📈 Scaling and Performance](https://claude.ai/chat/fa593166-6083-4d53-93d1-4fd28c98d9d7#scaling-and-performance)
- [📊 Monitoring and Diagnostics](https://claude.ai/chat/fa593166-6083-4d53-93d1-4fd28c98d9d7#monitoring-and-diagnostics)
- [🔌 Integration With Other Azure Services](https://claude.ai/chat/fa593166-6083-4d53-93d1-4fd28c98d9d7#integration-with-other-azure-services)
- [🌐 Disaster Recovery and Geo-Replication](https://claude.ai/chat/fa593166-6083-4d53-93d1-4fd28c98d9d7#disaster-recovery-and-geo-replication)
- [💻 C# Implementation Examples](https://claude.ai/chat/fa593166-6083-4d53-93d1-4fd28c98d9d7#c-implementation-examples)
- [🧩 Common Design Patterns](https://claude.ai/chat/fa593166-6083-4d53-93d1-4fd28c98d9d7#common-design-patterns)
- [✅ Exam Tips and Gotchas](https://claude.ai/chat/fa593166-6083-4d53-93d1-4fd28c98d9d7#exam-tips-and-gotchas)
- [❓ Practice Questions](https://claude.ai/chat/fa593166-6083-4d53-93d1-4fd28c98d9d7#practice-questions)

## 📋 Introduction to Azure Event Hubs

> _Azure Event Hubs is a big data streaming platform and event ingestion service capable of receiving and processing millions of events per second._

Azure Event Hubs acts as the "front door" for an event pipeline, often used in IoT scenarios, application instrumentation, user experience monitoring, or workflow processing.

|**Key Facts**|**Description**|
|---|---|
|📦 Service Type|Part of Azure's messaging services alongside Service Bus and Storage Queues|
|🚀 Throughput|Designed for high-throughput scenarios (millions of events per second)|
|⏱️ Processing|Enables real-time processing|
|🏗️ Architecture|Supports Event-Driven Architecture (EDA) for microservices|

### 🎯 Primary Use Cases

- 📱 **Telemetry/IoT data collection** - Gather data from connected devices
- 🌊 **Stream processing** - Real-time processing of event streams
- 📊 **Analytics pipelines** - Feed data into analytics solutions
- 📝 **Application logging** - Centralized collection of application logs
- 💳 **Transaction processing** - Handle high-volume transaction events

## 🔑 Core Concepts

### 📦 Event

> _An event is a small packet of information (a datagram) containing a notification._

- Events can be published individually or in batches
- Maximum size of 1MB per event
- Contains a payload and user-defined properties

### 🔼 Publisher/Producer

> _Any entity that sends data to Event Hubs._

- Publishers can publish events using:
    - HTTPS
    - AMQP 1.0
    - Kafka protocol (1.0+)
- Can use partition keys to organize related events

### 🔽 Consumer/Subscriber

> _Applications or services that read and process events from Event Hubs._

- Pull-based model for reading events
- Can checkpoint progress for resilience
- Grouped into consumer groups

### 🔀 Partition

> _Each Event Hub contains multiple partitions where events are stored._

- Range of 1-32 partitions per Event Hub
- Events are ordered within a partition
- Allows parallel reading by multiple consumers
- Partition count cannot be changed after creation ⚠️

### 👥 Consumer Group

> _A view (state, position, or offset) of an entire Event Hub._

- Default consumer group is created automatically
- Limit of 50 consumer groups per Event Hub (Standard/Premium)
- Enables multiple applications to have separate views

### ⚡ Throughput Units/Processing Units

> _The purchased units of capacity that control the throughput capacity._

- 1 TU = 1 MB/s or 1000 events per second ingress
- 2 MB/s egress
- Can be configured for auto-inflate

### 💾 Capture

> _A feature that automatically captures streaming data and saves it to Azure storage._

### 📥 Event Consumer Example

```csharp
using Azure.Messaging.EventHubs;
using Azure.Messaging.EventHubs.Consumer;
using Azure.Messaging.EventHubs.Processor;
using Azure.Storage.Blobs;
using System;
using System.Text;
using System.Threading;
using System.Threading.Tasks;

namespace EventHubsConsumer
{
    class Program
    {
        private const string ehConnectionString = "Endpoint=sb://your-namespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=your-key";
        private const string eventHubName = "your-eventhub";
        private const string blobConnectionString = "DefaultEndpointsProtocol=https;AccountName=youraccount;AccountKey=your-key;EndpointSuffix=core.windows.net";
        private const string containerName = "checkpoints";
        private const string consumerGroup = EventHubConsumerClient.DefaultConsumerGroupName;
        
        static async Task Main(string[] args)
        {
            Console.WriteLine("Starting the Event Processor...");
            
            // Create a blob container client for checkpointing
            BlobContainerClient storageClient = new BlobContainerClient(
                blobConnectionString, 
                containerName);
                
            // Create the container if it doesn't exist
            await storageClient.CreateIfNotExistsAsync();
            
            // Create an Event Processor Client
            EventProcessorClient processor = new EventProcessorClient(
                storageClient, 
                consumerGroup, 
                ehConnectionString, 
                eventHubName);
                
            // Register event handlers
            processor.ProcessEventAsync += ProcessEventHandler;
            processor.ProcessErrorAsync += ProcessErrorHandler;
            
            // Start processing
            await processor.StartProcessingAsync();
            
            Console.WriteLine("Processing events. Press any key to stop...");
            Console.ReadKey();
            
            // Stop processing
            await processor.StopProcessingAsync();
            Console.WriteLine("Event processor stopped");
        }
        
        static async Task ProcessEventHandler(ProcessEventArgs args)
        {
            try
            {
                // Get the event data
                string messageBody = Encoding.UTF8.GetString(args.Data.EventBody.ToArray());
                
                // Get partition information
                string partition = args.Partition.PartitionId;
                long sequenceNumber = args.Data.SequenceNumber;
                DateTimeOffset enqueuedTime = args.Data.EnqueuedTime;
                
                // Get custom properties
                string eventType = args.Data.Properties.TryGetValue("EventType", out object type) 
                    ? type.ToString() 
                    : "Unknown";
                    
                string deviceId = args.Data.Properties.TryGetValue("DeviceId", out object device) 
                    ? device.ToString() 
                    : "Unknown";
                
                // Log information
                Console.WriteLine($"Received event: '{messageBody}' from partition {partition}, " + 
                                 $"sequence #{sequenceNumber}, enqueuedTime: {enqueuedTime}, " +
                                 $"type: {eventType}, device: {deviceId}");
                
                // Process the event...
                await Task.Delay(100); // Simulating processing time
                
                // Update the checkpoint
                await args.UpdateCheckpointAsync();
            }
            catch (Exception ex)
            {
                Console.WriteLine($"Error processing event: {ex.Message}");
            }
        }
        
        static Task ProcessErrorHandler(ProcessErrorEventArgs args)
        {
            Console.WriteLine($"Error in partition '{args.PartitionId}': {args.Exception.Message}");
            return Task.CompletedTask;
        }
    }
}
```

## 🧩 Common Design Patterns

### 📐 Publisher-Subscriber Pattern

- Decouples producers from consumers
- Allows for multiple independent subscribers
- Enables parallel processing of events

```
                     ┌─────────────┐
                     │  Event Hub  │
                     │             │
┌───────────┐        │ ┌─────────┐ │        ┌───────────┐
│ Publisher │───────►│ │Partition├─┼───────►│ Subscriber│
└───────────┘        │ └─────────┘ │        └───────────┘
                     │             │
┌───────────┐        │ ┌─────────┐ │        ┌───────────┐
│ Publisher │───────►│ │Partition├─┼───────►│ Subscriber│
└───────────┘        │ └─────────┘ │        └───────────┘
                     │             │
                     └─────────────┘
```

### 🔄 Event Sourcing Pattern

- Store all state changes as events
- Rebuild state by replaying events
- Provides complete audit trail

```csharp
// Event sourcing with Event Hubs
public class EventSourcedEntity
{
    private readonly List<DomainEvent> _events = new List<DomainEvent>();
    private readonly EventHubProducerClient _producer;
    
    public EventSourcedEntity(EventHubProducerClient producer)
    {
        _producer = producer;
    }
    
    public async Task ApplyEvent(DomainEvent domainEvent)
    {
        // Update entity state
        _events.Add(domainEvent);
        
        // Serialize and send to Event Hub
        var eventData = new EventData(JsonSerializer.SerializeToUtf8Bytes(domainEvent));
        eventData.Properties.Add("EventType", domainEvent.GetType().Name);
        eventData.Properties.Add("AggregateId", domainEvent.AggregateId);
        
        await _producer.SendAsync(new[] { eventData });
    }
    
    public async Task RehydrateFromEvents(string aggregateId)
    {
        // Setup consumer
        var consumer = new EventHubConsumerClient(
            EventHubConsumerClient.DefaultConsumerGroupName,
            connectionString,
            eventHubName);
            
        // Read all events for this aggregate
        var events = consumer.ReadEventsFromPartitionAsync(
            partition,
            EventPosition.Earliest);
            
        await foreach (PartitionEvent partitionEvent in events)
        {
            if (partitionEvent.Data.Properties.TryGetValue("AggregateId", out var id) && 
                id.ToString() == aggregateId)
            {
                // Deserialize and apply event
                string json = Encoding.UTF8.GetString(partitionEvent.Data.Body.ToArray());
                var domainEvent = JsonSerializer.Deserialize<DomainEvent>(json);
                _events.Add(domainEvent);
            }
        }
    }
}
```

### 🔄 CQRS (Command Query Responsibility Segregation)

- Separate read and write operations
- Use Event Hubs for write operations
- Materialized views for read operations

```
┌───────────┐     ┌─────────────┐     ┌─────────────┐
│ Commands  │────►│ Event Hubs  │────►│ Event       │
└───────────┘     └─────────────┘     │ Processor   │
                                      └──────┬──────┘
                                             │
                                             ▼
┌───────────┐     ┌─────────────┐     ┌─────────────┐
│ Queries   │◄────│ Read Models │◄────│ Data Store  │
└───────────┘     └─────────────┘     └─────────────┘
```

### 🔄 Competing Consumers Pattern

- Multiple instances process events in parallel
- Automatic load distribution
- Horizontal scaling of event processing

```csharp
// Competing consumers pattern
public class WorkerService : BackgroundService
{
    private readonly EventProcessorClient _processor;
    
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        // Register event handlers
        _processor.ProcessEventAsync += ProcessEventHandler;
        _processor.ProcessErrorAsync += ProcessErrorHandler;
        
        // Start processing
        await _processor.StartProcessingAsync(stoppingToken);
        
        try
        {
            // Keep the service running until requested to stop
            await Task.Delay(Timeout.Infinite, stoppingToken);
        }
        catch (TaskCanceledException)
        {
            // Normal shutdown
        }
        finally
        {
            // Stop processing
            await _processor.StopProcessingAsync();
        }
    }
}
```

### 🔄 Lambda Architecture

- Combine batch and real-time processing
- Event Hubs for real-time path
- Capture for batch processing path

```
                     ┌─────────────┐
                     │  Event Hub  │
                     │             │
┌───────────┐        │ ┌─────────┐ │        ┌───────────────┐
│ Source    │───────►│ │         │ │───────►│ Speed Layer   │
└───────────┘        │ │         │ │        └───────┬───────┘
                     │ │         │ │                │
                     │ │         │ │                ▼
                     │ │         │ │        ┌───────────────┐
                     │ └─────────┘ │        │ Serving Layer │
                     │             │        └───────┬───────┘
                     └──────┬──────┘                │
                            │                       │
                            ▼                       │
                     ┌─────────────┐                │
                     │ Capture     │                │
                     └──────┬──────┘                │
                            │                       │
                            ▼                       │
                     ┌─────────────┐                │
                     │ Batch Layer │────────────────┘
                     └─────────────┘
```

## ✅ Exam Tips and Gotchas

### 🚨 Common Exam Gotchas

1. **Partition Count Limitations**
    
    - ⚠️ Partition count cannot be changed after creation
    - ⚠️ Basic tier: Max 32 partitions
    - ⚠️ Standard tier: Max 32 partitions
    - ⚠️ Premium tier: Up to 100 partitions
2. **Message Retention Limits**
    
    - ⚠️ Basic tier: 1 day only
    - ⚠️ Standard tier: 1-7 days
    - ⚠️ Premium tier: 1-90 days
    - ⚠️ Cannot be modified after creation for Basic tier
3. **Consumer Group Limits**
    
    - ⚠️ Basic tier: Only default consumer group
    - ⚠️ Standard tier: Up to 20 consumer groups
    - ⚠️ Premium tier: Up to 100 consumer groups
4. **Authentication and Security**
    
    - ⚠️ SAS tokens expire - check expiration policies
    - ⚠️ RBAC vs SAS authentication differences
    - ⚠️ Namespace-level vs entity-level policies
5. **Throughput Units**
    
    - ⚠️ Ingress limit: 1 MB/s or 1000 events per second per TU
    - ⚠️ Egress limit: 2 MB/s per TU
    - ⚠️ Standard tier max: 40 TUs (can request more)
    - ⚠️ Basic tier max: 20 TUs (cannot request more)
6. **Capture Feature**
    
    - ⚠️ Not available in Basic tier
    - ⚠️ Minimum capture window: 60 seconds or 10 MB
    - ⚠️ Cannot be used with Kafka protocol
7. **Geo-Disaster Recovery**
    
    - ⚠️ Metadata only - events are not replicated
    - ⚠️ Manual failover process
    - ⚠️ Only available for Standard and Premium tiers
8. **Kafka Endpoint**
    
    - ⚠️ Not available in Basic tier
    - ⚠️ Differences between Kafka native and Event Hubs

### 💡 Key Exam Tips

1. **Understanding Event Positions**
    
    - Know the difference between sequence number, offset, and enqueued time
    - Understand how to start from specific positions
2. **Authentication**
    
    - Know when to use SAS vs AAD
    - Understand token lifetimes and renewal strategies
3. **Partitioning Strategy**
    
    - Understand partition key selection best practices
    - Know the impact of partition count on throughput
4. **Tiers and SLAs**
    
    - Premium tier: 99.99% with Availability Zones
    - Standard tier: 99.95%
    - Basic tier: 99.95%
5. **Integration Knowledge**
    
    - Understand how Event Hubs connects with:
        - Stream Analytics
        - Functions
        - Logic Apps
        - IoT Hub
6. **Scaling Knowledge**
    
    - Understand auto-inflate
    - Know the throughput unit calculations
7. **Event Processing**
    
    - Understand checkpoint strategies
    - Know the difference between direct receivers and event processors
8. **Pricing Factors**
    
    - Ingress events
    - Throughput units/Processing units
    - Storage (for extended retention)
    - Capture
    - Standard operations

## ❓ Practice Questions

1. **Which of the following statements about Event Hubs partitions is true?**
    
    - A. The partition count can be increased after creation
    - B. The maximum number of partitions in the Premium tier is 32
    - C. Events with the same partition key will be sent to the same partition
    - D. Partition IDs always start from 0 and are consecutive
    
    **Answer: C** - Events with the same partition key will be sent to the same partition.
    
2. **How many consumer groups can you create in an Event Hub using the Standard tier?**
    
    - A. 5
    - B. 10
    - C. 20
    - D. 50
    
    **Answer: C** - The Standard tier supports up to 20 consumer groups.
    
3. **When using Event Hubs Capture, what is the minimum time window that can be configured?**
    
    - A. 30 seconds
    - B. 60 seconds
    - C. 5 minutes
    - D. 15 minutes
    
    **Answer: B** - The minimum time window for Capture is 60 seconds (1 minute).
    
4. **Which of the following is NOT a valid authentication method for Event Hubs?**
    
    - A. Shared Access Signature (SAS)
    - B. Azure Active Directory (AAD)
    - C. Basic authentication (username/password)
    - D. Managed identity
    
    **Answer: C** - Basic authentication is not supported for Event Hubs.
    
5. **What is the maximum event retention period for Event Hubs in the Premium tier?**
    
    - A. 7 days
    - B. 30 days
    - C. 90 days
    - D. 180 days
    
    **Answer: C** - Premium tier supports up to 90 days retention.
    
6. **Which protocol provides the best performance for high-throughput scenarios with Event Hubs?**
    
    - A. HTTP
    - B. HTTPS#### Partition Key Selection

- Choose partition keys that distribute events evenly
- Using device IDs is common for IoT scenarios
- Avoid "hot" partitions that receive most traffic

### 🔍 Performance Optimization

#### Publisher Optimizations

- Use batching to reduce network overhead
- Implement retry logic with exponential backoff
- Consider AMQP for high-throughput scenarios
- Pre-allocate EventData objects when possible

```csharp
// Efficient publisher with batching and retries
int maxRetries = 3;
TimeSpan delay = TimeSpan.FromSeconds(1);

for (int i = 0; i < maxRetries; i++)
{
    try
    {
        using EventDataBatch eventBatch = await producerClient.CreateBatchAsync(
            new CreateBatchOptions { PartitionKey = "device-category" });
        
        // Add multiple events efficiently
        foreach (var eventData in events)
        {
            if (!eventBatch.TryAdd(eventData))
            {
                // Batch is full, send and create a new one
                await producerClient.SendAsync(eventBatch);
                eventBatch = await producerClient.CreateBatchAsync(
                    new CreateBatchOptions { PartitionKey = "device-category" });
                
                if (!eventBatch.TryAdd(eventData))
                {
                    throw new Exception("Event is too large for empty batch");
                }
            }
        }
        
        // Send the final batch
        await producerClient.SendAsync(eventBatch);
        break; // Success, exit retry loop
    }
    catch (Exception ex) when (i < maxRetries - 1)
    {
        // Log exception and retry with backoff
        Console.WriteLine($"Send attempt {i+1} failed: {ex.Message}");
        await Task.Delay(delay);
        delay = TimeSpan.FromSeconds(delay.TotalSeconds * 2); // Exponential backoff
    }
}
```

#### Consumer Optimizations

- Increase PrefetchCount for high-throughput scenarios
- Process events in parallel within a partition
- Checkpoint in batches, not per event
- Tune MaxBatchSize for your workload

```csharp
// Optimized consumer configuration
var processorOptions = new EventProcessorClientOptions
{
    PrefetchCount = 500, // Default is 300
    MaxBatchSize = 200,  // Default is 100
    MaximumWaitTime = TimeSpan.FromSeconds(5)
};

// Parallel processing of events within a batch
async Task ProcessEventHandler(ProcessEventArgs args)
{
    try
    {
        // Process events in parallel if they don't require ordering
        var tasks = new List<Task>();
        foreach (var eventData in args.Events)
        {
            tasks.Add(ProcessSingleEventAsync(eventData));
        }
        
        await Task.WhenAll(tasks);
        
        // Checkpoint after processing the batch
        await args.UpdateCheckpointAsync();
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error in batch processing: {ex.Message}");
    }
}
```

## 📊 Monitoring and Diagnostics

### 📈 Azure Monitor Metrics

Event Hubs exposes the following key metrics:

#### Ingress Metrics:

- **IncomingBytes**: Bytes sent to Event Hubs
- **IncomingMessages**: Events sent to Event Hubs
- **IncomingRequests**: Number of send requests
- **ThrottledRequests**: Requests that exceeded capacity

#### Egress Metrics:

- **OutgoingBytes**: Bytes received from Event Hubs
- **OutgoingMessages**: Events received from Event Hubs
- **OutgoingRequests**: Number of receive requests

#### Capture Metrics:

- **CaptureBytes**: Bytes captured
- **CapturedMessages**: Events captured
- **CaptureBacklog**: Events pending capture

```powershell
# Get metrics using PowerShell
Get-AzMetric -ResourceId "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.EventHub/namespaces/{namespace}" `
             -MetricName "IncomingMessages" `
             -TimeGrain 00:01:00 `
             -StartTime (Get-Date).AddHours(-1) `
             -EndTime (Get-Date)
```

### 🔍 Diagnostic Logs

Event Hubs can send logs to:

- Azure Storage
- Azure Log Analytics
- Azure Event Hubs (for forwarding)

Log categories include:

- **ArchiveLogs**: Capture operations
- **OperationalLogs**: Operations performed on resources
- **AutoScaleLogs**: Auto-scaling operations
- **KafkaCoordinatorLogs**: Kafka coordinator operations
- **KafkaUserErrorLogs**: Kafka user errors
- **EventHubVNetConnectionEvents**: Virtual network connection events

```bash
# Enable diagnostic settings via CLI
az monitor diagnostic-settings create \
    --name "eventhub-diagnostics" \
    --resource "/subscriptions/{subscription}/resourceGroups/{resource-group}/providers/Microsoft.EventHub/namespaces/{namespace}" \
    --logs '[{"category": "ArchiveLogs", "enabled": true}]' \
    --metrics '[{"category": "AllMetrics", "enabled": true}]' \
    --workspace "/subscriptions/{subscription}/resourceGroups/{resource-group}/providers/microsoft.operationalinsights/workspaces/{workspace}"
```

### 🔎 Troubleshooting Common Issues

#### 1. Throttling Issues

- **Symptoms**: 429 errors, ThrottledRequests metric increasing
- **Solutions**:
    - Scale up throughput units
    - Enable auto-inflate
    - Implement retries with backoff
    - Review partition distribution

#### 2. Connectivity Issues

- **Symptoms**: Connection timeouts, connection refused
- **Solutions**:
    - Check network security group rules
    - Verify Private Link configuration
    - Test with Event Hub Explorer tool
    - Check IP filtering rules

#### 3. Authorization Issues

- **Symptoms**: 401 errors, 403 errors
- **Solutions**:
    - Verify SAS token hasn't expired
    - Check permissions of managed identity
    - Ensure correct role assignments
    - Verify connection string is correct

## 🔌 Integration With Other Azure Services

### 🔄 Common Integration Patterns

#### Azure Stream Analytics

Process and analyze event streams in real-time:

```sql
-- Stream Analytics query example
SELECT
    deviceId,
    AVG(temperature) AS avgTemperature,
    System.Timestamp AS windowEndTime
FROM
    EventHubInput TIMESTAMP BY eventTime
GROUP BY
    deviceId,
    TumblingWindow(minute, 5)
```

#### Azure Functions

Serverless event processing with Event Hub trigger:

```csharp
// Azure Function with Event Hub trigger
[FunctionName("ProcessEventHubMessages")]
public static async Task Run(
    [EventHubTrigger("myeventhub", Connection = "EventHubConnection")] EventData[] events,
    ILogger log)
{
    foreach (EventData eventData in events)
    {
        string messageBody = Encoding.UTF8.GetString(eventData.Body.Array);
        log.LogInformation($"Event: {messageBody}");
        
        // Process the event
        await ProcessMessageAsync(messageBody);
    }
}
```

#### Azure Logic Apps

Create workflows triggered by Event Hubs:

```json
{
  "triggers": {
    "When_events_are_available_in_Event_Hub": {
      "inputs": {
        "body": {
          "contentData": "@triggerBody()"
        },
        "host": {
          "connection": {
            "name": "@parameters('$connections')['eventhubs']['connectionId']"
          }
        },
        "method": "post",
        "path": "/@{encodeURIComponent('myeventhub')}/events/batch/head",
        "queries": {
          "consumerGroupName": "$Default",
          "contentType": "application/octet-stream",
          "maximumEventsCount": 50
        }
      },
      "type": "ApiConnection"
    }
  }
}
```

#### Azure Databricks

Process events with Apache Spark:

```python
# Databricks Python code for Event Hub consumption
connectionString = "Endpoint=sb://mynamespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=yourkey;EntityPath=myeventhub"

ehConf = {
  'eventhubs.connectionString': connectionString,
  'eventhubs.consumerGroup': "$Default"
}

# Read from Event Hub
df = spark.readStream \
  .format("eventhubs") \
  .options(**ehConf) \
  .load()

# Process the data
processedDF = df.select(F.from_json(F.col("body").cast("string"), schema).alias("data")) \
  .select("data.*")

# Write the output
query = processedDF.writeStream \
  .format("delta") \
  .outputMode("append") \
  .option("checkpointLocation", "/delta/events/_checkpoints/") \
  .start("/delta/events/")
```

#### Azure IoT Hub

Route IoT Hub messages to Event Hubs:

```json
{
  "properties.desired": {
    "schemaVersion": "1.0",
    "routing": {
      "routes": [
        {
          "name": "EventHubRoute",
          "source": "DeviceMessages",
          "condition": "true",
          "endpointNames": ["events"],
          "isEnabled": true
        }
      ],
      "fallbackRoute": {
        "name": "$fallback",
        "source": "DeviceMessages",
        "condition": "true",
        "endpointNames": ["events"],
        "isEnabled": true
      }
    }
  }
}
```

## 🌐 Disaster Recovery and Geo-Replication

### 🔄 Geo-Disaster Recovery

Event Hubs provides geo-disaster recovery through Azure Resource Manager and:

- **Pairing** - Links primary and secondary namespaces
- **Failover** - Swaps primary and secondary namespaces
- **Metadata synchronization** - Replicates entities and settings

```powershell
# Set up geo-disaster recovery
$primaryNamespace = Get-AzEventHubNamespace -ResourceGroupName myResourceGroup -Name primaryNamespace
$secondaryNamespace = Get-AzEventHubNamespace -ResourceGroupName myResourceGroup -Name secondaryNamespace

# Create pairing (alias)
New-AzEventHubGeoDRConfiguration -Name myDRConfig `
                               -ResourceGroupName myResourceGroup `
                               -Namespace $primaryNamespace.Name `
                               -PartnerNamespace $secondaryNamespace.Id `
                               -AlternateName mySecondaryAlias
```

#### Failover Process

```powershell
# Initiate failover
Start-AzEventHubGeoDRFailover -ResourceGroupName myResourceGroup `
                            -Namespace $secondaryNamespace.Name `
                            -Name myDRConfig
```

> ⚠️ **Exam Tip**: Failover is a manual process initiated by the customer, not automatic!

### 🏢 Availability Zones

Available in Premium tier for higher availability within a region:

- Distributes replicas across zones
- Maintains replication synchronously
- Provides 99.99% SLA

```json
// ARM template for Event Hubs with Availability Zones
{
  "resources": [
    {
      "type": "Microsoft.EventHub/namespaces",
      "apiVersion": "2021-11-01",
      "name": "myNamespace",
      "location": "eastus2",
      "sku": {
        "name": "Premium",
        "tier": "Premium",
        "capacity": 1
      },
      "properties": {
        "zoneRedundant": true
      }
    }
  ]
}
```

## 💻 C# Implementation Examples

### 📤 Event Producer Example

````csharp
using Azure.Messaging.EventHubs;
using Azure.Messaging.EventHubs.Producer;
using System;
using System.Collections.Generic;
using System.Text;
using System.Threading.Tasks;

namespace EventHubsProducer
{
    class Program
    {
        // Connection string to the Event Hubs namespace
        private const string connectionString = "Endpoint=sb://your-namespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=your-key";

        // Event Hub name
        private const string eventHubName = "your-eventhub";

        // Number of events to send
        private const int numOfEvents = 100;

        static async Task Main(string[] args)
        {
            // Create a producer client that you can use to send events to an event hub
            await using (var producerClient = new EventHubProducerClient(connectionString, eventHubName))
            {
                // Get information about the partition IDs and partition count
                EventHubProperties properties = await producerClient.GetEventHubPropertiesAsync();
                Console.WriteLine($"Event Hub Name: {eventHubName}");
                Console.WriteLine($"Partition Count: {properties.PartitionIds.Length}");
                Console.WriteLine("Sending events...");

                // Create a batch of events
                using EventDataBatch eventBatch = await producerClient.CreateBatchAsync();

                for (int i = 1; i <= numOfEvents; i++)
                {
                    // Create a new event to send
                    var eventData = new EventData(Encoding.UTF8.GetBytes($"Event {i}"));
                    
                    // Add metadata
                    eventData.Properties.Add("EventType", "DeviceReading");
                    eventData.Properties.Add("DeviceId", $"device-{i % 10}");
                    eventData.Properties.Add("Timestamp", DateTimeOffset.UtcNow);

                    // Try to add the event to the batch
                    if (!eventBatch.TryAdd(eventData))
                    {
                        // If the batch is full, send it and create a new batch
                        await producerClient.SendAsync(eventBatch);
                        Console.WriteLine($"Sent a batch of {eventBatch.Count} events");
                        
                        // Create a new batch
                        eventBatch = await producerClient.CreateBatchAsync();
                        
                        // Add the event that wouldn't fit to the new batch
                        if (!eventBatch.TryAdd(eventData))
                        {
                            throw new Exception("Event is too large for an empty batch");
                        }
                    }
                }

                // Send the last batch of remaining events
                if (eventBatch.Count > 0)
                {
                    await producerClient.SendAsync(eventBatch);
                    Console.WriteLine($"Sent a batch of {eventBatch.Count} events");
                }
                
                Console.WriteLine($"Sent {numOfEvents} events");
            }
        }
    }
}
