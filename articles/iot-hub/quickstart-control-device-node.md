---
title: 'Inicio rápido: control de un dispositivo desde Azure IoT (Node.js)'
description: En este inicio rápido, ejecuta dos aplicaciones Node.js de muestra. Una aplicación es una aplicación back-end que puede controlar dispositivos conectados al centro de manera remota. La otra aplicación simula un dispositivo conectado al centro que se puede controlar de manera remota.
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
- seo-javascript-october2019
- mqtt
- 'Role: Cloud Development'
- devx-track-javascript
ms.date: 06/21/2019
ms.openlocfilehash: 5f3064b9be35d9b9d02be3e18469ed12b33757a6
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87421559"
---
# <a name="quickstart-use-nodejs-to-control-a-device-connected-to-an-azure-iot-hub"></a>Inicio rápido: Uso de Node.js para controlar un dispositivo conectado a una instancia de Azure IoT Hub

[!INCLUDE [iot-hub-quickstarts-2-selector](../../includes/iot-hub-quickstarts-2-selector.md)]

En este inicio rápido, se usa un método directo para controlar un dispositivo simulado conectado a Azure IoT Hub. IoT Hub es un servicio de Azure que permite administrar dispositivos IoT desde la nube e ingerir grandes volúmenes de datos de telemetría desde los dispositivos en la nube para su almacenamiento o procesamiento. Puede usar métodos directos para cambiar el comportamiento de un dispositivo conectado a IoT Hub de manera remota. En este inicio rápido se usan dos aplicaciones de Node.js: una aplicación de un dispositivo simulado que responde a métodos directos llamados desde una aplicación de back-end y una aplicación de back-end que llama a los métodos directos en el dispositivo simulado.

## <a name="prerequisites"></a>Prerrequisitos

