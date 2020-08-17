---
title: Obtención de un token para una API web que llama a otras API web | Azure
titleSuffix: Microsoft identity platform
description: Aprenda a compilar una API web que llama a otras API web que necesitan adquirir un token para la aplicación.
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
ms.openlocfilehash: 7e0701cc5a9bb14800a48e2281dba1eb6ea0cf72
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87026465"
---
# <a name="a-web-api-that-calls-web-apis-acquire-a-token-for-the-app"></a>API web que llama a API web: Adquisición de un token para la aplicación

Una vez que ha creado un objeto de aplicación cliente, lo puede usar para adquirir un token y usarlo para llamar a una API web.

## <a name="code-in-the-controller"></a>Código del controlador

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

Este es un ejemplo de código que usa Microsoft.Identity.Web, que se llama en las acciones de los controladores de API. Llama a una API de bajada denominada *todolist*. Para obtener un token para llamar a la API de nivel inferior, se inserta el servicio `ITokenAcquisition` mediante la inserción de dependencias en el constructor del controlador (o el constructor de la página si utiliza Blazor) y se usa en las acciones del controlador, de modo que se obtiene un token para el usuario (`GetAccessTokenForUserAsync`) o para la propia aplicación (`GetAccessTokenForAppAsync`) en caso de un escenario de demonio.

```csharp
[Authorize]
public class MyApiController : Controller
{
    /// <summary>
    /// The web API will accept only tokens 1) for users, 2) that have the `access_as_user` scope for
    /// this API.
    /// </summary>
    static readonly string[] scopeRequiredByApi = new string[] { "access_as_user" };

     static readonly string[] scopesToAccessDownstreamApi = new string[] { "api://MyTodolistService/access_as_user" };

    private readonly ITokenAcquisition _tokenAcquisition;

    public MyApiController(ITokenAcquisition tokenAcquisition)
    {
        _tokenAcquisition = tokenAcquisition;
    }

    public IActionResult Index()
    {
        HttpContext.VerifyUserHasAnyAcceptedScope(scopeRequiredByApi);

        string accessToken = _tokenAcquisition.GetAccessTokenForUserAsync(scopesToAccessDownstreamApi);
        return await callTodoListService(accessToken);
    }
}
```

Para más información sobre el método `callTodoListService`, consulte [Una API web que llama a las API web: llamada a una API](scenario-web-api-call-api-call-api.md).

# <a name="java"></a>[Java](#tab/java)
A continuación, se muestra un ejemplo de código que se llama en las acciones de los controladores de API. Llama a la API de nivel inferior de Microsoft Graph.

```java
@RestController
public class ApiController {

    @Autowired
    MsalAuthHelper msalAuthHelper;

    @RequestMapping("/graphMeApi")
    public String graphMeApi() throws MalformedURLException {

        String oboAccessToken = msalAuthHelper.getOboToken("https://graph.microsoft.com/.default");

        return callMicrosoftGraphMeEndpoint(oboAccessToken);
    }

}
```

# <a name="python"></a>[Python](#tab/python)

Una API web de Python deberá usar algún middleware para validar el token de portador recibido del cliente. A continuación, la API web podrá obtener el token de acceso para la API de nivel inferior mediante la biblioteca de MSAL para Python al llamar al método [`acquire_token_on_behalf_of`](https://msal-python.readthedocs.io/en/latest/?badge=latest#msal.ConfidentialClientApplication.acquire_token_on_behalf_of). Aún no hay disponible ningún ejemplo de este flujo con MSAL Python.

---

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [API web que llama a las API web: llamada a una API](scenario-web-api-call-api-call-api.md)
