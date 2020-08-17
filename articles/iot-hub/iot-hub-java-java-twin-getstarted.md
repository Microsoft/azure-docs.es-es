---
title: Introducción a los dispositivos gemelos de Azure IoT Hub (Java) | Microsoft Docs
description: Describe cómo usar dispositivos gemelos de IoT Hub de Azure para agregar etiquetas y, luego, usar una consulta de IoT Hub. El SDK de dispositivos de Azure IoT para Java se usa con el fin de implementar la aplicación de dispositivo y el SDK de servicios Azure IoT para Java con el objetivo de implementar una aplicación de servicio que agrega las etiquetas y ejecuta la consulta de IoT Hub.
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.devlang: java
ms.topic: conceptual
ms.date: 08/26/2019
ms.custom: mqtt, devx-track-java
ms.openlocfilehash: c439f1a5aaa03774652dc815364c45633096f6de
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87327606"
---
# <a name="get-started-with-device-twins-java"></a>Introducción a los dispositivos gemelos (Java)

[!INCLUDE [iot-hub-selector-twin-get-started](../../includes/iot-hub-selector-twin-get-started.md)]

En este tutorial, creará dos aplicaciones de consola de Java:

* **add-tags-query**, una aplicación de back-end de Java que agrega etiquetas y consulta dispositivos gemelos.
* **simulated-device**, una aplicación de dispositivo de Java que se conecta a IoT Hub y notifica su condición de conectividad mediante una propiedad notificada.

> [!NOTE]
> En el artículo [SDK de IoT de Azure](iot-hub-devguide-sdks.md) se proporciona información sobre los SDK de IoT de Azure que se pueden usar para crear aplicaciones de dispositivo y de back-end.

## <a name="prerequisites"></a>Requisitos previos

