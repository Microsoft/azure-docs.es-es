---
title: Introducción a la identidad del módulo y el módulo gemelo de Azure IoT Hub (.NET)
description: Aprenda a crear la identidad de módulo y a actualizar módulos gemelos mediante los SDK de IoT para .NET.
author: chrissie926
ms.service: iot-hub
services: iot-hub
ms.devlang: csharp
ms.topic: conceptual
ms.date: 08/07/2019
ms.author: menchi
ms.custom: amqp, devx-track-csharp
ms.openlocfilehash: e990d122367581c785b411f3f6d3c39b334a14fb
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88998584"
---
# <a name="get-started-with-iot-hub-module-identity-and-module-twin-net"></a>Introducción a la identidad de módulo y a los módulos gemelos de IoT Hub (.NET)

[!INCLUDE [iot-hub-selector-module-twin-getstarted](../../includes/iot-hub-selector-module-twin-getstarted.md)]

> [!NOTE]
> [Las identidades de módulo y los módulos gemelos](iot-hub-devguide-module-twins.md) son similares a la identidad de dispositivo y el dispositivo gemelo de Azure IoT Hub, pero ofrecen granularidad más fina. Aunque la identidad del dispositivo y los dispositivos gemelos de Azure IoT Hub permiten que la aplicación back-end configure un dispositivo y proporcionan visibilidad sobre las condiciones de este, la identidad del módulo y los módulos gemelos proporcionan estas funcionalidades para los componentes individuales de un dispositivo. En los dispositivos compatibles con varios componentes, como dispositivos con sistema operativo o firmware, identidades de módulo y módulos gemelos permiten la configuración individual y condiciones específicas por componente.

Al final de este tutorial tendrá dos aplicaciones de consola de .NET:

* **CreateIdentities**. Esta aplicación crea una identidad del dispositivo, una de módulo y la clave de seguridad asociada para conectar el dispositivo y los clientes del módulo.

* **UpdateModuleTwinReportedProperties**. Esta aplicación envía las propiedades notificadas por el módulo gemelo actualizadas a IoT Hub.

> [!NOTE]
> Para más información acerca de los diversos SDK que puede usar para crear ambas aplicaciones para que se ejecuten en dispositivos y en el back-end de la solución, consulte el artículo de los [SDK de Azure IoT Hub](iot-hub-devguide-sdks.md).

## <a name="prerequisites"></a>Prerrequisitos

* Visual Studio.

* Una cuenta de Azure activa. En caso de no tener ninguna, puede crear una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) en tan solo unos minutos.

## <a name="create-a-hub"></a>Crear un concentrador

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="get-the-iot-hub-connection-string"></a>Obtención de la cadena de conexión de IoT Hub

[!INCLUDE [iot-hub-howto-module-twin-shared-access-policy-text](../../includes/iot-hub-howto-module-twin-shared-access-policy-text.md)]

[!INCLUDE [iot-hub-include-find-registryrw-connection-string](../../includes/iot-hub-include-find-registryrw-connection-string.md)]

[!INCLUDE [iot-hub-get-started-create-module-identity-csharp](../../includes/iot-hub-get-started-create-module-identity-csharp.md)]

## <a name="update-the-module-twin-using-net-device-sdk"></a>Actualización del módulo gemelo con el SDK de dispositivo .NET

En esta sección se crea una aplicación de consola .NET en el dispositivo simulado que actualiza las propiedades notificadas del módulo gemelo.

