---
title: Uso de temas de Azure Service Bus con PHP
description: En este tutorial, va a aprender a usar los temas de Azure Service Bus y las suscripciones de una aplicación PHP.
ms.devlang: PHP
ms.topic: quickstart
ms.date: 06/23/2020
ms.openlocfilehash: 706f523fdfb3c710bb16b048cfc68ce98875adb1
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88066209"
---
# <a name="quickstart-how-to-use-service-bus-topics-and-subscriptions-with-php"></a>Inicio rápido: Uso de temas y suscripciones de Service Bus con PHP

[!INCLUDE [service-bus-selector-topics](../../includes/service-bus-selector-topics.md)]

En este artículo se muestra cómo usar los temas y las suscripciones de Service Bus. Los ejemplos están escritos en PHP y utilizan el [SDK de Azure para PHP](https://github.com/Azure/azure-sdk-for-php). Los escenarios descritos incluyen:

- Creación de temas y suscripciones 
- Creación de filtros de suscripción 
- Envío de mensajes a un tema 
- Recepción de mensajes de una suscripción
- Eliminación de temas y suscripciones

## <a name="prerequisites"></a>Prerrequisitos
1. Suscripción a Azure. Para completar este tutorial, deberá tener una cuenta de Azure. Puede activar sus [beneficios de suscriptor de Visual Studio o MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A85619ABF) o registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF).
2. Siga los pasos de [Inicio rápido: Uso de Azure Portal para crear un tema de Service Bus y suscripciones a dicho tema](service-bus-quickstart-topics-subscriptions-portal.md) para crear un **espacio de nombres** de Service Bus y obtener la **cadena de conexión**.

    > [!NOTE]
    > En este inicio rápido, creará un **tema** y una **suscripción** al tema mediante **PHP**. 

