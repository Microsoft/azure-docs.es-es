---
title: Introducción a la plataforma de identidad de Microsoft en UWP | Azure
description: Cómo pueden llamar las aplicaciones de la Plataforma universal de Windows (UWP) a una API que requiera tokens de acceso mediante el punto de conexión de la plataforma de identidad de Microsoft.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.workload: identity
ms.date: 12/13/2019
ms.author: jmprieur
ms.custom: aaddev, identityplatformtop40
ms.openlocfilehash: 38bd83f8bd713f2d5bda658e052c919dd3d7625d
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88119715"
---
# <a name="call-the-microsoft-graph-api-from-a-universal-windows-platform-application-xaml"></a>Llamar a Microsoft Graph API desde una aplicación de la Plataforma universal de Windows (XAML)

> [!div renderon="docs"]

En esta guía se explica cómo una aplicación de la Plataforma universal de Windows (UWP) puede solicitar un token de acceso. A continuación, la aplicación llama a Microsoft Graph API. La guía también se aplica a otras API que requieren tokens de acceso del punto de la Plataforma de identidad de Microsoft.

Al final de esta guía, su aplicación llama a una API protegida mediante cuentas personales. Algunos ejemplos son outlook.com, live.com, etc. Su aplicación también llama a cuentas profesionales y educativas de cualquier empresa u organización que tenga Azure Active Directory (Azure AD).

>[!NOTE]
> En esta guía es necesario instalar el desarrollo de Visual Studio con la Plataforma universal de Windows. Para obtener instrucciones sobre cómo descargar y configurar Visual Studio para desarrollar aplicaciones de la Plataforma universal de Windows, consulte [Prepárese](/windows/uwp/get-started/get-set-up).

>[!NOTE]
> Si nunca ha usado la plataforma de identidad de Microsoft, comience con [Inicio rápido: Llamar a Microsoft Graph API desde una aplicación de la Plataforma universal de Windows (UWP)](quickstart-v2-uwp.md).

## <a name="how-this-guide-works"></a>Funcionamiento de esta guía

![Muestra cómo funciona la aplicación de ejemplo generada por este tutorial](./media/tutorial-v2-windows-uwp/uwp-intro.svg)

En esta guía se crea una aplicación de UWP de ejemplo que consulta Microsoft Graph API. En este escenario, se agrega un token a las solicitudes HTTP mediante el encabezado de autorización. La Biblioteca de autenticación de Microsoft administra las adquisiciones y renovaciones de los tokens.

## <a name="nuget-packages"></a>Paquetes NuGet

Esta guía utiliza los siguientes paquetes NuGet:

|Biblioteca|Descripción|
|---|---|
|[Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client)|Biblioteca de autenticación de Microsoft|
|[Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph)|Biblioteca de cliente de Microsoft Graph|

## <a name="set-up-your-project"></a>Configurar su proyecto

En esta sección se proporcionan instrucciones paso a paso para integrar una aplicación de Windows Desktop .NET (XAML) con el inicio de sesión de Microsoft. A continuación, la aplicación puede consultar las API web que requieran un token, como Microsoft Graph API.

En esta guía se crea una aplicación que muestra un botón que consulta Microsoft Graph API y un botón para cerrar la sesión. También se muestran los cuadros de texto que contienen los resultados de las llamadas.

