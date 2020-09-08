---
title: Ingesta de telemetría desde IoT Hub
titleSuffix: Azure Digital Twins
description: Consulte cómo ingerir los mensajes de telemetría de dispositivos desde IoT Hub.
author: alexkarcher-msft
ms.author: alkarche
ms.date: 8/11/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 8d720d77773e506a13f176723ab4583613f1e625
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89291762"
---
# <a name="ingest-iot-hub-telemetry-into-azure-digital-twins"></a>Ingesta de telemetría de IoT Hub en Azure Digital Twins

Azure Digital Twins se basa en los datos de los dispositivos IoT y otros orígenes. Un origen común de los datos del dispositivo que se va a usar en Azure Digital Twins es [IoT Hub](../iot-hub/about-iot-hub.md).

El proceso para la ingesta de datos en Azure Digital Twins consiste en configurar un recurso de proceso externo, como una [función de Azure](../azure-functions/functions-overview.md), que reciba los datos y use las [API de DigitalTwins](how-to-use-apis-sdks.md) para establecer propiedades o desencadenar eventos de telemetría en [Digital Twins](concepts-twins-graph.md) en consecuencia. 

Este documento de procedimientos le guía en el proceso de escritura de una función de Azure que pueda ingerir la telemetría de IoT Hub.

## <a name="prerequisites"></a>Requisitos previos

Antes de continuar con este ejemplo, debe configurar los siguientes recursos como requisito previo:
* **Una instancia de IoT Hub**. Para instrucciones, consulte la sección *Creación de una instancia de IoT Hub* de [este inicio rápido de IoT Hub](../iot-hub/quickstart-send-telemetry-cli.md).
* **Una función de Azure** con los permisos correctos para llamar a la instancia de Digital Twins. Para instrucciones, consulte [*Procedimiento: Configuración de una función de Azure para procesar datos*](how-to-create-azure-function.md). 
* **Una instancia de Azure Digital Twins** que recibirá la telemetría del dispositivo. Para instrucciones, consulte [*Procedimiento: Configuración de una instancia de Azure Digital Twins y autenticación*](./how-to-set-up-instance-portal.md).

### <a name="example-telemetry-scenario"></a>Escenario de telemetría de ejemplo

En este procedimiento se explica cómo enviar mensajes desde IoT Hub a Azure Digital Twins, mediante una función de Azure. Hay muchas configuraciones posibles y estrategias de coincidencia que puede usar para ello, pero el ejemplo de este artículo contiene las siguientes partes:
* Un dispositivo de termómetro en IoT Hub, con un identificador de dispositivo conocido.
* Un gemelo digital para representar el dispositivo, con un identificador coincidente

> [!NOTE]
> En este ejemplo se usa una coincidencia de identificador sencillo entre el identificador de dispositivo y el identificador de gemelo digital correspondiente, pero es posible proporcionar asignaciones más sofisticadas del dispositivo a su gemelo (por ejemplo, con una tabla de asignación).

Cada vez que el dispositivo de termómetro envíe un evento de telemetría de temperatura, se debe actualizar la propiedad *temperature* (temperatura) del gemelo digital. Este escenario se describe en un diagrama a continuación:

:::image type="content" source="media/how-to-ingest-iot-hub-data/events.png" alt-text="Diagrama que muestra un gráfico de flujo. En el gráfico, un dispositivo de IoT Hub envía la telemetría Temperature con IoT Hub a una función de Azure, que actualiza una propiedad de temperatura de un gemelo en Azure Digital Twins." border="false":::

## <a name="add-a-model-and-twin"></a>Adición de un modelo y un gemelo

Necesitará un gemelo que actualizar con la información de IoT Hub.

El modelo tiene este aspecto:
```JSON
{
  "@id": "dtmi:contosocom:DigitalTwins:Thermostat;1",
  "@type": "Interface",
  "@context": "dtmi:dtdl:context;2",
  "contents": [
    {
      "@type": "Property",
      "name": "Temperature",
      "schema": "double"
    }
  ]
}
```

Para **cargar este modelo en la instancia de Digital Twins**, abra el CLI de Azure y ejecute el siguiente comando:

```azurecli-interactive
az dt model create --models '{  "@id": "dtmi:contosocom:DigitalTwins:Thermostat;1",  "@type": "Interface",  "@context": "dtmi:dtdl:context;2",  "contents": [    {      "@type": "Property",      "name": "Temperature",      "schema": "double"    }  ]}' -n {digital_twins_instance_name}
```

[!INCLUDE [digital-twins-known-issue-cloud-shell](../../includes/digital-twins-known-issue-cloud-shell.md)]

