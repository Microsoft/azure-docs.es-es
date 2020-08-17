---
title: 'Tutorial: Creación y uso de discos para conjuntos de escalado con Azure PowerShell'
description: Obtenga información sobre cómo usar Azure PowerShell para crear y usar discos administrados con conjuntos de escalado de máquinas virtuales, incluido cómo agregar, preparar, enumerar y desconectar discos.
author: ju-shim
ms.author: jushiman
ms.topic: tutorial
ms.service: virtual-machine-scale-sets
ms.subservice: disks
ms.date: 03/27/2018
ms.reviewer: mimckitt
ms.custom: mimckitt
ms.openlocfilehash: ad610339d79eb0fd77403957c252a36e8243eb87
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87837274"
---
# <a name="tutorial-create-and-use-disks-with-virtual-machine-scale-set-with-azure-powershell"></a>Tutorial: Creación y uso de discos con conjuntos de escalado de máquinas virtuales con Azure PowerShell

Los conjuntos de escalado de máquinas virtuales usan discos para almacenar el sistema operativo, las aplicaciones y los datos de las máquinas virtuales. Al crear y administrar un conjunto de escalado, es importante elegir un tamaño de disco y la configuración adecuada para la carga de trabajo esperada. Este tutorial explica cómo crear y administrar discos de máquina virtual. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Discos del SO y temporales.
> * Discos de datos.
> * Discos Estándar y Premium.
> * Rendimiento de disco.
> * Conexión y preparación de los discos de datos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [updated-for-az.md](../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]


## <a name="default-azure-disks"></a>Discos de Azure predeterminados
Cuando se crea o se escala un conjunto de escalado, se conectan automáticamente dos discos a cada instancia de máquina virtual. 

**Disco del sistema operativo**: hospeda el sistema operativo de las instancias de máquina virtual. Pueden tener un tamaño de hasta 2 terabytes. El disco del sistema operativo lleva de forma predeterminada la etiqueta */dev/sda* . La configuración de almacenamiento en caché del disco del sistema operativo está optimizada para el rendimiento del sistema operativo. Debido a esta configuración, el disco del sistema operativo **no debe** hospedar aplicaciones o datos. Para aplicaciones y datos, use discos de datos, que se detallan más adelante en este artículo. 

**Disco temporal**: los discos temporales usan una unidad de estado sólido que se encuentra en el mismo host de Azure que la instancia de máquina virtual. Son discos de gran rendimiento y se pueden usar para operaciones tales como el procesamiento temporal de los datos. Sin embargo, si la instancia de máquina virtual se mueve a un nuevo host, los datos almacenados en un disco temporal se eliminarán. El tamaño del disco temporal se determina por el tamaño de la instancia de máquina virtual. Los discos temporales llevan la etiqueta */dev/sdb* y tienen un punto de montaje de */mnt*.

### <a name="temporary-disk-sizes"></a>Tamaños de disco temporal
| Tipo | Tamaños comunes | Tamaño máximo de disco temporal (GiB) |
|----|----|----|
| [Uso general](../virtual-machines/sizes-general.md) | Series A, B y D | 1600 |
| [Proceso optimizado](../virtual-machines/sizes-compute.md) | Serie F | 576 |
| [Memoria optimizada](../virtual-machines/sizes-memory.md) | Series D, E, G y M | 6144 |
| [Almacenamiento optimizado](../virtual-machines/sizes-storage.md) | Serie L | 5630 |
| [GPU](../virtual-machines/sizes-gpu.md) | Serie N | 1440 |
| [Alto rendimiento](../virtual-machines/sizes-hpc.md) | Series A y H | 2000 |


## <a name="azure-data-disks"></a>Discos de datos de Azure
Se pueden agregar discos de datos adicionales si necesita instalar aplicaciones y almacenar datos. Los discos de datos deben usarse en cualquier situación donde desee un almacenamiento de datos duradero y con capacidad de respuesta. Cada disco de datos tiene una capacidad máxima de 4 TB. El tamaño de la instancia de máquina virtual determina cuántos discos de datos se pueden conectar. Para cada vCPU de la máquina virtual, se pueden asociar dos discos de datos.

