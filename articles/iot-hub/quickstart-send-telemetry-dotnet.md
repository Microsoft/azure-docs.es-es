---
title: Guía de inicio rápido para enviar datos de telemetría a Azure IoT Hub (C#) | Microsoft Docs
description: En esta guía de inicio rápido, ejecutará dos aplicaciones C# de ejemplo para enviar datos de telemetría simulados a un centro de IoT y para leer los datos de telemetría procedentes de este para procesarlos en la nube.
author: robinsh
manager: philmea
ms.author: robinsh
ms.service: iot-hub
services: iot-hub
ms.devlang: csharp
ms.topic: quickstart
ms.custom:
- mvc
- mqtt
- 'Role: Cloud Development'
ms.date: 06/01/2020
ms.openlocfilehash: 98b50649b5a788270fa2b4cd8b62ca5598daa25f
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87320483"
---
# <a name="quickstart-send-telemetry-from-a-device-to-an-iot-hub-and-read-it-with-a-back-end-application-net"></a>Inicio rápido: Envío de telemetría desde un dispositivo a un centro de IoT y su lectura con una aplicación de back-end (.NET)

[!INCLUDE [iot-hub-quickstarts-1-selector](../../includes/iot-hub-quickstarts-1-selector.md)]

IoT Hub es un servicio de Azure que le permite ingerir grandes volúmenes de datos de telemetría desde los dispositivos IoT en la nube para su almacenamiento o procesamiento. En este inicio rápido, enviará datos de telemetría desde una aplicación de dispositivo simulado, por IoT Hub, a una aplicación back-end para su procesamiento.

El inicio rápido usa dos aplicaciones C# escritas previamente: una para enviar datos de telemetría y otra para leer datos de telemetría desde el centro. Antes de ejecutar estas dos aplicaciones, creará un centro de IoT Hub y registrará un dispositivo con el centro.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Las dos aplicaciones de ejemplo que se ejecutan en este inicio rápido se escriben con C#. Necesita el SDK de .NET Core 3.0, o una versión posterior, en la máquina de desarrollo.

Puede descargar el SDK de .NET Core para varias plataformas desde [.NET](https://www.microsoft.com/net/download/all).

Puede verificar la versión actual de C# en el equipo de desarrollo con el comando siguiente:

```cmd/sh
dotnet --version
```

> [!NOTE]
> Se recomienda el SDK de .NET Core 3.0, o una versión posterior, para compilar el código del servicio Event Hubs que se usa para leer la telemetría en este inicio rápido. Puede usar el SDK de .NET Core 2.1 si establece la versión del idioma del código de servicio en versión preliminar previa, como se indica en la sección [Lectura de los datos de telemetría procedentes de su instancia de IoT Hub](#read-the-telemetry-from-your-hub).

Ejecute el siguiente comando para agregar la extensión IoT de Microsoft Azure para la CLI de Azure a la instancia de Cloud Shell. La extensión IOT agrega comandos específicos de IoT Hub, IoT Edge e IoT Device Provisioning Service (DPS) a la CLI de Azure.

```azurecli-interactive
az extension add --name azure-iot
```

[!INCLUDE [iot-hub-cli-version-info](../../includes/iot-hub-cli-version-info.md)]

Descargue los ejemplos en C# de Azure IoT de [https://github.com/Azure-Samples/azure-iot-samples-csharp/archive/master.zip](https://github.com/Azure-Samples/azure-iot-samples-csharp/archive/master.zip) y extraiga el archivo ZIP.

Asegúrese de que está abierto el puerto 8883 del firewall. En el dispositivo de ejemplo de este inicio rápido se usa el protocolo MQTT, que se comunica mediante el puerto 8883. Este puerto puede estar bloqueado en algunos entornos de red corporativos y educativos. Para más información y saber cómo solucionar este problema, consulte [Conexión a IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="create-an-iot-hub"></a>Crear un centro de IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-device"></a>Registrar un dispositivo

Debe registrar un dispositivo con IoT Hub antes de poder conectarlo. En esta guía de inicio rápido, usará Azure Cloud Shell para registrar un dispositivo simulado.

1. Ejecute los siguientes comandos en Azure Cloud Shell para crear la identidad del dispositivo.

   **YourIoTHubName**: reemplace este marcador de posición por el nombre elegido para el centro de IoT.

   **MyDotnetDevice**: es el nombre del dispositivo que se va a registrar. Se recomienda usar **MyDotnetDevice** como se muestra. Si elige un nombre distinto para el dispositivo, tendrá que usarlo en todo el artículo y actualizar el nombre del dispositivo en las aplicaciones de ejemplo antes de ejecutarlas.

    ```azurecli-interactive
    az iot hub device-identity create --hub-name {YourIoTHubName} --device-id MyDotnetDevice
    ```

2. Ejecute el siguiente comandos en Azure Cloud Shell para obtener la _cadena de conexión_ del dispositivo que acaba de registrar:

   **YourIoTHubName**: reemplace este marcador de posición por el nombre elegido para el centro de IoT.

    ```azurecli-interactive
    az iot hub device-identity show-connection-string --hub-name {YourIoTHubName} --device-id MyDotnetDevice --output table
    ```

    Anote la cadena de conexión del dispositivo, que se parecerá a esta:

   `HostName={YourIoTHubName}.azure-devices.net;DeviceId=MyDotnetDevice;SharedAccessKey={YourSharedAccessKey}`

    Este valor lo usará más adelante en este inicio rápido.

3. También necesitará el _punto de conexión compatible con Event Hubs_, la _ruta de acceso compatible con Event Hubs_ y la _clave principal de servicio_ de IoT Hub para permitir que la aplicación back-end se conecte a IoT Hub y recupere los mensajes. Los siguientes comandos recuperan estos valores para IoT Hub:

   **YourIoTHubName**: reemplace este marcador de posición por el nombre elegido para el centro de IoT.

    ```azurecli-interactive
    az iot hub show --query properties.eventHubEndpoints.events.endpoint --name {YourIoTHubName}

    az iot hub show --query properties.eventHubEndpoints.events.path --name {YourIoTHubName}

    az iot hub policy show --name service --query primaryKey --hub-name {YourIoTHubName}
    ```

    Anote estos tres valores, ya que los usará más adelante en el inicio rápido.

## <a name="send-simulated-telemetry"></a>Envío de datos de telemetría simulados

La aplicación de dispositivo simulado se conecta a un punto de conexión específico del dispositivo en IoT Hub y envía datos de telemetría simulados sobre temperatura y humedad.

1. En una ventana de terminal local, vaya a la carpeta raíz del proyecto de C# de muestra. A continuación, vaya a la carpeta **iot-hub\Quickstarts\simulated-device**.

2. Abra el archivo **SimulatedDevice.cs** en el editor de texto de su elección.

    Reemplace el valor de la variable `s_connectionString` por la cadena de conexión del dispositivo que anotó anteriormente. Luego, guarde los cambios realizados en **SimulatedDevice.cs**.

3. En la ventana de terminal local, ejecute los comandos siguientes para instalar los paquetes necesarios para la aplicación de dispositivo simulado:

    ```cmd/sh
    dotnet restore
    ```

4. En la ventana de terminal local, ejecute el comando siguiente para compilar la aplicación de dispositivo simulado y ejecutarla:

    ```cmd/sh
    dotnet run
    ```

    La siguiente captura de pantalla muestra la salida en la que la aplicación de dispositivo simulado envía datos de telemetría a IoT Hub:

    ![Ejecutar el dispositivo simulado](media/quickstart-send-telemetry-dotnet/simulated-device.png)

## <a name="read-the-telemetry-from-your-hub"></a>Lectura de los datos de telemetría procedentes de su instancia de IoT Hub

La aplicación back-end se conecta a un punto de conexión de **Eventos** de servicio en IoT Hub. La aplicación recibe los mensajes del dispositivo a la nube enviados desde el dispositivo simulado. Normalmente, una aplicación back-end de IoT Hub se ejecuta en la nube para recibir y procesar los mensajes del dispositivo a la nube.

1. En otra ventana de terminal local, vaya a la carpeta raíz del proyecto de C# de muestra. A continuación, vaya a la carpeta **iot-hub\Quickstarts\read-d2c-messages**.

2. Abra el archivo **ReadDeviceToCloudMessages.cs** en el editor de texto de su elección. Actualice las siguientes variables y guarde los cambios en el archivo.

    | Variable | Value |
    | -------- | ----------- |
    | `EventHubsCompatibleEndpoint` | Reemplace el valor de la variable por el punto de conexión compatible con Event Hubs que anotó anteriormente. |
    | `EventHubName`                | Reemplace el valor de la variable por la ruta de acceso compatible con Event Hubs que ha anotó anteriormente. |
    | `IotHubSasKey`                | Reemplace el valor de la variable por la clave principal del servicio que anotó anteriormente. |

    > [!NOTE]
    > Si usa el SDK de .NET Core 2.1, debe establecer la versión de idioma en versión preliminar para compilar el código. Para ello, abra el archivo **read-d2c-messages.csproj** y establezca el valor del elemento`<LangVersion>` en `preview`.

3. En la ventana de terminal local, ejecute los comandos siguientes para instalar las bibliotecas necesarias para la aplicación back-end:

    ```cmd/sh
    dotnet restore
    ```

4. En la ventana de terminal local, ejecute los comandos siguientes para compilar la aplicación back-end y ejecutarla:

    ```cmd/sh
    dotnet run
    ```

    La siguiente captura de pantalla muestra la salida en la que la aplicación back-end recibe los datos de telemetría que el dispositivo simulado ha enviado al centro:

    ![Ejecutar la aplicación back-end](media/quickstart-send-telemetry-dotnet/read-device-to-cloud.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [iot-hub-quickstarts-clean-up-resources](../../includes/iot-hub-quickstarts-clean-up-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha configurado un centro de IoT, ha registrado un dispositivo, ha enviado datos de telemetría simulados al centro de conectividad mediante una aplicación en C# y ha leído datos de telemetría procedentes del centro de conectividad mediante una sencilla aplicación back-end.

Para aprender a controlar el dispositivo simulado desde una aplicación back-end, vaya al siguiente inicio rápido.

> [!div class="nextstepaction"]
> [Inicio rápido: Control de un dispositivo conectado a IoT Hub](quickstart-control-device-dotnet.md)