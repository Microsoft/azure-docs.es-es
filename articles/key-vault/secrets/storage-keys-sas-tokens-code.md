---
title: Captura de tokens de firma de acceso compartido en el código | Azure Key Vault
description: La característica de cuenta de almacenamiento administrado ofrece una integración sin problemas entre Azure Key Vault y una cuenta de almacenamiento de Azure.
ms.topic: tutorial
ms.service: key-vault
ms.subservice: secrets
author: msmbaldwin
ms.author: mbaldwin
manager: rkarlin
ms.date: 09/10/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 8ca89d06ea0d5e2396c820b25490b30e25c99f10
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89002936"
---
# <a name="fetch-shared-access-signature-tokens-in-code"></a>Captura de tokens de firma de acceso compartido en el código

Puede administrar su cuenta de almacenamiento con los tokens de Firma de acceso compartido (SAS) almacenados en el almacén de claves. Para obtener más información, consulte [Otorgar acceso limitado a recursos de Azure Storage con SAS](../../storage/common/storage-sas-overview.md).

En este artículo se proporcionan ejemplos de código .NET que captura un token de SAS y realiza operaciones con él. Para más información sobre cómo crear y almacenar tokens de SAS, consulte [Administración de claves de cuenta de almacenamiento con Key Vault y la CLI de Azure](overview-storage-keys.md) o [Administración de claves de cuenta de almacenamiento con Key Vault y Azure PowerShell](overview-storage-keys-powershell.md).

## <a name="code-samples"></a>Ejemplos de código

En este ejemplo, el código captura un token de SAS del almacén de claves, lo usa para crear una nueva cuenta de almacenamiento y crea un nuevo cliente de Blob service.

```cs
// The shared access signature is stored as a secret in keyvault. 
// After you get a security token, create a new SecretClient with vault credentials and the key vault URI.
// The format for the key vault URI (kvuri) is https://<YourKeyVaultName>.vault.azure.net

var kv = new SecretClient(new Uri(kvUri), new DefaultAzureCredential());

// Now retrive your storage SAS token from Key Vault using the name of the secret (secretName).

KeyVaultSecret secret = client.GetSecret(secretName);
var sasToken = secret.Value;

// Create new storage credentials using the SAS token.
StorageCredentials accountSAS = new StorageCredentials(sasToken);

// Use these credentials and your storage account name to create a Blob service client.
CloudStorageAccount accountWithSAS = new CloudStorageAccount(accountSAS, "<storage-account>", endpointSuffix: null, useHttps: true);
CloudBlobClient blobClientWithSAS = accountWithSAS.CreateCloudBlobClient();
```

Si el token de firma de acceso compartido está a punto de expirar, puede capturar el token de firma de acceso compartido desde el almacén de claves y actualizar el código.

```cs
// If your shared access signature token is about to expire,
// get the shared access signature token again from Key Vault and update it.
KeyVaultSecret secret = client.GetSecret(secretName);
var sasToken = secret.Value;
accountSAS.UpdateSASToken(sasToken);
```


## <a name="next-steps"></a>Pasos siguientes
- Aprenda a [Otorgar acceso limitado a recursos de Azure Storage con SAS](../../storage/common/storage-sas-overview.md).
- Consulte [Administración de claves de cuenta de almacenamiento con Key Vault y la CLI de Azure](overview-storage-keys.md) o [Azure PowerShell](overview-storage-keys-powershell.md).
- Consulte [Ejemplos de clave de cuenta de almacenamiento administrada](https://github.com/Azure-Samples?utf8=%E2%9C%93&q=key+vault+storage&type=&language=).
