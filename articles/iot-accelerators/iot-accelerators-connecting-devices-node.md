---
title: Aprovisionamiento de dispositivos para la supervisión remota en Node.js con Azure | Microsoft Docs
description: Se describe cómo conectar un dispositivo al acelerador de la solución de supervisión remota mediante una aplicación creada en Node.js.
author: dominicbetts
manager: timlt
ms.service: iot-accelerators
services: iot-accelerators
ms.topic: conceptual
ms.date: 01/24/2018
ms.author: dobett
ms.custom: mqtt, devx-track-javascript
ms.openlocfilehash: ab697778e00def1e23a3124b14458f1acb994041
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87423361"
---
# <a name="connect-your-device-to-the-remote-monitoring-solution-accelerator-nodejs"></a>Conexión del dispositivo al acelerador de soluciones de supervisión remota (Node.js)

[!INCLUDE [iot-suite-selector-connecting](../../includes/iot-suite-selector-connecting.md)]

En este tutorial se muestra cómo conectar un dispositivo real al acelerador de soluciones de supervisión remota. En este tutorial, se usa Node.js, que es una buena opción para entornos con las restricciones de recursos mínimos.

Si prefiere simular un dispositivo, consulte [Crear y probar un nuevo dispositivo simulado](iot-accelerators-remote-monitoring-create-simulated-device.md).

## <a name="create-a-nodejs-solution"></a>Creación de una solución Node.js

