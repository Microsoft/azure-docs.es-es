---
title: 'Creación de un equilibrador de carga público con IPv6: CLI de Azure'
titleSuffix: Azure Load Balancer
description: Con esta ruta de aprendizaje, empiece a crear un equilibrador de carga público con IPv6 mediante la CLI de Azure.
services: load-balancer
documentationcenter: na
author: asudbring
keywords: IPv6, Azure Load Balancer, pila doble, dirección ip pública, ipv6 nativo, móvil, iot
ms.service: load-balancer
ms.devlang: na
ms.topic: how-to
ms.custom: seodec18
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/25/2018
ms.author: allensu
ms.openlocfilehash: edc17b9636792ce00458716e3461077fa689b3ed
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87001580"
---
# <a name="create-a-public-load-balancer-with-ipv6-using-azure-cli"></a>Creación de un equilibrador de carga público con IPv6 mediante la CLI de Azure

>[!NOTE] 
>En este artículo se describe una característica de IPv6 introductoria que permite que los equilibradores de carga básicos proporcionen conectividad IPv4 e IPv6. Ahora hay disponible conectividad IPv6 completa con [IPv6 para redes virtuales de Azure](../virtual-network/ipv6-overview.md) que integra conectividad IPv6 con las redes virtuales e incluye características clave como las reglas de grupo de seguridad de red IPv6, el enrutamiento definido por el usuario IPv6, el equilibrio de carga de IPv6 básico y estándar, y mucho más.  IPv6 para redes virtuales de Azure es el estándar recomendado para las aplicaciones IPv6 en Azure. Vea [IPv6 para la implementación de PowerShell de red virtual de Azure](../virtual-network/virtual-network-ipv4-ipv6-dual-stack-standard-load-balancer-powershell.md). 

Azure Load Balancer es un equilibrador de carga de nivel 4 (TCP y UDP) Los equilibradores de carga proporcionan una alta disponibilidad mediante la distribución del tráfico entrante entre las instancias de servicio correctas de los servicios en la nube o las máquinas virtuales de un conjunto de equilibradores de carga. Los equilibradores de carga también pueden prestar estos servicios en varios puertos, varias direcciones IP o ambos.

## <a name="example-deployment-scenario"></a>Escenario de implementación de ejemplo

En el siguiente diagrama se ilustra la solución de equilibrio de carga que se implementa mediante la plantilla de ejemplo descrita en este artículo.

![Escenario del equilibrador de carga](./media/load-balancer-ipv6-internet-cli/lb-ipv6-scenario-cli.png)

En este escenario puede crear los siguientes recursos de Azure:

* Dos máquinas virtuales (VM)
* Una interfaz de red virtual para cada máquina virtual con las direcciones IPv4 e IPv6 asignadas
* Un equilibrador de carga público con una dirección IP pública IPv4 e IPv6
* Un conjunto de disponibilidad con las dos máquinas virtuales
* Dos reglas de equilibrio de carga para asignar las VIP públicas a los puntos de conexión privados

## <a name="deploy-the-solution-by-using-azure-cli"></a>Implementación de la solución mediante la CLI de Azure

Los pasos siguientes muestran cómo crear un equilibrador de carga público mediante la CLI de Azure. Con la interfaz de la línea de comandos, cree y configure cada objeto de forma individual y, después, únalos para crear un recurso.

Para implementar un equilibrador de carga, cree y configure los objetos siguientes:

* **Configuración de direcciones IP de front-end**: contiene direcciones IP públicas para el tráfico de red entrante.
* **Grupo de direcciones de back-end**: contiene interfaces de red (NIC) para que las máquinas virtuales reciban tráfico de red del equilibrador de carga.
* **Reglas de equilibrio de carga**: contiene reglas que asignan un puerto público en el equilibrador de carga a un puerto del grupo de direcciones de back-end.
* **Reglas NAT de entrada**: contiene reglas de traslación de direcciones de red (NAT) que asignan un puerto público en el equilibrador de carga a un puerto de una máquina virtual específica en el grupo de direcciones de back-end.
* **Sondeos**: contiene los sondeos de estado que se usan para comprobar la disponibilidad de las instancias de las máquinas virtuales del grupo de direcciones de back-end.

