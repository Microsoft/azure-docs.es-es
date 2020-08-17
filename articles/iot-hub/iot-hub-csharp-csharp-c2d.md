---
title: Mensajes de nube a dispositivo con IoT Hub de Azure (.NET) | Microsoft Docs
description: Cómo enviar mensajes de nube a un dispositivo de una instancia de IoT Hub de Azure mediante los SDK de IoT de Azure para .NET. Modifique una aplicación de dispositivo para recibir mensajes de nube a dispositivo y cambie una aplicación de back-end para enviar los mensajes de nube a dispositivo.
author: robinsh
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.devlang: csharp
ms.topic: conceptual
ms.date: 07/07/2020
ms.author: robinsh
ms.custom:
- amqp
- mqtt
- 'Role: Cloud Development'
- 'Role: IoT Device'
ms.openlocfilehash: 0b4fbe4ab7accec18f48e6319e16d10f083a8709
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87327453"
---
# <a name="send-messages-from-the-cloud-to-your-device-with-iot-hub-net"></a>Envío de mensajes desde la nube al dispositivo con IoT Hub (.NET)

[!INCLUDE [iot-hub-selector-c2d](../../includes/iot-hub-selector-c2d.md)]

IoT Hub de Azure es un servicio totalmente administrado que permite la comunicación bidireccional confiable y segura entre millones de dispositivos y una solución de back-end. En el inicio rápido [Envío de telemetría desde un dispositivo a un centro de IoT](quickstart-send-telemetry-dotnet.md) se muestra cómo crear un centro de IoT, aprovisionar en él la identidad del dispositivo y codificar una aplicación de dispositivo que envíe mensajes del dispositivo a la nube.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

Este tutorial se basa en el artículo [Envío de telemetría desde un dispositivo a un centro de IoT](quickstart-send-telemetry-dotnet.md). Muestra cómo completar las tareas siguientes:

* Desde el back-end de la nube de la aplicación, envíe mensajes de nube a dispositivo en un único dispositivo a través de IoT Hub.

* Reciba mensajes de nube a dispositivo en un dispositivo.

* Desde la solución de back-end, solicite confirmación de entrega (*comentarios*) para los mensajes enviados a un dispositivo desde IoT Hub.

Encontrará más información sobre los mensajes de nube a dispositivo en [Mensajería de dispositivo a nube y de nube a dispositivo con IoT Hub](iot-hub-devguide-messaging.md).

Al final de este tutorial, ejecutará dos aplicaciones de consola de .NET.

* **SimulatedDevice**. Esta aplicación se conecta al centro de IoT y recibe mensajes de la nube al dispositivo. Esta aplicación es una versión modificada de la aplicación creada en [Envío de telemetría desde un dispositivo a un centro de IoT](quickstart-send-telemetry-dotnet.md).

* **SendCloudToDevice**. Esta aplicación envía un mensaje de la nube al dispositivo para la aplicación de dispositivo mediante IoT Hub y, luego, recibe su confirmación de entrega.

> [!NOTE]
> IoT Hub ofrece compatibilidad con SDK en muchas plataformas de dispositivos y lenguajes, entre los que se incluyen C, Java, Python y Javascript, mediante los [SDK de dispositivo IoT de Azure](iot-hub-devguide-sdks.md). Para obtener instrucciones paso a paso sobre cómo conectar el dispositivo al código de este tutorial y, en general a Azure IoT Hub consulte la [Guía para desarrolladores de IoT Hub](iot-hub-devguide.md).
>

## <a name="prerequisites"></a>Requisitos previos

* Visual Studio

* Una cuenta de Azure activa. En caso de no tener ninguna, puede crear una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) en tan solo unos minutos.

