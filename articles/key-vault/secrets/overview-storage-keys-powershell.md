---
title: 'Cuenta de almacenamiento administrado por Azure Key Vault: versión de PowerShell'
description: La característica de cuenta de almacenamiento administrado ofrece una integración sin problemas entre Azure Key Vault y una cuenta de almacenamiento de Azure.
ms.topic: tutorial
ms.service: key-vault
ms.subservice: secrets
author: msmbaldwin
ms.author: mbaldwin
manager: rkarlin
ms.date: 09/10/2019
ms.openlocfilehash: 8e8479179aa74f2fb2ead41dec28d247de9657c3
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88585107"
---
# <a name="manage-storage-account-keys-with-key-vault-and-azure-powershell"></a>Administración de claves de cuenta de almacenamiento con Key Vault y Azure PowerShell

Una cuenta de Azure Storage usa credenciales que constan de un nombre de cuenta y una clave. La clave se genera automáticamente y actúa como una contraseña y no como una clave criptográfica. Key Vault, para administrar las claves de la cuenta de almacenamiento, las regenera periódicamente en una cuenta de almacenamiento y proporciona tokens de firma de acceso compartido para el acceso delegado a los recursos de la cuenta de almacenamiento.

Puede usar la característica de clave de cuenta de almacenamiento administrada de Key Vault para mostrar (sincronizar) las claves con una cuenta de almacenamiento de Azure y volver a generar (girar) las claves periódicamente. Puede administrar claves para las cuentas de almacenamiento y las cuentas de almacenamiento clásicas.

Cuando use la característica de clave de cuenta de almacenamiento administrada, tenga en cuenta los puntos siguientes:

- Los valores de clave nunca se devuelven como respuesta a un autor de la llamada.
- Solo Key Vault debe administrar las claves de cuenta de almacenamiento. No administre las claves por su cuenta y evite interferir en los procesos de Key Vault.
- Solo un único objeto de Key Vault debe administrar las claves de cuenta de almacenamiento. No permita la administración de claves desde varios objetos.
- Puede solicitar a Key Vault que administre la cuenta de almacenamiento con una entidad de seguridad de usuario, pero no con una entidad de servicio.
- Regenere las claves solo con Key Vault. No regenere manualmente las claves de cuenta de almacenamiento.

Es recomendable usar la integración de Azure Storage con Azure Active Directory (Azure AD), el servicio de administración de acceso y de identidades basado en la nube de Microsoft. La integración de Azure AD está disponible para los [Blobs y las colas de Azure](../../storage/common/storage-auth-aad.md) y proporciona acceso basado en tokens de OAuth2 a Azure Storage (al igual que Azure Key Vault).

Azure AD le permite autenticar la aplicación cliente mediante una identidad de aplicación o usuario, en lugar de las credenciales de cuenta de almacenamiento. Puede usar una [identidad administrada de Azure AD](/azure/active-directory/managed-identities-azure-resources/) cuando realice la ejecución en Azure. Las identidades administradas eliminan la necesidad de autenticación del cliente y de almacenar las credenciales en la aplicación o con ella.

Azure AD usa el control de acceso basado en rol (RBAC) para administrar la autorización, que también es compatible con Key Vault.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="service-principal-application-id"></a>Id. de aplicación de la entidad de servicio

