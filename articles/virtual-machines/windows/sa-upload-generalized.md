---
title: Carga de un disco duro virtual generalizado para crear varias máquinas virtuales en Azure
description: Cargue un VHD generalizado en una cuenta de Azure Storage a fin de crear una máquina virtual de Windows para usarla con el modelo de implementación de Resource Manager.
author: cynthn
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 05/18/2017
ms.author: cynthn
ROBOTS: NOINDEX
ms.custom: storage-accounts
ms.openlocfilehash: d340e37cf64961971c03af8d08a669c27d758116
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87074193"
---
# <a name="upload-a-generalized-vhd-to-azure-to-create-a-new-vm"></a>Carga de un VHD generalizado en Azure para crear una máquina virtual nueva

En este tema se explica cómo cargar un disco no administrado generalizado en una cuenta de almacenamiento para crear después una máquina virtual nueva con el disco cargado. Con Sysprep se elimina toda la información personal de la cuenta de una imagen de un VHD generalizado. 

Si desea crear una máquina virtual a partir de un VHD especializado en una cuenta de almacenamiento, consulte [Crear una VM a partir de un VHD especializado](sa-create-vm-specialized.md).

En este tema se aborda cómo usar las cuentas de almacenamiento, pero se recomienda a los clientes que utilicen Managed Disks. Para ver un tutorial completo sobre cómo preparar, cargar y crear una nueva máquina virtual con Managed Disks, vea [Creación de una VM nueva a partir de un VHD generalizado cargado en Azure a través de Managed Disks](upload-generalized-managed.md).

 

## <a name="prepare-the-vm"></a>Preparación de la VM

La información personal de la cuenta se elimina de un VHD generalizado mediante Sysprep. Si tiene previsto usar VHD como una imagen desde la que crear nuevas VM, debe:
  
  * [Preparar de un VHD de Windows para cargar en Azure](prepare-for-upload-vhd-image.md). 
  * Generalización de una máquina virtual con Sysprep

### <a name="generalize-a-windows-virtual-machine-using-sysprep"></a>Generalización de una máquina virtual Windows con Sysprep
En esta sección se muestra cómo generalizar la máquina virtual Windows para usarla como imagen. Entre otras características, Sysprep elimina toda la información personal de la cuenta y prepara, entre otras cosas, la máquina para usarse como imagen. Para más información acerca de Sysprep, consulte [Uso de Sysprep: Introducción](/previous-versions/windows/it-pro/windows-xp/bb457073(v=technet.10)).

Asegúrese de que los roles de servidor que se ejecutan en la máquina sean compatibles con Sysprep. Para más información, consulte [Sysprep Support for Server Roles](/windows-hardware/manufacture/desktop/sysprep-support-for-server-roles)

> [!IMPORTANT]
> Si ejecuta Sysprep antes de cargar el VHD en Azure por primera vez, asegúrese de que tiene [preparada la máquina virtual](prepare-for-upload-vhd-image.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) antes de ejecutar Sysprep. 
> 
> 

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


## <a name="upload-the-vhd"></a>Carga del disco duro virtual

Cargue el VHD en una cuenta de Azure Storage.

### <a name="log-in-to-azure"></a>Inicio de sesión en Azure
Si todavía no la ha instalado la versión 1.4 de PowerShell u otra superior, consulte [Cómo instalar y configurar Azure PowerShell](/powershell/azure/).

1. Abra Azure PowerShell e inicie sesión en la cuenta de Azure. Se abrirá una ventana emergente para que escriba sus credenciales de cuenta de Azure.
   
    ```powershell
    Connect-AzAccount
    ```
2. Obtenga los identificadores de suscripción para las suscripciones disponibles.
   
    ```powershell
    Get-AzSubscription
    ```
3. Establezca la suscripción correcta con el identificador de suscripción. Reemplace `<subscriptionID>` con el identificador de la suscripción correcta.
   
    ```powershell
    Select-AzSubscription -SubscriptionId "<subscriptionID>"
    ```

