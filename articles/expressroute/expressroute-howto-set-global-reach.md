---
title: 'Azure ExpressRoute: Configuración de Global Reach'
description: Este artículo le ayudará a vincular los circuitos de ExpressRoute para realizar una red privada entre las redes locales y habilitar Global Reach.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 02/25/2019
ms.author: duau
ms.openlocfilehash: dd4c6f0b9d518acf06f7d018a65cc2b9b92db33d
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89395458"
---
# <a name="configure-expressroute-global-reach"></a>Configuración de ExpressRoute Global Reach

Este artículo le ayuda a configurar Global Reach de ExpressRoute con PowerShell. Para más información, consulte [Global Reach de ExpressRoute](expressroute-global-reach.md).

 ## <a name="before-you-begin"></a>Antes de empezar

Antes de comenzar la configuración, confirme lo siguiente:

* Conoce los [flujos de trabajo](expressroute-workflows.md) del aprovisionamiento de circuitos ExpressRoute.
* Los circuitos ExpressRoute están en estado aprovisionado.
* El emparejamiento privado de Azure está configurado en los circuitos ExpressRoute.
* Si quiere ejecutar PowerShell localmente, compruebe que la versión más reciente de Azure PowerShell está instalada en el equipo.

### <a name="working-with-azure-powershell"></a>Trabajo con Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/hybrid-az-ps.md)]

[!INCLUDE [expressroute-cloudshell](../../includes/expressroute-cloudshell-powershell-about.md)]

## <a name="identify-circuits"></a>Identificar circuitos

1. Para iniciar la configuración, inicie sesión en su cuenta de Azure y seleccione la suscripción que quiere usar.

   [!INCLUDE [sign in](../../includes/expressroute-cloud-shell-connect.md)]
2. Identifique los circuitos ExpressRoute que quiere usar. Puede habilitar Global Reach de ExpressRoute entre el emparejamiento privado de dos circuitos ExpressRoute cualesquiera, siempre que se encuentren en los países o regiones admitidos y se hayan creado en distintas ubicaciones de emparejamiento. 

   * Si ambos circuitos pertenecen a la suscripción, puede elegir cualquiera de ellos para ejecutar la configuración en las secciones siguientes.
   * Si los circuitos están en diferentes suscripciones de Azure, necesita la autorización de una suscripción de Azure. Luego debe usar la clave de autorización al ejecutar el comando de configuración en la otra suscripción de Azure.

## <a name="enable-connectivity"></a>Habilitación de la conectividad

Habilite la conectividad entre las redes locales. Existen diferentes conjuntos de instrucciones para los circuitos de la misma suscripción de Azure y los circuitos de distintas suscripciones.

### <a name="expressroute-circuits-in-the-same-azure-subscription"></a>Circuitos ExpressRoute en la misma suscripción de Azure

1. Use los siguientes comandos para obtener el circuito 1 y el circuito 2. Los dos circuitos están en la misma suscripción.

   ```azurepowershell-interactive
   $ckt_1 = Get-AzExpressRouteCircuit -Name "Your_circuit_1_name" -ResourceGroupName "Your_resource_group"
   $ckt_2 = Get-AzExpressRouteCircuit -Name "Your_circuit_2_name" -ResourceGroupName "Your_resource_group"
   ```
2. Ejecute el siguiente comando en el circuito 1 y use el identificador de emparejamiento privado del circuito 2. Al ejecutar el comando, tenga en cuenta lo siguiente:

   * El identificador de emparejamiento privado es similar al del siguiente ejemplo: 

     ```
     /subscriptions/{your_subscription_id}/resourceGroups/{your_resource_group}/providers/Microsoft.Network/expressRouteCircuits/{your_circuit_name}/peerings/AzurePrivatePeering
     ```
   * *-AddressPrefix* debe ser una subred IPv4 /29, por ejemplo, "10.0.0.0/29". Las direcciones IP de esta subred se usan para establecer conectividad entre los dos circuitos ExpressRoute. Las direcciones de esta subred no se deben usar en las redes virtuales de Azure ni en la red local.

     ```azurepowershell-interactive
     Add-AzExpressRouteCircuitConnectionConfig -Name 'Your_connection_name' -ExpressRouteCircuit $ckt_1 -PeerExpressRouteCircuitPeering $ckt_2.Peerings[0].Id -AddressPrefix '__.__.__.__/29'
     ```
