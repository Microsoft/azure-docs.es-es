---
title: Cambio de tamaño de una máquina virtual Linux con la CLI de Azure
description: Escalado o reducción en vertical de una máquina virtual de Linux cambiando el tamaño de la VM.
author: DavidCBerry13
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 02/10/2017
ms.author: daberry
ms.openlocfilehash: a93b1dcfe803fdead0d741553c45894f1675372b
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87372270"
---
# <a name="resize-a-linux-virtual-machine-using-azure-cli"></a>Cambiar el tamaño de una máquina virtual Linux que usa la CLI de Azure 

Después de aprovisionar una máquina virtual (VM), puede escalarla o reducirla verticalmente cambiando su [tamaño][vm-sizes]. En algunos casos, hay que desasignarla antes. Necesita desasignar la VM si el tamaño deseado no se encuentra disponible en el clúster de hardware que hospeda la VM. En este artículo se detalla cómo cambiar el tamaño de una máquina virtual Linux con la CLI de Azure. 

## <a name="resize-a-vm"></a>Cambiar el tamaño de una máquina virtual
Para cambiar el tamaño de una máquina virtual, necesita tener instalada la última versión de la [CLI de Azure](/cli/azure/install-az-cli2) e iniciar sesión en una cuenta de Azure con [az login](/cli/azure/reference-index).

1. Consulte la lista de tamaños de VM disponibles en el clúster de hardware que hospeda la VM con [az vm list-vm-resize-options](/cli/azure/vm). En el ejemplo siguiente se enumeran los tamaños para la VM denominada `myVM` en la región del grupo de recursos `myResourceGroup`:
   
    ```azurecli
    az vm list-vm-resize-options --resource-group myResourceGroup --name myVM --output table
    ```

2. Si aparece el tamaño de máquina virtual deseado, cambie el tamaño de la máquina virtual con [az vm resize](/cli/azure/vm). En el ejemplo siguiente se cambia el tamaño de la VM denominada `myVM` por el tamaño `Standard_DS3_v2`:
   
    ```azurecli
    az vm resize --resource-group myResourceGroup --name myVM --size Standard_DS3_v2
    ```
   
    La VM se reinicia durante este proceso. Tras reiniciar, se reasignan los discos del SO y de datos. Se pierde todo lo que haya en el disco temporal.

3. Si no aparece el tamaño de VM deseado, debe desasignar primero la VM con [az vm deallocate](/cli/azure/vm). Este proceso permite cambiar el tamaño de la VM por otro tamaño disponible que la región admite y, a continuación, se inicia. Los siguientes pasos permiten desasignar, cambiar de tamaño y luego iniciar la VM denominada `myVM` en el grupo de recursos llamado `myResourceGroup`:
   
    ```azurecli
    az vm deallocate --resource-group myResourceGroup --name myVM
    az vm resize --resource-group myResourceGroup --name myVM --size Standard_DS3_v2
    az vm start --resource-group myResourceGroup --name myVM
    ```
   
   > [!WARNING]
   > Al desasignar la máquina virtual, también se liberan todas las direcciones IP dinámicas asignadas a ella. Esto no afecta a los discos del SO y de datos.

## <a name="next-steps"></a>Pasos siguientes
Para obtener una mayor escalabilidad, ejecute varias instancias de VM y escálelas horizontalmente. Para más información, consulte [Escalado automático de máquinas de Linux en un conjunto de escalado de máquinas virtuales][scale-set]. 

<!-- links -->
[boot-diagnostics]: https://azure.microsoft.com/blog/boot-diagnostics-for-virtual-machines-v2/
[scale-set]: ../../virtual-machine-scale-sets/tutorial-autoscale-cli.md 
[vm-sizes]:sizes.md