Antes de comenzar, obtenga la cadena de conexión del módulo. Inicie sesión en [Azure Portal](https://portal.azure.com/). Vaya al centro y seleccione **Dispositivos IoT**. Busque **MyFirstDevice**. Seleccione **myFirstDevice** para abrirlo y, a continuación, seleccione **myFirstModule** para abrirlo. En **Detalles de identidad de módulo**, copie la **Cadena de conexión (clave principal)** cuando sea necesario en el procedimiento siguiente.

   ![Detalles del módulo en Azure Portal](./media/iot-hub-csharp-csharp-module-twin-getstarted/module-identity-detail.png)

1. En Visual Studio, agregue un nuevo proyecto a la solución; para ello, seleccione **Archivo** > **Nuevo** > **Proyecto**. En Crear un proyecto, seleccione **Aplicación de consola (.NET Framework)** y seleccione **Siguiente**.

1. Denomine el proyecto *UpdateModuleTwinReportedProperties*. En **Solución**, seleccione **Agregar a solución**. Asegúrese de que la versión de .NET Framework sea 4.6.1 o una posterior.

    ![Creación de un proyecto de Visual Studio](./media/iot-hub-csharp-csharp-module-twin-getstarted/configure-update-twins-csharp1.png)

1. Seleccione **Crear** para crear el proyecto.

1. En Visual Studio, abra **Herramientas** > **Administrador de paquetes NuGet** > **Administrar paquetes NuGet para la solución**. Seleccione la pestaña **Examinar**.

1. Busque y seleccione **Microsoft. Azure.Devices.Client** y, después, **Instalar**.

    ![Instalación de la versión actual del SDK del servicio .NET de Azure IoT Hub](./media/iot-hub-csharp-csharp-module-twin-getstarted/install-client-sdk.png)

1. Agregue las siguientes instrucciones `using` al principio del archivo **Program.cs** :

    ```csharp
    using Microsoft.Azure.Devices.Client;
    using Microsoft.Azure.Devices.Shared;
    using System.Threading.Tasks;
    using Newtonsoft.Json;
    ```

1. Agregue los campos siguientes a la clase **Program** . Reemplace el valor del marcador de posición por la cadena de conexión del módulo.

    ```csharp
    private const string ModuleConnectionString = "<Your module connection string>";
    private static ModuleClient Client = null;
    static void ConnectionStatusChangeHandler(ConnectionStatus status, 
      ConnectionStatusChangeReason reason)
    {
        Console.WriteLine("Connection Status Changed to {0}; the reason is {1}", 
          status, reason);
    }
    ```

1. Agregue el método **OnDesiredPropertyChanged** siguiente a la clase **Program**:

    ```csharp
    private static async Task OnDesiredPropertyChanged(TwinCollection desiredProperties, 
      object userContext)
        {
            Console.WriteLine("desired property change:");
            Console.WriteLine(JsonConvert.SerializeObject(desiredProperties));
            Console.WriteLine("Sending current time as reported property");
            TwinCollection reportedProperties = new TwinCollection
            {
                ["DateTimeLastDesiredPropertyChangeReceived"] = DateTime.Now
            };

            await Client.UpdateReportedPropertiesAsync(reportedProperties).ConfigureAwait(false);
        }
    ```

1. Agregue las líneas siguientes al método **Main**:

    ```csharp
    static void Main(string[] args)
    {
        Microsoft.Azure.Devices.Client.TransportType transport = 
          Microsoft.Azure.Devices.Client.TransportType.Amqp;

        try
        {
            Client = 
              ModuleClient.CreateFromConnectionString(ModuleConnectionString, transport);
            Client.SetConnectionStatusChangesHandler(ConnectionStatusChangeHandler);
            Client.SetDesiredPropertyUpdateCallbackAsync(OnDesiredPropertyChanged, null).Wait();

            Console.WriteLine("Retrieving twin");
            var twinTask = Client.GetTwinAsync();
            twinTask.Wait();
            var twin = twinTask.Result;
            Console.WriteLine(JsonConvert.SerializeObject(twin.Properties)); 

            Console.WriteLine("Sending app start time as reported property");
            TwinCollection reportedProperties = new TwinCollection();
            reportedProperties["DateTimeLastAppLaunch"] = DateTime.Now;

            Client.UpdateReportedPropertiesAsync(reportedProperties);
        }
        catch (AggregateException ex)
        {
            Console.WriteLine("Error in sample: {0}", ex);
        }

        Console.WriteLine("Waiting for Events.  Press enter to exit...");
        Console.ReadLine();
        Client.CloseAsync().Wait();
    }
    ```

    Este ejemplo de código muestra cómo recuperar el módulo gemelo y actualizar las propiedades notificadas con el protocolo AMQP. En la versión preliminar pública solo se admite AMQP para las operaciones de módulos gemelos.

1. Opcionalmente, puede agregar estas instrucciones al método **Main** para enviar un evento al IoT Hub desde el módulo. Coloque estas líneas debajo del bloque `try catch`.

    ```csharp
    Byte[] bytes = new Byte[2];
    bytes[0] = 0;
    bytes[1] = 1;
    var sendEventsTask = Client.SendEventAsync(new Message(bytes));
    sendEventsTask.Wait();
    Console.WriteLine("Event sent to IoT Hub.");
    ```

## <a name="run-the-apps"></a>Ejecución de las aplicaciones

Ahora puede ejecutar las aplicaciones.

1. En Visual Studio, en el **Explorador de soluciones**, haga clic con el botón derecho en la solución y seleccione **Establecer proyectos de inicio**.

1. Seleccione **Propiedades comunes** y **Proyecto de inicio**.

1. Seleccione **Proyectos de inicio múltiples** y después **Iniciar** como acción para la aplicación y **Aceptar** para aceptar los cambios.

1. Presione **F5** para iniciar las aplicaciones.

## <a name="next-steps"></a>Pasos siguientes

Para continuar la introducción a IoT Hub y explorar otros escenarios de IoT, consulte:

* [Introducción a la administración de dispositivos](iot-hub-node-node-device-management-get-started.md)

* [Introducción a IoT Edge](../iot-edge/tutorial-simulate-device-linux.md)
