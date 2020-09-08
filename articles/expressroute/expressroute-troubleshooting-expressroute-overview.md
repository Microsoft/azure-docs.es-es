---
title: 'Azure ExpressRoute: Comprobación de la conectividad: guía de solución de problemas'
description: Esta página proporciona instrucciones para la solución de problemas y para validar la conectividad de extremo a extremo de un circuito ExpressRoute.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: troubleshooting
ms.date: 10/31/2019
ms.author: duau
ms.custom: seodec18
ms.openlocfilehash: 5689bf60144cf3d66335eb4d77a96d29d8cdcc96
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89401748"
---
# <a name="verifying-expressroute-connectivity"></a>Comprobación de la conectividad de ExpressRoute
En este artículo encontrará información útil para comprobar y solucionar problemas de conectividad de ExpressRoute. ExpressRoute amplía las redes locales en la nube de Microsoft a través de una conexión privada que suele facilitar un proveedor de conectividad. La conectividad de ExpressRoute implica tradicionalmente tres zonas de red distintas, como se indica a continuación:

-   Red del cliente
-   Red del proveedor
-   Centro de datos de Microsoft

> [!NOTE]
> En el modelo de conectividad directa de ExpressRoute (ofrecido con un ancho de banda de 10 o 100 Gb/s), los clientes pueden conectarse directamente al puerto de los enrutadores de Microsoft Enterprise Edge (MSEE). Por lo tanto, en el modelo de conectividad directa, solo hay zonas de red de clientes y de Microsoft.
>


El propósito de este documento es ayudar al usuario a identificar si existe un problema de conectividad. Además de ayudarle a solicitar soporte técnico al equipo adecuado para resolver un problema. Si se necesita soporte técnico de Microsoft para resolver un problema, abra una incidencia de soporte técnico dirigida al [Soporte técnico de Microsoft][Support].

> [!IMPORTANT]
> Este documento está pensado para ayudarle a diagnosticar y corregir problemas sencillos. No pretende sustituir al soporte técnico de Microsoft. Abra una incidencia de soporte técnico dirigida al [Soporte técnico de Microsoft][Support] si no puede resolver el problema siguiendo estas instrucciones.
>
>

## <a name="overview"></a>Información general
El siguiente diagrama muestra la conectividad lógica de una red de cliente a la red de Microsoft mediante ExpressRoute.
[![1]][1]

En el diagrama anterior, los números indican los puntos de red clave. En este artículo a veces se hace referencia a los puntos de red por su número asociado. Según el modelo de conectividad de ExpressRoute (una ubicación compartida de Cloud Exchange, una conexión Ethernet de punto a punto y una red de conectividad universal, IPVPN), los puntos de red 3 y 4 pueden ser conmutadores (dispositivos de nivel 2) o enrutadores (dispositivos de nivel 3). En el modelo de conectividad directa, no hay puntos de red 3 y 4; en su lugar, los CE (2) están conectados directamente a los MSEE a través de fibra oscura. Los puntos de red clave son los siguientes:

1.  Dispositivo de proceso del cliente (por ejemplo, un servidor o equipo)
2.  CE: enrutadores en el lado del cliente 
3.  PE (hacia los CE): enrutadores o conmutadores del lado del proveedor orientados hacia los enrutadores del lado del cliente. Se hace referencia como PE-CE en este documento.
4.  PE (hacia los MSEE): enrutadores o conmutadores del lado del proveedor orientados hacia los MSEE. Se hace referencia como PE-MSEE en este documento.
5.  MSEE: enrutadores de Microsoft Enterprise Edge (MSEE) ExpressRoute
6.  Puerta de enlace de Virtual Network (VNet)
7.  Dispositivo de proceso de Azure Virtual Network

Si se usan los modelos de conectividad directa, de ubicación compartida de Cloud Exchange o Ethernet de punto a punto, los enrutadores en el lado del cliente (2) establecerán un emparejamiento BGP con los MSEE (5). 

Si se usa el modelo de conectividad universal (IPVPN), los PE-MSEE (4) podrían establecer un emparejamiento BGP con los MSEE (5). Los PE-MSEE propagan las rutas recibidas de Microsoft de nuevo a la red del cliente a través de la red del proveedor de servicios IPVPN.

> [!NOTE]
>Para lograr una alta disponibilidad, Microsoft establece una conectividad paralela totalmente redundante entre pares de MSEE (5) y PE-MSEE (4). También se recomienda establecer una ruta de acceso a la red paralela y totalmente redundante entre la red del cliente y los pares PE-CE. Para obtener más información sobre la alta disponibilidad, consulte el artículo [Diseño de alta disponibilidad con ExpressRoute][HA].
>
>