3. Guarde la configuración en el circuito 1 como se indica a continuación:

   ```azurepowershell-interactive
   Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt_1
   ```

Una vez concluida la operación anterior, tendrá conectividad entre las redes locales en ambos lados a través de los dos circuitos ExpressRoute.

### <a name="expressroute-circuits-in-different-azure-subscriptions"></a>Circuitos ExpressRoute en distintas suscripciones de Azure

Si los dos circuitos no están en la misma suscripción de Azure, necesita autorización. En la configuración siguiente, la autorización se genera en la suscripción del circuito 2 y la clave de autorización se pasa al circuito 1.

1. Genere una clave de autorización.

   ```azurepowershell-interactive
   $ckt_2 = Get-AzExpressRouteCircuit -Name "Your_circuit_2_name" -ResourceGroupName "Your_resource_group"
   Add-AzExpressRouteCircuitAuthorization -ExpressRouteCircuit $ckt_2 -Name "Name_for_auth_key"
   Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt_2
   ```

   Anote el identificador de emparejamiento privado del circuito 2, así como de la clave de autorización.
2. Ejecute el siguiente comando en el circuito 1. Pase el identificador de emparejamiento privado del circuito 2 y la clave de autorización.

   ```azurepowershell-interactive
   Add-AzExpressRouteCircuitConnectionConfig -Name 'Your_connection_name' -ExpressRouteCircuit $ckt_1 -PeerExpressRouteCircuitPeering "circuit_2_private_peering_id" -AddressPrefix '__.__.__.__/29' -AuthorizationKey '########-####-####-####-############'
   ```
3. Guarde la configuración en el circuito 1.

   ```azurepowershell-interactive
   Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt_1
   ```

Una vez concluida la operación anterior, tendrá conectividad entre las redes locales en ambos lados a través de los dos circuitos ExpressRoute.

## <a name="verify-the-configuration"></a>Comprobar la configuración

Use el siguiente comando para comprobar la configuración en el circuito en el que se realizó, por ejemplo, el circuito 1 en el ejemplo anterior.
```azurepowershell-interactive
$ckt1 = Get-AzExpressRouteCircuit -Name "Your_circuit_1_name" -ResourceGroupName "Your_resource_group"
```

Si solo ejecuta *$ckt1* en PowerShell, se ve el valor de *CircuitConnectionStatus* en la salida. Le indica si se establece la conectividad, "Conectado" o "Desconectado". 

## <a name="disable-connectivity"></a>Deshabilitar la conectividad

Para deshabilitar la conectividad entre las redes locales, ejecute los comandos en el circuito en el que se ha realizado la configuración, (por ejemplo, el circuito 1 en el ejemplo anterior).

```azurepowershell-interactive
$ckt1 = Get-AzExpressRouteCircuit -Name "Your_circuit_1_name" -ResourceGroupName "Your_resource_group"
Remove-AzExpressRouteCircuitConnectionConfig -Name "Your_connection_name" -ExpressRouteCircuit $ckt_1
Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt_1
```

Puede ejecutar la operación Get para comprobar el estado.

Tras finalizar la operación anterior, dejará de tener conectividad entre la red local a través de los circuitos ExpressRoute.

## <a name="next-steps"></a>Pasos siguientes
1. [Más información acerca de Global Reach de ExpressRoute](expressroute-global-reach.md)
2. [Comprobación de la conectividad de ExpressRoute](expressroute-troubleshooting-expressroute-overview.md)
3. [Vinculación de un circuito ExpressRoute a una red virtual de Azure](expressroute-howto-linkvnet-arm.md)
