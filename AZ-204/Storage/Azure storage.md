https://docs.microsoft.com/en-us/azure/storage/common/storage-auth-aad-app

https://learn.microsoft.com/en-us/azure/storage/blobs/authorize-access-azure-active-directory

https://docs.microsoft.com/en-us/azure/storage/common/storage-redundancy

https://docs.microsoft.com/en-us/azure/storage/blobs/storage-lifecycle-management-concepts?tabs=azure-portal

https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blob-immutability-policies-manage?tabs=azure-portal

https://docs.microsoft.com/en-us/dotnet/api/microsoft.azure.storage.blob.cloudblockblob.startcopyasync

https://docs.microsoft.com/en-us/rest/api/storageservices/designing-a-scalable-partitioning-strategy-for-azure-table-storage

https://docs.microsoft.com/en-us/azure/storage/common/storage-ref-azcopy-sync

https://docs.microsoft.com/en-us/azure/storage/common/storage-ref-azcopy-copy

https://docs.microsoft.com/en-us/azure/storage/common/storage-ref-azcopy?toc=/azure/storage/blobs/toc.json

https://docs.microsoft.com/en-us/azure/storage/file-sync/file-sync-introduction

Massively unstructured data storage is the specialty of blob storage. Data that doesn't follow a certain data model or specification, such text or binary data, is referred to as unstructured data.

Blob storage is intended for use in: 
● delivering documents or images straight to a browser. 
● putting files in storage for later access. 
● audio and video streaming. 
● logging data onto files. 
● archiving, disaster recovery, and backup and restore data storage. 
● storing data so that it can be analysed by an Azure or on-premises service.

### Types of storage Accounts 

● **Standard**: For the majority of Azure Storage scenarios, this basic general-purpose v2 account is advised. 
● **Premium**: By utilizing solid-state drives, Premium accounts provide improved performance. Block blobs, page blobs, and file shares are the three account types you can select from when creating a premium account.

### Access Tiers for block blob
● The **Hot** Access tier is designed to provide frequent access to storage account objects. The Hot tier offers the **lowest access charges** but **the highest storage expense**s. By default, new storage accounts are formed in the hot tier. 
● **Cool** Tier: Large volumes of **data that are rarely accessed and kept for at least 30 days** are best served by the Cool access tier. In comparison to the Hot tier, the Cool tier **has greater access prices** and **lower storage expenses**. 
● **Cold** Tier - An online tier optimized for **storing data that is rarely accessed or modified,** but **still requires fast retrieval**. Data in cold tier **should be stored for at least 90 days**. A cold tire has **lower storage costs** and **higher access costs** compared to a cool tire. 
● **Archive** Tier: The only tier accessible for individual block blobs is Archive. Data that can **withstand several hours of retrieval delay** and will **stay in the archive layer for at least 180 days** is best suited for the archive tier. Although accessing data in the archive tier is more expensive than accessing data in the hot or cool tiers, it is the most economical option for keeping data.

### Blob storage Resource Types
● Storage account. 
	- Your data in Azure has its own namespace
	- The base address for the items in your storage account is made up of the account name and the Azure Storage blob endpoint.
● Containers
	- a file system directory that holds a collection of blob
	- name is a component of the unique URI
	- it needs to be a valid DNS name
● Blobs
	- Binary and text data
	- consist of separate data blocks
	- capable of holding up to 190.7 TiB

### About block blobs
Block blobs are composed of blocks, each of which is identified by a block ID
 A block blob can include up to 50,000 blocks.
 Block IDs are strings of equal length within a blob
 Block ID values can be duplicated in different blobs.
 
![[Pasted image 20250509192439.png]]

## About page blobs
Page blobs are a collection of 512-byte pages optimized for random read and write operations
A write to a page blob can overwrite just one page, some pages, or up to 4 MiB of the page blob
The maximum size for a page blob is 8 TiB


## About append blobs
An append blob is composed of blocks and is optimized for append operations.
Updating or deleting of existing blocks is not supported. Unlike a block blob, an append blob does not expose its block IDs.

# Manage blob properties and metadata with .NET