### <a name="max-data-disks-per-vm"></a>Discos de datos máximos por máquina virtual
| Tipo | Tamaños comunes | Discos de datos máximos por máquina virtual |
|----|----|----|
| [Uso general](../virtual-machines/sizes-general.md) | Series A, B y D | 64 |
| [Proceso optimizado](../virtual-machines/sizes-compute.md) | Serie F | 64 |
| [Memoria optimizada](../virtual-machines/sizes-memory.md) | Series D, E, G y M | 64 |
| [Almacenamiento optimizado](../virtual-machines/sizes-storage.md) | Serie L | 64 |
| [GPU](../virtual-machines/sizes-gpu.md) | Serie N | 64 |
| [Alto rendimiento](../virtual-machines/sizes-hpc.md) | Series A y H | 64 |


## <a name="vm-disk-types"></a>Tipos de disco de máquina virtual
Azure proporciona dos tipos de disco.

### <a name="standard-disk"></a>Disco estándar
Standard Storage está respaldado por unidades de disco duro y ofrece un almacenamiento rentable con un buen rendimiento. Los discos estándar son ideales para cargas de trabajo de desarrollo y prueba rentables.

### <a name="premium-disk"></a>Disco Premium
Los discos Premium están respaldados por discos de latencia reducida y alto rendimiento basados en SSD. Estos discos se recomiendan para las máquinas virtuales que ejecutan cargas de trabajo de producción. Premium Storage es compatible con las máquinas virtuales de las series DS, DSv2, GS y FS. Al seleccionar el tamaño de un disco, el valor se redondea al alza al siguiente tipo. Por ejemplo, si el tamaño del disco es inferior a 128 GB, el tipo de disco es P10. Si el tamaño de disco está entre 129 y 512 GB, el tamaño es un P20. Cualquier tamaño por encima de 512 GB equivale a un tipo P30.

### <a name="premium-disk-performance"></a>Rendimiento del disco Premium
|Tipo de disco de Premium Storage | P4 | P6 | P10 | P20 | P30 | P40 | P50 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Tamaño del disco (redondeo hacia arriba) | 32 GB | 64 GB | 128 GB | 512 GB | 1024 GB (1 TB) | 2048 GB (2 TB) | 4095 GB (4 TB) |
| Máximo de IOPS por disco | 120 | 240 | 500 | 2,300 | 5\.000 | 7500 | 7500 |
Rendimiento de disco. | 25 MB/s | 50 MB/s | 100 MB/s | 150 MB/s | 200 MB/s | 250 MB/s | 250 MB/s |

Aunque la tabla anterior identifica las IOPS máximas por disco, se puede obtener un mayor nivel de rendimiento dividiendo varios discos de datos. Por ejemplo, una máquina virtual Standard_GS5 puede conseguir 80 000 IOPS como máximo. Para más información sobre el número máximo de IOPS por máquina virtual, consulte los [tamaños de máquinas virtuales Windows](../virtual-machines/sizes.md).


## <a name="create-and-attach-disks"></a>Creación y conexión de discos
Puede crear y conectar discos durante la creación de un conjunto de escalado o a un conjunto de escalado existente.

### <a name="attach-disks-at-scale-set-creation"></a>Conexión de discos durante la creación del conjunto de escalado
Cree un conjunto de escalado de máquinas virtuales con [New-AzVmss](/powershell/module/az.compute/new-azvmss). Cuando se le solicite, escriba el nombre de usuario y la contraseña de las instancias de máquina virtual. Para distribuir el tráfico a las instancias individuales de VM, también se crea un equilibrador de carga. El equilibrador de carga incluye reglas para distribuir el tráfico en el puerto TCP 80, y permitir el tráfico de Escritorio remoto en el puerto TCP 3389 y la conexión remota de PowerShell en el puerto TCP 5985.

