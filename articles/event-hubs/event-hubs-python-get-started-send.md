---
title: Envío o recepción de eventos desde Azure Event Hubs mediante Python (más reciente)
description: Este artículo es un tutorial para crear una aplicación de Python que envíe eventos a Azure Event Hubs, o los reciba de él, mediante el más reciente paquete de la versión 5 de azure/event-hubs.
ms.topic: quickstart
ms.date: 02/11/2020
ms.openlocfilehash: b6a30ba0cef8c460a2a3035b3ab40fd8173d7b2e
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88933909"
---
# <a name="send-events-to-or-receive-events-from-event-hubs-by-using-python-azure-eventhub-version-5"></a>Envío o recepción de eventos en Event Hubs mediante Python (azure-eventhub versión 5)
En este inicio rápido se muestra cómo enviar y recibir eventos desde un centro de eventos mediante el paquete de Python **azure-eventhub, versión 5**.

## <a name="prerequisites"></a>Requisitos previos
Si es la primera vez que usa Azure Event Hubs, consulte la [información general de Event Hubs](event-hubs-about.md) antes de continuar con este inicio rápido. 

Para completar este tutorial de inicio rápido, debe cumplir los siguientes requisitos previos:

- Una **suscripción a Microsoft Azure**. Para usar los servicios de Azure, entre los que se incluye Azure Event Hubs, se necesita una suscripción.  Si no se dispone de una cuenta de Azure, es posible registrarse para obtener una [evaluación gratuita](https://azure.microsoft.com/free/), o bien usar las ventajas que disfrutan los suscriptores MSDN al [crear una cuenta](https://azure.microsoft.com).
- Python 2.7, 3.5 o cualquier versión posterior, con PIP instalado y actualizado.
- El paquete de Python para Event Hubs. 

    Para instalar el paquete, ejecute este comando en un símbolo del sistema que tenga Python en su ruta de acceso:

    ```cmd
    pip install azure-eventhub
    ```

    Instale el siguiente paquete para recibir los eventos y usar Azure Blob Storage como almacén de puntos de control:

    ```cmd
    pip install azure-eventhub-checkpointstoreblob-aio
    ```
- **Creación de un espacio de nombres de Event Hubs y un centro de eventos**. El primer paso consiste en usar [Azure Portal](https://portal.azure.com) para crear un espacio de nombres de tipo Event Hubs y obtener las credenciales de administración que la aplicación necesita para comunicarse con el centro de eventos. Para crear un espacio de nombres y un centro de eventos, siga el procedimiento que se indica en [este artículo](event-hubs-create.md). Después, obtenga la **cadena de conexión para el espacio de nombres de Event Hubs**. Para ello, siga las instrucciones del artículo: [Obtenga la cadena de conexión](event-hubs-get-connection-string.md#get-connection-string-from-the-portal). La utilizará más adelante en este inicio rápido.

## <a name="send-events"></a>Envío de eventos
En esta sección, se crea un script de Python para enviar eventos al centro de eventos que creó anteriormente.

1. Abra el editor de Python que prefiera, como [Visual Studio Code](https://code.visualstudio.com/).
2. Cree un script llamado *send.py*. Este script envía un lote de eventos al centro de eventos que creó anteriormente.
3. Pegue el siguiente código en *send.py*:

    ```python
    import asyncio
    from azure.eventhub.aio import EventHubProducerClient
    from azure.eventhub import EventData

    async def run():
        # Create a producer client to send messages to the event hub.
        # Specify a connection string to your event hubs namespace and
            # the event hub name.
        producer = EventHubProducerClient.from_connection_string(conn_str="EVENT HUBS NAMESPACE - CONNECTION STRING", eventhub_name="EVENT HUB NAME")
        async with producer:
            # Create a batch.
            event_data_batch = await producer.create_batch()

            # Add events to the batch.
            event_data_batch.add(EventData('First event '))
            event_data_batch.add(EventData('Second event'))
            event_data_batch.add(EventData('Third event'))

            # Send the batch of events to the event hub.
            await producer.send_batch(event_data_batch)

    loop = asyncio.get_event_loop()
    loop.run_until_complete(run())

    ```

    > [!NOTE]
    > Para ver todo el código fuente, incluidos los comentarios informativos, vaya a la [página send_async de GitHub](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/eventhub/azure-eventhub/samples/async_samples/send_async.py).
    

## <a name="receive-events"></a>Recepción de eventos
En este inicio rápido se usa Azure Blob Storage como almacén de puntos de control. El almacén de puntos de control se usa para conservar los puntos de control (es decir, las últimas posiciones de lectura).  

> [!NOTE]
> Si trabaja en Azure Stack Hub, esa plataforma puede admitir una versión diferente del SDK de Blob Storage que las que suelen estar disponibles en Azure. Por ejemplo, si trabaja [en la versión 2002 de Azure Stack Hub](/azure-stack/user/event-hubs-overview), la versión más alta disponible para el servicio Storage es 2017-11-09. En este caso, además de seguir los pasos de esta sección, también tendrá que agregar código para usar como destino la versión 2017-11-09 de la API del servicio de almacenamiento. Para ver un ejemplo de cómo usar como destino una versión de la API de Storage, consulte los ejemplos [sincrónicos](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/eventhub/azure-eventhub-checkpointstoreblob/samples/receive_events_using_checkpoint_store_storage_api_version.py) y [asincrónicos](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/eventhub/azure-eventhub-checkpointstoreblob-aio/samples/receive_events_using_checkpoint_store_storage_api_version_async.py) en GitHub. Para más información sobre las versiones del servicio Azure Storage que se admiten en Azure Stack Hub, consulte [Almacenamiento de Azure Stack Hub: diferencias y consideraciones](/azure-stack/user/azure-stack-acs-differences).


### <a name="create-an-azure-storage-account-and-a-blob-container"></a>Creación de una cuenta de Azure Storage y un contenedor de blobs
Cree una cuenta de Azure Storage y un contenedor de blobs en ella, para lo que debe seguir estos pasos:

1. [Creación de una cuenta de Azure Storage](../storage/common/storage-account-create.md?tabs=azure-portal)
2. [Creación de un contenedor de blobs](../storage/blobs/storage-quickstart-blobs-portal.md#create-a-container)
3. [Obtención de la cadena de conexión para una cuenta de almacenamiento](../storage/common/storage-configure-connection-string.md)

Asegúrese de registrar la cadena de conexión y el nombre del contenedor para usarlo posteriormente en el código de recepción.


### <a name="create-a-python-script-to-receive-events"></a>Creación de un script de Python para recibir eventos

En esta sección, creará un script de Python para recibir eventos del centro de eventos:

1. Abra el editor de Python que prefiera, como [Visual Studio Code](https://code.visualstudio.com/).
2. Cree un script llamado *recv.py*.
3. Pegue el siguiente código en *recv.py*:

    ```python
    import asyncio
    from azure.eventhub.aio import EventHubConsumerClient
    from azure.eventhub.extensions.checkpointstoreblobaio import BlobCheckpointStore


    async def on_event(partition_context, event):
        # Print the event data.
        print("Received the event: \"{}\" from the partition with ID: \"{}\"".format(event.body_as_str(encoding='UTF-8'), partition_context.partition_id))

        # Update the checkpoint so that the program doesn't read the events
        # that it has already read when you run it next time.
        await partition_context.update_checkpoint(event)

    async def main():
        # Create an Azure blob checkpoint store to store the checkpoints.
        checkpoint_store = BlobCheckpointStore.from_connection_string("AZURE STORAGE CONNECTION STRING", "BLOB CONTAINER NAME")

        # Create a consumer client for the event hub.
        client = EventHubConsumerClient.from_connection_string("EVENT HUBS NAMESPACE CONNECTION STRING", consumer_group="$Default", eventhub_name="EVENT HUB NAME", checkpoint_store=checkpoint_store)
        async with client:
            # Call the receive method. Read from the beginning of the partition (starting_position: "-1")
            await client.receive(on_event=on_event,  starting_position="-1")

    if __name__ == '__main__':
        loop = asyncio.get_event_loop()
        # Run the main method.
        loop.run_until_complete(main())    
    ```

    > [!NOTE]
    > Para ver todo el código fuente, incluidos los comentarios informativos adicionales, vaya a la [página recv_with_checkpoint_store_async.py de GitHub](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/eventhub/azure-eventhub/samples/async_samples/recv_with_checkpoint_store_async.py).


### <a name="run-the-receiver-app"></a>Ejecución de la aplicación del receptor

Para ejecutar el script, abra un símbolo del sistema que tenga Python en su ruta de acceso y, después, ejecute el siguiente comando:

```bash
python recv.py
```

### <a name="run-the-sender-app"></a>Ejecución de la aplicación del remitente

Para ejecutar el script, abra un símbolo del sistema que tenga Python en su ruta de acceso y, después, ejecute el siguiente comando:

```bash
python send.py
```

La ventana del destinatario debería mostrar los mensajes que se enviaron al centro de eventos.


## <a name="next-steps"></a>Pasos siguientes
En este inicio rápido, ha enviado y recibido eventos de forma asincrónica. Para aprender a enviar y recibir eventos de forma sincrónica, vaya a la [página sync_samples de GitHub](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/eventhub/azure-eventhub/samples/sync_samples).

Para ver todos los ejemplos (tanto sincrónicos como asincrónicos) de GitHub, vaya a [Biblioteca cliente de Azure Event Hubs para ejemplos de Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/eventhub/azure-eventhub/samples).
