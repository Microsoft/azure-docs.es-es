---
title: Habilitación de los discos compartidos para Azure Managed Disks
description: Configuración de un disco administrado de Azure con discos compartidos, con el fin de poder compartirlo entre varias máquinas virtuales
author: roygara
ms.service: virtual-machines
ms.topic: how-to
ms.date: 08/21/2020
ms.author: rogarana
ms.subservice: disks
ms.custom: references_regions
ms.openlocfilehash: ad70aa1d044649a0ca61060fff9880d4ef1e34c1
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "88752051"
---
# <a name="enable-shared-disk"></a>Habilitación del disco compartido

En este artículo se describe cómo habilitar la característica de discos compartidos en Azure Managed Disks. Los discos compartidos de Azure son una nueva característica de Azure Managed Disks que permite conectar un disco administrado a varias máquinas virtuales (VM) al mismo tiempo. Si adjunta un disco administrado en varias VM, podrá implementar nuevas aplicaciones en clúster o migrar las existentes a Azure. 

Si busca información conceptual sobre los discos administrados que tienen habilitados discos compartidos, vea:

* Para Linux: [Discos compartidos de Azure](linux/disks-shared.md)

* Para Windows: [Discos compartidos de Azure](windows/disks-shared.md)

## <a name="limitations"></a>Limitaciones

[!INCLUDE [virtual-machines-disks-shared-limitations](../../includes/virtual-machines-disks-shared-limitations.md)]

## <a name="supported-operating-systems"></a>Sistemas operativos admitidos

