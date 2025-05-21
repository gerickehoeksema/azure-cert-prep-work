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

Cosmos DB offers five consistency levels that balance availability, performance, and consistency guarantees. This is a **critical exam topic** that's frequently tested in scenario-based questions.

### Detailed Consistency Levels (Strongest to Weakest)

#### 1. **Strong Consistency**

- **Guarantee**: Linearizability - reads always return the most recent committed version
- **How it works**:
    - Reads are guaranteed to see the latest committed write
    - Synchronous replication across all regions before write confirmation
- **Performance Impact**:
    - Highest latency for writes (waiting for all regions)
    - Highest RU consumption (approximately 2x read cost)
    - Reduced availability during regional failures
- **When to use**:
    - Financial transactions
    - Inventory systems
    - Critical record updates
    - Primary key enforcement

```csharp
CosmosClientOptions options = new CosmosClientOptions
{
    ConsistencyLevel = ConsistencyLevel.Strong
};
```

#### 2. **Bounded Staleness Consistency**

- **Guarantee**: Reads might lag behind writes by at most "K" versions or "T" time interval
- **How it works**:
    - You configure maximum lag in terms of:
        - Operations (versions): How many operations can be behind
        - Time: How much time (in seconds) reads can lag behind writes
- **Performance Impact**:
    - Moderate latency
    - Lower RU consumption than Strong
    - Better availability than Strong during regional failures
    - Balance between consistency and availability
- **When to use**:
    - High availability scenarios that can tolerate some staleness
    - Applications needing predictable consistency guarantees
    - Group collaboration apps with near-real-time requirements

```csharp
CosmosClientOptions options = new CosmosClientOptions
{
    ConsistencyLevel = ConsistencyLevel.BoundedStaleness,
    MaxStalenessPrefix = 100,        // Max number of operations
    MaxStalenessIntervalInSeconds = 5 // Max time in seconds
};
```

#### 3. **Session Consistency**

- **Guarantee**: Within a single client session:
    - Read-your-writes
    - Monotonic reads
    - Monotonic writes
    - Consistent prefix
- **How it works**:
    - Uses session tokens to track writes
    - Only guarantees consistency within the same session
    - Different sessions may see different versions
- **Performance Impact**:
    - Low latency for reads and writes
    - Default consistency level
    - Good availability during failures
- **When to use**:
    - User profile data
    - Shopping cart scenarios
    - Any user-centric application
    - Most common choice for microservices

```csharp
// Default consistency level
CosmosClientOptions options = new CosmosClientOptions
{
    ConsistencyLevel = ConsistencyLevel.Session
};
```

#### 4. **Consistent Prefix Consistency**

- **Guarantee**: Reads never see out-of-order writes
- **How it works**:
    - Guarantees that updates are returned in the same order they were applied
    - If writes happen as A, B, C, you'll never see C without seeing A and B
- **Performance Impact**:
    - Lower latency than Session
    - Lower RU consumption
    - Better availability during failures
- **When to use**:
    - Social media feeds
    - Status updates or timelines
    - Order tracking across multiple steps

```csharp
CosmosClientOptions options = new CosmosClientOptions
{
    ConsistencyLevel = ConsistencyLevel.ConsistentPrefix
};
```

#### 5. **Eventual Consistency**

- **Guarantee**: Reads eventually reflect all writes (with no guarantees on order)
- **How it works**:
    - Provides weakest consistency but strongest availability
    - No ordering guarantees; might see data out of order
- **Performance Impact**:
    - Lowest latency for reads and writes
    - Highest throughput
    - Lowest RU consumption
    - Highest availability during regional failures
    - Most relaxed consistency level
- **When to use**:
    - Non-critical data
    - Count statistics (views, likes)
    - Catalog data that doesn't change frequently
    - Highly distributed global applications prioritizing performance

```csharp
CosmosClientOptions options = new CosmosClientOptions
{
    ConsistencyLevel = ConsistencyLevel.Eventual
};
```

### 📊 Consistency Level Comparison

|Consistency Level|Availability|Latency|Throughput|RU Cost|Use Cases|
|---|---|---|---|---|---|
|Strong|Lowest|Highest|Lowest|~2x|Financial transactions, inventory|
|Bounded Staleness|Medium-High|Medium-High|Medium|~1.5x|Near-real-time collaborative apps|
|Session|High|Medium|Medium-High|~1x|User profiles, shopping carts|
|Consistent Prefix|Very High|Low|High|~1x|Social media feeds, status updates|
|Eventual|Highest|Lowest|Highest|~1x|View counts, non-critical data|

### 🎯 Exam Scenario Tips

The exam often presents scenarios and asks you to identify the most appropriate consistency level. Here's how to approach these questions:

1. **Look for availability and performance requirements**:
    
    - If maximum availability is required: Lean toward Eventual or Consistent Prefix
    - If low latency is critical: Avoid Strong consistency
