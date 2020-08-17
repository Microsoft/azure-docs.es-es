---
title: Use a Windows troubleshooting VM in the Azure portal (Uso de una máquina virtual Windows de solución de problemas en Azure Portal) | Microsoft Docs
description: Aprenda a solucionar problemas de la máquina virtual Windows en Azure mediante la conexión del disco del sistema operativo a una máquina virtual de recuperación mediante Azure Portal.
services: virtual-machines-windows
documentationCenter: ''
author: genlin
manager: dcscontentpm
editor: ''
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 08/19/2018
ms.author: genli
ms.openlocfilehash: 9b51205fe67bfe5be46491b0238e987fc14f6737
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87074358"
---
# <a name="troubleshoot-a-windows-vm-by-attaching-the-os-disk-to-a-recovery-vm-using-the-azure-portal"></a>Solución de problemas de una máquina virtual Windows mediante la conexión del disco del sistema operativo a una máquina virtual de recuperación mediante Azure Portal
Si la máquina virtual Windows de Azure se encuentra un error de disco o de arranque, deberá realizar los pasos para solucionar problemas en el propio disco duro virtual. Un ejemplo habitual sería una actualización de aplicación con error que impide que la máquina virtual se pueda arrancar correctamente. En este artículo se detalla cómo utilizar Azure Portal para conectar el disco duro virtual a otra máquina virtual Windows para solucionar los errores y, posteriormente, volver a crear la máquina virtual original. 

## <a name="recovery-process-overview"></a>Introducción al proceso de recuperación
El proceso de solución de problemas es el siguiente:

1. Detenga la máquina virtual afectada.
1. Cree una instantánea del disco del sistema operativo de la máquina virtual.
1. Cree un disco duro virtual a partir de la instantánea.
1. Conecte y monte el disco duro virtual en otra máquina virtual Windows con el fin de solucionar problemas.
1. Conéctese a la máquina virtual de solución de problemas. Edite los archivos o ejecute cualquier herramienta necesaria para solucionar los problemas del disco duro virtual original.
1. Desmonte y desconecte el disco duro virtual de la máquina virtual de solución de problemas.
1. Intercambie el disco del sistema operativo de la máquina virtual.

> [!NOTE]
> Este artículo no se aplica a la máquina virtual con el disco no administrado.

## <a name="take-a-snapshot-of-the-os-disk"></a>Realización de la instantánea del disco del sistema operativo
Una instantánea es una copia completa de solo lectura de un disco duro virtual (VHD). Se recomienda apagar limpiamente la máquina virtual antes de realizar una instantánea para así limpiar cualquier proceso que esté en curso. Para realizar una instantánea de un disco del sistema operativo, siga estos pasos:

