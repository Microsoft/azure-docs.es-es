---
title: Inicio rápido para controlar un dispositivo desde Azure IoT Hub (.NET) | Microsoft Docs
description: En este inicio rápido, ejecuta dos aplicaciones C# de muestra. Una aplicación es una aplicación back-end que puede controlar dispositivos conectados al centro de manera remota. La otra aplicación simula un dispositivo conectado al centro que se puede controlar de manera remota.
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
ms.date: 03/04/2020
ms.openlocfilehash: 1b3b8382c81015e3278954dd0443ba44520e2e3b
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87315149"
---
# <a name="quickstart-control-a-device-connected-to-an-iot-hub-net"></a>Inicio rápido: Control de un dispositivo conectado a un centro de IoT (.NET)

[!INCLUDE [iot-hub-quickstarts-2-selector](../../includes/iot-hub-quickstarts-2-selector.md)]

IoT Hub es un servicio de Azure que permite administrar dispositivos IoT desde la nube e ingerir grandes volúmenes de datos de telemetría desde los dispositivos en la nube para su almacenamiento o procesamiento. En este inicio rápido, usa un *método directo* para controlar un dispositivo simulado conectado a IoT Hub. Puede usar métodos directos para cambiar el comportamiento de un dispositivo conectado a IoT Hub de manera remota.

El inicio rápido usa dos aplicaciones .NET escritas anteriormente:

* Una aplicación de dispositivo simulado que responde a métodos directos que se llaman desde una aplicación back-end. Para recibir las llamadas de método directo, esta aplicación se conecta a un punto de conexión específico del dispositivo en IoT Hub.

* Una aplicación back-end que llama a los métodos directos en el dispositivo simulado. Para llamar a un método directo en un dispositivo, esta aplicación se conecta a un punto de conexión de servicio en IoT Hub.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Las dos aplicaciones de ejemplo que se ejecutan en este inicio rápido se escriben con C#. Necesita el SDK de .NET Core 2.1.0 o una versión posterior en el equipo de desarrollo.

