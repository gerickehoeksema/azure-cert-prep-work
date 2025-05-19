## 🎯 Introduction

This guide covers the Azure Storage components needed for the AZ-204 (Developing Solutions for Microsoft Azure) certification exam. Azure Storage is a fundamental service that appears frequently in the exam.

## 📋 Exam Topic Breakdown

|Topic Area|Approximate Weight|
|---|---|
|Blob Storage|35-40%|
|Table Storage|15-20%|
|Queue Storage|15-20%|
|File Storage|10-15%|
|Storage Security|15-20%|

---

## 🧱 Core Concepts

### Storage Account Types

|Account Type|Performance Tiers|Access Tiers|Replication Options|
|---|---|---|---|
|**Standard General-purpose v2**|Standard|Hot, Cool, Archive|LRS, ZRS, GRS, RA-GRS, GZRS, RA-GZRS|
|**Premium Block Blobs**|Premium|Hot|LRS, ZRS|
|**Premium File Shares**|Premium|N/A|LRS, ZRS|
|**Premium Page Blobs**|Premium|N/A|LRS|

> ⚠️ **Exam Tip**: Know the differences between account types and when to use them. Premium storage uses SSDs and offers higher performance but at a higher cost.

### Replication Options

- **LRS (Locally Redundant Storage)**: Three copies within single data center
- **ZRS (Zone-Redundant Storage)**: Three copies across availability zones
- **GRS (Geo-Redundant Storage)**: Six copies (3 in primary region, 3 in secondary)
- **RA-GRS (Read-Access Geo-Redundant Storage)**: GRS with read access to secondary
- **GZRS (Geo-Zone-Redundant Storage)**: ZRS in primary region + 3 copies in secondary
- **RA-GZRS (Read-Access Geo-Zone-Redundant Storage)**: GZRS with read access to secondary

> 💡 **Exam Tip**: You need to understand the RPO (Recovery Point Objective) and RTO (Recovery Time Objective) for each replication type and their cost implications.

---

## 💾 Blob Storage

### Blob Types

- **Block Blobs**: Optimized for uploading large amounts of data (videos, backups)
- **Page Blobs**: Optimized for random read/write operations (VHD/disk files)
- **Append Blobs**: Optimized for append operations (logging data)

### Blob Access Tiers

- **Hot**: Frequently accessed data, highest storage cost, lowest access cost
- **Cool**: Infrequently accessed, stored for at least 30 days
- **Archive**: Rarely accessed, stored for at least 180 days, highest retrieval costs

### Lifecycle Management

Configure rules to automatically:

- Move blobs to cooler tiers
- Delete blobs after specified time period

```csharp
// C# example: Create a blob lifecycle management policy
BlobServiceClient blobServiceClient = new BlobServiceClient(connectionString);
BlobServiceProperties properties = await blobServiceClient.GetPropertiesAsync();

// Create a policy with rules
ManagementPolicy policy = new ManagementPolicy
{
    Rules =
    {
        new ManagementPolicyRule
        {
            Name = "rule1",
            Definition = new ManagementPolicyDefinition
            {
                Actions = new ManagementPolicyAction
                {
                    BaseBlob = new ManagementPolicyBaseBlob
                    {
                        TierToCool = new DateAfterModification { DaysAfterModificationGreaterThan = 30 },
                        TierToArchive = new DateAfterModification { DaysAfterModificationGreaterThan = 90 },
                        Delete = new DateAfterModification { DaysAfterModificationGreaterThan = 365 }
                    }
                },
                Filters = new ManagementPolicyFilter
                {
                    PrefixMatch = { "container1/log" }
                }
            }
        }
    }
};

await blobServiceClient.SetManagementPolicyAsync(policy);
```

> ⚠️ **Gotcha**: Remember that changing tiers incurs charges, and retrieving from Archive tier has high latency (hours).

### Blob Client Libraries

```csharp
// Create a BlobServiceClient
BlobServiceClient blobServiceClient = new BlobServiceClient(connectionString);

// Get a container reference
BlobContainerClient containerClient = blobServiceClient.GetBlobContainerClient("mycontainer");

// Upload a blob
BlobClient blobClient = containerClient.GetBlobClient("sample-blob.txt");
await blobClient.UploadAsync(stream, true);
```

> 💡 **Exam Tip**: The exam often tests your knowledge of the Azure.Storage.Blobs namespace and the BlobServiceClient, BlobContainerClient, and BlobClient classes.

