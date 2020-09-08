---
title: Introducción a la plataforma de identidad de Microsoft para el escritorio de Windows
description: Cómo una aplicación .NET de escritorio de Windows (XAML) puede obtener un token de acceso y llamar a una API protegida por la plataforma de identidad de Microsoft.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.workload: identity
ms.date: 12/12/2019
ms.author: jmprieur
ms.custom: aaddev, identityplatformtop40
ms.openlocfilehash: a865bab690c79288bdffcd7cebe424d1bb1969c0
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "82181550"
---
# <a name="call-the-microsoft-graph-api-from-a-windows-desktop-app"></a>Llamada a Microsoft Graph API desde una aplicación de escritorio de Windows

En esta guía se muestra cómo una aplicación nativa de Windows Desktop .NET (XAML) utiliza un token de acceso para llamar a Microsoft Graph API. La aplicación también puede acceder a otras API que requieren tokens de acceso desde una plataforma de identidad de Microsoft para desarrolladores del punto de conexión v2.0. Esta plataforma se denominaba anteriormente Azure AD.

Cuando haya completado la guía, la aplicación podrá llamar a una API protegida que usa cuentas personales (lo que incluye outlook.com, live.com y otras). La aplicación también usa cuentas profesionales y educativas de cualquier empresa y organización que utilice Azure Active Directory.

