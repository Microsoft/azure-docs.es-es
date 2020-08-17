---
title: 'Inicio rápido: Uso de Azure Storage v11 para .NET para administrar una cola'
description: En este inicio rápido, obtendrá información sobre cómo usar la biblioteca cliente de Azure Storage para .NET a fin de crear una cola y agregarle mensajes. A continuación, aprenderá a leer y procesar los mensajes de la cola.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 07/24/2020
ms.service: storage
ms.subservice: queues
ms.topic: quickstart
ms.reviewer: dineshm
ms.openlocfilehash: 71700efecfc173392562327f6d150d79d1a9c84f
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87292681"
---
# <a name="quickstart-use-the-azure-storage-sdk-v11-for-net-to-manage-a-queue"></a>Inicio rápido: Uso del SDK de Azure Storage v11 para .NET para administrar una cola

En este inicio rápido, obtendrá información sobre cómo usar la biblioteca cliente de Azure Storage versión 11 para .NET a fin de crear una cola y agregarle mensajes. A continuación, aprenderá a leer y procesar los mensajes de la cola. 

> [!NOTE]
> Este inicio rápido usa una versión heredada de la biblioteca cliente de Azure Queue Storage. Para empezar a trabajar con la versión más reciente, consulte [Inicio rápido: Biblioteca cliente de Azure Queue Storage v12 para .NET](storage-quickstart-queues-dotnet.md).

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [storage-quickstart-prereq-include](../../../includes/storage-quickstart-prereq-include.md)]

A continuación, descargue e instale .NET Core 2.0 para su sistema operativo. Si usa Windows, puede instalar Visual Studio y utilizar .NET Framework si lo prefiere. También puede elegir instalar un editor y usarlo con el sistema operativo.

### <a name="windows"></a>Windows

