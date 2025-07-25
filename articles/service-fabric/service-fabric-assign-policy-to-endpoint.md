---
title: Assign access policies to service endpoints 
description: Learn how to assign security access policies to HTTP or HTTPS endpoints in your Service Fabric service.
ms.topic: how-to
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-service-fabric
services: service-fabric
ms.date: 07/14/2022
# Customer intent: As a software developer, I want to assign security access policies to my service endpoints so that I can ensure proper access control and secure communication for my applications deployed on Service Fabric.
---

# Assign a security access policy for HTTP and HTTPS endpoints
If you apply a run-as policy and the service manifest declares HTTP endpoint resources, you must specify a **SecurityAccessPolicy**.  **SecurityAccessPolicy** ensures that ports allocated to these endpoints are correctly restricted to the user account that the service runs as. Otherwise, **http.sys** does not have access to the service, and you get failures with calls from the client. The following example applies the Customer1 account to an endpoint called **EndpointName**, which gives it full access rights.

```xml
<Policies>
  <RunAsPolicy CodePackageRef="Code" UserRef="Customer1" />
  <!--SecurityAccessPolicy is needed if RunAsPolicy is defined and the Endpoint is http -->
  <SecurityAccessPolicy ResourceRef="EndpointName" PrincipalRef="Customer1" />
</Policies>
```

For an HTTPS endpoint, also indicate the name of the certificate to return to the client. You reference the certificate using **EndpointBindingPolicy**.  The certificate is defined in the **Certificates** section of the application manifest.

```xml
<Policies>
  <RunAsPolicy CodePackageRef="Code" UserRef="Customer1" />
  <!--SecurityAccessPolicy is needed if RunAsPolicy is defined and the Endpoint is http -->
  <SecurityAccessPolicy ResourceRef="EndpointName" PrincipalRef="Customer1" />
  <!--EndpointBindingPolicy is needed if the EndpointName is secured with https -->
  <EndpointBindingPolicy EndpointRef="EndpointName" CertificateRef="Cert1" />
</Policies>
```

> [!WARNING] 
> When using HTTPS, do not use the same port and certificate for different service instances (independent of the application) deployed to the same node. Upgrading two different services using the same port in different application instances will result in an upgrade failure. For more information, see [Upgrading multiple applications with HTTPS endpoints
](service-fabric-application-upgrade.md#upgrading-multiple-applications-with-https-endpoints).
> 

<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
For next steps, read the following articles:
* [Understand the application model](service-fabric-application-model.md)
* [Specify resources in a service manifest](service-fabric-service-manifest-resources.md)
* [Deploy an application](service-fabric-deploy-remove-applications.md)
* [Azure Service Fabric security best practices](service-fabric-best-practices-security.md)

[image1]: ./media/service-fabric-application-runas-security/copy-to-output.png
