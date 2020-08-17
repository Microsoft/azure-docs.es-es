---
title: Uso de MQTT para crear un cliente de dispositivo IoT Plug and Play (versión preliminar) | Microsoft Docs
description: Uso del protocolo MQTT directamente para crear un cliente de dispositivo IoT Plug and Play (versión preliminar) sin usar los SDK de dispositivo IoT de Azure
author: ericmitt
ms.author: ericmitt
ms.date: 05/13/2020
ms.topic: tutorial
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: 56463b03fe633959585e14271050bcdaacb25663
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87535218"
---
# <a name="use-mqtt-to-develop-an-iot-plug-and-play-preview-device-client"></a>Uso de MQTT para desarrollar un cliente del dispositivo IoT Plug and Play (versión preliminar)

Si es posible, debe usar uno de los SDK de dispositivo IoT de Azure para compilar los clientes de dispositivo IoT Plug and Play. Sin embargo, en escenarios tales como el uso de un dispositivo con restricción de memoria, es posible que tenga que usar una biblioteca de MQTT para comunicarse con el centro de IoT.

En el ejemplo de este tutorial se usa la biblioteca de MQTT [Eclipse Mosquitto](http://mosquitto.org/) y Visual Studio. En los pasos de este tutorial se da por hecho que se usa Windows en el equipo de desarrollo.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial en Windows, instale el siguiente software en el entorno Windows local:

* [Visual Studio (Community, Professional o Enterprise)](https://visualstudio.microsoft.com/downloads/): asegúrese de incluir la carga de trabajo **Desarrollo para el escritorio con C++** al [instalar](https://docs.microsoft.com/cpp/build/vscpp-step-0-installation?view=vs-2019) Visual Studio.
* [Git](https://git-scm.com/download/)
* [CMake](https://cmake.org/download/)
* [Azure IoT Explorer](howto-install-iot-explorer.md)

[!INCLUDE [iot-pnp-prepare-iot-hub.md](../../includes/iot-pnp-prepare-iot-hub.md)]

Ejecute el siguiente comando para obtener la firma de acceso compartido para que el dispositivo se conecte al centro. Anote esta cadena, ya que la usará más adelante en el tutorial:

```azurecli-interactive
az iot hub generate-sas-token -d <YourDeviceID> -n <YourIoTHubName>
az iot hub show-connection-string --hub-name <YourIoTHubName> --output table
```

Use la cadena de conexión del centro de IoT para configurar la herramienta **Azure IoT Explorer**:

1. Inicie la herramienta **Azure IoT Explorer**.
1. En la página **Settings** (Configuración), pegue la cadena de conexión del centro de IoT en **App configurations** (Configuraciones de aplicaciones).
1. Seleccione **Save and Connect** (Guardar y continuar).
1. El dispositivo que agregó anteriormente se encuentra en la lista de dispositivos de la página principal.

## <a name="clone-sample-repo"></a>Clonación del repositorio de ejemplo

Use el siguiente comando para clonar el repositorio de ejemplo en una ubicación adecuada de la máquina local:

```cmd
git clone https://github.com/Azure-Samples/IoTMQTTSample.git
```

## <a name="install-mqtt-library"></a>Instalación de la biblioteca de MQTT

Use la herramienta `vcpkg` para descargar y compilar la biblioteca Eclipse Mosquitto.

Use el siguiente comando para clonar el repositorio **Vcpkg** en una ubicación adecuada de la máquina local:

```cmd
git clone https://github.com/Microsoft/vcpkg.git
```

Use los comandos siguientes para preparar entorno de `vcpkg`. Cuando se ejecuta `vcpkg integrate install`, se necesita un símbolo del sistema con privilegios elevados:

```cmd
cd vcpkg
.\bootstrap-vcpkg.bat
.\vcpkg integrate install
```

Use el siguiente comando para descargar y compilar la biblioteca Eclipse Mosquitto:

```cmd
.\vcpkg install mosquitto:x64-windows
```

## <a name="migrate-the-sample-to-iot-plug-and-play"></a>Migración del ejemplo a IoT Plug and Play

### <a name="review-the-non-iot-plug-and-play-sample-code"></a>Revisión de código de ejemplo que no es de IoT Plug and Play

Actualice el código con los detalles de su centro de IoT y del dispositivo antes de compilarlo y ejecutarlo.

Para ver el código de ejemplo en Visual Studio, abra el archivo de solución *MQTTWin32.sln*, que se encuentra en la carpeta *IoTMQTTSample\src\Windows*.

En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto **TelemetryMQTTWin32** y seleccione **Establecer como proyecto de inicio**.

En el proyecto **TelemetryMQTTWin32**, abra el archivo de origen **MQTT_Mosquitto.cpp**. Actualice las definiciones de la información de conexión con los detalles del dispositivo que anotó anteriormente. Reemplace el marcador de posición de la cadena del token en lo siguiente:

* Identificador `IOTHUBNAME` por el nombre del centro de IoT que creó.
* Identificador `DEVICEID` por el nombre del dispositivo que creó.
* Identificador `PWD` por el valor de la firma de acceso compartido que generó para el dispositivo.

Compruebe que el código funciona correctamente; para ello, inicie Azure IoT Explorer y empiece a escuchar los datos de telemetría.

Ejecute la aplicación (Ctrl+F5) y, después de unos segundos, verá una salida similar a la siguiente:

:::image type="content" source="media/tutorial-use-mqtt/mqtt-sample-output.png" alt-text="Salida de la aplicación de ejemplo de MQTT":::

En Azure IoT Explorer, puede ver que el dispositivo no es IoT Plug and Play:

:::image type="content" source="media/tutorial-use-mqtt/non-pnp-iot-explorer.png" alt-text="Dispositivo IoT Plug and Play en Azure IoT Explorer":::

### <a name="make-the-device-an-iot-plug-and-play-device"></a>Conversión de un dispositivo en IoT Plug and Play

Cualquier dispositivo IoT Plug and Play debe seguir un conjunto de convenciones simples. Si un dispositivo envía un identificador de modelo al conectarse, se convierte en un dispositivo IoT Plug and Play.

En este ejemplo, agregará el identificador de modelo** al paquete de conexión de MQTT. Pase el identificador del modelo como el parámetro querystring en `USERNAME` y cambie el valor de `api-version` a `2020-05-31-preview`:

```c
// computed Host Username and Topic
//#define USERNAME IOTHUBNAME ".azure-devices.net/" DEVICEID "/?api-version=2018-06-30"
#define USERNAME IOTHUBNAME ".azure-devices.net/" DEVICEID "/?api-version=2020-05-31-preview&model-id=dtmi:com:example:Thermostat;1"
#define PORT 8883
#define HOST IOTHUBNAME //".azure-devices.net"
#define TOPIC "devices/" DEVICEID "/messages/events/"
```

Recompile y ejecute el ejemplo.

El dispositivo gemelo ahora incluye el identificador del modelo:

:::image type="content" source="media/tutorial-use-mqtt/model-id-iot-explorer.png" alt-text="Visualización del identificador del modelo en Azure IoT Explorer":::

Ahora puede navegar por el componente IoT Plug and Play:

:::image type="content" source="media/tutorial-use-mqtt/components-iot-explorer.png" alt-text="Visualización de los componentes de Azure IoT Explorer":::

Ahora puede modificar el código del dispositivo para implementar los datos de telemetría, las propiedades y los comandos definidos en el modelo. Para ver una implementación de ejemplo del dispositivo termostato que usa la biblioteca Mosquitto, consulte el artículo sobre el [uso de MQTT PnP con Azure IoTHub sin el SDK de IoT en Windows](https://github.com/Azure-Samples/IoTMQTTSample/tree/master/src/Windows/PnPMQTTWin32) en GitHub.

> [!NOTE]
> De forma predeterminada, las firmas de acceso compartido tienen una validez de 60 minutos.

> [!NOTE]
>El cliente utiliza el archivo de certificado raíz `IoTHubRootCA_Baltimore.pem` para comprobar la identidad del centro de IoT al que se conecta.

### <a name="mqtt-topics"></a>Temas de MQTT

Las siguientes definiciones son para los temas de MQTT que el dispositivo usa para enviar información al centro de IoT. Para más información, consulte [Comunicación con la instancia de IoT Hub mediante el protocolo MQTT](../iot-hub/iot-hub-mqtt-support.md):

* `DEVICE_CAPABILITIES_MESSAGE` define el tema que el dispositivo usa para informar acerca de las interfaces que implementa.
* `DEVICETWIN_PATCH_MESSAGE` define el tema que el dispositivo usa para informar acerca de las actualizaciones de propiedades al centro de IoT.
* `DEVICE_TELEMETRY_MESSAGE` define el tema que el dispositivo usa para enviar datos de telemetría al centro de IoT.

Para más información sobre MQTT, visite el repositorio de GitHub de [ejemplos de MQTT para Azure IoT](https://github.com/Azure-Samples/IoTMQTTSample/).

[!INCLUDE [iot-pnp-clean-resources.md](../../includes/iot-pnp-clean-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a modificar un cliente de dispositivo MQTT para que siga las convenciones de IoT Plug and Play. Para más información acerca de IoT Plug and Play, consulte:

> [!div class="nextstepaction"]
> [Architecture](concepts-architecture.md)

Para más información acerca de la compatibilidad de Azure IoT Hub con el protocolo MQTT, consulte:

> [!div class="nextstepaction"]
> [Comunicación con la instancia de IoT Hub mediante el protocolo MQTT](../iot-hub/iot-hub-mqtt-support.md)
