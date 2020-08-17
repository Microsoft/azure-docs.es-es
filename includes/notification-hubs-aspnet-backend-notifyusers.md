---
title: archivo de inclusión
description: archivo de inclusión que contiene código para crear un proyecto de back-end de ASP .NET WebAPI.
services: notification-hubs
author: spelluru
ms.service: notification-hubs
ms.topic: include
ms.date: 09/11/2019
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: 3db9811322d27ab287fa568eeeffcb5f4d57bdf7
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86530186"
---
## <a name="create-the-webapi-project"></a>Creación del proyecto de API web

En las secciones siguientes se describe la creación de un nuevo back-end de ASP.NET WebAPI. Este proceso tiene tres objetivos principales:

- **Autenticación de clientes**: se agrega un controlador de mensajes para autenticar las solicitudes de cliente y asociar el usuario a la solicitud.
- **Registro para recibir notificaciones mediante el back-end de WebAPI**: se agrega un controlador para administrar los registros nuevos de un dispositivo cliente para que reciba notificaciones. El nombre de usuario autenticado se agrega automáticamente al registro como [etiqueta](../articles/notification-hubs/notification-hubs-tags-segment-push-message.md).
- **Envío de notificaciones a los clientes**: se agrega un controlador para que los usuarios puedan desencadenar una inserción segura en los dispositivos y clientes asociados con la etiqueta.

Realice las siguientes acciones para crear el nuevo back-end de ASP.NET WebAPI:

> [!IMPORTANT]
> Si usa Visual Studio 2015 o una versión anterior, asegúrese de que ha instalado la versión más reciente del Administrador de paquetes NuGet para Visual Studio antes de comenzar este tutorial.
>
>Para comprobarlo, inicie Visual Studio. En el menú **Herramientas**, seleccione **Extensiones y actualizaciones**. Busque el **Administrador de paquetes NuGet** correspondiente a su versión de Visual Studio y asegúrese de que tiene la versión más reciente. Si no es la más reciente, se debe desinstalar y volver a instalar el Administrador de paquetes NuGet.

![Captura de pantalla del cuadro de diálogo Extensiones y actualizaciones con el paquete NuGet de administración para Visual Studio resaltado.][B4]

