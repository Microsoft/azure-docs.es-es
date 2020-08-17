---
title: 'Concesión de permisos para que las aplicaciones tengan acceso a un almacén de claves de Azure: Azure Key Vault | Microsoft Docs'
description: Obtenga información sobre cómo conceder permisos para que muchas aplicaciones tengan acceso a almacén de claves.
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 09/27/2019
ms.author: mbaldwin
ms.openlocfilehash: 8369c507ee7a9e11969c17bbaac7c6e31892e196
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87406272"
---
# <a name="provide-key-vault-authentication-with-an-access-control-policy"></a>Autenticación en Key Vault con una directiva de control de acceso

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

La manera más sencilla de autenticar una aplicación basada en la nube en Key Vault es con una identidad administrada; consulte [Uso de identidades administradas de App Service para acceder a Azure Key Vault](managed-identity.md) para más información.  Si va a crear una aplicación local, al realizar el desarrollo local (o si no puede usar una identidad administrada), puede registrar manualmente una entidad de servicio y proporcionar acceso a su almacén de claves mediante una directiva de control de acceso.  

Key Vault admite hasta 1024 entradas de directiva de acceso, donde cada entrada concede un conjunto de permisos distinto a una "entidad de servicio":   Por ejemplo, este es el modo en que la aplicación de consola de [Inicio rápido: Biblioteca cliente de Azure Key Vault para .NET](../secrets/quick-create-net.md) accede al almacén de claves.

