---
title: Guía del protocolo AMQP 1.0 en Azure Service Bus y Event Hubs | Microsoft Docs
description: Guía del protocolo para expresiones y la descripción de AMQP 1.0 en Azure Service Bus y Event Hubs
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 5957e2d36b57be7db1af279736e8859d1a69b66b
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86511320"
---
# <a name="amqp-10-in-azure-service-bus-and-event-hubs-protocol-guide"></a>Guía del protocolo AMQP 1.0 Azure Service Bus y Event Hubs

Advanced Message Queueing Protocol 1.0 es un protocolo de tramas y transferencia estandarizado para transferir mensajes de forma asincrónica, segura y confiable entre dos partes. Es el principal protocolo de Azure Service Bus Messaging y Azure Event Hubs. Ambos servicios también admiten HTTPS. El protocolo SBMP propietario, que también se admite, está desapareciendo en favor de AMQP.

AMQP 1.0 es el resultado de una amplia colaboración del sector que reunió a proveedores de middleware, como Microsoft y Red Hat, con muchos usuarios de middleware de mensajería, como JP Morgan Chase, que representa al sector de los servicios financieros. El foro de normalización técnica para las especificaciones del protocolo AMQP y la extensión es OASIS, y ha logrado la aprobación formal como estándar internacional como ISO/IEC 19494.

## <a name="goals"></a>Objetivos

Este artículo resume brevemente los conceptos básicos de la especificación de mensajería AMQP 1.0, junto con un pequeño conjunto de borradores de especificaciones de la extensión que actualmente se está finalizando en el comité técnico de OASIS para AMQP; también explica cómo Azure Service Bus implementa y crea basándose en estas especificaciones.

El objetivo es que cualquier desarrollador que usa una pila de cliente de AMQP 1.0 existente en cualquier plataforma pueda interactuar con Azure Service Bus mediante AMQP 1.0.

Las pilas de propósito general de AMQP 1.0 comunes, como Apache Proton o AMQP.NET Lite, ya implementan todos los protocolos importantes de AMQP 1.0. Los gestos fundacionales a veces se encapsulan con una API de mayor nivel; Apache Proton incluso ofrece dos: la API de Messenger imperativa y la API de Reactor reactiva.

En la siguiente sección, se supone que la administración de conexiones, sesiones y vínculos de AMQP, y la administración de las transferencias de tramas y el control de flujo se tratan mediante la pila correspondiente (por ejemplo, Apache Proton-C) y no requieren demasiada atención específica de los desarrolladores de aplicaciones, o ninguna atención. Suponemos de forma abstracta la existencia de unas primitivas de API, como la capacidad de conectarse y de crear algún tipo de objetos de abstracción *remitente* y *receptor*, que luego tienen alguna forma de operaciones `send()` y `receive()`, respectivamente.

Cuando se habla de las funcionalidades avanzadas de Azure Service Bus, como la consulta de mensajes o la administración de sesiones, estas características se explican en relación a AMQP, pero también como una pseudoimplementación superpuesta sobre esta abstracción de API supuesta.

## <a name="what-is-amqp"></a>¿Qué es AMQP?

AMQP es un protocolo de tramas y transferencia. Las tramas significa que proporciona una estructura para los flujos de datos binarios que fluyan en ambas direcciones de una conexión de red. La estructura proporciona la delineación de bloques de datos distintivos *tramas* que se intercambiarán entre las partes conectadas. Las funcionalidades de transferencia se aseguran de que ambas partes de la comunicación pueden establecer un conocimiento compartido acerca de cuándo se transferirán las tramas y cuándo se considerarán completadas las transferencias.

A diferencia de los anteriores borradores expirados producidos por el grupo de trabajo de AMQP que todavía usan algunos agentes de mensajes, el protocolo AMQP 1.0 estandarizado y final del grupo de trabajo no prescribe la presencia de un agente de mensajes o de ninguna topología concreta para las entidades dentro de un agente de mensajes.

Puede usar el protocolo para la comunicación punto a punto simétrica, para la interacción con los agentes de mensajes que admiten las colas y las entidades de publicación/suscripción, como hace Azure Service Bus. También se puede utilizar para la interacción con la infraestructura de mensajería cuando los patrones de interacción son distintos de las colas regulares, como sucede con Azure Event Hubs. Un Centro de eventos actúa como una cola cuando se le envían eventos, pero actúa más como un servicio de almacenamiento en serie cuando se leen los eventos desde él; en cierto modo, se parece a una unidad de cinta. El cliente elige un desplazamiento en el flujo de datos disponible y, después, atiende todos los eventos desde ese desplazamiento hasta el más reciente disponible.

El protocolo AMQP 1.0 está diseñado para ser extensible, lo que permite que las especificaciones mejoren sus funcionalidades. Las tres especificaciones de la extensión que tratamos en este documento lo muestran. Para la comunicación a través de la infraestructura existente de HTTPS/WebSockets, la configuración de los puertos TCP de AMQP nativos puede ser difícil. Una especificación de enlace define cómo superponer AMQP a WebSockets. Para interactuar con la infraestructura de mensajería en forma de solicitud/respuesta para fines de administración o para proporcionar una funcionalidad avanzada, la especificación de Administración de AMQP define las primitivas de interacción básica necesarias. Para la integración del modelo de autorización federada, la especificación de seguridad basada en notificaciones de AMQP define cómo asociar y renovar los tokens de autorización asociados a los vínculos.

## <a name="basic-amqp-scenarios"></a>Escenarios básicos de AMQP

En esta sección se explica el uso básico de AMQP 1.0 con Azure Service Bus, lo que incluye la creación de conexiones, sesiones y vínculos, así como la transferencia de mensajes tanto a las entidades de Services Bus (colas, temas y suscripciones) como desde ellas.

La fuente con mayor autoridad para aprender cómo funciona AMQP es la especificación AMQP 1.0, pero se escribió para guiar de forma precisa la implementación, no para enseñar el protocolo. Esta sección se centra en la presentación de tanta terminología como sea necesario para describir el modo en que Service Bus usa AMQP 1.0. Para una introducción más completa a AMQP, así como una explicación más amplia de AMQP 1.0, puede consultar [este curso en vídeo][this video course].

### <a name="connections-and-sessions"></a>Conexiones y sesiones

