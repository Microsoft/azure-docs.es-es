---
title: Envío de mensajes de seguridad de dispositivos de Defender para IoT
description: Aprenda a enviar mensajes de seguridad mediante Defender para IoT.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/09/2020
ms.author: mlottner
ms.custom: devx-track-javascript
ms.openlocfilehash: 476e242559a5c6f8f389a411a563ffb97e683fb3
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90931876"
---
# <a name="send-security-messages-sdk"></a>SDK de Send Security Message

En esta guía paso a paso se describen las funcionalidades del servicio Defender para IoT al elegir recopilar y enviar mensajes de seguridad de un dispositivo sin utilizar ningún agente de Defender para IoT, y se explica cómo hacerlo.

En esta guía, aprenderá a:

> [!div class="checklist"]
> * Enviar mensajes de seguridad mediante el SDK de C de Azure IoT
> * Enviar mensajes de seguridad mediante el SDK de C# de Azure IoT
> * Enviar mensajes de seguridad mediante el SDK de Python de Azure IoT
> * Enviar mensajes de seguridad mediante el SDK de Node.js de Azure IoT
> * Enviar mensajes de seguridad mediante el SDK de Java de Azure IoT

## <a name="defender-for-iot-capabilities"></a>Funcionalidades de Defender para IoT