Un inquilino de Azure AD proporciona a cada aplicación registrada una [entidad de servicio](/azure/active-directory/develop/developer-glossary#service-principal-object). La entidad de servicio se usa como id. de aplicación, que se utiliza durante la configuración de la autorización para acceder a otros recursos de Azure mediante RBAC.

Key Vault es una aplicación de Microsoft que previamente se ha registrado en todos los inquilinos de Azure AD. Key Vault está registrado con el mismo identificador de aplicación en cada nube de Azure.

| Inquilinos | Nube | Identificador de aplicación |
| --- | --- | --- |
| Azure AD | Azure Government | `7e7c393b-45d0-48b1-a35e-2905ddf8183c` |
| Azure AD | Pública de Azure | `cfa8b339-82a2-471a-a3c9-0fc0be7a4093` |
| Otros  | Any | `cfa8b339-82a2-471a-a3c9-0fc0be7a4093` |

## <a name="prerequisites"></a>Requisitos previos

Para completar esta guía, antes debe completar los pasos siguientes:

- [Instale el módulo de Azure PowerShell](/powershell/azure/install-az-ps?view=azps-2.6.0).
- [Creación de un Almacén de claves](quick-create-powershell.md)
- [Creación de una cuenta de almacenamiento de Azure](../../storage/common/storage-account-create.md?tabs=azure-powershell). El nombre de la cuenta de almacenamiento solo debe contener letras minúsculas y números. El nombre debe tener entre 3 y 24 caracteres.
      

## <a name="manage-storage-account-keys"></a>Administración de las claves de cuenta de almacenamiento

### <a name="connect-to-your-azure-account"></a>Conexión a la cuenta de Azure

Autentique la sesión de PowerShell mediante el cmdlet [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount?view=azps-2.5.0). 

```azurepowershell-interactive
Connect-AzAccount
```
Si tiene varias suscripciones de Azure, puede enumerarlas con el cmdlet [Get-AzSubscription](/powershell/module/az.accounts/get-azsubscription?view=azps-2.5.0) y especificar la suscripción que desea usar con el cmdlet [Set-AzContext](/powershell/module/az.accounts/set-azcontext?view=azps-2.5.0). 

```azurepowershell-interactive
Set-AzContext -SubscriptionId <subscriptionId>
```

### <a name="set-variables"></a>Configuración de variables

En primer lugar, establezca las variables que usarán los cmdlets de PowerShell en los pasos siguientes. Asegúrese de actualizar los marcadores de posición <YourResourceGroupName>, <YourStorageAccountName> y <YourKeyVaultName> y establezca $keyVaultSpAppId en `cfa8b339-82a2-471a-a3c9-0fc0be7a4093` (según se especifica en [Id. de aplicación de la entidad de servicio](#service-principal-application-id) anteriormente).

También usaremos los cmdlets [Get-AzContext](/powershell/module/az.accounts/get-azcontext?view=azps-2.6.0) y [Get-AzStorageAccount](/powershell/module/az.storage/get-azstorageaccount?view=azps-2.6.0) de Azure PowerShell para obtener su identificador de usuario y el contexto de su cuenta de Azure Storage.

```azurepowershell-interactive
$resourceGroupName = <YourResourceGroupName>
$storageAccountName = <YourStorageAccountName>
$keyVaultName = <YourKeyVaultName>
$keyVaultSpAppId = "cfa8b339-82a2-471a-a3c9-0fc0be7a4093"
$storageAccountKey = "key1" #(key1 or key2 are allowed)

# Get your User Id
$userId = (Get-AzContext).Account.Id

# Get a reference to your Azure storage account
$storageAccount = Get-AzStorageAccount -ResourceGroupName $resourceGroupName -StorageAccountName $storageAccountName

```
>[!Note]
> Para la cuenta de almacenamiento clásica, use "primary" y "secondary" para $storageAccountKey <br>
> Use "Get-AzResource -Name 'ClassicStorageAccountName' -ResourceGroupName $resourceGroupName" en lugar de "Get-AzStorageAccount "para la cuenta de almacenamiento clásica

### <a name="give-key-vault-access-to-your-storage-account"></a>Proporcionar a Key Vault acceso a la cuenta de almacenamiento

Para que Key Vault tenga acceso y pueda administrar las claves de cuenta de almacenamiento, debe autorizar su acceso a la cuenta de almacenamiento. La aplicación Key Vault necesita permisos para *mostrar* y *regenerar* las claves de la cuenta de almacenamiento. Estos permisos se habilitan a través del rol integrado de Azure [Rol de servicio de operador de claves de cuentas de almacenamiento](/azure/role-based-access-control/built-in-roles#storage-account-key-operator-service-role). 

Para asignar este rol a la entidad de servicio de Key Vault, que limita el ámbito a la cuenta de almacenamiento, use el cmdlet [New-AzRoleAssignment](/powershell/module/az.resources/new-azroleassignment?view=azps-2.6.0) de Azure PowerShell.

```azurepowershell-interactive
# Assign Azure role "Storage Account Key Operator Service Role" to Key Vault, limiting the access scope to your storage account. For a classic storage account, use "Classic Storage Account Key Operator Service Role." 
New-AzRoleAssignment -ApplicationId $keyVaultSpAppId -RoleDefinitionName 'Storage Account Key Operator Service Role' -Scope $storageAccount.Id
```

Tras la correcta asignación del rol, debería ver una salida similar a la del siguiente ejemplo:

```console
RoleAssignmentId   : /subscriptions/03f0blll-ce69-483a-a092-d06ea46dfb8z/resourceGroups/rgContoso/providers/Microsoft.Storage/storageAccounts/sacontoso/providers/Microsoft.Authorization/roleAssignments/189cblll-12fb-406e-8699-4eef8b2b9ecz
Scope              : /subscriptions/03f0blll-ce69-483a-a092-d06ea46dfb8z/resourceGroups/rgContoso/providers/Microsoft.Storage/storageAccounts/sacontoso
DisplayName        : Azure Key Vault
SignInName         :
RoleDefinitionName : storage account Key Operator Service Role
RoleDefinitionId   : 81a9662b-bebf-436f-a333-f67b29880f12
ObjectId           : 93c27d83-f79b-4cb2-8dd4-4aa716542e74
ObjectType         : ServicePrincipal
CanDelegate        : False
```

Si Key Vault ya se ha agregado al rol en la cuenta de almacenamiento, recibirá un error *"La asignación de roles ya existe."* . También puede comprobar la asignación de roles con la página "Control de acceso (IAM)" de la cuenta de almacenamiento en Azure Portal.  

### <a name="give-your-user-account-permission-to-managed-storage-accounts"></a>Concesión del permiso de cuenta de usuario a las cuentas de almacenamiento administradas

Use el cmdlet [Set-AzKeyVaultAccessPolicy](/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy?view=azps-2.6.0) de Azure PowerShell para actualizar la directiva de acceso de Key Vault y conceder permisos de cuenta de almacenamiento a su cuenta de usuario.

```azurepowershell-interactive
# Give your user principal access to all storage account permissions, on your Key Vault instance

Set-AzKeyVaultAccessPolicy -VaultName $keyVaultName -UserPrincipalName $userId -PermissionsToStorage get, list, delete, set, update, regeneratekey, getsas, listsas, deletesas, setsas, recover, backup, restore, purge
```

Tenga en cuenta que los permisos para las cuentas de almacenamiento no están disponibles en la página "Directivas de acceso" de la cuenta de almacenamiento en Azure Portal.

### <a name="add-a-managed-storage-account-to-your-key-vault-instance"></a>Adición de una cuenta de almacenamiento administrada a la instancia de Key Vault

Use el cmdlet [Add-AzKeyVaultManagedStorageAccount](/powershell/module/az.keyvault/add-azkeyvaultmanagedstorageaccount?view=azps-2.6.0) de Azure PowerShell para crear una cuenta de almacenamiento administrada en la instancia de Key Vault. El modificador `-DisableAutoRegenerateKey` especifica que no se deben regenerar las claves de la cuenta de almacenamiento.

```azurepowershell-interactive
# Add your storage account to your Key Vault's managed storage accounts

Add-AzKeyVaultManagedStorageAccount -VaultName $keyVaultName -AccountName $storageAccountName -AccountResourceId $storageAccount.Id -ActiveKeyName $storageAccountKey -DisableAutoRegenerateKey
```

Tras la correcta incorporación de la cuenta de almacenamiento sin regeneración de claves, debería ver una salida similar a la del siguiente ejemplo:

```console
Id                  : https://kvcontoso.vault.azure.net:443/storage/sacontoso
Vault Name          : kvcontoso
AccountName         : sacontoso
Account Resource Id : /subscriptions/03f0blll-ce69-483a-a092-d06ea46dfb8z/resourceGroups/rgContoso/providers/Microsoft.Storage/storageAccounts/sacontoso
Active Key Name     : key1
Auto Regenerate Key : False
Regeneration Period : 90.00:00:00
Enabled             : True
Created             : 11/19/2018 11:54:47 PM
Updated             : 11/19/2018 11:54:47 PM
Tags                : 
```

### <a name="enable-key-regeneration"></a>Habilitación de la regeneración de clave

Si desea que Key Vault regenere las claves de cuenta de almacenamiento periódicamente, puede usar el cmdlet [Add-AzKeyVaultManagedStorageAccount](/powershell/module/az.keyvault/add-azkeyvaultmanagedstorageaccount?view=azps-2.6.0) de Azure PowerShell para establecer un período de regeneración. En el ejemplo siguiente se establece un período de regeneración de tres días. Cuando es el momento de girar, Key Vault regenera la clave que no está activa y, a continuación, establece la clave recién creada como activa. Solo se usa una de las claves para emitir tokens de SAS en cualquier momento. Esta es la clave activa.

```azurepowershell-interactive
$regenPeriod = [System.Timespan]::FromDays(3)

Add-AzKeyVaultManagedStorageAccount -VaultName $keyVaultName -AccountName $storageAccountName -AccountResourceId $storageAccount.Id -ActiveKeyName $storageAccountKey -RegenerationPeriod $regenPeriod
```

Tras la correcta incorporación de la cuenta de almacenamiento con regeneración de claves, debería ver una salida similar a la del siguiente ejemplo:

```console
Id                  : https://kvcontoso.vault.azure.net:443/storage/sacontoso
Vault Name          : kvcontoso
AccountName         : sacontoso
Account Resource Id : /subscriptions/03f0blll-ce69-483a-a092-d06ea46dfb8z/resourceGroups/rgContoso/providers/Microsoft.Storage/storageAccounts/sacontoso
Active Key Name     : key1
Auto Regenerate Key : True
Regeneration Period : 3.00:00:00
Enabled             : True
Created             : 11/19/2018 11:54:47 PM
Updated             : 11/19/2018 11:54:47 PM
Tags                : 
```

## <a name="shared-access-signature-tokens"></a>Tokens de firma de acceso compartido

También puede pedir a Key Vault que genere tokens de firma de acceso compartido. Una firma de acceso compartido ofrece acceso delegado a recursos en la cuenta de almacenamiento. Puede conceder a los clientes acceso a los recursos de su cuenta de almacenamiento sin compartir las claves de la cuenta. Una firma de acceso compartido le proporciona una forma segura de compartir los recursos de almacenamiento sin poner en peligro las claves de cuenta.

Los comandos de esta sección completan las acciones siguientes:

- Establecer una definición de firma de acceso compartido de cuenta. 
- Crear un token de firma de acceso compartido de cuenta para los servicios Blob, File, Table y Queue. El token se crea para los tipos de recurso Servicio, Contenedor y Objeto. El token se crea con todos los permisos, a través de https, y con las fechas de inicio y finalización especificadas.
- Establecer una definición de firma de acceso compartido de almacenamiento administrado de Key Vault en el almacén. La definición tiene el URI de plantilla del token de firma de acceso compartido que se creó. La definición tiene el tipo de firma de acceso compartido `account` y es válido durante N días.
- Compruebe que la firma de acceso compartido se haya guardado en el almacén de claves como secreto.
- 
### <a name="set-variables"></a>Configuración de variables

En primer lugar, establezca las variables que usarán los cmdlets de PowerShell en los pasos siguientes. Asegúrese de actualizar los marcadores de posición <YourStorageAccountName> y <YourKeyVaultName>.

También usaremos el cmdlet [New-AzStorageContext](/powershell/module/az.storage/new-azstoragecontext?view=azps-2.6.0) de Azure PowerShell para obtener el contexto de su cuenta de Azure Storage.

```azurepowershell-interactive
$storageAccountName = <YourStorageAccountName>
$keyVaultName = <YourKeyVaultName>

$storageContext = New-AzStorageContext -StorageAccountName $storageAccountName -Protocol Https -StorageAccountKey Key1 #(or "Primary" for Classic Storage Account)
```

### <a name="create-a-shared-access-signature-token"></a>Creación de un token de firma de acceso compartido

Cree una definición de firma de acceso compartido mediante el cmdlet [New-AzStorageAccountSASToken](/powershell/module/az.storage/new-azstorageaccountsastoken?view=azps-2.6.0) de Azure PowerShell.
 
```azurepowershell-interactive
$start = [System.DateTime]::Now.AddDays(-1)
$end = [System.DateTime]::Now.AddMonths(1)

$sasToken = New-AzStorageAccountSasToken -Service blob,file,Table,Queue -ResourceType Service,Container,Object -Permission "racwdlup" -Protocol HttpsOnly -StartTime $start -ExpiryTime $end -Context $storageContext
```
El valor de $sasToken tendrá un aspecto similar al siguiente.

```console
?sv=2018-11-09&sig=5GWqHFkEOtM7W9alOgoXSCOJO%2B55qJr4J7tHQjCId9S%3D&spr=https&st=2019-09-18T18%3A25%3A00Z&se=2019-10-19T18%3A25%3A00Z&srt=sco&ss=bfqt&sp=racupwdl
```

### <a name="generate-a-shared-access-signature-definition"></a>Generación de una definición de firma de acceso compartido

Use el cmdlet [Set-AzKeyVaultManagedStorageSasDefinition](/powershell/module/az.keyvault/set-azkeyvaultmanagedstoragesasdefinition?view=azps-2.6.0) de Azure PowerShell para crear una definición de firma de acceso compartido.  Puede proporcionar el nombre que elija al parámetro `-Name`.

```azurepowershell-interactive
Set-AzKeyVaultManagedStorageSasDefinition -AccountName $storageAccountName -VaultName $keyVaultName -Name <YourSASDefinitionName> -TemplateUri $sasToken -SasType 'account' -ValidityPeriod ([System.Timespan]::FromDays(30))
```

### <a name="verify-the-shared-access-signature-definition"></a>Comprobación de la definición de firma de acceso compartido

Puede comprobar que la definición de la firma de acceso compartido se ha almacenado en el almacén de claves mediante el cmdlet [Get-AzKeyVaultSecret](/powershell/module/az.keyvault/get-azkeyvaultsecret?view=azps-2.6.0) de Azure PowerShell.

En primer lugar, busque la definición de la firma de acceso compartido en el almacén de claves.

```azurepowershell-interactive
Get-AzKeyVaultSecret -VaultName <YourKeyVaultName>
```

El secreto correspondiente a la definición de SAS tendrá estas propiedades:

```console
Vault Name   : <YourKeyVaultName>
Name         : <SecretName>
...
Content Type : application/vnd.ms-sastoken-storage
Tags         :
```

Ahora puede usar el cmdlet [Get-AzKeyVaultSecret](/cli/azure/keyvault/secret?view=azure-cli-latest#az-keyvault-secret-show) y la propiedad `Name` del secreto para ver el contenido del secreto.

```azurepowershell-interactive
$secret = Get-AzKeyVaultSecret -VaultName <YourKeyVaultName> -Name <SecretName>

Write-Host $secret.SecretValueText
```

La salida de este comando mostrará la cadena de definición de SAS.


## <a name="next-steps"></a>Pasos siguientes

- [Ejemplos de clave de cuenta de almacenamiento administrada](https://github.com/Azure-Samples?utf8=%E2%9C%93&q=key+vault+storage&type=&language=)
- [Referencia de PowerShell para Key Vault](/powershell/module/az.keyvault/?view=azps-1.2.0#key_vault)
