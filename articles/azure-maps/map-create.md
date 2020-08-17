---
title: Creación de un mapa con Azure Maps | Microsoft Azure Maps
description: En este artículo, se aprende a representar un mapa en una página web mediante el SDK web de Microsoft Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 07/26/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen, devx-track-javascript
ms.openlocfilehash: b7bebfb227de3f9f1c51024845054d2d7a02f923
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87285652"
---
# <a name="create-a-map"></a>Creación de un mapa

En este artículo se muestran formas de crear y animar un mapa.  

## <a name="loading-a-map"></a>Carga de un mapa

Para cargar una asignación, cree una nueva instancia de la [clase Map](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map). Al inicializar el mapa, se pasa un identificador de elemento DIV para representar el mapa y un conjunto de opciones que se van a usar cuando se cargue este mapa. Si no se especifica la información de autenticación predeterminada en el espacio de nombres `atlas`, esta información tendrá que especificarse en las opciones del mapa cuando se cargue el mapa. El mapa carga varios recursos de forma asincrónica para mayor rendimiento. Por lo tanto, después de crear la instancia del mapa, asocie un evento `ready` o `load` al mapa y, luego, agregue al controlador de eventos código adicional que interactúe con el mapa. El evento `ready` se activa en cuanto el mapa tiene suficientes recursos cargados para que se interactúe con él mediante programación. El evento `load` se activa después de que la vista inicial del mapa ha terminado de cargarse por completo. 

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Carga básica del mapa" src="//codepen.io/azuremaps/embed/rXdBXx/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder="no" allowtransparency="true" allowfullscreen="true">
Consulte el Pen <a href='https://codepen.io/azuremaps/pen/rXdBXx/'>Basic map load</a> (Carga básica del mapa) de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

> [!TIP]
> Puede cargar varios mapas en la misma página. Cuando hay varios mapas en la misma página, se puede usar la misma configuración de autenticación e idioma u otra diferente.

## <a name="show-a-single-copy-of-the-world"></a>Mostrar una sola copia del mundo

Cuando se aleja el mapa en una pantalla ancha, se mostrarán varias copias del mundo horizontalmente. Esta opción es excelente en algunos escenarios, pero para otras aplicaciones es aconsejable tener una sola copia del mundo. Para implementar este comportamiento se establece la opción `renderWorldCopies` de los mapas en `false`.

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="renderWorldCopies = false" src="//codepen.io/azuremaps/embed/eqMYpZ/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder="no" allowtransparency="true" allowfullscreen="true">
Consulte el Pen <a href='https://codepen.io/azuremaps/pen/eqMYpZ/'>renderWorldCopies = false</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>


## <a name="map-options"></a>Opciones de mapa

Al crear un mapa allí, se pueden pasar varios tipos diferentes de opciones para personalizar cómo se enumeran las funciones de mapa a continuación.

- [CameraOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.cameraoptions) y [CameraBoundOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.cameraboundsoptions) se usan para especificar el área que debe mostrar el mapa.
- El objetivo de [ServiceOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.serviceoptions) es especificar cómo debe interactuar el mapa con los servicios que lo habilitan.
- La finalidad de [StyleOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.styleoptions) es especificar que el mapa se debe representar y definir con estilos.
- La utilidad de [UserInteractionOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.userinteractionoptions) es especificar cómo se debe acceder al mapa cuando el usuario interactúa con él. 

Estas opciones también se pueden actualizar después de cargar el mapa mediante las funciones `setCamera`, `setServiceOptions`, `setStyle` y `setUserInteraction`. 

## <a name="controlling-the-map-camera"></a>Control de la cámara del mapa

Hay dos maneras de establecer el área mostrada del mapa mediante la cámara de un mapa. Puede establecer las opciones de la cámara al cargar el mapa. O bien, puede llamar a la opción `setCamera` en cualquier momento después de que se haya cargado el mapa para actualizar la vista del mapa mediante programación.  

<a id="setCameraOptions"></a>

### <a name="set-the-camera"></a>Establecimiento de la cámara

La cámara del mapa controla lo que se muestra en la ventanilla del lienzo del mapa. Las opciones de cámara se pueden pasar a las opciones de mapa al inicializarse o pasarse a la función `setCamera` de mapas.

