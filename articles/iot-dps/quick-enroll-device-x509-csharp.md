---
title: Inscripción de un dispositivo X.509 al Servicio Azure Device Provisioning mediante C#
description: Esta guía de inicio rápido usa inscripciones de grupos. En este inicio rápido, inscribirá dispositivos X.509 en Azure IoT Hub Device Provisioning Service (DPS) mediante C#.
author: wesmc7777
ms.author: wesmc
ms.date: 11/08/2019
ms.topic: quickstart
ms.service: iot-dps
services: iot-dps
ms.devlang: csharp
ms.custom: mvc, devx-track-csharp
ms.openlocfilehash: 89d98cdf6f635cab3b85462adf5c6695f7c4482e
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89020939"
---
# <a name="quickstart-enroll-x509-devices-to-the-device-provisioning-service-using-c"></a>Guía de inicio rápido: Inscripción de dispositivos X.509 en el servicio Device Provisioning con C#

[!INCLUDE [iot-dps-selector-quick-enroll-device-x509](../../includes/iot-dps-selector-quick-enroll-device-x509.md)]

En esta guía de inicio rápido se muestra cómo usar C# para crear mediante programación un [grupo de inscripción](concepts-service.md#enrollment-group) que usa certificados X.509 intermedios o raíces de una entidad de certificación. El grupo de inscripción se crea mediante el [SDK de Microsoft Azure IoT para .NET](https://github.com/Azure/azure-iot-sdk-csharp) y una aplicación de .NET Core en C# de ejemplo. Un grupo de inscripción controla el acceso al servicio de aprovisionamiento de los dispositivos que comparten un certificado de firma común en su cadena de certificados. Para más información, consulte [Control del acceso de dispositivo al servicio de aprovisionamiento con certificados X.509](./concepts-security.md#controlling-device-access-to-the-provisioning-service-with-x509-certificates). Para más información sobre el uso de la infraestructura de clave pública (PKI) basada en certificados X.509 con Azure IoT Hub y el servicio Device Provisioning, consulte [Introducción a la seguridad mediante certificados de entidades de certificación X.509](https://docs.microsoft.com/azure/iot-hub/iot-hub-x509ca-overview). 

En este inicio rápido se asume que ya ha creado un centro de IoT y una instancia de Device Provisioning Service. Si aún no ha creado estos recursos, complete el inicio rápido [Configuración del servicio Azure IoT Hub Device Provisioning con Azure Portal](./quick-setup-auto-provision.md) antes de continuar con este artículo.

Aunque los pasos de este artículo funcionan en equipos Windows y Linux, en este artículo se usa un equipo de desarrollo de Windows.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prerequisites

