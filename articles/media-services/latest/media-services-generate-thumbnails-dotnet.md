---
title: Generación de miniaturas mediante Encoder Standard de Azure Media Services con .NET
description: En este artículo se muestra cómo se usa .NET para codificar un recurso y generar miniaturas al mismo tiempo con Media Encoder Standard.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: 5e4ad7ba75edd1899cbe2d7cd7d3b1b6c124ce35
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89267656"
---
# <a name="how-to-generate-thumbnails-using-encoder-standard-with-net"></a>Generación de miniaturas mediante Encoder Standard con .NET

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Puede utilizar Media Encoder Standard para generar una o varias vistas en miniatura de la entrada de vídeo en formatos de archivo de imagen [JPEG](https://en.wikipedia.org/wiki/JPEG), [PNG](https://en.wikipedia.org/wiki/Portable_Network_Graphics), o [BMP](https://en.wikipedia.org/wiki/BMP_file_format).

## <a name="recommended-reading-and-practice"></a>Práctica y lectura recomendadas

Se recomienda que se familiarice con las transformaciones personalizadas leyendo [Cómo codificar con una transformación personalizada: .NET](customize-encoder-presets-how-to.md).

## <a name="transform-code-example"></a>Ejemplo de código de transformación

En el ejemplo de código siguiente se crea solo una miniatura.  Debe establecer los siguientes parámetros:

- **start**: la posición en el vídeo de entrada desde donde comenzar a generar miniaturas. El valor puede estar en formato ISO 8601 (por ejemplo, PT05S para empezar en 5 segundos), ser un número de fotogramas (por ejemplo, 10 para comenzar en el décimo fotograma) o ser un valor relativo a la duración de la secuencia (por ejemplo, 10 % para empezar en el 10 % de la duración de la secuencia). También admite una macro {Best}, que indica al codificador que seleccione la mejor miniatura en los primeros segundos del vídeo y que solo se producirá una miniatura, independientemente de cuál sea la configuración para step y range. El valor predeterminado es la macro {Best}.
- **step**: los intervalos en los que se generan las miniaturas. El valor puede estar en formato ISO 8601 (por ejemplo, PT05S para una imagen cada 5 segundos), ser un número de fotogramas (por ejemplo, 30 para una imagen cada 30 fotogramas) o ser un valor relativo a la duración de la secuencia (por ejemplo, 10 % para una imagen cada 10 % de la duración de la secuencia). El valor de step afectará a la primera miniatura generada, que puede no ser exactamente la especificada en el momento de inicio preestablecido de la transformación. Esto se debe al codificador, que intenta seleccionar la mejor miniatura entre el momento de inicio y la posición del paso desde dicho momento como la primera salida. Dado que el valor predeterminado es 10 %, significa que si la secuencia tiene una duración larga, la primera miniatura generada podría estar lejos del momento de inicio especificado. Intente seleccionar un valor razonable para step si se espera que la primera miniatura esté próxima al momento de inicio, o establezca el valor de range en 1 si solo se necesita una miniatura en el momento de inicio.
- **range**: la posición relativa al momento de inicio preestablecido de la transformación en el vídeo de entrada en la que se dejan de generar miniaturas. El valor puede estar en formato ISO 8601 (por ejemplo, PT5M30S para detenerse a los 5 minutos y 30 segundos desde el momento de inicio), ser un número de fotogramas (por ejemplo, 300 para detenerse en el fotograma 300 desde el fotograma del momento de inicio. Si este valor es 1, significa que solo se genera una miniatura en el momento de inicio) o ser un valor relativo a la duración de la secuencia (por ejemplo, 50 % para detenerse a la mitad de la duración de la secuencia desde el momento de inicio). El valor predeterminado es 100 %, lo que significa que debe detenerse al final de la secuencia.
- **layers**: una colección de capas de imagen de salida que el codificador va a generar.

```csharp

private static Transform EnsureTransformExists(IAzureMediaServicesClient client, string resourceGroupName, string accountName, string transformName)
{
    // Does a Transform already exist with the desired name? Assume that an existing Transform with the desired name
    // also uses the same recipe or Preset for processing content.
    Transform transform = client.Transforms.Get(resourceGroupName, accountName, transformName);

    if (transform == null)
    {
        // Create a new Transform Outputs array - this defines the set of outputs for the Transform
        TransformOutput[] outputs = new TransformOutput[]
        {
            // Create a new TransformOutput with a custom Standard Encoder Preset
            // This demonstrates how to create custom codec and layer output settings

          new TransformOutput(
                new StandardEncoderPreset(
                    codecs: new Codec[]
                    {
                        // Generate a set of PNG thumbnails
                        new PngImage(
                            start: "25%",
                            step: "25%",
                            range: "80%",
                            layers: new PngLayer[]{
                                new PngLayer(
                                    width: "50%",
                                    height: "50%"
                                )
                            }
                        )
                    },
                    // Specify the format for the output files for the thumbnails
                    formats: new Format[]
                    {
                        new PngFormat(
                            filenamePattern:"Thumbnail-{Basename}-{Index}{Extension}"
                        )
                    }
                ),
                onError: OnErrorType.StopProcessingJob,
                relativePriority: Priority.Normal
            )
        };

        string description = "A transform that includes thumbnails.";
        // Create the custom Transform with the outputs defined above
        transform = client.Transforms.CreateOrUpdate(resourceGroupName, accountName, transformName, outputs, description);
    }

    return transform;
}
```

## <a name="next-steps"></a>Pasos siguientes
[Generación de miniaturas con REST](media-services-generate-thumbnails-rest.md)
