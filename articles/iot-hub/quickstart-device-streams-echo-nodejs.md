---
title: Comunicación con una aplicación de dispositivo en Node.js mediante flujos de dispositivo de Azure IoT Hub
description: En este inicio rápido, ejecutará una aplicación del servicio de Node.js que se comunica con un dispositivo IoT mediante un flujo de dispositivo.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.devlang: nodejs
ms.topic: quickstart
ms.custom: mvc, devx-track-javascript
ms.date: 03/14/2019
ms.author: robinsh
ms.openlocfilehash: d398e450175c9f97c419b397266cda0a52e7e21c
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87416135"
---
# <a name="quickstart-communicate-to-a-device-application-in-nodejs-via-iot-hub-device-streams-preview"></a>Inicio rápido: Comunicación con una aplicación de dispositivo en Node.js mediante flujos de dispositivo de IoT Hub (versión preliminar)

[!INCLUDE [iot-hub-quickstarts-3-selector](../../includes/iot-hub-quickstarts-3-selector.md)]

En este inicio rápido, se ejecuta una aplicación del lado del servicio y se configura la comunicación entre un dispositivo y un servicio mediante flujos de dispositivo. Los flujos de dispositivo de Azure IoT Hub permiten que las aplicaciones tanto de los servicios como de los dispositivos se comuniquen de forma segura y apta para el firewall. Durante la versión preliminar pública, el SDK para Node.js solo admite flujos de dispositivo en el lado del servicio. Por consiguiente, este artículo de inicio rápido solo abarca las instrucciones para ejecutar la aplicación del lado del servicio.

## <a name="prerequisites"></a>Prerrequisitos

