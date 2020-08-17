---
title: 'Ejemplos de la CLI de Azure: Conexión y uso de discos de datos'
description: Este script crea un conjunto de escalado de máquinas virtuales de Azure y conecta y prepara los discos de datos con la CLI de Azure.
author: mimckitt
ms.author: mimckitt
ms.topic: sample
ms.service: virtual-machine-scale-sets
ms.subservice: disks
ms.date: 03/27/2018
ms.reviewer: jushiman
ms.custom: mimckitt, devx-track-azurecli
ms.openlocfilehash: 3fed38814d3ed00c97288af41a2aa57d6a696d4e
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87499745"
---
# <a name="attach-and-use-data-disks-with-a-virtual-machine-scale-set-with-the-azure-cli"></a>Conexión y uso de discos de datos con un conjunto de escalado de máquinas virtuales con la CLI de Azure
Este script crea un conjunto de escalado de máquinas virtuales y conecta y prepara los discos de datos.

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Script de ejemplo
[!code-azurecli-interactive[main](../../../cli_scripts/virtual-machine-scale-sets/use-data-disks/use-data-disks.sh "Create a virtual machine scale set with data disks")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación
Ejecute el siguiente comando para quitar el grupo de recursos, el conjunto de escalado y todos los recursos relacionados.

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="script-explanation"></a>Explicación del script
Este script usa los siguientes comandos para crear un grupo de recursos, un conjunto de escalado de máquinas virtuales y todos los recursos relacionados. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [az group create](/cli/azure/ad/group) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az vmss create](/cli/azure/vmss) | Crea el conjunto de escalado de máquinas virtuales y lo conecta a la red virtual, la subred y el grupo de seguridad de red. También se crea un equilibrador de carga para distribuir el tráfico a varias instancias de máquina virtual. Este comando también especifica la imagen de máquina virtual que se usará y las credenciales administrativas.  |
| [az vmss disk attach](/cli/azure/vmss/disk) | Crea y asocia un disco de datos al conjunto de escalado de máquinas virtuales. |
| [az vmss extension set](/cli/azure/vmss/extension) | Instala la extensión de script personalizado de Azure para ejecutar un script que prepare los discos de datos en cada instancia de máquina virtual. |
| [az group delete](/cli/azure/ad/group) | Elimina un grupo de recursos, incluidos todos los recursos anidados. |

## <a name="next-steps"></a>Pasos siguientes
Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure/overview).
