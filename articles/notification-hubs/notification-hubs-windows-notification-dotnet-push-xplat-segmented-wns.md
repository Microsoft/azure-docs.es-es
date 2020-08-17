---
title: Envío de notificaciones a dispositivos concretos (Plataforma universal de Windows) | Microsoft Docs
description: Use Azure Notification Hubs con etiquetas en el registro para enviar noticias de última hora a una aplicación de la Plataforma universal de Windows.
services: notification-hubs
documentationcenter: windows
author: sethmanheim
manager: femila
editor: jwargo
ms.assetid: 994d2eed-f62e-433c-bf65-4afebf1c0561
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-windows
ms.devlang: dotnet
ms.topic: tutorial
ms.custom: mvc
ms.date: 09/30/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 03/22/2019
ms.openlocfilehash: be20cd583be7c9c87d9f3a3cebc6609764384c9a
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87084235"
---
# <a name="tutorial-send-notifications-to-specific-devices-running-universal-windows-platform-applications"></a>Tutorial: Envío de notificaciones a dispositivos concretos que ejecutan aplicaciones de la Plataforma universal de Windows

[!INCLUDE [notification-hubs-selector-breaking-news](../../includes/notification-hubs-selector-breaking-news.md)]

## <a name="overview"></a>Información general

En este tutorial se explica el uso de Azure Notification Hubs para difundir notificaciones de noticias de última hora. En este tutorial se describen las aplicaciones de la Tienda Windows o Windows Phone 8.1 (que no son de Silverlight). Si el destino es Windows Phone 8.1 Silverlight, consulte [Envío de notificaciones push a dispositivos Windows Phone concretos mediante Azure Notification Hubs](notification-hubs-windows-phone-push-xplat-segmented-mpns-notification.md).

En este tutorial aprenderá a usar Azure Notification Hubs para enviar notificaciones push a dispositivos Windows concretos en los que se ejecuta una aplicación de la Plataforma universal de Windows (UWP). Después de completar el tutorial, puede registrarse en las categorías de noticias de última hora que le interesen. Solo recibirá notificaciones push de esas categorías.

Para habilitar escenarios de difusión, incluya una o varias *etiquetas* al crear un registro en el centro de notificaciones. Cuando las notificaciones se envían a una etiqueta, todos los dispositivos registrados en ella reciben la notificación. Para más información sobre las etiquetas, consulte [Expresiones de etiqueta y enrutamiento](notification-hubs-tags-segment-push-message.md).

> [!NOTE]
> Los proyectos de la Tienda Windows y Windows Phone 8.1 y versiones anteriores no se admiten en Visual Studio 2019. Para más información, consulte [Compatibilidad y destinatarios de la plataforma de Visual Studio 2019](/visualstudio/releases/2019/compatibility).

En este tutorial se realizan las tareas siguientes:

> [!div class="checklist"]
> * Agregar una selección de categorías a la aplicación móvil
> * Registro de notificaciones
> * Envío de notificaciones con etiquetas
> * Ejecución de la aplicación y generación de notificaciones

## <a name="prerequisites"></a>Prerrequisitos

Complete el [Tutorial: Envío de notificaciones a aplicaciones de Plataforma universal de Windows mediante Azure Notification Hubs][get-started] antes de empezar este tutorial.  

## <a name="add-category-selection-to-the-app"></a>Adición de una selección de categorías a la aplicación

El primer paso consiste en agregar elementos de la interfaz de usuario a la página principal existente de modo que los usuarios puedan seleccionar categorías de registro. Las categorías seleccionadas se almacenan en el dispositivo. Cuando la aplicación se inicia, crea un registro de dispositivos en el centro de notificaciones con las categorías seleccionadas como etiquetas.

