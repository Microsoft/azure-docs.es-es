---
title: Azure IoT Hub y Event Grid | Microsoft Docs
description: Use Azure Event Grid para desencadenar procesos basados en acciones que se producen en IoT Hub.
author: robinsh
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 02/20/2019
ms.author: robinsh
ms.custom:
- amqp
- mqtt
- 'Role: Cloud Development'
ms.openlocfilehash: a5707ef266f3d49bdcbff9793a0b90e6c3f4cb68
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87327657"
---
# <a name="react-to-iot-hub-events-by-using-event-grid-to-trigger-actions"></a>Reacción a eventos de IoT Hub usando Event Grid para desencadenar acciones

Azure IoT Hub se integra con Azure Event Grid para poder enviar notificaciones de eventos a otros servicios y desencadenar procesos de descarga. Configure las aplicaciones empresariales para escuchar eventos de IoT Hub, a fin de poder crear reacciones a eventos críticos de manera confiable, escalable y segura. Por ejemplo, cree una aplicación que actualice una base de datos, cree un vale de trabajo y envíe una notificación por correo electrónico cada vez que se registre un nuevo dispositivo IoT en el centro de IoT.

[Azure Event Grid](../event-grid/overview.md) es un servicio de enrutamiento de eventos totalmente administrado que utiliza un modelo de publicación-suscripción. Event Grid tiene compatibilidad integrada para servicios de Azure, como [Azure Functions](../azure-functions/functions-overview.md) y [Azure Logic Apps](../logic-apps/logic-apps-what-are-logic-apps.md), y puede proporcionar alertas de eventos a los servicios que no son de Azure mediante webhooks. Para obtener una lista completa de los controladores de eventos que Event Grid admite, vea [una introducción a Azure Event Grid](../event-grid/overview.md).

![Arquitectura de Azure Event Grid](./media/iot-hub-event-grid/event-grid-functional-model.png)

## <a name="regional-availability"></a>Disponibilidad regional

La integración de Event Grid está disponible en centros de IoT ubicados en las regiones donde se admite Event Grid. Para conocer la lista más reciente de regiones disponibles, vea [Introducción a Azure Event Grid](../event-grid/overview.md).

## <a name="event-types"></a>Tipos de eventos

IoT Hub publica los siguientes tipos de eventos:

| Tipo de evento | Descripción |
| ---------- | ----------- |
| Microsoft.Devices.DeviceCreated | Se publica cuando se registra un dispositivo en una instancia de IoT Hub. |
| Microsoft.Devices.DeviceDeleted | Se publica cuando se elimina un dispositivo de una instancia de IoT Hub. |
| Microsoft.Devices.DeviceConnected | Se publica cuando se conecta un dispositivo a una instancia de IoT Hub. |
| Microsoft.Devices.DeviceDisconnected | Se publica cuando se desconecta un dispositivo de una instancia de IoT Hub. |
| Microsoft.Devices.DeviceTelemetry | Se publica cuando se envía un mensaje de telemetría del dispositivo a un centro de IoT. |

Use Azure Portal o la CLI de Azure para configurar qué eventos publicar desde cada instancia de IoT Hub. Por ejemplo, pruebe con el tutorial [Envío de notificaciones por correo electrónico sobre eventos de Azure IoT Hub mediante Logic Apps](../event-grid/publish-iot-hub-events-to-logic-apps.md).

## <a name="event-schema"></a>Esquema del evento

Los eventos de IoT Hub contienen toda la información necesaria para responder a cualquier cambio que se produzca en el ciclo de vida del dispositivo. Puede identificar un evento de IoT Hub comprobando que la propiedad eventType empieza por **Microsoft.Devices**. Para más información sobre cómo usar las propiedades de eventos de Event Grid, vea el [Esquema de eventos de Azure Event Grid](../event-grid/event-schema.md).

### <a name="device-connected-schema"></a>Esquema de dispositivo conectado

En el ejemplo siguiente, se muestra el esquema de un evento de dispositivo conectado:

```json
[{  
  "id": "f6bbf8f4-d365-520d-a878-17bf7238abd8",
  "topic": "/SUBSCRIPTIONS/<subscription ID>/RESOURCEGROUPS/<resource group name>/PROVIDERS/MICROSOFT.DEVICES/IOTHUBS/<hub name>",
  "subject": "devices/LogicAppTestDevice",
  "eventType": "Microsoft.Devices.DeviceConnected",
  "eventTime": "2018-06-02T19:17:44.4383997Z",
  "data": {
      "deviceConnectionStateEventInfo": {
        "sequenceNumber":
          "000000000000000001D4132452F67CE200000002000000000000000000000001"
      },
    "hubName": "egtesthub1",
    "deviceId": "LogicAppTestDevice",
    "moduleId" : "DeviceModuleID",
  }, 
  "dataVersion": "1",
  "metadataVersion": "1"
}]
```

### <a name="device-telemetry-schema"></a>Esquema de telemetría del dispositivo

