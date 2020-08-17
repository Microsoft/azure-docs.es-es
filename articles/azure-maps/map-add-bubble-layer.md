---
title: Adición de una capa de burbuja a un mapa | Microsoft Azure Maps
description: En este artículo aprenderá a agregar una capa de burbujas a un mapa mediante el SDK web de Microsoft Azure Maps.
author: rbrundritt
ms.author: richbrun
ms.date: 07/29/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen, devx-track-javascript
ms.openlocfilehash: 1a86c1ef12a4cae4132e79a742b99c1b95ee4cd5
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87287030"
---
# <a name="add-a-bubble-layer-to-a-map"></a>Adición de una capa de burbuja a un mapa

En este artículo, se explica cómo puede representar datos de punto procedentes de un origen de datos en un mapa como una capa de burbujas. Las capas de burbujas representan puntos como círculos en el mapa con un radio de píxel fijo. 

> [!TIP]
> Las capas de burbuja de forma predeterminada representarán las coordenadas de todos los objetos geométricos en un origen de datos. Para limitar la capa de forma que solo represente las características de geometría de puntos, configure la propiedad `filter` de la capa en `['==', ['geometry-type'], 'Point']` o `['any', ['==', ['geometry-type'], 'Point'], ['==', ['geometry-type'], 'MultiPoint']]` si desea incluir también las características de MultiPoint.

## <a name="add-a-bubble-layer"></a>Adición de una capa de burbuja

En el código siguiente, se carga una matriz de puntos en un origen de datos. A continuación, conecta los puntos de datos a una [capa de burbujas](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.layer.bubblelayer?view=azure-iot-typescript-latest). La capa de burbujas representa el radio de cada burbuja en cinco píxeles, el color de relleno en blanco, el color de trazo en azul y el ancho del trazo con seis píxeles. 

```javascript
//Add point locations.
var points = [
    new atlas.data.Point([-73.985708, 40.75773]),
    new atlas.data.Point([-73.985600, 40.76542]),
    new atlas.data.Point([-73.985550, 40.77900]),
    new atlas.data.Point([-73.975550, 40.74859]),
    new atlas.data.Point([-73.968900, 40.78859])
];

//Create a data source and add it to the map.
var dataSource = new atlas.source.DataSource();
map.sources.add(dataSource);

//Add multiple points to the data source.
dataSource.add(points);

//Create a bubble layer to render the filled in area of the circle, and add it to the map.
map.layers.add(new atlas.layer.BubbleLayer(dataSource, null, {
    radius: 5,
    strokeColor: "#4288f7",
    strokeWidth: 6, 
    color: "white" 
}));
```

A continuación se muestra el código de ejemplo de ejecución completo de la funcionalidad anterior.

<br/>

<iframe height='500' scrolling='no' title='BubbleLayer DataSource' src='//codepen.io/azuremaps/embed/mzqaKB/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen <a href='https://codepen.io/azuremaps/pen/mzqaKB/'>BubbleLayer DataSource</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="show-labels-with-a-bubble-layer"></a>Mostrar etiquetas con una capa de burbuja

Este código muestra cómo usar una capa de burbujas para representar un punto en el mapa y cómo usar una capa de símbolos para representar una etiqueta. Para ocultar el icono de la capa de símbolo, establezca la propiedad `image` de las opciones del icono en `'none'`.

<br/>

<iframe height='500' scrolling='no' title='MultiLayer DataSource' src='//codepen.io/azuremaps/embed/rqbQXy/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen <a href='https://codepen.io/azuremaps/pen/rqbQXy/'>MultiLayer DataSource</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="customize-a-bubble-layer"></a>Personalización de una capa de burbuja

La capa de burbuja solo tiene algunas opciones de estilo. Esta es una herramienta para probarlas.

<br/>

<iframe height='700' scrolling='no' title='Opciones de capa de burbuja' src='//codepen.io/azuremaps/embed/eQxbGm/?height=700&theme-id=0&default-tab=result' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen <a href='https://codepen.io/azuremaps/pen/eQxbGm/'>opciones de capa de burbuja</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="next-steps"></a>Pasos siguientes

Más información sobre las clases y los métodos utilizados en este artículo:

> [!div class="nextstepaction"]
> [BubbleLayer](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.layer.bubblelayer?view=azure-iot-typescript-latest)

> [!div class="nextstepaction"]
> [BubbleLayerOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.bubblelayeroptions?view=azure-iot-typescript-latest)

Para obtener más ejemplos de código para agregar a los mapas:

> [!div class="nextstepaction"]
> [Creación de un origen de datos](create-data-source-web-sdk.md)

> [!div class="nextstepaction"]
> [Adición de una capa de símbolo](map-add-pin.md)

> [!div class="nextstepaction"]
> [Uso de expresiones de estilo controladas por datos](data-driven-style-expressions-web-sdk.md)

> [!div class="nextstepaction"]
> [Ejemplos de código](https://docs.microsoft.com/samples/browse/?products=azure-maps)
