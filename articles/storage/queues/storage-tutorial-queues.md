---
title: 'Tutorial: Uso de colas de Azure Storage en .NET'
description: En este tutorial se describe cómo usar Azure Queue service para crear colas e insertar, obtener y eliminar mensajes mediante código de .NET.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 06/09/2020
ms.service: storage
ms.subservice: queues
ms.topic: tutorial
ms.reviewer: dineshm
ms.custom: devx-track-csharp
ms.openlocfilehash: 7474cfbd0182797bd62e97979e83e2aeb5244cbc
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89008801"
---
# <a name="tutorial-work-with-azure-storage-queues-in-net"></a>Tutorial: Uso de las colas de Azure Storage en .NET

Azure Queue Storage implementa colas basadas en la nube para permitir la comunicación entre los componentes de una aplicación distribuida. Cada cola incluye una lista de los mensajes que un componente emisor puede agregar y un componente receptor puede procesar. Con una cola, la aplicación se puede escalar inmediatamente para satisfacer la demanda. En este artículo se muestran los pasos básicos para trabajar con una cola de Azure Storage.

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> - Creación de una cuenta de Azure Storage
> - Creación de la aplicación
> - Agregar las bibliotecas cliente de Azure
> - Agregar compatibilidad con el código asincrónico
> - Creación de una cola
> - Insertar mensajes en una cola
> - Retirar mensajes de la cola
> - Eliminar una cola vacía
> - Buscar argumentos de línea de comandos
> - Compilación y ejecución de la aplicación

## <a name="prerequisites"></a>Requisitos previos

