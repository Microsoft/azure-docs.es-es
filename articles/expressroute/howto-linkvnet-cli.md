---
title: 'Azure ExpressRoute: Vinculación de una red virtual a un circuito: CLI'
description: Este artículo le muestra cómo vincular redes virtuales a circuitos ExpressRoute mediante el modelo de implementación de Resource Manager y la CLI.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 07/27/2020
ms.author: duau
ms.openlocfilehash: ac36e303cbeaf5167b5bbec3cea503c37b276058
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89393214"
---
# <a name="connect-a-virtual-network-to-an-expressroute-circuit-using-cli"></a>Conexión de una red virtual a un circuito ExpressRoute mediante la CLI

Este artículo lo ayudará a vincular redes virtuales a circuitos ExpressRoute de Azure a través la CLI. Para realizar la vinculación mediante la CLI de Azure, las redes virtuales deben crearse mediante el modelo de implementación de Resource Manager. Pueden estar en la misma suscripción o formar parte de otra suscripción. Si quiere usar un método diferente para conectar su red virtual con un circuito ExpressRoute, seleccione un artículo de la siguiente lista:

> [!div class="op_single_selector"]
> * [Azure Portal](expressroute-howto-linkvnet-portal-resource-manager.md)
> * [PowerShell](expressroute-howto-linkvnet-arm.md)
> * [CLI de Azure](howto-linkvnet-cli.md)
> * [Vídeo: Azure Portal](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-create-a-connection-between-your-vpn-gateway-and-expressroute-circuit)
> * [PowerShell (clásico)](expressroute-howto-linkvnet-classic.md)
> 

## <a name="configuration-prerequisites"></a>Requisitos previos de configuración

* Necesitará la versión más reciente de la interfaz de la línea de comandos (CLI). Para más información, consulte [Instalación de la CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli).

* Debe revisar los [requisitos previos](expressroute-prerequisites.md), los [requisitos de enrutamiento](expressroute-routing.md) y los [flujos de trabajo](expressroute-workflows.md) antes de comenzar la configuración.

* Tiene que tener un circuito ExpressRoute activo. 
  * Siga las instrucciones para [crear un circuito ExpressRoute](howto-circuit-cli.md) y habilite el circuito mediante el proveedor de conectividad. 
  * Asegúrese de que dispone de un emparejamiento privado de Azure configurado para el circuito. Consulte el artículo de [configuración del enrutamiento](howto-routing-cli.md) para obtener instrucciones sobre el enrutamiento. 
  * Asegúrese de que se haya configurado un emparejamiento privado de Azure. El emparejamiento BGP entre la red y Microsoft debe estar activo para habilitar la conectividad de un extremo a otro.
  * Asegúrese de que ha creado y aprovisionado totalmente una red virtual y una puerta de enlace de red virtual. Siga las instrucciones para realizar la [Configuración de una puerta de enlace de red virtual en ExpressRoute](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-cli). Asegúrese de usar `--gateway-type ExpressRoute`.

* Es posible vincular hasta 10 redes virtuales a un circuito ExpressRoute estándar. Todas las redes virtuales deben pertenecer a la misma región geopolítica al utilizar un circuito de ExpressRoute estándar. 

* Una red virtual solo se puede vincular a cuatro circuitos ExpressRoute como máximo. Use el procedimiento siguiente para crear un nuevo objeto de conexión para cada circuito ExpressRoute al que quiere conectarse. Los circuitos ExpressRoute pueden estar en la misma suscripción, en suscripciones diferentes o en una combinación de ambas.

* Si habilita el complemento premium de ExpressRoute, puede vincular una red virtual fuera de la región geopolítica del circuito ExpressRoute, o bien conectar un mayor número de redes virtuales a este. Para más información acerca del complemento premium, consulte las [preguntas más frecuentes](expressroute-faqs.md).

## <a name="connect-a-virtual-network-in-the-same-subscription-to-a-circuit"></a>Conexión de una red virtual en la misma suscripción a un circuito

Puede conectar una puerta de enlace de red virtual a un circuito ExpressRoute mediante el ejemplo. Asegúrese de que la puerta de enlace de red virtual se crea y está lista para vincular antes de ejecutar el comando.

```azurecli
az network vpn-connection create --name ERConnection --resource-group ExpressRouteResourceGroup --vnet-gateway1 VNet1GW --express-route-circuit2 MyCircuit
```

## <a name="connect-a-virtual-network-in-a-different-subscription-to-a-circuit"></a>Conexión de una red virtual en una suscripción diferente a un circuito

Puede compartir un circuito ExpressRoute entre varias suscripciones. En la ilustración siguiente se muestra un sencillo esquema de cómo funciona el uso compartido de circuitos ExpressRoute entre varias suscripciones.

Cada una de las nubes más pequeñas dentro de la nube de gran tamaño se usa para representar las suscripciones que pertenecen a diferentes departamentos dentro de una organización. Cada departamento de la organización puede usar su propia suscripción para implementar sus servicios, pero puede compartir un único circuito ExpressRoute para volver a conectarse a la red local. Un solo departamento (en este ejemplo: TI) puede ser el propietario de ExpressRoute. Otras suscripciones dentro de la organización pueden usar el circuito ExpressRoute.

> [!NOTE]
> Los cargos de conectividad y ancho de banda de un circuito dedicado recaerán en el propietario del circuito ExpressRoute. Todas las redes virtuales comparten el mismo ancho de banda.
> 
> 

