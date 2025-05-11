
## View logs
To view logs from your application code within a container, you can use the [az container logs](https://learn.microsoft.com/en-us/cli/azure/container#az_container_logs) command.

The following sample output is log output from the example task-based container in [Set the command line in a container instance](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-start-command#azure-cli-example), after being provided an invalid URL using a command-line override:

```bash
az container logs --resource-group myResourceGroup --name mycontainer
```

## Attach output streams
The [az container attach](https://learn.microsoft.com/en-us/cli/azure/container#az_container_attach) command provides diagnostic information during container startup. Once the container starts, it streams STDOUT and STDERR to your local console.

For example, here's output from the task-based container in [Set the command line in a container instance](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-start-command#azure-cli-example), after being supplied a valid URL of a large text file to process:

```bash
az container attach --resource-group myResourceGroup --name mycontainer
```

## Get diagnostic events
If your container fails to deploy successfully, review the diagnostic information provided by the Azure Container Instances resource provider. To view the events for your container, run the [az container show](https://learn.microsoft.com/en-us/cli/azure/container#az_container_show) command:

```bash
az container show --resource-group myResourceGroup --name mycontainer
```