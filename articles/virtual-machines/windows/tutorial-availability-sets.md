---
title: 'Tutorial: Alta disponibilidad para máquinas virtuales Windows en Azure'
description: En este tutorial, aprenderá a usar Azure PowerShell para implementar máquinas virtuales de alta disponibilidad en conjuntos de disponibilidad.
services: virtual-machines-windows
author: cynthn
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.topic: tutorial
ms.date: 11/30/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 7fe1c01542df2fcc38982fe2a30f9e94c712eacb
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87065259"
---
# <a name="tutorial-create-and-deploy-highly-available-virtual-machines-with-azure-powershell"></a>Tutorial: Creación e implementación de máquinas virtuales de alta disponibilidad con Azure PowerShell

En este tutorial, obtendrá información sobre cómo aumentar la disponibilidad y confiabilidad de Virtual Machines mediante conjuntos de disponibilidad. Los conjuntos de disponibilidad garantizan que las máquinas virtuales implementadas en Azure se distribuyan entre varios nodos de hardware aislados en un clúster. 

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear un conjunto de disponibilidad
> * Crear una máquina virtual en un conjunto de disponibilidad
> * Comprobar los tamaños de máquina virtual disponibles
> * Comprobar Azure Advisor


## <a name="availability-set-overview"></a>Información general sobre conjuntos de disponibilidad

Un conjunto de disponibilidad es una funcionalidad de agrupación lógica para aislar los recursos de máquina virtual entre sí cuando se implementan. Azure garantiza que las máquinas virtuales colocadas en un conjunto de disponibilidad se ejecuten en varios servidores físicos, grupos de proceso, unidades de almacenamiento y conmutadores de red. Si se produce un error de hardware o software, solo un subconjunto de las máquinas virtuales se ve afectado y la solución general permanece operativa. Los conjuntos de disponibilidad son esenciales para la creación de soluciones en la nube confiables.

Veamos una solución basada en máquina virtual típica en la podría haber cuatro servidores web front-end y dos máquinas virtuales de back-end. Con Azure, desea definir dos conjuntos de disponibilidad antes de implementar las máquinas virtuales: uno para el nivel de web y otro para el nivel de back-end. Al crear una nueva máquina virtual, especifique el conjunto de disponibilidad como un parámetro. Azure garantiza que las máquinas virtuales están aisladas en varios recursos de hardware físico. Si el hardware físico que uno de los servidores ejecuta tiene un problema, sabe que las restantes instancias de los servidores siguen ejecutándose correctamente porque están en otro hardware.

Use los conjuntos de disponibilidad cuando quiera implementar soluciones basadas en máquinas virtuales confiables en Azure.

