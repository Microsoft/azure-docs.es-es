---
title: Guía para la autenticación de clientes de Azure SignalR Service
description: Obtenga información sobre cómo implementar su propia autenticación e integrarla con Azure SignalR Service siguiendo el ejemplo e2e.
author: sffamily
ms.service: signalr
ms.topic: conceptual
ms.date: 11/13/2019
ms.author: zhshang
ms.custom: devx-track-javascript, devx-track-csharp
ms.openlocfilehash: da0c3284b42be6d9054b7386839004b57d8ca274
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88935745"
---
# <a name="azure-signalr-service-authentication"></a>Autenticación de Azure SignalR Service

Este tutorial se basa en la aplicación de salón de chat presentada en la guía de inicio rápido. Si no ha realizado [Creación de un salón de chat con el servicio SignalR](signalr-quickstart-dotnet-core.md), lleve a cabo ese ejercicio primero.

En este tutorial, aprenderá a implementar su propia autenticación y a integrarla con el servicio Microsoft Azure SignalR.

La autenticación utilizada inicialmente en la aplicación de salón de chat de la guía de inicio rápido es demasiado simple para escenarios del mundo real. La aplicación permite que cada cliente notifique quién es y el servidor lo acepta simplemente. Este enfoque no es muy útil en aplicaciones del mundo real, donde un usuario no autorizado podría suplantar a otros usuarios para acceder a información confidencial.

