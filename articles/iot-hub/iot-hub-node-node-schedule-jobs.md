---
title: Cómo programar trabajos con IoT Hub de Azure | Microsoft Docs
description: Cómo programar un trabajo de IoT Hub de Azure para invocar un método directo en varios dispositivos. El SDK de IoT de Azure para Node.js se usa para implementar la aplicación de dispositivo simulado y una aplicación de servicio para ejecutar el trabajo.
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.devlang: nodejs
ms.topic: conceptual
ms.date: 08/16/2019
ms.custom: mqtt, devx-track-javascript
ms.openlocfilehash: 16e6c72f9fbb710524752c2d220d442735d809b7
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87420777"
---
# <a name="schedule-and-broadcast-jobs-nodejs"></a>Programación y difusión de trabajos (Node.js)

[!INCLUDE [iot-hub-selector-schedule-jobs](../../includes/iot-hub-selector-schedule-jobs.md)]

IoT Hub de Azure es un servicio completamente administrado que permite a una aplicación back-end crear y realizar un seguimiento de los trabajos que programan y actualizan millones de dispositivos.  Los trabajos pueden utilizarse para las siguientes acciones:

* Actualizar las propiedades deseadas
* Actualizar etiquetas
* Invocar métodos directos

Conceptualmente, un trabajo contiene una de estas acciones y realiza un seguimiento del progreso de ejecución en un conjunto de dispositivos, que define una consulta de dispositivo gemelo.  Por ejemplo, una aplicación back-end puede usar un trabajo para invocar un método de reinicio en 10 000 dispositivos, especificado por una consulta de dispositivos gemelos y programada en el futuro. Esa aplicación puede después seguir el progreso cuando cada uno de estos dispositivos reciben y ejecutan el método de reinicio.

Más información sobre estas funcionalidades en estos artículos:

* Dispositivo gemelo y propiedades: [Introducción a los dispositivos gemelos](iot-hub-node-node-twin-getstarted.md) y [Tutorial: uso de las propiedades de dispositivos gemelos](tutorial-device-twins.md)

* Métodos directos: [Guía para desarrolladores de IoT Hub: métodos directos](iot-hub-devguide-direct-methods.md) y [Tutorial: Métodos directos](quickstart-control-device-node.md)

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

En este tutorial se muestra cómo realizar las siguientes acciones:

* Crear una aplicación de dispositivo simulado de Node.js con un método directo que permita **lockDoor**, donde el back-end de la solución pueda llamar.

* Crear una aplicación de consola de Node.js que llame al método directo **lockDoor** en la aplicación de dispositivo simulado mediante un trabajo y actualice las propiedades deseadas mediante un trabajo de dispositivo.

Al final de este tutorial tendrá dos aplicaciones de Node.js:

* **simDevice.js**, que se conecta a IoT Hub con la identidad del dispositivo y recibe un método directo **lockDoor**.

* **scheduleJobService.js**, que llama a un método directo en la aplicación de dispositivo simulado y actualiza las propiedades deseadas del dispositivo gemelo mediante un trabajo.

## <a name="prerequisites"></a>Prerrequisitos

* Node.js versión 10.0.x o posteriores. En [Preparación del entorno de desarrollo](https://github.com/Azure/azure-iot-sdk-node/tree/master/doc/node-devbox-setup.md) se describe cómo instalar Node.js para este tutorial en Windows o Linux.

* Una cuenta de Azure activa. (En caso de no tener ninguna, puede crear una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) en tan solo unos minutos).

