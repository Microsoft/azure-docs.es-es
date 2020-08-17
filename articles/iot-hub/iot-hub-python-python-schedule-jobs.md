---
title: Programación de trabajos con Azure IoT Hub (Python) | Microsoft Docs
description: Cómo programar un trabajo de IoT Hub de Azure para invocar un método directo en varios dispositivos. El SDK de IoT de Azure para Python se usa para implementar las aplicaciones de dispositivo simulado y una aplicación de servicio para ejecutar el trabajo.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.devlang: python
ms.topic: conceptual
ms.date: 03/17/2020
ms.author: robinsh
ms.custom: devx-track-python
ms.openlocfilehash: 733e3be21a1a1305b5c7947de1ae54ddce5e0d2f
ms.sourcegitcommit: dea88d5e28bd4bbd55f5303d7d58785fad5a341d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87876689"
---
# <a name="schedule-and-broadcast-jobs-python"></a>Programación y difusión de trabajos (Python)

[!INCLUDE [iot-hub-selector-schedule-jobs](../../includes/iot-hub-selector-schedule-jobs.md)]

IoT Hub de Azure es un servicio completamente administrado que permite a una aplicación back-end crear y realizar un seguimiento de los trabajos que programan y actualizan millones de dispositivos.  Los trabajos pueden utilizarse para las siguientes acciones:

* Actualizar las propiedades deseadas
* Actualizar etiquetas
* Invocar métodos directos

Conceptualmente, un trabajo contiene una de estas acciones y realiza un seguimiento del progreso de ejecución en un conjunto de dispositivos, que define una consulta de dispositivo gemelo.  Por ejemplo, una aplicación back-end puede usar un trabajo para invocar un método de reinicio en 10 000 dispositivos, especificado por una consulta de dispositivos gemelos y programada en el futuro.  Esa aplicación puede después seguir el progreso cuando cada uno de estos dispositivos reciben y ejecutan el método de reinicio.

Más información sobre estas funcionalidades en estos artículos:

* Dispositivo gemelo y propiedades: [Introducción a los dispositivos gemelos](iot-hub-python-twin-getstarted.md) y [Tutorial: uso de las propiedades de dispositivos gemelos](tutorial-device-twins.md)

* Métodos directos: [Guía para desarrolladores de IoT Hub: métodos directos](iot-hub-devguide-direct-methods.md) y [Tutorial: Métodos directos](quickstart-control-device-python.md)

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

En este tutorial se muestra cómo realizar las siguientes acciones:

* Crear una aplicación de dispositivo simulado en Python con un método directo, que habilita **lockDoor**, que la solución de back-end puede llamar.

* Crear una aplicación de consola de Python que llame al método directo **lockDoor** en la aplicación de dispositivo simulado mediante un trabajo y que actualice las propiedades deseadas mediante un trabajo de dispositivo.

Al final de este tutorial, tiene dos aplicaciones de Python:

**simDevice.py**, que se conecta a IoT Hub con la identidad del dispositivo y recibe un método directo **lockDoor**.

**scheduleJobService.py**, que llama a un método directo en la aplicación de dispositivo simulado y actualiza las propiedades deseadas del dispositivo gemelo mediante un trabajo.

