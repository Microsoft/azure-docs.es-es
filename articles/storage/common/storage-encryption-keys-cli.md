---
title: Uso de la CLI de Azure para configurar las claves administradas por el cliente
titleSuffix: Azure Storage
description: Aprenda a usar la CLI de Azure para configurar claves administradas por el cliente con Azure Key Vault para el cifrado de Azure Storage.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 08/24/2020
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.custom: devx-track-azurecli
ms.openlocfilehash: 25ee5d389bc70d82730c7056c752de393a6bf4c5
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88799152"
---
# <a name="configure-customer-managed-keys-with-azure-key-vault-by-using-azure-cli"></a>Configuración de claves administradas por el cliente con Azure Key Vault mediante la CLI de Azure

[!INCLUDE [storage-encryption-configure-keys-include](../../../includes/storage-encryption-configure-keys-include.md)]

En este artículo se muestra cómo configurar Azure Key Vault con claves administradas por el cliente mediante la CLI de Azure. Para aprender a crear un almacén de claves mediante la CLI de Azure, consulte [Inicio rápido: Establecimiento y recuperación de un secreto desde Azure Key Vault mediante la CLI de Azure](../../key-vault/secrets/quick-create-cli.md).

## <a name="assign-an-identity-to-the-storage-account"></a>Asignación de una identidad a la cuenta de almacenamiento

Para habilitar claves que administra el cliente para la cuenta de almacenamiento, debe asignar primero una identidad administrada que haya asignado el sistema a la cuenta de almacenamiento. Usará esta identidad administrada para conceder los permisos de la cuenta de almacenamiento para obtener acceso al almacén de claves.

