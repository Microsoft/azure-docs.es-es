---
title: Uso de Desired State Configuration con conjuntos de escalado de máquinas virtuales
description: Uso de conjuntos de escalado de máquinas virtuales con la extensión Desired State Configuration de Azure para configurar máquinas virtuales.
author: ju-shim
ms.author: jushiman
ms.topic: how-to
ms.service: virtual-machine-scale-sets
ms.subservice: extensions
ms.date: 6/25/2020
ms.reviewer: mimckitt
ms.custom: mimckitt
ms.openlocfilehash: 20e5bff87d5cd0d6e0a35a558462bb5598bfe3f4
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87080495"
---
# <a name="using-virtual-machine-scale-sets-with-the-azure-dsc-extension"></a>Uso de conjuntos de escalado de máquinas virtuales con la extensión de DSC de Azure
Los [conjuntos de escalado de máquinas virtuales](./overview.md) pueden usarse con el controlador de extensiones de [Configuración de estado deseado (DSC) de Azure](../virtual-machines/extensions/dsc-overview.md?toc=/azure/virtual-machines/windows/toc.json). Los conjuntos de escalado de máquinas virtuales proporcionan una manera de implementar y administrar un gran número de máquinas virtuales, y se pueden escalar y reducir horizontalmente en respuesta a la carga. DSC se utiliza para configurar las máquinas virtuales a medida que se conecten, ya que ejecutan el software de producción.

## <a name="differences-between-deploying-to-virtual-machines-and-virtual-machine-scale-sets"></a>Diferencias entre implementar máquinas virtuales y conjuntos de escalado de máquinas virtuales
La estructura subyacente de la plantilla para un conjunto de escalado de máquinas virtuales es ligeramente distinta a la de una sola máquina virtual. En concreto, una sola máquina virtual implementa extensiones en el nodo virtualMachines. Hay una entrada del tipo extensions, donde se agrega DSC a la plantilla.

```
"resources": [
          {
              "name": "Microsoft.Powershell.DSC",
              "type": "extensions",
              "location": "[resourceGroup().location]",
              "apiVersion": "2015-06-15",
              "dependsOn": [
                  "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
              ],
              "tags": {
                  "displayName": "dscExtension"
              },
              "properties": {
                  "publisher": "Microsoft.Powershell",
                  "type": "DSC",
                  "typeHandlerVersion": "2.20",
                  "autoUpgradeMinorVersion": false,
                  "forceUpdateTag": "[parameters('dscExtensionUpdateTagVersion')]",
                  "settings": {
                      "configuration": {
                          "url": "[concat(parameters('_artifactsLocation'), '/', variables('dscExtensionArchiveFolder'), '/', variables('dscExtensionArchiveFileName'))]",
                          "script": "DscExtension.ps1",
                          "function": "Main"
                      },
                      "configurationArguments": {
                          "nodeName": "[variables('vmName')]"
                      }
                  },
                  "protectedSettings": {
                      "configurationUrlSasToken": "[parameters('_artifactsLocationSasToken')]"
                  }
              }
          }
      ]
```

Un nodo del conjunto de escalado de máquinas virtuales tiene una sección "properties" con el atributo "extensionProfile" de "VirtualMachineProfile". DSC se agrega en "extensions".

```
"extensionProfile": {
            "extensions": [
                {
                    "name": "Microsoft.Powershell.DSC",
                    "properties": {
                        "publisher": "Microsoft.Powershell",
                        "type": "DSC",
                        "typeHandlerVersion": "2.20",
                        "autoUpgradeMinorVersion": false,
                        "forceUpdateTag": "[parameters('DscExtensionUpdateTagVersion')]",
                        "settings": {
                            "configuration": {
                                "url": "[concat(parameters('_artifactsLocation'), '/', variables('DscExtensionArchiveFolder'), '/', variables('DscExtensionArchiveFileName'))]",
                                "script": "DscExtension.ps1",
                                "function": "Main"
                            },
                            "configurationArguments": {
                                "nodeName": "localhost"
                            }
                        },
                        "protectedSettings": {
                            "configurationUrlSasToken": "[parameters('_artifactsLocationSasToken')]"
                        }
                    }
                }
            ]
```

## <a name="behavior-for-a-virtual-machine-scale-set"></a>Comportamiento de un conjunto de escalado de máquinas virtuales
El comportamiento del conjunto de escalado de máquinas virtuales es idéntico al de una sola máquina virtual. Cuando se crea una nueva máquina virtual, se aprovisiona automáticamente con la extensión de DSC. Si la extensión requiere una versión más reciente de WMF, se reinicia la máquina virtual antes de conectarse. Cuando se conecta, descarga el archivo .zip de configuración de DSC y lo aprovisiona en la máquina virtual. Pueden encontrar más información en el artículo de [introducción a la extensión de DSC de Azure](../virtual-machines/extensions/dsc-overview.md?toc=/azure/virtual-machines/windows/toc.json).

## <a name="next-steps"></a>Pasos siguientes
Examine la [plantilla de Azure Resource Manager para la extensión de DSC](../virtual-machines/extensions/dsc-template.md?toc=/azure/virtual-machines/windows/toc.json).

Obtenga información sobre cómo la [extensión de DSC administra de forma segura las credenciales](../virtual-machines/extensions/dsc-credentials.md?toc=/azure/virtual-machines/windows/toc.json). 

Para más información sobre el controlador de extensiones DSC de Azure, consulte [Introducción al controlador de extensiones de configuración de estado deseado de Azure](../virtual-machines/extensions/dsc-overview.md?toc=/azure/virtual-machines/windows/toc.json). 

Para más información sobre DSC de PowerShell, [visite el centro de documentación de PowerShell](/powershell/scripting/dsc/overview/overview). 
