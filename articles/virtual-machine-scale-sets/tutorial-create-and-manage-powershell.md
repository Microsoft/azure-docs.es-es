---
title: 'Tutorial: Creación y administración de un conjunto de escalado de máquinas virtuales de Azure: Azure PowerShell'
description: Aprenda a usar Azure PowerShell para crear un conjunto de escalado de máquinas virtuales, junto con algunas tareas de administración comunes, por ejemplo, cómo iniciar y detener una instancia o cambiar la capacidad del conjunto de escalado.
author: ju-shim
ms.author: jushiman
ms.topic: tutorial
ms.service: virtual-machine-scale-sets
ms.subservice: management
ms.date: 05/18/2018
ms.reviewer: mimckitt
ms.custom: mimckitt, devx-track-azurepowershell
ms.openlocfilehash: 1ab52c197ee57351a53919291f1c2e41108396ec
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89078544"
---
# <a name="tutorial-create-and-manage-a-virtual-machine-scale-set-with-azure-powershell"></a>Tutorial: Creación y administración de un conjunto de escalado de máquinas virtuales con Azure PowerShell

El conjunto de escalado de máquinas virtuales le permite implementar y administrar un conjunto de máquinas virtuales de escalado automático idénticas. Durante el ciclo de vida de la máquina virtual, es posible que deba ejecutar una o varias tareas de administración. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear un conjunto de escalado de máquinas virtuales y conectarse a él
> * Seleccionar y usar imágenes de máquinas virtuales
> * Ver y usar tamaños específicos de instancia de máquina virtual
> * Escalar manualmente un conjunto de escalado
> * Realizar tareas comunes de administración del conjunto de escalado

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [updated-for-az.md](../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]



## <a name="create-a-resource-group"></a>Crear un grupo de recursos
Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. Se debe crear un grupo de recursos antes de un conjunto de escalado de máquinas virtuales. Cree un grupo de recursos con el comando [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). En este ejemplo se crea un grupo de recursos denominado *myResourceGroup* en la región *EastUS*. 

```azurepowershell-interactive
New-AzResourceGroup -ResourceGroupName "myResourceGroup" -Location "EastUS"
```
El nombre del grupo de recursos se especifica al crear o modificar un conjunto de escalado mediante este tutorial.


## <a name="create-a-scale-set"></a>Creación de un conjunto de escalado
En primer lugar, establezca un nombre de usuario de administrador y una contraseña para las instancias de máquina virtual con [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential?view=powershell-5.1):

```azurepowershell-interactive
$cred = Get-Credential
```

Ahora, cree un conjunto de escalado de máquinas virtuales con [New-AzVmss](/powershell/module/az.compute/new-azvmss). Para distribuir el tráfico a las instancias individuales de VM, también se crea un equilibrador de carga. El equilibrador de carga incluye reglas para distribuir el tráfico en el puerto TCP 80, así como también permite el tráfico de Escritorio remoto en el puerto TCP 3389 y la conexión remota de PowerShell en el puerto TCP 5985:

```azurepowershell-interactive
New-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -VMScaleSetName "myScaleSet" `
  -Location "EastUS" `
  -VirtualNetworkName "myVnet" `
  -SubnetName "mySubnet" `
  -PublicIpAddressName "myPublicIPAddress" `
  -LoadBalancerName "myLoadBalancer" `
  -Credential $cred
```

Se tardan unos minutos en crear y configurar todos los recursos del conjunto de escalado y las instancias de máquina virtual.

> [!IMPORTANT]
> Si no puede conectarse al conjunto de escalado, puede que tenga que crear un grupo de seguridad de red agregando el parámetro *[-SecurityGroupName "mySecurityGroup"](/powershell/module/az.compute/new-azvmss)* .


## <a name="view-the-vm-instances-in-a-scale-set"></a>Visualización de las instancias de máquina virtual de un conjunto de escalado
Para ver una lista de las instancias de máquina virtual en un conjunto de escalado, use [Get-AzVmssVM](/powershell/module/az.compute/get-azvmssvm) de la forma siguiente:

```azurepowershell-interactive
Get-AzVmssVM -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet"
```

La salida del ejemplo siguiente muestra dos instancias de máquina virtual del conjunto de escalado:

```powershell
ResourceGroupName         Name Location             Sku InstanceID ProvisioningState
-----------------         ---- --------             --- ---------- -----------------
MYRESOURCEGROUP   myScaleSet_0   eastus Standard_DS1_v2          0         Succeeded
MYRESOURCEGROUP   myScaleSet_1   eastus Standard_DS1_v2          1         Succeeded
```

