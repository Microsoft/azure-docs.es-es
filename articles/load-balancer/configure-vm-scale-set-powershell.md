---
title: 'Configuración de un conjunto de escalado de máquinas virtuales con una instancia existente de Azure Load Balancer: Azure PowerShell'
description: Aprenda a configurar un conjunto de escalado de máquinas virtuales con una instancia existente de Azure Load Balancer.
author: asudbring
ms.author: allensu
ms.service: load-balancer
ms.topic: how-to
ms.date: 03/26/2020
ms.openlocfilehash: 919883025075ca38bcef6c23ac744528e7bd5502
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88510069"
---
# <a name="configure-a-virtual-machine-scale-set-with-an-existing-azure-load-balancer-using-azure-powershell"></a>Configuración de un conjunto de escalado de máquinas virtuales con una instancia existente de Azure Load Balancer mediante Azure PowerShell

En este artículo, aprenderá a configurar un conjunto de escalado de máquinas virtuales con una instancia existente de Azure Load Balancer.

## <a name="prerequisites"></a>Prerrequisitos

- Suscripción a Azure.
- Un equilibrador de carga de SKU estándar existente en la suscripción donde se implementará el conjunto de escalado de máquinas virtuales.
- Una instancia de Azure Virtual Network para el conjunto de escalado de máquinas virtuales.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="sign-in-to-azure-cli"></a>Inicio de sesión en la CLI de Azure

Inicie sesión en Azure.

```azurepowershell-interactive
Connect-AzAccount
```

## <a name="deploy-a-virtual-machine-scale-set-with-existing-load-balancer"></a>Implementación de un conjunto de escalado de máquinas virtuales con un equilibrador de carga existente

Reemplace los valores entre corchetes por los nombres de los recursos en la configuración.

```azurepowershell-interactive

$rsg = <resource-group>
$loc = <location>
$vms = <vm-scale-set-name>
$vnt = <virtual-network>
$sub = <subnet-name>
$lbn = <load-balancer-name>
$pol = <upgrade-policy-mode>

$lb = Get-AzLoadBalancer -ResourceGroupName $rsg -Name $lbn

New-AzVmss -ResourceGroupName $rsg -Location $loc -VMScaleSetName $vms -VirtualNetworkName $vnt -SubnetName $sub -LoadBalancerName $lb -UpgradePolicyMode $pol

```

En el ejemplo siguiente se implementa un conjunto de escalado de máquinas virtuales con:

- Un conjunto de escalado de máquinas virtuales denominado **myVMSS**
- Una instancia de Azure Load Balancer denominada **myLoadBalancer**
- Un grupo de back-end de equilibradores de carga denominado **myBackendPool**
- Una instancia de Azure Virtual Network denominada **myVnet**
- Una subred denominada **mySubnet**
- Un grupo de recursos denominado **myResourceGroup**

```azurepowershell-interactive

$rsg = "myResourceGroup"
$loc = "East US 2"
$vms = "myVMSS"
$vnt = "myVnet"
$sub = "mySubnet"
$pol = "Automatic"
$lbn = "myLoadBalancer"

$lb = Get-AzLoadBalancer -ResourceGroupName $rsg -Name $lbn

New-AzVmss -ResourceGroupName $rsg -Location $loc -VMScaleSetName $vms -VirtualNetworkName $vnt -SubnetName $sub -LoadBalancerName $lb -UpgradePolicyMode $pol
```
> [!NOTE]
> Después de crear el conjunto de escalado, no se puede modificar el puerto de back-end para una regla de equilibrio de carga que se usa en un sondeo de estado del equilibrador de carga. Para cambiar el puerto, puede quitar el sondeo de estado mediante la actualización del conjunto de escalado de máquinas virtuales de Azure, actualizar el puerto y, a continuación, volver a configurar el sondeo de estado.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, implementó un conjunto de escalado de máquinas virtuales con una instancia existente de Azure Load Balancer.  Para obtener más información sobre los conjuntos de escalado de máquinas virtuales y el equilibrador de carga, consulte:

- [¿Qué es Azure Load Balancer?](load-balancer-overview.md)
- [¿Qué son los conjuntos de escalado de máquinas virtuales?](../virtual-machine-scale-sets/overview.md)