> [!NOTE]
> Asegúrese de que ha instalado el [SDK de Azure](https://azure.microsoft.com/downloads/) para Visual Studio para la implementación de sitios web.

1. Inicie Visual Studio o Visual Studio Express.

2. Seleccione el **Explorador de servidores** e inicie sesión en su cuenta de Azure. Para crear los recursos del sitio web en su cuenta, tiene que iniciar sesión.

3. En Visual Studio, haga clic con el botón derecho en la solución de Visual Studio, seleccione **Agregar** y haga clic en **Nuevo proyecto**.
4. Expanda **Visual C#** , seleccione **Web** y haga clic en **Aplicación web ASP.NET**.

5. En el cuadro **Nombre**, escriba **AppBackend** y seleccione **Aceptar**.

    ![Ventana Nuevo proyecto][B1]

6. En la ventana **New ASP.NET Project** (Nuevo proyecto de ASP.NET), seleccione la casilla **API web** y **Aceptar**.

    ![Ventana New ASP.NET Project (Nuevo proyecto de ASP.NET)][B2]

7. En la ventana **Configurar aplicación web de Microsoft Azure**, seleccione una suscripción y, en la lista**Plan de App Service**, realice una de las siguientes acciones:

    * Seleccione un plan de App Service creado.
    * Seleccione **Crear un nuevo plan de App Service** para crear uno.

   No es necesario una base de datos para este tutorial. Una vez seleccionado el plan de App Service, seleccione **Aceptar** para crear el proyecto.

    ![Ventana Configurar aplicación web de Microsoft Azure][B5]

    Si no ve esta página para configurar el plan de App Service, continúe con el tutorial. Se puede configurar al publicar la aplicación más adelante. 

## <a name="authenticate-clients-to-the-webapi-backend"></a>Autenticar clientes en el back-end de WebAPI

En esta sección se crea una clase de controlador de mensajes llamada **AuthenticationTestHandler** para el nuevo back-end. Esta clase deriva de [DelegatingHandler](https://msdn.microsoft.com/library/system.net.http.delegatinghandler.aspx) y se agrega como controlador de mensajes para que procese todas las solicitudes que lleguen al back-end.

1. En el Explorador de soluciones, haga clic con el botón derecho en el proyecto **AppBackend**, y seleccione **Agregar** y **Clase**.
2. Asigne a la nueva clase el nombre **AuthenticationTestHandler.cs** y seleccione **Agregar** para generar la clase. Para simplificar, esta clase autentica a los usuarios con *Autenticación básica*. La aplicación puede utilizar cualquier esquema de autenticación.
3. En AuthenticationTestHandler.cs, agregue las siguientes instrucciones `using` :

    ```csharp
    using System.Net.Http;
    using System.Threading;
    using System.Security.Principal;
    using System.Net;
    using System.Text;
    using System.Threading.Tasks;
    ```

4. En AuthenticationTestHandler.cs, reemplace la definición de clase `AuthenticationTestHandler` con el código siguiente:

    Este controlador autoriza la solicitud cuando se cumplen las tres condiciones siguientes:

   * La solicitud incluya un encabezado *Autorización*.
   * La solicitud utiliza la autenticación *básica* .
   * La cadena de nombre de usuario y la cadena de contraseña son la misma cadena.

   En caso contrario, la solicitud se rechaza. Esta autenticación no es un enfoque de autorización y autenticación real. Es solo un ejemplo sencillo para este tutorial.

   Si `AuthenticationTestHandler` autentica y autoriza el mensaje de solicitud, el usuario de autenticación básica se adjuntará a la solicitud actual en [HttpContext](https://msdn.microsoft.com/library/system.web.httpcontext.current.aspx). Otro controlador (RegisterController) usará después la información de usuario de HttpContext para agregar una [etiqueta](https://msdn.microsoft.com/library/azure/dn530749.aspx) a la solicitud de registro de notificación.

    ```csharp
    public class AuthenticationTestHandler : DelegatingHandler
    {
        protected override Task<HttpResponseMessage> SendAsync(
        HttpRequestMessage request, CancellationToken cancellationToken)
        {
            var authorizationHeader = request.Headers.GetValues("Authorization").First();

            if (authorizationHeader != null && authorizationHeader
                .StartsWith("Basic ", StringComparison.InvariantCultureIgnoreCase))
            {
                string authorizationUserAndPwdBase64 =
                    authorizationHeader.Substring("Basic ".Length);
                string authorizationUserAndPwd = Encoding.Default
                    .GetString(Convert.FromBase64String(authorizationUserAndPwdBase64));
                string user = authorizationUserAndPwd.Split(':')[0];
                string password = authorizationUserAndPwd.Split(':')[1];

                if (VerifyUserAndPwd(user, password))
                {
                    // Attach the new principal object to the current HttpContext object
                    HttpContext.Current.User =
                        new GenericPrincipal(new GenericIdentity(user), new string[0]);
                    System.Threading.Thread.CurrentPrincipal =
                        System.Web.HttpContext.Current.User;
                }
                else return Unauthorized();
            }
            else return Unauthorized();

            return base.SendAsync(request, cancellationToken);
        }

        private bool VerifyUserAndPwd(string user, string password)
        {
            // This is not a real authentication scheme.
            return user == password;
        }

        private Task<HttpResponseMessage> Unauthorized()
        {
            var response = new HttpResponseMessage(HttpStatusCode.Forbidden);
            var tsc = new TaskCompletionSource<HttpResponseMessage>();
            tsc.SetResult(response);
            return tsc.Task;
        }
    }
    ```

    > [!NOTE]
    > Nota de seguridad: la clase `AuthenticationTestHandler` no proporciona una autenticación verdadera. Se utiliza únicamente para simular una autenticación básica y no es segura. Debe implementar un mecanismo de autenticación seguro en las aplicaciones y servicios de producción.
5. Para registrar el controlador de mensajes, agregue el siguiente código al final del método `Register` de la clase **App_Start/WebApiConfig.cs**:

    ```csharp
    config.MessageHandlers.Add(new AuthenticationTestHandler());
    ```
6. Guarde los cambios.

## <a name="register-for-notifications-by-using-the-webapi-backend"></a>Registrar notificaciones mediante el back-end de WebAPI

En esta sección, agregaremos un nuevo controlador al back-end de WebAPI para administrar las solicitudes de registro de un usuario y un dispositivo para que reciban notificaciones mediante la biblioteca cliente de los centros de notificaciones. El controlador agrega una etiqueta de usuario al usuario que el `AuthenticationTestHandler`autenticó y adjuntó a HttpContext. La etiqueta tiene el formato de cadena, `"username:<actual username>"`.

1. En el Explorador de soluciones, haga clic con el botón derecho en el proyecto **AppBackend** y seleccione **Administrar paquetes NuGet**.

2. En el panel izquierdo, seleccione **En línea** y en el cuadro **Buscar**, escriba **Microsoft.Azure.NotificationHubs**.

3. En la lista de resultados, seleccione **Microsoft Azure Notification Hubs** e **Instalar**. Complete la instalación y cierre la ventana del Administrador de paquetes NuGet.

    Esta acción agrega una referencia al SDK de Azure Notification Hubs mediante el [paquete NuGet Microsoft.Azure.NotificationHubs](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/).

4. Cree un archivo de clase que represente la conexión con el centro de notificaciones de envío. En el Explorador de soluciones, haga clic con el botón derecho en la carpeta **Modelos**, seleccione **Agregar** y **Clase**. Asigne el nombre a la nueva clase **Notifications.cs** y seleccione **Agregar** para generar la clase.

    ![Ventana Agregar nuevo elemento][B6]

5. En Notifications.cs, agregue la siguiente instrucción `using` en la parte superior del archivo:

    ```csharp
    using Microsoft.Azure.NotificationHubs;
    ```

6. Reemplace la definición de clase `Notifications` con el código siguiente y los dos marcadores de posición con la cadena de conexión (de acceso total) del centro de notificaciones y el nombre del centro (disponible en el [Portal de Azure](https://portal.azure.com)):

    ```csharp
    public class Notifications
    {
        public static Notifications Instance = new Notifications();

        public NotificationHubClient Hub { get; set; }

        private Notifications() {
            Hub = NotificationHubClient.CreateClientFromConnectionString("<your hub's DefaultFullSharedAccessSignature>",
                                                                            "<hub name>");
        }
    }
    ```
    > [!IMPORTANT]
    > Escriba el **nombre** y elemento **DefaultFullSharedAccessSignature** del centro antes de continuar. 
    
7. A continuación, cree un controlador denominado **RegisterController**. En el Explorador de soluciones, haga clic con el botón derecho en la carpeta **Controladores**, y seleccione **Agregar** y **Controlador**.

8. Seleccione **Controlador de Web API 2 - en blanco** y **Agregar**.

    ![Ventana Agregar scaffold][B7]

9. En el cuadro **Nombre del controlador**, escriba **RegisterController** para denominar la nueva clase y seleccione **Agregar**.

    ![Ventana Agregar controlador][B8]

10. En RegisterController.cs, agregue las siguientes instrucciones `using` :

    ```csharp
    using Microsoft.Azure.NotificationHubs;
    using Microsoft.Azure.NotificationHubs.Messaging;
    using AppBackend.Models;
    using System.Threading.Tasks;
    using System.Web;
    ```
11. Agregue el siguiente código a la definición de clase `RegisterController` . En este código, agregamos una etiqueta de usuario para el usuario asociado a HttpContext. El usuario se autenticó y se asoció a HttpContext mediante un filtro de mensaje que agregó, `AuthenticationTestHandler`. También puede realizar comprobaciones opcionales para comprobar que el usuario puede registrarse para las etiquetas solicitadas.

    ```csharp
    private NotificationHubClient hub;

    public RegisterController()
    {
        hub = Notifications.Instance.Hub;
    }

    public class DeviceRegistration
    {
        public string Platform { get; set; }
        public string Handle { get; set; }
        public string[] Tags { get; set; }
    }

    // POST api/register
    // This creates a registration id
    public async Task<string> Post(string handle = null)
    {
        string newRegistrationId = null;

        // make sure there are no existing registrations for this push handle (used for iOS and Android)
        if (handle != null)
        {
            var registrations = await hub.GetRegistrationsByChannelAsync(handle, 100);

            foreach (RegistrationDescription registration in registrations)
            {
                if (newRegistrationId == null)
                {
                    newRegistrationId = registration.RegistrationId;
                }
                else
                {
                    await hub.DeleteRegistrationAsync(registration);
                }
            }
        }

        if (newRegistrationId == null) 
            newRegistrationId = await hub.CreateRegistrationIdAsync();

        return newRegistrationId;
    }

    // PUT api/register/5
    // This creates or updates a registration (with provided channelURI) at the specified id
    public async Task<HttpResponseMessage> Put(string id, DeviceRegistration deviceUpdate)
    {
        RegistrationDescription registration = null;
        switch (deviceUpdate.Platform)
        {
            case "mpns":
                registration = new MpnsRegistrationDescription(deviceUpdate.Handle);
                break;
            case "wns":
                registration = new WindowsRegistrationDescription(deviceUpdate.Handle);
                break;
            case "apns":
                registration = new AppleRegistrationDescription(deviceUpdate.Handle);
                break;
            case "fcm":
                registration = new FcmRegistrationDescription(deviceUpdate.Handle);
                break;
            default:
                throw new HttpResponseException(HttpStatusCode.BadRequest);
        }

        registration.RegistrationId = id;
        var username = HttpContext.Current.User.Identity.Name;

        // add check if user is allowed to add these tags
        registration.Tags = new HashSet<string>(deviceUpdate.Tags);
        registration.Tags.Add("username:" + username);

        try
        {
            await hub.CreateOrUpdateRegistrationAsync(registration);
        }
        catch (MessagingException e)
        {
            ReturnGoneIfHubResponseIsGone(e);
        }

        return Request.CreateResponse(HttpStatusCode.OK);
    }

    // DELETE api/register/5
    public async Task<HttpResponseMessage> Delete(string id)
    {
        await hub.DeleteRegistrationAsync(id);
        return Request.CreateResponse(HttpStatusCode.OK);
    }

    private static void ReturnGoneIfHubResponseIsGone(MessagingException e)
    {
        var webex = e.InnerException as WebException;
        if (webex.Status == WebExceptionStatus.ProtocolError)
        {
            var response = (HttpWebResponse)webex.Response;
            if (response.StatusCode == HttpStatusCode.Gone)
                throw new HttpRequestException(HttpStatusCode.Gone.ToString());
        }
    }
    ```
12. Guarde los cambios.

## <a name="send-notifications-from-the-webapi-backend"></a>Enviar notificaciones desde el back-end de WebAPI

En esta sección se agrega un nuevo controlador que expone una manera de enviar notificaciones para los dispositivos cliente. La notificación se basa en la etiqueta de nombre de usuario que usa la biblioteca .NET de Microsoft Azure Notification Hubs en el back-end de ASP.NET WebAPI.

1. Cree otro controlador denominado **NotificationsController** del mismo modo que creó **RegisterController** en la sección anterior.

2. En NotificationsController.cs, agregue las siguientes instrucciones `using` :

    ```csharp
    using AppBackend.Models;
    using System.Threading.Tasks;
    using System.Web;
    ```
3. Agregue el método siguiente a la clase **NotificationsController**:

    Este código envía un tipo de notificación basado en el parámetro `pns` del servicio Sistema de notificación de plataforma (PNS). El valor de `to_tag` se usa para definir la etiqueta *username* en el mensaje. Esta etiqueta debe coincidir con una etiqueta de nombre de usuario de un registro de un centro de notificaciones activo. El mensaje de notificación se extrae del cuerpo de la solicitud POST y se formatea para el PNS de destino.

    Dependiendo del PNS que usen los dispositivos compatibles para recibir notificaciones, se admiten distintos formatos. Por ejemplo, en dispositivos Windows, puede usar una [notificación del sistema con WNS](https://msdn.microsoft.com/library/windows/apps/br230849.aspx) que no sea compatible directamente con otro PNS. En este caso, el back-end debe dar un formato a la notificación que sea compatible con el PNS de los dispositivos que prevea admitir. Posteriormente, use la API de envío adecuada en la [clase NotificationHubClient](https://msdn.microsoft.com/library/azure/microsoft.azure.notificationhubs.notificationhubclient_methods.aspx).

    ```csharp
    public async Task<HttpResponseMessage> Post(string pns, [FromBody]string message, string to_tag)
    {
        var user = HttpContext.Current.User.Identity.Name;
        string[] userTag = new string[2];
        userTag[0] = "username:" + to_tag;
        userTag[1] = "from:" + user;

        Microsoft.Azure.NotificationHubs.NotificationOutcome outcome = null;
        HttpStatusCode ret = HttpStatusCode.InternalServerError;

        switch (pns.ToLower())
        {
            case "wns":
                // Windows 8.1 / Windows Phone 8.1
                var toast = @"<toast><visual><binding template=""ToastText01""><text id=""1"">" + 
                            "From " + user + ": " + message + "</text></binding></visual></toast>";
                outcome = await Notifications.Instance.Hub.SendWindowsNativeNotificationAsync(toast, userTag);
                break;
            case "apns":
                // iOS
                var alert = "{\"aps\":{\"alert\":\"" + "From " + user + ": " + message + "\"}}";
                outcome = await Notifications.Instance.Hub.SendAppleNativeNotificationAsync(alert, userTag);
                break;
            case "fcm":
                // Android
                var notif = "{ \"data\" : {\"message\":\"" + "From " + user + ": " + message + "\"}}";
                outcome = await Notifications.Instance.Hub.SendFcmNativeNotificationAsync(notif, userTag);
                break;
        }

        if (outcome != null)
        {
            if (!((outcome.State == Microsoft.Azure.NotificationHubs.NotificationOutcomeState.Abandoned) ||
                (outcome.State == Microsoft.Azure.NotificationHubs.NotificationOutcomeState.Unknown)))
            {
                ret = HttpStatusCode.OK;
            }
        }

        return Request.CreateResponse(ret);
    }
    ```
4. Para ejecutar la aplicación y garantizar que hasta ahora todo es correcto, seleccione la tecla **F5**. La aplicación abre un explorador web y se muestra en la página de inicio de ASP.NET.

## <a name="publish-the-new-webapi-backend"></a>Publicación del nuevo back-end de WebAPI

A continuación implementaremos la aplicación en un sitio web de Azure para que todos los dispositivos puedan acceder a ella.

1. Haga clic con el botón derecho en el proyecto **AppBackend** y seleccione **Publicar**.

2. Seleccione **Microsoft Azure App Service** como destino de publicación y, luego, \*\*Publicar. La ventana Crear servicio de aplicaciones se abre. Aquí puede crear todos los recursos de Azure necesarios para ejecutar la aplicación web ASP.NET en Azure.

    ![Icono de Microsoft Azure App Service][B15]

3. En la ventana **Crear servicio de aplicaciones**, seleccione la cuenta de Azure. Seleccione **Cambiar tipo** > **Aplicación web**. Mantenga el **Nombre de aplicación web** predeterminado y seleccione los valores de **Suscripción**, **Grupo de recursos** y **Plan de App Service**.

4. Seleccione **Crear**.

5. Anote la propiedad **Dirección URL del sitio** de la sección **Resumen**. Esta dirección URL es el *punto de conexión de back-end* que se utilizará más adelante en el tutorial.

6. Seleccione **Publicar**.

Una vez completado el asistente, este publica la aplicación web ASP.NET en Azure y la abre en el explorador predeterminado.  La aplicación se puede ver en Azure App Services.

La dirección URL usa el nombre de la aplicación web que especificó anteriormente, con el formato http://<app_name>.azurewebsites.net.

[B1]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push1.png
[B2]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push2.png
[B3]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push3.png
[B4]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push4.png
[B5]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push5.png
[B6]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push6.png
[B7]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push7.png
[B8]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push8.png
[B14]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-secure-push14.png
[B15]: ./media/notification-hubs-aspnet-backend-notifyusers/publish-to-app-service.png
[B16]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-notify-users16.PNG
[B18]: ./media/notification-hubs-aspnet-backend-notifyusers/notification-hubs-notify-users18.PNG
