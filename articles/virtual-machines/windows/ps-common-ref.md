---
title: Common PowerShell commands for Azure Virtual Machines
description: Common PowerShell commands to get you started creating and managing VMs in Azure.
author: ju-shim
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 09/07/2023
ms.author: jushiman
# Customer intent: As a system administrator, I want to learn common PowerShell commands for managing Azure Virtual Machines, so that I can efficiently create, configure, and control VM resources in my cloud environment.
---
# Common PowerShell commands for creating and managing Azure Virtual Machines

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets 

This article covers some of the basic Azure PowerShell commands that you can use to create and manage virtual machines in your Azure subscription.  For more detailed help with specific command-line switches and options, you can use the **Get-Help** *command*.

These variables might be useful if running more than one of the commands in this article:

- $location - The location of the virtual machine. You can use [Get-AzLocation](/powershell/module/az.resources/get-azlocation) to find a [geographical region](https://azure.microsoft.com/regions/) that works for you.
- $myResourceGroup - The name of the resource group that contains the virtual machine.
- $myVM - The name of the virtual machine.

## Create a VM - simplified

| Task | Command |
| ---- | ------- |
| Create a simple VM | [New-AzVM](/powershell/module/az.compute/new-azvm) -Name $myVM <BR></BR><BR></BR> New-AzVM has a set of *simplified* parameters, where all that is required is a single name. The value for -Name will be used as the name for all of the resources required for creating a new VM. You can specify more, but this is all that is required.|
| Create a VM from a custom image | New-AzVm -ResourceGroupName $myResourceGroup -Name $myVM ImageName "myImage" -Location $location  <BR></BR><BR></BR>You need to have already created your own [managed image](capture-image-resource.yml). You can use an image to make multiple, identical VMs. |



## Create a VM - advanced

| Task | Command |
| ---- | ------- |
| Create a VM configuration |$vm = [New-AzVMConfig](/powershell/module/az.compute/new-azvmconfig) -VMName $myVM -VMSize "Standard_D1_v1"<BR></BR><BR></BR>The VM configuration is used to define or update settings for the VM. The configuration is initialized with the name of the VM and its [size](../sizes.md). |
| Add configuration settings |$vm = [Set-AzVMOperatingSystem](/powershell/module/az.compute/set-azvmoperatingsystem) -VM $vm -Windows -ComputerName $myVM -Credential $cred -ProvisionVMAgent -EnableAutoUpdate<BR></BR><BR></BR>Operating system settings including [credentials](/powershell/module/microsoft.powershell.security/get-credential) are added to the configuration object that you previously created using New-AzVMConfig. |
| Add a network interface |$vm = [Add-AzVMNetworkInterface](/powershell/module/az.compute/add-azvmnetworkinterface) -VM $vm -Id $nic.Id<BR></BR><BR></BR>A VM must have a [network interface](./quick-create-powershell.md?toc=/azure/virtual-machines/windows/toc.json) to communicate in a virtual network. You can also use [Get-AzNetworkInterface](/powershell/module/az.compute/add-azvmnetworkinterface) to retrieve an existing network interface object. |
| Specify a platform image |$vm = [Set-AzVMSourceImage](/powershell/module/az.compute/set-azvmsourceimage) -VM $vm -PublisherName "publisher_name" -Offer "publisher_offer" -Skus "product_sku" -Version "latest"<BR></BR><BR></BR>[Image information](cli-ps-findimage.md) is added to the configuration object that you previously created using New-AzVMConfig. The object returned from this command is only used when you set the OS disk to use a platform image. |
| Create a VM |[New-AzVM](/powershell/module/az.compute/new-azvm) -ResourceGroupName $myResourceGroup -Location $location -VM $vm<BR></BR><BR></BR>All resources are created in a [resource group](/azure/azure-resource-manager/management/manage-resource-groups-powershell). Before you run this command, run New-AzVMConfig, Set-AzVMOperatingSystem, Set-AzVMSourceImage, Add-AzVMNetworkInterface, and Set-AzVMOSDisk. |
| Update a VM |[Update-AzVM](/powershell/module/az.compute/update-azvm) -ResourceGroupName $myResourceGroup -VM $vm<BR></BR><BR></BR>Get the current VM configuration using Get-AzVM, change configuration settings on the VM object, and then run this command. |

## Get information about your VMs

| Task | Command |
| ---- | ------- |
| List VMs in a subscription |[Get-AzVM](/powershell/module/az.compute/get-azvm) |
| List VMs in a resource group |Get-AzVM -ResourceGroupName $myResourceGroup<BR></BR><BR></BR>To get a list of resource groups in your subscription, use [Get-AzResourceGroup](/powershell/module/az.resources/get-azresourcegroup). |
| Get information about a VM |Get-AzVM -ResourceGroupName $myResourceGroup -Name $myVM |

## Manage your VMs
| Task | Command |
| --- | --- |
| Start a VM |[Start-AzVM](/powershell/module/az.compute/start-azvm) -ResourceGroupName $myResourceGroup -Name $myVM |
| Stop a VM |[Stop-AzVM](/powershell/module/az.compute/stop-azvm) -ResourceGroupName $myResourceGroup -Name $myVM |
| Restart a running VM |[Restart-AzVM](/powershell/module/az.compute/restart-azvm) -ResourceGroupName $myResourceGroup -Name $myVM |
| Delete a VM |[Remove-AzVM](/powershell/module/az.compute/remove-azvm) -ResourceGroupName $myResourceGroup -Name $myVM |


## Next steps
* See the basic steps for creating a virtual machine in [Create a Windows VM using Resource Manager and PowerShell](./quick-create-powershell.md?toc=/azure/virtual-machines/windows/toc.json).
