---
title: 'Creación de una instancia de Azure VPN Gateway basada en la ruta: CLI'
description: Cree rápidamente una instancia de Azure VPN Gateway basada en rutas mediante la CLI de Azure, para una conexión VPN a una red local o para conectar redes virtuales.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/02/2020
ms.author: cherylmc
ms.openlocfilehash: e43f324a8c32615eded782e75291b575852c4382
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89393758"
---
# <a name="create-a-route-based-vpn-gateway-using-cli"></a>Creación de una instancia de VPN Gateway basada en rutas mediante la CLI

En este artículo se explica cómo crear rápidamente una instancia de Azure VPN Gateway basada en rutas mediante la CLI de Azure. Una instancia de VPN Gateway se usa al crear una conexión VPN a la red local. También puede utilizar una instancia de VPN Gateway para conectar redes virtuales.

Los pasos que se describen en este artículo crearán una red virtual, una subred, una subred de puerta de enlace y una instancia de VPN Gateway basada en rutas (puerta de enlace de red virtual). Una puerta de enlace de red virtual puede tardar 45 minutos o más en crearse. Una vez completada la creación de la puerta de enlace, puede crear las conexiones. Estos pasos requieren una suscripción a Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI localmente, para este artículo es preciso que ejecute la versión 2.0.4 o posterior de la CLI de Azure. Ejecute `az --version` para ver cuál es la versión instalada. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Cree un grupo de recursos con el comando [az group create](/cli/azure/group). Un grupo de recursos es un contenedor lógico en el que se implementan y se administran los recursos de Azure. 


```azurecli-interactive
az group create --name TestRG1 --location eastus
```

## <a name="create-a-virtual-network"></a><a name="vnet"></a>Creación de una red virtual

Cree una red virtual con el comando [az network vnet create](/cli/azure/network/vnet). En el siguiente ejemplo se crea una red virtual denominada **VNet1** en la ubicación **EastUS**:

```azurecli-interactive
az network vnet create \
  -n VNet1 \
  -g TestRG1 \
  -l eastus \
  --address-prefix 10.1.0.0/16 \
  --subnet-name Frontend \
  --subnet-prefix 10.1.0.0/24
```

## <a name="add-a-gateway-subnet"></a><a name="gwsubnet"></a>Incorporación de una subred de puerta de enlace

La subred de puerta de enlace contiene las direcciones IP reservadas que usan los servicios de puerta de enlace de la red virtual. Utilice los siguientes ejemplos para agregar una subred de puerta de enlace:

```azurecli-interactive
az network vnet subnet create \
  --vnet-name VNet1 \
  -n GatewaySubnet \
  -g TestRG1 \
  --address-prefix 10.1.255.0/27 
```

## <a name="request-a-public-ip-address"></a><a name="PublicIP"></a>Solicitud de una dirección IP pública

Una instancia de VPN Gateway debe tener una dirección IP pública asignada de forma dinámica. La dirección IP pública se asignará a la instancia de VPN Gateway creada para la red virtual. Utilice el siguiente ejemplo para solicitar una dirección IP pública:

```azurecli-interactive
az network public-ip create \
  -n VNet1GWIP \
  -g TestRG1 \
  --allocation-method Dynamic 
```

## <a name="create-the-vpn-gateway"></a><a name="CreateGateway"></a>Creación de la puerta de enlace de VPN

Cree la puerta de enlace VPN con el comando [az network vnet-gateway create](/cli/azure/group).

Si este comando se ejecuta con el parámetro `--no-wait`, no se verán los comentarios o resultados. Este parámetro `--no-wait` permite que la puerta de enlace se cree en segundo plano. No significa que la puerta de enlace de VPN se cree de inmediato.

```azurecli-interactive
az network vnet-gateway create \
  -n VNet1GW \
  -l eastus \
  --public-ip-address VNet1GWIP \
  -g TestRG1 \
  --vnet VNet1 \
  --gateway-type Vpn \
  --sku VpnGw1 \
  --vpn-type RouteBased \
  --no-wait
```