* Una cuenta de Azure con una suscripción activa. [cree una de forma gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

* [Node.js 10+](https://nodejs.org).

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

Si ha completado la anterior [Guía de inicio rápido: Envío de telemetría desde un dispositivo a un centro de IoT](quickstart-send-telemetry-node.md) puede omitir este paso.

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-device"></a>Registrar un dispositivo

Si ha completado la anterior [Guía de inicio rápido: Envío de telemetría desde un dispositivo a un centro de IoT](quickstart-send-telemetry-node.md) puede omitir este paso.

Debe registrar un dispositivo con IoT Hub antes de poder conectarlo. En esta guía de inicio rápido, usará Azure Cloud Shell para registrar un dispositivo simulado.

1. Ejecute los siguientes comandos en Azure Cloud Shell para crear la identidad del dispositivo.

   **YourIoTHubName**: reemplace este marcador de posición por el nombre elegido para el centro de IoT.

   **MyNodeDevice**: es el nombre del dispositivo que se va a registrar. Se recomienda usar **MyNodeDevice** como se muestra. Si elige otro nombre distinto para el dispositivo, tendrá que usarlo en todo el artículo y actualizar el nombre del dispositivo en las aplicaciones de ejemplo antes de ejecutarlas.

    ```azurecli-interactive
    az iot hub device-identity create \
      --hub-name {YourIoTHubName} --device-id MyNodeDevice
    ```

2. Ejecute los siguientes comandos en Azure Cloud Shell para obtener la _cadena de conexión del dispositivo_ que acaba de registrar:

    **YourIoTHubName**: reemplace este marcador de posición por el nombre elegido para el centro de IoT.

    ```azurecli-interactive
    az iot hub device-identity show-connection-string \
      --hub-name {YourIoTHubName} \
      --device-id MyNodeDevice \
      --output table
    ```

    Anote la cadena de conexión del dispositivo, que se parecerá a esta:

   `HostName={YourIoTHubName}.azure-devices.net;DeviceId=MyNodeDevice;SharedAccessKey={YourSharedAccessKey}`

    Usará este valor más adelante en este inicio rápido.

3. También necesita una _cadena de conexión de servicio_ para permitir que la aplicación back-end se conecte a IoT Hub y recupere los mensajes. El comando siguiente recupera la cadena de conexión del servicio de su instancia de IoT Hub:

    **YourIoTHubName**: reemplace este marcador de posición por el nombre elegido para el centro de IoT.

    ```azurecli-interactive
    az iot hub show-connection-string \
      --policy-name service --name {YourIoTHubName} --output table

    ```

    Anote la cadena de conexión del servicio, que se parecerá a esta:

   `HostName={YourIoTHubName}.azure-devices.net;SharedAccessKeyName=service;SharedAccessKey={YourSharedAccessKey}`

    Usará este valor más adelante en este inicio rápido. Esta cadena de conexión del servicio no es la que anotó en el paso anterior.

## <a name="listen-for-direct-method-calls"></a>Escuchas para llamadas de método directo

La aplicación del dispositivo simulado se conecta a un punto de conexión específico del dispositivo en IoT Hub, envía los datos de telemetría simulados y escucha llamadas de método directo desde el centro. En este inicio rápido, la llamada de método directo desde el centro indica al dispositivo que debe cambiar el intervalo en el que envía los datos de telemetría. El dispositivo simulado envía una confirmación al centro después de que ejecuta el método directo.

1. En una ventana de terminal local, vaya a la carpeta raíz del proyecto de Node.js de ejemplo. A continuación, vaya a la carpeta **iot-hub\Quickstarts\simulated-device-2**.

2. Abra el archivo **SimulatedDevice.js** en el editor de texto de su elección.

    Reemplace el valor de la variable `connectionString` por la cadena de conexión del dispositivo que anotó anteriormente. Luego, guarde los cambios realizados en **SimulatedDevice.js**.

3. En la ventana de terminal local, ejecute los comandos siguientes para instalar las bibliotecas necesarias y ejecute la aplicación de dispositivo simulado:

    ```cmd/sh
    npm install
    node SimulatedDevice.js
    ```

    La siguiente captura de pantalla muestra la salida en la que la aplicación de dispositivo simulado envía datos de telemetría a IoT Hub:

    ![Ejecutar el dispositivo simulado](./media/quickstart-control-device-node/simulated-device-telemetry-iot-hub.png)

## <a name="call-the-direct-method"></a>Llamar al método directo

La aplicación back-end se conecta a un punto de conexión de servicio en IoT Hub. La aplicación realiza llamadas de método directo a un dispositivo con IoT Hub y escucha las confirmaciones. Normalmente, una aplicación back-end de IoT Hub se ejecuta en la nube.

1. En otra ventana de terminal local, vaya a la carpeta raíz del proyecto de Node.js de ejemplo. A continuación, vaya a la carpeta **iot-hub\Quickstarts\back-end-application**.

2. Abra el archivo **BackEndApplication.js** en el editor de texto de su elección.

    Reemplace el valor de la variable `connectionString` por la cadena de conexión del servicio que anotó anteriormente. Luego, guarde los cambios en el archivo **BackEndApplication.js**.

3. En la ventana de terminal local, ejecute los comandos siguientes para instalar las bibliotecas necesarias y ejecute la aplicación back-end:

    ```cmd/sh
    npm install
    node BackEndApplication.js
    ```

    En la siguiente captura de pantalla se muestra la salida en la que la aplicación realiza una llamada de método directo al dispositivo y recibe una confirmación:

    ![Salida cuando la aplicación realiza una llamada de método directo al dispositivo](./media/quickstart-control-device-node/direct-method-device-call.png)

    Después de ejecutar la aplicación back-end, verá un mensaje en la ventana de consola que ejecuta el dispositivo simulado y cambiará la velocidad a la que envía mensajes:

    ![Salida cuando hay un cambio en el cliente simulado](./media/quickstart-control-device-node/simulated-device-message-change.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [iot-hub-quickstarts-clean-up-resources](../../includes/iot-hub-quickstarts-clean-up-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha llamado a un método directo en un dispositivo desde una aplicación de back-end y ha respondido a la llamada de método directo en una aplicación de dispositivo simulado.

Para obtener información sobre cómo redirigir mensajes del dispositivo a la nube a diferentes destinos en la nube, continúe con el siguiente tutorial.

> [!div class="nextstepaction"]
> [Tutorial: Enrutar datos de telemetría a distintos puntos de conexión para procesamiento](tutorial-routing.md)