Asegúrese de que tiene instalada la versión 4.0.0 o posterior de [Node.js](https://nodejs.org/) en la máquina de desarrollo. Para comprobar la versión, puede ejecutar `node --version` en la línea de comandos.

1. Cree una carpeta denominada `remotemonitoring` en la máquina de desarrollo. Vaya a esta carpeta en el entorno de línea de comandos.

1. Para descargar e instalar los paquetes que necesita para realizar la aplicación de ejemplo, ejecute los comandos siguientes:

    ```cmd/sh
    npm init
    npm install async azure-iot-device azure-iot-device-mqtt --save
    ```

1. En la carpeta `remotemonitoring`, cree un archivo llamado **remote_monitoring.js**. Abra este archivo en un editor de texto.

1. En el archivo **remote_monitoring.js**, agregue las siguientes instrucciones `require`:

    ```javascript
    var Protocol = require('azure-iot-device-mqtt').Mqtt;
    var Client = require('azure-iot-device').Client;
    var Message = require('azure-iot-device').Message;
    var async = require('async');
    ```

1. Agregue las siguientes declaraciones de variable después de las instrucciones `require` . Sustituya el valor del marcador de posición `{device connection string}` por el valor que ha anotado para el dispositivo que ha aprovisionado en la solución de supervisión remota:

    ```javascript
    var connectionString = '{device connection string}';
    ```

1. Para definir algunos datos de telemetría básicos, agregue las siguientes variables:

    ```javascript
    var temperature = 50;
    var temperatureUnit = 'F';
    var humidity = 50;
    var humidityUnit = '%';
    var pressure = 55;
    var pressureUnit = 'psig';
    ```

1. Para definir algunos valores de propiedades, agregue las siguientes variables:

    ```javascript
    var schema = "real-chiller;v1";
    var deviceType = "RealChiller";
    var deviceFirmware = "1.0.0";
    var deviceFirmwareUpdateStatus = "";
    var deviceLocation = "Building 44";
    var deviceLatitude = 47.638928;
    var deviceLongitude = -122.13476;
    var deviceOnline = true;
    ```

1. Agregue la siguiente variable para definir las propiedades notificadas para enviar a la solución. Estas propiedades incluyen los metadatos que se van a mostrar en la interfaz de usuario web:

    ```javascript
    var reportedProperties = {
      "SupportedMethods": "Reboot,FirmwareUpdate,EmergencyValveRelease,IncreasePressure",
      "Telemetry": {
        [schema]: ""
      },
      "Type": deviceType,
      "Firmware": deviceFirmware,
      "FirmwareUpdateStatus": deviceFirmwareUpdateStatus,
      "Location": deviceLocation,
      "Latitude": deviceLatitude,
      "Longitude": deviceLongitude,
      "Online": deviceOnline
    }
    ```

1. Para imprimir los resultados de la operación, agregue la función auxiliar siguiente:

    ```javascript
    function printErrorFor(op) {
        return function printError(err) {
            if (err) console.log(op + ' error: ' + err.toString());
        };
    }
    ```

1. Agregue la siguiente función auxiliar para aleatorizar los valores de telemetría:

     ```javascript
     function generateRandomIncrement() {
         return ((Math.random() * 2) - 1);
     }
     ```

1. Agregue la siguiente función genérica para controlar las llamadas al método directo desde la solución. La función muestra información sobre el método directo que se ha invocado, pero en este ejemplo no se modifica el dispositivo en modo alguno. La solución utiliza métodos directos para actuar en los dispositivos:

     ```javascript
     function onDirectMethod(request, response) {
       // Implement logic asynchronously here.
       console.log('Simulated ' + request.methodName);

       // Complete the response
       response.send(200, request.methodName + ' was called on the device', function (err) {
         if (err) console.error('Error sending method response :\n' + err.toString());
         else console.log('200 Response to method \'' + request.methodName + '\' sent successfully.');
       });
     }
     ```

1. Agregue la siguiente función para controlar las llamadas al método directo **FirmwareUpdate** desde la solución. La función verifica los parámetros pasados en la carga del método directo y, a continuación, ejecuta de forma asincrónica una simulación de actualización de firmware:

     ```javascript
     function onFirmwareUpdate(request, response) {
       // Get the requested firmware version from the JSON request body
       var firmwareVersion = request.payload.Firmware;
       var firmwareUri = request.payload.FirmwareUri;
      
       // Ensure we got a firmware values
       if (!firmwareVersion || !firmwareUri) {
         response.send(400, 'Missing firmware value', function(err) {
           if (err) console.error('Error sending method response :\n' + err.toString());
           else console.log('400 Response to method \'' + request.methodName + '\' sent successfully.');
         });
       } else {
         // Respond the cloud app for the device method
         response.send(200, 'Firmware update started.', function(err) {
           if (err) console.error('Error sending method response :\n' + err.toString());
           else {
             console.log('200 Response to method \'' + request.methodName + '\' sent successfully.');

             // Run the simulated firmware update flow
             runFirmwareUpdateFlow(firmwareVersion, firmwareUri);
           }
         });
       }
     }
     ```

1. Agregue la siguiente función para simular un flujo de actualización de firmware de larga duración que informe del progreso a la solución:

     ```javascript
     // Simulated firmwareUpdate flow
     function runFirmwareUpdateFlow(firmwareVersion, firmwareUri) {
       console.log('Simulating firmware update flow...');
       console.log('> Firmware version passed: ' + firmwareVersion);
       console.log('> Firmware URI passed: ' + firmwareUri);
       async.waterfall([
         function (callback) {
           console.log("Image downloading from " + firmwareUri);
           var patch = {
             FirmwareUpdateStatus: 'Downloading image..'
           };
           reportUpdateThroughTwin(patch, callback);
           sleep(10000, callback);
         },
         function (callback) {
           console.log("Downloaded, applying firmware " + firmwareVersion);
           deviceOnline = false;
           var patch = {
             FirmwareUpdateStatus: 'Applying firmware..',
             Online: false
           };
           reportUpdateThroughTwin(patch, callback);
           sleep(8000, callback);
         },
         function (callback) {
           console.log("Rebooting");
           var patch = {
             FirmwareUpdateStatus: 'Rebooting..'
           };
           reportUpdateThroughTwin(patch, callback);
           sleep(10000, callback);
         },
         function (callback) {
           console.log("Firmware updated to " + firmwareVersion);
           deviceOnline = true;
           var patch = {
             FirmwareUpdateStatus: 'Firmware updated',
             Online: true,
             Firmware: firmwareVersion
           };
           reportUpdateThroughTwin(patch, callback);
           callback(null);
         }
       ], function(err) {
         if (err) {
           console.error('Error in simulated firmware update flow: ' + err.message);
         } else {
           console.log("Completed simulated firmware update flow");
         }
       });

       // Helper function to update the twin reported properties.
       function reportUpdateThroughTwin(patch, callback) {
         console.log("Sending...");
         console.log(JSON.stringify(patch, null, 2));
         client.getTwin(function(err, twin) {
           if (!err) {
             twin.properties.reported.update(patch, function(err) {
               if (err) callback(err);
             });      
           } else {
             if (err) callback(err);
           }
         });
       }

       function sleep(milliseconds, callback) {
         console.log("Simulate a delay (milleseconds): " + milliseconds);
         setTimeout(function () {
           callback(null);
         }, milliseconds);
       }
     }
     ```

1. Agregue el código siguiente para enviar los datos de telemetría a la solución. La aplicación cliente agrega propiedades al mensaje para identificar el esquema del mensaje:

     ```javascript
     function sendTelemetry(data, schema) {
       if (deviceOnline) {
         var d = new Date();
         var payload = JSON.stringify(data);
         var message = new Message(payload);
         message.properties.add('iothub-creation-time-utc', d.toISOString());
         message.properties.add('iothub-message-schema', schema);

         console.log('Sending device message data:\n' + payload);
         client.sendEvent(message, printErrorFor('send event'));
       } else {
         console.log('Offline, not sending telemetry');
       }
     }
     ```

1. Agregue el código siguiente para crear una instancia de cliente:

     ```javascript
     var client = Client.fromConnectionString(connectionString, Protocol);
     ```

1. Agregue el siguiente código para:

    * Abrir la conexión.
    * Configurar un controlador para propiedades deseadas.
    * Enviar propiedades notificadas.
    * Registrar controladores para métodos directos. El ejemplo utiliza un controlador independiente para el método directo de actualización de firmware.
    * Empezar a enviar telemetría.

      ```javascript
      client.open(function (err) {
      if (err) {
        printErrorFor('open')(err);
      } else {
        // Create device Twin
        client.getTwin(function (err, twin) {
          if (err) {
            console.error('Could not get device twin');
          } else {
            console.log('Device twin created');

            twin.on('properties.desired', function (delta) {
              // Handle desired properties set by solution
              console.log('Received new desired properties:');
              console.log(JSON.stringify(delta));
            });

            // Send reported properties
            twin.properties.reported.update(reportedProperties, function (err) {
              if (err) throw err;
              console.log('Twin state reported');
            });

            // Register handlers for all the method names we are interested in.
            // Consider separate handlers for each method.
            client.onDeviceMethod('Reboot', onDirectMethod);
            client.onDeviceMethod('FirmwareUpdate', onFirmwareUpdate);
            client.onDeviceMethod('EmergencyValveRelease', onDirectMethod);
            client.onDeviceMethod('IncreasePressure', onDirectMethod);
          }
        });

        // Start sending telemetry
        var sendDeviceTelemetry = setInterval(function () {
          temperature += generateRandomIncrement();
          pressure += generateRandomIncrement();
          humidity += generateRandomIncrement();
          var data = {
            'temperature': temperature,
            'temperature_unit': temperatureUnit,
            'humidity': humidity,
            'humidity_unit': humidityUnit,
            'pressure': pressure,
            'pressure_unit': pressureUnit
          };
          sendTelemetry(data, schema)
        }, 5000);

        client.on('error', function (err) {
          printErrorFor('client')(err);
          if (sendTemperatureInterval) clearInterval(sendTemperatureInterval);
          if (sendHumidityInterval) clearInterval(sendHumidityInterval);
          if (sendPressureInterval) clearInterval(sendPressureInterval);
          client.close(printErrorFor('client.close'));
        });
      }
      });
      ```

1. Guarde los cambios al archivo **remote_monitoring.js**.

1. Ejecute el siguiente comando en un símbolo del sistema para iniciar la aplicación de ejemplo:

     ```cmd/sh
     node remote_monitoring.js
     ```

[!INCLUDE [iot-suite-visualize-connecting](../../includes/iot-suite-visualize-connecting.md)]
