---
title: Identidad de módulo y módulo gemelo de Azure IoT Hub (Python)
description: Aprenda a crear la identidad de módulo y a actualizar módulos gemelos mediante los SDK de IoT para Python.
author: chrissie926
ms.service: iot-hub
services: iot-hub
ms.devlang: python
ms.topic: conceptual
ms.date: 04/03/2020
ms.author: menchi
ms.custom: devx-track-python
ms.openlocfilehash: 6ed0290abe91453058589b032fb5ed2dd71fe98a
ms.sourcegitcommit: dea88d5e28bd4bbd55f5303d7d58785fad5a341d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87872572"
---
# <a name="get-started-with-iot-hub-module-identity-and-module-twin-python"></a>Introducción a la identidad de módulo y a los módulos gemelos de IoT Hub (Python)

[!INCLUDE [iot-hub-selector-module-twin-getstarted](../../includes/iot-hub-selector-module-twin-getstarted.md)]

> [!NOTE]
> [Las identidades de módulo y los módulos gemelos](iot-hub-devguide-module-twins.md) son similares a las identidades de dispositivo y los dispositivos gemelos de Azure IoT Hub, pero ofrecen granularidad más fina. Aunque las identidades de dispositivo y los dispositivos gemelos de Azure IoT Hub permiten que una aplicación back-end configure un dispositivo y proporcionan visibilidad para las condiciones del dispositivo, las identidades de módulo y los módulos gemelos proporcionan estas funcionalidades para los componentes individuales del dispositivo. En los dispositivos compatibles con varios componentes, como dispositivos con sistema operativo o firmware, permiten la configuración individual y condiciones específicas por componente.
>

Al final de este tutorial, tiene tres aplicaciones de Python:

* **CreateModule**, que crea una identidad de dispositivo, una de módulo y las claves de seguridad asociadas para conectar el dispositivo y los clientes del módulo.

* **UpdateModuleTwinDesiredProperties**, que envía las propiedades deseadas del módulo gemelo actualizadas a IoT Hub.

* **ReceiveModuleTwinDesiredPropertiesPatch**, que recibe la revisión de las propiedades deseadas del módulo gemelo en el dispositivo.

[!INCLUDE [iot-hub-include-python-sdk-note](../../includes/iot-hub-include-python-sdk-note.md)]

## <a name="prerequisites"></a>Prerrequisitos

[!INCLUDE [iot-hub-include-python-v2-installation-notes](../../includes/iot-hub-include-python-v2-installation-notes.md)]

## <a name="create-an-iot-hub"></a>Crear un centro de IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="get-the-iot-hub-connection-string"></a>Obtención de la cadena de conexión de IoT Hub

En este artículo, va a crear un servicio back-end que agregará un dispositivo en el registro de identidades y, a continuación, agregará un módulo a ese dispositivo. Este servicio requiere el permiso de **escribir en el Registro** (que también incluye el permiso de **leer el Registro**). También creará un servicio que agrega las propiedades deseadas al módulo gemelo del módulo recién creado. Este servicio necesita el permiso de **conexión del servicio**. Si bien hay directivas de acceso compartido predeterminadas que conceden estos permisos de forma individual, en esta sección se crea una directiva de acceso compartido personalizada que contiene ambos permisos.

[!INCLUDE [iot-hub-include-find-service-regrw-connection-string](../../includes/iot-hub-include-find-service-regrw-connection-string.md)]

## <a name="create-a-device-identity-and-a-module-identity-in-iot-hub"></a>Crear una identidad del dispositivo y una identidad de módulo en IoT Hub

En esta sección se crea una aplicación de servicio de Python que crea a su vez una identidad de dispositivo y otra de módulo en el registro de identidades de la instancia de IoT Hub. No se puede conectar un dispositivo o un módulo a IoT Hub a menos que tenga una entrada en el registro de identidades. Para más información, consulte [Descripción del registro de identidades de un centro de IoT](iot-hub-devguide-identity-registry.md). Cuando se ejecuta esta aplicación de consola, genera una clave y un identificador únicos para el dispositivo y el módulo. El dispositivo y el módulo usan estos valores para identificarse al enviar mensajes del dispositivo a la nube a IoT Hub. Los identificadores distinguen mayúsculas de minúsculas.

1. En el símbolo del sistema, ejecute el siguiente comando para instalar el paquete **azure-iot-hub**:

    ```cmd/sh
    pip install azure-iot-hub
    ```

1. En el símbolo del sistema, ejecute el siguiente comando para instalar el paquete **msrest**. Necesita este paquete para capturar excepciones **HTTPOperationError**.

    ```cmd/sh
    pip install msrest
    ```

1. Con un editor de texto, cree un archivo denominado **CreateModule.py** en el directorio de trabajo.

