---
title: Conexión del código del dispositivo del componente de Node.js de ejemplo de la versión preliminar de IoT Plug and Play a IoT Hub | Microsoft Docs
description: Compile y ejecute el código de dispositivo de Node.js de ejemplo de la versión preliminar de IoT Plug and Play que usa varios componentes y se conecta a un centro de IoT. Use la herramienta Azure IoT Explorer para ver la información enviada por el dispositivo al centro.
author: olivakar
ms.author: olkar
ms.date: 07/10/2020
ms.topic: tutorial
ms.service: iot-pnp
services: iot-pnp
ms.custom: devx-track-javascript
ms.openlocfilehash: d26179ab82f29ce8f937f5b444463c1308d92047
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87904112"
---
# <a name="tutorial-connect-a-sample-iot-plug-and-play-preview-multiple-component-device-application-to-iot-hub-nodejs"></a>Tutorial: Conexión de una aplicación de dispositivo de varios componentes de ejemplo de la versión preliminar de IoT Plug and Play a IoT Hub (Node.js)

[!INCLUDE [iot-pnp-tutorials-device-selector.md](../../includes/iot-pnp-tutorials-device-selector.md)]

En este tutorial se muestra cómo compilar una aplicación de dispositivo IoT Plug and Play de ejemplo con componentes e interfaz raíz, cómo conectarla a un centro de IoT y cómo usar la herramienta Azure IoT Explorer para ver la información que envía al centro. La aplicación de ejemplo se escribe para Node.js y se incluye en el SDK de dispositivo de Azure IoT Hub para Node.js. Un generador de soluciones puede usar la herramienta Azure IoT Explorer para comprender las funcionalidades de cualquier dispositivo IoT Plug and Play sin necesidad de ver nada de código del dispositivo.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, necesita Node.js en la máquina de desarrollo. Puede descargar la última versión recomendada para varias plataformas desde [nodejs.org](https://nodejs.org).

Puede verificar la versión actual de Node.js en el equipo de desarrollo con el comando siguiente:

```cmd/sh
node --version
```

### <a name="azure-iot-explorer"></a>Azure IoT Explorer

Para interactuar con el dispositivo de ejemplo en la segunda parte de este tutorial, use la herramienta **Azure IoT Explorer**. [Descargue e instale la versión de Azure IoT Explorer más reciente](./howto-use-iot-explorer.md) para su sistema operativo.

[!INCLUDE [iot-pnp-prepare-iot-hub.md](../../includes/iot-pnp-prepare-iot-hub.md)]

Ejecute el siguiente comando para obtener la _cadena de conexión del centro de IoT_ para su centro. Tome nota de esta cadena de conexión, ya que la usará más adelante en este tutorial:

```azurecli-interactive
az iot hub show-connection-string --hub-name <YourIoTHubName> --output table
```

> [!TIP]
> También puede usar la herramienta Azure IoT Explorer para buscar la cadena de conexión del centro de IoT.

Ejecute el siguiente comando para obtener la _cadena de conexión del dispositivo_ que ha agregado al centro. Tome nota de esta cadena de conexión, ya que la usará más adelante en este tutorial:

```azurecli-interactive
az iot hub device-identity show-connection-string --hub-name <YourIoTHubName> --device-id <YourDeviceID> --output table
```

[!INCLUDE [iot-pnp-download-models.md](../../includes/iot-pnp-download-models.md)]

## <a name="download-the-code"></a>Descargar el código

En este tutorial, se prepara un entorno de desarrollo que se puede usar para clonar y compilar el SDK del dispositivo de Azure IoT Hub para Node.js.

Abra un símbolo del sistema en el directorio que prefiera. Ejecute el siguiente comando para clonar el repositorio de GitHub del [SDK de IoT de Microsoft Azure para Node.js](https://github.com/Azure/azure-iot-sdk-node) en esta ubicación:

```cmd/sh
git clone https://github.com/Azure/azure-iot-sdk-node
```

Esta operación puede tardar varios minutos en completarse.

## <a name="install-required-libraries"></a>Instalación de bibliotecas necesarias

Para compilar el código de ejemplo incluido, se usa el SDK del dispositivo. La aplicación que compila simula un dispositivo Plug and Play con varios componentes y la interfaz raíz que se conecta a un centro de IoT. La aplicación envía datos de telemetría y propiedades, y recibe comandos.

1. En una ventana del terminal local, vaya a la carpeta del repositorio clonado y luego a la carpeta */azure-iot-sdk-node/device/samples/pnp*. Después, ejecute el siguiente comando para instalar las bibliotecas necesarias:

```cmd/sh
npm install
```

Con esto, se instalarán los archivos NPM pertinentes necesarios para ejecutar los ejemplos de la carpeta.

1. Configure la variable de entorno con la cadena de conexión del dispositivo que anotó anteriormente:

```cmd/sh
set DEVICE_CONNECTION_STRING=<YourDeviceConnectionString>
```

## <a name="review-the-code"></a>Revisión del código

Vaya a la carpeta *azure-iot-sdk-node\device\samples\pnp*.

La carpeta *azure-iot-sdk-node\device\samples\pnp* contiene el código de ejemplo para el dispositivo controlador de temperatura IoT Plug and Play.

El código del archivo *pnpTemperatureController.js* implementa un dispositivo controlador de temperatura IoT Plug and Play. El modelo que implementa este ejemplo usa [varios componentes](concepts-components.md). El [archivo de modelo del lenguaje de definición de gemelos digitales (DTDL) del dispositivo de temperatura](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/TemperatureController.json) define los datos de telemetría, las propiedades y los comandos que implementa el dispositivo.

Abra el archivo *pnpTemperatureController.js* en un editor de código de su elección. En el código de ejemplo se muestra cómo:

1. Definir el `modelId` que es el DTMI para el dispositivo que se está implementando. Este DTMI está definido por el usuario y debe coincidir con el DTMI del [modelo de DTDL del controlador de temperatura](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/TemperatureController.json).

2. Implementar los componentes definidos en el modelo de DTDL del controlador de temperatura. Los componentes de un controlador de temperatura real deben implementar estas dos interfaces. Estas dos interfaces ya están publicadas en un repositorio central. En este ejemplo, las dos interfaces son:
  - Thermostat
  - Información del dispositivo desarrollada por Azure

3. Definir los nombres de los componentes. Este ejemplo tiene dos termostatos y un componente de información del dispositivo.

4. Definir el nombre del comando. Son los comandos a los que responde el dispositivo.

5. Definir la constante `serialNumber`. `serialNumber` es fijo para todo dispositivo dado.

6. Definir los controladores de comandos.

7. Definir las funciones para enviar respuestas de comandos.

8. Definir las funciones auxiliares para registrar las solicitudes de comandos.

9. Definir una función auxiliar para crear las propiedades.

10. Definir un agente de escucha para las actualizaciones de propiedades.

11. Definir una función para enviar datos de telemetría desde este dispositivo. Ambos termostatos y el componente raíz envían datos de telemetría. Esta función recibe el nombre del componente como parámetro.

12. Definir una función `main` que:

    1. Usa el SDK de dispositivo para crear un cliente de dispositivo y conectarse a su centro de IoT. El dispositivo proporciona el `modelId` para que IoT Hub pueda identificar el dispositivo como dispositivo IoT Plug and Play.

    1. Inicia la escucha para las solicitudes de comandos mediante la función `onDeviceMethod`. La función configura un agente de escucha para las solicitudes de comandos desde el servicio:
        - El DTDL del dispositivo define los comandos `reboot` y `getMaxMinReport`.
        - La función `commandHandler` define cómo responde el dispositivo a un comando.

    1. Comienza el envío de datos de telemetría mediante `setInterval` y `sendTelemetry`.

    1. Usa la función `helperCreateReportedPropertiesPatch` para crear las propiedades, y `updateComponentReportedProperties` para actualizar las propiedades.

    1. Usa `desiredPropertyPatchListener` para escuchar las actualizaciones de propiedades.

    1. Deshabilita todos los agentes de escucha y las tareas, y sale del bucle al presionar **Q** o **q**.

Ahora que ha visto el código, use el siguiente comando para ejecutar el ejemplo:

```cmd\sh
node pnpTemperatureController.js
```

Verá la salida siguiente, lo que significa que el dispositivo ha comenzado a enviar datos de telemetría al concentrador y que ya está listo para recibir comandos y actualizaciones de propiedades.

![Mensajes de confirmación de dispositivo](media/tutorial-multiple-components-node/multiple-component.png)

Siga ejecutando el ejemplo a medida que complete los pasos siguientes.

## <a name="use-azure-iot-explorer-to-validate-the-code"></a>Uso de Azure IoT Explorer para validar el código

Una vez iniciado el ejemplo de cliente de dispositivo, compruebe que funciona con la herramienta Azure IoT Explorer.

[!INCLUDE [iot-pnp-iot-explorer.md](../../includes/iot-pnp-iot-explorer.md)]

[!INCLUDE [iot-pnp-clean-resources.md](../../includes/iot-pnp-clean-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha aprendido a conectar un dispositivo IoT Plug and Play con componentes a un centro de IoT. Para obtener más información sobre los modelos de dispositivo IoT Plug and Play, consulte:

> [!div class="nextstepaction"]
> [Guía para desarrolladores de modelado de IoT Plug and Play (versión preliminar)](concepts-developer-guide.md)
