---
title: Conexión del simulador web de Raspberry Pi a Azure IoT Hub (Node.js)
description: Conectar el simulador web de Raspberry Pi a Azure IoT Hub para que Raspberry Pi envíe datos a la nube de Azure.
author: wesmc7777
manager: philmea
keywords: simulador de raspberry pi, azure iot raspberry pi, raspberry pi iot hub, raspberry pi envía datos a la nube, raspberry pi a la nube
ms.service: iot-hub
services: iot-hub
ms.devlang: nodejs
ms.topic: conceptual
ms.date: 04/11/2018
ms.author: wesmc
ms.custom:
- 'Role: Cloud Development'
- devx-track-javascript
ms.openlocfilehash: d1b3858e62902a5c1c841de7618966e695b66425
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87423072"
---
# <a name="connect-raspberry-pi-online-simulator-to-azure-iot-hub-nodejs"></a>Conectar el simulador en línea de Raspberry Pi a Azure IoT Hub (Node.js)

[!INCLUDE [iot-hub-get-started-device-selector](../../includes/iot-hub-get-started-device-selector.md)]

En este tutorial, empezará por aprender los principios básicos del uso del simulador en línea de Raspberry Pi. A continuación, aprenderá a conectar sin problemas el simulador Pi a la nube con [Azure IoT Hub](about-iot-hub.md).

<p>
<div id="diag" style="width:100%; text-align:center">
<img src="media/iot-hub-raspberry-pi-web-simulator/3-banner.png" alt="Connect Raspberry Pi web simulator to Azure IoT Hub" width="400">
</div>
</p>
<p>
<div id="button" style="width:100%; text-align:center">
<a href="https://azure-samples.github.io/raspberry-pi-web-simulator/#getstarted" target="_blank">
<img src="media/iot-hub-raspberry-pi-web-simulator/6-button-default.png" alt="Start Raspberry Pi simulator" width="400" onmouseover="this.src='media/iot-hub-raspberry-pi-web-simulator/5-button-click.png';" onmouseout="this.src='media/iot-hub-raspberry-pi-web-simulator/6-button-default.png';">
</a>
</div>
</p>

Si tiene dispositivos físicos, visite [Conectar Raspberry Pi a Azure IoT Hub](iot-hub-raspberry-pi-kit-node-get-started.md) para comenzar.

## <a name="what-you-do"></a>Qué debe hacer

* Conozca los aspectos fundamentales del simulador en línea de Raspberry Pi.

* Cree un Centro de IoT.

* Registre un dispositivo para Pi en IoT Hub.

* Ejecuta una aplicación de ejemplo en PI para enviar datos de sensor simulados a IoT Hub.

Conecte Raspberry Pi simulado al IoT Hub que ha creado. A continuación, ejecute una aplicación de ejemplo con el simulador para generar datos de sensor. Por último, envíe los datos del sensor a IoT Hub.

## <a name="what-you-learn"></a>Conocimientos que adquirirá

* Cómo crear Azure IoT Hub y obtener la cadena de conexión del nuevo dispositivo. Si no tiene ninguna cuenta de Azure, [cree una cuenta de evaluación gratuita de Azure](https://azure.microsoft.com/free/) en solo unos minutos.

* Cómo trabajar con el simulador en línea de Raspberry Pi.

* Cómo enviar los datos del sensor a IoT Hub.

## <a name="overview-of-raspberry-pi-web-simulator"></a>Introducción al simulador web de Raspberry Pi

Haga clic en el botón para iniciar el simulador en línea de Raspberry Pi.

> [!div class="button"]
> <a href="https://azure-samples.github.io/raspberry-pi-web-simulator/#GetStarted" target="_blank">Iniciar simulador de Raspberry Pi</a>

Hay tres áreas en el simulador web.

1. Área de ensamblado: el circuito predeterminado es que Pi se conecta con un sensor BME280 y un LED. Esta área está bloqueada en la versión preliminar por lo que actualmente no puede hacer ninguna personalización.

2. Área de codificación: un editor de código en línea para codificar con Raspberry Pi. La aplicación de ejemplo predeterminada ayuda a recopilar datos de sensor desde el sensor BME280 y los envía a Azure IoT Hub. La aplicación es totalmente compatible con dispositivos Pi reales. 

3. Ventana de consola integrada: muestra la salida del código. En la parte superior de esta ventana aparecen tres botones.

   * **Run** (Ejecutar): ejecute la aplicación en el área de codificación.

   * **Reset** (Restablecer): restablezca el área de codificación a la aplicación de ejemplo predeterminada.

   * **Fold/Expand** (Plegar/expandir): al lado derecho hay un botón para plegar o expandir la ventana de consola.

> [!NOTE]
> El simulador web de Raspberry Pi ahora está disponible en versión preliminar. Nos gustaría oír su voz en la [sala de chat de Gitter](https://gitter.im/Microsoft/raspberry-pi-web-simulator). El código fuente es público en [GitHub](https://github.com/Azure-Samples/raspberry-pi-web-simulator).

![Introducción al simulador en línea de Pi](media/iot-hub-raspberry-pi-web-simulator/0-overview.png)

## <a name="create-an-iot-hub"></a>Crear un centro de IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-new-device-in-the-iot-hub"></a>Registro de un nuevo dispositivo en el centro de IoT

[!INCLUDE [iot-hub-include-create-device](../../includes/iot-hub-include-create-device.md)]

## <a name="run-a-sample-application-on-pi-web-simulator"></a>Ejecución de una aplicación de ejemplo en el simulador web de Pi

1. En el área de codificación, asegúrese de trabajar en la aplicación de ejemplo predeterminada. Reemplace el marcador de posición de la línea 15 con la cadena de conexión del dispositivo de Azure IoT Hub.
1. 
   ![Reemplazo de la cadena de conexión de dispositivo](media/iot-hub-raspberry-pi-web-simulator/1-connectionstring.png)

2. Seleccione **Ejecutar** o escriba `npm start` para ejecutar la aplicación.

Debería ver el resultado siguiente que muestra los datos de sensor y los mensajes que se envían a IoT Hub ![Resultado: datos de sensor enviados desde Raspberry Pi a IoT Hub](media/iot-hub-raspberry-pi-web-simulator/2-run-application.png)

## <a name="read-the-messages-received-by-your-hub"></a>Lectura de los mensajes recibidos por IoT Hub

Una forma de supervisar los mensajes que recibe el centro de IoT del dispositivo simulado consiste en usar Azure IoT Tools para Visual Studio Code. Para más información, vea [Uso de Azure IoT Tools para Visual Studio Code a fin de enviar y recibir mensajes entre el dispositivo e IoT Hub](iot-hub-vscode-iot-toolkit-cloud-device-messaging.md).

Para obtener más formas de procesar los datos enviados por el dispositivo, continúe con la sección siguiente.

## <a name="next-steps"></a>Pasos siguientes

Ha ejecutado una aplicación de ejemplo para recopilar datos de sensor y enviarlos a IoT Hub.

[!INCLUDE [iot-hub-get-started-next-steps](../../includes/iot-hub-get-started-next-steps.md)]
