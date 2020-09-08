---
title: 'Introducción a Azure Queue Storage mediante .NET: Azure Storage'
description: Las colas de Azure proporcionan mensajería asincrónica confiable entre componentes de aplicaciones. La mensajería en la nube permite que los componentes de las aplicaciones se escalen de forma independiente.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 05/08/2020
ms.service: storage
ms.subservice: queues
ms.topic: how-to
ms.reviewer: dineshm
ms.custom: devx-track-csharp
ms.openlocfilehash: e8dadc999f3bd26671b5a8ee4da26f051a822a26
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89001117"
---
# <a name="get-started-with-azure-queue-storage-using-net"></a>Introducción al Almacenamiento en cola de Azure mediante .NET

[!INCLUDE [storage-selector-queue-include](../../../includes/storage-selector-queue-include.md)]

## <a name="overview"></a>Información general

El almacenamiento en cola de Azure proporciona mensajería en la nube entre componentes de aplicaciones. A la hora de diseñar aplicaciones para realizar su escalado, los componentes de las mismas suelen desacoplarse, por lo que se pueden escalar de forma independiente. La instancia de Queue Storage ofrece mensajería asincrónica entre los componentes de las aplicaciones, independientemente de si se ejecutan en la nube, en el escritorio, en un servidor local o en un dispositivo móvil. Además, este tipo de almacenamiento admite la administración de tareas asincrónicas y la creación de flujos de trabajo de procesos.

### <a name="about-this-tutorial"></a>Acerca de este tutorial

Este tutorial muestra cómo escribir código .NET para algunos escenarios comunes con el Almacenamiento en cola de Azure. Entre los escenarios descritos se incluyen los siguientes: creación y eliminación de colas y adición, lectura y eliminación de mensajes de la cola.

**Tiempo estimado para completarla:** 45 minutos

### <a name="prerequisites"></a>Prerrequisitos

