---
title: 'Ejemplo de script de la CLI de Azure: Montaje del disco de sistema operativo'
description: 'Ejemplo de script de la CLI de Azure: Montaje del disco de sistema operativo'
services: virtual-machines-linux
documentationcenter: virtual-machines
author: cynthn
manager: gwallace
tags: azure-service-management
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 02/27/2017
ms.author: cynthn
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 735b16ad054edfea7cf0ad2e1e5b9c76b664e6c3
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87479657"
---
# <a name="troubleshoot-a-vms-operating-system-disk"></a>Solución de problemas de un disco de sistema operativo de máquina virtual

Este script monta el disco del sistema operativo de una máquina virtual problemática o que ha generado un error, como un disco de datos en una segunda máquina virtual. Esto puede ser útil para solucionar problemas de disco o de recuperación de datos.

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Script de ejemplo

[!code-azurecli-interactive[main](../../../cli_scripts/virtual-machine/mount-os-disk/mount-os-disk.sh "Quick Create VM")]

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear un grupo de recursos, una máquina virtual y todos los recursos relacionados. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [az vm show](/cli/azure/vm) | Devuelve una lista de máquinas virtuales. En este caso, la opción de consulta se utiliza para devolver el disco de sistema operativo de la máquina virtual. Este valor se agrega, posteriormente, a un nombre de variable "uri". |
| [az vm delete](/cli/azure/vm) | Elimina una máquina virtual. |
| [az vm create](/cli/azure/vm) | Crea una máquina virtual.  |
| [az vm disk attach](/cli/azure/vm/disk) | Conecta un disco a una máquina virtual. |
| [az vm list-ip-addresses](/cli/azure/vm) | Devuelve las direcciones IP de una máquina virtual. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Encontrará más ejemplos de scripts de la CLI de máquina virtual en la [documentación sobre máquinas virtuales Linux de Azure](../linux/cli-samples.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
