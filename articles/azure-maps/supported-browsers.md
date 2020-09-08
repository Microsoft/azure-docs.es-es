---
title: Exploradores compatibles con el SDK web | Microsoft Azure Maps
description: Averigüe cómo comprobar si el SDK web de Azure Maps admite un explorador. Vea una lista de los exploradores admitidos. Aprenda a usar los servicios de mapas con exploradores heredados.
author: rbrundritt
ms.author: richbrun
ms.date: 03/25/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.openlocfilehash: e6d4ea19e5547adb7bee25f4bdce295eb9a76d44
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88002930"
---
# <a name="web-sdk-supported-browsers"></a>Exploradores admitidos por el SDK web

El SDK web de Azure Maps proporciona una función auxiliar denominada [atlas.isSupported](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas?view=azure-iot-typescript-latest#issupported-boolean-). Esta función detecta si un explorador web tiene el conjunto mínimo de características de WebGL necesarias para admitir la carga y la representación del control de mapa. Este es un ejemplo de cómo utilizar la función:

```JavaScript
if (!atlas.isSupported()) {
    alert('Your browser is not supported by Azure Maps');
} else if (!atlas.isSupported(true)) {
    alert('Your browser is supported by Azure Maps, but may have major performance caveats.');
} else {
    // Your browser is supported. Add your map code here.
}
```

## <a name="desktop"></a>Escritorio

El SDK web de Azure Maps admite los siguientes exploradores de escritorio:

- Microsoft Edge (versión actual y anterior)
- Google Chrome (versión actual y anterior)
- Mozilla Firefox (versión actual y anterior)
- Apple Safari (Mac OS X) (versión actual y anterior)

Consulte también [Selección de exploradores heredados](#Target-Legacy-Browsers) más adelante en este artículo.

## <a name="mobile"></a>Móvil

El SDK web de Azure Maps admite los siguientes exploradores móviles:

- Android
  - Versión actual de Chrome en Android 6.0 y versiones posteriores
  - Chrome WebView en Android 6.0 y versiones posteriores
- iOS
  - Safari móvil en la versión principal actual y anterior de iOS
  - UIWebView y WKWebView en la versión principal actual y anterior de iOS
  - Versión actual de Chrome para iOS

> [!TIP]
> Si está insertando un mapa en una aplicación móvil mediante un control WebView, puede optar por usar el [paquete npm del SDK web de Azure Maps](https://www.npmjs.com/package/azure-maps-control), en lugar de hacer referencia a la versión del SDK que se hospeda en Azure Content Delivery Network. Este enfoque reduce el tiempo de carga, ya que el SDK ya se encuentra en el dispositivo del usuario y no tiene que descargarse en tiempo de ejecución.

## <a name="nodejs"></a>Node.js

También se admiten los siguientes módulos del SDK web en Node.js:

- Módulo de servicios ([documentación](how-to-use-services-module.md) | [módulo npm](https://www.npmjs.com/package/azure-maps-rest))

## <a name="target-legacy-browsers"></a><a name="Target-Legacy-Browsers"></a>Selección de exploradores heredados

Puede seleccionar exploradores más antiguos que no sean compatibles con WebGL o que solo tengan compatibilidad limitada. En tales casos, se recomienda usar los servicios de Azure Maps junto con un control de mapa de código abierto como [Folleto](https://leafletjs.com/). Este es un ejemplo:

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Azure Maps + Folleto" src="//codepen.io/azuremaps/embed/GeLgyx/?height=500&theme-id=0&default-tab=html,result" frameborder="no" allowtransparency="true" allowfullscreen="true">
Consulte el Pen <a href='https://codepen.io/azuremaps/pen/GeLgyx/'>Azure Maps + Folleto</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>


## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre el SDK web de Azure Maps:

> [!div class="nextstepaction"]
> [Control de mapa](how-to-use-map-control.md)

> [!div class="nextstepaction"]
> [Módulo de servicios](how-to-use-services-module.md)