[GitHub](https://github.com/) proporciona API de autenticación basadas en un protocolo estándar del sector muy popular llamado [OAuth](https://oauth.net/). Estas API permiten a aplicaciones de terceros autenticar cuentas de GitHub. En este tutorial, usará estas API para implementar la autenticación mediante una cuenta de Github antes de permitir inicios de sesión de clientes en la aplicación de salón de chat. Después de autenticar una cuenta de GitHub, la información de la cuenta se agregará como una cookie que será utilizada por el cliente web para la autenticación.

Para más información sobre las API de autenticación de OAuth proporcionadas a través de GitHub, consulte [Conceptos básicos de autenticación](https://developer.github.com/v3/guides/basics-of-authentication/).

Puede usar cualquier editor de código para realizar los pasos de esta guía de inicio rápido. Sin embargo, [Visual Studio Code](https://code.visualstudio.com/) es una excelente opción disponible en las plataformas Windows, macOS y Linux.

El código de este tutorial está disponible para su descarga en el [repositorio de GitHub AzureSignalR-samples](https://github.com/aspnet/AzureSignalR-samples/tree/master/samples/GitHubChat).

![OAuth completo hospedado en Azure](media/signalr-concept-authenticate-oauth/signalr-oauth-complete-azure.png)

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Registrar una nueva aplicación de OAuth con su cuenta de GitHub
> * Agregar un controlador de autenticación para admitir la autenticación de GitHub
> * Implementar la aplicación web de ASP.NET Core en Azure

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Requisitos previos

Para realizar este tutorial, debe disponer de los siguientes requisitos previos:

* Una cuenta creada en [GitHub](https://github.com/)
* [Git](https://git-scm.com/)
* [SDK de .NET Core](https://www.microsoft.com/net/download/windows)
* [Azure Cloud Shell configurado](https://docs.microsoft.com/azure/cloud-shell/quickstart)
* Descargue o clone el repositorio de GitHub [AzureSignalR-sample](https://github.com/aspnet/AzureSignalR-samples).

## <a name="create-an-oauth-app"></a>Creación de una aplicación de OAuth

1. Abra un explorador web, vaya a `https://github.com` e inicie sesión en su cuenta.

2. En su cuenta, vaya a **Settings** (Configuración)  > **Developer settings** (Opciones del desarrollador) y haga clic en **Register a new application** (Registrar una nueva aplicación) o bien vaya a **New OAuth App** (Nueva aplicación de OAuth) en *OAuth Apps* (Aplicaciones de OAuth).

3. Utilice la siguiente configuración para la nueva aplicación de OAuth y, a continuación, haga clic en **Register application** (Registrar aplicación):

    | Nombre del valor | Valor sugerido | Descripción |
    | ------------ | --------------- | ----------- |
    | Nombre de la aplicación | *Azure SignalR Chat* | El usuario de GitHub debe ser capaz de reconocer y confiar en la aplicación con la que se autentica.   |
    | Dirección URL de la página principal | `http://localhost:5000/home` | |
    | Descripción de la aplicación | *Un ejemplo de salón de chat con Azure SignalR Service y la autenticación de GitHub* | Una descripción útil de la aplicación que ayudará a los usuarios de la aplicación a entender el contexto de la autenticación que se va a usar. |
    | Dirección URL de devolución de llamada de autorización | `http://localhost:5000/signin-github` | Esta configuración es la más importante para la aplicación de OAuth. Es la dirección URL de devolución de llamada que GitHub devuelve al usuario después de una autenticación correcta. En este tutorial, debe usar la dirección URL de devolución de llamada predeterminada para el paquete *AspNet.Security.OAuth.GitHub*, */signin-github*.  |

4. Una vez completado el nuevo registro de aplicación de OAuth, agregue el *Identificador de cliente* y el *Secreto de cliente* al administrador de secretos utilizando los comandos siguientes. Reemplace *Your_GitHub_Client_Id* y *Your_GitHub_Client_Secret* por los valores de la aplicación de OAuth.

    ```dotnetcli
    dotnet user-secrets set GitHubClientId Your_GitHub_Client_Id
    dotnet user-secrets set GitHubClientSecret Your_GitHub_Client_Secret
    ```

## <a name="implement-the-oauth-flow"></a>Implementación del flujo de OAuth

### <a name="update-the-startup-class-to-support-github-authentication"></a>Actualización de la clase Startup para admitir la autenticación de GitHub

1. Agregue una referencia a los paquetes *Microsoft.AspNetCore.Authentication.Cookies* y *AspNet.Security.OAuth.GitHub* más recientes y restaure todos los paquetes.

    ```dotnetcli
    dotnet add package Microsoft.AspNetCore.Authentication.Cookies -v 2.1.0-rc1-30656
    dotnet add package AspNet.Security.OAuth.GitHub -v 2.0.0-rc2-final
    dotnet restore
    ```

1. Abra *Startup.cs* y agregue instrucciones `using` para los espacios de nombres siguientes:

    ```csharp
    using System.Net.Http;
    using System.Net.Http.Headers;
    using System.Security.Claims;
    using Microsoft.AspNetCore.Authentication.Cookies;
    using Microsoft.AspNetCore.Authentication.OAuth;
    using Newtonsoft.Json.Linq;
    ```

2. En la parte superior de la clase `Startup`, agregue constantes para las claves del administrador de secretos que contiene los secretos de la aplicación de OAuth de GitHub.

    ```csharp
    private const string GitHubClientId = "GitHubClientId";
    private const string GitHubClientSecret = "GitHubClientSecret";
    ```

3. Agregue el código siguiente al método `ConfigureServices` para admitir la autenticación con la aplicación de OAuth de GitHub:

    ```csharp
    services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
        .AddCookie()
        .AddGitHub(options =>
        {
            options.ClientId = Configuration[GitHubClientId];
            options.ClientSecret = Configuration[GitHubClientSecret];
            options.Scope.Add("user:email");
            options.Events = new OAuthEvents
            {
                OnCreatingTicket = GetUserCompanyInfoAsync
            };
        });
    ```

4. Agregue el método auxiliar `GetUserCompanyInfoAsync` a la clase `Startup`.

    ```csharp
    private static async Task GetUserCompanyInfoAsync(OAuthCreatingTicketContext context)
    {
        var request = new HttpRequestMessage(HttpMethod.Get, context.Options.UserInformationEndpoint);
        request.Headers.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
        request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", context.AccessToken);

        var response = await context.Backchannel.SendAsync(request,
            HttpCompletionOption.ResponseHeadersRead, context.HttpContext.RequestAborted);

        var user = JObject.Parse(await response.Content.ReadAsStringAsync());
        if (user.ContainsKey("company"))
        {
            var company = user["company"].ToString();
            var companyIdentity = new ClaimsIdentity(new[]
            {
                new Claim("Company", company)
            });
            context.Principal.AddIdentity(companyIdentity);
        }
    }
    ```

5. Actualice el método `Configure` de la clase Startup con la siguiente línea de código y guarde el archivo.

    ```csharp
    app.UseAuthentication();
    ```

### <a name="add-an-authentication-controller"></a>Adición de un controlador de autenticación

En esta sección, implementará una API `Login` que autentica a los clientes mediante la aplicación de OAuth de GitHub. Una vez autenticado, la API agregará una cookie a la respuesta al cliente web antes de redirigir al cliente de vuelta a la aplicación de chat. A continuación, se utilizará dicha cookie para identificar al cliente.

1. Agregue un nuevo archivo de código de controlador al directorio *chattest\Controllers*. Llame al archivo *AuthController.cs*.

2. Agregue el código siguiente para el controlador de autenticación. Asegúrese de actualizar el espacio de nombres si el directorio del proyecto no era *chattest*:

    ```csharp
    using AspNet.Security.OAuth.GitHub;
    using Microsoft.AspNetCore.Authentication;
    using Microsoft.AspNetCore.Mvc;

    namespace chattest.Controllers
    {
        [Route("/")]
        public class AuthController : Controller
        {
            [HttpGet("login")]
            public IActionResult Login()
            {
                if (!User.Identity.IsAuthenticated)
                {
                    return Challenge(GitHubAuthenticationDefaults.AuthenticationScheme);
                }

                HttpContext.Response.Cookies.Append("githubchat_username", User.Identity.Name);
                HttpContext.SignInAsync(User);
                return Redirect("/");
            }
        }
    }
    ```

3. Guarde los cambios.

### <a name="update-the-hub-class"></a>Actualización de la clase Hub

De forma predeterminada, cuando un cliente web intenta conectarse al servicio SignalR, la conexión se otorga en función de un token de acceso que se proporciona de forma interna. Este token de acceso no está asociado a una identidad autenticada. Este acceso es realmente anónimo.

En esta sección, se activará la autenticación real mediante la adición del atributo `Authorize` a la clase del concentrador y la actualización de los métodos del concentrador para leer el nombre de usuario de la notificación del usuario autenticado.

1. Abra *Hub\Chat.cs* y agregue referencias a estos espacios de nombres:

    ```csharp
    using System.Threading.Tasks;
    using Microsoft.AspNetCore.Authorization;
    ```

2. Actualice el código del concentrador tal y como se muestra a continuación. Este código agrega el atributo `Authorize` a la clase `Chat` y utiliza la identidad del usuario autenticado en los métodos de concentrador. Además, se agrega el método `OnConnectedAsync`, que registrará un mensaje del sistema en el salón de chat cada vez que se conecte un cliente nuevo.

    ```csharp
    [Authorize]
    public class Chat : Hub
    {
        public override Task OnConnectedAsync()
        {
            return Clients.All.SendAsync("broadcastMessage", "_SYSTEM_", $"{Context.User.Identity.Name} JOINED");
        }

        // Uncomment this line to only allow user in Microsoft to send message
        //[Authorize(Policy = "Microsoft_Only")]
        public void BroadcastMessage(string message)
        {
            Clients.All.SendAsync("broadcastMessage", Context.User.Identity.Name, message);
        }

        public void Echo(string message)
        {
            var echoMessage = $"{message} (echo from server)";
            Clients.Client(Context.ConnectionId).SendAsync("echo", Context.User.Identity.Name, echoMessage);
        }
    }
    ```

3. Guarde los cambios.

### <a name="update-the-web-client-code"></a>Actualización del código del cliente web

1. Abra *wwwroot\index.html* y reemplace el código que solicita el nombre de usuario por el código para usar la cookie devuelta por el controlador de autenticación.

    Elimine el siguiente código de *index.html*:

    ```javascript
    // Get the user name and store it to prepend to messages.
    var username = generateRandomName();
    var promptMessage = 'Enter your name:';
    do {
        username = prompt(promptMessage, username);
        if (!username || username.startsWith('_') || username.indexOf('<') > -1 || username.indexOf('>') > -1) {
            username = '';
            promptMessage = 'Invalid input. Enter your name:';
        }
    } while(!username)
    ```

    Agregue el código siguiente en lugar del código anterior para usar la cookie:

    ```javascript
    // Get the user name cookie.
    function getCookie(key) {
        var cookies = document.cookie.split(';').map(c => c.trim());
        for (var i = 0; i < cookies.length; i++) {
            if (cookies[i].startsWith(key + '=')) return unescape(cookies[i].slice(key.length + 1));
        }
        return '';
    }
    var username = getCookie('githubchat_username');
    ```

2. Justo debajo de la línea de código que agregó para usar la cookie, agregue la siguiente definición para la función `appendMessage`:

    ```javascript
    function appendMessage(encodedName, encodedMsg) {
        var messageEntry = createMessageEntry(encodedName, encodedMsg);
        var messageBox = document.getElementById('messages');
        messageBox.appendChild(messageEntry);
        messageBox.scrollTop = messageBox.scrollHeight;
    }
    ```

3. Actualice las funciones `bindConnectionMessage` y `onConnected` con el siguiente código para usar `appendMessage`.

    ```javascript
    function bindConnectionMessage(connection) {
        var messageCallback = function(name, message) {
            if (!message) return;
            // Html encode display name and message.
            var encodedName = name;
            var encodedMsg = message.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;");
            appendMessage(encodedName, encodedMsg);
        };
        // Create a function that the hub can call to broadcast messages.
        connection.on('broadcastMessage', messageCallback);
        connection.on('echo', messageCallback);
        connection.onclose(onConnectionError);
    }

    function onConnected(connection) {
        console.log('connection started');
        document.getElementById('sendmessage').addEventListener('click', function (event) {
            // Call the broadcastMessage method on the hub.
            if (messageInput.value) {
                connection
                    .invoke('broadcastMessage', messageInput.value)
                    .catch(e => appendMessage('_BROADCAST_', e.message));
            }

            // Clear text box and reset focus for next comment.
            messageInput.value = '';
            messageInput.focus();
            event.preventDefault();
        });
        document.getElementById('message').addEventListener('keypress', function (event) {
            if (event.keyCode === 13) {
                event.preventDefault();
                document.getElementById('sendmessage').click();
                return false;
            }
        });
        document.getElementById('echo').addEventListener('click', function (event) {
            // Call the echo method on the hub.
            connection.send('echo', messageInput.value);

            // Clear text box and reset focus for next comment.
            messageInput.value = '';
            messageInput.focus();
            event.preventDefault();
        });
    }
    ```

4. En la parte inferior de *index.html*, actualice el controlador de errores para `connection.start()` tal y como se muestra a continuación para solicitar al usuario que inicie sesión.

    ```javascript
    connection.start()
        .then(function () {
            onConnected(connection);
        })
        .catch(function (error) {
            if (error) {
                if (error.message) {
                    console.error(error.message);
                }
                if (error.statusCode && error.statusCode === 401) {
                    appendMessage('_BROADCAST_', 'You\'re not logged in. Click <a href="/login">here</a> to login with GitHub.');
                }
            }
        });
    ```

5. Guarde los cambios.

## <a name="build-and-run-the-app-locally"></a>Compilación y ejecución de la aplicación localmente

1. Guarde los cambios en todos los archivos.

2. Para compilar la aplicación mediante la CLI de .NET Core, ejecute el siguiente comando en el shell de comandos:

    ```dotnetcli
    dotnet build
    ```

3. Una vez que la compilación se complete correctamente, ejecute el siguiente comando para ejecutar la aplicación web localmente:

    ```dotnetcli
    dotnet run
    ```

    De forma predeterminada, la aplicación se hospeda localmente en el puerto 5000:

    ```output
    E:\Testing\chattest>dotnet run
    Hosting environment: Production
    Content root path: E:\Testing\chattest
    Now listening on: http://localhost:5000
                    Application started. Press Ctrl+C to shut down.
    ```

4. Inicie una ventana del explorador y vaya a `http://localhost:5000`. Haga clic en el vínculo **aquí** situado en la parte superior para iniciar sesión con GitHub.

    ![OAuth completo hospedado en Azure](media/signalr-concept-authenticate-oauth/signalr-oauth-complete-azure.png)

    Se le pedirá autorizar el acceso de la aplicación de chat a su cuenta de GitHub. Haga clic en el botón **Autorizar**.

    ![Autorizar la aplicación de OAuth](media/signalr-concept-authenticate-oauth/signalr-authorize-oauth-app.png)

    Se le redirige de nuevo a la aplicación de chat y se inicia sesión con su nombre de cuenta de GitHub. La aplicación web ha determinado el nombre de cuenta mediante la autenticación con la nueva autenticación que agregó.

    ![Cuenta identificada](media/signalr-concept-authenticate-oauth/signalr-oauth-account-identified.png)

    Ahora que la aplicación de chat realiza la autenticación con GitHub y almacena la información de autenticación como cookies, debe implementarla en Azure para que otros usuarios puedan autenticarse con sus cuentas y comunicarse desde otras estaciones de trabajo.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="deploy-the-app-to-azure"></a>Implementar la aplicación en Azure

En esta sección, utilizará la interfaz de la línea de comandos (CLI) de Azure desde Azure Cloud Shell para crear una nueva aplicación web de [Azure App Service](https://docs.microsoft.com/azure/app-service/) para hospedar la aplicación de ASP.NET en Azure. La aplicación web se configurará para usar la implementación de Git local. La aplicación web también se configurará con la cadena de conexión de SignalR, los secretos de la aplicación de OAuth de GitHub y un usuario de implementación.

En los pasos descritos en esta sección se usa la extensión *signalr* para la CLI de Azure. Ejecute el siguiente comando para instalar la extensión *signalr* para la versión de la CLI de Azure:

```azurecli-interactive
az extension add -n signalr
```

Al crear los recursos siguientes, asegúrese de usar el mismo grupo de recursos en el que reside el recurso del servicio SignalR. Este enfoque hará la limpieza mucho más fácil cuando posteriormente desee eliminar todos los recursos. En los ejemplos se da por supuesto que usa el nombre de grupo recomendado en los tutoriales anteriores, *SignalRTestResources*.

### <a name="create-the-web-app-and-plan"></a>Creación de la aplicación web y el plan

Copie el texto de los comandos siguientes y actualice los parámetros. Pegue el script actualizado en Azure Cloud Shell y presione **ENTRAR** para crear un nuevo plan y una aplicación web de App Service.

```azurecli-interactive
#========================================================================
#=== Update these variable for your resource group name.              ===
#========================================================================
ResourceGroupName=SignalRTestResources

#========================================================================
#=== Update these variable for your web app.                          ===
#========================================================================
WebAppName=myWebAppName
WebAppPlan=myAppServicePlanName

# Create an App Service plan.
az appservice plan create --name $WebAppPlan --resource-group $ResourceGroupName \
    --sku FREE

# Create the new Web App
az webapp create --name $WebAppName --resource-group $ResourceGroupName \
    --plan $WebAppPlan
```

| Parámetro | Descripción |
| -------------------- | --------------- |
| ResourceGroupName | Este nombre de grupo de recursos se sugirió en tutoriales anteriores. Es una buena idea para mantener agrupados todos juntos los recursos de los tutoriales. Use el mismo grupo de recursos que utilizó en los tutoriales anteriores. |
| WebAppPlan | Escriba un nombre nuevo y único del plan de App Service. |
| WebAppName | Este será el nombre de la nueva aplicación web y parte de la dirección URL. Utilice un nombre único. Por ejemplo, signalrtestwebapp22665120.   |

### <a name="add-app-settings-to-the-web-app"></a>Adición de la configuración de la aplicación a la aplicación web

En esta sección, agregará la configuración de la aplicación de los siguientes componentes:

* Cadena de conexión de los recursos del servicio SignalR
* Identificador de cliente de la aplicación de OAuth de GitHub
* Secreto de cliente de la aplicación de OAuth de GitHub

Copie el texto de los comandos siguientes y actualice los parámetros. Pegue el script actualizado en Azure Cloud Shell y presione **ENTRAR** para agregar la configuración de la aplicación:

```azurecli-interactive
#========================================================================
#=== Update these variables for your GitHub OAuth App.                ===
#========================================================================
GitHubClientId=1234567890
GitHubClientSecret=1234567890

#========================================================================
#=== Update these variables for your resources.                       ===
#========================================================================
ResourceGroupName=SignalRTestResources
SignalRServiceResource=mySignalRresourcename
WebAppName=myWebAppName

# Get the SignalR primary connection string 
primaryConnectionString=$(az signalr key list --name $SignalRServiceResource \
  --resource-group $ResourceGroupName --query primaryConnectionString -o tsv)

#Add an app setting to the web app for the SignalR connection
az webapp config appsettings set --name $WebAppName \
    --resource-group $ResourceGroupName \
    --settings "Azure__SignalR__ConnectionString=$primaryConnectionString"

#Add the app settings to use with GitHub authentication
az webapp config appsettings set --name $WebAppName \
    --resource-group $ResourceGroupName \
    --settings "GitHubClientId=$GitHubClientId"
az webapp config appsettings set --name $WebAppName \
    --resource-group $ResourceGroupName \
    --settings "GitHubClientSecret=$GitHubClientSecret"
```

| Parámetro | Descripción |
| -------------------- | --------------- |
| GitHubClientId | Asigne a esta variable el identificador de cliente secreto de la aplicación de OAuth de GitHub. |
| GitHubClientSecret | Asigne a esta variable la contraseña secreta de la aplicación de OAuth de GitHub. |
| ResourceGroupName | Actualice esta variable para que sea el mismo nombre de grupo de recursos que usó en la sección anterior. |
| SignalRServiceResource | Actualice esta variable con el nombre del recurso del servicio SignalR que creó en la guía de inicio rápido. Por ejemplo, signalrtestsvc48778624. |
| WebAppName | Actualice esta variable con el nombre de la nueva aplicación web que creó en la sección anterior. |

### <a name="configure-the-web-app-for-local-git-deployment"></a>Configuración de la aplicación web para la implementación de Git local

En Azure Cloud Shell, pegue el siguiente script. Este script crea un nuevo nombre de usuario de implementación y una contraseña que va a utilizar al implementar el código en la aplicación web con Git. El script también configura la aplicación web para la implementación con un repositorio de Git local y devuelve la dirección URL de implementación de Git.

```azurecli-interactive
#========================================================================
#=== Update these variables for your resources.                       ===
#========================================================================
ResourceGroupName=SignalRTestResources
WebAppName=myWebAppName

#========================================================================
#=== Update these variables for your deployment user.                 ===
#========================================================================
DeploymentUserName=myUserName
DeploymentUserPassword=myPassword

# Add the desired deployment user name and password
az webapp deployment user set --user-name $DeploymentUserName \
    --password $DeploymentUserPassword

# Configure Git deployment and note the deployment URL in the output
az webapp deployment source config-local-git --name $WebAppName \
    --resource-group $ResourceGroupName \
    --query [url] -o tsv
```

| Parámetro | Descripción |
| -------------------- | --------------- |
| DeploymentUserName | Elija un nuevo nombre de usuario de implementación. |
| DeploymentUserPassword | Elija una contraseña para el nuevo usuario de implementación. |
| ResourceGroupName | Utilice el mismo nombre de grupo de recursos que usó en la sección anterior. |
| WebAppName | Este será el nombre de la nueva aplicación web que creó anteriormente. |

Anote la dirección URL de implementación de Git que devuelve este comando. Más adelante usará esta dirección URL.

### <a name="deploy-your-code-to-the-azure-web-app"></a>Implementación del código en la aplicación web de Azure

Para implementar el código, ejecute los comandos siguientes en un shell de Git.

1. Vaya a la raíz del directorio del proyecto. Si no ha inicializado el proyecto con un repositorio Git, ejecute el comando siguiente:

    ```bash
    git init
    ```

2. Agregue un remoto para la dirección URL de implementación de Git que anotó anteriormente:

    ```bash
    git remote add Azure <your git deployment url>
    ```

3. Almacene provisionalmente todos los archivos en el repositorio inicializado y agregue una instrucción commit.

    ```bash
    git add -A
    git commit -m "init commit"
    ```

4. Implemente el código en la aplicación web en Azure.

    ```bash
    git push Azure master
    ```

    Se le pedirá autenticarse para implementar el código en Azure. Escriba el nombre de usuario y la contraseña del usuario de implementación que creó anteriormente.

### <a name="update-the-github-oauth-app"></a>Actualización de la aplicación de OAuth de GitHub

El último paso que debe realizar es actualizar la **dirección URL de la página principal** y la **dirección URL de devolución de llamada de autorización** de la aplicación de OAuth de GitHub para que apunte a la nueva aplicación hospedada.

1. Abra [https://github.com](https://github.com) en un explorador y, en la cuenta, vaya a **Settings** (Configuración)  > **Developer settings** (Opciones del desarrollador)  > **Oauth Apps** (Aplicaciones de Oauth).

2. Haga clic en la aplicación de autenticación y actualice la **dirección URL de la página principal** y la **dirección URL de devolución de llamada de autorización** tal y como se muestra a continuación:

    | Configuración | Ejemplo |
    | ------- | ------- |
    | Dirección URL de la página principal | `https://signalrtestwebapp22665120.azurewebsites.net/home` |
    | Dirección URL de devolución de llamada de autorización | `https://signalrtestwebapp22665120.azurewebsites.net/signin-github` |

3. Vaya a la dirección URL de la aplicación web y pruebe la aplicación.

    ![OAuth completo hospedado en Azure](media/signalr-concept-authenticate-oauth/signalr-oauth-complete-azure.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Si va a seguir con el tutorial siguiente, puede mantener los recursos creados en esta guía de inicio rápido y volverlos a utilizar con el tutorial siguiente.

En caso contrario, si ya ha terminado con la aplicación de ejemplo de la guía de inicio rápido, puede eliminar los recursos de Azure creados en este tutorial para evitar cargos.

> [!IMPORTANT]
> La eliminación de un grupo de recursos es irreversible y el grupo de recursos y todos los recursos que contiene se eliminarán de forma permanente. Asegúrese de no eliminar por accidente el grupo de recursos o los recursos equivocados. Si ha creado los recursos para hospedar este ejemplo dentro de un grupo de recursos existente que contiene recursos que desea mantener, puede eliminar cada recurso individualmente de sus hojas respectivas, en lugar de eliminar el grupo de recursos.

Inicie sesión en [Azure Portal](https://portal.azure.com) y haga clic en **Grupos de recursos**.

Escriba el nombre del grupo de recursos en el cuadro de texto **Filtrar por nombre...** . En las instrucciones de este artículo se usa un grupo de recursos llamado *SignalRTestResources*. En el grupo de recursos de la lista de resultados, haga clic en **...** y, a continuación, en **Eliminar grupo de recursos**.

![Eliminar](./media/signalr-concept-authenticate-oauth/signalr-delete-resource-group.png)

Se le pedirá que confirme la eliminación del grupo de recursos. Escriba el nombre del grupo de recursos para confirmar y haga clic en **Eliminar**.

Transcurridos unos instantes, el grupo de recursos y todos los recursos que contiene se eliminan.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha agregado la autenticación con OAuth para proporcionar un enfoque más adecuado para la autenticación con el servicio Azure SignalR. Para más información sobre el uso de Azure SignalR Server, continúe con los ejemplos de la CLI de Azure para SignalR Service.

> [!div class="nextstepaction"]
> [Ejemplos de la CLI de Azure SignalR](./signalr-reference-cli.md)