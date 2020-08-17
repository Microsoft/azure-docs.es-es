---
title: 'Inicio rápido: Control de un dispositivo desde Azure IoT Hub con Java'
description: En este inicio rápido, ejecuta dos aplicaciones de Java de muestra. Una aplicación es una aplicación back-end que puede controlar dispositivos conectados al centro de manera remota. La otra aplicación simula un dispositivo conectado al centro que se puede controlar de manera remota.
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
ms.date: 06/21/2019
ms.openlocfilehash: 3c6a8285c672e6683a259b2a65be4e6ecbd4aafa
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87322285"
---
# <a name="quickstart-control-a-device-connected-to-an-azure-iot-hub-with-java"></a>Inicio rápido: Control de un dispositivo conectado a una instancia de Azure IoT Hub con Java

[!INCLUDE [iot-hub-quickstarts-2-selector](../../includes/iot-hub-quickstarts-2-selector.md)]

En este inicio rápido, usa un método directo para controlar un dispositivo simulado conectado a Azure IoT Hub con una aplicación Java. IoT Hub es un servicio de Azure que permite administrar dispositivos de IoT desde la nube e ingerir grandes volúmenes de datos de telemetría desde los dispositivos en la nube para su almacenamiento o procesamiento. Puede usar métodos directos para cambiar el comportamiento de un dispositivo conectado a IoT Hub de manera remota. En este inicio rápido se usan dos aplicaciones Java: una aplicación de un dispositivo simulado que responde a métodos directos llamados desde una aplicación de back-end y una aplicación de servicio que llama al método directo en el dispositivo simulado.

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

Si ha completado la anterior [Guía de inicio rápido: Envío de telemetría desde un dispositivo a un centro de IoT](quickstart-send-telemetry-java.md) puede omitir este paso.

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-device"></a>Registrar un dispositivo

Si ha completado la anterior [Guía de inicio rápido: Envío de telemetría desde un dispositivo a un centro de IoT](quickstart-send-telemetry-java.md) puede omitir este paso.

Debe registrar un dispositivo con IoT Hub antes de poder conectarlo. En esta guía de inicio rápido, usará Azure Cloud Shell para registrar un dispositivo simulado.

1. Ejecute los siguientes comandos en Azure Cloud Shell para crear la identidad del dispositivo.

   **YourIoTHubName**: reemplace este marcador de posición por el nombre elegido para el centro de IoT.

   **MyJavaDevice**: es el nombre del dispositivo que se va a registrar. Se recomienda usar **MyJavaDevice** como se muestra. Si elige otro nombre distinto para el dispositivo, tendrá que usarlo en todo el artículo y actualizar el nombre del dispositivo en las aplicaciones de ejemplo antes de ejecutarlas.

    ```azurecli-interactive
    az iot hub device-identity create \
      --hub-name {YourIoTHubName} --device-id MyJavaDevice
    ```

2. Ejecute los siguientes comandos en Azure Cloud Shell para obtener la _cadena de conexión del dispositivo_ que acaba de registrar:

   **YourIoTHubName**: reemplace este marcador de posición por el nombre elegido para el centro de IoT.

    ```azurecli-interactive
    az iot hub device-identity show-connection-string \
      --hub-name {YourIoTHubName} \
      --device-id MyJavaDevice \
      --output table
    ```

    Anote la cadena de conexión del dispositivo, que se parecerá a esta:

   `HostName={YourIoTHubName}.azure-devices.net;DeviceId=MyNodeDevice;SharedAccessKey={YourSharedAccessKey}`

    Usará este valor más adelante en este inicio rápido.

## <a name="retrieve-the-service-connection-string"></a>Recuperación de la cadena de conexión de servicio

También necesita una _cadena de conexión de servicio_ para permitir que la aplicación back-end se conecte a IoT Hub y recupere los mensajes. El comando siguiente recupera la cadena de conexión del servicio de su instancia de IoT Hub:

**YourIoTHubName**: reemplace este marcador de posición por el nombre elegido para el centro de IoT.

```azurecli-interactive
az iot hub show-connection-string --policy-name service --name {YourIoTHubName} --output table
```

Anote la cadena de conexión del servicio, que se parecerá a esta:

`HostName={YourIoTHubName}.azure-devices.net;SharedAccessKeyName=service;SharedAccessKey={YourSharedAccessKey}`