Para ver información adicional acerca de una instancia específica de la máquina virtual, agregue el parámetro `-InstanceId` a [Get-AzVmssVM](/powershell/module/az.compute/get-azvmssvm). En el ejemplo siguiente, se ve información sobre la instancia de máquina virtual *1*:

```azurepowershell-interactive
Get-AzVmssVM -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId "1"
```


## <a name="list-connection-information"></a>Visualización de información de conexión
Se asigna una dirección IP pública al equilibrador de carga que enruta el tráfico a las instancias de máquina virtual individuales. De manera predeterminada, se agregan reglas de traducción de direcciones de red (NAT) a Azure Load Balancer, que reenvía el tráfico de conexión remota a cada máquina virtual en un puerto dado. Para conectarse a las instancias de máquina virtual de un conjunto de escalado, creará una conexión remota a una dirección IP pública y un número de puerto asignados.

Para obtener una lista de los puertos NAT para conectarse a instancias de máquina virtual en un conjunto de escalado, primero obtenga el objeto de equilibrador de carga con el comando [Get-AzLoadBalancer](/powershell/module/az.network/Get-AzLoadBalancer). Después, consulte las reglas NAT de entrada con [Get-AzLoadBalancerInboundNatRuleConfig](/powershell/module/az.network/Get-AzLoadBalancerInboundNatRuleConfig):


```azurepowershell-interactive
# Get the load balancer object
$lb = Get-AzLoadBalancer -ResourceGroupName "myResourceGroup" -Name "myLoadBalancer"

# View the list of inbound NAT rules
Get-AzLoadBalancerInboundNatRuleConfig -LoadBalancer $lb | Select-Object Name,Protocol,FrontEndPort,BackEndPort
```

La salida del ejemplo siguiente muestra el nombre de instancia, la dirección IP pública del equilibrador de carga y el número de puerto al que las reglas NAT reenvían el tráfico:

```powershell
Name             Protocol FrontendPort BackendPort
----             -------- ------------ -----------
myScaleSet3389.0 Tcp             50001        3389
myScaleSet5985.0 Tcp             51001        5985
myScaleSet3389.1 Tcp             50002        3389
myScaleSet5985.1 Tcp             51002        5985
```

El *nombre* de la regla se alinea con el nombre de la instancia de máquina virtual como se muestra en un comando [Get-AzVmssVM](/powershell/module/az.compute/get-azvmssvm) anterior. Por ejemplo, para conectarse a la instancia de máquina virtual *0*, usa *myScaleSet3389.0* y se conecta al puerto *50001*. Para conectarse a la instancia de máquina virtual *1*, utilice el valor de *myScaleSet3389.1* y conéctese al puerto *50002*. Para usar la comunicación remota de PowerShell, debe conectarse a la regla de instancia de máquina virtual correspondiente en *TCP* puerto *5985*.

Consulte la dirección IP pública del equilibrador de carga con [Get-AzPublicIpAddress](/powershell/module/az.network/Get-AzPublicIpAddress):


```azurepowershell-interactive
Get-AzPublicIpAddress -ResourceGroupName "myResourceGroup" -Name "myPublicIPAddress" | Select IpAddress
```

Salida de ejemplo:

```powershell
IpAddress
---------
52.168.121.216
```

Cree una conexión remota a la primera instancia de máquina virtual. Especifique la dirección IP pública y el número de puerto de la instancia de máquina virtual necesaria, tal como se muestra en los comandos anteriores. Cuando se le solicite, escriba las credenciales que ha utilizado al crear el conjunto de escalado (de forma predeterminada en los comandos de ejemplo, *azureuser* y *P\@ssw0rd!* ). Si se utiliza Azure Cloud Shell, realice este paso desde un símbolo del sistema local de PowerShell o un cliente de Escritorio remoto. En el ejemplo siguiente, se realiza una conexión a una instancia de máquina virtual *1*:

```powershell
mstsc /v 52.168.121.216:50001
```

Una vez que inicia sesión en la instancia de máquina virtual, puede realizar algunos cambios de configuración manual según sea necesario. De momento, cierre la conexión remota.


## <a name="understand-vm-instance-images"></a>Imágenes de instancia de máquina virtual
Azure Marketplace incluye muchas imágenes que pueden usarse para crear instancias de máquina virtual. Para ver una lista de publicadores disponibles, use el comando [Get-AzVMImagePublisher](/powershell/module/az.compute/get-azvmimagepublisher).

```azurepowershell-interactive
Get-AzVMImagePublisher -Location "EastUS"
```

