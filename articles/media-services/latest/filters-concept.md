---
title: Definición de filtros en Azure Media Services
description: En este tema se describe cómo crear filtros para que su cliente pueda usarlos para el streaming de secciones específicas de una secuencia. Media Services crea manifiestos dinámicos para lograr este streaming selectivo.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: bb5561ced93c3f5a899c6e48fdab0f14e52914bb
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89291558"
---
# <a name="filters"></a>Filtros

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Al entregar su contenido a los clientes (eventos de streaming en vivo o vídeo bajo demanda), es posible que el cliente necesite más flexibilidad que la descrita en el archivo de manifiesto del recurso predeterminado. Azure Media Services ofrece [manifiestos dinámicos](filters-dynamic-manifest-overview.md) basados en filtros predefinidos. 

Los filtros son reglas del servidor que permitirán a los clientes a hacer cosas como: 

- Reproducir solo una sección de un vídeo (en lugar de reproducir todo el vídeo). Por ejemplo:
  - Reduzca el manifiesto para mostrar un clip secundario de un evento en directo ("filtrado de vídeos secundarios"), o bien
  - Recorte el inicio de un vídeo ("recorte de un vídeo").
- Entregue únicamente las representaciones y pistas de idioma especificadas que admite el dispositivo que se usa para la reproducción del contenido ("filtrado de representaciones"). 
- Ajuste la ventana de presentación (DVR) para ofrecer una longitud limitada de la ventana de DVR en el reproductor ("ventana de presentación de ajuste").

Media Services le permite crear **filtros de cuentas** y **filtros de recursos** para su contenido. Además, puede asociar los filtros creados previamente con un **localizador de streaming**.

## <a name="defining-filters"></a>Definición de filtros

Existen dos tipos de filtros: 

* [Filtros de cuenta](/rest/api/media/accountfilters) (global): se pueden aplicar a cualquier recurso de la cuenta de Azure Media Services; tienen una duración de la cuenta.
* [Filtros de recursos](/rest/api/media/assetfilters) (local): solo se pueden aplicar a un recurso con el que estaba asociado el filtro una vez creado; tienen una duración del recurso. 

Los tipos **Filtros de cuentas** y **Filtros de recursos** tienen exactamente las mismas propiedades para definir o describir el filtro. Excepto cuando se crea el **filtro de recursos**, deberá especificar el nombre del recurso con el que desea asociar el filtro.

En función del escenario, se decide qué tipo de filtro es el más adecuado (filtro de recursos o filtro de cuentas). Los filtros de cuenta son adecuados para los perfiles de dispositivos (filtrado de representaciones) donde los filtros de recursos podrían usarse para recortar un recurso específico.

Utilice las siguientes propiedades para describir los filtros. 

