---
title: 'Creación de una dirección IP pública: CLI de Azure'
description: Obtenga información sobre cómo crear una dirección IP pública mediante la CLI de Azure
services: virtual-network
documentationcenter: na
author: blehr
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/28/2020
ms.author: blehr
ms.openlocfilehash: 4d0934dd30f9738e2a67d4aff23c96a48aaaa61b
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89301069"
---
# <a name="quickstart-create-a-public-ip-address-using-azure-cli"></a>Inicio rápido: Creación de una dirección IP pública mediante la CLI de Azure

En este artículo se muestra cómo crear un recurso de dirección IP pública mediante la CLI de Azure. Para más información sobre los recursos que se pueden asociar, la diferencia entre la SKU básica y estándar y otra información relacionada, consulte [Direcciones IP públicas](https://docs.microsoft.com/azure/virtual-network/public-ip-addresses).  En este ejemplo, nos centraremos solo en las direcciones IPv4. Para más información sobre las direcciones IPv6, consulte [IPv6 para la red virtual de Azure](https://docs.microsoft.com/azure/virtual-network/ipv6-overview).

## <a name="prerequisites"></a>Requisitos previos

- CLI de Azure instalada localmente o Azure Cloud Shell

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)] 

Si decide instalar y usar la CLI localmente, para esta guía de inicio rápido se necesita la versión 2.0.28 de la CLI de Azure o una versión posterior. Para encontrar la versión, ejecute `az --version`. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure.

Cree un grupo de recursos con [az group create](https://docs.microsoft.com/cli/azure/group?view=azure-cli-latest#az-group-create) llamado **myResourceGroup** en la ubicación **eastus2**.

```azurecli-interactive
  az group create \
    --name myResourceGroup \
    --location eastus2
```
---
# <a name="standard-sku---using-zones"></a>[**SKU estándar: uso de zonas**](#tab/option-create-public-ip-standard-zones)

>[!NOTE]
>El siguiente comando funciona con la versión de API 2020-08-01 o posterior.  Para más información sobre la versión de API que se usa actualmente, consulte [Tipos y proveedores de recursos](https://docs.microsoft.com/azure/azure-resource-manager/management/resource-providers-and-types).

Use [az network public-ip create](https://docs.microsoft.com/cli/azure/network/public-ip?view=azure-cli-latest#az-network-public-ip-create) para crear una dirección IP pública estándar y con redundancia de zona denominada **myStandardZRPublicIP** en **myResourceGroup**.

```azurecli-interactive
  az network public-ip create \
    --resource-group myResourceGroup \
    --name myStandardZRPublicIP \
    --sku Standard
    --zone 1,2,3
```
> [!IMPORTANT]
> En el caso de versiones de la API anteriores a 2020-08-01, ejecute el comando anterior sin especificar un parámetro de zona para crear una dirección IP con redundancia de zona. 
>

Para crear una dirección IP pública zonal estándar en la Zona 2 denominada **myStandardZonalPublicIP** en **myResourceGroup**, use el siguiente comando:

```azurecli-interactive
  az network public-ip create \
    --resource-group myResourceGroupLB \
    --name myStandardZonalPublicIP \
    --sku Standard \
    --zone 2
```

Tenga en cuenta que las opciones anteriores para las zonas son solo selecciones válidas en regiones con [zonas de disponibilidad](https://docs.microsoft.com/azure/availability-zones/az-overview?toc=/azure/virtual-network/toc.json#availability-zones).

# <a name="standard-sku---no-zones"></a>[**SKU estándar: sin zonas**](#tab/option-create-public-ip-standard)

>[!NOTE]
>El siguiente comando funciona con la versión de API 2020-08-01 o posterior.  Para más información sobre la versión de API que se usa actualmente, consulte [Tipos y proveedores de recursos](https://docs.microsoft.com/azure/azure-resource-manager/management/resource-providers-and-types).

Use [az network public-ip create](https://docs.microsoft.com/cli/azure/network/public-ip?view=azure-cli-latest#az-network-public-ip-create) para crear una dirección IP pública estándar como un recurso no de zona llamado **myStandardZRPublicIP** en **myResourceGroup**.

```azurecli-interactive
  az network public-ip create \
    --resource-group myResourceGroup \
    --name myStandardPublicIP \
    --sku Standard
```
Esta selección es válida en todas las regiones y es la selección predeterminada para las direccionesIP públicas estándar en regiones sin [zonas de disponibilidad](https://docs.microsoft.com/azure/availability-zones/az-overview?toc=/azure/virtual-network/toc.json#availability-zones).

# <a name="basic-sku"></a>[**SKU básica**](#tab/option-create-public-ip-basic)

Utilice [az network public-ip create](https://docs.microsoft.com/cli/azure/network/public-ip?view=azure-cli-latest#az-network-public-ip-create) para crear una dirección IP pública estática básica llamada **myBasicPublicIP** en **myResourceGroup**.  Las direcciones IP públicas básicas no tienen el concepto de zonas de disponibilidad.

```azurecli-interactive
  az network public-ip create \
    --resource-group myResourceGroup \
    --name myBasicPublicIP \
    --sku Standard
    --allocation-method Static
```
Si es aceptable que la dirección IP cambie con el tiempo, se puede seleccionar la asignación de dirección IP **dinámica** mediante el cambio del método de asignación a "Dinámico".

---

## <a name="additional-information"></a>Información adicional 

Para más información sobre las variables individuales enumeradas anteriormente, consulte [Creación, modificación o eliminación de una dirección IP pública](https://docs.microsoft.com/azure/virtual-network/virtual-network-public-ip-address#create-a-public-ip-address).

## <a name="next-steps"></a>Pasos siguientes
- Asocie una [dirección IP pública a una máquina virtual](https://docs.microsoft.com/azure/virtual-network/associate-public-ip-address-vm#azure-portal).
- Obtenga más información acerca de las [direcciones IP públicas](virtual-network-ip-addresses-overview-arm.md#public-ip-addresses) en Azure.
- Obtenga más información acerca de toda la [configuración de direcciones IP públicas](virtual-network-public-ip-address.md#create-a-public-ip-address).