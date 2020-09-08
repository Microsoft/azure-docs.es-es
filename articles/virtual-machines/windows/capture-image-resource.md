---
title: Creación de una imagen administrada en Azure
description: Cree una imagen administrada de un VHD o de una VM generalizada en Azure. Se pueden utilizar imágenes para crear varias VM que usan discos administrados.
author: cynthn
ms.service: virtual-machines-windows
ms.subservice: imaging
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 09/27/2018
ms.author: cynthn
ms.custom: legacy
ms.openlocfilehash: 751fa9f9fe2ba17a982b71a6332be302804f0dcc
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89047299"
---
# <a name="create-a-managed-image-of-a-generalized-vm-in-azure"></a>Captura de una imagen administrada de una máquina virtual generalizada en Azure

Se puede crear un recurso de imagen administrado a partir de una máquina virtual (VM) generalizada que se almacena como un disco administrado o como un disco no administrado en una cuenta de almacenamiento. A partir de ese momento, la imagen se puede utilizar para crear varias máquinas virtuales. Para obtener información sobre cómo se facturan las imágenes administradas, consulte [Precios de Managed Disks](https://azure.microsoft.com/pricing/details/managed-disks/). 

Una sola imagen administrada admite hasta 20 implementaciones simultáneas. Cuando se intentan crear más de 20 máquinas virtuales simultáneamente, desde una misma imagen administrada, se pueden producir tiempos de espera de aprovisionamiento debido a las limitaciones de rendimiento de almacenamiento de un solo VHD. Para crear más de 20 máquinas virtuales simultáneamente, use una imagen de [Instancias de Shared Image Gallery](shared-image-galleries.md) configurada con una réplica por cada 20 implementaciones simultáneas de máquina virtual.

## <a name="generalize-the-windows-vm-using-sysprep"></a>Generalización de VM con Windows mediante Sysprep

Sysprep elimina toda la información de seguridad y de la cuenta personal y luego prepara la máquina para usarse como imagen. Para más información acerca de Sysprep, consulte la [Introducción a Sysprep](/windows-hardware/manufacture/desktop/sysprep--system-preparation--overview).

Asegúrese de que los roles de servidor que se ejecutan en la máquina sean compatibles con Sysprep. Para más información, consulte [Compatibilidad de Sysprep con roles de servidor](/windows-hardware/manufacture/desktop/sysprep-support-for-server-roles) y [Escenarios no admitidos](/windows-hardware/manufacture/desktop/sysprep--system-preparation--overview#unsupported-scenarios). 

> [!IMPORTANT]
> Una vez que se ha ejecutado sysprep en una máquina virtual, se considera *generalizada* y no se puede reiniciar. El proceso de generalización de una máquina virtual no es reversible. Si necesita mantener el funcionamiento original de la máquina virtual, debe crear una [copia de la máquina virtual](create-vm-specialized.md#option-3-copy-an-existing-azure-vm) y generalizar la copia. 
>
>Sysprep requiere que las unidades se descifren por completo. Si ha habilitado el cifrado en la VM, deshabilite el cifrado antes de ejecutar Sysprep.
>
> Si tiene pensado ejecutar Sysprep antes de cargar el disco duro virtual (VHD) en Azure por primera vez, asegúrese de que tiene [preparada la máquina virtual](prepare-for-upload-vhd-image.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).  
> 
> 

Para generalizar la máquina virtual de Windows, siga estos pasos:

1. Inicie sesión en la máquina virtual Windows.
   
2. Abra una ventana de símbolo del sistema como administrador. 

3. Elimine el directorio de Panther (C:\Windows\Panther). Luego, sustitúyalo por %windir%\system32\sysprep, y, después, ejecute `sysprep.exe`.
   
4. En **Herramienta de preparación del sistema**, seleccione **Iniciar la Configuración rápida (OOBE)** y active la casilla **Generalizar**.
   
5. En **Opciones de apagado**, seleccione **Apagar**.
   
6. Seleccione **Aceptar**.
   
    ![Iniciar Sysprep](./media/upload-generalized-managed/sysprepgeneral.png)

6. Cuando Sysprep finaliza, apaga la máquina virtual. No reinicie la VM.

> [!TIP]
> **Opcional** Use [DISM](/windows-hardware/manufacture/desktop/dism-optimize-image-command-line-options) para optimizar la imagen y reducir el tiempo de arranque de la máquina virtual.
>
> Para optimizar la imagen, monte el disco duro virtual; para ello, haga doble clic en él en el Explorador de Windows y, a continuación, ejecute DISM con el parámetro `/optimize-image`.
>
> ```cmd
> DISM /image:D:\ /optimize-image /boot
> ```
> D: es la ruta de acceso del VHD montado.
>
> La ejecución de `DISM /optimize-image` debe ser la última modificación que realice en el disco duro virtual. Si realiza cambios en el disco duro virtual antes de la implementación, tendrá que volver a ejecutar `DISM /optimize-image`.

## <a name="create-a-managed-image-in-the-portal"></a>Creación de una imagen administrada en el portal 

1. Vaya a [Azure Portal](https://portal.azure.com) para administrar la imagen de la máquina virtual. Busque y seleccione **Máquinas virtuales**.

2. Seleccione la máquina virtual en la lista.

3. En la página **Máquina virtual** de la máquina virtual, en el menú superior, seleccione **Capturar**.

   Aparece la página **Crear imagen**.

4. En **Nombre**, acepte el nombre con que se rellena el espacio previamente o escriba un nombre que le gustaría que se usara para la imagen.

5. En **Grupo de recursos**, seleccione **Crear nuevo** y escriba un nombre, o seleccione en la lista desplegable el grupo de recursos que desea utilizar.

6. Si desea eliminar la máquina virtual de origen tras haberse creado la imagen, seleccione **Eliminar automáticamente esta máquina virtual después de crear la imagen**.

7. Si desea tener la posibilidad de usar la imagen en cualquier [zona de disponibilidad](../../availability-zones/az-overview.md), seleccione **Activar** para **Resistencia de zona**.

8. Seleccione **Crear** para crear la imagen.

Después de crear la imagen, esta aparecerá como un recurso **Imagen** en la lista de recursos del grupo de recursos.



## <a name="create-an-image-of-a-vm-using-powershell"></a>Creación de una imagen de una VM mediante PowerShell

 

Crear una imagen directamente desde la VM garantiza que la imagen incluya todos los discos asociados a la VM, incluido el disco del SO y todos los discos de datos. En este ejemplo se muestra cómo crear una imagen administrada a partir de una máquina virtual que utiliza discos administrados.

Antes de comenzar, asegúrese de que tiene la última versión del módulo de Azure PowerShell. Para buscar la versión, ejecute `Get-Module -ListAvailable Az` en PowerShell. Si tiene que actualizar, consulte [Install Azure PowerShell on Windows with PowerShellGet](/powershell/azure/install-az-ps) (Instalación de Azure PowerShell en Windows con PowerShellGet). Si PowerShell se ejecuta localmente, ejecute `Connect-AzAccount` para crear una conexión con Azure.


> [!NOTE]
> Si desea almacenar la imagen en un almacenamiento con redundancia de zona, debe crearla en una región que admita [zonas de disponibilidad](../../availability-zones/az-overview.md) e incluir el parámetro `-ZoneResilient` en la configuración de la imagen (comando `New-AzImageConfig`).

Para crear una imagen de VM, siga estos pasos:

1. Cree algunas variables.

    ```azurepowershell-interactive
    $vmName = "myVM"
    $rgName = "myResourceGroup"
    $location = "EastUS"
    $imageName = "myImage"
    ```
2. Asegúrese de que la VM se ha desasignado.

    ```azurepowershell-interactive
    Stop-AzVM -ResourceGroupName $rgName -Name $vmName -Force
    ```
    
3. Establezca el estado de la máquina virtual en **Generalizado**. 
   
    ```azurepowershell-interactive
    Set-AzVm -ResourceGroupName $rgName -Name $vmName -Generalized
    ```
    
4. Obtenga la máquina virtual. 

    ```azurepowershell-interactive
    $vm = Get-AzVM -Name $vmName -ResourceGroupName $rgName
    ```

5. Cree la configuración de la imagen.

    ```azurepowershell-interactive
    $image = New-AzImageConfig -Location $location -SourceVirtualMachineId $vm.Id 
    ```
6. Cree la imagen.

    ```azurepowershell-interactive
    New-AzImage -Image $image -ImageName $imageName -ResourceGroupName $rgName
    ``` 

## <a name="create-an-image-from-a-managed-disk-using-powershell"></a>Creación de una imagen a partir de un disco administrado mediante PowerShell

Si desea crear una imagen de solo el disco del SO, especifique el identificador del disco administrado como disco del SO:

    
1. Cree algunas variables. 

    ```azurepowershell-interactive
    $vmName = "myVM"
    $rgName = "myResourceGroup"
    $location = "EastUS"
    $imageName = "myImage"
    ```

2. Obtenga la máquina virtual.

   ```azurepowershell-interactive
   $vm = Get-AzVm -Name $vmName -ResourceGroupName $rgName
   ```

3. Obtenga el id. del disco administrado.

    ```azurepowershell-interactive
    $diskID = $vm.StorageProfile.OsDisk.ManagedDisk.Id
    ```
   
3. Cree la configuración de la imagen.

    ```azurepowershell-interactive
    $imageConfig = New-AzImageConfig -Location $location
    $imageConfig = Set-AzImageOsDisk -Image $imageConfig -OsState Generalized -OsType Windows -ManagedDiskId $diskID
    ```
    
4. Cree la imagen.

    ```azurepowershell-interactive
    New-AzImage -ImageName $imageName -ResourceGroupName $rgName -Image $imageConfig
    ``` 


## <a name="create-an-image-from-a-snapshot-using-powershell"></a>Creación de una imagen a partir de una instantánea mediante PowerShell

Puede crear una imagen administrada a partir de una instantánea de una VM generalizada siguiendo estos pasos:

    
1. Cree algunas variables. 

    ```azurepowershell-interactive
    $rgName = "myResourceGroup"
    $location = "EastUS"
    $snapshotName = "mySnapshot"
    $imageName = "myImage"
    ```

2. Obtenga la instantánea.

   ```azurepowershell-interactive
   $snapshot = Get-AzSnapshot -ResourceGroupName $rgName -SnapshotName $snapshotName
   ```
   
3. Cree la configuración de la imagen.

    ```azurepowershell-interactive
    $imageConfig = New-AzImageConfig -Location $location
    $imageConfig = Set-AzImageOsDisk -Image $imageConfig -OsState Generalized -OsType Windows -SnapshotId $snapshot.Id
    ```
4. Cree la imagen.

    ```azurepowershell-interactive
    New-AzImage -ImageName $imageName -ResourceGroupName $rgName -Image $imageConfig
    ``` 


## <a name="create-an-image-from-a-vm-that-uses-a-storage-account"></a>Creación de una imagen a partir de una máquina virtual que usa una cuenta de almacenamiento

Para crear una imagen administrada a partir de una máquina virtual que no usa discos administrados, se necesita el identificador URI del disco duro virtual del sistema operativo de la cuenta de almacenamiento en el siguiente formato: https://*mystorageaccount*.blob.core.windows.net/*vhdcontainer*/*vhdfilename.vhd*. En este ejemplo, el VHD está en *mystorageaccount*, en un contenedor denominado *vhdcontainer*, y el nombre de archivo de VHD es *vhdfilename.vhd*.


1.  Cree algunas variables.

    ```azurepowershell-interactive
    $vmName = "myVM"
    $rgName = "myResourceGroup"
    $location = "EastUS"
    $imageName = "myImage"
    $osVhdUri = "https://mystorageaccount.blob.core.windows.net/vhdcontainer/vhdfilename.vhd"
    ```
2. Detenga/desasigne la máquina virtual.

    ```azurepowershell-interactive
    Stop-AzVM -ResourceGroupName $rgName -Name $vmName -Force
    ```
    
3. Marque la VM como generalizada.

    ```azurepowershell-interactive
    Set-AzVm -ResourceGroupName $rgName -Name $vmName -Generalized  
    ```
4.  Cree la imagen con el VHD del SO generalizado.

    ```azurepowershell-interactive
    $imageConfig = New-AzImageConfig -Location $location
    $imageConfig = Set-AzImageOsDisk -Image $imageConfig -OsType Windows -OsState Generalized -BlobUri $osVhdUri
    $image = New-AzImage -ImageName $imageName -ResourceGroupName $rgName -Image $imageConfig
    ```

    
## <a name="next-steps"></a>Pasos siguientes
- [Creación de una máquina virtual a partir de una imagen administrada](create-vm-generalized-managed.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)    
