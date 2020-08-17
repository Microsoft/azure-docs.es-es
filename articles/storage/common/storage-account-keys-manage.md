---
title: Administración de claves de acceso de cuentas
titleSuffix: Azure Storage
description: Obtenga información sobre cómo ver, administrar y rotar las claves de acceso de la cuenta de almacenamiento.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 04/24/2020
ms.author: tamram
ms.custom: devx-track-azurecli
ms.openlocfilehash: 8fda0384dacaf1183458e90688b1f6b63cc63a35
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87501345"
---
# <a name="manage-storage-account-access-keys"></a>Administración de las claves de acceso de la cuenta de almacenamiento

Cuando se crea una cuenta de almacenamiento, Azure genera dos claves de acceso de cuenta de almacenamiento de 512 bits. Estas claves se pueden usar para autorizar el acceso a los datos de su cuenta de almacenamiento mediante la autorización con clave compartida.

Microsoft recomienda usar Azure Key Vault para administrar las claves de acceso, así como rotar y volver a generar las claves de forma periódica. El uso de Azure Key Vault facilita la rotación de las claves sin ocasionar interrupciones en las aplicaciones. También puede rotar las claves de forma manual.

[!INCLUDE [storage-account-key-note-include](../../../includes/storage-account-key-note-include.md)]

## <a name="view-account-access-keys"></a>Visualización de claves de acceso de cuenta

Puede ver y copiar las claves de acceso a la cuenta con Azure Portal, PowerShell o la CLI de Azure. Azure Portal también proporciona una cadena de conexión para la cuenta de almacenamiento que puede copiar.

# <a name="portal"></a>[Portal](#tab/azure-portal)

Para ver y copiar la cadena de conexión o las claves de acceso de la cuenta de almacenamiento desde Azure Portal:

1. Vaya a la cuenta de almacenamiento en [Azure Portal](https://portal.azure.com).
1. En **Configuración**, seleccione **Claves de acceso**. Aparecen las claves de acceso de la cuenta, así como la cadena de conexión completa de cada clave.
1. Busque el valor de **Clave** en **key1** y haga clic en el botón **Copiar** para copiar la clave de cuenta.
1. Como alternativa, puede copiar la cadena de conexión completa. Busque el valor de **Cadena de conexión** en **key1**y haga clic en el botón **Copiar** para copiar la cadena de conexión.

    :::image type="content" source="media/storage-account-keys-manage/portal-connection-string.png" alt-text="Captura de pantalla que muestra cómo ver las claves de acceso en Azure Portal":::

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para recuperar las claves de acceso a la cuenta con PowerShell, llame al comando [Get-AzStorageAccountKey](/powershell/module/az.Storage/Get-azStorageAccountKey).

En el ejemplo siguiente se recupera la primera clave. Para recuperar la segunda clave, use `Value[1]` en lugar de `Value[0]`. No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores.

```powershell
$storageAccountKey = `
    (Get-AzStorageAccountKey `
    -ResourceGroupName <resource-group> `
    -Name <storage-account>).Value[0]
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para enumerar las claves de acceso a la cuenta con la CLI de Azure, llame al comando [az storage account keys list](/cli/azure/storage/account/keys#az-storage-account-keys-list), tal como se muestra en el ejemplo siguiente. No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores. 

```azurecli-interactive
az storage account keys list \
  --resource-group <resource-group> \
  --account-name <storage-account>
```

---

Puede utilizar cualquiera de las dos claves para acceder a Azure Storage, pero en general es recomendable usar la primera clave y reservar el uso de la segunda clave para cuando se roten las claves.

Para ver o leer las claves de acceso de una cuenta, el usuario debe ser un administrador de servicios o debe tener asignado un rol de Azure que incluya **Microsoft.Storage/storageAccounts/listkeys/action**. Algunos de los roles integrados de Azure que incluyen esta acción son **Propietario**, **Colaborador** y **Rol de servicio del operador de claves de cuentas de almacenamiento**. Para obtener más información sobre el rol de administrador de servicios, consulte [Roles de administrador de suscripciones clásico, de Azure y de Azure AD](../../role-based-access-control/rbac-and-directory-admin-roles.md). Para obtener información detallada sobre los roles integrados de RBAC para Azure Storage, consulte la sección **Almacenamiento** en [Roles integrados de Azure para RBAC de Azure](../../role-based-access-control/built-in-roles.md#storage).

## <a name="use-azure-key-vault-to-manage-your-access-keys"></a>Uso de Azure Key Vault para administrar las claves de acceso

Microsoft recomienda usar Azure Key Vault para administrar y rotar las claves de acceso. La aplicación puede acceder de forma segura a las claves en Key Vault, de modo que puede evitar almacenarlas en el código de la aplicación. Para obtener más información sobre el uso de Key Vault en la administración de claves, consulte los siguientes artículos:

- [Administración de claves de cuenta de almacenamiento con Azure Key Vault y Azure PowerShell](../../key-vault/secrets/overview-storage-keys-powershell.md)
- [Administración de claves de cuenta de almacenamiento con Azure Key Vault y la CLI de Azure](../../key-vault/secrets/overview-storage-keys.md)

## <a name="manually-rotate-access-keys"></a>Rotación manual de las claves de acceso

Microsoft recomienda rotar las claves de acceso periódicamente para ayudar a proteger la cuenta de almacenamiento. De ser posible, use Azure Key Vault para administrar las claves de acceso. Si no está usando Key Vault, deberá rotar las claves manualmente.

Para poder rotar las claves, se asignan dos claves de acceso. De este modo, se garantiza que la aplicación mantiene el acceso a Azure Storage a lo largo del proceso.

> [!WARNING]
> La regeneración de las claves de acceso puede afectar a aplicaciones o servicios de Azure que dependen de la clave de cuenta de almacenamiento. Los clientes que usan la clave de cuenta para acceder a la cuenta de almacenamiento se deben actualizar para usar la nueva clave, lo que incluye los servicios multimedia, las aplicaciones de nube, escritorio y móviles y las aplicaciones de interfaz gráfica de usuario de Azure Storage, como el [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/).

# <a name="portal"></a>[Portal](#tab/azure-portal)

Para rotar las claves de acceso a la cuenta de almacenamiento en Azure Portal:

1. Actualice las cadenas de conexión en el código de su aplicación para hacer referencia a la clave de acceso secundaria de la cuenta de almacenamiento.
1. Vaya a la cuenta de almacenamiento en [Azure Portal](https://portal.azure.com).
1. En **Configuración**, seleccione **Claves de acceso**.
1. Para regenerar la clave de acceso principal de la cuenta de almacenamiento, seleccione el botón **Regenerar** que se encuentra junto a la clave de acceso principal.
1. Actualice las cadenas de conexión en su código para hacer referencia a la nueva clave de acceso primaria.
1. Vuelva a generar la clave de acceso secundaria de la misma forma.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para rotar las claves de acceso a la cuenta de almacenamiento con PowerShell:

1. Actualice las cadenas de conexión en el código de su aplicación para hacer referencia a la clave de acceso secundaria de la cuenta de almacenamiento.
1. Llame al comando [New-AzStorageAccountKey](/powershell/module/az.storage/new-azstorageaccountkey) para regenerar la clave de acceso principal, tal como se muestra en este ejemplo:

    ```powershell
    New-AzStorageAccountKey -ResourceGroupName <resource-group> `
      -Name <storage-account> `
      -KeyName key1
    ```

1. Actualice las cadenas de conexión en su código para hacer referencia a la nueva clave de acceso primaria.
1. Vuelva a generar la clave de acceso secundaria de la misma forma. Para regenerar la clave secundaria, use `key2` como el nombre de clave en lugar de `key1`.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para rotar las claves de acceso a la cuenta de almacenamiento con la CLI de Azure:

1. Actualice las cadenas de conexión en el código de su aplicación para hacer referencia a la clave de acceso secundaria de la cuenta de almacenamiento.
1. Llame al comando [az storage account keys renew](/cli/azure/storage/account/keys#az-storage-account-keys-renew) para regenerar la clave de cuenta principal, tal como se muestra en este ejemplo:

    ```azurecli-interactive
    az storage account keys renew \
      --resource-group <resource-group> \
      --account-name <storage-account>
      --key primary
    ```

1. Actualice las cadenas de conexión en su código para hacer referencia a la nueva clave de acceso primaria.
1. Vuelva a generar la clave de acceso secundaria de la misma forma. Para regenerar la clave secundaria, use `key2` como el nombre de clave en lugar de `key1`.

---

> [!NOTE]
> Microsoft recomienda usar solo una de las claves en todas las aplicaciones al mismo tiempo. Si utiliza la Clave 1 en algunos lugares y la Clave 2 en otros, no podrá rotar las claves sin que alguna aplicación pierda el acceso.

Para rotar las claves de acceso de una cuenta, el usuario debe ser un administrador de servicios o debe tener asignado un rol de Azure que incluya **Microsoft.Storage/storageAccounts/regeneratekey/Action**. Algunos de los roles integrados de Azure que incluyen esta acción son **Propietario**, **Colaborador** y **Rol de servicio del operador de claves de cuentas de almacenamiento**. Para obtener más información sobre el rol de administrador de servicios, consulte [Roles de administrador de suscripciones clásico, de Azure y de Azure AD](../../role-based-access-control/rbac-and-directory-admin-roles.md). Para obtener información detallada sobre los roles integrados de Azure para Azure Storage, consulte la sección **Almacenamiento** en [Roles integrados de Azure para Azure RBAC](../../role-based-access-control/built-in-roles.md#storage).

## <a name="next-steps"></a>Pasos siguientes

- [Introducción a las cuentas de Azure Storage](storage-account-overview.md)
- [Cree una cuenta de almacenamiento](storage-account-create.md)