2. **Identify data characteristics**:
    
    - Financial/transactional data: Strong consistency
    - User-specific data with session requirements: Session consistency
    - Globally distributed with performance priority: Eventual consistency
3. **Watch for these key phrases**:
    
    - "Always see latest data" → Strong
    - "Read your own writes" → At least Session
    - "Ordered sequence of updates" → At least Consistent Prefix
    - "Maximum availability" → Eventual
    - "Acceptable lag of X seconds" → Bounded Staleness
4. **Multi-region scenarios**:
    
    - Multi-region writes with strong consistency are NOT supported
    - For global distribution with writes in multiple regions, you must use a level weaker than Strong

```csharp
// Setting account-level default consistency
CosmosClientOptions options = new CosmosClientOptions
{
    ConsistencyLevel = ConsistencyLevel.Session
};
CosmosClient client = new CosmosClient(connectionString, options);

// Overriding consistency for a specific read operation
ItemRequestOptions requestOptions = new ItemRequestOptions
{
    ConsistencyLevel = ConsistencyLevel.Strong
};
ItemResponse<MyItem> response = await container.ReadItemAsync<MyItem>(
    id: "1",
    partitionKey: new PartitionKey("value"),
    requestOptions: requestOptions
);
```

> ⚠️ **Exam Gotcha**: Remember that you can override the account's default consistency level at the request level, but you can only request a stronger consistency than the account's default. For example, if the account is set to Eventual, you can request Session for a specific operation, but if the account is set to Strong, you cannot request Eventual for a specific operation.

> 💡 **Exam Tip**: The AZ-204 exam frequently asks about consistency levels in Cosmos DB. Be prepared to select the appropriate level based on business requirements like global distribution, read performance, high availability, and data accuracy needs.

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

## 🧮 Server-Side Programming

Server-side programming in Cosmos DB includes stored procedures, triggers, and user-defined functions (UDFs). These are written in JavaScript and executed directly within the database engine.

### Stored Procedures

Stored procedures are JavaScript functions registered per container that can perform CRUD operations on items within the same partition key.

**Key Features:**

- Atomic transactions (all-or-nothing execution)
- Business logic execution at the database tier
- Reduced network round trips for complex operations

```javascript
// Sample stored procedure that creates an item if it doesn't exist
function createOrUpdateItem(itemToCreate) {
    var context = getContext();
    var container = context.getCollection();
    var response = context.getResponse();
    
    // Query for the item
    var query = { query: "SELECT * FROM c WHERE c.id = @id", parameters: [{ name: "@id", value: itemToCreate.id }] };
    
    container.queryDocuments(container.getSelfLink(), query, {}, function(err, results) {
        if (err) throw err;
        
        if (results.length > 0) {
            // Item exists, replace it
            container.replaceDocument(results[0]._self, itemToCreate, function(err) {
                if (err) throw err;
                response.setBody({ status: "Updated existing item" });
            });
        } else {
            // Item doesn't exist, create it
            container.createDocument(container.getSelfLink(), itemToCreate, function(err) {
                if (err) throw err;
                response.setBody({ status: "Created new item" });
            });
        }
    });
}
```

```csharp
// C# code to register and execute a stored procedure
Scripts scripts = container.Scripts;

// Register the stored procedure
StoredProcedureResponse sprocResponse = await scripts.CreateStoredProcedureAsync(
    new StoredProcedureProperties
    {
        Id = "createOrUpdateItem",
        Body = File.ReadAllText("createOrUpdateItem.js")
    });

// Execute the stored procedure
dynamic itemToCreate = new { id = "1", name = "Test Item", category = "Test" };
StoredProcedureExecuteResponse<dynamic> executeResponse = await scripts.ExecuteStoredProcedureAsync<dynamic>(
    "createOrUpdateItem",
    new PartitionKey("Test"),
    new[] { itemToCreate }
);
```

> ⚠️ **Exam Gotcha**: Remember that stored procedures execute within a single logical partition. You cannot perform cross-partition operations within a stored procedure.

### Triggers

Cosmos DB supports two types of triggers:

#### 1. Pre-Triggers

- Executed **before** an item is created, updated, or deleted
- Can modify the request or validate data
- Cannot be used with stored procedures (only with direct operations)

```javascript
// Pre-trigger to validate and modify an item before creation
function validateAndModify() {
    var context = getContext();
    var request = context.getRequest();
    
    // Get the item to be created
    var itemToCreate = request.getBody();
    
    // Validate fields
    if (!itemToCreate.name) {
        throw new Error("Name is required");
    }
    
    // Add timestamp field
    itemToCreate.timestamp = new Date().toISOString();
    
    // Update the request with the modified document
    request.setBody(itemToCreate);
}
```

