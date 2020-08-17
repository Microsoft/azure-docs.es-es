---
title: Ingesta de telemetría desde IoT Hub
titleSuffix: Azure Digital Twins
description: Consulte cómo ingerir los mensajes de telemetría de dispositivos desde IoT Hub.
author: cschormann
ms.author: cschorm
ms.date: 3/17/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 7c73f007f85a963a09de4e05222082fd52f784c0
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87131572"
---
# <a name="ingest-iot-hub-telemetry-into-azure-digital-twins"></a>Ingesta de telemetría de IoT Hub en Azure Digital Twins

Azure Digital Twins se basa en los datos de los dispositivos IoT y otros orígenes. Un origen común de los datos del dispositivo que se va a usar en Azure Digital Twins es [IoT Hub](../iot-hub/about-iot-hub.md).

Durante la versión preliminar, el proceso para la ingesta de datos en Azure Digital Twins es configurar un recurso de proceso externo, como una [función de Azure](../azure-functions/functions-overview.md), que reciba los datos y use las [API de DigitalTwins](how-to-use-apis-sdks.md) para establecer propiedades o desencadenar eventos de telemetría en [Digital Twins](concepts-twins-graph.md) en consecuencia. 

Este documento de procedimientos le guía en el proceso de escritura de una función de Azure que pueda ingerir la telemetría de IoT Hub.

## <a name="example-telemetry-scenario"></a>Escenario de telemetría de ejemplo

En este procedimiento se explica cómo enviar mensajes desde IoT Hub a Azure Digital Twins, mediante una función de Azure. Hay muchas configuraciones posibles y estrategias de coincidencia que puede usar para ello, pero el ejemplo de este artículo contiene las siguientes partes:
* Un dispositivo de termómetro en IoT Hub, con un identificador de dispositivo conocido.
* Un gemelo digital para representar el dispositivo, con un identificador coincidente
* Un gemelo digital que representa una habitación

> [!NOTE]
> En este ejemplo se usa una coincidencia de identificador sencillo entre el identificador de dispositivo y el identificador de gemelo digital correspondiente, pero es posible proporcionar asignaciones más sofisticadas del dispositivo a su gemelo (por ejemplo, con una tabla de asignación).

Cada vez que el dispositivo de termómetro envíe un evento de telemetría de temperatura, se debe actualizar la propiedad *temperature* (temperatura) del gemelo *Room* (Habitación). Para que esto suceda, se asignará un evento de telemetría de un dispositivo a un configurador de propiedad en el gemelo digital. Usará la información de la topología del [grafo gemelo](concepts-twins-graph.md) para buscar el gemelo *Room*; a continuación, puede establecer la propiedad del gemelo. En otros escenarios, es posible que los usuarios prefieran establecer una propiedad en el gemelo coincidente (en este ejemplo, el que tiene el identificador *123*). Azure Digital Twins ofrece una gran flexibilidad para decidir cómo se asignan los datos de telemetría a los gemelos. 

Este escenario se describe en un diagrama a continuación:

:::image type="content" source="media/how-to-ingest-iot-hub-data/events.png" alt-text="Un dispositivo de IoT Hub envía la telemetría Temperature con IoT Hub, Event Grid o temas del sistema a una función de Azure, que actualiza una propiedad de temperatura de los gemelos en Azure Digital Twins." border="false":::

## <a name="prerequisites"></a>Requisitos previos

