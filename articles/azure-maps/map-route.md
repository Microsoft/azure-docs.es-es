---
title: Presentación de indicaciones de ruta en un mapa | Microsoft Azure Maps
description: En este artículo aprenderá cómo mostrar las indicaciones entre dos ubicaciones en un mapa mediante el SDK web de Microsoft Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 07/29/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.custom: codepen
ms.openlocfilehash: 54a196cc8323e676dfb054a5fad260302833fa53
ms.sourcegitcommit: 07166a1ff8bd23f5e1c49d4fd12badbca5ebd19c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90090693"
---
# <a name="show-directions-from-a-to-b"></a>Presentación de indicaciones de ruta de A a B

Este artículo muestra cómo realizar una solicitud de ruta y mostrar la ruta en el mapa.

Hay dos formas de hacerlo. La primera es consultar la [Route API de Azure Maps](https://docs.microsoft.com/rest/api/maps/route/getroutedirections) a través de un módulo de servicio. La segunda manera es usar la [API de captura](https://fetch.spec.whatwg.org/) para realizar una solicitud de búsqueda en la [API de ruta de Azure Maps](https://docs.microsoft.com/rest/api/maps/route/getroutedirections). Ambos métodos se describen a continuación.

## <a name="query-the-route-via-service-module"></a>Consultas a la ruta a través del módulo de servicio

<iframe height='500' scrolling='no' title='Presentación de indicaciones de ruta de A a B en un mapa (módulo de servicio)' src='//codepen.io/azuremaps/embed/RBZbep/?height=265&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen <a href='https://codepen.io/azuremaps/pen/RBZbep/'>Presentación de indicaciones de ruta de A a B en un mapa</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

En el código anterior, el primer bloque construye un objeto de mapa y establece el mecanismo de autenticación para usar el token de acceso. Puede consultar [Creación de un mapa](./map-create.md) para obtener instrucciones.

El segundo bloque de código crea un elemento `TokenCredential` para autenticar las solicitudes HTTP en Azure Maps con el token de acceso. A continuación, pasa `TokenCredential` a `atlas.service.MapsURL.newPipeline()` y crea una instancia de [canalización](https://docs.microsoft.com/javascript/api/azure-maps-rest/atlas.service.pipeline). `routeURL` representa una dirección URL para las operaciones [Route](https://docs.microsoft.com/rest/api/maps/route) de Azure Maps.

El tercer bloque de código crea y agrega un objeto [DataSource](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.source.datasource) al mapa.

El cuarto bloque de código crea objetos [points](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.data.point) de inicio y final y los agrega al objeto dataSource.

Una línea es un objeto [Feature](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.data.feature) de LineString. Una clase [LineLayer](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.layer.linelayer) presenta objetos de línea encapsulados en [DataSource](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.source.datasource) como líneas en el mapa. El cuarto bloque de código crea una capa de línea y la agrega al mapa. Puede consultar las propiedades de una capa de líneas en [LinestringLayerOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.linelayeroptions).

Una [capa de símbolos](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.layer.symbollayer) usa texto o iconos para representar los datos basados en puntos encapsulados en [DataSource](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.source.datasource). Los textos o los iconos se representan como símbolos en el mapa. El quinto bloque de código crea una capa de símbolos y la agrega al mapa.

El sexto bloque de código consulta el servicio de enrutamiento de Azure Maps, que forma parte del [módulo de servicio](how-to-use-services-module.md). El método [calculateRouteDirections](https://docs.microsoft.com/javascript/api/azure-maps-rest/atlas.service.routeurl#methods) de RouteURL se usa para obtener una ruta entre los puntos de inicio y final. Se extrae una colección de características de GeoJSON de la respuesta con el método `geojson.getFeatures()` y se agregan al origen de datos. A continuación, se procesa la respuesta como una ruta en el mapa. Para obtener más información acerca de cómo agregar una línea al mapa, consulte [Adición de una línea al mapa](map-add-line-layer.md).

El último bloque de código establece los límites del mapa con la propiedad [setCamera](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map#setcamera-cameraoptions---cameraboundsoptions---animationoptions-) del mapa.

La consulta de ruta, el origen de datos, el símbolo, las capas de líneas y los límites de cámara se crean en el [agente de escucha de eventos](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map#events). Esta estructura de código garantiza que los resultados se muestran solo una vez cargado el mapa por completo.

## <a name="query-the-route-via-fetch-api"></a>Consultar la ruta a través de la API de captura

<iframe height='500' scrolling='no' title='Presentación de indicaciones de ruta de A a B en un mapa' src='//codepen.io/azuremaps/embed/zRyNmP/?height=469&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen <a href='https://codepen.io/azuremaps/pen/zRyNmP/'>Presentación de indicaciones de ruta de A a B en un mapa</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

En el código anterior, el primer bloque de código construye un objeto de mapa y establece el mecanismo de autenticación para usar el token de acceso. Puede consultar [Creación de un mapa](./map-create.md) para obtener instrucciones.

El segundo bloque de código crea y agrega un objeto [DataSource](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.source.datasource) al mapa.

El tercer bloque de código crea los puntos de inicio y de destino para la ruta y los agrega al origen de datos. Puede consultar [Adición de un anclaje en el mapa](map-add-pin.md) para obtener instrucciones acerca del uso de [addPins](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map).

Una clase [LineLayer](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.layer.linelayer) presenta objetos de línea encapsulados en [DataSource](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.source.datasource) como líneas en el mapa. El cuarto bloque de código crea una capa de línea y la agrega al mapa. Puede consultar las propiedades de una capa de líneas en [LineLayerOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.linelayeroptions).

Una [capa de símbolos](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.layer.symbollayer) usa texto o iconos para representar los datos basados en puntos encapsulados en [DataSource](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.source.datasource) como símbolos en el mapa. El quinto bloque de código crea una capa de símbolos y la agrega al mapa. Consulte las propiedades de una capa de símbolos en [SymbolLayerOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.symbollayeroptions).

El bloque de código siguiente crea los puntos `SouthWest` y `NorthEast` desde los puntos de inicio y de destino y establece los límites del mapa con la propiedad [setCamera](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map#setcamera-cameraoptions---cameraboundsoptions---animationoptions-) del mapa.

El último bloque de código utiliza la [API de captura](https://fetch.spec.whatwg.org/) para realizar una solicitud de búsqueda en la [API de ruta de Azure Maps](https://docs.microsoft.com/rest/api/maps/route/getroutedirections). A continuación, se analiza la respuesta. Si la respuesta fue correcta, se utiliza la información sobre la latitud y longitud para crear una línea de una matriz mediante la conexión de esos puntos. Los datos de línea se agregan a continuación al origen de datos para representar la ruta en el mapa. Puede consultar [Adición de una línea en el mapa](map-add-line-layer.md) para obtener instrucciones.

La consulta de ruta, el origen de datos, el símbolo, las capas de líneas y los límites de cámara se crean en el [agente de escucha de eventos](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map#events). Como ya se indicó, queremos asegurarnos de que los resultados se muestran una vez que el mapa se carga por completo.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Procedimientos recomendados de uso del servicio de cálculo de ruta](how-to-use-best-practices-for-search.md)

Más información sobre las clases y los métodos utilizados en este artículo:

> [!div class="nextstepaction"]
> [Map](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map)

Consulte los siguientes artículos para obtener ejemplos de código completo:

> [!div class="nextstepaction"]
> [Visualización del tráfico en el mapa](./map-show-traffic.md)

> [!div class="nextstepaction"]
> [Interacción con el mapa: eventos del mouse](./map-events.md)