### Properties and Metadata

Azure Storage resources support both system properties and user-defined metadata.

#### System Properties

- **System properties**: Predefined properties maintained by the storage service
- Each storage resource (blobs, containers, files, etc.) has different system properties
- Examples for blobs: Content-Type, Content-Length, ETag, Last-Modified

```csharp
// Get blob properties
BlobProperties properties = await blobClient.GetPropertiesAsync();

// Access system properties
Console.WriteLine($"Content Type: {properties.ContentType}");
Console.WriteLine($"Content Length: {properties.ContentLength}");
Console.WriteLine($"Created On: {properties.CreatedOn}");
Console.WriteLine($"Last Modified: {properties.LastModified}");
Console.WriteLine($"ETag: {properties.ETag}");
```

#### User-defined Metadata

- **Metadata**: Name-value pairs you define and associate with a storage resource
- Metadata names must adhere to HTTP header naming conventions
- Each resource can have up to 8KB of metadata (total size of all name-value pairs)

```csharp
// Set metadata
IDictionary<string, string> metadata = new Dictionary<string, string>
{
    { "department", "HR" },
    { "project", "AnnualReport" },
    { "created-by", "john.smith" }
};

await blobClient.SetMetadataAsync(metadata);

// Get and read metadata
BlobProperties properties = await blobClient.GetPropertiesAsync();
foreach (var metadataItem in properties.Metadata)
{
    Console.WriteLine($"{metadataItem.Key}: {metadataItem.Value}");
}
```

> ⚠️ **Gotcha**: Metadata names are case-insensitive when set and retrieved, but preserved as they were when stored. Also, metadata cannot contain special characters like spaces - use only alphanumeric characters.

#### Using Metadata in Operations

- **Search and filter**: Metadata can be used in blob listing operations
- **Track usage**: Store custom information about when and how a resource is used
- **Authorization logic**: Implement custom authorization based on metadata values

```csharp
// List blobs with specific metadata
List<BlobItem> filteredBlobs = new List<BlobItem>();
await foreach (BlobItem blobItem in containerClient.GetBlobsAsync(BlobTraits.Metadata))
{
    if (blobItem.Metadata.TryGetValue("department", out string department) && 
        department == "HR")
    {
        filteredBlobs.Add(blobItem);
    }
}
```

> 💡 **Exam Tip**: Know how to use metadata for searching and organizing your data in storage services. The exam may test your ability to add, modify, and retrieve metadata.

---

## 📊 Table Storage

Azure Table Storage is a NoSQL datastore for semi-structured data.

### Key Concepts

- **Table**: Container for entities
- **Entity**: A set of properties (like a row)
- **Properties**: Name-value pairs (like columns)
- **Partition Key**: Determines which partition stores the entity
- **Row Key**: Uniquely identifies an entity within a partition

```csharp
// Create a table client
TableClient tableClient = new TableClient(connectionString, "customers");
await tableClient.CreateIfNotExistsAsync();

// Create an entity
var entity = new TableEntity("Smith", "John")
{
    ["Email"] = "john.smith@example.com",
    ["PhoneNumber"] = "123-456-7890"
};

// Add the entity to the table
await tableClient.AddEntityAsync(entity);
```

> ⚠️ **Gotcha**: Table Storage has significantly different query capabilities compared to SQL. No joins, limited WHERE clauses, and no stored procedures.

### Properties and Metadata in Table Storage

#### Entity Properties

Table Storage entities can have up to 255 properties including the system properties:

- **PartitionKey and RowKey**: Together form the primary key
- **Timestamp**: Last modified time (maintained by Azure)
- **ETag**: Used for optimistic concurrency

```csharp
// Create a custom entity class
public class CustomerEntity : ITableEntity
{
    public string PartitionKey { get; set; }
    public string RowKey { get; set; }
    public DateTimeOffset? Timestamp { get; set; }
    public ETag ETag { get; set; }
    
    // Custom properties
    public string Email { get; set; }
    public string PhoneNumber { get; set; }
    public bool IsActive { get; set; }
    
    // Required by ITableEntity interface
    public void ReadEntity(IDictionary<string, EntityProperty> properties, OperationContext operationContext)
    {
        // Logic to read entity
    }
    
    public IDictionary<string, EntityProperty> WriteEntity(OperationContext operationContext)
    {
        // Logic to write entity
        return null;
    }
}
```

> 💡 **Exam Tip**: You need to understand the difference between the older TableEntity class and the newer approaches using POCO (Plain Old CLR Objects) or dynamic objects for working with tables.

