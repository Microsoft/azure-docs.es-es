---
title: 'Prueba de máquina virtual (VM) implementada desde un disco duro virtual: Azure Marketplace'
description: Obtenga información sobre cómo probar y enviar una oferta de máquina virtual en Marketplace comercial.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: iqshahmicrosoft
ms.author: iqshah
ms.date: 07/29/2020
ms.openlocfilehash: 36c497a180070358332997649e768999c78935cb
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87433099"
---
# <a name="test-virtual-machine-vm-deployed-from-vhd"></a>Prueba de máquina virtual implementada desde un disco duro virtual

En este artículo se describe cómo implementar y probar una máquina virtual (VM) de Azure a partir de la imagen de disco duro virtual generalizada que se creó en la sección anterior ([Creación de recursos técnicos de máquina virtual de Azure)](create-azure-vm-technical-asset.md), para asegurarse de que la imagen del disco duro virtual cumple los requisitos de publicación de Azure Marketplace.

Siga estos pasos para generar un informe de compatibilidad que certifique que la imagen del disco duro virtual se puede usar en Azure Marketplace.

1. Cree e implemente el certificado necesario para administrar de forma remota la máquina virtual en Azure Key Vault.
2. Implemente una máquina virtual de Azure a partir de la imagen de disco duro virtual generalizada creada en [Creación de recursos técnicos de máquina virtual de Azure](create-azure-vm-technical-asset.md).
3. Ejecute las pruebas en la máquina virtual implementada para asegurarse de que la imagen del disco duro virtual esté preparada para publicarse y usarse para implementar máquinas virtuales.

## <a name="running-scripts"></a>Ejecución de scripts

Este artículo contiene tres scripts que se ejecutarán en PowerShell. El escritorio de PowerShell es lo que funciona mejor, de todas formas también puede usarse Azure Cloud Shell con la opción PowerShell seleccionada (parte superior izquierda de la ventana).

1. Asegúrese de que PowerShell está configurado para ejecutar scripts.

    - Abra siempre PowerShell con la opción **Ejecutar como administrador**.
    - Asegúrese de que puede ejecutar estos scripts: `Set-ExecutionPolicy` y `RemoteSigned`.

2. [Instalación de la CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest).

