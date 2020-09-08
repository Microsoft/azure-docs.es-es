---
title: Envío de notificaciones a usuarios concretos mediante Azure Notification Hubs | Microsoft Docs
description: Aprenda a enviar notificaciones a usuarios concretos que usen aplicaciones de la Plataforma universal de Windows (UWP).
documentationcenter: windows
author: sethmanheim
manager: femila
editor: jwargo
services: notification-hubs
ms.assetid: 012529f2-fdbc-43c4-8634-2698164b5880
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
ms.openlocfilehash: 865aaf748fd8fad5f10350cb5b57d31b3eadf7a0
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89018049"
---
# <a name="tutorial-send-notifications-to-specific-users-by-using-azure-notification-hubs"></a>Tutorial: Envío de notificaciones a usuarios concretos mediante Azure Notification Hubs

[!INCLUDE [notification-hubs-selector-aspnet-backend-notify-users](../../includes/notification-hubs-selector-aspnet-backend-notify-users.md)]

## <a name="overview"></a>Información general

Este tutorial muestra cómo puede utilizar los Centros de notificaciones de Azure para enviar notificaciones de inserción a un usuario de aplicaciones determinado en un dispositivo concreto. Se usa un back-end de WebAPI de ASP.NET para autenticar a los clientes. Cuando el back-end autentica un usuario de la aplicación cliente, agrega automáticamente una etiqueta al registro de notificaciones. El back-end utiliza dicha etiqueta para enviar notificaciones al usuario concreto.

