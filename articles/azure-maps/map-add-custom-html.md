---
title: Adición de un marcador HTML a un mapa | Microsoft Azure Maps
description: Obtenga información sobre cómo agregar marcadores HTML a los mapas. Vea cómo usar el SDK web de Azure Maps para personalizar marcadores y agregar elementos emergentes y eventos del mouse a un marcador.
author: anastasia-ms
ms.author: v-stharr
ms.date: 07/29/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen, devx-track-javascript
ms.openlocfilehash: 58954a98215fd353c5944486446dab5664e5349c
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88035440"
---
# <a name="add-html-markers-to-the-map"></a>Agregar marcadores HTML al mapa

En este artículo se explica cómo agregar al mapa código HTML personalizado como un archivo de imagen en forma de marcador HTML.

> [!NOTE]
> Los marcadores HTML no se conectan a los orígenes de datos. En su lugar, se agrega información de posición directamente en el marcador y el marcador se agrega a la propiedad `markers` de los mapas, que es un elemento [HtmlMarkerManager](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.htmlmarkermanager?view=azure-iot-typescript-latest).

> [!IMPORTANT]
> A diferencia de la mayoría de las capas del control web de Azure Maps que usan WebGL para la representación, los marcadores HTML usan elementos DOM tradicionales para la representación. Por tanto, cuantos más marcadores HTML se agreguen a una página, más elementos DOM habrá. Se puede degradar el rendimiento después de agregar cientos de marcadores HTML. Para conjuntos de datos grandes, considere la posibilidad de agrupar los datos en clústeres o de usar una capa de símbolo o burbuja.

## <a name="add-an-html-marker"></a>Adición de un marcador HTML

La clase [HtmlMarker](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.htmlmarker?view=azure-iot-typescript-latest) tiene un estilo predeterminado. Puede personalizar el marcador mediante la configuración de las opciones de color y texto del marcador. El estilo predeterminado de la clase del marcador HTML es una plantilla SVG que tiene un marcador de posición `{color}` y `{text}`. Configure las propiedades de color y texto en las opciones del marcador HTML para una rápida personalización. 

En el código siguiente se crea un marcador HTML y se establece la propiedad color en "DodgerBlue" y la propiedad text en "10". Se adjunta un elemento emergente al marcador, y el evento `click` se usa para alternar la visibilidad del elemento emergente.

```javascript
//Create an HTML marker and add it to the map.
var marker = new atlas.HtmlMarker({
    color: 'DodgerBlue',
    text: '10',
    position: [0, 0],
    popup: new atlas.Popup({
        content: '<div style="padding:10px">Hello World</div>',
        pixelOffset: [0, -30]
    })
});

map.markers.add(marker);

//Add a click event to toggle the popup.
map.events.add('click',marker, () => {
    marker.togglePopup();
});
```

A continuación se muestra el código de ejemplo de ejecución completo de la funcionalidad anterior.

<br/>

<iframe height='500' scrolling='no' title='Adición de un marcador HTML a un mapa' src='//codepen.io/azuremaps/embed/MVoeVw/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen <a href='https://codepen.io/azuremaps/pen/MVoeVw/'>Adición de un marcador HTML a un mapa</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="create-svg-templated-html-marker"></a>Creación de un marcador HTML con una plantilla SVG

El valor predeterminado `htmlContent` de un marcador HTML es una plantilla SVG que contiene las carpetas de posición `{color}` y `{text}`. Puede crear cadenas SVG personalizadas y agregar estos mismos marcadores de posición en las opciones SVG, de tal forma que la configuración de las opciones `color` y `text` del marcador actualice estos marcadores de posición en la plantilla SVG.

<br/>

<iframe height='500' scrolling='no' title='Marcador HTML con la plantilla SVG personalizada' src='//codepen.io/azuremaps/embed/LXqMWx/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen <a href='https://codepen.io/azuremaps/pen/LXqMWx/'>Marcador HTML con la plantilla SVG personalizada</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

> [!TIP]
> El SDK web de Azure Maps proporciona varias plantillas de imagen SVG que se pueden usar con marcadores HTML. Para más información, consulte el documento [Uso de plantillas de imagen](how-to-use-image-templates-web-sdk.md).

## <a name="add-a-css-styled-html-marker"></a>Adición de un marcador HTML con estilo CSS

Una de las ventajas de los marcadores HTML es que hay muchas personalizaciones excelentes que pueden conseguirse mediante CSS. En este ejemplo, el contenido de HtmlMarker consta de HTML y CSS que crean un pin animado que se coloca en su lugar y se mueve.

<br/>

<iframe height='500' scrolling='no' title='HTML DataSource' src='//codepen.io/azuremaps/embed/qJVgMx/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen <a href='https://codepen.io/azuremaps/pen/qJVgMx/'>HTML DataSource</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="draggable-html-markers"></a>Marcadores HTML arrastrables

En este ejemplo se muestra cómo hacer que un marcador HTML se pueda arrastrar. Los marcadores HTML admiten eventos `drag`, `dragstart` y `dragend`.

<br/>

<iframe height='500' scrolling='no' title='Marcador HTML arrastrable' src='//codepen.io/azuremaps/embed/wQZoEV/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen <a href='https://codepen.io/azuremaps/pen/wQZoEV/'>Marcador HTML arrastrable</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="add-mouse-events-to-html-markers"></a>Adición de eventos del mouse a los marcadores HTML

En estos ejemplos se muestra cómo agregar eventos del mouse y de arrastrar a un marcador HTML.

<br/>

<iframe height='500' scrolling='no' title='Adición de eventos del mouse a los marcadores HTML' src='//codepen.io/azuremaps/embed/RqOKRz/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen <a href='https://codepen.io/azuremaps/pen/RqOKRz/'>Adición de eventos del mouse a los marcadores HTML</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="next-steps"></a>Pasos siguientes

Más información sobre las clases y los métodos utilizados en este artículo:

> [!div class="nextstepaction"]
> [HtmlMarker](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.htmlmarker?view=azure-iot-typescript-latest)

> [!div class="nextstepaction"]
> [HtmlMarkerOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.htmlmarkeroptions?view=azure-iot-typescript-latest)

> [!div class="nextstepaction"]
> [HtmlMarkerManager](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.htmlmarkermanager?view=azure-iot-typescript-latest)

Para más ejemplos de código para agregar a los mapas, consulte los siguientes artículos:

> [!div class="nextstepaction"]
> [Uso de plantillas de imagen](how-to-use-image-templates-web-sdk.md)

> [!div class="nextstepaction"]
> [Adición de una capa de símbolo](./map-add-pin.md)

> [!div class="nextstepaction"]
> [Adición de una capa de burbuja](./map-add-bubble-layer.md)