Se crean dos discos con el parámetro `-DataDiskSizeGb`. El primer disco tiene un tamaño de *64* GB y, el segundo disco, de *128* GB: Cuando se le solicite, proporcione sus propias credenciales administrativas para las instancias de máquina virtual en el conjunto de escalado:

```azurepowershell-interactive
New-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -Location "EastUS" `
  -VMScaleSetName "myScaleSet" `
  -VirtualNetworkName "myVnet" `
  -SubnetName "mySubnet" `
  -PublicIpAddressName "myPublicIPAddress" `
  -LoadBalancerName "myLoadBalancer" `
  -UpgradePolicyMode "Automatic" `
  -DataDiskSizeInGb 64,128
```

Se tardan unos minutos en crear y configurar todos los recursos del conjunto de escalado y las instancias de máquina virtual.

### <a name="attach-a-disk-to-existing-scale-set"></a>Conexión de un disco a un conjunto de escalado existente
También puede conectar discos a un conjunto de escalado existente. Use el conjunto de escalado creado en el paso anterior para agregar otro disco con [Add-AzVmssDataDisk](/powershell/module/az.compute/add-azvmssdatadisk). En el ejemplo siguiente, se conecta un disco adicional de *128* GB a un conjunto de escalado existente:

```azurepowershell-interactive
# Get scale set object
$vmss = Get-AzVmss `
          -ResourceGroupName "myResourceGroup" `
          -VMScaleSetName "myScaleSet"

# Attach a 128 GB data disk to LUN 2
Add-AzVmssDataDisk `
  -VirtualMachineScaleSet $vmss `
  -CreateOption Empty `
  -Lun 2 `
  -DiskSizeGB 128

# Update the scale set to apply the change
Update-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -Name "myScaleSet" `
  -VirtualMachineScaleSet $vmss
```


## <a name="prepare-the-data-disks"></a>Preparación de los discos de datos
Los discos que se crean y conectan a las instancias de máquina virtual del conjunto de escalado son discos sin formato. Para poder usarlos con sus aplicaciones y datos, los discos deben prepararse. Para preparar los discos, cree una partición, cree un sistema de archivos y móntelos.

Para automatizar el proceso en varias instancias de máquina virtual en un conjunto de escalado, puede usar la extensión de script personalizado de Azure. Esta extensión puede ejecutar scripts localmente en cada instancia de máquina virtual, por ejemplo, para preparar los discos de datos conectados. Para obtener más información, consulte [Información general de la extensión de script personalizado](../virtual-machines/extensions/custom-script-windows.md).


En el ejemplo siguiente se ejecuta un script desde un repositorio de ejemplo de GitHub en cada instancia de máquina virtual con [Add-AzVmssExtension](/powershell/module/az.compute/Add-AzVmssExtension), que prepara todos los discos de datos conectados sin formato:


```azurepowershell-interactive
# Get scale set object
$vmss = Get-AzVmss `
          -ResourceGroupName "myResourceGroup" `
          -VMScaleSetName "myScaleSet"

# Define the script for your Custom Script Extension to run
$publicSettings = @{
  "fileUris" = (,"https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/prepare_vm_disks.ps1");
  "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File prepare_vm_disks.ps1"
}

# Use Custom Script Extension to prepare the attached data disks
Add-AzVmssExtension -VirtualMachineScaleSet $vmss `
  -Name "customScript" `
  -Publisher "Microsoft.Compute" `
  -Type "CustomScriptExtension" `
  -TypeHandlerVersion 1.8 `
  -Setting $publicSettings

# Update the scale set and apply the Custom Script Extension to the VM instances
Update-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -Name "myScaleSet" `
  -VirtualMachineScaleSet $vmss
```

