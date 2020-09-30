---
title: 'Inicio rápido: Biblioteca de Azure Queue Storage v12 para Java'
description: Aprenda a usar la biblioteca de Azure Queue v12 para Java para crear una cola y agregar mensajes a la cola. A continuación, aprenderá a leer y eliminar los mensajes de la cola. También aprenderá a eliminar una cola.
author: mhopkins-msft
ms.custom: devx-track-java
ms.author: mhopkins
ms.date: 09/10/2020
ms.service: storage
ms.subservice: queues
ms.topic: quickstart
ms.openlocfilehash: 9b9f68737ca714c0661ffb008347149235886c68
ms.sourcegitcommit: 43558caf1f3917f0c535ae0bf7ce7fe4723391f9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2020
ms.locfileid: "90015031"
---
# <a name="quickstart-azure-queue-storage-client-library-v12-for-java"></a>Inicio rápido: Biblioteca cliente de Azure Queue Storage v12 para Java

Comience a trabajar con la biblioteca cliente de Azure Queue Storage versión 12 para Java. Azure Queue Storage es un servicio para almacenar grandes cantidades de mensajes para su posterior recuperación y procesamiento. Siga estos pasos para instalar el paquete y probar el código de ejemplo para realizar tareas básicas.

Use la biblioteca cliente de Azure Queue Storage v12 para Java para realizar lo siguiente:

* Creación de una cola
* Adición de mensajes a una cola
* Lectura de los mensajes de una cola
* Eliminación de un mensaje de una cola
* Recepción y eliminación de mensajes de una cola
* Eliminación de una cola

Recursos adicionales:

* [Documentación de referencia de API](/java/api/overview/azure/storage-queue-readme)
* [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/storage/azure-storage-queue)
* [Paquete (Maven)](https://mvnrepository.com/artifact/com.azure/azure-storage-queue)
* [Muestras](/azure/storage/common/storage-samples-java?toc=%2fazure%2fstorage%2fqueues%2ftoc.json#queue-samples)

## <a name="prerequisites"></a>Requisitos previos

* [Kit de desarrollo de Java (JDK)](/java/azure/jdk/), versión 8 o posterior
* [Apache Maven](https://maven.apache.org/download.cgi)
* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/)
* Una cuenta de Azure Storage: [cree una cuenta de almacenamiento](/azure/storage/common/storage-quickstart-create-account)

## <a name="setting-up"></a>Instalación

En esta sección se explica cómo preparar un proyecto para que funcione con la biblioteca cliente de Azure Queue Storage v12 para Java.

### <a name="create-the-project"></a>Creación del proyecto

Cree una aplicación Java llamada *queues-quickstart-v12*.

1. En una ventana de consola (por ejemplo, cmd, PowerShell o Bash), use Maven para crear una nueva aplicación de consola con el nombre *queues-quickstart-v12*. Escriba el siguiente comando **mvn** para crear un proyecto "Hola mundo" sencillo de Java.

   ```console
   mvn archetype:generate -DgroupId=com.queues.quickstart \
                          -DartifactId=queues-quickstart-v12 \
                          -DarchetypeArtifactId=maven-archetype-quickstart \
                          -DarchetypeVersion=1.4 \
                          -DinteractiveMode=false
   ```

1. La salida a partir de la generación del proyecto debe ser similar a la siguiente:

    ```console
    [INFO] Scanning for projects...
    [INFO]
    [INFO] ------------------< org.apache.maven:standalone-pom >-------------------
    [INFO] Building Maven Stub Project (No POM) 1
    [INFO] --------------------------------[ pom ]---------------------------------
    [INFO]
    [INFO] >>> maven-archetype-plugin:3.1.2:generate (default-cli) > generate-sources @ standalone-pom >>>
    [INFO]
    [INFO] <<< maven-archetype-plugin:3.1.2:generate (default-cli) < generate-sources @ standalone-pom <<<
    [INFO]
    [INFO]
    [INFO] --- maven-archetype-plugin:3.1.2:generate (default-cli) @ standalone-pom ---
    [INFO] Generating project in Batch mode
    [INFO] ----------------------------------------------------------------------------
    [INFO] Using following parameters for creating project from Archetype: maven-archetype-quickstart:1.4
    [INFO] ----------------------------------------------------------------------------
    [INFO] Parameter: groupId, Value: com.queues.quickstart
    [INFO] Parameter: artifactId, Value: queues-quickstart-v12
    [INFO] Parameter: version, Value: 1.0-SNAPSHOT
    [INFO] Parameter: package, Value: com.queues.quickstart
    [INFO] Parameter: packageInPathFormat, Value: com/queues/quickstart
    [INFO] Parameter: version, Value: 1.0-SNAPSHOT
    [INFO] Parameter: package, Value: com.queues.quickstart
    [INFO] Parameter: groupId, Value: com.queues.quickstart
    [INFO] Parameter: artifactId, Value: queues-quickstart-v12
    [INFO] Project created from Archetype in dir: C:\quickstarts\queues\queues-quickstart-v12
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time:  6.394 s
    [INFO] Finished at: 2019-12-03T09:58:35-08:00
    [INFO] ------------------------------------------------------------------------
    ```

1. Cambie al directorio *queues-quickstart-v12* recién creado.

   ```console
   cd queues-quickstart-v12
   ```

### <a name="install-the-package"></a>Instalar el paquete

Abra el archivo *pom.xml* en el editor de texto. Agregue el siguiente elemento de dependencia al grupo de dependencias.

```xml
<dependency>
  <groupId>com.azure</groupId>
  <artifactId>azure-storage-queue</artifactId>
  <version>12.0.1</version>
</dependency>
```

### <a name="set-up-the-app-framework"></a>Instalación del marco de la aplicación

Desde el directorio del proyecto:

1. Vaya al directorio */src/main/java/com/blobs/quickstart*.
1. Abra el archivo *App.java* en el editor.
1. Eliminar la instrucción `System.out.println("Hello world!");`.
1. Agregue directivas `import`.

Este es el código:

```java
package com.queues.quickstart;

/**
 * Azure queue storage v12 SDK quickstart
 */
import com.azure.storage.queue.*;
import com.azure.storage.queue.models.*;
import java.io.*;
import java.time.*;

public class App
{
    public static void main( String[] args ) throws IOException
    {
    }
}
```

[!INCLUDE [storage-quickstart-credentials-include](../../../includes/storage-quickstart-credentials-include.md)]

## <a name="object-model"></a>Modelo de objetos

Azure Queue Storage es un servicio para almacenar grandes cantidades de mensajes, Un mensaje de la cola puede llegar a tener hasta 64 KB. Una cola puede contener millones de mensajes, hasta el límite de capacidad total de una cuenta de almacenamiento. Las colas se utilizan normalmente para crear un trabajo pendiente del trabajo que se va a procesar de forma asincrónica. Queue Storage ofrece tres tipos de recursos:

* La cuenta de almacenamiento
* Cola en la cuenta de almacenamiento
* Mensajes dentro de la cola

En el siguiente diagrama se muestra la relación entre estos recursos.

![Diagrama de la arquitectura de Queue Storage](./media/storage-queues-introduction/queue1.png)

Use las siguientes clases de Java para interactuar con estos recursos:

* [QueueClientBuilder](/java/api/com.azure.storage.queue.queueclientbuilder): La clase `QueueClientBuilder` configura y crea instancias de un objeto `QueueClient`.
* [QueueServiceClient](/java/api/com.azure.storage.queue.queueserviceclient): La clase `QueueServiceClient` permite administrar todas las colas de la cuenta de almacenamiento.
* [QueueClient](/java/api/com.azure.storage.queue.queueclient): La clase `QueueClient` permite administrar y manipular una cola individual y sus mensajes.
* [QueueMessageItem](/java/api/com.azure.storage.queue.models.queuemessageitem): La clase `QueueMessageItem` representa los objetos individuales devueltos al llamar a [receiveMessages](/java/api/com.azure.storage.queue.queueclient.receivemessages) en una cola.

## <a name="code-examples"></a>Ejemplos de código

Estos fragmentos de código de ejemplo muestran cómo realizar las siguientes acciones con la biblioteca cliente de Azure Queue Storage para Java:

* [Obtención de la cadena de conexión](#get-the-connection-string)
* [Creación de una cola](#create-a-queue)
* [Adición de mensajes a una cola](#add-messages-to-a-queue)
* [Leer los mensajes de una cola](#peek-at-messages-in-a-queue)
* [Eliminación de un mensaje de una cola](#update-a-message-in-a-queue)
* [Recepción y eliminación de mensajes de una cola](#receive-and-delete-messages-from-a-queue)
* [Eliminación de una cola](#delete-a-queue)

### <a name="get-the-connection-string"></a>Obtención de la cadena de conexión

El código siguiente recupera la cadena de conexión de la cuenta de almacenamiento. La cadena de conexión se almacena en la variable de entorno creada en la sección [Configuración de la cadena de conexión de almacenamiento](#configure-your-storage-connection-string).

Agregue este código dentro del método `main`:

```java
System.out.println("Azure Queues storage v12 - Java quickstart sample\n");

// Retrieve the connection string for use with the application. The storage
// connection string is stored in an environment variable on the machine
// running the application called AZURE_STORAGE_CONNECTION_STRING. If the environment variable
// is created after the application is launched in a console or with
// Visual Studio, the shell or application needs to be closed and reloaded
// to take the environment variable into account.
String connectStr = System.getenv("AZURE_STORAGE_CONNECTION_STRING");
```

### <a name="create-a-queue"></a>Creación de una cola

Decida un nombre para la nueva cola. El código siguiente anexa un valor de GUID al nombre de la cola para asegurarse de que sea único.

> [!IMPORTANT]
> Los nombres de la cola solo puede incluir letras minúsculas, números y guiones y debe empezar por una letra o un número. Antes y después de cada guion debe ir un carácter que no sea otro guión. El nombre debe tener entre 3 y 63 caracteres. Para más información sobre la nomenclatura de las colas, consulte [Asignación de nombres a colas y metadatos](/rest/api/storageservices/naming-queues-and-metadata).


Cree una instancia de la clase [QueueClient](/java/api/com.azure.storage.queue.queueclient). A continuación, llame al método [create](/java/api/com.azure.storage.queue.queueclient.create) para crear la cola en la cuenta de almacenamiento.

Agregue este código al final del método `main`:

```java
// Create a unique name for the queue
String queueName = "quickstartqueues-" + java.util.UUID.randomUUID();

System.out.println("Creating queue: " + queueName);

// Instantiate a QueueClient which will be
// used to create and manipulate the queue
QueueClient queueClient = new QueueClientBuilder()
                                .connectionString(connectStr)
                                .queueName(queueName)
                                .buildClient();

// Create the queue
queueClient.create();
```

### <a name="add-messages-to-a-queue"></a>Adición de mensajes a una cola

El siguiente fragmento de código agrega mensajes a la cola mediante una llamada al método [sendMessage](/java/api/com.azure.storage.queue.queueclient.sendmessage). También guarda la clase [SendMessageResult](/java/api/com.azure.storage.queue.models.sendmessageresult) devuelta desde una llamada a `sendMessage`. El resultado se utiliza para actualizar el mensaje más adelante en el programa.

Agregue este código al final del método `main`:

```java
System.out.println("\nAdding messages to the queue...");

// Send several messages to the queue
queueClient.sendMessage("First message");
queueClient.sendMessage("Second message");

// Save the result so we can update this message later
SendMessageResult result = queueClient.sendMessage("Third message");
```

### <a name="peek-at-messages-in-a-queue"></a>Visualización de los mensajes de una cola

Lea los mensajes de la cola llamando al método [peekMessages](/java/api/com.azure.storage.queue.queueclient.peekmessages). El método `peelkMessages` recupera uno o varios mensajes de la parte delantera de la cola, pero no modifica la visibilidad del mensaje.

Agregue este código al final del método `main`:

```java
System.out.println("\nPeek at the messages in the queue...");

// Peek at messages in the queue
queueClient.peekMessages(10, null, null).forEach(
    peekedMessage -> System.out.println("Message: " + peekedMessage.getMessageText()));
```

### <a name="update-a-message-in-a-queue"></a>Eliminación de un mensaje de una cola

Actualice el contenido de un mensaje mediante la llamada al método [updateMessage](/java/api/com.azure.storage.queue.queueclient.updatemessage). El método `updateMessage` puede cambiar el tiempo de espera de visibilidad y el contenido de un mensaje. El contenido del mensaje debe ser una cadena con codificación UTF-8 de hasta 64 KB de tamaño. Junto con el nuevo contenido del mensaje, pase el identificador de mensaje y la recepción de confirmación mediante `SendMessageResult` que se guardó anteriormente en el código. El identificador de mensaje y la confirmación de extracción identifican el mensaje que se va a actualizar.

```java
System.out.println("\nUpdating the third message in the queue...");

// Update a message using the result that
// was saved when sending the message
queueClient.updateMessage(result.getMessageId(),
                          result.getPopReceipt(), 
                          "Third message has been updated",
                          Duration.ofSeconds(1));
```

### <a name="receive-and-delete-messages-from-a-queue"></a>Recepción y eliminación de mensajes de una cola

Descargue los mensajes agregados anteriormente mediante la llamada al método [receiveMessages](/java/api/com.azure.storage.queue.queueclient.receivemessages). El código de ejemplo también elimina los mensajes de la cola una vez recibidos y procesados. En este caso, el procesamiento solo muestra el mensaje en la consola.

La aplicación se detiene para la entrada del usuario mediante una llamada a `System.console().readLine();` antes de recibir y eliminar los mensajes. Compruebe en [Azure Portal](https://portal.azure.com) que los recursos se crearon correctamente, antes de que se eliminen. Los mensajes que no se eliminen explícitamente volverán a estar visibles en la cola para que se procesen.

Agregue este código al final del método `main`:

```java
System.out.println("\nPress Enter key to receive messages and delete them from the queue...");
System.console().readLine();

// Get messages from the queue
queueClient.receiveMessages(10).forEach(
    // "Process" the message
    receivedMessage -> {
        System.out.println("Message: " + receivedMessage.getMessageText());

        // Let the service know we're finished with
        // the message and it can be safely deleted.
        queueClient.deleteMessage(receivedMessage.getMessageId(), receivedMessage.getPopReceipt());
    }
);
```

### <a name="delete-a-queue"></a>Eliminación de una cola

El código siguiente limpia los recursos que creó la aplicación; para ello, elimina la cola mediante el método [delete](/java/api/com.azure.storage.queue.queueclient.delete).

Agregue este código al final del método `main`:

```java
System.out.println("\nPress Enter key to delete the queue...");
System.console().readLine();

// Clean up
System.out.println("Deleting queue: " + queueClient.getQueueName());
queueClient.delete();

System.out.println("Done");
```

## <a name="run-the-code"></a>Ejecución del código

Esta aplicación crea y agrega tres mensajes a una cola de Azure. El código muestra los mensajes en la cola y, a continuación, los recupera y los elimina antes de eliminar la cola.

En la ventana de la consola, vaya al directorio de la aplicación y, después, compile y ejecute la aplicación.

```console
mvn compile
```

A continuación, compile el paquete.

```console
mvn package
```

Ejecute el siguiente comando `mvn` para ejecutar la aplicación.

```console
mvn exec:java -Dexec.mainClass="com.queues.quickstart.App" -Dexec.cleanupDaemonThreads=false
```

La salida de la aplicación es similar a la del ejemplo siguiente:

```output
Azure Queues storage v12 - Java quickstart sample

Adding messages to the queue...

Peek at the messages in the queue...
Message: First message
Message: Second message
Message: Third message

Updating the third message in the queue...

Press Enter key to receive messages and delete them from the queue...

Message: First message
Message: Second message
Message: Third message has been updated

Press Enter key to delete the queue...

Deleting queue: quickstartqueues-fbf58f33-4d5a-41ac-ac0e-1a05d01c7003
Done
```

Cuando la aplicación se detiene antes de recibir mensajes, compruebe la cuenta de almacenamiento en [Azure Portal](https://portal.azure.com). Compruebe que los mensajes están en la cola.

Presione la tecla **Entrar** para recibir y eliminar los mensajes. Cuando se le solicite, presione de nuevo la tecla **Entrar** para eliminar la cola y finalizar la demostración.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a crear una cola y a agregarle mensajes mediante código Java. Después aprendió a ver, recuperar y eliminar mensajes. Por último, ha aprendido a eliminar una cola de mensajes.

Para ver tutoriales, ejemplos, artículos de inicio rápido y otra documentación, visite:

> [!div class="nextstepaction"]
> [Azure para desarrolladores de Java Cloud](/azure/developer/java/)

* Para ver las aplicaciones de ejemplo de Azure Queue Storage, continúe con los [ejemplos de la biblioteca cliente del SDK de Azure Queue Storage para Java v12](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/storage/azure-storage-queue/src/samples/java/com/azure/storage/queue).
