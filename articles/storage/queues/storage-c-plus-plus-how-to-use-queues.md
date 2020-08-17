---
title: Uso del almacenamiento de colas (C++) - Azure Storage
description: Aprenda a usar el servicio de almacenamiento de colas en Azure. Los ejemplos están escritos en C++.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 07/16/2020
ms.service: storage
ms.subservice: queues
ms.topic: how-to
ms.reviewer: dineshm
ms.openlocfilehash: 6a4f8b99be564779b350bff2ab5b37f3c7ccc6f2
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87020988"
---
# <a name="how-to-use-queue-storage-from-c"></a>Uso de Queue Storage de C++
[!INCLUDE [storage-selector-queue-include](../../../includes/storage-selector-queue-include.md)]

[!INCLUDE [storage-try-azure-tools-queues](../../../includes/storage-try-azure-tools-queues.md)]

## <a name="overview"></a>Información general
Esta guía muestra cómo realizar algunas tareas comunes a través del servicio de almacenamiento en cola de Azure. Los ejemplos están escritos en C++ y usan la [biblioteca de cliente de Azure Storage para C++](https://github.com/Azure/azure-storage-cpp/blob/master/README.md). Entre los escenarios descritos se incluyen **insertar**, **ojear**, **obtener** y **eliminar** mensajes de la cola, así como **crear y eliminar colas**.

> [!NOTE]
> Esta guía se destina a la biblioteca de cliente de Azure Storage para C++, versión 1.0.0 y posteriores. La versión recomendada es la biblioteca de cliente de almacenamiento 2.2.0, que se encuentra disponible a través de [NuGet](https://www.nuget.org/packages/wastorage) o [GitHub](https://github.com/Azure/azure-storage-cpp/).
> 
> 

[!INCLUDE [storage-queue-concepts-include](../../../includes/storage-queue-concepts-include.md)]

[!INCLUDE [storage-create-account-include](../../../includes/storage-create-account-include.md)]

## <a name="create-a-c-application"></a>Creación de una aplicación de C++
En esta guía, usará las características de almacenamiento que se pueden ejecutar en una aplicación C++.

Para ello, deberá instalar la biblioteca de cliente de Azure Storage para C++ y crear una cuenta de Azure Storage en su suscripción de Azure.

Para instalar la biblioteca de cliente de Azure Storage para C++, puede usar los métodos siguientes:

* **Linux:** Follow the instructions given in the [Azure Storage Client Library for C++ README: Getting Started on Linux](https://github.com/Azure/azure-storage-cpp#getting-started-on-linux) page.
* **Windows:** En Windows, use [vcpkg](https://github.com/microsoft/vcpkg) como administrador de dependencias. Siga el [inicio rápido](https://github.com/microsoft/vcpkg#quick-start) para inicializar vcpkg. Después, use el comando siguiente para instalar la biblioteca:

```powershell
.\vcpkg.exe install azure-storage-cpp
```

Puede encontrar una guía sobre cómo compilar el código fuente y exportarlo a NuGet en el archivo [Léame](https://github.com/Azure/azure-storage-cpp#download--install).

## <a name="configure-your-application-to-access-queue-storage"></a>Configuración de la aplicación para obtener acceso a Queue Storage
Agregue las siguientes instrucciones include en la parte superior del archivo C++ en el que desea usar las API de almacenamiento de Azure para obtener acceso a las colas:

```cpp
#include <was/storage_account.h>
#include <was/queue.h>
```

## <a name="set-up-an-azure-storage-connection-string"></a>Configuración de una cadena de conexión de Almacenamiento de Azure
Un cliente de almacenamiento de Azure utiliza una cadena de conexión de almacenamiento para almacenar extremos y credenciales con el fin de obtener acceso a los servicios de administración de datos. Al ejecutarse en una aplicación cliente, debe proporcionar la cadena de conexión de almacenamiento en el siguiente formato, usando el nombre de su cuenta de almacenamiento y la clave de acceso de almacenamiento de la cuenta de almacenamiento que se muestra en [Azure Portal](https://portal.azure.com) para los valores *AccountName* y *AccountKey*. Para obtener información sobre las cuentas de almacenamiento y las claves de acceso, consulte [Acerca de las cuentas de Azure Storage](../common/storage-create-storage-account.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json). En este ejemplo se muestra cómo puede declarar un campo estático para mantener la cadena de conexión:

```cpp
// Define the connection-string with your values.
const utility::string_t storage_connection_string(U("DefaultEndpointsProtocol=https;AccountName=your_storage_account;AccountKey=your_storage_account_key"));
```

Para probar la aplicación en el equipo Windows local, puede usar el [emulador de almacenamiento Azurite](../common/storage-use-azurite.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json). Azurite es una utilidad que simula los servicios Blob y Queue disponibles en Azure en la máquina de desarrollo local. En el ejemplo siguiente se muestra cómo puede declarar un campo estático para mantener la cadena de conexión en el emulador de almacenamiento local:

```cpp
// Define the connection-string with Azurite.
const utility::string_t storage_connection_string(U("UseDevelopmentStorage=true;"));  
```

Para iniciar Azurite, consulte [Uso del emulador de Azurite para desarrollo local de Azure Storage](../common/storage-use-azurite.md).

En los ejemplos siguientes se supone que usó uno de estos dos métodos para obtener la cadena de conexión de almacenamiento.

## <a name="retrieve-your-connection-string"></a>Recuperación de la cadena de conexión
Puede usar la clase **cloud_storage_account** para representar la información de la cuenta de almacenamiento. Para recuperar la información de la cuenta de almacenamiento a partir de la cadena de conexión de almacenamiento, puede usar el método **parse** .

```cpp
// Retrieve storage account from connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);
```

## <a name="how-to-create-a-queue"></a>Procedimientos: Creación de una cola
Los objetos **cloud_queue_client** le permiten obtener objetos de referencia para las colas. El siguiente código crea un objeto **cloud_queue_client**.

```cpp
// Retrieve storage account from connection string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create a queue client.
azure::storage::cloud_queue_client queue_client = storage_account.create_cloud_queue_client();
```

Use el objeto **cloud_queue_client** para obtener una referencia a la cola que desea usar. En caso de que la cola no exista todavía, es posible crearla.

```cpp
// Retrieve a reference to a queue.
azure::storage::cloud_queue queue = queue_client.get_queue_reference(U("my-sample-queue"));

// Create the queue if it doesn't already exist.
 queue.create_if_not_exists();  
```

## <a name="how-to-insert-a-message-into-a-queue"></a>Procedimientos: un mensaje en una cola
Para insertar un mensaje en una cola existente, cree en primer lugar un nuevo **cloud_queue_message**. A continuación, llame al método **add_message**. Se puede crear un **cloud_queue_message** a partir de una cadena o de una matriz de **byte**. A continuación se muestra el código con el que se crea una cola (si no existe) y se inserta el mensaje "Hola, mundo":

```cpp
// Retrieve storage account from connection-string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the queue client.
azure::storage::cloud_queue_client queue_client = storage_account.create_cloud_queue_client();

// Retrieve a reference to a queue.
azure::storage::cloud_queue queue = queue_client.get_queue_reference(U("my-sample-queue"));

// Create the queue if it doesn't already exist.
queue.create_if_not_exists();

// Create a message and add it to the queue.
azure::storage::cloud_queue_message message1(U("Hello, World"));
queue.add_message(message1);  
```

## <a name="how-to-peek-at-the-next-message"></a>Procedimientos: siguiente mensaje
Puede ojear el mensaje situado en la parte delantera de una cola, sin quitarlo de la cola, llamando al método **peek_message**.

```cpp
// Retrieve storage account from connection-string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the queue client.
azure::storage::cloud_queue_client queue_client = storage_account.create_cloud_queue_client();

// Retrieve a reference to a queue.
azure::storage::cloud_queue queue = queue_client.get_queue_reference(U("my-sample-queue"));

// Peek at the next message.
azure::storage::cloud_queue_message peeked_message = queue.peek_message();

// Output the message content.
std::wcout << U("Peeked message content: ") << peeked_message.content_as_string() << std::endl;
```

## <a name="how-to-change-the-contents-of-a-queued-message"></a>Procedimientos: contenido de un mensaje en cola
Puede cambiar el contenido de un mensaje local en la cola. Si el mensaje representa una tarea de trabajo, puede usar esta característica para actualizar el estado de la tarea de trabajo. El siguiente código actualiza el mensaje de la cola con contenido nuevo y amplía el tiempo de espera de la visibilidad en 60 segundos más. De este modo, se guarda el estado de trabajo asociado al mensaje y se le proporciona al cliente un minuto más para que siga elaborando el mensaje. Esta técnica se puede utilizar para realizar un seguimiento de los flujos de trabajo de varios pasos en los mensajes en cola, sin que sea necesario volver a empezar desde el principio si se produce un error en un paso del proceso a causa de un error de hardware o software. Normalmente, también mantendría un número de reintentos y, si el mensaje se intentara más de n veces, lo eliminaría. Esto proporciona protección frente a un mensaje que produce un error en la aplicación cada vez que se procesa.

```cpp
// Retrieve storage account from connection-string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the queue client.
azure::storage::cloud_queue_client queue_client = storage_account.create_cloud_queue_client();

// Retrieve a reference to a queue.
azure::storage::cloud_queue queue = queue_client.get_queue_reference(U("my-sample-queue"));

// Get the message from the queue and update the message contents.
// The visibility timeout "0" means make it visible immediately.
// The visibility timeout "60" means the client can get another minute to continue
// working on the message.
azure::storage::cloud_queue_message changed_message = queue.get_message();

changed_message.set_content(U("Changed message"));
queue.update_message(changed_message, std::chrono::seconds(60), true);

// Output the message content.
std::wcout << U("Changed message content: ") << changed_message.content_as_string() << std::endl;  
```

## <a name="how-to-de-queue-the-next-message"></a>Procedimientos: siguiente mensaje de la cola
El código quita un mensaje de una cola en dos pasos. Si se llama a **get_message**, obtiene el siguiente mensaje de una cola. Un mensaje devuelto por **get_message** se hace invisible a cualquier otro código que lea mensajes de esta cola. Para terminar quitando el mensaje de la cola, también debe llamar a **delete_message**. Este proceso de extracción de un mensaje que consta de dos pasos garantiza que si su código no puede procesar un mensaje a causa de un error de hardware o software, otra instancia de su código puede obtener el mismo mensaje e intentarlo de nuevo. El código llama a **delete_message** justo después de que se haya procesado el mensaje.

```cpp
// Retrieve storage account from connection-string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the queue client.
azure::storage::cloud_queue_client queue_client = storage_account.create_cloud_queue_client();

// Retrieve a reference to a queue.
azure::storage::cloud_queue queue = queue_client.get_queue_reference(U("my-sample-queue"));

// Get the next message.
azure::storage::cloud_queue_message dequeued_message = queue.get_message();
std::wcout << U("Dequeued message: ") << dequeued_message.content_as_string() << std::endl;

// Delete the message.
queue.delete_message(dequeued_message);
```

## <a name="how-to-leverage-additional-options-for-de-queuing-messages"></a>Procedimientos: Uso de opciones adicionales para quitar mensajes de la cola
Hay dos formas de personalizar la recuperación de mensajes de una cola. En primer lugar, puede obtener un lote de mensajes (hasta 32). En segundo lugar, puede establecer un tiempo de espera de la invisibilidad más largo o más corto para que el código disponga de más o menos tiempo para procesar cada mensaje. El siguiente ejemplo de código utiliza el método **get_messages** para obtener 20 mensajes en una llamada. A continuación, procesa cada mensaje con un bucle **for** . También establece el tiempo de espera de la invisibilidad en cinco minutos para cada mensaje. Tenga en cuenta que los 5 minutos empiezan a contar para todos los mensajes al mismo tiempo, por lo que después de que pasen los 5 minutos desde la llamada a **get_messages**, todos los mensajes que no se han eliminado volverán a estar visibles.

```cpp
// Retrieve storage account from connection-string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the queue client.
azure::storage::cloud_queue_client queue_client = storage_account.create_cloud_queue_client();

// Retrieve a reference to a queue.
azure::storage::cloud_queue queue = queue_client.get_queue_reference(U("my-sample-queue"));

// Dequeue some queue messages (maximum 32 at a time) and set their visibility timeout to
// 5 minutes (300 seconds).
azure::storage::queue_request_options options;
azure::storage::operation_context context;

// Retrieve 20 messages from the queue with a visibility timeout of 300 seconds.
std::vector<azure::storage::cloud_queue_message> messages = queue.get_messages(20, std::chrono::seconds(300), options, context);

for (auto it = messages.cbegin(); it != messages.cend(); ++it)
{
    // Display the contents of the message.
    std::wcout << U("Get: ") << it->content_as_string() << std::endl;
}
```

## <a name="how-to-get-the-queue-length"></a>Procedimientos: la longitud de la cola
Puede obtener una estimación del número de mensajes existentes en una cola. El método **download_attributes** solicita a Queue service la recuperación de los atributos de la cola, incluido el número de mensajes. El método **approximate_message_count** obtiene el número aproximado de mensajes en la cola.

```cpp
// Retrieve storage account from connection-string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the queue client.
azure::storage::cloud_queue_client queue_client = storage_account.create_cloud_queue_client();

// Retrieve a reference to a queue.
azure::storage::cloud_queue queue = queue_client.get_queue_reference(U("my-sample-queue"));

// Fetch the queue attributes.
queue.download_attributes();

// Retrieve the cached approximate message count.
int cachedMessageCount = queue.approximate_message_count();

// Display number of messages.
std::wcout << U("Number of messages in queue: ") << cachedMessageCount << std::endl;  
```

## <a name="how-to-delete-a-queue"></a>Procedimientos: Eliminación de una cola
Para eliminar una cola y todos los mensajes contenidos en ella, llame al método **delete_queue_if_exists** en el objeto de cola.

```cpp
// Retrieve storage account from connection-string.
azure::storage::cloud_storage_account storage_account = azure::storage::cloud_storage_account::parse(storage_connection_string);

// Create the queue client.
azure::storage::cloud_queue_client queue_client = storage_account.create_cloud_queue_client();

// Retrieve a reference to a queue.
azure::storage::cloud_queue queue = queue_client.get_queue_reference(U("my-sample-queue"));

// If the queue exists and delete it.
queue.delete_queue_if_exists();  
```

## <a name="next-steps"></a>Pasos siguientes
Ahora que está familiarizado con los aspectos básicos de Queue Storage, siga estos vínculos para obtener más información sobre Azure Storage.

* [Cómo usar Blob Storage de C++](../blobs/storage-c-plus-plus-how-to-use-blobs.md)
* [Cómo usar Table Storage de C++](../../cosmos-db/table-storage-how-to-use-c-plus.md)
* [Enumeración de los recursos de Azure Storage en C++](../common/storage-c-plus-plus-enumeration.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json)
* [Referencia de la biblioteca de clientes de almacenamiento para C++](https://azure.github.io/azure-storage-cpp)
* [Documentación de Azure Storage](https://azure.microsoft.com/documentation/services/storage/)
