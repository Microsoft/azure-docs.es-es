---
title: Adición de una capa de extrusión de polígono a un mapa | Microsoft Azure Maps
description: Cómo agregar una capa de extrusión de polígonos al SDK web de Microsoft Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 10/08/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen, devx-track-javascript
ms.openlocfilehash: 9d7444ec0acd17ae0ad08f4932c109c5cf0d3fb9
ms.sourcegitcommit: 07166a1ff8bd23f5e1c49d4fd12badbca5ebd19c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90087259"
---
# <a name="add-a-polygon-extrusion-layer-to-the-map"></a>Adición de una capa de extrusión de polígonos al mapa

En este artículo se muestra cómo usar la capa de extrusión de polígono para representar áreas de las geometrías de características `Polygon` y `MultiPolygon` como formas extruidas. El SDK web de Azure Maps admite la representación de geometrías de círculo tal como se define en el [esquema extendido de GeoJSON](extend-geojson.md#circle). Estos círculos se pueden transformar en polígonos cuando se representan en el mapa. Todas las geometrías de características se pueden actualizar fácilmente si se ajustan con la clase [atlas.Shape](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.shape).

## <a name="use-a-polygon-extrusion-layer"></a>Uso de una capa de extrusión de polígono

Conecte la [capa de extrusión de polígono](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.layer.polygonextrusionlayer) a un origen de datos. Luego, cárguelo en el mapa. La capa de extrusión de polígono representa las áreas de las características `Polygon` y `MultiPolygon` como formas extruidas. Las propiedades `height` y `base` de la capa de extrusión de polígono definen la distancia base desde el suelo y la altura de la forma extruida en **metros**. En el código siguiente se muestra cómo crear un polígono, agregarlo a un origen de datos y representarlo con la clase de la capa de extrusión de polígono.

> [!Note]
> El valor `base` definido en la capa de extrusión de polígono debe ser menor o igual al de `height`.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Polígono extruido" src="https://codepen.io/azuremaps/embed/wvvBpvE?height=265&theme-id=0&default-tab=js,result&editable=true" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
Consulte el Pen <a href='https://codepen.io/azuremaps/pen/wvvBpvE'>Polígono extruido</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.</iframe>


## <a name="add-data-driven-polygons"></a>Incorporación de polígonos controlados por datos

Un mapa de coropletas se puede representar mediante la capa de extrusión de polígono. Establezca las propiedades `height` y `fillColor` de la capa de extrusión en la medida de la variable estadística de las geometrías de las características `Polygon` y `MultiPolygon`. En el ejemplo de código siguiente se muestra un mapa de coropletas extruido de los Estados Unidos basado en la medida de la densidad de población por estado.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Mapa de coropletas extruido" src="https://codepen.io/azuremaps/embed/eYYYNox?height=265&theme-id=0&default-tab=result&editable=true" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
Consulte el Pen <a href='https://codepen.io/azuremaps/pen/eYYYNox'>Mapa de coropletas extruido</a> por Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="add-a-circle-to-the-map"></a>Adición de un círculo al mapa

Azure Maps usa una versión extendida del esquema GeoJSON que proporciona una definición para círculos como se indica [aquí](https://docs.microsoft.com/azure/azure-maps/extend-geojson#circle). Se puede representar un círculo en el mapa mediante la creación de una característica `point` que tiene una propiedad `subType` con un valor de `Circle` y una propiedad `Radius` numerada que representa el radio en **metros**. Por ejemplo:

```javascript
{
    "type": "Feature",
    "geometry": {
        "type": "Point",
        "coordinates": [-105.203135, 39.664087]
    },
    "properties": {
        "subType": "Circle",
        "radius": 1000
    }
} 
```

El SDK web de Azure Maps contiene estas características `Point` en características `Polygon` en segundo plano. Estas características `Point` se pueden representar en el mapa mediante la capa de extrusión de polígono, tal como se muestra en el ejemplo de código siguiente.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Polígono de espacio aéreo de dron" src="https://codepen.io/azuremaps/embed/zYYYrxo?height=265&theme-id=0&default-tab=js,result&editable=true" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
Consulte el Pen <a href='https://codepen.io/azuremaps/pen/zYYYrxo'>Polígono de espacio aéreo de dron</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="customize-a-polygon-extrusion-layer"></a>Personalización de una capa de extrusión de polígono

La capa de extrusión de polígono tiene varias opciones de estilo. Esta es una herramienta para probarlas.

<br/>

<iframe height='700' scrolling='no' title='PoogBRJ' src='//codepen.io/azuremaps/embed/PoogBRJ/?height=700&theme-id=0&default-tab=result' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen <a href='https://codepen.io/azuremaps/pen/PoogBRJ/'>PoogBRJ</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="next-steps"></a>Pasos siguientes

Más información sobre las clases y los métodos utilizados en este artículo:

> [!div class="nextstepaction"]
> [Polygon](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.data.polygon)

> [!div class="nextstepaction"]
> [Capa de extrusión de polígono](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.layer.polygonextrusionlayer)

Recursos adicionales:

> [!div class="nextstepaction"]
> [Extensión de la especificación GeoJSON de Azure Maps](extend-geojson.md#circle)