#### Property Types

Table Storage supports the following property types:

- **String**: The most flexible type
- **Boolean**: True/false values
- **DateTime**: Date and time values (stored as UTC)
- **Double**: 8-byte floating-point numbers
- **GUID**: Globally unique identifiers
- **Int32**: 32-bit integers
- **Int64**: 64-bit integers
- **Binary**: Binary data (up to 64 KB per entity)

> ⚠️ **Gotcha**: Unlike blob storage, Table Storage doesn't support user-defined metadata. All data must be stored as entity properties.

### Key Table Operations

- **CRUD Operations**: Create, Read, Update, Delete entities
- **Batch Operations**: Perform multiple operations in a single request
- **Queries**: Filter based on property values

> 💡 **Exam Tip**: Understand how to design effective partition keys and row keys to optimize query performance.

---

## 📨 Queue Storage

Azure Queue Storage is a service for storing large numbers of messages that can be accessed from anywhere via HTTP or HTTPS.

### Key Concepts

- **Queue**: Container for messages
- **Message**: Can be up to 64KB in size
- **TTL (Time-to-Live)**: How long a message remains in the queue before automatic deletion

```csharp
// Create a queue client
QueueClient queueClient = new QueueClient(connectionString, "myqueue");
await queueClient.CreateIfNotExistsAsync();

// Send a message
await queueClient.SendMessageAsync("Hello, World!");

// Receive and process a message
QueueMessage[] messages = await queueClient.ReceiveMessagesAsync();
foreach (QueueMessage message in messages)
{
    // Process the message
    Console.WriteLine($"Message: {message.MessageText}");
    
    // Delete the message
    await queueClient.DeleteMessageAsync(message.MessageId, message.PopReceipt);
}
```

> ⚠️ **Gotcha**: Queue messages have a maximum visibility timeout of 7 days. If not processed within this period, they become visible again.

### Properties and Metadata in Queue Storage

#### Queue Properties

Queue Storage supports system properties and user-defined metadata at the queue level.

```csharp
// Get queue properties
QueueProperties properties = await queueClient.GetPropertiesAsync();

Console.WriteLine($"Approximate Message Count: {properties.ApproximateMessagesCount}");

// Set queue metadata
IDictionary<string, string> metadata = new Dictionary<string, string>
{
    { "purpose", "order-processing" },
    { "environment", "production" },
    { "owner", "fulfillment-team" }
};

await queueClient.SetMetadataAsync(metadata);

// Get queue metadata
properties = await queueClient.GetPropertiesAsync();
foreach (var metadataItem in properties.Metadata)
{
    Console.WriteLine($"{metadataItem.Key}: {metadataItem.Value}");
}
```

#### Message Properties

Queue messages have their own set of system properties:

- **MessageId**: Uniquely identifies the message within the queue
- **InsertionTime**: Time when the message was added to the queue
- **ExpirationTime**: Time when the message will automatically be deleted
- **PopReceipt**: Required for operations like delete and update
- **DequeueCount**: Number of times the message has been retrieved

```csharp
// Examine message properties
QueueMessage[] messages = await queueClient.ReceiveMessagesAsync();
foreach (QueueMessage message in messages)
{
    Console.WriteLine($"Message ID: {message.MessageId}");
    Console.WriteLine($"Insertion Time: {message.InsertionTime}");
    Console.WriteLine($"Expiration Time: {message.ExpirationTime}");
    Console.WriteLine($"Dequeue Count: {message.DequeueCount}");
    
    // Update a message's content and visibility timeout
    await queueClient.UpdateMessageAsync(
        message.MessageId,
        message.PopReceipt,
        "Updated message content",
        TimeSpan.FromSeconds(60)); // New visibility timeout
}
```

> 💡 **Exam Tip**: For the AZ-204 exam, know how to use message properties to implement features like poison message handling (using DequeueCount) and message timeout management.

### Queue Storage vs. Service Bus Queues

|Feature|Queue Storage|Service Bus Queues|
|---|---|---|
|**Size**|Up to 64KB|Up to 256KB|
|**Protocol**|HTTP/HTTPS|AMQP, HTTP/HTTPS|
|**Ordering**|No guaranteed ordering|FIFO guaranteed with sessions|
|**Transaction**|No|Yes|
|**Auto-delete**|TTL only|TTL or auto-delete on receive|
|**Duplicate detection**|No|Yes|

