---
title: Primeros pasos con la identidad del módulo y el módulo gemelo de Azure IoT Hub (Node.js)
description: Aprenda a crear la identidad de módulo y a actualizar módulos gemelos mediante los SDK de IoT para Node.js.
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.devlang: nodejs
ms.topic: conceptual
ms.date: 04/26/2018
ms.custom: amqp, devx-track-javascript
ms.openlocfilehash: 214c0533e6aefcfa496533359221f7a3061031b8
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87420743"
---
# <a name="get-started-with-iot-hub-module-identity-and-module-twin-nodejs"></a>Introducción a la identidad de módulo y a los módulos gemelos de IoT Hub (Node.js)

[!INCLUDE [iot-hub-selector-module-twin-getstarted](../../includes/iot-hub-selector-module-twin-getstarted.md)]

> [!NOTE]
> [Las identidades de módulo y los módulos gemelos](iot-hub-devguide-module-twins.md) son similares a la identidad de dispositivo y el dispositivo gemelo de Azure IoT Hub, pero ofrecen granularidad más fina. Aunque la identidad del dispositivo y los dispositivos gemelos de Azure IoT Hub permiten que la aplicación back-end configure un dispositivo y proporcionan visibilidad para las condiciones del dispositivo, la identidad de módulo y los módulos gemelos proporcionan estas funcionalidades para los componentes individuales del dispositivo. En los dispositivos compatibles con varios componentes, como dispositivos con sistema operativo o firmware, permiten la configuración individual y condiciones específicas por componente.

Al final de este tutorial tendrá dos aplicaciones de Node.js:

* **CreateIdentities**, que crea una identidad de dispositivo, una de módulo y la clave de seguridad asociada para conectar el dispositivo y los clientes del módulo.

* **UpdateModuleTwinReportedProperties**, que envía las propiedades notificadas por el módulo gemelo actualizadas a IoT Hub.

> [!NOTE]
> Para más información acerca de los diversos SDK que puede usar para crear ambas aplicaciones para que se ejecuten en dispositivos y en el back-end de la solución, consulte el artículo de los [SDK de Azure IoT Hub](iot-hub-devguide-sdks.md).

## <a name="prerequisites"></a>Prerrequisitos

* Node.js versión 10.0.x o posteriores. En [Preparación del entorno de desarrollo](https://github.com/Azure/azure-iot-sdk-node/tree/master/doc/node-devbox-setup.md) se describe cómo instalar Node.js para este tutorial en Windows o Linux.

* Una cuenta de Azure activa. (En caso de no tener ninguna, puede crear una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) en tan solo unos minutos).

## <a name="create-an-iot-hub"></a>Crear un centro de IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="get-the-iot-hub-connection-string"></a>Obtención de la cadena de conexión de IoT Hub

[!INCLUDE [iot-hub-howto-module-twin-shared-access-policy-text](../../includes/iot-hub-howto-module-twin-shared-access-policy-text.md)]

[!INCLUDE [iot-hub-include-find-registryrw-connection-string](../../includes/iot-hub-include-find-registryrw-connection-string.md)]

## <a name="create-a-device-identity-and-a-module-identity-in-iot-hub"></a>Crear una identidad del dispositivo y una identidad de módulo en IoT Hub

En esta sección se crea una aplicación de Node.js que crea una identidad de dispositivo y otra de módulo en el registro de identidades de la instancia de IoT Hub. No se puede conectar un dispositivo o un módulo a IoT Hub a menos que tenga una entrada en el registro de identidades. Para más información, consulte la sección "Registro de identidades" de la [guía para desarrolladores de IoT Hub](iot-hub-devguide-identity-registry.md). Cuando se ejecuta esta aplicación de consola, genera una clave y un identificador únicos para el dispositivo y el módulo. El dispositivo y el módulo usan estos valores para identificarse al enviar mensajes del dispositivo a la nube a IoT Hub. Los identificadores distinguen mayúsculas de minúsculas.

1. Cree un directorio para almacenar el código.

2. Dentro de ese directorio, ejecute primero **npm init -y** para crear un archivo package.json vacío con los valores predeterminados. Este es el archivo de proyecto para su código.

3. Ejecute **npm install -S azure-iothub\@modules-preview** para instalar el SDK del servicio dentro del subdirectorio **node_modules**.

    > [!NOTE]
    > El nombre de subdirectorio node_modules utiliza la palabra module para indicar "una biblioteca de nodos". En este caso, el término no tiene nada que ver con los módulos de IoT Hub.

4. Cree el siguiente archivo .js en su directorio. Llámelo **add.js**. Copie y pegue la cadena de conexión del centro y el nombre del centro.

    ```javascript
    var Registry = require('azure-iothub').Registry;
    var uuid = require('uuid');
    // Copy/paste your connection string and hub name here
    var serviceConnectionString = '<hub connection string from portal>';
    var hubName = '<hub name>.azure-devices.net';
    // Create an instance of the IoTHub registry
    var registry = Registry.fromConnectionString(serviceConnectionString);
    // Insert your device ID and moduleId here.
    var deviceId = 'myFirstDevice';
    var moduleId = 'myFirstModule';
    // Create your device as a SAS authentication device
    var primaryKey = new Buffer(uuid.v4()).toString('base64');
    var secondaryKey = new Buffer(uuid.v4()).toString('base64');
    var deviceDescription = {
      deviceId: deviceId,
      status: 'enabled',
      authentication: {
        type: 'sas',
        symmetricKey: {
          primaryKey: primaryKey,
          secondaryKey: secondaryKey
        }
      }
    };

    // First, create a device identity
    registry.create(deviceDescription, function(err) {
      if (err) {
        console.log('Error creating device identity: ' + err);
        process.exit(1);
      }
      console.log('device connection string = "HostName=' + hubName + ';DeviceId=' + deviceId + ';SharedAccessKey=' + primaryKey + '"');

      // Then add a module to that device
      registry.addModule({ deviceId: deviceId, moduleId: moduleId }, function(err) {
        if (err) {
          console.log('Error creating module identity: ' + err);
          process.exit(1);
        }

        // Finally, retrieve the module details from the hub so we can construct the connection string
        registry.getModule(deviceId, moduleId, function(err, foundModule) {
          if (err) {
            console.log('Error getting module back from hub: ' + err);
            process.exit(1);
          }
          console.log('module connection string = "HostName=' + hubName + ';DeviceId=' + foundModule.deviceId + ';ModuleId='+foundModule.moduleId+';SharedAccessKey=' + foundModule.authentication.symmetricKey.primaryKey + '"');
          process.exit(0);
        });
      });
    });

    ```

