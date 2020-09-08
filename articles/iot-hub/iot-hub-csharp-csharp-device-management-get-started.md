---
title: Introducción a la administración de dispositivos de Azure IoT Hub (.NET/.NET) | Microsoft Docs
description: Describe cómo usar la administración de dispositivos de IoT Hub de Azure para iniciar un reinicio del dispositivo remoto. Usará el SDK de dispositivo IoT de Azure para .NET con el fin de implementar una aplicación de dispositivo simulado que incluye un método directo, además del SDK de servicio IoT de Azure para .NET con el objetivo de implementar una aplicación de servicio que invoque el método directo.
author: robinsh
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.devlang: csharp
ms.topic: conceptual
ms.date: 08/20/2019
ms.author: robinsh
ms.custom: mqtt, devx-track-csharp
ms.openlocfilehash: 5760c574e64a3b3b4a1df12092cff44374790a90
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89018406"
---
# <a name="get-started-with-device-management-net"></a>Introducción a la administración de dispositivos (.NET)

[!INCLUDE [iot-hub-selector-dm-getstarted](../../includes/iot-hub-selector-dm-getstarted.md)]

En este tutorial se muestra cómo realizar las siguientes acciones:

* Usar Azure Portal para crear un centro de IoT y una identidad de dispositivo en este.

* Crear una aplicación de dispositivo simulado que contiene un método directo que reinicia ese dispositivo. Los métodos directos se invocan desde la nube.

* Crear una aplicación de consola de .NET que llame a un método directo de reinicio en la aplicación de dispositivo simulado mediante su centro de IoT Hub.

Al final de este tutorial tendrá dos aplicaciones de consola de .NET:

* **SimulateManagedDevice**. Esta aplicación se conecta al centro de IoT con la identidad del dispositivo creada anteriormente, recibe un método directo de reinicio, simula un reinicio físico y notifica la hora del último reinicio.

* **TriggerReboot**. Esta aplicación llama a un método directo en la aplicación de dispositivo simulado, muestra la respuesta y muestra las propiedades notificadas actualizadas.

## <a name="prerequisites"></a>Requisitos previos

* Visual Studio.

* Una cuenta de Azure activa. En caso de no tener ninguna, puede crear una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) en tan solo unos minutos.

