---
title: Famsv6-series summary include file
description: Include file for Famsv6-series summary
author: mattmcinnes
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.date: 07/30/2024
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
ms.custom: include file
# Customer intent: As a cloud architect, I want to evaluate the Famsv6 VM series specifications, so that I can determine if they meet the performance needs of our CPU-intensive workloads and optimize our compute costs.
---
The Famsv6-series utilizes AMD's fourth Generation EPYC™ 9004 processor that can achieve a boosted maximum frequency of 3.7 GHz with up to 320 MB L3 cache. The Famsv6 VM series comes without Simultaneous Multithreading (SMT), meaning a vCPU is now mapped to a full physical core, allowing software processes to run on dedicated and uncontested resources. These new full core VMs suit workloads demanding the highest CPU performance. Famsv6-series offers up to 64 full core vCPUs and 512 GiB of RAM. This series is optimized for scientific simulations, financial and risk analysis, gaming, rendering and other workloads able to take advantage of the exceptional performance. Customers running software licensed on per-vCPU basis can use these VMs to optimize compute costs within their infrastructure.
