---
title: Expand Unmanaged Disks in Azure
description: Expand the size of unmanaged virtual hard disks attached to a virtual machine by using Azure PowerShell in the Resource Manager deployment model.
author: kirpasingh
manager: roshar
ms.service: azure-disk-storage
ms.collection: windows
ms.topic: how-to
ms.date: 11/17/2021
ms.author: kirpas
ms.custom: devx-track-azurepowershell
# Customer intent: "As a cloud administrator, I want to expand unmanaged virtual hard disks for virtual machines using PowerShell, so that I can accommodate legacy applications and manage disk space effectively."
---
# Expand unmanaged virtual hard disks attached to a virtual machine

This article covers how to expand unmanaged disks. To learn how to expand a managed disk, use either the [Windows](windows/expand-os-disk.md) or [Linux](linux/expand-disks.md) articles.

**Applies to:** :heavy_check_mark: Windows VMs :heavy_check_mark: Linux VMs :heavy_check_mark: Flexible scale sets 

When you create a new virtual machine (VM) in a resource group by deploying an image from [Azure Marketplace](https://azure.microsoft.com/marketplace/), the default operating system (OS) drive is often 127 GB. (Some images have smaller OS disk sizes by default.) It's possible to add data disks to the VM, and the number depends on the version you chose. We recommend that you install applications and CPU-intensive workloads on these addendum disks, but customers often need to expand the OS drive to support specific scenarios:

- To support legacy applications that install components on the OS drive.
- To migrate a physical PC or VM from on-premises with a larger OS drive.

Resizing an OS or data disk of an Azure VM requires the VM to be deallocated.

Shrinking an existing disk isn't supported and can potentially result in data loss.

After you expand the disks, expand the volume within the OS in either [Windows](windows/expand-os-disk.md#expand-the-volume-in-the-operating-system) or [Linux](linux/expand-disks.md#expand-a-disk-partition-and-filesystem) to take advantage of the larger disk.

## Resize an unmanaged disk by using PowerShell

Open your PowerShell Integrated Scripting Environment or PowerShell window in administrative mode and follow these steps:

- Sign in to your Azure account in resource management mode, and select your subscription:

    ```powershell
    Connect-AzAccount
    Select-AzSubscription –SubscriptionName 'my-subscription-name'
    ```

- Set your resource group name and VM names:

    ```powershell
    $rgName = 'my-resource-group-name'
    $vmName = 'my-vm-name'
    ```

- Obtain a reference to your VM:

    ```powershell
    $vm = Get-AzVM -ResourceGroupName $rgName -Name $vmName
    ```

- Stop the VM before you resize the disk:

    ```powershell
    Stop-AzVM -ResourceGroupName $rgName -Name $vmName
    ```

- Set the size of the unmanaged OS disk to the desired value and update the VM:

    ```powershell
    $vm.StorageProfile.OSDisk.DiskSizeGB = 1023
    Update-AzVM -ResourceGroupName $rgName -VM $vm
    ```

    The new size should be greater than the existing disk size. The maximum allowed is 2,048 GB for OS disks. It's possible to expand the virtual hard disk blob beyond that size, but the OS can work with only the first 2,048 GB of space.

- Update the size of any data disks that you want to resize. To expand the first data disk attached to the VM, use a numeric index to obtain a reference to the first attached data disk:

    ```powershell
    $vm.StorageProfile.DataDisks[0].DiskSizeGB = 1023
    ```

    Similarly, you can reference other data disks attached to the VM, either by using an index or the `Name` property of the disk:

    ```powershell
    ($vm.StorageProfile.DataDisks | Where ({$_.Name -eq 'my-second-data-disk'})).DiskSizeGB = 1023
    ```

- Updating the VM might take a few seconds. When the command finishes running, restart the VM:

    ```powershell
    Start-AzVM -ResourceGroupName $rgName -Name $vmName
    ```

## Related content

You can also attach disks by using the [Azure portal](windows\attach-managed-disk-portal.yml).