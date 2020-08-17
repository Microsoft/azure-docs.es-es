---
title: 'Guía para desarrolladores: IoT Plug and Play (versión preliminar) | Microsoft Docs'
description: Descripción de IoT Plug and Play para desarrolladores
author: rido-min
ms.author: rmpablos
ms.date: 07/16/2020
ms.topic: conceptual
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: ef221ea068f2786a4a84f20a29e80dd7176f06c6
ms.sourcegitcommit: 46f8457ccb224eb000799ec81ed5b3ea93a6f06f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87337422"
---
# <a name="iot-plug-and-play-preview-developer-guide"></a>Guía para desarrolladores de IoT Plug and Play (versión preliminar)

IoT Plug and Play (versión preliminar) permite crear dispositivos inteligentes que anuncian sus funcionalidades en aplicaciones de Azure IoT. Los dispositivos IoT Plug and Play no requieren configuración manual cuando un cliente los conecta a aplicaciones habilitadas para IoT Plug and Play.

En esta guía se describen los pasos básicos necesarios para crear un dispositivo que siga las [convenciones de IoT Plug and Play](concepts-convention.md) y las API de REST disponibles que puede usar para interactuar con el dispositivo.

Para crear un dispositivo IoT Plug and Play, siga estos pasos:

1. Asegúrese de que el dispositivo usa el protocolo MQTT o MQTT sobre WebSockets para conectarse a Azure IoT Hub.
1. Cree un modelo de [Lenguaje de definición de Digital Twins (DTDL)](https://github.com/Azure/opendigitaltwins-dtdl) para describir el dispositivo. Para más información, consulte [Descripción de los componentes de los modelos de IoT Plug and Play](concepts-components.md).
1. Actualice el dispositivo para anunciar el elemento `model-id` como parte de la conexión del dispositivo.
1. Implemente telemetría, propiedades y comandos mediante las [convenciones de IoT Plug and Play](concepts-convention.md).

Una vez que la implementación del dispositivo esté lista, use [Azure IoT Explorer](howto-use-iot-explorer.md) para validar que el dispositivo sigue las convenciones de IoT Plug and Play.

> [!Tip]
> Todos los fragmentos de código de este artículo usan C#, pero los conceptos se aplican a cualquiera de los SDK disponibles para C, Python, Node y Java.

## <a name="model-id-announcement"></a>Anuncio del id. de modelo

Para anunciar el id. de modelo, el dispositivo debe incluirlo en la información de conexión:

```csharp
DeviceClient.CreateFromConnectionString(
  connectionString,
  TransportType.Mqtt,
  new ClientOptions() { ModelId = modelId })
```

La nueva sobrecarga `ClientOptions` está disponible en todos los métodos de `DeviceClient` usados para inicializar una conexión.

El anuncio del id. de modelo se ha agregado a las siguientes versiones de los SDK

|SDK|Versión|
|---|-------|
|C-SDK|1.3.9|
|.NET|1.27.0|
|Java|1.14.0|
|Nodo|1.17.0|
|Python|2.1.4|

## <a name="implement-telemetry-properties-and-commands"></a>Implementación de telemetría, propiedades y comandos

Como se describe en [Descripción de componentes de los modelos de IoT Plug and Play](concepts-components.md), los creadores de dispositivos deben decidir si quieren usar componentes para describir sus dispositivos. Cuando se usan componentes, los dispositivos deben seguir las reglas descritas en esta sección.

### <a name="telemetry"></a>Telemetría

Los modelos sin componentes no requieren ninguna propiedad especial.

Cuando se usan componentes, los dispositivos deben establecer una propiedad de mensaje con el nombre de componente:

```c#
public async Task SendComponentTelemetryValueAsync(string componentName, string serializedTelemetry)
{
  var message = new Message(Encoding.UTF8.GetBytes(serializedTelemetry));
  message.Properties.Add("$.sub", componentName);
  message.ContentType = "application/json";
  message.ContentEncoding = "utf-8";
  await deviceClient.SendEventAsync(message);
}
```

### <a name="read-only-properties"></a>Propiedades de solo lectura

Los modelos sin componentes no requieren ninguna construcción especial:

```csharp
TwinCollection reportedProperties = new TwinCollection();
reportedProperties["maxTemperature"] = 38.7;
await client.UpdateReportedPropertiesAsync(reportedProperties);
```

El dispositivo gemelo se actualiza con la siguiente propiedad notificada:

```json
{
  "reported": {
      "maxTemperature" : 38.7
  }
}
```

Al usar los componentes, se deben crear propiedades dentro del nombre de componente:

```csharp
TwinCollection reportedProperties = new TwinCollection();
TwinCollection component = new TwinCollection();
component["maxTemperature"] = 38.7;
component["__t"] = "c"; // marker to identify a component
reportedProperties["thermostat1"] = component;
await client.UpdateReportedPropertiesAsync(reportedProperties);
```

El dispositivo gemelo se actualiza con la siguiente propiedad notificada:

```json
{
  "reported": {
    "thermostat1" : {  
      "__t" : "c",  
      "maxTemperature" : 38.7
     } 
  }
}
```

### <a name="writable-properties"></a>Propiedades editables

Estas propiedades pueden establecerse por el dispositivo o actualizarse por la solución. Si la solución actualiza una propiedad, el cliente recibe una notificación como una devolución de llamada en `DeviceClient`. Para seguir las convenciones de IoT Plug and Play, el dispositivo debe informar al servicio de que la propiedad se ha recibido correctamente.

#### <a name="report-a-writable-property"></a>Notificación de una propiedad editable

Cuando un dispositivo notifica una propiedad editable, debe incluir los valores `ack` definidos en las convenciones.

Para notificar una propiedad editable sin componentes:

```csharp
TwinCollection reportedProperties = new TwinCollection();
TwinCollection ackProps = new TwinCollection();
ackProps["value"] = 23.2;
ackProps["ac"] = 200; // using HTTP status codes
ackProps["av"] = 0; // not readed from a desired property
ackProps["ad"] = "reported default value";
reportedProperties["targetTemperature"] = ackProps;
await client.UpdateReportedPropertiesAsync(reportedProperties);
```

El dispositivo gemelo se actualiza con la siguiente propiedad notificada:

```json
{
  "reported": {
      "targetTemperature": {
          "value": 23.2,
          "ac": 200,
          "av": 3,
          "ad": "complete"
      }
  }
}
```

Para notificar una propiedad editable de un componente, el gemelo debe incluir un marcador:

```csharp
TwinCollection reportedProperties = new TwinCollection();
TwinCollection component = new TwinCollection();
TwinCollection ackProps = new TwinCollection();
component["__t"] = "c"; // marker to identify a component
ackProps["value"] = 23.2;
ackProps["ac"] = 200; // using HTTP status codes
ackProps["av"] = 0; // not read from a desired property
ackProps["ad"] = "reported default value";
component["targetTemperature"] = ackProps;
reportedProperties["thermostat1"] = component;
await client.UpdateReportedPropertiesAsync(reportedProperties);
```

El dispositivo gemelo se actualiza con la siguiente propiedad notificada:

```json
{
  "reported": {
    "thermostat1": {
      "__t" : "c",
      "targetTemperature": {
          "value": 23.2,
          "ac": 200,
          "av": 3,
          "ad": "complete"
      }
    }
  }
}
```

#### <a name="subscribe-to-desired-property-updates"></a>Suscripción a las actualizaciones de propiedades deseadas

Los servicios pueden actualizar las propiedades deseadas que desencadenan una notificación en los dispositivos conectados. Esta notificación incluye las propiedades deseadas actualizadas, incluido el número de versión que identifica la actualización. Los dispositivos deben responder con el mismo mensaje `ack` que las propiedades notificadas.

Los modelos sin componentes ven la propiedad única y crean el `ack` notificado con la versión recibida:

```csharp
await client.SetDesiredPropertyUpdateCallbackAsync(async (desired, ctx) => 
{
  JValue targetTempJson = desired["targetTemperature"];
  double targetTemperature = targetTempJson.Value<double>();

  TwinCollection reportedProperties = new TwinCollection();
  TwinCollection ackProps = new TwinCollection();
  ackProps["value"] = targetTemperature;
  ackProps["ac"] = 200;
  ackProps["av"] = desired.Version; 
  ackProps["ad"] = "desired property received";
  reportedProperties["targetTemperature"] = ackProps;

  await client.UpdateReportedPropertiesAsync(reportedProperties);
}, null);
```

El dispositivo gemelo muestra la propiedad en las secciones desired y reported:

```json
{
  "desired" : {
    "targetTemperature": 23.2,
    "$version" : 3
  },
  "reported": {
      "targetTemperature": {
          "value": 23.2,
          "ac": 200,
          "av": 3,
          "ad": "complete"
      }
  }
}
```

Los modelos con componentes reciben las propiedades deseadas encapsuladas con el nombre de componente y deben notificar la propiedad notificada `ack`:

```csharp
await client.SetDesiredPropertyUpdateCallbackAsync(async (desired, ctx) =>
{
  JObject thermostatComponent = desired["thermostat1"];
  JToken targetTempProp = thermostatComponent["targetTemperature"];
  double targetTemperature = targetTempProp.Value<double>();

  TwinCollection reportedProperties = new TwinCollection();
  TwinCollection component = new TwinCollection();
  TwinCollection ackProps = new TwinCollection();
  component["__t"] = "c"; // marker to identify a component
  ackProps["value"] = targetTemperature;
  ackProps["ac"] = 200; // using HTTP status codes
  ackProps["av"] = desired.Version; // not readed from a desired property
  ackProps["ad"] = "desired property received";
  component["targetTemperature"] = ackProps;
  reportedProperties["thermostat1"] = component;

  await client.UpdateReportedPropertiesAsync(reportedProperties);
}, null);
```

En el dispositivo gemelo de los componentes se muestran las secciones desired y reported de la siguiente manera:

```json
{
  "desired" : {
    "thermostat1" : {
        "__t" : "c",
        "targetTemperature": 23.2,
    }
    "$version" : 3
  },
  "reported": {
    "thermostat1" : {
        "__t" : "c",
      "targetTemperature": {
          "value": 23.2,
          "ac": 200,
          "av": 3,
          "ad": "complete"
      }
    }
  }
}
```

### <a name="commands"></a>Comandos:

Los modelos sin componentes reciben el nombre del comando, ya que el servicio lo invocó.

Los modelos con componentes recibirán el nombre del comando con el prefijo del componente y el separador `*`.

```csharp
await client.SetMethodHandlerAsync("themostat*reboot", (MethodRequest req, object ctx) =>
{
  Console.WriteLine("REBOOT");
  return Task.FromResult(new MethodResponse(200));
},
null);
```

#### <a name="request-and-response-payloads"></a>Cargas de solicitud y respuesta

Los comandos usan tipos para definir sus cargas de solicitud y respuesta. Un dispositivo debe deserializar el parámetro de entrada y serializar la respuesta. En el ejemplo siguiente se muestra cómo implementar un comando con tipos complejos definidos en las cargas:

```json
{
  "@type": "Command",
  "name": "start",
  "request": {
    "name": "startRequest",
    "schema": {
      "@type": "Object",
      "fields": [
        {
          "name": "startPriority",
          "schema": "integer"
        },
        {
          "name": "startMessage",
          "schema" : "string"
        }
      ]
    }
  },
  "response": {
    "name": "startReponse",
    "schema": {
      "@type": "Object",
      "fields": [
        {
            "name": "startupTime",
            "schema": "integer" 
        },
        {
          "name": "startupMessage",
          "schema": "string"
        }
      ]
    }
  }
}
```

Los fragmentos de código siguientes muestran cómo un dispositivo implementa esta definición de comando, incluidos los tipos que se usan para habilitar la serialización y deserialización:

```csharp
class startRequest
{
  public int startPriority { get; set; }
  public string startMessage { get; set; }
}

class startResponse
{
  public int startupTime { get; set; }
  public string startupMessage { get; set; }
}

// ... 

await client.SetMethodHandlerAsync("start", (MethodRequest req, object ctx) =>
{
  var startRequest = JsonConvert.DeserializeObject<startRequest>(req.DataAsJson);
  Console.WriteLine($"Received start command with priority ${startRequest.startPriority} and ${startRequest.startMessage}");

  var startResponse = new startResponse
  {
    startupTime = 123,
    startupMessage = "device started with message " + startRequest.startMessage
  };

  string responsePayload = JsonConvert.SerializeObject(startResponse);
  MethodResponse response = new MethodResponse(Encoding.UTF8.GetBytes(responsePayload), 200);
  return Task.FromResult(response);
},null);
```

> [!Tip]
> Los nombres de solicitud y respuesta no están presentes en las cargas serializadas transmitidas a través de la conexión.

## <a name="interact-with-the-device"></a>Interacción con el dispositivo 

IoT Plug and Play le permite usar dispositivos que han anunciado su id. de modelo en su centro de IoT. Por ejemplo, puede acceder directamente a las propiedades y los comandos de un dispositivo.

Para usar un dispositivo IoT Plug and Play que esté conectado a su centro de IoT, use la API REST de IoT Hub o uno de los SDK de lenguaje de IoT. En los ejemplos siguientes se usa la API REST de IoT Hub. La versión actual de la API es `2020-05-31-preview`. Anexe `?api-version=2020-05-31` a las llamadas a la API REST.

Si se llama al dispositivo termostato `t-123`, logrará que su dispositivo implemente todas las propiedades con una llamada GET de la API REST:

```REST
GET /digitalTwins/t-123
```

Esta llamada incluirá la propiedad JSON `$metadata.$model` con el id. de modelo anunciado por el dispositivo.

Con la plantilla de API de REST `GET /DigitalTwin/{device-id}`, en la que `{device-id}` es el identificador del dispositivo, se accede a todas las propiedades de todas las interfaces:

```REST
GET /digitalTwins/{device-id}
```

Puede llamar directamente a los comandos de dispositivo IoT Plug and Play. Si el componente `Thermostat` del dispositivo `t-123` tiene un comando `restart`, puede usar una llamada POST de la API de REST para llamarlo:

```REST
POST /digitalTwins/t-123/components/Thermostat/commands/restart
```

En general, se puede llamar a los comandos mediante esta plantilla de API REST:

- `device-id`: el identificador del dispositivo.
- `component-name`: el nombre de la interfaz de la sección de implementaciones del modelo de funcionalidad del dispositivo.
- `command-name`: el nombre del comando.

```REST
/digitalTwins/{device-id}/components/{component-name}/commands/{command-name}
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido acerca del modelado de dispositivos, estos son algunos recursos adicionales:

- [Lenguaje de definición de Digital Twins (DTDL)](https://github.com/Azure/opendigitaltwins-dtdl)
- [SDK para dispositivos C](https://docs.microsoft.com/azure/iot-hub/iot-c-sdk-ref/)
- [API REST de IoT](https://docs.microsoft.com/rest/api/iothub/device)
- [Componentes del modelo](./concepts-components.md)
