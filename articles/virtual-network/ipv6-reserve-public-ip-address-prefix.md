---
title: Reserva de direcciones IPv6 públicas e intervalos de direcciones en una red virtual de Azure
titlesuffix: Azure Virtual Network
description: Aprenda a reservar direcciones IPv6 públicas e intervalos de direcciones en una red virtual de Azure.
services: virtual-network
documentationcenter: na
author: KumudD
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/31/2020
ms.author: kumud
ms.openlocfilehash: ac79e1eb5c4f7448dc17804cd8aac3cba582497e
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88509950"
---
# <a name="reserve-public-ipv6-address-prefix"></a>Reserva de un prefijo de dirección IPv6 pública
IPv6 para Azure Virtual Network (VNet) le permite hospedar aplicaciones en Azure con la conectividad IPv6 e IPv4, tanto en una red virtual como hacia y desde Internet. Además de reservar direcciones IPv6 individuales, puede reservar intervalos contiguos de direcciones IPv6 de Azure (conocidos como prefijo IP) para su uso. En este artículo se describe cómo crear IP públicas IPv6 e intervalos de direcciones mediante Azure PowerShell y la CLI de Azure.


## <a name="create-a-single-reserved-ipv6-public-ip"></a>Creación de una IP pública IPv6 reservada única

### <a name="using-azure-powershell"></a>Uso de Azure PowerShell

Puede crear una IP pública IPv6 reservada (estática) única con Azure PowerShell con [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress), como se indica a continuación:

```azurepowershell
 $myOwnIPv6Address = New-AzPublicIpAddress `
 -name PIPv6_WestUS `
 -ResourceGroup MyRG `
 -Location "West US" `
 -Sku Standard `
 -allocationMethod static `
 -IpAddressVersion IPv6
 ```

### <a name="using-azure-cli"></a>Uso de la CLI de Azure

 Puede crear una IP pública IPv6 reservada (estática) única con la CLI de Azure con [az network public-ip create](/cli/azure/network/public-ip), como se indica a continuación:

```azurecli
 az network public-ip create \
 --name dsPublicIP_v6 \
 --resource-group UpgradeInPlace_CLI_RG1 \
 --location WestUS \
 --sku Standard  \
 --allocation-method static  \
 --version IPv6
```

## <a name="create-a-reserved-ipv6-prefix-range"></a>Creación de un prefijo IPv6 reservado (intervalo)

Para reservar un prefijo IPv6, agregue la familia de direcciones IP de IPv6 al mismo comando que se usa para crear prefijos IPv4. Los comandos siguientes crean un prefijo de tamaño /125 (8 direcciones IPv6).

### <a name="using-azure-powershell"></a>Uso de Azure PowerShell

Puede crear una dirección IPv6 pública con la CLI de Azure con [az network public-ip create](/powershell/module/az.network/new-azpublicipprefix), como se indica a continuación:
```azurepowershell
 $myOwnIPv6Prefix = New-AzPublicIpPrefix `
 -name IPv6PrefixWestUS `
 -ResourceGroupName MyRG `
 -Location "West US" `
 -Sku Standard `
 -IpAddressVersion IPv6 `
 -PrefixLength 125
```

### <a name="using-azure-cli"></a>Uso de la CLI de Azure

Puede crear una dirección IPv6 pública con la CLI de Azure como se indica a continuación:

```azurecli
az network public-ip prefix create \
--name IPv6PrefixWestUS \
--resource-group MyRG \
--location WestUS \
--version IPv6 \
--length 125
```

## <a name="allocate-a-public-ip-address-from-a-reserved-ipv6-prefix"></a>Asignación de una IP pública desde un prefijo IPv6 reservado

### <a name="using-azure-powershell"></a>Uso de Azure PowerShell

 Puede crear una IP pública IPv6 estática a partir de un prefijo reservado si agrega el argumento `-PublicIpPrefix` al crear la IP pública mediante Azure PowerShell. En el ejemplo siguiente se presupone que se creó un prefijo y se almacenó en una variable de PowerShell denominada: *$myOwnIPv 6Prefix*.

```azurepowershell
 $MyIPv6PublicIPFromMyReservedPrefix = New-AzPublicIpAddress \
 -name PIPv6_fromPrefix `
 -ResourceGroup DsStdLb04 `
 -Location "West Central US" `
 -Sku Standard `
 -allocationMethod static `
 -IpAddressVersion IPv6 `
 -PublicIpPrefix $myOwnIPv6Prefix
```

### <a name="using-azure-cli"></a>Uso de la CLI de Azure

En el ejemplo siguiente se presupone que se creó un prefijo y se almacenó en una variable de CLI denominada: *IPv6PrefixWestUS*.

```azurecli
az network public-ip create \
--name dsPublicIP_v6 \
--resource-group UpgradeInPlace_CLI_RG1 \
--location WestUS \
--sku Standard \
--allocation-method static \
--version IPv6 \
--public-ip-prefix  IPv6PrefixWestUS
```

## <a name="next-steps"></a>Pasos siguientes
- Obtenga más información sobre el [prefijo de dirección IPv6](ipv6-public-ip-address-prefix.md).
- Obtenga más información sobre las [direcciones IPv6](ipv6-overview.md).
