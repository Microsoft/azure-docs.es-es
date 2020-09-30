---
title: 'Tutorial: Creación de una aplicación de localizador de comercios mediante Azure Maps | Microsoft Azure Maps'
description: Obtenga información sobre cómo crear aplicaciones web para localizar comercios. Use el SDK web de Azure Maps para crear una página web, consultar el servicio de búsqueda y mostrar los resultados en un mapa.
author: anastasia-ms
ms.author: v-stharr
ms.date: 08/11/2020
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.custom: mvc, devx-track-javascript
ms.openlocfilehash: 61c085cb721f9e1a8d9c44146a9d96cd5a08562c
ms.sourcegitcommit: 07166a1ff8bd23f5e1c49d4fd12badbca5ebd19c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90085321"
---
# <a name="tutorial-create-a-store-locator-by-using-azure-maps"></a>Tutorial: Creación de un localizador de almacén mediante Azure Maps

Este tutorial le guía por el proceso de creación de un localizador de almacén sencillo mediante Azure Maps. Los localizadores de almacén son comunes. Muchos de los conceptos que se usan en este tipo de aplicación son aplicables a muchos otros tipos de aplicaciones. Para la mayoría de las empresas que venden directamente a los consumidores, es imprescindible ofrecer a los clientes un localizador de almacén. En este tutorial, aprenderá a:
    
> [!div class="checklist"]
> * Crear una página web mediante la API de Control de mapa de Azure.
> * Cargar datos personalizados desde un archivo y mostrarlos en un mapa.
> * Usar el servicio de búsqueda de Azure Maps para encontrar una dirección o escribir una consulta.
> * Obtener la ubicación del usuario desde el explorador y mostrarla en el mapa.
> * Combinar varias capas para crear símbolos personalizados en el mapa.  
> * Agrupar puntos de datos.  
> * Agregar controles de zoom al mapa.

<a id="Intro"></a>

