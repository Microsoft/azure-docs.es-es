---
title: 'Validación de una imagen de máquina virtual de Azure: Azure Marketplace'
description: Obtenga información sobre cómo probar y enviar una oferta de máquina virtual en Marketplace comercial.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: iqshahmicrosoft
ms.author: iqshah
ms.date: 08/14/2020
ms.openlocfilehash: 2d19098ec82fe9361154d798b981341a86decf97
ms.sourcegitcommit: 3be3537ead3388a6810410dfbfe19fc210f89fec
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89647828"
---
# <a name="azure-virtual-machine-image-validation"></a>Validación de una imagen de máquina virtual de Azure

En este artículo se describe cómo probar y enviar una imagen de máquina virtual (VM) en Marketplace comercial para asegurarse de que cumple los requisitos de publicación más recientes de Azure Marketplace.

Siga estos pasos antes de enviar la oferta de máquina virtual:

- Implemente una máquina virtual de Azure con la imagen generalizada.
- Ejecute las validaciones.

## <a name="deploy-an-azure-vm-using-your-generalized-image"></a>Implementación de una máquina virtual de Azure con la imagen generalizada

En esta sección se describe cómo implementar una imagen de disco duro virtual (VHD) generalizada para crear un recurso de máquina virtual de Azure. En este proceso, usaremos la plantilla de Azure Resource Manager proporcionada y el script de Azure PowerShell.

### <a name="prepare-an-azure-resource-manager-template"></a>Preparación de una plantilla de Azure Resource Manager

En esta sección se describe cómo crear e implementar una imagen de máquina virtual (VM) proporcionada por el usuario. Para ello, proporcione imágenes del VHD del sistema operativo y del disco de datos desde un disco duro virtual implementado en Azure. En estos pasos se implementa la máquina virtual mediante un VHD generalizado.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. Cargue el VHD del sistema operativo generalizado y los VHD del disco de datos en su cuenta de Azure Storage.
3. En la página principal, seleccione **Crear un recurso**, busque "Template Deployment" y seleccione **Crear**.
4. Seleccione **Cree su propia plantilla en el editor**.

    :::image type="content" source="media/vm/template-deployment.png" alt-text="Muestra la selección de una plantilla.":::

5. Pegue la siguiente plantilla JSON en el editor y seleccione **Guardar**.

```JSON
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "userStorageAccountName": {
            "type": "String"
        },
        "userStorageContainerName": {
            "defaultValue": "vhds",
            "type": "String"
        },
        "dnsNameForPublicIP": {
            "type": "String"
        },
        "adminUserName": {
            "defaultValue": "isv",
            "type": "String"
        },
        "adminPassword": {
            "defaultValue": "",
            "type": "SecureString"
        },
        "osType": {
            "defaultValue": "windows",
            "allowedValues": [
                "windows",
                "linux"
            ],
            "type": "String"
        },
        "subscriptionId": {
            "type": "String"
        },
        "location": {
            "type": "String"
        },
        "vmSize": {
            "type": "String"
        },
        "publicIPAddressName": {
            "type": "String"
        },
        "vmName": {
            "type": "String"
        },
        "virtualNetworkName": {
            "type": "String"
        },
        "nicName": {
            "type": "String"
        },
        "vhdUrl": {
            "type": "String",
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
        "hostDNSNameScriptArgument": "[concat(parameters('dnsNameForPublicIP'),'.',parameters('location'),'.cloudapp.azure.com')]",
        "osDiskVhdName": "[concat('http://',parameters('userStorageAccountName'),'.blob.core.windows.net/',parameters('userStorageContainerName'),'/',parameters('vmName'),'osDisk.vhd')]"
    },
    "resources": [
        {
            "type": "Microsoft.Network/publicIPAddresses",
            "apiVersion": "2015-06-15",
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
            "type": "Microsoft.Network/virtualNetworks",
            "apiVersion": "2015-06-15",
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
            "type": "Microsoft.Network/networkInterfaces",
            "apiVersion": "2015-06-15",
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
            "type": "Microsoft.Compute/virtualMachines",
            "apiVersion": "2015-06-15",
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
                    "adminPassword": "[parameters('adminPassword')]"
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
                }
            }
        },
        {
            "type": "Microsoft.Compute/virtualMachines/extensions",
            "apiVersion": "2015-06-15",
            "name": "[concat(parameters('vmName'),'/WinRMCustomScriptExtension')]",
            "location": "[parameters('location')]",
            "dependsOn": [
                "[concat('Microsoft.Compute/virtualMachines/', parameters('vmName'))]"
            ],
            "properties": {
                "publisher": "Microsoft.Compute",
                "type": "CustomScriptExtension",
                "typeHandlerVersion": "1.4",
                "settings": {
                    "fileUris": [
                        "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-vm-winrm-windows/ConfigureWinRM.ps1",
                        "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-vm-winrm-windows/makecert.exe",
                        "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-vm-winrm-windows/winrmconf.cmd"
                    ],
                    "commandToExecute": "[concat('powershell -ExecutionPolicy Unrestricted -file ConfigureWinRM.ps1 ',variables('hostDNSNameScriptArgument'))]"
                }
            }
        }
    ]
}
```

