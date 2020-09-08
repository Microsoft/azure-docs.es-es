---
title: La API REST de conversión de recursos
description: Describe cómo convertir un recurso a través de la API REST
author: florianborn71
ms.author: flborn
ms.date: 02/04/2020
ms.topic: how-to
ms.openlocfilehash: 5c638b434ceb31b57689b11971f48eb322b94726
ms.sourcegitcommit: 25bb515efe62bfb8a8377293b56c3163f46122bf
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87985621"
---
# <a name="use-the-model-conversion-rest-api"></a>Uso de la API REST de conversión de modelos

El servicio de [conversión de ](model-conversion.md) se controla mediante una [API REST](https://en.wikipedia.org/wiki/Representational_state_transfer). En este artículo se describen los detalles de la API del servicio de conversión.

## <a name="regions"></a>Regions

Consulte en la [lista de regiones disponibles](../../reference/regions.md) las direcciones URL base a las que enviar las solicitudes.

## <a name="common-headers"></a>Encabezados comunes

### <a name="common-request-headers"></a>Encabezados de solicitud comunes

Estos encabezados deben especificarse para todas las solicitudes:

- El encabezado **Authorization** debe tener el valor "Bearer [*TOKEN*]", donde [*TOKEN*] es un [token de acceso de servicio](../tokens.md).

### <a name="common-response-headers"></a>Encabezados de respuesta comunes

Todas las respuestas contienen estos encabezados:

- El encabezado **MS-CV** contiene una cadena única que se puede utilizar para realizar el seguimiento de la llamada dentro del servicio.

## <a name="endpoints"></a>Puntos de conexión

El servicio de conversión proporciona tres puntos de conexión de la API REST para:

- Iniciar la conversión de modelos mediante una cuenta de almacenamiento vinculada a su cuenta de Azure Remote Rendering. 
- Iniciar la conversión de modelos mediante las *Firmas de acceso compartido (SAS)* proporcionadas.
- Consultar el estado de la conversión

### <a name="start-conversion-using-a-linked-storage-account"></a>Inicio de la conversión mediante una cuenta de almacenamiento vinculada
La cuenta de Azure Remote Rendering debe tener acceso a la cuenta de almacenamiento proporcionada; para ello, siga los pasos que se indican en [Vinculación de las cuentas de almacenamiento](../create-an-account.md#link-storage-accounts).

| Punto de conexión | Método |
|-----------|:-----------|
| /v1/accounts/**accountID**/conversions/create | POST |

Devuelve el identificador de la conversión en curso, encapsulado en un documento JSON. El nombre del campo es "conversionId".

#### <a name="request-body"></a>Cuerpo de la solicitud

> [!NOTE]
> Todo lo que hay en `input.folderPath` se recuperará para realizar la conversión en Azure. Si no se especifica `input.folderPath`, se recuperará todo el contenido del contenedor. Todos los blobs y carpetas que se recuperan deben tener [nombres de archivo de Windows válidos](https://docs.microsoft.com/windows/win32/fileio/naming-a-file#naming-conventions).

```json
{
    "input":
    {
        "storageAccountname": "<the name of a connected storage account - this does not include the domain suffix (.blob.core.windows.net)>",
        "blobContainerName": "<the name of the blob container containing your input asset data>",
        "folderPath": "<optional: can be omitted or empty - a subpath in the input blob container>",
        "inputAssetPath" : "<path to the model in the input blob container relative to the folderPath (or container root if no folderPath is specified)>"
    },
    "output":
    {
        "storageAccountname": "<the name of a connected storage account - this does not include the domain suffix (.blob.core.windows.net)>",
        "blobContainerName": "<the name of the blob container where the converted asset will be copied to>",
        "folderPath": "<optional: can be omitted or empty - a subpath in the output blob container. Will contain the asset and log files>",
        "outputAssetFileName": "<optional: can be omitted or empty. The filename of the converted asset. If provided the filename needs to end in .arrAsset>"
    }
}
```
### <a name="start-conversion-using-provided-shared-access-signatures"></a>Inicio de la conversión mediante las firmas de acceso compartido proporcionadas
Si su cuenta de ARR no está vinculada a su cuenta de almacenamiento, esta interfaz de REST le permite proporcionar acceso mediante *Firmas de acceso compartido (SAS)* .

| Punto de conexión | Método |
|-----------|:-----------|
| /v1/accounts/**accountID**/conversions/createWithSharedAccessSignature | POST |

Devuelve el identificador de la conversión en curso, encapsulado en un documento JSON. El nombre del campo es `conversionId`.

#### <a name="request-body"></a>Cuerpo de la solicitud

El cuerpo de la solicitud es el mismo que en la creación de una llamada REST anterior, pero la entrada y la salida contienen *tokens de firmas de acceso compartido (SAS)* . Estos tokens proporcionan acceso a la cuenta de almacenamiento para leer la entrada y escribir el resultado de la conversión.

> [!NOTE]
> Estos tokens de URI de SAS son las cadenas de consulta y no el URI completo. 

> [!NOTE]
> Todo lo que hay en `input.folderPath` se recuperará para realizar la conversión en Azure. Si no se especifica `input.folderPath`, se recuperará todo el contenido del contenedor. Todos los blobs y carpetas que se recuperan deben tener [nombres de archivo de Windows válidos](https://docs.microsoft.com/windows/win32/fileio/naming-a-file#naming-conventions).

```json
{
    "input":
    {
        "storageAccountname": "<the name of a connected storage account - this does not include the domain suffix (.blob.core.windows.net)>",
        "blobContainerName": "<the name of the blob container containing your input asset data>",
        "folderPath": "<optional: can be omitted or empty - a subpath in the input blob container>",
        "inputAssetPath" : "<path to the model in the input blob container relative to the folderPath (or container root if no folderPath is specified)>",
        "containerReadListSas" : "<a container SAS token which gives read and list access to the given input blob container>"
    },
    "output":
    {
        "storageAccountname": "<the name of a connected storage account - this does not include the domain suffix (.blob.core.windows.net)>",
        "blobContainerName": "<the name of the blob container where the converted asset will be copied to>",
        "folderPath": "<optional: can be omitted or empty - a subpath in the output blob container. Will contain the asset and log files>",
        "outputAssetFileName": "<optional: can be omitted or empty. The filename of the converted asset. If provided the filename needs to end in .arrAsset>",
        "containerWriteSas" : "<a container SAS token which gives write access to the given output blob container>"
    }
}
```

### <a name="poll-conversion-status"></a>Estado de conversión del sondeo
El estado de una conversión en curso iniciada con una de las llamadas REST anteriores se puede consultar mediante la interfaz siguiente:


| Punto de conexión | Método |
|-----------|:-----------|
| /v1/accounts/**accountID**/conversions/**conversionId** | GET |

Devuelve un documento JSON con un campo "status" que puede tener los valores siguientes:

- "Created"
- "Running"
- "Success"
- "Failure"

Si el estado es "Failure" (Error), habrá un campo "error" adicional con un subcampo "message" (mensaje) que contiene información de error. Se cargarán registros adicionales en el contenedor de salida.

## <a name="next-steps"></a>Pasos siguientes

- [Uso de Azure Blob Storage para la conversión de modelos](blob-storage.md)
- [Conversión de modelos](model-conversion.md)
