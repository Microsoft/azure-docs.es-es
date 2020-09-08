---
title: Configuración de permisos del servicio Azure Image Builder mediante PowerShell
description: Configure los requisitos para el servicio Azure VM Image Builder, incluidos los permisos y privilegios mediante PowerShell.
author: danielsollondon
ms.author: danis
ms.date: 05/06/2020
ms.topic: article
ms.service: virtual-machines
ms.subservice: imaging
ms.openlocfilehash: cfe3efc77e065ac3685b72d0eab501034609b59b
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88068041"
---
# <a name="configure-azure-image-builder-service-permissions-using-powershell"></a>Configuración de permisos del servicio Azure Image Builder mediante PowerShell

El servicio Azure Image Builder requiere la configuración de permisos y privilegios antes de generar una imagen. En las secciones siguientes se detalla cómo configurar posibles escenarios mediante PowerShell.

> [!IMPORTANT]
> Actualmente, el generador de imágenes de Azure se encuentra en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="register-the-features"></a>Registro de las características

En primer lugar, debe registrarse para el servicio Azure Image Builder. El registro concede al servicio permiso para crear, administrar y eliminar un grupo de recursos de almacenamiento provisional. El servicio también tiene derechos para agregar recursos al grupo que se requieren para la creación de la imagen.

```powershell-interactive
Register-AzProviderFeature -FeatureName VirtualMachineTemplatePreview -ProviderNamespace Microsoft.VirtualMachineImages
```

## <a name="create-an-azure-user-assigned-managed-identity"></a>Creación de una identidad administrada de Azure asignada por el usuario

Azure Image Builder requiere la creación de una [identidad administrada de Azure asignada por el usuario](../../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-cli.md). Azure Image Builder usa la identidad administrada asignada por el usuario para leer imágenes, escribir imágenes y acceder a las cuentas de Azure Storage. El permiso de identidad se concede para realizar acciones específicas en una suscripción.

> [!NOTE]
> Anteriormente, Azure Image Builder usaba el nombre de entidad de seguridad de servicio (SPN) de Azure Image Builder para conceder permisos a los grupos de recursos de imagen. El uso del SPN caerá en desuso. Use una identidad administrada asignada por el usuario en su lugar.

En el ejemplo siguiente se muestra cómo crear una identidad administrada de Azure asignada por el usuario. Reemplace la configuración del marcador de posición para establecer las variables.

| Configuración | Descripción |
|---------|-------------|
| \<Resource group\> | Grupo de recursos donde se creará la identidad administrada asignada por el usuario. |

```powershell-interactive
## Add AZ PS module to support AzUserAssignedIdentity
Install-Module -Name Az.ManagedServiceIdentity

$parameters = @{
    Name = 'aibIdentity'
    ResourceGroupName = '<Resource group>'
}
# create identity
New-AzUserAssignedIdentity @parameters
```

Para obtener más información sobre las identidades de Azure asignadas por el usuario, consulte la documentación de la [Identidad administrada de Azure asignada por el usuario](../../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-cli.md) sobre cómo crear una identidad.

## <a name="allow-image-builder-to-distribute-images"></a>Permiso para que Image Builder distribuya imágenes

Para que Azure Image Builder distribuya imágenes (imágenes administradas/galería de imágenes compartidas), se debe permitir que el servicio Azure Image Builder inserte las imágenes en estos grupos de recursos. Para conceder los permisos necesarios, debe crear una identidad administrada asignada por el usuario y concederle derechos en el grupo de recursos donde se genera la imagen. Azure Image Builder **no** tiene permiso para acceder a los recursos de otros grupos de recursos de la suscripción. Debe realizar acciones explícitas para permitir el acceso para evitar errores en la creación.

No es necesario conceder derechos de colaborador de identidad administrada asignada por el usuario en el grupo de recursos para distribuir las imágenes. Sin embargo, la identidad administrada asignada por el usuario necesita los siguientes permisos `Actions` de Azure en el grupo de recursos de distribución:

```Actions
Microsoft.Compute/images/write
Microsoft.Compute/images/read
Microsoft.Compute/images/delete
```

Si se distribuye a una galería de imágenes compartidas, también necesitará:

```Actions
Microsoft.Compute/galleries/read
Microsoft.Compute/galleries/images/read
Microsoft.Compute/galleries/images/versions/read
Microsoft.Compute/galleries/images/versions/write
```

## <a name="permission-to-customize-existing-images"></a>Permiso para personalizar las imágenes existentes

Para que Azure Image Builder genere imágenes a partir de imágenes personalizadas de origen (imágenes administradas/galería de imágenes compartidas), se debe permitir que el servicio Azure Image Builder lea las imágenes en estos grupos de recursos. Para conceder los permisos necesarios, debe crear una identidad administrada asignada por el usuario y concederle derechos en el grupo de recursos donde se ubica la imagen.

Generación a partir de una imagen personalizada existente:

```Actions
Microsoft.Compute/galleries/read
```

Generación a partir de una versión existente de una galería de imágenes compartidas:

```Actions
Microsoft.Compute/galleries/read
Microsoft.Compute/galleries/images/read
Microsoft.Compute/galleries/images/versions/read
```

## <a name="permission-to-customize-images-on-your-vnets"></a>Permiso para personalizar imágenes en redes virtuales

Azure Image Builder tiene la capacidad de implementar y usar una red virtual existente en su suscripción, lo que permite que las personalizaciones tengan acceso a los recursos conectados.

No es necesario conceder derechos de colaborador de identidad administrada asignada por el usuario en el grupo de recursos para implementar una VM en una red virtual existente. Sin embargo, la identidad administrada asignada por el usuario necesita los siguientes permisos `Actions` de Azure en el grupo de recursos de red virtual:

```Actions
Microsoft.Network/virtualNetworks/read
Microsoft.Network/virtualNetworks/subnets/join/action
```

## <a name="create-an-azure-role-definition"></a>Creación de una definición de roles de Azure

En los ejemplos siguientes se crea una definición de roles de Azure a partir de las acciones descritas en las secciones anteriores. Los ejemplos se aplican en el nivel de grupo de recursos. Evalúe y pruebe si los ejemplos son lo suficientemente detallados para sus requisitos. Para su escenario, puede que necesite refinarlo a una galería de imágenes compartidas específica.

Las acciones en las imágenes permiten la lectura y escritura. Decida qué es adecuado para su entorno. Por ejemplo, cree un rol para permitir que Azure Image Builder lea imágenes del grupo de recursos *example-rg-1* y escriba imágenes en el grupo de recursos *example-rg-2*.

### <a name="custom-image-azure-role-example"></a>Ejemplo de rol de Azure de imagen personalizada

En el ejemplo siguiente se crea un rol de Azure para usar y distribuir una imagen personalizada de origen. A continuación, conceda el rol personalizado a la identidad administrada asignada por el usuario para Azure Image Builder.

Para simplificar la sustitución de valores en el ejemplo, establezca las siguientes variables primero. Reemplace la configuración del marcador de posición para establecer las variables.

| Configuración | Descripción |
|---------|-------------|
| \<Subscription ID\> | El identificador de suscripción a Azure |
| \<Resource group\> | Grupo de recursos para la imagen personalizada |

