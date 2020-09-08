---
title: 'Tutorial: Creación y administración de máquinas virtuales Windows con Azure PowerShell'
description: En este tutorial, aprenderá a usar Azure PowerShell para crear y administrar máquinas virtuales Windows en Azure
author: cynthn
ms.service: virtual-machines-windows
ms.topic: tutorial
ms.workload: infrastructure
ms.date: 06/06/2019
ms.author: cynthn
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: 65116f979131590a66ff2f08896e7a89ae075303
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89074158"
---
# <a name="tutorial-create-and-manage-windows-vms-with-azure-powershell"></a>Tutorial: Creación y administración de máquinas virtuales Windows con Azure PowerShell

Las máquinas virtuales de Azure proporcionan un entorno informático completamente configurable y flexible. En este tutorial se tratan tareas básicas de la implementación de máquinas virtuales de Azure, como la selección de su tamaño, la selección de una imagen de máquina virtual y la implementación de una máquina virtual. Aprenderá a:

> [!div class="checklist"]
> * Crear y conectar elementos a una máquina virtual
> * Seleccionar y usar imágenes de máquinas virtuales
> * Ver y usar tamaños de una máquina virtual específicos
> * Cambiar el tamaño de una máquina virtual
> * Ver y entender el estado de las máquinas virtuales.

