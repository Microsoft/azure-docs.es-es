---
title: Comprobación de la conectividad de los dispositivos a Azure IoT Hub
description: Use las herramientas de IoT Hub para solucionar problemas relacionados con el desarrollo y la conectividad de dispositivos a IoT Hub.
services: iot-hub
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.custom:
- mvc
- amqp
- mqtt
- 'Role: Cloud Development'
- 'Role: IoT Device'
- devx-track-javascript
- devx-track-azurecli
ms.date: 02/22/2019
ms.topic: tutorial
ms.service: iot-hub
ms.openlocfilehash: f6ac83a0fc32b426a914b76d27c2920ff8ce6731
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87500255"
---
# <a name="tutorial-use-a-simulated-device-to-test-connectivity-with-your-iot-hub"></a>Tutorial: Uso de un dispositivo simulado para probar la conectividad con IoT Hubs

En este tutorial, usará herramientas del portal de Azure IoT Hub y comandos de la CLI de Azure para probar la conectividad de los dispositivos. En este tutorial también se usa un simulador de dispositivos sencillo que se ejecuta en la máquina de escritorio.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Comprobar la autenticación del dispositivo
> * Comprobar la conectividad del dispositivo a la nube
> * Comprobación de la conectividad de la nube al dispositivo
> * Comprobar la sincronización de dispositivos gemelos

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="prerequisites"></a>Requisitos previos

