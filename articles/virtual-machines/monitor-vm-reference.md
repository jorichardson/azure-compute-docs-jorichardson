---
title: Monitoring data reference for Azure Virtual Machines
description: This article contains important reference material you need when you monitor Azure Virtual Machines.
ms.date: 01/24/2025
ms.custom: horz-monitor
ms.topic: reference
ms.service: azure-virtual-machines
# Customer intent: As a cloud administrator, I want access to monitoring metrics and logs for Azure Virtual Machines, so that I can effectively track performance, availability, and resource utilization to ensure optimal operation and quickly address any issues.
---

# Azure Virtual Machines monitoring data reference

[!INCLUDE [horz-monitor-ref-intro](~/reusable-content/ce-skilling/azure/includes/azure-monitor/horizontals/horz-monitor-ref-intro.md)]

See [Monitor Azure Virtual Machines](monitor-vm.md) for details on the data you can collect for Azure Virtual Machines and how to use it.

[!INCLUDE [horz-monitor-ref-metrics-intro](~/reusable-content/ce-skilling/azure/includes/azure-monitor/horizontals/horz-monitor-ref-metrics-intro.md)]

>[!IMPORTANT]
>Metrics for the guest operating system (guest OS) that runs in a virtual machine (VM) aren't listed here. Guest OS metrics must be collected through one or more agents that run on or as part of the guest operating system. Guest OS metrics include performance counters that track guest CPU percentage or memory usage, both of which are frequently used for autoscaling or alerting.
>
>Host OS metrics are available and listed in the following tables. Host OS metrics relate to the Hyper-V session that's hosting your guest OS session. For more information, see [Guest OS and host OS metrics](/azure/azure-monitor/reference/supported-metrics/metrics-index#guest-os-and-host-os-metrics).

### Supported metrics for Microsoft.Compute/virtualMachines
The following table lists the metrics available for the Microsoft.Compute/virtualMachines resource type.

[!INCLUDE [horz-monitor-ref-metrics-tableheader](~/reusable-content/ce-skilling/azure/includes/azure-monitor/horizontals/horz-monitor-ref-metrics-tableheader.md)]
[!INCLUDE [microsoft-compute-virtualmachines-metrics-include](~/reusable-content/ce-skilling/azure/includes/azure-monitor/reference/metrics/microsoft-compute-virtualmachines-metrics-include.md)]

For an example that shows how to collect the *Percentage CPU* metric from a VM, see [Get virtual machine usage metrics using the REST API](linux/metrics-vm-usage-rest.md).

### VM availability metric (preview)
The VM availability metric is currently in public preview. This metric value indicates whether a machine is currently running and available. You can use the metric to trend availability over time and to alert if the machine is stopped. VM availability displays the following values.

The VM availability metric is computed based on an aggregate of different signals from the host.

To learn how to use the VM availability metric to monitor Azure Virtual Machine availability, see [Use Azure Monitor to monitor Azure Virtual Machine availability](flash-azure-monitor.md).

| Value | Description |
|:---|:---|
| 1 | VM is running and available. |
| 0 | VM is unavailable. The VM could be stopped or rebooting. If you shut down a VM from within the VM, it emits this value. |
| Null (dashed line) | State of the VM is unknown. If you stop a VM from the Azure portal, CLI, or PowerShell, it immediately stops emitting the availability metric, and you see null values. |

**Context dimension** informs whether VM availability was influenced due to Azure or user orchestrated activity. It can assume values of *Platform*, *Customer*, or *Unknown*.

