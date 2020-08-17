---
title: Almacenamiento en caché del token de autenticación
titleSuffix: Azure Cognitive Services
description: Este artículo le mostrará cómo almacenar en caché el token de autenticación.
author: metanMSFT
manager: guillasi
ms.service: cognitive-services
ms.subservice: immersive-reader
ms.topic: conceptual
ms.date: 01/14/2020
ms.author: metan
ms.custom: devx-track-javascript
ms.openlocfilehash: 70173e1904ce9d7a159532c8962932765060e4d9
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87406952"
---
# <a name="how-to-cache-the-authentication-token"></a>Cómo almacenar en caché el token de autenticación

En este artículo se muestra cómo almacenar en caché el token de autenticación para mejorar el rendimiento de su aplicación.

## <a name="using-aspnet"></a>Uso de ASP.NET

Importe el paquete NuGet **Microsoft.IdentityModel.Clients.ActiveDirectory**, que se usa para adquirir un token. A continuación, use el siguiente código para adquirir un `AuthenticationResult`, mediante los valores de autenticación obtenidos cuando [creó el recurso del Lector inmersivo](./how-to-create-immersive-reader.md).

```csharp
private async Task<AuthenticationResult> GetTokenAsync()
{
    AuthenticationContext authContext = new AuthenticationContext($"https://login.windows.net/{TENANT_ID}");
    ClientCredential clientCredential = new ClientCredential(CLIENT_ID, CLIENT_SECRET);
    AuthenticationResult authResult = await authContext.AcquireTokenAsync("https://cognitiveservices.azure.com/", clientCredential);
    return authResult;
}
```

El objeto `AuthenticationResult` tiene una propiedad `AccessToken` que es el token real que usará al iniciar el Lector inmersivo mediante el SDK. También tiene una propiedad `ExpiresOn` que indica cuándo expirará el token. Antes de iniciar el Lector inmersivo, puede comprobar si ha expirado el token y adquirir un nuevo token solo si lo ha hecho.

## <a name="using-nodejs"></a>Uso de Node.JS

Agregue el paquete de npm [**request**](https://www.npmjs.com/package/request) al proyecto. Use el siguiente código para adquirir un token, mediante los valores de autenticación obtenidos cuando [creó el recurso del Lector inmersivo](./how-to-create-immersive-reader.md).

```javascript
router.get('/token', function(req, res) {
    request.post(
        {
            headers: { 'content-type': 'application/x-www-form-urlencoded' },
            url: `https://login.windows.net/${TENANT_ID}/oauth2/token`,
            form: {
                grant_type: 'client_credentials',
                client_id: CLIENT_ID,
                client_secret: CLIENT_SECRET,
                resource: 'https://cognitiveservices.azure.com/'
            }
        },
        function(err, resp, json) {
            const result = JSON.parse(json);
            return res.send({
                access_token: result.access_token,
                expires_on: result.expires_on
            });
        }
    );
});
```

La propiedad `expires_on` es la fecha y hora en que expira el token, expresada como el número de segundos desde el 1 de enero de 1970 UTC. Use este valor para determinar si ha expirado su token antes de intentar adquirir uno nuevo.

```javascript
async function getToken() {
    if (Date.now() / 1000 > CREDENTIALS.expires_on) {
        CREDENTIALS = await refreshCredentials();
    }
    return CREDENTIALS.access_token;
}
```

## <a name="next-steps"></a>Pasos siguientes

* Lea la [Referencia del SDK del Lector inmersivo](./reference.md)