---
title: Introducción al control de mapa web de Microsoft Azure Maps
description: Descubra cómo agregar mapas a aplicaciones web y móviles mediante la biblioteca de JavaScript del lado cliente de Control de mapa de Azure Maps. Vea cómo localizar mapas.
author: anastasia-ms
ms.author: v-stharr
ms.date: 07/20/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.custom: devx-track-javascript
ms.openlocfilehash: e4309b3fdda1296aade19de2874e7f3423b9fc0f
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88035967"
---
# <a name="use-the-azure-maps-map-control"></a>Uso del control de mapa de Azure Maps

La biblioteca de JavaScript del lado cliente del Control de mapa le permite representar mapas y la funcionalidad insertada de Azure Maps en su aplicación web o móvil.

## <a name="prerequisites"></a>Requisitos previos

Para usar el control de mapa en una página web, debe cumplir uno de los siguientes requisitos previos:

* [Crear una cuenta de Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account) y [obtener una clave de suscripción principal](quick-demo-map-app.md#get-the-primary-key-for-your-account), también conocida como clave principal o clave de suscripción.

* Obtener las credenciales de Azure Active Directory (AAD) con las [opciones de autenticación](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.authenticationoptions).

## <a name="create-a-new-map-in-a-web-page"></a>Creación de un nuevo mapa en una página web

Puede insertar un mapa en una página web mediante la biblioteca de JavaScript del lado cliente de Control de mapa.

1. Cree un archivo HTML.

2. Cárguelo en el SDK web de Azure Maps. Puede elegir una de las dos opciones:

    * Use la versión de CDN hospedado globalmente del SDK web de Azure Maps; para ello, agregue referencias a la hoja de estilo y a JavaScript en el elemento `<head>` del archivo HTML:

      ```html
      <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css">
      <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js"></script>
      ```

    * Cargue localmente el código fuente del SDK web de Azure Maps mediante el paquete de NPM [azure-maps-control](https://www.npmjs.com/package/azure-maps-control) y hospédelo con la aplicación. Este paquete también incluye las definiciones de TypeScript.

      > **npm install azure-maps-control**

    Después, agregue las referencias a la hoja de estilos de Azure Maps al elemento `<head>` del archivo:

    ```html
    <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css" />
     ```

    > [!NOTE]
    > Las definiciones de Typescript se pueden importar en la aplicación si agrega el código siguiente:
    >
    > ```javascript
    > import * as atlas from 'azure-maps-control';
    > ```

3. Para representar el mapa para que rellene el cuerpo completo de la página, agregue el siguiente elemento `<style>` al elemento `<head>`.

   ```HTML
    <style>
        html, body {
            margin: 0;
        }

        #myMap {
            height: 100vh;
            width: 100vw;
        }
    </style>
   ```

4. En el cuerpo de la página, agregue un elemento `<div>` y asígnele un `id` de **myMap**.

   ```HTML
    <body onload="InitMap()">
        <div id="myMap"></div>
    </body>
   ```

5. Ahora, inicializaremos el control de mapa. Para autenticar el control, deberá poseer una clave de suscripción de Azure Maps o usar credenciales de Azure Active Directory (AAD) con [opciones de autenticación](https://docs.microsoft.com/javascript/api/azure-maps-control/atlas.authenticationoptions).

    Si utiliza una clave de suscripción para la autenticación, copie y pegue el siguiente elemento de script en el elemento `<head>` y debajo del primer elemento `<script>`. Reemplace `<Your Azure Maps Key>` por la clave de suscripción principal de Azure Maps.

     ```HTML
    <script type="text/javascript">
        function InitMap()
        {
            var map = new atlas.Map('myMap', {
                center: [-122.33, 47.6],
                zoom: 12,
                language: 'en-US',
                authOptions: {
                    authType: 'subscriptionKey',
                    subscriptionKey: '<Your Azure Maps Key>'
                }
            }
        });
    </script>
    ```

    Si usa Azure Active Directory (AAD) para la autenticación, copie y pegue el siguiente elemento de script dentro del elemento `<head>` y debajo del primer elemento `<script>`.

      ```HTML
    <script type="text/javascript">
        function InitMap()
        {
            var map = new atlas.Map('myMap', {
                center: [-122.33, 47.6],
                zoom: 12,
                language: 'en-US',
                authOptions: {
                    authType: 'aad',
                    clientId: '<Your AAD Client Id>',
                    aadAppId: '<Your AAD App Id>',
                    aadTenant: '<Your AAD Tenant Id>'
                }
            }
        });
    </script>
   ```

    Para más información sobre la autenticación con Azure Maps, consulte el documento [Autenticación con Azure Maps](azure-maps-authentication.md). Además, [aquí](https://github.com/Azure-Samples/Azure-Maps-AzureAD-Samples) puede encontrar una lista de ejemplos que muestran cómo integrar Azure Active Directory (AAD) con Azure Maps.

    >[!TIP]
    >En este ejemplo, se ha pasado el valor de `id` del mapa `<div>`. Otra forma de hacerlo es pasar el objeto `HTMLElement`, con `document.getElementById('myMap')` como primer parámetro.

6. Opcionalmente, puede que le resulte útil agregar los siguientes elementos de `meta` al elemento `head` de la página:

   ```HTML
    <!-- Ensures that IE and Edge uses the latest version and doesn't emulate an older version -->
    <meta http-equiv="x-ua-compatible" content="IE=Edge">

    <!-- Ensures the web page looks good on all screen sizes. -->
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
   ```

7. Si combina todo, el archivo HTML debería ser parecido al código marcado:

   ```HTML
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
        <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css">
        <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js"></script>


        <script type="text/javascript">
            //Create an instance of the map control and set some options.
            function InitMap()
            {
                var map = new atlas.Map('myMap', {
                    center: [-122.33, 47.6],
                    zoom: 12,
                    language: 'en-US',
                    authOptions: {
                        authType: 'subscriptionKey',
                        subscriptionKey: '<Your Azure Maps Key>'
                    }
                });
            }
        </script>

        <style>
            html, body {
                margin: 0;
            }

            #myMap {
                height: 100vh;
                width: 100vw;
            }
        </style>
    </head>
    <body onload="InitMap()">
        <div id="myMap"></div>
    </body>
    </html>
    ```

8. Abra el archivo en el explorador web y vea el mapa representado. Debería parecerse a la imagen siguiente:

   ![Imagen de mapa que muestra el resultado representado](./media/how-to-use-map-control/map-of-seattle.png)

## <a name="localizing-the-map"></a>Localización del mapa

Azure Maps proporciona dos formas diferentes de establecer el idioma y la vista regional del mapa representado. La primera opción es agregar esta información al espacio de nombres `atlas` global, lo que provocará que estos valores son los predeterminados en todas las instancias de control de mapa de la aplicación. El código siguiente establece el idioma en francés ("fr-FR") y la vista regional en "auto":

```javascript
atlas.setLanguage('fr-FR');
atlas.setView('Auto');
```

La segunda opción consiste en pasar esta información a las opciones del mapa al cargar el mapa de la siguiente manera:

```javascript
map = new atlas.Map('myMap', {
    language: 'fr-FR',
    view: 'Auto',

    authOptions: {
        authType: 'aad',
        clientId: '<Your AAD Client Id>',
        aadAppId: '<Your AAD App Id>',
        aadTenant: '<Your AAD Tenant Id>'
    }
});
```

> [!NOTE]
> Es posible cargar varias instancias del mapa en la misma página con diferentes configuraciones de idioma y región. Además, esta configuración se puede actualizar después de que se haya cargado el mapa mediante la función `setStyle` del mapa.

Este es un ejemplo de Azure Maps con el idioma establecido en "fr-FR" y la vista regional establecida en "auto".

![Imagen del mapa que muestra las etiquetas en francés](./media/how-to-use-map-control/websdk-localization.png)

[Aquí](supported-languages.md) encontrará una lista completa de los idiomas y vistas regionales admitidos.

## <a name="azure-government-cloud-support"></a>Compatibilidad con la nube de Azure Government

El SDK web de Azure Maps es compatible con la nube de Azure Government. Todas las direcciones URL de JavaScript y CSS usadas para acceder al SDK web de Azure Maps siguen siendo las mismas. Se deben realizar las siguientes tareas para conectarse a la versión en la nube de Azure Government de la plataforma de Azure Maps.

Cuando utilice el control de mapas interactivos, agregue la siguiente línea de código antes de crear una instancia de la clase `Map`.

```javascript
atlas.setDomain('atlas.azure.us');
```

No olvide usar los detalles de autenticación de Azure Maps de la plataforma en la nube de Azure Government cuando autentique el mapa y los servicios.

Si utiliza el módulo de servicios, el dominio de los servicios debe definirse al crear una instancia de un punto de conexión de la dirección URL de la API. Por ejemplo, el código siguiente crea una instancia de la clase `SearchURL` y hace que el dominio apunte a la nube de Azure Government.

```javascript
var searchURL = new atlas.service.SearchURL(pipeline, 'atlas.azure.us');
```

Si tiene acceso directo a los servicios REST de Azure Maps, cambie el dominio de la dirección URL a `atlas.azure.us`. Por ejemplo, si utiliza el servicio de la API de búsqueda, cambie el dominio de la dirección URL de `https://atlas.microsoft.com/search/` a `https://atlas.azure.us/search/`.

## <a name="next-steps"></a>Pasos siguientes

Aprenda a crear e interactuar con un mapa:

> [!div class="nextstepaction"]
> [Creación de un mapa](map-create.md)

Obtenga información sobre cómo aplicar estilos en un mapa:

> [!div class="nextstepaction"]
> [Elección de un estilo de mapa](choose-map-style.md)

Para agregar más datos al mapa:

> [!div class="nextstepaction"]
> [Creación de un mapa](map-create.md)

> [!div class="nextstepaction"]
> [Ejemplos de código](https://docs.microsoft.com/samples/browse/?products=azure-maps)

Para ver una lista de ejemplos que muestran cómo integrar Azure Active Directory (AAD) con Azure Maps, consulte:

> [!div class="nextstepaction"]
> [Muestras de autenticación de Azure AD](https://github.com/Azure-Samples/Azure-Maps-AzureAD-Samples)
