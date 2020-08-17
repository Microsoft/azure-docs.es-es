---
title: Introducción a la administración de dispositivos de Azure IoT Hub (Python) | Microsoft Docs
description: Describe cómo usar la administración de dispositivos de IoT Hub para iniciar un reinicio del dispositivo remoto. Usará el SDK de Azure IoT para Python con el fin de implementar una aplicación de dispositivo simulado que incluya un método directo y una aplicación de servicio que invoque el método directo.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.devlang: python
ms.topic: conceptual
ms.date: 01/17/2020
ms.author: robinsh
ms.custom: mqtt, devx-track-python
ms.openlocfilehash: d4c514042b89341b90b0bb9c939ef4b463741916
ms.sourcegitcommit: dea88d5e28bd4bbd55f5303d7d58785fad5a341d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87872725"
---
# <a name="get-started-with-device-management-python"></a>Introducción a la administración de dispositivos (Python)

[!INCLUDE [iot-hub-selector-dm-getstarted](../../includes/iot-hub-selector-dm-getstarted.md)]

En este tutorial se muestra cómo realizar las siguientes acciones:

* Usar Azure Portal para un IoT Hub y una identidad de dispositivo en este.

* Crear una aplicación de dispositivo simulado que contiene un método directo que reinicia ese dispositivo. Los métodos directos se invocan desde la nube.

* Crear una aplicación de consola de Python que llame al método directo de reinicio en la aplicación de dispositivo simulado mediante su centro de IoT.

Al final de este tutorial tendrá dos aplicaciones de consola de Python:

* **dmpatterns_getstarted_device.js**, que conecta al centro de IoT con la identidad del dispositivo creada anteriormente, recibe un método directo de reinicio, simula un reinicio físico y notifica la hora del último reinicio.

* **dmpatterns_getstarted_service.js**, que llama a un método directo en la aplicación de dispositivo simulado, muestra la respuesta y muestra las propiedades notificadas actualizadas.

[!INCLUDE [iot-hub-include-python-sdk-note](../../includes/iot-hub-include-python-sdk-note.md)]

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [iot-hub-include-python-installation-notes](../../includes/iot-hub-include-python-v2-installation-notes.md)]

