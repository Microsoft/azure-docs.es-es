---
title: 'Tutorial: Aprovisionamiento de un dispositivo mediante el servicio Azure IoT Hub Device Provisioning (.NET)'
description: En este tutorial se indica cómo aprovisionar el dispositivo en un único centro de IoT mediante Azure IoT Hub Device Provisioning Service (DPS) con .NET.
author: wesmc7777
ms.author: wesmc
ms.date: 11/12/2019
ms.topic: tutorial
ms.service: iot-dps
services: iot-dps
ms.devlang: csharp
ms.custom: mvc, devx-track-csharp
ms.openlocfilehash: 55195949cfaa741389f38deaea69806c568c0ce6
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89008274"
---
# <a name="tutorial-enroll-the-device-to-an-iot-hub-using-the-azure-iot-hub-provisioning-service-client-net"></a>Tutorial: Inscripción del dispositivo en una instancia de IoT Hub mediante el cliente del servicio Azure IoT Hub Device Provisioning (.NET)

En el tutorial anterior, aprendió a configurar un dispositivo para conectarse al servicio Device Provisioning. En este tutorial, aprenderá a usar este servicio para aprovisionar el dispositivo en una única instancia de IoT Hub mediante la **_inscripción individual_** y los **_grupos de inscripción_** . En este tutorial se muestra cómo realizar las siguientes acciones:

> [!div class="checklist"]
> * Inscribir el dispositivo
> * Iniciar el dispositivo
> * Comprobar que el dispositivo está registrado

## <a name="prerequisites"></a>Prerrequisitos

Antes de continuar, asegúrese de configurar el dispositivo y su *módulo de seguridad de hardware* como se describe en el tutorial [Configuración del dispositivo para el aprovisionamiento mediante el servicio Azure IoT Hub Device Provisioning](./tutorial-set-up-device.md).

* Visual Studio

