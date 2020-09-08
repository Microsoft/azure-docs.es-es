---
title: Tutorial para la seguridad de X.509 en Azure IoT Hub | Microsoft Docs
description: Introducción a la seguridad basada en X.509 en Azure IoT Hub en un entorno simulado.
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/20/2019
ms.custom:
- amqp
- 'Role: Cloud Development'
- 'Role: IoT Device'
- devx-track-csharp
ms.openlocfilehash: 1c1e29d43fefa6be1586aefc28249fe2f0f9e9fb
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89001032"
---
# <a name="set-up-x509-security-in-your-azure-iot-hub"></a>Configuración de la seguridad de X.509 en Azure IoT Hub

Este tutorial muestra los pasos necesarios para proteger Azure IoT Hub mediante la *autenticación de certificado X.509*. Con fines ilustrativos, usaremos la herramienta de código abierto OpenSSL para crear certificados localmente en una máquina Windows. Se recomienda usar este tutorial solo para la realización de pruebas. Para el entorno de producción debe comprar los certificados a una *entidad de certificación (CA) raíz*.

## <a name="prerequisites"></a>Prerrequisitos

Este tutorial requiere que tenga listos los siguientes recursos:

* Ha creado una instancia de IoT Hub con su suscripción de Azure. Para ver los pasos con detalle, consulte [Creación de una instancia de IoT Hub mediante Azure Portal](iot-hub-create-through-portal.md).

