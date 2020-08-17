---
title: Guía de inicio rápido para enviar datos de telemetría a Azure IoT Hub (Android) | Microsoft Docs
description: En esta guía de inicio rápido va a ejecutar una aplicación Android de ejemplo para enviar datos de telemetría simulados a IoT Hub y para leer los datos de telemetría procedentes de este para procesarlos en la nube.
author: wesmc7777
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.devlang: java
ms.topic: quickstart
ms.custom:
- mvc
- mqtt
- devx-track-java
ms.date: 03/15/2019
ms.author: wesmc
ms.openlocfilehash: 1e5e11d5d04c31fa77adcb0488447c91b0d05254
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87317967"
---
# <a name="quickstart-send-iot-telemetry-from-an-android-device"></a>Inicio rápido: Envío de datos de telemetría de IoT desde un dispositivo Android

[!INCLUDE [iot-hub-quickstarts-1-selector](../../includes/iot-hub-quickstarts-1-selector.md)]

En este inicio rápido se envían datos de telemetría a Azure IoT Hub desde una aplicación Android que se ejecuta en un dispositivo físico o simulado. IoT Hub es un servicio de Azure que le permite ingerir grandes volúmenes de datos de telemetría desde los dispositivos IoT en la nube para su almacenamiento o procesamiento. Este inicio rápido usa una aplicación Android escrita anteriormente para enviar los datos de telemetría. Los datos de telemetría se leerán desde la instancia de IoT Hub con Azure Cloud Shell. Antes de ejecutar la aplicación, creará un centro de IoT Hub y registrará un dispositivo con el centro.

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta de Azure con una suscripción activa. [cree una de forma gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

* [Android Studio con Android SDK 27](https://developer.android.com/studio/). Para más información, consulte [android-installation](https://developer.android.com/studio/install). En el ejemplo de este artículo se usa Android SDK 27.

* [Una aplicación Android de ejemplo](https://github.com/Azure-Samples/azure-iot-samples-java/tree/master/iot-hub/Samples/device/AndroidSample). Este ejemplo forma parte del repositorio [azure-iot-samples-java](https://github.com/Azure-Samples/azure-iot-samples-java).

* El puerto 8883 abierto en el firewall. En el dispositivo de ejemplo de este inicio rápido se usa el protocolo MQTT, que se comunica mediante el puerto 8883. Este puerto puede estar bloqueado en algunos entornos de red corporativos y educativos. Para más información y para saber cómo solucionar este problema, consulte el artículo sobre la [conexión a IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

### <a name="add-azure-iot-extension"></a>Adición de la extensión IoT de Azure

Ejecute el siguiente comando para agregar la extensión IoT de Microsoft Azure para la CLI de Azure a la instancia de Cloud Shell. La extensión IOT agrega comandos específicos de IoT Hub, IoT Edge e IoT Device Provisioning Service (DPS) a la CLI de Azure.

```azurecli-interactive
az extension add --name azure-iot
```

[!INCLUDE [iot-hub-cli-version-info](../../includes/iot-hub-cli-version-info.md)]

## <a name="create-an-iot-hub"></a>Crear un centro de IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-device"></a>Registrar un dispositivo

Debe registrar un dispositivo con IoT Hub antes de poder conectarlo. En esta guía de inicio rápido, usará Azure Cloud Shell para registrar un dispositivo simulado.

1. Ejecute los siguientes comandos en Azure Cloud Shell para crear la identidad del dispositivo.

   **YourIoTHubName**: reemplace este marcador de posición por el nombre elegido para el centro de IoT.

   **MyAndroidDevice**: es el nombre del dispositivo que se va a registrar. Se recomienda usar **MyAndroidDevice** como se muestra. Si elige un nombre distinto para el dispositivo, tendrá que usarlo en todo el artículo y actualizar el nombre del dispositivo en las aplicaciones de ejemplo antes de ejecutarlas.

    ```azurecli-interactive
    az iot hub device-identity create --hub-name {YourIoTHubName} --device-id MyAndroidDevice
    ```

2. Ejecute el siguiente comandos en Azure Cloud Shell para obtener la _cadena de conexión_ del dispositivo que acaba de registrar:

    **YourIoTHubName**: reemplace este marcador de posición por el nombre elegido para el centro de IoT.

    ```azurecli-interactive
    az iot hub device-identity show-connection-string --hub-name {YourIoTHubName} --device-id MyAndroidDevice --output table
    ```

    Anote la cadena de conexión del dispositivo, que se parecerá a esta:

   `HostName={YourIoTHubName}.azure-devices.net;DeviceId=MyAndroidDevice;SharedAccessKey={YourSharedAccessKey}`

    Este valor lo usará más adelante en este mismo inicio rápido para enviar datos de telemetría.

## <a name="send-simulated-telemetry"></a>Envío de datos de telemetría simulados

1. En Android Studio, abra el proyecto de Android de ejemplo de GitHub. El proyecto se encuentra en el siguiente directorio de la copia que ha clonado o descargado del repositorio [azure-iot-sample-java](https://github.com/Azure-Samples/azure-iot-samples-java): *\azure-iot-samples-java\iot-hub\Samples\device\AndroidSample*.

2. En Android Studio, abra *gradle.properties* para el proyecto de ejemplo y reemplace el marcador de posición **Device_Connection_String** por la cadena de conexión de dispositivo que anotó anteriormente.

    ```
    DeviceConnectionString=HostName={YourIoTHubName}.azure-devices.net;DeviceId=MyAndroidDevice;SharedAccessKey={YourSharedAccessKey}
    ```

3. En Android Studio, haga clic en **File**(Archivo) > **Sync Project with Gradle Files** (Sincronizar proyecto con archivos de Gradle). Compruebe que la compilación se completa.

   > [!NOTE]
   > Si se produce un error en la sincronización del proyecto, puede ser por uno de los siguientes motivos:
   >
   > * Las versiones del complemento Gradle para Android y Gradle al que se hace referencia en el proyecto están en desuso para su versión de Android Studio. Siga [estas instrucciones](https://developer.android.com/studio/releases/gradle-plugin) para hacer referencia a las versiones correctas del complemento y de Gradle e instalarlas.
   > * No se ha firmado el contrato de licencia de Android SDK. Siga las instrucciones de la salida de compilación para firmar el contrato de licencia y descargar el SDK.

4. Una vez completada la compilación, haga clic en **Run**(Ejecutar) > **Run 'app'** (Ejecutar "aplicación"). Configure la aplicación para que se ejecute en un dispositivo Android físico o un emulador de Android. Para más información sobre la ejecución de una aplicación de Android en un dispositivo físico o en un emulador, consulte [Ejecutar la aplicación](https://developer.android.com/training/basics/firstapp/running-app).

5. Cuando se cargue la aplicación, haga clic en el botón **Iniciar** para empezar a enviar datos de telemetría a IoT Hub:

    ![Application](media/quickstart-send-telemetry-android/sample-screenshot.png)


## <a name="read-the-telemetry-from-your-hub"></a>Lectura de los datos de telemetría procedentes de su instancia de IoT Hub

En esta sección, usará Azure Cloud Shell con la [extensión de IoT](https://docs.microsoft.com/cli/azure/ext/azure-iot/iot?view=azure-cli-latest) para supervisar los mensajes de dispositivo que envía el dispositivo Android.

1. Mediante Azure Cloud Shell, ejecute el siguiente comando para conectarse y leer mensajes desde el centro de IoT:

   **YourIoTHubName**: reemplace este marcador de posición por el nombre elegido para el centro de IoT.

    ```azurecli-interactive
    az iot hub monitor-events --hub-name {YourIoTHubName} --output table
    ```

    La siguiente captura de pantalla muestra la salida en la que IoT Hub recibe los datos de telemetría que el dispositivo Android ha enviado:

      ![Leer los mensajes de dispositivo mediante la CLI de Azure](media/quickstart-send-telemetry-android/read-data.png)
## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [iot-hub-quickstarts-clean-up-resources](../../includes/iot-hub-quickstarts-clean-up-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido ha configurado un centro de IoT, ha registrado un dispositivo, ha enviado datos de telemetría simulados al centro con una aplicación Android y ha leído datos de telemetría desde el centro con Azure Cloud Shell.

Para aprender a controlar el dispositivo simulado desde una aplicación back-end, vaya al siguiente inicio rápido.

> [!div class="nextstepaction"]
> [Inicio rápido: Control de un dispositivo conectado a IoT Hub](quickstart-control-device-android.md)