---
title: Manage Azure Service Fabric application secrets
description: Learn how to secure secret values in a Service Fabric application (platform-agnostic).
ms.topic: how-to
ms.author: tomcassidy
author: tomvcassidy
ms.service: azure-service-fabric
services: service-fabric
ms.date: 07/14/2022
# Customer intent: "As a cloud application developer, I want to securely manage application secrets in Service Fabric, so that I can protect sensitive information such as passwords and connection strings throughout the application lifecycle."
---

# Manage encrypted secrets in Service Fabric applications
This guide walks you through the steps of managing secrets in a Service Fabric application. Secrets can be any sensitive information, such as storage connection strings, passwords, or other values that should not be handled in plain text.

Using encrypted secrets in a Service Fabric application involves three steps:
* Set up an encryption certificate and encrypt secrets.
* Specify encrypted secrets in an application.
* Decrypt encrypted secrets from service code.

## Set up an encryption certificate and encrypt secrets
Setting up an encryption certificate and using it to encrypt secrets varies between Windows and Linux.
* [Set up an encryption certificate and encrypt secrets on Windows clusters.][secret-management-windows-specific-link]
* [Set up an encryption certificate and encrypt secrets on Linux clusters.][secret-management-linux-specific-link]

## Specify encrypted secrets in an application
The previous step describes how to encrypt a secret with a certificate and produce a base-64 encoded string for use in an application. This base-64 encoded string can be specified as an encrypted [parameter][parameters-link] in a service's Settings.xml or as an encrypted [environment variable][environment-variables-link] in a service's ServiceManifest.xml.

Specify an encrypted [parameter][parameters-link] in your service's Settings.xml configuration file with the `IsEncrypted` attribute set to `true`:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<Settings xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/2011/01/fabric">
  <Section Name="MySettings">
    <Parameter Name="MySecret" IsEncrypted="true" Value="I6jCCAeYCAxgFhBXABFxzAt ... gNBRyeWFXl2VydmjZNwJIM=" />
  </Section>
</Settings>
```
Specify an encrypted [environment variable][environment-variables-link] in your service's ServiceManifest.xml file with the `Type` attribute set to `Encrypted`:
```xml
<CodePackage Name="Code" Version="1.0.0">
  <EnvironmentVariables>
    <EnvironmentVariable Name="MyEnvVariable" Type="Encrypted" Value="I6jCCAeYCAxgFhBXABFxzAt ... gNBRyeWFXl2VydmjZNwJIM=" />
  </EnvironmentVariables>
</CodePackage>
```

The secrets should also be included in your Service Fabric application by specifying a certificate in the application manifest. Add a **SecretsCertificate** element to **ApplicationManifest.xml** and include the desired certificate's thumbprint.

```xml
<ApplicationManifest … >
  ...
  <Certificates>
    <SecretsCertificate Name="MyCert" X509FindType="FindByThumbprint" X509FindValue="[YourCertThumbrint]"/>
  </Certificates>
</ApplicationManifest>
```
> [!NOTE]
> Upon activating an application which specifies a SecretsCertificate, Service Fabric will find the matching certificate, and grant the identity the application is running under full permissions to the certificate's private key. Service Fabric will also monitor the certificate for changes, and re-apply the permissions accordingly. To detect changes for certificates declared by common name, Service Fabric runs a periodic task which finds all matching certificates, and compares it with a cached list of thumbprints. When a new thumbprint is detected, it means that a certificate by that subject has been renewed. The task runs once per minute on each node of the cluster.
>
> While the SecretsCertificate does allow subject-based declarations, do note that the encrypted settings are tied to the key pair which was used to encrypt the setting on the client. You must ensure that the original encryption certificate (or an equivalent) matches the subject-based declaration, and that it is installed, including its corresponding private key, on every node of the cluster which could host the application. All time-valid certificates matching the subject-based declaration and built from the same the key pair as the original encryption certificate are considered equivalents.
>

### Inject application secrets into application instances
Ideally, deployment to different environments should be as automated as possible. This can be accomplished by performing secret encryption in a build environment and providing the encrypted secrets as parameters when creating application instances.

#### Use overridable parameters in Settings.xml
The Settings.xml configuration file allows overridable parameters that can be provided at application creation time. Use the `MustOverride` attribute instead of providing a value for a parameter:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<Settings xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/2011/01/fabric">
  <Section Name="MySettings">
    <Parameter Name="MySecret" IsEncrypted="true" Value="" MustOverride="true" />
  </Section>
</Settings>
```

