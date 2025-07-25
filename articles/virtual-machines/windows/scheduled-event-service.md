---
title: Monitor scheduled events for your Windows VMs in Azure 
description: Learn how to monitor your Windows virtual machines for scheduled events.
author: mysarn
ms.service: azure-virtual-machines
ms.subservice: scheduled-events
ms.date: 12/12/2024
ms.author: sarn
ms.topic: how-to
# Monitoring Scheduled Events
# Customer intent: "As a system administrator, I want to monitor scheduled events for my Windows VMs, so that I can proactively manage maintenance and minimize downtime for critical applications."
---

# Monitor scheduled events for your Windows VMs

**Applies to:** VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

Updates are applied to different parts of Azure every day, to keep the services running on them secure, and up-to-date. In addition to planned updates, unplanned events may also occur. For example, if any hardware degradation or fault is detected, Azure services may need to perform unplanned maintenance. Using live migration, memory preserving updates and keeping a strict limit on the impact of updates, allow these events to be almost transparent to customers. At most, they can cause a few seconds of virtual machine freeze. For some applications however, even a few seconds of virtual machine freeze could cause an impact. Knowing in advance about upcoming Azure maintenance is important to ensure the best experience for those applications. [Scheduled Events service](scheduled-events.md) provides you with a programmatic interface to be notified about upcoming maintenance, and enables you to gracefully handle the maintenance. 

This article shows how you can use scheduled events to be notified about maintenance events that could affect your VMs and build some basic automation that can help with monitoring and analysis.


## Routing scheduled events to Log Analytics

Scheduled Events is available as part of the [Azure Instance Metadata Service](instance-metadata-service.md), which is available on every Azure virtual machine. Customers can write automation to query the endpoint of their virtual machines to find scheduled maintenance notifications and perform mitigations, like saving the state and taking the virtual machine out of rotation. We recommend building automation to record the Scheduled Events so you can have an auditing log of Azure maintenance events. 

In this article, we walk you through how to capture maintenance Scheduled Events to Log Analytics. Then, we trigger some basic notification actions, like sending an email to your team and getting a historical view of all events that have affected your virtual machines. For the event aggregation and automation we use [Log Analytics](/azure/azure-monitor/logs/quick-create-workspace), but you can use any monitoring solution to collect these logs and trigger automation.

![Diagram showing the event lifecycle](./media/notifications/events.png)

## Prerequisites

