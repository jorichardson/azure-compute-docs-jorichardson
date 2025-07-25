---
title: Parameterize config files in Azure Service Fabric 
description: Learn how to parameterize configuration files in Service Fabric, a useful technique when managing multiple environments.
ms.topic: how-to
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-service-fabric
services: service-fabric
ms.date: 05/05/2025
# Customer intent: As a developer, I want to parameterize configuration files in Service Fabric, so that I can efficiently manage application settings across multiple environments.
---

# How to parameterize configuration files in Service Fabric

This article shows you how to parameterize a configuration file in Service Fabric.  If you're not already familiar with the core concepts of managing applications for multiple environments, read [Manage applications for multiple environments](service-fabric-manage-multiple-environment-app-configuration.md).

## Procedure for parameterizing configuration files

In this example, you override a configuration value using parameters in your application deployment.

1. Open the *\<MyService>\PackageRoot\Config\Settings.xml* file in your service project.
1. Set a configuration parameter name and value, for example cache size equal to 25, by adding the following XML:

   ```xml
    <Section Name="MyConfigSection">
      <Parameter Name="CacheSize" Value="25" />
    </Section>
   ```

1. Save and close the file.
1. Open the *\<MyApplication>\ApplicationPackageRoot\ApplicationManifest.xml* file.
1. In the ApplicationManifest.xml file, declare a parameter and default value in the `Parameters` element.  It's recommended that the parameter name contains the name of the service (for example, "MyService").

   ```xml
    <Parameters>
      <Parameter Name="MyService_CacheSize" DefaultValue="80" />
    </Parameters>
   ```
1. In the `ServiceManifestImport` section of the ApplicationManifest.xml file, add a `ConfigOverrides` and `ConfigOverride` element, referencing the configuration package, the section, and the parameter.

   ```xml
    <ConfigOverrides>
      <ConfigOverride Name="Config">
          <Settings>
            <Section Name="MyConfigSection">
                <Parameter Name="CacheSize" Value="[MyService_CacheSize]" />
            </Section>
          </Settings>
      </ConfigOverride>
    </ConfigOverrides>
   ```

> [!NOTE]
> In the case where you add a ConfigOverride, Service Fabric always chooses the application parameters or the default value specified in the application manifest.

## Access parameterized configurations in code

> [!NOTE]
> Too many or large overrides can affect API performance.
>
> Instead of using [FabricClient.QueryClient.GetApplicationListAsync](/dotnet/api/system.fabric.fabricclient.queryclient.getapplicationlistasync?view=azure-dotnet#system-fabric-fabricclient-queryclient-getapplicationlistasync), fetch parameters using the method described in this section. For clusters with many applications, try using default values to improve performance. The impact of application parameters on performance depends on factors like VM size, the number of parameters, the number of applications, and the length of the values.

You can access the configuration in your settings.xml file programmatically. Take, for example, the following configuration XML file:

   ```xml
<Settings
	xmlns:xsd="http://www.w3.org/2001/XMLSchema"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://schemas.microsoft.com/2011/01/fabric">
	<!-- Add your custom configuration sections and parameters here -->
	<Section Name="MyConfigSection">
		<Parameter Name="MyParameter" Value="Value1" />
	</Section>
</Settings>     
   ```
  
Use the following code to access the parameters:

  ```C#
CodePackageActivationContext context = FabricRuntime.GetActivationContext();
var configSettings = context.GetConfigurationPackageObject("Config").Settings;
var data = configSettings.Sections["MyConfigSection"];
foreach (var parameter in data.Parameters)
{
    ServiceEventSource.Current.ServiceMessage(this.Context, "Working-{0} - {1}", parameter.Name, parameter.Value);
}
  ```

Here, `Parameter.Name` is MyParameter and `Parameter.Value` is Value1.

## Next steps
For information about other app management capabilities that are available in Visual Studio, see [Manage your Service Fabric applications in Visual Studio](service-fabric-manage-application-in-visual-studio.md).
