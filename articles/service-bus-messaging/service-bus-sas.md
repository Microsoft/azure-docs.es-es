---
title: Control de acceso de Azure Service Bus con Firmas de acceso compartido
description: Información general sobre el control de acceso de Service Bus con Firmas de acceso compartido, detalles de la autorización con SAS mediante Azure Service Bus.
ms.topic: article
ms.date: 07/30/2020
ms.openlocfilehash: 8e48858fd76bcf4667cfff1237f49597a477b3e8
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88066192"
---
# <a name="service-bus-access-control-with-shared-access-signatures"></a>Control de acceso de Service Bus con Firmas de acceso compartido

Las *firmas de acceso compartido* (SAS) son el mecanismo de seguridad principal de la mensajería de Service Bus. En este artículo se tratan las firmas acceso compartido, cómo funcionan y cómo usarlas independientemente de la plataforma.

SAS protege el acceso a Service Bus en función de las reglas de autorización. Estas se configuran en función de un espacio de nombres o una entidad de mensajería (retransmisión, cola o tema). Una regla de autorización tiene un nombre, se asocia con unos derechos específicos e incluye un par de claves criptográficas. Para generar un token de SAS, debe usar el nombre y la clave de la regla a través del SDK de Service Bus o en su propio código. A continuación, el cliente puede pasar el token a Service Bus para demostrar la autorización para la operación solicitada.

> [!NOTE]
> Azure Service Bus admite la autorización de acceso a un espacio de nombres de Service Bus y sus entidades mediante Azure Active Directory (Azure AD). La autorización de usuarios o aplicaciones mediante un token de OAuth 2.0 devuelto por Azure AD proporciona una seguridad superior y facilidad de uso sobre las firmas de acceso compartido (SAS). Con Azure AD, no hay ninguna necesidad de almacenar los tokens en su código y arriesgarse a posibles vulnerabilidades de seguridad.
>
> Microsoft recomienda usar Azure AD con las aplicaciones de Azure Service Bus cuando sea posible. Para más información, consulte los siguientes artículos.
> - [Autenticación y autorización de una aplicación con Azure Active Directory para acceder a entidades de Azure Service Bus](authenticate-application.md).
> - [Autenticación de una identidad administrada con Azure Active Directory para acceder a recursos de Azure Service Bus](service-bus-managed-service-identity.md).

## <a name="overview-of-sas"></a>Información general de SAS

Las Firmas de acceso compartido son un mecanismo de autorización basada en notificaciones que utilizan tokens simples. Si utiliza SAS, las claves nunca se pasan en la conexión. Las claves se utilizan para firmar criptográficamente información que más adelante pueda verificar el servicio. El uso de SAS es similar a un esquema de nombre de usuario y contraseña donde el cliente está en posesión inmediata de un nombre de regla de autorización y una clave coincidente. SAS también puede utilizarse de forma similar a un modelo de seguridad federado, donde el cliente recibe un token de acceso firmado y de tiempo limitado de un servicio de token de seguridad sin poseer en ningún momento la clave de firma.

