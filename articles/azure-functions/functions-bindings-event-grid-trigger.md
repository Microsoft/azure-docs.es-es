---
title: Desencadenador de Azure Event Grid para Azure Functions
description: Aprenda a ejecutar código cuando se envíen eventos de Event Grid en Azure Functions.
author: craigshoemaker
ms.topic: reference
ms.date: 02/14/2020
ms.author: cshoe
ms.custom: fasttrack-edit, devx-track-python
ms.openlocfilehash: 78b15e0ba73743f6d88d28b03c99ca64d73dfae1
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87848441"
---
# <a name="azure-event-grid-trigger-for-azure-functions"></a>Desencadenador de Azure Event Grid para Azure Functions

Use el desencadenador de funciones para responder a un evento enviado a un tema de Event Grid.

Para obtener información sobre los detalles de instalación y configuración, consulte la [información general](./functions-bindings-event-grid.md).

## <a name="example"></a>Ejemplo

# <a name="c"></a>[C#](#tab/csharp)

Para ver un ejemplo de un desencadenador HTTP, consulte [Recepción de eventos en un punto de conexión HTTP](../event-grid/receive-events.md).

### <a name="c-2x-and-higher"></a>C# (2.x y versiones posteriores)

En el ejemplo siguiente se muestra una [función de C#](functions-dotnet-class-library.md) que se enlaza a `EventGridEvent`:

```cs
using Microsoft.Azure.EventGrid.Models;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.EventGrid;
using Microsoft.Azure.WebJobs.Host;
using Microsoft.Extensions.Logging;

namespace Company.Function
{
    public static class EventGridTriggerCSharp
    {
        [FunctionName("EventGridTest")]
        public static void EventGridTest([EventGridTrigger]EventGridEvent eventGridEvent, ILogger log)
        {
            log.LogInformation(eventGridEvent.Data.ToString());
        }
    }
}
```

Para más información, consulte Paquetes, [Atributos](#attributes-and-annotations), [Configuración](#configuration) y [Uso](#usage).

### <a name="version-1x"></a>Versión 1.x

En el ejemplo siguiente se muestra una [función de C#](functions-dotnet-class-library.md) de Functions 1.x que enlaza a `JObject`:

```cs
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.EventGrid;
using Microsoft.Azure.WebJobs.Host;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;
using Microsoft.Extensions.Logging;

namespace Company.Function
{
    public static class EventGridTriggerCSharp
    {
        [FunctionName("EventGridTriggerCSharp")]
        public static void Run([EventGridTrigger]JObject eventGridEvent, ILogger log)
        {
            log.LogInformation(eventGridEvent.ToString(Formatting.Indented));
        }
    }
}
```

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

En el ejemplo siguiente se muestra un enlace de desencadenador en un archivo *function.json* y una [función de script de C#](functions-reference-csharp.md) que usa el enlace.

Estos son los datos de enlace del archivo *function.json*:

```json
{
  "bindings": [
    {
      "type": "eventGridTrigger",
      "name": "eventGridEvent",
      "direction": "in"
    }
  ],
  "disabled": false
}
```

### <a name="version-2x-and-higher"></a>Versión 2.x y posteriores

Este es un ejemplo que enlaza a `EventGridEvent`:

```csharp
#r "Microsoft.Azure.EventGrid"
using Microsoft.Azure.EventGrid.Models;
using Microsoft.Extensions.Logging;

public static void Run(EventGridEvent eventGridEvent, ILogger log)
{
    log.LogInformation(eventGridEvent.Data.ToString());
}
```

Para más información, consulte Paquetes, [Atributos](#attributes-and-annotations), [Configuración](#configuration) y [Uso](#usage).

### <a name="version-1x"></a>Versión 1.x

Este es el código de script de C# de Functions 1.x que enlaza a `JObject`:

```cs
#r "Newtonsoft.Json"

using Newtonsoft.Json;
using Newtonsoft.Json.Linq;

public static void Run(JObject eventGridEvent, TraceWriter log)
{
    log.Info(eventGridEvent.ToString(Formatting.Indented));
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

En el ejemplo siguiente se muestra un enlace de desencadenador en un archivo *function.json* y una [función de JavaScript](functions-reference-node.md) que usa el enlace.

Estos son los datos de enlace del archivo *function.json*:

```json
{
  "bindings": [
    {
      "type": "eventGridTrigger",
      "name": "eventGridEvent",
      "direction": "in"
    }
  ],
  "disabled": false
}
```

Este es el código de JavaScript:

```javascript
module.exports = function (context, eventGridEvent) {
    context.log("JavaScript Event Grid function processed a request.");
    context.log("Subject: " + eventGridEvent.subject);
    context.log("Time: " + eventGridEvent.eventTime);
    context.log("Data: " + JSON.stringify(eventGridEvent.data));
    context.done();
};
```

# <a name="python"></a>[Python](#tab/python)

En el ejemplo siguiente se muestra un enlace de desencadenador en un archivo *function.json* y una [función de Python](functions-reference-python.md) que usa el enlace.

Estos son los datos de enlace del archivo *function.json*:

```json
{
  "bindings": [
    {
      "type": "eventGridTrigger",
      "name": "event",
      "direction": "in"
    }
  ],
  "disabled": false,
  "scriptFile": "__init__.py"
}
```

Este es el código de Python:

```python
import json
import logging

import azure.functions as func

def main(event: func.EventGridEvent):

    result = json.dumps({
        'id': event.id,
        'data': event.get_json(),
        'topic': event.topic,
        'subject': event.subject,
        'event_type': event.event_type,
    })

    logging.info('Python EventGrid trigger processed an event: %s', result)
```

# <a name="java"></a>[Java](#tab/java)

En esta sección se incluyen los ejemplos siguientes:

* [Desencadenador de Event Grid, parámetro de cadena](#event-grid-trigger-string-parameter)
* [Desencadenador de Event Grid, parámetro POJO](#event-grid-trigger-pojo-parameter)

En los siguientes ejemplos, se muestra un enlace de desencadenador de [Java](functions-reference-java.md) que utiliza el enlace e imprime un evento. El evento se recibe primero como `String` y después como POJO.

### <a name="event-grid-trigger-string-parameter"></a>Desencadenador de Event Grid, parámetro de cadena

```java
  @FunctionName("eventGridMonitorString")
  public void logEvent(
    @EventGridTrigger(
      name = "event"
    ) 
    String content, 
    final ExecutionContext context) {
      context.getLogger().info("Event content: " + content);      
  }
```

### <a name="event-grid-trigger-pojo-parameter"></a>Desencadenador de Event Grid, parámetro POJO

En este ejemplo se usa el siguiente objeto POJO, que representa las propiedades de nivel superior de un evento de Event Grid:

```java
import java.util.Date;
import java.util.Map;

public class EventSchema {

  public String topic;
  public String subject;
  public String eventType;
  public Date eventTime;
  public String id;
  public String dataVersion;
  public String metadataVersion;
  public Map<String, Object> data;

}
```

Al llegar, la carga útil JSON del evento se deserializa en el POJO ```EventSchema``` para que la función pueda usarla. Este proceso permite que la función obtenga acceso a las propiedades del evento de manera que esté orientada a los objetos.

```java
  @FunctionName("eventGridMonitor")
  public void logEvent(
    @EventGridTrigger(
      name = "event"
    ) 
    EventSchema event, 
    final ExecutionContext context) {
      context.getLogger().info("Event content: ");
      context.getLogger().info("Subject: " + event.subject);
      context.getLogger().info("Time: " + event.eventTime); // automatically converted to Date by the runtime
      context.getLogger().info("Id: " + event.id);
      context.getLogger().info("Data: " + event.data);
  }
```

En la [biblioteca en tiempo de ejecución de funciones de Java](/java/api/overview/azure/functions/runtime), utilice la anotación `EventGridTrigger` en los parámetros cuyo valor provendría de EventGrid. Los parámetros con estas anotaciones hacen que la función se ejecuta cuando llega un evento.  Esta anotación se puede usar con tipos nativos de Java, POJO o valores que aceptan valores NULL mediante `Optional<T>`.

---

## <a name="attributes-and-annotations"></a>Atributos y anotaciones

# <a name="c"></a>[C#](#tab/csharp)

En las [bibliotecas de clases de C#](functions-dotnet-class-library.md), use el atributo [EventGridTrigger](https://github.com/Azure/azure-functions-eventgrid-extension/blob/master/src/EventGridExtension/TriggerBinding/EventGridTriggerAttribute.cs).

A continuación, se muestra un atributo `EventGridTrigger` en una signatura de método:

```csharp
[FunctionName("EventGridTest")]
public static void EventGridTest([EventGridTrigger] JObject eventGridEvent, ILogger log)
{
    ...
}
```

Para un ejemplo completo, consulte el ejemplo de C#.

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

El script de C# no admite atributos.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

JavaScript no admite atributos.

# <a name="python"></a>[Python](#tab/python)

Python no admite atributos.

# <a name="java"></a>[Java](#tab/java)

La anotación [EventGridTrigger](https://github.com/Azure/azure-functions-java-library/blob/master/src/main/java/com/microsoft/azure/functions/annotation/EventGridTrigger.java) permite configurar mediante declaración un enlace de Event Grid con valores de configuración. Para más información, consulte las secciones [Ejemplo](#example) y [Configuración](#configuration).

---

## <a name="configuration"></a>Configuración

En la siguiente tabla se explican las propiedades de configuración de enlace que se establecen en el archivo *function.json*. No hay parámetros de constructor ni propiedades que establecer en el atributo `EventGridTrigger`.

|Propiedad de function.json |Descripción|
|---------|---------|
| **type** | Requerida: se debe establecer en `eventGridTrigger`. |
| **direction** | Requerida: se debe establecer en `in`. |
| **name** | Requerido: el nombre de la variable que se utiliza en el código de función para el parámetro que recibe los datos del evento. |

## <a name="usage"></a>Uso

# <a name="c"></a>[C#](#tab/csharp)

En Azure Functions 1.x, puede usar los tipos de parámetro siguientes para el desencadenador de Event Grid:

* `JObject`
* `string`

En Azure Functions 2.x y versiones posteriores, también tiene la opción de usar el tipo de parámetro siguiente para el desencadenador de Event Grid:

* `Microsoft.Azure.EventGrid.Models.EventGridEvent`: define las propiedades de los campos comunes en todos los tipos de evento.

> [!NOTE]
> En Functions v1, si intenta enlazar a `Microsoft.Azure.WebJobs.Extensions.EventGrid.EventGridEvent`, el compilador mostrará un mensaje "en desuso" y le avisará de que use en su lugar `Microsoft.Azure.EventGrid.Models.EventGridEvent`. Para usar el tipo más nuevo, haga referencia al paquete NuGet [Microsoft.Azure.EventGrid](https://www.nuget.org/packages/Microsoft.Azure.EventGrid) y califique completamente el nombre del tipo `EventGridEvent` usando el prefijo `Microsoft.Azure.EventGrid.Models`.

# <a name="c-script"></a>[Script de C#](#tab/csharp-script)

En Azure Functions 1.x, puede usar los tipos de parámetro siguientes para el desencadenador de Event Grid:

* `JObject`
* `string`

En Azure Functions 2.x y versiones posteriores, también tiene la opción de usar el tipo de parámetro siguiente para el desencadenador de Event Grid:

* `Microsoft.Azure.EventGrid.Models.EventGridEvent`: define las propiedades de los campos comunes en todos los tipos de evento.

> [!NOTE]
> En Functions v1, si intenta enlazar a `Microsoft.Azure.WebJobs.Extensions.EventGrid.EventGridEvent`, el compilador mostrará un mensaje "en desuso" y le avisará de que use en su lugar `Microsoft.Azure.EventGrid.Models.EventGridEvent`. Para usar el tipo más nuevo, haga referencia al paquete NuGet [Microsoft.Azure.EventGrid](https://www.nuget.org/packages/Microsoft.Azure.EventGrid) y califique completamente el nombre del tipo `EventGridEvent` usando el prefijo `Microsoft.Azure.EventGrid.Models`. Para información sobre cómo hacer referencia a paquetes NuGet en una función de script de C#, consulte [Uso de paquetes NuGet](functions-reference-csharp.md#using-nuget-packages).

# <a name="javascript"></a>[JavaScript](#tab/javascript)

La instancia de Event Grid está disponible mediante el parámetro configurado en la propiedad `name` del archivo *function.json*.

# <a name="python"></a>[Python](#tab/python)

La instancia de Event Grid está disponible mediante el parámetro configurado en la propiedad `name` del archivo *function.json*, con el tipo `func.EventGridEvent`.

# <a name="java"></a>[Java](#tab/java)

La instancia del evento de Event Grid está disponible mediante el parámetro asociado al atributo `EventGridTrigger`, con el tipo `EventSchema`. Consulte el [ejemplo](#example) para más detalles.

---

## <a name="event-schema"></a>Esquema del evento

Los datos para un evento de Event Grid se reciben como un objeto JSON en el cuerpo de una solicitud HTTP. El código JSON es similar al del siguiente ejemplo:

```json
[{
  "topic": "/subscriptions/{subscriptionid}/resourceGroups/eg0122/providers/Microsoft.Storage/storageAccounts/egblobstore",
  "subject": "/blobServices/default/containers/{containername}/blobs/blobname.jpg",
  "eventType": "Microsoft.Storage.BlobCreated",
  "eventTime": "2018-01-23T17:02:19.6069787Z",
  "id": "{guid}",
  "data": {
    "api": "PutBlockList",
    "clientRequestId": "{guid}",
    "requestId": "{guid}",
    "eTag": "0x8D562831044DDD0",
    "contentType": "application/octet-stream",
    "contentLength": 2248,
    "blobType": "BlockBlob",
    "url": "https://egblobstore.blob.core.windows.net/{containername}/blobname.jpg",
    "sequencer": "000000000000272D000000000003D60F",
    "storageDiagnostics": {
      "batchId": "{guid}"
    }
  },
  "dataVersion": "",
  "metadataVersion": "1"
}]
```

El ejemplo que se muestra es una matriz de un elemento. Event Grid siempre envía una matriz y puede enviar más de un evento en la matriz. El entorno en tiempo de ejecución invoca la función una vez para cada elemento de matriz.

Las propiedades de nivel superior en los datos JSON del evento son las mismas entre todos los tipos de eventos, mientras que el contenido de la propiedad `data` es específico para cada tipo de evento. El ejemplo mostrado es para un evento de almacenamiento de blobs.

Para obtener una explicación de las propiedades comunes y específicas de eventos, consulte [Propiedades de evento](../event-grid/event-schema.md#event-properties) en la documentación de Event Grid.

El tipo `EventGridEvent` define solo las propiedades de nivel superior; la propiedad `Data` es un `JObject`.

## <a name="create-a-subscription"></a>una suscripción

Para empezar a recibir solicitudes HTTP de Event Grid, cree una suscripción a Event Grid que especifique la dirección URL de punto de conexión que invoca la función.

### <a name="azure-portal"></a>Azure portal

En el caso de las funciones que se desarrollen en Azure Portal con el desencadenador de Event Grid, seleccione **Integración** y, después, elija **Desencadenador de Event Grid** y seleccione **Crear suscripción de Event Grid**.

:::image type="content" source="media/functions-bindings-event-grid/portal-sub-create.png" alt-text="Conectar una suscripción de eventos nueva para desencadenar en el portal.":::

Al seleccionar este vínculo, el portal abre la página **Crear suscripciones de eventos** con el punto de conexión del desencadenador actual ya definido.

:::image type="content" source="media/functions-bindings-event-grid/endpoint-url.png" alt-text="Crear una suscripción de eventos con el punto de conexión de función ya definido" :::

Para obtener más información sobre cómo crear suscripciones mediante el Azure Portal, consulte [Creación de eventos personalizados: Azure Portal](../event-grid/custom-event-quickstart-portal.md) en la documentación de Event Grid.

### <a name="azure-cli"></a>Azure CLI

Para crear una suscripción mediante el uso de la [CLI de Azure](/cli/azure/get-started-with-azure-cli?view=azure-cli-latest), utilice el comando [az eventgrid event-subscription create](/cli/azure/eventgrid/event-subscription?view=azure-cli-latest#az-eventgrid-event-subscription-create).

El comando requiere la dirección URL del punto de conexión que invoca la función. En el siguiente ejemplo se muestra el patrón de la dirección URL específico de la versión:

#### <a name="version-2x-and-higher-runtime"></a>Entorno en tiempo de ejecución versión 2.x (y posteriores)

```http
https://{functionappname}.azurewebsites.net/runtime/webhooks/eventgrid?functionName={functionname}&code={systemkey}
```

#### <a name="version-1x-runtime"></a>Entorno en tiempo de ejecución versión 1.x

```http
https://{functionappname}.azurewebsites.net/admin/extensions/EventGridExtensionConfig?functionName={functionname}&code={systemkey}
```

La clave de sistema es una clave de autorización que debe incluirse en la dirección URL del punto de conexión para desencadenador de Event Grid. En la siguiente sección se explica cómo obtener la clave del sistema.

En este ejemplo se suscribe a una cuenta de almacenamiento de blobs (con un marcador de posición para la clave del sistema):

#### <a name="version-2x-and-higher-runtime"></a>Entorno en tiempo de ejecución versión 2.x (y posteriores)

```azurecli
az eventgrid resource event-subscription create -g myResourceGroup \
--provider-namespace Microsoft.Storage --resource-type storageAccounts \
--resource-name myblobstorage12345 --name myFuncSub  \
--included-event-types Microsoft.Storage.BlobCreated \
--subject-begins-with /blobServices/default/containers/images/blobs/ \
--endpoint https://mystoragetriggeredfunction.azurewebsites.net/runtime/webhooks/eventgrid?functionName=imageresizefunc&code=<key>
```

#### <a name="version-1x-runtime"></a>Entorno en tiempo de ejecución versión 1.x

```azurecli
az eventgrid resource event-subscription create -g myResourceGroup \
--provider-namespace Microsoft.Storage --resource-type storageAccounts \
--resource-name myblobstorage12345 --name myFuncSub  \
--included-event-types Microsoft.Storage.BlobCreated \
--subject-begins-with /blobServices/default/containers/images/blobs/ \
--endpoint https://mystoragetriggeredfunction.azurewebsites.net/admin/extensions/EventGridExtensionConfig?functionName=imageresizefunc&code=<key>
```

Para más información sobre cómo crear una suscripción, consulte [la guía de inicio rápido sobre el almacenamiento de blobs](../storage/blobs/storage-blob-event-quickstart.md#subscribe-to-your-storage-account) o cualquiera de las demás guías de inicio rápido de Event Grid.

### <a name="get-the-system-key"></a>Obtención de la clave del sistema

Puede obtener la clave del sistema mediante la siguiente API (HTTP GET):

#### <a name="version-2x-and-higher-runtime"></a>Entorno en tiempo de ejecución versión 2.x (y posteriores)

```
http://{functionappname}.azurewebsites.net/admin/host/systemkeys/eventgrid_extension?code={masterkey}
```

#### <a name="version-1x-runtime"></a>Entorno en tiempo de ejecución versión 1.x

```
http://{functionappname}.azurewebsites.net/admin/host/systemkeys/eventgridextensionconfig_extension?code={masterkey}
```

Es una API de administración, por lo que requiere la [clave maestra](functions-bindings-http-webhook-trigger.md#authorization-keys) de la aplicación de función. No confunda la clave del sistema (para invocar una función de desencadenador de Event Grid) con la clave maestra (para realizar tareas administrativas en la aplicación de función). Cuando se suscriba a un tema de Event Grid, asegúrese de usar la clave del sistema.

Este es un ejemplo de la respuesta que proporciona la clave del sistema:

```
{
  "name": "eventgridextensionconfig_extension",
  "value": "{the system key for the function}",
  "links": [
    {
      "rel": "self",
      "href": "{the URL for the function, without the system key}"
    }
  ]
}
```

Puede obtener la clave maestra de la aplicación de función en la pestaña **Configuración de Function App** del portal.

> [!IMPORTANT]
> La clave maestra proporciona acceso de administrador a la aplicación de función. Esta clave no debe compartirse con terceros ni distribuirse en aplicaciones cliente nativas.

Para más información, consulte [Claves de autorización](functions-bindings-http-webhook-trigger.md#authorization-keys) en el artículo de referencia del desencadenador HTTP.

También puede enviar un HTTP PUT para especificar el mismo valor de la clave.

## <a name="local-testing-with-viewer-web-app"></a>Pruebas locales con la aplicación web de visor

Para probar localmente un desencadenador de Event Grid, debe enviar las solicitudes HTTP de Event Grid desde su origen en la nube a la máquina local. Una manera de hacerlo es mediante la captura de solicitudes en línea y de reenviarlas manualmente a la máquina local:

1. [Cree una aplicación web de visor](#create-a-viewer-web-app) que capture los mensajes de eventos.
1. [Cree una suscripción a Event Grid](#create-an-event-grid-subscription) que envíe eventos a la aplicación de visor.
1. [Genere una solicitud](#generate-a-request) y copie el cuerpo de la solicitud desde la aplicación de visor.
1. [Envíe manualmente la solicitud](#manually-post-the-request) a la dirección URL de localhost de la función del desencadenador de Event Grid.

Cuando haya finalizado las pruebas, puede utilizar la misma suscripción para producción actualizando el punto de conexión. Utilice el comando [az eventgrid event-subscription update](/cli/azure/eventgrid/event-subscription?view=azure-cli-latest#az-eventgrid-event-subscription-update) de la CLI de Azure.

### <a name="create-a-viewer-web-app"></a>Creación de una aplicación web de visor

Para simplificar la captura de mensajes de evento, puede implementar una [aplicación web de ejemplo](https://github.com/Azure-Samples/azure-event-grid-viewer) que muestre los mensajes de los eventos. La solución implementada incluye un plan de App Service, una aplicación web de App Service y el código fuente desde GitHub.

Seleccione **Deploy to Azure** (Implementar en Azure) para implementar la solución en su suscripción. En Azure Portal, proporcione valores para los parámetros.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2Fazure-event-grid-viewer%2Fmaster%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png" alt="Button to Deploy to Azure." /></a>

La implementación puede tardar unos minutos en completarse. Después de que la implementación se haya realizado correctamente, puede ver la aplicación web para asegurarse de que se está ejecutando. En un explorador web, vaya a: `https://<your-site-name>.azurewebsites.net`

Verá el sitio, pero aún no se ha publicado en él ningún evento.

![Visualización del nuevo sitio](media/functions-bindings-event-grid/view-site.png)

### <a name="create-an-event-grid-subscription"></a>Creación de una suscripción de Event Grid

Cree una suscripción de Event Grid del tipo que desee probar y asígnele la dirección URL de su aplicación web como punto de conexión para la notificación de eventos. El punto de conexión de la aplicación web debe incluir el sufijo `/api/updates/`. Por lo tanto, dirección URL completa es `https://<your-site-name>.azurewebsites.net/api/updates`

Para obtener información sobre cómo crear suscripciones mediante Azure Portal, consulte [Creación de eventos personalizados: Azure Portal](../event-grid/custom-event-quickstart-portal.md) en la documentación de Event Grid.

### <a name="generate-a-request"></a>Generación de una solicitud

Desencadene un evento que generará tráfico HTTP a su punto de conexión de aplicación web.  Por ejemplo, si crea una suscripción de almacenamiento de blobs, cargue o elimine un blob. Cuando aparezca una solicitud en la aplicación web, copie el cuerpo de la solicitud.

La solicitud de validación de la suscripción se recibirá primero; ignore cualquier solicitud de validación y copie la solicitud del evento.

![Copia del cuerpo de la solicitud de la aplicación web](media/functions-bindings-event-grid/view-results.png)

### <a name="manually-post-the-request"></a>Envío manual de la solicitud

Ejecute la función de Event Grid localmente.

Use una herramienta como [Postman](https://www.getpostman.com/) o [curl](https://curl.haxx.se/docs/httpscripting.html) para crear una solicitud HTTP POST:

* Establezca un encabezado `Content-Type: application/json`.
* Establezca un encabezado `aeg-event-type: Notification`.
* Pegue los datos de RequestBin en el cuerpo de la solicitud.
* Envíe la dirección URL de la función del desencadenador de Event Grid.
  * Para 2.x y versiones posteriores, use el patrón siguiente:

    ```
    http://localhost:7071/runtime/webhooks/eventgrid?functionName={FUNCTION_NAME}
    ```

  * Para 1.x, use:

    ```
    http://localhost:7071/admin/extensions/EventGridExtensionConfig?functionName={FUNCTION_NAME}
    ```

El parámetro `functionName` debe ser el nombre especificado en el atributo `FunctionName`.

Las capturas de pantalla siguientes muestran los encabezados y el cuerpo de la solicitud en Postman:

![Encabezados en Postman](media/functions-bindings-event-grid/postman2.png)

![Cuerpo de la solicitud en Postman](media/functions-bindings-event-grid/postman.png)

La función de desencadenador de Event Grid ejecuta y muestra registros similares a los del ejemplo siguiente:

![Registros de la función de desencadenador de Event Grid de ejemplo](media/functions-bindings-event-grid/eg-output.png)

## <a name="next-steps"></a>Pasos siguientes

* [Envío de un evento de Event Grid](./functions-bindings-event-grid-output.md)
