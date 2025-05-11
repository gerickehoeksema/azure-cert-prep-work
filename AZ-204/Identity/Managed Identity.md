
# What are managed identities for Azure resources?

A common challenge for developers is the management of secrets, credentials, certificates, and keys used to secure communication between services. Manual handling of secrets and certificates are a known source of security issues and outages. Managed identities eliminate the need for developers to manage these credentials. Applications can use managed identities to obtain Microsoft Entra tokens without having to manage any credentials.

## What are managed identities?

At a high level, there are two types of identities: human and machine/non-human identities. Machine / non-human identities consist of device and workload identities. In Microsoft Entra, workload identities are applications, service principals, and managed identities.

A managed identity is an identity that can be assigned to an Azure compute resource (Virtual Machine (VM), Virtual Machine Scale Set (VMSS), Service Fabric Cluster, Azure Kubernetes cluster) or any [App hosting platform supported by Azure](https://learn.microsoft.com/en-us/azure/developer/intro/hosting-apps-on-azure?source=recommendations). Once a managed identity is assigned on the compute resource, it can be authorized, directly or indirectly, to access downstream dependency resources, such as a storage account, SQL database, CosmosDB, and so on. Managed identity replaces secrets such as access keys or passwords. In addition, managed identities can replace certificates or other forms of authentication for service-to-service dependencies.

## Managed identity types

- **System-assigned**. Some Azure resources, such as virtual machines allow you to enable a managed identity directly on the resource. When you enable a system-assigned managed identity:
    
    - A service principal of a special type is created in Microsoft Entra ID for the identity. The service principal is tied to the lifecycle of that Azure resource. When the Azure resource is deleted, Azure automatically deletes the service principal for you.
    - By design, only that Azure resource can use this identity to request tokens from Microsoft Entra ID.
    - You authorize the managed identity to have access to one or more services.
    - The name of the system-assigned service principal is always the same as the name of the Azure resource it's created for. For a deployment slot, the name of its system-assigned identity is `<app-name>/slots/<slot-name>`.
    
- **User-assigned**. You may also create a managed identity as a standalone Azure resource. You can [create a user-assigned managed identity](https://learn.microsoft.com/en-us/entra/identity/managed-identities-azure-resources/how-manage-user-assigned-managed-identities?pivots=identity-mi-methods-azp) and assign it to one or more Azure Resources. When you enable a user-assigned managed identity:
    
    - A service principal of a special type is created in Microsoft Entra ID for the identity. The service principal is managed separately from the resources that use it.
    - User-assigned identities can be used by multiple resources.
    - You authorize the managed identity to have access to one or more services.

## Differences between system-assigned and user-assigned managed identities

|Property|System-assigned managed identity|User-assigned managed identity|
|---|---|---|
|Creation|Created as part of an Azure resource (for example, Azure Virtual Machines or Azure App Service).|Created as a stand-alone Azure resource.|
|Life cycle|Shared life cycle with the Azure resource that the managed identity is created with.  <br>When the parent resource is deleted, the managed identity is deleted as well.|Independent life cycle.  <br>Must be explicitly deleted.|
|Sharing across Azure resources|Can’t be shared.  <br>It can only be associated with a single Azure resource.|Can be shared.  <br>The same user-assigned managed identity can be associated with more than one Azure resource.|
|Common use cases|Workloads contained within a single Azure resource.  <br>Workloads needing independent identities.  <br>For example, an application that runs on a single virtual machine.|Workloads that run on multiple resources and can share a single identity.  <br>Workloads needing preauthorization to a secure resource, as part of a provisioning flow.  <br>Workloads where resources are recycled frequently, but permissions should stay consistent.  <br>For example, a workload where multiple virtual machines need to access the same resource.|