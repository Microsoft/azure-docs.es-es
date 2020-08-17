---
title: Cómo mover los almacenes de Recovery Services de Azure Backup
description: Instrucciones sobre cómo mover el almacén de Recovery Services entre suscripciones y grupos de recursos de Azure.
ms.topic: conceptual
ms.date: 04/08/2019
ms.custom: references_regions
ms.openlocfilehash: 40ef55fa3b86856051b840c5d88ab8fadae3b7c3
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86514108"
---
# <a name="move-a-recovery-services-vault-across-azure-subscriptions-and-resource-groups"></a>Traslado del almacén de Recovery Services entre suscripciones y grupos de recursos de Azure

En este artículo se explica cómo mover un almacén de Recovery Services configurado para Azure Backup entre suscripciones de Azure, o a otro grupo de recursos en la misma suscripción. Puede usar Azure Portal o PowerShell para mover un almacén de Recovery Services.

## <a name="supported-regions"></a>Regiones admitidas

El traslado de recursos para el almacén de Recovery Services se admite en las regiones Este de Australia, Sudeste de Australia, Centro de Canadá, Este de Canadá, Sudeste de Asia, Este de Asia, Centro de EE. UU., Centro-norte de EE. UU., Este de EE. UU., Este de EE. UU. 2, Centro-sur de EE. UU., Centro-oeste de EE. UU., Centro-oeste de EE. UU. 2, Oeste de EE. UU., Oeste de EE. UU. 2, Centro de la India, Sur de la India, Este de Japón, Oeste de Japón, Centro de Corea del Sur, Sur de Corea del Sur, Norte de Europa, Oeste de Europa, Norte de Sudáfrica, Oeste de Sudáfrica, Sur de Reino Unido y Oeste de Reino Unido.

## <a name="unsupported-regions"></a>Regiones no admitidas

Centro de Francia, Sur de Francia, Nordeste de Alemania, Centro de Alemania, US Gov Iowa, Norte de China, Norte de China 2, Este de China, Este de China 2

## <a name="prerequisites-for-moving-recovery-services-vault"></a>Requisitos previos para el traslado al almacén de Recovery Services

