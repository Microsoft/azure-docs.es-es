---
title: Traslado de un recurso de máquina virtual Windows en Azure
description: Mueva una máquina virtual Windows a otro grupo de recursos o suscripción de Azure en el modelo de implementación de Resource Manager.
author: cynthn
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 07/03/2019
ms.author: cynthn
ms.openlocfilehash: 1f919a4af85a15bbe80d7176c316100c3bad634a
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "86998925"
---
# <a name="move-a-windows-vm-to-another-azure-subscription-or-resource-group"></a>Movimiento de una máquina virtual Windows a otro grupo de recursos o suscripción de Azure
Este artículo le guiará en el procedimiento para mover una máquina virtual (VM) Windows entre suscripciones o grupos de recursos. Mover máquinas virtuales entre suscripciones puede ser útil si originalmente creó una en una suscripción personal y ahora quiere moverla a la suscripción de su compañía para seguir trabajando. Para poder mover la VM, no es necesario detenerla y debe mantenerse en ejecución durante el desplazamiento.

> [!IMPORTANT]
>Como parte de esta operación, se crean nuevos identificadores de recurso. Después de haber movido la VM, debe actualizar sus herramientas y scripts para usar los nuevos id. de recursos.
>
>

[!INCLUDE [virtual-machines-common-move-vm](../../../includes/virtual-machines-common-move-vm.md)]

## <a name="use-powershell-to-move-a-vm"></a>Uso de PowerShell para mover una máquina virtual

Para mover una máquina virtual a otro grupo de recursos, debe asegurarse de trasladar todos los recursos dependientes. Para obtener una lista con el identificador de recurso de cada uno de estos recursos, use el cmdlet [Get-AzResource](/powershell/module/az.resources/get-azresource).

```azurepowershell-interactive
 Get-AzResource -ResourceGroupName myResourceGroup | Format-table -wrap -Property ResourceId
```

Se puede usar la salida del comando anterior para crear una lista separada por comas de identificadores de recursos de [Move-AzResource](/powershell/module/az.resources/move-azresource) para mover cada recurso hasta el destino.

```azurepowershell-interactive
Move-AzResource -DestinationResourceGroupName "myDestinationResourceGroup" `
    -ResourceId <myResourceId,myResourceId,myResourceId>
```

Para mover los recursos a otra suscripción, incluya el parámetro **-DestinationSubscriptionId** .

```azurepowershell-interactive
Move-AzResource -DestinationSubscriptionId "<myDestinationSubscriptionID>" `
    -DestinationResourceGroupName "<myDestinationResourceGroup>" `
    -ResourceId <myResourceId,myResourceId,myResourceId>
```


Cuando se le pida que confirme que quiere mover los recursos especificados, escriba **Y** para confirmar.

## <a name="next-steps"></a>Pasos siguientes
Puede mover muchos tipos diferentes de recursos entre suscripciones y grupos de recursos. Para obtener más información, consulte [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](../../azure-resource-manager/management/move-resource-group-and-subscription.md).    
