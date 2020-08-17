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
ms.date: 03/09/2020
ms.author: juliako
ms.custom: seodec18
ms.openlocfilehash: 6715014485b227713447ce5d552cf7ba79737845
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87053223"
---
# <a name="scaling-media-processing"></a>Escalado de procesamiento de elementos multimedia

Azure Media Services le permite escalar el procesamiento de elementos multimedia en su cuenta mediante la administración de unidades reservadas de multimedia (MRU). Las MRU determinan la velocidad con la que se procesan las tareas de procesamiento multimedia. Puede elegir uno de los siguientes tipos de unidad reservada: **S1**, **S2** o **S3**. Por ejemplo, el mismo trabajo de codificación se ejecuta más rápido cuando se usa el tipo de unidad reservada **S2** en comparación con el tipo**S1**. 

Además de especificar el tipo de unidad reservada, puede especificar el aprovisionamiento de su cuenta con unidades reservadas de codificación. El número de unidades reservadas de codificación aprovisionadas determina el número de tareas de medios que se pueden procesar de forma simultánea en una cuenta determinada. Por ejemplo, si la cuenta tiene cinco unidades reservadas, se ejecutarán simultáneamente cinco tareas multimedia siempre que haya tareas para procesar. Las tareas restantes esperarán en la cola y se elegirán para el procesamiento secuencialmente cuando finalice la tarea en ejecución. Si una cuenta no tiene ninguna unidad reservada aprovisionada, las tareas se elegirán de manera secuencial. En este caso, el tiempo de espera entre la finalización de una tarea y el inicio de la siguiente dependerá de la disponibilidad de los recursos del sistema.

## <a name="choosing-between-different-reserved-unit-types"></a>Selección de los distintos tipos de unidad reservada

Con esta tabla será más fácil tomar la decisión de elegir entre distintas velocidades de codificación. En ella se proporcionan también algunos casos de referencias comparativas en [un vídeo que puede descargar](https://nimbuspmteam.blob.core.windows.net/asset-46f1f723-5d76-477e-a153-3fd0f9f90f73/SeattlePikePlaceMarket_7min.ts?sv=2015-07-08&sr=c&si=013ab6a6-5ebf-431e-8243-9983a6b5b01c&sig=YCgEB8DxYKK%2B8W9LnBykzm1ZRUTwQAAH9QFUGw%2BIWuc%3D&se=2118-09-21T19%3A28%3A57Z) para realizar sus propias pruebas:

|Tipo de RU|Escenario|Resultados de ejemplo para el [vídeo de 7 min y 1080 p](https://nimbuspmteam.blob.core.windows.net/asset-46f1f723-5d76-477e-a153-3fd0f9f90f73/SeattlePikePlaceMarket_7min.ts?sv=2015-07-08&sr=c&si=013ab6a6-5ebf-431e-8243-9983a6b5b01c&sig=YCgEB8DxYKK%2B8W9LnBykzm1ZRUTwQAAH9QFUGw%2BIWuc%3D&se=2118-09-21T19%3A28%3A57Z)|
|---|---|---|
| **S1**|Codificación con velocidad de bits sencilla. <br/>Archivos con resolución SD o menor, no sujetos a limitación temporal y de bajo costo.|La codificación en el archivo MP4 de resolución SD de velocidad de bits única con "H264 Single Bitrate SD 16x9" tarda en torno a 7 minutos.|
| **S2**|Codificación con velocidad de bits sencilla y múltiple.<br/>Uso normal para codificación SD y HD.|La codificación con el valor predeterminado "H264 Single Bitrate 720p" tarda en torno a 6 minutos.<br/><br/>La codificación con el valor predeterminado "H264 Multiple Bitrate 720p" tarda en torno a 12 minutos.|
| **S3**|Codificación con velocidad de bits sencilla y múltiple.<br/>Vídeos con resolución Full HD y 4K. Codificación con respuesta más rápida, sujeta a limitación temporal.|La codificación con el valor predeterminado "H264 Single Bitrate 1080p" tardará aproximadamente 3 minutos.<br/><br/>La codificación con el valor predeterminado "H264 Multiple Bitrate 1080p" tarda aproximadamente 8 minutos.|

## <a name="considerations"></a>Consideraciones

* Para los trabajos de análisis de audio y vídeo desencadenados por Media Services v3 o Video Indexer, se recomienda encarecidamente el tipo de unidad S3.
* Si utiliza el grupo compartido, es decir, sin ninguna unidad reservada, las tareas de codificación tendrán el mismo rendimiento que con las RU S1. Sin embargo, no existe ningún límite superior para el tiempo que las tareas pueden estar en el estado en cola, y en un momento determinado, se estará ejecutando a lo sumo una tarea.

En el resto del artículo se muestra cómo usar la [CLI de Media Services v3](https://aka.ms/ams-v3-cli-ref) para escalar las MRU.

> [!NOTE]
> Para los trabajos de análisis de audio y vídeo desencadenados por Media Services v3 o Video Indexer, se recomienda encarecidamente aprovisionar la cuenta con 10 MRU S3. Si necesita más de 10 MRU S3, abra una incidencia de soporte técnico desde [Azure Portal](https://portal.azure.com/).

## <a name="prerequisites"></a>Prerrequisitos 

[Cree una cuenta de Media Services](./create-account-howto.md).

[!INCLUDE [media-services-cli-instructions](../../../includes/media-services-cli-instructions.md)]

## <a name="scale-media-reserved-units-with-cli"></a>Escalado de unidades reservadas de multimedia con CLI

Ejecute el comando `mru`.

El comando [mru de la cuenta de ams az](/cli/azure/ams/account/mru?view=azure-cli-latest) siguiente establece las unidades reservadas de multimedia en cuenta "amsaccount" mediante los parámetros **count** y **type**.

```azurecli
az ams account mru set -n amsaccount -g amsResourceGroup --count 10 --type S3
```

## <a name="billing"></a>Facturación

Se le cobra en función del número de minutos que se aprovisionan las unidades reservadas de multimedia en su cuenta. Esto ocurre independientemente de si se ejecutan trabajos en la cuenta. Para obtener una explicación detallada, vea la sección de preguntas más frecuentes de la página de [precios de Media Services](https://azure.microsoft.com/pricing/details/media-services/).   

## <a name="next-step"></a>Paso siguiente

[Análisis de vídeos](analyze-videos-tutorial-with-api.md) 

## <a name="see-also"></a>Consulte también

* [Cuotas y límites](limits-quotas-constraints.md)
* [CLI de Azure](/cli/azure/ams?view=azure-cli-latest)
