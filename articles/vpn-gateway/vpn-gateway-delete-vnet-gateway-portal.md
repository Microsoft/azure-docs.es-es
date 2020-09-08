---
title: 'Azure VPN Gateway: Eliminación de una puerta de enlace: portal'
description: Elimine una puerta de enlace de red virtual mediante Azure Portal en el modelo de implementación de Resource Manager.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.date: 09/02/2020
ms.author: cherylmc
ms.topic: how-to
ms.openlocfilehash: 7a7eb87f57676b469203e45e48b6f863cdf894c0
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89400660"
---
# <a name="delete-a-virtual-network-gateway-using-the-portal"></a>Eliminación de una puerta de enlace de red virtual mediante el portal

> [!div class="op_single_selector"]
> * [Azure Portal](vpn-gateway-delete-vnet-gateway-portal.md)
> * [PowerShell](vpn-gateway-delete-vnet-gateway-powershell.md)
> * [PowerShell (clásico)](vpn-gateway-delete-vnet-gateway-classic-powershell.md)

En este artículo se ofrece las instrucciones para eliminar puertas de enlace de VPN de Azure que se hayan implementado utilizando el modelo de implementación de Resource Manager. Hay un par de enfoques que puede adoptar cuando desee eliminar una puerta de enlace de red virtual correspondiente a una configuración de puerta de enlace VPN.

- Si quiere eliminar todo el contenido y volver a empezar, como en el caso de un entorno de prueba, puede eliminar el grupo de recursos. Cuando se elimina un grupo de recursos, se quitan todos los recursos de él. Este método solo se recomienda si no desea conservar los recursos del grupo en cuestión. Con este enfoque no podrá eliminar de forma selectiva solo unos pocos recursos.

- Si desea mantener algunos de los recursos en el grupo de recursos, será algo más complicado eliminar una puerta de enlace de red virtual. Antes de poder eliminar la puerta de enlace de red virtual, primero debe quitar todos los recursos que dependen de la puerta de enlace. Los pasos que seguirá dependerán del tipo de conexiones que creó y de los recursos dependientes de cada conexión.

> [!IMPORTANT]
> Las instrucciones a continuación describen cómo eliminar puertas de enlace de VPN de Azure que se hayan implementado utilizando el modelo de implementación de Resource Manager. Para eliminar una puerta de enlace VPN que se haya implementado utilizando el modelo de implementación clásica, use Azure PowerShell como se describe [aquí](vpn-gateway-delete-vnet-gateway-classic-powershell.md).


## <a name="delete-a-vpn-gateway"></a>Eliminación de una instancia de VPN Gateway

Para eliminar una puerta de enlace de red virtual, primero debe eliminar cada recurso que pertenece a la puerta de enlace de red virtual. Los recursos deben eliminarse en un orden determinado debido a las dependencias.

[!INCLUDE [delete gateway](../../includes/vpn-gateway-delete-vnet-gateway-portal-include.md)]

En este punto, se elimina la puerta de enlace de red virtual. Los pasos siguientes ayudan a eliminar todos los recursos que ya no se usan.

### <a name="to-delete-the-local-network-gateway"></a>Para eliminar la puerta de enlace de red local

1. En **Todos los recursos**, busque las puertas de enlace de red local asociadas a cada conexión.
2. En la hoja **Información general** de la puerta de enlace de red local, haga clic en **Eliminar**.

### <a name="to-delete-the-public-ip-address-resource-for-the-gateway"></a>Para eliminar el recurso de dirección IP pública para la puerta de enlace

1. En **Todos los recursos**, busque el recurso de dirección IP pública que se asoció a la puerta de enlace. Si la puerta de enlace de red virtual era activa-activa, verá dos direcciones IP públicas. 
2. En la página **Introducción general** de la dirección IP pública, haga clic en **Eliminar** y después en **Sí** para confirmar.

### <a name="to-delete-the-gateway-subnet"></a>Para eliminar la subred de la puerta de enlace

1. En **Todos los recursos**, busque la red virtual. 
2. En la hoja **Subredes**, haga clic en el elemento **GatewaySubnet** y después en **Eliminar**. 
3. Haga clic en **Sí** para confirmar que desea eliminar la subred de la puerta de enlace.

## <a name="delete-a-vpn-gateway-by-deleting-the-resource-group"></a><a name="deleterg"></a>Eliminación de una puerta de enlace VPN mediante la eliminación del grupo de recursos

Si no desea mantener ninguno de los recursos del grupo, sino que desea empezar de nuevo, puede eliminar dicho grupo al completo. Se trata de una forma rápida de quitarlos todos. Los siguientes pasos se aplican solo al modelo de implementación de Resource Manager.

1. En **Todos los recursos**, busque el grupo de recursos y haga clic para abrir la hoja.
2. Haga clic en **Eliminar**. En la hoja Eliminar, revise los recursos afectados. Asegúrese de que desea eliminar todos estos recursos. Si no es así, siga los pasos de Eliminación de una puerta de enlace VPN en la parte superior de este artículo.
3. Para continuar, escriba el nombre del grupo de recursos que quiere eliminar y, después, haga clic en **Eliminar**.
