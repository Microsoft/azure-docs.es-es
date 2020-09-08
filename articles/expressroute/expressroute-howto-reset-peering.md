---
title: 'Azure ExpressRoute: Restablecimiento de emparejamiento de circuitos'
description: Aprenda a deshabilitar y habilitar emparejamientos de un circuito de Azure ExpressRoute mediante Azure PowerShell. Cuando se configuran emparejamientos, están habilitados de forma predeterminada.
services: expressroute
author: charwen
ms.service: expressroute
ms.topic: how-to
ms.date: 01/13/2018
ms.author: duau
ms.openlocfilehash: f3b34966aa46ca8d663f83ab2aceafa4b0dda2eb
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89395747"
---
# <a name="reset-expressroute-circuit-peerings"></a>Restablecimiento de emparejamientos de circuitos ExpressRoute

En este artículo se describe cómo deshabilitar y habilitar los emparejamientos de un circuito de ExpressRoute mediante PowerShell. Cuando se deshabilita un emparejamiento, la sesión de BGP en la conexión principal y en la conexión secundaria del circuito de ExpressRoute se cerrará. Perderá la conectividad mediante este emparejamiento a Microsoft. Cuando habilita un emparejamiento, la sesión de BGP en la conexión principal y en la conexión secundaria del circuito de ExpressRoute se desactivará. Volverá a establecer la conectividad mediante este emparejamiento a Microsoft. Puede habilitar y deshabilitar el emparejamiento de Microsoft y el emparejamiento privado de Azure en un circuito de ExpressRoute de forma independiente. La primera vez que configure los emparejamientos en el circuito de ExpressRoute, los emparejamientos se habilitan de forma predeterminada.

Hay un par de escenarios en los que puede resultarle útil restablecer los emparejamientos de ExpressRoute.
* Pruebe la implementación y el diseño de recuperación ante desastres. Por ejemplo, tiene dos circuitos de ExpressRoute. Puede deshabilitar los emparejamientos de un circuido y forzar la conmutar por error del tráfico de red a otro circuito.
* Habilite la detección de reenvío bidireccional (BFD) en el emparejamiento privado de Azure o el emparejamiento de Microsoft de su circuito de ExpressRoute. BFD está habilitado de forma predeterminada en el emparejamiento privado de Azure si el circuito ExpressRoute se ha creado después del 1 de agosto de 2018, y en el emparejamiento de Microsoft si se ha creado después del 10 de enero de 2020. Si el circuito se creó antes de esa fecha, BFD no estará habilitado. Para habilitar BFD, debe deshabilitar el emparejamiento y volverlo a habilitar. 

### <a name="working-with-azure-powershell"></a>Trabajo con Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/hybrid-az-ps.md)]

[!INCLUDE [expressroute-cloudshell](../../includes/expressroute-cloudshell-powershell-about.md)]

## <a name="reset-a-peering"></a>Restablecimiento de un emparejamiento

1. Si ejecuta PowerShell de manera local, abra la consola de PowerShell con privilegios elevados y conéctese a su cuenta. Use el siguiente ejemplo para conectarse:

   ```azurepowershell
   Connect-AzAccount
   ```
2. Si tiene varias suscripciones de Azure, compruebe las suscripciones de la cuenta.

   ```azurepowershell-interactive
   Get-AzSubscription
   ```
3. Especifique la suscripción que desea usar.

   ```azurepowershell-interactive
   Select-AzSubscription -SubscriptionName "Replace_with_your_subscription_name"
   ```
4. Ejecute los comandos siguientes para recuperar el circuito de ExpressRoute.

   ```azurepowershell-interactive
   $ckt = Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"
   ```
