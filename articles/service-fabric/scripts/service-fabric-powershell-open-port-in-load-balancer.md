---
title: Open application port in load balancer in PowerShell
description: Azure PowerShell Script Sample - Open a port in the Azure load balancer for a Service Fabric application.
services: service-fabric
author: athinanthny
manager: chackdan
ms.service: azure-service-fabric
ms.topic: sample
ms.date: 05/18/2018
ms.author: atsenthi
ms.custom: mvc, devx-track-azurepowershell
# Customer intent: As a cloud administrator, I want to open application ports in an Azure load balancer using PowerShell so that my Service Fabric application can communicate with external clients effectively.
---

# Open an application port in the Azure load balancer

A Service Fabric application running in Azure sits behind the Azure load balancer. This sample script opens a port in an Azure load balancer so that a Service Fabric application can communicate with external clients. Customize the parameters as needed. If your cluster is in a network security group, also [add an inbound network security group rule](service-fabric-powershell-add-nsg-rule.md) to allow inbound traffic.

[!INCLUDE [updated-for-az](~/reusable-content/ce-skilling/azure/includes/updated-for-az.md)]

If needed, install the Service Fabric PowerShell module with the [Service Fabric SDK](../service-fabric-get-started.md). 

## Sample script

[!code-powershell[main](../../../powershell_scripts/service-fabric/open-port-in-load-balancer/open-port-in-load-balancer.ps1 "Open a port in the load balancer")]

## Script explanation

This script uses the following commands. Each command in the table links to command-specific documentation.

| Command | Notes |
|---|---|
| [Get-AzResource](/powershell/module/az.resources/get-azresource) | Gets an Azure resource.  |
| [Get-AzLoadBalancer](/powershell/module/az.network/get-azloadbalancer) | Gets the Azure load balancer. |
| [Add-AzLoadBalancerProbeConfig](/powershell/module/az.network/add-azloadbalancerprobeconfig) | Adds a probe configuration to a load balancer.|
| [Get-AzLoadBalancerProbeConfig](/powershell/module/az.network/get-azloadbalancerprobeconfig) | Gets a probe configuration for a load balancer. |
| [Add-AzLoadBalancerRuleConfig](/powershell/module/az.network/add-azloadbalancerruleconfig) | Adds a rule configuration to a load balancer. |
| [Set-AzLoadBalancer](/powershell/module/az.network/set-azloadbalancer) | Sets the goal state for a load balancer. |

## Next steps

For more information on the Azure PowerShell module, see [Azure PowerShell documentation](/powershell/azure/).

Additional PowerShell samples for Azure Service Fabric can be found in the [Azure PowerShell samples](../service-fabric-powershell-samples.md).
