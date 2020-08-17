---
title: 'Creación de una máquina virtual a partir de una instantánea: Ejemplo de la CLI'
description: 'Ejemplo de script de la CLI de Azure: creación de una máquina virtual a partir de una instantánea'
services: virtual-machines-linux
documentationcenter: virtual-machines
author: ramankumarlive
manager: kavithag
editor: ramankum
tags: azure-service-management
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 05/10/2017
ms.author: ramankum
ms.custom: mvc
ms.openlocfilehash: a6110ba2787cb99e20c099eb466e2dbd0c3df28e
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87085306"
---
# <a name="create-a-virtual-machine-from-a-snapshot-with-cli"></a>Creación de una máquina virtual a partir de una instantánea con la CLI

Este script crea una máquina virtual a partir de una instantánea de un disco del SO.

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Script de ejemplo

[!code-azurecli-interactive[main](../../../cli_scripts/virtual-machine/create-vm-from-snapshot/create-vm-from-snapshot.sh "Create VM from snapshot")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación 

Ejecute el siguiente comando para quitar el grupo de recursos, la máquina virtual y todos los recursos relacionados.

```azurecli-interactive 
az group delete --name myResourceGroup
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear un disco administrado, una máquina virtual y todos los recursos relacionados. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [az snapshot show](/cli/azure/snapshot) | Obtiene instantáneas mediante el nombre de la instantánea y el del grupo de recursos. La propiedad de identificador del objeto devuelto se utiliza para crear un disco administrado.  |
| [az disk create](/cli/azure/disk) | Crea discos administrados a partir de una instantánea usando el identificador de las instantáneas, el nombre del disco, el tipo de almacenamiento y el tamaño.  |
| [az vm create](/cli/azure/vm) | Crea una máquina virtual con un disco del SO administrado. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Encontrará más ejemplos de scripts de la CLI de máquina virtual en la [documentación sobre máquinas virtuales Linux de Azure](../linux/cli-samples.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