> 💡 **Exam Tip**: Know when to use each type of queue based on requirements.

---

## 📁 File Storage

Azure File Storage offers fully managed file shares that use the SMB protocol.

### Key Concepts

- **File Share**: An SMB file share in the cloud
- **Directory**: A hierarchical organization structure for files
- **File**: Individual file with content up to 1TB in size

```csharp
// Create a share client
ShareClient share = new ShareClient(connectionString, "myshare");
await share.CreateIfNotExistsAsync();

// Create a directory
ShareDirectoryClient directory = share.GetDirectoryClient("mydirectory");
await directory.CreateIfNotExistsAsync();

// Upload a file
ShareFileClient file = directory.GetFileClient("myfile.txt");
await file.CreateAsync(1024); // Size in bytes
await file.UploadRangeAsync(
    new HttpRange(0, stream.Length),
    stream);
```

> ⚠️ **Gotcha**: Azure File shares have different performance characteristics compared to traditional SAN/NAS. Premium file shares offer better performance but at a higher cost.

### Properties and Metadata in File Storage

#### File and Directory Properties

Azure File Storage supports the following properties:

- **System properties**: Similar to blob storage (Last-Modified, ETag, Content-Length, etc.)
- **SMB properties**: Additional properties specific to SMB protocol

```csharp
// Get file properties
ShareFileProperties properties = await file.GetPropertiesAsync();

Console.WriteLine($"Last Modified: {properties.LastModified}");
Console.WriteLine($"Content Length: {properties.ContentLength}");
Console.WriteLine($"Content Type: {properties.ContentType}");
Console.WriteLine($"ETag: {properties.ETag}");

// Get directory properties
ShareDirectoryProperties dirProperties = await directory.GetPropertiesAsync();
```

#### File and Directory Metadata

Like blob storage, File Storage also supports user-defined metadata:

```csharp
// Set file metadata
IDictionary<string, string> metadata = new Dictionary<string, string>
{
    { "department", "Finance" },
    { "classification", "Confidential" },
    { "review-date", "2025-12-31" }
};

await file.SetMetadataAsync(metadata);

// Get and read metadata
ShareFileProperties properties = await file.GetPropertiesAsync();
foreach (var metadataItem in properties.Metadata)
{
    Console.WriteLine($"{metadataItem.Key}: {metadataItem.Value}");
}

// Set directory metadata
await directory.SetMetadataAsync(metadata);
```

> 💡 **Exam Tip**: For the exam, know how metadata can help with organizing and managing file shares, especially in enterprise environments where classification and data management are important.

### Common Use Cases

- **Lift-and-shift applications** that rely on file shares
- **Shared application settings**
- **Diagnostic logs, metrics, and crash dumps**
- **Development and testing environments**

---

## 🔐 Security Features

### Authentication and Authorization

#### Shared Key (Account Key)

```csharp
// Using account key
BlobServiceClient blobServiceClient = new BlobServiceClient(
    new Uri("https://mystorageaccount.blob.core.windows.net"),
    new StorageSharedKeyCredential("accountName", "accountKey"));
```

> ⚠️ **Gotcha**: Account keys provide full access to the storage account. Rotate keys regularly and consider using more granular access control mechanisms.

#### Shared Access Signatures (SAS)

- **Service SAS**: Access to one service (Blob, Queue, Table, File)
- **Account SAS**: Access to one or more services
- **User delegation SAS**: Secured with Azure AD credentials

```csharp
// Generate a service SAS for a blob
BlobSasBuilder sasBuilder = new BlobSasBuilder
{
    BlobContainerName = "mycontainer",
    BlobName = "myblob.txt",
    Resource = "b", // b for blob
    ExpiresOn = DateTimeOffset.UtcNow.AddHours(1)
};
sasBuilder.SetPermissions(BlobSasPermissions.Read); // Read-only access

BlobServiceClient blobServiceClient = new BlobServiceClient(connectionString);
string sasToken = sasBuilder.ToSasQueryParameters(
    new StorageSharedKeyCredential("accountName", "accountKey")).ToString();

// The SAS URL
string sasUrl = $"https://mystorageaccount.blob.core.windows.net/mycontainer/myblob.txt?{sasToken}";
```

> 💡 **Exam Tip**: Understand the different types of SAS, their permissions model, and how to create them programmatically.

#### Azure AD Integration

