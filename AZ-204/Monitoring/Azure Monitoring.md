
https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-overview

https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/action-groups
https://learn.microsoft.com/en-us/azure/virtual-network/virtual-network-troubleshoot-connectivity-problem-between-vms
https://learn.microsoft.com/en-us/Azure/azure-monitor/alerts/tutorial-metric-alert

## Enable logging using the Azure CLI

To enable app logging to the file system, run this command.
```bash
az webapp log config 
	--application-logging filesystem 
	--level verbose 
	--name <app-name> 
	--resource-group <resource-group-name>
```

There's currently no way to disable application logging by using Azure CLI commands. However, the following command resets file system logging to error-level only.
```bash
az webapp log config 
	--application-logging off 
	--name <app-name> 
	--resource-group <resource-group-name>
```

To view the current logging status for an app, use this command.
```bash
az webapp log show 
	--name <app-name> 
	--resource-group <resource-group-name>
```


# Autoscale a web app by using custom metrics
Autoscale allows you to add and remove resources to handle increases and decreases in load

Azure Monitor autoscale applies to:

- [Azure Virtual Machine Scale Sets](https://azure.microsoft.com/services/virtual-machine-scale-sets/)
- [Azure Cloud Services](https://azure.microsoft.com/services/cloud-services/)
- [Azure App Service - Web Apps](https://azure.microsoft.com/services/app-service/web/)
- [Azure Data Explorer cluster](https://azure.microsoft.com/services/data-explorer/)
- [Azure API Management](https://learn.microsoft.com/en-us/azure/api-management/api-management-key-concepts)


# az monitor autoscale rule


|Name|Description|Type|Status|
|---|---|---|---|
|[az monitor autoscale rule copy](https://learn.microsoft.com/en-us/cli/azure/monitor/autoscale/rule?view=azure-cli-latest#az-monitor-autoscale-rule-copy)|Copy autoscale rules from one profile to another.|Core|GA|
|[az monitor autoscale rule create](https://learn.microsoft.com/en-us/cli/azure/monitor/autoscale/rule?view=azure-cli-latest#az-monitor-autoscale-rule-create)|Add a new autoscale rule.|Core|GA|
|[az monitor autoscale rule delete](https://learn.microsoft.com/en-us/cli/azure/monitor/autoscale/rule?view=azure-cli-latest#az-monitor-autoscale-rule-delete)|Remove autoscale rules from a profile.|Core|GA|
|[az monitor autoscale rule list](https://learn.microsoft.com/en-us/cli/azure/monitor/autoscale/rule?view=azure-cli-latest#az-monitor-autoscale-rule-list)|List autoscale rules for a profile.|Core|GA|

## az monitor autoscale rule copy
Copy autoscale rules from one profile to another.

```bash
az monitor autoscale rule copy --autoscale-name
                               --dest-schedule
                               --index
                               --resource-group
                               [--source-schedule]
```


## az monitor autoscale rule create
Add a new autoscale rule.

```bash
az monitor autoscale rule create --autoscale-name
                                 --condition
                                 --scale
                                 [--cooldown]
                                 [--profile-name]
                                 [--resource]
                                 [--resource-group]
                                 [--resource-namespace]
                                 [--resource-parent]
                                 [--resource-type]
                                 [--timegrain]
```

```bash
az monitor autoscale rule create -g {myrg} 
	--autoscale-name {myvmss} 
	--scale to 5 
	--condition "Percentage CPU > 75 avg 10m"
```

## az monitor autoscale rule delete
Remove autoscale rules from a profile.

```bash
az monitor autoscale rule delete --autoscale-name
                                 --index
                                 --resource-group
                                 [--profile-name]
```

## az monitor autoscale rule list
List autoscale rules for a profile.

```bash
az monitor autoscale rule list --autoscale-name
                               --resource-group
                               [--profile-name]
```

```bash
az monitor autoscale rule list 
	--autoscale-name MyAutoscale 
	--profile-name MyProfile 
	--resource-group MyResourceGroup
```

# What are Azure Monitor alerts?

Alerts help you detect and address issues before users notice them by proactively notifying you when Azure Monitor data indicates there might be a problem with your infrastructure or application.

You can alert on any metric or log data source in the Azure Monitor data platform.

![[Pasted image 20250511193513.png]]

An **alert rule** monitors your data and captures a signal that indicates something is happening on the specified resource. The alert rule captures the signal and checks to see if the signal meets the criteria of the condition.

An alert rule combines:

- The resources to be monitored.
- The signal or data from the resource.
- Conditions.

An **alert** is triggered if the conditions of the alert rule are met. The alert initiates the associated action group and updates the state of the alert. If you're monitoring more than one resource, the alert rule condition is evaluated separately for each of the resources, and alerts are fired for each resource separately.

Alerts are stored for 30 days and are deleted after the 30-day retention period. You can see all alert instances for all of your Azure resources on the [Alerts page](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-manage-alert-instances) in the Azure portal.

Alerts consist of:

- **Action groups**: These groups can trigger notifications to let users know that an alert has been triggered or start automated workflows. Action groups can include:
    - Notification methods, such as email, SMS, and push notifications.
    - Automation runbooks.
    - Azure functions.
    - ITSM incidents.
    - Logic apps.
    - Secure webhooks.
    - Webhooks.
    - Event hubs.
- **Alert conditions**: These conditions are set by the system. When an alert fires, the alert condition is set to **fired**. After the underlying condition that caused the alert to fire clears, the alert condition is set to **resolved**.
- **User response**: The response is set by the user and doesn't change until the user changes it. The User response can be **New**, **Acknowledged**, or **Closed**.
- **Alert processing rules**: You can use alert processing rules to make modifications to triggered alerts as they're being fired. You can use alert processing rules to add or suppress action groups, apply filters, or have the rule processed on a predefined schedule.

## Types of alerts

|Alert type|Description|
|---|---|
|[Metric alerts](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-types#metric-alerts)|Metric alerts evaluate resource metrics at regular intervals. Metrics can be platform metrics, custom metrics, logs from Azure Monitor converted to metrics, or Application Insights metrics. Metric alerts can also apply multiple conditions and dynamic thresholds.|
|[Log search alerts](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-types#log-alerts)|Log search alerts allow users to use a Log Analytics query to evaluate resource logs at a predefined frequency.|
|[Activity log alerts](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-types#activity-log-alerts)|Activity log alerts are triggered when a new activity log event occurs that matches defined conditions. Resource Health alerts and Service Health alerts are activity log alerts that report on your service and resource health.|
|[Smart detection alerts](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-types#smart-detection-alerts)|Smart detection on an Application Insights resource automatically warns you of potential performance problems and failure anomalies in your web application. You can migrate smart detection on your Application Insights resource to create alert rules for the different smart detection modules.|
|[Prometheus alerts](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-types#prometheus-alerts)|Prometheus alerts are used for alerting on Prometheus metrics stored in [Azure Monitor managed services for Prometheus](https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/prometheus-metrics-overview). The alert rules are based on the PromQL open-source query language.|