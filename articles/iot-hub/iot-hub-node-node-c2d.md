---
title: Mensajes de nube a dispositivo con IoT Hub de Azure (Node) | Microsoft Docs
description: Cómo enviar mensajes de nube a un dispositivo de una instancia de IoT Hub de Azure mediante los SDK de IoT de Azure para Node.js. Modifique una aplicación de dispositivo simulado para recibir mensajes de nube a dispositivo y cambie una aplicación de back-end para enviar los mensajes de nube a dispositivo.
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.devlang: javascript
ms.topic: conceptual
ms.date: 06/16/2017
ms.custom:
- amqp
- mqtt
- devx-track-javascript
ms.openlocfilehash: 2d5baf0f1c4298d597b620f02f3e463a05203f5a
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87424041"
---
# <a name="send-cloud-to-device-messages-with-iot-hub-nodejs"></a>Envío de mensajes de nube a dispositivo con IoT Hub (Node.js)

[!INCLUDE [iot-hub-selector-c2d](../../includes/iot-hub-selector-c2d.md)]

IoT Hub de Azure es un servicio totalmente administrado que permite la comunicación bidireccional confiable y segura entre millones de dispositivos y una solución de back-end. En el inicio rápido [Envío de telemetría desde un dispositivo a un centro de IoT](quickstart-send-telemetry-node.md) se muestra cómo crear un centro de IoT, aprovisionar en él la identidad del dispositivo y codificar una aplicación de dispositivo simulado que envíe mensajes del dispositivo a la nube.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

Este tutorial se basa en el artículo [Envío de telemetría desde un dispositivo a un centro de IoT](quickstart-send-telemetry-node.md). En él se muestra cómo realizar las siguientes acciones:

* Desde el back-end de la nube de la aplicación, envíe mensajes de nube a dispositivo en un único dispositivo a través de IoT Hub.
* Reciba mensajes de nube a dispositivo en un dispositivo.
* Desde la solución de back-end, solicite confirmación de entrega (*comentarios*) para los mensajes enviados a un dispositivo desde IoT Hub.

Encontrará más información sobre los mensajes de nube a dispositivo en la [Guía para desarrolladores de IoT Hub](iot-hub-devguide-messaging.md).

Al final de este tutorial, ejecutará dos aplicaciones de consola de Node.js:

* **SimulatedDevice**, una versión modificada de la aplicación que se creó en [Envío de telemetría desde un dispositivo a un centro de IoT](quickstart-send-telemetry-node.md) que se conecta al centro de IoT y recibe mensajes de la nube al dispositivo.

* **SendCloudToDeviceMessage**, que envía un mensaje de la nube a la aplicación de dispositivo simulado mediante IoT Hub y recibe su confirmación de entrega.

