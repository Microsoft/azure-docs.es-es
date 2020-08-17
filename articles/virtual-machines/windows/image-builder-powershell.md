---
title: Creación de una máquina virtual Windows con Azure Image Builder mediante PowerShell
description: Cree una máquina virtual Windows con el módulo de PowerShell de Azure Image Builder.
author: cynthn
ms.author: cynthn
ms.date: 06/17/2020
ms.topic: how-to
ms.service: virtual-machines-windows
ms.subservice: imaging
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: e25b2b53acdfb05af8572a01109961bf3002e429
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87499439"
---
# <a name="preview-create-a-windows-vm-with-azure-image-builder-using-powershell"></a>Vista previa: Creación de una máquina virtual Windows con Azure Image Builder mediante PowerShell

En este artículo se muestra cómo puede crear una imagen personalizada de Windows mediante el módulo de PowerShell de Azure VM Image Builder.

> [!CAUTION]
> Actualmente, el generador de imágenes de Azure se encuentra en versión preliminar pública. Esta versión preliminar se proporciona sin un acuerdo de nivel de servicio. No se recomienda para las cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

Si decide usar PowerShell de forma local, para este artículo es preciso que instale el módulo Az PowerShell y que se conecte a su cuenta de Azure con el cmdlet [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount). Para más información sobre cómo instalar el módulo Az PowerShell, consulte [Instalación de Azure PowerShell](/powershell/azure/install-az-ps).

> [!IMPORTANT]
> Mientras los módulos de PowerShell **Az.ImageBuilder** y **Az.ManagedServiceIdentity** estén en versión preliminar, debe instalarlos de forma independiente mediante el cmdlet `Install-Module` con el parámetro `AllowPrerelease`. Una vez que estos módulos de PowerShell estén disponibles con carácter general, formarán parte de las futuras versiones del módulo Az PowerShell y estarán disponibles de forma nativa en Azure Cloud Shell.

```azurepowershell-interactive
'Az.ImageBuilder', 'Az.ManagedServiceIdentity' | ForEach-Object {Install-Module -Name $_ -AllowPrerelease}
```

[!INCLUDE [cloud-shell-try-it](../../../includes/cloud-shell-try-it.md)]

Si tiene varias suscripciones a Azure, elija la suscripción adecuada en la que se debe facturar el recurso. Seleccione una suscripción específica con el cmdlet [Set-AzContext](/powershell/module/az.accounts/set-azcontext).

```azurepowershell-interactive
Set-AzContext -SubscriptionId 00000000-0000-0000-0000-000000000000
```

### <a name="register-features"></a>Registro de características

Si esta es la primera vez que usa Azure Image Builder durante la versión preliminar, registre la nueva característica **VirtualMachineTemplatePreview**.

```azurepowershell-interactive
Register-AzProviderFeature -ProviderNamespace Microsoft.VirtualMachineImages -FeatureName VirtualMachineTemplatePreview
```

Compruebe el estado del registro de la característica.

> [!NOTE]
> **RegistrationState** puede estar en el estado `Registering` durante varios minutos antes de cambiar a `Registered`. Espere hasta que el estado sea **Registrado** antes de continuar.

```azurepowershell-interactive
Get-AzProviderFeature -ProviderNamespace Microsoft.VirtualMachineImages -FeatureName VirtualMachineTemplatePreview
```

Registre los siguientes proveedores de recursos para usarlos con la suscripción de Azure si aún no están registrados.

- Microsoft.Compute
- Microsoft.KeyVault
- Microsoft.Storage
- Microsoft.VirtualMachineImages

```azurepowershell-interactive
Get-AzResourceProvider -ProviderNamespace Microsoft.Compute, Microsoft.KeyVault, Microsoft.Storage, Microsoft.VirtualMachineImages |
  Where-Object RegistrationState -ne Registered |
    Register-AzResourceProvider
```

## <a name="define-variables"></a>Definición de variables

Usará varias partes de la información de forma repetida. Cree variables para almacenar la información.

```azurepowershell-interactive
# Destination image resource group name
$imageResourceGroup = 'myWinImgBuilderRG'

# Azure region
$location = 'WestUS2'

# Name of the image to be created
$imageTemplateName = 'myWinImage'

# Distribution properties of the managed image upon completion
$runOutputName = 'myDistResults'
```

Cree una variable para el id. de suscripción de Azure. Para confirmar que la variable `subscriptionID` contiene el id. de la suscripción, puede ejecutar la segunda línea en el ejemplo siguiente.

