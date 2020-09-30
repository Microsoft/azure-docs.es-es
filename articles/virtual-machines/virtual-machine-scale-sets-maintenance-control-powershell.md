---
title: Control de mantenimiento para las actualizaciones de imágenes del sistema operativo en conjuntos de escalado de máquinas virtuales de Azure mediante PowerShell
description: Aprenda a controlar cuándo se implementan las actualizaciones automáticas de las imágenes del sistema operativo en los conjuntos de escalado de máquinas virtuales de Azure mediante el control de mantenimiento y PowerShell.
author: ju-shim
ms.service: virtual-machine-scale-sets
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 09/11/2020
ms.author: jushiman
ms.openlocfilehash: 62b2fd0554c218b733501805779745049d3b8c92
ms.sourcegitcommit: 03662d76a816e98cfc85462cbe9705f6890ed638
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90532994"
---
# <a name="preview-maintenance-control-for-os-image-upgrades-on-azure-virtual-machine-scale-sets-using-powershell"></a>Vista previa: Control de mantenimiento para las actualizaciones de imágenes del sistema operativo en conjuntos de escalado de máquinas virtuales de Azure mediante PowerShell

El control de mantenimiento le permite decidir cuándo aplicar las actualizaciones automáticas de imágenes del sistema operativo invitado a los conjuntos de escalado de máquinas virtuales. En este tema se tratan las opciones de Azure PowerShell para el control de mantenimiento. Para más información sobre el uso del control de mantenimiento, consulte [Control de mantenimiento para conjuntos de escalado de máquinas virtuales de Azure](virtual-machine-scale-sets-maintenance-control.md).

> [!IMPORTANT]
> El control de mantenimiento para las actualizaciones de imágenes del sistema operativo en conjuntos de escalado de máquinas virtuales de Azure se encuentra actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin contrato de nivel de servicio y no es aconsejable usarla para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).


## <a name="enable-the-powershell-module"></a>Habilitación del módulo de PowerShell

Asegúrese de que `PowerShellGet` está actualizado.    

```azurepowershell-interactive  
Install-Module -Name PowerShellGet -Repository PSGallery -Force 
``` 

Instalación del módulo de PowerShell `Az.Maintenance`.     

```azurepowershell-interactive  
Install-Module -Name Az.Maintenance
``` 

Si lo va a instalar localmente, asegúrese de abrir el símbolo del sistema de PowerShell como administrador.

También es posible que se le pida que confirme que desea realizar la instalación desde un *repositorio que no es de confianza*. Escriba `Y` o seleccione **Sí a todo** para instalar el módulo.

## <a name="connect-to-an-azure-account"></a>Conexión a la cuenta de Azure

Conéctese a la cuenta de Azure que quiera mediante [Connect-AzAccount](https://docs.microsoft.com/powershell/module/az.accounts/connect-azaccount) y [Set-AzAccount](https://docs.microsoft.com/powershell/module/az.accounts/set-azcontext).

```azurepowershell-interactive
Connect-AzAccount
Set-AzContext 00a000aa-0a00-0a0a-00aa-a00a000aaa00

$RGName="myMaintenanceRG"
$MaintenanceConfig="myMaintenanceConfig"
$location="eastus2"
$vmss="myMaintenanceVMSS"
```

## <a name="create-a-maintenance-configuration"></a>Creación de una configuración de mantenimiento

Cree un grupo de recursos como contenedor para la configuración. En este ejemplo, se crea un grupo de recursos denominado *myMaintenanceRG* en  *eastus2*. Puede omitir esta parte si ya tiene un grupo de recursos que le gustaría usar. Simplemente reemplace el nombre del grupo de recursos por el suyo en el resto de los ejemplos.

```azurepowershell-interactive
New-AzResourceGroup `
   -Location $location `
   -Name $RGName
```

Use [New-AzMaintenanceConfiguration](/powershell/module/az.maintenance/new-azmaintenanceconfiguration) para crear una configuración de mantenimiento. En este ejemplo se crea una configuración de mantenimiento denominada *myConfig* en el ámbito de la imagen del sistema operativo. 

```azurepowershell-interactive
$config = New-AzMaintenanceConfiguration `
   -ResourceGroup $RGName `
   -Name $MaintenanceConfig `
   -MaintenanceScope OSImage `
   -Location $location `
   -StartDateTime "2020-10-01 00:00" `
   -TimeZone "Pacific Standard Time" `
   -Duration "05:00" `
   -RecurEvery "Day"
```

> [!IMPORTANT]
> La **duración** del mantenimiento debe ser de *5 horas* o más. La **periodicidad** del mantenimiento debe establecerse en *Día*.

El uso de `-MaintenanceScope OSImage` garantiza que se usa la configuración de mantenimiento para controlar las actualizaciones del sistema operativo invitado.

Si intenta crear una configuración con el mismo nombre, pero en una ubicación diferente, obtendrá un error. Los nombres de configuración deben ser únicos dentro de su grupo de recursos.

Puede consultar las configuraciones de mantenimiento disponibles mediante [Get-AzMaintenanceConfiguration](/powershell/module/az.maintenance/get-azmaintenanceconfiguration).

```azurepowershell-interactive
Get-AzMaintenanceConfiguration | Format-Table -Property Name,Id
```

## <a name="associate-your-virtual-machine-scale-set-to-the-maintenance-configuration"></a>Asociación de un conjunto de escalado de máquinas virtuales a una configuración de mantenimiento

Un conjunto de escalado de máquinas virtuales se puede asociar a cualquier configuración de mantenimiento independientemente de la región y la suscripción de esta. Al optar por la configuración de mantenimiento, se programarán automáticamente nuevas actualizaciones de la imagen del sistema operativo para el conjunto de escalado en la siguiente ventana de mantenimiento disponible.

Use [New-AzConfigurationAssignment](/powershell/module/az.maintenance/new-azconfigurationassignment) para asociar un conjunto de escalado de máquinas virtuales a la configuración de mantenimiento.

```azurepowershell-interactive
New-AzConfigurationAssignment `
   -ResourceGroupName $RGName `
   -Location $location `
   -ResourceName $vmss `
   -ResourceType VirtualMachineScaleSets `
   -ProviderName Microsoft.Compute `
   -ConfigurationAssignmentName $config.Name`
   -MaintenanceConfigurationId $config.Id
``` 

## <a name="enable-automatic-os-upgrade"></a>Habilitación de la actualización automática del sistema operativo

Puede habilitar las actualizaciones automáticas del sistema operativo para cada conjunto de escalado de máquinas virtuales que vaya a usar el control de mantenimiento. Consulte el documento [Actualización automática de imágenes del sistema operativo en un conjunto de escalado de máquinas virtuales de Azure](../virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-upgrade.md) para habilitar las actualizaciones automáticas del sistema operativo en el conjunto de escalado de máquinas virtuales. 


## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre el mantenimiento y las actualizaciones de las máquinas virtuales que se ejecutan en Azure.

> [!div class="nextstepaction"]
> [Mantenimiento y actualizaciones](maintenance-and-updates.md)
