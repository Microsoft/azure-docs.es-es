---
title: Introducción a la administración de dispositivos de IoT Hub de Azure (Node) | Microsoft Docs
description: Describe cómo usar la administración de dispositivos de IoT Hub para iniciar un reinicio del dispositivo remoto. Usará los SDK de IoT de Azure para Node.js con el fin de implementar una aplicación de dispositivo simulado que incluye un método directo y un servicio que invoque el método directo.
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/20/2019
ms.custom: mqtt, devx-track-javascript
ms.openlocfilehash: 45637a093361a177de880788bb7eda69f365dad2
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87420811"
---
# <a name="get-started-with-device-management-nodejs"></a>Introducción a la administración de dispositivos (Node.js)

[!INCLUDE [iot-hub-selector-dm-getstarted](../../includes/iot-hub-selector-dm-getstarted.md)]

En este tutorial se muestra cómo realizar las siguientes acciones:

* Usar [Azure Portal](https://portal.azure.com) para crear una instancia de IoT Hub y una identidad de dispositivo en ella.

* Crear una aplicación de dispositivo simulado que contiene un método directo que reinicia ese dispositivo. Los métodos directos se invocan desde la nube.

* Crear una aplicación de consola de Node.js que llame a un método directo de reinicio en la aplicación de dispositivo simulado mediante su centro de IoT Hub.

Al final de este tutorial tendrá dos aplicaciones de consola de Node.js:

* **dmpatterns_getstarted_device.js**, que se conecta al IoT Hub con la identidad del dispositivo creada anteriormente, recibe un método directo de reinicio, simula un reinicio físico y notifica la hora del último reinicio.

* **dmpatterns_getstarted_service.js**, que llama a un método directo en la aplicación de dispositivo simulado, muestra la respuesta y muestra las propiedades notificadas actualizadas.

## <a name="prerequisites"></a>Requisitos previos

* Node.js versión 10.0.x o posteriores. En [Preparación del entorno de desarrollo](https://github.com/Azure/azure-iot-sdk-node/tree/master/doc/node-devbox-setup.md) se describe cómo instalar Node.js para este tutorial en Windows o Linux.

* Una cuenta de Azure activa. (En caso de no tener ninguna, puede crear una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) en tan solo unos minutos).

* Asegúrese de que el puerto 8883 está abierto en el firewall. En el ejemplo de dispositivo de este artículo se usa el protocolo MQTT, que se comunica mediante el puerto 8883. Este puerto puede estar bloqueado en algunos entornos de red corporativos y educativos. Para más información y saber cómo solucionar este problema, consulte [Conexión a IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="create-an-iot-hub"></a>Crear un centro de IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-new-device-in-the-iot-hub"></a>Registro de un nuevo dispositivo en el centro de IoT

[!INCLUDE [iot-hub-get-started-create-device-identity](../../includes/iot-hub-get-started-create-device-identity.md)]

## <a name="create-a-simulated-device-app"></a>Creación de una aplicación de dispositivo simulado

En esta sección:

* Creará una aplicación de consola de Node.js que responda a un método directo que se llama desde la nube.

* Desencadenará un reinicio del dispositivo simulado.

* Usará las propiedades notificadas para permitir consultas de dispositivo gemelo a fin de identificar los dispositivos y cuándo se reiniciaron por última vez.

1. Cree una nueva carpeta vacía denominada **manageddevice**.  En la carpeta **manageddevice** , cree un archivo package.json con el siguiente comando en el símbolo del sistema.  Acepte todos los valores predeterminados:

    ```cmd/sh
    npm init
    ```

2. En el símbolo del sistema, en la carpeta **manageddevice**, ejecute el siguiente comando para instalar el paquete del SDK de dispositivo **azure-iot-device** y el paquete **azure-iot-device-mqtt**:

    ```cmd/sh
    npm install azure-iot-device azure-iot-device-mqtt --save
    ```

3. Con un editor de texto, cree un archivo **dmpatterns_getstarted_device.js** en la carpeta **manageddevice**.

4. Agregue las siguientes instrucciones "require" al principio del archivo **dmpatterns_getstarted_device.js**:

    ```javascript
    'use strict';

    var Client = require('azure-iot-device').Client;
    var Protocol = require('azure-iot-device-mqtt').Mqtt;
    ```

5. Agregue una variable **connectionString** y utilícela para crear una instancia de **cliente**.  Reemplace el valor del marcador de posición `{yourdeviceconnectionstring}` por la cadena de conexión de dispositivo que copió anteriormente en [Registro de un nuevo dispositivo en el centro de IoT](#register-a-new-device-in-the-iot-hub).  

    ```javascript
    var connectionString = '{yourdeviceconnectionstring}';
    var client = Client.fromConnectionString(connectionString, Protocol);
    ```

6. Agregue la siguiente función para implementar el método directo en el dispositivo

    ```javascript
    var onReboot = function(request, response) {

        // Respond the cloud app for the direct method
        response.send(200, 'Reboot started', function(err) {
            if (err) {
                console.error('An error occurred when sending a method response:\n' + err.toString());
            } else {
                console.log('Response to method \'' + request.methodName + '\' sent successfully.');
            }
        });

        // Report the reboot before the physical restart
        var date = new Date();
        var patch = {
            iothubDM : {
                reboot : {
                    lastReboot : date.toISOString(),
                }
            }
        };

        // Get device Twin
        client.getTwin(function(err, twin) {
            if (err) {
                console.error('could not get twin');
            } else {
                console.log('twin acquired');
                twin.properties.reported.update(patch, function(err) {
                    if (err) throw err;
                    console.log('Device reboot twin state reported')
                });  
            }
        });

        // Add your device's reboot API for physical restart.
        console.log('Rebooting!');
    };
    ```

7. Abra la conexión al IoT Hub e inicie la escucha del método directo:

    ```javascript
    client.open(function(err) {
        if (err) {
            console.error('Could not open IotHub client');
        }  else {
            console.log('Client opened.  Waiting for reboot method.');
            client.onDeviceMethod('reboot', onReboot);
        }
    });
    ```

8. Guarde y cierre el archivo **dmpatterns_getstarted_device.js**.

> [!NOTE]
> Por simplificar, este tutorial no implementa ninguna directiva de reintentos. En el código de producción, deberá implementar directivas de reintentos (por ejemplo, retroceso exponencial), tal y como se sugiere en el artículo [Control de errores transitorios](/azure/architecture/best-practices/transient-faults).

## <a name="get-the-iot-hub-connection-string"></a>Obtención de la cadena de conexión de IoT Hub

[!INCLUDE [iot-hub-howto-device-management-shared-access-policy-text](../../includes/iot-hub-howto-device-management-shared-access-policy-text.md)]

[!INCLUDE [iot-hub-include-find-service-connection-string](../../includes/iot-hub-include-find-service-connection-string.md)]

## <a name="trigger-a-remote-reboot-on-the-device-using-a-direct-method"></a>Desencadenar un reinicio remoto en el dispositivo con un método directo

En esta sección, creará una aplicación de consola de Node.js que inicia un reinicio remoto en un dispositivo mediante un método directo. La aplicación usa las consultas gemelas de dispositivo para detectar la hora en que se reinició por última vez el dispositivo.

1. Cree una carpeta vacía denominada **triggerrebootondevice**. En la carpeta **triggerrebootondevice**, cree un archivo package.json con el siguiente comando en el símbolo del sistema. Acepte todos los valores predeterminados:

    ```cmd/sh
    npm init
    ```

2. En el símbolo del sistema, en la carpeta **triggerrebootondevice**, ejecute el siguiente comando para instalar el paquete del SDK de dispositivo **azure-iothub** y el paquete **azure-iot-device-mqtt**:

    ```cmd/sh
    npm install azure-iothub --save
    ```

3. Con un editor de texto, cree un archivo **dmpatterns_getstarted_service.js** en la carpeta **triggerrebootondevice**.

4. Agregue las siguientes instrucciones "require" al principio del archivo **dmpatterns_getstarted_service.js**:

    ```javascript
    'use strict';

    var Registry = require('azure-iothub').Registry;
    var Client = require('azure-iothub').Client;
    ```

5. Agregue las declaraciones de variantes siguientes y reemplace el valor del marcador de posición `{iothubconnectionstring}` por la cadena de conexión del centro de IoT que copió anteriormente en [Obtención de la cadena de conexión de IoT Hub](#get-the-iot-hub-connection-string):

    ```javascript
    var connectionString = '{iothubconnectionstring}';
    var registry = Registry.fromConnectionString(connectionString);
    var client = Client.fromConnectionString(connectionString);
    var deviceToReboot = 'myDeviceId';
    ```

6. Agregue la siguiente función para invocar el método del dispositivo con el fin de reiniciar el dispositivo de destino:

    ```javascript
    var startRebootDevice = function(twin) {

        var methodName = "reboot";

        var methodParams = {
            methodName: methodName,
            payload: null,
            timeoutInSeconds: 30
        };

        client.invokeDeviceMethod(deviceToReboot, methodParams, function(err, result) {
            if (err) {
                console.error("Direct method error: "+err.message);
            } else {
                console.log("Successfully invoked the device to reboot.");  
            }
        });
    };
    ```

7. Agregue la siguiente función para consultar el dispositivo y obtener la última hora de reinicio:

    ```javascript
    var queryTwinLastReboot = function() {

        registry.getTwin(deviceToReboot, function(err, twin){

            if (twin.properties.reported.iothubDM != null)
            {
                if (err) {
                    console.error('Could not query twins: ' + err.constructor.name + ': ' + err.message);
                } else {
                    var lastRebootTime = twin.properties.reported.iothubDM.reboot.lastReboot;
                    console.log('Last reboot time: ' + JSON.stringify(lastRebootTime, null, 2));
                }
            } else 
                console.log('Waiting for device to report last reboot time.');
        });
    };
    ```

8. Agregue el siguiente código para llamar a las funciones que desencadenan el método directo de reinicio y la consulta de última hora de reinicio:

    ```javascript
    startRebootDevice();
    setInterval(queryTwinLastReboot, 2000);
    ```

9. Guarde y cierre el archivo **dmpatterns_getstarted_service.js**.

## <a name="run-the-apps"></a>Ejecución de las aplicaciones

Ya está preparado para ejecutar las aplicaciones.

1. En el símbolo del sistema dentro de la carpeta **manageddevice**, ejecute el siguiente comando para iniciar la escucha del método directo de reinicio.

    ```cmd/sh
    node dmpatterns_getstarted_device.js
    ```

2. En el símbolo del sistema dentro de la carpeta **triggerrebootondevice**, ejecute el siguiente comando para desencadenar el reinicio remoto y la consulta en el dispositivo gemelo para buscar la hora del último reinicio.

    ```cmd/sh
    node dmpatterns_getstarted_service.js
    ```

3. Verá la respuesta del dispositivo al método directo de reinicio y el estado del reinicio en la consola.

   A continuación, puede ver la respuesta del dispositivo al método directo de reinicio que envía el servicio:

   ![salida de la aplicación manageddevice](./media/iot-hub-node-node-device-management-get-started/device.png)

   A continuación se muestra el servicio que desencadena el reinicio y sondea el dispositivo gemelo para la hora del último reinicio:

   ![salida de la aplicación triggerrebootondevice](./media/iot-hub-node-node-device-management-get-started/service.png)

[!INCLUDE [iot-hub-dm-followup](../../includes/iot-hub-dm-followup.md)]
