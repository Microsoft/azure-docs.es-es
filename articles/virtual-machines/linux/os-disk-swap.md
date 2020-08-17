---
title: Intercambio entre discos del sistema operativo mediante la CLI '
description: Intercambiar el disco del sistema operativo que usa una máquina virtual de Azure mediante la CLI.
author: cynthn
ms.service: virtual-machines
ms.subservice: disks
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 04/24/2018
ms.author: cynthn
ms.openlocfilehash: adcbc5b0ba0056c79576e8d4c8ded2150b206513
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87371879"
---
# <a name="change-the-os-disk-used-by-an-azure-vm-using-the-cli"></a>Intercambiar el disco del sistema operativo que usa una máquina virtual de Azure mediante la CLI


Si ya tiene una máquina virtual, pero quiere intercambiar el disco por una copia de seguridad u otro disco del sistema operativo, puede usar la CLI de Azure para intercambiar los discos del sistema operativo. No es necesario eliminar ni volver a crear la máquina virtual. Incluso puede utilizar un disco administrado de otro grupo de recursos, siempre y cuando no esté en uso.

Es necesario que la máquina virtual esté detenida o sin asignar; a continuación, el identificador de recurso del disco administrado se puede reemplazar con el identificador de recurso de otro disco administrado. 

Asegúrese de que el tipo de almacenamiento y el tamaño de la máquina virtual son compatibles con el disco que quiere adjuntar. Por ejemplo, si el disco que quiere usar está en Premium Storage, la máquina virtual debe ser compatible con Premium Storage (por ejemplo, debe tener un tamaño de la serie DS).

Para poder realizar los pasos de este artículo, debe tener la versión 2.0.25 o posterior de la CLI de Azure. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli). 


Use [az disk list](/cli/azure/disk) para obtener una lista de los discos del grupo de recursos.

```azurecli-interactive
az disk list \
   -g myResourceGroupDisk \
   --query '[*].{diskId:id}' \
   --output table
```


Use [az vm stop](/cli/azure/vm) para detener o desasignar la máquina virtual antes de intercambiar los discos.

```azurecli-interactive
az vm stop \
   -n myVM \
   -g myResourceGroup
```


Use [az vm update](/cli/azure/vm#az-vm-update) con el id. de recurso completo del nuevo disco para el parámetro `--osdisk`. 

```azurecli-interactive 
az vm update \
   -g myResourceGroup \
   -n myVM \
   --os-disk /subscriptions/<subscription ID>/resourceGroups/swap/providers/Microsoft.Compute/disks/myDisk 
   ```
   
Reinicie la máquina virtual con [az vm start](/cli/azure/vm).

```azurecli-interactive
az vm start \
   -n myVM \
   -g myResourceGroup
```

   
**Pasos siguientes**

Para crear una copia de un disco, consulte [Instantánea de un disco](snapshot-copy-managed-disk.md).
