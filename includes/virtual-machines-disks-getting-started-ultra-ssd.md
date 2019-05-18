---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-machines
author: roygara
ms.service: virtual-machines
ms.topic: include
ms.date: 05/10/2019
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 495326c172f900dc8bcff78b0df38f2cb64ed27e
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65546541"
---
# <a name="enable-and-deploy-azure-ultra-ssds-preview"></a>Habilitar e implementar Azure SSD ultra (versión preliminar)

Unidades de Azure ultra de estado sólido (SSD) (versión preliminar) oferta de alta rendimiento, una IOPS elevada y almacenamiento en disco baja latencia coherente para las máquinas virtuales de IaaS de Azure (VM). En esta nueva oferta se proporciona un rendimiento exclusivo que se encuentra en los mismos niveles de disponibilidad que nuestras ofertas de discos existentes. Una ventaja importante de ultra SSDs es la capacidad de cambiar dinámicamente el rendimiento de la SSD, junto con las cargas de trabajo sin necesidad de reiniciar las máquinas virtuales. SSDs Ultra son adecuados para cargas de trabajo de uso intensivo de datos como SAP HANA, las bases de datos de nivel superior y las cargas de trabajo intensivo de la transacción.

Actualmente, ultra SSD está en versión preliminar y se debe [inscribir](https://aka.ms/UltraSSDPreviewSignUp) en la vista previa con el fin de tener acceso a ellos.

## <a name="determine-your-availability-zone"></a>Determinar la zona de disponibilidad

Una vez aprobada, deberá determinar en qué zona de disponibilidad que está utilizando, para poder usar ultra SSDs. Ejecute cualquiera de los comandos siguientes para determinar qué zona este de Estados Unidos 2 para implementar el disco ultra para:

PowerShell: `Get-AzComputeResourceSku | where {$_.ResourceType -eq "disks" -and $_.Name -eq "UltraSSD_LRS" }`

CLI: `az vm list-skus --resource-type disks --query “[?name==UltraSSD_LRS]”`

La respuesta será similar a la forma siguiente, donde X es la zona que se utilizará para la implementación en East US 2. X podría ser 1, 2 o 3.

Conservar la **zonas** valor, representa la zona de disponibilidad y que la necesitará para implementar una SSD ultra.

|ResourceType  |NOMBRE  |Ubicación  |Zonas  |Restricción  |Funcionalidad  |Valor  |
|---------|---------|---------|---------|---------|---------|---------|
|Discos     |UltraSSD_LRS         |eastus2         |X         |         |         |         |

Si no hubo respuesta del comando, el registro a la función sigue siendo pendiente, o no aprobado todavía.

Ahora que conoce la zona de implementación, siga los pasos de implementación de este artículo para obtener sus primeras máquinas virtuales implementadas con discos SSD Ultra.

## <a name="deploy-an-ultra-ssd-using-azure-resource-manager"></a>Implementar una SSD ultra mediante Azure Resource Manager

En primer lugar, determine el tamaño de máquina virtual para implementar. Como parte de esta versión preliminar, se admiten solo las familias de máquinas virtuales DsV3 y EsV3. Consulte la segunda tabla de este [blog](https://azure.microsoft.com/blog/introducing-the-new-dv3-and-ev3-vm-sizes/) para obtener más información acerca de estos tamaños de máquina virtual.

Si desea crear una máquina virtual con varios SSD ultra, consulte el ejemplo [crear una máquina virtual con varios SSD ultra](https://aka.ms/UltraSSDTemplate).

Si piensa utilizar su propia plantilla, asegúrese de que **apiVersion** para `Microsoft.Compute/virtualMachines` y `Microsoft.Compute/Disks` se establece como `2018-06-01` (o posterior).

Establezca la sku de disco en **UltraSSD_LRS**, a continuación, establezca la capacidad de disco, IOPS, zona de disponibilidad y rendimiento en MBps para crear un disco ultra.

Cuando se aprovisiona la máquina virtual, puede realizar una partición y dar formato a los discos de datos y configurarlos para las cargas de trabajo.

## <a name="deploy-an-ultra-ssd-using-cli"></a>Implementar una SSD ultra mediante la CLI

En primer lugar, determine el tamaño de máquina virtual para implementar. Como parte de esta versión preliminar, se admiten solo las familias de máquinas virtuales DsV3 y EsV3. Consulte la segunda tabla de este [blog](https://azure.microsoft.com/blog/introducing-the-new-dv3-and-ev3-vm-sizes/) para obtener más información acerca de estos tamaños de máquina virtual.

Para usar SSDs ultra, debe crear una máquina virtual que es capaz de usar ultra SSDs.

Reemplazar o establecer el **$vmname**, **$rgname**, **$diskname**, **$location**, **$password**, **$user** variables con sus propios valores. Establecer **$zone** al valor de la zona de disponibilidad que obtuvo en el [inicio de este artículo](#determine-your-availability-zone). A continuación, ejecute el siguiente comando CLI para crear una máquina virtual habilitada ultra:

```azurecli-interactive
az vm create --subscription $subscription -n $vmname -g $rgname --image Win2016Datacenter --ultra-ssd-enabled --zone $zone --authentication-type password --admin-password $password --admin-username $user --attach-data-disks $diskname --size Standard_D4s_v3 --location $location
```

### <a name="create-an-ultra-ssd-using-cli"></a>Creación de una SSD ultra mediante la CLI

Ahora que tiene una máquina virtual que es capaz de usar ultra SSD, puede crear y adjuntar una SSD ultra a él.

```azurecli-interactive
location="eastus2"
subscription="xxx"
rgname="ultraRG"
diskname="ssd1"
vmname="ultravm1"
zone=123

#create an Ultra SSD disk
az disk create `
--subscription $subscription `
-n $diskname `
-g $rgname `
--size-gb 4 `
--location $location `
--zone $zone `
--sku UltraSSD_LRS `
--disk-iops-read-write 1000 `
--disk-mbps-read-write 50
```

### <a name="adjust-the-performance-of-an-ultra-ssd-using-cli"></a>Ajustar el rendimiento de una SSD ultra mediante la CLI

SSDs Ultra ofrecen una funcionalidad única que permite ajustar su rendimiento, el comando siguiente muestra cómo usar esta característica:

```azurecli-interactive
az disk update `
--subscription $subscription `
--resource-group $rgname `
--name $diskName `
--set diskIopsReadWrite=80000 `
--set diskMbpsReadWrite=800
```

## <a name="deploy-an-ultra-ssd-using-powershell"></a>Implementar una SSD ultra mediante PowerShell

En primer lugar, determine el tamaño de máquina virtual para implementar. Como parte de esta versión preliminar, se admiten solo las familias de máquinas virtuales DsV3 y EsV3. Consulte la segunda tabla de este [blog](https://azure.microsoft.com/blog/introducing-the-new-dv3-and-ev3-vm-sizes/) para obtener más información acerca de estos tamaños de máquina virtual.

Para usar SSDs ultra, debe crear una máquina virtual que es capaz de usar ultra SSDs. Reemplazar o establecer el **$resourcegroup** y **$vmName** variables con sus propios valores. Establecer **$zone** al valor de la zona de disponibilidad que obtuvo en el [inicio de este artículo](#determine-your-availability-zone). A continuación, ejecute el siguiente [New-AzVm](/powershell/module/az.compute/new-azvm) comando para crear un ultra máquina virtual habilitada:

```powershell
New-AzVm `
    -ResourceGroupName $resourcegroup `
    -Name $vmName `
    -Location "eastus2" `
    -Image "Win2016Datacenter" `
    -EnableUltraSSD `
    -size "Standard_D4s_v3" `
    -zone $zone
```

### <a name="create-an-ultra-ssd-using-powershell"></a>Creación de una SSD ultra mediante PowerShell

Ahora que tiene una máquina virtual que es capaz de usar ultra SSD, puede crear y adjuntar una SSD ultra a él:

```powershell
$diskconfig = New-AzDiskConfig `
-Location 'EastUS2' `
-DiskSizeGB 8 `
-DiskIOPSReadWrite 1000 `
-DiskMBpsReadWrite 100 `
-AccountType UltraSSD_LRS `
-CreateOption Empty `
-zone $zone;

New-AzDisk `
-ResourceGroupName $resourceGroup `
-DiskName 'Disk02' `
-Disk $diskconfig;
```

### <a name="adjust-the-performance-of-an-ultra-ssd-using-powershell"></a>Ajustar el rendimiento de una SSD ultra mediante PowerShell

SSDs Ultra tienen una funcionalidad única que permite ajustar su rendimiento, el comando siguiente es un ejemplo que ajusta el rendimiento sin tener que desconectar el disco:

```powershell
$diskupdateconfig = New-AzDiskUpdateConfig -DiskMBpsReadWrite 2000
Update-AzDisk -ResourceGroupName $resourceGroup -DiskName $diskName -DiskUpdate $diskupdateconfig
```

## <a name="next-steps"></a>Pasos siguientes

Si desea probar el nuevo tipo de disco [solicitar acceso a la vista previa con esta encuesta](https://aka.ms/UltraSSDPreviewSignUp).