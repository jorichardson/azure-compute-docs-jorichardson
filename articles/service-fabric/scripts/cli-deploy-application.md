---
title: Azure Service Fabric CLI (sfctl) Script Deploy Sample
description: Deploy an application to an Azure Service Fabric cluster using the Azure Service Fabric CLI
services: service-fabric
author: athinanthny
manager: chackdan
ms.service: azure-service-fabric
ms.topic: sample
ms.date: 04/16/2018
ms.author: atsenthi
ms.custom: mvc
# Customer intent: "As a developer, I want to deploy an application using the Service Fabric CLI, so that I can efficiently manage application instances within my Azure Service Fabric cluster."
---

# Deploy an application to a Service Fabric cluster using the Service Fabric CLI

This sample script copies an application package to a cluster image store, registers the application type in the cluster, and creates an application instance from the application type. Any default services are also created at this time.

If needed, install the [Service Fabric CLI](../service-fabric-cli.md).

## Sample script

[!code-sh[main](../../../cli_scripts/service-fabric/deploy-application/deploy-application.sh "Deploy an application to a cluster")]

## Clean up deployment

When done, the [remove](cli-remove-application.md) script can be used to remove the application. The remove script
deletes the application instance, unregisters the application type, and deletes the application package from the
image store.

## Next steps

For more information, see the [Service Fabric CLI documentation](../service-fabric-cli.md).

Additional Service Fabric CLI samples for Azure Service Fabric can be found in the [Service Fabric CLI samples](../samples-cli.md).