* Tener instalado [Visual Studio 2017 o Visual Studio 2019](https://www.visualstudio.com/vs/).

## <a name="get-x509-ca-certificates"></a>Obtención de certificados de entidad de certificación X.509

La seguridad basada en certificados X.509 de IoT Hub requiere que comience con un [cadena de certificados X.509](https://en.wikipedia.org/wiki/X.509#Certificate_chains_and_cross-certification), que incluya el certificado raíz, así como cualquier certificado intermedio hasta el certificado de hoja.

Puede elegir cualquiera de los siguientes métodos para obtener los certificados:

* Comprar certificados X.509 a una *entidad de certificación (CA) raíz*. Este método se recomienda para entornos de producción.

* Crear sus propios certificados X.509 mediante una herramienta de terceros como [OpenSSL](https://www.openssl.org/). Esta técnica es apropiada tanto para pruebas como para desarrollo. Consulte [Managing test CA certificates for samples and tutorials](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md) (Administración de certificados de entidad de certificación para ejemplos y tutoriales) para información sobre la generación de certificados de entidad de certificación mediante PowerShell o Bash. En el resto de este tutorial se usan certificados de entidad de certificación generados siguiendo las instrucciones de [Managing test CA certificates for samples and tutorials](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md) (Administración de certificados de entidad de certificación de prueba para ejemplos y tutoriales).

* Genere un [certificado de entidad de certificación intermedio X.509](iot-hub-x509ca-overview.md#sign-devices-into-the-certificate-chain-of-trust) firmado por un certificado de entidad de certificación raíz existente y cárguelo en el centro. Una vez que el certificado intermedio se haya cargado y comprobado, como se indica a continuación, se puede usar en lugar de un certificado de entidad de certificación raíz mencionado a continuación. Se pueden usar herramientas como OpenSSL ([openssl req](https://www.openssl.org/docs/man1.1.0/man1/req.html) y [openssl ca](https://www.openssl.org/docs/man1.1.0/man1/ca.html)) para generar y firmar un certificado de entidad de certificación intermedio.

> [!NOTE]
> No cargue la raíz de terceros si no es única, ya que eso permitiría a otros clientes de terceros conectar sus dispositivos a su IoT Hub.

## <a name="register-x509-ca-certificates-to-your-iot-hub"></a>Registro de certificados de entidad de certificación X.509 en una instancia de IoT Hub

Estos pasos muestran cómo agregar una entidad de certificación nueva a una instancia de IoT Hub a través del portal.

> [!NOTE]
> El número máximo de certificados de entidad de certificación X.509 que se pueden registrar en una instancia de IoT Hub es de 25. Para más información, consulte [Cuotas y limitaciones de Azure IoT Hub](iot-hub-devguide-quotas-throttling.md).

1. En Azure Portal, navegue a su instancia de IoT Hub y seleccione **Configuración** > **Certificados** para el centro.

1. Seleccione **Agregar** para agregar un certificado nuevo.

1. En **Nombre del certificado**, escriba un nombre para mostrar descriptivo y seleccione en el equipo el archivo de certificado que creó en la sección anterior.

1. Una vez que reciba la notificación de que el certificado se ha cargado correctamente, seleccione **Guardar**.

    ![Carga del certificado](./media/iot-hub-security-x509-get-started/iot-hub-add-cert.png)  

   El certificado aparecerá en la lista de certificados con el estado **Sin comprobar**.

1. Seleccione el certificado que acaba de agregar para mostrar **Detalles del certificado** y, a continuación, seleccione **Generar código de verificación**.

   ![Comprobar certificado](./media/iot-hub-security-x509-get-started/copy-verification-code.png)  

1. Copie el **código de verificación** en el portapapeles. Este código se usa para validar la propiedad del certificado.

1. Siga el paso 3 descrito en [Administración de certificados de entidad de certificación de prueba para ejemplos y tutoriales](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md).  Este proceso firma el código de verificación con la clave privada asociada con el certificado de entidad de certificación X.509, lo que genera una firma. Hay herramientas disponibles para realizar este proceso de firma, como por ejemplo, OpenSSL. Este proceso se conoce como la [prueba de posesión](https://tools.ietf.org/html/rfc5280#section-3.1).

1. En **Detalles del certificado**, en el **archivo de certificado de verificación .pem o .cer**, busque y abra el archivo de firma. A continuación, seleccione **Comprobar**.

   El estado del certificado cambia a **Comprobado**. Seleccione **Actualizar** si el certificado no se actualiza automáticamente.

## <a name="create-an-x509-device-for-your-iot-hub"></a>Creación de un dispositivo X.509 para una instancia de IoT Hub

1. En Azure Portal, vaya a la página de su instancia de IoT Hub y seleccione **Exploradores** > **Dispositivos IoT**.

1. Seleccione **Nuevo** para agregar un dispositivo nuevo.

1. En **Id. de dispositivo**, escriba un nombre para mostrar descriptivo. En **Tipo de autenticación**, seleccione **X.509 firmado por CA** y, a continuación, seleccione **Guardar**.

   ![Crear un dispositivo X.509 en el portal](./media/iot-hub-security-x509-get-started/new-x509-device.png)

## <a name="authenticate-your-x509-device-with-the-x509-certificates"></a>Autenticación de un dispositivo X.509 con los certificados X.509

Para autenticar un dispositivo X.509, en primer lugar hay que firmarlo con el certificado de entidad de certificación. La firma de dispositivos de hoja se suele realizar en la fábrica, donde se han habilitado las herramientas de fabricación pertinentes. Cuando el dispositivo pasa de un fabricante a otro, la acción de firma de cada fabricante se captura como un certificado intermedio en la cadena. El resultado es una cadena de certificados del certificado de la entidad de certificación al certificado de hoja del dispositivo. En el paso 4 de [Managing test CA certificates for samples and tutorials](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md) (Administración de certificados de entidad de certificación de prueba para ejemplos y tutoriales) se genera un certificado de dispositivo.

A continuación, se mostrará cómo crear una aplicación de C# para simular el dispositivo X.509 registrado en su instancia de IoT Hub. Se enviarán los valores de temperatura y humedad del dispositivo simulado al centro. En este tutorial, solo se va a crear la aplicación del dispositivo. La creación de la aplicación del servicio IoT Hub de centro de IoT que enviará una respuesta a los eventos enviados por este dispositivo simulado queda como ejercicio para los lectores. La aplicación de C# da por hecho que ha seguido los pasos descritos en [Managing test CA certificates for samples and tutorials](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md) (Administración de certificados de entidad de certificación de prueba para ejemplos y tutoriales).

1. Abra Visual Studio, seleccione **Crear un proyecto nuevo** y luego seleccione la plantilla de proyecto **Aplicación de consola (.NET Framework)** . Seleccione **Next** (Siguiente).

1. En **Configure su nuevo proyecto**, asigne al proyecto el nombre *SimulateX509Device* y luego seleccione **Crear**.

   ![Creación del proyecto de dispositivo X.509 en Visual Studio](./media/iot-hub-security-x509-get-started/create-device-project-vs2019.png)

1. En el Explorador de soluciones, haga clic con el botón derecho en el proyecto **SimulateX509Device** y seleccione **Administrar paquetes NuGet**.

1. En **Administrador de paquetes NuGet**, seleccione **Examinar** y busque y elija **Microsoft.Azure.Devices.Client**. Seleccione **Instalar**.

   ![Adición del paquete NuGet del SDK de dispositivo en Visual Studio](./media/iot-hub-security-x509-get-started/device-sdk-nuget.png)

    Este paso permite descargar, instalar y agregar una referencia al paquete NuGet del SDK de dispositivo IoT de Azure y sus dependencias.

1. Agregue las siguientes instrucciones `using` al principio del archivo **Program.cs** :

    ```csharp
        using Microsoft.Azure.Devices.Client;
        using Microsoft.Azure.Devices.Shared;
        using System.Security.Cryptography.X509Certificates;
    ```

1. Agregue los campos siguientes a la clase **Program**:

    ```csharp
        private static int MESSAGE_COUNT = 5;
        private const int TEMPERATURE_THRESHOLD = 30;
        private static String deviceId = "<your-device-id>";
        private static float temperature;
        private static float humidity;
        private static Random rnd = new Random();
    ```

    En lugar de _<your_device_id>_ , utilice el nombre de dispositivo descriptivo que usó en la sección anterior.

1. Agregue la siguiente función para crear números aleatorios de temperatura y humedad, y enviar dichos valores al centro:

    ```csharp
    static async Task SendEvent(DeviceClient deviceClient)
    {
        string dataBuffer;
        Console.WriteLine("Device sending {0} messages to IoTHub...\n", MESSAGE_COUNT);

        for (int count = 0; count < MESSAGE_COUNT; count++)
        {
            temperature = rnd.Next(20, 35);
            humidity = rnd.Next(60, 80);
            dataBuffer = string.Format("{{\"deviceId\":\"{0}\",\"messageId\":{1},\"temperature\":{2},\"humidity\":{3}}}", deviceId, count, temperature, humidity);
            Message eventMessage = new Message(Encoding.UTF8.GetBytes(dataBuffer));
            eventMessage.Properties.Add("temperatureAlert", (temperature > TEMPERATURE_THRESHOLD) ? "true" : "false");
            Console.WriteLine("\t{0}> Sending message: {1}, Data: [{2}]", DateTime.Now.ToLocalTime(), count, dataBuffer);

            await deviceClient.SendEventAsync(eventMessage);
        }
    }
    ```

1. Por último, agregue las siguientes líneas de código a la función **Main**, pero reemplace los marcadores de posición _device-id_, _your-iot-hub-name_ y _absolute-path-to-your-device-pfx-file_ en función de lo que solicite el programa de instalación.

    ```csharp
    try
    {
        var cert = new X509Certificate2(@"<absolute-path-to-your-device-pfx-file>", "1234");
        var auth = new DeviceAuthenticationWithX509Certificate("<device-id>", cert);
        var deviceClient = DeviceClient.Create("<your-iot-hub-name>.azure-devices.net", auth, TransportType.Amqp_Tcp_Only);

        if (deviceClient == null)
        {
            Console.WriteLine("Failed to create DeviceClient!");
        }
        else
        {
            Console.WriteLine("Successfully created DeviceClient!");
            SendEvent(deviceClient).Wait();
        }

        Console.WriteLine("Exiting...\n");
    }
    catch (Exception ex)
    {
        Console.WriteLine("Error in sample: {0}", ex.Message);
    }
    ```

   Este código se conecta a su instancia de IoT Hub mediante la creación de la cadena de conexión para el dispositivo X.509. Una vez que se ha conectado correctamente, envía eventos de temperatura y humedad al centro y espera su respuesta.

1. Ejecute la aplicación. Dado que esta aplicación accede a un archivo *.pfx*, puede que deba ejecutarla en modo de administrador.

   1. Compile la solución de Visual Studio.

   1. Abra una nueva ventana del símbolo del sistema mediante **Ejecutar como administrador**.  

   1. Navegue hasta la carpeta que contiene la solución y vaya hasta la ruta de acceso *bin/Debug* dentro de la carpeta de la solución.

   1. Ejecute la aplicación **SimulateX509Device.exe** desde el símbolo del sistema.

   Debería ver que el dispositivo se conecta correctamente al centro y envía los eventos.

   ![Ejecución de la aplicación de dispositivo](./media/iot-hub-security-x509-get-started/device-app-success.png)

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre cómo proteger la solución de IoT, consulte:

* [Procedimientos recomendados de seguridad de IoT](../iot-fundamentals/iot-security-best-practices.md)

* [Arquitectura de seguridad de IoT](../iot-fundamentals/iot-security-architecture.md)

* [Protección de su implementación de IoT](../iot-fundamentals/iot-security-deployment.md)

Para explorar aún más las funcionalidades de IoT Hub, consulte:

* [Implementación de IA en dispositivos perimetrales con Azure IoT Edge](../iot-edge/tutorial-simulate-device-linux.md)
