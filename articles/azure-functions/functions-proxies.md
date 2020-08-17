---
title: Uso de servidores proxy en Azure Functions
description: Información general sobre cómo usar Azure Functions Proxies
ms.topic: conceptual
ms.date: 01/22/2018
ms.openlocfilehash: 3e08b9cf633162cc7015f47774b043cf58c115a0
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87385881"
---
# <a name="work-with-azure-functions-proxies"></a>Uso de Azure Functions Proxies

En este artículo se explica cómo configurar y usar Azure Functions Proxies. Con esta característica, puede especificar puntos de conexión en su aplicación de función que se implementan con otro recurso. Puede utilizar a estos servidores proxy para dividir una API grande en varias Function App (como en una arquitectura de microservicio), mientras que presenta una superficie de API única para los clientes.

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

> [!NOTE] 
> Se aplica la facturación estándar de Functions a las ejecuciones de proxy. Para más información, consulte los [precios de Azure Functions](https://azure.microsoft.com/pricing/details/functions/).

## <a name="create-a-proxy"></a><a name="create"></a>Creación de un proxy

En esta sección se explica cómo crear un proxy en el portal de Functions.

1. Abra [Azure Portal] y vaya a la aplicación de función.
2. En el panel izquierdo, seleccione **Nuevo servidor proxy**.
3. Proporcione un nombre para el proxy.
4. Configure el punto de conexión expuesto en esta aplicación de función; para ello, especifique la **plantilla de ruta** y los **métodos HTTP**. Estos parámetros se comportan según las reglas de los [desencadenadores HTTP].
5. Establezca la **dirección URL de back-end** en otro punto de conexión. El punto de conexión podría tratarse de una función en otra aplicación de función o podría ser cualquier otra API. El valor no necesita ser estático y puede hacer referencia a la [configuración de la aplicación] y a [parámetros de la solicitud de cliente original].
6. Haga clic en **Crear**.

El proxy ahora existe como un nuevo punto de conexión en la Function App. Desde una perspectiva de cliente, es equivalente a un HttpTrigger en Azure Functions. Puede probar el nuevo proxy; para ello, copie la dirección URL del proxy y pruébela con el cliente HTTP favorito.

## <a name="modify-requests-and-responses"></a><a name="modify-requests-responses"></a>Modificación de solicitudes y respuestas

Con Azure Functions Proxies, puede modificar solicitudes y respuestas del back-end. Estas transformaciones pueden usar variables, como se define en [Uso de variables].

### <a name="modify-the-back-end-request"></a><a name="modify-backend-request"></a>Modificación de la solicitud de back-end

De forma predeterminada, la solicitud de back-end se inicializa como copia de la solicitud original. Además de establecer la dirección URL de back-end, se pueden realizar cambios en el método HTTP, los encabezados y los parámetros de cadena de consulta. Los valores modificados pueden hacer referencia a la [configuración de la aplicación] y a los [parámetros de la solicitud de cliente original].

Las solicitudes de back-end pueden modificarse en el portal expandiendo la sección *Invalidación de solicitud* de la página de detalles del proxy. 

### <a name="modify-the-response"></a><a name="modify-response"></a>Modificación de la respuesta

De forma predeterminada, la respuesta del cliente se inicializa como copia de la respuesta de back-end. Puede realizar cambios en el código de estado, la expresión del motivo, los encabezados y el cuerpo de la respuesta. Los valores modificados pueden hacer referencia a la [configuración de la aplicación], a [parámetros de la solicitud de cliente original] y a [parámetros de la respuesta de back-end].

Las solicitudes de back-end pueden modificarse en el portal expandiendo la sección *Invalidación de respuesta* de la página de detalles del proxy. 

## <a name="use-variables"></a><a name="using-variables"></a>Uso de variables

La configuración de un proxy no necesita ser estática. Puede condicionarlo para que use variables de la solicitud de cliente original, la respuesta de back-end o la configuración de la aplicación.

### <a name="reference-local-functions"></a><a name="reference-localhost"></a>Funciones locales de referencia
Puede utilizar `localhost` para hacer referencia a una función dentro de la misma aplicación de función directamente, sin una solicitud de proxy de ida y vuelta.

`"backendurl": "https://localhost/api/httptriggerC#1"` hará referencia a una función desencadenada mediante HTTP local en la ruta `/api/httptriggerC#1`

 
>[!Note]  
>Si la función usa los niveles de autorización de *función, administrador o sistema*, deberá proporcionar el código y el valor de clientId, según la URL de la función original. En este caso, la referencia tendría el siguiente aspecto: `"backendurl": "https://localhost/api/httptriggerC#1?code=<keyvalue>&clientId=<keyname>"` Se recomienda almacenar estas claves en la [configuración de la aplicación] y hacer referencia a ellas en los servidores proxy. De esta forma, se evita tener que almacenar los secretos en el código fuente. 

### <a name="reference-request-parameters"></a><a name="request-parameters"></a>Referencia a los parámetros de solicitud

Puede usar parámetros de solicitud como entradas para la propiedad de dirección URL de back-end o como parte de la modificación de solicitudes y respuestas. Algunos parámetros pueden enlazarse desde la plantilla de ruta especificada en la configuración de proxy base, y otros pueden proceder de propiedades de la solicitud entrante.

#### <a name="route-template-parameters"></a>Parámetros de plantilla de ruta
Los parámetros que se usan en la plantilla de ruta están disponibles para hacer referencia a ellos por su nombre. Los nombres de parámetro se incluyen entre llaves ({}).

Por ejemplo, si un proxy tiene una plantilla de ruta, como `/pets/{petId}`, la dirección URL de back-end puede incluir el valor de `{petId}`, como en `https://<AnotherApp>.azurewebsites.net/api/pets/{petId}`. Si la plantilla de ruta finaliza en un carácter comodín, como `/api/{*restOfPath}`, el valor `{restOfPath}` es una representación de cadena de los segmentos de ruta de acceso restantes procedentes de la solicitud entrante.

#### <a name="additional-request-parameters"></a>Parámetros de solicitud adicionales
Además de los parámetros de la plantilla de ruta, pueden usarse los siguientes valores en la configuración:

* **{request.method}** : método HTTP que se usa en la solicitud original.
* **{request.headers.\<HeaderName\>}** : encabezado que puede leerse desde la solicitud original. Reemplace *\<HeaderName\>* por el nombre del encabezado que quiere leer. Si el encabezado no se incluye en la solicitud, el valor será una cadena vacía.
* **{request.querystring.\<ParameterName\>}** : Parámetro de cadena de consulta que puede leerse de la solicitud original. Reemplace *\<ParameterName\>* por el nombre del parámetro que desea leer. Si el parámetro no se incluye en la solicitud, el valor será una cadena vacía.

### <a name="reference-back-end-response-parameters"></a><a name="response-parameters"></a>Referencia a parámetros de respuesta de back-end

Los parámetros de respuesta pueden utilizarse como parte de la modificación de la respuesta al cliente. Se pueden usar los siguientes valores en la configuración:

* **{backend.response.statusCode}** : código de estado HTTP que se devuelve en la respuesta de back-end.
* **{backend.response.statusReason}** : frase de motivo HTTP que se devuelve en la respuesta de back-end.
* **{backend.response.headers.\<HeaderName\>}** : encabezado que puede leerse desde la respuesta de back-end. Reemplace *\<HeaderName\>* por el nombre del encabezado que quiere leer. Si el encabezado no se incluye en la respuesta, el valor será una cadena vacía.

### <a name="reference-application-settings"></a><a name="use-appsettings"></a>Referencia a la configuración de la aplicación

También puede hacer referencia a la [configuración de la aplicación definida para la aplicación de función](./functions-how-to-use-azure-function-app-settings.md) rodeando el nombre de la configuración con signos de porcentaje (%).

Por ejemplo, en la dirección URL de back-end *https://%ORDER_PROCESSING_HOST%/api/orders* , "%ORDER_PROCESSING_HOST%" se sustituiría por el valor de la opción ORDER_PROCESSING_HOST.

> [!TIP] 
> Use la configuración de la aplicación para hosts back-end si tiene varias implementaciones o entornos de prueba. De este modo, puede asegurarse de que siempre se comunica con el back-end correcto para ese entorno.

## <a name="troubleshoot-proxies"></a><a name="debugProxies"></a>Solución de problemas con Proxies

Si añade la marca `"debug":true` a cualquier proxy de su `proxies.json`, se habilitará el registro de depuración. Los registros se almacenan en `D:\home\LogFiles\Application\Proxies\DetailedTrace` y son accesibles mediante herramientas avanzadas (kudu). Las respuestas HTTP también contendrán un encabezado `Proxy-Trace-Location` con una dirección URL para acceder al archivo de registro.

Puede depurar un proxy del cliente mediante la adición de un encabezado `Proxy-Trace-Enabled` establecido en `true`. De este modo, también se registrará un seguimiento en el sistema de archivos y se devolverá la dirección URL de seguimiento como un encabezado en la respuesta.

### <a name="block-proxy-traces"></a>Bloqueo de seguimiento de proxy

Por motivos de seguridad, es posible que no desee que nadie llame a su servicio para generar un seguimiento. Nadie podrá acceder al contenido de seguimiento sin sus credenciales de inicio de sesión, pero la generación del seguimiento consume recursos y revela que está utilizando Function Proxies.

Deshabilite por completo los seguimientos añadiendo `"debug":false` a cualquier proxy determinado en su `proxies.json`.

## <a name="advanced-configuration"></a>Configuración avanzada

Los servidores proxy que configura se almacenan en un archivo *proxies.json*, ubicado en la raíz de un directorio de la aplicación de función. Puede editar este archivo manualmente e implementarlo como parte de la aplicación cuando se utiliza cualquiera de los [métodos de implementación](./functions-continuous-deployment.md) compatibles con Functions. 

> [!TIP] 
> Si no ha configurado alguno de los métodos de implementación, también puede trabajar con el archivo *proxies.json* en el portal. Vaya a la aplicación de función y seleccione **Características de la plataforma** y **Editor de App Service**. Al hacerlo, puede ver toda la estructura de archivos de la aplicación de función y realizar cambios.

El archivo *proxies.json* lo define un objeto de servidores proxy, compuesto de servidores proxy con nombre y de sus definiciones. De forma opcional, si el editor lo admite, puede hacer referencia a un [esquema JSON](http://json.schemastore.org/proxies) para completar el código. Un archivo de ejemplo puede tener el siguiente aspecto:

```json
{
    "$schema": "http://json.schemastore.org/proxies",
    "proxies": {
        "proxy1": {
            "matchCondition": {
                "methods": [ "GET" ],
                "route": "/api/{test}"
            },
            "backendUri": "https://<AnotherApp>.azurewebsites.net/api/<FunctionName>"
        }
    }
}
```

Cada proxy tiene un nombre descriptivo, como *proxy1* en el ejemplo anterior. El objeto de definición de proxy correspondiente se define mediante las siguientes propiedades:

* **matchCondition**: (requerida) objeto que define las solicitudes que desencadenan la ejecución de este proxy. Contiene dos propiedades compartidas con los [desencadenadores HTTP]:
    * _methods_: matriz de los métodos HTTP a los que responde el servidor proxy. Si no se especifica, el proxy responde a todos los métodos HTTP de la ruta.
    * _route_: (requerida) define la plantilla de ruta y controla a qué direcciones URL de solicitud responde el proxy. A diferencia de los desencadenadores HTTP, no hay ningún valor predeterminado.
* **backendUri**: dirección URL del recurso de back-end a la que se redirigirá la solicitud mediante proxy. Este valor puede hacer referencia a la configuración de la aplicación y a parámetros de la solicitud de cliente original. Si no se incluye esta propiedad, Azure Functions responde con el mensaje HTTP 200 - Correcto.
* **requestOverrides**: objeto que define transformaciones a la solicitud de back-end. Consulte [Definición de un objeto requestOverrides].
* **responseOverrides**: objeto que define transformaciones a la respuesta del cliente. Consulte [Definición de un objeto responseOverrides].

> [!NOTE] 
> La propiedad *route* de Azure Functions Proxies no respeta la propiedad *routePrefix* de la configuración del host de Function App. Si quiere incluir un prefijo como `/api`, se debe incluir en la propiedad *route*.

### <a name="disable-individual-proxies"></a><a name="disableProxies"></a> Deshabilitación de servidores proxy individuales

Puede deshabilitar servidores proxy individuales añadiendo `"disabled": true` al proxy en el archivo `proxies.json`. Esto hará que las solicitudes que cumplan matchCondition devuelvan un error 404.
```json
{
    "$schema": "http://json.schemastore.org/proxies",
    "proxies": {
        "Root": {
            "disabled":true,
            "matchCondition": {
                "route": "/example"
            },
            "backendUri": "https://<AnotherApp>.azurewebsites.net/api/<FunctionName>"
        }
    }
}
```

### <a name="application-settings"></a><a name="applicationSettings"></a> Configuración de la aplicación

El comportamiento del proxy puede controlarse mediante varias opciones de configuración de la aplicación. Opciones que se describen en [Referencia de configuración de aplicación para Azure Functions](./functions-app-settings.md).

* [AZURE_FUNCTION_PROXY_DISABLE_LOCAL_CALL](./functions-app-settings.md#azure_function_proxy_disable_local_call)
* [AZURE_FUNCTION_PROXY_BACKEND_URL_DECODE_SLASHES](./functions-app-settings.md#azure_function_proxy_backend_url_decode_slashes)

### <a name="reserved-characters-string-formatting"></a><a name="reservedChars"></a> Caracteres reservados (formato de cadena)

Los proxies leen todas las cadenas de un archivo JSON usando \ como un símbolo de escape. Los proxies también interpretan las llaves. Consulte a continuación un conjunto completo de ejemplos.

|Carácter|Carácter de escape|Ejemplo|
|-|-|-|
|{ o }|{{ o }}|`{{ example }}` --> `{ example }`
| \ | \\\\ | `example.com\\text.html` --> `example.com\text.html`
|"|\\\"| `\"example\"` --> `"example"`

### <a name="define-a-requestoverrides-object"></a><a name="requestOverrides"></a>Definición de un objeto requestOverrides

El objeto requestOverrides define los cambios realizados en la solicitud cuando se llama al recurso de back-end. El objeto se define mediante las siguientes propiedades:

* **backend.request.method**: método HTTP que se usa para llamar al back-end.
* **backend.request.querystring.\<ParameterName\>** : parámetro de cadena de consulta que se puede establecer para llamar al back-end. Reemplace *\<ParameterName\>* por el nombre del parámetro que quiere establecer. Tenga en cuenta que si se proporciona una cadena vacía, el parámetro se sigue incluyendo en la solicitud de back-end.
* **backend.request.headers.\<HeaderName\>** : encabezado que se puede establecer para llamar al back-end. Reemplace *\<HeaderName\>* por el nombre del encabezado que desea establecer. Tenga en cuenta que si se proporciona una cadena vacía, el parámetro se sigue incluyendo en la solicitud de back-end.

Los valores pueden hacer referencia a la configuración de la aplicación y a los parámetros de la solicitud de cliente original.

Una configuración de ejemplo puede tener el siguiente aspecto:

```json
{
    "$schema": "http://json.schemastore.org/proxies",
    "proxies": {
        "proxy1": {
            "matchCondition": {
                "methods": [ "GET" ],
                "route": "/api/{test}"
            },
            "backendUri": "https://<AnotherApp>.azurewebsites.net/api/<FunctionName>",
            "requestOverrides": {
                "backend.request.headers.Accept": "application/xml",
                "backend.request.headers.x-functions-key": "%ANOTHERAPP_API_KEY%"
            }
        }
    }
}
```

### <a name="define-a-responseoverrides-object"></a><a name="responseOverrides"></a>Definición de un objeto responseOverrides

El objeto requestOverrides define los cambios realizados en la respuesta que se pasa de nuevo al cliente. El objeto se define mediante las siguientes propiedades:

* **response.statusCode**: código de estado HTTP que se va a devolver al cliente.
* **response.statusReason**: frase de motivo HTTP que se va a devolver al cliente.
* **response.body**: representación de cadena del cuerpo que se va a devolver al cliente.
* **response.headers.\<HeaderName\>** : encabezado que se puede establecer para la respuesta al cliente. Reemplace *\<HeaderName\>* por el nombre del encabezado que desea establecer. Si se proporciona una cadena vacía, el encabezado no se incluye en la respuesta.

Los valores pueden hacer referencia a la configuración de la aplicación, a parámetros de la solicitud de cliente original y a parámetros de la respuesta de back-end.

Una configuración de ejemplo puede tener el siguiente aspecto:

```json
{
    "$schema": "http://json.schemastore.org/proxies",
    "proxies": {
        "proxy1": {
            "matchCondition": {
                "methods": [ "GET" ],
                "route": "/api/{test}"
            },
            "responseOverrides": {
                "response.body": "Hello, {test}",
                "response.headers.Content-Type": "text/plain"
            }
        }
    }
}
```
> [!NOTE] 
> En este ejemplo, el cuerpo de la respuesta se define directamente, por lo que no se necesita ninguna propiedad `backendUri`. En el ejemplo, se muestra cómo usar Azure Functions Proxies para simular las API.

[Azure Portal]: https://portal.azure.com
[Desencadenadores HTTP]: ./functions-bindings-http-webhook.md
[Modify the back-end request]: #modify-backend-request
[Modify the response]: #modify-response
[Definición de un objeto requestOverrides]: #requestOverrides
[Definición de un objeto responseOverrides]: #responseOverrides
[Configuración de la aplicación]: #use-appsettings
[Uso de variables]: #using-variables
[Parámetros de la solicitud de cliente original]: #request-parameters
[Parámetros de la respuesta de back-end]: #response-parameters