#### Create a client object
To connect an app to Blob Storage, create an instance of [BlobServiceClient](https://learn.microsoft.com/en-us/dotnet/api/azure.storage.blobs.blobserviceclient). The following example shows how to create a client object using `DefaultAzureCredential` for authorization:

```c#
public BlobServiceClient GetBlobServiceClient(string accountName)
{
    BlobServiceClient client = new(
        new Uri($"https://{accountName}.blob.core.windows.net"),
        new DefaultAzureCredential());

    return client;
}
```

#### Authorization
For authorization with Microsoft Entra ID (recommended), you need Azure RBAC built-in role **Storage Blob Data Reader** or higher for the _get_ operations, and **Storage Blob Data Contributor** or higher for the _set_ operations

## About properties and metadata
- **System properties**: System properties exist on each Blob storage resource. Some of them can be read or set, while others are read-only. Under the covers, some system properties correspond to certain standard HTTP headers. The Azure Storage client library for .NET maintains these properties for you.
- **User-defined metadata**: User-defined metadata consists of one or more name-value pairs that you specify for a Blob storage resource. You can use metadata to store additional values with the resource. Metadata values are for your own purposes only, and don't affect how the resource behaves.

## Set and retrieve properties
To set properties on a blob, call [SetHttpHeaders](https://learn.microsoft.com/en-us/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.sethttpheaders) or [SetHttpHeadersAsync](https://learn.microsoft.com/en-us/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.sethttpheadersasync). Any properties not explicitly set are cleared. The following code example first gets the existing properties on the blob, then uses them to populate the headers that aren't being updated.

```c#
public static async Task SetBlobPropertiesAsync(BlobClient blob)
{
    Console.WriteLine("Setting blob properties...");

    try
    {
        // Get the existing properties
        BlobProperties properties = await blob.GetPropertiesAsync();

        BlobHttpHeaders headers = new BlobHttpHeaders
        {
            // Set the MIME ContentType every time the properties 
            // are updated or the field will be cleared
            ContentType = "text/plain",
            ContentLanguage = "en-us",

            // Populate remaining headers with 
            // the pre-existing properties
            CacheControl = properties.CacheControl,
            ContentDisposition = properties.ContentDisposition,
            ContentEncoding = properties.ContentEncoding,
            ContentHash = properties.ContentHash
        };

        // Set the blob's properties.
        await blob.SetHttpHeadersAsync(headers);
    }
    catch (RequestFailedException e)
    {
        Console.WriteLine($"HTTP error code {e.Status}: {e.ErrorCode}");
        Console.WriteLine(e.Message);
        Console.ReadLine();
    }
}
```

```c#
private static async Task GetBlobPropertiesAsync(BlobClient blob)
{
    try
    {
        // Get the blob properties
        BlobProperties properties = await blob.GetPropertiesAsync();

        // Display some of the blob's property values
        Console.WriteLine($" ContentLanguage: {properties.ContentLanguage}");
        Console.WriteLine($" ContentType: {properties.ContentType}");
        Console.WriteLine($" CreatedOn: {properties.CreatedOn}");
        Console.WriteLine($" LastModified: {properties.LastModified}");
    }
    catch (RequestFailedException e)
    {
        Console.WriteLine($"HTTP error code {e.Status}: {e.ErrorCode}");
        Console.WriteLine(e.Message);
        Console.ReadLine();
    }
}
```


## Set and retrieve metadata
You can specify metadata as one or more name-value pairs on a blob or container resource. To set metadata, add name-value pairs to the `Metadata` collection on the resource. Then, call one of the following methods to write the values:

- [SetMetadata](https://learn.microsoft.com/en-us/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.setmetadata)
- [SetMetadataAsync](https://learn.microsoft.com/en-us/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.setmetadataasync)

```c#
public static async Task AddBlobMetadataAsync(BlobClient blob)
{
    Console.WriteLine("Adding blob metadata...");

    try
    {
        IDictionary<string, string> metadata =
           new Dictionary<string, string>();

        // Add metadata to the dictionary by calling the Add method
        metadata.Add("docType", "textDocuments");

        // Add metadata to the dictionary by using key/value syntax
        metadata["category"] = "guidance";

        // Set the blob's metadata.
        await blob.SetMetadataAsync(metadata);
    }
    catch (RequestFailedException e)
    {
        Console.WriteLine($"HTTP error code {e.Status}: {e.ErrorCode}");
        Console.WriteLine(e.Message);
        Console.ReadLine();
    }
}
```

To retrieve metadata, call the [GetProperties](https://learn.microsoft.com/en-us/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.getproperties) or [GetPropertiesAsync](https://learn.microsoft.com/en-us/dotnet/api/azure.storage.blobs.specialized.blobbaseclient.getpropertiesasync) method on your blob or container to populate the [Metadata](https://learn.microsoft.com/en-us/dotnet/api/azure.storage.blobs.models.blobproperties.metadata) collection, then read the values, as shown in the example below. The `GetProperties` method retrieves blob properties and metadata by calling both the [Get Blob Properties](https://learn.microsoft.com/en-us/rest/api/storageservices/get-blob-properties) operation and the [Get Blob Metadata](https://learn.microsoft.com/en-us/rest/api/storageservices/get-blob-metadata) operation.

```c#
public static async Task ReadBlobMetadataAsync(BlobClient blob)
{
    try
    {
        // Get the blob's properties and metadata.
        BlobProperties properties = await blob.GetPropertiesAsync();

        Console.WriteLine("Blob metadata:");

        // Enumerate the blob's metadata.
        foreach (var metadataItem in properties.Metadata)
        {
            Console.WriteLine($"\tKey: {metadataItem.Key}");
            Console.WriteLine($"\tValue: {metadataItem.Value}");
        }
    }
    catch (RequestFailedException e)
    {
        Console.WriteLine($"HTTP error code {e.Status}: {e.ErrorCode}");
        Console.WriteLine(e.Message);
        Console.ReadLine();
    }
}
```


# az storage blob copy
Manage blob copy operations. Use `az storage blob show` to check the status of the blobs.

  

|Name|Description|Type|Status|
|---|---|---|---|
|[az storage blob copy cancel](https://learn.microsoft.com/en-us/cli/azure/storage/blob/copy?view=azure-cli-latest#az-storage-blob-copy-cancel)|Abort an ongoing copy operation.|Core|GA|
|[az storage blob copy start](https://learn.microsoft.com/en-us/cli/azure/storage/blob/copy?view=azure-cli-latest#az-storage-blob-copy-start)|Copy a blob asynchronously. Use `az storage blob show` to check the status of the blobs.|Core|GA|
|[az storage blob copy start (storage-blob-preview extension)](https://learn.microsoft.com/en-us/cli/azure/storage/blob/copy?view=azure-cli-latest#az-storage-blob-copy-start\(storage-blob-preview\))|Start a copy blob job.|Extension|GA|
|[az storage blob copy start-batch](https://learn.microsoft.com/en-us/cli/azure/storage/blob/copy?view=azure-cli-latest#az-storage-blob-copy-start-batch)|Copy multiple blobs to a blob container. Use `az storage blob show` to check the status of the blobs.|Core|GA|

### Examples

Copy a blob asynchronously. Use `az storage blob show` to check the status of the blobs.
```bash
az storage blob copy start 
	--account-key 00000000 
	--account-name MyAccount 
	--destination-blob MyDestinationBlob 
	--destination-container MyDestinationContainer 
	--source-uri https://storage.blob.core.windows.net/photos
```

Copy a blob asynchronously. Use `az storage blob show` to check the status of the blobs.
```bash
az storage blob copy start 
	--account-name MyAccount 
	--destination-blob MyDestinationBlob 
	--destination-container MyDestinationContainer 
	--sas-token $sas 
	--source-uri https://storage.blob.core.windows.net/photos
```

Copy a blob specific version
```bash
az storage blob copy start 
	--account-name MyAccount 
	--destination-blob MyDestinationBlob 
	--destination-container MyDestinationContainer 
	--source-uri https://my-account.blob.core.windows.net/my-container/my-blob?versionId=2022-03-21T18:28:44.4431011Z 
	--auth-mode login
```


# Start-AzStorageBlobCopy

The **Start-AzStorageBlobCopy** cmdlet starts to copy a blob.

## Examples
### Example 1: Copy a named blob
```bash
Start-AzStorageBlobCopy 
	-SrcBlob "ContosoPlanning2015" 
	-DestContainer "ContosoArchives" 
	-SrcContainer "ContosoUploads"
```
This command starts the copy operation of the blob named ContosoPlanning2015 from the container named ContosoUploads to the container named ContosoArchives.

### Example 2: Get a container to specify blobs to copy
```bash
Get-AzStorageContainer 
	-Name "ContosoUploads" | Start-AzStorageBlobCopy 
	-SrcBlob "ContosoPlanning2015" 
	-DestContainer "ContosoArchives"
```
This command gets the container named ContosoUploads, by using the **Get-AzStorageContainer** cmdlet, and then passes the container to the current cmdlet by using the pipeline operator. That cmdlet starts the copy operation of the blob named ContosoPlanning2015. The previous cmdlet provides the source container. The _DestContainer_ parameter specifies ContosoArchives as the destination container.

### Example 3: Get all blobs in a container and copy them
```bash
Get-AzStorageBlob 
	-Container "ContosoUploads" | Start-AzStorageBlobCopy 
	-DestContainer "ContosoArchives"
```
This command gets the blobs in the container named ContosoUploads, by using the **Get-AzStorageBlob** cmdlet, and then passes the results to the current cmdlet by using the pipeline operator. That cmdlet starts the copy operation of the blobs to the container named ContosoArchives.

### Example 4: Copy a blob specified as an object
```bash
$SrcBlob = Get-AzStorageBlob -Container "ContosoUploads" -Blob "ContosoPlanning2015"
$DestBlob = Get-AzStorageBlob -Container "ContosoArchives" -Blob "ContosoPlanning2015Archived"
Start-AzStorageBlobCopy -ICloudBlob $SrcBlob.ICloudBlob -DestICloudBlob $DestBlob.ICloudBlob
```
The first command gets the blob named ContosoPlanning2015 in the container named ContosoUploads. The command stores that object in the $SrcBlob variable. The second command gets the blob named ContosoPlanning2015Archived in the container named ContosoArchives. The command stores that object in the $DestBlob variable. The last command starts the copy operation from the source container to the destination container. The command uses standard dot notation to specify the **ICloudBlob** objects for the $SrcBlob and $DestBlob blobs.

### Example 5: Copy a blob from a URI
```bash
$Context = New-AzStorageContext -StorageAccountName "ContosoGeneral" -StorageAccountKey "< Storage Key for ContosoGeneral ends with == >"
Start-AzStorageBlobCopy -AbsoluteUri "http://www.contosointernal.com/planning" -DestContainer "ContosoArchive" -DestBlob "ContosoPlanning2015" -DestContext $Context
```
This command creates a context for the account named ContosoGeneral that uses the specified key, and then stores that key in the $Context variable. The second command copies the file from the specified URI to the blob named ContosoPlanning in the container named ContosoArchive. The command starts the copy operation to the destination context stored in $Context. There are no source storage context, so the source Uri must have access to the source object. E.g: if the source is a none public Azure blob, the Uri should contain SAS token which has read access to the blob.

### Example 6: Copy a block blob to destination container with a new blob name, and set destination blob StandardBlobTier as Hot, RehydratePriority as High
```bash
Start-AzStorageBlobCopy 
	-SrcContainer "ContosoUploads" 
	-SrcBlob "BlockBlobName" 
	-DestContainer "ContosoArchives" 
	-DestBlob "NewBlockBlobName" 
	-StandardBlobTier Hot 
	-RehydratePriority High
```
This command starts the copy operation of a block blob to destination container with a new blob name, and set destination blob StandardBlobTier as Hot, RehydratePriority as High


# Lease Blob

The `Lease Blob` operation creates and manages a lock on a blob for write and delete operations. The lock duration can be 15 to 60 seconds, or can be infinite. In versions prior to 2012-02-12, the lock duration is 60 seconds.

You can call the `Lease Blob` operation in one of the following modes:
- `Acquire`, to request a new lease.
- `Renew`, to renew an existing lease.
- `Change`, to change the ID of an existing lease.
- `Release`, to free the lease if it's no longer needed, so that another client can immediately acquire a lease against the blob.
- `Break`, to end the lease, but ensure that another client can't acquire a new lease until the current lease period has expired.