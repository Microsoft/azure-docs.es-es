---
title: Creación de un conjunto de escalado a partir de una imagen generalizada con Azure PowerShell
description: Cree un conjunto de escalado mediante una imagen generalizada en Shared Image Gallery.
author: cynthn
ms.service: virtual-machine-scale-sets
ms.subservice: imaging
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 05/04/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.openlocfilehash: 3b8c9d46a5bc5dde3e39bfa5673247ca752d2a89
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87077566"
---
# <a name="create-a-scale-set-from-a-generalized-image"></a>Creación de un conjunto de escalado a partir de una imagen generalizada

Cree una máquina virtual a partir de una versión de imagen generalizada almacenada en [Shared Image Gallery](shared-image-galleries.md). Si desea crear un conjunto de escalado mediante una imagen especializada, consulte el artículo sobre la [creación de instancias de un conjunto de escalado a partir de una imagen especializada](instance-specialized-image-version-powershell.md).

Una vez que tenga una imagen generalizada, puede crear un conjunto de escalado de máquinas virtuales mediante el cmdlet [New-AzVmss](/powershell/module/az.compute/new-azvmss). 

En este ejemplo, se usa el identificador de definición de imagen para asegurarse de que la nueva máquina virtual usará la versión más reciente de una imagen. También puede usar una versión específica con el identificador de la versión de imagen de `-ImageReferenceId`. Por ejemplo, para usar la versión de imagen *1.0.0*, escriba: `-ImageReferenceId "/subscriptions/<subscription ID where the gallery is located>/resourceGroups/myGalleryRG/providers/Microsoft.Compute/galleries/myGallery/images/myImageDefinition/versions/1.0.0"`. 

Tenga en cuenta que el uso de una versión de imagen específica significa que la automatización podría producir un error si dicha versión específica no está disponible porque se eliminó o se quitó de la región. Se recomienda usar el identificador de definición de la imagen para crear la nueva máquina virtual, a menos que se requiera una versión de imagen específica.


En los ejemplos siguientes se crea un conjunto de escalado denominado *myScaleSet*, en el grupo de recursos *myVMSSRG*, en la ubicación *SouthCentralUS*. El conjunto de escalado se creará a partir de la imagen *myImageDefinition*, en la galería de imágenes *myGallery* del grupo de recursos *myGalleryRG*. Cuando se le solicite, establezca sus propias credenciales administrativas para las instancias de máquina virtual del conjunto de escalado.


## <a name="simplified-parameter-set"></a>Conjunto de parámetros simplificado

Para crear rápidamente un conjunto de escalado y proporcionar la información mínima, use el conjunto de parámetros simplificado para crear un conjunto de escalado a partir de una imagen de Shared Image Gallery.

```azurepowershell-interactive
$imageDefinition = Get-AzGalleryImageDefinition `
   -GalleryName myGallery `
   -ResourceGroupName myGalleryRG `
   -Name myImageDefinition

# Create user object

$cred = Get-Credential `
   -Message "Enter a username and password for the virtual machine."

# Create the resource group and scale set
New-AzResourceGroup -ResourceGroupName myVMSSRG -Location SouthCentralUS
New-AzVmss `
   -Credential $cred `
   -VMScaleSetName myScaleSet `
   -ImageName $imageDefinition.Id `
   -UpgradePolicyMode Automatic `
   -ResourceGroupName myVMSSRG
```

Se tardan unos minutos en crear y configurar todos los recursos de conjunto de escalado y máquinas virtuales.

## <a name="extended-parameter-set"></a>Conjunto de parámetros extendido

Para tener un control total sobre todos los recursos, entre los que se incluye la nomenclatura, use el conjunto de parámetros completo para crear un conjunto de escalado mediante una imagen de Shared Image Gallery. 

```azurepowershell-interactive
# Get the image definition

$imageDefinition = Get-AzGalleryImageDefinition `
   -GalleryName myGallery `
   -ResourceGroupName myGalleryRG `
   -Name myImageDefinition

# Create user object

$cred = Get-Credential `
   -Message "Enter a username and password for the virtual machine."
   
# Define variables for the scale set
$resourceGroupName = "myVMSSRG"
$scaleSetName = "myScaleSet"
$location = "South Central US"

# Create a resource group
New-AzResourceGroup -ResourceGroupName $resourceGroupName -Location $location

# Create a networking pieces
$subnet = New-AzVirtualNetworkSubnetConfig `
  -Name "mySubnet" `
  -AddressPrefix 10.0.0.0/24
