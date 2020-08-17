---
title: Inicio rápido para enviar datos de telemetría a Azure IoT Hub (Python) | Microsoft Docs
description: En esta guía de inicio rápido, ejecutará una aplicación Python de ejemplo para enviar datos de telemetría simulados a un centro de IoT y usará una utilidad para leer los datos de telemetría procedentes del centro de IoT.
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.devlang: python
ms.topic: quickstart
ms.custom:
- mvc
- mqtt
- devx-track-python
- 'Role: Cloud Development'
ms.date: 06/16/2020
ms.openlocfilehash: 3df26f78e66aa1806fd13fd1a46444bb5dc79742
ms.sourcegitcommit: dea88d5e28bd4bbd55f5303d7d58785fad5a341d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87876230"
---
# <a name="quickstart-send-telemetry-from-a-device-to-an-iot-hub-and-read-it-with-a-back-end-application-python"></a>Inicio rápido: Envío de telemetría desde un dispositivo a un centro de IoT y su lectura con una aplicación de back-end (Python)

[!INCLUDE [iot-hub-quickstarts-1-selector](../../includes/iot-hub-quickstarts-1-selector.md)]

En este inicio rápido, se envían a través de Azure IoT Hub datos de telemetría desde una aplicación de dispositivo simulado a una aplicación de back-end para su procesamiento. IoT Hub es un servicio de Azure que le permite ingerir grandes volúmenes de datos de telemetría desde los dispositivos IoT en la nube para su almacenamiento o procesamiento. En este inicio rápido se usan dos aplicaciones de Python escritas previamente: una para enviar datos de telemetría y la otra para leer datos de telemetría desde el centro. Antes de ejecutar estas dos aplicaciones, creará un centro de IoT Hub y registrará un dispositivo con el centro.

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta de Azure con una suscripción activa. [cree una de forma gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

* [Python 3.7+](https://www.python.org/downloads/). Para obtener información acerca de otras versiones de Python compatibles, consulte el tema acerca de las [características de los dispositivos IoT de Azure](https://github.com/Azure/azure-iot-sdk-python/tree/master/azure-iot-device#azure-iot-device-features).

* [Un proyecto de Python de ejemplo](https://github.com/Azure-Samples/azure-iot-samples-python/archive/master.zip).

* El puerto 8883 abierto en el firewall. En el dispositivo de ejemplo de este inicio rápido se usa el protocolo MQTT, que se comunica mediante el puerto 8883. Este puerto puede estar bloqueado en algunos entornos de red corporativos y educativos. Para más información y para saber cómo solucionar este problema, consulte el artículo sobre la [conexión a IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

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

    **MyPythonDevice**: es el nombre del dispositivo que se va a registrar. Se recomienda usar **MyPythonDevice** como se muestra. Si elige un nombre distinto para el dispositivo, tendrá que usarlo en todo el artículo y actualizar el nombre del dispositivo en las aplicaciones de ejemplo antes de ejecutarlas.

    ```azurecli-interactive
    az iot hub device-identity create --hub-name {YourIoTHubName} --device-id MyPythonDevice
    ```

1. Ejecute el siguiente comando en Azure Cloud Shell para obtener la _cadena de conexión del dispositivo_ que registró:

    **YourIoTHubName**: reemplace este marcador de posición por el nombre elegido para el centro de IoT.

    ```azurecli-interactive
    az iot hub device-identity show-connection-string --hub-name {YourIoTHubName} --device-id MyPythonDevice --output table
    ```

    Anote la cadena de conexión del dispositivo, que se parecerá a esta:

   `HostName={YourIoTHubName}.azure-devices.net;DeviceId=MyPythonDevice;SharedAccessKey={YourSharedAccessKey}`

    Este valor lo usará más adelante en este inicio rápido.

1. También necesitará el _punto de conexión compatible con Event Hubs_, la _ruta de acceso compatible con Event Hubs_ y la _clave principal de servicio_ de IoT Hub para permitir que la aplicación back-end se conecte a IoT Hub y recupere los mensajes. Los siguientes comandos recuperan estos valores para IoT Hub:

   **YourIoTHubName**: reemplace este marcador de posición por el nombre elegido para el centro de IoT.

    ```azurecli-interactive
    az iot hub show --query properties.eventHubEndpoints.events.endpoint --name {YourIoTHubName}

    az iot hub show --query properties.eventHubEndpoints.events.path --name {YourIoTHubName}

    az iot hub policy show --name service --query primaryKey --hub-name {YourIoTHubName}
    ```

    Anote estos tres valores, ya que los usará más adelante en el inicio rápido.

## <a name="send-simulated-telemetry"></a>Envío de datos de telemetría simulados

La aplicación de dispositivo simulado se conecta a un punto de conexión específico del dispositivo en IoT Hub y envía datos de telemetría simulados sobre temperatura y humedad.

1. En una ventana de terminal local, vaya a la carpeta raíz del proyecto de Python de muestra. A continuación, vaya a la carpeta **iot-hub\Quickstarts\simulated-device**.

1. Abra el archivo **SimulatedDevice.py** en el editor de texto de su elección.

    Reemplace el valor de la variable `CONNECTION_STRING` por la cadena de conexión del dispositivo que anotó anteriormente. Luego, guarde los cambios realizados en **SimulatedDevice.py**.

1. En la ventana de terminal local, ejecute los comandos siguientes para instalar las bibliotecas necesarias para la aplicación de dispositivo simulado:

    ```cmd/sh
    pip install azure-iot-device
    ```

1. En la ventana de terminal local, ejecute los comandos siguientes para ejecutar la aplicación de dispositivo simulado:

    ```cmd/sh
    python SimulatedDevice.py
    ```

    La siguiente captura de pantalla muestra la salida en la que la aplicación de dispositivo simulado envía datos de telemetría a IoT Hub:

    ![Ejecutar el dispositivo simulado](media/quickstart-send-telemetry-python/simulated-device.png)

## <a name="read-the-telemetry-from-your-hub"></a>Lectura de los datos de telemetría procedentes de su instancia de IoT Hub

La aplicación back-end se conecta a un punto de conexión de **Eventos** de servicio en IoT Hub. La aplicación recibe los mensajes del dispositivo a la nube enviados desde el dispositivo simulado. Normalmente, una aplicación back-end de IoT Hub se ejecuta en la nube para recibir y procesar los mensajes del dispositivo a la nube.

> [!NOTE]
> En los pasos siguientes se usa el ejemplo sincrónico **read_device_to_cloud_messages_sync.py**. Puede realizar los mismos pasos con el ejemplo asincrónico **read_device_to_cloud_messages_async.py**.

1. En otra ventana de terminal local, vaya a la carpeta raíz del proyecto de Python de muestra. A continuación, vaya a la carpeta **iot-hub\Quickstarts\read-d2c-messages**.

2. Abra el archivo **read_device_to_cloud_messages_sync.py** en un editor de texto de su elección. Actualice las siguientes variables y guarde los cambios en el archivo.

    | Variable | Value |
    | -------- | ----------- |
    | `EVENTHUB_COMPATIBLE_ENDPOINT` | Reemplace el valor de la variable por el punto de conexión compatible con Event Hubs que anotó anteriormente. |
    | `EVENTHUB_COMPATIBLE_PATH`     | Reemplace el valor de la variable por la ruta de acceso compatible con Event Hubs que ha anotó anteriormente. |
    | `IOTHUB_SAS_KEY`                | Reemplace el valor de la variable por la clave principal del servicio que anotó anteriormente. |

3. En la ventana de terminal local, ejecute los comandos siguientes para instalar las bibliotecas necesarias para la aplicación back-end:

    ```cmd/sh
    pip install azure-eventhub
    ```

4. En la ventana de terminal local, ejecute los comandos siguientes para compilar la aplicación back-end y ejecutarla:

    ```cmd/sh
    python read_device_to_cloud_messages_sync.py
    ```

    La siguiente captura de pantalla muestra la salida en la que la aplicación back-end recibe los datos de telemetría que el dispositivo simulado ha enviado al centro:

    ![Ejecutar la aplicación back-end](media/quickstart-send-telemetry-python/read-device-to-cloud.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [iot-hub-quickstarts-clean-up-resources](../../includes/iot-hub-quickstarts-clean-up-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha configurado IoT Hub, registrado un dispositivo, enviado datos de telemetría simulados al centro con una aplicación Python y leído datos de telemetría desde el centro de conectividad mediante una sencilla aplicación de back-end.

Para aprender a controlar el dispositivo simulado desde una aplicación back-end, vaya al siguiente inicio rápido.

> [!div class="nextstepaction"]
> [Inicio rápido: Control de un dispositivo conectado a IoT Hub](quickstart-control-device-python.md)