* Asegúrese de que el puerto 8883 está abierto en el firewall. En el ejemplo de dispositivo de este artículo se usa el protocolo MQTT, que se comunica mediante el puerto 8883. Este puerto puede estar bloqueado en algunos entornos de red corporativos y educativos. Para más información y saber cómo solucionar este problema, consulte el artículo sobre la [conexión a IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="receive-messages-in-the-device-app"></a>Recepción de mensajes en la aplicación de dispositivo

En esta sección, modificará la aplicación de dispositivo que creó en [Enviar datos de telemetría desde un dispositivo a un centro de IoT](quickstart-send-telemetry-dotnet.md) para recibir mensajes de la nube al dispositivo desde IoT Hub.

1. En el proyecto **SimulatedDevice** de Visual Studio, agregue el método siguiente a la clase de **SimulatedDevice**.

   ```csharp
    private static async void ReceiveC2dAsync()
    {
        Console.WriteLine("\nReceiving cloud to device messages from service");
        while (true)
        {
            Message receivedMessage = await s_deviceClient.ReceiveAsync();
            if (receivedMessage == null) continue;

            Console.ForegroundColor = ConsoleColor.Yellow;
            Console.WriteLine("Received message: {0}", 
            Encoding.ASCII.GetString(receivedMessage.GetBytes()));
            Console.ResetColor();

            await s_deviceClient.CompleteAsync(receivedMessage);
        }
    }
   ```

1. Agregue el siguiente método en el método **Main**, inmediatamente delante de la línea `Console.ReadLine()`:

   ```csharp
   ReceiveC2dAsync();
   ```

El método `ReceiveAsync` devuelve de forma asincrónica el mensaje recibido en el momento en que lo recibe el dispositivo. Devuelve *null* después de un período de tiempo de espera especificable. En este ejemplo, se usa el valor predeterminado de un minuto. Cuando la aplicación recibe un valor *null*, debe continuar esperando nuevos mensajes. Este requisito es el motivo de la línea `if (receivedMessage == null) continue`.

La llamada a `CompleteAsync()` notifica al Centro de IoT que el mensaje se ha procesado correctamente. Los mensajes pueden quitarse con seguridad de la cola del dispositivo. Si ha sucedido algo que ha impedido que la aplicación del dispositivo termine de procesar el mensaje, el Centro de IoT lo entrega de nuevo. La lógica de procesamiento de mensajes de la aplicación del dispositivo debe ser *idempotente*, de modo que, si se recibe el mismo mensaje varias veces, se genera el mismo resultado.

Una aplicación también puede abandonar temporalmente un mensaje, lo que da lugar a que el Centro de IoT retenga el mensaje en la cola para su consumo futuro. O bien, la aplicación puede rechazar un mensaje, de forma que este se quita permanentemente de la cola. Para más información sobre el ciclo de vida de los mensajes de nube a dispositivo, consulte [Mensajería de dispositivo a nube y de nube a dispositivo con IoT Hub](iot-hub-devguide-messaging.md).

   > [!NOTE]
   > Cuando se usa HTTP en lugar de MQTT o AMQP como transporte, el método `ReceiveAsync` se devuelve inmediatamente. El patrón admitido para los mensajes de nube a dispositivo con HTTP es dispositivos conectados de forma intermitente que buscan mensajes con poca frecuencia (menos de cada 25 minutos). Emitir más recepciones HTTP tendrá como resultado la limitación de solicitudes de IoT Hub. Para más información sobre las diferencias entre la compatibilidad con MQTT, AMQP y HTTPS, y la limitación de IoT Hub, consulte [Mensajería de dispositivo a nube y de nube a dispositivo con IoT Hub](iot-hub-devguide-messaging.md).
   >

## <a name="get-the-iot-hub-connection-string"></a>Obtención de la cadena de conexión de IoT Hub

En este artículo, se crea un servicio de back-end para enviar mensajes de la nube al dispositivo mediante el centro de IoT que creó en [Envío de telemetría desde un dispositivo a IoT Hub](quickstart-send-telemetry-dotnet.md). Para enviar mensajes de nube a un dispositivo, el servicio necesita el permiso de **conexión de servicio**. De forma predeterminada, todas las instancias de IoT Hub se crean con una directiva de acceso compartido denominada **servicio** que concede este permiso.

[!INCLUDE [iot-hub-include-find-service-connection-string](../../includes/iot-hub-include-find-service-connection-string.md)]

## <a name="send-a-cloud-to-device-message"></a>Envío de mensajes de nube a dispositivo

En esta sección, escribirá una aplicación de consola de .NET que envíe mensajes de nube a dispositivo a la aplicación del dispositivo simulado.

1. En la solución actual de Visual Studio, seleccione **Archivo** > **Nuevo** > **Proyecto**. En **Crear un proyecto**, seleccione **Aplicación de consola (.NET Framework)** y seleccione **Siguiente**.

1. Denomine el proyecto *SendCloudToDevice*. En **Solución**, seleccione **Agregar a la solución** y acepte la versión más reciente de .NET Framework. Seleccione **Crear** para crear el proyecto.

   ![Configuración de un proyecto nuevo en Visual Studio](./media/iot-hub-csharp-csharp-c2d/sendcloudtodevice-project-configure.png)

1. En el Explorador de soluciones, haga clic con el botón derecho en el nuevo proyecto y, después, seleccione **Administrar paquetes NuGet**.

1. En **Administrar paquetes NuGet**, seleccione **Examinar** y busque y seleccione **Microsoft.Azure.Devices**. Seleccione **Instalar**.

   En ese paso, se descarga, instala y agrega una referencia al [paquete NuGet del SDK de Servicios IoT de Azure](https://www.nuget.org/packages/Microsoft.Azure.Devices/).

1. Agregue la instrucción `using` en la parte superior del archivo **Program.cs**.

   ``` csharp
   using Microsoft.Azure.Devices;
   ```

1. Agregue los campos siguientes a la clase **Program** . Reemplace el valor del marcador de posición `{iot hub connection string}` por la cadena de conexión del centro de IoT que tomó anteriormente de [Obtención de la cadena de conexión del centro de IoT](#get-the-iot-hub-connection-string). Sustituya el marcador de posición `{device id}` por la id. del dispositivo que agregó en el inicio rápido de [Envío de telemetría desde un dispositivo a un centro de IoT](quickstart-send-telemetry-dotnet.md).

   ``` csharp
   static ServiceClient serviceClient;
   static string connectionString = "{iot hub connection string}";
   static string targetDevice = "{device id}";
   ```

1. Agregue el método siguiente a la clase de **Programa** para enviar un mensaje al dispositivo.

   ``` csharp
   private async static Task SendCloudToDeviceMessageAsync()
   {
        var commandMessage = new
         Message(Encoding.ASCII.GetBytes("Cloud to device message."));
        await serviceClient.SendAsync(targetDevice, commandMessage);
   }
   ```

1. Finalmente, agregue las líneas siguientes al método **Main** .

   ``` csharp
   Console.WriteLine("Send Cloud-to-Device message\n");
   serviceClient = ServiceClient.CreateFromConnectionString(connectionString);

   Console.WriteLine("Press any key to send a C2D message.");
   Console.ReadLine();
   SendCloudToDeviceMessageAsync().Wait();
   Console.ReadLine();
   ```

1. En el Explorador de soluciones, haga clic con el botón derecho en la solución y seleccione **Establecer proyectos de inicio**.

1. En **Propiedades comunes** > **Proyecto de inicio**, seleccione **Varios proyectos de inicio** y, a continuación, seleccione la acción **Iniciar** para **SimulatedDevice** y **SendCloudToDevice**. Seleccione **Aceptar** para guardar los cambios.

1. Presione **F5**. Deberían iniciarse las dos aplicaciones. Seleccione la ventana **SendCloudToDevice** y presione **Entrar**. Debería ver el mensaje que recibe la aplicación de dispositivo.

   ![Aplicación que recibe el mensaje](./media/iot-hub-csharp-csharp-c2d/sendc2d1.png)

## <a name="receive-delivery-feedback"></a>Recepción de comentarios de entrega

Es posible solicitar confirmaciones de entrega (o expiración) del Centro de IoT para cada mensaje de nube a dispositivo. Esta opción permite que el back-end de solución notifique fácilmente la lógica de reintento o compensación. Para más información sobre los comentarios de nube a dispositivo, consulte [Mensajería de dispositivo a nube y de nube a dispositivo con IoT Hub](iot-hub-devguide-messaging.md).

En esta sección, modificará la aplicación **SendCloudToDevice** para solicitar comentarios y recibirlos de IoT Hub.

1. En Visual Studio, en el proyecto **SendCloudToDevice**, agregue el método siguiente a la clase **Program**.

   ```csharp
   private async static void ReceiveFeedbackAsync()
   {
        var feedbackReceiver = serviceClient.GetFeedbackReceiver();

        Console.WriteLine("\nReceiving c2d feedback from service");
        while (true)
        {
            var feedbackBatch = await feedbackReceiver.ReceiveAsync();
            if (feedbackBatch == null) continue;

            Console.ForegroundColor = ConsoleColor.Yellow;
            Console.WriteLine("Received feedback: {0}",
              string.Join(", ", feedbackBatch.Records.Select(f => f.StatusCode)));
            Console.ResetColor();

            await feedbackReceiver.CompleteAsync(feedbackBatch);
        }
    }
    ```

    Tenga en cuenta que este patrón de recepción es el mismo que se usa para recibir mensajes de nube a dispositivo desde la aplicación de dispositivo.

1. Agregue la línea siguiente en el método **Main** inmediatamente después de `serviceClient = ServiceClient.CreateFromConnectionString(connectionString)`.

   ``` csharp
   ReceiveFeedbackAsync();
   ```

1. Para solicitar comentarios de la entrega del mensaje de la nube a un dispositivo, debe especificar una propiedad en el método **SendCloudToDeviceMessageAsync** . Agregue la línea siguiente, inmediatamente después de la línea `var commandMessage = new Message(...);`.

   ``` csharp
   commandMessage.Ack = DeliveryAcknowledgement.Full;
   ```

1. Ejecute las aplicaciones presionando **F5**. Debería ver que se inician ambas aplicaciones. Seleccione la ventana **SendCloudToDevice** y presione **Entrar**. Verá los mensajes que recibe la aplicación de dispositivo y, al cabo de unos segundos, el mensaje de comentarios que recibe la aplicación **SendCloudToDevice**.

   ![Aplicación que recibe el mensaje](./media/iot-hub-csharp-csharp-c2d/sendc2d2.png)

> [!NOTE]
> Por simplificar, este tutorial no implementa ninguna directiva de reintentos. En el código de producción, deberá implementar directivas de reintentos (por ejemplo, retroceso exponencial), tal y como se sugiere en el artículo [Control de errores transitorios](/azure/architecture/best-practices/transient-faults).
>

## <a name="next-steps"></a>Pasos siguientes

En este procedimiento, ha aprendido a enviar y recibir mensajes de nube a dispositivo.

Para ver ejemplos de soluciones de un extremo a otro que usen IoT Hub, vea el [Acelerador de la solución de supervisión remota de Azure IoT](https://docs.microsoft.com/azure/iot-suite/).

Para obtener más información sobre cómo desarrollar soluciones con IoT Hub, consulte la [Guía para desarrolladores de IoT Hub](iot-hub-devguide.md).
