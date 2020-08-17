---
title: 'Inicio rápido: Biblioteca de Azure Queue Storage v12 para Python'
description: Aprenda a usar la biblioteca de Azure Queue v12 para Python para crear una cola y agregar mensajes a la cola. A continuación, aprenderá a leer y eliminar los mensajes de la cola. También aprenderá a eliminar una cola.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 12/10/2019
ms.service: storage
ms.subservice: queues
ms.topic: quickstart
ms.custom: devx-track-python
ms.openlocfilehash: 12c2652b4dcef46c5affde2c3fb9ef9288176eb9
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87852266"
---
# <a name="quickstart-azure-queue-storage-client-library-v12-for-python"></a>Inicio rápido: Biblioteca cliente de Azure Queue Storage v12 para Python

Comience a trabajar con la biblioteca cliente de Azure Queue Storage versión 12 para Python. Azure Queue Storage es un servicio para almacenar grandes cantidades de mensajes para su posterior recuperación y procesamiento. Siga estos pasos para instalar el paquete y probar el código de ejemplo para realizar tareas básicas.

Use la biblioteca cliente de Azure Queue Storage v12 para Python para realizar lo siguiente:

* Creación de una cola
* Adición de mensajes a una cola
* Lectura de los mensajes de una cola
* Eliminación de un mensaje de una cola
* mensajes de una cola
* Eliminación de mensajes de una cola
* Eliminación de una cola

Recursos adicionales:

