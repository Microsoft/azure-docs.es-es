---
title: Actualizaciones de iOS 13 para Azure Notification Hubs | Microsoft Docs
description: Información sobre los cambios importantes de iOS 13 en Azure Notification Hubs
author: sethmanheim
ms.author: sethm
ms.date: 10/16/2019
ms.topic: article
ms.service: notification-hubs
ms.reviewer: jowargo
ms.lastreviewed: 10/16/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: df8560bec3671a9f05628ee6ed8ea95c31e9b16f
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88998057"
---
# <a name="azure-notification-hubs-updates-for-ios-13"></a>Actualizaciones de Azure Notification Hubs para iOS 13

Recientemente, Apple hizo algunas modificaciones en su servicio de inserción público y la mayoría de estos cambios se alinea con las versiones de iOS 13 y Xcode. En este artículo se describe el impacto de estos cambios en Azure Notification Hubs.

## <a name="apns-push-payload-changes"></a>Cambios en la carga de inserción de APNs

### <a name="apns-push-type"></a>Tipo de inserción de APNs

Apple ahora requiere que los desarrolladores identifiquen las notificaciones como alertas o notificaciones en segundo plano a través del nuevo encabezado `apns-push-type` de la API APNs. Según la [documentación de Apple](https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/sending_notification_requests_to_apns): "El valor de este encabezado debe reflejar con precisión el contenido de la carga de la notificación. Si no coinciden o si falta el encabezado en los sistemas requeridos, APNs puede devolver un error, retrasar la entrega de la notificación o quitarla por completo".

Los desarrolladores ahora deben establecer este encabezado en las aplicaciones que envían notificaciones a través de Azure Notification Hubs. Debido a una limitación técnica, los clientes deben usar la autenticación basada en token para las credenciales de APNs con solicitudes que incluyan este atributo. Si usa la autenticación basada en certificados para las credenciales de APNs, debe cambiar al uso de la autenticación basada en token.

Los ejemplos de código siguientes muestran cómo establecer este atributo de encabezado en las solicitudes de notificación enviadas a través de Azure Notification Hubs.

#### <a name="template-notifications---net-sdk"></a>Notificaciones de plantilla: SDK de .NET

```csharp
var hub = NotificationHubClient.CreateFromConnectionString(...);
var headers = new Dictionary<string, string> {{"apns-push-type", "alert"}};
var tempprop = new Dictionary<string, string> {{"message", "value"}};
var notification = new TemplateNotification(tempprop);
notification.Headers = headers;
await hub.SendNotificationAsync(notification);
```

#### <a name="native-notifications---net-sdk"></a>Notificaciones nativas: SDK de .NET

```csharp
var hub = NotificationHubClient.CreateFromConnectionString(...);
var headers = new Dictionary<string, string> {{"apns-push-type", "alert"}};
var notification = new AppleNotification("notification text", headers);
await hub.SendNotificationAsync(notification);
```

#### <a name="direct-rest-calls"></a>Llamadas de REST directas

```csharp
var request = new HttpRequestMessage(method, $"<resourceUri>?api-version=2017-04");
request.Headers.Add("Authorization", createToken(resourceUri, KEY_NAME, KEY_VALUE));
request.Headers.Add("ServiceBusNotification-Format", "apple");
request.Headers.Add("apns-push-type", "alert");
```

Para ayudarlo durante esta transición, cuando Azure Notification Hubs detecta una notificación que no tiene establecido el `apns-push-type`, el servicio deduce el tipo de inserción de la solicitud de notificación y establece el valor de manera automática. Recuerde que debe configurar Azure Notification Hubs para usar la autenticación basada en token para establecer el encabezado necesario; para más información, consulte [Autenticación basada en token (HTTP/2) para APNs](./notification-hubs-push-notification-http2-token-authentication.md).

## <a name="apns-priority"></a>Prioridad de APNs

Otro cambio menor, pero que requiere un cambio en la aplicación back-end que envía notificaciones, es el requisito de que, para las notificaciones en segundo plano, el encabezado `apns-priority` debe estar establecido en 5. Muchas aplicaciones establecen el encabezado `apns-priority` en 10 (que indica la entrega inmediata) o no lo establecen y obtienen el valor predeterminado (que también es 10).

Ya no se permite establecer este valor en 10 para las notificaciones en segundo plano y se debe establecer el valor para cada solicitud. Apple no enviará notificaciones en segundo plano si falta este valor. Por ejemplo:

```csharp
var hub = NotificationHubClient.CreateFromConnectionString(...);
var headers = new Dictionary<string, string> {{"apns-push-type", "background"}, { "apns-priority", "5" }};
var notification = new AppleNotification("notification text", headers);
await hub.SendNotificationAsync(notification);
```

## <a name="sdk-changes"></a>Cambios en el SDK

Durante años, los desarrolladores de iOS usaron el atributo `description` de los datos de `deviceToken` enviados al delegado de tokens de inserción para extraer el token de inserción que usa una aplicación de back-end para enviar notificaciones al dispositivo. Con Xcode 11, ese atributo `description` cambió a otro formato. El código existente que los desarrolladores usan para este atributo se ha interrumpido. Hemos actualizado el SDK de Azure Notification Hubs para dar cabida a este cambio, por lo que debe actualizar el SDK que usan las aplicaciones a la versión 2.0.4 o posterior del [SDK de iOS para Azure Notification Hubs](https://github.com/Azure/azure-notificationhubs-ios).
