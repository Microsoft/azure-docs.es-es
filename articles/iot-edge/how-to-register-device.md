---
title: Registro de un nuevo dispositivo de Azure IoT Edge | Microsoft Docs
description: Uso de la extensión de IoT para la CLI de Azure para registrar un dispositivo IoT Edge nuevo y recuperar la cadena de conexión
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 01/08/2020
ms.topic: conceptual
ms.reviewer: menchi
ms.service: iot-edge
ms.custom: devx-track-azurecli
services: iot-edge
ms.openlocfilehash: 0f41dbb4e6231e804249c3fce3dfc8275dcc00aa
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87489017"
---
# <a name="register-an-azure-iot-edge-device"></a>Registro de un dispositivo de Azure IoT Edge

Para poder usar los dispositivos IoT con Azure IoT Edge, debe registrarlos con IoT Hub. Una vez registrado un dispositivo, puede recuperar una cadena de conexión para configurar el dispositivo para las cargas de trabajo de IoT Edge.

Tiene la opción de registrarse mediante una de las siguientes herramientas:

* [Registre un dispositivo en Azure Portal](#register-in-the-azure-portal) si prefiere una interfaz gráfica de usuario para crear, ver y administrar los recursos de Azure.
* [Registre un dispositivo con Visual Studio Code](#register-with-visual-studio-code) si prefiere administrar los recursos de Azure IoT en el mismo lugar donde desarrolla las soluciones de IoT.
* [Registre un dispositivo con la CLI de Azure](#register-with-the-azure-cli) si prefiere herramientas de la línea de comandos para administrar los recursos de Azure, o bien para automatizar las tareas.

## <a name="register-in-the-azure-portal"></a>Registro en Azure Portal

Puede realizar todas las tareas de registro en Azure Portal.

### <a name="prerequisites-for-the-azure-portal"></a>Requisitos previos para Azure Portal

Un [centro de IoT](../iot-hub/iot-hub-create-through-portal.md) gratuito o estándar en la suscripción de Azure.

### <a name="create-an-iot-edge-device-in-the-azure-portal"></a>Creación de un dispositivo IoT Edge en Azure Portal

En IoT Hub de Azure Portal, los dispositivos IoT Edge se crean y administran por separado de los dispositivos IOT que no están habilitados para Edge.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y vaya a IoT Hub.
2. En el panel izquierdo, seleccione **IoT Edge** en el menú.
3. Seleccione **Agregar un dispositivo IoT Edge**.
4. Proporcione un identificador de dispositivo descriptivo. Use la configuración predeterminada para la generación automática de claves de autenticación y para conectar el dispositivo nuevo a su centro.
5. Seleccione **Guardar**.

### <a name="view-iot-edge-devices-in-the-azure-portal"></a>Visualización de dispositivos IoT Edge en Azure Portal

Todos los dispositivos habilitados para Edge que se conectan al centro de IoT se enumeran en la página **IoT Edge**.

![Visualización de todos los dispositivos IoT Edge en su centro de IoT](./media/how-to-register-device/portal-view-devices.png)

### <a name="retrieve-the-connection-string-in-the-azure-portal"></a>Recuperación de la cadena de conexión en Azure Portal

Cuando esté listo para configurar el dispositivo, necesitará la cadena de conexión que vincula el dispositivo físico con su identidad en el centro de IoT.

1. En la página **IoT Edge** del portal, haga clic en el identificador de dispositivo en la lista de dispositivos de IoT Edge.
2. Copie el valor de la **cadena de conexión primaria** o la **cadena de conexión secundaria**.

## <a name="register-with-visual-studio-code"></a>Registro con Visual Studio Code

Hay varias maneras de realizar la mayoría de operaciones en VS Code. En este artículo se usa Explorer, pero también puede usar la paleta de comandos para ejecutar los pasos.

### <a name="prerequisites-for-visual-studio-code"></a>Requisitos previos para Visual Studio Code

* Una instancia de [IoT Hub](../iot-hub/iot-hub-create-through-portal.md) en la suscripción de Azure.
* [Visual Studio Code](https://code.visualstudio.com/)
* [Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools) para Visual Studio Code

### <a name="sign-in-to-access-your-iot-hub"></a>Iniciar sesión para acceder a IoT Hub

Puede usar las extensiones de Azure IoT para Visual Studio Code para realizar operaciones con IoT Hub. Para que estas operaciones funcionen, debe iniciar sesión en su cuenta de Azure y seleccionar su instancia de IoT Hub.

1. En Visual Studio Code, abra la vista **Explorer** (Explorador).
1. En la parte inferior de Explorer, expanda la sección **Azure IoT Hub**.

   ![Sección Expansión de los dispositivos de Azure IoT Hub](./media/how-to-register-device/azure-iot-hub-devices.png)

1. Haga clic en los puntos suspensivos **...** en el encabezado de la sección **Azure IoT Hub**. Si no ve el botón de los puntos suspensivos, mantenga el ratón sobre el encabezado o haga clic en él.
1. Elija **Select IoT Hub** (Seleccionar IoT Hub).
1. Si no ha iniciado sesión en su cuenta de Azure, siga las indicaciones para hacerlo.
1. Seleccione su suscripción a Azure.
1. Seleccione IoT Hub.

### <a name="create-an-iot-edge-device-with-visual-studio-code"></a>Creación de un dispositivo IoT Edge con Visual Studio Code

1. En el explorador de VS Code, expanda la sección **Azure IoT Hub Devices** (Dispositivos de Azure IoT Hub).
1. Haga clic en los puntos suspensivos **...** en el encabezado de la sección **Azure IoT Hub Devices** (Dispositivos de Azure IoT Hub). Si no ve el botón de los puntos suspensivos, mantenga el ratón sobre el encabezado o haga clic en él.
1. Seleccione **Create IoT Edge Device** (Crear un dispositivo de IoT Edge).
1. En el cuadro de texto que aparece, otorgue a su dispositivo un identificador.

En la pantalla de salida, consulte el resultado del comando. Se imprime la información del dispositivo, que incluye el valor de **deviceId** que proporcionó, y el valor de **connectionString** que puede usar para conectar su dispositivo físico a su instancia de IoT Hub.

En la pantalla de salida, consulte el resultado del comando. Se imprime la información del dispositivo, que incluye el valor de **deviceId** que proporcionó, y el valor de **connectionString** que puede usar para conectar su dispositivo físico a su instancia de IoT Hub.

### <a name="view-iot-edge-devices-with-visual-studio-code"></a>Creación de dispositivos IoT Edge con Visual Studio Code

Todos los dispositivos que se conectan a IoT Hub se enumeran en la sección **Azure IoT Hub** del explorador de Visual Studio Code. Los dispositivos de IoT Edge se distinguen de los dispositivos que no son Edge con un icono diferente y por el hecho de que los módulos **$edgeAgent** y **$edgeHub** se implementan en cada dispositivo IoT Edge.

![Visualización de todos los dispositivos IoT Edge en su centro de IoT](./media/how-to-register-device/view-devices.png)

### <a name="retrieve-the-connection-string-with-visual-studio-code"></a>Recuperación de la cadena de conexión con Visual Studio Code

Cuando esté listo para configurar el dispositivo, necesitará la cadena de conexión que vincula el dispositivo físico con su identidad en el centro de IoT.

1. Haga clic con el botón derecho en el identificador del dispositivo en la sección **Azure IoT Hub**.
1. Seleccione **Copy Device Connection String** (Copiar la cadena de conexión del dispositivo).

   La cadena de conexión se copiará al portapapeles.

También puede seleccionar **Get Device Info** (Obtener información del dispositivo), si hace clic con el botón derecho del mouse en el menú para ver toda la información del dispositivo, incluyendo la cadena de conexión, en la ventana de salida.

## <a name="register-with-the-azure-cli"></a>Registro con la CLI de Azure

La [CLI de Azure](https://docs.microsoft.com/cli/azure?view=azure-cli-latest) es una herramienta de la línea de comandos multiplataforma y de código abierto para la administración de recursos de Azure, como IoT Edge. Esta permite administrar recursos de Azure IoT Hub, instancias de servicio de aprovisionamiento de dispositivos y centros vinculados listos para usar. La extensión de IoT enriquece la CLI de Azure con características como administración de dispositivos y funcionalidad completa de IoT Edge.

### <a name="prerequisites-for-the-azure-cli"></a>Requisitos previos para la CLI de Azure

* Una instancia de [IoT Hub](../iot-hub/iot-hub-create-using-cli.md) en la suscripción de Azure.
* La [CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli) en su entorno. Como mínimo, la versión de la CLI de Azure debe ser la 2.0.70. Use `az --version` para asegurarse. Esta versión admite comandos az extension e introduce la plataforma de comandos de Knack.
* La [extensión de IoT para la CLI de Azure](https://github.com/Azure/azure-iot-cli-extension).

### <a name="create-an-iot-edge-device-with-the-azure-cli"></a>Creación de un dispositivo IoT Edge con la CLI de Azure

Use el comando [az iot hub device-identity create](https://docs.microsoft.com/cli/azure/ext/azure-iot/iot/hub/device-identity?view=azure-cli-latest#ext-azure-iot-az-iot-hub-device-identity-create) para crear una nueva identidad de dispositivo en su centro de IoT. Por ejemplo:

   ```azurecli
   az iot hub device-identity create --device-id [device id] --hub-name [hub name] --edge-enabled
   ```

Este comando incluye tres parámetros:

* **device-id**: proporcione un nombre descriptivo que sea único de su centro de IoT.
* **hub-name**: proporcione el nombre de su centro de IoT.
* **edge-enabled**: este parámetro declara que el dispositivo se va a usar con IoT Edge.

   ![az iot hub device-identity create output](./media/how-to-register-device/Create-edge-device.png)

### <a name="view-iot-edge-devices-with-the-azure-cli"></a>Creación de dispositivos IoT Edge con la CLI de Azure

Use el comando [az iot hub device-identity list](https://docs.microsoft.com/cli/azure/ext/azure-iot/iot/hub/device-identity?view=azure-cli-latest#ext-azure-iot-az-iot-hub-device-identity-list) para ver todos los dispositivos en su centro de IoT. Por ejemplo:

   ```azurecli
   az iot hub device-identity list --hub-name [hub name]
   ```

Los dispositivos que están registrados como dispositivos IoT Edge tendrán la propiedad **capabilities.iotEdge** establecida en **true**.

### <a name="retrieve-the-connection-string-with-the-azure-cli"></a>Recuperación de la cadena de conexión con la CLI de Azure

Cuando esté listo para configurar el dispositivo, necesitará la cadena de conexión que vincula el dispositivo físico con su identidad en el centro de IoT. Use el comando [az iot hub device-identity show-connection-string](https://docs.microsoft.com/cli/azure/ext/azure-iot/iot/hub/device-identity?view=azure-cli-latest#ext-azure-iot-az-iot-hub-device-identity-show-connection-string) para devolver la cadena de conexión para un único dispositivo:

   ```azurecli
   az iot hub device-identity show-connection-string --device-id [device id] --hub-name [hub name]
   ```

El valor del parámetro `device-id` distingue mayúsculas y minúsculas. No copie las comillas alrededor de la cadena de conexión.

## <a name="next-steps"></a>Pasos siguientes

Ahora que tiene una identidad de dispositivo registrada en IoT Hub, está listo para instalar el entorno de tiempo de ejecución de IoT Edge en los dispositivos. Instale el entorno de tiempo de ejecución según el sistema operativo del dispositivo:

* [Instalación de Azure IoT Edge en Windows](how-to-install-iot-edge-windows.md)
* [Instalación del entorno de ejecución de Azure IoT Edge en Linux](how-to-install-iot-edge-linux.md)
