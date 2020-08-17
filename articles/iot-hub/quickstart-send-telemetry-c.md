---
title: Guía de inicio rápido para enviar datos de telemetría a Azure IoT Hub (C) | Microsoft Docs
description: En esta guía de inicio rápido, ejecutará dos aplicaciones C de ejemplo para enviar datos de telemetría simulados a un centro de IoT y para leer los datos de telemetría procedentes de este para procesarlos en la nube.
author: wesmc7777
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.devlang: c
ms.topic: quickstart
ms.custom:
- mvc
- mqtt
- 'Role: Cloud Development'
ms.date: 04/10/2019
ms.author: wesmc
ms.openlocfilehash: ecc3c4e97ea533f19e570db21eceb6f759969a8f
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87317950"
---
# <a name="quickstart-send-telemetry-from-a-device-to-an-iot-hub-and-read-it-with-a-back-end-application-c"></a>Inicio rápido: Envío de telemetría desde un dispositivo a un centro de IoT y su lectura con una aplicación de back-end (C)

[!INCLUDE [iot-hub-quickstarts-1-selector](../../includes/iot-hub-quickstarts-1-selector.md)]

IoT Hub es un servicio de Azure que le permite ingerir grandes volúmenes de datos de telemetría desde los dispositivos IoT en la nube para su almacenamiento o procesamiento. En este inicio rápido, enviará datos de telemetría desde una aplicación de dispositivo simulado, por IoT Hub, a una aplicación back-end para su procesamiento.