La autenticación de SAS en Service Bus se configura con [reglas de autorización de acceso compartido](/dotnet/api/microsoft.servicebus.messaging.sharedaccessauthorizationrule) con nombre que tienen derechos de acceso asociados y algunas claves criptográficas principales y secundarias. Las claves son valores de 256 bits en representación de Base64. Puede configurar reglas en el nivel de espacio de nombres, en [retransmisiones](../azure-relay/relay-what-is-it.md), [colas](service-bus-messaging-overview.md#queues) y [temas](service-bus-messaging-overview.md#topics) de Service Bus.

El token [Firma de acceso compartido](/dotnet/api/microsoft.servicebus.sharedaccesssignaturetokenprovider) contiene el nombre de la regla de autorización elegida, el URI del recurso al que se debe acceder, un instante de expiración y una firma criptográfica de HMAC-SHA256 calculada sobre estos campos utilizando la clave criptográfica principal o secundaria de la regla de autorización elegida.

## <a name="shared-access-authorization-policies"></a>Directivas de autorización de acceso compartido

Cada espacio de nombres de Service Bus y cada entidad de Service Bus tiene una directiva de autorización de acceso compartido compuesta por reglas. La directiva a nivel de espacio de nombres se aplica a todas las entidades del espacio de nombres, independientemente de su configuración de directiva individual.

Para cada regla de directiva de autorización, decida tres tipos de información: **nombre**, **ámbito** y **derechos**. El **nombre** es solo eso; un nombre único dentro de ese ámbito. El ámbito es bastante sencillo: es el URI del recurso en cuestión. Para un espacio de nombres de Service Bus, el ámbito es el nombre de dominio completo (FQDN), como `https://<yournamespace>.servicebus.windows.net/`.

Los derechos otorgados por la regla de directiva pueden ser una combinación de:

* "Enviar": otorga el derecho a enviar mensajes a la entidad.
* "Escuchar": otorga el derecho a escuchar (retransmisión) o a recibir (cola, suscripciones) y a administrar todos los mensajes relacionados.
* "Administrar": otorga el derecho a administrar la topología del espacio de nombres, incluidas la creación y la eliminación de entidades.

El derecho "Administrar" incluye los derechos "Enviar" y "Recibir".

Una directiva de espacio de nombres o entidad puede contener hasta 12 reglas de autorización de acceso compartido, lo que proporciona espacio para tres conjuntos de reglas, y cada una de ellas cubre los derechos básicos y la combinación de "Enviar" y "Escuchar". Este límite subraya que el almacén de directivas de SAS no pretende ser un almacén de la cuenta de usuario o servicio. Si la aplicación necesita conceder acceso a Service Bus en función de las identidades de usuario o servicio, debe implementar un servicio de token de seguridad que emita tokens de SAS después de una comprobación de acceso y autenticación.

A la regla de autorización se le asigna una *clave principal* y una *clave secundaria*. Son claves de alta seguridad criptográfica. No las pierda ni las revele; siempre estarán disponibles en [Azure Portal][Azure portal]. Puede usar cualquiera de las claves generadas y regenerarlas en cualquier momento. Si vuelve a generar o cambia una clave en la directiva, todos los tokens emitidos previamente en base a esa clave dejan de ser válidos inmediatamente. Sin embargo, las conexiones continuas creadas de acuerdo con esos tokens seguirán funcionando hasta que el token expire.

Cuando se crea un espacio de nombres de Service Bus, se crea automáticamente una regla de directiva denominada **RootManageSharedAccessKey** para el espacio de nombres. Esta directiva tiene permisos de administración para todo el espacio de nombres. Se recomienda tratar esta regla como una cuenta **raíz** administrativa, así que no la use en la aplicación. Puede crear reglas de directivas adicionales en la pestaña **Configurar** para el espacio de nombres en el portal, a través de PowerShell o la CLI de Azure.

## <a name="configuration-for-shared-access-signature-authentication"></a>Configuración de la autenticación de firma de acceso compartido

Puede configurar la regla [SharedAccessAuthorizationRule](/dotnet/api/microsoft.servicebus.messaging.sharedaccessauthorizationrule) en los espacios de nombres, las colas o los temas de Service Bus. La configuración de una regla [SharedAccessAuthorizationRule](/dotnet/api/microsoft.servicebus.messaging.sharedaccessauthorizationrule) en una suscripción de Service Bus no se admite, pero puede usar las reglas configuradas en un espacio de nombres o un tema para asegurar el acceso a las suscripciones. Para ver un ejemplo funcional que ilustra este procedimiento, consulte el ejemplo [Uso de la autenticación de firma de acceso compartido (SAS) con suscripciones de Service Bus](https://code.msdn.microsoft.com/Using-Shared-Access-e605b37c) .

![SAS](./media/service-bus-sas/service-bus-namespace.png)

En esta ilustración, las reglas de autorización *manageRuleNS*, *sendRuleNS* y *listenRuleNS* se aplican tanto a la cola Q1 como al tema T1, mientras que *listenRuleQ* y *sendRuleQ* se aplican solo a la cola Q1 y *sendRuleT* solo al tema T1.

## <a name="generate-a-shared-access-signature-token"></a>Generación de un token de Firmas de acceso compartido

Cualquier cliente que tenga acceso al nombre de una regla de autorización y una de sus claves de firma puede generar un token de SAS. El token se genera diseñando una cadena con el siguiente formato:

```
SharedAccessSignature sig=<signature-string>&se=<expiry>&skn=<keyName>&sr=<URL-encoded-resourceURI>
```

* **`se`** : instante de expiración del token. Entero que refleja los segundos transcurridos desde la época `00:00:00 UTC` del 1 de enero de 1970 (época de UNIX) cuando expira el token.
* **`skn`** : nombre de la regla de autorización.
* **`sr`** : URI del recurso al que se accede.
* **`sig`** : firma.

El valor de `signature-string` es el hash SHA-256 calculado sobre el URI del recurso (**ámbito** tal y como se describe en la sección anterior) y la representación de cadenas del instante de expiración del token, separados por LF.

El cálculo del hash es similar al siguiente pseudocódigo y devuelve un valor de hash de 256 bits/32 bytes.

```
SHA-256('https://<yournamespace>.servicebus.windows.net/'+'\n'+ 1438205742)
```

El token contiene los valores sin el hash para que el destinatario pueda volver a calcular el hash con los mismos parámetros y verificar que el emisor posee una clave de firma válida.

El URI de recurso es el URI completo del recurso de Service Bus al que se solicita el acceso. Por ejemplo, `http://<namespace>.servicebus.windows.net/<entityPath>` o `sb://<namespace>.servicebus.windows.net/<entityPath>`; es decir, `http://contoso.servicebus.windows.net/contosoTopics/T1/Subscriptions/S3`. 

**El URI debe estar [codificado por porcentaje](/dotnet/api/system.web.httputility.urlencode?view=netcore-3.1).**

La regla de autorización de acceso compartido usada para firmar debe configurarse en la entidad especificada por este URI, o uno de sus primarios jerárquicos. Por ejemplo, `http://contoso.servicebus.windows.net/contosoTopics/T1` o `http://contoso.servicebus.windows.net` en el ejemplo anterior.

Un token SAS es válido para todos los recursos con el prefijo `<resourceURI>` que se usen en `signature-string`.

> [!NOTE]
> Para obtener ejemplos de cómo generar un token de SAS con distintos lenguajes de programación, consulte [Generación de un token de SAS](/rest/api/eventhub/generate-sas-token). 

## <a name="regenerating-keys"></a>Regeneración de claves

Se recomienda regenerar periódicamente las claves usadas en el objeto [SharedAccessAuthorizationRule](/dotnet/api/microsoft.servicebus.messaging.sharedaccessauthorizationrule) . Las ranuras de las claves principales y secundarias existen para que pueda girar las claves de forma gradual. Si la aplicación suele utilizar la clave principal, puede copiarla en la ranura de la clave secundaria y, a continuación, volver a generar la clave principal. Entonces podrá configurar el nuevo valor de la clave principal en las aplicaciones cliente, las cuales tienen acceso continuo mediante la clave principal antigua en la ranura secundaria. Una vez que se actualicen todos los clientes, puede volver a generar la clave secundaria para finalmente eliminar la clave principal antigua.

Si está seguro o sospecha que una clave está en peligro y debe revocar las claves, puede volver a generar los valores de [PrimaryKey](/dotnet/api/microsoft.servicebus.messaging.sharedaccessauthorizationrule) y [SecondaryKey](/dotnet/api/microsoft.servicebus.messaging.sharedaccessauthorizationrule) de un objeto [SharedAccessAuthorizationRule](/dotnet/api/microsoft.servicebus.messaging.sharedaccessauthorizationrule) y reemplazarlos por las nuevas claves. Este procedimiento invalida todos los tokens firmados con las claves antiguas.

## <a name="shared-access-signature-authentication-with-service-bus"></a>Autenticación con Firma de acceso compartido en Service Bus

Los escenarios descritos a continuación incluyen la configuración de reglas de autorización, la generación de tokens SAS y la autorización del cliente.

Para un ejemplo funcional completo de una aplicación del Bus de servicio que ilustra la configuración y usa la autorización SAS, consulte [Autenticación con firma de acceso compartido en Service Bus](https://code.msdn.microsoft.com/Shared-Access-Signature-0a88adf8). Hay un ejemplo relacionado que ilustra el uso de reglas de autorización SAS configuradas en espacios de nombres o temas para proteger las suscripciones de Service Bus disponible aquí: [Uso de la autenticación de firma de acceso compartido (SAS) con suscripciones de Service Bus](https://code.msdn.microsoft.com/Using-Shared-Access-e605b37c).

## <a name="access-shared-access-authorization-rules-on-an-entity"></a>Acceso a las reglas de autorización de acceso compartido en una entidad

Gracias a las bibliotecas de .NET Framework de Service Bus, puede acceder a un objeto [Microsoft.ServiceBus.Messaging.SharedAccessAuthorizationRule](/dotnet/api/microsoft.servicebus.messaging.sharedaccessauthorizationrule) configurado en una cola o un tema de Service Bus mediante la colección [AuthorizationRules](/dotnet/api/microsoft.servicebus.messaging.authorizationrules) en los objetos [QueueDescription](/dotnet/api/microsoft.servicebus.messaging.queuedescription) o [TopicDescription](/dotnet/api/microsoft.servicebus.messaging.topicdescription) correspondientes.

El código siguiente muestra cómo agregar reglas de autorización para una cola.

```csharp
// Create an instance of NamespaceManager for the operation
NamespaceManager nsm = NamespaceManager.CreateFromConnectionString(
    <connectionString> );
QueueDescription qd = new QueueDescription( <qPath> );

// Create a rule with send rights with keyName as "contosoQSendKey"
// and add it to the queue description.
qd.Authorization.Add(new SharedAccessAuthorizationRule("contosoSendKey",
    SharedAccessAuthorizationRule.GenerateRandomKey(),
    new[] { AccessRights.Send }));

// Create a rule with listen rights with keyName as "contosoQListenKey"
// and add it to the queue description.
qd.Authorization.Add(new SharedAccessAuthorizationRule("contosoQListenKey",
    SharedAccessAuthorizationRule.GenerateRandomKey(),
    new[] { AccessRights.Listen }));

// Create a rule with manage rights with keyName as "contosoQManageKey"
// and add it to the queue description.
// A rule with manage rights must also have send and receive rights.
qd.Authorization.Add(new SharedAccessAuthorizationRule("contosoQManageKey",
    SharedAccessAuthorizationRule.GenerateRandomKey(),
    new[] {AccessRights.Manage, AccessRights.Listen, AccessRights.Send }));

// Create the queue.
nsm.CreateQueue(qd);
```

## <a name="use-shared-access-signature-authorization"></a>Uso de la autorización de la firma de acceso compartido

Las aplicaciones que usan el SDK de .NET de Azure con las bibliotecas .NET de Service Bus pueden usar la autorización SAS mediante la clase [SharedAccessSignatureTokenProvider](/dotnet/api/microsoft.servicebus.sharedaccesssignaturetokenprovider) . El código siguiente muestra el uso del proveedor de token para enviar mensajes a una cola de Service Bus. Como alternativa al uso que se muestra aquí, también puede pasar un token emitido anteriormente al patrón de diseño Factory Method del proveedor de tokens.

```csharp
Uri runtimeUri = ServiceBusEnvironment.CreateServiceUri("sb",
    <yourServiceNamespace>, string.Empty);
MessagingFactory mf = MessagingFactory.Create(runtimeUri,
    TokenProvider.CreateSharedAccessSignatureTokenProvider(keyName, key));
QueueClient sendClient = mf.CreateQueueClient(qPath);

//Sending hello message to queue.
BrokeredMessage helloMessage = new BrokeredMessage("Hello, Service Bus!");
helloMessage.MessageId = "SAS-Sample-Message";
sendClient.Send(helloMessage);
```

También puede utilizar el proveedor de tokens directamente para emitir tokens para que se pasen a otros clientes.

Las cadenas de conexión pueden incluir un nombre de regla (*SharedAccessKeyName*) y una clave de regla (*SharedAccessKey*) o un token emitido anteriormente (*SharedAccessSignature*). Cuando existen en la cadena de conexión que se pasó a algún método de constructor o fabricante que acepte una cadena de conexión, el proveedor de tokens de SAS se crea y rellena automáticamente.

Tenga en cuenta que para usar la autorización SAS con retransmisiones de Service Bus, puede usar claves SAS configuradas en el espacio de nombres de Service Bus. Si crea explícitamente una retransmisión en el espacio de nombres [NamespaceManager](/dotnet/api/microsoft.servicebus.namespacemanager) con un objeto [RelayDescription](/dotnet/api/microsoft.servicebus.messaging.relaydescription), puede establecer las reglas SAS para dicha retransmisión. Para usar la autorización SAS con suscripciones de Service Bus, puede usar claves SAS configuradas en un espacio de nombres de Service Bus o en un tema.

## <a name="use-the-shared-access-signature-at-http-level"></a>Uso de la firma de acceso compartido (en el nivel HTTP)

Ahora que sabe cómo crear firmas de acceso compartido para cualquier entidad de Service Bus, estará listo para llevar a cabo una solicitud HTTP POST:

```http
POST https://<yournamespace>.servicebus.windows.net/<yourentity>/messages
Content-Type: application/json
Authorization: SharedAccessSignature sr=https%3A%2F%2F<yournamespace>.servicebus.windows.net%2F<yourentity>&sig=<yoursignature from code above>&se=1438205742&skn=KeyName
ContentType: application/atom+xml;type=entry;charset=utf-8
```

Recuerde que esto funciona para todo. Puede crear SAS para una cola, un tema o una suscripción.

Si le da un token de SAS a un remitente o cliente, no tiene la clave directamente y no pueden invertir el hash para obtenerla. Como tal, tiene control sobre a lo que pueden tener acceso y durante cuánto tiempo. Algo importante de recordar es que si cambia la clave principal de la directiva, se invalidará cualquier Firma de acceso compartido creada a partir de ella.

## <a name="use-the-shared-access-signature-at-amqp-level"></a>Uso de la firma de acceso compartido (en el nivel AMQP)

En la sección anterior, vimos cómo usar el token SAS con una solicitud HTTP POST para enviar datos a Service Bus. Como sabe, puede obtener acceso a Service Bus mediante Advanced Message Queuing Protocol (AMQP), que es el protocolo preferido por motivos de rendimiento en muchos escenarios. El uso de tokens SAS con AMQP se describe en el documento [Seguridad basada en notificaciones de AMQP, versión 1.0](https://www.oasis-open.org/committees/download.php/50506/amqp-cbs-v1%200-wd02%202013-08-12.doc) que está en fase de borrador desde 2013, pero que es compatible con Azure actualmente.

Antes de comenzar a enviar datos a Service Bus, el publicador debe enviar el token de SAS de un mensaje de AMQP a un nodo de AMQP bien definido con el nombre **$cbs** (puede verlo como una cola especial usada por el servicio para adquirir y validar todos los tokens de SAS). El publicador debe especificar el campo **ReplyTo** en el mensaje de AMQP. Este es el nodo en el que el servicio contestará al publicador con el resultado de la validación del token (un patrón sencillo de solicitud/respuesta entre el publicador y el servicio). Este nodo de respuesta se crea "sobre la marcha" en lo que respecta a la "creación dinámica de nodo remoto", tal como describe la especificación de AMQP 1.0. Después de comprobar que el token SAS es válido, el publicador puede avanzar y comenzar a enviar datos al servicio.

Los pasos siguientes muestran cómo enviar el token de SAS con protocolo AMQP mediante la biblioteca [AMQP.NET Lite](https://github.com/Azure/amqpnetlite). Esto es útil si no puede usar el SDK oficial de Service Bus (por ejemplo, en WinRT, .NET Compact Framework, .NET Micro Framework y Mono) al desarrollar en C\#. Por supuesto, esta biblioteca es útil para comprender cómo funciona la seguridad basada en notificaciones en el nivel AMQP, como pudo observar en el nivel HTTP (con una solicitud HTTP POST y el token de SAS enviado en el encabezado "Autorización"). Si no necesita un conocimiento tan profundo sobre AMQP, puede usar el SDK oficial de Service Bus con aplicaciones de .NET Framework, que lo hará por usted.

### <a name="c35"></a>C&#35;

```csharp
/// <summary>
/// Send claim-based security (CBS) token
/// </summary>
/// <param name="shareAccessSignature">Shared access signature (token) to send</param>
private bool PutCbsToken(Connection connection, string sasToken)
{
    bool result = true;
    Session session = new Session(connection);

    string cbsClientAddress = "cbs-client-reply-to";
    var cbsSender = new SenderLink(session, "cbs-sender", "$cbs");
    var cbsReceiver = new ReceiverLink(session, cbsClientAddress, "$cbs");

    // construct the put-token message
    var request = new Message(sasToken);
    request.Properties = new Properties();
    request.Properties.MessageId = Guid.NewGuid().ToString();
    request.Properties.ReplyTo = cbsClientAddress;
    request.ApplicationProperties = new ApplicationProperties();
    request.ApplicationProperties["operation"] = "put-token";
    request.ApplicationProperties["type"] = "servicebus.windows.net:sastoken";
    request.ApplicationProperties["name"] = Fx.Format("amqp://{0}/{1}", sbNamespace, entity);
    cbsSender.Send(request);

    // receive the response
    var response = cbsReceiver.Receive();
    if (response == null || response.Properties == null || response.ApplicationProperties == null)
    {
        result = false;
    }
    else
    {
        int statusCode = (int)response.ApplicationProperties["status-code"];
        if (statusCode != (int)HttpStatusCode.Accepted && statusCode != (int)HttpStatusCode.OK)
        {
            result = false;
        }
    }

    // the sender/receiver may be kept open for refreshing tokens
    cbsSender.Close();
    cbsReceiver.Close();
    session.Close();

    return result;
}
```

El método `PutCbsToken()` recibe la *conexión* (instancia de clase de conexión de AMQP, proporcionada por la [biblioteca .NET Lite de AMQP](https://github.com/Azure/amqpnetlite)) que representa la conexión TCP al servicio y el parámetro *sasToken* que es el token de SAS que se va a enviar.

> [!NOTE]
> Es importante que la conexión se cree con el **mecanismo de autenticación SASL establecido en ANONYMOUS** (y no con el valor PLAIN predeterminado con el nombre de usuario y la contraseña usados cuando no tiene que enviar el token de SAS).
>
>

A continuación, el publicador crea dos vínculos de AMQP para el envío del token de SAS y la recepción de la respuesta (resultado de validación del token) del servicio.

El mensaje de AMQP contiene una serie de propiedades y más información que un mensaje simple. El token de SAS es el cuerpo del mensaje (mediante su constructor). La propiedad **"ReplyTo"** se establece en el nombre del nodo para recibir el resultado de la validación en el vínculo del receptor (puede cambiar su nombre si lo desea y el servicio lo creará dinámicamente). El servicio usa las últimas tres propiedades personalizadas/de aplicación para indicar qué tipo de operación tiene que ejecutar. Como describe la especificación del borrador de CBS, deben ser el **nombre de la operación** ("put-token"), el **tipo de token** (en este caso, `servicebus.windows.net:sastoken`) y el **"nombre" de la audiencia** a la que se aplica el token (toda la entidad).

Después de enviar el token de SAS en el vínculo del remitente, el publicador debe leer la respuesta en el vínculo del receptor. La respuesta es un mensaje de AMQP simple con una propiedad de la aplicación denominada **"status-code"** que puede contener los mismos valores que un código de estado HTTP.

## <a name="rights-required-for-service-bus-operations"></a>Derechos necesarios para realizar operaciones de Service Bus

La siguiente tabla muestra los derechos de acceso necesarios para realizar diversas operaciones en recursos de Service Bus.

| Operación | Solicitud necesaria | Ámbito de solicitud |
| --- | --- | --- |
| **Espacio de nombres** | | |
| Configurar la regla de autorización en un espacio de nombres |Administración |Cualquier dirección de espacio de nombres |
| **Registro del servicio** | | |
| Enumerar directivas privadas |Administración |Cualquier dirección de espacio de nombres |
| Empezar a escuchar en un espacio de nombres |Escuchar |Cualquier dirección de espacio de nombres |
| Enviar mensajes a un agente de escucha en un espacio de nombres |Envío |Cualquier dirección de espacio de nombres |
| **Cola** | | |
| Creación de una cola |Administración |Cualquier dirección de espacio de nombres |
| Eliminación de una cola |Administración |Cualquier dirección de cola válida |
| Enumerar colas |Administración |/$Resources/Queues |
| Obtener la descripción de la cola |Administración |Cualquier dirección de cola válida |
| Configurar la regla de autorización para una cola |Administración |Cualquier dirección de cola válida |
| Enviar a la cola |Envío |Cualquier dirección de cola válida |
| mensajes de una cola |Escuchar |Cualquier dirección de cola válida |
| Abandone o complete los mensajes después de recibir el mensaje en el modo de bloqueo de información |Escuchar |Cualquier dirección de cola válida |
| Aplazar un mensaje para su recuperación posterior |Escuchar |Cualquier dirección de cola válida |
| Mensaje fallido |Escuchar |Cualquier dirección de cola válida |
| Obtener el estado asociado a una sesión de cola de mensajes |Escuchar |Cualquier dirección de cola válida |
| Establecer el estado asociado a una sesión de cola de mensajes |Escuchar |Cualquier dirección de cola válida |
| Programe un mensaje para entregarlo más tarde; por ejemplo, [ScheduleMessageAsync()](/dotnet/api/microsoft.azure.servicebus.queueclient.schedulemessageasync#Microsoft_Azure_ServiceBus_QueueClient_ScheduleMessageAsync_Microsoft_Azure_ServiceBus_Message_System_DateTimeOffset_) |Escuchar | Cualquier dirección de cola válida
| **Tema.** | | |
| de un tema |Administración |Cualquier dirección de espacio de nombres |
| Eliminación de un tema |Administración |Cualquier dirección de tema válida |
| Enumerar temas |Administración |/$Resources/Topics |
| Obtener la descripción del tema |Administración |Cualquier dirección de tema válida |
| Configurar la regla de autorización para un tema |Administración |Cualquier dirección de tema válida |
| Enviar al tema |Envío |Cualquier dirección de tema válida |
| **Suscripción** | | |
| una suscripción |Administración |Cualquier dirección de espacio de nombres |
| Eliminar suscripción |Administración |../myTopic/Subscriptions/mySubscription |
| Enumerar suscripciones |Administración |../myTopic/Subscriptions |
| Obtener la descripción de la suscripción |Administración |../myTopic/Subscriptions/mySubscription |
| Abandone o complete los mensajes después de recibir el mensaje en el modo de bloqueo de información |Escuchar |../myTopic/Subscriptions/mySubscription |
| Aplazar un mensaje para su recuperación posterior |Escuchar |../myTopic/Subscriptions/mySubscription |
| Mensaje fallido |Escuchar |../myTopic/Subscriptions/mySubscription |
| Obtener el estado asociado a una sesión de tema |Escuchar |../myTopic/Subscriptions/mySubscription |
| Establecer el estado asociado a una sesión de tema |Escuchar |../myTopic/Subscriptions/mySubscription |
| **Reglas** | | |
| Crear una regla |Administración |../myTopic/Subscriptions/mySubscription |
| Eliminar una regla |Administración |../myTopic/Subscriptions/mySubscription |
| Enumerar reglas |Administrar o escuchar |../myTopic/Subscriptions/mySubscription/Rules

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre la mensajería de Service Bus, consulte los siguientes temas:

* [Colas, temas y suscripciones de Service Bus](service-bus-queues-topics-subscriptions.md)
* [Utilización de las colas de Service Bus](service-bus-dotnet-get-started-with-queues.md)
* [Uso de temas y suscripciones de Service Bus](service-bus-dotnet-how-to-use-topics-subscriptions.md)

[Azure portal]: https://portal.azure.com