---
title: Introducción a los dispositivos gemelos de IoT Hub de Azure | Microsoft Docs
description: Describe cómo usar dispositivos gemelos de IoT Hub de Azure para agregar etiquetas y, luego, usar una consulta de IoT Hub. El SDK de IoT de Azure para Node.js se usa para implementar la aplicación de dispositivo simulado y una aplicación de servicio que agrega las etiquetas y ejecuta la consulta de IoT Hub.
author: fsautomata
ms.service: iot-hub
services: iot-hub
ms.devlang: nodejs
ms.topic: conceptual
ms.date: 08/26/2019
ms.author: elioda
ms.custom: mqtt, devx-track-javascript
ms.openlocfilehash: 874e9dd0454fa4f0a0877d949a34058baa711b83
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87420726"
---
# <a name="get-started-with-device-twins-nodejs"></a>Introducción a los dispositivos gemelos (Node.js)

[!INCLUDE [iot-hub-selector-twin-get-started](../../includes/iot-hub-selector-twin-get-started.md)]

Al final de este tutorial tendrá dos aplicaciones de consola de Node.js:

* **AddTagsAndQuery.js**, una aplicación Node.js de back-end diseñada para ejecutarse desde el back-end, que agrega etiquetas y consulta dispositivos gemelos.

* **TwinSimulatedDevice.js**, una aplicación de Node.js que simula un dispositivo que se conecta a su instancia de IoT Hub con la identidad del dispositivo creada anteriormente e informa de su estado de conectividad.

> [!NOTE]
> En el artículo [SDK de IoT de Azure](iot-hub-devguide-sdks.md) se proporciona información sobre los SDK de IoT de Azure que se pueden usar para crear aplicaciones de dispositivo y de back-end.
>

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, necesita:

* Node.js versión 10.0.x o posteriores.

* Una cuenta de Azure activa. (En caso de no tener ninguna, puede crear una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) en tan solo unos minutos).