* Asegúrese de que el puerto 8883 está abierto en el firewall. En el ejemplo de dispositivo de este artículo se usa el protocolo MQTT, que se comunica mediante el puerto 8883. Este puerto puede estar bloqueado en algunos entornos de red corporativos y educativos. Para más información y saber cómo solucionar este problema, consulte [Conexión a IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="create-an-iot-hub"></a>Crear un centro de IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-new-device-in-the-iot-hub"></a>Registro de un nuevo dispositivo en el centro de IoT

[!INCLUDE [iot-hub-get-started-create-device-identity](../../includes/iot-hub-get-started-create-device-identity.md)]

## <a name="create-a-simulated-device-app"></a>Creación de una aplicación de dispositivo simulado

En esta sección:

* Creará una aplicación de consola de Python que responda a un método directo que se llama desde la nube

* Simulará el reinicio de un dispositivo

* Usará las propiedades notificadas para permitir consultas de dispositivo gemelo a fin de identificar los dispositivos y cuándo se reiniciaron por última vez.

1. En el símbolo del sistema, ejecute el siguiente comando para instalar el paquete **azure-iot-device**:

    ```cmd/sh
    pip install azure-iot-device
    ```

2. Con un editor de texto, cree un archivo denominado **dmpatterns_getstarted_service.py** en el directorio de trabajo.

3. Agregue las siguientes instrucciones `import` al comienzo del archivo **dmpatterns_getstarted_device.py**.

    ```python
    import threading
    import time
    import datetime
    from azure.iot.device import IoTHubDeviceClient, MethodResponse
    ```

4. Agregue la variable **CONNECTION_STRING**. Reemplace el valor del marcador de posición `{deviceConnectionString}` por la cadena de conexión del dispositivo. Ha copiado esta cadena de conexión previamente en [Registro de un nuevo dispositivo en el centro de IoT](#register-a-new-device-in-the-iot-hub).  

    ```python
    CONNECTION_STRING = "{deviceConnectionString}"
    ```

5. Agregue las siguientes devoluciones de llamada a la función para implementar el método directo en el dispositivo.

    ```python
    def reboot_listener(client):
        while True:
            # Receive the direct method request
            method_request = client.receive_method_request("rebootDevice")  # blocking call

            # Act on the method by rebooting the device...
            print( "Rebooting device" )
            time.sleep(20)
            print( "Device rebooted")

            # ...and patching the reported properties
            current_time = str(datetime.datetime.now())
            reported_props = {"rebootTime": current_time}
            client.patch_twin_reported_properties(reported_props)
            print( "Device twins updated with latest rebootTime")

            # Send a method response indicating the method request was resolved
            resp_status = 200
            resp_payload = {"Response": "This is the response from the device"}
            method_response = MethodResponse(method_request.request_id, resp_status, resp_payload)
            client.send_method_response(method_response)
    ```

6. Inicie el agente de escucha de método directo y espere.

    ```python
    def iothub_client_init():
        client = IoTHubDeviceClient.create_from_connection_string(CONNECTION_STRING)
        return client

    def iothub_client_sample_run():
        try:
            client = iothub_client_init()

            # Start a thread listening for "rebootDevice" direct method invocations
            reboot_listener_thread = threading.Thread(target=reboot_listener, args=(client,))
            reboot_listener_thread.daemon = True
            reboot_listener_thread.start()

            while True:
                time.sleep(1000)

        except KeyboardInterrupt:
            print ( "IoTHubDeviceClient sample stopped" )

    if __name__ == '__main__':
        print ( "Starting the IoT Hub Python sample..." )
        print ( "IoTHubDeviceClient waiting for commands, press Ctrl-C to exit" )

        iothub_client_sample_run()
    ```

7. Guarde y cierre el archivo **dmpatterns_getstarted_device.py**.

> [!NOTE]
> Por simplificar, este tutorial no implementa ninguna directiva de reintentos. En el código de producción, deberá implementar directivas de reintentos (por ejemplo, retroceso exponencial), tal y como se sugiere en el artículo [Control de errores transitorios](/azure/architecture/best-practices/transient-faults).

## <a name="get-the-iot-hub-connection-string"></a>Obtención de la cadena de conexión de IoT Hub

[!INCLUDE [iot-hub-howto-device-management-shared-access-policy-text](../../includes/iot-hub-howto-device-management-shared-access-policy-text.md)]

[!INCLUDE [iot-hub-include-find-service-connection-string](../../includes/iot-hub-include-find-service-connection-string.md)]

## <a name="trigger-a-remote-reboot-on-the-device-using-a-direct-method"></a>Desencadenar un reinicio remoto en el dispositivo con un método directo

En esta sección, creará una aplicación de consola de Python que inicia un reinicio remoto en un dispositivo mediante un método directo. La aplicación usa las consultas gemelas de dispositivo para detectar la hora en que se reinició por última vez el dispositivo.

1. En el símbolo del sistema, ejecute el siguiente comando para instalar el paquete **azure-iot-hub**:

    ```cmd/sh
    pip install azure-iot-hub
    ```

2. Con un editor de texto, cree un archivo denominado **dmpatterns_getstarted_service.py** en el directorio de trabajo.

3. Agregue las siguientes instrucciones `import` al comienzo del archivo **dmpatterns_getstarted_service.py**.

    ```python
    import sys, time

    from azure.iot.hub import IoTHubRegistryManager
    from azure.iot.hub.models import CloudToDeviceMethod, CloudToDeviceMethodResult, Twin
    ```

4. Agregue las siguientes declaraciones de variable. Reemplace el valor del marcador de posición `{IoTHubConnectionString}` por la cadena de conexión del centro de IoT que copió anteriormente en [Obtención de la cadena de conexión de IoT Hub](#get-the-iot-hub-connection-string). Reemplace el marcador de posición `{deviceId}` por el identificador de dispositivo que registró en [Registro de un nuevo dispositivo en el centro de IoT](#register-a-new-device-in-the-iot-hub).

    ```python
    CONNECTION_STRING = "{IoTHubConnectionString}"
    DEVICE_ID = "{deviceId}"

    METHOD_NAME = "rebootDevice"
    METHOD_PAYLOAD = "{\"method_number\":\"42\"}"
    TIMEOUT = 60
    WAIT_COUNT = 10
    ```

5. Agregue la siguiente función para invocar el método de reinicio del dispositivo de destino, luego consulte los dispositivos gemelos y obtenga la hora del último reinicio.

    ```python
    def iothub_devicemethod_sample_run():
        try:
            # Create IoTHubRegistryManager
            registry_manager = IoTHubRegistryManager(CONNECTION_STRING)

            print ( "" )
            print ( "Invoking device to reboot..." )

            # Call the direct method.
            deviceMethod = CloudToDeviceMethod(method_name=METHOD_NAME, payload=METHOD_PAYLOAD)
            response = registry_manager.invoke_device_method(DEVICE_ID, deviceMethod)

            print ( "" )
            print ( "Successfully invoked the device to reboot." )

            print ( "" )
            print ( response.payload )

            while True:
                print ( "" )
                print ( "IoTHubClient waiting for commands, press Ctrl-C to exit" )

                status_counter = 0
                while status_counter <= WAIT_COUNT:
                    twin_info = registry_manager.get_twin(DEVICE_ID)

                    if twin_info.properties.reported.get("rebootTime") != None :
                        print ("Last reboot time: " + twin_info.properties.reported.get("rebootTime"))
                    else:
                        print ("Waiting for device to report last reboot time...")

                    time.sleep(5)
                    status_counter += 1

        except Exception as ex:
            print ( "" )
            print ( "Unexpected error {0}".format(ex) )
            return
        except KeyboardInterrupt:
            print ( "" )
            print ( "IoTHubDeviceMethod sample stopped" )

    if __name__ == '__main__':
        print ( "Starting the IoT Hub Service Client DeviceManagement Python sample..." )
        print ( "    Connection string = {0}".format(CONNECTION_STRING) )
        print ( "    Device ID         = {0}".format(DEVICE_ID) )

        iothub_devicemethod_sample_run()
    ```

6. Guarde y cierre el archivo **dmpatterns_getstarted_service.py**.

## <a name="run-the-apps"></a>Ejecución de las aplicaciones

Ya está preparado para ejecutar las aplicaciones.

1. En el símbolo del sistema, ejecute el siguiente comando para iniciar la escucha del método directo de reinicio.

    ```cmd/sh
    python dmpatterns_getstarted_device.py
    ```

2. En otro símbolo del sistema, ejecute el siguiente comando para desencadenar el reinicio remoto y la consulta en el dispositivo gemelo para buscar la hora del último reinicio.

    ```cmd/sh
    python dmpatterns_getstarted_service.py
    ```

3. Verá la respuesta del dispositivo al método directo en la consola.

   A continuación, puede ver la respuesta del dispositivo al método directo de reinicio:

   ![Salida de una aplicación de dispositivo simulado](./media/iot-hub-python-python-device-management-get-started/device.png)

   A continuación se muestra el servicio que llama al método directo de reinicio y sondea el estado del dispositivo gemelo:

   ![Salida del servicio de reinicio del desencadenador](./media/iot-hub-python-python-device-management-get-started/service.png)

[!INCLUDE [iot-hub-dm-followup](../../includes/iot-hub-dm-followup.md)]