1. Abra el archivo de proyecto *MainPage.xaml* y copie el siguiente código en el elemento `Grid`:

    ```xml
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition/>
            <RowDefinition/>
            <RowDefinition/>
            <RowDefinition/>
            <RowDefinition/>
        </Grid.RowDefinitions>
        <Grid.ColumnDefinitions>
            <ColumnDefinition/>
            <ColumnDefinition/>
        </Grid.ColumnDefinitions>
        <TextBlock Grid.Row="0" Grid.Column="0" Grid.ColumnSpan="2"  TextWrapping="Wrap" Text="Breaking News" FontSize="42" VerticalAlignment="Top" HorizontalAlignment="Center"/>
        <ToggleSwitch Header="World" Name="WorldToggle" Grid.Row="1" Grid.Column="0" HorizontalAlignment="Center"/>
        <ToggleSwitch Header="Politics" Name="PoliticsToggle" Grid.Row="2" Grid.Column="0" HorizontalAlignment="Center"/>
        <ToggleSwitch Header="Business" Name="BusinessToggle" Grid.Row="3" Grid.Column="0" HorizontalAlignment="Center"/>
        <ToggleSwitch Header="Technology" Name="TechnologyToggle" Grid.Row="1" Grid.Column="1" HorizontalAlignment="Center"/>
        <ToggleSwitch Header="Science" Name="ScienceToggle" Grid.Row="2" Grid.Column="1" HorizontalAlignment="Center"/>
        <ToggleSwitch Header="Sports" Name="SportsToggle" Grid.Row="3" Grid.Column="1" HorizontalAlignment="Center"/>
        <Button Name="SubscribeButton" Content="Subscribe" HorizontalAlignment="Center" Grid.Row="4" Grid.Column="0" Grid.ColumnSpan="2" Click="SubscribeButton_Click"/>
    </Grid>
    ```

1. En el **Explorador de soluciones**, haga clic con el botón derecho en el proyecto y seleccione **Agregar** > **Clase**. En **Agregar nuevo elemento**, asígnele a la clase el nombre *Notifications* y seleccione **Agregar**. Si es necesario, agregue el modificador `public` a la definición de la clase.

1. Agregue las siguientes instrucciones `using` al archivo nuevo:

    ```csharp
    using Windows.Networking.PushNotifications;
    using Microsoft.WindowsAzure.Messaging;
    using Windows.Storage;
    using System.Threading.Tasks;
    ```

1. Copie el código siguiente en la clase `Notifications` nueva:

    ```csharp
    private NotificationHub hub;

    public Notifications(string hubName, string listenConnectionString)
    {
        hub = new NotificationHub(hubName, listenConnectionString);
    }

    public async Task<Registration> StoreCategoriesAndSubscribe(IEnumerable<string> categories)
    {
        ApplicationData.Current.LocalSettings.Values["categories"] = string.Join(",", categories);
        return await SubscribeToCategories(categories);
    }

    public IEnumerable<string> RetrieveCategories()
    {
        var categories = (string) ApplicationData.Current.LocalSettings.Values["categories"];
        return categories != null ? categories.Split(','): new string[0];
    }

    public async Task<Registration> SubscribeToCategories(IEnumerable<string> categories = null)
    {
        var channel = await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();

        if (categories == null)
        {
            categories = RetrieveCategories();
        }

        // Using a template registration to support notifications across platforms.
        // Any template notifications that contain messageParam and a corresponding tag expression
        // will be delivered for this registration.

        const string templateBodyWNS = "<toast><visual><binding template=\"ToastText01\"><text id=\"1\">$(messageParam)</text></binding></visual></toast>";

        return await hub.RegisterTemplateAsync(channel.Uri, templateBodyWNS, "simpleWNSTemplateExample",
                categories);
    }
    ```

    Esta clase usa el almacenamiento local para almacenar las categorías de noticias que este dispositivo debe recibir. En lugar de llamar al método `RegisterNativeAsync`, llame al método `RegisterTemplateAsync` para registrar las categorías mediante un registro de plantilla.

    Si desea registrar más de una plantilla, especifique un nombre de plantilla, por ejemplo, *simpleWNSTemplateExample*. Se asigna un nombre a las plantillas para que pueda actualizarlas o eliminarlas. Puede registrar más de una plantilla, por ejemplo, para tener una para las notificaciones del sistema y otra para los iconos.

    >[!NOTE]
    > Con Notification Hubs, un dispositivo puede registrar varias plantillas con la misma etiqueta. En este caso, un mensaje entrante dirigido a esa etiqueta da lugar a que se entreguen varias notificaciones al dispositivo, una por cada plantilla. Este proceso le permite mostrar el mismo mensaje en varias notificaciones visuales, como distintivo y como notificación del sistema en una aplicación de la Tienda Windows.

    Para obtener más información, consulte [Plantillas](notification-hubs-templates-cross-platform-push-messages.md).