> [!NOTE]
> El código completo de este tutorial se puede encontrar en [GitHub](https://github.com/Azure/azure-notificationhubs-dotnet/tree/master/Samples/NotifyUsers).

En este tutorial, realizará los siguientes pasos:

> [!div class="checklist"]
> * Creación del proyecto de API web
> * Autenticar clientes en el back-end de WebAPI
> * Registrar notificaciones mediante el back-end de WebAPI
> * Enviar notificaciones desde el back-end de WebAPI
> * Publicación del nuevo back-end de WebAPI
> * Actualización del código para el proyecto de cliente
> * Prueba de la aplicación

## <a name="prerequisites"></a>Requisitos previos

Este tutorial se basa en el centro de notificaciones y el proyecto de Visual Studio que creó en el [Tutorial: Envío de notificaciones a aplicaciones de Plataforma universal de Windows mediante Azure Notification Hubs](notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md). Por consiguiente, complételo antes de iniciar este.

> [!NOTE]
> Si usa Mobile Apps como su servicio back-end en Azure App Service, consulte la [versión de Mobile Apps](/previous-versions/azure/app-service-mobile/app-service-mobile-windows-store-dotnet-get-started-push) de este tutorial.

[!INCLUDE [notification-hubs-aspnet-backend-notifyusers](../../includes/notification-hubs-aspnet-backend-notifyusers.md)]

## <a name="update-the-code-for-the-uwp-client"></a>Actualización del código para el cliente de UWP

En esta sección, actualizará el código del proyecto que completó en el [Tutorial: Envío de notificaciones a aplicaciones de Plataforma universal de Windows mediante Azure Notification Hubs](notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md). El proyecto ya debe estar asociado a la Tienda Windows. También debe estar configurado para usar el Centro de notificaciones. En esta sección, agregue código para llamar el nuevo back-end de WebAPI y usarlo para registrar y enviar notificaciones.

1. En Visual Studio, abra la solución que creó en el [Tutorial: Envío de notificaciones a aplicaciones de Plataforma universal de Windows mediante Azure Notification Hubs](notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md).
2. En el Explorador de soluciones, haga clic con el botón derecho en el proyecto Universal Windows Platform (UWP) y, a continuación, haga clic en **Administrar paquetes de NuGet**.
3. En el lado izquierdo, seleccione **Examinar**.
4. En el cuadro **Buscar**, escriba **Cliente http**.
5. En la lista de resultados, haga clic en **System.Net.Http**y haga clic en **Instalar**. Complete la instalación.
6. Nuevamente en el cuadro **Buscar** de NuGet, escriba **Json.net**. Instale el paquete **Newtonsoft.json** y cierre la ventana Administrador de paquetes NuGet.
7. En el Explorador de soluciones, en el proyecto **WindowsApp**, haga doble clic en **MainPage.xaml** para abrirlo en el editor de Visual Studio.
8. En el código XML `MainPage.xaml`, sustituya la sección `<Grid>` por el código siguiente: Este código agrega un cuadro de texto de nombre de usuario y contraseña con el que el usuario se autentica. También agrega cuadros de texto para el mensaje de notificación y la etiqueta de nombre de usuario que debería recibir la notificación:

    ```xml
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
        </Grid.RowDefinitions>

        <TextBlock Grid.Row="0" Text="Notify Users" HorizontalAlignment="Center" FontSize="48"/>

        <StackPanel Grid.Row="1" VerticalAlignment="Center">
            <Grid>
                <Grid.RowDefinitions>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                    <RowDefinition Height="Auto"/>
                </Grid.RowDefinitions>
                <Grid.ColumnDefinitions>
                    <ColumnDefinition></ColumnDefinition>
                    <ColumnDefinition></ColumnDefinition>
                    <ColumnDefinition></ColumnDefinition>
                </Grid.ColumnDefinitions>
                <TextBlock Grid.Row="0" Grid.ColumnSpan="3" Text="Username" FontSize="24" Margin="20,0,20,0"/>
                <TextBox Name="UsernameTextBox" Grid.Row="1" Grid.ColumnSpan="3" Margin="20,0,20,0"/>
                <TextBlock Grid.Row="2" Grid.ColumnSpan="3" Text="Password" FontSize="24" Margin="20,0,20,0" />
                <PasswordBox Name="PasswordTextBox" Grid.Row="3" Grid.ColumnSpan="3" Margin="20,0,20,0"/>

                <Button Grid.Row="4" Grid.ColumnSpan="3" HorizontalAlignment="Center" VerticalAlignment="Center"
                            Content="1. Login and register" Click="LoginAndRegisterClick" Margin="0,0,0,20"/>

                <ToggleButton Name="toggleWNS" Grid.Row="5" Grid.Column="0" HorizontalAlignment="Right" Content="WNS" IsChecked="True" />
                <ToggleButton Name="toggleFCM" Grid.Row="5" Grid.Column="1" HorizontalAlignment="Center" Content="FCM" />
                <ToggleButton Name="toggleAPNS" Grid.Row="5" Grid.Column="2" HorizontalAlignment="Left" Content="APNS" />

                <TextBlock Grid.Row="6" Grid.ColumnSpan="3" Text="Username Tag To Send To" FontSize="24" Margin="20,0,20,0"/>
                <TextBox Name="ToUserTagTextBox" Grid.Row="7" Grid.ColumnSpan="3" Margin="20,0,20,0" TextWrapping="Wrap" />
                <TextBlock Grid.Row="8" Grid.ColumnSpan="3" Text="Enter Notification Message" FontSize="24" Margin="20,0,20,0"/>
                <TextBox Name="NotificationMessageTextBox" Grid.Row="9" Grid.ColumnSpan="3" Margin="20,0,20,0" TextWrapping="Wrap" />
                <Button Grid.Row="10" Grid.ColumnSpan="3" HorizontalAlignment="Center" Content="2. Send push" Click="PushClick" Name="SendPushButton" />
            </Grid>
        </StackPanel>
    </Grid>
    ```
9. En el Explorador de soluciones, abra el archivo `MainPage.xaml.cs` de los proyectos **(Windows 8.1)** y **(Windows Phone 8.1)** . Agregue las siguientes instrucciones `using` en la parte superior de ambos archivos:

    ```csharp
    using System.Net.Http;
    using Windows.Storage;
    using System.Net.Http.Headers;
    using Windows.Networking.PushNotifications;
    using Windows.UI.Popups;
    using System.Threading.Tasks;
    ```
10. En `MainPage.xaml.cs` del proyecto **WindowsApp**, agregue el siguiente miembro a la clase `MainPage`. Asegúrese de reemplazar `<Enter Your Backend Endpoint>` por el punto de conexión del back-end obtenido anteriormente. Por ejemplo, `http://mybackend.azurewebsites.net`.

    ```csharp
    private static string BACKEND_ENDPOINT = "<Enter Your Backend Endpoint>";
    ```
11. Agregue el código siguiente a la clase MainPage en `MainPage.xaml.cs` para los proyectos **(Windows 8.1)** y **(Windows Phone 8.1)** .

    El método `PushClick` es el controlador de clics para el botón **Enviar inserción** . Llama al back-end para desencadenar una notificación a todos los dispositivos con una etiqueta de nombre de usuario que coincida con el parámetro `to_tag` . El mensaje de notificación se envía como contenido JSON en el cuerpo de la solicitud.

    El método `LoginAndRegisterClick` es el controlador de clics del botón **Login and register** (Iniciar sesión y registrarse). Almacena el token de autenticación básico (representa cualquier token que usa el esquema de autenticación) localmente y después usa `RegisterClient` para registrar las notificaciones que usan el back-end.

    ```csharp
    private async void PushClick(object sender, RoutedEventArgs e)
    {
        if (toggleWNS.IsChecked.Value)
        {
            await sendPush("wns", ToUserTagTextBox.Text, this.NotificationMessageTextBox.Text);
        }
        if (toggleFCM.IsChecked.Value)
        {
            await sendPush("fcm", ToUserTagTextBox.Text, this.NotificationMessageTextBox.Text);
        }
        if (toggleAPNS.IsChecked.Value)
        {
            await sendPush("apns", ToUserTagTextBox.Text, this.NotificationMessageTextBox.Text);

        }
    }

    private async Task sendPush(string pns, string userTag, string message)
    {
        var POST_URL = BACKEND_ENDPOINT + "/api/notifications?pns=" +
            pns + "&to_tag=" + userTag;

        using (var httpClient = new HttpClient())
        {
            var settings = ApplicationData.Current.LocalSettings.Values;
            httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Basic", (string)settings["AuthenticationToken"]);

            try
            {
                await httpClient.PostAsync(POST_URL, new StringContent("\"" + message + "\"",
                    System.Text.Encoding.UTF8, "application/json"));
            }
            catch (Exception ex)
            {
                MessageDialog alert = new MessageDialog(ex.Message, "Failed to send " + pns + " message");
                alert.ShowAsync();
            }
        }
    }

    private async void LoginAndRegisterClick(object sender, RoutedEventArgs e)
    {
        SetAuthenticationTokenInLocalStorage();

        var channel = await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();

        // The "username:<user name>" tag gets automatically added by the message handler in the backend.
        // The tag passed here can be whatever other tags you may want to use.
        try
        {
            // The device handle used is different depending on the device and PNS.
            // Windows devices use the channel uri as the PNS handle.
            await new RegisterClient(BACKEND_ENDPOINT).RegisterAsync(channel.Uri, new string[] { "myTag" });

            var dialog = new MessageDialog("Registered as: " + UsernameTextBox.Text);
            dialog.Commands.Add(new UICommand("OK"));
            await dialog.ShowAsync();
            SendPushButton.IsEnabled = true;
        }
        catch (Exception ex)
        {
            MessageDialog alert = new MessageDialog(ex.Message, "Failed to register with RegisterClient");
            alert.ShowAsync();
        }
    }

    private void SetAuthenticationTokenInLocalStorage()
    {
        string username = UsernameTextBox.Text;
        string password = PasswordTextBox.Password;

        var token = Convert.ToBase64String(System.Text.Encoding.UTF8.GetBytes(username + ":" + password));
        ApplicationData.Current.LocalSettings.Values["AuthenticationToken"] = token;
    }
    ```
12. Abra `App.xaml.cs` y encuentre la llamada a `InitNotificationsAsync()` en el controlador de eventos `OnLaunched()`. Marque como comentario o elimine la llamada a `InitNotificationsAsync()`. El controlador del botón inicializa los registros de notificaciones.

    ```csharp
    protected override void OnLaunched(LaunchActivatedEventArgs e)
    {
        //InitNotificationsAsync();
    ```
13. Haga clic con el botón derecho en el proyecto **WindowsApp**, haga clic en **Agregar** y, después, en **Clase**. Asigne un nombre a la clase `RegisterClient.cs` y, luego, haga clic en **Aceptar** para generar la clase.

    Esta clase contiene las llamadas REST requeridas para ponerse en contacto con el back-end de la aplicación con la finalidad de registrarlas para las notificaciones push. También almacena localmente los identificadores *registrationIds* creados por el Centro de notificaciones, como se detalla en la sección [Administración de registros desde un back-end](/previous-versions/azure/azure-services/dn743807(v=azure.100)). Usa un token de autorización almacenado localmente cuando hace clic en el botón **Login and register** (Iniciar sesión y registrarse).
14. Agregue las siguientes instrucciones `using` en la parte superior del archivo RegisterClient.cs:

    ```csharp
    using Windows.Storage;
    using System.Net;
    using System.Net.Http;
    using System.Net.Http.Headers;
    using Newtonsoft.Json;
    using System.Threading.Tasks;
    using System.Linq;
    ```
15. Agregue el siguiente código a la definición de clase `RegisterClient` .

    ```csharp
    private string POST_URL;

    private class DeviceRegistration
    {
        public string Platform { get; set; }
        public string Handle { get; set; }
        public string[] Tags { get; set; }
    }

    public RegisterClient(string backendEndpoint)
    {
        POST_URL = backendEndpoint + "/api/register";
    }

    public async Task RegisterAsync(string handle, IEnumerable<string> tags)
    {
        var regId = await RetrieveRegistrationIdOrRequestNewOneAsync();

        var deviceRegistration = new DeviceRegistration
        {
            Platform = "wns",
            Handle = handle,
            Tags = tags.ToArray<string>()
        };

        var statusCode = await UpdateRegistrationAsync(regId, deviceRegistration);

        if (statusCode == HttpStatusCode.Gone)
        {
            // regId is expired, deleting from local storage & recreating
            var settings = ApplicationData.Current.LocalSettings.Values;
            settings.Remove("__NHRegistrationId");
            regId = await RetrieveRegistrationIdOrRequestNewOneAsync();
            statusCode = await UpdateRegistrationAsync(regId, deviceRegistration);
        }

        if (statusCode != HttpStatusCode.Accepted && statusCode != HttpStatusCode.OK)
        {
            // log or throw
            throw new System.Net.WebException(statusCode.ToString());
        }
    }

    private async Task<HttpStatusCode> UpdateRegistrationAsync(string regId, DeviceRegistration deviceRegistration)
    {
        using (var httpClient = new HttpClient())
        {
            var settings = ApplicationData.Current.LocalSettings.Values;
            httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Basic", (string) settings["AuthenticationToken"]);

            var putUri = POST_URL + "/" + regId;

            string json = JsonConvert.SerializeObject(deviceRegistration);
                            var response = await httpClient.PutAsync(putUri, new StringContent(json, Encoding.UTF8, "application/json"));
            return response.StatusCode;
        }
    }

    private async Task<string> RetrieveRegistrationIdOrRequestNewOneAsync()
    {
        var settings = ApplicationData.Current.LocalSettings.Values;
        if (!settings.ContainsKey("__NHRegistrationId"))
        {
            using (var httpClient = new HttpClient())
            {
                httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Basic", (string)settings["AuthenticationToken"]);

                var response = await httpClient.PostAsync(POST_URL, new StringContent(""));
                if (response.IsSuccessStatusCode)
                {
                    string regId = await response.Content.ReadAsStringAsync();
                    regId = regId.Substring(1, regId.Length - 2);
                    settings.Add("__NHRegistrationId", regId);
                }
                else
                {
                    throw new System.Net.WebException(response.StatusCode.ToString());
                }
            }
        }
        return (string)settings["__NHRegistrationId"];

    }
    ```
16. Guarde todos los cambios.

## <a name="test-the-application"></a>Prueba de la aplicación

1. Inicie la aplicación en ambas versiones de Windows.
2. Especifique un **nombre de usuario** y una **contraseña**, tal como se muestra en la pantalla siguiente. Debe diferir del nombre de usuario y contraseña que escriba en Windows Phone.
3. Haga clic en **Iniciar sesión y registrarse** y compruebe que el cuadro de diálogo se muestre que ha iniciado sesión. Este código también habilita el botón **Send Push** (Enviar inserción).

    ![Captura de pantalla de la aplicación Notification Hubs que muestra el nombre de usuario y la contraseña rellenados.][14]
5. Después, en el campo **Recipient Username Tag** (Etiqueta de nombre de usuario destinatario), especifique el nombre de usuario registrado. Escriba un mensaje de notificación y haga clic en **Enviar inserción**.
6. Solo los dispositivos que se han registrado con la etiqueta de nombre de usuario coincidente reciben el mensaje de notificación.

    ![Captura de pantalla de la aplicación Notification Hubs que muestra el mensaje que se ha insertado.][15]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a enviar notificaciones push a usuarios concretos que tienen etiquetas asociadas a sus registros. Para aprender a enviar notificaciones push en función de la ubicación, pase al tutorial siguiente:

> [!div class="nextstepaction"]
>[Envío de notificaciones push basadas en la ubicación](notification-hubs-push-bing-spatial-data-geofencing-notification.md)

[9]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-secure-push9.png
[10]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-secure-push10.png
[11]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-secure-push11.png
[12]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-secure-push12.png
[13]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-wp-ui.png
[14]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-windows-instance-username.png
[15]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-notification-received.png
[16]: ./media/notification-hubs-aspnet-backend-windows-dotnet-notify-users/notification-hubs-wp-send-message.png

<!-- URLs. -->
[Get started with Notification Hubs]: notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md
[Secure Push]: notification-hubs-aspnet-backend-windows-dotnet-wns-secure-push-notification.md
[Use Notification Hubs to send breaking news]: notification-hubs-windows-notification-dotnet-push-xplat-segmented-wns.md
[Notification Hubs Guidance]: /previous-versions/azure/azure-services/jj927170(v=azure.100)
