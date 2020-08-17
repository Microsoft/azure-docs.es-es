---
title: Creación de una cuenta que admita las claves administradas por el cliente para tablas y colas
titleSuffix: Azure Storage
description: Obtenga información sobre cómo crear una cuenta de almacenamiento que admita la configuración de claves administradas por el cliente para tablas y colas. Use la CLI de Azure o una plantilla de Azure Resource Manager para crear una cuenta de almacenamiento que se base en la clave de cifrado de la cuenta para el cifrado de Azure Storage. Después, puede configurar las claves administradas por el cliente para la cuenta.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 02/05/2020
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.custom: devx-track-azurecli
ms.openlocfilehash: e6ae39fea8c3839208fc366ba0ce8d69799357d2
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88056727"
---
# <a name="create-an-account-that-supports-customer-managed-keys-for-tables-and-queues"></a>Creación de una cuenta que admita las claves administradas por el cliente para tablas y colas

Azure Storage cifra todos los datos de las cuentas de almacenamiento en reposo. De forma predeterminada, Queue Storage y Table Storage usan una clave cuyo ámbito es el servicio y que administra Microsoft. También puede optar por usar las claves administradas por el cliente para cifrar los datos de la cola o la tabla. Para usar claves administradas por el cliente con colas y tablas, primero debe crear una cuenta de almacenamiento que use una clave de cifrado cuyo ámbito sea la cuenta, en lugar del servicio. Después de crear una cuenta que use la clave de cifrado de la cuenta para los datos de la cola y la tabla, puede configurar las claves administradas por el cliente con Azure Key Vault para esa cuenta de almacenamiento.

En este artículo se describe cómo crear una cuenta de almacenamiento que se basa en una clave cuyo ámbito es la cuenta. Cuando la cuenta se crea por primera vez, Microsoft usa la clave de cuenta para cifrar los datos de la cuenta y administra la clave. Posteriormente, puede configurar las claves administradas por el cliente para la cuenta con el fin de aprovechar estas ventajas, incluida la capacidad de proporcionar sus propias claves, actualizar la versión de la clave, rotar las claves y revocar los controles de acceso.

## <a name="about-the-feature"></a>Sobre la característica

Para crear una cuenta de almacenamiento que se base en la clave de cifrado de la cuenta para Queue Storage y Table Storage, primero debe registrarse para poder usar esta característica con Azure. Debido a la capacidad limitada, tenga en cuenta que puede tardar varios meses antes de que se aprueben las solicitudes de acceso.

Puede crear una cuenta de almacenamiento que se base en la clave de cifrado de la cuenta para Queue Storage y Table Storage en las siguientes regiones:

- Este de EE. UU.
- Centro-sur de EE. UU.
- Oeste de EE. UU. 2  

### <a name="register-to-use-the-account-encryption-key"></a>Registro para usar la clave de cifrado de la cuenta

Para registrarse para usar la clave de cifrado de la cuenta con Queue Storage o Table Storage, use PowerShell o la CLI de Azure.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para registrarse con PowerShell, llame al comando [Register-AzProviderFeature](/powershell/module/az.resources/register-azproviderfeature).

```powershell
Register-AzProviderFeature -ProviderNamespace Microsoft.Storage `
    -FeatureName AllowAccountEncryptionKeyForQueues
Register-AzProviderFeature -ProviderNamespace Microsoft.Storage `
    -FeatureName AllowAccountEncryptionKeyForTables
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para registrarse con la CLI de Azure, llame al comando [az feature register](/cli/azure/feature#az-feature-register).

```azurecli
az feature register --namespace Microsoft.Storage \
    --name AllowAccountEncryptionKeyForQueues
az feature register --namespace Microsoft.Storage \
    --name AllowAccountEncryptionKeyForTables
