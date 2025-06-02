# 📚 AZ-204 Study Guide: Azure Storage Solutions

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

#### Blob Rehydration (Archive Tier)

**Rehydration** is the process of moving a blob from the Archive tier to an online tier (Hot or Cool) so it can be accessed.

##### Key Concepts

- **Archive tier blobs are offline**: Cannot be read, copied, or modified until rehydrated
- **Rehydration time**: Can take up to 15 hours for standard priority
- **Rehydration priority**: Standard (up to 15 hours) or High (under 1 hour for blobs <10GB)
- **Cost implications**: Rehydration incurs additional charges

##### Rehydration Methods

**Method 1: Change the access tier**

```http
PUT https://myaccount.blob.core.windows.net/mycontainer/myblob.txt?comp=tier
Authorization: SharedKey myaccount:signature
x-ms-version: 2021-08-06
x-ms-access-tier: Hot
x-ms-rehydrate-priority: High
```

**Method 2: Copy to a new blob in an online tier**

```http
PUT https://myaccount.blob.core.windows.net/mycontainer/newblob.txt
Authorization: SharedKey myaccount:signature
x-ms-version: 2021-08-06
x-ms-copy-source: https://myaccount.blob.core.windows.net/mycontainer/archivedblob.txt
x-ms-access-tier: Hot
x-ms-rehydrate-priority: Standard
```

##### Azure CLI Examples

```bash
# Rehydrate by changing tier (modifies original blob)
az storage blob set-tier \
    --account-name mystorageaccount \
    --container-name mycontainer \
    --name archivedblob.txt \
    --tier Hot \
    --rehydrate-priority High

# Rehydrate by copying to new blob (preserves original)
az storage blob copy start \
    --account-name mystorageaccount \
    --destination-container mycontainer \
    --destination-blob rehydratedblob.txt \
    --source-container mycontainer \
    --source-blob archivedblob.txt \
    --tier Cool \
    --rehydrate-priority Standard
```

##### C# SDK Examples

```csharp
// Method 1: Change access tier (rehydrate in place)
BlobClient blobClient = containerClient.GetBlobClient("archivedblob.txt");

await blobClient.SetAccessTierAsync(
    AccessTier.Hot, 
    rehydratePriority: RehydratePriority.High);

// Method 2: Copy to new blob with online tier
BlobClient sourceBlobClient = containerClient.GetBlobClient("archivedblob.txt");
BlobClient destBlobClient = containerClient.GetBlobClient("rehydratedblob.txt");

await destBlobClient.StartCopyFromUriAsync(
    sourceBlobClient.Uri,
    accessTier: AccessTier.Cool,
    rehydratePriority: RehydratePriority.Standard);
```

##### Monitoring Rehydration Status

```csharp
// Check rehydration status
BlobProperties properties = await blobClient.GetPropertiesAsync();

Console.WriteLine($"Access Tier: {properties.AccessTier}");
Console.WriteLine($"Archive Status: {properties.ArchiveStatus}");
Console.WriteLine($"Rehydrate Priority: {properties.RehydratePriority}");

// Archive status values:
// - null: Blob is in online tier
// - "rehydrate-pending-to-hot": Rehydration to Hot tier in progress
// - "rehydrate-pending-to-cool": Rehydration to Cool tier in progress
```

##### REST API Response for Archived Blob

```http
GET https://myaccount.blob.core.windows.net/mycontainer/archivedblob.txt
Authorization: SharedKey myaccount:signature

Response:
HTTP/1.1 409 Conflict
x-ms-error-code: BlobArchived
x-ms-access-tier: Archive
x-ms-archive-status: rehydrate-pending-to-hot
x-ms-rehydrate-priority: High
```

> ⚠️ **Critical Gotchas for Exam:**
> 
> - **Cannot read archived blobs**: Any attempt to read/download will return HTTP 409 (Conflict)
> - **Rehydration takes time**: Up to 15 hours for standard priority
> - **Early deletion charges**: Moving from Archive before 180 days incurs fees
> - **Rehydration costs**: Additional charges apply for retrieving archived data
> - **High priority limitations**: Only for blobs smaller than 10GB

