---
title: Uso del almacenamiento de colas de Ruby - Azure Storage
description: Aprenda a utilizar el servicio Cola de Azure para crear y eliminar colas e insertar, obtener y eliminar mensajes. Los ejemplos están escritos en Ruby.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 12/08/2016
ms.service: storage
ms.subservice: queues
ms.topic: how-to
ms.reviewer: dineshm
ms.openlocfilehash: ac47747f7789b23801900804522681e5a4b10e5a
ms.sourcegitcommit: 2ffa5bae1545c660d6f3b62f31c4efa69c1e957f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88078706"
---
# <a name="how-to-use-queue-storage-from-ruby"></a>Uso del almacenamiento de colas de Ruby
[!INCLUDE [storage-selector-queue-include](../../../includes/storage-selector-queue-include.md)]

[!INCLUDE [storage-try-azure-tools-queues](../../../includes/storage-try-azure-tools-queues.md)]

## <a name="overview"></a>Información general
Esta guía muestra cómo realizar algunas tareas comunes a través del servicio Microsoft Azure Queue Storage. Los ejemplos están escritos usando la API Ruby de Azure.
Entre los escenarios descritos se incluyen **insertar**, **ojear**, **obtener** y **eliminar** mensajes de la cola, así como **crear y eliminar colas**.

[!INCLUDE [storage-queue-concepts-include](../../../includes/storage-queue-concepts-include.md)]

[!INCLUDE [storage-create-account-include](../../../includes/storage-create-account-include.md)]

