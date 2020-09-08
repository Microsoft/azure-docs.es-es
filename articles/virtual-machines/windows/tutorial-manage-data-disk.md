---
title: 'Tutorial: Administración de discos de Azure con Azure PowerShell'
description: En este tutorial, aprenderá a usar Azure PowerShell para crear y administrar discos de Azure para máquinas virtuales
author: cynthn
ms.service: virtual-machines-windows
ms.subservice: disks
ms.topic: tutorial
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 11/29/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 528fe5dea533faf9447e03dd901568d783891ce9
ms.sourcegitcommit: 6fc156ceedd0fbbb2eec1e9f5e3c6d0915f65b8e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88718941"
---
# <a name="tutorial---manage-azure-disks-with-azure-powershell"></a>Tutorial: Administración de discos de Azure con Azure PowerShell

Las máquinas virtuales de Azure usan discos para almacenar el sistema operativo, las aplicaciones y los datos de máquinas virtuales. Al crear una máquina virtual es importante elegir un tamaño de disco y la configuración adecuada para la carga de trabajo esperada. Este tutorial trata la implementación y administración de discos de máquina virtual. Aprenderá sobre los siguientes temas:

> [!div class="checklist"]
> * Discos del SO y temporales.
> * Discos de datos.
> * Discos Estándar y Premium.
> * Rendimiento de disco.
> * Conectar y preparar los discos de datos

