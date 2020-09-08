---
title: Envío de notificaciones programas | Microsoft Docs
description: En este tema se describe cómo usar Notificaciones programadas con Azure Notification Hubs.
services: notification-hubs
documentationcenter: .net
keywords: notificaciones push, notificación push, programación de notificaciones push
author: sethmanheim
manager: femila
editor: jwargo
ms.assetid: 6b718c75-75dd-4c99-aee3-db1288235c1a
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-android
ms.devlang: dotnet
ms.topic: article
ms.date: 01/04/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 01/04/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 56eedda7f79fedce1e34ad837c92006e5cd8f191
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88998278"
---
# <a name="how-to-send-scheduled-notifications"></a>Envío de notificaciones programadas

Si tiene un escenario en el que desea enviar una notificación en algún momento del futuro, pero no tiene una forma sencilla de activar el código de back-end para enviar la notificación. Notification Hubs de nivel estándar admite una característica que le permite programar notificaciones para hasta siete días en el futuro.


## <a name="schedule-your-notifications"></a>Programación de las notificaciones
Cuando envíe una notificación, simplemente use la clase [`ScheduledNotification`](/dotnet/api/microsoft.azure.notificationhubs.schedulednotification?view=azure-dotnet#microsoft_azure_notificationhubs_schedulednotification) en los SDK de Notification Hubs, tal como se muestra en el ejemplo siguiente:

```csharp
Notification notification = new AppleNotification("{\"aps\":{\"alert\":\"Happy birthday!\"}}");
var scheduled = await hub.ScheduleNotificationAsync(notification, new DateTime(2014, 7, 19, 0, 0, 0));
```

## <a name="cancel-scheduled-notifications"></a>Cancelación de notificaciones programadas
Además, puede cancelar una notificación programada anteriormente con su notificationId:

```csharp
await hub.CancelNotificationAsync(scheduled.ScheduledNotificationId);
```

No hay límite de notificaciones programadas que puede enviar.

## <a name="next-steps"></a>Pasos siguientes

Vea los siguientes tutoriales:

 - [Envío de notificaciones push a todos los dispositivos registrados](notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md)
 - [Envío de notificaciones push a dispositivos específicos](notification-hubs-windows-notification-dotnet-push-xplat-segmented-wns.md)
 - [Envío de notificaciones push localizadas](notification-hubs-windows-store-dotnet-xplat-localized-wns-push-notification.md)
 - [Envío de notificaciones push a usuarios específicos](notification-hubs-aspnet-backend-windows-dotnet-wns-notification.md) 
 - [Envío de notificaciones push basadas en la ubicación](notification-hubs-push-bing-spatial-data-geofencing-notification.md)
