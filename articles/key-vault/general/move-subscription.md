---
title: Azure Key Vault traslada un almacén a una suscripción diferente | Microsoft Docs
description: Instrucciones para trasladar un almacén de claves a una suscripción diferente.
services: key-vault
author: ShaneBala-keyvault
manager: ravijan
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: conceptual
ms.date: 05/05/2020
ms.author: sudbalas
Customer intent: As a key vault administrator, I want to move my vault to another subscription.
ms.openlocfilehash: b95cecdc5470e50830371b7a6556f9526dae7591
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095569"
---
# <a name="moving-an-azure-key-vault-to-another-subscription"></a>Traslado de Azure Key Vault a otra suscripción

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="overview"></a>Información general

> [!IMPORTANT]
> **Trasladar un almacén de claves a otra suscripción, producirá un cambio importante en el entorno.**
> Asegúrese de entender el impacto de este cambio y siga atentamente las instrucciones de este artículo antes de decidir si desea trasladar el almacén de claves a una nueva suscripción.
> Si usa identidades de servicio administradas (MSI), lea las instrucciones posteriores al traslado, al final del documento. 

Cuando se crea un almacén de claves, se asocia automáticamente a un identificador de inquilino de Azure Active Directory para la suscripción en la que ha sido creado. Las entradas de la directiva de acceso también se asocian con este identificador de inquilino. Al trasladar una suscripción de Azure del inquilino A al inquilino B, las entidades de servicio (usuarios y aplicaciones) del inquilino B no podrán acceder a los almacenes de claves ya existentes. Para corregir este problema, es necesario:

* Cambiar el identificador de inquilino asociado a todas las instancias de Key Vault existentes en la suscripción al inquilino B.
* Quitar todas las entradas de la directiva de acceso existentes.
* Agregar nuevas entradas de la directiva de acceso asociadas al inquilino B.

## <a name="limitations"></a>Limitaciones

Algunas entidades de servicio (usuarios y aplicaciones) están enlazadas a un inquilino específico. Si traslada el almacén de claves a una suscripción de otro inquilino, existe la posibilidad de que no pueda restaurar el acceso a una entidad de servicio específica. Asegúrese de que todas las entidades de servicio esenciales existen en el inquilino al que va a trasladar el almacén de claves.

## <a name="design-considerations"></a>Consideraciones de diseño

Su organización puede haber implementado Azure Policy con cumplimiento o exclusión al nivel de suscripción. Puede haber un conjunto diferente de asignaciones de directiva en la suscripción en la que actualmente se encuentra el almacén de claves y la suscripción a la que se va a trasladar el almacén de claves. Un conflicto en los requisitos de la directiva tiene el potencial de interrumpir las aplicaciones.

### <a name="example"></a>Ejemplo

Tiene una aplicación conectada al almacén de claves que crea certificados válidos durante dos años. La suscripción a la que está intentando trasladar el almacén de claves tiene una asignación de directiva que bloquea la creación de certificados válidos durante más de un año. Después de trasladar el almacén de claves a la nueva suscripción, la operación de creación de un certificado válido durante dos años se bloqueará mediante una asignación de directiva de Azure.

### <a name="solution"></a>Solución

Asegúrese de ir a la página de Azure Policy en Azure Portal y examine las asignaciones de directiva de su suscripción actual, así como la suscripción a la que está trasladando y asegúrese de que no haya ninguna discrepancia.

## <a name="prerequisites"></a>Prerrequisitos

* Acceso de un nivel de colaborador o superior a la suscripción actual en la que existe el almacén de claves.
* Acceso de nivel de colaborador o superior a la suscripción a la que quiere trasladar el almacén de claves.
* Un grupo de recursos en la nueva suscripción.

## <a name="procedure"></a>Procedimiento

### <a name="initial-steps-moving-key-vault"></a>Pasos iniciales (trasladar el almacén de claves)

1. Iniciar sesión en Azure Portal
2. Vaya a almacén de claves.
3. Haga clic en la pestaña "Introducción".
4. Seleccione el botón «Mover»
5. Seleccione «Mover a otra suscripción» de las opciones de la lista desplegable
6. Seleccione el grupo de recursos a donde quiere trasladar el almacén de claves
7. Confirme la advertencia relacionada con el traslado de recursos.
8. Seleccione Aceptar.

### <a name="additional-steps-post-move"></a>Pasos adicionales (después del traslado)

Ahora que ha trasladado el almacén de claves a la nueva suscripción, debe actualizar el identificador de inquilino y quitar las directivas de acceso anteriores. Estos son los tutoriales de estos pasos en PowerShell y la CLI de Azure.

```azurepowershell
Select-AzSubscription -SubscriptionId <your-subscriptionId>                # Select your Azure Subscription
$vaultResourceId = (Get-AzKeyVault -VaultName myvault).ResourceId          # Get your key vault's Resource ID 
$vault = Get-AzResource –ResourceId $vaultResourceId -ExpandProperties     # Get the properties for your key vault
$vault.Properties.TenantId = (Get-AzContext).Tenant.TenantId               # Change the Tenant that your key vault resides in
$vault.Properties.AccessPolicies = @()                                     # Access policies can be updated with real
                                                                           # applications/users/rights so that it does not need to be                             # done after this whole activity. Here we are not setting 
                                                                           # any access policies. 
Set-AzResource -ResourceId $vaultResourceId -Properties $vault.Properties  # Modifies the key vault's properties.

Clear-AzContext                                                            #Clear the context from PowerShell
Connect-AzAccount                                                          #Log in again to confirm you have the correct tenant id
````

```azurecli
az account set -s <your-subscriptionId>                                    # Select your Azure Subscription
tenantId=$(az account show --query tenantId)                               # Get your tenantId
az keyvault update -n myvault --remove Properties.accessPolicies           # Remove the access policies
az keyvault update -n myvault --set Properties.tenantId=$tenantId          # Update the key vault tenantId
```

Ahora que el almacén está asociado al identificador de inquilino correcto y que se han quitado las entradas de directiva de acceso antiguas, establezca nuevas entradas de directiva de acceso con el cmdlet [Set-AzKeyVaultAccessPolicy](/powershell/module/az.keyvault/Set-azKeyVaultAccessPolicy) de Azure PowerShell o el comando [az keyvault set-policy](/cli/azure/keyvault?view=azure-cli-latest#az-keyvault-set-policy) de la CLI de Azure.

Si usa una identidad administrada para los recursos de Azure, tendrá que actualizarla también al nuevo inquilino de Azure AD. Para más información sobre las identidades administradas, consulte [Autenticación de Key Vault con una identidad administrada](managed-identity.md).

Si usa MSI, también tendrá que actualizar la identidad de MSI, ya que la anterior ya no estará en el inquilino de AAD correcto. Consulte los siguientes documentos para ayudar a resolver este problema. 

* [Actualización de MSI](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/known-issues#transferring-a-subscription-between-azure-ad-directories)
* [Transferencia de una suscripción a un nuevo directorio](https://docs.microsoft.com/azure/role-based-access-control/transfer-subscription)