```csharp
// C# code to register a pre-trigger
Scripts scripts = container.Scripts;
TriggerResponse triggerResponse = await scripts.CreateTriggerAsync(
    new TriggerProperties
    {
        Id = "validateAndModify",
        Body = File.ReadAllText("validateAndModify.js"),
        TriggerType = TriggerType.Pre,
        TriggerOperation = TriggerOperation.Create
    });

// Execute operation with pre-trigger
ItemRequestOptions options = new ItemRequestOptions
{
    PreTriggerInclude = new List<string> { "validateAndModify" }
};

await container.CreateItemAsync(
    new { id = "1", category = "Test" },
    new PartitionKey("Test"),
    options
);
```

#### 2. Post-Triggers

- Executed **after** an item is created, updated, or deleted
- Cannot modify the response to the original operation
- Can perform additional operations like auditing or notifications
- Cannot be used with stored procedures (only with direct operations)

```javascript
// Post-trigger to create an audit record after item creation
function createAuditRecord() {
    var context = getContext();
    var container = context.getCollection();
    var request = context.getRequest();
    var response = context.getResponse();
    
    // Get the created item from the response
    var createdItem = response.getBody();
    
    // Create an audit record
    var auditRecord = {
        id: "audit_" + createdItem.id,
        operation: "CREATE",
        itemId: createdItem.id,
        timestamp: new Date().toISOString()
    };
    
    // Create the audit item
    container.createDocument(container.getSelfLink(), auditRecord, function(err) {
        if (err) throw err;
    });
}
```

```csharp
// C# code to register a post-trigger
Scripts scripts = container.Scripts;
TriggerResponse triggerResponse = await scripts.CreateTriggerAsync(
    new TriggerProperties
    {
        Id = "createAuditRecord",
        Body = File.ReadAllText("createAuditRecord.js"),
        TriggerType = TriggerType.Post,
        TriggerOperation = TriggerOperation.Create
    });

// Execute operation with post-trigger
ItemRequestOptions options = new ItemRequestOptions
{
    PostTriggerInclude = new List<string> { "createAuditRecord" }
};

await container.CreateItemAsync(
    new { id = "1", category = "Test" },
    new PartitionKey("Test"),
    options
);
```

### User-Defined Functions (UDFs)

- Used within queries for custom data processing
- Can be used in the SELECT or WHERE clauses of a query
- Extend the SQL query language capabilities

```javascript
// UDF to calculate age from birth date
function getAge(birthDate) {
    var today = new Date();
    var birthDateObj = new Date(birthDate);
    var age = today.getFullYear() - birthDateObj.getFullYear();
    var monthDiff = today.getMonth() - birthDateObj.getMonth();
    
    if (monthDiff < 0 || (monthDiff === 0 && today.getDate() < birthDateObj.getDate())) {
        age--;
    }
    
    return age;
}
```

```csharp
// C# code to register a UDF
Scripts scripts = container.Scripts;
UserDefinedFunctionResponse udfResponse = await scripts.CreateUserDefinedFunctionAsync(
    new UserDefinedFunctionProperties
    {
        Id = "getAge",
        Body = File.ReadAllText("getAge.js")
    });

// Query using the UDF
QueryDefinition query = new QueryDefinition(
    "SELECT c.id, c.name, udf.getAge(c.birthDate) AS age FROM c WHERE udf.getAge(c.birthDate) > 18"
);

FeedIterator<dynamic> resultSet = container.GetItemQueryIterator<dynamic>(query);
while (resultSet.HasMoreResults)
{
    foreach (var item in await resultSet.ReadNextAsync())
    {
        Console.WriteLine($"ID: {item.id}, Name: {item.name}, Age: {item.age}");
    }
}
```

### Server-Side Programming Limitations

1. **Execution Time Limit**: Maximum execution time of 5 seconds
2. **JavaScript Version**: ECMAScript 5.1 with some limitations
3. **Resource Limitations**:
    - Memory: Limited to a few MBs
    - CPU: Limited processing power
4. **No External Access**: Cannot access external systems or make HTTP calls
5. **Partition Key Restriction**: Stored procedures and triggers execute within a single partition

> 💡 **Exam Tip**: The exam often asks about which server-side programming feature to use for specific scenarios:
> 
> - Need atomic transactions? → Stored Procedures
> - Need to validate data before save? → Pre-Triggers
> - Need to extend query capabilities? → UDFs
> - Need to log changes after save? → Post-Triggers

> ⚠️ **Exam Gotcha**: The exam may present scenarios where server-side programming appears to be the solution but isn't due to the limitations mentioned above (like cross-partition operations or long-running processes).

---

## 🔢 Cosmos DB Enumerations

Understanding the key enumerations used in the Cosmos DB SDK is crucial for the AZ-204 exam. Here are the most important enums with their values, usage examples, and exam considerations:

### ConsistencyLevel Enum

Defines the consistency levels available in Cosmos DB.

```csharp
namespace Microsoft.Azure.Cosmos
{
    public enum ConsistencyLevel
    {
        Strong = 0,
        BoundedStaleness = 1,
        Session = 2,
        Eventual = 3,
        ConsistentPrefix = 4
    }
}
```