Para la información completa sobre el control de acceso a Key Vault, consulte [Seguridad en Azure Key Vault: Administración de identidades y acceso](overview-security.md#identity-and-access-management). Para obtener información completa sobre el control de acceso, consulte: 

- [Claves](../keys/index.yml)
- [Control de acceso al secreto](../secrets/index.yml)
- [Control de acceso al certificado](../certificates/index.yml)

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="prerequisites"></a>Requisitos previos

- Un almacén de claves. Puede usar un almacén de claves existente o crear uno nuevo siguiendo los pasos de uno de estos artículos de inicio rápido:
   - [Creación de un almacén de claves con la CLI de Azure](../secrets/quick-create-cli.md)
   - [Creación de un almacén de claves con Azure PowerShell](../secrets/quick-create-powershell.md)
   - [Creación de un almacén de claves con Azure Portal](../secrets/quick-create-portal.md)
- La [CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest) o [Azure PowerShell](/powershell/azure/) Como alternativa, puede usar [Azure Portal](https://portal.azure.com).

## <a name="grant-access-to-your-key-vault"></a>Concesión de acceso al almacén de claves

Cada entrada de directiva de acceso al almacén de claves concede un conjunto de permisos distinto para una entidad de servicio:

- **Aplicación** Si la aplicación se basa en la nube, si se puede, [use mejor una identidad administrada para acceder a Azure Key Vault](managed-identity.md).
- **Grupo de Azure AD** Aunque el almacén de claves solo admite 1024 entradas de directiva de acceso, puede agregar varias aplicaciones y usuarios a un grupo de Azure AD y, después, agregar este grupo como entrada individual a la directiva de control de acceso.
- **Usuario** Conceder a los usuarios acceso directo a un almacén de claves **no se recomienda**. Lo ideal es que los usuarios se agreguen a un grupo de Azure AD, al que se le concederá acceso al almacén de claves. Consulte [Seguridad en Azure Key Vault: Administración de identidades y acceso](overview-security.md#identity-and-access-management).


### <a name="get-the-objectid"></a>Obtención del valor de objectID

Para conceder acceso al almacén de claves a una aplicación, un grupo de Azure AD o un usuario, primero debe obtener el valor de objectId.

#### <a name="applications"></a>APLICACIONES

El valor de objectId de una aplicación corresponde a su entidad de servicio asociada. Para todos los detalles sobre las entidades de servicio, consulte [Objetos de aplicación y de entidad de servicio de Azure Active Directory](../../active-directory/develop/app-objects-and-service-principals.md). 

Hay dos maneras de obtener el valor de objectId de una aplicación.  La primera es registrar la aplicación en Azure Active Directory. Para ello, siga los pasos de [Inicio rápido: Registro de una aplicación en la plataforma de identidad de Microsoft](../../active-directory/develop/quickstart-register-app.md). Una vez completado el registro, el valor de objectID se mostrará como "ID de aplicación (cliente)".

La segunda manera consiste en crear una entidad de servicio en una ventana de terminal. Con la CLI de Azure, use el comando [az ad sp create-for-rbac](/cli/azure/ad/sp?view=azure-cli-latest#az-ad-sp-create-for-rbac) y proporcione un nombre de entidad de seguridad de servicio único en la marca -n con el formato "http://&lt;mi-nombre-de-entidad-de-seguridad-de-servicio-único&gt;".

```azurecli-interactive
az ad sp create-for-rbac -n "http://<my-unique-service-principal-name"
```

El valor de objectId se mostrará en la salida como `clientID`.

Con Azure PowerShell, use el cmdlet [New-AzADServicePrincipal](/powershell/module/Az.Resources/New-AzADServicePrincipal?view=azps-2.7.0).


```azurepowershell-interactive
New-AzADServicePrincipal -DisplayName <my-unique-service-principal-name>
```

El valor de objectId se mostrará en la salida como `Id` (no `ApplicationId`).

#### <a name="azure-ad-groups"></a>Grupos de Azure AD

Puede agregar varias aplicaciones y usuarios a un grupo de Azure AD y concederle acceso al grupo al almacén de claves.  Para más información, consulte la sección [Creación e incorporación de miembros a un grupo de Azure AD](#creating-and-adding-members-to-an-azure-ad-group) a continuación.

Para buscar el valor de objectId de un grupo de Azure AD con la CLI de Azure, use el comando [az ad group list](/cli/azure/ad/group?view=azure-cli-latest#az-ad-group-list). Debido al gran número de grupos que pueden encontrarse en su organización, también debe proporcionar una cadena de búsqueda al parámetro `--display-name`.

```azurecli-interactive
az ad group list --display-name <search-string>
```
El valor de objectId se devolverá en el JSON:

```json
    "objectId": "48b21bfb-74d6-48d2-868f-ff9eeaf38a64",
    "objectType": "Group",
    "odata.type": "Microsoft.DirectoryServices.Group",
```

Para buscar el valor de objectId de un grupo de Azure AD con Azure PowerShell, use el cmdlet [Get-AzADGroup](/powershell/module/az.resources/get-azadgroup?view=azps-2.7.0). Debido al gran número de grupos que pueden encontrarse en su organización, es probable que también desee proporcionar una cadena de búsqueda al parámetro `-SearchString`.

```azurepowershell-interactive
Get-AzADGroup -SearchString <search-string>
```

En la salida, el valor de objectId aparece como `Id`:

```console
...
Id                    : 1cef38c4-388c-45a9-b5ae-3d88375e166a
...
```

#### <a name="users"></a>Usuarios

También puede agregar un usuario individual a una directiva de control de acceso al almacén de claves, no obstante, **esto no se recomienda**. En su lugar, es preferible incorporar usuarios a un grupo de Azure AD y agregar el grupo a las directivas.

Sin embargo, si desea buscar un usuario con la CLI de Azure, use el comando [az ad user show](/cli/azure/ad/user?view=azure-cli-latest#az-ad-user-show) y pase la dirección de correo electrónico del usuario al parámetro `--id`.


```azurecli-interactive
az ad user show --id <email-address-of-user>
```

El valor de objectId del usuario se devolverá en la salida:

```console
  ...
  "objectId": "f76a2a6f-3b6d-4735-9abd-14dccbf70fd9",
  "objectType": "User",
  ...
```

Para buscar un usuario con Azure PowerShell, use el cmdlet [Get-AzADUser](/powershell/module/az.resources/get-azaduser?view=azps-2.7.0) y pase la dirección de correo electrónico del usuario al parámetro `-UserPrincipalName`.

```azurepowershell-interactive
 Get-AzAdUser -UserPrincipalName <email-address-of-user>
```

El valor de objectId del usuario se devolverá en la salida como `Id`.

```console
...
Id                : f76a2a6f-3b6d-4735-9abd-14dccbf70fd9
Type              :
```

### <a name="give-the-principal-access-to-your-key-vault"></a>Acceso de la entidad de servicio al almacén de claves

Ahora que tiene un valor de objectID para la entidad de servicio, puede crear una directiva de acceso al almacén de claves que le proporcione los permisos de obtención, enumeración, establecimiento y eliminación tanto para las claves como para los secretos, así como los permisos adicionales que desee.

Con la CLI de Azure, esto se hace al pasar el valor de objectId al comando [az keyvault set-policy](/cli/azure/keyvault?view=azure-cli-latest#az-keyvault-set-policy).

```azurecli-interactive
az keyvault set-policy -n <your-unique-keyvault-name> --spn <ApplicationID-of-your-service-principal> --secret-permissions get list set delete --key-permissions create decrypt delete encrypt get list unwrapKey wrapKey
```

Con Azure PowerShell, esto se hace al pasar el valor de objectId al cmdlet [Set-AzKeyVaultAccessPolicy](/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy?view=azps-2.7.0). 

```azurepowershell-interactive
Set-AzKeyVaultAccessPolicy -VaultName <your-key-vault-name> -PermissionsToKeys create,decrypt,delete,encrypt,get,list,unwrapKey,wrapKey -PermissionsToSecrets get,list,set,delete -ObjectId <Id>

```

## <a name="creating-and-adding-members-to-an-azure-ad-group"></a>Creación e incorporación de miembros a un grupo de Azure AD

Puede crear un grupo de Azure AD, agregar aplicaciones y usuarios al grupo y conceder al grupo acceso al almacén de claves.  Esto permite agregar una serie de aplicaciones a un almacén de claves como una sola entrada de directiva de acceso y elimina la necesidad de proporcionar a los usuarios acceso directo al almacén de claves (procedimiento no recomendado). Para más detalles, consulte [Administración del acceso a recursos y aplicaciones con grupos en Azure Active Directory](../../active-directory/fundamentals/active-directory-manage-groups.md).

### <a name="additional-prerequisites"></a>Requisitos previos adicionales

Además de los [requisitos previos anteriores](#prerequisites), necesitará permisos de creación/edición de grupos en el inquilino de Azure Active Directory. Si no tiene permisos, debe ponerse en contacto con el administrador de Azure Active Directory.

Si tiene previsto usar PowerShell, también necesitará el [módulo de PowerShell para Azure AD](https://www.powershellgallery.com/packages/AzureAD/2.0.2.50).

### <a name="create-an-azure-active-directory-group"></a>Creación de un grupo de Azure Active Directory

Cree un grupo de Azure Active Directory con el comando [az ad group create](/cli/azure/ad/group?view=azure-cli-latest#az-ad-group-create) de la CLI de Azure o con el cmdlet [New-AzureADGroup](/powershell/module/azuread/new-azureadgroup?view=azureadps-2.0) de Azure PowerShell.


```azurecli-interactive
az ad group create --display-name <your-group-display-name> --mail-nickname <your-group-mail-nickname>
```

```powershell
New-AzADGroup -DisplayName <your-group-display-name> -MailNickName <your-group-mail-nickname>
```

En cualquier caso, tome nota del valor de GroupId de los grupos recién creados, ya que lo necesitará en los pasos siguientes.

### <a name="find-the-objectids-of-your-applications-and-users"></a>Búsqueda de los valores de objectId de las aplicaciones y los usuarios

Encontrará los valores de objectId de las aplicaciones mediante la CLI de Azure, con el comando [az ad sp list](/cli/azure/ad/sp?view=azure-cli-latest#az-ad-sp-list) y el parámetro `--show-mine`.

```azurecli-interactive
az ad sp list --show-mine
```

Busque los valores de objectId de las aplicaciones con Azure PowerShell y el cmdlet [Get-AzADServicePrincipal](/powershell/module/az.resources/get-azadserviceprincipal?view=azps-2.7.0), al pasar una cadena de búsqueda al parámetro `-SearchString`.

```azurepowershell-interactive
Get-AzADServicePrincipal -SearchString <search-string>
```

Para buscar los valores de objectId de los usuarios, siga los pasos que se indican en la sección [Usuarios](#users) anterior.

### <a name="add-your-applications-and-users-to-the-group"></a>Incorporación de las aplicaciones y los usuarios al grupo

Ahora, agregue los valores de objectId al grupo de Azure AD recién creado.

Con la CLI de Azure, use [az ad group member add](/cli/azure/ad/group/member?view=azure-cli-latest#az-ad-group-member-add) y pase el valor de objectId al parámetro `--member-id`.


```azurecli-interactive
az ad group member add -g <groupId> --member-id <objectId>
```

Con Azure PowerShell, use el cmdlet [Add-AzADGroupMember](/powershell/module/az.resources/add-azadgroupmember?view=azps-2.7.0) y pase el valor de objectId al parámetro `-MemberObjectId`.

```azurepowershell-interactive
Add-AzADGroupMember -TargetGroupObjectId <groupId> -MemberObjectId <objectId> 
```

### <a name="give-the-ad-group-access-to-your-key-vault"></a>Concesión de acceso al grupo de AD al almacén de claves

Por último, conceda al grupo de AD permisos en el almacén de claves mediante el comando [az keyvault set-policy](/cli/azure/keyvault?view=azure-cli-latest#az-keyvault-set-policy) de la CLI de Azure o el cmdlet [Set-AzKeyVaultAccessPolicy](/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy?view=azps-2.7.0) de Azure PowerShell. Para ejemplos, consulte la sección anterior sobre la [concesión de acceso al almacén de claves a la aplicación, al grupo de Azure AD o a los usuarios](#give-the-principal-access-to-your-key-vault).

La aplicación también necesita al menos un rol de Administración de identidades y acceso (IAM) asignado al almacén de claves. De lo contrario, no podrá iniciar sesión y se producirá un error por derechos insuficientes para acceder a la suscripción.

> [!WARNING]
> Grupos de Azure AD con identidades administradas puede requerir hasta ocho horas para actualizar el token y entrar en vigor.

## <a name="next-steps"></a>Pasos siguientes

- [Seguridad en Azure Key Vault: Administración de identidades y acceso](overview-security.md#identity-and-access-management)
- [Uso de identidades administradas de App Service para la autenticación en Key Vault](managed-identity.md)
- [Protección del almacén de claves](secure-your-key-vault.md)
- [Guía del desarrollador de Azure Key Vault](developers-guide.md)
- Consulte los [procedimientos recomendados de Azure Key Vault](best-practices.md).
