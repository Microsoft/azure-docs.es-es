---
title: Adición de una capa de polígono a un mapa | Microsoft Azure Maps
description: En este artículo, obtendrá información sobre cómo representar un polígono y un multipolígono en un mapa del SDK web de Microsoft Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 07/29/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen, devx-track-javascript
ms.openlocfilehash: b267109113ce4d2ec3e653fb079b760eb4a5a514
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87286877"
---
# <a name="add-a-polygon-layer-to-the-map"></a>Adición de una capa de polígono al mapa

En este artículo se muestra cómo representar las áreas de las geometrías de las características `Polygon` y `MultiPolygon` en el mapa con una capa de polígono. El SDK web de Azure Maps también admite la creación de geometrías de círculo tal como se define en el [esquema extendido de GeoJSON](extend-geojson.md#circle). Estos círculos se transforman en polígonos cuando se representan en el mapa. Todas las geometrías de características se pueden actualizar fácilmente si se encapsulan con la clase [atlas.Shape](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.shape?view=azure-iot-typescript-latest).

## <a name="use-a-polygon-layer"></a>Uso de una capa de polígono 

Cuando una capa de polígono se conecta a un origen de datos y se carga en el mapa, representa el área con las características `Polygon` y `MultiPolygon`. Para crear un polígono, agréguelo a un origen de datos y represéntelo con una capa de polígono mediante la clase [PolygonLayer](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.layer.polygonlayer?view=azure-iot-typescript-latest).

```javascript
//Create a data source and add it to the map.
var dataSource = new atlas.source.DataSource();
map.sources.add(dataSource);

//Create a rectangular polygon.
dataSource.add(new atlas.data.Feature(
    new atlas.data.Polygon([[
        [-73.98235, 40.76799],
        [-73.95785, 40.80044],
        [-73.94928, 40.7968],
        [-73.97317, 40.76437],
        [-73.98235, 40.76799]
    ]])
));

//Create and add a polygon layer to render the polygon to the map, below the label layer.
map.layers.add(new atlas.layer.PolygonLayer(dataSource, null,{
    fillColor: 'red',
    fillOpacity: 0.7
}), 'labels');
```

A continuación se muestra el ejemplo completo y en ejecución del código anterior.

<br/>

<iframe height='500' scrolling='no' title='Adición de un polígono a un mapa ' src='//codepen.io/azuremaps/embed/yKbOvZ/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen <a href='https://codepen.io/azuremaps/pen/yKbOvZ/'>Adición de un polígono a un mapa</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="use-a-polygon-and-line-layer-together"></a>Uso de un polígono y una capa de línea conjuntamente

Una capa de línea se usa para representar el contorno de los polígonos. En el ejemplo de código siguiente se representa un polígono como en el ejemplo anterior, pero ahora se agrega una capa de línea. Esta capa de línea es una segunda capa conectada al origen de datos.  

<br/>

<iframe height='500' scrolling='no' title='Capas de polígono y línea para agregar un polígono' src='//codepen.io/azuremaps/embed/aRyEPy/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen sobre <a href='https://codepen.io/azuremaps/pen/aRyEPy/'>capas de polígono y línea para agregar un polígono</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="fill-a-polygon-with-a-pattern"></a>Relleno de un polígono con un patrón

Además de rellenar un polígono con un color, puede usar un patrón de imágenes para rellenarlo. Cargue un patrón de imagen en los recursos de sprite de la imagen de los mapas y luego haga referencia a esta imagen con la propiedad `fillPattern` de la capa de polígono.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Patrón de relleno poligonal" src="//codepen.io/azuremaps/embed/JzQpYX/?height=500&theme-id=0&default-tab=js,result" frameborder="no" allowtransparency="true" allowfullscreen="true">
Consulte el Pen <a href='https://codepen.io/azuremaps/pen/JzQpYX/'>Patrón de relleno poligonal</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>


> [!TIP]
> El SDK web de Azure Maps proporciona varias plantillas de imagen personalizables que puede usar como patrones de relleno. Para más información, consulte el documento [Uso de plantillas de imagen](how-to-use-image-templates-web-sdk.md).

## <a name="customize-a-polygon-layer"></a>Personalizar una capa de polígono

La capa de polígono solo tiene algunas opciones de estilo. Esta es una herramienta para probarlas.

<br/>

<iframe height='700' scrolling='no' title='LXvxpg' src='//codepen.io/azuremaps/embed/LXvxpg/?height=700&theme-id=0&default-tab=result' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen <a href='https://codepen.io/azuremaps/pen/LXvxpg/'>LXvxpg</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

<a id="addACircle"></a>

## <a name="add-a-circle-to-the-map"></a>Adición de un círculo al mapa

Azure Maps usa una versión ampliada del esquema GeoJSON que proporciona una definición para círculos como se indica [aquí](extend-geojson.md#circle). Un círculo se representa en el mapa mediante la creación de una característica `Point`. Esta característica `Point` tiene una propiedad `subType` con un valor de `"Circle"` y una propiedad `radius` con un número que representa el radio en metros. 

```javascript
{
    "type": "Feature",
    "geometry": {
        "type": "Point",
        "coordinates": [-122.126986, 47.639754]
    },
    "properties": {
        "subType": "Circle",
        "radius": 100
    }
}  
```

El SDK web de Azure Maps convierte estas características `Point` en características `Polygon`. Después, estas características se representan en el mapa con las capas de polígono y línea, tal y como se muestra en el ejemplo de código siguiente.

<br/>

<iframe height='500' scrolling='no' title='Adición de un círculo a un mapa' src='//codepen.io/azuremaps/embed/PRmzJX/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen <a href='https://codepen.io/azuremaps/pen/PRmzJX/'>Adición de un círculo a un mapa</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="make-a-geometry-easy-to-update"></a>Fácil actualización para geometría

Una clase `Shape` encapsula un objeto [Geometry](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.data.geometry?view=azure-iot-typescript-latest) o [Feature](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.data.feature?view=azure-iot-typescript-latest) y simplifica la actualización y el mantenimiento de estas características. Para crear una instancia de una variable de forma, pase un objeto Geometry o un conjunto de propiedades al constructor de la forma.

```javascript
//Creating a shape by passing in a geometry and a object containing properties.
var shape1 = new atlas.Shape(new atlas.data.Point[0,0], { myProperty: 1 });

//Creating a shape using a feature.
var shape2 = new atlas.Shape(new atlas.data.Feature(new atlas.data.Point[0,0], { myProperty: 1 });
```

En el ejemplo de código siguiente se muestra cómo encapsular un objeto GeoJSON de círculo con una clase de forma. A medida que el valor del radio cambia en la forma, el círculo se representa automáticamente en el mapa.

<br/>

<iframe height='500' scrolling='no' title='Actualización de propiedades de forma' src='//codepen.io/azuremaps/embed/ZqMeQY/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen sobre <a href='https://codepen.io/azuremaps/pen/ZqMeQY/'>actualización del estilo</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="next-steps"></a>Pasos siguientes

Más información sobre las clases y los métodos utilizados en este artículo:

> [!div class="nextstepaction"]
> [Polygon](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.data.polygon?view=azure-iot-typescript-latest)

> [!div class="nextstepaction"]
> [PolygonLayer](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.layer.polygonlayer?view=azure-iot-typescript-latest)

> [!div class="nextstepaction"]
> [PolygonLayerOptions](/javascript/api/azure-maps-control/atlas.polygonlayeroptions?view=azure-iot-typescript-latest)

Para más ejemplos de código para agregar a los mapas, consulte los siguientes artículos:

> [!div class="nextstepaction"]
> [Creación de un origen de datos](create-data-source-web-sdk.md)

> [!div class="nextstepaction"]
> [Adición de un elemento emergente](map-add-popup.md)

> [!div class="nextstepaction"]
> [Uso de expresiones de estilo controladas por datos](data-driven-style-expressions-web-sdk.md)

> [!div class="nextstepaction"]
> [Uso de plantillas de imagen](how-to-use-image-templates-web-sdk.md)

> [!div class="nextstepaction"]
> [Adición de una capa de línea](map-add-line-layer.md)

Recursos adicionales:

> [!div class="nextstepaction"]
> [Extensión de la especificación GeoJSON de Azure Maps](extend-geojson.md#circle)