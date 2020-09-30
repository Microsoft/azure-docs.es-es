---
title: Asignación de una directiva de acceso de Azure Key Vault
description: Cómo se usa Azure Portal, la CLI de Azure o Azure PowerShell para asignar una directiva de acceso de Key Vault a una identidad de aplicación o una entidad de servicio.
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 08/27/2020
ms.author: mbaldwin
ms.openlocfilehash: 817b9bfc0af054b344ca9f770085ac022a8e6eac
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89380574"
---
# <a name="assign-a-key-vault-access-policy-using-azure-powershell"></a>Asignación de una directiva de acceso de Key Vault mediante Azure PowerShell

Las directivas de acceso de Key Vault determinan si una entidad de servicio determinada, es decir, una aplicación o un grupo de usuarios, puede realizar distintas operaciones en los [secretos](../secrets/index.yml), las [claves](../keys/index.yml) y los [certificados](../certificates/index.yml) de Key Vault. Las directivas de acceso se pueden asignar mediante [Azure Portal](assign-access-policy-portal.md), la [CLI de Azure](assign-access-policy-cli.md) o Azure PowerShell (en este artículo).

[!INCLUDE [key-vault-access-policy-limits.md](../../../includes/key-vault-access-policy-limits.md)]

Para más información sobre cómo crear grupos en Azure Active Directory mediante Azure PowerShell, consulte [New-AzureADGroup](/powershell/module/azuread/new-azureadgroup) y [Add-AzADGroupMember](/powershell/module/az.resources/add-azadgroupmember).

## <a name="configure-powershell-and-sign-in"></a>Configuración de PowerShell e inicio de sesión

1. Para ejecutar los comandos localmente, instale [Azure PowerShell](/powershell/azure/) si todavía no lo ha hecho.

    Para ejecutar comandos directamente en la nube, use [Azure Cloud Shell](/azure/cloud-shell/overview).

1. Solo PowerShell local:

    1. Instale el [módulo PowerShell de Azure Active Directory](https://www.powershellgallery.com/packages/AzureAD).

    1. Inicie de sesión en Azure:

        ```azurepowershell-interactive
        Login-AzAccount
        ```
    
## <a name="acquire-the-object-id"></a>Obtención del identificador del objeto

Determine el identificador del objeto de la aplicación, el grupo o el usuario al que desea asignar la directiva de acceso:

- Aplicaciones y otras entidades de servicio: use el cmdlet [Get-AzADServicePrincipal](/powershell/module/az.resources/get-azadserviceprincipal) con el parámetro `-SearchString` para filtrar los resultados por el nombre de la entidad de servicio deseada:

    ```azurepowershell-interactive
    Get-AzADServicePrincipal -SearchString <search-string>
    ```

- Grupos: use el cmdlet [Get-AzADGroup](/powershell/module/az.resources/get-azadgroup?view=azps-2.7.0) con el parámetro `-SearchString` para filtrar los resultados por el nombre del grupo deseado:

    ```azurepowershell-interactive
    Get-AzADGroup -SearchString <search-string>
    ```
    
    En la salida, el identificador de objeto aparece como `Id`.

- Usuarios: use el cmdlet [Get-AzADUser](/powershell/module/az.resources/get-azaduser) y pase la dirección de correo electrónico del usuario al parámetro `-UserPrincipalName`.

    ```azurepowershell-interactive
     Get-AzAdUser -UserPrincipalName <email-address-of-user>
    ```

    En la salida, el identificador de objeto aparece como `Id`.

## <a name="assign-the-access-policy"></a>Asignación de una directiva de acceso

Use el cmdlet [Set-AzKeyVaultAccessPolicy](/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy) para asignar la directiva de acceso:

```azurepowershell-interactive
Set-AzKeyVaultAccessPolicy -VaultName <key-vault-name> -ObjectId <Id> -PermissionsToSecrets <secrets-permissions> -PermissionsToKeys <keys-permissions> -PermissionsToCertificates <certificate-permissions    
```

Solo necesita incluir `-PermissionsToSecrets`, `-PermissionsToKeys` y `-PermissionsToCertificates` al asignar permisos a esos tipos concretos. Los valores permitidos para `<secret-permissions>`, `<key-permissions>` y `<certificate-permissions>` se proporcionan en la documentación de [Set-AzKeyVaultAccessPolicy: Parámetros](/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy#parameters).

## <a name="next-steps"></a>Pasos siguientes

- [Seguridad en Azure Key Vault: Administración de identidades y acceso](overview-security.md#identity-and-access-management)
- [Protección del almacén de claves](secure-your-key-vault.md)
- [Guía del desarrollador de Azure Key Vault](developers-guide.md)
- [Procedimientos recomendados de Azure Key Vault](best-practices.md)
