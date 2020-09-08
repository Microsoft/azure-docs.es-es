---
title: 'Tutorial: Autenticación de usuarios E2E'
description: Aprenda a usar la autenticación y autorización de App Service para proteger sus aplicaciones de App Service de un extremo a otro, incluido el acceso a las API remotas.
keywords: app service, azure app service, authN, authZ, secure, security, multi-tiered, azure active directory, azure ad
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 04/29/2020
ms.custom: devx-track-csharp, seodec18
zone_pivot_groups: app-service-platform-windows-linux
ms.openlocfilehash: c1c3f52dafe63e3f829eb12d4fb872ed3ce85f36
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88211713"
---
# <a name="tutorial-authenticate-and-authorize-users-end-to-end-in-azure-app-service"></a>Tutorial: Autenticación y autorización de usuarios de extremo a extremo en Azure App Service

::: zone pivot="platform-windows"  

[Azure App Service](overview.md) proporciona un servicio de hospedaje web muy escalable y con aplicación de revisiones de un modo automático. Además, App Service incluye compatibilidad integrada con la [autenticación y autorización de usuarios](overview-authentication-authorization.md). En este tutorial se muestra cómo proteger las aplicaciones con la autenticación y autorización de App Service. Usa una aplicación ASP.NET Core con un front-end Angular.js como ejemplo. La autenticación y autorización de App Service admiten entornos de ejecución de todos los lenguajes; en este tutorial, aprenderá a aplicarlas a su lenguaje preferido.

::: zone-end

::: zone pivot="platform-linux"

[Azure App Service](overview.md) proporciona un servicio de hospedaje web muy escalable y con aplicación automática de revisiones con el sistema operativo Linux. Además, App Service incluye compatibilidad integrada con la [autenticación y autorización de usuarios](overview-authentication-authorization.md). En este tutorial se muestra cómo proteger las aplicaciones con la autenticación y autorización de App Service. Se usa una aplicación de ASP.NET Core con un front-end Angular.js, pero solo como ejemplo. La autenticación y autorización de App Service admiten entornos de ejecución de todos los lenguajes; en este tutorial, aprenderá a aplicarlas a su lenguaje preferido.

::: zone-end

![Autenticación y autorización simples](./media/tutorial-auth-aad/simple-auth.png)

