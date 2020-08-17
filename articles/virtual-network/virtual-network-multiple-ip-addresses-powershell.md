---
title: 'Varias direcciones IP para máquinas virtuales de Azure: PowerShell | Microsoft Docs'
description: Aprenda a asignar varias direcciones IP a una máquina virtual con PowerShell. | Resource Manager
services: virtual-network
documentationcenter: na
author: asudbring
manager: KumudD
ms.service: virtual-network
ms.subservice: ip-services
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/24/2017
ms.author: allensu
ms.openlocfilehash: 5cd050c88fbc954a211c3a75cdabcb557ae998c4
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87073932"
---
# <a name="assign-multiple-ip-addresses-to-virtual-machines-using-powershell"></a>Asignación de varias direcciones IP a máquinas virtuales mediante PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [virtual-network-multiple-ip-addresses-intro.md](../../includes/virtual-network-multiple-ip-addresses-intro.md)]

En este artículo se describe cómo crear una máquina virtual (VM) con el modelo de implementación de Azure Resource Manager mediante PowerShell. No se pueden asignar varias direcciones IP a los recursos creados mediante el modelo de implementación clásica. Para información acerca de los modelos de implementación de Azure, lea el artículo [Understand deployment models](../resource-manager-deployment-model.md) (Descripción de los modelos de implementación).

[!INCLUDE [virtual-network-multiple-ip-addresses-scenario.md](../../includes/virtual-network-multiple-ip-addresses-scenario.md)]

## <a name="create-a-vm-with-multiple-ip-addresses"></a><a name = "create"></a>Creación de una máquina virtual con varias direcciones IP

En los pasos siguientes se explica cómo crear una VM de ejemplo con varias direcciones IP, tal como se describe en el escenario. Cambie los valores de variable según sea necesario para la implementación.

1. Abra un símbolo del sistema de PowerShell y complete los pasos restantes de esta sección en una única sesión de PowerShell. Si todavía no tiene PowerShell instalado y configurado, complete los pasos del artículo [Cómo instalar y configurar Azure PowerShell](/powershell/azure/) .
2. Inicie sesión en la cuenta con el comando `Connect-AzAccount`.
3. Reemplace *myResourceGroup* y *westus* por un nombre y una ubicación de su elección. Cree un grupo de recursos. Un grupo de recursos es un contenedor lógico en el que se implementan y se administran los recursos de Azure.

   ```powershell
   $RgName   = "MyResourceGroup"
   $Location = "westus"

   New-AzResourceGroup `
   -Name $RgName `
   -Location $Location
   ```