## <a name="launch-azure-cloud-shell"></a>Inicio de Azure Cloud Shell

Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. 

Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior derecha de un bloque de código. También puede ir a [https://shell.azure.com/powershell](https://shell.azure.com/powershell) para iniciar Cloud Shell en una pestaña independiente del explorador. Seleccione **Copiar** para copiar los bloques de código, péguelos en Cloud Shell y, luego, presione Entrar para ejecutarlos.

## <a name="create-resource-group"></a>Creación de un grupo de recursos

Cree un grupo de recursos con el comando [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup).

Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. Se debe crear un grupo de recursos antes de una máquina virtual. En el siguiente ejemplo, se crea un grupo de recursos denominado *myResourceGroupVM* en la región *EastUS*:

```azurepowershell-interactive
New-AzResourceGroup `
   -ResourceGroupName "myResourceGroupVM" `
   -Location "EastUS"
```

Se especifica el grupo de recursos al crear o modificar una máquina virtual, como se ve a lo largo de este tutorial.

## <a name="create-a-vm"></a>Crear una VM

Al crear una máquina virtual, hay varias opciones disponibles, como la imagen del sistema operativo, la configuración de red y las credenciales administrativas. En este ejemplo se crea una máquina virtual denominada *myVM* que ejecuta la versión predeterminada de Windows Server 2016 Datacenter.

Establezca el nombre de usuario y la contraseña que se necesitan para la cuenta de administrador en la máquina virtual con [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential?view=powershell-6):

```azurepowershell-interactive
$cred = Get-Credential
```

Cree la máquina virtual con [New-AzVM](/powershell/module/az.compute/new-azvm).

```azurepowershell-interactive
New-AzVm `
    -ResourceGroupName "myResourceGroupVM" `
    -Name "myVM" `
    -Location "EastUS" `
    -VirtualNetworkName "myVnet" `
    -SubnetName "mySubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -PublicIpAddressName "myPublicIpAddress" `
    -Credential $cred
```

## <a name="connect-to-vm"></a>Conexión con una máquina virtual

Una vez finalizada la implementación, cree una conexión del escritorio remoto con la máquina virtual.

Ejecute los siguientes comandos para devolver la dirección IP pública de la máquina virtual. Tome nota de esta dirección IP para poder conectarse a ella con un explorador para probar la conectividad web en un paso posterior.

```azurepowershell-interactive
Get-AzPublicIpAddress `
   -ResourceGroupName "myResourceGroupVM"  | Select IpAddress
```

Ejecute el siguiente comando en la máquina local para crear una sesión de escritorio remoto con la máquina virtual. Reemplace la dirección IP por el valor de *publicIPAddress* de la máquina virtual. Cuando se le solicite, escriba las credenciales usadas al crear la máquina virtual.

```powershell
mstsc /v:<publicIpAddress>
```

En la ventana **Seguridad de Windows**, seleccione **Más opciones** y, después, **Usar otra cuenta**. Escriba el nombre de usuario y la contraseña que creó para la máquina virtual y haga clic en **Aceptar**.

## <a name="understand-marketplace-images"></a>Descripción de las imágenes de Marketplace

Azure Marketplace incluye muchas imágenes que pueden usarse para crear una máquina virtual. En los pasos anteriores se creó una máquina virtual mediante la imagen de Windows Server 2016 Datacenter. En este paso, el módulo de PowerShell se usa para buscar en Marketplace otras imágenes de Windows, que también se pueden usar como base para nuevas máquinas virtuales. Este proceso consiste en buscar el publicador, la oferta, la SKU y, opcionalmente, un número de versión para [identificar](cli-ps-findimage.md#terminology) la imagen.

Use el comando [Get-AzVMImagePublisher](/powershell/module/az.compute/get-azvmimagepublisher) para devolver una lista de publicadores de imágenes:

```azurepowershell-interactive
Get-AzVMImagePublisher -Location "EastUS"
```

Use el comando [Get-AzVMImageOffer](/powershell/module/az.compute/get-azvmimageoffer) para devolver una lista de ofertas de imágenes. Con este comando, la lista devuelta se filtra por el publicador especificado denominado `MicrosoftWindowsServer`:

```azurepowershell-interactive
Get-AzVMImageOffer `
   -Location "EastUS" `
   -PublisherName "MicrosoftWindowsServer"
```

Los resultados deberán tener un aspecto similar a este ejemplo: 

```powershell
Offer             PublisherName          Location
-----             -------------          --------
Windows-HUB       MicrosoftWindowsServer EastUS
WindowsServer     MicrosoftWindowsServer EastUS
WindowsServer-HUB MicrosoftWindowsServer EastUS
```

El comando [Get-AzVMImageSku](/powershell/module/az.compute/get-azvmimagesku) filtrará entonces por el publicador y el nombre de la oferta para devolver una lista de nombres de imágenes.

```azurepowershell-interactive
Get-AzVMImageSku `
   -Location "EastUS" `
   -PublisherName "MicrosoftWindowsServer" `
   -Offer "WindowsServer"
```

Los resultados deberán tener un aspecto similar a este ejemplo: 

```powershell
Skus                                      Offer         PublisherName          Location
----                                      -----         -------------          --------
2008-R2-SP1                               WindowsServer MicrosoftWindowsServer EastUS  
2008-R2-SP1-smalldisk                     WindowsServer MicrosoftWindowsServer EastUS  
2012-Datacenter                           WindowsServer MicrosoftWindowsServer EastUS  
2012-Datacenter-smalldisk                 WindowsServer MicrosoftWindowsServer EastUS  
2012-R2-Datacenter                        WindowsServer MicrosoftWindowsServer EastUS  
2012-R2-Datacenter-smalldisk              WindowsServer MicrosoftWindowsServer EastUS  
2016-Datacenter                           WindowsServer MicrosoftWindowsServer EastUS  
2016-Datacenter-Server-Core               WindowsServer MicrosoftWindowsServer EastUS  
2016-Datacenter-Server-Core-smalldisk     WindowsServer MicrosoftWindowsServer EastUS
2016-Datacenter-smalldisk                 WindowsServer MicrosoftWindowsServer EastUS
2016-Datacenter-with-Containers           WindowsServer MicrosoftWindowsServer EastUS
2016-Datacenter-with-Containers-smalldisk WindowsServer MicrosoftWindowsServer EastUS
2016-Datacenter-with-RDSH                 WindowsServer MicrosoftWindowsServer EastUS
2016-Nano-Server                          WindowsServer MicrosoftWindowsServer EastUS
```

Esta información puede usarse para implementar una máquina virtual con una imagen específica. En este ejemplo se implementa una máquina virtual mediante la versión más reciente de Windows Server 2016 con una imagen de Containers.

```azurepowershell-interactive
New-AzVm `
    -ResourceGroupName "myResourceGroupVM" `
    -Name "myVM2" `
    -Location "EastUS" `
    -VirtualNetworkName "myVnet" `
    -SubnetName "mySubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -PublicIpAddressName "myPublicIpAddress2" `
    -ImageName "MicrosoftWindowsServer:WindowsServer:2016-Datacenter-with-Containers:latest" `
    -Credential $cred `
    -AsJob
```

El parámetro `-AsJob` crea la máquina virtual como tarea en segundo plano, por lo que PowerShell solicita la vuelta. Puede ver detalles de trabajos en segundo plano con el cmdlet `Get-Job`.

## <a name="understand-vm-sizes"></a>Comprensión de los tamaños de máquina virtual

El tamaño de la máquina virtual determina la cantidad de recursos de proceso, como memoria, CPU y GPU, que están disponibles para la máquina virtual. Las máquinas virtuales deben crearse con un tamaño de máquina virtual adecuado para la carga de trabajo. Si esta aumenta, también se puede cambiar el tamaño de las máquinas virtuales existentes.

### <a name="vm-sizes"></a>Tamaños de máquina virtual

En la tabla siguiente se clasifican los tamaños en casos de uso.  

| Tipo                     | Tamaños comunes           |    Descripción       |
|--------------------------|-------------------|------------------------------------------------------------------------------------------------------------------------------------|
| [Uso general](../sizes-general.md)         |B, Dsv3, Dv3, DSv2, Dv2, Av2, DC| Uso equilibrado de CPU y memoria. Ideal para desarrollo/pruebas, así como soluciones de datos y aplicaciones de tamaño pequeño a mediano.  |
| [Proceso optimizado](../sizes-compute.md)   | Fsv2          | Uso elevado de la CPU respecto a la memoria. Adecuado para aplicaciones, dispositivos de red y procesos por lotes con tráfico mediano.        |
| [Memoria optimizada](../sizes-memory.md)    | Esv3, Ev3, M, DSv2, Dv2  | Uso elevado de memoria respecto al núcleo. Excelente para bases de datos relacionales, memorias caché de capacidad de mediana a grande y análisis en memoria.                 |
| [Almacenamiento optimizado](../sizes-storage.md)      | Lsv2, Ls              | Alto rendimiento de disco y E/S. Perfecto para bases de datos SQL y NoSQL y macrodatos.                                                         |
| [GPU](../sizes-gpu.md)          | NV, NVv2, NC, NCv2, NCv3, ND            | Máquinas virtuales especializadas para actividades intensas de representación de gráficos y edición de vídeo.       |
| [Alto rendimiento](../sizes-hpc.md) | H        | Nuestras máquinas virtuales con CPU más eficaces e interfaces de red de alto rendimiento (RDMA) opcionales. |

### <a name="find-available-vm-sizes"></a>Búsqueda de los tamaños de máquina virtual disponibles

Para ver una lista de tamaños de máquina virtual disponibles en una región determinada, use el comando [Get-AzVMSize](/powershell/module/az.compute/get-azvmsize).

```azurepowershell-interactive
Get-AzVMSize -Location "EastUS"
```

## <a name="resize-a-vm"></a>Cambiar el tamaño de una máquina virtual

Una vez implementada una máquina virtual, se puede cambiar su tamaño para aumentar o disminuir la asignación de recursos.

Antes de cambiar el tamaño de una máquina virtual, compruebe si el tamaño deseado está disponible en el clúster actual de máquinas virtuales. El comando [Get-AzVMSize](/powershell/module/az.compute/get-azvmsize) devuelve una lista de tamaños.

```azurepowershell-interactive
Get-AzVMSize -ResourceGroupName "myResourceGroupVM" -VMName "myVM"
```

Si el tamaño está disponible, puede cambiarlo con la máquina virtual encendida, aunque se reiniciará durante la operación.

```azurepowershell-interactive
$vm = Get-AzVM `
   -ResourceGroupName "myResourceGroupVM"  `
   -VMName "myVM"
$vm.HardwareProfile.VmSize = "Standard_DS3_v2"
Update-AzVM `
   -VM $vm `
   -ResourceGroupName "myResourceGroupVM"
```

Si el tamaño que desea no está disponible en el clúster actual, se debe desasignar la máquina virtual para que se pueda llevar a cabo la operación de cambio de tamaño. La desasignación de una máquina virtual se traduce en la eliminación de los datos del disco temporal y la dirección IP pública cambiará, a menos que se use una dirección IP estática.

```azurepowershell-interactive
Stop-AzVM `
   -ResourceGroupName "myResourceGroupVM" `
   -Name "myVM" -Force
$vm = Get-AzVM `
   -ResourceGroupName "myResourceGroupVM"  `
   -VMName "myVM"
$vm.HardwareProfile.VmSize = "Standard_E2s_v3"
Update-AzVM -VM $vm `
   -ResourceGroupName "myResourceGroupVM"
Start-AzVM `
   -ResourceGroupName "myResourceGroupVM"  `
   -Name $vm.name
```

## <a name="vm-power-states"></a>Estados de energía de una máquina virtual

Una máquina virtual de Azure puede tener uno de muchos estados de energía. 


| Estado de energía | Descripción
|----|----|
| Iniciando | La máquina virtual se está iniciando. |
| En ejecución | La máquina virtual se está ejecutando. |
| Deteniéndose | La máquina virtual se está deteniendo. |
| Detenido | La VM está detenida. Las máquinas virtuales en el estado detenido siguen acumulando cargos por procesos.  |
| Desasignando | La máquina virtual se está desasignando. |
| Desasignado | Indica que la máquina virtual se quitó del hipervisor, pero sigue estando disponible en el plano de control. Las máquinas virtuales en el estado `Deallocated` no incurren en cargos por los procesos. |
| - | Se desconoce el estado de la máquina virtual. |


Para recuperar el estado de una máquina virtual concreta, use el comando [Get-AzVM](/powershell/module/az.compute/get-azvm). Asegúrese de especificar un nombre válido para la máquina virtual y el grupo de recursos.

```azurepowershell-interactive
Get-AzVM `
    -ResourceGroupName "myResourceGroupVM" `
    -Name "myVM" `
    -Status | Select @{n="Status"; e={$_.Statuses[1].Code}}
```

El resultado tendrá un aspecto similar al siguiente ejemplo:

```powershell
Status
------
PowerState/running
```

Para recuperar el estado de energía de todas las máquinas virtuales de la suscripción, use [Virtual Machines - List All API](https://docs.microsoft.com/rest/api/compute/virtualmachines/listall) con el parámetro **statusOnly** establecido en *true*.

## <a name="management-tasks"></a>Tareas de administración

Durante el ciclo de vida de una máquina virtual, puede que desee ejecutar tareas de administración como iniciarla, detenerla o eliminarla. Además, puede crear scripts para automatizar tareas repetitivas o complejas. Con Azure PowerShell, se pueden ejecutar muchas tareas comunes de administración desde la línea de comandos o en scripts.

### <a name="stop-a-vm"></a>Detención de una máquina virtual

Detenga y desasigne una máquina virtual con [Stop-AzVM](/powershell/module/az.compute/stop-azvm):

```azurepowershell-interactive
Stop-AzVM `
   -ResourceGroupName "myResourceGroupVM" `
   -Name "myVM" -Force
```

Si desea mantener la máquina virtual en un estado aprovisionado, use el parámetro -StayProvisioned.

### <a name="start-a-vm"></a>Inicio de una máquina virtual

```azurepowershell-interactive
Start-AzVM `
   -ResourceGroupName "myResourceGroupVM" `
   -Name "myVM"
```

### <a name="delete-resource-group"></a>Eliminación de un grupo de recursos

Todo el contenido dentro de un grupo de recursos se elimina al eliminar el grupo de recursos.

```azurepowershell-interactive
Remove-AzResourceGroup `
   -Name "myResourceGroupVM" `
   -Force
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido conceptos básicos sobre la creación y administración de máquinas virtuales. Por ejemplo:

> [!div class="checklist"]
> * Crear y conectar elementos a una máquina virtual
> * Seleccionar y usar imágenes de máquinas virtuales
> * Ver y usar tamaños de una máquina virtual específicos
> * Cambiar el tamaño de una máquina virtual
> * Ver y entender el estado de las máquinas virtuales.

Prosiga con el siguiente tutorial para aprender sobre los discos en máquinas virtuales.  

> [!div class="nextstepaction"]
> [Creación y administración de discos de máquinas virtuales](./tutorial-manage-data-disk.md)