**Usage Example:**

```csharp
CosmosClientOptions options = new CosmosClientOptions
{
    ConsistencyLevel = ConsistencyLevel.Session
};
CosmosClient client = new CosmosClient(connectionString, options);
```

> 💡 **Exam Tip**: You may be asked to identify the numeric value of a specific consistency level or select the appropriate enum value for a given scenario.

### IndexingMode Enum

Defines the indexing modes for Cosmos DB containers.

```csharp
namespace Microsoft.Azure.Cosmos
{
    public enum IndexingMode
    {
        Consistent = 0,  // Index is updated synchronously with writes
        Lazy = 1,        // Index is updated asynchronously (deprecated)
        None = 2         // No indexing is performed
    }
}
```

**Usage Example:**

```csharp
ContainerProperties properties = new ContainerProperties("myContainer", "/partitionKey");
properties.IndexingPolicy = new IndexingPolicy
{
    IndexingMode = IndexingMode.Consistent,
    Automatic = true
};
```

> ⚠️ **Exam Gotcha**: `IndexingMode.Lazy` is deprecated but might still appear in exam questions. Know that `Consistent` is now the recommended mode.

### ConnectionMode Enum

Determines how the SDK connects to the Cosmos DB service.

```csharp
namespace Microsoft.Azure.Cosmos
{
    public enum ConnectionMode
    {
        Direct = 0,     // Direct connection to backend nodes
        Gateway = 1     // Connection through the gateway service
    }
}
```

**Usage Example:**

```csharp
CosmosClientOptions options = new CosmosClientOptions
{
    ConnectionMode = ConnectionMode.Direct,
    ApplicationRegion = Regions.WestUS2
};
CosmosClient client = new CosmosClient(connectionString, options);
```

> 💡 **Exam Tip**: `Direct` mode offers better performance but has some firewall and networking limitations. `Gateway` mode works in more restricted network environments.

### TriggerType Enum

Specifies the type of trigger.

```csharp
namespace Microsoft.Azure.Cosmos
{
    public enum TriggerType
    {
        Pre = 0,    // Executes before the operation
        Post = 1    // Executes after the operation
    }
}
```

**Usage Example:**

```csharp
TriggerProperties properties = new TriggerProperties
{
    Id = "validateDocument",
    Body = "function() { /* trigger code */ }",
    TriggerType = TriggerType.Pre,
    TriggerOperation = TriggerOperation.Create
};
```

### TriggerOperation Enum

Specifies the database operation that activates the trigger.

```csharp
namespace Microsoft.Azure.Cosmos
{
    public enum TriggerOperation
    {
        All = 0,     // All operations
        Create = 1,  // Create operations
        Update = 2,  // Update operations
        Delete = 3,  // Delete operations
        Replace = 4  // Replace operations
    }
}
```

**Usage Example:**

```csharp
TriggerProperties properties = new TriggerProperties
{
    Id = "validateDocument",
    Body = "function() { /* trigger code */ }",
    TriggerType = TriggerType.Pre,
    TriggerOperation = TriggerOperation.Create | TriggerOperation.Replace
};
```

> ⚠️ **Exam Gotcha**: You can combine operations using the bitwise OR operator, but this is not always obvious in multiple-choice questions.

### PartitionKeyDefinitionVersion Enum

Specifies the version of the partition key definition.

```csharp
namespace Microsoft.Azure.Cosmos
{
    public enum PartitionKeyDefinitionVersion
    {
        V1 = 1,  // Original version
        V2 = 2   // Enhanced version with better support for large partition keys
    }
}
```

**Usage Example:**

```csharp
ContainerProperties properties = new ContainerProperties("myContainer", "/partitionKey");
properties.PartitionKeyDefinitionVersion = PartitionKeyDefinitionVersion.V2;
```

> 💡 **Exam Tip**: Version 2 supports larger partition keys (up to 2KB) and is recommended for new containers.

### IndexingDirective Enum

Controls whether indexing happens for a specific operation.

```csharp
namespace Microsoft.Azure.Cosmos
{
    public enum IndexingDirective
    {
        Default = 0,  // Use the container's indexing policy
        Include = 1,  // Always index this document
        Exclude = 2   // Do not index this document
    }
}
```

**Usage Example:**

```csharp
ItemRequestOptions options = new ItemRequestOptions
{
    IndexingDirective = IndexingDirective.Exclude
};

await container.CreateItemAsync(
    item: new { id = "1", largeData = "..." },
    partitionKey: new PartitionKey("value"),
    requestOptions: options
);
```

> 💡 **Exam Tip**: Use `Exclude` for large documents that don't need to be queried to save RUs.

### RequestVerb Enum

Used internally in the Cosmos DB SDK to define the HTTP verb for a request.

