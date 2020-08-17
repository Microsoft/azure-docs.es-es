---
title: 'Crear una VM Windows con redes aceleradas: Azure PowerShell'
description: Cree una máquina virtual (VM) Windows con redes aceleradas para mejorar considerablemente el rendimiento de la red.
services: virtual-network
documentationcenter: ''
author: gsilva5
manager: gedegrac
editor: ''
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 04/15/2020
ms.author: gsilva
ms.openlocfilehash: fd50af98fe0d7f20273c45e2b86c18215a3626f0
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87289622"
---
# <a name="create-a-windows-vm-with-accelerated-networking-using-azure-powershell"></a>Crear una VM Windows con redes aceleradas mediante Azure PowerShell

En este tutorial, aprenderá a crear una máquina virtual (VM) Windows con redes aceleradas.

> [!NOTE]
> Para usar redes aceleradas con una máquina virtual Linux, consulte [Creación de una máquina virtual Linux con redes aceleradas](create-vm-accelerated-networking-cli.md).

Accelerated Networking habilita la virtualización de E/S de raíz única (SR-IOV) en una máquina virtual (VM), lo que mejora significativamente su rendimiento en la red. Este método de alto rendimiento omite el host de la ruta de acceso de datos, lo que reduce la latencia, la inestabilidad y el uso de la CPU para las cargas de trabajo de red más exigentes en los tipos de VM admitidos. En el diagrama siguiente, se muestra la comunicación entre dos VM con y sin redes aceleradas:

![Comunicación entre máquinas virtuales de Azure con y sin redes aceleradas](./media/create-vm-accelerated-networking/accelerated-networking.png)

Sin Accelerated Networking, todo el tráfico de red de entrada y salida de la máquina virtual tiene que atravesar el host y el conmutador virtual. El conmutador virtual se encarga de toda la aplicación de directivas, como grupos de seguridad de red, listas de control de acceso, aislamiento y otros servicios virtualizados de red, al tráfico de red.

> [!NOTE]
> Para obtener más información acerca de los conmutadores virtuales, consulte [Conmutador virtual de Hyper-V](/windows-server/virtualization/hyper-v-virtual-switch/hyper-v-virtual-switch).

Con redes aceleradas, el tráfico de red llega a la interfaz de red (NIC) de la VM y se reenvía después a la VM. Todas las directivas de red que el conmutador virtual aplica se descargan y aplican en el hardware. Dado que la directiva se aplica al hardware, la NIC puede reenviar el tráfico de red directamente a la VM. La NIC omite el host y el conmutador virtual, y mantiene toda la directiva que aplicó en el host.

Las ventajas de las redes aceleradas solo se aplican a la VM en que estén habilitadas. Para obtener resultados óptimos, habilite esta característica en al menos dos VM conectadas a la misma red virtual de Azure. Al comunicarse entre redes virtuales o conectarse de forma local, esta característica tiene un efecto mínimo sobre la latencia general.

## <a name="benefits"></a>Ventajas

- **Menor latencia / más paquetes por segundo (pps)** : Al eliminar el conmutador virtual de la ruta de acceso de datos, se quita el tiempo que los paquetes pasan en el host para el procesamiento de directivas. También aumenta el número de paquetes que se pueden procesar dentro de la VM.

- **Inestabilidad reducida**: El procesamiento de conmutadores virtuales depende de la cantidad de directiva que se deba aplicar. También depende de la carga de trabajo de la CPU que realiza el procesamiento. La descarga de la aplicación de directivas en el hardware elimina esa variabilidad mediante la entrega de paquetes directamente a la VM. La descarga también quita la comunicación entre el host y la VM, todas las interrupciones de software y todos los cambios de contexto.

- **Disminución de la utilización de la CPU**: el pasar por alto el conmutador virtual en el host conlleva una disminución de la utilización de la CPU para procesar el tráfico de red.

## <a name="supported-operating-systems"></a>Sistemas operativos admitidos

Se admiten las siguientes distribuciones de fábrica desde la galería de Azure:

- **Windows Server 2019 Datacenter**
- **Windows Server 2016 Datacenter** 
- **Windows Server 2012 R2 Datacenter**

## <a name="limitations-and-constraints"></a>Limitaciones y restricciones

### <a name="supported-vm-instances"></a>Instancias de máquina virtual admitidas

Las redes aceleradas son compatibles con la mayoría de los tamaños de instancia de uso general y optimizados para procesos con dos o más CPU virtuales (vCPU).  Estas series admitidas son: Dv2/DSv2 y F/Fs.

