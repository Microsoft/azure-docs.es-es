---
title: Envío de notificaciones a aplicaciones de Plataforma universal de Windows mediante Azure Notification Hubs | Microsoft Docs
description: Aprenda a usar Azure Notification Hubs para enviar notificaciones push a una aplicación de Plataforma universal de Windows.
services: notification-hubs
documentationcenter: windows
author: sethmanheim
manager: femila
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-windows
ms.devlang: dotnet
ms.topic: tutorial
ms.custom: mvc
ms.date: 12/05/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 12/04/2019
ms.openlocfilehash: 232bd96373f22957a88c94585f53237e05d5e5cf
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87076702"
---
# <a name="tutorial-send-notifications-to-universal-windows-platform-apps-using-azure-notification-hubs"></a>Tutorial: Envío de notificaciones a aplicaciones de Plataforma universal de Windows mediante Azure Notification Hubs

[!INCLUDE [notification-hubs-selector-get-started](../../includes/notification-hubs-selector-get-started.md)]

En este tutorial se crea un Centro de notificaciones para enviar notificaciones push a una aplicación de Plataforma universal de Windows (UWP). Cree una aplicación vacía de la Tienda Windows que reciba notificaciones push a través de los Servicios de notificaciones de inserción de Windows (WNS). Luego, use el Centro de notificaciones para difundir notificaciones push a todos los dispositivos que ejecuten la aplicación.

