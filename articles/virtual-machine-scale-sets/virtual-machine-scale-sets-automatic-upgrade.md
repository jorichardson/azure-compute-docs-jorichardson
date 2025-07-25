---
title: Automatic OS image upgrades with Azure Virtual Machine Scale Sets
description: Learn how to automatically upgrade the OS image on virtual machines  in a scale set
author: ju-shim
ms.author: jushiman
ms.topic: concept-article
ms.service: azure-virtual-machine-scale-sets
ms.subservice: automatic-os-upgrade
ms.custom: linux-related-content, devx-track-azurecli, devx-track-azurepowershell
ms.date: 07/08/2024
ms.reviewer: mimckitt
# Customer intent: As a system administrator managing a Virtual Machine Scale Set, I want to enable automatic OS image upgrades so that I can ensure all instances are running the latest OS without manual intervention, maintaining system security and performance.
---
# Azure Virtual Machine Scale Set automatic OS image upgrades

> [!NOTE]
> Many of the steps listed in this document apply to Virtual Machine Scale Sets using Uniform Orchestration mode. We recommend using Flexible Orchestration for new workloads. For more information, see [Orchestration modes for Virtual Machine Scale Sets in Azure](virtual-machine-scale-sets-orchestration-modes.md).

Enabling automatic OS image upgrades on your scale set helps ease update management by safely and automatically upgrading the OS disk for all instances in the scale set.

Automatic OS upgrade has the following characteristics:

- Once configured, the latest OS image published by image publishers is automatically applied to the scale set without user intervention.
- Upgrades batches of instances in a rolling manner each time a new image is published by the publisher.
- Integrates with application health probes and [Application Health extension](virtual-machine-scale-sets-health-extension.md).
- Works for all virtual machine sizes, and for both Windows and Linux images including custom images through [Azure Compute Gallery](../virtual-machines/shared-image-galleries.md).
- You can opt out of automatic upgrades at any time (OS Upgrades can be initiated manually as well).
- The OS Disk of a virtual machine is replaced with the new OS Disk created with latest image version. Configured extensions and custom data scripts are run, while persisted data disks are retained.
- [Extension sequencing](virtual-machine-scale-sets-extension-sequencing.md) is supported.
- Can be enabled on a scale set of any size.

