---
title: Envío de notificaciones entre plataformas a usuarios con Azure Notification Hubs (ASP.NET)
description: Obtenga información acerca de cómo utilizar las plantillas de Notification Hubs para enviar, en una sola solicitud, una notificación independiente de plataforma que tenga como destino todas las plataformas.
services: notification-hubs
documentationcenter: ''
author: sethmanheim
manager: femila
editor: jwargo
ms.assetid: 11d2131b-f683-47fd-a691-4cdfc696f62b
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-windows
ms.devlang: multiple
ms.topic: article
ms.date: 09/30/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 10/02/2019
ms.openlocfilehash: 467664f4c102a81f6469765ea1742995f04c60e6
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87022011"
---
# <a name="send-cross-platform-notifications-with-azure-notification-hubs"></a>Envío de notificaciones entre plataformas con Azure Notification Hubs

Este tutorial se basa en el anterior, [Envío de notificaciones a usuarios concretos mediante Azure Notification Hubs]. En ese tutorial se describe cómo insertar notificaciones en todos los dispositivos que están registrados en un usuario autenticado concreto. Ese enfoque requería que varias solicitudes enviaran una notificación a cada plataforma de cliente compatible. Azure Notification Hubs admite plantillas, que le permiten especificar cómo un dispositivo específico desea recibir notificaciones. Este método simplifica el envío de notificaciones entre plataformas.

En este artículo se muestra cómo aprovechar las plantillas para enviar una notificación que se dirige a todas las plataformas. En este artículo se usa una única solicitud para enviar una notificación que no tiene en cuenta la plataforma. Para más información sobre las plantillas, consulte [Introducción a Notification Hubs][Templates].

> [!IMPORTANT]
> Windows Phone 8.1 y las versiones anteriores no se admiten en Visual Studio 2019. Para más información, consulte [Compatibilidad y destinatarios de la plataforma de Visual Studio 2019](/visualstudio/releases/2019/compatibility).

> [!NOTE]
> Con Notification Hubs, un dispositivo puede registrar varias plantillas con la misma etiqueta. En este caso, un mensaje entrante dirigido a esa etiqueta da lugar a que se entreguen varias notificaciones al dispositivo, una por cada plantilla. Este proceso le permite mostrar el mismo mensaje en varias notificaciones visuales, como distintivo y como notificación del sistema en una aplicación de la Tienda Windows.

## <a name="send-cross-platform-notifications-using-templates"></a>Envío de notificaciones entre plataformas mediante plantillas

En esta sección se usa el código de ejemplo que se creó en el tutorial [Envío de notificaciones a usuarios concretos mediante Azure Notification Hubs]. Puede descargar el ejemplo de [GitHub](https://github.com/Azure/azure-notificationhubs-dotnet/tree/master/Samples/NotifyUsers).

Para enviar notificaciones multiplataforma mediante plantillas, siga estos pasos:

1. En Visual Studio, en el **Explorador de soluciones**, expanda la carpeta **Controladores** y, después, abra el archivo *RegisterController.cs*.

1. Encuentre el bloque de código en el método `Put` que crea un nuevo registro y reemplace el contenido de `switch` por el código siguiente:

    ```csharp
    switch (deviceUpdate.Platform)
    {
        case "mpns":
            var toastTemplate = "<?xml version=\"1.0\" encoding=\"utf-8\"?>" +
                "<wp:Notification xmlns:wp=\"WPNotification\">" +
                    "<wp:Toast>" +
                        "<wp:Text1>$(message)</wp:Text1>" +
                    "</wp:Toast> " +
                "</wp:Notification>";
            registration = new MpnsTemplateRegistrationDescription(deviceUpdate.Handle, toastTemplate);
            break;
        case "wns":
            toastTemplate = @"<toast><visual><binding template=""ToastText01""><text id=""1"">$(message)</text></binding></visual></toast>";
            registration = new WindowsTemplateRegistrationDescription(deviceUpdate.Handle, toastTemplate);
            break;
        case "apns":
            var alertTemplate = "{\"aps\":{\"alert\":\"$(message)\"}}";
            registration = new AppleTemplateRegistrationDescription(deviceUpdate.Handle, alertTemplate);
            break;
        case "fcm":
            var messageTemplate = "{\"data\":{\"message\":\"$(message)\"}}";
            registration = new FcmTemplateRegistrationDescription(deviceUpdate.Handle, messageTemplate);
            break;
        default:
            throw new HttpResponseException(HttpStatusCode.BadRequest);
    }
    ```

    Este código llama al método específico de la plataforma para crear un registro de plantilla en lugar de un registro nativo. Como los registros de plantilla se obtienen de registros nativos, no es necesario modificar los registros existentes.

1. En el **Explorador de soluciones**, en la carpeta **Controllers**, abra el archivo *NotificationsController.cs*. Reemplace el método `Post` por el código siguiente:

    ```csharp
    public async Task<HttpResponseMessage> Post()
    {
        var user = HttpContext.Current.User.Identity.Name;
        var userTag = "username:" + user;

        var notification = new Dictionary<string, string> { { "message", "Hello, " + user } };
        await Notifications.Instance.Hub.SendTemplateNotificationAsync(notification, userTag);

        return Request.CreateResponse(HttpStatusCode.OK);
    }
    ```

    Este código envía una notificación a todas las plataformas al mismo tiempo. No se especifica una carga nativa. Notification Hubs crea y entrega la carga correcta a cada dispositivo con el valor tag proporcionado, como se especifica en las plantillas registradas.

1. Vuelva a publicar el proyecto de API Web.

1. Vuelva a ejecutar la aplicación cliente para comprobar que el registro se ha realizado correctamente.

1. Opcionalmente, implemente la aplicación cliente en un segundo dispositivo y ejecute la aplicación. Aparece una notificación en cada dispositivo.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha completado este tutorial, puede obtener más información acerca de Notification Hubs y las plantillas en estos artículos:

* Para conocer otro escenario para utilizar plantillas, consulte el tutorial [Notificaciones de inserción a dispositivos Windows concretos que ejecutan aplicaciones de Plataforma universal de Windows][Use Notification Hubs to send breaking news].
* Para más información sobre las plantillas, consulte [Introducción a Notification Hubs][Templates].

<!-- Anchors. -->

<!-- Images. -->

<!-- URLs. -->
[Push to users ASP.NET]: notification-hubs-aspnet-backend-ios-apple-apns-notification.md
[Push to users Mobile Services]: notification-hubs-aspnet-backend-windows-dotnet-wns-notification.md
[Visual Studio 2012 Express for Windows 8]: https://visualstudio.microsoft.com/downloads/

[Use Notification Hubs to send breaking news]: notification-hubs-windows-notification-dotnet-push-xplat-segmented-wns.md
[Azure Notification Hubs]: https://go.microsoft.com/fwlink/p/?LinkId=314257
[Envío de notificaciones a usuarios concretos mediante Azure Notification Hubs]: notification-hubs-aspnet-backend-windows-dotnet-wns-notification.md
[Templates]: https://go.microsoft.com/fwlink/p/?LinkId=317339
[Notification Hub How to for Windows Store]: /previous-versions/azure/azure-services/jj927170(v=azure.100)
