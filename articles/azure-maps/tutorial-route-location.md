---
title: 'Tutorial: Cómo mostrar direcciones de ruta mediante el servicio Route de Microsoft Azure Maps y el Control de mapa'
description: Aprenda a mostrar direcciones de ruta mediante el servicio Route de Microsoft Azure Maps y el Control de mapa.
author: anastasia-ms
ms.author: v-stharr
ms.date: 09/01/2020
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.custom: mvc, devx-track-javascript
ms.openlocfilehash: 992640424f6fdb632327866e132fdbb1c6244492
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89400337"
---
# <a name="tutorial-how-to-display-route-directions-using-azure-maps-route-service-and-map-control"></a>Tutorial: Cómo mostrar direcciones de ruta mediante el servicio Route de Azure Maps y el Control de mapa

En este tutorial se muestra cómo usar la [API del servicio Route](https://docs.microsoft.com/rest/api/maps/route) de Azure Maps y el [Control de mapa](https://docs.microsoft.com/azure/azure-maps/how-to-use-map-control) para mostrar las direcciones de ruta de principio a fin. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear y mostrar el Control de mapa en una página web. 
> * Definir la representación de pantalla de la ruta mediante la definición de [capas de símbolo](map-add-pin.md) y [capas de línea](map-add-line-layer.md).
> * Crear y agregar objetos GeoJSON al mapa para representar los puntos inicial y final.
> * Obtener las direcciones de ruta desde los puntos inicial y final mediante [Get Route directions API](https://docs.microsoft.com/rest/api/maps/route/getroutedirections).

Puede obtener el código fuente completo para el ejemplo [aquí](https://github.com/Azure-Samples/AzureMapsCodeSamples/blob/master/AzureMapsCodeSamples/Tutorials/route.html). Se puede encontrar un ejemplo dinámico [aquí](https://azuremapscodesamples.azurewebsites.net/?sample=Route%20to%20a%20destination).

## <a name="prerequisites"></a>Requisitos previos

1. [Cree una cuenta de Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account).
2. [Obtenga una clave de suscripción principal](quick-demo-map-app.md#get-the-primary-key-for-your-account), también conocida como clave principal o clave de suscripción.

<a id="getcoordinates"></a>

## <a name="create-and-display-the-map-control"></a>Creación y muestra del Control de mapa

En los pasos siguientes se muestra cómo crear y mostrar el Control de mapa en una página web.

1. En la máquina local, cree un nuevo archivo y asígnele el nombre **MapRoute.html**.
2. Copie y pegue el siguiente marcado HTML en el archivo.

    ```HTML
    <!DOCTYPE html>
    <html>
    <head>
        <title>Map Route</title>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

        <!-- Add references to the Azure Maps Map control JavaScript and CSS files. -->
        <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css">
        <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js"></script>

        <!-- Add a reference to the Azure Maps Services Module JavaScript file. -->
        <script src="https://atlas.microsoft.com/sdk/javascript/service/2/atlas-service.min.js"></script>

        <script>
            var map, datasource, client;

            function GetMap() {
                //Add Map Control JavaScript code here.
            }
        </script>
        <style>
            html,
            body {
                width: 100%;
                height: 100%;
                padding: 0;
                margin: 0;
            }

            #myMap {
                width: 100%;
                height: 100%;
            }
        </style>
    </head>
    <body onload="GetMap()">
        <div id="myMap"></div>
    </body>
    </html>
    ```

    El encabezado HTML incluye los archivos de recursos CSS y JavaScript hospedados por la biblioteca de Control de mapa de Azure. El evento `onload` del cuerpo llama a la función `GetMap`. En el paso siguiente, agregaremos el código de inicialización del Control de mapa.

3. Agregue el siguiente código JavaScript a la función `GetMap`. Reemplace la cadena `<Your Azure Maps Key>` por la clave principal que copió de la cuenta de Maps.

    ```JavaScript
   //Instantiate a map object
   var map = new atlas.Map("myMap", {
        //Add your Azure Maps subscription key to the map SDK. Get an Azure Maps key at https://azure.com/maps
        authOptions: {
           authType: 'subscriptionKey',
           subscriptionKey: '<Your Azure Maps Key>'
        }
   });
   ```

4. Guarde el archivo y ábralo en el explorador. Se muestra un ejemplo.

     :::image type="content" source="./media/tutorial-route-location/basic-map.png" alt-text="Representación de mapa básica del Control de mapa":::

## <a name="define-route-display-rendering"></a>Definición de la representación de la visualización de ruta

En este tutorial, se representará la ruta mediante una capa de línea. Los puntos inicial y final se representarán mediante una capa de símbolo. Para obtener más información sobre cómo agregar capas de línea, vea [Adición de una capa de línea al mapa](map-add-line-layer.md). Para obtener más información acerca de las capas de símbolo, consulte [Adición de una capa de símbolo a un mapa](map-add-pin.md).

1. Anexe el siguiente código JavaScript a la función `GetMap`. Este código implementa el controlador de eventos `ready` del Control de mapa. El resto del código de este tutorial se colocará dentro del controlador de eventos `ready`.

    ```JavaScript
    //Wait until the map resources are ready.
    map.events.add('ready', function() {

        //Create a data source and add it to the map.
        datasource = new atlas.source.DataSource();
        map.sources.add(datasource);

        //Add a layer for rendering the route lines and have it render under the map labels.
        map.layers.add(new atlas.layer.LineLayer(datasource, null, {
            strokeColor: '#2272B9',
            strokeWidth: 5,
            lineJoin: 'round',
            lineCap: 'round'
        }), 'labels');

        //Add a layer for rendering point data.
        map.layers.add(new atlas.layer.SymbolLayer(datasource, null, {
            iconOptions: {
                image: ['get', 'icon'],
                allowOverlap: true
           },
            textOptions: {
                textField: ['get', 'title'],
                offset: [0, 1.2]
            },
            filter: ['any', ['==', ['geometry-type'], 'Point'], ['==', ['geometry-type'], 'MultiPoint']] //Only render Point or MultiPoints in this layer.
        }));
    });
    ```

    En el controlador de eventos `ready` del Control de mapa, se crea un origen de datos para almacenar la ruta desde los puntos inicial y final. Para definir cómo se representará la línea de la ruta, se crea una capa de línea y se adjunta al origen de datos.  Para asegurarse de que la línea de ruta no cubre las etiquetas de la carretera, hemos pasado un segundo parámetro con el valor de `'labels'`.

    A continuación, se crea una capa de símbolo y se adjunta al origen de datos. Esta capa especifica cómo se representan los puntos inicial y final. En este caso, se han agregado expresiones para recuperar la imagen del icono y la información de la etiqueta de texto a partir de las propiedades de cada objeto de punto.

2. Establezca como punto inicial Microsoft y como punto final una gasolinera de Seattle.  En el controlador de eventos `ready` del Control de mapa, anexe el código siguiente.

    ```JavaScript
    //Create the GeoJSON objects which represent the start and end points of the route.
    var startPoint = new atlas.data.Feature(new atlas.data.Point([-122.130137, 47.644702]), {
        title: "Redmond",
        icon: "pin-blue"
    });

    var endPoint = new atlas.data.Feature(new atlas.data.Point([-122.3352, 47.61397]), {
        title: "Seattle",
        icon: "pin-round-blue"
    });

    //Add the data to the data source.
    datasource.add([startPoint, endPoint]);

    map.setCamera({
        bounds: atlas.data.BoundingBox.fromData([startPoint, endPoint]),
        padding: 80
    });
    ```

    Este código crea dos [objetos de punto de GeoJSON](https://en.wikipedia.org/wiki/GeoJSON) para representar los puntos inicial y final, que se agregan al origen de datos. El último bloque de código establece la vista de la cámara con la latitud y longitud de los puntos inicial y final. Para obtener más información sobre la propiedad setCamera del Control de mapa, vea la propiedad [setCamera(CameraOptions | CameraBoundsOptions & AnimationOptions)](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map?view=azure-maps-typescript-latest#setcamera-cameraoptions---cameraboundsoptions---animationoptions-).

3. Guarde el archivo **MapRoute.html** y actualice el explorador. Ahora el mapa se centra en Seattle. La chincheta azul en forma de lágrima marca el punto inicial. La chincheta azul redonda marca el punto final.

    :::image type="content" source="./media/tutorial-route-location/map-pins.png" alt-text="Visualización de los puntos de inicio y fin de las rutas en el mapa":::

<a id="getroute"></a>

## <a name="get-route-directions"></a>Obtención de direcciones de ruta

En esta sección se muestra cómo usar la API del servicio Route de Azure Maps para obtener direcciones de un punto a otro. En este servicio, existen otras API que le permiten planear la ruta *más rápida*, *más corta*, *ecológica* o *emocionante* entre dos ubicaciones. Este servicio también permite a los usuarios planear rutas futuras en función de las condiciones del tráfico del historial. Los usuarios pueden ver la predicción de las duraciones de ruta en un momento dado. Para más información, consulte [Get Route directions API](https://docs.microsoft.com/rest/api/maps/route/getroutedirections).

1. En la función `GetMap`, dentro del controlador de eventos `ready` del control, agregue lo siguiente al código de JavaScript.

    ```JavaScript
    // Use SubscriptionKeyCredential with a subscription key
    var subscriptionKeyCredential = new atlas.service.SubscriptionKeyCredential(atlas.getSubscriptionKey());

    // Use subscriptionKeyCredential to create a pipeline
    var pipeline = atlas.service.MapsURL.newPipeline(subscriptionKeyCredential);

    // Construct the RouteURL object
    var routeURL = new atlas.service.RouteURL(pipeline);
    ```

   `SubscriptionKeyCredential` crea un `SubscriptionKeyCredentialPolicy` para autenticar las solicitudes HTTP en Azure Maps con la clave de suscripción. `atlas.service.MapsURL.newPipeline()` toma la directiva `SubscriptionKeyCredential` y crea una instancia de [canalización](https://docs.microsoft.com/javascript/api/azure-maps-rest/atlas.service.pipeline?view=azure-maps-typescript-latest). `routeURL` representa una dirección URL para las operaciones [Route](https://docs.microsoft.com/rest/api/maps/route) de Azure Maps.

2. Después de configurar las credenciales y la dirección URL, anexe el código siguiente en el controlador de eventos `ready` del control. Este código crea la ruta desde el punto inicial hasta el punto final. `routeURL` solicita a la API del servicio Route de Azure Maps que calcule las direcciones de la ruta. Se extrae una colección de características de GeoJSON de la respuesta con el método `geojson.getFeatures()` y se agregan al origen de datos.

    ```JavaScript
    //Start and end point input to the routeURL
    var coordinates= [[startPoint.geometry.coordinates[0], startPoint.geometry.coordinates[1]], [endPoint.geometry.coordinates[0], endPoint.geometry.coordinates[1]]];

    //Make a search route request
    routeURL.calculateRouteDirections(atlas.service.Aborter.timeout(10000), coordinates).then((directions) => {
        //Get data features from response
        var data = directions.geojson.getFeatures();
        datasource.add(data);
    });
    ```

3. Guarde el archivo **MapRoute.html** y actualice el explorador web. El mapa ya debería mostrar la ruta desde el punto inicial hasta el final.

     :::image type="content" source="./media/tutorial-route-location/map-route.png" alt-text="Control de mapa y servicio Route de Azure":::

    Puede obtener el código fuente completo para el ejemplo [aquí](https://github.com/Azure-Samples/AzureMapsCodeSamples/blob/master/AzureMapsCodeSamples/Tutorials/route.html). Se puede encontrar un ejemplo dinámico [aquí](https://azuremapscodesamples.azurewebsites.net/?sample=Route%20to%20a%20destination).

## <a name="next-steps"></a>Pasos siguientes

En el siguiente tutorial se muestra cómo crear una consulta de ruta con restricciones, como el modo de desplazamiento o el tipo de carga. Después, puede mostrar varias rutas en el mismo mapa.

> [!div class="nextstepaction"]
> [Búsqueda de rutas para diferentes modos de desplazamiento](./tutorial-prioritized-routes.md)
