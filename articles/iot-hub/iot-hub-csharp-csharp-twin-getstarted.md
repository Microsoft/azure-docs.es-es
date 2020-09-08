---
title: Introducción a los dispositivos gemelos de Azure IoT Hub (.NET/.NET) | Microsoft Docs
description: Describe cómo usar dispositivos gemelos de IoT Hub de Azure para agregar etiquetas y, luego, usar una consulta de IoT Hub. Usará el SDK de dispositivo Azure IoT para .NET con el fin de implementar una aplicación para dispositivo simulado, además del SDK de servicios Azure IoT para .NET con el objetivo de implementar una aplicación de servicio que agrega las etiquetas y ejecuta la consulta de IoT Hub.
author: robinsh
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.devlang: csharp
ms.topic: conceptual
ms.date: 08/26/2019
ms.author: robinsh
ms.custom: mqtt, devx-track-csharp
ms.openlocfilehash: 267a69486dc91ef95c0de704346eeb1d1780ef48
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89013765"
---
# <a name="get-started-with-device-twins-net"></a>Introducción a los dispositivos gemelos (.NET)

[!INCLUDE [iot-hub-selector-twin-get-started](../../includes/iot-hub-selector-twin-get-started.md)]

En este tutorial, creará estas aplicaciones de consola de .NET:

* **AddTagsAndQuery**. Esta aplicación de back-end agrega dispositivos gemelos de consultas y etiquetas.

* **ReportConnectivity**. Esta aplicación para dispositivo simula un dispositivo que se conecta a su centro de IoT con la identidad del dispositivo creada anteriormente e informa de su estado de conectividad.

> [!NOTE]
> En el artículo [SDK de IoT de Azure](iot-hub-devguide-sdks.md) se proporciona información sobre los SDK de IoT de Azure que se pueden usar para crear aplicaciones de dispositivo y de back-end.
>

## <a name="prerequisites"></a>Requisitos previos

* Visual Studio.

* Una cuenta de Azure activa. En caso de no tener ninguna, puede crear una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) en tan solo unos minutos.