```azurepowershell-interactive
# Your Azure Subscription ID
$subscriptionID = (Get-AzContext).Subscription.Id
Write-Output $subscriptionID
```

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Cree un [grupo de recursos de Azure](../../azure-resource-manager/management/overview.md) con el cmdlet [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Un grupo de recursos es un contenedor lógico en el que se implementan y se administran recursos de Azure como un grupo.

En el ejemplo siguiente se crea un grupo de recursos basado en el nombre de la variable `$imageResourceGroup` en la región especificada en la variable `$location`. Este grupo de recursos se usa para almacenar el artefacto de la plantilla de configuración de la imagen y la imagen misma.

```azurepowershell-interactive
New-AzResourceGroup -Name $imageResourceGroup -Location $location
```

## <a name="create-user-identity-and-set-role-permissions"></a>Creación de una identidad de usuario y configuración de los permisos de rol

Conceda permisos a Azure Image Builder para crear imágenes en el grupo de recursos especificado mediante el ejemplo siguiente. Sin este permiso, el proceso de compilación de la imagen no se completará correctamente.

Cree variables para la definición de roles y los nombres de identidad. Estos valores deben ser únicos.

```azurepowershell-interactive
[int]$timeInt = $(Get-Date -UFormat '%s')
$imageRoleDefName = "Azure Image Builder Image Def $timeInt"
$identityName = "myIdentity$timeInt"
```

Cree una identidad de usuario.

```azurepowershell-interactive
New-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $identityName
```

Almacene el recurso de identidad y los id. de entidad de seguridad en las variables.

```azurepowershell-interactive
$identityNameResourceId = (Get-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $identityName).Id
$identityNamePrincipalId = (Get-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $identityName).PrincipalId
```

### <a name="assign-permissions-for-identity-to-distribute-images"></a>Asignación de permisos para identidad para distribuir imágenes

Descargue el archivo de configuración JSON y modifíquelo en función de la configuración definida en este artículo.

```azurepowershell-interactive
$myRoleImageCreationUrl = 'https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/solutions/12_Creating_AIB_Security_Roles/aibRoleImageCreation.json'
$myRoleImageCreationPath = "$env:TEMP\myRoleImageCreation.json"

Invoke-WebRequest -Uri $myRoleImageCreationUrl -OutFile $myRoleImageCreationPath -UseBasicParsing

$Content = Get-Content -Path $myRoleImageCreationPath -Raw
$Content = $Content -replace '<subscriptionID>', $subscriptionID
$Content = $Content -replace '<rgName>', $imageResourceGroup
$Content = $Content -replace 'Azure Image Builder Service Image Creation Role', $imageRoleDefName
$Content | Out-File -FilePath $myRoleImageCreationPath -Force
```

Cree la definición de roles.

```azurepowershell-interactive
New-AzRoleDefinition -InputFile $myRoleImageCreationPath
```

Conceda la definición de roles a la entidad de servicio de Image Builder.

```azurepowershell-interactive
$RoleAssignParams = @{
  ObjectId = $identityNamePrincipalId
  RoleDefinitionName = $imageRoleDefName
  Scope = "/subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup"
}
New-AzRoleAssignment @RoleAssignParams
```

> [!NOTE]
> Si recibe el error: "_New-AzRoleDefinition: Se ha superado el límite de definiciones de rol. No se pueden crear más definiciones de roles._ ", consulte [Solución de problemas de Azure RBAC](../../role-based-access-control/troubleshooting.md).

## <a name="create-a-shared-image-gallery"></a>Creación de una instancia de Shared Image Gallery

Cree la galería.

```azurepowershell-interactive
$myGalleryName = 'myImageGallery'
$imageDefName = 'winSvrImages'

New-AzGallery -GalleryName $myGalleryName -ResourceGroupName $imageResourceGroup -Location $location
```

Cree una definición de galería.

```azurepowershell-interactive
$GalleryParams = @{
  GalleryName = $myGalleryName
  ResourceGroupName = $imageResourceGroup
  Location = $location
  Name = $imageDefName
  OsState = 'generalized'
  OsType = 'Windows'
  Publisher = 'myCo'
  Offer = 'Windows'
  Sku = 'Win2019'
}
New-AzGalleryImageDefinition @GalleryParams
```

## <a name="create-an-image"></a>Crear una imagen

Cree un objeto de origen de Azure Image Builder. Consulte [Búsqueda de imágenes de VM de Windows en Azure Marketplace con Azure PowerShell](./cli-ps-findimage.md) para ver los valores de parámetros válidos.

```azurepowershell-interactive
$SrcObjParams = @{
  SourceTypePlatformImage = $true
  Publisher = 'MicrosoftWindowsServer'
  Offer = 'WindowsServer'
  Sku = '2019-Datacenter'
  Version = 'latest'
}
$srcPlatform = New-AzImageBuilderSourceObject @SrcObjParams
```

Cree un objeto distribuidor de Azure Image Builder.

```azurepowershell-interactive
$disObjParams = @{
  SharedImageDistributor = $true
  ArtifactTag = @{tag='dis-share'}
  GalleryImageId = "/subscriptions/$subscriptionID/resourceGroups/$imageResourceGroup/providers/Microsoft.Compute/galleries/$myGalleryName/images/$imageDefName"
  ReplicationRegion = $location
  RunOutputName = $runOutputName
  ExcludeFromLatest = $false
}
$disSharedImg = New-AzImageBuilderDistributorObject @disObjParams
```

Cree un objeto de personalización de Azure Image Builder.

```azurepowershell-interactive
$ImgCustomParams = @{
  PowerShellCustomizer = $true
  CustomizerName = 'settingUpMgmtAgtPath'
  RunElevated = $false
  Inline = @("mkdir c:\\buildActions", "echo Azure-Image-Builder-Was-Here  > c:\\buildActions\\buildActionsOutput.txt")
}
$Customizer = New-AzImageBuilderCustomizerObject @ImgCustomParams
```

Cree una plantilla de Azure Image Builder.

```azurepowershell-interactive
$ImgTemplateParams = @{
  ImageTemplateName = $imageTemplateName
  ResourceGroupName = $imageResourceGroup
  Source = $srcPlatform
  Distribute = $disSharedImg
  Customize = $Customizer
  Location = $location
  UserAssignedIdentityId = $identityNameResourceId
}
New-AzImageBuilderTemplate @ImgTemplateParams
```

Al completarse, se devuelve un mensaje y se crea una plantilla de configuración de Image Builder en `$imageResourceGroup`.

Para determinar si el proceso de creación de la plantilla se ha realizado correctamente, puede usar el ejemplo siguiente.

```azurepowershell-interactive
Get-AzImageBuilderTemplate -ImageTemplateName $imageTemplateName -ResourceGroupName $imageResourceGroup |
  Select-Object -Property Name, LastRunStatusRunState, LastRunStatusMessage, ProvisioningState
```

Asimismo, Image Builder crea en segundo plano un grupo de recursos de almacenamiento provisional en la suscripción. Este grupo de recursos se usa para la compilación de la imagen. Está en el formato `IT_<DestinationResourceGroup>_<TemplateName>`.

> [!WARNING]
> No elimine el grupo de recursos de almacenamiento provisional directamente. Elimine el artefacto de la plantilla de imagen. Esto hará que se elimine el grupo de recursos de almacenamiento provisional.

Si el servicio informa de un error durante el envío de la plantilla de configuración de la imagen:

- Consulte [Solución de errores de Azure VM Image Builder (AIB)](https://github.com/danielsollondon/azvmimagebuilder/blob/master/troubleshootingaib.md#template-submission-errors--troubleshooting).
- Elimine la plantilla mediante el ejemplo siguiente antes de volver a intentarlo.

```azurepowershell-interactive
Remove-AzImageBuilderTemplate -ImageTemplateName $imageTemplateName -ResourceGroupName $imageResourceGroup
```

## <a name="start-the-image-build"></a>Iniciar la generación de imágenes

Envíe la configuración de la imagen al servicio de VM Image Builder.

```azurepowershell-interactive
Start-AzImageBuilderTemplate -ResourceGroupName $imageResourceGroup -Name $imageTemplateName
```

Espere a que se complete el proceso de aprovisionamiento. Este paso puede tardar hasta una hora.

Si se producen errores, consulte [Solución de errores de Azure VM Image Builder (AIB)](https://github.com/danielsollondon/azvmimagebuilder/blob/master/troubleshootingaib.md#image-build-errors--troubleshooting).

## <a name="create-a-vm"></a>Crear una VM

Almacene las credenciales de inicio de sesión de la máquina virtual en una variable. La contraseña debe ser compleja.

```azurepowershell-interactive
$Cred = Get-Credential
```

Cree la máquina virtual con la imagen que ha creado.

```azurepowershell-interactive
$ArtifactId = (Get-AzImageBuilderRunOutput -ImageTemplateName $imageTemplateName -ResourceGroupName $imageResourceGroup).ArtifactId

New-AzVM -ResourceGroupName $imageResourceGroup -Image $ArtifactId -Name myWinVM01 -Credential $Cred
```

## <a name="verify-the-customization"></a>Comprobación de la personalización

Cree una conexión de Escritorio remoto a la máquina virtual con el nombre de usuario y la contraseña que ha establecido al crear la máquina virtual. Dentro de la máquina virtual, abra PowerShell y ejecute `Get-Content` como se muestra en el ejemplo siguiente:

```azurepowershell-interactive
Get-Content -Path C:\buildActions\buildActionsOutput.txt
```

Debería ver la salida en función del contenido del archivo creado durante el proceso de personalización de la imagen.

```Output
Azure-Image-Builder-Was-Here
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no se necesitan los recursos que se han creado en este artículo, puede eliminarlos con el siguiente comando.

### <a name="delete-the-image-builder-template"></a>Eliminar la plantilla de Image Builder

```azurepowershell-interactive
Remove-AzImageBuilderTemplate -ResourceGroupName $imageResourceGroup -Name $imageTemplateName
```

### <a name="delete-the-image-resource-group"></a>Eliminar el grupo de recursos de imagen

> [!CAUTION]
> En el ejemplo siguiente se elimina el grupo de recursos especificado y todos los recursos que contiene.
> Si los recursos que están fuera del ámbito de este artículo existen en el grupo de recursos especificado, también se eliminarán.

```azurepowershell-interactive
Remove-AzResourceGroup -Name $imageResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre los componentes del archivo .json que se usan en este artículo, vea [Referencia de la plantilla de generador de imágenes](../linux/image-builder-json.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
