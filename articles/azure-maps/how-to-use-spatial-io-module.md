---
title: Cómo usar el módulo de E/E espacial de Azure Maps | Microsoft Azure Maps
description: Obtenga información sobre cómo usar el módulo de E/S espacial proporcionado por el SDK web de Azure Maps. Este módulo proporciona características sólidas para facilitar a los desarrolladores la integración de datos espaciales con el SDK web de Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 02/28/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: devx-track-javascript
ms.openlocfilehash: f848274f67b5ebf5349cbef24a769aaae39b9955
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87287036"
---
# <a name="how-to-use-the-azure-maps-spatial-io-module"></a>Cómo usar el módulo de E/S espacial Azure Maps

El SDK web de Azure Maps proporciona el **módulo de E/S espacial**, que integra los datos espaciales con el SDK web de Azure Maps mediante JavaScript o TypeScript. Las eficaces características de este módulo permiten a los desarrolladores:

- [Leer y escribir datos de archivos espaciales comunes](spatial-io-read-write-spatial-data.md). Entre otros, estos son los formatos de archivos admitidos: Archivos KML, KMZ, GPX, GeoRSS, GML, GeoJSON y CSV que contengan columnas con información espacial. También admite el texto Well-Known Text (WKT).
- [Conectarse a los servicios de Open Geospatial Consortium (OGC) e integre con el SDK web de Azure Maps. Superponer Web Mapping Services (WMS) y Web Map tile Services (WMTS) como capas en el mapa](spatial-io-add-ogc-map-layer.md).
- [Consultar los datos en Web Feature Service (WFS)](spatial-io-connect-wfs-service.md).
- [Superponer los conjuntos de datos complejos que contienen información de estilo y hacer que se representen automáticamente usando el mínimo código](spatial-io-add-simple-data-layer.md).
- [Aprovechar las clases de lector y escritor de archivos XML y delimitados de alta velocidad](spatial-io-core-operations.md).

En esta guía, aprenderá a integrar y usar el módulo de E/S espacial en una aplicación web.

En este vídeo se proporciona información general del módulo de E/S espacial en el SDK web de Azure Maps.

<br/>

<iframe src="https://channel9.msdn.com/Shows/Internet-of-Things-Show/Easily-integrate-spatial-data-into-the-Azure-Maps/player" width="960" height="540" allowFullScreen frameBorder="0" title="Integre fácilmente los datos espaciales en el vídeo Azure Maps - Microsoft Channel 9"></iframe>


> [!WARNING]
> Use solo los datos y servicios que procedan de un origen de confianza, sobre todo si se hace referencia a él desde otro dominio. El módulo de E/S espacial lleva a cabo pasos para minimizar el riesgo; sin embargo, el método más seguro consiste en no permitir que se inicien datos peligrosos en la aplicación. 

## <a name="prerequisites"></a>Prerrequisitos

