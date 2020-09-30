---
title: Expansión de la unidad de sistema operativo de una máquina virtual de Windows en Azure
description: Expanda el tamaño de la unidad de sistema operativo de una máquina virtual con Azure PowerShell en el modelo de implementación de Resource Manager.
services: virtual-machines-windows
documentationcenter: ''
author: kirpasingh
manager: roshar
editor: ''
tags: azure-resource-manager
ms.assetid: d9edfd9f-482f-4c0b-956c-0d2c2c30026c
ms.service: virtual-machines-windows
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 09/02/2020
ms.author: kirpas
ms.subservice: disks
ms.openlocfilehash: 3f1c7fb08c3dcaa078de8f1ee0b90409289cfb43
ms.sourcegitcommit: 4feb198becb7a6ff9e6b42be9185e07539022f17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89469258"
---
# <a name="how-to-expand-the-os-drive-of-a-virtual-machine"></a>Cómo ampliar la unidad de sistema operativo de una máquina virtual

Cuando se crea una nueva máquina virtual (VM) en un grupo de recursos mediante la implementación de una imagen de [Azure Marketplace](https://azure.microsoft.com/marketplace/), la unidad del sistema operativo predeterminada suele tener 127 GB (algunas imágenes son más pequeñas de manera predeterminada). Aunque es posible agregar discos de datos a la máquina virtual (el número depende de la SKU que elija) y se recomienda instalar aplicaciones y cargas de trabajo intensivas de CPU en estos discos anexos, a menudo los clientes necesitan expandir la unidad del sistema operativo para admitir escenarios concretos:

- Para admitir aplicaciones heredadas que instalan componentes en la unidad del sistema operativo.
- Para migrar una VM o un equipo físico del entorno local con una unidad del sistema operativo más grande.

> [!IMPORTANT]
> Para cambiar el tamaño del disco del sistema operativo de una máquina virtual de Azure requiere que se desasigne la máquina virtual.
>
> Después de expandir los discos, necesita [expandir el volumen dentro del sistema operativo](#expand-the-volume-within-the-os) para aprovechar el disco más grande.
> 

## <a name="resize-a-managed-disk-in-the-azure-portal"></a>Cambio de tamaño de un disco administrado en Azure Portal

1. En [Azure Portal](https://portal.azure.com), navegue a la máquina virtual para la que quiere expandir el disco. Seleccione **Detener** para detener o desasignar la máquina virtual.
2. Cuando la máquina virtual esté detenida, en **Configuración** del menú de la izquierda, seleccione **Discos**.

    :::image type="content" source="./media/expand-os-disk/select-disks.png" alt-text="Captura de pantalla que muestra la opción Discos seleccionada en la sección Configuración del menú.":::

 
3. En **Nombre del disco**, seleccione el disco cuyo tamaño quiere cambiar.

    :::image type="content" source="./media/expand-os-disk/disk-name.png" alt-text="Captura de pantalla que muestra la opción Discos seleccionada en la sección Configuración del menú.":::

4. En la sección **Configuración** del menú de la izquierda, seleccione **Configuración**.

    :::image type="content" source="./media/expand-os-disk/configuration.png" alt-text="Captura de pantalla que muestra la opción Discos seleccionada en la sección Configuración del menú.":::

5. En **Tamaño (GiB)** , seleccione el tamaño de disco que quiera.
   
   > [!WARNING]
   > El nuevo tamaño debe ser mayor que el tamaño de disco existente. El máximo permitido es 2048 GB para los discos del sistema operativo. (El blob de VHD se puede expandir más, pero el sistema operativo solo usa los primeros 2048 GB de espacio).
   > 

    :::image type="content" source="./media/expand-os-disk/size.png" alt-text="Captura de pantalla que muestra la opción Discos seleccionada en la sección Configuración del menú.":::

6. Seleccione **Guardar**.

    :::image type="content" source="./media/expand-os-disk/save.png" alt-text="Captura de pantalla que muestra la opción Discos seleccionada en la sección Configuración del menú.":::


## <a name="resize-a-managed-disk-by-using-powershell"></a>Cambio de tamaño de un disco administrado mediante PowerShell

Abra la ventana de PowerShell o PowerShell ISE en el modo administrativo y siga estos pasos:

1. Inicie sesión en su cuenta de Microsoft Azure en el modo de administración de recursos y seleccione su suscripción:
   
    ```powershell
    Connect-AzAccount
    Select-AzSubscription –SubscriptionName 'my-subscription-name'
    ```

2. Establezca el nombre del grupo de recursos y el nombre de la máquina virtual:
   
    ```powershell
    $rgName = 'my-resource-group-name'
    $vmName = 'my-vm-name'
    ```

3. Obtenga una referencia a la máquina virtual:
   
    ```powershell
    $vm = Get-AzVM -ResourceGroupName $rgName -Name $vmName
    ```

4. Detenga la máquina virtual antes de cambiar el tamaño del disco:
   
    ```powershell
    Stop-AzVM -ResourceGroupName $rgName -Name $vmName
    ```

5. Obtenga una referencia al disco del sistema operativo administrado. Configure el tamaño del disco del sistema operativo en el valor deseado y actualice el disco:
   
    ```powershell
    $disk= Get-AzDisk -ResourceGroupName $rgName -DiskName $vm.StorageProfile.OsDisk.Name
    $disk.DiskSizeGB = 1023
    Update-AzDisk -ResourceGroupName $rgName -Disk $disk -DiskName $disk.Name
    ```   
    > [!WARNING]
    > El nuevo tamaño debe ser mayor que el tamaño de disco existente. El máximo permitido es 2048 GB para los discos del sistema operativo. (El blob de VHD se puede expandir más, pero el sistema operativo solo usa los primeros 2048 GB de espacio).
    > 
         
6. La actualización de la máquina virtual puede tardar unos segundos. Cuando el comando acabe de ejecutarse, reinicie la máquina virtual:
   
    ```powershell
    Start-AzVM -ResourceGroupName $rgName -Name $vmName
    ```

Eso es todo. Ahora RDP en la máquina virtual, abra Administración de equipos (o Administración de discos) y expanda la unidad utilizando el espacio recién asignado.

## <a name="resize-an-unmanaged-disk-by-using-powershell"></a>Cambio de tamaño de un disco no administrado mediante PowerShell

Abra la ventana de PowerShell o PowerShell ISE en el modo administrativo y siga estos pasos:

1. Inicie sesión en su cuenta de Microsoft Azure en el modo de administración de recursos y seleccione su suscripción:
   
    ```powershell
    Connect-AzAccount
    Select-AzSubscription –SubscriptionName 'my-subscription-name'
    ```

2. Establezca el nombre del grupo de recursos y de la máquina virtual:
   
    ```powershell
    $rgName = 'my-resource-group-name'
    $vmName = 'my-vm-name'
    ```

3. Obtenga una referencia a la máquina virtual:
   
    ```powershell
    $vm = Get-AzVM -ResourceGroupName $rgName -Name $vmName
    ```

4. Detenga la máquina virtual antes de cambiar el tamaño del disco:
   
    ```powershell
    Stop-AzVM -ResourceGroupName $rgName -Name $vmName
    ```

5. Configure el tamaño del disco del sistema operativo no administrado en el valor deseado y actualice la máquina virtual:
   
    ```powershell
    $vm.StorageProfile.OSDisk.DiskSizeGB = 1023
    Update-AzVM -ResourceGroupName $rgName -VM $vm
    ```
   
    > [!WARNING]
    > El nuevo tamaño debe ser mayor que el tamaño de disco existente. El máximo permitido es 2048 GB para los discos del sistema operativo. (El blob de VHD se puede expandir más, pero el sistema operativo solo podrá trabajar con los primeros 2048 GB).
    > 
    > 
   
6. La actualización de la máquina virtual puede tardar unos segundos. Cuando el comando acabe de ejecutarse, reinicie la máquina virtual:
   
    ```powershell
    Start-AzVM -ResourceGroupName $rgName -Name $vmName
    ```


## <a name="scripts-for-os-disk"></a>Scripts para el disco del sistema operativo

A continuación se muestra el script completo para su referencia tanto para discos administrados como no administrados:


**Discos administrados**

```powershell
Connect-AzAccount
Select-AzSubscription -SubscriptionName 'my-subscription-name'
$rgName = 'my-resource-group-name'
$vmName = 'my-vm-name'
$vm = Get-AzVM -ResourceGroupName $rgName -Name $vmName
Stop-AzVM -ResourceGroupName $rgName -Name $vmName
$disk= Get-AzDisk -ResourceGroupName $rgName -DiskName $vm.StorageProfile.OsDisk.Name
$disk.DiskSizeGB = 1023
Update-AzDisk -ResourceGroupName $rgName -Disk $disk -DiskName $disk.Name
Start-AzVM -ResourceGroupName $rgName -Name $vmName
```

**Discos no administrados**

```powershell
Connect-AzAccount
Select-AzSubscription -SubscriptionName 'my-subscription-name'
$rgName = 'my-resource-group-name'
$vmName = 'my-vm-name'
$vm = Get-AzVM -ResourceGroupName $rgName -Name $vmName
Stop-AzVM -ResourceGroupName $rgName -Name $vmName
$vm.StorageProfile.OSDisk.DiskSizeGB = 1023
Update-AzVM -ResourceGroupName $rgName -VM $vm
Start-AzVM -ResourceGroupName $rgName -Name $vmName
```

## <a name="resizing-data-disks"></a>Cambio de tamaño de los discos de datos

Este artículo se centra principalmente en expandir el disco del sistema operativo de la máquina virtual, pero el script también puede utilizarse para expandir los discos de datos asociados a la máquina virtual. Por ejemplo, para expandir el primer disco de datos conectado a la máquina virtual, reemplace el objeto `OSDisk` de `StorageProfile` por la matriz `DataDisks` y utilice un índice numérico para obtener una referencia al primer disco de datos conectado, como se muestra a continuación:

**Disco administrado**

```powershell
$disk= Get-AzDisk -ResourceGroupName $rgName -DiskName $vm.StorageProfile.DataDisks[0].Name
$disk.DiskSizeGB = 1023
```

**Disco no administrado**

```powershell
$vm.StorageProfile.DataDisks[0].DiskSizeGB = 1023
```

Del mismo modo, puede hacer referencia a otros discos de datos conectados a la máquina virtual, ya sea mediante un índice, como se muestra arriba, o con la propiedad **Name** del disco:


**Disco administrado**

```powershell
(Get-AzDisk -ResourceGroupName $rgName -DiskName ($vm.StorageProfile.DataDisks | Where ({$_.Name -eq 'my-second-data-disk'})).Name).DiskSizeGB = 1023
```

**Disco no administrado**

```powershell
($vm.StorageProfile.DataDisks | Where ({$_.Name -eq 'my-second-data-disk'}).DiskSizeGB = 1023
```

## <a name="expand-the-volume-within-the-os"></a>Expansión del volumen en el sistema operativo

Una vez expandido el disco para la máquina virtual, deberá entrar en el sistema operativo y expandir el volumen para abarcar el nuevo espacio. Existen varios métodos para expandir una partición. En esta sección se trata la conexión de la máquina virtual mediante una conexión RDP para expandir la partición mediante **DiskPart**.

1. Abra una conexión RDP a la máquina virtual.

2. Abra un símbolo del sistema y escriba **diskpart**.

3. En el símbolo del sistema **DISKPART**, escriba `list volume`. Tome nota del volumen que desea extender.

4. En el símbolo del sistema **DISKPART**, escriba `select volume <volumenumber>`. Este comando selecciona el volumen *númeroDeVolumen* que desea extender en el espacio vacío contiguo del mismo disco.

5. En el símbolo del sistema **DISKPART**, escriba `extend [size=<size>]`. Este comando extiende el volumen seleccionado por *tamaño* en megabytes (MB).


## <a name="next-steps"></a>Pasos siguientes

También puede asociar discos mediante [Azure Portal](attach-managed-disk-portal.md).
