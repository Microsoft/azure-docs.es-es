---
title: Conexión del código de dispositivo de C de ejemplo de la versión preliminar de IoT Plug and Play a IoT Hub | Microsoft Docs
description: Compile y ejecute el código de dispositivo de C de ejemplo de la versión preliminar de IoT Plug and Play que usa varios componentes y realiza la conexión a un centro de IoT. Use la herramienta Azure IoT Explorer para ver la información enviada por el dispositivo al centro.
author: ericmitt
ms.author: ericmitt
ms.date: 07/22/2020
ms.topic: tutorial
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: 29017ec11429b26018093980ca71c317b12085b5
ms.sourcegitcommit: 59ea8436d7f23bee75e04a84ee6ec24702fb2e61
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/07/2020
ms.locfileid: "89505760"
---
# <a name="tutorial-connect-an-iot-plug-and-play-multiple-component-device-applications-running-on-linux-or-windows-to-iot-hub-c"></a>Tutorial: Conexión de una aplicación de dispositivo de varios componentes de IoT Plug and Play en Linux o Windows a IoT Hub (C)

[!INCLUDE [iot-pnp-tutorials-device-selector.md](../../includes/iot-pnp-tutorials-device-selector.md)]

En este tutorial se muestra cómo compilar una aplicación de dispositivo IoT Plug and Play de ejemplo con componentes e interfaz raíz, cómo conectarla a un centro de IoT y cómo usar la herramienta Azure IoT Explorer para ver la información que envía al centro. La aplicación de ejemplo está escrita en C y se incluye en el SDK de dispositivo IoT de Azure para C. Un generador de soluciones puede usar la herramienta Azure IoT Explorer para comprender las funcionalidades de cualquier dispositivo IoT Plug and Play sin necesidad de ver nada de código del dispositivo.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="prerequisites"></a>Requisitos previos

Para este tutorial se puede usar Linux o Windows. Los comandos de shell de este tutorial siguen la convención de Linux para los separadores de ruta de acceso "`/`". Si utiliza Windows, asegúrese de cambiar estos separadores por "`\`".

Los requisitos previos difieren en función del sistema operativo:

### <a name="linux"></a>Linux

En este tutorial se da por supuesto que usa Ubuntu Linux. Los pasos de este tutorial se han probado con Ubuntu 18.04.

Para realizar este tutorial en Linux, instale el siguiente software en el entorno de Linux local:

Instale **GCC**, **Git**, **cmake** y todas las dependencias necesarias con el comando `apt-get`:

```sh
sudo apt-get update
sudo apt-get install -y git cmake build-essential curl libcurl4-openssl-dev libssl-dev uuid-dev
```

Compruebe que la versión de `cmake` es posterior a **2.8.12** y que la versión de **GCC** es posterior a **4.4.7**.

```sh
cmake --version
gcc --version
```

### <a name="windows"></a>Windows

Para completar este tutorial en Windows, instale el siguiente software en el entorno Windows local:

* [Visual Studio (Community, Professional o Enterprise)](https://visualstudio.microsoft.com/downloads/): asegúrese de incluir la carga de trabajo **Desarrollo para el escritorio con C++** cuando [instale](https://docs.microsoft.com/cpp/build/vscpp-step-0-installation?view=vs-2019) Visual Studio.
* [Git](https://git-scm.com/download/).
* [CMake](https://cmake.org/download/).

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

## <a name="download-the-code"></a>Descargar el código

En este tutorial, se prepara un entorno de desarrollo que se puede usar para clonar y compilar el SDK de dispositivos de Azure IoT Hub para C.

Abra un símbolo del sistema en la carpeta de su elección. Ejecute el siguiente comando para clonar el repositorio de GitHub de las [bibliotecas y los SDK de C de Azure IoT](https://github.com/Azure/azure-iot-sdk-c) en esta ubicación:

```cmd/bash
git clone https://github.com/Azure/azure-iot-sdk-c.git
cd azure-iot-sdk-c
git submodule update --init
```

Esta operación puede tardar varios minutos en completarse.

## <a name="build-and-run-the-code"></a>Compilación y ejecución del código

Puede compilar y ejecutar el código con Visual Studio o usar `cmake` en la línea de comandos.

### <a name="use-visual-studio"></a>Usar Visual Studio

1. Abra la carpeta raíz del repositorio clonado. Al cabo de unos segundos, la compatibilidad de **CMake** en Visual Studio crea todo lo que necesita para ejecutar y depurar el proyecto.
1. Cuando Visual Studio esté listo, en el **Explorador de soluciones**, desplácese hasta el ejemplo *iothub_client/samples/PNP/pnp_temperature_controller/* .
1. Haga clic con el botón derecho en el archivo *pnp_temperature_controller.c* y seleccione **Add Debug Configuration** (Agregar configuración de depuración). Seleccione **Predeterminado**.
1. Visual Studio abre el archivo *launch.vs.json*. Edite este archivo como se muestra en el siguiente fragmento de código para establecer las variables de entorno necesarias:

    ```json
    {
      "version": "0.2.1",
      "defaults": {},
      "configurations": [
        {
          "type": "default",
          "project": "iothub_client\\samples\\pnp\\pnp_temperature_controller\\pnp_temperature_controller.c",
          "projectTarget": "",
          "name": "pnp_temperature_controller.c",
          "env": {
            "IOTHUB_DEVICE_SECURITY_TYPE": "connectionString",
            "IOTHUB_DEVICE_CONNECTION_STRING": "<Your device connection string>"
          }
        }
      ]
    }
    ```

1. Haga clic con el botón derecho en el archivo *pnp_temperature_controller.c* y seleccione **Establecer como elemento de inicio**.
1. Para realizar el seguimiento de la ejecución del código en Visual Studio, agregue un punto de interrupción a la función `main` en el archivo *pnp_temperature_controller.c*.
1. Ahora puede ejecutar y depurar el ejemplo desde el menú **Depurar**.

El dispositivo ya está listo para recibir comandos y actualizaciones de propiedades, y ha empezado a enviar datos de telemetría al centro. Siga ejecutando el ejemplo a medida que complete los pasos siguientes.

### <a name="use-cmake-at-the-command-line"></a>En la línea de comandos, use `cmake`.

Para compilar el ejemplo:

1. Cree una subcarpeta _cmake_ en la carpeta raíz del SDK del dispositivo clonado y vaya a esa carpeta:

    ```cmd/bash
    cd azure-iot-sdk-c
    mkdir cmake
    cd cmake
    ```

1. Ejecute los siguientes comandos para generar y compilar los archivos de proyecto del SDK y los ejemplos:

    ```cmd/bash
    cmake ..
    cmake --build .
    ```

Para ejecutar el ejemplo:

1. Cree dos variables de entorno para configurar el ejemplo para que use una cadena de conexión para establecer la conexión con su centro de IoT:

    * **IOTHUB_DEVICE_SECURITY_TYPE** con el valor `"connectionString"`
    * **IOTHUB_DEVICE_CONNECTION_STRING** para almacenar la cadena de conexión del dispositivo que anotó anteriormente.

1. Desde la carpeta _cmake_, navegue hasta la carpeta que contiene el archivo ejecutable y ejecute lo siguiente:

    ```bash
    # Bash
    cd iothub_client/samples/pnp/pnp_temperature_controller/
    ./pnp_temperature_controller
    ```

    ```cmd
    REM Windows
    iothub_client\samples\pnp\pnp_temperature_controller\Debug\pnp_temperature_controller.exe
    ```

El dispositivo ya está listo para recibir comandos y actualizaciones de propiedades, y ha empezado a enviar datos de telemetría al centro. Siga ejecutando el ejemplo a medida que complete los pasos siguientes.

### <a name="use-the-azure-iot-explorer-to-validate-the-code"></a>Uso de Azure IoT Explorer para validar el código

Una vez iniciado el ejemplo de cliente de dispositivo, compruebe que funciona con la herramienta Azure IoT Explorer.

[!INCLUDE [iot-pnp-iot-explorer.md](../../includes/iot-pnp-iot-explorer.md)]

## <a name="review-the-code"></a>Revisión del código

En este ejemplo se implementa un dispositivo controlador de temperatura IoT Plug and Play. En este ejemplo se implementa un modelo con [varios componentes](concepts-components.md). El [archivo de modelo del lenguaje de definición de gemelos digitales (DTDL) del dispositivo de temperatura](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/TemperatureController.json) define los datos de telemetría, las propiedades y los comandos que implementa el dispositivo.

### <a name="iot-plug-and-play-helper-functions"></a>Funciones auxiliares de IoT Plug and Play

En este ejemplo, el código usa algunas funciones auxiliares de la carpeta */common*:

*pnp_device_client_ll* contiene el método de conexión para IoT Plug and Play con `model-id` incluido como parámetro: `PnP_CreateDeviceClientLLHandle`.

*pnp_protocol*: contiene las funciones auxiliares de IoT Plug and Play:

* `PnP_CreateReportedProperty`
* `PnP_CreateReportedPropertyWithStatus`
* `PnP_ParseCommandName`
* `PnP_CreateTelemetryMessageHandle`
* `PnP_ProcessTwinData`
* `PnP_CopyPayloadToString`
* `PnP_CreateDeviceClientLLHandle_ViaDps`

Estas funciones auxiliares son lo suficientemente genéricas como para usarse en su propio proyecto. En este ejemplo se usan en los tres archivos correspondientes a cada componente del modelo:

* *pnp_deviceinfo_component*
* *pnp_temperature_controller*
* *pnp_thermostat_component*

Por ejemplo, en el archivo *pnp_deviceinfo_component*, la función `SendReportedPropertyForDeviceInformation` usa dos de las funciones auxiliares:

```c
if ((jsonToSend = PnP_CreateReportedProperty(componentName, propertyName, propertyValue)) == NULL)
{
    LogError("Unable to build reported property response for propertyName=%s, propertyValue=%s", propertyName, propertyValue);
}
else
{
    const char* jsonToSendStr = STRING_c_str(jsonToSend);
    size_t jsonToSendStrLen = strlen(jsonToSendStr);

    if ((iothubClientResult = IoTHubDeviceClient_LL_SendReportedState(deviceClientLL, (const unsigned char*)jsonToSendStr, jsonToSendStrLen, NULL, NULL)) != IOTHUB_CLIENT_OK)
    {
        LogError("Unable to send reported state for property=%s, error=%d", propertyName, iothubClientResult);
    }
    else
    {
        LogInfo("Sending device information property to IoTHub.  propertyName=%s, propertyValue=%s", propertyName, propertyValue);
    }
}
```

Cada componente del ejemplo sigue este patrón.

### <a name="code-flow"></a>flujo de código

La función `main` inicializa la conexión y envía el identificador del modelo:

```c
deviceClient = CreateDeviceClientAndAllocateComponents();
```

El código usa `PnP_CreateDeviceClientLLHandle` para conectarse al centro de IoT, establece `modelId` como opción y configura el método de dispositivo y los controladores de devolución de llamada de dispositivos gemelos para métodos directos y actualizaciones de dispositivos gemelos:

```c
g_pnpDeviceConfiguration.deviceMethodCallback = PnP_TempControlComponent_DeviceMethodCallback;
g_pnpDeviceConfiguration.deviceTwinCallback = PnP_TempControlComponent_DeviceTwinCallback;
g_pnpDeviceConfiguration.modelId = g_temperatureControllerModelId;
...

deviceClient = PnP_CreateDeviceClientLLHandle(&g_pnpDeviceConfiguration);
```

`&g_pnpDeviceConfiguration` también contiene la información de conexión. La variable de entorno **IOTHUB_DEVICE_SECURITY_TYPE** determina si en el ejemplo se usa una cadena de conexión o el servicio de aprovisionamiento de dispositivos para conectarse a IoT Hub.

Cuando el dispositivo envía un identificador de modelo, se convierte en un dispositivo IoT Plug and Play.

Con los controladores de devolución de llamada colocados, el dispositivo reacciona a las actualizaciones de los dispositivos gemelos y a las llamadas a métodos directos:

* En la devolución de llamada de dispositivos gemelos, `PnP_TempControlComponent_DeviceTwinCallback` llama a la función `PnP_ProcessTwinData` para procesar los datos. `PnP_ProcessTwinData` usa el *patrón de visitante* para analizar el archivo JSON y, luego, visita cada propiedad, mediante la llamada a `PnP_TempControlComponent_ApplicationPropertyCallback` en cada elemento.

* En la devolución de llamada de los comandos, la función `PnP_TempControlComponent_DeviceMethodCallback` utiliza la función auxiliar para analizar los nombres de los comandos y componentes:

    ```c
    PnP_ParseCommandName(methodName, &componentName, &componentNameSize, &pnpCommandName);
    ```

    A continuación, la función `PnP_TempControlComponent_DeviceMethodCallback` llama al comando en el componente:

    ```c
    LogInfo("Received PnP command for component=%.*s, command=%s", (int)componentNameSize, componentName, pnpCommandName);
    if (strncmp((const char*)componentName, g_thermostatComponent1Name, g_thermostatComponent1Size) == 0)
    {
        result = PnP_ThermostatComponent_ProcessCommand(g_thermostatHandle1, pnpCommandName, rootValue, response, responseSize);
    }
    else if (strncmp((const char*)componentName, g_thermostatComponent2Name, g_thermostatComponent2Size) == 0)
    {
        result = PnP_ThermostatComponent_ProcessCommand(g_thermostatHandle2, pnpCommandName, rootValue, response, responseSize);
    }
    else
    {
        LogError("PnP component=%.*s is not supported by TemperatureController", (int)componentNameSize, componentName);
        result = PNP_STATUS_NOT_FOUND;
    }
    ```

La función `main` inicializa las propiedades de solo lectura enviadas al centro de IoT:

```c
PnP_TempControlComponent_ReportSerialNumber_Property(deviceClient);
PnP_DeviceInfoComponent_Report_All_Properties(g_deviceInfoComponentName, deviceClient);
PnP_TempControlComponent_Report_MaxTempSinceLastReboot_Property(g_thermostatHandle1, deviceClient);
PnP_TempControlComponent_Report_MaxTempSinceLastReboot_Property(g_thermostatHandle2, deviceClient);
```

La función `main` entra en un bucle para actualizar los datos de eventos y telemetría de cada componente:

```c
while (true)
{
    PnP_TempControlComponent_SendWorkingSet(deviceClient);
    PnP_ThermostatComponent_SendTelemetry(g_thermostatHandle1, deviceClient);
    PnP_ThermostatComponent_SendTelemetry(g_thermostatHandle2, deviceClient);
}
```

La función `PnP_ThermostatComponent_SendTelemetry` muestra cómo usar la estructura `PNP_THERMOSTAT_COMPONENT`. En el ejemplo se usa esta estructura para almacenar información sobre los dos termostatos en el controlador de temperatura. El código usa la función `PnP_CreateTelemetryMessageHandle` para preparar el mensaje y enviarlo:

```c
messageHandle = PnP_CreateTelemetryMessageHandle(pnpThermostatComponent->componentName, temperatureStringBuffer);
...
iothubResult = IoTHubDeviceClient_LL_SendEventAsync(deviceClientLL, messageHandle, NULL, NULL);
```

Finalmente, la función `main` destruye los distintos componentes y cierra la conexión al centro.

[!INCLUDE [iot-pnp-clean-resources.md](../../includes/iot-pnp-clean-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha aprendido a conectar un dispositivo IoT Plug and Play con componentes a un centro de IoT. Para obtener más información sobre los modelos de dispositivo IoT Plug and Play, consulte:

> [!div class="nextstepaction"]
> [Guía para desarrolladores de modelado de IoT Plug and Play (versión preliminar)](concepts-developer-guide.md)
