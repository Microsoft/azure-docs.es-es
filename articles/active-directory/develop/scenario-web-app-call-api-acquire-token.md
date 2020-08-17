---
title: 'Obtención de un token en una aplicación web que llama a API web: plataforma de identidad de Microsoft | Azure'
description: Aprenda a adquirir un token para una aplicación web que llama a API web.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 07/14/2020
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: 4904cd95dc81aad959c88c1dfdb09416923046e6
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86518188"
---
# <a name="a-web-app-that-calls-web-apis-acquire-a-token-for-the-app"></a>Aplicación web que llama a API web: Adquisición de un token para la aplicación

Ha compilado el objeto de aplicación cliente. Ahora lo va a usar para adquirir un token para llamar a una API web. En ASP.NET o ASP.NET Core, las llamadas a las API web se hacen en el controlador:

- Obtenga un token para la API web mediante la caché de tokens. Para obtener este token, llame al método MSAL `AcquireTokenSilent` (o al equivalente en Microsoft.Identity.Web).
- Llame a la API protegida; para ello, pásele el token de acceso como un parámetro.

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

Los métodos de controlador están protegidos por un atributo `[Authorize]` que obliga a los usuarios a autenticarse para usar la aplicación web. Este es el código que llama a Microsoft Graph:

```csharp
[Authorize]
public class HomeController : Controller
{
 readonly ITokenAcquisition tokenAcquisition;

 public HomeController(ITokenAcquisition tokenAcquisition)
 {
  this.tokenAcquisition = tokenAcquisition;
 }

 // Code for the controller actions (see code below)

}
```

ASP.NET inserta el servicio `ITokenAcquisition` mediante la inserción de dependencias.

Este es código simplificado para la acción de `HomeController`, que obtiene un token para llamar a Microsoft Graph:

```csharp
[AuthorizeForScopes(Scopes = new[] { "user.read" })]
public async Task<IActionResult> Profile()
{
 // Acquire the access token.
 string[] scopes = new string[]{"user.read"};
 string accessToken = await tokenAcquisition.GetAccessTokenForUserAsync(scopes);

 // Use the access token to call a protected web API.
 HttpClient client = new HttpClient();
 client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);
 string json = await client.GetStringAsync(url);
}
```

