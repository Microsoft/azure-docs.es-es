---
title: Creación de una imagen no administrada de una máquina virtual generalizada en Azure
description: Cree una imagen no administrada de una máquina virtual de Windows generalizada a fin de utilizarla para crear varias copias de una máquina virtual en Azure.
author: cynthn
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 05/23/2017
ms.author: cynthn
ROBOTS: NOINDEX
ms.custom: storage-accounts
ms.openlocfilehash: f83e2157fcf70eff90d7444d647552ba7e34c390
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87291058"
---
# <a name="how-to-create-an-unmanaged-vm-image-from-an-azure-vm"></a>Creación de una imagen de máquina virtual no administrada a partir de una máquina virtual de Azure

En este artículo se explica cómo usar cuentas de almacenamiento. Se recomienda utilizar Managed Disks e imágenes administradas en lugar de una cuenta de almacenamiento. Para más información, vea [Captura de una imagen administrada de una máquina virtual generalizada en Azure](capture-image-resource.md).

En este artículo se muestra cómo usar Azure PowerShell para crear una imagen de una máquina virtual de Azure generalizada con una cuenta de almacenamiento. Después, puede usar la imagen para crear otra máquina virtual. Dicha imagen incluye el disco del SO y los discos de datos conectados a la máquina virtual. La imagen no incluye los recursos de red virtual, por lo que tiene que es preciso configurarlos al crear la máquina virtual nueva. 

 

## <a name="generalize-the-vm"></a>Generalización de la máquina virtual 
En esta sección se muestra cómo generalizar la máquina virtual Windows para usarla como imagen. La generalización de una máquina virtual elimina toda la información personal de la cuenta, entre otras cosas, y prepara la máquina para usarse como imagen. Para más información acerca de Sysprep, consulte [Uso de Sysprep: Introducción](/previous-versions/windows/it-pro/windows-xp/bb457073(v=technet.10)).

Asegúrese de que los roles de servidor que se ejecutan en la máquina sean compatibles con Sysprep. Para más información, consulte [Sysprep Support for Server Roles](/windows-hardware/manufacture/desktop/sysprep-support-for-server-roles)

> [!IMPORTANT]
> Si va a cargar el VHD en Azure por primera vez, asegúrese de que tiene [preparada la máquina virtual](prepare-for-upload-vhd-image.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) antes de ejecutar Sysprep. 
> 
> 

También puede generalizar una máquina virtual Linux con `sudo waagent -deprovision+user` y, luego, usar PowerShell para capturar la máquina virtual. Para obtener información sobre cómo usar la CLI para capturar una máquina virtual, consulte el artículo sobre [cómo generalizar y capturar una máquina virtual Linux mediante la CLI de Azure](../linux/capture-image.md).


1. Inicie sesión en la máquina virtual de Windows.
2. Abra una ventana del símbolo del sistema como administrador. Cambie el directorio a **%windir%\system32\sysprep** y, después, ejecute `sysprep.exe`.
3. En **Herramienta de preparación del sistema**, seleccione **Iniciar la Configuración rápida (OOBE)** y asegúrese de que la casilla **Generalizar** está activada.
4. En **Opciones de apagado**, seleccione **Apagar**.
5. Haga clic en **OK**.
   
    ![Iniciar Sysprep](./media/upload-generalized-managed/sysprepgeneral.png)
6. Cuando Sysprep finaliza, apaga la máquina virtual. 

> [!IMPORTANT]
> No reinicie la máquina virtual hasta que haya terminado de cargar el VHD en Azure o de crear una imagen de la máquina virtual. Si la máquina virtual se reinicia accidentalmente, ejecute Sysprep para generalizarla de nuevo.
> 
> 

## <a name="log-in-to-azure-powershell"></a>Iniciar sesión en Azure PowerShell
1. Abra Azure PowerShell e inicie sesión en la cuenta de Azure.
   
    ```powershell
    Connect-AzAccount
    ```
   
    Se abrirá una ventana emergente para que escriba sus credenciales de cuenta de Azure.
2. Obtenga los identificadores de suscripción para las suscripciones disponibles.
   
    ```powershell
    Get-AzSubscription
    ```
3. Establezca la suscripción correcta con el identificador de suscripción.
   
    ```powershell
    Select-AzSubscription -SubscriptionId "<subscriptionID>"
    ```

## <a name="deallocate-the-vm-and-set-the-state-to-generalized"></a>Desasignar la máquina virtual y establecer el estado en generalizado

> [!IMPORTANT] 
> No se puede agregar, editar o quitar etiquetas de una máquina virtual una vez que está marcada como generalizada. Si desea agregar una etiqueta a la máquina virtual, asegúrese de que lo hace antes de marcarla como generalizada.
> 

