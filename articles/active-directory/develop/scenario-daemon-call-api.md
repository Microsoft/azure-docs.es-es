---
title: 'Llamada a una API web desde una aplicación de demonio: Plataforma de identidad de Microsoft | Azure'
description: Obtenga información sobre cómo compilar una aplicación de demonio que llama a las API web
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 10/30/2019
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: 7319aacabd2c479c578fbb71ae0bc9fbb2dee9f2
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88119137"
---
# <a name="daemon-app-that-calls-web-apis---call-a-web-api-from-the-app"></a>Aplicación de demonio que llama a las API web: llamada a una API web desde la aplicación

Las aplicaciones de demonio de .NET pueden llamar a una API web. Las aplicaciones de demonio de .NET también pueden llamar a varias API web aprobadas previamente.

## <a name="calling-a-web-api-from-a-daemon-application"></a>Llamada a una API web desde una aplicación de demonio

Aquí se muestra cómo usar el token para llamar a una API:

# <a name="net"></a>[.NET](#tab/dotnet)

[!INCLUDE [Call web API in .NET](../../../includes/active-directory-develop-scenarios-call-apis-dotnet.md)]

# <a name="python"></a>[Python](#tab/python)

```Python
endpoint = "url to the API"
http_headers = {'Authorization': 'Bearer ' + result['access_token'],
                'Accept': 'application/json',
                'Content-Type': 'application/json'}
data = requests.get(endpoint, headers=http_headers, stream=False).json()
```

# <a name="java"></a>[Java](#tab/java)

```Java
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
```

---

## <a name="calling-several-apis"></a>Llamada a varias API

Para las aplicaciones de demonio, las API web a las que llame deben estar aprobadas previamente. No hay ningún consentimiento incremental con las aplicaciones de demonio. (No hay interacción del usuario). El administrador de inquilinos debe dar el consentimiento previo a la aplicación y a todos los permisos de la API. Si quiere llamar a varias API, debe adquirir un token para cada recurso, cada vez llamando a `AcquireTokenForClient`. MSAL usará la caché de tokens de aplicación para evitar las llamadas de servicio innecesarias.

## <a name="next-steps"></a>Pasos siguientes

# <a name="net"></a>[.NET](#tab/dotnet)

> [!div class="nextstepaction"]
> [Aplicación de demonio: paso a producción](./scenario-daemon-production.md?tabs=dotnet)

# <a name="python"></a>[Python](#tab/python)

> [!div class="nextstepaction"]
> [Aplicación de demonio: paso a producción](./scenario-daemon-production.md?tabs=python)

# <a name="java"></a>[Java](#tab/java)

> [!div class="nextstepaction"]
> [Aplicación de demonio: paso a producción](./scenario-daemon-production.md?tabs=java)

---