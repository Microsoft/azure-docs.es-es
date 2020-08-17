---
title: Personalización de un modelo de persona con la API de Video Indexer
titleSuffix: Azure Media Services
description: Obtenga información sobre cómo personalizar un modelo de persona con la API de Video Indexer.
services: media-services
author: anikaz
manager: johndeu
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 01/14/2020
ms.author: kumud
ms.openlocfilehash: 09366dea1a0d77052b6f99e9f5ab52c270e341b4
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87047019"
---
# <a name="customize-a-person-model-with-the-video-indexer-api"></a>Personalización de un modelo de persona con la API de Video Indexer

Video Indexer admite la detección de caras y el reconocimiento de celebridades en contenido de vídeo. La característica de reconocimiento de celebridades abarca aproximadamente un millón de caras basadas en orígenes de datos comúnmente solicitados, como IMDB, Wikipedia y personas con más influencia de LinkedIn. Las caras que la característica de reconocimiento de celebridades no reconozca se detectan, pero se dejan sin nombre. Después de cargar un vídeo en Video Indexer y recibir los resultados, puede volver y poner nombre a las caras que no se reconocieron. Cuando se etiqueta una cara con un nombre, la cara y el nombre se agregan al modelo de persona de la cuenta. Así, Video Indexer reconocerá esta cara en los vídeos futuros y pasados.

Puede usar la API de Video Indexer para editar las caras que se detectaron en un vídeo, como se describe en este tema. También puede usar el sitio web de Video Indexer, como se describe en [Customize Person model using the Video Indexer website](customize-person-model-with-api.md) (Personalización de un modelo de persona mediante el sitio web de Video Indexer).

## <a name="managing-multiple-person-models"></a>Administración de varios modelos de persona

Video Indexer admite varios modelos de persona por cuenta. Esta característica está actualmente disponible solo mediante las API de Video Indexer.

Si su cuenta sirve a diferentes escenarios de casos de uso, podría crear varios modelos de persona por cuenta. Por ejemplo, si su contenido está relacionado con deportes, puede crear un modelo de persona independiente para cada deporte (fútbol, baloncesto, etc.).

Una vez que se crea un modelo, para usarlo, proporcione el identificador de modelo de un modelo de persona específico al cargar/indexar o volver a indexar un vídeo. Al entrenar una nueva cara de un vídeo se actualiza el modelo personalizado específico con el que está asociado ese vídeo.

Cada cuenta tiene un límite de 50 modelos de persona. Si no necesita compatibilidad con varios modelos de persona, no asigne un identificador de modelo de persona a su vídeo al cargar/indexar o volver a indexar. En este caso, Video Indexer usa el modelo de persona personalizado predeterminado de su cuenta.

## <a name="create-a-new-person-model"></a>Creación de un modelo de persona

Para crear un modelo de persona en la cuenta especificada, use la API para [crear un modelo de persona](https://api-portal.videoindexer.ai/docs/services/operations/operations/Create-Person-Model?).

La respuesta proporciona el nombre y el identificador de modelo generado del modelo de persona que acaba de crear, siguiendo el formato del ejemplo siguiente.

```json
{
    "id": "227654b4-912c-4b92-ba4f-641d488e3720",
    "name": "Example Person Model"
}
```

Así, se usa el valor **id** para el parámetro **personModelId** al [cargar un vídeo para indexar](https://api-portal.videoindexer.ai/docs/services/operations/operations/Upload-video?) o al [volver a indexar un vídeo](https://api-portal.videoindexer.ai/docs/services/operations/operations/Re-index-video?).

## <a name="delete-a-person-model"></a>Eliminación de un modelo de persona

Para eliminar un modelo de persona de la cuenta especificada, use la API para [eliminar un modelo de persona](https://api-portal.videoindexer.ai/docs/services/operations/operations/Delete-Person-Model?).

Una vez que el modelo de persona se elimina correctamente, el índice de los vídeos actuales que estaban usando el modelo eliminado permanecerá sin cambios hasta que los vuelva a indexar. Tras volverlos a indexar, Video Indexer no reconocerá las caras que tengan un nombre en el modelo eliminado en los vídeos actuales que se indexaron con ese modelo, pero esas caras se seguirán detectando. Los vídeos actuales que se indexaron mediante el modelo eliminado ahora usarán el modelo de persona predeterminado de la cuenta. Si las caras del modelo eliminado también tienen un nombre en el modelo predeterminado de la cuenta, esas caras se seguirán reconociendo en los vídeos.

No se devuelve ningún contenido cuando el modelo de persona se elimina correctamente.

## <a name="get-all-person-models"></a>Obtención de todos los modelos de persona

Para obtener todos los modelos de persona en la cuenta especificada, use la API para [obtener un modelo de persona](https://api-portal.videoindexer.ai/docs/services/operations/operations/Get-Person-Models?).

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

Para elegir qué modelo quiere usar en un vídeo, use el valor `id` del modelo de persona en el parámetro `personModelId` al [cargar un vídeo para indexarlo](https://api-portal.videoindexer.ai/docs/services/operations/operations/Upload-video?) o al [volver a indexar un vídeo](https://api-portal.videoindexer.ai/docs/services/operations/operations/Re-index-video?).

## <a name="update-a-face"></a>Actualización de una cara

Este comando permite actualizar una cara del vídeo con un nombre mediante el identificador del vídeo y el identificador de la cara. Esta acción actualiza el modelo de persona con el que el vídeo estaba asociado tras cargar/indexar o volver a indexar el vídeo. Si no se ha asignado ningún modelo de persona, se actualiza el modelo de persona predeterminado de la cuenta.

Tras esto, el sistema reconoce las apariciones de la misma cara en otros vídeos actuales que comparten el mismo modelo de persona. El reconocimiento de la cara en los otros vídeos actuales puede tardar algún tiempo en aplicarse ya que se trata de un proceso por lotes.

Puede actualizar una cara que Video Indexer haya reconocido como una celebridad con un nuevo nombre. El nuevo nombre que proporcione tendrá prioridad sobre el reconocimiento de celebridades integrado.

Para actualizar la cara, use la API para [actualizar una cara de un vídeo](https://api-portal.videoindexer.ai/docs/services/operations/operations/Update-Video-Face?).

Los nombres son únicos para los modelos de persona, así que si da el mismo valor de parámetro `name` a dos caras diferentes del mismo modelo de persona, Video Indexer considera que las caras son de la misma persona y las junta después de volver a indexar el vídeo.

## <a name="next-steps"></a>Pasos siguientes

[Customize Person model using the Video Indexer website](customize-person-model-with-website.md) (Personalización de un modelo de persona mediante el sitio web de Video Indexer)
