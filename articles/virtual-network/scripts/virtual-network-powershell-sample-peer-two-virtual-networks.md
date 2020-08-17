---
title: 'Emparejamiento de dos redes virtuales: ejemplo de script de Azure PowerShell'
description: Cree y conecte dos redes virtuales en la misma región. Use el script de Azure para dos redes virtuales del mismo nivel para conectar las redes a través de la red de Azure.
services: virtual-network
documentationcenter: virtual-network
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: powershell
ms.topic: sample
ms.tgt_pltfrm: ''
ms.workload: infrastructure
ms.date: 03/20/2018
ms.author: kumud
ms.openlocfilehash: 7cbcae51228e5bf722f3ea802a322483aed6c225
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87286128"
---
# <a name="peer-two-virtual-networks-script-sample"></a>Ejemplo de script de emparejamiento de dos redes virtuales

Este script de ejemplo crea y conecta dos redes virtuales de la misma región mediante la red de Azure. Después de ejecutar el script, se creará un emparejamiento entre dos redes virtuales.

Puede ejecutar el script desde Azure [Cloud Shell](https://shell.azure.com/powershell) o desde una instalación de PowerShell local. Si usa PowerShell de forma local, este script requiere la versión del módulo Az de PowerShell 5.4.1 o posterior. Ejecute `Get-Module -ListAvailable Az` para ver cuál es la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-Az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Script de ejemplo

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!code-azurepowershell-interactive[main](../../../powershell_scripts/virtual-network/peer-two-virtual-networks/peer-two-virtual-networks.ps1 "Peer two networks")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación

Ejecute el siguiente comando para quitar el grupo de recursos, la VM y todos los recursos relacionados:

```powershell
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear un grupo de recursos, una máquina virtual y todos los recursos relacionados. Cada comando de la tabla siguiente contiene un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork)| Crea una red virtual y una subred de Azure. |
| [Add-AzVirtualNetworkPeering](/powershell/module/az.network/add-azvirtualnetworkpeering) | Crea un emparejamiento entre dos redes virtuales.  |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Elimina un grupo de recursos, incluidos todos los recursos anidados. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).

Puede encontrar más ejemplos de script de PowerShell de red virtual en [ejemplos de PowerShell de red virtual](../powershell-samples.md).