<br>

6. Proporcione los valores de parámetro de las páginas de propiedades Implementación personalizada que se muestran.

| ResourceGroupName | El nombre del grupo de recursos de Azure existente. Normalmente, use el mismo grupo de recursos que el almacén de claves. |
| --- | --- |
| TemplateFile | Ruta de acceso completa al archivo VHDtoImage.json. |
| userStorageAccountName | Nombre de la cuenta de almacenamiento. |
| dnsNameForPublicIP | Nombre DNS para la dirección IP pública; debe estar en minúscula. |
| subscriptionId | Identificador de la suscripción de Azure. |
| Location | Ubicación geográfica de Azure estándar del grupo de recursos. |
| vmName | Nombre de la máquina virtual. |
| vhdUrl | Dirección web del disco duro virtual. |
| vmSize | Tamaño de la instancia de máquina virtual. |
| publicIPAddressName | Nombre de la dirección IP pública. |
| virtualNetworkName | Nombre de la red virtual. |
| nicName | Nombre de la tarjeta de interfaz de red de la red virtual. |
| adminUserName | Nombre de usuario de la cuenta de administrador. |
| adminPassword | Contraseña de administrador. |
|

7. Después de proporcionar estos valores, seleccione **Comprar**.

Azure comenzará la implementación. Crea una máquina virtual con el VHD no administrado especificado en la ruta de acceso de la cuenta de almacenamiento especificada. Puede realizar un seguimiento del progreso en Azure Portal si selecciona **Máquinas virtuales** en el lado izquierdo del portal. Una vez creada la máquina virtual, el estado cambiará de Iniciando a En ejecución.

#### <a name="for-generation-2-vm-deployment-use-this-template"></a>Para la implementación de máquinas virtuales de generación 2, use esta plantilla:

```JSON
{
   "$schema":"https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
   "contentVersion":"1.0.0.0",
   "parameters":{
      "userStorageAccountName":{
         "type":"String"
      },
      "userStorageContainerName":{
         "type":"String",
         "defaultValue":"vhds-86350-720-f4efbbb2-611b-4cd7-ad1e-afdgfuadfluad"
      },
      "dnsNameForPublicIP":{
         "type":"String"
      },
      "adminUserName":{
         "defaultValue":"isv",
         "type":"String"
      },
      "adminPassword":{
         "type":"securestring",
         "defaultValue":"Certcare@86350"
      },
      "osType":{
         "type":"string",
         "defaultValue":"linux",
         "allowedValues":[
            "windows",
            "linux"
         ]
      },
      "subscriptionId":{
         "type":"string"
      },
      "location":{
         "type":"string"
      },
      "vmSize":{
         "type":"string"
      },
      "publicIPAddressName":{
         "type":"string"
      },
      "vmName":{
         "type":"string"
      },
      "vNetNewOrExisting":{
         "defaultValue":"existing",
         "allowedValues":[
            "new",
            "existing"
         ],
         "type":"String",
         "metadata":{
            "description":"Specify whether to create a new or existing virtual network for the VM."
         }
      },
      "virtualNetworkName":{
         "type":"String",
         "defaultValue":""
      },
      "SubnetName":{
         "defaultValue":"subnet-5",
         "type":"String"
      },
      "SubnetPrefix":{
         "defaultValue":"10.0.5.0/24",
         "type":"String"
      },
      "nicName":{
         "type":"string"
      },
      "vhdUrl":{
         "type":"string",
         "metadata":{
            "description":"VHD Url..."
         }
      },
      "Datadisk1":{
         "type":"string",
         "metadata":{
            "description":"datadisk1 Url..."
         }
      },
      "Datadisk2":{
         "type":"string",
         "metadata":{
            "description":"datadisk2 Url..."
         }
      },
      "Datadisk3":{
         "type":"string",
         "metadata":{
            "description":"datadisk2 Url..."
         }
      }
   },
   "variables":{
      "addressPrefix":"10.0.0.0/16",
      "subnet1Name":"[parameters('SubnetName')]",
      "subnet1Prefix":"[parameters('SubnetPrefix')]",
      "publicIPAddressType":"Static",
      "vnetID":"[resourceId('Microsoft.Network/virtualNetworks',parameters('virtualNetworkName'))]",
      "subnet1Ref":"[concat(variables('vnetID'),'/subnets/',variables('subnet1Name'))]",
      "osDiskVhdName":"[concat('http://',parameters('userStorageAccountName'),'.blob.core.windows.net/',parameters('userStorageContainerName'),'/',parameters('vmName'),'osDisk.vhd')]",
      "dataDiskVhdName":"[concat('http://',parameters('userStorageAccountName'),'.blob.core.windows.net/',parameters('userStorageContainerName'),'/',parameters('vmName'),'datadisk')]",
      "imageName":"[concat(parameters('vmName'), '-image')]"
   },
   "resources":[
      {
         "apiVersion":"2015-05-01-preview",
         "type":"Microsoft.Network/publicIPAddresses",
         "name":"[parameters('publicIPAddressName')]",
         "location":"[parameters('location')]",
         "properties":{
            "publicIPAllocationMethod":"[variables('publicIPAddressType')]",
            "dnsSettings":{
               "domainNameLabel":"[parameters('dnsNameForPublicIP')]"
            }
         }
      },
      {
         "type":"Microsoft.Compute/images",
         "apiVersion":"2019-12-01",
         "name":"[variables('imageName')]",
         "location":"[parameters('location')]",
         "properties":{
            "storageProfile":{
               "osDisk":{
                  "osType":"[parameters('osType')]",
                  "osState":"Generalized",
                  "blobUri":"[parameters('vhdUrl')]",
                  "storageAccountType":"Standard_LRS"
               },
               "dataDisks":[
                  {
                     "lun":0,
                     "blobUri":"[parameters('Datadisk1')]",
                     "storageAccountType":"Standard_LRS"
                  },
                  {
                     "lun":1,
                     "blobUri":"[parameters('Datadisk2')]",
                     "storageAccountType":"Standard_LRS"
                  },
                  {
                     "lun":2,
                     "blobUri":"[parameters('Datadisk3')]",
                     "storageAccountType":"Standard_LRS"
                  }
               ]
            },
            "hyperVGeneration":"V2"
         }
      },
      {
         "apiVersion":"2015-05-01-preview",
         "type":"Microsoft.Network/virtualNetworks",
         "name":"[parameters('virtualNetworkName')]",
         "location":"[parameters('location')]",
         "properties":{
            "addressSpace":{
               "addressPrefixes":[
                  "[variables('addressPrefix')]"
               ]
            },
            "subnets":[
               {
                  "name":"[variables('subnet1Name')]",
                  "properties":{
                     "addressPrefix":"[variables('subnet1Prefix')]"
                  }
               }
            ]
         },
         "condition":"[equals(parameters('vNetNewOrExisting'), 'new')]"
      },
      {
         "apiVersion":"2016-09-01",
         "type":"Microsoft.Network/networkInterfaces",
         "name":"[parameters('nicName')]",
         "location":"[parameters('location')]",
         "dependsOn":[
            "[concat('Microsoft.Network/publicIPAddresses/', parameters('publicIPAddressName'))]",
            "[concat('Microsoft.Network/virtualNetworks/', parameters('virtualNetworkName'))]"
         ],
         "properties":{
            "ipConfigurations":[
               {
                  "name":"ipconfig1",
                  "properties":{
                     "privateIPAllocationMethod":"Dynamic",
                     "publicIPAddress":{
                        "id":"[resourceId('Microsoft.Network/publicIPAddresses',parameters('publicIPAddressName'))]"
                     },
                     "subnet":{
                        "id":"[variables('subnet1Ref')]"
                     }
                  }
               }
            ]
         }
      },
      {
         "apiVersion":"2019-12-01",
         "type":"Microsoft.Compute/virtualMachines",
         "name":"[parameters('vmName')]",
         "location":"[parameters('location')]",
         "dependsOn":[
            "[concat('Microsoft.Network/networkInterfaces/', parameters('nicName'))]",
            "[variables('imageName')]"
         ],
         "properties":{
            "hardwareProfile":{
               "vmSize":"[parameters('vmSize')]"
            },
            "osProfile":{
               "computername":"[parameters('vmName')]",
               "adminUsername":"[parameters('adminUsername')]",
               "adminPassword":"[parameters('adminPassword')]"
            },
            "storageProfile":{
               "imageReference":{
                  "id":"[resourceId('Microsoft.Compute/images', variables('imageName'))]"
               },
               "dataDisks":[
                  {
                     "name":"[concat(parameters('vmName'),'_DataDisk0')]",
                     "lun":0,
                     "createOption":"FromImage",
                     "managedDisk":{
                        "storageAccountType":"Standard_LRS"
                     }
                  },
                  {
                     "name":"[concat(parameters('vmName'),'_DataDisk1')]",
                     "lun":1,
                     "createOption":"FromImage",
                     "managedDisk":{
                        "storageAccountType":"Standard_LRS"
                     }
                  },
                  {
                     "name":"[concat(parameters('vmName'),'_DataDisk2')]",
                     "lun":2,
                     "createOption":"FromImage",
                     "managedDisk":{
                        "storageAccountType":"Standard_LRS"
                     }
                  }
               ],
               "osDisk":{
                  "name":"[concat(parameters('vmName'),'-OSDisk')]",
                  "createOption":"FromImage",
                  "managedDisk":{
                     "storageAccountType":"Standard_LRS"
                  }
               }
            },
            "networkProfile":{
               "networkInterfaces":[
                  {
                     "id":"[resourceId('Microsoft.Network/networkInterfaces',parameters('nicName'))]"
                  }
               ]
            }
         }
      }
   ]
}
```