* Asegúrese de que el puerto 8883 está abierto en el firewall. En el ejemplo de dispositivo de este artículo se usa el protocolo MQTT, que se comunica mediante el puerto 8883. Este puerto puede estar bloqueado en algunos entornos de red corporativos y educativos. Para más información y saber cómo solucionar este problema, consulte [Conexión a IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="create-an-iot-hub"></a>Crear un centro de IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-new-device-in-the-iot-hub"></a>Registro de un nuevo dispositivo en el centro de IoT

[!INCLUDE [iot-hub-get-started-create-device-identity](../../includes/iot-hub-get-started-create-device-identity.md)]

## <a name="get-the-iot-hub-connection-string"></a>Obtención de la cadena de conexión de IoT Hub

[!INCLUDE [iot-hub-howto-twin-shared-access-policy-text](../../includes/iot-hub-howto-twin-shared-access-policy-text.md)]

[!INCLUDE [iot-hub-include-find-custom-connection-string](../../includes/iot-hub-include-find-custom-connection-string.md)]

## <a name="create-the-service-app"></a>Creación de la aplicación de servicio

En esta sección, creará una aplicación de consola de Node.js que agrega metadatos de ubicación al dispositivo gemelo asociado con **myDeviceId**. Después, consulta los dispositivos gemelos almacenados en la instancia de IoT Hub mediante la selección de los dispositivos ubicados en Estados Unidos y, después, los que informan de una conexión de red de telefonía móvil.

1. Cree una nueva carpeta vacía denominada **addtagsandqueryapp**. En la carpeta **addtagsandqueryapp** , cree un nuevo archivo package.json con el siguiente comando en el símbolo del sistema. El parámetro `--yes` acepta todos los valores predeterminados.

    ```cmd/sh
    npm init --yes
    ```

2. En el símbolo del sistema, en la carpeta **addtagsandqueryapp**, ejecute el siguiente comando para instalar el paquete **azure-iothub**:

    ```cmd/sh
    npm install azure-iothub --save
    ```

3. Con un editor de texto, cree un nuevo archivo **AddTagsAndQuery.js** en la carpeta **addtagsandqueryapp**.

4. Agregue el código siguiente al archivo **AddTagsAndQuery.js**. Reemplace `{iot hub connection string}` por la cadena de conexión de IoT Hub que copió en [Obtención de la cadena de conexión de IoT Hub](#get-the-iot-hub-connection-string).

   ``` javascript
        'use strict';
        var iothub = require('azure-iothub');
        var connectionString = '{iot hub connection string}';
        var registry = iothub.Registry.fromConnectionString(connectionString);

        registry.getTwin('myDeviceId', function(err, twin){
            if (err) {
                console.error(err.constructor.name + ': ' + err.message);
            } else {
                var patch = {
                    tags: {
                        location: {
                            region: 'US',
                            plant: 'Redmond43'
                      }
                    }
                };

                twin.update(patch, function(err) {
                  if (err) {
                    console.error('Could not update twin: ' + err.constructor.name + ': ' + err.message);
                  } else {
                    console.log(twin.deviceId + ' twin updated successfully');
                    queryTwins();
                  }
                });
            }
        });
   ```

    El objeto **Registro** expone todos los métodos necesarios para interactuar con dispositivos gemelos del servicio. El código anterior inicializa primero el objeto **Registro**, a continuación, recupera el dispositivo gemelo de **myDeviceId**, y por último actualiza sus etiquetas con la información de la ubicación deseada.

    Después de actualizar las etiquetas, llama a la función **queryTwins**.

5. Agregue el código siguiente al final de  **AddTagsAndQuery.js** para implementar la función **queryTwins**:

   ```javascript
        var queryTwins = function() {
            var query = registry.createQuery("SELECT * FROM devices WHERE tags.location.plant = 'Redmond43'", 100);
            query.nextAsTwin(function(err, results) {
                if (err) {
                    console.error('Failed to fetch the results: ' + err.message);
                } else {
                    console.log("Devices in Redmond43: " + results.map(function(twin) {return twin.deviceId}).join(','));
                }
            });

            query = registry.createQuery("SELECT * FROM devices WHERE tags.location.plant = 'Redmond43' AND properties.reported.connectivity.type = 'cellular'", 100);
            query.nextAsTwin(function(err, results) {
                if (err) {
                    console.error('Failed to fetch the results: ' + err.message);
                } else {
                    console.log("Devices in Redmond43 using cellular network: " + results.map(function(twin) {return twin.deviceId}).join(','));
                }
            });
        };
   ```

    El código anterior ejecuta dos consultas: la primera selecciona solo los dispositivos gemelos que se encuentran en la planta **Redmond43**, y la segunda mejora la consulta para seleccionar solo los dispositivos que están también conectados a través de la red de telefonía móvil.

    Cuando el código crea el objeto **consulta**, especifica el número máximo de documentos devueltos en el segundo parámetro. El objeto **consulta** contiene una propiedad booleana **hasMoreResults** que puede utilizar para invocar a los métodos **nextAsTwin** varias veces para recuperar todos los resultados. Un método llamado **siguiente** está disponible para los resultados que no son dispositivos gemelos, por ejemplo, los resultados de consultas de agregación.

6. Ejecute la aplicación con:

    ```cmd/sh
        node AddTagsAndQuery.js
    ```

   Debería ver un dispositivo en los resultados de la consulta que pregunta por todos los dispositivos que se encuentran en **Redmond43** y ninguno para la consulta que restringe los resultados a los dispositivos que utilizan una red de telefonía móvil.

   ![Visualización del dispositivo en los resultados de la consulta](media/iot-hub-node-node-twin-getstarted/service1.png)

En la siguiente sección, creará una aplicación de dispositivo que notifica la información de conectividad y cambia el resultado de la consulta en la sección anterior.

## <a name="create-the-device-app"></a>Creación de la aplicación del dispositivo

En esta sección, creará una aplicación de consola de Node.js que se conecta al centro como **myDeviceId**, y luego actualiza las propiedades notificadas de su dispositivo gemelo para contener la información que está conectada mediante una red de telefonía móvil.

1. Cree una nueva carpeta vacía denominada **reportconnectivity**. En la carpeta **reportconnectivity** , cree un nuevo archivo package.json con el siguiente comando en el símbolo del sistema. El parámetro `--yes` acepta todos los valores predeterminados.

    ```cmd/sh
    npm init --yes
    ```

2. En el símbolo del sistema, en la carpeta **reportconnectivity**, ejecute el siguiente comando para instalar **azure-iot-device** y el paquete **azure-iot-device-mqtt**:

    ```cmd/sh
    npm install azure-iot-device azure-iot-device-mqtt --save
    ```

3. Con un editor de texto, cree un nuevo archivo **ReportConnectivity.js** en la carpeta **reportconnectivity**.

4. Agregue el código siguiente al archivo **ReportConnectivity.js**. Reemplace `{device connection string}` por la cadena de conexión de dispositivo que copió al crear la identidad del dispositivo **myDeviceId** en [Registro de un nuevo dispositivo en el centro de IoT](#register-a-new-device-in-the-iot-hub).

    ```javascript
        'use strict';
        var Client = require('azure-iot-device').Client;
        var Protocol = require('azure-iot-device-mqtt').Mqtt;

        var connectionString = '{device connection string}';
        var client = Client.fromConnectionString(connectionString, Protocol);

        client.open(function(err) {
        if (err) {
            console.error('could not open IotHub client');
        }  else {
            console.log('client opened');

            client.getTwin(function(err, twin) {
            if (err) {
                console.error('could not get twin');
            } else {
                var patch = {
                    connectivity: {
                        type: 'cellular'
                    }
                };

                twin.properties.reported.update(patch, function(err) {
                    if (err) {
                        console.error('could not update twin');
                    } else {
                        console.log('twin state reported');
                        process.exit();
                    }
                });
            }
            });
        }
        });
    ```

    El objeto **Cliente** expone todos los métodos necesarios para interactuar con dispositivos gemelos del dispositivo. El código anterior, una vez que inicialice el objeto **Cliente**, recupera el dispositivo gemelo de **myDeviceId**, y actualiza su propiedad notificada con la información de conectividad.

5. Ejecute la aplicación del dispositivo

    ```cmd/sh
        node ReportConnectivity.js
    ```

    Verá el mensaje `twin state reported`.

6. Ahora que el dispositivo ha informado sobre su información de conectividad, debe aparecer en ambas consultas. Vuelva a la carpeta **addtagsandqueryapp** y vuelva a ejecutar las consultas:

    ```cmd/sh
        node AddTagsAndQuery.js
    ```

    Esta vez **myDeviceId** debe aparecer en los resultados de ambas consulta.

    ![Representación de myDeviceId en los resultados de las dos consultas](media/iot-hub-node-node-twin-getstarted/service2.png)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, configuró un centro de IoT nuevo en Azure Portal y, después, creó una identidad de dispositivo en el registro de identidades del centro de IoT. Ha agregado metadatos de dispositivo como etiquetas desde una aplicación back-end y ha escrito una aplicación de dispositivo simulado para notificar la información de conectividad de dispositivos en el dispositivo gemelo. También ha aprendido cómo consultar esta información mediante el lenguaje de consulta de IoT Hub de tipo SQL.

Use los siguientes recursos para obtener información sobre cómo:

* Enviar telemetría desde dispositivos con el tutorial [Introducción a IoT Hub](quickstart-send-telemetry-node.md).

* Configurar dispositivos mediante las propiedades deseadas del dispositivo gemelo con el tutorial [Uso de las propiedades deseadas para configurar dispositivos](tutorial-device-twins.md).

* Controlar los dispositivos de forma interactiva (por ejemplo, encender un ventilador desde una aplicación controlada por el usuario), con el tutorial [Uso de métodos directos](quickstart-control-device-node.md).
