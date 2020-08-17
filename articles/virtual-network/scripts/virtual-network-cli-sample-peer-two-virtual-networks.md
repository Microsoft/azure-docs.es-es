---
title: 'Emparejamiento de dos redes virtuales: ejemplo de script de la CLI de Azure'
description: Use un script de ejemplo de la CLI de Azure para crear y conectar dos redes virtuales de la misma región mediante la red de Azure.
services: virtual-network
documentationcenter: virtual-network
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: ''
ms.workload: infrastructure
ms.date: 03/20/2018
ms.author: kumud
ms.custom: devx-track-azurecli
ms.openlocfilehash: 6cd6d61ba1404ca6b6de63a0151ec414b2df54e7
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87492469"
---
# <a name="peer-two-virtual-networks-with-an-azure-cli-script-sample"></a>Emparejamiento de dos redes virtuales con un script de ejemplo de la CLI de Azure

Este script de ejemplo crea y conecta dos redes virtuales de la misma región mediante la red de Azure. Después de ejecutar el script, tendrá un emparejamiento entre dos redes virtuales.

Puede ejecutar el script desde Azure [Cloud Shell](https://shell.azure.com/bash) o desde una instalación de CLI de Azure local. Si usa la CLI localmente, este script requiere que ejecute la versión 2.0.28 o posterior. Ejecute `az --version` para ver cuál es la versión instalada. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). Si ejecuta la CLI localmente, también debe ejecutar `az login` para crear una conexión con Azure.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]


## <a name="sample-script"></a>Script de ejemplo

[!code-azurecli-interactive[main](../../../cli_scripts/virtual-network/peer-two-virtual-networks/peer-two-virtual-networks.sh "Peer two networks")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación 

Ejecute el siguiente comando para quitar el grupo de recursos, la VM y todos los recursos relacionados:

```azurecli
az group delete --name myResourceGroup --yes
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear un grupo de recursos, una máquina virtual y todos los recursos relacionados. Cada comando de la tabla siguiente crea un vínculo a documentación específica del comando:

| Get-Help | Notas |
|---|---|
| [az group create](/cli/azure/group) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az network vnet create](/cli/azure/network/vnet) | Crea una red virtual y una subred de Azure. |
| [az network vnet peering create](/cli/azure/network/vnet/peering) | Crea un emparejamiento entre dos redes virtuales.  |
| [az group delete](/cli/azure/vm/extension) | Elimina un grupo de recursos, incluidos todos los recursos anidados. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](https://docs.microsoft.com/cli/azure).

Puede encontrar más ejemplos de script de CLI de red virtual en [ejemplos de CLI de red virtual](../cli-samples.md).