```csharp
namespace Microsoft.Azure.Cosmos
{
    internal enum RequestVerb
    {
        Get = 0,
        Post = 1,
        Put = 2,
        Delete = 3,
        Head = 4,
        Options = 5,
        Patch = 6
    }
}
```

> ⚠️ **Exam Gotcha**: This enum is internal and not directly used in application code, but understanding the REST operations might be tested.

### OfferType Enum (Legacy)

Defines the throughput levels in legacy collections.

```csharp
namespace Microsoft.Azure.Cosmos
{
    public enum OfferType
    {
        Invalid = -1,
        Standard = 0,
        S1 = 1,
        S2 = 2,
        S3 = 3
    }
}
```

> ⚠️ **Exam Gotcha**: This enum is legacy and replaced by explicit RU/s provisioning, but older questions might still reference it.

### StatusCode Enum

Represents HTTP status codes returned by the Cosmos DB service.

```csharp
namespace Microsoft.Azure.Cosmos
{
    public enum StatusCode
    {
        OK = 200,
        Created = 201,
        NoContent = 204,
        BadRequest = 400,
        Unauthorized = 401,
        Forbidden = 403,
        NotFound = 404,
        RequestTimeout = 408,
        Conflict = 409,
        PreconditionFailed = 412,
        RequestEntityTooLarge = 413,
        TooManyRequests = 429,
        RetryWith = 449,
        InternalServerError = 500,
        ServiceUnavailable = 503
    }
}
```

**Usage Example:**

```csharp
try
{
    ItemResponse<MyItem> response = await container.ReadItemAsync<MyItem>("1", new PartitionKey("value"));
}
catch (CosmosException ex) when (ex.StatusCode == HttpStatusCode.NotFound)
{
    Console.WriteLine("Item not found");
}
catch (CosmosException ex) when (ex.StatusCode == HttpStatusCode.TooManyRequests)
{
    Console.WriteLine("Rate limited. Retry after: " + ex.RetryAfter);
}
```

> 💡 **Exam Tip**: Pay special attention to status code 429 (TooManyRequests), which indicates you've exceeded your provisioned RU/s, and how to properly handle it.

### Enum Values in JSON

Sometimes, the exam may show enum values used in JSON configuration:

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

> ⚠️ **Exam Gotcha**: In JSON, enum values are represented as strings (e.g., "consistent" instead of IndexingMode.Consistent).

### Exam Tips for Enumerations

1. **Memorize Default Values**: Know the default values for important enums (e.g., ConsistencyLevel.Session is the default consistency level)
    
2. **Focus on Scenario Selection**: The exam often gives scenarios and asks which enum value to use
    
3. **Understand Cross-Connections**: Know which enums are related to each other (e.g., TriggerType and TriggerOperation work together)
    
4. **Error Handling**: Understand how to properly handle specific StatusCode values in catch blocks
    
5. **Performance Implications**: Understand how different enum values affect performance and cost (e.g., consistency levels impact RU consumption)
    

> 🔑 **Key Exam Focus**: The most frequently tested enums are ConsistencyLevel, IndexingMode, and ConnectionMode.

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

1. **Know Your RUs**:
    
    - Understand how different operations consume RUs and how to optimize costs
    - Remember that queries without partition key filters are more expensive
    - Strong consistency costs roughly 2x the RUs of eventual consistency
2. **Consistency Tradeoffs**:
    
    - Memorize the five consistency levels and their tradeoffs
    - Session consistency is the default and most commonly used level
    - Strong consistency is not available with multi-region writes
    - Bounded staleness lets you configure staleness by operations or time
3. **Partitioning Strategy**:
    
    - Be able to choose appropriate partition keys for different scenarios
    - Good partition keys have high cardinality and distribute data evenly
    - Poor partition keys create "hot partitions" (skewed traffic patterns)
    - Once set, a container's partition key cannot be changed without data migration
4. **API Selection**:
    
    - Know when to use each API model based on requirements
    - SQL (Core) API is the most feature-rich and most commonly tested
    - MongoDB API is best for migrating from existing MongoDB applications
    - Gremlin API is used for graph data and relationship-heavy use cases
5. **Cost Optimization**:
    
    - Understand strategies to reduce costs while maintaining performance
    - Autoscale is good for variable workloads (10-100% of max RUs)
    - Serverless is best for development or intermittent workloads
    - Standard (manual) throughput is most cost-effective for predictable workloads
6. **SDK Patterns**:
    
    - Know when to use transactional batches vs. bulk operations
    - Understand error handling, especially for 429 (TooManyRequests) errors
    - Implement retry logic with exponential backoff for rate limiting
7. **Server-Side Programming**:
    
    - Understand when to use stored procedures, triggers, and UDFs
    - Stored procedures provide atomic transactions within a partition
    - Pre-triggers can validate data before it's written
    - UDFs extend SQL query capabilities for custom logic
8. **Multi-Region Strategy**:
    
    - Know the difference between single-region, multi-region read, and multi-region write configurations
    - Understand failover configuration (automatic vs. manual)
    - Multi-region writes require a consistency level weaker than Strong

