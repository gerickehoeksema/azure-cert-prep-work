## 🌟 Overview

Azure Cosmos DB is Microsoft's globally distributed, multi-model database service designed for mission-critical applications. As an AZ-204 candidate, understanding Cosmos DB is crucial as it's a significant part of the exam.

---

## 📋 Table of Contents

- [Core Concepts](https://claude.ai/chat/9c5abc95-08d1-4780-84e3-2ba2e10c5465#core-concepts)
- [Data Models & APIs](https://claude.ai/chat/9c5abc95-08d1-4780-84e3-2ba2e10c5465#data-models--apis)
- [Partitioning](https://claude.ai/chat/9c5abc95-08d1-4780-84e3-2ba2e10c5465#partitioning)
- [Consistency Levels](https://claude.ai/chat/9c5abc95-08d1-4780-84e3-2ba2e10c5465#consistency-levels)
- [Indexing](https://claude.ai/chat/9c5abc95-08d1-4780-84e3-2ba2e10c5465#indexing)
- [Request Units](https://claude.ai/chat/9c5abc95-08d1-4780-84e3-2ba2e10c5465#request-units)
- [Security](https://claude.ai/chat/9c5abc95-08d1-4780-84e3-2ba2e10c5465#security)
- [SDK & Programming](https://claude.ai/chat/9c5abc95-08d1-4780-84e3-2ba2e10c5465#sdk--programming)
- [Scaling & Performance](https://claude.ai/chat/9c5abc95-08d1-4780-84e3-2ba2e10c5465#scaling--performance)
- [Backup & Disaster Recovery](https://claude.ai/chat/9c5abc95-08d1-4780-84e3-2ba2e10c5465#backup--disaster-recovery)
- [Cosmos DB vs Other Azure Data Services](https://claude.ai/chat/9c5abc95-08d1-4780-84e3-2ba2e10c5465#cosmos-db-vs-other-azure-data-services)
- [Exam Tips & Gotchas](https://claude.ai/chat/9c5abc95-08d1-4780-84e3-2ba2e10c5465#exam-tips--gotchas)
- [Practice Questions](https://claude.ai/chat/9c5abc95-08d1-4780-84e3-2ba2e10c5465#practice-questions)

---

## 🧩 Core Concepts

### What is Cosmos DB?

- **Global Distribution**: Automatically replicated worldwide with multi-region writes
- **Multi-Model Support**: Document, key-value, column-family, and graph databases
- **Horizontal Scalability**: Elastic scaling of throughput and storage
- **Comprehensive SLAs**: Guarantees for availability, latency, throughput, and consistency

### Key Features

- **Global Distribution**: Turnkey global distribution with multi-homing APIs
- **Multi-Region Writes**: Active-active setup across any number of Azure regions
- **Guaranteed Low Latency**: <10 ms reads and <15 ms writes at the 99th percentile
- **Automatic Indexing**: Automatic and customizable indexing of all properties
- **Tunable Consistency**: 5 well-defined consistency models to choose from

> ⚠️ **Exam Gotcha**: Know the differences between Cosmos DB and other Azure data services like Azure SQL Database and Azure Table Storage. The exam often tests your ability to select the right data service for specific scenarios.

---

## 📊 Data Models & APIs

Cosmos DB supports multiple data models through different APIs:

|API|Data Model|Use Case|
|---|---|---|
|SQL (Core) API|Document|General purpose document storage|
|MongoDB API|Document|Migration from MongoDB or using MongoDB drivers|
|Cassandra API|Column-family|Migration from Cassandra or using CQL|
|Gremlin API|Graph|Relationship-heavy data and graph queries|
|Table API|Key-value|Migration from Azure Table Storage|

### SQL (Core) API

- Primary and most feature-rich API
- Uses JSON documents
- SQL-like query language
- Recommended for new applications

```csharp
// C# example with SQL API
Container container = cosmosClient.GetContainer("DatabaseName", "ContainerName");
ItemResponse<dynamic> response = await container.CreateItemAsync(
    new { id = "1", name = "Item", category = "Sample" }
);
```

> 💡 **Exam Tip**: For the AZ-204 exam, focus primarily on the SQL API as it's the most commonly tested.

---

## 🧩 Partitioning

### Logical Partitioning

- **Partition Key**: Determines data distribution across physical partitions
- **Logical Partitions**: Items with the same partition key
- **Physical Partitions**: Backend storage units managed by Cosmos DB

### Choosing a Partition Key

Good partition keys should:

- Have a high cardinality (many distinct values)
- Distribute requests evenly
- Distribute data evenly

```csharp
// Creating a container with a partition key
ContainerProperties containerProperties = new ContainerProperties(
    id: "myContainer",
    partitionKeyPath: "/category"
);
```

> ⚠️ **Exam Gotcha**: The exam may present scenarios where you need to identify the optimal partition key. Remember that a good partition key evenly distributes both storage and throughput.

---

## 🔄 Consistency Levels

Cosmos DB offers five consistency levels, ordered from strongest to weakest:

1. **Strong**: Linearizability guarantee - reads are guaranteed to return the most recent committed version
2. **Bounded Staleness**: Reads lag behind writes by at most "K" versions or "T" time interval
3. **Session**: Consistent prefix, monotonic reads/writes, read-your-writes within a session
4. **Consistent Prefix**: Updates returned are some prefix of all the updates, with no gaps
5. **Eventual**: No ordering guarantee; reads eventually converge

```csharp
// Setting consistency level at the client
CosmosClientOptions options = new CosmosClientOptions
{
    ConsistencyLevel = ConsistencyLevel.Session
};
CosmosClient client = new CosmosClient(connectionString, options);
```

> 💡 **Exam Tip**: Understand the tradeoffs between consistency levels. Strong consistency has the highest latency and RU consumption, while Eventual has the lowest.

---

## 📑 Indexing

### Automatic Indexing

- By default, Cosmos DB automatically indexes all properties
- Can be customized for specific workloads
- Supports multiple index types: Range, Spatial, and Composite

### Index Policy

```json
{
  "indexingMode": "consistent",
  "automatic": true,
  "includedPaths": [
    {
      "path": "/*"
    }
  ],
  "excludedPaths": [
    {
      "path": "/metaData/*"
    }
  ]
}
```

> ⚠️ **Exam Gotcha**: The exam often tests how to optimize indexing for specific query patterns. Remember that excluding paths from indexing reduces RU consumption for writes.

---

## ⚡ Request Units (RUs)

### Understanding RUs

- RUs are the currency of throughput in Cosmos DB
- 1 RU = resources needed for a point read of a 1KB document
- Operations like writes, queries, and stored procedures consume more RUs

### Provisioning Models

- **Provisioned Throughput**: Set a fixed number of RUs per second
    - **Standard (Manual)**: Fixed RU/s set manually
    - **Autoscale**: Automatically scales between min and max RU/s (10% to 100% of max)
- **Serverless**: Pay only for consumed RUs, no minimum

```csharp
// Creating a container with 400 RU/s
ContainerProperties properties = new ContainerProperties("myContainer", "/partitionKey");
Container container = await database.CreateContainerAsync(
    properties, 
    throughput: 400
);
```

> 💡 **Exam Tip**: Know how to calculate RU costs for different operations and how to choose the right provisioning model based on workload patterns.

---

## 🔒 Security

### Authentication & Authorization

- **Master Keys**: Full administrative access
- **Resource Tokens**: Limited, time-bounded access to specific resources
- **Azure AD Integration**: Role-based access control (RBAC)

### Data Encryption

- **At Rest**: Automatic encryption of all data
- **In Transit**: TLS encryption for all communications

```csharp
// Using Azure AD authentication
TokenCredential credential = new DefaultAzureCredential();
CosmosClient client = new CosmosClient(endpoint, credential);
```

> ⚠️ **Exam Gotcha**: Know the differences between master keys and resource tokens, and when to use each one. The exam often tests security best practices.

---

## 💻 SDK & Programming

### C# SDK

```csharp
// Basic CRUD operations
// Create
ItemResponse<MyItem> createResponse = await container.CreateItemAsync(
    item: new MyItem { Id = "1", Name = "Sample" },
    partitionKey: new PartitionKey("Sample")
);

// Read
ItemResponse<MyItem> readResponse = await container.ReadItemAsync<MyItem>(
    id: "1",
    partitionKey: new PartitionKey("Sample")
);

// Update
MyItem item = readResponse.Resource;
item.Name = "Updated Sample";
ItemResponse<MyItem> replaceResponse = await container.ReplaceItemAsync(
    item: item,
    id: item.Id,
    partitionKey: new PartitionKey("Sample")
);

// Delete
ItemResponse<MyItem> deleteResponse = await container.DeleteItemAsync<MyItem>(
    id: "1",
    partitionKey: new PartitionKey("Sample")
);
```

### Querying

```csharp
// SQL query
QueryDefinition query = new QueryDefinition(
    "SELECT * FROM c WHERE c.Name = @name"
).WithParameter("@name", "Sample");

FeedIterator<MyItem> resultSet = container.GetItemQueryIterator<MyItem>(query);
while (resultSet.HasMoreResults)
{
    FeedResponse<MyItem> response = await resultSet.ReadNextAsync();
    foreach (MyItem item in response)
    {
        Console.WriteLine($"Id: {item.Id}, Name: {item.Name}");
    }
}
```

> 💡 **Exam Tip**: Be familiar with common Cosmos DB SDK operations and how to optimize them for performance and cost.

---

## 📈 Scaling & Performance

### Scaling Strategies

- **Horizontal Scaling**: Add more physical partitions (automatic)
- **Vertical Scaling**: Increase RUs per container/database

### Performance Optimization

- Use appropriate partition keys
- Optimize queries to minimize cross-partition queries
- Implement appropriate indexing policies
- Use direct mode connection policy where possible

```csharp
// Optimized connection policy
CosmosClientOptions options = new CosmosClientOptions
{
    ConnectionMode = ConnectionMode.Direct,
    ApplicationRegion = Regions.WestUS2
};
```

> ⚠️ **Exam Gotcha**: The exam may present scenarios where you need to diagnose and resolve performance issues. Understand how partition key choice affects query performance.

---

## 🔄 Backup & Disaster Recovery

### Backup Options

- **Periodic Automatic Backups**: Default backup mechanism (point-in-time restore)
- **Continuous Backup**: Continuous backup with point-in-time restore
- **Custom Backups**: Manual export of data

### Multi-Region Deployment

- **Active-Active**: Write to any region, data automatically replicated
- **Active-Passive**: Write to primary, read from secondaries

> 💡 **Exam Tip**: Know the differences between periodic and continuous backup modes, and when to use each.

---

## 🔄 Cosmos DB vs Other Azure Data Services

|Feature|Cosmos DB|Azure SQL|Azure Table Storage|
|---|---|---|---|
|Model|Multi-model|Relational|Key-value|
|Scaling|Horizontal|Vertical|Horizontal|
|Distribution|Global, built-in|Manual via replication|Limited, no built-in|
|Query|SQL-like, native APIs|T-SQL|OData, limited query|
|Consistency|Multiple levels|Strong|Strong within partition|
|Pricing|RU-based|DTU/vCore-based|Storage + transaction|

> ⚠️ **Exam Gotcha**: Be prepared to select the appropriate data service based on specific requirements such as global distribution, consistency needs, or schema flexibility.

---

## 🎯 Exam Tips & Gotchas

### Top Exam Tips

1. **Know Your RUs**: Understand how different operations consume RUs and how to optimize costs
2. **Consistency Tradeoffs**: Memorize the five consistency levels and their tradeoffs
3. **Partitioning Strategy**: Be able to choose appropriate partition keys for different scenarios
4. **API Selection**: Know when to use each API model based on requirements
5. **Cost Optimization**: Understand strategies to reduce costs while maintaining performance

### Common Gotchas

1. **Cross-Partition Queries**: These can be expensive and slow; know how to avoid them
2. **Changing Partition Keys**: You can't change a container's partition key after creation without migrating data
3. **Consistency vs Performance**: Strong consistency has performance and cost implications
4. **RU Estimation**: The exam may test your ability to estimate RU needs for specific workloads
5. **SDK Versions**: Be aware of differences between older SDKs (V2) and newer ones (V3)

> 💡 **Exam Tip**: When you see Cosmos DB questions, immediately check if they're testing your understanding of partition keys, consistency models, or RU optimization.

---

## ❓ Practice Questions

1. Which consistency level provides the strongest guarantee but may impact performance?
    
    - A) Eventual
    - B) Session
    - C) Strong
    - D) Bounded Staleness
2. A company needs to store product data where each product belongs to a category. There are 5 categories and thousands of products. What would be a poor choice for a partition key?
    
    - A) Product ID
    - B) Category
    - C) Combination of Category and Product Name
    - D) Timestamp
3. Which of these operations consumes the most RUs for the same document size?
    
    - A) Point read operation
    - B) Write operation
    - C) Query with a filter on the partition key
    - D) Query without a filter on the partition key
4. Which API would you recommend for a new application with document data that needs SQL-like queries?
    
    - A) MongoDB API
    - B) Cassandra API
    - C) Gremlin API
    - D) SQL API

