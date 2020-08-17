---
title: 'Tutorial: Aprovisionamiento de un dispositivo X.509 simulado para Azure IoT Hub mediante Java y los grupos de inscripción'
description: En este tutorial va a crear y aprovisionar un dispositivo X.509 simulado mediante el SDK de dispositivos y servicios Java y los grupos de inscripción de Azure IoT Hub Device Provisioning Service (DPS).
author: wesmc7777
ms.author: wesmc
ms.date: 11/12/2019
ms.topic: tutorial
ms.service: iot-dps
services: iot-dps
ms.devlang: java
ms.custom: mvc, devx-track-java
ms.openlocfilehash: 43ae9bd0e844a23adabf52e7fbb4b8cd077d23bd
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87326722"
---
# <a name="tutorial-create-and-provision-a-simulated-x509-device-using-java-device-and-service-sdk-and-group-enrollments-for-iot-hub-device-provisioning-service"></a>Tutorial: Creación y aprovisionamiento de un dispositivo X.509 simulado mediante el SDK de dispositivos y servicios Java y las inscripciones de grupos para el servicio Azure IoT Hub Device Provisioning

Estos pasos muestran cómo simular un dispositivo X.509 en la máquina de desarrollo en la que se ejecuta el sistema operativo Windows, y usar un ejemplo de código para conectar este dispositivo simulado con el servicio Device Provisioning y la instancia de IoT Hub mediante los grupos de inscripción. 

Asegúrese de completar los pasos descritos en [Configuración de un servicio Azure IoT Hub Device Provisioning con Azure Portal](./quick-setup-auto-provision.md) antes de continuar.


## <a name="prepare-the-environment"></a>Preparación del entorno 