> [!NOTE]
> El **SDK de IoT de Azure para Python** no admite directamente la funcionalidad de **trabajos**. En su lugar, este tutorial ofrece una solución alternativa con temporizadores y subprocesos asincrónicos. Para obtener más actualizaciones, consulte la lista de características del **SDK del cliente de servicio** en la página [Azure IoT SDK for Python](https://github.com/Azure/azure-iot-sdk-python) (SDK de IoT de Azure para Python).
>

[!INCLUDE [iot-hub-include-python-sdk-note](../../includes/iot-hub-include-python-sdk-note.md)]

## <a name="prerequisites"></a>Prerrequisitos

[!INCLUDE [iot-hub-include-python-v2-installation-notes](../../includes/iot-hub-include-python-v2-installation-notes.md)]

## <a name="create-an-iot-hub"></a>Crear un centro de IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-new-device-in-the-iot-hub"></a>Registro de un nuevo dispositivo en el centro de IoT

[!INCLUDE [iot-hub-include-create-device](../../includes/iot-hub-include-create-device.md)]

## <a name="create-a-simulated-device-app"></a>Creación de una aplicación de dispositivo simulado

En esta sección, creará una aplicación de consola de Python que responda a un método directo al que la nube llama, que desencadena un método **lockDoor** simulado.

1. En el símbolo del sistema, ejecute el siguiente comando para instalar el paquete **azure-iot-device**:

    ```cmd/sh
    pip install azure-iot-device
    ```

2. Con un editor de texto, cree un nuevo archivo **simDevice.py** en el directorio de trabajo.

3. Agregue las siguientes instrucciones y variables `import` al principio del archivo **simDevice.py**. Reemplace `deviceConnectionString` con la cadena de conexión del dispositivo que creó anteriormente:

    ```python
    import threading
    import time
    from azure.iot.device import IoTHubDeviceClient, MethodResponse

    CONNECTION_STRING = "{deviceConnectionString}"
    ```

4. Agregue la siguiente devolución de llamada de función para controlar el método **lockDoor**:

    ```python
    def lockdoor_listener(client):
        while True:
            # Receive the direct method request
            method_request = client.receive_method_request("lockDoor")  # blocking call
            print( "Locking Door!" )

            resp_status = 200
            resp_payload = {"Response": "lockDoor called successfully"}
            method_response = MethodResponse(method_request.request_id, resp_status, resp_payload)
            client.send_method_response(method_response)
    ```

5. Agregue otra devolución de llamada de función para controlar las actualizaciones de los dispositivos gemelos:

    ```python
    def twin_update_listener(client):
        while True:
            patch = client.receive_twin_desired_properties_patch()  # blocking call
            print ("")
            print ("Twin desired properties patch received:")
            print (patch)
    ```

6. Agregue el código siguiente para registrar el controlador del método **lockDoor**. Incluya también la rutina `main`:

    ```python
    def iothub_jobs_sample_run():
        try:
            client = IoTHubDeviceClient.create_from_connection_string(CONNECTION_STRING)

            print( "Beginning to listen for 'lockDoor' direct method invocations...")
            lockdoor_listener_thread = threading.Thread(target=lockdoor_listener, args=(client,))
            lockdoor_listener_thread.daemon = True
            lockdoor_listener_thread.start()

            # Begin listening for updates to the Twin desired properties
            print ( "Beginning to listen for updates to Twin desired properties...")
            twin_update_listener_thread = threading.Thread(target=twin_update_listener, args=(client,))
            twin_update_listener_thread.daemon = True
            twin_update_listener_thread.start()
            
            while True:
                time.sleep(1000)

        except KeyboardInterrupt:
            print ( "IoTHubDeviceClient sample stopped" )

    if __name__ == '__main__':
        print ( "Starting the IoT Hub Python jobs sample..." )
        print ( "IoTHubDeviceClient waiting for commands, press Ctrl-C to exit" )

        iothub_jobs_sample_run()
    ```

7. Guarde y cierre el archivo **simDevice.py**.

> [!NOTE]
> Por simplificar, este tutorial no implementa ninguna directiva de reintentos. En el código de producción, deberá implementar directivas de reintentos (por ejemplo, retroceso exponencial), tal y como se sugiere en el artículo [Control de errores transitorios](/azure/architecture/best-practices/transient-faults).
>

## <a name="get-the-iot-hub-connection-string"></a>Obtención de la cadena de conexión de IoT Hub

En este artículo, creará un servicio back-end que invoca un método directo en un dispositivo y actualiza el dispositivo gemelo. El servicio necesita el permiso **Conexión del servicio** para realizar una llamada al método directo de un dispositivo. El servicio también necesita los permisos **Lectura del registro** y **Escribir en el registro** para leer y escribir en el registro de identidad. No hay ninguna directiva de acceso compartido predeterminada que contenga solo estos permisos, por lo que tendrá que crearla.

Para crear una directiva de acceso compartido que conceda los permisos **Conexión del servicio**, **Lectura del registro** y **Escribir en el registro** y obtener una cadena de conexión para esta directiva, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com), abra IoT Hub. La manera más fácil de acceder a IoT Hub es seleccionar **Grupos de recursos**, seleccionar el grupo de recursos donde se encuentra IoT Hub y después seleccionarlo en la lista de recursos.