|Nombre|Descripción|
|---|---|
|firstQuality|La velocidad de bits de primera calidad del filtro.|
|presentationTimeRange|El intervalo de tiempo de la presentación. Esta propiedad se utiliza para filtrar los puntos de inicio y de fin del manifiesto, la duración de la ventana de presentación y la posición de inicio activa. <br/>Para más información, consulte [PresentationTimeRange](#presentationtimerange).|
|tracks|Las condiciones de selección de pistas. Para más información, consulte [Pistas](#tracks).|

### <a name="presentationtimerange"></a>presentationTimeRange

Utilice esta propiedad con los **filtros de recursos**. No se recomienda establecer la propiedad con los **filtros de cuenta**.

|Nombre|Descripción|
|---|---|
|**endTimestamp**|Se aplica al vídeo bajo demanda (VoD).<br/>Para la presentación de streaming en vivo, se ignora silenciosamente y se aplica cuando la presentación termina y la secuencia se convierte en VoD.<br/>Se trata de un valor largo que representa un punto final absoluto de la presentación, redondeado al siguiente inicio del grupo de imágenes más cercano. La unidad es la escala temporal, por lo que un valor endTimestamp de 1800000000 sería de 3 minutos.<br/>Use startTimestamp y endTimestamp para recortar los fragmentos que estarán en la lista de reproducción (manifiesto).<br/>Por ejemplo, startTimestamp=40000000 y endTimestamp=100000000 usando la escala temporal predeterminada generarán una lista de reproducción que contiene fragmentos de entre 4 y 10 segundos de la presentación de VoD. Si un fragmento sobrepasa el límite, se incluirá todo el fragmento en el manifiesto.|
|**forceEndTimestamp**|Solo se aplica a streaming en vivo.<br/>Indica si la propiedad endTimestamp debe estar presente. Si el valor true, se debe especificar el valor de endTimestamp o se devolverá un código de solicitud incorrecta.<br/>Valores permitidos: false, true.|
|**liveBackoffDuration**|Solo se aplica a streaming en vivo.<br/> Este valor define la última posición en directo a la que puede buscar un cliente.<br/>Con esta propiedad, se puede retrasar la posición de reproducción en directo y crear un búfer en el servidor para los reproductores.<br/>La unidad de esta propiedad es la escala temporal (consultar a continuación).<br/>La duración de retroceso en directo máxima es de 300 segundos (3000000000).<br/>Por ejemplo, un valor de 2000000000 significa que el último contenido disponible tiene 20 segundos de retraso con respecto al borde en directo.|
|**presentationWindowDuration**|Solo se aplica a streaming en vivo.<br/>Use presentationWindowDuration para aplicar una ventana deslizante de fragmentos a incluir en una lista de reproducción.<br/>La unidad de esta propiedad es la escala temporal (consultar a continuación).<br/>Por ejemplo, configure la presentaciónWindowDuration=1200000000 para aplicar una ventana deslizante de dos minutos. Los medios que se encuentren a menos de dos minutos del borde en directo se incluirán en la lista de reproducción. Si un fragmento sobrepasa el límite, se incluirá todo el fragmento en la lista de reproducción. La duración de la ventana de presentación mínima es de 60 segundos.|
|**startTimestamp**|Se aplica a vídeo bajo demanda (VoD) o a streaming en vivo.<br/>Se trata de un valor largo que representa un punto inicial absoluto de la transmisión. El valor redondea al siguiente inicio del grupo de imágenes más cercano. La unidad es la escala temporal, por lo que un valor startTimestamp de 150000000 sería de 15 segundos.<br/>Use startTimestamp y endTimestampp para recortar los fragmentos que estarán en la lista de reproducción (manifiesto).<br/>Por ejemplo, startTimestamp=40000000 y endTimestamp=100000000 usando la escala temporal predeterminada generarán una lista de reproducción que contiene fragmentos de entre 4 y 10 segundos de la presentación de VoD. Si un fragmento sobrepasa el límite, se incluirá todo el fragmento en el manifiesto.|
|**timescale**|Se aplica a todas las marcas de tiempo y las duraciones en el intervalo de tiempo de una presentación y se especifica como el número de incrementos en un segundo.<br/>El valor predeterminado es 10000000, diez millones de incrementos en un segundo, donde cada incremento tendría una duración de 100 nanosegundos.<br/>Por ejemplo, si quiere establecer un valor de startTimestamp en 30 segundos, debería usar un valor de 300000000 cuando use la escala temporal predeterminada.|

### <a name="tracks"></a>Pistas

Se especifica una lista de condiciones de propiedad de pista de filtro (FilterTrackPropertyConditions) en función de las pistas de la secuencia (streaming en vivo o vídeo bajo demanda) que deben incluirse en el manifiesto creado dinámicamente. Los filtros se combinan mediante una operación lógica **AND** y **OR**.

Las condiciones de la propiedad de la pista del filtro describen tipos de pista, valores (descritos en la siguiente tabla) y operaciones (Equal, NotEqual). 

|Nombre|Descripción|
|---|---|
|**Bitrate**|Utilice la velocidad de bits de la pista para filtrar.<br/><br/>El valor recomendado es un rango de velocidades de bits, en bits por segundo. Por ejemplo, "0-2427000".<br/><br/>Nota: aunque puede utilizar un valor de velocidad de bits específico, como 250000 (bits por segundo), no se recomienda este enfoque, ya que las velocidades de bits exactas pueden fluctuar de un recurso a otro.|
|**FourCC**|Utilice el valor FourCC de la pista para filtrar.<br/><br/>El valor es el primer elemento de formato de códecs, según se especifica en [RFC 6381](https://tools.ietf.org/html/rfc6381). Actualmente se admiten los siguientes códecs: <br/>Para vídeo: "avc1", "hev1", "hvc1"<br/>Para audio: "mp4a", "ec-3"<br/><br/>Para determinar los valores de FourCC para las pistas de un recurso, obtenga y examine el archivo de manifiesto.|
|**Lenguaje**|Utilice el idioma de la pista para filtrar.<br/><br/>El valor es la etiqueta de un idioma que desea incluir, como se especifica en RFC 5646. Por ejemplo, "en".|
|**Nombre**|Utilice el nombre de la pista para filtrar.|
|**Tipo**|Utilice el tipo de la pista para filtrar.<br/><br/>Se permiten los siguientes valores: "video", "audio" o "text".|

### <a name="example"></a>Ejemplo

En el ejemplo siguiente se define un filtro de streaming en vivo: 

```json
{
  "properties": {
    "presentationTimeRange": {
      "startTimestamp": 0,
      "endTimestamp": 170000000,
      "presentationWindowDuration": 9223372036854776000,
      "liveBackoffDuration": 0,
      "timescale": 10000000,
      "forceEndTimestamp": false
    },
    "firstQuality": {
      "bitrate": 128000
    },
    "tracks": [
      {
        "trackSelections": [
          {
            "property": "Type",
            "operation": "Equal",
            "value": "Audio"
          },
          {
            "property": "Language",
            "operation": "NotEqual",
            "value": "en"
          },
          {
            "property": "FourCC",
            "operation": "NotEqual",
            "value": "EC-3"
          }
        ]
      },
      {
        "trackSelections": [
          {
            "property": "Type",
            "operation": "Equal",
            "value": "Video"
          },
          {
            "property": "Bitrate",
            "operation": "Equal",
            "value": "3000000-5000000"
          }
        ]
      }
    ]
  }
}
```

## <a name="associating-filters-with-streaming-locator"></a>Asociación de filtros con localizadores de streaming

Ahora puede especificar una lista de los [filtros de recursos o de cuentas](filters-concept.md) en el [localizador de streaming](/rest/api/media/streaminglocators/create#request-body). El [empaquetador dinámico](dynamic-packaging-overview.md) se aplica a esta lista de filtros junto con los que el cliente especifica en la dirección URL. Esta combinación se genera un [manifiesto dinámico](filters-dynamic-manifest-overview.md), que se basa en los filtros de la dirección URL y en los filtros que especifique en el localizador de streaming. 

Consulte los siguientes ejemplos:

* [Associate filters with Streaming Locator - .NET](filters-dynamic-manifest-dotnet-howto.md#associate-filters-with-streaming-locator) (Asociación de filtros con el localizador de streaming: .NET)
* [Associate filters with Streaming Locator - CLI](filters-dynamic-manifest-cli-howto.md#associate-filters-with-streaming-locator) (Asociación de filtros con el localizador de streaming: CLI)

## <a name="updating-filters"></a>Actualización de los filtros
 
Si bien los **localizadores de streaming** no se pueden actualizar, los filtros sí. 

No se recomienda actualizar la definición de los filtros asociados con un **localizador de streaming** publicado activamente, especialmente si la red CDN está habilitada. Las redes CDN y los servidores de streaming pueden tener cachés internas que pueden generar que se devuelvan datos en caché obsoletos. 

Si es necesario modificar la definición de los filtros, considere crear un filtro y agregarlo a la dirección URL del **localizador de streaming** o publicar un nuevo **Streaming Locator** que haga referencia directamente al filtro.

## <a name="next-steps"></a>Pasos siguientes

Los artículos siguientes muestran cómo crear filtros mediante programación.  

- [Creación de filtros con las API REST](filters-dynamic-manifest-rest-howto.md)
- [Creación de filtros con .NET](filters-dynamic-manifest-dotnet-howto.md)
- [Creación de filtros con la CLI](filters-dynamic-manifest-cli-howto.md)