Los scripts de la CLI que se ejecutan en este tutorial usan la [extensión de Microsoft Azure IoT para la CLI de Azure](https://github.com/Azure/azure-iot-cli-extension/blob/master/README.md). Para instalar esta extensión, ejecute el siguiente comando de la CLI:

```azurecli-interactive
az extension add --name azure-iot
```

[!INCLUDE [iot-hub-cli-version-info](../../includes/iot-hub-cli-version-info.md)]

La aplicación de simulación de dispositivos que se ejecuta en este tutorial está escrita con Node.js. Necesitará Node.js v10.x.x o una versión posterior en la máquina de desarrollo.

Puede descargar Node.js para varias plataformas desde [nodejs.org](https://nodejs.org).

Puede verificar la versión actual de Node.js en el equipo de desarrollo con el comando siguiente:

```cmd/sh
node --version
```

Descargue el simulador de dispositivos de ejemplo de Node.js desde https://github.com/Azure-Samples/azure-iot-samples-node/archive/master.zip y extraiga el archivo ZIP.

Asegúrese de que está abierto el puerto 8883 del firewall. En el dispositivos de ejemplo de este tutorial se usa el protocolo MQTT, que se comunica mediante el puerto 8883. Este puerto puede estar bloqueado en algunos entornos de red corporativos y educativos. Para más información y saber cómo solucionar este problema, consulte [Conexión a IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="create-an-iot-hub"></a>Crear un centro de IoT

Si ha creado un centro de IoT de nivel gratuito o estándar en un inicio rápido o tutorial anterior, puede omitir este paso.

[!INCLUDE [iot-hub-tutorials-create-free-hub](../../includes/iot-hub-tutorials-create-free-hub.md)]

## <a name="check-device-authentication"></a>Comprobación de la autenticación de dispositivos

Un dispositivo debe autenticarse con el centro antes de poder intercambiar datos con él. Para administrar los dispositivos y comprobar las claves de autenticación que usan, puede emplear la herramienta **Dispositivos IoT** de la sección **Administración de dispositivos** del portal. En esta sección del tutorial, agregará un nuevo dispositivo de prueba, recuperará su clave y comprobará que el dispositivo de prueba puede conectarse al centro. Más adelante, restablecerá la clave de autenticación para observar lo que sucede cuando un dispositivo intenta usar una clave no actualizada. En esta sección del tutorial se usa Azure Portal para crear, administrar y supervisar un dispositivo, y el simulador de dispositivos de ejemplo de Node.js.

Inicie sesión en el portal y desplácese hasta su centro de IoT. A continuación, vaya a la herramienta **Dispositivos IoT**:

![Herramienta Dispositivos IoT](media/tutorial-connectivity/iot-devices-tool.png)

Para registrar un nuevo dispositivo, haga clic en **+ Agregar**, establezca **Id. de dispositivo** en **MyTestDevice** y haga clic en **Guardar**:

![Adición de un nuevo dispositivo](media/tutorial-connectivity/add-device.png)

Para recuperar la cadena de conexión de **MyTestDevice**, haga clic en ella en la lista de dispositivos y, a continuación, copie el valor de **Cadena de conexión: clave principal**. La cadena de conexión incluye la *clave de acceso compartido* del dispositivo.

![Recuperación de la cadena de conexión del dispositivo](media/tutorial-connectivity/copy-connection-string.png)

Para simular el envío de telemetría de **MyTestDevice** a su centro de IoT, ejecute la aplicación de dispositivo simulado de Node.js que descargó anteriormente.

En una ventana de terminal de su máquina de desarrollo, desplácese a la carpeta raíz del proyecto de ejemplo de Node.js que descargó. A continuación, vaya a la carpeta **iot-hub\Tutorials\ConnectivityTests**.

En la ventana de terminal, ejecute los comandos siguientes para instalar las bibliotecas necesarias y ejecute la aplicación de dispositivo simulado. Use la cadena de conexión del dispositivo que anotó cuando agregó el dispositivo en el portal.

```cmd/sh
npm install
node SimulatedDevice-1.js "{your device connection string}"
```

En la ventana de terminal se muestra información a medida que se intenta conectar con su centro:

![Conexión del dispositivo simulado](media/tutorial-connectivity/sim-1-connected.png)

Ahora se ha autenticado correctamente desde un dispositivo mediante una clave de dispositivo generada por el centro de IoT.

### <a name="reset-keys"></a>Restablecimiento de claves

En esta sección, restablecerá la clave de dispositivo y observará el error cuando el dispositivo simulado intenta conectarse.

Para restablecer la clave de dispositivo principal para **MyTestDevice**, ejecute los siguientes comandos:

```azurecli-interactive
# Generate a new Base64 encoded key using the current date
read key < <(date +%s | sha256sum | base64 | head -c 32)

# Requires the IoT Extension for Azure CLI
# az extension add --name azure-iot

# Reset the primary device key for MyTestDevice
az iot hub device-identity update --device-id MyTestDevice --set authentication.symmetricKey.primaryKey=$key --hub-name {YourIoTHubName}
```

En la ventana de terminal de su máquina de desarrollo, ejecute de nuevo la aplicación de dispositivo simulado:

```cmd/sh
npm install
node SimulatedDevice-1.js "{your device connection string}"
```

Esta vez verá un error de autenticación cuando la aplicación intenta conectarse:

![Error de conexión](media/tutorial-connectivity/sim-1-fail.png)

### <a name="generate-shared-access-signature-sas-token"></a>Generación de un token de firma de acceso compartido (SAS)

Si el dispositivo usa uno de los SDK de dispositivo de IoT Hub, el código de biblioteca del SDK genera el token de SAS usado para autenticarse con el centro. Un token de SAS se genera a partir del nombre del centro, el nombre del dispositivo y la clave del dispositivo.

En algunos escenarios, como en una puerta de enlace de protocolo de nube o como parte de un esquema de autenticación personalizado, puede que deba generar el token de SAS por su cuenta. Para solucionar problemas con el código de generación de SAS, es útil generar un token de SAS conocido para usar durante las pruebas.

> [!NOTE]
> El ejemplo SimulatedDevice 2.js incluye ejemplos de generación de un token de SAS con y sin el SDK.

Para generar un token de SAS conocido mediante la CLI, ejecute el siguiente comando:

```azurecli-interactive
az iot hub generate-sas-token --device-id MyTestDevice --hub-name {YourIoTHubName}
```

Anote el texto completo del token de SAS generado. Un token de SAS se parece a este: `SharedAccessSignature sr=tutorials-iot-hub.azure-devices.net%2Fdevices%2FMyTestDevice&sig=....&se=1524155307`

En una ventana de terminal de su máquina de desarrollo, desplácese a la carpeta raíz del proyecto de ejemplo de Node.js que descargó. A continuación, vaya a la carpeta **iot-hub\Tutorials\ConnectivityTests**.

En la ventana de terminal, ejecute los comandos siguientes para instalar las bibliotecas necesarias y ejecute la aplicación de dispositivo simulado:

```cmd/sh
npm install
node SimulatedDevice-2.js "{Your SAS token}"
```

En la ventana de terminal se muestra información a medida que se intenta conectar con su centro mediante el token de SAS:

![Conexión del dispositivo simulado con token](media/tutorial-connectivity/sim-2-connected.png)

Ahora se ha autenticado correctamente desde un dispositivo mediante un token de SAS de prueba generado por un comando de la CLI. El archivo **SimulatedDevice-2.js** incluye código de ejemplo que muestra cómo generar un token de SAS en el código.

### <a name="protocols"></a>Protocolos

Un dispositivo puede usar cualquiera de los siguientes protocolos para conectarse al centro de IoT:

| Protocolo | Puerto de salida |
| --- | --- |
| MQTT |8883 |
| MQTT sobre WebSockets |443 |
| AMQP |5671 |
| AMQP sobre WebSockets |443 |
| HTTPS |443 |

Si el puerto de salida está bloqueado por un firewall, el dispositivo no se puede conectar:

![Puerto bloqueado](media/tutorial-connectivity/port-blocked.png)

## <a name="check-device-to-cloud-connectivity"></a>Comprobar la conectividad del dispositivo a la nube

Después de que se conecta un dispositivo, normalmente intenta enviar telemetría al centro de IoT. En esta sección se muestra cómo puede comprobar que la telemetría enviada por el dispositivo llega al centro.

En primer lugar, recupere la cadena de conexión actual del dispositivo simulado mediante el siguiente comando:

```azurecli-interactive
az iot hub device-identity show-connection-string --device-id MyTestDevice --output table --hub-name {YourIoTHubName}
```

Para ejecutar un dispositivo simulado que envía mensajes, vaya hasta la carpeta **iot-hub\Tutorials\ConnectivityTests** en el código descargado.

En la ventana de terminal, ejecute los comandos siguientes para instalar las bibliotecas necesarias y ejecute la aplicación de dispositivo simulado:

```cmd/sh
npm install
node SimulatedDevice-3.js "{your device connection string}"
```

En la ventana de terminal se muestra información a medida que se envía telemetría a su centro:

![Dispositivo simulado enviando mensajes](media/tutorial-connectivity/sim-3-sending.png)

Puede usar **Métricas** en el portal para comprobar que los mensajes de telemetría llegan al centro de IoT. Seleccione el centro de IoT en la lista desplegable **Recurso**, seleccione **Telemetry messages sent** (Mensajes de telemetría enviados) como métrica y establezca el intervalo de tiempo en **Última hora**. En el gráfico se muestra el recuento total de mensajes enviados por el dispositivo simulado:

![Presentación de las métricas de IoT Hub](media/tutorial-connectivity/metrics-portal.png)

Las métricas tardan unos minutos en estar disponibles después de iniciar el dispositivo simulado.

## <a name="check-cloud-to-device-connectivity"></a>Comprobación de la conectividad de la nube al dispositivo

En esta sección se muestra cómo realizar una llamada del método directo de prueba a un dispositivo para comprobar la conectividad de la nube al dispositivo. Ejecutará un dispositivo simulado en la máquina de desarrollo para escuchar llamadas del método directo desde su centro.

En la ventana de terminal, use el comando siguiente para ejecutar la aplicación de dispositivo simulado:

```cmd/sh
node SimulatedDevice-3.js "{your device connection string}"
```

Use un comando de la CLI para llamar a un método directo en el dispositivo:

```azurecli-interactive
az iot hub invoke-device-method --device-id MyTestDevice --method-name TestMethod --timeout 10 --method-payload '{"key":"value"}' --hub-name {YourIoTHubName}
```

El dispositivo simulado imprime un mensaje en la consola cuando recibe una llamada del método directo:

![El dispositivo simulado recibe la llamada del método directo](media/tutorial-connectivity/receive-method-call.png)

Cuando el dispositivo simulado recibe correctamente la llamada del método directo, envía una confirmación al centro:

![Recepción de la confirmación del método directo](media/tutorial-connectivity/method-acknowledgement.png)

## <a name="check-twin-synchronization"></a>Comprobación de sincronización de dispositivos gemelos

Los dispositivos usan gemelos para sincronizar el estado entre el dispositivo y el centro. En esta sección, usará comandos de la CLI para enviar las _propiedades deseadas_ a un dispositivo y leer las _propiedades notificadas_ enviadas por el dispositivo.

El dispositivo simulado que se usa en esta sección envía propiedades notificadas al centro cada vez que se inicia, e imprime las propiedades deseadas en la consola cada vez que las recibe.

En la ventana de terminal, use el comando siguiente para ejecutar la aplicación de dispositivo simulado:

```cmd/sh
node SimulatedDevice-3.js "{your device connection string}"
```

Para comprobar que el centro recibió las propiedades notificadas del dispositivo, use el siguiente comando de la CLI:

```azurecli-interactive
az iot hub device-twin show --device-id MyTestDevice --hub-name {YourIoTHubName}
```

En la salida del comando, puede ver la propiedad **devicelaststarted** en la sección de propiedades notificadas. Esta propiedad muestra la fecha y hora en que se inició por última vez el dispositivo simulado.

![Visualización de propiedades notificadas](media/tutorial-connectivity/reported-properties.png)

Para comprobar que el centro puede enviar valores de propiedad deseada al dispositivo, use el siguiente comando de la CLI:

```azurecli-interactive
az iot hub device-twin update --set properties.desired='{"mydesiredproperty":"propertyvalue"}' --device-id MyTestDevice --hub-name {YourIoTHubName}
```

El dispositivo simulado imprime un mensaje cuando recibe una actualización de propiedad deseada del centro:

![Recepción de propiedades deseadas](media/tutorial-connectivity/desired-properties.png)

Además de recibir los cambios en las propiedades deseadas a medida que se realizan, el dispositivo simulado busca automáticamente propiedades deseadas al inicio.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ya no los va a necesitar más, elimínelos en el portal. Para ello, seleccione el grupo de recursos **tutorials-iot-hub-rg** que contiene el centro de IoT y haga clic en **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha visto cómo comprobar las claves de dispositivo, la conectividad del dispositivo a la nube, la conectividad de la nube al dispositivo y la sincronización de dispositivos gemelos. Para aprender más sobre cómo supervisar el centro de IoT, visite el artículo sobre procedimientos relativo a la supervisión de IoT Hub.

> [!div class="nextstepaction"]
> [Supervisión con Diagnostics](iot-hub-monitor-resource-health.md)
