---
title: Uso del módulo de servicios de Azure Maps | Microsoft Azure Maps
description: Aprenda a usar el módulo de servicios de Azure Maps. Vea cómo cargar y usar esta biblioteca auxiliar para acceder a los servicios REST de Azure Maps en las aplicaciones web o Node.js.
author: rbrundritt
ms.author: richbrun
ms.date: 03/25/2019
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: cpendleton
ms.custom: devx-track-javascript
ms.openlocfilehash: 2f8d41c8248ab761c6d781263f3978937ad01e22
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88004144"
---
# <a name="use-the-azure-maps-services-module"></a>Uso del módulo de servicios de Azure Maps

El SDK web de Azure Maps proporciona un *módulo de servicios*. Este módulo es una biblioteca de ayuda que facilita el uso de los servicios REST de Azure Maps en la web o aplicaciones de Node.js mediante JavaScript o TypeScript.

## <a name="use-the-services-module-in-a-webpage"></a>Uso del módulo de servicios en una página web

1. Cree un archivo HTML.
1. Cargue el módulo de servicios de Azure Maps. Puede realizar ese procedimiento de alguna de estas dos formas:
    - Utilizar la versión de Azure Content Delivery Network hospedada globalmente del módulo de servicios de Azure Maps. Agregue una referencia de script al elemento `<head>` del archivo:

        ```html
        <script src="https://atlas.microsoft.com/sdk/javascript/service/2/atlas-service.min.js"></script>
        ```

    - Si lo desea, también puede cargar localmente el módulo de servicios del SDK web de Azure Maps utilizando el paquete npm [azure-maps-rest](https://www.npmjs.com/package/azure-maps-rest) y hospedarlo después con la aplicación. Este paquete también incluye las definiciones de TypeScript. Use este comando:
    
        > **npm install azure-maps-rest**
    
        A continuación, agregue una referencia de script al elemento `<head>` del archivo:

         ```html
        <script src="node_modules/azure-maps-rest/dist/atlas-service.min.js"></script>
         ```

1. Cree una canalización de autenticación. Para poder inicializar un punto de conexión de cliente con la dirección URL del servicio, primero debe crearse la canalización. Use su propia clave de cuenta de Azure Maps o las credenciales de Azure Active Directory (Azure AD) para autenticar el cliente de servicio de búsqueda de Azure Maps. En este ejemplo, se creará el cliente de dirección URL de servicio de búsqueda. 

    Si usa una clave de suscripción para la autenticación:

    ```javascript
    // Get an Azure Maps key at https://azure.com/maps.
    var subscriptionKey = '<Your Azure Maps Key>';

    // Use SubscriptionKeyCredential with a subscription key.
    var subscriptionKeyCredential = new atlas.service.SubscriptionKeyCredential(subscriptionKey);

    // Use subscriptionKeyCredential to create a pipeline.
    var pipeline = atlas.service.MapsURL.newPipeline(subscriptionKeyCredential, {
      retryOptions: { maxTries: 4 } // Retry options
    });

    // Create an instance of the SearchURL client.
    var searchURL = new atlas.service.SearchURL(pipeline);
    ```

    Si usa una Azure AD para la autenticación:

    ```javascript
    // Enter your Azure AD client ID.
    var clientId = "<Your Azure Active Directory Client Id>";

    // Use TokenCredential with OAuth token (Azure AD or Anonymous).
    var aadToken = await getAadToken();
    var tokenCredential = new atlas.service.TokenCredential(clientId, aadToken);

    // Create a repeating time-out that will renew the Azure AD token.
    // This time-out must be cleared when the TokenCredential object is no longer needed.
    // If the time-out is not cleared, the memory used by the TokenCredential will never be reclaimed.
    var renewToken = async () => {
      try {
        console.log("Renewing token");
        var token = await getAadToken();
        tokenCredential.token = token;
        tokenRenewalTimer = setTimeout(renewToken, getExpiration(token));
      } catch (error) {
        console.log("Caught error when renewing token");
        clearTimeout(tokenRenewalTimer);
        throw error;
      }
    }
    tokenRenewalTimer = setTimeout(renewToken, getExpiration(aadToken));

    // Use tokenCredential to create a pipeline.
    var pipeline = atlas.service.MapsURL.newPipeline(tokenCredential, {
      retryOptions: { maxTries: 4 } // Retry options
    });

    // Create an instance of the SearchURL client.
    var searchURL = new atlas.service.SearchURL(pipeline);

    function getAadToken() {
      // Use the signed-in auth context to get a token.
      return new Promise((resolve, reject) => {
        // The resource should always be https://atlas.microsoft.com/.
        const resource = "https://atlas.microsoft.com/";
        authContext.acquireToken(resource, (error, token) => {
          if (error) {
            reject(error);
          } else {
            resolve(token);
          }
        });
      })
    }

    function getExpiration(jwtToken) {
      // Decode the JSON Web Token (JWT) to get the expiration time stamp.
      const json = atob(jwtToken.split(".")[1]);
      const decode = JSON.parse(json);

      // Return the milliseconds remaining until the token must be renewed.
      // Reduce the time until renewal by 5 minutes to avoid using an expired token.
      // The exp property is the time stamp of the expiration, in seconds.
      const renewSkew = 300000;
      return (1000 * decode.exp) - Date.now() - renewSkew;
    }
    ```

    Para obtener más información, consulte [Autenticación con Azure Maps](azure-maps-authentication.md).

1. El código siguiente usa el cliente de dirección URL de servicio de Azure Maps Search recién creado para establecer un código geográfico de una dirección: "1 Microsoft Way, Redmond, WA". El código usa la función `searchAddress` y muestra los resultados en forma de tabla en el cuerpo de la página.

    ```javascript
    // Search for "1 microsoft way, redmond, wa".
    searchURL.searchAddress(atlas.service.Aborter.timeout(10000), '1 microsoft way, redmond, wa')
      .then(response => {
        var html = [];

        // Display the total results.
        html.push('Total results: ', response.summary.numResults, '<br/><br/>');

        // Create a table of the results.
        html.push('<table><tr><td></td><td>Result</td><td>Latitude</td><td>Longitude</td></tr>');

        for(var i=0;i<response.results.length;i++){
          html.push('<tr><td>', (i+1), '.</td><td>', 
            response.results[i].address.freeformAddress, 
            '</td><td>', 
            response.results[i].position.lat,
            '</td><td>', 
            response.results[i].position.lon,
            '</td></tr>');
        }

        html.push('</table>');

        // Add the resulting HTML to the body of the page.
        document.body.innerHTML = html.join('');
    });
    ```

    Aquí se muestra el ejemplo de código de ejecución completo:

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Uso del módulo de servicios" src="//codepen.io/azuremaps/embed/zbXGMR/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder="no" allowtransparency="true" allowfullscreen="true">
Consulte el Pen <a href='https://codepen.io/azuremaps/pen/zbXGMR/'>Using the Services Module</a> (Uso de módulo de servicios) de Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) en <a href='https://codepen.io'>CodePen</a>.
</iframe>