En instancias que admiten hyperthreading, las redes aceleradas se admiten en instancias de VM con cuatro o más vCPU. Las series admitidas son: D/Dsv3, D/Dsv4, Da/Dasv4, E/Esv3, Ea/Easv4, Fsv2, Lsv2, Ms/Mms y Ms/Mmsv2.

Para obtener más información sobre instancias de VM, consulte [Tamaños de las máquinas virtuales Windows en Azure](../virtual-machines/windows/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

### <a name="custom-images"></a>Imágenes personalizadas

Si usa una imagen personalizada compatible con redes aceleradas, asegúrese de que dispone de los controladores necesarios para trabajar con las NIC ConnectX-3 y ConnectX-4 Lx de Mellanox en Azure.

### <a name="regions"></a>Regions

Las redes aceleradas están disponibles en todas las regiones globales de Azure y la nube de Azure Government.

### <a name="enabling-accelerated-networking-on-a-running-vm"></a>Habilitación de redes aceleradas en una VM en ejecución

Un tamaño de VM admitido sin redes aceleradas habilitadas solo puede tener la característica habilitada cuando se detiene y desasigna.

### <a name="deployment-through-azure-resource-manager"></a>Implementación mediante Azure Resource Manager

Las máquinas virtuales (clásicas) no se pueden implementar con redes aceleradas.

## <a name="vm-creation-using-the-portal"></a>Creación de VM desde el portal

Aunque en este artículo se explica cómo crear una VM con redes aceleradas mediante Azure PowerShell, también puede [usar Azure Portal para crear una máquina virtual](../virtual-machines/windows/quick-create-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json) que habilite las redes aceleradas. Al crear una VM en el portal, en la página **Crear una máquina virtual**, seleccione la pestaña **Redes**. En esta pestaña hay una opción para **Redes aceleradas**. Si ha seleccionado un [sistema operativo compatible](#supported-operating-systems) y un [tamaño de VM](#supported-vm-instances), esta opción se define automáticamente como **Activadas**. De lo contrario, la opción se establece en **Desactivadas** y Azure muestra el motivo por el que no se pueden habilitar.

> [!NOTE]
> Solo los sistemas operativos compatibles se pueden habilitar a través del portal. Si usa una imagen personalizada y esta es compatible con las redes aceleradas, cree la VM mediante la CLI o PowerShell. 

Después de crear la VM, puede confirmar si las redes aceleradas están habilitadas. Siga estas instrucciones:

1. Vaya a [Azure Portal](https://portal.azure.com) para administrar sus máquinas virtuales. Busque y seleccione **Máquinas virtuales**.

2. En la lista de máquinas virtuales, elija la nueva VM.

3. En la barra de menús de la máquina virtual, elija **Redes**.

En la información de la interfaz de red, junto a la etiqueta **Redes aceleradas**, el portal muestra **Deshabilitadas** o **Habilitadas** como estado.

## <a name="vm-creation-using-powershell"></a>Creación de VM mediante PowerShell

Antes de continuar, instale [Azure PowerShell](/powershell/azure/install-az-ps) versión 1.0.0 o posterior. Para encontrar la versión instalada actualmente, ejecute `Get-Module -ListAvailable Az`. Si necesita instalar o actualizar, instale la versión más reciente del módulo Az desde la [Galería de PowerShell](https://www.powershellgallery.com/packages/Az). En una sesión de PowerShell, inicie sesión en una cuenta de Azure con [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount).

En los ejemplos siguientes, reemplace los nombres de parámetros de ejemplo por los suyos propios. Los nombres de parámetro de ejemplo incluían *myResourceGroup*, *myNic* y *myVM*.

### <a name="create-a-virtual-network"></a>Creación de una red virtual

1. Cree un grupo de recursos con [New-AzResourceGroup](/powershell/module/az.Resources/New-azResourceGroup). Con el comando siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *centralus*:

    ```azurepowershell
    New-AzResourceGroup -Name "myResourceGroup" -Location "centralus"
    ```

2. Cree una configuración de subred con [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.Network/New-azVirtualNetworkSubnetConfig). Con el comando siguiente se crea una subred denominada *mySubnet*:

    ```azurepowershell
    $subnet = New-AzVirtualNetworkSubnetConfig `
        -Name "mySubnet" `
        -AddressPrefix "192.168.1.0/24"
    ```

3. Cree una red virtual con [New-AzVirtualNetwork](/powershell/module/az.Network/New-azVirtualNetwork), con la subred *mySubnet*.

    ```azurepowershell
    $vnet = New-AzVirtualNetwork -ResourceGroupName "myResourceGroup" `
        -Location "centralus" `
        -Name "myVnet" `
        -AddressPrefix "192.168.0.0/16" `
        -Subnet $Subnet
    ```

### <a name="create-a-network-security-group"></a>Crear un grupo de seguridad de red

1. Cree una regla de grupo de seguridad de red con [New-AzNetworkSecurityRuleConfig](/powershell/module/az.Network/New-azNetworkSecurityRuleConfig).

    ```azurepowershell
    $rdp = New-AzNetworkSecurityRuleConfig `
        -Name 'Allow-RDP-All' `
        -Description 'Allow RDP' `
        -Access Allow `
        -Protocol Tcp `
        -Direction Inbound `
        -Priority 100 `
        -SourceAddressPrefix * `
        -SourcePortRange * `
        -DestinationAddressPrefix * `
        -DestinationPortRange 3389
    ```

2. Cree un grupo de seguridad de red con [New-AzNetworkSecurityGroup](/powershell/module/az.Network/New-azNetworkSecurityGroup) y asígnele la regla de seguridad *Allow-RDP-All*. Aparte de la regla *Allow-RDP-All*, el grupo de seguridad de red contiene varias reglas predeterminadas. Una regla predeterminada deshabilita todo el acceso entrante desde Internet. Una vez creada, la regla *Allow-RDP-All* se asigna al grupo de seguridad de red para que pueda conectarse de forma remota a la VM.

    ```azurepowershell
    $nsg = New-AzNetworkSecurityGroup `
        -ResourceGroupName myResourceGroup `
        -Location centralus `
        -Name "myNsg" `
        -SecurityRules $rdp
    ```

3. Asocie el grupo de seguridad de red a la subred *mySubnet* con [Set-AzVirtualNetworkSubnetConfig](/powershell/module/az.Network/Set-azVirtualNetworkSubnetConfig). La regla en el grupo de seguridad de red es eficaz para todos los recursos implementados en la subred.

    ```azurepowershell
    Set-AzVirtualNetworkSubnetConfig `
        -VirtualNetwork $vnet `
        -Name 'mySubnet' `
        -AddressPrefix "192.168.1.0/24" `
        -NetworkSecurityGroup $nsg
    ```

### <a name="create-a-network-interface-with-accelerated-networking"></a>Creación de una interfaz de red con Accelerated Networking

1. Cree una dirección IP pública con [New-AzPublicIpAddress](/powershell/module/az.Network/New-azPublicIpAddress). Una dirección IP pública es innecesaria si no tiene previsto acceder a la VM desde Internet. Sin embargo, se necesita para completar los pasos de este artículo.

    ```azurepowershell
    $publicIp = New-AzPublicIpAddress `
        -ResourceGroupName myResourceGroup `
        -Name 'myPublicIp' `
        -location centralus `
        -AllocationMethod Dynamic
    ```

2. Cree una interfaz de red con [New-AzNetworkInterface](/powershell/module/az.Network/New-azNetworkInterface) con la redes aceleradas habilitadas y asígnele la dirección IP pública. En el ejemplo siguiente se crea una interfaz de red denominada *myNic* en la subred *mySubnet* de la red virtual *myVnet* y se le asigna la dirección IP pública *myPublicIp* :

    ```azurepowershell
    $nic = New-AzNetworkInterface `
        -ResourceGroupName "myResourceGroup" `
        -Name "myNic" `
        -Location "centralus" `
        -SubnetId $vnet.Subnets[0].Id `
        -PublicIpAddressId $publicIp.Id `
        -EnableAcceleratedNetworking
    ```

### <a name="create-a-vm-and-attach-the-network-interface"></a>Creación de una VM y conexión de la interfaz de red

1. Establezca las credenciales de la VM en la variable `$cred` con [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential). Se le pedirá que inicie sesión:

    ```azurepowershell
    $cred = Get-Credential
    ```

2. Defina la VM con [New-AzVMConfig](/powershell/module/az.compute/new-azvmconfig). En el ejemplo siguiente se define una VM denominada *myVM* con un tamaño que admite redes aceleradas (*Standard_DS4_v2*):

    ```azurepowershell
    $vmConfig = New-AzVMConfig -VMName "myVm" -VMSize "Standard_DS4_v2"
    ```

    Para obtener una lista de todos los tamaños de máquinas virtuales y las características, consulte [tamaños de máquina virtual de Windows](../virtual-machines/windows/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

3. Cree el resto de la configuración de VM con [Set-AzVMOperatingSystem](/powershell/module/az.compute/set-azvmoperatingsystem) y [Set-AzVMSourceImage](/powershell/module/az.compute/set-azvmsourceimage). Con el comando siguiente, se crea una VM con Windows Server 2016:

    ```azurepowershell
    $vmConfig = Set-AzVMOperatingSystem -VM $vmConfig `
        -Windows `
        -ComputerName "myVM" `
        -Credential $cred `
        -ProvisionVMAgent `
        -EnableAutoUpdate
    $vmConfig = Set-AzVMSourceImage -VM $vmConfig `
        -PublisherName "MicrosoftWindowsServer" `
        -Offer "WindowsServer" `
        -Skus "2016-Datacenter" `
        -Version "latest"
    ```

4. Adjunte la interfaz de red creada anteriormente con [Add-AzVMNetworkInterface](/powershell/module/az.compute/add-azvmnetworkinterface):

    ```azurepowershell
    $vmConfig = Add-AzVMNetworkInterface -VM $vmConfig -Id $nic.Id
    ```

5. Cree la VM con [New-AzVM](/powershell/module/az.compute/new-azvm).

    ```azurepowershell
    New-AzVM -VM $vmConfig -ResourceGroupName "myResourceGroup" -Location "centralus"
    ```

### <a name="confirm-the-ethernet-controller-is-installed-in-the-windows-vm"></a>Confirmación de que el controlador de Ethernet está instalado en la VM Windows

Una vez creada la VM en Azure, conéctese a ella y confirme que el controlador de Ethernet está instalado en Windows.

1. Vaya a [Azure Portal](https://portal.azure.com) para administrar sus máquinas virtuales. Busque y seleccione **Máquinas virtuales**.

2. En la lista de máquinas virtuales, elija la nueva VM.

3. En la página información general de la VM, si el **Estado** de la VM aparece como **Creando**, espere a que Azure termine de crear la VM. El **Estado** cambiará a **En ejecución** cuando se complete la creación de la VM.

4. En la barra de herramientas de información general de la VM, seleccione **Conectar** > **RDP** > **Descargar archivo RDP**.

5. Abra el archivo .rdp y, a continuación, inicie sesión en la VM con las credenciales que especificó en la sección [Creación de una VM y conexión de la interfaz de red](#create-a-vm-and-attach-the-network-interface). Si nunca se ha conectado a una máquina virtual Windows en Azure, consulte [Conexión a la máquina virtual](../virtual-machines/windows/quick-create-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json#connect-to-virtual-machine).

6. Cuando aparezca la sesión de escritorio remoto para la VM, haga clic con el botón derecho en el botón Inicio de Windows y elija **Administrador de dispositivos**.

7. En la ventana **Administrador de dispositivos**, expanda el nodo **Adaptadores de red**.

8. Compruebe que el **Adaptador Ethernet de función virtual ConnectX-3 de Mellanox** aparezca como se muestra en la siguiente imagen:

    ![Adaptador Ethernet de función virtual ConnectX-3 de Mellanox, nuevo adaptador de red para redes aceleradas, Administrador de dispositivos](./media/create-vm-accelerated-networking/device-manager.png)

Las redes aceleradas ya están habilitadas para su VM.

> [!NOTE]
> Si el adaptador de Mellanox no se inicia, abra un símbolo del sistema del administrador en la sesión de escritorio remoto y escriba el siguiente comando:
>
> `netsh int tcp set global rss = enabled`

## <a name="enable-accelerated-networking-on-existing-vms"></a>Habilitación de redes aceleradas en VM existentes

Si ha creado una VM sin redes aceleradas, es posible habilitar esta característica en una VM existente. Para admitir redes aceleradas, la VM debe cumplir los siguientes requisitos previos, que también se han descrito antes:

* La VM debe tener un tamaño admitido para redes aceleradas.
* La VM debe ser una imagen admitida de la galería de Azure (y la versión de kernel de Linux).
* Todas las VM de un conjunto de disponibilidad o conjunto de escalado de máquinas virtuales se deben detener o desasignar antes de habilitar las redes aceleradas en cualquier NIC.

### <a name="individual-vms-and-vms-in-an-availability-set"></a>VM individuales y VM de un conjunto de disponibilidad

1. Detenga o desasigne la VM o, en el caso de un conjunto de disponibilidad, todas las VM del conjunto:

    ```azurepowershell
    Stop-AzVM -ResourceGroup "myResourceGroup" -Name "myVM"
    ```

    > [!NOTE]
    > Si la VM se crea de forma individual, sin un conjunto de disponibilidad, solo es necesario detener o desasignar esa VM para habilitar las redes aceleradas. Si la VM se creó con un conjunto de disponibilidad, debe detener o desasignar todas las VM contenidas en el conjunto de disponibilidad antes de habilitar las redes aceleradas en cualquiera de las NIC, de modo que las VM terminen en un clúster que admita redes aceleradas. El requisito de detención o desasignación no es necesario si se deshabilita la aceleración de redes, ya que los clústeres que admiten redes aceleradas también funcionan correctamente con las NIC que no usan redes aceleradas.

2. Habilite las redes aceleradas en la NIC de la VM:

    ```azurepowershell
    $nic = Get-AzNetworkInterface -ResourceGroupName "myResourceGroup" `
        -Name "myNic"
    
    $nic.EnableAcceleratedNetworking = $true
    
    $nic | Set-AzNetworkInterface
    ```

3. Reinicie la VM o, en el caso de un conjunto de disponibilidad, todas las máquinas virtuales del conjunto, y confirme que las redes aceleradas están habilitadas:

    ```azurepowershell
    Start-AzVM -ResourceGroup "myResourceGroup" `
        -Name "myVM"
    ```

### <a name="virtual-machine-scale-set"></a>Conjunto de escalado de máquina virtual

Un conjunto de escalado de máquinas virtuales es ligeramente diferente, pero sigue el mismo flujo de trabajo.

1. Detenga las VM:

    ```azurepowershell
    Stop-AzVmss -ResourceGroupName "myResourceGroup" `
        -VMScaleSetName "myScaleSet"
    ```

2. Actualice la propiedad de redes aceleradas en la interfaz de red:

    ```azurepowershell
    $vmss = Get-AzVmss -ResourceGroupName "myResourceGroup" `
        -VMScaleSetName "myScaleSet"
    
    $vmss.VirtualMachineProfile.NetworkProfile.NetworkInterfaceConfigurations[0].EnableAcceleratedNetworking = $true
    
    Update-AzVmss -ResourceGroupName "myResourceGroup" `
        -VMScaleSetName "myScaleSet" `
        -VirtualMachineScaleSet $vmss
    ```

3. Defina las actualizaciones aplicadas como automáticas para recoger los cambios inmediatamente:

    ```azurepowershell
    $vmss.UpgradePolicy.Mode = "Automatic"
    
    Update-AzVmss -ResourceGroupName "myResourceGroup" `
        -VMScaleSetName "myScaleSet" `
        -VirtualMachineScaleSet $vmss
    ```

    > [!NOTE]
    > Un conjunto de escalado tiene actualizaciones de VM que aplican actualizaciones mediante tres valores diferentes: automático, gradual y manual. En estas instrucciones, la directiva se define como automática para que el conjunto de escalado recoja los cambios inmediatamente después de reiniciarse.

4. Reinicie el conjunto de escalado:

    ```azurepowershell
    Start-AzVmss -ResourceGroupName "myResourceGroup" `
        -VMScaleSetName "myScaleSet"
    ```

Después de reiniciar, espere a que finalicen las actualizaciones. Una vez finalizadas las actualizaciones, la función virtual (VF) aparece dentro de la VM. Asegúrese de que usa un tamaño de VM y SO admitidos.

### <a name="resizing-existing-vms-with-accelerated-networking"></a>Cambio del tamaño de las VM existentes con redes aceleradas

Si una VM tiene habilitadas las redes aceleradas, solo podrá cambiar su tamaño al de una VM que admita redes aceleradas.  

Una VM con redes aceleradas habilitadas no puede cambiar al tamaño de una instancia de VM que no admite redes aceleradas mediante la operación de cambio de tamaño. Para cambiar el tamaño de una de estas VM, debe hacer lo siguiente:

1. Detenga o desasigne la VM. Para un conjunto de disponibilidad o conjunto de escalado, detenga o desasigne todas las VM del conjunto de disponibilidad o escalado.

2. Deshabilite las redes aceleradas en la NIC de la VM. Para un conjunto de disponibilidad o conjunto de escalado, deshabilite las redes aceleradas en las NIC de todas las VM del conjunto de disponibilidad o de escalado.

3. Después de deshabilitar las redes aceleradas, cambie la VM, el conjunto de disponibilidad o el conjunto de escalado a un nuevo tamaño que no admita redes aceleradas y reinicie.
