---
title: Personalización de un modelo de lenguaje con la API de Video Indexer
titlesuffix: Azure Media Services
description: Obtenga información sobre cómo personalizar un modelo de lenguaje con las API de Video Indexer.
services: media-services
author: anikaz
manager: johndeu
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 02/04/2020
ms.author: kumud
ms.openlocfilehash: f373afae03357ffb65eb459f806fe441e29b21b9
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87047078"
---
# <a name="customize-a-language-model-with-the-video-indexer-api"></a>Personalización de un modelo de lenguaje con la API de Video Indexer

Video Indexer permite crear modelos de lenguaje personalizados para personalizar el reconocimiento de voz mediante la carga de texto de adaptación, es decir, texto del dominio a cuyo vocabulario desea que se adapte el motor. Una vez que se entrena el modelo, se reconocerán las nuevas palabras que aparecen en el texto de adaptación.

Para información general detallada y conocer los procedimientos recomendados para modelos de lenguaje personalizados, consulte [Customize a Language model with Video Indexer](customize-language-model-overview.md) (Personalización de un modelo de lenguaje con Video Indexer).

Puede usar las API de Video Indexer para crear y editar modelos de lenguaje personalizados en su cuenta, como se describe en este tema. También puede usar el sitio web, como se describe en [Customize Language model using the Video Indexer website](customize-language-model-with-api.md) (Personalización del modelo de lenguaje mediante el sitio web de Video Indexer).

## <a name="create-a-language-model"></a>Creación de un modelo de lenguaje

