---
title: 'Aplicación móvil que llama a web API: llamar a una API web | Plataforma de identidad de Microsoft'
description: Aprenda a crear una aplicación móvil que llama a web API (llamando a una API web)
services: active-directory
documentationcenter: dev-center-name
author: danieldobalian
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/07/2019
ms.author: dadobali
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 46d8b138a566727f9172b627b8df3353e7216fa5
ms.sourcegitcommit: f013c433b18de2788bf09b98926c7136b15d36f1
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2019
ms.locfileid: "65550318"
---
# <a name="mobile-app-that-calls-web-apis---call-a-web-api"></a>Aplicación móvil que llama a web API: llamar a una API web

Después de la aplicación ha iniciado sesión un usuario y recibe los tokens, MSAL expone varias piezas de información sobre el usuario, el entorno del usuario y los tokens emitidos. La aplicación puede utilizar estos valores para llamar a una API web o mostrar un mensaje de bienvenida al usuario.

En primer lugar, buscaremos el resultado MSAL. A continuación, veremos cómo usar un token de acceso de la `AuthenticationResult` o `result` para llamar a una API web protegida.

## <a name="msal-result"></a>Resultado MSAL
MSAL proporciona los siguientes valores: 

- `AccessToken`: Se usa para llamar a API web protegidas en una solicitud de portador de HTTP.
- `IdToken`: Contiene información útil sobre el usuario que inició sesión, como el nombre del usuario, el inquilino principal y un identificador único para el almacenamiento.
- `ExpiresOn`: Fecha de expiración del token. MSAL controla la actualización automática para las aplicaciones.
- `TenantId`: El identificador del inquilino que el usuario ha iniciado sesión. Para los usuarios invitados (Azure Active Directory B2B), este valor identificará al inquilino que el usuario ha iniciado sesión con no inquilino principal del usuario.  
- `Scopes`: Los ámbitos que se haya concedido con el token. Los ámbitos concedidos pueden ser un subconjunto de los ámbitos que ha solicitado.

MSAL también proporciona una abstracción para un `Account`. Un `Account` representa la actual cuenta de usuario con sesión iniciada.

- `HomeAccountIdentifier`: El identificador del inquilino principal del usuario.
- `UserName`: Nombre de usuario preferido del usuario. Esto podría estar vacío para los usuarios de Azure Active Directory B2C.
- `AccountIdentifier`: El identificador del usuario con sesión iniciada. Este valor será el mismo que el `HomeAccountIdentifier` valor en la mayoría de los casos, a menos que el usuario es un invitado en otro inquilino.

## <a name="call-an-api"></a>Llamar a una API

Una vez que el token de acceso, es fácil llamar a una API web. La aplicación usará el token para construir una solicitud HTTP y, a continuación, ejecutar la solicitud.

### <a name="android"></a>Android

```Java
        RequestQueue queue = Volley.newRequestQueue(this);
        JSONObject parameters = new JSONObject();

        try {
            parameters.put("key", "value");
        } catch (Exception e) {
            // Error when constructing.
        }
        JsonObjectRequest request = new JsonObjectRequest(Request.Method.GET, MSGRAPH_URL,
                parameters,new Response.Listener<JSONObject>() {
            @Override
            public void onResponse(JSONObject response) {
                // Successfully called Graph. Process data and send to UI.
            }
        }, new Response.ErrorListener() {
            @Override
            public void onErrorResponse(VolleyError error) {
                // Error.
            }
        }) {
            @Override
            public Map<String, String> getHeaders() throws AuthFailureError {
                Map<String, String> headers = new HashMap<>();
                
                // Put access token in HTTP request.
                headers.put("Authorization", "Bearer " + authResult.getAccessToken());
                return headers;
            }
        };

        request.setRetryPolicy(new DefaultRetryPolicy(
                3000,
                DefaultRetryPolicy.DEFAULT_MAX_RETRIES,
                DefaultRetryPolicy.DEFAULT_BACKOFF_MULT));
        queue.add(request);
```

### <a name="ios"></a>iOS

```swift
        let url = URL(string: kGraphURI)
        var request = URLRequest(url: url!)

        // Put access token in HTTP request.
        request.setValue("Bearer \(self.accessToken)", forHTTPHeaderField: "Authorization")

        URLSession.shared.dataTask(with: request) { data, response, error in
            if let error = error {
                self.updateLogging(text: "Couldn't get graph result: \(error)")
                return
            }
            guard let result = try? JSONSerialization.jsonObject(with: data!, options: []) else {
                self.updateLogging(text: "Couldn't deserialize result JSON")
                return
            }

            // Successfully got data from Graph.
            self.updateLogging(text: "Result from Graph: \(result))")
        }.resume()
```

### <a name="xamarin"></a>Xamarin

```CSharp
httpClient = new HttpClient();

// Put access token in HTTP request.
httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", result.AccessToken);

// Call Graph.
HttpResponseMessage response = await _httpClient.GetAsync(apiUri);
...
}
```

## <a name="making-several-api-requests"></a>Realización de varias solicitudes de API

Si tiene que llamar varias veces a la misma API, o si se debe llamar a varias API, tenga en cuenta la siguiente al compilar la aplicación:

- **Consentimiento incremental**: Plataforma de identidad de Microsoft permite que las aplicaciones obtener el consentimiento del usuario como los permisos son necesarios, en lugar de todo al principio. Cada vez que la aplicación está lista para llamar a una API, debe solicitar solo los ámbitos que debe usar.
- **Acceso condicional**: En determinados escenarios, podría obtener los requisitos de acceso condicional adicional al realizar varias solicitudes de API. Esto puede ocurrir si la primera solicitud no tiene aplicado ninguna directiva de acceso condicional y la aplicación intenta acceder en modo silencioso una nueva API que requiera el acceso condicional. Para controlar este escenario, asegúrese de detectar errores de solicitudes silenciosas y prepárese realizar una solicitud interactiva.  Para obtener más información, consulte [orientación para el acceso condicional](conditional-access-dev-guide.md).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Pasar a producción](scenario-mobile-production.md)