Para confirmar que los discos se han preparado correctamente, conéctese mediante RDP a una de las instancias de máquina virtual. 

En primer lugar, obtenga el objeto de equilibrador de carga con [Get-AzLoadBalancer](/powershell/module/az.network/Get-AzLoadBalancer). Después, consulte las reglas NAT de entrada con [Get-AzLoadBalancerInboundNatRuleConfig](/powershell/module/az.network/Get-AzLoadBalancerInboundNatRuleConfig). Las reglas NAT muestran el valor de *FrontendPort* para cada instancia de máquina virtual en la que RDP escucha. Por último, consiga la dirección IP pública del equilibrador de carga con [Get-AzPublicIpAddress](/powershell/module/az.network/Get-AzPublicIpAddress):


```azurepowershell-interactive
# Get the load balancer object
$lb = Get-AzLoadBalancer -ResourceGroupName "myResourceGroup" -Name "myLoadBalancer"

# View the list of inbound NAT rules
Get-AzLoadBalancerInboundNatRuleConfig -LoadBalancer $lb | Select-Object Name,Protocol,FrontEndPort,BackEndPort

# View the public IP address of the load balancer
Get-AzPublicIpAddress -ResourceGroupName "myResourceGroup" -Name myPublicIPAddress | Select IpAddress
```

Para conectar con su máquina virtual, especifique su propia dirección IP pública y el número de puerto de la instancia de máquina virtual necesaria, tal y como se muestra en los comandos anteriores. Cuando se le solicite, escriba las credenciales usadas al crear el conjunto de escalado. Si se utiliza Azure Cloud Shell, realice este paso desde un símbolo del sistema local de PowerShell o un cliente de Escritorio remoto. En el ejemplo siguiente, se realiza una conexión a una instancia de máquina virtual *1*:

```powershell
mstsc /v 52.168.121.216:50001
```

Abra una sesión de PowerShell local en la instancia de máquina virtual y examine los discos conectados con [Get-Disk](/powershell/module/storage/get-disk):

```powershell
Get-Disk
```

La salida del ejemplo siguiente muestra que los tres discos de datos están conectados con la instancia de máquina virtual.

```powershell
Number  Friendly Name      HealthStatus  OperationalStatus  Total Size  Partition Style
------  -------------      ------------  -----------------  ----------  ---------------
0       Virtual HD         Healthy       Online                 127 GB  MBR
1       Virtual HD         Healthy       Online                  14 GB  MBR
2       Msft Virtual Disk  Healthy       Online                  64 GB  MBR
3       Msft Virtual Disk  Healthy       Online                 128 GB  MBR
4       Msft Virtual Disk  Healthy       Online                 128 GB  MBR
```

Examine los sistemas de archivos y los puntos de montaje en la instancia de máquina virtual, de la siguiente manera:

```powershell
Get-Partition
```

La salida del ejemplo siguiente muestra que los tres discos de datos tienen asignadas letras de unidad:

```powershell
   DiskPath: \\?\scsi#disk&ven_msft&prod_virtual_disk#000001

PartitionNumber  DriveLetter  Offset   Size  Type
---------------  -----------  ------   ----  ----
1                F            1048576  64 GB  IFS

   DiskPath: \\?\scsi#disk&ven_msft&prod_virtual_disk#000002

PartitionNumber  DriveLetter  Offset   Size   Type
---------------  -----------  ------   ----   ----
1                G            1048576  128 GB  IFS

   DiskPath: \\?\scsi#disk&ven_msft&prod_virtual_disk#000003

PartitionNumber  DriveLetter  Offset   Size   Type
---------------  -----------  ------   ----   ----
1                H            1048576  128 GB  IFS
```

Los discos en cada instancia de máquina virtual del conjunto de escalado se preparan automáticamente de la misma manera. A medida que conjunto de escalado escale verticalmente, se conectan los discos de datos necesarios a las nuevas instancias de máquina virtual. La extensión de script personalizado también se ejecuta para preparar automáticamente los discos.

