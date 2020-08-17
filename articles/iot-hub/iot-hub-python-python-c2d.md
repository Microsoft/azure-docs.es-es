---
title: Mensajes de nube a dispositivo con IoT Hub de Azure (Python) | Microsoft Docs
description: Cómo enviar mensajes de nube a un dispositivo de una instancia de IoT Hub de Azure mediante los SDK de IoT de Azure para Python. Modifique una aplicación de dispositivo simulado para recibir mensajes de nube a dispositivo y cambie una aplicación de back-end para enviar los mensajes de nube a dispositivo.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.devlang: python
ms.topic: conceptual
ms.date: 04/09/2020
ms.author: robinsh
ms.custom: mqtt, devx-track-python
ms.openlocfilehash: ad6399a4713520ca0550d143cf3f19f87d55337c
ms.sourcegitcommit: dea88d5e28bd4bbd55f5303d7d58785fad5a341d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87876808"
---
# <a name="send-cloud-to-device-messages-with-iot-hub-python"></a>Envío de mensajes de nube a dispositivo con IoT Hub (Python)

[!INCLUDE [iot-hub-selector-c2d](../../includes/iot-hub-selector-c2d.md)]

IoT Hub de Azure es un servicio totalmente administrado que permite la comunicación bidireccional confiable y segura entre millones de dispositivos y una solución de back-end. En el inicio rápido [Envío de telemetría desde un dispositivo a un centro de IoT](quickstart-send-telemetry-python.md) se muestra cómo crear un centro de IoT, aprovisionar en él la identidad del dispositivo y codificar una aplicación de dispositivo simulado que envíe mensajes del dispositivo a la nube.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

Este tutorial se basa en el artículo [Envío de telemetría desde un dispositivo a un centro de IoT](quickstart-send-telemetry-python.md). En él se muestra cómo realizar las siguientes acciones:

* Desde el back-end de la nube de la aplicación, envíe mensajes de nube a dispositivo en un único dispositivo a través de IoT Hub.

* Reciba mensajes de nube a dispositivo en un dispositivo.

Encontrará más información sobre los mensajes de nube a dispositivo en la [Guía para desarrolladores de IoT Hub](iot-hub-devguide-messaging.md).

Al final de este tutorial tendrá dos aplicaciones de consola de Python:

* **SimulatedDevice.py**, una versión modificada de la aplicación que se creó en [Envío de telemetría desde un dispositivo a un centro de IoT](quickstart-send-telemetry-python.md), que se conecta al centro de IoT y recibe mensajes de la nube al dispositivo.

* **SendCloudToDeviceMessage.py**, que envía un mensaje de la nube a la aplicación de dispositivo simulado mediante IoT Hub.

[!INCLUDE [iot-hub-include-python-sdk-note](../../includes/iot-hub-include-python-sdk-note.md)]

## <a name="prerequisites"></a>Prerrequisitos

[!INCLUDE [iot-hub-include-python-v2-installation-notes](../../includes/iot-hub-include-python-v2-installation-notes.md)]

