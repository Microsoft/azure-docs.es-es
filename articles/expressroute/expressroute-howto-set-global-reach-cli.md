---
title: 'Azure ExpressRoute: Configuración de ExpressRoute Global Reach: CLI'
description: Este artículo le ayudará a vincular los circuitos de ExpressRoute para realizar una red privada entre las redes locales y habilitar Global Reach.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 12/12/2018
ms.author: duau
ms.custom: devx-track-azurecli
ms.openlocfilehash: 20ec5bb565a506d935ef7de6475e5062a21faa22
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89395492"
---
# <a name="configure-expressroute-global-reach-by-using-the-azure-cli"></a>Configuración de Global Reach de ExpressRoute mediante la CLI de Azure

Gracias a este artículo podrá configurar Global Reach de Azure ExpressRoute con la CLI de Azure. Para obtener más información, consulte [Global Reach de ExpressRoute](expressroute-global-reach.md).
 
Antes de iniciar la configuración, complete los requisitos siguientes:

* Instale la versión más reciente de la CLI de Azure. Consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli) e [Introducción a la CLI de Azure](/cli/azure/get-started-with-azure-cli).
* Comprenda los [flujos de trabajo](expressroute-workflows.md) del aprovisionamiento de los circuitos ExpressRoute.
* Asegúrese de que los circuitos ExpressRoute están en estado aprovisionado.
* Asegúrese de que el emparejamiento privado de Azure está configurado en los circuitos ExpressRoute.  

### <a name="sign-in-to-your-azure-account"></a>Inicio de sesión en la cuenta de Azure.

Para comenzar la configuración, inicie sesión con la cuenta de Azure. El comando siguiente abre el explorador predeterminado y le pide las credenciales de inicio de sesión para la cuenta de Azure:  

```azurecli
az login
```

Si tiene varias suscripciones de Azure, compruebe las suscripciones de la cuenta:

```azurecli
az account list
```

Especifique la suscripción que quiere usar:

```azurecli
az account set --subscription <your subscription ID>
```

### <a name="identify-your-expressroute-circuits-for-configuration"></a>Identificación de los circuitos ExpressRoute para la configuración

Puede habilitar Global Reach de ExpressRoute entre dos circuitos ExpressRoute cualesquiera, siempre que se encuentren en países o regiones admitidos y se hayan creado en distintas ubicaciones de emparejamiento. Si ambos circuitos pertenecen a la suscripción, puede elegir cualquiera de ellos para ejecutar la configuración como se explica más adelante en este artículo. Si los dos circuitos están en diferentes suscripciones de Azure, debe tener la autorización de una suscripción de Azure y pasar la clave de autorización al ejecutar el comando de configuración en la otra suscripción de Azure.

## <a name="enable-connectivity-between-your-on-premises-networks"></a>Habilitación de la conectividad entre las redes locales

Al ejecutar el comando para habilitar la conectividad, tenga en cuenta los siguientes requisitos para los valores de parámetro:

* El valor de *peer-circuit* debe ser el id. de recurso completo. Por ejemplo:

  > /subscriptions/{id_de_la_suscripción}/resourceGroups/{grupo_de_recursos}/providers/Microsoft.Network/expressRouteCircuits/{nombre_del_circuito}

* *address-prefix* debe ser una subred IPv4 "/29" (por ejemplo, "10.0.0.0/29"). Las direcciones IP de esta subred se usan para establecer conectividad entre los dos circuitos ExpressRoute. No debe usar direcciones de esta subred en las redes virtuales de Azure ni en las redes locales.

Ejecute el siguiente comando de la CLI para conectar dos circuitos de ExpressRoute:

```azurecli
az network express-route peering connection create -g <ResourceGroupName> --circuit-name <Circuit1Name> --peering-name AzurePrivatePeering -n <ConnectionName> --peer-circuit <Circuit2ResourceID> --address-prefix <__.__.__.__/29>
```

El resultado de la CLI tendrá un aspecto similar al siguiente:

