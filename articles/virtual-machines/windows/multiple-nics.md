---
title: Creación y administración de máquinas virtuales Windows en Azure que usan varias NIC
description: Obtenga información sobre cómo crear y administrar una máquina virtual con Windows que tenga varias tarjetas de interfaz de red (NIC) conectadas a ella mediante Azure PowerShell o las plantillas de Resource Manager.
author: cynthn
ms.service: virtual-machines-windows
ms.topic: how-to
ms.workload: infrastructure
ms.date: 09/26/2017
ms.author: cynthn
ms.openlocfilehash: ed1c5b749b778ef8334ea3b31ef17d3bf106484f
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87835551"
---
# <a name="create-and-manage-a-windows-virtual-machine-that-has-multiple-nics"></a>Crear y administrar una máquina virtual con Windows que tiene varias NIC
En Azure, las máquinas virtuales (VM) pueden tener varias tarjetas de interfaz de red virtual (NIC) conectadas a ellas. Un escenario común es tener distintas subredes para la conectividad front-end y back-end. Puede asociar varias NIC de una máquina virtual a varias subredes, pero esas subredes deben residir en la misma red virtual (vNet). En este artículo se describe cómo crear una máquina virtual con varias NIC conectadas. También obtendrá información sobre cómo agregar o quitar NIC de una máquina virtual existente. Diferentes [tamaños de máquina virtual](../sizes.md) admiten un número distinto de NIC, así que ajuste el tamaño de su máquina virtual teniendo esto en cuenta.

## <a name="prerequisites"></a>Requisitos previos

En los ejemplos siguientes, reemplace los nombres de parámetros de ejemplo por los suyos propios. Los nombres de parámetro de ejemplo incluyen *myResourceGroup*, *myVnet* y *myVM*.

 

## <a name="create-a-vm-with-multiple-nics"></a>Creación de una máquina virtual con varias NIC
En primer lugar, cree un grupo de recursos. En el ejemplo siguiente se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *EastUs*:

```powershell
New-AzResourceGroup -Name "myResourceGroup" -Location "EastUS"
```

### <a name="create-virtual-network-and-subnets"></a>Creación de redes virtuales y subredes
Un escenario común es que una red virtual tenga dos o más subredes. Una subred puede ser para el tráfico de front-end y la otra para el tráfico de back-end. Para conectarse a las dos subredes, se usan varias NIC en la máquina virtual.

1. Defina dos subredes de red virtual con [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig). En el ejemplo siguiente se definen dos subredes para *mySubnetFrontEnd* y *mySubnetBackEnd*:

    ```powershell
    $mySubnetFrontEnd = New-AzVirtualNetworkSubnetConfig -Name "mySubnetFrontEnd" `
        -AddressPrefix "192.168.1.0/24"
    $mySubnetBackEnd = New-AzVirtualNetworkSubnetConfig -Name "mySubnetBackEnd" `
        -AddressPrefix "192.168.2.0/24"
    ```

2. Cree la red virtual y las subredes con [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork). En el ejemplo siguiente se crea una red virtual denominada *myVnet*:

    ```powershell
    $myVnet = New-AzVirtualNetwork -ResourceGroupName "myResourceGroup" `
        -Location "EastUs" `
        -Name "myVnet" `
        -AddressPrefix "192.168.0.0/16" `
        -Subnet $mySubnetFrontEnd,$mySubnetBackEnd
    ```


### <a name="create-multiple-nics"></a>Creación de varias NIC
Cree dos NIC con [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface). Conecte una NIC a la subred de front-end y la otra a la subred de back-end. En el ejemplo siguiente se crean las NIC denominadas *myNic1* y *myNic2*:

```powershell
$frontEnd = $myVnet.Subnets|?{$_.Name -eq 'mySubnetFrontEnd'}
$myNic1 = New-AzNetworkInterface -ResourceGroupName "myResourceGroup" `
    -Name "myNic1" `
    -Location "EastUs" `
    -SubnetId $frontEnd.Id

$backEnd = $myVnet.Subnets|?{$_.Name -eq 'mySubnetBackEnd'}
$myNic2 = New-AzNetworkInterface -ResourceGroupName "myResourceGroup" `
    -Name "myNic2" `
    -Location "EastUs" `
    -SubnetId $backEnd.Id
```

Normalmente también crea un [grupo de seguridad de red](../../virtual-network/security-overview.md) para filtrar el tráfico de red a la máquina virtual y un [equilibrador de carga](../../load-balancer/load-balancer-overview.md) para distribuir el tráfico entre varias máquinas virtuales.

