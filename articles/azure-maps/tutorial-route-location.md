---
title: 'Tutorial: Búsqueda de la ruta a una ubicación | Microsoft Azure Maps'
description: En este tutorial se muestra cómo representar la ruta a una ubicación (punto de interés) en un mapa mediante el servicio de enrutamiento de Microsoft Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 01/14/2020
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.custom: mvc, devx-track-javascript
ms.openlocfilehash: d6a526e23be47c9e05de0c9064bdb4095a49d132
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87287408"
---
# <a name="tutorial-route-to-a-point-of-interest-using-azure-maps"></a>Tutorial: Ruta a un punto de interés mediante Azure Maps

En este tutorial se muestra cómo usar la cuenta de Azure Maps y el SDK de Route Service para buscar la ruta al punto de interés. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear una nueva página web con Map Control API
> * Establecer las coordenadas de dirección
> * Consultar en Route Service direcciones a un punto de interés

## <a name="prerequisites"></a>Prerrequisitos

Antes de continuar, siga las instrucciones de [Creación de una cuenta](quick-demo-map-app.md#create-an-azure-maps-account), necesitará una suscripción en el plan de tarifa S1. Siga los pasos descritos en [Obtención de la clave principal](quick-demo-map-app.md#get-the-primary-key-for-your-account) para obtener la clave principal de la cuenta. Para más información sobre la autenticación en Azure Maps, consulte [Administración de la autenticación en Azure Maps](how-to-manage-authentication.md).

<a id="getcoordinates"></a>

## <a name="create-a-new-map"></a>Creación de un nuevo mapa

En los pasos siguientes se muestra cómo crear una página HTML estática insertada con Map Control API.

1. En la máquina local, cree un nuevo archivo y asígnele el nombre **MapRoute.html**.
2. Agregue los siguientes componentes HTML al archivo:

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

    Observe que el encabezado HTML incluye los archivos de recursos CSS y JavaScript hospedados por la biblioteca de Control de mapa de Azure. Observe el evento `onload` en el cuerpo de la página, el cual llamará a la función `GetMap` cuando el cuerpo de la página se haya cargado. Esta función contendrá el código JavaScript insertado para acceder a las API de Azure Maps. 

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

    `atlas.Map` proporciona el control para un mapa web visual e interactivo, y es un componente de la API de Control de mapa de Azure.

4. Guarde el archivo y ábralo en el explorador. En este punto, tiene un mapa básico que puede desarrollar aún más.

   ![Visualización del mapa básico](media/tutorial-route-location/basic-map.png)

## <a name="define-how-the-route-will-be-rendered"></a>Definición de cómo se representará la ruta

En este tutorial, se representará una ruta simple mediante un icono de símbolo para el inicio y el final de la ruta, y una línea para el trazado de la ruta.

1. Después de inicializar el mapa, agregue el siguiente código JavaScript.

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
    
    En el controlador de eventos `ready` de los mapas, se crea un origen de datos para almacenar la línea de ruta y los puntos inicial y final. Se crea una capa de línea y se asocia al origen de datos para definir cómo se representará la línea de la ruta. La línea de ruta se representará como un tono azul. Tendrá un ancho de cinco píxeles, uniones de líneas redondeadas y límites. Al agregar la capa al mapa, se pasa un segundo parámetro con el valor `'labels'` que especifica que esta capa se debe representar por debajo de las etiquetas del mapa. Esto garantiza que la línea de ruta no cubrirá las etiquetas de la carretera. Se crea una capa de símbolos y se asocia al origen de datos. Esta capa especifica cómo se representan los puntos inicial y final. En este caso, se han agregado expresiones para recuperar la imagen del icono y la información de la etiqueta de texto a partir de las propiedades de cada objeto de punto. 
    
2. En este tutorial, establezca como punto inicial Microsoft y como punto final una gasolinera de Seattle. En el controlador de eventos `ready` de los mapas, agregue el código siguiente.

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

    Este código crea dos [objetos de punto de GeoJSON](https://en.wikipedia.org/wiki/GeoJSON) para representar los puntos inicial y final de la ruta y agrega estos puntos al origen de datos. Se agrega una propiedad `title` y `icon` a cada punto. El último bloque establece la vista de la cámara con la latitud y longitud de los puntos inicial y final, mediante la propiedad [setCamera](/javascript/api/azure-maps-control/atlas.map#setcamera-cameraoptions---cameraboundsoptions---animationoptions-) del mapa.

3. Guarde el archivo **MapRoute.html** y actualice el explorador. Ahora el mapa se centra en Seattle, y puede ver como la chincheta azul marca el punto inicial y otra chincheta azul redonda, el punto final.

   ![Visualización de los puntos de inicio y fin de las rutas en el mapa](media/tutorial-route-location/map-pins.png)

<a id="getroute"></a>

## <a name="get-directions"></a>Obtención de las direcciones

En esta sección se muestra cómo usar la API del servicio de rutas de Azure Maps. La API del servicio de rutas encuentra la ruta desde un punto de inicio determinado a un punto final. En este servicio, existen diversas API para planear la ruta *más rápida*, *más corta*, *ecológica* o *emocionante* entre dos ubicaciones. Este servicio también permite a los usuarios planear rutas futuras mediante el uso de bases de datos de Azure que contienen un historial amplio del tráfico. Los usuarios pueden ver la predicción de la duración de la ruta en cualquier día y hora elegidos. Para más información, consulte [Obtención de direcciones de ruta](https://docs.microsoft.com/rest/api/maps/route/getroutedirections). Todas las funcionalidades siguientes deben agregarse **dentro del elemento eventListener preparado para mapas** para asegurarse de que se cargan cuando ya se puede acceder a los recursos del mapa.

1. En la función GetMap, agregue lo siguiente al código JavaScript.

    ```JavaScript
    // Use SubscriptionKeyCredential with a subscription key
    var subscriptionKeyCredential = new atlas.service.SubscriptionKeyCredential(atlas.getSubscriptionKey());

    // Use subscriptionKeyCredential to create a pipeline
    var pipeline = atlas.service.MapsURL.newPipeline(subscriptionKeyCredential);

    // Construct the RouteURL object
    var routeURL = new atlas.service.RouteURL(pipeline);
    ```

   `SubscriptionKeyCredential` crea un `SubscriptionKeyCredentialPolicy` para autenticar las solicitudes HTTP en Azure Maps con la clave de suscripción. `atlas.service.MapsURL.newPipeline()` toma la directiva `SubscriptionKeyCredential` y crea una instancia de [canalización](https://docs.microsoft.com/javascript/api/azure-maps-rest/atlas.service.pipeline?view=azure-maps-typescript-latest). `routeURL` representa una dirección URL para las operaciones [Route](https://docs.microsoft.com/rest/api/maps/route) de Azure Maps.

2. Después de configurar las credenciales y la dirección URL, agregue el siguiente código JavaScript para construir la ruta desde el punto inicial hasta el final. `routeURL` solicita a Route Service de Azure Maps que calcule las direcciones de la ruta. Se extrae una colección de características de GeoJSON de la respuesta con el método `geojson.getFeatures()` y se agregan al origen de datos.

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

3. Guarde el archivo **MapRoute.html** y actualice el explorador web. Para conectarse correctamente con las API de Maps, debe ver un mapa similar al siguiente.

    ![Control de mapa y Route Service de Azure](./media/tutorial-route-location/map-route.png)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Crear una nueva página web con Map Control API
> * Establecer las coordenadas de dirección
> * Consultar en el servicio de ruta las direcciones a un punto de interés

> [!div class="nextstepaction"]
> [Ver el código fuente completo](https://github.com/Azure-Samples/AzureMapsCodeSamples/blob/master/AzureMapsCodeSamples/Tutorials/route.html)

> [!div class="nextstepaction"]
> [Ver un ejemplo publicado](https://azuremapscodesamples.azurewebsites.net/?sample=Route%20to%20a%20destination)

En el siguiente tutorial se muestra cómo crear una consulta de ruta con restricciones, como el modo de desplazamiento o el tipo de carga y, después, visualizar varias rutas en el mismo mapa.

> [!div class="nextstepaction"]
> [Búsqueda de rutas para diferentes modos de desplazamiento](./tutorial-prioritized-routes.md)
