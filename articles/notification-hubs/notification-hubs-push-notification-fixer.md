---
title: Diagnóstico de notificaciones eliminadas en Azure Notification Hubs
description: Aprenda a diagnosticar problemas comunes con las notificaciones eliminadas de Azure Notification Hubs.
services: notification-hubs
documentationcenter: Mobile
author: sethmanheim
manager: femila
editor: jwargo
ms.assetid: b5c89a2a-63b8-46d2-bbed-924f5a4cce61
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: NA
ms.devlang: multiple
ms.topic: article
ms.date: 02/25/2020
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 04/04/2019
ms.openlocfilehash: 3bad65dbecd6e4a7d9bcb7fbdee6faafc31ead86
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87078021"
---
# <a name="diagnose-dropped-notifications-in-azure-notification-hubs"></a>Diagnóstico de notificaciones eliminadas en Azure Notification Hubs

Una pregunta común sobre Azure Notification Hubs es cómo solucionar problemas cuando las notificaciones de una aplicación no aparecen en los dispositivos del cliente. Los clientes quieren saber dónde y por qué se eliminaron las notificaciones y cómo solucionar el problema. En este artículo se identifica por qué las notificaciones pueden ser eliminadas o no recibidas por los dispositivos. También se explica cómo determinar la causa principal.

Es fundamental entender primero cómo Notification Hubs inserta notificaciones en un dispositivo.

![Arquitectura de Notification Hubs][0]

En un flujo de notificaciones de envío típico, el mensaje se envía desde el *back-end de la aplicación* a Notification Hubs. Notification Hubs procesa todos los registros. Tiene en cuenta las etiquetas y expresiones de etiquetas configuradas para determinar los destinos. Los destinos son los registros que necesitan recibir la notificación push. Estos registros pueden comprender cualquiera de nuestras plataformas compatibles: Android, Baidu (dispositivos Android en China), Fire OS (Amazon), iOS, Windows y Windows Phone.

Con los destinos establecidos, Notification Hubs inserta las notificaciones en el *servicio de notificaciones push* para la plataforma del dispositivo. Algunos ejemplos incluyen Apple Push Notification Service (APNs) para iOS y macOS, y Firebase Cloud Messaging (FCM) para dispositivos Android. Notification Hubs lleva a las notificaciones a dividirse en varios lotes de registros. Se autentica con el servicio de notificaciones push respectivo basado en las credenciales que se establecieron en Azure Portal, en **Configuración del centro de notificaciones**. Luego, el servicio de notificaciones de inserción reenvía las notificaciones a los respectivos *dispositivos cliente*.

El tramo final de la entrega de notificaciones tiene lugar entre el servicio de notificaciones push de la plataforma y el dispositivo. La entrega de la notificación se puede producir en cualquiera de las cuatro fases del proceso de notificación push (cliente, back-end de la aplicación, Notification Hubs y servicio de notificaciones push de la plataforma). Para más información sobre la arquitectura de Notification Hubs, consulte [Introducción a Notification Hubs].

Durante la fase inicial de prueba o de ensayo puede ocurrir que no se entreguen las notificaciones. Las notificaciones eliminadas en esta fase podrían indicar un problema de configuración. Si se produce un error en la entrega de notificaciones durante producción, es posible que se eliminen algunas o todas las notificaciones. En este caso, se indica una aplicación más profunda o un problema de patrón de mensajería.

La siguiente sección examina los escenarios en los que las notificaciones podrían resultar eliminadas, desde los más comunes hasta los poco frecuentes.

## <a name="notification-hubs-misconfiguration"></a>Configuración incorrecta de Notification Hubs

Para poder enviar notificaciones al servicio de notificaciones push, Notification Hubs debe autenticarse a sí mismo en el contexto de la aplicación. Debe crear una cuenta de desarrollador con el servicio de notificaciones de la plataforma de destino (Microsoft, Apple, Google, etc.). A continuación, debe registrar la aplicación con el sistema operativo donde obtener un token o una clave que se utiliza para trabajar con el PNS de destino.