* Asegúrese de que el puerto 8883 está abierto en el firewall. En el ejemplo de dispositivo de este artículo se usa el protocolo MQTT, que se comunica mediante el puerto 8883. Este puerto puede estar bloqueado en algunos entornos de red corporativos y educativos. Para más información y saber cómo solucionar este problema, consulte [Conexión a IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="create-an-iot-hub"></a>Crear un centro de IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-new-device-in-the-iot-hub"></a>Registro de un nuevo dispositivo en el centro de IoT

[!INCLUDE [iot-hub-get-started-create-device-identity](../../includes/iot-hub-get-started-create-device-identity.md)]

## <a name="create-a-simulated-device-app"></a>Creación de una aplicación de dispositivo simulado

En esta sección, creará una aplicación de consola de Node.js que responda a un método directo al que la nube llama, que desencadena un método **lockDoor** simulado.

1. Cree una nueva carpeta vacía denominada **simDevice**.  En la carpeta **simDevice**, cree un archivo package.json con el siguiente comando en el símbolo del sistema.  Acepte todos los valores predeterminados:

   ```console
   npm init
   ```

2. En el símbolo del sistema, en la carpeta **simDevice**, ejecute el siguiente comando para instalar el paquete del SDK de dispositivo **azure-iot-device** y el paquete **azure-iot-device-mqtt**:

   ```console
   npm install azure-iot-device azure-iot-device-mqtt --save
   ```

3. Con un editor de texto, cree un nuevo archivo **simDevice.js** en la carpeta **simDevice**.

4. Agregue las siguientes instrucciones "require" al principio del archivo **simDevice.js**:

    ```javascript
    'use strict';

    var Client = require('azure-iot-device').Client;
    var Protocol = require('azure-iot-device-mqtt').Mqtt;
    ```

5. Agregue una variable **connectionString** y utilícela para crear una instancia de **cliente**. Sustituya el valor del marcador de posición `{yourDeviceConnectionString}` por la cadena de conexión del dispositivo que copió anteriormente.

    ```javascript
    var connectionString = '{yourDeviceConnectionString}';
    var client = Client.fromConnectionString(connectionString, Protocol);
    ```

6. Agregue la siguiente función para controlar el método **lockDoor**.

    ```javascript
    var onLockDoor = function(request, response) {

        // Respond the cloud app for the direct method
        response.send(200, function(err) {
            if (err) {
                console.error('An error occurred when sending a method response:\n' + err.toString());
            } else {
                console.log('Response to method \'' + request.methodName + '\' sent successfully.');
            }
        });

        console.log('Locking Door!');
    };
    ```

7. Agregue el código siguiente para registrar el controlador del método **lockDoor**.

   ```javascript
   client.open(function(err) {
        if (err) {
            console.error('Could not connect to IotHub client.');
        }  else {
            console.log('Client connected to IoT Hub. Register handler for lockDoor direct method.');
            client.onDeviceMethod('lockDoor', onLockDoor);
        }
   });
   ```

8. Guarde y cierre el archivo **simDevice.js**.

> [!NOTE]
> Por simplificar, este tutorial no implementa ninguna directiva de reintentos. En el código de producción, deberá implementar directivas de reintentos (por ejemplo, retroceso exponencial), tal y como se sugiere en el artículo [Control de errores transitorios](/azure/architecture/best-practices/transient-faults).
>

## <a name="get-the-iot-hub-connection-string"></a>Obtención de la cadena de conexión de IoT Hub

[!INCLUDE [iot-hub-howto-schedule-jobs-shared-access-policy-text](../../includes/iot-hub-howto-schedule-jobs-shared-access-policy-text.md)]

[!INCLUDE [iot-hub-include-find-registryrw-connection-string](../../includes/iot-hub-include-find-registryrw-connection-string.md)]

## <a name="schedule-jobs-for-calling-a-direct-method-and-updating-a-device-twins-properties"></a>Programación de trabajos para llamar un método directo y actualización de las propiedades de un dispositivo gemelo

En esta sección, creará una aplicación de consola de Node.js que inicia un **lockDook** remoto en un dispositivo con un método directo y actualiza las propiedades del dispositivo gemelo.

1. Cree una nueva carpeta vacía denominada **scheduleJobService**.  En la carpeta **scheduleJobService**, cree un archivo package.json con el siguiente comando en el símbolo del sistema.  Acepte todos los valores predeterminados:

    ```console
    npm init
    ```

2. En el símbolo del sistema, en la carpeta **scheduleJobService**, ejecute el siguiente comando para instalar el paquete del SDK de dispositivo **azure-iothub** y el paquete **azure-iot-device-mqtt**:

    ```console
    npm install azure-iothub uuid --save
    ```

3. Con un editor de texto, cree un nuevo archivo **scheduleJobService.js** en la carpeta **scheduleJobService**.

4. Agregue las siguientes instrucciones "require" al principio del archivo **scheduleJobService.js**:

    ```javascript
    'use strict';

    var uuid = require('uuid');
    var JobClient = require('azure-iothub').JobClient;
    ```

5. Agregue las siguientes declaraciones de variable. Reemplace el valor del marcador de posición `{iothubconnectionstring}` por el valor que copió en [Obtención de la cadena de conexión de IoT Hub](#get-the-iot-hub-connection-string). Si registró un dispositivo distinto de **myDeviceId**, asegúrese de cambiarlo en la condición de consulta.

    ```javascript
    var connectionString = '{iothubconnectionstring}';
    var queryCondition = "deviceId IN ['myDeviceId']";
    var startTime = new Date();
    var maxExecutionTimeInSeconds =  300;
    var jobClient = JobClient.fromConnectionString(connectionString);
    ```

6. Agregue la función siguiente que se utiliza para supervisar la ejecución del trabajo:

    ```javascript
    function monitorJob (jobId, callback) {
        var jobMonitorInterval = setInterval(function() {
            jobClient.getJob(jobId, function(err, result) {
            if (err) {
                console.error('Could not get job status: ' + err.message);
            } else {
                console.log('Job: ' + jobId + ' - status: ' + result.status);
                if (result.status === 'completed' || result.status === 'failed' || result.status === 'cancelled') {
                clearInterval(jobMonitorInterval);
                callback(null, result);
                }
            }
            });
        }, 5000);
    }
    ```

7. Agregue el código siguiente para programar el trabajo que llama al método de dispositivo:
  
    ```javascript
    var methodParams = {
        methodName: 'lockDoor',
        payload: null,
        responseTimeoutInSeconds: 15 // Timeout after 15 seconds if device is unable to process method
    };

    var methodJobId = uuid.v4();
    console.log('scheduling Device Method job with id: ' + methodJobId);
    jobClient.scheduleDeviceMethod(methodJobId,
                                queryCondition,
                                methodParams,
                                startTime,
                                maxExecutionTimeInSeconds,
                                function(err) {
        if (err) {
            console.error('Could not schedule device method job: ' + err.message);
        } else {
            monitorJob(methodJobId, function(err, result) {
                if (err) {
                    console.error('Could not monitor device method job: ' + err.message);
                } else {
                    console.log(JSON.stringify(result, null, 2));
                }
            });
        }
    });
    ```

8. Agregue el código siguiente para programar el trabajo para que actualice el dispositivo gemelo:

    ```javascript
    var twinPatch = {
       etag: '*',
       properties: {
           desired: {
               building: '43',
               floor: 3
           }
       }
    };

    var twinJobId = uuid.v4();

    console.log('scheduling Twin Update job with id: ' + twinJobId);
    jobClient.scheduleTwinUpdate(twinJobId,
                                queryCondition,
                                twinPatch,
                                startTime,
                                maxExecutionTimeInSeconds,
                                function(err) {
        if (err) {
            console.error('Could not schedule twin update job: ' + err.message);
        } else {
            monitorJob(twinJobId, function(err, result) {
                if (err) {
                    console.error('Could not monitor twin update job: ' + err.message);
                } else {
                    console.log(JSON.stringify(result, null, 2));
                }
            });
        }
    });
    ```

9. Guarde y cierre el archivo **scheduleJobService.js**.

## <a name="run-the-applications"></a>Ejecución de las aplicaciones

Ahora está preparado para ejecutar las aplicaciones.

1. En el símbolo del sistema dentro de la carpeta **simDevice**, ejecute el siguiente comando para iniciar la escucha del método directo de reinicio.

    ```console
    node simDevice.js
    ```

2. En el símbolo del sistema de la carpeta **scheduleJobService**, ejecute el siguiente comando para desencadenar los trabajos a fin de bloquear la puerta y actualizar el dispositivo gemelo.

    ```console
    node scheduleJobService.js
    ```

3. Verá la respuesta del dispositivo al método directo y el estado del trabajo en la consola.

   A continuación, puede ver la respuesta del dispositivo al método directo:

   ![Salida de una aplicación de dispositivo simulado](./media/iot-hub-node-node-schedule-jobs/sim-device.png)

   A continuación se muestran los trabajos de programación del servicio para el método directo, la actualización del dispositivo gemelo y los trabajos que se ejecutan hasta su finalización:

   ![Ejecución de una aplicación de dispositivo simulada](./media/iot-hub-node-node-schedule-jobs/schedule-job-service.png)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha utilizado un trabajo para programar un método directo para un dispositivo y la actualización de las propiedades del dispositivo gemelo.

Para continuar con la introducción de IoT Hub y los patrones de administración de dispositivos, como la actualización de firmware remota de forma inalámbrica, consulte [Tutorial: cómo realizar una actualización de firmware](tutorial-firmware-update.md).

Para continuar con la introducción a IoT Hub, consulte [Introducción a Azure IoT Edge](../iot-edge/tutorial-simulate-device-linux.md).