3. Instale el módulo Az de Azure PowerShell.
    1. **Opción A**: Todavía no hay módulos instalados.
        - `Install-Module -Name Az -AllowClobber -Scope AllUsers`

        Para más información, consulte [Instalación del módulo de Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-az-ps?view=azps-4.2.0).

    2. **Opción B**: Actualmente se está usando el módulo AzureRM.

        - Uninstall-AzureRM
        - Install-Module -Name Az -AllowClobber -Scope AllUsers
        - Enable-AzureRmAlias -Scope CurrentUser

        Para más información, vea [Migración de Azure PowerShell de AzureRM a Az](https://docs.microsoft.com/powershell/azure/migrate-from-azurerm-to-az?view=azps-4.2.0).

4. Guarde los parámetros de sesión.

Los scripts de esta sección usan variables o parámetros de sesión. Si cierra la sesión, se borran los parámetros. Se recomienda usar una sesión para ejecutar todos los scripts con el fin de evitar errores de valores de parámetro. Si esto no es posible, tendrá que reinicializar los parámetros al abrir una nueva sesión, especialmente para los scripts posteriores.

## <a name="create-and-deploy-certificates-for-azure-key-vault"></a>Creación e implementación de certificados para Azure Key Vault

En esta sección se describe cómo crear e implementar los certificados autofirmados requeridos para configurar la conectividad de Administración remota de Windows (WinRM) a una máquina virtual hospedada en Azure.

### <a name="create-certificates-for-azure-key-vault"></a>Creación de certificados para Azure Key Vault

Este proceso consta de tres pasos:

1. Cree el certificado de seguridad.
2. Cree la instancia de Azure Key Vault para almacenar el certificado.
3. Almacene los certificados en el almacén de claves.

Puede usar un grupo de recursos de Azure nuevo o existente para este trabajo.

#### <a name="create-the-security-certificate"></a>Creación del certificado de seguridad

Ejecute este script para crear el archivo de certificado (. pfx) en una carpeta local. El certificado pertenece a la máquina virtual de Azure planeada que se implementará a partir de la imagen del disco duro virtual. La máquina virtual necesitará un nombre, una ubicación y una contraseña, tal y como se especifica en los parámetros del script. Edite el siguiente **script de creación de certificados** de Azure PowerShell para especificar los valores correctos para los parámetros de script que se muestran en la tabla.

| **Parámetro** | **Descripción** |
| --- | --- |
| $certroopath | Carpeta local donde guardar el archivo .pfx. |
| $location | Una de las ubicaciones geográficas estándares de Azure. |
| $vmName | Nombre de la máquina virtual de Azure planeada. |
| $certname | Nombre del certificado; debe coincidir con el nombre de dominio completo de la máquina virtual planeada. |
| $certpassword | Contraseña para los certificados; debe coincidir con la contraseña usada para la máquina virtual planeada. |
| | |

```PowerShell
   # Certification creation script

    # pfx certification stored path
    $certroopath = "C:\certLocation"

    # location of the resource group
    $location = "westus"

    # Azure virtual machine name that we are going to create
    $vmName = "testvm000000906"

    # Certification name - should match with FQDN of Windows Azure creating VM
    $certname = "$vmName.$location.cloudapp.azure.com"

    # Certification password - should be match with password of Windows Azure creating VM
    $certpassword = "SecretPassword@123"

    $cert=New-SelfSignedCertificate -DnsName "$certname" -CertStoreLocation cert:\LocalMachine\My
    $pwd = ConvertTo-SecureString -String $certpassword -Force -AsPlainText
    $certwithThumb="cert:\localMachine\my\"+$cert.Thumbprint
    $filepath="$certroopath\$certname.pfx"
    Export-PfxCertificate -cert $certwithThumb -FilePath $filepath -Password $pwd
    Remove-Item -Path $certwithThumb

```

> [!TIP]
> Mantenga la misma sesión de consola de Azure PowerShell abierta y en ejecución durante estos pasos para que se conserven los valores de los distintos parámetros.

> [!WARNING]
> Si guarda este script, almacénelo solo en una ubicación segura, ya que contiene información de seguridad (una contraseña).

#### <a name="create-the-azure-key-vault-to-store-the-certificate"></a>Creación de la instancia de Azure Key Vault para almacenar el certificado

Copie el contenido de la plantilla siguiente en un archivo en el equipo local. En el script de ejemplo siguiente, este recurso es `C:\certLocation\keyvault.json`.

```JSON
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "keyVaultName": {
      "type": "string",
      "defaultValue":"isvkv0001",
      "metadata": {
        "description": "Name of the Vault"
      }
    },
    "tenantId": {
      "type": "string",
      "defaultValue":"72f988bf-86f1-41af-91ab-2d7cd011db47",
      "metadata": {
        "description": "Tenant Id of the subscription. Get using Get-AzureSubscription cmdlet or Get Subscription API"
      }
    },
    "objectId": {
      "type": "string",
      "defaultValue":"d55739bf-d5d6-4ce0-be1c-49ade53c4315",
      "metadata": {
        "description": "Object Id of the AD user. Get using Get-AzureADUser or Get-AzureADServicePrincipal cmdlets"
      }
    },
    "keysPermissions": {
      "type": "array",
      "defaultValue": ["all"],
      "metadata": {
        "description": "Permissions to keys in the vault. Valid values are: all, create, import, update, get, list, delete, backup, restore, encrypt, decrypt, wrapkey, unwrapkey, sign, and verify."
      }
    },
    "secretsPermissions": {
      "type": "array",
      "defaultValue": ["all"],
      "metadata": {
        "description": "Permissions to secrets in the vault. Valid values are: all, get, set, list, and delete."
      }
    },
    "skuName": {
      "type": "string",
      "defaultValue": "Standard",
      "allowedValues": [
        "Standard",
        "Premium"
      ],
      "metadata": {
        "description": "SKU for the vault"
      }
    },
    "enableVaultForDeployment": {
      "type": "bool",
      "defaultValue": true,
      "allowedValues": [
        true,
        false
      ],
      "metadata": {
        "description": "Specifies if the vault is enabled for a VM deployment"
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.KeyVault/vaults",
      "name": "[parameters('keyVaultName')]",
      "apiVersion": "2015-06-01",
      "location": "[resourceGroup().location]",
      "properties": {
        "enabledForDeployment": "[parameters('enableVaultForDeployment')]",
        "tenantId": "[parameters('tenantId')]",
        "accessPolicies": [
          {
            "tenantId": "[parameters('tenantId')]",
            "objectId": "[parameters('objectId')]",
            "permissions": {
              "keys": "[parameters('keysPermissions')]",
              "secrets": "[parameters('secretsPermissions')]"
            }
          }
        ],
        "sku": {
          "name": "[parameters('skuName')]",
          "family": "A"
        }
      }
    }
  ]
}

```

Edite y ejecute el siguiente script de Azure PowerShell para crear una instancia de Azure Key Vault y el grupo de recursos asociado. Reemplace los valores para los parámetros que se muestran en la tabla siguiente.

| **Parámetro** | **Descripción** |
| --- | --- |
| $postfix | Cadena numérica aleatoria adjunta a identificadores de implementación. |
| $rgName | Nombre del grupo de recursos de Azure que se va a crear. |
| $location | Una de las ubicaciones geográficas estándares de Azure. |
| $kvTemplateJson | Ruta de acceso al archivo (keyvault.json) que contiene la plantilla de Resource Manager para el almacén de claves. |
| $kvname | Nombre del nuevo almacén de claves.|
|   |   |

```PowerShell
# Creating Key vault in resource group

    # "Random" number for deployment identifiers
    $postfix = "0101048"

    # Resource group name
    $rgName = "TestRG$postfix"

    # Location of Resource Group
    $location = "westus"

    # Key vault template location
    $kvTemplateJson = "C:\certLocation\keyvault.json"

    # Key vault name
    $kvname = "isvkv$postfix"

    # code snippet to get the Azure user object ID
    try
       {
        $accounts = Get-AzContext
        $accountNum = 0
        $accounts.Account | %{ ++$accountNum; Write-Host $accountNum $_}
        Write-Host "`nPlease select User, e.g. 1:" -ForegroundColor DarkYellow
        [Int] $accountChoice = Read-Host

        While($accountChoice -lt 1 -or $accountChoice -gt $accounts.Length)
        {
            Write-Host "incorrect input" -ForegroundColor Red
            Write-Host "`nPlease select User, e.g. 1:" -ForegroundColor DarkYellow
            [Int] $accountChoice = Read-Host
        }

        $accountSelected = $accounts[$accountChoice-1]
        echo $accountSelected
        $id = $accountSelected.Account

        Write-Host "User $id Selected"
        $myobjectId=(Get-AzADUser -Mail $id).Id
      }
      catch
      {
      Write-Host $_.Exception.Message
      Break
      }

    # code snippet to get Azure User Tenant Id
      # SELECT Subscriptions
        #**************************************
        try
        {
        $subslist=Get-AzSubscription
        for($i=1; $i -le $subslist.Length;$i++)
        {
           Write-Host ($i.ToString() +":"+ $subslist[$i-1].Name)
        }
        Write-Host "`nPlease pick subscription from above, e.g. 1:" -ForegroundColor DarkYellow
        [int] $selectedsub=Read-Host

        While($selectedsub -lt 1 -or $selectedsub -gt $subslist.Length)
        {
            Write-Host "incorrect input" -ForegroundColor Red
            for($i=1; $i -le $subslist.Length;$i++)
             {
              Write-Host ($i.ToString() +":"+ $subslist[$i-1].Name)
             }
            Write-Host "`nPlease pick subscription from above, e.g. 1:" -ForegroundColor DarkYellow
           [int] $selectedsub=Read-Host
        }
        if($selectedsub -ge 1 -and $selectedsub -le $subslist.Length)
        {
        $mysubid=$subslist[$selectedsub-1].Id
        $mysubName=$subslist[$selectedsub-1].Name
        $mytenantId=$subslist[$selectedsub-1].TenantId
        Write-Host "$mysubName selected"
        }
        }
        catch
        {
        Write-Host $_.Exception.Message
        Break
        }

    # Create a resource group
     Write-Host "Creating Resource Group $rgName"
     az group create --name $rgName --location $location
     Write-Host "-----------------------------------"

    # Create key vault and configure access
    New-AzResourceGroupDeployment -Name "kvdeploy$postfix" -ResourceGroupName $rgName -TemplateFile $kvTemplateJson -keyVaultName $kvname -tenantId $mytenantId -objectId $myobjectId

    Set-AzKeyVaultAccessPolicy -VaultName $kvname -ObjectId $myobjectId -PermissionsToKeys Decrypt,Encrypt,UnwrapKey,WrapKey,Verify,Sign,Get,List,Update,Create,Import,Delete,Backup,Restore,Recover,Purge -PermissionsToSecrets Get,List,Set,Delete,Backup,Restore,Recover,Purge

