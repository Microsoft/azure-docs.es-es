---
title: Envío de notificaciones push a aplicaciones Windows Phone mediante Azure Notification Hubs | Microsoft Docs
description: En este tutorial aprenderá a usar Azure Notification Hubs para enviar notificaciones push a aplicaciones Windows Phone 8 o Windows Phone 8.1 Silverlight.
services: notification-hubs
documentationcenter: windows
keywords: notificación push, notificación push, notificación push de windows phone
author: sethmanheim
manager: femila
editor: jwargo
ms.assetid: d872d8dc-4658-4d65-9e71-fa8e34fae96e
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-windows-phone
ms.devlang: dotnet
ms.topic: tutorial
ms.custom: mvc, devx-track-csharp
ms.date: 01/04/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 01/04/2019
ms.openlocfilehash: e91d250b8cc9b80f2c97910c7fa972af32fa9104
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88998346"
---
# <a name="tutorial-send-push-notifications-to-windows-phone-apps-using-notification-hubs"></a>Tutorial: Envío de notificaciones push a aplicaciones de Windows Phone Notification Hubs

[!INCLUDE [notification-hubs-selector-get-started](../../includes/notification-hubs-selector-get-started.md)]

En este tutorial se muestra cómo usar Azure Notification Hubs para enviar notificaciones push a aplicaciones Windows Phone 8 o Windows Phone 8.1 Silverlight. Si va a desarrollar para Windows Phone 8.1 (no Silverlight), consulte la versión [Windows Universal](notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md) de este tutorial.

En este tutorial puede crear una aplicación de Windows Phone 8 vacía que recibe notificaciones push mediante el Servicio de notificaciones de inserción de Microsoft (MPNS). Después de crear la aplicación use el Centro de notificaciones para difundir notificaciones push a todos los dispositivos que la ejecutan.

> [!NOTE]
> El SDK de Windows Phone para Notification Hubs no admite el uso de Servicios de notificaciones de inserción de Windows (WNS) con aplicaciones de Windows Phone 8.1 Silverlight. Para usar WNS (en lugar de MPNS) con aplicaciones de Windows Phone 8.1 Silverlight, siga el [tutorial Notification Hubs: Windows Phone Silverlight], que usa API de REST.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Creación de un centro de notificaciones
> * Crear una aplicación Windows Phone
> * Realizar un envío de prueba de una notificación

## <a name="prerequisites"></a>Prerrequisitos