Puede descargar el SDK de .NET Core para varias plataformas desde [.NET](https://www.microsoft.com/net/download/all).

Puede verificar la versión actual de C# en el equipo de desarrollo con el comando siguiente:

```cmd/sh
dotnet --version
```

Ejecute el siguiente comando para agregar la extensión IoT de Microsoft Azure para la CLI de Azure a la instancia de Cloud Shell. La extensión IOT agrega comandos específicos de IoT Hub, IoT Edge e IoT Device Provisioning Service (DPS) a la CLI de Azure.

```azurecli-interactive
az extension add --name azure-iot
```

[!INCLUDE [iot-hub-cli-version-info](../../includes/iot-hub-cli-version-info.md)]

Si aún no lo ha hecho, descargue los ejemplos de C# de Azure IoT de https://github.com/Azure-Samples/azure-iot-samples-csharp/archive/master.zip y extraiga el archivo ZIP.

Asegúrese de que está abierto el puerto 8883 del firewall. En el dispositivo de ejemplo de este inicio rápido se usa el protocolo MQTT, que se comunica mediante el puerto 8883. Este puerto puede estar bloqueado en algunos entornos de red corporativos y educativos. Para más información y saber cómo solucionar este problema, consulte [Conexión a IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="create-an-iot-hub"></a>Crear un centro de IoT

Si ha completado la anterior [Guía de inicio rápido: Envío de telemetría desde un dispositivo a un centro de IoT](quickstart-send-telemetry-dotnet.md) puede omitir este paso.

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-device"></a>Registrar un dispositivo

Si ha completado la anterior [Guía de inicio rápido: Envío de telemetría desde un dispositivo a un centro de IoT](quickstart-send-telemetry-dotnet.md) puede omitir este paso.

Debe registrar un dispositivo con IoT Hub antes de poder conectarlo. En esta guía de inicio rápido, usará Azure Cloud Shell para registrar un dispositivo simulado.

1. Ejecute los siguientes comandos en Azure Cloud Shell para crear la identidad del dispositivo.

   **YourIoTHubName**: reemplace este marcador de posición por el nombre elegido para el centro de IoT.

   **MyDotnetDevice**: es el nombre del dispositivo que se va a registrar. Se recomienda usar **MyDotnetDevice** como se muestra. Si elige otro nombre distinto para el dispositivo, tendrá que usarlo en todo el artículo y actualizar el nombre del dispositivo en las aplicaciones de ejemplo antes de ejecutarlas.

    ```azurecli-interactive
    az iot hub device-identity create \
      --hub-name {YourIoTHubName} --device-id MyDotnetDevice
    ```

2. Ejecute los siguientes comandos en Azure Cloud Shell para obtener la _cadena de conexión del dispositivo_ que acaba de registrar:

   **YourIoTHubName**: reemplace este marcador de posición por el nombre elegido para el centro de IoT.

    ```azurecli-interactive
    az iot hub device-identity show-connection-string \
      --hub-name {YourIoTHubName} \
      --device-id MyDotnetDevice \
      --output table
    ```

    Anote la cadena de conexión del dispositivo, que se parecerá a esta:

   `HostName={YourIoTHubName}.azure-devices.net;DeviceId=MyNodeDevice;SharedAccessKey={YourSharedAccessKey}`

    Usará este valor más adelante en este inicio rápido.

## <a name="retrieve-the-service-connection-string"></a>Recuperación de la cadena de conexión de servicio

También necesita una _cadena de conexión de servicio_ de IoT Hub para permitir que la aplicación back-end se conecte al centro y recupere los mensajes. El comando siguiente recupera la cadena de conexión del servicio de su instancia de IoT Hub:

```azurecli-interactive
az iot hub show-connection-string --policy-name service --name {YourIoTHubName} --output table
```

Anote la cadena de conexión del servicio, que se parecerá a esta:

   `HostName={YourIoTHubName}.azure-devices.net;SharedAccessKeyName=service;SharedAccessKey={YourSharedAccessKey}`

Usará este valor más adelante en este inicio rápido. Esta cadena de conexión del servicio no es la que anotó en el paso anterior.

## <a name="listen-for-direct-method-calls"></a>Escuchas para llamadas de método directo

La aplicación del dispositivo simulado se conecta a un punto de conexión específico del dispositivo en IoT Hub, envía los datos de telemetría simulados y escucha llamadas de método directo desde el centro. En este inicio rápido, la llamada de método directo desde el centro indica al dispositivo que debe cambiar el intervalo en el que envía los datos de telemetría. El dispositivo simulado envía una confirmación al centro después de que ejecuta el método directo.

1. En una ventana de terminal local, vaya a la carpeta raíz del proyecto de C# de muestra. A continuación, vaya a la carpeta **iot-hub\Quickstarts\simulated-device-2**.

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

    ![Ejecutar el dispositivo simulado](./media/quickstart-control-device-dotnet/SimulatedDevice-1.png)

## <a name="call-the-direct-method"></a>Llamar al método directo

La aplicación back-end se conecta a un punto de conexión de servicio en IoT Hub. La aplicación realiza llamadas de método directo a un dispositivo con IoT Hub y escucha las confirmaciones. Normalmente, una aplicación back-end de IoT Hub se ejecuta en la nube.

1. En otra ventana de terminal local, vaya a la carpeta raíz del proyecto de C# de muestra. A continuación, vaya a la carpeta **iot-hub\Quickstarts\back-end-application**.

2. Abra el archivo **BackEndApplication.cs** en el editor de texto de su elección.

    Reemplace el valor de la variable `s_connectionString` por la cadena de conexión del servicio que anotó anteriormente. Luego, guarde los cambios en el archivo **BackEndApplication.cs**.

3. En la ventana de terminal local, ejecute los comandos siguientes para instalar las bibliotecas necesarias para la aplicación back-end:

    ```cmd/sh
    dotnet restore
    ```

4. En la ventana de terminal local, ejecute los comandos siguientes para compilar la aplicación back-end y ejecutarla:

    ```cmd/sh
    dotnet run
    ```

    En la siguiente captura de pantalla se muestra la salida en la que la aplicación realiza una llamada de método directo al dispositivo y recibe una confirmación:

    ![Ejecutar la aplicación back-end](./media/quickstart-control-device-dotnet/BackEndApplication.png)

    Después de ejecutar la aplicación back-end, verá un mensaje en la ventana de consola que ejecuta el dispositivo simulado y cambiará la velocidad a la que envía mensajes:

    ![Cambio en el cliente simulado](./media/quickstart-control-device-dotnet/SimulatedDevice-2.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [iot-hub-quickstarts-clean-up-resources](../../includes/iot-hub-quickstarts-clean-up-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha llamado a un método directo en un dispositivo desde una aplicación de back-end y ha respondido a la llamada de método directo en una aplicación de dispositivo simulado.

Para obtener información sobre cómo redirigir mensajes del dispositivo a la nube a diferentes destinos en la nube, continúe con el siguiente tutorial.

> [!div class="nextstepaction"]
> [Tutorial: Enrutar datos de telemetría a distintos puntos de conexión para procesamiento](tutorial-routing.md)