```javascript
//Set the camera options when creating the map.
var map = new atlas.Map('map', {
    center: [-122.33, 47.6],
    zoom: 12

    //Additional map options.
};

//Update the map camera at anytime using setCamera function.
map.setCamera({
    center: [-110, 45],
    zoom: 5 
});
```

En el código siguiente, se crea un [objeto Map](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map) y se establecen las opciones de centro y zoom. Las propiedades del mapa, como centro y nivel de zoom, forman parte de [CameraOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.cameraoptions).

<br/>

<iframe height='500' scrolling='no' title='Creación de un mapa mediante CameraOptions' src='//codepen.io/azuremaps/embed/qxKBMN/?height=543&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen <a href='https://codepen.io/azuremaps/pen/qxKBMN/'>Creación de un mapa mediante `CameraOptions` </a>de Azure Location Based Services (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

<a id="setCameraBoundsOptions"></a>

### <a name="set-the-camera-bounds"></a>Establecimiento de los límites de cámara

Se puede usar un cuadro de límite para actualizar la cámara del mapa. Si el cuadro de límite se calculó a partir de datos de punto, a menudo resulta útil especificar también un valor de relleno de píxeles en las opciones de la cámara para tener en cuenta el tamaño del icono. Esto le ayudará a asegurarse de que los puntos no queden fuera del borde de la ventanilla del mapa.

```javascript
map.setCamera({
    bounds: [-122.4, 47.6, -122.3, 47.7],
    padding: 10
});
```

En el código siguiente, se crea un [objeto Map](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map) mediante `new atlas.Map()`. Las propiedades del mapa, como `CameraBoundsOptions`, se pueden definir a través de la función [setCamera](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map) de la clase Map. Los límites y las propiedades de relleno se establecen mediante `setCamera`.

<br/>

<iframe height='500' scrolling='no' title='Creación de un mapa mediante CameraBoundsOptions' src='//codepen.io/azuremaps/embed/ZrRbPg/?height=543&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen <a href='https://codepen.io/azuremaps/pen/ZrRbPg/'>Creación de un mapa mediante `CameraBoundsOptions` </a>de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

### <a name="animate-map-view"></a>Animación de la vista de mapa

Al establecer las opciones de la cámara del mapa, también se pueden establecer las [opciones de animación](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.animationoptions). Estas opciones especifican el tipo de animación y la duración del movimiento de la cámara.

```javascript
map.setCamera({
    center: [-122.33, 47.6],
    zoom: 12,
    duration: 1000,
    type: 'fly'  
});
```

En el código siguiente, el primer bloque de código crea un mapa y establece los estilos de mapa de zoom y entrada. En el segundo bloque de código, se crea un controlador de eventos click para el botón Animate. Cuando se hace clic en este botón, se llama a la función `setCamera` con algunos valores aleatorios para [CameraOptions](/javascript/api/azure-maps-control/atlas.cameraoptions) y [AnimationOptions](/javascript/api/azure-maps-control/atlas.animationoptions).

<br/>

<iframe height='500' scrolling='no' title='Animación de la vista de mapa' src='//codepen.io/azuremaps/embed/WayvbO/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen <a href='https://codepen.io/azuremaps/pen/WayvbO/'>Animación de la vista de mapa</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="try-out-the-code"></a>Prueba del código

Examine los ejemplos de código. Puede editar el código JavaScript dentro de la **pestaña JS** y ver los cambios en la vista del mapa en la **pestaña Resultado**. También puede hacer clic en **Editar en CodePen**, en la esquina superior derecha, y modificar el código en CodePen.

<a id="relatedReference"></a>

## <a name="next-steps"></a>Pasos siguientes

Más información sobre las clases y los métodos utilizados en este artículo:

> [!div class="nextstepaction"]
> [Map](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map)

> [!div class="nextstepaction"]
> [CameraOptions](/javascript/api/azure-maps-control/atlas.cameraoptions)

> [!div class="nextstepaction"]
> [AnimationOptions](/javascript/api/azure-maps-control/atlas.animationoptions)

Vea los siguientes ejemplos de código para agregar funcionalidad a la aplicación:

> [!div class="nextstepaction"]
> [Cambio del estilo del mapa](choose-map-style.md)

> [!div class="nextstepaction"]
> [Incorporación de controles al mapa](map-add-controls.md)

> [!div class="nextstepaction"]
> [Ejemplos de código](https://docs.microsoft.com/samples/browse/?products=azure-maps)