2. En el panel de la izquierda de IoT Hub, seleccione **Directivas de acceso compartido**.

3. En el menú superior situado encima de la lista de directivas, seleccione **Agregar**.

4. En el panel **Agregar una directiva de acceso compartida**, escriba un nombre descriptivo para la directiva, por ejemplo *serviceAndRegistryReadWrite*. En **Permisos**, seleccione **Conexión del servicio** y **Escribir en el registro** (**Lectura del registro** se selecciona automáticamente al seleccionar **Escribir en el registro**). Seleccione **Crear**.

    ![Cómo agregar una nueva directiva de acceso compartido](./media/iot-hub-python-python-schedule-jobs/add-policy.png)

5. En el panel **Directivas de acceso compartido**, seleccione la nueva directiva en la lista de directivas.

6. En **Claves de acceso compartido**, seleccione el icono de **Cadena de conexión: clave principal** y guarde el valor.

    ![Recuperación de la cadena de conexión](./media/iot-hub-python-python-schedule-jobs/get-connection-string.png)

Para obtener más información sobre las directivas de acceso compartido y los permisos de IoT Hub, consulte [Permisos y control del acceso](./iot-hub-devguide-security.md#access-control-and-permissions).

## <a name="schedule-jobs-for-calling-a-direct-method-and-updating-a-device-twins-properties"></a>Programación de trabajos para llamar un método directo y actualización de las propiedades de un dispositivo gemelo

En esta sección, creará una aplicación de consola de Python que inicia un **lockDoor** remoto en un dispositivo con un método directo y actualiza las propiedades seleccionadas del dispositivo gemelo.

1. En el símbolo del sistema, ejecute el siguiente comando para instalar el paquete **azure-iot-hub**:

    ```cmd/sh
    pip install azure-iot-hub
    ```

2. Con un editor de texto, cree un nuevo archivo **scheduleJobService.py** en el directorio de trabajo.

3. Agregue las siguientes instrucciones y variables `import` al principio del archivo **scheduleJobService.py**. Reemplace el marcador de posición `{IoTHubConnectionString}` por la cadena de conexión de IoT Hub que copió anteriormente en [Obtención de la cadena de conexión de IoT Hub](#get-the-iot-hub-connection-string). Reemplace el marcador de posición `{deviceId}` por el id. de dispositivo que registró en [Registro de un nuevo dispositivo en el centro de IoT](#register-a-new-device-in-the-iot-hub):

    ```python
    import sys
    import time
    import threading
    import uuid

    from azure.iot.hub import IoTHubRegistryManager
    from azure.iot.hub.models import Twin, TwinProperties, CloudToDeviceMethod, CloudToDeviceMethodResult, QuerySpecification, QueryResult

    CONNECTION_STRING = "{IoTHubConnectionString}"
    DEVICE_ID = "{deviceId}"

    METHOD_NAME = "lockDoor"
    METHOD_PAYLOAD = "{\"lockTime\":\"10m\"}"
    UPDATE_PATCH = {"building":43,"floor":3}
    TIMEOUT = 60
    WAIT_COUNT = 5
    ```

4. Agregue la siguiente función que se usa para consultar los dispositivos:

    ```python
    def query_condition(iothub_registry_manager, device_id):

        query_spec = QuerySpecification(query="SELECT * FROM devices WHERE deviceId = '{}'".format(device_id))
        query_result = iothub_registry_manager.query_iot_hub(query_spec, None, 1)

        return len(query_result.items)
    ```

5. Agregue los métodos siguientes para ejecutar los trabajos que llaman al método directo y al dispositivo gemelo:

    ```python
    def device_method_job(job_id, device_id, wait_time, execution_time):
        print ( "" )
        print ( "Scheduling job: " + str(job_id) )
        time.sleep(wait_time)

        iothub_registry_manager = IoTHubRegistryManager(CONNECTION_STRING)


        if query_condition(iothub_registry_manager, device_id):
            deviceMethod = CloudToDeviceMethod(method_name=METHOD_NAME, payload=METHOD_PAYLOAD)

            response = iothub_registry_manager.invoke_device_method(DEVICE_ID, deviceMethod)

            print ( "" )
            print ( "Direct method " + METHOD_NAME + " called." )

    def device_twin_job(job_id, device_id, wait_time, execution_time):
        print ( "" )
        print ( "Scheduling job " + str(job_id) )
        time.sleep(wait_time)

        iothub_registry_manager = IoTHubRegistryManager(CONNECTION_STRING)

        if query_condition(iothub_registry_manager, device_id):

            twin = iothub_registry_manager.get_twin(DEVICE_ID)
            twin_patch = Twin(properties= TwinProperties(desired=UPDATE_PATCH))
            twin = iothub_registry_manager.update_twin(DEVICE_ID, twin_patch, twin.etag)

            print ( "" )
            print ( "Device twin updated." )
    ```

6. Agregue el código siguiente para programar los trabajos y actualizar sus estados. Incluya también la rutina `main`:

    ```python
    def iothub_jobs_sample_run():
        try:
            method_thr_id = uuid.uuid4()
            method_thr = threading.Thread(target=device_method_job, args=(method_thr_id, DEVICE_ID, 20, TIMEOUT), kwargs={})
            method_thr.start()

            print ( "" )
            print ( "Direct method called with Job Id: " + str(method_thr_id) )

            twin_thr_id = uuid.uuid4()
            twin_thr = threading.Thread(target=device_twin_job, args=(twin_thr_id, DEVICE_ID, 10, TIMEOUT), kwargs={})
            twin_thr.start()

            print ( "" )
            print ( "Device twin called with Job Id: " + str(twin_thr_id) )

            while True:
                print ( "" )

                if method_thr.is_alive():
                    print ( "...job " + str(method_thr_id) + " still running." )
                else:
                    print ( "...job " + str(method_thr_id) + " complete." )

                if twin_thr.is_alive():
                    print ( "...job " + str(twin_thr_id) + " still running." )
                else:
                    print ( "...job " + str(twin_thr_id) + " complete." )

                print ( "Job status posted, press Ctrl-C to exit" )

                status_counter = 0
                while status_counter <= WAIT_COUNT:
                    time.sleep(1)
                    status_counter += 1

        except Exception as ex:
            print ( "" )
            print ( "Unexpected error {0}" % ex )
            return
        except KeyboardInterrupt:
            print ( "" )
            print ( "IoTHubService sample stopped" )

    if __name__ == '__main__':
        print ( "Starting the IoT Hub jobs Python sample..." )
        print ( "    Connection string = {0}".format(CONNECTION_STRING) )
        print ( "    Device ID         = {0}".format(DEVICE_ID) )

        iothub_jobs_sample_run()
    ```

7. Guarde y cierre el archivo **scheduleJobService.py**.

## <a name="run-the-applications"></a>Ejecución de las aplicaciones

Ahora está preparado para ejecutar las aplicaciones.

1. En el símbolo del sistema, en el directorio de trabajo, ejecute el siguiente comando para iniciar la escucha del método directo de reinicio:

    ```cmd/sh
    python simDevice.py
    ```

2. En otro símbolo del sistema, en el directorio de trabajo, ejecute el siguiente comando para desencadenar los trabajos a fin de bloquear la puerta y actualizar el dispositivo gemelo:
  
    ```cmd/sh
    python scheduleJobService.py
    ```

3. Verá las respuestas del dispositivo al método directo y la actualización de los dispositivos gemelos en la consola.

    ![Ejemplo de trabajo de IoT Hub 1: salida del dispositivo](./media/iot-hub-python-python-schedule-jobs/sample1-deviceoutput.png)

    ![Ejemplo de trabajo de IoT Hub 2: salida del dispositivo](./media/iot-hub-python-python-schedule-jobs/sample2-deviceoutput.png)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha utilizado un trabajo para programar un método directo para un dispositivo y la actualización de las propiedades del dispositivo gemelo.

Para continuar con la introducción de IoT Hub y los patrones de administración de dispositivos como remotos a través de la actualización de firmware de aire, vea [Realización de una actualización de firmware](tutorial-firmware-update.md).