- Durante el traslado de almacén entre grupos de recursos, los grupos de recursos de origen y destino están bloqueados para evitar las operaciones de escritura y eliminación. Para más información, consulte este [artículo](../azure-resource-manager/management/move-resource-group-and-subscription.md).
- Solo la suscripción del administrador tiene los permisos para mover un almacén.
- Para trasladar almacenes entre suscripciones, la suscripción de destino debe existir en el mismo inquilino que la suscripción de origen y su estado debe estar habilitado.
- Debe tener permiso para realizar operaciones de escritura en el grupo de recursos de destino.
- El traslado del almacén solo cambia el grupo de recursos. El almacén de Recovery Services existirá en la misma ubicación y no se puede cambiar.
- Solo puede trasladar un almacén de Recovery Services por región cada vez.
- Si una máquina virtual no se traslada con el almacén de Recovery Services entre suscripciones, o a un nuevo grupo de recursos, los puntos de recuperación de la máquina virtual actuales permanecerán intactos en el almacén hasta que expiren.
- Independientemente de que la máquina virtual se mueva con el almacén o no, siempre podrá restaurar la máquina virtual desde el historial de copia de seguridad conservado en el almacén.
- Azure Disk Encryption requiere que el almacén de claves y las máquinas virtuales residan en la misma región y suscripción de Azure.
- Para mover una máquina virtual con discos administrados, consulte este [artículo](https://azure.microsoft.com/blog/move-managed-disks-and-vms-now-available/).
- Las opciones para mover recursos implementados mediante el modelo clásico varían en función de si traslada los recursos dentro de una misma suscripción o a una nueva suscripción. Para más información, consulte este [artículo](../azure-resource-manager/management/move-resource-group-and-subscription.md).
- Las directivas de copia de seguridad definidas para el almacén se conservan después de que el almacén se mueva entre suscripciones o a un nuevo grupo de recursos.
- Solo puede trasladar un almacén que contenga cualquiera de los siguientes tipos de elementos de copia de seguridad. Los elementos de copia de seguridad de tipos que no se enumeran a continuación deberán detenerse y los datos deberán eliminarse de forma permanente antes de mover el almacén.
  - Azure Virtual Machines
  - Agente de Microsoft Azure Recovery Services (MARS)
  - Microsoft Azure Backup Server (MABS)
  - Data Protection Manager (DPM)
- Si traslada un almacén con datos de copia de seguridad de VM entre suscripciones, deberá trasladar sus VM a la misma suscripción y usar el mismo nombre de grupo de recursos de VM de destino (tal como estaba en la suscripción anterior) para continuar con las copias de seguridad.

> [!NOTE]
> No se admite la migración de almacenes de Recovery Services para Azure Backup entre regiones de Azure.<br><br>
> Si ha configurado alguna máquina virtual (Azure IaaS, Hyper-V, VMware) o máquinas físicas para la recuperación ante desastres mediante **Azure Site Recovery**, se bloqueará la operación de traslado. Si desea mover almacenes para Azure Site Recovery, consulte [este artículo](../site-recovery/move-vaults-across-regions.md) para más información sobre cómo mover los almacenes manualmente.

## <a name="use-azure-portal-to-move-recovery-services-vault-to-different-resource-group"></a>Uso de Azure Portal para trasladar un almacén de Recovery Services a otro grupo de recursos

Para mover un almacén de Recovery Services y sus recursos asociados a un grupo de recursos diferente

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. Abra la lista de **almacenes de Recovery Services** y seleccione el nombre del almacén que desea mover. Cuando se abre el panel del almacén, aparece como se muestra en la siguiente imagen.

   ![Almacén de Recovery Services abierto](./media/backup-azure-move-recovery-services/open-recover-service-vault.png)

   Si no ve la **Información esencial** del almacén, haga clic en el icono de la lista desplegable. Ahora debería ver la información esencial del almacén.

   ![Pestaña Información esencial](./media/backup-azure-move-recovery-services/essentials-information-tab.png)

3. En el menú de información general del almacén, haga clic en **Cambiar** junto a **Grupo de recursos** para abrir la hoja **Mover recursos**.

   ![Cambiar el grupo de recursos](./media/backup-azure-move-recovery-services/change-resource-group.png)

4. En la hoja **Mover recursos**, para el almacén seleccionado se recomienda mover los recursos relacionados opcionales activando la casilla de verificación, tal como se muestra en la siguiente imagen.

   ![Mover suscripción](./media/backup-azure-move-recovery-services/move-resource.png)

5. Para agregar el grupo de recursos de destino, en la lista desplegable **Grupo de recursos**, seleccione un grupo de recursos existente o haga clic en la opción **Crear un grupo nuevo**.

   ![Crear recurso](./media/backup-azure-move-recovery-services/create-a-new-resource.png)

6. Después de agregar el grupo de recursos, confirme la opción **Comprendo que las herramientas y los scripts asociados con recursos movidos no funcionarán hasta que los actualice para que usen nuevos identificadores de recursos** y, a continuación, haga clic en **Aceptar** para completar el traslado del almacén.

   ![Mensaje de confirmación](./media/backup-azure-move-recovery-services/confirmation-message.png)

## <a name="use-azure-portal-to-move-recovery-services-vault-to-a-different-subscription"></a>Uso de Azure Portal para trasladar un almacén de Recovery Services a otra suscripción

Puede mover un almacén de Recovery Services y sus recursos asociados a otra suscripción

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. Abra la lista de almacenes de Recovery Services y seleccione el almacén que desea mover. Cuando se abre el panel del almacén, aparece como se muestra en la siguiente imagen.

    ![Almacén de Recovery Services abierto](./media/backup-azure-move-recovery-services/open-recover-service-vault.png)

    Si no ve la **Información esencial** del almacén, haga clic en el icono de la lista desplegable. Ahora debería ver la información esencial del almacén.

    ![Pestaña Información esencial](./media/backup-azure-move-recovery-services/essentials-information-tab.png)

3. En el menú de información general del almacén, haga clic en **Cambiar** junto a **Suscripción** para abrir la hoja **Mover recursos**.

   ![Cambiar suscripción](./media/backup-azure-move-recovery-services/change-resource-subscription.png)

4. Seleccione los recursos que se van a mover; para ello, se recomienda usar la opción **Seleccionar todo** para seleccionar todos los recursos enumerados opcionales.

   ![Mover recurso](./media/backup-azure-move-recovery-services/move-resource-source-subscription.png)

5. Seleccione la suscripción de destino desde la lista desplegable **Suscripción** a donde desea que se mueva el almacén.
6. Para agregar el grupo de recursos de destino, en la lista desplegable **Grupo de recursos**, seleccione un grupo de recursos existente o haga clic en la opción **Crear un grupo nuevo**.

   ![Agregar suscripción](./media/backup-azure-move-recovery-services/add-subscription.png)

7. Haga clic en la opción **Comprendo que las herramientas y los scripts asociados a recursos movidos no funcionarán hasta que los actualice para que usen nuevos identificadores de recursos** para confirmar y luego haga clic en **Aceptar**.

> [!NOTE]
> La copia de seguridad entre suscripciones (el almacén de RS y las máquinas virtuales protegidas están en distintas suscripciones) no es un escenario admitido. Además, la opción de redundancia de almacenamiento desde el almacenamiento con redundancia local (LRS) al almacenamiento con redundancia global (GRS) y viceversa no se puede modificar durante la operación de movimiento del almacén.
>
>

## <a name="use-powershell-to-move-recovery-services-vault"></a>Uso de PowerShell para trasladar un almacén de Recovery Services

Para mover un almacén de Recovery Services a otro grupo de recursos, use el cmdlet `Move-AzureRMResource`. `Move-AzureRMResource` requiere el nombre del recurso y el tipo de recurso. Puede obtener ambos con el cmdlet `Get-AzureRmRecoveryServicesVault`.

```powershell
$destinationRG = "<destinationResourceGroupName>"
$vault = Get-AzureRmRecoveryServicesVault -Name <vaultname> -ResourceGroupName <vaultRGname>
Move-AzureRmResource -DestinationResourceGroupName $destinationRG -ResourceId $vault.ID
```

Para mover los recursos a otra suscripción, incluya el parámetro `-DestinationSubscriptionId`.

```powershell
Move-AzureRmResource -DestinationSubscriptionId "<destinationSubscriptionID>" -DestinationResourceGroupName $destinationRG -ResourceId $vault.ID
```

Después de ejecutar los cmdlets anteriores, se le pedirá que confirme que desea mover los recursos especificados. Escriba **Y** para continuar. Cuando se haya validado correctamente, el recurso se moverá.

## <a name="use-cli-to-move-recovery-services-vault"></a>Uso de la CLI para trasladar un almacén de Recovery Services

Para mover un almacén de Recovery Services a otro grupo de recursos, use el siguiente cmdlet:

```azurecli
az resource move --destination-group <destinationResourceGroupName> --ids <VaultResourceID>
```

Para mover a una nueva suscripción, proporcione el parámetro `--destination-subscription-id`.

## <a name="post-migration"></a>Después de la migración

1. Establezca/verifique los controles de acceso para los grupos de recursos.  
2. La característica de supervisión y creación de informes de Backup debe volver a configurarse para el almacén una vez finalizado el proceso de traslado. La configuración anterior se perderá durante la esta operación.

## <a name="next-steps"></a>Pasos siguientes

Puede mover muchos tipos diferentes de recursos entre suscripciones y grupos de recursos.

Para obtener más información, consulte [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](../azure-resource-manager/management/move-resource-group-and-subscription.md).
