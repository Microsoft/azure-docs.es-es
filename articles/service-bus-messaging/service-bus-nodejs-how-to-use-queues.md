---
title: Uso de colas de Azure Service Bus en Node.js con el paquete azure-sb
description: Aprenderá a crear aplicaciones de Node.js para enviar mensajes a una cola de Service Bus, y recibirlos de ella, mediante el paquete azure-sb.
author: spelluru
ms.devlang: nodejs
ms.topic: quickstart
ms.date: 06/23/2020
ms.author: spelluru
ms.custom: seo-javascript-september2019, seo-javascript-october2019, devx-track-javascript
ms.openlocfilehash: ba0fd6cf7214beeaca9e93b1fe003a144c247cb5
ms.sourcegitcommit: 2ffa5bae1545c660d6f3b62f31c4efa69c1e957f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88077006"
---
# <a name="quickstart-use-service-bus-queues-in-azure-with-nodejs-and-the-azure-sb-package"></a>Inicio rápido: Uso de colas de Service Bus en Azure con Node.js y el paquete azure-sb
En este tutorial, obtendrá información sobre cómo crear aplicaciones de Node.js para enviar mensajes a una cola Service Bus y recibir mensajes de una cola de Azure Service Bus mediante el paquete [azure-sb](https://www.npmjs.com/package/azure-sb). Los ejemplos están escritos en JavaScript y usan el [módulo Node.js de Azure](https://www.npmjs.com/package/azure), que internamente usa el paquete azure-sb.

> [!IMPORTANT]
> El paquete [azure-sb](https://www.npmjs.com/package/azure-sb) usa las [API de REST de Service Bus en tiempo de ejecución](/rest/api/servicebus/service-bus-runtime-rest). Puede obtener una experiencia más rápida con el nuevo paquete [@azure/service-bus](https://www.npmjs.com/package/@azure/service-bus) que usa el [protocolo AMQP 1.0](service-bus-amqp-overview.md) más rápido. 
> 
> Para obtener más información sobre el nuevo paquete, consulte [Uso de colas con Service Bus con Node.js y el paquete @azure/service-bus](./service-bus-nodejs-how-to-use-queues-new-package.md); en caso contrario, siga leyendo para ver cómo se usa el paquete [azure](https://www.npmjs.com/package/azure).

## <a name="prerequisites"></a>Prerrequisitos
- Suscripción a Azure. Para completar este tutorial, deberá tener una cuenta de Azure. Puede activar sus [ventajas de suscriptor a MSDN](https://azure.microsoft.com/pricing/member-offers/credit-for-visual-studio-subscribers/?WT.mc_id=A85619ABF) o registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF).
- Si no tiene una cola con la que trabajar, siga los pasos del artículo [Uso de Azure Portal para crear una cola de Service Bus](service-bus-quickstart-portal.md) para crear una.
    1. Lea la **introducción** rápida de las **colas** de Service Bus. 
    2. Cree un **espacio de nombres** de Service Bus. 
    3. Obtenga la **cadena de conexión**. 

        > [!NOTE]
        > Creará una **cola** en el espacio de nombres de Service Bus con Node.js en este tutorial. 
 

## <a name="create-a-nodejs-application"></a>Creación de una aplicación Node.js
Cree una aplicación Node.js vacía. Para obtener instrucciones acerca de cómo crear una aplicación Node.js, consulte [Creación e implementación de una aplicación Node.js en un sitio web de Azure][Create and deploy a Node.js application to an Azure Website] o [Servicio en la nube Node.js][Node.js Cloud Service] (con Windows PowerShell).

## <a name="configure-your-application-to-use-service-bus"></a>Configuración de la aplicación para usar Service Bus
Para utilizar Azure Service Bus, descargue y use el paquete Azure para Node.js. Este paquete incluye un conjunto de bibliotecas que se comunican con los servicios REST de Service Bus.

### <a name="use-node-package-manager-npm-to-obtain-the-package"></a>Uso del Administrador de paquetes para Node (NPM) para obtener el paquete
1. Use la ventana de comandos de **Windows PowerShell for Node.js** para navegar a la carpeta **c:\\node\\sbqueues\\WebRole1** en la que ha creado la aplicación de ejemplo.
2. Escriba **npm install azure** en la ventana de comandos, cuyo resultado debe ser similar al siguiente ejemplo:

    ```
    azure@0.7.5 node_modules\azure
        ├── dateformat@1.0.2-1.2.3
        ├── xmlbuilder@0.4.2
        ├── node-uuid@1.2.0
        ├── mime@1.2.9
        ├── underscore@1.4.4
        ├── validator@1.1.1
        ├── tunnel@0.0.2
        ├── wns@0.5.3
        ├── xml2js@0.2.7 (sax@0.5.2)
        └── request@2.21.0 (json-stringify-safe@4.0.0, forever-agent@0.5.0, aws-sign@0.3.0, tunnel-agent@0.3.0, oauth-sign@0.3.0, qs@0.6.5, cookie-jar@0.3.0, node-uuid@1.4.0, http-signature@0.9.11, form-data@0.0.8, hawk@0.13.1)
    ```
3. Puede ejecutar manualmente el comando **ls** para comprobar si se ha creado una carpeta **node_modules**. Dentro de esa carpeta, busque el paquete **azure**, que contiene las bibliotecas necesarias para obtener acceso a las colas de Service Bus.

### <a name="import-the-module"></a>Importación del módulo
Utilizando el Bloc de notas u otro editor de texto, agregue el código siguiente en la parte superior del archivo **server.js** de la aplicación:

```javascript
var azure = require('azure');
```

### <a name="set-up-an-azure-service-bus-connection"></a>Configuración de una conexión de Azure Service Bus
El módulo de Azure lee las variables de entorno `AZURE_SERVICEBUS_CONNECTION_STRING` para obtener la información necesaria para conectarse a Service Bus. Si no se configura esta variable de entorno, debe especificar la información de la cuenta al llamar a `createServiceBusService`.

Para ver un ejemplo de configuración de las variables de entorno en [Azure Portal][Azure portal] para un sitio web de Azure, consulte [Aplicación web Node.js con Storage][Node.js Web Application with Storage].

## <a name="create-a-queue"></a>Creación de una cola
El objeto **ServiceBusService** le permite trabajar con colas de Service Bus. El siguiente código crea un objeto **ServiceBusService**. Agréguelo cerca de la parte superior del archivo **server.js** , tras la instrucción para importar el módulo azure:

```javascript
var serviceBusService = azure.createServiceBusService();
```

Al llamar a `createQueueIfNotExists` en el objeto **ServiceBusService**, se obtiene la cola especificada (si existe) o se crea una nueva con el nombre especificado. El código siguiente usa `createQueueIfNotExists` para crear una cola llamada `myqueue` o conectarse a ella:

```javascript
serviceBusService.createQueueIfNotExists('myqueue', function(error){
    if(!error){
        // Queue exists
    }
});
```

El método `createServiceBusService` también admite opciones adicionales, que le permiten invalidar la configuración predeterminada de las colas, como el período de vida de los mensajes o el tamaño máximo de las colas. En el siguiente ejemplo se establece el tamaño máximo de las colas en 5 GB y el valor del período de vida (TTL) en 1 minuto:

```javascript
var queueOptions = {
      MaxSizeInMegabytes: '5120',
      DefaultMessageTimeToLive: 'PT1M'
    };

serviceBusService.createQueueIfNotExists('myqueue', queueOptions, function(error){
    if(!error){
        // Queue exists
    }
});
```

### <a name="filters"></a>Filtros
Las operaciones de filtrado opcionales pueden aplicarse a las tareas realizadas utilizando **ServiceBusService**. Las operaciones de filtrado pueden incluir registros, reintentos automáticos, etc. Los filtros son objetos que implementan un método con la firma:

```javascript
function handle (requestOptions, next)
```

Después de realizar el preprocesamiento en las opciones de solicitud, el método tiene que llamar a `next`, pasando una devolución de llamada con la firma siguiente:

```javascript
function (returnObject, finalCallback, next)
```

En esta devolución de llamada y después de procesar `returnObject` (la respuesta de la solicitud al servidor), la devolución de llamada tiene que invocar `next`, si existe, para continuar procesando otros filtros, o bien invocar `finalCallback`, que finaliza la invocación del servicio.

Con el SDK de Azure, se incluyen dos filtros que implementan la lógica de reintento para Node.js, `ExponentialRetryPolicyFilter` y `LinearRetryPolicyFilter`. El siguiente código crea un objeto `ServiceBusService` que usa `ExponentialRetryPolicyFilter`:

```javascript
var retryOperations = new azure.ExponentialRetryPolicyFilter();
var serviceBusService = azure.createServiceBusService().withFilter(retryOperations);
```

## <a name="send-messages-to-a-queue"></a>mensajes a una cola
Para enviar un mensaje a una cola de Service Bus, la aplicación debe llamar al método `sendQueueMessage` del objeto **ServiceBusService**. Los mensajes enviados a las colas de Service Bus (y recibidos de ellas) son objetos **BrokeredMessage** y cuentan con un conjunto de propiedades estándar (como **Label** y **TimeToLive**), un diccionario que se usa para mantener las propiedades personalizadas específicas de la aplicación y un conjunto de datos arbitrarios de aplicaciones. Una aplicación puede establecer el cuerpo del mensaje pasando una cadena como el mensaje. Las propiedades estándar requeridas se rellenan con valores predeterminados.

En el ejemplo siguiente se muestra cómo enviar un mensaje de prueba a la cola `myqueue` mediante `sendQueueMessage`:

```javascript
var message = {
    body: 'Test message',
    customProperties: {
        testproperty: 'TestValue'
    }};
serviceBusService.sendQueueMessage('myqueue', message, function(error){
    if(!error){
        // message sent
    }
});
```

El tamaño máximo de mensaje que admiten las colas de Service Bus es de 256 KB en el [nivel Estándar](service-bus-premium-messaging.md) y de 1 MB en el [nivel Premium](service-bus-premium-messaging.md). El encabezado, que incluye propiedades de la aplicación estándar y personalizadas, puede tener un tamaño máximo de 64 KB. No hay límite para el número de mensajes que contiene una cola, pero hay un tope para el tamaño total de los mensajes contenidos en una cola. El tamaño de la cola se define en el momento de la creación, con un límite de 5 GB. Para obtener más información sobre las cuotas, consulte [Cuotas de Service Bus][Service Bus quotas].

## <a name="receive-messages-from-a-queue"></a>mensajes de una cola
Los mensajes se reciben de una cola utilizando el método `receiveQueueMessage` del objeto **ServiceBusService**. De manera predeterminada, los mensajes se eliminan de la cola una vez que se leen; sin embargo, puede leer (echar un vistazo) y bloquear los mensajes sin eliminarlos de la cola estableciendo el parámetro opcional `isPeekLock` en **true**.

El comportamiento predeterminado de lectura y eliminación del mensaje como parte de la operación de recepción es el modelo más sencillo y el que mejor funciona en aquellas situaciones en las que una aplicación puede tolerar que no se procese un mensaje en caso de error. Para entender este comportamiento, pongamos una situación en la que un consumidor emite la solicitud de recepción que se bloquea antes de procesarla. Como Service Bus habrá marcado el mensaje como consumido, cuando la aplicación se reinicie y empiece a consumir mensajes de nuevo, habrá perdido el mensaje que se consumió antes del bloqueo.

Si el parámetro `isPeekLock` está establecido en **true**, el proceso de recepción se convierte en una operación en dos fases que permite admitir aplicaciones que no toleran la pérdida de mensajes. Cuando Service Bus recibe una solicitud, busca el siguiente mensaje que se va a consumir, lo bloquea para impedir que otros consumidores lo reciban y, a continuación, lo devuelve a la aplicación. Una vez que la aplicación termina de procesar el mensaje (o lo almacena de forma fiable para su futuro procesamiento), completa la segunda fase del proceso de recepción llamando al método `deleteMessage` y facilitando el mensaje que se va a eliminar a modo de parámetro. El método `deleteMessage` marca el mensaje como consumido y lo elimina de la cola.

En el ejemplo siguiente se muestra cómo recibir y procesar mensajes mediante `receiveQueueMessage`. En primer lugar, el ejemplo recibe y elimina un mensaje, después recibe un mensaje con `isPeekLock` establecido en **true** y luego lo elimina mediante `deleteMessage`:

```javascript
serviceBusService.receiveQueueMessage('myqueue', function(error, receivedMessage){
    if(!error){
        // Message received and deleted
    }
});
serviceBusService.receiveQueueMessage('myqueue', { isPeekLock: true }, function(error, lockedMessage){
    if(!error){
        // Message received and locked
        serviceBusService.deleteMessage(lockedMessage, function (deleteError){
            if(!deleteError){
                // Message deleted
            }
        });
    }
});
```

## <a name="how-to-handle-application-crashes-and-unreadable-messages"></a>Actuación ante errores de la aplicación y mensajes que no se pueden leer
Service Bus proporciona una funcionalidad que le ayuda a superar sin problemas los errores de la aplicación o las dificultades para procesar un mensaje. Si por cualquier motivo una aplicación de recepción es incapaz de procesar el mensaje, entonces puede llamar al método `unlockMessage` del objeto **ServiceBusService**. Esto hará que Service Bus desbloquee el mensaje de la cola y que esté disponible para que pueda volver a recibirse, ya sea por la misma aplicación que lo consume o por otra.

También hay un tiempo de expiración asociado con un mensaje bloqueado en la cola y, si la aplicación no puede procesar el mensaje antes de que finalice el tiempo de expiración de bloqueo (por ejemplo, si la aplicación sufre un bloqueo), entonces Service Bus desbloqueará el mensaje automáticamente y hará que esté disponible para que pueda volver a recibirse.

En caso de que la aplicación sufra un error después de procesar el mensaje y antes de llamar al método `deleteMessage`, entonces el mensaje se volverá a entregar a la aplicación cuando esta se reinicie. Habitualmente, este enfoque se denomina *Al menos un procesamiento*, es decir, cada mensaje se procesará al menos una vez; aunque en determinadas situaciones podría volver a entregarse el mismo mensaje. Si el escenario no puede tolerar el procesamiento duplicado, entonces los desarrolladores de la aplicación deberían agregar lógica adicional a su aplicación para solucionar la entrega de mensajes duplicados. A menudo, esto se consigue usando la propiedad **MessageId** del mensaje, que permanecerá constante en todos los intentos de entrega.

> [!NOTE]
> Puede administrar los recursos de Service Bus con el [Explorador de Service Bus](https://github.com/paolosalvatori/ServiceBusExplorer/). El Explorador de Service Bus permite a los usuarios conectarse a un espacio de nombres de Service Bus y administrar las entidades de mensajería de una forma sencilla. La herramienta dispone de características avanzadas, como la funcionalidad de importación y exportación o la capacidad de probar el tema, las colas, las suscripciones, los servicios de retransmisión, los centros de notificaciones y los centros de eventos. 

## <a name="next-steps"></a>Pasos siguientes
Para más información sobre las colas, consulte los siguientes recursos:

* [Colas, temas y suscripciones][Queues, topics, and subscriptions]
* [Repositorio de Azure SDK][Azure SDK for Node] para Node en GitHub
* [Centro para desarrolladores de Node.js](https://azure.microsoft.com/develop/nodejs/)

[Azure SDK for Node]: https://github.com/Azure/azure-sdk-for-node
[Azure portal]: https://portal.azure.com

[Node.js Cloud Service]: ../cloud-services/cloud-services-nodejs-develop-deploy-app.md
[Queues, topics, and subscriptions]: service-bus-queues-topics-subscriptions.md
[Create and deploy a Node.js application to an Azure Website]: ../app-service/quickstart-nodejs.md
[Node.js Web Application with Storage]:../cosmos-db/table-storage-how-to-use-nodejs.md
[Service Bus quotas]: service-bus-quotas.md