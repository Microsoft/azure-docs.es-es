---
title: Uso del módulo de mapas de Azure Maps Indoor
description: Obtenga información sobre cómo usar el módulo de mapas de Microsoft Azure Maps Indoor para representar mapas mediante la inserción de las bibliotecas de JavaScript del módulo.
author: anastasia-ms
ms.author: v-stharr
ms.date: 07/20/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: devx-track-javascript
ms.openlocfilehash: b9ec42620ee5ffaaf5fd79da5dabc944fc3bc422
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87287088"
---
# <a name="use-the-azure-maps-indoor-maps-module"></a>Uso del módulo de mapas de Azure Maps Indoor

El SDK web de Azure Maps incluye el módulo de *Azure Maps Indoor*. El módulo de *Azure Maps Indoor* permite representar planos interiores creados en Azure Maps Creator.

## <a name="prerequisites"></a>Requisitos previos

1. [Cree una cuenta de Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account).
2. [Cree un recurso de Creator](how-to-manage-creator.md).
3. [Obtenga una clave de suscripción principal](quick-demo-map-app.md#get-the-primary-key-for-your-account), también conocida como clave principal o clave de suscripción.
4. Complete el [tutorial de creación de planos interiores](tutorial-creator-indoor-maps.md) para obtener `tilesetId` y `statesetId`.
 Deberá utilizar estos identificadores para representar planos interiores con el módulo de mapas de Azure Maps Indoor.

## <a name="embed-the-indoor-maps-module"></a>Inserción del módulo de mapas de Indoor

Puede instalar e insertar el módulo de *Azure Maps Indoor* de una de estas dos maneras.

Para usar la versión de Azure Content Delivery Network hospedada globalmente del módulo de *Azure Maps Indoor*, consulte las siguientes referencias de la hoja de estilos y JavaScript en el elemento `<head>` del archivo HTML:

```html
<script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js"></script>
<script src="https://atlas.microsoft.com/sdk/javascript/indoor/0.1/atlas-indoor.min.js"></script>
<link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css" />
<link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/indoor/0.1/atlas-indoor.min.css" type="text/css"/>
```

 También puede descargar el módulo de *Azure Maps Indoor*. El módulo de *Azure Maps Indoor* contiene una biblioteca cliente para tener acceso a los servicios de Azure Maps. Siga los pasos que se indican a continuación para instalar y cargar el módulo de *Indoor* en la aplicación web.  
  
  1. Descargue el [paquete azure-maps-indoor](https://www.npmjs.com/package/azure-maps-indoor).
  
      ```powershell
      >npm install azure-maps-control
      >npm install azure-maps-indoor
      ```

  2. Haga referencia a la hoja de estilos y JavaScript del módulo de *Azure Maps Indoor* en el elemento `<head>` del archivo HTML:

      ```html
      <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css" />
      <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/indoor/0.1/atlas-indoor.min.css" type="text/css"/>
      ```

## <a name="instantiate-the-map-object"></a>Creación de una instancia del objeto de mapa

En primer lugar, cree un *objeto de mapa*. El *objeto de mapa* se usará en el paso siguiente para crear una instancia del objeto de *Indoor Manager*.  En el código siguiente se muestra cómo crear una instancia del *objeto de mapa*:

```javascript
const subscriptionKey = "<Your Azure Maps Primary Subscription Key>";

const map = new atlas.Map("map-id", {
  //use your facility's location
  center: [-122.13315, 47.63637],
  //or, you can use bounds: [# west, # south, # east, # north] and replace # with your map's bounds
  style: "blank",
  view: 'Auto',
  authOptions: {
      authType: 'subscriptionKey',
      subscriptionKey: subscriptionKey
  },
  zoom: 19,
});
```

## <a name="instantiate-the-indoor-manager"></a>Creación de una instancia de Indoor Manager

Para cargar el conjunto de mosaicos de interior y el estilo de mapa de los mosaicos, debe crear una instancia de *Indoor Manager*. Para crear una instancia de *Indoor Manager*, proporcione el *objeto de mapa* y el `tilesetId` correspondiente. Si quiere admitir el [estilo de mapa dinámico](indoor-map-dynamic-styling.md), debe pasar el `statesetId`. El nombre de la variable `statesetId` distingue mayúsculas de minúsculas. El código debe ser como el JavaScript siguiente.

```javascript
const tilesetId = "";
const statesetId = "";

const indoorManager = new atlas.indoor.IndoorManager(map, {
    tilesetId: "<tilesetId>",
    statesetId: "<statesetId>" // Optional
});
```

Para habilitar el sondeo de los datos de estado que facilite, debe proporcionar `statesetId` y llamar a `indoorManager.setDynamicStyling(true)`. El sondeo de los datos de estado le permite actualizar dinámicamente el estado de las propiedades dinámicas o los *estados*. Por ejemplo, una característica como una sala puede tener una propiedad dinámica (*estado*) denominada `occupancy`. Es posible que la aplicación quiera sondear los cambios de *estado* para reflejar el cambio en el mapa visual. En el código siguiente se muestra cómo habilitar el sondeo de estado:

```javascript
const tilesetId = "";
const statesetId = "";

const indoorManager = new atlas.indoor.IndoorManager(map, {
    tilesetId: "<tilesetId>",
    statesetId: "<statesetId>" // Optional
});

if (statesetId.length > 0) {
    indoorManager.setDynamicStyling(true);
}
```

## <a name="indoor-level-picker-control"></a>Control de selector de nivel de Indoor

 El control de *selector de nivel de Indoor* le permite cambiar el nivel del mapa representado. Opcionalmente, puede inicializar el control de *selector de nivel de Indoor* a través de *Indoor Manager*. Este es el código para inicializar el selector de control de nivel:

```javascript
const levelControl = new atlas.control.LevelControl({ position: "top-right" });
indoorManager.setOptions({ levelControl });
```

## <a name="indoor-events"></a>Eventos de Indoor

 El módulo de *Azure Maps Indoor* admite eventos de *objeto de mapa*. Los clientes de escucha de eventos de *objeto de mapa* se invocan cuando cambia un nivel o una instalación. Si quiere ejecutar código cuando cambie un nivel o una instalación, coloque el código dentro del cliente de escucha de eventos. En el código siguiente se muestra cómo se pueden agregar clientes de escucha de eventos al *objeto de mapa*.

```javascript
map.events.add("levelchanged", indoorManager, (eventData) => {

  //code that you want to run after a level has been changed
  console.log("The level has changed: ", eventData);
});

map.events.add("facilitychanged", indoorManager, (eventData) => {

  //code that you want to run after a facility has been changed
  console.log("The facility has changed: ", eventData);
});
```

La variable `eventData` contiene información sobre el nivel o la instalación que invocó el evento `levelchanged` o `facilitychanged`, respectivamente. Cuando un nivel cambia, el objeto `eventData` incluye `facilityId`, el nuevo `levelNumber` y otros metadatos. Cuando una instalación cambia, el objeto `eventData` incluye el nuevo `facilityId`, el nuevo `levelNumber` y otros metadatos.

## <a name="example-use-the-indoor-maps-module"></a>Ejemplo: Uso del módulo de mapas de Indoor

En este ejemplo se muestra cómo usar el módulo de *Azure Maps Indoor* en la aplicación web. Aunque el ejemplo está limitado en el ámbito, trata los aspectos básicos de lo que necesita para empezar a usar el módulo de *Azure Maps Indoor*. El código HTML completo está después de estos pasos.

1. Use la [opción](#embed-the-indoor-maps-module) de Azure Content Delivery Network para instalar el módulo de *Azure Maps Indoor*.

2. Cree un archivo HTML.

3. En el encabezado HTML, haga referencia a los estilos de la hoja de estilos y JavaScript del módulo de *Azure Maps Indoor*.

4. Inicialice un *objeto de mapa*. El *objeto de mapa* admite las siguientes opciones:
    - `Subscription key` es la clave de suscripción principal de Azure Maps.
    - `center` define la latitud y la longitud de la ubicación del centro del plano interior. Proporcione un valor para `center` si no quiere proporcionar un valor para `bounds`. El formato debe mostrarse como `center`: [-122.13315, 47.63637].
    - `bounds` es la forma rectangular más pequeña que incluye los datos del mapa de conjunto de mosaicos. Establezca un valor para `bounds` si no quiere establecer un valor para `center`. Puede buscar los límites del mapa mediante una llamada a [Tileset List API](https://docs.microsoft.com/rest/api/maps/tileset/listpreview). Tileset List API devuelve `bbox`, que puede analizar y asignar a `bounds`. El formato debe aparecer como `bounds`: [# west, # south, # east, # north].
    - `style` le permite establecer el color del fondo. Para mostrar un fondo blanco, defina `style` como "en blanco".
    - `zoom` permite especificar los niveles mínimo y máximo de zoom para el mapa.

5. A continuación, cree el módulo de *Indoor Manager*. Asigne `tilesetId` de *Azure Maps Indoor* y, opcionalmente, agregue `statesetId`.

6. Cree una instancia del control de *selector de nivel de Indoor*.

7. Agregue clientes de escucha de eventos de *objeto de mapa*.  

Ahora el archivo debería ser similar al HTML siguiente.

  ```html
  <!DOCTYPE html>
  <html lang="en">
    <head>
      <meta charset="utf-8" />
      <meta name="viewport" content="width=device-width, user-scalable=no" />
      <title>Indoor Maps App</title>
      
      <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css" />
      <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/indoor/0.1/atlas-indoor.min.css" type="text/css"/>

      <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js"></script>
      <script src="https://atlas.microsoft.com/sdk/javascript/indoor/0.1/atlas-indoor.min.js"></script>
        
      <style>
        html,
        body {
          width: 100%;
          height: 100%;
          padding: 0;
          margin: 0;
        }

        #map-id {
          width: 100%;
          height: 100%;
        }
      </style>
    </head>

    <body>
      <div id="map-id"></div>
      <script>
        const subscriptionKey = "<Your Azure Maps Primary Subscription Key>";
        const tilesetId = "<your tilesetId>";
        const statesetId = "<your statesetId>";

        const map = new atlas.Map("map-id", {
          //use your facility's location
          center: [-122.13315, 47.63637],
          //or, you can use bounds: [# west, # south, # east, # north] and replace # with your Map bounds
          style: "blank",
          view: 'Auto',
          authOptions: { 
              authType: 'subscriptionKey',
              subscriptionKey: subscriptionKey
          },
          zoom: 19,
        });

        const levelControl = new atlas.control.LevelControl({
          position: "top-right",
        });

        const indoorManager = new atlas.indoor.IndoorManager(map, {
          levelControl, //level picker
          tilesetId,
          statesetId, //optional
        });

        if (statesetId.length > 0) {
          indoorManager.setDynamicStyling(true);
        }

        map.events.add("levelchanged", indoorManager, (eventData) => {
          //put code that runs after a level has been changed
          console.log("The level has changed:", eventData);
        });

        map.events.add("facilitychanged", indoorManager, (eventData) => {
          //put code that runs after a facility has been changed
          console.log("The facility has changed:", eventData);
        });
      </script>
    </body>
  </html>
  ```

Para ver el plano interior, cárguelo en un explorador web. Debería mostrarse la imagen siguiente. Si hace clic en la característica de hueco de escalera, el *selector de nivel* aparecerá en la esquina superior derecha.

  ![imagen de plano interior](media/how-to-use-indoor-module/indoor-map-graphic.png)

## <a name="next-steps"></a>Pasos siguientes

Obtenga información acerca de las API relacionadas con el módulo de *Azure Maps Indoor*:

> [!div class="nextstepaction"]
> [Requisitos de paquetes de dibujos](drawing-requirements.md)

>[!div class="nextstepaction"]
> [Creator para planos interiores](creator-indoor-maps.md)

Obtenga más información sobre cómo agregar más datos al mapa:

> [!div class="nextstepaction"]
> [Estilo dinámico de planos interiores](indoor-map-dynamic-styling.md)

> [!div class="nextstepaction"]
> [Ejemplos de código](https://docs.microsoft.com/samples/browse/?products=azure-maps)