Debe agregar las credenciales de plataforma en Azure Portal. Si no llega ninguna notificación al dispositivo, el primer paso debe ser asegurarse de que estén configuradas las credenciales correctas en Notification Hubs. Las credenciales deben coincidir con la aplicación creada en una cuenta de desarrollador específica de plataforma.

Para obtener instrucciones paso a paso para completar este proceso, consulte [Introducción a Azure Notification Hubs].

Estos son algunos errores comunes de configuración para comprobar:

### <a name="notification-hub-name-location"></a>La ubicación del nombre del centro de notificaciones.

Asegúrese de que el nombre de su centro de notificaciones (escrito sin errores) sea el mismo en cada una de estas ubicaciones:

* Donde se registra en el cliente.
* Donde envía notificaciones desde el back-end.
* Donde ha configurado las credenciales de servicios de notificaciones push.

Asegúrese de que usa las cadenas de configuración de firmas de acceso compartido correctas en el cliente y en el back-end de la aplicación. Por lo general, debe usar **DefaultListenSharedAccessSignature** en el cliente y **DefaultFullSharedAccessSignature** en el back-end de la aplicación. Esto concede permisos para enviar notificaciones a Notification Hubs.

### <a name="apn-configuration"></a>Configuración de APN

Debe mantener dos centros diferentes: uno para producción y otro para pruebas. Debe cargar el certificado que va a usar en un entorno de espacio aislado en un centro distinto del certificado o del centro que usará en producción. No intente cargar diferentes tipos de certificados en el mismo centro, ya que provocará errores de notificación.

Si carga involuntariamente diferentes tipos de certificados en el mismo centro, debe eliminar el centro y volver a empezar con uno nuevo. Si, por algún motivo, no puede eliminar el centro, debe eliminar al menos todos los registros existentes del centro.

### <a name="fcm-configuration"></a>Configuración de FCM

1. Asegúrese de que la *clave de servidor* que obtuvo de Firebase coincide con la clave de servidor registrada en Azure Portal.

   ![Clave de servidor de Firebase][3]

2. Asegúrese de que ha configurado el **identificador de proyecto** en el cliente. Puede obtener el valor del **identificador de proyecto** en el panel de Firebase.

   ![Identificador del proyecto de Firebase][1]

## <a name="application-issues"></a>Problemas de la aplicación

### <a name="tags-and-tag-expressions"></a>Etiquetas y expresiones de etiqueta

Si usa etiquetas o expresiones de etiqueta para segmentar la audiencia, es posible que cuando envía la notificación no se encuentre ningún destino. Este error se basa en las etiquetas o expresiones de etiqueta que especifica en la llamada de envío.

Revise los registros para asegurarse de que las etiquetas coincidan cuando envíe una notificación. Después, compruebe la recepción de la notificación solo de los clientes que tienen esos registros.

Por ejemplo, supongamos que todos los registros con Notification Hubs usan la etiqueta "Política". Si, a continuación, envía una notificación con la etiqueta "Deportes", no se enviará la notificación a ningún dispositivo. Un caso complejo podría incluir expresiones de etiqueta donde se registró con "Etiqueta A" *o* "Etiqueta B", pero dirigió "Etiqueta A && Etiqueta B". En la sección de sugerencias de autodiagnóstico, más adelante en el artículo, se muestra cómo revisar los registros y sus etiquetas.

### <a name="template-issues"></a>Problemas de plantillas

Si utiliza plantillas, asegúrese de seguir las directrices descritas en [Templates].

### <a name="invalid-registrations"></a>Registros no válidos

Si el centro de notificaciones se ha configurado correctamente y las etiquetas o expresiones de etiqueta se han usado correctamente, se encuentran destinos válidos. Las notificaciones se enviarán a estos destinos. Después, Notification Hubs desactiva varios lotes de procesamiento en paralelo. Cada lote envía mensajes a un conjunto de registros.

> [!NOTE]
> Puesto que Notification Hubs procesa los lotes en paralelo, no se garantiza el orden en el que se entregan las notificaciones.

