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
ms.openlocfilehash: e2c3c3988193242cd0afe0135b019c7e6f73b59c
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65596722"
---
# <a name="send-cloud-to-device-messages-with-iot-hub-node"></a>Envío de mensajes de nube a dispositivo con IoT Hub (Node)
[!INCLUDE [iot-hub-selector-c2d](../../includes/iot-hub-selector-c2d.md)]

## <a name="introduction"></a>Introducción
IoT Hub de Azure es un servicio totalmente administrado que permite la comunicación bidireccional confiable y segura entre millones de dispositivos y una solución de back-end. El tutorial [Introducción a Iot Hub](quickstart-send-telemetry-node.md) muestra cómo crear un centro de IoT, aprovisionar la identidad de un dispositivo en él y codificar una aplicación de dispositivo simulado que envía mensajes de dispositivo a nube.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

Este tutorial se basa en la [introducción al Centro de IoT](quickstart-send-telemetry-node.md). En él se muestra cómo realizar las siguientes acciones:

* Desde el back-end de la nube de la aplicación, envíe mensajes de nube a dispositivo en un único dispositivo a través de IoT Hub.
* Reciba mensajes de nube a dispositivo en un dispositivo.
* Desde el back-end de la nube de la aplicación, solicite confirmación de entrega (*comentarios*) para los mensajes enviados a un dispositivo desde IoT Hub.

Puede encontrar más información sobre los mensajes de nube a dispositivo en el [Guía del desarrollador de IoT Hub](iot-hub-devguide-messaging.md).

Al final de este tutorial, ejecutará dos aplicaciones de consola de Node.js:

* **SimulatedDevice**, versión modificada de la aplicación creada en [Introducción al Centro de IoT](quickstart-send-telemetry-node.md), que se conecta al Centro de IoT y recibe mensajes de nube a dispositivo.

* **SendCloudToDeviceMessage**, que envía un mensaje de la nube a la aplicación de dispositivo simulado mediante IoT Hub y, luego, recibe su confirmación de entrega.

> [!NOTE]
> El Centro de IoT ofrece compatibilidad con el SDK en muchas plataformas de dispositivos y lenguajes (incluido C, Java y Javascript), mediante los SDK de dispositivos IoT de Azure. Visite el [Centro para desarrolladores de IoT de Azure](https://azure.microsoft.com/develop/iot)para obtener instrucciones paso a paso sobre cómo conectar el dispositivo al código de este tutorial y, en general, al Centro de IoT de Azure.
>

Para completar este tutorial, necesitará lo siguiente:

* Node.js versión 10.0 o posterior.
* Una cuenta de Azure activa. (En caso de no tener ninguna, puede crear una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial) en tan solo unos minutos).

## <a name="receive-messages-in-the-simulated-device-app"></a>Recepción de mensajes en la aplicación de dispositivo simulado

En esta sección, modificará la aplicación de dispositivo simulado que creó en el tutorial [Introducción a IoT Hub](quickstart-send-telemetry-node.md) para recibir mensajes de nube a dispositivo desde el centro de IoT.

1. Con un editor de texto, abra el archivo SimulatedDevice.js.

2. Modifique la función **connectCallback** para controlar los mensajes enviados desde IoT Hub. En este ejemplo, el dispositivo siempre invoca la función **complete** para notificar a IoT Hub que ha procesado el mensaje. La nueva versión de la función **connectCallback** tiene un aspecto similar al fragmento siguiente:
   
    ```javascript
    var connectCallback = function (err) {
      if (err) {
        console.log('Could not connect: ' + err);
      } else {
        console.log('Client connected');
        client.on('message', function (msg) {
          console.log('Id: ' + msg.messageId + ' Body: ' + msg.data);
          client.complete(msg, printResultFor('completed'));
        });
        // Create a message and send it to the IoT Hub every second
        setInterval(function(){
            var temperature = 20 + (Math.random() * 15);
            var humidity = 60 + (Math.random() * 20);
            var data = JSON.stringify({ deviceId: 'myFirstNodeDevice', temperature: temperature, humidity: humidity });
            var message = new Message(data);
            message.properties.add('temperatureAlert', (temperature > 30) ? 'true' : 'false');
            console.log("Sending message: " + message.getData());
            client.sendEvent(message, printResultFor('send'));
        }, 1000);
      }
    };
    ```
  
   > [!NOTE]
   > Si usa HTTPS en lugar de MQTT o AMQP como transporte, la instancia de **DeviceClient** busca mensajes de IoT Hub con menos frecuencia (menos de 25 minutos). Para obtener más información sobre las diferencias entre la compatibilidad con MQTT, AMQP y HTTPS y la limitación de IoT Hub, consulte el [Guía del desarrollador de IoT Hub](iot-hub-devguide-messaging.md).
   >

## <a name="send-a-cloud-to-device-message"></a>Envío de mensajes de nube a dispositivo

En esta sección, usted crea una aplicación de consola de Node.js que envía mensajes de nube a dispositivo a la aplicación del dispositivo simulado. Necesita el identificador de dispositivo que agregó en el tutorial de [Introducción a Iot Hub](quickstart-send-telemetry-node.md) También necesita la cadena de conexión para IoT Hub que encontrará en [Azure Portal](https://portal.azure.com).

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

5. Agregue el código siguiente al archivo **SendCloudToDeviceMessage.js** . Sustituya el valor de marcador de posición "{iot hub connection string}" por la cadena de conexión de IoT Hub que creó en el tutorial [Introducción a IoT Hub](quickstart-send-telemetry-node.md). Sustituya el marcador de posición "{device id}" por el identificador del dispositivo que agregó en el tutorial [Introducción a IoT Hub](quickstart-send-telemetry-node.md):
   
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

1. En el símbolo del sistema de la carpeta **simulateddevice** , ejecute el comando siguiente para enviar la telemetría al centro de IoT Hub y escuchar mensajes de nube al dispositivo:
   
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