5. Identifique el emparejamiento que quiere habilitar o deshabilitar. *Peerings* es una matriz. En el ejemplo siguiente, Peerings[0] es el emparejamiento privado de Azure y Peerings[1], el emparejamiento de Microsoft.

   ```azurepowershell-interactive
   Name                             : ExpressRouteARMCircuit
   ResourceGroupName                : ExpressRouteResourceGroup
   Location                         : westus
   Id                               : /subscriptions/########-####-####-####-############/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
   Etag                             : W/"cd011bef-dc79-49eb-b4c6-81fb6ea5d178"
   ProvisioningState                : Succeeded
   Sku                              : {
                                     "Name": "Standard_MeteredData",
                                     "Tier": "Standard",
                                     "Family": "MeteredData"
                                   }
   CircuitProvisioningState         : Enabled
   ServiceProviderProvisioningState : Provisioned
   ServiceProviderNotes             :
   ServiceProviderProperties        : {
                                     "ServiceProviderName": "Coresite",
                                     "PeeringLocation": "Los Angeles",
                                     "BandwidthInMbps": 50
                                   }
   ServiceKey                       : ########-####-####-####-############
   Peerings                         : [
                                     {
                                       "Name": "AzurePrivatePeering",
                                       "Etag": "W/\"cd011bef-dc79-49eb-b4c6-81fb6ea5d178\"",
                                       "Id": "/subscriptions/########-####-####-####-############/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit/peerings/AzurePrivatePeering",
                                       "PeeringType": "AzurePrivatePeering",
                                       "State": "Enabled",
                                       "AzureASN": 12076,
                                       "PeerASN": 123,
                                       "PrimaryPeerAddressPrefix": "10.0.0.0/30",
                                       "SecondaryPeerAddressPrefix": "10.0.0.4/30",
                                       "PrimaryAzurePort": "",
                                       "SecondaryAzurePort": "",
                                       "VlanId": 789,
                                       "MicrosoftPeeringConfig": {
                                         "AdvertisedPublicPrefixes": [],
                                         "AdvertisedCommunities": [],
                                         "AdvertisedPublicPrefixesState": "NotConfigured",
                                         "CustomerASN": 0,
                                         "LegacyMode": 0,
                                         "RoutingRegistryName": "NONE"
                                       },
                                       "ProvisioningState": "Succeeded",
                                       "GatewayManagerEtag": "",
                                       "LastModifiedBy": "Customer",
                                       "Connections": []
                                     },
                                     {
                                       "Name": "MicrosoftPeering",
                                       "Etag": "W/\"cd011bef-dc79-49eb-b4c6-81fb6ea5d178\"",
                                       "Id": "/subscriptions/########-####-####-####-############/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit/peerings/MicrosoftPeering",
                                       "PeeringType": "MicrosoftPeering",
                                       "State": "Enabled",
                                       "AzureASN": 12076,
                                       "PeerASN": 123,
                                       "PrimaryPeerAddressPrefix": "3.0.0.0/30",
                                       "SecondaryPeerAddressPrefix": "3.0.0.4/30",
                                       "PrimaryAzurePort": "",
                                       "SecondaryAzurePort": "",
                                       "VlanId": 345,
                                       "MicrosoftPeeringConfig": {
                                         "AdvertisedPublicPrefixes": [
                                           "3.0.0.3/32"
                                         ],
                                         "AdvertisedCommunities": [],
                                         "AdvertisedPublicPrefixesState": "ValidationNeeded",
                                         "CustomerASN": 0,
                                         "LegacyMode": 0,
                                         "RoutingRegistryName": "NONE"
                                       },
                                       "ProvisioningState": "Succeeded",
                                       "GatewayManagerEtag": "",
                                       "LastModifiedBy": "Customer",
                                       "Connections": []
                                     }
                                   ]
   Authorizations                   : []
   AllowClassicOperations           : False
   GatewayManagerEtag               :
   ```
6. Ejecute el comando siguiente para cambiar el estado del emparejamiento.

   ```azurepowershell-interactive
   $ckt.Peerings[0].State = "Disabled"
   Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
   ```
   El emparejamiento debe tener el estado que le haya establecido. 

## <a name="next-steps"></a>Pasos siguientes
Si necesita ayuda para solucionar un problema de ExpressRoute, consulte los artículos siguientes:
* [Comprobación de la conectividad de ExpressRoute](expressroute-troubleshooting-expressroute-overview.md)
* [Solución de problemas de rendimiento de red](expressroute-troubleshooting-network-performance.md)