$vnet = New-AzVirtualNetwork `
  -ResourceGroupName $resourceGroupName `
  -Name "myVnet" `
  -Location $location `
  -AddressPrefix 10.0.0.0/16 `
  -Subnet $subnet
$publicIP = New-AzPublicIpAddress `
  -ResourceGroupName $resourceGroupName `
  -Location $location `
  -AllocationMethod Static `
  -Name "myPublicIP"
$frontendIP = New-AzLoadBalancerFrontendIpConfig `
  -Name "myFrontEndPool" `
  -PublicIpAddress $publicIP
$backendPool = New-AzLoadBalancerBackendAddressPoolConfig -Name "myBackEndPool"
$inboundNATPool = New-AzLoadBalancerInboundNatPoolConfig `
  -Name "myRDPRule" `
  -FrontendIpConfigurationId $frontendIP.Id `
  -Protocol TCP `
  -FrontendPortRangeStart 50001 `
  -FrontendPortRangeEnd 50010 `
  -BackendPort 3389
# Create the load balancer and health probe
$lb = New-AzLoadBalancer `
  -ResourceGroupName $resourceGroupName `
  -Name "myLoadBalancer" `
  -Location $location `
  -FrontendIpConfiguration $frontendIP `
  -BackendAddressPool $backendPool `
  -InboundNatPool $inboundNATPool
Add-AzLoadBalancerProbeConfig -Name "myHealthProbe" `
  -LoadBalancer $lb `
  -Protocol TCP `
  -Port 80 `
  -IntervalInSeconds 15 `
  -ProbeCount 2
Add-AzLoadBalancerRuleConfig `
  -Name "myLoadBalancerRule" `
  -LoadBalancer $lb `
  -FrontendIpConfiguration $lb.FrontendIpConfigurations[0] `
  -BackendAddressPool $lb.BackendAddressPools[0] `
  -Protocol TCP `
  -FrontendPort 80 `
  -BackendPort 80 `
  -Probe (Get-AzLoadBalancerProbeConfig -Name "myHealthProbe" -LoadBalancer $lb)
Set-AzLoadBalancer -LoadBalancer $lb

# Create IP address configurations
$ipConfig = New-AzVmssIpConfig `
  -Name "myIPConfig" `
  -LoadBalancerBackendAddressPoolsId $lb.BackendAddressPools[0].Id `
  -LoadBalancerInboundNatPoolsId $inboundNATPool.Id `
  -SubnetId $vnet.Subnets[0].Id

# Create a configuration 
$vmssConfig = New-AzVmssConfig `
    -Location $location `
    -SkuCapacity 2 `
    -SkuName "Standard_DS2" `
    -UpgradePolicyMode "Automatic"

# Reference the image version
Set-AzVmssStorageProfile $vmssConfig `
  -OsDiskCreateOption "FromImage" `
  -ImageReferenceId $imageDefinition.Id

# Complete the configuration
Set-AzVmssOsProfile $vmssConfig `
  -AdminUsername $cred.UserName `
  -AdminPassword $cred.Password `
  -ComputerNamePrefix "myVM"
Add-AzVmssNetworkInterfaceConfiguration `
  -VirtualMachineScaleSet $vmssConfig `
  -Name "network-config" `
  -Primary $true `
  -IPConfiguration $ipConfig

# Create the scale set 
New-AzVmss `
  -ResourceGroupName $resourceGroupName `
  -Name $scaleSetName `
  -VirtualMachineScaleSet $vmssConfig
```

Se tardan unos minutos en crear y configurar todos los recursos de conjunto de escalado y máquinas virtuales.

## <a name="next-steps"></a>Pasos siguientes
[Azure Image Builder (versión preliminar)](../virtual-machines/linux/image-builder-overview.md) puede ayudar a automatizar la creación de versiones de la imagen, incluso se puede usar para actualizar y [crear una nueva versión de la imagen a partir de una versión de imagen existente](../virtual-machines/linux/image-builder-gallery-update-image-version.md). 

Puede crear también recursos de galería de imágenes compartidas con plantillas. Hay varias plantillas de Inicio rápido de Azure disponibles: 

- [Creación de una galería de imágenes compartidas](https://azure.microsoft.com/resources/templates/101-sig-create/)
- [Creación de una definición de imagen en una galería de imágenes compartidas](https://azure.microsoft.com/resources/templates/101-sig-image-definition-create/)
- [Creación de una versión de imagen en una galería de imágenes compartidas](https://azure.microsoft.com/resources/templates/101-sig-image-version-create/)

Para más información sobre las galerías de imágenes compartidas, consulte la [Introducción](shared-image-galleries.md). Si encuentra problemas, consulte [Solución de problemas de galerías de imágenes compartidas](troubleshooting-shared-images.md).