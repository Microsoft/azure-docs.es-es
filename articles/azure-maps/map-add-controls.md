---
title: Incorporación de controles a un mapa | Microsoft Azure Maps
description: Cómo agregar control de zoom, control de inclinación, control de giro y un selector de estilos a un mapa en Microsoft Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 07/29/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.custom: devx-track-javascript
ms.openlocfilehash: 9a54ef02677c657d9a86507370344dd335fd497c
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87286998"
---
# <a name="add-controls-to-a-map"></a>Incorporación de controles a un mapa

En este artículo se muestra cómo agregar controles a un mapa. También aprenderá a crear un mapa con todos los controles y un [selector de estilos](https://docs.microsoft.com/azure/azure-maps/choose-map-style).

## <a name="add-zoom-control"></a>Agregar un control de zoom

Un control de zoom agrega botones para acercar y alejar el mapa. En el siguiente ejemplo de código, se crea una instancia de la clase [ZoomControl](/javascript/api/azure-maps-control/atlas.control.zoomcontrol) y se agrega a la esquina inferior derecha del mapa.

```javascript
//Construct a zoom control and add it to the map.
map.controls.add(new atlas.control.ZoomControl(), {
    position: 'bottom-right'
});
```

A continuación se muestra el código de ejemplo de ejecución completo de la funcionalidad anterior.

<br/>

<iframe height='500' scrolling='no' title='Adición de un control de zoom' src='//codepen.io/azuremaps/embed/WKOQyN/?height=265&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen <a href='https://codepen.io/azuremaps/pen/WKOQyN/'>Adding a zoom control</a> (Agregar un control de zoom) de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="add-pitch-control"></a>Agregar un control de inclinación

Un control de inclinación agrega botones para inclinar el mapa con respecto al horizonte. En el siguiente ejemplo de código, se crea una instancia de la clase [PitchControl](/javascript/api/azure-maps-control/atlas.control.pitchcontrol). Después, esta instancia se agrega en la esquina superior derecha del mapa.

```javascript
//Construct a pitch control and add it to the map.
map.controls.add(new atlas.control.PitchControl(), {
    position: 'top-right'
});
```

A continuación se muestra el código de ejemplo de ejecución completo de la funcionalidad anterior.

<br/>

<iframe height='500' scrolling='no' title='Adición de un control de inclinación' src='//codepen.io/azuremaps/embed/xJrwaP/?height=265&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen <a href='https://codepen.io/azuremaps/pen/xJrwaP/'>Adding a pitch control</a> (Agregar un control de inclinación) de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="add-compass-control"></a>Agregar un control de brújula

Un control de brújula agrega un botón para girar el mapa. El siguiente ejemplo de código crea una instancia de la clase [CompassControl](/javascript/api/azure-maps-control/atlas.control.compasscontrol) y la agrega a la esquina inferior izquierda del mapa.

```javascript
//Construct a compass control and add it to the map.
map.controls.add(new atlas.control.Compass(), {
    position: 'bottom-left'
});
```

A continuación se muestra el código de ejemplo de ejecución completo de la funcionalidad anterior.

<br/>

<iframe height='500' scrolling='no' title='Adición de un control de giro' src='//codepen.io/azuremaps/embed/GBEoRb/?height=265&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen <a href='https://codepen.io/azuremaps/pen/GBEoRb/'>Adding a rotate control</a> (Agregar un control de giro) de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="a-map-with-all-controls"></a>Mapa con todos los controles

Se pueden colocar varios controles en una matriz, agregarlos al mapa a la vez y colocarlos en la misma área del mapa para simplificar el desarrollo. Lo siguiente agrega los controles de navegación estándar al mapa usando este método.

```javascript
map.controls.add([
    new atlas.control.ZoomControl(),
    new atlas.control.CompassControl(),
    new atlas.control.PitchControl(),
    new atlas.control.StyleControl()
], {
    position: "top-right"
});
```

En el siguiente código de ejemplo se agregan los controles de zoom, inclinación, brújula y selector de estilo a la esquina superior derecha del mapa. Observe cómo se apilan automáticamente. El orden de los objetos de control del script determina el orden en que aparecen en el mapa. Para cambiar el orden de los controles en el mapa, puede cambiar su orden en la matriz.

<br/>

<iframe height='500' scrolling='no' title='Mapa con todos los controles' src='//codepen.io/azuremaps/embed/qyjbOM/?height=265&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen <a href='https://codepen.io/azuremaps/pen/qyjbOM/'>A map with all the controls</a> (Mapa con todos los controles) de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

El control selector de estilo se define mediante la clase [StyleControl](/javascript/api/azure-maps-control/atlas.control.stylecontrol). Para más información sobre el uso del control selector de estilo, consulte el artículo en el que se explica cómo [elegir un estilo de mapa](choose-map-style.md).

## <a name="customize-controls"></a>Personalización de controles

Esta es una herramienta para probar las distintas opciones de personalización de los controles.

<br/>

<iframe height="700" style="width: 100%;" scrolling="no" title="Opciones de control de la navegación" src="//codepen.io/azuremaps/embed/LwBZMx/?height=700&theme-id=0&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
Consulte el Pen <a href='https://codepen.io/azuremaps/pen/LwBZMx/'>Symbol Layer Options</a> (Opciones de capa de símbolos) de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

Si desea crear controles de navegación personalizados, cree una clase que extienda desde la clase `atlas.Control`, o bien cree un elemento HTML y colóquelo encima de la división del mapa. Haga que este control de la interfaz de usuario llame a la función `setCamera` de Maps para mover el mapa. 

## <a name="next-steps"></a>Pasos siguientes

Más información sobre las clases y los métodos utilizados en este artículo:

> [!div class="nextstepaction"]
> [Control de brújula](/javascript/api/azure-maps-control/atlas.control.compasscontrol)

> [!div class="nextstepaction"]
> [PitchControl](/javascript/api/azure-maps-control/atlas.control.pitchcontrol) 

> [!div class="nextstepaction"]
> [StyleControl](/javascript/api/azure-maps-control/atlas.control.stylecontrol) 

> [!div class="nextstepaction"]
> [ZoomControl](/javascript/api/azure-maps-control/atlas.control.zoomcontrol) 

Consulte los siguientes artículos para el código completo:

> [!div class="nextstepaction"]
> [Adición de un anclaje](./map-add-pin.md)

> [!div class="nextstepaction"]
> [Adición de un elemento emergente](./map-add-popup.md)

> [!div class="nextstepaction"]
> [Adición de una capa de línea](map-add-line-layer.md)

> [!div class="nextstepaction"]
> [Adición de una capa de polígono](map-add-shape.md)

> [!div class="nextstepaction"]
> [Adición de una capa de burbuja](map-add-bubble-layer.md)

