---
title: 'Colas de Azure Storage y colas de Service Bus: comparación y diferencias | Microsoft Docs'
description: Analiza las diferencias y similitudes entre dos tipos de colas que se ofrecen en Azure.
services: service-bus-messaging
documentationcenter: na
author: axisc
manager: timlt
editor: spelluru
ms.assetid: f07301dc-ca9b-465c-bd5b-a0f99bab606b
ms.service: service-bus-messaging
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: tbd
ms.date: 01/23/2019
ms.author: aschhab
ms.openlocfilehash: dbbc43bc7a2f42f8a72ce12d84da1ae406a588d2
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65799350"
---
# <a name="storage-queues-and-service-bus-queues---compared-and-contrasted"></a>Colas de Storage y de Service Bus: comparación y diferencias
En este artículo se analizan las diferencias y similitudes entre los dos tipos de cola que actualmente ofrece Microsoft Azure: las colas de Storage y las colas de Service Bus. Con esta información, puede comparar y contrastar las tecnologías respectivas y puede tomar una decisión más fundamentada sobre la solución que satisfaga mejor sus necesidades.

## <a name="introduction"></a>Introducción
Azure admite dos tipos de mecanismos de cola: **colas de Storage** y **colas de Service Bus**.

Las **colas de Storage**, que forman parte de la infraestructura de [Azure Storage](https://azure.microsoft.com/services/storage/), ofrecen una interfaz de GET/PUT/PEEK sencilla basada en REST, que ofrece una mensajería confiable y persistente dentro de los servicios y entre ellos.

Las **colas de Service Bus** forman parte de una infraestructura de [mensajería de Azure](https://azure.microsoft.com/services/service-bus/) más amplia que admite la puesta en cola, así como la publicación/suscripción, y patrones de integración más avanzados. Para más información sobre las colas de Service Bus, los temas y las suscripciones, consulte la [introducción a Service Bus](service-bus-messaging-overview.md).

Aunque ambas tecnologías de cola existen de manera simultánea, las colas de Storage se presentaron en primer lugar, como un mecanismo de almacenamiento de cola dedicado creado a partir de los servicios de Azure Storage. Las colas de Service Bus se generan a partir de la infraestructura de mensajería más amplia diseñada para integrar aplicaciones o componentes de aplicaciones que pueden abarcar varios protocolos de comunicación, contratos de datos, dominios de confianza o entornos de red.

## <a name="technology-selection-considerations"></a>Consideraciones de selección de tecnología
Tanto las colas de Storage como las colas de Service Bus son implementaciones del servicio de cola de mensajes ofrecido actualmente por Microsoft Azure. Cada una tiene un conjunto de características ligeramente diferente, lo que significa que puede elegir una u otra, o usar ambas, según las necesidades de su solución concreta o problema empresarial o técnico que va a resolver.

Al determinar qué tecnología de cola se ajusta al propósito de una solución determinada, los desarrolladores y arquitectos de soluciones deben considerar estas recomendaciones. Para obtener información detallada vea la siguiente sección.

Como arquitecto o desarrollador de soluciones, **debe considerar el uso de colas de Storage** en los siguientes casos:

* La aplicación debe almacenar más de 80 GB de mensajes en una cola.
* Su aplicación desea realizar un seguimiento del progreso para procesar un mensaje dentro de la cola. Esto es útil si se bloquea el trabajador que procesa un mensaje. Un trabajador posterior puede usar esa información para continuar desde donde se marchó el trabajador anterior.
* Necesita registros de lado de servidor de todas las transacciones ejecutadas con las colas.

Como arquitecto o desarrollador de soluciones, **debe considerar el uso de colas de Service Bus** cuando:

* Su solución debe poder recibir mensajes sin tener que sondear la cola. Con Service Bus, esto se logra mediante el uso de la operación de recepción de sondeo largo mediante los protocolos basados en TCP que admite Service Bus.
* Su solución requiere que la cola ofrezca una entrega ordenada por primero en entrar es el primero en salir (FIFO).
* Su solución debe ser capaz de admitir la detección automática de duplicados.
* Quiere que su aplicación procese mensajes como secuencias de larga ejecución en paralelo (los mensajes están asociados a una secuencia mediante la propiedad [SessionId](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.sessionid) del mensaje). En este modelo, cada nodo de la aplicación de consumo compite por secuencias, en lugar de los mensajes. Cuando se proporciona una secuencia a un nodo de consumo, el nodo puede examinar el estado de la secuencia de aplicación mediante transacciones.
* Su solución requiere un comportamiento transaccional y atomicidad al enviar o recibir varios mensajes desde una cola.
* Su aplicación administra mensajes que pueden superar los 64 KB pero probablemente no alcanzarán el límite de 256 KB.
* Aborda un requisito para ofrecer un modelo de acceso basado en roles a las colas y diferentes derechos o permisos para los remitentes y receptores.
* El tamaño de la cola no aumentará a más de 80 GB.
* Quiere usar el protocolo de mensajería basado en estándares AMQP 1.0. Para obtener más información sobre AMQP, vea [Introducción al AMQP de Service Bus](service-bus-amqp-overview.md).
* Puede idear una migración eventual desde la comunicación punto a punto basada en cola a un patrón de intercambio de mensajes que permite la integración perfecta de receptores adicionales (suscriptores), cada uno de los cuales recibe copias independientes de algunos o todos los mensajes enviados a la cola. El segundo se refiere a la capacidad de publicación o suscripción ofrecida por Service Bus de forma nativa.
* La solución de mensajería debe ser capaz de admitir la garantía de entrega "Una vez como máximo" sin necesidad de compilar los componentes de infraestructura adicionales.
* Le gustaría poder publicar y consumir lotes de mensajes.

## <a name="comparing-storage-queues-and-service-bus-queues"></a>Comparación de las colas de Storage y las colas de Service Bus
En las tablas de las secciones siguientes se ofrece una agrupación lógica de las características de cola, lo que le permite comparar de un vistazo las funcionalidades disponibles en las colas de Azure Storage y de Service Bus.

## <a name="foundational-capabilities"></a>Capacidades fundamentales
En esta sección se comparan algunas de las funcionalidades de puesta en cola fundamentales ofrecidas por las colas de Storage y las colas de Service Bus.

| Criterios de comparación | Colas de almacenamiento | Colas de Service Bus |
| --- | --- | --- |
| Garantía de ordenación |**No** <br/><br>Para obtener más información, vea la primera nota de la sección "Información adicional".</br> |**Sí- Primero en caducar primero en salir (FIFO)**<br/><br>(mediante el uso de sesiones de mensajería) |
| Garantía de entrega |**Al menos una vez** |**Al menos una vez**<br/><br/>**Como máximo una vez** |
| Compatibilidad con la operación atómica |**No** |**Sí**<br/><br/> |
| Comportamiento de recepción |**Sin bloqueo**<br/><br/>(se completa inmediatamente si no se encuentra ningún mensaje nuevo) |**Bloqueo con/sin tiempo de espera**<br/><br/>(ofrece sondeo largo o ["Técnica de cometa"](https://go.microsoft.com/fwlink/?LinkId=613759))<br/><br/>**Sin bloqueo**<br/><br/>(solo mediante el uso de la API administrada de .NET) |
| API de estilo de inserción |**No** |**Sí**<br/><br/>API de .NET de sesiones de [OnMessage](/dotnet/api/microsoft.servicebus.messaging.queueclient.onmessage#Microsoft_ServiceBus_Messaging_QueueClient_OnMessage_System_Action_Microsoft_ServiceBus_Messaging_BrokeredMessage__) y **OnMessage**. |
| Modo de recepción |**Ojear y alquilar** |**Ojear y bloquear**<br/><br/>**Recibir y eliminar** |
| Modo de acceso exclusivo |**Basado en concesión** |**Basado en bloqueo** |
| Duración de concesión/bloqueo |**30 segundos (valor predeterminado)**<br/><br/>**7 días (máximo)** (puede renovar o liberar una concesión de mensaje mediante la API de [UpdateMessage](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.updatemessage)). |**60 segundos (valor predeterminado)**<br/><br/>Puede renovar un bloqueo de mensaje mediante la API de [RenewLock](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.renewlock#Microsoft_ServiceBus_Messaging_BrokeredMessage_RenewLock). |
| Precisión de concesión/bloqueo |**Nivel de mensaje**<br/><br/>(cada mensaje puede tener un valor de tiempo de espera diferente, que luego puede actualizar según sea necesario al procesar el mensaje, mediante el uso de la API de [UpdateMessage](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.updatemessage)) |**Nivel de cola**<br/><br/>(cada cola tiene una precisión de bloqueo que se aplica a todos sus mensajes, pero puede renovar el bloqueo mediante la API de [RenewLock](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.renewlock#Microsoft_ServiceBus_Messaging_BrokeredMessage_RenewLock)). |
| Recepción por lotes |**Sí**<br/><br/>(especificando explícitamente el número de mensajes al recuperar mensajes, hasta un máximo de 32 mensajes) |**Sí**<br/><br/>(habilitando implícitamente una propiedad de captura previa o explícitamente mediante el uso de transacciones) |
| Envío por lotes |**No** |**Sí**<br/><br/>(mediante el uso de transacciones o de procesamiento por lotes del lado cliente) |

### <a name="additional-information"></a>Información adicional
* El sistema de los mensajes en las colas de Storage es normalmente primero en entrar, primero en salir; sin embargo, en ocasiones, pueden estar desordenados; por ejemplo, cuando expira la duración de tiempo de espera de visibilidad de un mensaje (por ejemplo, como resultado de una aplicación cliente que se bloquea durante el proceso). Cuando expira el tiempo de espera de visibilidad, el mensaje se vuelve visible en la cola para que otro trabajador lo quite de la cola. En ese momento, el mensaje recién visible se puede colocar en la cola (para quitarlo después) después de un mensaje que se colocó originalmente en cola después de él.
* El patrón de FIFO garantizado en las colas de Service Bus requiere el uso de sesiones de mensajería. En caso de que la aplicación se bloquee al procesar un mensaje recibido en el modo **Ojear y bloquear**, la próxima vez que un receptor de la cola acepte una sesión de mensajería, empezará con el mensaje de error después de que expire su período de vida (TTL).
* Las colas de Storage están diseñadas para admitir escenarios de puesta en cola estándar, como componentes de aplicación de desacoplamiento para aumentar la escalabilidad y tolerancia a errores, nivelación de carga y creación de flujos de trabajo de proceso.
* Las colas de Service Bus admiten la garantía de entrega *Al menos una vez*. 
* Se pueden evitar incoherencias con respecto al control de mensajes en el contexto de las sesiones de Service Bus con el estado de sesión para almacenar el estado de la aplicación en relación con el progreso de la secuencia del mensaje de la sesión de control y mediante el uso de transacciones en torno a fijar recibe los mensajes y actualizar el estado de sesión. Este tipo de característica de coherencia se denomina a veces *exactamente-procesamiento una vez* en productos de otros proveedores, pero transacción errores obviamente hará que se vuelva a entregar los mensajes y, por tanto, el término es suficiente no exactamente.
* Las colas de Storage ofrecen un modelo de programación coherente y uniforme en las colas, tablas y blobs, tanto para desarrolladores como para los equipos de operaciones.
* Las colas de Service Bus ofrecen compatibilidad con transacciones locales en el contexto de una sola cola.
* El modo **Recibir y eliminar** compatible con el Service Bus ofrece la capacidad de reducir el número de operaciones de mensajería (y el costo asociado) a cambio de una garantía de entrega menor.
* Las colas de Storage ofrecen concesiones con la posibilidad de ampliar las concesiones para mensajes. Esto permite que los trabajadores mantengan breves concesiones en los mensajes. Por lo tanto, si se bloquea a un trabajador, el mensaje se puede volver a procesar rápidamente por otro trabajador. Además, un trabajador puede ampliar la concesión de un mensaje si necesita procesarlo durante más tiempo que el tiempo de concesión actual.
* Las colas de Storage ofrecen un tiempo de espera de visibilidad que puede establecer al poner en cola un mensaje o quitarlo de la cola. Además, puede actualizar un mensaje con distintos valores de concesión en el sistema en tiempo de ejecución y actualizar distintos valores entre mensajes de la misma cola. Los tiempos de espera de bloqueo de Service Bus lock se definen en los metadatos de la cola; sin embargo, puede renovar el bloqueo mediante una llamada al método [RenewLock](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.renewlock#Microsoft_ServiceBus_Messaging_BrokeredMessage_RenewLock).
* El tiempo de espera máximo para una operación de recepción de bloqueo en las colas de Service Bus es de 24 días. Sin embargo, los tiempos de espera basados en REST tienen un valor máximo de 55 segundos.
* El procesamiento por lotes del cliente ofrecido por el Service Bus permite a un cliente de cola procesar varios mensajes por lotes en una sola operación de envío. El procesamiento por lotes solo está disponible para las operaciones de envío asincrónicas.
* Características como el límite de 200 TB de las colas de Storage (más cuando se virtualizan las cuentas) y las colas ilimitadas la convierten en la plataforma idónea para los proveedores de SaaS.
* Las colas de Storage ofrecen un mecanismo de control de acceso delegado flexible y eficiente.

## <a name="advanced-capabilities"></a>Capacidades avanzadas
En esta sección se comparan algunas de las funcionalidades avanzadas ofrecidas por las colas de Storage y las colas de Service Bus.

| Criterios de comparación | Colas de almacenamiento | Colas de Service Bus |
| --- | --- | --- |
| Entrega programada |**Sí** |**Sí** |
| Mensajes fallidos automáticos |**No** |**Sí** |
| Aumentar el valor de período de vida de cola |**Sí**<br/><br/>(a través de la actualización local de tiempo de espera de visibilidad) |**Sí**<br/><br/>(proporcionado mediante una función de API dedicada) |
| Compatibilidad con mensajes dudosos |**Sí** |**Sí** |
| Actualización local |**Sí** |**Sí** |
| Registro de transacciones del servidor |**Sí** |**No** |
| Métricas de almacenamiento |**Sí**<br/><br/>**Métricas de minuto**: ofrece métricas en tiempo real para disponibilidad, TPS, recuentos de llamadas de API, recuentos de errores y mucho más, todo en tiempo de real (agregado por minuto y notificado en unos minutos a partir de lo que acaba de ocurrir en producción. Para obtener más información, vea [Acerca de las métricas de Storage Analytics](/rest/api/storageservices/fileservices/About-Storage-Analytics-Metrics). |**Sí**<br/><br/>(consultas masivas llamando a [GetQueues](/dotnet/api/microsoft.servicebus.namespacemanager.getqueues#Microsoft_ServiceBus_NamespaceManager_GetQueues)) |
| Administración de estados |**No** |**Sí**<br/><br/>[Microsoft.ServiceBus.Messaging.EntityStatus.Active](/dotnet/api/microsoft.servicebus.messaging.entitystatus), [Microsoft.ServiceBus.Messaging.EntityStatus.Disabled](/dotnet/api/microsoft.servicebus.messaging.entitystatus), [Microsoft.ServiceBus.Messaging.EntityStatus.SendDisabled](/dotnet/api/microsoft.servicebus.messaging.entitystatus), [Microsoft.ServiceBus.Messaging.EntityStatus.ReceiveDisabled](/dotnet/api/microsoft.servicebus.messaging.entitystatus) |
| Reenvío automático de mensajes |**No** |**Sí** |
| Purgar la función de cola |**Sí** |**No** |
| Grupos de mensajes |**No** |**Sí**<br/><br/>(mediante el uso de sesiones de mensajería) |
| Estado de la aplicación por grupo de mensajes |**No** |**Sí** |
| Detección de duplicados |**No** |**Sí**<br/><br/>(configurable en el lado del remitente) |
| Exploración de grupos de mensaje |**No** |**Sí** |
| Captura de sesiones de mensajes por id. |**No** |**Sí** |

### <a name="additional-information"></a>Información adicional
* Ambas tecnologías de cola permiten que se programe un mensaje para su entrega posteriormente.
* El reenvío automático de cola permite el reenvío automático de miles de colas de sus mensajes a una única cola, desde la que la aplicación receptora consume el mensaje. Puede usar este mecanismo para lograr seguridad, flujo de control y aislar el almacenamiento aislado entre cada publicador de mensajes.
* Las colas de Storage ofrecen compatibilidad para actualizar el contenido del mensaje. Puede usar esta funcionalidad para conservar información de estado y actualizaciones incrementales de progreso en el mensaje para que se pueda procesar desde el último punto de comprobación conocido, en lugar de hacerlo desde el principio. Con las colas de Service Bus, puede habilitar el mismo escenario mediante el uso de sesiones de mensajes. Las sesiones le permiten guardar y recuperar el estado de procesamiento de la aplicación (mediante [SetState](/dotnet/api/microsoft.servicebus.messaging.messagesession.setstate#Microsoft_ServiceBus_Messaging_MessageSession_SetState_System_IO_Stream_) y [GetState](/dotnet/api/microsoft.servicebus.messaging.messagesession.getstate#Microsoft_ServiceBus_Messaging_MessageSession_GetState)).
* El [proceso como correo devuelto](service-bus-dead-letter-queues.md), solo admitido por las colas de Service Bus, puede ser útiles para aislar los mensajes que la aplicación receptora no puede procesar correctamente o cuando los mensajes no pueden llegar a su destino debido a una propiedad de período de vida (TTL) expirada. El valor de TTL especifica cuánto tiempo permanece un mensaje en la cola. Con Service Bus, el mensaje se moverá a una cola especial denominada $DeadLetterQueue cuando expira el período de vida.
* Para encontrar mensajes "dudosos" en las colas de Storage, al quitar de la cola un mensaje de la aplicación se examina la propiedad [DequeueCount](/dotnet/api/microsoft.azure.storage.queue.cloudqueuemessage.dequeuecount) del mensaje. Si **DequeueCount** se encuentra por encima de un umbral determinado, la aplicación mueve el mensaje a una cola de proceso como correo devuelto definida por la aplicación.
* Las colas de Storage le permiten obtener un registro detallado de todas las transacciones ejecutadas con la cola, así como las métricas agregadas. Ambas opciones son útiles para depurar y entender cómo usa su aplicación las colas de Storage. También son útiles para optimizar el rendimiento de la aplicación y reducir los costos de usar colas.
* El concepto de "sesiones de mensajes" admitido por Service Bus permite que los mensajes que pertenecen a un determinado grupo lógico se asocien a un receptor específico, que a su vez crea una afinidad de sesiones entre los mensajes y sus receptores respectivos. Puede habilitar esta funcionalidad avanzada en el Service Bus estableciendo la propiedad [SessionID](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.sessionid#Microsoft_ServiceBus_Messaging_BrokeredMessage_SessionId) en un mensaje. Los receptores pueden escuchar entonces en un id. de sesión específico y recibir mensajes que comparten el identificador de sesión especificado.
* La funcionalidad de detección de duplicación admitida por las colas de Service Bus elimina automáticamente los mensajes duplicados enviados a una cola o tema, según el valor de la propiedad [MessageID](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.messageid#Microsoft_ServiceBus_Messaging_BrokeredMessage_MessageId).

## <a name="capacity-and-quotas"></a>Capacidad y cuotas
En esta sección se comparan las colas de Storage y las colas de Service Bus desde la perspectiva de la [capacidad y las cuotas](service-bus-quotas.md) que se pueden aplicar.

| Criterios de comparación | Colas de almacenamiento | Colas de Service Bus |
| --- | --- | --- |
| Tamaño de cola máximo |**500 TB**<br/><br/>(limitado a una [capacidad de cuenta de almacenamiento única](../storage/common/storage-introduction.md#queue-storage)) |**De 1 GB a 80 GB**<br/><br/>(definido al crear una cola y [habilitar particiones](service-bus-partitioning.md): vea la sección "Información adicional") |
| Tamaño de mensaje máximo |**64 KB**<br/><br/>(48 K cuando se usa la codificación **Base64**)<br/><br/>Azure admite mensajes de gran tamaño mediante la combinación de colas y blobs, momento en el que puede poner en cola hasta 200 GB para un solo elemento. |**256 KB** o **1 MB**<br/><br/>(incluidos tanto el encabezado como el cuerpo, el tamaño máximo de encabezado es: 64 KB).<br/><br/>Depende del [nivel de servicio](service-bus-premium-messaging.md). |
| TTL de mensaje máximo |**Infinitas** (a partir de la versión de API 2017-07-27) |**TimeSpan.Max** |
| Número máximo de colas |**Sin límite** |**10.000**<br/><br/>(por espacio de nombres de servicio) |
| Número máximo de clientes simultáneos |**Sin límite** |**Sin límite**<br/><br/>(el límite de 100 conexiones simultáneas solo se aplica a la comunicación basada en protocolo TCP) |

### <a name="additional-information"></a>Información adicional
* Service Bus aplica límites de tamaño de cola. El tamaño máximo de la cola se especifica al crear la cola y puede tener un valor entre 1 y 80 GB. Si se alcanza el valor de tamaño de la cola establecido al crear la cola, se rechazarán los mensajes entrantes adicionales y el código de llamada recibirá una excepción. Para obtener más información sobre las cuotas en el Service Bus, vea [Cuotas de Service Bus](service-bus-quotas.md).
* No se admite la creación de particiones en el [nivel Premium](service-bus-premium-messaging.md). En el nivel Estándar, puede crear colas de Service Bus en tamaños de 1, 2, 3, 4 o 5 GB (el valor predeterminado es 1 GB). En el nivel Estándar, con las particiones habilitadas (que es el valor predeterminado), Service Bus crea 16 particiones por cada GB que especifique. Por lo tanto, si crea una cola con un tamaño de 5 GB y con 16 particiones, el tamaño de cola máximo pasa a ser (5 * 16) = 80 GB. Puede ver el tamaño máximo de la cola o tema con particiones examinando su entrada en [Azure Portal][Azure portal].
* Con las colas de Storage, si el contenido del mensaje no es seguro para XML, debe estar codificado con **Base64**. Si codifica el mensaje con **Base64**, la carga de usuario puede ser de hasta 48 KB, en lugar de 64 KB.
* Con las colas de Service Bus, cada mensaje almacenado en una cola consta de dos partes: un encabezado y un cuerpo. El tamaño total del mensaje no puede superar el tamaño máximo admitido por el nivel de servicio.
* Cuando los clientes se comunican con colas de Service Bus por el protocolo TCP, el número máximo de conexiones simultáneas a una única cola de Service Bus se limita a 100. Este número se comparte entre remitentes y receptores. Si se alcanza esta cuota, se rechazarán las solicitudes posteriores de conexiones adicionales y el código de llamada recibirá una excepción. Este límite no se impone en clientes que se conectan a las colas mediante la API basada en REST.
* Si necesita más de 10.000 colas en un único espacio de nombres de Service Bus, puede ponerse en contacto con el equipo de soporte técnico de Azure y solicitar un aumento. Para escalar más allá de las 10 000 colas con Service Bus, también puede crear espacios de nombres adicionales mediante [Azure Portal][Azure portal].

## <a name="management-and-operations"></a>Administración y operaciones
En esta sección se comparan algunas de las características de administración ofrecidas por las colas de Storage y las colas de Service Bus.

| Criterios de comparación | Colas de almacenamiento | Colas de Service Bus |
| --- | --- | --- |
| Protocolo de administración |**REST sobre HTTP/HTTPS** |**REST sobre HTTPS** |
| Protocolo de tiempo de ejecución |**REST sobre HTTP/HTTPS** |**REST sobre HTTPS**<br/><br/>**AMQP 1.0 estándar (TCP con TLS)** |
| API de .NET |**Sí**<br/><br/>(API de cliente de Storage para .NET) |**Sí**<br/><br/>(API de Service Bus para .NET) |
| C++ nativo |**Sí** |**Sí** |
| API de Java |**Sí** |**Sí** |
| API de PHP |**Sí** |**Sí** |
| API de Node.js. |**Sí** |**Sí** |
| Compatibilidad con metadatos arbitrarios |**Sí** |**No** |
| Reglas de nomenclatura de cola |**Hasta 63 caracteres**<br/><br/>(las letras de un nombre de cola deben estar en minúscula). |**Hasta 260 caracteres**<br/><br/>(los nombres y las rutas de acceso de las colas no distinguen mayúsculas de minúsculas). |
| Función de obtención de la longitud de la cola |**Sí**<br/><br/>(valor aproximado si los mensajes expiran más allá del TTL sin eliminarse). |**Sí**<br/><br/>(valor exacto en un momento dado). |
| Función de ojear |**Sí** |**Sí** |

### <a name="additional-information"></a>Información adicional
* Las colas de Storage ofrecen compatibilidad con atributos arbitrarios que se pueden aplicar a la descripción de la cola, en forma de pares de nombre-valor.
* Ambas tecnologías de cola ofrecen la capacidad de ojear un mensaje sin tener que bloquearlo, lo que puede resultar útil al implementar una herramienta de explorador de colas.
* Las API de mensajería asíncrona de .NET de Service Bus aprovechan las conexiones TCP de dúplex completo para mejorar el rendimiento en comparación con REST sobre HTTP, y admiten el protocolo estándar AMQP 1.0.
* Los nombres de colas de Storage pueden tener de 3 a 63 caracteres de longitud que pueden incluir letras minúsculas, números y guiones. Para obtener más información, vea [Nomenclatura de colas y metadatos](/rest/api/storageservices/fileservices/Naming-Queues-and-Metadata).
* Los nombres de cola de Service Bus pueden tener hasta 260 caracteres y reglas de nomenclatura menos restrictivas. Los nombres de cola de Service Bus pueden contener letras, números, puntos, guiones y caracteres de subrayado.

## <a name="authentication-and-authorization"></a>Autenticación y autorización
En esta sección se describen las características de autenticación y autorización compatibles con las colas de Storage y las colas de Service Bus.

| Criterios de comparación | Colas de almacenamiento | Colas de Service Bus |
| --- | --- | --- |
| Authentication |**Clave simétrica** |**Clave simétrica** |
| Modelo de seguridad |Acceso delegado a través de tokens SAS. |SAS |
| Federación de proveedor de identidad: |**No** |**Sí** |

### <a name="additional-information"></a>Información adicional
* Se debe autenticar cada solicitud a cualquiera de las tecnologías de cola. No se admiten colas públicas con acceso anónimo. Con [SAS](service-bus-sas.md), puede abordar este escenario publicando un SAS de solo escritura, un SAS de solo lectura o incluso un SAS de acceso completo.
* El esquema de autenticación ofrecido por las colas de Storage implica el uso de una clave simétrica, que es un código de autenticación de mensajes basado en hash (HMAC), calculado con el algoritmo SHA-256 y codificado como una cadena **Base64**. Para obtener más información sobre el protocolo respectivo, consulte [Autenticación para los servicios de Azure Storage](/rest/api/storageservices/fileservices/Authentication-for-the-Azure-Storage-Services). Las colas de Service Bus admiten un modelo similar mediante claves simétricas. Para obtener más información, vea [Autenticación con firma de acceso compartido con Service Bus](service-bus-sas.md).

## <a name="conclusion"></a>Conclusión
Al comprender mejor las dos tecnologías, podrá tomar una decisión más fundamentada sobre la tecnología de cola que usará y cuándo. La decisión sobre cuándo usar las colas de Storage o las colas de Service Bus depende claramente de una serie de factores. Estos factores pueden dependen en gran medida de las necesidades individuales de la aplicación y de su arquitectura. Si la aplicación ya usa las funcionalidades principales de Microsoft Azure, quizás prefiera elegir las colas de Storage, especialmente si necesita comunicación básica y mensajería entre servicios o necesita colas que puedan tener un tamaño superior a 80 GB.

Dado que las colas de Service Bus ofrecen varias características avanzadas, como sesiones, transacciones, detección de duplicados, mensajes con problemas de entrega automáticos, y capacidades de publicación o suscripción duraderas, pueden ser la opción preferida si está creando una aplicación híbrida o si su aplicación necesita por otra parte estas características.

## <a name="next-steps"></a>Pasos siguientes
En los artículos siguientes se ofrece más orientación e información sobre el uso de las colas de Storage o las colas de Service Bus.

* [Introducción a las colas de Service Bus](service-bus-dotnet-get-started-with-queues.md)
* [Uso del servicio Queue Storage](../storage/queues/storage-dotnet-how-to-use-queues.md)
* [Procedimientos recomendados para mejorar el rendimiento mediante la mensajería asincrónica de Service Bus](service-bus-performance-improvements.md)
* [Introducción a las colas y los temas de Azure Service Bus (publicación en un blog)](https://www.code-magazine.com/article.aspx?quickid=1112041)
* [Guía para desarrolladores sobre el Service Bus](http://www.cloudcasts.net/devguide/Default.aspx?id=11030)
* [Uso del servicio de cola en Azure ](https://www.developerfusion.com/article/120197/using-the-queuing-service-in-windows-azure/)

[Azure portal]: https://portal.azure.com

