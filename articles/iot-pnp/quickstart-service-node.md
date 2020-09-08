---
title: Interacción con un dispositivo IoT Plug and Play (versión preliminar) conectado a una solución de Azure IoT (Node.js) | Microsoft Docs
description: Use Node.js para conectarse e interactuar con un dispositivo IoT Plug and Play (versión preliminar) que esté conectado a su solución de Azure IoT.
author: elhorton
ms.author: elhorton
ms.date: 08/11/2020
ms.topic: quickstart
ms.service: iot-pnp
services: iot-pnp
ms.custom: mvc, devx-track-javascript
ms.openlocfilehash: fd65dcc9ce0be07daa5848a0ac583cf795150e47
ms.sourcegitcommit: faeabfc2fffc33be7de6e1e93271ae214099517f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88184762"
---
# <a name="quickstart-interact-with-an-iot-plug-and-play-preview-device-thats-connected-to-your-solution-nodejs"></a>Inicio rápido: Interacción con un dispositivo IoT Plug and Play (versión preliminar) que esté conectado a una solución (Node.js)

[!INCLUDE [iot-pnp-quickstarts-service-selector.md](../../includes/iot-pnp-quickstarts-service-selector.md)]

IoT Plug and Play (versión preliminar) simplifica IoT, ya que permite interactuar con las funcionalidades de un dispositivo sin tener que conocer la implementación subyacente del dispositivo. En este inicio rápido se muestra cómo usar Node.js para conectarse y controlar un dispositivo IoT Plug and Play que esté conectado a una solución.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="prerequisites"></a>Requisitos previos

