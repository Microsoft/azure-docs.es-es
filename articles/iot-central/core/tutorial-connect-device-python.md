---
title: 'Tutorial: Conexión de una aplicación cliente Python genérica a Azure IoT Central | Microsoft Docs'
description: En este tutorial se muestra cómo los desarrolladores de dispositivos pueden conectar un dispositivo que ejecuta una aplicación cliente Python a la aplicación de Azure IoT Central. Para crear una plantilla de dispositivo, importe un modelo de funcionalidad de dispositivo y agregue vistas que permitan interactuar con un dispositivo conectado
author: dominicbetts
ms.author: dobett
ms.date: 07/07/2020
ms.topic: tutorial
ms.service: iot-central
services: iot-central
ms.custom:
- devx-track-python
- device-developer
ms.openlocfilehash: 1be7087e99ca2e4dc605a8d1c6b9821567aecfef
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90968135"
---
# <a name="tutorial-create-and-connect-a-client-application-to-your-azure-iot-central-application-python"></a>Tutorial: Creación y conexión de un aplicación cliente a una aplicación de Azure IoT Central (Python)

[!INCLUDE [iot-central-selector-tutorial-connect](../../../includes/iot-central-selector-tutorial-connect.md)]

*Este artículo se aplica a los generadores de soluciones y a los desarrolladores de dispositivos.*

En este tutorial se muestra cómo los desarrolladores de dispositivos pueden conectar una aplicación cliente Python a la aplicación de Azure IoT Central. La aplicación de Python simula el comportamiento de un dispositivo sensor ambiental. Para crear una _plantilla de dispositivo_ en IoT Central, se usa un _modelo de funcionalidad del dispositivo_ de muestra. Agregue vistas a la plantilla de dispositivo para permitir que un operador interactúe con un dispositivo.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Importar un modelo de funcionalidad de dispositivo para crear una plantilla de dispositivo.
> * Agregar las vistas predeterminada y personalizada a una plantilla de dispositivo.
> * Publicar una plantilla de dispositivo y agregar un dispositivo real a la aplicación de IoT Central.
> * Crear y ejecutar el código del dispositivo Python y ver si se conecta a la aplicación de IoT Central.
> * Ver los datos de telemetría simulados que envía el dispositivo.
> * Usar una vista para administrar las propiedades del dispositivo.
> * Llamar a comandos sincrónicos y asincrónicos para controlar el dispositivo.

## <a name="prerequisites"></a>Requisitos previos

Necesitará lo siguiente para completar los pasos de este artículo:

* Una aplicación de Azure IoT Central creada a partir de la plantilla **Custom application** (Aplicación personalizada). Para más información, consulte la [guía de inicio rápido para crear una aplicación](quick-deploy-iot-central.md). La aplicación se debe haber creado el 14/07/2020 o después.
* Una máquina de desarrollo que tenga instalada la versión 3.7 de [Python](https://www.python.org/) o una posterior. Puede ejecutar `python3 --version` en la línea de comandos para comprobar la versión. Python está disponible para una amplia variedad de sistemas operativos. En las instrucciones de este tutorial se da por hecho que se ejecuta el comando **python3** en el símbolo del sistema de Windows.

[!INCLUDE [iot-central-add-environmental-sensor](../../../includes/iot-central-add-environmental-sensor.md)]

### <a name="create-a-python-application"></a>Creación de una aplicación de Python

En los pasos siguientes se muestra cómo crear una aplicación cliente Python que se conecta al dispositivo real que se agregó a la aplicación. Esta aplicación de Python simula el comportamiento de un dispositivo real.

1. Vaya a la carpeta `environmental-sensor` que ha creado anteriormente en el entorno de la línea de comandos.

1. Ejecute los siguientes comandos para instalar las bibliotecas necesarias:

    ```cmd
    pip install azure-iot-device
    ```

1. Cree un archivo llamado **environmental_sensor.py** en la carpeta `environmental-sensor`.

1. Agregue las siguientes instrucciones `import` al principio del archivo **environmental_sensor.py**:

    ```python
    import asyncio
    import os
    import json
    import datetime
    import random

    from azure.iot.device.aio import ProvisioningDeviceClient
    from azure.iot.device.aio import IoTHubDeviceClient
    from azure.iot.device import MethodResponse
    from azure.iot.device import Message
    ```

1. Agregue las siguientes declaraciones de variable y la función `main` asincrónica al archivo:

    ```python
    async def main():
        # In a production environment, don't store
        # connection information in the code.
        provisioning_host = 'global.azure-devices-provisioning.net'
        id_scope = '{your Scope ID}'
        registration_id = '{your Device ID}'
        symmetric_key = '{your Primary Key}'
  
        delay = 2
  
        # All the remaining code is nested within this main function

    if __name__ == '__main__':
        asyncio.run(main())
    ```

    Actualice los marcadores de posición `{your Scope ID}`, `{your Device ID}` y `{your Primary Key}` con los valores que anotó anteriormente. En una aplicación real, no codifique esta información en la aplicación.

    Todas las definiciones de funciones y el código siguientes están anidados dentro de la función `main`.

1. Agregue las dos funciones siguientes dentro de la función `main` para registrar el dispositivo y conectarlo a la aplicación de IoT Central. El registro usa el servicio Azure Device Provisioning:

    ```python
        async def register_device():
            provisioning_device_client = ProvisioningDeviceClient.create_from_symmetric_key(
                provisioning_host=provisioning_host,
                registration_id=registration_id,
                id_scope=id_scope,
                symmetric_key=symmetric_key,
            )

            registration_result = await provisioning_device_client.register()

            print(f'Registration result: {registration_result.status}')

            return registration_result
  
        async def connect_device():
            device_client = None
            try:
                registration_result = await register_device()
                if registration_result.status == 'assigned':
                    device_client = IoTHubDeviceClient.create_from_symmetric_key(
                        symmetric_key=symmetric_key,
                        hostname=registration_result.registration_state.assigned_hub,
                        device_id=registration_result.registration_state.device_id
                    )
                    # Connect the client.
                    await device_client.connect()
                    print('Device connected successfully')
            finally:
                return device_client
    ```

1. Agregue la siguiente función a la función `main` para enviar datos de telemetría a la aplicación de IoT Central:

    ```python
        async def send_telemetry():
            print(f'Sending telemetry from the provisioned device every {delay} seconds')
            while True:
                temp = random.randrange(1, 75)
                humid = random.randrange(30, 99)
                payload = json.dumps(
                    {
                        'temp': temp,
                        'humid': humid
                    })
                msg = Message(payload)
                await device_client.send_message(msg, )
                print(f'Sent message: {msg}')
                await asyncio.sleep(delay)
    ```

    Los nombres de los elementos de telemetría (`temp` y `humid`) deben coincidir con los nombres que se usan en la plantilla de dispositivo.

1. Agregue las siguientes funciones dentro de la función `main` para controlar los comandos llamados desde la aplicación de IoT Central:

    ```python
        async def blink_command(request):
            print('Received synchronous call to blink')
            response = MethodResponse.create_from_method_request(
                request,
                status = 200,
                payload = {'description': f'Blinking LED every {request.payload} seconds'}
            )
            await device_client.send_method_response(response)  # send response
            print(f'Blinking LED every {request.payload} seconds')

        async def diagnostics_command(request):
            print('Starting asynchronous diagnostics run...')
            response = MethodResponse.create_from_method_request(
                request,
                status = 202
            )
            await device_client.send_method_response(response)  # send response
            print('Generating diagnostics...')
            await asyncio.sleep(2)
            print('Generating diagnostics...')
            await asyncio.sleep(2)
            print('Generating diagnostics...')
            await asyncio.sleep(2)
            print('Sending property update to confirm command completion')
            await device_client.patch_twin_reported_properties(
                {
                    'rundiagnostics':
                    {
                        'value': f'Diagnostics run complete at {datetime.datetime.today()}.'
                    }
                })

        async def turnon_command(request):
            print('Turning on the LED')
            response = MethodResponse.create_from_method_request(
                request, status = 200
            )
            await device_client.send_method_response(response)  # send response

        async def turnoff_command(request):
            print('Turning off the LED')
            response = MethodResponse.create_from_method_request(
                request, status = 200
            )
            await device_client.send_method_response(response)  # send response

        commands = {
            'blink': blink_command,
            'rundiagnostics': diagnostics_command,
            'turnon': turnon_command,
            'turnoff': turnoff_command,
        }

        # Define behavior for handling commands
        async def command_listener():
            while True:
                method_request = await device_client.receive_method_request()  # Wait for commands
                await commands[method_request.name](method_request)
    ```

    Los nombres de los comandos (`blink`, `turnon`, `turnoff` y `rundiagnostics`) deben coincidir con los nombres que se usan en la plantilla de dispositivo.

    Actualmente, IoT Central no usa el esquema de respuestas definido en el modelo de funcionalidad del dispositivo. Para un comando sincrónico, la carga útil de respuesta puede ser cualquier JSON válido. Para un comando asincrónico, el dispositivo debe devolver una respuesta 202 inmediatamente, seguida de una actualización de propiedad notificada cuando el trabajo ha finalizado. El formato de la actualización de propiedad notificada es:

    ```json
    {
      [command name] : {
        value: 'response message'
      }
    }
    ```

    Un operador puede ver la carga útil de respuesta en el historial de comandos.

1. Agregue las siguientes funciones dentro de la función `main` para controlar las actualizaciones de propiedades enviadas desde la aplicación de IoT Central. El mensaje que el dispositivo envía en respuesta a la [actualización de la propiedad grabable](concepts-telemetry-properties-commands.md#writeable-property-types) debe incluir los campos `av` y `ac`. El campo `ad` es opcional:

    ```python
        async def name_setting(value, version):
            await asyncio.sleep(1)
            print(f'Setting name value {value} - {version}')
            await device_client.patch_twin_reported_properties(
                {
                    'name' :
                    {
                        'value': value,
                        'ad': 'completed',
                        'ac': 200,
                        'av': version
                    }
                })
  
        async def brightness_setting(value, version):
            await asyncio.sleep(5)
            print(f'Setting brightness value {value} - {version}')
            await device_client.patch_twin_reported_properties(
                {
                    'brightness' :
                    {
                        'value': value,
                        'ad': 'completed',
                        'ac': 200,
                        'av': version
                    }
                })
  
        settings = {
            'name': name_setting,
            'brightness': brightness_setting
        }

        # define behavior for receiving a twin patch
        async def twin_patch_listener():
            while True:
                patch = await device_client.receive_twin_desired_properties_patch() # blocking
                to_update = patch.keys() & settings.keys()
                await asyncio.gather(
                    *[settings[setting](patch[setting], patch['$version']) for setting in to_update]
                )
    ```

    Cuando el operador establece una propiedad que se puede escribir en la aplicación de IoT Central, la aplicación usa una propiedad deseada del dispositivo gemelo para enviar el valor al dispositivo. A continuación, el dispositivo responde con una propiedad notificada del dispositivo gemelo. Cuando IoT Central recibe el valor de la propiedad notificada, actualiza la vista de la propiedad con el estado **sincronizado**.

    Los nombres de las propiedades (`name` y `brightness`) deben coincidir con los nombres que se usan en la plantilla de dispositivo.

1. Agregue las siguientes funciones dentro de la función `main` para controlar la aplicación:

    ```python
        # Define behavior for halting the application
        def stdin_listener():
            while True:
                selection = input('Press Q to quit\n')
                if selection == 'Q' or selection == 'q':
                    print('Quitting...')
                    break
  
        device_client = await connect_device()
  
        if device_client is not None and device_client.connected:
            print('Send reported properties on startup')
            await device_client.patch_twin_reported_properties(
                {
                    'state': 'true',
                    'processorArchitecture': 'ARM',
                    'swVersion': '1.0.0'
                })
            tasks = asyncio.gather(
                send_telemetry(),
                command_listener(),
                twin_patch_listener(),
            )

            # Run the stdin listener in the event loop
            loop = asyncio.get_running_loop()
            user_finished = loop.run_in_executor(None, stdin_listener)

            # Wait for user to indicate they are done listening for method calls
            await user_finished

            # Cancel tasks
            tasks.add_done_callback(lambda r: r.exception())
            tasks.cancel()
            await device_client.disconnect()
  
        else:
            print('Device could not connect')
    ```

1. Guarde el archivo **environmental_sensor.py**.

## <a name="run-your-python-application"></a>Ejecución de la aplicación de Python

Para iniciar la aplicación cliente del dispositivo, ejecute el comando siguiente en el entorno de la línea de comandos:

```cmd
python3 environmental_sensor.py
```

Verá que el dispositivo se conecta a la aplicación de Azure IoT Central y comienza a enviar telemetría:

![Ejecución de la aplicación cliente](media/tutorial-connect-device-python/run-application.png)

[!INCLUDE [iot-central-monitor-environmental-sensor](../../../includes/iot-central-monitor-environmental-sensor.md)]

Puede ver cómo responde el dispositivo a los comandos y las actualizaciones de propiedades:

![Visualización de la aplicación cliente](media/tutorial-connect-device-python/run-application-2.png)

## <a name="view-raw-data"></a>Visualización de datos sin procesar

[!INCLUDE [iot-central-monitor-environmental-sensor-raw-data](../../../includes/iot-central-monitor-environmental-sensor-raw-data.md)]

## <a name="next-steps"></a>Pasos siguientes

Si prefiere continuar con el conjunto de tutoriales de IoT Central y obtener más información sobre la creación de una solución de IoT Central, consulte:

> [!div class="nextstepaction"]
> [Creación de una plantilla de dispositivo de puerta de enlace](./tutorial-define-gateway-device-type.md)

Como desarrollador de dispositivos, ahora que ha aprendido los aspectos básicos de cómo crear un dispositivo mediante Python, se recomiendan los siguientes pasos:

* Para más información sobre el rol de las plantillas de dispositivo cuando se está implementando el código del dispositivo, consulte [¿Qué son las plantillas de dispositivo?](./concepts-device-templates.md).
* Lea [Conexión a Azure IoT Central](./concepts-get-connected.md) para más información sobre cómo registrar dispositivos con IoT Central y cómo IoT Central protege las conexiones de dispositivos.