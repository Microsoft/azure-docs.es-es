---
title: Visualización del tráfico en un mapa | Microsoft Azure Maps
description: En este artículo aprenderá cómo mostrar los datos de tráfico en un mapa mediante el SDK web de Microsoft Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 07/29/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: ''
ms.custom: codepen, devx-track-javascript
ms.openlocfilehash: 5bb088c4650cfdfec59383a7bcf8f340952e5d55
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87273021"
---
# <a name="show-traffic-on-the-map"></a>Visualización del tráfico en el mapa

Hay dos tipos de datos de tráfico disponibles en Azure Maps:

- Datos de incidentes, que se componen de datos basados en puntos y líneas para aspectos como construcción, cierres de carreteras y accidentes.
- Datos de flujo, que proporcionan métricas sobre el flujo de tráfico en las carreteras. A menudo, los datos de flujo de tráfico se utilizan para colorear las carreteras. Los colores se basan en la cantidad de tráfico que ralentiza el flujo en relación con el límite de velocidad u otra métrica. Los datos de flujo de tráfico en Azure Maps tienen tres métricas de medida diferentes:
    - `relative`: está relacionada con la velocidad de flujo libre de la carretera.
    - `absolute`: es la velocidad absoluta de todos los vehículos de la carretera.
    - `relative-delay`: muestra las áreas que son más lentas que el retraso promedio esperado.

En el código siguiente se muestra cómo mostrar los datos de tráfico en el mapa.

```javascript
//Show traffic on the map using the traffic options.
map.setTraffic({
    incidents: true,
    flow: 'relative'
});
```

A continuación se muestra el código de ejemplo de ejecución completo de la funcionalidad anterior.

<br/>

<iframe height='500' scrolling='no' title='Visualización del tráfico en un mapa' src='//codepen.io/azuremaps/embed/WMLRPw/?height=500&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>Consulte el fragmento de código (pen) <a href='https://codepen.io/azuremaps/pen/WMLRPw/'>Mostrar tráfico en un mapa</a> por Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="traffic-overlay-options"></a>Opciones de superposición de tráfico

La siguiente herramienta le permite cambiar entre las diferentes configuraciones de superposición de tráfico para ver cómo cambia la representación. 

<br/>

<iframe height="700" style="width: 100%;" scrolling="no" title="Opciones de superposición de tráfico" src="//codepen.io/azuremaps/embed/RwbPqRY/?height=700&theme-id=0&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
Consulte el fragmento de código (pen) <a href='https://codepen.io/azuremaps/pen/RwbPqRY/'>Opciones de superposición de tráfico</a> de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

## <a name="next-steps"></a>Pasos siguientes

Más información sobre las clases y los métodos utilizados en este artículo:

> [!div class="nextstepaction"]
> [Map](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map)

> [!div class="nextstepaction"]
> [TrafficOptions](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.trafficoptions)

Mejore sus experiencias de usuario:

> [!div class="nextstepaction"]
> [Interacción del mapa con eventos del mouse](map-events.md)

> [!div class="nextstepaction"]
> [Building an accessible map](map-accessibility.md) (Creación de un mapa accesible)

> [!div class="nextstepaction"]
> [Página de ejemplos de código](https://aka.ms/AzureMapsSamples)
