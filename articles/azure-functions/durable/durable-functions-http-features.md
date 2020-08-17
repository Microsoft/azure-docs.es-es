---
title: 'Características de HTTP en Durable Functions: Azure Functions'
description: Aprenda a implementar las características de HTTP integradas en la extensión Durable Functions para Azure Functions.
author: cgillum
ms.topic: conceptual
ms.date: 07/14/2020
ms.author: azfuncdf
ms.openlocfilehash: 16a133205b13a3d0a4aa76f75c8ce316f6c09199
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87014905"
---
# <a name="http-features"></a>Características de HTTP

Durable Functions tiene varias características que facilitan la incorporación de orquestaciones y entidades duraderas en flujos de trabajo HTTP. En este artículo se detallan algunas de esas características.

## <a name="exposing-http-apis"></a>Exposición de las API HTTP

Las orquestaciones y entidades se pueden invocar y administrar mediante solicitudes HTTP. La extensión Durable Functions expone las API de HTTP integradas. También proporciona API para interactuar con las orquestaciones y entidades desde las funciones desencadenadas por HTTP.

### <a name="built-in-http-apis"></a>API HTTP integradas

La extensión de Durable Functions agrega automáticamente un conjunto de API HTTP al host de Azure Functions. Con estas API, puede interactuar con las orquestaciones y entidades, y administrarlas sin necesidad de escribir ningún código.

Se admiten los siguientes tipos de API HTTP integradas.