### Common Gotchas

1. **Cross-Partition Queries**:
    
    - These can be expensive and slow; know how to avoid them
    - Filter by partition key when possible
    - Use the container.ReadItem API for point reads instead of queries when possible
2. **Changing Partition Keys**:
    
    - You can't change a container's partition key after creation without migrating data
    - You must create a new container and migrate the data if you need to change the partition key
3. **Consistency vs Performance**:
    
    - Strong consistency has performance and cost implications
    - Cannot use Strong consistency with multi-region writes
    - Session consistency only guarantees consistency within the same client session
4. **RU Estimation**:
    
    - The exam may test your ability to estimate RU needs for specific workloads
    - Know the baseline: 1 RU = resources for a 1KB point read
    - Writes typically cost 5-7x more than reads
    - Cross-partition queries cost more than single-partition queries
5. **SDK Versions**:
    
    - Be aware of differences between older SDKs (V2) and newer ones (V3)
    - V3 SDK uses more modern patterns (async/await, container/item model)
    - V2 used DocumentClient while V3 uses CosmosClient
6. **TTL Settings**:
    
    - Container-level TTL setting doesn't automatically delete items without an explicit TTL property
    - Individual items need a TTL value set to be automatically deleted
    - Setting container TTL to -1 enables TTL but doesn't delete anything without item-level TTL
7. **Indexing Policy**:
    
    - Changing indexing policy can temporarily affect write performance
    - Excluding paths from indexing saves RUs but prevents queries on those paths
    - Composite indexes are required for ORDER BY clauses with multiple properties
8. **Request Timeouts**:
    
    - Default SDK timeout might be too short for large operations
    - Configure appropriate timeouts for bulk operations
9. **Connection Modes**:
    
    - Direct mode is faster but requires more ports and specific firewall rules
    - Gateway mode works through HTTPS port 443 for restricted networks
    - Direct mode TCP uses port 10250-10256 plus 443 for the service bus
10. **Resource Governance**:
    
    - Maximum document size: 2MB
    - Maximum request size: 2MB
    - Maximum execution time for stored procedures: 5 seconds

### Scenario-Based Tips

The AZ-204 exam often presents scenario-based questions asking you to choose the right approach. Here's how to identify what the question is really testing:

1. **Cost Optimization Scenarios**:
    
    - Look for phrases like "minimize cost," "most cost-effective," or "reduce expenses"
    - Consider provisioning models (standard, autoscale, serverless)
    - Think about indexing and partition key optimizations
2. **Performance Optimization Scenarios**:
    
    - Watch for "minimize latency," "fastest response time," or "high throughput"
    - Consider direct connection mode, partition key selection
    - Look for read-heavy vs. write-heavy workload hints
3. **Availability Scenarios**:
    
    - Key phrases: "highest availability," "minimize downtime," "geographic redundancy"
    - Consider multi-region deployments, automatic failover
    - Consider consistency level tradeoffs (lower consistency = higher availability)
4. **Security Scenarios**:
    
    - Watch for "least privilege," "secure access," or "role-based access"
    - Look for hints about Azure AD integration vs. master keys
    - Consider resource tokens for limited-scope access
5. **Data Modeling Scenarios**:
    
    - Look for hints about relationship types (one-to-many, many-to-many)
    - Consider document structure and embedding vs. referencing
    - Watch for partition key selection hints

> 💡 **Exam Strategy Tip**: For each Cosmos DB question, first identify which aspect is being tested (performance, cost, availability, security, or data modeling) and then apply the appropriate best practices for that aspect.

### Question Keywords to Watch For

- **"Minimize cost"** → Look for opportunities to reduce RU consumption (indexing, querying, consistency)
- **"Atomic transactions"** → Stored procedures within a partition
- **"Cross-partition operations"** → Cannot use stored procedures; need client-side transactions
- **"Global distribution"** → Multi-region configuration with appropriate consistency level
- **"High write throughput"** → Partition key selection, bulk operations
- **"Point reads"** → Direct item reads vs. queries
- **"Lock-free optimistic concurrency"** → ETag-based concurrency control
- **"Real-time analytics"** → Change feed processing

Remember: the AZ-204 exam is testing your practical knowledge of implementing and optimizing real-world solutions with Cosmos DB, not just theoretical understanding of features.

---

## ❓ Practice Questions

Below are 20 practice questions covering various aspects of Azure Cosmos DB that might appear on the AZ-204 exam. Each question includes the correct answer and an explanation to help solidify your understanding.

### 1. Which consistency level provides the strongest guarantees for data consistency but may impact performance and availability?

A) Eventual  
B) Session  
C) Strong  
D) Consistent Prefix

**Answer: C) Strong**

_Explanation: Strong consistency provides linearizability guarantees, ensuring all reads see the most recent write. It has the highest latency and RU consumption (approximately 2x more than other levels) and cannot be used with multi-region writes._

