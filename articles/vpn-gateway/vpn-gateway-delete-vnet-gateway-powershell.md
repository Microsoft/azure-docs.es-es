---
title: 'Azure VPN Gateway: Eliminación de una puerta de enlace: PowerShell'
description: Elimine una puerta de enlace de red virtual mediante PowerShell en el modelo de implementación de Resource Manager.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.date: 09/03/2020
ms.author: cherylmc
ms.topic: how-to
ms.openlocfilehash: 48e0998719ea19c0b360b50d8c9171d2b448ad19
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89440769"
---
# <a name="delete-a-virtual-network-gateway-using-powershell"></a>Eliminación de una puerta de enlace de red virtual mediante PowerShell
> [!div class="op_single_selector"]
> * [Azure Portal](vpn-gateway-delete-vnet-gateway-portal.md)
> * [PowerShell](vpn-gateway-delete-vnet-gateway-powershell.md)
> * [PowerShell (clásico)](vpn-gateway-delete-vnet-gateway-classic-powershell.md)
>
>

Hay un par de enfoques que puede adoptar cuando desee eliminar una puerta de enlace de red virtual correspondiente a una configuración de puerta de enlace VPN.

- Si quiere eliminar todo el contenido y volver a empezar, como en el caso de un entorno de prueba, puede eliminar el grupo de recursos. Cuando se elimina un grupo de recursos, se quitan todos los recursos de él. Este método solo se recomienda si no desea conservar los recursos del grupo en cuestión. Con este enfoque no podrá eliminar de forma selectiva solo unos pocos recursos.

- Si desea mantener algunos de los recursos en el grupo de recursos, será algo más complicado eliminar una puerta de enlace de red virtual. Antes de poder eliminar la puerta de enlace de red virtual, primero debe quitar todos los recursos que dependen de la puerta de enlace. Los pasos que seguirá dependerán del tipo de conexiones que creó y de los recursos dependientes de cada conexión.

## <a name="before-beginning"></a>Antes de comenzar



### <a name="1-download-the-latest-azure-resource-manager-powershell-cmdlets"></a>1. Descargue la versión más reciente de los cmdlets de PowerShell de Azure Resource Manager.

Descargue e instale la versión más reciente de los cmdlets de PowerShell de Azure Resource Manager. Consulte [Cómo instalar y configurar Azure PowerShell](/powershell/azure/) para obtener más información sobre cómo instalar y configurar los cmdlets de PowerShell.

### <a name="2-connect-to-your-azure-account"></a>2. Conéctese a su cuenta de Azure.

Abre la consola de PowerShell y conéctate a tu cuenta. Use el siguiente ejemplo para conectarse:

```powershell
Connect-AzAccount
```

Compruebe las suscripciones para la cuenta.

```powershell
Get-AzSubscription
```

Si tiene varias suscripciones, seleccione la que quera usar.

```powershell
Select-AzSubscription -SubscriptionName "Replace_with_your_subscription_name"
```

## <a name="delete-a-site-to-site-vpn-gateway"></a><a name="S2S"></a>Eliminación de una VPN de sitio a sitio

Para eliminar una puerta de enlace de red virtual correspondiente a una configuración de S2S, primero debe eliminar cada recurso que pertenece a la puerta de enlace de red virtual. Los recursos deben eliminarse en un orden determinado debido a las dependencias. Cuando se trabaja con los ejemplos siguientes, deben especificarse algunos de los valores, mientras que otros son un resultado de salida. Los siguientes valores específicos de los ejemplos se utilizan para fines de demostración:

Nombre de la red virtual: VNet1<br>
Nombre del grupo de recursos: RG1<br>
Nombre de la puerta de enlace de red virtual: GW1<br>

Los siguientes pasos se aplican al modelo de implementación de Resource Manager.

### <a name="1-get-the-virtual-network-gateway-that-you-want-to-delete"></a>1. Obtenga la puerta de enlace de red virtual que quiera eliminar.

```powershell
$GW=get-Azvirtualnetworkgateway -Name "GW1" -ResourceGroupName "RG1"
```

### <a name="2-check-to-see-if-the-virtual-network-gateway-has-any-connections"></a>2. Compruebe si la puerta de enlace de red virtual tiene todas las conexiones.

```powershell
get-Azvirtualnetworkgatewayconnection -ResourceGroupName "RG1" | where-object {$_.VirtualNetworkGateway1.Id -eq $GW.Id}
$Conns=get-Azvirtualnetworkgatewayconnection -ResourceGroupName "RG1" | where-object {$_.VirtualNetworkGateway1.Id -eq $GW.Id}
```

