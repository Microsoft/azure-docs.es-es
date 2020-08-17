---
title: Envío de notificaciones push localizadas a iOS mediante Azure Notification Hubs | Microsoft Docs
description: Obtenga información sobre cómo enviar notificaciones push localizadas a dispositivos iOS mediante Azure Notification Hubs.
services: notification-hubs
documentationcenter: ios
author: sethmanheim
manager: femila
editor: jwargo
ms.assetid: 484914b5-e081-4a05-a84a-798bbd89d428
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: ios
ms.devlang: objective-c
ms.topic: article
ms.date: 01/04/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 01/04/2019
ms.openlocfilehash: 4bc392cb6d499d967656358e920f46e261af2906
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87030341"
---
# <a name="tutorial-send-localized-push-notifications-to-ios-using-azure-notification-hubs"></a>Tutorial: Envío de notificaciones push localizadas a iOS mediante Azure Notification Hubs

> [!div class="op_single_selector"]
> * [C# para Tienda Windows](notification-hubs-windows-store-dotnet-xplat-localized-wns-push-notification.md)
> * [iOS](notification-hubs-ios-xplat-localized-apns-push-notification.md)

En este tutorial se muestra cómo usar la característica de [plantillas](notification-hubs-templates-cross-platform-push-messages.md) de Azure Notification Hubs para difundir notificaciones de noticias de última hora localizadas por lenguaje y dispositivo. En este tutorial comenzará con la aplicación de iOS que se creó en [Uso de Notification Hubs para enviar noticias de última hora]. Una vez que lo haya completado, podrá registrarse en las categorías que le interesen, especificar un idioma para recibir las notificaciones y recibir solo notificaciones push para las categorías seleccionadas en dicho idioma.

Este escenario tiene dos partes:

* La aplicación iOS permite que los dispositivos cliente especifiquen un idioma y que se suscriban a distintas categorías de noticias de última hora;
* El back-end difunde las notificaciones mediante las características **etiqueta** y **plantilla** de Azure Notification Hubs.

En este tutorial, realizará los siguientes pasos:

> [!div class="checklist"]
> * Actualizar la interfaz de usuario de la aplicación
> * Crear la aplicación de iOS
> * Enviar notificaciones de plantillas localizadas desde la aplicación de consola .NET
> * Enviar notificaciones de plantillas localizadas desde el dispositivo

## <a name="overview"></a>Información general

En el tutorial [Uso de Notification Hubs para enviar noticias de última hora] creó una aplicación que utilizó **etiquetas** para suscribirse a notificaciones para distintas categorías de noticias. Sin embargo, muchas aplicaciones están dirigidas a varios mercados y requieren localización. Esto significa que el contenido de las notificaciones mismas se debe localizar y entregar al conjunto de dispositivos correcto. En este tutorial se explica cómo usar la característica **plantilla** de Notification Hubs para entregar fácilmente notificaciones de noticias de última hora localizadas.

> [!NOTE]
> Una forma de enviar notificaciones localizadas es crear varias versiones de cada etiqueta. Por ejemplo, para admitir inglés, francés y chino mandarín, necesitaría tres etiquetas distintas para noticias mundiales: "world_en", "world_fr" y "world_ch". Luego tendría que enviar una versión localizada de las noticias mundiales a cada una de estas etiquetas. En este tema se usan plantillas para evitar la proliferación de etiquetas y el requisito de enviar varios mensajes.

Las plantillas son una forma de especificar la manera en que un dispositivo específico debe recibir una notificación. La plantilla especifica el formato de carga exacto haciendo referencia a las propiedades que forman parte del mensaje enviado por el back-end de la aplicación. En su caso, se envía un mensaje independiente de la configuración regional que contiene todos los idiomas compatibles:

```json
{
    "News_English": "...",
    "News_French": "...",
    "News_Mandarin": "..."
}
```

Esto garantizará que los dispositivos se registren con una plantilla que hace referencia a la propiedad correcta. Por ejemplo, una aplicación iOS que quiera registrarse para noticias en francés se registrará con la siguiente sintaxis:

```json
{
    aps: {
        alert: "$(News_French)"
    }
}
```

Para más información sobre el uso de plantillas, vea el artículo [Plantillas](notification-hubs-templates-cross-platform-push-messages.md).

## <a name="prerequisites"></a>Prerrequisitos

* Complete el tutorial [Envío de notificaciones push a dispositivos iOS concretos](notification-hubs-ios-xplat-segmented-apns-push-notification.md) y disponga del código, porque este tutorial se basa directamente en ese código.
* Visual Studio 2019 es opcional.

## <a name="update-the-app-user-interface"></a>Actualizar la interfaz de usuario de la aplicación

En esta sección, se modificará la aplicación de noticias de última hora que creó en el tema [Uso de Notification Hubs para enviar noticias de última hora] para enviar noticias de última hora localizadas con plantillas.

En `MainStoryboard_iPhone.storyboard`, agregue un control segmentado con los tres idiomas: inglés, francés y chino mandarín.

![Creación del guion gráfico de interfaz de usuario de iOS][13]

A continuación, asegúrese de agregar un IBOutlet en ViewController.h tal como se muestra en la imagen siguiente:

![Creación de salidas para los modificadores][14]

## <a name="build-the-ios-app"></a>Crear la aplicación de iOS

1. En `Notification.h`, agregue el método `retrieveLocale`, modifique el almacén y suscríbase a métodos tal como se muestra en el siguiente código:

    ```objc
    - (void) storeCategoriesAndSubscribeWithLocale:(int) locale categories:(NSSet*) categories completion: (void (^)(NSError* error))completion;

    - (void) subscribeWithLocale:(int) locale categories:(NSSet*) categories completion:(void (^)(NSError *))completion;

    - (NSSet*) retrieveCategories;

    - (int) retrieveLocale;
    ```
    En `Notification.m`, modifique el método `storeCategoriesAndSubscribe` agregando el parámetro `locale` y almacenándolo en los valores predeterminados del usuario:

    ```objc
    - (void) storeCategoriesAndSubscribeWithLocale:(int) locale categories:(NSSet *)categories completion:(void (^)(NSError *))completion {
        NSUserDefaults* defaults = [NSUserDefaults standardUserDefaults];
        [defaults setValue:[categories allObjects] forKey:@"BreakingNewsCategories"];
        [defaults setInteger:locale forKey:@"BreakingNewsLocale"];
        [defaults synchronize];

        [self subscribeWithLocale: locale categories:categories completion:completion];
    }
    ```

    Luego modifique el método *subscribe* para incluir la configuración regional:

    ```objc
    - (void) subscribeWithLocale: (int) locale categories:(NSSet *)categories completion:(void (^)(NSError *))completion{
        SBNotificationHub* hub = [[SBNotificationHub alloc] initWithConnectionString:@"<connection string>" notificationHubPath:@"<hub name>"];

        NSString* localeString;
        switch (locale) {
            case 0:
                localeString = @"English";
                break;
            case 1:
                localeString = @"French";
                break;
            case 2:
                localeString = @"Mandarin";
                break;
        }

        NSString* template = [NSString stringWithFormat:@"{\"aps\":{\"alert\":\"$(News_%@)\"},\"inAppMessage\":\"$(News_%@)\"}", localeString, localeString];

        [hub registerTemplateWithDeviceToken:self.deviceToken name:@"localizednewsTemplate" jsonBodyTemplate:template expiryTemplate:@"0" tags:categories completion:completion];
    }
    ```

    Utilice el método `registerTemplateWithDeviceToken`, en lugar de `registerNativeWithDeviceToken`. Al registrarse para obtener una plantilla, tiene que proporcionar la plantilla JSON y también un nombre para la plantilla (dado que puede que la aplicación quiera registrar diferentes plantillas). Asegúrese de registrar sus categorías como etiquetas, ya que queremos asegurarnos de recibir las notificaciones para esas noticias.

    Agregue un método para recuperar la configuración regional a partir de los ajustes predeterminados del usuario:

    ```objc
    - (int) retrieveLocale {
        NSUserDefaults* defaults = [NSUserDefaults standardUserDefaults];

        int locale = [defaults integerForKey:@"BreakingNewsLocale"];

        return locale < 0?0:locale;
    }
    ```

2. Ahora que ha modificado la clase `Notifications`, debe asegurarse de que la clase `ViewController` utiliza la nueva `UISegmentControl`. Agregue la siguiente línea en el método `viewDidLoad` para asegurarse de mostrar la configuración regional actualmente seleccionada:

    ```objc
    self.Locale.selectedSegmentIndex = [notifications retrieveLocale];
    ```

    Después, en el método `subscribe`, cambie la llamada a `storeCategoriesAndSubscribe` al código siguiente:

    ```objc
    [notifications storeCategoriesAndSubscribeWithLocale: self.Locale.selectedSegmentIndex categories:[NSSet setWithArray:categories] completion: ^(NSError* error) {
        if (!error) {
            UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Notification" message:
                                    @"Subscribed!" delegate:nil cancelButtonTitle:
                                    @"OK" otherButtonTitles:nil, nil];
            [alert show];
        } else {
            NSLog(@"Error subscribing: %@", error);
        }
    }];
    ```

3. Finalmente, debe actualizar el método `didRegisterForRemoteNotificationsWithDeviceToken` en AppDelegate.m, para que pueda actualizar correctamente el registro cuando se inicie la aplicación. Cambie la llamada al método `subscribe` de notificaciones con el código siguiente:

    ```obj-c
    NSSet* categories = [self.notifications retrieveCategories];
    int locale = [self.notifications retrieveLocale];
    [self.notifications subscribeWithLocale: locale categories:categories completion:^(NSError* error) {
        if (error != nil) {
            NSLog(@"Error registering for notifications: %@", error);
        }
    }];
    ```

## <a name="optional-send-localized-template-notifications-from-net-console-app"></a>(Opcional) Envíe notificaciones de plantillas localizadas desde la aplicación de consola .NET.

[!INCLUDE [notification-hubs-localized-back-end](../../includes/notification-hubs-localized-back-end.md)]

## <a name="optional-send-localized-template-notifications-from-the-device"></a>(Opcional) Enviar notificaciones de plantillas localizadas desde el dispositivo

Si no tiene acceso a Visual Studio, o simplemente quiere probar el envío de las notificaciones de plantilla localizadas directamente desde la aplicación del dispositivo. Puede agregar los parámetros de plantilla localizada al método `SendNotificationRESTAPI` que ha definido en el tutorial anterior.

```objc
- (void)SendNotificationRESTAPI:(NSString*)categoryTag
{
    NSURLSession* session = [NSURLSession sessionWithConfiguration:[NSURLSessionConfiguration
                                defaultSessionConfiguration] delegate:nil delegateQueue:nil];

    NSString *json;

    // Construct the messages REST endpoint
    NSURL* url = [NSURL URLWithString:[NSString stringWithFormat:@"%@%@/messages/%@", HubEndpoint,
                                        HUBNAME, API_VERSION]];

    // Generated the token to be used in the authorization header.
    NSString* authorizationToken = [self generateSasToken:[url absoluteString]];

    //Create the request to add the template notification message to the hub
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
    [request setHTTPMethod:@"POST"];

    // Add the category as a tag
    [request setValue:categoryTag forHTTPHeaderField:@"ServiceBusNotification-Tags"];

    // Template notification
    json = [NSString stringWithFormat:@"{\"messageParam\":\"Breaking %@ News : %@\","
            \"News_English\":\"Breaking %@ News in English : %@\","
            \"News_French\":\"Breaking %@ News in French : %@\","
            \"News_Mandarin\":\"Breaking %@ News in Mandarin : %@\","
            categoryTag, self.notificationMessage.text,
            categoryTag, self.notificationMessage.text,  // insert English localized news here
            categoryTag, self.notificationMessage.text,  // insert French localized news here
            categoryTag, self.notificationMessage.text]; // insert Mandarin localized news here

    // Signify template notification format
    [request setValue:@"template" forHTTPHeaderField:@"ServiceBusNotification-Format"];

    // JSON Content-Type
    [request setValue:@"application/json;charset=utf-8" forHTTPHeaderField:@"Content-Type"];

    //Authenticate the notification message POST request with the SaS token
    [request setValue:authorizationToken forHTTPHeaderField:@"Authorization"];

    //Add the notification message body
    [request setHTTPBody:[json dataUsingEncoding:NSUTF8StringEncoding]];

    // Send the REST request
    NSURLSessionDataTask* dataTask = [session dataTaskWithRequest:request
                completionHandler:^(NSData *data, NSURLResponse *response, NSError *error)
        {
        NSHTTPURLResponse* httpResponse = (NSHTTPURLResponse*) response;
            if (error || httpResponse.statusCode != 200)
            {
                NSLog(@"\nError status: %d\nError: %@", httpResponse.statusCode, error);
            }
            if (data != NULL)
            {
                //xmlParser = [[NSXMLParser alloc] initWithData:data];
                //[xmlParser setDelegate:self];
                //[xmlParser parse];
            }
        }];

    [dataTask resume];
}
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, se envían notificaciones localizadas a dispositivos iOS. Para aprender a enviar notificaciones push a usuarios específicos de aplicaciones iOS, pase al siguiente tutorial:

> [!div class="nextstepaction"]
>[Envío de notificaciones push a usuarios específicos](notification-hubs-aspnet-backend-ios-apple-apns-notification.md)

<!-- Images. -->
[13]: ./media/notification-hubs-ios-send-localized-breaking-news/ios_localized1.png
[14]: ./media/notification-hubs-ios-send-localized-breaking-news/ios_localized2.png

<!-- URLs. -->
[How To: Service Bus Notification Hubs (iOS Apps)]: /previous-versions/azure/reference/dn223264(v=azure.100)
[Uso de Notification Hubs para enviar noticias de última hora]: notification-hubs-ios-xplat-segmented-apns-push-notification.md
[Mobile Service]: /develop/mobile/tutorials/get-started
[Notify users with Notification Hubs: ASP.NET]: notification-hubs-aspnet-backend-ios-apple-apns-notification.md
[Notify users with Notification Hubs: Mobile Services]: notification-hubs-aspnet-backend-windows-dotnet-wns-notification.md
[Submit an app page]: https://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: https://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: https://go.microsoft.com/fwlink/p/?LinkId=262253
[Get started with Mobile Services]: /develop/mobile/tutorials/get-started/#create-new-service
[Get started with data]: /develop/mobile/tutorials/get-started-with-data-ios
[Get started with authentication]: /develop/mobile/tutorials/get-started-with-users-ios
[Get started with push notifications]: /develop/mobile/tutorials/get-started-with-push-ios
[Push notifications to app users]: /develop/mobile/tutorials/push-notifications-to-users-ios
[Authorize users with scripts]: /develop/mobile/tutorials/authorize-users-in-scripts-ios
[JavaScript and HTML]: ../get-started-with-push-js.md
[Windows Developer Preview registration steps for Mobile Services]: ../mobile-services-windows-developer-preview-registration.md
[wns object]: https://go.microsoft.com/fwlink/p/?LinkId=260591
[Notification Hubs Guidance]: /previous-versions/azure/azure-services/jj927170(v=azure.100)
[Notification Hubs How-To for iOS]: /previous-versions/azure/reference/dn223264(v=azure.100)