> [!NOTE]
> IoT Hub ofrece compatibilidad con SDK en muchas plataformas de dispositivos y lenguajes (entre los que se incluyen C, Java, Python y Javascript), mediante los SDK de dispositivo IoT de Azure. Visite el [Centro para desarrolladores de IoT de Azure](https://azure.microsoft.com/develop/iot)para obtener instrucciones paso a paso sobre cómo conectar el dispositivo al código de este tutorial y, en general, al Centro de IoT de Azure.
>

## <a name="prerequisites"></a>Requisitos previos

* Node.js versión 10.0.x o posteriores. En [Preparación del entorno de desarrollo](https://github.com/Azure/azure-iot-sdk-node/tree/master/doc/node-devbox-setup.md) se describe cómo instalar Node.js para este tutorial en Windows o Linux.

* Una cuenta de Azure activa. (En caso de no tener ninguna, puede crear una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial) en tan solo unos minutos).

* Asegúrese de que el puerto 8883 está abierto en el firewall. En el ejemplo de dispositivo de este artículo se usa el protocolo MQTT, que se comunica mediante el puerto 8883. Este puerto puede estar bloqueado en algunos entornos de red corporativos y educativos. Para más información y saber cómo solucionar este problema, consulte [Conexión a IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="receive-messages-in-the-simulated-device-app"></a>Recepción de mensajes en la aplicación de dispositivo simulado

En esta sección, modificará la aplicación de dispositivo simulado que creó en [Envío de telemetría desde un dispositivo a un centro de IoT](quickstart-send-telemetry-node.md) para recibir mensajes de la nube al dispositivo desde el centro de IoT.

1. Mediante un editor de texto, abra el archivo **SimulatedDevice.js**. Este archivo se encuentra en la carpeta **iot-hub\Quickstarts\simulated-device** folder, fuera de la carpeta raíz, del código de ejemplo de Node.js que descargó en el inicio rápido [Envío de telemetría desde un dispositivo a un centro de IoT](quickstart-send-telemetry-node.md).

2. Registre un controlador con el cliente del dispositivo para recibir los mensajes enviados desde IoT Hub. Agregue la llamada a `client.on` inmediatamente después de la línea que crea el cliente del dispositivo como en el fragmento de código siguiente:

    ```javascript
    var client = DeviceClient.fromConnectionString(connectionString, Mqtt);

    client.on('message', function (msg) {
      console.log('Id: ' + msg.messageId + ' Body: ' + msg.data);
      client.complete(msg, function (err) {
        if (err) {
          console.error('complete error: ' + err.toString());
        } else {
          console.log('complete sent');
        }
      });
    });
    ```

    En este ejemplo, el dispositivo invoca la función **complete** para notificar a IoT Hub que ha procesado el mensaje. La llamada a **complete** no es necesaria si se usa el transporte MQTT y se puede omitir. Se requiere para HTTPS y AMQP.
  
   > [!NOTE]
   > Si usa HTTPS en lugar de MQTT o AMQP como transporte, la instancia de **DeviceClient** busca mensajes de IoT Hub con menos frecuencia (menos de 25 minutos). Para más información sobre las diferencias entre la compatibilidad con MQTT, AMQP y HTTPS, y la limitación de IoT Hub, vea la [Guía del desarrollador de IoT Hub](iot-hub-devguide-messaging.md).
   >

## <a name="get-the-iot-hub-connection-string"></a>Obtención de la cadena de conexión de IoT Hub

En este artículo, se crea un servicio de back-end para enviar mensajes de la nube a un dispositivo a través de la instancia de IoT Hub que creó en [Envío de telemetría desde un dispositivo a un centro de IoT](quickstart-send-telemetry-node.md). Para enviar mensajes de nube a un dispositivo, el servicio necesita el permiso de **conexión de servicio**. De forma predeterminada, todas las instancias de IoT Hub se crean con una directiva de acceso compartido denominada **servicio** que concede este permiso.

[!INCLUDE [iot-hub-include-find-service-connection-string](../../includes/iot-hub-include-find-service-connection-string.md)]

## <a name="send-a-cloud-to-device-message"></a>Envío de mensajes de nube a dispositivo

En esta sección, usted crea una aplicación de consola de Node.js que envía mensajes de nube a dispositivo a la aplicación del dispositivo simulado. Necesitará el identificador del dispositivo que agregó en el inicio rápido [Envío de telemetría desde un dispositivo a un centro de IoT](quickstart-send-telemetry-node.md). También necesitará la cadena de conexión de IoT Hub que copió anteriormente en [Obtención de la cadena de conexión de IoT Hub](#get-the-iot-hub-connection-string).

1. Cree una carpeta vacía denominada **sendcloudtodevicemessage**. En la carpeta **sendcloudtodevicemessage** , cree un archivo package.json con el siguiente comando en el símbolo del sistema. Acepte todos los valores predeterminados:

    ```shell
    npm init
    ```

2. En el símbolo del sistema en la carpeta **sendcloudtodevicemessage**, ejecute el siguiente comando para instalar el paquete **azure-iothub**:

    ```shell
    npm install azure-iothub --save
    ```

3. Con un editor de texto, cree un archivo **SendCloudToDeviceMessage.js** en la carpeta **sendcloudtodevicemessage**.

4. Agregue las siguientes instrucciones `require` al principio del archivo **SendCloudToDeviceMessage.js** :

    ```javascript
    'use strict';

    var Client = require('azure-iothub').Client;
    var Message = require('azure-iot-common').Message;
    ```

5. Agregue el código siguiente al archivo **SendCloudToDeviceMessage.js** . Reemplace los valores de marcador de posición "{iot hub connection string}" y "{device id}" por la cadena de conexión de IoT Hub y el identificador de dispositivo que anotó anteriormente:

    ```javascript
    var connectionString = '{iot hub connection string}';
    var targetDevice = '{device id}';

    var serviceClient = Client.fromConnectionString(connectionString);
    ```

6. Agregue la función siguiente para imprimir los resultados de la operación en la consola:

    ```javascript
    function printResultFor(op) {
      return function printResult(err, res) {
        if (err) console.log(op + ' error: ' + err.toString());
        if (res) console.log(op + ' status: ' + res.constructor.name);
      };
    }
    ```

7. Agregue la función siguiente para imprimir mensajes de comentarios de entrega en la consola:

    ```javascript
    function receiveFeedback(err, receiver){
      receiver.on('message', function (msg) {
        console.log('Feedback message:')
        console.log(msg.getData().toString('utf-8'));
      });
    }
    ```

8. Agregue el código siguiente para enviar un mensaje a su dispositivo y procesar el mensaje de comentarios cuando el dispositivo reconozca el mensaje de la nube al dispositivo:

    ```javascript
    serviceClient.open(function (err) {
      if (err) {
        console.error('Could not connect: ' + err.message);
      } else {
        console.log('Service client connected');
        serviceClient.getFeedbackReceiver(receiveFeedback);
        var message = new Message('Cloud to device message.');
        message.ack = 'full';
        message.messageId = "My Message ID";
        console.log('Sending message: ' + message.getData());
        serviceClient.send(targetDevice, message, printResultFor('send'));
      }
    });
    ```

9. Guarde y cierre el archivo **SendCloudToDeviceMessage.js** .

## <a name="run-the-applications"></a>Ejecución de las aplicaciones

Ahora está preparado para ejecutar las aplicaciones.

1. En el símbolo del sistema de la carpeta **simulated-device**, ejecute el comando siguiente para enviar la telemetría a IoT Hub y escuchar mensajes de la nube al dispositivo:

    ```shell
    node SimulatedDevice.js
    ```

    ![Ejecución de una aplicación de dispositivo simulada](./media/iot-hub-node-node-c2d/receivec2d.png)

2. En un símbolo del sistema de la carpeta **sendcloudtodevicemessage** , ejecute el comando siguiente para enviar un mensaje de nube al dispositivo y espere el comentario de confirmación:

    ```shell
    node SendCloudToDeviceMessage.js
    ```

    ![Ejecución de la aplicación para enviar el comando de nube a dispositivo](./media/iot-hub-node-node-c2d/sendc2d.png)

   > [!NOTE]
   > Por simplificar, este tutorial no implementa ninguna directiva de reintentos. En el código de producción, deberá implementar directivas de reintentos (por ejemplo, retroceso exponencial), tal y como se sugiere en el artículo [Control de errores transitorios](/azure/architecture/best-practices/transient-faults).
   >

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, aprendió a enviar y recibir mensajes de nube a dispositivo.

Para ver ejemplos de soluciones de un extremo a otro que usen IoT Hub, vea el [Acelerador de la solución de supervisión remota de Azure IoT](https://azure.microsoft.com/documentation/suites/iot-suite/).

Para obtener más información sobre cómo desarrollar soluciones con IoT Hub, consulte la [Guía para desarrolladores de IoT Hub](iot-hub-devguide.md).
