---
title: Programación de trabajos con Azure IoT Hub (Java) | Microsoft Docs
description: Cómo programar un trabajo de Azure IoT Hub para invocar un método directo y definir una propiedad deseada en varios dispositivos. Usará el SDK de dispositivo IoT de Azure para Java con el fin de implementar las aplicaciones de dispositivo simulado, además del SDK de servicio IoT de Azure para Java con el objetivo de implementar una aplicación de servicio para ejecutar el trabajo.
author: wesmc7777
manager: philmea
ms.author: wesmc
ms.service: iot-hub
services: iot-hub
ms.devlang: java
ms.topic: conceptual
ms.date: 08/16/2019
ms.custom: mqtt, devx-track-java
ms.openlocfilehash: c1cac67d79363ce25fb55d73cc3d8c42ea901fb1
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87325090"
---
# <a name="schedule-and-broadcast-jobs-java"></a>Programación y difusión de trabajos (Java)

[!INCLUDE [iot-hub-selector-schedule-jobs](../../includes/iot-hub-selector-schedule-jobs.md)]

Use Azure IoT Hub para programar y realizar el seguimiento de los trabajos que actualizan millones de dispositivos. Use los trabajos para:

* Actualizar las propiedades deseadas
* Actualizar etiquetas
* Invocar métodos directos

Un trabajo encapsula una de estas acciones y realiza el seguimiento de la ejecución en un conjunto de dispositivos. Una consulta de dispositivo gemelo define el conjunto de dispositivos en que se ejecuta el trabajo. Por ejemplo, una aplicación de back-end puede utilizar un trabajo para invocar un método directo en 10 000 dispositivos que reinicie los dispositivos. Especifique el conjunto de dispositivos con una consulta de dispositivo gemelo y programe el trabajo para que se ejecute en otro momento. Este trabajo realiza el seguimiento del progreso mientras los dispositivos reciben y ejecutan el método directo de reinicio.

Para más información sobre estas funcionalidades, vea:

* Dispositivo gemelo y propiedades: [Introducción a los dispositivos gemelos](iot-hub-java-java-twin-getstarted.md)

* Métodos directos: [Guía para desarrolladores de IoT Hub: métodos directos](iot-hub-devguide-direct-methods.md) y [Tutorial: Uso de métodos directos](quickstart-control-device-java.md)

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

En este tutorial se muestra cómo realizar las siguientes acciones:

* Creación de una aplicación de dispositivo que implemente un método directo denominado **lockDoor**. La aplicación de dispositivo también recibe cambios en la propiedad deseada de la aplicación de back-end.

* Creación de una aplicación de back-end que cree un trabajo para llamar al método directo **lockDoor** en varios dispositivos. Otro trabajo envía las actualizaciones en la propiedad deseada a varios dispositivos.

Al final de este tutorial, tendrá una aplicación de dispositivo de consola de Java y una aplicación de back-end de consola de Java:

**simulated-device**, que se conecta a la instancia de IoT Hub, implementa el método directo **lockDoor** y controla los cambios en la propiedad deseada.

**schedule-jobs**, que usa trabajos para llamar al método directo **lockDoor** y actualizar las propiedades del dispositivo gemelo deseadas en varios dispositivos.

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

