---
title: 'Uso de la API de Video Indexer para personalizar un modelo de persona: Azure'
titlesuffix: Azure Media Services
description: En este artículo se muestra cómo personalizar un modelo de persona con la API de Video Indexer.
services: media-services
author: anikaz
manager: johndeu
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 05/15/2019
ms.author: anzaman
ms.openlocfilehash: 6c4980536eddd0226fac422ae17ddb717e34630d
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65799471"
---
# <a name="customize-a-person-model-with-the-video-indexer-api"></a>Personalización de un modelo de persona con la API de Video Indexer

Video Indexer admite la detección de caras y el reconocimiento de celebridades en contenido de vídeo. La característica de reconocimiento de celebridades abarca aproximadamente un millón de caras basadas en orígenes de datos comúnmente solicitados, como IMDB, Wikipedia y personas con más influencia de LinkedIn. Las caras que no reconoce la característica de reconocimiento de celebridades se detectan, pero se dejan sin nombre. Después de cargar un vídeo en Video Indexer y recibir los resultados, puede volver y poner nombre a las caras que no se reconocieron. Cuando se etiqueta una cara con un nombre, la cara y el nombre se agregan al modelo de persona de la cuenta. Así, Video Indexer reconocerá esta cara en los vídeos futuros y pasados.

Puede usar la API de Video Indexer para editar las caras que se detectaron en un vídeo, como se describe en este tema. También puede usar el sitio web de Video Indexer, como se describe en [Customize Person model using the Video Indexer website](customize-person-model-with-api.md) (Personalización de un modelo de persona mediante el sitio web de Video Indexer).

## <a name="managing-multiple-person-models"></a>Administración de varios modelos de persona 

Video Indexer admite varios modelos de persona por cuenta. Esta característica está actualmente disponible solo mediante las API de Video Indexer.

Si su cuenta sirve a diferentes escenarios de casos de uso, podría crear varios modelos de persona por cuenta. Por ejemplo, si su contenido está relacionado con deportes, puede crear un modelo de persona independiente para cada deporte (fútbol, baloncesto, etc.). 

Una vez que se crea un modelo, para usarlo, proporcione el identificador de modelo de un modelo de persona específico al cargar/indexar o volver a indexar un vídeo. Al entrenar una nueva cara de un vídeo se actualiza el modelo personalizado específico con el que está asociado ese vídeo.

Cada cuenta tiene un límite de 50 modelos de persona. Si no necesita compatibilidad con varios modelos de persona, no asigne un identificador de modelo de persona a su vídeo al cargar/indexar o volver a indexar. En este caso, Video Indexer usa el modelo de persona personalizado predeterminado de su cuenta.

## <a name="create-a-new-person-model"></a>Creación de un modelo de persona

Cree un modelo de persona en la cuenta especificada. 

### <a name="request-url"></a>URL de la solicitud

Esta es una solicitud POST.

```
https://api.videoindexer.ai/{location}/Accounts/{accountId}/Customization/PersonModels?name={name}&accessToken={accessToken}
```

Debajo está la solicitud en Curl.

```curl
curl -v -X POST "https://api.videoindexer.ai/{location}/Accounts/{accountId}/Customization/PersonModels?name={name}&accessToken={accessToken}"
```

