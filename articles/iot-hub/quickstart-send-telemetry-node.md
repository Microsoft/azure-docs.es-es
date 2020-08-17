---
title: 'Inicio rápido: Envío de telemetría a Azure IoT (Node.js)'
description: En esta guía de inicio rápido, ejecutará dos aplicaciones Node.js de ejemplo para enviar datos de telemetría simulados a un centro de IoT y para leer los datos de telemetría procedentes de este para procesarlos en la nube.
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.devlang: nodejs
ms.topic: quickstart
ms.custom:
- mvc
- seo-javascript-september2019
- mqtt
- 'Role: Cloud Development'
- devx-track-javascript
ms.date: 06/21/2019
ms.openlocfilehash: ea15c0d810f0ce51a52bc883ee44a4a90391b1d3
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87420947"
---
# <a name="quickstart-send-telemetry-from-a-device-to-an-iot-hub-and-read-it-with-a-back-end-application-nodejs"></a>Inicio rápido: Envío de telemetría desde un dispositivo a un centro de IoT y su lectura con una aplicación de back-end (Node.js)

[!INCLUDE [iot-hub-quickstarts-1-selector](../../includes/iot-hub-quickstarts-1-selector.md)]

 En este inicio rápido, se envían a través de Azure IoT Hub datos de telemetría desde una aplicación de dispositivo simulado a una aplicación de back-end para su procesamiento. IoT Hub es un servicio de Azure que le permite ingerir grandes volúmenes de datos de telemetría desde los dispositivos IoT en la nube para su almacenamiento o procesamiento. Este inicio rápido usa dos aplicaciones Node.js escritas previamente: una para enviar datos de telemetría y otra para leer datos de telemetría desde el centro. Antes de ejecutar estas dos aplicaciones, creará un centro de IoT Hub y registrará un dispositivo con el centro.

## <a name="prerequisites"></a>Prerrequisitos

