---
title: Conexión del código del dispositivo del componente de Python de ejemplo de la versión preliminar de IoT Plug and Play a IoT Hub | Microsoft Docs
description: Compile y ejecute el código de dispositivo de Python de ejemplo de la versión preliminar de IoT Plug and Play que usa varios componentes y se conecta a un centro de IoT. Use la herramienta Azure IoT Explorer para ver la información enviada por el dispositivo al centro.
author: ericmitt
ms.author: ericmitt
ms.date: 7/14/2020
ms.topic: tutorial
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: 571f0e0ceff0adfbf1814abc627fcab6b23acbe1
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87905863"
---
# <a name="tutorial-connect-a-sample-iot-plug-and-play-preview-multiple-component-device-application-to-iot-hub-python"></a>Tutorial: Conexión de una aplicación de dispositivo de varios componentes de ejemplo de la versión preliminar de IoT Plug and Play a IoT Hub (Python)

[!INCLUDE [iot-pnp-tutorials-device-selector.md](../../includes/iot-pnp-tutorials-device-selector.md)]

En este tutorial se muestra cómo compilar una aplicación de dispositivo IoT Plug and Play de ejemplo con componentes e interfaz raíz, cómo conectarla a un centro de IoT y cómo usar la herramienta Azure IoT Explorer para ver la información que envía al centro. La aplicación de ejemplo está escrita en Python y se incluye en el SDK de dispositivo IoT de Azure para Python. Un generador de soluciones puede usar la herramienta Azure IoT Explorer para comprender las funcionalidades de cualquier dispositivo IoT Plug and Play sin necesidad de ver nada de código del dispositivo.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, necesita Python 3.7 en la máquina de desarrollo. Puede descargar la última versión recomendada para varias plataformas desde [python.org](https://www.python.org/). Puede comprobar la versión de Python con el siguiente comando:  

```cmd/sh
python --version
```

Puede descargar la última versión recomendada para varias plataformas desde [python.org](https://www.python.org/).

### <a name="azure-iot-explorer"></a>Azure IoT Explorer

Para interactuar con el dispositivo de ejemplo en la segunda parte de este tutorial, use la herramienta **Azure IoT Explorer**. [Descargue e instale la versión de Azure IoT Explorer más reciente](./howto-use-iot-explorer.md) para su sistema operativo.

[!INCLUDE [iot-pnp-prepare-iot-hub.md](../../includes/iot-pnp-prepare-iot-hub.md)]

Ejecute el siguiente comando para obtener la _cadena de conexión del centro de IoT_ para su centro. Tome nota de esta cadena de conexión, ya que la usará más adelante en este tutorial:

```azurecli-interactive
az iot hub show-connection-string --hub-name <YourIoTHubName> --output table
```

> [!TIP]
> También puede usar la herramienta Azure IoT Explorer para buscar la cadena de conexión del centro de IoT.

Ejecute el siguiente comando para obtener la _cadena de conexión del dispositivo_ que ha agregado al centro. Tome nota de esta cadena de conexión, ya que la usará más adelante en este tutorial:

```azurecli-interactive
az iot hub device-identity show-connection-string --hub-name <YourIoTHubName> --device-id <YourDeviceID> --output table
```

[!INCLUDE [iot-pnp-download-models.md](../../includes/iot-pnp-download-models.md)]

## <a name="set-up-your-environment"></a>Configurar el entorno

Este paquete se publica como PIP para la actualización de la versión preliminar pública. La versión del paquete debe ser la más reciente o `2.1.4`.

En el entorno de Python local, instale el archivo de la siguiente manera:

```cmd/sh
pip install azure-iot-device
```

Clone el repositorio IoT del SDK de Python y consulte el **pnp-preview-refresh**:

```cmd/sh
git clone https://github.com/Azure/azure-iot-sdk-python
```

## <a name="review-the-code"></a>Revisión del código

En este ejemplo se implementa un dispositivo controlador de temperatura IoT Plug and Play. El modelo que implementa este ejemplo usa [varios componentes](concepts-components.md). El [archivo de modelo del lenguaje de definición de gemelos digitales (DTDL) del dispositivo de temperatura](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/TemperatureController.json) define los datos de telemetría, las propiedades y los comandos que implementa el dispositivo.

La carpeta *azure-iot-sdk-python\azure-iot-device\samples\pnp* contiene el código de ejemplo para el dispositivo IoT Plug and Play. Los archivos del ejemplo de controlador de temperatura son:

- pnp_temp_controller_with_thermostats.py
- pnp_helper_preview_refresh.py

El controlador de temperatura tiene varios componentes y una interfaz raíz, basados en el modelo de DTDL del controlador de temperatura.

Abra el archivo *pnp_temp_controller_with_thermostats.py* en el editor de su elección. El código de este archivo:

1. Importa `pnp_helper_preview_refresh.py` para obtener acceso a los métodos auxiliares.

2. Define dos identificadores del modelo de gemelos digitales (DTMI) que representan de manera exclusiva dos interfaces diferentes, definidas en el modelo DTDL. Los componentes de un controlador de temperatura real deben implementar estas dos interfaces. Estas dos interfaces ya están publicadas en un repositorio central. El usuario debe conocer estos DTMI, que varía en función del escenario de implementación del dispositivo. En el ejemplo actual, estas dos interfaces representan:

  - Un termostato
  - Información del dispositivo desarrollada por Azure

3. Define el `model_id` de DTMI para el dispositivo que se esté implementando. El DTMI está definido por el usuario y debe coincidir con el DTMI en el archivo del modelo DTDL.

4. Define los nombres asignados a los componentes en el archivo DTDL. Hay dos termostatos en el DTDL y un componente de información del dispositivo. También se define una constante denominada `serial_number` en la interfaz raíz. No se puede cambiar `serial_number` de un dispositivo.

5. Define las implementaciones del controlador de comandos. Estas definen lo que hace el dispositivo cuando recibe solicitudes de comandos.

6. Define las funciones para crear una respuesta de comandos. Estas definen lo que responde el dispositivo a las solicitudes de comandos. Puede crear funciones de respuesta de comandos si un comando tiene que devolver una respuesta personalizada al centro de IoT. Si no se proporciona una función de respuesta para un comando, se envía una respuesta genérica. En este ejemplo, solo el comando **getMaxMinReport** tiene una respuesta personalizada.

7. Define una función para enviar datos de telemetría desde este dispositivo. Ambos termostatos y la interfaz raíz envían datos de telemetría. Esta función toma un parámetro de nombre de componente opcional para que pueda identificar qué componente envió los datos de telemetría.

8. Define un agente de escucha para las solicitudes de comandos.

9. Define un agente de escucha para las actualizaciones de propiedades deseadas.

10. Tiene una función `main` que:

    1. Usa el SDK de dispositivo para crear un cliente de dispositivo y conectarse a su centro de IoT. El dispositivo envía el `model_id` para que el centro de IoT pueda identificar el dispositivo como dispositivo IoT Plug and Play.

    1. Usa la función `create_reported_properties` en el archivo auxiliar para crear las propiedades. Pase el nombre del componente, y las propiedades, como pares clave-valor a esta función.

    1. Actualiza las propiedades legibles de sus componentes mediante una llamada a `patch_twin_reported_properties`.

    1. Inicia la escucha para las solicitudes de comandos mediante la función `execute_command_listener`. La función configura un agente de escucha para las solicitudes de comandos desde el servicio. Al configurar el agente de escucha, debe proporcionar `method_name`, `user_command_handler` y un `create_user_response_handler` opcional como parámetros.
        - `method_name` define la solicitud del comando. En este ejemplo, el modelo define los comandos **reboot** y **getMaxMinReport**.
        - La función `user_command_handler` define lo que debe hacer el dispositivo cuando recibe un comando.
        - La función `create_user_response_handler` crea una respuesta para enviar al centro de IoT cuando un comando se ejecuta correctamente. Puede ver esta respuesta en el portal. Si no se proporciona esta función, se envía una respuesta genérica al servicio.

    1. Usa `execute_property_listener` para escuchar las actualizaciones de propiedades.

    1. Empieza a enviar datos de telemetría mediante `send_telemetry`. El código de ejemplo usa un bucle para llamar a tres funciones de envío de datos de telemetría. Se llama a cada una de ellas cada ocho segundos

    1. Deshabilita todos los agentes de escucha y las tareas, y sale del bucle al presionar **Q** o **q**.

Ahora que ha visto el código, cree una variable de entorno denominada **IOTHUB_DEVICE_CONNECTION_STRING** para almacenar la cadena de conexión del dispositivo que anotó anteriormente. Use el comando siguiente para ejecutar el ejemplo:

```cmd/sh
python pnp_temp_controller_with_thermostats.py
```

El dispositivo de ejemplo envía mensajes de telemetría cada pocos segundos al centro de IoT.

Verá la salida siguiente, que indica que el dispositivo está enviando datos de telemetría al centro, y que ya está listo para recibir comandos y actualizaciones de propiedades.

![Mensajes de confirmación de dispositivo](media/tutorial-multiple-components-python/multiple-component.png)

Siga ejecutando el ejemplo a medida que complete los pasos siguientes.

## <a name="use-azure-iot-explorer-to-validate-the-code"></a>Uso de Azure IoT Explorer para validar el código

Una vez iniciado el ejemplo de cliente de dispositivo, compruebe que funciona con la herramienta Azure IoT Explorer.

[!INCLUDE [iot-pnp-iot-explorer.md](../../includes/iot-pnp-iot-explorer.md)]

[!INCLUDE [iot-pnp-clean-resources.md](../../includes/iot-pnp-clean-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha aprendido a conectar un dispositivo IoT Plug and Play con componentes a un centro de IoT. Para obtener más información sobre los modelos de dispositivo IoT Plug and Play, consulte:

> [!div class="nextstepaction"]
> [Guía para desarrolladores de modelado de IoT Plug and Play (versión preliminar)](concepts-developer-guide.md)
