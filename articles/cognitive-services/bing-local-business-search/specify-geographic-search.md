---
title: Uso de límites geográficos para filtrar los resultados de Bing Local Business Search API | Microsoft Docs
titleSuffix: Azure Cognitive Services
description: Use este artículo para obtener información sobre cómo filtrar los resultados de la búsqueda de Bing Local Business Search API.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: article
ms.date: 11/01/2018
ms.author: rosh, v-gedod
ms.openlocfilehash: e87ef96184974d10f99028be13399d1422a1c774
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65796591"
---
# <a name="use-geographic-boundaries-to-filter-results-from-the-bing-local-business-search-api"></a>Uso de límites geográficos para filtrar los resultados de Bing Local Business Search API

Bing Local Business Search API le permite establecer límites en la zona geográfica específica que le gustaría buscar mediante los parámetros de consulta `localCircularView` o `localMapView`. Asegúrese de usar solo un parámetro en las consultas. 

Si un término de búsqueda contiene una ubicación geográfica explícita, Bing Local Business API la usará automáticamente para establecer límites para los resultados de la búsqueda. Por ejemplo, si es el término de búsqueda es `sailing in San Diego`, `San Diego` se usará como la ubicación y cualquier otra ubicación especificada en los parámetros de consulta o en los encabezados de usuario se ignorará. 

Si no se detecta una ubicación geográfica en el término de búsqueda y no se especifica ninguna ubicación geográfica mediante los parámetros de consulta, Bing Local Business Search API intentará determinar la ubicación a partir de los encabezados `X-Search-ClientIP` o `X-Search-Location` de la solicitud. Si no se especifica ningún encabezado, la API determinará la ubicación a partir de la dirección IP del cliente de la solicitud o las coordenadas GPS en dispositivos móviles.

## <a name="localcircularview"></a>localCircularView

El parámetro `localCircularView` crea un área geográfica circular en torno a un conjunto de coordenadas de latitud y longitud definida por un radio. Cuando se usa este parámetro, las respuestas de Bing Local Business Search API solo incluirán las ubicaciones dentro de este círculo, a diferencia del parámetro `localMapView`, que puede incluir ubicaciones ligeramente fuera del área de búsqueda.

Para especificar un área de búsqueda geográfica circular, elija una latitud y una longitud para que actúen como el centro del círculo y un radio en metros. Este parámetro se puede anexar a una cadena de consulta, por ejemplo: `q=Restaurants&localCircularView=47.6421,-122.13715,5000`.

Consulta completa:

```
https://api.cognitive.microsoft.com/bing/v7.0/localbusinesses/search?q=restaurant&localCircularView=47.6421,-122.13715,5000&appid=0123456789ABCDEF&mkt=en-us&form=monitr
```

## <a name="localmapview"></a>localMapView

El parámetro `localMapView` especifica un área geográfica rectangular para la búsqueda, con dos conjuntos de coordenadas para especificar sus esquinas sudeste y noroeste. Cuando se usa este parámetro, las respuestas de Bing Local Business Search API pueden incluir ubicaciones dentro y ligeramente fuera del área especificada, a diferencia del parámetro `localCircularView`, que solo incluye ubicaciones dentro del área de búsqueda.

Para especificar un área de búsqueda rectangular, elija dos conjuntos de coordenadas de latitud y longitud para que actúen como las esquinas sudeste y noroeste del límite. Asegúrese de definir las coordenadas sudeste en primer lugar, como en el ejemplo siguiente: `localMapView=47.619987,-122.181671,47.6421,-122.13715`.

Consulta completa:

```
https://api.cognitive.microsoft.com/bing/v7.0/localbusinesses/search?q=restaurant&localMapView=47.619987,-122.181671,47.6421,-122.13715&appid=0123456789ABCDEF&mkt=en-us&form=monitr
```

## <a name="next-steps"></a>Pasos siguientes
- [Guía de inicio rápido de Local Business Search para Java](quickstarts/local-search-java-quickstart.md)
- [Guía de inicio rápido de Local Business Search para C#](quickstarts/local-quickstart.md)
- [Guía de inicio rápido de Local Business Search para Node](quickstarts/local-search-node-quickstart.md)
- [Guía de inicio rápido de Local Business Search para Python](quickstarts/local-search-python-quickstart.md)