To override values in Settings.xml, declare an override parameter for the service in ApplicationManifest.xml:

```xml
<ApplicationManifest ... >
  <Parameters>
    <Parameter Name="MySecret" DefaultValue="" />
  </Parameters>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="Stateful1Pkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides>
      <ConfigOverride Name="Config">
        <Settings>
          <Section Name="MySettings">
            <Parameter Name="MySecret" Value="[MySecret]" IsEncrypted="true" />
          </Section>
        </Settings>
      </ConfigOverride>
    </ConfigOverrides>
  </ServiceManifestImport>
 ```

Now the value can be specified as an *application parameter* when creating an instance of the application. Creating an application instance can be scripted using PowerShell, or written in C#, for easy integration in a build process.

Using PowerShell, the parameter is supplied to the `New-ServiceFabricApplication` command as a [hash table](/previous-versions/windows/it-pro/windows-powershell-1.0/ee692803(v=technet.10)):

```powershell
New-ServiceFabricApplication -ApplicationName fabric:/MyApp -ApplicationTypeName MyAppType -ApplicationTypeVersion 1.0.0 -ApplicationParameter @{"MySecret" = "I6jCCAeYCAxgFhBXABFxzAt ... gNBRyeWFXl2VydmjZNwJIM="}
```

Using C#, application parameters are specified in an `ApplicationDescription` as a `NameValueCollection`:

```csharp
FabricClient fabricClient = new FabricClient();

NameValueCollection applicationParameters = new NameValueCollection();
applicationParameters["MySecret"] = "I6jCCAeYCAxgFhBXABFxzAt ... gNBRyeWFXl2VydmjZNwJIM=";

ApplicationDescription applicationDescription = new ApplicationDescription(
    applicationName: new Uri("fabric:/MyApp"),
    applicationTypeName: "MyAppType",
    applicationTypeVersion: "1.0.0",
    applicationParameters: applicationParameters)
);

await fabricClient.ApplicationManager.CreateApplicationAsync(applicationDescription);
```

## Decrypt encrypted secrets from service code
The APIs for accessing [parameters][parameters-link] and [environment variables][environment-variables-link] allow for easy decryption of encrypted values. Since the encrypted string contains information about the certificate used for encryption, you do not need to manually specify the certificate. The certificate just needs to be installed on the node that the service is running on.

```csharp
// Access decrypted parameters from Settings.xml
ConfigurationPackage configPackage = FabricRuntime.GetActivationContext().GetConfigurationPackageObject("Config");
bool MySecretIsEncrypted = configPackage.Settings.Sections["MySettings"].Parameters["MySecret"].IsEncrypted;
if (MySecretIsEncrypted)
{
    SecureString MySecretDecryptedValue = configPackage.Settings.Sections["MySettings"].Parameters["MySecret"].DecryptValue();
}

// Access decrypted environment variables from ServiceManifest.xml
// Note: you do not have to call any explicit API to decrypt the environment variable.
string MyEnvVariable = Environment.GetEnvironmentVariable("MyEnvVariable");
```

## Next steps
* Service Fabric [Secrets Store](service-fabric-application-secret-store.md) 
* Learn more about [application and service security](service-fabric-application-and-service-security.md)

<!-- Links -->
[parameters-link]:service-fabric-how-to-parameterize-configuration-files.md
[environment-variables-link]: service-fabric-how-to-specify-environment-variables.md
[secret-management-windows-specific-link]: service-fabric-application-secret-management-windows.md
[secret-management-linux-specific-link]: service-fabric-application-secret-management-linux.md
[service fabric secrets store]: service-fabric-application-secret-store.md
