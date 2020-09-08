---
title: Enrutamiento y etiquetado de expresiones en Azure Notification Hubs
description: Aprenda a enrutar y etiquetar expresiones para Azure Notification Hubs.
services: notification-hubs
documentationcenter: .net
author: sethmanheim
manager: femila
editor: jwargo
ms.assetid: 0fffb3bb-8ed8-4e0f-89e8-0de24a47f644
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-multiple
ms.devlang: dotnet
ms.topic: article
ms.date: 12/09/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 12/04/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 4ff8225522e79e2be40682fb5e4823777dde2aa0
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88998310"
---
# <a name="routing-and-tag-expressions"></a>Expresiones de etiqueta y enrutamiento

## <a name="overview"></a>Información general

Las expresiones de etiqueta le permiten dirigirse a conjuntos específicos de dispositivos, o más específicamente a registros, al enviar una notificación push a través de Notification Hubs.

## <a name="targeting-specific-registrations"></a>Selección del destino de registros específicos

La única forma de seleccionar el destino de registros de notificaciones específicos es asociar etiquetas con ellos y, a continuación, seleccionar el destino de esas etiquetas. Como se describe en [Administración de registros](notification-hubs-push-notification-registration-management.md), para recibir notificaciones de inserción, las aplicaciones deben registrar un identificador de dispositivo en un centro de notificaciones. Una vez que la aplicación crea un registro en un centro de notificaciones, el back-end de la aplicación puede enviar notificaciones de inserción a este. El back-end de la aplicación puede elegir a qué registros dirigirse con una notificación específica de las maneras siguientes:

1. **Difusión**: todos los registros del centro de notificaciones reciben la notificación.
2. **Etiquetar**: todos los registros que contienen la etiqueta especificada reciben la notificación.
3. **Expresión de etiqueta**: todos los registros cuyo conjunto de etiquetas coincide con la expresión especificada reciben la notificación.

## <a name="tags"></a>Etiquetas

Una etiqueta puede ser cualquier cadena, de hasta 120 caracteres, que contenga caracteres alfanuméricos y los siguientes caracteres no alfanuméricos: "`_`", "`@`", "`#`", "`.`", "`:`", "`-`". En el ejemplo siguiente se muestra una aplicación desde la que puede recibir notificaciones del sistema sobre grupos musicales específicos. En este escenario, una manera sencilla de enrutar notificaciones es usar en los registros etiquetas que representen las distintas bandas, como se muestra en la siguiente ilustración:

![Información general de las etiquetas](./media/notification-hubs-tags-segment-push-message/notification-hubs-tags.png)

En la ilustración, el mensaje con la etiqueta **Beatles** llega solamente a la tableta registrada con la etiqueta **Beatles**.

Para obtener más información acerca de la creación de registros de etiquetas, consulte [Administración de registros](notification-hubs-push-notification-registration-management.md).

Puede enviar notificaciones a etiquetas mediante los métodos de las notificaciones de envío de la clase `Microsoft.Azure.NotificationHubs.NotificationHubClient` en el SDK de [Microsoft Azure Notification Hubs](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/) . También puede usar Node.js o las API de REST de notificaciones de inserción.  A continuación se facilita un ejemplo mediante el uso del SDK.

```csharp
Microsoft.Azure.NotificationHubs.NotificationOutcome outcome = null;

// Windows 8.1 / Windows Phone 8.1
var toast = @"<toast><visual><binding template=""ToastText01""><text id=""1"">" +
"You requested a Beatles notification</text></binding></visual></toast>";
outcome = await Notifications.Instance.Hub.SendWindowsNativeNotificationAsync(toast, "Beatles");

// Windows 10
toast = @"<toast><visual><binding template=""ToastGeneric""><text id=""1"">" +
"You requested a Wailers notification</text></binding></visual></toast>";
outcome = await Notifications.Instance.Hub.SendWindowsNativeNotificationAsync(toast, "Wailers");
```