<br/>

## <a name="azure-government-cloud-support"></a>Compatibilidad con la nube de Azure Government

El SDK web de Azure Maps es compatible con la nube de Azure Government. Todas las direcciones URL de JavaScript y CSS que se usan para acceder al SDK web de Azure Maps siguen siendo las mismas; sin embargo, es necesario realizar las tareas siguientes para conectarse a la versión de la plataforma Azure Maps situada en la nube de Azure Government.

Cuando utilice el control de mapas interactivos, agregue la siguiente línea de código antes de crear una instancia de la clase `Map`. 

```javascript
atlas.setDomain('atlas.azure.us');
```

No olvide usar los datos de autenticación de Azure Maps de la plataforma en la nube de Azure Government cuando autentique el mapa y los servicios.

Si utiliza el módulo de servicios, el dominio de los servicios debe definirse al crear una instancia de un punto de conexión de la dirección URL de la API. Por ejemplo, el código siguiente crea una instancia de la clase `SearchURL` y hace que el dominio apunte a la nube de Azure Government.

```javascript
var searchURL = new atlas.service.SearchURL(pipeline, 'atlas.azure.us');
```

Si tiene acceso directo a los servicios REST de Azure Maps, cambie el dominio de la dirección URL a `atlas.azure.us`. Por ejemplo, si utiliza el servicio de la API de búsqueda, cambie el dominio de la dirección URL de `https://atlas.microsoft.com/search/` a `https://atlas.azure.us/search/`.

## <a name="next-steps"></a>Pasos siguientes

Más información sobre las clases y los métodos utilizados en este artículo:

> [!div class="nextstepaction"]
> [MapsURL](https://docs.microsoft.com/javascript/api/azure-maps-rest/atlas.service.mapsurl?view=azure-maps-typescript-latest)

> [!div class="nextstepaction"]
> [SearchURL](https://docs.microsoft.com/javascript/api/azure-maps-rest/atlas.service.searchurl?view=azure-maps-typescript-latest)

> [!div class="nextstepaction"]
> [RouteURL](https://docs.microsoft.com/javascript/api/azure-maps-rest/atlas.service.routeurl?view=azure-maps-typescript-latest)

> [!div class="nextstepaction"]
> [SubscriptionKeyCredential](https://docs.microsoft.com/javascript/api/azure-maps-rest/atlas.service.subscriptionkeycredential?view=azure-maps-typescript-latest)

> [!div class="nextstepaction"]
> [TokenCredential](https://docs.microsoft.com/javascript/api/azure-maps-rest/atlas.service.tokencredential?view=azure-maps-typescript-latest)

Para obtener más ejemplos de código que usan el módulo de servicios, consulte estos artículos:

> [!div class="nextstepaction"]
> [Presentación de los resultados de la búsqueda en el mapa](./map-search-location.md)

> [!div class="nextstepaction"]
> [Obtención de información de una coordenada](./map-get-information-from-coordinate.md)

> [!div class="nextstepaction"]
> [Presentación de indicaciones de ruta de A a B](./map-route.md)