Notification Hubs está optimizado para un modelo de entrega de mensajes "una vez como máximo". Intentamos una desduplicación para que ninguna notificación se entregue más de una vez a un dispositivo. Los registros se comprueban para asegurarnos de que solo se envía un mensaje por identificador de dispositivo antes de que se envíe al servicio de notificaciones push.

Cada lote se envía al servicio de notificaciones push que, a su vez, acepta y valida los registros. Durante este proceso, es posible que el servicio de notificaciones push detecte un error con uno o varios registros en un lote. El servicio de notificaciones push devuelve un error a Notification Hubs y el proceso se detiene. El servicio de notificaciones push elimina ese lote por completo. Esto ocurre así con APNs, que usa un protocolo de transmisión TCP.

En este caso, el registro de errores se quita de la base de datos. A continuación, volvemos a intentar la entrega de notificaciones para el resto de los dispositivos de ese lote.

Para obtener más información del error del intento de entrega erróneo en un registro, pude usar las API REST de Notification Hubs: [Telemetría por mensaje: Obtención de telemetría de mensaje de notificación](/rest/api/notificationhubs/get-notification-message-telemetry) y [Comentarios de PNS](/previous-versions/azure/reference/mt705560(v=azure.100)). Para ver un ejemplo de código, consulte el [ejemplo de REST de envío](https://github.com/Azure/azure-notificationhubs-dotnet/tree/master/Samples/SendRestExample/).

## <a name="push-notification-service-issues"></a>Problemas del servicio de notificaciones push

Una vez que el servicio de notificaciones push recibe la notificación, la envía al dispositivo. En este punto, Notification Hubs no tiene ningún control sobre la entrega de la notificación al dispositivo.

Como los servicios de notificaciones de plataforma son bastante sólidos, las notificaciones tienden a llegar a los dispositivos en unos segundos. Si se está limitando servicio de notificaciones push, Notification Hubs aplica una estrategia de retroceso exponencial. Si el servicio de notificaciones push sigue inaccesible durante 30 minutos, hay una directiva establecida para hacer que esos mensajes expiren y se eliminen permanentemente.

Si un servicio de notificaciones push intenta entregar una notificación pero el dispositivo está sin conexión, el servicio de notificaciones push almacena la notificación solo durante un período de tiempo limitado. La notificación se entrega al dispositivo cuando este está disponible.

Cada aplicación almacena solo una notificación reciente. Si se envían varias notificaciones mientras el dispositivo está sin conexión, cada nueva notificación provoca que se descarte la más reciente. Mantener solo la notificación más reciente se denomina *fusión* en APNs y *contracción* en FCM. (FCM usa una clave de contracción). Cuando el dispositivo permanece sin conexión durante un período de tiempo prolongado, las notificaciones que se almacenaron para el dispositivo se descartan. Para más información, consulte [APNs overview] (Introducción a Apple Push Notification Service) y [Acerca de los mensajes de FCM].

Con Notification Hubs, puede pasar una clave de fusión a través de un encabezado HTTP mediante el uso de SendNotification API genérica. Por ejemplo, para el SDK de .NET, debería usar `SendNotificationAsync`. SendNotification API también toma los encabezados HTTP que se pasan tal cual al servicio de notificaciones push respectivo.

## <a name="self-diagnosis-tips"></a>Sugerencias de autodiagnóstico

Estas son rutas de acceso para diagnosticar la causa principal de las notificaciones eliminadas en Notification Hubs.

### <a name="verify-credentials"></a>Comprobar las credenciales

#### <a name="push-notification-service-developer-portal"></a>Portal para desarrolladores del servicio de notificaciones push

Compruebe las credenciales en el portal para desarrolladores de los servicios de notificaciones push respectivos (APN, FCM, servicio de notificaciones de Windows, etc.). Para más información, consulte el [Tutorial: Envío de notificaciones a aplicaciones de Plataforma universal de Windows mediante Azure Notification Hubs](./notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md).

#### <a name="azure-portal"></a>Portal de Azure

Para revisar y comparar las credenciales con las que obtuvo en el portal para desarrolladores de los servicios de notificaciones push, vaya a la pestaña **Directivas de acceso** en Azure Portal.

![Directivas de acceso de Azure Portal][4]

### <a name="verify-registrations"></a>Verificar los registros

#### <a name="visual-studio"></a>Visual Studio

En Visual Studio, puede conectarse a Azure mediante el Explorador de servidores para ver y administrar muchos servicios de Azure, incluido Notification Hubs. Este acceso directo es especialmente útil para su entorno de desarrollo y prueba.

![Explorador de servidores de Visual Studio][9]

![Explorador de servidores](media/notification-hubs-push-notification-fixer/vsserverexplorer2.png)

Puede ver y administrar todos los registros de su centro. Los registros se pueden clasificar por plataforma, registro nativo o de plantilla, etiqueta, identificador del servicio de notificaciones push, identificador de registro y fecha de expiración. También puede editar un registro en esta página. Esto es especialmente útil para editar etiquetas.

Haga clic con el botón derecho en el centro de notificaciones en **Explorador de servidores** y seleccione **Diagnosticar**. 

![Explorador de servidores de Visual Studio: Menú de diagnóstico](./media/notification-hubs-push-notification-fixer/diagnose-menu.png)

Verá la página siguiente:

![Visual Studio: Página de diagnóstico](./media/notification-hubs-push-notification-fixer/diagnose-page.png)

Cambie a la página **Registros de dispositivos**:

![Visual Studio: Registros de dispositivos](./media/notification-hubs-push-notification-fixer/VSRegistrations.png)

Puede usar la página **Envío de prueba** para enviar un mensaje de notificación de prueba:

![Visual Studio: Envío de prueba](./media/notification-hubs-push-notification-fixer/test-send-vs.png)

> [!NOTE]
> Use Visual Studio para editar registros solo durante las pruebas o el desarrollo, y con un número limitado de registros. Si necesita modificar sus registros en bloque, considere la posibilidad de usar la función para exportar o importar registros que se describe en [Procedimiento: exportar y modificar registros en bloque](/previous-versions/azure/azure-services/dn790624(v=azure.100)).

#### <a name="service-bus-explorer"></a>Explorador de Service Bus

Muchos clientes usan el [Explorador de Service Bus](https://github.com/paolosalvatori/ServiceBusExplorer) para consultar y administrar su centro de notificaciones. El Explorador de Service Bus es un proyecto de código abierto. 

### <a name="verify-message-notifications"></a>Comprobar las notificaciones de mensajes

#### <a name="azure-portal"></a>Portal de Azure

Para enviar una notificación de prueba a los clientes sin que el back-end del servicio esté en funcionamiento, en **SOPORTE TÉCNICO Y SOLUCIÓN DE PROBLEMAS**, seleccione **Envío de prueba**.

![Funcionalidad Envío de prueba en Azure][7]

#### <a name="visual-studio"></a>Visual Studio

También puede enviar notificaciones de prueba desde Visual Studio.

![Funcionalidad Envío de prueba en Visual Studio][10]

Para más información acerca del uso de Notification Hubs con el Explorador de servidores de Visual Studio, consulte estos artículos:

* [Cómo ver los registros de dispositivos de los centros de notificaciones](/previous-versions/windows/apps/dn792122(v=win.10))
* [En profundidad: Visual Studio 2013 Update 2 RC y Azure SDK 2.3] (Análisis a fondo: Visual Studio 2013 Update 2 RC y Azure SDK 2.3)
* [Announcing release of Visual Studio 2013 Update 3 and Azure SDK 2.4] (Anuncio del lanzamiento de Visual Studio 2013 Update 3 y Azure SDK 2.4)

### <a name="debug-failed-notifications-and-review-notification-outcome"></a>Depuración de las notificaciones incorrectas y revisión del resultado de la notificación

#### <a name="enabletestsend-property"></a>Propiedad EnableTestSend

Cuando se envía una notificación mediante Notification Hubs, la notificación se pone en cola inicialmente. Notification Hubs determina los destinos correctos y después envía la notificación al servicio de notificaciones push. Si usa la API REST o cualquiera de los SDK de cliente, la devolución de su llamada de envío solo implica que el mensaje se ha puesto en cola en Notification Hubs. No proporciona información de lo que ocurrió cuando Notification Hubs envió finalmente la notificación al servicio de notificaciones push.

Si la notificación no llega al dispositivo cliente, es posible que se haya producido un error cuando Notification Hubs intentó enviarla al servicio de notificaciones push. Por ejemplo, el tamaño de la carga podría exceder del máximo permitido por el servicio de notificaciones push, o las credenciales configuradas en Notification Hubs podrían no ser válidas.

Para obtener información sobre los errores del servicio de notificaciones push, puede utilizar la propiedad [EnableTestSend]. Esta propiedad se habilita automáticamente cuando envía mensajes de prueba desde el portal o el cliente de Visual Studio. Puede usar esta propiedad para ver la información de depuración detallada, así como a través de API. Actualmente, se puede utilizar en .NET SDK. Finalmente, se agregará a todos los SDK de cliente.

Para usar la propiedad `EnableTestSend` con la llamada a REST, anexe un parámetro de cadena de consulta llamado *test* al final de su llamada de envío. Por ejemplo:

```text
https://mynamespace.servicebus.windows.net/mynotificationhub/messages?api-version=2013-10&test
```

#### <a name="net-sdk-example"></a>Ejemplo de SDK de .NET

Este es un ejemplo de uso de .NET SDK para enviar una notificación emergente nativa (notificación del sistema):

```csharp
NotificationHubClient hub = NotificationHubClient.CreateClientFromConnectionString(connString, hubName);
var result = await hub.SendWindowsNativeNotificationAsync(toast);
Console.WriteLine(result.State);
```

Al final de la ejecución, `result.State` sólo indica `Enqueued`. Los resultados no proporcionan ninguna información sobre qué ha ocurrido con la notificación push.

A continuación, puede usar la propiedad booleana `EnableTestSend`. Use la propiedad `EnableTestSend` mientras se inicializa `NotificationHubClient` para obtener el estado detallado de los errores del servicio de notificaciones push que se producen cuando se envía la notificación. La llamada de envío tarda un tiempo adicional en devolverse porque primero es necesario que Notification Hubs entregue la notificación al servicio de notificaciones push.

```csharp
    bool enableTestSend = true;
    NotificationHubClient hub = NotificationHubClient.CreateClientFromConnectionString(connString, hubName, enableTestSend);

    var outcome = await hub.SendWindowsNativeNotificationAsync(toast);
    Console.WriteLine(outcome.State);

    foreach (RegistrationResult result in outcome.Results)
    {
        Console.WriteLine(result.ApplicationPlatform + "\n" + result.RegistrationId + "\n" + result.Outcome);
    }
```

#### <a name="sample-output"></a>Salida de ejemplo

```text
DetailedStateAvailable
windows
7619785862101227384-7840974832647865618-3
The Token obtained from the Token Provider is wrong
```

Este mensaje indica que se han configurado credenciales no válidas en Notification Hubs o que hay un problema con los registros en el centro. Elimine este registro y deje que el cliente vuelva a crear el registro antes de enviar el mensaje.

> [!NOTE]
> El uso de la propiedad `EnableTestSend` está muy limitado. Use esta opción únicamente en un entorno de desarrollo y pruebas y con un conjunto limitado de registros. Las notificaciones de depuración se envían únicamente a 10 dispositivos. También hay un límite de procesamiento de envíos de depuración (10 por minuto).

### <a name="review-telemetry"></a>Revisar la telemetría

#### <a name="azure-portal"></a>Portal de Azure

En el portal, puede obtener una rápida introducción de toda la actividad que tiene lugar en el centro de notificaciones.

1. En la pestaña **Información general**, puede ver una vista agregada de registros, notificaciones y errores por plataforma.

   ![Panel de introducción a Notification Hubs][5]

2. En la pestaña **Supervisión**, puede agregar muchas otras métricas específicas de la plataforma para obtener una visión más profunda. Puede ver específicamente los errores que se devuelven cuando Notification Hubs intenta enviar la notificación al servicio de notificaciones push.

   ![Registro de actividad de Azure Portal][6]

3. Comience por revisar **los mensajes entrantes**, **las operaciones de registro** y las **notificaciones correctas**. Después, vaya a la pestaña por plataforma para revisar los errores específicos del servicio de notificaciones push.

4. Si la configuración de autenticación para el centro de notificaciones no es correcta, se muestra el mensaje **Error de autenticación PNS**. Es un buen indicio de que se deben comprobar las credenciales del servicio de notificaciones push.

#### <a name="programmatic-access"></a>Acceso mediante programación

Para más información sobre el acceso mediante programación, consulte [Acceso mediante programación](/previous-versions/azure/azure-services/dn458823(v=azure.100)).

> [!NOTE]
> Varias características relacionadas con la telemetría, como la exportación e importación de registros y el acceso a la telemetría a través de API, solo están disponibles en el nivel de servicio Estándar. Si intenta usar estas características del nivel de servicio Gratis o Básico, recibirá un mensaje de excepción si usa el SDK. Si usa las características directamente desde las API REST, recibirá un error HTTP 403 (Prohibido).
>
> Para usar las características relacionadas con la telemetría, asegúrese primero en Azure Portal de que está usando el nivel de servicio Estándar.  

<!-- IMAGES -->
[0]: ./media/notification-hubs-push-notification-fixer/Architecture.png
[1]: ./media/notification-hubs-push-notification-fixer/FCMConfigure.png
[3]: ./media/notification-hubs-push-notification-fixer/FCMServerKey.png
[4]: ../../includes/media/notification-hubs-portal-create-new-hub/notification-hubs-connection-strings-portal.png
[5]: ./media/notification-hubs-push-notification-fixer/PortalDashboard.png
[6]: ./media/notification-hubs-push-notification-fixer/PortalAnalytics.png
[7]: ./media/notification-hubs-ios-get-started/notification-hubs-test-send.png
[8]: ./media/notification-hubs-push-notification-fixer/VSRegistrations.png
[9]: ./media/notification-hubs-push-notification-fixer/vsserverexplorer.png
[10]: ./media/notification-hubs-push-notification-fixer/VSTestNotification.png

<!-- LINKS -->
[Introducción a Notification Hubs]: notification-hubs-push-notification-overview.md
[Introducción a Azure Notification Hubs]: notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md
[Templates]: /previous-versions/azure/azure-services/dn530748(v=azure.100) (Plantillas [C++])
[APNs overview]: https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html (Introducción a Apple Push Notification Service)
[Acerca de los mensajes de FCM]: https://firebase.google.com/docs/cloud-messaging/concept-options
[Export and modify registrations in bulk]: /previous-versions/azure/azure-services/dn790624(v=azure.100)
[Service Bus Explorer code]: https://code.msdn.microsoft.com/windowsazure/Service-Bus-Explorer-f2abca5a
[View device registrations for notification hubs]: /previous-versions/windows/apps/dn792122(v=win.10)
[En profundidad: Visual Studio 2013 Update 2 RC y Azure SDK 2.3]: https://azure.microsoft.com/blog/2014/04/09/deep-dive-visual-studio-2013-update-2-rc-and-azure-sdk-2-3/#NotificationHubs (Análisis a fondo: Visual Studio 2013 Update 2 RC y Azure SDK 2.3)
[Announcing release of Visual Studio 2013 Update 3 and Azure SDK 2.4]: https://azure.microsoft.com/blog/2014/08/04/announcing-release-of-visual-studio-2013-update-3-and-azure-sdk-2-4/ (Anuncio del lanzamiento de Visual Studio 2013 Update 3 y Azure SDK 2.4)
[EnableTestSend]: /dotnet/api/microsoft.azure.notificationhubs.notificationhubclient.enabletestsend?view=azure-dotnet
[Programmatic telemetry access]: /previous-versions/azure/azure-services/dn458823(v=azure.100)