## <a name="set-up-azure-cli"></a>Configuración de la CLI de Azure

En este ejemplo, puede ejecutar las herramientas de la CLI de Azure en una ventana de comandos de PowerShell. Para mejorar la legibilidad y la reutilización, puede usar las funcionalidades de scripting de PowerShell en lugar de los cmdlets de Azure PowerShell.

1. [Instale y configure la CLI de Azure ](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) siguiendo los pasos que se describen en el artículo vinculado e inicie sesión en la cuenta de Azure.

2. Configure variables de PowerShell para usarlas con los comandos de la CLI de Azure:

    ```powershell
    $subscriptionid = "########-####-####-####-############"  # enter subscription id
    $location = "southcentralus"
    $rgName = "pscontosorg1southctrlus09152016"
    $vnetName = "contosoIPv4Vnet"
    $vnetPrefix = "10.0.0.0/16"
    $subnet1Name = "clicontosoIPv4Subnet1"
    $subnet1Prefix = "10.0.0.0/24"
    $subnet2Name = "clicontosoIPv4Subnet2"
    $subnet2Prefix = "10.0.1.0/24"
    $dnsLabel = "contoso09152016"
    $lbName = "myIPv4IPv6Lb"
    ```

## <a name="create-a-resource-group-a-load-balancer-a-virtual-network-and-subnets"></a>Creación de un grupo de recursos, un equilibrador de carga, una red virtual y subredes

1. Cree un grupo de recursos:

    ```azurecli
    az group create --name $rgName --location $location
    ```

2. Cree un equilibrador de carga:

    ```azurecli
    $lb = az network lb create --resource-group $rgname --location $location --name $lbName
    ```

3. Cree una red virtual:

    ```azurecli
    $vnet = az network vnet create  --resource-group $rgname --name $vnetName --location $location --address-prefixes $vnetPrefix
    ```

4. En esta red virtual, cree dos subredes:

    ```azurecli
    $subnet1 = az network vnet subnet create --resource-group $rgname --name $subnet1Name --address-prefix $subnet1Prefix --vnet-name $vnetName
    $subnet2 = az network vnet subnet create --resource-group $rgname --name $subnet2Name --address-prefix $subnet2Prefix --vnet-name $vnetName
    ```

## <a name="create-public-ip-addresses-for-the-front-end-pool"></a>Creación de direcciones IP públicas para el grupo de servidores front-end

1. Configure las variables de PowerShell:

    ```powershell
    $publicIpv4Name = "myIPv4Vip"
    $publicIpv6Name = "myIPv6Vip"
    ```

2. Cree una dirección IP pública para el grupo de servidores front-end:

    ```azurecli
    $publicipV4 = az network public-ip create --resource-group $rgname --name $publicIpv4Name --location $location --version IPv4 --allocation-method Dynamic --dns-name $dnsLabel
    $publicipV6 = az network public-ip create --resource-group $rgname --name $publicIpv6Name --location $location --version IPv6 --allocation-method Dynamic --dns-name $dnsLabel
    ```

    > [!IMPORTANT]
    > El equilibrador de carga usa la etiqueta de dominio de la dirección IP pública como nombre de dominio completo (FQDN). Esto representa un cambio en la implementación clásica, que usa el nombre del servicio en la nube como el FQDN del equilibrador de carga.
    >
    > En este ejemplo, el FQDN es *contoso09152016.southcentralus.cloudapp.azure.com*.

## <a name="create-front-end-and-back-end-pools"></a>Creación de grupos de servidores front-end y back-end

En esta sección, puede crear los siguientes grupos de direcciones IP:
* El grupo de direcciones IP de front-end que recibe el tráfico de red entrante en el equilibrador de carga.
* El grupo de direcciones IP de back-end al que el grupo de front-end envía el tráfico de red de carga equilibrada.

