---
title: 'Inicio rápido: Protección de una API web de ASP.NET Core con la plataforma de identidad de Microsoft | Azure'
titleSuffix: Microsoft identity platform
description: En esta guía de inicio rápido, descargará y modificará un ejemplo de código que muestra cómo proteger una API web de ASP.NET Core mediante la plataforma de identidad de Microsoft para la autorización.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 09/22/2020
ms.author: jmprieur
ms.custom: devx-track-csharp, scenarios:getting-started, languages:aspnet-core
ms.openlocfilehash: dc0cdca2355403bc8f5409d9a6ca7f4ae89caf25
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945594"
---
# <a name="quickstart-protect-an-aspnet-core-web-api-with-microsoft-identity-platform"></a>Inicio rápido: Protección de una API web de ASP.NET Core con la plataforma de identidad de Microsoft

En esta guía de inicio rápido, se usa un ejemplo de código para aprender a proteger una API web de ASP.NET Core de modo que solo sea accesible para las cuentas autorizadas. Estas pueden ser cuentas personales (hotmail.com, outlook.com y otras), profesionales y educativas de cualquier instancia de Azure Active Directory (Azure AD).

> [!div renderon="docs"]
> ## <a name="prerequisites"></a>Requisitos previos
>
> - Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
> - [Inquilino de Azure Active Directory](quickstart-create-new-tenant.md)
> - [SDK de .NET Core 3.1+](https://dotnet.microsoft.com/)
> - [Visual Studio 2019](https://visualstudio.microsoft.com/vs/) o [Visual Studio Code](https://code.visualstudio.com/)
>
> ## <a name="step-1-register-the-application"></a>Paso 1: Registro de la aplicación
>
> En primer lugar, registre la API web en el inquilino de Azure AD y agregue un ámbito siguiendo estos pasos:
>
> 1. Inicie sesión en [Azure Portal](https://portal.azure.com).
> 1. Si tiene acceso a varios inquilinos, use el filtro **Directorio + suscripción** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false"::: del menú superior para seleccionar el inquilino en el que desea registrar una aplicación.
> 1. Busque y seleccione **Azure Active Directory**.
> 1. En **Administrar**, seleccione **Registros de aplicaciones** y luego **Nuevo registro**.
> 1. Escriba el **Nombre** de la aplicación, por ejemplo `AspNetCoreWebApi-Quickstart`. Los usuarios de la aplicación pueden ver este nombre, el cual se puede cambiar más tarde.
> 1. Seleccione **Registrar**.
> 1. En **Administrar**, seleccione **Exponer una API**.
> 1. Seleccione **Agregar un ámbito** y, a continuación, **Guardar y continuar** para aceptar el **URI de id. de aplicación** predeterminado.
> 1. En el panel **Agregar un ámbito**, escriba los valores siguientes:
>    - **Nombre de ámbito**: `access_as_user`
>    - **¿Quién puede dar el consentimiento?** : **Administradores y usuarios**
>    - **Nombre para mostrar del consentimiento del administrador**: `Access AspNetCoreWebApi-Quickstart`
>    - **Descripción del consentimiento del administrador**: `Allows the app to access AspNetCoreWebApi-Quickstart as the signed-in user.`
>    - **Nombre para mostrar del consentimiento del usuario**: `Access AspNetCoreWebApi-Quickstart`
>    - **Descripción del consentimiento del usuario**: `Allow the application to access AspNetCoreWebApi-Quickstart on your behalf.`
>    - **Estado**: **Enabled**
> 1. Seleccione **Agregar ámbito** para completar la adición del ámbito.

## <a name="step-2-download-the-aspnet-core-project"></a>Paso 2: Descarga del proyecto de ASP.NET Core

> [!div renderon="docs"]
> [Descargue la solución de ASP.NET Core](https://github.com/Azure-Samples/active-directory-dotnet-native-aspnetcore-v2/archive/aspnetcore3-1.zip) de GitHub.

> [!div renderon="docs"]
> ## <a name="step-3-configure-the-aspnet-core-project"></a>Paso 3: Configuración del proyecto de ASP.NET Core
>
> En este paso, configure el código de ejemplo para que funcione con el registro de la aplicación que creó anteriormente.
>
> 1. Extraiga el archivo. zip en una carpeta cerca de la raíz de la unidad. Por ejemplo, en *C:\Azure-Samples*.
> 1. Abra la solución en la carpeta *webapi* en el editor de código.
> 1. Abra el archivo *appsettings.json* y modifique lo siguiente:
>
>    ```json
>    "ClientId": "Enter_the_Application_Id_here",
>    "TenantId": "Enter_the_Tenant_Info_Here"
>    ```
>
>    - Reemplace `Enter_the_Application_Id_here` por el **Id. de aplicación (cliente)** de la aplicación que ha registrado en Azure Portal. Puede encontrar el **Identificador de aplicación (cliente)** en la página **Información general** de la aplicación.
>    - Reemplace `Enter_the_Tenant_Info_Here` por una de las opciones siguientes:
>       - Si la aplicación admite **Solo las cuentas de este directorio organizativo**, reemplace este valor por el **Id. de directorio (inquilino)** (un GUID) o por el **Nombre del inquilino** (por ejemplo, `contoso.onmicrosoft.com`). Puede encontrar el **Id. de directorio (inquilino)** en la página **Introducción** de la aplicación.
>       - Si la aplicación admite **Cuentas en cualquier directorio organizativo**, reemplace este valor por `organizations`
>       - Si la aplicación admite **Todos los usuarios de cuentas Microsoft**, deje este valor establecido en `common`.
>
> Para este inicio rápido, no cambie ningún otro valor del archivo *appsettings.json*.

## <a name="how-the-sample-works"></a>Funcionamiento del ejemplo

La API web recibe un token de una aplicación cliente, y el código de la API web valida lo valida. Este escenario se explica con más detalle en [Escenario: API web protegida](scenario-protected-web-api-overview.md).

### <a name="startup-class"></a>Clase Startup

El middleware *Microsoft.AspNetCore.Authentication* usa una clase `Startup` que se ejecuta cuando se inicializa el proceso de hospedaje. En su método `ConfigureServices` , se llama al método de extensión `AddMicrosoftIdentityWebApi` proporcionado por *Microsoft.Identity.Web*.

```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
                .AddMicrosoftIdentityWebApi(Configuration, "AzureAd");
    }
```

El método `AddAuthentication()` configura el servicio para agregar la autenticación basada en JwtBearer.

La línea que contiene `.AddMicrosoftIdentityWebApi` agrega la autorización de la plataforma de identidad de Microsoft a la API web. Después se configura para validar los tokens de acceso emitidos por el punto de conexión de la plataforma de identidad de Microsoft según la información de la sección `AzureAD` del archivo de configuración *appsettings.json*:

| Clave *appsettings.json* | Descripción                                                                                                                                                          |
|------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `ClientId`             | El **Id. de aplicación (cliente)** de la aplicación registrada en Azure Portal.                                                                                       |
| `Instance`             | El punto de conexión del servicio de token de seguridad (STS) para que el usuario se autentique. Normalmente, este valor es `https://login.microsoftonline.com/`, que indica la nube pública de Azure. |
| `TenantId`             | Nombre o id. del inquilino (un GUID), o bien *common*, para el inicio de sesión de usuarios con cuentas profesionales o educativas o cuentas personales de Microsoft.                             |

El método `Configure()` contiene dos métodos importantes, `app.UseAuthentication()` y `app.UseAuthorization()`, que habilitan su funcionalidad con nombre:

```csharp
// This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    // more code
    app.UseAuthentication();
    app.UseAuthorization();
    // more code
}
```

### <a name="protect-a-controller-a-controllers-method-or-a-razor-page"></a>Protección de un controlador, del método de un controlador o de una página de Razor

Puede proteger un controlador o los métodos de un controlador mediante el atributo `[Authorize]`. Este atributo restringe el acceso al controlador o a los métodos, ya que solo permite usuarios autenticados, lo que significa que se puede iniciar el desafío de autenticación para acceder al controlador si el usuario no está autenticado.

```csharp
namespace webapi.Controllers
{
    [Authorize]
    [ApiController]
    [Route("[controller]")]
    public class WeatherForecastController : ControllerBase
```

### <a name="validate-the-scope-in-the-controller"></a>Validación del ámbito en el controlador

A continuación, el código de la API comprueba que los ámbitos necesarios están en el token mediante `HttpContext.VerifyUserHasAnyAcceptedScope(scopeRequiredByApi);`.

```csharp
namespace webapi.Controllers
{
    [Authorize]
    [ApiController]
    [Route("[controller]")]
    public class WeatherForecastController : ControllerBase
    {
        // The Web API will only accept tokens 1) for users, and 2) having the "access_as_user" scope for this API
        static readonly string[] scopeRequiredByApi = new string[] { "access_as_user" };

        [HttpGet]
        public IEnumerable<WeatherForecast> Get()
        {
            HttpContext.VerifyUserHasAnyAcceptedScope(scopeRequiredByApi);

            // some code here
        }
    }
}
```

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Pasos siguientes

El repositorio de GitHub que contiene este ejemplo de código de API web de ASP.NET Core incluye instrucciones y más ejemplos de código que muestran cómo:

- Agregar autenticación a una nueva API web de ASP.NET Core.
- Llamar a la API web desde una aplicación de escritorio.
- Llamar a las API de bajada como Microsoft Graph y otras API de Microsoft.

> [!div class="nextstepaction"]
> [Tutoriales de API web de ASP.NET Core en GitHub](https://github.com/Azure-Samples/active-directory-dotnet-native-aspnetcore-v2)