Para poder usar el módulo de E/S espacial, deberá [crear una cuenta Azure Maps](https://docs.microsoft.com/azure/azure-maps/quick-demo-map-app#create-an-azure-maps-account) y [obtener la clave de suscripción principal de la cuenta](https://docs.microsoft.com/azure/azure-maps/quick-demo-map-app#get-the-primary-key-for-your-account).

## <a name="installing-the-spatial-io-module"></a>Instalación del módulo de E/S espacial

Puede cargar el módulo de E/E espacial de Azure Maps mediante una de las dos opciones:

* Azure CDN hospedado globalmente para el módulo de E/S espacial de Azure Maps. Para esta opción, se agrega una referencia a JavaScript en el elemento `<head>` del archivo HTML.

    ```html
    <script src="https://atlas.microsoft.com/sdk/javascript/spatial/0/atlas-spatial.js"></script>
    ```

* El código fuente para [azure-maps-spatial-io](https://www.npmjs.com/package/azure-maps-spatial-io) se puede cargar localmente y, después, hospedarse con la aplicación. Este paquete también incluye las definiciones de TypeScript. Para esta opción, use el siguiente comando para instalar el paquete:

    ```sh
    npm install azure-maps-spatial-io
    ```

    A continuación, agregue una referencia al código JavaScript en el elemento `<head>` del documento HTML:

    ```html
    <script src="node_modules/azure-maps-spatial-io/dist/atlas-spatial.min.js"></script>
    ```

## <a name="using-the-spatial-io-module"></a>Uso del módulo de E/S espacial

1. Cree un archivo HTML.

2. Cargue el SDK web de Azure Maps e inicialice el control de mapas. Consulte en la guía [Uso del control de mapa de Azure Maps](https://docs.microsoft.com/azure/azure-maps/how-to-use-map-control) los detalles. Una vez que haya terminado este paso, el archivo HTML debe tener la siguiente apariencia:

    ```html
    <!DOCTYPE html>
    <html>

    <head>
        <title></title>

        <meta charset="utf-8">

        <!-- Ensures that IE and Edge uses the latest version and doesn't emulate an older version -->
        <meta http-equiv="x-ua-compatible" content="IE=Edge">

        <!-- Ensures the web page looks good on all screen sizes. -->
        <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

        <!-- Add references to the Azure Maps Map control JavaScript and CSS files. -->
        <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css" />
        <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.js"></script>

        <script type='text/javascript'>

            var map;

            function GetMap() {
                //Initialize a map instance.
                map = new atlas.Map('myMap', {
                    view: 'Auto',

                    //Add your Azure Maps subscription key to the map SDK. Get an Azure Maps key at https://azure.com/maps
                    authOptions: {
                        authType: 'subscriptionKey',
                        subscriptionKey: '<Your Azure Maps Key>'
                    }
                });

                //Wait until the map resources are ready.
                map.events.add('ready', function() {

                    // Write your code here to make sure it runs once the map resources are ready

                });
            }
        </script>
    </head>

    <body onload="GetMap()">
        <div id="myMap"></div>
    </body>

    </html>
    ```

2. Cargue el módulo de E/S espacial de Azure Maps. Para este ejercicio, use la red CDN para el módulo de E/S espacial de Azure Maps. Agregue la siguiente referencia al elemento `<head>` del archivo HTML:

    ```html
    <script src="https://atlas.microsoft.com/sdk/javascript/spatial/0/atlas-spatial.js"></script>
    ```

3. Inicialice un elemento `datasource` y agregue el origen de datos al mapa. Inicialice un elemento `layer` y agregue el origen de datos a la capa de mapa. A continuación, represente tanto el origen de datos como la capa. Antes de desplazarse hacia abajo para ver el código completo en el paso siguiente, piense en los mejores lugares donde colocar los fragmentos de código de la capa y el origen de datos. Recuerde que, antes de manipular el mapa mediante programación, deberíamos esperar hasta que el recurso de mapa esté listo.

    ```javascript
    var datasource, layer;
    ```

    y

    ```javascript
    //Create a data source and add it to the map.
    datasource = new atlas.source.DataSource();
    map.sources.add(datasource);
    
    //Add a simple data layer for rendering the data.
    layer = new atlas.layer.SimpleDataLayer(datasource);
    map.layers.add(layer);
    ```

4. En suma, el archivo HTML debería ser parecido al código siguiente. Este ejemplo muestra cómo leer un archivo XML desde una dirección URL. A continuación, cargue y muestre los datos de las características del archivo en el mapa. 

    ```html
    <!DOCTYPE html>
    <html>

    <head>
        <title>Spatial IO Module Example</title>

        <meta charset="utf-8">

        <!-- Ensures that IE and Edge uses the latest version and doesn't emulate an older version -->
        <meta http-equiv="x-ua-compatible" content="IE=Edge">

        <!-- Ensures the web page looks good on all screen sizes. -->
        <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

        <!-- Add references to the Azure Maps Map control JavaScript and CSS files. -->
        <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css" />
        <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.js"></script>

        <!-- Add reference to the Azure Maps Spatial IO module. -->
        <script src="https://atlas.microsoft.com/sdk/javascript/spatial/0/atlas-spatial.js"></script>

        <script type='text/javascript'>
            var map, datasource, layer;

            function GetMap() {
                //Initialize a map instance.
                map = new atlas.Map('myMap', {
                    view: 'Auto',

                    //Add your Azure Maps subscription key to the map SDK. Get an Azure Maps key at https://azure.com/maps
                    authOptions: {
                        authType: 'subscriptionKey',
                        subscriptionKey: '<Your Azure Maps Key>'
                    }
                });

                //Wait until the map resources are ready.
                map.events.add('ready', function() {

                    //Create a data source and add it to the map.
                    datasource = new atlas.source.DataSource();
                    map.sources.add(datasource);

                    //Add a simple data layer for rendering the data.
                    layer = new atlas.layer.SimpleDataLayer(datasource);
                    map.layers.add(layer);

                    //Read an XML file from a URL or pass in a raw XML string.
                    atlas.io.read('superCoolKmlFile.xml').then(r => {
                        if (r) {
                            //Add the feature data to the data source.
                            datasource.add(r);

                            //If bounding box information is known for data, set the map view to it.
                            if (r.bbox) {
                                map.setCamera({
                                    bounds: r.bbox,
                                    padding: 50
                                });
                            }
                        }
                    });
                });
            }
        </script>
    </head>

    <body onload="GetMap()">
        <div id="myMap"></div>
    </body>

    </html>
    ```

5. No olvide reemplazar `<Your Azure Maps Key>` por la clave principal. Abra el archivo HTML y verá resultados similares a la siguiente imagen:

    <center>

    ![Ejemplo de datos espaciales](./media/how-to-use-spatial-io-module/spatial-data-example.png)

    </center>

## <a name="next-steps"></a>Pasos siguientes

La característica que hemos mostrado aquí es solo una de las muchas disponibles en el módulo de E/S espacial. Lea las guías siguientes para obtener información sobre cómo usar otras funcionalidades del módulo de E/S espacial:

> [!div class="nextstepaction"]
> [Adición de una capa de datos simple](spatial-io-add-simple-data-layer.md)

> [!div class="nextstepaction"]
> [Lectura y escritura de datos espaciales](spatial-io-read-write-spatial-data.md)

> [!div class="nextstepaction"]
> [Adición de una capa de mapa de OGC](spatial-io-add-ogc-map-layer.md)

> [!div class="nextstepaction"]
> [Conexión a un servicio WFS](spatial-io-connect-wfs-service.md)

> [!div class="nextstepaction"]
> [Aprovechamiento de las operaciones básicas](spatial-io-core-operations.md)

> [!div class="nextstepaction"]
> [Detalles de formatos de datos admitidos](spatial-io-supported-data-format-details.md)

Consulte la documentación del módulo de E/S espacial de Azure Maps:

> [!div class="nextstepaction"]
> [Paquete de E/S espacial de Azure Maps](https://docs.microsoft.com/javascript/api/azure-maps-spatial-io/)