### 2. A company needs to store product data where each product belongs to a category. There are 5 categories and thousands of products distributed evenly across these categories. What would be a poor choice for a partition key?

A) Product ID  
B) Category  
C) Combination of Category and Product Name  
D) Timestamp

**Answer: B) Category**

_Explanation: Using Category as the partition key would create only 5 logical partitions (hot partitions), which would not distribute the workload effectively across physical partitions. This could lead to rate limiting and performance issues. A high-cardinality value like Product ID would be a better choice._

### 3. Which of these operations consumes the most RUs for equally sized documents?

A) Point read operation  
B) Write operation  
C) Query with a filter on the partition key  
D) Query without a filter on the partition key

**Answer: D) Query without a filter on the partition key**

_Explanation: Cross-partition queries (those without a partition key filter) consume the most RUs because they need to scan multiple physical partitions. Write operations typically consume 5-7x more RUs than point reads, but cross-partition queries can be even more expensive depending on the number of partitions._

### 4. Which API would you recommend for a new application with document data that needs SQL-like queries?

A) MongoDB API  
B) Cassandra API  
C) Gremlin API  
D) SQL API

**Answer: D) SQL API**

_Explanation: The SQL (Core) API is the native and most feature-rich API for new applications in Cosmos DB. It supports SQL-like queries on JSON documents and provides access to all Cosmos DB features._

### 5. A company needs to implement atomic transactions within a specific logical partition in Cosmos DB. Which feature should they use?

A) Transactional Batch  
B) Bulk Operations  
C) Stored Procedures  
D) User-Defined Functions

**Answer: C) Stored Procedures**

_Explanation: Stored procedures provide atomic transaction guarantees within a single logical partition. They execute on the server and can perform multiple operations that either all succeed or all fail together. Transactional Batch is also correct for newer SDK versions, but stored procedures provide more flexibility for complex logic._

### 6. What is the maximum size for a single document in Cosmos DB?

A) 1 MB  
B) 2 MB  
C) 10 MB  
D) Unlimited

**Answer: B) 2 MB**

_Explanation: The maximum size for a single document in Cosmos DB is 2 MB. This includes all property values, metadata, and indexing overhead._

### 7. A developer needs to validate and modify documents before they're inserted into Cosmos DB. Which feature should they use?

A) Post-Trigger  
B) Pre-Trigger  
C) User-Defined Function  
D) Stored Procedure

**Answer: B) Pre-Trigger**

_Explanation: Pre-triggers execute before a document is created, updated, or deleted. They can validate the document content and even modify it before it's committed to the database._

### 8. Which of the following is NOT a valid provisioning mode for Cosmos DB throughput?

A) Standard (Manual)  
B) Autoscale  
C) Serverless  
D) Dynamic

**Answer: D) Dynamic**

_Explanation: Cosmos DB offers three throughput provisioning modes: Standard (Manual), Autoscale, and Serverless. "Dynamic" is not a valid provisioning mode._

### 9. Which consistency level would you recommend for a global application that requires maximum availability and performance but can tolerate some degree of staleness?

A) Strong  
B) Bounded Staleness  
C) Session  
D) Eventual

**Answer: D) Eventual**

_Explanation: Eventual consistency provides the highest availability and lowest latency of all consistency levels, making it ideal for scenarios where maximum availability and performance are critical and some staleness is acceptable._

### 10. A company wants to implement optimistic concurrency control in their Cosmos DB application. Which feature should they use?

A) ETags  
B) Lock statements  
C) Isolation levels  
D) Snapshot isolation

**Answer: A) ETags**

_Explanation: Cosmos DB supports optimistic concurrency control using ETags. Each item has an ETag that changes when the item is updated. By including the ETag in update operations with the `IfMatchEtag` option, you can ensure that updates only succeed if the item hasn't been modified since it was last read._

### 11. Which index type should be used when you need to execute ORDER BY queries on multiple properties in Cosmos DB?

A) Range index  
B) Spatial index  
C) Composite index  
D) Hash index

**Answer: C) Composite index**

_Explanation: Composite indexes are required for efficient execution of ORDER BY queries that involve multiple properties. They allow the database to pre-sort the data according to the specified combination of properties._

### 12. When using the change feed in Cosmos DB, which types of operations can be captured?

A) Inserts and updates only  
B) Inserts, updates, and deletes  
C) Inserts only  
D) All operations including stored procedure execution

**Answer: A) Inserts and updates only**

_Explanation: The Cosmos DB change feed captures inserts and updates to items but does not natively capture deletes. To track deletes, you typically need to implement soft deletes (using a flag or TTL) or use the change feed processor library's built-in support for deletes (in newer versions)._

### 13. A company has a multi-region Cosmos DB account with write regions in East US and West Europe. Which consistency level can they NOT use?

A) Strong  
B) Bounded Staleness  
C) Session  
D) Eventual