> [!NOTE]
> Esta guía requiere Visual Studio 2015 Update 3, Visual Studio 2017 o Visual Studio 2019. ¿No tiene ninguna de estas versiones? [Descargue Visual Studio 2019 de manera gratuita](https://www.visualstudio.com/downloads/).

>[!NOTE]
> Si es la primera vez que usa la plataforma de identidad de Microsoft, es aconsejable que comience con [Adquisición de un token y llamada a Microsoft Graph API desde una aplicación de escritorio de Windows](quickstart-v2-windows-desktop.md).

## <a name="how-the-sample-app-generated-by-this-guide-works"></a>Funcionamiento de la aplicación de ejemplo generada por esta guía

![Muestra cómo funciona la aplicación de ejemplo generada por este tutorial](./media/active-directory-develop-guidedsetup-windesktop-intro/windesktophowitworks.svg)

La aplicación de ejemplo que se crea con esta guía permite que una aplicación de escritorio de Windows consulte Microsoft Graph API o una API web que acepte tokens de un punto de conexión de la Plataforma de identidad de Microsoft. En este escenario, agregará un token a las solicitudes HTTP mediante el encabezado de autorización. La Biblioteca de autenticación de Microsoft (MSAL) administra la adquisición y renovación de los tokens.

## <a name="handling-token-acquisition-for-accessing-protected-web-apis"></a>Tratamiento de la adquisición de tokens para acceder a API web protegidas

Una vez que el usuario se autentica, la aplicación de ejemplo recibe un token que se puede usar para consultar Microsoft Graph API o a una API web protegida mediante la Plataforma de identidad de Microsoft para desarrolladores.

Las API, como Microsoft Graph, requieren un token para permitir el acceso a recursos específicos. Por ejemplo, un token es necesario para leer un perfil del usuario, acceder al calendario de un usuario o enviar correo electrónico. La aplicación puede solicitar un token de acceso mediante MSAL para acceder a estos recursos mediante la especificación de ámbitos de API. Este token de acceso se agrega luego al encabezado de autorización de HTTP para todas las llamadas efectuadas al recurso protegido.

MSAL administra el almacenamiento en caché y la actualización de los tokens de acceso automáticamente, así que no tiene que preocuparse por ello.

## <a name="nuget-packages"></a>Paquetes NuGet

Esta guía utiliza los siguientes paquetes NuGet:

|Biblioteca|Descripción|
|---|---|
|[Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client)|Biblioteca de autenticación de Microsoft (MSAL.NET)|

## <a name="set-up-your-project"></a>Configurar su proyecto

En esta sección, creará un nuevo proyecto para mostrar cómo integrar una aplicación .NET de escritorio de Windows (XAML) con el *inicio de sesión en Microsoft* para que la aplicación pueda consultar las API web que requieren un token.

La aplicación que se crea con esta guía muestra un botón que se utiliza para llamar a un gráfico, un área para mostrar los resultados en la pantalla y un botón de cierre de sesión.

> [!NOTE]
> ¿Prefiere descargar este proyecto de Visual Studio de ejemplo en su lugar? [Descargue un proyecto](https://github.com/Azure-Samples/active-directory-dotnet-desktop-msgraph-v2/archive/msal3x.zip) y vaya al [paso de configuración](#register-your-application) para configurar el código de ejemplo antes de ejecutarlo.
>

Para crear la aplicación, lleva a cabo los siguientes pasos:

1. Abra Visual Studio, seleccione **Archivo** > **Nuevo** > **Proyecto**.
2. En **Plantillas**, seleccione **Visual C#** .
3. Seleccione **Aplicación de WPF (.NET Framework)** , en función de la versión de Visual Studio que use.

## <a name="add-msal-to-your-project"></a>Adición de MSAL al proyecto

1. En Visual Studio, seleccione **Herramientas** > **Administrador de paquetes NuGet**> **Consola del administrador de paquetes**.
2. En la ventana de la Consola del administrador de paquetes, pegue el siguiente comando de Azure PowerShell:

    ```powershell
    Install-Package Microsoft.Identity.Client -Pre
    ```

    > [!NOTE]
    > Este comando instala la biblioteca de autenticación de Microsoft. MSAL controla la adquisición, el almacenamiento en caché y la actualización de los tokens de usuario que se usan para acceder a las API protegidas por Azure Active Directory v2.0
    >

## <a name="register-your-application"></a>Registrar su aplicación

Puede registrar la aplicación de dos maneras.

### <a name="option-1-express-mode"></a>Opción 1: Modo rápido

Puede registrar rápidamente la aplicación mediante estos pasos:
1. Vaya a [Azure Portal: registro de aplicaciones](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/applicationsListBlade/quickStartType/WinDesktopQuickstartPage/sourceType/docs).
1. Escriba un nombre para la aplicación y seleccione **Registrar**.
1. Siga las instrucciones para descargar y configurar automáticamente la nueva aplicación con un solo clic.

### <a name="option-2-advanced-mode"></a>Opción 2: Modo avanzado

Para registrar la aplicación y agregar la información de registro de aplicación a la solución, siga estos pasos:
1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. Si la cuenta proporciona acceso a más de un inquilino, haga clic en la cuenta en la esquina superior derecha y establezca la sesión del portal en el inquilino de Azure AD deseado.
1. Vaya a la página [Registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908) de la plataforma de identidad de Microsoft para desarrolladores.
1. Seleccione **Nuevo registro**.
   - En la sección **Nombre**, escriba un nombre significativo para la aplicación, que se mostrará a los usuarios de la aplicación, por ejemplo, `Win-App-calling-MsGraph`.
   - En **Tipos de cuenta admitidos**, seleccione **Cuentas en cualquier directorio de organización y cuentas personales de Microsoft (por ejemplo, Skype, Xbox o Outlook.com)** .
   - Seleccione **Registrar** para crear la aplicación.
1. En la lista de páginas de la aplicación, seleccione **Autenticación**.
   1. En la sección **URI de redirección**, en la lista de identificadores URI de redirección:
   1. En la columna **TIPO**, seleccione **Cliente público/Nativo (móvil y escritorio)** .
   1. En la columna **URI DE REDIRECCIÓN**, escriba `https://login.microsoftonline.com/common/oauth2/nativeclient`.
1. Seleccione **Registrar**.
1. Vaya a Visual Studio, abra el archivo *App.xaml.cs* y, luego, reemplace `Enter_the_Application_Id_here` en el siguiente fragmento de código por el identificador de aplicación que acaba de registrar y copiar.

    ```csharp
    private static string ClientId = "Enter_the_Application_Id_here";
    ```

## <a name="add-the-code-to-initialize-msal"></a>Adición del código para inicializar MSAL

En este caso, creará una clase para administrar la interacción con MSAL, como la administración de tokens.

1. Abra el archivo *App.xaml.cs* y, a continuación, agregue la referencia de MSAL a la clase:

    ```csharp
    using Microsoft.Identity.Client;
    ```
   <!-- Workaround for Docs conversion bug -->

2. Actualice la clase app a la siguiente:

    ```csharp
    public partial class App : Application
    {
        static App()
        {
            _clientApp = PublicClientApplicationBuilder.Create(ClientId)
                .WithAuthority(AzureCloudInstance.AzurePublic, Tenant)
                .Build();
        }

        // Below are the clientId (Application Id) of your app registration and the tenant information.
        // You have to replace:
        // - the content of ClientID with the Application Id for your app registration
        // - the content of Tenant by the information about the accounts allowed to sign-in in your application:
        //   - For Work or School account in your org, use your tenant ID, or domain
        //   - for any Work or School accounts, use `organizations`
        //   - for any Work or School accounts, or Microsoft personal account, use `common`
        //   - for Microsoft Personal account, use consumers
        private static string ClientId = "0b8b0665-bc13-4fdc-bd72-e0227b9fc011";

        private static string Tenant = "common";

        private static IPublicClientApplication _clientApp ;

        public static IPublicClientApplication PublicClientApp { get { return _clientApp; } }
    }
    ```

## <a name="create-the-application-ui"></a>Creación de la interfaz de usuario de la aplicación

En esta sección se muestra cómo puede una aplicación consultar un servidor back-end protegido como Microsoft Graph.

Debe crearse un archivo *MainWindow.xaml* automáticamente como parte de la plantilla de proyecto. Abra este archivo y reemplace el nodo *\<Grid>* de la aplicación por el código siguiente:

```xml
<Grid>
    <StackPanel Background="Azure">
        <StackPanel Orientation="Horizontal" HorizontalAlignment="Right">
            <Button x:Name="CallGraphButton" Content="Call Microsoft Graph API" HorizontalAlignment="Right" Padding="5" Click="CallGraphButton_Click" Margin="5" FontFamily="Segoe Ui"/>
            <Button x:Name="SignOutButton" Content="Sign-Out" HorizontalAlignment="Right" Padding="5" Click="SignOutButton_Click" Margin="5" Visibility="Collapsed" FontFamily="Segoe Ui"/>
        </StackPanel>
        <Label Content="API Call Results" Margin="0,0,0,-5" FontFamily="Segoe Ui" />
        <TextBox x:Name="ResultText" TextWrapping="Wrap" MinHeight="120" Margin="5" FontFamily="Segoe Ui"/>
        <Label Content="Token Info" Margin="0,0,0,-5" FontFamily="Segoe Ui" />
        <TextBox x:Name="TokenInfoText" TextWrapping="Wrap" MinHeight="70" Margin="5" FontFamily="Segoe Ui"/>
    </StackPanel>
</Grid>
```

## <a name="use-msal-to-get-a-token-for-the-microsoft-graph-api"></a>Uso de MSAL para obtener un token de Microsoft Graph API

En esta sección, usará MSAL para obtener un token de la API de Microsoft Graph.

1. En el archivo *MainWindow.xaml.cs*, añada la referencia de MSAL a la clase:

    ```csharp
    using Microsoft.Identity.Client;
    ```

2. Sustituya el código de clase `MainWindow` por lo siguiente:

    ```csharp
    public partial class MainWindow : Window
    {
        //Set the API Endpoint to Graph 'me' endpoint
        string graphAPIEndpoint = "https://graph.microsoft.com/v1.0/me";

        //Set the scope for API call to user.read
        string[] scopes = new string[] { "user.read" };


        public MainWindow()
        {
            InitializeComponent();
        }

      /// <summary>
        /// Call AcquireToken - to acquire a token requiring user to sign-in
        /// </summary>
        private async void CallGraphButton_Click(object sender, RoutedEventArgs e)
        {
            AuthenticationResult authResult = null;
            var app = App.PublicClientApp;
            ResultText.Text = string.Empty;
            TokenInfoText.Text = string.Empty;

            var accounts = await app.GetAccountsAsync();
            var firstAccount = accounts.FirstOrDefault();

            try
            {
                authResult = await app.AcquireTokenSilent(scopes, firstAccount)
                    .ExecuteAsync();
            }
            catch (MsalUiRequiredException ex)
            {
                // A MsalUiRequiredException happened on AcquireTokenSilent.
                // This indicates you need to call AcquireTokenInteractive to acquire a token
                System.Diagnostics.Debug.WriteLine($"MsalUiRequiredException: {ex.Message}");

                try
                {
                    authResult = await app.AcquireTokenInteractive(scopes)
                        .WithAccount(accounts.FirstOrDefault())
                        .WithPrompt(Prompt.SelectAccount)
                        .ExecuteAsync();
                }
                catch (MsalException msalex)
                {
                    ResultText.Text = $"Error Acquiring Token:{System.Environment.NewLine}{msalex}";
                }
            }
            catch (Exception ex)
            {
                ResultText.Text = $"Error Acquiring Token Silently:{System.Environment.NewLine}{ex}";
                return;
            }

            if (authResult != null)
            {
                ResultText.Text = await GetHttpContentWithToken(graphAPIEndpoint, authResult.AccessToken);
                DisplayBasicTokenInfo(authResult);
                this.SignOutButton.Visibility = Visibility.Visible;
            }
        }
    ```

### <a name="more-information"></a>Más información

#### <a name="get-a-user-token-interactively"></a>Obtención de un token de usuario interactivamente

La llamada a `AcquireTokenInteractive` genera una ventana que pide al usuario que inicie sesión. Las aplicaciones suelen requerir a los usuarios que inicien sesión de forma interactiva la primera vez que necesitan acceder a un recurso protegido. Es posible que también deban iniciar sesión cuando se produce un error en una operación silenciosa para adquirir un token (por ejemplo, si ha caducado la contraseña de usuario).

#### <a name="get-a-user-token-silently"></a>Obtención de un token de usuario en silencio

El método `AcquireTokenSilent` controla las renovaciones y las adquisiciones de tokens sin la interacción del usuario. Después de que `AcquireTokenInteractive` se ejecute por primera vez, `AcquireTokenSilent` es el método habitual que se utiliza para obtener los tokens empleados para acceder a recursos protegidos en las llamadas posteriores, ya que las llamadas para solicitar o renovar los tokens se realizan en modo silencioso.

En última instancia, se producirá un error en el método `AcquireTokenSilent`. El error puede deberse a que el usuario haya cerrado sesión o cambiado su contraseña en otro dispositivo. Si MSAL detecta que el problema puede solucionarse requiriendo una acción interactiva, desencadena una excepción `MsalUiRequiredException`. La aplicación puede abordar esta excepción de dos maneras:

* Puede realizar una llamada en `AcquireTokenInteractive` inmediatamente. Esta llamada provoca que se solicite al usuario que inicie sesión. Este patrón se da normalmente en aplicaciones en línea en las que no hay ningún contenido disponible sin conexión para el usuario. La aplicación de ejemplo generada en esta instalación guiada utiliza este patrón, que puede verse en acción la primera vez que se ejecuta la aplicación.

* Dado que ningún usuario ha usado la aplicación, `PublicClientApp.Users.FirstOrDefault()` contendrá un valor NULL y se iniciará una excepción `MsalUiRequiredException`.

* El código del ejemplo trata entonces la excepción llamando a `AcquireTokenInteractive`, lo que provoca que se pida al usuario que inicie sesión.

* En su lugar, puede presentar una indicación visual a los usuarios para señalar que es necesario iniciar sesión de manera interactiva, lo que les permitirá escoger el momento oportuno para iniciar sesión. Asimismo, la aplicación puede volver a probar el método `AcquireTokenSilent` más tarde. Este patrón se utiliza con frecuencia cuando los usuarios pueden usar otras funciones de aplicación sin interrupciones; por ejemplo, cuando hay contenido sin conexión disponible en la aplicación. En este caso, los usuarios pueden decidir cuándo desean iniciar sesión para acceder al recurso protegido o para actualizar la información obsoleta. Como alternativa, la aplicación puede decidir probar el método `AcquireTokenSilent` de nuevo cuando se restablezca la red después de no haber estado disponible temporalmente.

## <a name="call-the-microsoft-graph-api-by-using-the-token-you-just-obtained"></a>Llamada a Microsoft Graph API con el token que acaba de obtener

Añada el siguiente método nuevo a su archivo `MainWindow.xaml.cs`. El método se utiliza para realizar una solicitud de `GET` a Graph API con un encabezado de autorización:

```csharp
/// <summary>
/// Perform an HTTP GET request to a URL using an HTTP Authorization header
/// </summary>
/// <param name="url">The URL</param>
/// <param name="token">The token</param>
/// <returns>String containing the results of the GET operation</returns>
public async Task<string> GetHttpContentWithToken(string url, string token)
{
    var httpClient = new System.Net.Http.HttpClient();
    System.Net.Http.HttpResponseMessage response;
    try
    {
        var request = new System.Net.Http.HttpRequestMessage(System.Net.Http.HttpMethod.Get, url);
        //Add the token in Authorization header
        request.Headers.Authorization = new System.Net.Http.Headers.AuthenticationHeaderValue("Bearer", token);
        response = await httpClient.SendAsync(request);
        var content = await response.Content.ReadAsStringAsync();
        return content;
    }
    catch (Exception ex)
    {
        return ex.ToString();
    }
}
```

### <a name="more-information-about-making-a-rest-call-against-a-protected-api"></a>Más información acerca de cómo realizar una llamada de REST a una API protegida

En esta aplicación de ejemplo, el método `GetHttpContentWithToken` se usa para realizar una solicitud HTTP `GET` a un recurso protegido que requiere un token y, a continuación, devolver el contenido al autor de la llamada. Este método agrega el token adquirido al encabezado de autorización HTTP. En este ejemplo, el recurso es el punto de conexión *me* de Microsoft Graph API, que muestra información del perfil del usuario.

## <a name="add-a-method-to-sign-out-a-user"></a>Adición de un método para cerrar la sesión de un usuario

Añada el método siguiente a su archivo `MainWindow.xaml.cs` para cerrar la sesión de un usuario:

```csharp
/// <summary>
/// Sign out the current user
/// </summary>
private async void SignOutButton_Click(object sender, RoutedEventArgs e)
{
    var accounts = await App.PublicClientApp.GetAccountsAsync();

    if (accounts.Any())
    {
        try
        {
            await App.PublicClientApp.RemoveAsync(accounts.FirstOrDefault());
            this.ResultText.Text = "User has signed-out";
            this.CallGraphButton.Visibility = Visibility.Visible;
            this.SignOutButton.Visibility = Visibility.Collapsed;
        }
        catch (MsalException ex)
        {
            ResultText.Text = $"Error signing-out user: {ex.Message}";
        }
    }
}
```

### <a name="more-information-about-user-sign-out"></a>Más información sobre cerrar la sesión de un usuario

El método `SignOutButton_Click` anterior quita a los usuarios de la caché de usuario MSAL, lo que de hecho indicará a MSAL que olvide al usuario actual de forma que una solicitud futura para adquirir un token solo se completará correctamente si se realiza para ser interactiva.

Aunque la aplicación en este ejemplo es compatible con usuarios individuales, MSAL admite escenarios en los que es posible iniciar sesión en varias cuentas al mismo tiempo. Un ejemplo de esto es una aplicación de correo electrónico donde un usuario tiene varias cuentas.

## <a name="display-basic-token-information"></a>Visualización de información de token básica

Para mostrar información básica sobre el token, añada el método siguiente a su archivo *MainWindow.xaml.cs*:

```csharp
/// <summary>
/// Display basic information contained in the token
/// </summary>
private void DisplayBasicTokenInfo(AuthenticationResult authResult)
{
    TokenInfoText.Text = "";
    if (authResult != null)
    {
        TokenInfoText.Text += $"Username: {authResult.Account.Username}" + Environment.NewLine;
        TokenInfoText.Text += $"Token Expires: {authResult.ExpiresOn.ToLocalTime()}" + Environment.NewLine;
    }
}
```

### <a name="more-information"></a>Más información

Además del token de acceso que se usa para llamar a la API de Microsoft Graph, después de que el usuario inicie sesión, MSAL también obtiene un token de identificador. Este token contiene un pequeño subconjunto de información que es pertinente para los usuarios. El método `DisplayBasicTokenInfo` muestra la información básica que contiene el token. Por ejemplo, muestra el nombre para mostrar del usuario y su identificador, así como la fecha de expiración del token y la cadena que representa al propio token de acceso. Puede seleccionar varias veces el botón *Call Microsoft Graph API* (Llamar a la API de Microsoft Graph) y ver que el mismo token se reutilizó para solicitudes posteriores. También puede ver que la fecha de expiración se amplía si MSAL decide que es el momento de renovar el token.

[!INCLUDE [5. Test and Validate](../../../includes/active-directory-develop-guidedsetup-windesktop-test.md)]
