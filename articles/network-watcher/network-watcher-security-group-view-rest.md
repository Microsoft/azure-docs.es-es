---
title: 'Análisis de la seguridad de red: vista de grupo de seguridad: API REST de Azure'
titleSuffix: Azure Network Watcher
description: En este artículo se describe cómo utilizar PowerShell para analizar la seguridad de máquinas virtuales con la vista de grupos de seguridad.
services: network-watcher
documentationcenter: na
author: damendo
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: damendo
ms.openlocfilehash: 466879daae65b8c7fa896b0ead417c8d5eb51e28
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87084354"
---
# <a name="analyze-your-virtual-machine-security-with-security-group-view-using-rest-api"></a>Análisis de la seguridad de una máquina virtual con la vista de grupos de seguridad mediante la API de REST

> [!div class="op_single_selector"]
> - [PowerShell](network-watcher-security-group-view-powershell.md)
> - [CLI de Azure](network-watcher-security-group-view-cli.md)
> - [REST API](network-watcher-security-group-view-rest.md)

> [!NOTE]
> La API de vista de grupo de seguridad ya no se mantiene y dejará de utilizarse pronto. Use la [característica Reglas de seguridad vigentes](https://docs.microsoft.com/azure/network-watcher/network-watcher-security-group-view-overview) que proporciona la misma funcionalidad. 

La vista de grupos de seguridad devuelve las reglas de seguridad de red configuradas y vigentes que se aplican a una máquina virtual. Esta funcionalidad resulta útil para auditar y diagnosticar los grupos de seguridad de red y las reglas que están configuradas en una máquina virtual para asegurarse de que el tráfico se está permitiendo o denegando correctamente. En este artículo se muestra cómo recuperar las reglas de seguridad configuradas y vigentes para una máquina virtual mediante la API de REST.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="before-you-begin"></a>Antes de empezar

En este escenario, llame a la API de REST de Network Watcher para obtener la vista de grupos de seguridad de una máquina virtual. ARMclient se usa para llamar a la API de REST con PowerShell. ARMClient se encuentra en Chocolatey en [ARMClient en Chocolatey](https://chocolatey.org/packages/ARMClient)

En este escenario, se da por hecho que ya ha seguido los pasos descritos en [Creación de una instancia de Network Watcher](network-watcher-create.md) para crear un monitor de red. En este escenario también se da por hecho que existe un grupo de recursos con una máquina virtual válida.

## <a name="scenario"></a>Escenario

El escenario descrito en este artículo recupera las reglas de seguridad aplicadas y vigentes para una máquina virtual dada.

## <a name="log-in-with-armclient"></a>Inicio de sesión con ARMClient

```powershell
armclient login
```

## <a name="retrieve-a-virtual-machine"></a>Recuperación de una máquina virtual

Ejecute el siguiente script para devolver una máquina virtual. El siguiente código necesita variables:

- **subscriptionId**: el identificador de la suscripción también puede recuperarse con el cmdlet **Get-AzSubscription**.
- **resourceGroupName**: el nombre de un grupo de recursos que contiene máquinas virtuales.

```powershell
$subscriptionId = '<subscription id>'
$resourceGroupName = '<resource group name>'

armclient get https://management.azure.com/subscriptions/${subscriptionId}/ResourceGroups/${resourceGroupName}/providers/Microsoft.Compute/virtualMachines?api-version=2015-05-01-preview
```

La información necesaria se corresponde con el **Id** que aparece debajo del tipo `Microsoft.Compute/virtualMachines` en la respuesta, como se observa en el ejemplo siguiente:

```json
...,
        "networkProfile": {
          "networkInterfaces": [
            {
              "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft
.Network/networkInterfaces/{nicName}"
            }
          ]
        },
        "provisioningState": "Succeeded"
      },
      "resources": [
        {
          "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Com
pute/virtualMachines/{vmName}/extensions/CustomScriptExtension"
        }
      ],
      "type": "Microsoft.Compute/virtualMachines",
      "location": "westcentralus",
      "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute
/virtualMachines/{vmName}",
      "name": "{vmName}"
    }
  ]
}
```

## <a name="get-security-group-view-for-virtual-machine"></a>Obtención de la vista de grupos de seguridad para una máquina virtual

El ejemplo siguiente solicita la vista de grupos de seguridad de una máquina virtual específica. Los resultados de este ejemplo pueden usarse para compararlos con las reglas y la seguridad definidas en el origen para buscar desfases de configuración.

```powershell
$subscriptionId = "<subscription id>"
$resourceGroupName = "<resource group name>"
$networkWatcherName = "<network watcher name>"
$targetUri = "<uri of target resource>" # Example: /subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/Microsoft.compute/virtualMachine/$vmName

$requestBody = @"
{
    'targetResourceId': '${targetUri}'

}
"@
armclient post "https://management.azure.com/subscriptions/${subscriptionId}/ResourceGroups/${resourceGroupName}/providers/Microsoft.Network/networkWatchers/${networkWatcherName}/securityGroupView?api-version=2016-12-01" $requestBody -verbose
```

## <a name="view-the-response"></a>Visualización de la respuesta

El ejemplo siguiente es la respuesta devuelta por el comando anterior. Los resultados muestran todas las reglas de seguridad vigentes y aplicadas en la máquina virtual, clasificadas en los grupos **NetworkInterfaceSecurityRules**, **DefaultSecurityRules** y **EffectiveSecurityRules**.

```json

{
  "networkInterfaces": [
    {
      "securityRuleAssociations": {
        "networkInterfaceAssociation": {
          "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/networkInterfaces/{nicName}",
          "securityRules": [
            {
              "name": "default-allow-rdp",
              "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/networkSecurityGroups/{nsgName}/securityRules/default-allow-rdp",
              "etag": "W/\"d4c411d4-0d62-49dc-8092-3d4b57825740\"",
              "properties": {
                "provisioningState": "Succeeded",
                "protocol": "TCP",
                "sourcePortRange": "*",
                "destinationPortRange": "3389",
                "sourceAddressPrefix": "*",
                "destinationAddressPrefix": "*",
                "access": "Allow",
                "priority": 1000,
                "direction": "Inbound"
              }
            }
          ]
        },
        "defaultSecurityRules": [
          {
            "name": "AllowVnetInBound",
            "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/networkSecurityGroups/{nsgName}/defaultSecurityRules/",
            "properties": {
              "provisioningState": "Succeeded",
              "description": "Allow inbound traffic from all VMs in VNET",
              "protocol": "*",
              "sourcePortRange": "*",
              "destinationPortRange": "*",
              "sourceAddressPrefix": "VirtualNetwork",
              "destinationAddressPrefix": "VirtualNetwork",
              "access": "Allow",
              "priority": 65000,
              "direction": "Inbound"
            }
          },
          ...
        ],
        "effectiveSecurityRules": [
          {
            "name": "DefaultOutboundDenyAll",
            "protocol": "All",
            "sourcePortRange": "0-65535",
            "destinationPortRange": "0-65535",
            "sourceAddressPrefix": "*",
            "destinationAddressPrefix": "*",
            "access": "Deny",
            "priority": 65500,
            "direction": "Outbound"
          },
          ...
        ]
      }
    }
  ]
}
```

## <a name="next-steps"></a>Pasos siguientes

Visite [Automate NSG auditing with Azure Network Watcher Security group view](network-watcher-security-group-view-powershell.md) (Automatización de la auditoría de grupos de seguridad de red (NSG) con la vista de grupos de seguridad de Azure Network Watcher) para aprender a automatizar la validación de grupos de seguridad de red.