* Una cuenta de Azure con una suscripción activa. [cree una de forma gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

* [Node.js 10+](https://nodejs.org). Si usa Azure Cloud Shell, no actualice la versión instalada de Node.js. Azure Cloud Shell ya tiene la versión más reciente de Node.js.

* [Un proyecto de Node.js de ejemplo](https://github.com/Azure-Samples/azure-iot-samples-node/archive/master.zip).

* El puerto 8883 abierto en el firewall. En el dispositivo de ejemplo de este inicio rápido se usa el protocolo MQTT, que se comunica mediante el puerto 8883. Este puerto puede estar bloqueado en algunos entornos de red corporativos y educativos. Para más información y para saber cómo solucionar este problema, consulte el artículo sobre la [conexión a IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

Puede verificar la versión actual de Node.js en el equipo de desarrollo con el comando siguiente:

```cmd/sh
node --version
```

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

### <a name="add-azure-iot-extension"></a>Adición de la extensión IoT de Azure

Ejecute el siguiente comando para agregar la extensión IoT de Microsoft Azure para la CLI de Azure a la instancia de Cloud Shell. La extensión IoT agrega comandos específicos de IoT Hub, IoT Edge e IoT Device Provisioning Service (DPS) a la CLI de Azure.

```azurecli-interactive
az extension add --name azure-iot
```

[!INCLUDE [iot-hub-cli-version-info](../../includes/iot-hub-cli-version-info.md)]

## <a name="create-an-iot-hub"></a>Crear un centro de IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-device"></a>Registrar un dispositivo

Debe registrar un dispositivo con IoT Hub antes de poder conectarlo. En esta guía de inicio rápido, usará Azure Cloud Shell para registrar un dispositivo simulado.

1. Ejecute los siguientes comandos en Azure Cloud Shell para crear la identidad del dispositivo.

   **YourIoTHubName**: reemplace este marcador de posición por el nombre elegido para el centro de IoT.

   **MyNodeDevice**: es el nombre del dispositivo que se va a registrar. Se recomienda usar **MyNodeDevice** como se muestra. Si elige un nombre distinto para el dispositivo, tendrá que usarlo en todo el artículo y actualizar el nombre del dispositivo en las aplicaciones de ejemplo antes de ejecutarlas.

    ```azurecli-interactive
    az iot hub device-identity create --hub-name {YourIoTHubName} --device-id MyNodeDevice
    ```

1. Ejecute el siguiente comandos en Azure Cloud Shell para obtener la _cadena de conexión_ del dispositivo que acaba de registrar:

   **YourIoTHubName**: reemplace este marcador de posición por el nombre elegido para el centro de IoT.

    ```azurecli-interactive
    az iot hub device-identity show-connection-string --hub-name {YourIoTHubName} --device-id MyNodeDevice --output table
    ```

    Anote la cadena de conexión del dispositivo, que se parecerá a esta:

   `HostName={YourIoTHubName}.azure-devices.net;DeviceId=MyNodeDevice;SharedAccessKey={YourSharedAccessKey}`

    Este valor lo usará más adelante en este inicio rápido.

1. También necesitará el _punto de conexión compatible con Event Hubs_, la _ruta de acceso compatible con Event Hubs_ y la _clave principal de servicio_ de IoT Hub para permitir que la aplicación back-end se conecte a IoT Hub y recupere los mensajes. Los siguientes comandos recuperan estos valores para IoT Hub:

   **YourIoTHubName**: reemplace este marcador de posición por el nombre elegido para el centro de IoT.

    ```azurecli-interactive
    az iot hub show --query properties.eventHubEndpoints.events.endpoint --name {YourIoTHubName}

    az iot hub show --query properties.eventHubEndpoints.events.path --name {YourIoTHubName}

    az iot hub policy show --name service --query primaryKey --hub-name {YourIoTHubName}
    ```

    Anote estos tres valores, ya que los usará más adelante en el inicio rápido.

## <a name="send-simulated-telemetry"></a>Envío de datos de telemetría simulados

La aplicación de dispositivo simulado se conecta a un punto de conexión específico del dispositivo en IoT Hub y envía datos de telemetría simulados sobre temperatura y humedad.

1. Abra la ventana de terminal local y vaya a la carpeta raíz del proyecto de Node.js de ejemplo. A continuación, vaya a la carpeta **iot-hub\Quickstarts\simulated-device**.

1. Abra el archivo **SimulatedDevice.js** en el editor de texto de su elección.

    Reemplace el valor de la variable `connectionString` por la cadena de conexión del dispositivo que anotó anteriormente. Luego, guarde los cambios realizados en **SimulatedDevice.js**.

1. En la ventana de terminal local, ejecute los comandos siguientes para instalar las bibliotecas necesarias y ejecute la aplicación de dispositivo simulado:

    ```cmd/sh
    npm install
    node SimulatedDevice.js
    ```

    La siguiente captura de pantalla muestra la salida en la que la aplicación de dispositivo simulado envía datos de telemetría a IoT Hub:

    ![Ejecutar el dispositivo simulado](media/quickstart-send-telemetry-node/SimulatedDevice.png)

## <a name="read-the-telemetry-from-your-hub"></a>Lectura de los datos de telemetría procedentes de su instancia de IoT Hub

La aplicación back-end se conecta a un punto de conexión de **Eventos** de servicio en IoT Hub. La aplicación recibe los mensajes del dispositivo a la nube enviados desde el dispositivo simulado. Normalmente, una aplicación back-end de IoT Hub se ejecuta en la nube para recibir y procesar los mensajes del dispositivo a la nube.

1. En otra ventana de terminal local, vaya a la carpeta raíz del proyecto de Node.js de ejemplo. A continuación, vaya a la carpeta **iot-hub\Quickstarts\read-d2c-messages**.

1. Abra el archivo **ReadDeviceToCloudMessages.cs** en el editor de texto de su elección. Actualice las siguientes variables y guarde los cambios en el archivo.

    | Variable | Value |
    | -------- | ----------- |
    | `eventHubsCompatibleEndpoint` | Reemplace el valor de la variable por el punto de conexión compatible con Event Hubs que anotó anteriormente. |
    | `eventHubsCompatiblePath`     | Reemplace el valor de la variable por la ruta de acceso compatible con Event Hubs que ha anotó anteriormente. |
    | `iotHubSasKey`                | Reemplace el valor de la variable por la clave principal del servicio que anotó anteriormente. |

1. En la ventana de terminal local, ejecute los comandos siguientes para instalar las bibliotecas necesarias y ejecute la aplicación back-end:

    ```cmd/sh
    npm install
    node ReadDeviceToCloudMessages.js
    ```

    La siguiente captura de pantalla muestra la salida en la que la aplicación back-end recibe los datos de telemetría que el dispositivo simulado ha enviado al centro:

    ![Ejecutar la aplicación back-end](media/quickstart-send-telemetry-node/ReadDeviceToCloud.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [iot-hub-quickstarts-clean-up-resources](../../includes/iot-hub-quickstarts-clean-up-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha configurado un centro de IoT, ha registrado un dispositivo, ha enviado datos de telemetría simulados al centro de conectividad mediante una aplicación en Node.js y ha leído datos de telemetría procedentes del centro de conectividad mediante una sencilla aplicación back-end.

Para aprender a controlar el dispositivo simulado desde una aplicación back-end, vaya al siguiente inicio rápido.

> [!div class="nextstepaction"]
> [Inicio rápido: Control de un dispositivo conectado a IoT Hub](quickstart-control-device-node.md)
