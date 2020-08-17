---
title: 'Uso de Azure Queue Storage v2.1 desde Python: Azure Storage'
description: Aprenda a usar Azure Queue service v2.1 desde Python para crear y eliminar colas e insertar, obtener y eliminar mensajes.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 09/17/2019
ms.service: storage
ms.subservice: queues
ms.topic: how-to
ms.reviewer: dineshm
ms.custom: seo-javascript-october2019, devx-track-python
ms.openlocfilehash: 97cb41a47b18d403881432098ecdda9bb26146d2
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87848203"
---
# <a name="how-to-use-azure-queue-storage-v21-from-python"></a>Uso de Azure Queue Storage v2.1 desde Python

[!INCLUDE [storage-selector-queue-include](../../../includes/storage-selector-queue-include.md)]

En este artículo se muestran escenarios comunes con el servicio de Azure Queue Storage. Entre los escenarios descritos se incluyen insertar, ojear, obtener y eliminar mensajes de la cola, así como crear y eliminar colas.

> [!IMPORTANT]
> En este artículo se hace referencia a la versión heredada de la biblioteca cliente de Azure Storage para Python. Para empezar a trabajar con la versión más reciente, vea [Inicio rápido: Biblioteca cliente de Azure Queue Storage v12 para Python](storage-quickstart-queues-python.md)

[!INCLUDE [storage-try-azure-tools-queues](../../../includes/storage-try-azure-tools-queues.md)]

## <a name="overview"></a>Información general