### <a name="deploy-an-azure-vm-using-powershell"></a>Implementación de una máquina virtual de Azure mediante PowerShell

Copie y edite el script siguiente para proporcionar valores para las variables `$storageaccount` y `$vhdUrl`. Ejecútelo para crear un recurso de máquina virtual de Azure a partir de su disco duro virtual generalizado existente.

```PowerShell
# storage account of existing generalized VHD

$storageaccount = "testwinrm11815" # generalized VHD URL
$vhdUrl = "https://testwinrm11815.blob.core.windows.net/vhds/testvm1234562016651857.vhd"

echo "New-AzResourceGroupDeployment -Name "dplisvvm$postfix" -ResourceGroupName "$rgName" -TemplateFile "C:\certLocation\VHDtoImage.json" -userStorageAccountName "$storageaccount" -dnsNameForPublicIP "$vmName" -subscriptionId "$mysubid" -location "$location" -vmName "$vmName" -vaultName "$kvname" -vaultResourceGroup "$rgName" -certificateUrl
$objAzureKeyVaultSecret.Id -vhdUrl "$vhdUrl" -vmSize "Standard\_A2" -publicIPAddressName "myPublicIP1" -virtualNetworkName "myVNET1" -nicName "myNIC1" -adminUserName "isv" -adminPassword $pwd"

# deploying VM with existing VHD

New-AzResourceGroupDeployment -Name"dplisvvm$postfix" -ResourceGroupName"$rgName" -TemplateFile"C:\certLocation\VHDtoImage.json" - userStorageAccountName"$storageaccount" -dnsNameForPublicIP"$vmName" -subscriptionId"$mysubid" -location"$location" - vmName"$vmName" -vaultName"$kvname" -vaultResourceGroup"$rgName" -certificateUrl$objAzureKeyVaultSecret.Id -vhdUrl"$vhdUrl" - vmSize"Standard\_A2" -publicIPAddressName"myPublicIP1" -virtualNetworkName"myVNET1" -nicName"myNIC1" -adminUserName"isv" - adminPassword$pwd
```

