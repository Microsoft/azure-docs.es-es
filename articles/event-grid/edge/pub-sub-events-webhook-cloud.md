---
title: 'Publicación de eventos y suscripción a ellos en la nube: Azure Event Grid en IoT Edge | Microsoft Docs'
description: Publicación de eventos y suscripción a ellos en la nube mediante webhook con Event Grid en IoT Edge
author: VidyaKukke
manager: rajarv
ms.author: vkukke
ms.reviewer: spelluru
ms.date: 07/08/2020
ms.topic: article
ms.custom: devx-track-csharp
ms.openlocfilehash: 12bcb54f4bfdf17209324febeba380ff7789fc0f
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88998992"
---
# <a name="tutorial-publish-subscribe-to-events-in-cloud"></a>Tutorial: Publicación de eventos y suscripción a ellos en la nube

Este artículo le guía por todos los pasos necesarios para publicar eventos y suscribirse a ellos mediante Event Grid en IoT Edge. En este tutorial se usa Azure Functions como controlador de eventos. Para obtener más tipos de destino, consulte [Controladores de eventos](event-handlers.md).

Antes de continuar, consulte [Conceptos de Event Grid](concepts.md) para entender qué es un tema y una suscripción de Event Grid.

## <a name="prerequisites"></a>Prerrequisitos 
Para realizar este tutorial, necesitará lo siguiente:

* Una **suscripción a Azure**: cree una [cuenta gratuita](https://azure.microsoft.com/free) si aún no tiene una. 
* **Dispositivo Azure IoT Hub e IoT Edge**: siga los pasos del inicio rápido para dispositivos [Linux](../../iot-edge/quickstart-linux.md) o [Windows](../../iot-edge/quickstart.md) si aún no tiene uno.

[!INCLUDE [event-grid-deploy-iot-edge](../../../includes/event-grid-deploy-iot-edge.md)]

## <a name="create-an-azure-function-in-the-azure-portal"></a>Cree una función de Azure en Azure Portal.

Siga los pasos descritos en el [tutorial](../../azure-functions/functions-create-first-azure-function.md) para crear una función de Azure. 

Reemplace el fragmento de código por el código siguiente:

```csharp
#r "Newtonsoft.Json"

using System.Net;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Primitives;
using Newtonsoft.Json;

public static async Task<IActionResult> Run(HttpRequest req, ILogger log)
{
    log.LogInformation("C# HTTP trigger function processed a request.");

    string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
    dynamic data = JsonConvert.DeserializeObject(requestBody);

    log.LogInformation($"C# HTTP trigger received {data}.");
    return data != null
        ? (ActionResult)new OkResult()
        : new BadRequestObjectResult("Please pass in the request body");
}
```

En la nueva función, seleccione **Obtener la dirección URL de la función** en la parte superior derecha, seleccione el valor predeterminado (**Clave de función**) y, después, seleccione **Copiar**. Usará el valor de dirección URL de la función más adelante en el tutorial.

> [!NOTE]
> Consulte la documentación de [Azure Functions](../../azure-functions/functions-overview.md) para ver más ejemplos y tutoriales sobre cómo reaccionar a los eventos mediante desencadenadores de eventos EventGrid.

## <a name="create-a-topic"></a>de un tema

Como publicador de un evento, debe crear un tema de Event Grid. El tema hace referencia a un punto de conexión al que los publicadores pueden enviar eventos.

1. Cree el archivo topic2.json con el siguiente contenido. Para más información sobre la carga, consulte nuestra [documentación de API](api.md).

    ```json
         {
          "name": "sampleTopic2",
          "properties": {
            "inputschema": "eventGridSchema"
          }
        }
    ```
1. Para crear el tema, ejecute el siguiente comando. Se devolverá el código de estado HTTP "200 - Correcto".

    ```sh
    curl -k -H "Content-Type: application/json" -X PUT -g -d @topic2.json https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic2?api-version=2019-01-01-preview
    ```
1. Use el siguiente comando para comprobar que se creó el tema correctamente. Se devolverá el código de estado HTTP "200 - Correcto".

    ```sh
    curl -k -H "Content-Type: application/json" -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic2?api-version=2019-01-01-preview
    ```

   Salida del ejemplo:

   ```json
        [
          {
            "id": "/iotHubs/eg-iot-edge-hub/devices/eg-edge-device/modules/eventgridmodule/topics/sampleTopic2",
            "name": "sampleTopic2",
            "type": "Microsoft.EventGrid/topics",
            "properties": {
              "endpoint": "https://<edge-vm-ip>:4438/topics/sampleTopic2/events?api-version=2019-01-01-preview",
              "inputSchema": "EventGridSchema"
            }
          }
        ]
   ```

## <a name="create-an-event-subscription"></a>Creación de una suscripción a evento

Los suscriptores pueden registrarse en eventos publicados en un tema. Para recibir cualquier evento, los suscriptores deberán crear una suscripción a Event Grid en un tema de interés.

[!INCLUDE [event-grid-deploy-iot-edge](../../../includes/event-grid-edge-persist-event-subscriptions.md)]

1. Cree el archivo subscription2.json con el siguiente contenido. Para más información sobre la carga, consulte nuestra [documentación de API](api.md).

    ```json
        {
          "properties": {
            "destination": {
              "endpointType": "WebHook",
              "properties": {
                "endpointUrl": "<your-az-func-cloud-url>"
              }
            }
          }
        }
    ```

   >[!NOTE]
   > La propiedad **endpointType** especifica que el suscriptor es un webhook.  La propiedad **endpointUrl** especifica la dirección URL en la que el suscriptor escucha los eventos. Esta dirección URL corresponde al ejemplo de Azure Functions que configuró anteriormente.
2. Ejecute el siguiente comando para crear la suscripción. Se devolverá el código de estado HTTP "200 - Correcto".

    ```sh
    curl -k -H "Content-Type: application/json" -X PUT -g -d @subscription2.json https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic2/eventSubscriptions/sampleSubscription2?api-version=2019-01-01-preview
    ```
3. Ejecute el siguiente comando para comprobar que la suscripción se ha creado correctamente. Se devolverá el código de estado HTTP "200 - Correcto".

    ```sh
    curl -k -H "Content-Type: application/json" -X GET -g https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic2/eventSubscriptions/sampleSubscription2?api-version=2019-01-01-preview
    ```

    Salida del ejemplo:

   ```json
        {
          "id": "/iotHubs/eg-iot-edge-hub/devices/eg-edge-device/modules/eventgridmodule/topics/sampleTopic2/eventSubscriptions/sampleSubscription2",
          "type": "Microsoft.EventGrid/eventSubscriptions",
          "name": "sampleSubscription2",
          "properties": {
            "Topic": "sampleTopic2",
            "destination": {
              "endpointType": "WebHook",
              "properties": {
                "endpointUrl": "<your-az-func-cloud-url>"
              }
            }
          }
        }
    ```

## <a name="publish-an-event"></a>Publicación de un evento

1. Cree el archivo event2.json con el siguiente contenido. Para más información sobre la carga, consulte nuestra [documentación de API](api.md).

    ```json
        [
          {
            "id": "eventId-func-1",
            "eventType": "recordInserted",
            "subject": "myapp/vehicles/motorcycles",
            "eventTime": "2019-07-28T21:03:07+00:00",
            "dataVersion": "1.0",
            "data": {
              "make": "Ducati",
              "model": "Monster"
            }
          }
        ]
    ```
1. Para publicar el evento, ejecute el siguiente comando:

    ```sh
    curl -k -H "Content-Type: application/json" -X POST -g -d @event2.json https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic2/events?api-version=2019-01-01-preview
    ```

## <a name="verify-event-delivery"></a>Comprobación de la entrega del evento

Puede ver el evento entregado en Azure Portal en la opción **Monitor** de la función.

## <a name="cleanup-resources"></a>Limpieza de recursos

* Ejecute el siguiente comando para eliminar el tema y todas sus suscripciones

    ```sh
    curl -k -H "Content-Type: application/json" -X DELETE https://<your-edge-device-public-ip-here>:4438/topics/sampleTopic2?api-version=2019-01-01-preview
    ```

* Elimine la función de Azure creada en Azure Portal.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado un tema y una suscripción de Event Grid y ha publicado eventos. Ahora que conoce los pasos básicos, consulte los siguientes artículos:

* Para solucionar problemas relacionados con el uso de Azure Event Grid en IoT Edge, consulte la [Guía de solución de problemas](troubleshoot.md).
* Cree o actualice una suscripción con [filtros](advanced-filtering.md).
* Configure la persistencia del módulo de Event Grid en [Linux](persist-state-linux.md) o [Windows](persist-state-windows.md)
* Siga la [documentación](configure-client-auth.md) para configurar la autenticación de cliente
* Reenvíe eventos a Azure Event Grid en la nube siguiendo este [tutorial](forward-events-event-grid-cloud.md)
* [Supervisión de temas y suscripciones en el perímetro](monitor-topics-subscriptions.md)