| Display name | Description |
| --- | --- |
| Aggregation | *Average* (default aggregation): for prioritized investigations based on extent of downtime incurred. <br><br>*Min*: immediately pinpoints all the times where the VM was unavailable. <br><br>*Max*: immediately pinpoints all the instances where the VM was available. <br><br>For more information on chart range, granularity, and data aggregation, see [Azure Monitor metrics aggregation and display explained](/azure/azure-monitor/essentials/metrics-aggregation-explained). |
| Data retention | Data for the VM availability metric is [stored for 93 days](/azure/azure-monitor/essentials/data-platform-metrics#retention-of-metrics) to help trend analysis and historical lookback. |
| Pricing | Refer to the [Pricing breakdown](https://azure.microsoft.com/pricing/details/monitor/#pricing), specifically in the *Metrics* and *Alert Rules* sections. |

[!INCLUDE [horz-monitor-ref-metrics-dimensions-intro](~/reusable-content/ce-skilling/azure/includes/azure-monitor/horizontals/horz-monitor-ref-metrics-dimensions-intro.md)]

The dimension Logical Unit Number (`LUN`) is associated with some of the preceding metrics.

[!INCLUDE [horz-monitor-ref-logs-tables](~/reusable-content/ce-skilling/azure/includes/azure-monitor/horizontals/horz-monitor-ref-logs-tables.md)]

| Table | Categories | Data collection method|[Supports basic log plan](/azure/azure-monitor/logs/basic-logs-configure?tabs=portal-1#compare-the-basic-and-analytics-log-data-plans)| Queries|
|---|---|---|---|---|
| [ADAssessmentRecommendation](/azure/azure-monitor/reference/tables/ADAssessmentRecommendation)<br>Recommendations generated by AD assessments that are started through a scheduled task. When you schedule the assessment it runs by default every seven days and uploads the data into Azure Log Analytics. | workloads | [Active Directory On-Demand Assessment](/services-hub/unified/health/getting-started-ad) | No| [Yes](/azure/azure-monitor/reference/queries/adassessmentrecommendation)|
| [AzureActivity](/azure/azure-monitor/reference/tables/AzureActivity)<br>Entries from the Azure Activity log that provides insight into any subscription-level or management group level events that have occurred in Azure. | resources, audit, security | [Export Activity log](/azure/azure-monitor/essentials/activity-log) | No| [Yes](/azure/azure-monitor/reference/queries/azureactivity)|
| [CommonSecurityLog](/azure/azure-monitor/reference/tables/CommonSecurityLog)<br>This table is for collecting events in the Common Event Format, that are most often sent from different security appliances such as Check Point, Palo Alto and more. | security | [Common Event Format (CEF) via AMA connector for Microsoft Sentinel](/azure/sentinel/data-connectors/common-event-format-cef) | No| [Yes](/azure/azure-monitor/reference/queries/commonsecuritylog)|
| [ConfigurationChange](/azure/azure-monitor/reference/tables/ConfigurationChange)<br>View changes to in-guest configuration data such as Files Software Registry Keys Windows Services and Linux Daemons | management |[Enable Change Tracking and Inventory](/azure/automation/change-tracking/enable-vms-monitoring-agent) | No| [Yes](/azure/azure-monitor/reference/queries/configurationchange)|
| [ConfigurationData](/azure/azure-monitor/reference/tables/ConfigurationData)<br>View the last reported state for in-guest configuration data such as Files Software Registry Keys Windows Services and Linux Daemons | management | [Enable Change Tracking and Inventory](/azure/automation/change-tracking/enable-vms-monitoring-agent) | No| [Yes](/azure/azure-monitor/reference/queries/configurationdata)|
| [ContainerLog](/azure/azure-monitor/reference/tables/ContainerLog)<br>Log lines collected from stdout and stderr streams for containers. | container, applications | [Container Insights](/azure/azure-monitor/containers/kubernetes-monitoring-enable) | No| [Yes](/azure/azure-monitor/reference/queries/containerlog)|
| [DnsEvents](/azure/azure-monitor/reference/tables/DnsEvents) | network | [Stream and filter data from Windows DNS servers with Azure Monitor Agent](/azure/sentinel/connect-dns-ama) | No| [Yes](/azure/azure-monitor/reference/queries/dnsevents)|
 | [DnsInventory](/azure/azure-monitor/reference/tables/DnsInventory) | network | [Stream and filter data from Windows DNS servers with Azure Monitor Agent](/azure/sentinel/connect-dns-ama) | No| -|
| [Event](/azure/azure-monitor/reference/tables/Event)<br>Events from Windows Event Log on Windows computers using Azure Monitor Agent Analytics agent. | virtualmachines | [Collect events with Azure Monitor Agent](/azure/azure-monitor/agents/data-collection-rule-azure-monitor-agent) | No| [Yes](/azure/azure-monitor/reference/queries/event)|
| [HealthStateChangeEvent](/azure/azure-monitor/reference/tables/HealthStateChangeEvent)<br>Workload Monitor Health. This data represents state transitions of a health monitor. | undefined | [VM Insights](/azure/azure-monitor/vm/vminsights-enable-overview) | No| -|
| [Heartbeat](/azure/azure-monitor/reference/tables/Heartbeat)<br>Records logged by Azure Monitor Agent once per minute to report on agent health. | virtualmachines, container, management | [Azure Monitor Agent](/azure/azure-monitor/agents/agents-overview) | No| [Yes](/azure/azure-monitor/reference/queries/heartbeat)|
| [InsightsMetrics](/azure/azure-monitor/reference/tables/InsightsMetrics)<br>Table that stores metrics. 'Perf' table also stores many metrics and over time they all will converge to InsightsMetrics.  | virtualmachines, container, resources | [VM Insights](/azure/azure-monitor/vm/vminsights-enable-overview), [Container Insights](/azure/azure-monitor/containers/kubernetes-monitoring-enable) | No| [Yes](/azure/azure-monitor/reference/queries/insightsmetrics)|
| [Perf](/azure/azure-monitor/reference/tables/Perf)<br>Performance counters from Windows and Linux agents that provide insight into the performance of hardware components operating systems and applications. | virtualmachines, container | [Collect performance counters from VMs with Azure Monitor Agent](/azure/azure-monitor/agents/data-collection-rule-azure-monitor-agent) | No| [Yes](/azure/azure-monitor/reference/queries/perf)|
| [ProtectionStatus](/azure/azure-monitor/reference/tables/ProtectionStatus)<br>Antimalware installation info and security health status of the machine: | security | [Enable Azure Monitor Agent in Defender for Cloud](/azure/defender-for-cloud/auto-deploy-azure-monitoring-agent) | No| [Yes](/azure/azure-monitor/reference/queries/protectionstatus)|
| [SQLAssessmentRecommendation](/azure/azure-monitor/reference/tables/SQLAssessmentRecommendation)<br>Recommendations generated by SQL assessments that are started through a scheduled task. When you schedule the assessment it runs by default every seven days and uploads the data into Azure Log Analytics. | workloads | [SQL Server On-Demand Assessment](/services-hub/unified/health/getting-started-sql) | No| [Yes](/azure/azure-monitor/reference/queries/sqlassessmentrecommendation)|
| [SecurityBaseline](/azure/azure-monitor/reference/tables/SecurityBaseline) | security | [Enable Azure Monitor Agent in Defender for Cloud](/azure/defender-for-cloud/auto-deploy-azure-monitoring-agent) | No| -|
| [SecurityBaselineSummary](/azure/azure-monitor/reference/tables/SecurityBaselineSummary) | security | [Enable Azure Monitor Agent in Defender for Cloud](/azure/defender-for-cloud/auto-deploy-azure-monitoring-agent) | No| -|
| [SecurityEvent](/azure/azure-monitor/reference/tables/SecurityEvent)<br>Security events collected from windows machines by Azure Security Center or Azure Sentinel. | security | [Windows Security Events via AMA connector for Microsoft Sentinel](/azure/sentinel/data-connectors/security-events-via-legacy-agent) | No| [Yes](/azure/azure-monitor/reference/queries/securityevent)|
| [Syslog](/azure/azure-monitor/reference/tables/Syslog)<br>Syslog events on Linux computers using Azure Monitor Agent. | virtualmachines, security | [Collect Syslog events with Azure Monitor Agent](/azure/azure-monitor/agents/data-collection-syslog) | No| [Yes](/azure/azure-monitor/reference/queries/syslog)|
| [Update](/azure/azure-monitor/reference/tables/Update)<br>Details for update schedule run. Includes information such as which updates where available and which were installed. | management, security | [Enable Update Management](/azure/automation/update-management/enable-from-portal) | No| [Yes](/azure/azure-monitor/reference/queries/update)|
| [UpdateRunProgress](/azure/azure-monitor/reference/tables/UpdateRunProgress)<br>Breaks down each run of your update schedule by the patches available at the time with details on the installation status of each patch. | management | [Enable Update Management](/azure/automation/update-management/enable-from-portal) | No| [Yes](/azure/azure-monitor/reference/queries/updaterunprogress)|
| [UpdateSummary](/azure/azure-monitor/reference/tables/UpdateSummary)<br>Summary for each update schedule run. Includes information such as how many updates weren't installed. | virtualmachines | [Enable Update Management](/azure/automation/update-management/enable-from-portal) | No| [Yes](/azure/azure-monitor/reference/queries/updatesummary)|
| [VMBoundPort](/azure/azure-monitor/reference/tables/VMBoundPort)<br>Traffic for open server ports on the monitored machine. | virtualmachines | [VM Insights](/azure/azure-monitor/vm/vminsights-enable-overview) | No| -|
| [VMComputer](/azure/azure-monitor/reference/tables/VMComputer)<br>Inventory data for servers collected by the Service Map and VM insights solutions using the Dependency agent and Azure Monitor Agent. | virtualmachines | [VM Insights](/azure/azure-monitor/vm/vminsights-enable-overview) | No| -|
| [VMConnection](/azure/azure-monitor/reference/tables/VMConnection)<br>Traffic for inbound and outbound connections to and from monitored computers. | virtualmachines | [VM Insights](/azure/azure-monitor/vm/vminsights-enable-overview) | No| -|
| [VMProcess](/azure/azure-monitor/reference/tables/VMProcess)<br>Process data for servers collected by the Service Map and VM insights solutions using the Dependency agent and Azure Monitor Agent. | virtualmachines | [VM Insights](/azure/azure-monitor/vm/vminsights-enable-overview) | No| -|
| [W3CIISLog](/azure/azure-monitor/reference/tables/W3CIISLog)<br>Internet Information Server (IIS) log on Windows computers using Azure Monitor Agent. | management, virtualmachines | [Collect IIS logs with Azure Monitor Agent](/azure/azure-monitor/agents/data-collection-iis) | No| [Yes](/azure/azure-monitor/reference/queries/w3ciislog)|
| [WindowsFirewall](/azure/azure-monitor/reference/tables/WindowsFirewall) | security | [Enable Azure Monitor Agent in Defender for Cloud](/azure/defender-for-cloud/auto-deploy-azure-monitoring-agent) | No| -|

[!INCLUDE [horz-monitor-ref-activity-log](~/reusable-content/ce-skilling/azure/includes/azure-monitor/horizontals/horz-monitor-ref-activity-log.md)]

The following table lists a few example operations that relate to creating VMs in the activity log. For a complete list of operations, see [Microsoft.Compute resource provider operations](/azure/role-based-access-control/resource-provider-operations#microsoftcompute).

| Operation | Description |
|:---|:---|
| Microsoft.Compute/virtualMachines/start/action | Starts the virtual machine |
| Microsoft.Compute/virtualMachines/restart/action | Deletes a managed cluster |
| Microsoft.Compute/virtualMachines/write | Creates a new virtual machine or updates an existing one |
| Microsoft.Compute/virtualMachines/deallocate/action | Powers off the virtual machine and releases the compute resources |
| Microsoft.Compute/virtualMachines/extensions/write | Creates a new virtual machine extension or updates an existing one |
| Microsoft.Compute/virtualMachineScaleSets/write | Starts the instances of the virtual machine scale set |

## Related content

- See [Monitor Virtual Machines](monitor-vm.md) for a description of monitoring Virtual Machines.
- See [Monitor Azure resources with Azure Monitor](/azure/azure-monitor/essentials/monitor-azure-resource) for details on monitoring Azure resources.