* Instale [Visual Studio 2019](https://www.visualstudio.com/vs/).
* Instalación del [SDK de .NET Core](https://www.microsoft.com/net/download/windows).
* Instale [Git](https://git-scm.com/download/).

## <a name="prepare-test-certificates"></a>Preparación de los certificados de prueba

En esta guía de inicio rápido debe tener un archivo .pem o .cer que contenga la porción pública de un certificado X.509 intermedio o raíz de entidad de certificación. Se debe cargar este certificado en el servicio de aprovisionamiento y que este lo compruebe.

El [SDK de Azure IoT para C](https://github.com/Azure/azure-iot-sdk-c) contiene herramientas de prueba que pueden ayudarle a crear una cadena de certificados X.509, cargar un certificado raíz o intermedio a partir de esa cadena y realizar la prueba de posesión con el servicio para comprobar el certificado.

> [!CAUTION]
> Use los certificados creados con las herramientas del SDK para pruebas de desarrollo únicamente.
> No use estos certificados en producción.
> Contienen contraseñas codificadas de forma rígida como, por ejemplo *1234*, que expiran después de 30 días.
> Para saber cómo obtener certificados adecuados para el código de producción, consulte [Cómo obtener un certificado de entidad de certificación X.509](https://docs.microsoft.com/azure/iot-hub/iot-hub-x509ca-overview#how-to-get-an-x509-ca-certificate) en la documentación de Azure IoT Hub.
>

Para usar estas herramientas de prueba para generar certificados, realice los pasos siguientes:

1. Busque el nombre de etiqueta de la [versión más reciente](https://github.com/Azure/azure-iot-sdk-c/releases/latest) del SDK de C de IoT de Azure.

2. Abra un símbolo del sistema o el shell de Git Bash y cambie a una carpeta de trabajo de la máquina. Ejecute los siguientes comandos para clonar la versión más reciente del repositorio de GitHub del [SDK de Azure IoT para C](https://github.com/Azure/azure-iot-sdk-c). Use la etiqueta que encontró en el paso anterior como valor del parámetro `-b`:

    ```cmd/sh
    git clone -b <release-tag> https://github.com/Azure/azure-iot-sdk-c.git
    cd azure-iot-sdk-c
    git submodule update --init
    ```

    Esta operación puede tardar varios minutos en completarse.

   Las herramientas de prueba se encuentran en el directorio *azure-iot-sdk-c/tools/CACertificates* del repositorio que clonó.

3. Siga los pasos descritos en [Managing test CA certificates for samples and tutorials](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md) (Administración de certificados de entidad de certificación de prueba para ejemplos y tutoriales).

Además de las herramientas del SDK de C, el [ejemplo de comprobación del certificado de grupo](https://github.com/Azure-Samples/azure-iot-samples-csharp/tree/master/provisioning/Samples/service/GroupCertificateVerificationSample) del *SDK de Microsoft Azure IoT para .NET* muestra cómo realizar la prueba de posesión en C# con un certificado X.509 existente de entidad de certificación raíz o intermedio.

## <a name="get-the-connection-string-for-your-provisioning-service"></a>Obtención de la cadena de conexión para el servicio de aprovisionamiento

En el ejemplo de esta guía de inicio rápido, necesitará la cadena de conexión del servicio de aprovisionamiento.

1. Inicie sesión en Azure Portal, seleccione **Todos los recursos** y abra la instancia de Device Provisioning Service.

1. Seleccione **Directivas de acceso compartido** y, a continuación, elija la directiva de acceso que quiere usar para abrir sus propiedades. En **Directiva de acceso**, copie y guarde la cadena de conexión de la clave principal.

    ![Obtención de la cadena de conexión del servicio de aprovisionamiento desde el portal](media/quick-enroll-device-x509-csharp/get-service-connection-string-vs2019.png)

## <a name="create-the-enrollment-group-sample"></a>Crear el ejemplo del grupo de inscripción 

En esta sección se muestra cómo crear una aplicación de consola de .NET Core que agrega un grupo de inscripción al servicio de aprovisionamiento. Con algunas modificaciones, también puede seguir estos pasos para crear una aplicación de consola de [Windows IoT Core](https://developer.microsoft.com/en-us/windows/iot) para agregar el grupo de inscripción. Para más información sobre el desarrollo con IoT Core, consulte la [documentación para desarrolladores de Windows IoT Core](https://docs.microsoft.com/windows/iot-core/).

1. Abra Visual Studio y seleccione **Crear un proyecto**. En **Crear un proyecto**, elija la plantilla de proyecto **Aplicación de consola (.NET Core)** para C# y seleccione **Siguiente**.

1. Asigne al proyecto el nombre *CreateEnrollmentGroup* y presione **Crear**.

    ![Configuración de un proyecto de escritorio clásico de Windows de Visual C#](media//quick-enroll-device-x509-csharp/configure-app-vs2019.png)

1. Cuando se abre la solución en Visual Studio, en el panel **Explorador de soluciones**, haga clic con el botón derecho en el proyecto **CreateEnrollmentGroup** y, a continuación, seleccione **Administrar paquetes NuGet**.

1. En **Administrador de paquetes NuGet**, seleccione **Examinar**, busque y elija **Microsoft.Azure.Devices.Provisioning.Service** y presione **Instalar**.

    ![Ventana del Administrador de paquetes NuGet](media//quick-enroll-device-x509-csharp/add-nuget.png)

   Este paso descarga, instala y agrega una referencia al paquete NuGet del [SDK de cliente del servicio de aprovisionamiento de Azure IoT](https://www.nuget.org/packages/Microsoft.Azure.Devices.Provisioning.Service/) y sus dependencias.

1. Agregue las siguientes instrucciones `using` después de las demás instrucciones `using` en la parte superior de `Program.cs`:

   ```csharp
   using System.Security.Cryptography.X509Certificates;
   using System.Threading.Tasks;
   using Microsoft.Azure.Devices.Provisioning.Service;
   ```

1. Agregue los siguientes campos a la clase `Program` y realice los cambios de la lista.  

   ```csharp
   private static string ProvisioningConnectionString = "{ProvisioningServiceConnectionString}";
   private static string EnrollmentGroupId = "enrollmentgrouptest";
   private static string X509RootCertPath = @"{Path to a .cer or .pem file for a verified root CA or intermediate CA X.509 certificate}";
   ```

   * Reemplace el valor del marcador de posición `ProvisioningServiceConnectionString` por la cadena de conexión del servicio de aprovisionamiento para el que quiere crear la inscripción.

   * Reemplace el valor del marcador de posición `X509RootCertPath` por la ruta de acceso a un archivo .pem o .cer. Este archivo representa la parte pública de un certificado X.509 de entidad de certificación raíz o intermedio que se haya cargado y comprobado previamente en el servicio de aprovisionamiento.

   * Tiene la opción de cambiar el valor `EnrollmentGroupId`. La cadena solo puede contener caracteres en minúsculas y guiones.

   > [!IMPORTANT]
   > En el código de producción, tenga en cuenta las consideraciones de seguridad siguientes:
   >
   > * La codificación de forma rígida de la cadena de conexión para el administrador del servicio de aprovisionamiento va contra los procedimientos recomendados de seguridad. En su lugar, la cadena de conexión debe mantenerse de forma segura, como en un archivo de configuración seguro o en el registro.
   > * Asegúrese de cargar solo la parte pública del certificado de firma. No cargue nunca archivos .pfx (PKCS12) o .pem que contengan claves privadas en el servicio de aprovisionamiento.

1. Agregue el siguiente método a la clase `Program`. Este código crea una entrada de grupo de inscripción y luego llama al método `CreateOrUpdateEnrollmentGroupAsync` en `ProvisioningServiceClient` para agregar el grupo de inscripción al servicio de aprovisionamiento.

   ```csharp
   public static async Task RunSample()
   {
       Console.WriteLine("Starting sample...");
 
       using (ProvisioningServiceClient provisioningServiceClient =
               ProvisioningServiceClient.CreateFromConnectionString(ProvisioningConnectionString))
       {
           #region Create a new enrollmentGroup config
           Console.WriteLine("\nCreating a new enrollmentGroup...");
           var certificate = new X509Certificate2(X509RootCertPath);
           Attestation attestation = X509Attestation.CreateFromRootCertificates(certificate);
           EnrollmentGroup enrollmentGroup =
                   new EnrollmentGroup(
                           EnrollmentGroupId,
                           attestation)
                   {
                       ProvisioningStatus = ProvisioningStatus.Enabled
                   };
           Console.WriteLine(enrollmentGroup);
           #endregion
 
           #region Create the enrollmentGroup
           Console.WriteLine("\nAdding new enrollmentGroup...");
           EnrollmentGroup enrollmentGroupResult =
               await provisioningServiceClient.CreateOrUpdateEnrollmentGroupAsync(enrollmentGroup).ConfigureAwait(false);
           Console.WriteLine("\nEnrollmentGroup created with success.");
           Console.WriteLine(enrollmentGroupResult);
           #endregion
 
       }
   }
   ```

1. Por último, reemplace el cuerpo del método `Main` por las siguientes líneas:

   ```csharp
   RunSample().GetAwaiter().GetResult();
   Console.WriteLine("\nHit <Enter> to exit ...");
   Console.ReadLine();
   ```

1. Compile la solución.

## <a name="run-the-enrollment-group-sample"></a>Ejecutar el ejemplo del grupo de inscripción
  
Ejecute el ejemplo en Visual Studio para crear el grupo de inscripción. Aparecerá una ventana del símbolo del sistema que comenzará a mostrar los mensajes de confirmación. Tras una creación correcta, la ventana del símbolo del sistema muestra las propiedades del grupo de inscripción nuevo.

Puede comprobar que se ha creado el grupo de inscripción. En el resumen de Device Provisioning Service, seleccione **Administrar inscripciones** y, después, **Grupos de inscripción**. Verá una nueva entrada de inscripción que corresponde al identificador de registro usado en el ejemplo.

![Propiedades de la inscripción en el portal](media/quick-enroll-device-x509-csharp/verify-enrollment-portal-vs2019.png)

Seleccione la entrada para comprobar la huella digital del certificado y otras propiedades de la entrada.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si tiene previsto explorar el ejemplo del servicio de C#, no elimine los recursos que se crearon con este inicio rápido. De lo contrario, use el siguiente comando para eliminar todos los recursos creados.

1. Cierre la ventana de salida de ejemplo de C# en el equipo.

1. Vaya a Device Provisioning Service en Azure Portal, seleccione **Administrar inscripciones** y, a continuación, seleccione **Grupos de inscripción**. Seleccione el *identificador de registro* de la entrada de inscripción que creó en este inicio rápido y presione **Eliminar**.

1. Desde el servicio Device Provisioning en Azure Portal, seleccione **Certificados**, elija el certificado que cargó para este inicio rápido y presione **Eliminar** en la parte superior de **Detalles del certificado**.  

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado un grupo de inscripción para un certificado X.509 de entidad de certificación intermedio o raíz mediante Azure IoT Hub Device Provisioning Service. Para más información acerca del aprovisionamiento de dispositivos, continúe con el tutorial para instalar el servicio Device Provisioning en Azure Portal.

> [!div class="nextstepaction"]
> [Tutoriales del servicio Azure IoT Hub Device Provisioning](./tutorial-set-up-cloud.md)