```

# <a name="template"></a>[Plantilla](#tab/template)

N/D

---

### <a name="check-the-status-of-your-registration"></a>Comprobación del estado del registro

Para comprobar el estado del registro de Queue Storage o Table Storage, use PowerShell o la CLI de Azure.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para comprobar el estado del registro con PowerShell, llame al comando [Get-AzProviderFeature](/powershell/module/az.resources/get-azproviderfeature).

```powershell
Get-AzProviderFeature -ProviderNamespace Microsoft.Storage `
    -FeatureName AllowAccountEncryptionKeyForQueues
Get-AzProviderFeature -ProviderNamespace Microsoft.Storage `
    -FeatureName AllowAccountEncryptionKeyForTables
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para comprobar el estado del registro con la CLI de Azure, llame al comando [az feature](/cli/azure/feature#az-feature-show).

```azurecli
az feature show --namespace Microsoft.Storage \
    --name AllowAccountEncryptionKeyForQueues
az feature show --namespace Microsoft.Storage \
    --name AllowAccountEncryptionKeyForTables
```

# <a name="template"></a>[Plantilla](#tab/template)

N/D

---

### <a name="re-register-the-azure-storage-resource-provider"></a>Repetición del registro del proveedor de recursos de Azure Storage

Una vez aprobado el registro, debe volver a registrar el proveedor de recursos de Azure Storage. Use PowerShell o la CLI de Azure para volver a registrar el proveedor de recursos.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para volver a registrar el proveedor de recursos con PowerShell, llame al comando [Register-AzResourceProvider](/powershell/module/az.resources/register-azresourceprovider).

```powershell
Register-AzResourceProvider -ProviderNamespace 'Microsoft.Storage'
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para volver a registrar el proveedor de recursos con la CLI de Azure, llame al comando [az provider register](/cli/azure/provider#az-provider-register).

```azurecli
az provider register --namespace 'Microsoft.Storage'
```

# <a name="template"></a>[Plantilla](#tab/template)

N/D

---

## <a name="create-an-account-that-uses-the-account-encryption-key"></a>Creación de una cuenta que use la clave de cifrado de la cuenta

Debe configurar una nueva cuenta de almacenamiento para usar la clave de cifrado de la cuenta para las colas y tablas en el momento de crear la cuenta de almacenamiento. El ámbito de la clave de cifrado no se puede cambiar una vez creada la cuenta.

La cuenta de almacenamiento debe ser de tipo de uso general v2. Puede crear la cuenta de almacenamiento y configurarla para que se base en la clave de cifrado de la cuenta mediante la CLI de Azure o una plantilla de Azure Resource Manager.

> [!NOTE]
> Solo puede configurar Queue Storage y Table Storage para cifrar datos con la clave de cifrado de la cuenta cuando se crea la cuenta de almacenamiento. Blob Storage y Azure Files siempre usan la clave de cifrado de la cuenta para cifrar los datos.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para usar PowerShell para crear una cuenta de almacenamiento que utilice la clave de cifrado de la cuenta, asegúrese de haber instalado la versión 3.4.0 del módulo de Azure PowerShell, o cualquier versión posterior. Para más información, consulte el artículo en el que se indica cómo [instalar el módulo de Azure PowerShell](/powershell/azure/install-az-ps).

A continuación, cree una cuenta de almacenamiento de uso general v2. Para ello, debe llamar al comando [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount), con los parámetros adecuados:

- Incluya la opción `-EncryptionKeyTypeForQueue` y establezca su valor en `Account` a fin de usar la clave de cifrado de la cuenta para cifrar los datos en Queue Storage.
- Incluya la opción `-EncryptionKeyTypeForTable` y establezca su valor en `Account` a fin de usar la clave de cifrado de la cuenta para cifrar los datos en Table Storage.

En el ejemplo siguiente se muestra cómo crear una cuenta de almacenamiento de uso general v2 configurada para el almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS) y que usa la clave de cifrado de la cuenta para cifrar los datos tanto de Queue Storage como de Table Storage. No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores:

```powershell
New-AzStorageAccount -ResourceGroupName <resource_group> `
    -AccountName <storage-account> `
    -Location <location> `
    -SkuName "Standard_RAGRS" `
    -Kind StorageV2 `
    -EncryptionKeyTypeForTable Account `
    -EncryptionKeyTypeForQueue Account
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para usar la CLI de Azure para crear una cuenta de almacenamiento que se base en la clave de cifrado de la cuenta, asegúrese de haber instalado la CLI de Azure de la versión 2.0.80 o posterior. Para más información, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

A continuación, cree una cuenta de almacenamiento de uso general v2. Para ello, llame al comando [az storage account create](/cli/azure/storage/account#az-storage-account-create), con los parámetros adecuados:

- Incluya la opción `--encryption-key-type-for-queue` y establezca su valor en `Account` a fin de usar la clave de cifrado de la cuenta para cifrar los datos en Queue Storage.
- Incluya la opción `--encryption-key-type-for-table` y establezca su valor en `Account` a fin de usar la clave de cifrado de la cuenta para cifrar los datos en Table Storage.

En el ejemplo siguiente se muestra cómo crear una cuenta de almacenamiento de uso general v2 configurada para el almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS) y que usa la clave de cifrado de la cuenta para cifrar los datos tanto de Queue Storage como de Table Storage. No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores:

```azurecli
az storage account create \
    --name <storage-account> \
    --resource-group <resource-group> \
    --location <location> \
    --sku Standard_RAGRS \
    --kind StorageV2 \
    --encryption-key-type-for-table Account \
    --encryption-key-type-for-queue Account
```

# <a name="template"></a>[Plantilla](#tab/template)

En el ejemplo de JSON siguiente se crea una cuenta de almacenamiento de uso general v2 configurada para el almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS) y que usa la clave de cifrado de la cuenta para cifrar los datos tanto de Queue Storage como de Table Storage. No olvide reemplazar los valores del marcador de posición entre corchetes angulares por sus propios valores:

```json
"resources": [
    {
        "type": "Microsoft.Storage/storageAccounts",
        "apiVersion": "2019-06-01",
        "name": "[parameters('<storage-account>')]",
        "location": "[parameters('<location>')]",
        "dependsOn": [],
        "tags": {},
        "sku": {
            "name": "[parameters('Standard_RAGRS')]"
        },
        "kind": "[parameters('StorageV2')]",
        "properties": {
            "accessTier": "[parameters('<accessTier>')]",
            "supportsHttpsTrafficOnly": "[parameters('supportsHttpsTrafficOnly')]",
            "largeFileSharesState": "[parameters('<largeFileSharesState>')]",
            "encryption": {
                "services": {
                    "queue": {
                        "keyType": "Account"
                    },
                    "table": {
                        "keyType": "Account"
                    }
                },
                "keySource": "Microsoft.Storage"
            }
        }
    }
],
```

---

Después de crear una cuenta que se base en la clave de cifrado de la cuenta, consulte uno de los siguientes artículos para configurar las claves administradas por el cliente con Azure Key Vault:

- [Configuración de claves administradas por el cliente con Azure Key Vault mediante Azure Portal](storage-encryption-keys-portal.md)
- [Configuración de claves administradas por el cliente con Azure Key Vault mediante PowerShell](storage-encryption-keys-powershell.md)
- [Configuración de claves administradas por el cliente con Azure Key Vault mediante la CLI de Azure](storage-encryption-keys-cli.md)

## <a name="verify-the-account-encryption-key"></a>Verificación de la clave de cifrado de la cuenta

Para verificar que un servicio de una cuenta de almacenamiento usa la clave de cifrado de la cuenta, llame al comando [az storage account](/cli/azure/storage/account#az-storage-account-show) de la CLI de Azure. Este comando devuelve un conjunto de propiedades de la cuenta de almacenamiento y sus valores. Busque el campo `keyType` de cada servicio dentro de la propiedad de cifrado y verifique que esté establecido en `Account`.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para comprobar que un servicio de una cuenta de almacenamiento usa la clave de cifrado de la cuenta, llame al comando [Get-AzStorageAccount](/powershell/module/az.storage/get-azstorageaccount). Este comando devuelve un conjunto de propiedades de la cuenta de almacenamiento y sus valores. Busque el campo `KeyType` en cada servicio dentro de la propiedad `Encryption` y compruebe que esté establecida en `Account`.

```powershell
$account = Get-AzStorageAccount -ResourceGroupName <resource-group> `
    -StorageAccountName <storage-account>
$account.Encryption.Services.Queue
$account.Encryption.Services.Table
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para comprobar que un servicio de una cuenta de almacenamiento usa la clave de cifrado de la cuenta, llame al comando [az storage account show](/cli/azure/storage/account#az-storage-account-show). Este comando devuelve un conjunto de propiedades de la cuenta de almacenamiento y sus valores. Busque el campo `keyType` de cada servicio dentro de la propiedad de cifrado y verifique que esté establecido en `Account`.

```azurecli
az storage account show /
    --name <storage-account> /
    --resource-group <resource-group>
```

# <a name="template"></a>[Plantilla](#tab/template)

N/D

---

## <a name="next-steps"></a>Pasos siguientes

- [Cifrado de Azure Storage para datos en reposo](storage-service-encryption.md) 
- [¿Qué es Azure Key Vault?](https://docs.microsoft.com/azure/key-vault/key-vault-overview)