> [!NOTE]
> Visual Studio no es obligatorio. La instalación de [.NET](https://www.microsoft.com/net) es suficiente y los desarrolladores pueden utilizar su editor preferido en Windows o Linux.  

Este tutorial simula el período durante el proceso de fabricación del hardware (o justo después de esto) cuando se agrega la información del dispositivo al servicio de aprovisionamiento. Normalmente, este código se ejecuta en un equipo o un dispositivo de fábrica que puede ejecutar código .NET y no se debe agregar a los propios dispositivos.


## <a name="enroll-the-device"></a>Inscribir el dispositivo

Este paso incluye la incorporación de artefactos de seguridad únicos del dispositivo al servicio Device Provisioning. Estos artefactos de seguridad son los siguientes:

- Para dispositivos basados en TPM:
    - La *clave de aprobación* que es única para cada chip de TPM o simulación. Consulte [Descripción de la clave de aprobación de TPM](https://technet.microsoft.com/library/cc770443.aspx) para más información.
    - El *identificador de registro* que se usa para identificar de forma única un dispositivo en el espacio de nombres o el ámbito. Este puede ser igual que el identificador de dispositivo o no. El identificador es obligatorio para todos los dispositivos. En el caso de los dispositivos basados en TPM, el identificador de registro puede derivarse del propio TPM, por ejemplo, un hash SHA-256 de la clave de aprobación del TPM.

- Para dispositivos basados en X.509:
    - El [certificado X.509 emitido para el dispositivo](https://msdn.microsoft.com/library/windows/desktop/bb540819.aspx), en un formato de archivo *.pem* o *.cer*. Para la inscripción individual, debe usar el *certificado de hoja* para su sistema X.509, mientras que para los grupos de inscripción, debe usar el *certificado raíz* o un *certificado del firmante*.
    - El *identificador de registro* que se usa para identificar de forma única un dispositivo en el espacio de nombres o el ámbito. Este puede ser igual que el identificador de dispositivo o no. El identificador es obligatorio para todos los dispositivos. Para los dispositivos basados en X.509, el identificador de registro se deriva del nombre común del certificado (CN). Para más información sobre estos requisitos consulte [Conceptos de dispositivo](https://docs.microsoft.com/azure/iot-dps/concepts-device).

Hay dos maneras de inscribir el dispositivo en el servicio Device Provisioning:

- **Inscripciones individuales**: esta representa una entrada de un único dispositivo que se puede registrar con el servicio Device Provisioning. Las inscripciones individuales pueden usar certificados X.509 o tokens de SAS (en un módulo de plataforma segura real o virtual) como mecanismos de atestación. Se recomiendan las inscripciones individuales para los dispositivos que requieran configuraciones iniciales únicas o para los dispositivos que solo se puedan utilizar tokens de SAS a través de TPM como mecanismo de atestación. En las inscripciones individuales se puede especificar el identificador de dispositivo del centro de IoT deseado.

- **Grupos de inscripción**: esta representa un grupo de dispositivos que comparten un mecanismo de atestación específico. Se recomienda usar un grupo de inscripción para un gran número de dispositivos que compartan la configuración inicial deseada o para dispositivos que vayan todos al mismo inquilino. Los grupos de inscripción solo son X.509 y todos comparten un certificado de firma de su cadena de certificados X.509.

### <a name="enroll-the-device-using-individual-enrollments"></a>Inscripción del dispositivo mediante las inscripciones individuales

1. En Visual Studio, cree un nuevo proyecto de aplicación de consola de Visual C# con la plantilla de proyecto **Aplicación de consola**. Llame al proyecto **DeviceProvisioning**.
    
1. En el Explorador de soluciones, haga clic con el botón derecho en el proyecto **DeviceProvisioning** y luego haga clic en **Administrar paquetes NuGet...** .

1. En la ventana **Administrador de paquetes NuGet**, seleccione **Examinar** y busque **microsoft.azure.devices.provisioning.service**. Seleccione la entrada y haga clic en **Instalar** para instalar el paquete **Microsoft.Azure.Devices.Provisioning.Service** y acepte las condiciones de uso. Este procedimiento descarga, instala y agrega una referencia al paquete NuGet del [SDK de Azure IoT Device Provisioning Service](https://www.nuget.org/packages/Microsoft.Azure.Devices.Provisioning.Service/) y a sus dependencias.

1. Agregue las siguientes instrucciones `using` al principio del archivo **Program.cs** :
   
    ```csharp
    using Microsoft.Azure.Devices.Provisioning.Service;
    ```

1. Agregue los campos siguientes a la clase **Program** . Sustituya el valor del marcador de posición por la cadena de conexión de Device Provisioning Service que anotó en la sección anterior.
   
    ```csharp
    static readonly string ServiceConnectionString = "{Device Provisioning Service connection string}";

    private const string SampleRegistrationId = "sample-individual-csharp";
    private const string SampleTpmEndorsementKey =
            "AToAAQALAAMAsgAgg3GXZ0SEs/gakMyNRqXXJP1S124GUgtk8qHaGzMUaaoABgCAAEMAEAgAAAAAAAEAxsj2gUS" +
            "cTk1UjuioeTlfGYZrrimExB+bScH75adUMRIi2UOMxG1kw4y+9RW/IVoMl4e620VxZad0ARX2gUqVjYO7KPVt3d" +
            "yKhZS3dkcvfBisBhP1XH9B33VqHG9SHnbnQXdBUaCgKAfxome8UmBKfe+naTsE5fkvjb/do3/dD6l4sGBwFCnKR" +
            "dln4XpM03zLpoHFao8zOwt8l/uP3qUIxmCYv9A7m69Ms+5/pCkTu/rK4mRDsfhZ0QLfbzVI6zQFOKF/rwsfBtFe" +
            "WlWtcuJMKlXdD8TXWElTzgh7JS4qhFzreL0c1mI0GCj+Aws0usZh7dLIVPnlgZcBhgy1SSDQMQ==";
    private const string OptionalDeviceId = "myCSharpDevice";
    private const ProvisioningStatus OptionalProvisioningStatus = ProvisioningStatus.Enabled;
    ```

1. Agregue lo siguiente para implementar la inscripción del dispositivo:

    ```csharp
    static async Task SetRegistrationDataAsync()
    {
        Console.WriteLine("Starting SetRegistrationData");

        Attestation attestation = new TpmAttestation(SampleTpmEndorsementKey);

        IndividualEnrollment individualEnrollment = new IndividualEnrollment(SampleRegistrationId, attestation);

        individualEnrollment.DeviceId = OptionalDeviceId;
        individualEnrollment.ProvisioningStatus = OptionalProvisioningStatus;

        Console.WriteLine("\nAdding new individualEnrollment...");
        var serviceClient = ProvisioningServiceClient.CreateFromConnectionString(ServiceConnectionString);

        IndividualEnrollment individualEnrollmentResult =
            await serviceClient.CreateOrUpdateIndividualEnrollmentAsync(individualEnrollment).ConfigureAwait(false);

        Console.WriteLine("\nIndividualEnrollment created with success.");
        Console.WriteLine(individualEnrollmentResult);
    }
    ```

1. Finalmente, agregue el código siguiente al método **Main** para abrir la conexión a su IoT Hub y empezar la inscripción:
   
    ```csharp
    try
    {
        Console.WriteLine("IoT Device Provisioning example");

        SetRegistrationDataAsync().GetAwaiter().GetResult();
            
        Console.WriteLine("Done, hit enter to exit.");
    }
    catch (Exception ex)
    {
        Console.WriteLine();
        Console.WriteLine("Error in sample: {0}", ex.Message);
    }
    Console.ReadLine();
    ```
        
1. En el Explorador de soluciones de Visual Studio, haga clic con el botón derecho en la solución y seleccione **Establecer proyectos de inicio...** . Seleccione **Proyecto de inicio único** y, a continuación, seleccione el proyecto **DeviceProvisioning** en el menú desplegable.  

1. Ejecute la aplicación de dispositivos .NET **DeviceProvisioning**. Esta debe configurar el aprovisionamiento del dispositivo: 

    ![Ejecución del registro individual](./media/tutorial-net-provision-device-to-hub/individual.png)

Cuando el dispositivo se haya inscrito satisfactoriamente, verá que aparece en el portal como sigue:

   ![Inscripción correcta en el portal](./media/tutorial-net-provision-device-to-hub/individual-portal.png)

### <a name="enroll-the-device-using-enrollment-groups"></a>Inscripción del dispositivo mediante los grupos de inscripción

> [!NOTE]
> El ejemplo del grupo de inscripción requiere un certificado X.509.

1. En el Explorador de soluciones de Visual Studio, abra el proyecto **DeviceProvisioning** creado anteriormente. 

1. Agregue las siguientes instrucciones `using` al principio del archivo **Program.cs** :
    
    ```csharp
    using System.Security.Cryptography.X509Certificates;
    ```

1. Agregue los campos siguientes a la clase **Program** . Sustituya el valor de marcador de posición por la ubicación del certificado X509.
   
    ```csharp
    private const string X509RootCertPathVar = "{X509 Certificate Location}";
    private const string SampleEnrollmentGroupId = "sample-group-csharp";
    ```

1. Agregue lo siguiente a **Program.cs** para implementar la inscripción del grupo:

    ```csharp
    public static async Task SetGroupRegistrationDataAsync()
    {
        Console.WriteLine("Starting SetGroupRegistrationData");

        using (ProvisioningServiceClient provisioningServiceClient =
                ProvisioningServiceClient.CreateFromConnectionString(ServiceConnectionString))
        {
            Console.WriteLine("\nCreating a new enrollmentGroup...");

            var certificate = new X509Certificate2(X509RootCertPathVar);

            Attestation attestation = X509Attestation.CreateFromRootCertificates(certificate);

            EnrollmentGroup enrollmentGroup = new EnrollmentGroup(SampleEnrollmentGroupId, attestation);

            Console.WriteLine(enrollmentGroup);
            Console.WriteLine("\nAdding new enrollmentGroup...");

            EnrollmentGroup enrollmentGroupResult =
                await provisioningServiceClient.CreateOrUpdateEnrollmentGroupAsync(enrollmentGroup).ConfigureAwait(false);

            Console.WriteLine("\nEnrollmentGroup created with success.");
            Console.WriteLine(enrollmentGroupResult);
        }
    }
    ```

1. Finalmente, sustituya el código siguiente en el método **Main** para abrir la conexión a su IoT Hub y empezar la inscripción de los grupos:
   
    ```csharp
    try
    {
        Console.WriteLine("IoT Device Group Provisioning example");

        SetGroupRegistrationDataAsync().GetAwaiter().GetResult();
            
        Console.WriteLine("Done, hit enter to exit.");
        Console.ReadLine();
    }
    catch (Exception ex)
    {
        Console.WriteLine();
        Console.WriteLine("Error in sample: {0}", ex.Message);
    }
    ```

1. Ejecute la aplicación de dispositivos .NET **DeviceProvisioning**. Esta debe configurar el aprovisionamiento de grupos del dispositivo: 

    ![Ejecución del registro de grupos](./media/tutorial-net-provision-device-to-hub/group.png)

    Cuando el grupo de dispositivos se haya inscrito satisfactoriamente, verá que aparece en el portal como sigue:

   ![Inscripción de grupos satisfactoria en el portal](./media/tutorial-net-provision-device-to-hub/group-portal.png)


## <a name="start-the-device"></a>Iniciar el dispositivo

En este momento, la siguiente configuración está lista para el registro del dispositivo:

1. El dispositivo o grupo de dispositivos ya están inscritos en el servicio Device Provisioning, y 
2. El dispositivo está preparado con la seguridad configurada y accesible a través de la aplicación mediante el SDK de cliente del servicio Device Provisioning.

Inicie el dispositivo para permitir que la aplicación cliente inicie el registro con el servicio Device Provisioning.  


## <a name="verify-the-device-is-registered"></a>Comprobar que el dispositivo está registrado

Una vez que arranque el dispositivo, se deben realizar las siguientes acciones. Para más información, consulte el [ejemplo de aprovisionamiento de dispositivos cliente](https://github.com/Azure-Samples/azure-iot-samples-csharp/tree/master/provisioning/Samples/device). 

1. El dispositivo envía una solicitud de registro al servicio Device Provisioning.
2. En el caso de los dispositivos TPM, el servicio Device Provisioning devuelve un desafío de registro al que responde el dispositivo. 
3. Una vez que el registro se ha realizado correctamente, el servicio Device Provisioning envía el identificador URI del centro de IoT, el del dispositivo y la clave cifrada nuevamente al dispositivo. 
4. Luego, la aplicación cliente de IoT Hub del dispositivo se conecta al centro. 
5. Después de una conexión correcta, el dispositivo aparecerá en la página **Device Explorer** (Explorador de dispositivos) del centro de IoT. 

    ![Conexión correcta al centro en el portal](./media/tutorial-net-provision-device-to-hub/hub-connect-success.png)

## <a name="next-steps"></a>Pasos siguientes
En este tutorial, ha aprendido a:

> [!div class="checklist"]
> * Inscribir el dispositivo
> * Iniciar el dispositivo
> * Comprobar que el dispositivo está registrado

Pase al siguiente tutorial para aprender a aprovisionar varios dispositivos en centros con equilibrio de carga. 

> [!div class="nextstepaction"]
> [Aprovisionamiento de dispositivos en instancias de IoT Hub con equilibrio de carga](./tutorial-provision-multiple-hubs.md)
