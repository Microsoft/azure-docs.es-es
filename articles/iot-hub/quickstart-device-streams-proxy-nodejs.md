---
title: Inicio rápido sobre los flujos de dispositivo de Azure IoT Hub en Node.js para SSH y RDP
description: En este inicio rápido, ejecutará una aplicación de Node.js de ejemplo que actúa como proxy para habilitar escenarios de SSH y RDP mediante flujos de dispositivo de IoT Hub.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.devlang: nodejs
ms.topic: quickstart
ms.custom: mvc, devx-track-javascript
ms.date: 03/14/2019
ms.author: robinsh
ms.openlocfilehash: 910ea6d333da08e69b6c5e6d6fe86421bda743ee
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87422596"
---
# <a name="quickstart-enable-ssh-and-rdp-over-an-iot-hub-device-stream-by-using-a-nodejs-proxy-application-preview"></a>Inicio rápido: Habilitar SSH y RDP mediante un flujo de dispositivo de IoT Hub con una aplicación proxy en Node.js (versión preliminar)

[!INCLUDE [iot-hub-quickstarts-4-selector](../../includes/iot-hub-quickstarts-4-selector.md)]

En este inicio rápido se habilita que el tráfico de Secure Shell (SSH) y Protocolo de escritorio remoto (RDP) se envíe al dispositivo a través de un flujo de dispositivo. Los flujos de dispositivo de Azure IoT Hub permiten que las aplicaciones tanto de los servicios como de los dispositivos se comuniquen de forma segura y apta para el firewall. En este inicio rápido se describe la ejecución de una aplicación de proxy de Node.js que se ejecuta en el lado del servicio. Durante la versión preliminar pública, el SDK de Node.js solo admite flujos de dispositivo en el lado del servicio. Como resultado, en este inicio rápido solo se incluyen las instrucciones para ejecutar la aplicación de proxy local de dispositivo.

## <a name="prerequisites"></a>Prerrequisitos

