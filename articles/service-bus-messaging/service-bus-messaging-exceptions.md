---
title: Excepciones de mensajería de Azure Service Bus | Microsoft Docs
description: En este artículo se proporciona una lista de las excepciones de mensajería de Azure Service Bus y las acciones sugeridas para cuando se produce la excepción.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 3c648cdce87b26e6258ff8bc25506ca2ebd3bbd9
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88064611"
---
# <a name="service-bus-messaging-exceptions"></a>Excepciones de mensajería de Service Bus
En este artículo se enumeran las excepciones de .NET generadas por API de .NET Framework. 

## <a name="exception-categories"></a>Categorías de excepciones
Las API de mensajería generan excepciones que pueden dividirse en las siguientes categorías, junto con la acción asociada que puede realizar para intentar corregirlas. El significado y las causas de una excepción pueden variar en función del tipo de entidad de mensajería:

1. Error de codificación de usuario ([System.ArgumentException](/dotnet/api/system.argumentexception?view=netcore-3.1), [System.InvalidOperationException](/dotnet/api/system.invalidoperationexception?view=netcore-3.1), [System.OperationCanceledException](/dotnet/api/system.operationcanceledexception?view=netcore-3.1), [System.Runtime.Serialization.SerializationException](/dotnet/api/system.runtime.serialization.serializationexception?view=netcore-3.1)). Acción general: intente corregir el código antes de continuar.
2. Error de instalación/configuración ([Microsoft.ServiceBus.Messaging.MessagingEntityNotFoundException](/dotnet/api/microsoft.azure.servicebus.messagingentitynotfoundexception) y [System.UnauthorizedAccessException](/dotnet/api/system.unauthorizedaccessexception?view=netcore-3.1)). Acción general: revise la configuración y cámbiela si es necesario.
3. Excepciones transitorias ([Microsoft.ServiceBus.Messaging.MessagingException](/dotnet/api/microsoft.servicebus.messaging.messagingexception), [Microsoft.ServiceBus.Messaging.ServerBusyException](/dotnet/api/microsoft.azure.servicebus.serverbusyexception), [Microsoft.ServiceBus.Messaging.MessagingCommunicationException](/dotnet/api/microsoft.servicebus.messaging.messagingcommunicationexception)). Acción general: intente realizar de nuevo la operación o informe a los usuarios. La clase `RetryPolicy` del SDK de cliente puede configurarse para controlar reintentos automáticamente. Consulte [Instrucciones de reintento](/azure/architecture/best-practices/retry-service-specific#service-bus) para más información.
4. Otras excepciones ([System.Transactions.TransactionException](/dotnet/api/system.transactions.transactionexception?view=netcore-3.1), [System.TimeoutException](/dotnet/api/system.timeoutexception?view=netcore-3.1), [Microsoft.ServiceBus.Messaging.MessageLockLostException](/dotnet/api/microsoft.azure.servicebus.messagelocklostexception), [Microsoft.ServiceBus.Messaging.SessionLockLostException](/dotnet/api/microsoft.azure.servicebus.sessionlocklostexception)). Acción general: específica del tipo de excepción; consulte la tabla de la siguiente sección: 

## <a name="exception-types"></a>Tipos de excepciones
En la tabla siguiente se describen los tipos de excepción de mensajería, sus causas y las acciones sugeridas que puede realizar.

| **Tipo de excepción** | **Descripción/causa/ejemplos** | **Acción sugerida** | **Nota sobre el reintento automático o inmediato** |
| --- | --- | --- | --- |
| [TimeoutException](/dotnet/api/system.timeoutexception?view=netcore-3.1) |El servidor no respondió a la operación solicitada en el tiempo especificado, que está controlado por [OperationTimeout](/dotnet/api/microsoft.servicebus.messaging.messagingfactorysettings). Puede que el servidor haya completado la operación solicitada. Esto puede producirse debido a un retraso de red o a otros retrasos de infraestructura. |Compruebe la coherencia del estado del sistema y vuelva a intentarlo si es necesario. Consulte [Excepciones de tiempo de espera](#timeoutexception). |El reintento podría resultar útil en algunos casos; agregue lógica de reintento al código. |
| [InvalidOperationException](/dotnet/api/system.invalidoperationexception?view=netcore-3.1) |La operación del usuario solicitada no está permitida en el servidor o servicio. Consulte el mensaje de excepción para obtener detalles. Por ejemplo, [Complete()](/dotnet/api/microsoft.azure.servicebus.queueclient.completeasync) genera esta excepción si el mensaje se recibió en el modo [ReceiveAndDelete](/dotnet/api/microsoft.azure.servicebus.receivemode) . |Compruebe el código y la documentación. Asegúrese de que la operación solicitada sea válida. |Los reintentos no funcionan. |
| [OperationCanceledException](/dotnet/api/system.operationcanceledexception?view=netcore-3.1) |Se realiza un intento para invocar una operación en un objeto que ya se ha cerrado, anulado o eliminado. En raras ocasiones, la transacción de ambiente ya se ha eliminado. |Compruebe el código y asegúrese de que no invoca operaciones de un objeto desechado. |Los reintentos no funcionan. |
| [UnauthorizedAccessException](/dotnet/api/system.unauthorizedaccessexception?view=netcore-3.1) |El objeto [TokenProvider](/dotnet/api/microsoft.servicebus.tokenprovider) no pudo adquirir un token, el token no es válido o no contiene las notificaciones necesarias para realizar la operación. |Asegúrese de que el proveedor de tokens se cree con los valores correctos. Compruebe la configuración de Access Control Service. |El reintento podría resultar útil en algunos casos; agregue lógica de reintento al código. |
| [ArgumentException](/dotnet/api/system.argumentexception?view=netcore-3.1)<br /> [ArgumentNullException](/dotnet/api/system.argumentnullexception?view=netcore-3.1)<br />[ArgumentOutOfRangeException](/dotnet/api/system.argumentoutofrangeexception?view=netcore-3.1) |Uno o varios de los argumentos proporcionados para el método no son válidos.<br /> El URI proporcionado a [NamespaceManager](/dotnet/api/microsoft.servicebus.namespacemanager) o [Create](/dotnet/api/microsoft.servicebus.messaging.messagingfactory) contiene segmentos de ruta de acceso.<br /> El esquema de URI proporcionado a [NamespaceManager](/dotnet/api/microsoft.servicebus.namespacemanager) o [Create](/dotnet/api/microsoft.servicebus.messaging.messagingfactory) no es válido. <br />El valor de la propiedad es mayor de 32 KB. |Compruebe el código de llamada y asegúrese de que los argumentos sean correctos. |Los reintentos no funcionan. |
| [MessagingEntityNotFoundException](/dotnet/api/microsoft.azure.servicebus.messagingentitynotfoundexception) |La entidad asociada a la operación no existe o se eliminó. |Asegúrese de que la entidad exista. |Los reintentos no funcionan. |
| [MessageNotFoundException](/dotnet/api/microsoft.servicebus.messaging.messagenotfoundexception) |Se intenta recibir un mensaje con una secuencia numérica concreta. Este mensaje no se encuentra. |Asegúrese de que aún no se recibió el mensaje. Compruebe la cola de correo devuelto para ver si el mensaje se procesó como correo devuelto. |Los reintentos no funcionan. |
| [MessagingCommunicationException](/dotnet/api/microsoft.servicebus.messaging.messagingcommunicationexception) |El cliente no puede establecer una conexión a Service Bus. |Asegúrese de que el nombre de host proporcionado sea correcto y que este sea accesible. |El reintento podría resultar útil si hay problemas de conectividad intermitente. |
| [ServerBusyException](/dotnet/api/microsoft.azure.servicebus.serverbusyexception) |El servicio no puede procesar la solicitud en este momento. |El cliente puede esperar durante un período de tiempo y volver a intentar realizar la operación. |El cliente puede volver a intentarlo tras un determinado intervalo de tiempo. Si el reintento genera otra excepción, compruebe el comportamiento de reintento de esa excepción. |
| [MessagingException](/dotnet/api/microsoft.servicebus.messaging.messagingexception) |Excepción de mensajería genérica que puede producirse en los siguientes casos:<p>Se intentó crear [QueueClient](/dotnet/api/microsoft.azure.servicebus.queueclient) con un nombre o una ruta de acceso que pertenece a un tipo de entidad diferente (por ejemplo, un tema).</p><p>Intento de enviar un mensaje mayor de 256 KB. </p>El servidor o servicio encontró un error durante el procesamiento de la solicitud. Consulte el mensaje de excepción para obtener detalles. Por lo general, se trata de una excepción transitoria.</p><p>La solicitud finalizó porque la entidad se está limitando. Código de error: 50001, 50002, 50008. </p> | Compruebe el código y asegúrese de que solo se usan objetos serializables en el cuerpo del mensaje (o use un serializador personalizado). <p>Consulte la documentación de los tipos de valor de las propiedades admitidos y use solo los tipos compatibles.</p><p> Compruebe la propiedad [IsTransient](/dotnet/api/microsoft.servicebus.messaging.messagingexception) . Si es **true**, puede volver a intentar la operación. </p>| Si la excepción se debe a una limitación, espere unos segundos y vuelva a intentar la operación. El comportamiento de reintento es indefinido y quizá no resulte útil en otros escenarios.|
| [MessagingEntityAlreadyExistsException](/dotnet/api/microsoft.servicebus.messaging.messagingentityalreadyexistsexception) |Se intenta crear una entidad con un nombre que ya se usa en otra entidad de ese espacio de nombres de servicio. |Elimine la entidad existente o elija un nombre diferente para la entidad que quiere crear. |Los reintentos no funcionan. |
| [QuotaExceededException](/dotnet/api/microsoft.azure.servicebus.quotaexceededexception) |La entidad de mensajería ha alcanzado su tamaño máximo permitido, o bien se superó el número máximo de conexiones en un espacio de nombres. |Cree espacio en la entidad recibiendo mensajes de esta o de sus subcolas. Consulte [QuotaExceededException](#quotaexceededexception). |El reintento podría resultar útil si los mensajes se eliminan mientras este se lleva a cabo. |
| [RuleActionException](/dotnet/api/microsoft.servicebus.messaging.ruleactionexception) |Service Bus devuelve esta excepción si se intenta crear una acción de regla no válida. Si se produjo un error al procesar la acción de regla de un mensaje, Service Bus adjunta esta excepción a dicho mensaje. |Compruebe si la acción de regla es correcta. |Los reintentos no funcionan. |
| [FilterException](/dotnet/api/microsoft.servicebus.messaging.filterexception) |Service Bus devuelve esta excepción si se intenta crear un filtro no válido. Si se produjo un error al procesar el filtro de un mensaje, Service Bus adjunta esta excepción a dicho mensaje. |Compruebe si el filtro es correcto. |Los reintentos no funcionan. |
| [SessionCannotBeLockedException](/dotnet/api/microsoft.servicebus.messaging.sessioncannotbelockedexception) |Se intenta aceptar una sesión con un id. de sesión específico, pero actualmente esta se encuentra bloqueada por otro cliente. |Asegúrese de que otros clientes desbloqueen la sesión. |El reintento podría ser útil si la sesión se publica mientras este se lleva a cabo. |
| [TransactionSizeExceededException](/dotnet/api/microsoft.servicebus.messaging.transactionsizeexceededexception) |Hay demasiadas operaciones que forman parte de la transacción. |Reduzca el número de operaciones que forman parte de esta transacción. |Los reintentos no funcionan. |
| [MessagingEntityDisabledException](/dotnet/api/microsoft.azure.servicebus.messagingentitydisabledexception) |Solicitud para realizar una operación en tiempo de ejecución en una entidad deshabilitada. |Active la entidad. |El reintento podría ser útil si la entidad se activa mientras este se lleva a cabo. |
| [NoMatchingSubscriptionException](/dotnet/api/microsoft.servicebus.messaging.nomatchingsubscriptionexception) |Service Bus devuelve esta excepción si envía un mensaje a un tema que tiene el filtrado previo habilitado y dicho mensaje no coincide con ninguno de los filtros. |Asegúrese de que coincida con al menos un filtro. |Los reintentos no funcionan. |
| [MessageSizeExceededException](/dotnet/api/microsoft.servicebus.messaging.messagesizeexceededexception) |Una carga de mensaje supera el límite de 256 KB. El límite de 256 KB es el tamaño total del mensaje, que puede incluir las propiedades del sistema y cualquier sobrecarga .NET. |Reduzca el tamaño de la carga del mensaje y vuelva a intentar la operación. |Los reintentos no funcionan. |
| [TransactionException](/dotnet/api/system.transactions.transactionexception?view=netcore-3.1) |La transacción de ambiente (*Transaction.Current*) no es válida. Puede se haya completado o anulado. La excepción interna puede proporcionar información adicional. | |Los reintentos no funcionan. |
| [TransactionInDoubtException](/dotnet/api/system.transactions.transactionindoubtexception?view=netcore-3.1) |Se intenta realizar una operación en una transacción dudosa o se intenta confirmar una transacción y esta se convierte en dudosa. |La aplicación debe controlar esta excepción (como caso especial), porque puede que ya se haya confirmado la transacción. |- |

## <a name="quotaexceededexception"></a>QuotaExceededException
[QuotaExceededException](/dotnet/api/microsoft.azure.servicebus.quotaexceededexception) indica que se ha superado una cuota de una entidad específica.

### <a name="queues-and-topics"></a>Colas y temas
Para las colas y los temas, suele tratarse del tamaño de la cola. La propiedad de mensaje de error contiene más detalles, como en el ejemplo siguiente:

```Output
Microsoft.ServiceBus.Messaging.QuotaExceededException
Message: The maximum entity size has been reached or exceeded for Topic: 'xxx-xxx-xxx'. 
    Size of entity in bytes:1073742326, Max entity size in bytes:
1073741824..TrackingId:xxxxxxxxxxxxxxxxxxxxxxxxxx, TimeStamp:3/15/2013 7:50:18 AM
```

El mensaje indica que el tema superaba su límite de tamaño, en este caso 1 GB (el límite de tamaño predeterminado). 

### <a name="namespaces"></a>Espacios de nombres

Para espacios de nombres, [QuotaExceededException](/dotnet/api/microsoft.azure.servicebus.quotaexceededexception) puede indicar que una aplicación ha superado el número máximo de conexiones en un espacio de nombres. Por ejemplo:

```Output
Microsoft.ServiceBus.Messaging.QuotaExceededException: ConnectionsQuotaExceeded for namespace xxx.
<tracking-id-guid>_G12 ---> 
System.ServiceModel.FaultException`1[System.ServiceModel.ExceptionDetail]: 
ConnectionsQuotaExceeded for namespace xxx.
```

### <a name="common-causes"></a>Causas comunes
Hay dos causas comunes de este error: la cola de mensajes fallidos y receptores de mensajes que no funcionan.

1. **[Cola de mensajes fallidos](service-bus-dead-letter-queues.md)** Un lector no puede completar los mensajes, que se devuelven a la cola o tema al expirar el bloqueo. Esto puede suceder si el lector encuentra una excepción que impide que llame a [BrokeredMessage.Complete](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.complete). Una vez que un mensaje se ha leído 10 veces, se mueve a la cola de mensajes fallidos de forma predeterminada. La propiedad [QueueDescription.MaxDeliveryCount](/dotnet/api/microsoft.servicebus.messaging.queuedescription.maxdeliverycount) controla este comportamiento, que tiene un valor predeterminado de 10. Al acumularse los mensajes en la cola de mensajes fallidos, estos ocupan espacio.
   
    Para resolver el problema, lea y complete los mensajes de la cola de mensajes fallidos, igual que haría si se encontraran en cualquier otra cola. Puede usar el método [FormatDeadLetterPath](/dotnet/api/microsoft.azure.servicebus.entitynamehelper.formatdeadletterpath) para ayudar a dar formato a la ruta de acceso de la cola de mensajes fallidos.
2. **Receptor detenido**. Un receptor ha dejado de recibir mensajes de una cola o suscripción. Para identificar este aspecto, eche un vistazo a la propiedad [QueueDescription.MessageCountDetails](/dotnet/api/microsoft.servicebus.messaging.messagecountdetails) , que muestra el desglose completo de los mensajes. Si la propiedad [ActiveMessageCount](/dotnet/api/microsoft.servicebus.messaging.messagecountdetails.activemessagecount) es alta o creciente, los mensajes no se leen tan rápido como se escriben.

## <a name="timeoutexception"></a>TimeoutException
[TimeoutException](/dotnet/api/system.timeoutexception?view=netcore-3.1) indica que la operación iniciada por el usuario está superando el tiempo de espera. 

Debe comprobar el valor de la propiedad [ServicePointManager.DefaultConnectionLimit](/dotnet/api/system.net.servicepointmanager.defaultconnectionlimit?view=netcore-3.1), porque si alcanza este límite también puede causar una excepción [TimeoutException](/dotnet/api/system.timeoutexception?view=netcore-3.1).

### <a name="queues-and-topics"></a>Colas y temas
Para las colas y los temas, el tiempo de espera se especifica en la propiedad [MessagingFactorySettings.OperationTimeout](/dotnet/api/microsoft.servicebus.messaging.messagingfactorysettings) como parte de la cadena de conexión o mediante [ServiceBusConnectionStringBuilder](/dotnet/api/microsoft.azure.servicebus.servicebusconnectionstringbuilder). El propio mensaje de error puede variar, pero siempre contiene el valor de tiempo de espera especificado para la operación actual. 

## <a name="messagelocklostexception"></a>MessageLockLostException

### <a name="cause"></a>Causa

**MessageLockLostException** se produce cuando se recibe un mensaje mediante el modo Receive [PeekLock](message-transfers-locks-settlement.md#peeklock) y el bloqueo mantenido por el cliente expira en el lado del servicio.

El bloqueo de un mensaje puede expirar debido a varios motivos: 

  * El temporizador de bloqueo ha expirado antes de que la aplicación cliente lo renovara.
  * La aplicación cliente adquirió el bloqueo, lo guardó en un almacén persistente y luego se reinició. Una vez que se reinició, la aplicación cliente examinó los mensajes en proceso e intentó completarlos.

### <a name="resolution"></a>Solución

En caso de producirse **MessageLockLostException**, la aplicación cliente ya no podrá procesar el mensaje. De manera opcional, la aplicación cliente puede considerar registrar la excepción para su análisis, pero el cliente *debe* eliminar el mensaje.

Dado que el bloqueo del mensaje ha expirado, volvería a la cola (o suscripción), y la siguiente aplicación cliente que llame a Receive podría procesarlo.

Si se ha superado **MaxDeliveryCount**, el mensaje puede moverse a **DeadLetterQueue**.

## <a name="sessionlocklostexception"></a>SessionLockLostException

### <a name="cause"></a>Causa

**SessionLockLostException** se produce cuando se acepta una sesión y el bloqueo mantenido por el cliente expira en el lado del servicio.

El bloqueo de una sesión puede expirar debido a varios motivos: 

  * El temporizador de bloqueo ha expirado antes de que la aplicación cliente lo renovara.
  * La aplicación cliente adquirió el bloqueo, lo guardó en un almacén persistente y luego se reinició. Una vez que se reinició, la aplicación cliente examinó las sesiones en proceso e intentó procesar los mensajes de esas sesiones.

### <a name="resolution"></a>Solución

En caso de producirse **SessionLockLostException**, la aplicación cliente ya no podrá procesar los mensajes en la sesión. La aplicación cliente puede considerar registrar la excepción para su análisis, pero el cliente *debe* eliminar el mensaje.

Dado que el bloqueo de la sesión ha expirado, volvería a la cola (o suscripción), y la siguiente aplicación cliente que acepte la sesión podría bloquearla. Dado que una única aplicación cliente mantiene el bloqueo de la sesión en un momento dado, el procesamiento en orden está garantizado.

## <a name="socketexception"></a>SocketException

### <a name="cause"></a>Causa

**SocketException** se produce en los casos siguientes:
   * Cuando se produce un error en un intento de conexión porque el host no respondió correctamente después de un tiempo especificado (código de error de TCP 10060).
   * Hubo un error en una conexión establecida porque el host conectado no respondió.
   * Hubo un error al procesar el mensaje o el host remoto superó el tiempo de espera.
   * Problema de recursos de red subyacentes.

### <a name="resolution"></a>Solución

Los errores **SocketException** indican que la VM que hospeda las aplicaciones no puede convertir el nombre `<mynamespace>.servicebus.windows.net` en la dirección IP correspondiente. 

Compruebe si el comando siguiente logra una asignación a una dirección IP.

```Powershell
PS C:\> nslookup <mynamespace>.servicebus.windows.net
```

Lo que proporcionaría una salida como la siguiente:

```bash
Name:    <cloudappinstance>.cloudapp.net
Address:  XX.XX.XXX.240
Aliases:  <mynamespace>.servicebus.windows.net
```

Si el nombre anterior **no se resuelve** en una dirección IP y en el alias del espacio de nombres, pida al administrador de red que investigue. La resolución de nombres se realiza a través de un servidor DNS, normalmente un recurso de la red del cliente. Si la resolución de DNS se realiza mediante Azure DNS, póngase en contacto con Soporte técnico de Azure.

Si la resolución de nombres **funciona según lo previsto**, compruebe [aquí](service-bus-troubleshooting-guide.md#connectivity-certificate-or-timeout-issues) si se permite la conexión a Azure Service Bus.


## <a name="messagingexception"></a>MessagingException

### <a name="cause"></a>Causa

**MessagingException** es una excepción genérica que puede producirse por varios motivos. A continuación se enumeran algunos de ellos.

   * Se realiza un intento de crear una clase **QueueClient** en un **Tema** o una **Suscripción**.
   * El tamaño del mensaje enviado es mayor que el límite para el nivel dado. Lea más sobre los [límites y cuotas](service-bus-quotas.md) de Service Bus.
   * Se finalizó la solicitud del plano de datos específica (envío, recepción, finalización, abandono) debido a la limitación.
   * Problemas transitorios causados por actualizaciones y reinicios del servicio.

> [!NOTE]
> La lista anterior de excepciones no es exhaustiva.

### <a name="resolution"></a>Solución

Los pasos de resolución dependen de lo que haya provocado el error **MessagingException**.

   * Por **problemas transitorios** (donde ***isTransient*** está establecido en ***true***) o para **problemas de limitación**, volver a intentar la operación puede ser una solución. Para esto, puede aprovecharse la directiva de reintentos predeterminada en el SDK.
   * Por otros problemas, los detalles de la excepción indican el problema, y los pasos de resolución se pueden deducir desde allí.

## <a name="next-steps"></a>Pasos siguientes
Para obtener la referencia completa de la API de .NET para Service Bus, consulte la [referencia de la API de .NET de Azure](/dotnet/api/overview/azure/service-bus).
Para obtener consejos de solución de problemas, consulte la [Guía de solución de problemas](service-bus-troubleshooting-guide.md).