
# View log streams in Azure Container Apps

- [system logs](https://learn.microsoft.com/en-us/azure/container-apps/logging#system-logs) from the Container Apps environment and your container app.
- container [console logs](https://learn.microsoft.com/en-us/azure/container-apps/logging#container-console-logs) from your container app.

Log streams are accessible through the Azure portal or the Azure CLI.

## View log streams via the Azure CLI
You can view your container app's log streams from the Azure CLI with the `az containerapp logs show` command or your container app's environment system log stream with the `az containerapp env logs show` command.

Control the log stream with the following arguments:

- `--tail` (Default) View the last n log messages. Values are 0-300 messages. The default is 20.
- `--follow` View a continuous live stream of the log messages.

### Stream Container app logs
You can stream the system or console logs for your container app. To stream the container app system logs, use the `--type` argument with the value `system`. To stream the container console logs, use the `--type` argument with the value `console`. The default is `console`.

#### View container app system log stream
This example uses the `--tail` argument to display the last 50 system log messages from the container app.

```bash
az containerapp logs show 
	--name <CONTAINER_APP_NAME> 
	--resource-group <RESOURCE_GROUP> 
	--type system 
	--tail 50
```

This example displays a continuous live stream of system log messages from the container app using the `--follow` argument

```bash
az containerapp logs show 
	--name <CONTAINER_APP_NAME> 
	--resource-group <RESOURCE_GROUP> 
	--type system 
	--follow
```

### View container console log stream
To connect to a container's console log stream in a container app with multiple revisions, replicas, and containers, include the following parameters in the `az containerapp logs show` command.

|Argument|Description|
|---|---|
|`--revision`|The revision name.|
|`--replica`|The replica name in the revision.|
|`--container`|The container name to connect to.|
```bash
az containerapp revision list 
	--name <CONTAINER_APP_NAME> 
	--resource-group <RESOURCE_GROUP> 
	--query "[].name"
```

Use the `az containerapp replica list` command to get the replica and container names.

```bash
az containerapp replica list 
	--name <CONTAINER_APP_NAME> 
	--resource-group <RESOURCE_GROUP> 
	--revision <REVISION_NAME> 
	--query "[].{Containers:properties.containers[].name, Name:name}"
```

Live stream the container console using the `az container app show` command with the `--follow` argument

```bash
az containerapp logs show 
	--name <CONTAINER_APP_NAME> 
	--resource-group <RESOURCE_GROUP> 
	--revision <REVISION_NAME> 
	--replica <REPLICA_NAME> 
	--container <CONTAINER_NAME> 
	--type console 
	--follow
```

View the last 50 console log messages using the `az containerapp logs show` command with the `--tail` argument.

```bash
az containerapp logs show 
	--name <CONTAINER_APP_NAME> 
	--resource-group <RESOURCE_GROUP> 
	--revision <REVISION_NAME> 
	--replica <REPLICA_NAME> 
	--container <CONTAINER_NAME> 
	--type console 
	--tail 50
```

### View environment system log stream

```bash
az containerapp env logs show 
	--name <ENVIRONMENT_NAME> 
	--resource-group <RESOURCE_GROUP> 
	--follow
```