## <a name="launch-azure-cloud-shell"></a>Inicio de Azure Cloud Shell

Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. 

Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior derecha de un bloque de código. También puede ir a [https://shell.azure.com/powershell](https://shell.azure.com/powershell) para iniciar Cloud Shell en una pestaña independiente del explorador. Seleccione **Copiar** para copiar los bloques de código, péguelos en Cloud Shell y, luego, presione Entrar para ejecutarlos.

## <a name="default-azure-disks"></a>Discos de Azure predeterminados

Cuando se crea una máquina virtual de Azure, se conectan dos discos automáticamente a la máquina virtual. 

**Disco del sistema operativo**: hospeda el sistema operativo de las máquinas virtuales. Se puede cambiar su tamaño hasta 4 terabyte. Si crea una nueva máquina virtual a partir de una imagen de [Azure Marketplace](https://azure.microsoft.com/marketplace/), normalmente será de 127 GB (aunque algunas imágenes tienen discos del sistema operativo más pequeños). Al disco del sistema operativo se le asigna la letra de unidad *C:* de forma predeterminada. La configuración de almacenamiento en caché del disco del sistema operativo está optimizada para el rendimiento del sistema operativo. El disco del sistema operativo **no debería** hospedar aplicaciones o datos. Para aplicaciones y datos, use un disco de datos. Explicamos esto más adelante en este artículo.

**Disco temporal**: los discos temporales usan una unidad de estado sólido que se encuentra en el mismo host de Azure que la máquina virtual. Los discos temporales son muy eficiente y se pueden usar para operaciones tales como el procesamiento temporal de los datos. Sin embargo, si la máquina virtual se mueve a un nuevo host, los datos almacenados en un disco temporal se eliminarán. El tamaño del disco temporal lo determina el [tamaño de la máquina virtual](../sizes.md). A los discos temporales se les asigna la letra de unidad *D:* de forma predeterminada.

## <a name="azure-data-disks"></a>Discos de datos de Azure

Se pueden agregar discos de datos adicionales para instalar aplicaciones y almacenar datos. Los discos de datos deben usarse en cualquier situación donde se necesite un almacenamiento de datos duradero y con capacidad de respuesta. El tamaño de la máquina virtual determina cuántos discos de datos se pueden conectar a una máquina virtual.

## <a name="vm-disk-types"></a>Tipos de disco de máquina virtual

Azure proporciona dos tipos de discos.

**Discos estándar**: respaldados por unidades de disco duro, ofrecen un almacenamiento rentable y buen rendimiento. Los discos estándar son ideales para cargas de trabajo de desarrollo y prueba rentables.

**Discos Premium**: respaldados por un disco de latencia reducida y alto rendimiento basado en SSD. Es perfecto para máquinas virtuales que ejecutan cargas de trabajo de producción. Los tamaños de máquina virtual con una letra **S** en el [nombre de tamaño](../vm-naming-conventions.md), normalmente admiten Premium Storage. Por ejemplo, las máquinas virtuales de las series DS, DSv2, GS y FS admiten Premium Storage. Al seleccionar el tamaño de un disco, el valor se redondea al alza al siguiente tipo. Por ejemplo, si el tamaño del disco es superior a 64 GB, pero inferior a 128 GB, el tipo de disco es P10. 
<br>
[!INCLUDE [disk-storage-premium-ssd-sizes](../../../includes/disk-storage-premium-ssd-sizes.md)]

Cuando se aprovisiona un disco de Premium Storage, a diferencia de Standard Storage, se garantizan la capacidad, las E/S por segundo y el rendimiento del mismo. Por ejemplo, si crea un disco P50, Azure aprovisiona una capacidad de almacenamiento de 4095 GB, 7500 E/S por segundo y un rendimiento de 250 MB/s para él. La aplicación puede usar toda la capacidad y el rendimiento o parte de ellos. Los discos SSD Premium están diseñados para proporcionar bajas latencias inferiores a 10 milisegundos y un IOPS y rendimiento que se describen en la tabla anterior como del 99,9 % del tiempo.

Aunque la tabla anterior identifica las IOPS máximas por disco, se puede obtener un mayor nivel de rendimiento dividiendo varios discos de datos. Por ejemplo, 64 discos de datos pueden conectarse a la máquina virtual Standard_GS5. Si cada uno de estos discos tiene un tamaño P30, se puede lograr un máximo de 80 000 IOPS. Para más información sobre el número máximo de IOPS por máquina virtual, vea los [tamaños y topos de máquinas virtuales](../sizes.md).

## <a name="create-and-attach-disks"></a>Creación y conexión de discos

Para completar el ejemplo de este tutorial, debe tener una máquina virtual. Si es necesario, cree una máquina virtual con los siguientes comandos.

Establezca el nombre de usuario y la contraseña que se necesitan para la cuenta de administrador en la máquina virtual con [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential?view=powershell-5.1):


Cree la máquina virtual con [New-AzVM](/powershell/module/az.compute/new-azvm). Se le pedirá que escriba un nombre de usuario y una contraseña para la cuenta de administrador de la máquina virtual.

```azurepowershell-interactive
New-AzVm `
    -ResourceGroupName "myResourceGroupDisk" `
    -Name "myVM" `
    -Location "East US" `
    -VirtualNetworkName "myVnet" `
    -SubnetName "mySubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -PublicIpAddressName "myPublicIpAddress" 
```


Cree la configuración inicial con [New-AzDiskConfig](/powershell/module/az.compute/new-azdiskconfig). En el ejemplo siguiente se crea un disco denominado con un tamaño de 128 gigabytes.

```azurepowershell-interactive
$diskConfig = New-AzDiskConfig `
    -Location "EastUS" `
    -CreateOption Empty `
    -DiskSizeGB 128
```

Cree el disco de datos con el comando [New-AzDisk](/powershell/module/az.compute/new-azdisk).

```azurepowershell-interactive
$dataDisk = New-AzDisk `
    -ResourceGroupName "myResourceGroupDisk" `
    -DiskName "myDataDisk" `
    -Disk $diskConfig
```

Obtenga la máquina virtual que desea agregar al disco de datos con el comando [Get-AzVM](/powershell/module/az.compute/get-azvm).

```azurepowershell-interactive
$vm = Get-AzVM -ResourceGroupName "myResourceGroupDisk" -Name "myVM"
```

Agregue el disco de datos a la configuración de máquina virtual con el comando [Add-AzVMDataDisk](/powershell/module/az.compute/add-azvmdatadisk).

```azurepowershell-interactive
$vm = Add-AzVMDataDisk `
    -VM $vm `
    -Name "myDataDisk" `
    -CreateOption Attach `
    -ManagedDiskId $dataDisk.Id `
    -Lun 1
```

Actualice la máquina virtual con el comando [Update-AzVM](/powershell/module/az.compute/add-azvmdatadisk).

```azurepowershell-interactive
Update-AzVM -ResourceGroupName "myResourceGroupDisk" -VM $vm
```

## <a name="prepare-data-disks"></a>Preparación de los discos de datos

Después de que se ha conectado un disco a la máquina virtual, es necesario configurar el sistema operativo para usar el disco. En el ejemplo siguiente se muestra cómo configurar manualmente el primer disco que se agrega a la máquina virtual. Este proceso también se puede automatizar utilizando la [extensión de script personalizado](./tutorial-automate-vm-deployment.md).

### <a name="manual-configuration"></a>Configuración manual

Crear una conexión RDP con la máquina virtual. Abra PowerShell y ejecute este script.

```azurepowershell
Get-Disk | Where partitionstyle -eq 'raw' |
    Initialize-Disk -PartitionStyle MBR -PassThru |
    New-Partition -AssignDriveLetter -UseMaximumSize |
    Format-Volume -FileSystem NTFS -NewFileSystemLabel "myDataDisk" -Confirm:$false
```

## <a name="verify-the-data-disk"></a>Comprobación de los datos

Para comprobar que el disco de datos está conectado, consulte `StorageProfile` de los discos `DataDisks` conectados.

```azurepowershell-interactive
$vm.StorageProfile.DataDisks
```

El resultado debe parecerse a este ejemplo:

```
Name            : myDataDisk
DiskSizeGB      : 128
Lun             : 1
Caching         : None
CreateOption    : Attach
SourceImage     :
VirtualHardDisk :
```


## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido sobre temas relacionados con los discos de máquina virtual; por ejemplo:

> [!div class="checklist"]
> * Discos del SO y temporales.
> * Discos de datos.
> * Discos Estándar y Premium.
> * Rendimiento de disco.
> * Conectar y preparar los discos de datos

Siga con el siguiente tutorial para aprender sobre la automatización de la configuración de la máquina virtual.

> [!div class="nextstepaction"]
> [Automatización de la configuración de máquinas virtuales](./tutorial-automate-vm-deployment.md)
