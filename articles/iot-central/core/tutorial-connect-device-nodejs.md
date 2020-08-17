---
title: 'Tutorial: Conexión de una aplicación cliente Node.js genérica a Azure IoT Central | Microsoft Docs'
description: En este tutorial se muestra cómo los desarrolladores de dispositivos pueden conectar un dispositivo que ejecuta una aplicación cliente Node.js a la aplicación de Azure IoT Central. Para crear una plantilla de dispositivo, importe un modelo de funcionalidad de dispositivo y agregue vistas que permitan interactuar con un dispositivo conectado
author: dominicbetts
ms.author: dobett
ms.date: 07/07/2020
ms.topic: tutorial
ms.service: iot-central
services: iot-central
ms.custom: mqtt, devx-track-javascript
ms.openlocfilehash: 92b28b89d181de368ef1e39eb695be2e1fad6c37
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87423531"
---
# <a name="tutorial-create-and-connect-a-client-application-to-your-azure-iot-central-application-nodejs"></a>Tutorial: Creación y conexión de un aplicación cliente a una aplicación de Azure IoT Central (Node.js)

[!INCLUDE [iot-central-selector-tutorial-connect](../../../includes/iot-central-selector-tutorial-connect.md)]

*Este artículo se aplica a los generadores de soluciones y a los desarrolladores de dispositivos.*

En este tutorial se muestra cómo los desarrolladores de dispositivos pueden conectar una aplicación cliente Node.js a la aplicación de Azure IoT Central. La aplicación de Node.js simula el comportamiento de un dispositivo sensor ambiental. Para crear una _plantilla de dispositivo_ en IoT Central, se usa un _modelo de funcionalidad del dispositivo_ de muestra. Agregue vistas a la plantilla de dispositivo para permitir que un operador interactúe con un dispositivo.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Importar un modelo de funcionalidad de dispositivo para crear una plantilla de dispositivo.
> * Agregar las vistas predeterminada y personalizada a una plantilla de dispositivo.
> * Publicar una plantilla de dispositivo y agregar un dispositivo real a la aplicación de IoT Central.
> * Crear y ejecutar el código del dispositivo Node.js y ver si se conecta a la aplicación de IoT Central.
> * Ver los datos de telemetría simulados que envía el dispositivo.
> * Usar una vista para administrar las propiedades del dispositivo.
> * Llamar a comandos sincrónicos y asincrónicos para controlar el dispositivo.

## <a name="prerequisites"></a>Requisitos previos

Necesitará lo siguiente para completar los pasos de este artículo:

* Una aplicación de Azure IoT Central creada a partir de la plantilla **Custom application** (Aplicación personalizada). Para más información, consulte la [guía de inicio rápido para crear una aplicación](quick-deploy-iot-central.md). La aplicación se debe haber creado el 14/07/2020 o después.
* Una máquina de desarrollo que tenga instalada la versión 10.0.0 de [Node.js](https://nodejs.org/) o una posterior. Puede ejecutar `node --version` en la línea de comandos para comprobar la versión. En las instrucciones de este tutorial se da por hecho que se ejecuta el comando **node** en el símbolo del sistema de Windows. No obstante, puede usar Node.js en muchos otros sistemas operativos.

[!INCLUDE [iot-central-add-environmental-sensor](../../../includes/iot-central-add-environmental-sensor.md)]

### <a name="create-a-nodejs-application"></a>Creación de una aplicación Node.js

En los pasos siguientes se muestra cómo crear una aplicación cliente Node.js que se conecta al dispositivo real que se agregó a la aplicación. Esta aplicación Node.js simula el comportamiento de un dispositivo real.

1. Vaya a la carpeta `environmental-sensor` que ha creado anteriormente en el entorno de la línea de comandos.

1. Para inicializar el proyecto de Node.js e instalar las dependencias necesarias, ejecute los siguientes comandos y acepte todas las opciones predeterminadas al ejecutar `npm init`:

    ```cmd/sh
    npm init
    npm install azure-iot-device azure-iot-device-mqtt azure-iot-provisioning-device-mqtt azure-iot-security-symmetric-key --save
    ```

1. Cree un archivo llamado **environmentalSensor.js** en la carpeta `environmental-sensor`.

1. Agregue las siguientes instrucciones `require` al principio del archivo **environmentalSensor.js**:

    ```javascript
    "use strict";

    // Use the Azure IoT device SDK for devices that connect to Azure IoT Central.
    var iotHubTransport = require('azure-iot-device-mqtt').Mqtt;
    var Client = require('azure-iot-device').Client;
    var Message = require('azure-iot-device').Message;
    var ProvisioningTransport = require('azure-iot-provisioning-device-mqtt').Mqtt;
    var SymmetricKeySecurityClient = require('azure-iot-security-symmetric-key').SymmetricKeySecurityClient;
    var ProvisioningDeviceClient = require('azure-iot-provisioning-device').ProvisioningDeviceClient;
    ```

1. Agregue las siguientes declaraciones de variable al archivo:

    ```javascript
    var provisioningHost = 'global.azure-devices-provisioning.net';
    var idScope = '{your Scope ID}';
    var registrationId = '{your Device ID}';
    var symmetricKey = '{your Primary Key}';
    var provisioningSecurityClient = new SymmetricKeySecurityClient(registrationId, symmetricKey);
    var provisioningClient = ProvisioningDeviceClient.create(provisioningHost, idScope, new ProvisioningTransport(), provisioningSecurityClient);
    var hubClient;

    var targetTemperature = 0;
    var ledOn = true;
    ```

    Actualice los marcadores de posición `{your Scope ID}`, `{your Device ID}` y `{your Primary Key}` con los valores que anotó anteriormente. En este ejemplo, se inicializará `targetTemperature` a cero, pero puede usar la lectura actual del dispositivo o un valor del dispositivo gemelo.

1. Para enviar datos de telemetría simulados a la aplicación de Azure IoT Central, agregue la siguiente función al archivo:

    ```javascript
    // Send simulated device telemetry.
    function sendTelemetry() {
      var temp = targetTemperature + (Math.random() * 15);
      var humid = 70 + (Math.random() * 10);
      var data = JSON.stringify({
        temp: temp,
        humid: humid,
        });
      var message = new Message(data);
      hubClient.sendEvent(message, (err, res) => console.log(`Sent message: ${message.getData()}` +
        (err ? `; error: ${err.toString()}` : '') +
        (res ? `; status: ${res.constructor.name}` : '')));
    }
    ```

    Los nombres de los elementos de telemetría (`temp` y `humid`) deben coincidir con los nombres que se usan en la plantilla de dispositivo.

1. Para enviar las propiedades del dispositivo gemelo a la aplicación de Azure IoT Central, agregue la siguiente función al archivo:

    ```javascript
    // Send device twin reported properties.
    function sendDeviceProperties(twin, properties) {
      twin.properties.reported.update(properties, (err) => console.log(`Sent device properties: ${JSON.stringify(properties)}; ` +
        (err ? `error: ${err.toString()}` : `status: success`)));
    }
    ```

    IoT Central usa dispositivos gemelos para sincronizar valores de propiedad entre el dispositivo y la aplicación de IoT Central. Los valores de propiedades del dispositivo usan propiedades notificadas del dispositivo gemelo. Las propiedades que se pueden escribir usan propiedades notificadas y deseadas del dispositivo gemelo.

1. Para definir y controlar las propiedades con posibilidad de escritura a las que responda el dispositivo, agregue el código siguiente. El mensaje que el dispositivo envía en respuesta a la [actualización de la propiedad grabable](concepts-telemetry-properties-commands.md#writeable-property-types) debe incluir los campos `av` y `ac`. El campo `ad` es opcional:

    ```javascript
    // Add any writeable properties your device supports,
    // mapped to a function that's called when the writeable property
    // is updated in the IoT Central application.
    var writeableProperties = {
      'name': (newValue, callback) => {
          setTimeout(() => {
            callback(newValue, 'completed', 200);
          }, 1000);
      },
      'brightness': (newValue, callback) => {
        setTimeout(() => {
            callback(newValue, 'completed', 200);
        }, 5000);
      }
    };

    // Handle writeable property updates that come from IoT Central via the device twin.
    function handleWriteablePropertyUpdates(twin) {
      twin.on('properties.desired', function (desiredChange) {
        for (let setting in desiredChange) {
          if (writeableProperties[setting]) {
            console.log(`Received setting: ${setting}: ${desiredChange[setting]}`);
            writeableProperties[setting](desiredChange[setting], (newValue, status, code) => {
              var patch = {
                [setting]: {
                  value: newValue,
                  ad: status,
                  ac: code,
                  av: desiredChange.$version
                }
              }
              sendDeviceProperties(twin, patch);
            });
          }
        }
      });
    }
    ```

    Cuando el operador establece una propiedad que se puede escribir en la aplicación de IoT Central, la aplicación usa una propiedad deseada del dispositivo gemelo para enviar el valor al dispositivo. A continuación, el dispositivo responde con una propiedad notificada del dispositivo gemelo. Cuando IoT Central recibe el valor de la propiedad notificada, actualiza la vista de la propiedad con el estado **sincronizado**.

    Los nombres de las propiedades (`name` y `brightness`) deben coincidir con los nombres que se usan en la plantilla de dispositivo.

1. Agregue el código siguiente para controlar los comandos enviados desde la aplicación de IoT Central:

    ```javascript
    // Setup command handlers
    function setupCommandHandlers(twin) {

      // Handle synchronous LED blink command with request and response payload.
      function onBlink(request, response) {
        console.log('Received synchronous call to blink');
        var responsePayload = {
          status: 'Blinking LED every ' + request.payload  + ' seconds'
        }
        response.send(200, responsePayload, (err) => {
          if (err) {
            console.error('Unable to send method response: ' + err.toString());
          } else {
            console.log('Blinking LED every ' + request.payload  + ' seconds');
          }
        });
      }

      // Handle synchronous LED turn on command
      function turnOn(request, response) {
        console.log('Received synchronous call to turn on LED');
        if(!ledOn){
          console.log('Turning on the LED');
          ledOn = true;
        }
        response.send(200, (err) => {
          if (err) {
            console.error('Unable to send method response: ' + err.toString());
          }
        });
      }

      // Handle synchronous LED turn off command
      function turnOff(request, response) {
        console.log('Received synchronous call to turn off LED');
        if(ledOn){
          console.log('Turning off the LED');
          ledOn = false;
        }
        response.send(200, (err) => {
          if (err) {
            console.error('Unable to send method response: ' + err.toString());
          }
        });
      }

      // Handle asynchronous sensor diagnostics command with response payload.
      function diagnostics(request, response) {
        console.log('Starting asynchronous diagnostics run...');
        response.send(202, (err) => {
          if (err) {
            console.error('Unable to send method response: ' + err.toString());
          } else {
            var repetitions = 3;
            var intervalID = setInterval(() => {
              console.log('Generating diagnostics...');
              if (--repetitions === 0) {
                clearInterval(intervalID);
                var properties = {
                  rundiagnostics: {
                    value: 'Diagnostics run complete at ' + new Date().toLocaleString()
                  }
                };
                sendDeviceProperties(twin, properties);
              }
            }, 2000);
          }
        });
      }

      hubClient.onDeviceMethod('blink', onBlink);
      hubClient.onDeviceMethod('turnon', turnOn);
      hubClient.onDeviceMethod('turnoff', turnOff);
      hubClient.onDeviceMethod('rundiagnostics', diagnostics);
    }
    ```

    Los nombres de los comandos (`blink`, `turnon`, `turnoff` y `rundiagnostics`) deben coincidir con los nombres que se usan en la plantilla de dispositivo.

    Actualmente, IoT Central no usa el esquema de respuestas definido en el modelo de funcionalidad del dispositivo. Para un comando sincrónico, la carga útil de respuesta puede ser cualquier JSON válido. Para un comando asincrónico, el dispositivo debe devolver una respuesta 202 inmediatamente, seguida de una actualización de propiedad notificada cuando el trabajo ha finalizado. El formato de la actualización de propiedad notificada es:

    ```json
    {
      [command name] : {
        value: 'response message'
      }
    }
    ```

    Un operador puede ver la carga útil de respuesta en el historial de comandos.

1. Agregue el código siguiente para completar la conexión a Azure IoT Central y enlazar las funciones en el código de cliente:

    ```javascript
    // Handle device connection to Azure IoT Central.
    var connectCallback = (err) => {
      if (err) {
        console.log(`Device could not connect to Azure IoT Central: ${err.toString()}`);
      } else {
        console.log('Device successfully connected to Azure IoT Central');

        // Send telemetry to Azure IoT Central every 1 second.
        setInterval(sendTelemetry, 1000);

        // Get device twin from Azure IoT Central.
        hubClient.getTwin((err, twin) => {
          if (err) {
            console.log(`Error getting device twin: ${err.toString()}`);
          } else {
            // Send device properties once on device start up.
            var properties = {
              state: 'true',
              processorArchitecture: 'ARM',
              swVersion: '1.0.0'
            };
            sendDeviceProperties(twin, properties);

            handleWriteablePropertyUpdates(twin);

            setupCommandHandlers(twin);
          }
        });
      }
    };

    // Start the device (register and connect to Azure IoT Central).
    provisioningClient.register((err, result) => {
      if (err) {
        console.log('Error registering device: ' + err);
      } else {
        console.log('Registration succeeded');
        console.log('Assigned hub=' + result.assignedHub);
        console.log('DeviceId=' + result.deviceId);
        var connectionString = 'HostName=' + result.assignedHub + ';DeviceId=' + result.deviceId + ';SharedAccessKey=' + symmetricKey;
        hubClient = Client.fromConnectionString(connectionString, iotHubTransport);

        hubClient.open(connectCallback);
      }
    });
    ```

## <a name="run-your-nodejs-application"></a>Ejecución de la aplicación de Node.js

Para iniciar la aplicación cliente del dispositivo, ejecute el comando siguiente en el entorno de la línea de comandos:

```cmd/sh
node environmentalSensor.js
```

Verá que el dispositivo se conecta a la aplicación de Azure IoT Central y comienza a enviar telemetría:

![Ejecución de la aplicación cliente](media/tutorial-connect-device-nodejs/run-application.png)

[!INCLUDE [iot-central-monitor-environmental-sensor](../../../includes/iot-central-monitor-environmental-sensor.md)]

Puede ver cómo responde el dispositivo a los comandos y las actualizaciones de propiedades:

![Visualización de la aplicación cliente](media/tutorial-connect-device-nodejs/run-application-2.png)

## <a name="view-raw-data"></a>Visualización de datos sin procesar

[!INCLUDE [iot-central-monitor-environmental-sensor-raw-data](../../../includes/iot-central-monitor-environmental-sensor-raw-data.md)]

## <a name="next-steps"></a>Pasos siguientes

Como desarrollador de dispositivos, ahora que ha aprendido los aspectos básicos de cómo crear un dispositivo mediante Node.js, se recomiendan los siguientes pasos:

* Para más información sobre el rol de las plantillas de dispositivo cuando se está implementando el código del dispositivo, consulte [¿Qué son las plantillas de dispositivo?](./concepts-device-templates.md).
* Lea [Conexión a Azure IoT Central](./concepts-get-connected.md) para más información sobre cómo registrar dispositivos con IoT Central y cómo IoT Central protege las conexiones de dispositivos.

Si prefiere continuar con el conjunto de tutoriales de IoT Central y obtener más información sobre la creación de una solución de IoT Central, consulte:

> [!div class="nextstepaction"]
> [Creación de una plantilla de dispositivo de puerta de enlace](./tutorial-define-gateway-device-type.md)
