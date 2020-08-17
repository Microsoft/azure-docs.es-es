---
title: Interacción con un dispositivo IoT Plug and Play (versión preliminar) que esté conectado a una solución de Azure IoT (Python) | Microsoft Docs
description: Use Python para conectarse e interactuar con un dispositivo IoT Plug and Play (versión preliminar) que esté conectado a su solución de IoT de Azure.
author: elhorton
ms.author: elhorton
ms.date: 7/13/2020
ms.topic: quickstart
ms.service: iot-pnp
services: iot-pnp
ms.custom: mvc
ms.openlocfilehash: e3f00bb601cce17721c5247941588be1c2de788d
ms.sourcegitcommit: 46f8457ccb224eb000799ec81ed5b3ea93a6f06f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87352732"
---
# <a name="quickstart-interact-with-an-iot-plug-and-play-preview-device-thats-connected-to-your-solution-python"></a>Inicio rápido: Interacción con un dispositivo IoT Plug and Play (versión preliminar) que esté conectado a la solución (Python)

[!INCLUDE [iot-pnp-quickstarts-service-selector.md](../../includes/iot-pnp-quickstarts-service-selector.md)]

IoT Plug and Play (versión preliminar) simplifica IoT, ya que permite interactuar con el modelo de un dispositivo sin tener que conocer la implementación subyacente del dispositivo. En este inicio rápido se muestra cómo usar Python para conectarse y controlar un dispositivo IoT Plug and Play que esté conectado a la solución.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="prerequisites"></a>Requisitos previos

