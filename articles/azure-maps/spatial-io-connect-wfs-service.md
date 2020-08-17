---
title: Conexión a un servicio Web Feature Service (WFS) | Microsoft Azure Maps
description: Obtenga información sobre cómo conectarse a un servicio WFS y consultar después el servicio WFS con el SDK web de Azure Maps y el módulo de E/S espacial.
author: anastasia-ms
ms.author: v-stharr
ms.date: 03/03/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: devx-track-javascript
ms.openlocfilehash: 890bf60ac794740716817094a60f2b4cb4ba5835
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87282796"
---
# <a name="connect-to-a-wfs-service"></a>Conexión a un servicio WFS

Web Feature Service (WFS) es un servicio web para consultar datos espaciales que tiene una API estandarizada definida por Open Geospatial Consortium (OGC). La clase `WfsClient` del módulo de E/S espacial permite a los desarrolladores conectarse a un servicio WFS y consultar los datos del servicio.

La clase `WfsClient` admite las características siguientes:

- Versiones compatibles: `1.0.0`, `1.1.0` y `2.0.0`
- Operadores de filtro admitidos: comparaciones binarias, lógicos, matemáticos, de valor y `bbox`.
- Las solicitudes solo se realizan mediante `HTTP GET`.
- Operaciones compatibles:

    | Operación | Descripción |
    | :-- | :-- |
    | GetCapabilities | Genera un documento de metadatos con operaciones y parámetros de WFS válidos |
    | GetFeature | Devuelve una selección de características de un origen de datos |
    | DescribeFeatureType | Devuelve los tipos de características admitidos |

## <a name="using-the-wfs-client"></a>Uso del cliente de WFS

La clase `atlas.io.ogc.WfsClient` del módulo de E/S espacial facilita la consulta de un servicio WFS y la conversión de las respuestas en objetos GeoJSON. Este objeto GeoJSON se puede usar para otros fines de asignación.

El código siguiente consulta un servicio WFS y representa las características devueltas en el mapa.

<br/>

<iframe height='700' scrolling='no' title='Ejemplo de WFS simple' src='//codepen.io/azuremaps/embed/MWwvVYY/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el lápiz <a href='https://codepen.io/azuremaps/pen/MWwvVYY/'>Ejemplo de WFS simple</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="supported-filters"></a>Filtros admitidos

La especificación del estándar WFS usa filtros OGC. Los siguientes filtros son compatibles con el cliente WFS, suponiendo que el servicio al que se llama también admita estos filtros. Las cadenas de filtro personalizado se pueden pasar en la clase `CustomFilter`.

**Operadores lógicos**

- `And`
- `Or`
- `Not`

**Operadores de valor**

- `GmlObjectId`
- `ResourceId`

**Operadores matemáticos**

- `Add`
- `Sub`
- `Mul`
- `Div`

**Operadores de comparación**

- `PropertyIsEqualTo`
- `PropertyIsNotEqualTo`
- `PropertyIsLessThan`
- `PropertyIsGreaterThan`
- `PropertyIsLessThanOrEqualTo`
- `PropertyIsGreaterThanOrEqualTo`
- `PropertyIsLike`
- `PropertyIsNull`
- `PropertyIsNil`
- `PropertyIsBetween`

En el código siguiente se muestra el uso de distintos filtros con el cliente de WFS.

<br/>

<iframe height='500' scrolling='no' title= 'Ejemplos de filtros de WFS' src='//codepen.io/azuremaps/embed/NWqvYrV/?height=500&theme-id=0&default-tab=result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Vea el lápiz <a href='https://codepen.io/azuremaps/pen/NWqvYrV/'>Ejemplos de filtros de WFS</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="wfs-service-explorer"></a>Explorador de servicios WFS

En el código siguiente se usa el cliente de WFS para explorar los servicios WFS. Seleccione una capa de tipo de propiedad dentro del servicio y vea la leyenda asociada.

<br/>

<iframe height='700' style='width: 100%;' scrolling='no' title= 'Explorador de servicios WFS' src='//codepen.io/azuremaps/embed/bGdrvmG/?height=700&theme-id=0&default-tab=result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Vea el lápiz <a href='https://codepen.io/azuremaps/pen/bGdrvmG/'>Explorador de servicios WFS</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

Para acceder a los servicios de WFS hospedados en puntos de conexión habilitados sin CORS, se puede pasar un servicio de proxy habilitado con CORS a la opción `proxyService` del cliente de WFS, como se muestra a continuación. 

```JavaScript
//Create the WFS client to access the service and use the proxy service settings
client = new atlas.io.ogc.WfsClient({
    url: url,
    proxyService: window.location.origin + '/YourCorsEnabledProxyService.ashx?url='
});
```

## <a name="next-steps"></a>Pasos siguientes

Más información sobre las clases y los métodos utilizados en este artículo:

> [!div class="nextstepaction"]
> [WfsClient](https://docs.microsoft.com/JavaScript/api/azure-maps-spatial-io/atlas.io.ogc.wfsclient)

> [!div class="nextstepaction"]
> [WfsServiceOptions](https://docs.microsoft.com/JavaScript/api/azure-maps-spatial-io/atlas.wfsserviceoptions)

Para obtener más ejemplos de código para agregar a los mapas:

> [!div class="nextstepaction"]
> [Aprovechamiento de las operaciones básicas](spatial-io-core-operations.md)

> [!div class="nextstepaction"]
> [Detalles de formatos de datos admitidos](spatial-io-supported-data-format-details.md)
