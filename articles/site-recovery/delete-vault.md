---
title: Eliminación de un almacén de Azure Site Recovery
description: Aprenda a eliminar un almacén de Recovery Services configurado para Azure Site Recovery.
author: rajani-janaki-ram
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 11/05/2019
ms.author: rajanaki
ms.openlocfilehash: 8237888d46e1f1ad48e678234cf1edce3558876a
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87089590"
---
# <a name="delete-a-site-recovery-services-vault"></a>Eliminación de un almacén de Site Recovery Services

En este artículo se describe cómo eliminar un almacén de Recovery Services para Site Recovery. Para eliminar un almacén de Azure Backup, consulte el artículo sobre la [eliminación de un almacén de Azure Backup](../backup/backup-azure-delete-vault.md).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]


## <a name="before-you-start"></a>Antes de comenzar

Antes de poder eliminar un almacén, debe quitar los servidores registrados y los elementos del almacén. Lo que necesita quitar depende de los escenarios de replicación que haya implementado. 


## <a name="delete-a-vault-azure-vm-to-azure"></a>Eliminación de un almacén: máquina virtual de Azure en Azure

1. Siga [estas instrucciones](site-recovery-manage-registration-and-protection.md#disable-protection-for-a-azure-vm-azure-to-azure) para eliminar todas las máquinas virtuales protegidas.
2. Luego, elimine el almacén.

## <a name="delete-a-vault-vmware-vm-to-azure"></a>Eliminación de un almacén: máquina virtual de VMware en Azure

1. Siga [estas instrucciones](site-recovery-manage-registration-and-protection.md#disable-protection-for-a-vmware-vm-or-physical-server-vmware-to-azure) para eliminar todas las máquinas virtuales protegidas.
2. Siga [estos pasos](vmware-azure-set-up-replication.md#disassociate-or-delete-a-replication-policy) para eliminar todas las directivas de replicación.
3. Elimine las referencias a vCenter con [estos pasos](vmware-azure-manage-vcenter.md#delete-a-vcenter-server).
4. Siga [estas instrucciones](vmware-azure-manage-configuration-server.md#delete-or-unregister-a-configuration-server) para retirar un servidor de configuración.
5. Luego, elimine el almacén.


## <a name="delete-a-vault-hyper-v-vm-with-vmm-to-azure"></a>Eliminación de un almacén: máquina virtual de Hyper-V (con VMM) en Azure

1. Siga [estos pasos](site-recovery-manage-registration-and-protection.md#disable-protection-for-a-hyper-v-virtual-machine-replicating-to-azure-using-the-system-center-vmm-to-azure-scenario) para eliminar máquinas virtuales de Hyper-V administradas por System Center VMM.
2. Desasocie y elimine todas las directivas de replicación. Haga esto en el almacén > **Infraestructura de Site Recovery** > **Para System Center VMM** > **Directivas de replicación**.
3. Siga [estos pasos](site-recovery-manage-registration-and-protection.md#unregister-a-vmm-server) para anular el registro de un servidor VMM conectado.
4. Luego, elimine el almacén.

## <a name="delete-a-vault-hyper-v-vm-to-azure"></a>Eliminación de un almacén: máquina virtual de Hyper-V en Azure

1. Siga [estos pasos](site-recovery-manage-registration-and-protection.md#disable-protection-for-a-hyper-v-virtual-machine-hyper-v-to-azure) para eliminar todas las máquinas virtuales protegidas.
2. Desasocie y elimine todas las directivas de replicación. Haga esto en el almacén > **Infraestructura de Site Recovery** > **Para sitios de Hyper-V** > **Directivas de replicación**.
3. Siga [estas instrucciones](site-recovery-manage-registration-and-protection.md#unregister-a-hyper-v-host-in-a-hyper-v-site) para anular el registro de un host de Hyper-V.
4. Elimine el sitio de Hyper-V.
5. Luego, elimine el almacén.


## <a name="use-powershell-to-force-delete-the-vault"></a>Uso de PowerShell para forzar la eliminación del almacén 

> [!Important]
> Si va a probar el producto y no le importa perder datos, puede usar el método de eliminación forzada para eliminar el almacén y todas sus dependencias rápidamente.
> El comando de PowerShell eliminará todo el contenido del almacén y la acción **es irreversible**.

Para eliminar el almacén de Site Recovery aunque haya elementos protegidos, use estos comandos:

```azurepowershell
Connect-AzAccount

Select-AzSubscription -SubscriptionName "XXXXX"

$vault = Get-AzRecoveryServicesVault -Name "vaultname"

Remove-AzRecoveryServicesVault -Vault $vault
```

Obtenga más información sobre [Get-AzRecoveryServicesVault](/powershell/module/az.recoveryservices/get-azrecoveryservicesvault) y [Remove-AzRecoveryServicesVault](/powershell/module/az.recoveryservices/remove-azrecoveryservicesvault).