* Asegúrese de que el puerto 8883 está abierto en el firewall. En el ejemplo de dispositivo de este artículo se usa el protocolo MQTT, que se comunica mediante el puerto 8883. Este puerto puede estar bloqueado en algunos entornos de red corporativos y educativos. Para más información y saber cómo solucionar este problema, consulte [Conexión a IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="create-an-iot-hub"></a>Crear un centro de IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-new-device-in-the-iot-hub"></a>Registro de un nuevo dispositivo en el centro de IoT

[!INCLUDE [iot-hub-include-create-device](../../includes/iot-hub-include-create-device.md)]

## <a name="get-the-iot-hub-connection-string"></a>Obtención de la cadena de conexión de IoT Hub

[!INCLUDE [iot-hub-howto-twin-shared-access-policy-text](../../includes/iot-hub-howto-twin-shared-access-policy-text.md)]

[!INCLUDE [iot-hub-include-find-custom-connection-string](../../includes/iot-hub-include-find-custom-connection-string.md)]

## <a name="create-the-service-app"></a>Creación de la aplicación de servicio

En esta sección, creará una aplicación de consola de .NET que usa C# y agrega metadatos de ubicación al dispositivo gemelo asociado con **myDeviceId**. A continuación, consulta los dispositivos gemelos almacenados en IoT Hub seleccionando los dispositivos ubicados en Estados Unidos y, después, los que informan de una conexión de red de telefonía móvil.

1. En Visual Studio, seleccione **Crear un proyecto nuevo**. En **Crear proyecto**, seleccione **Aplicación de consola (.NET Framework)** y seleccione **Siguiente**.

1. En **Configure su nuevo proyecto**, asigne al proyecto el nombre **AddTagsAndQuery**.

    ![Configuración del proyecto AddTagsAndQuery](./media/iot-hub-csharp-csharp-twin-getstarted/config-addtagsandquery-app.png)

1. En el Explorador de soluciones, haga clic con el botón derecho en el proyecto **AddTagsAndQuery** y luego seleccione **Administrar paquetes NuGet**.

1. Seleccione **Examinar** y busque y seleccione **Microsoft.Azure.Devices**. Seleccione **Instalar**.

    ![Ventana del Administrador de paquetes NuGet](./media/iot-hub-csharp-csharp-twin-getstarted/nuget-package-addtagsandquery-app.png)

   Este paso permite descargar, instalar y agregar una referencia al paquete NuGet del [SDK de servicio IoT de Azure](https://www.nuget.org/packages/Microsoft.Azure.Devices/) y sus dependencias.

1. Agregue las siguientes instrucciones `using` al principio del archivo **Program.cs** :

    ```csharp  
    using Microsoft.Azure.Devices;
    ```

1. Agregue los campos siguientes a la clase **Program** . Reemplace `{iot hub connection string}` por la cadena de conexión de IoT Hub que copió en [Obtención de la cadena de conexión de IoT Hub](#get-the-iot-hub-connection-string).

    ```csharp  
    static RegistryManager registryManager;
    static string connectionString = "{iot hub connection string}";
    ```

1. Agregue el método siguiente a la clase **Program** :

    ```csharp  
    public static async Task AddTagsAndQuery()
    {
        var twin = await registryManager.GetTwinAsync("myDeviceId");
        var patch =
            @"{
                tags: {
                    location: {
                        region: 'US',
                        plant: 'Redmond43'
                    }
                }
            }";
        await registryManager.UpdateTwinAsync(twin.DeviceId, patch, twin.ETag);

        var query = registryManager.CreateQuery(
          "SELECT * FROM devices WHERE tags.location.plant = 'Redmond43'", 100);
        var twinsInRedmond43 = await query.GetNextAsTwinAsync();
        Console.WriteLine("Devices in Redmond43: {0}", 
          string.Join(", ", twinsInRedmond43.Select(t => t.DeviceId)));

        query = registryManager.CreateQuery("SELECT * FROM devices WHERE tags.location.plant = 'Redmond43' AND properties.reported.connectivity.type = 'cellular'", 100);
        var twinsInRedmond43UsingCellular = await query.GetNextAsTwinAsync();
        Console.WriteLine("Devices in Redmond43 using cellular network: {0}", 
          string.Join(", ", twinsInRedmond43UsingCellular.Select(t => t.DeviceId)));
    }
    ```

    La clase **RegistryManager** expone todos los métodos necesarios para interactuar con dispositivos gemelos del servicio. El código anterior inicializa primero el objeto **registryManager**, luego recupera el dispositivo gemelo de **myDeviceId** y, por último, actualiza sus etiquetas con la información de la ubicación deseada.

    Después de la actualización, ejecuta dos consultas: la primera selecciona solo los dispositivos gemelos que se encuentran en la planta **Redmond43** y la segunda mejora la consulta para seleccionar solo los dispositivos que están también conectados a través de la red de telefonía móvil.

    El código anterior, cuando crea el objeto de **consulta**, especifica un número máximo de documentos devueltos. El objeto **consulta** contiene una propiedad booleana **HasMoreResults** que puede utilizar para invocar a los métodos **GetNextAsTwinAsync** varias veces para recuperar todos los resultados. Un método llamado **GetNextAsJson** está disponible para los resultados que no son dispositivos gemelos, por ejemplo, los resultados de consultas de agregación.

1. Por último, agregue las líneas siguientes al método **Main** :

    ```csharp  
    registryManager = RegistryManager.CreateFromConnectionString(connectionString);
    AddTagsAndQuery().Wait();
    Console.WriteLine("Press Enter to exit.");
    Console.ReadLine();
    ```

1. Ejecute esta aplicación haciendo clic con el botón derecho en el proyecto **AddTagsAndQuery** y seleccionando **Depurar**, seguido de **Iniciar nueva instancia**. Debería ver un dispositivo en los resultados de la consulta que pregunta por todos los dispositivos que se encuentran en **Redmond43** y ninguno para la consulta que restringe los resultados a los dispositivos que utilizan una red de telefonía móvil.

    ![Resultados de consulta en ventana](./media/iot-hub-csharp-csharp-twin-getstarted/addtagapp.png)

En la siguiente sección, creará una aplicación de dispositivo que notifica la información de conectividad y cambia el resultado de la consulta en la sección anterior.

## <a name="create-the-device-app"></a>Creación de la aplicación del dispositivo

En esta sección, creará una aplicación de consola de .NET que se conecta al centro como **myDeviceId** y, luego, actualiza las propiedades notificadas para contener la información que indica que está conectado mediante una red de telefonía móvil.

1. Abra Visual Studio, seleccione **Archivo** > **Nuevo** > **Proyecto**. En **Crear proyecto**, elija **Aplicación de consola (.NET Framework)** y seleccione **Siguiente**.

1. En **Configure su nuevo proyecto**, asigne al proyecto el nombre **ReportConnectivity**. En **Solución**, elija **Agregar a solución** y seleccione **Crear**.

1. En el Explorador de soluciones, haga clic con el botón derecho en el proyecto **ReportConnectivity** y seleccione **Administrar paquetes NuGet**.

1. Seleccione **Examinar** y busque y elija **Microsoft.Azure.Devices.Client**. Seleccione **Instalar**.

   Este paso permite descargar, instalar y agregar una referencia al paquete NuGet del [SDK de dispositivo IoT de Azure](https://www.nuget.org/packages/Microsoft.Azure.Devices.Client/) y sus dependencias.

1. Agregue las siguientes instrucciones `using` al principio del archivo **Program.cs** :

    ```csharp  
    using Microsoft.Azure.Devices.Client;
    using Microsoft.Azure.Devices.Shared;
    using Newtonsoft.Json;
    ```

1. Agregue los campos siguientes a la clase **Program** . Reemplace `{device connection string}` por la cadena de conexión de dispositivo que anotó en [Registro de un nuevo dispositivo en el centro de IoT](#register-a-new-device-in-the-iot-hub).

    ```csharp  
    static string DeviceConnectionString = "HostName=<yourIotHubName>.azure-devices.net;DeviceId=<yourIotDeviceName>;SharedAccessKey=<yourIotDeviceAccessKey>";
    static DeviceClient Client = null;
    ```

1. Agregue el método siguiente a la clase **Program** :

    ```csharp
    public static async void InitClient()
    {
        try
        {
            Console.WriteLine("Connecting to hub");
            Client = DeviceClient.CreateFromConnectionString(DeviceConnectionString, 
              TransportType.Mqtt);
            Console.WriteLine("Retrieving twin");
            await Client.GetTwinAsync();
        }
        catch (Exception ex)
        {
            Console.WriteLine();
            Console.WriteLine("Error in sample: {0}", ex.Message);
        }
    }
    ```

    El objeto **Cliente** expone todos los métodos necesarios para interactuar con dispositivos gemelos del dispositivo. El código mostrado anteriormente, inicializa el objeto **Client** y, luego, recupera el dispositivo gemelo para **myDeviceId**.

1. Agregue el método siguiente a la clase **Program** :

    ```csharp  
    public static async void ReportConnectivity()
    {
        try
        {
            Console.WriteLine("Sending connectivity data as reported property");

            TwinCollection reportedProperties, connectivity;
            reportedProperties = new TwinCollection();
            connectivity = new TwinCollection();
            connectivity["type"] = "cellular";
            reportedProperties["connectivity"] = connectivity;
            await Client.UpdateReportedPropertiesAsync(reportedProperties);
        }
        catch (Exception ex)
        {
            Console.WriteLine();
            Console.WriteLine("Error in sample: {0}", ex.Message);
        }
    }
    ```

   El código anterior actualiza la propiedad notificada de **myDeviceId** con la información de conectividad.

1. Por último, agregue las líneas siguientes al método **Main** :

    ```csharp
    try
    {
        InitClient();
        ReportConnectivity();
    }
    catch (Exception ex)
    {
        Console.WriteLine();
        Console.WriteLine("Error in sample: {0}", ex.Message);
    }
    Console.WriteLine("Press Enter to exit.");
    Console.ReadLine();
    ```

1. En el Explorador de soluciones, haga clic con el botón derecho en la solución y seleccione **Establecer proyectos de inicio**.

1. En **Propiedades comunes** > **Proyecto de inicio**, seleccione **Proyectos de inicio múltiples**. En **ReportConnectivity**, seleccione **Iniciar** como **Acción**. Seleccione **Aceptar** para guardar los cambios.  

1. Ejecute esta aplicación haciendo clic con el botón derecho en el proyecto **ReportConnectivity** y seleccionando **Depurar**, seguido de **Iniciar nueva instancia**. Debería ver cómo obtiene la información del gemelo y, luego, envía la conectividad como una ***propiedad notificada***.

    ![Ejecución la aplicación para dispositivo para notificar sobre la conectividad](./media/iot-hub-csharp-csharp-twin-getstarted/rundeviceapp.png)

   Una vez que el dispositivo ha informado sobre su información de conectividad, debe aparecer en ambas consultas.

1. Haga clic con el botón derecho en el proyecto **AddTagsAndQuery** y seleccione **Depurar** > **Iniciar nueva instancia** para volver a ejecutar las consultas. Esta vez **myDeviceId** debe aparecer en los resultados de ambas consultas.

    ![Conectividad de dispositivo notificada correctamente](./media/iot-hub-csharp-csharp-twin-getstarted/tagappsuccess.png)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, configuró un centro de IoT nuevo en Azure Portal y, después, creó una identidad de dispositivo en el registro de identidades del centro de IoT. Ha agregado metadatos de dispositivo como etiquetas desde una aplicación back-end y ha escrito una aplicación de dispositivo simulado para notificar la información de conectividad de dispositivos en el dispositivo gemelo. También ha aprendido cómo consultar esta información mediante el lenguaje de consulta de IoT Hub de tipo SQL.

Puede obtener más información en estos recursos:

* Para aprender a enviar telemetría desde dispositivos, consulte el tutorial [Envío de telemetría desde un dispositivo a IoT Hub](quickstart-send-telemetry-dotnet.md).

* Para aprender a configurar dispositivos mediante las propiedades deseadas del dispositivo gemelo, consulte el tutorial [Uso de las propiedades deseadas para configurar dispositivos](tutorial-device-twins.md).

* Para aprender a controlar los dispositivos de forma interactiva, por ejemplo, encender un ventilador desde una aplicación controlada por el usuario, consulte el tutorial [Uso de métodos directos](quickstart-control-device-dotnet.md).