* Asegúrese de que el puerto 8883 está abierto en el firewall. En el ejemplo de dispositivo de este artículo se usa el protocolo MQTT, que se comunica mediante el puerto 8883. Este puerto puede estar bloqueado en algunos entornos de red corporativos y educativos. Para más información y saber cómo solucionar este problema, consulte [Conexión a IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="create-an-iot-hub"></a>Crear un centro de IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-new-device-in-the-iot-hub"></a>Registro de un nuevo dispositivo en el centro de IoT

[!INCLUDE [iot-hub-include-create-device](../../includes/iot-hub-include-create-device.md)]

## <a name="get-the-iot-hub-connection-string"></a>Obtención de la cadena de conexión de IoT Hub

[!INCLUDE [iot-hub-howto-device-management-shared-access-policy-text](../../includes/iot-hub-howto-device-management-shared-access-policy-text.md)]

[!INCLUDE [iot-hub-include-find-service-connection-string](../../includes/iot-hub-include-find-service-connection-string.md)]

## <a name="trigger-a-remote-reboot-on-the-device-using-a-direct-method"></a>Desencadenar un reinicio remoto en el dispositivo con un método directo

En esta sección, creará una aplicación de consola de .NET, mediante C#, que inicia una actualización remota en un dispositivo con un método directo. La aplicación usa las consultas gemelas de dispositivo para detectar la hora en que se reinició por última vez el dispositivo.

1. En Visual Studio, seleccione **Crear un proyecto nuevo**.

1. En **Crear un proyecto nuevo**, busque y seleccione la plantilla de proyecto **Aplicación de consola (.NET Framework)** y, después, seleccione **Siguiente**.

1. En **Configure su nuevo proyecto**, asigne al proyecto el nombre *TriggerReboot* y seleccione .NET Framework versión 4.5.1 o posterior. Seleccione **Crear**.

    ![Nuevo proyecto de escritorio clásico de Windows de Visual C#](./media/iot-hub-csharp-csharp-device-management-get-started/create-trigger-reboot-configure.png)

1. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto **TriggerReboot** y seleccione **Administrar paquetes NuGet**.

1. Seleccione **Examinar** y busque y seleccione **Microsoft. Azure.Devices**. Seleccione **Instalar** para instalar el paquete **Microsoft.Azure.Devices**.

    ![Ventana del Administrador de paquetes NuGet](./media/iot-hub-csharp-csharp-device-management-get-started/create-trigger-reboot-nuget-devices.png)

   Este paso permite descargar, instalar y agregar una referencia al paquete NuGet del [SDK de servicio IoT de Azure](https://www.nuget.org/packages/Microsoft.Azure.Devices/) y sus dependencias.

1. Agregue las siguientes instrucciones `using` al principio del archivo **Program.cs** :

   ```csharp
   using Microsoft.Azure.Devices;
   using Microsoft.Azure.Devices.Shared;
   ```

1. Agregue los campos siguientes a la clase **Program** . Reemplace el valor del marcador de posición `{iot hub connection string}` por la cadena de conexión del centro de IoT que copió anteriormente en [Obtención de la cadena de conexión de IoT Hub](#get-the-iot-hub-connection-string).

   ```csharp
   static RegistryManager registryManager;
   static string connString = "{iot hub connection string}";
   static ServiceClient client;
   static string targetDevice = "myDeviceId";
   ```

1. Agregue el método siguiente a la clase **Program**.  Este código obtiene el dispositivo gemelo para el dispositivo de reinicio y genera las propiedades notificadas.

   ```csharp
   public static async Task QueryTwinRebootReported()
   {
       Twin twin = await registryManager.GetTwinAsync(targetDevice);
       Console.WriteLine(twin.Properties.Reported.ToJson());
   }
   ```

1. Agregue el método siguiente a la clase **Program**.  Este código inicia el reinicio en el dispositivo mediante un método directo.

   ```csharp
   public static async Task StartReboot()
   {
       client = ServiceClient.CreateFromConnectionString(connString);
       CloudToDeviceMethod method = new CloudToDeviceMethod("reboot");
       method.ResponseTimeout = TimeSpan.FromSeconds(30);

       CloudToDeviceMethodResult result = await 
         client.InvokeDeviceMethodAsync(targetDevice, method);

       Console.WriteLine("Invoked firmware update on device.");
   }
   ```

1. Por último, agregue las líneas siguientes al método **Main** :

   ```csharp
   registryManager = RegistryManager.CreateFromConnectionString(connString);
   StartReboot().Wait();
   QueryTwinRebootReported().Wait();
   Console.WriteLine("Press ENTER to exit.");
   Console.ReadLine();
   ```

1. Seleccione **Compilar** > **Compilar solución**.

> [!NOTE]
> Este tutorial realiza solo una consulta individual de las propiedades notificadas del dispositivo. En el código de producción, se recomienda realizar un sondeo detectar cambios en las propiedades notificadas.

## <a name="create-a-simulated-device-app"></a>Creación de una aplicación de dispositivo simulado

En esta sección:

* Creará una aplicación de consola de .NET que responda a un método directo que se llama desde la nube.

* Desencadenará un reinicio del dispositivo simulado.

* Usará las propiedades notificadas para permitir consultas de dispositivo gemelo a fin de identificar los dispositivos y cuándo se reiniciaron por última vez.

Para crear la aplicación de dispositivo simulado, siga estos pasos:

1. En Visual Studio, en la solución TriggerReboot que ya ha creado, seleccione **Archivo** > **Nuevo** > **Proyecto**. En **Crear un proyecto nuevo**, busque y seleccione la plantilla de proyecto **Aplicación de consola (.NET Framework)** y, después, seleccione **Siguiente**.

1. En **Configure su nuevo proyecto**, asigne al proyecto el nombre *SimulateManagedDeviceel* y, en **Solución**, seleccione **Agregar a la solución**. Seleccione **Crear**.

    ![Asignar un nombre al proyecto y agregarlo a la solución](./media/iot-hub-csharp-csharp-device-management-get-started/configure-device-app.png)

1. En el Explorador de soluciones, haga clic con el botón derecho en el nuevo proyecto **SimulateManagedDevice** y seleccione **Administrar paquetes NuGet**.

1. Seleccione **Examinar** y busque y seleccione **Microsoft. Azure.Devices.Client**. Seleccione **Instalar**.

    ![Ventana del Administrador de paquetes NuGet: aplicación cliente](./media/iot-hub-csharp-csharp-device-management-get-started/create-device-nuget-devices-client.png)

   Este paso permite descargar, instalar y agregar una referencia al paquete NuGet del [SDK de dispositivo IoT de Azure](https://www.nuget.org/packages/Microsoft.Azure.Devices.Client/) y sus dependencias.

1. Agregue las siguientes instrucciones `using` al principio del archivo **Program.cs** :

    ```csharp
    using Microsoft.Azure.Devices.Client;
    using Microsoft.Azure.Devices.Shared;
    ```

1. Agregue los campos siguientes a la clase **Program** . Reemplace el valor del marcador de posición `{device connection string}` por la cadena de conexión de dispositivo que anotó anteriormente en [Registro de un nuevo dispositivo en el centro de IoT](#register-a-new-device-in-the-iot-hub).

    ```csharp
    static string DeviceConnectionString = "{device connection string}";
    static DeviceClient Client = null;
    ```

1. Agregue lo siguiente para implementar el método directo en el dispositivo:

   ```csharp
   static Task<MethodResponse> onReboot(MethodRequest methodRequest, object userContext)
   {
       // In a production device, you would trigger a reboot 
       //   scheduled to start after this method returns.
       // For this sample, we simulate the reboot by writing to the console
       //   and updating the reported properties.
       try
       {
           Console.WriteLine("Rebooting!");

           // Update device twin with reboot time. 
           TwinCollection reportedProperties, reboot, lastReboot;
           lastReboot = new TwinCollection();
           reboot = new TwinCollection();
           reportedProperties = new TwinCollection();
           lastReboot["lastReboot"] = DateTime.Now;
           reboot["reboot"] = lastReboot;
           reportedProperties["iothubDM"] = reboot;
           Client.UpdateReportedPropertiesAsync(reportedProperties).Wait();
       }
       catch (Exception ex)
       {
           Console.WriteLine();
           Console.WriteLine("Error in sample: {0}", ex.Message);
       }

       string result = @"{""result"":""Reboot started.""}";
       return Task.FromResult(new MethodResponse(Encoding.UTF8.GetBytes(result), 200));
   }
   ```

1. Finalmente, agregue el código siguiente al método **Main** para abrir la conexión a su IoT Hub e inicializar la escucha del método directo:

   ```csharp
   try
   {
       Console.WriteLine("Connecting to hub");
       Client = DeviceClient.CreateFromConnectionString(DeviceConnectionString, 
         TransportType.Mqtt);

       // setup callback for "reboot" method
       Client.SetMethodHandlerAsync("reboot", onReboot, null).Wait();
       Console.WriteLine("Waiting for reboot method\n Press enter to exit.");
       Console.ReadLine();

       Console.WriteLine("Exiting...");

       // as a good practice, remove the "reboot" handler
       Client.SetMethodHandlerAsync("reboot", null, null).Wait();
       Client.CloseAsync().Wait();
   }
   catch (Exception ex)
   {
       Console.WriteLine();
       Console.WriteLine("Error in sample: {0}", ex.Message);
   }
   ```

1. En el Explorador de soluciones, haga clic con el botón derecho en su solución y seleccione **Establecer proyectos de inicio**.

1. En **Propiedades comunes** > **Proyecto de inicio**, seleccione **Proyecto de inicio único** y, después, seleccione el proyecto **SimulateManagedDevice**. Seleccione **Aceptar** para guardar los cambios.

1. Seleccione **Compilar** > **Compilar solución**.

> [!NOTE]
> Por simplificar, este tutorial no implementa ninguna directiva de reintentos. En el código de producción, deberá implementar directivas de reintentos (por ejemplo, retroceso exponencial), tal y como se sugiere en [Control de errores transitorios](/azure/architecture/best-practices/transient-faults).

## <a name="run-the-apps"></a>Ejecución de las aplicaciones

Ya está preparado para ejecutar las aplicaciones.

1. Para ejecutar la aplicación de dispositivo .NET **SimulateManagedDevice**, en el Explorador de soluciones, haga clic con el botón derecho en**SimulateManagedDevice**, seleccione **Depurar** y, después, seleccione **Iniciar nueva instancia**. La aplicación debería empezar a escuchar llamadas de método desde el centro de IoT.

1. Una vez que el dispositivo esté conectado y esperando invocaciones de método, haga clic con el botón derecho en **TriggerReboot**, seleccione **Depurar** y, después, **Iniciar nueva instancia**.

   Debería ver el mensaje "Reiniciando" escrito en la consola de **SimulatedManagedDevice** y las propiedades notificadas del dispositivo, que incluyen la hora en que se reinició por última vez, escritas en la consola de **TriggerReboot**.

    ![Ejecución de servicio y de aplicación de dispositivo](./media/iot-hub-csharp-csharp-device-management-get-started/combinedrun.png)

[!INCLUDE [iot-hub-dm-followup](../../includes/iot-hub-dm-followup.md)]