Avance al [ejemplo de localizador de almacén activo](https://azuremapscodesamples.azurewebsites.net/?sample=Simple%20Store%20Locator) o al [código fuente](https://github.com/Azure-Samples/AzureMapsCodeSamples/tree/master/AzureMapsCodeSamples/Tutorials/Simple%20Store%20Locator).

## <a name="prerequisites"></a>Prerrequisitos

1. [Crear una cuenta de Azure Maps con el plan de tarifa S1](quick-demo-map-app.md#create-an-azure-maps-account)
2. [Obtenga una clave de suscripción principal](quick-demo-map-app.md#get-the-primary-key-for-your-account), también conocida como clave principal o clave de suscripción.

Para más información sobre la autenticación en Azure Maps, consulte [Administración de la autenticación en Azure Maps](how-to-manage-authentication.md).

## <a name="design"></a>Diseño

Antes de empezar con el código, es una buena idea comenzar con un diseño. El localizador de almacén puede ser tan sencillo o complejo como se quiera. En este tutorial, crearemos un localizador de almacén sencillo. Se incluyen algunas sugerencias a lo largo del camino para ayudarle a ampliar algunas funcionalidades si elige hacerlo. Crearemos un localizador de almacén para una compañía ficticia llamada Contoso Coffee. En la siguiente ilustración se muestra un contorno reticular del diseño general del localizador de almacén que se va a crear en este tutorial:

![Esquema de página de una aplicación de localizador de comercios para las ubicaciones de los comercios Contoso Coffee](./media/tutorial-create-store-locator/SimpleStoreLocatorWireframe.png)

Para sacar el máximo provecho de este localizador de almacén, se incluye un diseño dinámico que se ajusta cuando el ancho de pantalla de un usuario es inferior a 700 píxeles. Un diseño dinámico facilita el uso del localizador de almacén en una pantalla pequeña, como la de un dispositivo móvil. Este es el contorno reticular de un diseño de pantalla pequeña:  

![Esquema de página de la aplicación de localizador de los comercios Contoso Coffee en un dispositivo móvil](./media/tutorial-create-store-locator/SimpleStoreLocatorMobileWireframe.png)</

Los contornos reticulares muestran una aplicación bastante sencilla. La aplicación tiene un cuadro de búsqueda, una lista de almacenes cercanos y un mapa con algunos marcadores, como símbolos. Además, tiene una ventana emergente que muestra información adicional cuando el usuario selecciona un marcador. Con más detalle, estas son las características que se van a crear en este localizador de almacén en este tutorial:

* Todas las ubicaciones del archivo de datos importado delimitado por tabulaciones se cargan en el mapa.
* El usuario puede realizar una panorámica del mapa y acercarlo o alejarlo, efectuar una búsqueda y seleccionar el botón del GPS My Location (Mi ubicación).
* El diseño de página se ajusta según el ancho de la pantalla del dispositivo.  
* Un encabezado muestra el logotipo de la tienda.  
* El usuario puede usar un cuadro de búsqueda y un botón de búsqueda para buscar una ubicación, como una dirección, un código postal o una ciudad. 
* Un evento `keypress` agregado al cuadro de búsqueda desencadena una búsqueda si el usuario presiona Entrar. Esta funcionalidad a menudo se pasa por alto, pero crea una mejor experiencia de usuario.
* Cuando el mapa se mueve, se calcula la distancia a cada ubicación desde el centro del mapa. La lista de resultados se actualiza para mostrar las ubicaciones más próximas en la parte superior del mapa.  
* Cuando se selecciona un resultado de la lista de resultados, el mapa se centra en la ubicación seleccionada y aparece información sobre la ubicación en una ventana emergente.  
* Al seleccionar una ubicación específica en el mapa, también se muestra una ventana emergente.
* Cuando el usuario aleja el mapa, las ubicaciones se agrupan en clústeres. Los clústeres se representan mediante un círculo con un número dentro. Los clústeres se forman y se separan a medida que el usuario cambia el nivel de zoom.
* Al seleccionar un clúster se acerca el mapa dos niveles y se centra sobre la ubicación del clúster.

<a id="create a data-set"></a>

## <a name="create-the-store-location-dataset"></a>Creación del conjunto de datos de la ubicación de almacén

Antes de desarrollar una aplicación de localizador de almacén, se debe crear un conjunto de datos de las tiendas que quiere mostrar en el mapa. En este tutorial, se usará un conjunto de datos de una cafetería ficticia llamada Contoso Coffee. El conjunto de datos de este localizador de almacén sencillo se administra en un libro de Excel. El conjunto de datos contiene 10 213 ubicaciones de cafetería de Contoso Coffee repartidas alrededor de nueve países y regiones: Estados Unidos, Canadá, Reino Unido, Francia, Alemania, Italia, Países Bajos, Dinamarca y España. Esta es una captura de pantalla del aspecto de los datos:

![Captura de pantalla de los datos del localizador de almacén en un libro de Excel](./media/tutorial-create-store-locator/StoreLocatorDataSpreadsheet.png)

Puede [descargar el libro de Excel](https://github.com/Azure-Samples/AzureMapsCodeSamples/tree/master/AzureMapsCodeSamples/Tutorials/Simple%20Store%20Locator/data).

Al examinar la captura de pantalla de los datos, podemos hacer las observaciones siguientes:

* La información de ubicación se almacena mediante las columnas **AddressLine**, **City**, **Municipality** (municipio), **AdminDivision** (región/provincia), **PostCode** (código postal) y **Country**.  
* Las columnas **Latitude** y **Longitude** contienen las coordenadas de cada ubicación de cafetería de Contoso Coffee. Si no tiene información de coordenadas, puede usar los servicios de búsqueda de Azure Maps para determinar las coordenadas de ubicación.
* Algunas columnas adicionales contienen metadatos relacionados con las cafeterías: un número de teléfono, columnas booleanas y el horario de apertura y cierre de la tienda en formato de 24 horas. Las columnas booleanas son para la Wi-Fi y la accesibilidad en silla de ruedas. Puede crear sus propias columnas para incluir los metadatos más significativos en relación con los datos de su ubicación.

> [!NOTE]
> Azure Maps representa los datos en la proyección esférica de Mercator "EPSG:3857" pero lee los datos en "EPSG:4325" que usan los datos de WGS84.

Hay muchas maneras de exponer el conjunto de datos a la aplicación. Un enfoque es cargar los datos en una base de datos y exponer un servicio web que consulte los datos. Puede enviar los resultados al explorador del usuario. Esta opción es muy conveniente para grandes conjuntos de datos o para conjuntos de datos que se actualizan con frecuencia. Sin embargo, esta opción requiere más trabajo de desarrollo y el costo es mayor.

Otro enfoque consiste en convertir este conjunto de datos en un archivo de texto sin formato que el explorador pueda analizar fácilmente. El archivo en sí se puede hospedar con el resto de la aplicación. Esta opción simplifica las cosas, pero solo es adecuada para conjuntos de datos más pequeños debido a que el usuario descarga todos los datos. Con este conjunto de datos se usará el archivo de texto sin formato porque el tamaño del archivo de datos es inferior a 1 MB.  

Para convertir el libro en un archivo de texto sin formato, guárdelo como un archivo delimitado por tabulaciones. Cada columna está delimitada por un carácter de tabulación, lo que facilita el análisis de las columnas en nuestro código. Puede usar el formato de valores separados por comas (CSV), pero esa opción requiere más lógica de análisis. Cualquier campo que tenga una coma alrededor podría incluirse entre comillas. Para exportar estos datos como un archivo delimitado por tabulaciones en Excel, seleccione **Save As** (Guardar como). En la lista desplegable **Save as type** (Guardar como tipo), seleccione **(Texto (delimitado por tabulaciones)(*.txt)** . Asigne al archivo el nombre *ContosoCoffee.txt*.

![Captura de pantalla del cuadro de diálogo Guardar como tipo](./media/tutorial-create-store-locator/SaveStoreDataAsTab.png)

Si abre el archivo de texto en el Bloc de notas, se parecerá a la siguiente ilustración:

![Captura de pantalla de un archivo de Bloc de notas que muestra un conjunto de datos delimitado por tabulaciones](./media/tutorial-create-store-locator/StoreDataTabFile.png)

## <a name="set-up-the-project"></a>Configuración del proyecto

Para crear el proyecto, puede usar [Visual Studio](https://visualstudio.microsoft.com) o el editor de código de su elección. En la carpeta del proyecto, cree tres archivos: *index.html*, *index.css* e *index.js*. Estos archivos definen el diseño, el estilo y la lógica de la aplicación. Cree una carpeta llamada *data* y agregue a ella *ContosoCoffee.txt*. Cree otra carpeta llamada *images*. Se usarán 10 imágenes en esta aplicación para iconos, botones y marcadores en el mapa. También puede [descargar estas imágenes](https://github.com/Azure-Samples/AzureMapsCodeSamples/tree/master/AzureMapsCodeSamples/Tutorials/Simple%20Store%20Locator/data). La carpeta del proyecto ahora debería parecerse a la siguiente ilustración:

![Captura de pantalla de la carpeta de proyecto del localizador de almacén sencillo](./media/tutorial-create-store-locator/StoreLocatorVSProject.png)

## <a name="create-the-user-interface"></a>Creación de la interfaz de usuario

Para crear la interfaz de usuario, agregue código a *index.html*:

1. Agregue las siguientes etiquetas `meta` a `head` de *index.html*. La etiqueta `charset` define el juego de caracteres (UTF-8). El valor de `http-equiv` indica a Internet Explorer y Microsoft Edge que usen las versiones más recientes del explorador. Y la última etiqueta `meta` especifica una ventanilla que funciona bien para diseños dinámicos.

    ```HTML
    <meta charset="utf-8">
    <meta http-equiv="x-ua-compatible" content="IE=Edge">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    ```

1. Agregue referencias a los archivos CSS y JavaScript de control web de Azure Maps:

    ```HTML
    <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css">
    <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js"></script>
    ```

1. Agregue una referencia al módulo de servicios de Azure Maps. El módulo es una biblioteca de JavaScript que contiene los servicios REST de Azure Maps y facilita su uso en JavaScript. Resulta útil para activar la funcionalidad de búsqueda.

    ```HTML
    <script src="https://atlas.microsoft.com/sdk/javascript/service/2/atlas-service.min.js"></script>
    ```

1. Agregue referencias a *index.js* e *index.css*:

    ```HTML
    <link rel="stylesheet" href="index.css" type="text/css">
    <script src="index.js"></script>
    ```

1. En el cuerpo del documento, agregue una etiqueta `header`. Dentro de la etiqueta `header`, agregue el logotipo y el nombre de la empresa.

    ```HTML
    <header>
        <img src="images/Logo.png" />
        <span>Contoso Coffee</span>
    </header>
    ```

1. Agregue una etiqueta `main` y cree un panel de búsqueda que tenga un cuadro de texto y un botón de búsqueda. Además, agregue referencias `div` al mapa, el panel de lista y el botón de GPS My Location (Mi ubicación).

    ```HTML
    <main>
        <div class="searchPanel">
            <div>
                <input id="searchTbx" type="search" placeholder="Find a store" />
                <button id="searchBtn" title="Search"></button>
            </div>
        </div>
        <div id="listPanel"></div>
        <div id="myMap"></div>
        <button id="myLocationBtn" title="My Location"></button>
    </main>
    ```

Cuando haya terminado, *index.html* se parecerá a [este archivo index.html de ejemplo](https://github.com/Azure-Samples/AzureMapsCodeSamples/blob/master/AzureMapsCodeSamples/Tutorials/Simple%20Store%20Locator/index.html).

El siguiente paso consiste en definir los estilos CSS. Los estilos CSS definen cómo se distribuyen los componentes de aplicación y la apariencia de la aplicación. Abra *index.css* y agréguele el código siguiente. El estilo `@media` define las opciones de estilo alternativas que se usarán cuando el ancho de pantalla sea inferior a 700 píxeles.  

   ```CSS
    html, body {
        padding: 0;
        margin: 0;
        font-family: Gotham, Helvetica, sans-serif;
        overflow-x: hidden;
    }

    header {
        width: calc(100vw - 10px);
        height: 30px;
        padding: 15px 0 20px 20px;
        font-size: 25px;
        font-style: italic;
        font-family: "Comic Sans MS", cursive, sans-serif;
        line-height: 30px;
        font-weight: bold;
        color: white;
        background-color: #007faa;
    }

    header span {
        vertical-align: middle;
    }

    header img {
        height: 30px;
        vertical-align: middle;
    }

    .searchPanel {
        position: relative;
        width: 350px;
    }

    .searchPanel div {
        padding: 20px;
    }

    .searchPanel input {
        width: calc(100% - 50px);
        font-size: 16px;
        border: 0;
        border-bottom: 1px solid #ccc;
    }

    #listPanel {
        position: absolute;
        top: 135px;
        left: 0px;
        width: 350px;
        height: calc(100vh - 135px);
        overflow-y: auto;
    }

    #myMap { 
        position: absolute;
        top: 65px;
        left: 350px;
        width: calc(100vw - 350px);
        height: calc(100vh - 65px);
    }

    .statusMessage {
        margin: 10px;
    }

    #myLocationBtn, #searchBtn {
        margin: 0;
        padding: 0;
        border: none;
        border-collapse: collapse;
        width: 32px;
        height: 32px; 
        text-align: center;
        cursor: pointer;
        line-height: 32px;
        background-repeat: no-repeat;
        background-size: 20px;
        background-position: center center;
        z-index: 200;
    }

    #myLocationBtn {
        position: absolute;
        top: 150px;
        right: 10px;
        box-shadow: 0px 0px 4px rgba(0,0,0,0.16);
        background-color: white;
        background-image: url("images/GpsIcon.png");
    }

    #myLocationBtn:hover {
        background-image: url("images/GpsIcon-hover.png");
    }

    #searchBtn {
        background-color: transparent;
        background-image: url("images/SearchIcon.png");
    }

    #searchBtn:hover {
        background-image: url("images/SearchIcon-hover.png");
    }

    .listItem {
        height: 50px;
        padding: 20px;
        font-size: 14px;
    }

    .listItem:hover {
        cursor: pointer;
        background-color: #f1f1f1;
    }

    .listItem-title {
        color: #007faa;
        font-weight: bold;
    }

    .storePopup {
        min-width: 150px;
    }

    .storePopup .popupTitle {
        border-top-left-radius: 4px;
        border-top-right-radius: 4px;
        padding: 8px;
        height: 30px;
        background-color: #007faa;
        color: white;
        font-weight: bold;
    }

    .storePopup .popupSubTitle {
        font-size: 10px;
        line-height: 12px;
    }

    .storePopup .popupContent {
        font-size: 11px;
        line-height: 18px;
        padding: 8px;
    }

    .storePopup img {
        vertical-align:middle;
        height: 12px;
        margin-right: 5px;
    }

    /* Adjust the layout of the page when the screen width is less than 700 pixels. */
    @media screen and (max-width: 700px) {
        .searchPanel {
            width: 100vw;
        }

        #listPanel {
            top: 385px;
            width: 100%;
            height: calc(100vh - 385px);
        }

        #myMap {
            width: 100vw;
            height: 250px;
            top: 135px;
            left: 0px;
        }

        #myLocationBtn {
            top: 220px;
        }
    }

    .mapCenterIcon {
        display: block;
        width: 10px;
        height: 10px;
        border-radius: 50%;
        background: orange;
        border: 2px solid white;
        cursor: pointer;
        box-shadow: 0 0 0 rgba(0, 204, 255, 0.4);
        animation: pulse 3s infinite;
    }

    @keyframes pulse {
        0% {
            box-shadow: 0 0 0 0 rgba(0, 204, 255, 0.4);
        }

        70% {
            box-shadow: 0 0 0 50px rgba(0, 204, 255, 0);
        }

        100% {
            box-shadow: 0 0 0 0 rgba(0, 204, 255, 0);
        }
    }
   ```

Ejecute la aplicación ahora y verá el encabezado, el cuadro de búsqueda y el botón Buscar. Pero la asignación no está visible porque todavía no se ha cargado. Si intenta realizar una búsqueda, no ocurre nada. Es necesario configurar la lógica de JavaScript, que se describe en la sección siguiente. Esta lógica tiene acceso a toda la funcionalidad del localizador de tiendas.

## <a name="wire-the-application-with-javascript"></a>Conexión de la aplicación con JavaScript

Llegados a este punto, todo está configurado en la interfaz de usuario. Ahora, es necesario agregar el código JavaScript para cargar y analizar los datos y, luego, representar los datos en el mapa. Para empezar, abra *index.js* y agréguele código, como se describe en los pasos siguientes.

1. Agregue opciones globales para facilitar la actualización de la configuración. Defina las variables para el mapa, la ventana emergente, el origen de datos, la capa de iconos y el marcador HTML. Establezca el marcador HTML para indicar el centro de un área de búsqueda. Defina también una instancia de cliente del servicio de búsqueda de Azure Maps.

    ```JavaScript
    //The maximum zoom level to cluster data point data on the map.
    var maxClusterZoomLevel = 11;

    //The URL to the store location data.
    var storeLocationDataUrl = 'data/ContosoCoffee.txt';

    //The URL to the icon image.
    var iconImageUrl = 'images/CoffeeIcon.png';
    var map, popup, datasource, iconLayer, centerMarker, searchURL;
    ```

1. Agregue código a *index.js*. El código siguiente inicializa el mapa. Hemos agregado un [cliente de escucha de eventos](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.map#events) para esperar hasta que la página termine de cargarse. A continuación, se realiza una conexión de los eventos para supervisar la carga del mapa y se proporciona la funcionalidad para el botón Buscar y el botón Mi ubicación.

   Cuando el usuario selecciona el botón de búsqueda o escribe una ubicación en el cuadro de búsqueda y presiona Entrar, se inicia una búsqueda aproximada con la consulta del usuario. Pase una matriz de valores de país o región ISO 2 a la opción `countrySet` para limitar los resultados de la búsqueda a esos países o regiones. Limitar los países y regiones de búsqueda ayuda a aumentar la precisión de los resultados que se devuelven. 
  
   Una vez finalizada la búsqueda, tome el primer resultado y establezca la cámara del mapa sobre esa zona. Cuando el usuario seleccione el botón My Location (Mi ubicación), recuperará su ubicación mediante HTML5 Geolocation API. Esta API está integrada en el explorador. A continuación, centre el mapa en su ubicación.  

   > [!Tip]
   > Cuando se usen ventanas emergentes, es mejor crear una única instancia de `Popup` y reutilizarla mediante la actualización de su contenido y posición. Para cada instancia de `Popup` que agrega al código, se agregan varios elementos DOM a la página. Cuantos más elementos DOM haya en una página, de más cosas tiene que realizar el explorador un seguimiento. Si hay demasiados elementos, el explorador podría ralentizarse.

    ```JavaScript
    function initialize() {
        //Initialize a map instance.
        map = new atlas.Map('myMap', {
            center: [-90, 40],
            zoom: 2,

            //Add your Azure Maps primary subscription key to the map SDK.
            authOptions: {
                authType: 'subscriptionKey',
                subscriptionKey: '<Your Azure Maps Key>'
            }
        });

        //Create a pop-up window, but leave it closed so we can update it and display it later.
        popup = new atlas.Popup();

        //Use SubscriptionKeyCredential with a subscription key
        const subscriptionKeyCredential = new atlas.service.SubscriptionKeyCredential(atlas.getSubscriptionKey());

        //Use subscriptionKeyCredential to create a pipeline
        const pipeline = atlas.service.MapsURL.newPipeline(subscriptionKeyCredential, {
            retryOptions: { maxTries: 4 } // Retry options
        });

        //Create an instance of the SearchURL client.
        searchURL = new atlas.service.SearchURL(pipeline);

        //If the user selects the search button, geocode the value the user passed in.
        document.getElementById('searchBtn').onclick = performSearch;

        //If the user presses Enter in the search box, perform a search.
        document.getElementById('searchTbx').onkeyup = function(e) {
            if (e.keyCode === 13) {
                performSearch();
            }
        };

        //If the user selects the My Location button, use the Geolocation API to get the user's location. Center and zoom the map on that location.
        document.getElementById('myLocationBtn').onclick = setMapToUserLocation;

        //Wait until the map resources are ready.
        map.events.add('ready', function() {

            //Add your post-map load functionality.

        });
    }

    //Create an array of country/region ISO 2 values to limit searches to. 
    var countrySet = ['US', 'CA', 'GB', 'FR','DE','IT','ES','NL','DK'];

    function performSearch() {
        var query = document.getElementById('searchTbx').value;

        //Perform a fuzzy search on the users query.
        searchURL.searchFuzzy(atlas.service.Aborter.timeout(3000), query, {
            //Pass in the array of country/region ISO2 for which we want to limit the search to.
            countrySet: countrySet
        }).then(results => {
            //Parse the response into GeoJSON so that the map can understand.
            var data = results.geojson.getFeatures();

            if (data.features.length > 0) {
                //Set the camera to the bounds of the results.
                map.setCamera({
                    bounds: data.features[0].bbox,
                    padding: 40
                });
            } else {
                document.getElementById('listPanel').innerHTML = '<div class="statusMessage">Unable to find the location you searched for.</div>';
            }
        });
    }

    function setMapToUserLocation() {
        //Request the user's location.
        navigator.geolocation.getCurrentPosition(function(position) {
            //Convert the Geolocation API position to a longitude and latitude position value that the map can interpret and center the map over it.
            map.setCamera({
                center: [position.coords.longitude, position.coords.latitude],
                zoom: maxClusterZoomLevel + 1
            });
        }, function(error) {
            //If an error occurs when the API tries to access the user's position information, display an error message.
            switch (error.code) {
                case error.PERMISSION_DENIED:
                    alert('User denied the request for geolocation.');
                    break;
                case error.POSITION_UNAVAILABLE:
                    alert('Position information is unavailable.');
                    break;
                case error.TIMEOUT:
                    alert('The request to get user position timed out.');
                    break;
                case error.UNKNOWN_ERROR:
                    alert('An unknown error occurred.');
                    break;
            }
        });
    }

    //Initialize the application when the page is loaded.
    window.onload = initialize;
    ```

1. En el agente de escucha de eventos `ready` del mapa, agregue un control de zoom y un marcador de HTML para mostrar el centro de una zona de búsqueda.

    ```JavaScript
    //Add a zoom control to the map.
    map.controls.add(new atlas.control.ZoomControl(), {
        position: 'top-right'
    });

    //Add an HTML marker to the map to indicate the center to use for searching.
    centerMarker = new atlas.HtmlMarker({
        htmlContent: '<div class="mapCenterIcon"></div>',
        position: map.getCamera().center
    });

    map.markers.add(centerMarker);
    ```

1. En el agente de escucha de eventos `ready` del mapa, agregue un origen de datos. A continuación, realice una llamada para cargar y analizar el conjunto de datos. Habilite la agrupación en clústeres en el origen de datos. La agrupación en clústeres en el origen de datos agrupa los puntos superpuestos en un clúster. El clúster se separa en puntos individuales cuando el usuario acerca el mapa. Este comportamiento proporciona una mejor experiencia de usuario y mejora el rendimiento.

    ```JavaScript
    //Create a data source, add it to the map, and then enable clustering.
    datasource = new atlas.source.DataSource(null, {
        cluster: true,
        clusterMaxZoom: maxClusterZoomLevel - 1
    });

    map.sources.add(datasource);

    //Load all the store data now that the data source is defined.  
    loadStoreData();
    ```

1. Después de cargar el conjunto de datos en el agente de escucha de eventos `ready` del mapa, defina un conjunto de capas para representar los datos. Se usa una capa de burbuja para representar los puntos de datos en clúster. Se usa una capa de símbolo para representar el número de puntos en cada clúster por encima de la capa de burbuja. Una segunda capa de símbolo representa un icono personalizado para ubicaciones individuales en el mapa.

   Agregue los eventos `mouseover` y `mouseout` a las capas de burbuja e icono para que el cursor del mouse cambie cuando el usuario lo mantenga sobre un clúster o un icono en el mapa. Agregue un evento `click` a la capa de burbuja del clúster. Este evento `click` acerca el mapa dos niveles y lo centra sobre un clúster cuando el usuario selecciona cualquiera de ellos. Agregue un evento `click` a la capa de icono. Este evento `click` muestra una ventana emergente con los detalles de una cafetería cuando un usuario selecciona un icono de ubicación. Agregue un evento al mapa para supervisar el momento en que el mapa termina de moverse. Cuando se active este evento, actualice los elementos del panel de lista.  

    ```JavaScript
    //Create a bubble layer to render clustered data points.
    var clusterBubbleLayer = new atlas.layer.BubbleLayer(datasource, null, {
        radius: 12,
        color: '#007faa',
        strokeColor: 'white',
        strokeWidth: 2,
        filter: ['has', 'point_count'] //Only render data points that have a point_count property; clusters have this property.
    });

    //Create a symbol layer to render the count of locations in a cluster.
    var clusterLabelLayer = new atlas.layer.SymbolLayer(datasource, null, {
        iconOptions: {
            image: 'none' //Hide the icon image.
        },

        textOptions: {
            textField: ['get', 'point_count_abbreviated'],
            size: 12,
            font: ['StandardFont-Bold'],
            offset: [0, 0.4],
            color: 'white'
        }
    });

    map.layers.add([clusterBubbleLayer, clusterLabelLayer]);

    //Load a custom image icon into the map resources.
    map.imageSprite.add('myCustomIcon', iconImageUrl).then(function() {

    //Create a layer to render a coffee cup symbol above each bubble for an individual location.
    iconLayer = new atlas.layer.SymbolLayer(datasource, null, {
        iconOptions: {
            //Pass in the ID of the custom icon that was loaded into the map resources.
            image: 'myCustomIcon',

            //Optionally, scale the size of the icon.
            font: ['SegoeUi-Bold'],

            //Anchor the center of the icon image to the coordinate.
            anchor: 'center',

            //Allow the icons to overlap.
            allowOverlap: true
        },

        filter: ['!', ['has', 'point_count']] //Filter out clustered points from this layer.
    });

    map.layers.add(iconLayer);

    //When the mouse is over the cluster and icon layers, change the cursor to a pointer.
    map.events.add('mouseover', [clusterBubbleLayer, iconLayer], function() {
        map.getCanvasContainer().style.cursor = 'pointer';
    });

    //When the mouse leaves the item on the cluster and icon layers, change the cursor back to the default (grab).
    map.events.add('mouseout', [clusterBubbleLayer, iconLayer], function() {
        map.getCanvasContainer().style.cursor = 'grab';
    });

    //Add a click event to the cluster layer. When the user selects a cluster, zoom into it by two levels.  
    map.events.add('click', clusterBubbleLayer, function(e) {
        map.setCamera({
            center: e.position,
            zoom: map.getCamera().zoom + 2
        });
    });

    //Add a click event to the icon layer and show the shape that was selected.
    map.events.add('click', iconLayer, function(e) {
        showPopup(e.shapes[0]);
    });

    //Add an event to monitor when the map is finished rendering the map after it has moved.
    map.events.add('render', function() {
        //Update the data in the list.
        updateListItems();
    });
    ```

1. Cuando se cargue el conjunto de datos de cafeterías, primero debe descargarse. Luego, se debe dividir el archivo de texto en líneas. La primera línea contiene la información de encabezado. Para que el código sea más fácil de seguir, se analiza el encabezado en un objeto, que luego se puede usar para consultar el índice de celda de cada propiedad. Después de la primera línea, recorra en bucle el resto de líneas y cree una característica de punto. Agregue la característica de punto al origen de datos. Por último, actualice el panel de lista.

    ```JavaScript
    function loadStoreData() {

    //Download the store location data.
    fetch(storeLocationDataUrl)
        .then(response => response.text())
        .then(function(text) {

            //Parse the tab-delimited file data into GeoJSON features.
            var features = [];

            //Split the lines of the file.
            var lines = text.split('\n');

            //Grab the header row.
            var row = lines[0].split('\t');

            //Parse the header row and index each column to make the code for parsing each row easier to follow.
            var header = {};
            var numColumns = row.length;
            for (var i = 0; i < row.length; i++) {
                header[row[i]] = i;
            }

            //Skip the header row and then parse each row into a GeoJSON feature.
            for (var i = 1; i < lines.length; i++) {
                row = lines[i].split('\t');

                //Ensure that the row has the correct number of columns.
                if (row.length >= numColumns) {

                    features.push(new atlas.data.Feature(new atlas.data.Point([parseFloat(row[header['Longitude']]), parseFloat(row[header['Latitude']])]), {
                        AddressLine: row[header['AddressLine']],
                        City: row[header['City']],
                        Municipality: row[header['Municipality']],
                        AdminDivision: row[header['AdminDivision']],
                        Country: row[header['Country']],
                        PostCode: row[header['PostCode']],
                        Phone: row[header['Phone']],
                        StoreType: row[header['StoreType']],
                        IsWiFiHotSpot: (row[header['IsWiFiHotSpot']].toLowerCase() === 'true') ? true : false,
                        IsWheelchairAccessible: (row[header['IsWheelchairAccessible']].toLowerCase() === 'true') ? true : false,
                        Opens: parseInt(row[header['Opens']]),
                        Closes: parseInt(row[header['Closes']])
                    }));
                }
            }

            //Add the features to the data source.
            datasource.add(new atlas.data.FeatureCollection(features));

            //Initially, update the list items.
            updateListItems();
        });
    }
    ```

1. Cuando se actualiza el panel de lista, se calcula la distancia. Esta distancia es desde el centro del mapa a todas las características de puntos en la vista del mapa actual. Luego, las características se ordenan por distancia. Se genera código HTML para mostrar cada ubicación en el panel de lista.

    ```JavaScript
    var listItemTemplate = '<div class="listItem" onclick="itemSelected(\'{id}\')"><div class="listItem-title">{title}</div>{city}<br />Open until {closes}<br />{distance} miles away</div>';

    function updateListItems() {
        //Hide the center marker.
        centerMarker.setOptions({
            visible: false
        });

        //Get the current camera and view information for the map.
        var camera = map.getCamera();
        var listPanel = document.getElementById('listPanel');

        //Check to see whether the user is zoomed out a substantial distance. If they are, tell the user to zoom in and to perform a search or select the My Location button.
        if (camera.zoom < maxClusterZoomLevel) {
            //Close the pop-up window; clusters might be displayed on the map.  
            popup.close(); 
            listPanel.innerHTML = '<div class="statusMessage">Search for a location, zoom the map, or select the My Location button to see individual locations.</div>';
        } else {
            //Update the location of the centerMarker property.
            centerMarker.setOptions({
                position: camera.center,
                visible: true
            });

            //List the ten closest locations in the side panel.
            var html = [], properties;

            /*
            Generating HTML for each item that looks like this:
            <div class="listItem" onclick="itemSelected('id')">
                <div class="listItem-title">1 Microsoft Way</div>
                Redmond, WA 98052<br />
                Open until 9:00 PM<br />
                0.7 miles away
            </div>
            */

            //Get all the shapes that have been rendered in the bubble layer. 
            var data = map.layers.getRenderedShapes(map.getCamera().bounds, [iconLayer]);

            //Create an index of the distances of each shape.
            var distances = {};

            data.forEach(function (shape) {
                if (shape instanceof atlas.Shape) {

                    //Calculate the distance from the center of the map to each shape and store in the index. Round to 2 decimals.
                    distances[shape.getId()] = Math.round(atlas.math.getDistanceTo(camera.center, shape.getCoordinates(), 'miles') * 100) / 100;
                }
            });

            //Sort the data by distance.
            data.sort(function (x, y) {
                return distances[x.getId()] - distances[y.getId()];
            });

            data.forEach(function(shape) {
                properties = shape.getProperties();
                html.push('<div class="listItem" onclick="itemSelected(\'', shape.getId(), '\')"><div class="listItem-title">',
                properties['AddressLine'],
                '</div>',
                //Get a formatted addressLine2 value that consists of City, Municipality, AdminDivision, and PostCode.
                getAddressLine2(properties),
                '<br />',

                //Convert the closing time to a format that is easier to read.
                getOpenTillTime(properties),
                '<br />',

                //Get the distance of the shape.
                distances[shape.getId()],
                ' miles away</div>');
            });

            listPanel.innerHTML = html.join('');

            //Scroll to the top of the list panel in case the user has scrolled down.
            listPanel.scrollTop = 0;
        }
    }

    //This converts a time that's in a 24-hour format to an AM/PM time or noon/midnight string.
    function getOpenTillTime(properties) {
        var time = properties['Closes'];
        var t = time / 100;
        var sTime;

        if (time === 1200) {
            sTime = 'noon';
        } else if (time === 0 || time === 2400) {
            sTime = 'midnight';
        } else {
            sTime = Math.round(t) + ':';

            //Get the minutes.
            t = (t - Math.round(t)) * 100;

            if (t === 0) {
                sTime += '00';
            } else if (t < 10) {
                sTime += '0' + t;
            } else {
                sTime += Math.round(t);
            }

            if (time < 1200) {
                sTime += ' AM';
            } else {
                sTime += ' PM';
            }
        }

        return 'Open until ' + sTime;
    }

    //Create an addressLine2 string that contains City, Municipality, AdminDivision, and PostCode.
    function getAddressLine2(properties) {
        var html = [properties['City']];

        if (properties['Municipality']) {
            html.push(', ', properties['Municipality']);
        }

        if (properties['AdminDivision']) {
            html.push(', ', properties['AdminDivision']);
        }

        if (properties['PostCode']) {
            html.push(' ', properties['PostCode']);
        }

        return html.join('');
    }
    ```

1. Cuando el usuario selecciona un elemento en el panel de lista, la forma con la que está relacionado el elemento se recupera del origen de datos. Se genera una ventana emergente que se basa en la información de propiedad almacenada en la forma. El mapa se centra sobre la forma. Si el ancho del mapa es inferior a 700 píxeles, la vista del mapa se desplaza para que la ventana emergente sea visible.

    ```JavaScript
    //When a user selects a result in the side panel, look up the shape by its ID value and display the pop-up window.
    function itemSelected(id) {
        //Get the shape from the data source by using its ID.  
        var shape = datasource.getShapeById(id);
        showPopup(shape);

        //Center the map over the shape on the map.
        var center = shape.getCoordinates();
        var offset;

        //If the map is less than 700 pixels wide, then the layout is set for small screens.
        if (map.getCanvas().width < 700) {
            //When the map is small, offset the center of the map relative to the shape so that there is room for the popup to appear.
            offset = [0, -80];
        }

        map.setCamera({
            center: center,
            centerOffset: offset
        });
    }

    function showPopup(shape) {
        var properties = shape.getProperties();

        /* Generating HTML for the pop-up window that looks like this:

            <div class="storePopup">
                <div class="popupTitle">
                    3159 Tongass Avenue
                    <div class="popupSubTitle">Ketchikan, AK 99901</div>
                </div>
                <div class="popupContent">
                    Open until 22:00 PM<br/>
                    <img title="Phone Icon" src="images/PhoneIcon.png">
                    <a href="tel:1-800-XXX-XXXX">1-800-XXX-XXXX</a>
                    <br>Amenities:
                    <img title="Wi-Fi Hotspot" src="images/WiFiIcon.png">
                    <img title="Wheelchair Accessible" src="images/WheelChair-small.png">
                </div>
            </div>
        */

         //Calculate the distance from the center of the map to the shape in miles, round to 2 decimals.
        var distance = Math.round(atlas.math.getDistanceTo(map.getCamera().center, shape.getCoordinates(), 'miles') * 100)/100;

        var html = ['<div class="storePopup">'];
        html.push('<div class="popupTitle">',
            properties['AddressLine'],
            '<div class="popupSubTitle">',
            getAddressLine2(properties),
            '</div></div><div class="popupContent">',

            //Convert the closing time to a format that's easier to read.
            getOpenTillTime(properties),

            //Add the distance information.  
            '<br/>', distance,
            ' miles away',
            '<br /><img src="images/PhoneIcon.png" title="Phone Icon"/><a href="tel:',
            properties['Phone'],
            '">',  
            properties['Phone'],
            '</a>'
        );

        if (properties['IsWiFiHotSpot'] || properties['IsWheelchairAccessible']) {
            html.push('<br/>Amenities: ');

            if (properties['IsWiFiHotSpot']) {
                html.push('<img src="images/WiFiIcon.png" title="Wi-Fi Hotspot"/>');
            }

            if (properties['IsWheelchairAccessible']) {
                html.push('<img src="images/WheelChair-small.png" title="Wheelchair Accessible"/>');
            }
        }

        html.push('</div></div>');

        //Update the content and position of the pop-up window for the specified shape information.
        popup.setOptions({

            //Create a table from the properties in the feature.
            content:  html.join(''),
            position: shape.getCoordinates()
        });

        //Open the pop-up window.
        popup.open(map);
    }
    ```

Ahora, tiene un localizador de almacén totalmente funcional. En un explorador web, abra el archivo *index.html* correspondiente al localizador de almacén. Cuando los clústeres aparezcan en el mapa, puede buscar una ubicación, bien mediante el cuadro de búsqueda, o también puede seleccionar el botón My Location (Mi ubicación), un clúster o acercar el mapa para ver las ubicaciones individuales.

La primera vez que un usuario selecciona el botón My Location (Mi ubicación), el explorador muestra una advertencia de seguridad que solicita permiso para acceder a la ubicación del usuario. Si el usuario acepta compartir su ubicación, el mapa la acerca y se muestran las cafeterías cercanas.

![Captura de pantalla de la solicitud del explorador para acceder a la ubicación del usuario](./media/tutorial-create-store-locator/GeolocationApiWarning.png)

Al acercar lo suficiente una zona que tiene ubicaciones de cafetería, los clústeres se separan en ubicaciones individuales. Seleccione uno de los iconos del mapa o un elemento del panel lateral para ver una ventana emergente que muestre información de esa ubicación.

![Captura de pantalla del localizador de almacén finalizado](./media/tutorial-create-store-locator/FinishedSimpleStoreLocator.png)

Si cambia de tamaño la ventana del explorador a un ancho inferior a 700 píxeles o abre la aplicación en un dispositivo móvil, el diseño cambia para adaptarse mejor a pantallas más pequeñas.

![Captura de pantalla de la versión de pantalla pequeña del localizador de almacén](./media/tutorial-create-store-locator/FinishedSimpleStoreLocatorSmallScreen.png)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, aprenderá a crear un localizador de almacén básico mediante Azure Maps. El localizador de almacén que crea en este tutorial podría tener toda la funcionalidad necesaria. Puede agregar características a su localizador de almacén o usar características más avanzadas para conseguir una experiencia de usuario más personalizada: 

> [!div class="checklist"]
> * Habilitar [sugerencias mientras escribe](https://azuremapscodesamples.azurewebsites.net/?sample=Search%20Autosuggest%20and%20JQuery%20UI) en el cuadro de búsqueda  
> * Agregar [compatibilidad con varios idiomas](https://azuremapscodesamples.azurewebsites.net/?sample=Map%20Localization) 
> * Permitir que el usuario [filtrar las ubicaciones a lo largo de una ruta](https://azuremapscodesamples.azurewebsites.net/?sample=Filter%20Data%20Along%20Route) 
> * Agregar la capacidad de [establecer filtros](https://azuremapscodesamples.azurewebsites.net/?sample=Filter%20Symbols%20by%20Property) 
> * Agregar compatibilidad para especificar un valor inicial de búsqueda mediante una cadena de consulta Al incluir esta opción en el localizador de almacén, los usuarios pueden marcar y compartir búsquedas. También proporciona un método sencillo de pasar las búsquedas a esta página desde otra página.  
> * Implementar el localizador de almacén como una [aplicación web de Azure App Service](https://docs.microsoft.com/azure/app-service/quickstart-html) 
> * Almacenar los datos en una base de datos y buscar ubicaciones cercanas Para más información, consulte [Información general de los tipos de datos espaciales de SQL Server](https://docs.microsoft.com/sql/relational-databases/spatial/spatial-data-types-overview) y [Consultar datos espaciales para el vecino más próximo](https://docs.microsoft.com/sql/relational-databases/spatial/query-spatial-data-for-nearest-neighbor).

> [!div class="nextstepaction"]
> [Ver el código fuente completo](https://github.com/Azure-Samples/AzureMapsCodeSamples/tree/master/AzureMapsCodeSamples/Tutorials/Simple%20Store%20Locator)

> [!div class="nextstepaction"]
> [Ver un ejemplo publicado](https://azuremapscodesamples.azurewebsites.net/index.html?sample=Simple%20Store%20Locator)

Para más información sobre la cobertura y las funcionalidades de Azure Maps:

> [!div class="nextstepaction"]
> [Niveles de zoom y cuadrícula de mosaico](zoom-levels-and-tile-grid.md)

Para ver más ejemplos de código y una experiencia interactiva de codificación:

> [!div class="nextstepaction"]
> [Uso del control de mapa](how-to-use-map-control.md)

> [!div class="nextstepaction"]
> [Uso de expresiones de estilo controladas por datos](data-driven-style-expressions-web-sdk.md)