Los ejemplos de este artículo están escritos en Python y usan el [SDK de Microsoft Azure Storage para Python]. Para obtener más información sobre las colas, consulte la sección [Pasos siguientes](#next-steps) .

[!INCLUDE [storage-queue-concepts-include](../../../includes/storage-queue-concepts-include.md)]

[!INCLUDE [storage-create-account-include](../../../includes/storage-create-account-include.md)]

## <a name="download-and-install-azure-storage-sdk-for-python"></a>Descarga e instalación del SDK de Azure Storage para Python

El [SDK de Azure Storage para Python](https://github.com/azure/azure-storage-python) requiere la versión de Python 2.7, 3.3 o posterior.
 
### <a name="install-via-pypi"></a>Instalación mediante PyPi

Para realizar la instalación mediante el índice de paquetes de Python (PyPI), escriba:

```bash
pip install azure-storage-queue==2.1.0
```

> [!NOTE]
> Si va a actualizar desde el SDK de Azure Storage para Python 0.36 o una versión anterior, desinstale el SDK anterior mediante `pip uninstall azure-storage` antes de instalar el paquete más reciente.

Si quiere conocer métodos de instalación alternativos, consulte el [SDK de Azure Storage para Python](https://github.com/Azure/azure-storage-python/).

## <a name="view-the-sample-application"></a>Visualización de la aplicación de ejemplo

Para ver y ejecutar una aplicación de ejemplo que muestra cómo usar Python con Azure Queues, consulte [Azure Storage: Getting Started with Azure Queues in Python](https://github.com/Azure-Samples/storage-queue-python-getting-started) (Azure Storage: Introducción a Azure Queues en Python). 

Para ejecutar la aplicación de ejemplo, asegúrese de que ha instalado los paquetes `azure-storage-queue` y `azure-storage-common`.

## <a name="create-a-queue"></a>Creación de una cola

El objeto [QueueService](/python/api/azure-storage-queue/azure.storage.queue.queueservice.queueservice) permite trabajar con colas. El código siguiente crea un objeto `QueueService`. Agregue lo siguiente cerca de la parte superior de todo archivo Python en el que desee obtener acceso a Azure Storage mediante programación:

```python
from azure.storage.queue import QueueService
```

El código siguiente crea un objeto `QueueService` mediante el nombre de la cuenta de almacenamiento y la clave de la cuenta. Reemplace *myaccount* y *mykey* por el nombre y la clave de su cuenta.

```python
queue_service = QueueService(account_name='myaccount', account_key='mykey')

queue_service.create_queue('taskqueue')
```

## <a name="insert-a-message-into-a-queue"></a>un mensaje en una cola

Para insertar un mensaje en una cola, use el método [put_message](/python/api/azure-storage-queue/azure.storage.queue.queueservice.queueservice#put-message-queue-name--content--visibility-timeout-none--time-to-live-none--timeout-none-) para crear un mensaje y agregarlo a la cola.

```python
queue_service.put_message('taskqueue', u'Hello World')
```

Los mensajes de la cola de Azure se almacenan como texto. Si quiere almacenar datos binarios, configure las funciones de codificación y descodificación de Base64 en el objeto del servicio de cola antes de colocar un mensaje en la cola.

```python
# setup queue Base64 encoding and decoding functions
queue_service.encode_function = QueueMessageFormat.binary_base64encode
queue_service.decode_function = QueueMessageFormat.binary_base64decode
```

## <a name="peek-at-the-next-message"></a>siguiente mensaje

Puede ojear el mensaje situado en la parte delantera de una cola, sin quitarlo de ella, llamando al método [peek_message](/python/api/azure-storage-queue/azure.storage.queue.queueservice.queueservice#peek-messages-queue-name--num-messages-none--timeout-none-). De forma predeterminada, `peek_messages` inspecciona un único mensaje.

```python
messages = queue_service.peek_messages('taskqueue')
for message in messages:
    print(message.content)
```

## <a name="dequeue-messages"></a>Retirar mensajes de la cola

El código borra un mensaje de una cola en dos pasos. Si llama a [get_messages](/python/api/azure-storage-queue/azure.storage.queue.queueservice.queueservice#get-messages-queue-name--num-messages-none--visibility-timeout-none--timeout-none-), obtiene, de forma predeterminada, el siguiente mensaje de una cola. Un mensaje devuelto por `get_messages` se hace invisible a cualquier otro código de lectura de mensajes de esta cola. De forma predeterminada, este mensaje permanece invisible durante 30 segundos. Para terminar quitando el mensaje de la cola, también debe llamar a [delete_message](/python/api/azure-storage-queue/azure.storage.queue.queueservice.queueservice#delete-message-queue-name--message-id--pop-receipt--timeout-none-). Este proceso de extracción de un mensaje que consta de dos pasos garantiza que si su código no puede procesar un mensaje a causa de un error de hardware o software, otra instancia de su código puede obtener el mismo mensaje e intentarlo de nuevo. El código siguiente llama a `delete_message` justo después de haberse procesado el mensaje.

```python
messages = queue_service.get_messages('taskqueue')
for message in messages:
    print(message.content)
    queue_service.delete_message('taskqueue', message.id, message.pop_receipt)
```

Hay dos formas de personalizar la recuperación de mensajes de una cola. En primer lugar, puede obtener un lote de mensajes (hasta 32). En segundo lugar, puede establecer un tiempo de espera de la invisibilidad más largo o más corto para que el código disponga de más o menos tiempo para procesar cada mensaje. El siguiente ejemplo de código usa el método `get_messages` para obtener 16 mensajes en una llamada. A continuación, procesa cada mensaje con un bucle for. También establece el tiempo de espera de la invisibilidad en cinco minutos para cada mensaje.

```python
messages = queue_service.get_messages(
    'taskqueue', num_messages=16, visibility_timeout=5*60)
for message in messages:
    print(message.content)
    queue_service.delete_message('taskqueue', message.id, message.pop_receipt)
```

## <a name="change-the-contents-of-a-queued-message"></a>contenido de un mensaje en cola

Puede cambiar el contenido de un mensaje local en la cola. Si el mensaje representa una tarea de trabajo, puede usar esta característica para actualizar el estado de la tarea de trabajo. El código siguiente usa el método [update_message()](/python/api/azure-storage-queue/azure.storage.queue.queueservice.queueservice#update-message-queue-name--message-id--pop-receipt--visibility-timeout--content-none--timeout-none-) para actualizar un mensaje. El tiempo de espera de visibilidad se establece en 0, lo que significa que el mensaje aparece inmediatamente y se actualiza el contenido.

```python
messages = queue_service.get_messages('taskqueue')
for message in messages:
    queue_service.update_message(
        'taskqueue', message.id, message.pop_receipt, 0, u'Hello World Again')
```

## <a name="get-the-queue-length"></a>la longitud de la cola

Puede obtener una estimación del número de mensajes existentes en una cola. El método [get_queue_metadata](/python/api/azure-storage-queue/azure.storage.queue.queueservice.queueservice#get-queue-metadata-queue-name--timeout-none-) solicita a Queue service que devuelva los metadatos sobre la cola y el valor de `approximate_message_count`. El resultado solo es aproximado, ya que se pueden agregar o borrar mensajes después de que el servicio de cola haya respondido su solicitud.

```python
metadata = queue_service.get_queue_metadata('taskqueue')
count = metadata.approximate_message_count
```

## <a name="delete-a-queue"></a>Eliminación de una cola

Para eliminar una cola y todos los mensajes que contiene, llame al método [delete_queue](/python/api/azure-storage-queue/azure.storage.queue.queueservice.queueservice#delete-queue-queue-name--fail-not-exist-false--timeout-none-).

```python
queue_service.delete_queue('taskqueue')
```

## <a name="next-steps"></a>Pasos siguientes

Ahora que ya conoce los aspectos básicos del almacenamiento en cola, siga estos vínculos para saber más.

* [Referencia de la API de Python de colas de Azure](/python/api/azure-storage-queue)
* [Centro para desarrolladores de Python](https://azure.microsoft.com/develop/python/)
* [API de REST de servicios de Azure Storage](https://msdn.microsoft.com/library/azure/dd179355)

[Azure Storage Team Blog]: https://blogs.msdn.com/b/windowsazurestorage/
[SDK de Microsoft Azure Storage para Python]: https://github.com/Azure/azure-storage-python
