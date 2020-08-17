---
title: Mensajes de nube a dispositivo con IoT Hub de Azure (Java) | Microsoft Docs
description: Cómo enviar mensajes de nube a un dispositivo de una instancia de IoT Hub de Azure mediante los SDK de IoT de Azure para Java. Modifique una aplicación de dispositivo simulado para recibir mensajes de nube a dispositivo y cambie una aplicación de back-end para enviar los mensajes de nube a dispositivo.
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
ms.openlocfilehash: 7f04483415253145cd485ccf870160e83a6e0e4b
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87319123"
---
# <a name="send-cloud-to-device-messages-with-iot-hub-java"></a>Envío mensajes de nube a dispositivo con IoT Hub (Java)

[!INCLUDE [iot-hub-selector-c2d](../../includes/iot-hub-selector-c2d.md)]

IoT Hub de Azure es un servicio totalmente administrado que permite la comunicación bidireccional confiable y segura entre millones de dispositivos y una solución de back-end. En el inicio rápido [Envío de telemetría desde un dispositivo a un centro de IoT](quickstart-send-telemetry-java.md) se muestra cómo crear un centro de IoT, aprovisionar en él la identidad del dispositivo y codificar una aplicación de dispositivo simulado que envíe mensajes del dispositivo a la nube.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

Este tutorial se basa en el artículo [Envío de telemetría desde un dispositivo a un centro de IoT](quickstart-send-telemetry-java.md). En él se muestra cómo realizar los pasos siguientes:

* Desde el back-end de la nube de la aplicación, envíe mensajes de nube a dispositivo en un único dispositivo a través de IoT Hub.

* Reciba mensajes de nube a dispositivo en un dispositivo.

* Desde la solución de back-end, solicite confirmación de entrega (*comentarios*) para los mensajes enviados a un dispositivo desde IoT Hub.

Encontrará más información sobre los mensajes de nube a dispositivo en la [Guía para desarrolladores de IoT Hub](iot-hub-devguide-messaging.md).

Al final de este tutorial, ejecutará dos aplicaciones de consola de Java:

* **simulated-device**, una versión modificada de la aplicación que se creó en el artículo [Envío de telemetría desde un dispositivo a un centro de IoT](quickstart-send-telemetry-java.md), que se conecta al centro de IoT y recibe mensajes de la nube al dispositivo.

* **send-c2d-messages**, que envía un mensaje de la nube a la aplicación de dispositivo simulado mediante IoT Hub y recibe la confirmación de entrega.