1. Agregue el código siguiente al archivo de Python. Reemplace *YourIoTHubConnectionString* por la cadena de conexión que copió en [Obtención de la cadena de conexión de IoT Hub](#get-the-iot-hub-connection-string).

    ```python
    import sys
    from msrest.exceptions import HttpOperationError
    from azure.iot.hub import IoTHubRegistryManager

    CONNECTION_STRING = "YourIotHubConnectionString"
    DEVICE_ID = "myFirstDevice"
    MODULE_ID = "myFirstModule"

    try:
        # RegistryManager
        iothub_registry_manager = IoTHubRegistryManager(CONNECTION_STRING)

        try:
            # CreateDevice - let IoT Hub assign keys
            primary_key = ""
            secondary_key = ""
            device_state = "enabled"
            new_device = iothub_registry_manager.create_device_with_sas(
                DEVICE_ID, primary_key, secondary_key, device_state
            )
        except HttpOperationError as ex:
            if ex.response.status_code == 409:
                # 409 indicates a conflict. This happens because the device already exists.
                new_device = iothub_registry_manager.get_device(DEVICE_ID)
            else:
                raise

        print("device <" + DEVICE_ID +
              "> has primary key = " + new_device.authentication.symmetric_key.primary_key)

        try:
            # CreateModule - let IoT Hub assign keys
            primary_key = ""
            secondary_key = ""
            managed_by = ""
            new_module = iothub_registry_manager.create_module_with_sas(
                DEVICE_ID, MODULE_ID, managed_by, primary_key, secondary_key
            )
        except HttpOperationError as ex:
            if ex.response.status_code == 409:
                # 409 indicates a conflict. This happens because the module already exists.
                new_module = iothub_registry_manager.get_module(DEVICE_ID, MODULE_ID)
            else:
                raise

        print("device/module <" + DEVICE_ID + "/" + MODULE_ID +
              "> has primary key = " + new_module.authentication.symmetric_key.primary_key)

    except Exception as ex:
        print("Unexpected error {0}".format(ex))
    except KeyboardInterrupt:
        print("IoTHubRegistryManager sample stopped")
    ```

1. En el símbolo del sistema, ejecute el comando siguiente:

    ```cmd/sh
    python CreateModule.py
    ```

Esta aplicación crea una identidad del dispositivo con el identificador **myFirstDevice** y una de módulo con el identificador **myFirstModule** en el dispositivo **myFirstDevice**. (Si el id. del dispositivo o del módulo ya existe en el registro de identidades, el código simplemente recupera la información del dispositivo o módulo existente). La aplicación muestra el identificador y la clave principal de cada identidad.

> [!NOTE]
> El registro de identidades de IoT Hub solo almacena identidades de dispositivos y módulos para permitir el acceso seguro a IoT Hub. El registro de identidades almacena los identificadores y las claves de dispositivo para usarlas como credenciales de seguridad. El registro de identidades también almacena una marca de habilitado o deshabilitado de cada dispositivo que se puede usar para deshabilitar el acceso a dicho dispositivo. Si la aplicación necesita almacenar otros metadatos específicos del dispositivo, debe usar un almacén específico de la aplicación. No hay marcas de habilitado/deshabilitado para las identidades de módulo. Para más información, consulte [Descripción del registro de identidades de un centro de IoT](iot-hub-devguide-identity-registry.md).
>

## <a name="update-the-module-twin-using-python-service-sdk"></a>Actualización del módulo gemelo con el SDK del servicio de Python

En esta sección, creará una aplicación de servicio de Python que actualiza las propiedades deseadas del módulo gemelo.

1. En el símbolo del sistema, ejecute el comando siguiente para instalar el paquete **azure-iot-hub**. Puede omitir este paso si instaló el paquete **azure-iot-hub** en la sección anterior.

    ```cmd/sh
    pip install azure-iot-hub
    ```

1. Con un editor de texto, cree un archivo denominado **UpdateModuleTwinDesiredProperties.py** en el directorio de trabajo.

1. Agregue el código siguiente al archivo de Python. Reemplace *YourIoTHubConnectionString* por la cadena de conexión que copió en [Obtención de la cadena de conexión de IoT Hub](#get-the-iot-hub-connection-string).

    ```python
    import sys
    from azure.iot.hub import IoTHubRegistryManager
    from azure.iot.hub.models import Twin, TwinProperties

    CONNECTION_STRING = "YourIoTHubConnectionString"
    DEVICE_ID = "myFirstDevice"
    MODULE_ID = "myFirstModule"

    try:
        # RegistryManager
        iothub_registry_manager = IoTHubRegistryManager(CONNECTION_STRING)

        module_twin = iothub_registry_manager.get_module_twin(DEVICE_ID, MODULE_ID)
        print ( "" )
        print ( "Module twin properties before update    :" )
        print ( "{0}".format(module_twin.properties) )

        # Update twin
        twin_patch = Twin()
        twin_patch.properties = TwinProperties(desired={"telemetryInterval": 122})
        updated_module_twin = iothub_registry_manager.update_module_twin(
            DEVICE_ID, MODULE_ID, twin_patch, module_twin.etag
        )
        print ( "" )
        print ( "Module twin properties after update     :" )
        print ( "{0}".format(updated_module_twin.properties) )

    except Exception as ex:
        print ( "Unexpected error {0}".format(ex) )
    except KeyboardInterrupt:
        print ( "IoTHubRegistryManager sample stopped" )
    ```

## <a name="get-updates-on-the-device-side"></a>Obtener actualizaciones en el lado de dispositivo

En esta sección, creará una aplicación de Python para obtener la actualización de las propiedades deseadas del módulo gemelo en el dispositivo.

1. Obtenga la cadena de conexión del módulo. En [Azure Portal](https://portal.azure.com/), vaya a IoT Hub y seleccione **Dispositivos IoT** en el panel de la izquierda. Seleccione **myFirstDevice** en la lista de dispositivos y ábralo. En **Identidades de módulo**, seleccione **myFirstModule**. Copie la cadena de conexión del módulo, La necesitará en un paso siguiente.

   ![Detalles del módulo en Azure Portal](./media/iot-hub-python-python-module-twin-getstarted/module-detail.png)

1. En el símbolo del sistema, ejecute el siguiente comando para instalar el paquete **azure-iot-device**:

    ```cmd/sh
    pip install azure-iot-device
    ```

1. Con un editor de texto, cree un archivo denominado **ReceiveModuleTwinDesiredPropertiesPatch.py** en el directorio de trabajo.

1. Agregue el código siguiente al archivo de Python. Reemplace *YourModuleConnectionString* por la cadena de conexión de módulo que copió en el paso 1.

    ```python
    import time
    import threading
    from azure.iot.device import IoTHubModuleClient

    CONNECTION_STRING = "YourModuleConnectionString"


    def twin_update_listener(client):
        while True:
            patch = client.receive_twin_desired_properties_patch()  # blocking call
            print("")
            print("Twin desired properties patch received:")
            print(patch)

    def iothub_client_sample_run():
        try:
            module_client = IoTHubModuleClient.create_from_connection_string(CONNECTION_STRING)

            twin_update_listener_thread = threading.Thread(target=twin_update_listener, args=(module_client,))
            twin_update_listener_thread.daemon = True
            twin_update_listener_thread.start()

            while True:
                time.sleep(1000000)

        except KeyboardInterrupt:
            print("IoTHubModuleClient sample stopped")


    if __name__ == '__main__':
        print ( "Starting the IoT Hub Python sample..." )
        print ( "IoTHubModuleClient waiting for commands, press Ctrl-C to exit" )

        iothub_client_sample_run()
    ```

## <a name="run-the-apps"></a>Ejecución de las aplicaciones

En esta sección, ejecuta la aplicación de dispositivo **ReceiveModuleTwinDesiredPropertiesPatch** y, luego, ejecuta la aplicación de servicio **UpdateModuleTwinDesiredProperties** para actualizar las propiedades deseadas del módulo.

1. Abra un símbolo del sistema y ejecute la aplicación de dispositivo:

    ```cmd/sh
    python ReceiveModuleTwinDesiredPropertiesPatch.py
    ```

   ![Salida inicial de la aplicación de dispositivo](./media/iot-hub-python-python-module-twin-getstarted/device-1.png)

1. Abra otro símbolo del sistema y ejecute la aplicación de dispositivo:

    ```cmd/sh
    python UpdateModuleTwinDesiredProperties.py
    ```

    Observe que la propiedad **TelemetryInterval** deseada aparece en el módulo gemelo actualizado en la salida de la aplicación de servicio:

   ![Salida de la aplicación de servicio](./media/iot-hub-python-python-module-twin-getstarted/service.png)

    La misma propiedad aparece en la revisión de propiedades deseada recibida en la salida de la aplicación de dispositivo:

   ![La salida de la aplicación de dispositivo muestra la revisión de propiedades deseadas](./media/iot-hub-python-python-module-twin-getstarted/device-2.png)

## <a name="next-steps"></a>Pasos siguientes

Para continuar la introducción a IoT Hub y explorar otros escenarios de IoT, consulte:

* [Introducción a la administración de dispositivos](iot-hub-node-node-device-management-get-started.md)

* [Introducción a IoT Edge](../iot-edge/tutorial-simulate-device-linux.md)