El inicio rápido usa una aplicación de ejemplo de C del [SDK de dispositivo IoT de Azure para C](iot-hub-device-sdk-c-intro.md) para enviar telemetría a un centro de IoT. Los SDK de dispositivo IoT de Azure están escritos en [ANSI C (C99)](https://wikipedia.org/wiki/C99) para portabilidad y amplia compatibilidad de plataformas. Antes de ejecutar el código de ejemplo, creará un centro de IoT y registrará el dispositivo simulado con ese centro.

El artículo se ha escrito para Windows, pero este inicio rápido también puede completar en Linux.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

* Instale [Visual Studio 2019](https://www.visualstudio.com/vs/) con la carga de trabajo ["Desarrollo para el escritorio con C++"](https://www.visualstudio.com/vs/support/selecting-workloads-visual-studio-2017/) habilitada.

* Instale la última versión de [Git](https://git-scm.com/download/).

* Asegúrese de que el puerto 8883 está abierto en el firewall. En el dispositivo de ejemplo de este inicio rápido se usa el protocolo MQTT, que se comunica mediante el puerto 8883. Este puerto puede estar bloqueado en algunos entornos de red corporativos y educativos. Para más información y para saber cómo solucionar este problema, consulte el artículo sobre la [conexión a IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).


* Ejecute el siguiente comando para agregar la extensión IoT de Microsoft Azure para la CLI de Azure a la instancia de Cloud Shell. La extensión IoT agrega comandos específicos de IoT Hub, IoT Edge e IoT Device Provisioning Service (DPS) a la CLI de Azure.

   ```azurecli-interactive
   az extension add --name azure-iot
   ```

[!INCLUDE [iot-hub-cli-version-info](../../includes/iot-hub-cli-version-info.md)]

## <a name="prepare-the-development-environment"></a>Preparación del entorno de desarrollo

En esta guía de inicio rápido, usará el [SDK de dispositivo IoT de Azure para C](iot-hub-device-sdk-c-intro.md). 

Para los siguientes entornos puede usar el SDK mediante la instalación de paquetes y bibliotecas:

* **Linux**: los paquetes apt-get están disponibles para Ubuntu 16.04 y 18.04 con el uso de las siguientes arquitecturas de CPU: amd64, arm64, armhf e i386. Para más información, vea [Using apt-get to create a C device client project on Ubuntu](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/ubuntu_apt-get_sample_setup.md) (Uso de apt-get para crear un proyecto cliente de dispositivos en C en Ubuntu).

* **mbed**: para los desarrolladores que crean aplicaciones de dispositivo en la plataforma mbed, hemos publicado una biblioteca y ejemplos que le ayudarán a ponerse en marcha en cuestión de minutos con Azure IoT Hub. Para más información, vea [Use the mbed library](https://github.com/Azure/azure-iot-sdk-c/blob/master/iothub_client/readme.md#mbed) (Uso de la biblioteca mbed).

* **Arduino**: si desarrolla en Arduino, puede utilizar la biblioteca de Azure IoT disponible en el administrador de bibliotecas de IDE de Arduino. Para más información, vea [The Azure IoT Hub library for Arduino](https://github.com/azure/azure-iot-arduino) (Biblioteca de Azure IoT Hub para Arduino).

* **iOS**: el SDK de dispositivo de IoT Hub está disponible como CocoaPods para el desarrollo de dispositivos iOS y Mac. Para más información, vea [iOS Samples for Microsoft Azure IoT](https://cocoapods.org/pods/AzureIoTHubClient) (Ejemplos de iOS para IoT de Microsoft Azure).

Sin embargo, en esta guía de inicio rápido, preparará un entorno de desarrollo utilizado para clonar y compilar el [SDK de IoT de Azure para C](https://github.com/Azure/azure-iot-sdk-c) desde GitHub. El SDK de GitHub incluye el código de ejemplo usado en este inicio rápido.

1. Descargue el [sistema de compilación CMake](https://cmake.org/download/).

    **Antes** de comenzar la instalación de `CMake`, es importante que los requisitos previos de Visual Studio (Visual Studio y la carga de trabajo de desarrollo de escritorio con C++) estén instalados en la máquina. Una vez que los requisitos previos están en su lugar, y se ha comprobado la descarga, instale el sistema de compilación de CMake.

2. Busque el nombre de etiqueta de la [versión más reciente](https://github.com/Azure/azure-iot-sdk-c/releases/latest) del SDK.

3. Abra un símbolo del sistema o el shell de Bash de Git. Ejecute los siguientes comandos para clonar la versión más reciente del repositorio de GitHub del [SDK de Azure IoT para C](https://github.com/Azure/azure-iot-sdk-c). Use la etiqueta que encontró en el paso anterior como valor del parámetro `-b`:

    ```cmd/sh
    git clone -b <release-tag> https://github.com/Azure/azure-iot-sdk-c.git
    cd azure-iot-sdk-c
    git submodule update --init
    ```

    Esta operación puede tardar varios minutos en completarse.

4. Cree un subdirectorio `cmake` en el directorio raíz del repositorio de Git y vaya a esa carpeta. Ejecute los siguientes comandos desde el directorio `azure-iot-sdk-c`:

    ```cmd/sh
    mkdir cmake
    cd cmake
    ```

5. Ejecute el siguiente comando para compilar una versión del SDK específica para su plataforma de cliente de desarrollo. Se generará una solución de Visual Studio para el dispositivo simulado en el directorio `cmake`.

    ```cmd
    cmake ..
    ```

    Si `cmake` no puede encontrar el compilador de C++, es posible que obtenga errores de compilación durante la ejecución del comando anterior. Si sucede, intente ejecutar este comando en el [símbolo del sistema de Visual Studio](https://docs.microsoft.com/dotnet/framework/tools/developer-command-prompt-for-vs). 

    Una vez realizada la compilación, las últimas líneas de salida tendrán un aspecto similar al siguiente:

    ```cmd/sh
    $ cmake ..
    -- Building for: Visual Studio 15 2017
    -- Selecting Windows SDK version 10.0.16299.0 to target Windows 10.0.17134.
    -- The C compiler identification is MSVC 19.12.25835.0
    -- The CXX compiler identification is MSVC 19.12.25835.0

    ...

    -- Configuring done
    -- Generating done
    -- Build files have been written to: E:/IoT Testing/azure-iot-sdk-c/cmake
    ```

## <a name="create-an-iot-hub"></a>Crear un centro de IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-device"></a>Registrar un dispositivo

Debe registrar un dispositivo con IoT Hub antes de poder conectarlo. En esta sección, usará Azure Cloud Shell con la [extensión de IoT](https://docs.microsoft.com/cli/azure/ext/azure-iot/iot?view=azure-cli-latest) para registrar un dispositivo simulado.

1. Ejecute los siguientes comandos en Azure Cloud Shell para crear la identidad del dispositivo.

   **YourIoTHubName**: reemplace este marcador de posición por el nombre elegido para el centro de IoT.

   **MyCDevice**: es el nombre del dispositivo que se va a registrar. Se recomienda usar **MyCDevice** como se muestra. Si elige un nombre distinto para el dispositivo, tendrá que usarlo en todo el artículo y actualizar el nombre del dispositivo en las aplicaciones de ejemplo antes de ejecutarlas.

    ```azurecli-interactive
    az iot hub device-identity create --hub-name {YourIoTHubName} --device-id MyCDevice
    ```

2. Ejecute el siguiente comandos en Azure Cloud Shell para obtener la _cadena de conexión_ del dispositivo que acaba de registrar:

   **YourIoTHubName**: reemplace este marcador de posición por el nombre elegido para el centro de IoT.

    ```azurecli-interactive
    az iot hub device-identity show-connection-string --hub-name {YourIoTHubName} --device-id MyCDevice --output table
    ```

    Anote la cadena de conexión del dispositivo, que se parecerá a esta:

   `HostName={YourIoTHubName}.azure-devices.net;DeviceId=MyCDevice;SharedAccessKey={YourSharedAccessKey}`

    Este valor lo usará más adelante en este inicio rápido.

## <a name="send-simulated-telemetry"></a>Envío de datos de telemetría simulados

La aplicación de dispositivo simulado se conecta a un punto de conexión específico del dispositivo en el centro de IoT y envía una cadena como datos de telemetría simulados.

1. Con un editor de texto, abra el archivo de origen iothub_convenience_sample.c y revise el código de ejemplo para enviar telemetría. El archivo se encuentra en la siguiente ubicación del directorio de trabajo en que clonó el SDK de C de Azure IoT:

    ```
    azure-iot-sdk-c\iothub_client\samples\iothub_convenience_sample\iothub_convenience_sample.c
    ```

2. Busque la declaración de la constante `connectionString`:

    ```C
    /* Paste in your device connection string  */
    static const char* connectionString = "[device connection string]";
    ```

    Reemplace el valor de la constante `connectionString` por la cadena de conexión del dispositivo que anotó anteriormente. Luego guarde los cambios en **iothub_convenience_sample.c**.

3. En una ventana de terminal local, vaya al directorio del proyecto *iothub_convenience_sample* en el directorio de CMake que creó en el SDK de IoT de Azure para C. Escriba el siguiente comando desde el directorio de trabajo:

    ```cmd/sh
    cd azure-iot-sdk-c/cmake/iothub_client/samples/iothub_convenience_sample
    ```

4. Ejecute CMake en la ventana de terminal local para crear el ejemplo con el valor `connectionString` actualizado:

    ```cmd/sh
    cmake --build . --target iothub_convenience_sample --config Debug
    ```

5. En una ventana del terminal local, escriba el siguiente comando para ejecutar la aplicación de dispositivo simulado:

    ```cmd/sh
    Debug\iothub_convenience_sample.exe
    ```

    La siguiente captura de pantalla muestra la salida en la que la aplicación de dispositivo simulado envía datos de telemetría al centro de IoT:

    ![Ejecutar el dispositivo simulado](media/quickstart-send-telemetry-c/simulated-device-app.png)

## <a name="read-the-telemetry-from-your-hub"></a>Lectura de los datos de telemetría procedentes de su instancia de IoT Hub

En esta sección, usará Azure Cloud Shell con la [extensión de IoT](https://docs.microsoft.com/cli/azure/ext/azure-iot/iot?view=azure-cli-latest) para supervisar los mensajes de dispositivo que envía el dispositivo simulado.

1. Mediante Azure Cloud Shell, ejecute el siguiente comando para conectarse y leer mensajes desde el centro de IoT:

   **YourIoTHubName**: reemplace este marcador de posición por el nombre elegido para el centro de IoT.

    ```azurecli-interactive
    az iot hub monitor-events --hub-name {YourIoTHubName} --output table
    ```

    ![Leer los mensajes de dispositivo mediante la CLI de Azure](media/quickstart-send-telemetry-c/read-device-to-cloud-messages-app.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [iot-hub-quickstarts-clean-up-resources](../../includes/iot-hub-quickstarts-clean-up-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este artículo de inicio rápido, ha configurado un centro de IoT, ha registrado un dispositivo, ha enviado datos de telemetría simulados al centro con una aplicación C y ha leído datos de telemetría desde el centro de conectividad con Azure Cloud Shell.

Para más información sobre el desarrollo con el SDK de Azure IoT Hub para C, continúe con la guía paso a paso siguiente:

> [!div class="nextstepaction"]
> [Desarrollo con el SDK de Azure IoT Hub para C](iot-hub-devguide-develop-for-constrained-devices.md)