Usará este valor más adelante en este inicio rápido. Esta cadena de conexión del servicio no es la que anotó en el paso anterior.

## <a name="listen-for-direct-method-calls"></a>Escuchas para llamadas de método directo

La aplicación del dispositivo simulado se conecta a un punto de conexión específico del dispositivo en IoT Hub, envía los datos de telemetría simulados y escucha llamadas de método directo desde el centro. En este inicio rápido, la llamada de método directo desde el centro indica al dispositivo que debe cambiar el intervalo en el que envía los datos de telemetría. El dispositivo simulado envía una confirmación al centro después de que ejecuta el método directo.

1. En una ventana de terminal local, vaya a la carpeta raíz del proyecto de Java de ejemplo. A continuación, vaya a la carpeta **iot-hub\Quickstarts\simulated-device-2**.

2. Abra el archivo **src/main/java/com/microsoft/docs/iothub/samples/SimulatedDevice.java** en el editor de texto de su elección.

    Reemplace el valor de la variable `connString` por la cadena de conexión del dispositivo que anotó anteriormente. A continuación, guarde los cambios realizados en **SimulatedDevice.java**.

3. En la ventana de terminal local, ejecute los comandos siguientes para instalar las bibliotecas necesarias y compile la aplicación de dispositivo simulado:

    ```cmd/sh
    mvn clean package
    ```

4. En la ventana de terminal local, ejecute los comandos siguientes para ejecutar la aplicación de dispositivo simulado:

    ```cmd/sh
    java -jar target/simulated-device-2-1.0.0-with-deps.jar
    ```

    La siguiente captura de pantalla muestra la salida en la que la aplicación de dispositivo simulado envía datos de telemetría a IoT Hub:

    ![Salida de los datos de telemetría enviados por el dispositivo al centro de IoT](./media/quickstart-control-device-java/iot-hub-application-send-telemetry-output.png)

## <a name="call-the-direct-method"></a>Llamar al método directo

La aplicación back-end se conecta a un punto de conexión de servicio en IoT Hub. La aplicación realiza llamadas de método directo a un dispositivo con IoT Hub y escucha las confirmaciones. Normalmente, una aplicación back-end de IoT Hub se ejecuta en la nube.

1. En otra ventana de terminal local, vaya a la carpeta raíz del proyecto de Java de ejemplo. A continuación, vaya a la carpeta **iot-hub\Quickstarts\back-end-application**.

2. Abra el archivo **src/main/java/com/microsoft/docs/iothub/samples/BackEndApplication.java** en el editor de texto que prefiera.

    Reemplace el valor de la variable `iotHubConnectionString` por la cadena de conexión del servicio que anotó anteriormente. Luego, guarde los cambios en el archivo **BackEndApplication.java**.

3. En la ventana de terminal local, ejecute los comandos siguientes para instalar las bibliotecas necesarias y compile la aplicación back-end:

    ```cmd/sh
    mvn clean package
    ```

4. En la ventana de terminal local, ejecute los comandos siguientes para ejecutar la aplicación back-end:

    ```cmd/sh
    java -jar target/back-end-application-1.0.0-with-deps.jar
    ```

    En la siguiente captura de pantalla se muestra la salida en la que la aplicación realiza una llamada de método directo al dispositivo y recibe una confirmación:

    ![Salida cuando la aplicación realiza una llamada de método directo mediante IoT Hub](./media/quickstart-control-device-java/iot-hub-direct-method-call-output.png)

    Después de ejecutar la aplicación back-end, verá un mensaje en la ventana de consola que ejecuta el dispositivo simulado y cambiará la velocidad a la que envía mensajes:

    ![El mensaje de la consola desde el dispositivo muestra la velocidad a la que cambia](./media/quickstart-control-device-java/iot-hub-sent-message-change-rate.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [iot-hub-quickstarts-clean-up-resources](../../includes/iot-hub-quickstarts-clean-up-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha llamado a un método directo en un dispositivo desde una aplicación de back-end y ha respondido a la llamada de método directo en una aplicación de dispositivo simulado.

Para obtener información sobre cómo redirigir mensajes del dispositivo a la nube a diferentes destinos en la nube, continúe con el siguiente tutorial.

> [!div class="nextstepaction"]
> [Tutorial: Enrutar datos de telemetría a distintos puntos de conexión para procesamiento](tutorial-routing.md)
