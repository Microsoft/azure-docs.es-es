---
title: 'Inicio rápido: Envío de telemetría a Azure IoT Hub con Java'
description: En esta guía de inicio rápido, ejecutará dos aplicaciones Java de ejemplo para enviar datos de telemetría simulados a un centro de IoT y para leer los datos de telemetría procedentes de este para procesarlos en la nube.
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.devlang: java
ms.topic: quickstart
ms.custom:
- mvc
- seo-java-august2019
- seo-java-september2019
- mqtt
- devx-track-java
ms.date: 05/26/2020
ms.openlocfilehash: 757e0892469e52b1f2716d837d8423c5940322e2
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87317933"
---
# <a name="quickstart-send-telemetry-to-an-azure-iot-hub-and-read-it-with-a-java-application"></a>Inicio rápido: Envío de telemetría a una instancia de Azure IoT Hub y su lectura en una aplicación Java

[!INCLUDE [iot-hub-quickstarts-1-selector](../../includes/iot-hub-quickstarts-1-selector.md)]

En este inicio rápido se envían datos de telemetría a Azure IoT Hub y se leen con una aplicación de Java. IoT Hub es un servicio de Azure que le permite ingerir grandes volúmenes de datos de telemetría desde los dispositivos IoT en la nube para su almacenamiento o procesamiento. En este inicio rápido se usan dos aplicaciones de Java escritas previamente: una para enviar datos de telemetría y la otra para leer datos de telemetría desde el centro. Antes de ejecutar estas dos aplicaciones, creará un centro de IoT Hub y registrará un dispositivo con el centro.

## <a name="prerequisites"></a>Prerrequisitos

