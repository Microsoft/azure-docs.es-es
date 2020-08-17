---
title: Preguntas frecuentes acerca de Azure Notification Hubs | Microsoft Docs
description: Preguntas frecuentes sobre el diseño y la implementación de soluciones en Azure Notification Hubs.
services: notification-hubs
documentationcenter: mobile
author: sethmanheim
manager: femila
keywords: notificación push, notificaciones push, notificaciones push de iOS, notificaciones push de android, inserción de ios, inserción de android
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-multiple
ms.devlang: multiple
ms.topic: article
ms.date: 11/13/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 11/13/2019
ms.openlocfilehash: 85ebb7f5ac52f4eea25f9e6f1a2b1b5ac6f4caa5
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87077921"
---
# <a name="push-notifications-with-azure-notification-hubs-frequently-asked-questions"></a>Notificaciones de inserción con Azure Notification Hubs: Preguntas más frecuentes

## <a name="general"></a>General

### <a name="what-is-the-resource-structure-of-notification-hubs"></a>¿Cuál es la estructura de los recursos de Notification Hubs?

Azure Notification Hubs tiene dos niveles de recursos, concentradores y espacios de nombres. Un concentrador es un recurso de inserción que puede contener la información de inserción multiplataforma de una aplicación. Un espacio de nombres es una colección de concentradores en una región. La asignación recomendada hace coincidir un espacio de nombres con una aplicación. Dentro de un espacio de nombres, puede tener un concentrador de producción que funcione con la aplicación de producción, un concentrador de prueba que funcione con la aplicación de prueba, etc.

### <a name="what-is-the-price-model-for-notification-hubs"></a>¿Cuál es el modelo de precios para Notification Hubs?

Los detalles más recientes sobre los precios pueden encontrarse en la página [Precios de Notification Hubs]. Notification Hubs se factura a nivel de espacio de nombres. (Si necesita ver la definición de espacio de nombres, consulte "¿Cuál es la estructura de los recursos de Notification Hubs?") Notification Hubs ofrece tres niveles:

* **Gratis**: este nivel es un buen punto de partida para explorar las funcionalidades de inserción. No se recomienda para las aplicaciones de producción. Se incluyen 500 dispositivos y 1 millón de inserciones por espacio de nombres por mes, sin garantía de Acuerdo de Nivel de Servicio (SLA).
* **Básico**: este nivel (o el Estándar) se recomienda para las aplicaciones de producción más pequeñas. Se incluyen 200 000 dispositivos y 10 millones de inserciones por espacio de nombres por mes como línea base.
* **Estándar**: es el nivel recomendado para aplicaciones de producción de tamaño medio a grande. Se incluyen 10 millones de dispositivos y 10 millones de inserciones por espacio de nombres por mes como línea base. Incluye telemetría enriquecida (se proporcionan datos adicionales sobre el estado de inserción).

Características del nivel Estándar:

* **Telemetría enriquecida**: puede usar Telemetría por mensaje de Notification Hubs para realizar el seguimiento de cualquier solicitud de inserción y comentarios del Sistema de notificación de plataforma para depurar.
* **Servicios multiinquilino**: puede trabajar con credenciales del Sistema de notificación de plataforma en el nivel de un espacio de nombres. Esta opción permite dividir fácilmente los inquilinos en los concentradores dentro del mismo espacio de nombres.
* **Inserción programada**: puede programar que las notificaciones se envíen en cualquier momento.
* **Operaciones masivas**: Permite la funcionalidad de exportación/importación de registros, tal como se describe en el documento [Procedimiento: cómo exportar y modificar registros en bloque].

### <a name="what-is-the-notification-hubs-sla"></a>¿Cuál es el SLA de Notification Hubs?

