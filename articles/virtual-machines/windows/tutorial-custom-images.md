---
title: 'Tutorial: Creación de imágenes de máquina virtual personalizadas con Azure PowerShell'
description: En este tutorial, aprenderá a usar Azure PowerShell para crear una imagen de máquina virtual personalizada de Windows almacenada en una instancia de Azure Shared Image Gallery.
author: cynthn
ms.service: virtual-machines-windows
ms.subservice: imaging
ms.topic: tutorial
ms.workload: infrastructure
ms.date: 05/01/2020
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: cf8fc9916384c9eef24c4c50f7647632c0e6b7a2
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87077468"
---
# <a name="tutorial-create-windows-vm-images-with-azure-powershell"></a>Tutorial: Creación de imágenes de máquina virtual Windows con Azure PowerShell

Se pueden usar imágenes personalizadas en el arranque de las implementaciones y para garantizar la coherencia entre varias máquinas virtuales. En este tutorial, creará su propia imagen especializada de una máquina virtual de Azure mediante PowerShell y la almacenará en una instancia de Shared Image Gallery. Aprenderá a:

> [!div class="checklist"]
> * Creación de una instancia de Shared Image Gallery
> * Creación de la definición de una imagen
> * Creación de la versión de una imagen
> * Crear una máquina virtual a partir de una imagen 
> * Compartir una galería de imágenes



## <a name="before-you-begin"></a>Antes de empezar

En los pasos siguientes se explica cómo tomar una máquina virtual existente y convertirla en una imagen personalizada reutilizable que puede usar para crear otras máquinas virtuales.

Para completar el ejemplo de este tutorial, debe tener una máquina virtual. Si es necesario, puede consultar el [inicio rápido de PowerShell](quick-create-powershell.md) para crear una máquina virtual y usarla en este tutorial. Al trabajar en el tutorial, reemplace los nombres de recursos cuando sea necesario.

## <a name="overview"></a>Información general

Una [galería de imágenes compartidas](shared-image-galleries.md) simplifica el uso compartido de imágenes personalizadas en toda una organización. Las imágenes personalizadas son como las imágenes de Marketplace, pero las puede crear usted mismo. Las imágenes personalizadas pueden usarse para configuraciones de arranque como la carga previa de aplicaciones, configuraciones de aplicaciones y otras configuraciones del sistema operativo. 

Shared Image Gallery le permite compartir sus imágenes de máquina virtual personalizadas con otras personas. Elija las imágenes que desea compartir, qué regiones desea que estén disponibles en ellas y con quién desea compartirlas. 

La característica de galería de imágenes compartidas tiene varios tipos de recursos:

[!INCLUDE [virtual-machines-shared-image-gallery-resources](../../../includes/virtual-machines-shared-image-gallery-resources.md)]


