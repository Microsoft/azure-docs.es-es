---
title: Adición de inicio de sesión en la Plataforma de identidad de Microsoft a una aplicación web de ASP.NET
titleSuffix: Microsoft identity platform
description: Implementación del inicio de sesión de Microsoft en una solución ASP.NET mediante una aplicación basada en un explorador web tradicional y el estándar OpenID Connect
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.workload: identity
ms.date: 08/28/2019
ms.author: jmprieur
ms.custom: devx-track-csharp, aaddev, identityplatformtop40
ms.openlocfilehash: 740d62136393cf0c9cf31d367735bffed1c05276
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88165590"
---
# <a name="add-sign-in-to-microsoft-to-an-aspnet-web-app"></a>Adición del inicio de sesión en Microsoft a una aplicación web ASP.NET

Esta guía muestra cómo implementar el inicio de sesión en Microsoft a través de una solución ASP.NET MVC mediante una aplicación basada en explorador web tradicional y OpenID Connect.

Cuando haya completado esta guía, la aplicación podrá aceptar inicios de sesión de cuentas personales de outlook.com y live.com. Además, tanto las cuentas profesionales como las educativas de cualquier empresa u organización que esté integrada con la Plataforma de identidad de Microsoft podrán iniciar sesión en la aplicación.

> Esta guía requiere Microsoft Visual Studio 2019.  ¿No lo tiene?  [Descargue Visual Studio 2019 de manera gratuita](https://www.visualstudio.com/downloads/).

>[!NOTE]
> Si es la primera vez que usa la plataforma de identidad de Microsoft, es aconsejable que comience por el artículo [Adición del inicio de sesión de la plataforma de identidad de Microsoft a una aplicación web de ASP.NET](quickstart-v2-aspnet-webapp.md).

## <a name="how-the-sample-app-generated-by-this-guide-works"></a>Funcionamiento de la aplicación de ejemplo generada por esta guía

![Muestra cómo funciona la aplicación de ejemplo generada por este tutorial](media/active-directory-develop-guidedsetup-aspnetwebapp-intro/aspnetbrowsergeneral.svg)

La aplicación de ejemplo que se crea se basa en un escenario en el que se usa el explorador para acceder a un sitio web de ASP.NET que solicita a un usuario que se autentique mediante un botón de inicio de sesión. En esa situación, la mayoría del trabajo para representar la página web se produce en el servidor.

## <a name="libraries"></a>Bibliotecas

Esta guía utiliza las siguientes bibliotecas:

|Biblioteca|Descripción|
|---|---|
|[Microsoft.Owin.Security.OpenIdConnect](https://www.nuget.org/packages/Microsoft.Owin.Security.OpenIdConnect/)|Middleware que permite a una aplicación que use OpenIDConnect para la autenticación|
|[Microsoft.Owin.Security.Cookies](https://www.nuget.org/packages/Microsoft.Owin.Security.Cookies)|Middleware que permite a una aplicación mantener una sesión de usuario mediante cookies|
|[Microsoft.Owin.Host.SystemWeb](https://www.nuget.org/packages/Microsoft.Owin.Host.SystemWeb)|Middleware que permite a las aplicaciones basadas en OWIN ejecutarse en Internet Information Services (IIS) mediante la canalización de solicitudes de ASP.NET|

## <a name="set-up-your-project"></a>Configurar su proyecto

En esta sección se describe cómo instalar y configurar la canalización de autenticación mediante middleware de OWIN en un proyecto de ASP.NET mediante OpenID Connect.

> ¿Prefiere descargar este proyecto de Visual Studio de ejemplo en su lugar? [Descargue un proyecto](https://github.com/AzureADQuickStarts/AppModelv2-WebApp-OpenIDConnect-DotNet/archive/master.zip) y vaya a la sección [Registro de la aplicación](#register-your-application) para configurar el código de ejemplo antes de ejecutarlo.

### <a name="create-your-aspnet-project"></a>Creación del proyecto de ASP.NET

1. En Visual Studio: Vaya a **Archivo** > **Nuevo** > **Proyecto**.
2. En **Visual C#\Web**, seleccione **Aplicación web ASP.NET (.NET Framework)** .
3. Asigne un nombre a la aplicación y seleccione **Aceptar**.
4. Seleccione **Vacío** y, después, seleccione la casilla para agregar referencias de **MVC**.

## <a name="add-authentication-components"></a>Adición de componentes de autenticación

1. En Visual Studio: Vaya a **Herramientas** > **Administrador de paquetes NuGet** > **Consola del administrador de paquetes**.
2. Agregue los *paquetes de NuGet de middleware de OWIN* escribiendo lo siguiente en la ventana de la Consola del Administrador de paquetes:

    ```powershell
    Install-Package Microsoft.Owin.Security.OpenIdConnect
    Install-Package Microsoft.Owin.Security.Cookies
    Install-Package Microsoft.Owin.Host.SystemWeb
    ```

### <a name="about-these-libraries"></a>Acerca de estas bibliotecas
Estas bibliotecas habilitan el inicio de sesión único (SSO) mediante OpenID Connect a través de la autenticación basada en cookies. Una vez que se completa la autenticación y se envía el token que representa al usuario a la aplicación, el middleware de OWIN crea una cookie de sesión. El explorador utiliza esta cookie en las solicitudes posteriores, con el fin de que el usuario no tenga que volver a escribir la contraseña; además, no es necesaria ninguna comprobación adicional.

## <a name="configure-the-authentication-pipeline"></a>Configuración de la canalización de autenticación

Los pasos siguientes se usan para crear una clase de inicio del middleware de OWIN para configurar la autenticación de OpenID Connect. Esta clase se ejecuta automáticamente cuando se inicia el proceso de IIS.

> [!TIP]
> Si el proyecto no tiene un archivo `Startup.cs` en la carpeta raíz:
> 1. Haga clic con el botón derecho en la carpeta raíz del proyecto y seleccione **Agregar** > **Nuevo elemento** > **Clase de inicio OWIN**.<br/>
> 2. Asígnele el nombre **Startup.cs**.
>
>> Asegúrese de que la clase seleccionada es una clase de inicio OWIN, no una clase estándar de C#. Para confirmarlo, compruebe que ve [assembly: OwinStartup(typeof({NameSpace}.Startup))] encima del espacio de nombres.

1. Agregue las referencias *OWIN* y *Microsoft.IdentityModel* a Startup.cs:

    ```csharp
    using Microsoft.Owin;
    using Owin;
    using Microsoft.IdentityModel.Protocols.OpenIdConnect;
    using Microsoft.IdentityModel.Tokens;
    using Microsoft.Owin.Security;
    using Microsoft.Owin.Security.Cookies;
    using Microsoft.Owin.Security.OpenIdConnect;
    using Microsoft.Owin.Security.Notifications;
    ```

2. Reemplace la clase de inicio por el código siguiente:

    ```csharp
    public class Startup
    {
        // The Client ID is used by the application to uniquely identify itself to Microsoft identity platform.
        string clientId = System.Configuration.ConfigurationManager.AppSettings["ClientId"];

        // RedirectUri is the URL where the user will be redirected to after they sign in.
        string redirectUri = System.Configuration.ConfigurationManager.AppSettings["RedirectUri"];

        // Tenant is the tenant ID (e.g. contoso.onmicrosoft.com, or 'common' for multi-tenant)
        static string tenant = System.Configuration.ConfigurationManager.AppSettings["Tenant"];

        // Authority is the URL for authority, composed by Microsoft identity platform endpoint and the tenant name (e.g. https://login.microsoftonline.com/contoso.onmicrosoft.com/v2.0)
        string authority = String.Format(System.Globalization.CultureInfo.InvariantCulture, System.Configuration.ConfigurationManager.AppSettings["Authority"], tenant);

        /// <summary>
        /// Configure OWIN to use OpenIdConnect
        /// </summary>
        /// <param name="app"></param>
        public void Configuration(IAppBuilder app)
        {
            app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);

            app.UseCookieAuthentication(new CookieAuthenticationOptions());
            app.UseOpenIdConnectAuthentication(
                new OpenIdConnectAuthenticationOptions
                {
                    // Sets the ClientId, authority, RedirectUri as obtained from web.config
                    ClientId = clientId,
                    Authority = authority,
                    RedirectUri = redirectUri,
                    // PostLogoutRedirectUri is the page that users will be redirected to after sign-out. In this case, it is using the home page
                    PostLogoutRedirectUri = redirectUri,
                    Scope = OpenIdConnectScope.OpenIdProfile,
                    // ResponseType is set to request the id_token - which contains basic information about the signed-in user
                    ResponseType = OpenIdConnectResponseType.IdToken,
                    // ValidateIssuer set to false to allow personal and work accounts from any organization to sign in to your application
                    // To only allow users from a single organizations, set ValidateIssuer to true and 'tenant' setting in web.config to the tenant name
                    // To allow users from only a list of specific organizations, set ValidateIssuer to true and use ValidIssuers parameter
                    TokenValidationParameters = new TokenValidationParameters()
                    {
                        ValidateIssuer = false // This is a simplification
                    },
                    // OpenIdConnectAuthenticationNotifications configures OWIN to send notification of failed authentications to OnAuthenticationFailed method
                    Notifications = new OpenIdConnectAuthenticationNotifications
                    {
                        AuthenticationFailed = OnAuthenticationFailed
                    }
                }
            );
        }

        /// <summary>
        /// Handle failed authentication requests by redirecting the user to the home page with an error in the query string
        /// </summary>
        /// <param name="context"></param>
        /// <returns></returns>
        private Task OnAuthenticationFailed(AuthenticationFailedNotification<OpenIdConnectMessage, OpenIdConnectAuthenticationOptions> context)
        {
            context.HandleResponse();
            context.Response.Redirect("/?errormessage=" + context.Exception.Message);
            return Task.FromResult(0);
        }
    }
    ```

> [!NOTE]
> Establecer `ValidateIssuer = false` es una simplificación para este inicio rápido. En las aplicaciones reales debe validar el emisor.
> Para aprender a hacerlo, vea los ejemplos.

### <a name="more-information"></a>Más información

Los parámetros que se proporcionan en *OpenIDConnectAuthenticationOptions* sirven de coordenadas para que la aplicación se comunique con la Plataforma de identidad de Microsoft. Dado que el middleware de OpenID Connect usa cookies en segundo plano, también debe configurar la autenticación con cookies como muestra el código anterior. El valor *ValidateIssuer* indica a OpenIdConnect que no restrinja el acceso a una organización específica.

## <a name="add-a-controller-to-handle-sign-in-and-sign-out-requests"></a>Agregar un controlador para controlar las solicitudes de inicio de sesión y cierre de sesión

Para crear un controlador que exponga los métodos de inicio de sesión y cierre de sesión, siga estos pasos:

1.  Haga clic con el botón derecho en la carpeta **Controladores** y seleccione **Agregar** > **Controlador**.
2.  Seleccione **Controlador MVC (versión .NET): vacío**.
3.  Seleccione **Agregar**.
4.  Asígnele el nombre**HomeController** y, después, seleccione **Agregar**.
5.  Agregue referencias de OWIN a la clase:

    ```csharp
    using Microsoft.Owin.Security;
    using Microsoft.Owin.Security.Cookies;
    using Microsoft.Owin.Security.OpenIdConnect;
    ```

6. Agregue los dos métodos siguientes para controlar el inicio de sesión y el cierre de sesión en el controlador iniciando un desafío de autenticación:

    ```csharp
    /// <summary>
    /// Send an OpenID Connect sign-in request.
    /// Alternatively, you can just decorate the SignIn method with the [Authorize] attribute
    /// </summary>
    public void SignIn()
    {
        if (!Request.IsAuthenticated)
        {
            HttpContext.GetOwinContext().Authentication.Challenge(
                new AuthenticationProperties{ RedirectUri = "/" },
                OpenIdConnectAuthenticationDefaults.AuthenticationType);
        }
    }

    /// <summary>
    /// Send an OpenID Connect sign-out request.
    /// </summary>
    public void SignOut()
    {
        HttpContext.GetOwinContext().Authentication.SignOut(
                OpenIdConnectAuthenticationDefaults.AuthenticationType,
                CookieAuthenticationDefaults.AuthenticationType);
    }
    ```

## <a name="create-the-apps-home-page-for-user-sign-in"></a>Creación de la página principal de la aplicación para el inicio de sesión del usuario

En Visual Studio, cree otra vista para agregar el botón de inicio de sesión y mostrar información del usuario tras la autenticación:

1.  Haga clic con el botón derecho en la carpeta **Vistas\Inicio** y seleccione **Agregar vista**.
2.  Asigne a la nueva vista el nombre **Índice**.
3.  Agregue el código HTML siguiente, que incluye el botón de inicio de sesión, al archivo:

    ```html
    <html>
    <head>
        <meta name="viewport" content="width=device-width" />
        <title>Sign in with Microsoft Guide</title>
    </head>
    <body>
    @if (!Request.IsAuthenticated)
    {
        <!-- If the user is not authenticated, display the sign-in button -->
        <a href="@Url.Action("SignIn", "Home")" style="text-decoration: none;">
            <svg xmlns="http://www.w3.org/2000/svg" xml:space="preserve" width="300px" height="50px" viewBox="0 0 3278 522" class="SignInButton">
            <style type="text/css">.fil0:hover {fill: #4B4B4B;} .fnt0 {font-size: 260px;font-family: 'Segoe UI Semibold', 'Segoe UI'; text-decoration: none;}</style>
            <rect class="fil0" x="2" y="2" width="3174" height="517" fill="black" />
            <rect x="150" y="129" width="122" height="122" fill="#F35325" />
            <rect x="284" y="129" width="122" height="122" fill="#81BC06" />
            <rect x="150" y="263" width="122" height="122" fill="#05A6F0" />
            <rect x="284" y="263" width="122" height="122" fill="#FFBA08" />
            <text x="470" y="357" fill="white" class="fnt0">Sign in with Microsoft</text>
            </svg>
        </a>
    }
    else
    {
        <span><br/>Hello @System.Security.Claims.ClaimsPrincipal.Current.FindFirst("name").Value;</span>
        <br /><br />
        @Html.ActionLink("See Your Claims", "Index", "Claims")
        <br /><br />
        @Html.ActionLink("Sign out", "SignOut", "Home")
    }
    @if (!string.IsNullOrWhiteSpace(Request.QueryString["errormessage"]))
    {
        <div style="background-color:red;color:white;font-weight: bold;">Error: @Request.QueryString["errormessage"]</div>
    }
    </body>
    </html>
    ```

### <a name="more-information"></a>Más información
Esta página agrega un botón de inicio de sesión en formato SVG con un fondo negro:<br/>![Iniciar sesión con Microsoft](media/active-directory-develop-guidedsetup-aspnetwebapp-use/aspnetsigninbuttonsample.png)<br/> Para más botones de inicio de sesión, vaya a [Directrices de personalización de marca](./howto-add-branding-in-azure-ad-apps.md "Directrices de marca").

## <a name="add-a-controller-to-display-users-claims"></a>Agregar un controlador para mostrar las notificaciones del usuario
Este controlador muestra los usos del atributo `[Authorize]` para proteger un controlador. Este atributo restringe el acceso al controlador, ya que solo permite usuarios autenticados. El código siguiente usa el atributo para mostrar las notificaciones de usuario que se recuperaron como parte del inicio de sesión:

1.  Haga clic con el botón derecho en la carpeta **Controladores** y seleccione **Agregar** > **Controlador**.
2.  Seleccione **Controlador MVC {version}: vacío**.
3.  Seleccione **Agregar**.
4.  Asígnele el nombre **ClaimsController**.
5.  Reemplace el código de la clase del controlador por el siguiente. De esta forma se agrega el atributo `[Authorize]` a la clase:

    ```csharp
    [Authorize]
    public class ClaimsController : Controller
    {
        /// <summary>
        /// Add user's claims to viewbag
        /// </summary>
        /// <returns></returns>
        public ActionResult Index()
        {
            var userClaims = User.Identity as System.Security.Claims.ClaimsIdentity;

            //You get the user’s first and last name below:
            ViewBag.Name = userClaims?.FindFirst("name")?.Value;

            // The 'preferred_username' claim can be used for showing the username
            ViewBag.Username = userClaims?.FindFirst("preferred_username")?.Value;

            // The subject/ NameIdentifier claim can be used to uniquely identify the user across the web
            ViewBag.Subject = userClaims?.FindFirst(System.Security.Claims.ClaimTypes.NameIdentifier)?.Value;

            // TenantId is the unique Tenant Id - which represents an organization in Azure AD
            ViewBag.TenantId = userClaims?.FindFirst("http://schemas.microsoft.com/identity/claims/tenantid")?.Value;

            return View();
        }
    }
    ```

### <a name="more-information"></a>Más información
Debido al uso del atributo `[Authorize]`, todos los métodos de este controlador solo pueden ejecutarse si el usuario está autenticado. Si no lo está e intenta acceder al controlador, OWIN inicia un desafío de autenticación y le obliga al usuario a autenticarse. El código anterior busca en la lista de notificaciones atributos de usuario específicos que se hayan incluido en el token del usuario. Estos atributos incluyen el nombre completo del usuario y el nombre de usuario, así como el firmante del identificador de usuario global. También contienen el *Id. del inquilino*, que representa el identificador de la organización del usuario.

## <a name="create-a-view-to-display-the-users-claims"></a>Creación de una vista que muestre las notificaciones del usuario

En Visual Studio, cree otra vista para mostrar las notificaciones del usuario en una página web:

1.  Haga clic con el botón derecho en la carpeta **Vistas\Notificaciones** y, después, seleccione **Agregar vista**.
2.  Asigne a la nueva vista el nombre **Índice**.
3.  Agregue el código HTML siguiente al archivo:

    ```html
    <html>
    <head>
        <meta name="viewport" content="width=device-width" />
        <title>Sign in with Microsoft Sample</title>
        <link href="@Url.Content("~/Content/bootstrap.min.css")" rel="stylesheet" type="text/css" />
    </head>
    <body style="padding:50px">
        <h3>Main Claims:</h3>
        <table class="table table-striped table-bordered table-hover">
            <tr><td>Name</td><td>@ViewBag.Name</td></tr>
            <tr><td>Username</td><td>@ViewBag.Username</td></tr>
            <tr><td>Subject</td><td>@ViewBag.Subject</td></tr>
            <tr><td>TenantId</td><td>@ViewBag.TenantId</td></tr>
        </table>
        <br />
        <h3>All Claims:</h3>
        <table class="table table-striped table-bordered table-hover table-condensed">
        @foreach (var claim in System.Security.Claims.ClaimsPrincipal.Current.Claims)
        {
            <tr><td>@claim.Type</td><td>@claim.Value</td></tr>
        }
        </table>
        <br />
        <br />
        @Html.ActionLink("Sign out", "SignOut", "Home", null, new { @class = "btn btn-primary" })
    </body>
    </html>
    ```

## <a name="register-your-application"></a>Registrar su aplicación

Tiene dos opciones para registrar la aplicación y agregar la información de registro de la misma a la solución:

### <a name="option-1-express-mode"></a>Opción 1: Modo rápido

Para registrar rápidamente la aplicación, siga estos pasos:

1. Vaya al nuevo panel de [Azure Portal: Registros de aplicaciones](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/applicationsListBlade/quickStartType/AspNetWebAppQuickstartPage/sourceType/docs).
1. Escriba un nombre para la aplicación y seleccione **Registrar**.
1. Siga las instrucciones para descargar y configurar automáticamente la nueva aplicación con un solo clic.

### <a name="option-2-advanced-mode"></a>Opción 2: Modo avanzado

Para registrar la aplicación y agregar la información de registro de la aplicación a la solución de forma manual, siga estos pasos:

1. Abra Visual Studio y:
   1. En el Explorador de soluciones, seleccione el proyecto y vea la ventana Propiedades (si no la ve, presione F4).
   1. Cambie SSL habilitado a `True`.
   1. Haga clic con el botón derecho en el proyecto en Visual Studio, seleccione **Propiedades** y, después, seleccione la pestaña **Web**. En la sección **Servidores**, cambie el valor **URL del proyecto** a **Dirección URL de SSL**.
   1. Copie la URL de SSL. En el siguiente paso, agregará esta dirección URL a la lista de direcciones URL de redireccionamiento del portal de registro correspondiente.<br/><br/>![Propiedades de proyecto](media/active-directory-develop-guidedsetup-aspnetwebapp-configure/vsprojectproperties.png)<br />
1. Inicie sesión en [Azure Portal](https://portal.azure.com) mediante una cuenta profesional o educativa, o bien una cuenta personal de Microsoft.
1. Si la cuenta proporciona acceso a más de un inquilino, haga clic en la cuenta en la esquina superior derecha y establezca la sesión del portal en el inquilino de Azure AD que desee.
1. Vaya a la página [Registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908) de la plataforma de identidad de Microsoft para desarrolladores.
1. Seleccione **Nuevo registro**.
1. Cuando aparece la página **Registrar una aplicación**, escriba la información de registro de la aplicación:
   1. En la sección **Nombre**, escriba un nombre de aplicación significativo que se mostrará a los usuarios de la aplicación, como **ASPNET-Tutorial**.
   1. Agregue la dirección URL de SSL que copió de Visual Studio en el paso 1 (por ejemplo, `https://localhost:44368/`) en **URL de respuesta** y seleccione **Registrar**.
1. Seleccione el menú **Autenticación**, seleccione **Tokens de id.** en **Concesión implícita** y, después, seleccione **Guardar**.
1. Agregue lo siguiente en el archivo web.config, que se encuentra en la carpeta raíz, en la sección `configuration\appSettings`:

    ```xml
    <add key="ClientId" value="Enter_the_Application_Id_here" />
    <add key="redirectUri" value="Enter_the_Redirect_URL_here" />
    <add key="Tenant" value="common" />
    <add key="Authority" value="https://login.microsoftonline.com/{0}/v2.0" />
    ```

1. Reemplace `ClientId` por el identificador de aplicación que acaba de registrar.
1. Reemplace `redirectUri` por la dirección URL de SSL del proyecto.

## <a name="test-your-code"></a>Prueba del código

Para probar la aplicación en Visual Studio, presione F5 para ejecutar el proyecto. El explorador se abre en la ubicación http://<span></span>localhost:{port}, donde verá el botón **Iniciar sesión con Microsoft**. Seleccione el botón para iniciar el proceso de inicio de sesión.

Cuando esté listo para realizar la prueba, use una cuenta de Azure AD (profesional o educativa) o una cuenta Microsoft personal (<span>live.</span>com, <span>outlook.</span>com) para iniciar sesión.

![Iniciar sesión con Microsoft](media/active-directory-develop-guidedsetup-aspnetwebapp-test/aspnetbrowsersignin.png)
<br/><br/>
![Iniciar sesión en la cuenta de Microsoft](media/active-directory-develop-guidedsetup-aspnetwebapp-test/aspnetbrowsersignin2.png)

#### <a name="permissions-and-consent-in-the-microsoft-identity-platform-endpoint"></a>Permisos y consentimiento en el punto de conexión de la Plataforma de identidad de Microsoft

Las aplicaciones que se integran con la plataforma de identidad de Microsoft siguen un modelo de autorización que permite a los usuarios y los administradores controlar el modo en que se puede acceder a los datos. Después de que un usuario se autentique con la Plataforma de identidad de Microsoft para acceder a esta aplicación, se le pedirá que dé su consentimiento a los permisos solicitados por la aplicación ("Ver su perfil básico" y "Mantener el acceso a los datos a los que ha dado acceso"). Después de aceptar estos permisos, el usuario continuará con los resultados de la aplicación. Sin embargo, en su lugar, el usuario puede ver la página **Need admin consent** (Se necesita el consentimiento del administrador) si se produce alguna de las siguientes situaciones:

- El desarrollador de aplicaciones agrega todos los permisos adicionales que requieran **consentimiento del administrador**.
- O bien se configura el inquilino (en **Aplicaciones empresariales -> Configuración de usuario**), donde los usuarios no pueden dar el consentimiento a las aplicaciones que accedan a los datos de la empresa en su nombre.

Para más información, consulte [Permisos y consentimiento en el punto de conexión de la Plataforma de identidad de Microsoft](./v2-permissions-and-consent.md).

### <a name="view-application-results"></a>Visualización de los resultados de la aplicación

Después de iniciar sesión, se redirige al usuario a la página principal del sitio web. Esta página principal es la dirección URL HTTPS especificada en la información de registro de la aplicación en el Portal de registro de aplicaciones de Microsoft. La página principal incluye un mensaje de bienvenida, *"Hola, \<user>"* , un vínculo para cerrar la sesión y otro para ver las notificaciones del usuario. El vínculo de las notificaciones del usuario se conecta al controlador de notificaciones que creó anteriormente.

### <a name="view-the-users-claims"></a>Ver las notificaciones del usuario

Para ver las notificaciones de usuario, seleccione el vínculo para ir a la vista del controlador que solo está disponible para los usuarios autenticados.

#### <a name="view-the-claims-results"></a>Ver los resultados de las notificaciones

Una vez en la vista del controlador, verá una tabla que contiene las propiedades básicas del usuario:

|Propiedad |Value |Descripción |
|---|---|---|
|**Nombre** |Nombre completo del usuario | Nombre y apellido del usuario
|**Nombre de usuario** |usuario<span>@domain.com</span> | Nombre de usuario que se usa para identificar al usuario|
|**Subject** |Asunto |Cadena que identifica al usuario de forma exclusiva en la web|
|**Id. de inquilino** |Guid | Un **GUID** que representa de forma única la organización de Azure AD del usuario|

Además, debería ver una tabla de todas las notificaciones que se encuentran en la solicitud de autenticación. Para más información, consulte la [lista de notificaciones que se encuentran en un token de identificación](./id-tokens.md).

### <a name="test-access-to-a-method-that-has-an-authorize-attribute-optional"></a>Probar el acceso a un método que tiene un atributo Authorize (opcional)

Para probar el acceso como usuario anónimo a un controlador que esté protegido por el atributo `Authorize`, siga estos pasos:

1. Seleccione el vínculo para cerrar la sesión del usuario y complete el proceso de cierre de sesión.
2. En el explorador, escriba http://<span></span>localhost:{port}/claims para acceder al controlador que está protegido por el atributo `Authorize`.

#### <a name="expected-results-after-access-to-a-protected-controller"></a>Resultados esperados después de obtener acceso a un controlador protegido

Se le pedirá que se autentique para poder usar la vista del controlador protegido.

## <a name="advanced-options"></a>Opciones avanzadas

### <a name="protect-your-entire-website"></a>Proteger todo el sitio web

Para proteger todo el sitio web, vaya al archivo **Global.asax** y agregue el atributo `AuthorizeAttribute` al filtro `GlobalFilters` que se encuentra en el método `Application_Start`:

```csharp
GlobalFilters.Filters.Add(new AuthorizeAttribute());
```

### <a name="restrict-who-can-sign-in-to-your-application"></a>Restringir quién puede iniciar sesión a la aplicación

De forma predeterminada, cuando compile la aplicación que creó con esta guía, la aplicación podrá aceptar inicios de sesión de cuentas personales (como outlook.com, live.com y otras), así como cuentas profesionales y educativas de cualquier empresa u organización que se haya integrado con la Plataforma de identidad de Microsoft. Esta es una opción recomendada para las aplicaciones SaaS.

Existen varias opciones para restringir el acceso de los usuarios al proceso de inicio de sesión de la aplicación.

#### <a name="option-1-restrict-users-from-only-one-organizations-active-directory-instance-to-sign-in-to-your-application-single-tenant"></a>Opción 1: Restricción de los usuarios de una instancia de Active Directory de solo una organización para que inicien sesión en la aplicación (único inquilino)

Esta opción se usa con frecuencia para las *aplicaciones de línea de negocio*: Si desea que la aplicación acepte inicios de sesión solo de cuentas que pertenezcan a una instancia específica de Azure AD (incluidas las *cuentas invitadas* de esa instancia), siga estos pasos:

1. En el archivo web.config, reemplace el valor del parámetro `Tenant` de `Common` por el nombre de inquilino de la organización como, por ejemplo, `contoso.onmicrosoft.com`.
2. A continuación, en la [clase OWIN Startup](#configure-the-authentication-pipeline), establezca el argumento `ValidateIssuer` en `true`.

#### <a name="option-2-restrict-access-to-users-in-a-specific-list-of-organizations"></a>Opción 2: Restringir el acceso a los usuarios de una lista específica de organizaciones

Puede restringir el acceso al proceso de inicio de sesión solo a aquellas cuentas de usuario que formen parte de una organización de Azure AD que se encuentre en una lista de organizaciones permitidas:
1. A continuación, en la [clase OWIN Startup](#configure-the-authentication-pipeline), establezca el argumento `ValidateIssuer` en `true`.
2. Establezca el valor del parámetro `ValidIssuers` en la lista de organizaciones permitidas.

#### <a name="option-3-use-a-custom-method-to-validate-issuers"></a>Opción 3: Usar un método personalizado para validar emisores

Puede implementar un método personalizado para validar los emisores con el parámetro **IssuerValidator**. Para más información acerca de cómo usar este parámetro, consulte clase [TokenValidationParameters](/dotnet/api/microsoft.identitymodel.tokens.tokenvalidationparameters).

## <a name="next-steps"></a>Pasos siguientes

Obtenga información acerca de la forma en que las aplicaciones web pueden llamar a las API web.

### <a name="learn-how-to-create-the-application-used-in-this-quickstart"></a>Aprenda a crear la aplicación que se ha usado en este inicio rápido.

Más información acerca de las aplicaciones web que llaman a API web con la plataforma de identidad de Microsoft:

> [!div class="nextstepaction"]
> [Aplicaciones web que llaman a las API web](scenario-web-app-sign-user-overview.md)

Aprenda a crear aplicaciones web que llamen a Microsoft Graph:

> [!div class="nextstepaction"]
> [Tutorial de Microsoft Graph ASP.NET](/graph/tutorials/aspnet)

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]