* Asegúrese de que el puerto 8883 está abierto en el firewall. En el ejemplo de dispositivo de este artículo se usa el protocolo MQTT, que se comunica mediante el puerto 8883. Este puerto puede estar bloqueado en algunos entornos de red corporativos y educativos. Para más información y saber cómo solucionar este problema, consulte [Conexión a IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="receive-messages-in-the-simulated-device-app"></a>Recepción de mensajes en la aplicación de dispositivo simulado

En esta sección, creará una aplicación de consola de Python para simular el dispositivo y recibir mensajes de la nube al dispositivo desde IoT Hub.

1. En un símbolo del sistema del directorio de trabajo, instale el **SDK del dispositivo de Azure IoT Hub para Python**:

    ```cmd/sh
    pip install azure-iot-device
    ```

1. Con un editor de texto, cree un archivo llamado **SimulatedDevice.py**.

1. Agregue las siguientes instrucciones y variables `import` al principio del archivo **SimulatedDevice.py**:

    ```python
    import threading
    import time
    from azure.iot.device import IoTHubDeviceClient

    RECEIVED_MESSAGES = 0
    ```

1. Agregue el código siguiente al archivo **SimulatedDevice.py**. Sustituya el valor de marcador de posición `{deviceConnectionString}` por la cadena de conexión del dispositivo para el dispositivo que ha creado en el inicio rápido [Envío de telemetría desde un dispositivo a un centro de IoT](quickstart-send-telemetry-python.md):

    ```python
    CONNECTION_STRING = "{deviceConnectionString}"
    ```

1. Agregue la función siguiente para imprimir los mensajes recibidos en la consola:

    ```python
    def message_listener(client):
        global RECEIVED_MESSAGES
        while True:
            message = client.receive_message()
            RECEIVED_MESSAGES += 1
            print("\nMessage received:")

            #print data and both system and application (custom) properties
            for property in vars(message).items():
                print ("    {0}".format(property))

            print( "Total calls received: {}".format(RECEIVED_MESSAGES))
            print()
    ```

1. Agregue el código siguiente para inicializar el cliente y espere a recibir el mensaje de la nube al dispositivo:

    ```python
    def iothub_client_sample_run():
        try:
            client = IoTHubDeviceClient.create_from_connection_string(CONNECTION_STRING)

            message_listener_thread = threading.Thread(target=message_listener, args=(client,))
            message_listener_thread.daemon = True
            message_listener_thread.start()

            while True:
                time.sleep(1000)

        except KeyboardInterrupt:
            print ( "IoT Hub C2D Messaging device sample stopped" )
    ```

1. Agregue la siguiente función main:

    ```python
    if __name__ == '__main__':
        print ( "Starting the Python IoT Hub C2D Messaging device sample..." )
        print ( "Waiting for C2D messages, press Ctrl-C to exit" )

        iothub_client_sample_run()
    ```

1. Guarde y cierre el archivo **SimulatedDevice.py**.

## <a name="get-the-iot-hub-connection-string"></a>Obtención de la cadena de conexión de IoT Hub

En este artículo, se crea un servicio de back-end para enviar mensajes de la nube a un dispositivo a través de la instancia de IoT Hub que creó en [Envío de telemetría desde un dispositivo a un centro de IoT](quickstart-send-telemetry-python.md). Para enviar mensajes de nube a un dispositivo, el servicio necesita el permiso de **conexión de servicio**. De forma predeterminada, todas las instancias de IoT Hub se crean con una directiva de acceso compartido denominada **servicio** que concede este permiso.

[!INCLUDE [iot-hub-include-find-service-connection-string](../../includes/iot-hub-include-find-service-connection-string.md)]

## <a name="send-a-cloud-to-device-message"></a>Envío de mensajes de nube a dispositivo

En esta sección, escribirá una aplicación de consola de Python que envía mensajes de nube a dispositivo a la aplicación del dispositivo simulado. Necesitará el identificador del dispositivo que agregó en el inicio rápido [Envío de telemetría desde un dispositivo a un centro de IoT](quickstart-send-telemetry-python.md). También necesitará la cadena de conexión de IoT Hub que copió anteriormente en [Obtención de la cadena de conexión de IoT Hub](#get-the-iot-hub-connection-string).

1. En el directorio de trabajo, abra un símbolo del sistema e instale el **SDK del servicio Azure IoT Hub para Python**.

   ```cmd/sh
   pip install azure-iot-hub
   ```

1. Con un editor de texto, cree un archivo llamado **SendCloudToDeviceMessage.py**.

1. Agregue las siguientes instrucciones y variables `import` al principio del archivo **SendCloudToDeviceMessage.py**:

    ```python
    import random
    import sys
    from azure.iot.hub import IoTHubRegistryManager

    MESSAGE_COUNT = 2
    AVG_WIND_SPEED = 10.0
    MSG_TXT = "{\"service client sent a message\": %.2f}"
    ```

1. Agregue el código siguiente al archivo **SendCloudToDeviceMessage.py**. Reemplace los valores de marcador de posición `{iot hub connection string}` y `{device id}` por la cadena de conexión de IoT Hub y el identificador de dispositivo que anotó anteriormente:

    ```python
    CONNECTION_STRING = "{IoTHubConnectionString}"
    DEVICE_ID = "{deviceId}"
    ```

1. Agregue el código siguiente para enviar mensajes al dispositivo:

    ```python
    def iothub_messaging_sample_run():
        try:
            # Create IoTHubRegistryManager
            registry_manager = IoTHubRegistryManager(CONNECTION_STRING)

            for i in range(0, MESSAGE_COUNT):
                print ( 'Sending message: {0}'.format(i) )
                data = MSG_TXT % (AVG_WIND_SPEED + (random.random() * 4 + 2))

                props={}
                # optional: assign system properties
                props.update(messageId = "message_%d" % i)
                props.update(correlationId = "correlation_%d" % i)
                props.update(contentType = "application/json")

                # optional: assign application properties
                prop_text = "PropMsg_%d" % i
                props.update(testProperty = prop_text)

                registry_manager.send_c2d_message(DEVICE_ID, data, properties=props)

            try:
                # Try Python 2.xx first
                raw_input("Press Enter to continue...\n")
            except:
                pass
                # Use Python 3.xx in the case of exception
                input("Press Enter to continue...\n")

        except Exception as ex:
            print ( "Unexpected error {0}" % ex )
            return
        except KeyboardInterrupt:
            print ( "IoT Hub C2D Messaging service sample stopped" )
    ```

1. Agregue la siguiente función main:

    ```python
    if __name__ == '__main__':
        print ( "Starting the Python IoT Hub C2D Messaging service sample..." )

        iothub_messaging_sample_run()
    ```

1. Guarde y cierre el archivo **SendCloudToDeviceMessage.py**.

## <a name="run-the-applications"></a>Ejecución de las aplicaciones

Ahora está preparado para ejecutar las aplicaciones.

1. En el símbolo del sistema en el directorio de trabajo, ejecute el comando siguiente para escuchar mensajes de nube a dispositivo:

    ```shell
    python SimulatedDevice.py
    ```

    ![Ejecución de una aplicación de dispositivo simulada](./media/iot-hub-python-python-c2d/device-1.png)

1. Abra un nuevo símbolo del sistema en el directorio de trabajo y ejecute el siguiente comando para enviar mensajes de nube a dispositivo:

    ```shell
    python SendCloudToDeviceMessage.py
    ```

    ![Ejecución de la aplicación para enviar el comando de nube a dispositivo](./media/iot-hub-python-python-c2d/service.png)

1. Anote el mensaje que recibirá en el dispositivo.

    ![Mensaje recibido](./media/iot-hub-python-python-c2d/device-2.png)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, aprendió a enviar y recibir mensajes de nube a dispositivo.

Para ver ejemplos de soluciones de un extremo a otro que usen IoT Hub, vea el [Acelerador de la solución de supervisión remota de Azure IoT](https://azure.microsoft.com/documentation/suites/iot-suite/).

Para obtener más información sobre cómo desarrollar soluciones con IoT Hub, consulte la [Guía para desarrolladores de IoT Hub](iot-hub-devguide.md).
