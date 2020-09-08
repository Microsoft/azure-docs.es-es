---
title: Determinación del modelo de clave de cifrado que está en uso para la cuenta de almacenamiento
titleSuffix: Azure Storage
description: Use Azure Portal, PowerShell o la CLI de Azure para comprobar cómo se administran las claves de cifrado para la cuenta de almacenamiento. Las claves las pueden administrar tanto Microsoft (valor predeterminado) como el cliente. Las claves administradas por el cliente se deben almacenar en Azure Key Vault.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 03/13/2020
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: 11e08427067efa5e7bd33b8d08d84443444a190b
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89078272"
---
# <a name="determine-which-azure-storage-encryption-key-model-is-in-use-for-the-storage-account"></a>Determinación del modelo de clave de cifrado de Azure Storage que está en uso para la cuenta de almacenamiento

Azure Storage cifra automáticamente los datos de la cuenta de almacenamiento. El cifrado de Azure Storage ofrece dos opciones para administrar claves de cifrado en el nivel de la cuenta de almacenamiento:

- **Claves administradas por Microsoft.** De forma predeterminada, Microsoft administra las claves que se usan para cifrar su cuenta de almacenamiento.
- **Claves administradas por el cliente.** Opcionalmente, puede elegir administrar las claves de cifrado de su cuenta de almacenamiento. Las claves administradas por el cliente se deben almacenar en Azure Key Vault.

Además, puede proporcionar una clave de cifrado en el nivel de una solicitud individual para algunas operaciones de Blob Storage. Si se especifica una clave de cifrado en la solicitud, esta invalida la clave de cifrado que está activa en la cuenta de almacenamiento. Para más información, consulte [Especificación de una clave proporcionada por el cliente en una solicitud a Blob Storage](../blobs/storage-blob-customer-provided-key.md).

Para más información sobre claves de cifrado, consulte [Cifrado de Azure Storage para datos en reposo](storage-service-encryption.md).

## <a name="check-the-encryption-key-model-for-the-storage-account"></a>Comprobación del modelo de clave de cifrado de la cuenta de almacenamiento

Para determinar si una cuenta de almacenamiento usa claves administradas por Microsoft o claves administradas por el cliente para el cifrado, use uno de los métodos siguientes.

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

Para comprobar el modelo de cifrado de la cuenta de almacenamiento mediante Azure Portal, siga estos pasos:

1. En Azure Portal, vaya a la cuenta de almacenamiento.
1. Seleccione el valor de **Cifrado** y anótelo.

En la imagen siguiente se muestra una cuenta de almacenamiento cifrada con claves administradas por Microsoft:

![Ver cuenta cifrada con claves administradas por Microsoft](media/storage-encryption-key-model-get/microsoft-managed-encryption-key-setting-portal.png)

En la imagen siguiente se muestra una cuenta de almacenamiento cifrada con claves administradas por el cliente:

![Captura de pantalla que muestra el valor de la clave de cifrado en Azure Portal](media/storage-encryption-key-model-get/customer-managed-encryption-key-setting-portal.png)

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para comprobar el modelo de cifrado de la cuenta de almacenamiento mediante PowerShell, llame al comando [Get-AzStorageAccount](/powershell/module/az.storage/get-azstorageaccount) y, después, compruebe la propiedad **KeySource** de la cuenta.

```powershell
$account = Get-AzStorageAccount -ResourceGroupName <resource-group> `
    -Name <storage-account>
$account.Encryption.KeySource
```

Si el valor de la propiedad **KeySource** es `Microsoft.Storage`, la cuenta está cifrada con claves administradas por Microsoft. Sin embargo, si el valor de la propiedad **KeySource** es `Microsoft.Keyvault`, la cuenta está cifrada con claves administradas por el cliente.

# <a name="azure-cli"></a>[CLI de Azure](#tab/cli)

Para comprobar el modelo de cifrado de la cuenta de almacenamiento mediante la CLI de Azure, llame al comando [az storage account show](/cli/azure/storage/account#az-storage-account-show) y, después, compruebe la propiedad **keySource** de la cuenta.

```azurecli-interactive
key_source=$(az storage account show \
    --name <storage-account> \
    --resource-group <resource-group> \
    --query encryption.keySource \
    --output tsv)
```

Si el valor de la propiedad **keySource** es `Microsoft.Storage`, la cuenta está cifrada con claves administradas por Microsoft. Sin embargo, si el valor de la propiedad **keySource** es `Microsoft.Keyvault`, la cuenta está cifrada con claves administradas por el cliente.

---

## <a name="next-steps"></a>Pasos siguientes

- [Cifrado de Azure Storage para datos en reposo](storage-service-encryption.md)
- [Uso de claves administradas por el cliente con Azure Key Vault para administrar el cifrado de Azure Storage](encryption-customer-managed-keys.md)