Una instancia de VPN Gateway puede tardar 45 minutos o más en crearse.

## <a name="view-the-vpn-gateway"></a><a name="viewgw"></a>Visualización de VPN Gateway

```azurecli-interactive
az network vnet-gateway show \
  -n VNet1GW \
  -g TestRG1
```

La respuesta será similar a la siguiente:

```output
{
  "activeActive": false,
  "bgpSettings": null,
  "enableBgp": false,
  "etag": "W/\"6c61f8cb-d90f-4796-8697\"",
  "gatewayDefaultSite": null,
  "gatewayType": "Vpn",
  "id": "/subscriptions/<subscription ID>/resourceGroups/TestRG11/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW",
  "ipConfigurations": [
    {
      "etag": "W/\"6c61f8cb-d90f-4796-8697\"",
      "id": "/subscriptions/<subscription ID>/resourceGroups/TestRG11/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW/ipConfigurations/vnetGatewayConfig0",
      "name": "vnetGatewayConfig0",
      "privateIpAllocationMethod": "Dynamic",
      "provisioningState": "Updating",
      "publicIpAddress": {
        "id": "/subscriptions/<subscription ID>/resourceGroups/TestRG11/providers/Microsoft.Network/publicIPAddresses/VNet1GWIP",
        "resourceGroup": "TestRG1"
      },
      "resourceGroup": "TestRG1",
      "subnet": {
        "id": "/subscriptions/<subscription ID>/resourceGroups/TestRG11/providers/Microsoft.Network/virtualNetworks/VNet1/subnets/GatewaySubnet",
        "resourceGroup": "TestRG1"
      }
    }
  ],
  "location": "eastus",
  "name": "VNet1GW",
  "provisioningState": "Updating",
  "resourceGroup": "TestRG1",
  "resourceGuid": "69c269e3-622c-4123-9231",
  "sku": {
    "capacity": 2,
    "name": "VpnGw1",
    "tier": "VpnGw1"
  },
  "tags": null,
  "type": "Microsoft.Network/virtualNetworkGateways",
  "vpnClientConfiguration": null,
  "vpnType": "RouteBased"
}
```

### <a name="view-the-public-ip-address"></a>Visualización de la dirección IP pública

Para ver la dirección IP pública asignada a la puerta de enlace, use el ejemplo siguiente:

```azurecli-interactive
az network public-ip show \
  --name VNet1GWIP \
  --resource-group TestRG11
```

El valor asociado al campo **ipAddress** es la dirección IP pública de la puerta de enlace de VPN.

Respuesta de ejemplo:

```output
{
  "dnsSettings": null,
  "etag": "W/\"a12d4d03-b27a-46cc-b222-8d9364b8166a\"",
  "id": "/subscriptions/<subscription ID>/resourceGroups/TestRG1/providers/Microsoft.Network/publicIPAddresses/VNet1GWIP",
  "idleTimeoutInMinutes": 4,
  "ipAddress": "13.90.195.184",
  "ipConfiguration": {
    "etag": null,
    "id": "/subscriptions/<subscription ID>/resourceGroups/TestRG1/providers/Microsoft.Network/virtualNetworkGateways/VNet1GW/ipConfigurations/vnetGatewayConfig0",
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite los recursos que ha creado, use [az group delete](/cli/azure/group) para eliminar el grupo de recursos. Se eliminarán el grupo de recursos y todos los recursos que contiene.

```azurecli-interactive 
az group delete --name TestRG1 --yes
```

## <a name="next-steps"></a>Pasos siguientes

Una vez creada la puerta de enlace, puede crear una conexión entre su red virtual y otra red virtual. O bien, cree una conexión entre su red virtual y una ubicación local.

> [!div class="nextstepaction"]
> [Creación de una conexión de sitio a sitio](vpn-gateway-create-site-to-site-rm-powershell.md)<br><br>
> [Creación de una conexión de punto a sitio](vpn-gateway-howto-point-to-site-rm-ps.md)<br><br>
> [Creación de una conexión a otra red virtual](vpn-gateway-vnet-vnet-rm-ps.md)
