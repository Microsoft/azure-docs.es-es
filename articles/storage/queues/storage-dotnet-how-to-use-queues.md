---
title: 'Introducción a Azure Queue storage mediante. NET: Azure Storage'
description: Las colas de Azure proporcionan mensajería asincrónica confiable entre componentes de aplicaciones. La mensajería en la nube permite que los componentes de las aplicaciones se escalen de forma independiente.
services: storage
author: mhopkins-msft
ms.service: storage
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 06/13/2018
ms.author: mhopkins
ms.reviewer: cbrooks
ms.subservice: queues
ms.openlocfilehash: 81bf178a97944d4110cf99a442163229a283bd25
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65797751"
---
# <a name="get-started-with-azure-queue-storage-using-net"></a>Introducción al Almacenamiento en cola de Azure mediante .NET
[!INCLUDE [storage-selector-queue-include](../../../includes/storage-selector-queue-include.md)]

[!INCLUDE [storage-check-out-samples-dotnet](../../../includes/storage-check-out-samples-dotnet.md)]

## <a name="overview"></a>Información general
El almacenamiento en cola de Azure proporciona mensajería en la nube entre componentes de aplicaciones. A la hora de diseñar aplicaciones para escala, los componentes de las mismas suelen desacoplarse para poder escalarlos de forma independiente. El almacenamiento en cola ofrece mensajería asincrónica para la comunicación entre los componentes de las aplicaciones, independientemente de si se ejecutan en la nube, en el escritorio, en un servidor local o en un dispositivo móvil. Además, este tipo de almacenamiento admite la administración de tareas asincrónicas y la creación de flujos de trabajo de procesos.

### <a name="about-this-tutorial"></a>Acerca de este tutorial
Este tutorial muestra cómo escribir código .NET para algunos escenarios comunes con el Almacenamiento en cola de Azure. Entre los escenarios descritos se incluyen los siguientes: creación y eliminación de colas y adición, lectura y eliminación de mensajes de la cola.

**Tiempo estimado para completarla:** 45 minutos

**Requisitos previos:**