* [Iniciar nueva orquestación](durable-functions-http-api.md#start-orchestration)
* [Consultar instancia de orquestación](durable-functions-http-api.md#get-instance-status)
* [Terminar instancia de orquestación](durable-functions-http-api.md#terminate-instance)
* [Enviar un evento externo a una orquestación](durable-functions-http-api.md#raise-event)
* [Purgar el historial de orquestación](durable-functions-http-api.md#purge-single-instance-history)
* [Enviar un evento de operación](durable-functions-http-api.md#signal-entity)
* [Obtener el estado de una entidad](durable-functions-http-api.md#get-entity)
* [Consultar la lista de entidades](durable-functions-http-api.md#list-entities)

Consulte el [artículo API HTTP](durable-functions-http-api.md) para obtener una descripción completa de todas las API de HTTP integradas que expone la extensión de Durable Functions.

### <a name="http-api-url-discovery"></a>Detección de la dirección URL de la API de HTTP

El [enlace del cliente de orquestación](durable-functions-bindings.md#orchestration-client) expone las API que pueden generar las cargas de respuesta HTTP útiles. Por ejemplo, puede crear una respuesta que contenga vínculos a las API de administración de una instancia de orquestación específica. Los siguientes ejemplos muestran una función de desencadenador HTTP que muestra cómo utilizar esta API para una nueva instancia de orquestación:

# <a name="c"></a>[C#](#tab/csharp)

[!code-csharp[Main](~/samples-durable-functions/samples/precompiled/HttpStart.cs)]

# <a name="javascript"></a>[JavaScript](#tab/javascript)

**index.js**

[!code-javascript[Main](~/samples-durable-functions/samples/javascript/HttpStart/index.js)]

**function.json**

[!code-json[Main](~/samples-durable-functions/samples/javascript/HttpStart/function.json)]

# <a name="python"></a>[Python](#tab/python)

**__init__.py**

```python
import logging
import azure.functions as func
import azure.durable_functions as df

async def main(req: func.HttpRequest, starter: str) -> func.HttpResponse:
    client = df.DurableOrchestrationClient(starter)
    function_name = req.route_params['functionName']
    event_data = req.get_body()

    instance_id = await client.start_new(function_name, instance_id, event_data)
    
    logging.info(f"Started orchestration with ID = '{instance_id}'.")
    return client.create_check_status_response(req, instance_id)
```

**function.json**

```json
{
  "scriptFile": "__init__.py",
  "bindings": [
    {
      "authLevel": "function",
      "name": "req",
      "type": "httpTrigger",
      "direction": "in",
      "route": "orchestrators/{functionName}",
      "methods": [
        "post",
        "get"
      ]
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    },
    {
      "name": "starter",
      "type": "orchestrationClient",
      "direction": "in"
    }
  ]
}
```

---

El inicio de una función de orquestador mediante las funciones de desencadenador HTTP mostradas anteriormente se puede hacer con cualquier cliente HTTP. El siguiente comando de cURL inicia una función de orquestador denominada `DoWork`:

```bash
curl -X POST https://localhost:7071/orchestrators/DoWork -H "Content-Length: 0" -i
```

A continuación se incluye un ejemplo de respuesta para una orquestación que tiene `abc123` como su identificador. Se quitaron algunos detalles para una mayor claridad.

```http
HTTP/1.1 202 Accepted
Content-Type: application/json; charset=utf-8
Location: http://localhost:7071/runtime/webhooks/durabletask/instances/abc123?code=XXX
Retry-After: 10

{
    "id": "abc123",
    "purgeHistoryDeleteUri": "http://localhost:7071/runtime/webhooks/durabletask/instances/abc123?code=XXX",
    "sendEventPostUri": "http://localhost:7071/runtime/webhooks/durabletask/instances/abc123/raiseEvent/{eventName}?code=XXX",
    "statusQueryGetUri": "http://localhost:7071/runtime/webhooks/durabletask/instances/abc123?code=XXX",
    "terminatePostUri": "http://localhost:7071/runtime/webhooks/durabletask/instances/abc123/terminate?reason={text}&code=XXX"
}
```

En el ejemplo anterior, cada uno de los campos que terminan en `Uri`corresponde a las API de HTTP integradas. Puede usar estas API para administrar la instancia de orquestación de destino.

> [!NOTE]
> El formato de las direcciones URL del webhook depende de la versión del host de Azure Functions que se esté ejecutando. El ejemplo anterior es para el host de Azure Functions 2.0.

Para una descripción de todas las API de HTTP integradas, consulte la [referencia de la API de HTTP](durable-functions-http-api.md).

### <a name="async-operation-tracking"></a>Seguimiento de operaciones asincrónicas

La respuesta HTTP que se ha mencionado anteriormente está diseñada para ayudar a implementar API de HTTP asincrónicas de ejecución prolongada con Durable Functions. Este patrón se conoce también como *Patrón de sondeo del consumidor*. El flujo cliente/servidor funciona del siguiente modo:

1. El cliente emite una solicitud HTTP para iniciar un proceso de ejecución prolongada como una función de orquestador.
1. El desencadenador HTTP de destino devuelve una respuesta HTTP 202 con un encabezado de ubicación que tiene el valor "statusQueryGetUri".
1. El cliente sondea la dirección URL en el encabezado de ubicación. El cliente continúa viendo las respuestas HTTP 202 con un encabezado de ubicación.
1. Cuando la instancia finaliza o se produce un error, el punto de conexión en el encabezado de ubicación devuelve HTTP 200.

Este protocolo permite la coordinación de procesos de ejecución prolongada con clientes o servicios externos que pueden sondear un punto de conexión HTTP y seguir el encabezado de ubicación. Las implementaciones de cliente y de servidor de este patrón están integradas en las API HTTP de Durable Functions.

> [!NOTE]
> De forma predeterminada, todas las acciones basadas en HTTP proporcionadas por [Azure Logic Apps](https://azure.microsoft.com/services/logic-apps/) admiten el patrón estándar de operación asincrónica. Esta funcionalidad permite insertar una función durable de ejecución prolongada como parte del flujo de trabajo de Logic Apps. Puede encontrar más detalles sobre la compatibilidad de Logic Apps con los patrones asincrónicos de HTTP en la [documentación de los desencadenadores y acciones del flujo de trabajo de Azure Logic Apps](../../logic-apps/logic-apps-workflow-actions-triggers.md).

> [!NOTE]
> Las interacciones con las orquestaciones se puede realizar desde cualquier tipo de función, no solo con las funciones desencadenadas por HTTP.

Para más información acerca de la administración de orquestaciones y entidades mediante las API cliente, consulte el [artículo sobre administración de instancias](durable-functions-instance-management.md).

## <a name="consuming-http-apis"></a>Consumo de API HTTP

Como se describe en las [restricciones de código de las funciones de orquestador](durable-functions-code-constraints.md), las funciones del orquestador no pueden hacer E/S directamente. En su lugar, suelen llamar a [funciones de actividad](durable-functions-types-features-overview.md#activity-functions) que realizan operaciones de E/S.

A partir de Durable Functions 2.0, las orquestaciones pueden consumir de forma nativa las API de HTTP mediante el [enlace desencadenador de orquestación](durable-functions-bindings.md#orchestration-trigger).

En el ejemplo de código siguiente se muestra una función de orquestador que realiza una solicitud HTTP saliente:

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("CheckSiteAvailable")]
public static async Task CheckSiteAvailable(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    Uri url = context.GetInput<Uri>();

    // Makes an HTTP GET request to the specified endpoint
    DurableHttpResponse response = 
        await context.CallHttpAsync(HttpMethod.Get, url);

    if (response.StatusCode >= 400)
    {
        // handling of error codes goes here
    }
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context){
    const url = context.df.getInput();
    const response = yield context.df.callHttp("GET", url)

    if (response.statusCode >= 400) {
        // handling of error codes goes here
    }
});
```
# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df
from datetime import datetime, timedelta

def orchestrator_function(context: df.DurableOrchestrationContext):
    url = context.get_input()
    response = yield context.call_http('GET', url)
    
    if response["statusCode"] >= 400:
        # handling of error codes goes here

main = df.Orchestrator.create(orchestrator_function)
```

---

Mediante la acción "llamar a HTTP", puede realizar las siguientes acciones en las funciones de orquestador:

* Llamar a las API de HTTP directamente desde las funciones de orquestador, con algunas limitaciones que se mencionan más adelante.
* Admite automáticamente los patrones de sondeo de estado HTTP 202 del lado del cliente.
* Use [Identidades administradas de Azure](../../active-directory/managed-identities-azure-resources/overview.md) para realizar llamadas HTTP autorizadas a otros puntos de conexión de Azure.

La capacidad de usar API HTTP directamente desde las funciones de orquestador resulta conveniente en un determinado conjunto de escenarios comunes. Puede implementar todas estas características mediante las funciones de actividad. En muchos casos, las funciones de actividad pueden proporcionar más flexibilidad.

### <a name="http-202-handling"></a>Control de HTTP 202

La API "llamar a HTTP" puede implementar automáticamente el lado cliente del patrón de sondeo del consumidor. Si una API llamada devuelve una respuesta HTTP 202 con un encabezado de ubicación, la función de orquestador sondea automáticamente el recurso de ubicación hasta recibir una respuesta distinta de 202. Esta respuesta será la que se devuelva al código de la función de orquestador.

> [!NOTE]
> Las funciones de orquestador también admiten de forma nativa el patrón de sondeo del consumidor en el lado servidor, tal y como se describe en [Seguimiento de operaciones asincrónicas](#async-operation-tracking). Esta compatibilidad significa que las orquestaciones en una aplicación de función pueden coordinar fácilmente las funciones del orquestador en las demás aplicaciones de función. Esto es similar al concepto de [suborquestación](durable-functions-sub-orchestrations.md), pero con compatibilidad para la comunicación entre aplicaciones. Esta compatibilidad es especialmente útil para el desarrollo de aplicaciones del estilo microservicio.

### <a name="managed-identities"></a>Identidades administradas

Durable Functions admite de forma nativa llamadas a las API que aceptan tokens de Azure Active Directory (Azure AD) para la autorización. Esta compatibilidad usa [identidades administradas de Azure](../../active-directory/managed-identities-azure-resources/overview.md) para adquirir estos tokens.

El código siguiente es un ejemplo de una función de orquestadoror de .NET. La función realiza llamadas autenticadas para reiniciar una máquina virtual mediante la API de REST de [máquinas virtuales de Azure Resource Manager](/rest/api/compute/virtualmachines).

# <a name="c"></a>[C#](#tab/csharp)

```csharp
[FunctionName("RestartVm")]
public static async Task RunOrchestrator(
    [OrchestrationTrigger] IDurableOrchestrationContext context)
{
    string subscriptionId = "mySubId";
    string resourceGroup = "myRG";
    string vmName = "myVM";
    string apiVersion = "2019-03-01";
    
    // Automatically fetches an Azure AD token for resource = https://management.core.windows.net
    // and attaches it to the outgoing Azure Resource Manager API call.
    var restartRequest = new DurableHttpRequest(
        HttpMethod.Post, 
        new Uri($"https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.Compute/virtualMachines/{vmName}/restart?api-version={apiVersion}"),
        tokenSource: new ManagedIdentityTokenSource("https://management.core.windows.net"));
    DurableHttpResponse restartResponse = await context.CallHttpAsync(restartRequest);
    if (restartResponse.StatusCode != HttpStatusCode.OK)
    {
        throw new ArgumentException($"Failed to restart VM: {restartResponse.StatusCode}: {restartResponse.Content}");
    }
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = df.orchestrator(function*(context) {
    const subscriptionId = "mySubId";
    const resourceGroup = "myRG";
    const vmName = "myVM";
    const apiVersion = "2019-03-01";
    const tokenSource = new df.ManagedIdentityTokenSource("https://management.core.windows.net");

    // get a list of the Azure subscriptions that I have access to
    const restartResponse = yield context.df.callHttp(
        "POST",
        `https://management.azure.com/subscriptions/${subscriptionId}/resourceGroups/${resourceGroup}/providers/Microsoft.Compute/virtualMachines/${vmName}/restart?api-version=${apiVersion}`,
        undefined, // no request content
        undefined, // no request headers (besides auth which is handled by the token source)
        tokenSource);

    return restartResponse;
});
```

# <a name="python"></a>[Python](#tab/python)

```python
import azure.functions as func
import azure.durable_functions as df

def orchestrator_function(context: df.DurableOrchestrationContext):
    subscription_id = "mySubId"
    resource_group = "myRg"
    vm_name = "myVM"
    api_version = "2019-03-01"
    token_source = df.ManagedIdentityTokenSource("https://management.core.windows.net")

    # get a list of the Azure subscriptions that I have access to
    restart_response = yield context.call_http("POST", 
        f"https://management.azure.com/subscriptions/${subscription_id}/resourceGroups/${resource_group}/providers/Microsoft.Compute/virtualMachines/${vm_name}/restart?api-version=${api_version}",
        None,
        None,
        token_source)
    return restart_response

main = df.Orchestrator.create(orchestrator_function)
```

---

En el ejemplo anterior, el parámetro `tokenSource` se configura para adquirir tokens de Azure AD para [Azure Resource Manager](../../azure-resource-manager/management/overview.md). Los tokens se identifican mediante el URI de recurso `https://management.core.windows.net`. En el ejemplo se da por supuesto que la aplicación de funciones actual se está ejecutando localmente o se ha implementado como una aplicación de función con una identidad administrada. Se supone que la identidad local o la identidad administrada tienen permiso para administrar las máquinas virtuales en el grupo de recursos especificado `myRG`.

En runtime, el origen de token configurado devuelve automáticamente un token de acceso de OAuth 2.0. Después, el origen agrega el token como un token de portador al encabezado de Authorization de la solicitud saliente. Este modelo es una mejora respecto a la adición manual de encabezados de autorización a las solicitudes HTTP por los siguientes motivos:

* La actualización de tokens se controla automáticamente. No es necesario preocuparse por los tokens expirados.
* Los tokens nunca se almacenan en el estado de orquestación duradero.
* No es necesario escribir código para administrar la adquisición de tokens.

Puede encontrar un ejemplo más completo en la [muestra precompilada de C# RestartVMs](https://github.com/Azure/azure-functions-durable-extension/blob/dev/samples/precompiled/RestartVMs.cs).

Las identidades administradas no se limitan a la administración de recursos de Azure. Puede usar identidades administradas para acceder a cualquier API que acepte tokens de portador de Azure AD, incluidos los servicios de Azure de Microsoft y las aplicaciones Web de los asociados. Una aplicación Web de asociado puede ser también otra aplicación de funciones. Para ver la lista de servicios de Azure de Microsoft que admiten la autenticación con Azure AD, consulte [Servicios de Azure que admiten la autenticación de Azure AD](../../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication).

### <a name="limitations"></a>Limitaciones

La compatibilidad integrada para llamar a las API HTTP es una característica útil. No es adecuada para todos los escenarios.

Las solicitudes HTTP enviadas por las funciones de orquestador y sus respuestas se serializan y se conservan como mensajes en cola. Este comportamiento de puesta en cola garantiza que las llamadas HTTP sean [confiables y seguras para la reproducción de orquestación](durable-functions-orchestrations.md#reliability). Sin embargo, el comportamiento de puesta en cola también tiene limitaciones:

* Cada solicitud HTTP implica una latencia adicional en comparación con un cliente HTTP nativo.
* Los mensajes de solicitud o respuesta de gran tamaño que no caben en un mensaje de cola pueden degradar significativamente el rendimiento de la orquestación. La sobrecarga que supone descargar cargas de mensajes en el almacenamiento de blobs puede causar una degradación potencial del rendimiento.
* No se admiten las cargas de streaming, fragmentadas ni binarias.
* La capacidad de personalizar el comportamiento del cliente HTTP es limitada.

Si alguna de estas limitaciones puede afectar a su caso de uso, considere la posibilidad de usar en su lugar las funciones de actividad y las bibliotecas de cliente HTTP específicas del lenguaje para realizar llamadas HTTP salientes.

> [!NOTE]
> Si es un desarrollador de .NET, es posible que se pregunte por qué esta función utiliza los tipos **DurableHttpRequest** y **DurableHttpResponse** en lugar de los tipos integrados de .NET **HttpRequestMessage** y **HttpResponseMessage** .
>
> Esta opción de diseño es intencionada. La razón principal es que los tipos personalizados ayudan a garantizar que los usuarios no realicen suposiciones incorrectas sobre los comportamientos admitidos del cliente HTTP interno. Los tipos específicos de Durable Functions también permiten simplificar el diseño de la API. También pueden poner a disposición más fácilmente características especiales como [integración de identidad administrada](#managed-identities) y el [patrón de consumidor de sondeo](#http-202-handling). 

### <a name="extensibility-net-only"></a>Extensibilidad (solo .NET)

La personalización del comportamiento del cliente HTTP interno de la orquestación es posible mediante [Inserción de dependencias en Azure Functions con .NET](../functions-dotnet-dependency-injection.md). Esta capacidad puede ser útil para realizar pequeños cambios de comportamiento. También puede ser útil para las pruebas unitarias del cliente HTTP mediante la inserción de objetos ficticios.

En el ejemplo siguiente se muestra cómo usar la inserción de dependencias para deshabilitar la validación de certificados TLS/SSL para las funciones del orquestador que llaman a puntos de conexión HTTP externos.

```csharp
public class Startup : FunctionsStartup
{
    public override void Configure(IFunctionsHostBuilder builder)
    {
        // Register own factory
        builder.Services.AddSingleton<
            IDurableHttpMessageHandlerFactory,
            MyDurableHttpMessageHandlerFactory>();
    }
}

public class MyDurableHttpMessageHandlerFactory : IDurableHttpMessageHandlerFactory
{
    public HttpMessageHandler CreateHttpMessageHandler()
    {
        // Disable TLS/SSL certificate validation (not recommended in production!)
        return new HttpClientHandler
        {
            ServerCertificateCustomValidationCallback =
                HttpClientHandler.DangerousAcceptAnyServerCertificateValidator,
        };
    }
}
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Más información sobre las entidades duraderas](durable-functions-entities.md)