4. Cree una red virtual y una subred en la misma ubicación del grupo de recursos:

   ```powershell

   # Create a subnet configuration
   $SubnetConfig = New-AzVirtualNetworkSubnetConfig `
   -Name MySubnet `
   -AddressPrefix 10.0.0.0/24

   # Create a virtual network
   $VNet = New-AzVirtualNetwork `
   -ResourceGroupName $RgName `
   -Location $Location `
   -Name MyVNet `
   -AddressPrefix 10.0.0.0/16 `
   -Subnet $subnetConfig

   # Get the subnet object
   $Subnet = Get-AzVirtualNetworkSubnetConfig -Name $SubnetConfig.Name -VirtualNetwork $VNet
   ```

5. Cree un grupo de seguridad de red (NSG) y una regla. El grupo de seguridad de red protege la máquina virtual con reglas de entrada y salida. En este caso, se crea una regla de entrada para el puerto 3389, que permite las conexiones entrantes al Escritorio remoto.

    ```powershell
    
    # Create an inbound network security group rule for port 3389

    $NSGRule = New-AzNetworkSecurityRuleConfig `
    -Name MyNsgRuleRDP `
    -Protocol Tcp `
    -Direction Inbound `
    -Priority 1000 `
    -SourceAddressPrefix * `
    -SourcePortRange * `
    -DestinationAddressPrefix * `
    -DestinationPortRange 3389 -Access Allow
    
    # Create a network security group
    $NSG = New-AzNetworkSecurityGroup `
    -ResourceGroupName $RgName `
    -Location $Location `
    -Name MyNetworkSecurityGroup `
    -SecurityRules $NSGRule
    ```

6. Defina la configuración de la dirección IP principal para la NIC. Cambie 10.0.0.4 a una dirección válida en la subred que creó, si no usó el valor definido anteriormente. Antes de asignar una dirección IP estática, se recomienda que primero confirme que no está en uso. Escriba el comando `Test-AzPrivateIPAddressAvailability -IPAddress 10.0.0.4 -VirtualNetwork $VNet`. Si la dirección está disponible, la salida devuelve *True*. Si no está disponible, la salida devuelve *False* y una lista de direcciones que están disponibles. 

    En los siguientes comandos, **reemplace \<replace-with-your-unique-name> por el nombre DNS único que se va a usar.** El nombre debe ser único en todas las direcciones IP públicas dentro de una región de Azure. Se trata de un parámetro opcional. Se puede quitar si solo desea conectarse a la máquina virtual con la dirección IP pública.

    ```powershell
    
    # Create a public IP address
    $PublicIP1 = New-AzPublicIpAddress `
    -Name "MyPublicIP1" `
    -ResourceGroupName $RgName `
    -Location $Location `
    -DomainNameLabel <replace-with-your-unique-name> `
    -AllocationMethod Static
        
    #Create an IP configuration with a static private IP address and assign the public IP address to it
    $IpConfigName1 = "IPConfig-1"
    $IpConfig1     = New-AzNetworkInterfaceIpConfig `
    -Name $IpConfigName1 `
    -Subnet $Subnet `
    -PrivateIpAddress 10.0.0.4 `
    -PublicIpAddress $PublicIP1 `
    -Primary
    ```

    Cuando se asignan varias configuraciones de dirección IP a una NIC, se debe asignar una como la *principal*.

    > [!NOTE]
    > Las direcciones IP públicas tienen un precio simbólico. Para más información sobre los precios de las direcciones IP, lea la página [Precios de las direcciones IP](https://azure.microsoft.com/pricing/details/ip-addresses) . Existe un límite para el número de direcciones IP públicas que pueden usarse dentro de una suscripción. Para más información sobre los límites, lea el artículo sobre los [límites de Azure](../azure-resource-manager/management/azure-subscription-service-limits.md#networking-limits).

7. Defina las configuraciones de la dirección IP secundaria para la NIC. Puede agregar o quitar las configuraciones que sean necesarias. Cada configuración de dirección IP debe tener asignada una dirección IP privada. Cada configuración puede tener asignada opcionalmente una dirección IP pública.

    ```powershell
    
    # Create a public IP address
    $PublicIP2 = New-AzPublicIpAddress `
    -Name "MyPublicIP2" `
    -ResourceGroupName $RgName `
    -Location $Location `
    -AllocationMethod Static
        
    #Create an IP configuration with a static private IP address and assign the public IP address to it
    $IpConfigName2 = "IPConfig-2"
    $IpConfig2     = New-AzNetworkInterfaceIpConfig `
    -Name $IpConfigName2 `
    -Subnet $Subnet `
    -PrivateIpAddress 10.0.0.5 `
    -PublicIpAddress $PublicIP2
        
    $IpConfigName3 = "IpConfig-3"
    $IpConfig3 = New-AzNetworkInterfaceIpConfig `
    -Name $IPConfigName3 `
    -Subnet $Subnet `
    -PrivateIpAddress 10.0.0.6
    ```

8. Cree la NIC y asocie las tres configuraciones de dirección IP con ella:

   ```powershell
   $NIC = New-AzNetworkInterface `
   -Name MyNIC `
   -ResourceGroupName $RgName `
   -Location $Location `
   -NetworkSecurityGroupId $NSG.Id `
   -IpConfiguration $IpConfig1,$IpConfig2,$IpConfig3
   ```

   >[!NOTE]
   >Aunque en este artículo todas las configuraciones están asignadas a una NIC, puede asignar varias configuraciones de dirección IP a cada NIC conectada con la máquina virtual. Para aprender a crear una VM con varias NIC, lea el artículo [Implementación de máquinas virtuales con varias NIC mediante PowerShell](../virtual-machines/windows/multiple-nics.md).

9. Escriba los comandos siguientes para crear la máquina virtual:

    ```powershell
    
    # Define a credential object. When you run these commands, you're prompted to enter a username and password for the VM you're creating.
    $cred = Get-Credential
    
    # Create a virtual machine configuration
    $VmConfig = New-AzVMConfig `
    -VMName MyVM `
    -VMSize Standard_DS1_v2 | `
    Set-AzVMOperatingSystem -Windows `
    -ComputerName MyVM `
    -Credential $cred | `
    Set-AzVMSourceImage `
    -PublisherName MicrosoftWindowsServer `
    -Offer WindowsServer `
    -Skus 2016-Datacenter `
    -Version latest | `
    Add-AzVMNetworkInterface `
    -Id $NIC.Id
    
    # Create the VM
    New-AzVM `
    -ResourceGroupName $RgName `
    -Location $Location `
    -VM $VmConfig
    ```