[Consulte los parámetros necesarios y haga la prueba mediante el portal para desarrolladores de Video Indexer](https://api-portal.videoindexer.ai/docs/services/operations/operations/Create-Person-Model?).

### <a name="request-parameters"></a>Parámetros de solicitud 

|**Nombre**|**Tipo**|**Obligatorio**|**Descripción**|
|---|---|---|---|
|ubicación|string|Sí|La región de Azure a la que se debe enrutar la llamada. Para más información, consulte [Regiones de Azure y Video Indexer](regions.md).|
|accountId|string|Sí|Identificador único global de la cuenta.|
|Nombre|string|Sí|El nombre del modelo de persona.|
|accessToken|string|Sí|Token de acceso (debe ser de ámbito [Token de acceso de cuenta](https://api-portal.videoindexer.ai/docs/services/authorization/operations/Get-Account-Access-Token?)) para autenticarse en la llamada. Los tokens de acceso expiran en 1 hora.|

### <a name="request-body"></a>Cuerpo de la solicitud

No hay ningún otro cuerpo de solicitud necesario para esta llamada.

### <a name="response"></a>Respuesta

La respuesta proporciona el nombre y el identificador de modelo generado del modelo de persona que acaba de crear, siguiendo el formato del ejemplo siguiente.

```json
{
    "id": "227654b4-912c-4b92-ba4f-641d488e3720",
    "name": "Example Person Model"
}
```

Así, usaría el valor **id** para el parámetro **personModelId** al [cargar un vídeo para indexar](https://api-portal.videoindexer.ai/docs/services/operations/operations/Upload-video?) o al [volver a indexar un vídeo](https://api-portal.videoindexer.ai/docs/services/operations/operations/Re-index-video?).

## <a name="delete-a-person-model"></a>Eliminación de un modelo de persona

Elimine un modelo de persona personalizado de la cuenta especificada. 

Una vez que el modelo de persona se elimina correctamente, el índice de los vídeos actuales que estaban usando el modelo eliminado permanecerá sin cambios hasta que los vuelva a indexar. Tras volverlos a indexar, Video Indexer no reconocerá las caras que tienen un nombre en el modelo eliminado en los vídeos actuales que se indexaron mediante ese modelo; sin embargo, esas caras se seguirán detectando. Los vídeos actuales que se indexaron mediante el modelo eliminado ahora usarán el modelo de persona predeterminado de la cuenta. Si las caras del modelo eliminado también tienen un nombre en el modelo predeterminado de la cuenta, esas caras se seguirán reconociendo en los vídeos.

### <a name="request-url"></a>URL de la solicitud

```
https://api.videoindexer.ai/{location}/Accounts/{accountId}/Customization/PersonModels/{id}?accessToken={accessToken}
```

Debajo está la solicitud en Curl.
```curl
curl -v -X DELETE "https://api.videoindexer.ai/{location}/Accounts/{accountId}/Customization/PersonModels/{id}?accessToken={accessToken}"
```

[Consulte los parámetros necesarios y haga la prueba mediante el portal para desarrolladores de Video Indexer](https://api-portal.videoindexer.ai/docs/services/operations/operations/Delete-Person-Model?).

### <a name="request-parameters"></a>Parámetros de solicitud

|**Nombre**|**Tipo**|**Obligatorio**|**Descripción**|
|---|---|---|---|
|ubicación|string|Sí|La región de Azure a la que se debe enrutar la llamada. Para más información, consulte [Regiones de Azure y Video Indexer](regions.md).|
|accountId|string|Sí|Identificador único global de la cuenta.|
|id|string|Sí|El identificador del modelo de persona (se genera cuando se crea el modelo de persona)|
|accessToken|string|Sí|Token de acceso (debe ser de ámbito [Token de acceso de cuenta](https://api-portal.videoindexer.ai/docs/services/authorization/operations/Get-Account-Access-Token?)) para autenticarse en la llamada. Los tokens de acceso expiran en 1 hora.|

### <a name="request-body"></a>Cuerpo de la solicitud

No hay ningún otro cuerpo de solicitud necesario para esta llamada.

### <a name="response"></a>Respuesta

No se devuelve ningún contenido cuando el modelo de persona se elimina correctamente.

## <a name="get-all-person-models"></a>Obtención de todos los modelos de persona

Obtenga todos los modelos de persona de la cuenta especificada. 

### <a name="request-call"></a>Solicitud de llamada

Es una solicitud GET.

```
https://api.videoindexer.ai/{location}/Accounts/{accountId}/Customization/PersonModels?accessToken={accessToken}
```

Debajo está la solicitud en Curl.

```curl
curl -v -X GET "https://api.videoindexer.ai/{location}/Accounts/{accountId}/Customization/PersonModels?accessToken={accessToken}"
```

[Consulte los parámetros necesarios y haga la prueba mediante el portal para desarrolladores de Video Indexer](https://api-portal.videoindexer.ai/docs/services/operations/operations/Get-Person-Models?).

### <a name="request-parameters"></a>Parámetros de solicitud

|**Nombre**|**Tipo**|**Obligatorio**|**Descripción**|
|---|---|---|---|
|ubicación|string|Sí|La región de Azure a la que se debe enrutar la llamada. Para más información, consulte [Regiones de Azure y Video Indexer](regions.md).|
|accountId|string|Sí|Identificador único global de la cuenta.|
|accessToken|string|Sí|Token de acceso (debe ser de ámbito [Token de acceso de cuenta](https://api-portal.videoindexer.ai/docs/services/authorization/operations/Get-Account-Access-Token?)) para autenticarse en la llamada. Los tokens de acceso expiran en 1 hora.|

### <a name="request-body"></a>Cuerpo de la solicitud

No hay ningún otro cuerpo de solicitud necesario para esta llamada.

### <a name="response"></a>Respuesta

La respuesta proporciona una lista de todos los modelos de persona de su cuenta (incluido el modelo de persona predeterminado de la cuenta especificada) y cada uno de sus nombres e identificadores, siguiendo el formato del ejemplo siguiente.

```json
[
    {
        "id": "59f9c326-b141-4515-abe7-7d822518571f",
        "name": "Default"
    }, 
    {
        "id": "9ef2632d-310a-4510-92e1-cc70ae0230d4",
        "name": "Test"
    }
]
```

Para elegir qué modelo quiere usar para un vídeo, utilice el valor **id** del modelo de persona para el parámetro **personModelId** al [cargar un vídeo para indexar](https://api-portal.videoindexer.ai/docs/services/operations/operations/Upload-video?) o al [volver a indexar un vídeo](https://api-portal.videoindexer.ai/docs/services/operations/operations/Re-index-video?).

## <a name="update-a-face"></a>Actualización de una cara

Este comando permite actualizar una cara del vídeo con un nombre mediante el identificador del vídeo y el identificador de la cara. A continuación, actualiza el modelo de persona con el que el vídeo estaba asociado tras cargar/indexar o volver a indexar el vídeo. Si no se ha asignado ningún modelo de persona, se actualiza el modelo de persona predeterminado de la cuenta. 

Una vez que esto sucede, reconoce las apariciones de la misma cara en otros vídeos actuales que comparten el mismo modelo de persona. El reconocimiento de la cara en los otros vídeos actuales puede tardar algún tiempo en aplicarse ya que se trata de un proceso por lotes.

Puede actualizar una cara que Video Indexer haya reconocido como una celebridad con un nuevo nombre. El nuevo nombre que proporcione tendrá prioridad sobre el reconocimiento de celebridades integrado.

### <a name="request-call"></a>Solicitud de llamada

Esta es una solicitud POST.

```
https://api.videoindexer.ai/{location}/Accounts/{accountId}/Videos/{videoId}/Index/Faces/{faceId}?accessToken={accessToken}&newName={newName}
```

Debajo está la solicitud en Curl.

```curl
curl -v -X PUT "https://api.videoindexer.ai/{location}/Accounts/{accountId}/Videos/{videoId}/Index/Faces/{faceId}?accessToken={accessToken}&newName={newName}"
```

[Consulte los parámetros necesarios y haga la prueba mediante el portal para desarrolladores de Video Indexer](https://api-portal.videoindexer.ai/docs/services/operations/operations/Update-Video-Face?).

### <a name="request-parameters"></a>Parámetros de solicitud

|**Nombre**|**Tipo**|**Obligatorio**|**Descripción**|
|---|---|---|---|
|ubicación|string|Sí|La región de Azure a la que se debe enrutar la llamada. Para más información, consulte [Regiones de Azure y Video Indexer](regions.md).|
|accountId|string|Sí|Identificador único global de la cuenta.|
|videoId|string|Sí|Identificador del vídeo en el que aparece la cara que quiere actualizar. Se crea cuando el vídeo se carga e indexa.|
|faceId|integer|Sí|Identificador de la cara que se actualizará. Puede obtener el valor de faceId del índice de vídeo.|
|accessToken|string|Sí|Token de acceso (debe ser de ámbito [Token de acceso de cuenta](https://api-portal.videoindexer.ai/docs/services/authorization/operations/Get-Account-Access-Token?)) para autenticarse en la llamada. Los tokens de acceso expiran en 1 hora.|
|Nombre|string|Sí|Nuevo nombre con el que actualizar la cara.|

Los nombres son únicos para los modelos de persona, así que si da el mismo valor de parámetro **name** a dos caras diferentes del mismo modelo de persona, Video Indexer considera que las caras son de la misma persona y las junta después de volver a indexar el vídeo. 

### <a name="request-body"></a>Cuerpo de la solicitud

No hay ningún otro cuerpo de solicitud necesario para esta llamada.

### <a name="response"></a>Respuesta

No se devuelve ningún contenido cuando la cara se ha actualizado correctamente.

## <a name="next-steps"></a>Pasos siguientes

[Customize Person model using the Video Indexer website](customize-person-model-with-website.md) (Personalización de un modelo de persona mediante el sitio web de Video Indexer)