### <a name="3-delete-all-connections"></a>3. Elimine todas las conexiones.

Se le pedirá que confirme la eliminación de cada una de las conexiones.

```powershell
$Conns | ForEach-Object {Remove-AzVirtualNetworkGatewayConnection -Name $_.name -ResourceGroupName $_.ResourceGroupName}
```

### <a name="4-delete-the-virtual-network-gateway"></a>4. Elimine la puerta de enlace de red virtual.

Se le pedirá que confirme la eliminación de la puerta de enlace. Si tiene una configuración P2S en esta red virtual además de la configuración S2S, eliminando la puerta de enlace de red virtual se desconectarán automáticamente todos los clientes P2S sin previo aviso.


```powershell
Remove-AzVirtualNetworkGateway -Name "GW1" -ResourceGroupName "RG1"
```

En este momento, la puerta de enlace de virtual se ha eliminado. Puede usar los pasos siguientes para eliminar los recursos que ya no se usan.

### <a name="5-delete-the-local-network-gateways"></a>5\. Elimine las puertas de enlace de red locales.

Obtenga la lista de las puertas de enlace de red locales correspondientes.

```powershell
$LNG=Get-AzLocalNetworkGateway -ResourceGroupName "RG1" | where-object {$_.Id -In $Conns.LocalNetworkGateway2.Id}
```

Elimine las puertas de enlace de red locales. Se le pedirá que confirme la eliminación de cada una de las puertas de enlace de red locales.

```powershell
$LNG | ForEach-Object {Remove-AzLocalNetworkGateway -Name $_.Name -ResourceGroupName $_.ResourceGroupName}
```

### <a name="6-delete-the-public-ip-address-resources"></a>6. Elimine los recursos de dirección IP pública.

Obtenga las configuraciones de direcciones IP de la puerta de enlace de red virtual.

```powershell
$GWIpConfigs = $Gateway.IpConfigurations
```

Obtenga la lista de recursos de direcciones IP públicas que se usan para esta puerta de enlace de red virtual. Si la puerta de enlace de red virtual era activa-activa, verá dos direcciones IP públicas.

```powershell
$PubIP=Get-AzPublicIpAddress | where-object {$_.Id -In $GWIpConfigs.PublicIpAddress.Id}
```

Elimine los recursos de IP pública.

```powershell
$PubIP | foreach-object {remove-AzpublicIpAddress -Name $_.Name -ResourceGroupName "RG1"}
```

### <a name="7-delete-the-gateway-subnet-and-set-the-configuration"></a>7. Elimine la subred de puerta de enlace y establezca la configuración.

```powershell
$GWSub = Get-AzVirtualNetwork -ResourceGroupName "RG1" -Name "VNet1" | Remove-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet"
Set-AzVirtualNetwork -VirtualNetwork $GWSub
```

## <a name="delete-a-vnet-to-vnet-vpn-gateway"></a><a name="v2v"></a>Eliminación de una puerta de enlace VPN de red virtual a red virtual

Para eliminar una puerta de enlace de red virtual correspondiente a una configuración de V2V, primero debe eliminar cada recurso que pertenece a la puerta de enlace de red virtual. Los recursos deben eliminarse en un orden determinado debido a las dependencias. Cuando se trabaja con los ejemplos siguientes, deben especificarse algunos de los valores, mientras que otros son un resultado de salida. Los siguientes valores específicos de los ejemplos se utilizan para fines de demostración:

Nombre de la red virtual: VNet1<br>
Nombre del grupo de recursos: RG1<br>
Nombre de la puerta de enlace de red virtual: GW1<br>

Los siguientes pasos se aplican al modelo de implementación de Resource Manager.

### <a name="1-get-the-virtual-network-gateway-that-you-want-to-delete"></a>1. Obtenga la puerta de enlace de red virtual que quiera eliminar.

```powershell
$GW=get-Azvirtualnetworkgateway -Name "GW1" -ResourceGroupName "RG1"
```

### <a name="2-check-to-see-if-the-virtual-network-gateway-has-any-connections"></a>2. Compruebe si la puerta de enlace de red virtual tiene todas las conexiones.

```powershell
get-Azvirtualnetworkgatewayconnection -ResourceGroupName "RG1" | where-object {$_.VirtualNetworkGateway1.Id -eq $GW.Id}
```
 