```csharp
// Using Azure AD authentication
TokenCredential credential = new DefaultAzureCredential();
BlobServiceClient blobServiceClient = new BlobServiceClient(
    new Uri("https://mystorageaccount.blob.core.windows.net"),
    credential);
```

### Encryption

- **Encryption at rest**: All data automatically encrypted using Storage Service Encryption (SSE)
- **Encryption in transit**: HTTPS/TLS by default
- **Client-side encryption**: Data encrypted before sending to Azure

> 💡 **Exam Tip**: Know how to implement customer-managed keys with Azure Key Vault.

---

## 📱 Azure Storage Data Movement

### AzCopy

Command-line utility for copying data to/from Azure Storage.

```bash
# Copy a local file to blob storage
azcopy copy "C:\local\path\file.txt" "https://myaccount.blob.core.windows.net/mycontainer/file.txt?<SAS>"

# Copy a blob to another storage account
azcopy copy "https://sourceaccount.blob.core.windows.net/mycontainer/file.txt?<SAS>" "https://destaccount.blob.core.windows.net/mycontainer/file.txt?<SAS>"
```

### Azure Storage Data Movement Library

```csharp
// Copy a blob from one container to another
BlobClient sourceBlob = sourceBlobContainerClient.GetBlobClient("sourcefile.txt");
BlobClient destBlob = destBlobContainerClient.GetBlobClient("destfile.txt");

// Start the copy operation
await destBlob.StartCopyFromUriAsync(sourceBlob.Uri);
```

> ⚠️ **Gotcha**: Large file uploads should use the recommended block size of 100MB for optimal performance.

---

## 🔍 Monitoring & Troubleshooting

### Metrics & Diagnostics

- **Azure Monitor**: Provides metrics for all storage services
- **Storage Analytics**: Logs detailed information about requests
- **Azure Storage resource logs**: Detailed logs about read, write, and delete operations

### Common Issues & Solutions

|Issue|Possible Cause|Solution|
|---|---|---|
|**403 (Forbidden)**|Invalid SAS token, expired token, insufficient permissions|Check SAS parameters, expiry time, and permissions|
|**404 (Not Found)**|Container/blob does not exist, incorrect path|Verify all path components are correct|
|**409 (Conflict)**|Resource already exists, lease conflict|Handle concurrency issues, check if resource exists before creating|
|**Timeout errors**|Network issues, server busy|Implement retry policy with exponential backoff|

> 💡 **Exam Tip**: Understand how to implement proper retry policies for transient errors and throttling.

---

## 📝 Practice Tasks

1. Create a storage account with RA-GZRS replication and move blobs through different access tiers
2. Implement a Shared Access Signature with limited permissions and time restrictions
3. Set up lifecycle management policies to archive blobs older than 30 days
4. Implement a message processor with Queue storage that handles poison messages
5. Use AzCopy to perform a large data migration between storage accounts

---

## 🔄 Key SDK Versions for AZ-204

For the AZ-204 exam, you should be familiar with the following NuGet package versions:

|Package|Recommended Version|
|---|---|
|Azure.Storage.Blobs|12.x|
|Azure.Storage.Queues|12.x|
|Azure.Storage.Files.Shares|12.x|
|Azure.Data.Tables|12.x|

> ⚠️ **Gotcha**: The exam may still contain questions based on older SDK versions (WindowsAzure.Storage). Understand the differences between the old and new SDKs.

---

## 🎯 Exam Preparation Tips

1. **Focus on authentication methods** - know how to use account keys, SAS tokens, and Azure AD
2. **Understand the right storage service** for different scenarios
3. **Be familiar with the C# SDK** for each storage service
4. **Know the tradeoffs** between different redundancy options
5. **Memorize the access tier details** including costs and retrieval times
6. **Practice creating and managing** Storage accounts using both Azure Portal and Azure CLI

---

## 🚨 Common Exam Pitfalls

- **Confusing Geo-Redundant Storage (GRS) with Geo-Zone-Redundant Storage (GZRS)**
- **Miscalculating storage costs** when changing access tiers
- **Forgetting SAS token permissions** and not accounting for necessary permissions
- **Not understanding the differences** between Blob, Queue, Table, and File storage
- **Overlooking the limitations** of each storage service
- **Using inappropriate container access levels** for specific scenarios
- **Not considering performance implications** of large blob storage operations
- **Misunderstanding of metadata limits** - 8KB total per storage resource
- **Forgetting that metadata headers** are case-insensitive for retrieval but case-preserving for storage
- **Not knowing how to effectively query based on properties or metadata**
