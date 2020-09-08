---
title: Carga de archivos desde dispositivos a Azure IoT Hub con .NET | Microsoft Docs
description: Cómo cargar archivos de un dispositivo a la nube mediante el SDK de dispositivo IoT de Azure para. NET. Los archivos cargados se almacenan en un contenedor de blobs de Azure Storage.
author: robinsh
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.devlang: csharp
ms.topic: conceptual
ms.date: 07/04/2017
ms.author: robinsh
ms.custom: mqtt, devx-track-csharp
ms.openlocfilehash: ae145f5a1ce462edb28171a1f5bb6f3ebb72f538
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89018219"
---
# <a name="upload-files-from-your-device-to-the-cloud-with-iot-hub-net"></a>Carga de archivos de un dispositivo a la nube con IoT Hub (.NET)

[!INCLUDE [iot-hub-file-upload-language-selector](../../includes/iot-hub-file-upload-language-selector.md)]

Este tutorial se basa en el código del tutorial [Envío de mensajes desde la nube al dispositivo con IoT Hub](iot-hub-csharp-csharp-c2d.md) para mostrarle cómo usar las funcionalidades de carga de archivos de IoT Hub. En él se muestra cómo realizar las siguientes acciones:

* Proporcionar un dispositivo de forma segura con un identificador URI de blob de Azure para cargar un archivo.

* Usar las notificaciones de carga de archivo de IoT Hub para desencadenar el procesamiento del archivo en el back-end de aplicación.

En el inicio rápido [Envío de telemetría de un dispositivo a IoT Hub](quickstart-send-telemetry-dotnet.md) y el tutorial de [Envío de mensajes de la nube al dispositivo con IoT Hub](iot-hub-csharp-csharp-c2d.md) se muestra cómo usar la funcionalidad básica de mensajería del dispositivo a la nube y de la nube al dispositivo de IoT Hub. En el tutorial [Configuración del enrutamiento de mensajes con IoT Hub](tutorial-routing.md) se describe una forma de almacenar de manera confiable los mensajes enviados del dispositivo a la nube en Microsoft Azure Blob Storage. Sin embargo, en algunos casos no se pueden asignar fácilmente los datos que los dispositivos envían en los mensajes de dispositivo a nube con un tamaño relativamente reducido que acepta IoT Hub. Por ejemplo:

* Archivos grandes con imágenes

* Vídeos

* Datos de vibración muestreados con alta frecuencia

* Cierto tipo de datos procesados previamente

Dichos archivos se suelen procesar por lotes en la nube mediante herramientas como [Azure Data Factory](../data-factory/introduction.md) o la pila [Hadoop](../hdinsight/index.yml). Cuando necesite cargar archivos desde un dispositivo, todavía puede usar la seguridad y confiabilidad de IoT Hub.

Al final de este tutorial, ejecutará dos aplicaciones de consola de .NET:

* **SimulatedDevice**. Esta aplicación carga un archivo en el almacenamiento mediante un identificador URI de SAS proporcionado por el centro de IoT. Es una versión modificada de la aplicación creada en el tutorial [Envío de mensajes de la nube al dispositivo con IoT Hub](iot-hub-csharp-csharp-c2d.md).

* **ReadFileUploadNotification**. Esta aplicación recibe notificaciones de carga de archivos de IoT Hub.

> [!NOTE]
> IoT Hub admite muchas plataformas de dispositivos y lenguajes (entre los que se incluyen C, Java, Python y JavaScript), mediante los SDK de dispositivo IoT de Azure. Consulte el [Centro para desarrolladores de IoT de Azure](https://azure.microsoft.com/develop/iot) para obtener instrucciones paso a paso sobre cómo conectar el dispositivo a Azure IoT Hub.

## <a name="prerequisites"></a>Requisitos previos

* Visual Studio

* Una cuenta de Azure activa. En caso de no tener ninguna, puede crear una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) en tan solo unos minutos.