1. En el archivo de proyecto *App.xaml.cs*, agregue la siguiente propiedad a la clase `App`:

    ```csharp
    public Notifications notifications = new Notifications("<hub name>", "<connection string with listen access>");
    ```

    Esta propiedad se usa para crear una instancia de `Notifications` y tener acceso a ella.

    En el código, reemplace los marcadores de posición `<hub name>` y `<connection string with listen access>` por el nombre del centro de notificaciones y la cadena de conexión de **DefaultListenSharedAccessSignature** que obtuvo anteriormente.

   > [!NOTE]
   > Puesto que las credenciales que se distribuyen con una aplicación cliente no son normalmente seguras, distribuya solo la clave para el acceso de *escucha* con la aplicación cliente. El acceso de escucha permite a la aplicación registrarse en las notificaciones, pero los registros existentes no pueden modificarse y las notificaciones no se pueden enviar. La clave de acceso completo se usa en un servicio back-end protegido para el envío de notificaciones y el cambio de registros existentes.

1. En el archivo *MainPage.xaml.cs*, agregue la siguiente línea:

    ```csharp
    using Windows.UI.Popups;
    ```

1. En el archivo *MainPage.xaml.cs*, agregue el siguiente método:

    ```csharp
    private async void SubscribeButton_Click(object sender, RoutedEventArgs e)
    {
        var categories = new HashSet<string>();
        if (WorldToggle.IsOn) categories.Add("World");
        if (PoliticsToggle.IsOn) categories.Add("Politics");
        if (BusinessToggle.IsOn) categories.Add("Business");
        if (TechnologyToggle.IsOn) categories.Add("Technology");
        if (ScienceToggle.IsOn) categories.Add("Science");
        if (SportsToggle.IsOn) categories.Add("Sports");

        var result = await ((App)Application.Current).notifications.StoreCategoriesAndSubscribe(categories);

        var dialog = new MessageDialog("Subscribed to: " + string.Join(",", categories) + " on registration Id: " + result.RegistrationId);
        dialog.Commands.Add(new UICommand("OK"));
        await dialog.ShowAsync();
    }
    ```

    Este método crea una lista de categorías y usa la clase `Notifications` para almacenar la lista en el almacenamiento local. También registra las etiquetas correspondientes en el centro de notificaciones. Cuando se modifican las categorías, el registro vuelve a crearse con las nuevas categorías.

La aplicación puede almacenar ahora un conjunto de categorías en el almacenamiento local en el dispositivo. La aplicación se registra en el centro de notificaciones siempre que los usuarios cambian la selección de categoría.

## <a name="register-for-notifications"></a>Registro de notificaciones

En esta sección, se registrará en el centro de notificaciones al inicio mediante las categorías que ha almacenado en el almacenamiento local.

> [!NOTE]
> Puesto que el URI de canal asignado por el Servicio de notificaciones de Windows (WNS) puede cambiar en cualquier momento, debe registrarse en las notificaciones con frecuencia a fin de evitar errores de notificación. En este ejemplo se registra la notificación cada vez que se inicia la aplicación. En las aplicaciones que se ejecutan con frecuencia (más de una vez al día), es posible que pueda omitir el registro para conservar el ancho de banda si pasa menos de un día desde el registro anterior.

1. Para usar la clase `notifications` para suscribirse por categorías, abra el archivo *App.xaml.cs* y actualice el método `InitNotificationsAsync`.

    ```csharp
    // *** Remove or comment out these lines ***
    //var channel = await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();
    //var hub = new NotificationHub("your hub name", "your listen connection string");
    //var result = await hub.RegisterNativeAsync(channel.Uri);

    var result = await notifications.SubscribeToCategories();
    ```

    Este proceso garantiza que cuando se inicia la aplicación, recupera las categorías del almacenamiento local. A continuación, solicita el registro de estas categorías. Ha creado el método `InitNotificationsAsync` como parte del tutorial [Envío de notificaciones a aplicaciones de la Plataforma universal de Windows mediante Azure Notification Hubs][get-started].