Esta aplicación crea una identidad del dispositivo con el identificador **myFirstDevice** y una de módulo con el identificador **myFirstModule** en el dispositivo **myFirstDevice**. (Si el identificador de módulo ya existe en el Registro de identidad, el código simplemente recupera la información existente del módulo). A continuación, la aplicación muestra la clave principal de esa identidad. Esta clave se usará en la aplicación de módulo simulado para conectarse a IoT Hub.

Ejecútelo con el comando node add.js. Se le ofrecerá una cadena de conexión para la identidad del dispositivo y otra para la identidad del módulo.

> [!NOTE]
> El registro de identidades de IoT Hub solo almacena identidades de dispositivos y módulos para permitir el acceso seguro a IoT Hub. El registro de identidades almacena los identificadores y las claves de dispositivo para usarlas como credenciales de seguridad. El registro de identidades también almacena una marca de habilitado o deshabilitado de cada dispositivo que se puede usar para deshabilitar el acceso a dicho dispositivo. Si la aplicación necesita almacenar otros metadatos específicos del dispositivo, debe usar un almacén específico de la aplicación. No hay marcas de habilitado/deshabilitado para las identidades de módulo. Consulte la [guía de desarrolladores de IoT Hub](iot-hub-devguide-identity-registry.md) para obtener más información.

## <a name="update-the-module-twin-using-nodejs-device-sdk"></a>Actualización del módulo gemelo con el SDK de dispositivo Node.js

En esta sección se crea una aplicación de Node.js en el dispositivo simulado que actualiza las propiedades notificadas del módulo gemelo.

1. **Obtenga la cadena de conexión del módulo**: inicie sesión en [Azure Portal](https://portal.azure.com/). Vaya a la instancia de IoT Hub y haga clic en Dispositivos IoT. Busque myFirstDevice, ábralo y verá que myFirstModule se creó correctamente. Copie la cadena de conexión del módulo, la necesitará en el paso siguiente.

   ![Detalles del módulo en Azure Portal](./media/iot-hub-node-node-module-twin-getstarted/module-detail.png)

2. De forma similar al paso anterior, cree un directorio para el código de dispositivo y use NPM para inicializarlo e instalar el SDK del dispositivo (**npm install -S azure-iot-device-amqp\@modules-preview**).

   > [!NOTE]
   > El comando npm install puede resultar lento. Sea paciente, porque extrae una gran cantidad de código del repositorio de paquetes.

   > [!NOTE]
   > Si ve un error "npm ERR! registry error parsing json" (Error de npm: error del registro al analizar json), puede ignorarlo sin problema. Si ve un error "npm ERR! registry error parsing json" (Error de npm: error del registro al analizar json), puede ignorarlo sin problema.

3. Cree un archivo llamado twin.js. Copie y pegue la cadena de identidad del módulo.

    ```javascript
    var Client = require('azure-iot-device').Client;
    var Protocol = require('azure-iot-device-amqp').Amqp;
    // Copy/paste your module connection string here.
    var connectionString = '<insert module connection string here>';
    // Create a client using the Amqp protocol.
    var client = Client.fromConnectionString(connectionString, Protocol);
    client.on('error', function (err) {
      console.error(err.message);
    });
    // connect to the hub
    client.open(function(err) {
      if (err) {
        console.error('error connecting to hub: ' + err);
        process.exit(1);
      }
      console.log('client opened');
    // Create device Twin
      client.getTwin(function(err, twin) {
        if (err) {
          console.error('error getting twin: ' + err);
          process.exit(1);
        }
        // Output the current properties
        console.log('twin contents:');
        console.log(twin.properties);
        // Add a handler for desired property changes
        twin.on('properties.desired', function(delta) {
            console.log('new desired properties received:');
            console.log(JSON.stringify(delta));
        });
        // create a patch to send to the hub
        var patch = {
          updateTime: new Date().toString(),
          firmwareVersion:'1.2.1',
          weather:{
            temperature: 72,
            humidity: 17
          }
        };
        // send the patch
        twin.properties.reported.update(patch, function(err) {
          if (err) throw err;
          console.log('twin state reported');
        });
      });
    });
    ```

4. Ahora, ejecútelo con el comando **node twin.js**.

   ```cmd/sh
   F:\temp\module_twin>node twin.js
   ```

   A continuación, verá:

   ```console
   client opened
   twin contents:
   { reported: { update: [Function: update], '$version': 1 },
     desired: { '$version': 1 } }
   new desired properties received:
   {"$version":1}
   twin state reported
   ```

## <a name="next-steps"></a>Pasos siguientes

Para continuar la introducción a IoT Hub y explorar otros escenarios de IoT, consulte:

* [Introducción a la administración de dispositivos](iot-hub-node-node-device-management-get-started.md)

* [Introducción a IoT Edge](../iot-edge/tutorial-simulate-device-linux.md)