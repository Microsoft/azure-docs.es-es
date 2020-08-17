---
title: Configuración de la conectividad desde la red virtual hasta SAP HANA en Azure (instancias grandes) | Microsoft Docs
description: Configuración de la conectividad desde la red virtual para usar SAP HANA en Azure (instancias grandes).
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 05/25/2019
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 8c7e8d4875a8bf3f53ac536ae95ac7499a74d45c
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87082161"
---
# <a name="connect-a-virtual-network-to-hana-large-instances"></a>Conexión de una red virtual a instancias grandes de HANA

Tras crear una red virtual de Azure, puede conectarla a SAP HANA en instancias grandes de Azure. Cree una puerta de enlace de ExpressRoute de Azure en la red virtual. Esta puerta de enlace le permite vincular la red virtual al circuito ExpressRoute que se conecta al inquilino del cliente en el sello de instancia grande de HANA.

> [!NOTE] 
> Este paso puede tardar hasta 30 minutos en completarse. La nueva puerta de enlace se crea en la suscripción de Azure designada y, después, se conecta a la red virtual de Azure especificada.

[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]

Si ya existe una puerta de enlace, compruebe si se trata de una puerta de enlace de ExpressRoute o no. Si no es una puerta de enlace de ExpressRoute, elimínela y vuelva a crearla como una puerta de enlace de ExpressRoute. Si ya hay una puerta de enlace de ExpressRoute establecida, consulte la sección siguiente de este artículo, "Vinculación de redes virtuales". 