Para ver una lista de imágenes de un publicador determinado, utilice el comando [Get-AzVMImageSku](/powershell/module/az.compute/get-azvmimagesku). También puede filtrar la lista de imágenes por `-PublisherName` u `-Offer`. En el siguiente ejemplo, la lista se filtra por todas las imágenes con el nombre del editor de *MicrosoftWindowsServer* y una oferta que coincida con *WindowsServer*:

```azurepowershell-interactive
Get-AzVMImageSku -Location "EastUS" -PublisherName "MicrosoftWindowsServer" -Offer "WindowsServer"
```

En la siguiente salida de ejemplo se muestran todas las imágenes de Windows Server disponibles:

```powershell
Skus                                  Offer         PublisherName          Location
----                                  -----         -------------          --------
2008-R2-SP1                           WindowsServer MicrosoftWindowsServer eastus
2008-R2-SP1-smalldisk                 WindowsServer MicrosoftWindowsServer eastus
2012-Datacenter                       WindowsServer MicrosoftWindowsServer eastus
2012-Datacenter-smalldisk             WindowsServer MicrosoftWindowsServer eastus
2012-R2-Datacenter                    WindowsServer MicrosoftWindowsServer eastus
2012-R2-Datacenter-smalldisk          WindowsServer MicrosoftWindowsServer eastus
2016-Datacenter                       WindowsServer MicrosoftWindowsServer eastus
2016-Datacenter-Server-Core           WindowsServer MicrosoftWindowsServer eastus
2016-Datacenter-Server-Core-smalldisk WindowsServer MicrosoftWindowsServer eastus
2016-Datacenter-smalldisk             WindowsServer MicrosoftWindowsServer eastus
2016-Datacenter-with-Containers       WindowsServer MicrosoftWindowsServer eastus
2016-Datacenter-with-RDSH             WindowsServer MicrosoftWindowsServer eastus
2016-Nano-Server                      WindowsServer MicrosoftWindowsServer eastus
```

Cuando creó un conjunto de escalado al inicio del tutorial, se proporcionó una imagen de máquina virtual predeterminada de *Windows Server 2016 DataCenter* para las instancias de máquina virtual. Puede especificar una imagen de instancia de máquina virtual diferente en función de la salida de [Get-AzVMImageSku](/powershell/module/az.compute/get-azvmimagesku). En el ejemplo siguiente, se va a crear un conjunto de escalado con el parámetro `-ImageName` para especificar una imagen de máquina virtual de *MicrosoftWindowsServer:WindowsServer:2016-Datacenter-with-Containers:latest*. Como se tarda unos minutos en crear y configurar todos los recursos del conjunto de escalado y las instancias de máquina virtual, no tiene que implementar el siguiente conjunto de escalado:

```azurepowershell-interactive
New-AzVmss `
  -ResourceGroupName "myResourceGroup2" `
  -Location "EastUS" `
  -VMScaleSetName "myScaleSet2" `
  -VirtualNetworkName "myVnet2" `
  -SubnetName "mySubnet2" `
  -PublicIpAddressName "myPublicIPAddress2" `
  -LoadBalancerName "myLoadBalancer2" `
  -UpgradePolicyMode "Automatic" `
  -ImageName "MicrosoftWindowsServer:WindowsServer:2016-Datacenter-with-Containers:latest" `
  -Credential $cred
```

> [!IMPORTANT]
> Se recomienda usar la versión *más reciente* de la imagen. Especifique "más reciente" para usar la versión más reciente de una imagen disponible en el momento de la implementación. Tenga en cuenta que, aunque use "más reciente", la imagen de la máquina virtual no se actualizará automáticamente después de la implementación, incluso si hay disponible una nueva versión.

## <a name="understand-vm-instance-sizes"></a>Tamaños de instancia de máquina virtual
El tamaño de instancia de la máquina virtual, o *SKU*, determina la cantidad de recursos de proceso, como memoria, CPU y GPU, que están disponibles para la instancia de máquina virtual. Las instancias de máquina virtual de un conjunto de escalado deben tener el tamaño adecuado para la carga de trabajo esperada.

### <a name="vm-instance-sizes"></a>Tamaños de instancia de máquina virtual
En la tabla siguiente se clasifican los tamaños de máquina virtual comunes en casos de uso.