> [!NOTE]
>Before enabling automatic OS image upgrades, check [requirements section](#requirements-for-configuring-automatic-os-image-upgrade) of this documentation.

## How does automatic OS image upgrade work?

An upgrade works by replacing the OS disk of a virtual machine with a new disk created using the  image version. Any configured extensions and custom data scripts are run on the OS disk, while data disks are retained. To minimize the application downtime, upgrades take place in batches, with no more than 20% of the scale set upgrading at any time.

You must integrate an Azure Load Balancer application health probe or [Application Health extension](virtual-machine-scale-sets-health-extension.md) to track the health of the application after an upgrade. This allows the platform to validate the virtual machine health to ensure updates are applied in a safe manner. We recommend incorporating an application heartbeat to validate upgrade success.

### Availability-first Updates
The availability-first model for platform orchestrated updates described below ensures that availability configurations in Azure are respected across multiple availability levels.

**Across regions:**
- An update moves across Azure globally in a phased manner to prevent Azure-wide deployment failures.
- A 'phase' can have one or more regions, and an update moves across phases only if eligible virtual machines in the previous phase update successfully.
- Geo-paired regions won't be updated concurrently and cannot be in the same regional phase.
- The success of an update is measured by tracking the health of a virtual machine post update.

**Within a region:**
- Virtual machines in different Availability Zones are not updated concurrently with the same update.

**Within a 'set':**
- All virtual machines in a common scale set are not updated concurrently.
- Virtual machines in a common Virtual Machine Scale Set are grouped in batches and updated within Update Domain boundaries as described below.

The platform orchestrated updates process is followed for rolling out supported OS platform image upgrades every month. For custom images through Azure Compute Gallery, an image upgrade is only kicked off for a particular Azure region when the new image is published and [replicated](../virtual-machines/azure-compute-gallery.md#replication) to the region of that scale set.

### Upgrading virtual machines in a scale set

The region of a scale set becomes eligible to get image upgrades either through the availability-first process for platform images or replicating new custom image versions for Share Image Gallery. The image upgrade is then applied to an individual scale set in a batched manner as follows:

1. Before you begin the upgrade process, the orchestrator ensures that no more than 20% of instances in the entire scale set are unhealthy (for any reason).
2. The upgrade orchestrator identifies the batch of virtual machines to upgrade, with any one batch having a maximum of 20% of the total instance count, subject to a minimum batch size of one virtual machine. There is no minimum scale set size requirement and scale sets with 5 or fewer instances have 1 virtual machine  per upgrade batch (minimum batch size).
3. The OS disk of every virtual machine in the selected upgrade batch is replaced with a new OS disk created from the  image. All specified extensions and configurations in the scale set model are applied to the upgraded instance.
4. For scale sets with configured application health probes or Application Health extension, the upgrade waits up to 5 minutes for the instance to become healthy, before moving on to upgrade the next batch. If an instance does not recover its health in 5 minutes after an upgrade, then by default the previous OS disk for the instance is restored.
5. The upgrade orchestrator also tracks the percentage of instances that become unhealthy post an upgrade. The upgrade stops if more than 20% of upgraded instances become unhealthy during the upgrade process.
6. The above process continues until all instances in the scale set have been upgraded.

The scale set OS upgrade orchestrator checks for the overall scale set health before upgrading every batch. While you're upgrading a batch, there could be other concurrent planned or unplanned maintenance activities that could impact the health of your scale set instances. In such cases if more than 20% of the scale set's instances become unhealthy, then the scale set upgrade stops at the end of current batch.

To modify the default settings associated with Rolling Upgrades, review Azure's [Rolling Upgrade Policy](/rest/api/compute/virtual-machine-scale-sets/create-or-update?tabs=HTTP#rollingupgradepolicy).

> [!NOTE]
>Automatic OS upgrade does not upgrade the reference image Sku on the scale set. To change the Sku (such as Ubuntu 18.04-LTS to 20.04-LTS), you must update the [scale set model](virtual-machine-scale-sets-upgrade-scale-set.md#the-scale-set-model) directly with the desired image Sku. Image publisher and offer can't be changed for an existing scale set.

## OS image upgrade versus reimage

Both **OS Image Upgrade** and **[Reimage](/rest/api/compute/virtual-machine-scale-sets/reimage)** are methods used to update virtual machines within a scale set, but they serve different purposes and have distinct impacts.

OS image upgrade involves updating the underlying operating system image that is used to create new instances in a scale set. When you perform an OS image upgrade, Azure creates new virtual machines with the updated OS image and gradually replace the old virtual machines in the scale set with the new ones. This process is typically performed in stages to ensure high availability. OS image upgrades are a non-disruptive way to apply updates or changes to the underlying OS of the virtual machines in a scale set. Existing virtual machines are not affected until they are replaced with the new instances.

Reimaging a virtual machine in a scale set is a more immediate and disruptive action. When you choose to reimage a virtual machine , Azure stops the selected virtual machine, perform the reimage operation, and then restart the virtual machine using the same OS image. This effectively reinstalls the OS on that specific virtual machine. Reimaging is typically used when you need to troubleshoot or reset a specific virtual machine due to issues with that instance.

**Key differences:**
- OS Image Upgrade is a gradual and non-disruptive process that updates the OS image for the entire Virtual Machine Scale Set over time, ensuring minimal impact on running workloads.
- Reimage is a more immediate and disruptive action that affects only the selected virtual machine, stopping it temporarily and reinstalling the OS.

**When to use each method:**
- Use OS Image Upgrade when you want to update the OS image for the entire scale set while maintaining high availability.
- Use Reimage when you need to troubleshoot or reset a specific virtual machine within the virtual Machine Scale Set.

It's essential to carefully plan and choose the appropriate method based on your specific requirements to minimize any disruption to your applications and services running in a Virtual Machine Scale Set.

## Supported OS images
Only certain OS platform images are currently supported. Custom images [are supported](virtual-machine-scale-sets-automatic-upgrade.md#automatic-os-image-upgrade-for-custom-images) if the scale set uses custom images through [Azure Compute Gallery](../virtual-machines/shared-image-galleries.md).

The following platform SKUs are currently supported (and more are added periodically):

| Publisher               | OS Offer      |  Sku               |
|-------------------------|---------------|--------------------|
| Canonical               | UbuntuServer  | 18.04-LTS          |
| Canonical               | UbuntuServer  | 18_04-LTS-Gen2          |
| Canonical               | 0001-com-ubuntu-server-focal  | 20_04-LTS          |
| Canonical               | 0001-com-ubuntu-server-focal  | 20_04-LTS-Gen2     |
| Canonical               | 0001-com-ubuntu-server-jammy  | 22_04-LTS    |
| Canonical               | 0001-com-ubuntu-server-jammy  | 22_04-LTS-Gen2    |
| MicrosoftCblMariner     | Cbl-Mariner   | cbl-mariner-1      |
| MicrosoftCblMariner     | Cbl-Mariner   | 1-Gen2             |
| MicrosoftCblMariner     | Cbl-Mariner   | cbl-mariner-2
| MicrosoftCblMariner     | Cbl-Mariner   | cbl-mariner-2-Gen2 |
| MicrosoftSqlServer      | Sql2017-ws2019| enterprise |
| MicrosoftWindowsServer  | WindowsServer | 2012-R2-Datacenter |
| MicrosoftWindowsServer  | WindowsServer | 2016-Datacenter    |
| MicrosoftWindowsServer  | WindowsServer | 2016-Datacenter-gensecond    |
| MicrosoftWindowsServer  | WindowsServer | 2016-Datacenter-gs        |
| MicrosoftWindowsServer  | WindowsServer | 2016-Datacenter-smalldisk |
| MicrosoftWindowsServer  | WindowsServer | 2016-Datacenter-with-Containers |
| MicrosoftWindowsServer  | WindowsServer | 2016-Datacenter-with-containers-gs |
| MicrosoftWindowsServer  | WindowsServer | 2019-Datacenter |
| MicrosoftWindowsServer  | WindowsServer | 2019-Datacenter-Core |
| MicrosoftWindowsServer  | WindowsServer | 2019-Datacenter-Core-with-Containers |
| MicrosoftWindowsServer  | WindowsServer | 2019-Datacenter-gensecond |
| MicrosoftWindowsServer  | WindowsServer | 2019-Datacenter-gs |
| MicrosoftWindowsServer  | WindowsServer | 2019-Datacenter-smalldisk |
| MicrosoftWindowsServer  | WindowsServer | 2019-Datacenter-with-Containers |
| MicrosoftWindowsServer  | WindowsServer | 2019-Datacenter-with-Containers-gs |
| MicrosoftWindowsServer  | WindowsServer | 2022-Datacenter |
| MicrosoftWindowsServer  | WindowsServer | 2022-Datacenter-smalldisk |
| MicrosoftWindowsServer  | WindowsServer | 2022-Datacenter-smalldisk-g2 |
| MicrosoftWindowsServer  | WindowsServer | 2022-Datacenter-core |
| MicrosoftWindowsServer  | WindowsServer | 2022-Datacenter-core-smalldisk |
| MicrosoftWindowsServer  | WindowsServer | 2022-Datacenter-g2 |
| MicrosoftWindowsServer  | WindowsServer | Datacenter-core-20h2-with-containers-smalldisk-gs |
| MicrosoftWindowsServer  | WindowsServer | 2022-Datacenter-azure-edition |
| MicrosoftWindowsServer  | WindowsServer | 2022-Datacenter-azure-edition-smalldisk |
| Mirantis  | Windows_with_Mirantis_Container_Runtime_2019 | win_2019_mcr_23_0 |
| Mirantis  | Windows_with_Mirantis_Container_Runtime_2019 | win_2019_mcr_23_0_gen2 |


## Requirements for configuring automatic OS image upgrade

- The *version* property of the image must be set to latest.
- Must use application health probes or [Application Health extension](virtual-machine-scale-sets-health-extension.md) for non-Service Fabric scale sets. For Service Fabric requirements, see [Service Fabric requirement](#service-fabric-requirements).
- Use Compute API version 2018-10-01 or higher.
- Ensure that external resources specified in the scale set model are available and updated. Examples include SAS URI for bootstrapping payload in virtual machine extension properties, payload in storage account, reference to secrets in the model, and more.
- For scale sets using Windows virtual machines, starting with Compute API version 2019-03-01, the *virtualMachineProfile.osProfile.windowsConfiguration.enableAutomaticUpdates* property must set to *false* in the scale set model definition. The *enableAutomaticUpdates* property enables in-VM patching where "Windows Update" applies operating system patches without replacing the OS disk. With automatic OS image upgrades enabled on your scale set, which can be done by setting the *automaticOSUpgradePolicy.enableAutomaticOSUpgrade* to *true*, an extra patching process through Windows Update is not required.
- The [patch orchestration mode](../virtual-machines/automatic-vm-guest-patching.md#patch-orchestration-modes) must _not_ be set to `AutomaticByPlatform` in the scale set model definition. With automatic OS image upgrades enabled on your scale set, a platform orchestration patching process is not required. 

> [!NOTE]
> After an OS disk is replaced through reimage or upgrade, the attached data disks may have their drive letters reassigned. To retain the same drive letters for attached disks, it is suggested to use a custom boot script.


### Service Fabric requirements

If you are using Service Fabric, ensure the following conditions are met:
-	Service Fabric [durability level](../service-fabric/service-fabric-cluster-capacity.md#durability-characteristics-of-the-cluster) is Silver or Gold. If Service Fabric durability is Bronze, only Stateless-only node types support automatic OS image upgrades).
-	The Service Fabric extension on the scale set model definition must have TypeHandlerVersion 1.1 or above.
-	Durability level should be the same at the Service Fabric cluster and Service Fabric extension on the scale set model definition.
- More health probes or use of application health extension is not required for Silver or Gold durability. Bronze durability with Stateless-only node types requires an additional health probe.
- The property *virtualMachineProfile.osProfile.windowsConfiguration.enableAutomaticUpdates* property must set to *false* in the scale set model definition. The *enableAutomaticUpdates* property enables in-VM patching using "Windows Update" and is not supported on Service Fabric scale sets. You should use the *automaticOSUpgradePolicy.enableAutomaticOSUpgrade* property instead.

Ensure that durability settings are not mismatched on the Service Fabric cluster and Service Fabric extension, as a mismatch results in upgrade errors. Durability levels can be modified per the guidelines outlined on [this page](../service-fabric/service-fabric-cluster-capacity.md#changing-durability-levels).


## Automatic OS image upgrade for custom images

Automatic OS image upgrade is supported for custom images deployed through [Azure Compute Gallery](../virtual-machines/shared-image-galleries.md). Other custom images are not supported for automatic OS image upgrades.

### Additional requirements for custom images
- The setup and configuration process for automatic OS image upgrade is the same for all scale sets as detailed in the [configuration section](virtual-machine-scale-sets-automatic-upgrade.md#configure-automatic-os-image-upgrade) of this page.
- Scale sets instances configured for automatic OS image upgrades are upgraded to the  version of the Azure Compute Gallery image when a new version of the image is published and [replicated](../virtual-machines/azure-compute-gallery.md#replication) to the region of that scale set. If the new image is not replicated to the region where the scale is deployed, the scale set instances are not upgraded to the version. Regional image replication allows you to control the rollout of the new image for your scale sets.
- The new image version should not be excluded from the  version for that gallery image. Image versions excluded from the gallery image's  version are not rolled out to the scale set through automatic OS image upgrade.

> [!NOTE]
> It can take up to 3 hours for a scale set to trigger the first image upgrade rollout after the scale set is first configured for automatic OS upgrades due to certain factors such as Maintenance Windows or other restrictions. Customers on the latest image may not get an upgrade until a new image is available.


## Configure automatic OS image upgrade
To configure automatic OS image upgrade, ensure that the *automaticOSUpgradePolicy.enableAutomaticOSUpgrade* property is set to *true* in the scale set model definition.

> [!NOTE]
> **Upgrade Policy mode** and **Automatic OS Upgrade Policy** are separate settings and control different aspects of the scale set. When there are changes in the scale set template, the Upgrade Policy `mode` determines what happens to existing instances in the scale set. However, Automatic OS Upgrade Policy `enableAutomaticOSUpgrade` is specific to the OS image and tracks changes the image publisher has made and determines what happens when there is an update to the image.

> [!Note]
> If `enableAutomaticOSUpgrade` is set to *true*, `enableAutomaticUpdates` is automatically set to *false* and cannot be set to *true*.

### REST API
The following example describes how to set automatic OS upgrades on a scale set model:

```
PUT or PATCH on `/subscriptions/subscription_id/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet?api-version=2021-03-01`
```

```json
{
  "properties": {
    "upgradePolicy": {
      "automaticOSUpgradePolicy": {
        "enableAutomaticOSUpgrade":  true
      }
    }
  }
}
```

### Azure PowerShell
Use the [New-AzVmss](/powershell/module/az.compute//new-azvmss) cmdlet to configure automatic OS image upgrades for your scale set during provisioning. The following example configures automatic upgrades for the scale set named *myScaleSet* in the resource group named *myResourceGroup*:

```azurepowershell-interactive
New-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -AutomaticOSUpgrade $true
```

Use the [Update-AzVmss](/powershell/module/az.compute/update-azvmss) cmdlet to configure automatic OS image upgrades for your existing scale set. The following example configures automatic upgrades for the scale set named *myScaleSet* in the resource group named *myResourceGroup*:

```azurepowershell-interactive
Update-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -AutomaticOSUpgrade $true
```

### Azure CLI 2.0
Use [az vmss create](/cli/azure/vmss#az-vmss-create) to configure automatic OS image upgrades for your scale set during provisioning. Use Azure CLI 2.0.47 or above. The following example configures automatic upgrades for the scale set named *myScaleSet* in the resource group named *myResourceGroup*:

```azurecli-interactive
az vmss create --name myScaleSet --resource-group myResourceGroup --enable-auto-os-upgrade true --upgrade-policy-mode Rolling
```

Use [az vmss update](/cli/azure/vmss#az-vmss-update) to configure automatic OS image upgrades for your existing scale set. Use Azure CLI 2.0.47 or above. The following example configures automatic upgrades for the scale set named *myScaleSet* in the resource group named *myResourceGroup*:

```azurecli-interactive
az vmss update --name myScaleSet --resource-group myResourceGroup --enable-auto-os-upgrade true --upgrade-policy-mode Rolling
```

> [!NOTE]
>After configuring automatic OS image upgrades for your scale set, you must also bring the scale set virtual machines to the latest scale set model if your scale set uses the 'Manual' [upgrade policy](virtual-machine-scale-sets-upgrade-policy.md).

### ARM templates
The following example describes how to set automatic OS upgrades on a scale set model via Azure Resource Manager templates (ARM templates):

```json
"properties": {
   "upgradePolicy": {
     "mode": "Automatic",
     "RollingUpgradePolicy": {
         "BatchInstancePercent": 20,
         "MaxUnhealthyInstancePercent": 25,
         "MaxUnhealthyUpgradedInstancePercent": 25,
         "PauseTimeBetweenBatches": "PT0S"
     },
    "automaticOSUpgradePolicy": {
      "enableAutomaticOSUpgrade": true,
        "useRollingUpgradePolicy": true,
        "disableAutomaticRollback": false
    }
  },
  },
"imagePublisher": {
   "type": "string",
   "defaultValue": "MicrosoftWindowsServer"
 },
 "imageOffer": {
   "type": "string",
   "defaultValue": "WindowsServer"
 },
 "imageSku": {
   "type": "string",
   "defaultValue": "2022-datacenter"
 },
 "imageOSVersion": {
   "type": "string",
   "defaultValue": "latest"
 }

```

### Bicep
The following example describes how to set automatic OS upgrades on a scale set model via Bicep:

```json
properties: {
    overprovision: overProvision
    upgradePolicy: {
      mode: 'Automatic'
      automaticOSUpgradePolicy: {
        enableAutomaticOSUpgrade: true
      }
    }
}
```

## Using Application Health Extension

During an OS Upgrade, virtual machines in a scale set are upgraded one batch at a time. The upgrade should continue only if the customer application is healthy on the upgraded virtual machines. We recommend that the application provides health signals to the scale set OS Upgrade engine. By default, during OS Upgrades the platform considers virtual machine power state and extension provisioning state to determine if a virtual machine is healthy after an upgrade. During the OS Upgrade of a virtual machine, the OS disk on a virtual machine is replaced with a new disk based on latest image version. After the OS Upgrade has completed, the configured extensions are run on these virtual machines. The application is considered healthy only when all the extensions on the instance are successfully provisioned.

A scale set can optionally be configured with Application Health Probes to provide the platform with accurate information on the ongoing state of the application. Application Health Probes are Custom Load Balancer Probes that are used as a health signal. The application running on a scale set virtual machine can respond to external HTTP or TCP requests indicating whether it's healthy. For more information on how Custom Load Balancer Probes work, see to [Understand load balancer probes](/azure/load-balancer/load-balancer-custom-probe-overview). Application Health Probes are not supported for Service Fabric scale sets. Non-Service Fabric scale sets require either Load Balancer application health probes or [Application Health extension](virtual-machine-scale-sets-health-extension.md).

If the scale set is configured to use multiple placement groups, probes using a [Standard Load Balancer](/azure/load-balancer/load-balancer-overview) need to be used.

> [!NOTE]
> Only one source of health monitoring can be used for a Virtual Machine Scale Set, either an Application Health Extension or a Health Probe. If you have both options enabled, you need to remove one before using orchestration services like Instance Repairs or Automatic OS Upgrades.

### Configuring a Custom Load Balancer Probe as Application Health Probe on a scale set

As a best practice, create a load balancer probe explicitly for scale set health. The same endpoint for an existing HTTP probe or TCP probe can be used, but a health probe could require different behavior from a traditional load-balancer probe. For example, a traditional load balancer probe could return unhealthy if the load on the instance is too high, but that would not be appropriate for determining the instance health during an automatic OS upgrade. Configure the probe to have a high probing rate of less than two minutes.

The load-balancer probe can be referenced in the *networkProfile* of the scale set and can be associated with either an internal or public facing load-balancer as follows:

```json
"networkProfile": {
  "healthProbe" : {
    "id": "[concat(variables('lbId'), '/probes/', variables('sshProbeName'))]"
  },
  "networkInterfaceConfigurations":
  ...
}
```

> [!NOTE]
> When using Automatic OS Upgrades with Service Fabric, the new OS image is rolled out Update Domain by Update Domain to maintain high availability of the services running in Service Fabric. To utilize Automatic OS Upgrades in Service Fabric your cluster node type must be configured to use the Silver Durability Tier or higher. For Bronze Durability tier, automatic OS image upgrade is only supported for Stateless node types. For more information on the durability characteristics of Service Fabric clusters, see [this documentation](../service-fabric/service-fabric-cluster-capacity.md#durability-characteristics-of-the-cluster).

## Using Application Health extension

The Application Health extension is deployed inside a Virtual Machine Scale Set instance and reports on virtual machine health from inside the scale set instance. You can configure the extension to probe on an application endpoint and update the status of the application on that instance. This instance status is checked by Azure to determine whether an instance is eligible for upgrade operations.

As the extension reports health from within a virtual machine, the extension can be used in situations where external probes such as Application Health Probes (that utilize custom Azure Load Balancer [probes](/azure/load-balancer/load-balancer-custom-probe-overview)) can’t be used.

There are multiple ways of deploying the Application Health extension to your scale sets as detailed in the examples in [this article](virtual-machine-scale-sets-health-extension.md#deploy-the-application-health-extension).

> [!NOTE]
> Only one source of health monitoring can be used for a Virtual Machine Scale Set, either an Application Health Extension or a Health Probe. If you have both options enabled, you need to remove one before using orchestration services like Instance Repairs or Automatic OS Upgrades.


## Configure custom metrics for rolling upgrades on Virtual Machine Scale Sets (Preview)

> [!NOTE]
>**Custom metrics for rolling upgrades on Virtual Machine Scale Sets is currently in preview.** Previews are made available to you on the condition that you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of these features may change prior to general availability (GA).

Custom metrics for rolling upgrades enables you to utilize the [application health extension](virtual-machine-scale-sets-health-extension.md) to emit custom metrics to your Virtual Machine Scale Set. These custom metrics can be used to tell the scale set the order in which virtual machines should be updated when a rolling upgrade is triggered. The custom metrics can also inform your scale set when an upgrade should be skipped on a specific instance. This allows you to have more control over the ordering and the update process itself. 

Custom metrics can be used in combination with other rolling upgrade functionality such as [automatic OS upgrades](virtual-machine-scale-sets-automatic-upgrade.md), [automatic extension upgrades](../virtual-machines/automatic-extension-upgrade.md) and [MaxSurge rolling upgrades](virtual-machine-scale-sets-maxsurge.md). 

For more information, see [custom metrics for rolling upgrades on Virtual Machine Scale Sets](virtual-machine-scale-sets-rolling-upgrade-custom-metrics.md)

## Get the history of automatic OS image upgrades
You can check the history of the most recent OS upgrade performed on your scale set with Azure PowerShell, Azure CLI 2.0, or the REST APIs. You can get history for the last five OS upgrade attempts within the past two months.

### Keep credentials up to date

If your scale set uses any credentials to access external resources, such as a virtual machine extension configured to use a SAS token for storage account, then ensure that the credentials are updated. If any credentials including certificates and tokens have expired, the upgrade fails and the first batch of virtual machines are left in a failed state.

The recommended steps to recover virtual machines and re-enable automatic OS upgrade if there's a resource authentication failure are:

* Regenerate the token (or any other credentials) passed into your extensions.
* Ensure that any credential used from inside the virtual machine to talk to external entities is up to date.
* Update extensions in the scale set model with any new tokens.
* Deploy the updated scale set, which updates all virtual machines including the failed ones.

### REST API
The following example uses [REST API](/rest/api/compute/virtualmachinescalesets/getosupgradehistory) to check the status for the scale set named *myScaleSet* in the resource group named *myResourceGroup*:

```
GET on `/subscriptions/subscription_id/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/osUpgradeHistory?api-version=2021-03-01`
```

The GET call returns properties similar to the following example output:

```json
{
	"value": [
		{
			"properties": {
        "runningStatus": {
          "code": "RollingForward",
          "startTime": "2018-07-24T17:46:06.1248429+00:00",
          "completedTime": "2018-04-21T12:29:25.0511245+00:00"
        },
        "progress": {
          "successfulInstanceCount": 16,
          "failedInstanceCount": 0,
          "inProgressInstanceCount": 4,
          "pendingInstanceCount": 0
        },
        "startedBy": "Platform",
        "targetImageReference": {
          "publisher": "MicrosoftWindowsServer",
          "offer": "WindowsServer",
          "sku": "2016-Datacenter",
          "version": "2016.127.20180613"
        },
        "rollbackInfo": {
          "successfullyRolledbackInstanceCount": 0,
          "failedRolledbackInstanceCount": 0
        }
      },
      "type": "Microsoft.Compute/virtualMachineScaleSets/rollingUpgrades",
      "location": "westeurope"
    }
  ]
}
```

### Azure PowerShell
Use the [Get-AzVmss](/powershell/module/az.compute/get-azvmss) cmdlet to check OS upgrade history for your scale set. The following example details how you review the OS upgrade status for a scale set named *myScaleSet* in the resource group named *myResourceGroup*:

```azurepowershell-interactive
Get-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -OSUpgradeHistory
```

### Azure CLI 2.0
Use [az vmss get-os-upgrade-history](/cli/azure/vmss#az-vmss-get-os-upgrade-history) to check the OS upgrade history for your scale set. Use Azure CLI 2.0.47 or above. The following example details how you review the OS upgrade status for a scale set named *myScaleSet* in the resource group named *myResourceGroup*:

```azurecli-interactive
az vmss get-os-upgrade-history --resource-group myResourceGroup --name myScaleSet
```

## How to get the latest version of a platform OS image?

You can get the available image versions for automatic OS upgrade supported SKUs using the below examples:

### REST API
```
GET on `/subscriptions/subscription_id/providers/Microsoft.Compute/locations/{location}/publishers/{publisherName}/artifacttypes/vmimage/offers/{offer}/skus/{skus}/versions?api-version=2021-03-01`
```

### Azure PowerShell
```azurepowershell-interactive
Get-AzVmImage -Location "westus" -PublisherName "Canonical" -offer "0001-com-ubuntu-server-jammy" -sku "22_04-lts"
```

### Azure CLI 2.0
```azurecli-interactive
az vm image list --location "westus" --publisher "Canonical" --offer "0001-com-ubuntu-server-jammy" --sku "22_04-lts" --all
```

## Manually trigger OS image upgrades
With automatic OS image upgrade enabled on your scale set, you don't need to manually trigger image updates on your scale set. The OS upgrade orchestrator automatically applies the latest available image version to your scale set instances without any manual intervention.

For specific cases where you don't want to wait for the orchestrator to apply the latest image, you can trigger an OS image upgrade manually using the below examples.

> [!NOTE]
> Manual trigger of OS image upgrades does not provide automatic rollback capabilities. If an instance does not recover its health after an upgrade operation, its previous OS disk can't be restored.

### REST API
Use the [Start OS Upgrade](/rest/api/compute/virtualmachinescalesetrollingupgrades/startosupgrade) API call to start a rolling upgrade to move all Virtual Machine Scale Set instances to the latest available image OS version. Instances that are already running the latest available OS version are not affected. The following example details how you can start a rolling OS upgrade on a scale set named *myScaleSet* in the resource group named *myResourceGroup*:

```
POST on `/subscriptions/subscription_id/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachineScaleSets/myScaleSet/osRollingUpgrade?api-version=2021-03-01`
```

### Azure PowerShell
Use the [Start-AzVmssRollingOSUpgrade](/powershell/module/az.compute/Start-AzVmssRollingOSUpgrade) cmdlet to check OS upgrade history for your scale set. The following example details how you can start a rolling OS upgrade on a scale set named *myScaleSet* in the resource group named *myResourceGroup*:

```azurepowershell-interactive
Start-AzVmssRollingOSUpgrade -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet"
```

### Azure CLI 2.0
Use [az vmss rolling-upgrade start](/cli/azure/vmss/rolling-upgrade#az-vmss-rolling-upgrade-start) to check the OS upgrade history for your scale set. Use Azure CLI 2.0.47 or above. The following example details how you can start a rolling OS upgrade on a scale set named *myScaleSet* in the resource group named *myResourceGroup*:

```azurecli-interactive
az vmss rolling-upgrade start --resource-group "myResourceGroup" --name "myScaleSet" --subscription "subscriptionId"
```

## Leverage Activity Logs for Upgrade Notifications and Insights

[Activity Log](/azure/azure-monitor/essentials/activity-log?tabs=powershell) is a subscription log that provides insight into subscription-level events that have occurred in Azure. Customers are able to:
* See events related to operations performed on their resources in Azure portal
* Create action groups to tune notification methods like email, sms, webhooks, or ITSM
*  Set up suitable alerts using different criteria using Portal, ARM resource template, PowerShell or CLI to be sent to action groups

Customers receive three types of notifications related to Automatic OS Upgrade operation:
* Submission of upgrade request for a particular resource
* Outcome of submission request along with any error details
* Outcome of upgrade completion along with any error details

### Setting up Action Groups for Activity log alerts

An [action group](/azure/azure-monitor/alerts/action-groups) is a collection of notification preferences defined by the owner of an Azure subscription. Azure Monitor and Service Health alerts use action groups to notify users that an alert has been triggered. 

Action groups can be created and managed using: 
* [ARM Resource Manager](/azure/azure-monitor/alerts/action-groups#create-an-action-group-with-a-resource-manager-template)
* [Portal](/azure/azure-monitor/alerts/action-groups#create-an-action-group-in-the-azure-portal) 
* PowerShell:
  *  [New-AzActionGroup](/powershell/module/az.monitor/new-azactiongroup)  
  *  [Get-AzActionGroup](/powershell/module/az.monitor/get-azactiongroup)
  *  [Remove-AzActionGroup](/powershell/module/az.monitor/remove-azactiongroup)
* [CLI](/cli/azure/monitor/action-group#az-monitor-action-group-create)

Customers can set up the following using action groups:
* [SMS and/or Email notifications](/azure/azure-monitor/alerts/action-groups#email-azure-resource-manager)
* [Webhooks](/azure/azure-monitor/alerts/action-groups#webhook) - Customers can attach webhooks to their automation runbooks and configure their action groups to trigger the runbooks. You can start a runbook from a [webhook](/azure/automation/automation-webhooks)
* [ITSM Connections](/azure/azure-monitor/alerts/itsmc-overview)

## Investigate and Resolve Auto Upgrade Errors

The platform can return errors on virtual machines while performing Automatic Image Upgrade with Rolling Upgrade policy. The [Get Instance View](/rest/api/compute/virtual-machine-scale-sets/get-instance-view) of a virtual machine contains the detailed error message to investigate and resolve an error. The [Rolling Upgrades - Get Latest](/rest/api/compute/virtual-machine-scale-sets/get) can provide more details on rolling upgrade configuration and status. The [Get OS Upgrade History](/rest/api/compute/virtual-machine-scale-sets/get-os-upgrade-history) provides details on the last image upgrade operation on the scale set. Below are the topmost errors that can result in Rolling Upgrades.

**RollingUpgradeInProgressWithFailedUpgradedVMs**
- Error is triggered for a virtual machine failure.
- The detailed error message mentions whether the rollout continues/pauses based on the configured threshold.

**MaxUnhealthyUpgradedInstancePercentExceededInRollingUpgrade**
- Error is triggered when the percent of upgraded virtual machines exceed the max threshold allowed for unhealthy virtual machines.
- The detailed error message aggregates the most common error contributing to the unhealthy virtual machines. See [MaxUnhealthyUpgradedInstancePercent](/rest/api/compute/virtual-machine-scale-sets/create-or-update?tabs=HTTP#rollingupgradepolicy).

**MaxUnhealthyInstancePercentExceededInRollingUpgrade**
- Error is triggered when the percent of unhealthy virtual machines exceed the max threshold allowed for unhealthy virtual machines during an upgrade.
- The detailed error message displays the current unhealthy percent and the configured allowable unhealthy virtual machine percentage. See [maxUnhealthyInstancePercent](/rest/api/compute/virtual-machine-scale-sets/create-or-update?tabs=HTTP#rollingupgradepolicy).

**MaxUnhealthyInstancePercentExceededBeforeRollingUpgrade**
- Error is triggered when the percent of unhealthy virtual machines exceed the max threshold allowed for unhealthy virtual machines before an upgrade takes place.
- The detailed error message displays the current unhealthy percent and the configured allowable unhealthy virtual machine percentage. See [maxUnhealthyInstancePercent](/rest/api/compute/virtual-machine-scale-sets/create-or-update?tabs=HTTP#rollingupgradepolicy).

**InternalExecutionError**
- Error is triggered when an unhandled, unformatted or unexpected occurs during execution.
- The detailed error message displays the cause of the error.

**RollingUpgradeTimeoutError**
- Error is triggered when the rolling upgrade process has timed out.
- The detailed error message displays the length of time the system timed out after attempting to update.

## Next steps
> [!div class="nextstepaction"]
> [Learn about the Application Health Extension](../virtual-machine-scale-sets/virtual-machine-scale-sets-health-extension.md)
