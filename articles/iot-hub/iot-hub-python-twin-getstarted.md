---
title: Introducción a los dispositivos gemelos de Azure IoT Hub (Python) | Microsoft Docs
description: Describe cómo usar dispositivos gemelos de IoT Hub de Azure para agregar etiquetas y, luego, usar una consulta de IoT Hub. El SDK de IoT de Azure para Python se usa para implementar la aplicación de dispositivo simulado y una aplicación de servicio que agrega las etiquetas y ejecuta la consulta de IoT Hub.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.devlang: python
ms.topic: conceptual
ms.date: 03/11/2020
ms.author: robinsh
ms.custom: mqtt, devx-track-python
ms.openlocfilehash: 12b1d083ae1481f7c8b5fe60cac9156a56aeaa0a
ms.sourcegitcommit: dea88d5e28bd4bbd55f5303d7d58785fad5a341d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87875479"
---
# <a name="get-started-with-device-twins-python"></a>Introducción a los dispositivos gemelos (Python)

[!INCLUDE [iot-hub-selector-twin-get-started](../../includes/iot-hub-selector-twin-get-started.md)]

Al final de este tutorial, tendrá dos aplicaciones de consola de Python:

* **AddTagsAndQuery.py**, una aplicación de Python de back-end diseñada para ejecutarse desde el back-end, que agrega etiquetas y consulta dispositivos gemelos.

* **ReportConnectivity.py**, una aplicación Python para dispositivo que simula un dispositivo que se conecta a su instancia de IoT Hub con la identidad del dispositivo creada anteriormente e informa de su estado de conectividad.

[!INCLUDE [iot-hub-include-python-sdk-note](../../includes/iot-hub-include-python-sdk-note.md)]

## <a name="prerequisites"></a>Prerrequisitos

[!INCLUDE [iot-hub-include-python-v2-installation-notes](../../includes/iot-hub-include-python-v2-installation-notes.md)]

