---
title: Obtención de notificaciones de mantenimiento de máquinas virtuales de Azure con PowerShell
description: Use PowerShell para ver las notificaciones de mantenimiento de máquinas virtuales que se ejecutan en Azure e iniciar el mantenimiento de autoservicio.
author: shants123
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 11/19/2019
ms.author: shants
ms.openlocfilehash: 4858a2e448955877107f6928b0aa01726e5db082
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87028592"
---
# <a name="handling-planned-maintenance-using-powershell"></a>Control del mantenimiento planeado con PowerShell

**Este artículo es válido para máquinas virtuales que se ejecutan tanto en Linux como en Windows.**

Azure PowerShell se puede usar para ver cuándo está programado el [mantenimiento](maintenance-notifications.md) de las máquinas virtuales. La información de mantenimiento planeado está disponible con el cmdlet [Get-AzVM](/powershell/module/az.compute/get-azvm) cuando se usa el parámetro `-status`.
  
La información de mantenimiento se devuelve solo si hay mantenimiento planeado. Si no está programado ningún mantenimiento que afecte a la máquina virtual, el cmdlet no devuelve ninguna información de mantenimiento. 


```powershell
Get-AzVM -ResourceGroupName myResourceGroup -Name myVM -Status
```

Las siguientes propiedades se devuelven en MaintenanceRedeployStatus: 

| Value | Descripción   |
|-------|---------------|
| IsCustomerInitiatedMaintenanceAllowed | Indica si puede iniciar el mantenimiento en la máquina virtual en este momento |
| PreMaintenanceWindowStartTime         | El comienzo de la ventana de autoservicio de mantenimiento en la que puede iniciar el mantenimiento en la máquina virtual |
| PreMaintenanceWindowEndTime           | El final de la ventana de autoservicio de mantenimiento en la que puede iniciar el mantenimiento en la máquina virtual |
| MaintenanceWindowStartTime            | El comienzo del mantenimiento programado en que Azure inicia el mantenimiento de la máquina virtual |
| MaintenanceWindowEndTime              | El final de la ventana de mantenimiento programado en la que Azure inicia el mantenimiento de la máquina virtual |
| LastOperationResultCode               | El resultado del último intento de iniciar el mantenimiento en la máquina virtual |



También puede obtener el estado de mantenimiento de todas las VM en un grupo de recursos mediante el uso de [Get-AzVM](/powershell/module/az.compute/get-azvm) sin especificar una VM.
 
```powershell
Get-AzVM -ResourceGroupName myResourceGroup -Status
```

En el siguiente ejemplo de PowerShell se toma el identificador de la suscripción y se devuelve una lista de las máquinas virtuales cuyo mantenimiento está programado.

```powershell

function MaintenanceIterator
{
    Select-AzSubscription -SubscriptionId $args[0]

    $rgList= Get-AzResourceGroup 

    for ($rgIdx=0; $rgIdx -lt $rgList.Length ; $rgIdx++)
    {
        $rg = $rgList[$rgIdx]        
    $vmList = Get-AzVM -ResourceGroupName $rg.ResourceGroupName 
        for ($vmIdx=0; $vmIdx -lt $vmList.Length ; $vmIdx++)
        {
            $vm = $vmList[$vmIdx]
            $vmDetails = Get-AzVM -ResourceGroupName $rg.ResourceGroupName -Name $vm.Name -Status
              if ($vmDetails.MaintenanceRedeployStatus )
            {
                Write-Output "VM: $($vmDetails.Name)  IsCustomerInitiatedMaintenanceAllowed: $($vmDetails.MaintenanceRedeployStatus.IsCustomerInitiatedMaintenanceAllowed) $($vmDetails.MaintenanceRedeployStatus.LastOperationMessage)"               
            }
          }
    }
}

```

### <a name="start-maintenance-on-your-vm-using-powershell"></a>Inicio del mantenimiento en la máquina virtual mediante PowerShell

Con la información de la función de la sección anterior, lo siguiente inicia el mantenimiento en una máquina virtual si **IsCustomerInitiatedMaintenanceAllowed** está establecido en true.

```powershell
Restart-AzVM -PerformMaintenance -name $vm.Name -ResourceGroupName $rg.ResourceGroupName 
```

## <a name="classic-deployments"></a>Implementaciones clásicas

[!INCLUDE [classic-vm-deprecation](../../includes/classic-vm-deprecation.md)]

Si todavía tiene máquinas virtuales heredadas que se han implementado según el modelo de implementación clásico, puede usar PowerShell para consultar las máquinas virtuales e iniciar el mantenimiento.

Para obtener el estado de mantenimiento de una máquina virtual, escriba lo siguiente:

```
Get-AzureVM -ServiceName <Service name> -Name <VM name>
```

Para iniciar el mantenimiento en una máquina virtual clásica, escriba lo siguiente:

```
Restart-AzureVM -InitiateMaintenance -ServiceName <service name> -Name <VM name>
```

## <a name="next-steps"></a>Pasos siguientes

El mantenimiento planeado también se puede controlar con la [CLI de Azure](maintenance-notifications-cli.md) o con [Portal](maintenance-notifications-portal.md).