- Use [Azure Portal](https://portal.azure.com/) o PowerShell para crear una puerta de enlace VPN de ExpressRoute conectada a la red virtual.
  - Si usa Azure Portal, agregue una nueva **puerta de enlace de red virtual** y seleccione **ExpressRoute** como tipo de puerta de enlace.
  - Si usa PowerShell, descargue y use la versión más reciente del [SDK de Azure PowerShell](https://azure.microsoft.com/downloads/). 
 
Los siguientes comandos crean una puerta de enlace de ExpressRoute. Los textos precedidos por un signo _$_ son variables definidas por el usuario que se deben actualizar con la información específica.

```powershell
# These Values should already exist, update to match your environment
$myAzureRegion = "eastus"
$myGroupName = "SAP-East-Coast"
$myVNetName = "VNet01"

# These values are used to create the gateway, update for how you wish the GW components to be named
$myGWName = "VNet01GW"
$myGWConfig = "VNet01GWConfig"
$myGWPIPName = "VNet01GWPIP"
$myGWSku = "HighPerformance" # Supported values for HANA large instances are: HighPerformance or UltraPerformance

# These Commands create the Public IP and ExpressRoute Gateway
$vnet = Get-AzVirtualNetwork -Name $myVNetName -ResourceGroupName $myGroupName
$subnet = Get-AzVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -VirtualNetwork $vnet
New-AzPublicIpAddress -Name $myGWPIPName -ResourceGroupName $myGroupName `
-Location $myAzureRegion -AllocationMethod Dynamic
$gwpip = Get-AzPublicIpAddress -Name $myGWPIPName -ResourceGroupName $myGroupName
$gwipconfig = New-AzVirtualNetworkGatewayIpConfig -Name $myGWConfig -SubnetId $subnet.Id `
-PublicIpAddressId $gwpip.Id

New-AzVirtualNetworkGateway -Name $myGWName -ResourceGroupName $myGroupName -Location $myAzureRegion `
-IpConfigurations $gwipconfig -GatewayType ExpressRoute `
-GatewaySku $myGWSku -VpnType PolicyBased -EnableBgp $true
```

En este ejemplo, se usó la SKU de puerta de enlace HighPerformance. Puede elegir HighPerformance o UltraPerformance, las únicas SKU de puerta de enlace que son compatibles con SAP HANA en Azure (instancias grandes).

> [!IMPORTANT]
> Para las instancias grandes de HANA de las SKU de la clase Tipo II, debe usar la SKU de puerta de enlace UltraPerformance.

## <a name="link-virtual-networks"></a>Vinculación de redes virtuales

La red virtual de Azure ahora tiene una puerta de enlace de ExpressRoute. Use la información de autorización que facilita Microsoft para conectar la puerta de enlace de ExpressRoute al circuito de ExpressRoute de SAP HANA (Instancias grandes). Puede usar Azure Portal o PowerShell para realizar la conexión. Las instrucciones de PowerShell son las siguientes. 

Ejecute los siguientes comandos para cada puerta de enlace de ExpressRoute con un AuthGUID diferente para cada conexión. Las dos primeras entradas mostradas en el siguiente script proceden de la información proporcionada por Microsoft. Además, el AuthGUID es específico de cada red virtual y su puerta de enlace. Si quiere agregar otra red virtual de Azure, tiene que obtener otro AuthID para el circuito ExpressRoute que conecta instancias grandes de HANA con Azure desde Microsoft. 

```powershell
# Populate with information provided by Microsoft Onboarding team
$PeerID = "/subscriptions/9cb43037-9195-4420-a798-f87681a0e380/resourceGroups/Customer-USE-Circuits/providers/Microsoft.Network/expressRouteCircuits/Customer-USE01"
$AuthGUID = "76d40466-c458-4d14-adcf-3d1b56d1cd61"

# Your ExpressRoute Gateway information
$myGroupName = "SAP-East-Coast"
$myGWName = "VNet01GW"
$myGWLocation = "East US"

# Define the name for your connection
$myConnectionName = "VNet01GWConnection"

# Create a new connection between the ER Circuit and your Gateway using the Authorization
$gw = Get-AzVirtualNetworkGateway -Name $myGWName -ResourceGroupName $myGroupName
    
New-AzVirtualNetworkGatewayConnection -Name $myConnectionName `
-ResourceGroupName $myGroupName -Location $myGWLocation -VirtualNetworkGateway1 $gw `
-PeerId $PeerID -ConnectionType ExpressRoute -AuthorizationKey $AuthGUID -ExpressRouteGatewayBypass
```

> [!NOTE]
> El último parámetro del comando New-AzVirtualNetworkGatewayConnection, **ExpressRouteGatewayBypass**, es un nuevo parámetro que habilita ExpressRoute Fast Path. Se trata de una funcionalidad que reduce la latencia de red entre las unidades de instancias grandes de HANA y las máquinas virtuales de Azure. Esta funcionalidad se agregó en mayo de 2019. Para más información, consulte el artículo [Arquitectura de red de SAP HANA (instancias grandes)](./hana-network-architecture.md). Antes de ejecutar los comandos, asegúrese de que ejecuta la versión más reciente de los cmdlets de PowerShell.

Para conectar la puerta de enlace a más de un circuito de ExpressRoute asociado a la suscripción, puede que tenga que ejecutar este paso más de una vez. Por ejemplo, probablemente vaya a conectar la misma puerta de enlace de la red virtual al circuito de ExpressRoute que conecta la red virtual a la red local.

## <a name="applying-expressroute-fast-path-to-existing-hana-large-instance-expressroute-circuits"></a>Aplicación de ExpressRoute Fast Path a circuitos de ExpressRoute de instancias grandes de HANA existentes
Hasta ahora, se ha explicado en la documentación cómo conectar un nuevo circuito ExpressRoute que se creó con una implementación de instancia grande de HANA a una puerta de enlace de Azure ExpressRoute de una de las redes virtuales de Azure. Pero muchos clientes ya tienen configurados sus circuitos ExpressRoute y ya tienen conectadas sus redes virtuales a instancias grandes de HANA. Como la nueva característica ExpressRoute Fast Path reduce la latencia de red, se recomienda que aplique el cambio para usar esta funcionalidad. Los comandos para conectar un nuevo circuito ExpreesRoute y cambiar un circuito ExpressRoute existente son los mismos. Como resultado, es necesario ejecutar esta secuencia de comandos de PowerShell para cambiar un circuito existente. 

```powershell
# Populate with information provided by Microsoft Onboarding team
$PeerID = "/subscriptions/9cb43037-9195-4420-a798-f87681a0e380/resourceGroups/Customer-USE-Circuits/providers/Microsoft.Network/expressRouteCircuits/Customer-USE01"
$AuthGUID = "76d40466-c458-4d14-adcf-3d1b56d1cd61"

# Your ExpressRoute Gateway information
$myGroupName = "SAP-East-Coast"
$myGWName = "VNet01GW"
$myGWLocation = "East US"

# Define the name for your connection
$myConnectionName = "VNet01GWConnection"

# Create a new connection between the ER Circuit and your Gateway using the Authorization
$gw = Get-AzVirtualNetworkGateway -Name $myGWName -ResourceGroupName $myGroupName
    
New-AzVirtualNetworkGatewayConnection -Name $myConnectionName `
-ResourceGroupName $myGroupName -Location $myGWLocation -VirtualNetworkGateway1 $gw `
-PeerId $PeerID -ConnectionType ExpressRoute -AuthorizationKey $AuthGUID -ExpressRouteGatewayBypass
```

Para habilitar la funcionalidad ExpressRoute Fast Path, es importante agregar el último parámetro como se mostró anteriormente.


## <a name="expressroute-global-reach"></a>Global Reach de ExpressRoute
Puede que quiera habilitar Global Reach en uno de estos dos casos, o en ambos:

 - La replicación del sistema HANA sin firewalls ni servidores proxy adicionales.
 - La copia de seguridad entre las unidades de HANA (Instancias grandes) en dos regiones diferentes para realizar copias o actualizaciones del sistema.

Debe tener en cuenta lo siguiente:

- Es necesario proporcionar un intervalo de espacio de direcciones /29. Ese intervalo de direcciones no puede superponerse con ninguno de los otros intervalos de espacio de direcciones que haya usado hasta el momento para conectar instancias grandes de HANA a Azure y no puede superponerse con ninguno de los intervalos de direcciones IP que haya usado en alguna otra parte de Azure o del entorno local.
- No hay ningún límite en los ASN (número de sistema autónomo) que se pueden usar para anunciar las rutas locales en las instancias grandes de HANA. El entorno local no debe anunciar ninguna ruta con ASN privados que se encuentren dentro del intervalo 65000 - 65020 o 65515. 
- En el escenario de conexión del acceso directo local a instancias grandes de HANA, debe calcular una tarifa para el circuito que le conecta a Azure. Para información sobre los precios, consulte el [complemento Global Reach](https://azure.microsoft.com/pricing/details/expressroute/).

Para aplicar uno o ambos escenarios a la implementación, abra un mensaje de soporte técnico con Azure como se describe en [Abrir una solicitud de soporte técnico para HANA (instancias grandes)](./hana-li-portal.md#open-a-support-request-for-hana-large-instances).

Los datos necesarios y las palabras clave que se deben usar para que Microsoft pueda enrutar y ejecutar la solicitud se parecen a estos:

- Servicio: Instancia grande de SAP HANA
- Tipo de problema: Instalación y configuración
- Subtipo de problema: Mi problema no aparece aquí
- Asunto "Modificar mi red: agregar Global Reach"
- Detalles: "Agregar Global Reach a instancia grande de HANA/inquilino de instancia grande de HANA" o "Agregar Global Reach a entorno local/inquilino de instancia grande de HANA".
- Detalles adicionales para el caso de instancia grande de HANA al inquilino de instancia grande de HANA: Debe definir las **dos regiones de Azure** donde se encuentran los dos inquilinos a los que quiere conectarse **Y** debe enviar el **intervalo de direcciones IP /29**.
- Detalles adicionales del caso de entorno local a inquilino de instancia grande de HANA: Debe definir la **región de Azure** donde se implementa el inquilino de instancia grande de HANA al que quiere conectarse directamente. Además, debe proporcionar el **AuthGUID** y el **identificador del mismo nivel de circuito** que recibió al establecer el circuito ExpressRoute entre el entorno local y Azure. Además, debe nombrar el **ASN**. La última entrega es un **intervalo de direcciones IP /29** para ExpressRoute Global Reach.

> [!NOTE]
> Si quiere gestionar ambos casos, debe proporcionar dos intervalos de direcciones IP /29 diferentes que no se superpongan con ningún otro intervalo de direcciones IP usado hasta el momento. 




## <a name="next-steps"></a>Pasos siguientes

- [Requisitos de red adicionales para HLI](hana-additional-network-requirements.md)