1. Configure las variables de PowerShell:

    ```powershell
    $frontendV4Name = "FrontendVipIPv4"
    $frontendV6Name = "FrontendVipIPv6"
    $backendAddressPoolV4Name = "BackendPoolIPv4"
    $backendAddressPoolV6Name = "BackendPoolIPv6"
    ```

2. Cree un grupo de direcciones IP de front-end y asócielo con la dirección IP pública que creó en el paso anterior y el equilibrador de carga.

    ```azurecli
    $frontendV4 = az network lb frontend-ip create --resource-group $rgname --name $frontendV4Name --public-ip-address $publicIpv4Name --lb-name $lbName
    $frontendV6 = az network lb frontend-ip create --resource-group $rgname --name $frontendV6Name --public-ip-address $publicIpv6Name --lb-name $lbName
    $backendAddressPoolV4 = az network lb address-pool create --resource-group $rgname --name $backendAddressPoolV4Name --lb-name $lbName
    $backendAddressPoolV6 = az network lb address-pool create --resource-group $rgname --name $backendAddressPoolV6Name --lb-name $lbName
    ```

## <a name="create-the-probe-nat-rules-and-load-balancer-rules"></a>Creación del sondeo, las reglas NAT y las reglas del equilibrador de carga

En este ejemplo se crean los siguientes elementos:

* Una regla de sondeo para comprobar la conectividad con el puerto TCP 80.
* Una regla NAT para trasladar todo el tráfico entrante del puerto 3389 al puerto 3389 para RDP.\*
* Una regla NAT para trasladar todo el tráfico entrante del puerto 3391 al puerto 3389 para el protocolo de escritorio remoto (RDP).\*
* Una regla de equilibrador de carga para equilibrar todo el tráfico entrante del puerto 80 al puerto 80 en las direcciones del grupo de back-end.

\* Las reglas NAT están asociadas a una instancia de máquina virtual específica detrás del equilibrador de carga. El tráfico de red que llega al puerto 3389 se envía a la máquina virtual específica y al puerto que está asociado a la regla NAT. Debe especificar un protocolo (UDP o TCP) para una regla NAT. No puede asignar ambos protocolos al mismo puerto.

1. Configure las variables de PowerShell:

    ```powershell
    $probeV4V6Name = "ProbeForIPv4AndIPv6"
    $natRule1V4Name = "NatRule-For-Rdp-VM1"
    $natRule2V4Name = "NatRule-For-Rdp-VM2"
    $lbRule1V4Name = "LBRuleForIPv4-Port80"
    $lbRule1V6Name = "LBRuleForIPv6-Port80"
    ```

2. Cree el sondeo.

    En el siguiente ejemplo se crea un sondeo TCP que comprueba la conectividad con el puerto TCP 80 de back-end cada 15 segundos. Este marcará el recurso de back-end como no disponible si se producen dos errores consecutivos.

    ```azurecli
    $probeV4V6 = az network lb probe create --resource-group $rgname --name $probeV4V6Name --protocol tcp --port 80 --interval 15 --threshold 2 --lb-name $lbName
    ```

3. Cree reglas NAT de entrada que permitan conexiones RDP a los recursos de back-end:

    ```azurecli
    $inboundNatRuleRdp1 = az network lb inbound-nat-rule create --resource-group $rgname --name $natRule1V4Name --frontend-ip-name $frontendV4Name --protocol Tcp --frontend-port 3389 --backend-port 3389 --lb-name $lbName
    $inboundNatRuleRdp2 = az network lb inbound-nat-rule create --resource-group $rgname --name $natRule2V4Name --frontend-ip-name $frontendV4Name --protocol Tcp --frontend-port 3391 --backend-port 3389 --lb-name $lbName
    ```

4. Cree reglas de equilibrador de carga que envíen tráfico a distintos puertos de back-end dependiendo de qué front-end recibió la solicitud.

    ```azurecli
    $lbruleIPv4 = az network lb rule create --resource-group $rgname --name $lbRule1V4Name --frontend-ip-name $frontendV4Name --backend-pool-name $backendAddressPoolV4Name --probe-name $probeV4V6Name --protocol Tcp --frontend-port 80 --backend-port 80 --lb-name $lbName
    $lbruleIPv6 = az network lb rule create --resource-group $rgname --name $lbRule1V6Name --frontend-ip-name $frontendV6Name --backend-pool-name $backendAddressPoolV6Name --probe-name $probeV4V6Name --protocol Tcp --frontend-port 80 --backend-port 8080 --lb-name $lbName
    ```