![Conectividad entre suscripciones](./media/expressroute-howto-linkvnet-classic/cross-subscription.png)

### <a name="administration---circuit-owners-and-circuit-users"></a>Administración: los propietarios del circuito y los usuarios del circuito

El "propietario del circuito" es un usuario avanzado autorizado del recurso del circuito ExpressRoute. Puede crear autorizaciones que los "propietarios del circuito", a su vez, pueden canjear. Los usuarios del circuito son propietarios de puertas de enlace de red virtual que no se incluyen en la misma suscripción que el circuito ExpressRoute. Los usuarios del circuito pueden canjear autorizaciones (una autorización por red virtual).

El propietario del circuito tiene la capacidad de modificar y revocar las autorizaciones en cualquier momento. Cuando se revoca una autorización, da como resultado la eliminación de todas las conexiones de vínculos de la suscripción cuyo acceso se haya revocado.

### <a name="circuit-owner-operations"></a>Operaciones del propietario del circuito

**Creación de una autorización**

El propietario del circuito crea una autorización, que crea una clave de autorización que puede usar un usuario del circuito para conectar sus puertas de enlace de red virtual al circuito ExpressRoute. Una autorización solo es válida para una conexión.

En el ejemplo siguiente, se muestra cómo crear una autorización:

```azurecli
az network express-route auth create --circuit-name MyCircuit -g ExpressRouteResourceGroup -n MyAuthorization
```

La respuesta contendrá la clave y el estado de la autorización:

```output
"authorizationKey": "0a7f3020-541f-4b4b-844a-5fb43472e3d7",
"authorizationUseStatus": "Available",
"etag": "W/\"010353d4-8955-4984-807a-585c21a22ae0\"",
"id": "/subscriptions/81ab786c-56eb-4a4d-bb5f-f60329772466/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/MyCircuit/authorizations/MyAuthorization1",
"name": "MyAuthorization1",
"provisioningState": "Succeeded",
"resourceGroup": "ExpressRouteResourceGroup"
```

**Revisión de autorizaciones**

El propietario del circuito puede revisar todas las autorizaciones emitidas en un circuito concreto ejecutando el siguiente ejemplo:

```azurecli
az network express-route auth list --circuit-name MyCircuit -g ExpressRouteResourceGroup
```

**Adición de autorizaciones**

El propietario del circuito puede agregar las autorizaciones mediante el siguiente ejemplo:

```azurecli
az network express-route auth create --circuit-name MyCircuit -g ExpressRouteResourceGroup -n MyAuthorization1
```

**Eliminación de autorizaciones**

El propietario del circuito puede revocar o eliminar las autorizaciones al usuario ejecutando el siguiente ejemplo:

```azurecli
az network express-route auth delete --circuit-name MyCircuit -g ExpressRouteResourceGroup -n MyAuthorization1
```

### <a name="circuit-user-operations"></a>Operaciones del usuario del circuito

El usuario del circuito necesita el ID de mismo nivel y una clave de autorización del propietario del circuito. la clave de autorización es un GUID.

```powershell
Get-AzExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "MyRG"
```

**Canje de una autorización de conexión**

El usuario del circuito puede ejecutar el siguiente ejemplo para canjear una autorización de vínculo:

```azurecli
az network vpn-connection create --name ERConnection --resource-group ExpressRouteResourceGroup --vnet-gateway1 VNet1GW --express-route-circuit2 MyCircuit --authorization-key "^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^"
```

**Liberación de una autorización de conexión**

Puede liberar una autorización eliminando la conexión que vincula el circuito ExpressRoute a la red virtual.

## <a name="modify-a-virtual-network-connection"></a>Modificación de una conexión de red virtual
Puede actualizar ciertas propiedades de una conexión de red virtual. 

**Actualización del peso de la conexión**

La red virtual puede estar conectada a varios circuitos ExpressRoute. Es posible que reciba el mismo prefijo de más de un circuito ExpressRoute. Para elegir la conexión por la que enviar el tráfico destinado a este prefijo, puede cambiar el valor de *RoutingWeight* de una conexión. El tráfico se enviará por la conexión con el valor más alto de *RoutingWeight*.

```azurecli
az network vpn-connection update --name ERConnection --resource-group ExpressRouteResourceGroup --routing-weight 100
```

El intervalo de *RoutingWeight* abarca de 0 a 32 000. El valor predeterminado es 0.

## <a name="configure-expressroute-fastpath"></a>Configuración de FastPath de ExpressRoute 
Puede habilitar [FastPath de ExpressRoute](expressroute-about-virtual-network-gateways.md) si la puerta de enlace de red virtual es Ultra Performance o ErGw3AZ. FastPath mejora el rendimiento de la ruta de datos, como paquetes y conexiones por segundo entre la red local y la red virtual. 

**Configuración de FastPath en una conexión nueva**

```azurecli
az network vpn-connection create --name ERConnection --resource-group ExpressRouteResourceGroup --express-route-gateway-bypass true --vnet-gateway1 VNet1GW --express-route-circuit2 MyCircuit
```

**Actualización de una conexión existente para habilitar FastPath**

```azurecli
az network vpn-connection update --name ERConnection --resource-group ExpressRouteResourceGroup --express-route-gateway-bypass true
```

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información acerca de ExpressRoute, consulte [P+F de ExpressRoute](expressroute-faqs.md).
