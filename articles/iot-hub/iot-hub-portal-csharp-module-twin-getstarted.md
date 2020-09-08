---
title: Identidad del módulo y módulo gemelo de Azure IoT Hub (porta y .NET)
description: Aprenda a crear la identidad de módulo y a actualizar módulos gemelos mediante el portal y .NET.
author: robinsh
manager: philmea
ms.author: robinsh
ms.service: iot-hub
services: iot-hub
ms.devlang: csharp
ms.topic: conceptual
ms.date: 08/20/2019
ms.custom: amqp, devx-track-csharp
ms.openlocfilehash: a10c8b21da1e4dc08a186ccb6ba6528b7a655943
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89022078"
---
# <a name="get-started-with-iot-hub-module-identity-and-module-twin-using-the-portal-and-net-device"></a>Introducción a la identidad de módulo y los módulos gemelos de IoT Hub con el portal y un dispositivo .NET

[!INCLUDE [iot-hub-selector-module-twin-getstarted](../../includes/iot-hub-selector-module-twin-getstarted.md)]

> [!NOTE]
> [Las identidades de módulo y los módulos gemelos](iot-hub-devguide-module-twins.md) son similares a la identidad de dispositivo y el dispositivo gemelo de Azure IoT Hub, pero ofrecen granularidad más fina. Aunque la identidad del dispositivo y los dispositivos gemelos de Azure IoT Hub permiten que la aplicación back-end configure un dispositivo y proporcionan visibilidad sobre las condiciones de este, la identidad del módulo y los módulos gemelos proporcionan estas funcionalidades para los componentes individuales de un dispositivo. En los dispositivos compatibles con varios componentes, como dispositivos con sistema operativo o firmware, identidades de módulo y módulos gemelos permiten la configuración individual y condiciones específicas por componente.
>

En este tutorial, aprenderá a:

* Crear una identidad de módulo en el portal.

* Usar el SDK de dispositivo para .NET para actualizar el módulo gemelo desde el dispositivo.

> [!NOTE]
> Para más información acerca de los diversos SDK que puede usar para crear ambas aplicaciones para que se ejecuten en dispositivos y en el back-end de la solución, consulte el artículo de los [SDK de Azure IoT](iot-hub-devguide-sdks.md).
>

## <a name="prerequisites"></a>Prerrequisitos

* Visual Studio.

* Una cuenta de Azure activa. En caso de no tener ninguna, puede crear una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) en tan solo unos minutos.

## <a name="create-a-hub"></a>Crear un concentrador

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-new-device-in-the-hub"></a>Registrar un nuevo dispositivo en el concentrador

[!INCLUDE [iot-hub-include-create-device](../../includes/iot-hub-include-create-device.md)]

## <a name="create-a-module-identity-in-the-portal"></a>Creación de una identidad de módulo en el portal

En una identidad del dispositivo se pueden crear hasta 20 identidades de módulo. Para agregar una identidad, siga estos pasos:

1. Para el dispositivo que creó en la sección anterior, seleccione **Agregar identidad de módulo** para crear su primera identidad de módulo.

1. Escriba el nombre *myFirstModule*. Guarde su identidad de módulo.

    ![Adición de identidad de módulo](./media/iot-hub-portal-csharp-module-twin-getstarted/add-module-identity.png)

    La nueva identidad de módulo aparece en la parte inferior de la pantalla. Selecciónela para ver los detalles de la identidad del módulo.

    ![Visualización de los detalles de la identidad de módulo](./media/iot-hub-portal-csharp-module-twin-getstarted/module-identity-details.png)

Guarde la **cadena de conexión: clave principal**. Se utilizará en la siguiente sección para configurar el módulo en el dispositivo.

## <a name="update-the-module-twin-using-net-device-sdk"></a>Actualización del módulo gemelo con el SDK de dispositivo .NET

Ha creado correctamente la identidad del módulo en la instancia de IoT Hub. Probemos la comunicación con la nube desde el dispositivo simulado. Al crearse una identidad de módulo, se crea también un módulo gemelo en IoT Hub de manera implícita. En esta sección creará una aplicación de consola .NET en el dispositivo simulado que actualiza las propiedades notificadas del módulo gemelo.

### <a name="create-a-visual-studio-project"></a>Creación de un proyecto de Visual Studio

Para crear una aplicación que actualice las propiedades indicadas del módulo gemelo, siga estos pasos:

1. En Visual Studio, seleccione **Crear un proyecto nuevo**, **Aplicación de consola (.NET Framework)** y luego **Siguiente**.