* **Suscripción de Azure**. Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.
* [Visual Studio 2015 Express con componentes de desarrollo para dispositivos móviles](https://www.visualstudio.com/vs/older-downloads/)

La realización de este tutorial es un requisito previo para todos los demás tutoriales de Notification Hubs para aplicaciones de Windows Phone 8.

## <a name="create-your-notification-hub"></a>Creación de su centro de notificaciones

[!INCLUDE [notification-hubs-portal-create-new-hub](../../includes/notification-hubs-portal-create-new-hub.md)]

### <a name="configure-windows-phone-mpns-settings"></a>Configuración de los valores de Windows Phone (MPNS)

1. Seleccione **Windows Phone (MPNS)** en **NOTIFICATION SETTINGS** (CONFIGURACIÓN DE NOTIFICACIONES).
2. Seleccione **Enable authentication push** (Habilitar inserción de autenticación).
3. Seleccione **Guardar** en la barra de herramientas.

    ![Azure Portal: Habilitación de notificaciones push sin autenticar](./media/notification-hubs-windows-phone-get-started/azure-portal-unauth.png)

    El concentrador se crea y configura ahora para enviar una notificación sin autenticar para Windows Phone.

    > [!NOTE]
    > Este tutorial usa MPNS en modo sin autenticar. El modo sin autenticar de MPNS viene con restricciones sobre las notificaciones que puede enviar a cada canal. Notification Hubs admite el [modo autenticado de MPNS](/previous-versions/windows/apps/ff941099(v=vs.105)) al permitir que cargue su certificado.

## <a name="create-a-windows-phone-application"></a>Crear una aplicación Windows Phone

En esta sección se creará una aplicación de Windows Phone que se registra automáticamente en el Centro de notificaciones.

1. En Visual Studio, cree una aplicación nueva de Windows Phone 8.

    ![Visual Studio: Nuevo proyecto: Aplicación de Windows Phone][13]

    En Visual Studio 2013 Update 2 o versiones posteriores, cree en su lugar una aplicación Silverlight de Windows Phone.

    ![Visual Studio, nuevo proyecto, aplicación vacía, Windows Phone Silverlight][11]
2. En Visual Studio, haga clic con el botón derecho en la solución y, a continuación, haga clic en **Administrar paquetes NuGet**.
3. Busque `WindowsAzure.Messaging.Managed` , haga clic en **Instalar**y acepte los términos de uso.

    ![Visual Studio, Administrador de paquetes NuGet][20]
4. Abra el archivo App.xaml.cs y agregue las siguientes instrucciones `using` :

    ```csharp
    using Microsoft.Phone.Notification;
    using Microsoft.WindowsAzure.Messaging;
    ```

5. Agregue el código siguiente en la parte superior del método `Application_Launching` de `App.xaml.cs`:

    ```csharp
    private void Application_Launching(object sender, LaunchingEventArgs e)
    {

        var channel = HttpNotificationChannel.Find("MyPushChannel");
        if (channel == null)
        {
            channel = new HttpNotificationChannel("MyPushChannel");
            channel.Open();
            channel.BindToShellToast();
        }

        channel.ChannelUriUpdated += new EventHandler<NotificationChannelUriEventArgs>(async (o, args) =>
        {
            var hub = new NotificationHub("<hub name>", "<connection string>");
            var result = await hub.RegisterNativeAsync(args.ChannelUri.ToString());

            System.Windows.Deployment.Current.Dispatcher.BeginInvoke(() =>
            {
                MessageBox.Show("Registration :" + result.RegistrationId, "Registered", MessageBoxButton.OK);
            });
        });
    }
    ```

   > [!NOTE]
   > El valor `MyPushChannel` es un índice que se utiliza para buscar un canal existente en la colección [HttpNotificationChannel](/previous-versions/ff402781(v=vs.110)). Si no hay uno ya allí, cree una nueva entrada con ese nombre.

    Inserte el nombre del centro y la cadena de conexión denominada `DefaultListenSharedAccessSignature` que anotó en la sección anterior.
    Este código recupera el URI del canal de la aplicación desde MPNS y, luego, lo registra con su centro de notificaciones. Garantiza también que el URI del canal se registre en su centro de notificaciones cada vez que se inicia la aplicación.

   > [!NOTE]
   > Este tutorial envía una notificación del sistema al dispositivo. Cuando envía una notificación de icono, debe llamar al método `BindToShellTile` en el canal. Para admitir notificaciones del sistema y notificaciones de icono, llame tanto a `BindToShellTile` como a `BindToShellToast`.

6. En el Explorador de soluciones, expanda **Propiedades**, abra el archivo `WMAppManifest.xml`, haga clic en la pestaña **Funcionalidades** y asegúrese de que la funcionalidad **ID_CAP_PUSH_NOTIFICATION** esté marcada. La aplicación ya puede recibir notificaciones push.

    ![Visual Studio, funcionalidades de la aplicación de Windows Phone][14]
7. Presione la tecla `F5` para ejecutar la aplicación. Se muestra un mensaje de registro en la aplicación.
8. Cierre la aplicación o cambie a la página principal.

   > [!NOTE]
   > Para recibir una notificación push, la aplicación no se debe estar ejecutando en primer plano.

## <a name="test-send-a-notification"></a>Realizar un envío de prueba de una notificación

1. En Azure Portal, cambie a la pestaña información general.
2. Seleccione **Envío de prueba**.

    ![Botón Envío de prueba](./media/notification-hubs-windows-phone-get-started/test-send-button.png)
3. En la ventana **Envío de prueba**, siga estos pasos:

    1. En **Plataformas**, seleccione **Windows Phone**.
    2. En **Tipo de notificación**, seleccione **Notificación del sistema**.
    3. Seleccione **Enviar**.
    4. Vea el **resultado** en la lista en la parte inferior de la ventana.

        ![Ventana Envío de prueba](./media/notification-hubs-windows-phone-get-started/test-send-window.png)
4. En el emulador de Windows Phone o en el dispositivo Windows Phone, confirme que ve el mensaje de notificación.

    ![Notificación en el dispositivo Windows Phone](./media/notification-hubs-windows-phone-get-started/notification-on-windows-phone.png)

## <a name="next-steps"></a>Pasos siguientes

En este sencillo ejemplo, ha difundido notificaciones push a todos los dispositivos Windows Phone 8. Para aprender a enviar notificaciones push a dispositivos específicos, pase al siguiente tutorial:

> [!div class="nextstepaction"]
>[Envío de notificaciones push a dispositivos específicos](notification-hubs-windows-phone-push-xplat-segmented-mpns-notification.md)

<!-- Images. -->
[6]: ./media/notification-hubs-windows-phone-get-started/notification-hub-create-console-app.png
[7]: ./media/notification-hubs-windows-phone-get-started/notification-hub-create-from-portal.png
[8]: ./media/notification-hubs-windows-phone-get-started/notification-hub-create-from-portal2.png
[9]: ./media/notification-hubs-windows-phone-get-started/notification-hub-select-from-portal.png
[10]: ./media/notification-hubs-windows-phone-get-started/notification-hub-select-from-portal2.png
[11]: ./media/notification-hubs-windows-phone-get-started/notification-hub-create-wp-silverlight-app.png
[12]: ./media/notification-hubs-windows-phone-get-started/notification-hub-connection-strings.png
[13]: ./media/notification-hubs-windows-phone-get-started/notification-hub-create-wp-app.png
[14]: ./media/notification-hubs-windows-phone-get-started/mobile-app-enable-push-wp8.png
[15]: ./media/notification-hubs-windows-phone-get-started/notification-hub-pushauth.png
[20]: ./media/notification-hubs-windows-phone-get-started/notification-hub-windows-universal-app-install-package.png
[213]: ./media/notification-hubs-windows-phone-get-started/notification-hub-create-console-app.png

<!-- URLs. -->
[Notification Hubs Guidance]: /previous-versions/azure/azure-services/jj927170(v=azure.100)
[MPNS authenticated mode]: /previous-versions/windows/apps/ff941099(v=vs.105)
[Use Notification Hubs to push notifications to users]: notification-hubs-aspnet-backend-windows-dotnet-wns-notification.md
[Use Notification Hubs to send breaking news]: notification-hubs-windows-phone-push-xplat-segmented-mpns-notification.md
[toast catalog]: /previous-versions/windows/apps/jj662938(v=vs.105)
[tile catalog]: /previous-versions/windows/apps/hh202948(v=vs.105)
[tutorial Notification Hubs: Windows Phone Silverlight]: https://github.com/Azure/azure-notificationhubs-samples/tree/master/PushToSafari