AMQP llama a los programas de comunicación *contenedores*, que contienen *nodos*, unas entidades que se comunican dentro de estos contenedores. Una cola puede ser uno de esos nodos. AMQP permite la multiplexación, por lo que se puede usar una sola conexión para muchas rutas de comunicación entre los nodos; por ejemplo, un cliente de aplicación puede recibir al mismo tiempo desde una cola y enviar a otra cola a través de la misma conexión de red.

![Diagrama que muestra las sesiones y conexiones entre los conectores.][1]

Por lo tanto, la conexión de red está anclada en el contenedor. El contenedor la inicia en el rol de cliente, realizando una conexión de socket TCP saliente a un contenedor en el rol de receptor, que escucha y acepta las conexiones TCP entrantes. El protocolo de enlace de conexión incluye negociar la versión del protocolo, declarar o negociar el uso de la seguridad de nivel de transporte (TLS/SSL) y un protocolo de enlace de autenticación y autorización en el ámbito de la conexión que se basa en SASL.

Azure Service Bus requiere el uso de TLS en todo momento. Admite conexiones a través del puerto TCP 5671, donde la conexión TCP primero se superpone con TLS antes de entrar en el protocolo de enlace del protocolo AMQP y también admite conexiones a través del puerto TCP 5672, donde el servidor ofrece inmediatamente una actualización obligatoria de la conexión a TLS con el modelo prescrito por AMQP. El enlace de WebSockets de AMQP crea un túnel a través del puerto TCP 443 que es equivalente a las conexiones 5671 de AMQP.

Después de configurar la conexión y TLS, Service Bus ofrece dos opciones de mecanismo SASL:

* Normalmente, SASL PLAIN se utiliza para pasar las credenciales de usuario y la contraseña a un servidor. Service Bus no tiene cuentas, sino [reglas de seguridad de acceso compartido](service-bus-sas.md) con nombre, que confieren derechos y están asociadas a una clave. El nombre de una regla se usa como nombre de usuario y la clave (como texto codificado con base64) se utiliza como contraseña. Los derechos asociados a la regla elegida rigen las operaciones permitidas en la conexión.
* SASL ANONYMOUS se utiliza para omitir la autorización de SASL cuando el cliente desea usar el modelo de seguridad basada en notificaciones (CBS), que se describe más adelante. Con esta opción, se puede establecer una conexión de cliente de forma anónima por un breve período, durante el cual el cliente solo puede interactuar con el punto de conexión CBS y se debe completar el protocolo de enlace CBS.

Una vez establecida la conexión de transporte, cada contenedor declara el tamaño máximo de trama que desea controlar y después de qué tiempo de espera inactivo se desconectará unilateralmente si no hay ninguna actividad en la conexión.

También declaran cuántos canales simultáneos se admiten. Un canal es una ruta de transferencia unidireccional, saliente y virtual sobre la conexión. Una sesión toma un canal de cada uno de los contenedores interconectados para formar una ruta de comunicación bidireccional.

Las sesiones tienen un modelo de control de flujo basado en ventanas; cuando se crea una sesión, cada parte declara el número de tramas que está dispuesto a aceptar en su ventana de recepción. A medida que las partes intercambian tramas, las tramas transferidas llenan esa ventana y se detienen cuando la ventana está llena, y hasta que la ventana se restablece o se expande mediante el *performativo del flujo* (*performativo* es el término de AMQP para los gestos en el nivel de protocolo intercambiados entre las dos partes).

Este modelo basado en ventanas es parecido al concepto de control de flujo basado en ventanas de TCP, pero en el nivel de sesión dentro del socket. El concepto del protocolo de permitir que haya varias sesiones simultáneas existe para que el tráfico de alta prioridad pueda adelantar al tráfico normal limitado, como en un carril rápido de una autopista.