> [!NOTE]
> IoT Hub ofrece compatibilidad con SDK en muchas plataformas de dispositivos y lenguajes (entre los que se incluyen C, Java, Python y Javascript), mediante los SDK de dispositivo IoT de Azure. Visite el [Centro para desarrolladores de IoT de Azure](https://azure.microsoft.com/develop/iot)para obtener instrucciones paso a paso sobre cómo conectar el dispositivo al código de este tutorial y, en general, al Centro de IoT de Azure.

## <a name="prerequisites"></a>Requisitos previos

* Una versión funcional completa del inicio rápido [Envío de telemetría desde un dispositivo a un centro de IoT](quickstart-send-telemetry-java.md) o del tutorial [Configuración del enrutamiento de mensajes con IoT Hub](tutorial-routing.md).

* [Java SE Development Kit 8](https://docs.microsoft.com/java/azure/jdk/?view=azure-java-stable). Asegúrese de seleccionar **Java 8** en **Long-term support** (Soporte técnico a largo plazo) para obtener descargas de JDK 8.

* [Maven 3](https://maven.apache.org/download.cgi)

* Una cuenta de Azure activa. En caso de no tener ninguna, puede crear una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) en tan solo unos minutos.

* Asegúrese de que el puerto 8883 está abierto en el firewall. En el ejemplo de dispositivo de este artículo se usa el protocolo MQTT, que se comunica mediante el puerto 8883. Este puerto puede estar bloqueado en algunos entornos de red corporativos y educativos. Para más información y saber cómo solucionar este problema, consulte [Conexión a IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

## <a name="receive-messages-in-the-simulated-device-app"></a>Recepción de mensajes en la aplicación de dispositivo simulado

En esta sección, modificará la aplicación de dispositivo simulado que creó en [Envío de telemetría desde un dispositivo a un centro de IoT](quickstart-send-telemetry-java.md) para recibir mensajes de la nube al dispositivo desde el centro de IoT.

1. Con un editor de texto, abra el archivo simulated-device\src\main\java\com\mycompany\app\App.java.

2. Agregue la siguiente clase **MessageCallback** como una clase anidada dentro de la clase **aplicación**. El método **execute** se invoca cuando el dispositivo recibe un mensaje del Centro de IoT. En este ejemplo, el dispositivo siempre notifica al centro de IoT que ha completado el mensaje:

    ```java
    private static class AppMessageCallback implements MessageCallback {
      public IotHubMessageResult execute(Message msg, Object context) {
        System.out.println("Received message from hub: "
          + new String(msg.getBytes(), Message.DEFAULT_IOTHUB_MESSAGE_CHARSET));

        return IotHubMessageResult.COMPLETE;
      }
    }
    ```

3. Modifique el método **main** para crear una instancia de **AppMessageCallback** y llame al método **setMessageCallback** antes de que se abra el cliente, de la manera siguiente:

    ```java
    client = new DeviceClient(connString, protocol);

    MessageCallback callback = new AppMessageCallback();
    client.setMessageCallback(callback, null);
    client.open();
    ```

    > [!NOTE]
    > Si usa HTTPS en lugar de MQTT o AMQP como transporte, la instancia de **DeviceClient** busca mensajes de IoT Hub con menos frecuencia (menos de 25 minutos). Para más información sobre las diferencias entre la compatibilidad con MQTT, AMQP y HTTPS, y la limitación de IoT Hub, consulte la [sección de mensajería de la guía para desarrolladores de IoT Hub](iot-hub-devguide-messaging.md).

4. Para compilar la aplicación **simulated-device** con Maven, ejecute el siguiente comando en el símbolo del sistema en la carpeta simulated-device:

    ```cmd/sh
    mvn clean package -DskipTests
    ```

## <a name="get-the-iot-hub-connection-string"></a>Obtención de la cadena de conexión de IoT Hub

En este artículo, creará un servicio de back-end para enviar mensajes de la nube a un dispositivo a través de la instancia de IOT Hub que creó en [Enviar telemetría desde un dispositivo a IoT Hub](quickstart-send-telemetry-java.md). Para enviar mensajes de nube a un dispositivo, el servicio necesita el permiso de **conexión de servicio**. De forma predeterminada, todas las instancias de IoT Hub se crean con una directiva de acceso compartido denominada **servicio** que concede este permiso.

[!INCLUDE [iot-hub-include-find-service-connection-string](../../includes/iot-hub-include-find-service-connection-string.md)]

## <a name="send-a-cloud-to-device-message"></a>Envío de mensajes de nube a dispositivo

En esta sección, escribirá una aplicación de consola de Java que envía mensajes de nube a dispositivo a la aplicación del dispositivo simulado. Necesitará el identificador del dispositivo que agregó en el inicio rápido [Envío de telemetría desde un dispositivo a un centro de IoT](quickstart-send-telemetry-java.md). También necesitará la cadena de conexión de IoT Hub que copió anteriormente en [Obtención de la cadena de conexión de IoT Hub](#get-the-iot-hub-connection-string).

1. Cree un proyecto de Maven denominado **send-c2d-messages** mediante el siguiente comando en el símbolo del sistema. Observe que este es un comando único y largo:

    ```cmd/sh
    mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=send-c2d-messages -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    ```

2. En el símbolo del sistema, vaya a la nueva carpeta send-c2d-messages.

3. Con un editor de texto, abra el archivo pom.xml de la carpeta send-c2d-messages y agregue la siguiente dependencia al nodo **dependencies** . Esto le permite usar el paquete **iothub-java-service-client** en la aplicación para comunicarse con el servicio IoT Hub:

    ```xml
    <dependency>
      <groupId>com.microsoft.azure.sdk.iot</groupId>
      <artifactId>iot-service-client</artifactId>
      <version>1.7.23</version>
    </dependency>
    ```

    > [!NOTE]
    > Puede comprobar la versión más reciente de **iot-service-client** mediante la [búsqueda de Maven](https://search.maven.org/#search%7Cga%7C1%7Ca%3A%22iot-service-client%22%20g%3A%22com.microsoft.azure.sdk.iot%22).

4. Guarde y cierre el archivo pom.xml.

5. Con un editor de texto, abra el archivo send-c2d-messages\src\main\java\com\mycompany\app\App.java.

6. Agregue las siguientes instrucciones **import** al archivo:

    ```java
    import com.microsoft.azure.sdk.iot.service.*;
    import java.io.IOException;
    import java.net.URISyntaxException;
    ```

7. Agregue las siguientes variables de nivel de clase a la clase **App** y sustituya **{yourhubconnectionstring}** y **{yourdeviceid}** por los valores anotados anteriormente:

    ```java
    private static final String connectionString = "{yourhubconnectionstring}";
    private static final String deviceId = "{yourdeviceid}";
    private static final IotHubServiceClientProtocol protocol =    
        IotHubServiceClientProtocol.AMQPS;
    ```

8. Reemplace el método **main** por el código siguiente: Este código conecta con el centro de IoT, envía un mensaje al dispositivo y luego espera una confirmación de que el dispositivo ha recibido y procesado el mensaje:

    ```java
    public static void main(String[] args) throws IOException,
        URISyntaxException, Exception {
      ServiceClient serviceClient = ServiceClient.createFromConnectionString(
        connectionString, protocol);

      if (serviceClient != null) {
        serviceClient.open();
        FeedbackReceiver feedbackReceiver = serviceClient
          .getFeedbackReceiver();
        if (feedbackReceiver != null) feedbackReceiver.open();

        Message messageToSend = new Message("Cloud to device message.");
        messageToSend.setDeliveryAcknowledgement(DeliveryAcknowledgement.Full);

        serviceClient.send(deviceId, messageToSend);
        System.out.println("Message sent to device");

        FeedbackBatch feedbackBatch = feedbackReceiver.receive(10000);
        if (feedbackBatch != null) {
          System.out.println("Message feedback received, feedback time: "
            + feedbackBatch.getEnqueuedTimeUtc().toString());
        }

        if (feedbackReceiver != null) feedbackReceiver.close();
        serviceClient.close();
      }
    }
    ```

    > [!NOTE]
    > Por simplificar, este tutorial no implementa ninguna directiva de reintentos. En el código de producción, deberá implementar directivas de reintentos (por ejemplo, retroceso exponencial), tal y como se sugiere en el artículo [Control de errores transitorios](/azure/architecture/best-practices/transient-faults).

9. Para compilar la aplicación **simulated-device** con Maven, ejecute el siguiente comando en el símbolo del sistema en la carpeta simulated-device:

    ```cmd/sh
    mvn clean package -DskipTests
    ```

## <a name="run-the-applications"></a>Ejecución de las aplicaciones

Ahora está preparado para ejecutar las aplicaciones.

1. En un símbolo del sistema, en la carpeta simulated-device, ejecute el siguiente comando para empezar el envío de datos de telemetría a IoT Hub:

    ```cmd/sh
    mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
    ```

    ![Ejecución de una aplicación de dispositivo simulada](./media/iot-hub-java-java-c2d/receivec2d.png)

2. En un símbolo del sistema de la carpeta send-c2d-messages, ejecute el siguiente comando para enviar un mensaje de la nube a dispositivo y esperar una confirmación de comentarios:

    ```cmd/sh
    mvn exec:java -Dexec.mainClass="com.mycompany.app.App"
    ```

    ![Ejecución del comando para enviar el mensaje C2D](media/iot-hub-java-java-c2d/sendc2d.png)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, aprendió a enviar y recibir mensajes de nube a dispositivo.

Para ver ejemplos de soluciones completas que usen IoT Hub, consulte [Aceleradores de soluciones de IoT](https://azure.microsoft.com/documentation/suites/iot-suite/).

Para obtener más información sobre cómo desarrollar soluciones con IoT Hub, consulte la [Guía para desarrolladores de IoT Hub](iot-hub-devguide.md).
