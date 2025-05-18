# Azure Container Instances (ACI) - AZ-204 Exam Study Guide

## 📌 Overview
Azure Container Instances (ACI) provides the fastest and simplest way to run containers in Azure without having to manage virtual machines or adopt a higher-level orchestration service. It's a serverless container offering that allows you to run containerized applications on-demand in a managed, serverless Azure environment.

## Key Features

### Fast Startup
- Containers start in **seconds** without the need to provision or manage VMs
- Perfect for event-driven and burst workloads

### Per-Second Billing
- Precise billing per second for actual compute resources used
- No need to pay for unused capacity

### Hypervisor-Level Security
- Isolates your container from other customers for enhanced security
- Each container group is isolated like a VM would be

### Custom Sizes
- Specify exact CPU cores and memory for precise control
- Configure to your workload needs (no pre-defined sizes like VMs)

### Persistent Storage
- Mount Azure Files shares directly to containers
- Access shared data across container instances

### Linux and Windows Containers
- Supports both Linux and Windows container images
- Schedule Windows and Linux containers using the same API

## Container Groups

Container groups are the foundation of ACI. A container group:

- Is a collection of containers that share a lifecycle, resources, local network, and storage volumes
- Similar to a pod in Kubernetes
- Can contain one or multiple containers
- Is scheduled on the same host machine

### Multi-Container Groups
- Useful for splitting a single functional task into a small number of container images
- Enables reuse of container images developed by different teams
- Common scenarios:
  - Application container with a logging/monitoring sidecar
  - Application with an init container that runs setup tasks before the main container

### Networking in Container Groups
- All containers in a group share an IP address and port namespace
- Containers within a group can reach each other via localhost
- Container groups can communicate with each other via their private IP addresses within a VNet

## Deployment Options

### Resource Manager Templates
- Define complex deployments with multiple container instances
- Specify volumes, networking, and other resource configurations

### YAML Configuration
- Use YAML files for container group definitions
- Similar to Kubernetes pod definitions but with ACI-specific schemas

### Command Line
- Deploy using Azure CLI with `az container create`
- Quick deployments for testing or simple scenarios

### Portal
- Visual deployment using the Azure portal
- Good for learning and infrequent deployment needs

## Networking Options

### Public IP
- By default, container groups are assigned a public IP
- Accessible through the FQDN: `<container-group-name>.<region>.azurecontainer.io`

### Virtual Networks
- Deploy container groups into an Azure Virtual Network
- Communicate securely with other resources in the VNet
- Requires delegating a subnet to ACI using the Microsoft.ContainerInstance/containerGroups resource provider

### DNS Name Label
- Configure a DNS name label for internet-accessible containers
- Format: `<dns-name-label>.<region>.azurecontainer.io`

## Storage Options

### Temporary Storage
- Every container group has a default `/tmp` directory accessible by containers
- Contents lost when container group stops

### Azure Files Volume Mount
- Persist data beyond container lifecycle
- Share data between containers
- Mount an Azure Files share directly to a path in your container

### EmptyDir Volume
- Shared storage between containers in the same group
- Lifetime tied to the container group
- Useful for sharing data between containers during execution

### Secret Volume
- Mount sensitive data as files inside your containers
- Useful for certificates, keys, and other secrets

## Resource Allocation

### Allocation Settings
- **CPU**: Specify cores needed (can be fractional, like 0.5 cores)
- **Memory**: Specify GB needed for each container
- **GPU**: Optionally attach NVIDIA Tesla GPUs for compute-intensive workloads

### Resource Constraints
- Linux: Up to 4 cores, 16GB RAM per container group
- Windows: Up to 4 cores, 16GB RAM per container group
- GPU: Limited to specific regions and container versions

## Container Restart Policies

ACI offers three restart policy options:
- **Always**: Containers always restart if they stop
- **Never**: Containers run once and don't restart on exit
- **OnFailure**: Containers restart only if they exit with non-zero exit code

## Monitoring & Logging

### Container Logs
- View with Azure CLI: `az container logs`
- Available in Azure portal
- Stream logs in real-time with `az container attach`

### Metrics
- Available in Azure Monitor
- Track CPU and memory usage

### Diagnostic Events
- Review deployment failures with `az container show`
- Helpful for troubleshooting container startup issues

## Use Cases

