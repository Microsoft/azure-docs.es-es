---
title: Conexión de MXChip IoT DevKit a Azure IoT Hub Remote Monitoring
description: En este tutorial, aprenda cómo enviar el estado de los sensores de IoT DevKit AZ3166 al acelerador de la solución de supervisión remota de Azure IoT.
author: liydu
manager: jeffya
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 02/02/2018
ms.author: liydu
ms.openlocfilehash: d75e7e7b4870f46c978fd70039441c8851844082
ms.sourcegitcommit: 46f8457ccb224eb000799ec81ed5b3ea93a6f06f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87337966"
---
# <a name="connect-mxchip-iot-devkit-to-azure-iot-remote-monitoring-solution-accelerator"></a>Conexión de MXChip IoT DevKit al acelerador de la solución de supervisión remota de Azure IoT

En este tutorial, aprenderá a ejecutar una aplicación de ejemplo en DevKit para enviar datos de sensor a su acelerador de la solución de supervisión remota de Azure IoT.

[MXChip IoT DevKit](https://aka.ms/iot-devkit) es una placa integral compatible de Arduino con periféricos y sensores varios. Se puede desarrollar para ella con la [extensión de Visual Studio Code para Arduino](https://aka.ms/arduino). Incluye un [catálogo de proyectos](https://microsoft.github.io/azure-iot-developer-kit/docs/projects/) cada vez mayor que sirve de guía para crear prototipos de soluciones de Internet de las cosas (IoT) que aprovechen los servicios de Microsoft Azure.

## <a name="what-you-need"></a>Lo que necesita

Siga la [Guía de introducción](https://docs.microsoft.com/azure/iot-hub/iot-hub-arduino-iot-devkit-az3166-get-started) para:

* Conexión de DevKit con Wi-Fi
* Preparación del entorno de desarrollo

Una suscripción de Azure activa. Si no tiene una, puede registrarse a través de uno de estos dos métodos:

* Active una [cuenta de Microsoft Azure de prueba de 30 días gratis](https://azure.microsoft.com/free/)

* Reclame su [crédito de Azure](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) si es suscriptor de MSDN o Visual Studio

## <a name="create-an-azure-iot-remote-monitoring-solution-accelerator"></a>Creación de un acelerador de la solución de supervisión de Azure IoT

1. Vaya al [sitio de los aceleradores de la solución de Azure IoT](https://www.azureiotsolutions.com/) y haga clic en **Crear una solución nueva**.

   ![Selección del tipo de acelerador de soluciones de Azure IoT](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/azure-iot-suite-solution-types.png)

   > [!WARNING]
   > De forma predeterminada, este ejemplo crea una instancia de S2 IoT Hub después de crear un acelerador de la solución de supervisión remota de IoT. Si esta instancia de IoT Hub no se utiliza con un gran número de dispositivos, se recomienda cambiar de S2 a S1 y eliminar el acelerador de la solución de supervisión remota de IoT para poder eliminar también la instancia de IoT Hub relacionada cuando ya no la necesite. 

2. Seleccione **Supervisión remota**.

3. Escriba un nombre para la solución, seleccione una suscripción y una región y después haga clic en **Crear solución**. La solución puede tardar varios minutos en aprovisionar.
  
   ![Crear solución](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/azure-iot-suite-new-solution.png)

4. Cuando finalice el aprovisionamiento, haga clic en **Iniciar**. Algunos dispositivos simulados se crean para la solución durante el proceso de aprovisionamiento. Haga clic en **DISPOSITIVOS** para revisarlos.

   ![Panel](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/azure-iot-suite-new-solution-created.png)
  
   ![Consola](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/azure-iot-suite-console.png)

5. Haga clic en **AGREGAR UN DISPOSITIVO**.

6. Haga clic en **Añadir nuevo** para **Dispositivo personalizado**.
  
   ![Adición de un nuevo dispositivo](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/azure-iot-suite-add-new-device.png)

7. Haga clic en **Permitirme definir mi propio Id. de dispositivo**, escriba `AZ3166`y, a continuación, haga clic en **Crear**.
  
   ![Creación de un dispositivo con identificador](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/azure-iot-suite-new-device-configuration.png)

8. Tome nota del **nombre de host de IoT Hub** y haga clic en **Listo**.

## <a name="open-the-remotemonitoring-sample"></a>Apertura del ejemplo de RemoteMonitoring

1. Desconecte el DevKit del equipo, si está conectado.

2. Inicie VS Code.

3. Conecte el DevKit al equipo. Código de VS detecta automáticamente el DevKit y abre las páginas siguientes:

   * Página de introducción del DevKit.
   * Ejemplos de Arduino: ejemplos prácticos para empezar a trabajar con DevKit.

4. Expanda la sección de la izquierda **EJEMPLOS DE ARDUINO**, vaya a **Ejemplos para MXCHIP AZ3166 > AzureIoT** y seleccione **RemoteMonitoring**. Se abre una nueva ventana de VS Code que contiene la carpeta de proyecto.

   > [!NOTE]
   > Si cierra el panel, puede volver a abrirlo. Utilice `Ctrl+Shift+P` (macOS: `Cmd+Shift+P`) para abrir la paleta de comandos, escriba **Arduino** y, a continuación, busque y seleccione **Arduino: Ejemplos**.

## <a name="provision-required-azure-services"></a>Aprovisionamiento de los servicios de Azure necesarios

En la ventana de la solución, ejecute la tarea mediante `Ctrl+P` (macOS: `Cmd+P`) escribiendo `task cloud-provision`en el cuadro de texto proporcionado.

En el terminal de VS Code, una línea de comandos interactiva le guiará durante el proceso de aprovisionamiento de los servicios de Azure correspondientes.

![Aprovisionamiento de los recursos de Azure](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/provision.png)

## <a name="build-and-upload-the-device-code"></a>Creación y carga del código del dispositivo

1. Use `Ctrl+P` (macOS: `Cmd + P`) y escriba **task config-device-connection**.

2. El terminal le pregunta si desea usar la cadena de conexión que se recupera en el paso `task cloud-provision`. También podría introducir su propia cadena de conexión de dispositivo, haciendo clic en "Create New..." (Crear nuevo...)

3. El terminal le pide que especifique el modo de configuración. Para ello, mantenga presionado el botón A, y presione y suelte el botón de restablecimiento. La pantalla muestra el identificador de DevKit y "Configuration" (Configuración).

   ![Cadena de conexión de entrada](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/config-device-connection.png)

4. Cuando termine `task config-device-connection`, haga clic en `F1` para cargar los comandos de VS Code y seleccione `Arduino: Upload`. VS Code inicia la comprobación y la carga del boceto de Arduino.
  
   ![Comprobación y carga del boceto de Arduino](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/arduino-upload.png)

El DevKit se reinicia y comienza a ejecutar el código.

## <a name="test-the-project"></a>Prueba del proyecto

Cuando se ejecuta la aplicación de ejemplo, DevKit envía datos de sensor vía Wi-Fi al acelerador de la solución de supervisión remota de Azure IoT. Para ver el resultado, siga estos pasos:

1. Vaya al acelerador de la solución de supervisión remota de Azure IoT y haga clic en **PANEL**.

2. En la consola de solución de supervisión remota, verá el estado del sensor de DevKit.

   ![Datos de sensor en el acelerador de la solución de supervisión remota de Azure IoT](media/iot-hub-arduino-iot-devkit-az3166-devkit-remote-monitoring/sensor-status.png)

## <a name="change-device-id"></a>Cambio del identificador del dispositivo

Si desea cambiar el valor **AZ3166** codificado de forma rígida a un ID de dispositivo personalizado en el código, modifique la línea de código que se muestra en el [ejemplo de supervisión remota](../iot-accelerators/iot-accelerators-arduino-iot-devkit-az3166-devkit-remote-monitoring-v2.md).

## <a name="problems-and-feedback"></a>Problemas y comentarios

Si tiene problemas, consulte las [preguntas frecuentes del kit para desarrolladores de IoT](https://microsoft.github.io/azure-iot-developer-kit/docs/faq/) o póngase en contacto con nosotros mediante los siguientes canales:

* [Gitter.im](https://gitter.im/Microsoft/azure-iot-developer-kit)
* [Stack Overflow](https://stackoverflow.com/questions/tagged/iot-devkit)

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido cómo conectar un dispositivo DevKit al acelerador de la solución de supervisión remota de Azure IoT y visualizar los datos de sensor, estos son los siguientes pasos sugeridos:

* [Introducción a los aceleradores de la solución de Azure IoT](https://docs.microsoft.com/azure/iot-suite/)

* [Kit para desarrolladores de IoT](https://microsoft.github.io/azure-iot-developer-kit/) 
