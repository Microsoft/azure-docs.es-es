---
title: 'Ejemplo de script de la CLI de Azure: emparejar dos redes virtuales | Microsoft Docs'
description: Use un ejemplo de script de la CLI de Azure para crear y conectar dos redes virtuales de la misma región mediante la red de Azure.
services: virtual-network
documentationcenter: virtual-network
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: infrastructure
ms.date: 07/07/2017
ms.author: kumud
ms.custom: devx-track-azurecli
ms.openlocfilehash: 507e17b2edb0db2d21265f3e945745d97395e7c3
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87503861"
---
# <a name="use-an-azure-cli-sample-script-to-connect-two-virtual-networks"></a>Uso de un script de ejemplo de la CLI de Azure para conectar dos redes virtuales

Este script crea y conecta dos redes virtuales de la misma región a través de la red de Azure. Después de ejecutar el script, se creará un emparejamiento entre dos redes virtuales.

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]


## <a name="sample-script"></a>Script de ejemplo

[!code-azurecli-interactive[main](../../../cli_scripts/virtual-network/peer-two-virtual-networks/peer-two-virtual-networks.sh "Peer two networks")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación 

Ejecute el siguiente comando para quitar el grupo de recursos, la máquina virtual y todos los recursos relacionados.

```azurecli
az group delete --name myResourceGroup --yes
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear un grupo de recursos, una máquina virtual y todos los recursos relacionados. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [az group create](https://docs.microsoft.com/cli/azure/group) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az network vnet create](https://docs.microsoft.com/cli/azure/network/vnet) | Crea una red virtual y una subred de Azure. |
| [az network vnet peering create](https://docs.microsoft.com/cli/azure/network/vnet/peering) | Crea un emparejamiento entre dos redes virtuales.  |
| [az group delete](https://docs.microsoft.com/cli/azure/vm/extension) | Elimina un grupo de recursos, incluidos todos los recursos anidados. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](https://docs.microsoft.com/cli/azure).

En la [documentación de la información general de redes de Azure](../cli-samples.md) puede encontrar ejemplos adicionales de script de la CLI de redes.