1. Desasigne los recursos de máquina virtual.
   
    ```powershell
    Stop-AzVM -ResourceGroupName <resourceGroup> -Name <vmName>
    ```
   
    El *estado* de la máquina virtual en Azure Portal cambia de **Detenido** a **Detenido (desasignado)** .
2. Establezca el estado de la máquina virtual en **Generalizado**. 
   
    ```powershell
    Set-AzVm -ResourceGroupName <resourceGroup> -Name <vmName> -Generalized
    ```
3. Compruebe el estado de la máquina virtual. En la sección **OSState/generalized** de la máquina virtual, **DisplayStatus** debe estar establecido en **VM generalized** (Máquina virtual generalizada).  
   
    ```powershell
    $vm = Get-AzVM -ResourceGroupName <resourceGroup> -Name <vmName> -Status
    $vm.Statuses
    ```

## <a name="create-the-image"></a>Crear la imagen

Cree una imagen de máquina virtual no administrada en el contenedor de almacenamiento de destino con este comando. La imagen se crea en la misma cuenta de almacenamiento que la de la máquina virtual original. El parámetro `-Path` guarda una copia de la plantilla JSON para la máquina virtual de origen en el equipo local. El parámetro `-DestinationContainerName` es el nombre del contenedor que desea que contenga las imágenes. Si el contenedor no existe, se creará.
   
```powershell
Save-AzVMImage -ResourceGroupName <resourceGroupName> -Name <vmName> `
    -DestinationContainerName <destinationContainerName> -VHDNamePrefix <templateNamePrefix> `
    -Path <C:\local\Filepath\Filename.json>
```
   
Puede obtener la dirección URL de la imagen de la plantilla del archivo JSON. Vaya a la sección **recursos** > **storageProfile** > **osDisk** > **imagen** > **uri** para ver la ruta de acceso completa de la imagen. La dirección URL de la imagen es similar a esta: `https://<storageAccountName>.blob.core.windows.net/system/Microsoft.Compute/Images/<imagesContainer>/<templatePrefix-osDisk>.xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx.vhd`.
   
También puede comprobar el URI en el portal. La imagen se copia en un contenedor denominado **system** de su cuenta de almacenamiento. 

## <a name="create-a-vm-from-the-image"></a>Creación de una máquina virtual a partir de una imagen

Ahora puede crear una o varias máquinas virtuales a partir de la imagen no administrada.

### <a name="set-the-uri-of-the-vhd"></a>Establecimiento del URI del VHD

El URI para el VHD que se usará está en el formato: https://**mystorageaccount**.blob.core.windows.net/**mycontainer**/**MyVhdName**.vhd. En este ejemplo, el VHD denominado "**myVHD**" es la cuenta de almacenamiento **mystorageaccount** del contenedor **mycontainer**.

```powershell
$imageURI = "https://mystorageaccount.blob.core.windows.net/mycontainer/myVhd.vhd"
```


### <a name="create-a-virtual-network"></a>Creación de una red virtual
Cree la red virtual y la subred de la [red virtual](../../virtual-network/virtual-networks-overview.md).

1. Cree la subred. El ejemplo siguiente crea una subred denominada "**mySubnet**" en el grupo de recursos "**myResourceGroup**" con el prefijo de dirección **10.0.0.0/24**.  
   
    ```powershell
    $rgName = "myResourceGroup"
    $subnetName = "mySubnet"
    $singleSubnet = New-AzVirtualNetworkSubnetConfig -Name $subnetName -AddressPrefix 10.0.0.0/24
    ```
2. Creación de la red virtual. El ejemplo siguiente crea una red virtual denominada "**myVnet**" en la ubicación **oeste de EE. UU.** con el prefijo de dirección **10.0.0.0/16**.  
   
    ```powershell
    $location = "West US"
    $vnetName = "myVnet"
    $vnet = New-AzVirtualNetwork -Name $vnetName -ResourceGroupName $rgName -Location $location `
        -AddressPrefix 10.0.0.0/16 -Subnet $singleSubnet
    ```    

### <a name="create-a-public-ip-address-and-network-interface"></a>Creación de una dirección IP y una interfaz de red
Para permitir la comunicación con la máquina virtual en la red virtual, necesitará una [dirección IP pública](../../virtual-network/public-ip-addresses.md) y una interfaz de red.

1. Cree una dirección IP pública. Este ejemplo crea una dirección IP pública denominada "**myPip**". 
   
    ```powershell
    $ipName = "myPip"
    $pip = New-AzPublicIpAddress -Name $ipName -ResourceGroupName $rgName -Location $location `
        -AllocationMethod Dynamic
    ```       
