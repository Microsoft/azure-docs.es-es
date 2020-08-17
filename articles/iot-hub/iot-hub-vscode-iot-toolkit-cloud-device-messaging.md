---
title: Uso de Azure IoT Tools para VSCode para la mensajería del Hub de TI de administrador
description: Aprenda a usar Azure IoT Tools para Visual Studio Code con el fin de supervisar los mensajes del dispositivo a la nube y enviar mensajes de la nube al dispositivo en Azure IoT Hub.
author: formulahendry
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 01/18/2019
ms.author: junhan
ms.custom:
- 'Role: Cloud Development'
ms.openlocfilehash: b9d8f0c2a573ba1749b414a389cd7181ddd5f95a
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87321690"
---
# <a name="use-azure-iot-tools-for-visual-studio-code-to-send-and-receive-messages-between-your-device-and-iot-hub"></a>Uso de Azure IoT Tools para Visual Studio Code a fin de enviar y recibir mensajes entre el dispositivo e IoT Hub

![Diagrama integral](./media/iot-hub-vscode-iot-toolkit-cloud-device-messaging/e-to-e-diagram.png)

[Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-toolkit) es una útil extensión de Visual Studio Code que facilita la administración de IoT Hub y el desarrollo de aplicaciones de IoT. Este artículo trata de cómo utilizar Azure IoT Tools para Visual Studio Code a fin de enviar y recibir mensajes entre el dispositivo e IoT Hub.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

## <a name="what-you-will-learn"></a>Aprendizaje

Aprenderá a usar Azure IoT Tools para Visual Studio Code a fin de supervisar los mensajes del dispositivo a la nube y de enviar mensajes de la nube al dispositivo. Los mensajes del dispositivo a la nube podrían ser datos de sensor que el dispositivo recopila y, a continuación, envía al IoT Hub. Los mensajes de la nube al dispositivo podrían ser comandos que el IoT Hub envía al dispositivo para hacer parpadear un LED que está conectado al dispositivo.

## <a name="what-you-will-do"></a>Lo que hará

* Utilizará Azure IoT Tools para Visual Studio Code a fin de supervisar los mensajes del dispositivo a la nube.

* Utilizará Azure IoT Tools para Visual Studio Code a fin de enviar mensajes de la nube al dispositivo.

## <a name="what-you-need"></a>Lo que necesita

* Una suscripción de Azure activa.

* Un centro de Azure IoT en su suscripción.

* [Visual Studio Code](https://code.visualstudio.com/)

* [Azure IoT Tools para VS Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools) o copie esta URL y péguela en una ventana del explorador: `vscode:extension/vsciot-vscode.azure-iot-tools`.

## <a name="sign-in-to-access-your-iot-hub"></a>Iniciar sesión para acceder a IoT Hub

1. En la vista **Explorador** de VS Code, expanda la sección **Azure IoT Hub Devices** (Dispositivos de Azure IoT Hub) en la esquina inferior izquierda.

2. Haga clic en **Select IoT Hub** (Seleccionar IoT Hub) en el menú contextual.

3. Se mostrará una ventana emergente en la esquina inferior derecha que le permite iniciar sesión en Azure por primera vez.

4. Después de iniciar sesión, se mostrará la lista de suscripciones de Azure y luego podrá seleccionar la suscripción de Azure e IoT Hub.

5. En unos segundos, aparecerá la lista de dispositivos en la pestaña **Azure IoT Hub Devices** (Dispositivos de Azure IoT Hub).

   > [!Note]
   > También puede completar la configuración seleccionando **Set IoT Hub Connection String** (Establecer cadena de conexión de IoT Hub). Escriba la cadena de conexión de la directiva **iothubowner** del centro de IoT al que se conecta el dispositivo IoT en la ventana emergente.

## <a name="monitor-device-to-cloud-messages"></a>Supervisión de mensajes de dispositivo a nube

Para supervisar los mensajes que se envían desde el dispositivo al IoT Hub, siga estos pasos:

1. Haga clic con el botón derecho en el dispositivo y seleccione **Start Monitoring Built-in Event Endpoint** (Iniciar supervisión del punto de conexión del evento integrado).

2. Se mostrarán los mensajes controlados en la vista **SALIDA** > **Azure IoT Hub**.

3. Para detener la supervisión, haga clic con el botón derecho en la vista **SALIDA** y seleccione **Stop Monitoring Built-in Event Endpoint** (Detener supervisión del punto de conexión del evento integrado).

## <a name="send-cloud-to-device-messages"></a>Envío de mensajes de nube a dispositivo

Para enviar un mensaje desde el IoT Hub al dispositivo, siga estos pasos:

1. Haga clic con el botón derecho en el dispositivo y seleccione **Send C2D Message to Device** (Enviar mensaje de C2D al dispositivo).

2. Escriba el mensaje en el cuadro de entrada.

3. Se mostrarán los resultados en la vista **SALIDA** > **Azure IoT Hub**.

## <a name="next-steps"></a>Pasos siguientes

Ahora sabe cómo supervisar los mensajes del dispositivo a la nube y enviar mensajes de la nube al dispositivo entre el dispositivo de IoT y la instancia de Azure IoT Hub.

[!INCLUDE [iot-hub-get-started-next-steps](../../includes/iot-hub-get-started-next-steps.md)]