For this example, you need to create a [Windows Virtual Machine in an Availability Set](tutorial-availability-sets.md). Scheduled Events provide notifications about changes that can affect any of the virtual machines in your availability set, Cloud Service, Virtual Machine Scale Set or standalone VMs. A [service](https://github.com/microsoft/AzureScheduledEventsService) is run that polls for scheduled events on one of the VMs that act as a collector, to get events for all of the other VMs in the availability set.    

Don't delete the group resource group at the end of the tutorial.

You also need to [create a Log Analytics workspace](/azure/azure-monitor/logs/quick-create-workspace) that we use to aggregate information from the VMs in the availability set.

## Set up the environment

You should now have 2 initial VMs in an availability set. Now we need to create a 3rd VM, called `myCollectorVM`, in the same availability set. 

```azurepowershell-interactive
New-AzVm `
   -ResourceGroupName "myResourceGroupAvailability" `
   -Name "myCollectorVM" `
   -Location "East US" `
   -VirtualNetworkName "myVnet" `
   -SubnetName "mySubnet" `
   -SecurityGroupName "myNetworkSecurityGroup" `
   -OpenPorts 3389 `
   -PublicIpAddressName "myPublicIpAddress3" `
   -AvailabilitySetName "myAvailabilitySet" `
   -Credential $cred
```
 

1. Download the installation .zip file of the project from [GitHub](https://github.com/microsoft/AzureScheduledEventsService/archive/master.zip).

1. Connect to **myCollectorVM** and copy the .zip file to the virtual machine and extract all of the files. On your VM, open a PowerShell prompt. Move your prompt into the folder containing `SchService.ps1`, for example: `PS C:\Users\azureuser\AzureScheduledEventsService-master\AzureScheduledEventsService-master\Powershell>`,  and set up the service.

   ```powershell
   .\SchService.ps1 -Setup
   ```

1. Start the service.

   ```powershell
   .\SchService.ps1 -Start
   ```

1. Validate the service status and make sure it is running.

   ```powershell
   .\SchService.ps1 -status  
   ```

   The validation command should return `Running`.

The service will now start polling every 10 seconds for any scheduled events and approve the events to expedite the maintenance.  Freeze, Reboot, Redeploy, and Preempt are the events captured by Schedule events. You can extend the script to trigger some mitigations prior to approving the event.

When any of the above events are captured by Schedule Event service, it will get logged in the Application Event Log Event Status, Event Type, Resources (Virtual machine names) and NotBefore (minimum notice period). You can locate the events with ID 1234 in the Application Event Log.

Once the service is set up and started, it will log events in the Windows Application logs.   To verify this works, restart one of the virtual machines in the availability set and you should see an event being logged in Event viewer in  Windows Logs >  Application log showing the VM restarted. 

![Screenshot of event viewer.](./media/notifications/event-viewer.png)

When events are captured by the Schedule Event service, it will get logged in the application even log with Event Status, Event Type, Resources (VM name) and NotBefore (minimum notice period). You can locate the events with ID 1234 in the Application Event Log.

> [!NOTE] 
> In this example, the virtual machines were are in an availability set, which enabled us to designate a single virtual machine as the collector to listen and route scheduled events to our log analytics works space. If you have standalone virtual machines, you can run the service on every virtual machine, and then connect them individually to your log analytics workspace.
>
> For our set up, we chose Windows, but you can design a similar solution on Linux.

At any point you can stop/remove the Scheduled Event Service by using the switches `–stop` and `–remove`.

## Connect to the Log Analytics Workspace

We now want to connect a Log Analytics Workspace to the collector VM. The Log Analytics workspace acts as a repository and we will configure event log collection to capture the application logs from the collector VM. 

To route the Scheduled Events to the Events Log, which is saved as Application log by our service, you will need to connect your virtual machine to your Log Analytics workspace.  

### Set up data collection

1. Open the Azure portal.

1. In the search bar at the top, type **Log Analytics Workspaces** and select it from the search results.

1. Choose the workspace you created to open its page.

1. Under Settings, select **Agents** and then click **Virtual Machines**.

1. Under the **Windows servers** tab, click **Data Collection Rules**.

1. Enter the **Collect and Deliver** tab and click **Add data source**
    ![Screenshot of the 'Collect and deliver' tab with an empty data source section.](./media/notifications/create-data-collection-rule.png)

1. Under the **Data source** tab, select **Windows Event Logs** from the dropdown.

1. Select the event logs you'd like to collect. Ensure that **ERROR**, **WARNING**, and **INFORMATION** are selected.
    ![Screenshot of the 'Add data source' tab, showing several selected checkboxes.](./media/notifications/add-data-source-windows-event-logs.png)

1. Click **Next : Destination >**

1. Under the **Destination** tab, click **Add destination**.

1. Fill out the **Destination Type**, **Subscription**, and **Destination Details** sections with your collector VM and its subscription's details.
    ![Screenshot of the Destination tab showing type, subscription, and destination details.](./media/notifications/add-destination-details.png)

1. Once you've selected the correct VM, the [Microsoft Monitoring agent](../extensions/oms-windows.md) will be automatically installed on your virtual machine. It will take a few minutes to connect your VM to the workspace and install the extension.


> [!NOTE]
> There is some delay, and it may take up to 10 minutes before the log is available. 


## Creating an alert rule with Azure Monitor 

Once the events are pushed to Log Analytics, you can run the following [query](/azure/azure-monitor/logs/log-analytics-tutorial) to look for the schedule Events.

1. At the top of the page, select **Logs** and paste the following into the text box:

    ```
    Event
    | where EventLog == "Application" and Source contains "AzureScheduledEvents" and RenderedDescription contains "Scheduled" and RenderedDescription contains "EventStatus" 
    | project TimeGenerated, RenderedDescription
    | extend ReqJson= parse_json(RenderedDescription)
    | extend EventId = ReqJson["EventId"]
    ,EventStatus = ReqJson["EventStatus"]
    ,EventType = ReqJson["EventType"]
    ,NotBefore = ReqJson["NotBefore"]
    ,ResourceType = ReqJson["ResourceType"]
    ,Resources = ReqJson["Resources"]
    | project-away RenderedDescription,ReqJson
    ```

1. Select **Save**, and then type `ogQuery` for the name, leave **Query** as the type, type `VMLogs` as the **Category**, and then select **Save**. 

    ![Save the query](./media/notifications/save-query.png)

1. Select **New alert rule**. 
1. In the **Create rule** page, leave `collectorworkspace` as the **Resource**.
1. Under **Condition**, select the entry *Whenever the customer log search is \<login undefined\>*. The **Configure signal logic** page will open.
1. Under **Threshold value**, enter *0* and then select **Done**.
1. Under **Actions**, select **Create action group**. The **Add action group** page will open.
1. In **Action group name**, type *myActionGroup*.
1. In **Short name**, type *myActionGroup*.
1. In **Resource group**, select **myResourceGroupAvailability**.
1. Under Actions, in **ACTION NAME** type **Email**, and then select **Email/SMS/Push/Voice**. The **Email/SMS/Push/Voice** page will open.
1. Select **Email**, type in your e-mail address, then select **OK**.
1. In the **Add action group** page, select **OK**. 
1. In the **Create rule** page, under **ALERT DETAILS**, type *myAlert* for the **Alert rule name**, and then type *Email alert rule* for the **Description**.
1. When you're finished, select **Create alert rule**.
1. Restart one of the VMs in the availability set. Within a few minutes, you should get an e-mail that the alert has been triggered.

To manage your alert rules, go to the resource group, select **Alerts** from the left menu, and then select **Manage alert rules** from the top of the page.

     
## Next steps

To learn more, see the [Scheduled events service](https://github.com/microsoft/AzureScheduledEventsService) page on GitHub.
