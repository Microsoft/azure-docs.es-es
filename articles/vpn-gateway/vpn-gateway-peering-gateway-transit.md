---
title: Configuración del tránsito de la puerta de enlace de VPN para el emparejamiento de red virtual
description: Configure el tránsito de puerta de enlace para el emparejamiento de red virtual, a fin de conectar de forma sencilla dos redes virtuales de Azure a una con fines de conectividad.
services: vpn-gateway
titleSuffix: Azure VPN Gateway
author: yushwang
ms.service: vpn-gateway
ms.topic: how-to
ms.tgt_pltfrm: na
ms.date: 09/02/2020
ms.author: yushwang
ms.openlocfilehash: 16a53c440bc818a65175cf6a4a92cd007349e3b4
ms.sourcegitcommit: 9c262672c388440810464bb7f8bcc9a5c48fa326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89421346"
---
# <a name="configure-vpn-gateway-transit-for-virtual-network-peering"></a>Configuración del tránsito de la puerta de enlace de VPN para el emparejamiento de red virtual

Este artículo le ayuda a configurar el tránsito de la puerta de enlace para el emparejamiento de red virtual. El [emparejamiento de red virtual](../virtual-network/virtual-network-peering-overview.md) conecta perfectamente dos redes virtuales de Azure, combinando las dos redes virtuales en una sola para favorecer la conectividad. El [tránsito de la puerta de enlace](../virtual-network/virtual-network-peering-overview.md#gateways-and-on-premises-connectivity) es una propiedad del emparejamiento que permite que una red virtual utilice la puerta de enlace de VPN en la red virtual emparejada para la conectividad entre locales o entre redes virtuales. El siguiente diagrama muestra cómo funciona el tránsito de la puerta de enlace con el emparejamiento de red virtual.

![gateway-transit](./media/vpn-gateway-peering-gateway-transit/gatewaytransit.png)

En el diagrama, el tránsito de la puerta de enlace permite que las redes virtuales emparejadas usen Azure VPN Gateway en Hub-RM. La conectividad disponible en la puerta de enlace de VPN, que incluye las conexiones S2S, P2S y entre redes virtuales, se aplica a las tres redes virtuales. La opción de tránsito está disponible para el emparejamiento entre modelos de implementación iguales o diferentes. La restricción es que la puerta de enlace de VPN solo puede estar en la red virtual mediante el modelo de implementación de Resource Manager, como se muestra en el diagrama.

En la arquitectura de red del tipo hub-and-spoke, el tránsito de la puerta de enlace permite que las redes virtuales de radio compartan la puerta de enlace de VPN en el concentrador, en lugar de implementar puertas de enlace de VPN en todas las redes virtuales de radio. Las rutas a las redes virtuales conectadas a la puerta de enlace o a las redes locales se propagarán a las tablas de enrutamiento de las redes virtuales emparejadas mediante el tránsito de la puerta de enlace. Puede deshabilitar la propagación automática de rutas de la puerta de enlace de VPN. Cree una tabla de enrutamiento con la opción "**Deshabilitar la propagación de rutas BGP**" y asóciela con las subredes para evitar la distribución de rutas en dichas subredes. Para más información, consulte [Tabla de enrutamiento de redes virtuales](../virtual-network/manage-route-table.md).

En este documento se describen dos escenarios:

1. Ambas redes virtuales usan el modelo de implementación de Resource Manager
2. La red virtual de radio se implemente en modo clásico y la red virtual del concentrador con puerta de enlace, en modo de Resource Manager


>[!NOTE]
> Si realiza un cambio en la topología de la red y tiene clientes VPN de Windows, el paquete de cliente VPN para clientes de Windows se debe descargar e instalar nuevamente para que los cambios se apliquen en el cliente.
>

## <a name="requirements"></a>Requisitos



El ejemplo de este documento requiere los siguientes recursos para crearse:

1. Una red virtual Hub-RM con una puerta de enlace de VPN
2. Una red virtual Spoke-RM
3. Una red virtual Spoke-Classic con el modelo de implementación clásica
4. La cuenta que use requiere los permisos y roles necesarios. Para detalles, consulte la sección [Permisos](#permissions) de este artículo.

Para obtener instrucciones al respecto, consulte los siguientes documentos:

1. [Creación de una puerta de enlace de VPN y una red virtual](vpn-gateway-howto-site-to-site-resource-manager-portal.md)
2. [Creación de un emparejamiento de red virtual con el mismo modelo de implementación](../virtual-network/tutorial-connect-virtual-networks-portal.md)
3. [Creación de un emparejamiento de red virtual con distintos modelos de implementación](../virtual-network/create-peering-different-deployment-models.md)

## <a name="permissions"></a><a name="permissions"></a>Permisos

Las cuentas que use para crear un emparejamiento de redes virtuales deben tener los rol o permisos necesarios. En el ejemplo siguiente, si fuera a emparejar dos redes virtuales Hub-RM and Spoke-Classic, la cuenta debe tener los siguientes roles o permisos mínimos para cada red virtual:
    
|Virtual network|Modelo de implementación|Role|Permisos|
|---|---|---|---|
|Hub-RM|Resource Manager|[Colaborador de la red](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor)|Microsoft.Network/virtualNetworks/virtualNetworkPeerings/write|
| |Clásico|[Colaborador de la red clásica](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#classic-network-contributor)|N/D|
|Spoke-Classic|Resource Manager|[Colaborador de la red](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor)|Microsoft.Network/virtualNetworks/peer|
||Clásico|[Colaborador de la red clásica](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#classic-network-contributor)|Microsoft.ClassicNetwork/virtualNetworks/peer|

Obtenga más información sobre los [roles integrados](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) y la asignación de permisos específicos a [roles personalizados](../active-directory/role-based-access-control-custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) (solo Resource Manager).

## <a name="resource-manager-to-resource-manager-peering-with-gateway-transit"></a>Emparejamiento de Resource Manager con Resource Manager con tránsito de la puerta de enlace

Siga las instrucciones para crear o actualizar los emparejamientos de red virtual para habilitar el tránsito de la puerta de enlace.

1. Cree o actualizar el emparejamiento de red virtual de Spoke-RM a Hub-RM desde Azure Portal. Navegue hasta el recurso de la red virtual Spoke-RM y haga clic en "Emparejamientos" y, después, en "Agregar":
    - Establezca la opción "Resource Manager"
    - Seleccione la red virtual Hub-RM en la suscripción correspondiente
    - Asegúrese de que el valor de "Permitir acceso a red virtual" es "Habilitado"
    - Establezca la opción "**Use remote gateways** (Usar puertas de enlace remotas)"
    - Haga clic en "Aceptar"

      ![spokerm-to-hubrm](./media/vpn-gateway-peering-gateway-transit/spokerm-hubrm-peering.png)

2. Si ya se ha creado el emparejamiento, vaya al recurso de emparejamiento y habilite la opción "**Usar puertas de enlace remotas**", con lo que la pantalla debería similar a la de la captura de pantalla se muestra en el paso (1)

3. Cree o actualice el emparejamiento de red virtual de Hub-RM a Spoke-RM desde Azure Portal. Vaya al recurso de la red virtual Hub-RM y haga clic en "Emparejamientos" y, después, en "Agregar":
    - Establezca la opción "Resource Manager"
    - Asegúrese de que el valor de "Permitir acceso a red virtual" es "Habilitado"
    - Seleccione la red virtual "Spoke-RM" en la suscripción correspondiente
    - Establezca la opción "**Permitir tránsito de puerta de enlace**"
    - Haga clic en "Aceptar"

      ![hubrm-to-spokerm](./media/vpn-gateway-peering-gateway-transit/hubrm-spokerm-peering.png)

4. Si ya se ha creado el emparejamiento, navegue al recurso de emparejamiento y habilite la opción "**Permitir tránsito de puerta de enlace**", con lo que la pantalla debería similar a la de la captura de pantalla se muestra en el paso (3)

5. Compruebe que el estado de emparejamiento es "**Conectado**" en ambas redes virtuales

### <a name="powershell-sample"></a>Ejemplo de PowerShell

También puede usar PowerShell para crear o actualizar el emparejamiento con el ejemplo anterior. Reemplace las variables por los nombres de las redes virtuales y los grupos de recursos.

```azurepowershell-interactive
$SpokeRG = "SpokeRG1"
$SpokeRM = "Spoke-RM"
$HubRG   = "HubRG1"
$HubRM   = "Hub-RM"

$spokermvnet = Get-AzVirtualNetwork -Name $SpokeRM -ResourceGroup $SpokeRG
$hubrmvnet   = Get-AzVirtualNetwork -Name $HubRM -ResourceGroup $HubRG

Add-AzVirtualNetworkPeering `
  -Name SpokeRMtoHubRM `
  -VirtualNetwork $spokermvnet `
  -RemoteVirtualNetworkId $hubrmvnet.Id `
  -UseRemoteGateways

Add-AzVirtualNetworkPeering `
  -Name HubRMToSpokeRM `
  -VirtualNetwork $hubrmvnet `
  -RemoteVirtualNetworkId $spokermvnet.Id `
  -AllowGatewayTransit
```

## <a name="classic-to-resource-manager-peering-with-gateway-transit"></a>Emparejamiento de Resource Manager con Clásico con tránsito de la puerta de enlace

Los pasos son similares a los del ejemplo de Resource Manager, salvo que las operaciones se aplican solo a la red virtual Hub-RM.

1. Cree o actualice el emparejamiento de red virtual de Hub-RM a Spoke-RM desde Azure Portal. Vaya al recurso de la red virtual Hub-RM y haga clic en "Emparejamientos" y, después, en "Agregar":
   - Establezca la opción "Clásico" para el modelo de implementación de red virtual
   - Seleccione la red virtual "Spoke-Classic" en la suscripción correspondiente
   - Asegúrese de que el valor de "Permitir acceso a red virtual" es "Habilitado"
   - Establezca la opción "**Permitir tránsito de puerta de enlace**"
   - Haga clic en "Aceptar"

     ![hubrm-to-spokeclassic](./media/vpn-gateway-peering-gateway-transit/hubrm-spokeclassic-peering.png)

2. Si ya se ha creado el emparejamiento, navegue al recurso de emparejamiento y habilite la opción "**Permitir tránsito de puerta de enlace**", con lo que la pantalla debería similar a la de la captura de pantalla se muestra en el paso (1)

3. No hay ninguna operación en la red virtual Spoke-Classic

4. Compruebe que el estado de emparejamiento es "**Conectado**" en la red virtual Hub-RM

Una vez que el estado muestra "Conectado", las redes virtuales de radio pueden empezar a usar la conectividad entre locales o entre redes virtuales a través de la puerta de enlace de VPN en la red virtual del concentrador.

### <a name="powershell-sample"></a>Ejemplo de PowerShell

También puede usar PowerShell para crear o actualizar el emparejamiento con el ejemplo anterior. Reemplace las variables y el identificador de la suscripción por los valores de la red virtual y de los grupos de recursos, así como de la suscripción. Basta con crear un emparejamiento de red virtual en la red virtual del concentrador.

```azurepowershell-interactive
$HubRG   = "HubRG1"
$HubRM   = "Hub-RM"

$hubrmvnet   = Get-AzVirtualNetwork -Name $HubRM -ResourceGroup $HubRG

Add-AzVirtualNetworkPeering `
  -Name HubRMToSpokeRM `
  -VirtualNetwork $hubrmvnet `
  -RemoteVirtualNetworkId "/subscriptions/<subscription Id>/resourceGroups/Default-Networking/providers/Microsoft.ClassicNetwork/virtualNetworks/Spoke-Classic" `
  -AllowGatewayTransit
```

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información acerca de las [restricciones y comportamientos importantes del emparejamiento de redes virtuales](../virtual-network/virtual-network-manage-peering.md#requirements-and-constraints), así como la [configuración del emparejamiento de redes virtuales](../virtual-network/virtual-network-manage-peering.md#create-a-peering) antes de crear uno para usarlo en un entorno de producción.
* Aprenda a [crear una topología del tipo hub-and-spoke](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke#virtual-network-peering) con emparejamiento de redes virtuales y tránsito de la puerta de enlace.