La API para [crear un modelo de lenguaje personalizado](https://api-portal.videoindexer.ai/docs/services/Operations/operations/Create-Language-Model?) crea un modelo de lenguaje en la cuenta especificada. Puede cargar archivos del modelo de lenguaje en esta llamada. También, puede crear aquí el modelo de lenguaje y cargar más adelante los archivos al actualizarlo.

> [!NOTE]
> Todavía tiene que entrenar al modelo con sus archivos habilitados para que aprenda el contenido de sus archivos. En la siguiente sección están las instrucciones para el entrenamiento en un lenguaje.

Para cargar los archivos que se agregarán al modelo de lenguaje, debe cargar los archivos en el cuerpo mediante FormData y además proporcionar valores para los parámetros necesarios anteriores. Existen dos formas de realizar esta tarea:

* La clave será el nombre de archivo y el valor será el archivo txt.
* La clave será el nombre de archivo y el valor será una dirección URL al archivo txt.

### <a name="response"></a>Response

La respuesta proporciona metadatos sobre el modelo de lenguaje recién creado, junto con metadatos sobre cada uno de los archivos del modelo, siguiendo el formato de la salida JSON de este ejemplo:

```json
{
    "id": "dfae5745-6f1d-4edd-b224-42e1ab57a891",
    "name": "TestModel",
    "language": "En-US",
    "state": "None",
    "languageModelId": "00000000-0000-0000-0000-000000000000",
    "files": [
    {
        "id": "25be7c0e-b6a6-4f48-b981-497e920a0bc9",
        "name": "hellofile",
        "enable": true,
        "creator": "John Doe",
        "creationTime": "2018-04-28T11:55:34.6733333"
    },
    {
        "id": "33025f5b-2354-485e-a50c-4e6b76345ca7",
        "name": "worldfile",
        "enable": true,
        "creator": "John Doe",
        "creationTime": "2018-04-28T11:55:34.86"
    }
    ]
}

```

## <a name="train-a-language-model"></a>Entrenamiento de un modelo de lenguaje

La API para [entrenar un modelo de lenguaje](https://api-portal.videoindexer.ai/docs/services/operations/operations/Train-Language-Model?&pattern=train) entrena un modelo de lenguaje personalizado en la cuenta especificada con el contenido de los archivos que se cargaron en el modelo de lenguaje y se habilitaron ahí.

> [!NOTE]
> En primer lugar, debe crear el modelo de lenguaje y cargar sus archivos. Puede cargar los archivos al crear el modelo de lenguaje o al actualizarlo.

### <a name="response"></a>Response

La respuesta proporciona metadatos sobre el modelo de lenguaje recién entrenado, junto con metadatos sobre cada uno de los archivos del modelo, siguiendo el formato de la salida JSON de este ejemplo:

```json
{
    "id": "41464adf-e432-42b1-8e09-f52905d7e29d",
    "name": "TestModel",
    "language": "En-US",
    "state": "Waiting",
    "languageModelId": "531e5745-681d-4e1d-b124-12e5ab57a891",
    "files": [
    {
        "id": "84fcf1ac-1952-48f3-b372-18f768eedf83",
        "name": "RenamedFile",
        "enable": false,
        "creator": "John Doe",
        "creationTime": "2018-04-27T20:10:10.5233333"
    },
    {
        "id": "9ac35b4b-1381-49c4-9fe4-8234bfdd0f50",
        "name": "hellofile",
        "enable": true,
        "creator": "John Doe",
        "creationTime": "2018-04-27T20:10:10.68"
    }
    ]
}
```

El `id` devuelto es un identificador único que se usa para distinguir entre modelos de lenguaje, mientras que se usa `languageModelId` para las API para [cargar un vídeo para indexarlo](https://api-portal.videoindexer.ai/docs/services/operations/operations/Upload-video?) y [volver a indexar un vídeo](https://api-portal.videoindexer.ai/docs/services/operations/operations/Re-index-video?) (también conocido como `linguisticModelId` en las API de carga y nueva indexación de Video Indexer).

## <a name="delete-a-language-model"></a>Eliminación de un modelo de lenguaje

La API para [eliminar un modelo de lenguaje](https://api-portal.videoindexer.ai/docs/services/operations/operations/Delete-Language-Model?&pattern=delete) elimina un modelo de lenguaje personalizado de la cuenta especificada. Cualquier vídeo que haya estado usando el modelo de lenguaje eliminado mantendrá el mismo índice hasta que vuelva a indexarlo. Si vuelve a indexar el vídeo, puede asignarle un nuevo modelo de lenguaje. En caso contrario, Video Indexer usará el modelo predeterminado para volver a indexar el vídeo.

### <a name="response"></a>Response

No se devuelve ningún contenido cuando el modelo de lenguaje se elimina correctamente.

## <a name="update-a-language-model"></a>Actualización de un modelo de lenguaje

La API para [actualizar un modelo de lenguaje](https://api-portal.videoindexer.ai/docs/services/operations/operations/Update-Language-Model?&pattern=update) actualiza un modelo de persona de lenguaje personalizado en la cuenta especificada.

> [!NOTE]
> Ya debe haber creado el modelo de lenguaje. Puede usar esta llamada para habilitar o deshabilitar todos los archivos en el modelo, actualizar el nombre del modelo de lenguaje y cargar los archivos que se agregarán al modelo de lenguaje.

Para cargar los archivos que se agregarán al modelo de lenguaje, debe cargar los archivos en el cuerpo mediante FormData y además proporcionar valores para los parámetros necesarios anteriores. Existen dos formas de realizar esta tarea:

* La clave será el nombre de archivo y el valor será el archivo txt.
* La clave será el nombre de archivo y el valor será una dirección URL al archivo txt.

### <a name="response"></a>Response

La respuesta proporciona metadatos sobre el modelo de lenguaje recién entrenado, junto con metadatos sobre cada uno de los archivos del modelo, siguiendo el formato de la salida JSON de este ejemplo:

```json
{
    "id": "41464adf-e432-42b1-8e09-f52905d7e29d",
    "name": "TestModel",
    "language": "En-US",
    "state": "Waiting",
    "languageModelId": "531e5745-681d-4e1d-b124-12e5ab57a891",
    "files": [
    {
        "id": "84fcf1ac-1952-48f3-b372-18f768eedf83",
        "name": "RenamedFile",
        "enable": true,
        "creator": "John Doe",
        "creationTime": "2018-04-27T20:10:10.5233333"
    },
    {
        "id": "9ac35b4b-1381-49c4-9fe4-8234bfdd0f50",
        "name": "hellofile",
        "enable": true,
        "creator": "John Doe",
        "creationTime": "2018-04-27T20:10:10.68"
    }
    ]
}
```

Use el `id` de los archivos devueltos en la respuesta para descargar el contenido del archivo.

## <a name="update-a-file-from-a-language-model"></a>Actualización de un archivo de un modelo de lenguaje

La API para [actualizar un archivo](https://api-portal.videoindexer.ai/docs/services/operations/operations/Update-Language-Model-file?&pattern=update) le permite actualizar el nombre y el estado del parámetro `enable` de un archivo en un modelo de lenguaje personalizado de la cuenta especificada.

### <a name="response"></a>Response

La respuesta proporciona metadatos sobre el archivo que actualizó, siguiendo el formato de la salida JSON de ejemplo siguiente.

```json
{
  "id": "84fcf1ac-1952-48f3-b372-18f768eedf83",
  "name": "RenamedFile",
  "enable": false,
  "creator": "John Doe",
  "creationTime": "2018-04-27T20:10:10.5233333"
}
```

Use el `id` del archivo devuelto en la respuesta para descargar el contenido del archivo.

## <a name="get-a-specific-language-model"></a>Obtención de un modelo de lenguaje específico

La API para [obtener](https://api-portal.videoindexer.ai/docs/services/operations/operations/Get-Language-Model?&pattern=get) devuelve información sobre el modelo de lenguaje especificado en la cuenta especificada, como el lenguaje y los archivos que están en el modelo de lenguaje.

### <a name="response"></a>Response

La respuesta proporciona metadatos sobre el modelo de lenguaje especificado, junto con metadatos sobre cada uno de los archivos del modelo, siguiendo el formato de la salida JSON de este ejemplo:

```json
{
    "id": "dfae5745-6f1d-4edd-b224-42e1ab57a891",
    "name": "TestModel",
    "language": "En-US",
    "state": "None",
    "languageModelId": "00000000-0000-0000-0000-000000000000",
    "files": [
    {
        "id": "25be7c0e-b6a6-4f48-b981-497e920a0bc9",
        "name": "hellofile",
        "enable": true,
        "creator": "John Doe",
        "creationTime": "2018-04-28T11:55:34.6733333"
    },
    {
        "id": "33025f5b-2354-485e-a50c-4e6b76345ca7",
        "name": "worldfile",
        "enable": true,
        "creator": "John Doe",
        "creationTime": "2018-04-28T11:55:34.86"
    }
    ]
}
```

Use el `id` del archivo devuelto en la respuesta para descargar el contenido del archivo.

## <a name="get-all-the-language-models"></a>Obtención de todos los modelos de lenguaje

La API para [obtener todo](https://api-portal.videoindexer.ai/docs/services/operations/operations/Get-Language-Models?&pattern=get) devuelve todos los modelos de lenguaje personalizados en la cuenta especificada en una lista.

### <a name="response"></a>Response

La respuesta proporciona una lista de todos los modelos de lenguaje en su cuenta y cada uno de sus metadatos y archivos, siguiendo el formato de la salida JSON de este ejemplo:

```json
[
    {
        "id": "dfae5745-6f1d-4edd-b224-42e1ab57a891",
        "name": "TestModel",
        "language": "En-US",
        "state": "None",
        "languageModelId": "00000000-0000-0000-0000-000000000000",
        "files": [
        {
            "id": "25be7c0e-b6a6-4f48-b981-497e920a0bc9",
            "name": "hellofile",
            "enable": true,
            "creator": "John Doe",
            "creationTime": "2018-04-28T11:55:34.6733333"
        },
        {
            "id": "33025f5b-2354-485e-a50c-4e6b76345ca7",
            "name": "worldfile",
            "enable": true,
            "creator": "John Doe",
            "creationTime": "2018-04-28T11:55:34.86"
        }
        ]
    },
    {
        "id": "dfae5745-6f1d-4edd-b224-42e1ab57a892",
        "name": "AnotherTestModel",
        "language": "En-US",
        "state": "None",
        "languageModelId": "00000000-0000-0000-0000-000000000001",
        "files": []
    }
]
```

## <a name="delete-a-file-from-a-language-model"></a>Eliminación de un archivo de un modelo de lenguaje

La API para [eliminar](https://api-portal.videoindexer.ai/docs/services/operations/operations/Delete-Language-Model-File?&pattern=delete) elimina el archivo especificado del modelo de lenguaje especificado en la cuenta especificada.

### <a name="response"></a>Response

No se devuelve ningún contenido cuando el archivo se elimina correctamente del modelo de lenguaje.

## <a name="get-metadata-on-a-file-from-a-language-model"></a>Obtención de metadatos sobre un archivo de un modelo de lenguaje

La API para [obtener metadatos de un archivo](https://api-portal.videoindexer.ai/docs/services/operations/operations/Get-Language-Model-File-Data?&pattern=get%20language%20model) devuelve el contenido del archivo especificado y los metadatos sobre este archivo del modelo de lenguaje elegido en su cuenta.

### <a name="response"></a>Response

La respuesta proporciona el contenido y los metadatos del archivo en formato JSON, de forma parecida a este ejemplo:

```json
{
    "content": "hello\r\nworld",
    "id": "84fcf1ac-1952-48f3-b372-18f768eedf83",
    "name": "Hello",
    "enable": true,
    "creator": "John Doe",
    "creationTime": "2018-04-27T20:10:10.5233333"
}
```

> [!NOTE]
> El contenido de este archivo de ejemplo son las palabras "hello" y "world"en dos líneas distintas.

## <a name="download-a-file-from-a-language-model"></a>Descarga de un archivo de un modelo de lenguaje

La API para [descargar un archivo](https://api-portal.videoindexer.ai/docs/services/operations/operations/Download-Language-Model-File-Content?) descarga un archivo de texto, que incluye el contenido del archivo especificado, del modelo de lenguaje especificado en la cuenta especificada. Este archivo de texto debe coincidir con el contenido del archivo de texto que se cargó originalmente.

### <a name="response"></a>Response

La respuesta será la descarga de un archivo de texto con el contenido del archivo en formato JSON.

## <a name="next-steps"></a>Pasos siguientes

[Customize Language model using website](customize-language-model-with-website.md) (Personalización del modelo de lenguaje mediante el sitio web)
