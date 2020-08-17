---
title: Conexión del código C# del dispositivo de ejemplo de la versión preliminar de IoT Plug and Play a IoT Hub | Microsoft Docs
description: Compile y ejecute el código de dispositivo de ejemplo de la versión preliminar de IoT Plug and Play en Windows que se conecta a un centro de IoT. Use la herramienta Azure IoT Explorer para ver la información enviada por el dispositivo al centro.
author: ericmitt
ms.author: ericmitt
ms.date: 07/14/2020
ms.topic: quickstart
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: 015e20fa975563fee8ac2d61f9bad1f9f03738ce
ms.sourcegitcommit: 46f8457ccb224eb000799ec81ed5b3ea93a6f06f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87352670"
---
# <a name="quickstart-connect-a-sample-iot-plug-and-play-preview-device-application-running-on-windows-to-iot-hub-c"></a>Inicio rápido: Conexión de una aplicación de dispositivo de ejemplo de la versión preliminar de IoT Plug and Play en Windows a IoT Hub (C#)

[!INCLUDE [iot-pnp-quickstarts-device-selector.md](../../includes/iot-pnp-quickstarts-device-selector.md)]

En este inicio rápido se muestra cómo compilar una aplicación de dispositivo IoT Plug and Play de ejemplo, conectarla al centro de IoT y usar la herramienta Azure IoT Explorer para ver los datos de telemetría que envía. La aplicación de ejemplo se escribe en CSharp y se incluye en el SDK de dispositivo de Azure IoT para C#. Un generador de soluciones puede usar la herramienta Azure IoT Explorer para comprender las funcionalidades de cualquier dispositivo IoT Plug and Play sin necesidad de ver nada de código del dispositivo.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="prerequisites"></a>Requisitos previos

Para completar este inicio rápido en Windows, instale el siguiente software en el entorno Windows local:

* [Visual Studio 2019 (Community, Professional o Enterprise)](https://visualstudio.microsoft.com/downloads/).
* [Git](https://git-scm.com/download/).
* [CMake](https://cmake.org/download/).

### <a name="azure-iot-explorer"></a>Azure IoT Explorer

Para interactuar con el dispositivo de ejemplo en la segunda parte de esta guía de inicio rápido, use la herramienta **Azure IoT Explorer**. [Descargue e instale la versión de Azure IoT Explorer más reciente](./howto-use-iot-explorer.md) para su sistema operativo.

[!INCLUDE [iot-pnp-prepare-iot-hub.md](../../includes/iot-pnp-prepare-iot-hub.md)]

Ejecute el siguiente comando para obtener la _cadena de conexión del centro de IoT_ para el centro. Tome nota de esta cadena de conexión, ya que la usará más adelante en este inicio rápido:

```azurecli-interactive
az iot hub show-connection-string --hub-name <YourIoTHubName> --output table
```

> [!TIP]
> También puede usar la herramienta Azure IoT Explorer para buscar la cadena de conexión del centro de IoT.

Ejecute el siguiente comando para obtener la _cadena de conexión del dispositivo_ que ha agregado al centro. Tome nota de esta cadena de conexión, ya que la usará más adelante en este inicio rápido:

```azurecli-interactive
az iot hub device-identity show-connection-string --hub-name <YourIoTHubName> --device-id <YourDeviceID> --output table
```

[!INCLUDE [iot-pnp-download-models.md](../../includes/iot-pnp-download-models.md)]

## <a name="download-the-code"></a>Descargar el código

En este inicio rápido, se prepara un entorno de desarrollo que se puede usar para clonar y compilar el SDK de C# del dispositivo de Azure IoT Hub.

Abra un símbolo del sistema en el directorio que prefiera. Ejecute el siguiente comando para clonar el repositorio de GitHub de las [bibliotecas y los SDK de C# de Azure IoT](https://github.com/Azure/azure-iot-sdk-csharp) en esta ubicación:

```cmd
git clone https://github.com/Azure/azure-iot-sdk-csharp.git
```

## <a name="build-the-code"></a>Compilación del código

Abra el archivo de proyecto *azure-iot-sdk-csharp/iothub/device/samples/PnpDeviceSamples/Thermostat/Thermostat.csproj* en Visual Studio 2019.

Ahora puede compilar el ejemplo en Visual Studio y ejecutarlo en modo de depuración.

## <a name="run-the-device-sample"></a>Ejecución del ejemplo de dispositivo

Cree una variable de entorno denominada **IOTHUB_DEVICE_CONNECTION_STRING** para almacenar la cadena de conexión del dispositivo que anotó anteriormente.

Para realizar el seguimiento de la ejecución del código en Visual Studio en Windows, agregue un punto de interrupción a la función `main` en el archivo program.cs.

El dispositivo ya está listo para recibir comandos y actualizaciones de propiedades, y ha empezado a enviar datos de telemetría al centro. Siga ejecutando el ejemplo a medida que complete los pasos siguientes.

## <a name="use-azure-iot-explorer-to-validate-the-code"></a>Uso de Azure IoT Explorer para validar el código

Una vez iniciado el ejemplo de cliente de dispositivo, compruebe que funciona con la herramienta Azure IoT Explorer.

[!INCLUDE [iot-pnp-iot-explorer.md](../../includes/iot-pnp-iot-explorer.md)]

## <a name="review-the-code"></a>Revisión del código

En este ejemplo se implementa un dispositivo termostato de IoT Plug and Play sencillo. El modelo que se implementa en este ejemplo no usa los [componentes](concepts-components.md) de IoT Plug and Play. El [archivo de modelo del lenguaje de definición de Digital Twins (DTDL) del dispositivo termostato](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/Thermostat.json) define la telemetría, las propiedades y los comandos que implementa el dispositivo.

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

El código que actualiza las propiedades, controla los comandos y envía la telemetría es idéntico al código de un dispositivo que no usa las convenciones de IoT Plug and Play.

En el ejemplo se usa una biblioteca JSON para analizar objetos JSON en las cargas enviadas desde el centro de IoT:

```csharp
using Newtonsoft.Json;

...

DateTime since = JsonConvert.DeserializeObject<DateTime>(request.DataAsJson);
```

[!INCLUDE [iot-pnp-clean-resources.md](../../includes/iot-pnp-clean-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido ha aprendido a conectar un dispositivo IoT Plug and Play a un centro de IoT. Para más información sobre cómo crear una solución que interactúe con dispositivos IoT Plug and Play, consulte:

> [!div class="nextstepaction"]
> [Procedimiento: Conexión e interacción con un dispositivo](howto-develop-solution.md)
