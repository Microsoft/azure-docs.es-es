---
title: Supervisión de operaciones de Azure IoT Hub | Microsoft Docs
description: Describe cómo usar la supervisión de operaciones de Azure IoT Hub para supervisar el estado de las operaciones de su centro de IoT en tiempo real.
author: nberdy
manager: briz
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 03/11/2019
ms.author: nberdy
ms.custom: amqp, devx-track-csharp
ms.openlocfilehash: 9d9824be536ca657d9213a47898ad19b0c39d8b4
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89022027"
---
# <a name="iot-hub-operations-monitoring-deprecated"></a>Supervisión de operaciones de IoT Hub (en desuso)

La supervisión de operaciones de IoT Hub permite supervisar el estado de las operaciones de su centro de IoT en tiempo real. IoT Hub realiza el seguimiento de eventos a través de varias categorías de operaciones. Se puede optar por que los eventos de una o varias categorías se envíen a un punto de conexión de su centro de IoT para su procesamiento. Los usuarios pueden supervisar los datos en busca de errores o configurar un procesamiento más complejo basado en patrones de datos.

>[!NOTE]
>**La supervisión de operaciones de IoT Hub está en desuso y se quitará de IoT Hub el 10 de marzo de 2019.** Para supervisar las operaciones y el mantenimiento de IoT Hub, consulte [Supervisión del mantenimiento de Azure IoT Hub y diagnóstico de problemas rápidamente](iot-hub-monitor-resource-health.md). Para más información sobre la escala de tiempo de desuso, consulte [Monitor your Azure IoT solutions with Azure Monitor and Azure Resource Health](https://azure.microsoft.com/blog/monitor-your-azure-iot-solutions-with-azure-monitor-and-azure-resource-health) (Supervisión de las soluciones de Azure IoT con Azure Monitor y Azure Resource Health).

IoT Hub supervisa seis categorías de eventos:

* Operaciones de identidad de dispositivos
* Telemetría de dispositivo
* Mensajes de nube a dispositivo
* Conexiones
* Cargas de archivos
* Enrutamiento de mensajes

> [!IMPORTANT]
> La supervisión de operaciones de IoT Hub no garantiza una entrega de eventos confiable u ordenada. En función de la infraestructura subyacente de IoT Hub de que se trate, algunos eventos pueden perderse o entregarse de forma desordenada. Use la supervisión de operaciones para generar alertas basadas en señales de error, como intentos de conexión con errores o desconexiones de alta frecuencia de dispositivos específicos. No debe confiar en los eventos de supervisión de operaciones para crear un almacén coherente para el estado del dispositivo, como por ejemplo un almacén en el que se realice un seguimiento del estado de conexión o desconexión de un dispositivo. 

## <a name="how-to-enable-operations-monitoring"></a>Habilitación de la supervisión de operaciones

1. Cree un Centro de IoT. Puede encontrar instrucciones sobre cómo crear un centro de IoT en la guía [Introducción](quickstart-send-telemetry-dotnet.md).

2. Abra la hoja de su centro de IoT. Desde allí, haga clic en **Supervisión de operaciones**.

    ![Acceder a la configuración de la supervisión de operaciones en el portal](./media/iot-hub-operations-monitoring/enable-OM-1.png)

3. Seleccione las categorías que desea supervisar y luego haga clic en **Guardar**. Los eventos están disponibles para su lectura desde el punto de conexión compatible con el Centro de eventos que aparece en **Configuración de supervisión**. El punto de conexión de IoT Hub se denomina `messages/operationsmonitoringevents`.

    ![Configurar operaciones de supervisión en la instancia de IoT Hub](./media/iot-hub-operations-monitoring/enable-OM-2.png)

> [!NOTE]
> Al seleccionar el tipo de supervisión **Detallado** en la categoría **Conexiones**, IoT Hub generará más mensajes de diagnóstico. Para el resto de las categorías, la configuración **Detallado** cambia la cantidad de información que IoT Hub incluye en cada mensaje de error.

## <a name="event-categories-and-how-to-use-them"></a>Categorías de eventos y su uso

Cada categoría de supervisión de operaciones realiza el seguimiento de un tipo diferente de interacción con IoT Hub, y cada categoría de supervisión tiene un esquema que define cómo se estructuran los eventos de esa categoría.

### <a name="device-identity-operations"></a>Operaciones de identidad de dispositivos

La categoría de operaciones de identidad de dispositivo supervisa los errores que se producen cuando se intenta crear, actualizar o eliminar una entrada en el registro de identidades de IoT Hub. El seguimiento de esta categoría resulta útil para los escenarios de aprovisionamiento.

```json
{
    "time": "UTC timestamp",
    "operationName": "create",
    "category": "DeviceIdentityOperations",
    "level": "Error",
    "statusCode": 4XX,
    "statusDescription": "MessageDescription",
    "deviceId": "device-ID",
    "durationMs": 1234,
    "userAgent": "userAgent",
    "sharedAccessPolicy": "accessPolicy"
}
```

### <a name="device-telemetry"></a>Telemetría de dispositivo

La categoría de telemetría del dispositivo realiza el seguimiento de los errores que se producen en el centro de IoT y se relacionan con la canalización de telemetría. Esta categoría incluye los errores que se producen al enviar eventos de telemetría (por ejemplo, la limitación) y recibir eventos de telemetría (por ejemplo, un lector no autorizado). Esta categoría no puede detectar los errores causados por el código que se ejecuta en el propio dispositivo.

```json
{
    "messageSizeInBytes": 1234,
    "batching": 0,
    "protocol": "Amqp",
    "authType": "{\"scope\":\"device\",\"type\":\"sas\",\"issuer\":\"iothub\"}",
    "time": "UTC timestamp",
    "operationName": "ingress",
    "category": "DeviceTelemetry",
    "level": "Error",
    "statusCode": 4XX,
    "statusType": 4XX001,
    "statusDescription": "MessageDescription",
    "deviceId": "device-ID",
    "EventProcessedUtcTime": "UTC timestamp",
    "PartitionId": 1,
    "EventEnqueuedUtcTime": "UTC timestamp"
}
```

### <a name="cloud-to-device-commands"></a>Comandos de nube a dispositivo

La categoría de comandos de nube a dispositivo realiza el seguimiento de los errores que se producen en el centro de IoT y se relacionan con la canalización de mensajes de nube a dispositivo. Esta categoría incluye errores que se producen al enviar mensajes de nube a dispositivo (por ejemplo, un remitente no autorizado), recibir mensajes de nube a dispositivo (por ejemplo, el número de entregas superado) y recibir mensajes de nube a dispositivo (por ejemplo, comentarios expirados). Esta categoría no detecta errores de un dispositivo que trata incorrectamente un mensaje de nube a dispositivo si este se ha entregado correctamente.

```json
{
    "messageSizeInBytes": 1234,
    "authType": "{\"scope\":\"hub\",\"type\":\"sas\",\"issuer\":\"iothub\"}",
    "deliveryAcknowledgement": 0,
    "protocol": "Amqp",
    "time": " UTC timestamp",
    "operationName": "ingress",
    "category": "C2DCommands",
    "level": "Error",
    "statusCode": 4XX,
    "statusType": 4XX001,
    "statusDescription": "MessageDescription",
    "deviceId": "device-ID",
    "EventProcessedUtcTime": "UTC timestamp",
    "PartitionId": 1,
    "EventEnqueuedUtcTime": "UTC timestamp"
}
```

### <a name="connections"></a>Conexiones

La categoría Conexiones supervisa los errores que se producen cuando los dispositivos se conectan o desconectan de un centro de IoT. El seguimiento de esta categoría resulta útil para identificar intentos de conexión no autorizados y para realizar el seguimiento de las situaciones en que una conexión se pierde para los dispositivos en las áreas de conectividad deficiente.

```json
{
    "durationMs": 1234,
    "authType": "{\"scope\":\"hub\",\"type\":\"sas\",\"issuer\":\"iothub\"}",
    "protocol": "Amqp",
    "time": " UTC timestamp",
    "operationName": "deviceConnect",
    "category": "Connections",
    "level": "Error",
    "statusCode": 4XX,
    "statusType": 4XX001,
    "statusDescription": "MessageDescription",
    "deviceId": "device-ID"
}
```

### <a name="file-uploads"></a>Cargas de archivos

La categoría de carga de archivos supervisa los errores que se producen en el centro de IoT y está relacionada con la funcionalidad de carga de archivos. Esta categoría incluye lo siguiente:

* Errores que se producen con el URI de SAS, como cuando caduca antes de que un dispositivo notifique al centro una carga completada.

* Cargas con errores notificadas por el dispositivo.

* Errores que se producen cuando no se encuentra un archivo en el almacenamiento durante la creación del mensaje de notificación de IoT Hub.

Esta categoría no puede detectar los errores que se producen directamente mientras el dispositivo está cargando un archivo a Storage.

```json
{
    "authType": "{\"scope\":\"hub\",\"type\":\"sas\",\"issuer\":\"iothub\"}",
    "protocol": "HTTP",
    "time": " UTC timestamp",
    "operationName": "ingress",
    "category": "fileUpload",
    "level": "Error",
    "statusCode": 4XX,
    "statusType": 4XX001,
    "statusDescription": "MessageDescription",
    "deviceId": "device-ID",
    "blobUri": "http//bloburi.com",
    "durationMs": 1234
}
```

### <a name="message-routing"></a>Enrutamiento de mensajes

La categoría de enrutamiento de mensajes realiza un seguimiento de los errores que se producen durante la evaluación de este proceso y el estado del punto de conexión según lo que observa IoT Hub. Esta categoría incluye eventos, como cuando una regla se evalúa como "sin definir", cuando IoT Hub marca un punto de conexión como inactivo y todos los errores recibidos de un punto de conexión. Esta categoría no incluye errores específicos de los mensajes (por ejemplo, de limitación del dispositivo), que se notifican en la categoría de telemetría de dispositivo.

```json
{
    "messageSizeInBytes": 1234,
    "time": "UTC timestamp",
    "operationName": "ingress",
    "category": "routes",
    "level": "Error",
    "deviceId": "device-ID",
    "messageId": "ID of message",
    "routeName": "myroute",
    "endpointName": "myendpoint",
    "details": "ExternalEndpointDisabled"
}
```

## <a name="connect-to-the-monitoring-endpoint"></a>Conexión con el punto de conexión de supervisión

El punto de conexión de supervisión en su centro de IoT es un punto de conexión compatible con centros de eventos. Puede utilizar cualquier mecanismo que funcione con Event Hubs para leer mensajes de supervisión desde este punto de conexión. El ejemplo siguiente crea un lector básico que no es apto para una implementación de alta capacidad de procesamiento. Para más información sobre cómo procesar los mensajes de Event Hubs, consulte el tutorial [Introducción a Event Hubs](../event-hubs/event-hubs-csharp-ephcs-getstarted.md).

Para conectarse al punto de conexión de supervisión, necesita una cadena de conexión y el nombre del punto de conexión. Los pasos siguientes le muestran cómo buscar los valores necesarios en el portal:

1. En el portal, vaya a la hoja de recursos de IoT Hub.

2. Elija **Supervisión de operaciones** y anote los valores del **nombre compatible con el centro de eventos** y el **punto de conexión compatible con centro de eventos**:

    ![Valores del punto de conexión compatible con centro de eventos](./media/iot-hub-operations-monitoring/monitoring-endpoint.png)

3. Elija **Directivas de acceso compartido** y, a continuación, elija **servicio**. Anote el valor de la **clave principal**:

    ![Clave principal de directiva de acceso compartido del servicio](./media/iot-hub-operations-monitoring/service-key.png)

El siguiente ejemplo de código C# se ha tomado de una aplicación de consola C# del **escritorio clásico de Windows** de Visual Studio. El proyecto tiene el paquete NuGet **WindowsAzure.ServiceBus** instalado.

* Reemplace el marcador de posición de la cadena de conexión por una cadena de conexión que use los valores **punto de conexión compatible con el centro de eventos** y **clave principal** del servicio que anotó anteriormente, como se muestra en el ejemplo siguiente:

    ```csharp
    "Endpoint={your Event Hub-compatible endpoint};SharedAccessKeyName=service;SharedAccessKey={your service primary key value}"
    ```

* Reemplace el marcador de posición del nombre del punto de conexión de supervisión por el valor **nombre compatible con el centro de eventos** que anotó anteriormente.

```csharp
class Program
{
    static string connectionString = "{your monitoring endpoint connection string}";
    static string monitoringEndpointName = "{your monitoring endpoint name}";
    static EventHubClient eventHubClient;

    static void Main(string[] args)
    {
        Console.WriteLine("Monitoring. Press Enter key to exit.\n");

        eventHubClient = EventHubClient.CreateFromConnectionString(connectionString, monitoringEndpointName);
        var d2cPartitions = eventHubClient.GetRuntimeInformation().PartitionIds;
        CancellationTokenSource cts = new CancellationTokenSource();
        var tasks = new List<Task>();

        foreach (string partition in d2cPartitions)
        {
            tasks.Add(ReceiveMessagesFromDeviceAsync(partition, cts.Token));
        }

        Console.ReadLine();
        Console.WriteLine("Exiting...");
        cts.Cancel();
        Task.WaitAll(tasks.ToArray());
    }

    private static async Task ReceiveMessagesFromDeviceAsync(string partition, CancellationToken ct)
    {
        var eventHubReceiver = eventHubClient.GetDefaultConsumerGroup().CreateReceiver(partition, DateTime.UtcNow);
        while (true)
        {
            if (ct.IsCancellationRequested)
            {
                await eventHubReceiver.CloseAsync();
                break;
            }

            EventData eventData = await eventHubReceiver.ReceiveAsync(new TimeSpan(0,0,10));

            if (eventData != null)
            {
                string data = Encoding.UTF8.GetString(eventData.GetBytes());
                Console.WriteLine("Message received. Partition: {0} Data: '{1}'", partition, data);
            }
        }
    }
}
```

## <a name="next-steps"></a>Pasos siguientes

Para explorar aún más las funcionalidades de IoT Hub, consulte:

* [Guía para desarrolladores de IoT Hub](iot-hub-devguide.md)

* [Implementación de IA en dispositivos perimetrales con Azure IoT Edge](../iot-edge/tutorial-simulate-device-linux.md)