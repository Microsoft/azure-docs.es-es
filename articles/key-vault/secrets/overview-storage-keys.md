---
title: Administración de claves de cuenta de almacenamiento con Azure Key Vault y la CLI de Azure
description: Las claves de cuenta de almacenamiento proporcionan integración sin problemas entre Azure Key Vault y el acceso basado en claves a una cuenta de almacenamiento de Azure.
ms.topic: conceptual
services: key-vault
ms.service: key-vault
ms.subservice: secrets
author: msmbaldwin
ms.author: mbaldwin
manager: rkarlin
ms.date: 09/18/2019
ms.custom: devx-track-azurecli
ms.openlocfilehash: 727a5052b0531cc0a37cc631e11bc498498be5b3
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87534981"
---
# <a name="manage-storage-account-keys-with-key-vault-and-the-azure-cli"></a>Administración de claves de cuenta de almacenamiento con Key Vault y la CLI de Azure

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

- [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).
- [Creación de un Almacén de claves](quick-create-cli.md)
- [Creación de una cuenta de almacenamiento de Azure](../../storage/common/storage-account-create.md?tabs=azure-cli). El nombre de la cuenta de almacenamiento solo debe contener letras minúsculas y números. El nombre debe tener entre 3 y 24 caracteres.
      
## <a name="manage-storage-account-keys"></a>Administración de las claves de cuenta de almacenamiento

### <a name="connect-to-your-azure-account"></a>Conexión a la cuenta de Azure

Autentique la sesión de la CLI de Azure mediante los comandos [az login](/powershell/module/az.accounts/connect-azaccount?view=azps-2.5.0).

```azurecli-interactive
az login
``` 

### <a name="give-key-vault-access-to-your-storage-account"></a>Proporcionar a Key Vault acceso a la cuenta de almacenamiento

Use el comando de la CLI de Azure [az role assignment create](/cli/azure/role/assignment?view=azure-cli-latest) para proporcionar acceso a Key Vault a la cuenta de almacenamiento. Proporcione al comando los siguientes valores de parámetro:

- `--role`: Pase el rol de Azure "Rol de servicio del operador de claves de cuentas de almacenamiento". Este rol limita el ámbito de acceso a la cuenta de almacenamiento. Si se trata de una cuenta de almacenamiento clásica, pase el "Rol de servicio de operador de claves de cuentas de almacenamiento clásicas".
- `--assignee`: Pase el valor "https://vault.azure.net", que es la URL de Key Vault en la nube pública de Azure. (Para la nube de Azure Government, use "--asingee-object-id" en su lugar; consulte [Id. de la aplicación de la entidad de servicio](#service-principal-application-id)).
- `--scope`: Pase el identificador del recurso de la cuenta de almacenamiento, que tiene el formato `/subscriptions/<subscriptionID>/resourceGroups/<StorageAccountResourceGroupName>/providers/Microsoft.Storage/storageAccounts/<YourStorageAccountName>`. Para buscar el identificador de suscripción, use el comando de la CLI de Azure [az account list](/cli/azure/account?view=azure-cli-latest#az-account-list). Para buscar el nombre de la cuenta de almacenamiento y el grupo de recursos de la cuenta de almacenamiento, use el comando de la CLI de Azure [az storage account list](/cli/azure/storage/account?view=azure-cli-latest#az-storage-account-list).

```azurecli-interactive
az role assignment create --role "Storage Account Key Operator Service Role" --assignee 'https://vault.azure.net' --scope "/subscriptions/<subscriptionID>/resourceGroups/<StorageAccountResourceGroupName>/providers/Microsoft.Storage/storageAccounts/<YourStorageAccountName>"
 ```
### <a name="give-your-user-account-permission-to-managed-storage-accounts"></a>Concesión del permiso de cuenta de usuario a las cuentas de almacenamiento administradas

Use el cmdlet [az keyvault-set-policy](/cli/azure/keyvault?view=azure-cli-latest#az-keyvault-set-policy) de la CLI de Azure para actualizar la directiva de acceso de Key Vault y conceder permisos de cuenta de almacenamiento a su cuenta de usuario.

```azurecli-interactive
# Give your user principal access to all storage account permissions, on your Key Vault instance

az keyvault set-policy --name <YourKeyVaultName> --upn user@domain.com --storage-permissions get list delete set update regeneratekey getsas listsas deletesas setsas recover backup restore purge
```

Tenga en cuenta que los permisos para las cuentas de almacenamiento no están disponibles en la página "Directivas de acceso" de la cuenta de almacenamiento en Azure Portal.
### <a name="create-a-key-vault-managed-storage-account"></a>Crear una cuenta de almacenamiento administrada de Key Vault

 Cree una cuenta de almacenamiento administrada de Key Vault mediante el comando de la CLI de Azure [az keyvault storage](/cli/azure/keyvault/storage?view=azure-cli-latest#az-keyvault-storage-add). Establezca un período de regeneración de 90 días. Cuando sea el momento de girar, KeyVault regenera la clave que no está activa y, a continuación, establece la clave recién creada como activa. Solo una de las claves se usa para emitir tokens de SAS en cualquier momento, es decir, la clave activa. Proporcione al comando los siguientes valores de parámetro:

- `--vault-name`: Pase el nombre del almacén de claves. Para buscar el nombre del almacén de claves, use el comando de la CLI de Azure [az keyvault list](/cli/azure/keyvault?view=azure-cli-latest#az-keyvault-list).
- `-n`: Pase el nombre de la cuenta de almacenamiento. Para buscar el nombre de la cuenta de almacenamiento, use el comando de la CLI de Azure [az storage account list](/cli/azure/storage/account?view=azure-cli-latest#az-storage-account-list).
- `--resource-id`: Pase el identificador del recurso de la cuenta de almacenamiento, que tiene el formato `/subscriptions/<subscriptionID>/resourceGroups/<StorageAccountResourceGroupName>/providers/Microsoft.Storage/storageAccounts/<YourStorageAccountName>`. Para buscar el identificador de suscripción, use el comando de la CLI de Azure [az account list](/cli/azure/account?view=azure-cli-latest#az-account-list). Para buscar el nombre de la cuenta de almacenamiento y el grupo de recursos de la cuenta de almacenamiento, use el comando de la CLI de Azure [az storage account list](/cli/azure/storage/account?view=azure-cli-latest#az-storage-account-list).
   
 ```azurecli-interactive
az keyvault storage add --vault-name <YourKeyVaultName> -n <YourStorageAccountName> --active-key-name key1 --auto-regenerate-key --regeneration-period P90D --resource-id "/subscriptions/<subscriptionID>/resourceGroups/<StorageAccountResourceGroupName>/providers/Microsoft.Storage/storageAccounts/<YourStorageAccountName>"
 ```

## <a name="shared-access-signature-tokens"></a>Tokens de firma de acceso compartido

También puede pedir a Key Vault que genere tokens de firma de acceso compartido. Una firma de acceso compartido ofrece acceso delegado a recursos en la cuenta de almacenamiento. Puede conceder a los clientes acceso a los recursos de su cuenta de almacenamiento sin compartir las claves de la cuenta. Una firma de acceso compartido le proporciona una forma segura de compartir los recursos de almacenamiento sin poner en peligro las claves de cuenta.

Los comandos de esta sección completan las acciones siguientes:

- Establecer una definición de firma de acceso compartido de cuenta `<YourSASDefinitionName>`. La definición se establece en una cuenta de almacenamiento administrada de Key Vault `<YourStorageAccountName>` en el almacén de claves `<YourKeyVaultName>`.
- Crear un token de firma de acceso compartido de cuenta para los servicios Blob, File, Table y Queue. El token se crea para los tipos de recurso Servicio, Contenedor y Objeto. El token se crea con todos los permisos, a través de https, y con las fechas de inicio y finalización especificadas.
- Establecer una definición de firma de acceso compartido de almacenamiento administrado de Key Vault en el almacén. La definición tiene el URI de plantilla del token de firma de acceso compartido que se creó. La definición tiene el tipo de firma de acceso compartido `account` y es válido durante N días.
- Compruebe que la firma de acceso compartido se haya guardado en el almacén de claves como secreto.

### <a name="create-a-shared-access-signature-token"></a>Creación de un token de firma de acceso compartido

Cree una definición de firma de acceso compartido mediante el comando de la CLI de Azure [az storage account generate-sas](/cli/azure/storage/account?view=azure-cli-latest#az-storage-account-generate-sas). Esta operación requiere los permisos `storage` y `setsas`.


```azurecli-interactive
az storage account generate-sas --expiry 2020-01-01 --permissions rw --resource-types sco --services bfqt --https-only --account-name <YourStorageAccountName> --account-key 00000000
```
Después de que la operación se ejecute correctamente, copie la salida.

```console
"se=2020-01-01&sp=***"
```

Esta salida se pasará al parámetro `--template-uri` en el paso siguiente.

### <a name="generate-a-shared-access-signature-definition"></a>Generación de una definición de firma de acceso compartido

Use el comando de la CLI de Azure [az keyvault storage sas-definition create](/cli/azure/keyvault/storage/sas-definition?view=azure-cli-latest#az-keyvault-storage-sas-definition-create) para pasar el resultado del paso anterior al parámetro `--template-uri` para crear una definición de firma de acceso compartido.  Puede proporcionar el nombre que elija al parámetro `-n`.

```azurecli-interactive
az keyvault storage sas-definition create --vault-name <YourKeyVaultName> --account-name <YourStorageAccountName> -n <YourSASDefinitionName> --validity-period P2D --sas-type account --template-uri <OutputOfSasTokenCreationStep>
```

### <a name="verify-the-shared-access-signature-definition"></a>Comprobación de la definición de firma de acceso compartido

Puede comprobar que la definición de la firma de acceso compartido se ha almacenado en el almacén de claves mediante los comandos de la CLI de Azure [az keyvault secret list](/cli/azure/keyvault/secret?view=azure-cli-latest#az-keyvault-secret-list) y [az keyvault secret show](/cli/azure/keyvault/secret?view=azure-cli-latest#az-keyvault-secret-show).

En primer lugar, busque la definición de la firma de acceso compartido en el almacén de claves mediante el comando [az keyvault secret list](/cli/azure/keyvault/secret?view=azure-cli-latest#az-keyvault-secret-list).

```azurecli-interactive
az keyvault secret list --vault-name <YourKeyVaultName>
```

El secreto correspondiente a la definición de SAS tendrá estas propiedades:

```console
    "contentType": "application/vnd.ms-sastoken-storage",
    "id": "https://<YourKeyVaultName>.vault.azure.net/secrets/<YourStorageAccountName>-<YourSASDefinitionName>",
```

Ahora puede usar el comando [az keyvault secret show](/cli/azure/keyvault/secret?view=azure-cli-latest#az-keyvault-secret-show) y la propiedad `id` para ver el contenido del secreto.

```azurecli-interactive
az keyvault secret show --vault-name <YourKeyVaultName> --id <SasDefinitionID>
```

La salida de este comando mostrará la cadena de definición de SAS como `value`.


## <a name="next-steps"></a>Pasos siguientes

- Más información sobre [claves, secretos y certificados](https://docs.microsoft.com/rest/api/keyvault/).
- Revise los artículos del [blog del equipo de Azure Key Vault](https://blogs.technet.microsoft.com/kv/).
- Consulte la documentación de referencia de [az keyvault storage](https://docs.microsoft.com/cli/azure/keyvault/storage?view=azure-cli-latest).