El mensaje de telemetría del dispositivo debe estar en un formato JSON válido, el valor de contentType debe establecerse en **application/json** y contentEncoding establecerse en **UTF-8** en las [propiedades del sistema](iot-hub-devguide-routing-query-syntax.md#system-properties) del mensaje. Ambas propiedades no distinguen mayúsculas de minúsculas. Si no está establecida la codificación del contenido, IoT Hub escribirá los mensajes en formato codificado de base 64.

Puede enriquecer los eventos de telemetría del dispositivo antes de que se publiquen en Event Grid; para ello, seleccione el punto de conexión como Event Grid. Para más información, consulte la [introducción al enriquecimiento de mensajes](iot-hub-message-enrichments-overview.md).

En el ejemplo siguiente, se muestra el esquema de un evento de telemetría del dispositivo:

```json
[{  
  "id": "9af86784-8d40-fe2g-8b2a-bab65e106785",
  "topic": "/SUBSCRIPTIONS/<subscription ID>/RESOURCEGROUPS/<resource group name>/PROVIDERS/MICROSOFT.DEVICES/IOTHUBS/<hub name>",
  "subject": "devices/LogicAppTestDevice",
  "eventType": "Microsoft.Devices.DeviceTelemetry",
  "eventTime": "2019-01-07T20:58:30.48Z",
  "data": {
      "body": {
          "Weather": {
              "Temperature": 900
            },
            "Location": "USA"
        },
        "properties": {
            "Status": "Active"
        },
        "systemProperties": {
          "iothub-content-type": "application/json",
          "iothub-content-encoding": "utf-8",
          "iothub-connection-device-id": "d1",
          "iothub-connection-auth-method": "{\"scope\":\"device\",\"type\":\"sas\",\"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}",
          "iothub-connection-auth-generation-id": "123455432199234570",
          "iothub-enqueuedtime": "2019-01-07T20:58:30.48Z",
          "iothub-message-source": "Telemetry"
        }
  },
  "dataVersion": "",
  "metadataVersion": "1"
}]
```

### <a name="device-created-schema"></a>Esquema de dispositivo creado

En el ejemplo siguiente, se muestra el esquema de un evento de dispositivo creado:

```json
[{
  "id": "56afc886-767b-d359-d59e-0da7877166b2",
  "topic": "/SUBSCRIPTIONS/<subscription ID>/RESOURCEGROUPS/<resource group name>/PROVIDERS/MICROSOFT.DEVICES/IOTHUBS/<hub name>",
  "subject": "devices/LogicAppTestDevice",
  "eventType": "Microsoft.Devices.DeviceCreated",
  "eventTime": "2018-01-02T19:17:44.4383997Z",
  "data": {
    "twin": {
      "deviceId": "LogicAppTestDevice",
      "etag": "AAAAAAAAAAE=",
      "deviceEtag":"null",
      "status": "enabled",
      "statusUpdateTime": "0001-01-01T00:00:00",
      "connectionState": "Disconnected",
      "lastActivityTime": "0001-01-01T00:00:00",
      "cloudToDeviceMessageCount": 0,
      "authenticationType": "sas",
      "x509Thumbprint": {
        "primaryThumbprint": null,
        "secondaryThumbprint": null
      },
      "version": 2,
      "properties": {
        "desired": {
          "$metadata": {
            "$lastUpdated": "2018-01-02T19:17:44.4383997Z"
          },
          "$version": 1
        },
        "reported": {
          "$metadata": {
            "$lastUpdated": "2018-01-02T19:17:44.4383997Z"
          },
          "$version": 1
        }
      }
    },
    "hubName": "egtesthub1",
    "deviceId": "LogicAppTestDevice"
  },
  "dataVersion": "1",
  "metadataVersion": "1"
}]
```

Para obtener una descripción detallada de cada propiedad, consulte [Esquema de eventos de Azure Event Grid para IoT Hub](../event-grid/event-schema-iot-hub.md).

## <a name="filter-events"></a>Filtrado de eventos

Las suscripciones de eventos de IoT Hub pueden filtrar los eventos según el tipo de evento, contenido de datos y tema, que es el nombre de dispositivo.

Event Grid permite [filtrar](../event-grid/event-filtering.md) por tipos de eventos, temas y contenido de datos. Al crear la suscripción de Event Grid, puede elegir suscribirse a eventos de IoT seleccionados. Los filtros de asunto en Event Grid funcionan según las coincidencias de **Empieza con** (prefijo) y de **Termina con** (sufijo). El filtro utiliza un operador `AND`, por lo que se entregan al suscriptor los eventos con un asunto que coincide con el prefijo y el sufijo.

El asunto de los eventos de IoT Hub utiliza el formato:

```json
devices/{deviceId}
```

Event Grid también permite filtrar los atributos de cada evento, incluido el contenido de datos. Esto le permite elegir qué eventos se entregan en función del contenido del mensaje de telemetría. Consulte el [filtrado avanzado](../event-grid/event-filtering.md#advanced-filtering) para ver ejemplos. Para filtrar por el cuerpo del mensaje de telemetría, debe establecer contentType en **application/json** y contentEncoding en **UTF-8** en las [propiedades del sistema](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-routing-query-syntax#system-properties) del mensaje. Ambas propiedades no distinguen mayúsculas de minúsculas.

Para eventos que no son de telemetría como DeviceConnected, DeviceDisconnected, DeviceCreated y DeviceDeleted, se puede usar el filtrado de Event Grid al crear la suscripción. Para eventos de telemetría, además del filtrado en Event Grid, los usuarios también pueden filtrar en dispositivos gemelos, propiedades de mensajes y cuerpo mediante la consulta de enrutamiento de mensajes. 

Al suscribirse a eventos de telemetría mediante Event Grid, IoT Hub crea una ruta de mensaje predeterminada para enviar los mensajes del dispositivo del tipo de origen de datos a Event Grid. Para más información sobre el enrutamiento de mensajes, consulte [Enrutamiento de mensajes de IoT Hub](iot-hub-devguide-messages-d2c.md). Esta ruta será visible en el portal en IoT Hub > Enrutamiento de mensajes. Solo se crea una ruta a Event Grid independientemente del número de suscripciones de Event Grid creadas para eventos de telemetría. Por lo tanto, si necesita varias suscripciones con filtros diferentes, puede usar el operador OR en estas consultas en la misma ruta. La creación y eliminación de la ruta se controla mediante la suscripción de eventos de telemetría a través de Event Grid. No se puede crear ni eliminar una ruta a Event Grid mediante el enrutamiento de mensajes de IoT Hub.

Para filtrar los mensajes antes de que ser envíen los datos de telemetría, puede actualizar la [consulta de enrutamiento](iot-hub-devguide-routing-query-syntax.md). Tenga en cuenta que esa consulta de enrutamiento puede aplicarse al cuerpo del mensaje solo si el cuerpo es JSON. También debe establecer contentType en **application/json** y contentEncoding en **UTF-8** en las [propiedades del sistema](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-routing-query-syntax#system-properties) del mensaje.

## <a name="limitations-for-device-connected-and-device-disconnected-events"></a>Limitaciones de los eventos de dispositivo conectado y dispositivo desconectado

Para recibir los eventos de estado de conexión del dispositivo, un dispositivo tiene que hacer una operación "D2C Send Telemetry" O "C2D Receive Message" con IoT Hub. De todas formas, debe tener en cuenta que si un dispositivo usa el protocolo AMQP para conectarse con IoT Hub, se recomienda que realice una operación "C2D Receive Message"; de lo contrario, las notificaciones de estado de conexión se pueden retrasar unos minutos. Si el dispositivo usa el protocolo MQTT, IoT Hub mantendrá el vínculo C2D abierto. Para AMQP puede abrir el vínculo C2D mediante una llamada a [Receive Async API](https://docs.microsoft.com/dotnet/api/microsoft.azure.devices.client.deviceclient.receiveasync?view=azure-dotnet) para el SDK de C# de IoT Hub, o al [cliente del dispositivo para AMQP](iot-hub-amqp-support.md#device-client).

Si está enviando telemetría, el vínculo D2C está abierto. 

Si la conexión del dispositivo es intermitente, lo que significa que el dispositivo se conecta y desconecta con frecuencia, no se le enviarán todos y cada uno de los estados de conexión, sino que se publicará el estado de conexión de ese momento, que se toma de una serie de instantáneas periódicas, mientras la conexión siga siendo intermitente. Si se recibe el mismo evento de estado de conexión con diferentes números de secuencia, o eventos de estado de conexión distintos, quiere decir que se ha producido un cambio en el estado de conexión del dispositivo.

## <a name="tips-for-consuming-events"></a>Sugerencias para consumir eventos

Las aplicaciones que controlan los eventos de IoT Hub deben seguir estas prácticas recomendadas:

* Se pueden configurar varias suscripciones para enrutar los eventos al mismo controlador de eventos, por lo que no hay que asumir que los eventos proceden de un origen particular. Compruebe siempre el tema del mensaje para asegurarse de que se trata de la instancia de IoT Hub que está esperando.

* No asuma que todos los eventos que recibe son del tipo de evento que espera. Compruebe siempre eventType antes de procesar el mensaje.

* Los mensajes pueden llegar desordenados o después de un retraso. Utilice el campo de la etiqueta de entidad para saber si la información sobre los objetos está actualizada para los eventos de dispositivos creados o de dispositivos eliminados.

## <a name="next-steps"></a>Pasos siguientes

* [Pruebe el tutorial de eventos de IoT Hub](../event-grid/publish-iot-hub-events-to-logic-apps.md)

* [Aprenda a ordenar eventos conectados y desconectados de dispositivos](iot-hub-how-to-order-connection-state-events.md)

* [Más información sobre Event Grid](../event-grid/overview.md)

* [Compare las diferencias entre el enrutamiento de eventos y mensajes de IoT Hub](iot-hub-event-grid-routing-comparison.md)

* [Obtenga información sobre cómo usar los eventos de telemetría de IoT para implementar el análisis espacial de IoT mediante Azure Maps](../azure-maps/tutorial-iot-hub-maps.md#create-an-azure-function-and-add-an-event-grid-subscription)
