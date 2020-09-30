---
title: Cambio del estilo del Control de mapa web de Azure Maps
description: Aprenda a cambiar el estilo y las opciones de un mapa. Vea cómo agregar un control selector de estilo a un mapa de Azure Maps para que los usuarios puedan cambiar entre distintos estilos.
author: anastasia-ms
ms.author: v-stharr
ms.date: 07/27/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: devx-track-javascript
ms.openlocfilehash: fe07d6d54296a3c55cb0b2cadb4d4ae0a311873b
ms.sourcegitcommit: 07166a1ff8bd23f5e1c49d4fd12badbca5ebd19c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90089571"
---
# <a name="change-the-style-of-the-map"></a>Cambio del estilo del mapa

El control de mapa admite varias [opciones de estilo](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.styleoptions) de mapa y [estilos de mapa base](supported-map-styles.md) diferentes. Todos los estilos se pueden establecer cuando se inicializa el control de mapa. O bien puede establecer estilos si usa la función `setStyle` del control de mapa. En este artículo se muestra cómo usar estas opciones de estilo para personalizar la apariencia de los mapas. Además, aprenderá a implementar el control selector de estilo en el mapa. El control selector de estilo permite al usuario alternar entre distintos estilos base.

## <a name="set-map-style-options"></a>Establecimiento de opciones de estilo de mapa

Las opciones de estilo se pueden establecer durante la inicialización del control web. O bien puede actualizar las opciones de estilo si llama a la función `setStyle` del control de mapa. Para ver todas las opciones de estilo disponibles, consulte las [opciones de estilo](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.styleoptions).

```javascript
//Set the style options when creating the map.
var map = new atlas.Map('map', {
    renderWorldCopies: false,
    showBuildingModels: false,
    showLogo = true,
    showFeedbackLink = true,
    style='road'

    //Additional map options.
};

//Update the style options at anytime using `setStyle` function.
map.setStyle({
    renderWorldCopies: true,
    showBuildingModels: true,
    showLogo = false,
    showFeedbackLink = false
});
```

La siguiente herramienta muestra el modo en que las distintas opciones de estilo cambian la manera en que el mapa se representa. Para ver los edificios en 3D, amplíe mucho una gran urbe.

<br/>

<iframe height="700" style="width: 100%;" scrolling="no" title="Opciones de estilo de mapa" src="https://codepen.io/azuremaps/embed/eYNMjPb?height=700&theme-id=0&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
Consulte el lápiz <a href='https://codepen.io/azuremaps/pen/eYNMjPb'>Map style options</a> (Opciones de estilo de mapa) de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="set-a-base-map-style"></a>Establecimiento de un estilo de mapa base

También puede inicializar el control de mapa con uno de los [estilos de mapa base](supported-map-styles.md) disponibles en el SDK web. Después puede usar la función `setStyle` para actualizar el estilo base con un estilo de mapa distinto.

### <a name="set-a-base-map-style-on-initialization"></a>Establecimiento de un estilo de mapa base en la inicialización

Los estilos base del control de mapa se pueden establecer durante la inicialización. En el código siguiente, la opción `style` del control de mapa se establece en el [estilo de mapa base `grayscale_dark`](supported-map-styles.md#grayscale_dark).  

```javascript
var map = new atlas.Map('map', {
    style: 'grayscale_dark',

    //Additional map options
);
```

<br/>

<iframe height='500' scrolling='no' title='Establecer el estilo al cargar un mapa' src='//codepen.io/azuremaps/embed/WKOQRq/?height=265&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen <a href='https://codepen.io/azuremaps/pen/WKOQRq/'>Setting the style on map load</a> (Establecer el estilo al cargar un mapa) de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

### <a name="update-the-base-map-style"></a>Actualización del estilo de mapa base

El estilo de mapa base se puede actualizar si usa la función `setStyle` y establece la opción `style` en cambiar a un estilo de mapa base distinto o agregar opciones de estilo adicionales.

```javascript
map.setStyle({ style: 'satellite' });
```

En el código siguiente, después de cargar una instancia de mapa, el estilo de mapa se actualiza de `grayscale_dark` a `satellite` con la función [setStyle](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map#setstyle-styleoptions-).

<br/>

<iframe height='500' scrolling='no' title='Actualizar el estilo' src='//codepen.io/azuremaps/embed/yqXYzY/?height=265&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen <a href='https://codepen.io/azuremaps/pen/yqXYzY/'>Updating the style</a> (Actualizar el estilo) de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="add-the-style-picker-control"></a>Incorporación del control selector de estilos

El control de selector de estilos proporciona un botón con panel flotante fácil de usar que el usuario final puede emplear para cambiar entre los estilos base.

El selector de estilos tiene dos opciones de diseño distintas: `icon` y `list`. Además, el selector de estilos permite elegir dos opciones `style` de control de estilos distintas: `light` y `dark`. En este ejemplo, el selector de estilo usa el diseño `icon` y muestra una lista de selección de los estilos de mapa base en forma de iconos. El selector de control de estilos incluye el conjunto de estilos base siguientes: `["road", "grayscale_light", "grayscale_dark", "night", "road_shaded_relief"]`. Para más información sobre las opciones del control selector de estilos, consulte [Opciones de control de estilo](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.stylecontroloptions).

En la imagen siguiente se muestra el control selector de estilos que aparece en el diseño `icon`.

:::image type="content" source="./media/choose-map-style/style-picker-icon-layout.png" alt-text="Diseño de iconos del selector de estilos":::

En la imagen siguiente se muestra el control selector de estilos que aparece en el diseño `list`.

:::image type="content" source="./media/choose-map-style/style-picker-list-layout.png" alt-text="Diseño de lista del selector de estilos":::

> [!IMPORTANT]
> De forma predeterminada, el control selector de estilos muestra todos los estilos disponibles bajo el plan de tarifa S0 de Azure Maps. Si desea reducir el número de estilos de esta lista, pase una matriz de los estilos que desea que aparezcan en la lista a la opción `mapStyle` del selector de estilo. Si está utilizando S1 y desea mostrar todos los estilos disponibles, establezca la opción `mapStyles` del selector de estilo en `"all"`.

En el código siguiente se muestra cómo reemplazar la lista de estilos base `mapStyles` predeterminada. En este ejemplo, estableceremos la opción `mapStyles` para ver los estilos base que queremos que se muestren con el control selector de estilos.

<br/>

<iframe height='500' scrolling='no' title='Agregar el selector de estilos' src='//codepen.io/azuremaps/embed/OwgyvG/?height=265&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el Pen <a href='https://codepen.io/azuremaps/pen/OwgyvG/'>Adding the style picker</a> (Agregar el selector de estilos) de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="next-steps"></a>Pasos siguientes

Más información sobre las clases y los métodos utilizados en este artículo:

> [!div class="nextstepaction"]
> [Map](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map)

> [!div class="nextstepaction"]
> [StyleOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.styleoptions)

> [!div class="nextstepaction"]
> [StyleControl](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.control.stylecontrol)

> [!div class="nextstepaction"]
> [StyleControlOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.stylecontroloptions)

Adición de controles a los mapas:

> [!div class="nextstepaction"]
> [Incorporación de controles de mapa](map-add-controls.md)

> [!div class="nextstepaction"]
> [Adición de un anclaje](map-add-pin.md)