* Haber completado [Comunicación con aplicaciones de dispositivo en C mediante flujos de dispositivo de IoT Hub](./quickstart-device-streams-echo-c.md) o [Comunicación con aplicaciones de dispositivo en C# mediante flujos de dispositivo de IoT Hub](./quickstart-device-streams-echo-csharp.md).

* Una cuenta de Azure con una suscripción activa. [cree una de forma gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

* [Node.js 10+](https://nodejs.org).

* [Un proyecto de Node.js de ejemplo](https://github.com/Azure-Samples/azure-iot-samples-node/archive/streams-preview.zip).

Puede verificar la versión actual de Node.js en el equipo de desarrollo con el comando siguiente:

```cmd/sh
node --version
```

Microsoft Azure IoT Hub actualmente admite flujos de dispositivos como una [versión preliminar](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

> [!IMPORTANT]
> La versión preliminar de los flujos de dispositivos solo se admite en este momento en instancias de IoT Hub creadas en las siguientes regiones:
>
> * Centro de EE. UU.
> * EUAP del centro de EE. UU.
> * Norte de Europa
> * Sudeste de Asia

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

### <a name="add-azure-iot-extension"></a>Adición de la extensión IoT de Azure

Ejecute el siguiente comando para agregar la extensión IoT de Microsoft Azure para la CLI de Azure a la instancia de Cloud Shell. La extensión IoT agrega comandos de IoT Hub, IoT Edge e IoT Device Provisioning Service (DPS) a la CLI de Azure.

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

   **MyDevice**: es el nombre del dispositivo que va a registrar. Se recomienda usar **MyDevice** como se muestra. Si elige otro nombre distinto para el dispositivo, tendrá que usarlo en todo el artículo y actualizar el nombre del dispositivo en las aplicaciones de ejemplo antes de ejecutarlas.

    ```azurecli-interactive
    az iot hub device-identity create --hub-name {YourIoTHubName} --device-id MyDevice
    ```

2. También necesita una *cadena de conexión de servicio* para permitir que la aplicación back-end se conecte a IoT Hub y recupere los mensajes. El comando siguiente recupera la cadena de conexión del servicio de su instancia de IoT Hub:

    **YourIoTHubName**: reemplace este marcador de posición por el nombre elegido para el centro de IoT.

    ```azurecli-interactive
    az iot hub show-connection-string --policy-name service --name {YourIoTHubName} --output table
    ```

    Anote la cadena de conexión del servicio devuelta para usarla más adelante en este inicio rápido. Debe ser similar al ejemplo siguiente:

   `"HostName={YourIoTHubName}.azure-devices.net;SharedAccessKeyName=service;SharedAccessKey={YourSharedAccessKey}"`

## <a name="communicate-between-device-and-service-via-device-streams"></a>Comunicación entre un dispositivo y un servicio mediante flujos de dispositivo

En esta sección, ejecuta la aplicación del lado del dispositivo y la aplicación del lado del servicio y se comunica entre las dos.

### <a name="run-the-device-side-application"></a>Ejecución de la aplicación del lado del dispositivo

Como ya se ha mencionado, el SDK de IoT Hub para Node.js solo admite flujos de dispositivo en el lado del servicio. Para la aplicación del dispositivo, use los programas de dispositivo complementarios disponibles en estos inicios rápidos:

* [Comunicación con una aplicación de dispositivo en C mediante flujos de dispositivo de IoT Hub](./quickstart-device-streams-echo-c.md)

* [Comunicación con aplicaciones de dispositivo en C# mediante flujos de dispositivo de IoT Hub](./quickstart-device-streams-echo-csharp.md)

Asegúrese de que la aplicación del lado del dispositivo se está ejecutando antes de continuar con el paso siguiente.

### <a name="run-the-service-side-application"></a>Ejecución de la aplicación del lado del servicio

La aplicación Node.js del lado del servidor de este inicio rápido tiene las siguientes funcionalidades:

* Crea un flujo de dispositivo a un dispositivo de IoT.
* Lee la entrada de línea de comandos y la envía a la aplicación del dispositivo, que la devuelve.

El código mostrará el proceso de inicio de un flujo de dispositivos y cómo usarlo para enviar y recibir datos.

Si se da por hecho que la aplicación del dispositivo está en ejecución, realice los siguientes pasos en una ventana de terminal local para ejecutar la aplicación del servicio escrita en Node.js:

* Especifique las credenciales del servicio y el identificador del dispositivo como variables de entorno.
 
   ```cmd/sh
   # In Linux
   export IOTHUB_CONNECTION_STRING="{ServiceConnectionString}"
   export STREAMING_TARGET_DEVICE="MyDevice"

   # In Windows
   SET IOTHUB_CONNECTION_STRING={ServiceConnectionString}
   SET STREAMING_TARGET_DEVICE=MyDevice
   ```
  
   Cambie el marcador de posición ServiceConnectionString para que coincida con la cadena de conexión del servicio y **MyDevice**  para que coincida con el identificador del dispositivo, si ha asignado otro nombre al suyo.

* Vaya a `Quickstarts/device-streams-service` en la carpeta del proyecto descomprimida y ejecute el ejemplo que usa el nodo.

   ```cmd/sh
   cd azure-iot-samples-node-streams-preview/iot-hub/Quickstarts/device-streams-service
    
   # Install the preview service SDK, and other dependencies
   npm install azure-iothub@streams-preview
   npm install

   node echo.js
   ```

Al final del último paso, el programa del lado del servicio iniciará un flujo hacia el dispositivo y, una vez que establecido, enviará un búfer de cadena al servicio a través de dicho flujo. En este ejemplo, el programa del lado del servicio simplemente lee `stdin` del terminal y lo envía al dispositivo, que lo devuelve. Aquí se muestra una comunicación bidireccional correcta entre las dos aplicaciones.

![Salida de la consola del servicio](./media/quickstart-device-streams-echo-nodejs/service-console-output.png)

Luego, para terminar el programa vuelva a presionar Entrar.

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [iot-hub-quickstarts-clean-up-resources-device-streams](../../includes/iot-hub-quickstarts-clean-up-resources-device-streams.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha configurado un centro de IoT, registrado un dispositivo, establecido un flujo de dispositivo entre las aplicaciones del dispositivo y del servicio, y usado dicho flujo para enviar datos entre las aplicaciones.

Use los siguientes vínculos para más información sobre los flujos de dispositivo:

> [!div class="nextstepaction"]
> [Introducción a los flujos de dispositivo](./iot-hub-device-streams-overview.md) 