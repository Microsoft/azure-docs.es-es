---
title: Comprobación de ámbitos y roles de aplicación con la API web protegida | Azure
titleSuffix: Microsoft identity platform
description: Obtenga información sobre cómo compilar una API web protegida y configurar el código de la aplicación.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 07/15/2020
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: 204bc7dd8cc31f48fdc09eae6b00247023de64f3
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88120973"
---
# <a name="protected-web-api-verify-scopes-and-app-roles"></a>API web protegida: Comprobación de ámbitos y roles de aplicación

En este artículo se describe cómo se puede agregar autorización a la API web. Esta protección garantiza que solo llamen a la API:

- Aplicaciones en nombre de usuarios que tienen los ámbitos correctos.
- Aplicaciones de demonio que tienen los roles de aplicación correctos.

> [!NOTE]
> Los fragmentos de código de este artículo se han extraído de los ejemplos de código siguientes en GitHub:
>
> - [Tutorial incremental de la API web de ASP.NET Core](https://github.com/Azure-Samples/active-directory-dotnet-native-aspnetcore-v2/blob/master/1.%20Desktop%20app%20calls%20Web%20API/TodoListService/Controllers/TodoListController.cs)
> - [API web de ASP.NET de ejemplo](https://github.com/Azure-Samples/ms-identity-aspnet-webapi-onbehalfof/blob/master/TodoListService/Controllers/TodoListController.cs)

Para proteger una API web de ASP.NET o ASP.NET Core, debe agregar el atributo `[Authorize]` en uno de los siguientes elementos:

- El propio controlador, si quiere que todas las acciones del controlador estén protegidas.
- La acción de controlador individual para la API.

```csharp
    [Authorize]
    public class TodoListController : Controller
    {
     ...
    }
```

Aun así, esta protección no es suficiente. Solo garantiza que ASP.NET y ASP.NET Core validen el token. La API debe verificar que el token usado para llamar a la API se haya solicitado con las notificaciones esperadas. Estas notificaciones requieren una verificación en particular:

- Los *ámbitos*, si se llama a la API en nombre de un usuario.
- Los *roles de aplicación*, si se puede llamar a la API desde una aplicación de demonio.

## <a name="verify-scopes-in-apis-called-on-behalf-of-users"></a>Verificación de los ámbitos de las API a las que se llama en nombre de los usuarios

Si una aplicación cliente llama a la API en nombre de un usuario, la API debe solicitar un token de portador con ámbitos específicos para la API. Para más información, consulte [Configuración del código | Token de portador](scenario-protected-web-api-app-configuration.md#bearer-token).

### <a name="net-core"></a>.NET Core

#### <a name="verify-the-scopes-on-each-controller-action"></a>Verificación de los ámbitos en cada acción de controlador

```csharp
[Authorize]
public class TodoListController : Controller
{
    /// <summary>
    /// The web API will accept only tokens 1) for users, 2) that have the `access_as_user` scope for
    /// this API.
    /// </summary>
    static readonly string[] scopeRequiredByApi = new string[] { "access_as_user" };

    // GET: api/values
    [HttpGet]
    public IEnumerable<TodoItem> Get()
    {
         HttpContext.VerifyUserHasAnyAcceptedScope(scopeRequiredByApi);
        // Do the work and return the result.
        // ...
    }
...
}
```

El método `VerifyUserHasAnyAcceptedScope` hace algo similar a los pasos siguientes:

- Verificar que hay una notificación denominada `http://schemas.microsoft.com/identity/claims/scope` o `scp`.
- Verificar que la notificación tiene un valor que contiene el ámbito que espera la API.


#### <a name="verify-the-scopes-more-globally"></a>Verificación de los ámbitos más globalmente

La definición de ámbitos pormenorizados para la API web y la verificación de los ámbitos en cada acción del controlador es el enfoque recomendado. Sin embargo, también es posible verificar los ámbitos en el nivel de la aplicación o un controlador mediante ASP.NET Core. Para obtener más información, consulte [Autorización basada en notificaciones](/aspnet/core/security/authorization/claims) en la documentación de ASP.NET Core.

### <a name="net-mvc"></a>.NET MVC

Para ASP.NET, solo tiene que reemplazar `HttpContext.User` por `ClaimsPrincipal.Current` y reemplazar el tipo de notificación `"http://schemas.microsoft.com/identity/claims/scope"` por `"scp"`. Consulte también el fragmento de código incluido más adelante en este artículo.

## <a name="verify-app-roles-in-apis-called-by-daemon-apps"></a>Verificación de los roles de aplicación de las API a las que llaman aplicaciones de demonio

Si una [aplicación de demonio](scenario-daemon-overview.md) llama a la API web, la aplicación debe solicitar un permiso de aplicación a la API web. Como se muestra en [Exposición de los permisos de aplicación (roles de aplicación)](./scenario-protected-web-api-app-registration.md#exposing-application-permissions-app-roles), la API expone estos permisos. Un ejemplo es el rol de aplicación `access_as_application`.

Ahora debe hacer que la API verifique que el token recibido contiene la notificación `roles` y que esta notificación tiene el valor esperado. El código de verificación es similar al código que verifica los permisos delegados, con la diferencia de que la acción del controlador prueba los roles en lugar de los ámbitos:

### <a name="aspnet-core"></a>ASP.NET Core

```csharp
[Authorize]
public class TodoListController : ApiController
{
    public IEnumerable<TodoItem> Get()
    {
        ValidateAppRole("access_as_application");
        ...
    }
```

El método `ValidateAppRole` se define en Microsoft.Identity.Web en [RolesRequiredHttpContextExtensions.cs](https://github.com/AzureAD/microsoft-identity-web/blob/d2ad0f5f830391a34175d48621a2c56011a45082/src/Microsoft.Identity.Web/Resource/RolesRequiredHttpContextExtensions.cs#L28).

### <a name="aspnet-mvc"></a>ASP.NET MVC

```csharp
private void ValidateAppRole(string appRole)
{
    //
    // The `role` claim tells you what permissions the client application has in the service.
    // In this case, we look for a `role` value of `access_as_application`.
    //
    Claim roleClaim = ClaimsPrincipal.Current.FindFirst("roles");
    if (roleClaim == null || !roleClaim.Value.Split(' ').Contains(appRole))
    {
        throw new HttpResponseException(new HttpResponseMessage
        { StatusCode = HttpStatusCode.Unauthorized,
            ReasonPhrase = $"The 'roles' claim does not contain '{appRole}' or was not found"
        });
    }
}
}
```

### <a name="accepting-app-only-tokens-if-the-web-api-should-be-called-only-by-daemon-apps"></a>Aceptación de tokens solo de aplicación si la API web debe invocarse únicamente mediante aplicaciones de demonio

Los usuarios también pueden usar notificaciones de roles en patrones de asignación de usuarios, tal como se muestra en [Procedimientos: Agregar roles de aplicación en la aplicación y recibirlos en el token](howto-add-app-roles-in-azure-ad-apps.md). Si los roles se asignan a ambos, comprobar los roles permitirá a las aplicaciones iniciar sesión como usuarios y a los usuarios iniciar sesión como aplicaciones. Le recomendamos que declare roles diferentes para los usuarios y las aplicaciones a fin de evitar esta confusión.

Si quiere que únicamente las aplicaciones de demonio llamen a la API web, al validar el rol de aplicación, agregue una condición de que el token es solo de aplicación.

```csharp
string oid = ClaimsPrincipal.Current.FindFirst("oid")?.Value;
string sub = ClaimsPrincipal.Current.FindFirst("sub")?.Value;
bool isAppOnlyToken = oid == sub;
```

Si aplica la condición inversa, permite que llamen a la API solo las aplicaciones que inician la sesión de un usuario.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Paso a producción](scenario-protected-web-api-production.md)