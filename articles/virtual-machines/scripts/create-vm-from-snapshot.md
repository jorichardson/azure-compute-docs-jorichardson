---
title: Create a VM from a snapshot - CLI Sample
description: Azure CLI Script Sample - Create a VM from a snapshot
services: virtual-machines-linux
author: roygara
editor: ramankum
ms.service: azure-disk-storage
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: vm-linux
ms.date: 02/23/2022
ms.author: rogarana
ms.custom: mvc, devx-track-azurecli
# Customer intent: As a cloud engineer, I want to create a virtual machine from a snapshot using CLI commands, so that I can efficiently deploy virtual machines with pre-configured settings and reduce setup time.
---

# Create a virtual machine from a snapshot with CLI

This script creates a virtual machine from a snapshot of an OS disk.

[!INCLUDE [quickstarts-free-trial-note](~/reusable-content/ce-skilling/azure/includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](~/reusable-content/azure-cli/azure-cli-prepare-your-environment.md)]

## Sample script

[!INCLUDE [cli-launch-cloud-shell-sign-in.md](~/reusable-content/ce-skilling/azure/includes/cli-launch-cloud-shell-sign-in.md)]

### Run the script

:::code language="azurecli" source="~/azure_cli_scripts/virtual-machine/create-vm-from-snapshot/create-vm-from-snapshot.sh" id="FullScript":::

## Clean up resources

Run the following command to remove the resource group, VM, and all related resources.

```azurecli-interactive
az group delete --name myResourceGroupName
```

## Sample reference

This script uses the following commands to create a managed disk, virtual machine, and all related resources. Each command in the table links to command specific documentation.

| Command | Notes |
|---|---|
| [az snapshot show](/cli/azure/snapshot) | Gets snapshot using snapshot name and resource group name. Id property of the returned object is used to create a managed disk.  |
| [az disk create](/cli/azure/disk) | Creates managed disks from a snapshot using snapshot Id, disk name, storage type, and size  |
| [az vm create](/cli/azure/vm) | Creates a VM using a managed OS disk |

## Next steps

For more information on the Azure CLI, see [Azure CLI documentation](/cli/azure).

Additional virtual machine CLI script samples can be found in the [Azure Linux VM documentation](../linux/cli-samples.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