5. Compruebe la configuración:

    ```azurecli
    az network lb show --resource-group $rgName --name $lbName
    ```

    Resultado esperado:

    ```output
    info:    Executing command network lb show
    info:    Looking up the load balancer "myIPv4IPv6Lb"
    data:    Id                              : /subscriptions/########-####-####-####-############/resourceGroups/pscontosorg1southctrlus09152016/providers/Microsoft.Network/loadBalancers/myIPv4IPv6Lb
    data:    Name                            : myIPv4IPv6Lb
    data:    Type                            : Microsoft.Network/loadBalancers
    data:    Location                        : southcentralus
    data:    Provisioning state              : Succeeded
    data:
    data:    Frontend IP configurations:
    data:    Name             Provisioning state  Private IP allocation  Private IP   Subnet  Public IP
    data:    ---------------  ------------------  ---------------------  -----------  ------  ---------
    data:    FrontendVipIPv4  Succeeded           Dynamic                                     myIPv4Vip
    data:    FrontendVipIPv6  Succeeded           Dynamic                                     myIPv6Vip
    data:
    data:    Probes:
    data:    Name                 Provisioning state  Protocol  Port  Path  Interval  Count
    data:    -------------------  ------------------  --------  ----  ----  --------  -----
    data:    ProbeForIPv4AndIPv6  Succeeded           Tcp       80          15        2
    data:
    data:    Backend Address Pools:
    data:    Name             Provisioning state
    data:    ---------------  ------------------
    data:    BackendPoolIPv4  Succeeded
    data:    BackendPoolIPv6  Succeeded
    data:
    data:    Load Balancing Rules:
    data:    Name                  Provisioning state  Load distribution  Protocol  Frontend port  Backend port  Enable floating IP  Idle timeout in minutes
    data:    --------------------  ------------------  -----------------  --------  -------------  ------------  ------------------  -----------------------
    data:    LBRuleForIPv4-Port80  Succeeded           Default            Tcp       80             80            false               4
    data:    LBRuleForIPv6-Port80  Succeeded           Default            Tcp       80             8080          false               4
    data:
    data:    Inbound NAT Rules:
    data:    Name                 Provisioning state  Protocol  Frontend port  Backend port  Enable floating IP  Idle timeout in minutes
    data:    -------------------  ------------------  --------  -------------  ------------  ------------------  -----------------------
    data:    NatRule-For-Rdp-VM1  Succeeded           Tcp       3389           3389          false               4
    data:    NatRule-For-Rdp-VM2  Succeeded           Tcp       3391           3389          false               4
    info:    network lb show
    ```

## <a name="create-nics"></a>Creación de tarjetas NIC

Cree tarjetas NIC y asócielas a reglas NAT, reglas de equilibrador de carga y sondeos.

1. Configure las variables de PowerShell:

    ```powershell
    $nic1Name = "myIPv4IPv6Nic1"
    $nic2Name = "myIPv4IPv6Nic2"
    $subnet1Id = "/subscriptions/$subscriptionid/resourceGroups/$rgName/providers/Microsoft.Network/VirtualNetworks/$vnetName/subnets/$subnet1Name"
    $subnet2Id = "/subscriptions/$subscriptionid/resourceGroups/$rgName/providers/Microsoft.Network/VirtualNetworks/$vnetName/subnets/$subnet2Name"
    $backendAddressPoolV4Id = "/subscriptions/$subscriptionid/resourceGroups/$rgname/providers/Microsoft.Network/loadbalancers/$lbName/backendAddressPools/$backendAddressPoolV4Name"
    $backendAddressPoolV6Id = "/subscriptions/$subscriptionid/resourceGroups/$rgname/providers/Microsoft.Network/loadbalancers/$lbName/backendAddressPools/$backendAddressPoolV6Name"
    $natRule1V4Id = "/subscriptions/$subscriptionid/resourceGroups/$rgname/providers/Microsoft.Network/loadbalancers/$lbName/inboundNatRules/$natRule1V4Name"
    $natRule2V4Id = "/subscriptions/$subscriptionid/resourceGroups/$rgname/providers/Microsoft.Network/loadbalancers/$lbName/inboundNatRules/$natRule2V4Name"
    ```

