---
title: PowerShell Sample - Export/Copy snapshot as VHD to a storage account in different region
description: Azure PowerShell Script Sample -  Export/Copy snapshot as VHD to a storage account in same different region
author: roygara
ms.service: azure-disk-storage
ms.topic: sample
ms.custom: devx-track-azurepowershell
ms.date: 06/05/2017
ms.author: rogarana
# Customer intent: As a cloud administrator, I want to export managed snapshots as VHDs to a storage account in a different region, so that I can ensure robust disaster recovery and maintain backups of my managed disks.
---

# Export/Copy managed snapshots as VHD to a storage account in different region with PowerShell

This script exports a managed snapshot to a storage account in different region. It first generates the SAS URI of the snapshot and then uses it to copy it to a storage account in different region. Use this script to maintain backup of your managed disks in different region for disaster recovery.  

[!INCLUDE [sample-powershell-install](../includes/sample-powershell-install.md)]

[!INCLUDE [quickstarts-free-trial-note](~/reusable-content/ce-skilling/azure/includes/quickstarts-free-trial-note.md)]

 

## Sample script

[!code-powershell[main](../../../powershell_scripts/virtual-machine/copy-snapshot-to-storage-account/copy-snapshot-to-storage-account.ps1 "Copy snapshot")]


## Script explanation

This script uses following commands to generate SAS URI for a managed snapshot and copies the snapshot to a storage account using SAS URI. Each command in the table links to command specific documentation.

| Command | Notes |
|---|---|
| [Grant-AzSnapshotAccess](/powershell/module/az.compute/new-azdisk) | Generates SAS URI for a snapshot that is used to copy it to a storage account. |
| [New-AzStorageContext](/powershell/module/az.storage/new-azstoragecontext) | Creates a storage account context using the account name and key. This context can be used to perform read/write operations on the storage account. |
| [Start-AzStorageBlobCopy](/powershell/module/az.storage/start-azstorageblobcopy) | Copies the underlying VHD of a snapshot to a storage account |

## Next steps

[Create a managed disk from a VHD](virtual-machines-powershell-sample-create-managed-disk-from-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

[Create a virtual machine from a managed disk](./virtual-machines-powershell-sample-create-vm-from-managed-os-disks.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

For more information on the Azure PowerShell module, see [Azure PowerShell documentation](/powershell/azure/).

Additional virtual machine PowerShell script samples can be found in the [Azure Linux VM documentation](../linux/powershell-samples.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
