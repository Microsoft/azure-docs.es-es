---
title: 'Llamada a una API web desde una aplicación web: Plataforma de identidad de Microsoft | Azure'
description: Obtenga información sobre cómo compilar una aplicación web que llame a las API web (llamada a una API web protegida)
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 07/14/2019
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: 1e448f52f4e8c24dd8552cae873edac841e57fc6
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87058441"
---
# <a name="a-web-app-that-calls-web-apis-call-a-web-api"></a>Aplicación web que llama a las API web: Llamada a una API de web

Ahora que dispone de un token, puede llamar a una API web protegida.

## <a name="call-a-protected-web-api"></a>Llamada a una API web protegida

La llamada a una API web protegida depende del lenguaje y el marco elegidos:

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

Aquí encontrará un código simplificado de la acción de `HomeController`. Este código obtiene un token para llamar a Microsoft Graph. Se ha agregado el código para mostrar cómo llamar a Microsoft Graph como API de REST. La URL de Microsoft Graph API se proporciona en el archivo appsettings.json y se lee en una variable denominada `webOptions`:

```json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    ...
  },
  ...
  "GraphApiUrl": "https://graph.microsoft.com"
}
```

```csharp
public async Task<IActionResult> Profile()
{
 // Acquire the access token.
 string[] scopes = new string[]{"user.read"};
 string accessToken = await tokenAcquisition.GetAccessTokenForUserAsync(scopes);

 // Use the access token to call a protected web API.
 HttpClient client = new HttpClient();
 client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);

  var response = await httpClient.GetAsync($"{webOptions.GraphApiUrl}/beta/me");

  if (response.StatusCode == HttpStatusCode.OK)
  {
   var content = await response.Content.ReadAsStringAsync();

   dynamic me = JsonConvert.DeserializeObject(content);
   ViewData["Me"] = me;
  }

  return View();
}
```

> [!NOTE]
> Puede usar el mismo principio para llamar a cualquier API web.
>
> La mayoría de las API web de Azure proporcionan un SDK que simplifica la llamada a la API. Esto también se cumple para Microsoft Graph. En el siguiente artículo, aprenderá dónde encontrar un tutorial que ilustra el uso de la API.

# <a name="java"></a>[Java](#tab/java)

```Java
private String getUserInfoFromGraph(String accessToken) throws Exception {
    // Microsoft Graph user endpoint
    URL url = new URL("https://graph.microsoft.com/v1.0/me");

    HttpURLConnection conn = (HttpURLConnection) url.openConnection();

    // Set the appropriate header fields in the request header.
    conn.setRequestProperty("Authorization", "Bearer " + accessToken);
    conn.setRequestProperty("Accept", "application/json");

    String response = HttpClientHelper.getResponseStringFromConn(conn);

    int responseCode = conn.getResponseCode();
    if(responseCode != HttpURLConnection.HTTP_OK) {
        throw new IOException(response);
    }

    JSONObject responseObject = HttpClientHelper.processResponse(responseCode, response);
    return responseObject.toString();
}

```

# <a name="python"></a>[Python](#tab/python)

```Python
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
> [Paso a producción](scenario-web-app-call-api-production.md)