## <a name="launch-azure-cloud-shell"></a>Inicio de Azure Cloud Shell

Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. 

Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior derecha de un bloque de código. También puede ir a [https://shell.azure.com/powershell](https://shell.azure.com/powershell) para iniciar Cloud Shell en una pestaña independiente del explorador. Seleccione **Copiar** para copiar los bloques de código, péguelos en Cloud Shell y, luego, presione Entrar para ejecutarlos.

## <a name="create-an-availability-set"></a>Crear un conjunto de disponibilidad

El hardware de una ubicación está dividido en varios dominios de actualización y de error. Un **dominio de actualización** es un grupo de máquinas virtuales y hardware físico subyacente que se pueden reiniciar al mismo tiempo. Las máquinas virtuales en el mismo **dominio de error** comparten un almacenamiento común, así como una fuente de alimentación y un conmutador de red comunes.  

Puede crear un conjunto de disponibilidad con [New-AzAvailabilitySet](/powershell/module/az.compute/new-azavailabilityset). En este ejemplo, el número de dominios de error y de actualización es *2*, y el conjunto de disponibilidad se denomina *myAvailabilitySet*.

Cree un grupo de recursos.

```azurepowershell-interactive
New-AzResourceGroup `
   -Name myResourceGroupAvailability `
   -Location EastUS
```

Cree un conjunto de disponibilidad administrado mediante [New-AzAvailabilitySet](/powershell/module/az.compute/new-azavailabilityset) con el parámetro `-sku aligned`.

```azurepowershell-interactive
New-AzAvailabilitySet `
   -Location "EastUS" `
   -Name "myAvailabilitySet" `
   -ResourceGroupName "myResourceGroupAvailability" `
   -Sku aligned `
   -PlatformFaultDomainCount 2 `
   -PlatformUpdateDomainCount 2
```

## <a name="create-vms-inside-an-availability-set"></a>Creación de VM dentro de un conjunto de disponibilidad
Las máquinas virtuales deben crearse en el conjunto de disponibilidad para asegurarse de que se distribuyan correctamente en el hardware. No se puede agregar una máquina virtual existente a un conjunto de disponibilidad después de crearla. 


Al crear una máquina virtual mediante [New-AzVM](/powershell/module/az.compute/new-azvm), con el parámetro `-AvailabilitySetName` se especifica el nombre del conjunto de disponibilidad.

En primer lugar, establezca un nombre de usuario de administrador y una contraseña para la máquina virtual con [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential?view=powershell-5.1):

```azurepowershell-interactive
$cred = Get-Credential
```

Ahora, cree dos máquinas virtuales con [New-AzVM](/powershell/module/az.compute/new-azvm) en el conjunto de disponibilidad.

```azurepowershell-interactive
for ($i=1; $i -le 2; $i++)
{
    New-AzVm `
        -ResourceGroupName "myResourceGroupAvailability" `
        -Name "myVM$i" `
        -Location "East US" `
        -VirtualNetworkName "myVnet" `
        -SubnetName "mySubnet" `
        -SecurityGroupName "myNetworkSecurityGroup" `
        -PublicIpAddressName "myPublicIpAddress$i" `
        -AvailabilitySetName "myAvailabilitySet" `
        -Credential $cred
}
```

Se tarda unos minutos en crear y configurar ambas VM. Al terminar, tendrá dos máquinas virtuales distribuidas en el hardware subyacente. 

Si observa el conjunto de disponibilidad en el portal en **Grupos de recursos** > **myResourceGroupAvailability** > **myAvailabilitySet**, debería ver cómo se distribuyen las máquinas virtuales en ambos dominios, el de error y el de actualización.

![Conjunto de disponibilidad en el portal](./media/tutorial-availability-sets/fd-ud.png)

## <a name="check-for-available-vm-sizes"></a>Comprobación de los tamaños de VM disponibles 

Cuando cree una máquina virtual en un conjunto de disponibilidad, debe saber qué tamaños de máquina virtual están disponibles en el hardware. Use el comando [Get-AzVMSize](/powershell/module/az.compute/get-azvmsize) para obtener todos los tamaños disponibles para las máquinas virtuales que puede implementar en el conjunto de disponibilidad.

```azurepowershell-interactive
Get-AzVMSize `
   -ResourceGroupName "myResourceGroupAvailability" `
   -AvailabilitySetName "myAvailabilitySet"
```

## <a name="check-azure-advisor"></a>Comprobar Azure Advisor 

También puede usar Azure Advisor para más información acerca de las distintas maneras de mejorar la disponibilidad de las máquinas virtuales. Azure Advisor analiza la configuración de los recursos y la telemetría de uso, y recomienda soluciones que pueden ayudar a mejorar la rentabilidad, el rendimiento, la alta disponibilidad y la seguridad de los recursos de Azure.

Inicie sesión en [Azure Portal](https://portal.azure.com), seleccione **Todos los servicios** y escriba **Advisor**. El panel de Advisor muestra recomendaciones personalizadas de la suscripción seleccionada. Para más información, consulte [Introducción con Azure Advisor](../../advisor/advisor-get-started.md).


## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Crear un conjunto de disponibilidad
> * Crear una máquina virtual en un conjunto de disponibilidad
> * Comprobar los tamaños de máquina virtual disponibles
> * Comprobar Azure Advisor

Avance al siguiente tutorial para informarse sobre los conjuntos de escalado de máquinas virtuales.

> [!div class="nextstepaction"]
> [Creación de un conjunto de escalado de máquinas virtuales](tutorial-create-vmss.md)