Defender para IoT puede procesar y analizar cualquier tipo de datos de mensajes de seguridad, siempre que los datos enviados se ajusten al [esquema de Defender para IoT](https://aka.ms/iot-security-schemas) y el mensaje en cuestión esté configurado como un mensaje de seguridad.

## <a name="security-message"></a>Mensaje de seguridad

Defender para IoT define un mensaje de seguridad teniendo en cuenta los siguientes criterios:

- Si el mensaje se envió con SDK de Azure IoT.
- Si el mensaje se ajusta al [esquema de mensaje de seguridad](https://aka.ms/iot-security-schemas).
- Si el mensaje se ha configurado como un mensaje de seguridad antes de enviarse.

Cada mensaje de seguridad incluye los metadatos del remitente, como `AgentId`, `AgentVersion`, `MessageSchemaVersion` y una lista de eventos de seguridad.
El esquema define las propiedades válidas y obligatorias del mensaje de seguridad, incluidos los tipos de eventos.

> [!NOTE]
> Los mensajes enviados que no cumplan el esquema se ignoran. Asegúrese de comprobar el esquema antes de iniciar el envío de datos, ya que los mensajes ignorados no se almacenan.

> [!NOTE]
> Los mensajes enviados que no estén configurados como mensajes de seguridad con el SDK de Azure IoT no se enrutarán a la canalización de Defender para IoT.

## <a name="valid-message-example"></a>Ejemplo de mensaje válido

En el siguiente ejemplo se muestra un objeto de mensaje de seguridad válido. El ejemplo contiene los metadatos del mensaje y un evento de seguridad `ProcessCreate`.

Una vez que el mensaje esté configurado como mensaje de seguridad y se haya enviado, Defender para IoT lo procesará.

```json
"AgentVersion": "0.0.1",
"AgentId": "e89dc5f5-feac-4c3e-87e2-93c16f010c25",
"MessageSchemaVersion": "1.0",
"Events": [
    {
        "EventType": "Security",
        "Category": "Triggered",
        "Name": "ProcessCreate",
        "IsEmpty": false,
        "PayloadSchemaVersion": "1.0",
        "Id": "21a2db0b-44fe-42e9-9cff-bbb2d8fdf874",
        "TimestampLocal": "2019-01-27 15:48:52Z",
        "TimestampUTC": "2019-01-27 13:48:52Z",
        "Payload":
            [
                {
                    "Executable": "/usr/bin/myApp",
                    "ProcessId": 11750,
                    "ParentProcessId": 1593,
                    "UserName": "aUser",
                    "CommandLine": "myApp -a -b"
                }
            ]
    }
]
```

## <a name="send-security-messages"></a>Envío de mensajes de seguridad

Envíe mensajes de seguridad *sin* usar el agente de Defender para IoT mediante el [SDK de dispositivo C de Azure IoT](https://github.com/Azure/azure-iot-sdk-c/tree/public-preview), el [SDK de dispositivo C# de Azure IoT](https://github.com/Azure/azure-iot-sdk-csharp/tree/preview), el [SDK de Node.js de Azure IoT](https://github.com/Azure/azure-iot-sdk-node), el [SDK de Python de Azure IoT](https://github.com/Azure/azure-iot-sdk-python) o el [SDK de Java de Azure IoT](https://github.com/Azure/azure-iot-sdk-java).

Para enviar los datos de los dispositivos para que los procese Defender para IoT, use una de las siguientes API para marcar los mensajes para que se enruten correctamente a la canalización de procesamiento de Defender para IoT.

Todos los datos que se envíen, aunque se hayan marcado con el encabezado correcto, también deben cumplir el [esquema de mensajes de Defender para IoT](https://aka.ms/iot-security-schemas).

### <a name="send-security-message-api"></a>Send Security Message API

**Send Security Message** API está disponible actualmente en C y C#, Python, Node.js y Java.

#### <a name="c-api"></a>API de C

```c
bool SendMessageAsync(IoTHubAdapter* iotHubAdapter, const void* data, size_t dataSize) {

    bool success = true;
    IOTHUB_MESSAGE_HANDLE messageHandle = NULL;

    messageHandle = IoTHubMessage_CreateFromByteArray(data, dataSize);

    if (messageHandle == NULL) {
        success = false;
        goto cleanup;
    }

    if (IoTHubMessage_SetAsSecurityMessage(messageHandle) != IOTHUB_MESSAGE_OK) {
        success = false;
        goto cleanup;
    }

    if (IoTHubModuleClient_SendEventAsync(iotHubAdapter->moduleHandle, messageHandle, SendConfirmCallback, iotHubAdapter) != IOTHUB_CLIENT_OK) {
        success = false;
        goto cleanup;
    }

cleanup:
    if (messageHandle != NULL) {
        IoTHubMessage_Destroy(messageHandle);
    }

    return success;
}

static void SendConfirmCallback(IOTHUB_CLIENT_CONFIRMATION_RESULT result, void* userContextCallback) {
    if (userContextCallback == NULL) {
        //error handling
        return;
    }

    if (result != IOTHUB_CLIENT_CONFIRMATION_OK){
        //error handling
    }
}
```

#### <a name="c-api"></a>API de C#

```cs

private static async Task SendSecurityMessageAsync(string messageContent)
{
    ModuleClient client = ModuleClient.CreateFromConnectionString("<connection_string>");
    Message  securityMessage = new Message(Encoding.UTF8.GetBytes(messageContent));
    securityMessage.SetAsSecurityMessage();
    await client.SendEventAsync(securityMessage);
}
```

#### <a name="nodejs-api"></a>API de Node.js.

```typescript
var Protocol = require('azure-iot-device-mqtt').Mqtt

function SendSecurityMessage(messageContent)
{
  var client = Client.fromConnectionString(connectionString, Protocol);

  var connectCallback = function (err) {
    if (err) {
      console.error('Could not connect: ' + err.message);
    } else {
      var message = new Message(messageContent);
      message.setAsSecurityMessage();
      client.sendEvent(message);
  
      client.on('error', function (err) {
        console.error(err.message);
      });
  
      client.on('disconnect', function () {
        clearInterval(sendInterval);
        client.removeAllListeners();
        client.open(connectCallback);
      });
    }
  };

  client.open(connectCallback);
}
```

#### <a name="python-api"></a>API de Python

Para poder usar la API de Python, debe instalar el paquete [azure-iot-device](https://pypi.org/project/azure-iot-device/).

Cuando utiliza la API de Python, puede enviar el mensaje de seguridad por el módulo o por el dispositivo utilizando la cadena de conexión única del dispositivo o del módulo. Cuando utilice el siguiente script de ejemplo de Python, use **IoTHubDeviceClient** con un dispositivo y **IoTHubModuleClient** con un módulo.

```python
from azure.iot.device.aio import IoTHubDeviceClient, IoTHubModuleClient
from azure.iot.device import Message

async def send_security_message_async(message_content):
    conn_str = os.getenv("<connection_string>")
    device_client = IoTHubDeviceClient.create_from_connection_string(conn_str)
    await device_client.connect()
    security_message = Message(message_content)
    security_message.set_as_security_message()
    await device_client.send_message(security_message)
    await device_client.disconnect()
```

#### <a name="java-api"></a>API de Java

```java
public void SendSecurityMessage(string message)
{
    ModuleClient client = new ModuleClient("<connection_string>", IotHubClientProtocol.MQTT);
    Message msg = new Message(message);
    msg.setAsSecurityMessage();
    EventCallback callback = new EventCallback();
    string context = "<user_context>";
    client.sendEventAsync(msg, callback, context);
}
```

## <a name="next-steps"></a>Pasos siguientes

- Leer la [introducción](overview.md) a Defender para IoT
- Obtener más información sobre la [arquitectura](architecture.md) de Defender para IoT
- Habilite el [servicio](quickstart-onboard-iot-hub.md)
- Lea las [preguntas más frecuentes](resources-frequently-asked-questions.md)
- Aprenda a acceder a [datos de seguridad sin procesar](how-to-security-data-access.md)
- Obtenga información acerca de las [recomendaciones](concept-recommendations.md)
- Obtenga información acerca de las [alertas](concept-security-alerts.md)