A continuación, deberá **crear un gemelo con este modelo**. Use el comando siguiente para crear un gemelo y establezca 0,0 como valor de temperatura inicial.

```azurecli-interactive
az dt twin create --dtmi "dtmi:contosocom:DigitalTwins:Thermostat;1" --twin-id thermostat67 --properties '{"Temperature": 0.0,}' --dt-name {digital_twins_instance_name}
```

[!INCLUDE [digital-twins-known-issue-cloud-shell](../../includes/digital-twins-known-issue-cloud-shell.md)]

La salida de un comando de creación de gemelo correcto debe ser similar a la siguiente:
```json
{
  "$dtId": "thermostat67",
  "$etag": "W/\"0000000-9735-4f41-98d5-90d68e673e15\"",
  "$metadata": {
    "$model": "dtmi:contosocom:DigitalTwins:Thermostat;1",
    "Temperature": {
      "ackCode": 200,
      "ackDescription": "Auto-Sync",
      "ackVersion": 1,
      "desiredValue": 0.0,
      "desiredVersion": 1
    }
  },
  "Temperature": 0.0
}
```

## <a name="create-an-azure-function"></a>Creación de una función de Azure

En esta sección se usan los mismos pasos de inicio de Visual Studio y el esqueleto de la función de Azure en [*Procedimientos: Configuración de una función de Azure para procesar datos*](how-to-create-azure-function.md). El esqueleto controla la autenticación y crea un cliente de servicio, listo para procesar los datos y llamar a las API de Azure Digital Twins como respuesta. 

En los pasos siguientes, agregará código específico para procesar los eventos de telemetría de IoT desde IoT Hub.  

### <a name="add-telemetry-processing"></a>Procesamiento de datos de telemetría
    
Los eventos de telemetría tienen el formato de mensajes del dispositivo. El primer paso para agregar el código de procesamiento de telemetría es extraer la parte pertinente de este mensaje del dispositivo del evento Event Grid. 

Los distintos dispositivos pueden estructurar sus mensajes de forma diferente, por lo que el código de **este paso depende del dispositivo conectado**. 

En el código siguiente se muestra un ejemplo de un dispositivo simple que envía telemetría como JSON. Este ejemplo se explora completamente en [*Tutorial: Conexión de una solución de un extremo a otro*](./tutorial-end-to-end.md). El código siguiente extrae el identificador de dispositivo del dispositivo que envió el mensaje, así como el valor de temperatura.

```csharp
JObject deviceMessage = (JObject)JsonConvert.DeserializeObject(eventGridEvent.Data.ToString());
string deviceId = (string)deviceMessage["systemProperties"]["iothub-connection-device-id"];
var temperature = deviceMessage["body"]["Temperature"];
```

En el ejemplo de código siguiente se toma el valor de identificador y temperatura y se usan para realizar actualizaciones en ese gemelo.

```csharp
//Update twin using device temperature
var uou = new UpdateOperationsUtility();
uou.AppendReplaceOp("/Temperature", temperature.Value<double>());
await client.UpdateDigitalTwinAsync(deviceId, uou.Serialize());
...
```

### <a name="update-your-azure-function-code"></a>Actualización del código de la función de Azure

Ahora que conoce el código de los ejemplos anteriores, abra Visual Studio y reemplace el código de la función de Azure por este código de ejemplo.

```csharp
using System;
using System.Net.Http;
using Azure.Core.Pipeline;
using Azure.DigitalTwins.Core;
using Azure.DigitalTwins.Core.Serialization;
using Azure.Identity;
using Microsoft.Azure.EventGrid.Models;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.EventGrid;
using Microsoft.Extensions.Logging;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;

namespace IotHubtoTwins
{
    public class IoTHubtoTwins
    {
        private static readonly string adtInstanceUrl = Environment.GetEnvironmentVariable("ADT_SERVICE_URL");
        private static readonly HttpClient httpClient = new HttpClient();

        [FunctionName("IoTHubtoTwins")]
        public async void Run([EventGridTrigger] EventGridEvent eventGridEvent, ILogger log)
        {
            if (adtInstanceUrl == null) log.LogError("Application setting \"ADT_SERVICE_URL\" not set");

            try
            {
                //Authenticate with Digital Twins
                ManagedIdentityCredential cred = new ManagedIdentityCredential("https://digitaltwins.azure.net");
                DigitalTwinsClient client = new DigitalTwinsClient(
                    new Uri(adtInstanceUrl), cred, new DigitalTwinsClientOptions 
                    { Transport = new HttpClientTransport(httpClient) });
                log.LogInformation($"ADT service client connection created.");
            
                if (eventGridEvent != null && eventGridEvent.Data != null)
                {
                    log.LogInformation(eventGridEvent.Data.ToString());

                    // Reading deviceId and temperature for IoT Hub JSON
                    JObject deviceMessage = (JObject)JsonConvert.DeserializeObject(eventGridEvent.Data.ToString());
                    string deviceId = (string)deviceMessage["systemProperties"]["iothub-connection-device-id"];
                    var temperature = deviceMessage["body"]["Temperature"];
                    
                    log.LogInformation($"Device:{deviceId} Temperature is:{temperature}");

                    //Update twin using device temperature
                    var uou = new UpdateOperationsUtility();
                    uou.AppendReplaceOp("/Temperature", temperature.Value<double>());
                    await client.UpdateDigitalTwinAsync(deviceId, uou.Serialize());
                }
            }
            catch (Exception e)
            {
                log.LogError($"Error in ingest function: {e.Message}");
            }
        }
    }
}
```

