---
title: Azure Virtual Machine Scale Sets overview
description: Learn about Azure Virtual Machine Scale Sets and how to automatically scale your applications
author: ju-shim
ms.author: jushiman
ms.topic: overview
ms.service: azure-virtual-machine-scale-sets
ms.subservice:
ms.date: 03/21/2025
ms.reviewer: mimckitt

# Customer intent: As a cloud architect, I want to implement Virtual Machine Scale Sets, so that I can efficiently manage application instances and automatically scale resources based on demand to ensure high availability and performance for my applications.
---
# What are Virtual Machine Scale Sets?

Azure Virtual Machine Scale Sets let you create and manage a group of load balanced virtual machines (VM) instances. The number of VM instances can automatically increase or decrease in response to demand or a defined schedule. Scale sets provide the following key benefits:
- Easy to create and manage multiple VMs
- Provides high availability and application resiliency by distributing VMs across availability zones or fault domains
- Allows your application to automatically scale as resource demand changes
- Works at large-scale

With Flexible orchestration, Azure provides a unified experience across the Azure VM ecosystem. Flexible orchestration offers high availability guarantees (up to 1,000 VMs) by spreading VMs across fault domains in a region or within an Availability Zone. This enables you to scale out your application while maintaining fault domain isolation that is essential to run workloads, including:
- Quorum-based workloads
- Open-source databases
- Stateful applications
- Services that require high availability and large scale
- Services that want to mix virtual machine types or use Spot and on-demand VMs together
- Existing Availability Set applications

Learn more about the differences between Uniform scale sets and Flexible scale sets in [Orchestration Modes](../virtual-machine-scale-sets/virtual-machine-scale-sets-orchestration-modes.md).

> [!IMPORTANT]
> The orchestration mode is defined when you create the scale set and cannot be changed or updated later.

:::image type="content" source="media/overview/azure-virtual-machine-scale-sets-video-thumbnail.jpg" alt-text="YouTube video about Virtual Machine Scale Sets." link="https://youtu.be/lE2xJXYHnB8":::

## Why use Virtual Machine Scale Sets?
To provide redundancy and improved performance, applications are typically distributed across multiple instances. Customers may access your application through a load balancer that distributes requests to one of the application instances. If you need to perform maintenance or update an application instance, your customers must be distributed to another available application instance. To keep up with extra customer demand, you may need to increase the number of application instances that run your application.

Azure Virtual Machine Scale Sets provide the management capabilities for applications that run across many VMs, automatic scaling of resources, and load balancing of traffic. Scale sets provide the following key benefits:

- **Easy to create and manage multiple VMs**
    - When you have many VMs that run your application, it's important to maintain a consistent configuration across your environment. For reliable performance of your application, the VM size, disk configuration, and application installs should match across all VMs.
    - With scale sets, all VM instances are created from the same base OS image and configuration. This approach lets you easily manage hundreds of VMs without extra configuration tasks or network management.
    - Scale sets support the use of the [Azure load balancer](/azure/load-balancer/load-balancer-overview) for basic layer-4 traffic distribution, and [Azure Application Gateway](/azure/application-gateway/overview) for more advanced layer-7 traffic distribution and TLS termination.

- **Provides high availability and application resiliency**
    - Scale sets are used to run multiple instances of your application. If one of these VM instances has a problem, customers continue to access your application through one of the other VM instances with minimal interruption.
    - For more availability, you can use [Availability Zones](/azure/reliability/availability-zones-overview) to automatically distribute VM instances in a scale set within a single datacenter or across multiple datacenters. A scale set alone can't protect you against data center failures. Deploying VMs across Availability Zones within a scale set can protect you against data center failure. 

- **Allows your application to automatically scale as resource demand changes**
    - Customer demand for your application may change throughout the day or week. To match customer demand, scale sets can automatically increase the number of VM instances as application demand increases, then reduce the number of VM instances as demand decreases.
    - Autoscale helps reduce the number of unnecessary VMs when demand is low. As demand increases, the scale set automatically adds more VMs to maintain an acceptable level of performance for your application. This ability helps reduce costs and efficiently create Azure resources as required.

- **Works at large-scale**
    - Scale sets support up to 1,000 VM instances for standard marketplace images and custom images through the Azure Compute Gallery (formerly known as Shared Image Gallery). If you create a scale set using a managed image, the limit is 600 VM instances.
    - For the best performance with production workloads, use [Azure Managed Disks](../virtual-machines/managed-disks-overview.md).

- **Cost-effective service**
    - There's no extra cost for using scale sets. You're charged based on the compute, network, and storage resources that the scale set uses.
    - For virtual machine pricing information, see [Azure pricing](https://azure.microsoft.com/pricing/).

## Next steps
> [!div class="nextstepaction"]
> [Flexible orchestration mode for your scale sets with Azure portal.](flexible-virtual-machine-scale-sets-portal.md)