## <a name="create-a-php-application"></a>Creación de una aplicación PHP
El único requisito a la hora de crear una aplicación PHP para obtener acceso a Azure Blob service es que se haga referencia a clases de [Azure SDK para PHP](https://github.com/Azure/azure-sdk-for-php) dentro del código. Puede utilizar cualquier herramienta de desarrollo para crear la aplicación, o bien el Bloc de notas.

> [!NOTE]
> La instalación de PHP debe tener también la [extensión OpenSSL](https://php.net/openssl) instalada y habilitada.
> 
> 

En este artículo se describe cómo se usan las características de servicio a las que se puede llamar desde una aplicación PHP localmente o bien mediante código a través de un rol web, un rol de trabajo o un sitio web de Azure.

## <a name="get-the-azure-client-libraries"></a>Obtención de las bibliotecas de clientes de Azure

### <a name="install-via-composer"></a>Instalación mediante el compositor
1. Cree un archivo con el nombre **composer.json** en la raíz del proyecto y agréguele el código siguiente:
   
    ```json
    {
      "require": {
        "microsoft/windowsazure": "*"
      }
    }
    ```
2. Descargue **[composer.phar][composer-phar]** en la raíz del proyecto.
3. Abra un símbolo del sistema y ejecute el siguiente comando en la raíz del proyecto
   
    ```
    php composer.phar install
    ```

## <a name="configure-your-application-to-use-service-bus"></a>Configuración de la aplicación para usar Service Bus
Para usar las API de Service Bus:

1. Haga referencia al archivo cargador automático mediante la instrucción [require_once][require-once].
2. Hacer referencia a todas las clases que utilice.

En el siguiente ejemplo se muestra cómo incluir el archivo autocargador y hacer referencia a la clase **ServiceBusService**.

> [!NOTE]
> En este ejemplo (así como en otros ejemplos de este artículo), se asume que ha instalado las bibliotecas de clientes PHP para Azure mediante el compositor. Si las instaló las bibliotecas manualmente o como un paquete PEAR, debe hacer referencia al archivo autocargador **WindowsAzure.php**.
> 
> 

```php
require_once 'vendor/autoload.php';
use WindowsAzure\Common\ServicesBuilder;
```

En los ejemplos siguientes, la instrucción `require_once` aparece siempre, pero solo se hará referencia a las clases necesarias para la ejecución del ejemplo.

## <a name="set-up-a-service-bus-connection"></a>Configuración de una conexión de Service Bus
Para crear una instancia de un cliente de Service Bus, primero debe disponer de una cadena de conexión válida con el siguiente formato:

```
Endpoint=[yourEndpoint];SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=[Primary Key]
```

Donde `Endpoint` tiene generalmente el formato `https://[yourNamespace].servicebus.windows.net`.

Para crear cualquier cliente de servicio de Azure, debe usar la clase `ServicesBuilder`. Puede:

* pasarle directamente la cadena de conexión, o bien
* usar **CloudConfigurationManager (CCM)** para buscar la cadena de conexión en varios orígenes externos:
  * De manera predeterminada, admite un origen externo: variables de entorno.
  * Puede agregar nuevos orígenes extendiendo la clase `ConnectionStringSource`.

En los ejemplos descritos aquí, la cadena de conexión se pasa directamente.

```php
require_once 'vendor/autoload.php';

use WindowsAzure\Common\ServicesBuilder;

$connectionString = "Endpoint=[yourEndpoint];SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=[Primary Key]";

$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);
```

## <a name="create-a-topic"></a>de un tema
Puede realizar operaciones de administración para temas de Service Bus mediante la clase `ServiceBusRestProxy`. Un objeto `ServiceBusRestProxy` se construye mediante el método de generador `ServicesBuilder::createServiceBusService` con una cadena de conexión adecuada que encapsula los permisos de token para administrarlo.

En el ejemplo siguiente se muestra cómo crear una instancia de un objeto `ServiceBusRestProxy` y llamar a `ServiceBusRestProxy->createTopic` para crear un tema denominado `mytopic` dentro de un espacio de nombres `MySBNamespace`:

```php
require_once 'vendor/autoload.php';

use WindowsAzure\Common\ServicesBuilder;
use WindowsAzure\Common\ServiceException;
use WindowsAzure\ServiceBus\Models\TopicInfo;

// Create Service Bus REST proxy.
$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

try {
    // Create topic.
    $topicInfo = new TopicInfo("mytopic");
    $serviceBusRestProxy->createTopic($topicInfo);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here: 
    // https://docs.microsoft.com/rest/api/storageservices/Common-REST-API-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

> [!NOTE]
> Puede usar el método `listTopics` en los objetos `ServiceBusRestProxy` para comprobar si ya existe un tema con un nombre especificado en un espacio de nombres de servicio.
> 
> 

## <a name="create-a-subscription"></a>una suscripción
También se crean suscripciones de temas con el método `ServiceBusRestProxy->createSubscription`. A las suscripciones se les puede asignar un nombre y pueden tener un filtro opcional que restrinja el conjunto de mensajes que pasan a la cola virtual de la suscripción.

### <a name="create-a-subscription-with-the-default-matchall-filter"></a>Creación de una suscripción con el filtro predeterminado (MatchAll)
Se usa el filtro predeterminado **MatchAll** en caso de que no se haya especificado ninguno al crear una nueva suscripción. Al usar el filtro **MatchAll**, todos los mensajes publicados en el tema se colocan en la cola virtual de la suscripción. En el ejemplo siguiente se crea una suscripción llamada `mysubscription` que usa el filtro predeterminado **MatchAll**.

```php
require_once 'vendor/autoload.php';

use WindowsAzure\Common\ServicesBuilder;
use WindowsAzure\Common\ServiceException;
use WindowsAzure\ServiceBus\Models\SubscriptionInfo;

// Create Service Bus REST proxy.
$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

try {
    // Create subscription.
    $subscriptionInfo = new SubscriptionInfo("mysubscription");
    $serviceBusRestProxy->createSubscription("mytopic", $subscriptionInfo);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here: 
    // https://docs.microsoft.com/rest/api/storageservices/Common-REST-API-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

### <a name="create-subscriptions-with-filters"></a>Creación de suscripciones con filtros
También puede configurar filtros que le permitan especificar qué mensajes enviados a un tema deben aparecer dentro de una suscripción a un tema determinado. El tipo de filtro más flexible compatible con las suscripciones es [SqlFilter](/dotnet/api/microsoft.servicebus.messaging.sqlfilter), que implementa un subconjunto de SQL92. Los filtros de SQL operan en las propiedades de los mensajes que se publican en el tema. Para más información sobre SqlFilters, consulte la [propiedad SqlFilter.SqlExpression][sqlfilter].

> [!NOTE]
> Cada una de las reglas de una suscripción procesa los mensajes entrantes de forma independiente, lo que agrega sus mensajes resultantes a la suscripción. Además, cada nueva suscripción dispone de objeto **Rule** predeterminado con un filtro que agrega todos los mensajes del tema a la suscripción. Para recibir solo mensajes que coincidan con su filtro, debe quitar la regla predeterminada. Puede eliminar la regla predeterminada mediante el método `ServiceBusRestProxy->deleteRule`.
> 
> 

En el ejemplo siguiente, se crea una suscripción denominada `HighMessages` con un objeto **SqlFilter** que solo selecciona los mensajes con una propiedad `MessageNumber` personalizada cuyo valor es mayor que 3: Consulte [Envío de mensajes a un tema](#send-messages-to-a-topic) para más información sobre cómo agregar propiedades personalizadas a los mensajes.

```php
$subscriptionInfo = new SubscriptionInfo("HighMessages");
$serviceBusRestProxy->createSubscription("mytopic", $subscriptionInfo);

$serviceBusRestProxy->deleteRule("mytopic", "HighMessages", '$Default');

$ruleInfo = new RuleInfo("HighMessagesRule");
$ruleInfo->withSqlFilter("MessageNumber > 3");
$ruleResult = $serviceBusRestProxy->createRule("mytopic", "HighMessages", $ruleInfo);
```

Para este código hay que usar un espacio de nombres adicional: `WindowsAzure\ServiceBus\Models\SubscriptionInfo`.

Del mismo modo, en el ejemplo que aparece a continuación, se crea una suscripción denominada `LowMessages` con un filtro `SqlFilter` que solo selecciona los mensajes cuyo valor de la propiedad `MessageNumber` sea menor o igual a 3:

```php
$subscriptionInfo = new SubscriptionInfo("LowMessages");
$serviceBusRestProxy->createSubscription("mytopic", $subscriptionInfo);

$serviceBusRestProxy->deleteRule("mytopic", "LowMessages", '$Default');

$ruleInfo = new RuleInfo("LowMessagesRule");
$ruleInfo->withSqlFilter("MessageNumber <= 3");
$ruleResult = $serviceBusRestProxy->createRule("mytopic", "LowMessages", $ruleInfo);
```

Cuando se envía un mensaje al tema `mytopic`, siempre se entrega a los destinatarios que tengan la suscripción a `mysubscription`, mientras que se entrega de forma selectiva a los que tengan suscripciones a `HighMessages` y `LowMessages` (según el contenido del mensaje).

## <a name="send-messages-to-a-topic"></a>Envío de mensajes a un tema
Para enviar un mensaje a un tema de Service Bus, su aplicación llama al método `ServiceBusRestProxy->sendTopicMessage`. El código siguiente muestra cómo enviar un mensaje al tema `mytopic` que se ha creado anteriormente en el espacio de nombres del servicio `MySBNamespace`.

```php
require_once 'vendor/autoload.php';

use WindowsAzure\Common\ServicesBuilder;
use WindowsAzure\Common\ServiceException;
use WindowsAzure\ServiceBus\Models\BrokeredMessage;

// Create Service Bus REST proxy.
$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

try {
    // Create message.
    $message = new BrokeredMessage();
    $message->setBody("my message");

    // Send message.
    $serviceBusRestProxy->sendTopicMessage("mytopic", $message);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here: 
    // https://docs.microsoft.com/rest/api/storageservices/Common-REST-API-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

Los mensajes enviados a los temas de Service Bus son instancias de la clase [BrokeredMessage][BrokeredMessage]. Los objetos [BrokeredMessage][BrokeredMessage] tienen un conjunto de propiedades y métodos estándar, así como propiedades que se pueden usar para contener propiedades personalizadas específicas de la aplicación. En el ejemplo siguiente se muestra cómo enviar cinco mensajes de prueba al tema `mytopic` creado anteriormente. El método `setProperty` se usa para agregar una propiedad personalizada (`MessageNumber`) a cada mensaje. El valor de la propiedad `MessageNumber` varía en cada mensaje (este valor se puede usar para determinar qué suscripciones lo reciben, como se muestra en la sección [Creación de una suscripción](#create-a-subscription)):

```php
for($i = 0; $i < 5; $i++){
    // Create message.
    $message = new BrokeredMessage();
    $message->setBody("my message ".$i);

    // Set custom property.
    $message->setProperty("MessageNumber", $i);

    // Send message.
    $serviceBusRestProxy->sendTopicMessage("mytopic", $message);
}
```

El tamaño máximo de mensaje que admiten los temas de Service Bus es de 256 KB en el [nivel Estándar](service-bus-premium-messaging.md) y de 1 MB en el [nivel Premium](service-bus-premium-messaging.md). El encabezado, que incluye propiedades de la aplicación estándar y personalizadas, puede tener un tamaño máximo de 64 KB. No hay límite para el número de mensajes que contiene un tema, pero hay un tope para el tamaño total de los mensajes contenidos en un tema. El límite superior para el tamaño del tema es de 5 GB. Para obtener más información sobre las cuotas, consulte [Cuotas de Service Bus][Service Bus quotas].

## <a name="receive-messages-from-a-subscription"></a>Recepción de mensajes de una suscripción
La mejor manera de recibir mensajes de una suscripción es usar un método `ServiceBusRestProxy->receiveSubscriptionMessage`. Los mensajes se pueden recibir de dos modos distintos: [*ReceiveAndDelete* y *PeekLock*](/dotnet/api/microsoft.servicebus.messaging.receivemode). **PeekLock** es el valor predeterminado.

Al usar el modo [ReceiveAndDelete](/dotnet/api/microsoft.servicebus.messaging.receivemode), la operación de recepción consta de una sola fase; es decir, cuando Service Bus recibe una solicitud de lectura de un mensaje de una suscripción, marca el mensaje como consumido y lo devuelve a la aplicación. El modo [ReceiveAndDelete](/dotnet/api/microsoft.servicebus.messaging.receivemode)* es el modelo más sencillo y funciona mejor para los escenarios en los que una aplicación puede tolerar no procesar un mensaje en caso de error. Para entenderlo mejor, pongamos una situación en la que un consumidor emite la solicitud de recepción que se bloquea antes de procesarla. Como Service Bus ha marcado el mensaje como consumido, cuando la aplicación se reinicie y empiece a consumir mensajes de nuevo, habrá perdido el mensaje que se consumió antes del bloqueo.

En el modo [PeekLock](/dotnet/api/microsoft.servicebus.messaging.receivemode), la recepción de un mensaje se convierte en una operación de dos etapas, lo que hace posible admitir aplicaciones que no pueden tolerar la pérdida de mensajes. Cuando Service Bus recibe una solicitud, busca el siguiente mensaje que se va a consumir, lo bloquea para impedir que otros consumidores lo reciban y, a continuación, lo devuelve a la aplicación. Una vez que la aplicación termina de procesar el mensaje (o lo almacena de forma confiable para su futuro procesamiento), completa la segunda fase del proceso de recepción mediante la realización de una llamada a `ServiceBusRestProxy->deleteMessage` en el mensaje recibido. Cuando Service Bus ve la llamada `deleteMessage`, marca el mensaje como consumido y lo elimina de la cola.

En el ejemplo que aparece a continuación, se indica cómo se puede recibir y procesar un mensaje usando el modo [PeekLock](/dotnet/api/microsoft.servicebus.messaging.receivemode) (el modo predeterminado). 

```php
require_once 'vendor/autoload.php';

use WindowsAzure\Common\ServicesBuilder;
use WindowsAzure\Common\ServiceException;
use WindowsAzure\ServiceBus\Models\ReceiveMessageOptions;

// Create Service Bus REST proxy.
$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

try {
    // Set receive mode to PeekLock (default is ReceiveAndDelete)
    $options = new ReceiveMessageOptions();
    $options->setPeekLock();

    // Get message.
    $message = $serviceBusRestProxy->receiveSubscriptionMessage("mytopic", "mysubscription", $options);

    echo "Body: ".$message->getBody()."<br />";
    echo "MessageID: ".$message->getMessageId()."<br />";

    /*---------------------------
        Process message here.
    ----------------------------*/

    // Delete message. Not necessary if peek lock is not set.
    echo "Deleting message...<br />";
    $serviceBusRestProxy->deleteMessage($message);
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here:
    // https://docs.microsoft.com/rest/api/storageservices/Common-REST-API-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

## <a name="how-to-handle-application-crashes-and-unreadable-messages"></a>Actuación ante errores de la aplicación y mensajes que no se pueden leer
Service Bus proporciona una funcionalidad que le ayuda a superar sin problemas los errores de la aplicación o las dificultades para procesar un mensaje. Si por cualquier motivo una aplicación de recepción no puede procesar el mensaje, puede llamar al método `unlockMessage` en el mensaje recibido (en lugar de al método `deleteMessage`). Esto hará que Service Bus desbloquee el mensaje de la cola y esté disponible para volver a recibirse, ya sea por la misma aplicación que lo consume o por otra.

También hay otro tiempo de expiración asociado con un mensaje bloqueado en la cola y, si la aplicación no puede procesar el mensaje antes de que finalice el tiempo de expiración del bloqueo (por ejemplo, si la aplicación se bloquea), entonces Service Bus desbloquea el mensaje automáticamente y hace que esté disponible para que pueda volver a recibirse.

En caso de que la aplicación se bloquee después de procesar el mensaje, pero antes de realizar la solicitud `deleteMessage`, el mensaje se vuelve a entregar a la aplicación cuando esta se reinicie. Este proceso habitualmente se denomina *Al menos un procesamiento*, es decir, cada mensaje se procesará al menos una vez; aunque en determinadas situaciones, podría volver a entregarse el mismo mensaje. Si el escenario no puede tolerar el procesamiento duplicado, entonces los desarrolladores de la aplicación deberían agregar lógica adicional a las aplicaciones para solucionar la entrega de mensajes duplicados. A menudo, esto se consigue con el método `getMessageId` del mensaje, que permanece constante en todos los intentos de entrega.

## <a name="delete-topics-and-subscriptions"></a>Eliminación de temas y suscripciones
Para eliminar un tema o una suscripción, use los métodos `ServiceBusRestProxy->deleteTopic` o `ServiceBusRestProxy->deleteSubscripton`, respectivamente. Al eliminar un tema también se eliminan todas las suscripciones que estén registradas con él.

En el ejemplo siguiente se muestra cómo eliminar un tema denominado `mytopic` y sus suscripciones registradas.

```php
require_once 'vendor/autoload.php';

use WindowsAzure\ServiceBus\ServiceBusService;
use WindowsAzure\ServiceBus\ServiceBusSettings;
use WindowsAzure\Common\ServiceException;

// Create Service Bus REST proxy.
$serviceBusRestProxy = ServicesBuilder::getInstance()->createServiceBusService($connectionString);

try {
    // Delete topic.
    $serviceBusRestProxy->deleteTopic("mytopic");
}
catch(ServiceException $e){
    // Handle exception based on error codes and messages.
    // Error codes and messages are here: 
    // https://docs.microsoft.com/rest/api/storageservices/Common-REST-API-Error-Codes
    $code = $e->getCode();
    $error_message = $e->getMessage();
    echo $code.": ".$error_message."<br />";
}
```

Si usa el método `deleteSubscription`, puede eliminar una suscripción de manera independiente:

```php
$serviceBusRestProxy->deleteSubscription("mytopic", "mysubscription");
```

> [!NOTE]
> Puede administrar los recursos de Service Bus con el [Explorador de Service Bus](https://github.com/paolosalvatori/ServiceBusExplorer/). El Explorador de Service Bus permite a los usuarios conectarse a un espacio de nombres de Service Bus y administrar las entidades de mensajería de una forma sencilla. La herramienta dispone de características avanzadas, como la funcionalidad de importación y exportación o la capacidad de probar el tema, las colas, las suscripciones, los servicios de retransmisión, los centros de notificaciones y los centros de eventos. 

## <a name="next-steps"></a>Pasos siguientes
Para más información, consulte [Colas, temas y suscripciones de Service Bus][Queues, topics, and subscriptions].

[BrokeredMessage]: /dotnet/api/microsoft.servicebus.messaging.brokeredmessage
[Queues, topics, and subscriptions]: service-bus-queues-topics-subscriptions.md
[sqlfilter]: /dotnet/api/microsoft.servicebus.messaging.sqlfilter
[require-once]: https://php.net/require_once
[Service Bus quotas]: service-bus-quotas.md