Para entender mejor el código necesario para este escenario, consulte el paso de la fase 2 [2-1- Aplicación web llama a Microsoft Graph](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/2-WebApp-graph-user/2-1-Call-MSGraph) del tutorial [ms-identity-aspnetcore-webapp-tutorial](https://github.com/Azure-Samples/ms-identity-aspnetcore-webapp-tutorial).

Microsoft.Identity.Web proporciona el atributo `AuthorizeForScopes` en la parte superior de la acción del controlador (o de la página de Razor si usa una plantilla de Razor). Garantiza que se le solicita consentimiento al usuario, en caso necesario y de manera incremental.

Hay otras variaciones complejas, como:

- Llamada a varias API.
- Procesamiento del consentimiento incremental y el acceso condicional.

Estos pasos avanzados se tratan en el capítulo 3 del tutorial [3-WebApp-multi-APIs](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/3-WebApp-multi-APIs).

# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

El código para ASP.NET es similar al que se muestra para ASP.NET Core:

- Una acción de controlador protegida por un atributo [Authorize] extrae el identificador de inquilino y el identificador de usuario del miembro `ClaimsPrincipal` del controlador. (ASP.NET usa `HttpContext.User`).
- A partir de ahí, compila un objeto `IConfidentialClientApplication` de MSAL.NET.
- Por útimo, llama al método `AcquireTokenSilent` de la aplicación cliente confidencial.
- Si se requiere interacción, la aplicación web debe desafiar al usuario (volver a iniciar sesión) y solicitar más notificaciones.

El fragmento de código siguiente se extrae de [HomeController.cs#L157-L192](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect/blob/257c8f96ec3ff875c351d1377b36403eed942a18/WebApp/Controllers/HomeController.cs#L157-L192) en el ejemplo de código de ASP.NET MVC [ms-identity-aspnet-webapp-openidconnect](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect):

```C#
public async Task<ActionResult> ReadMail()
{
    IConfidentialClientApplication app = MsalAppBuilder.BuildConfidentialClientApplication();
    AuthenticationResult result = null;
    var account = await app.GetAccountAsync(ClaimsPrincipal.Current.GetMsalAccountId());
    string[] scopes = { "Mail.Read" };

    try
    {
        // try to get token silently
        result = await app.AcquireTokenSilent(scopes, account).ExecuteAsync().ConfigureAwait(false);
    }
    catch (MsalUiRequiredException)
    {
        ViewBag.Relogin = "true";
        return View();
    }

    // More code here
    return View();
}
```

Para obtener información detallada, vea el código para [BuildConfidentialClientApplication()](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect/blob/master/WebApp/Utils/MsalAppBuilder.cs) y [GetMsalAccountId](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect/blob/257c8f96ec3ff875c351d1377b36403eed942a18/WebApp/Utils/ClaimPrincipalExtension.cs#L38) en el ejemplo de código.


# <a name="java"></a>[Java](#tab/java)

En el ejemplo de Java, el código que llama a una API está en el método getUsersFromGraph de [AuthPageController.java#L62](https://github.com/Azure-Samples/ms-identity-java-webapp/blob/d55ee4ac0ce2c43378f2c99fd6e6856d41bdf144/src/main/java/com/microsoft/azure/msalwebsample/AuthPageController.java#L62).

El método intenta llamar a `getAuthResultBySilentFlow`. Si el usuario necesita dar su consentimiento a más ámbitos, el código procesa el objeto `MsalInteractionRequiredException` para desafiar al usuario.

```java
@RequestMapping("/msal4jsample/graph/me")
public ModelAndView getUserFromGraph(HttpServletRequest httpRequest, HttpServletResponse response)
        throws Throwable {

    IAuthenticationResult result;
    ModelAndView mav;
    try {
        result = authHelper.getAuthResultBySilentFlow(httpRequest, response);
    } catch (ExecutionException e) {
        if (e.getCause() instanceof MsalInteractionRequiredException) {

            // If the silent call returns MsalInteractionRequired, redirect to authorization endpoint
            // so user can consent to new scopes.
            String state = UUID.randomUUID().toString();
            String nonce = UUID.randomUUID().toString();

            SessionManagementHelper.storeStateAndNonceInSession(httpRequest.getSession(), state, nonce);

            String authorizationCodeUrl = authHelper.getAuthorizationCodeUrl(
                    httpRequest.getParameter("claims"),
                    "User.Read",
                    authHelper.getRedirectUriGraph(),
                    state,
                    nonce);

            return new ModelAndView("redirect:" + authorizationCodeUrl);
        } else {

            mav = new ModelAndView("error");
            mav.addObject("error", e);
            return mav;
        }
    }

    if (result == null) {
        mav = new ModelAndView("error");
        mav.addObject("error", new Exception("AuthenticationResult not found in session."));
    } else {
        mav = new ModelAndView("auth_page");
        setAccountInfo(mav, httpRequest);

        try {
            mav.addObject("userInfo", getUserInfoFromGraph(result.accessToken()));

            return mav;
        } catch (Exception e) {
            mav = new ModelAndView("error");
            mav.addObject("error", e);
        }
    }
    return mav;
}
// Code omitted here
```

# <a name="python"></a>[Python](#tab/python)

En el ejemplo de Python, el código que llama a Microsoft Graph está en [app.py#L53-L62](https://github.com/Azure-Samples/ms-identity-python-webapp/blob/48637475ed7d7733795ebeac55c5d58663714c60/app.py#L53-L62).

El código intenta obtener un token de la caché de tokens. Luego, después de establecer el encabezado de autorización, llama a la API web. Si no puede obtener un token, vuelve a iniciar la sesión del usuario.

```python
@app.route("/graphcall")
def graphcall():
    token = _get_token_from_cache(app_config.SCOPE)
    if not token:
        return redirect(url_for("login"))
    graph_data = requests.get(  # Use token to call downstream service.
        app_config.ENDPOINT,
        headers={'Authorization': 'Bearer ' + token['access_token']},
        ).json()
    return render_template('display.html', result=graph_data)
```

---

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Llamada a una API de web](scenario-web-app-call-api-call-api.md)
