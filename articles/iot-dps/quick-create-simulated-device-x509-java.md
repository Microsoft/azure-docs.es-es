---
title: Aprovisionamiento de un dispositivo X.509 simulado en Azure IoT Hub mediante Java
description: 'Inicio rápido de Azure: Creación y aprovisionamiento de un dispositivo X.509 simulado mediante el SDK de dispositivos para Java para IoT Hub Device Provisioning Service (DPS). Este inicio rápido utiliza inscripciones individuales.'
author: wesmc7777
ms.author: wesmc
ms.date: 11/08/2018
ms.topic: quickstart
ms.service: iot-dps
services: iot-dps
ms.devlang: java
ms.custom: mvc, devx-track-java
ms.openlocfilehash: f16ace3e54db4d30f89da7335f00153191a7e226
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87326756"
---
# <a name="quickstart-create-and-provision-a-simulated-x509-device-using-java-device-sdk-for-iot-hub-device-provisioning-service"></a>Inicio rápido: Creación y aprovisionamiento de un dispositivo X.509 simulado mediante el SDK de dispositivos Java para el servicio Azure IoT Hub Device Provisioning

[!INCLUDE [iot-dps-selector-quick-create-simulated-device-x509](../../includes/iot-dps-selector-quick-create-simulated-device-x509.md)]

En este inicio rápido se crea un dispositivo simulado X.509 en un equipo Windows. Use el código de ejemplo de Java del dispositivo para conectar este dispositivo simulado con su centro de IoT mediante una inscripción individual con el servicio de aprovisionamiento de dispositivos.

## <a name="prerequisites"></a>Prerrequisitos