```

#### <a name="store-the-certificates-to-the-key-vault"></a>Almacenamiento de los certificados en el almacén de claves

Almacene los certificados contenidos en el archivo .pfx, en el nuevo almacén de claves mediante este script:

```PowerShell
 $fileName =$certroopath+"\$certname"+".pfx"

     $fileContentBytes = get-content $fileName -Encoding Byte
     $fileContentEncoded = [System.Convert]::ToBase64String($fileContentBytes)

            $jsonObject = @"
    {
    "data": "$filecontentencoded",
    "dataType" :"pfx",
    "password": "$certpassword"
    }
"@
            echo $certpassword
            $jsonObjectBytes = [System.Text.Encoding]::UTF8.GetBytes($jsonObject)
            $jsonEncoded = [System.Convert]::ToBase64String($jsonObjectBytes)
            $secret = ConvertTo-SecureString -String $jsonEncoded -AsPlainText -Force
            $objAzureKeyVaultSecret=Set-AzKeyVaultSecret -VaultName $kvname -Name "ISVSecret$postfix" -SecretValue $secret
            echo $objAzureKeyVaultSecret.Id

```

## <a name="deploy-an-azure-vm-from-your-generalized-vhd-image"></a>Implementación de una máquina virtual de Azure desde la imagen de disco duro virtual generalizada

En esta sección se describe cómo implementar una imagen de VHD generalizada para crear un recurso de máquina virtual de Azure. En este proceso, usaremos la plantilla de Azure Resource Manager proporcionada y el script de Azure PowerShell.

### <a name="prepare-an-azure-resource-manager-template"></a>Preparación de una plantilla de Azure Resource Manager

Copie una de las siguientes plantillas de Azure Resource Manager para la implementación del disco duro virtual (ya sea para Windows o Linux) en un archivo local denominado VHDtoImage.json. El siguiente script solicitará la ubicación en el equipo local para usar este archivo JSON.

#### <a name="for-windows-based-vms"></a>Para máquinas virtuales basadas en Windows

```JSON
{
    "$schema": "https://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "userStorageAccountName": {
            "type": "string"
        },
        "userStorageContainerName": {
            "type": "string",
            "defaultValue": "vhds"
        },
        "dnsNameForPublicIP": {
            "type": "string"
        },
        "adminUserName": {
            "defaultValue": "isv",
            "type": "string"
        },
        "adminPassword": {
            "type": "securestring",
            "defaultValue": "Password@123"
        },
        "osType": {
            "type": "string",
            "defaultValue": "windows",
            "allowedValues": [
                "windows",
                "linux"
            ]
        },
        "subscriptionId": {
            "type": "string"
        },
        "location": {
            "type": "string"
        },
        "vmSize": {
            "type": "string"
        },
        "publicIPAddressName": {
            "type": "string"
        },
        "vmName": {
            "type": "string"
        },
        "virtualNetworkName": {
            "type": "string"
        },
        "nicName": {
            "type": "string"
        },
        "vaultName": {
            "type": "string",
            "metadata": {
                "description": "Name of the KeyVault"
            }
        },
        "vaultResourceGroup": {
            "type": "string",
            "metadata": {
                "description": "Resource Group of the KeyVault"
            }
        },
        "certificateUrl": {
            "type": "string",
            "metadata": {
                "description": "Url of the certificate with version in KeyVault e.g. https://testault.vault.azure.net/secrets/testcert/b621es1db241e56a72d037479xab1r7"
            }
        },
        "vhdUrl": {
            "type": "string",
            "metadata": {
                "description": "VHD Url..."
            }
        }
    },
        "variables": {
            "addressPrefix": "10.0.0.0/16",
            "subnet1Name": "Subnet-1",
            "subnet2Name": "Subnet-2",
            "subnet1Prefix": "10.0.0.0/24",
            "subnet2Prefix": "10.0.1.0/24",
            "publicIPAddressType": "Dynamic",
            "vnetID": "[resourceId('Microsoft.Network/virtualNetworks',parameters('virtualNetworkName'))]",
            "subnet1Ref": "[concat(variables('vnetID'),'/subnets/',variables('subnet1Name'))]",
            "osDiskVhdName": "[concat('http://',parameters('userStorageAccountName'),'.blob.core.windows.net/',parameters('userStorageContainerName'),'/',parameters('vmName'),'osDisk.vhd')]"
        },
        "resources": [
            {
                "apiVersion": "2015-05-01-preview",
                "type": "Microsoft.Network/publicIPAddresses",
                "name": "[parameters('publicIPAddressName')]",
                "location": "[parameters('location')]",
                "properties": {
                    "publicIPAllocationMethod": "[variables('publicIPAddressType')]",
                    "dnsSettings": {
                        "domainNameLabel": "[parameters('dnsNameForPublicIP')]"
                    }
                }
            },
            {
                "apiVersion": "2015-05-01-preview",
                "type": "Microsoft.Network/virtualNetworks",
                "name": "[parameters('virtualNetworkName')]",
                "location": "[parameters('location')]",
                "properties": {
                    "addressSpace": {
                        "addressPrefixes": [
                            "[variables('addressPrefix')]"
                        ]
                    },
                    "subnets": [
                        {
                            "name": "[variables('subnet1Name')]",
                            "properties": {
                                "addressPrefix": "[variables('subnet1Prefix')]"
                            }
                        },
                        {
                            "name": "[variables('subnet2Name')]",
                            "properties": {
                                "addressPrefix": "[variables('subnet2Prefix')]"
                            }
                        }
                    ]
                }
            },
            {
                "apiVersion": "2015-05-01-preview",
                "type": "Microsoft.Network/networkInterfaces",
                "name": "[parameters('nicName')]",
                "location": "[parameters('location')]",
                "dependsOn": [
                    "[concat('Microsoft.Network/publicIPAddresses/', parameters('publicIPAddressName'))]",
                    "[concat('Microsoft.Network/virtualNetworks/', parameters('virtualNetworkName'))]"
                ],
                "properties": {
                    "ipConfigurations": [
                        {
                            "name": "ipconfig1",
                            "properties": {
                                "privateIPAllocationMethod": "Dynamic",
                                "publicIPAddress": {
                                    "id": "[resourceId('Microsoft.Network/publicIPAddresses',parameters('publicIPAddressName'))]"
                                },
                                "subnet": {
                                    "id": "[variables('subnet1Ref')]"
                                }
                            }
                        }
                    ]
                }
            },
            {
                "apiVersion": "2015-06-15",
                "type": "Microsoft.Compute/virtualMachines",
                "name": "[parameters('vmName')]",
                "location": "[parameters('location')]",
                "dependsOn": [
                    "[concat('Microsoft.Network/networkInterfaces/', parameters('nicName'))]"
                ],
                "properties": {
                    "hardwareProfile": {
                        "vmSize": "[parameters('vmSize')]"
                    },
                    "osProfile": {
                        "computername": "[parameters('vmName')]",
                        "adminUsername": "[parameters('adminUsername')]",
                        "adminPassword": "[parameters('adminPassword')]",
                        "secrets": [
                            {
                                "sourceVault": {
                                    "id": "[resourceId(parameters('vaultResourceGroup'), 'Microsoft.KeyVault/vaults', parameters('vaultName'))]"
                                },
                                "vaultCertificates": [
                                    {
                                        "certificateUrl": "[parameters('certificateUrl')]",
                                        "certificateStore": "My"
                                    }
                                ]
                            }
                        ],
                        "windowsConfiguration": {
                            "provisionVMAgent": "true",
                            "winRM": {
                                "listeners": [
                                    {
                                        "protocol": "http"
                                    },
                                    {
                                        "protocol": "https",
                                        "certificateUrl": "[parameters('certificateUrl')]"
                                    }
                                ]
                            },
                            "enableAutomaticUpdates": "true"
                        }
                    },
                    "storageProfile": {
                        "osDisk": {
                            "name": "[concat(parameters('vmName'),'-osDisk')]",
                            "osType": "[parameters('osType')]",
                            "caching": "ReadWrite",
                            "image": {
                                "uri": "[parameters('vhdUrl')]"
                            },
                            "vhd": {
                                "uri": "[variables('osDiskVhdName')]"
                            },
                            "createOption": "FromImage"
                        }
                    },
                    "networkProfile": {
                        "networkInterfaces": [
                            {
                                "id": "[resourceId('Microsoft.Network/networkInterfaces',parameters('nicName'))]"
                            }
                        ]
                    },
                "diagnosticsProfile": {
                    "bootDiagnostics": {
                        "enabled": true,
                        "storageUri": "[concat('http://', parameters('userStorageAccountName'), '.blob.core.windows.net')]"
                    }
                }
                }
            }
        ]
    }