Para asignar una identidad administrada mediante la CLI de Azure, llame al comando [az storage account update](/cli/azure/storage/account#az-storage-account-update). No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores.

```azurecli-interactive
az login
az account set --subscription <subscription-id>

az storage account update \
    --name <storage-account> \
    --resource-group <resource_group> \
    --assign-identity
```

Para obtener información sobre cómo configurar identidades administradas que haya asignado el sistema con la CLI de Azure, consulte [Configure managed identities for Azure resources on an Azure VM using Azure CLI](../../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md) (Configuración de identidades administradas de recursos de Azure en una VM de Azure con la CLI de Azure).

## <a name="create-a-new-key-vault"></a>Creación de un almacén de claves nuevo

El almacén de claves que se usará para almacenar las claves que administre el cliente para el cifrado de Azure Storage debe tener dos configuraciones de protección de claves habilitadas: la **eliminación temporal** y la opción de **no purgar**. Para crear un nuevo almacén de claves con PowerShell o la CLI de Azure mediante esta configuración habilitada, ejecute los siguientes comandos. No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores.

Para crear un nuevo almacén de claves con la CLI de Azure, llame al comando [az keyvault create](/cli/azure/keyvault#az-keyvault-create). No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores.

```azurecli-interactive
az keyvault create \
    --name <key-vault> \
    --resource-group <resource_group> \
    --location <region> \
    --enable-soft-delete \
    --enable-purge-protection
```

Para obtener información sobre cómo habilitar **Eliminación temporal** y **No purgar** en un almacén de claves existente con la CLI de Azure, consulte las secciones tituladas **Habilitar la eliminación temporal** y **Habilitación de la protección de purgas** del artículo sobre [uso de la eliminación temporal con la CLI](../../key-vault/general/soft-delete-cli.md).

## <a name="configure-the-key-vault-access-policy"></a>Configuración de la directiva de acceso del almacén de claves

A continuación, configure la directiva de acceso del almacén de claves, para que la cuenta de almacenamiento tenga permiso para acceder a él. En este paso, usará la identidad administrada que asignó previamente a la cuenta de almacenamiento.

Use [az keyvault set-policy](/cli/azure/keyvault#az-keyvault-set-policy) para establecer la directiva de acceso del almacén de claves. No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores.

```azurecli-interactive
storage_account_principal=$(az storage account show \
    --name <storage-account> \
    --resource-group <resource-group> \
    --query identity.principalId \
    --output tsv)
az keyvault set-policy \
    --name <key-vault> \
    --resource-group <resource_group>
    --object-id $storage_account_principal \
    --key-permissions get unwrapKey wrapKey
```

## <a name="create-a-new-key"></a>Creación de una nueva clave

A continuación, cree una clave en el almacén de claves. Para crear una clave, llame a [az keyvault key create](/cli/azure/keyvault/key#az-keyvault-key-create). No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores.

```azurecli-interactive
az keyvault key create \
    --name <key> \
    --vault-name <key-vault>
```

El cifrado de almacenamiento de Azure admite claves RSA y RSA-HSM de los tamaños 2048, 3072 y 4096. Para más información acerca de las claves, consulte la sección **Claves en Key Vault** en [Información acerca de claves, secretos y certificados de Azure Key Vault](../../key-vault/about-keys-secrets-and-certificates.md#key-vault-keys).

## <a name="configure-encryption-with-customer-managed-keys"></a>Configuración del cifrado con claves que administra el cliente

De forma predeterminada, el cifrado de Azure Storage usa claves que administra Microsoft. En este paso, configurará su cuenta de Azure Storage para usar claves administradas por el cliente con Azure Key Vault y, después, especificará la clave que se va a asociar con la cuenta de almacenamiento.

Al configurar el cifrado con claves administradas por el cliente, puede optar por actualizar automáticamente la clave que se usa para el cifrado cuando la versión de clave cambie en el almacén de claves asociado. Como alternativa, puede especificar explícitamente la versión de la clave que se usará para el cifrado hasta que la versión de la clave se actualice manualmente.

> [!NOTE]
> Para rotar una clave, cree una nueva versión de la misma en Azure Key Vault. Azure Storage no controla la rotación de la clave en Azure Key Vault, por lo que deberá rotar la clave manualmente o crear una función para hacerlo según una programación.

### <a name="configure-encryption-to-automatically-update-the-key-version"></a>Configuración del cifrado para actualizar automáticamente la versión de la clave

Para configurar el cifrado con claves administradas por el cliente para que se actualice automáticamente la versión de la clave, instale la [versión 2.4.0 de la CLI de Azure](/cli/azure/release-notes-azure-cli#april-21-2020), o cualquier versión posterior. Para más información, consulte [Instalación de la CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest).

Para actualizar automáticamente la versión de una clave administrada por el cliente, omita la versión de la clave al configurar el cifrado con las claves administradas por el cliente para la cuenta de almacenamiento. Llame a [az storage account update](/cli/azure/storage/account#az-storage-account-update) para actualizar la configuración de cifrado de la cuenta de almacenamiento, como se muestra en el ejemplo siguiente. Incluya el parámetro `--encryption-key-source` y establézcalo en `Microsoft.Keyvault` para habilitar las claves administradas por el cliente para la cuenta. No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores.

```azurecli-interactive
key_vault_uri=$(az keyvault show \
    --name <key-vault> \
    --resource-group <resource_group> \
    --query properties.vaultUri \
    --output tsv)
az storage account update
    --name <storage-account> \
    --resource-group <resource_group> \
    --encryption-key-name <key> \
    --encryption-key-source Microsoft.Keyvault \
    --encryption-key-vault $key_vault_uri
```

### <a name="configure-encryption-for-manual-updating-of-key-versions"></a>Configuración del cifrado para la actualización manual de las versiones de la clave

Para especificar de manera explícita la versión de la clave que se va a usar para el cifrado, indique la versión de la clave al configurar el cifrado con las claves administradas por el cliente para la cuenta de almacenamiento. Llame a [az storage account update](/cli/azure/storage/account#az-storage-account-update) para actualizar la configuración de cifrado de la cuenta de almacenamiento, como se muestra en el ejemplo siguiente. Incluya el parámetro `--encryption-key-source` y establézcalo en `Microsoft.Keyvault` para habilitar las claves administradas por el cliente para la cuenta. No olvide reemplazar los valores del marcador de posición entre corchetes con sus propios valores.

```azurecli-interactive
key_vault_uri=$(az keyvault show \
    --name <key-vault> \
    --resource-group <resource_group> \
    --query properties.vaultUri \
    --output tsv)
key_version=$(az keyvault key list-versions \
    --name <key> \
    --vault-name <key-vault> \
    --query [-1].kid \
    --output tsv | cut -d '/' -f 6)
az storage account update
    --name <storage-account> \
    --resource-group <resource_group> \
    --encryption-key-name <key> \
    --encryption-key-version $key_version \
    --encryption-key-source Microsoft.Keyvault \
    --encryption-key-vault $key_vault_uri
```

Al actualizar manualmente la versión de la clave, deberá actualizar la configuración de cifrado de la cuenta de almacenamiento para que use la nueva versión. En primer lugar, consulte el URI del almacén de claves mediante una llamada a [az keyvault show](/cli/azure/keyvault#az-keyvault-show) y, para la versión de la clave, llame a [az keyvault key list-versions](/cli/azure/keyvault/key#az-keyvault-key-list-versions). Luego, llame a [az storage account update](/cli/azure/storage/account#az-storage-account-update) para actualizar la configuración de cifrado de la cuenta de almacenamiento y así poder usar la nueva versión de la clave, tal como se muestra en el ejemplo anterior.

## <a name="use-a-different-key"></a>Uso de una clave distinta

Para cambiar la clave usada para el cifrado de Azure Storage, llame a [az storage account update](/cli/azure/storage/account#az-storage-account-update) tal y como se muestra en [Configuración del cifrado con claves que administra el cliente](#configure-encryption-with-customer-managed-keys) y proporcione el nombre y la versión de la nueva clave. Si la nueva clave se encuentra en otro almacén de claves, actualice también el URI del almacén de claves.

## <a name="revoke-customer-managed-keys"></a>Revocación de claves administradas por el cliente

Para revocar las claves administradas por el cliente, quite la directiva de acceso del almacén de claves. Para revocar una clave administrada por el cliente, llame al comando [az keyvault delete-policy](/cli/azure/keyvault#az-keyvault-delete-policy), como se muestra en el siguiente ejemplo. Recuerde reemplazar los valores de marcador de posición entre corchetes por sus propios valores y usar las variables definidas en los ejemplos anteriores.

```azurecli-interactive
az keyvault delete-policy \
    --name <key-vault> \
    --object-id $storage_account_principal
```

## <a name="disable-customer-managed-keys"></a>Deshabilitación de claves administradas por el cliente

Cuando las claves administradas por el cliente se deshabilitan, la cuenta de almacenamiento se vuelve a cifrar con claves administradas por Microsoft. Para deshabilitar las claves administradas por el cliente, llame [az storage account update](/cli/azure/storage/account#az-storage-account-update) y establezca el `--encryption-key-source parameter` en `Microsoft.Storage`, tal y como se muestra en el ejemplo siguiente. Recuerde reemplazar los valores de marcador de posición entre corchetes por sus propios valores y usar las variables definidas en los ejemplos anteriores.

```azurecli-interactive
az storage account update
    --name <storage-account> \
    --resource-group <resource_group> \
    --encryption-key-source Microsoft.Storage
```

## <a name="next-steps"></a>Pasos siguientes

- [Cifrado de Azure Storage para datos en reposo](storage-service-encryption.md) 
- [¿Qué es Azure Key Vault?](https://docs.microsoft.com/azure/key-vault/key-vault-overview)