Antes de continuar con este ejemplo, debe completar los siguientes requisitos previos.
1. Cree un Centro de IoT. Consulte la sección *Creación de una instancia de IoT Hub* de [este inicio rápido de IoT Hub](../iot-hub/quickstart-send-telemetry-cli.md) para obtener instrucciones.
2. Cree al menos una función de Azure para procesar los eventos de IoT Hub. Consulte [*Procedimientos: Conexión de aplicaciones de Azure Functions para el procesamiento de datos*](how-to-create-azure-function.md) para crear una función básica de Azure que pueda conectarse a Azure Digital Twins y llamar a las funciones de la API de Azure Digital Twins. El resto de este procedimiento se creará en esta función.
3. Configure un destino de evento para los datos del centro de conectividad. En [Azure Portal](https://portal.azure.com/), vaya a su instancia de IoT Hub. En *Eventos*, cree una suscripción para su función de Azure. 

    :::image type="content" source="media/how-to-ingest-iot-hub-data/add-event-subscription.png" alt-text="Azure Portal: Incorporación de una suscripción a un evento":::

4. En la página *Crear suscripción de eventos*, rellene los campos como se indica a continuación:
   * En *DETALLES DE SUSCRIPCIONES DE EVENTOS*, asigne a la suscripción el nombre que desee.
   * En *TIPOS DE EVENTO*, seleccione *Telemetría del dispositivo* como el tipo de evento por el que filtrar.
      - Si lo desea, agregue filtros a otros tipos de eventos.
   * En *DETALLES DE PUNTO DE CONEXIÓN*, seleccione la función de Azure como un punto de conexión.

## <a name="create-an-azure-function-in-visual-studio"></a>Creación de una función de Azure en Visual Studio

En esta sección se usan los mismos pasos de inicio de Visual Studio y el esqueleto de la función de Azure en [*Procedimientos: Configuración de una función de Azure para procesar datos*](how-to-create-azure-function.md). El esqueleto controla la autenticación y crea un cliente de servicio, listo para procesar los datos y llamar a las API de Azure Digital Twins como respuesta. 

El corazón de la función de esqueleto es el siguiente:

```csharp
namespace FunctionSample
{
    public static class FooFunction
    {
        const string adtAppId = "https://digitaltwins.azure.net";
        private static string adtInstanceUrl = Environment.GetEnvironmentVariable("ADT_SERVICE_URL");
        private static HttpClient httpClient = new HttpClient();

        [FunctionName("Foo")]
        public static async Task Run([EventGridTrigger]EventGridEvent eventGridEvent, ILogger log)
        {
            DigitalTwinsClient client = null;
            try
            {
                ManagedIdentityCredential cred = new ManagedIdentityCredential(adtAppId);
                DigitalTwinsClientOptions opts = 
                    new DigitalTwinsClientOptions { Transport = new HttpClientTransport(httpClient) });
                client = new DigitalTwinsClient(new Uri(adtInstanceUrl), cred, opts);
                                                
                log.LogInformation($"ADT service client connection created.");
            }
            catch (Exception e)
            {
                log.LogError($"ADT service client connection failed. " + e.ToString());
                return;
            }
            log.LogInformation(eventGridEvent.Data.ToString());
        }
    }
}
```

En los pasos siguientes, agregará código específico para procesar los eventos de telemetría de IoT desde IoT Hub.  

## <a name="add-telemetry-processing"></a>Procesamiento de datos de telemetría

Los eventos de telemetría tienen el formato de mensajes del dispositivo. El primer paso para agregar el código de procesamiento de telemetría es extraer la parte pertinente de este mensaje del dispositivo del evento Event Grid. 

Los distintos dispositivos pueden estructurar sus mensajes de forma diferente, por lo que el código de este paso depende del dispositivo conectado. 

En el código siguiente se muestra un ejemplo de un dispositivo simple que envía telemetría como JSON. El ejemplo extrae el identificador de dispositivo del dispositivo que envió el mensaje, así como el valor de temperatura.

```csharp
JObject job = eventGridEvent.Data as JObject;
string devid = (string)job["systemProperties"].ToObject<JObject>().Property("IoT-hub-connection-device-ID").Value;
double temp = (double)job["body"].ToObject<JObject>().Property("temperature").Value;
```

Recuerde que el propósito de este ejercicio es actualizar la temperatura de una *habitación* dentro del grafo gemelo. Esto significa que el destino del mensaje no es el gemelo digital que está asociado a este dispositivo, sino el gemelo *Room* (habitación) que es su elemento primario. Puede buscar el gemelo principal con el valor de identificador de dispositivo que extrajo del mensaje de telemetría mediante el código anterior.

Para ello, use las API de Azure Digital Twins para acceder a las relaciones entrantes con el gemelo que representa el dispositivo (que, en este caso, tiene el mismo identificador que el dispositivo). En la relación de entrada, puede encontrar el identificador del elemento primario con el siguiente fragmento de código.

Muestra cómo recuperar las relaciones entrantes de un gemelo:

```csharp
AsyncPageable<IncomingRelationship> res = client.GetIncomingRelationshipsAsync(twin_id);
await foreach (IncomingRelationship irel in res)
{
    Log.Ok($"Relationship: {irel.RelationshipName} from {irel.SourceId} | {irel.RelationshipId}");
}
```

El elemento primario del gemelo está en la propiedad *SourceId* de la relación.

Es bastante común para un modelo de un gemelo que representa un dispositivo tener solo una única relación entrante. En este caso, puede seleccionar la primera (y única) relación devuelta. Si los modelos permiten varios tipos de relaciones con este gemelo, puede que tenga que especificar más para elegir entre varias relaciones entrantes. Una forma habitual de hacerlo es seleccionar la relación por `RelationshipName`. 

Una vez que tenga el identificador del gemelo primario que represente la *habitación*, puede "revisar" (realizar actualizaciones) dicho gemelo. Para ello, ejecute el siguiente código:

```csharp
UpdateOperationsUtility uou = new UpdateOperationsUtility();
uou.AppendAddOp("/Temperature", temp);
try
{
    await client.UpdateDigitalTwinAsync(twin_id, uou.Serialize());
    Log.Ok($"Twin '{twin_id}' updated successfully!");
}
...
```

### <a name="full-azure-function-code"></a>Código de función completo de Azure

Con el código de los ejemplos anteriores, esta es toda la función de Azure en contexto:

```csharp
[FunctionName("ProcessHubToDTEvents")]
public async void Run([EventGridTrigger]EventGridEvent eventGridEvent, ILogger log)
{
    // After this is deployed, in order for this function to be authorized on Azure Digital Twins APIs,
    // you'll need to turn the Managed Identity Status to "On", 
    // grab the Object ID of the function, and assign the "Azure Digital Twins Owner (Preview)" role to this function identity.

    DigitalTwinsClient client = null;
    //log.LogInformation(eventGridEvent.Data.ToString());
    // Authenticate on Azure Digital Twins APIs
    try
    {
        
        ManagedIdentityCredential cred = new ManagedIdentityCredential(adtAppId);
        client = new DigitalTwinsClient(new Uri(adtInstanceUrl), cred, new DigitalTwinsClientOptions { Transport = new HttpClientTransport(httpClient) });
        log.LogInformation($"ADT service client connection created.");
    }
    catch (Exception e)
    {
        log.LogError($"ADT service client connection failed. " + e.ToString());
        return;
    }

    if (client != null)
    {
        try
        {
            if (eventGridEvent != null && eventGridEvent.Data != null)
            {
                #region Open this region for message format information
                // Telemetry message format
                //{
                //  "properties": { },
                //  "systemProperties": 
                // {
                //    "iothub-connection-device-id": "thermostat1",
                //    "iothub-connection-auth-method": "{\"scope\":\"device\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}",
                //    "iothub-connection-auth-generation-id": "637199981642612179",
                //    "iothub-enqueuedtime": "2020-03-18T18:35:08.269Z",
                //    "iothub-message-source": "Telemetry"
                //  },
                //  "body": "eyJUZW1wZXJhdHVyZSI6NzAuOTI3MjM0MDg3MTA1NDg5fQ=="
                //}
                #endregion

                // Reading deviceId from message headers
                log.LogInformation(eventGridEvent.Data.ToString());
                JObject job = (JObject)JsonConvert.DeserializeObject(eventGridEvent.Data.ToString());
                string deviceId = (string)job["systemProperties"]["iothub-connection-device-id"];
                log.LogInformation($"Found device: {deviceId}");

                // Extracting temperature from device telemetry
                byte[] body = System.Convert.FromBase64String(job["body"].ToString());
                var value = System.Text.ASCIIEncoding.ASCII.GetString(body);
                var bodyProperty = (JObject)JsonConvert.DeserializeObject(value);
                var temperature = bodyProperty["Temperature"];
                log.LogInformation($"Device Temperature is:{temperature}");

                // Update device Temperature property
                await AdtUtilities.UpdateTwinProperty(client, deviceId, "/Temperature", temperature, log);

                // Find parent using incoming relationships
                string parentId = await AdtUtilities.FindParent(client, deviceId, "contains", log);
                if (parentId != null)
                {
                    await AdtUtilities.UpdateTwinProperty(client, parentId, "/Temperature", temperature, log);
                }

            }
        }
        catch (Exception e)
        {
            log.LogError($"Error in ingest function: {e.Message}");
        }
    }
}
```

La función de utilidad para buscar las relaciones entrantes:
```csharp
public static async Task<string> FindParent(DigitalTwinsClient client, string child, string relname, ILogger log)
{
    // Find parent using incoming relationships
    try
    {
        AsyncPageable<IncomingRelationship> rels = client.GetIncomingRelationshipsAsync(child);

        await foreach (IncomingRelationship ie in rels)
        {
            if (ie.RelationshipName == relname)
                return (ie.SourceId);
        }
    }
    catch (RequestFailedException exc)
    {
        log.LogInformation($"*** Error in retrieving parent:{exc.Status}:{exc.Message}");
    }
    return null;
}
```

Y la función de utilidad para aplicar revisiones al gemelo:
```csharp
public static async Task UpdateTwinProperty(DigitalTwinsClient client, string twinId, string propertyPath, object value, ILogger log)
{
    // If the twin does not exist, this will log an error
    try
    {
        // Update twin property
        UpdateOperationsUtility uou = new UpdateOperationsUtility();
        uou.AppendAddOp(propertyPath, value);
        await client.UpdateDigitalTwinAsync(twinId, uou.Serialize());
    }
    catch (RequestFailedException exc)
    {
        log.LogInformation($"*** Error:{exc.Status}/{exc.Message}");
    }
}
```

Ahora tiene una función de Azure que está equipada para leer e interpretar los datos del escenario procedentes de IoT Hub.

## <a name="debug-azure-function-apps-locally"></a>Depuración de aplicaciones de funciones de Azure de forma local.

Es posible depurar funciones de Azure con un desencadenador de Event Grid de forma local. Para más información, consulte [*Depuración de un desencadenador de Event Grid de forma local*](../azure-functions/functions-debug-event-grid-trigger-local.md).

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre la entrada y salida de datos con Azure Digital Twins:
* [*Conceptos: Integración con otros servicios*](concepts-integration.md)