Cierre la sesión de conexión a Escritorio remoto con la instancia de máquina virtual.


## <a name="list-attached-disks"></a>Enumeración de los discos conectados
Para ver información acerca de los discos conectados a un conjunto de escalado, use [Get-AzVmss](/powershell/module/az.compute/get-azvmss) de la siguiente manera:

```azurepowershell-interactive
Get-AzVmss -ResourceGroupName "myResourceGroup" -Name "myScaleSet"
```

En la propiedad *VirtualMachineProfile.StorageProfile*, se muestra la lista de *DataDisks*. Se muestra información sobre el tamaño del disco, la capa de almacenamiento y el LUN (número de unidad lógica). La salida del ejemplo siguiente muestra los tres discos de datos conectados con el conjunto de escalado:

```powershell
DataDisks[0]                            :
  Lun                                   : 0
  Caching                               : None
  CreateOption                          : Empty
  DiskSizeGB                            : 64
  ManagedDisk                           :
    StorageAccountType                  : PremiumLRS
DataDisks[1]                            :
  Lun                                   : 1
  Caching                               : None
  CreateOption                          : Empty
  DiskSizeGB                            : 128
  ManagedDisk                           :
    StorageAccountType                  : PremiumLRS
DataDisks[2]                            :
  Lun                                   : 2
  Caching                               : None
  CreateOption                          : Empty
  DiskSizeGB                            : 128
  ManagedDisk                           :
    StorageAccountType                  : PremiumLRS
```


## <a name="detach-a-disk"></a>Desconexión de un disco
Cuando ya no se necesita un disco determinado, se puede desconectar del conjunto de escalado. El disco se quita de todas las instancias de máquina virtual del conjunto de escalado. Para desconectar un disco de un conjunto de escalado, use [Remove-AzVmssDataDisk](/powershell/module/az.compute/remove-azvmssdatadisk) y especifique el LUN del disco. Los LUN se muestran en la salida de [Get-AzVmss](/powershell/module/az.compute/get-azvmss), en la sección anterior. En el ejemplo siguiente, se desconecta el LUN *3* del conjunto de escalado:

```azurepowershell-interactive
# Get scale set object
$vmss = Get-AzVmss `
          -ResourceGroupName "myResourceGroup" `
          -VMScaleSetName "myScaleSet"

# Detach a disk from the scale set
Remove-AzVmssDataDisk `
  -VirtualMachineScaleSet $vmss `
  -Lun 2

# Update the scale set and detach the disk from the VM instances
Update-AzVmss `
  -ResourceGroupName "myResourceGroup" `
  -Name "myScaleSet" `
  -VirtualMachineScaleSet $vmss
```


## <a name="clean-up-resources"></a>Limpieza de recursos
Para quitar el conjunto de escalado y los discos, elimine el grupo de recursos y todos sus recursos con [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup). El parámetro `-Force` confirma que desea eliminar los recursos sin pedir confirmación adicional. El parámetro `-AsJob` devuelve el control a la petición de confirmación sin esperar a que finalice la operación.

```azurepowershell-interactive
Remove-AzResourceGroup -Name "myResourceGroup" -Force -AsJob
```


## <a name="next-steps"></a>Pasos siguientes
En este tutorial, aprendió a crear y utilizar discos con conjuntos de escalado con Azure PowerShell:

> [!div class="checklist"]
> * Discos del SO y temporales.
> * Discos de datos.
> * Discos Estándar y Premium.
> * Rendimiento de disco.
> * Conexión y preparación de los discos de datos

Vaya al siguiente tutorial para aprender a usar una imagen personalizada para las instancias de máquina virtual de su conjunto de escalado.

> [!div class="nextstepaction"]
> [Uso de una imagen personalizada para las instancias de máquina virtual de un conjunto de escalado](tutorial-use-custom-image-powershell.md)