### <a name="create-the-virtual-machine"></a>Creación de la máquina virtual
Comience ahora a compilar la configuración de la máquina virtual. El tamaño de cada máquina virtual tiene un límite en cuanto al número total de NIC que se pueden agregar a una máquina virtual. Para más información, vea [Tamaños de las máquinas virtuales con Windows](../sizes.md).

1. Establezca las credenciales de la máquina virtual en la variable `$cred` como se indica a continuación:

    ```powershell
    $cred = Get-Credential
    ```

2. Defina la VM con [New-AzVMConfig](/powershell/module/az.compute/new-azvmconfig). En el ejemplo siguiente se define una máquina virtual denominada *myVM* y se usa un tamaño de máquina virtual que admite hasta dos NIC (*Standard_DS3_v2*):

    ```powershell
    $vmConfig = New-AzVMConfig -VMName "myVM" -VMSize "Standard_DS3_v2"
    ```

3. Cree el resto de la configuración de VM con [Set-AzVMOperatingSystem](/powershell/module/az.compute/set-azvmoperatingsystem) y [Set-AzVMSourceImage](/powershell/module/az.compute/set-azvmsourceimage). En el ejemplo siguiente se crea una máquina virtual con Windows Server 2016:

    ```powershell
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

4. Conecte las dos NIC que creó anteriormente con [Add-AzVMNetworkInterface](/powershell/module/az.compute/add-azvmnetworkinterface):

    ```powershell
    $vmConfig = Add-AzVMNetworkInterface -VM $vmConfig -Id $myNic1.Id -Primary
    $vmConfig = Add-AzVMNetworkInterface -VM $vmConfig -Id $myNic2.Id
    ```

5. Cree la VM con [New-AzVM](/powershell/module/az.compute/new-azvm):

    ```powershell
    New-AzVM -VM $vmConfig -ResourceGroupName "myResourceGroup" -Location "EastUs"
    ```

6. Agregue rutas de tarjetas NIC secundarias al sistema operativo mediante los pasos que se indican en [Configuración del sistema operativo invitado para varias NIC](#configure-guest-os-for-multiple-nics).

## <a name="add-a-nic-to-an-existing-vm"></a>Adición de una NIC a una máquina virtual existente
Para agregar una NIC virtual a una máquina virtual existente, se desasigna la máquina virtual, se agrega la NIC virtual y, después, se inicia la máquina virtual. Diferentes [tamaños de máquina virtual](../sizes.md) admiten un número distinto de NIC, así que ajuste el tamaño de su máquina virtual teniendo esto en cuenta. Si es necesario, puede [cambiar el tamaño de una máquina virtual](resize-vm.md).

1. Desasigne la VM con [Stop-AzVM](/powershell/module/az.compute/stop-azvm). En el ejemplo siguiente se desasigna la máquina virtual denominada *myVM* en *myResourceGroup*:

    ```powershell
    Stop-AzVM -Name "myVM" -ResourceGroupName "myResourceGroup"
    ```

2. Obtenga la configuración existente de la VM con [Get-AzVm](/powershell/module/az.compute/get-azvm). En el ejemplo siguiente se obtiene información de la máquina virtual denominada *myVM* en *myResourceGroup*:

    ```powershell
    $vm = Get-AzVm -Name "myVM" -ResourceGroupName "myResourceGroup"
    ```

3. En el ejemplo siguiente se crea una NIC virtual con [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface) denominada *myNic3* que está conectada a *mySubnetBackEnd*. Después, la NIC virtual se conecta a la VM denominada *myVM* en *myResourceGroup* con [Add-AzVMNetworkInterface](/powershell/module/az.compute/add-azvmnetworkinterface):

    ```powershell
    # Get info for the back end subnet
    $myVnet = Get-AzVirtualNetwork -Name "myVnet" -ResourceGroupName "myResourceGroup"
    $backEnd = $myVnet.Subnets|?{$_.Name -eq 'mySubnetBackEnd'}

    # Create a virtual NIC
    $myNic3 = New-AzNetworkInterface -ResourceGroupName "myResourceGroup" `
        -Name "myNic3" `
        -Location "EastUs" `
        -SubnetId $backEnd.Id

    # Get the ID of the new virtual NIC and add to VM
    $nicId = (Get-AzNetworkInterface -ResourceGroupName "myResourceGroup" -Name "MyNic3").Id
    Add-AzVMNetworkInterface -VM $vm -Id $nicId | Update-AzVm -ResourceGroupName "myResourceGroup"
    ```

    ### <a name="primary-virtual-nics"></a>NIC virtuales principales
    En una máquina virtual de varias NIC una de las NIC debe ser la principal. Si una de las NIC virtuales existentes en la máquina virtual ya se ha establecido como principal, puede omitir este paso. En el siguiente ejemplo se da por supuesto que hay dos NIC virtuales en una máquina virtual y que se va a agregar la primera NIC (`[0]`) como la principal:
        
    ```powershell
    # List existing NICs on the VM and find which one is primary
    $vm.NetworkProfile.NetworkInterfaces
    
    # Set NIC 0 to be primary
    $vm.NetworkProfile.NetworkInterfaces[0].Primary = $true
    $vm.NetworkProfile.NetworkInterfaces[1].Primary = $false
    
    # Update the VM state in Azure
    Update-AzVM -VM $vm -ResourceGroupName "myResourceGroup"
    ```