2. Cree una NIC para cada back-end y agregue una configuración de IPv6:

    ```azurecli
    $nic1 = az network nic create --name $nic1Name --resource-group $rgname --location $location --private-ip-address-version "IPv4" --subnet $subnet1Id --lb-address-pools $backendAddressPoolV4Id --lb-inbound-nat-rules $natRule1V4Id
    $nic1IPv6 = az network nic ip-config create --resource-group $rgname --name "IPv6IPConfig" --private-ip-address-version "IPv6" --lb-address-pools $backendAddressPoolV6Id --nic-name $nic1Name

    $nic2 = az network nic create --name $nic2Name --resource-group $rgname --location $location --private-ip-address-version "IPv4" --subnet $subnet2Id --lb-address-pools $backendAddressPoolV4Id --lb-inbound-nat-rules $natRule2V4Id
    $nic2IPv6 = az network nic ip-config create --resource-group $rgname --name "IPv6IPConfig" --private-ip-address-version "IPv6" --lb-address-pools $backendAddressPoolV6Id --nic-name $nic2Name
    ```

## <a name="create-the-back-end-vm-resources-and-attach-each-nic"></a>Creación de los recursos de la máquina virtual de back-end y conexión de cada NIC

Para crear máquinas virtuales, debe tener una cuenta de almacenamiento. Para el equilibrio de carga, es necesario que las máquinas virtuales formen parte de un conjunto de disponibilidad. Para más información sobre cómo crear máquinas virtuales, consulte [Creación de una máquina virtual de Azure con PowerShell](../virtual-machines/windows/quick-create-powershell.md?toc=%2fazure%2fload-balancer%2ftoc.json).

1. Configure las variables de PowerShell:

    ```powershell
    $availabilitySetName = "myIPv4IPv6AvailabilitySet"
    $vm1Name = "myIPv4IPv6VM1"
    $vm2Name = "myIPv4IPv6VM2"
    $nic1Id = "/subscriptions/$subscriptionid/resourceGroups/$rgname/providers/Microsoft.Network/networkInterfaces/$nic1Name"
    $nic2Id = "/subscriptions/$subscriptionid/resourceGroups/$rgname/providers/Microsoft.Network/networkInterfaces/$nic2Name"
    $imageurn = "MicrosoftWindowsServer:WindowsServer:2012-R2-Datacenter:latest"
    $vmUserName = "vmUser"
    $mySecurePassword = "PlainTextPassword*1"
    ```

    > [!WARNING]
    > En este ejemplo se usa el nombre de usuario y la contraseña de las máquinas virtuales en texto no cifrado. Tome las precauciones necesarias cuando use estas credenciales en texto no cifrado. Para ver un método más seguro de administración de credenciales en PowerShell, consulte el cmdlet [`Get-Credential`](https://technet.microsoft.com/library/hh849815.aspx).

2. Cree el conjunto de disponibilidad:

    ```azurecli
    $availabilitySet = az vm availability-set create --name $availabilitySetName --resource-group $rgName --location $location
    ```

3. Cree las máquinas virtuales con las NIC asociadas:

    ```azurecli
    az vm create --resource-group $rgname --name $vm1Name --image $imageurn --admin-username $vmUserName --admin-password $mySecurePassword --nics $nic1Id --location $location --availability-set $availabilitySetName --size "Standard_A1" 

    az vm create --resource-group $rgname --name $vm2Name --image $imageurn --admin-username $vmUserName --admin-password $mySecurePassword --nics $nic2Id --location $location --availability-set $availabilitySetName --size "Standard_A1" 
    ```