2. En el archivo de proyecto *MainPage.xaml.cs*, agregue el siguiente código al método `OnNavigatedTo`:

    ```csharp
    protected override void OnNavigatedTo(NavigationEventArgs e)
    {
        var categories = ((App)Application.Current).notifications.RetrieveCategories();

        if (categories.Contains("World")) WorldToggle.IsOn = true;
        if (categories.Contains("Politics")) PoliticsToggle.IsOn = true;
        if (categories.Contains("Business")) BusinessToggle.IsOn = true;
        if (categories.Contains("Technology")) TechnologyToggle.IsOn = true;
        if (categories.Contains("Science")) ScienceToggle.IsOn = true;
        if (categories.Contains("Sports")) SportsToggle.IsOn = true;
    }
    ```

    Este código actualiza la página principal según el estado de las categorías guardadas anteriormente.

La aplicación ya está completa y puede almacenar un conjunto de categorías en el almacenamiento local del dispositivo. Cuando los usuarios cambian la selección de categoría, las guardadas se utilizan para el registro en el centro de notificaciones. En la siguiente sección, definirá un back-end que puede enviar notificaciones de categoría a esta aplicación.

## <a name="run-the-uwp-app"></a>Ejecución de la aplicación para UWP

1. En Visual Studio, presione F5 para compilar e iniciar la aplicación. La interfaz de usuario de la aplicación ofrece un conjunto de elementos de alternancia que le permiten seleccionar las categorías a las que suscribirse.

   ![Aplicación de noticias de última hora](./media/notification-hubs-windows-store-dotnet-send-breaking-news/notification-hub-breaking-news.png)

1. Habilite uno o más elementos de alternancia de categorías y seleccione **Suscribirse**.

   La aplicación convierte las categorías seleccionadas en etiquetas y solicita un nuevo registro de dispositivo para las etiquetas seleccionadas al Centro de notificaciones. La aplicación muestra las categorías registradas en un cuadro de diálogo.

    ![Elementos de alternancia de categorías y el botón Suscribirse](./media/notification-hubs-windows-store-dotnet-send-breaking-news/notification-hub-windows-toast.png)

## <a name="create-a-console-app-to-send-tagged-notifications"></a>Creación de una aplicación de consola para enviar notificaciones con etiquetas

[!INCLUDE [notification-hubs-send-categories-template](../../includes/notification-hubs-send-categories-template.md)]

## <a name="run-the-console-app-to-send-tagged-notifications"></a>Ejecución de la aplicación de consola para enviar notificaciones con etiquetas

Ejecute la aplicación creada en la sección anterior. Las notificaciones para las categorías seleccionadas aparecen como notificaciones del sistema.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a difundir noticias de última hora por categoría. La aplicación de back-end inserta notificaciones con etiquetas en los dispositivos que se han registrado para recibir notificaciones de esas etiquetas. Para aprender a enviar notificaciones push a usuarios específicos, con independencia del dispositivo que usen, pase al siguiente tutorial:

> [!div class="nextstepaction"]
> [Envío de notificaciones push localizadas en aplicaciones Windows mediante Azure Notification Hubs](notification-hubs-windows-store-dotnet-xplat-localized-wns-push-notification.md)

<!-- Anchors. -->
[Add category selection to the app]: #adding-categories
[Register for notifications]: #register
[Send notifications from your back-end]: #send
[Run the app and generate notifications]: #test-app
[Next Steps]: #next-steps

<!-- URLs.-->
[get-started]: notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md
[Use Notification Hubs to broadcast localized breaking news]: notification-hubs-windows-store-dotnet-xplat-localized-wns-push-notification.md
[Notify users with Notification Hubs]: notification-hubs-aspnet-backend-windows-dotnet-wns-notification.md
[Mobile Service]: /develop/mobile/tutorials/get-started/
[Notification Hubs Guidance]: /previous-versions/azure/azure-services/jj927170(v=azure.100)
[Notification Hubs How-To for Windows Store]: /previous-versions/azure/azure-services/jj927170(v=azure.100)
[Submit an app page]: https://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: https://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: https://go.microsoft.com/fwlink/p/?LinkId=262253
[wns object]: https://go.microsoft.com/fwlink/p/?LinkId=260591
