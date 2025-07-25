---
title: Scenarios to use a virtual network
description: Scenarios, resources, and limitations to deploy container groups to an Azure virtual network.
ms.topic: concept-article
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-container-instances
services: container-instances
ms.date: 08/29/2024
# Customer intent: As a cloud architect, I want to deploy container groups into an Azure virtual network so that I can enable secure communication between my containerized applications and other resources while ensuring proper network configurations and compliance with limitations.
---

# Virtual network scenarios and resources

[Azure Virtual Network](/azure/virtual-network/virtual-networks-overview) provides secure, private networking for your Azure and on-premises resources. By deploying container groups into an Azure virtual network, your containers can communicate securely with other resources in the virtual network. 

This article provides background about virtual network scenarios, limitations, and resources. For deployment examples using the Azure CLI, see [Deploy container instances into an Azure virtual network](container-instances-vnet.md).

> [!IMPORTANT]
> Container group deployment to a virtual network is generally available for Linux and Windows containers, in most regions where Azure Container Instances is available. For details, see [Resource availability and quota limits](container-instances-resource-and-quota-limits.md). 

## Scenarios

Container groups deployed into an Azure virtual network enable scenarios like:

* Direct communication between container groups in the same subnet
* Send [task-based](container-instances-restart-policy.md) workload output from container instances to a database in the virtual network
* Retrieve content for container instances from a [service endpoint](/azure/virtual-network/virtual-network-service-endpoints-overview) in the virtual network
* Enable container communication with on-premises resources through a [VPN gateway](/azure/vpn-gateway/vpn-gateway-about-vpngateways) or [ExpressRoute](/azure/expressroute/expressroute-introduction)
* Integrate with [Azure Firewall](/azure/firewall/overview) to identify outbound traffic originating from the container 
* Resolve names via the internal Azure DNS for communication with Azure resources in the virtual network, such as virtual machines
* Use NSG rules to control container access to subnets or other network resources

## Unsupported networking scenarios 

* **Azure Load Balancer** - Placing an Azure Load Balancer in front of container instances in a networked container group isn't supported
* **Global virtual network peering** - Global peering (connecting virtual networks across Azure regions) isn't supported
* **Public IP or DNS label** - Container groups deployed to a virtual network don't currently support exposing containers directly to the internet with a public IP address or a fully qualified domain name
* **Managed Identity with Virtual Network in Azure Government Regions** - Managed Identity with virtual networking capabilities isn't supported in Azure Government Regions

## Other limitations

* To deploy container groups to a subnet, the subnet can't contain other resource types. Remove all existing resources from an existing subnet before deploying container groups to it, or create a new subnet.
* To deploy container groups to a subnet, the subnet and the container group must be on the same Azure subscription.
* Due to the additional networking resources involved, deployments to a virtual network are typically slower than deploying a standard container instance.
* Outbound connections to port 25 and 19390 aren't supported at this time. Port 19390 needs to be opened in your Firewall for connecting to ACI from Azure portal when container groups are deployed in virtual networks.
* For inbound connections, the firewall should also allow all ip addresses within the virtual network.
* If you're connecting your container group to an Azure Storage Account, you must add a [service endpoint](/azure/virtual-network/virtual-network-service-endpoints-overview) to that resource.
* [IPv6 addresses](/azure/virtual-network/ip-services/ipv6-overview) aren't supported at this time.
* Depending on your subscription type, [certain ports could be blocked](/azure/virtual-network/network-security-groups-overview#azure-platform-considerations).
* Container instances don't read or inherit DNS settings from an associated virtual network. DNS settings must be explicitly set for container instances.

## Required network resources

There are three Azure Virtual Network resources required for deploying container groups to a virtual network: the [virtual network](#virtual-network) itself, a [delegated subnet](#subnet-delegated) within the virtual network, and a [network profile](#network-profile). 

### Virtual network

A virtual network defines the address space in which you create one or more subnets. You then deploy Azure resources (like container groups) into the subnets in your virtual network.

### Subnet (delegated)

Subnets segment the virtual network into separate address spaces usable by the Azure resources you place in them. You create one or several subnets within a virtual network.

The subnet that you use for container groups can contain only container groups. Before you deploy a container group to a subnet, you must explicitly delegate the subnet before provisioning. Once delegated, the subnet can be used only for container groups. If you attempt to deploy resources other than container groups to a delegated subnet, the operation fails.

### Network profile

[!INCLUDE [network profile callout](./includes/network-profile-callout.md)]

A network profile is a network configuration template for Azure resources. It specifies certain network properties for the resource, for example, the subnet into which it should be deployed. When you first use the [az container create][az-container-create] command to deploy a container group to a subnet (and thus a virtual network), Azure creates a network profile for you. You can then use that network profile for future deployments to the subnet. 

To use a Resource Manager template, YAML file, or a programmatic method to deploy a container group to a subnet, you need to provide the full Resource Manager resource ID of a network profile. You can use a profile previously created using [az container create][az-container-create], or create a profile using a Resource Manager template (see [template example](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.containerinstance/aci-vnet) and [reference](/azure/templates/microsoft.network/networkprofiles)). To get the ID of a previously created profile, use the [az network profile list][az-network-profile-list] command. 

The following diagram depicts several container groups deployed to a subnet delegated to Azure Container Instances. Once you deploy one container group to a subnet, you can deploy more container groups to it by specifying the same network profile.

![Container groups within a virtual network][aci-vnet-01]

## Next steps

* For deployment examples with the Azure CLI, see [Deploy container instances into an Azure virtual network](container-instances-vnet.md).
* To deploy a new virtual network, subnet, network profile, and container group using a Resource Manager template, see [Create an Azure container group with virtual network](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.containerinstance/aci-vnet).
* When using the [Azure portal](container-instances-quickstart-portal.md) to create a container instance, you can also provide settings for a new or existing virtual network on the **Networking** tab.


<!-- IMAGES -->
[aci-vnet-01]: ./media/container-instances-virtual-network-concepts/aci-vnet-01.png

<!-- LINKS - Internal -->
[az-container-create]: /cli/azure/container#az_container_create
[az-network-profile-list]: /cli/azure/network/profile#az_network_profile_list