```

#### <a name="for-linux-based-vms"></a>Para máquinas virtuales basadas en Linux

```JSON
{
    "$schema": "https://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "userStorageAccountName": {
            "type": "string"
        },
        "userStorageContainerName": {
            "type": "string",
            "defaultValue": "vhds"
        },
        "dnsNameForPublicIP": {
            "type": "string"
        },
        "adminUserName": {
            "defaultValue": "isv",
            "type": "string"
        },
        "adminPassword": {
            "type": "securestring",
            "defaultValue": "Password@123"
        },
        "osType": {
            "type": "string",
            "defaultValue": "linux",
            "allowedValues": [
                "windows",
                "linux"
            ]
        },
        "subscriptionId": {
            "type": "string"
        },
        "location": {
            "type": "string"
        },
        "vmSize": {
            "type": "string"
        },
        "publicIPAddressName": {
            "type": "string"
        },
        "vmName": {
            "type": "string"
        },
        "virtualNetworkName": {
            "type": "string"
        },
        "nicName": {
            "type": "string"
        },
        "vaultName": {
            "type": "string",
            "metadata": {
                "description": "Name of the KeyVault"
            }
        },
        "vaultResourceGroup": {
            "type": "string",
            "metadata": {
                "description": "Resource Group of the KeyVault"
            }
        },
        "certificateUrl": {
            "type": "string",
            "metadata": {
                "description": "Url of the certificate with version in KeyVault e.g. https://testault.vault.azure.net/secrets/testcert/b621es1db241e56a72d037479xab1r7"
            }
        },
        "vhdUrl": {
            "type": "string",
            "metadata": {
                "description": "VHD Url..."
            }
        }
    },
        "variables": {
            "addressPrefix": "10.0.0.0/16",
            "subnet1Name": "Subnet-1",
            "subnet2Name": "Subnet-2",
            "subnet1Prefix": "10.0.0.0/24",
            "subnet2Prefix": "10.0.1.0/24",
            "publicIPAddressType": "Dynamic",
            "vnetID": "[resourceId('Microsoft.Network/virtualNetworks',parameters('virtualNetworkName'))]",
            "subnet1Ref": "[concat(variables('vnetID'),'/subnets/',variables('subnet1Name'))]",
            "osDiskVhdName": "[concat('http://',parameters('userStorageAccountName'),'.blob.core.windows.net/',parameters('userStorageContainerName'),'/',parameters('vmName'),'osDisk.vhd')]"
        },
        "resources": [
            {
                "apiVersion": "2015-05-01-preview",
                "type": "Microsoft.Network/publicIPAddresses",
                "name": "[parameters('publicIPAddressName')]",
                "location": "[parameters('location')]",
                "properties": {
                    "publicIPAllocationMethod": "[variables('publicIPAddressType')]",
                    "dnsSettings": {
                        "domainNameLabel": "[parameters('dnsNameForPublicIP')]"
                    }
                }
            },
            {
                "apiVersion": "2015-05-01-preview",
                "type": "Microsoft.Network/virtualNetworks",
                "name": "[parameters('virtualNetworkName')]",
                "location": "[parameters('location')]",
                "properties": {
                    "addressSpace": {
                        "addressPrefixes": [
                            "[variables('addressPrefix')]"
                        ]
                    },
                    "subnets": [
                        {
                            "name": "[variables('subnet1Name')]",
                            "properties": {
                                "addressPrefix": "[variables('subnet1Prefix')]"
                            }
                        },
                        {
                            "name": "[variables('subnet2Name')]",
                            "properties": {
                                "addressPrefix": "[variables('subnet2Prefix')]"
                            }
                        }
                    ]
                }
            },
            {
                "apiVersion": "2015-05-01-preview",
                "type": "Microsoft.Network/networkInterfaces",
                "name": "[parameters('nicName')]",
                "location": "[parameters('location')]",
                "dependsOn": [
                    "[concat('Microsoft.Network/publicIPAddresses/', parameters('publicIPAddressName'))]",
                    "[concat('Microsoft.Network/virtualNetworks/', parameters('virtualNetworkName'))]"
                ],
                "properties": {
                    "ipConfigurations": [
                        {
                            "name": "ipconfig1",
                            "properties": {
                                "privateIPAllocationMethod": "Dynamic",
                                "publicIPAddress": {
                                    "id": "[resourceId('Microsoft.Network/publicIPAddresses',parameters('publicIPAddressName'))]"
                                },
                                "subnet": {
                                    "id": "[variables('subnet1Ref')]"
                                }
                            }
                        }
                    ]
                }
            },
            {
                "apiVersion": "2015-06-15",
                "type": "Microsoft.Compute/virtualMachines",
                "name": "[parameters('vmName')]",
                "location": "[parameters('location')]",
                "dependsOn": [
                    "[concat('Microsoft.Network/networkInterfaces/', parameters('nicName'))]"
                ],
                "properties": {
                    "hardwareProfile": {
                        "vmSize": "[parameters('vmSize')]"
                    },
                    "osProfile": {
                        "computername": "[parameters('vmName')]",
                        "adminUsername": "[parameters('adminUsername')]",
                        "adminPassword": "[parameters('adminPassword')]",
                        "secrets": [
                            {
                                "sourceVault": {
                                    "id": "[resourceId(parameters('vaultResourceGroup'), 'Microsoft.KeyVault/vaults', parameters('vaultName'))]"
                                },
                                "vaultCertificates": [
                                    {
                                        "certificateUrl": "[parameters('certificateUrl')]",
                                        "certificateStore": "My"
                                    }
                                ]
                            }
                        ],
                        "windowsConfiguration": {
                            "provisionVMAgent": "true",
                            "winRM": {
                                "listeners": [
                                    {
                                        "protocol": "http"
                                    },
                                    {
                                        "protocol": "https",
                                        "certificateUrl": "[parameters('certificateUrl')]"
                                    }
                                ]
                            },
                            "enableAutomaticUpdates": "true"
                        }
                    },
                    "storageProfile": {
                        "osDisk": {
                            "name": "[concat(parameters('vmName'),'-osDisk')]",
                            "osType": "[parameters('osType')]",
                            "caching": "ReadWrite",
                            "image": {
                                "uri": "[parameters('vhdUrl')]"
                            },
                            "vhd": {
                                "uri": "[variables('osDiskVhdName')]"
                            },
                            "createOption": "FromImage"
                        }
                    },
                    "networkProfile": {
                        "networkInterfaces": [
                            {
                                "id": "[resourceId('Microsoft.Network/networkInterfaces',parameters('nicName'))]"
                            }
                        ]
                    },
                "diagnosticsProfile": {
                    "bootDiagnostics": {
                        "enabled": false,
                        "storageUri": "[concat('http://', parameters('userStorageAccountName'), '.blob.core.windows.net')]"
                    }
                }
                }
            }
        ]
    }

