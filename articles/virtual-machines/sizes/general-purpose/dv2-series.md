---
title: Dv2 size series
description: Information on and specifications of the Dv2-series sizes
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 07/29/2024
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
# Customer intent: "As a cloud architect, I want to assess the specifications and capabilities of the Dv2-series virtual machines, so that I can select the most suitable size for my application's performance and resource requirements."
---

# Dv2 sizes series
[!INCLUDE [previous-gen-header](../includes/sizes-previous-gen-header.md)]

[!INCLUDE [dv2-summary](./includes/dv2-series-summary.md)]

## Host specifications
[!INCLUDE [dv2-series-specs](./includes/dv2-series-specs.md)]

## Feature support

Premium Storage: Not Supported<br>
Premium Storage caching: Not Supported<br>
Live Migration: Supported<br>
Memory Preserving Updates: Supported<br>
VM Generation Support: Generation 1<br>
Accelerated Networking: Supported<br>
Ephemeral OS Disks: Not Supported<br>
Nested Virtualization: Not Supported<br>

## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size

| Size Name | vCPUs (Qty.) | Memory (GB) |
| --- | --- | --- |
| Standard_D1_v2 | 1 | 3.5 |
| Standard_D2_v2 | 2 | 7 |
| Standard_D3_v2 | 4 | 14 |
| Standard_D4_v2 | 8 | 28 |
| Standard_D5_v2 | 16 | 56 |

#### VM Basics resources
- [What are vCPUs](../../../virtual-machines/managed-disks-overview.md)
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local Storage](#tab/sizestoragelocal)

Local (temp) storage info for each size

| Size Name | Max Temp Storage Disks (Qty.) | Temp Disk Size (GiB) | Temp Disk Random Read (RR)<sup>1</sup> IOPS | Temp Disk Random Read (RR)<sup>1</sup> Throughput (MB/s) | Temp Disk Random Write (RW)<sup>1</sup> Throughput (MB/s) |
| --- | --- | --- | --- | --- | --- |
| Standard_D1_v2 | 1 | 50 | 3000 | 46 | 23 |
| Standard_D2_v2 | 1 | 100 | 6000 | 93 | 46 |
| Standard_D3_v2 | 1 | 200 | 12000 | 187 | 93 |
| Standard_D4_v2 | 1 | 400 | 24000 | 375 | 187 |
| Standard_D5_v2 | 1 | 800 | 48000 | 750 | 375 |

#### Storage resources
- [Introduction to Azure managed disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure managed disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure managed disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- <sup>1</sup>Temp disk speed often differs between RR (Random Read) and RW (Random Write) operations. RR operations are typically faster than RW operations. The RW speed is usually slower than the RR speed on series where only the RR speed value is listed.
- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).

### [Remote Storage](#tab/sizestorageremote)

Remote (uncached) storage info for each size

| Size Name | Max Remote Storage Disks (Qty.) | Uncached Premium SSD Disk IOPS |
| --- | --- | --- |
| Standard_D1_v2 | 4 | 4x500 |
| Standard_D2_v2 | 8 | 8x500 |
| Standard_D3_v2 | 16 | 16x500 |
| Standard_D4_v2 | 32 | 32x500 |
| Standard_D5_v2 | 64 | 64x500 |

#### Storage resources
- [Introduction to Azure managed disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure managed disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure managed disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- <sup>1</sup>These sizes support [bursting](../../disk-bursting.md) to temporarily increase disk performance. Burst speeds can be maintained for up to 30 minutes at a time.

- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- Data disks can operate in cached or uncached modes. For cached data disk operation, the host cache mode is set to ReadOnly or ReadWrite. For uncached data disk operation, the host cache mode is set to None.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).


### [Network](#tab/sizenetwork)

Network interface info for each size

| Size Name | Max NICs (Qty.) | Max Network Bandwidth (Mb/s) |
| --- | --- | --- |
| Standard_D1_v2 | 2 | 750 |
| Standard_D2_v2 | 2 | 1500 |
| Standard_D3_v2 | 4 | 3000 |
| Standard_D4_v2 | 8 | 6000 |
| Standard_D5_v2 | 8 | 12000 |
|  |  |  |

#### Networking resources
- [Virtual networks and virtual machines in Azure](/azure/virtual-network/network-overview)
- [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)

#### Table definitions
- Expected network bandwidth is the maximum aggregated bandwidth allocated per VM type across all NICs, for all destinations. For more information, see [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)
- Upper limits aren't guaranteed. Limits offer guidance for selecting the right VM type for the intended application. Actual network performance will depend on several factors including network congestion, application loads, and network settings. For information on optimizing network throughput, see [Optimize network throughput for Azure virtual machines](/azure/virtual-network/virtual-network-optimize-network-bandwidth). 
-  To achieve the expected network performance on Linux or Windows, you may need to select a specific version or optimize your VM. For more information, see [Bandwidth/Throughput testing (NTTTCP)](/azure/virtual-network/virtual-network-bandwidth-testing).

### [Accelerators](#tab/sizeaccelerators)

Accelerator (GPUs, FPGAs, etc.) info for each size

> [!NOTE]
> No accelerators are present in this series.

---

[!INCLUDE [sizes-footer](../includes/sizes-footer.md)]


