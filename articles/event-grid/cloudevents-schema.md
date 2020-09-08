---
title: Uso de Azure Event Grid con eventos del esquema CloudEvents
description: Se describe cómo utilizar el esquema CloudEvents para eventos de Azure Event Grid. El servicio admite eventos en la implementación JSON de eventos en la nube.
ms.topic: conceptual
ms.date: 07/07/2020
ms.custom: devx-track-javascript, devx-track-csharp
ms.openlocfilehash: 9c58811cc13a9eb37104abbe9901cb94dda25a9e
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89008784"
---
# <a name="use-cloudevents-v10-schema-with-event-grid"></a>Uso del esquema CloudEvents v1.0 con Event Grid
Además de su [esquema de eventos predeterminado](event-schema.md), Azure Event Grid admite de forma nativa eventos de la [implementación de JSON de CloudEvents v1.0](https://github.com/cloudevents/spec/blob/v1.0/json-format.md) y el [enlace del protocolo HTTP](https://github.com/cloudevents/spec/blob/v1.0/http-protocol-binding.md). [CloudEvents](https://cloudevents.io/) es una [especificación abierta](https://github.com/cloudevents/spec/blob/v1.0/spec.md) para la descripción de datos de eventos.

CloudEvents simplifica la interoperabilidad al proporcionar un esquema de eventos común para la publicación y el consumo de eventos basados en la nube. Este esquema permite herramientas uniformes, formas estándar de enrutar y administrar eventos y formas universales de deserializar el esquema de eventos externo. Con un esquema común, puede integrar más fácilmente el trabajo entre plataformas.

En la compilación de CloudEvents participan varios [colaboradores](https://github.com/cloudevents/spec/blob/master/community/contributors.md), entre ellos Microsoft, mediante [Cloud Native Compute Foundation](https://www.cncf.io/). Actualmente está disponible en versión 1.0.

En este artículo se describe cómo usar el esquema CloudEvents con Event Grid.

[!INCLUDE [requires-azurerm](../../includes/requires-azurerm.md)]

## <a name="install-preview-feature"></a>Instalación de la característica en vista previa

[!INCLUDE [event-grid-preview-feature-note.md](../../includes/event-grid-preview-feature-note.md)]

## <a name="cloudevent-schema"></a>Esquema CloudEvent

Este es un ejemplo de un evento de Azure Blob Storage en formato de CloudEvents:

``` JSON
{
    "specversion": "1.0",
    "type": "Microsoft.Storage.BlobCreated",  
    "source": "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Storage/storageAccounts/{storage-account}",
    "id": "9aeb0fdf-c01e-0131-0922-9eb54906e209",
    "time": "2019-11-18T15:13:39.4589254Z",
    "subject": "blobServices/default/containers/{storage-container}/blobs/{new-file}",
    "dataschema": "#",
    "data": {
        "api": "PutBlockList",
        "clientRequestId": "4c5dd7fb-2c48-4a27-bb30-5361b5de920a",
        "requestId": "9aeb0fdf-c01e-0131-0922-9eb549000000",
        "eTag": "0x8D76C39E4407333",
        "contentType": "image/png",
        "contentLength": 30699,
        "blobType": "BlockBlob",
        "url": "https://gridtesting.blob.core.windows.net/testcontainer/{new-file}",
        "sequencer": "000000000000000000000000000099240000000000c41c18",
        "storageDiagnostics": {
            "batchId": "681fe319-3006-00a8-0022-9e7cde000000"
        }
    }
}
```

Una descripción detallada de los campos disponibles, sus tipos y definiciones en CloudEvents v1.0 se encuentra [disponible aquí](https://github.com/cloudevents/spec/blob/v1.0/spec.md#required-attributes).

Los valores de encabezados de los eventos proporcionados en el esquema de CloudEvents y el esquema de Event Grid son los mismos excepto para `content-type`. En el esquema de CloudEvents, ese valor de encabezado es `"content-type":"application/cloudevents+json; charset=utf-8"`. En el esquema de Event Grid, ese valor de encabezado es `"content-type":"application/json; charset=utf-8"`.

## <a name="configure-event-grid-for-cloudevents"></a>Configuración de Event Grid para CloudEvents

Puede usar Event Grid con eventos de entrada y salida de esquema CloudEvents. CloudEvents se puede usar con eventos del sistema, como eventos de Blob Storage y eventos de IoT Hub, y con eventos personalizados. También pueden transformar esos eventos en la red de un formato a otro.


| Esquema de entrada       | Esquema de salida
|--------------------|---------------------
| Formato de CloudEvents | Formato de CloudEvents
| Formato de Event Grid  | Formato de CloudEvents
| Formato de Event Grid  | Formato de Event Grid

Con todos los esquemas de eventos, Event Grid requiere validación al publicar en un tema de Event Grid y al crear una suscripción de eventos. Para más información, vea [Event Grid security and authentication](security-authentication.md) (Seguridad y autenticación de Event Grid).

### <a name="input-schema"></a>Esquema de entrada

El esquema de entrada de un tema personalizado se establece al crear este.

Para la CLI de Azure, utilice:

```azurecli-interactive
# If you have not already installed the extension, do it now.
# This extension is required for preview features.
az extension add --name eventgrid

az eventgrid topic create \
  --name <topic_name> \
  -l westcentralus \
  -g gridResourceGroup \
  --input-schema cloudeventschemav1_0
```

Para PowerShell, use:

```azurepowershell-interactive
# If you have not already installed the module, do it now.
# This module is required for preview features.
Install-Module -Name AzureRM.EventGrid -AllowPrerelease -Force -Repository PSGallery

New-AzureRmEventGridTopic `
  -ResourceGroupName gridResourceGroup `
  -Location westcentralus `
  -Name <topic_name> `
  -InputSchema CloudEventSchemaV1_0
```

### <a name="output-schema"></a>Esquema de salida

El esquema de salida se establece al crear la suscripción de eventos.

Para la CLI de Azure, utilice:

```azurecli-interactive
topicID=$(az eventgrid topic show --name <topic-name> -g gridResourceGroup --query id --output tsv)

az eventgrid event-subscription create \
  --name <event_subscription_name> \
  --source-resource-id $topicID \
  --endpoint <endpoint_URL> \
  --event-delivery-schema cloudeventschemav1_0
```

Para PowerShell, use:
```azurepowershell-interactive
$topicid = (Get-AzureRmEventGridTopic -ResourceGroupName gridResourceGroup -Name <topic-name>).Id

New-AzureRmEventGridSubscription `
  -ResourceId $topicid `
  -EventSubscriptionName <event_subscription_name> `
  -Endpoint <endpoint_URL> `
  -DeliverySchema CloudEventSchemaV1_0
```

 Actualmente, no puede usar un desencadenador de Event Grid para una aplicación de Azure Functions cuando el evento se entrega en el esquema de CloudEvents. Use un desencadenador HTTP. Para obtener ejemplos de la implementación de un desencadenador HTTP que recibe eventos en el esquema de CloudEvents, consulte el [uso de CloudEvents con Azure Functions](#azure-functions).

 ## <a name="endpoint-validation-with-cloudevents-v10"></a>Validación de punto de conexión con CloudEvents v1.0

Si ya está familiarizado con Event Grid, es posible que conozca el protocolo de enlace de validación de punto de conexión de Event Grid para evitar el uso inapropiado. CloudEvents v1.0 implementa su propia [semántica de protección contra abusos](webhook-event-delivery.md) mediante el método HTTP OPTIONS. Puede leer más sobre este tema [aquí](https://github.com/cloudevents/spec/blob/v1.0/http-webhook.md#4-abuse-protection). Al usar el esquema de CloudEvents para la salida, Event Grid lo hace con la protección contra abusos de CloudEvents v1.0 en lugar del mecanismo de eventos de validación de Event Grid.

<a name="azure-functions"></a>

## <a name="use-with-azure-functions"></a>Uso con Azure Functions

El [enlace de Event Grid para Azure Functions](../azure-functions/functions-bindings-event-grid.md) no admite de forma nativa CloudEvents, por lo que las funciones desencadenadas por HTTP se usan para leer los mensajes de CloudEvents. Cuando se usa un desencadenador HTTP para leer CloudEvents, tiene que escribir código para lo que el desencadenador de Event Grid hace automáticamente:

* Envía una respuesta de validación a una [solicitud de validación de suscripción](../event-grid/webhook-event-delivery.md).
* Invoca la función una vez por elemento de la matriz de eventos contenida en el cuerpo de la solicitud.

Para más información acerca de la dirección URL que se debe utilizar para invocar la función localmente o cuando se ejecuta en Azure, consulte la [documentación de referencia de enlaces del desencadenador HTTP](../azure-functions/functions-bindings-http-webhook.md).

El siguiente ejemplo de código C# para un desencadenador HTTP simula el comportamiento del desencadenador de Event Grid.  Utilice este ejemplo para los eventos entregados en el esquema de CloudEvents.

```csharp
[FunctionName("HttpTrigger")]
public static async Task<HttpResponseMessage> Run([HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", "options", Route = null)]HttpRequestMessage req, ILogger log)
{
    log.LogInformation("C# HTTP trigger function processed a request.");
    if (req.Method == "OPTIONS")
    {
        // If the request is for subscription validation, send back the validation code
        
        var response = req.CreateResponse(HttpStatusCode.OK);
        response.Add("Webhook-Allowed-Origin", "eventgrid.azure.net");

        return response;
    }

    var requestmessage = await req.Content.ReadAsStringAsync();
    var message = JToken.Parse(requestmessage);

    // The request is not for subscription validation, so it's for an event.
    // CloudEvents schema delivers one event at a time.
    log.LogInformation($"Source: {message["source"]}");
    log.LogInformation($"Time: {message["eventTime"]}");
    log.LogInformation($"Event data: {message["data"].ToString()}");

    return req.CreateResponse(HttpStatusCode.OK);
}
```

El siguiente ejemplo de código JavaScript para un desencadenador HTTP simula el comportamiento del desencadenador de Event Grid. Utilice este ejemplo para los eventos entregados en el esquema de CloudEvents.

```javascript
module.exports = function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');
    
    if (req.method == "OPTIONS") {
        // If the request is for subscription validation, send back the validation code
        
        context.log('Validate request received');
        context.res = { status: 200 };
        context.res.headers.append('Webhook-Allowed-Origin', 'eventgrid.azure.net');
    }
    else
    {
        var message = req.body;
        
        // The request is not for subscription validation, so it's for an event.
        // CloudEvents schema delivers one event at a time.
        var event = JSON.parse(message);
        context.log('Source: ' + event.source);
        context.log('Time: ' + event.eventTime);
        context.log('Data: ' + JSON.stringify(event.data));
    }
 
    context.done();
};
```

## <a name="next-steps"></a>Pasos siguientes

* Para información sobre la supervisión de las entregas de eventos, consulte [Supervisar la entrega de mensajes de Event Grid](monitor-event-delivery.md).
* Le animamos a que pruebe CloudEvents, nos deje su opinión y [contribuya](https://github.com/cloudevents/spec/blob/master/community/CONTRIBUTING.md) a dicho esquema.
* Para más información acerca de la creación de una suscripción de Azure Event Grid, consulte [Esquema de suscripción de Event Grid](subscription-creation-schema.md).
