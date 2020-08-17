---
title: 'Tutorial: Búsqueda de ubicaciones cercanas en un mapa | Microsoft Azure Maps'
description: En este tutorial aprenderá a buscar puntos de interés en un mapa mediante Microsoft Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 1/15/2020
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.custom: mvc, devx-track-javascript
ms.openlocfilehash: 4e16c4e88d749f6dbc4f6271a7ceaf77661a208c
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87281538"
---
# <a name="tutorial-search-nearby-points-of-interest-using-azure-maps"></a>Tutorial: Búsqueda de puntos cercanos de interés mediante Azure Maps

En este tutorial se muestra cómo configurar una cuenta con Azure Maps y luego usar las API de Maps para buscar un punto de interés. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear una cuenta de Azure Maps
> * Recuperar la clave principal de la cuenta de Maps
> * Crear una nueva página web con Map Control API
> * Usar el servicio de búsqueda de Maps para encontrar un punto cercano de interés

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en [Azure Portal](https://portal.azure.com).

<a id="createaccount"></a>

## <a name="create-an-account-with-azure-maps"></a>Crear una cuenta con Azure Maps

Cree una nueva cuenta de Maps con los pasos siguientes:

1. En la esquina superior izquierda de [Azure Portal](https://portal.azure.com), haga clic en **Crear un recurso**.
2. En el campo *Buscar en el Marketplace*, escriba **Maps**.
3. En *Resultados*, seleccione **Maps**. Haga clic en el botón **Crear** que aparece debajo del mapa.
4. En la página **Create Maps Account** (Crear una cuenta de Azure Maps), escriba los siguientes valores:
    * La *suscripción* que quiere usar para esta cuenta.
    * El nombre del *grupo de recursos* para esta cuenta. Puede elegir *Crear nuevo* o *Usar existente* para el grupo de recursos.
    * El *nombre* de la nueva cuenta.
    * El *Plan de tarifa* de la cuenta.
    * Lea la *licencia* y la *declaración de privacidad* y active la casilla para aceptar los términos.
    * Haga clic en el botón **Crear**.

![Creación de una cuenta de Azure Maps en Azure Portal](./media/tutorial-search-location/create-account.png)

<a id="getkey"></a>

## <a name="get-the-primary-key-for-your-account"></a>Obtención de la clave principal de una cuenta

Una vez que se haya creado correctamente la cuenta de Maps, recupere la clave que le permite consultar las API de Maps. Se recomienda el uso de la clave principal de la cuenta como clave de suscripción al llamar a los servicios Azure Maps.

1. Abra su cuenta de Maps en el portal.
2. En la sección de configuración, seleccione **Autenticación**.
3. Copie la **clave principal** al Portapapeles. Guárdela localmente para usarla más adelante en este tutorial.

![Obtención de la clave principal en Azure Portal](./media/tutorial-search-location/get-key.png)

Para más información sobre la autenticación en Azure Maps, consulte [Administración de la autenticación en Azure Maps](how-to-manage-authentication.md).

<a id="createmap"></a>

## <a name="create-a-new-map"></a>Creación de un nuevo mapa

La API de Control de mapa es una práctica biblioteca cliente. Esta API permite integrar fácilmente Azure Maps en la aplicación web. Oculta la complejidad de las llamadas sin servicio de REST e impulsa la productividad con componentes que se pueden personalizar. En los pasos siguientes se muestra cómo crear una página HTML estática insertada con Map Control API.

1. En el equipo local, cree un nuevo archivo y asígnele el nombre **MapSearch.html**.
2. Agregue los siguientes componentes HTML al archivo:

   ```HTML
    <!DOCTYPE html>
    <html>
    <head>
        <title>Map Search</title>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

        <!-- Add references to the Azure Maps Map control JavaScript and CSS files. -->
        <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css">
        <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js"></script>

        <!-- Add a reference to the Azure Maps Services Module JavaScript file. -->
        <script src="https://atlas.microsoft.com/sdk/javascript/service/2/atlas-service.min.js"></script>

        <script>
        function GetMap(){
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

   Observe que el encabezado HTML incluye los archivos de recursos CSS y JavaScript hospedados por la biblioteca de Control de mapa de Azure. Observe el evento `onload` en el cuerpo de la página, el cual llamará a la función `GetMap` cuando el cuerpo de la página se haya cargado. La función `GetMap` contendrá el código JavaScript insertado para acceder a las API de Azure Maps.

3. Agregue el siguiente código JavaScript a la función `GetMap` del archivo HTML. Reemplace la cadena `<Your Azure Maps Key>` por la clave principal que copió de la cuenta de Maps.

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

   Este segmento inicializa Map Control API para la clave de cuenta de Azure Maps. `atlas` es el espacio de nombres que contiene la API y los componentes visuales relacionados. `atlas.Map` proporciona el control para un mapa web visual e interactivo.

4. Guarde los cambios en el archivo y abra la página HTML en un explorador. El mapa mostrado es el más básico que puede crear con una llamada a `atlas.Map` mediante la clave de cuenta.

   ![Visualización del mapa](./media/tutorial-search-location/basic-map.png)

5. En la función `GetMap`, después de inicializar el mapa, agregue el siguiente código JavaScript.

    ```JavaScript
    //Wait until the map resources are ready.
    map.events.add('ready', function() {

        //Create a data source and add it to the map.
        datasource = new atlas.source.DataSource();
        map.sources.add(datasource);

        //Add a layer for rendering point data.
        var resultLayer = new atlas.layer.SymbolLayer(datasource, null, {
            iconOptions: {
                image: 'pin-round-darkblue',
                anchor: 'center',
                allowOverlap: true
            },
            textOptions: {
                anchor: "top"
            }
        });

        map.layers.add(resultLayer);
    });
    ```

   En este segmento de código se agrega un evento `ready` al mapa que se activará cuando los recursos del mapa se hayan cargado y se pueda acceder al mapa. En el controlador de eventos `ready` del mapa, se crea un origen de datos para almacenar datos de resultados. Se crea una capa de símbolos y se asocia al origen de datos. Esta capa especifica cómo se deben representar los datos de los resultados en el origen de datos. En este caso, el resultado se representa con un icono de chincheta redonda de color azul oscuro, centrado sobre la coordenada de resultados y que permite que otros iconos se superpongan. La capa resultante se agrega a las capas de mapa.

<a id="usesearch"></a>

## <a name="add-search-capabilities"></a>Adición de funcionalidades de búsqueda

En esta sección se muestra cómo usar [Search API](https://docs.microsoft.com/rest/api/maps/search) de Maps para buscar un punto de interés en el mapa. Se trata de una API RESTful diseñada para los desarrolladores para buscar direcciones, puntos de interés y otra información geográfica. El servicio Search asigna información de latitud y longitud a una dirección especificada. El **módulo de servicio** que se explica a continuación, puede usarse para buscar una ubicación mediante Maps Search API.

### <a name="service-module"></a>Módulo de servicio

1. En el controlador de eventos `ready` del mapa, construya la URL del servicio de búsqueda; para ello, agregue el siguiente código de Javascript.

    ```JavaScript
   // Use SubscriptionKeyCredential with a subscription key
   var subscriptionKeyCredential = new atlas.service.SubscriptionKeyCredential(atlas.getSubscriptionKey());

   // Use subscriptionKeyCredential to create a pipeline
   var pipeline = atlas.service.MapsURL.newPipeline(subscriptionKeyCredential);

   // Construct the SearchURL object
   var searchURL = new atlas.service.SearchURL(pipeline); 
   ```

   `SubscriptionKeyCredential` crea un `SubscriptionKeyCredentialPolicy` para autenticar las solicitudes HTTP en Azure Maps con la clave de suscripción. `atlas.service.MapsURL.newPipeline()` toma la directiva `SubscriptionKeyCredential` y crea una instancia de [canalización](https://docs.microsoft.com/javascript/api/azure-maps-rest/atlas.service.pipeline?view=azure-maps-typescript-latest). `searchURL` representa una dirección URL para las operaciones [Search](https://docs.microsoft.com/rest/api/maps/search) de Azure Maps.

2. A continuación, agregue el siguiente bloque de script para compilar la consulta de búsqueda. Este fragmento de código usa Fuzzy Search Service, que es una API de búsqueda básica de Search Service. Fuzzy Search Service controla la mayoría de las entradas aproximadas como, por ejemplo, direcciones, lugares y puntos de interés (POI). Este código busca las gasolineras más cercanas en un radio especificado de las coordenadas de latitud y longitud proporcionadas. Después se extrae una colección de características GeoJSON de la respuesta con el método `geojson.getFeatures()` y se agregan al origen de datos, lo cual provoca que los datos se representen automáticamente en el mapa mediante la capa de símbolos. La última parte del script permite establecer la vista de cámaras del mapa mediante el rectángulo delimitador de los resultados con la propiedad [setCamera](/javascript/api/azure-maps-control/atlas.map#setcamera-cameraoptions---cameraboundsoptions---animationoptions-) de Map.

    ```JavaScript
    var query =  'gasoline-station';
    var radius = 9000;
    var lat = 47.64452336193245;
    var lon = -122.13687658309935;

    searchURL.searchPOI(atlas.service.Aborter.timeout(10000), query, {
        limit: 10,
        lat: lat,
        lon: lon,
        radius: radius
    }).then((results) => {

        // Extract GeoJSON feature collection from the response and add it to the datasource
        var data = results.geojson.getFeatures();
        datasource.add(data);

        // set camera to bounds to show the results
        map.setCamera({
            bounds: data.bbox,
            zoom: 10
        });
    });
    ```

3. Guarde el archivo **MapSearch.html** y actualice el explorador. El mapa se centrará en Seattle y aparecerán marcadas con chinchetas azules redondas las ubicaciones de las gasolineras de la zona.

   ![Visualización del mapa con los resultados de la búsqueda](./media/tutorial-search-location/pins-map.png)

4. Puede ver los datos sin procesar que está representando el mapa especificando la siguiente solicitud HTTPRequest en el explorador. Reemplace \<Your Azure Maps Key\> por la clave principal.

   ```http
   https://atlas.microsoft.com/search/poi/json?api-version=1.0&query=gasoline%20station&subscription-key=<subscription-key>&lat=47.6292&lon=-122.2337&radius=100000
   ```

En este momento, la página MapSearch puede mostrar las ubicaciones de los puntos de interés que se devuelven en una consulta de búsqueda aproximada. Vamos a agregar algunas funcionalidades interactivas y más información sobre las ubicaciones.

## <a name="add-interactive-data"></a>Adición de datos interactivos

La asignación que hemos hecho hasta ahora solo examina los datos de latitud o longitud de los resultados de búsqueda. No obstante, el JSON sin formato que devuelve el servicio de búsqueda de Azure Maps contiene información adicional sobre cada gasolinera, como el nombre y la dirección. Puede incorporar esos datos al mapa con cuadros emergentes interactivos.

1. Agregue las siguientes líneas de código en el controlador de eventos `ready` del mapa después del código para consultar el servicio de búsquedas aproximadas. Este código creará una instancia de un elemento emergente y agregará un evento mouseover a la capa de símbolos.

    ```JavaScript
   //Create a popup but leave it closed so we can update it and display it later.
    popup = new atlas.Popup();

    //Add a mouse over event to the result layer and display a popup when this event fires.
    map.events.add('mouseover', resultLayer, showPopup);
    ```

    La API `*atlas.Popup` proporciona una ventana de información anclada en la posición requerida en el mapa. 

2. Agregue el código siguiente en la función `GetMap` para mostrar la información de resultados del evento mouseover en el elemento emergente.

    ```JavaScript
    function showPopup(e) {
        //Get the properties and coordinates of the first shape that the event occurred on.

        var p = e.shapes[0].getProperties();
        var position = e.shapes[0].getCoordinates();

        //Create HTML from properties of the selected result.
        var html = `
          <div style="padding:5px">
            <div><b>${p.poi.name}</b></div>
            <div>${p.address.freeformAddress}</div>
            <div>${position[1]}, ${position[0]}</div>
          </div>`;

        //Update the content and position of the popup.
        popup.setPopupOptions({
            content: html,
            position: position
        });

        //Open the popup.
        popup.open(map);
    }
    ```

3. Guarde el archivo y actualice el explorador. Ahora el mapa del explorador muestra los cuadros emergentes de información cuando se mantiene el mouse sobre cualquiera de las chinchetas de búsqueda.

    ![Control de mapa y Search Service de Azure](./media/tutorial-search-location/popup-map.png)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Crear una cuenta con Azure Maps
> * Obtención de la clave principal de una cuenta
> * Crear una nueva página web mediante la API de Control de mapa
> * Uso de Search Service para encontrar un punto de interés cercano

> [!div class="nextstepaction"]
> [Ver el código fuente completo](https://github.com/Azure-Samples/AzureMapsCodeSamples/blob/master/AzureMapsCodeSamples/Tutorials/search.html)

> [!div class="nextstepaction"]
> [Ver un ejemplo publicado](https://azuremapscodesamples.azurewebsites.net/?sample=Search%20for%20points%20of%20interest)

En el siguiente tutorial se muestra cómo visualizar una ruta entre dos ubicaciones.

> [!div class="nextstepaction"]
> [Ruta a un destino](./tutorial-route-location.md)