En el caso de los niveles Básico y Estándar de Notification Hubs, las aplicaciones configuradas correctamente pueden enviar notificaciones push o realizar operaciones de administración de registros al menos un 99,9 por ciento del tiempo. Para más información sobre el Acuerdo de Nivel de Servicio, vaya a la página [Contrato de nivel de servicio para Notification Hubs](https://azure.microsoft.com/support/legal/sla/notification-hubs/).

> [!NOTE]
> Dado que las notificaciones push dependen de Sistemas de notificación de plataforma de terceros como Apple Push Notification Service (APNs) y Google's Firebase Cloud Messaging (FCM), no hay ninguna garantía de Acuerdo de Nivel de Servicio para la entrega de estos mensajes. Una vez que Notification Hubs envía los lotes a los Sistemas de notificación de plataforma (Acuerdo de Nivel de Servicio garantizado), entregar las notificaciones es responsabilidad de los Sistemas de notificación de plataforma (no se garantiza ningún Acuerdo de Nivel de Servicio).

### <a name="how-do-i-upgrade-or-downgrade-my-hub-or-namespace-to-a-different-tier"></a>¿Cómo se puede subir o bajar de nivel mi concentrador o espacio de nombres?

Vaya a **[Azure Portal]**  > **Espacios de nombres de Notification Hubs** o **Notification Hubs**. Seleccione el recurso que desea actualizar y, luego, vaya a **Plan de tarifa**. Tenga en cuenta los siguientes requisitos:

* El plan de tarifa actualizado se aplica a *todos* los concentradores del espacio de nombres con el que esté trabajando.
* Si el número de dispositivos supera el límite del nivel al que está cambiando, deberá eliminar dispositivos antes de bajar de nivel.

## <a name="design-and-development"></a>Diseño y desarrollo

### <a name="which-server-side-platforms-do-you-support"></a>¿Qué plataformas de servicio se admiten?

Hay SDK de servidor para .NET, Java, Node.js, PHP y Python. Las API de Notification Hubs se basan en interfaces REST, por lo que puede trabajar directamente con las API de REST si usa distintas plataformas o no desea una dependencia adicional. Para más información, vaya a la página [API de REST de Notification Hubs].

### <a name="which-client-platforms-do-you-support"></a>¿Qué plataformas de cliente se admiten?

Las notificaciones de inserción son compatibles con [iOS](ios-sdk-get-started.md), [Android](notification-hubs-android-push-notification-google-fcm-get-started.md), [Windows Universal](notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md), [Windows Phone](notification-hubs-windows-mobile-push-notifications-mpns.md), [Android China (vía Baidu)](notification-hubs-baidu-china-android-notifications-get-started.md), Xamarin [iOS](xamarin-notification-hubs-ios-push-notification-apns-get-started.md) y [Android](xamarin-notification-hubs-push-notifications-android-gcm.md) y [Safari](https://github.com/Azure/azure-notificationhubs-samples/tree/master/PushToSafari). Para más información, vea la página [Tutoriales de introducción a Notification Hubs](ios-sdk-get-started.md).

### <a name="do-you-support-text-message-email-or-web-notifications"></a>¿Se admiten notificaciones por mensaje de texto, correo electrónico o web?

Notification Hubs envía notificaciones a los dispositivos que ejecutan aplicaciones móviles. No ofrece funcionalidades de correo electrónico ni mensaje de texto. Notification Hubs tampoco proporciona un servicio de entrega de notificaciones push en el explorador listo para usarse. Los clientes pueden implementar esta característica con SignalR sobre las plataformas de servidor admitidas. 

### <a name="how-many-devices-can-i-support-if-i-send-push-notifications-via-notification-hubs"></a>¿Cuántos dispositivos puedo admitir si envío notificaciones push a través de Notification Hubs?

Consulte la página [Precios de Notification Hubs] para detalles sobre la cantidad de dispositivos admitidos.

Si necesita compatibilidad con más de 10 millones de dispositivos registrados, debe crear particiones de los dispositivos en varios espacios de nombres.

### <a name="how-many-push-notifications-can-i-send-out"></a>¿Cuántas notificaciones de inserción puedo enviar?

Dependiendo del nivel seleccionado, Azure Notification Hubs realizará un escalado vertical y automático en función del número de notificaciones que fluyan a través del sistema.

> [!NOTE]
> El costo de uso total puede aumentar en función del número de notificaciones push enviadas. Asegúrese de conocer los límites de nivel que se describen en la página [Precios de Notification Hubs].

Nuestros clientes usan Notification Hubs para enviar millones de notificaciones push a diario. No es necesario hacer nada especial para escalar la cobertura de las notificaciones push, siempre y cuando esté usando Azure Notification Hubs.

### <a name="how-long-does-it-take-for-sent-push-notifications-to-reach-my-device"></a>¿Cuánto tiempo pasa desde que las notificaciones push enviadas llegan a mi dispositivo?

En un escenario de uso normal, donde la carga entrante es coherente y uniforme, Azure Notification Hubs puede procesar al menos *el envío de 1 millón de notificaciones push en un minuto*. Esta velocidad puede variar según el número de etiquetas, la naturaleza de los envíos entrantes y otros factores externos.

Durante el tiempo estimado de entrega, el servicio calcula los destinos por plataforma y enruta los mensajes Push Notification Service (PNS) según las etiquetas registradas o las expresiones de etiquetas. Enviar notificaciones al dispositivo es responsabilidad de PNS.

PNS no garantiza ningún Acuerdo de Nivel de Servicio por la entrega de notificaciones. Sin embargo, la mayoría de las notificaciones push se entregan a los dispositivos de destino en cuestión de minutos (habitualmente en 10 minutos) desde el momento en que se envían a Notification Hubs. Algunas notificaciones podrían tardar más.

> [!NOTE]
> Azure Notification Hubs tiene una directiva para retirar las notificaciones push que no se pueden entregar a PNS dentro de 30 minutos. Este retraso puede ocurrir por una serie de motivos, aunque más comúnmente se debe a que PNS está limitando la aplicación.

### <a name="is-there-any-latency-guarantee"></a>¿Hay alguna garantía de latencia?

Debido a la naturaleza de las notificaciones push (la entrega la realiza un PNS externo específico para la plataforma), no hay ninguna garantía de latencia. Habitualmente, la mayoría de las notificaciones push se entregan en cuestión de minutos.

### <a name="what-do-i-need-to-consider-when-designing-a-solution-with-namespaces-and-notification-hubs"></a>¿Qué es necesario considerar a la hora de diseñar una solución con espacios de nombres y centros de notificaciones?

#### <a name="mobile-appenvironment"></a>Entorno/aplicación móvil

* Use un centro de notificaciones por aplicación móvil por entorno.
* En un escenario multiinquilino, cada inquilino debe tener un centro independiente.
* No comparta nunca el mismo centro de notificaciones para entornos de producción y prueba. Si lo hace, podría tener problemas al enviar notificaciones. (Apple ofrece puntos de conexión de inserción de producción y espacio aislado, cada uno con credenciales independientes).
* De forma predeterminada, puede enviar notificaciones de prueba a los dispositivos registrados a través de Azure Portal o del componente de Azure integrado en Visual Studio. El umbral se establece en 10 dispositivos que se seleccionan de forma aleatoria desde el grupo de registro.

> [!NOTE]
> Si el centro se configuró originalmente con un certificado de espacio aislado de Apple y luego se reconfiguró para usar un certificado de producción de Apple, los tokens originales de dispositivo no son válidos. Los tokens no válidos generan errores en las inserciones. Separe los entornos de producción y prueba y use centros distintos para entornos diferentes.

#### <a name="pns-credentials"></a>Credenciales PNS:

Cuando una aplicación móvil se registra con un portal para desarrolladores de una plataforma (por ejemplo, Apple o Google), se envían tokens de seguridad y un identificador de aplicación. El back-end de aplicación proporciona estos tokens al PNS de la plataforma para que se puedan enviar notificaciones push a los dispositivos. Los tokens de seguridad pueden tener el formato de certificados (por ejemplo, Apple iOS o Windows Phone) o claves de seguridad (por ejemplo, Google Android o Windows). Se deben configurar en los centros de notificaciones. La configuración habitualmente se hace en el nivel de centro de notificaciones, pero también se puede hacer en el nivel de espacio de nombres en un escenario multiinquilino.

#### <a name="namespaces"></a>Espacios de nombres

Los espacios de nombres pueden usarse para la agrupación de implementación. También se pueden usar para representar todos los centros de notificaciones para todos los inquilinos de la misma aplicación en un escenario multiinquilino.

#### <a name="geo-distribution"></a>Distribución geográfica

La distribución geográfica no siempre es fundamental en el caso de las notificaciones push. Los diversos PNS (por ejemplo, APNs o FCM) que entregan notificaciones push a los dispositivos no están distribuidos de manera uniforme.

Si tiene una aplicación que se usa globalmente, puede crear centros en distintos espacios de nombres; para ello, se usa el servicio de Notification Hubs en distintas regiones de Azure de todo el mundo.

> [!NOTE]
> No se recomienda esta disposición porque aumenta el costo de administración, en especial para los registros. Solo se debe hacer si es explícitamente necesario.

### <a name="should-i-do-registrations-from-the-app-backend-or-directly-through-client-devices"></a>¿Debo hacer registros desde el back-end de aplicación o directamente a través de los dispositivos cliente?

Los registros desde el back-end de aplicación son útiles cuando tiene que autenticar clientes antes de crear el registro. También son útiles cuando se tienen etiquetas que el back-end de aplicación debe crear o modificar según la lógica de aplicación. Para más información, vaya a las páginas [Registro desde el back-end de la aplicación] y [Administración de registro].

### <a name="what-is-the-push-notification-delivery-security-model"></a>¿Cuál es el modelo de seguridad de entrega de las notificaciones push?

Azure Notification Hubs usa un modelo de seguridad basado en [firma de acceso compartido](../storage/common/storage-sas-overview.md). Puede usar los tokens de firma de acceso compartido en el nivel de espacio de nombres raíz o en el nivel de centro de notificaciones detallado. Los tokens de firma de acceso compartido se pueden establecer para que sigan distintas reglas de autorización, por ejemplo, para enviar permisos de mensajes o para escuchar los permisos de notificaciones. Para más información, consulte el documento sobre el [modelo de seguridad de Notification Hubs].

### <a name="how-should-i-handle-sensitive-payload-in-push-notifications"></a>¿Cómo debo controlar la carga confidencial de las notificaciones push?

Todas las notificaciones se entregan a los dispositivos de destino mediante el PNS de la plataforma. Cuando se envía una notificación a Azure Notification Hubs, se procesa y transmite al PNS respectivo.

Todas las conexiones, desde el remitente a Azure Notification Hubs al PNS, usan HTTPS.

> [!NOTE]
> Azure Notification Hubs no registra la carga de los mensajes.

Para enviar cargas confidenciales, se recomienda usar un patrón de inserción segura. El remitente entrega una notificación de ping con un identificador de mensaje al dispositivo sin la carga confidencial. Cuando la aplicación del dispositivo recibe la carga, llama directamente a una API segura para capturar los detalles del mensaje. Para una guía sobre cómo implementar este patrón, vaya a la página del [tutorial sobre inserciones seguras de Notification Hubs].

## <a name="operations"></a>Operaciones

### <a name="what-support-is-provided-for-disaster-recovery"></a>¿Qué compatibilidad se proporciona para la recuperación ante desastres?

En nuestro extremo se proporciona cobertura de recuperación ante desastres de metadatos (el nombre de Notification Hubs, la cadena de conexión y otra información crítica). Cuando se desencadena un escenario de recuperación ante desastres, los datos de registro son el *único segmento* de la infraestructura de Notification Hubs que se pierde. Debe implementar una solución para volver a rellenar estos datos en el nuevo centro posterior a la recuperación:

1. Cree un centro de notificaciones secundario en otro centro de datos. Se recomienda que cree uno desde el comienzo para protegerse de un evento de recuperación ante desastres que pudiera afectar sus funcionalidades de administración. También puede crear uno en el momento del evento de recuperación ante desastres.

2. Rellene el centro de notificaciones secundario con los registros del centro de notificaciones principal. No se recomienda que intente conservar los registros en ambos centros y mantenerlos sincronizados a medida que se ingresan registros. Esta práctica no funciona bien debido a la tendencia inherente que tienen los registros de expirar en el lado del PNS. Notification Hubs los limpia cuando recibe informaciones del PNS con respecto a los registros expirados o no válidos.  

Hay dos recomendaciones para los back-ends de aplicación:

* Use un back-end de aplicación que mantenga un conjunto determinado de registros en su extremo. Luego puede realizar una inserción masiva en el centro de notificaciones secundario.
* Use un back-end de aplicación que obtenga un volcado habitual de registros desde el centro de notificaciones principal como copia de seguridad. Luego puede realizar una inserción masiva en el centro de notificaciones secundario.

> [!NOTE]
> La funcionalidad de exportación/importación de registros disponible en el nivel Estándar se describe en el documento sobre [Procedimiento: cómo exportar y modificar registros en bloque].

Si no tiene un back-end, cuando la aplicación se inicie en los dispositivos de destino, realizan un registro nuevo en el centro de notificaciones secundario. A la larga, el centro de notificaciones secundario tendrá registrados todos los dispositivos activos.

Habrá un período en el que los dispositivos que no tengan aplicaciones abiertas no recibirán notificaciones.

### <a name="is-all-of-my-data-stored-in-encrypted-form"></a>¿Están todos mis datos almacenados en un formato cifrado?

Azure Notification Hubs cifra todos los datos en reposo del cliente a excepción de las etiquetas de registro. Por esta razón, no debe almacenar datos personales o confidenciales mediante etiquetas.

### <a name="is-there-audit-log-capability"></a>¿Hay alguna funcionalidad de registro de auditoría?

Sí. Todas las operaciones de administración de Notification Hubs actualizan el registro de actividad de Azure que se expone en [Azure Portal]. El registro de actividad de Azure proporciona información sobre las operaciones que se realizan en los recursos de las suscripciones. Con el registro de actividad, se puede dar respuesta a los interrogantes qué, quién y cuándo de las operaciones de escritura (PUT, POST y DELETE) realizadas en los recursos de la suscripción. También permite entender el estado de las operaciones y otras propiedades relevantes. Sin embargo, el registro de actividad no incluye las operaciones de lectura (GET).

## <a name="monitoring-and-troubleshooting"></a>Supervisión y solución de problemas

### <a name="what-troubleshooting-capabilities-are-available"></a>¿Qué funcionalidades de solución de problemas hay disponibles?

Azure Notification Hubs proporciona varias características de solución de problemas, especialmente para el escenario más común relacionado con las notificaciones quitadas. Para detalles, consulte las notas del producto sobre la [solución de problemas de Notification Hubs].

### <a name="what-telemetry-features-are-available"></a>¿Qué características de telemetría hay disponibles?

Azure Notification Hubs le permite ver los datos de telemetría en [Azure Portal]. Puede encontrar detalles sobre las métricas en la página [Métricas de Notification Hubs].

También puede acceder a las métricas mediante programación. Para más información, consulte los siguientes artículos.

- [Retrieve Azure Monitor metrics with .NET](https://azure.microsoft.com/resources/samples/monitor-dotnet-metrics-api/) (Recuperación de métricas de Azure Monitor con .NET). En este ejemplo, se utiliza el nombre de usuario y la contraseña. Si quiere usar un certificado, sobrecargue el método FromServicePrincipal para proporcionar un certificado, tal y como se muestra en [este ejemplo](https://github.com/Azure/azure-libraries-for-net/blob/master/src/ResourceManagement/ResourceManager/Authentication/AzureCredentialsFactory.cs). 
- [Getting metrics and activity logs for a resource](https://azure.microsoft.com/resources/samples/monitor-dotnet-query-metrics-activitylogs/) (Obtención de métricas y registros de actividad de un recurso)
- [Tutorial sobre la API de REST de supervisión de Azure](../azure-monitor/platform/rest-api-walkthrough.md)

> [!NOTE]
> La entrega correcta de las notificaciones significa simplemente que las notificaciones push se entregaron al PNS externo (por ejemplo APNs para iOS y macOS, o FCM para dispositivos Android). Entregar las notificaciones a los dispositivos de destino es responsabilidad del PNS. Habitualmente, el PNS no muestra las métricas de entrega a terceros.  

[Azure Portal]: https://portal.azure.com
[Precios de Notification Hubs]: https://azure.microsoft.com/pricing/details/notification-hubs/
[Notification Hubs SLA]: https://azure.microsoft.com/support/legal/sla/
[API de REST de Notification Hubs]: /previous-versions/azure/reference/dn530746(v=azure.100)
[Mobile Services Pricing]: https://azure.microsoft.com/pricing/details/mobile-services/
[Registro desde el back-end de la aplicación]: /previous-versions/azure/azure-services/dn743807(v=azure.100)
[Administración de registro]: /previous-versions/azure/azure-services/dn530747(v=azure.100)
[Modelo de seguridad de Notification Hubs]: /previous-versions/azure/azure-services/dn495373(v=azure.100)
[Tutorial sobre inserciones seguras de Notification Hubs]: ./notification-hubs-aspnet-backend-ios-push-apple-apns-secure-notification.md
[Solución de problemas de Notification Hubs]: ./notification-hubs-push-notification-fixer.md
[Métricas de Notification Hubs]: ../azure-monitor/platform/metrics-supported.md#microsoftnotificationhubsnamespacesnotificationhubs
[Procedimiento: cómo exportar y modificar registros en bloque]: ./export-modify-registrations-bulk.md
[Azure Portal]: https://portal.azure.com
[complete samples]: https://github.com/Azure/azure-notificationhubs-samples
[App Service Pricing]: https://azure.microsoft.com/pricing/details/app-service/
