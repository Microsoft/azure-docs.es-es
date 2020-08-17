---
title: Uso de temas de Azure Service Bus con el paquete azure/service-bus de Node.js
description: Aprenda a usar los temas y suscripciones de Service Bus en Azure desde una aplicación Node.js con el paquete azure/service-bus.
author: spelluru
ms.devlang: nodejs
ms.topic: quickstart
ms.date: 06/23/2020
ms.author: spelluru
ms.custom: devx-track-javascript
ms.openlocfilehash: 9d4a3a66d967bd003534c7931091979d1667526c
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87432797"
---
# <a name="quickstart-how-to-use-service-bus-topics-and-subscriptions-with-nodejs-and-the-azure-sb-package"></a>Inicio rápido: Uso de los temas y las suscripciones de Service Bus con Node.js y el paquete azure-sb
En este tutorial, obtendrá información sobre cómo crear aplicaciones de Node.js para enviar mensajes a un tema de Service Bus y recibir mensajes de una suscripción de Service Bus mediante el paquete [azure-sb](https://www.npmjs.com/package/azure-sb). Los ejemplos están escritos en JavaScript y usan el [módulo Node.js de Azure](https://www.npmjs.com/package/azure), que internamente usa el paquete `azure-sb`.

> [!IMPORTANT]
> El paquete [azure-sb](https://www.npmjs.com/package/azure-sb) usa las [API de REST de Service Bus en tiempo de ejecución](/rest/api/servicebus/service-bus-runtime-rest). Puede obtener una experiencia más rápida con el nuevo paquete [@azure/service-bus](https://www.npmjs.com/package/@azure/service-bus) que usa el [protocolo AMQP 1.0](service-bus-amqp-overview.md) más rápido. 
> 
> Para obtener más información sobre el nuevo paquete, consulte [Uso de temas y suscripciones de Service Bus con Node.js y el paquete @azure/service-bus](https://docs.microsoft.com/azure/service-bus-messaging/service-bus-nodejs-how-to-use-topics-subscriptions-new-package); en caso contrario, siga leyendo para ver cómo se usa el paquete [azure](https://www.npmjs.com/package/azure).

Los escenarios que se explican aquí incluyen:

- Creación de temas y suscripciones 
- Creación de filtros de suscripción 
- Envío de mensajes a un tema 
- Recepción de mensajes de una suscripción
- Eliminación de temas y suscripciones 

Para más información sobre los temas y las suscripciones, consulte la sección [Pasos siguientes](#next-steps).

## <a name="prerequisites"></a>Requisitos previos
- Suscripción a Azure. Para completar este tutorial, deberá tener una cuenta de Azure. Puede activar sus [beneficios de suscriptor de Visual Studio o MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A85619ABF) o registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF).
- Siga los pasos de [Inicio rápido: Uso de Azure Portal para crear un tema de Service Bus y suscripciones a dicho tema](service-bus-quickstart-topics-subscriptions-portal.md) para crear un **espacio de nombres** de Service Bus y obtener la **cadena de conexión**.

    > [!NOTE]
    > En este inicio rápido, creará un **tema** y una **suscripción** al tema mediante **Node.js**. 

## <a name="create-a-nodejs-application"></a>Creación de una aplicación Node.js
Cree una aplicación Node.js vacía. Para obtener instrucciones sobre cómo crear una aplicación Node.js, vea [Creación e implementación de una aplicación Node.js en un sitio web de Azure], [Servicio en la nube Node.js][Node.js Cloud Service] (con Windows PowerShell) o Sitio web con WebMatrix.

## <a name="configure-your-application-to-use-service-bus"></a>Configuración de la aplicación para usar Service Bus
Para usar Service Bus, descargue el paquete Node.js de Azure. Este paquete incluye un conjunto de bibliotecas que se comunican con los servicios REST de Service Bus.

### <a name="use-node-package-manager-npm-to-obtain-the-package"></a>Uso del Administrador de paquetes para Node (NPM) para obtener el paquete
1. Abra una interfaz de la línea de comandos como **PowerShell** (Windows), **Terminal** (Mac) o **Bash** (Unix).
2. Vaya a la carpeta donde se creó la aplicación de ejemplo.
3. Escriba **npm install azure** en la ventana de comandos. Esto debería devolver la salida siguiente:

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
3. Puede ejecutar manualmente el comando **ls** para comprobar si se ha creado la carpeta **node\_modules**. Dentro de dicha carpeta, busque el paquete **azure**, que contiene las bibliotecas necesarias para acceder a los temas de Service Bus.

### <a name="import-the-module"></a>Importación del módulo
Utilizando el Bloc de notas u otro editor de texto, agregue el código siguiente en la parte superior del archivo **server.js** de la aplicación:

```javascript
var azure = require('azure');
```

### <a name="set-up-a-service-bus-connection"></a>Configuración de una conexión de Service Bus
El módulo de Azure lee la variable de entorno `AZURE_SERVICEBUS_CONNECTION_STRING` de la cadena de conexión que ha obtenido como parte de los [requisitos previos](#prerequisites). Si necesita instrucciones para volver a obtener la cadena de conexión, consulte [Obtención de la cadena de conexión](service-bus-quickstart-topics-subscriptions-portal.md#get-the-connection-string). Si no se configura esta variable de entorno, debe especificar la información de la cuenta al llamar a `createServiceBusService`.

Para ver un ejemplo de configuración de las variables de entorno para un servicio en la nube de Azure, consulte [Establecimiento de variables de entorno](../container-instances/container-instances-environment-variables.md#azure-cli-example).



## <a name="create-a-topic"></a>de un tema
El objeto **ServiceBusService** le permite trabajar con temas. El siguiente código crea un objeto **ServiceBusService**. Agréguelo cerca de la parte superior del archivo **server.js** , tras la instrucción para importar el módulo azure:

```javascript
var serviceBusService = azure.createServiceBusService();
```

Si llama a `createTopicIfNotExists` en el objeto **ServiceBusService**, se devuelve el tema especificado (si existe) o se crea uno nuevo con el nombre especificado. El código siguiente usa `createTopicIfNotExists` para crear un tema llamado `MyTopic` o conectarse a él:

```javascript
serviceBusService.createTopicIfNotExists('MyTopic',function(error){
    if(!error){
        // Topic was created or exists
        console.log('topic created or exists.');
    }
});
```

El método `createTopicIfNotExists` también admite opciones adicionales, lo que permite invalidar la configuración predeterminada de los temas, como el período de vida de los mensajes o el tamaño máximo de los temas. 

En el siguiente ejemplo se establece el tamaño máximo de los temas en 5 GB y el valor del período de vida en un minuto:

```javascript
var topicOptions = {
        MaxSizeInMegabytes: '5120',
        DefaultMessageTimeToLive: 'PT1M'
    };

serviceBusService.createTopicIfNotExists('MyTopic', topicOptions, function(error){
    if(!error){
        // topic was created or exists
    }
});
```

### <a name="filters"></a>Filtros
Las operaciones de filtrado opcionales pueden aplicarse a las tareas realizadas utilizando **ServiceBusService**. Las operaciones de filtrado pueden incluir registros, reintentos automáticos, etc. Los filtros son objetos que implementan un método con la firma:

```javascript
function handle (requestOptions, next)
```

Después de realizar el preprocesamiento en las opciones de solicitud, el método llama a `next` y pasa una devolución de llamada con la firma siguiente:

```javascript
function (returnObject, finalCallback, next)
```

En esta devolución de llamada, y después de procesar `returnObject` (la respuesta de la solicitud al servidor), la devolución de llamada debe invocar a next, si existe, para continuar procesando otros filtros, o invocar a `finalCallback` para finalizar la invocación del servicio.

Se incluyen dos filtros que implementan la lógica de reintento con el SDK de Azure para Node.js: **ExponentialRetryPolicyFilter** y **LinearRetryPolicyFilter**. Con el siguiente código se crea un objeto **ServiceBusService** que utiliza el filtro **ExponentialRetryPolicyFilter**:

```javascript
var retryOperations = new azure.ExponentialRetryPolicyFilter();
var serviceBusService = azure.createServiceBusService().withFilter(retryOperations);
```

## <a name="create-subscriptions"></a>Creación de suscripciones
Las suscripciones a temas también se crean con el objeto **ServiceBusService**. A las suscripciones se les asigna un nombre y pueden tener un filtro opcional que restrinja el conjunto de mensajes que se entregan a su cola virtual.

> [!NOTE]
> De forma predeterminada, las suscripciones son permanentes hasta que se eliminan, o hasta que se elimina el tema al que están asociadas. Si una aplicación contiene lógica para crear una suscripción, en primer lugar debe comprobar si esta existe, para lo que se utiliza el método `getSubscription`.
>
> Para que las suscripciones se eliminen automáticamente, establezca la [propiedad AutoDeleteOnIdle](https://docs.microsoft.com/javascript/api/@azure/arm-servicebus/sbsubscription?view=azure-node-latest#autodeleteonidle).

### <a name="create-a-subscription-with-the-default-matchall-filter"></a>Creación de una suscripción con el filtro predeterminado (MatchAll)
El filtro **MatchAll** es el filtro predeterminado utilizado cuando se crea una suscripción. Si se usa el filtro **MatchAll**, todos los mensajes publicados en el tema se colocan en la cola virtual de la suscripción. En el ejemplo siguiente se crea una suscripción llamada AllMessages que usa el filtro predeterminado **MatchAll**.

```javascript
serviceBusService.createSubscription('MyTopic','AllMessages',function(error){
    if(!error){
        // subscription created
    }
});
```

### <a name="create-subscriptions-with-filters"></a>Creación de suscripciones con filtros
También se pueden crear filtros que permitan especificar qué mensajes enviados a un tema deben aparecer dentro de una suscripción de tema determinada.

El tipo de filtro más flexible compatible con las suscripciones es **SqlFilter**, que implementa un subconjunto de SQL92. Los filtros de SQL operan en las propiedades de los mensajes que se publican en el tema. Para obtener más información sobre las expresiones que se pueden usar con un filtro de SQL, revise la sintaxis de [SqlFilter.SqlExpression][SqlFilter.SqlExpression].

Es posible agregar filtros a una suscripción mediante el método `createRule` del objeto **ServiceBusService**. Este método permite agregar nuevos filtros a una suscripción existente.

> [!NOTE]
> Debido a que el filtro predeterminado se aplica automáticamente a todas las suscripciones nuevas, debe eliminar primero el filtro predeterminado, si no quiere que **MatchAll** invalide los demás filtros que especifique. Puede eliminar la regla predeterminada utilizando el método `deleteRule` del objeto **ServiceBusService**.
>
>

En el ejemplo siguiente, se crea una suscripción denominada `HighMessages` con un objeto **SqlFilter** que solo selecciona los mensajes con una propiedad `messagenumber` personalizada cuyo valor es mayor que 3:

```javascript
serviceBusService.createSubscription('MyTopic', 'HighMessages', function (error){
    if(!error){
        // subscription created
        rule.create();
    }
});
var rule={
    deleteDefault: function(){
        serviceBusService.deleteRule('MyTopic',
            'HighMessages',
            azure.Constants.ServiceBusConstants.DEFAULT_RULE_NAME,
            rule.handleError);
    },
    create: function(){
        var ruleOptions = {
            sqlExpressionFilter: 'messagenumber > 3'
        };
        rule.deleteDefault();
        serviceBusService.createRule('MyTopic',
            'HighMessages',
            'HighMessageFilter',
            ruleOptions,
            rule.handleError);
    },
    handleError: function(error){
        if(error){
            console.log(error)
        }
    }
}
```

Del mismo modo, en el ejemplo que aparece a continuación, se crea una suscripción denominada `LowMessages` con un filtro **SqlFilter** que solo selecciona los mensajes cuyo valor de la propiedad `messagenumber` es menor o igual que 3:

```javascript
serviceBusService.createSubscription('MyTopic', 'LowMessages', function (error){
    if(!error){
        // subscription created
        rule.create();
    }
});
var rule={
    deleteDefault: function(){
        serviceBusService.deleteRule('MyTopic',
            'LowMessages',
            azure.Constants.ServiceBusConstants.DEFAULT_RULE_NAME,
            rule.handleError);
    },
    create: function(){
        var ruleOptions = {
            sqlExpressionFilter: 'messagenumber <= 3'
        };
        rule.deleteDefault();
        serviceBusService.createRule('MyTopic',
            'LowMessages',
            'LowMessageFilter',
            ruleOptions,
            rule.handleError);
    },
    handleError: function(error){
        if(error){
            console.log(error)
        }
    }
}
```

Ahora, cuando se envíe un mensaje a `MyTopic`, se entregará a los destinatarios que tengan la suscripción al tema `AllMessages`, mientras que se entrega de forma selectiva a los que tengan suscripciones a los temas `HighMessages` y `LowMessages` (según el contenido del mensaje).

## <a name="how-to-send-messages-to-a-topic"></a>Envío de mensajes a un tema
Para enviar un mensaje a un tema de Service Bus, la aplicación debe utilizar el método `sendTopicMessage` del objeto **ServiceBusService**.
Los mensajes enviados a los temas de Service Bus son objetos **BrokeredMessage**.
Los objetos **BrokeredMessage** cuentan con un conjunto de propiedades estándar (como `Label` y `TimeToLive`), un diccionario que se usa para mantener las propiedades personalizadas específicas de la aplicación y un conjunto de datos de cadenas. Una aplicación puede establecer el cuerpo del mensaje pasando un valor de cadena al método `sendTopicMessage`, con lo que las propiedades estándar requeridas adquieren valores predeterminados.

En el ejemplo siguiente se demuestra cómo enviar cinco mensajes de prueba a `MyTopic`. El valor de la propiedad `messagenumber` de cada mensaje varía en función de la iteración del bucle (esta propiedad determina qué suscripciones lo reciben):

```javascript
var message = {
    body: '',
    customProperties: {
        messagenumber: 0
    }
}

for (i = 0;i < 5;i++) {
    message.customProperties.messagenumber=i;
    message.body='This is Message #'+i;
    serviceBusService.sendTopicMessage(topic, message, function(error) {
      if (error) {
        console.log(error);
      }
    });
}
```

El tamaño máximo de mensaje que admiten los temas de Service Bus es de 256 KB en el [nivel Estándar](service-bus-premium-messaging.md) y de 1 MB en el [nivel Premium](service-bus-premium-messaging.md). El encabezado, que incluye propiedades de la aplicación estándar y personalizadas, puede tener un tamaño máximo de 64 KB. No hay límite para el número de mensajes que puede contener un tema, pero sí lo hay para el tamaño total de los mensajes de un tema. El tamaño de los temas se define en el momento de la creación (el límite máximo es de 5 GB).

## <a name="receive-messages-from-a-subscription"></a>Recepción de mensajes de una suscripción
Los mensajes se reciben de una suscripción utilizando el método `receiveSubscriptionMessage` del objeto **ServiceBusService**. De forma predeterminada, los mensajes se eliminan de la suscripción a medida que se leen. Sin embargo, puede establecer el parámetro opcional `isPeekLock` en **True** para leer y bloquear los mensajes sin eliminarlos de la suscripción.

El comportamiento predeterminado de lectura y eliminación del mensaje como parte de la operación de recepción es el modelo más sencillo y el que mejor funciona en aquellas situaciones en las que una aplicación puede tolerar que no se procese un mensaje en caso de error. Para entender este comportamiento, pongamos una situación en la que un consumidor emite la solicitud de recepción que se bloquea antes de procesarla. Como Service Bus ha marcado el mensaje como consumido, cuando la aplicación se reinicie y empiece a consumir mensajes de nuevo, habrá perdido el mensaje que se consumió antes del bloqueo.

Si el parámetro `isPeekLock` está establecido en **true**, el proceso de recepción se convierte en una operación en dos fases que permite admitir aplicaciones que no toleran mensajes perdidos. Cuando Service Bus recibe una solicitud, busca el siguiente mensaje que se va a consumir, lo bloquea para impedir que otros consumidores lo reciban y lo devuelve a la aplicación.
Una vez que la aplicación procesa el mensaje (o lo almacena de forma fiable para su futuro procesamiento), completa la segunda fase del proceso de recepción llamando al método **deleteMessage** y pasa el mensaje que se va a eliminar como parámetro. El método **deleteMessage** marca el mensaje como consumido y lo quita de la suscripción.

En el ejemplo siguiente se muestra cómo se pueden recibir y procesar mensajes mediante `receiveSubscriptionMessage`. En el ejemplo, primero se recibe un mensaje y se elimina de la suscripción "LowMessages" y, luego, se recibe un mensaje de la suscripción "HighMessages" con `isPeekLock` establecido en true. A continuación, se elimina el mensaje utilizando `deleteMessage`:

```javascript
serviceBusService.receiveSubscriptionMessage('MyTopic', 'LowMessages', function(error, receivedMessage){
    if(!error){
        // Message received and deleted
        console.log(receivedMessage);
    }
});
serviceBusService.receiveSubscriptionMessage('MyTopic', 'HighMessages', { isPeekLock: true }, function(error, lockedMessage){
    if(!error){
        // Message received and locked
        console.log(lockedMessage);
        serviceBusService.deleteMessage(lockedMessage, function (deleteError){
            if(!deleteError){
                // Message deleted
                console.log('message has been deleted.');
            }
        })
    }
});
```

## <a name="how-to-handle-application-crashes-and-unreadable-messages"></a>Actuación ante errores de la aplicación y mensajes que no se pueden leer
Service Bus proporciona una funcionalidad que le ayuda a superar sin problemas los errores de la aplicación o las dificultades para procesar un mensaje. Si por cualquier motivo una aplicación de recepción es incapaz de procesar el mensaje, entonces puede llamar al método `unlockMessage` del objeto **ServiceBusService**. Este método hace que Service Bus desbloquee el mensaje de la suscripción y lo ponga a disposición para que pueda volver a recibirse. En este caso, ya sea por la misma aplicación consumidora o por otra aplicación consumidora.

También hay un tiempo de espera asociado con un mensaje bloqueado dentro de la suscripción. Si la aplicación no puede procesar el mensaje antes de que finalice el tiempo de espera de bloqueo (por ejemplo, si la aplicación se bloquea), entonces Service Bus desbloquea el mensaje automáticamente y hace que esté disponible para que pueda volver a recibirse.

En caso de que la aplicación sufra un error después de procesar el mensaje y antes de llamar al método `deleteMessage`, el mensaje se vuelve a entregar a la aplicación cuando esta se reinicie. Este comportamiento se suele denominar *Al menos un procesamiento*. Es decir, cada mensaje se procesa al menos una vez, aunque en determinadas situaciones podría volver a entregarse el mismo mensaje. Si el escenario no puede tolerar el procesamiento duplicado, entonces debería agregar lógica a la aplicación para solucionar la entrega de mensajes duplicados. Puede usar la propiedad **MessageId** del mensaje, que permanece constante en todos los intentos de entrega.

## <a name="delete-topics-and-subscriptions"></a>Eliminación de temas y suscripciones
Los temas y las suscripciones son permanentes salvo que la [propiedad AutoDeleteOnIdle](https://docs.microsoft.com/javascript/api/@azure/arm-servicebus/sbsubscription?view=azure-node-latest#autodeleteonidle) esté establecida, por lo que deben eliminarse explícitamente a través de [Azure Portal][Azure portal] o mediante programación.
En el ejemplo siguiente se muestra cómo eliminar el tema denominado `MyTopic`:

```javascript
serviceBusService.deleteTopic('MyTopic', function (error) {
    if (error) {
        console.log(error);
    }
});
```

Al eliminar un tema también se eliminan todas las suscripciones que estén registradas con él. También se pueden eliminar las suscripciones de forma independiente. El ejemplo siguiente muestra cómo eliminar una suscripción denominada `HighMessages` del tema `MyTopic`:

```javascript
serviceBusService.deleteSubscription('MyTopic', 'HighMessages', function (error) {
    if(error) {
        console.log(error);
    }
});
```

> [!NOTE]
> Puede administrar los recursos de Service Bus con el [Explorador de Service Bus](https://github.com/paolosalvatori/ServiceBusExplorer/). El Explorador de Service Bus permite a los usuarios conectarse a un espacio de nombres de Service Bus y administrar las entidades de mensajería de una forma sencilla. La herramienta dispone de características avanzadas, como la funcionalidad de importación y exportación o la capacidad de probar el tema, las colas, las suscripciones, los servicios de retransmisión, los centros de notificaciones y los centros de eventos. 

## <a name="next-steps"></a>Pasos siguientes
Ahora que conoce los fundamentos de los temas de Service Bus, siga estos vínculos para obtener más información.

* Vea [Colas, temas y suscripciones][Queues, topics, and subscriptions].
* Referencia de API para [SqlFilter][SqlFilter].
* Visite el repositorio del [SDK de Azure para Node][Azure SDK for Node] en GitHub.

[Azure SDK for Node]: https://github.com/Azure/azure-sdk-for-node
[Azure portal]: https://portal.azure.com
[SqlFilter.SqlExpression]: service-bus-messaging-sql-filter.md
[Queues, topics, and subscriptions]: service-bus-queues-topics-subscriptions.md
[SqlFilter]: /javascript/api/@azure/arm-servicebus/sqlfilter?view=azure-node-latest
[Node.js Cloud Service]: ../cloud-services/cloud-services-nodejs-develop-deploy-app.md
[Creación e implementación de una aplicación Node.js en un sitio web de Azure]: ../app-service/app-service-web-get-started-nodejs.md
[Node.js Cloud Service with Storage]: ../cloud-services/cloud-services-nodejs-develop-deploy-app.md

