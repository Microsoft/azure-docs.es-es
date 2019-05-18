---
title: Use la CLI para escalar unidades reservadas de multimedia en Azure | Microsoft Docs
description: En este tema se muestra cómo usar la CLI para escalar el procesamiento multimedia con Azure Media Services.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/24/2019
ms.author: juliako
ms.custom: seodec18
ms.openlocfilehash: b1a79f2798fc98fd7361c47788c79e329e2cb827
ms.sourcegitcommit: 9e8dfa1169a55c3c8af93a6c5f4e0dace4de48b2
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2019
ms.locfileid: "65556186"
---
# <a name="scaling-media-processing"></a>Escalado de procesamiento de elementos multimedia

Azure Media Services le permite escalar el procesamiento de elementos multimedia en su cuenta mediante la administración de unidades reservadas de multimedia (MRU). MRU determinan la velocidad con la que se procesan las tareas de procesamiento multimedia. Puede elegir uno de los siguientes tipos de unidad reservada: **S1**, **S2** o **S3**. Por ejemplo, el mismo trabajo de codificación se ejecuta más rápido cuando se usa el tipo de unidad reservada **S2** en comparación con el tipo**S1**. 

Además de especificar el tipo de unidad reservada, puede especificar el aprovisionamiento de su cuenta con unidades reservadas de codificación. El número de unidades reservadas de codificación aprovisionadas determina el número de tareas de medios que se pueden procesar de forma simultánea en una cuenta determinada. Por ejemplo, si la cuenta tiene cinco unidades reservadas, se ejecutarán simultáneamente cinco tareas multimedia siempre que haya tareas para procesar. Las tareas restantes esperarán en la cola y se elegirán para el procesamiento secuencialmente cuando finalice la tarea en ejecución. Si una cuenta no tiene ninguna unidad reservada aprovisionada, las tareas se elegirán de manera secuencial. En este caso, el tiempo de espera entre la finalización de una tarea y el inicio de la siguiente dependerá de la disponibilidad de los recursos del sistema.

## <a name="choosing-between-different-reserved-unit-types"></a>Selección de los distintos tipos de unidad reservada

Con esta tabla será más fácil tomar la decisión de elegir entre distintas velocidades de codificación. En ella se proporcionan también algunos casos de referencias comparativas en [un vídeo que puede descargar](https://nimbuspmteam.blob.core.windows.net/asset-46f1f723-5d76-477e-a153-3fd0f9f90f73/SeattlePikePlaceMarket_7min.ts?sv=2015-07-08&sr=c&si=013ab6a6-5ebf-431e-8243-9983a6b5b01c&sig=YCgEB8DxYKK%2B8W9LnBykzm1ZRUTwQAAH9QFUGw%2BIWuc%3D&se=2118-09-21T19%3A28%3A57Z) para realizar sus propias pruebas:

|Tipo de RU|Escenario|Resultados de ejemplo para el [vídeo de 7 min y 1080 p](https://nimbuspmteam.blob.core.windows.net/asset-46f1f723-5d76-477e-a153-3fd0f9f90f73/SeattlePikePlaceMarket_7min.ts?sv=2015-07-08&sr=c&si=013ab6a6-5ebf-431e-8243-9983a6b5b01c&sig=YCgEB8DxYKK%2B8W9LnBykzm1ZRUTwQAAH9QFUGw%2BIWuc%3D&se=2118-09-21T19%3A28%3A57Z)|
|---|---|---|
| **S1**|Codificación con velocidad de bits sencilla. <br/>Archivos con resolución SD o menor, no sujetos a limitación temporal y de bajo costo.|Codificación de archivo de MP4 de velocidad de bits única SD resolución mediante "H264 Single Bitrate SD 16 x 9" tarda aproximadamente 7 minutos.|
| **S2**|Codificación con velocidad de bits sencilla y múltiple.<br/>Uso normal para codificación SD y HD.|Codificación con "H264 Single Bitrate 720p" el valor predeterminado tardará aproximadamente 6 minutos.<br/><br/>Codificación con "H264 Multiple Bitrate 720p" valor predeterminado tardará aproximadamente 12 minutos.|
| **S3**|Codificación con velocidad de bits sencilla y múltiple.<br/>Vídeos con resolución Full HD y 4K. Codificación con respuesta más rápida, sujeta a limitación temporal.|Codificación con "H264 Single Bitrate 1080p" había preestablecido tarda unos 3 minutos.<br/><br/>Codificación con "H264 Multiple Bitrate 1080p" valor predeterminado tardará aproximadamente unos 8 minutos.|

## <a name="considerations"></a>Consideraciones

* Para los trabajos de análisis de audio y vídeo desencadenados por Media Services v3 o Video Indexer, se recomienda encarecidamente el tipo de unidad S3.
* Si utiliza el grupo compartido, es decir, sin ninguna unidad reservada, las tareas de codificación tendrán el mismo rendimiento que con las RU S1. Sin embargo, no existe ningún límite superior para el tiempo que las tareas pueden estar en el estado en cola, y en un momento determinado, se estará ejecutando a lo sumo una tarea.

El resto del artículo muestra cómo usar [Media Services v3 CLI](https://aka.ms/ams-v3-cli-ref) escalar MRU.

> [!NOTE]
> Para los trabajos de análisis de audio y vídeo desencadenados por Media Services v3 o Video Indexer, se recomienda encarecidamente aprovisionar la cuenta con 10 MRU S3. Si necesita más de 10 MRU S3, abra una incidencia de soporte técnico desde [Azure Portal](https://portal.azure.com/).
>
> Actualmente, no se puede usar el portal de Azure para administrar otros recursos v3. Use la [API REST](https://aka.ms/ams-v3-rest-ref), la [CLI](https://aka.ms/ams-v3-cli-ref) o uno de los [SDK](media-services-apis-overview.md#sdks) admitidos.

## <a name="prerequisites"></a>Requisitos previos 

[Cree una cuenta de Media Services](create-account-cli-how-to.md).

[!INCLUDE [media-services-cli-instructions](../../../includes/media-services-cli-instructions.md)]

## <a name="scale-media-reserved-units-with-cli"></a>Escalado de unidades reservadas de multimedia con CLI

Ejecute el comando `mru`.

El comando [mru de la cuenta de ams az](https://docs.microsoft.com/cli/azure/ams/account/mru?view=azure-cli-latest) siguiente establece las unidades reservadas de multimedia en cuenta "amsaccount" mediante los parámetros **count** y **type**.

```azurecli
az ams account mru set -n amsaccount -g amsResourceGroup --count 10 --type S3
```

## <a name="billing"></a>Facturación

Se cobran en función del número de minutos que se aprovisionan las unidades reservadas de multimedia en su cuenta. Esto ocurre independientemente de si existen todos los trabajos que se ejecutan en su cuenta. Para obtener una explicación detallada, vea la sección de preguntas más frecuentes de la página de [precios de Media Services](https://azure.microsoft.com/pricing/details/media-services/).   

## <a name="next-step"></a>Paso siguiente

[Análisis de vídeos](analyze-videos-tutorial-with-api.md) 

## <a name="see-also"></a>Vea también

* [Cuotas y limitaciones](limits-quotas-constraints.md)
* [CLI de Azure](https://docs.microsoft.com/cli/azure/ams?view=azure-cli-latest)