## <a name="launch-azure-cloud-shell"></a>Inicio de Azure Cloud Shell

Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. 

Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior derecha de un bloque de código. También puede ir a [https://shell.azure.com/powershell](https://shell.azure.com/powershell) para iniciar Cloud Shell en una pestaña independiente del explorador. Seleccione **Copiar** para copiar los bloques de código, péguelos en Cloud Shell y, luego, presione Entrar para ejecutarlos.

## <a name="get-the-vm"></a>Obtención de la máquina virtual

Puede ver una lista de máquinas virtuales disponibles en un grupo de recursos con [Get-AzVM](/powershell/module/az.compute/get-azvm). Una vez que sepa el nombre de la máquina virtual y cuál es el grupo de recursos, puede usar `Get-AzVM` nuevamente para obtener el objeto de máquina virtual y almacenarlo en una variable para su uso posterior. En este ejemplo se obtiene una máquina virtual denominada *sourceVM* del grupo de recursos "myResourceGroup" y se asigna a la variable *$sourceVM*. 

```azurepowershell-interactive
$sourceVM = Get-AzVM `
   -Name sourceVM `
   -ResourceGroupName myResourceGroup
```

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Cree un grupo de recursos con el comando [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup).

Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. En el siguiente ejemplo, se crea un grupo de recursos denominado *myGalleryRG* en la región *EastUS*:

```azurepowershell-interactive
$resourceGroup = New-AzResourceGroup `
   -Name 'myGalleryRG' `
   -Location 'EastUS'
```

## <a name="create-an-image-gallery"></a>Creación de una galería de imágenes 

Una galería de imágenes es el recurso principal que se usa para habilitar el uso compartido de imágenes. Los caracteres permitidos para el nombre de la galería son letras mayúsculas o minúsculas, números y puntos. El nombre de la galería no puede contener guiones. Los nombres de las galerías deben ser únicos dentro de su suscripción. 

Cree una galería de imágenes con [New-AzGallery](/powershell/module/az.compute/new-azgallery). En el ejemplo siguiente se crea una galería denominada *myGallery* en el grupo de recursos *myGalleryRG*.

```azurepowershell-interactive
$gallery = New-AzGallery `
   -GalleryName 'myGallery' `
   -ResourceGroupName $resourceGroup.ResourceGroupName `
   -Location $resourceGroup.Location `
   -Description 'Shared Image Gallery for my organization'  
```


## <a name="create-an-image-definition"></a>Creación de la definición de una imagen 

Las definiciones de imagen crean una agrupación lógica de imágenes. Estas se usan para administrar la información sobre las versiones de la imagen que se crean dentro de ellas. Los nombres de las definiciones de imagen pueden estar formados por letras mayúsculas o minúsculas, números, puntos y guiones. Para más información sobre los valores que se pueden especificar para una definición de imagen, consulte [Definiciones de imagen](./shared-image-galleries.md#image-definitions).

Cree la definición de imagen mediante [New-AzGalleryImageDefinition](/powershell/module/az.compute/new-azgalleryimageversion). En este ejemplo, la imagen de la galería se denomina *myGalleryImage* y se crea para una imagen especializada. 

```azurepowershell-interactive
$galleryImage = New-AzGalleryImageDefinition `
   -GalleryName $gallery.Name `
   -ResourceGroupName $resourceGroup.ResourceGroupName `
   -Location $gallery.Location `
   -Name 'myImageDefinition' `
   -OsState specialized `
   -OsType Windows `
   -Publisher 'myPublisher' `
   -Offer 'myOffer' `
   -Sku 'mySKU'
```


## <a name="create-an-image-version"></a>Creación de la versión de una imagen

Cree una versión de la imagen a partir de una máquina virtual mediante [New-AzGalleryImageVersion](/powershell/module/az.compute/new-azgalleryimageversion). 

Los caracteres permitidos para la versión de una imagen son números y puntos. Los números deben estar dentro del rango de un entero de 32 bits. Formato: *VersiónPrincipal*.*VersiónSecundaria*.*Revisión*.

En este ejemplo, la versión de la imagen es *1.0.0* y se replica en los centros de datos *Centro-este de EE. UU.* y *Centro-sur de EE. UU.* Al elegir las regiones de destino de la replicación, debe incluir la región de *origen* como destino de la replicación.

Para crear una versión de la imagen a partir de la máquina virtual, use `$vm.Id.ToString()` como `-Source`.

```azurepowershell-interactive
   $region1 = @{Name='South Central US';ReplicaCount=1}
   $region2 = @{Name='East US';ReplicaCount=2}
   $targetRegions = @($region1,$region2)

New-AzGalleryImageVersion `
   -GalleryImageDefinitionName $galleryImage.Name`
   -GalleryImageVersionName '1.0.0' `
   -GalleryName $gallery.Name `
   -ResourceGroupName $resourceGroup.ResourceGroupName `
   -Location $resourceGroup.Location `
   -TargetRegion $targetRegions  `
   -Source $sourceVM.Id.ToString() `
   -PublishingProfileEndOfLifeDate '2020-12-01'
```

La imagen puede tardar un rato en replicarse en todas las regiones de destino.


## <a name="create-a-vm"></a>Crear una VM 

Cuando tenga una imagen especializada, puede crear una o varias máquinas virtuales. Mediante el uso del cmdlet [New-AzVM](/powershell/module/az.compute/new-azvm). Para usar la imagen, emplee `Set-AzVMSourceImage` y establezca el `-Id` en el identificador de la definición de imagen ($galleryImage.Id en este caso) para utilizar siempre la versión más reciente de la imagen. 

Reemplace los nombres de los recursos según sea necesario en este ejemplo. 

```azurepowershell-interactive
# Create some variables for the new VM.
$resourceGroup = "myResourceGroup"
$location = "South Central US"
$vmName = "mySpecializedVM"

# Create a resource group
New-AzResourceGroup -Name $resourceGroup -Location $location

# Create the network resources.
$subnetConfig = New-AzVirtualNetworkSubnetConfig -Name mySubnet -AddressPrefix 192.168.1.0/24
$vnet = New-AzVirtualNetwork -ResourceGroupName $resourceGroup -Location $location `
  -Name MYvNET -AddressPrefix 192.168.0.0/16 -Subnet $subnetConfig
$pip = New-AzPublicIpAddress -ResourceGroupName $resourceGroup -Location $location `
  -Name "mypublicdns$(Get-Random)" -AllocationMethod Static -IdleTimeoutInMinutes 4
$nsgRuleRDP = New-AzNetworkSecurityRuleConfig -Name myNetworkSecurityGroupRuleRDP  -Protocol Tcp `
  -Direction Inbound -Priority 1000 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
  -DestinationPortRange 3389 -Access Allow
$nsg = New-AzNetworkSecurityGroup -ResourceGroupName $resourceGroup -Location $location `
  -Name myNetworkSecurityGroup -SecurityRules $nsgRuleRDP
$nic = New-AzNetworkInterface -Name $vmName -ResourceGroupName $resourceGroup -Location $location `
  -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id -NetworkSecurityGroupId $nsg.Id

# Create a virtual machine configuration using $imageVersion.Id to specify the image version.
$vmConfig = New-AzVMConfig -VMName $vmName -VMSize Standard_D1_v2 | `
Set-AzVMSourceImage -Id $galleryImage.Id | `
Add-AzVMNetworkInterface -Id $nic.Id

# Create a virtual machine
New-AzVM -ResourceGroupName $resourceGroup -Location $location -VM $vmConfig
```


## <a name="share-the-gallery"></a>Uso compartido de la galería

Se recomienda compartir el acceso en el nivel de la galería de imágenes. Use una dirección de correo electrónico y el cmdlet [Get-AzADUser](/powershell/module/az.resources/get-azaduser) para obtener el identificador de objeto del usuario y, después, utilice [New-AzRoleAssignment](/powershell/module/Az.Resources/New-AzRoleAssignment) para concederle acceso a la galería. Reemplace el correo electrónico alinne_montes@contoso.com de este ejemplo por su propia información.

```azurepowershell-interactive
# Get the object ID for the user
$user = Get-AzADUser -StartsWith alinne_montes@contoso.com
# Grant access to the user for our gallery
New-AzRoleAssignment `
   -ObjectId $user.Id `
   -RoleDefinitionName Reader `
   -ResourceName $gallery.Name `
   -ResourceType Microsoft.Compute/galleries `
   -ResourceGroupName $resourceGroup.ResourceGroupName
```
   
## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no se necesiten, puede usar el cmdlet [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) para quitar el grupo de recursos y todos los recursos relacionados:

```azurepowershell-interactive
# Delete the gallery 
Remove-AzResourceGroup -Name myGalleryRG

# Delete the VM
Remove-AzResourceGroup -Name myResoureceGroup
```

## <a name="azure-image-builder"></a>Azure Image Builder

Azure también ofrece un servicio, basado en Packer, [Azure VM Image Builder](./image-builder-overview.md). Solo tiene que describir sus personalizaciones en una plantilla y esta servirá para crear la imagen. 

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado una imagen de máquina virtual especializada. Ha aprendido a:

> [!div class="checklist"]
> * Creación de una instancia de Shared Image Gallery
> * Creación de la definición de una imagen
> * Creación de la versión de una imagen
> * Crear una máquina virtual a partir de una imagen 
> * Compartir una galería de imágenes

En al siguiente tutorial aprenderá a crear máquinas virtuales de alta disponibilidad.

> [!div class="nextstepaction"]
> [Creación de máquinas virtuales de alta disponibilidad](tutorial-availability-sets.md)
