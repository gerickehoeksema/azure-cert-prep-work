# 📊 AZ-204 Study Guide: Azure Monitor

## 📋 Table of Contents

- [Overview](https://claude.ai/chat/88347036-0ffc-4d4e-8ffa-f2bbf686f443#overview)
- [Alert Types](https://claude.ai/chat/88347036-0ffc-4d4e-8ffa-f2bbf686f443#alert-types)
- [Autoscaling](https://claude.ai/chat/88347036-0ffc-4d4e-8ffa-f2bbf686f443#autoscaling)
- [Logging](https://claude.ai/chat/88347036-0ffc-4d4e-8ffa-f2bbf686f443#logging)
- [CLI Commands](https://claude.ai/chat/88347036-0ffc-4d4e-8ffa-f2bbf686f443#cli-commands)
- [Action Groups](https://claude.ai/chat/88347036-0ffc-4d4e-8ffa-f2bbf686f443#action-groups)
- [Alert Processing Rules](https://claude.ai/chat/88347036-0ffc-4d4e-8ffa-f2bbf686f443#alert-processing-rules)
- [Troubleshooting](https://claude.ai/chat/88347036-0ffc-4d4e-8ffa-f2bbf686f443#troubleshooting)
- [Exam Tips & Gotchas](https://claude.ai/chat/88347036-0ffc-4d4e-8ffa-f2bbf686f443#exam-tips--gotchas)

## Overview

Azure Monitor is a comprehensive monitoring solution for collecting, analyzing, and responding to telemetry from your cloud and on-premises environments. It helps you maximize performance and availability and proactively identify problems in your applications.

### 🔑 Key Components

- **Metrics** - Numerical values collected at regular intervals
- **Logs** - Records containing different kinds of data organized into records
- **Alerts** - Proactive notifications of issues
- **Autoscale** - Dynamic resource adjustment based on demand

## Alert Types

Alerts help detect and address issues before users notice them by proactively notifying you when monitoring data indicates potential problems.

|Alert Type|Description|Common Use Cases|
|---|---|---|
|**Metric Alerts**|Evaluate resource metrics at regular intervals|CPU usage, memory usage, response time|
|**Log Search Alerts**|Use Log Analytics queries to evaluate resource logs|Error patterns, security events, application logs|
|**Activity Log Alerts**|Triggered by new activity log events|Resource changes, service health notifications|
|**Smart Detection Alerts**|AI-powered anomaly detection|Performance issues, failure anomalies in web apps|
|**Prometheus Alerts**|Used for alerting on Prometheus metrics|Container monitoring, Kubernetes metrics|

### 🔍 Alert Components

An **alert rule** combines:

- Resources to be monitored
- Signals or data from the resource
- Conditions that trigger the alert

When an alert is triggered, it:

1. Initiates the associated action group
2. Updates the alert state
3. Stores the alert for 30 days before deletion

### 📝 Alert States

Alerts have two types of states:

1. **Condition states** (system-defined):
    
    - **Fired**: Alert condition is active
    - **Resolved**: Underlying condition cleared
2. **User response states**:
    
    - **New**: Alert detected but not reviewed
    - **Acknowledged**: Alert has been seen and is being worked on
    - **Closed**: Issue has been resolved

## Autoscaling

Autoscale allows you to dynamically add or remove resources to handle load fluctuations, ensuring optimal performance while minimizing costs.

### 🎯 Supported Services

- Azure Virtual Machine Scale Sets
- Azure Cloud Services
- Azure App Service (Web Apps)
- Azure Data Explorer cluster
- Azure API Management

### ⚙️ Autoscale Components

- **Autoscale Setting**: The overall configuration
- **Autoscale Profile**: Defines scale conditions and rules
- **Autoscale Rule**: Specific trigger conditions and resulting actions

### 📈 Metric-Based Autoscaling

Common metrics used for autoscaling:

- CPU percentage
- Memory usage
- Queue length
- HTTP queue length
- Data in/out
- Custom metrics

### 💡 Example Scenario

```
When: CPU > 70% for 10 minutes
Action: Add 1 instance
Cooldown: Wait 5 minutes before next scale action

When: CPU < 30% for 10 minutes
Action: Remove 1 instance
Cooldown: Wait 5 minutes before next scale action
```

## Logging

Application logging provides insights into application behavior, helps troubleshoot issues, and monitors application health.

### 🔧 Enabling App Logging (CLI)

```bash
# Enable filesystem logging
az webapp log config 
    --application-logging filesystem 
    --level verbose 
    --name <app-name> 
    --resource-group <resource-group-name>

# Reset to error-level only
az webapp log config 
    --application-logging off 
    --name <app-name> 
    --resource-group <resource-group-name>

# View current logging status
az webapp log show 
    --name <app-name> 
    --resource-group <resource-group-name>
```

### 📊 Log Levels

- **Error**: Errors and exceptions
- **Warning**: Potentially harmful situations
- **Information**: General informational messages
- **Verbose**: Detailed debugging information

## CLI Commands

### 📝 Autoscale Rule Management

|Command|Description|
|---|---|
|`az monitor autoscale rule copy`|Copy rules between profiles|
|`az monitor autoscale rule create`|Add a new autoscale rule|
|`az monitor autoscale rule delete`|Remove rules from a profile|
|`az monitor autoscale rule list`|List rules for a profile|

### 📋 Examples

**Create an autoscale rule**:

```bash
az monitor autoscale rule create -g myResourceGroup 
    --autoscale-name myVMSS 
    --scale to 5 
    --condition "Percentage CPU > 75 avg 10m"
```

**List autoscale rules**:

```bash
az monitor autoscale rule list 
    --autoscale-name MyAutoscale 
    --profile-name MyProfile 
    --resource-group MyResourceGroup
```

## Action Groups

Action groups define the list of actions to perform when an alert is triggered.

### 🔄 Notification Types

- Email notifications
- SMS messages
- Push notifications
- Voice calls
- Azure Function
- Logic App
- Webhook
- ITSM ticket
- Automation Runbook
- ARM Role

### 🛠️ Configuration

1. Navigate to Azure Monitor
2. Select "Alerts" > "Action groups"
3. Click "Add action group"
4. Configure name, short name, subscription, and resource group
5. Add one or more actions
6. Save the action group

## Alert Processing Rules

Alert processing rules allow you to modify alerts as they're being fired.

### 🔧 Capabilities

- Add or suppress action groups
- Apply filters to alerts
- Process alerts on a predefined schedule
- Set scope for alert processing

## Troubleshooting

### 🔍 Common Issues

- **Connectivity Problems**: Use Network Watcher or network diagnostics
- **Performance Issues**: Check metrics and logs for bottlenecks
- **Application Errors**: Review application logs and configure proper log levels
- **Alert Fatigue**: Refine alert thresholds and use alert processing rules

### 🛠️ Troubleshooting Steps

1. Identify the problem area (network, application, infrastructure)
2. Check relevant metrics and logs
3. Review recent changes or deployments
4. Apply appropriate resolution steps
5. Document the issue and solution

## Exam Tips & Gotchas

### ⚠️ Common Mistakes

- **Forgetting about cooldown periods** in autoscale rules
- Confusing **metric alerts** with **log alerts**
- Not understanding the difference between **platform metrics** and **custom metrics**
- Overlooking **alert processing rules** for alert management
- Missing the 30-day **retention period** for alerts

### 💡 Exam Tips

- Know the **alert types** and when to use each
- Understand **autoscale components** (settings, profiles, rules)
- Memorize key **CLI commands** for managing autoscale rules
- Be familiar with **action group capabilities**
- Understand how to configure **logging levels**
- Know which Azure services support **autoscaling**
- Recognize the difference between **condition states** and **user response states**

### 🎯 Focus Areas

1. **Alert Configuration**
    
    - Metric thresholds
    - Log query alerts
    - Action groups
2. **Autoscale Settings**
    
    - Rules and conditions
    - Profiles (recurring schedules)
    - Cooldown periods
3. **CLI Commands**
    
    - Monitoring autoscale rules
    - Logging configuration
4. **Troubleshooting**
    
    - Network connectivity
    - Resource performance
    - Application issues

---

## 📚 Additional Resources

- [Azure Monitor Documentation](https://learn.microsoft.com/en-us/azure/azure-monitor/)
- [Alerts Overview](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/alerts-overview)
- [Autoscale Overview](https://learn.microsoft.com/en-us/azure/azure-monitor/autoscale/autoscale-overview)
- [Azure CLI Monitor Commands](https://learn.microsoft.com/en-us/cli/azure/monitor)