Para completar este inicio rápido, necesita Python 3.7 en la máquina de desarrollo. Puede descargar la última versión recomendada para varias plataformas desde [python.org](https://www.python.org/). Puede comprobar la versión de Python con el siguiente comando:  

```cmd/sh
python --version
```

Ejecute el comando siguiente para instalar el [paquete de versión preliminar del SDK de servicio de Python](https://pypi.org/project/azure-iot-hub/2.2.1rc0/):

```cmd/sh
pip3 install azure-iot-hub==2.2.1rc0
```

[!INCLUDE [iot-pnp-prepare-iot-hub.md](../../includes/iot-pnp-prepare-iot-hub.md)]

Ejecute el siguiente comando para obtener la _cadena de conexión del centro de IoT_ para el centro. Tome nota de esta cadena de conexión, ya que la usará más adelante en este inicio rápido:

```azurecli-interactive
az iot hub show-connection-string --hub-name <YourIoTHubName> --output table
```

Ejecute el siguiente comando para obtener la _cadena de conexión del dispositivo_ que ha agregado al centro. Tome nota de esta cadena de conexión, ya que la usará más adelante en este inicio rápido:

```azurecli-interactive
az iot hub device-identity show-connection-string --hub-name <YourIoTHubName> --device-id <YourDeviceID> --output
```

## <a name="run-the-sample-device"></a>Ejecución del dispositivo de ejemplo

En este inicio rápido, se usa un dispositivo termostato de ejemplo, que se ha escrito en Python, como dispositivo IoT Plug and Play. Para ejecutar el dispositivo de ejemplo:

1. Abra una ventana de terminal en la carpeta que prefiera. Ejecute el siguiente comando para clonar el repositorio de GitHub del [SDK de Python de IoT de Azure](https://github.com/Azure/azure-iot-sdk-python) en esta ubicación:

    ```cmd/sh
    git clone https://github.com/Azure/azure-iot-sdk-python
    ```

1. Esta ventana de terminal se usa ahora como terminal del **dispositivo**. Vaya a la carpeta del repositorio clonado y luego a la carpeta */azure-iot-sdk-python/azure-iot-device/samples/pnp*.

1. Configure la _cadena de conexión del dispositivos_:

    ```cmd/sh
    set IOTHUB_DEVICE_CONNECTION_STRING=<YourDeviceConnectionString>
    ```

1. Ejecute el dispositivo termostato de ejemplo con el siguiente comando:

    ```cmd/sh
    python pnp_thermostat.py
    ```

1. Verá mensajes que señalan que el dispositivo ha enviado cierta información y ha indicado que está en línea. Estos mensajes indican que el dispositivo ha comenzado a enviar datos de telemetría al centro y que ya está listo para recibir comandos y actualizaciones de propiedades. No cierre este terminal, ya que lo necesita para confirmar que el ejemplo del servicio funciona.

## <a name="run-the-sample-solution"></a>Ejecución de la solución de ejemplo

En este inicio rápido, se usa una solución de IoT de ejemplo en Python para interactuar con el dispositivo de ejemplo que acaba de configurar.

1. Abra otra ventana de terminal para utilizarla como terminal de **servicio**. El SDK del servicio se encuentra en versión preliminar, por lo que debe clonar los ejemplos desde una [rama de versión preliminar del SDK de Python](https://github.com/Azure/azure-iot-sdk-python/tree/pnp-preview-refresh):

    ```cmd/sh
    git clone https://github.com/Azure/azure-iot-sdk-python -b pnp-preview-refresh
    ```

1. Vaya a la carpeta de esta rama del repositorio clonado y luego a la carpeta */azure-iot-sdk-python/azure-iot-hub/samples*.

1. Configure las variables de entorno para el identificador del dispositivo y _la cadena de conexión de IoT Hub_:

    ```cmd/sh
    set IOTHUB_CONNECTION_STRING=<YourIOTHubConnectionString>
    set IOTHUB_DEVICE_ID=<Your device ID>
    ```

1. En la carpeta de ejemplos, hay cuatro archivos de ejemplo con un prefijo `pnp`. En estos ejemplos se muestra cómo usar cada una de las API para interactuar con dispositivos IoT Plug and Play:

### <a name="get-digital-twin"></a>Obtención del gemelo digital

Use el siguiente comando en el terminal de **servicio** para ejecutar este ejemplo:

```cmd/sh
python pnp_get_digital_twin_sample.py
```

La salida muestra el gemelo digital del dispositivo e imprime su identificador de modelo:

```cmd/sh
{'$dtId': 'mySimpleThermostat', '$metadata': {'$model': 'dtmi:com:example:Thermostat;1'}}
Model Id: dtmi:com:example:Thermostat;1
```

En el fragmento de código siguiente se muestra el código de ejemplo de *pnp_get_digital_twin_sample.py*:

```python
    # Get digital twin and retrieve the modelId from it
    digital_twin = iothub_digital_twin_manager.get_digital_twin(device_id)
    if digital_twin:
        print(digital_twin)
        print("Model Id: " + digital_twin["$metadata"]["$model"])
    else:
        print("No digital_twin found")
```

### <a name="update-a-digital-twin"></a>Actualización de un gemelo digital

En este ejemplo se muestra cómo usar una *revisión* para actualizar las propiedades a través del gemelo digital del dispositivo. En el siguiente fragmento de código de *pnp_update_digital_twin_sample.py* se muestra cómo construir la revisión:

```python
# If you already have a component thermostat1:
# patch = [{"op": "replace", "path": "/thermostat1/targetTemperature", "value": 42}]
patch = [{"op": "add", "path": "/targetTemperature", "value": 42}]
iothub_digital_twin_manager.update_digital_twin(device_id, patch)
print("Patch has been succesfully applied")
```

Use el siguiente comando en el terminal de **servicio** para ejecutar este ejemplo:

```cmd/sh
python pnp_update_digital_twin_sample.py
```

Puede comprobar que la actualización está aplicada en el terminal del **dispositivo** que muestra la salida siguiente:

```cmd/sh
the data in the desired properties patch was: {'targetTemperature': 42, '$version': 2}
previous values
42
```

El terminal de **servicio** confirma que la revisión se realizó correctamente:

```cmd/sh
Patch has been successfully applied
```

### <a name="invoke-a-command"></a>Invocación de un comando

Para invocar un comando, ejecute el ejemplo *pnp_invoke_command_sample.py*. En este ejemplo se muestra cómo invocar un comando en un dispositivo termostato sencillo. Antes de ejecutar este ejemplo, establezca las variables de entorno `IOTHUB_COMMAND_NAME` y `IOTHUB_COMMAND_PAYLOAD` en el terminal de **servicio**:

```cmd/sh
set IOTHUB_COMMAND_NAME="getMaxMinReport" # this is the relevant command for the thermostat sample
set IOTHUB_COMMAND_PAYLOAD="hello world" # this payload doesn't matter for this sample
```

Use el siguiente comando en el terminal de **servicio** para ejecutar el ejemplo:
  
```cmd/sh
python pnp_invoke_command_sample.py
```

El terminal de **servicio** muestra un mensaje de confirmación del dispositivo:

```cmd/sh
{"tempReport": {"avgTemp": 34.5, "endTime": "13/07/2020 16:03:38", "maxTemp": 49, "minTemp": 11, "startTime": "13/07/2020 16:02:18"}}
```

En el terminal del **dispositivo**, puede ver que el dispositivo recibe el comando:

```cmd/sh
Command request received with payload
hello world
Will return the max, min and average temperature from the specified time hello to the current time
Done generating
{"tempReport": {"avgTemp": 34.2, "endTime": "09/07/2020 09:58:11", "maxTemp": 49, "minTemp": 10, "startTime": "09/07/2020 09:56:51"}}
Sent message
```

[!INCLUDE [iot-pnp-clean-resources.md](../../includes/iot-pnp-clean-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a conectar un dispositivo IoT Plug and Play a una solución de IoT. Para más información acerca de los modelos de dispositivo IoT Plug and Play, consulte:

> [!div class="nextstepaction"]
> [Guía para desarrolladores de modelado de IoT Plug and Play (versión preliminar)](concepts-developer-guide.md)
