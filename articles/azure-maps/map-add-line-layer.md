---
title: Adición de una capa de línea a un mapa | Microsoft Azure Maps
description: Aprenda a agregar líneas a mapas. Vea los ejemplos que usan el SDK web de Azure Maps para agregar capas de línea a los mapas y a personalizar las líneas con símbolos y gradientes de color.
author: rbrundritt
ms.author: richbrun
ms.date: 08/08/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen, devx-track-javascript
ms.openlocfilehash: 11783ceae1b2a2f9782df74976d8ea6d66573a1c
ms.sourcegitcommit: 07166a1ff8bd23f5e1c49d4fd12badbca5ebd19c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90086035"
---
# <a name="add-a-line-layer-to-the-map"></a>Adición de una capa de línea al mapa

Una capa de línea se puede usar para representar las características `LineString` y `MultiLineString` como rutas de acceso o rutas en el mapa. También se puede usar una capa de línea para representar el contorno de las características `Polygon` y `MultiPolygon`. Se puede conectar un origen de datos a la capa de línea para que proporcione los datos que se van a representar. 

> [!TIP]
> Las capas de línea de forma predeterminada representarán las coordenadas de los polígonos y las líneas en un origen de datos. Para limitar la capa de forma que solo represente las características de LineString, configure la propiedad `filter` de la capa en `['==', ['geometry-type'], 'LineString']` o `['any', ['==', ['geometry-type'], 'LineString'], ['==', ['geometry-type'], 'MultiLineString']]` si desea incluir también las características de MultiLineString.

En el código siguiente, se muestra cómo se crea una línea. Agregue la línea a un origen de datos y represéntela con una capa de líneas utilizando la clase [LineLayer](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.layer.linelayer).

```javascript
//Create a data source and add it to the map.
var dataSource = new atlas.source.DataSource();
map.sources.add(dataSource);

//Create a line and add it to the data source.
dataSource.add(new atlas.data.LineString([[-73.972340, 40.743270], [-74.004420, 40.756800]]));
  
//Create a line layer to render the line to the map.
map.layers.add(new atlas.layer.LineLayer(dataSource, null, {
    strokeColor: 'blue',
    strokeWidth: 5
}));
```

A continuación se muestra el código de ejemplo de ejecución completo de la funcionalidad anterior.

<br/>

<iframe height='500' scrolling='no' title='Adición de una línea a un mapa' src='//codepen.io/azuremaps/embed/qomaKv/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen <a href='https://codepen.io/azuremaps/pen/qomaKv/'>Adición de una línea a un mapa</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

Se puede aplicar estilo a las capas de línea mediante [LineLayerOptions](/javascript/api/azure-maps-control/atlas.linelayeroptions) y [Uso de expresiones de estilo controladas por datos](data-driven-style-expressions-web-sdk.md).

## <a name="add-symbols-along-a-line"></a>Adición de símbolos a lo largo de una línea

En este ejemplo se muestra cómo agregar iconos de flecha situados a lo largo de una línea en el mapa. Si utiliza una capa de símbolos, establezca la opción "placement" en "line". Esta opción representará los símbolos a lo largo de la línea y girará los iconos (0 grados = derecha).

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Visualización de la flecha a lo largo de la línea" src="//codepen.io/azuremaps/embed/drBJwX/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder='no' loading="lazy" loading="lazy" allowtransparency="true" allowfullscreen="true">
Consulte el Pen <a href='https://codepen.io/azuremaps/pen/drBJwX/'>Visualización de la flecha a lo largo de la línea</a> por Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

> [!TIP]
> El SDK de Azure Maps para web proporciona varias plantillas de imagen personalizables que puede usar con la capa de símbolo. Para más información, consulte el documento [Uso de plantillas de imagen](how-to-use-image-templates-web-sdk.md).

<a name="line-stroke-gradient"></a>

## <a name="add-a-stroke-gradient-to-a-line"></a>Adición de un degradado de trazo a una línea

Puede aplicar un color de trazo único a la línea. También puede rellenar la línea con un degradado de colores para mostrar la transición de un segmento al siguiente. Por ejemplo, los degradados de línea pueden utilizarse para representar cambios en el tiempo y en la distancia, o bien diferentes temperaturas a lo largo de una línea de objetos conectada. Para poder aplicar esta característica a una línea, el origen de datos debe tener la opción `lineMetrics` establecida en true; de este modo, se puede pasar una expresión de degradado a la opción `strokeColor` de la línea. La expresión de degradado del trazo tiene que hacer referencia a la expresión de datos `['line-progress']` que expone las métricas de línea calculadas a la expresión.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Línea con degradado de trazo" src="//codepen.io/azuremaps/embed/wZwWJZ/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder='no' loading="lazy" loading="lazy" allowtransparency="true" allowfullscreen="true">
Consulte el Pen <a href='https://codepen.io/azuremaps/pen/wZwWJZ/'>Línea con degradado de trazo</a> por Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="customize-a-line-layer"></a>Personalizar una capa de línea

La capa de líneas dispone de varias opciones de estilo. Esta es una herramienta para probarlas.

<br/>

<iframe height='700' scrolling='no' title='Opciones de capa de línea' src='//codepen.io/azuremaps/embed/GwLrgb/?height=700&theme-id=0&default-tab=result' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen <a href='https://codepen.io/azuremaps/pen/GwLrgb/'>Opciones de capa de línea</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="next-steps"></a>Pasos siguientes

Más información sobre las clases y los métodos utilizados en este artículo:

> [!div class="nextstepaction"]
> [LineLayer](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.layer.linelayer) 

> [!div class="nextstepaction"]
> [LineLayerOptions](/javascript/api/azure-maps-control/atlas.linelayeroptions)

Para obtener más ejemplos de código para agregar a los mapas:

> [!div class="nextstepaction"]
> [Creación de un origen de datos](create-data-source-web-sdk.md)

> [!div class="nextstepaction"]
> [Adición de un elemento emergente](map-add-popup.md)

> [!div class="nextstepaction"]
> [Uso de expresiones de estilo controladas por datos](data-driven-style-expressions-web-sdk.md)

> [!div class="nextstepaction"]
> [Uso de plantillas de imagen](how-to-use-image-templates-web-sdk.md)

> [!div class="nextstepaction"]
> [Adición de una capa de polígono](map-add-shape.md)