```

Copie y edite el siguiente script para proporcionar valores para estos parámetros:

| **Parámetro** | **Descripción** |
| --- | --- |
| ResourceGroupName | El nombre del grupo de recursos de Azure existente. Normalmente, use el mismo grupo de recursos que el almacén de claves. |
| TemplateFile | Ruta de acceso completa al archivo VHDtoImage.json. |
| userStorageAccountName | Nombre de la cuenta de almacenamiento. |
| sNameForPublicIP | Nombre DNS para la dirección IP pública; debe estar en minúscula. |
| subscriptionId | Identificador de la suscripción de Azure. |
| Location | Ubicación geográfica de Azure estándar del grupo de recursos. |
| vmName | Nombre de la máquina virtual. |
| vaultName | Nombre del almacén de claves. |
| vaultResourceGroup | Grupo de recursos del almacén de claves. |
| certificateUrl | Dirección web (URL) del certificado, incluida la versión almacenada en el almacén de claves; por ejemplo: `https://testault.vault.azure.net/secrets/testcert/b621es1db241e56a72d037479xab1r7`. |
| vhdUrl | Dirección web del disco duro virtual. |
| vmSize | Tamaño de la instancia de máquina virtual. |
| publicIPAddressName | Nombre de la dirección IP pública. |
| virtualNetworkName | Nombre de la red virtual. |
| nicName | Nombre de la tarjeta de interfaz de red de la red virtual. |
| adminUserName | Nombre de usuario de la cuenta de administrador. |
| adminPassword | Contraseña de administrador. |
|   |   |

