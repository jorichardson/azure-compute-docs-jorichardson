---
title: Full Trusted launch Scale set ARM template.
description: Include file for Full Trusted launch Scale set ARM template JSON.
author: AjKundnani
ms.topic: include
ms.service: azure-virtual-machines
ms.subservice: trusted-launch
ms.date: 06/13/2024
ms.author: ajkundna
ms.custom: include file, devx-track-arm-template
# Customer intent: "As an Azure architect, I want to deploy a Virtual Machine Scale Set with Trusted Launch security features, so that I can enhance the security of my applications by ensuring trusted boot and running state verification."
---

<details>

<summary> Expand to view the complete sample ARM template, which supports upgrade of existing scale set to Trusted launch and roll-back (if needed). </summary>

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmSku": {
      "type": "string",
      "defaultValue": "Standard_D2s_v3",
      "metadata": {
        "description": "Size of VMs in the VM Scale Set."
      }
    },
    "sku": {
      "type": "string",
      "defaultValue": "2022-datacenter-azure-edition",
      "allowedValues": [
        "2022-datacenter-azure-edition"
      ],
      "metadata": {
        "description": "The Windows version for the VM. This will pick a fully patched image of this given Windows version."
      }
    },
    "vmssName": {
      "type": "string",
      "maxLength": 61,
      "metadata": {
        "description": "String used as a base for naming resources. Must be 3-61 characters in length and globally unique across Azure. A hash is prepended to this string for some resources, and resource-specific information is appended."
      }
    },
    "instanceCount": {
      "type": "int",
      "defaultValue": 2,
      "maxValue": 100,
      "minValue": 1,
      "metadata": {
        "description": "Number of VM instances (100 or less)."
      }
    },
    "adminUsername": {
      "type": "string",
      "metadata": {
        "description": "Admin username on all VMs."
      }
    },
    "adminPassword": {
      "type": "securestring",
      "metadata": {
        "description": "Admin password on all VMs."
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Location for all resources."
      }
    },
    "publicIpName": {
      "type": "string",
      "defaultValue": "myPublicIP",
      "metadata": {
        "description": "Name for the Public IP used to access the virtual machine Scale set."
      }
    },
    "publicIPAllocationMethod": {
      "type": "string",
      "defaultValue": "Static",
      "allowedValues": [
        "Dynamic",
        "Static"
      ],
      "metadata": {
        "description": "Allocation method for the Public IP used to access the virtual machine set."
      }
    },
    "publicIpSku": {
      "type": "string",
      "defaultValue": "Standard",
      "allowedValues": [
        "Basic",
        "Standard"
      ],
      "metadata": {
        "description": "SKU for the Public IP used to access the virtual machine Scale set."
      }
    },
    "dnsLabelPrefix": {
      "type": "string",
      "defaultValue": "[toLower(format('{0}-{1}', parameters('vmssName'), uniqueString(resourceGroup().id)))]",
      "metadata": {
        "description": "Unique DNS Name for the Public IP used to access the virtual machine Scale set."
      }
    },
    "healthExtensionProtocol": {
      "type": "string",
      "defaultValue": "TCP",
      "allowedValues": [
        "TCP",
        "HTTP",
        "HTTPS"
      ]
    },
    "healthExtensionPort": {
      "type": "int",
      "defaultValue": 3389
    },
    "healthExtensionRequestPath": {
      "type": "string",
      "defaultValue": "/"
    },
    "overprovision": {
      "type": "bool",
      "defaultValue": false
    },
    "upgradePolicy": {
      "type": "string",
      "defaultValue": "Manual",
      "allowedValues": [
        "Manual",
        "Rolling",
        "Automatic"
      ]
    },
    "maxBatchInstancePercent": {
      "type": "int",
      "defaultValue": 20
    },
    "maxUnhealthyInstancePercent": {
      "type": "int",
      "defaultValue": 20
    },
    "maxUnhealthyUpgradedInstancePercent": {
      "type": "int",
      "defaultValue": 20
    },
    "pauseTimeBetweenBatches": {
      "type": "string",
      "defaultValue": "PT5S"
    },
    "securityType": {
      "type": "string",
      "defaultValue": "TrustedLaunch",
      "allowedValues": [
        "Standard",
        "TrustedLaunch"
      ],
      "metadata": {
        "description": "Security Type of the Virtual Machine."
      }
    },
    "encryptionAtHost": {
        "type": "bool",
        "defaultValue": false,
        "metadata": {
            "description": "This property can be used by user in the request to enable or disable the Host Encryption for the virtual machine or virtual machine Scale set. This will enable the encryption for all the disks including Resource/Temp disk at host itself. The default behavior is: The Encryption at host will be disabled unless this property is set to true for the resource."
        }
    }
  },
  "variables": {
    "namingInfix": "[toLower(substring(format('{0}{1}', parameters('vmssName'), uniqueString(resourceGroup().id)), 0, 9))]",
    "addressPrefix": "10.0.0.0/16",
    "subnetPrefix": "10.0.0.0/24",
    "virtualNetworkName": "[format('{0}vnet', variables('namingInfix'))]",
    "subnetName": "[format('{0}subnet', variables('namingInfix'))]",
    "loadBalancerName": "[format('{0}lb', variables('namingInfix'))]",
    "natPoolName": "[format('{0}natpool', variables('namingInfix'))]",
    "bePoolName": "[format('{0}bepool', variables('namingInfix'))]",
    "natStartPort": 50000,
    "natEndPort": 50119,
    "natBackendPort": 3389,
    "nicName": "[format('{0}nic', variables('namingInfix'))]",
    "ipConfigName": "[format('{0}ipconfig', variables('namingInfix'))]",
    "imageReference": {
      "2022-datacenter-azure-edition": {
        "publisher": "MicrosoftWindowsServer",
        "offer": "WindowsServer",
        "sku": "[parameters('sku')]",
        "version": "latest"
      }
    },
    "extensionName": "GuestAttestation",
    "extensionPublisher": "Microsoft.Azure.Security.WindowsAttestation",
    "extensionVersion": "1.0",
    "maaTenantName": "GuestAttestation",
    "maaEndpoint": "[substring('emptyString', 0, 0)]",
    "uefiSettingsJson": {
        "secureBootEnabled": true,
        "vTpmEnabled": true
    },
    "rollingUpgradeJson": {
      "maxBatchInstancePercent": "[parameters('maxBatchInstancePercent')]",
      "maxUnhealthyInstancePercent": "[parameters('maxUnhealthyInstancePercent')]",
      "maxUnhealthyUpgradedInstancePercent": "[parameters('maxUnhealthyUpgradedInstancePercent')]",
      "pauseTimeBetweenBatches": "[parameters('pauseTimeBetweenBatches')]"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Network/virtualNetworks",
      "apiVersion": "2022-05-01",
      "name": "[variables('virtualNetworkName')]",
      "location": "[parameters('location')]",
      "properties": {
        "addressSpace": {
          "addressPrefixes": [
            "[variables('addressPrefix')]"
          ]
        },
        "subnets": [
          {
            "name": "[variables('subnetName')]",
            "properties": {
              "addressPrefix": "[variables('subnetPrefix')]"
            }
          }
        ]
      }
    },
    {
      "type": "Microsoft.Network/publicIPAddresses",
      "apiVersion": "2022-05-01",
      "name": "[parameters('publicIpName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "[parameters('publicIpSku')]"
      },
      "properties": {
        "publicIPAllocationMethod": "[parameters('publicIPAllocationMethod')]",
        "dnsSettings": {
          "domainNameLabel": "[parameters('dnsLabelPrefix')]"
        }
      }
    },
    {
      "type": "Microsoft.Network/loadBalancers",
      "apiVersion": "2022-05-01",
      "name": "[variables('loadBalancerName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "[parameters('publicIpSku')]",
        "tier": "Regional"
      },
      "properties": {
        "frontendIPConfigurations": [
          {
            "name": "LoadBalancerFrontEnd",
            "properties": {
              "publicIPAddress": {
                "id": "[resourceId('Microsoft.Network/publicIPAddresses', parameters('publicIpName'))]"
              }
            }
          }
        ],
        "backendAddressPools": [
          {
            "name": "[variables('bePoolName')]"
          }
        ],
        "inboundNatPools": [
          {
            "name": "[variables('natPoolName')]",
            "properties": {
              "frontendIPConfiguration": {
                "id": "[resourceId('Microsoft.Network/loadBalancers/frontendIPConfigurations', variables('loadBalancerName'), 'loadBalancerFrontEnd')]"
              },
              "protocol": "Tcp",
              "frontendPortRangeStart": "[variables('natStartPort')]",
              "frontendPortRangeEnd": "[variables('natEndPort')]",
              "backendPort": "[variables('natBackendPort')]"
            }
          }
        ]
      },
      "dependsOn": [
        "[resourceId('Microsoft.Network/publicIPAddresses', parameters('publicIpName'))]"
      ]
    },
    {
      "type": "Microsoft.Compute/virtualMachineScaleSets",
      "apiVersion": "2022-03-01",
      "name": "[parameters('vmssName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "[parameters('vmSku')]",
        "tier": "Standard",
        "capacity": "[parameters('instanceCount')]"
      },
      "properties": {
        "virtualMachineProfile": {
          "storageProfile": {
            "osDisk": {
              "createOption": "FromImage",
              "caching": "ReadWrite"
            },
            "imageReference": "[variables('imageReference')[parameters('sku')]]"
          },
          "osProfile": {
            "computerNamePrefix": "[variables('namingInfix')]",
            "adminUsername": "[parameters('adminUsername')]",
            "adminPassword": "[parameters('adminPassword')]"
          },
          "securityProfile": {
              "encryptionAtHost": "[parameters('encryptionAtHost')]",
              "securityType": "[parameters('securityType')]",
              "uefiSettings": "[if(equals(parameters('securityType'), 'TrustedLaunch'), variables('uefiSettingsJson'), null())]"
              
          },
          "networkProfile": {
            "networkInterfaceConfigurations": [
              {
                "name": "[variables('nicName')]",
                "properties": {
                  "primary": true,
                  "ipConfigurations": [
                    {
                      "name": "[variables('ipConfigName')]",
                      "properties": {
                        "subnet": {
                          "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('virtualNetworkName'), variables('subnetName'))]"
                        },
                        "loadBalancerBackendAddressPools": [
                          {
                            "id": "[resourceId('Microsoft.Network/loadBalancers/backendAddressPools', variables('loadBalancerName'), variables('bePoolName'))]"
                          }
                        ],
                        "loadBalancerInboundNatPools": [
                          {
                            "id": "[resourceId('Microsoft.Network/loadBalancers/inboundNatPools', variables('loadBalancerName'), variables('natPoolName'))]"
                          }
                        ]
                      }
                    }
                  ]
                }
              }
            ]
          },
          "extensionProfile": {
            "extensions": [
              {
                "name": "HealthExtension",
                "properties": {
                  "publisher": "Microsoft.ManagedServices",
                  "type": "ApplicationHealthWindows",
                  "typeHandlerVersion": "1.0",
                  "autoUpgradeMinorVersion": false,
                  "settings": {
                    "protocol": "[parameters('healthExtensionProtocol')]",
                    "port": "[parameters('healthExtensionPort')]",
                    "requestPath": "[if(equals(parameters('healthExtensionProtocol'), 'TCP'), null(), parameters('healthExtensionRequestPath'))]"
                  }
                }
              }
            ]
          },
          "diagnosticsProfile": {
            "bootDiagnostics": {
              "enabled": true
            }
          }
        },
        "orchestrationMode": "Uniform",
        "overprovision": "[parameters('overprovision')]",
        "upgradePolicy": {
          "mode": "[parameters('upgradePolicy')]",
          "rollingUpgradePolicy": "[if(equals(parameters('upgradePolicy'), 'Rolling'), variables('rollingUpgradeJson'), null())]",
          "automaticOSUpgradePolicy": {
            "enableAutomaticOSUpgrade": true
          }
        }
      },
      "dependsOn": [
        "[resourceId('Microsoft.Network/loadBalancers', variables('loadBalancerName'))]",
        "[resourceId('Microsoft.Network/virtualNetworks', variables('virtualNetworkName'))]"
      ]
    },
    {
      "condition": "[and(equals(parameters('securityType'), 'TrustedLaunch'), and(equals(variables('uefiSettingsJson').secureBootEnabled, true()), equals(variables('uefiSettingsJson').vTpmEnabled, true())))]",
      "type": "Microsoft.Compute/virtualMachineScaleSets/extensions",
      "apiVersion": "2022-03-01",
      "name": "[format('{0}/{1}', parameters('vmssName'), variables('extensionName'))]",
      "location": "[parameters('location')]",
      "properties": {
        "publisher": "[variables('extensionPublisher')]",
        "type": "[variables('extensionName')]",
        "typeHandlerVersion": "[variables('extensionVersion')]",
        "autoUpgradeMinorVersion": true,
        "enableAutomaticUpgrade": true,
        "settings": {
          "AttestationConfig": {
            "MaaSettings": {
              "maaEndpoint": "[variables('maaEndpoint')]",
              "maaTenantName": "[variables('maaTenantName')]"
            }
          }
        }
      },
      "dependsOn": [
        "[resourceId('Microsoft.Compute/virtualMachineScaleSets', parameters('vmssName'))]"
      ]
    }
  ]
}
```

</details>
