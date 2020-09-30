---
title: Traslado de máquinas virtuales de Azure a una nueva suscripción o grupo de recursos
description: Use Azure Resource Manager para trasladar máquinas virtuales a un nuevo grupo de recursos o a una nueva suscripción.
ms.topic: conceptual
ms.date: 08/31/2020
ms.openlocfilehash: 3878113f6874c40953bec87518a89519bdc6cb1a
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89230966"
---
# <a name="move-guidance-for-virtual-machines"></a>Guía del traslado de máquinas virtuales

En este artículo se describen los escenarios que actualmente no se admiten y los pasos para trasladar máquinas virtuales con copia de seguridad.

## <a name="scenarios-not-supported"></a>Escenarios no admitidos

Todavía no se admiten los siguientes escenarios:

* No es posible trasladar Virtual Machine Scale Sets con equilibrador de carga o IP pública de SKU estándar.
* Las máquinas virtuales creadas a partir de recursos de Marketplace con planes asociados no se pueden mover entre suscripciones. Desaprovisione el recurso en la suscripción activa y vuelva a implementarlo en la nueva suscripción.
* Las máquinas virtuales de una red virtual existente no se pueden mover a una suscripción nueva si no se van a mover todos los recursos de la red virtual.
* Las máquinas virtuales de prioridad baja y los conjuntos de escalado de máquinas virtuales de prioridad baja no pueden moverse entre grupos de recursos o suscripciones.
* Las máquinas virtuales de un conjunto de disponibilidad no se pueden mover individualmente.

## <a name="azure-disk-encryption"></a>Azure Disk Encryption

No puede mover una VM que esté integrada con un almacén de claves para implementar [Azure Disk Encryption para máquinas virtuales Linux](../../../virtual-machines/linux/disk-encryption-overview.md) o [Azure Disk Encryption para máquinas virtuales Windows](../../../virtual-machines/windows/disk-encryption-overview.md). Para mover la VM, debe deshabilitar el cifrado.

```azurecli-interactive
az vm encryption disable --resource-group demoRG --name myVm1
```

```azurepowershell-interactive
Disable-AzVMDiskEncryption -ResourceGroupName demoRG -VMName myVm1
```

## <a name="virtual-machines-with-azure-backup"></a>Máquinas virtuales con Azure Backup

Para trasladar las máquinas virtuales configuradas con Azure Backup, debe eliminar los puntos de restauración del almacén.

Si la [eliminación temporal](../../../backup/backup-azure-security-feature-cloud.md) está habilitada para la máquina virtual, no podrá moverla mientras se conserven esos puntos de restauración. [Deshabilite la eliminación temporal](../../../backup/backup-azure-security-feature-cloud.md#enabling-and-disabling-soft-delete) o espere 14 días después de eliminar los puntos de restauración.

### <a name="portal"></a>Portal

1. Detenga temporalmente la copia de seguridad y conserve los datos de esta.
2. Para mover máquinas virtuales configuradas con Azure Backup, siga los pasos a continuación:

   1. Busque la ubicación de la máquina virtual.
   2. Busque un grupo de recursos con el siguiente patrón de nomenclatura: `AzureBackupRG_<VM location>_1`. Por ejemplo, el nombre tiene el formato *AzureBackupRG_westus2_1*.
   3. En Azure Portal, active **Mostrar tipos ocultos**.
   4. Busque el recurso con el tipo **Microsoft.Compute/restorePointCollections** que tenga el patrón de nomenclatura `AzureBackup_<name of your VM that you're trying to move>_###########`.
   5. Elimine este recurso. Esta operación elimina solo los puntos de recuperación instantáneos, no los datos de copia de seguridad que se encuentran en el almacén.
   6. Una vez completada la operación de eliminación, puede mover la máquina virtual.

3. Traslade la máquina virtual al grupo de recursos de destino.
4. Reanude la copia de seguridad.

### <a name="powershell"></a>PowerShell

1. Busque la ubicación de la máquina virtual.

1. Busque un grupo de recursos con el patrón de nomenclatura `AzureBackupRG_<VM location>_1`. Por ejemplo, el nombre podría ser `AzureBackupRG_westus2_1`.

1. Use el siguiente comando para obtener la colección de puntos de restauración.

   ```azurepowershell
   $RestorePointCollection = Get-AzResource -ResourceGroupName AzureBackupRG_<VM location>_1 -ResourceType Microsoft.Compute/restorePointCollections
   ```

1. Elimine este recurso. Esta operación elimina solo los puntos de recuperación instantáneos, no los datos de copia de seguridad que se encuentran en el almacén.

   ```azurepowershell
   Remove-AzResource -ResourceId $RestorePointCollection.ResourceId -Force
   ```

### <a name="azure-cli"></a>Azure CLI

1. Busque la ubicación de la máquina virtual.

1. Busque un grupo de recursos con el patrón de nomenclatura `AzureBackupRG_<VM location>_1`. Por ejemplo, el nombre podría ser `AzureBackupRG_westus2_1`.

1. Utilice el comando siguiente para obtener la colección de puntos de restauración.

   ```azurecli
   az resource list -g AzureBackupRG_<VM location>_1 --resource-type Microsoft.Compute/restorePointCollections
   ```

1. Busque el identificador de recurso con el patrón de nomenclatura `AzureBackup_<VM name>_###########`.

1. Elimine este recurso. Esta operación elimina solo los puntos de recuperación instantáneos, no los datos de copia de seguridad que se encuentran en el almacén.

   ```azurecli
   az resource delete --ids /subscriptions/<sub-id>/resourceGroups/<resource-group>/providers/Microsoft.Compute/restorePointCollections/<name>
   ```

## <a name="next-steps"></a>Pasos siguientes

* Para ver los comandos para trasladar recursos, vea [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](../move-resource-group-and-subscription.md).

* Para más información sobre cómo mover los almacenes de Recovery Services para realizar copias de seguridad, consulte [Limitaciones de Recovery Services](../../../backup/backup-azure-move-recovery-services-vault.md?toc=/azure/azure-resource-manager/toc.json).