### <a name="deploy-an-azure-vm"></a>Implementación de una máquina virtual de Azure

Copie y edite el script siguiente para proporcionar valores para las variables `$storageaccount` y `$vhdUrl`. Ejecútelo para crear un recurso de máquina virtual de Azure a partir de su disco duro virtual generalizado existente.

```PowerShell

# storage account of existing generalized VHD

$storageaccount = "testwinrm11815"

# generalized VHD URL
$vhdUrl = "https://testwinrm11815.blob.core.windows.net/vhds/testvm1234562016651857.vhd"

# Full pathname to the file VHDtoImage.json. inserted these highlighted lines
$templateFile = "$certroopath\VHDtoImage.json"

# Size of the virtual machine instance.
$vmSize = "Standard_D2s_v3"

# Name of the public IP address.
$publicIPAddressName = "myPublicIP1"

# Name of the virtual network
$virtualNetworkName = "myVNET1"

# Name of the network interface card for the virtual network
$nicName = "myNIC1"

# Username of the administrator account
$adminUserName = "isv"

# The OS of the virtual machine
$osType = "windows"

echo "New-AzResourceGroupDeployment -Name "dplisvvm$postfix" -ResourceGroupName "$rgName" -TemplateFile $templateFile -userStorageAccountName "$storageaccount" -dnsNameForPublicIP "$vmName" -subscriptionId "$mysubid" -location "$location" -vmName "$vmName" -vaultName "$kvname" -vaultResourceGroup "$rgName" -certificateUrl $objAzureKeyVaultSecret.Id  -vhdUrl "$vhdUrl" -vmSize "$vmSize" -publicIPAddressName "$publicIPAddressName" -virtualNetworkName "$virtualNetworkName" -nicName "$nicName" -adminUserName "$adminUserName" -adminPassword $pwd -osType "$osType""

# deploying VM with existing VHD
New-AzResourceGroupDeployment -Name "dplisvvm$postfix" -ResourceGroupName "$rgName" -TemplateFile $templateFile -userStorageAccountName "$storageaccount" -dnsNameForPublicIP "$vmName" -subscriptionId "$mysubid" -location "$location" -vmName "$vmName" -vaultName "$kvname" -vaultResourceGroup "$rgName" -certificateUrl $objAzureKeyVaultSecret.Id -vhdUrl "$vhdUrl" -vmSize "$vmSize" -publicIPAddressName "$publicIPAddressName" -virtualNetworkName "$virtualNetworkName" -nicName "$nicName" -adminUserName "$adminUserName" -adminPassword $pwd -osType "$osType"

```

