---
title: Envío de notificaciones localizadas a aplicaciones Windows mediante Azure Notification Hubs | Microsoft Docs
description: Obtenga información sobre cómo usar Azure Notification Hubs para enviar notificaciones de noticias de última hora localizadas.
services: notification-hubs
documentationcenter: windows
author: sethmanheim
manager: femila
editor: jwargo
ms.assetid: c454f5a3-a06b-45ac-91c7-f91210889b25
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-windows
ms.devlang: dotnet
ms.topic: tutorial
ms.custom: mvc, devx-track-csharp
ms.date: 03/22/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 03/22/2019
ms.openlocfilehash: 2fc29eeea7969812f0fbef17892893f7a79a048a
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88998193"
---
# <a name="tutorial-send-localized-push-notifications-to-windows-apps-using-azure-notification-hubs"></a>Tutorial: Envío de notificaciones push localizadas a aplicaciones Windows mediante Azure Notification Hubs

> [!div class="op_single_selector"]
> * [C# para Tienda Windows](notification-hubs-windows-store-dotnet-xplat-localized-wns-push-notification.md)
> * [iOS](notification-hubs-ios-xplat-localized-apns-push-notification.md)

## <a name="overview"></a>Información general

En este tutorial se muestra cómo insertar notificaciones localizadas a dispositivos móviles registrados en el servicio Notification Hubs. En el tutorial, va a actualizar aplicaciones creadas en el [Tutorial: Envío de notificaciones a dispositivos concretos (Plataforma universal de Windows)](notification-hubs-windows-phone-push-xplat-segmented-mpns-notification.md) para admitir los siguientes escenarios:

- La aplicación de la Tienda Windows permite que los dispositivos cliente especifiquen un idioma y se suscriban a distintas categorías de noticias de última hora.
- La aplicación de back-end difunde las notificaciones mediante las características **etiqueta** y **plantilla** de Azure Notification Hubs.

Al completar el tutorial, la aplicación móvil le permite registrar las categorías que le interesen, así como especificar el idioma en el que va a recibir las notificaciones. La aplicación de back-end envía notificaciones que están localizadas por idioma y dispositivo.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Actualización de una aplicación de Windows para que admita la información de configuración regional
> * Actualizar una aplicación de back-end para enviar notificaciones localizadas
> * Prueba de la aplicación

## <a name="prerequisites"></a>Prerrequisitos

Complete el [Tutorial: Envío de notificaciones a dispositivos concretos (Plataforma universal de Windows)](notification-hubs-windows-phone-push-xplat-segmented-mpns-notification.md).

En el [Tutorial: Envío de notificaciones a dispositivos concretos (Plataforma universal de Windows)](notification-hubs-windows-phone-push-xplat-segmented-mpns-notification.md), ha creado una aplicación que utilizaba **etiquetas** para suscribirse a notificaciones para diferentes **categorías** de noticias. En este tema se usa la característica de **plantilla** de Notification Hubs para entregar fácilmente notificaciones de noticias de última hora **localizadas**.

A un alto nivel, las plantillas son una forma de especificar el formato en que un dispositivo específico debe recibir una notificación. La plantilla especifica el formato de carga exacto haciendo referencia a las propiedades que forman parte del mensaje enviado por el back-end de la aplicación. En este tutorial, la aplicación de back-end envía un mensaje independiente de la configuración regional que contiene todos los idiomas admitidos:

```json
{
    "News_English": "...",
    "News_French": "...",
    "News_Mandarin": "..."
}
```

Esto garantizará que los dispositivos se registren con una plantilla que hace referencia a la propiedad correcta. Por ejemplo, una aplicación de la Tienda Windows que quiere recibir un simple mensaje del sistema en inglés se registra en la plantilla siguiente con las etiquetas correspondientes:

```xml
<toast>
    <visual>
    <binding template=\"ToastText01\">
        <text id=\"1\">$(News_English)</text>
    </binding>
    </visual>
</toast>
```

Para más información acerca de las plantillas, consulte el artículo sobre [Plantillas push](notification-hubs-templates-cross-platform-push-messages.md).

## <a name="update-windows-app-to-support-locale-information"></a>Actualización de una aplicación de Windows para que admita la información de configuración regional

1. Abra la solución de Visual Studio que creó en el [Tutorial: Envío de notificaciones a dispositivos concretos (Plataforma universal de Windows)](notification-hubs-windows-phone-push-xplat-segmented-mpns-notification.md).
2. Actualice el archivo `MainPage.xaml` para incluir un cuadro combinado de configuración regional:

    ```xml
    <Grid Margin="120, 58, 120, 80"  
            Background="{StaticResource ApplicationPageBackgroundThemeBrush}">
        <Grid.RowDefinitions>
            <RowDefinition />
            <RowDefinition />
            <RowDefinition />
            <RowDefinition />
            <RowDefinition />
            <RowDefinition />
        </Grid.RowDefinitions>
        <Grid.ColumnDefinitions>
            <ColumnDefinition />
            <ColumnDefinition />
        </Grid.ColumnDefinitions>
        <TextBlock Grid.Row="0" Grid.Column="0" Grid.ColumnSpan="2"  TextWrapping="Wrap" Text="Breaking News" FontSize="42" VerticalAlignment="Top"/>
        <ComboBox Name="Locale" HorizontalAlignment="Left" VerticalAlignment="Center" Width="200" Grid.Row="1" Grid.Column="0">
            <x:String>English</x:String>
            <x:String>French</x:String>
            <x:String>Mandarin</x:String>
        </ComboBox>
        <ToggleSwitch Header="World" Name="WorldToggle" Grid.Row="2" Grid.Column="0"/>
        <ToggleSwitch Header="Politics" Name="PoliticsToggle" Grid.Row="3" Grid.Column="0"/>
        <ToggleSwitch Header="Business" Name="BusinessToggle" Grid.Row="4" Grid.Column="0"/>
        <ToggleSwitch Header="Technology" Name="TechnologyToggle" Grid.Row="2" Grid.Column="1"/>
        <ToggleSwitch Header="Science" Name="ScienceToggle" Grid.Row="3" Grid.Column="1"/>
        <ToggleSwitch Header="Sports" Name="SportsToggle" Grid.Row="4" Grid.Column="1"/>
        <Button Content="Subscribe" HorizontalAlignment="Center" Grid.Row="5" Grid.Column="0" Grid.ColumnSpan="2" Click="SubscribeButton_Click" />
    </Grid>
    ```
3. En la clase `Notifications`, agregue un parámetro de configuración regional para los métodos `StoreCategoriesAndSubscribe` y `SubscribeToCategories`.

    ```csharp
    public async Task<Registration> StoreCategoriesAndSubscribe(string locale, IEnumerable<string> categories)
    {
        ApplicationData.Current.LocalSettings.Values["categories"] = string.Join(",", categories);
        ApplicationData.Current.LocalSettings.Values["locale"] = locale;
        return await SubscribeToCategories(locale, categories);
    }

    public async Task<Registration> SubscribeToCategories(string locale, IEnumerable<string> categories = null)
    {
        var channel = await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();

        if (categories == null)
        {
            categories = RetrieveCategories();
        }

        // Using a template registration. This makes supporting notifications across other platforms much easier.
        // Using the localized tags based on locale selected.
        string templateBodyWNS = String.Format("<toast><visual><binding template=\"ToastText01\"><text id=\"1\">$(News_{0})</text></binding></visual></toast>", locale);

        return await hub.RegisterTemplateAsync(channel.Uri, templateBodyWNS, "localizedWNSTemplateExample", categories);
    }
    ```

    En lugar de llamar al método `RegisterNativeAsync`, llame a `RegisterTemplateAsync`. Registre un formato de notificación específico en el que la plantilla dependa de la configuración regional. También puede especificar un nombre para la plantilla ("localizedWNSTemplateExample"), porque desea registrar más de una plantilla (por ejemplo, una para las notificaciones del sistema y otra para los iconos). También debe asignarle un nombre para actualizarla o eliminarla.

    Si un dispositivo registra varias plantillas con la misma etiqueta, el envío de un mensaje destinado a dicha etiqueta dará como resultado la entrega de varias notificaciones al dispositivo (una para cada plantilla). Este comportamiento resulta útil cuando un mismo mensaje lógico debe dar como resultado varias notificaciones visuales que muestren, por ejemplo, tanto un distintivo como una notificación del sistema en una aplicación de la Tienda Windows.
4. Agregue el método siguiente para recuperar la configuración regional almacenada:

    ```csharp
    public string RetrieveLocale()
    {
        var locale = (string) ApplicationData.Current.LocalSettings.Values["locale"];
        return locale != null ? locale : "English";
    }
    ```

5. En el archivo `MainPage.xaml.cs`, actualice el controlador de clic del botón para recuperar el valor actual del cuadro combinado de configuración regional y proporcionarlo a la llamada a la clase `Notifications`:

    ```csharp
    private async void SubscribeButton_Click(object sender, RoutedEventArgs e)
    {
        var locale = (string)Locale.SelectedItem;

        var categories = new HashSet<string>();
        if (WorldToggle.IsOn) categories.Add("World");
        if (PoliticsToggle.IsOn) categories.Add("Politics");
        if (BusinessToggle.IsOn) categories.Add("Business");
        if (TechnologyToggle.IsOn) categories.Add("Technology");
        if (ScienceToggle.IsOn) categories.Add("Science");
        if (SportsToggle.IsOn) categories.Add("Sports");

        var result = await ((App)Application.Current).notifications.StoreCategoriesAndSubscribe(locale,
                categories);

        var dialog = new MessageDialog("Locale: " + locale + " Subscribed to: " + 
            string.Join(",", categories) + " on registration Id: " + result.RegistrationId);
        dialog.Commands.Add(new UICommand("OK"));
        await dialog.ShowAsync();
    }
    ```
6. Por último, en el archivo `App.xaml.cs`, actualice el método `InitNotificationsAsync` para recuperar la configuración regional y usarla al suscribirse:

    ```csharp
    private async void InitNotificationsAsync()
    {
        var result = await notifications.SubscribeToCategories(notifications.RetrieveLocale());

        // Displays the registration ID so you know it was successful
        if (result.RegistrationId != null)
        {
            var dialog = new MessageDialog("Registration successful: " + result.RegistrationId);
            dialog.Commands.Add(new UICommand("OK"));
            await dialog.ShowAsync();
        }
    }
    ```



## <a name="run-the-uwp-application"></a>Ejecución de la aplicación de UWP

1. Ejecute la aplicación Universal Windows Platform. Espere hasta que vea el mensaje **Registration successful** (Registro correcto).

    ![Registro y aplicación para dispositivos móviles](./media/notification-hubs-windows-store-dotnet-xplat-localized-wns-push-notification/registration-successful.png)
2. Seleccione las **categorías** y la **configuración regional**, y haga clic en **Subscribe** (Suscribir). La aplicación convierte las categorías seleccionadas en etiquetas y solicita un nuevo registro de dispositivo para las etiquetas seleccionadas al Centro de notificaciones.

    ![Aplicación para dispositivos móviles](./media/notification-hubs-windows-store-dotnet-xplat-localized-wns-push-notification/mobile-app.png)
3. Se ve un mensaje de **confirmación** acerca de las **suscripciones**.

    ![Mensaje de suscripción](./media/notification-hubs-windows-store-dotnet-xplat-localized-wns-push-notification/subscription-message.png)

## <a name="update-console-app-to-send-localized-notifications"></a>Actualización de una aplicación de consola para enviar notificaciones localizadas

Cuando se envían notificaciones de plantilla, solo es necesario especificar un conjunto de propiedades; en este tutorial, la aplicación de back-end envía el conjunto de propiedades que contiene la versión localizada de las noticias de actualidad, como por ejemplo:

```json
{
    "News_English": "World News in English!",
    "News_French": "World News in French!",
    "News_Mandarin": "World News in Mandarin!"
}
```

En esta sección se actualiza el proyecto de aplicación de consola en la solución. Modifique el método `SendTemplateNotificationAsync` en la aplicación de consola que creó anteriormente con el código siguiente:

> [!IMPORTANT]
> Especifique en el código el nombre y la cadena de conexión con acceso completo para el Centro de notificaciones.

```csharp
private static async void SendTemplateNotificationAsync()
{
    // Define the notification hub.
    NotificationHubClient hub = NotificationHubClient.CreateClientFromConnectionString(
        "<connection string with full access>", "<hub name>");

    // Sending the notification as a template notification. All template registrations that contain
    // "messageParam" or "News_<local selected>" and the proper tags will receive the notifications.
    // This includes APNS, FCM, WNS, and MPNS template registrations.
    Dictionary<string, string> templateParams = new Dictionary<string, string>();

    // Create an array of breaking news categories.
    var categories = new string[] { "World", "Politics", "Business", "Technology", "Science", "Sports"};
    var locales = new string[] { "English", "French", "Mandarin" };

    foreach (var category in categories)
    {
        templateParams["messageParam"] = "Breaking " + category + " News!";

        // Sending localized News for each tag too...
        foreach( var locale in locales)
        {
            string key = "News_" + locale;

            // Your real localized news content would go here.
            templateParams[key] = "Breaking " + category + " News in " + locale + "!";
        }

        await hub.SendTemplateNotificationAsync(templateParams, category);
    }
}
```

Esta simple llamada entregará la noticia localizada a **todos** los dispositivos, con independencia de la plataforma, ya que el Centro de notificaciones crea y entrega la carga nativa correcta a todos los dispositivos suscritos a una etiqueta específica.

## <a name="run-console-app-to-send-localized-notification"></a>Ejecución de una aplicación de consola para enviar notificaciones localizadas
Ejecute la **aplicación de consola** para enviar notificaciones de todas las categorías y en todos los idiomas admitidos. Compruebe que solo recibe una notificación de las categorías a las que se ha suscrito y que el mensaje es para la configuración regional que ha seleccionado.

![Mensajes de notificación](./media/notification-hubs-windows-store-dotnet-xplat-localized-wns-push-notification/notifications.png)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a enviar notificaciones push localizadas a dispositivos concretos que tienen etiquetas asociadas a sus registros. Para aprender a enviar notificaciones a usuarios específicos que puede que usen más de un dispositivo, pase al siguiente tutorial:

> [!div class="nextstepaction"]
>[Envío de notificaciones push a usuarios específicos](notification-hubs-aspnet-backend-windows-dotnet-wns-notification.md)

<!-- Anchors. -->
[Template concepts]: #concepts
[The app user interface]: #ui
[Building the Windows Store client app]: #building-client
[Send notifications from your back-end]: #send
[Next Steps]:#next-steps

<!-- Images. -->

<!-- URLs. -->
[Mobile Service]: /develop/mobile/tutorials/get-started
[Notify users with Notification Hubs: ASP.NET]: notification-hubs-aspnet-backend-ios-apple-apns-notification.md
[Notify users with Notification Hubs: Mobile Services]: notification-hubs-aspnet-backend-windows-dotnet-wns-notification.md
[Use Notification Hubs to send breaking news]: notification-hubs/notification-hubs-windows-notification-dotnet-push-xplat-segmented-wns.md
[Submit an app page]: https://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: https://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: https://go.microsoft.com/fwlink/p/?LinkId=262253
[Get started with Mobile Services]: /develop/mobile/tutorials/get-started/#create-new-service
[Get started with data]: /develop/mobile/tutorials/get-started-with-data-dotnet
[Get started with authentication]: /develop/mobile/tutorials/get-started-with-users-dotnet
[Get started with push notifications]: /develop/mobile/tutorials/get-started-with-push-dotnet
[Push notifications to app users]: /develop/mobile/tutorials/push-notifications-to-app-users-dotnet
[Authorize users with scripts]: /develop/mobile/tutorials/authorize-users-in-scripts-dotnet
[JavaScript and HTML]: /develop/mobile/tutorials/get-started-with-push-js
[wns object]: https://go.microsoft.com/fwlink/p/?LinkId=260591
[Notification Hubs Guidance]: /previous-versions/azure/azure-services/jj927170(v=azure.100)
[Notification Hubs How-To for iOS]: /previous-versions/azure/azure-services/jj927170(v=azure.100)
[Notification Hubs How-To for Windows Store]: /previous-versions/azure/azure-services/jj927170(v=azure.100)