**Answer: A) Strong**

_Explanation: Strong consistency cannot be used with multi-region write configurations. It requires that all writes be replicated synchronously to all regions before being committed, which would result in unacceptable latency for multi-region writes._

### 14. What happens to request units (RUs) that are not consumed within their provisioned time period?

A) They accumulate for future use  
B) They expire and are lost  
C) They are automatically refunded as credits  
D) They convert to storage capacity

**Answer: B) They expire and are lost**

_Explanation: RUs are a measure of throughput per second. If you don't use your provisioned RUs in a given second, they do not accumulate or carry over to the next second. Unused capacity is effectively lost._

### 15. Which of the following is true about the Cosmos DB emulator?

A) It supports all Cosmos DB APIs except MongoDB  
B) It supports only the SQL API  
C) It can run in the cloud but not locally  
D) It supports all Cosmos DB APIs

**Answer: D) It supports all Cosmos DB APIs**

_Explanation: The Azure Cosmos DB Emulator provides a local environment that emulates the Azure Cosmos DB service for development purposes. It supports all Cosmos DB APIs: SQL, MongoDB, Cassandra, Gremlin, and Table._

### 16. A developer needs to execute a query that returns all documents where a property value is within a geographic area. Which index type should they ensure is configured?

A) Range index  
B) Spatial index  
C) Composite index  
D) Text index

**Answer: B) Spatial index**

_Explanation: Spatial indexes support geospatial queries in Cosmos DB, allowing you to efficiently query for points, lines, and polygons within geographic areas. Range indexes would not efficiently support these types of queries._

### 17. What is the main advantage of using the SDK's `ConnectionMode.Direct` setting instead of `ConnectionMode.Gateway`?

A) Better security with encrypted connections  
B) Lower latency and higher throughput  
C) Support for more regions  
D) Simplified firewall configuration

**Answer: B) Lower latency and higher throughput**

_Explanation: Direct mode provides lower latency and higher throughput by connecting directly to the Cosmos DB backends using the proprietary protocol over TCP. Gateway mode routes requests through the gateway service using HTTPS, which adds some overhead but works through more restrictive firewalls._

### 18. Which of the following C# code snippets correctly implements retry logic for handling rate limiting (429) errors in Cosmos DB?

A)

```csharp
try {
    await container.CreateItemAsync(item);
} catch (Exception ex) {
    Thread.Sleep(1000);
    await container.CreateItemAsync(item);
}
```

B)

```csharp
try {
    await container.CreateItemAsync(item);
} catch (CosmosException ex) when (ex.StatusCode == HttpStatusCode.TooManyRequests) {
    await Task.Delay(ex.RetryAfter ?? TimeSpan.FromSeconds(1));
    await container.CreateItemAsync(item);
}
```

C)

```csharp
await container.CreateItemAsync(item, new PartitionKey(item.Id));
```

D)

```csharp
if (!container.CreateItemAsync(item).IsCompleted)
    await Task.Delay(1000);
    await container.CreateItemAsync(item);
```

**Answer: B)**

_Explanation: This code correctly catches the CosmosException with a status code of 429 (TooManyRequests), retrieves the suggested retry-after duration from the exception, and waits for that duration before retrying. This is the recommended pattern for handling rate limiting in Cosmos DB._

### 19. A company wants to use Cosmos DB but needs to enable automatic expiration of documents after a certain period. Which feature should they use?

A) Soft delete  
B) Time-to-Live (TTL)  
C) Material Views  
D) Scheduled deletion

**Answer: B) Time-to-Live (TTL)**

_Explanation: Cosmos DB's Time-to-Live (TTL) feature allows you to automatically delete items after a specified period. You can set TTL at the container level and then specify TTL values for individual items, after which they will be automatically deleted._

### 20. A developer needs to ensure that all queries against a Cosmos DB container are executed even when the client's rate limits are exceeded, by having the service automatically retry with backoff. Which client configuration option should they use?

A) EnableEndpointDiscovery  
B) EnableTcpConnectionEndpointRediscovery  
C) EnableRetryOnThrottling  
D) MaxRetryAttemptsOnRateLimitedRequests and MaxRetryWaitTimeOnRateLimitedRequests

**Answer: D) MaxRetryAttemptsOnRateLimitedRequests and MaxRetryWaitTimeOnRateLimitedRequests**

_Explanation: The `MaxRetryAttemptsOnRateLimitedRequests` and `MaxRetryWaitTimeOnRateLimitedRequests` properties control the automatic retry behavior for rate-limited requests. They define how many times the SDK will retry and the maximum wait time between retries._

```csharp
CosmosClientOptions options = new CosmosClientOptions
{
    MaxRetryAttemptsOnRateLimitedRequests = 9,
    MaxRetryWaitTimeOnRateLimitedRequests = TimeSpan.FromSeconds(30)
};
CosmosClient client = new CosmosClient(connectionString, options);
```

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