```powershell-interactive
$sub_id = "<Subscription ID>"
# Resource group - For Preview, image builder will only support creating custom images in the same Resource Group as the source managed image.
$imageResourceGroup = "<Resource group>"
$identityName = "aibIdentity"

# Use a web request to download the sample JSON description
$sample_uri="https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/solutions/12_Creating_AIB_Security_Roles/aibRoleImageCreation.json"
$role_definition="aibRoleImageCreation.json"

Invoke-WebRequest -Uri $sample_uri -Outfile $role_definition -UseBasicParsing

# Create a unique role name to avoid clashes in the same Azure Active Directory domain
$timeInt=$(get-date -UFormat "%s")
$imageRoleDefName="Azure Image Builder Image Def"+$timeInt

# Update the JSON definition placeholders with variable values
((Get-Content -path $role_definition -Raw) -replace '<subscriptionID>',$sub_id) | Set-Content -Path $role_definition
((Get-Content -path $role_definition -Raw) -replace '<rgName>', $imageResourceGroup) | Set-Content -Path $role_definition
((Get-Content -path $role_definition -Raw) -replace 'Azure Image Builder Service Image Creation Role', $imageRoleDefName) | Set-Content -Path $role_definition

# Create a custom role from the aibRoleImageCreation.json description file. 
New-AzRoleDefinition -InputFile $role_definition

# Get the user-identity properties
$identityNameResourceId=$(Get-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $identityName).Id
$identityNamePrincipalId=$(Get-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $identityName).PrincipalId

# Grant the custom role to the user-assigned managed identity for Azure Image Builder.
$parameters = @{
    ObjectId = $identityNamePrincipalId
    RoleDefinitionName = $imageRoleDefName
    Scope = '/subscriptions/' + $sub_id + '/resourceGroups/' + $imageResourceGroup
}

New-AzRoleAssignment @parameters
```

### <a name="existing-vnet-azure-role-example"></a>Ejemplo de rol de Azure de red virtual existente

En el ejemplo siguiente se crea un rol de Azure para usar y distribuir una imagen de red virtual existente. A continuación, conceda el rol personalizado a la identidad administrada asignada por el usuario para Azure Image Builder.

Para simplificar la sustitución de valores en el ejemplo, establezca las siguientes variables primero. Reemplace la configuración del marcador de posición para establecer las variables.

| Configuración | Descripción |
|---------|-------------|
| \<Subscription ID\> | El identificador de suscripción a Azure |
| \<Resource group\> | Grupos de recursos de red virtual |

```powershell-interactive
$sub_id = "<Subscription ID>"
$res_group = "<Resource group>"
$identityName = "aibIdentity"

# Use a web request to download the sample JSON description
$sample_uri="https://raw.githubusercontent.com/danielsollondon/azvmimagebuilder/master/solutions/12_Creating_AIB_Security_Roles/aibRoleNetworking.json"
$role_definition="aibRoleNetworking.json"

Invoke-WebRequest -Uri $sample_uri -Outfile $role_definition -UseBasicParsing

# Create a unique role name to avoid clashes in the same AAD domain
$timeInt=$(get-date -UFormat "%s")
$networkRoleDefName="Azure Image Builder Network Def"+$timeInt

# Update the JSON definition placeholders with variable values
((Get-Content -path $role_definition -Raw) -replace '<subscriptionID>',$sub_id) | Set-Content -Path $role_definition
((Get-Content -path $role_definition -Raw) -replace '<vnetRgName>', $res_group) | Set-Content -Path $role_definition
((Get-Content -path $role_definition -Raw) -replace 'Azure Image Builder Service Networking Role',$networkRoleDefName) | Set-Content -Path $role_definition

# Create a custom role from the aibRoleNetworking.json description file
New-AzRoleDefinition -InputFile $role_definition

# Get the user-identity properties
$identityNameResourceId=$(Get-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $identityName).Id
$identityNamePrincipalId=$(Get-AzUserAssignedIdentity -ResourceGroupName $imageResourceGroup -Name $identityName).PrincipalId

# Assign the custom role to the user-assigned managed identity for Azure Image Builder
$parameters = @{
    ObjectId = $identityNamePrincipalId
    RoleDefinitionName = $networkRoleDefName
    Scope = '/subscriptions/' + $sub_id + '/resourceGroups/' + $res_group
}

New-AzRoleAssignment @parameters
```

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, consulte [Introducción a Azure Image Builder](image-builder-overview.md).