10. Agregue al sistema operativo de la máquina virtual la dirección IP privada siguiendo las instrucciones de la sección [Incorporación de direcciones IP a un sistema operativo de la VM](#os-config) de este artículo. No agregue las direcciones IP públicas al sistema operativo.

## <a name="add-ip-addresses-to-a-vm"></a><a name="add"></a>Incorporación de direcciones IP a una VM

Puede agregar direcciones IP públicas y privadas a una interfaz de red de Azure mediante los pasos que se indican a continuación. En los ejemplos de las secciones siguientes se da por sentado que ya tiene una máquina virtual con las tres configuraciones de IP descritas en el [escenario](#scenario) de este artículo, pero no es necesario que lo haga.

1. Abra un símbolo del sistema de PowerShell y complete los pasos restantes de esta sección en una única sesión de PowerShell. Si todavía no tiene PowerShell instalado y configurado, complete los pasos del artículo [Cómo instalar y configurar Azure PowerShell](/powershell/azure/) .
2. Cambie los "valores" de los elementos $Variables siguientes al nombre de la NIC a la que desee agregar una dirección IP y al grupo de recursos y la ubicación donde existe la NIC:

   ```powershell
   $NicName  = "MyNIC"
   $RgName   = "MyResourceGroup"
   $Location = "westus"
   ```

   Si no conoce el nombre de la NIC que desea cambiar, escriba los siguientes comandos y cambie los valores de las anteriores variables:

   ```powershell
   Get-AzNetworkInterface | Format-Table Name, ResourceGroupName, Location
   ```

3. Cree una variable y establézcala en la NIC existente con el siguiente comando:

   ```powershell
   $MyNIC = Get-AzNetworkInterface -Name $NicName -ResourceGroupName $RgName
   ```

4. En los siguientes comandos, cambie *MyVNet* y *MySubnet* a los nombres de la red virtual y la subred a las que está conectada la NIC. Escriba los comandos para recuperar los objetos de red virtual y subred a las que está conectada la NIC:

   ```powershell
   $MyVNet = Get-AzVirtualnetwork -Name MyVNet -ResourceGroupName $RgName
   $Subnet = $MyVnet.Subnets | Where-Object { $_.Name -eq "MySubnet" }
   ```

   Si no conoce el nombre de red virtual o la subred de a la que está conectada la NIC, escriba el siguiente comando:

   ```powershell
   $MyNIC.IpConfigurations
   ```

   En la salida, busque un texto similar a la salida del ejemplo siguiente:

   ```
   "Id": "/subscriptions/[Id]/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/MyVNet/subnets/MySubnet"
   ```

    En esta salida, *MyVnet* es la red virtual y *MySubnet* es la subred a la que está conectada la NIC.

5. Complete los pasos de una de las secciones siguientes, según sus requisitos:

   **Incorporación de una dirección IP privada**

   Para agregar una dirección IP privada a una NIC, debe crear una configuración de IP. El siguiente comando crea una configuración con una dirección IP estática de 10.0.0.7. Al especificar una dirección IP estática, debe ser una dirección no utilizada para la subred. Se recomienda que pruebe en primer lugar la dirección para asegurarse de que está disponible escribiendo el comando `Test-AzPrivateIPAddressAvailability -IPAddress 10.0.0.7 -VirtualNetwork $myVnet`. Si la dirección IP está disponible, la salida devuelve *True*. Si no está disponible, la salida devuelve *False* y una lista de direcciones que están disponibles.

   ```powershell
   Add-AzNetworkInterfaceIpConfig -Name IPConfig-4 -NetworkInterface `
   $MyNIC -Subnet $Subnet -PrivateIpAddress 10.0.0.7
   ```

   Cree tantas configuraciones como sea necesario mediante nombres de configuración únicos y direcciones IP privadas (para las configuraciones con direcciones IP estáticas).

   Agregue al sistema operativo de la máquina virtual la dirección IP privada siguiendo las instrucciones de la sección [Incorporación de direcciones IP a un sistema operativo de la VM](#os-config) de este artículo.

   **Incorporación de una dirección IP pública**

   Se agrega una dirección IP pública mediante la asociación de un recurso de dirección IP pública a una nueva configuración de IP o una configuración de IP existente. Complete los pasos de una de las secciones siguientes, según sea preciso.

   > [!NOTE]
   > Las direcciones IP públicas tienen un precio simbólico. Para más información sobre los precios de las direcciones IP, lea la página [Precios de las direcciones IP](https://azure.microsoft.com/pricing/details/ip-addresses) . Existe un límite para el número de direcciones IP públicas que pueden usarse dentro de una suscripción. Para más información sobre los límites, lea el artículo sobre los [límites de Azure](../azure-resource-manager/management/azure-subscription-service-limits.md#networking-limits).
   >

   **Asociación del recurso de dirección IP pública a una nueva configuración de IP**

   Siempre que agregue una dirección IP pública en una nueva configuración de IP, también debe agregar una dirección IP privada, porque todas las configuraciones de IP deben tener una dirección IP privada. Puede agregar un recurso de dirección IP pública existente o crear uno nuevo. Para crear uno nuevo, escriba el comando siguiente:

   ```powershell
   $myPublicIp3 = New-AzPublicIpAddress `
   -Name "myPublicIp3" `
   -ResourceGroupName $RgName `
   -Location $Location `
   -AllocationMethod Static
   ```

   Para crear una nueva configuración de IP con una dirección IP privada estática y el recurso de dirección IP pública *myPublicIp3*, escriba el comando siguiente:

   ```powershell
   Add-AzNetworkInterfaceIpConfig `
   -Name IPConfig-4 `
   -NetworkInterface $myNIC `
   -Subnet $Subnet `
   -PrivateIpAddress 10.0.0.7 `
   -PublicIpAddress $myPublicIp3
   ```

   **Asociación del recurso de dirección IP pública a una configuración de IP existente**

   Solo se puede asociar un recurso de dirección IP pública a una configuración de IP que ya no tiene asociado uno. Puede determinar si una configuración de IP tiene una dirección IP pública asociada escribiendo el comando siguiente:

   ```powershell
   $MyNIC.IpConfigurations | Format-Table Name, PrivateIPAddress, PublicIPAddress, Primary
   ```

   Verá un resultado similar al siguiente:

   ```
   Name       PrivateIpAddress PublicIpAddress                                           Primary

   IPConfig-1 10.0.0.4         Microsoft.Azure.Commands.Network.Models.PSPublicIpAddress    True
   IPConfig-2 10.0.0.5         Microsoft.Azure.Commands.Network.Models.PSPublicIpAddress   False
   IpConfig-3 10.0.0.6                                                                     False
   ```

   Puesto que la columna **PublicIpAddress** para *IpConfig-3* está en blanco, ningún recurso de dirección IP pública está asociado actualmente a ella. Puede agregar un recurso de dirección IP pública existente a IpConfig-3 o escribir el siguiente comando para crear uno:

   ```powershell
   $MyPublicIp3 = New-AzPublicIpAddress `
   -Name "MyPublicIp3" `
   -ResourceGroupName $RgName `
   -Location $Location -AllocationMethod Static
   ```

   Escriba el siguiente comando para asociar el recurso de dirección IP pública a la configuración de IP existente llamada *IpConfig-3*:

   ```powershell
   Set-AzNetworkInterfaceIpConfig `
   -Name IpConfig-3 `
   -NetworkInterface $mynic `
   -Subnet $Subnet `
   -PublicIpAddress $myPublicIp3
   ```

6. Establezca la NIC con la nueva configuración de IP escribiendo el siguiente comando:

   ```powershell
   Set-AzNetworkInterface -NetworkInterface $MyNIC
   ```

7. Vea las direcciones IP privadas y los recursos de dirección IP pública asignados a la NIC escribiendo el siguiente comando:

   ```powershell
   $MyNIC.IpConfigurations | Format-Table Name, PrivateIPAddress, PublicIPAddress, Primary
   ```

8. Agregue al sistema operativo de la máquina virtual la dirección IP privada siguiendo las instrucciones de la sección [Incorporación de direcciones IP a un sistema operativo de la VM](#os-config) de este artículo. No agregue la dirección IP pública al sistema operativo.vo.

[!INCLUDE [virtual-network-multiple-ip-addresses-os-config.md](../../includes/virtual-network-multiple-ip-addresses-os-config.md)]