1. En **Configure su nuevo proyecto**, escriba *UpdateModuleTwinReportedProperties* como **Nombre de proyecto**. Seleccione **Crear** para continuar.

    ![Configuración del proyecto de Visual Studio](./media/iot-hub-portal-csharp-module-twin-getstarted/configure-twins-project.png)

### <a name="install-the-latest-azure-iot-hub-net-device-sdk"></a>Instalación del SDK de dispositivo más reciente de .NET de Azure IoT Hub

La identidad de módulo y el módulo gemelo se encuentran en versión preliminar pública. Solo está disponible en los SDK de dispositivo de la versión preliminar de IoT Hub. Para instalarlo, siga estos pasos:

1. En Visual Studio, abra **Herramientas** > **Administrador de paquetes NuGet** > **Administrar paquetes NuGet para la solución**.

1. Seleccione **Examinar** y luego seleccione **Incluir versión preliminar**. Busque *Microsoft.Azure.Devices.Client*. Seleccione la versión más reciente e instale.

    ![Instalación de la versión preliminar del SDK de servicio .NET de Azure IoT Hub](./media/iot-hub-csharp-csharp-module-twin-getstarted/install-sdk.png)

    Ahora tendrá acceso a todas las características del módulo.

### <a name="get-your-module-connection-string"></a>Obtención de la cadena de conexión del módulo

Necesita la cadena de conexión del módulo para la aplicación de consola. Siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. Vaya al centro de IoT y seleccione **Dispositivos IoT**. Abra **myFirstDevice** y verá que **myFirstModule** se creó correctamente.

1. Seleccione **myFirstModule** en **Identidades de módulo**. En **Detalles de identidad de módulo**, copie la **cadena de conexión (clave principal)** .

    ![Detalles del módulo en Azure Portal](./media/iot-hub-portal-csharp-module-twin-getstarted/module-identity-details.png)

### <a name="create-updatemoduletwinreportedproperties-console-app"></a>Cree la aplicación de consola UpdateModuleTwinReportedProperties

Para crear la aplicación, siga estos pasos:

1. Agregue las siguientes instrucciones `using` al principio del archivo **Program.cs** :

  ```csharp
  using Microsoft.Azure.Devices.Client;
  using Microsoft.Azure.Devices.Shared;
  using Newtonsoft.Json;
  ```

2. Agregue los campos siguientes a la clase **Program** . Reemplace el valor del marcador de posición por la cadena de conexión del módulo.

  ```csharp
  private const string ModuleConnectionString = "<Your module connection string>";
  private static ModuleClient Client = null;
  ```

3. Agregue el método **OnDesiredPropertyChanged** siguiente a la clase **Program**:

  ```csharp
  private static async Task OnDesiredPropertyChanged(TwinCollection desiredProperties, object userContext)
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

4. Por último, reemplace el método **Main** por el código siguiente:

  ```csharp
  static void Main(string[] args)
  {
      Microsoft.Azure.Devices.Client.TransportType transport = Microsoft.Azure.Devices.Client.TransportType.Amqp;
  
      try
      {
          Client = ModuleClient.CreateFromConnectionString(ModuleConnectionString, transport);
          Client.SetConnectionStatusChangesHandler(ConnectionStatusChangeHandler);
          Client.SetDesiredPropertyUpdateCallbackAsync(OnDesiredPropertyChanged, null).Wait();
  
          Console.WriteLine("Retrieving twin");
          var twinTask = Client.GetTwinAsync();
          twinTask.Wait();
          var twin = twinTask.Result;
          Console.WriteLine(JsonConvert.SerializeObject(twin));
  
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
      Console.ReadKey();
      Client.CloseAsync().Wait();
  }
  
  private static void ConnectionStatusChangeHandler(ConnectionStatus status, ConnectionStatusChangeReason reason)
  {
      Console.WriteLine($"Status {status} changed: {reason}");
  }
  ```
  
  Puede compilar y ejecutar esta aplicación mediante **F5**.

Este ejemplo de código muestra cómo recuperar el módulo gemelo y actualizar las propiedades notificadas con el protocolo AMQP. En la versión preliminar pública solo se admite AMQP para las operaciones de módulos gemelos.

## <a name="next-steps"></a>Pasos siguientes

Para continuar la introducción a IoT Hub y explorar otros escenarios de IoT, consulte:

* [Introducción a la identidad de módulo y los módulos gemelos de IoT Hub con un dispositivo .NET y la copia de seguridad de .NET](iot-hub-csharp-csharp-module-twin-getstarted.md)

* [Introducción a IoT Edge](../iot-edge/tutorial-simulate-device-linux.md)