> 💡 **Answers**: 1-C, 2-B, 3-D, 4-D

---

## 📚 Resources

- [Official Microsoft Azure Cosmos DB Documentation](https://docs.microsoft.com/en-us/azure/cosmos-db/)
- [Cosmos DB Capacity Calculator](https://cosmos.azure.com/capacitycalculator/)
- [Cosmos DB Modeling and Partitioning](https://docs.microsoft.com/en-us/azure/cosmos-db/modeling-data)
- [Choose the Right API for Azure Cosmos DB](https://docs.microsoft.com/en-us/azure/cosmos-db/choose-api)

---

## 🚀 C# SDK Quick Reference

```csharp
// Initialize client
CosmosClient cosmosClient = new CosmosClient(connectionString);

// Create database
Database database = await cosmosClient.CreateDatabaseIfNotExistsAsync("myDatabase");

// Create container
ContainerProperties containerProperties = new ContainerProperties("myContainer", "/partitionKey");
Container container = await database.CreateContainerIfNotExistsAsync(containerProperties, throughput: 400);

// Create item
dynamic newItem = new { id = "1", partitionKey = "value", name = "Item 1" };
ItemResponse<dynamic> createResponse = await container.CreateItemAsync(newItem, new PartitionKey("value"));

// Read item
ItemResponse<dynamic> readResponse = await container.ReadItemAsync<dynamic>("1", new PartitionKey("value"));

// Query items
QueryDefinition query = new QueryDefinition("SELECT * FROM c WHERE c.name = @name").WithParameter("@name", "Item 1");
FeedIterator<dynamic> resultSet = container.GetItemQueryIterator<dynamic>(query);

// Update item
dynamic updateItem = readResponse.Resource;
updateItem.name = "Updated Item 1";
ItemResponse<dynamic> updateResponse = await container.ReplaceItemAsync(updateItem, "1", new PartitionKey("value"));

// Delete item
ItemResponse<dynamic> deleteResponse = await container.DeleteItemAsync<dynamic>("1", new PartitionKey("value"));
```