También puede usar la herramienta [extensión de IoT para la CLI de Azure](https://github.com/Azure/azure-iot-cli-extension) para agregar un dispositivo a su instancia de IoT Hub.

## <a name="get-the-iot-hub-connection-string"></a>Obtención de la cadena de conexión de IoT Hub

[!INCLUDE [iot-hub-howto-schedule-jobs-shared-access-policy-text](../../includes/iot-hub-howto-schedule-jobs-shared-access-policy-text.md)]

[!INCLUDE [iot-hub-include-find-registryrw-connection-string](../../includes/iot-hub-include-find-registryrw-connection-string.md)]

## <a name="create-the-service-app"></a>Creación de la aplicación de servicio

En esta sección, creará una aplicación de consola de Java que usa trabajos para:

* Llamar al método directo **lockDoor** en varios dispositivos.

* Usar las propiedades deseadas en varios dispositivos.

Para crear la aplicación:

1. En el equipo de desarrollo, cree una carpeta vacía denominada **iot-java-schedule-jobs**.

2. En la carpeta **iot-java-schedule-jobs**, cree un proyecto de Maven denominado **schedule-jobs** mediante el siguiente comando en el símbolo del sistema. Observe que este es un comando único y largo:

   ```cmd/sh
   mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=schedule-jobs -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
   ```

3. En el símbolo del sistema, vaya a la carpeta **schedule-jobs**.

4. Con un editor de texto, abra el archivo **pom.xml** en la carpeta **schedule-jobs** y agregue la dependencia siguiente al nodo **dependencies**. Esta dependencia permite usar el paquete **iot-service-client** en la aplicación para comunicarse con la instancia de IoT Hub:

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

7. Con un editor de texto, abra el archivo **schedule-jobs\src\main\java\com\mycompany\app\App.java**.

8. Agregue las siguientes instrucciones **import** al archivo:

    ```java
    import com.microsoft.azure.sdk.iot.service.devicetwin.DeviceTwinDevice;
    import com.microsoft.azure.sdk.iot.service.devicetwin.Pair;
    import com.microsoft.azure.sdk.iot.service.devicetwin.Query;
    import com.microsoft.azure.sdk.iot.service.devicetwin.SqlQuery;
    import com.microsoft.azure.sdk.iot.service.jobs.JobClient;
    import com.microsoft.azure.sdk.iot.service.jobs.JobResult;
    import com.microsoft.azure.sdk.iot.service.jobs.JobStatus;

    import java.util.Date;
    import java.time.Instant;
    import java.util.HashSet;
    import java.util.Set;
    import java.util.UUID;
    ```

9. Agregue las siguientes variables de nivel de clase a la clase **App** . Reemplace `{youriothubconnectionstring}` por la cadena de conexión de IoT Hub que copió anteriormente en [Obtención de la cadena de conexión de IoT Hub](#get-the-iot-hub-connection-string):

    ```java
    public static final String iotHubConnectionString = "{youriothubconnectionstring}";
    public static final String deviceId = "myDeviceId";

    // How long the job is permitted to run without
    // completing its work on the set of devices
    private static final long maxExecutionTimeInSeconds = 30;
    ```

10. Agregue el método siguiente a la clase **App** para programar un trabajo de actualización de las propiedades **Building** y **Floor** deseadas en el dispositivo gemelo:

    ```java
    private static JobResult scheduleJobSetDesiredProperties(JobClient jobClient, String jobId) {
      DeviceTwinDevice twin = new DeviceTwinDevice(deviceId);
      Set<Pair> desiredProperties = new HashSet<Pair>();
      desiredProperties.add(new Pair("Building", 43));
      desiredProperties.add(new Pair("Floor", 3));
      twin.setDesiredProperties(desiredProperties);
      // Optimistic concurrency control
      twin.setETag("*");

      // Schedule the update twin job to run now
      // against a single device
      System.out.println("Schedule job " + jobId + " for device " + deviceId);
      try {
        JobResult jobResult = jobClient.scheduleUpdateTwin(jobId, 
          "deviceId='" + deviceId + "'",
          twin,
          new Date(),
          maxExecutionTimeInSeconds);
        return jobResult;
      } catch (Exception e) {
        System.out.println("Exception scheduling desired properties job: " + jobId);
        System.out.println(e.getMessage());
        return null;
      }
    }
    ```

11. Para programar un trabajo de llamada al método **lockDoor**, agregue el método siguiente a la clase **App**:

    ```java
    private static JobResult scheduleJobCallDirectMethod(JobClient jobClient, String jobId) {
      // Schedule a job now to call the lockDoor direct method
      // against a single device. Response and connection
      // timeouts are set to 5 seconds.
      System.out.println("Schedule job " + jobId + " for device " + deviceId);
      try {
        JobResult jobResult = jobClient.scheduleDeviceMethod(jobId,
          "deviceId='" + deviceId + "'",
          "lockDoor",
          5L, 5L, null,
          new Date(),
          maxExecutionTimeInSeconds);
        return jobResult;
      } catch (Exception e) {
        System.out.println("Exception scheduling direct method job: " + jobId);
        System.out.println(e.getMessage());
        return null;
      }
    };
    ```

12. Para supervisar un trabajo, agregue el siguiente método a la clase **App**:

    ```java
    private static void monitorJob(JobClient jobClient, String jobId) {
      try {
        JobResult jobResult = jobClient.getJob(jobId);
        if(jobResult == null)
        {
          System.out.println("No JobResult for: " + jobId);
          return;
        }
        // Check the job result until it's completed
        while(jobResult.getJobStatus() != JobStatus.completed)
        {
          Thread.sleep(100);
          jobResult = jobClient.getJob(jobId);
          System.out.println("Status " + jobResult.getJobStatus() + " for job " + jobId);
        }
        System.out.println("Final status " + jobResult.getJobStatus() + " for job " + jobId);
      } catch (Exception e) {
        System.out.println("Exception monitoring job: " + jobId);
        System.out.println(e.getMessage());
        return;
      }
    }
    ```

13. Para consultar los detalles de los trabajos ejecutados, agregue el método siguiente:

    ```java
    private static void queryDeviceJobs(JobClient jobClient, String start) throws Exception {
      System.out.println("\nQuery device jobs since " + start);

      // Create a jobs query using the time the jobs started
      Query deviceJobQuery = jobClient
          .queryDeviceJob(SqlQuery.createSqlQuery("*", SqlQuery.FromType.JOBS, "devices.jobs.startTimeUtc > '" + start + "'", null).getQuery());

      // Iterate over the list of jobs and print the details
      while (jobClient.hasNextJob(deviceJobQuery)) {
        System.out.println(jobClient.getNextJob(deviceJobQuery));
      }
    }
    ```

14. Actualice la firma del método **main** para incluir la cláusula `throws` siguiente:

    ```java
    public static void main( String[] args ) throws Exception
    ```

15. Para ejecutar y supervisar dos trabajos de forma secuencial, reemplace el código del método **main** con lo siguiente:

    ```java
    // Record the start time
    String start = Instant.now().toString();

    // Create JobClient
    JobClient jobClient = JobClient.createFromConnectionString(iotHubConnectionString);
    System.out.println("JobClient created with success");

    // Schedule twin job desired properties
    // Maximum concurrent jobs is 1 for Free and S1 tiers
    String desiredPropertiesJobId = "DPCMD" + UUID.randomUUID();
    scheduleJobSetDesiredProperties(jobClient, desiredPropertiesJobId);
    monitorJob(jobClient, desiredPropertiesJobId);

    // Schedule twin job direct method
    String directMethodJobId = "DMCMD" + UUID.randomUUID();
    scheduleJobCallDirectMethod(jobClient, directMethodJobId);
    monitorJob(jobClient, directMethodJobId);

    // Run a query to show the job detail
    queryDeviceJobs(jobClient, start);

    System.out.println("Shutting down schedule-jobs app");
    ```

16. Guarde y cierre el archivo **schedule-jobs\src\main\java\com\mycompany\app\App.java**.

17. Compile la aplicación **schedule-jobs** y corrija los errores. En el símbolo del sistema, vaya a la carpeta **schedule-jobs** y ejecute el comando siguiente:

    ```cmd/sh
    mvn clean package -DskipTests
    ```

## <a name="create-a-device-app"></a>Creación de una aplicación de dispositivo

En esta sección, creará una aplicación de consola de Java que controla las propiedades deseadas enviadas desde IoT Hub e implementa la llamada al método directo.

1. En la carpeta **iot-java-schedule-jobs**, cree un proyecto de Maven denominado **simulated-device** mediante el comando siguiente en el símbolo del sistema. Observe que este es un comando único y largo:

   ```cmd/sh
   mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=simulated-device -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
   ```

2. En el símbolo del sistema, vaya a la nueva carpeta **simulated-device**.

3. Con un editor de texto, abra el archivo **pom.xml** de la carpeta **simulated-device** y agregue las siguientes dependencias al nodo **dependencies**. Esta dependencia permite usar el paquete **iot-device-client** en la aplicación para comunicarse con la instancia de IoT Hub:

    ```xml
    <dependency>
      <groupId>com.microsoft.azure.sdk.iot</groupId>
      <artifactId>iot-device-client</artifactId>
      <version>1.17.5</version>
    </dependency>
    ```

    > [!NOTE]
    > Puede comprobar la versión más reciente de **iot-device-client** mediante la [búsqueda de Maven](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22iot-device-client%22%20g%3A%22com.microsoft.azure.sdk.iot%22).

4. Agregue la siguiente dependencia al nodo **dependencies**. Esta dependencia configura una instrucción NOP para la fachada de registro de Apache [SLF4J](https://www.slf4j.org/), que usa el SDK de cliente de dispositivo para implementar el registro. Esta configuración es opcional, pero si la omite, es posible que vea una advertencia en la consola al ejecutar la aplicación. Para obtener más información sobre el registro en SDK de cliente de dispositivo, consulte [Registro](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-samples/readme.md#logging) en el archivo Léame *Samples for the Azure IoT device SDK for Java* (Ejemplos para el SDK de dispositivo IoT de Azure para Java).

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

9. Agregue las siguientes variables de nivel de clase a la clase **App** . Reemplace `{yourdeviceconnectionstring}` por la cadena de conexión de dispositivo que copió anteriormente en la sección [Registro de un nuevo dispositivo en el centro de IoT](#register-a-new-device-in-the-iot-hub):

    ```java
    private static String connString = "{yourdeviceconnectionstring}";
    private static IotHubClientProtocol protocol = IotHubClientProtocol.MQTT;
    private static final int METHOD_SUCCESS = 200;
    private static final int METHOD_NOT_DEFINED = 404;
    ```

    Esta aplicación de ejemplo usa la variable **protocol** al crear una instancia de un objeto **DeviceClient**.

10. Para imprimir las notificaciones de dispositivos gemelos en la consola, agregue la siguiente clase anidada a la clase **App**:

    ```java
    // Handler for device twin operation notifications from IoT Hub
    protected static class DeviceTwinStatusCallBack implements IotHubEventCallback {
      public void execute(IotHubStatusCode status, Object context) {
        System.out.println("IoT Hub responded to device twin operation with status " + status.name());
      }
    }
    ```

11. Para imprimir las notificaciones de métodos directos en la consola, agregue la siguiente clase anidada a la clase **App**:

    ```java
    // Handler for direct method notifications from IoT Hub
    protected static class DirectMethodStatusCallback implements IotHubEventCallback {
      public void execute(IotHubStatusCode status, Object context) {
        System.out.println("IoT Hub responded to direct method operation with status " + status.name());
      }
    }
    ```

12. Para controlar las llamadas a métodos directos desde IoT Hub, agregue la siguiente clase anidada a la clase **App**:

    ```java
    // Handler for direct method calls from IoT Hub
    protected static class DirectMethodCallback
        implements DeviceMethodCallback {
      @Override
      public DeviceMethodData call(String methodName, Object methodData, Object context) {
        DeviceMethodData deviceMethodData;
        switch (methodName) {
          case "lockDoor": {
            System.out.println("Executing direct method: " + methodName);
            deviceMethodData = new DeviceMethodData(METHOD_SUCCESS, "Executed direct method " + methodName);
            break;
          }
          default: {
            deviceMethodData = new DeviceMethodData(METHOD_NOT_DEFINED, "Not defined direct method " + methodName);
          }
        }
        // Notify IoT Hub of result
        return deviceMethodData;
      }
    }
    ```

13. Actualice la firma del método **main** para incluir la cláusula `throws` siguiente:

    ```java
    public static void main( String[] args ) throws IOException, URISyntaxException
    ```

14. Reemplace el código del método **main** por el código siguiente:
    * Cree un cliente de dispositivo para comunicarse con IoT Hub.
    * Cree un objeto **Device** para almacenar las propiedades del dispositivo gemelo.

    ```java
    // Create a device client
    DeviceClient client = new DeviceClient(connString, protocol);

    // An object to manage device twin desired and reported properties
    Device dataCollector = new Device() {
      @Override
      public void PropertyCall(String propertyKey, Object propertyValue, Object context)
      {
        System.out.println("Received desired property change: " + propertyKey + " " + propertyValue);
      }
    };
    ```

15. Para iniciar los servicios de cliente de dispositivo, agregue el código siguiente al método **main**:

    ```java
    try {
      // Open the DeviceClient
      // Start the device twin services
      // Subscribe to direct method calls
      client.open();
      client.startDeviceTwin(new DeviceTwinStatusCallBack(), null, dataCollector, null);
      client.subscribeToDeviceMethod(new DirectMethodCallback(), null, new DirectMethodStatusCallback(), null);
    } catch (Exception e) {
      System.out.println("Exception, shutting down \n" + " Cause: " + e.getCause() + " \n" + e.getMessage());
      dataCollector.clean();
      client.closeNow();
      System.out.println("Shutting down...");
    }
    ```

16. Para esperar a que el usuario presione la tecla **ENTRAR** antes de apagarlo, agregue el código siguiente al final del método **main**:

    ```java
    // Close the app
    System.out.println("Press any key to exit...");
    Scanner scanner = new Scanner(System.in);
    scanner.nextLine();
    dataCollector.clean();
    client.closeNow();
    scanner.close();
    ```

17. Guarde y cierre el archivo **simulated-device\src\main\java\com\mycompany\app\App.java**.

18. Compile la aplicación **simulated-device** y corrija los errores. En el símbolo del sistema, vaya a la carpeta **simulated-device** y ejecute el comando siguiente:

    ```cmd/sh
    mvn clean package -DskipTests
    ```

## <a name="run-the-apps"></a>Ejecución de las aplicaciones

Ya está preparado para ejecutar las aplicaciones de consola.

1. En un símbolo del sistema en la carpeta **simulated-device**, ejecute el siguiente comando para iniciar la escucha de cambios en las propiedades deseadas y de llamadas a métodos directos de la aplicación de dispositivo:

   ```cmd/sh
   mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
   ```

   ![El cliente de dispositivo se inicia](./media/iot-hub-java-java-schedule-jobs/device-app-1.png)

2. En un símbolo del sistema en la carpeta `schedule-jobs`, ejecute el siguiente comando para ejecutar la aplicación de servicio **schedule-jobs** a fin de ejecutar dos trabajos. El primero establece los valores de la propiedad deseada, mientras que el segundo llama al método directo:

   ```cmd\sh
   mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
   ```

   ![La aplicación de IoT Hub de Java crea dos trabajos](./media/iot-hub-java-java-schedule-jobs/service-app-1.png)

3. La aplicación de dispositivo controla el cambio de la propiedad deseada y la llamada al método directo:

   ![El cliente de dispositivo responde a los cambios](./media/iot-hub-java-java-schedule-jobs/device-app-2.png)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha utilizado un trabajo para programar un método directo para un dispositivo y la actualización de las propiedades del dispositivo gemelo.

Use los siguientes recursos para obtener información sobre cómo:

* Enviar telemetría desde dispositivos con el tutorial [Introducción a IoT Hub](quickstart-send-telemetry-java.md).

* Controlar los dispositivos de forma interactiva (por ejemplo, encender un ventilador desde una aplicación controlada por el usuario), con el tutorial [Uso de métodos directos](quickstart-control-device-java.md).