A continuación se indican los pasos lógicos para solucionar problemas del circuito ExpressRoute:

* [Comprobación del aprovisionamiento y del estado del circuito](#verify-circuit-provisioning-and-state)
  
* [Validación de la configuración de emparejamiento](#validate-peering-configuration)
  
* [Validación de ARP](#validate-arp)
  
* [Validación de BGP y de las rutas en el MSEE](#validate-bgp-and-routes-on-the-msee)
  
* [Confirmación del flujo de tráfico](#confirm-the-traffic-flow)


## <a name="verify-circuit-provisioning-and-state"></a>Comprobación del aprovisionamiento y del estado del circuito
El aprovisionamiento de un circuito ExpressRoute establece conexiones redundantes de nivel 2 entre los CE o PE-MSEE (2 o 4) y los MSEE (5). Para más información acerca de cómo crear, modificar, aprovisionar y comprobar un circuito ExpressRoute, consulte el artículo [Creación y modificación de un circuito ExpressRoute][CreateCircuit].

>[!TIP]
>Una clave de servicio identifica de forma única un circuito ExpressRoute. Si necesita asistencia de Microsoft o de un partner de ExpressRoute para solucionar un problema de ExpressRoute, especifique la clave de servicio para identificar inmediatamente el circuito.
>
>

### <a name="verification-via-the-azure-portal"></a>Comprobación a través de Azure Portal
En Azure Portal, abra la hoja del circuito ExpressRoute. En la sección ![3][3] de la hoja, se enumera la información esencial de ExpressRoute como se muestra en la captura de pantalla siguiente:

![4][4]    

En Essentials (Fundamentos) de ExpressRoute, *Circuit status* (Estado del circuito) indica el estado del circuito en el lado de Microsoft. *Provider status* (Estado del proveedor) indica si el circuito ha sido aprovisionado o no (*Provisioned/Not provisioned*) en el lado del proveedor de servicios. 

Para que un circuito ExpressRoute sea operativo, el valor de *Circuit status* (Estado del circuito) debe ser *Enabled* (Habilitado) y *Provider status* (Estado del proveedor) debe ser *Provisioned* (Aprovisionado)

> [!NOTE]
> Después de configurar un circuito ExpressRoute, si *Estado del circuito* se encuentra no habilitado, póngase en contacto con el [Soporte técnico de Microsoft][Support]. Por otro lado, si *Estado del proveedor* se muestra como no aprovisionado, póngase en contacto con su proveedor de servicios.
>
>

### <a name="verification-via-powershell"></a>Comprobación a través de PowerShell
Para obtener una lista de todos los circuitos ExpressRoute en un grupo de recursos, utilice el siguiente comando:

```azurepowershell
Get-AzExpressRouteCircuit -ResourceGroupName "Test-ER-RG"
```

>[!TIP]
>Si busca el nombre de un grupo de recursos, para encontrarlo, puede enumerar todos los grupos de recursos de la suscripción con el comando *Get-AzResourceGroup*.
>


Para seleccionar un circuito ExpressRoute concreto en un grupo de recursos, utilice el siguiente comando:

```azurepowershell
Get-AzExpressRouteCircuit -ResourceGroupName "Test-ER-RG" -Name "Test-ER-Ckt"
```

Una respuesta de ejemplo es:

```output
Name                             : Test-ER-Ckt
ResourceGroupName                : Test-ER-RG
Location                         : westus2
Id                               : /subscriptions/***************************/resourceGroups/Test-ER-RG/providers/***********/expressRouteCircuits/Test-ER-Ckt
Etag                             : W/"################################"
ProvisioningState                : Succeeded
Sku                              : {
                                    "Name": "Standard_UnlimitedData",
                                    "Tier": "Standard",
                                    "Family": "UnlimitedData"
                                    }
CircuitProvisioningState         : Enabled
ServiceProviderProvisioningState : Provisioned
ServiceProviderNotes             : 
ServiceProviderProperties        : {
                                    "ServiceProviderName": "****",
                                    "PeeringLocation": "******",
                                    "BandwidthInMbps": 100
                                    }
ServiceKey                       : **************************************
Peerings                         : []
Authorizations                   : []
```

Para confirmar si un circuito ExpressRoute está operativo, preste especial atención a los campos siguientes:

```output
CircuitProvisioningState         : Enabled
ServiceProviderProvisioningState : Provisioned
```

> [!NOTE]
> Después de configurar un circuito ExpressRoute, si *Estado del circuito* se encuentra no habilitado, póngase en contacto con el [Soporte técnico de Microsoft][Support]. Por otro lado, si *Estado del proveedor* se muestra como no aprovisionado, póngase en contacto con su proveedor de servicios.
>
>

## <a name="validate-peering-configuration"></a>Validación de la configuración de emparejamiento
Cuando el proveedor de servicios haya completado el aprovisionamiento del circuito ExpressRoute, pueden crearse varias configuraciones de enrutamiento basadas en BGP externo a través del circuito ExpressRoute entre los CE o MSEE-PE (2 o 4) y los MSEE (5). Cada circuito ExpressRoute puede tener: emparejamiento privado de Azure (tráfico a las redes privadas virtuales de Azure) y emparejamiento de Microsoft (tráfico a los puntos de conexión públicos de PaaS y SaaS). Para más información sobre cómo crear y modificar la configuración de enrutamiento, consulte el artículo [Creación y modificación del enrutamiento de un circuito ExpressRoute][CreatePeering].

### <a name="verification-via-the-azure-portal"></a>Comprobación a través de Azure Portal

> [!NOTE]
> En el modelo de conectividad de IPVPN, los proveedores de servicios se encargan de configurar los emparejamientos (servicios de nivel 3). En este modelo, una vez que el proveedor de servicios ha configurado un emparejamiento, si este se muestra en blanco en el portal, pruebe a actualizar la configuración del circuito con el botón de actualización del portal. Esta operación extraerá la configuración de enrutamiento actual del circuito. 
>

En Azure Portal, el estado de un emparejamiento de circuito ExpressRoute se puede comprobar en la hoja del circuito ExpressRoute. En la sección ![3][3] de la hoja, se enumeran los emparejamientos de ExpressRoute como se muestra en la captura de pantalla siguiente:

![5][5]

Como se indicó en el ejemplo anterior, el emparejamiento privado de Azure está aprovisionado, mientras que los emparejamientos públicos de Azure y el emparejamiento de Microsoft no lo están. Un contexto de emparejamiento aprovisionado correctamente también mostrará las subredes de punto a punto principales y secundarias. Las subredes /30 se usan para la dirección IP de interfaz de los MSEE y los CE o los PE-MSEE. En el caso de los emparejamientos que se aprovisionan, también se indica quién modificó la configuración por última vez. 

> [!NOTE]
> Si se produce un error al habilitar un emparejamiento, compruebe si las subredes principales y secundarias asignadas coinciden con la configuración del CE o el PE-MSEE vinculado. Compruebe también si se usan los *VlanId*, *AzureASN* y *PeerASN* correctos en los MSEE, y si estos valores se asignan a los que se usan en el CE o el PE-MSEE vinculado. Si se elige el hash MD5, la clave compartida debe coincidir en el par MSEE y PE-MSEE o CE. La clave compartida que se ha configurado previamente no se mostraría por motivos de seguridad. Si necesita cambiar la configuración en un enrutador MSEE, consulte [Creación y modificación del enrutamiento de un circuito ExpressRoute][CreatePeering].  
>

> [!NOTE]
> En una subred /30 asignada para la interfaz, Microsoft elegirá la segunda dirección IP utilizable de la subred para la interfaz MSEE. Por lo tanto, asegúrese de que se ha asignado la primera dirección IP utilizable de la subred en el CE o el PE-MSEE emparejado.
>


### <a name="verification-via-powershell"></a>Comprobación a través de PowerShell
Para obtener los detalles de configuración del emparejamiento privado de Azure, use los siguientes comandos:

```azurepowershell
$ckt = Get-AzExpressRouteCircuit -ResourceGroupName "Test-ER-RG" -Name "Test-ER-Ckt"
Get-AzExpressRouteCircuitPeeringConfig -Name "AzurePrivatePeering" -ExpressRouteCircuit $ckt
```

Una respuesta de ejemplo para un emparejamiento privado configurado correctamente es:

```output
Name                       : AzurePrivatePeering
Id                         : /subscriptions/***************************/resourceGroups/Test-ER-RG/providers/***********/expressRouteCircuits/Test-ER-Ckt/peerings/AzurePrivatePeering
Etag                       : W/"################################"
PeeringType                : AzurePrivatePeering
AzureASN                   : 12076
PeerASN                    : 123##
PrimaryPeerAddressPrefix   : 172.16.0.0/30
SecondaryPeerAddressPrefix : 172.16.0.4/30
PrimaryAzurePort           : 
SecondaryAzurePort         : 
SharedKey                  : 
VlanId                     : 200
MicrosoftPeeringConfig     : null
ProvisioningState          : Succeeded
```

 Un contexto de emparejamiento habilitado correctamente mostrará los prefijos de dirección principal y secundaria en la lista. Las subredes /30 se usan para la dirección IP de interfaz de los MSEE y los CE o los PE-MSEE.

Para obtener los detalles de configuración del emparejamiento público de Azure, use los siguientes comandos:

```azurepowershell
$ckt = Get-AzExpressRouteCircuit -ResourceGroupName "Test-ER-RG" -Name "Test-ER-Ckt"
Get-AzExpressRouteCircuitPeeringConfig -Name "AzurePublicPeering" -ExpressRouteCircuit $ckt
```

Para obtener los detalles de configuración del emparejamiento de Microsoft, use los siguientes comandos:

```azurepowershell
$ckt = Get-AzExpressRouteCircuit -ResourceGroupName "Test-ER-RG" -Name "Test-ER-Ckt"
Get-AzExpressRouteCircuitPeeringConfig -Name "MicrosoftPeering" -ExpressRouteCircuit $ckt
```

Si no se configura un emparejamiento, aparecerá un mensaje de error. Esta es una respuesta de ejemplo de cuando el emparejamiento indicado (emparejamiento público de Azure en este ejemplo) no está configurado en el circuito:

```azurepowershell
Get-AzExpressRouteCircuitPeeringConfig : Sequence contains no matching element
At line:1 char:1
    + Get-AzExpressRouteCircuitPeeringConfig -Name "AzurePublicPeering ...
    + ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
        + CategoryInfo          : CloseError: (:) [Get-AzExpr...itPeeringConfig], InvalidOperationException
        + FullyQualifiedErrorId : Microsoft.Azure.Commands.Network.GetAzureExpressRouteCircuitPeeringConfigCommand
```

> [!NOTE]
> Si se produce un error al habilitar un emparejamiento, compruebe si las subredes principales y secundarias asignadas coinciden con la configuración del CE o el PE-MSEE vinculado. Compruebe también si se usan los *VlanId*, *AzureASN* y *PeerASN* correctos en los MSEE, y si estos valores se asignan a los que se usan en el CE o el PE-MSEE vinculado. Si se elige el hash MD5, la clave compartida debe coincidir en el par MSEE y PE-MSEE o CE. La clave compartida que se ha configurado previamente no se mostraría por motivos de seguridad. Si necesita cambiar la configuración en un enrutador MSEE, consulte [Creación y modificación del enrutamiento de un circuito ExpressRoute][CreatePeering].  
>
>

> [!NOTE]
> En una subred /30 asignada para la interfaz, Microsoft elegirá la segunda dirección IP utilizable de la subred para la interfaz MSEE. Por lo tanto, asegúrese de que se ha asignado la primera dirección IP utilizable de la subred en el CE o el PE-MSEE emparejado.
>

## <a name="validate-arp"></a>Validación de ARP

La tabla ARP proporciona una asignación de la dirección IP y la dirección MAC para un emparejamiento determinado. La tabla ARP de un emparejamiento de circuito ExpressRoute proporciona la siguiente información para cada interfaz (principal y secundaria):
* Asignación de la dirección IP de la interfaz del enrutador local a la dirección MAC
* Asignación de la dirección IP de la interfaz del enrutador de ExpressRoute a la dirección MAC
* La antigüedad de las tablas ARP de asignación puede ayudar a validar la configuración de nivel 2 y a solucionar los problemas básicos de conectividad de nivel 2.


Consulte el documento [Obtención de tablas ARP en el modelo de implementación de Resource Manager][ARP] para ver la tabla ARP de un emparejamiento de ExpressRoute y descubrir cómo usar la información para solucionar problemas de conectividad de nivel 2.


## <a name="validate-bgp-and-routes-on-the-msee"></a>Validación de BGP y las rutas en el MSEE

Para obtener la tabla de enrutamiento de MSEE en la ruta de acceso *principal* del contexto de enrutamiento *privado*, use el siguiente comando:

```azurepowershell
Get-AzExpressRouteCircuitRouteTable -DevicePath Primary -ExpressRouteCircuitName ******* -PeeringType AzurePrivatePeering -ResourceGroupName ****
```

Ejemplo de respuesta:

```output
Network : 10.1.0.0/16
NextHop : 10.17.17.141
LocPrf  : 
Weight  : 0
Path    : 65515

Network : 10.1.0.0/16
NextHop : 10.17.17.140*
LocPrf  : 
Weight  : 0
Path    : 65515

Network : 10.2.20.0/25
NextHop : 172.16.0.1
LocPrf  : 
Weight  : 0
Path    : 123##
```

> [!NOTE]
> Si el estado de un emparejamiento EBGP entre un MSEE y un CE o un PE-MSEE es activo o está en espera, compruebe si las subredes principales y secundarias asignadas del mismo nivel coinciden con la configuración de los CE o de los PE-MSEE vinculados. Compruebe también si se usan los *VlanId*, *AzureAsn* y *PeerAsn* correctos en los MSEE, y si estos valores se asignan a los que se usan en el PE-MSEE o el CE vinculado. Si se elige el hash MD5, la clave compartida debe coincidir en el par MSEE y CE o PE-MSEE. Si necesita cambiar la configuración en un enrutador MSEE, consulte [Creación y modificación del enrutamiento de un circuito ExpressRoute][CreatePeering].
>


> [!NOTE]
> Si determinados destinos no son accesibles a través de un emparejamiento, compruebe la tabla de rutas de los MSEE para el contexto de emparejamiento correspondiente. Si se encuentra un prefijo coincidente (podría ser IP en NAT) en la tabla de enrutamiento, compruebe si hay firewall, NSG o ACL en la ruta de acceso que bloquee el tráfico.
>


En el ejemplo siguiente se muestra la respuesta del comando para un emparejamiento que no existe:

```azurepowershell
Get-AzExpressRouteCircuitRouteTable : The BGP Peering AzurePublicPeering with Service Key ********************* is not found.
StatusCode: 400
```

## <a name="confirm-the-traffic-flow"></a>Confirmación del flujo de tráfico
Para obtener las estadísticas del tráfico de la ruta de acceso principal y secundaria combinados (bytes de entrada y salida) de un contexto de emparejamiento, use el siguiente comando:

```azurepowershell
Get-AzExpressRouteCircuitStats -ResourceGroupName $RG -ExpressRouteCircuitName $CircuitName -PeeringType 'AzurePrivatePeering'
```

Ejemplo de resultado del comando:

```output
PrimaryBytesIn PrimaryBytesOut SecondaryBytesIn SecondaryBytesOut
-------------- --------------- ---------------- -----------------
     240780020       239863857        240565035         239628474
```

Ejemplo de resultado del comando para un emparejamiento inexistente:

```azurepowershell
Get-AzExpressRouteCircuitRouteTable : The BGP Peering AzurePublicPeering with Service Key ********************* is not found.
StatusCode: 400
```

## <a name="next-steps"></a>Pasos siguientes
Para obtener más información, consulte los siguientes artículos:

- [Ayuda y soporte técnico de Microsoft][Support]
- [Creación y modificación de un circuito ExpressRoute][CreateCircuit]
- [Creación y modificación del enrutamiento de un circuito ExpressRoute][CreatePeering]

<!--Image References-->
[1]: ./media/expressroute-troubleshooting-expressroute-overview/expressroute-logical-diagram.png "Conectividad lógica de Expressroute"
[2]: ./media/expressroute-troubleshooting-expressroute-overview/portal-all-resources.png "Icono All resources"
[3]: ./media/expressroute-troubleshooting-expressroute-overview/portal-overview.png "Icono Overview"
[4]: ./media/expressroute-troubleshooting-expressroute-overview/portal-circuit-status.png "Captura de pantalla de ejemplo de Essentials de ExpressRoute"
[5]: ./media/expressroute-troubleshooting-expressroute-overview/portal-private-peering.png "Captura de pantalla de ejemplo de Essentials de ExpressRoute"

<!--Link References-->
[Support]: https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade
[CreateCircuit]: https://docs.microsoft.com/azure/expressroute/expressroute-howto-circuit-portal-resource-manager 
[CreatePeering]: https://docs.microsoft.com/azure/expressroute/expressroute-howto-routing-portal-resource-manager
[ARP]: https://docs.microsoft.com/azure/expressroute/expressroute-troubleshooting-arp-resource-manager
[HA]: https://docs.microsoft.com/azure/expressroute/designing-for-high-availability-with-expressroute
[DR-Pvt]: https://docs.microsoft.com/azure/expressroute/designing-for-disaster-recovery-with-expressroute-privatepeering





