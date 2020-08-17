---
title: Conexión del código Node.js del dispositivo de ejemplo de la versión preliminar de IoT Plug and Play a IoT Hub | Microsoft Docs
description: Con Node.js, compile y ejecute el código del dispositivo de ejemplo de la versión preliminar de IoT Plug and Play que se conecta a una instancia de IoT Hub. Use la herramienta Azure IoT Explorer para ver la información enviada por el dispositivo al centro.
author: ericmitt
ms.author: ericmitt
ms.date: 07/10/2020
ms.topic: quickstart
ms.service: iot-pnp
services: iot-pnp
ms.custom: mvc, devx-track-javascript
ms.openlocfilehash: 00a748c3c372f1980042cff201edec720587a511
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87422562"
---
# <a name="quickstart-connect-a-sample-iot-plug-and-play-preview-device-application-to-iot-hub-nodejs"></a>Inicio rápido: Conexión de una aplicación de dispositivo de ejemplo de la versión preliminar de IoT Plug and Play a IoT Hub (Node.js)

[!INCLUDE [iot-pnp-quickstarts-device-selector.md](../../includes/iot-pnp-quickstarts-device-selector.md)]

En este inicio rápido se muestra cómo compilar una aplicación de dispositivo IoT Plug and Play de ejemplo, conectarla al centro de IoT y usar la herramienta Azure IoT Explorer para ver los datos de telemetría que envía. La aplicación de ejemplo se escribe en Node.js y se incluye en el SDK de dispositivo IoT de Azure para Node.js. Un generador de soluciones puede usar la herramienta Azure IoT Explorer para comprender las funcionalidades de cualquier dispositivo IoT Plug and Play sin necesidad de ver nada de código del dispositivo.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="prerequisites"></a>Requisitos previos

Para completar este inicio rápido, necesita Node.js en la máquina de desarrollo. Puede descargar la última versión recomendada para varias plataformas desde [nodejs.org](https://nodejs.org).

Puede verificar la versión actual de Node.js en el equipo de desarrollo con el comando siguiente:

```cmd/sh
node --version
```

### <a name="azure-iot-explorer"></a>Azure IoT Explorer

Para interactuar con el dispositivo de ejemplo en la segunda parte de esta guía de inicio rápido, use la herramienta **Azure IoT Explorer**. [Descargue e instale la versión de Azure IoT Explorer más reciente](./howto-use-iot-explorer.md) para su sistema operativo.

[!INCLUDE [iot-pnp-prepare-iot-hub.md](../../includes/iot-pnp-prepare-iot-hub.md)]

Ejecute el siguiente comando para obtener la _cadena de conexión del centro de IoT_ para el centro. Tome nota de esta cadena de conexión, ya que la usará más adelante en este inicio rápido:

```azurecli-interactive
az iot hub show-connection-string --hub-name <YourIoTHubName> --output table
```

> [!TIP]
> También puede usar la herramienta Azure IoT Explorer para buscar la cadena de conexión del centro de IoT.

Ejecute el siguiente comando para obtener la _cadena de conexión del dispositivo_ que ha agregado al centro. Tome nota de esta cadena de conexión, ya que la usará más adelante en este inicio rápido:

```azurecli-interactive
az iot hub device-identity show-connection-string --hub-name <YourIoTHubName> --device-id <YourDeviceID> --output table
```

[!INCLUDE [iot-pnp-download-models.md](../../includes/iot-pnp-download-models.md)]

## <a name="download-the-code"></a>Descargar el código

En este inicio rápido, se prepara un entorno de desarrollo que se puede usar para clonar y compilar el SDK del dispositivo de Azure IoT Hub para Node.js.

Abra un símbolo del sistema en el directorio que prefiera. Ejecute el siguiente comando para clonar el repositorio de GitHub del [SDK de IoT de Microsoft Azure para Node.js](https://github.com/Azure/azure-iot-sdk-node) en esta ubicación:

```cmd/sh
git clone https://github.com/Azure/azure-iot-sdk-node
```

## <a name="install-required-libraries"></a>Instalación de bibliotecas necesarias

Para compilar el código de ejemplo incluido, se usa el SDK del dispositivo. La aplicación que se compila simula un dispositivo que se conecta a un centro de IoT. La aplicación envía datos de telemetría y propiedades, y recibe comandos.

1. En una ventana del terminal local, vaya a la carpeta del repositorio clonado y luego a la carpeta */azure-iot-sdk-node/device/samples/pnp*. Después, ejecute el siguiente comando para instalar las bibliotecas necesarias:

    ```cmd/sh
    npm install
    ```

1. Configure la variable de entorno con la cadena de conexión del dispositivo que anotó anteriormente:

    ```cmd/sh
    set IOTHUB_DEVICE_CONNECTION_STRING=<YourDeviceConnectionString>
    ```

## <a name="run-the-sample-device"></a>Ejecución del dispositivo de ejemplo

Abra el archivo _simple_thermostat.js_. En este archivo, puede ver cómo:

1. Importar las interfaces necesarias.
1. Escribir un controlador de actualización de propiedades y un controlador de comandos.
1. Controlar las revisiones de propiedades deseadas y enviar telemetría.
1. De manera opcional, aprovisionar el dispositivo mediante el una única instancia de IoT Hub mediante Azure Device Provisioning Service (DPS).

En la función main, puede ver cómo todo encaja:

1. Cree el dispositivo a partir de la cadena de conexión o aprovisiónelo con DPS.
1. Use la opción **modelID** para especificar el modelo de dispositivo IoT Plug and Play.
1. Habilite el controlador de comandos.
1. Envíe telemetría desde el dispositivo al centro.
1. Obtenga los dispositivos gemelos y actualice las propiedades proporcionadas.
1. Habilite el controlador de actualizaciones de propiedades deseado.

Ejecute una aplicación de ejemplo para simular un dispositivo IoT Plug and Play que envía telemetría al centro de IoT. Para ejecutar la aplicación de ejemplo, utilice el comando siguiente:

```cmd\sh
node simple_thermostat.js
```

Verá la salida siguiente, lo que significa que el dispositivo ha comenzado a enviar datos de telemetría al concentrador y que ya está listo para recibir comandos y actualizaciones de propiedades.

![Mensajes de confirmación de dispositivo](media/quickstart-connect-device-node/device-confirmation-node.png)

Siga ejecutando el ejemplo a medida que complete los pasos siguientes.

## <a name="use-azure-iot-explorer-to-validate-the-code"></a>Uso de Azure IoT Explorer para validar el código

Una vez iniciado el ejemplo de cliente de dispositivo, compruebe que funciona con la herramienta Azure IoT Explorer.

[!INCLUDE [iot-pnp-iot-explorer.md](../../includes/iot-pnp-iot-explorer.md)]

[!INCLUDE [iot-pnp-clean-resources.md](../../includes/iot-pnp-clean-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido ha aprendido a conectar un dispositivo IoT Plug and Play a un centro de IoT. Para más información sobre cómo crear una solución que interactúe con dispositivos IoT Plug and Play, consulte:

> [!div class="nextstepaction"]
> [Interacción con un dispositivo IoT Plug and Play (versión preliminar) que esté conectado a una solución](quickstart-service-node.md)
