---
title: Texturas
description: Flujo de trabajo de recursos de textura
author: florianborn71
ms.author: flborn
ms.date: 02/05/2020
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.openlocfilehash: 20c2951b601690beb9ec64040cf650bb5208d0e4
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88997717"
---
# <a name="textures"></a>Texturas

Las texturas son un recurso compartido [inmutable](../concepts/lifetime.md). Las texturas se pueden cargar desde [Blob Storage](../how-tos/conversion/blob-storage.md) y se pueden aplicar directamente a los modelos, como se muestra en el [Tutorial: Cambio del entorno y los materiales](../tutorials/unity/materials-lighting-effects/materials-lighting-effects.md). Sin embargo, normalmente, las texturas formarán parte de un [modelo convertido](../how-tos/conversion/model-conversion.md), donde se hace referencia a ellas por sus [materiales](materials.md).

## <a name="texture-types"></a>Tipos de textura

Los diferentes tipos de textura tienen casos de uso diferentes:

* Las **texturas 2D** se usan principalmente en [materiales](materials.md).
* Los **mapas de cubos** se pueden usar para el [cielo](../overview/features/sky.md).

## <a name="supported-texture-formats"></a>Formatos de textura admitidos

Todas las texturas dadas a ARR deben estar en [formato DDS](https://en.wikipedia.org/wiki/DirectDraw_Surface). Preferiblemente con los mapas MIP y compresión de textura. Consulte la [herramienta de línea de comandos TexConv](../resources/tools/tex-conv.md) si quiere automatizar el proceso de conversión.

## <a name="loading-textures"></a>Carga de texturas

Al cargar una textura, tiene que especificar el tipo esperado. Si el tipo no coincide, se produce un error en la carga de la textura.
La carga de una textura con el mismo URI dos veces devolverá el mismo objeto de textura, ya que se trata de un [recurso compartido](../concepts/lifetime.md).

De forma similar a la carga de modelos, hay dos variantes a la hora de direccionar un recurso de textura en el almacenamiento de blobs de origen:

* El recurso de textura se puede direccionar por su URI de SAS. La función de carga pertinente es `LoadTextureFromSASAsync` con el parámetro `LoadTextureFromSASParams`. Use también esta variante al cargar [texturas integradas](../overview/features/sky.md#built-in-environment-maps).
* La textura se puede direccionar directamente mediante parámetros de Blob Storage, en caso de que [Blob Storage esté vinculado a la cuenta](../how-tos/create-an-account.md#link-storage-accounts). En este caso, la función de carga pertinente es `LoadTextureAsync` con el parámetro `LoadTextureParams`.

En el código de ejemplo siguiente se muestra cómo cargar una textura a través de su URI de SAS (o textura integrada); tenga en cuenta que solo la función o el parámetro de carga se diferencian en el otro caso:

```cs
LoadTextureAsync _textureLoad = null;
void LoadMyTexture(AzureSession session, string textureUri)
{
    _textureLoad = session.Actions.LoadTextureFromSASAsync(new LoadTextureFromSASParams(textureUri, TextureType.Texture2D));
    _textureLoad.Completed +=
        (LoadTextureAsync res) =>
        {
            if (res.IsRanToCompletion)
            {
                //use res.Result
            }
            else
            {
                System.Console.WriteLine("Texture loading failed!");
            }
            _textureLoad = null;
        };
}
```

```cpp
void LoadMyTexture(ApiHandle<AzureSession> session, std::string textureUri)
{
    LoadTextureFromSASParams params;
    params.TextureType = TextureType::Texture2D;
    params.TextureUrl = std::move(textureUri);
    ApiHandle<LoadTextureAsync> textureLoad = *session->Actions()->LoadTextureFromSASAsync(params);
    textureLoad->Completed([](ApiHandle<LoadTextureAsync> res)
    {
        if (res->GetIsRanToCompletion())
        {
            //use res->Result()
        }
        else
        {
            printf("Texture loading failed!");
        }
    });
}
```


Dependiendo del tipo de textura que se supone que se va a usar, puede haber restricciones en cuanto al tipo y el contenido. Por ejemplo, el mapa de rugosidad de un [material PBR](../overview/features/pbr-materials.md) debe ser una escala de grises.

> [!CAUTION]
> Todas las funciones *asincrónicas* en ARR devuelven objetos de operación asincrónica. Debe almacenar una referencia a esos objetos hasta que finalice la operación. De lo contrario, el recolector de elementos no utilizados de C# podría eliminar la operación con anterioridad y que nunca finalice. En el código de ejemplo anterior, la variable miembro "_textureLoad" se usa para contener una referencia hasta que llega el evento *completado*.

## <a name="next-steps"></a>Pasos siguientes

* [Materiales](materials.md)
* [Cielo](../overview/features/sky.md)
