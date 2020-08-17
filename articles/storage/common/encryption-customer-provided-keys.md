---
title: Especificación de una clave de cifrado en una solicitud a Blob Storage
titleSuffix: Azure Storage
description: Los clientes que realizan solicitudes en Azure Blob Storage tienen la opción de proporcionar una clave de cifrado por solicitud. La inclusión de la clave de cifrado en la solicitud proporciona un control detallado sobre la configuración de cifrado para las operaciones de almacenamiento de blobs.
services: storage
author: tamram
ms.service: storage
ms.date: 07/20/2020
ms.topic: conceptual
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.openlocfilehash: fec3d760450fb5cc20deed14dac03f58f23c2b4b
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87029746"
---
# <a name="provide-an-encryption-key-on-a-request-to-blob-storage"></a>Especificación de una clave de cifrado en una solicitud a Blob Storage

Los clientes que realizan solicitudes en Azure Blob Storage tienen la opción de proporcionar una clave de cifrado por solicitud. La inclusión de la clave de cifrado en la solicitud proporciona un control detallado sobre la configuración de cifrado para las operaciones de almacenamiento de blobs. Las claves proporcionadas por el cliente se pueden almacenar en Azure Key Vault o en otro almacén de claves.

## <a name="encrypting-read-and-write-operations"></a>Cifrado de operaciones de lectura y escritura

Cuando una aplicación cliente proporciona una clave de cifrado en la solicitud, Azure Storage realiza el cifrado y el descifrado de forma transparente mientras lee y escribe datos de blobs. Azure Storage escribe un hash SHA-256 de la clave de cifrado junto con el contenido del blob. El hash se utiliza para comprobar que todas las operaciones posteriores en el blob usan la misma clave de cifrado.

Azure Storage no almacena ni administra la clave de cifrado que el cliente envía con la solicitud. La clave se descarta de forma segura en cuanto se completa el proceso de cifrado o descifrado.

Cuando un cliente crea o actualiza un blob con una clave proporcionada por el cliente en la solicitud, las solicitudes de lectura y escritura posteriores para ese blob también deben proporcionar la clave. Si no se proporciona la clave en una solicitud para un blob que ya se ha cifrado con una clave proporcionada por el cliente, se produce un error en la solicitud con el código de error 409 (conflicto).

Si la aplicación cliente envía una clave de cifrado en la solicitud, y la cuenta de almacenamiento también está cifrada mediante una clave administrada por Microsoft o una clave administrada por el cliente, Azure Storage usa la clave proporcionada en la solicitud para el cifrado y el descifrado.

Para enviar la clave de cifrado como parte de la solicitud, un cliente debe establecer una conexión segura con Azure Storage mediante HTTPS.

Cada instantánea de blob puede tener su propia clave de cifrado.

## <a name="request-headers-for-specifying-customer-provided-keys"></a>Encabezados de solicitud para especificar las claves proporcionadas por el cliente

En el caso de las llamadas REST, los clientes pueden usar los siguientes encabezados para pasar de forma segura la información de clave de cifrado de una solicitud a Blob Storage:

|Encabezado de solicitud | Descripción |
|---------------|-------------|
|`x-ms-encryption-key` |Se requiere para solicitudes de lectura y escritura. Valor de clave de cifrado AES-256 con codificación Base64. |
|`x-ms-encryption-key-sha256`| Se requiere para solicitudes de lectura y escritura. SHA256 con codificación Base64 de la clave de cifrado. |
|`x-ms-encryption-algorithm` | Se requiere para solicitudes de escritura, opcional para solicitudes de lectura. Especifica el algoritmo que se va a usar al cifrar datos con la clave especificada. Debe ser AES256. |

La especificación de claves de cifrado en la solicitud es opcional. Sin embargo, si especifica uno de los encabezados enumerados anteriormente para una operación de escritura, deberá especificarlos todos.

## <a name="blob-storage-operations-supporting-customer-provided-keys"></a>Operaciones de almacenamiento de blobs que admiten claves proporcionadas por el cliente

Las siguientes operaciones de almacenamiento de blobs admiten el envío de claves de cifrado proporcionadas por el cliente en una solicitud:

- [Put Blob](/rest/api/storageservices/put-blob)
- [Put Block List](/rest/api/storageservices/put-block-list)
- [Put Block](/rest/api/storageservices/put-block)
- [Put Block from URL](/rest/api/storageservices/put-block-from-url)
- [Put Page](/rest/api/storageservices/put-page)
- [Put Page from URL](/rest/api/storageservices/put-page-from-url) (Poner página de dirección URL)
- [Append Block](/rest/api/storageservices/append-block)
- [Set Blob Properties](/rest/api/storageservices/set-blob-properties)
- [Set Blob Metadata](/rest/api/storageservices/set-blob-metadata)
- [Get Blob](/rest/api/storageservices/get-blob)
- [Get Blob Properties](/rest/api/storageservices/get-blob-properties)
- [Get Blob Metadata](/rest/api/storageservices/get-blob-metadata)
- [Snapshot Blob](/rest/api/storageservices/snapshot-blob)

## <a name="rotate-customer-provided-keys"></a>Rotación de claves proporcionadas por el cliente

Para girar una clave de cifrado que se ha usado para cifrar un blob, descargue el blob y vuelva a cargarlo con la nueva clave de cifrado.

> [!IMPORTANT]
> Azure Portal no se puede usar para leer o escribir en un contenedor o blob que esté cifrado con una clave proporcionada en la solicitud.
>
> Asegúrese de proteger la clave de cifrado que proporcione en una solicitud a Blob Storage en un almacén de claves seguro como Azure Key Vault. Si intenta realizar una operación de escritura en un contenedor o blob sin la clave de cifrado, se producirá un error en la operación y se perderá el acceso al objeto.

## <a name="next-steps"></a>Pasos siguientes

- [Especificación de una clave proporcionada por el cliente en una solicitud a Blob Storage con .NET](../blobs/storage-blob-customer-provided-key.md)
- [Cifrado de Azure Storage para datos en reposo](storage-service-encryption.md)