Puede haber otra conexión a la puerta de enlace de red virtual que forman parte de otro grupo de recursos. Busque conexiones adicionales en cada grupo de recursos extra. En este ejemplo, vamos a comprobar las conexiones desde RG2. Ejecute este comando en cada grupo de recursos que tenga que puede estar conectado a la puerta de enlace de red virtual.

```powershell
get-Azvirtualnetworkgatewayconnection -ResourceGroupName "RG2" | where-object {$_.VirtualNetworkGateway2.Id -eq $GW.Id}
```

### <a name="3-get-the-list-of-connections-in-both-directions"></a>3. Obtenga la lista de conexiones en ambas direcciones.

Como se trata de una configuración de red virtual a red virtual, necesita la lista de conexiones en ambas direcciones.

```powershell
$ConnsL=get-Azvirtualnetworkgatewayconnection -ResourceGroupName "RG1" | where-object {$_.VirtualNetworkGateway1.Id -eq $GW.Id}
```
 
En este ejemplo, vamos a comprobar las conexiones desde RG2. Ejecute este comando en cada grupo de recursos que tenga que puede estar conectado a la puerta de enlace de red virtual.

```powershell
 $ConnsR=get-Azvirtualnetworkgatewayconnection -ResourceGroupName "<NameOfResourceGroup2>" | where-object {$_.VirtualNetworkGateway2.Id -eq $GW.Id}
 ```

### <a name="4-delete-all-connections"></a>4. Elimine todas las conexiones.

Se le pedirá que confirme la eliminación de cada una de las conexiones.

```powershell
$ConnsL | ForEach-Object {Remove-AzVirtualNetworkGatewayConnection -Name $_.name -ResourceGroupName $_.ResourceGroupName}
$ConnsR | ForEach-Object {Remove-AzVirtualNetworkGatewayConnection -Name $_.name -ResourceGroupName $_.ResourceGroupName}
```

### <a name="5-delete-the-virtual-network-gateway"></a>5. Elimine la puerta de enlace de red virtual.

Se le pedirá que confirme la eliminación de la puerta de enlace de red virtual. Si tiene una configuración P2S en esta red virtual además de la configuración V2V, eliminando la puerta de enlace de red virtual se desconectarán automáticamente todos los clientes P2S sin previo aviso.

```powershell
Remove-AzVirtualNetworkGateway -Name "GW1" -ResourceGroupName "RG1"
```

En este momento, la puerta de enlace de virtual se ha eliminado. Puede usar los pasos siguientes para eliminar los recursos que ya no se usan.

### <a name="6-delete-the-public-ip-address-resources"></a>6. Elimine los recursos de dirección IP pública.

Obtenga las configuraciones de direcciones IP de la puerta de enlace de red virtual.

```powershell
$GWIpConfigs = $Gateway.IpConfigurations
```

Obtenga la lista de recursos de direcciones IP públicas que se usan para esta puerta de enlace de red virtual. Si la puerta de enlace de red virtual era activa-activa, verá dos direcciones IP públicas.

```powershell
$PubIP=Get-AzPublicIpAddress | where-object {$_.Id -In $GWIpConfigs.PublicIpAddress.Id}
```

Elimine los recursos de IP pública. Se le pedirá que confirme la eliminación de la dirección IP pública.

```powershell
$PubIP | foreach-object {remove-AzpublicIpAddress -Name $_.Name -ResourceGroupName "<NameOfResourceGroup1>"}
```

### <a name="7-delete-the-gateway-subnet-and-set-the-configuration"></a>7. Elimine la subred de puerta de enlace y establezca la configuración.

```powershell
$GWSub = Get-AzVirtualNetwork -ResourceGroupName "RG1" -Name "VNet1" | Remove-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet"
Set-AzVirtualNetwork -VirtualNetwork $GWSub
```

## <a name="delete-a-point-to-site-vpn-gateway"></a><a name="deletep2s"></a>Eliminación de una puerta de enlace VPN de sitio a sitio

Para eliminar una puerta de enlace de red virtual correspondiente a una configuración de P2S, primero debe eliminar cada recurso que pertenece a la puerta de enlace de red virtual. Los recursos deben eliminarse en un orden determinado debido a las dependencias. Cuando se trabaja con los ejemplos siguientes, deben especificarse algunos de los valores, mientras que otros son un resultado de salida. Los siguientes valores específicos de los ejemplos se utilizan para fines de demostración:

Nombre de la red virtual: VNet1<br>
Nombre del grupo de recursos: RG1<br>
Nombre de la puerta de enlace de red virtual: GW1<br>