En la actualidad, Azure Service Bus utiliza exactamente una sesión para cada conexión. El tamaño de trama máximo de Service Bus es 262 144 bytes (256 KB) para Service Bus estándar y Event Hubs. Para Service Bus Premium es 1 048 576 (1 MB). Service Bus no impone ninguna ventana específica de limitación en el nivel de sesión, pero restablece la ventana periódicamente como parte del control de flujo en el nivel de vínculo (consulte [la siguiente sección](#links)).

Las conexiones, los canales y las sesiones son efímeros. Si la conexión subyacente se contrae, es necesario restablecer las conexiones, el túnel TLS, el contexto de autorización SASL y las sesiones.

### <a name="amqp-outbound-port-requirements"></a>Requisitos de puertos de salida de AMQP

Los clientes que usan conexiones AMQP a través de TCP requieren que se abran los puertos 5671 y 5672 en el firewall local. Junto con estos puertos, podría ser necesario abrir puertos adicionales si está habilitada la característica [EnableLinkRedirect](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.amqp.amqptransportsettings.enablelinkredirect?view=azure-dotnet). `EnableLinkRedirect` es una nueva característica de mensajería que ayuda a omitir un salto al recibir mensajes, lo que ayuda a mejorar el rendimiento. El cliente comenzará a comunicarse directamente con el servicio back-end a través del intervalo de puertos 104XX como se muestra en la siguiente imagen. 

![Lista de puertos de destino][4]

Un cliente de .NET producirá un error SocketException ("Intento de obtener acceso a un socket de una manera no permitida por los permisos de acceso") si el firewall bloquea estos puertos. La característica se puede deshabilitar estableciendo `EnableAmqpLinkRedirect=false` en la cadena de conexión, lo que obliga a los clientes a comunicarse con el servicio remoto a través del puerto 5671.


### <a name="links"></a>Vínculos

AMQP transfiere los mensajes a través de vínculos. Un vínculo es una ruta de comunicación creada en una sesión que permite transferir mensajes en un sentido; la negociación de estado de la transferencia es a través del vínculo y bidireccional entre las partes conectadas.

![Captura de pantalla que muestra una sesión que ha establecido una conexión de vínculo entre dos contenedores.][2]

Cualquier contenedor pueden crear vínculos en cualquier momento en una sesión existente, lo que hace que AMQP sea diferente de muchos otros protocolos, incluidos HTTP y MQTT, donde la iniciación de las transferencias y la ruta de la transferencia es un privilegio exclusivo de la parte que se crea la conexión de socket.

El contenedor que inicia el vínculo pide al contenedor opuesto que acepte un vínculo y elije un el rol de remitente o receptor. Por lo tanto, cualquier contenedor puede iniciar la creación de rutas de comunicación unidireccionales o bidireccionales, con esta última modelada como pares de vínculos.

Se asigna nombre a los vínculos y se asocian a los nodos. Como se indicó al principio, los nodos son las entidades que se comunican dentro de un contenedor.

En Service Bus, un nodo es directamente equivalente a una cola, un tema, una suscripción o una subcola de mensajes fallidos de una cola o suscripción. Por lo tanto, el nombre de nodo utilizado en AMQP es el nombre relativo de la entidad dentro del espacio de nombres de Service Bus. Si una cola se denomina `myqueue`, ese es también su nombre de nodo de AMQP. Una suscripción de un tema sigue la convención de API HTTP, ya que se ordena en una colección de recursos de "suscripciones" y, por consiguiente, una suscripción **sub** en un tema **mytopic** tiene el nombre de nodo de AMQP **mytopic/subscriptions/sub**.

El cliente que se conecta también debe usar un nombre de nodo local para crear los vínculos; Service Bus no es preceptivo acerca de esos nombres de nodo y no los interpreta. Normalmente, las pilas de cliente de AMQP 1.0 utilizan un esquema para asegurarse de que estos nombres de nodo efímero son únicos en el ámbito del cliente.

### <a name="transfers"></a>Transferencias

Una vez establecido un vínculo, los mensajes se pueden transferir a través de ese vínculo. En AMQP, se realiza una transferencia con un gesto de protocolo explícito (el performativo *transfer*) que mueve un mensaje del remitente al receptor a través de un vínculo. Una transferencia está completa cuando que se "determina", lo que significa que ambas partes han establecido una conocimiento compartido del resultado de esa transferencia.

![Diagrama que muestra la transferencia de un mensaje entre el remitente y el receptor y la disposición resultante.][3]

En el caso más simple, el remitente puede enviar mensajes "previamente determinados", lo que significa que el cliente no está interesado en el resultado y el receptor no proporciona ningún comentario sobre el resultado de la operación. Este modo es compatible con Service Bus en el nivel del protocolo AMQP, pero no se expone en ninguna de las API de cliente.

Lo normal es que los mensajes se envíen sin determinar y el receptor indique entonces la aceptación o el rechazo mediante el performativo *disposition*. Se produce un rechazo cuando el receptor no puede aceptar el mensaje por alguna razón y el mensaje de rechazo contiene información sobre el motivo, que es una estructura de error definida por AMQP. Si se rechazan los mensajes debido a errores internos dentro de Service Bus, el servicio devuelve información adicional dentro de esa estructura que puede utilizarse para proporcionar sugerencias de diagnóstico al personal de soporte si está registrando solicitudes de soporte técnico. Obtendrá más detalles sobre los errores más adelante.

Una forma especial de rechazo es el estado *publicado*, que indica que el receptor no tiene ninguna objeción técnica a la transferencia, pero tampoco tiene ningún interés en determinarla. Ese es el caso, por ejemplo, cuando un mensaje se entrega a un cliente de Service Bus y el cliente elige "abandonar" el mensaje porque no puede realizar el trabajo resultante de procesar el mensaje, mientras que la entrega del mensaje en sí no tiene ningún error. Una variación de este estado es el estado *modificado*, que permite realizar cambios en el mensaje cuando se envía. En la actualidad, Service Bus no utiliza ese estado.

La especificación AMQP 1.0 define otro estado de disposición, denominado *recibido*, que ayuda específicamente a controlar la recuperación de vínculos. La recuperación de vínculos permite reconstituir el estado de un vínculo y las entregas pendientes sobre una nueva conexión y sesión, cuando se pierden la conexión y sesión anteriores.

Service Bus no admite la recuperación de vínculos; si el cliente pierde la conexión a Service Bus con una transferencia de mensaje sin determinar pendiente, dicha transferencia se pierde y el cliente debe volver a conectarse, restablecer el vínculo e intentar de nuevo la transferencia.

Como tal, Service Bus y Event Hubs admiten transferencias "al menos una vez", donde el remitente puede estar seguro de que el mensaje se ha almacenado y aceptado, pero no admite transferencias "exactamente una vez" en el nivel de AMQP, donde el sistema intentará recuperar el vínculo y continuar negociando el estado de entrega para evitar la duplicación de la transferencia del mensaje.

Para compensar posibles envíos duplicados, Service Bus admite la detección de duplicados como una característica opcional en colas y temas. La detección de duplicados registra los identificadores de todos los mensajes entrantes durante un período definido por el usuario y después elimina silenciosamente todos los mensajes enviados con los mismos identificadores de mensaje durante esa misma ventana.

### <a name="flow-control"></a>Control de flujo

Además del modelo de control de flujo en el nivel de sesión que se ha tratado anteriormente, cada vínculo tiene su propio modelo de control de flujo. El control de flujo en el nivel de sesión protege el contenedor para que no tenga que controlar muchas tramas de una vez; el control de flujo de nivel de vínculo pone la aplicación a cargo de cuántos mensajes desea controlar desde un vínculo y cuándo.

![Captura de pantalla de un registro que muestra los campos origen, destino, puerto de origen, puerto de destino y nombre del protocolo. En la primera fila, el puerto de destino 10401 (0x28 A 1) está marcado en negro.][4]

En un vínculo, las transferencias solo se pueden producir si el remitente tiene suficiente *crédito del vínculo*. El crédito del vínculo es un contador establecido por el receptor con el performativo *flow*, que tiene un ámbito en un vínculo. Cuando el remitente tiene crédito del vínculo asignado, intenta utilizar ese crédito con la entrega de mensajes. Cada entrega de mensajes reduce en uno el crédito del vínculo restante. Cuando se agota el crédito del vínculo, las entregas se detienen.

Cuando Service Bus está en el rol de receptor, proporciona instantáneamente al remitente el crédito del vínculo suficiente para que los mensajes puedan enviarse de inmediato. Puesto que se utiliza el crédito del vínculo, Service Bus envía ocasionalmente un performativo *flow* al remitente para actualizar el saldo del crédito del vínculo.

En el rol de remitente, Service Bus envía mensajes para utilizar todo el crédito del vínculo pendiente.

Una llamada "recibir" en el nivel de API se traduce en un performativo *flow* enviado a Service Bus por el cliente; Service Bus usa ese crédito tomando el primer mensaje desbloqueado y disponible de la cola, bloqueándolo y transfiriéndolo. Si no hay ningún mensaje disponible para la entrega, todo el crédito pendiente por cualquier vínculo establecido con esa entidad concreta permanece registrado en orden de llegada, y los mensajes se bloquean y transfieren a medida que estén disponibles para usar el crédito pendiente.

Se libera el bloqueo de un mensaje cuando la transferencia se determina en uno de los estados terminales *aceptado*, *rechazado* o *publicado*. El mensaje se quita de Service Bus cuando el estado terminal *aceptado*. Permanece en Service Bus y se entrega al siguiente receptor cuando la transferencia alcance cualquiera de los otros estados. Service Bus pasa automáticamente el mensaje a la cola de mensajes fallidos de la entidad al alcanzar el número máximo de entregas permitido para la entidad debido a rechazos o lanzamientos repetidos.

Aunque las API de Service Bus no exponen directamente dicha opción en la actualidad, un cliente del protocolo AMQP de nivel inferior puede utilizar el modelo de crédito del vínculo para convertir la interacción de "estilo de extracción", que emite una unidad de crédito para cada solicitud de recepción, en un modelo de "estilo de inserción" al emitir un gran número de créditos del vínculo y, a continuación, recibir los mensajes cuando estén disponibles sin intervención adicional. Se admite la inserción mediante la configuración de las propiedades [MessagingFactory.PrefetchCount](/dotnet/api/microsoft.servicebus.messaging.messagingfactory) o [MessageReceiver.PrefetchCount](/dotnet/api/microsoft.servicebus.messaging.messagereceiver). Si son distintas de cero, el cliente de AMQP las usa como crédito del vínculo.

En este contexto, es importante comprender que el reloj de la expiración del bloqueo en el mensaje dentro de la entidad se inicia cuando el mensaje se toma de la entidad, no cuando se coloca en la transferencia. Cada vez que el cliente indica que está preparado para recibir mensajes mediante la emisión de crédito del vínculo, se espera que extraiga activamente los mensajes a través de la red y que esté preparado para controlarlos. De lo contrario, el bloqueo del mensaje puede haber expirado incluso antes de que el mensaje se entregue. El uso del control de flujo del crédito del vínculo debe reflejar directamente la disponibilidad inmediata para tratar con mensajes disponibles enviados al receptor.

En resumen, las secciones siguientes proporcionan una introducción esquemática del flujo de performativos durante diferentes interacciones de API. Cada sección describe una operación lógica diferente. Algunas de esas interacciones pueden ser "perezosas", lo que significa que solo pueden realizarse cuando se solicitan. La creación de un remitente del mensaje puede no provocar una interacción de la red hasta que se envía o solicita el primer mensaje.

Las flechas de la tabla siguiente muestran la dirección de flujo de performativos.

#### <a name="create-message-receiver"></a>Creación del receptor del mensaje

| Remoto | Azure Service Bus |
| --- | --- |
| --> attach(<br/>name={link name},<br/>handle={numeric handle},<br/>role=**receiver**,<br/>source={entity name},<br/>target={client link ID}<br/>) |El cliente se adjunta a la entidad como receptor |
| Service Bus responde asociando su extremo del vínculo |<-- attach(<br/>name={link name},<br/>handle={numeric handle},<br/>role=**sender**,<br/>source={entity name},<br/>target={client link ID}<br/>) |

#### <a name="create-message-sender"></a>Creación del remitente del mensaje

| Remoto | Azure Service Bus |
| --- | --- |
| --> attach(<br/>name={link name},<br/>handle={numeric handle},<br/>role=**sender**,<br/>source={client link ID},<br/>target={entity name}<br/>) |Ninguna acción |
| Ninguna acción |<-- attach(<br/>name={link name},<br/>handle={numeric handle},<br/>role=**receiver**,<br/>source={client link ID},<br/>target={entity name}<br/>) |

#### <a name="create-message-sender-error"></a>Creación del remitente del mensaje (error)

| Remoto | Azure Service Bus |
| --- | --- |
| --> attach(<br/>name={link name},<br/>handle={numeric handle},<br/>role=**sender**,<br/>source={client link ID},<br/>target={entity name}<br/>) |Ninguna acción |
| Ninguna acción |<-- attach(<br/>name={link name},<br/>handle={numeric handle},<br/>role=**receiver**,<br/>source=null,<br/>target=null<br/>)<br/><br/><-- detach(<br/>handle={numeric handle},<br/>closed=**true**,<br/>error={error info}<br/>) |

#### <a name="close-message-receiversender"></a>Cierre del remitente/receptor del mensaje

| Remoto | Azure Service Bus |
| --- | --- |
| --> detach(<br/>handle={numeric handle},<br/>closed=**true**<br/>) |Ninguna acción |
| Ninguna acción |<-- detach(<br/>handle={numeric handle},<br/>closed=**true**<br/>) |

#### <a name="send-success"></a>Envío (correcto)

| Remoto | Azure Service Bus |
| --- | --- |
| --> transfer(<br/>delivery-id={numeric handle},<br/>delivery-tag={binary handle},<br/>settled=**false**,,more=**false**,<br/>state=**null**,<br/>resume=**false**<br/>) |Ninguna acción |
| Ninguna acción |<-- disposition(<br/>role=receiver,<br/>first={delivery ID},<br/>last={delivery ID},<br/>settled=**true**,<br/>state=**accepted**<br/>) |

#### <a name="send-error"></a>Envío (error)

| Remoto | Azure Service Bus |
| --- | --- |
| --> transfer(<br/>delivery-id={numeric handle},<br/>delivery-tag={binary handle},<br/>settled=**false**,,more=**false**,<br/>state=**null**,<br/>resume=**false**<br/>) |Ninguna acción |
| Ninguna acción |<-- disposition(<br/>role=receiver,<br/>first={delivery ID},<br/>last={delivery ID},<br/>settled=**true**,<br/>state=**rejected**(<br/>error={error info}<br/>)<br/>) |

#### <a name="receive"></a>Recepción

| Remoto | Azure Service Bus |
| --- | --- |
| --> flow(<br/>link-credit=1<br/>) |Ninguna acción |
| Ninguna acción |< transfer(<br/>delivery-id={numeric handle},<br/>delivery-tag={binary handle},<br/>settled=**false**,<br/>more=**false**,<br/>state=**null**,<br/>resume=**false**<br/>) |
| --> disposition(<br/>role=**receiver**,<br/>first={delivery ID},<br/>last={delivery ID},<br/>settled=**true**,<br/>state=**accepted**<br/>) |Ninguna acción |

#### <a name="multi-message-receive"></a>Recepción de múltiples mensajes

| Remoto | Azure Service Bus |
| --- | --- |
| --> flow(<br/>link-credit=3<br/>) |Ninguna acción |
| Ninguna acción |< transfer(<br/>delivery-id={numeric handle},<br/>delivery-tag={binary handle},<br/>settled=**false**,<br/>more=**false**,<br/>state=**null**,<br/>resume=**false**<br/>) |
| Ninguna acción |< transfer(<br/>delivery-id={numeric handle+1},<br/>delivery-tag={binary handle},<br/>settled=**false**,<br/>more=**false**,<br/>state=**null**,<br/>resume=**false**<br/>) |
| Ninguna acción |< transfer(<br/>delivery-id={numeric handle+2},<br/>delivery-tag={binary handle},<br/>settled=**false**,<br/>more=**false**,<br/>state=**null**,<br/>resume=**false**<br/>) |
| --> disposition(<br/>role=receiver,<br/>first={delivery ID},<br/>last={delivery ID+2},<br/>settled=**true**,<br/>state=**accepted**<br/>) |Ninguna acción |

### <a name="messages"></a>error de Hadoop

En las siguientes secciones se explican las propiedades de las secciones de mensajes de AMQP estándar utilizadas por Service Bus y cómo se asignan al conjunto de API de Service Bus.

Cualquier propiedad que la aplicación necesite definir debe asignarse al valor `application-properties` de AMQP.

#### <a name="header"></a>encabezado

| Nombre del campo | Uso | Nombre de la API |
| --- | --- | --- |
| durable |- |- |
| priority |- |- |
| ttl |Período de vida para este mensaje |[TimeToLive](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage) |
| first-acquirer |- |- |
| delivery-count |- |[DeliveryCount](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage) |

#### <a name="properties"></a>properties

| Nombre del campo | Uso | Nombre de la API |
| --- | --- | --- |
| message-id |Identificador de formato libre definido por la aplicación para este mensaje. Se usa para la detección de duplicados. |[MessageId](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage) |
| user-id |Identificador del usuario definido por la aplicación; no interpretado por Service Bus. |No es accesible a través de la API de Service Bus. |
| to |Identificador del destino definido por la aplicación; no interpretado por Service Bus. |[To](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage) |
| subject |Identificador del propósito de mensaje definido por la aplicación; no interpretado por Service Bus. |[Label](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage) |
| reply-to |Indicador de la ruta de respuesta definido por la aplicación; no interpretado por Service Bus. |[ReplyTo](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage) |
| correlation-id |Identificador de la correlación definido por la aplicación; no interpretado por Service Bus. |[CorrelationId](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage) |
| content-type |Indicador del tipo de contenido definido por la aplicación para el cuerpo; no interpretado por Service Bus. |[ContentType](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage) |
| content-encoding |Indicador de la codificación de contenido definido por la aplicación para el cuerpo; no interpretado por Service Bus. |No es accesible a través de la API de Service Bus. |
| absolute-expiry-time |Declara en qué instante absoluto expira el mensaje. Se ignora en la entrada (se observa el TTL de encabezado), es autoritativo en la salida. |[ExpiresAtUtc](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage) |
| creation-time |Declara en qué momento creó el mensaje. No usado por Service Bus |No es accesible a través de la API de Service Bus. |
| group-id |Identificador definido por la aplicación para un conjunto de mensajes relacionado. Se utiliza para sesiones de Service Bus. |[SessionId](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage) |
| group-sequence |Contador que identifica el número de secuencia relativa del mensaje dentro de una sesión. Omitido por Service Bus. |No es accesible a través de la API de Service Bus. |
| reply-to-group-id |- |[ReplyToSessionId](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage) |

#### <a name="message-annotations"></a>Anotaciones del mensaje

Hay algunas otras propiedades de mensaje de Service Bus, que no forman parte de las propiedades de mensaje de AMQP, que se transmiten como `MessageAnnotations` en el mensaje.

| Clave de asignación de anotaciones | Uso | Nombre de la API |
| --- | --- | --- |
| x-opt-scheduled-enqueue-time | Indica en qué momento debe aparecer el mensaje en la entidad. |[ScheduledEnqueueTime](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.scheduledenqueuetimeutc?view=azure-dotnet) |
| x-opt-partition-key | Clave que define la aplicación que determina a qué partición debe llegar el mensaje. | [PartitionKey](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.partitionkey?view=azure-dotnet) |
| x-opt-via-partition-key | Valor de clave de partición que define la aplicación cuando se va a utilizar una transacción para enviar mensajes a través de una cola de transferencias. | [ViaPartitionKey](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.viapartitionkey?view=azure-dotnet) |
| x-opt-enqueued-time | Hora UTC que define el servicio que representa la hora real al poner en cola el mensaje. Se omite en la entrada. | [EnqueuedTimeUtc](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.enqueuedtimeutc?view=azure-dotnet) |
| x-opt-sequence-number | Número único que define el servicio asignado a un mensaje. | [SequenceNumber](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.sequencenumber?view=azure-dotnet) |
| x-opt-offset | Número de secuencia en cola que define el servicio y que pertenece al mensaje. | [EnqueuedSequenceNumber](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.enqueuedsequencenumber?view=azure-dotnet) |
| x-opt-locked-until | El servicio se encarga de definir este valor. Es la fecha y la hora hasta las cuales se bloqueará el mensaje en la cola o la suscripción. | [LockedUntilUtc](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.lockeduntilutc?view=azure-dotnet) |
| x-opt-deadletter-source | El servicio se encarga de definir este valor. Si el mensaje se recibe de la cola de mensajes fallidos, este es la fuente del mensaje original. | [DeadLetterSource](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.deadlettersource?view=azure-dotnet) |

### <a name="transaction-capability"></a>Capacidad de transacciones

Una transacción agrupa dos o más operaciones en un ámbito de ejecución. Por naturaleza, una transacción de este tipo debe garantizar que todas las operaciones que pertenecen a un grupo determinado de operaciones tendrán éxito o darán error de forma conjunta.
Las operaciones se agrupan según un identificador `txn-id`.

Para permitir la interacción transaccional, el cliente actúa como un `transaction controller` que controla las operaciones que deben agruparse. El servicio de Bus Service actúa como un `transactional resource` y funciona tal como solicitó el `transaction controller`.

El cliente y el servicio se comunican a través de un `control link` establecido por el cliente. El controlador envía los mensajes `declare` y `discharge` a través del vínculo de control para asignar y completar las transacciones respectivamente (no representan la demarcación del trabajo transaccional). El envío o la recepción real no se realiza en este vínculo. Cada operación transaccional solicitada se identifica explícitamente con el `txn-id` deseado y, por lo tanto, puede aparecer en cualquier vínculo de la conexión. Si el vínculo de control se cierra mientras siguen presentes las transacciones no descargadas que este creó, todas esas transacciones se revierten inmediatamente, y los intentos de llevar a cabo más trabajo transaccional en ellas darán lugar a un error. Los mensajes en el vínculo de control no deben estar preestablecidos.

Cada conexión debe iniciar su propio vínculo de control para poder iniciar y finalizar transacciones. El servicio define un objetivo especial que funciona como un `coordinator`. El cliente o el controlador establece un vínculo de control para este objetivo. El vínculo de control está fuera del límite de una entidad, es decir, el mismo vínculo de control se puede usar para iniciar y descargar transacciones de varias entidades.

#### <a name="starting-a-transaction"></a>Iniciar una transacción

Para iniciar el trabajo transaccional. el controlador debe obtener un elemento `txn-id` desde el coordinador. Para ello, envía un mensaje de tipo `declare`. Si la declaración se realiza con éxito, el coordinador responde con un resultado de disposición, que lleva el elemento `txn-id` asignado.

| Cliente (controlador) | Dirección | Service Bus (coordinador) |
| :--- | :---: | :--- |
| attach(<br/>name={link name},<br/>... ,<br/>role=**sender**,<br/>target=**Coordinator**<br/>) | ------> |  |
|  | <------ | attach(<br/>name={link name},<br/>... ,<br/>target=Coordinator()<br/>) |
| transfer(<br/>delivery-id=0, ...)<br/>{ AmqpValue (**Declare()** )}| ------> |  |
|  | <------ | disposition( <br/> first=0, last=0, <br/>state=**Declared**(<br/>**txn-id**={transaction ID}<br/>))|

#### <a name="discharging-a-transaction"></a>Descargar una transacción

El controlador concluye el trabajo transaccional mediante el envío de un mensaje `discharge` al coordinador. El controlador indica que quiere confirmar o revertir el trabajo transaccional al establecer la marca `fail` en el cuerpo de la descarga. Si el coordinador no puede completar la descarga, se rechaza el mensaje con este resultado que lleva el valor `transaction-error`.

> Nota: El elemento fail=true indica la reversión de una transacción, y fail=false hace referencia a la confirmación.

| Cliente (controlador) | Dirección | Service Bus (coordinador) |
| :--- | :---: | :--- |
| transfer(<br/>delivery-id=0, ...)<br/>{ AmqpValue (Declare())}| ------> |  |
|  | <------ | disposition( <br/> first=0, last=0, <br/>state=Declared(<br/>txn-id={transaction ID}<br/>))|
| | . . . <br/>Trabajo transaccional<br/>en otros vínculos<br/> . . . |
| transfer(<br/>delivery-id=57, ...)<br/>{ AmqpValue (<br/>**Discharge(txn-id=0,<br/>fail=false)** )}| ------> |  |
| | <------ | disposition( <br/> first=57, last=57, <br/>state=**Accepted()** )|

#### <a name="sending-a-message-in-a-transaction"></a>Enviar un mensaje en una transacción

Todo el trabajo transaccional se realiza gracias al estado de la entrega transaccional `transactional-state` que transporta el valor de txn-id. En lo que respecta al envío de mensajes, el estado transaccional lo realiza el marco de transferencias del mensaje. 

| Cliente (controlador) | Dirección | Service Bus (coordinador) |
| :--- | :---: | :--- |
| transfer(<br/>delivery-id=0, ...)<br/>{ AmqpValue (Declare())}| ------> |  |
|  | <------ | disposition( <br/> first=0, last=0, <br/>state=Declared(<br/>txn-id={transaction ID}<br/>))|
| transfer(<br/>handle=1,<br/>delivery-id=1, <br/>**state=<br/>TransactionalState(<br/>txn-id=0)** )<br/>{ payload }| ------> |  |
| | <------ | disposition( <br/> first=1, last=1, <br/>state=**TransactionalState(<br/>txn-id=0,<br/>outcome=Accepted()** ))|

#### <a name="disposing-a-message-in-a-transaction"></a>Disponer de un mensaje en una transacción

La disposición del mensaje incluye operaciones como `Complete` / `Abandon` / `DeadLetter` / `Defer`. Para llevar a cabo estas operaciones dentro de una transacción, pase el elemento `transactional-state` con la disposición.

| Cliente (controlador) | Dirección | Service Bus (coordinador) |
| :--- | :---: | :--- |
| transfer(<br/>delivery-id=0, ...)<br/>{ AmqpValue (Declare())}| ------> |  |
|  | <------ | disposition( <br/> first=0, last=0, <br/>state=Declared(<br/>txn-id={transaction ID}<br/>))|
| | <------ |transfer(<br/>handle=2,<br/>delivery-id=11, <br/>state=null)<br/>{ payload }|  
| disposition( <br/> first=11, last=11, <br/>state=**TransactionalState(<br/>txn-id=0,<br/>outcome=Accepted()** ))| ------> |


## <a name="advanced-service-bus-capabilities"></a>Funcionalidades avanzadas de Service Bus

En esta sección se tratan las funcionalidades avanzadas de Azure Service Bus que se basan en los borradores de las extensiones AMQP que actualmente se están desarrollando en el comité técnico de OASIS para AMQP. Service Bus implementa las versiones más recientes de estos borradores y adopta los cambios introducidos a medida que los borradores alcanzan el estado estándar.

> [!NOTE]
> Se admiten las operaciones avanzadas de mensajería de Service Bus mediante un modelo de solicitud y respuesta. Los detalles de estas operaciones se describen en el artículo [El protocolo AMQP 1.0 de Microsoft Azure Service Bus: operaciones de respuesta/solicitud](service-bus-amqp-request-response.md).
> 
> 

### <a name="amqp-management"></a>Administración de AMQP

La especificación de la administración de AMQP es la primera de las extensiones de borrador que se analizan en este artículo. Esta especificación define un conjunto de protocolos superpuesto al protocolo AMQP que permite las interacciones de administración con la infraestructura de mensajería a través de AMQP. La especificación define operaciones genéricas como *crear*, *leer*, *actualizar* y *eliminar* para administrar entidades dentro de una infraestructura de mensajería y un conjunto de operaciones de consulta.

Todos los gestos requieren una interacción de solicitud/respuesta entre el cliente y la infraestructura de mensajería y, por tanto, la especificación define cómo modelar ese patrón de interacción sobre AMQP: el cliente se conecta a la infraestructura de mensajería, inicia una sesión y crea un par de vínculos. En un vínculo, el cliente actúa como remitente y en el otro actúa como receptor, creando así un par de vínculos que puede actuar como un canal bidireccional.

| Operadores lógicos | Remoto | Azure Service Bus |
| --- | --- | --- |
| Creación de una ruta de acceso de respuesta de solicitud |--> attach(<br/>name={*link name*},<br/>handle={*numeric handle*},<br/>role=**sender**,<br/>source=**null**,<br/>target=”myentity/$management”<br/>) |Ninguna acción |
| Creación de una ruta de acceso de respuesta de solicitud |Ninguna acción |\<-- attach(<br/>name={*link name*},<br/>handle={*numeric handle*},<br/>role=**receiver**,<br/>source=null,<br/>target=”myentity”<br/>) |
| Creación de una ruta de acceso de respuesta de solicitud |--> attach(<br/>name={*link name*},<br/>handle={*numeric handle*},<br/>role=**receiver**,<br/>source=”myentity/$management”,<br/>target=”myclient$id”<br/>) | |
| Creación de una ruta de acceso de respuesta de solicitud |Ninguna acción |\<-- attach(<br/>name={*link name*},<br/>handle={*numeric handle*},<br/>role=**sender**,<br/>source=”myentity”,<br/>target=”myclient$id”<br/>) |

Con ese par de vínculos en su lugar, la implementación de la solicitud/respuesta es sencilla: una solicitud es un mensaje enviado a una entidad dentro de la infraestructura de mensajería que comprende este patrón. En ese mensaje de solicitud, el campo *reply-to* de la sección *properties* se establece en el identificador *target* para el vínculo en el que se va a entregar la respuesta. La entidad de control procesa la solicitud y después entrega la respuesta a través del vínculo cuyo identificador *target* coincide con el identificador *reply-to* indicado.

Obviamente, el patrón requiere que el contenedor del cliente y el identificador generado por el cliente para el destino de la respuesta sean únicos en todos los clientes y, por motivos de seguridad, también difíciles de predecir.

Los intercambios de mensajes usados para el protocolo de administración y para todos los demás protocolos que usan el mismo patrón se producen en el nivel de la aplicación; no definen nuevos gestos en el nivel de protocolo AMQP. Eso es intencionado para que las aplicaciones puedan aprovechar inmediatamente estas extensiones con pilas de AMQP 1.0 compatibles.

Service Bus no implementa actualmente ninguna de las características principales de la especificación de administración, pero el patrón de solicitud/respuesta definido por la especificación de administración es fundamental para la característica de seguridad basada en notificaciones y para casi todas las funcionalidades avanzadas que se tratan en las secciones siguientes:

### <a name="claims-based-authorization"></a>Autorización basada en notificaciones

El borrador de la especificación de autorización basada en notificaciones (CBS) de AMQP se basa en el patrón de solicitud/respuesta de la especificación de administración; describe un modelo generalizado para el uso de tokens de seguridad federados con AMQP.

El modelo de seguridad predeterminado de AMQP que se describe en la introducción se basa en SASL y se integra con el protocolo de enlace de conexión de AMQP. El uso de SASL tiene la ventaja de ofrecer un modelo extensible para el que se ha definido un conjunto de mecanismos del que se puede beneficiar cualquier protocolo que emplee formalmente SASL. Entre estos mecanismos están: "PLAIN" para la transferencia de usuarios y contraseñas, “EXTERNAL” para enlazar con la seguridad de nivel de TLS, “ANONYMOUS” para expresar la ausencia de autenticación y autorización explícita y una amplia variedad de mecanismos adicionales que permiten pasar credenciales de autenticación y autorización o tokens.

La integración de SASL de AMQP tiene dos inconvenientes:

* Todas las credenciales y los tokens se limitan al ámbito de la conexión. Una infraestructura de mensajería puede desear proporcionar un control de acceso diferenciado por entidad. Por ejemplo, permitiendo que el portador de un token envíe a la cola A pero no a cola B. Con el contexto de autorización anclado en la conexión, no es posible usar una sola conexión y utilizar tokens de acceso diferentes para la cola A y la cola B.
* Normalmente, los tokens de acceso solo son válidos durante un tiempo limitado. Esta validez obliga al usuario a volver a adquirir periódicamente los tokens y proporciona una oportunidad al emisor del token de rechazar la emisión de un nuevo token si los permisos de acceso del usuario han cambiado. Las conexiones de AMQP pueden durar períodos muy largos. El modelo SASL solo proporciona una oportunidad para establecer un token en tiempo de conexión, lo que significa que la infraestructura de mensajería tiene para desconectar el cliente cuando el token expira o debe aceptar el riesgo de permitir una comunicación continua con un cliente cuyos derechos de acceso pueden haberse revocado mientras tanto.

La especificación de CBS de AMQP, implementada por Service Bus, proporciona una solución alternativa elegante para estos problemas: Permite que un cliente asocie los tokens de acceso a cada nodo y actualice esos tokens antes de que expiren, sin interrumpir el flujo de mensajes.

CBS define un nodo de administración virtual denominado *$cbs*, proporcionado por la infraestructura de mensajería. El nodo de administración acepta los tokens en nombre de los otros nodos de la infraestructura de mensajería.

El gesto del protocolo es un intercambio de solicitud y respuesta, tal como está definido en la especificación de administración. Esto significa que el cliente establece un par de vínculos con el nodo *$cbs*, pasa una solicitud en el vínculo de salida y, a continuación, espera la respuesta en el vínculo de entrada.

El mensaje de solicitud tiene las siguientes propiedades de la aplicación:

| Clave | Opcional | Tipo de valor | Contenido del valor |
| --- | --- | --- | --- |
| operation |No |string |**put-token** |
| type |No |string |Tipo del token que se coloca. |
| name |No |string |El "público" al que se aplica el token. |
| expiration |Sí |timestamp |La hora de expiración del token. |

La propiedad *name* identifica la entidad a la que se va a asociar el token. En Service Bus es la ruta de acceso a la cola, el tema o la suscripción. La propiedad *type* identifica el tipo de token:

| Tipo de token | Descripción del token | Tipo de cuerpo | Notas |
| --- | --- | --- | --- |
| amqp:jwt |JSON Web Token (JWT) |Valor de AMQP (cadena) |No disponible todavía. |
| amqp:swt |Simple Web Token (SWT) |Valor de AMQP (cadena) |Solo se admite para los tokens SWT emitidos por AAD y ACS |
| servicebus.windows.net:sastoken |Token SAS de Service Bus |Valor de AMQP (cadena) |- |

Los tokens confieren derechos. Service Bus conoce tres derechos fundamentales: "Enviar" permite enviar, "Escuchar" permite recibir y "Administrar" permite manipular las entidades. Los tokens SWT emitidos por AAD/ACS incluyen explícitamente dichos derechos como notificaciones. Los tokens de SAS de Service Bus hacen referencia a las reglas configuradas en el espacio de nombres o la entidad, y dichas reglas se configuran con derechos. Firmar el token con la clave asociada a esa regla hace que el token exprese los derechos correspondientes. El token asociado a una entidad con *put-token* permite que el cliente conectado interactúe con la entidad de acuerdo con los derechos del token. Un vínculo en el que cliente adopta el rol *sender* requiere el derecho "Enviar"; adoptar el rol *receiver* requiere el derecho "Escuchar".

El mensaje de respuesta tiene los siguientes valores de *application-properties*

| Clave | Opcional | Tipo de valor | Contenido del valor |
| --- | --- | --- | --- |
| status-code |No |int |Código de respuesta HTTP **[RFC2616]** . |
| status-description |Sí |string |Descripción del estado. |

El cliente puede llamar a *put-token* repetidamente y para cualquier entidad de la infraestructura de mensajería. El ámbito de los tokens es el cliente actual y se anclan en la conexión actual, lo que significa que el servidor elimina todos los tokens retenidos cuando la conexión se interrumpe.

La implementación actual de Service Bus solo permite CBS junto con el método SASL “ANONYMOUS”. Siempre debe existir una conexión SSL/TLS antes del protocolo de enlace SASL.

Por tanto, el mecanismo ANONYMOUS debe ser compatible con el cliente de AMQP 1.0 elegido. El acceso anónimo significa que el protocolo de enlace de conexión inicial, incluida la creación de la sesión inicial, tiene lugar sin que Service Bus sepa quién está creando la conexión.

Una vez establecida la conexión y la sesión, las únicas operaciones permitidas son asociar los vínculos al nodo *$cbs* y enviar la solicitud *put-token*. Un token válido debe establecerse correctamente mediante una solicitud *put-token* para algún nodo de entidad en un plazo de 20 segundos después de establecer la conexión; de lo contrario, Service Bus interrumpe la conexión unilateralmente.

El cliente es responsable de realizar un seguimiento de la expiración del token. Cuando expira un token, Service Bus elimina rápidamente todos los vínculos de la conexión a la entidad correspondiente. Para evitar el problema, el cliente puede sustituir el token del nodo por uno nuevo en cualquier momento mediante el nodo de administración virtual *$cbs* con el mismo gesto *put-token* y sin interferir con el tráfico de carga que fluye en diferentes vínculos.

### <a name="send-via-functionality"></a>Funcionalidad "enviar por"

[Enviar por / Transferir remitente ](service-bus-transactions.md#transfers-and-send-via) es una funcionalidad que permite a Service Bus desviar un mensaje dado a una entidad de destino a través de otra entidad. Esta característica se usa para realizar operaciones entre entidades en una sola transacción.

Gracias a esta funcionalidad, puede crear un remitente y establecer el vínculo a `via-entity`. Al establecer el vínculo, se pasa información adicional para establecer el verdadero destino de los mensajes o transferencias en ese vínculo. Una vez que la asociación se ha realizado correctamente, todos los mensajes enviados en este vínculo se reenvían automáticamente a *destination-entity* a través de *via-entity*. 

> Nota: La autenticación debe realizarse en ambos elementos *via-entity* y *destination-entity* antes de establecer este vínculo.

| Remoto | Dirección | Azure Service Bus |
| :--- | :---: | :--- |
| attach(<br/>name={link name},<br/>role=sender,<br/>source={client link ID},<br/>target= **{via-entity}** ,<br/>**properties=map [(<br/>com.microsoft:transfer-destination-address=<br/>{destination-entity} )]** ) | ------> | |
| | <------ | attach(<br/>name={link name},<br/>role=receiver,<br/>source={client link ID},<br/>target={via-entity},<br/>properties=map [(<br/>com.microsoft:transfer-destination-address=<br/>{destination-entity} )] ) |

## <a name="next-steps"></a>Pasos siguientes

Para aprender más sobre AMQP, visite los siguientes vínculos:

* [Información general sobre AMQP para Service Bus]
* [Compatibilidad de AMQP 1.0 con los temas y las colas con particiones de Service Bus]
* [AMQP de Service Bus para Windows Server]

[this video course]: https://www.youtube.com/playlist?list=PLmE4bZU0qx-wAP02i0I7PJWvDWoCytEjD
[1]: ./media/service-bus-amqp-protocol-guide/amqp1.png
[2]: ./media/service-bus-amqp-protocol-guide/amqp2.png
[3]: ./media/service-bus-amqp-protocol-guide/amqp3.png
[4]: ./media/service-bus-amqp-protocol-guide/amqp4.png

[Información general sobre AMQP para Service Bus]: service-bus-amqp-overview.md
[Compatibilidad de AMQP 1.0 con los temas y las colas con particiones de Service Bus]: service-bus-partitioned-queues-and-topics-amqp-overview.md
[AMQP de Service Bus para Windows Server]: https://msdn.microsoft.com/library/dn574799.aspx