Los discos compartidos admiten varios sistemas operativos. Consulte las secciones [Windows](windows/disks-shared.md#windows) y [Linux](linux/disks-shared.md#linux) del artículo conceptual sobre los sistemas operativos compatibles.

## <a name="disk-sizes"></a>Tamaños de disco

[!INCLUDE [virtual-machines-disks-shared-sizes](../../includes/virtual-machines-disks-shared-sizes.md)]

## <a name="deploy-shared-disks"></a>Implementación de discos compartidos

### <a name="deploy-a-premium-ssd-as-a-shared-disk"></a>Implementación de un SSD Premium como un disco compartido

Para implementar un disco administrado con la característica de disco compartido habilitada, use la nueva propiedad `maxShares` y defina un valor mayor que 1. Esto permite que el disco se pueda compartir en varias VM.

> [!IMPORTANT]
> El valor de `maxShares` solo se puede establecer o cambiar cuando se desmonta un disco de todas las VM. Consulte los [tamaños de disco](#disk-sizes) para ver los valores permitidos para `maxShares`.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
az disk create -g myResourceGroup -n mySharedDisk --size-gb 1024 -l westcentralus --sku Premium_LRS --max-shares 2
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
$dataDiskConfig = New-AzDiskConfig -Location 'WestCentralUS' -DiskSizeGB 1024 -AccountType Premium_LRS -CreateOption Empty -MaxSharesCount 2

New-AzDisk -ResourceGroupName 'myResourceGroup' -DiskName 'mySharedDisk' -Disk $dataDiskConfig
```

# <a name="resource-manager-template"></a>[Plantilla de Resource Manager](#tab/azure-resource-manager)

Antes de usar la siguiente plantilla, reemplace `[parameters('dataDiskName')]`, `[resourceGroup().location]`, `[parameters('dataDiskSizeGB')]` y `[parameters('maxShares')]` con sus propios valores.

[Plantilla de disco compartido de SSD Premium](https://aka.ms/SharedPremiumDiskARMtemplate)

---

### <a name="deploy-an-ultra-disk-as-a-shared-disk"></a>Implementación de un disco Ultra como un disco compartido

Para implementar un disco administrado con la característica de disco compartido habilitada, cambie el parámetro `maxShares` por un valor mayor que 1. Esto permite que el disco se pueda compartir en varias VM.

> [!IMPORTANT]
> El valor de `maxShares` solo se puede establecer o cambiar cuando se desmonta un disco de todas las VM. Consulte los [tamaños de disco](#disk-sizes) para ver los valores permitidos para `maxShares`.


# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

##### <a name="regional-disk-example"></a>Ejemplo de disco regional

```azurecli
#Creating an Ultra shared Disk 
az disk create -g rg1 -n clidisk --size-gb 1024 -l westus --sku UltraSSD_LRS --max-shares 5 --disk-iops-read-write 2000 --disk-mbps-read-write 200 --disk-iops-read-only 100 --disk-mbps-read-only 1

#Updating an Ultra shared Disk 
az disk update -g rg1 -n clidisk --disk-iops-read-write 3000 --disk-mbps-read-write 300 --set diskIopsReadOnly=100 --set diskMbpsReadOnly=1

#Show shared disk properties:
az disk show -g rg1 -n clidisk
```

##### <a name="zonal-disk-example"></a>Ejemplo de disco de zona

Este ejemplo es casi el mismo que el anterior, salvo que crea un disco en la zona de disponibilidad 1.

```azurecli
#Creating an Ultra shared Disk 
az disk create -g rg1 -n clidisk --size-gb 1024 -l westus --sku UltraSSD_LRS --max-shares 5 --disk-iops-read-write 2000 --disk-mbps-read-write 200 --disk-iops-read-only 100 --disk-mbps-read-only 1 --zone 1

#Updating an Ultra shared Disk 
az disk update -g rg1 -n clidisk --disk-iops-read-write 3000 --disk-mbps-read-write 300 --set diskIopsReadOnly=100 --set diskMbpsReadOnly=1

#Show shared disk properties:
az disk show -g rg1 -n clidisk
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

##### <a name="regional-disk-example"></a>Ejemplo de disco regional

```azurepowershell-interactive
$datadiskconfig = New-AzDiskConfig -Location 'WestCentralUS' -DiskSizeGB 1024 -AccountType UltraSSD_LRS -CreateOption Empty -DiskIOPSReadWrite 2000 -DiskMBpsReadWrite 200 -DiskIOPSReadOnly 100 -DiskMBpsReadOnly 1 -MaxSharesCount 5

New-AzDisk -ResourceGroupName 'myResourceGroup' -DiskName 'mySharedDisk' -Disk $datadiskconfig
```

##### <a name="zonal-disk-example"></a>Ejemplo de disco de zona

Este ejemplo es casi el mismo que el anterior, salvo que crea un disco en la zona de disponibilidad 1.

```azurepowershell-interactive
$datadiskconfig = New-AzDiskConfig -Location 'WestCentralUS' -DiskSizeGB 1024 -AccountType UltraSSD_LRS -CreateOption Empty -DiskIOPSReadWrite 2000 -DiskMBpsReadWrite 200 -DiskIOPSReadOnly 100 -DiskMBpsReadOnly 1 -MaxSharesCount 5 -Zone 1

New-AzDisk -ResourceGroupName 'myResourceGroup' -DiskName 'mySharedDisk' -Disk $datadiskconfig
```

# <a name="resource-manager-template"></a>[Plantilla de Resource Manager](#tab/azure-resource-manager)

##### <a name="regional-disk-example"></a>Ejemplo de disco regional

Antes de usar la siguiente plantilla, reemplace `[parameters('dataDiskName')]`, `[resourceGroup().location]`, `[parameters('dataDiskSizeGB')]`, `[parameters('maxShares')]`, `[parameters('diskIOPSReadWrite')]`, `[parameters('diskMBpsReadWrite')]`, `[parameters('diskIOPSReadOnly')]` y `[parameters('diskMBpsReadOnly')]` con sus propios valores.

[Plantilla de discos Ultra compartidos regionales](https://aka.ms/SharedUltraDiskARMtemplateRegional)

##### <a name="zonal-disk-example"></a>Ejemplo de disco de zona

Antes de usar la siguiente plantilla, reemplace `[parameters('dataDiskName')]`, `[resourceGroup().location]`, `[parameters('dataDiskSizeGB')]`, `[parameters('maxShares')]`, `[parameters('diskIOPSReadWrite')]`, `[parameters('diskMBpsReadWrite')]`, `[parameters('diskIOPSReadOnly')]` y `[parameters('diskMBpsReadOnly')]` con sus propios valores.

[Plantilla de discos Ultra compartidos de zona](https://aka.ms/SharedUltraDiskARMtemplateZonal)

---

## <a name="using-azure-shared-disks-with-your-vms"></a>Uso de discos compartidos de Azure con las VM

Una vez implementado un disco compartido con `maxShares>1`, puede montar el disco en una o varias VM.

> [!NOTE]
> Si va a implementar un disco Ultra, asegúrese de que cumple los requisitos necesarios. Consulte la sección [PowerShell](disks-enable-ultra-ssd.md#enable-ultra-disk-compatibility-on-an-existing-vm-1) o [CLI](disks-enable-ultra-ssd.md#enable-ultra-disk-compatibility-on-an-existing-vm) del artículo sobre el disco Ultra para más información.

```azurepowershell-interactive

$resourceGroup = "myResourceGroup"
$location = "WestCentralUS"

$vm = New-AzVm -ResourceGroupName $resourceGroup -Name "myVM" -Location $location -VirtualNetworkName "myVnet" -SubnetName "mySubnet" -SecurityGroupName "myNetworkSecurityGroup" -PublicIpAddressName "myPublicIpAddress"

$dataDisk = Get-AzDisk -ResourceGroupName $resourceGroup -DiskName "mySharedDisk"

$vm = Add-AzVMDataDisk -VM $vm -Name "mySharedDisk" -CreateOption Attach -ManagedDiskId $dataDisk.Id -Lun 0

update-AzVm -VM $vm -ResourceGroupName $resourceGroup
```

## <a name="supported-scsi-pr-commands"></a>Comandos de SCSI PR admitidos

Una vez montado el disco compartido en las VM del clúster, puede establecer cuórum y lectura/escritura en el disco mediante SCSI PR. Los siguientes comandos de PR están disponibles al usar discos compartidos de Azure:

Para interactuar con el disco, comience con la lista de acciones de reservas persistentes:

```
PR_REGISTER_KEY 

PR_REGISTER_AND_IGNORE 

PR_GET_CONFIGURATION 

PR_RESERVE 

PR_PREEMPT_RESERVATION 

PR_CLEAR_RESERVATION 

PR_RELEASE_RESERVATION 
```

Al usar PR_RESERVE, PR_PREEMPT_RESERVATION o PR_RELEASE_RESERVATION, proporcione uno de los siguientes tipos de reserva persistente:

```
PR_NONE 

PR_WRITE_EXCLUSIVE 

PR_EXCLUSIVE_ACCESS 

PR_WRITE_EXCLUSIVE_REGISTRANTS_ONLY 

PR_EXCLUSIVE_ACCESS_REGISTRANTS_ONLY 

PR_WRITE_EXCLUSIVE_ALL_REGISTRANTS 

PR_EXCLUSIVE_ACCESS_ALL_REGISTRANTS 
```

También debe proporcionar una clave de reserva persistente al usar PR_RESERVE, PR_REGISTER_AND_IGNORE, PR_REGISTER_KEY, PR_PREEMPT_RESERVATION, PR_CLEAR_RESERVATION o PR_RELEASE-RESERVATION.


## <a name="next-steps"></a>Pasos siguientes

Si prefiere usar plantillas de Azure Resource Manager para implementar el disco, están disponibles las siguientes plantillas de ejemplo:
- [SSD Premium](https://aka.ms/SharedPremiumDiskARMtemplate)
- [Discos Ultra regionales](https://aka.ms/SharedUltraDiskARMtemplateRegional)
- [Discos Ultra de zona](https://aka.ms/SharedUltraDiskARMtemplateZonal)