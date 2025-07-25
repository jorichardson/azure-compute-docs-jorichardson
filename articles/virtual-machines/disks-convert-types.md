---
title: Convert managed disks storage between different disk types
description: How to convert Azure managed disks between the different disks types by using Azure PowerShell, Azure CLI, or the Azure portal.
author: roygara
ms.service: azure-disk-storage
ms.custom: devx-track-azurecli, devx-track-azurepowershell, references_regions
ms.topic: how-to
ms.date: 04/17/2025
ms.author: rogarana
# Customer intent: As a cloud administrator, I want to convert Azure managed disks between different disk types, so that I can optimize storage performance and cost of my environments, according to my workloads' requirements.
---

# Convert the disk type of an Azure managed disk

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows 

There are five disk types of Azure managed disks: Azure Ultra Disks, Premium SSD v2, premium SSD, Standard SSD, and Standard HDD. You can easily switch between Premium SSD, Standard SSD, and Standard HDD based on your performance needs. Premium SSD and Standard SSD are also available with [Zone-redundant storage](disks-redundancy.md#zone-redundant-storage-for-managed-disks). For most cases, you can't yet switch from or to an Ultra Disk, you must [deploy a new one with a snapshot of an existing disk](#migrate-to-premium-ssd-v2-or-ultra-disk-using-snapshots). However, you can switch from existing disks to a Premium SSD v2. See [Convert Premium SSD v2 disks](#convert-premium-ssd-v2-disks) for details.

This functionality isn't supported for unmanaged disks. But you can easily convert an unmanaged disk to a managed disk with [CLI](linux/convert-unmanaged-to-managed-disks.md) or [PowerShell](windows/convert-unmanaged-to-managed-disks.md) to be able to switch between disk types.


## Before you begin

Because conversion requires a restart of the virtual machine (VM), schedule the migration of your disk during a pre-existing maintenance window.

## Restrictions

- You can only change disk type twice per day.
- You can only change the disk type of managed disks. If your disk is unmanaged, convert it to a managed disk with [CLI](linux/convert-unmanaged-to-managed-disks.md) or [PowerShell](windows/convert-unmanaged-to-managed-disks.md) to switch between disk types.
- You can't migrate a disk to Premium SSD v2 if your original disk was created from an Azure Compute Gallery image.

## Change the type of an individual managed disk

For your dev/test workload, you might want a mix of Standard and Premium disks to reduce your costs. You can choose to upgrade only those disks that need better performance. This example shows how to convert a single VM disk from Standard to Premium storage. However, by changing the $storageType variable in this example, you can convert the VM's disks type to standard SSD or standard HDD. To use Premium managed disks, your VM must use a [VM size](sizes.md) that supports Premium storage. You can also use these examples to change a disk from [Locally redundant storage (LRS)](disks-redundancy.md#locally-redundant-storage-for-managed-disks) disk to a [Zone-redundant storage (ZRS)](disks-redundancy.md#zone-redundant-storage-for-managed-disks) disk or vice-versa. This example also shows how to switch to a size that supports Premium storage:

# [Azure PowerShell](#tab/azure-powershell)

[!INCLUDE [managed-disk-premium-ssd-v2-conversion-preview](./includes/managed-disk-premium-ssd-v2-conversion-preview.md)]

```azurepowershell-interactive

$diskName = 'yourDiskName'
# resource group that contains the managed disk
$rgName = 'yourResourceGroupName'
# Choose between Standard_LRS, StandardSSD_LRS, StandardSSD_ZRS, Premium_ZRS, and Premium_LRS based on your scenario
$storageType = 'Premium_LRS'
# Premium capable size 
$size = 'Standard_DS2_v2'

$disk = Get-AzDisk -DiskName $diskName -ResourceGroupName $rgName

# Get parent VM resource
$vmResource = Get-AzResource -ResourceId $disk.ManagedBy

# Stop and deallocate the VM before changing the storage type
Stop-AzVM -ResourceGroupName $vmResource.ResourceGroupName -Name $vmResource.Name -Force

$vm = Get-AzVM -ResourceGroupName $vmResource.ResourceGroupName -Name $vmResource.Name 

# Change the VM size to a size that supports Premium storage
# Skip this step if converting storage from Premium to Standard
$vm.HardwareProfile.VmSize = $size
Update-AzVM -VM $vm -ResourceGroupName $rgName

# Update the storage type
$disk.Sku = [Microsoft.Azure.Management.Compute.Models.DiskSku]::new($storageType)
$disk | Update-AzDisk

Start-AzVM -ResourceGroupName $vm.ResourceGroupName -Name $vm.Name
```

# [Azure CLI](#tab/azure-cli)

[!INCLUDE [managed-disk-premium-ssd-v2-conversion-preview](./includes/managed-disk-premium-ssd-v2-conversion-preview.md)]

 ```azurecli

#resource group that contains the managed disk
rgName='yourResourceGroup'

#Name of your managed disk
diskName='yourManagedDiskName'

#Premium capable size 
#Required only if converting from Standard to Premium
size='Standard_DS2_v2'

#Choose between Standard_LRS, StandardSSD_LRS, StandardSSD_ZRS, Premium_ZRS, and Premium_LRS based on your scenario
sku='Premium_LRS'

#Get the parent VM Id 
vmId=$(az disk show --name $diskName --resource-group $rgName --query managedBy --output tsv)

#Deallocate the VM before changing the size of the VM
az vm deallocate --ids $vmId 

#Change the VM size to a size that supports Premium storage 
#Skip this step if converting storage from Premium to Standard
az vm resize --ids $vmId --size $size

# Update the SKU
az disk update --sku $sku --name $diskName --resource-group $rgName 

az vm start --ids $vmId 
```

# [Portal](#tab/azure-portal)

[!INCLUDE [managed-disk-premium-ssd-v2-conversion-preview](./includes/managed-disk-premium-ssd-v2-conversion-preview.md)]

Follow these steps:

1. Sign in to the [Azure portal](https://portal.azure.com).
1. Select the VM from the list of **Virtual machines**.
1. If the VM isn't stopped, select **Stop** at the top of the VM **Overview** pane, and wait for the VM to stop.
1. In the pane for the VM, select **Disks** from the menu.
1. Select the disk that you want to convert.
1. Select **Size + performance** from the menu.
1. Change the **Account type** from the original disk type to the desired disk type.
1. Select **Save**, and close the disk pane.

The disk type conversion is instantaneous. You can start your VM after the conversion.

---


## Switch all managed disks of a VM from one account to another

This example shows how to convert all of a VM's disks to premium storage. However, by changing the $storageType variable in this example, you can convert the VM's disks type to standard SSD or standard HDD. To use Premium managed disks, your VM must use a [VM size](sizes.md) that supports Premium storage. This example also switches to a size that supports premium storage:

# [Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
# Name of the resource group that contains the VM
$rgName = 'yourResourceGroup'

# Name of the your virtual machine
$vmName = 'yourVM'

# Choose between Standard_LRS, StandardSSD_LRS, StandardSSD_ZRS, Premium_ZRS, Premium_LRS, and PremiumV2_LRS based on your scenario
$storageType = 'Premium_LRS'

# Premium capable size
# Required only if converting storage from Standard to Premium
$size = 'Standard_DS2_v2'

# Stop and deallocate the VM before changing the size
Stop-AzVM -ResourceGroupName $rgName -Name $vmName -Force

$vm = Get-AzVM -Name $vmName -resourceGroupName $rgName

# Change the VM size to a size that supports Premium storage
# Skip this step if converting storage from Premium to Standard
$vm.HardwareProfile.VmSize = $size
Update-AzVM -VM $vm -ResourceGroupName $rgName

# Get all disks in the resource group of the VM
$vmDisks = Get-AzDisk -ResourceGroupName $rgName 

# For disks that belong to the selected VM, convert to Premium storage
foreach ($disk in $vmDisks)
{
	if ($disk.ManagedBy -eq $vm.Id)
	{
		$disk.Sku = [Microsoft.Azure.Management.Compute.Models.DiskSku]::new($storageType)
		$disk | Update-AzDisk
	}
}

Start-AzVM -ResourceGroupName $rgName -Name $vmName
```

# [Azure CLI](#tab/azure-cli)

 ```azurecli

#resource group that contains the virtual machine
rgName='yourResourceGroup'

#Name of the virtual machine
vmName='yourVM'

#Premium capable size 
#Required only if converting from Standard to Premium
size='Standard_DS2_v2'

#Choose between Standard_LRS, StandardSSD_LRS, StandardSSD_ZRS, Premium_ZRS, and Premium_LRS based on your scenario
sku='Premium_LRS'

#Deallocate the VM before changing the size of the VM
az vm deallocate --name $vmName --resource-group $rgName

#Change the VM size to a size that supports Premium storage 
#Skip this step if converting storage from Premium to Standard
az vm resize --resource-group $rgName --name $vmName --size $size

#Update the SKU of all the data disks 
az vm show -n $vmName -g $rgName --query storageProfile.dataDisks[*].managedDisk -o tsv \
 | awk -v sku=$sku '{system("az disk update --sku "sku" --ids "$1)}'

#Update the SKU of the OS disk
az vm show -n $vmName -g $rgName --query storageProfile.osDisk.managedDisk -o tsv \
| awk -v sku=$sku '{system("az disk update --sku "sku" --ids "$1)}'

az vm start --name $vmName --resource-group $rgName
```

# [Portal](#tab/azure-portal)

Use either PowerShell or CLI.

---

## Convert Premium SSD v2 disks 
You can switch existing disks to Premium SSD v2 disks the same way you do for other disk types. Premium SSD v2 disks have some limitations, see the [Premium SSD v2 limitations](disks-deploy-premium-v2.md#limitations) section of their article to learn more.

Switching to Premium SSD v2 disks has some additional limitations:

- You can't switch an OS disk to a Premium SSD v2 disk.
- Existing disks can only be directly switched to 512 sector size Premium SSD v2 disks.
- You can only perform 50 conversions at the same time per subscription per region.
- If your existing disk is a shared disk, detach all VMs before changing to Premium SSD v2.
- If your existing disk is using host caching, [set it to none](#disable-host-caching) before changing to Premium SSD v2.
- If your existing disk is using bursting, [disable it](#disable-bursting) before changing to Premium SSD v2.
- If your existing disk is using double encryption, [switch to one of the single encryption options](#disable-double-encryption) before changing to Premium SSD v2.
- You can't directly switch from a Premium SSD v2 to another disk type. If you want to change a Premium SSD v2 to another disk type, migrate using [snapshots](#migrate-to-premium-ssd-v2-or-ultra-disk-using-snapshots).
- You can't directly switch from Ultra Disks to Premium SSD v2 disks, migrate using [snapshots](#migrate-to-premium-ssd-v2-or-ultra-disk-using-snapshots).
- If your disk has Azure Site Recovery configured on it, disable it before changing to Premium SSD v2.
- If your disk is attached to a VM with Azure Backup enabled, switch to the Enhanced Backup policy before converting to Premium SSD v2.
- If you're using the rest API, use an API version `2020-12-01` or newer for both the Compute Resource Provider and the Disk Resource Provider.
- Until the conversion process from your previous disk type to Premium SSD v2 is completed, the performance of the disk is degraded, and you can't change or rotate customer-managed keys for the disk if they're in use.
    - You can use the following command to check the conversion process, replace `$diskName` and `$resourceGroupName` with your values: `az disk show -n $diskName  -g  $resourceGroupName --query [completionPercent] -o tsv`


> [!NOTE]
> If you're using Azure Backup and you convert a disk to Premium SSD v2, a full snapshot is taken of the new disk. This is a billable event and you'll be charged for that snapshot.

### Disable host caching

If your disk is using host caching, you must disable it before converting to Premium SSD v2. You'll need the LUN of the disk you want to disable host caching on. The following script outputs the name of the disks attached to your VM, and their LUNs. You can use this to identify the LUN of the disk. Replace `yourResourceGroup` and `nameOfYourVM` with your own values, then run the script.

```azurecli
myRG="yourResourceGroup"
myVM="nameOfYourVM"

az vm show -g $myRG -n $myVM --query "[storageProfile.dataDisks[].name, storageProfile.dataDisks[].lun]"
```

Once you've got the disk's LUN, replace `LunHere` with the LUN and run the following command to disable host caching:

```azurecli
lun=LunHere

az vm update --resource-group $myRG --name $myVM --disk-caching $lun=None
```

### Disable bursting

If your disk is using bursting, you must disable it before converting to Premium SSD v2. If you enabled bursting within 12 hours, you have to wait until the 13th hour or later to disable it.

You can use the following command to disable disk bursting: `az disk update --name "yourDiskNameHere" --resource-group "yourRGNameHere" --enable-bursting false`

### Disable double encryption

If your disk is using double encryption, you must disable it before converting to Premium SSD v2. You can use the following command to change your disk from double encryption to encryption at rest with customer-managed keys:

```azurecli
az disk-encryption-set update --name "nameOfYourDiskEncryptionSetHere" --resource-group "yourRGNameHere" --key-url yourKeyURL --source-vault "yourKeyVaultName" --encryption-type EncryptionAtRestWithCustomerKey
```

## Migrate to Premium SSD v2 or Ultra Disk using snapshots

[!INCLUDE [managed-disk-premium-ssd-v2-conversion-preview](./includes/managed-disk-premium-ssd-v2-conversion-preview.md)]

Premium SSD, Standard SSD, and Standard HDD disks support both incremental and full snapshots. However, migration using full snapshots is not supported for these disk types. To create Premium SSD v2 or Ultra disks from a snapshot, you must use incremental snapshots.

Both Premium SSD v2 disks and Ultra Disks have their own set of restrictions. For example, neither can be used as an OS disk, and also aren't available in all regions. See the [Premium SSD v2 limitations](disks-deploy-premium-v2.md#limitations) and [Ultra Disk GA scope and limitations](disks-enable-ultra-ssd.md#ga-scope-and-limitations) sections of their articles for more information.

> [!IMPORTANT]
> When migrating a Standard HDD, Standard SSD, or Premium SSD to either an Ultra Disk or Premium SSD v2, the logical sector size must be 512.

# [Azure PowerShell](#tab/azure-powershell)

The following script migrates a snapshot of a Standard HDD, Standard SSD, or Premium SSD to either an Ultra Disk or a Premium SSD v2.

```PowerShell
$diskName = "yourDiskNameHere"
$resourceGroupName = "yourResourceGroupNameHere"
$snapshotName = "yourDesiredSnapshotNameHere"

# Valid values are 1, 2, or 3
$zone = "yourZoneNumber"

#Provide the size of the disks in GB. It should be greater than the VHD file size.
$diskSize = '128'

#Provide the storage type. Use PremiumV2_LRS or UltraSSD_LRS.
$storageType = 'PremiumV2_LRS'

#Provide the Azure region (e.g. westus) where Managed Disks will be located.
#This location should be same as the snapshot location
#Get all the Azure location using command below:
#Get-AzLocation

#Select the same location as the current disk
#Note that Premium SSD v2 and Ultra Disks are only supported in a select number of regions
$location = 'eastus'

#When migrating a Standard HDD, Standard SSD, or Premium SSD to either an Ultra Disk or Premium SSD v2, the logical sector size must be 512
$logicalSectorSize=512

# Get the disk that you need to backup by creating an incremental snapshot
$yourDisk = Get-AzDisk -DiskName $diskName -ResourceGroupName $resourceGroupName

# Create an incremental snapshot by setting the SourceUri property with the value of the Id property of the disk
$snapshotConfig=New-AzSnapshotConfig -SourceUri $yourDisk.Id -Location $yourDisk.Location -CreateOption Copy -Incremental 
$snapshot = New-AzSnapshot -ResourceGroupName $resourceGroupName -SnapshotName $snapshotName -Snapshot $snapshotConfig

$diskConfig = New-AzDiskConfig -SkuName $storageType -Location $location -CreateOption Copy -SourceResourceId $snapshot.Id -DiskSizeGB $diskSize -LogicalSectorSize $logicalSectorSize -Zone $zone
 
New-AzDisk -Disk $diskConfig -ResourceGroupName $resourceGroupName -DiskName $diskName
```


# [Azure CLI](#tab/azure-cli)

The following script migrates a snapshot of a Standard HDD, Standard SSD, or Premium SSD to either an Ultra Disk or a Premium SSD v2.

```azurecli
# Declare variables
diskName="yourExistingDiskNameHere"
newDiskName="newDiskNameHere"
resourceGroupName="yourResourceGroupNameHere"
snapshotName="desiredSnapshotNameHere"
#Provide the storage type. Use PremiumV2_LRS or UltraSSD_LRS.
storageType=PremiumV2_LRS
#Select the same location as the current disk
#Note that Premium SSD v2 and Ultra Disks are only supported in a select number of regions
location=eastus
#When migrating a Standard HDD, Standard SSD, or Premium SSD to either an Ultra Disk or Premium SSD v2, the logical sector size must be 512
logicalSectorSize=512
#Select an Availability Zone, acceptable values are 1,2, or 3
zone=1
#Enter your disk size in GiB, disk size cannot be smaller than snapshot size
diskSize=128


# Get the disk you need to create snapshot
yourDiskID=$(az disk show -n $diskName -g $resourceGroupName --query "id" --output tsv)

# Create the snapshot
snapshot=$(az snapshot create -g $resourceGroupName -n $snapshotName --source $yourDiskID --incremental true --query "id" -o tsv)

az disk create -g $resourceGroupName -n $newDiskName --source $snapshot --logical-sector-size $logicalSectorSize --location $location --zone $zone --sku $storageType --size-gb $diskSize 

```

# [Portal](#tab/azure-portal)

The following steps assume you already have a snapshot. To learn how to create one, see [Create a snapshot of a virtual hard disk](snapshot-copy-managed-disk.md).

1. Sign in to the [Azure portal](https://portal.azure.com).
1. Select the search bar at the top. Search for and select Disks.
1. Select **+Create** and fill in the details.
1. Make sure the **Region** and **Availability** zone meet the requirements of either your Premium SSD v2 or Ultra Disk.
1. For **Region** select the same region as the snapshot you took.
1. For **Source Type** select **Snapshot**.
1. Select the snapshot you created.
1. Select **Change size** and select either **Premium SSD v2** or **Ultra Disk** for the **Storage Type**.
1. Select the performance and capacity you'd like the disk to have.
1. Continue to the **Advanced** tab.
1. Select **512** for **Logical sector size (bytes)**.
1. Select **Review+Create** and then **Create**.


---

## Next steps

Make a read-only copy of a VM by using a [snapshot](snapshot-copy-managed-disk.md).
