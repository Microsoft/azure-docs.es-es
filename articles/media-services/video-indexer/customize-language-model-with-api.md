---
title: 'Uso de las API de Video Indexer para personalizar un modelo de lenguaje: Azure'
titlesuffix: Azure Media Services
description: En este artículo se muestra cómo personalizar un modelo de lenguaje con las API de Video Indexer.
services: media-services
author: anikaz
manager: johndeu
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 05/15/2019
ms.author: anzaman
ms.openlocfilehash: 4ef5354a94ae707df8dd1f2767efe04dfbacd7ad
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65799588"
---
# <a name="customize-a-language-model-with-the-video-indexer-apis"></a>Personalización de un modelo de lenguaje con las API de Video Indexer

Video Indexer permite crear modelos de lenguaje personalizados para personalizar el reconocimiento de voz mediante la carga de texto de adaptación, es decir, texto del dominio a cuyo vocabulario desea que se adapte el motor. Una vez que se entrena el modelo, se reconocerán las nuevas palabras que aparecen en el texto de adaptación. 

Para información general detallada y conocer los procedimientos recomendados para modelos de lenguaje personalizados, consulte [Customize a Language model with Video Indexer](customize-language-model-overview.md) (Personalización de un modelo de lenguaje con Video Indexer).

Puede usar las API de Video Indexer para crear y editar modelos de lenguaje personalizados en su cuenta, como se describe en este tema. También puede usar el sitio web, como se describe en [Customize Language model using the Video Indexer website](customize-language-model-with-api.md) (Personalización del modelo de lenguaje mediante el sitio web de Video Indexer).

## <a name="create-a-language-model"></a>Creación de un modelo de lenguaje

El siguiente comando crea un modelo de lenguaje personalizado en la cuenta especificada. Puede cargar archivos del modelo de lenguaje en esta llamada. También, puede crear aquí el modelo de lenguaje y cargar más adelante los archivos al actualizarlo.

> [!NOTE]
> Todavía tiene que entrenar al modelo con sus archivos habilitados para que aprenda el contenido de sus archivos. En la siguiente sección están las instrucciones para el entrenamiento en un lenguaje.

### <a name="request-url"></a>URL de la solicitud

Esta es una solicitud POST.

```
https://api.videoindexer.ai/{location}/Accounts/{accountId}/Customization/PersonModels?name={name}&accessToken={accessToken}
```

Debajo está la solicitud en Curl.

```curl
curl -v -X POST "https://api.videoindexer.ai/{location}/Accounts/{accountId}/Customization/Language?accessToken={accessToken}&modelName={modelName}&language={language}"

--data-ascii "{body}" 
```

