---
title: Creación y administración del emparejamiento público de Azure ExpressRoute
description: Más información sobre el emparejamiento público de Azure y cómo administrarlo
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: conceptual
ms.date: 12/16/2019
ms.author: duau
ms.openlocfilehash: a5829399b70871903c8eb433b95e0cb09e5d2c60
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89398093"
---
# <a name="create-and-manage-expressroute-public-peering"></a>Creación y administración del emparejamiento público de ExpressRoute

> [!div class="op_single_selector"]
> * [Artículo: emparejamiento público](about-public-peering.md)
> * [Vídeo: pares públicos](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-set-up-azure-public-peering-for-your-expressroute-circuit)
> * [Artículo: emparejamiento de Microsoft](expressroute-circuit-peerings.md#microsoftpeering)
>

Este artículo le ayuda a crear y administrar la configuración de enrutamiento del emparejamiento público de un circuito ExpressRoute. También puede comprobar el estado de los emparejamientos, así como actualizarlos o eliminarlos y desaprovisionarlos. Este artículo se aplica a los circuitos de Resource Manager creados antes de que se dejara de usar el emparejamiento público. Si ya tiene un circuito (creado antes de que se dejara de usar el emparejamiento público), puede administrar o configurar el emparejamiento público mediante [Azure PowerShell](#powershell), la [CLI de Azure](#cli) y [Azure Portal](#portal).

>[!NOTE]
>El emparejamiento público está en desuso. No puede crear ningún emparejamiento público en los nuevos circuitos ExpressRoute. En su lugar, si tiene un nuevo circuito ExpressRoute, use el [emparejamiento de Microsoft](expressroute-circuit-peerings.md#microsoftpeering) para los servicios de Azure.
>

## <a name="connectivity"></a>Conectividad

La conectividad siempre se inicia desde la WAN a los servicios de Microsoft Azure. Servicios de Microsoft Azure no podrá iniciar conexiones en la red a través de este dominio de enrutamiento. Si el circuito ExpressRoute está habilitado para el emparejamiento público de Azure, puede acceder a los [intervalos de IP públicas que se usan en Azure](../virtual-network/virtual-network-ip-addresses-overview-arm.md#public-ip-addresses) a través del circuito.

Una vez que se habilita el emparejamiento público, puede conectarse a la mayoría de servicios de Azure. No se permite elegir de forma selectiva servicios para los que anunciemos rutas.

* Los servicios como Azure Storage, SQL Database y Websites se ofrecen en direcciones IP públicas.
* A través del dominio de enrutamiento de emparejamiento público, puede conectarse de forma privada a servicios hospedados en direcciones IP públicas, incluidas las VIP de servicios en la nube.
* Puede conectar el dominio de emparejamiento público a la red perimetral y conectarse a todos los servicios de Azure en sus direcciones IP públicas desde la WAN sin tener que conectarse a través de Internet.

## <a name="services"></a><a name="services"></a>Servicios

En esta sección se muestran los servicios disponibles a través del emparejamiento público. Dado que el emparejamiento público está en desuso, no hay ningún plan para agregar servicios nuevos o adicionales a un emparejamiento público. Si usa el emparejamiento público y el servicio que quiere usar solo es compatible con el emparejamiento de Microsoft, debe cambiar al emparejamiento de Microsoft. Consulte el [emparejamiento de Microsoft](expressroute-faqs.md#microsoft-peering) para obtener una lista de los servicios compatibles.

**Compatibles:**

* Power BI
* Se admiten la mayoría de los servicios de Azure. Compruébelo directamente con el servicio que quiere utilizar para verificar la compatibilidad.

**No compatibles:**
  * CDN
  * Azure Front Door
  * Servidor de la autenticación multifactor (heredado)
  * Traffic Manager

Para validar la disponibilidad de un servicio específico, puede consultar la documentación de ese servicio a fin de comprobar si hay un intervalo reservado publicado para ese servicio. Después, puede buscar los intervalos IP del servicio de destino y compararlos con los intervalos que se enumeran en [Etiquetas de servicio e intervalos IP de Azure: archivo XML de la nube pública](https://www.microsoft.com/download/details.aspx?id=56519). Como alternativa, puede abrir una incidencia de soporte técnico para el servicio a fin de clarificarlo.

## <a name="peering-comparison"></a><a name="compare"></a>Comparación de emparejamiento

[!INCLUDE [peering comparison](../../includes/expressroute-peering-comparison.md)]

> [!NOTE]
> El emparejamiento público de Azure tiene 1 dirección IP de NAT asociada a cada sesión BGP. Para más de 2 direcciones IP de NAT, migre al emparejamiento de Microsoft. El emparejamiento de Microsoft le permite configurar sus propias asignaciones de NAT, así como usar filtros de ruta para los anuncios de prefijo selectivo. Para obtener más información, consulte [Cambio al emparejamiento de Microsoft](https://docs.microsoft.com/azure/expressroute/how-to-move-peering).
>

## <a name="custom-route-filters"></a>Filtros de ruta personalizados

Puede definir filtros de ruta personalizados dentro de la red para usar solo las rutas que necesita. Consulte la página [Enrutamiento](expressroute-routing.md) para obtener información detallada sobre la configuración de enrutamiento.

## <a name="azure-powershell-steps"></a><a name="powershell"></a>Pasos de Azure PowerShell


[!INCLUDE [CloudShell](../../includes/expressroute-cloudshell-powershell-about.md)]

Dado que el emparejamiento público está en desuso, no se puede configurar en un nuevo circuito ExpressRoute.

1. Verifique que tenga un circuito ExpressRoute que esté aprovisionado y también habilitado. Utilice el ejemplo siguiente:

   ```azurepowershell-interactive
   Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"
   ```

   La respuesta es similar al siguiente ejemplo:

   ```
   Name                             : ExpressRouteARMCircuit
   ResourceGroupName                : ExpressRouteResourceGroup
   Location                         : westus
   Id                               : /subscriptions/***************************/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
   Etag                             : W/"################################"
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
                                       "ServiceProviderName": "Equinix",
                                       "PeeringLocation": "Silicon Valley",
                                       "BandwidthInMbps": 200
                                     }
   ServiceKey                       : **************************************
   Peerings                         : []
   ```
2. Establezca la configuración del emparejamiento publico de Azure para el circuito. Asegúrese de que tiene la siguiente información antes de continuar.

   * Una subred /30 para el vínculo principal. Tiene que ser un prefijo IPv4 público válido.
   * Una subred /30 para el vínculo secundario. Tiene que ser un prefijo IPv4 público válido.
   * Un identificador VLAN válido para establecer esta configuración entre pares. Asegúrese de que ninguna otra configuración entre pares en el circuito usa el mismo identificador de VLAN.
   * Número de sistema autónomo (AS) para la configuración entre pares. Puede usar 2 bytes o 4 bytes como números AS.
   * Opcional:
   * Un hash MD5, en caso de que haya decidido usarlo.

   Ejecute el siguiente ejemplo para configurar el emparejamiento público de Azure para su circuito:

   ```azurepowershell-interactive
   Add-AzExpressRouteCircuitPeeringConfig -Name "AzurePublicPeering" -ExpressRouteCircuit $ckt -PeeringType AzurePublicPeering -PeerASN 100 -PrimaryPeerAddressPrefix "12.0.0.0/30" -SecondaryPeerAddressPrefix "12.0.0.4/30" -VlanId 100

   Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
   ```

   Si decide usar un hash MD5, use el ejemplo siguiente:

   ```azurepowershell-interactive
   Add-AzExpressRouteCircuitPeeringConfig -Name "AzurePublicPeering" -ExpressRouteCircuit $ckt -PeeringType AzurePublicPeering -PeerASN 100 -PrimaryPeerAddressPrefix "12.0.0.0/30" -SecondaryPeerAddressPrefix "12.0.0.4/30" -VlanId 100  -SharedKey "A1B2C3D4"

   Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
   ```

   > [!IMPORTANT]
   > Asegúrese de especificar su número AS como ASN de configuración entre pares, no como cliente ASN.
   > 
   >

### <a name="to-get-azure-public-peering-details"></a><a name="getpublic"></a>Obtención de detalles de un emparejamiento público de Azure

Puede obtener detalles sobre la configuración mediante el siguiente cmdlet:

```azurepowershell-interactive
  $ckt = Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

  Get-AzExpressRouteCircuitPeeringConfig -Name "AzurePublicPeering" -Circuit $ckt
  ```

### <a name="to-update-azure-public-peering-configuration"></a><a name="updatepublic"></a>Actualización del establecimiento de configuración del emparejamiento público de Azure

Puede actualizar cualquier parte de la configuración mediante el ejemplo siguiente. En este ejemplo, se va a actualizar el identificador de VLAN del circuito de 200 a 600.

```azurepowershell-interactive
Set-AzExpressRouteCircuitPeeringConfig  -Name "AzurePublicPeering" -ExpressRouteCircuit $ckt -PeeringType AzurePublicPeering -PeerASN 100 -PrimaryPeerAddressPrefix "123.0.0.0/30" -SecondaryPeerAddressPrefix "123.0.0.4/30" -VlanId 600

Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
```

### <a name="to-delete-azure-public-peering"></a><a name="deletepublic"></a>Eliminación del emparejamiento público de Azure

Puede quitar la configuración de emparejamiento mediante la ejecución del ejemplo siguiente:

```azurepowershell-interactive
Remove-AzExpressRouteCircuitPeeringConfig -Name "AzurePublicPeering" -ExpressRouteCircuit $ckt
Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
```

## <a name="azure-cli-steps"></a><a name="cli"></a>Pasos de la CLI de Azure


[!INCLUDE [CloudShell](../../includes/expressroute-cloudshell-powershell-about.md)]

1. Compruebe el circuito ExpressRoute para asegurarse de que está aprovisionado y también habilitado. Utilice el ejemplo siguiente:

   ```azurecli-interactive
   az network express-route list
   ```

   La respuesta es similar al siguiente ejemplo:

   ```output
   "allowClassicOperations": false,
   "authorizations": [],
   "circuitProvisioningState": "Enabled",
   "etag": "W/\"1262c492-ffef-4a63-95a8-a6002736b8c4\"",
   "gatewayManagerEtag": null,
   "id": "/subscriptions/81ab786c-56eb-4a4d-bb5f-f60329772466/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/MyCircuit",
   "location": "westus",
   "name": "MyCircuit",
   "peerings": [],
   "provisioningState": "Succeeded",
   "resourceGroup": "ExpressRouteResourceGroup",
   "serviceKey": "1d05cf70-1db5-419f-ad86-1ca62c3c125b",
   "serviceProviderNotes": null,
   "serviceProviderProperties": {
    "bandwidthInMbps": 200,
    "peeringLocation": "Silicon Valley",
    "serviceProviderName": "Equinix"
   },
   "serviceProviderProvisioningState": "Provisioned",
   "sku": {
    "family": "UnlimitedData",
    "name": "Standard_MeteredData",
    "tier": "Standard"
   },
   "tags": null,
   "type": "Microsoft.Network/expressRouteCircuits]
   ```

2. Establezca la configuración del emparejamiento publico de Azure para el circuito. Asegúrese de que tiene la siguiente información antes de continuar.

   * Una subred /30 para el vínculo principal. Tiene que ser un prefijo IPv4 público válido.
   * Una subred /30 para el vínculo secundario. Tiene que ser un prefijo IPv4 público válido.
   * Un identificador VLAN válido para establecer esta configuración entre pares. Asegúrese de que ninguna otra configuración entre pares en el circuito usa el mismo identificador de VLAN.
   * Número de sistema autónomo (AS) para la configuración entre pares. Puede usar 2 bytes o 4 bytes como números AS.
   * **Opcional:** un hash MD5 si elige usar uno.

   Ejecute el siguiente ejemplo para configurar el emparejamiento público de Azure para su circuito:

   ```azurecli-interactive
   az network express-route peering create --circuit-name MyCircuit --peer-asn 100 --primary-peer-subnet 12.0.0.0/30 -g ExpressRouteResourceGroup --secondary-peer-subnet 12.0.0.4/30 --vlan-id 200 --peering-type AzurePublicPeering
   ```

   Si decide usar un hash MD5, use el ejemplo siguiente:

   ```azurecli-interactive
   az network express-route peering create --circuit-name MyCircuit --peer-asn 100 --primary-peer-subnet 12.0.0.0/30 -g ExpressRouteResourceGroup --secondary-peer-subnet 12.0.0.4/30 --vlan-id 200 --peering-type AzurePublicPeering --SharedKey "A1B2C3D4"
   ```

   > [!IMPORTANT]
   > Asegúrese de especificar su número AS como ASN de configuración entre pares, no como cliente ASN.

### <a name="to-view-azure-public-peering-details"></a><a name="getpublic"></a>Visualización de detalles de un emparejamiento público de Azure

Puede obtener detalles sobre la configuración mediante el ejemplo siguiente:

```azurecli
az network express-route peering show -g ExpressRouteResourceGroup --circuit-name MyCircuit --name AzurePublicPeering
```

La salida es similar a la del ejemplo siguiente:

```output
{
  "azureAsn": 12076,
  "etag": "W/\"2e97be83-a684-4f29-bf3c-96191e270666\"",
  "gatewayManagerEtag": "18",
  "id": "/subscriptions/9a0c2943-e0c2-4608-876c-e0ddffd1211b/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/MyCircuit/peerings/AzurePublicPeering",
  "lastModifiedBy": "Customer",
  "microsoftPeeringConfig": null,
  "name": "AzurePublicPeering",
  "peerAsn": 7671,
  "peeringType": "AzurePublicPeering",
  "primaryAzurePort": "",
  "primaryPeerAddressPrefix": "",
  "provisioningState": "Succeeded",
  "resourceGroup": "ExpressRouteResourceGroup",
  "routeFilter": null,
  "secondaryAzurePort": "",
  "secondaryPeerAddressPrefix": "",
  "sharedKey": null,
  "state": "Enabled",
  "stats": null,
  "vlanId": 100
}
```

### <a name="to-update-azure-public-peering-configuration"></a><a name="updatepublic"></a>Actualización del establecimiento de configuración del emparejamiento público de Azure

Puede actualizar cualquier parte de la configuración mediante el ejemplo siguiente. En este ejemplo, se va a actualizar el identificador de VLAN del circuito de 200 a 600.

```azurecli-interactive
az network express-route peering update --vlan-id 600 -g ExpressRouteResourceGroup --circuit-name MyCircuit --name AzurePublicPeering
```

### <a name="to-delete-azure-public-peering"></a><a name="deletepublic"></a>Eliminación del emparejamiento público de Azure

Puede quitar la configuración de emparejamiento mediante la ejecución del ejemplo siguiente:

```azurecli-interactive
az network express-route peering delete -g ExpressRouteResourceGroup --circuit-name MyCircuit --name AzurePublicPeering
```

## <a name="azure-portal-steps"></a><a name="portal"></a>Pasos de Azure Portal

Para configurar el emparejamiento, siga los pasos de PowerShell o de la CLI que se incluyen en este artículo. Para administrar un emparejamiento, puede usar las secciones siguientes. Como referencia, estos pasos son similares a la administración de un [emparejamiento de Microsoft en el portal](expressroute-howto-routing-portal-resource-manager.md#msft).

### <a name="to-view-azure-public-peering-details"></a><a name="get"></a>Visualización de detalles de un emparejamiento público de Azure

Para ver las propiedades de un emparejamiento público de Azure, seleccione el emparejamiento en el portal.

### <a name="to-update-azure-public-peering-configuration"></a><a name="update"></a>Actualización del establecimiento de configuración del emparejamiento público de Azure

Seleccione la fila del emparejamiento y modifique las propiedades de este.

### <a name="to-delete-azure-public-peering"></a><a name="delete"></a>Eliminación del emparejamiento público de Azure

Para quitar la configuración de emparejamiento, seleccione el icono de eliminación.

## <a name="next-steps"></a>Pasos siguientes

El siguiente paso consiste en [vincular una red virtual a un circuito ExpressRoute](expressroute-howto-linkvnet-arm.md).

* Para más información sobre los flujos de trabajo de ExpressRoute, vea [Flujos de trabajo de ExpressRoute](expressroute-workflows.md).
* Para más información sobre el emparejamiento de circuitos, vea [Circuitos y dominios de enrutamiento de ExpressRoute](expressroute-circuit-peerings.md).
* Para más información sobre redes virtuales, vea [Información general sobre redes virtuales](../virtual-network/virtual-networks-overview.md).