## <a name="run-tests-on-the-deployed-vm"></a>Ejecución de pruebas en la máquina virtual implementada

### <a name="download-and-run-the-certification-test-tool"></a>Descargar y ejecutar Certification Test Tool

Certification Test Tool for Azure Certified es una herramienta de prueba automática que se ejecuta en una máquina Windows local, pero prueba una máquina virtual Linux o Windows basada en Azure. Verifica que la imagen de máquina virtual de usuario se puede usar con Microsoft Azure y que se han cumplido las instrucciones y los requisitos relacionados con la preparación del VHD. Esta herramienta garantiza que la máquina virtual está preparada para publicarse de acuerdo a los requisitos de Azure Marketplace.

1. Descargue e instale la versión más reciente de [Certification Test Tool for Azure Certified](https://www.microsoft.com/download/details.aspx?id=44299).
2. Abra la herramienta de certificación y seleccione **Iniciar nueva prueba**.
3. En la pantalla **Información de la prueba**, escriba un **Nombre de prueba** para la serie de pruebas.
4. Seleccione la **Plataforma** de la máquina virtual, ya sea Windows Server o Linux. La elección de plataforma afecta a las opciones restantes.
5. Si la máquina virtual usa este servicio de base de datos, seleccione la casilla **Prueba de Azure SQL Database**.

### <a name="connect-the-certification-tool-to-a-vm-image"></a>Conectar la herramienta de certificación a una imagen de máquina virtual

La herramienta se conecta a máquinas virtuales basadas en Windows con [Azure PowerShell](https://docs.microsoft.com/powershell/) y a máquinas virtuales basadas en Linux mediante [SSH.Net](https://www.ssh.com/ssh/protocol/). Elija una de las dos opciones siguientes, Linux o Windows.

#### <a name="option-1-connect-the-certification-tool-to-a-linux-vm-image"></a>Opción 1: Conectar la herramienta de certificación a una imagen de máquina virtual Linux

1. Seleccione el modo **Autenticación SSH**: Autenticación de contraseña o autenticación de archivo de clave.
2. Si usa la autenticación de contraseña, escriba los valores para **Nombre DNS de máquina virtual**, **Nombre de usuario** y **Contraseña**. También puede cambiar el número predeterminado de **Puerto SSH**.

    ![Herramienta de prueba de certificados de Azure, autenticación de contraseña de la imagen de máquina virtual Linux](media/avm-cert2.png)

3. Si usa autenticación con archivo de clave, escriba valores para **Nombre DNS de máquina virtual**, **Nombre de usuario** y **Ubicación de clave privada**. También puede incluir una **Frase de contraseña** o cambiar el número predeterminado de **Puerto SSH**.

#### <a name="option-2-connect-the-certification-tool-to-a-windows-based-vm-image"></a>Opción 2: Conectar la herramienta de certificación a una imagen de máquina virtual basada en Windows

1. Escriba el **nombre DNS completo de la máquina virtual** (por ejemplo, MyVMName.ClOudapp.net).
2. Escriba valores para **Nombre de usuario** y **Contraseña**.

    ![Herramienta de prueba de certificados de Azure, autenticación de contraseña de la imagen de máquina virtual basada en Windows](media/avm-cert4.png)

### <a name="run-a-certification-test"></a>Ejecutar una prueba de certificación

Una vez proporcionados los valores de parámetro para la imagen de máquina virtual en la herramienta de certificación, seleccione **Probar conexión** para crear una conexión válida a la máquina virtual. Una vez verificada la conexión, seleccione **Siguiente** para iniciar la prueba. Una vez completada la prueba, los resultados de la prueba se muestran en una tabla. La columna Estado se muestra (Correcto/Error/Advertencia) para cada prueba. Si se produce un error en alguna de las pruebas, la imagen _no_ se certifica. En ese caso, revise los requisitos y los mensajes de error, realice los cambios sugeridos y vuelva a ejecutar la prueba.

Una vez completada la prueba automatizada, proporcione información adicional sobre la imagen de máquina virtual en las dos pestañas de la pantalla **Cuestionario**, es decir, **Evaluación general** y **Personalización del kernel**, y, después, seleccione **Siguiente**.

La última pantalla permite proporcionar más información, como datos de acceso SSH para una imagen de máquina virtual Linux y una explicación sobre las evaluaciones con errores, en caso de que busque excepciones.

Por último, seleccione **Generar informe** para descargar los resultados de las pruebas y los archivos de registro de los casos de prueba ejecutados, junto con las respuestas al cuestionario. Guarde los resultados en el mismo contenedor que los VHD.

## <a name="next-step"></a>Paso siguiente

- [Problemas comunes y correcciones de los identificadores URI de SAS](common-sas-uri-issues.md)