* Asegúrese de que el puerto 8883 está abierto en el firewall. En el ejemplo de dispositivo de este artículo se usa el protocolo MQTT, que se comunica mediante el puerto 8883. Este puerto puede estar bloqueado en algunos entornos de red corporativos y educativos. Para más información y saber cómo solucionar este problema, consulte [Conexión a IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

[!INCLUDE [iot-hub-associate-storage](../../includes/iot-hub-associate-storage.md)]

## <a name="upload-a-file-from-a-device-app"></a>Carga de un archivo desde una aplicación de dispositivo

En esta sección, modificará la aplicación de dispositivo que creó en [Envío de mensajes de la nube al dispositivo con IoT Hub](iot-hub-csharp-csharp-c2d.md) para recibir mensajes de la nube al dispositivo de IoT Hub.

1. En el Explorador de soluciones de Visual Studio, haga clic con el botón derecho en el proyecto **SimulatedDevice** y seleccione **Agregar** > **Elemento existente**. Encuentre un archivo de imagen e inclúyalo en su proyecto. En este tutorial se supone que la imagen se llama `image.jpg`.

1. Haga clic con el botón derecho en la imagen y seleccione **Propiedades**. Asegúrese de que **Copiar en el directorio de salida** está establecido en **Copiar siempre**.

    ![Muestra dónde se actualiza la propiedad de imagen para Copiar en el directorio de salida](./media/iot-hub-csharp-csharp-file-upload/image-properties.png)

1. En el archivo **Program.cs** , agregue las siguientes instrucciones al principio del archivo:

    ```csharp
    using System.IO;
    ```

1. Agregue el método siguiente a la clase **Program** :

    ```csharp
    private static async void SendToBlobAsync()
    {
        string fileName = "image.jpg";
        Console.WriteLine("Uploading file: {0}", fileName);
        var watch = System.Diagnostics.Stopwatch.StartNew();

        using (var sourceData = new FileStream(@"image.jpg", FileMode.Open))
        {
            await deviceClient.UploadToBlobAsync(fileName, sourceData);
        }

        watch.Stop();
        Console.WriteLine("Time to upload file: {0}ms\n", watch.ElapsedMilliseconds);
    }
    ```

    El método `UploadToBlobAsync` toma el nombre de archivo y el origen de streaming del archivo que se va a cargar y administra la carga en el almacenamiento. La aplicación de consola muestra el tiempo que se tarda en cargar el archivo.

1. Agregue la siguiente línea en el método **Main**, justo antes de `Console.ReadLine()`:

    ```csharp
    SendToBlobAsync();
    ```

> [!NOTE]
> Por simplificar, este tutorial no implementa ninguna directiva de reintentos. En el código de producción, deberá implementar directivas de reintentos (por ejemplo, retroceso exponencial), tal y como se sugiere en el artículo [Control de errores transitorios](/azure/architecture/best-practices/transient-faults).

## <a name="get-the-iot-hub-connection-string"></a>Obtención de la cadena de conexión de IoT Hub

En este artículo, creará un servicio de back-end para recibir mensajes de notificación de carga de archivos desde la instancia de IoT Hub que creó en [Envío de telemetría de un dispositivo a IoT Hub](quickstart-send-telemetry-dotnet.md). Para recibir mensajes de notificación de carga de archivos, el servicio necesita el permiso de **conexión de servicio**. De forma predeterminada, todas las instancias de IoT Hub se crean con una directiva de acceso compartido denominada **servicio** que concede este permiso.

[!INCLUDE [iot-hub-include-find-service-connection-string](../../includes/iot-hub-include-find-service-connection-string.md)]

## <a name="receive-a-file-upload-notification"></a>Recepción de una notificación de carga de archivos

En esta sección, se escribe una aplicación de consola de .NET que recibe mensajes de notificación de carga de archivos de IoT Hub.

1. En la solución actual de Visual Studio, seleccione **Archivo** > **Nuevo** > **Proyecto**. En **Crear un proyecto**, seleccione **Aplicación de consola (.NET Framework)** y seleccione **Siguiente**.

1. Denomine al proyecto *ReadFileUploadNotification*. En **Solución**, seleccione **Agregar a solución**. Seleccione **Crear** para crear el proyecto.

    ![Configuración del proyecto ReadFileUploadNotification en Visual Studio](./media/iot-hub-csharp-csharp-file-upload/read-file-upload-project-configure.png)

1. En el Explorador de soluciones, haga clic con el botón derecho en el proyecto **ReadFileUploadNotification** y, luego, haga clic en **Administrar paquetes NuGet...** .

1. En **Administrador de paquetes NuGet**, seleccione **Examinar**. Busque y seleccione **Microsoft.Azure.Devices.Client** y, después, **Instalar**.

    Este paso descarga, instala y agrega una referencia al [paquete NuGet del SDK de servicios IoT de Azure](https://www.nuget.org/packages/Microsoft.Azure.Devices/) en el proyecto **ReadFileUploadNotification**.

1. En el archivo **Program.cs** de este proyecto, agregue la siguiente instrucción al principio del archivo:

    ```csharp
    using Microsoft.Azure.Devices;
    ```

1. Agregue los campos siguientes a la clase **Program** . Reemplace el valor de marcador de posición `{iot hub connection string}` por la cadena de conexión de IoT Hub que copió anteriormente en [Obtener la cadena de conexión de IoT Hub](#get-the-iot-hub-connection-string).

    ```csharp
    static ServiceClient serviceClient;
    static string connectionString = "{iot hub connection string}";
    ```

1. Agregue el método siguiente a la clase **Program** :

    ```csharp
    private async static void ReceiveFileUploadNotificationAsync()
    {
        var notificationReceiver = serviceClient.GetFileNotificationReceiver();

        Console.WriteLine("\nReceiving file upload notification from service");
        while (true)
        {
            var fileUploadNotification = await notificationReceiver.ReceiveAsync();
            if (fileUploadNotification == null) continue;

            Console.ForegroundColor = ConsoleColor.Yellow;
            Console.WriteLine("Received file upload notification: {0}", 
              string.Join(", ", fileUploadNotification.BlobName));
            Console.ResetColor();

            await notificationReceiver.CompleteAsync(fileUploadNotification);
        }
    }
    ```

    Tenga en cuenta que este patrón de recepción es el mismo que se usa para recibir mensajes de nube a dispositivo desde la aplicación de dispositivo.

1. Por último, agregue las líneas siguientes al método **Main** :

    ```csharp
    Console.WriteLine("Receive file upload notifications\n");
    serviceClient = ServiceClient.CreateFromConnectionString(connectionString);
    ReceiveFileUploadNotificationAsync();
    Console.WriteLine("Press Enter to exit\n");
    Console.ReadLine();
    ```

## <a name="run-the-applications"></a>Ejecución de las aplicaciones

Ahora está preparado para ejecutar las aplicaciones.

1. En el Explorador de soluciones, haga clic con el botón derecho en la solución y seleccione **Establecer proyectos de inicio**.

1. En **Propiedades comunes** > **Proyecto de inicio**, seleccione **Varios proyectos de inicio** y seleccione la acción **Inicio** para **ReadFileUploadNotification** y **SimulatedDevice**. Seleccione **Aceptar** para guardar los cambios.

1. Presione **F5**. Deberían iniciarse las dos aplicaciones. Debería ver que se ha completado la carga en una aplicación de consola y que la otra aplicación de consola ha recibido el mensaje de notificación de carga. Puede usar [Azure Portal](https://portal.azure.com/) o el Explorador de servidores de Visual Studio para comprobar que el archivo cargado está en la cuenta de Azure Storage.

    ![Captura de pantalla que muestra la pantalla de salida](./media/iot-hub-csharp-csharp-file-upload/run-apps1.png)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha aprendido a usar la funcionalidad de carga de archivos de IoT Hub para simplificar la carga de archivos desde los dispositivos. Consulte los siguientes artículos para seguir explorando las características y los escenarios de IoT Hub:

* [Creación de un centro de IoT mediante programación](iot-hub-rm-template-powershell.md)

* [Introducción a SDK para C](iot-hub-device-sdk-c-intro.md)

* [SDK de IoT de Azure](iot-hub-devguide-sdks.md)

Para explorar aún más las funcionalidades de IoT Hub, consulte:

* [Implementación de IA en dispositivos perimetrales con Azure IoT Edge](../iot-edge/tutorial-simulate-device-linux.md)
