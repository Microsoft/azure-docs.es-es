---
title: Control de eventos de mapa | Microsoft Azure Maps
description: En este artículo, obtendrá información sobre cómo crear un mapa de SDK de web interactivo con eventos de mapa mediante el SDK web de Microsoft Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 09/10/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen, devx-track-javascript
ms.openlocfilehash: 667ab684ef4a453d210af36c0221d350b25c9829
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87282898"
---
# <a name="interact-with-the-map"></a>Interacción con el mapa

En este artículo se muestra cómo usar la [clase de eventos de mapa](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?#events). La propiedad resalta los eventos en el mapa y en diferentes capas del mapa. También puede resaltar eventos cuando se interactúa con un marcador HTML.

## <a name="interact-with-the-map"></a>Interacción con el mapa

Practique con el mapa siguiente y observe los eventos del mouse correspondientes resaltados en la parte derecha. Puede hacer clic en la **pestaña JS** para ver y editar el código JavaScript. También puede hacer clic en **Editar en CodePen** para modificar el código en CodePen.

<br/>

<iframe height='600' scrolling='no' title='Interacción con el mapa: eventos del mouse' src='//codepen.io/azuremaps/embed/bLZEWd/?height=600&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen <a href='https://codepen.io/azuremaps/pen/bLZEWd/'>Interacción con el mapa: eventos del mouse</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="interact-with-map-layers"></a>Interacción con las capas de mapa

En el código siguiente se resalta el evento que se activa cuando se interactúa con la capa de símbolos. Las capas de símbolo, burbuja, línea y polígono admiten el mismo conjunto de eventos. Las capas de mapa térmico y de mosaico no admiten ninguno de estos eventos.

<br/>

<iframe height='600' scrolling='no' title='Interacción con el mapa: eventos de las capas' src='//codepen.io/azuremaps/embed/bQRRPE/?height=600&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen <a href='https://codepen.io/azuremaps/pen/bQRRPE/'>Interacción con el mapa: eventos de las capas</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="interact-with-html-marker"></a>Interacción con el marcador HTML

El código siguiente agrega eventos de mapa JavaScript a un marcador HTML. También resalta el nombre de los eventos que se activan al interactuar con el marcador HTML.

<br/>

<iframe height='500' scrolling='no' title='Interacción con el mapa: eventos del marcador HTML' src='//codepen.io/azuremaps/embed/VVzKJY/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen <a href='https://codepen.io/azuremaps/pen/VVzKJY/'>Interacción con el mapa: eventos del marcador HTML</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

En la siguiente tabla se muestran todos los eventos de clase de mapa admitidos.

| Evento               | Descripción |
|---------------------|-------------|
| `boxzoomend`        | Se genera cuando finaliza una interacción "zoom del cuadro".|
| `boxzoomstart`      | Se genera cuando se inicia una interacción "zoom del cuadro".|
| `click`             | Se genera cuando se presiona un dispositivo señalador y se libera en el mismo punto del mapa.|
| `close`             | Se genera cuando se cierra manualmente o mediante programación el elemento emergente.|
| `contextmenu`       | Se genera cuando se hace clic en el botón derecho del mouse.|
| `data`              | Se genera cuando se cargan o cambian datos del mapa. |
| `dataadded`         | Se genera cuando se agregan formas a `DataSource`.|
| `dataremoved`       | Se genera cuando se quitan formas de `DataSource`.|
| `datasourceupdated` | Se genera cuando se actualiza el objeto `DataSource`.|
| `dblclick`          | Se genera cuando se hace doble clic en un dispositivo señalador en el mismo punto del mapa.|
| `drag`              | Se genera repetidamente durante una interacción "arrastrar para obtener panorámica" en el mapa, elemento emergente o marcador HTML.|
| `dragend`           | Se genera cuando finaliza una interacción "arrastrar para obtener panorámica" en el mapa, elemento emergente o marcador HTML.|
| `dragstart`         | Se genera cuando se inicia una interacción "arrastrar para obtener panorámica" en el mapa, elemento emergente o marcador HTML.|
| `error`             | Se genera cuando se produce un error.|
| `idle`              | <p>Se genera después del último fotograma representado antes de que el mapa entre en un estado "inactivo":<ul><li>No hay transiciones de cámara en curso.</li><li>Todos los elementos solicitados actualmente se han cargado.</li><li>Todas las animaciones de fundido y transición se han completado.</li></ul></p>|
| `keydown`           | Se genera cuando se presiona una tecla.|
| `keypress`          | Se genera cuando se presiona una tecla que produce un carácter que se puede escribir (una tecla ANSI).|
| `keyup`             | Se genera cuando se libera una tecla.|
| `layeradded`        | Se genera cuando se agrega una capa al mapa.|
| `layerremoved`      | Se genera cuando se quita una capa del mapa.|
| `load`              | Se genera inmediatamente después de que se hayan descargado todos los recursos necesarios y de que se haya producido la primera representación visualmente completa del mapa.|
| `mousedown`         | Se genera cuando se presiona un dispositivo señalador en el mapa o cuando está sobre un elemento.|
| `mouseenter`        | Se genera cuando un dispositivo señalador se mueve inicialmente sobre el mapa o un elemento. |
| `mouseleave`        | Se genera cuando un dispositivo señalador se mueve fuera del mapa o un elemento. |
| `mousemove`         | Se genera cuando un dispositivo señalador se mueve en el mapa o un elemento.|
| `mouseout`          | Se genera cuando un dispositivo señalador deja el lienzo del mapa o un elemento.|
| `mouseover`         | Se genera cuando un dispositivo señalador se mueve sobre el mapa o un elemento.|
| `mouseup`           | Se genera cuando se libera un dispositivo señalador en el mapa o cuando está sobre un elemento.|
| `move`              | Se genera repetidamente durante una transición animada desde una vista a otra, como resultado de la interacción del usuario o los métodos.|
| `moveend`           | Se genera justo después de que el mapa complete una transición desde una vista a otra, como resultado de la interacción del usuario o los métodos.|
| `movestart`         | Se genera justo antes de que el mapa inicie una transición desde una vista a otra, como resultado de la interacción del usuario o los métodos.|
| `open`              | Se genera cuando se abre manualmente o mediante programación el elemento emergente.|
| `pitch`             | Se genera cuando el evento pitch (inclinación) del mapa cambia como resultado de la interacción del usuario o los métodos.|
| `pitchend`          | Se genera inmediatamente después de que el evento pitch (inclinación) del mapa termine de cambiar como resultado de la interacción del usuario o los métodos.|
| `pitchstart`        | Se genera si el evento pitch (inclinación) del mapa inicia un cambio como resultado de la interacción del usuario o los métodos.|
| `ready`             | Se genera cuando se cargan los recursos de asignación necesarios mínimos antes de que el mapa esté listo para que se interactúe con él mediante programación.|
| `render`            | <p>Se genera siempre que se dibuje el mapa en la pantalla, como resultado de:<ul><li>Un cambio en la posición, el zoom, el evento pitch o el rumbo del mapa.</li><li>Un cambio en el estilo del mapa.</li><li>Un cambio en un origen de `DataSource`.</li><li>La carga de un corte vectorial, un archivo GeoJSON, un glifo o un sprite.</li></ul></p>|
| `resize`            | Se genera inmediatamente después de que se haya cambiado el tamaño del mapa.|
| `rotate`            | Se genera repetidamente durante una interacción "arrastrar para girar".|
| `rotateend`         | Se genera cuando finaliza una interacción "arrastrar para girar".|
| `rotatestart`       | Se genera cuando se inicia una interacción "arrastrar para girar".|
| `shapechanged`      | Se genera cuando se cambia una propiedad del objeto de forma.|
| `sourcedata`        | Se genera cuando uno de los orígenes de un mapa se carga o cambia, también si un elemento que pertenece a un origen se carga o cambia. |
| `sourceadded`       | Se genera cuando se agrega `DataSource` o `VectorTileSource` al mapa.|
| `sourceremoved`     | Se genera cuando se quita `DataSource` o `VectorTileSource` del mapa.|
| `styledata`         | Se genera cuando se carga o cambia el estilo del mapa.|
| `styleimagemissing` | Se genera cuando una capa intenta cargar una imagen desde el sprite de imagen que no existe. |
| `tokenacquired`     | Se genera cuando se obtiene un token de acceso de AAD.|
| `touchcancel`       | Se genera cuando se produce un evento touchcancel en el mapa.|
| `touchend`          | Se genera cuando se produce un evento touchend en el mapa.|
| `touchmove`         | Se genera cuando se produce un evento touchmove en el mapa.|
| `touchstart`        | Se genera cuando se produce un evento touchstart en el mapa.|
| `wheel`             | Se genera cuando se produce un evento wheel del mouse en el mapa.|
| `zoom`              | Se genera repetidamente durante una transición animada desde un nivel de zoom a otro, como resultado de la interacción del usuario o los métodos.|
| `zoomend`           | Se genera justo después de que el mapa complete una transición desde un nivel de zoom a otro, como resultado de la interacción del usuario o los métodos.|
| `zoomstart`         | Se genera justo antes de que el mapa inicie una transición desde un nivel de zoom a otro, como resultado de la interacción del usuario o los métodos.|


## <a name="next-steps"></a>Pasos siguientes

Consulte los siguientes artículos para obtener ejemplos de código completo:

> [!div class="nextstepaction"]
> [Usar el módulo de servicios de Azure Maps](./how-to-use-services-module.md)

> [!div class="nextstepaction"]
> [Ejemplos de código](https://docs.microsoft.com/samples/browse/?products=azure-maps)