Las etiquetas no deben aprovisionarse previamente y pueden hacer referencia a varios conceptos específicos de la aplicación. Por ejemplo, los usuarios de esta aplicación de ejemplo pueden comentar sobre bandas y desean recibir notificaciones del sistema, no solo de los comentarios sobre sus bandas preferidas, sino también de todos los comentarios de sus amigos, independientemente de la banda sobre la que estén comentando. En la siguiente ilustración se muestra un ejemplo de este escenario:

![Etiquetar amigos](./media/notification-hubs-tags-segment-push-message/notification-hubs-tags2.png)

En este ejemplo, Ana está interesada en recibir actualizaciones sobre los Beatles, mientras que Pedro está interesado en las actualizaciones sobre los Wailers. Pedro también está interesado en los comentarios de Juan Carlos, y Juan Carlos está interesado en los Wailers. Cuando se envía una notificación sobre el comentario de Juan Carlos acerca de los Beatles, Notification Hubs la envía tanto a Ana como a Pedro.

Aunque se pueden codificar varios intereses en etiquetas (por ejemplo, `band_Beatles` o `follows_Charlie`), estas son cadenas simples, no propiedades con valores. Los registros coinciden solo con la presencia o ausencia de una etiqueta concreta.

Para obtener un tutorial completo detallado sobre cómo usar etiquetas para enviar a grupos de interés, consulte [Noticias de última hora](notification-hubs-windows-notification-dotnet-push-xplat-segmented-wns.md).

> [!NOTE]
> Azure Notification Hubs admite un máximo de 60 etiquetas por registro.

## <a name="using-tags-to-target-users"></a>Uso de etiquetas para los usuarios de destino

Otra forma de usar etiquetas es identificar todos los dispositivos asociados a un usuario concreto. Los registros se pueden etiquetar con una etiqueta que contenga el identificador de usuario, como se muestra en la siguiente ilustración:

![Etiquetar usuarios](./media/notification-hubs-tags-segment-push-message/notification-hubs-tags3.png)

En la ilustración, el mensaje con la etiqueta `user_Alice` llega a todos los dispositivos con la etiqueta `user_Alice`.

## <a name="tag-expressions"></a>Expresiones de etiqueta

Hay casos en los que las notificaciones deben tener como destino un conjunto de registros identificados no por una etiqueta individual, sino por una expresión booleana que usa etiquetas.

Tome una aplicación de deportes que envía un recordatorio a todas las personas de Boston sobre un partido entre los Red Sox y los Cardinals. Si la aplicación cliente registra etiquetas sobre intereses en equipos y ubicaciones, la notificación deberá dirigirse a todas las personas de Boston interesadas en los Red Sox o en los Cardinals. Esta condición se puede expresar mediante la siguiente expresión booleana:

```csharp
(follows_RedSox || follows_Cardinals) && location_Boston
```

![Expresiones de etiqueta](./media/notification-hubs-tags-segment-push-message/notification-hubs-tags4.png)

Las expresiones de etiqueta admiten operadores booleanos comunes como `AND` (`&&`), `OR` (`||`) y `NOT` (`!`); también pueden contener paréntesis. Las expresiones de etiqueta que usan solo operadores `OR` pueden hacer referencia a 20 etiquetas; la expresión con operadores `AND`, pero ningún operador `OR`, puede hacer referencia a 10 etiquetas; de lo contrario, las expresiones de etiqueta se limitan a 6 etiquetas.

A continuación se muestra un ejemplo de envío de notificaciones con expresiones de etiqueta mediante el SDK:

```csharp
Microsoft.Azure.NotificationHubs.NotificationOutcome outcome = null;

String userTag = "(location_Boston && !follows_Cardinals)";

// Windows 8.1 / Windows Phone 8.1
var toast = @"<toast><visual><binding template=""ToastText01""><text id=""1"">" +
"You want info on the Red Sox</text></binding></visual></toast>";
outcome = await Notifications.Instance.Hub.SendWindowsNativeNotificationAsync(toast, userTag);

// Windows 10
toast = @"<toast><visual><binding template=""ToastGeneric""><text id=""1"">" +
"You want info on the Red Sox</text></binding></visual></toast>";
outcome = await Notifications.Instance.Hub.SendWindowsNativeNotificationAsync(toast, userTag);
```
