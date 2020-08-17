---
title: Versión preliminar de grupos de selección de ubicación de proximidad para conjuntos de escalado de máquinas virtuales
description: Obtenga información sobre la creación y el uso de grupos de selección de ubicación de proximidad para conjuntos de escalado de máquinas virtuales Windows en Azure.
author: cynthn
ms.author: cynthn
ms.topic: how-to
ms.service: virtual-machine-scale-sets
ms.subservice: availability
ms.date: 07/01/2019
ms.reviewer: zivr
ms.custom: mimckitt
ms.openlocfilehash: 1525ef7c1e19aa9821df3f111fdce3518d37afd5
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87011063"
---
# <a name="preview-creating-and-using-proximity-placement-groups-using-powershell"></a>Vista previa: Creación y uso de grupos de selección de ubicación de proximidad con PowerShell

Para conseguir máquinas virtuales lo más cercanas posible con la menor latencia posible, debe implementar el conjunto de escalado dentro de un [grupo de selección de ubicación de proximidad](co-location.md#preview-proximity-placement-groups).

Un grupo de selección de ubicación de proximidad es una agrupación lógica que se usa para asegurarse de que los recursos de proceso de Azure se encuentran físicamente cercanos entre sí. Los grupos de selección de ubicación de proximidad son útiles para las cargas de trabajo en las que la latencia baja es un requisito.

> [!IMPORTANT]
> Los grupos de selección de ubicación de proximidad se encuentran actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).
>
> Los grupos de selección de ubicación de proximidad no están disponibles en estas regiones durante la versión preliminar: **Japón Oriental**, **Este de Australia** e **India central**.


## <a name="create-a-proximity-placement-group"></a>Creación de un grupo de selección de ubicación por proximidad
Cree un grupo de selección de ubicación de proximidad con el cmdlet [New-AzProximityPlacementGroup](/powershell/module/az.compute/new-azproximityplacementgroup). 

```azurepowershell-interactive
$resourceGroup = "myPPGResourceGroup"
$location = "East US"
$ppgName = "myPPG"
New-AzResourceGroup -Name $resourceGroup -Location $location
$ppg = New-AzProximityPlacementGroup `
   -Location $location `
   -Name $ppgName `
   -ResourceGroupName $resourceGroup `
   -ProximityPlacementGroupType Standard
```

## <a name="list-proximity-placement-groups"></a>Enumeración de los grupos de selección de ubicación de proximidad

Puede enumerar todos los grupos de selección de ubicación de proximidad mediante el cmdlet [Get-AzProximityPlacementGroup](/powershell/module/az.compute/get-azproximityplacementgroup).

```azurepowershell-interactive
Get-AzProximityPlacementGroup
```


## <a name="create-a-scale-set"></a>Creación de un conjunto de escalado

Cree una escala en el grupo de selección de ubicación de proximidad con `-ProximityPlacementGroup $ppg.Id` para hacer referencia al identificador de grupo de selección de ubicación de proximidad cuando use [New-AzVMSS](/powershell/module/az.compute/new-azvmss) para crear el conjunto de escalado.

```azurepowershell-interactive
$scalesetName = "myVM"

New-AzVmss `
  -ResourceGroupName $resourceGroup `
  -Location $location `
  -VMScaleSetName $scalesetName `
  -VirtualNetworkName "myVnet" `
  -SubnetName "mySubnet" `
  -PublicIpAddressName "myPublicIPAddress" `
  -LoadBalancerName "myLoadBalancer" `
  -UpgradePolicyMode "Automatic" `
  -ProximityPlacementGroup $ppg.Id
```

Puede ver la instancia en el grupo de selección de ubicación mediante [Get-AzProximityPlacementGroup](/powershell/module/az.compute/get-azproximityplacementgroup).

```azurepowershell-interactive
  Get-AzProximityPlacementGroup `
   -ResourceId $ppg.Id | Format-Table `
   -Wrap `
   -Property VirtualMachineScaleSets
```

## <a name="next-steps"></a>Pasos siguientes

También puede usar la [CLI de Azure](../virtual-machines/linux/proximity-placement-groups.md) para crear grupos de selección de ubicación de proximidad.
