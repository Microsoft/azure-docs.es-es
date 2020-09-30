---
title: Carga de archivos desde dispositivos a Azure IoT Hub con Java | Microsoft Docs
description: Cómo cargar archivos de un dispositivo a la nube mediante el SDK de dispositivo IoT de Azure para Java. Los archivos cargados se almacenan en un contenedor de blobs de Azure Storage.
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.devlang: java
ms.topic: conceptual
ms.date: 06/28/2017
ms.custom:
- amqp
- mqtt
- devx-track-java
ms.openlocfilehash: e6006444e933dd93467dde01affd29c5d0c4a146
ms.sourcegitcommit: 43558caf1f3917f0c535ae0bf7ce7fe4723391f9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2020
ms.locfileid: "90019553"
---
# <a name="upload-files-from-your-device-to-the-cloud-with-iot-hub-java"></a>Carga de archivos de un dispositivo a la nube con IoT Hub (Java)

[!INCLUDE [iot-hub-file-upload-language-selector](../../includes/iot-hub-file-upload-language-selector.md)]

Este tutorial se basa en el código del tutorial sobre el [envío de mensajes de la nube al dispositivo con IoT Hub](iot-hub-java-java-c2d.md) para mostrar cómo se usan las [funcionalidades de carga de archivos de IoT Hub](iot-hub-devguide-file-upload.md) para cargar un archivo en [Azure Blob Storage](../storage/index.yml). En este tutorial se muestra cómo realizar las siguientes acciones:

* Proporcionar un dispositivo de forma segura con un identificador URI de blob de Azure para cargar un archivo.

* Usar las notificaciones de carga de archivo de IoT Hub para desencadenar el procesamiento del archivo en el back-end de aplicación.

En el inicio rápido [Envío de telemetría de un dispositivo a IoT Hub](quickstart-send-telemetry-java.md) y el tutorial de [Envío de mensajes de la nube al dispositivo con IoT Hub](iot-hub-java-java-c2d.md) se muestra cómo usar la funcionalidad básica de mensajería del dispositivo a la nube y de la nube al dispositivo de IoT Hub. En el tutorial [Configuración del enrutamiento de mensajes con IoT Hub](tutorial-routing.md) se describe una forma de almacenar de manera confiable los mensajes enviados del dispositivo a la nube en Azure Blob Storage. Sin embargo, en algunos casos no se pueden asignar fácilmente los datos de que los dispositivos envían en los mensajes de dispositivo a nube con un tamaño relativamente reducido que acepta Azure IoT Hub. Por ejemplo:

* Archivos grandes con imágenes
* Vídeos
* Datos de vibración muestreados con alta frecuencia
* Algún tipo de datos de procesamiento previo.

Dichos archivos se suelen procesar por lotes en la nube mediante herramientas como [Azure Data Factory](../data-factory/introduction.md) o la pila [Hadoop](../hdinsight/index.yml). Cuando necesite cargar archivos desde un dispositivo, todavía puede usar la seguridad y confiabilidad de IoT Hub.

Al final de este tutorial, ejecutará dos aplicaciones de consola de Java:

* **simulated-device**, una versión modificada de la aplicación que creó en el tutorial [Envío de mensajes de la nube al dispositivo con IoT Hub]. Esta aplicación carga un archivo en el almacenamiento mediante un identificador URI de SAS proporcionado por el centro de IoT.

* **read-file-upload-notification**, que recibe notificaciones de carga de archivos de IoT Hub.

