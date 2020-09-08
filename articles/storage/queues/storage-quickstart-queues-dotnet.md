---
title: 'Inicio rápido: Biblioteca de Azure Queue Storage v12 para .NET'
description: Aprenda a usar la biblioteca de Azure Queue v12 para .NET para crear una cola y agregar mensajes a la cola. A continuación, aprenderá a leer y eliminar los mensajes de la cola. También aprenderá a eliminar una cola.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 07/24/2020
ms.service: storage
ms.subservice: queues
ms.topic: quickstart
ms.custom: devx-track-csharp
ms.openlocfilehash: fd60adf94e57be3892a264c2a8457b839e1c8778
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89011300"
---
# <a name="quickstart-azure-queue-storage-client-library-v12-for-net"></a>Inicio rápido: Biblioteca cliente de Azure Queue Storage v12 para .NET

Comience a trabajar con la biblioteca cliente de Azure Queue Storage versión 12 para .NET. Azure Queue Storage es un servicio para almacenar grandes cantidades de mensajes para su posterior recuperación y procesamiento. Siga estos pasos para instalar el paquete y probar el código de ejemplo para realizar tareas básicas.

Use la biblioteca cliente de Azure Queue Storage v12 para .NET para realizar lo siguiente:

* Creación de una cola
* Adición de mensajes a una cola
* Lectura de los mensajes de una cola
* Eliminación de un mensaje de una cola
* mensajes de una cola
* Eliminación de mensajes de una cola
* Eliminación de una cola

Recursos adicionales:

* [Documentación de referencia de API](/dotnet/api/azure.storage.queues)
* [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/storage/Azure.Storage.Queues)
* [Paquete (NuGet)](https://www.nuget.org/packages/Azure.Storage.Queues/12.0.0)
* [Muestras](https://docs.microsoft.com/azure/storage/common/storage-samples-dotnet?toc=%2fazure%2fstorage%2fqueues%2ftoc.json#queue-samples)

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/)
* Una cuenta de Azure Storage: [cree una cuenta de almacenamiento](https://docs.microsoft.com/azure/storage/common/storage-quickstart-create-account)
* El [SDK de NET Core](https://dotnet.microsoft.com/download/dotnet-core) actual del sistema operativo. Asegúrese de obtener el SDK y no el entorno de ejecución.

## <a name="setting-up"></a>Instalación

En esta sección se explica cómo preparar un proyecto para que funcione con la biblioteca cliente de Azure Queue Storage v12 para .NET.

### <a name="create-the-project"></a>Creación del proyecto

Cree una aplicación de .NET Core llamada *QueuesQuickstartV12*.

1. En una ventana de consola (por ejemplo, cmd, PowerShell o Bash), use el comando `dotnet new` para crear una nueva aplicación de consola con el nombre *QueuesQuickstartV12*. Este comando crea un sencillo proyecto "Hola mundo" de C# con un solo archivo de origen: *Program.cs*.

   ```console
   dotnet new console -n QueuesQuickstartV12
   ```

1. Cambie al directorio *QueuesQuickstartV12* recién creado.

   ```console
   cd QueuesQuickstartV12
   ```

### <a name="install-the-package"></a>Instalar el paquete

Mientras sigue en el directorio de aplicaciones, instale el paquete de la biblioteca cliente de Azure Queue Storage para .NET con el comando `dotnet add package`.

```console
dotnet add package Azure.Storage.Queues
```

### <a name="set-up-the-app-framework"></a>Instalación del marco de la aplicación

Desde el directorio del proyecto:

1. Abra el archivo *Program.cs* en el editor.
1. Quite la instrucción `Console.WriteLine("Hello World!");`.
1. Agregue directivas `using`.
1. Actualice la declaración del método `Main` para [admitir código asincrónico](https://docs.microsoft.com/dotnet/csharp/whats-new/csharp-7-1#async-main).



Este es el código:

```csharp
using Azure;
using Azure.Storage.Queues;
using Azure.Storage.Queues.Models;
using System;
using System.Threading.Tasks;

namespace QueuesQuickstartV12
{
    class Program
    {
        static async Task Main(string[] args)
        {
        }
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

Use las siguientes clases de .NET para interactuar con estos recursos:

* [QueueServiceClient](/dotnet/api/azure.storage.queues.queueserviceclient): La clase `QueueServiceClient` permite administrar todas las colas de la cuenta de almacenamiento.
* [QueueClient](/dotnet/api/azure.storage.queues.queueclient): La clase `QueueClient` permite administrar y manipular una cola individual y sus mensajes.
* [QueueMessage](/dotnet/api/azure.storage.queues.models.queuemessage): La clase `QueueMessage` representa los objetos individuales devueltos al llamar a [ReceiveMessages](/dotnet/api/azure.storage.queues.queueclient.receivemessages) en una cola.

## <a name="code-examples"></a>Ejemplos de código

Estos fragmentos de código de ejemplo muestran cómo realizar las siguientes acciones con la biblioteca cliente de Azure Queue Storage para .NET:

* [Obtención de la cadena de conexión](#get-the-connection-string)
* [Creación de una cola](#create-a-queue)
* [Adición de mensajes a una cola](#add-messages-to-a-queue)
* [Leer los mensajes de una cola](#peek-at-messages-in-a-queue)
* [Eliminación de un mensaje de una cola](#update-a-message-in-a-queue)
* [Recepción de mensajes de una cola](#receive-messages-from-a-queue)
* [Eliminación de mensajes de una cola](#delete-messages-from-a-queue)
* [Eliminación de una cola](#delete-a-queue)

### <a name="get-the-connection-string"></a>Obtención de la cadena de conexión

El código siguiente recupera la cadena de conexión de la cuenta de almacenamiento. La cadena de conexión se almacena en la variable de entorno creada en la sección [Configuración de la cadena de conexión de almacenamiento](#configure-your-storage-connection-string).

Agregue este código dentro del método `Main`:

```csharp
Console.WriteLine("Azure Queue storage v12 - .NET quickstart sample\n");

// Retrieve the connection string for use with the application. The storage
// connection string is stored in an environment variable called
// AZURE_STORAGE_CONNECTION_STRING on the machine running the application.
// If the environment variable is created after the application is launched
// in a console or with Visual Studio, the shell or application needs to be
// closed and reloaded to take the environment variable into account.
string connectionString = Environment.GetEnvironmentVariable("AZURE_STORAGE_CONNECTION_STRING");
```

### <a name="create-a-queue"></a>Creación de una cola

Decida un nombre para la nueva cola. El código siguiente anexa un valor de GUID al nombre de la cola para asegurarse de que sea único.

> [!IMPORTANT]
> Los nombres de la cola solo puede incluir letras minúsculas, números y guiones y debe empezar por una letra o un número. Antes y después de cada guion debe ir un carácter que no sea otro guión. El nombre debe tener entre 3 y 63 caracteres. Para más información sobre la nomenclatura de las colas, consulte [Asignación de nombres a colas y metadatos](/rest/api/storageservices/naming-queues-and-metadata).


Cree una instancia de la clase [QueueClient](/dotnet/api/azure.storage.queues.queueclient). A continuación, llame al método [CreateAsync](/dotnet/api/azure.storage.queues.queueclient.createasync) para crear la cola en la cuenta de almacenamiento.

Agregue este código al final del método `Main`:

```csharp
// Create a unique name for the queue
string queueName = "quickstartqueues-" + Guid.NewGuid().ToString();

Console.WriteLine($"Creating queue: {queueName}");

// Instantiate a QueueClient which will be
// used to create and manipulate the queue
QueueClient queueClient = new QueueClient(connectionString, queueName);

// Create the queue
await queueClient.CreateAsync();
```

### <a name="add-messages-to-a-queue"></a>Adición de mensajes a una cola

El siguiente fragmento de código agrega de manera asincrónica mensajes a la cola mediante una llamada al método [SendMessageAsync](/dotnet/api/azure.storage.queues.queueclient.sendmessageasync). También guarda una clase [SendReceipt](/dotnet/api/azure.storage.queues.models.sendreceipt) devuelta desde una llamada a `SendMessageAsync`. La confirmación se utiliza para actualizar el mensaje más adelante en el programa.

Agregue este código al final del método `Main`:

```csharp
Console.WriteLine("\nAdding messages to the queue...");

// Send several messages to the queue
await queueClient.SendMessageAsync("First message");
await queueClient.SendMessageAsync("Second message");

// Save the receipt so we can update this message later
SendReceipt receipt = await queueClient.SendMessageAsync("Third message");
```

### <a name="peek-at-messages-in-a-queue"></a>Lectura de los mensajes de una cola

Lea los mensajes de la cola llamando al método [PeekMessagesAsync](/dotnet/api/azure.storage.queues.queueclient.peekmessagesasync). El método `PeekMessagesAsync` recupera uno o varios mensajes de la parte delantera de la cola, pero no modifica la visibilidad del mensaje.

Agregue este código al final del método `Main`:

```csharp
Console.WriteLine("\nPeek at the messages in the queue...");

// Peek at messages in the queue
PeekedMessage[] peekedMessages = await queueClient.PeekMessagesAsync(maxMessages: 10);

foreach (PeekedMessage peekedMessage in peekedMessages)
{
    // Display the message
    Console.WriteLine($"Message: {peekedMessage.MessageText}");
}
```

### <a name="update-a-message-in-a-queue"></a>Eliminación de un mensaje de una cola

Actualice el contenido de un mensaje mediante la llamada al método [UpdateMessageAsync](/dotnet/api/azure.storage.queues.queueclient.updatemessageasync). El método `UpdateMessageAsync` puede cambiar el tiempo de espera de visibilidad y el contenido de un mensaje. El contenido del mensaje debe ser una cadena con codificación UTF-8 de hasta 64 KB de tamaño. Junto con el nuevo contenido del mensaje, pase los valores de `SendReceipt` que se guardó anteriormente en el código. Los valores `SendReceipt` identifican el mensaje que se va a actualizar.

```csharp
Console.WriteLine("\nUpdating the third message in the queue...");

// Update a message using the saved receipt from sending the message
await queueClient.UpdateMessageAsync(receipt.MessageId, receipt.PopReceipt, "Third message has been updated");
```

### <a name="receive-messages-from-a-queue"></a>mensajes de una cola

Descargue los mensajes agregados anteriormente mediante la llamada al método [ReceiveMessagesAsync](/dotnet/api/azure.storage.queues.queueclient.receivemessagesasync).

Agregue este código al final del método `Main`:

```csharp
Console.WriteLine("\nReceiving messages from the queue...");

// Get messages from the queue
QueueMessage[] messages = await queueClient.ReceiveMessagesAsync(maxMessages: 10);
```

### <a name="delete-messages-from-a-queue"></a>Eliminación de mensajes de una cola

Elimine los mensajes de la cola una vez procesados. En este caso, el procesamiento solo muestra el mensaje en la consola.

La aplicación se detiene para la entrada del usuario mediante una llamada a `Console.ReadLine` antes de procesar y eliminar los mensajes. Compruebe en [Azure Portal](https://portal.azure.com) que los recursos se crearon correctamente, antes de que se eliminen. Los mensajes que no se eliminen explícitamente volverán a estar visibles en la cola para que se procesen.

Agregue este código al final del método `Main`:

```csharp
Console.WriteLine("\nPress Enter key to 'process' messages and delete them from the queue...");
Console.ReadLine();

// Process and delete messages from the queue
foreach (QueueMessage message in messages)
{
    // "Process" the message
    Console.WriteLine($"Message: {message.MessageText}");

    // Let the service know we're finished with
    // the message and it can be safely deleted.
    await queueClient.DeleteMessageAsync(message.MessageId, message.PopReceipt);
}
```

### <a name="delete-a-queue"></a>Eliminación de una cola

El código siguiente limpia los recursos que creó la aplicación; para ello, elimina la cola mediante el método [DeleteAsync](/dotnet/api/azure.storage.queues.queueclient.deleteasync).

Agregue este código al final del método `Main`:

```csharp
Console.WriteLine("\nPress Enter key to delete the queue...");
Console.ReadLine();

// Clean up
Console.WriteLine($"Deleting queue: {queueClient.Name}");
await queueClient.DeleteAsync();

Console.WriteLine("Done");
```

## <a name="run-the-code"></a>Ejecución del código

Esta aplicación crea y agrega tres mensajes a una cola de Azure. El código muestra los mensajes en la cola y, a continuación, los recupera y los elimina antes de eliminar la cola.

En la ventana de la consola, vaya al directorio de la aplicación y, después, compile y ejecute la aplicación.

```console
dotnet build
```

```console
dotnet run
```

La salida de la aplicación es similar a la del ejemplo siguiente:

```output
Azure Queue storage v12 - .NET quickstart sample

Creating queue: quickstartqueues-5c72da2c-30cc-4f09-b05c-a95d9da52af2

Adding messages to the queue...

Peek at the messages in the queue...
Message: First message
Message: Second message
Message: Third message

Updating the third message in the queue...

Receiving messages from the queue...

Press Enter key to 'process' messages and delete them from the queue...

Message: First message
Message: Second message
Message: Third message has been updated

Press Enter key to delete the queue...

Deleting queue: quickstartqueues-5c72da2c-30cc-4f09-b05c-a95d9da52af2
Done
```

Cuando la aplicación se detiene antes de recibir mensajes, compruebe la cuenta de almacenamiento en [Azure Portal](https://portal.azure.com). Compruebe que los mensajes están en la cola.

Presione la tecla **Entrar** para recibir y eliminar los mensajes. Cuando se le solicite, presione de nuevo la tecla **Entrar** para eliminar la cola y finalizar la demostración.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a crear una cola y a agregarle mensajes mediante código .NET asincrónico. Después aprendió a ver, recuperar y eliminar mensajes. Por último, ha aprendido a eliminar una cola de mensajes.

Para ver tutoriales, ejemplos, artículos de inicio rápido y otra documentación, visite:

> [!div class="nextstepaction"]
> [Azure para desarrolladores de .NET y .NET Core](https://docs.microsoft.com/dotnet/azure/)

* Para más información, consulte [Bibliotecas de Azure Storage para .NET](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/storage).
* Para ver más aplicaciones de ejemplo de Azure Queue Storage, continúe con los [ejemplos de la biblioteca cliente de Azure Queue Storage para .NET v12](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/storage/Azure.Storage.Queues/samples).
* Para más información sobre .NET Core, consulte [Get started with .NET in 10 minutes](https://www.microsoft.com/net/learn/get-started/) (Introducción a .NET en 10 minutos).
