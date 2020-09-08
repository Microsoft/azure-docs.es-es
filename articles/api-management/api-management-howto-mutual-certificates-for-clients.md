---
title: Protección de API mediante la autenticación de certificados de cliente en API Management
titleSuffix: Azure API Management
description: Aprenda a proteger el acceso a las API mediante certificados de cliente. Se pueden usar expresiones de directiva para validar los certificados entrantes.
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 01/13/2020
ms.author: apimpm
ms.openlocfilehash: 19e0d741d959eba704f26e7e8f7b5d311aa77775
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87904876"
---
# <a name="how-to-secure-apis-using-client-certificate-authentication-in-api-management"></a>Protección de API mediante la autenticación de certificados de cliente en API Management

API Management proporciona la capacidad de proteger el acceso a las API (es decir, de cliente a API Management) mediante certificados de cliente. Puede validar el certificado entrante y comprobar las propiedades del certificado con los valores deseados mediante expresiones de directiva.

Para información acerca de cómo proteger el acceso al servicio de back-end de una API mediante certificados de cliente (por ejemplo, de API Management para back-end), consulte [Cómo asegurar servicios back-end con la autenticación de certificados de cliente en Azure API Management](./api-management-howto-mutual-certificates.md)

> [!IMPORTANT]
> Para recibir y comprobar los certificados de cliente a través de HTTP/2 en los niveles Desarrollador, Básico, Estándar o Premium, debe activar la opción "Negociación del certificado de cliente" en la hoja "Dominios personalizados", tal como se muestra a continuación.

![Negociación del certificado de cliente](./media/api-management-howto-mutual-certificates-for-clients/negotiate-client-certificate.png)

> [!IMPORTANT]
> Para recibir y comprobar los certificados de cliente en el nivel Consumo, debe activar la opción "Solicitar certificado de cliente" en la hoja "Dominios personalizados", tal como se muestra a continuación.

![Solicitar certificado de cliente](./media/api-management-howto-mutual-certificates-for-clients/request-client-certificate.png)

## <a name="checking-the-issuer-and-subject"></a>Comprobación del emisor y el asunto

Es posible configurar las directivas siguientes para comprobar el emisor y el firmante de un certificado de cliente:

```xml
<choose>
    <when condition="@(context.Request.Certificate == null || !context.Request.Certificate.Verify() || context.Request.Certificate.Issuer != "trusted-issuer" || context.Request.Certificate.SubjectName.Name != "expected-subject-name")" >
        <return-response>
            <set-status code="403" reason="Invalid client certificate" />
        </return-response>
    </when>
</choose>
```

> [!NOTE]
> Para deshabilitar la comprobación de la lista de revocación de certificados, use `context.Request.Certificate.VerifyNoRevocation()` en lugar de `context.Request.Certificate.Verify()`.
> Si el certificado de cliente es un certificado autofirmado, los certificados CA raíz (o intermedios) deben [cargarse](api-management-howto-ca-certificates.md) en API Management para que `context.Request.Certificate.Verify()` y `context.Request.Certificate.VerifyNoRevocation()` funcionen.

## <a name="checking-the-thumbprint"></a>Comprobación de la huella digital

Es posible configurar las directivas siguientes para comprobar la huella digital de un certificado de cliente:

```xml
<choose>
    <when condition="@(context.Request.Certificate == null || !context.Request.Certificate.Verify() || context.Request.Certificate.Thumbprint != "DESIRED-THUMBPRINT-IN-UPPER-CASE")" >
        <return-response>
            <set-status code="403" reason="Invalid client certificate" />
        </return-response>
    </when>
</choose>
```

> [!NOTE]
> Para deshabilitar la comprobación de la lista de revocación de certificados, use `context.Request.Certificate.VerifyNoRevocation()` en lugar de `context.Request.Certificate.Verify()`.
> Si el certificado de cliente es un certificado autofirmado, los certificados CA raíz (o intermedios) deben [cargarse](api-management-howto-ca-certificates.md) en API Management para que `context.Request.Certificate.Verify()` y `context.Request.Certificate.VerifyNoRevocation()` funcionen.

## <a name="checking-a-thumbprint-against-certificates-uploaded-to-api-management"></a>Comprobación de una huella digital en relación con certificados cargados en API Management

En el ejemplo siguiente se muestra cómo comprobar la huella digital de un certificado de cliente en relación con los certificados cargados en API Management:

```xml
<choose>
    <when condition="@(context.Request.Certificate == null || !context.Request.Certificate.Verify()  || !context.Deployment.Certificates.Any(c => c.Value.Thumbprint == context.Request.Certificate.Thumbprint))" >
        <return-response>
            <set-status code="403" reason="Invalid client certificate" />
        </return-response>
    </when>
</choose>

```

> [!NOTE]
> Para deshabilitar la comprobación de la lista de revocación de certificados, use `context.Request.Certificate.VerifyNoRevocation()` en lugar de `context.Request.Certificate.Verify()`.
> Si el certificado de cliente es un certificado autofirmado, los certificados CA raíz (o intermedios) deben [cargarse](api-management-howto-ca-certificates.md) en API Management para que `context.Request.Certificate.Verify()` y `context.Request.Certificate.VerifyNoRevocation()` funcionen.

> [!TIP]
> El problema de interbloqueo de certificados de cliente descrito en este [artículo](https://techcommunity.microsoft.com/t5/Networking-Blog/HTTPS-Client-Certificate-Request-freezes-when-the-Server-is/ba-p/339672) puede manifestarse de varias maneras, por ejemplo, las solicitudes se inmovilizan, las solicitudes resultan en un código de estado `403 Forbidden` después de agotar el tiempo, `context.Request.Certificate` es `null`. Este problema afecta normalmente a las solicitudes `POST` y `PUT` con la longitud del contenido de aproximadamente 60 KB o más.
> Para evitar que este problema se produzca, active "Negociar certificado de cliente" para los nombres de host deseados en la hoja "Dominios personalizados", tal como se muestra a continuación. Esta característica no está disponible en el nivel Consumo.

![Negociación del certificado de cliente](./media/api-management-howto-mutual-certificates-for-clients/negotiate-client-certificate.png)

## <a name="next-steps"></a>Pasos siguientes

-   [Cómo asegurar servicios back-end con la autenticación de certificados de cliente](./api-management-howto-mutual-certificates.md)
-   [¿Cómo se cargan certificados?](./api-management-howto-mutual-certificates.md)