## <a name="run-validations"></a>Ejecución de validaciones

Hay dos maneras de ejecutar validaciones en la imagen implementada.

### <a name="use-certification-test-tool-for-azure-certified"></a>Usar Certification Test Tool for Azure Certified

#### <a name="download-and-run-the-certification-test-tool"></a>Descargar y ejecutar Certification Test Tool

Certification Test Tool for Azure Certified se ejecuta en un equipo Windows local, pero prueba una máquina virtual Linux o Windows basada en Azure. Verifica que la imagen de máquina virtual de usuario se puede usar con Microsoft Azure y que se han cumplido las instrucciones y los requisitos relacionados con la preparación del VHD.

1. Descargue e instale la versión más reciente de [Certification Test Tool for Azure Certified](https://www.microsoft.com/download/details.aspx?id=44299).
2. Abra la herramienta de certificación y seleccione **Iniciar nueva prueba**.
3. En la pantalla Información de la prueba, escriba un **Nombre de prueba** para la serie de pruebas.
4. Seleccione la plataforma de la máquina virtual, ya sea **Windows Server** o **Linux**. La elección de plataforma afecta a las opciones restantes.
5. Si la máquina virtual usa este servicio de base de datos, seleccione la casilla **Prueba de Azure SQL Database**.

#### <a name="connect-the-certification-tool-to-a-vm-image"></a>Conectar la herramienta de certificación a una imagen de máquina virtual

1. Seleccione el modo Autenticación SSH: Autenticación de contraseña o autenticación de archivo de clave.
2. Si usa la autenticación de contraseña, escriba los valores para **Nombre DNS de máquina virtual**, **Nombre de usuario** y **Contraseña**. También puede cambiar el número predeterminado de Puerto SSH.

    :::image type="content" source="media/vm/azure-vm-cert-2.png" alt-text="Muestra la selección de la información de la prueba de la máquina virtual.":::

3. Si usa autenticación con archivo de clave, escriba valores para Nombre DNS de máquina virtual, Nombre de usuario y Ubicación de clave privada. También puede incluir una Frase de contraseña o cambiar el número predeterminado de Puerto SSH.
4. Escriba el nombre DNS completo de la máquina virtual (por ejemplo, MyVMName.ClOudapp.net).
5. Escriba el **Nombre de usuario** y la **Contraseña**.

    :::image type="content" source="media/vm/azure-vm-cert-4.png" alt-text="Muestra la selección del nombre de usuario y contraseña de la máquina virtual.":::

6. Seleccione **Next** (Siguiente).

#### <a name="run-a-certification-test"></a>Ejecutar una prueba de certificación

Una vez proporcionados los valores de parámetro para la imagen de máquina virtual en la herramienta de certificación, seleccione Probar conexión para crear una conexión válida a la máquina virtual. Una vez verificada la conexión, seleccione **Siguiente** para iniciar la prueba. Una vez completada la prueba, los resultados se muestran en una tabla. La columna Estado se muestra (Correcto/Error/Advertencia) para cada prueba. Si se produce un error en alguna de las pruebas, la imagen no se certifica. En ese caso, revise los requisitos y los mensajes de error, realice los cambios sugeridos y vuelva a ejecutar la prueba.

Una vez completada la prueba automatizada, proporcione información adicional sobre la imagen de máquina virtual en las dos pestañas de la pantalla Cuestionario, es decir, Evaluación general y Personalización del kernel, y, después, seleccione Siguiente.

La última pantalla permite proporcionar más información, como datos de acceso SSH para una imagen de máquina virtual Linux y una explicación sobre las evaluaciones con errores, en caso de que busque excepciones.

Por último, seleccione Generar informe para descargar los resultados de las pruebas y los archivos de registro de los casos de prueba ejecutados, junto con las respuestas al cuestionario. Guarde los resultados en el mismo contenedor que los VHD.

## <a name="how-to-use-powershell-to-consume-the-self-test-api"></a>Cómo usar PowerShell para consumir la API de prueba automática

### <a name="on-linux-os"></a>En el SO Linux

Llame a la API en PowerShell:

1. Use el comando Invoke-WebRequest para llamar a la API.
2. El método es Post y el tipo de contenido es JSON, tal como se muestra en la siguiente captura de pantalla y el ejemplo de código.
3. Especifique los parámetros del cuerpo en formato JSON.

El ejemplo siguiente muestra una llamada de PowerShell a la API:

```POWERSHELL
$accesstoken = “token”
$headers = New-Object “System.Collections.Generic.Dictionary[[String],[String]]”
$headers.Add(“Authorization”, “Bearer $accesstoken”)
$DNSName = “\&lt;\&lt;Machine DNS Name\&gt;\&gt;”
$UserName = “\&lt;\&lt;User ID\&gt;\&gt;”
$Password = “\&lt;\&lt;Password\&gt;\&gt;”
$OS = “Linux”
$PortNo = “22”
$CompanyName = “ABCD”
$AppID = “\&lt;\&lt;Application ID\&gt;\&gt;”
$TenantId = “\&lt;\&lt;Tenant ID\&gt;\&gt;”

$body =
@{
DNSName = $DNSName
UserName = $UserName
Password = $Password
OS = $OS
PortNo = $PortNo
CompanyName = $CompanyName
AppID = $AppID
TenantId = $TenantId
}| ConvertTo-Json

$body

$uri = “URL”

$res = (Invoke-WebRequest -Method “Post” -Uri $uri -Body $body -ContentType “application/json” -Headers $headers).Content
```

<br>Este es un ejemplo de llamada a la API en PowerShell:

[![Ejemplo de pantalla de una llamada a la API en PowerShell.](media/vm/call-api-in-powershell.png)](media/vm/call-api-in-powershell.png#lightbox)

<br>Mediante el ejemplo anterior, puede recuperar el archivo JSON y analizarlo para obtener los detalles siguientes:

```PowerShell
$resVar=$res|ConvertFrom-Json

$actualresult =$resVar.Response |ConvertFrom-Json

Write-Host”OSName: $($actualresult.OSName)”Write-Host”OSVersion: $($actualresult.OSVersion)”Write-Host”Overall Test Result: $($actualresult.TestResult)”For ($i=0; $i -lt$actualresult.Tests.Length; $i++){ Write-Host”TestID: $($actualresult.Tests[$i].TestID)”Write-Host”TestCaseName: $($actualresult.Tests[$i].TestCaseName)”Write-Host”Description: $($actualresult.Tests[$i].Description)”Write-Host”Result: $($actualresult.Tests[$i].Result)”Write-Host”ActualValue: $($actualresult.Tests[$i].ActualValue)”}
```

<br>En esta pantalla de ejemplo, que muestra `$res.Content`, se pueden ver los detalles de los resultados de las pruebas en formato JSON:

[![Ejemplo de pantalla de una llamada a la API en PowerShell con los detalles de los resultados de la prueba.](media/vm/call-api-in-powershell-details.png)](media/vm/call-api-in-powershell-details.png#lightbox)

<br>Este es un ejemplo de los resultados JSON de las pruebas que se ven en un visor JSON en línea (por ejemplo, [Code Beautify](https://codebeautify.org/jsonviewer) o [JSON Viewer](https://jsonformatter.org/json-viewer)).

![Más resultados de pruebas en un visor JSON en línea.](media/vm/test-results-json-viewer-1.png)

### <a name="on-windows-os"></a>En el SO Windows

Llame a la API en PowerShell:

1. Use el comando Invoke-WebRequest para llamar a la API.
2. El método es Post y el tipo de contenido es JSON, tal como se muestra en la siguiente pantalla de ejemplo y en el ejemplo de código.
3. Cree los parámetros del cuerpo en formato JSON.

El siguiente ejemplo de código muestra una llamada de PowerShell a la API:

```PowerShell
$accesstoken = “Get token for your Client AAD App”$headers = New-Object”System.Collections.Generic.Dictionary[[String],[String]]”$headers.Add(“Authorization”, “Bearer $accesstoken”)$Body = @{ “DNSName” = “XXXX.westus.cloudapp.azure.com”“UserName” = “XXX”“Password” = “XXX@123456”“OS” = “Windows”“PortNo” = “5986”“CompanyName” = “ABCD” “AppID” = “XXXX-XXXX-XXXX” “TenantId” = “XXXX-XXXX-XXXX” } | ConvertTo-Json$res = Invoke-WebRequest -Method”Post” -Uri$uri -Body$Body -ContentType”application/json” –Headers $headers;$Content = $res | ConvertFrom-Json
```

Estos ejemplos de pantalla muestra una llamada a la API en PowerShell:

**Con la clave SSH**:

 :::image type="content" source="media/vm/call-api-with-ssh-key.png" alt-text="Llamar a la API en PowerShell con una clave SSH.":::

**Con contraseña**:

 :::image type="content" source="media/vm/call-api-with-password.png" alt-text="Llamar a la API en PowerShell con una contraseña.":::

<br>Mediante el ejemplo anterior, puede recuperar el archivo JSON y analizarlo para obtener los detalles siguientes:

```PowerShell
$resVar=$res|ConvertFrom-Json

$actualresult =$resVar.Response |ConvertFrom-Json

Write-Host”OSName: $($actualresult.OSName)”Write-Host”OSVersion: $($actualresult.OSVersion)”Write-Host”Overall Test Result: $($actualresult.TestResult)”For ($i=0; $i -lt$actualresult.Tests.Length; $i++){ Write-Host”TestID: $($actualresult.Tests[$i].TestID)”Write-Host”TestCaseName: $($actualresult.Tests[$i].TestCaseName)”Write-Host”Description: $($actualresult.Tests[$i].Description)”Write-Host”Result: $($actualresult.Tests[$i].Result)”Write-Host”ActualValue: $($actualresult.Tests[$i].ActualValue)”}
```

<br>En esta pantalla, que muestra `$res.Content`, se pueden ver los detalles de los resultados de las pruebas en formato JSON:

 :::image type="content" source="media/vm/test-results-json-format.png" alt-text="Detalles de los resultados de la prueba en formato JSON.":::

<br>Este es un ejemplo de los resultados de las pruebas que se ven en un visor JSON en línea (por ejemplo, [Code Beautify](https://codebeautify.org/jsonviewer) o [JSON Viewer](https://jsonformatter.org/json-viewer)).

![Resultados de pruebas en un visor JSON en línea.](media/vm/test-results-json-viewer-2.png)

## <a name="how-to-use-curl-to-consume-the-self-test-api-on-linux-os"></a>Cómo usar cURL para consumir la API de prueba automática en el SO Linux

Llame a la API en cURL:

1. Use el comando curl para llamar a la API.
2. El método es Post y el tipo de contenido es JSON, tal como se muestra en el siguiente fragmento de código.

```JSON
CURL POST -H “Content-Type:application/json”

-H “Authorization: Bearer XXXXXX-Token-XXXXXXXX”

[https://isvapp.azure-api.net/selftest-vm](https://isvapp.azure-api.net/selftest-vm)

-d ‘{ “DNSName”:”XXXX.westus.cloudapp.azure.com”, “UserName”:”XXX”, “Password”:”XXXX@123456”, “OS”:”Linux”, “PortNo”:”22”, “CompanyName”:”ABCD”, “AppId”:”XXXX-XXXX-XXXX”, “TenantId “XXXX-XXXX-XXXX”}’
```

<br>Este es un ejemplo del uso de cURL para llamar a la API:

![Ejemplo de uso de cURL para llamar a la API.](media/vm/use-curl-call-api.png)

<br>Estos son los resultados JSON de la llamada cURL:

![Resultados JSON de la llamada cURL.](media/vm/test-results-json-viewer-3.png)

## <a name="next-step"></a>Paso siguiente

- Consultar los [problemas comunes y correcciones de los identificadores URI de SAS](common-sas-uri-issues.md).
