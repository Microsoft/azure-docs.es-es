---
title: 'Uso de Azure Queue Storage de Node.js: Azure Storage'
description: Aprenda a utilizar el servicio Cola de Azure para crear y eliminar colas e insertar, obtener y eliminar mensajes. Ejemplos escritos en Node.js.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 12/08/2016
ms.service: storage
ms.subservice: queues
ms.topic: how-to
ms.reviewer: dineshm
ms.custom: seo-javascript-september2019, devx-track-javascript
ms.openlocfilehash: 56099a7d350a29db5a57708bc87b7174cfdb3b29
ms.sourcegitcommit: 2ffa5bae1545c660d6f3b62f31c4efa69c1e957f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88079286"
---
# <a name="use-azure-queue-service-to-create-and-delete-queues-from-nodejs"></a>Uso de Azure Queue service para crear y eliminar colas de Node.js
[!INCLUDE [storage-selector-queue-include](../../../includes/storage-selector-queue-include.md)]

[!INCLUDE [storage-check-out-samples-all](../../../includes/storage-check-out-samples-all.md)]

## <a name="overview"></a>Información general
Esta guía le indicará cómo actuar en situaciones habituales usando el servicio Cola de Microsoft Azure. Los ejemplos están escritos usando la API Node.js. Entre los escenarios descritos se incluyen **insertar**, **ojear**, **obtener** y **eliminar** mensajes de la cola, así como **crear y eliminar colas**.

> [!IMPORTANT]
> En este artículo se hace referencia a la versión heredada de la biblioteca cliente de Azure Storage para JavaScript. Para empezar a trabajar con la versión más reciente, consulte [Inicio rápido: Biblioteca cliente de Azure Queue Storage v12 para JavaScript](storage-quickstart-queues-nodejs.md).

[!INCLUDE [storage-queue-concepts-include](../../../includes/storage-queue-concepts-include.md)]

[!INCLUDE [storage-create-account-include](../../../includes/storage-create-account-include.md)]