## <a name="create-a-ruby-application"></a>Creación de una aplicación de Ruby
Cree una aplicación de Ruby. Para obtener más instrucciones, consulte [Creación de una aplicación de Ruby en App Service en Linux](https://docs.microsoft.com/azure/app-service/quickstart-ruby).

## <a name="configure-your-application-to-access-storage"></a>Configuración de la aplicación para obtener acceso al almacenamiento
Para usar el almacenamiento de Azure tendrá que descargar y usar el paquete Ruby azure, que incluye un conjunto de útiles bibliotecas que se comunican con los servicios REST de almacenamiento.

### <a name="use-rubygems-to-obtain-the-package"></a>Uso de RubyGems para obtener el paquete
1. Use una interfaz de línea de comandos como **PowerShell** (Windows), **Terminal** (Mac) o **Bash** (Unix).
2. Escriba "gem install azure" en la ventana de comandos para instalar la gema y las dependencias.

### <a name="import-the-package"></a>Importación del paquete
Con el editor de texto que prefiera, agregue lo siguiente al principio del archivo de Ruby en el que pretenda utilizar el almacenamiento:

```ruby
require "azure"
```

## <a name="setup-an-azure-storage-connection"></a>Configuración de una conexión de Azure Storage
El módulo azure leerá las variables de entorno **AZURE\_STORAGE\_ACCOUNT** y **AZURE\_STORAGE\_ACCESS_KEY** para obtener información necesaria para conectarse a su cuenta de Azure Storage. Si no se establecen estas variables de entorno, debe especificar la información de la cuenta antes de usar **Azure::QueueService** con el siguiente código:

```ruby
Azure.config.storage_account_name = "<your azure storage account>"
Azure.config.storage_access_key = "<your Azure storage access key>"
```

Para obtener estos valores desde una cuenta de almacenamiento de Azure Resource Manager o clásica en el Portal de Azure:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Vaya a la cuenta de almacenamiento que desea utilizar.
3. En la hoja Configuración que se encuentra a la derecha, haga clic en **Claves de acceso**.
4. En la hoja Claves de acceso que aparece, verá la clave de acceso 1 y 2. Puede usar cualquiera de estas. 
5. Haga clic en el icono de copia para copiar la clave en el Portapapeles. 

## <a name="how-to-create-a-queue"></a>Creación de una cola
El siguiente código crea un objeto **Azure::QueueService** , que permite trabajar con colas.

```ruby
azure_queue_service = Azure::QueueService.new
```

Utilice el método **create_queue()** para crear una cola con el nombre especificado.

```ruby
begin
  azure_queue_service.create_queue("test-queue")
rescue
  puts $!
end
```

## <a name="how-to-insert-a-message-into-a-queue"></a>Inserción de un mensaje en una cola
Para insertar un mensaje en una cola, utilice el método **create_message()** para crear un nuevo mensaje y agregarlo a la cola.

```ruby
azure_queue_service.create_message("test-queue", "test message")
```

## <a name="how-to-peek-at-the-next-message"></a>Inspección del siguiente mensaje
Puede inspeccionar el mensaje situado en la parte delantera de una cola, sin quitarlo de la cola, mediante una llamada al método **peek\_messages()** . De forma predeterminada, **peek\_messages()** inspecciona un único mensaje. También puede indicar cuántos mensajes desea inspeccionar.

```ruby
result = azure_queue_service.peek_messages("test-queue",
  {:number_of_messages => 10})
```

## <a name="how-to-dequeue-the-next-message"></a>Extracción del siguiente mensaje de la cola
Puede borrar un mensaje de una cola en dos pasos.

1. Al llamar a **list\_messages()** , obtiene, de forma predeterminada, el siguiente mensaje en una cola. También puede indicar cuántos mensajes desea obtener. Los mensajes devueltos por **list\_messages()** se hacen invisibles para cualquier otro código que lea mensajes de esta cola. Usted proporciona el tiempo de espera de la visibilidad en segundos a modo de parámetro.
2. Para terminar quitando el mensaje de la cola, también debe llamar a **delete_message()** .

Este proceso de extracción de un mensaje que consta de dos pasos garantiza que si su código no puede procesar un mensaje a causa de un error de hardware o software, otra instancia de su código puede obtener el mismo mensaje e intentarlo de nuevo. El código llama a **delete\_message()** justo después de que se haya procesado el mensaje.

```ruby
messages = azure_queue_service.list_messages("test-queue", 30)
azure_queue_service.delete_message("test-queue", 
  messages[0].id, messages[0].pop_receipt)
```

## <a name="how-to-change-the-contents-of-a-queued-message"></a>Cambio del contenido de un mensaje en cola
Puede cambiar el contenido de un mensaje local en la cola. El código siguiente usa el método **update_message()** para actualizar un mensaje. Este método devolverá una tupla que contiene la recepción de confirmación del mensaje en cola y un valor de fecha y hora UTC que representa el momento en que el mensaje estará visible en la cola.

```ruby
message = azure_queue_service.list_messages("test-queue", 30)
pop_receipt, time_next_visible = azure_queue_service.update_message(
  "test-queue", message.id, message.pop_receipt, "updated test message", 
  30)
```

## <a name="how-to-additional-options-for-dequeuing-messages"></a>Opciones adicionales para extraer mensajes de la cola
Hay dos formas de personalizar la recuperación de mensajes de una cola.

1. En primer lugar, puede obtener un lote de mensajes.
2. En segundo lugar, puede establecer un tiempo de espera de la invisibilidad más largo o más corto para que el código disponga de más o menos tiempo para procesar cada mensaje.

El siguiente ejemplo de código utiliza el método **list\_messages()** para obtener 15 mensajes en una llamada. A continuación, imprime y elimina cada mensaje. También establece el tiempo de espera de la invisibilidad en cinco minutos para cada mensaje.

```ruby
azure_queue_service.list_messages("test-queue", 300
  {:number_of_messages => 15}).each do |m|
  puts m.message_text
  azure_queue_service.delete_message("test-queue", m.id, m.pop_receipt)
end
```

## <a name="how-to-get-the-queue-length"></a>Obtención de la longitud de la cola
Puede obtener una estimación del número de mensajes existentes en la cola. El método **get\_queue\_metadata()** solicita a Queue service que devuelva el recuento aproximado de mensajes y metadatos sobre la cola.

```ruby
message_count, metadata = azure_queue_service.get_queue_metadata(
  "test-queue")
```

## <a name="how-to-delete-a-queue"></a>Eliminación de una cola
Para eliminar una cola y todos los mensajes contenidos en ella, llame al método **delete\_queue()** en el objeto de cola.

```ruby
azure_queue_service.delete_queue("test-queue")
```

## <a name="next-steps"></a>Pasos siguientes
Ahora que está familiarizado con los aspectos básicos del almacenamiento de colas, utilice estos vínculos para obtener más información acerca de tareas de almacenamiento más complejas.

* Visite el [blog del equipo de Azure Storage](https://blogs.msdn.com/b/windowsazurestorage/)
* Visite el repositorio de [SDK de Azure para Ruby](https://github.com/WindowsAzure/azure-sdk-for-ruby) en GitHub

Podrá encontrar una comparación entre Azure Queue Service, que se explica en este artículo, y Azure Service Bus Queues, que se explican en el artículo [Utilización de las colas de Service Bus](https://azure.microsoft.com/develop/ruby/how-to-guides/service-bus-queues/), en el documento [Colas de Azure y Colas de Service Bus: comparación y diferencias](../../service-bus-messaging/service-bus-azure-and-service-bus-queues-compared-contrasted.md).
