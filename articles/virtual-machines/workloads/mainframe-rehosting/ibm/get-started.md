---
title: IBM workloads on Azure | Microsoft Docs
description: Use a mainframe emulator and other services from Microsoft partners to rehost your IBM z/OS workloads using Microsoft Azure.
services: virtual-machines
ms.service: azure-virtual-machines
ms.subservice: mainframe-rehosting
author: swread
ms.author: sread
manager: mamccrea 
ms.topic: concept-article
ms.date: 02/22/2019
# Customer intent: As an IT manager, I want to rehost our existing IBM z/OS applications on Azure so that I can leverage cloud elasticity and cost savings while maintaining functionality and minimizing disruption to our users.
---
# IBM workloads on Azure

Many IBM mainframe workloads based on z/OS can be replicated in Azure with no loss of functionality and without users even noticing changes in their underlying systems. Rehosting applications on Azure gives you the mainframe-like features you need plus the elasticity, availability, and potential cost savings of the cloud.

Azure supports integration with existing IBM mainframe environments, enabling you to migrate the applications that make sense, run hybrid solutions where needed, and migrate over time. Although you can completely rewrite existing mainframe-based programs for Azure, it’s more common to rehost them. Rewriting adds cost, complexity, and time to migration projects. With rehosting, you can:

- Move applications to a cloud-based emulator.

- Migrate the database to a cloud-based database.

- Replace modules and code using code transformation engines.

In addition, IBM software, including WebSphere and MQ, is now in the Azure Marketplace. With a license for IBM software, you can take advantage of the on-demand infrastructure scaling provided by Azure to quickly start a virtual machine.

An extensive partner ecosystem is available to help you migrate IBM mainframe systems to Azure. Most follow a pragmatic approach of reuse wherever possible before embarking on a phased deployment of rewriting or replacing applications. Get more guidance and help from partners at the [Azure Mainframe Migration center](https://azure.microsoft.com/migration/mainframe/).

**Next steps**

- [Mainframe migration: myths and facts](/azure/architecture/cloud-adoption/infrastructure/mainframe-migration/myths-and-facts)
- [Install IBM zD&T dev/test environment on Azure](./install-ibm-z-environment.md)
- [Set up an Application Developers Controlled Distribution (ADCD) in IBM zD&T v1](./demo.md)
- [IBM DB2 pureScale on Azure](ibm-db2-purescale-azure.md)