| Tipo                     | Tamaños comunes           |    Descripción       |
|--------------------------|-------------------|------------------------------------------------------------------------------------------------------------------------------------|
| [Uso general](../virtual-machines/sizes-general.md)         |Dsv3, Dv3, DSv2, Dv2, DS, D, Av2, A0-7| Uso equilibrado de CPU y memoria. Ideal para desarrollo/pruebas, así como soluciones de datos y aplicaciones de tamaño pequeño a mediano.  |
| [Proceso optimizado](../virtual-machines/sizes-compute.md)   | Fs, F             | Uso elevado de la CPU respecto a la memoria. Adecuado para aplicaciones, dispositivos de red y procesos por lotes con tráfico mediano.        |
| [Memoria optimizada](../virtual-machines/sizes-memory.md)    | Esv3, Ev3, M, GS, G, DSv2, DS, Dv2, D   | Uso elevado de memoria respecto al núcleo. Excelente para bases de datos relacionales, memorias caché de capacidad de mediana a grande y análisis en memoria.                 |
| [Almacenamiento optimizado](../virtual-machines/sizes-storage.md)      | LS                | Alto rendimiento de disco y E/S. Perfecto para bases de datos SQL y NoSQL y macrodatos.                                                         |
| [GPU](../virtual-machines/sizes-gpu.md)          | NV, NC            | Máquinas virtuales especializadas para actividades intensas de representación de gráficos y edición de vídeo.       |
| [Alto rendimiento](../virtual-machines/sizes-hpc.md) | H, A8-11          | Nuestras máquinas virtuales con CPU más eficaces e interfaces de red de alto rendimiento (RDMA) opcionales. 

### <a name="find-available-vm-instance-sizes"></a>Búsqueda de tamaños de instancia de máquina virtual disponibles
Para ver una lista de tamaños de instancias de máquina virtual disponibles en una región determinada, use el comando [Get-AzVMSize](/powershell/module/az.compute/get-azvmsize). 

```azurepowershell-interactive
Get-AzVMSize -Location "EastUS"
```

La salida es similar al siguiente ejemplo reducido, que muestra los recursos asignados a cada tamaño de máquina virtual:

```powershell
Name                   NumberOfCores MemoryInMB MaxDataDiskCount OSDiskSizeInMB ResourceDiskSizeInMB
----                   ------------- ---------- ---------------- -------------- --------------------
Standard_DS1_v2                    1       3584                4        1047552                 7168
Standard_DS2_v2                    2       7168                8        1047552                14336
[...]
Standard_A0                        1        768                1        1047552                20480
Standard_A1                        1       1792                2        1047552                71680
[...]
Standard_F1                        1       2048                4        1047552                16384
Standard_F2                        2       4096                8        1047552                32768
[...]
Standard_NV6                       6      57344               24        1047552               389120
Standard_NV12                     12     114688               48        1047552               696320
```

Cuando creó un conjunto de escalado al comienzo del tutorial, se proporcionó una SKU de máquina virtual predeterminada de *Standard_DS1_v2* para las instancias de máquina virtual. Puede especificar un tamaño de instancia de máquina virtual diferente en función de la salida de [Get-AzVMSize](/powershell/module/az.compute/get-azvmsize). En el ejemplo siguiente se crearía un conjunto de escalado con el parámetro `-VmSize` para especificar un tamaño de instancia de máquina virtual de *Standard_F1*. Como se tarda unos minutos en crear y configurar todos los recursos del conjunto de escalado y las instancias de máquina virtual, no tiene que implementar el siguiente conjunto de escalado:

```azurepowershell-interactive
New-AzVmss `
  -ResourceGroupName "myResourceGroup3" `
  -Location "EastUS" `
  -VMScaleSetName "myScaleSet3" `
  -VirtualNetworkName "myVnet3" `
  -SubnetName "mySubnet3" `
  -PublicIpAddressName "myPublicIPAddress3" `
  -LoadBalancerName "myLoadBalancer3" `
  -UpgradePolicyMode "Automatic" `
  -VmSize "Standard_F1" `
  -Credential $cred
```


## <a name="change-the-capacity-of-a-scale-set"></a>Cambio de la capacidad de un conjunto de escalado
Cuando crea un conjunto de escalado, se solicitan dos instancias de máquina virtual. Para aumentar o disminuir el número de instancias de máquina virtual en el conjunto de escalado, puede cambiar manualmente la capacidad. El conjunto de escalado crea o quita el número necesario de instancias de máquina virtual y, luego, configura el equilibrador de carga para distribuir el tráfico.

En primer lugar, cree un objeto de conjunto de escalado con [Get-AzVmss](/powershell/module/az.compute/get-azvmss) y, después, especifique un nuevo valor para `sku.capacity`. Para aplicar el cambio de capacidad, use [Update-AzVmss](/powershell/module/az.compute/update-azvmss). En el ejemplo siguiente se establece el número de instancias de máquina virtual del conjunto de escalado en *3*:

```azurepowershell-interactive
# Get current scale set
$vmss = Get-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet"

# Set and update the capacity of your scale set
$vmss.sku.capacity = 3
Update-AzVmss -ResourceGroupName "myResourceGroup" -Name "myScaleSet" -VirtualMachineScaleSet $vmss 
```

La capacidad del conjunto de escalado tarda unos minutos en actualizarse. Para ver el número de instancias que tiene ahora en un conjunto de escalado, use [Get-AzVmss](/powershell/module/az.compute/get-azvmss):

```azurepowershell-interactive
Get-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet"
```

La siguiente salida de ejemplo muestra que la capacidad del conjunto de escalado es ahora *3*:

```powershell
Sku        :
  Name     : Standard_DS2
  Tier     : Standard
  Capacity : 3
```


## <a name="common-management-tasks"></a>Tareas comunes de administración
Ahora puede crear un conjunto de escalado, mostrar información de conexión y conectarse a instancias de máquina virtual. Ha aprendido cómo podría usar otra imagen de sistema operativo para sus instancias de máquina virtual, cómo seleccionar otro tamaño de máquina virtual o cómo escalar manualmente el número de instancias. Como parte de la administración diaria, puede que deba detener, iniciar o reiniciar las instancias de máquina virtual del conjunto de escalado.

### <a name="stop-and-deallocate-vm-instances-in-a-scale-set"></a>Detención y desasignación de instancias de máquina virtual de un conjunto de escalado
Para detener una o varias VM en un conjunto de escalado, use [Stop-AzVmss](/powershell/module/az.compute/stop-azvmss). El parámetro `-InstanceId` le permite especificar una o más máquinas virtuales para que se detengan. Si no especifica un identificador de instancia, se detienen todas las máquinas virtuales del conjunto de escalado. En el ejemplo siguiente se detiene la instancia *1*:

```azurepowershell-interactive
Stop-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId "1"
```

De forma predeterminada, se cancela la asignación de las máquinas virtuales para no acumular cargos de proceso. Si quiere que la máquina virtual permanezca en un estado de aprovisionamiento cuando se detenga, agregue el parámetro `-StayProvisioned` al comando anterior. Las máquinas virtuales detenidas que se mantienen aprovisionadas acumulan cargos regulares de proceso.

### <a name="start-vm-instances-in-a-scale-set"></a>Inicio de las instancias de máquina virtual de un conjunto de escalado
Para iniciar una o varias VM en un conjunto de escalado, use [Start-AzVmss](/powershell/module/az.compute/start-azvmss). El parámetro `-InstanceId` le permite especificar una o más máquinas virtuales para que se inicien. Si no especifica un identificador de instancia, se inician todas las máquinas virtuales del conjunto de escalado. En el ejemplo siguiente se inicia la instancia *1*:

```azurepowershell-interactive
Start-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId "1"
```

### <a name="restart-vm-instances-in-a-scale-set"></a>Reinicio de las instancias de máquina virtual de un conjunto de escalado
Para reiniciar una o varias VM en un conjunto de escalado, use [Restart-AzVmss](/powershell/module/az.compute/restart-azvmss). El parámetro `-InstanceId` le permite especificar una o más máquinas virtuales para que se reinicien. Si no especifica un identificador de instancia, se reinician todas las máquinas virtuales del conjunto de escalado. En el ejemplo siguiente, se reinicia la instancia *1*:

```azurepowershell-interactive
Restart-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myScaleSet" -InstanceId "1"
```


## <a name="clean-up-resources"></a>Limpieza de recursos
Cuando se elimina un grupo de recursos, todos los recursos que contiene, como las instancias de máquina virtual, la red virtual y los discos, también se eliminan. El parámetro `-Force` confirma que desea eliminar los recursos sin pedir confirmación adicional. El parámetro `-AsJob` devuelve el control a la petición de confirmación sin esperar a que finalice la operación.

```azurepowershell-interactive
Remove-AzResourceGroup -Name "myResourceGroup" -Force -AsJob
```


## <a name="next-steps"></a>Pasos siguientes
En este tutorial, ha aprendido cómo realizar algunas tareas básicas de creación y administración del conjunto de escalado con Azure PowerShell:

> [!div class="checklist"]
> * Crear un conjunto de escalado de máquinas virtuales y conectarse a él
> * Seleccionar y usar imágenes de máquinas virtuales
> * Ver y usar tamaños de una máquina virtual específicos
> * Escalar manualmente un conjunto de escalado
> * Realizar tareas comunes de administración del conjunto de escalado

Vaya al siguiente tutorial para aprender sobre los discos de conjuntos de escalado.

> [!div class="nextstepaction"]
> [Uso de discos de datos con conjuntos de escalado](tutorial-use-disks-powershell.md)