[Consulte los parámetros necesarios y haga la prueba mediante el portal para desarrolladores de Video Indexer](https://api-portal.videoindexer.ai/docs/services/operations/operations/Create-Person-Model?).

### <a name="request-parameters"></a>Parámetros de solicitud

|**Nombre**|**Tipo**|**Obligatorio**|**Descripción**|
|---|---|---|---|
|ubicación|string|Sí|La región de Azure a la que se debe enrutar la llamada. Para más información, consulte [Regiones de Azure y Video Indexer](regions.md).|
|accountId|string|Sí|Identificador único global de la cuenta.|
|accessToken|string|Sí|Token de acceso (debe ser de ámbito [Token de acceso de cuenta](https://api-portal.videoindexer.ai/docs/services/authorization/operations/Get-Account-Access-Token?)) para autenticarse en la llamada. Los tokens de acceso expiran en 1 hora.|
|modelName|string|Sí|El nombre del modelo de lenguaje.|
|language|string|Sí|El idioma del modelo de lenguaje. <br/>Se debe proporcionar al parámetro **language** el idioma en formato BCP-47 de "etiqueta idioma-región" (p. ej.: "es-es"). Idiomas admitidos son inglés (en-US), alemán (de-DE), español (es-SP), árabe (ar-EG), francés (fr-FR), hindú (hi-HI), italiano (it-IT), japonés (ja-JP), portugués (pt-BR), ruso (ru-RU) y chino (zh-CN).  |

### <a name="request-body"></a>Cuerpo de la solicitud

Para cargar los archivos que se agregarán al modelo de lenguaje, debe cargar los archivos en el cuerpo mediante datos de formulario y además proporcionar valores para los parámetros necesarios anteriores. Existen dos formas de hacerlo: 

1. La clave será el nombre de archivo y el valor será el archivo .txt.
2. La clave será el nombre de archivo y el valor será una dirección URL al archivo .txt.

### <a name="response"></a>Respuesta

La respuesta proporciona metadatos sobre el modelo de lenguaje recién creado, junto con metadatos sobre cada uno de los archivos del modelo, siguiendo el formato de la salida JSON de ejemplo.

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

El siguiente comando entrena un modelo de lenguaje personalizado en la cuenta especificada con el contenido de los archivos que se cargaron en el modelo de lenguaje y se habilitaron ahí. 

> [!NOTE]
> En primer lugar, debe crear el modelo de lenguaje y cargar sus archivos. Puede cargar los archivos al crear el modelo de lenguaje o al actualizarlo. 

### <a name="request-url"></a>URL de la solicitud

Es una solicitud PUT.

```
https://api.videoindexer.ai/{location}/Accounts/{accountId}/Customization/Language/{modelId}/Train?accessToken={accessToken}
```

Debajo está la solicitud en Curl.

```curl
curl -v -X PUT "https://api.videoindexer.ai/{location}/Accounts/{accountId}/Customization/Language/{modelId}/Train?accessToken={accessToken}"
```
 
[Consulte los parámetros necesarios y haga la prueba mediante el portal para desarrolladores de Video Indexer](https://api-portal.videoindexer.ai/docs/services/operations/operations/Train-Language-Model?&pattern=train).

### <a name="request-parameters"></a>Parámetros de solicitud

|**Nombre**|**Tipo**|**Obligatorio**|**Descripción**|
|---|---|---|---|
|ubicación|string|Sí|La región de Azure a la que se debe enrutar la llamada. Para más información, consulte [Regiones de Azure y Video Indexer](regions.md).|
|accountID|string|Sí|Identificador único global de la cuenta.|
|modelId|string|Sí|El identificador del modelo de lenguaje (se genera cuando se crea el modelo de lenguaje).|
|accessToken|string|Sí|Token de acceso (debe ser de ámbito [Token de acceso de cuenta](https://api-portal.videoindexer.ai/docs/services/authorization/operations/Get-Account-Access-Token?)) para autenticarse en la llamada. Los tokens de acceso expiran en 1 hora.|

### <a name="request-body"></a>Cuerpo de la solicitud

No hay ningún otro cuerpo de solicitud necesario para esta llamada.

### <a name="response"></a>Respuesta

La respuesta proporciona metadatos sobre el modelo de lenguaje recién entrenado, junto con metadatos sobre cada uno de los archivos del modelo, siguiendo el formato de la salida JSON de ejemplo.

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

Debe usar el valor **id** del modelo de lenguaje para el parámetro **linguisticModelId** al [cargar un vídeo para indexar](https://api-portal.videoindexer.ai/docs/services/operations/operations/Upload-video?) y para el parámetro **languageModelId** al [volver a indexar un vídeo](https://api-portal.videoindexer.ai/docs/services/operations/operations/Re-index-video?).

## <a name="delete-a-language-model"></a>Eliminación de un modelo de lenguaje

El comando siguiente elimina un modelo de lenguaje personalizado de la cuenta especificada. Cualquier vídeo que estuviera usando el modelo de lenguaje eliminado mantendrá el mismo índice hasta que vuelva a indexar el vídeo. Si vuelve a indexar el vídeo, puede asignarle un nuevo modelo de lenguaje. En caso contrario, Video Indexer usará su modelo predeterminado para volver a indexar el vídeo.

### <a name="request-url"></a>URL de la solicitud

Esta es una solicitud de eliminación.

```
https://api.videoindexer.ai/{location}/Accounts/{accountId}/Customization/Language/{modelId}?accessToken={accessToken}
```

Debajo está la solicitud en Curl.

```curl
curl -v -X DELETE "https://api.videoindexer.ai/{location}/Accounts/{accountId}/Customization/Language/{modelId}?accessToken={accessToken}"
```
 
[Consulte los parámetros necesarios y haga la prueba mediante el portal para desarrolladores de Video Indexer](https://api-portal.videoindexer.ai/docs/services/operations/operations/Delete-Language-Model?&pattern=delete).

### <a name="request-parameters"></a>Parámetros de solicitud 

|**Nombre**|**Tipo**|**Obligatorio**|**Descripción**|
|---|---|---|---|
|ubicación|string|Sí|La región de Azure a la que se debe enrutar la llamada. Para más información, consulte [Regiones de Azure y Video Indexer](regions.md).|
|accountID|string|Sí|Identificador único global de la cuenta.|
|modelId|string|Sí|El identificador del modelo de lenguaje (se genera cuando se crea el modelo de lenguaje).|
|accessToken|string|Sí|Token de acceso (debe ser de ámbito [Token de acceso de cuenta](https://api-portal.videoindexer.ai/docs/services/authorization/operations/Get-Account-Access-Token?)) para autenticarse en la llamada. Los tokens de acceso expiran en 1 hora.|

### <a name="request-body"></a>Cuerpo de la solicitud

No hay ningún otro cuerpo de solicitud necesario para esta llamada.

### <a name="response"></a>Respuesta

No se devuelve ningún contenido cuando el modelo de lenguaje se elimina correctamente.

## <a name="update-a-language-model"></a>Actualización de un modelo de lenguaje

El comando siguiente actualiza un modelo de lenguaje personalizado en la cuenta especificada.

> [!NOTE]
> Ya debe haber creado el modelo de lenguaje. Puede usar esta llamada para habilitar o deshabilitar todos los archivos en el modelo, actualizar el nombre del modelo de lenguaje y cargar los archivos que se agregarán al modelo de lenguaje.

### <a name="request-url"></a>URL de la solicitud

Es una solicitud PUT.

```
https://api.videoindexer.ai/{location}/Accounts/{accountId}/Customization/Language/{modelId}?accessToken={accessToken}[&modelName][&enable]
```

Debajo está la solicitud en Curl.

```curl
curl -v -X PUT "https://api.videoindexer.ai/{location}/Accounts/{accountId}/Customization/Language/{modelId}?accessToken={accessToken}?modelName={string}&enable={string}"

--data-ascii "{body}" 
```
 
[Consulte los parámetros necesarios y haga la prueba mediante el portal para desarrolladores de Video Indexer](https://api-portal.videoindexer.ai/docs/services/operations/operations/Update-Language-Model?&pattern=update).

### <a name="request-parameters"></a>Parámetros de solicitud 

|**Nombre**|**Tipo**|**Obligatorio**|**Descripción**|
|---|---|---|---|
|ubicación|string|Sí|La región de Azure a la que se debe enrutar la llamada. Para más información, consulte [Regiones de Azure y Video Indexer](regions.md).|
|accountID|string|Sí|Identificador único global de la cuenta.|
|modelId|string|Sí|El identificador del modelo de lenguaje (se genera cuando se crea el modelo de lenguaje).|
|accessToken|string|Sí|Token de acceso (debe ser de ámbito [Token de acceso de cuenta](https://api-portal.videoindexer.ai/docs/services/authorization/operations/Get-Account-Access-Token?)) para autenticarse en la llamada. Los tokens de acceso expiran en 1 hora.|
|modelName|string|No|Nuevo nombre que puede dar al modelo.|
|habilitar|boolean|No|Elija si todos los archivos de este modelo están habilitados (true) o deshabilitados (false).|

### <a name="request-body"></a>Cuerpo de la solicitud

Para cargar los archivos que se agregarán al modelo de lenguaje, debe cargar los archivos en el cuerpo mediante datos de formulario y además proporcionar valores para los parámetros necesarios anteriores. Existen dos formas de hacerlo: 

1. La clave será el nombre de archivo y el valor será el archivo .txt.
2. La clave será el nombre de archivo y el valor será una dirección URL al archivo .txt.

### <a name="response"></a>Respuesta

La respuesta proporciona metadatos sobre el modelo de lenguaje recién entrenado, junto con metadatos sobre cada uno de los archivos del modelo, siguiendo el formato de la salida JSON de ejemplo.

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
Puede usar el valor de **id** de los archivos aquí devueltos para descargar el contenido del archivo.

## <a name="update-a-file-from-a-language-model"></a>Actualización de un archivo de un modelo de lenguaje

El siguiente comando le permite actualizar el nombre y **habilitar** el estado de un archivo en un modelo de lenguaje personalizado de la cuenta especificada.

### <a name="request-url"></a>URL de la solicitud

Es una solicitud PUT.

```
https://api.videoindexer.ai/{location}/Accounts/{accountId}/Customization/Language/{modelId}/Files/{fileId}?accessToken={accessToken}[&fileName][&enable]
```

Debajo está la solicitud en Curl.

```curl
curl -v -X PUT "https://api.videoindexer.ai/{location}/Accounts/{accountId}/Customization/Language/{modelId}/Files/{fileId}?accessToken={accessToken}?fileName={string}&enable={string}"
```
 
[Consulte los parámetros necesarios y haga la prueba mediante el portal para desarrolladores de Video Indexer](https://api-portal.videoindexer.ai/docs/services/operations/operations/Update-Language-Model-file?&pattern=update).

### <a name="request-parameters"></a>Parámetros de solicitud 

|**Nombre**|**Tipo**|**Obligatorio**|**Descripción**|
|---|---|---|---|
|ubicación|string|Sí|La región de Azure a la que se debe enrutar la llamada. Para más información, consulte [Regiones de Azure y Video Indexer](regions.md).|
|accountId|string|Sí|Identificador único global de la cuenta.|
|modelId|string|Sí|Identificador del modelo de lenguaje que contiene el archivo (se genera cuando se crea el modelo de lenguaje).|
|fileId|string|Sí|Identificador del archivo que se está actualizando (se genera cuando se carga el archivo al crearse o actualizarse el modelo de lenguaje).|
|accessToken|string|Sí|Token de acceso (debe ser de ámbito [Token de acceso de cuenta](https://api-portal.videoindexer.ai/docs/services/authorization/operations/Get-Account-Access-Token?)) para autenticarse en la llamada. Los tokens de acceso expiran en 1 hora.|
|fileName|string|No|Nombre al que se actualiza el nombre de archivo.|
|habilitar|boolean|No|Actualizar si este archivo está habilitado (true) o deshabilitado (false) en el modelo de lenguaje.|

### <a name="request-body"></a>Cuerpo de la solicitud

No hay ningún otro cuerpo de solicitud necesario para esta llamada.

### <a name="response"></a>Respuesta

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
Puede usar el valor de **id** de los archivos aquí devueltos para descargar el contenido del archivo.

## <a name="get-a-specific-language-model"></a>Obtención de un modelo de lenguaje específico

El siguiente comando devuelve información sobre el modelo de lenguaje especificado en la cuenta especificada, como el lenguaje y los archivos que se encuentran en el modelo de lenguaje. 

### <a name="request-url"></a>URL de la solicitud

Es una solicitud GET.
```
https://api.videoindexer.ai/{location}/Accounts/{accountId}/Customization/Language/{modelId}?accessToken={accessToken}
```

Debajo está la solicitud en Curl.

```curl
curl -v -X GET "https://api.videoindexer.ai/{location}/Accounts/{accountId}/Customization/Language/{modelId}?accessToken={accessToken}"
```
 
[Consulte los parámetros necesarios y haga la prueba mediante el portal para desarrolladores de Video Indexer](https://api-portal.videoindexer.ai/docs/services/operations/operations/Get-Language-Model?&pattern=get).

### <a name="request-parameters-and-request-body"></a>Parámetros de solicitud y cuerpo de solicitud

|**Nombre**|**Tipo**|**Obligatorio**|**Descripción**|
|---|---|---|---|
|ubicación|string|Sí|La región de Azure a la que se debe enrutar la llamada. Para más información, consulte [Regiones de Azure y Video Indexer](regions.md).|
|accountID|string|Sí|Identificador único global de la cuenta.|
|modelId|string|Sí|El identificador del modelo de lenguaje (se genera cuando se crea el modelo de lenguaje).|
|accessToken|string|Sí|Token de acceso (debe ser de ámbito [Token de acceso de cuenta](https://api-portal.videoindexer.ai/docs/services/authorization/operations/Get-Account-Access-Token?)) para autenticarse en la llamada. Los tokens de acceso expiran en 1 hora.|

### <a name="request-body"></a>Cuerpo de la solicitud

No hay ningún otro cuerpo de solicitud necesario para esta llamada.

### <a name="response"></a>Respuesta

La respuesta proporciona metadatos sobre el modelo de lenguaje especificado, junto con metadatos sobre cada uno de los archivos del modelo, siguiendo el formato de la salida JSON de ejemplo.

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

Puede usar el valor de **id** de los archivos aquí devueltos para descargar el contenido del archivo.

## <a name="get-all-the-language-models"></a>Obtención de todos los modelos de lenguaje

El comando siguiente devuelve todos los modelos de lenguaje personalizados en la cuenta especificada en una lista.

### <a name="request-url"></a>URL de la solicitud

Es una solicitud GET.

```
https://api.videoindexer.ai/{location}/Accounts/{accountId}/Customization/Language?accessToken={accessToken}
```

Debajo está la solicitud en Curl.

```curl
curl -v -X GET "https://api.videoindexer.ai/{location}/Accounts/{accountId}/Customization/Language?accessToken={accessToken}"
```
 
[Consulte los parámetros necesarios y haga la prueba mediante el portal para desarrolladores de Video Indexer](https://api-portal.videoindexer.ai/docs/services/operations/operations/Get-Language-Models?&pattern=get).

### <a name="request-parameters"></a>Parámetros de solicitud

|**Nombre**|**Tipo**|**Obligatorio**|**Descripción**|
|---|---|---|---|
|ubicación|string|Sí|La región de Azure a la que se debe enrutar la llamada. Para más información, consulte [Regiones de Azure y Video Indexer](regions.md).|
|accountID|string|Sí|Identificador único global de la cuenta.|
|accessToken|string|Sí|Token de acceso (debe ser de ámbito [Token de acceso de cuenta](https://api-portal.videoindexer.ai/docs/services/authorization/operations/Get-Account-Access-Token?)) para autenticarse en la llamada. Los tokens de acceso expiran en 1 hora.|

### <a name="request-body"></a>Cuerpo de la solicitud

No hay ningún otro cuerpo de solicitud necesario para esta llamada.

### <a name="response"></a>Respuesta

La respuesta proporciona una lista de todos los modelos de lenguaje en su cuenta y cada uno de sus metadatos y archivos, siguiendo el formato de la salida JSON de ejemplo siguiente.

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

El siguiente comando elimina el archivo especificado del modelo de lenguaje especificado en la cuenta especificada. 

### <a name="request-url"></a>URL de la solicitud

Esta es una solicitud de eliminación.
```
https://api.videoindexer.ai/{location}/Accounts/{accountId}/Customization/Language/{modelId}/Files/{fileId}?accessToken={accessToken}
```

Debajo está la solicitud en Curl.

```curl
curl -v -X DELETE "https://api.videoindexer.ai/{location}/Accounts/{accountId}/Customization/Language/{modelId}/Files/{fileId}?accessToken={accessToken}"
```
 
[Consulte los parámetros necesarios y haga la prueba mediante el portal para desarrolladores de Video Indexer](https://api-portal.videoindexer.ai/docs/services/operations/operations/Delete-Language-Model-File?&pattern=delete).

### <a name="request-parameters"></a>Parámetros de solicitud 

|**Nombre**|**Tipo**|**Obligatorio**|**Descripción**|
|---|---|---|---|
|ubicación|string|Sí|La región de Azure a la que se debe enrutar la llamada. Para más información, consulte [Regiones de Azure y Video Indexer](regions.md).|
|accountID|string|Sí|Identificador único global de la cuenta.|
|modelId|string|Sí|Identificador del modelo de lenguaje que contiene el archivo (se genera cuando se crea el modelo de lenguaje).|
|fileId|string|Sí|Identificador del archivo que se está actualizando (se genera cuando se carga el archivo al crearse o actualizarse el modelo de lenguaje).|
|accessToken|string|Sí|Token de acceso (debe ser de ámbito [Token de acceso de cuenta](https://api-portal.videoindexer.ai/docs/services/authorization/operations/Get-Account-Access-Token?)) para autenticarse en la llamada. Los tokens de acceso expiran en 1 hora.|

### <a name="request-body"></a>Cuerpo de la solicitud

No hay ningún otro cuerpo de solicitud necesario para esta llamada.

### <a name="response"></a>Respuesta

No se devuelve ningún contenido cuando el archivo se elimina correctamente del modelo de lenguaje.

## <a name="get-metadata-on-a-file-from-a-language-model"></a>Obtención de metadatos sobre un archivo de un modelo de lenguaje

Se devuelve el contenido del archivo especificado y los metadatos sobre este archivo del modelo de lenguaje elegido en su cuenta.

### <a name="request-url"></a>URL de la solicitud

Es una solicitud GET.

```
https://api.videoindexer.ai/{location}/Accounts/{accountId}/Customization/PersonModels?accessToken={accessToken}
```

Debajo está la solicitud en Curl.
```curl
curl -v -X GET "https://api.videoindexer.ai/{location}/Accounts/{accountId}/Customization/Language/{modelId}/Files/{fileId}?accessToken={accessToken}"
```
 
[Consulte los parámetros necesarios y haga la prueba mediante el portal para desarrolladores de Video Indexer](https://api-portal.videoindexer.ai/docs/services/operations/operations/Get-Language-Model-File-Data?&pattern=get%20language%20model).

### <a name="request-parameters"></a>Parámetros de solicitud 

|**Nombre**|**Tipo**|**Obligatorio**|**Descripción**|
|---|---|---|---|
|ubicación|string|Sí|La región de Azure a la que se debe enrutar la llamada. Para más información, consulte [Regiones de Azure y Video Indexer](regions.md).|
|accountID|string|Sí|Identificador único global de la cuenta.|
|modelId|string|Sí|Identificador del modelo de lenguaje que contiene el archivo (se genera cuando se crea el modelo de lenguaje).|
|fileId|string|Sí|Identificador del archivo que se está actualizando (se genera cuando se carga el archivo al crearse o actualizarse el modelo de lenguaje).|
|accessToken|string|Sí|Token de acceso (debe ser de ámbito [Token de acceso de cuenta](https://api-portal.videoindexer.ai/docs/services/authorization/operations/Get-Account-Access-Token?)) para autenticarse en la llamada. Los tokens de acceso expiran en 1 hora.|

### <a name="request-body"></a>Cuerpo de la solicitud

No hay ningún otro cuerpo de solicitud necesario para esta llamada.

### <a name="response"></a>Respuesta

La respuesta proporciona el contenido y los metadatos del archivo en formato JSON, de forma parecida a la siguiente:

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

El siguiente comando descarga un archivo de texto, que incluye el contenido del archivo especificado, del modelo de lenguaje especificado en la cuenta especificada. Este archivo de texto debe coincidir con el contenido del archivo de texto que se cargó originalmente.

### <a name="request-url"></a>URL de la solicitud
```
https://api.videoindexer.ai/{location}/Accounts/{accountId}/Customization/Language/{modelId}/Files/{fileId}/download?accessToken={accessToken}
```

Debajo está la solicitud en Curl.

```curl
curl -v -X GET "https://api.videoindexer.ai/{location}/Accounts/{accountId}/Customization/Language/{modelId}/Files/{fileId}/download?accessToken={accessToken}"
```
 
[Consulte los parámetros necesarios y haga la prueba mediante el portal para desarrolladores de Video Indexer](https://api-portal.videoindexer.ai/docs/services/operations/operations/Download-Language-Model-File-Content?).

### <a name="request-parameters"></a>Parámetros de solicitud 

|**Nombre**|**Tipo**|**Obligatorio**|**Descripción**|
|---|---|---|---|
|ubicación|string|Sí|La región de Azure a la que se debe enrutar la llamada. Para más información, consulte [Regiones de Azure y Video Indexer](regions.md).|
|accountID|string|Sí|Identificador único global de la cuenta.|
|modelId|string|Sí|Identificador del modelo de lenguaje que contiene el archivo (se genera cuando se crea el modelo de lenguaje).|
|fileId|string|Sí|Identificador del archivo que se está actualizando (se genera cuando se carga el archivo al crearse o actualizarse el modelo de lenguaje).|
|accessToken|string|Sí|Token de acceso (debe ser de ámbito [Token de acceso de cuenta](https://api-portal.videoindexer.ai/docs/services/authorization/operations/Get-Account-Access-Token?)) para autenticarse en la llamada. Los tokens de acceso expiran en 1 hora.|

### <a name="request-body"></a>Cuerpo de la solicitud 

No hay ningún otro cuerpo de solicitud necesario para esta llamada.

### <a name="response"></a>Respuesta

La respuesta será la descarga de un archivo de texto con el contenido del archivo en formato JSON. 

## <a name="next-steps"></a>Pasos siguientes

[Customize Language model using website](customize-language-model-with-website.md) (Personalización del modelo de lenguaje mediante el sitio web)