También se muestra cómo proteger una aplicación de niveles múltiples accediendo a una API back-end protegida en nombre del usuario autenticado, tanto [desde el código de servidor](#call-api-securely-from-server-code) como [desde el código de explorador](#call-api-securely-from-browser-code).

![Autenticación y autorización avanzadas](./media/tutorial-auth-aad/advanced-auth.png)

Estos son solo algunos de los posibles escenarios de autenticación y autorización de App Service. 

Esta es una lista más completa de lo que aprenderá en el tutorial:

> [!div class="checklist"]
> * Habilitación de la autenticación y autorización integradas
> * Protección de las aplicaciones frente a las solicitudes no autenticadas
> * Uso de Azure Active Directory como proveedor de identidades
> * Acceso a una aplicación remota en nombre del usuario que inició sesión
> * Protección de llamadas de servicio a servicio con autenticación de tokens
> * Uso de tokens de acceso desde el código de servidor
> * Uso de tokens de acceso desde el código de cliente (explorador)

También puede seguir los pasos de este tutorial en macOS, Linux, Windows.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial:

* <a href="https://git-scm.com/" target="_blank">Instalación de Git</a>
* <a href="https://dotnet.microsoft.com/download/dotnet-core/3.1" target="_blank">Instalación del SDK más reciente de .NET Core 3.1</a>

## <a name="create-local-net-core-app"></a>Creación de una aplicación .NET Core local

En este paso, configurará el proyecto .NET Core local. Usará el mismo proyecto para implementar una aplicación de API de back-end y una aplicación web de front-end.

### <a name="clone-and-run-the-sample-application"></a>Clonación y ejecución de la aplicación de ejemplo

Ejecute los comandos siguientes para clonar el repositorio de ejemplo y ejecutarlo.

```bash
git clone https://github.com/Azure-Samples/dotnet-core-api
cd dotnet-core-api
dotnet run
```

Vaya a `http://localhost:5000` e intente agregar, modificar y quitar elementos de lista de tareas. 

![API de ASP.NET Core en ejecución local](./media/tutorial-auth-aad/local-run.png)

Para detener ASP.NET Core en cualquier momento, presione `Ctrl+C` en el terminal.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="deploy-apps-to-azure"></a>Implementación de aplicaciones en Azure

En este paso, implementará el proyecto en dos aplicaciones de App Service. Una es la aplicación de front-end y la otra es la aplicación de back-end.

### <a name="configure-a-deployment-user"></a>Configuración de un usuario de implementación

[!INCLUDE [Configure deployment user](../../includes/configure-deployment-user-no-h.md)]

### <a name="create-azure-resources"></a>Creación de recursos de Azure

::: zone pivot="platform-windows"  

En Cloud Shell, ejecute los siguientes comandos para crear dos aplicaciones web en Windows. Reemplace _\<front-end-app-name>_ y _\<back-end-app-name>_ por dos nombres de aplicación globalmente únicos (los caracteres válidos son `a-z`, `0-9` y `-`). Para más información sobre cada comando, consulte [API RESTful con CORS en Azure App Service](app-service-web-tutorial-rest-api.md).

```azurecli-interactive
az group create --name myAuthResourceGroup --location "West Europe"
az appservice plan create --name myAuthAppServicePlan --resource-group myAuthResourceGroup --sku FREE
az webapp create --resource-group myAuthResourceGroup --plan myAuthAppServicePlan --name <front-end-app-name> --deployment-local-git --query deploymentLocalGitUrl
az webapp create --resource-group myAuthResourceGroup --plan myAuthAppServicePlan --name <back-end-app-name> --deployment-local-git --query deploymentLocalGitUrl
```

::: zone-end

::: zone pivot="platform-linux"

En Cloud Shell, ejecute los siguientes comandos para crear dos aplicaciones web. Reemplace _\<front-end-app-name>_ y _\<back-end-app-name>_ por dos nombres de aplicación globalmente únicos (los caracteres válidos son `a-z`, `0-9` y `-`). Para más información acerca de los distintos comandos, consulte [Creación de una aplicación de .NET Core en Azure App Service](quickstart-dotnetcore.md).

```azurecli-interactive
az group create --name myAuthResourceGroup --location "West Europe"
az appservice plan create --name myAuthAppServicePlan --resource-group myAuthResourceGroup --sku FREE --is-linux
az webapp create --resource-group myAuthResourceGroup --plan myAuthAppServicePlan --name <front-end-app-name> --runtime "DOTNETCORE|3.1" --deployment-local-git --query deploymentLocalGitUrl
az webapp create --resource-group myAuthResourceGroup --plan myAuthAppServicePlan --name <back-end-app-name> --runtime "DOTNETCORE|3.1" --deployment-local-git --query deploymentLocalGitUrl
```

::: zone-end

> [!NOTE]
> Guarde las direcciones URL de los repositorios Git remotos de su aplicación de front-end y de back-end, que se muestran en la salida de `az webapp create`.
>

### <a name="push-to-azure-from-git"></a>Inserción en Azure desde Git

De nuevo en la _ventana de terminal local_, ejecute los siguientes comandos de Git para implementar la aplicación de back-end. Reemplace _\<deploymentLocalGitUrl-of-back-end-app>_ por la dirección URL del repositorio de Git remoto que guardó en [Creación de recursos de Azure](#create-azure-resources). Cuando el administrador de credenciales de Git solicite las credenciales, asegúrese de especificar sus [credenciales de implementación](deploy-configure-credentials.md) y no las usadas para iniciar sesión en Azure Portal.

```bash
git remote add backend <deploymentLocalGitUrl-of-back-end-app>
git push backend master
```

En la ventana de terminal local, ejecute los siguientes comandos de Git para implementar el mismo código en la aplicación de front-end. Reemplace _\<deploymentLocalGitUrl-of-front-end-app>_ por la dirección URL del repositorio de Git remoto que guardó en [Creación de recursos de Azure](#create-azure-resources).

```bash
git remote add frontend <deploymentLocalGitUrl-of-front-end-app>
git push frontend master
```

### <a name="browse-to-the-apps"></a>Navegación hasta las aplicaciones

Use un explorador para ir a las siguientes direcciones URL y ver las dos aplicaciones en funcionamiento.

```
http://<back-end-app-name>.azurewebsites.net
http://<front-end-app-name>.azurewebsites.net
```

![API de ASP.NET Core que se ejecuta en Azure App Service](./media/tutorial-auth-aad/azure-run.png)

> [!NOTE]
> Si se reinicia la aplicación, quizás haya observado que se han borrado los nuevos datos. Este comportamiento se debe a que la aplicación de ASP.NET Core de ejemplo usa una base de datos en memoria.
>
>

## <a name="call-back-end-api-from-front-end"></a>Llamada a una API de back-end desde el front-end

En este paso, vinculará el código de servidor de la aplicación de front-end para acceder a la API de back-end. Más adelante, habilitará el acceso autenticado del front-end al back-end.

### <a name="modify-front-end-code"></a>Modificación del código de front-end

En el repositorio local, abra _Controllers/TodoController.cs_. Al comienzo de la clase `TodoController`, agregue las siguientes líneas y reemplace _\<back-end-app-name>_ por el nombre de la aplicación de back-end:

```cs
private static readonly HttpClient _client = new HttpClient();
private static readonly string _remoteUrl = "https://<back-end-app-name>.azurewebsites.net";
```

Busque el método decorado con `[HttpGet]` y reemplace el código entre llaves por:

```cs
var data = await _client.GetStringAsync($"{_remoteUrl}/api/Todo");
return JsonConvert.DeserializeObject<List<TodoItem>>(data);
```

La primera línea realiza una llamada `GET /api/Todo` a la aplicación de API de back-end.

Después, busque el método decorado con `[HttpGet("{id}")]` y reemplace el código entre llaves por:

```cs
var data = await _client.GetStringAsync($"{_remoteUrl}/api/Todo/{id}");
return Content(data, "application/json");
```

La primera línea realiza una llamada `GET /api/Todo/{id}` a la aplicación de API de back-end.

Después, busque el método decorado con `[HttpPost]` y reemplace el código entre llaves por:

```cs
var response = await _client.PostAsJsonAsync($"{_remoteUrl}/api/Todo", todoItem);
var data = await response.Content.ReadAsStringAsync();
return Content(data, "application/json");
```

La primera línea realiza una llamada `POST /api/Todo` a la aplicación de API de back-end.

Después, busque el método decorado con `[HttpPut("{id}")]` y reemplace el código entre llaves por:

```cs
var res = await _client.PutAsJsonAsync($"{_remoteUrl}/api/Todo/{id}", todoItem);
return new NoContentResult();
```

La primera línea realiza una llamada `PUT /api/Todo/{id}` a la aplicación de API de back-end.

Después, busque el método decorado con `[HttpDelete("{id}")]` y reemplace el código entre llaves por:

```cs
var res = await _client.DeleteAsync($"{_remoteUrl}/api/Todo/{id}");
return new NoContentResult();
```

La primera línea realiza una llamada `DELETE /api/Todo/{id}` a la aplicación de API de back-end.

Guarde todos los cambios. En la ventana de terminal local, implemente los cambios en la aplicación de front-end con los siguientes comandos de Git:

```bash
git add .
git commit -m "call back-end API"
git push frontend master
```

### <a name="check-your-changes"></a>Comprobación de los cambios

Vaya a `http://<front-end-app-name>.azurewebsites.net` y agregue algunos elementos, como `from front end 1` y `from front end 2`.

Vaya a `http://<back-end-app-name>.azurewebsites.net` para ver los elementos agregados desde la aplicación de front-end. Además, agregue algunos elementos, como `from back end 1` y `from back end 2`, y luego actualice la aplicación de front-end para ver si refleja los cambios.

![API de ASP.NET Core que se ejecuta en Azure App Service](./media/tutorial-auth-aad/remote-api-call-run.png)

## <a name="configure-auth"></a>Configuración de la autenticación

En este paso, habilitará la autenticación y autorización para las dos aplicaciones. También configurará la aplicación de front-end para generar un token de acceso que puede usar para realizar llamadas autenticadas a la aplicación de back-end.

Usará Azure Active Directory como proveedor de identidades. Para más información, consulte [Configuración de la autenticación de Azure Active Directory para una aplicación de App Services](configure-authentication-provider-aad.md).

### <a name="enable-authentication-and-authorization-for-back-end-app"></a>Habilitación de la autenticación y autorización en una aplicación de back-end

En el menú de [Azure Portal](https://portal.azure.com), seleccione **Grupos de recursos** o busque y seleccione *Grupos de recursos* desde cualquier página.

En **Grupos de recursos**, busque y seleccione el grupo de recursos. En **Información general**, seleccione la página de administración de la aplicación de back-end.

![API de ASP.NET Core que se ejecuta en Azure App Service](./media/tutorial-auth-aad/portal-navigate-back-end.png)

En el menú de la izquierda de la aplicación de back-end, seleccione **Autenticación/autorización** y luego seleccione **Activado** para habilitar la autenticación de App Service.

En **Action to take when request is not authenticated** (Acción necesaria cuando la solicitud no está autenticada), seleccione **Log in with Azure Active Directory** (Iniciar sesión con Azure Active Directory).

En **Proveedores de autenticación,** seleccione **Azure Active Directory**.

![API de ASP.NET Core que se ejecuta en Azure App Service](./media/tutorial-auth-aad/configure-auth-back-end.png)

Seleccione **Rápido**, acepte la configuración predeterminada para crear una nueva aplicación de AD y seleccione **Aceptar**.

En la página **Autenticación/autorización**, seleccione **Guardar**.

Cuando vea la notificación con el mensaje `Successfully saved the Auth Settings for <back-end-app-name> App`, actualice la página del portal.

Vuelva a seleccionar **Azure Active Directory** y, después, seleccione la **aplicación de Azure AD**.

Copie el valor de **Identificador de cliente** de la aplicación de Azure AD en un bloc de notas. Este valor lo necesitará más adelante.

![API de ASP.NET Core que se ejecuta en Azure App Service](./media/tutorial-auth-aad/get-application-id-back-end.png)

Si se detiene aquí, tiene una aplicación independiente que ya está protegida por la autenticación y la autorización de App Service. En las secciones restantes se muestra cómo proteger una solución de varias aplicaciones al "fluir" el usuario autenticado desde el front-end hasta el back-end. 

### <a name="enable-authentication-and-authorization-for-front-end-app"></a>Habilitación de la autenticación y autorización en una aplicación de front-end

Siga los mismos pasos para la aplicación de front-end, pero omita el último paso. No es necesario el identificador de cliente para la aplicación de front-end.

Si quiere, vaya a `http://<front-end-app-name>.azurewebsites.net`. Debería dirigirle ahora a una página de inicio de sesión segura. Después de iniciar sesión, *todavía no puede tener acceso a los datos desde la aplicación de back-end*, porque la aplicación de back-end requiere ahora un inicio de sesión de Azure Active Directory desde la aplicación de front-end. Tiene que realizar tres acciones:

- Conceder al front-end acceso al back-end
- Configurar App Service para devolver un token que se pueda usar
- Usar el token en el código

> [!TIP]
> Si se producen errores y vuelve a configurar la autenticación/autorización de la aplicación, es posible que no se regeneren los tokens del almacén de tokens con esta nueva configuración. Para asegurarse de que se regeneran, debe cerrar sesión en la aplicación e iniciarla de nuevo. Una manera sencilla de hacerlo es usar el explorador en modo privado y cerrarlo y abrirlo de nuevo en este modo después de cambiar la configuración en las aplicaciones.

### <a name="grant-front-end-app-access-to-back-end"></a>Conceder a la aplicación de front-end acceso al back-end

Ahora que ha habilitado la autenticación y autorización en las dos aplicaciones, cada una de ellas está respaldada por una aplicación de AD. En este paso, concederá a la aplicación de front-end permisos para acceder al back-end en nombre del usuario. (Técnicamente, asignará a la _aplicación de AD_ del front-end permisos para acceder a la _aplicación de AD_ del back-end en nombre del usuario).

En el menú de [Azure Portal](https://portal.azure.com), seleccione **Azure Active Directory** o busque y seleccione *Azure Active Directory* desde cualquier página.

Seleccione **Registros de aplicaciones** > **Aplicaciones propias** > **View all applications in this directory** (Ver todas las aplicaciones de este directorio). Seleccione el nombre de la aplicación de front-end y, después, seleccione **Permisos de API**.

![API de ASP.NET Core que se ejecuta en Azure App Service](./media/tutorial-auth-aad/add-api-access-front-end.png)

Seleccione **Agregar un permiso** y, a continuación, **API usadas en mi organización** >  **\<back-end-app-name>** .

En la página **Solicitud de permisos de API** de la aplicación de back-end, seleccione **Permisos delegados** y **user_impersonation** y, después, seleccione **Agregar permisos**.

![API de ASP.NET Core que se ejecuta en Azure App Service](./media/tutorial-auth-aad/select-permission-front-end.png)

### <a name="configure-app-service-to-return-a-usable-access-token"></a>Configuración de App Service para devolver un token de acceso que se pueda usar

La aplicación de front-end ya tiene los permisos necesarios para acceder a la aplicación de back-end como usuario con sesión iniciada. En este paso, configurará la autenticación y autorización de App Service para proporcionarle un token de acceso que se pueda usar para acceder al back end. Para este paso necesitará el identificador de cliente del back-end, que copió en [Habilitación de la autenticación y autorización en una aplicación de back-end](#enable-authentication-and-authorization-for-back-end-app).

Vaya a [Azure Resource Explorer](https://resources.azure.com) y, mediante el árbol de recursos, busque la aplicación web de front-end.

Se abre ahora [Azure Resource Explorer](https://resources.azure.com) con la aplicación de front-end seleccionada en el árbol de recursos. En la parte superior de la página, haga clic en **Read/Write** (Lectura/escritura) para permitir la edición de los recursos de Azure.

![API de ASP.NET Core que se ejecuta en Azure App Service](./media/tutorial-auth-aad/resources-enable-write.png)

En el explorador izquierdo, explore en profundidad hasta **config** > **authsettings**.

En la vista **authsettings**, haga clic en **Edit** (Editar). Establezca `additionalLoginParams` en la siguiente cadena JSON, utilizando el identificador de cliente que copió. 

```json
"additionalLoginParams": ["response_type=code id_token","resource=<back-end-client-id>"],
```

![API de ASP.NET Core que se ejecuta en Azure App Service](./media/tutorial-auth-aad/additional-login-params-front-end.png)

Haga clic en **PUT** para guardar la configuración.

Ahora las aplicaciones están configuradas. El front-end ahora está listo para acceder al back-end con un token de acceso apropiado.

Para más información acerca de cómo configurar el token de acceso para otros proveedores, consulte [Actualización de tokens del proveedor de identidades](app-service-authentication-how-to.md#refresh-identity-provider-tokens).

## <a name="call-api-securely-from-server-code"></a>Llamada a la API de forma segura desde el código de servidor

En este paso, permitirá que el código de servidor modificado anteriormente realice llamadas autenticadas a la API de back-end.

La aplicación de front-end dispone ahora del permiso necesario y también agrega el identificador de cliente del back-end a los parámetros de inicio de sesión. Por lo tanto, puede obtener un token de acceso para la autenticación con la aplicación de back-end. App Service suministra este token al código de servicio mediante la inserción de un encabezado `X-MS-TOKEN-AAD-ACCESS-TOKEN` en cada solicitud autenticada (consulte [Retrieve tokens in app code](app-service-authentication-how-to.md#retrieve-tokens-in-app-code) ([Recuperación de tokens en el código de aplicación]).

> [!NOTE]
> Estos encabezados se insertan con todos los lenguajes admitidos. Se accede a ellos mediante el patrón estándar de cada lenguaje respectivo.

En el repositorio local, abra de nuevo _Controllers/TodoController.cs_. En la construcción `TodoController(TodoContext context)`, agregue el siguiente código:

```cs
public override void OnActionExecuting(ActionExecutingContext context)
{
    base.OnActionExecuting(context);

    _client.DefaultRequestHeaders.Accept.Clear();
    _client.DefaultRequestHeaders.Authorization =
        new AuthenticationHeaderValue("Bearer", Request.Headers["X-MS-TOKEN-AAD-ACCESS-TOKEN"]);
}
```

Este código agrega el encabezado HTTP estándar `Authorization: Bearer <access-token>` a todas las llamadas de API remotas. En la canalización de ejecución de solicitudes de MVC de ASP.NET Core, `OnActionExecuting` se ejecuta justo antes de la acción correspondiente, de modo que cada una de las llamadas API salientes presenta ahora el token de acceso.

Guarde todos los cambios. En la ventana de terminal local, implemente los cambios en la aplicación de front-end con los siguientes comandos de Git:

```bash
git add .
git commit -m "add authorization header for server code"
git push frontend master
```

Vuelva a iniciar sesión en `https://<front-end-app-name>.azurewebsites.net`. En la página de contrato de uso de datos del usuario, haga clic en **Accept** (Aceptar).

Ahora deberá poder crear, leer, actualizar y eliminar datos de la aplicación de back-end como antes. La única diferencia ahora es que ambas aplicaciones están protegidas mediante la autenticación y autorización de App Service, incluidas las llamadas de servicio a servicio.

Felicidades. El código de servidor puede acceder ahora a los datos de back-end en nombre del usuario autenticado.

## <a name="call-api-securely-from-browser-code"></a>Llamada a la API de forma segura desde el código de explorador

En este paso, vinculará la aplicación de front-end de Angular.js a la API de back-end. De esta manera, aprenderá a recuperar el token de acceso y a realizar llamadas API con él a la aplicación de back-end.

Mientras que el código de servidor tiene acceso a los encabezados de solicitud, el código de cliente puede acceder a `GET /.auth/me` para obtener los mismos tokens de acceso (consulte [Retrieve tokens in app code](app-service-authentication-how-to.md#retrieve-tokens-in-app-code) [Recuperación de tokens en el código de aplicación]).

> [!TIP]
> En esta sección se usan los métodos HTTP estándar para mostrar las llamadas HTTP seguras. De todas formas, puede usar [Microsoft Authentication Library para JavaScript](https://github.com/AzureAD/microsoft-authentication-library-for-js) para ayudar a simplificar el patrón de aplicación de Angular.js.
>

### <a name="configure-cors"></a>Configuración de CORS

En Cloud Shell, habilite CORS en la dirección URL del cliente mediante el comando [`az webapp cors add`](/cli/azure/webapp/cors#az-webapp-cors-add). De nuevo, reemplace los marcadores _\<back-end-app-name>_ y _\<front-end-app-name>_ .

```azurecli-interactive
az webapp cors add --resource-group myAuthResourceGroup --name <back-end-app-name> --allowed-origins 'https://<front-end-app-name>.azurewebsites.net'
```

Este paso no está relacionado con la autenticación y autorización. Sin embargo, son necesarios para que el explorador permita las llamadas API entre dominios desde la aplicación Angular.js. Para más información, consulte [Adición de funcionalidad CORS](app-service-web-tutorial-rest-api.md#add-cors-functionality).

### <a name="point-angularjs-app-to-back-end-api"></a>Vinculación de la aplicación Angular.js a la API de back-end

En el repositorio local, abra _wwwroot/index.html_.

En la línea 51, establezca la variable `apiEndpoint` en la dirección URL HTTPS de la aplicación de back-end (`https://<back-end-app-name>.azurewebsites.net`). Reemplace _\<back-end-app-name>_ por el nombre de la aplicación de App Service.

En el repositorio local, abra _wwwroot/app/scripts/todoListSvc.js_ y observe que `apiEndpoint` está anexado a todas las llamadas API. La aplicación Angular.js ahora está llamando a las API de back-end. 

### <a name="add-access-token-to-api-calls"></a>Adición del token de acceso a las llamadas API

En _wwwroot/app/scripts/todoListSvc.js_, encima de la lista de llamadas API (encima de la línea `getItems : function(){`), agregue la siguiente función a la lista:

```javascript
setAuth: function (token) {
    $http.defaults.headers.common['Authorization'] = 'Bearer ' + token;
},
```

Esta función se invoca para establecer el encabezado `Authorization` predeterminado con el token de acceso. La invocará en el paso siguiente.

En el repositorio local, abra _wwwroot/app/scripts/app.js_ y busque el código siguiente:

```javascript
$routeProvider.when("/Home", {
    controller: "todoListCtrl",
    templateUrl: "/App/Views/TodoList.html",
}).otherwise({ redirectTo: "/Home" });
```

Reemplace el bloque de código completo por el siguiente código:

```javascript
$routeProvider.when("/Home", {
    controller: "todoListCtrl",
    templateUrl: "/App/Views/TodoList.html",
    resolve: {
        token: ['$http', 'todoListSvc', function ($http, todoListSvc) {
            return $http.get('/.auth/me').then(function (response) {
                todoListSvc.setAuth(response.data[0].access_token);
                return response.data[0].access_token;
            });
        }]
    },
}).otherwise({ redirectTo: "/Home" });
```

El nuevo cambio se agrega a la asignación `resolve` que invoca a `/.auth/me` y establece el token de acceso. Esto garantiza que tiene el token de acceso antes de crear una instancia del controlador `todoListCtrl`. De este modo, todas las llamadas API por el controlador incluyen el token.

### <a name="deploy-updates-and-test"></a>Implementación de actualizaciones y prueba

Guarde todos los cambios. En la ventana de terminal local, implemente los cambios en la aplicación de front-end con los siguientes comandos de Git:

```bash
git add .
git commit -m "add authorization header for Angular"
git push frontend master
```

Vaya de nuevo a `https://<front-end-app-name>.azurewebsites.net`. Ahora debería poder crear, leer, actualizar y eliminar datos de la aplicación back-end, directamente en la aplicación Angular.js.

Felicidades. El código de cliente puede acceder ahora a los datos de back-end en nombre del usuario autenticado.

## <a name="when-access-tokens-expire"></a>Cuando expiren los tokens de acceso

El token de acceso expira después de un tiempo. Para obtener información sobre cómo actualizar los tokens de acceso sin requerir que los usuarios vuelvan a autenticarse con la aplicación, consulte [Refresh identity provider tokens](app-service-authentication-how-to.md#refresh-identity-provider-tokens) (Actualización de tokens del proveedor de identidades).

## <a name="clean-up-resources"></a>Limpieza de recursos

En los pasos anteriores, creó recursos de Azure en un grupo de recursos. Si prevé que no necesitará estos recursos en el futuro, elimine el grupo de recursos ejecutando el siguiente comando en Cloud Shell:

```azurecli-interactive
az group delete --name myAuthResourceGroup
```

Este comando puede tardar varios segundos en ejecutarse.

<a name="next"></a>
## <a name="next-steps"></a>Pasos siguientes

¿Qué ha aprendido?

> [!div class="checklist"]
> * Habilitación de la autenticación y autorización integradas
> * Protección de las aplicaciones frente a las solicitudes no autenticadas
> * Uso de Azure Active Directory como proveedor de identidades
> * Acceso a una aplicación remota en nombre del usuario que inició sesión
> * Protección de llamadas de servicio a servicio con autenticación de tokens
> * Uso de tokens de acceso desde el código de servidor
> * Uso de tokens de acceso desde el código de cliente (explorador)

Vaya al siguiente tutorial para aprender a asignar un nombre DNS personalizado a una aplicación web.

> [!div class="nextstepaction"]
> [Asignación de un nombre DNS personalizado existente a Azure App Service](app-service-web-tutorial-custom-domain.md)