Para completar este inicio rápido, necesita Node.js en la máquina de desarrollo. Puede descargar la última versión recomendada para varias plataformas desde [nodejs.org](https://nodejs.org).

Puede verificar la versión actual de Node.js en el equipo de desarrollo con el comando siguiente:

```cmd/sh
node --version
```

[!INCLUDE [iot-pnp-prepare-iot-hub.md](../../includes/iot-pnp-prepare-iot-hub.md)]

Ejecute el siguiente comando para obtener la _cadena de conexión del centro de IoT_ para su centro. Tome nota de esta cadena de conexión, ya que la usará más adelante en este inicio rápido:

```azurecli-interactive
az iot hub show-connection-string --hub-name <YourIoTHubName> --output table
```

Ejecute el siguiente comando para obtener la _cadena de conexión del dispositivo_ que ha agregado al centro. Tome nota de esta cadena de conexión, ya que la usará más adelante en este inicio rápido:

```azurecli-interactive
az iot hub device-identity show-connection-string --hub-name <YourIoTHubName> --device-id <YourDeviceID> --output
```

### <a name="clone-the-sdk-repository-with-the-sample-code"></a>Clonación del repositorio de SDK con el código de ejemplo

El SDK del servicio se encuentra en versión preliminar, por lo que debe clonar los ejemplos desde una [rama de versión preliminar del SDK de Node](https://github.com/Azure/azure-iot-sdk-node/tree/pnp-preview-refresh). Abra una ventana de terminal en la carpeta que prefiera. Ejecute el siguiente comando para clonar la rama **pnp-preview-refresh** del repositorio de GitHub del SDK de [Microsoft Azure IoT para Node.js](https://github.com/Azure/azure-iot-sdk-node):

```cmd/sh
git clone https://github.com/Azure/azure-iot-sdk-node -b pnp-preview-refresh
```

## <a name="run-the-sample-device"></a>Ejecución del dispositivo de ejemplo

En este inicio rápido, puede usar un dispositivo termostato de ejemplo, que se ha escrito en Node.js, como dispositivo IoT Plug and Play. Para ejecutar el dispositivo de ejemplo:

1. Abra una ventana de terminal y vaya a la carpeta local que contiene el repositorio de SDK de Microsoft Azure IoT para Node.js que ha clonado de GitHub.

1. Esta ventana de terminal se usa ahora como terminal del **dispositivo**. Vaya a la carpeta del repositorio clonado y luego a la carpeta */azure-iot-sdk-node/device/samples/pnp*. Instale todas las dependencias, para lo que debe ejecutar el siguiente comando:

    ```cmd/sh
    npm install
    ```

1. Configure la _cadena de conexión del dispositivos_:

    ```cmd/sh
    set IOTHUB_DEVICE_CONNECTION_STRING=<YourDeviceConnectionString>
    ```

1. Ejecute el dispositivo termostato de ejemplo con el siguiente comando:

    ```cmd/sh
    node simple_thermostat.js
    ```

1. Verá mensajes que señalan que el dispositivo ha enviado cierta información y ha indicado que está en línea. Estos mensajes indican que el dispositivo ha comenzado a enviar datos de telemetría al centro y que ya está listo para recibir comandos y actualizaciones de propiedades. No cierre este terminal, ya que lo necesita para confirmar que el ejemplo del servicio funciona.

## <a name="run-the-sample-solution"></a>Ejecución de la solución de ejemplo

En este inicio rápido, se usa una solución de IoT de ejemplo en Node.js para interactuar con el dispositivo de ejemplo que acaba de configurar.

1. Abra otra ventana de terminal para utilizarla como terminal de **servicio**. El SDK del servicio se encuentra en versión preliminar, por lo que debe clonar los ejemplos desde una [rama de versión preliminar del SDK de Node](https://github.com/Azure/azure-iot-sdk-node/tree/pnp-preview-refresh):

    ```cmd/sh
    git clone https://github.com/Azure/azure-iot-sdk-node -b pnp-preview-refresh
    ```

1. Vaya a la carpeta de esta rama clonada del repositorio y vaya a la carpeta */azure-iot-sdk-node/digitaltwins/samples/service/javascript*. Instale todas las dependencias, para lo que debe ejecutar el siguiente comando:

    ```cmd/sh
    npm install
    ```

1. Configure las variables de entorno para el identificador del dispositivo y _la cadena de conexión de IoT Hub_:

    ```cmd/sh
    set IOTHUB_CONNECTION_STRING=<YourIOTHubConnectionString>
    set IOTHUB_DEVICE_ID=<Your device ID>
    ```

### <a name="read-a-property"></a>Lectura de una propiedad

1. Al ejecutar el dispositivo termostato de ejemplo en el terminal del **dispositivo**, vio los siguientes mensajes que indicaban su estado en línea:

    ```cmd/sh
    properties have been reported for component
    sending telemetry message 0...
    ```

1. Vaya al terminal de **servicio** y use el siguiente comando para ejecutar el ejemplo para leer información del dispositivo:

    ```cmd/sh
    node get_digital_twin.js
    ```

1. En la salida del terminal de **servicio**, observe la respuesta del gemelo digital. Se muestra el identificador de modelo del dispositivo y las propiedades asociadas notificadas:

    ```json
    "$dtId": "mySimpleThermostat",
    "serialNumber": "123abc",
    "maxTempSinceLastReboot": 51.96167432818655,
    "$metadata": {
      "$model": "dtmi:com:example:Thermostat;1",
      "serialNumber": { "lastUpdateTime": "2020-07-09T14:04:00.6845182Z" },
      "maxTempSinceLastReboot": { "lastUpdateTime": "2020-07-09T14:04:00.6845182" }
    }
    ```

1. En el fragmento de código siguiente se muestra el código del archivo *get_digital_twin.js*, que recupera el identificador de modelo del gemelo del dispositivo:

    ```javascript
    console.log("Model Id: " + inspect(digitalTwin.$metadata.$model))
    ```

En este escenario, la salida es `Model Id: dtmi:com:example:Thermostat;1`.

### <a name="update-a-writable-property"></a>Actualización de una propiedad editable

1. Abra el archivo *update_digital_twin.js* en un editor de código.

1. Revise el código de ejemplo. Puede ver cómo crear una revisión de JSON para actualizar el gemelo digital del dispositivo. En este ejemplo, el código reemplaza la temperatura del termostato por el valor 42:

    ```javascript
    const patch = [{
        op: 'add',
        path: '/targetTemperature',
        value: '42'
      }]
    ```

1. En el terminal de **servicio**, use el siguiente comando para ejecutar el ejemplo para actualizar la propiedad:

    ```cmd/sh
    node update_digital_twin.js
    ```

1. En el terminal del **dispositivo**, se muestra que el dispositivo ha recibido la actualización:

    ```cmd/sh
    The following properties will be updated for root interface:
    {
      targetTemperature: {
        value: 42,
        ac: 200,
        ad: 'Successfully executed patch for targetTemperature',
        av: 2
      }
    }
    updated the property
    Properties have been reported for component
    ```

1. En el terminal de **servicio**, ejecute el siguiente comando para comprobar que la propiedad está actualizada:

    ```cmd/sh
    node get_digital_twin.js
    ```

1. En la salida del terminal de **servicio**, en la respuesta del gemelo digital bajo el componente `thermostat1`, se notifica la temperatura de destino actualizada. El dispositivo puede tardar cierto tiempo en finalizar la actualización. Repita este paso hasta que el dispositivo haya procesado la actualización de la propiedad:

    ```json
    targetTemperature: 42,
    ```

### <a name="invoke-a-command"></a>Invocación de un comando

1. Abra el archivo *invoke_command.js* y revise el código.

1. Vaya al terminal de **servicio**. Use el siguiente comando para ejecutar el ejemplo para invocar el comando:

    ```cmd/sh
    set IOTHUB_COMMAND_NAME=getMaxMinReport
    set IOTHUB_COMMAND_PAYLOAD=commandpayload
    node invoke_command.js
    ```

1. La salida del terminal de **servicio** muestra la siguiente confirmación:

    ```cmd/sh
    {
        xMsCommandStatuscode: 200,  
        xMsRequestId: 'ee9dd3d7-4405-4983-8cee-48b4801fdce2',  
        connection: 'close',  'content-length': '18',  
        'content-type': 'application/json; charset=utf-8',  
        date: 'Thu, 09 Jul 2020 15:05:14 GMT',  
        server: 'Microsoft-HTTPAPI/2.0',  vary: 'Origin',  
        body: 'min/max response'
    }
    ```

1. En el terminal del **dispositivo**, se muestra que el comando se ha confirmado:

    ```cmd/sh
    MaxMinReport commandpayload
    Response to method 'getMaxMinReport' sent successfully.
    ```

[!INCLUDE [iot-pnp-clean-resources.md](../../includes/iot-pnp-clean-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a conectar un dispositivo IoT Plug and Play a una solución de IoT. Para más información acerca de los modelos de dispositivo IoT Plug and Play, consulte:

> [!div class="nextstepaction"]
> [Guía para desarrolladores de modelado de IoT Plug and Play (versión preliminar)](concepts-developer-guide.md)