* Asegúrese de que el puerto 8883 está abierto en el firewall. En el ejemplo de dispositivo de este artículo se usa el protocolo MQTT, que se comunica mediante el puerto 8883. Este puerto puede estar bloqueado en algunos entornos de red corporativos y educativos. Para más información y saber cómo solucionar este problema, consulte [Conexión a IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="create-an-iot-hub"></a>Crear un centro de IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-new-device-in-the-iot-hub"></a>Registro de un nuevo dispositivo en el centro de IoT

[!INCLUDE [iot-hub-include-create-device](../../includes/iot-hub-include-create-device.md)]

## <a name="get-the-iot-hub-connection-string"></a>Obtención de la cadena de conexión de IoT Hub

[!INCLUDE [iot-hub-howto-twin-shared-access-policy-text](../../includes/iot-hub-howto-twin-shared-access-policy-text.md)]

[!INCLUDE [iot-hub-include-find-custom-connection-string](../../includes/iot-hub-include-find-custom-connection-string.md)]

## <a name="create-the-service-app"></a>Creación de la aplicación de servicio

En esta sección, creará una aplicación de consola de Python que agrega metadatos de ubicación al dispositivo gemelo asociado con **{Device ID}** . Después, consulta los dispositivos gemelos almacenados en la instancia de IoT Hub mediante la selección de los dispositivos ubicados en Redmond (EE. UU.) y, después, los que informan de una conexión de red de telefonía móvil.

1. En el directorio de trabajo, abra un símbolo del sistema e instale el **SDK del servicio Azure IoT Hub para Python**.

   ```cmd/sh
   pip install azure-iot-hub
   ```

2. Con un editor de texto, cree un nuevo archivo **AddTagsAndQuery.py**.

3. Agregue el código siguiente para importar los módulos necesarios desde el SDK del servicio:

   ```python
   import sys
   from time import sleep
   from azure.iot.hub import IoTHubRegistryManager
   from azure.iot.hub.models import Twin, TwinProperties, QuerySpecification, QueryResult
   ```

4. Agregue el código siguiente: Reemplace `[IoTHub Connection String]` por la cadena de conexión de IoT Hub que copió en [Obtención de la cadena de conexión de IoT Hub](#get-the-iot-hub-connection-string). Reemplace `[Device Id]` por el identificador de dispositivo que registró en [Registro de un nuevo dispositivo en el centro de IoT](#register-a-new-device-in-the-iot-hub).
  
    ```python
    IOTHUB_CONNECTION_STRING = "[IoTHub Connection String]"
    DEVICE_ID = "[Device Id]"
    ```

5. Agregue el código siguiente al archivo **AddTagsAndQuery.py**:

    ```python
    def iothub_service_sample_run():
        try:
            iothub_registry_manager = IoTHubRegistryManager(IOTHUB_CONNECTION_STRING)

            new_tags = {
                    'location' : {
                        'region' : 'US',
                        'plant' : 'Redmond43'
                    }
                }

            twin = iothub_registry_manager.get_twin(DEVICE_ID)
            twin_patch = Twin(tags=new_tags, properties= TwinProperties(desired={'power_level' : 1}))
            twin = iothub_registry_manager.update_twin(DEVICE_ID, twin_patch, twin.etag)

            # Add a delay to account for any latency before executing the query
            sleep(1)

            query_spec = QuerySpecification(query="SELECT * FROM devices WHERE tags.location.plant = 'Redmond43'")
            query_result = iothub_registry_manager.query_iot_hub(query_spec, None, 100)
            print("Devices in Redmond43 plant: {}".format(', '.join([twin.device_id for twin in query_result.items])))

            print()

            query_spec = QuerySpecification(query="SELECT * FROM devices WHERE tags.location.plant = 'Redmond43' AND properties.reported.connectivity = 'cellular'")
            query_result = iothub_registry_manager.query_iot_hub(query_spec, None, 100)
            print("Devices in Redmond43 plant using cellular network: {}".format(', '.join([twin.device_id for twin in query_result.items])))

        except Exception as ex:
            print("Unexpected error {0}".format(ex))
            return
        except KeyboardInterrupt:
            print("IoT Hub Device Twin service sample stopped")
    ```

    El objeto **IoTHubRegistryManager** expone todos los métodos necesarios para interactuar con dispositivos gemelos del servicio. En primer lugar, el código inicializa el objeto **IoTHubRegistryManager** y, después, actualiza el dispositivo gemelo para **DEVICE_ID** que, por último, ejecuta dos consultas. La primera selecciona solo los dispositivos gemelos de los dispositivos que se encuentran en la planta **Redmond43** y la segunda mejora la consulta para seleccionar solo los dispositivos que están también conectados a través de la red de telefonía móvil.

6. Agregue el código siguiente al final de **AddTagsAndQuery.py** para implementar la función **iothub_service_sample_run**:

    ```python
    if __name__ == '__main__':
        print("Starting the Python IoT Hub Device Twin service sample...")
        print()

        iothub_service_sample_run()
    ```

7. Ejecute la aplicación con:

    ```cmd/sh
    python AddTagsAndQuery.py
    ```

    Debería ver un dispositivo en los resultados de la consulta que pregunta por todos los dispositivos que se encuentran en **Redmond43** y ninguno para la consulta que restringe los resultados a los dispositivos que utilizan una red de telefonía móvil.

    ![primera consulta que muestra todos los dispositivos de Redmond](./media/iot-hub-python-twin-getstarted/service-1.png)

En la siguiente sección, creará una aplicación de dispositivo que notifica la información de conectividad y cambia el resultado de la consulta en la sección anterior.

## <a name="create-the-device-app"></a>Creación de la aplicación del dispositivo

En esta sección, creará una aplicación de consola de Python que se conecta al centro como su **{Device ID}** y, luego, actualiza las propiedades notificadas de su dispositivo gemelo para que contenga la información conectada mediante una red de telefonía móvil.

1. En un símbolo del sistema del directorio de trabajo, instale el **SDK del dispositivo de Azure IoT Hub para Python**:

    ```cmd/sh
    pip install azure-iot-device
    ```

2. Con un editor de texto, cree un nuevo archivo **ReportConnectivity.py**.

3. Agregue el código siguiente para importar los módulos necesarios desde el SDK del dispositivo:

    ```python
    import time
    import threading
    from azure.iot.device import IoTHubModuleClient
    ```

4. Agregue el código siguiente: Reemplace el valor del marcador de posición `[IoTHub Device Connection String]` por la cadena de conexión de dispositivo que copió en [Registro de un nuevo dispositivo en el centro de IoT](#register-a-new-device-in-the-iot-hub).

    ```python
    CONNECTION_STRING = "[IoTHub Device Connection String]"
    ```

5. Agregue el código siguiente al archivo **ReportConnectivity.py** archivo para implementar la funcionalidad de dispositivos gemelos:

    ```python
    def twin_update_listener(client):
        while True:
            patch = client.receive_twin_desired_properties_patch()  # blocking call
            print("Twin patch received:")
            print(patch)

    def iothub_client_init():
        client = IoTHubModuleClient.create_from_connection_string(CONNECTION_STRING)
        return client

    def iothub_client_sample_run():
        try:
            client = iothub_client_init()

            twin_update_listener_thread = threading.Thread(target=twin_update_listener, args=(client,))
            twin_update_listener_thread.daemon = True
            twin_update_listener_thread.start()

            # Send reported 
            print ( "Sending data as reported property..." )
            reported_patch = {"connectivity": "cellular"}
            client.patch_twin_reported_properties(reported_patch)
            print ( "Reported properties updated" )

            while True:
                time.sleep(1000000)
        except KeyboardInterrupt:
            print ( "IoT Hub Device Twin device sample stopped" )
    ```

    El objeto **IoTHubModuleClient** expone todos los métodos necesarios para interactuar con dispositivos gemelos del dispositivo. El código anterior, después de inicializar el objeto **IoTHubModuleClient**, recupera el dispositivo gemelo del dispositivo y actualiza su propiedad notificada con la información de conectividad.

6. Agregue el código siguiente al final de **ReportConnectivity.py** para implementar la función **iothub_client_sample_run**:

    ```python
    if __name__ == '__main__':
        print ( "Starting the Python IoT Hub Device Twin device sample..." )
        print ( "IoTHubModuleClient waiting for commands, press Ctrl-C to exit" )

        iothub_client_sample_run()
    ```

7. Ejecute la aplicación del dispositivo:

    ```cmd/sh
    python ReportConnectivity.py
    ```

    Debería ver la confirmación de que se actualizaron las propiedades notificadas de los dispositivos gemelos.

    ![Actualización de las propiedades notificadas desde la aplicación del dispositivo](./media/iot-hub-python-twin-getstarted/device-1.png)

8. Ahora que el dispositivo ha informado sobre su información de conectividad, debe aparecer en ambas consultas. Vuelva atrás y ejecute las consultas de nuevo:

    ```cmd/sh
    python AddTagsAndQuery.py
    ```

    Esta vez **{Device Id}** debe aparecer en los resultados de ambas consultas.

    ![Segunda consulta sobre la aplicación de servicio](./media/iot-hub-python-twin-getstarted/service-2.png)

    En la aplicación del dispositivo, verá una confirmación de la recepción de la revisión de las propiedades gemelas deseadas enviadas por la aplicación de servicio.

    ![Recepción de las propiedades deseadas en la aplicación del dispositivo](./media/iot-hub-python-twin-getstarted/device-2.png)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, configuró un centro de IoT nuevo en Azure Portal y, después, creó una identidad de dispositivo en el registro de identidades del centro de IoT. Ha agregado metadatos de dispositivo como etiquetas desde una aplicación back-end y ha escrito una aplicación de dispositivo simulado para notificar la información de conectividad de dispositivos en el dispositivo gemelo. También ha aprendido cómo consultar esta información mediante el Registro.

Use los siguientes recursos para obtener información sobre cómo:

* Enviar telemetría desde dispositivos con el tutorial [Introducción a IoT Hub](quickstart-send-telemetry-python.md).

* Configurar dispositivos mediante las propiedades deseadas del dispositivo gemelo con el tutorial [Uso de las propiedades deseadas para configurar dispositivos](tutorial-device-twins.md).

* Controlar los dispositivos de forma interactiva (por ejemplo, encender un ventilador desde una aplicación controlada por el usuario), con el tutorial [Uso de métodos directos](quickstart-control-device-python.md).