2. Cree la NIC. Este ejemplo crea una NIC denominada "**myNic**". 
   
    ```powershell
    $nicName = "myNic"
    $nic = New-AzNetworkInterface -Name $nicName -ResourceGroupName $rgName -Location $location `
        -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id
    ```

### <a name="create-the-network-security-group-and-an-rdp-rule"></a>Creación del grupo de seguridad de red y las reglas de RDP
Para poder iniciar sesión en la máquina virtual mediante RDP, debe tener una regla de seguridad que permita el acceso RDP en el puerto 3389. 

Este ejemplo crea un NSG denominado"**myNsg**" que contiene una regla llamada "**myRdpRule**" que permita el tráfico RDP en el puerto 3389. Para obtener más información sobre NSG, consulte [Apertura de puertos para una máquina virtual en Azure mediante PowerShell](nsg-quickstart-powershell.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

```powershell
$nsgName = "myNsg"

$rdpRule = New-AzNetworkSecurityRuleConfig -Name myRdpRule -Description "Allow RDP" `
    -Access Allow -Protocol Tcp -Direction Inbound -Priority 110 `
    -SourceAddressPrefix Internet -SourcePortRange * `
    -DestinationAddressPrefix * -DestinationPortRange 3389

$nsg = New-AzNetworkSecurityGroup -ResourceGroupName $rgName -Location $location `
    -Name $nsgName -SecurityRules $rdpRule
```


### <a name="create-a-variable-for-the-virtual-network"></a>Creación de una variable para la red virtual
Cree una variable para la red virtual completada. 

```powershell
$vnet = Get-AzVirtualNetwork -ResourceGroupName $rgName -Name $vnetName
```

### <a name="create-the-vm"></a>Creación de la máquina virtual
El siguiente script de PowerShell completa las configuraciones de la máquina virtual y usa una imagen no administrada como origen para la nueva instalación.

</br>

```powershell
    # Enter a new user name and password to use as the local administrator account 
    # for remotely accessing the VM.
    $cred = Get-Credential

    # Name of the storage account where the VHD is located. This example sets the 
    # storage account name as "myStorageAccount"
    $storageAccName = "myStorageAccount"

    # Name of the virtual machine. This example sets the VM name as "myVM".
    $vmName = "myVM"

    # Size of the virtual machine. This example creates "Standard_D2_v2" sized VM. 
    # See the VM sizes documentation for more information: 
    # https://azure.microsoft.com/documentation/articles/virtual-machines-windows-sizes/
    $vmSize = "Standard_D2_v2"

    # Computer name for the VM. This examples sets the computer name as "myComputer".
    $computerName = "myComputer"

    # Name of the disk that holds the OS. This example sets the 
    # OS disk name as "myOsDisk"
    $osDiskName = "myOsDisk"

    # Assign a SKU name. This example sets the SKU name as "Standard_LRS"
    # Valid values for -SkuName are: Standard_LRS - locally redundant storage, Standard_ZRS - zone redundant
    # storage, Standard_GRS - geo redundant storage, Standard_RAGRS - read access geo redundant storage,
    # Premium_LRS - premium locally redundant storage. 
    $skuName = "Standard_LRS"

    # Get the storage account where the uploaded image is stored
    $storageAcc = Get-AzStorageAccount -ResourceGroupName $rgName -AccountName $storageAccName

    # Set the VM name and size
    $vmConfig = New-AzVMConfig -VMName $vmName -VMSize $vmSize

    #Set the Windows operating system configuration and add the NIC
    $vm = Set-AzVMOperatingSystem -VM $vmConfig -Windows -ComputerName $computerName `
        -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
    $vm = Add-AzVMNetworkInterface -VM $vm -Id $nic.Id

    # Create the OS disk URI
    $osDiskUri = '{0}vhds/{1}-{2}.vhd' `
        -f $storageAcc.PrimaryEndpoints.Blob.ToString(), $vmName.ToLower(), $osDiskName

    # Configure the OS disk to be created from the existing VHD image (-CreateOption fromImage).
    $vm = Set-AzVMOSDisk -VM $vm -Name $osDiskName -VhdUri $osDiskUri `
        -CreateOption fromImage -SourceImageUri $imageURI -Windows

    # Create the new VM
    New-AzVM -ResourceGroupName $rgName -Location $location -VM $vm
```

### <a name="verify-that-the-vm-was-created"></a>Comprobación de que se creó la máquina virtual
Cuando finalice, debería ver la máquina virtual recién creada en el [Azure Portal](https://portal.azure.com), en **Examinar** > **Máquinas virtuales** o mediante los comandos de PowerShell siguientes:

```powershell
    $vmList = Get-AzVM -ResourceGroupName $rgName
    $vmList.Name
```

## <a name="next-steps"></a>Pasos siguientes
Para administrar la nueva máquina virtual de Azure PowerShell, consulte [Administración de máquinas virtuales de Azure con Resource Manager y PowerShell](tutorial-manage-vm.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