4. Inicie la VM con [Start-AzVm](/powershell/module/az.compute/start-azvm):

    ```powershell
    Start-AzVM -ResourceGroupName "myResourceGroup" -Name "myVM"
    ```

5. Agregue rutas de tarjetas NIC secundarias al sistema operativo mediante los pasos que se indican en [Configuración del sistema operativo invitado para varias NIC](#configure-guest-os-for-multiple-nics).

## <a name="remove-a-nic-from-an-existing-vm"></a>Eliminación de una NIC de una máquina virtual existente
Para quitar una NIC virtual de una máquina virtual existente, se desasigna la máquina virtual, se quita la NIC virtual y, después, se inicia la máquina virtual.

1. Desasigne la VM con [Stop-AzVM](/powershell/module/az.compute/stop-azvm). En el ejemplo siguiente se desasigna la máquina virtual denominada *myVM* en *myResourceGroup*:

    ```powershell
    Stop-AzVM -Name "myVM" -ResourceGroupName "myResourceGroup"
    ```

2. Obtenga la configuración existente de la VM con [Get-AzVm](/powershell/module/az.compute/get-azvm). En el ejemplo siguiente se obtiene información de la máquina virtual denominada *myVM* en *myResourceGroup*:

    ```powershell
    $vm = Get-AzVm -Name "myVM" -ResourceGroupName "myResourceGroup"
    ```

3. Obtenga información sobre la eliminación de NIC con [Get-AzNetworkInterface](/powershell/module/az.network/get-aznetworkinterface). En el ejemplo siguiente se obtiene información sobre *myNic3*:

    ```powershell
    # List existing NICs on the VM if you need to determine NIC name
    $vm.NetworkProfile.NetworkInterfaces

    $nicId = (Get-AzNetworkInterface -ResourceGroupName "myResourceGroup" -Name "myNic3").Id   
    ```

4. Quite la NIC con [Remove-AzVMNetworkInterface](/powershell/module/az.compute/remove-azvmnetworkinterface) y, después, actualice la VM con [Update-AzVm](/powershell/module/az.compute/update-azvm). En el ejemplo siguiente se quita *myNic3* obtenida por `$nicId` en el paso anterior:

    ```powershell
    Remove-AzVMNetworkInterface -VM $vm -NetworkInterfaceIDs $nicId | `
        Update-AzVm -ResourceGroupName "myResourceGroup"
    ```   

5. Inicie la VM con [Start-AzVm](/powershell/module/az.compute/start-azvm):

    ```powershell
    Start-AzVM -Name "myVM" -ResourceGroupName "myResourceGroup"
    ```   

## <a name="create-multiple-nics-with-templates"></a>Crear varias NIC con plantillas
Las plantillas de Azure Resource Manager ofrecen una manera de crear varias instancias de un recurso durante la implementación; por ejemplo, se pueden crear varias NIC. Las plantillas de Resource Manager usan archivos JSON declarativos para definir el entorno. Para más información, vea [Información general de Azure Resource Manager](../../azure-resource-manager/management/overview.md). Puede usar *copy* para especificar el número de instancias que se crearán:

```json
"copy": {
    "name": "multiplenics",
    "count": "[parameters('count')]"
}
```

Para más información, vea [Creación de varias instancias mediante *copy*](../../azure-resource-manager/templates/copy-resources.md). 

También puede usar `copyIndex()` para anexar un número a un nombre de recurso. Después, puede crear *myNic1*, *MyNic2* y así sucesivamente. En el código siguiente se muestra un ejemplo de cómo anexar el valor de índice:

```json
"name": "[concat('myNic', copyIndex())]", 
```

Puede leer un ejemplo completo de [cómo crear varias NIC mediante plantillas de Resource Manager](../../virtual-network/template-samples.md).

Agregue rutas de tarjetas NIC secundarias al sistema operativo mediante los pasos que se indican en [Configuración del sistema operativo invitado para varias NIC](#configure-guest-os-for-multiple-nics).

## <a name="configure-guest-os-for-multiple-nics"></a>Configuración del sistema operativo invitado para varias NIC

Azure asigna una puerta de enlace predeterminada a la primera interfaz de red (principal) conectada a la máquina virtual. Azure no asigna una puerta de enlace predeterminada a las interfaces de red adicionales (secundarias) conectadas a una máquina virtual. Por lo tanto, de manera predeterminada, no es posible comunicarse con recursos externos a la subred en la que se encuentra una interfaz de red secundaria. Sin embargo, las interfaces de red secundarias pueden comunicarse con recursos externos a su subred, aunque los pasos necesarios para habilitar la comunicación son diferentes para los distintos sistemas operativos.

1. Desde un símbolo del sistema de Windows, ejecute el comando `route print`. Este comando devuelve una salida similar a la siguiente para una máquina virtual con dos interfaces de red conectadas:

    ```
    ===========================================================================
    Interface List
    3...00 0d 3a 10 92 ce ......Microsoft Hyper-V Network Adapter #3
    7...00 0d 3a 10 9b 2a ......Microsoft Hyper-V Network Adapter #4
    ===========================================================================
    ```
 
    En este ejemplo, **Microsoft Hyper-V Network Adapter #4** (interfaz 7) es la interfaz de red secundaria que no tiene una puerta de enlace predeterminada asignada.

2. Desde un símbolo del sistema, ejecute el comando `ipconfig` para ver la dirección IP asignada a la interfaz de red secundaria. En este ejemplo, la dirección IP 192.168.2.4 está asignada a la interfaz 7. No se devuelve ninguna dirección de puerta de enlace predeterminada para la interfaz de red secundaria.

3. Para que todo el tráfico destinado a direcciones que están fuera de la subred de la interfaz de red secundaria se enrute a la puerta de enlace de la subred, ejecute el siguiente comando:

    ```
    route add -p 0.0.0.0 MASK 0.0.0.0 192.168.2.1 METRIC 5015 IF 7
    ```

    La dirección de la puerta de enlace para la subred es la primera dirección IP (terminada en .1) del intervalo de direcciones definido para la subred. Si no quiere enrutar todo el tráfico fuera de la subred, puede agregar rutas individuales a destinos específicos en su lugar. Por ejemplo, si solo quiere enrutar el tráfico de la interfaz de red secundaria a la red 192.168.3.0, escriba el comando:

      ```
      route add -p 192.168.3.0 MASK 255.255.255.0 192.168.2.1 METRIC 5015 IF 7
      ```
  
4. Para confirmar la comunicación correcta con un recurso en la red 192.168.3.0, por ejemplo, escriba el siguiente comando para hacer ping a 192.168.3.4 mediante la interfaz 7 (192.168.2.4):

    ```
    ping 192.168.3.4 -S 192.168.2.4
    ```

    Puede que necesite abrir ICMP a través del firewall de Windows del dispositivo en el que está haciendo ping con el siguiente comando:
  
      ```
      netsh advfirewall firewall add rule name=Allow-ping protocol=icmpv4 dir=in action=allow
      ```
  
5. Para confirmar que la ruta agregada está en la tabla de rutas, escriba el comando `route print`, que devuelve una salida similar al texto siguiente:

    ```
    ===========================================================================
    Active Routes:
    Network Destination        Netmask          Gateway       Interface  Metric
              0.0.0.0          0.0.0.0      192.168.1.1      192.168.1.4     15
              0.0.0.0          0.0.0.0      192.168.2.1      192.168.2.4   5015
    ```

    La ruta que se muestra con *192.168.1.1* debajo de **Puerta de enlace** es la ruta que aparece de manera predeterminada para la interfaz de red principal. La ruta con *192.168.2.1* debajo de **Puerta de enlace** es la ruta agregada.

## <a name="next-steps"></a>Pasos siguientes
Revise [Tamaños de máquina virtual con Windows](../sizes.md) cuando esté intentando crear una máquina virtual que tiene varias NIC. Preste atención al número máximo de NIC que admite cada tamaño de máquina virtual. 