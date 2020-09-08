---
title: Envío de notificaciones push a aplicaciones de Xamarin.Android mediante Azure Notification Hubs | Microsoft Docs
description: En este tutorial, obtenga información acerca de cómo usar Azure Notification Hubs para enviar notificaciones push a una aplicación Xamarin Android.
author: sethmanheim
manager: femila
editor: jwargo
services: notification-hubs
documentationcenter: xamarin
ms.assetid: 0be600fe-d5f3-43a5-9e5e-3135c9743e54
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-xamarin-android
ms.devlang: dotnet
ms.topic: tutorial
ms.custom: mvc, devx-track-csharp
ms.date: 08/01/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 08/01/2019
ms.openlocfilehash: ce47d2ae65a4ea9e6878381ce58d62cb6c998599
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88998210"
---
# <a name="tutorial-send-push-notifications-to-xamarinandroid-apps-using-notification-hubs"></a>Tutorial: Envío de notificaciones push a aplicaciones de Xamarin.Android mediante Notification Hubs

[!INCLUDE [notification-hubs-selector-get-started](../../includes/notification-hubs-selector-get-started.md)]

## <a name="overview"></a>Información general

Este tutorial muestra cómo puede usar Azure Notification Hubs para enviar notificaciones push a una aplicación Xamarin.Android. Cree una aplicación de Xamarin.Android en blanco que reciba notificaciones push mediante Firebase Cloud Messaging (FCM). Use el Centro de notificaciones para difundir notificaciones push a todos los dispositivos que ejecutan la aplicación. El código acabado está disponible en el ejemplo de la [aplicación NotificationHubs](https://github.com/Azure/azure-notificationhubs-dotnet/tree/master/Samples/Xamarin/GetStartedXamarinAndroid).

En este tutorial, realizará los siguientes pasos:

> [!div class="checklist"]
> * Crear un proyecto de Firebase y habilitar Firebase Cloud Messaging
> * Creación de un centro de notificaciones
> * Crear una aplicación de Xamarin.Android y conectarla al Centro de notificaciones
> * Enviar notificaciones de prueba desde Azure Portal

## <a name="prerequisites"></a>Requisitos previos

* **Suscripción de Azure**. Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.
* [Visual Studio con Xamarin] en Windows o [Visual Studio para Mac] en OS X.
* Cuenta de Google activa

## <a name="create-a-firebase-project-and-enable-firebase-cloud-messaging"></a>Crear un proyecto de Firebase y habilitar Firebase Cloud Messaging

[!INCLUDE [notification-hubs-enable-firebase-cloud-messaging](../../includes/notification-hubs-enable-firebase-cloud-messaging-xamarin.md)]

## <a name="create-a-notification-hub"></a>Creación de un centro de notificaciones

[!INCLUDE [notification-hubs-portal-create-new-hub](../../includes/notification-hubs-portal-create-new-hub.md)]

### <a name="configure-gcmfcm-settings-for-the-notification-hub"></a>Configuración de los valores de GCM/FCM para el Centro de notificaciones

1. Seleccione **Google (GCM/FCM)** en la sección **Configuración** en el menú de la izquierda.
2. Escriba la **clave de servidor** que anotó en Google Firebase Console.
3. Seleccione **Guardar** en la barra de herramientas.

    ![Captura de pantalla del centro de notificaciones en Azure Portal con la opción Google G C M F C M resaltada y destacada en rojo.](./media/notification-hubs-android-get-started/notification-hubs-gcm-api.png)

El centro de notificaciones ya está configurado para funcionar con FCM y dispone de las cadenas de conexión para registrar la aplicación para que reciba notificaciones y para enviar notificaciones push.

## <a name="create-a-xamarinandroid-app-and-connect-it-to-notification-hub"></a>Creación de una aplicación de Xamarin.Android y su conexión al centro de notificaciones

### <a name="create-visual-studio-project-and-add-nuget-packages"></a>Creación de un proyecto de Visual Studio y adición de paquetes NuGet

> [!NOTE]
> Los pasos documentados en este tutorial son para Visual Studio 2017. 

1. En Visual Studio, en el menú **Archivo**, seleccione **Nuevo** y, después, **Proyecto**. En la ventana **Nuevo proyecto**, siga estos pasos:
    1. Expanda **Instalado**, **Visual C#** y haga clic en **Android**.
    2. Seleccione **Aplicación de Android (Xamarin)** en la lista.
    3. Escriba un **nombre** para el proyecto.
    4. Seleccione una **ubicación** para el proyecto.
    5. Seleccione **Aceptar**.

        ![Cuadro de diálogo Nuevo proyecto](./media/partner-xamarin-notification-hubs-android-get-started/new-project-dialog-new.png)
2. En el cuadro de diálogo **New Android App** (Nueva aplicación de Android), seleccione **Blank App** (Aplicación vacía) y seleccione **Aceptar**.

    ![Cuadro de diálogo Nuevo proyecto](./media/partner-xamarin-notification-hubs-android-get-started/new-android-app-dialog.png)
3. En la ventana del **Explorador de soluciones**, expanda **Propiedades**y haga clic en **AndroidManifest.xml**. Actualice el nombre del paquete para que coincida con el nombre del paquete que especificó al agregar Firebase Cloud Messaging al proyecto en Google Firebase Console.

    ![Nombre del paquete en GCM](./media/partner-xamarin-notification-hubs-android-get-started/package-name-gcm.png)
4. Establezca la versión de Android de destino del proyecto en**Android 9.0 (Pie)** siguiendo estos pasos: 
    1. Haga clic con el botón derecho en el proyecto y seleccione **Propiedades**. 
    1. En el campo **Compilar mediante la versión de Android: (plataforma de destino)** , seleccione **Android 9.0 (Pie)** . 
    1. Seleccione **Sí** en el cuadro de mensaje para continuar con el cambio de versión de la plataforma de destino.
1. Agregue los paquetes de NuGet necesarios al proyecto siguiendo estos pasos:
    1. Haga clic con el botón derecho en el proyecto y seleccione **Administrar paquetes NuGet...**
    1. Vaya a la pestaña **Instalado**, seleccione **Xamarin.Android.Support.Design** y **Actualizar** en el panel derecho para actualizar el paquete a la versión más reciente.
    1. Cambie a la pestaña **Examinar**. Busque **Xamarin.GooglePlayServices.Base**. Seleccione **Xamarin.GooglePlayServices.Base** en la lista de resultados. Luego, seleccione **Instalar**.

        ![Google Play Services NuGet](./media/partner-xamarin-notification-hubs-android-get-started/google-play-services-nuget.png)
    6. En la ventana **Administrador de paquetes NuGet**, busque **Xamarin.Firebase.Messaging**. Seleccione **Xamarin.Firebase.Messaging** en la lista de resultados. Luego, seleccione **Instalar**.
    7. Ahora, busque **Xamarin.Azure.NotificationHubs.Android**. Seleccione **Xamarin.Azure.NotificationHubs.Android** en la lista de resultados. Luego, seleccione **Instalar**.

### <a name="add-the-google-services-json-file"></a>Adición del archivo JSON de Google Services

1. Copie el archivo `google-services.json` que ha descargado de Google Firebase Console a la carpeta del proyecto.
2. Agregue `google-services.json` al proyecto.
3. Seleccione `google-services.json` en la ventana **Explorador de soluciones**.
4. En el panel **Propiedades**, seleccione **GoogleServicesJson** en Acción de compilación. Si no ve **GoogleServicesJson**, cierre Visual Studio, vuelva a iniciarlo, vuelva a abrir el proyecto y vuelva a intentarlo.

    ![Acción de compilación GoogleServicesJson](./media/partner-xamarin-notification-hubs-android-get-started/google-services-json-build-action.png)

### <a name="set-up-notification-hubs-in-your-project"></a>Configuración de los centros de notificaciones en su proyecto

#### <a name="registering-with-firebase-cloud-messaging"></a>Registro en Firebase Cloud Messaging

1. Abra el archivo `AndroidManifest.xml` e inserte los siguientes elementos `<receiver>` en el elemento `<application>`:

    ```xml
    <receiver android:name="com.google.firebase.iid.FirebaseInstanceIdInternalReceiver" android:exported="false" />
    <receiver android:name="com.google.firebase.iid.FirebaseInstanceIdReceiver" android:exported="true" android:permission="com.google.android.c2dm.permission.SEND">
        <intent-filter>
        <action android:name="com.google.android.c2dm.intent.RECEIVE" />
        <action android:name="com.google.android.c2dm.intent.REGISTRATION" />
        <category android:name="${applicationId}" />
        </intent-filter>
    </receiver>
    ```

2. Agregue las siguientes instrucciones **antes del elemento de la aplicación**.

    ```xml
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
    <uses-permission android:name="android.permission.WAKE_LOCK" />
    <uses-permission android:name="android.permission.GET_ACCOUNTS"/>
    ```

3. Recopile la siguiente información para el centro de notificaciones y la aplicación Android:

   * **Cadena de conexión de escucha**: en el panel de [Azure Portal], elija **Ver cadenas de conexión**. Copie la cadena de conexión `DefaultListenSharedAccessSignature` para este valor.
   * **Nombre del centro**: es el nombre del centro en [Azure Portal]. Por ejemplo, *mynotificationhub2*.
4. En la ventana **Explorador de soluciones**, haga clic con el botón derecho en el **proyecto**, seleccione **Agregar** y, después, **Clase**.
5. Cree una clase `Constants.cs` para el proyecto Xamarin y defina los siguientes valores de constante en la clase. Reemplace los marcadores de posición por sus valores.

    ```csharp
    public static class Constants
    {
        public const string ListenConnectionString = "<Listen connection string>";
        public const string NotificationHubName = "<hub name>";
    }
    ```

6. Agregue las siguientes instrucciones using a `MainActivity.cs`:

    ```csharp
    using Android.Util;
    using Android.Gms.Common;
    ```

7. Agregue las siguientes propiedades a la clase MainActivity: La variable TAG se usará para mostrar un cuadro de diálogo de alerta cuando se ejecuta la aplicación:

    ```csharp
    public const string TAG = "MainActivity";
    internal static readonly string CHANNEL_ID = "my_notification_channel";
    ```

8. Agregue el siguiente método a la clase MainActivity. Comprueba si **Google Play Services** está disponible en el dispositivo.

    ```csharp
    public bool IsPlayServicesAvailable()
    {
        int resultCode = GoogleApiAvailability.Instance.IsGooglePlayServicesAvailable(this);
        if (resultCode != ConnectionResult.Success)
        {
            if (GoogleApiAvailability.Instance.IsUserResolvableError(resultCode))
                Log.Debug(TAG, GoogleApiAvailability.Instance.GetErrorString(resultCode));
            else
            {
                Log.Debug(TAG, "This device is not supported");
                Finish();
            }
            return false;
        }

        Log.Debug(TAG, "Google Play Services is available.");
        return true;
    }
    ```

9. Agregue el método siguiente a la clase MainActivity que crea un canal de notificaciones.

    ```csharp
    private void CreateNotificationChannel()
    {
        if (Build.VERSION.SdkInt < BuildVersionCodes.O)
        {
            // Notification channels are new in API 26 (and not a part of the
            // support library). There is no need to create a notification
            // channel on older versions of Android.
            return;
        }

        var channelName = CHANNEL_ID;
        var channelDescription = string.Empty;
        var channel = new NotificationChannel(CHANNEL_ID, channelName, NotificationImportance.Default)
        {
            Description = channelDescription
        };

        var notificationManager = (NotificationManager)GetSystemService(NotificationService);
        notificationManager.CreateNotificationChannel(channel);
    }
    ```

10. En `MainActivity.cs`, agregue el siguiente código a `OnCreate` después de `base.OnCreate(savedInstanceState)`:

    ```csharp
    if (Intent.Extras != null)
    {
        foreach (var key in Intent.Extras.KeySet())
        {
            if(key!=null)
            {
                var value = Intent.Extras.GetString(key);
                Log.Debug(TAG, "Key: {0} Value: {1}", key, value);
            }
        }
    }

    IsPlayServicesAvailable();
    CreateNotificationChannel();
    ```

15. Agregue una clase denominada `MyFirebaseMessagingService` al proyecto. 
16. Agregue las siguientes instrucciones using a `MyFirebaseMessagingService.cs`.

    ```csharp
    using Android.Util;
    using Firebase.Messaging;
    using Android.Support.V4.App;    
    using WindowsAzure.Messaging;
    ```

17. Agregue lo siguiente sobre la declaración de clase y haga que la clase herede de `FirebaseMessagingService`:

    ```csharp
    [Service]
    [IntentFilter(new[] { "com.google.firebase.MESSAGING_EVENT" })]
    [IntentFilter(new[] { "com.google.firebase.INSTANCE_ID_EVENT" })]
    public class MyFirebaseMessagingService : FirebaseMessagingService
    ```

18. Agregue el código siguiente dentro de la clase `MyFirebaseMessagingService` para procesar los mensajes que se reciban. 

    ```csharp
        const string TAG = "MyFirebaseMsgService";
        NotificationHub hub;
    
        public override void OnMessageReceived(RemoteMessage message)
        {
            Log.Debug(TAG, "From: " + message.From);
            if (message.GetNotification() != null)
            {
                //These is how most messages will be received
                Log.Debug(TAG, "Notification Message Body: " + message.GetNotification().Body);
                SendNotification(message.GetNotification().Body);
            }
            else
            {
                //Only used for debugging payloads sent from the Azure portal
                SendNotification(message.Data.Values.First());
    
            }
        }
    
        void SendNotification(string messageBody)
        {
            var intent = new Intent(this, typeof(MainActivity));
            intent.AddFlags(ActivityFlags.ClearTop);
            var pendingIntent = PendingIntent.GetActivity(this, 0, intent, PendingIntentFlags.OneShot);
    
            var notificationBuilder = new NotificationCompat.Builder(this, MainActivity.CHANNEL_ID);
    
            notificationBuilder.SetContentTitle("FCM Message")
                        .SetSmallIcon(Resource.Drawable.ic_launcher)
                        .SetContentText(messageBody)
                        .SetAutoCancel(true)
                        .SetShowWhen(false)
                        .SetContentIntent(pendingIntent);
    
            var notificationManager = NotificationManager.FromContext(this);
    
            notificationManager.Notify(0, notificationBuilder.Build());
        }
    ```

19. Agregue los siguientes métodos a la clase MyFirebaseMessagingService (justo debajo del código agregado en el paso anterior) para recibir el token de registro de FCM y enviarlo a la instancia de Notification Hubs (centro). 

    ```csharp
        public override void OnNewToken(string token)
        {
            Log.Debug(TAG, "FCM token: " + token);
            SendRegistrationToServer(token);
        }

        void SendRegistrationToServer(string token)
        {
            // Register with Notification Hubs
            hub = new NotificationHub(Constants.NotificationHubName,
                                        Constants.ListenConnectionString, this);

            var tags = new List<string>() { };
            var regID = hub.Register(token, tags.ToArray()).RegistrationId;

            Log.Debug(TAG, $"Successful registration of ID {regID}");
        }
    ```
1. **Compile** el proyecto.
1. **Ejecute** la aplicación en su dispositivo o en el emulador cargado.

## <a name="send-test-notification-from-the-azure-portal"></a>Envío de notificaciones de prueba desde Azure Portal

Puede probar de recibir notificaciones en la aplicación con la opción **Envío de prueba** en [Azure Portal]. Envía una notificación push de prueba al dispositivo.

![Azure Portal: envío de prueba](media/partner-xamarin-notification-hubs-android-get-started/send-test-notification.png)

Las notificaciones push se envían normalmente en un servicio back-end como Mobile Services o ASP.NET mediante una biblioteca compatible. Si no hay disponible ninguna biblioteca para su back-end, también puede usar la API REST directamente para enviar mensajes de notificación.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial se envían las notificaciones de difusión a todos los dispositivos Android registrados con el back-end. Para aprender a enviar notificaciones push a dispositivos Android específicos, pase al siguiente tutorial:

> [!div class="nextstepaction"]
>[Envío de notificaciones push a dispositivos específicos](push-notifications-android-specific-devices-firebase-cloud-messaging.md)

<!-- Anchors. -->
[Enable Google Cloud Messaging]: #register
[Configure your Notification Hub]: #configure-hub
[Connecting your app to the Notification Hub]: #connecting-app
[Run your app with the emulator]: #run-app
[Send notifications from your back-end]: #send
[Next steps]:#next-steps

<!-- Images. -->
[11]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-configure-android.png
[13]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-create-xamarin-android-app1.png
[15]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-create-xamarin-android-app3.png
[18]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-create-android-app7.png
[19]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-create-android-app8.png
[20]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-create-console-app.png
[21]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-android-toast.png
[22]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-create-xamarin-android-project1.png
[23]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-create-xamarin-android-project2.png
[24]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-xamarin-android-app-options.png
[25]: ./media/partner-xamarin-notification-hubs-android-get-started/notification-hub-google-services-json.png
[30]: ./media/notification-hubs-android-get-started/notification-hubs-test-send.png

<!-- URLs. -->
[Submit an app page]: https://go.microsoft.com/fwlink/p/?LinkID=266582
[My Applications]: https://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: https://go.microsoft.com/fwlink/p/?LinkId=262253
[Get started with Mobile Services]: /develop/mobile/tutorials/get-started-xamarin-android/#create-new-service
[JavaScript and HTML]: /develop/mobile/tutorials/get-started-with-push-js
[Visual Studio con Xamarin]: /visualstudio/install/install-visual-studio
[Visual Studio para Mac]: https://www.visualstudio.com/vs/visual-studio-mac/
[Azure Portal]: https://portal.azure.com/
[wns object]: https://go.microsoft.com/fwlink/p/?LinkId=260591
[Notification Hubs Guidance]: /previous-versions/azure/azure-services/jj927170(v=azure.100)
[Notification Hubs How-To for Android]: /previous-versions/azure/dn282661(v=azure.100)
[Use Notification Hubs to push notifications to users]: notification-hubs-aspnet-backend-ios-apple-apns-notification.md
[Use Notification Hubs to send breaking news]: notification-hubs-windows-notification-dotnet-push-xplat-segmented-wns.md
[GitHub]: https://github.com/Azure/azure-notificationhubs-android
