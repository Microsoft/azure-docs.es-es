---
title: Adición de la barra de herramientas de dibujo a un mapa | Microsoft Azure Maps
description: Cómo agregar una barra de herramientas de dibujo a un mapa mediante Azure Maps SDK Web
author: anastasia-ms
ms.author: v-stharr
ms.date: 09/04/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: devx-track-javascript
ms.openlocfilehash: a156896faff4262d9b8bd3cef10817a5b919749d
ms.sourcegitcommit: 07166a1ff8bd23f5e1c49d4fd12badbca5ebd19c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90086155"
---
# <a name="add-a-drawing-tools-toolbar-to-a-map"></a>Agregar una barra de herramientas de dibujo a un mapa

Este artículo muestra cómo usar el módulo de herramientas de dibujo y mostrar la barra de herramientas de Dibujo en el mapa. El control [DrawingToolbar](https://docs.microsoft.com/javascript/api/azure-maps-drawing-tools/atlas.control.drawingtoolbar) agrega la barra de herramientas de dibujo en el mapa. Aprenderá a crear mapas con solo una y todas las herramientas de dibujo y cómo personalizar la representación de las formas de dibujo en el administrador de dibujos.

## <a name="add-drawing-toolbar"></a>Incorporación de la barra de herramientas Dibujo

En el código siguiente crea una instancia del administrador de dibujos y muestra la barra de herramientas en el mapa.

```javascript
//Create an instance of the drawing manager and display the drawing toolbar.
drawingManager = new atlas.drawing.DrawingManager(map, {
        toolbar: new atlas.control.DrawingToolbar({
            position: 'top-right',
            style: 'dark'
        })
    });
```

A continuación se muestra el ejemplo de código de ejecución completo de la funcionalidad anterior:

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Incorporación de la barra de herramientas Dibujo" src="//codepen.io/azuremaps/embed/ZEzLeRg/?height=265&theme-id=0&default-tab=js,result&editable=true" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
Consulte el Pen <a href='https://codepen.io/azuremaps/pen/ZEzLeRg/'>Adición de una barra de herramientas de dibujo</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>


## <a name="limit-displayed-toolbar-options"></a>Limite las opciones de la barra de herramientas mostradas

En el código siguiente crea una instancia del administrador de dibujos y muestra la barra de herramientas solo con una herramienta de dibujo de polígono en el mapa. 

```javascript
//Create an instance of the drawing manager and display the drawing toolbar with polygon drawing tool.
drawingManager = new atlas.drawing.DrawingManager(map, {
        toolbar: new atlas.control.DrawingToolbar({
            position: 'top-right',
            style: 'light',
            buttons: ["draw-polygon"]
        })
    });
```

A continuación se muestra el ejemplo de código de ejecución completo de la funcionalidad anterior:

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Agregar una herramienta de dibujo de polígono" src="//codepen.io/azuremaps/embed/OJLWWMy/?height=265&theme-id=0&default-tab=js,result&editable=true" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
Consulte el Pen <a href='https://codepen.io/azuremaps/pen/OJLWWMy/'>Adición de una herramienta de dibujo de polígono</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>


## <a name="change-drawing-rendering-style"></a>Cambiar el estilo de representación de dibujo

Se puede personalizar el estilo de las formas que se dibujan recuperando las capas subyacentes del administrador de dibujos usando la función `drawingManager.getLayers()` y, después, estableciendo opciones en las capas individuales. Los controladores de arrastre que aparecen para las coordenadas al editar una forma son marcadores HTML. El estilo de los controladores de arrastre se puede personalizar pasando opciones de marcador HTML en las opciones `dragHandleStyle` y `secondaryDragHandleStyle` del administrador de dibujos.  

El código siguiente obtiene las capas de representación del administrador de dibujos y modifica sus opciones para cambiar el estilo de representación del dibujo. En este caso, los puntos se representarán con un icono de marcador azul. Las líneas serán rojas y tendrán cuatro píxeles de ancho. Los polígonos tendrán el relleno verde y el contorno naranja. Después, cambia los estilos de los controladores de arrastre para que sean iconos cuadrados. 

```javascript
//Get rendering layers of drawing manager.
var layers = drawingManager.getLayers();

//Change the icon rendered for points.
layers.pointLayer.setOptions({
    iconOptions: {
        image: 'marker-blue'
    }
});

//Change the color and width of lines.
layers.lineLayer.setOptions({
    strokeColor: 'red',
    strokeWidth: 4
});

//Change fill color of polygons.
layers.polygonLayer.setOptions({
    fillColor: 'green'
});

//Change the color of polygon outlines.
layers.polygonOutlineLayer.setOptions({
    strokeColor: 'orange'
});

//Update the style of the drag handles that appear when editting.
drawingManager.setOptions({
    //Primary drag handle that represents coordinates in the shape.
    dragHandleStyle: {
        anchor: 'center',
        htmlContent: '<svg width="15" height="15" viewBox="0 0 15 15" xmlns="http://www.w3.org/2000/svg" style="cursor:pointer"><rect x="0" y="0" width="15" height="15" style="stroke:black;fill:white;stroke-width:4px;"/></svg>',
        draggable: true
    },

    //Secondary drag hanle that represents mid-point coordinates that users can grab to add new cooridnates in the middle of segments.
    secondaryDragHandleStyle: {
        anchor: 'center',
        htmlContent: '<svg width="10" height="10" viewBox="0 0 10 10" xmlns="http://www.w3.org/2000/svg" style="cursor:pointer"><rect x="0" y="0" width="10" height="10" style="stroke:white;fill:black;stroke-width:4px;"/></svg>',
        draggable: true
    }
});  
```

A continuación se muestra el ejemplo de código de ejecución completo de la funcionalidad anterior:

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Cambiar el estilo de representación de dibujo" src="//codepen.io/azuremaps/embed/OJLWpyj/?height=265&theme-id=0&default-tab=js,result&editable=true" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
Consulte el Pen <a href='https://codepen.io/azuremaps/pen/OJLWpyj/'>Cambiar el estilo de representación de dibujo</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>)en <a href='https://codepen.io'>CodePen</a>.
</iframe>


## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo usar características adicionales del módulo de herramientas de dibujo:

> [!div class="nextstepaction"]
> [Obtención de datos de forma](map-get-shape-data.md)

> [!div class="nextstepaction"]
> [Reacción a eventos de dibujo](drawing-tools-events.md)

> [!div class="nextstepaction"]
> [Tipos de interacción y métodos abreviados de teclado](drawing-tools-interactions-keyboard-shortcuts.md)

Más información sobre las clases y los métodos utilizados en este artículo:

> [!div class="nextstepaction"]
> [Map](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map)

> [!div class="nextstepaction"]
> [Barra de herramientas de dibujo](https://docs.microsoft.com/javascript/api/azure-maps-drawing-tools/atlas.control.drawingtoolbar)

> [!div class="nextstepaction"]
> [Administrador de dibujo](https://docs.microsoft.com/javascript/api/azure-maps-drawing-tools/atlas.drawing.drawingmanager)