- Instale [.NET Core para Windows](https://www.microsoft.com/net/download/windows) o [.NET Framework](https://www.microsoft.com/net/download/windows) (se incluye con Visual Studio para Windows).
- Instale [Visual Studio para Windows](https://www.visualstudio.com/). Si usa .NET Core, la instalación de Visual Studio es opcional.  

Para obtener información acerca de cuándo elegir .NET Core o .NET Framework, consulte [Selección entre .NET Core y .NET Framework para aplicaciones de servidor](https://docs.microsoft.com/dotnet/standard/choosing-core-framework-server).

### <a name="linux"></a>Linux

- Instale [.NET Core para Linux](https://www.microsoft.com/net/download/linux)
- Opcionalmente, instale [Visual Studio Code](https://www.visualstudio.com/) y la [extensión de C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)

### <a name="macos"></a>macOS

- Instale [.NET Core para macOS](https://www.microsoft.com/net/download/macos).
- Opcionalmente, instale [Visual Studio para Mac](https://www.visualstudio.com/vs/visual-studio-mac/)

## <a name="download-the-sample-application"></a>Descarga de la aplicación de ejemplo

La aplicación de ejemplo utilizada en este tutorial de inicio rápido es una aplicación de consola básica. Puede explorar la aplicación de ejemplo en [GitHub](https://github.com/Azure-Samples/storage-queues-dotnet-quickstart).

Use [git](https://git-scm.com/) para descargar una copia de la aplicación en su entorno de desarrollo. 

```bash
git clone https://github.com/Azure-Samples/storage-queues-dotnet-quickstart.git
```

Este comando clona el repositorio en la carpeta git local. Para abrir la solución de Visual Studio, busque la carpeta *storage-queues-dotnet-quickstart*, ábrala y haga doble clic en *storage-queues-dotnet-quickstart.sln*. 

[!INCLUDE [storage-copy-connection-string-portal](../../../includes/storage-copy-connection-string-portal.md)]

## <a name="configure-your-storage-connection-string"></a>Configuración de la cadena de conexión de almacenamiento.

Para ejecutar la aplicación, debe proporcionar la cadena de conexión de la cuenta de almacenamiento. La aplicación de ejemplo lee la cadena de conexión de una variable de entorno y la utiliza para autorizar las solicitudes en Azure Storage.

Una vez que haya copiado la cadena de conexión, escríbala en una variable de entorno nueva en la máquina local que ejecuta la aplicación. Para establecer la variable de entorno, abra una ventana de consola y siga las instrucciones de su sistema operativo. Reemplace `<yourconnectionstring>` por la cadena de conexión real:

### <a name="windows"></a>Windows

```cmd
setx storageconnectionstring "<yourconnectionstring>"
```

Después de agregar la variable de entorno, puede que tenga que reiniciar todos los programas en ejecución que necesiten leer la variable de entorno, incluida la ventana de consola. Por ejemplo, si usa Visual Studio como editor, reinícielo antes de ejecutar el ejemplo. 

### <a name="linux"></a>Linux

```bash
export storageconnectionstring=<yourconnectionstring>
```

Después de agregar la variable de entorno, ejecute `source ~/.bashrc` desde la ventana de consola para que los cambios surtan efecto.

### <a name="macos"></a>macOS

Edite .bash_profile y agregue la variable de entorno:

```bash
export STORAGE_CONNECTION_STRING=<yourconnectionstring>
```

Después de agregar la variable de entorno, ejecute `source .bash_profile` desde la ventana de consola para que los cambios surtan efecto.

## <a name="run-the-sample"></a>Ejecución del ejemplo

La aplicación de ejemplo crea una cola y le agrega un mensaje. En primer lugar, la aplicación inspecciona el mensaje sin eliminarlo de la cola y, a continuación, lo recupera y lo elimina de la cola.

### <a name="windows"></a>Windows

Si utiliza Visual Studio como editor, puede presionar **F5** para ejecutarlo. 

Otra opción es ir al directorio de la aplicación y ejecutar la aplicación con el comando `dotnet run`.

```
dotnet run
```

### <a name="linux"></a>Linux

Desplácese hasta el directorio de la aplicación y ejecute la aplicación con el comando `dotnet run`.

```
dotnet run
```

### <a name="macos"></a>macOS

Desplácese hasta el directorio de la aplicación y ejecute la aplicación con el comando `dotnet run`.

```
dotnet run
```

La salida de la aplicación de ejemplo es similar a la siguiente:

```
Azure Queues - .NET Quickstart sample

Created queue 'quickstartqueues-3136fe9a-fa52-4b19-a447-8999a847da52'

Added message 'aa8fa95f-07ea-4df7-bf86-82b3f7debfb7' to queue 'quickstartqueues-3136fe9a-fa52-4b19-a447-8999a847da52'
Message insertion time: 2/7/2019 4:30:46 AM +00:00
Message expiration time: 2/14/2019 4:30:46 AM +00:00

Contents of peeked message 'aa8fa95f-07ea-4df7-bf86-82b3f7debfb7': Hello, World

Message 'aa8fa95f-07ea-4df7-bf86-82b3f7debfb7' becomes visible again at 2/7/2019 4:31:16 AM +00:00

Processed and deleted message 'aa8fa95f-07ea-4df7-bf86-82b3f7debfb7'

Press any key to delete the sample queue.
```

## <a name="understand-the-sample-code"></a>Descripción del código de ejemplo

Después, explore el código de ejemplo para poder saber cómo funciona.

### <a name="try-parsing-the-connection-string"></a>Pruebe a analizar la cadena de conexión

En el ejemplo, en primer lugar se comprueba que la variable de entorno contiene una cadena de conexión que se pueda analizar para crear un objeto [CloudStorageAccount](/dotnet/api/microsoft.azure.cosmos.table.cloudstorageaccount) que señala a la cuenta de almacenamiento. Para comprobar que la cadena de conexión es válida, use el método [TryParse](/dotnet/api/microsoft.azure.cosmos.table.cloudstorageaccount.tryparse). Si **TryParse** funciona correctamente, inicializa la variable *storageAccount* y devuelve el valor **true**.

```csharp
// Retrieve the connection string for use with the application. The storage connection string is stored
// in an environment variable called storageconnectionstring, on the machine where the application is running.
// If the environment variable is created after the application is launched in a console or with Visual
// Studio, the shell needs to be closed and reloaded to take the environment variable into account.
string storageConnectionString = Environment.GetEnvironmentVariable("storageconnectionstring");

// Check whether the connection string can be parsed.
if (CloudStorageAccount.TryParse(storageConnectionString, out storageAccount))
{
    // If the connection string is valid, proceed with calls to Azure Queues here.
    ...    
}
else
{
    Console.WriteLine(
        "A connection string has not been defined in the system environment variables. " +
        "Add an environment variable named 'storageconnectionstring' with your storage " +
        "connection string as a value.");
}
```

### <a name="create-the-queue"></a>Creación de la cola

En primer lugar, en el ejemplo se crea una cola y se le agrega un mensaje. 

```csharp
// Create a queue called 'quickstartqueues' and append a GUID value so that the queue name 
// is unique in your storage account. 
queue = cloudQueueClient.GetQueueReference("quickstartqueues-" + Guid.NewGuid().ToString());
await queue.CreateAsync();

Console.WriteLine("Created queue '{0}'", queue.Name);
Console.WriteLine();
```

### <a name="add-a-message"></a>Agregar un mensaje

A continuación, en el ejemplo se agrega un mensaje al final de la cola. 

El mensaje debe estar en un formato que pueda incluirse en una solicitud XML con codificación UTF-8 y puede tener hasta 64 KB de tamaño. Si un mensaje contiene datos binarios, se recomienda codificarlo en Base64.

De forma predeterminada, el tiempo de vida máximo de un mensaje se establece en 7 días. Puede especificar cualquier número positivo para el período de vida de un mensaje.

```csharp
// Create a message and add it to the queue. Set expiration time to 14 days.
CloudQueueMessage message = new CloudQueueMessage("Hello, World");
await queue.AddMessageAsync(message, new TimeSpan(14,0,0,0), null, null, null);
Console.WriteLine("Added message '{0}' to queue '{1}'", message.Id, queue.Name);
Console.WriteLine("Message insertion time: {0}", message.InsertionTime.ToString());
Console.WriteLine("Message expiration time: {0}", message.ExpirationTime.ToString());
Console.WriteLine();
```

Para agregar un mensaje que no caduque, use `Timespan.FromSeconds(-1)` en la llamada a [AddMessageAsync](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.addmessageasync).

```csharp
await queue.AddMessageAsync(message, TimeSpan.FromSeconds(-1), null, null, null);
```

### <a name="peek-a-message-from-the-queue"></a>Inspección de un mensaje de la cola

El ejemplo muestra cómo inspeccionar un mensaje de una cola. Al inspeccionar un mensaje, puede leer su contenido. Sin embargo, el mensaje permanece visible para otros clientes, de modo que otro cliente pueda recuperarlo y procesarlo posteriormente.

```csharp
// Peek at the message at the front of the queue. Peeking does not alter the message's 
// visibility, so that another client can still retrieve and process it. 
CloudQueueMessage peekedMessage = await queue.PeekMessageAsync();

// Display the ID and contents of the peeked message.
Console.WriteLine("Contents of peeked message '{0}': {1}", peekedMessage.Id, peekedMessage.AsString);
Console.WriteLine();
```

### <a name="dequeue-a-message"></a>Quitar de la cola un mensaje

En el ejemplo también se muestra cómo quitar un mensaje de la cola. Cuando un mensaje se quita de la cola, se recupera de la parte delantera y permanece temporalmente invisible para otros clientes. De forma predeterminada, un mensaje permanece invisible durante 30 segundos. Durante este tiempo, el código puede procesarlo. Para terminar de eliminar el mensaje de la cola, hágalo inmediatamente después del procesamiento, de modo que otro cliente no elimine de la cola el mismo mensaje.

Si se produce un error en el código al procesar un mensaje debido a un error de hardware o de software, el mensaje se vuelve visible una vez que haya transcurrido el período de invisibilidad. Otro cliente puede recuperar el mismo mensaje e intentarlo de nuevo.

```csharp
// Retrieve the message at the front of the queue. The message becomes invisible for 
// a specified interval, during which the client attempts to process it.
CloudQueueMessage retrievedMessage = await queue.GetMessageAsync();

// Display the time at which the message will become visible again if it is not deleted.
Console.WriteLine("Message '{0}' becomes visible again at {1}", retrievedMessage.Id, retrievedMessage.NextVisibleTime);
Console.WriteLine();

//Process and delete the message within the period of invisibility.
await queue.DeleteMessageAsync(retrievedMessage);
Console.WriteLine("Processed and deleted message '{0}'", retrievedMessage.Id);
Console.WriteLine();
```

### <a name="clean-up-resources"></a>Limpieza de recursos

En el ejemplo se limpian los recursos que se crearon al eliminar la cola. Al eliminar la cola, también se eliminan los mensajes que contenga.

```csharp
Console.WriteLine("Press any key to delete the sample queue.");
Console.ReadLine();
Console.WriteLine("Deleting the queue and any messages it contains...");
Console.WriteLine();
if (queue != null)
{
    await queue.DeleteIfExistsAsync();
}
```

## <a name="resources-for-developing-net-applications-with-queues"></a>Recursos para el desarrollo de aplicaciones .NET con colas

Consulte estos recursos adicionales para el desarrollo de .NET con Azure Queues:

### <a name="binaries-and-source-code"></a>Archivos binarios y código fuente

- Descargue los paquetes NuGet para la versión más reciente de la [Biblioteca cliente de Azure Storage para .NET](/dotnet/api/overview/azure/storage?view=azure-dotnet)
    - [Común](https://www.nuget.org/packages/Microsoft.Azure.Storage.Common/)
    - [Colas](https://www.nuget.org/packages/Azure.Storage.Queues/)
- Vea el [código fuente de la biblioteca de cliente de .NET](https://github.com/Azure/azure-storage-net) en GitHub.

### <a name="client-library-reference-and-samples"></a>Referencia a la biblioteca de clientes y ejemplos

- Para más información acerca de la biblioteca de cliente de .NET, consulte la [referencia de la API de .NET](https://docs.microsoft.com/dotnet/api/overview/azure/storage).
- Explore los [ejemplos de Queue Storage](https://azure.microsoft.com/resources/samples/?sort=0&service=storage&platform=dotnet&term=queues) escritos mediante la biblioteca cliente de .NET.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a agregar mensajes a una cola, a inspeccionar los mensajes de una cola y a quitar de la cola y a procesar los mensajes mediante. NET. 

> [!div class="nextstepaction"]
> [Comunicación entre las aplicaciones con Azure Queue storage](https://docs.microsoft.com/learn/modules/communicate-between-apps-with-azure-queue-storage/index)

- Para más información sobre .NET Core, consulte [Get started with .NET in 10 minutes](https://www.microsoft.com/net/learn/get-started/) (Introducción a .NET en 10 minutos).