- [Microsoft Visual Studio](https://www.visualstudio.com/downloads/)
- [Biblioteca cliente Azure Storage Common para .NET](https://www.nuget.org/packages/Microsoft.Azure.Storage.Common/)
- [Biblioteca cliente Azure Storage Queue para .NET](https://www.nuget.org/packages/Microsoft.Azure.Storage.Queue/)
- [Administrador de configuración Azure para .NET](https://www.nuget.org/packages/Microsoft.Azure.ConfigurationManager/)
- Una [cuenta de almacenamiento de Azure](../common/storage-account-create.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json)

[!INCLUDE [storage-queue-concepts-include](../../../includes/storage-queue-concepts-include.md)]

[!INCLUDE [storage-create-account-include](../../../includes/storage-create-account-include.md)]

## <a name="set-up-your-development-environment"></a>Configurado su entorno de desarrollo

A continuación, configure el entorno de desarrollo en Visual Studio para poder probar los ejemplos de código de esta guía.

### <a name="create-a-windows-console-application-project"></a>Creación de un proyecto de aplicación de consola de Windows

En Visual Studio, cree una nueva aplicación de consola de Windows. Los siguientes pasos muestran cómo crear una aplicación de consola en Visual Studio 2019. Los pasos son similares en otras versiones de Visual Studio.

1. Seleccione **Archivo** > **Nuevo** > **Proyecto**
2. Seleccione **Plataforma** > **Windows**
3. Seleccione **Aplicación de consola (.NET Framework)**
4. Seleccione **Siguiente**.
5. Escriba el nombre de la aplicación en el campo **Nombre del proyecto**.
6. Seleccione **Crear**

Todos los ejemplos de código de este tutorial se pueden agregar al método **Main()** del archivo **Program.cs** de la aplicación de consola.

Las bibliotecas cliente de Azure Storage se pueden usar en cualquier tipo de aplicación .NET, incluidos cualquier servicio en la nube o aplicación web de Azure y aplicaciones de escritorio o móviles. En esta guía, usamos una aplicación de consola para hacerlo más sencillo.

### <a name="use-nuget-to-install-the-required-packages"></a>Uso de NuGet para instalar los paquetes necesarios

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

Para completar este tutorial, es preciso que haga referencia a los siguientes cuatro paquetes en el proyecto:

- [Biblioteca de Azure Core para .NET](https://www.nuget.org/packages/Azure.Core/): este paquete proporciona primitivas, abstracciones y aplicaciones auxiliares compartidas para las bibliotecas de cliente modernas de Azure SDK de .NET.
- [Biblioteca cliente de Azure Storage Common para .NET](https://www.nuget.org/packages/Azure.Storage.Common/): este paquete proporciona una infraestructura que comparte el resto de bibliotecas de cliente de Azure Storage.
- [Biblioteca de Azure Storage Queue para .NET](https://www.nuget.org/packages/Azure.Storage.Queues/): este paquete permite trabajar con el servicio Azure Storage Queue para almacenar los mensajes a los que puede acceder un cliente.
- [Biblioteca de Configuration Manager para .NET](https://www.nuget.org/packages/System.Configuration.ConfigurationManager/): este paquete proporciona acceso a los archivos de configuración de las aplicaciones cliente.

Puede usar NuGet para obtener estos paquetes. Siga estos pasos:

1. Haga clic con el botón derecho en el proyecto, en el **Explorador de soluciones**, y elija **Administrar paquetes NuGet**.
1. Seleccione **Examinar**
1. Busque "Azure.Storage.Queues" en línea y seleccione **Instalar** para instalar la biblioteca cliente de Storage y sus dependencias. También se instalarán las bibliotecas Azure.Storage.Common y Azure.Core, que son dependencias de la biblioteca de colas.
1. Busque "System.Configuration.ConfigurationManager" en línea y seleccione **Instalar** para instalar Configuration Manager.

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

Para completar este tutorial, es preciso que haga referencia a los siguientes tres paquetes en el proyecto:

- [Biblioteca cliente Microsoft Azure Storage Common para .NET](https://www.nuget.org/packages/Microsoft.Azure.Storage.Common/): Este paquete proporciona acceso mediante programación a los recursos de datos de la cuenta de almacenamiento.
- [Biblioteca cliente Microsoft Azure Storage Queue para .NET](https://www.nuget.org/packages/Microsoft.Azure.Storage.Queue/): Esta biblioteca cliente permite trabajar con el servicio Microsoft Azure Storage Queue para almacenar los mensajes a los que puede acceder un cliente.
- [Biblioteca del Administrador de configuración de Microsoft Azure para .NET](https://www.nuget.org/packages/Microsoft.Azure.ConfigurationManager/): Este paquete proporciona una clase para analizar una cadena de conexión en un archivo de configuración, independientemente del lugar en que se ejecute la aplicación.

Puede usar NuGet para obtener estos paquetes. Siga estos pasos:

1. Haga clic con el botón derecho en el proyecto, en el **Explorador de soluciones**, y elija **Administrar paquetes NuGet**.
1. Seleccione **Examinar**
1. Busque "Microsoft.Azure.Storage.Queue" en línea y seleccione **Instalar** para instalar la biblioteca cliente de Storage y sus dependencias. Esto instalará también la biblioteca Microsoft.Azure.Storage.Common, que es una dependencia de la biblioteca de Queue.
1. Busque "Microsoft.Azure.ConfigurationManager" en línea y seleccione **Instalar** para instalar Azure Configuration Manager.

---

> [!NOTE]
> El paquete de las bibliotecas cliente de Storage también se incluye en el [SDK de Azure para .NET](https://azure.microsoft.com/downloads/). Sin embargo, se recomienda que instale también las bibliotecas cliente de Storage desde NuGet para garantizar que siempre dispone de las versiones más recientes.
>
> Las dependencias de ODataLib en las bibliotecas cliente de Storage para .NET las resuelven los paquetes de ODataLib disponibles en NuGet, no WCF Data Services. A través de NuGet, es posible descargar directamente las bibliotecas ODataLib o bien hacer referencia a ellas con el código del proyecto. Los paquetes de ODataLib específicos utilizados por las bibliotecas cliente de Storage son [OData](https://nuget.org/packages/Microsoft.Data.OData/), [Edm](https://nuget.org/packages/Microsoft.Data.Edm/) y [Spatial](https://nuget.org/packages/System.Spatial/). Aunque las clases de Azure Table Storage usan estas bibliotecas, son dependencias necesarias para programar con las bibliotecas cliente de Storage.

### <a name="determine-your-target-environment"></a>Determine su entorno de destino

Tiene dos opciones de entorno para ejecutar los ejemplos de esta guía:

- Puede ejecutar el código en una cuenta de Azure Storage en la nube.
- Puede ejecutar el código en el emulador de almacenamiento de Azurite. Azurite es un entorno local que emula una cuenta de Azure Storage en la nube. Azurite es una opción gratis para probar y depurar el código mientras la aplicación está en desarrollo. El emulador usa una cuenta y una clave conocidas. Para obtener más información, consulte [Uso del emulador Azurite en la instancia local de Azure Storage para desarrollo y pruebas](../common/storage-use-azurite.md).

> [!NOTE]
> Puede dirigirse al emulador de almacenamiento para evitar incurrir en cualquier coste asociado con Azure Storage. Sin embargo, si selecciona dirigirse a una cuenta de almacenamiento de Azure en la nube, los costes derivados de la realización de este tutorial serán insignificantes.

## <a name="get-your-storage-connection-string"></a>Obtención de la cadena de conexión de almacenamiento

Las bibliotecas cliente de Azure Storage para .NET admiten el uso de una cadena de conexión de almacenamiento para configurar puntos de conexión y credenciales a fin de acceder a los servicios de almacenamiento. Para más información, consulte [Administración de las claves de acceso de la cuenta de almacenamiento](../common/storage-account-keys-manage.md).

### <a name="copy-your-credentials-from-the-azure-portal"></a>Copia de las credenciales desde Azure Portal

El código de ejemplo debe autorizar el acceso a su cuenta de almacenamiento. Para realizar la autorización, proporcionará a la aplicación sus credenciales de cuenta de almacenamiento en forma de cadena de conexión. Para ver las credenciales de la cuenta de almacenamiento:

1. Acceda a [Azure Portal](https://portal.azure.com).
2. Busque su cuenta de almacenamiento.
3. En la sección **Configuración** de la información general de la cuenta de almacenamiento, seleccione **Claves de acceso**. Aparecen las claves de acceso de la cuenta, así como la cadena de conexión completa de cada clave.
4. Busque el valor de **Cadena de conexión** en **key1**y haga clic en el botón **Copiar** para copiar la cadena de conexión. En el paso siguiente, agregará el valor de la cadena de conexión a una variable de entorno.

    ![Captura de pantalla que muestra cómo copiar una cadena de conexión desde Azure Portal](media/storage-dotnet-how-to-use-queues/portal-connection-string.png)

Para más información acerca de las cadenas de conexión, consulte [Configuración de una cadena de conexión a Azure Storage](../common/storage-configure-connection-string.md).

> [!NOTE]
> La clave de la cuenta de almacenamiento es similar a la contraseña raíz de la cuenta de almacenamiento. Siempre debe proteger la clave de la cuenta de almacenamiento. Evite distribuirla a otros usuarios, codificarla de forma rígida o guardarla en un archivo de texto que sea accesible a otros usuarios. Vuelva a generar la clave mediante Azure Portal si cree que puede verse comprometida.

La mejor manera de conservar la cadena de conexión de almacenamiento es mediante un archivo de configuración. Para configurar la cadena de conexión, abra el archivo *app.config* en el Explorador de soluciones de Visual Studio. Agregue el contenido del elemento `\<appSettings\>` , que se muestra a continuación. Reemplace la *cadena de conexión* con el valor que copió de la cuenta de almacenamiento en el portal:

```xml
<configuration>
    <startup>
        <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.7.2" />
    </startup>
    <appSettings>
        <add key="StorageConnectionString" value="connection-string" />
    </appSettings>
</configuration>
```

Por ejemplo, el valor de configuración es similar a:

```xml
<add key="StorageConnectionString" value="DefaultEndpointsProtocol=https;AccountName=storagesample;AccountKey=GMuzNHjlB3S9itqZJHHCnRkrokLkcSyW7yK9BRbGp0ENePunLPwBgpxV1Z/pVo9zpem/2xSHXkMqTHHLcx8XRA==EndpointSuffix=core.windows.net" />
```

Para elegir como destino el emulador de almacenamiento Azurite, puede utilizar un acceso directo que se asigna al nombre y la clave conocidas de la cuenta. En ese caso, la configuración de la cadena de conexión es:

```xml
<add key="StorageConnectionString" value="UseDevelopmentStorage=true" />
```

### <a name="add-using-directives"></a>Adición de directivas using

Agregue las siguientes directivas `using` al principio del archivo `Program.cs`:

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/queues/howto/dotnet/dotnet-v12/QueueBasics.cs" id="snippet_UsingStatements":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

```csharp
using System; // Namespace for Console output
using Microsoft.Azure; // Namespace for CloudConfigurationManager
using Microsoft.Azure.Storage; // Namespace for CloudStorageAccount
using Microsoft.Azure.Storage.Queue; // Namespace for Queue storage types
```

---

### <a name="create-the-queue-service-client"></a>Creación del cliente del servicio Cola

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

La clase [QueueClient](/dotnet/api/azure.storage.queues.queueclient) le permite recuperar las colas almacenadas en Queue Storage. Esta es una forma de crear el cliente de servicio:

:::code language="csharp" source="~/azure-storage-snippets/queues/howto/dotnet/dotnet-v12/QueueBasics.cs" id="snippet_CreateClient":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

La clase [CloudQueueClient](/dotnet/api/microsoft.azure.storage.queue.cloudqueueclient?view=azure-dotnet-legacy) le permite recuperar las colas almacenadas en Almacenamiento en cola. Esta es una forma de crear el cliente de servicio:

```csharp
// Retrieve storage account from connection string
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the queue client
CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();
```

---

Ahora ya puede escribir código que lee y escribe datos en el Almacenamiento en cola.

## <a name="create-a-queue"></a>Creación de una cola

En este ejemplo se muestra cómo crear una cola:

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/queues/howto/dotnet/dotnet-v12/QueueBasics.cs" id="snippet_CreateQueue":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

```csharp
// Retrieve storage account from connection string
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the queue client
CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

// Retrieve a reference to a queue
CloudQueue queue = queueClient.GetQueueReference("myqueue");

// Create the queue if it doesn't already exist
queue.CreateIfNotExists();
```

---

## <a name="insert-a-message-into-a-queue"></a>un mensaje en una cola

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

Para insertar un mensaje en una cola existente, llame al método [SendMessage](/dotnet/api/azure.storage.queues.queueclient.sendmessage). Un mensaje puede ser un valor `string` (en formato UTF-8) o una matriz de `byte`. En el código siguiente se crea una cola (si es que no existe) y se inserta un mensaje:

:::code language="csharp" source="~/azure-storage-snippets/queues/howto/dotnet/dotnet-v12/QueueBasics.cs" id="snippet_InsertMessage":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

Para insertar un mensaje en una cola existente, cree en primer lugar un nuevo [CloudQueueMessage](/dotnet/api/microsoft.azure.storage.queue.cloudqueuemessage?view=azure-dotnet-legacy). A continuación, llame al método [AddMessage](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.addmessage?view=azure-dotnet-legacy) . Se puede crear `CloudQueueMessage` a partir de un elemento `string` (en formato UTF-8) o de una matriz `byte`. A continuación, se muestra el código con el que se crea una cola (si no existe) y se inserta el mensaje "Hola, mundo":

```csharp
// Retrieve storage account from connection string
CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

// Create the queue client
CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

// Retrieve a reference to a queue
CloudQueue queue = queueClient.GetQueueReference("myqueue");

// Create the queue if it doesn't already exist
queue.CreateIfNotExists();

// Create a message and add it to the queue
CloudQueueMessage message = new CloudQueueMessage("Hello, World");
queue.AddMessage(message);
```

---

## <a name="peek-at-the-next-message"></a>siguiente mensaje

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

Puede inspeccionar el mensaje de la cola sin tener que quitarlo de la misma, mediante una llamada al método [PeekMessages](/dotnet/api/azure.storage.queues.queueclient.peekmessages). Si no pasa un valor para el parámetro *maxMessages*, el valor predeterminado es inspeccionar un mensaje.

:::code language="csharp" source="~/azure-storage-snippets/queues/howto/dotnet/dotnet-v12/QueueBasics.cs" id="snippet_PeekMessage":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

Puede inspeccionar el mensaje situado en la parte delantera de una cola, sin quitarlo de la cola, mediante una llamada al método [PeekMessage](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.peekmessage?view=azure-dotnet-legacy) .

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

---

## <a name="change-the-contents-of-a-queued-message"></a>contenido de un mensaje en cola

Puede cambiar el contenido de un mensaje local en la cola. Si el mensaje representa una tarea de trabajo, puede usar esta característica para actualizar el estado de la tarea de trabajo. El siguiente código actualiza el mensaje de la cola con contenido nuevo y amplía el tiempo de espera de la visibilidad en 60 segundos más. De este modo, se guarda el estado de trabajo asociado al mensaje y se le proporciona al cliente un minuto más para que siga elaborando el mensaje. Esta técnica se puede utilizar para realizar un seguimiento de los flujos de trabajo de varios pasos en los mensajes en cola, sin que sea necesario volver a empezar desde el principio si se produce un error en un paso del proceso a causa de un error de hardware o software. Normalmente, también mantendría un número de reintentos y, si el mensaje se intentara más de *n* veces, lo eliminaría. Esto proporciona protección frente a un mensaje que produce un error en la aplicación cada vez que se procesa.

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/queues/howto/dotnet/dotnet-v12/QueueBasics.cs" id="snippet_UpdateMessage":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

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
message.SetMessageContent2("Updated contents.", false);
queue.UpdateMessage(message,
    TimeSpan.FromSeconds(60.0),  // Make it invisible for another 60 seconds.
    MessageUpdateFields.Content | MessageUpdateFields.Visibility);
```

---

## <a name="de-queue-the-next-message"></a>siguiente mensaje de la cola

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

Quitar un mensaje de una cola en dos pasos. Al llamar a [ReceiveMessages](/dotnet/api/azure.storage.queues.queueclient.receivemessages), obtiene el siguiente mensaje de una cola. Un mensaje devuelto por `ReceiveMessages` se hace invisible a cualquier otro código de lectura de mensajes de esta cola. De forma predeterminada, este mensaje permanece invisible durante 30 segundos. Para acabar de quitar el mensaje de la cola, también debe llamar a [DeleteMessage](/dotnet/api/azure.storage.queues.queueclient.deletemessage). Este proceso de extracción de un mensaje que consta de dos pasos garantiza que si su código no puede procesar un mensaje a causa de un error de hardware o software, otra instancia de su código puede obtener el mismo mensaje e intentarlo de nuevo. El código siguiente llama a `DeleteMessage` justo después de haberse procesado el mensaje.

:::code language="csharp" source="~/azure-storage-snippets/queues/howto/dotnet/dotnet-v12/QueueBasics.cs" id="snippet_DequeueMessage":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

El código quita un mensaje de una cola en dos pasos. Si llama a [GetMessage](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.getmessage?view=azure-dotnet-legacy), obtiene el siguiente mensaje en una cola. Un mensaje devuelto por `GetMessage` se hace invisible a cualquier otro código de lectura de mensajes de esta cola. De forma predeterminada, este mensaje permanece invisible durante 30 segundos. Para acabar de quitar el mensaje de la cola, también debe llamar a [DeleteMessage](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.deletemessage?view=azure-dotnet-legacy). Este proceso de extracción de un mensaje que consta de dos pasos garantiza que si su código no puede procesar un mensaje a causa de un error de hardware o software, otra instancia de su código puede obtener el mismo mensaje e intentarlo de nuevo. El código siguiente llama a `DeleteMessage` justo después de haberse procesado el mensaje.

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

---

## <a name="use-async-await-pattern-with-common-queue-storage-apis"></a>Uso del patrón Async-Await con API comunes de almacenamiento de colas

En este ejemplo se muestra cómo usar el patrón Async-Await con API comunes de almacenamiento de colas. El ejemplo llama a la versión asincrónica de cada uno de los métodos indicados, tal como se puede ver por el sufijo *Async* de cada método. Cuando se utiliza un método asincrónico, el patrón Async-Await suspende la ejecución local hasta que se completa la llamada. Este comportamiento permite que el subproceso actual realice otro trabajo, lo que ayuda a evitar cuellos de botella en el rendimiento y mejora la capacidad de respuesta general de la aplicación. Para más información sobre el uso del patrón Async-Await en. NET, consulte [Async y Await (C# y Visual Basic)](https://msdn.microsoft.com/library/hh191443.aspx).

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/queues/howto/dotnet/dotnet-v12/QueueBasics.cs" id="snippet_AsyncQueue":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

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

---

## <a name="leverage-additional-options-for-de-queuing-messages"></a>Uso de opciones adicionales para quitar mensajes de la cola

Hay dos formas de personalizar la recuperación de mensajes de una cola. En primer lugar, puede obtener un lote de mensajes (hasta 32). En segundo lugar, puede establecer un tiempo de espera de la invisibilidad más largo o más corto para que el código disponga de más o menos tiempo para procesar cada mensaje.

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

El siguiente ejemplo de código utiliza el método [ReceiveMessages](/dotnet/api/azure.storage.queues.queueclient.receivemessages) para obtener 20 mensajes en una llamada. A continuación, procesa cada mensaje con un bucle `foreach`. También establece el tiempo de espera de la invisibilidad en cinco minutos para cada mensaje. Tenga en cuenta que los 5 minutos empiezan a contar para todos los mensajes al mismo tiempo, por lo que después de pasar los 5 minutos desde la llamada a `ReceiveMessages`, todos los mensajes que no se han eliminado volverán a estar visibles.

:::code language="csharp" source="~/azure-storage-snippets/queues/howto/dotnet/dotnet-v12/QueueBasics.cs" id="snippet_DequeueMessages":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

El siguiente ejemplo de código utiliza el método [GetMessages](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.getmessages?view=azure-dotnet-legacy) para obtener 20 mensajes en una llamada. A continuación, procesa cada mensaje con un bucle `foreach`. También establece el tiempo de espera de la invisibilidad en cinco minutos para cada mensaje. Tenga en cuenta que los 5 minutos empiezan a contar para todos los mensajes al mismo tiempo, por lo que después de pasar los 5 minutos desde la llamada a `GetMessages`, todos los mensajes que no se han eliminado volverán a estar visibles.

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

---

## <a name="get-the-queue-length"></a>la longitud de la cola

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

Puede obtener una estimación del número de mensajes existentes en una cola. El método [GetProperties](/dotnet/api/azure.storage.queues.queueclient.getproperties) solicita a Queue service la recuperación de las propiedades de la cola, incluido el número de mensajes. El método [ApproximateMessagesCount](/dotnet/api/azure.storage.queues.models.queueproperties.approximatemessagescount) obtiene el número aproximado de mensajes en la cola. Este número no es menor que el número real de mensajes de la cola, pero podría ser mayor.

:::code language="csharp" source="~/azure-storage-snippets/queues/howto/dotnet/dotnet-v12/QueueBasics.cs" id="snippet_GetQueueLength":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

Puede obtener una estimación del número de mensajes existentes en una cola. El método [FetchAttributes](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.fetchattributes?view=azure-dotnet-legacy) solicita al servicio de cola la recuperación de los atributos de la cola, incluido el número de mensajes. La propiedad [ApproximateMessageCount](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.approximatemessagecount?view=azure-dotnet-legacy) devuelve el último valor que recuperó el método `FetchAttributes`, sin llamar a Queue service.

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
int cachedMessageCount = queue.ApproximateMessageCount;

// Display number of messages.
Console.WriteLine("Number of messages in queue: " + cachedMessageCount);
```

---

## <a name="delete-a-queue"></a>Eliminación de una cola

# <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

Para eliminar una cola y todos los mensajes contenidos en ella, llame al método [Delete](/dotnet/api/azure.storage.queues.queueclient.delete) en el objeto de cola.

:::code language="csharp" source="~/azure-storage-snippets/queues/howto/dotnet/dotnet-v12/QueueBasics.cs" id="snippet_DeleteQueue":::

# <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

Para eliminar una cola y todos los mensajes contenidos en ella, llame al método [Delete](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.delete?view=azure-dotnet-legacy) en el objeto de cola.

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

---

## <a name="next-steps"></a>Pasos siguientes

Ahora que está familiarizado con los aspectos básicos del almacenamiento de colas, utilice estos vínculos para obtener más información acerca de tareas de almacenamiento más complejas.

- Consulte la documentación de referencia del servicio de cola para obtener información detallada acerca de las API disponibles:
  - [Referencia de la biblioteca de clientes de almacenamiento para .NET](https://go.microsoft.com/fwlink/?LinkID=390731&clcid=0x409)
  - [Referencia de API de REST](https://msdn.microsoft.com/library/azure/dd179355)
- Aprenda a simplificar el código que escriba para trabajar con Azure Storage mediante [SDK de Azure WebJobs](https://github.com/Azure/azure-webjobs-sdk/wiki).
- Consulte más guías de características para obtener información acerca de otras opciones del almacenamiento de datos en Azure.
  - [Introducción al Almacenamiento de tablas de Azure mediante .NET](../../cosmos-db/table-storage-how-to-use-dotnet.md) para almacenar datos estructurados.
  - [Introducción al Almacenamiento de blobs de Azure mediante .NET](../blobs/storage-dotnet-how-to-use-blobs.md) para almacenar datos estructurados.
  - Para almacenar datos relacionales, consulte [Conexión a SQL Database mediante .NET (C#)](../../azure-sql/database/connect-query-dotnet-core.md).

[Download and install the Azure SDK for .NET]: /develop/net/
[.NET client library reference]: https://go.microsoft.com/fwlink/?LinkID=390731&clcid=0x409
[Creating an Azure Project in Visual Studio]: https://msdn.microsoft.com/library/azure/ee405487.aspx
[Azure Storage Team Blog]: https://blogs.msdn.com/b/windowsazurestorage/
[OData]: https://nuget.org/packages/Microsoft.Data.OData/5.0.2
[Edm]: https://nuget.org/packages/Microsoft.Data.Edm/5.0.2
[Spatial]: https://nuget.org/packages/System.Spatial/5.0.2
