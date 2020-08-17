---
title: Conexión del código del dispositivo del componente de C# de ejemplo de la versión preliminar de IoT Plug and Play a IoT Hub | Microsoft Docs
description: Compile y ejecute el código de dispositivo de C# de ejemplo de la versión preliminar de IoT Plug and Play que usa varios componentes y se conecta a un centro de IoT. Use la herramienta Azure IoT Explorer para ver la información enviada por el dispositivo al centro.
author: ericmitt
ms.author: ericmitt
ms.date: 07/14/2020
ms.topic: tutorial
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: 67b71399332fb29a277381a8c2806dbe7fb31d85
ms.sourcegitcommit: 1b2d1755b2bf85f97b27e8fbec2ffc2fcd345120
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87552169"
---
# <a name="tutorial-connect-an-iot-plug-and-play-multiple-component-device-application-running-on-windows-to-iot-hub-c"></a>Tutorial: Conexión de una aplicación de dispositivo de varios componentes de IoT Plug and Play en Windows a IoT Hub (C#)

[!INCLUDE [iot-pnp-tutorials-device-selector.md](../../includes/iot-pnp-tutorials-device-selector.md)]

En este tutorial se muestra cómo compilar una aplicación de dispositivo IoT Plug and Play de ejemplo con componentes e interfaz raíz, cómo conectarla a un centro de IoT y cómo usar la herramienta Azure IoT Explorer para ver la información que envía al centro. La aplicación de ejemplo se escribe en C# y se incluye en el SDK de dispositivo IoT de Azure para C#. Un generador de soluciones puede usar la herramienta Azure IoT Explorer para comprender las funcionalidades de cualquier dispositivo IoT Plug and Play sin necesidad de ver nada de código del dispositivo.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial en Windows, instale el siguiente software en el entorno Windows local:

* [Visual Studio 2019 (Community, Professional o Enterprise)](https://visualstudio.microsoft.com/downloads/).
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

En este tutorial, se prepara un entorno de desarrollo que se puede usar para clonar y compilar el SDK del dispositivo de Azure IoT Hub para C#.

Abra un símbolo del sistema en el directorio que prefiera. Ejecute el siguiente comando para clonar el repositorio de GitHub de las [bibliotecas y SDK de Azure IoT para C#](https://github.com/Azure/azure-iot-sdk-csharp) en esta ubicación:

```cmd
git clone https://github.com/Azure/azure-iot-sdk-csharp.git
```

## <a name="build-the-code"></a>Compilación del código

Abra el archivo de solución **azureiot.sln** en Visual Studio 2019 y establezca el proyecto **TemperatureController** como proyecto de inicio. En **Explorador de soluciones**, puede encontrar este archivo de proyecto en **iothub > device > samples**.

Ahora puede compilar el ejemplo en Visual Studio y ejecutarlo en modo de depuración.

## <a name="run-the-device-sample"></a>Ejecución del ejemplo de dispositivo

Cree una variable de entorno denominada **IOTHUB_DEVICE_CONNECTION_STRING** para almacenar la cadena de conexión del dispositivo que anotó anteriormente.

Para realizar el seguimiento de la ejecución del código en Visual Studio en Windows, agregue un punto de interrupción a la función `main` en el archivo program.cs.

El dispositivo ya está listo para recibir comandos y actualizaciones de propiedades, y ha empezado a enviar datos de telemetría al centro. Siga ejecutando el ejemplo a medida que complete los pasos siguientes.

## <a name="use-azure-iot-explorer-to-validate-the-code"></a>Uso de Azure IoT Explorer para validar el código

Una vez iniciado el ejemplo de cliente de dispositivo, compruebe que funciona con la herramienta Azure IoT Explorer.

[!INCLUDE [iot-pnp-iot-explorer.md](../../includes/iot-pnp-iot-explorer.md)]

## <a name="review-the-code"></a>Revisión del código

En este ejemplo se implementa un dispositivo controlador de temperatura IoT Plug and Play. El modelo que implementa este ejemplo usa [varios componentes](concepts-components.md). El [archivo de modelo del lenguaje de definición de gemelos digitales (DTDL) del dispositivo de temperatura](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/TemperatureController.json) define los datos de telemetría, las propiedades y los comandos que implementa el dispositivo.

El código del dispositivo se conecta al centro de IoT mediante el método `CreateFromConnectionString` estándar. El dispositivo envía el identificador de modelo del modelo DTDL que implementa en la solicitud de conexión. Un dispositivo que envía un identificador de modelo es un dispositivo IoT Plug and Play:

```csharp
private static void InitializeDeviceClientAsync()
{
  var options = new ClientOptions
  {
      ModelId = ModelId,
  };
  s_deviceClient = DeviceClient.CreateFromConnectionString(s_deviceConnectionString, TransportType.Mqtt, options);
  s_deviceClient.SetConnectionStatusChangesHandler((status, reason) =>
  {
      s_logger.LogDebug($"Connection status change registered - status={status}, reason={reason}.");
  });
}
```

El identificador del modelo se almacena en el código tal y como se muestra en el siguiente fragmento de código:

```csharp
private const string ModelId = "dtmi:com:example:Thermostat;1";
```

Una vez que el dispositivo se conecta al centro de IoT, el código registra los controladores de comandos. El comando `reboot` se define en la interfaz raíz. El comando `getMaxMinReport` se define en cada uno de los dos componentes de termostato:

```csharp
await s_deviceClient.SetMethodHandlerAsync("reboot", HandleRebootCommandAsync, s_deviceClient);
await s_deviceClient.SetMethodHandlerAsync("thermostat1*getMaxMinReport", HandleMaxMinReportCommandAsync, Thermostat1);
await s_deviceClient.SetMethodHandlerAsync("thermostat2*getMaxMinReport", HandleMaxMinReportCommandAsync, Thermostat2);
```

Existen controladores independientes para las actualizaciones de propiedades deseadas en los dos componentes de termostato:

```csharp
s_desiredPropertyUpdateCallbacks.Add(Thermostat1, TargetTemperatureUpdateCallbackAsync);
s_desiredPropertyUpdateCallbacks.Add(Thermostat2, TargetTemperatureUpdateCallbackAsync);
```

El código de ejemplo envía los datos de telemetría de cada componente de termostato:

```csharp
await SendTemperatureAsync(Thermostat1);
await SendTemperatureAsync(Thermostat2);
```

El método `SendTemperature` usa la clase `PnpHhelper` para crear mensajes para cada componente:

```csharp
Message msg = PnpHelper.CreateIothubMessageUtf8(telemetryName, JsonConvert.SerializeObject(currentTemperature), componentName);
```

La clase `PnpHelper` contiene otros métodos de ejemplo que se pueden usar con un modelo de varios componentes.

Use la herramienta Azure IoT Explorer para ver los datos de telemetría y las propiedades de los dos componentes de termostato:

:::image type="content" source="media/tutorial-multiple-components-csharp/multiple-component.png" alt-text="Dispositivo de varios componentes en Azure IoT Explorer":::

También puede usar la herramienta Azure IoT Explorer para llamar a comandos en cualquiera de los dos componentes de termostato o en la interfaz raíz.

[!INCLUDE [iot-pnp-clean-resources.md](../../includes/iot-pnp-clean-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha aprendido a conectar un dispositivo IoT Plug and Play con componentes a un centro de IoT. Para obtener más información sobre los modelos de dispositivo IoT Plug and Play, consulte:

> [!div class="nextstepaction"]
> [Guía para desarrolladores de modelado de IoT Plug and Play (versión preliminar)](concepts-developer-guide.md)
