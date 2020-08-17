---
title: Uso de dispositivos gemelos de Azure para controlar el LED de usuario MXChip IoT DevKit |Microsoft Docs
description: En este tutorial, aprenderá a supervisar los estados de DevKit y a controlar el LED de usuario con dispositivos gemelos de Azure IoT Hub.
author: liydu
manager: jeffya
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 04/04/2018
ms.author: liydu
ms.openlocfilehash: 0d8e10a18436b0b52820dd0bf15ad0b2de969b79
ms.sourcegitcommit: 46f8457ccb224eb000799ec81ed5b3ea93a6f06f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87337949"
---
# <a name="mxchip-iot-devkit"></a>MXChip IoT DevKit

Este ejemplo se puede utilizar para supervisar la información del Wi-Fi de MXChip IoT DevKit y para controlar el color de lo LED de usuario mediante dispositivos gemelos de Azure IoT Hub.

## <a name="what-you-learn"></a>Conocimientos que adquirirá

- Cómo supervisar los estados del sensor de MXChip IoT DevKit.

- Cómo usar dispositivos gemelos de Azure para controlar el color del LED RGB de DevKit.

## <a name="what-you-need"></a>Lo que necesita

- Configurar el entorno de desarrollo siguiendo la [Guía de introducción](https://docs.microsoft.com/azure/iot-hub/iot-hub-arduino-iot-devkit-az3166-get-started).

- En la ventana de terminal GitBash (u otra interfaz de línea de comandos de Git), escriba los siguientes comandos:

   ```bash
   git clone https://github.com/DevKitExamples/DevKitState.git
   cd DevKitState
   code .
   ```

## <a name="provision-azure-services"></a>Aprovisionamiento de servicios de Azure

1. Haga clic en el menú desplegable **Tareas** de Visual Studio Code y seleccione **Ejecutar tarea...**  - **cloud-provision**.

2. El progreso se muestra en la pestaña **TERMINAL** del panel de **bienvenida**.

3. Cuando aparezca el mensaje *What subscription would you like to choose* (¿Qué suscripción desea?), seleccione una.

4. Seleccione o elija un grupo de recursos. 
 
   > [!NOTE]
   > Si ya tiene una instancia gratuita de IoT Hub, puede omitir este paso.

5. Cuando aparezca el mensaje *What IoT hub would you like to choose* (¿Qué instancia de IoT Hub desea?), seleccione una.

6. Se muestra algo similar a *aplicación de función: nombre de aplicación de función: xxx*. Anote el nombre de la aplicación de función, ya que se utilizará en un paso posterior.

7. Espere hasta que finalice la implementación de la plantilla de Azure Resource Manager, momento que se indica mediante el mensaje *Resource Manager template deployment: Done* (Se implementa la aplicación de función: listo).

## <a name="deploy-function-app"></a>Implementación de aplicación de función

1. Haga clic en el menú desplegable **Tareas** de Visual Studio Code y seleccione **Ejecutar tarea...**  - **cloud-deploy**.

2. Espere hasta que finalice el proceso de carga del código de la aplicación de función; se muestra el mensaje *function app deploys: Done* (Se implementa la aplicación de función: listo).

## <a name="configure-iot-hub-device-connection-string-in-devkit"></a>Configuración de la cadena de conexión de un dispositivo IoT Hub

1. Conecte MXChip IoT DevKit al equipo.

2. Haga clic en el menú desplegable **Tareas** de Visual Studio Code y seleccione **Ejecutar tarea...**  - **config-device-connection**.

3. En MXChip IoT DevKit, mantenga presionado el botón **A**, presione el botón **Reset** y suelte el botón **A** para que DekKit entre en el modo de configuración.

4. Espere hasta que se complete el proceso de configuración de la cadena de conexión.

## <a name="upload-arduino-code-to-devkit"></a>Carga de código de Arduino en DevKit

Con MXChip IoT DevKit conectado al equipo:

1. Haga clic en el menú desplegable **Tareas** de Visual Studio Code y seleccione **Ejecutar tarea de compilación...** El sketch de Arduino se compila y se carga en DevKit.

2. Cuando se haya cargado correctamente, se mostrará el mensaje *Build & Upload Sketch: success* (Compilación y carga de sketch:correctos).

## <a name="monitor-devkit-state-in-browser"></a>Supervisión del estado de DevKit en un explorador

1. En un explorador web, abra el archivo `DevKitState\web\index.html` (que se creó en el paso Lo que necesita).

2. Aparece la siguiente página web:![Especifique el nombre de la aplicación de función.](media/iot-hub-arduino-iot-devkit-az3166-devkit-state/devkit-state-function-app-name.png)

3. Escriba el nombre de la aplicación de función que anotó anteriormente.

4. Haga clic en el botón **Connect** (Conectar).

5. Al cabo de unos segundos, la página se actualiza y muestra tanto el estado de la conexión Wi-Fi de DevKit como el estado de cada uno de los sensores incorporados.

## <a name="control-the-devkits-user-led"></a>Control del LED de usuario de DevKit

1. Haga clic en el gráfico del LED de usuario de la ilustración de la página web.

2. Al cabo de unos segundos, la página se actualiza y muestra el estado de color actual del LED del usuario.

3. Pruebe a cambiar el valor de color del LED RGB, para lo que debe hacer clic en varias ubicaciones en los controles deslizantes de RGB.

## <a name="example-operation"></a>Operación de ejemplo

![Procedimiento de prueba de ejemplo](media/iot-hub-arduino-iot-devkit-az3166-devkit-state/devkit-state.gif)

> [!NOTE]
> Los datos sin procesar del dispositivo gemelo se pueden ver en Azure Portal: IoT Hub-\>dispositivos IoT -\> *\<your device\>*  -\> dispositivo gemelo.

## <a name="next-steps"></a>Pasos siguientes

Ha aprendido a:
- Conexión de un dispositivo MXChip IoT DevKit al acelerador de la solución de supervisión remota de Azure IoT.
- Usar la función de dispositivos gemelos de Azure IoT para detectar y controlar el color del LED RGB de DevKit.

Este es el siguiente paso que le sugerimos: [Información general sobre el acelerador de la solución de supervisión remota de Azure IoT](https://docs.microsoft.com/azure/iot-suite/)