* [Microsoft Visual Studio](https://www.visualstudio.com/downloads/)
* [Biblioteca de cliente de Azure Storage para .NET](https://www.nuget.org/packages/WindowsAzure.Storage/)
* [Administrador de configuración Azure para .NET](https://www.nuget.org/packages/Microsoft.Azure.ConfigurationManager/)
* Una [cuenta de almacenamiento de Azure](../common/storage-quickstart-create-account.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json)

[!INCLUDE [storage-queue-concepts-include](../../../includes/storage-queue-concepts-include.md)]

[!INCLUDE [storage-create-account-include](../../../includes/storage-create-account-include.md)]

[!INCLUDE [storage-development-environment-include](../../../includes/storage-development-environment-include.md)]

### <a name="add-using-directives"></a>Adición de directivas using
Agregue las siguientes directivas `using` al principio del archivo `Program.cs`:

```csharp
using Microsoft.Azure; // Namespace for CloudConfigurationManager
using Microsoft.Azure.Storage; // Namespace for CloudStorageAccount
using Microsoft.Azure.Storage.Queue; // Namespace for Queue storage types
```

### <a name="copy-your-credentials-from-the-azure-portal"></a>Copia de las credenciales desde Azure Portal

El código de ejemplo debe autorizar el acceso a su cuenta de almacenamiento. Para realizar la autorización, proporcionará a la aplicación sus credenciales de cuenta de almacenamiento en forma de cadena de conexión. Para ver las credenciales de la cuenta de almacenamiento:

1. Acceda a [Azure Portal](https://portal.azure.com).
2. Busque su cuenta de almacenamiento.
3. En la sección **Configuración** de la información general de la cuenta de almacenamiento, seleccione **Claves de acceso**. Aparecen las claves de acceso de la cuenta, así como la cadena de conexión completa de cada clave.   
4. Busque el valor de **Cadena de conexión** en **key1**y haga clic en el botón **Copiar** para copiar la cadena de conexión. En el paso siguiente, agregará el valor de la cadena de conexión a una variable de entorno.

    ![Captura de pantalla que muestra cómo copiar una cadena de conexión desde Azure Portal](media/storage-dotnet-how-to-use-queues/portal-connection-string.png)

### <a name="parse-the-connection-string"></a>Análisis de la cadena de conexión
[!INCLUDE [storage-cloud-configuration-manager-include](../../../includes/storage-cloud-configuration-manager-include.md)]

### <a name="create-the-queue-service-client"></a>Creación del cliente del servicio Cola
La clase **CloudQueueClient** le permite recuperar las colas almacenadas en Almacenamiento en cola. Esta es una forma de crear el cliente de servicio:

```csharp
CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();
```
    
Ahora ya puede escribir código que lee y escribe datos en el Almacenamiento en cola.

## <a name="create-a-queue"></a>Crear una cola
En este ejemplo se muestra cómo crear una cola si todavía no existe:

```csharp
// Retrieve storage account from connection string.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the queue client.
CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

// Retrieve a reference to a container.
CloudQueue queue = queueClient.GetQueueReference("myqueue");

// Create the queue if it doesn't already exist
queue.CreateIfNotExists();
```

## <a name="insert-a-message-into-a-queue"></a>un mensaje en una cola
Para insertar un mensaje en una cola existente, cree en primer lugar un nuevo **CloudQueueMessage**. A continuación, llame al método **AddMessage** . Se puede crear un valor de **CloudQueueMessage** a partir de una cadena (en formato UTF-8) o de una matriz de **bytes**. A continuación se muestra el código con el que se crea una cola (si no existe) y se inserta el mensaje "Hola, mundo":

```csharp
// Retrieve storage account from connection string.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the queue client.
CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

// Retrieve a reference to a queue.
CloudQueue queue = queueClient.GetQueueReference("myqueue");

// Create the queue if it doesn't already exist.
queue.CreateIfNotExists();

// Create a message and add it to the queue.
CloudQueueMessage message = new CloudQueueMessage("Hello, World");
queue.AddMessage(message);
```

## <a name="peek-at-the-next-message"></a>siguiente mensaje
Puede inspeccionar el mensaje situado en la parte delantera de una cola, sin quitarlo de la cola, mediante una llamada al método **PeekMessage** .

```csharp
// Retrieve storage account from connection string
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the queue client
CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

// Retrieve a reference to a queue
CloudQueue queue = queueClient.GetQueueReference("myqueue");

// Peek at the next message
CloudQueueMessage peekedMessage = queue.PeekMessage();

// Display message.
Console.WriteLine(peekedMessage.AsString);
```

## <a name="change-the-contents-of-a-queued-message"></a>contenido de un mensaje en cola
Puede cambiar el contenido de un mensaje local en la cola. Si el mensaje representa una tarea de trabajo, puede usar esta característica para actualizar el estado de la tarea de trabajo. El siguiente código actualiza el mensaje de la cola con contenido nuevo y amplía el tiempo de espera de la visibilidad en 60 segundos más. De este modo, se guarda el estado de trabajo asociado al mensaje y se le proporciona al cliente un minuto más para que siga elaborando el mensaje. Esta técnica se puede utilizar para realizar un seguimiento de los flujos de trabajo de varios pasos en los mensajes en cola, sin que sea necesario volver a empezar desde el principio si se produce un error en un paso del proceso a causa de un error de hardware o software. Normalmente, también mantendría un número de reintentos y, si el mensaje se intentara más de *n* veces, lo eliminaría. Esto proporciona protección frente a un mensaje que produce un error en la aplicación cada vez que se procesa.

```csharp
// Retrieve storage account from connection string.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the queue client.
CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

// Retrieve a reference to a queue.
CloudQueue queue = queueClient.GetQueueReference("myqueue");

// Get the message from the queue and update the message contents.
CloudQueueMessage message = queue.GetMessage();
message.SetMessageContent("Updated contents.");
queue.UpdateMessage(message,
    TimeSpan.FromSeconds(60.0),  // Make it invisible for another 60 seconds.
    MessageUpdateFields.Content | MessageUpdateFields.Visibility);
```

## <a name="de-queue-the-next-message"></a>siguiente mensaje de la cola
El código quita un mensaje de una cola en dos pasos. Si llama a **GetMessage**, obtiene el siguiente mensaje en una cola. Un mensaje devuelto por **GetMessage** se hace invisible a cualquier otro código de lectura de mensajes de esta cola. De forma predeterminada, este mensaje permanece invisible durante 30 segundos. Para acabar de quitar el mensaje de la cola, también debe llamar a **DeleteMessage**. Este proceso de extracción de un mensaje que consta de dos pasos garantiza que si su código no puede procesar un mensaje a causa de un error de hardware o software, otra instancia de su código puede obtener el mismo mensaje e intentarlo de nuevo. El código llama a **DeleteMessage** justo después de que se haya procesado el mensaje.

```csharp
// Retrieve storage account from connection string
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the queue client
CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

// Retrieve a reference to a queue
CloudQueue queue = queueClient.GetQueueReference("myqueue");

// Get the next message
CloudQueueMessage retrievedMessage = queue.GetMessage();

//Process the message in less than 30 seconds, and then delete the message
queue.DeleteMessage(retrievedMessage);
```

## <a name="use-async-await-pattern-with-common-queue-storage-apis"></a>Uso del patrón Async-Await con API comunes de almacenamiento de colas
En este ejemplo se muestra cómo usar el patrón Async-Await con API comunes de almacenamiento de colas. El ejemplo llama a la versión asincrónica de cada uno de los métodos indicados, tal como se puede ver por el sufijo *Async* de cada método. Cuando se utiliza un método asincrónico, el patrón Async-Await suspende la ejecución local hasta que se completa la llamada. Este comportamiento permite que el subproceso actual realice otro trabajo, lo que ayuda a evitar cuellos de botella en el rendimiento y mejora la capacidad de respuesta general de la aplicación. Para más información sobre el uso del patrón Async-Await en. NET, consulte [Async y Await (C# y Visual Basic)](https://msdn.microsoft.com/library/hh191443.aspx).

```csharp
// Create the queue if it doesn't already exist
if(await queue.CreateIfNotExistsAsync())
{
    Console.WriteLine("Queue '{0}' Created", queue.Name);
}
else
{
    Console.WriteLine("Queue '{0}' Exists", queue.Name);
}

// Create a message to put in the queue
CloudQueueMessage cloudQueueMessage = new CloudQueueMessage("My message");

// Async enqueue the message
await queue.AddMessageAsync(cloudQueueMessage);
Console.WriteLine("Message added");

// Async dequeue the message
CloudQueueMessage retrievedMessage = await queue.GetMessageAsync();
Console.WriteLine("Retrieved message with content '{0}'", retrievedMessage.AsString);

// Async delete the message
await queue.DeleteMessageAsync(retrievedMessage);
Console.WriteLine("Deleted message");
```
    
## <a name="leverage-additional-options-for-de-queuing-messages"></a>Uso de opciones adicionales para quitar mensajes de la cola
Hay dos formas de personalizar la recuperación de mensajes de una cola.
En primer lugar, puede obtener un lote de mensajes (hasta 32). En segundo lugar, puede establecer un tiempo de espera de la invisibilidad más largo o más corto para que el código disponga de más o menos tiempo para procesar cada mensaje. El siguiente ejemplo de código utiliza el método **GetMessages** para obtener 20 mensajes en una llamada. A continuación, procesa cada mensaje con un bucle **foreach** . También establece el tiempo de espera de la invisibilidad en cinco minutos para cada mensaje. Tenga en cuenta que los 5 minutos empiezan a contar para todos los mensajes al mismo tiempo, por lo que después de pasar los 5 minutos desde la llamada a **GetMessages**, todos los mensajes que no se han eliminado volverán a estar visibles.

```csharp
// Retrieve storage account from connection string.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the queue client.
CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

// Retrieve a reference to a queue.
CloudQueue queue = queueClient.GetQueueReference("myqueue");

foreach (CloudQueueMessage message in queue.GetMessages(20, TimeSpan.FromMinutes(5)))
{
    // Process all messages in less than 5 minutes, deleting each message after processing.
    queue.DeleteMessage(message);
}
```

## <a name="get-the-queue-length"></a>la longitud de la cola
Puede obtener una estimación del número de mensajes existentes en una cola. El método **FetchAttributes** solicita al servicio de cola la recuperación de los atributos de la cola, incluido el número de mensajes. La propiedad **ApproximateMessageCount** devuelve el último valor recuperado por el método **FetchAttributes**, sin llamar a Queue service.

```csharp
// Retrieve storage account from connection string.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the queue client.
CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

// Retrieve a reference to a queue.
CloudQueue queue = queueClient.GetQueueReference("myqueue");

// Fetch the queue attributes.
queue.FetchAttributes();

// Retrieve the cached approximate message count.
int? cachedMessageCount = queue.ApproximateMessageCount;

// Display number of messages.
Console.WriteLine("Number of messages in queue: " + cachedMessageCount);
```

## <a name="delete-a-queue"></a>Eliminación de una cola
Para eliminar una cola y todos los mensajes contenidos en ella, llame al método **Delete** en el objeto de cola.

```csharp
// Retrieve storage account from connection string.
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the queue client.
CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

// Retrieve a reference to a queue.
CloudQueue queue = queueClient.GetQueueReference("myqueue");

// Delete the queue.
queue.Delete();
```
    

## <a name="next-steps"></a>Pasos siguientes
Ahora que está familiarizado con los aspectos básicos del almacenamiento de colas, utilice estos vínculos para obtener más información acerca de tareas de almacenamiento más complejas.

* Consulte la documentación de referencia del servicio de cola para obtener información detallada acerca de las API disponibles:
  * [Referencia de la biblioteca de clientes de almacenamiento para .NET](https://go.microsoft.com/fwlink/?LinkID=390731&clcid=0x409)
  * [Referencia de API de REST](https://msdn.microsoft.com/library/azure/dd179355)
* Aprenda a simplificar el código que escriba para trabajar con Azure Storage mediante [SDK de Azure WebJobs](https://github.com/Azure/azure-webjobs-sdk/wiki).
* Consulte más guías de características para obtener información acerca de otras opciones del almacenamiento de datos en Azure.
  * [Introducción al Almacenamiento de tablas de Azure mediante .NET](../../cosmos-db/table-storage-how-to-use-dotnet.md) para almacenar datos estructurados.
  * [Introducción al Almacenamiento de blobs de Azure mediante .NET](../blobs/storage-dotnet-how-to-use-blobs.md) para almacenar datos estructurados.
  * Para almacenar datos relacionales, consulte [Conexión a SQL Database mediante .NET (C#)](../../sql-database/sql-database-connect-query-dotnet-core.md).

[Download and install the Azure SDK for .NET]: /develop/net/
[.NET client library reference]: https://go.microsoft.com/fwlink/?LinkID=390731&clcid=0x409
[Creating an Azure Project in Visual Studio]: https://msdn.microsoft.com/library/azure/ee405487.aspx
[Azure Storage Team Blog]: https://blogs.msdn.com/b/windowsazurestorage/
[OData]: https://nuget.org/packages/Microsoft.Data.OData/5.0.2
[Edm]: https://nuget.org/packages/Microsoft.Data.Edm/5.0.2
[Spatial]: https://nuget.org/packages/System.Spatial/5.0.2
