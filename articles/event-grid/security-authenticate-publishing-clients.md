---
title: Autenticación de los clientes que publican eventos en temas o dominios personalizados de Event Grid.
description: En este artículo se describen diferentes maneras de autenticar a los clientes que publican eventos en temas personalizados de Event Grid.
ms.topic: conceptual
ms.date: 07/07/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: e934ce0d8f5e31dc8dd7592a2e553cd278af2b10
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89019120"
---
# <a name="authenticate-publishing-clients-azure-event-grid"></a>Autenticación de clientes que hacen publicaciones (Azure Event Grid).
En este artículo se proporciona información sobre la autenticación de clientes que publican eventos en temas o dominios de Azure Event Grid mediante **clave de acceso** o token de **Firma de acceso compartido (SAS)** . Se recomienda el uso de SAS, pero la autenticación de clave proporciona programación simple y es compatible con muchos editores de webhook existentes.  

## <a name="authenticate-using-an-access-key"></a>Autenticación mediante una clave de acceso
La autenticación de clave es la forma más sencilla de autenticación. Puede pasar la clave de acceso como un encabezado HTTP o un parámetro de consulta de dirección URL. 

### <a name="access-key-in-a-http-header"></a>Clave de acceso en un encabezado HTTP
Pase la clave de acceso como un valor para el encabezado HTTP: `aeg-sas-key`.

```
aeg-sas-key: XXXXXXXXXXXXXXXXXX0GXXX/nDT4hgdEj9DpBeRr38arnnm5OFg==
```

### <a name="access-key-as-a-query-parameter"></a>Clave de acceso como parámetro de consulta
También puede especificar `aeg-sas-key` como parámetro de consulta. 

```
https://<yourtopic>.<region>.eventgrid.azure.net/api/events?aeg-sas-key=XXXXXXXX53249XX8XXXXX0GXXX/nDT4hgdEj9DpBeRr38arnnm5OFg==
```

Para obtener instrucciones sobre cómo obtener las claves de acceso para un tema o un dominio, consulte [Obtener claves de acceso](get-access-keys.md).

## <a name="authenticate-using-a-sas-token"></a>Autenticación mediante un token de SAS
Los tokens de SAS para Event Grid incluyen el recurso, un tiempo de expiración y una firma. El formato del token de SAS es: `r={resource}&e={expiration}&s={signature}`.

El recurso es la ruta de acceso del tema de Event Grid al que envía los eventos. Por ejemplo, una ruta de acceso de recurso válida es: `https://<yourtopic>.<region>.eventgrid.azure.net/api/events`. Para ver todas las versiones de API compatibles, vea [Tipos de recursos de Microsoft.EventGrid](/azure/templates/microsoft.eventgrid/allversions). 

En primer lugar, genere un token de SAS mediante programación y, a continuación, use el encabezado de `aeg-sas-token` o el encabezado de `Authorization SharedAccessSignature` para autenticarse con Event Grid. 

### <a name="generate-sas-token-programmatically"></a>Generar tokens de SAS mediante programación
En el ejemplo siguiente se crea un token de SAS para su uso con Event Grid:

```cs
static string BuildSharedAccessSignature(string resource, DateTime expirationUtc, string key)
{
    const char Resource = 'r';
    const char Expiration = 'e';
    const char Signature = 's';

    string encodedResource = HttpUtility.UrlEncode(resource);
    var culture = CultureInfo.CreateSpecificCulture("en-US");
    var encodedExpirationUtc = HttpUtility.UrlEncode(expirationUtc.ToString(culture));

    string unsignedSas = $"{Resource}={encodedResource}&{Expiration}={encodedExpirationUtc}";
    using (var hmac = new HMACSHA256(Convert.FromBase64String(key)))
    {
        string signature = Convert.ToBase64String(hmac.ComputeHash(Encoding.UTF8.GetBytes(unsignedSas)));
        string encodedSignature = HttpUtility.UrlEncode(signature);
        string signedSas = $"{unsignedSas}&{Signature}={encodedSignature}";

        return signedSas;
    }
}
```

### <a name="using-aeg-sas-token-header"></a>Uso del encabezado de token AEG-SAS
Este es un ejemplo de cómo pasar el token de SAS como un valor para el encabezado `aeg-sas-toke`. 

```http
aeg-sas-token: r=https%3a%2f%2fmytopic.eventgrid.azure.net%2fapi%2fevent&e=6%2f15%2f2017+6%3a20%3a15+PM&s=XXXXXXXXXXXXX%2fBPjdDLOrc6THPy3tDcGHw1zP4OajQ%3d
```

### <a name="using-authorization-header"></a>Uso del encabezado de autorización
Este es un ejemplo de cómo pasar el token de SAS como un valor para el encabezado `Authorization`. 

```http
Authorization: SharedAccessSignature r=https%3a%2f%2fmytopic.eventgrid.azure.net%2fapi%2fevent&e=6%2f15%2f2017+6%3a20%3a15+PM&s=XXXXXXXXXXXXX%2fBPjdDLOrc6THPy3tDcGHw1zP4OajQ%3d
```

## <a name="next-steps"></a>Pasos siguientes
Consulte [Autenticación de entrega de eventos](security-authentication.md) para obtener información sobre la autenticación con controladores de eventos para entregar eventos. 