1. Vaya al [Portal de Azure](https://portal.azure.com). Seleccione **Máquinas virtuales** en la barra lateral y luego la máquina virtual que tiene el problema.
1. En el panel izquierdo, seleccione **Discos** y luego el nombre del disco del sistema operativo.
    ![Imagen sobre el nombre del disco del sistema operativo](./media/troubleshoot-recovery-disks-portal-windows/select-osdisk.png)
1. En la página **Información general** del disco del sistema operativo, seleccione **Crear instantánea**.
1. Cree una instantánea en la misma ubicación que el disco del sistema operativo.

## <a name="create-a-disk-from-the-snapshot"></a>Creación de un disco a partir de la instantánea
Para crear un disco a partir de la instantánea, siga estos pasos:

1. Seleccione **Cloud Shell** en Azure Portal.

    ![Imagen sobre Cloud Shell abierto](./media/troubleshoot-recovery-disks-portal-windows/cloud-shell.png)
1. Ejecute los siguientes comandos de PowerShell para crear un disco administrado a partir de la instantánea. Debe reemplazar estos nombres de ejemplo por los nombres adecuados.

    ```powershell
    #Provide the name of your resource group
    $resourceGroupName ='myResourceGroup'
    
    #Provide the name of the snapshot that will be used to create Managed Disks
    $snapshotName = 'mySnapshot' 
    
    #Provide the name of theManaged Disk
    $diskName = 'newOSDisk'
    
    #Provide the size of the disks in GB. It should be greater than the VHD file size. In this sample, the size of the snapshot is 127 GB. So we set the disk size to 128 GB.
    $diskSize = '128'
    
    #Provide the storage type for Managed Disk.  Premium_LRS or Standard_LRS.
    $storageType = 'Standard_LRS'
    
    #Provide the Azure region (e.g. westus) where Managed Disks will be located.
    #This location should be same as the snapshot location
    #Get all the Azure location using command below:
    #Get-AzLocation
    $location = 'westus'
    
    $snapshot = Get-AzSnapshot -ResourceGroupName $resourceGroupName -SnapshotName $snapshotName 
     
    $diskConfig = New-AzDiskConfig -AccountType $storageType -Location $location -CreateOption Copy -SourceResourceId $snapshot.Id
     
    New-AzDisk -Disk $diskConfig -ResourceGroupName $resourceGroupName -DiskName $diskName
    ```
3. Si los comandos se ejecutan correctamente, se verá el nuevo disco en el grupo de recursos proporcionado.

## <a name="attach-the-disk-to-another-vm"></a>Conexión del disco a otra máquina virtual
Para los pasos siguientes, se usa otra máquina virtual con el fin de solucionar problemas. Después de conectar el disco a la máquina virtual de solución de problemas, puede examinar y modificar el contenido del disco. Este proceso permite corregir los errores de configuración o revisar otros archivos de registro de la aplicación o el sistema. Para conectar el disco a otra máquina virtual, siga estos pasos:

1. Seleccione el grupo de recursos desde el portal y, a continuación, seleccione la máquina virtual de solución de problemas. Seleccione **Discos**, elija **Editar** y haga clic en **Agregar disco de datos**:

    ![Conexión del disco existente en el portal](./media/troubleshoot-recovery-disks-portal-windows/attach-existing-disk.png)

2. En la lista **Discos de datos**, seleccione el disco del sistema operativo de la máquina virtual que identificó. Si no ve el disco del sistema operativo, asegúrese de que la máquina virtual de solución de problemas y el disco del sistema operativo se encuentran en la misma región (ubicación). 
3. Seleccione **Guardar** para aplicar los cambios.

## <a name="mount-the-attached-data-disk-to-the-vm"></a>Montaje del disco de datos conectado en la máquina virtual

1. Abra una conexión de Escritorio remoto a la máquina virtual de solución de problemas. 
2. En esta máquina virtual, abra **Administrador del servidor** y, a continuación, seleccione **Servicios de archivos y almacenamiento**. 

    ![Seleccione Servicios de archivos y almacenamiento en Administrador del servidor.](./media/troubleshoot-recovery-disks-portal-windows/server-manager-select-storage.png)

3. El disco de datos se detecta y conecta automáticamente. Para ver una lista de los discos conectados, seleccione **Discos**. Puede seleccionar el disco de datos para ver información de volumen, incluida la letra de unidad. En el ejemplo siguiente se muestra el disco de datos conectado a una máquina virtual y con **F:** :

    ![Disco conectado e información de volumen en Administrador del servidor](./media/troubleshoot-recovery-disks-portal-windows/server-manager-disk-attached.png)

## <a name="fix-issues-on-original-virtual-hard-disk"></a>Solución de problemas en el disco duro virtual original
Con el disco duro virtual existente montado, ahora puede realizar todos los pasos de mantenimiento y solución de problemas según sea necesario. Una vez que se han resuelto los problemas, continúe con los pasos siguientes.

## <a name="unmount-and-detach-original-virtual-hard-disk"></a>Desmontaje y desconexión del disco duro virtual original
Una vez resueltos los errores, desconecte el disco duro virtual existente de la máquina virtual de solución de problemas. No se podrá usar el disco duro virtual en ninguna otra máquina virtual hasta que se libere la concesión que conecta el disco duro virtual a la máquina virtual de solución de problemas.

1. En la sesión RDP de la máquina virtual, abra **Administrador del servidor** y, a continuación, seleccione **Servicios de archivos y almacenamiento**:

    ![Seleccione Servicios de archivos y almacenamiento en Administrador del servidor.](./media/troubleshoot-recovery-disks-portal-windows/server-manager-select-storage.png)

2. Seleccione **Discos** y, a continuación, seleccione el disco de datos. Haga clic con el botón derecho en el disco de datos y seleccione **Desconectar**:

    ![Establezca el disco de datos como sin conexión en Administrador del servidor.](./media/troubleshoot-recovery-disks-portal-windows/server-manager-set-disk-offline.png)

3. Ahora, desconecte el disco duro virtual de la máquina virtual. Seleccione la máquina virtual en Azure Portal y haga clic en **Discos**. 
4. Seleccione **Editar**, elija el disco del sistema operativo que ha conectado y haga clic en **Desasociar**:

    ![Desconecte el disco duro virtual existente](./media/troubleshoot-recovery-disks-portal-windows/detach-disk.png)

    Espere hasta que la máquina virtual haya desconectado correctamente el disco de datos antes de continuar.

## <a name="swap-the-os-disk-for-the-vm"></a>Intercambio del disco del sistema operativo de la máquina virtual

Azure Portal ahora permite cambiar el disco del sistema operativo de la máquina virtual. Para ello, siga estos pasos.

1. Vaya al [Portal de Azure](https://portal.azure.com). Seleccione **Máquinas virtuales** en la barra lateral y luego la máquina virtual que tiene el problema.
1. En el panel izquierdo, seleccione **Discos** y luego **Intercambiar disco del sistema operativo**.
        ![Imagen sobre el intercambio del disco del sistema operativo en Azure Portal](./media/troubleshoot-recovery-disks-portal-windows/swap-os-ui.png)

1. Seleccione el nuevo disco que ha reparado y escriba el nombre de la máquina virtual para confirmar el cambio. Si no ve el disco en la lista, espere entre 10 y 15 minutos después de desconectar el disco de la máquina virtual de solución de problemas. Asegúrese también de que el disco está en la misma ubicación que la máquina virtual.
1. Seleccione Aceptar.

## <a name="next-steps"></a>Pasos siguientes
Si tiene problemas para conectarse a la máquina virtual, consulte [Solución de problemas de conexiones RDP a una máquina virtual de Azure](troubleshoot-rdp-connection.md). Para problemas con el acceso a aplicaciones que se ejecutan en su máquina virtual, consulte [Solucionar problemas de conectividad de aplicaciones en una máquina virtual Windows](troubleshoot-app-connection.md).

Para más información sobre el uso de Resource Manager, consulte [Información general de Azure Resource Manager](../../azure-resource-manager/management/overview.md).