> 💡 **Exam Tips:**
> 
> - **Use Copy method** when you want to preserve the original archived blob
> - **Use SetTier method** when you want to permanently move the blob online
> - **Monitor archive status** to determine when rehydration is complete
> - **High priority rehydration** costs significantly more but completes in under 1 hour

### Lifecycle Management

Blob lifecycle management provides a rich, rule-based policy to transition your data to the best access tier and to expire data at the end of its lifecycle.

#### Key Concepts

- **Lifecycle policies**: Rules that automatically manage blob data based on age or other criteria
- **Actions**: What happens to blobs (tier changes, deletion)
- **Filters**: Which blobs the rule applies to (by prefix, blob type, etc.)
- **Conditions**: When the action should occur (days after creation, modification, etc.)

#### Supported Actions

|Action|Description|Applies To|
|---|---|---|
|**tierToCool**|Move to Cool tier|Block blobs, Append blobs|
|**tierToArchive**|Move to Archive tier|Block blobs, Append blobs|
|**delete**|Delete the blob|All blob types|
|**enableAutoTierToHotFromCool**|Auto-move back to Hot if accessed|Block blobs|

#### Condition Types

- **daysAfterModificationGreaterThan**: Days since last modification
- **daysAfterCreationGreaterThan**: Days since blob creation
- **daysAfterLastAccessTimeGreaterThan**: Days since last access (requires access tracking)

```json
{
  "rules": [
    {
      "name": "LogFilePolicy",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["logs/", "temp/"]
        },
        "actions": {
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 7
            },
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 30
            },
            "delete": {
              "daysAfterModificationGreaterThan": 365
            }
          },
          "snapshot": {
            "delete": {
              "daysAfterCreationGreaterThan": 90
            }
          },
          "version": {
            "delete": {
              "daysAfterCreationGreaterThan": 90
            }
          }
        }
      }
    },
    {
      "name": "MediaFilePolicy",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["media/videos/", "media/images/"]
        },
        "actions": {
          "baseBlob": {
            "tierToCool": {
              "daysAfterLastAccessTimeGreaterThan": 30
            },
            "tierToArchive": {
              "daysAfterLastAccessTimeGreaterThan": 180
            },
            "enableAutoTierToHotFromCool": true
          }
        }
      }
    }
  ]
}
```

#### Advanced Filtering with Blob Index Tags

```json
{
  "rules": [
    {
      "name": "TagBasedPolicy",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": ["blockBlob"],
          "blobIndexMatch": [
            {
              "name": "department",
              "op": "==",
              "value": "finance"
            },
            {
              "name": "classification",
              "op": "==",
              "value": "archived"
            }
          ]
        },
        "actions": {
          "baseBlob": {
            "delete": {
              "daysAfterModificationGreaterThan": 2555
            }
          }
        }
      }
    }
  ]
}
```

> ⚠️ **Gotcha**: Lifecycle policies run once per day and may take up to 24 hours to take effect. Early deletion fees apply when moving blobs to cooler tiers before minimum storage duration.

> 💡 **Exam Tip**: Know the minimum storage durations: Cool tier (30 days), Archive tier (180 days). Moving blobs before these periods incurs early deletion charges.

#### CLI Commands for Lifecycle Management

```bash
# Set a lifecycle policy from JSON file
az storage account management-policy create \
    --account-name mystorageaccount \
    --policy @policy.json \
    --resource-group myresourcegroup

# Get current lifecycle policy
az storage account management-policy show \
    --account-name mystorageaccount \
    --resource-group myresourcegroup
```

#### Best Practices for Lifecycle Policies

- **Test policies carefully**: Start with a small subset of data
- **Use appropriate filters**: Be specific with prefixes to avoid unintended actions
- **Consider access patterns**: Use last access time conditions for frequently accessed data
- **Monitor costs**: Understand the cost implications of tier transitions
- **Plan for retrieval**: Archive tier has hours of latency for data retrieval

### Blob Client Libraries