### Ideal For:
- Simple applications
- Task automation and scheduled jobs
- Build jobs
- Event-driven processing
- Short-term batch processing

### Not Ideal For:
- Microservices requiring orchestration
- Complex multi-tier applications
- Long-running, high-availability applications

## Integration Points

### Azure Container Registry
- Seamlessly pull images from ACR
- Use ACR Tasks to build, run, and push container images to ACR

### Azure Logic Apps
- Run containers as part of a workflow
- Process data, send emails, or perform other tasks

### Azure Functions
- Trigger container instances from Azure Functions
- Use for extended compute capabilities beyond Functions limits

### Event Grid
- Trigger container instances from Event Grid events
- Respond to cloud or application events by running containers

## CLI Commands Reference

### Create a Container
```bash
az container create --resource-group myResourceGroup --name mycontainer \
    --image mycontainerimage:latest --dns-name-label mydnsname \
    --ports 80
```

### List Containers
```bash
az container list --resource-group myResourceGroup
```

### View Container Logs
```bash
az container logs --resource-group myResourceGroup --name mycontainer
```

### Attach to Containers
```bash
az container attach --resource-group myResourceGroup --name mycontainer
```

### View Container Details
```bash
az container show --resource-group myResourceGroup --name mycontainer
```

### Delete a Container
```bash
az container delete --resource-group myResourceGroup --name mycontainer
```

## Exam Tips & Gotchas

### Important Exam Tips
1. **Container Group Concept**: Understand that container groups are like Kubernetes pods - multiple containers that share resources and network
   
2. **Restart Policy**: Know the differences between Always, Never, and OnFailure restart policies and when to use each

3. **Networking**: Remember that all containers in a group share the same IP address and port space

4. **Resource Limits**: Memorize the maximum resources for container groups (4 CPUs, 16GB RAM)

5. **Mount Options**: Know the different types of volume mounts (Azure Files, Empty Dir, Secret volumes)

6. **Windows vs Linux**: Be aware of differences in features available between Windows and Linux containers

7. **Integration**: Understand how ACI works with Azure Functions, Logic Apps, and Event Grid

8. **Init Containers**: Know that ACI supports init containers for setup tasks before the main container runs

### Common Gotchas

1. **DNS Label Uniqueness**: DNS name labels must be unique within a region

2. **Virtual Network Limitations**: Container groups in a VNet must be created using Resource Manager templates or YAML files (not directly through portal)

3. **Windows Container Size**: Windows containers are larger and may start more slowly than Linux containers

4. **Port Exposure**: You must explicitly specify which ports to expose when creating container groups

5. **IP Address Sharing**: All containers in a group share the same IP address - remember to use different ports for different containers

6. **Restart Behavior**: OnFailure only restarts if the container exits with a non-zero code (not for crashes/OOM kills)

7. **Storage Persistence**: Data in the default /tmp directory is lost when containers stop - use Azure Files for persistence

8. **Region Availability**: Not all features (like GPU support) are available in all regions

9. **Resource Allocation**: Remember that you get billed for the resources you specify, not actual usage

10. **Container Registry Authentication**: When using private registries, you must provide registry credentials

## Comparison with Other Azure Container Services

| Feature | ACI | AKS | App Service | Service Fabric |
|---------|-----|-----|-------------|----------------|
| Complexity | Low | High | Medium | High |
| Startup Time | Seconds | Minutes | Minutes | Minutes |
| Orchestration | Basic | Advanced | Basic | Advanced |
| Auto-scaling | No | Yes | Yes | Yes |
| Ideal Workload | Simple, short-lived | Complex, long-running | Web apps | Stateful services |
| Management Overhead | Minimal | Significant | Moderate | Significant |

## Sample Questions

1. A company needs to run a container that processes data for 30 minutes and then exits. Which restart policy is most appropriate?
   - Answer: Never

2. Your container group needs to share data between two containers during execution. The data doesn't need to persist after the group stops. Which volume type should you use?
   - Answer: EmptyDir

3. What is the maximum number of CPU cores you can allocate to a single container group in ACI?
   - Answer: 4 cores

4. Which command would you use to stream logs in real-time from a running container instance?
   - Answer: `az container attach`

5. Your container needs to access sensitive connection strings. Which volume type provides the most secure way to mount these as files?
   - Answer: Secret volume