```output
{
  "addressPrefix": "<__.__.__.__/29>",
  "authorizationKey": null,
  "circuitConnectionStatus": "Connected",
  "etag": "W/\"48d682f9-c232-4151-a09f-fab7cb56369a\"",
  "expressRouteCircuitPeering": {
    "id": "/subscriptions/<SubscriptionID>/resourceGroups/<ResourceGroupName>/providers/Microsoft.Network/expressRouteCircuits/<Circuit1Name>/peerings/AzurePrivatePeering",
    "resourceGroup": "<ResourceGroupName>"
  },
  "id": "/subscriptions/<SubscriptionID>/resourceGroups/<ResourceGroupName>/providers/Microsoft.Network/expressRouteCircuits/<Circuit1Name>/peerings/AzurePrivatePeering/connections/<ConnectionName>",
  "name": "<ConnectionName>",
  "peerExpressRouteCircuitPeering": {
    "id": "/subscriptions/<SubscriptionID>/resourceGroups/<Circuit2ResourceGroupName>/providers/Microsoft.Network/expressRouteCircuits/<Circuit2Name>/peerings/AzurePrivatePeering",
    "resourceGroup": "<Circuit2ResourceGroupName>"
  },
  "provisioningState": "Succeeded",
  "resourceGroup": "<ResourceGroupName>",
  "type": "Microsoft.Network/expressRouteCircuits/peerings/connections"
}
```

Una vez completada la operación, tendrá conectividad entre las redes locales en ambos lados a través de los dos circuitos de ExpressRoute.

## <a name="enable-connectivity-between-expressroute-circuits-in-different-azure-subscriptions"></a>Habilitación de la conectividad entre circuitos ExpressRoute en distintas suscripciones de Azure

Si los dos circuitos no están en la misma suscripción de Azure, necesita autorización. En la configuración siguiente, genera autorización en la suscripción del circuito 2 y pasa la clave de autorización al circuito 1.

1. Genere una clave de autorización:

   ```azurecli
   az network express-route auth create --circuit-name <Circuit2Name> -g <Circuit2ResourceGroupName> -n <AuthorizationName>
   ```

   El resultado de la CLI tendrá un aspecto similar al siguiente:

   ```output
   {
     "authorizationKey": "<authorizationKey>",
     "authorizationUseStatus": "Available",
     "etag": "W/\"cfd15a2f-43a1-4361-9403-6a0be00746ed\"",
     "id": "/subscriptions/<SubscriptionID>/resourceGroups/<Circuit2ResourceGroupName>/providers/Microsoft.Network/expressRouteCircuits/<Circuit2Name>/authorizations/<AuthorizationName>",
     "name": "<AuthorizationName>",
     "provisioningState": "Succeeded",
     "resourceGroup": "<Circuit2ResourceGroupName>",
     "type": "Microsoft.Network/expressRouteCircuits/authorizations"
   }
   ```

1. Anote tanto el identificador del recurso como la clave de autorización para el circuito 2.

1. Ejecute el siguiente comando en el circuito 1, pasando el identificador de recurso y la clave de autorización del circuito 2:

   ```azurecli
   az network express-route peering connection create -g <ResourceGroupName> --circuit-name <Circuit1Name> --peering-name AzurePrivatePeering -n <ConnectionName> --peer-circuit <Circuit2ResourceID> --address-prefix <__.__.__.__/29> --authorization-key <authorizationKey>
   ```

Una vez completada la operación, tendrá conectividad entre las redes locales en ambos lados a través de los dos circuitos de ExpressRoute.

## <a name="get-and-verify-the-configuration"></a>Obtención y comprobación de la configuración

Use el siguiente comando para verificar la configuración en el circuito donde se ha realizado (el circuito 1 del ejemplo anterior):

```azurecli
az network express-route show -n <CircuitName> -g <ResourceGroupName>
```

En la salida de la CLI, verá *CircuitConnectionStatus*. Le indica si se estableció la conectividad entre los dos circuitos ("Conectado") o no ("Desconectado"). 

## <a name="disable-connectivity-between-your-on-premises-networks"></a>Deshabilitación de la conectividad entre las redes locales

Para deshabilitar la conectividad, ejecute el comando siguiente en el circuito donde se hizo la configuración, (el circuito 1 en el ejemplo anterior).

```azurecli
az network express-route peering connection delete -g <ResourceGroupName> --circuit-name <Circuit1Name> --peering-name AzurePrivatePeering -n <ConnectionName>
```

Use el comando ```show``` para verificar el estado.

Al completar esta operación, dejará de tener conectividad entre las redes locales a través de los circuitos ExpressRoute.

## <a name="next-steps"></a>Pasos siguientes

* [Más información acerca de Global Reach de ExpressRoute](expressroute-global-reach.md)
* [Comprobación de la conectividad de ExpressRoute](expressroute-troubleshooting-expressroute-overview.md)
* [Vinculación de una red virtual a un circuito ExpressRoute](expressroute-howto-linkvnet-arm.md)