Los siguientes pasos se aplican al modelo de implementación de Resource Manager.


>[!NOTE]
> Cuando se elimina la puerta de enlace VPN, se desconectarán todos los clientes conectados de la red virtual sin previo aviso.
>
>

### <a name="1-get-the-virtual-network-gateway-that-you-want-to-delete"></a>1. Obtenga la puerta de enlace de red virtual que quiera eliminar.

```powershell
$GW=get-Azvirtualnetworkgateway -Name "GW1" -ResourceGroupName "RG1"
```

### <a name="2-delete-the-virtual-network-gateway"></a>2. Elimine la puerta de enlace de red virtual.

Se le pedirá que confirme la eliminación de la puerta de enlace de red virtual.

```powershell
Remove-AzVirtualNetworkGateway -Name "GW1" -ResourceGroupName "RG1"
```

En este momento, la puerta de enlace de virtual se ha eliminado. Puede usar los pasos siguientes para eliminar los recursos que ya no se usan.

### <a name="3-delete-the-public-ip-address-resources"></a>3. Elimine los recursos de dirección IP pública.

Obtenga las configuraciones de direcciones IP de la puerta de enlace de red virtual.

```powershell
$GWIpConfigs = $Gateway.IpConfigurations
```

Obtenga la lista de direcciones IP públicas que se usan para esta puerta de enlace de red virtual. Si la puerta de enlace de red virtual era activa-activa, verá dos direcciones IP públicas.

```powershell
$PubIP=Get-AzPublicIpAddress | where-object {$_.Id -In $GWIpConfigs.PublicIpAddress.Id}
```

Elimine las direcciones IP públicas. Se le pedirá que confirme la eliminación de la dirección IP pública.

```powershell
$PubIP | foreach-object {remove-AzpublicIpAddress -Name $_.Name -ResourceGroupName "<NameOfResourceGroup1>"}
```

### <a name="4-delete-the-gateway-subnet-and-set-the-configuration"></a>4. Elimine la subred de puerta de enlace y establezca la configuración.

```powershell
$GWSub = Get-AzVirtualNetwork -ResourceGroupName "RG1" -Name "VNet1" | Remove-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet"
Set-AzVirtualNetwork -VirtualNetwork $GWSub
```

## <a name="delete-a-vpn-gateway-by-deleting-the-resource-group"></a><a name="delete"></a>Eliminación de una puerta de enlace VPN mediante la eliminación del grupo de recursos

Si no desea mantener ninguno de los recursos del grupo, sino que desea empezar de nuevo, puede eliminar dicho grupo al completo. Se trata de una forma rápida de quitarlos todos. Los siguientes pasos se aplican solo al modelo de implementación de Resource Manager.

### <a name="1-get-a-list-of-all-the-resource-groups-in-your-subscription"></a>1. Obtenga una lista de todos los grupos de recursos de la suscripción:

```powershell
Get-AzResourceGroup
```

### <a name="2-locate-the-resource-group-that-you-want-to-delete"></a>2. Localice el grupo de recursos que desea eliminar.

Localice el grupo de recursos que quiera eliminar y vea la lista de recursos de dicho grupo. En el ejemplo, el nombre del grupo de recursos es RG 1. Modifique el ejemplo para recuperar una lista de todos los recursos.

```powershell
Find-AzResource -ResourceGroupNameContains RG1
```

### <a name="3-verify-the-resources-in-the-list"></a>3. Compruebe los recursos de la lista.

Cuando se devuelve la lista, revísela para comprobar que desea eliminar todos los recursos del grupo de recursos, así como dicho grupo. Si desea mantener algunos de los recursos del grupo de recursos, utilice los pasos de las secciones anteriores de este artículo para eliminar la puerta de enlace.

### <a name="4-delete-the-resource-group-and-resources"></a>4. Elimine el grupo de recursos y los recursos.

Para eliminar el grupo de recursos y todos los recursos de este, modifique el ejemplo y ejecútelo.

```powershell
Remove-AzResourceGroup -Name RG1
```

### <a name="5-check-the-status"></a>5. Compruebe el estado.

Azure tarda unos minutos en eliminar todos los recursos. Puede comprobar el estado de su grupo de recursos con este cmdlet.

```powershell
Get-AzResourceGroup -ResourceGroupName RG1
```

El resultado que se devuelve se muestra "Correcto".

```
ResourceGroupName : RG1
Location          : eastus
ProvisioningState : Succeeded
```