* Una cuenta de Azure con una suscripción activa. [cree una de forma gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

* Java SE Development Kit 8. En el artículo [Soporte técnico de Java a largo plazo para Azure y Azure Stack](https://docs.microsoft.com/java/azure/jdk/?view=azure-java-stable), en la sección **Soporte técnico a largo plazo**, seleccione **Java 8**.

* [Apache Maven 3](https://maven.apache.org/download.cgi).

* [Un proyecto en Java de ejemplo](https://github.com/Azure-Samples/azure-iot-samples-java/archive/master.zip).

* El puerto 8883 abierto en el firewall. En el dispositivo de ejemplo de este inicio rápido se usa el protocolo MQTT, que se comunica mediante el puerto 8883. Este puerto puede estar bloqueado en algunos entornos de red corporativos y educativos. Para más información y para saber cómo solucionar este problema, consulte el artículo sobre la [conexión a IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

Puede verificar la versión actual de Java en el equipo de desarrollo con el comando siguiente:

```cmd/sh
java -version
```

Puede verificar la versión actual de Maven en el equipo de desarrollo con el comando siguiente:

```cmd/sh
mvn --version
```

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

### <a name="add-azure-iot-extension"></a>Adición de la extensión IoT de Azure

Ejecute el siguiente comando para agregar la extensión IoT de Microsoft Azure para la CLI de Azure a la instancia de Cloud Shell. La extensión IoT agrega comandos específicos de IoT Hub, IoT Edge e IoT Device Provisioning Service (DPS) a la CLI de Azure.

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

   **MyJavaDevice**: es el nombre del dispositivo que se va a registrar. Se recomienda usar **MyJavaDevice** como se muestra. Si elige un nombre distinto para el dispositivo, tendrá que usarlo en todo el artículo y actualizar el nombre del dispositivo en las aplicaciones de ejemplo antes de ejecutarlas.

    ```azurecli-interactive
    az iot hub device-identity create --hub-name {YourIoTHubName} --device-id MyJavaDevice
    ```

2. Ejecute el siguiente comandos en Azure Cloud Shell para obtener la _cadena de conexión_ del dispositivo que acaba de registrar:

    **YourIoTHubName**: reemplace este marcador de posición por el nombre elegido para el centro de IoT.

    ```azurecli-interactive
    az iot hub device-identity show-connection-string --hub-name {YourIoTHubName} --device-id MyJavaDevice --output table
    ```

    Anote la cadena de conexión del dispositivo, que se parecerá a esta:

   `HostName={YourIoTHubName}.azure-devices.net;DeviceId=MyJavaDevice;SharedAccessKey={YourSharedAccessKey}`

    Este valor lo usará más adelante en este inicio rápido.

3. También necesitará el _punto de conexión compatible con Event Hubs_, la _ruta de acceso compatible con Event Hubs_ y la _clave principal de servicio_ de IoT Hub para permitir que la aplicación back-end se conecte a IoT Hub y recupere los mensajes. Los siguientes comandos recuperan estos valores para IoT Hub:

     **YourIoTHubName**: reemplace este marcador de posición por el nombre elegido para el centro de IoT.

    ```azurecli-interactive
    az iot hub show --query properties.eventHubEndpoints.events.endpoint --name {YourIoTHubName}

    az iot hub show --query properties.eventHubEndpoints.events.path --name {YourIoTHubName}

    az iot hub policy show --name service --query primaryKey --hub-name {YourIoTHubName}
    ```

    Anote estos tres valores, ya que los usará más adelante en el inicio rápido.

## <a name="send-simulated-telemetry"></a>Envío de datos de telemetría simulados

La aplicación de dispositivo simulado se conecta a un punto de conexión específico del dispositivo en IoT Hub y envía datos de telemetría simulados sobre temperatura y humedad.

1. En una ventana de terminal local, vaya a la carpeta raíz del proyecto de Java de ejemplo. A continuación, vaya a la carpeta **iot-hub\Quickstarts\simulated-device**.

2. Abra el archivo **src/main/java/com/microsoft/docs/iothub/samples/SimulatedDevice.java** en el editor de texto de su elección.

    Reemplace el valor de la variable `connString` por la cadena de conexión del dispositivo que anotó anteriormente. A continuación, guarde los cambios realizados en **SimulatedDevice.java**.

3. En la ventana de terminal local, ejecute los comandos siguientes para instalar las bibliotecas necesarias y compile la aplicación de dispositivo simulado:

    ```cmd/sh
    mvn clean package
    ```

4. En la ventana de terminal local, ejecute los comandos siguientes para ejecutar la aplicación de dispositivo simulado:

    ```cmd/sh
    java -jar target/simulated-device-1.0.0-with-deps.jar
    ```

    La siguiente captura de pantalla muestra la salida en la que la aplicación de dispositivo simulado envía datos de telemetría a IoT Hub:

    ![Salida de los datos de telemetría enviados por el dispositivo al centro de IoT](media/quickstart-send-telemetry-java/simulated-device.png)

## <a name="read-the-telemetry-from-your-hub"></a>Lectura de los datos de telemetría procedentes de su instancia de IoT Hub

La aplicación back-end se conecta a un punto de conexión de **Eventos** de servicio en IoT Hub. La aplicación recibe los mensajes del dispositivo a la nube enviados desde el dispositivo simulado. Normalmente, una aplicación back-end de IoT Hub se ejecuta en la nube para recibir y procesar los mensajes del dispositivo a la nube.

1. En otra ventana de terminal local, vaya a la carpeta raíz del proyecto de Java de ejemplo. A continuación, vaya a la carpeta **iot-hub\Quickstarts\read-d2c-messages**.

2. Abra el archivo **src/main/java/com/microsoft/docs/iothub/samples/ReadDeviceToCloudMessages.java** en el editor de texto de su elección. Actualice las siguientes variables y guarde los cambios en el archivo.

    | Variable | Value |
    | -------- | ----------- |
    | `EVENT_HUBS_COMPATIBLE_ENDPOINT` | Reemplace el valor de la variable por el punto de conexión compatible con Event Hubs que anotó anteriormente. |
    | `EVENT_HUBS_COMPATIBLE_PATH`     | Reemplace el valor de la variable por la ruta de acceso compatible con Event Hubs que ha anotó anteriormente. |
    | `IOT_HUB_SAS_KEY`                | Reemplace el valor de la variable por la clave principal del servicio que anotó anteriormente. |

3. En la ventana de terminal local, ejecute los comandos siguientes para instalar las bibliotecas necesarias y compile la aplicación back-end:

    ```cmd/sh
    mvn clean package
    ```

4. En la ventana de terminal local, ejecute los comandos siguientes para ejecutar la aplicación back-end:

    ```cmd/sh
    java -jar target/read-d2c-messages-1.0.0-with-deps.jar
    ```

    La siguiente captura de pantalla muestra la salida en la que la aplicación back-end recibe los datos de telemetría que el dispositivo simulado ha enviado al centro:

    ![La salida como aplicación de back-end recibe los datos de telemetría que se han enviado a su centro de IoT](media/quickstart-send-telemetry-java/read-device-to-cloud.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [iot-hub-quickstarts-clean-up-resources](../../includes/iot-hub-quickstarts-clean-up-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha configurado un centro de IoT, registrado un dispositivo, enviado datos de telemetría simulados al centro de conectividad mediante una aplicación de Java y leído datos de telemetría desde el centro de conectividad mediante una aplicación de back-end básica.

Para aprender a controlar el dispositivo simulado desde una aplicación back-end, vaya al siguiente inicio rápido.

> [!div class="nextstepaction"]
> [Inicio rápido: Control de un dispositivo conectado a IoT Hub](quickstart-control-device-java.md)