1. Asegúrese de que tiene [Java SE Development Kit 8](https://aka.ms/azure-jdks) instalado en su máquina.

1. Descargue e instale [Maven](https://maven.apache.org/install.html).

1. Asegúrese de que `git` está instalado en su máquina y se agrega a las variables de entorno accesibles para la ventana de comandos. Consulte las [herramientas de cliente de Git de Software Freedom Conservancy](https://git-scm.com/download/) para instalar la versión más reciente de las herramientas `git`, lo que incluye **Git Bash**, la aplicación de línea de comandos que puede usar para interactuar con su repositorio de Git local. 

1. Utilice el siguiente articulo de [introducción a los certificados](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md) para crear los certificados de prueba.

    > [!NOTE]
    > Este paso requiere [OpenSSL](https://www.openssl.org/), que se puede integrar e instalar desde el origen o descargarse e instalarse desde el sitio web de un [tercero](https://wiki.openssl.org/index.php/Binaries) como [este](https://sourceforge.net/projects/openssl/). Si ya ha creado los certificados _raíz_, _intermedio_ y de _dispositivo_, puede omitir este paso.
    >

    1. Siga los dos primeros pasos para crear sus certificados _raíz_ e _intermedio_.

    1. Inicie sesión en Azure Portal, haga clic en el botón **Todos los recursos** del menú izquierdo y abra el servicio de aprovisionamiento.

        1. En la hoja de resumen del servicio Device Provisioning, seleccione **Certificados** y haga clic en el botón **Agregar** situado en la parte superior.

        1. En **Agregar certificado**, escriba la siguiente información:
            - Escriba un nombre de certificado único.
            - Seleccione el archivo **_RootCA.pem_** que ha creado.
            - Una vez completado, haga clic en el botón **Guardar**.

           ![Agregar certificado](./media/tutorial-group-enrollments/add-certificate.png)

        1. Seleccione el certificado recién creado:
            - Haga clic en **Generar código de verificación**. Copie el código generado.
            - Ejecute el paso de comprobación. Escriba el _código de verificación_ o haga clic con el botón derecho para pegarlo en la ventana de PowerShell que se está ejecutando.  Presione **Entrar**.
            - Seleccione el archivo **_verifyCert4.pem_** recién creado en Azure Portal. Haga clic en **Comprobar**.

              ![Validación del certificado](./media/tutorial-group-enrollments/validate-certificate.png)

    1. Para finalizar, ejecute los pasos necesarios para crear los certificados de los dispositivos y limpiar recursos.

       > [!NOTE]
       > Al crear los certificados de dispositivo, asegúrese de usar solo caracteres alfanuméricos en minúsculas y guiones en el nombre del dispositivo.
       >


## <a name="create-a-device-enrollment-entry"></a>Creación de una entrada de inscripción de dispositivo

1. Abra un símbolo del sistema. Clone el repositorio de GitHub para obtener ejemplos de código del SDK de Java:

    ```cmd/sh
    git clone https://github.com/Azure/azure-iot-sdk-java.git --recursive
    ```

1. En el código fuente descargado, navegue hasta la carpeta de ejemplo **_azure-iot-sdk-java/provisioning/provisioning-samples/service-enrollment-group-sample_**. Abra el archivo **_/src/main/java/samples/com/microsoft/azure/sdk/iot/ServiceEnrollmentGroupSample.java_** en el editor que prefiera y agregue los detalles siguientes:

    1. Desde el portal, agregue la cadena `[Provisioning Connection String]` para su servicio de aprovisionamiento como se muestra a continuación:

        1. Vaya al servicio de aprovisionamiento en [Azure Portal](https://portal.azure.com).

        1. Abra las **directivas de acceso compartido** y seleccione una directiva que tenga el permiso *EnrollmentWrite*.

        1. Copie la **cadena de conexión de clave principal**.

            ![Obtención de la cadena de conexión de aprovisionamiento desde el portal](./media/tutorial-group-enrollments/provisioning-string.png)  

        1. En el archivo de código de ejemplo **_ServiceEnrollmentGroupSample.java_**, reemplace la cadena `[Provisioning Connection String]` con la **cadena de conexión de clave principal**.

            ```java
            private static final String PROVISIONING_CONNECTION_STRING = "[Provisioning Connection String]";
            ```

    1. Abra el archivo de certificado de firma intermedio en un editor de texto. Actualice el valor `PUBLIC_KEY_CERTIFICATE_STRING` con el del certificado de firma intermedio.

        Si ha generado los certificados de dispositivo con el shell de Bash, *./certs/azure-iot-test-only.intermediate.cert.pem* contiene la clave del certificado intermedio. Si los certificados se generaron con PowerShell, *./Intermediate1.pem* será el archivo del certificado intermedio.

        ```java
        private static final String PUBLIC_KEY_CERTIFICATE_STRING =
                "-----BEGIN CERTIFICATE-----\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "-----END CERTIFICATE-----\n";
        ```

    1. Vaya a la instancia de IoT Hub enlazada con el servicio de aprovisionamiento en [Azure Portal](https://portal.azure.com). Abra la pestaña **Información general** de dicha instancia y copie el **nombre de host**. Asigne este **nombre de host** al parámetro *IOTHUB_HOST_NAME*.

        ```java
        private static final String IOTHUB_HOST_NAME = "[Host name].azure-devices.net";
        ```

    1. Estudie el código de ejemplo. Crea, actualiza, consulta y elimina una inscripción de un grupo para dispositivos X.509. Para comprobar si la inscripción se realizó correctamente en el portal, convierta temporalmente en comentario las siguientes líneas de código al final del archivo _ServiceEnrollmentGroupSample.java_:

        ```java
        // ************************************** Delete info of enrollmentGroup ***************************************
        System.out.println("\nDelete the enrollmentGroup...");
        provisioningServiceClient.deleteEnrollmentGroup(enrollmentGroupId);
        ```

    1. Guarde el archivo _ServiceEnrollmentGroupSample.java_.

1. Abra una ventana de comandos y navegue hasta la carpeta **_azure-iot-sdk-java/provisioning/provisioning-samples/service-enrollment-group-sample_**.

1. Compile el código de ejemplo con este comando:

    ```cmd\sh
    mvn install -DskipTests
    ```

1. Para ejecutar el ejemplo, use estos comandos en la ventana de comandos:

    ```cmd\sh
    cd target
    java -jar ./service-enrollment-group-sample-{version}-with-deps.jar
    ```

1. Observe la ventana de salida para comprobar si la inscripción se realizó correctamente.

    ![Inscripción correcta](./media/tutorial-group-enrollments/enrollment.png) 

1. Vaya al servicio de aprovisionamiento en Azure Portal. Haga clic en **Administrar inscripciones**. Tenga en cuenta que el grupo de dispositivos X.509 aparecerá en la pestaña **Grupos de inscripción** junto con un *NOMBRE DE GRUPO* generado automáticamente.

## <a name="simulate-the-device"></a>Simulación del dispositivo

1. En la hoja de resumen del servicio Device Provisioning, seleccione **Introducción** y tome nota de los campos _Ámbito de id._ y _Provisioning Service Global Endpoint_ (Punto de conexión global del Servicio de aprovisionamiento).

    ![Información del servicio](./media/tutorial-group-enrollments/extract-dps-endpoints.png)

1. Abra un símbolo del sistema. Navegue hasta la carpeta del proyecto de ejemplo.

    ```cmd/sh
    cd azure-iot-sdk-java/provisioning/provisioning-samples/provisioning-X509-sample
    ```

1. Edite `/src/main/java/samples/com/microsoft/azure/sdk/iot/ProvisioningX509Sample.java` para que incluya el _Ámbito de id._ y el _punto de conexión global del aprovisionamiento_ que anotó previamente.

    ```java
    private static final String idScope = "[Your ID scope here]";
    private static final String globalEndpoint = "[Your Provisioning Service Global Endpoint here]";
    private static final ProvisioningDeviceClientTransportProtocol PROVISIONING_DEVICE_CLIENT_TRANSPORT_PROTOCOL = ProvisioningDeviceClientTransportProtocol.HTTPS;
    private static final int MAX_TIME_TO_WAIT_FOR_REGISTRATION = 10000; // in milli seconds
    private static final String leafPublicPem = "<Your Public PEM Certificate here>";
    private static final String leafPrivateKey = "<Your Private PEM Key here>";
    ```

1. Actualice las variables `leafPublicPem` y `leafPrivateKey` con los certificados de dispositivo público y privado.

    Si ha generado los certificados de dispositivo con PowerShell, los archivos mydevice* contienen la clave pública, la privada y el PFX del dispositivo.

    Si ha generado los certificados de dispositivo con el shell de Bash, ./certs/new-device.cert.pem contiene la clave pública. La clave privada del dispositivo estará en el archivo ./private/new-device.key.pem.

    Abra el archivo de clave pública y actualice la variable `leafPublicPem` con ese valor. Copie el texto de _-----BEGIN PRIVATE KEY-----_ en _-----END PRIVATE KEY-----_ .

    ```java
    private static final String leafPublicPem = "-----BEGIN CERTIFICATE-----\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "-----END CERTIFICATE-----\n";
    ```

    Abra el archivo de clave privada y actualice la variable `leafPrivatePem` con ese valor. Copie el texto de _-----BEGIN RSA PRIVATE KEY-----_ en _-----END RSA PRIVATE KEY-----_ .

    ```java
    private static final String leafPrivateKey = "-----BEGIN RSA PRIVATE KEY-----\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "-----END RSA PRIVATE KEY-----\n";
    ```

1. Agregue una nueva variable justo debajo de `leafPrivateKey` para su certificado intermedio. Dele el nombre `intermediateKey` a esta nueva variable. Dele el valor del certificado de firma intermedio.

    Si ha generado los certificados de dispositivo con el shell de Bash, *./certs/azure-iot-test-only.intermediate.cert.pem* contiene la clave del certificado intermedio. Si los certificados se generaron con PowerShell, *./Intermediate1.pem* será el archivo del certificado intermedio.

    ```java
    private static final String intermediateKey = "-----BEGIN CERTIFICATE-----\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "-----END CERTIFICATE-----\n";
    ```

1. En la función `main`, agregue `intermediateKey` a la colección `signerCertificates` antes de inicializar `securityProviderX509`.

    ```java
    public static void main(String[] args) throws Exception
    {
        ...

        try
        {
            ProvisioningStatus provisioningStatus = new ProvisioningStatus();

            // Add intermediate certificate as part of the certificate key chain.
            signerCertificates.add(intermediateKey);

            SecurityProvider securityProviderX509 = new SecurityProviderX509Cert(leafPublicPem, leafPrivateKey, signerCertificates);
    ```

1. Guarde los cambios y compile el ejemplo. Navegue hasta la carpeta de destino y ejecute el archivo .jar que se creó.

    ```cmd/sh
    mvn clean install
    cd target
    java -jar ./provisioning-x509-sample-{version}-with-deps.jar
    ```

    ![El registro se completó correctamente.](./media/tutorial-group-enrollments/registration.png)

1. En el portal, navegue hasta IoT Hub vinculado a su servicio de aprovisionamiento y abra la hoja **Device Explorer**. En el aprovisionamiento correcto del dispositivo simulado X.509 con el Hub, su identificador de dispositivo aparece en la hoja **Device Explorer** con el *ESTADO* **habilitado**. Tenga en cuenta que debe hacer clic en el botón **Actualizar** situado en la parte superior si ya ha abierto la hoja antes de ejecutar la aplicación de ejemplo del dispositivo. 

    ![El dispositivo se registra con el centro de IoT](./media/tutorial-group-enrollments/hub-registration.png) 


## <a name="clean-up-resources"></a>Limpieza de recursos

Si piensa seguir trabajando con el ejemplo de cliente de dispositivo y explorándolo, no limpie los recursos que se crean en esta guía de inicio rápido. Si no va a continuar, use el siguiente comando para eliminar todos los recursos creados.

1. Cierre la ventana de salida de ejemplo del cliente del dispositivo en su máquina.
1. Desde el menú de la izquierda en Azure Portal, haga clic en **Todos los recursos** y seleccione el servicio Device Provisioning. Abra la hoja **Administrar inscripciones** de su servicio y, a continuación, haga clic en la pestaña **Inscripciones individuales**. Seleccione el *ID. DE REGISTRO* del dispositivo que inscribió en esta guía de inicio rápido y haga clic en el botón **Eliminar** situado en la parte superior. 
1. Desde el menú de la izquierda en Azure Portal, haga clic en **Todos los recursos** y seleccione su centro de IoT. Abra la hoja **IoT Devices** (Dispositivos IoT) de su centro, seleccione el *ID. DE DISPOSITIVO* del dispositivo que registró en esta guía de inicio rápido y, a continuación, haga clic en el botón **Eliminar** situado en la parte superior.


## <a name="next-steps"></a>Pasos siguientes

En este tutorial, creó un dispositivo simulado X.509 en su máquina Windows y lo aprovisionó con IoT Hub mediante el servicio Azure IoT Hub Device Provisioning y los grupos de inscripción. Para más información sobre el dispositivo X.509, continúe con los conceptos de dispositivo. 

> [!div class="nextstepaction"]
> [Conceptos de dispositivos del servicio IoT Hub Device Provisioning](concepts-device.md)
