---
title: Presentación de los resultados de la búsqueda en un mapa | Microsoft Azure Maps
description: En este artículo, obtendrá información sobre cómo realizar una solicitud de búsqueda mediante el SDK web de Microsoft Azure Maps y mostrar los resultados en el mapa.
author: anastasia-ms
ms.author: v-stharr
ms.date: 07/29/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen
ms.openlocfilehash: 678781ff4bade7484f28cc9ab14d960f2b24c323
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87130297"
---
# <a name="show-search-results-on-the-map"></a>Presentación de los resultados de la búsqueda en el mapa

En este artículo se describe cómo buscar ubicaciones de interés y mostrar los resultados de búsqueda en el mapa.

Hay dos maneras de buscar una ubicación de interés. Una consiste en usar un módulo de servicio para realizar una solicitud de búsqueda. La otra consiste en realizar una solicitud de búsqueda en la [Azure Maps Fuzzy Search API](https://docs.microsoft.com/rest/api/maps/search/getsearchfuzzy) a través de la [API de captura](https://fetch.spec.whatwg.org/). Ambos métodos se describen a continuación.

## <a name="make-a-search-request-via-service-module"></a>Realización de una solicitud de búsqueda a través del módulo de servicio

<iframe height='500' scrolling='no' title='Presentación de los resultados de búsqueda en un mapa (módulo de servicio)' src='//codepen.io/azuremaps/embed/zLdYEB/?height=265&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen <a href='https://codepen.io/azuremaps/pen/zLdYEB/'>Show search results on a map (Service Module)</a> (Presentación de los resultados de búsqueda en un mapa [módulo de servicio]) de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

En el código anterior, el primer bloque construye un objeto de mapa y establece el mecanismo de autenticación para usar el token de acceso. Puede consultar [Creación de un mapa](./map-create.md) para obtener instrucciones.

El segundo bloque de código crea un elemento `TokenCredential` para autenticar las solicitudes HTTP en Azure Maps con el token de acceso. A continuación, pasa `TokenCredential` a `atlas.service.MapsURL.newPipeline()` y crea una instancia de [canalización](https://docs.microsoft.com/javascript/api/azure-maps-rest/atlas.service.pipeline?view=azure-maps-typescript-latest). `searchURL` representa una dirección URL para las operaciones [Search](https://docs.microsoft.com/rest/api/maps/search) de Azure Maps.

El tercer bloque de código crea un objeto de origen de datos con la clase [DataSource](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.source.datasource?view=azure-iot-typescript-latest) y le agrega los resultados de la búsqueda. Una [capa de símbolos](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.layer.symbollayer?view=azure-iot-typescript-latest) usa texto o iconos para representar los datos basados en puntos encapsulados en [DataSource](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.source.datasource?view=azure-iot-typescript-latest) como símbolos en el mapa.  A continuación, se crea una capa de símbolos. El origen de datos se agrega a la capa de símbolos y, después, esta se agrega al mapa.

El cuarto bloque de código usa el método [SearchFuzzy](/javascript/api/azure-maps-rest/atlas.service.models.searchgetsearchfuzzyoptionalparams) en el [módulo de servicio](how-to-use-services-module.md). Permite realizar una búsqueda de texto de forma libre a través de la [Get Search Fuzzy rest API](https://docs.microsoft.com/rest/api/maps/search/getsearchfuzzy) para buscar el punto de interés. Las solicitudes Get Search Fuzzy API puede administrar cualquier combinación de entradas aproximadas. Después se extrae una colección de características GeoJSON de la respuesta con el método `geojson.getFeatures()` y se agregan al origen de datos, lo cual provoca que los datos se representen automáticamente en el mapa mediante la capa de símbolos.

El último bloque de código ajusta los límites de cámara para el mapa mediante la propiedad [setCamera](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#setcamera-cameraoptions---cameraboundsoptions---animationoptions-) del mapa.

La solicitud de búsqueda, el origen de datos, la capa de símbolos y los límites de cámara están dentro del [agente de escucha de eventos](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#events) del mapa. Queremos asegurarnos de que los resultados se muestran una vez que el mapa se carga por completo.


## <a name="make-a-search-request-via-fetch-api"></a>Realización de una solicitud de búsqueda a través de la API de captura

<iframe height='500' scrolling='no' title='Presentación de los resultados de la búsqueda en un mapa' src='//codepen.io/azuremaps/embed/KQbaeM/?height=265&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen <a href='https://codepen.io/azuremaps/pen/KQbaeM/'>Presentación de los resultados de la búsqueda en un mapa</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

En el código anterior, el primer bloque de código construye un objeto mapa. Establece el mecanismo de autenticación para usar el token de acceso. Puede consultar [Creación de un mapa](./map-create.md) para obtener instrucciones.

El segundo bloque de código crea una dirección URL para realizar una solicitud de búsqueda. También crea dos matrices para almacenar límites y chinchetas para los resultados de búsqueda.

El tercer bloque de código usa la [API de captura](https://fetch.spec.whatwg.org/). La [API de captura](https://fetch.spec.whatwg.org/) se usa para hacer una solicitud a [Azure Maps Fuzzy search API](https://docs.microsoft.com/rest/api/maps/search/getsearchfuzzy) para buscar los puntos de interés. Fuzzy Search API puede administrar cualquier combinación de entradas aproximadas. A continuación, administra y analiza la respuesta de búsqueda y agrega las chinchetas de resultados a la matriz searchPins.

El cuarto bloque de código crea un objeto de origen de datos con la clase [DataSource](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.source.datasource?view=azure-iot-typescript-latest). En el código, se agregan los resultados de la búsqueda al objeto de origen. Una [capa de símbolos](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.layer.symbollayer?view=azure-iot-typescript-latest) usa texto o iconos para representar los datos basados en puntos encapsulados en [DataSource](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.source.datasource?view=azure-iot-typescript-latest) como símbolos en el mapa. A continuación, se crea una capa de símbolos. El origen de datos se agrega a la capa de símbolos y, después, esta se agrega al mapa.

El último bloque de código crea un objeto [BoundingBox](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.data.boundingbox?view=azure-iot-typescript-latest). Usa la matriz de resultados y, a continuación, ajusta los límites de cámara para el mapa mediante la configuración [setCamera](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#setcamera-cameraoptions---cameraboundsoptions---animationoptions-) del mapa. A continuación, muestra las chinchetas de resultados.

La solicitud de búsqueda, el origen de datos, la capa de símbolos y los límites de cámara se establecen en el [agente de escucha de eventos](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest#events) del mapa para asegurar que los resultados se muestran una vez que el mapa se carga completamente.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Procedimientos recomendados de uso del servicio de búsqueda](how-to-use-best-practices-for-search.md)

Más información sobre la **búsqueda aproximada**:

> [!div class="nextstepaction"]
> [Fuzzy Search API de Azure Maps](https://docs.microsoft.com/rest/api/maps/search/getsearchfuzzy)

Más información sobre las clases y los métodos utilizados en este artículo:

> [!div class="nextstepaction"]
> [Map](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-iot-typescript-latest)

Consulte los siguientes artículos para obtener ejemplos de código completo:

> [!div class="nextstepaction"]
> [Obtención de información de una coordenada](map-get-information-from-coordinate.md)
<!-- Comment added to suppress false positive warning -->
> [!div class="nextstepaction"]
> [Presentación de indicaciones de ruta de A a B](map-route.md)