> [!NOTE]
> IoT Hub admite muchas plataformas de dispositivos y lenguajes (incluido C, .NET y JavaScript), mediante los SDK de dispositivo IoT de Azure. Consulte el [Centro para desarrolladores de IoT de Azure](https://azure.microsoft.com/develop/iot) para obtener instrucciones paso a paso sobre cómo conectar el dispositivo a Azure IoT Hub.

[!INCLUDE [iot-hub-include-x509-ca-signed-file-upload-support-note](../../includes/iot-hub-include-x509-ca-signed-file-upload-support-note.md)]

## <a name="prerequisites"></a>Requisitos previos

* [Java SE Development Kit 8](https://docs.microsoft.com/java/azure/jdk/?view=azure-java-stable). Asegúrese de seleccionar **Java 8** en **Long-term support** (Soporte técnico a largo plazo) para obtener descargas de JDK 8.

* [Maven 3](https://maven.apache.org/download.cgi)

* Una cuenta de Azure activa. (En caso de no tener ninguna, puede crear una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) en tan solo unos minutos).

* Asegúrese de que el puerto 8883 está abierto en el firewall. En el ejemplo de dispositivo de este artículo se usa el protocolo MQTT, que se comunica mediante el puerto 8883. Este puerto puede estar bloqueado en algunos entornos de red corporativos y educativos. Para más información y saber cómo solucionar este problema, consulte [Conexión a IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

[!INCLUDE [iot-hub-associate-storage](../../includes/iot-hub-associate-storage.md)]

## <a name="upload-a-file-from-a-device-app"></a>Carga de un archivo desde una aplicación de dispositivo

En esta sección, modificará la aplicación de dispositivo que creó en [Envío de mensajes de la nube al dispositivo con IoT Hub](iot-hub-java-java-c2d.md) para cargar un archivo en el centro de IoT.

1. Copie un archivo de imagen a la carpeta `simulated-device` llámelo `myimage.png`.

2. Con un editor de texto, abra el archivo `simulated-device\src\main\java\com\mycompany\app\App.java`.

3. Agregue la declaración de variable a la clase **App**:

    ```java
    private static String fileName = "myimage.png";
    ```

4. Para procesar mensajes de devolución de llamada de estado de carga de archivos, agregue la siguiente clase anidada a la clase **App**:

    ```java
    // Define a callback method to print status codes from IoT Hub.
    protected static class FileUploadStatusCallBack implements IotHubEventCallback {
      public void execute(IotHubStatusCode status, Object context) {
        System.out.println("IoT Hub responded to file upload for " + fileName
            + " operation with status " + status.name());
      }
    }
    ```

5. Para cargar imágenes en IoT Hub, agregue el método siguiente a la clase **App**:

    ```java
    // Use IoT Hub to upload a file asynchronously to Azure blob storage.
    private static void uploadFile(String fullFileName) throws FileNotFoundException, IOException
    {
      File file = new File(fullFileName);
      InputStream inputStream = new FileInputStream(file);
      long streamLength = file.length();

      client.uploadToBlobAsync(fileName, inputStream, streamLength, new FileUploadStatusCallBack(), null);
    }
    ```

6. Modifique el método **principal** para llamar al método **uploadFile** tal como se muestra en el fragmento de código siguiente:

    ```java
    client.open();

    try
    {
      // Get the filename and start the upload.
      String fullFileName = System.getProperty("user.dir") + File.separator + fileName;
      uploadFile(fullFileName);
      System.out.println("File upload started with success");
    }
    catch (Exception e)
    {
      System.out.println("Exception uploading file: " + e.getCause() + " \nERROR: " + e.getMessage());
    }

    MessageSender sender = new MessageSender();
    ```

7. Use el siguiente comando para compilar la aplicación **simulated-device** y busque errores:

    ```cmd/sh
    mvn clean package -DskipTests
    ```

## <a name="get-the-iot-hub-connection-string"></a>Obtención de la cadena de conexión de IoT Hub

En este artículo, creará un servicio de back-end para recibir mensajes de notificación de carga de archivos desde la instancia de IOT Hub que creó en [Enviar telemetría desde un dispositivo a IoT Hub](quickstart-send-telemetry-java.md). Para recibir mensajes de notificación de carga de archivos, el servicio necesita el permiso de **conexión de servicio**. De forma predeterminada, todas las instancias de IoT Hub se crean con una directiva de acceso compartido denominada **servicio** que concede este permiso.

[!INCLUDE [iot-hub-include-find-service-connection-string](../../includes/iot-hub-include-find-service-connection-string.md)]

## <a name="receive-a-file-upload-notification"></a>Recepción de una notificación de carga de archivos

En esta sección, se crea una aplicación de consola de Java que recibe mensajes de notificación de carga de archivos de IoT Hub.

1. Cree un proyecto de Maven denominado **read-file-upload-notification** mediante el siguiente comando en el símbolo del sistema. Observe que este es un comando único y largo:

    ```cmd/sh
    mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=read-file-upload-notification -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    ```

2. En el símbolo del sistema, vaya a la carpeta `read-file-upload-notification` nueva.

3. Con un editor de texto, abra el archivo `pom.xml` en la carpeta `read-file-upload-notification` y agregue la dependencia siguiente al nodo **dependencies**. Esto le permite usar el paquete **iothub-java-service-client** en la aplicación para comunicarse con el servicio IoT Hub:

    ```xml
    <dependency>
      <groupId>com.microsoft.azure.sdk.iot</groupId>
      <artifactId>iot-service-client</artifactId>
      <version>1.7.23</version>
    </dependency>
    ```

    > [!NOTE]
    > Puede comprobar la versión más reciente de **iot-service-client** mediante la [búsqueda de Maven](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22iot-service-client%22%20g%3A%22com.microsoft.azure.sdk.iot%22).

4. Guarde y cierre el archivo `pom.xml`.

5. Con un editor de texto, abra el archivo `read-file-upload-notification\src\main\java\com\mycompany\app\App.java`.

6. Agregue las siguientes instrucciones **import** al archivo:

    ```java
    import com.microsoft.azure.sdk.iot.service.*;
    import java.io.IOException;
    import java.net.URISyntaxException;
    import java.util.concurrent.ExecutorService;
    import java.util.concurrent.Executors;
    ```

7. Agregue las siguientes variables de nivel de clase a la clase **App** . Reemplace el valor de marcador de posición `{Your IoT Hub connection string}` por la cadena de conexión de IoT Hub que copió anteriormente en [Obtener la cadena de conexión de IoT Hub](#get-the-iot-hub-connection-string).

    ```java
    private static final String connectionString = "{Your IoT Hub connection string}";
    private static final IotHubServiceClientProtocol protocol = IotHubServiceClientProtocol.AMQPS;
    private static FileUploadNotificationReceiver fileUploadNotificationReceiver = null;
    ```

8. Para imprimir la información acerca de la carga de archivos en la consola, agregue la siguiente clase anidada para la clase **App**:

    ```java
    // Create a thread to receive file upload notifications.
    private static class ShowFileUploadNotifications implements Runnable {
      public void run() {
        try {
          while (true) {
            System.out.println("Receive file upload notifications...");
            FileUploadNotification fileUploadNotification = fileUploadNotificationReceiver.receive();
            if (fileUploadNotification != null) {
              System.out.println("File Upload notification received");
              System.out.println("Device Id : " + fileUploadNotification.getDeviceId());
              System.out.println("Blob Uri: " + fileUploadNotification.getBlobUri());
              System.out.println("Blob Name: " + fileUploadNotification.getBlobName());
              System.out.println("Last Updated : " + fileUploadNotification.getLastUpdatedTimeDate());
              System.out.println("Blob Size (Bytes): " + fileUploadNotification.getBlobSizeInBytes());
              System.out.println("Enqueued Time: " + fileUploadNotification.getEnqueuedTimeUtcDate());
            }
          }
        } catch (Exception ex) {
          System.out.println("Exception reading reported properties: " + ex.getMessage());
        }
      }
    }
    ```

9. Para iniciar el subproceso que realiza escuchas para las notificaciones de carga de archivos, agregue el código siguiente al método **principal**:

    ```java
    public static void main(String[] args) throws IOException, URISyntaxException, Exception {
      ServiceClient serviceClient = ServiceClient.createFromConnectionString(connectionString, protocol);

      if (serviceClient != null) {
        serviceClient.open();

        // Get a file upload notification receiver from the ServiceClient.
        fileUploadNotificationReceiver = serviceClient.getFileUploadNotificationReceiver();
        fileUploadNotificationReceiver.open();

        // Start the thread to receive file upload notifications.
        ShowFileUploadNotifications showFileUploadNotifications = new ShowFileUploadNotifications();
        ExecutorService executor = Executors.newFixedThreadPool(1);
        executor.execute(showFileUploadNotifications);

        System.out.println("Press ENTER to exit.");
        System.in.read();
        executor.shutdownNow();
        System.out.println("Shutting down sample...");
        fileUploadNotificationReceiver.close();
        serviceClient.close();
      }
    }
    ```

10. Guarde y cierre el archivo `read-file-upload-notification\src\main\java\com\mycompany\app\App.java`.

11. Use el siguiente comando para compilar la aplicación **read-file-upload-notification** y busque errores:

    ```cmd/sh
    mvn clean package -DskipTests
    ```

## <a name="run-the-applications"></a>Ejecución de las aplicaciones

Ahora está preparado para ejecutar las aplicaciones.

En la carpeta `read-file-upload-notification` del símbolo del sistema, ejecute el siguiente comando:

```cmd/sh
mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
```

En la carpeta `simulated-device` del símbolo del sistema, ejecute el siguiente comando:

```cmd/sh
mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
```

En la captura de pantalla siguiente se muestra la salida de la aplicación **simulated-device**:

![Salida de la aplicación simulated-device](media/iot-hub-java-java-upload/simulated-device.png)

En la captura de pantalla siguiente se muestra la salida de la aplicación **read-file-upload-notification**:

![Salida de la aplicación read-file-upload-notification](media/iot-hub-java-java-upload/read-file-upload-notification.png)

Puede usar el portal para ver el archivo cargado en el contenedor de almacenamiento que configuró:

![Archivo cargado](media/iot-hub-java-java-upload/uploaded-file.png)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha aprendido a usar la funcionalidad de carga de archivos de IoT Hub para simplificar la carga de archivos desde los dispositivos. Puede continuar explorando las características y los escenarios del centro de IoT con los siguientes artículos:

* [Creación de un centro de IoT mediante programación](iot-hub-rm-template-powershell.md)

* [Introducción a SDK para C](iot-hub-device-sdk-c-intro.md)

* [SDK de IoT de Azure](iot-hub-devguide-sdks.md)

Para explorar aún más las funcionalidades de IoT Hub, consulte:

* [Simulación de un dispositivo con IoT Edge](../iot-edge/tutorial-simulate-device-linux.md)