* [Java SE Development Kit 8](https://docs.microsoft.com/java/azure/jdk/?view=azure-java-stable). Asegúrese de seleccionar **Java 8** en **Long-term support** (Soporte técnico a largo plazo) para obtener descargas de JDK 8.

* [Maven 3](https://maven.apache.org/download.cgi)

* Una cuenta de Azure activa. (En caso de no tener ninguna, puede crear una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) en tan solo unos minutos).

* Asegúrese de que el puerto 8883 está abierto en el firewall. En el ejemplo de dispositivo de este artículo se usa el protocolo MQTT, que se comunica mediante el puerto 8883. Este puerto puede estar bloqueado en algunos entornos de red corporativos y educativos. Para más información y saber cómo solucionar este problema, consulte [Conexión a IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="create-an-iot-hub"></a>Crear un centro de IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="register-a-new-device-in-the-iot-hub"></a>Registro de un nuevo dispositivo en el centro de IoT

[!INCLUDE [iot-hub-include-create-device](../../includes/iot-hub-include-create-device.md)]

## <a name="get-the-iot-hub-connection-string"></a>Obtención de la cadena de conexión de IoT Hub

[!INCLUDE [iot-hub-howto-twin-shared-access-policy-text](../../includes/iot-hub-howto-twin-shared-access-policy-text.md)]

[!INCLUDE [iot-hub-include-find-custom-connection-string](../../includes/iot-hub-include-find-custom-connection-string.md)]

## <a name="create-the-service-app"></a>Creación de la aplicación de servicio

En esta sección, creará una aplicación de Java que agrega metadatos de ubicación como una etiqueta al dispositivo gemelo en IoT Hub asociado con **myDeviceId**. La aplicación consulta primero IoT Hub para los dispositivos que se encuentran en Estados Unidos y, después, para los que notifican una conexión de red de telefonía móvil.

1. En el equipo de desarrollo, cree una carpeta vacía denominada **iot-java-twin-getstarted**.

2. En la carpeta **iot-java-twin-getstarted**, cree un proyecto de Maven denominado **add-tags-query** mediante el siguiente comando en el símbolo del sistema:

    ```cmd/sh
    mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=add-tags-query -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    ```

3. En el símbolo del sistema, vaya a la carpeta **add-tags-query**.

4. Con un editor de texto, abra el archivo **pom.xml** en la carpeta **add-tags-query** y agregue la dependencia siguiente al nodo **dependencies**. Esta dependencia permite usar el paquete **iot-service-client** en la aplicación para comunicarse con la instancia de IoT Hub:

    ```xml
    <dependency>
      <groupId>com.microsoft.azure.sdk.iot</groupId>
      <artifactId>iot-service-client</artifactId>
      <version>1.17.1</version>
      <type>jar</type>
    </dependency>
    ```

    > [!NOTE]
    > Puede comprobar la versión más reciente de **iot-service-client** mediante la [búsqueda de Maven](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22iot-service-client%22%20g%3A%22com.microsoft.azure.sdk.iot%22).

5. Agregue el nodo **build** después del nodo **dependencies**. Esta configuración indica a Maven que use Java 1.8 para compilar la aplicación.

    ```xml
    <build>
      <plugins>
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-compiler-plugin</artifactId>
          <version>3.3</version>
          <configuration>
            <source>1.8</source>
            <target>1.8</target>
          </configuration>
        </plugin>
      </plugins>
    </build>
    ```

6. Guarde y cierre el archivo **pom.xml**.

7. Con un editor de texto, abra el archivo **add-tags-query\src\main\java\com\mycompany\app\App.java**.

8. Agregue las siguientes instrucciones **import** al archivo:

    ```java
    import com.microsoft.azure.sdk.iot.service.devicetwin.*;
    import com.microsoft.azure.sdk.iot.service.exceptions.IotHubException;

    import java.io.IOException;
    import java.util.HashSet;
    import java.util.Set;
    ```

9. Agregue las siguientes variables de nivel de clase a la clase **App** . Reemplace `{youriothubconnectionstring}` por la cadena de conexión de IoT Hub que copió en [Obtención de la cadena de conexión de IoT Hub](#get-the-iot-hub-connection-string).

    ```java
    public static final String iotHubConnectionString = "{youriothubconnectionstring}";
    public static final String deviceId = "myDeviceId";

    public static final String region = "US";
    public static final String plant = "Redmond43";
    ```

10. Actualice la firma del método **main** para incluir la cláusula `throws` siguiente:

    ```java
    public static void main( String[] args ) throws IOException
    ```

11. Reemplace el código del método **main** por el código siguiente para crear los objetos **DeviceTwin** y **DeviceTwinDevice**. El objeto **DeviceTwin** administra la comunicación con el IoT Hub. El objeto **DeviceTwinDevice** representa el dispositivo gemelo con sus propiedades y etiquetas:

    ```java
    // Get the DeviceTwin and DeviceTwinDevice objects
    DeviceTwin twinClient = DeviceTwin.createFromConnectionString(iotHubConnectionString);
    DeviceTwinDevice device = new DeviceTwinDevice(deviceId);
    ```

12. Agregue el siguiente bloque `try/catch` al método **main**:

    ```java
    try {
      // Code goes here
    } catch (IotHubException e) {
      System.out.println(e.getMessage());
    } catch (IOException e) {
      System.out.println(e.getMessage());
    }
    ```

13. Para actualizar las etiquetas de dispositivo gemelo **region** y **plant** en el dispositivo gemelo, agregue el código siguiente en el bloque `try`:

    ```java
    // Get the device twin from IoT Hub
    System.out.println("Device twin before update:");
    twinClient.getTwin(device);
    System.out.println(device);

    // Update device twin tags if they are different
    // from the existing values
    String currentTags = device.tagsToString();
    if ((!currentTags.contains("region=" + region) && !currentTags.contains("plant=" + plant))) {
      // Create the tags and attach them to the DeviceTwinDevice object
      Set<Pair> tags = new HashSet<Pair>();
      tags.add(new Pair("region", region));
      tags.add(new Pair("plant", plant));
      device.setTags(tags);

      // Update the device twin in IoT Hub
      System.out.println("Updating device twin");
      twinClient.updateTwin(device);
    }

    // Retrieve the device twin with the tag values from IoT Hub
    System.out.println("Device twin after update:");
    twinClient.getTwin(device);
    System.out.println(device);
    ```

14. Para consultar los dispositivos gemelos en IoT Hub, agregue el código siguiente al bloque `try` después del código que agregó en el paso anterior. El código ejecuta dos consultas. Cada consulta devuelve un máximo de 100 dispositivos.

    ```java
    // Query the device twins in IoT Hub
    System.out.println("Devices in Redmond:");

    // Construct the query
    SqlQuery sqlQuery = SqlQuery.createSqlQuery("*", SqlQuery.FromType.DEVICES, "tags.plant='Redmond43'", null);

    // Run the query, returning a maximum of 100 devices
    Query twinQuery = twinClient.queryTwin(sqlQuery.getQuery(), 100);
    while (twinClient.hasNextDeviceTwin(twinQuery)) {
      DeviceTwinDevice d = twinClient.getNextDeviceTwin(twinQuery);
      System.out.println(d.getDeviceId());
    }

    System.out.println("Devices in Redmond using a cellular network:");

    // Construct the query
    sqlQuery = SqlQuery.createSqlQuery("*", SqlQuery.FromType.DEVICES, "tags.plant='Redmond43' AND properties.reported.connectivityType = 'cellular'", null);

    // Run the query, returning a maximum of 100 devices
    twinQuery = twinClient.queryTwin(sqlQuery.getQuery(), 3);
    while (twinClient.hasNextDeviceTwin(twinQuery)) {
      DeviceTwinDevice d = twinClient.getNextDeviceTwin(twinQuery);
      System.out.println(d.getDeviceId());
    }
    ```

15. Guarde y cierre el archivo **add-tags-query\src\main\java\com\mycompany\app\App.java**.

16. Compile la aplicación **add-tags-query** y corrija los errores. En el símbolo del sistema, vaya a la carpeta **add-tags-query** y ejecute el comando siguiente:

    ```cmd/sh
    mvn clean package -DskipTests
    ```

## <a name="create-a-device-app"></a>Creación de una aplicación de dispositivo

En esta sección, creará una aplicación de consola de Java que establece un valor de propiedad notificada que se envía a IoT Hub.

1. En la carpeta**iot-java-twin-getstarted**, cree un proyecto de Maven denominado **simulated-device** mediante el siguiente comando en el símbolo del sistema:

    ```cmd/sh
    mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=simulated-device -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    ```

2. En el símbolo del sistema, vaya a la nueva carpeta **simulated-device**.

3. Con un editor de texto, abra el archivo **pom.xml** de la carpeta **simulated-device** y agregue las siguientes dependencias al nodo **dependencies**. Esta dependencia le permite usar el paquete **iot-device-client** en la aplicación para comunicarse con la instancia de IoT Hub.

    ```xml
    <dependency>
      <groupId>com.microsoft.azure.sdk.iot</groupId>
      <artifactId>iot-device-client</artifactId>
      <version>1.17.5</version>
    </dependency>
    ```

    > [!NOTE]
    > Puede comprobar la versión más reciente de **iot-device-client** mediante la [búsqueda de Maven](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22iot-device-client%22%20g%3A%22com.microsoft.azure.sdk.iot%22).

4. Agregue la siguiente dependencia al nodo **dependencies**. Esta dependencia configura una instrucción NOP para la fachada de registro de Apache [SLF4J](https://www.slf4j.org/), que usa el SDK de cliente de dispositivo para implementar el registro. Esta configuración es opcional, pero si la omite, es posible que vea una advertencia en la consola al ejecutar la aplicación. Para más información sobre el registro en el SDK de cliente de dispositivo, consulte [Registro](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-samples/readme.md#logging) en el archivo Léame *Ejemplos para el SDK de dispositivo IoT de Azure para Java*.

    ```xml
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-nop</artifactId>
      <version>1.7.28</version>
    </dependency>
    ```

5. Agregue el nodo **build** después del nodo **dependencies**. Esta configuración indica a Maven que use Java 1.8 para compilar la aplicación:

    ```xml
    <build>
      <plugins>
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-compiler-plugin</artifactId>
          <version>3.3</version>
          <configuration>
            <source>1.8</source>
            <target>1.8</target>
          </configuration>
        </plugin>
      </plugins>
    </build>
    ```

6. Guarde y cierre el archivo **pom.xml**.

7. Con un editor de texto, abra el archivo **simulated-device\src\main\java\com\mycompany\app\App.java**.

8. Agregue las siguientes instrucciones **import** al archivo:

    ```java
    import com.microsoft.azure.sdk.iot.device.*;
    import com.microsoft.azure.sdk.iot.device.DeviceTwin.*;

    import java.io.IOException;
    import java.net.URISyntaxException;
    import java.util.Scanner;
    ```

9. Agregue las siguientes variables de nivel de clase a la clase **App** . Reemplace `{yourdeviceconnectionstring}` por la cadena de conexión de dispositivo que copió en [Registro de un nuevo dispositivo en el centro de IoT](#register-a-new-device-in-the-iot-hub).

    ```java
    private static String connString = "{yourdeviceconnectionstring}";
    private static IotHubClientProtocol protocol = IotHubClientProtocol.MQTT;
    private static String deviceId = "myDeviceId";
    ```

    Esta aplicación de ejemplo usa la variable **protocol** al crear una instancia de un objeto **DeviceClient**.

10. Agregue el método siguiente a la clase **App** para imprimir información sobre las actualizaciones de dispositivos gemelos:

    ```java
    protected static class DeviceTwinStatusCallBack implements IotHubEventCallback {
        @Override
        public void execute(IotHubStatusCode status, Object context) {
          System.out.println("IoT Hub responded to device twin operation with status " + status.name());
        }
      }
    ```

11. Reemplace el código del método **main** por el código siguiente:

    * Cree un cliente de dispositivo para comunicarse con IoT Hub.

    * Cree un objeto **Device** para almacenar las propiedades del dispositivo gemelo.

    ```java
    DeviceClient client = new DeviceClient(connString, protocol);

    // Create a Device object to store the device twin properties
    Device dataCollector = new Device() {
      // Print details when a property value changes
      @Override
      public void PropertyCall(String propertyKey, Object propertyValue, Object context) {
        System.out.println(propertyKey + " changed to " + propertyValue);
      }
    };
    ```

12. Agregue el código siguiente al método **main** para crear una propiedad notificada **connectivityType** y enviarla a IoT Hub:

    ```java
    try {
      // Open the DeviceClient and start the device twin services.
      client.open();
      client.startDeviceTwin(new DeviceTwinStatusCallBack(), null, dataCollector, null);

      // Create a reported property and send it to your IoT hub.
      dataCollector.setReportedProp(new Property("connectivityType", "cellular"));
      client.sendReportedProperties(dataCollector.getReportedProp());
    }
    catch (Exception e) {
      System.out.println("On exception, shutting down \n" + " Cause: " + e.getCause() + " \n" + e.getMessage());
      dataCollector.clean();
      client.closeNow();
      System.out.println("Shutting down...");
    }
    ```

13. Agregue el siguiente código al final del método **main**. Esperar a la clave **Enter** da tiempo a IoT Hub a informar del estado de las operaciones del dispositivo gemelo.

    ```java
    System.out.println("Press any key to exit...");

    Scanner scanner = new Scanner(System.in);
    scanner.nextLine();

    dataCollector.clean();
    client.close();
    ```

14. Modifique la firma del método **main** para incluir las excepciones de la siguiente manera:

     ```java
     public static void main(String[] args) throws URISyntaxException, IOException
     ```

15. Guarde y cierre el archivo **simulated-device\src\main\java\com\mycompany\app\App.java**.

16. Compile la aplicación **simulated-device** y corrija los errores. En el símbolo del sistema, vaya a la carpeta **simulated-device** y ejecute el comando siguiente:

    ```cmd/sh
    mvn clean package -DskipTests
    ```

## <a name="run-the-apps"></a>Ejecución de las aplicaciones

Ya está preparado para ejecutar las aplicaciones de consola.

1. En un símbolo del sistema en la carpeta **add-tags-query**, ejecute el siguiente comando para ejecutar la aplicación de servicio **add-tags-query**:

    ```cmd/sh
    mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
    ```

    ![Aplicación de servicio de IoT Hub de Java para actualizar valores de etiqueta y ejecutar consultas de dispositivo](./media/iot-hub-java-java-twin-getstarted/service-app-1.png)

    Puede ver las etiquetas **plant** y **region** agregadas al dispositivo gemelo. La primera consulta devuelve el dispositivo, pero la segunda no.

2. En un símbolo del sistema en la carpeta **simulated-device**, ejecute el siguiente comando para agregar la propiedad notificada **connectivityType** al dispositivo gemelo:

    ```cmd/sh
    mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
    ```

    ![El cliente de dispositivo agrega la propiedad notificada **connectivityType**](./media/iot-hub-java-java-twin-getstarted/device-app-1.png)

3. En un símbolo del sistema en la carpeta **add-tags-query**, ejecute el siguiente comando para ejecutar la aplicación de servicio **add-tags-query** una segunda vez:

    ```cmd/sh
    mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
    ```

    ![Aplicación de servicio de IoT Hub de Java para actualizar valores de etiqueta y ejecutar consultas de dispositivo](./media/iot-hub-java-java-twin-getstarted/service-app-2.png)

    Ahora que el dispositivo ha enviado la propiedad **connectivityType** a IoT Hub, la segunda consulta devuelve el dispositivo.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, configuró un centro de IoT nuevo en Azure Portal y, después, creó una identidad de dispositivo en el registro de identidades del centro de IoT. Ha agregado metadatos de dispositivo como etiquetas desde una aplicación de back-end y ha escrito una aplicación de dispositivo para notificar la información de conectividad de dispositivos en el dispositivo gemelo. También ha obtenido información sobre cómo consultar la información del dispositivo gemelo mediante el lenguaje de consulta de IoT Hub de tipo SQL.

Use los siguientes recursos para obtener información sobre cómo:

* Enviar telemetría desde dispositivos con el tutorial [Introducción a IoT Hub](quickstart-send-telemetry-java.md).

* Controlar los dispositivos de forma interactiva (por ejemplo, encender un ventilador desde una aplicación controlada por el usuario), con el tutorial [Uso de métodos directos](quickstart-control-device-java.md).