> [!NOTE]
> ¿Desea descargar este proyecto de Visual Studio de ejemplo en lugar de crearlo? [Descargue un proyecto](https://github.com/Azure-Samples/active-directory-dotnet-native-uwp-v2/archive/msal3x.zip) y vaya al paso [Registro de aplicaciones](#register-your-application "paso de registro de aplicación") para configurar el código de ejemplo antes de ejecutarlo.

### <a name="create-your-application"></a>Creación de la aplicación

1. Abra Visual Studio y seleccione **Crear un proyecto**.
1. En **Crear un proyecto nuevo**, elija **Aplicación en blanco (Universal Windows)** para C# y seleccione **Siguiente**.
1. En **Configure su nuevo proyecto**, asigne un nombre al proyecto y seleccione **Crear**.
1. Si se le solicita, en **Nuevo proyecto de la plataforma universal de Windows**, seleccione cualquier versión de tipo **Destino** y **Mínima**, y seleccione **Aceptar**.

   ![Versiones mínima y de destino](./media/tutorial-v2-windows-uwp/select-uwp-target-minimum.png)

### <a name="add-microsoft-authentication-library-to-your-project"></a>Adición de la biblioteca de autenticación de Microsoft a su proyecto

1. En Visual Studio, seleccione **Herramientas** > **Administrador de paquetes NuGet** > **Consola del administrador de paquetes**.
1. Copie y pegue los siguientes comandos en la ventana de la **Consola del administrador de paquetes**:

    ```powershell
    Install-Package Microsoft.Identity.Client
    Install-Package Microsoft.Graph
    ```

   > [!NOTE]
   > El primer comando instala la [Biblioteca de autenticación de Microsoft (MSAL.NET)](https://aka.ms/msal-net). MSAL.NET adquiere, almacena en memoria caché y actualiza los tokens de usuario que acceden a las API protegidas mediante la Plataforma de identidad de Microsoft. El segundo comando instala la [Biblioteca de cliente Microsoft Graph .NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet) para autenticar las solicitudes a Microsoft Graph y realizar llamadas al servicio.

### <a name="create-your-applications-ui"></a>Creación de la interfaz de usuario de la aplicación

Visual Studio crea *MainPage.xaml* como parte de la plantilla de proyecto. Abra este archivo y reemplace el nodo **Grid** de la aplicación por el código siguiente:

```xml
<Grid>
    <StackPanel Background="Azure">
        <StackPanel Orientation="Horizontal" HorizontalAlignment="Right">
            <Button x:Name="CallGraphButton" Content="Call Microsoft Graph API" HorizontalAlignment="Right" Padding="5" Click="CallGraphButton_Click" Margin="5" FontFamily="Segoe Ui"/>
            <Button x:Name="SignOutButton" Content="Sign-Out" HorizontalAlignment="Right" Padding="5" Click="SignOutButton_Click" Margin="5" Visibility="Collapsed" FontFamily="Segoe Ui"/>
        </StackPanel>
        <TextBlock Text="API Call Results" Margin="2,0,0,-5" FontFamily="Segoe Ui" />
        <TextBox x:Name="ResultText" TextWrapping="Wrap" MinHeight="120" Margin="5" FontFamily="Segoe Ui"/>
        <TextBlock Text="Token Info" Margin="2,0,0,-5" FontFamily="Segoe Ui" />
        <TextBox x:Name="TokenInfoText" TextWrapping="Wrap" MinHeight="70" Margin="5" FontFamily="Segoe Ui"/>
    </StackPanel>
</Grid>
```

### <a name="use-microsoft-authentication-library-to-get-a-token-for-the-microsoft-graph-api"></a>Utilice la Biblioteca de autenticación de Microsoft para obtener un token para Microsoft Graph API.

En esta sección se muestra cómo usar la Biblioteca de autenticación de Microsoft para obtener un token para Microsoft Graph API. Haga cambios en el archivo *MainPage.xaml.cs*.

1. En *MainPage.xaml.cs*, agregue las siguientes referencias:

    ```csharp
    using Microsoft.Identity.Client;
    using Microsoft.Graph;
    using System.Diagnostics;
    using System.Threading.Tasks;
    ```

1. Reemplace la clase `MainPage` por el código siguiente:

    ```csharp
    public sealed partial class MainPage : Page
    {
       
        //Set the scope for API call to user.read
        private string[] scopes = new string[] { "user.read" };

        // Below are the clientId (Application Id) of your app registration and the tenant information.
        // You have to replace:
        // - the content of ClientID with the Application Id for your app registration
        private const string ClientId = "[Application Id pasted from the application registration portal]";

        private const string Tenant = "common"; // Alternatively "[Enter your tenant, as obtained from the Azure portal, e.g. kko365.onmicrosoft.com]"
        private const string Authority = "https://login.microsoftonline.com/" + Tenant;

        // The MSAL Public client app
        private static IPublicClientApplication PublicClientApp;

        private static string MSGraphURL = "https://graph.microsoft.com/v1.0/";
        private static AuthenticationResult authResult;

        public MainPage()
        {
            this.InitializeComponent();
        }

        /// <summary>
        /// Call AcquireTokenAsync - to acquire a token requiring user to sign in
        /// </summary>
        private async void CallGraphButton_Click(object sender, RoutedEventArgs e)
        {
            try
            {
                // Sign in user using MSAL and obtain an access token for Microsoft Graph
                GraphServiceClient graphClient = await SignInAndInitializeGraphServiceClient(scopes);

                // Call the /me endpoint of Graph
                User graphUser = await graphClient.Me.Request().GetAsync();

                // Go back to the UI thread to make changes to the UI
                await Dispatcher.RunAsync(Windows.UI.Core.CoreDispatcherPriority.Normal, () =>
                {
                    ResultText.Text = "Display Name: " + graphUser.DisplayName + "\nBusiness Phone: " + graphUser.BusinessPhones.FirstOrDefault()
                                      + "\nGiven Name: " + graphUser.GivenName + "\nid: " + graphUser.Id
                                      + "\nUser Principal Name: " + graphUser.UserPrincipalName;
                    DisplayBasicTokenInfo(authResult);
                    this.SignOutButton.Visibility = Visibility.Visible;
                });
            }
            catch (MsalException msalEx)
            {
                await DisplayMessageAsync($"Error Acquiring Token:{System.Environment.NewLine}{msalEx}");
            }
            catch (Exception ex)
            {
                await DisplayMessageAsync($"Error Acquiring Token Silently:{System.Environment.NewLine}{ex}");
                return;
            }
        }
                /// <summary>
        /// Signs in the user and obtains an access token for Microsoft Graph
        /// </summary>
        /// <param name="scopes"></param>
        /// <returns> Access Token</returns>
        private static async Task<string> SignInUserAndGetTokenUsingMSAL(string[] scopes)
        {
            // Initialize the MSAL library by building a public client application
            PublicClientApp = PublicClientApplicationBuilder.Create(ClientId)
                .WithAuthority(Authority)
                .WithUseCorporateNetwork(false)
                .WithRedirectUri("https://login.microsoftonline.com/common/oauth2/nativeclient")
                 .WithLogging((level, message, containsPii) =>
                 {
                     Debug.WriteLine($"MSAL: {level} {message} ");
                 }, LogLevel.Warning, enablePiiLogging: false, enableDefaultPlatformLogging: true)
                .Build();

            // It's good practice to not do work on the UI thread, so use ConfigureAwait(false) whenever possible.
            IEnumerable<IAccount> accounts = await PublicClientApp.GetAccountsAsync().ConfigureAwait(false);
            IAccount firstAccount = accounts.FirstOrDefault();

            try
            {
                authResult = await PublicClientApp.AcquireTokenSilent(scopes, firstAccount)
                                                  .ExecuteAsync();
            }
            catch (MsalUiRequiredException ex)
            {
                // A MsalUiRequiredException happened on AcquireTokenSilentAsync. This indicates you need to call AcquireTokenAsync to acquire a token
                Debug.WriteLine($"MsalUiRequiredException: {ex.Message}");

                authResult = await PublicClientApp.AcquireTokenInteractive(scopes)
                                                  .ExecuteAsync()
                                                  .ConfigureAwait(false);

            }
            return authResult.AccessToken;
        }
    }
    ```

#### <a name="get-a-user-token-interactively"></a>Obtención de un token de usuario interactivamente<a name="more-information"></a>

El método `AcquireTokenInteractive` genera una ventana que pide al usuario que inicie sesión. Las aplicaciones suelen requerir a los usuarios que inicien sesión de forma interactiva la primera vez que acceden a un recurso protegido. Es posible que también deban iniciar sesión cuando se produce un error en una operación silenciosa para adquirir un token. Por ejemplo, si ha caducado la contraseña de un usuario.

#### <a name="get-a-user-token-silently"></a>Obtención de un token de usuario en silencio

El método `AcquireTokenSilent` controla las renovaciones y las adquisiciones de tokens sin la interacción del usuario. Después de que `AcquireTokenInteractive` se ejecute por primera vez y solicite las credenciales del usuario, use el método `AcquireTokenSilent` para solicitar tokens para las llamadas posteriores. Ese método adquiere los tokens de forma silenciosa. La Biblioteca de autenticación de Microsoft controla la renovación y la memoria caché de tokens.

En última instancia, se producirá un error en el método `AcquireTokenSilent`. El error puede deberse a que los usuarios hayan cerrado sesión o cambiado su contraseña en otro dispositivo. Cuando la Biblioteca de autenticación de Microsoft detecta que el problema requiere una acción interactiva, se produce una excepción `MsalUiRequiredException`. La aplicación puede abordar esta excepción de dos maneras:

* La aplicación llama a `AcquireTokenInteractive` inmediatamente. Esta llamada provoca que se solicite al usuario que inicie sesión. Este enfoque se usa normalmente en aplicaciones en línea en las que no hay ningún contenido sin conexión disponible para el usuario. El ejemplo generado por esta instalación guiada sigue este patrón. Lo ve en acción la primera vez que ejecuta el ejemplo.

   Como ningún usuario ha usado la aplicación, `accounts.FirstOrDefault()` contiene un valor NULL y se inicia una excepción `MsalUiRequiredException`.

   El código del ejemplo se ocupa entonces de la excepción llamando a `AcquireTokenInteractive`. Esta llamada provoca que se solicite al usuario que inicie sesión.

* La aplicación muestra una indicación visual a los usuarios que necesitan iniciar sesión. A continuación, se puede seleccionar el momento adecuado para iniciar sesión. La aplicación puede volver a probar el método `AcquireTokenSilent` más tarde. Este enfoque se usa cuando los usuarios pueden utilizar otra funcionalidad de la aplicación sin interrupciones. Por ejemplo, si hay contenido sin conexión disponible en la aplicación. En este caso, los usuarios pueden decidir cuándo desean iniciar sesión. La aplicación puede reintentar `AcquireTokenSilent` después de que la red no esté disponible temporalmente.

### <a name="instantiate-the-microsoft-graph-service-client-by-obtaining-the-token-from-the-signinuserandgettokenusingmsal-method"></a>Creación de una instancia del cliente del servicio Microsoft Graph mediante la obtención del token del método SignInUserAndGetTokenUsingMSAL

Agregue el siguiente método nuevo al archivo *MainPage.xaml.cs*:

```csharp
      /// <summary>
     /// Sign in user using MSAL and obtain a token for Microsoft Graph
     /// </summary>
     /// <returns>GraphServiceClient</returns>
     private async static Task<GraphServiceClient> SignInAndInitializeGraphServiceClient(string[] scopes)
     {
         GraphServiceClient graphClient = new GraphServiceClient(MSGraphURL,
             new DelegateAuthenticationProvider(async (requestMessage) =>
             {
                 requestMessage.Headers.Authorization = new AuthenticationHeaderValue("bearer", await SignInUserAndGetTokenUsingMSAL(scopes));
             }));

         return await Task.FromResult(graphClient);
     }
```

#### <a name="more-information-on-making-a-rest-call-against-a-protected-api"></a>Más información acerca de cómo realizar una llamada de REST a una API protegida

En esta aplicación de ejemplo, el método `GetGraphServiceClient` crea instancias de `GraphServiceClient` mediante un token de acceso. A continuación, se usa `GraphServiceClient` para obtener la información de perfil del usuario del punto de conexión **me**.

### <a name="add-a-method-to-sign-out-the-user"></a>Adición de un método para cerrar la sesión del usuario

Para cerrar la sesión del usuario, agregue el método siguiente a *MainPage.xaml.cs*:

```csharp
/// <summary>
/// Sign out the current user
/// </summary>
private async void SignOutButton_Click(object sender, RoutedEventArgs e)
{
    IEnumerable<IAccount> accounts = await PublicClientApp.GetAccountsAsync().ConfigureAwait(false);
    IAccount firstAccount = accounts.FirstOrDefault();

    try
    {
        await PublicClientApp.RemoveAsync(firstAccount).ConfigureAwait(false);
        await Dispatcher.RunAsync(Windows.UI.Core.CoreDispatcherPriority.Normal, () =>
        {
            ResultText.Text = "User has signed out";
            this.CallGraphButton.Visibility = Visibility.Visible;
                this.SignOutButton.Visibility = Visibility.Collapsed;
            });
        }
        catch (MsalException ex)
        {
            ResultText.Text = $"Error signing out user: {ex.Message}";
        }
    }
```

> [!NOTE]
> MSAL.NET usa métodos asincrónicos para adquirir tokens o manipular cuentas. Debe admitir acciones de interfaz de usuario en el subproceso de la interfaz de usuario. Este es el motivo de la llamada a `Dispatcher.RunAsync` y las precauciones para llamar a `ConfigureAwait(false)`.

#### <a name="more-information-about-signing-out"></a>Más información acerca de cómo cerrar sesión<a name="more-information-on-sign-out"></a>

El método `SignOutButton_Click` quita el usuario de la memoria caché de usuario de la Biblioteca de autenticación de Microsoft. Este método indica de forma eficaz a la Biblioteca de autenticación de Microsoft que olvide al usuario actual. Una futura solicitud de adquisición de un token solo se realizará correctamente si es interactiva.

La aplicación de este ejemplo admite un solo usuario. La Biblioteca de autenticación de Microsoft admite escenarios en los que el usuario puede iniciar sesión en más de una cuenta. Un ejemplo de esto es una aplicación de correo electrónico donde un usuario tiene varias cuentas.

### <a name="display-basic-token-information"></a>Visualización de información de token básica

Para mostrar información básica sobre el token, agregue el método siguiente al archivo *MainPage.xaml.cs*:

```csharp
/// <summary>
/// Display basic information contained in the token. Needs to be called from the UI thread.
/// </summary>
private void DisplayBasicTokenInfo(AuthenticationResult authResult)
{
    TokenInfoText.Text = "";
    if (authResult != null)
    {
        TokenInfoText.Text += $"User Name: {authResult.Account.Username}" + Environment.NewLine;
        TokenInfoText.Text += $"Token Expires: {authResult.ExpiresOn.ToLocalTime()}" + Environment.NewLine;
    }
}
```

#### <a name="more-information"></a>Más información<a name="more-information-1"></a>

Los identificadores de token adquiridos mediante **OpenID Connect** también contienen un pequeño subconjunto de información relativa al usuario. `DisplayBasicTokenInfo` muestra información básica incluida en el token. Esta información incluye el nombre para mostrar y el identificador del usuario. También incluye la fecha de expiración del token y la cadena que representa al propio token de acceso. Si selecciona el botón **Llamar a Microsoft Graph API** varias veces, verá que el mismo token se reutilizó para solicitudes posteriores. También puede ver la fecha de expiración extendida cuando la Biblioteca de autenticación de Microsoft decide que es el momento de renovar el token.

### <a name="display-message"></a>Mensaje para mostrar

Agregue el siguiente método nuevo al archivo *MainPage.xaml.cs*:

```csharp
/// <summary>
/// Displays a message in the ResultText. Can be called from any thread.
/// </summary>
private async Task DisplayMessageAsync(string message)
{
     await Dispatcher.RunAsync(Windows.UI.Core.CoreDispatcherPriority.Normal,
         () =>
         {
             ResultText.Text = message;
         });
     }
```

## <a name="register-your-application"></a>Registrar su aplicación

Ahora debe registrar la aplicación:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione **Azure Active Directory** > **Registros de aplicaciones**.
1. Seleccione **Nuevo registro**. Escriba un nombre de aplicación significativo que se mostrará a los usuarios de la misma, por ejemplo, *UWP-App-calling-MSGraph*.
1. En **Tipos de cuenta compatibles**, seleccione **Cuentas en cualquier directorio organizativo y cuentas Microsoft personales (por ejemplo, Skype o Xbox)** . A continuación, seleccione **Registrar** para continuar.
1. En la página Información general, busque el valor de **Id. de aplicación (cliente)** y cópielo. Vuelva a Visual Studio, abra *MainPage.xaml.cs* y reemplace el valor de `ClientId` por este.

Configure la autenticación para su aplicación:

1. De nuevo en [Azure Portal](https://portal.azure.com), en **Administrar**, seleccione **Autenticación**.
1. En la sección **URI de redirección** | **URI de redireccionamiento sugeridos para clientes públicos (para dispositivos móviles o de escritorio)** , marque https://login.microsoftonline.com/common/oauth2/nativeclient.
1. Seleccione **Guardar**.

Configure los permisos de la API para la aplicación:

1. En **Administrar**, seleccione **Permisos de API**.
1. Seleccione **Agregar un permiso** y asegúrese de que ha seleccionado **API de Microsoft**.
1. Seleccione **Microsoft Graph**.
1. Seleccione **Permisos delegados**, busque *User.Read* y compruebe que **User.Read** esté seleccionado.
1. Si realizó algún cambio, seleccione **Agregar permisos** para guardarlos.

## <a name="enable-integrated-authentication-on-federated-domains-optional"></a>Habilitar la autenticación integrada en dominios federados (opcional)

Para habilitar la autenticación integrada de Windows cuando se utiliza con un dominio federado de Azure AD, el manifiesto de aplicación debe habilitar funcionalidades adicionales. Vuelva a su aplicación en Visual Studio.

1. Abra *Package.appxmanifest*.
1. Seleccione **Funcionalidades** y habilite las siguientes opciones:

   * **Autenticación empresarial**
   * **Redes privadas (cliente y servidor)**
   * **Certificados de usuario compartidos**

> [!IMPORTANT]
> La [autenticación de Windows integrada](https://aka.ms/msal-net-iwa) no está configurada de forma predeterminada para este ejemplo. Las aplicaciones que solicitan funcionalidades `Enterprise Authentication` o `Shared User Certificates` requieren un mayor nivel de comprobación por parte de Windows Store. Además, no todos los desarrolladores desean llevar a cabo un nivel de comprobación más alto. Habilite esta opción solo si necesita usar la autenticación de Windows integrada con un dominio de Azure AD federado.

## <a name="alternate-approach-to-using-withdefaultredirecturi"></a>Enfoque alternativo al uso de WithDefaultRedirectURI()

En el ejemplo actual, se usa el método `WithRedirectUri("https://login.microsoftonline.com/common/oauth2/nativeclient")`. Para usar `WithDefaultRedirectURI()`, complete también los siguientes pasos:

1. En *MainPage.XAML.cs*, reemplace `WithRedirectUri` por `WithDefaultRedirectUri`:

   **Código actual**

   ```csharp

   PublicClientApp = PublicClientApplicationBuilder.Create(ClientId)
       .WithAuthority(Authority)
       .WithUseCorporateNetwork(false)
       .WithRedirectUri("https://login.microsoftonline.com/common/oauth2/nativeclient")
       .WithLogging((level, message, containsPii) =>
        {
            Debug.WriteLine($"MSAL: {level} {message} ");
        }, LogLevel.Warning, enablePiiLogging: false, enableDefaultPlatformLogging: true)
       .Build();

   ```

   **Código actualizado**

   ```csharp

   PublicClientApp = PublicClientApplicationBuilder.Create(ClientId)
       .WithAuthority("https://login.microsoftonline.com/common")
       .WithUseCorporateNetwork(false)
       .WithDefaultRedirectUri()
       .WithLogging((level, message, containsPii) =>
        {
            Debug.WriteLine($"MSAL: {level} {message} ");
        }, LogLevel.Warning, enablePiiLogging: false, enableDefaultPlatformLogging: true)
       .Build();
   ```

2.  Busque el URI de devolución de llamada para la aplicación; para ello, agregue el campo `redirectURI` en *MainPage.xaml.cs* y establezca un punto de interrupción en él:

    ```csharp

    public sealed partial class MainPage : Page
    {
            ...

            string redirectURI = Windows.Security.Authentication.Web.WebAuthenticationBroker
                                .GetCurrentApplicationCallbackUri().ToString();
            public MainPage()
            {
                ...
            }
           ...
    }
  
    ```

    Ejecute la aplicación y, después, copie el valor de `redirectUri` cuando se alcance el punto de interrupción. El valor tendrá un aspecto similar al siguiente:  
    `ms-app://s-1-15-2-1352796503-54529114-405753024-3540103335-3203256200-511895534-1429095407/`

    Después, puede quitar la línea de código porque solo es necesaria una vez para capturar el valor. 

3. En el portal de registro de aplicaciones, agregue el valor devuelto en **RedirectUri**, en el panel **Autenticación**.
   
## <a name="test-your-code"></a>Prueba del código

Para probar la aplicación, seleccione **F5** para ejecutar el proyecto en Visual Studio. Aparecerá la ventana principal:

![Interfaz de usuario de la aplicación](./media/tutorial-v2-windows-uwp/testapp-ui-vs2019.png)

Cuando esté listo para realizar la prueba, seleccione **Llamar a Microsoft Graph API**. A continuación, use una cuenta profesional de Azure AD o una cuenta Microsoft, como live.com u outlook.com, para iniciar sesión. La primera vez que un usuario ejecuta esta prueba, la aplicación muestra una ventana que pide al usuario que inicie sesión.

### <a name="consent"></a>Consentimiento

La primera vez que inicie sesión en la aplicación, aparecerá una pantalla de consentimiento similar a la siguiente. Seleccione **Sí** para consentir el acceso de forma explícita:

![Acceso a pantalla de consentimiento](./media/tutorial-v2-windows-uwp/consentscreen-vs2019.png)

### <a name="expected-results"></a>Resultados esperados

Verá la información de perfil de usuario que devolvió la llamada de Microsoft Graph API en la pantalla de **resultados de llamadas de la API**:

![Pantalla de resultados de llamada a la API](./media/tutorial-v2-windows-uwp/uwp-results-screen-vs2019.png)

También verá información básica sobre el token adquirido a través de `AcquireTokenInteractive` o `AcquireTokenSilent` en el cuadro de **información de token**:

|Propiedad  |Formato  |Descripción |
|---------|---------|---------|
|`Username` |`user@domain.com` |El nombre de usuario que identifica al usuario.|
|`Token Expires` |`DateTime` |La hora a la que expira el token. Para extender la fecha de expiración, la Biblioteca de autenticación de Microsoft renueva el token según sea necesario.|

### <a name="more-information-about-scopes-and-delegated-permissions"></a>Más información sobre los ámbitos y permisos delegados

Microsoft Graph API requiere el ámbito `user.read` para leer un perfil de usuario. Este ámbito se agrega de forma predeterminada en todas las aplicaciones que se van a registrar en el Portal de registro de aplicaciones. Otras API de Microsoft Graph, así como las API personalizadas para el servidor back-end, pueden requerir ámbitos adicionales. Por ejemplo, Microsoft Graph API requiere que el ámbito `Calendars.Read` enumere los calendarios del usuario.

Para acceder a los calendarios del usuario en el contexto de una aplicación, agregue el permiso delegado `Calendars.Read` a la información del registro de la aplicación. A continuación, agregue el ámbito `Calendars.Read` a la llamada `acquireTokenSilent`.

> [!NOTE]
> Es posible que se pida al usuario algún consentimiento adicional a medida que aumente el número de ámbitos.

## <a name="known-issues"></a>Problemas conocidos

### <a name="issue-1"></a>Problema 1

Recibirá uno de los siguientes mensajes de error al iniciar sesión en la aplicación en un dominio federado de Azure AD:

* "No se encontró un certificado de cliente válido en la solicitud".
* "No se encontraron certificados válidos en el almacén de certificados del usuario".
* "Vuelva a intentarlo mediante un método de autenticación diferente".

**Causa:** las funcionalidades de Enterprise y los certificados no están habilitados.

**Solución:** Siga los pasos descritos en [Habilitar la autenticación integrada en dominios federados (opcional)](#enable-integrated-authentication-on-federated-domains-optional).

### <a name="issue-2"></a>Problema 2

Puede habilitar la [autenticación integrada en dominios federados](#enable-integrated-authentication-on-federated-domains-optional) e intentar usar Windows Hello en un equipo con Windows 10 para iniciar sesión en un entorno con la autenticación multifactor configurada. Aparece la lista de certificados. Si decide usar el código PIN, la ventana de PIN nunca aparece.

**Causa:** este problema es una limitación conocida del agente de autenticación web en aplicaciones de UWP que se ejecutan en el escritorio de Windows 10. Funciona bien en Windows 10 Mobile.

**Solución alternativa:** seleccione **Iniciar sesión con otras opciones**. A continuación, seleccione **Iniciar sesión con un nombre de usuario y contraseña**. Seleccione **Proporcionar la contraseña**. A continuación, pase por el proceso de autenticación telefónica.

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]