### <a name="get-the-storage-account"></a>Obtención de la cuenta de almacenamiento
Necesita una cuenta de almacenamiento de Azure para almacenar la imagen de VM cargada. Puede usar una cuenta de almacenamiento existente o crear una nueva. 

Para mostrar las cuentas de almacenamiento disponibles, escriba:

```powershell
Get-AzStorageAccount
```

Si desea utilizar una cuenta de almacenamiento existente, puede continuar con la sección Carga de la imagen de VM en la cuenta de almacenamiento.

Si necesita crear una nueva cuenta de almacenamiento, siga estos pasos:

1. Necesita el nombre del grupo de recursos donde se debe crear la cuenta de almacenamiento. Para averiguar todos los grupos de recursos que están en la suscripción, escriba:
   
    ```powershell
    Get-AzResourceGroup
    ```

    Para crear un grupo de recursos denominado **myResourceGroup** en la región **Oeste de EE. UU.** , escriba:

    ```powershell
    New-AzResourceGroup -Name myResourceGroup -Location "West US"
    ```

2. Cree una cuenta de almacenamiento denominada **mystorageaccount** en este grupo de recursos con el cmdlet [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount):
   
    ```powershell
    New-AzStorageAccount -ResourceGroupName myResourceGroup -Name mystorageaccount -Location "West US" `
        -SkuName "Standard_LRS" -Kind "Storage"
    ```
 
### <a name="start-the-upload"></a>Inicio de la carga 

Use el cmdlet [Add-AzVhd](/powershell/module/az.compute/add-azvhd) para cargar la imagen en un contenedor de su cuenta de almacenamiento. En este ejemplo se carga el archivo **myVHD.vhd** de `"C:\Users\Public\Documents\Virtual hard disks\"` para una cuenta de almacenamiento denominada **mystorageaccount** en el grupo de recursos **myResourceGroup**. El archivo se colocará en el contenedor llamado **mycontainer** y el nuevo nombre de archivo será **myUploadedVHD.vhd**.

```powershell
$rgName = "myResourceGroup"
$urlOfUploadedImageVhd = "https://mystorageaccount.blob.core.windows.net/mycontainer/myUploadedVHD.vhd"
Add-AzVhd -ResourceGroupName $rgName -Destination $urlOfUploadedImageVhd `
    -LocalFilePath "C:\Users\Public\Documents\Virtual hard disks\myVHD.vhd"
```


Si se realiza correctamente, obtendrá una respuesta similar a la siguiente:

```powershell
MD5 hash is being calculated for the file C:\Users\Public\Documents\Virtual hard disks\myVHD.vhd.
MD5 hash calculation is completed.
Elapsed time for the operation: 00:03:35
Creating new page blob of size 53687091712...
Elapsed time for upload: 01:12:49

LocalFilePath           DestinationUri
-------------           --------------
C:\Users\Public\Doc...  https://mystorageaccount.blob.core.windows.net/mycontainer/myUploadedVHD.vhd
```

Dependiendo de la conexión de red y del tamaño del archivo VHD, este comando puede tardar algún tiempo en completarse.


## <a name="create-a-new-vm"></a>Creación de una máquina virtual nueva 

Ya puede usar el VHD cargado para crear una nueva máquina virtual. 

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
    $location = "WestUS"
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
En el siguiente script de PowerShell se muestra cómo establecer las configuraciones de máquina virtual y usar la imagen de máquina virtual cargada como origen para la nueva instalación.



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

## <a name="verify-that-the-vm-was-created"></a>Comprobación de que se creó la máquina virtual
Cuando finalice, debería ver la máquina virtual recién creada en el [Azure Portal](https://portal.azure.com), en **Examinar** > **Máquinas virtuales** o mediante los comandos de PowerShell siguientes:

```powershell
    $vmList = Get-AzVM -ResourceGroupName $rgName
    $vmList.Name
```

## <a name="next-steps"></a>Pasos siguientes
Para administrar la nueva máquina virtual de Azure PowerShell, consulte [Administración de máquinas virtuales de Azure con Resource Manager y PowerShell](tutorial-manage-vm.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