- Obtenga una copia gratuita del editor de [Visual Studio Code](https://code.visualstudio.com/download) multiplataforma.
- Descargue e instale la versión 3.1 o posterior del [SDK de .NET Core](https://dotnet.microsoft.com/download).
- Si no tiene una suscripción actual a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="create-an-azure-storage-account"></a>Creación de una cuenta de Azure Storage

Cree primero una cuenta de Azure Storage. Para obtener una guía detallada sobre la creación de una cuenta de almacenamiento, consulte el inicio rápido [Creación de una cuenta de almacenamiento](../common/storage-account-create.md?toc=%2Fazure%2Fstorage%2Fqueues%2Ftoc.json). Este es un paso independiente que debe realizar después de crear una cuenta de Azure gratuita en los requisitos previos.

## <a name="create-the-app"></a>Creación de la aplicación

Cree una aplicación de .NET Core llamada **QueueApp**. Por motivos de sencillez, esta aplicación enviará y recibirá mensajes a través de la cola.

1. En una ventana de consola (por ejemplo, CMD, PowerShell o CLI de Azure), use el comando `dotnet new` para crear una nueva aplicación de consola con el nombre **QueueApp**. Este comando crea un sencillo proyecto "Hola mundo" de C# con un solo archivo de origen: **Program.cs**.

   ```console
   dotnet new console -n QueueApp
   ```

2. Cambie a la carpeta **QueueApp** recién creada y compile la aplicación para comprobar que todo está bien.

   ```console
   cd QueueApp
   ```

   ```console
   dotnet build
   ```

   Debería ver resultados similares a lo siguiente:

   ```output
   C:\Tutorials>dotnet new console -n QueueApp
   The template "Console Application" was created successfully.

   Processing post-creation actions...
   Running 'dotnet restore' on QueueApp\QueueApp.csproj...
     Restore completed in 155.63 ms for C:\Tutorials\QueueApp\QueueApp.csproj.

   Restore succeeded.

   C:\Tutorials>cd QueueApp

   C:\Tutorials\QueueApp>dotnet build
   Microsoft (R) Build Engine version 16.0.450+ga8dc7f1d34 for .NET Core
   Copyright (C) Microsoft Corporation. All rights reserved.

     Restore completed in 40.87 ms for C:\Tutorials\QueueApp\QueueApp.csproj.
     QueueApp -> C:\Tutorials\QueueApp\bin\Debug\netcoreapp3.1\QueueApp.dll

   Build succeeded.
       0 Warning(s)
       0 Error(s)

   Time Elapsed 00:00:02.40

   C:\Tutorials\QueueApp>_
   ```

## <a name="add-the-azure-client-libraries"></a>Adición de las bibliotecas cliente de Azure

1. Agregue las bibliotecas cliente de Azure Storage al proyecto mediante el comando `dotnet add package`.

   # <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

   Ejecute el siguiente comando desde la carpeta de proyecto en la ventana de la consola.

   ```console
   dotnet add package Azure.Storage.Queues
   ```

   # <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

   Ejecute los siguientes comandos desde la carpeta de proyecto en la ventana de la consola.

   ```console
   dotnet add package Microsoft.Azure.Storage.Common
   ```

   ```console
   dotnet add package Microsoft.Azure.Storage.Queue
   ```
   ---

### <a name="add-using-statements"></a>Adición de instrucciones using

1. En la línea de comandos del directorio del proyecto, escriba `code .` para abrir Visual Studio Code en el directorio actual. Mantenga la ventana de la línea de comandos abierta. Habrá más comandos que ejecutar más tarde. Si se le pide que agregue los recursos de C# necesarios para compilar y depurar, haga clic en el botón **Sí**.

1. Abra el archivo de origen **Program.cs** y agregue los siguientes espacios de nombres justo después de la instrucción `using System;`. Esta aplicación utiliza tipos de estos espacios de nombres para conectarse a Azure Storage y trabajar con colas.

   # <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v12/QueueApp/Program.cs" id="snippet_UsingStatements":::

   # <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v11/QueueApp/Program.cs" id="snippet_UsingStatements":::

1. Guarde el archivo **Program.cs**.

## <a name="add-support-for-asynchronous-code"></a>Agregar compatibilidad con el código asincrónico

Como la aplicación usa recursos en la nube, el código se ejecuta de forma asincrónica.

1. Actualice el método **Main** para que se ejecute de forma asincrónica. Reemplace **void** por un valor devuelto **async Task**.

   ```csharp
   static async Task Main(string[] args)
   ```

1. Guarde el archivo **Program.cs**.

## <a name="create-a-queue"></a>Creación de una cola

Antes de realizar llamadas a las API de Azure, debe obtener las credenciales de Azure Portal.

[!INCLUDE [storage-quickstart-credentials-include](../../../includes/storage-quickstart-credentials-include.md)]

### <a name="add-the-connection-string-to-the-app"></a>Incorporación de la cadena de conexión a la aplicación

Agregue la cadena de conexión a la aplicación para que pueda acceder a la cuenta de almacenamiento.

1. Vuelva a Visual Studio Code.

1. En el método **Main**, reemplace el código `Console.WriteLine("Hello World!");` por la línea siguiente que obtiene la cadena de conexión de la variable de entorno.

   # <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v12/QueueApp/Program.cs" id="snippet_DeclareConnectionString":::

   # <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v11/QueueApp/Program.cs" id="snippet_DeclareConnectionString":::

1. Actualice el código siguiente a **Main** para crear un objeto de cola, que se pasa posteriormente a los métodos de envío y recepción.

   # <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v12/QueueApp/Program.cs" id="snippet_CreateQueueClient":::

   # <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v11/QueueApp/Program.cs" id="snippet_CreateQueueClient":::

1. Guarde el archivo.

## <a name="insert-messages-into-the-queue"></a>Inserción de mensajes en la cola

Cree un nuevo método para enviar un mensaje a la cola.

1. Agregue el siguiente método **InsertMessageAsync** a la clase **Program**.

   # <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

   A este método se le pasa una referencia de cola. Se crea una cola, si aún no existe, mediante la llamada a [CreateIfNotExistsAsync](/dotnet/api/azure.storage.queues.queueclient.createifnotexistsasync). Luego, se agrega *newMessage* a la cola mediante la llamada a [SendMessageAsync](/dotnet/api/azure.storage.queues.queueclient.sendmessageasync).

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v12/QueueApp/Program.cs" id="snippet_InsertMessage":::

   # <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

   A este método se le pasa una referencia de cola. Se crea una cola, si aún no existe, mediante la llamada a [CreateIfNotExistsAsync](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.createifnotexistsasync). Luego, se agrega *newMessage* a la cola mediante la llamada a [AddMessageAsync](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.addmessageasync).

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v11/QueueApp/Program.cs" id="snippet_InsertMessage":::

1. **Opcional** De forma predeterminada, el tiempo de vida máximo de un mensaje se establece en siete días. Puede especificar cualquier número positivo para el período de vida de un mensaje. El siguiente fragmento de código agrega un mensaje que *nunca* expira.

   # <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

    Para agregar un mensaje que no expire, use `Timespan.FromSeconds(-1)` en la llamada a **SendMessageAsync**.

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v12/QueueApp/Initial.cs" id="snippet_SendNonExpiringMessage":::

   # <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

    Para agregar un mensaje que no expire, use `Timespan.FromSeconds(-1)` en la llamada a **AddMessageAsync**.

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v11/QueueApp/Initial.cs" id="snippet_SendNonExpiringMessage":::

1. Guarde el archivo.

Un mensaje de cola debe tener un formato compatible con una solicitud XML con codificación UTF-8. Un mensaje puede tener un tamaño de hasta 64 KB. Si un mensaje contiene datos binarios, se recomienda [codificarlo en Base64](/dotnet/api/system.convert.tobase64string).

## <a name="dequeue-messages"></a>Retirar mensajes de la cola

Cree un método para recuperar un mensaje de la cola. Una vez que el mensaje se recibe correctamente, es importante eliminarlo de la cola para que no se procese más de una vez.

1. Agregue un nuevo método llamado **RetrieveNextMessageAsync** a la clase **Program**.

   # <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

   Este método recibe un mensaje de la cola mediante la llamada a [ReceiveMessagesAsync](/dotnet/api/azure.storage.queues.queueclient.receivemessagesasync), y pasa 1 en el primer parámetro para recuperar solo el siguiente mensaje de la cola. Una vez recibido el mensaje, elimínelo de la cola mediante una llamada a [DeleteMessageAsync](/dotnet/api/azure.storage.queues.queueclient.deletemessageasync).

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v12/QueueApp/Initial.cs" id="snippet_InitialRetrieveMessage":::

   # <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

   Este método recibe un mensaje de la cola mediante una llamada a [GetMessageAsync](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.getmessageasync). Una vez recibido el mensaje, elimínelo de la cola mediante una llamada a [DeleteMessageAsync](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.deletemessageasync).

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v11/QueueApp/Initial.cs" id="snippet_InitialRetrieveMessage":::

1. Guarde el archivo.

## <a name="delete-an-empty-queue"></a>Eliminar una cola vacía

Al final de un proyecto es un procedimiento recomendado identificar si aún necesita los recursos que creó. Los recursos que se dejan en ejecución pueden costarle mucho dinero. Si la cola existe pero está vacía, pregunte al usuario si desea eliminarla.

1. Expanda el método **ReceiveMessageAsync** para incluir un aviso para eliminar la cola vacía.

   # <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v12/QueueApp/Program.cs" id="snippet_RetrieveMessage":::

   # <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v11/QueueApp/Program.cs" id="snippet_RetrieveMessage":::

1. Guarde el archivo.

## <a name="check-for-command-line-arguments"></a>Buscar argumentos de línea de comandos

Si no se pasa ningún argumento de línea de comandos a la aplicación, se supone que hay un mensaje para agregar a la cola. Una los argumentos para crear una cadena. Agregue esta cadena a la cola de mensajes mediante una llamada al método **SendMessageAsync** que se agregó anteriormente.

Si no hay ningún argumento de línea de comandos, intente una operación de recuperación. Llame al método **RetrieveNextMessageAsync** para recuperar el siguiente mensaje de la cola.

Por último, espere la entrada del usuario antes de salir mediante una llamada a **Console.ReadLine**.

1. Expanda el método **Main** para que busque argumentos de línea de comandos y espere la entrada del usuario.

   # <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v12/QueueApp/Program.cs" id="snippet_Main":::

   # <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v11/QueueApp/Program.cs" id="snippet_Main":::

1. Guarde el archivo.

## <a name="complete-code"></a>Código completo

Aquí está la lista de códigos completa de este proyecto.

   # <a name="net-v12"></a>[\.NET v12](#tab/dotnet)

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v12/QueueApp/Program.cs" id="snippet_AllCode":::

   # <a name="net-v11"></a>[\.NET v11](#tab/dotnetv11)

   :::code language="csharp" source="~/azure-storage-snippets/queues/tutorial/dotnet/dotnet-v11/QueueApp/Program.cs" id="snippet_AllCode":::
   ---

## <a name="build-and-run-the-app"></a>Compilación y ejecución de la aplicación

1. En la línea de comandos del directorio del proyecto, ejecute el siguiente comando dotnet para compilar el proyecto.

   ```console
   dotnet build
   ```

1. Después de que el proyecto se compile correctamente, ejecute el siguiente comando para agregar el primer mensaje a la cola.

   ```console
   dotnet run First queue message
   ```

   Debería ver este resultado:

   ```output
   C:\Tutorials\QueueApp>dotnet run First queue message
   The queue was created.
   Sent: First queue message
   Press Enter..._
   ```

1. Ejecute la aplicación sin argumentos de línea de comandos para recibir y quitar el primer mensaje de la cola.

   ```console
   dotnet run
   ```

1. Continúe ejecutando la aplicación hasta que se eliminen todos los mensajes. Si la ejecuta más de una vez, recibirá un mensaje de que la cola está vacía y un aviso para que elimine la cola.

   ```output
   C:\Tutorials\QueueApp>dotnet run First queue message
   The queue was created.
   Sent: First queue message
   Press Enter...

   C:\Tutorials\QueueApp>dotnet run Second queue message
   Sent: Second queue message
   Press Enter...

   C:\Tutorials\QueueApp>dotnet run Third queue message
   Sent: Third queue message
   Press Enter...

   C:\Tutorials\QueueApp>dotnet run
   Received: First queue message
   Press Enter...

   C:\Tutorials\QueueApp>dotnet run
   Received: Second queue message
   Press Enter...

   C:\Tutorials\QueueApp>dotnet run
   Received: Third queue message
   Press Enter...

   C:\Tutorials\QueueApp>dotnet run
   The queue is empty. Attempt to delete it? (Y/N) Y
   Received: The queue was deleted.
   Press Enter...

   C:\Tutorials\QueueApp>_
   ```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

1. Creación de una cola
1. Adición y eliminación de mensajes de una cola
1. Eliminación de una cola de Azure Storage

Consulte los inicios rápidos sobre las colas de Azure para más información.

> [!div class="nextstepaction"]
> [Inicio rápido de colas para el portal](storage-quickstart-queues-portal.md)

- [Inicio rápido de colas para .NET](storage-quickstart-queues-dotnet.md)
- [Inicio rápido de colas para Java](storage-quickstart-queues-java.md)
- [Inicio rápido de colas para Python](storage-quickstart-queues-python.md)
- [Inicio rápido de colas para JavaScript](storage-quickstart-queues-nodejs.md)