## <a name="connect-your-function-to-iot-hub"></a>Conexión de la función a IoT Hub

1. Configure un destino de evento para los datos del centro de conectividad. En [Azure Portal](https://portal.azure.com/), vaya a su instancia de IoT Hub. En **Eventos**, cree una suscripción para su función de Azure. 

    :::image type="content" source="media/how-to-ingest-iot-hub-data/add-event-subscription.png" alt-text="Captura de pantalla de Azure Portal que muestra la adición de una suscripción de evento.":::

2. En la página **Crear suscripción de eventos**, rellene los campos como se indica a continuación:
    1. En **Nombre**, asigne a la suscripción el nombre que desee.
    2. En **Esquema de eventos**, elija **Esquema de Event Grid**.
    3. En **Nombre del tema del sistema**, elija un nombre único.
    4. En **Tipos de evento**, seleccione **Telemetría del dispositivo** como el tipo de evento por el que filtrar.
    5. En **Detalles del punto de conexión**, seleccione la función de Azure como un punto de conexión.

    :::image type="content" source="media/how-to-ingest-iot-hub-data/event-subscription-2.png" alt-text="Captura de pantalla de Azure Portal que muestra los detalles de una suscripción de evento":::

## <a name="send-simulated-iot-data"></a>Envío de datos de IoT simulados

Para probar la nueva función de entrada, use el simulador de dispositivos de [*Tutorial: Conexión de una solución de un extremo a otro*](./tutorial-end-to-end.md). El tutorial utiliza un proyecto de ejemplo escrito en C#. El código de ejemplo se encuentra aquí: [Ejemplos de Azure Digital Twins](https://docs.microsoft.com/samples/azure-samples/digital-twins-samples/digital-twins-samples). Usará el proyecto **DeviceSimulator** en ese repositorio.

En este tutorial integral, va a completar los siguientes pasos:
1. [*Registro del dispositivo simulado en el centro de IoT*](./tutorial-end-to-end.md#register-the-simulated-device-with-iot-hub)
2. [*Configuración y ejecución de la simulación*](./tutorial-end-to-end.md#configure-and-run-the-simulation)

## <a name="validate-your-results"></a>Validación de los resultados

Mientras se ejecuta el simulador de dispositivos anterior, cambiará el valor de temperatura del gemelo digital. En la CLI de Azure, ejecute el siguiente comando para ver el valor de temperatura.

[!INCLUDE [digital-twins-known-issue-cloud-shell](../../includes/digital-twins-known-issue-cloud-shell.md)]

```azurecli-interactive
az dt twin query -q "select * from digitaltwins" -n {digital_twins_instance_name}
```

La salida debe contener un valor de temperatura similar al siguiente:

```json
{
  "result": [
    {
      "$dtId": "thermostat67",
      "$etag": "W/\"0000000-1e83-4f7f-b448-524371f64691\"",
      "$metadata": {
        "$model": "dtmi:contosocom:DigitalTwins:Thermostat;1",
        "Temperature": {
          "ackCode": 200,
          "ackDescription": "Auto-Sync",
          "ackVersion": 1,
          "desiredValue": 69.75806974934324,
          "desiredVersion": 1
        }
      },
      "Temperature": 69.75806974934324
    }
  ]
}
```

Para ver el cambio de valor, ejecute repetidamente el comando de consulta anterior.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre la entrada y salida de datos con Azure Digital Twins:
* [*Conceptos: Integración con otros servicios*](concepts-integration.md)