- Revisión de [Conceptos de aprovisionamiento automático](concepts-auto-provisioning.md).
- Finalización de [Configuración de Azure IoT Hub Device Provisioning Service con Azure Portal](./quick-setup-auto-provision.md).
- Una cuenta de Azure con una suscripción activa. [cree una de forma gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
- [Java SE Development Kit 8](https://aka.ms/azure-jdks).
- [Maven](https://maven.apache.org/install.html).
- [Git](https://git-scm.com/download/).

## <a name="prepare-the-environment"></a>Preparación del entorno 

1. Asegúrese de que tiene [Java SE Development Kit 8](https://aka.ms/azure-jdks) instalado en su máquina.

2. Descargue e instale [Maven](https://maven.apache.org/install.html).

3. Asegúrese de que Git está instalado en la máquina y se ha agregado a las variables de entorno accesibles desde la ventana de comandos. Consulte las [herramientas de cliente de Git de Software Freedom Conservancy](https://git-scm.com/download/) para instalar la versión más reciente de las herramientas `git`, lo que incluye **Git Bash**, la aplicación de línea de comandos que puede usar para interactuar con su repositorio de Git local. 

4. Abra un símbolo del sistema. Clone el repositorio GitHub para el ejemplo de código de simulación de dispositivo:
    
    ```cmd/sh
    git clone https://github.com/Azure/azure-iot-sdk-java.git --recursive
    ```
5. Vaya al directorio `azure-iot-sdk-`java raíz y compile el proyecto para descargar todos los paquetes necesarios.
   
   ```cmd/sh
   cd azure-iot-sdk-java
   mvn install -DskipTests=true
   ```
6. Navegue hasta el proyecto de generación de certificados y compílelo. 

    ```cmd/sh
    cd azure-iot-sdk-java/provisioning/provisioning-tools/provisioning-x509-cert-generator
    mvn clean install
    ```

## <a name="create-a-self-signed-x509-device-certificate-and-individual-enrollment-entry"></a>Creación de un certificado de dispositivo X.509 autofirmado y una entrada de inscripción individual

En esta sección, usará un certificado X.509 autofirmado, así que es importante tener en cuenta lo siguiente:

* Los certificados autofirmados son solo para la realización de pruebas, no se deben usar en producción.
* La fecha de expiración predeterminada de un certificado autofirmado es de un año.

Usará código de ejemplo del [SDK para Java de Azure IoT](https://github.com/Azure/azure-iot-sdk-java.git) para crear el certificado que se usará con la entrada de inscripción individual en el dispositivo simulado.

Azure IoT Hub Device Provisioning Service admite dos tipos de inscripciones:

- [Grupos de inscripción](concepts-service.md#enrollment-group): usados para inscribir varios dispositivos relacionados.
- [Inscripciones individuales](concepts-service.md#individual-enrollment): usadas para inscribir un solo dispositivo.

En este artículo se muestran las inscripciones individuales.

1. Siga usando el símbolo del sistema que empleó en los pasos anteriores para ir a la carpeta `target` y, después, ejecute el archivo .jar que creó en el paso anterior.

    ```cmd/sh
    cd target
    java -jar ./provisioning-x509-cert-generator-{version}-with-deps.jar
    ```

2. Escriba **N** en _Do you want to input common name?_ (¿Quiere escribir un nombre de entrada?). Copie en el Portapapeles la salida de `Client Cert` desde *---BEGIN CERTIFICATE---* hasta *---END CERTIFICATE---* .

   ![Generador de certificados individuales](./media/java-quick-create-simulated-device-x509/individual.png)

3. Cree un archivo denominado **_X509individual.pem_** en su máquina con Windows, ábralo en el editor que prefiera y copie el contenido del Portapapeles en este archivo. Guarde el archivo y cierre el editor.

4. En el símbolo del sistema, escriba **N** en _Do you want to input Verification Code_ (¿Desea escribir un código de verificación?) y mantenga la salida del programa abierta para usarla como referencia más adelante en el inicio rápido. A continuación, copie los valores `Client Cert` y `Client Cert Private Key` para su uso en la sección siguiente.

5. Inicie sesión en [Azure Portal](https://portal.azure.com), seleccione el botón **Todos los recursos** situado en el menú izquierdo y abra la instancia de Device Provisioning Service.

6. En el menú de Device Provisioning Service, seleccione **Administrar inscripciones**. Seleccione la pestaña **Inscripciones individuales** y seleccione el botón **Agregar inscripción individual** de la parte superior. 

7. En el panel **Agregar inscripción**, escriba la siguiente información:
   - Seleccione **X.509** como *Mecanismo* de atestación de identidad.
   - En *Archivo .pem o .cer del certificado principal*, elija *Seleccionar un archivo* para seleccionar el archivo de certificado **X509individual.pem** que creó en los pasos anteriores.  
   - De forma opcional, puede proporcionar la siguiente información:
     - Seleccione un centro de IoT vinculado con el servicio de aprovisionamiento.
     - Escriba un identificador de dispositivo único. Asegúrese de evitar datos confidenciales al asignar nombre al dispositivo. 
     - Actualice el **Estado inicial del dispositivo gemelo** con la configuración inicial deseada para el dispositivo.
     - Una vez completado, presione el botón **Guardar**. 

     [![Agregar inscripción individual para la atestación X.509 en el portal](./media/java-quick-create-simulated-device-x509/device-enrollment.png)](./media/how-to-manage-enrollments/individual-enrollment.png#lightbox)

     Al inscribir el dispositivo correctamente, el dispositivo X.509 aparece como **microsoftriotcore** en la columna *Identificador de registro* de la pestaña *Inscripciones individuales*. 



## <a name="simulate-the-device"></a>Simulación del dispositivo

1. En la el menú Device Provisioning Service, seleccione **Overview** (Introducción) y tome nota de los campos _ID Scope_ (Ámbito de id.) y _Provisioning Service Global Endpoint_ (Punto de conexión global del Servicio de aprovisionamiento).

    ![Información del servicio](./media/java-quick-create-simulated-device-x509/extract-dps-endpoints.png)

2. Abra un símbolo del sistema. Vaya a la carpeta del proyecto de ejemplo en el repositorio del SDK de Java.

    ```cmd/sh
    cd azure-iot-sdk-java/provisioning/provisioning-samples/provisioning-X509-sample
    ```

3. Escriba la información del servicio de aprovisionamiento y de la identidad X.509 en el código. Se utiliza durante el aprovisionamiento automático para la atestación del dispositivo simulado, antes del registro de dispositivo:

   - Edite el archivo `/src/main/java/samples/com/microsoft/azure/sdk/iot/ProvisioningX509Sample.java` para que incluya el _Ámbito de id._ y el _Provisioning Service Global Endpoint_ (Punto de conexión global del Servicio de aprovisionamiento) tal y como se indicó previamente. Incluya también el valor de _Client cert_ (Certificado de cliente) y _Client Cert Private Key_ (Clave privada del certificado de cliente) tal y como se indicó en la sección anterior.

      ```java
      private static final String idScope = "[Your ID scope here]";
      private static final String globalEndpoint = "[Your Provisioning Service Global Endpoint here]";
      private static final ProvisioningDeviceClientTransportProtocol PROVISIONING_DEVICE_CLIENT_TRANSPORT_PROTOCOL = ProvisioningDeviceClientTransportProtocol.HTTPS;
      private static final String leafPublicPem = "<Your Public PEM Certificate here>";
      private static final String leafPrivateKey = "<Your Private PEM Key here>";
      ```

   - Al copiar y pegar el certificado y la clave privada, utilice el siguiente formato:
        
      ```java
      private static final String leafPublicPem = "-----BEGIN CERTIFICATE-----\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "+XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "-----END CERTIFICATE-----\n";
      private static final String leafPrivateKey = "-----BEGIN PRIVATE KEY-----\n" +
            "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
            "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
            "XXXXXXXXXX\n" +
            "-----END PRIVATE KEY-----\n";
      ```

4. Compile el ejemplo. Vaya a la carpeta `target` y ejecute el archivo .jar que se creó.

    ```cmd/sh
    mvn clean install
    cd target
    java -jar ./provisioning-x509-sample-{version}-with-deps.jar
    ```

5. En Azure Portal, vaya a la instancia de IoT Hub vinculada a su servicio de aprovisionamiento y abra la hoja **Device Explorer**. Después del aprovisionamiento correcto del dispositivo simulado X.509 en el centro, su identificador de dispositivo aparece en la hoja **Device Explorer** con el *ESTADO* **habilitado**.  Es posible que deba presionar el botón **Actualizar** de la parte superior si ha abierto la hoja antes de ejecutar la aplicación de ejemplo del dispositivo. 

    ![El dispositivo se registra con el centro de IoT](./media/java-quick-create-simulated-device-x509/hubregistration.png) 

> [!NOTE]
> Si ha cambiado el valor predeterminado de *Estado inicial del dispositivo gemelo* en la entrada de inscripción para el dispositivo, el dispositivo puede extraer el estado gemelo deseado desde el centro y actuar en consecuencia. Para más información, consulte [Información y uso de dispositivos gemelos en IoT Hub](../iot-hub/iot-hub-devguide-device-twins.md).
>


## <a name="clean-up-resources"></a>Limpieza de recursos

Si planea seguir trabajando con el ejemplo de cliente de dispositivo y explorándolo, no limpie los recursos que se crean en este inicio rápido. Si no va a continuar, use el siguiente comando para eliminar todos los recursos que se han creado en este inicio rápido.

1. Cierre la ventana de salida de ejemplo del cliente del dispositivo en su máquina.
2. En el menú de la izquierda de Azure Portal, seleccione **Todos los recursos** y seleccione Device Provisioning Service. Abra la hoja **Administrar inscripciones** de su servicio y, después, seleccione la pestaña **Inscripciones individuales**. Active la casilla situada junto al campo *ID. DE REGISTRO* del dispositivo que ha inscrito en este inicio rápido y presione el botón **Eliminar** situado en la parte superior del panel. 
3. En el menú de la izquierda de Azure Portal, seleccione **Todos los recursos** y después su centro de IoT. Abra la hoja **Dispositivos IoT** de su centro, active la casilla junto al *identificador de dispositivo* del dispositivo que registró en este inicio rápido y, luego, presione el botón **Eliminar** situado en la parte superior del panel.


## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido ha creado un dispositivo X.509 simulado en una máquina Windows. Ha configurado su inscripción en el servicio Azure IoT Hub Device Provisioning y se ha realizado el aprovisionamiento automático del dispositivo en su centro de IoT. Para aprender a inscribir un dispositivo X.509 mediante programación, pase al inicio rápido para la inscripción de dispositivos X.509 mediante programación. 

> [!div class="nextstepaction"]
> [Inicio rápido de Azure: Inscripción de dispositivos X.509 en Azure IoT Hub Device Provisioning Service](quick-enroll-device-x509-java.md)