> [!NOTE]
> El código completo de este tutorial se encuentra en [GitHub](https://github.com/Azure/azure-notificationhubs-dotnet/tree/master/Samples/UwpSample).

Siga estos pasos:

> [!div class="checklist"]
> * Crear una aplicación en Tienda Windows
> * Creación de un centro de notificaciones
> * Crear una aplicación de Windows de ejemplo
> * Enviar notificaciones de prueba

## <a name="prerequisites"></a>Prerrequisitos

- **Suscripción de Azure**. Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.
- Microsoft Visual Studio 2017 o posterior. En el ejemplo de este tutorial se usa [Visual Studio 2019](https://www.visualstudio.com/products).
- [Herramientas de desarrollo de aplicaciones de la plataforma universal de Windows instaladas](/windows/uwp/get-started/get-set-up)
- Una cuenta de la Tienda Windows activa
- Confirme que el valor **Obtener notificaciones de aplicaciones y otros remitentes** está habilitado. 
    - Abra la ventana **Configuración** en el equipo.
    - Seleccione el icono **Sistema**.
    - Seleccione **Notificaciones y acciones** en el menú izquierdo. 
    - Confirme que el valor **Obtener notificaciones de aplicaciones y otros remitentes**  está habilitado. Si aún no está habilitado, habilítelo.

La realización de este tutorial es un requisito previo para todos los demás tutoriales de Notification Hubs para las aplicaciones de la plataforma universal de Windows.

## <a name="create-an-app-in-windows-store"></a>Crear una aplicación en Tienda Windows

Para enviar notificaciones push a las aplicaciones de la plataforma universal de Windows, asocie su aplicación a la Tienda Windows. A continuación, configure su centro de notificaciones para que se integre con los Servicios de notificaciones de inserción de Windows.

1. Vaya al [Centro de desarrollo de Windows](https://partner.microsoft.com/dashboard/windows/first-run-experience), inicie sesión en su cuenta Microsoft y seleccione **Create a new app** (Crear aplicación).

    ![Botón de aplicación nueva](./media/notification-hubs-windows-store-dotnet-get-started/windows-store-new-app-button.png)
2. Escriba el nombre de la aplicación y seleccione **Reserve product name** (Reservar nombre de producto). Así se crea un nuevo registro de la Tienda Windows para el registro de la aplicación.

    ![Nombre de aplicación de Store](./media/notification-hubs-windows-store-dotnet-get-started/store-app-name.png)
3. Expanda **Administración de productos**, seleccione **WNS/MPNS** y, a continuación, seleccione **Sitio de Servicios Live**. Iniciar sesión en su cuenta de Microsoft. La página de registro de la aplicación se abre en una nueva pestaña. Como alternativa, puede ir directamente a la página [Mis aplicaciones](https://apps.dev.microsoft.com) y seleccionar el nombre de la aplicación para llegar a esta página.

    ![Página de WNS MPNS](./media/notification-hubs-windows-store-dotnet-get-started/wns-mpns-page.png)
4. Anote la contraseña de **Secretos de aplicación** y el **Identificador de seguridad de paquete (SID)** .

    >[!WARNING]
    >El secreto de aplicación y el SID del paquete son credenciales de seguridad importantes. No los comparta con nadie ni los distribuya con su aplicación.

## <a name="create-a-notification-hub"></a>Creación de un centro de notificaciones

[!INCLUDE [notification-hubs-portal-create-new-hub](../../includes/notification-hubs-portal-create-new-hub.md)]

### <a name="configure-wns-settings-for-the-hub"></a>Configuración de los valores de WNS del centro

1. En la categoría **CONFIGURACIÓN DE NOTIFICACIONES**, seleccione **Windows (WNS)** .
2. Especifique los valores de **SID del paquete** y **Clave de seguridad** que anotó en la sección anterior.
3. Haga clic en **Guardar** en la barra de herramientas.

    ![Cuadros SID del paquete y Clave de seguridad](./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-configure-wns.png)

El centro de notificaciones ya está configurado para funcionar con WNS. Tiene la cadena de conexión para registrar la aplicación y enviar notificaciones.

## <a name="create-a-sample-windows-app"></a>Crear una aplicación de Windows de ejemplo

1. En Visual Studio, en el menú **Archivo**, seleccione **Nuevo** y, después, **Proyecto**.
2. En el cuadro de diálogo **Crear un nuevo proyecto**, realice estos pasos:

    1. En el cuadro de búsqueda de la parte superior, escriba **Windows Universal**.
    2. En los resultados de la búsqueda, seleccione **Aplicación en blanco (Windows universal)** y, después, seleccione **Siguiente**.

       ![Cuadro de diálogo Nuevo proyecto](./media/notification-hubs-windows-store-dotnet-get-started/new-project-dialog.png)

    3. En el cuadro de diálogo **Configurar el nuevo proyecto**, escriba un **Nombre de proyecto** y una **Ubicación** para los archivos del proyecto.
    4. Seleccione **Crear**.

3. Acepte los valores predeterminados de las versiones de plataforma **mínima** y **de destino** y seleccione **Aceptar**.
4. En el Explorador de soluciones, haga clic con el botón derecho en el proyecto de la aplicación para la Tienda Windows, seleccione **Publicar** y, a continuación, seleccione **Asociar aplicación con la Tienda**. Aparece el asistente **Asocie la aplicación con la Tienda Windows** .
5. En el asistente, inicie sesión con su cuenta de Microsoft.
6. Seleccione la aplicación que registró en el paso 2, **Siguiente** y **Asociar**. De esta manera, se agrega la información de registro necesaria de la Tienda Windows al manifiesto de aplicación.
7. En Visual Studio, haga clic con el botón derecho en la solución y seleccione **Administrar paquetes NuGet**. Se abre la ventana **Administrar paquetes NuGet**.
8. En el cuadro de búsqueda, escriba **WindowsAzure.Messaging.Managed**, seleccione **Instalar** y acepte las condiciones de uso.

    ![Ventana Administrar paquetes NuGet][20]

    Esta acción descarga, instala y agrega una referencia a la biblioteca de Azure Notification Hubs para Windows mediante el [paquete NuGet Microsoft.Azure.NotificationHubs](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs).
9. Abra el archivo de proyecto `App.xaml.cs` y agregue las siguientes instrucciones:

    ```csharp
    using Windows.Networking.PushNotifications;
    using Microsoft.WindowsAzure.Messaging;
    using Windows.UI.Popups;
    ```

10. En el archivo `App.xaml.cs` del proyecto, busque la clase `App` y agregue la siguiente definición de método `InitNotificationsAsync`. Reemplace `<your hub name>` por el nombre del centro de notificaciones que creó en Azure Portal y reemplace `<Your DefaultListenSharedAccessSignature connection string>` por la cadena de conexión `DefaultListenSharedAccessSignature` de la página **Directivas de acceso** del centro de notificaciones:

    ```csharp
    private async void InitNotificationsAsync()
    {
        var channel = await PushNotificationChannelManager.CreatePushNotificationChannelForApplicationAsync();

        var hub = new NotificationHub("<your hub name>", "<Your DefaultListenSharedAccessSignature connection string>");
        var result = await hub.RegisterNativeAsync(channel.Uri);

        // Displays the registration ID so you know it was successful
        if (result.RegistrationId != null)
        {
            var dialog = new MessageDialog("Registration successful: " + result.RegistrationId);
            dialog.Commands.Add(new UICommand("OK"));
            await dialog.ShowAsync();
        }
    }
    ```

    Este código recupera el URI del canal de la aplicación desde WNS y, luego, lo registra en el Centro de notificaciones.

    >[!NOTE]
    > Reemplace el marcador de posición `hub name` por el nombre del centro de notificaciones que aparece en Azure Portal. Reemplace también el marcador de posición de cadena de conexión por la cadena de conexión `DefaultListenSharedAccessSignature` que obtuvo de la página **Directivas de acceso** del centro de notificaciones en una sección anterior.

11. En la parte superior del controlador de eventos `OnLaunched` en `App.xaml.cs`, agregue la siguiente llamada al nuevo método `InitNotificationsAsync`:

    ```csharp
    InitNotificationsAsync();
    ```

    Esta acción garantiza que el URI del canal se registra en su centro de notificaciones cada vez que se inicia la aplicación.

12. Para ejecutar la aplicación, presione la tecla **F5** del teclado. Se mostrará un cuadro de diálogo que contiene la clave de registro. Para cerrar el cuadro de diálogo, haga clic en **Aceptar**.

    ![El registro se completó correctamente.](./media/notification-hubs-windows-store-dotnet-get-started/registration-successful.png)

La carpeta ahora ya está lista para recibir notificaciones.

## <a name="send-test-notifications"></a>Enviar notificaciones de prueba

Puede probar rápidamente la recepción de notificaciones en la aplicación si envía alguna desde [Azure Portal](https://portal.azure.com/).

1. En Azure Portal, cambie a la pestaña Información general y seleccione **Envío de prueba** en la barra de herramientas.

    ![Botón Envío de prueba](./media/notification-hubs-windows-store-dotnet-get-started/test-send-button.png)
2. En la ventana **Envío de prueba**, lleve a cabo las siguientes acciones:
    1. En **Plataformas**, seleccione **Windows**.
    2. En **Tipo de notificación**, seleccione **Notificación del sistema**.
    3. Seleccione **Enviar**.

        ![Panel Envío de prueba](./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-test-send-wns.png)
3. Verá el resultado de la operación de envío en la lista **Resultado** de la parte inferior de la ventana. También verá un mensaje de alerta.

    ![Resultado de la operación de envío](./media/notification-hubs-windows-store-dotnet-get-started/result-of-send.png)
4. Verá el mensaje de notificación: **Mensaje de prueba** en el escritorio.

    ![Mensaje de notificación](./media/notification-hubs-windows-store-dotnet-get-started/test-notification-message.png)

## <a name="next-steps"></a>Pasos siguientes
Ha enviado notificaciones de difusión a todos los dispositivos con Windows mediante el portal o una aplicación de consola. Para aprender a enviar notificaciones push a dispositivos específicos, pase al siguiente tutorial:

> [!div class="nextstepaction"]
>[Envío de notificaciones push a dispositivos específicos](
notification-hubs-windows-notification-dotnet-push-xplat-segmented-wns.md)

<!-- Images. -->
[13]: ./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-create-console-app.png
[14]: ./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-windows-toast.png
[19]: ./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-windows-reg.png
[20]: ./media/notification-hubs-windows-store-dotnet-get-started/notification-hub-windows-universal-app-install-package.png

<!-- URLs. -->
[Use Notification Hubs to push notifications to users]: notification-hubs-aspnet-backend-windows-dotnet-wns-notification.md
[Use Notification Hubs to send breaking news]: notification-hubs-windows-notification-dotnet-push-xplat-segmented-wns.md
[toast catalog]: /previous-versions/windows/apps/hh761494(v=win.10)
[tile catalog]: /previous-versions/windows/apps/hh761491(v=win.10)
[badge overview]: /previous-versions/windows/apps/hh779719(v=win.10)