* [Documentación de referencia de API](https://docs.microsoft.com/python/api/azure-storage-queue/index)
* [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-queue)
* [Paquete (índice de paquetes de Python)](https://pypi.org/project/azure-storage-queue/)
* [Muestras](https://docs.microsoft.com/azure/storage/common/storage-samples-python?toc=%2fazure%2fstorage%2fqueues%2ftoc.json#queue-samples)

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/)
* Una cuenta de Azure Storage: [cree una cuenta de almacenamiento](https://docs.microsoft.com/azure/storage/common/storage-quickstart-create-account)
* [Python](https://www.python.org/downloads/) para su sistema operativo: 2.7, 3.5 o superior

## <a name="setting-up"></a>Instalación

En esta sección se explica cómo preparar un proyecto para que funcione con la biblioteca cliente de Azure Queue Storage v12 para Python.

### <a name="create-the-project"></a>Creación del proyecto

Cree una aplicación Python llamada *queues-quickstart-v12*.

1. En una ventana de la consola (como cmd, PowerShell o Bash), cree un directorio para el proyecto.

    ```console
    mkdir queues-quickstart-v12
    ```

1. Cambie al directorio *queues-quickstart-v12* recién creado.

    ```console
    cd queues-quickstart-v12
    ```

### <a name="install-the-package"></a>Instalar el paquete

Instale el paquete de la biblioteca cliente de Azure Blob Storage para Python con el comando `pip install`.

```console
pip install azure-storage-queue
```

Este comando instala el paquete de la biblioteca cliente de Azure Queue Storage para Python y todas las bibliotecas de las que depende. En este caso, se trata solo de la biblioteca principal de Azure para Python.

### <a name="set-up-the-app-framework"></a>Instalación del marco de la aplicación

1. Abra un nuevo archivo de texto en el editor de código.
1. Agregue instrucciones `import`.
1. Cree la estructura del programa, incluido un control de excepciones muy básico.

    Este es el código:

    ```python
    import os, uuid
    from azure.storage.queue import QueueServiceClient, QueueClient, QueueMessage

    try:
        print("Azure Queue storage v12 - Python quickstart sample")
        # Quick start code goes here
    except Exception as ex:
        print('Exception:')
        print(ex)

    ```

1. Guarde el nuevo archivo como *queues-quickstart-v12.py* en el directorio *queues-quickstart-v12*.

[!INCLUDE [storage-quickstart-credentials-include](../../../includes/storage-quickstart-credentials-include.md)]

## <a name="object-model"></a>Modelo de objetos

Azure Queue Storage es un servicio para almacenar grandes cantidades de mensajes, Un mensaje de la cola puede llegar a tener hasta 64 KB. Una cola puede contener millones de mensajes, hasta el límite de capacidad total de una cuenta de almacenamiento. Las colas se utilizan normalmente para crear un trabajo pendiente del trabajo que se va a procesar de forma asincrónica. Queue Storage ofrece tres tipos de recursos:

* La cuenta de almacenamiento
* Cola en la cuenta de almacenamiento
* Mensajes dentro de la cola

En el siguiente diagrama se muestra la relación entre estos recursos.

![Diagrama de la arquitectura de Queue Storage](./media/storage-queues-introduction/queue1.png)

Use las siguientes clases de Python para interactuar con estos recursos:

* [QueueServiceClient](https://docs.microsoft.com/python/api/azure-storage-queue/azure.storage.queue.queueserviceclient): La clase `QueueServiceClient` permite administrar todas las colas de la cuenta de almacenamiento.
* [QueueClient](https://docs.microsoft.com/python/api/azure-storage-queue/azure.storage.queue.queueclient): La clase `QueueClient` permite administrar y manipular una cola individual y sus mensajes.
* [QueueMessage](https://docs.microsoft.com/python/api/azure-storage-queue/azure.storage.queue.queuemessage): La clase `QueueMessage` representa los objetos individuales devueltos al llamar a [receive_messages](https://docs.microsoft.com/python/api/azure-storage-queue/azure.storage.queue.queueclient#receive-messages---kwargs-) en una cola.

## <a name="code-examples"></a>Ejemplos de código

Estos fragmentos de código de ejemplo muestran cómo realizar las siguientes acciones con la biblioteca cliente de Azure Queue Storage para Python:

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

Agregue este código dentro del bloque `try`:

```python
    # Retrieve the connection string for use with the application. The storage
    # connection string is stored in an environment variable on the machine
    # running the application called AZURE_STORAGE_CONNECTION_STRING. If the
    # environment variable is created after the application is launched in a
    # console or with Visual Studio, the shell or application needs to be
    # closed and reloaded to take the environment variable into account.
    connect_str = os.getenv('AZURE_STORAGE_CONNECTION_STRING')
```

### <a name="create-a-queue"></a>Creación de una cola

Decida un nombre para la nueva cola. El código siguiente anexa un valor de UUID al nombre de la cola para asegurarse de que sea único.

> [!IMPORTANT]
> Los nombres de la cola solo puede incluir letras minúsculas, números y guiones y debe empezar por una letra o un número. Antes y después de cada guion debe ir un carácter que no sea otro guión. El nombre debe tener entre 3 y 63 caracteres. Para más información sobre la nomenclatura de las colas, consulte [Asignación de nombres a colas y metadatos](https://docs.microsoft.com/rest/api/storageservices/naming-queues-and-metadata).

Cree una instancia de la clase [QueueClient](https://docs.microsoft.com/python/api/azure-storage-queue/azure.storage.queue.queueclient). A continuación, llame al método [create_queue](https://docs.microsoft.com/python/api/azure-storage-queue/azure.storage.queue.queueclient#create-queue---kwargs-) para crear la cola en la cuenta de almacenamiento.

Agregue este código al final del bloque `try`:

```python
    # Create a unique name for the queue
    queue_name = "quickstartqueues-" + str(uuid.uuid4())

    print("Creating queue: " + queue_name)

    # Instantiate a QueueClient which will be
    # used to create and manipulate the queue
    queue_client = QueueClient.from_connection_string(connect_str, queue_name)

    # Create the queue
    queue_client.create_queue()
```

### <a name="add-messages-to-a-queue"></a>Adición de mensajes a una cola

El siguiente fragmento de código agrega mensajes a la cola mediante una llamada al método [send_message](https://docs.microsoft.com/python/api/azure-storage-queue/azure.storage.queue.queueclient#send-message-content----kwargs-). También guarda la clase [QueueMessage](https://docs.microsoft.com/python/api/azure-storage-queue/azure.storage.queue.queuemessage) devuelta desde la tercera llamada a `send_message`. La clase `saved_message` se utiliza para actualizar el contenido del mensaje más adelante en el programa.

Agregue este código al final del bloque `try`:

```python
    print("\nAdding messages to the queue...")

    # Send several messages to the queue
    queue_client.send_message(u"First message")
    queue_client.send_message(u"Second message")
    saved_message = queue_client.send_message(u"Third message")
```

### <a name="peek-at-messages-in-a-queue"></a>Lectura de los mensajes de una cola

Lea los mensajes de la cola llamando al método [peek_messages](https://docs.microsoft.com/python/api/azure-storage-queue/azure.storage.queue.queueclient#peek-messages-max-messages-none----kwargs-). El método `peek_messages` recupera uno o varios mensajes de la parte delantera de la cola, pero no modifica la visibilidad del mensaje.

Agregue este código al final del bloque `try`:

```python
    print("\nPeek at the messages in the queue...")

    # Peek at messages in the queue
    peeked_messages = queue_client.peek_messages(max_messages=5)

    for peeked_message in peeked_messages:
        # Display the message
        print("Message: " + peeked_message.content)
```

### <a name="update-a-message-in-a-queue"></a>Eliminación de un mensaje de una cola

Actualice el contenido de un mensaje mediante la llamada al método [update_message](https://docs.microsoft.com/python/api/azure-storage-queue/azure.storage.queue.queueclient#update-message-message--pop-receipt-none--content-none----kwargs-). El método `update_message` puede cambiar el tiempo de espera de visibilidad y el contenido de un mensaje. El contenido del mensaje debe ser una cadena con codificación UTF-8 de hasta 64 KB de tamaño. Junto con el nuevo contenido, pase los valores del mensaje que se guardó anteriormente en el código. Los valores `saved_message` identifican el mensaje que se va a actualizar.

```python
    print("\nUpdating the third message in the queue...")

    # Update a message using the message saved when calling send_message earlier
    queue_client.update_message(saved_message, pop_receipt=saved_message.pop_receipt, \
        content="Third message has been updated")
```

### <a name="receive-messages-from-a-queue"></a>mensajes de una cola

Descargue los mensajes agregados anteriormente mediante la llamada al método [receive_messages](https://docs.microsoft.com/python/api/azure-storage-queue/azure.storage.queue.queueclient#receive-messages---kwargs-).

Agregue este código al final del bloque `try`:

```python
    print("\nReceiving messages from the queue...")

    # Get messages from the queue
    messages = queue_client.receive_messages(messages_per_page=5)
```

### <a name="delete-messages-from-a-queue"></a>Eliminación de mensajes de una cola

Elimine los mensajes de la cola una vez recibidos y procesados. En este caso, el procesamiento solo muestra el mensaje en la consola.

La aplicación se detiene para la entrada del usuario mediante una llamada a `input` antes de procesar y eliminar los mensajes. Compruebe en [Azure Portal](https://portal.azure.com) que los recursos se crearon correctamente, antes de que se eliminen. Los mensajes que no se eliminen explícitamente volverán a estar visibles en la cola para que se procesen.

Agregue este código al final del bloque `try`:

```python
    print("\nPress Enter key to 'process' messages and delete them from the queue...")
    input()

    for msg_batch in messages.by_page():
            for msg in msg_batch:
                # "Process" the message
                print(msg.content)
                # Let the service know we're finished with
                # the message and it can be safely deleted.
                queue_client.delete_message(msg)
```

### <a name="delete-a-queue"></a>Eliminación de una cola

El código siguiente limpia los recursos que creó la aplicación; para ello, elimina la cola mediante el método [delete_queue](https://docs.microsoft.com/python/api/azure-storage-queue/azure.storage.queue.queueclient#delete-queue---kwargs-).

Agregue este código al final del bloque `try` y guarde el archivo:

```python
    print("\nPress Enter key to delete the queue...")
    input()

    # Clean up
    print("Deleting queue...")
    queue_client.delete_queue()

    print("Done")
```

## <a name="run-the-code"></a>Ejecución del código

Esta aplicación crea y agrega tres mensajes a una cola de Azure. El código muestra los mensajes en la cola y, a continuación, los recupera y los elimina antes de eliminar la cola.

En la ventana de la consola, vaya al directorio que contiene el archivo *queues-quickstart-v12.py* y, a continuación, ejecute el siguiente comando `python` para ejecutar la aplicación.

```console
python queues-quickstart-v12.py
```

La salida de la aplicación es similar a la del ejemplo siguiente:

```output
Azure Queue storage v12 - Python quickstart sample
Creating queue: quickstartqueues-cac365be-7ce6-4065-bd65-3756ea052cb8

Adding messages to the queue...

Peek at the messages in the queue...
Message: First message
Message: Second message
Message: Third message

Updating the third message in the queue...

Receiving messages from the queue...

Press Enter key to 'process' messages and delete them from the queue...

First message
Second message
Third message has been updated

Press Enter key to delete the queue...

Deleting queue...
Done
```

Cuando la aplicación se detiene antes de recibir mensajes, compruebe la cuenta de almacenamiento en [Azure Portal](https://portal.azure.com). Compruebe que los mensajes están en la cola.

Presione la tecla **Entrar** para recibir y eliminar los mensajes. Cuando se le solicite, presione de nuevo la tecla **Entrar** para eliminar la cola y finalizar la demostración.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha aprendido a crear una cola y a agregarle mensajes mediante código de Python. Después aprendió a ver, recuperar y eliminar mensajes. Por último, ha aprendido a eliminar una cola de mensajes.

Para ver tutoriales, ejemplos, artículos de inicio rápido y otra documentación, visite:

> [!div class="nextstepaction"]
> [Azure para desarrolladores de Python](https://docs.microsoft.com/azure/python/)

* Para más información, consulte [Bibliotecas de Azure Storage para Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage).
* Para ver más aplicaciones de ejemplo de Azure Queue Storage, continúe con los [ejemplos de la biblioteca cliente de Python v12 de Azure Queue Storage](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/storage/azure-storage-queue/samples).