* Haber completado [Habilitar SSH y RDP mediante un flujo de dispositivo de IoT Hub con una aplicación proxy en C ](./quickstart-device-streams-proxy-c.md) o [Habilitar SSH y RDP mediante un flujo de dispositivo de IoT Hub con una aplicación proxy en C#](./quickstart-device-streams-proxy-csharp.md).

* Una cuenta de Azure con una suscripción activa. [cree una de forma gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

* [Node.js 10+](https://nodejs.org).

* [Un proyecto de Node.js de ejemplo](https://github.com/Azure-Samples/azure-iot-samples-node/archive/streams-preview.zip).

Para comprobar la versión actual de Node.js en la máquina de desarrollo, use el siguiente comando:

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

Ejecute el siguiente comando para agregar la extensión IoT de Azure para la CLI de Azure a la instancia de Cloud Shell. La extensión IoT agrega comandos específicos de IoT Hub, IoT Edge e IoT Device Provisioning Service (DPS) a la CLI de Azure.

```azurecli-interactive
az extension add --name azure-iot
```

[!INCLUDE [iot-hub-cli-version-info](../../includes/iot-hub-cli-version-info.md)]

## <a name="create-an-iot-hub"></a>Crear un centro de IoT

Si ha completado la anterior [Guía de inicio rápido: Envío de telemetría desde un dispositivo a un centro de IoT](quickstart-send-telemetry-node.md) puede omitir este paso.

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-device"></a>Registrar un dispositivo

Si ha completado [Inicio rápido: Envío de telemetría desde un dispositivo a un centro de IoT](quickstart-send-telemetry-node.md) puede omitir este paso.

Debe registrar un dispositivo con IoT Hub antes de poder conectarlo. En esta sección, usará Azure Cloud Shell para registrar un dispositivo simulado.

1. Ejecute el siguiente comando en Cloud Shell para crear la identidad del dispositivo:

   > [!NOTE]
   > * Reemplace el marcador de posición *YourIoTHubName* por el nombre que eligió para su centro de IoT.
   > * Para el nombre del dispositivo que va a registrar se recomienda usar *MyDevice* , tal como se muestra. Si elige otro nombre para el dispositivo, úselo en todo el artículo y actualice el nombre del dispositivo en las aplicaciones de ejemplo antes de ejecutarlas.

    ```azurecli-interactive
    az iot hub device-identity create --hub-name {YourIoTHubName} --device-id MyDevice
    ```

1. También necesita una *cadena de conexión de servicio* para permitir que la aplicación back-end se conecte a IoT Hub y recupere los mensajes. El siguiente comando recupera esta cadena para el centro de IoT:

   > [!NOTE]
   > Reemplace el marcador de posición *YourIoTHubName* por el nombre que eligió para su centro de IoT.

    ```azurecli-interactive
    az iot hub show-connection-string --policy-name service --name {YourIoTHubName} --output table
    ```

   Anote la cadena de conexión del servicio devuelta para usarla más adelante en este inicio rápido. Debe ser similar al ejemplo siguiente:

   `"HostName={YourIoTHubName}.azure-devices.net;SharedAccessKeyName=service;SharedAccessKey={YourSharedAccessKey}"`

## <a name="ssh-to-a-device-via-device-streams"></a>Conexión mediante SSH a un dispositivo mediante flujos de dispositivo

En esta sección, establece una secuencia de un extremo a otro para tunelizar el tráfico SSH.

### <a name="run-the-device-local-proxy-application"></a>Ejecución de la aplicación de proxy local de dispositivo

Como ya se ha mencionado, el SDK de IoT Hub para Node.js solo admite flujos de dispositivo en el lado del servicio. Para la aplicación de dispositivo local, use una aplicación de proxy de dispositivo que esté disponible en uno de los inicios rápidos siguientes:

   * [Habilitar SSH y RDP mediante un flujo de dispositivo de IoT Hub con una aplicación proxy en C](./quickstart-device-streams-proxy-c.md)
   * [Habilitar SSH y RDP mediante un flujo de dispositivo de IoT Hub con una aplicación proxy en C#](./quickstart-device-streams-proxy-csharp.md) 

Antes de continuar con el paso siguiente, asegúrese de que se está ejecutando la aplicación de proxy de dispositivo local. Consulte [Ejemplo Proxy local](./iot-hub-device-streams-overview.md#local-proxy-sample-for-ssh-or-rdp) para obtener información general de la configuración.

### <a name="run-the-service-local-proxy-application"></a>Ejecución de la aplicación de proxy local de servicio

En este artículo se describe la configuración de SSH (mediante el puerto 22) y, a continuación, se describe cómo modificar la configuración para RDP (que usa el puerto 3389). Puesto que los flujos de dispositivo son independientes del protocolo y de la aplicación, se puede modificar el mismo ejemplo para dar cabida a otros tipos de tráfico de la aplicación cliente/servidor (normalmente mediante la modificación del puerto de comunicación).

Con la aplicación de proxy local de dispositivo en ejecución, ejecute la aplicación de proxy local de servicio escrita en Node.js. Para ello, haga lo siguiente en una ventana de terminal local:

1. Como variables de entorno proporcione las credenciales del servicio, el identificador de dispositivo de destino donde se ejecuta el demonio de SSH y el número de puerto del proxy que se ejecuta en el dispositivo.

   ```
   # In Linux
   export IOTHUB_CONNECTION_STRING="{ServiceConnectionString}"
   export STREAMING_TARGET_DEVICE="MyDevice"
   export PROXY_PORT=2222

   # In Windows
   SET IOTHUB_CONNECTION_STRING={ServiceConnectionString}
   SET STREAMING_TARGET_DEVICE=MyDevice
   SET PROXY_PORT=2222
   ```

   Cambie el marcador de posición ServiceConnectionString para que coincida con la cadena de conexión del servicio y **MyDevice**  para que coincida con el identificador del dispositivo, si ha asignado otro nombre al suyo.

1. Vaya al directorio `Quickstarts/device-streams-service` en la carpeta del proyecto descomprimida. Use el código siguiente para ejecutar la aplicación proxy local de servicio:

   ```
   cd azure-iot-samples-node-streams-preview/iot-hub/Quickstarts/device-streams-service

   # Install the preview service SDK, and other dependencies
   npm install azure-iothub@streams-preview
   npm install

   # Run the service-local proxy application
   node proxy.js
   ```

### <a name="ssh-to-your-device-via-device-streams"></a>Conexión mediante SSH al dispositivo mediante flujos de dispositivo

En Linux, ejecute SSH mediante `ssh $USER@localhost -p 2222` en un terminal. En Windows, use su cliente SSH favorito (por ejemplo, PuTTY).

Salida de la consola en el servicio local una vez establecida la sesión SSH (la aplicación de proxy local de servicio escucha en el puerto 2222):

![Salida del terminal de SSH](./media/quickstart-device-streams-proxy-nodejs/service-console-output.png)

Salida de la consola de la aplicación cliente de SSH (el cliente de SSH se comunica con el demonio de SSH mediante la conexión al puerto 22, en el que escucha la aplicación de proxy local de servicio):

![Resultado del cliente de SSH](./media/quickstart-device-streams-proxy-nodejs/ssh-console-output.png)

### <a name="rdp-to-your-device-via-device-streams"></a>Conexión mediante RDP al dispositivo mediante flujos de dispositivo

Ahora utilice la aplicación cliente de RDP y conéctese a la aplicación de proxy de servicio en el puerto 2222 (este era un puerto arbitrario que eligió anteriormente).

> [!NOTE]
> Asegúrese de que el proxy de dispositivo está configurado correctamente para RDP y con el puerto 3389 de RDP.

![El cliente RDP se conecta a la aplicación de proxy local de servicio](./media/quickstart-device-streams-proxy-nodejs/rdp-screen-capture.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [iot-hub-quickstarts-clean-up-resources](../../includes/iot-hub-quickstarts-clean-up-resources-device-streams.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido se ha configurado un centro de IoT, se ha registrado un dispositivo y se ha implementado una aplicación proxy de servicio para habilitar RDP y SSH en un dispositivo IoT. El tráfico RDP y SSH se canalizará a través de un flujo de dispositivo al centro de IoT. Este proceso elimina la necesidad de conectividad directa al dispositivo.

Para más información sobre los flujos de dispositivo, consulte:

> [!div class="nextstepaction"]
> [Introducción a los flujos de dispositivo](./iot-hub-device-streams-overview.md)