| Class                 | Description                                                                                                                                                                                                                                                          |
| --------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `BlobClient`          | The [`BlobClient`](https://learn.microsoft.com/en-us/dotnet/api/azure.storage.blobs.blobclient) allows you to manipulate Azure Storage blobs.                                                                                                                        |
| `BlobClientOptions`   | Provides the client configuration options for connecting to Azure Blob Storage.                                                                                                                                                                                      |
| `BlobContainerClient` | The [BlobContainerClient](https://learn.microsoft.com/en-us/dotnet/api/azure.storage.blobs.blobcontainerclient) allows you to manipulate Azure Storage containers and their blobs.                                                                                   |
| `BlobServiceClient`   | The [BlobServiceClient](https://learn.microsoft.com/en-us/dotnet/api/azure.storage.blobs.blobserviceclient) allows you to manipulate Azure Storage service resources and blob containers. The storage account provides the top-level namespace for the Blob service. |
| `BlobUriBuilder`      | The [BlobUriBuilder](https://learn.microsoft.com/en-us/dotnet/api/azure.storage.blobs.bloburibuilder) class provides a convenient way to modify the contents of a Uri instance to point to different Azure Storage resources like an account, container, or blob.    |

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

#### SetHttpHeaders vs SetMetadata

This is a critical distinction that frequently appears on the AZ-204 exam:

**SetHttpHeaders** - Sets standard HTTP headers that affect blob behavior:

```csharp
// Set HTTP headers - these control blob behavior and client interaction
BlobHttpHeaders headers = new BlobHttpHeaders
{
    ContentType = "application/pdf",           // MIME type for browsers/clients
    ContentLanguage = "en-US",                 // Content language
    ContentEncoding = "gzip",                  // Compression method
    ContentDisposition = "attachment; filename=report.pdf", // Download behavior
    CacheControl = "max-age=3600",             // Browser caching instructions
    ContentHash = computedHash                 // MD5 hash for integrity
};

await blobClient.SetHttpHeadersAsync(headers);
```

**SetMetadata** - Sets custom key-value pairs for application use:

```csharp
// Set metadata - custom information for your application logic
IDictionary<string, string> metadata = new Dictionary<string, string>
{
    { "department", "Finance" },               // Custom business logic
    { "confidentiality", "Internal" },        // Custom classification
    { "retention-period", "7-years" }         // Custom retention info
};

await blobClient.SetMetadataAsync(metadata);
```

**Key Differences:**

|Aspect|SetHttpHeaders|SetMetadata|
|---|---|---|
|**Purpose**|Controls how browsers/clients handle the blob|Stores custom application data|
|**Visibility**|Sent as HTTP headers to clients|Only accessible via Azure Storage APIs|
|**Impact**|Affects download behavior, caching, MIME type|No impact on client behavior|
|**Standard Headers**|Uses predefined HTTP headers|Uses custom key-value pairs|
|**Browser Interaction**|Directly affects how browsers process the file|Invisible to browsers|

> 💡 **Exam Tip**: Use SetHttpHeaders when you want to control how the blob behaves when accessed via HTTP (download as attachment, set MIME type, etc.). Use SetMetadata for storing custom information about the blob that your application needs to track.

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

## Standard HTTP properties for containers and blobs

Containers and blobs also support certain standard HTTP properties. Properties and metadata are both represented as standard HTTP headers; the difference between them is in the naming of the headers. Metadata headers are named with the header prefix `x-ms-meta-` and a custom name. Property headers use standard HTTP header names, as specified in the Header Field Definitions section 14 of the HTTP/1.1 protocol specification.

The standard HTTP headers supported on containers include:

- `ETag`
- `Last-Modified`

The standard HTTP headers supported on blobs include:

- `ETag`
- `Last-Modified`
- `Content-Length`
- `Content-Type`
- `Content-MD5`
- `Content-Encoding`
- `Content-Language`
- `Cache-Control`
- `Origin`
- `Range`
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

```json
{
  "signedVersion": "2021-08-06",
  "signedResource": "c",
  "signedPermissions": "rl",
  "signedStart": "2025-06-01T00:00:00Z",
  "signedExpiry": "2025-06-02T00:00:00Z",
  "signedIP": "192.168.1.0/24",
  "signedProtocol": "https"
}
```

**SAS URL Structure:**

```
https://myaccount.blob.core.windows.net/mycontainer?sv=2021-08-06&ss=b&srt=sco&sp=rwdlacupx&se=2025-06-02T00:00:00Z&st=2025-06-01T00:00:00Z&spr=https&sig=<signature>
```

**SAS Parameters:**

- **sv**: Signed version
- **ss**: Signed services (b=blob, q=queue, t=table, f=file)
- **srt**: Signed resource types (s=service, c=container, o=object)
- **sp**: Signed permissions (r=read, w=write, d=delete, l=list, a=add, c=create, u=update, p=process, x=execute)
- **se**: Signed expiry time
- **st**: Signed start time
- **sip**: Signed IP addresses
- **spr**: Signed protocol (https or http,https)
- **sig**: Signature

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

AzCopy is a **command-line utility** for copying data to/from Azure Storage. It's not a library that you embed in applications, but a standalone executable.

#### How AzCopy Works

- **Standalone executable**: Downloaded and run from command line or PowerShell
- **Cross-platform**: Available for Windows, Linux, and macOS
- **High performance**: Optimized for large-scale data transfers
- **Resumable transfers**: Can resume interrupted transfers

#### Basic AzCopy Commands

```bash
# Copy a local file to blob storage
azcopy copy "C:\local\path\file.txt" "https://myaccount.blob.core.windows.net/mycontainer/file.txt?<SAS>"

# Copy a blob to another storage account
azcopy copy "https://sourceaccount.blob.core.windows.net/mycontainer/file.txt?<SAS>" "https://destaccount.blob.core.windows.net/mycontainer/file.txt?<SAS>"

# Sync a directory to blob storage (one-way sync)
azcopy sync "C:\local\directory" "https://myaccount.blob.core.windows.net/mycontainer?<SAS>" --recursive

# Copy entire container
azcopy copy "https://sourceaccount.blob.core.windows.net/sourcecontainer?<SAS>" "https://destaccount.blob.core.windows.net/destcontainer?<SAS>" --recursive

# Copy with specific options
azcopy copy "C:\local\path\*" "https://myaccount.blob.core.windows.net/mycontainer?<SAS>" \
    --recursive \
    --include-pattern="*.pdf;*.docx" \
    --exclude-pattern="temp*" \
    --overwrite=true \
    --block-size-mb=100
```

#### Authentication Methods for AzCopy

```bash
# 1. Using SAS tokens (most common)
azcopy copy "source" "destination?<SAS>"

# 2. Using Azure AD authentication
azcopy login
azcopy copy "source" "destination"

# 3. Using Azure AD with service principal
azcopy login --service-principal --application-id <app-id> --tenant-id <tenant-id>
azcopy copy "source" "destination"
```

#### AzCopy Job Management

```bash
# List jobs
azcopy jobs list

# Show job details
azcopy jobs show <job-id>

# Resume a job
azcopy jobs resume <job-id>

# Clean up job files
azcopy jobs clean
```

#### Using AzCopy in Scripts and Automation

```bash
# PowerShell script example
$source = "C:\data\*"
$destination = "https://mystorageaccount.blob.core.windows.net/backup?$sasToken"

# Run AzCopy with error handling
$result = & azcopy copy $source $destination --recursive --overwrite=true
if ($LASTEXITCODE -ne 0) {
    Write-Error "AzCopy failed with exit code $LASTEXITCODE"
    exit 1
}
```

```bash
# Bash script example
#!/bin/bash
SOURCE="/home/data/*"
DEST="https://mystorageaccount.blob.core.windows.net/backup?$SAS_TOKEN"

# Run AzCopy with logging
azcopy copy "$SOURCE" "$DEST" --recursive --log-level=INFO --output-type=text

if [ $? -eq 0 ]; then
    echo "Transfer completed successfully"
else
    echo "Transfer failed"
    exit 1
fi
```

#### Calling AzCopy from Applications

While AzCopy itself is a CLI tool, you can call it from applications:

```csharp
// C# example: Calling AzCopy from an application
using System.Diagnostics;

public async Task<bool> TransferWithAzCopy(string source, string destination)
{
    var processStartInfo = new ProcessStartInfo
    {
        FileName = "azcopy",
        Arguments = $"copy \"{source}\" \"{destination}\" --recursive",
        RedirectStandardOutput = true,
        RedirectStandardError = true,
        UseShellExecute = false,
        CreateNoWindow = true
    };

    using var process = Process.Start(processStartInfo);
    if (process == null) return false;

    await process.WaitForExitAsync();
    return process.ExitCode == 0;
}
```

> ⚠️ **Gotcha**: AzCopy is not a .NET library - it's a separate executable that must be installed on the machine where it runs.

> 💡 **Exam Tip**: For the AZ-204 exam, know that AzCopy is the recommended tool for bulk data transfers and migration scenarios, but for programmatic access within applications, use the Azure Storage SDKs instead.

### Azure CLI Commands for Storage Management

```bash
# Create storage account
az storage account create \
    --name mystorageaccount \
    --resource-group myresourcegroup \
    --location eastus \
    --sku Standard_LRS \
    --kind StorageV2

# Create container
az storage container create \
    --name mycontainer \
    --account-name mystorageaccount \
    --public-access blob

# Upload blob with metadata
az storage blob upload \
    --account-name mystorageaccount \
    --container-name mycontainer \
    --name myblob.txt \
    --file ./local-file.txt \
    --metadata department=HR project=Q1Report

# Generate SAS token
az storage blob generate-sas \
    --account-name mystorageaccount \
    --container-name mycontainer \
    --name myblob.txt \
    --permissions r \
    --expiry 2025-06-03T00:00:00Z \
    --https-only
```

### Azure Storage SDK for Programmatic Access

For applications, use the Azure Storage SDKs instead of AzCopy:

```csharp
// Copy a blob from one container to another using SDK
BlobClient sourceBlob = sourceBlobContainerClient.GetBlobClient("sourcefile.txt");
BlobClient destBlob = destBlobContainerClient.GetBlobClient("destfile.txt");

// Start the copy operation
await destBlob.StartCopyFromUriAsync(sourceBlob.Uri);

// For large file uploads, use block upload
using var fileStream = File.OpenRead("largefile.dat");
await destBlob.UploadAsync(fileStream, new BlobUploadOptions
{
    TransferOptions = new StorageTransferOptions
    {
        MaximumTransferSize = 100 * 1024 * 1024, // 100MB blocks
        InitialTransferSize = 100 * 1024 * 1024
    }
});
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

For the AZ-204 exam, you should be familiar with REST API calls and JSON configurations:

### REST API Examples

#### Upload Blob with Metadata

```http
PUT https://myaccount.blob.core.windows.net/mycontainer/myblob.txt
Authorization: SharedKey myaccount:signature
x-ms-version: 2021-08-06
x-ms-blob-type: BlockBlob
Content-Type: text/plain
x-ms-meta-department: HR
x-ms-meta-project: AnnualReport
Content-Length: 1024

[blob content]
```

#### Set Blob Metadata

```http
PUT https://myaccount.blob.core.windows.net/mycontainer/myblob.txt?comp=metadata
Authorization: SharedKey myaccount:signature
x-ms-version: 2021-08-06
x-ms-meta-department: Finance
x-ms-meta-classification: Confidential
```

#### Set Blob Properties (HTTP Headers)

```http
PUT https://myaccount.blob.core.windows.net/mycontainer/myblob.txt?comp=properties
Authorization: SharedKey myaccount:signature
x-ms-version: 2021-08-06
x-ms-blob-content-type: application/pdf
x-ms-blob-content-disposition: attachment; filename=report.pdf
x-ms-blob-cache-control: max-age=3600
```

### Common JSON Configurations

#### Storage Account ARM Template

```json
{
  "type": "Microsoft.Storage/storageAccounts",
  "apiVersion": "2023-01-01",
  "name": "[parameters('storageAccountName')]",
  "location": "[parameters('location')]",
  "sku": {
    "name": "Standard_LRS"
  },
  "kind": "StorageV2",
  "properties": {
    "accessTier": "Hot",
    "supportsHttpsTrafficOnly": true,
    "minimumTlsVersion": "TLS1_2",
    "allowBlobPublicAccess": false
  }
}
```

### NuGet Package Versions

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
