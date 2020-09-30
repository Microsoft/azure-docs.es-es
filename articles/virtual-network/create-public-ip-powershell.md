---
title: 'Creación de una dirección IP pública: Azure PowerShell'
description: Aprenda a crear una dirección IP pública mediante Azure PowerShell
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
ms.openlocfilehash: 6174d108fd80df9725ca5ef0fb9296dfffaf4a64
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89301032"
---
# <a name="quickstart-create-a-public-ip-address-using-azure-powershell"></a>Inicio rápido: Creación de una dirección IP pública mediante Azure PowerShell

En este artículo se muestra cómo crear un recurso de dirección IP pública mediante Azure PowerShell. Para más información sobre los recursos que se pueden asociar, la diferencia entre la SKU básica y estándar y otra información relacionada, consulte [Direcciones IP públicas](https://docs.microsoft.com/azure/virtual-network/public-ip-addresses).  En este ejemplo, nos centraremos solo en las direcciones IPv4. Para más información sobre las direcciones IPv6, consulte [IPv6 para la red virtual de Azure](https://docs.microsoft.com/azure/virtual-network/ipv6-overview).

## <a name="prerequisites"></a>Requisitos previos

- Azure PowerShell instalado localmente o Azure Cloud Shell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar PowerShell de forma local, para realizar los pasos de este artículo necesita la versión 5.4.1 del módulo de Azure PowerShell o cualquier versión posterior. Ejecute `Get-Module -ListAvailable Az` para buscar la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-Az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure.

Cree un grupo de recursos con [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) con el nombre **myResourceGroup** en la ubicación **eastus2**.

```azurepowershell-interactive
## Variables for the command ##
$rg = 'myResourceGroup'
$loc = 'eastus2'

New-AzResourceGroup -Name $rg -Location $loc
```
---
# <a name="standard-sku---using-zones"></a>[**SKU estándar: uso de zonas**](#tab/option-create-public-ip-standard-zones)

>[!NOTE]
>El siguiente comando funciona con la versión de API 2020-08-01 o posterior.  Para más información sobre la versión de API que se usa actualmente, consulte [Tipos y proveedores de recursos](https://docs.microsoft.com/azure/azure-resource-manager/management/resource-providers-and-types).

Use [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) para crear una dirección IP pública estándar con redundancia de zona llamada **myStandardZRPublicIP** en **myResourceGroup**.

```azurepowershell-interactive
## Variables for the command ##
$rg = 'myResourceGroup'
$loc = 'eastus2'
$pubIP = 'myStandardZRPublicIP'
$sku = 'Standard'
$alloc = 'Static'
$zone = 1,2,3

New-AzPublicIpAddress -ResourceGroupName $rg -Name $pubIP -Location $loc -AllocationMethod $alloc -SKU $sku -zone $zone
```
> [!IMPORTANT]
> En el caso de versiones de la API anteriores a 2020-08-01, ejecute el comando anterior sin especificar un parámetro de zona para crear una dirección IP con redundancia de zona. 
>

Para crear una dirección IP pública estándar de zona en la Zona 2 llamada **myStandardZonalPublicIP** en **myResourceGroup**, use el siguiente comando:

```azurepowershell-interactive
## Variables for the command ##
$rg = 'myResourceGroup'
$loc = 'eastus2'
$pubIP = 'myStandardZonalPublicIP'
$sku = 'Standard'
$alloc = 'Static'
$zone = 2

New-AzPublicIpAddress -ResourceGroupName $rg -Name $pubIP -Location $loc -AllocationMethod $alloc -SKU $sku -zone $zone
```

Tenga en cuenta que las opciones anteriores para zonas son solo selecciones válidas en regiones con [Availability Zones](https://docs.microsoft.com/azure/availability-zones/az-overview?toc=/azure/virtual-network/toc.json#availability-zones).

# <a name="standard-sku---no-zones"></a>[**SKU estándar: sin zonas**](#tab/option-create-public-ip-standard)

>[!NOTE]
>El siguiente comando funciona con la versión de API 2020-08-01 o posterior.  Para más información sobre la versión de API que se usa actualmente, consulte [Tipos y proveedores de recursos](https://docs.microsoft.com/azure/azure-resource-manager/management/resource-providers-and-types).

Use [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) para crear una dirección IP pública estándar como un recurso no de zona llamada **myStandardPublicIP** en **myResourceGroup**.

```azurepowershell-interactive
## Variables for the command ##
$rg = 'myResourceGroup'
$loc = 'eastus2'
$pubIP = 'myStandardPublicIP'
$sku = 'Standard'
$alloc = 'Static'

New-AzPublicIpAddress -ResourceGroupName $rg -Name $pubIP -Location $loc -AllocationMethod $alloc -SKU $sku
```

Esta selección es válida en todas las regiones y es la selección predeterminada para las direcciones IP públicas estándar en regiones sin [Availability Zones](https://docs.microsoft.com/azure/availability-zones/az-overview?toc=/azure/virtual-network/toc.json#availability-zones).

# <a name="basic-sku"></a>[**SKU básica**](#tab/option-create-public-ip-basic)

Use [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) para crear una dirección IP pública estática básica llamada **myStandardZRPublicIP** en **myResourceGroup**.  Las direcciones IP públicas básicas no tienen el concepto de zonas de disponibilidad.

```azurepowershell-interactive
## Variables for the command ##
$rg = 'myResourceGroup'
$loc = 'eastus2'
$pubIP = 'myBasicPublicIP'
$sku = 'Basic'
$alloc = 'Static'

New-AzPublicIpAddress -ResourceGroupName $rg -Name $pubIP -Location $loc -AllocationMethod $alloc -SKU $sku
```
Si es admisible que la dirección IP cambie con el tiempo, se puede seleccionar la asignación de IP **dinámica** cambiando el valor de AllocationMethod a "Dynamic".

---

## <a name="additional-information"></a>Información adicional 

Para más información sobre las variables individuales enumeradas anteriormente, consulte [Administración de direcciones IP públicas](https://docs.microsoft.com/azure/virtual-network/virtual-network-public-ip-address#create-a-public-ip-address).

## <a name="next-steps"></a>Pasos siguientes
- Asocie una [dirección IP pública a una máquina virtual](https://docs.microsoft.com/azure/virtual-network/associate-public-ip-address-vm#azure-portal).
- Obtenga más información acerca de las [direcciones IP públicas](virtual-network-ip-addresses-overview-arm.md#public-ip-addresses) en Azure.
- Obtenga más información acerca de toda la [configuración de direcciones IP públicas](virtual-network-public-ip-address.md#create-a-public-ip-address).