## <a name="create-a-nodejs-application"></a>Creación de una aplicación Node.js
Cree una aplicación Node.js vacía. Para obtener instrucciones sobre cómo crear una aplicación Node.js, consulte [Creación de una aplicación web Node.js en Azure App Service](../../app-service/quickstart-nodejs.md), [Creación e implementación de una aplicación Node.js en Azure Cloud Services](../../cloud-services/cloud-services-nodejs-develop-deploy-app.md) con Windows PowerShell o [Visual Studio Code](https://code.visualstudio.com/docs/nodejs/nodejs-tutorial).

## <a name="configure-your-application-to-access-storage"></a>Configuración de la aplicación para obtener acceso al almacenamiento
Para usar el almacenamiento de Azure necesitará el SDK de Azure Storage para Node.js, que incluye un conjunto de útiles bibliotecas que se comunican con los servicios REST de almacenamiento.

### <a name="use-node-package-manager-npm-to-obtain-the-package"></a>Uso del Administrador de paquetes para Node (NPM) para obtener el paquete
1. Utilice una interfaz de línea de comandos como **PowerShell** (Windows), **Terminal** (Mac) o **Bash** (Unix) y vaya a la carpeta donde ha creado la aplicación de ejemplo.
2. Escriba **npm install azure-storage** en la ventana de comandos. La salida del comando es similar al ejemplo siguiente.
 
    ```bash
    azure-storage@0.5.0 node_modules\azure-storage
    +-- extend@1.2.1
    +-- xmlbuilder@0.4.3
    +-- mime@1.2.11
    +-- node-uuid@1.4.3
    +-- validator@3.22.2
    +-- underscore@1.4.4
    +-- readable-stream@1.0.33 (string_decoder@0.10.31, isarray@0.0.1, inherits@2.0.1, core-util-is@1.0.1)
    +-- xml2js@0.2.7 (sax@0.5.2)
    +-- request@2.57.0 (caseless@0.10.0, aws-sign2@0.5.0, forever-agent@0.6.1, stringstream@0.0.4, oauth-sign@0.8.0, tunnel-agent@0.4.1, isstream@0.1.2, json-stringify-safe@5.0.1, bl@0.9.4, combined-stream@1.0.5, qs@3.1.0, mime-types@2.0.14, form-data@0.2.0, http-signature@0.11.0, tough-cookie@2.0.0, hawk@2.3.1, har-validator@1.8.0)
    ```

3. Puede ejecutar manualmente el comando **ls** para comprobar si se ha creado la carpeta **node\_modules**. Dentro de dicha carpeta, encontrará el paquete **azure-storage** , que contiene las bibliotecas necesarias para el acceso al almacenamiento.

### <a name="import-the-package"></a>Importación del paquete
Con el Bloc de notas u otro editor de texto, agregue lo siguiente en la parte superior del archivo **server.js** de la aplicación en la que pretenda usar el almacenamiento:

```javascript
var azure = require('azure-storage');
```

## <a name="setup-an-azure-storage-connection"></a>Configuración de una conexión de Azure Storage
El módulo azure leerá las variables de entorno AZURE\_STORAGE\_ACCOUNT y AZURE\_STORAGE\_ACCESS\_KEY o AZURE\_STORAGE\_CONNECTION\_STRING para obtener información necesaria para conectarse a su cuenta de Azure Storage. Si no se configuran estas variables de entorno, debe especificar la información de la cuenta al llamar a **createQueueService**.

## <a name="how-to-create-a-queue"></a>Instrucciones: Creación de una cola
El siguiente código crea un objeto **QueueService** , que le permite trabajar con colas.

```javascript
var queueSvc = azure.createQueueService();
```

Utilice el método **createQueueIfNotExists** , que devuelve la cola especificada si ya existe o crea una nueva cola con el nombre especificado si todavía no existe.

```javascript
queueSvc.createQueueIfNotExists('myqueue', function(error, results, response){
  if(!error){
    // Queue created or exists
  }
});
```

Si la cola se crea, `result.created` es verdadero. Si la cola existe, `result.created` es falso.

### <a name="filters"></a>Filtros
Las operaciones de filtrado opcionales pueden aplicarse a las tareas realizadas mediante **QueueService**. Las operaciones de filtrado pueden incluir registros, reintentos automáticos, etc. Los filtros son objetos que implementan un método con la firma:

```javascript
function handle (requestOptions, next)
```

Después de realizar el preprocesamiento en las opciones de solicitud, el método tiene que llamar a "next" pasando una devolución de llamada con la firma siguiente:

```javascript
function (returnObject, finalCallback, next)
```

En esta devolución de llamada y después de procesar returnObject (la respuesta de la solicitud al servidor), la devolución de llamada tiene que invocar a next, si existe, para continuar procesando otros filtros, o bien simplemente invocar a finalCallback para finalizar la invocación del servicio.

Se incluyen dos filtros que implementan la lógica de reintento con el SDK de Azure para Node.js: **ExponentialRetryPolicyFilter** y **LinearRetryPolicyFilter**. Lo siguiente crea un objeto **QueueService** que usa **ExponentialRetryPolicyFilter**:

```javascript
var retryOperations = new azure.ExponentialRetryPolicyFilter();
var queueSvc = azure.createQueueService().withFilter(retryOperations);
```

## <a name="how-to-insert-a-message-into-a-queue"></a>Instrucciones: Inserción de un mensaje en una cola
Para insertar un mensaje en una cola, utilice el método **createMessage** para crear un nuevo mensaje y agregarlo a la cola.

```javascript
queueSvc.createMessage('myqueue', "Hello world!", function(error, results, response){
  if(!error){
    // Message inserted
  }
});
```

## <a name="how-to-peek-at-the-next-message"></a>Instrucciones: Inspección del siguiente mensaje
Puede inspeccionar el mensaje situado en la parte delantera de una cola, sin quitarlo de la cola, mediante una llamada al método **peekMessages** . De forma predeterminada, **peekMessages** inspecciona un único mensaje.

```javascript
queueSvc.peekMessages('myqueue', function(error, results, response){
  if(!error){
    // Message text is in results[0].messageText
  }
});
```

El `result` contiene el mensaje.

> [!NOTE]
> Si se usa **peekMessages** cuando no existen mensajes en la cola, no se devolverá un error, pero tampoco se devolverán mensajes.
> 
> 

## <a name="how-to-dequeue-the-next-message"></a>Instrucciones: Extracción del siguiente mensaje de la cola
El procesamiento de un mensaje es un proceso que consta de dos etapas:

1. Extracción del mensaje de la cola.
2. Eliminación del mensaje.

Para quitar un mensaje de la cola, use **getMessages**. De esta forma los mensajes se hacen invisible en la cola, así que ningún otro cliente puede procesarlos. Después de que la aplicación haya procesado un mensaje, llame a **deleteMessage** para eliminarlo de la cola. En el siguiente ejemplo se obtiene un mensaje y luego se elimina:

```javascript
queueSvc.getMessages('myqueue', function(error, results, response){
  if(!error){
    // Message text is in results[0].messageText
    var message = results[0];
    queueSvc.deleteMessage('myqueue', message.messageId, message.popReceipt, function(error, response){
      if(!error){
        //message deleted
      }
    });
  }
});
```

> [!NOTE]
> De manera predeterminada, un mensaje solo está oculto durante 30 segundos, después de lo cual es visible para otros clientes. Puede especificar un valor diferente usando `options.visibilityTimeout` con **getMessages**.
> 
> [!NOTE]
> Si usa **getMessages** cuando no existen mensajes en la cola, no se devolverá un error, pero tampoco se devolverán mensajes.
> 
> 

## <a name="how-to-change-the-contents-of-a-queued-message"></a>Instrucciones: Cambio del contenido de un mensaje en cola
Puede cambiar el contenido de un mensaje local en la cola mediante **updateMessage**. En el ejemplo siguiente se actualiza el texto de un mensaje:

```javascript
queueSvc.getMessages('myqueue', function(error, getResults, getResponse){
  if(!error){
    // Got the message
    var message = getResults[0];
    queueSvc.updateMessage('myqueue', message.messageId, message.popReceipt, 10, {messageText: 'new text'}, function(error, updateResults, updateResponse){
      if(!error){
        // Message updated successfully
      }
    });
  }
});
```

## <a name="how-to-additional-options-for-dequeuing-messages"></a>Instrucciones: Opciones adicionales para quitar mensajes de la cola
Hay dos formas de personalizar la recuperación de mensajes de una cola:

* `options.numOfMessages` - Recuperar un lote de mensajes (hasta 32).
* `options.visibilityTimeout` - Establecer un tiempo de espera de invisibilidad más largo o más corto.

En el siguiente ejemplo se usa el método **getMessages** para obtener 15 mensajes en una llamada. A continuación, procesa cada mensaje con un bucle for. También se establece el tiempo de espera de invisibilidad en cinco minutos para todos los mensajes que devuelve este método.

```javascript
queueSvc.getMessages('myqueue', {numOfMessages: 15, visibilityTimeout: 5 * 60}, function(error, results, getResponse){
  if(!error){
    // Messages retrieved
    for(var index in result){
      // text is available in result[index].messageText
      var message = results[index];
      queueSvc.deleteMessage(queueName, message.messageId, message.popReceipt, function(error, deleteResponse){
        if(!error){
          // Message deleted
        }
      });
    }
  }
});
```

## <a name="how-to-get-the-queue-length"></a>Instrucciones: Obtención de la longitud de la cola
El método **getQueueMetadata** devuelve metadatos sobre la cola, junto con el número aproximado de mensajes que esperan en la cola.

```javascript
queueSvc.getQueueMetadata('myqueue', function(error, results, response){
  if(!error){
    // Queue length is available in results.approximateMessageCount
  }
});
```

## <a name="how-to-list-queues"></a>Instrucciones: Enumeración de las colas
Para recuperar una lista de colas, use **listQueuesSegmented**. Para recuperar una lista filtrada por un prefijo determinado, use **listQueuesSegmentedWithPrefix**.

```javascript
queueSvc.listQueuesSegmented(null, function(error, results, response){
  if(!error){
    // results.entries contains the list of queues
  }
});
```

Si no se pueden devolver todas las colas, `result.continuationToken` se puede usar como el primer parámetro de **listQueuesSegmented** o el segundo parámetro de **listQueuesSegmentedWithPrefix** para recuperar más resultados.

## <a name="how-to-delete-a-queue"></a>Instrucciones: Eliminación de una cola
Para eliminar una cola y todos los mensajes contenidos en ella, llame al método **deleteQueue** en el objeto de cola.

```javascript
queueSvc.deleteQueue(queueName, function(error, response){
  if(!error){
    // Queue has been deleted
  }
});
```

Para borrar todos los mensajes de una cola sin eliminarla, use **clearMessages**.

## <a name="how-to-work-with-shared-access-signatures"></a>Procedimientos: Trabajo con firmas de acceso compartido
Las firmas de acceso compartido (SAS) constituyen una manera segura de ofrecer acceso granular a las colas sin proporcionar el nombre o las claves de su cuenta de almacenamiento. Las SAS se usan con frecuencia para proporcionar acceso limitado a sus colas, por ejemplo, para permitir que una aplicación móvil envíe mensajes.

Una aplicación de confianza, como un servicio basado en la nube, genera una SAS mediante el valor de **generateSharedAccessSignature** del elemento **QueueService**, y lo proporciona a una aplicación en la que no se confía o en la que se confía parcialmente. Por ejemplo, una aplicación móvil. La SAS se genera usando una directiva que describe las fechas de inicio y de finalización durante las cuales la SAS es válida, junto con el nivel de acceso otorgado al titular de la SAS.

En el siguiente ejemplo se genera una nueva directiva de acceso compartido que permitirá al titular de la SAS agregar mensajes a la cola, y que expira 100 minutos después de la hora en que se crea.

```javascript
var startDate = new Date();
var expiryDate = new Date(startDate);
expiryDate.setMinutes(startDate.getMinutes() + 100);
startDate.setMinutes(startDate.getMinutes() - 100);

var sharedAccessPolicy = {
  AccessPolicy: {
    Permissions: azure.QueueUtilities.SharedAccessPermissions.ADD,
    Start: startDate,
    Expiry: expiryDate
  }
};

var queueSAS = queueSvc.generateSharedAccessSignature('myqueue', sharedAccessPolicy);
var host = queueSvc.host;
```

Tenga en cuenta que también se debe proporcionar la información del host, puesto que es necesaria cuando el titular de la SAS intenta acceder a la cola.

La aplicación cliente usa entonces la SAS con **QueueServiceWithSAS** para realizar operaciones contra la cola. En el siguiente ejemplo se realiza la conexión a la cola y se crea un mensaje.

```javascript
var sharedQueueService = azure.createQueueServiceWithSas(host, queueSAS);
sharedQueueService.createMessage('myqueue', 'Hello world from SAS!', function(error, result, response){
  if(!error){
    //message added
  }
});
```

Dado que la SAS se generó solo con acceso para agregar, si se realizara un intento para leer, actualizar o eliminar mensajes, se devolvería un error.

### <a name="access-control-lists"></a>Listas de control de acceso
Se puede usar una lista de control de acceso (ACL) para definir la directiva de acceso para una SAS. Esto es útil si desea permitir que varios clientes accedan a la cola, pero cada uno con directivas de acceso diferentes.

Una ACL se implementa mediante el uso de un conjunto de directivas de acceso, con un Id. asociado a cada directiva. En los siguientes ejemplos se definen dos directivas; una para "user1" y otra para "user2":

```javascript
var sharedAccessPolicy = {
  user1: {
    Permissions: azure.QueueUtilities.SharedAccessPermissions.PROCESS,
    Start: startDate,
    Expiry: expiryDate
  },
  user2: {
    Permissions: azure.QueueUtilities.SharedAccessPermissions.ADD,
    Start: startDate,
    Expiry: expiryDate
  }
};
```

En el siguiente ejemplo se obtiene la ACL actual para **myqueue** y luego se agregan las nuevas directivas mediante **setQueueAcl**. Este enfoque permite lo siguiente:

```javascript
var extend = require('extend');
queueSvc.getQueueAcl('myqueue', function(error, result, response) {
  if(!error){
    var newSignedIdentifiers = extend(true, result.signedIdentifiers, sharedAccessPolicy);
    queueSvc.setQueueAcl('myqueue', newSignedIdentifiers, function(error, result, response){
      if(!error){
        // ACL set
      }
    });
  }
});
```

Después de establecer una ACL, puede crear luego una SAS basada en el Id. de una directiva. En el siguiente ejemplo se crea una nueva SAS para 'user2':

```javascript
queueSAS = queueSvc.generateSharedAccessSignature('myqueue', { Id: 'user2' });
```

## <a name="next-steps"></a>Pasos siguientes
Ahora que está familiarizado con los aspectos básicos del almacenamiento de colas, utilice estos vínculos para obtener más información acerca de tareas de almacenamiento más complejas.

* Visite el [Blog del equipo de Azure Storage][Azure Storage Team Blog].
* Visite el repositorio del [SDK de Azure Storage para Node][Azure Storage SDK for Node] en GitHub.



[Azure Storage SDK for Node]: https://github.com/Azure/azure-storage-node

[using the REST API]: https://msdn.microsoft.com/library/azure/hh264518.aspx

[Azure Portal]: https://portal.azure.com

[Creación de una aplicación web de Node.js en Azure App Service](../../app-service/quickstart-nodejs.md)

[Creación e implementación de una aplicación Node.js en un servicio en la nube de Azure](../../cloud-services/cloud-services-nodejs-develop-deploy-app.md)

[Azure Storage Team Blog]: https://blogs.msdn.com/b/windowsazurestorage/

[Build and deploy a Node.js web app to Azure using Web Matrix]: https://www.microsoft.com/web/webmatrix/
